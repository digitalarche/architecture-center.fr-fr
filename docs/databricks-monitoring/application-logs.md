---
title: Envoyer les journaux des applications Azure Databricks à Azure Monitor
description: Comment envoyer des journaux personnalisés et les métriques à partir d’Azure Databricks à Azure Monitor
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: 49c631687fb3e3bbd807ffbbb49d9c5f6526bfb4
ms.sourcegitcommit: 9854bd27fb5cf92041bbfb743d43045cd3552a69
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/27/2019
ms.locfileid: "58503404"
---
# <a name="send-azure-databricks-application-logs-to-azure-monitor"></a><span data-ttu-id="6ddb9-103">Envoyer les journaux des applications Azure Databricks à Azure Monitor</span><span class="sxs-lookup"><span data-stu-id="6ddb9-103">Send Azure Databricks application logs to Azure Monitor</span></span>

<span data-ttu-id="6ddb9-104">Cet article explique comment envoyer des journaux des applications et des métriques à partir d’Azure Databricks pour un [espace de travail Analytique de journal](/azure/azure-monitor/platform/manage-access).</span><span class="sxs-lookup"><span data-stu-id="6ddb9-104">This article shows how to send application logs and metrics from Azure Databricks to a [Log Analytics workspace](/azure/azure-monitor/platform/manage-access).</span></span> <span data-ttu-id="6ddb9-105">Il utilise le [bibliothèque d’analyse Azure Databricks](https://github.com/mspnp/spark-monitoring), qui est disponible sur GitHub.</span><span class="sxs-lookup"><span data-stu-id="6ddb9-105">It uses the [Azure Databricks Monitoring Library](https://github.com/mspnp/spark-monitoring), which is available on GitHub.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="6ddb9-106">Conditions préalables</span><span class="sxs-lookup"><span data-stu-id="6ddb9-106">Prerequisites</span></span>

<span data-ttu-id="6ddb9-107">Configurez votre cluster Azure Databricks pour utiliser la bibliothèque d’analyse, comme décrit dans [configurer Azure Databricks pour envoyer des métriques à Azure Monitor][config-cluster].</span><span class="sxs-lookup"><span data-stu-id="6ddb9-107">Configure your Azure Databricks cluster to use the monitoring library, as described in [Configure Azure Databricks to send metrics to Azure Monitor][config-cluster].</span></span>

> [!NOTE]
> <span data-ttu-id="6ddb9-108">La bibliothèque d’analyse diffuse les événements de niveau d’Apache Spark et Spark Structured Streaming les métriques à partir de vos travaux à Azure Monitor.</span><span class="sxs-lookup"><span data-stu-id="6ddb9-108">The monitoring library streams Apache Spark level events and Spark Structured Streaming metrics from your jobs to Azure Monitor.</span></span> <span data-ttu-id="6ddb9-109">Vous n’avez pas besoin d’apporter des modifications à votre code d’application pour ces événements et les mesures.</span><span class="sxs-lookup"><span data-stu-id="6ddb9-109">You don't need to make any changes to your application code for these events and metrics.</span></span>

## <a name="send-application-metrics-using-dropwizard"></a><span data-ttu-id="6ddb9-110">Envoyer des mesures d’application à l’aide de Dropwizard</span><span class="sxs-lookup"><span data-stu-id="6ddb9-110">Send application metrics using Dropwizard</span></span>

<span data-ttu-id="6ddb9-111">Spark utilise un système de métriques configurable en fonction de la bibliothèque de métriques Dropwizard.</span><span class="sxs-lookup"><span data-stu-id="6ddb9-111">Spark uses a configurable metrics system based on the Dropwizard Metrics Library.</span></span> <span data-ttu-id="6ddb9-112">Pour plus d’informations, consultez [métriques](https://spark.apache.org/docs/latest/monitoring.html#metrics) dans la documentation Spark.</span><span class="sxs-lookup"><span data-stu-id="6ddb9-112">For more information, see [Metrics](https://spark.apache.org/docs/latest/monitoring.html#metrics) in the Spark documentation.</span></span>

<span data-ttu-id="6ddb9-113">Pour envoyer des mesures d’application à partir du code d’application Azure Databricks à Azure Monitor, procédez comme suit :</span><span class="sxs-lookup"><span data-stu-id="6ddb9-113">To send application metrics from Azure Databricks application code to Azure Monitor, follow these steps:</span></span>

1. <span data-ttu-id="6ddb9-114">Générer le **spark-écouteurs-loganalytics-1.0-snapshot.jar** fichier JAR comme décrit dans [générer la bibliothèque de surveillance Azure Databricks][build-lib].</span><span class="sxs-lookup"><span data-stu-id="6ddb9-114">Build the **spark-listeners-loganalytics-1.0-SNAPSHOT.jar** JAR file as described in [Build the Azure Databricks Monitoring Library][build-lib].</span></span>

1. <span data-ttu-id="6ddb9-115">Créer Dropwizard [jauges ou compteurs](https://metrics.dropwizard.io/4.0.0/manual/core.html) dans votre code d’application.</span><span class="sxs-lookup"><span data-stu-id="6ddb9-115">Create Dropwizard [gauges or counters](https://metrics.dropwizard.io/4.0.0/manual/core.html) in your application code.</span></span> <span data-ttu-id="6ddb9-116">Vous pouvez utiliser la `UserMetricsSystem` classe définie dans la bibliothèque d’analyse.</span><span class="sxs-lookup"><span data-stu-id="6ddb9-116">You can use the `UserMetricsSystem` class defined in the monitoring library.</span></span> <span data-ttu-id="6ddb9-117">L’exemple suivant crée un compteur nommé `counter1`.</span><span class="sxs-lookup"><span data-stu-id="6ddb9-117">The following example creates a counter named `counter1`.</span></span>

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

    <span data-ttu-id="6ddb9-118">La bibliothèque de surveillance comprend un [exemple d’application] [ sample-app] qui montre comment utiliser le `UserMetricsSystem` classe.</span><span class="sxs-lookup"><span data-stu-id="6ddb9-118">The monitoring library includes a [sample application][sample-app] that demonstrates how to use the `UserMetricsSystem` class.</span></span>

## <a name="send-application-logs-using-log4j"></a><span data-ttu-id="6ddb9-119">Envoyer les journaux des applications à l’aide de Log4j</span><span class="sxs-lookup"><span data-stu-id="6ddb9-119">Send application logs using Log4j</span></span>

<span data-ttu-id="6ddb9-120">Pour envoyer les journaux des applications Azure Databricks à l’aide de l’Analytique de journal Azure le [l’appender Log4j](https://logging.apache.org/log4j/2.x/manual/appenders.html) dans la bibliothèque, procédez comme suit :</span><span class="sxs-lookup"><span data-stu-id="6ddb9-120">To send your Azure Databricks application logs to Azure Log Analytics using the [Log4j appender](https://logging.apache.org/log4j/2.x/manual/appenders.html) in the library, follow these steps:</span></span>

1. <span data-ttu-id="6ddb9-121">Générer le **spark-écouteurs-loganalytics-1.0-snapshot.jar** fichier JAR comme décrit dans [générer la bibliothèque de surveillance Azure Databricks][build-lib].</span><span class="sxs-lookup"><span data-stu-id="6ddb9-121">Build the **spark-listeners-loganalytics-1.0-SNAPSHOT.jar** JAR file as described in [Build the Azure Databricks Monitoring Library][build-lib].</span></span>

1. <span data-ttu-id="6ddb9-122">Créer un **log4j.properties** [fichier de configuration](https://logging.apache.org/log4j/2.x/manual/configuration.html) pour votre application.</span><span class="sxs-lookup"><span data-stu-id="6ddb9-122">Create a **log4j.properties** [configuration file](https://logging.apache.org/log4j/2.x/manual/configuration.html) for your application.</span></span> <span data-ttu-id="6ddb9-123">Inclure les propriétés de configuration suivantes.</span><span class="sxs-lookup"><span data-stu-id="6ddb9-123">Include the following configuration properties.</span></span> <span data-ttu-id="6ddb9-124">Remplacez votre nom de package d’application et les journaux de niveau lorsque cela est indiqué :</span><span class="sxs-lookup"><span data-stu-id="6ddb9-124">Substitute your application package name and log level where indicated:</span></span>

    ```YAML
    log4j.appender.A1=com.microsoft.pnp.logging.loganalytics.LogAnalyticsAppender
    log4j.appender.A1.layout=com.microsoft.pnp.logging.JSONLayout
    log4j.appender.A1.layout.LocationInfo=false
    log4j.additivity.<your application package name>=false
    log4j.logger.<your application package name>=<log level>, A1
    ```

    <span data-ttu-id="6ddb9-125">Vous trouverez un exemple de fichier de configuration [ici][log4j.properties].</span><span class="sxs-lookup"><span data-stu-id="6ddb9-125">You can find a sample configuration file [here][log4j.properties].</span></span>

1. <span data-ttu-id="6ddb9-126">Dans votre code d’application, incluez le **spark-écouteurs-loganalytics** le projet, puis importer `com.microsoft.pnp.logging.Log4jconfiguration` à votre code d’application.</span><span class="sxs-lookup"><span data-stu-id="6ddb9-126">In your application code, include the **spark-listeners-loganalytics** project, and import `com.microsoft.pnp.logging.Log4jconfiguration` to your application code.</span></span>

    ```Scala
    import com.microsoft.pnp.logging.Log4jConfiguration
    ```

1. <span data-ttu-id="6ddb9-127">Configurer à l’aide de Log4j le **log4j.properties** fichier que vous avez créé à l’étape 3 :</span><span class="sxs-lookup"><span data-stu-id="6ddb9-127">Configure Log4j using the **log4j.properties** file you created in step 3:</span></span>

    ```Scala
    getClass.getResourceAsStream("<path to file in your JAR file>/log4j.properties")) {
          stream => {
            Log4jConfiguration.configure(stream)
          }
    }
    ```

1. <span data-ttu-id="6ddb9-128">Ajouter des messages de journal d’Apache Spark au niveau approprié dans votre code en fonction des besoins.</span><span class="sxs-lookup"><span data-stu-id="6ddb9-128">Add Apache Spark log messages at the appropriate level in your code as required.</span></span> <span data-ttu-id="6ddb9-129">Par exemple, utiliser le `logDebug` méthode pour envoyer un meesage de journal de débogage.</span><span class="sxs-lookup"><span data-stu-id="6ddb9-129">For example, use the `logDebug` method to send a debug log meesage.</span></span> <span data-ttu-id="6ddb9-130">Pour plus d’informations, consultez [journalisation] [ spark-logging] dans la documentation Spark.</span><span class="sxs-lookup"><span data-stu-id="6ddb9-130">For more information, see [Logging][spark-logging] in the Spark documentation.</span></span>

    ```Scala
    logTrace("Trace message")
    logDebug("Debug message")
    logInfo("Info message")
    logWarning("Warning message")
    logError("Error message")
    ```

## <a name="run-the-sample-application"></a><span data-ttu-id="6ddb9-131">Exécuter l’exemple d’application</span><span class="sxs-lookup"><span data-stu-id="6ddb9-131">Run the sample application</span></span>

<span data-ttu-id="6ddb9-132">La bibliothèque de surveillance comprend un [exemple d’application] [ sample-app] qui montre comment envoyer des mesures d’application et journaux d’application à Azure Monitor.</span><span class="sxs-lookup"><span data-stu-id="6ddb9-132">The monitoring library includes a [sample application][sample-app] that demonstrates how to send both application metrics and application logs to Azure Monitor.</span></span> <span data-ttu-id="6ddb9-133">Pour exécuter l'exemple :</span><span class="sxs-lookup"><span data-stu-id="6ddb9-133">To run the sample:</span></span>

1. <span data-ttu-id="6ddb9-134">Générer le **des travaux spark** de projet dans la bibliothèque d’analyse, comme décrit dans [générer la bibliothèque de surveillance Azure Databricks][build-lib].</span><span class="sxs-lookup"><span data-stu-id="6ddb9-134">Build the **spark-jobs** project in the monitoring library, as described in [Build the Azure Databricks Monitoring Library][build-lib].</span></span>

1. <span data-ttu-id="6ddb9-135">Accédez à votre espace de travail Databricks et créer un nouveau travail, comme décrit [ici](https://docs.azuredatabricks.net/user-guide/jobs.html#create-a-job).</span><span class="sxs-lookup"><span data-stu-id="6ddb9-135">Navigate to your Databricks workspace and create a new job, as described [here](https://docs.azuredatabricks.net/user-guide/jobs.html#create-a-job).</span></span>

1. <span data-ttu-id="6ddb9-136">Dans la page Détails du travail, sélectionnez **définir JAR**.</span><span class="sxs-lookup"><span data-stu-id="6ddb9-136">In the job detail page, select **Set JAR**.</span></span>

1. <span data-ttu-id="6ddb9-137">Téléchargez le fichier JAR à partir de `/src/spark-jobs/target/spark-jobs-1.0-SNAPSHOT.jar`.</span><span class="sxs-lookup"><span data-stu-id="6ddb9-137">Upload the JAR file from `/src/spark-jobs/target/spark-jobs-1.0-SNAPSHOT.jar`.</span></span>

1. <span data-ttu-id="6ddb9-138">Pour **classe principale**, entrez `com.microsoft.pnp.samplejob.StreamingQueryListenerSampleJob`.</span><span class="sxs-lookup"><span data-stu-id="6ddb9-138">For **Main class**, enter `com.microsoft.pnp.samplejob.StreamingQueryListenerSampleJob`.</span></span>

1. <span data-ttu-id="6ddb9-139">Sélectionnez un cluster qui est déjà configuré pour utiliser la bibliothèque d’analyse.</span><span class="sxs-lookup"><span data-stu-id="6ddb9-139">Select a cluster that is already configured to use the monitoring library.</span></span> <span data-ttu-id="6ddb9-140">Consultez [configurer Azure Databricks pour envoyer des métriques à Azure Monitor][config-cluster].</span><span class="sxs-lookup"><span data-stu-id="6ddb9-140">See [Configure Azure Databricks to send metrics to Azure Monitor][config-cluster].</span></span>

<span data-ttu-id="6ddb9-141">Lorsque le travail s’exécute, vous pouvez afficher les journaux des applications et les mesures dans votre espace de travail Analytique de journal.</span><span class="sxs-lookup"><span data-stu-id="6ddb9-141">When the job runs, you can view the application logs and metrics in your Log Analytics workspace.</span></span>

<span data-ttu-id="6ddb9-142">Journaux des applications s’affichent sous SparkLoggingEvent_CL :</span><span class="sxs-lookup"><span data-stu-id="6ddb9-142">Application logs appear under SparkLoggingEvent_CL:</span></span>

```Kusto
SparkLoggingEvent_CL | where logger_name_s contains "com.microsoft.pnp"
```

<span data-ttu-id="6ddb9-143">Mesures de l’application s’affichent sous SparkMetric_CL :</span><span class="sxs-lookup"><span data-stu-id="6ddb9-143">Application metrics appear under SparkMetric_CL:</span></span>

```Kusto
SparkMetric_CL | where name_s contains "rowcounter" | limit 50
```

## <a name="next-steps"></a><span data-ttu-id="6ddb9-144">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="6ddb9-144">Next steps</span></span>

<span data-ttu-id="6ddb9-145">Déployer le tableau de bord qui accompagne cette bibliothèque de code pour résoudre les problèmes de performances dans vos charges de travail Azure Databricks production de surveillance des performances.</span><span class="sxs-lookup"><span data-stu-id="6ddb9-145">Deploy the performance monitoring dashboard that accompanies this code library to troubleshoot performance issues in your production Azure Databricks workloads.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="6ddb9-146">Utiliser des tableaux de bord pour visualiser les métriques d’Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="6ddb9-146">Use dashboards to visualize Azure Databricks metrics</span></span>](./dashboards.md)

<!-- links -->

[build-lib]: ./configure-cluster.md##build-the-azure-databricks-monitoring-library
[config-cluster]: ./configure-cluster.md
[log4j.properties]: https://github.com/mspnp/spark-monitoring/blob/master/src/spark-jobs/src/main/resources/com/microsoft/pnp/samplejob/log4j.properties
[sample-app]: https://github.com/mspnp/spark-monitoring/tree/master/src/spark-jobs
[spark-logging]: https://spark.apache.org/docs/2.3.0/api/java/org/apache/spark/internal/Logging.html