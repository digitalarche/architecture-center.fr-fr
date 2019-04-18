---
title: Envoyer les journaux des applications Azure Databricks à Azure Monitor
description: Comment envoyer des journaux personnalisés et les métriques à partir d’Azure Databricks à Azure Monitor
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: ea67122d7871663e8aaf42b7af0043492f63b6b1
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59639182"
---
# <a name="send-azure-databricks-application-logs-to-azure-monitor"></a>Envoyer les journaux des applications Azure Databricks à Azure Monitor

Cet article explique comment envoyer des journaux des applications et des métriques à partir d’Azure Databricks pour un [espace de travail Analytique de journal](/azure/azure-monitor/platform/manage-access). Il utilise le [bibliothèque d’analyse Azure Databricks](https://github.com/mspnp/spark-monitoring), qui est disponible sur GitHub.

## <a name="prerequisites"></a>Conditions préalables

Configurez votre cluster Azure Databricks pour utiliser la bibliothèque d’analyse, comme décrit dans [configurer Azure Databricks pour envoyer des métriques à Azure Monitor][config-cluster].

> [!NOTE]
> La bibliothèque d’analyse diffuse les événements de niveau d’Apache Spark et Spark Structured Streaming les métriques à partir de vos travaux à Azure Monitor. Vous n’avez pas besoin d’apporter des modifications à votre code d’application pour ces événements et les mesures.

## <a name="send-application-metrics-using-dropwizard"></a>Envoyer des mesures d’application à l’aide de Dropwizard

Spark utilise un système de métriques configurable en fonction de la bibliothèque de métriques Dropwizard. Pour plus d’informations, consultez [métriques](https://spark.apache.org/docs/latest/monitoring.html#metrics) dans la documentation Spark.

Pour envoyer des mesures d’application à partir du code d’application Azure Databricks à Azure Monitor, procédez comme suit :

1. Générer le **spark-écouteurs-loganalytics-1.0-snapshot.jar** fichier JAR comme décrit dans [générer la bibliothèque de surveillance Azure Databricks][build-lib].

1. Créer Dropwizard [jauges ou compteurs](https://metrics.dropwizard.io/4.0.0/manual/core.html) dans votre code d’application. Vous pouvez utiliser la `UserMetricsSystem` classe définie dans la bibliothèque d’analyse. L’exemple suivant crée un compteur nommé `counter1`.

    ```Scala
    import org.apache.spark.metrics.UserMetricsSystems
    import org.apache.spark.sql.SparkSession

    object StreamingQueryListenerSampleJob  {

      private final val METRICS_NAMESPACE = "samplejob"
      private final val COUNTER_NAME = "counter1"

      def main(args: Array[String]): Unit = {

        val spark = SparkSession
          .builder
          .getOrCreate

        val driverMetricsSystem = UserMetricsSystems
            .getMetricSystem(METRICS_NAMESPACE, builder => {
              builder.registerCounter(COUNTER_NAME)
            })

        driverMetricsSystem.counter(COUNTER_NAME).inc(5)
      }
    }
    ```

    La bibliothèque de surveillance comprend un [exemple d’application] [ sample-app] qui montre comment utiliser le `UserMetricsSystem` classe.

## <a name="send-application-logs-using-log4j"></a>Envoyer les journaux des applications à l’aide de Log4j

Pour envoyer les journaux des applications Azure Databricks à l’aide de l’Analytique de journal Azure le [l’appender Log4j](https://logging.apache.org/log4j/2.x/manual/appenders.html) dans la bibliothèque, procédez comme suit :

1. Générer le **spark-écouteurs-loganalytics-1.0-snapshot.jar** fichier JAR comme décrit dans [générer la bibliothèque de surveillance Azure Databricks][build-lib].

1. Créer un **log4j.properties** [fichier de configuration](https://logging.apache.org/log4j/2.x/manual/configuration.html) pour votre application. Inclure les propriétés de configuration suivantes. Remplacez votre nom de package d’application et les journaux de niveau lorsque cela est indiqué :

    ```YAML
    log4j.appender.A1=com.microsoft.pnp.logging.loganalytics.LogAnalyticsAppender
    log4j.appender.A1.layout=com.microsoft.pnp.logging.JSONLayout
    log4j.appender.A1.layout.LocationInfo=false
    log4j.additivity.<your application package name>=false
    log4j.logger.<your application package name>=<log level>, A1
    ```

    Vous trouverez un exemple de fichier de configuration [ici][log4j.properties].

1. Dans votre code d’application, incluez le **spark-écouteurs-loganalytics** le projet, puis importer `com.microsoft.pnp.logging.Log4jconfiguration` à votre code d’application.

    ```Scala
    import com.microsoft.pnp.logging.Log4jConfiguration
    ```

1. Configurer à l’aide de Log4j le **log4j.properties** fichier que vous avez créé à l’étape 3 :

    ```Scala
    getClass.getResourceAsStream("<path to file in your JAR file>/log4j.properties")) {
          stream => {
            Log4jConfiguration.configure(stream)
          }
    }
    ```

1. Ajouter des messages de journal d’Apache Spark au niveau approprié dans votre code en fonction des besoins. Par exemple, utiliser le `logDebug` méthode pour envoyer un message de journal de débogage. Pour plus d’informations, consultez [journalisation] [ spark-logging] dans la documentation Spark.

    ```Scala
    logTrace("Trace message")
    logDebug("Debug message")
    logInfo("Info message")
    logWarning("Warning message")
    logError("Error message")
    ```

## <a name="run-the-sample-application"></a>Exécuter l’exemple d’application

La bibliothèque de surveillance comprend un [exemple d’application] [ sample-app] qui montre comment envoyer des mesures d’application et journaux d’application à Azure Monitor. Pour exécuter l'exemple :

1. Générer le **des travaux spark** de projet dans la bibliothèque d’analyse, comme décrit dans [générer la bibliothèque de surveillance Azure Databricks][build-lib].

1. Accédez à votre espace de travail Databricks et créer un nouveau travail, comme décrit [ici](https://docs.azuredatabricks.net/user-guide/jobs.html#create-a-job).

1. Dans la page Détails du travail, sélectionnez **définir JAR**.

1. Téléchargez le fichier JAR à partir de `/src/spark-jobs/target/spark-jobs-1.0-SNAPSHOT.jar`.

1. Pour **classe principale**, entrez `com.microsoft.pnp.samplejob.StreamingQueryListenerSampleJob`.

1. Sélectionnez un cluster qui est déjà configuré pour utiliser la bibliothèque d’analyse. Consultez [configurer Azure Databricks pour envoyer des métriques à Azure Monitor][config-cluster].

Lorsque le travail s’exécute, vous pouvez afficher les journaux des applications et les mesures dans votre espace de travail Analytique de journal.

Journaux des applications s’affichent sous SparkLoggingEvent_CL :

```Kusto
SparkLoggingEvent_CL | where logger_name_s contains "com.microsoft.pnp"
```

Mesures de l’application s’affichent sous SparkMetric_CL :

```Kusto
SparkMetric_CL | where name_s contains "rowcounter" | limit 50
```

## <a name="next-steps"></a>Étapes suivantes

Déployer le tableau de bord qui accompagne cette bibliothèque de code pour résoudre les problèmes de performances dans vos charges de travail Azure Databricks production de surveillance des performances.

> [!div class="nextstepaction"]
> [Utiliser des tableaux de bord pour visualiser les métriques Azure Databricks](./dashboards.md)

<!-- links -->

[build-lib]: ./configure-cluster.md##build-the-azure-databricks-monitoring-library
[config-cluster]: ./configure-cluster.md
[log4j.properties]: https://github.com/mspnp/spark-monitoring/blob/master/src/spark-jobs/src/main/resources/com/microsoft/pnp/samplejob/log4j.properties
[sample-app]: https://github.com/mspnp/spark-monitoring/tree/master/src/spark-jobs
[spark-logging]: https://spark.apache.org/docs/2.3.0/api/java/org/apache/spark/internal/Logging.html