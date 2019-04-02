---
title: Configurer Azure Databricks pour envoyer des métriques Azure Monitor
description: Une bibliothèque scala pour activer la surveillance des métriques et la journalisation des données dans Azure Log Analytique
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: af6b6433f87964ac60c179ecf498e54129344126
ms.sourcegitcommit: 9854bd27fb5cf92041bbfb743d43045cd3552a69
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/27/2019
ms.locfileid: "58503405"
---
<!-- markdownlint-disable MD040 -->

# <a name="configure-azure-databricks-to-send-metrics-to-azure-monitor"></a>Configurer Azure Databricks pour envoyer des métriques Azure Monitor

Cet article explique comment configurer un cluster Azure Databricks pour envoyer des mesures à un [espace de travail Analytique de journal](/azure/azure-monitor/platform/manage-access). Il utilise le [bibliothèque d’analyse Azure Databricks](https://github.com/mspnp/spark-monitoring), qui est disponible sur GitHub. Présentation de Java, Scala et Maven sont recommandées comme prerequisistes.

## <a name="about-the-azure-databricks-monitoring-library"></a>À propos de la bibliothèque d’analyse de Databricks Azure

Le [référentiel GitHub](https://github.com/mspnp/spark-monitoring) pour la bibliothèque d’analyse Azure Databricks a suivant la structure de répertoires :

```
/src  
    /spark-jobs  
    /spark-listeners-loganalytics  
    /spark-listeners  
    /pom.xml  
```

Le **des travaux spark** directory est un exemple d’application Spark qui montre comment implémenter un compteur de métrique application Spark.

Le **spark-écouteurs** directory inclut une fonctionnalité qui permet à Azure Databricks envoyer des événements de Apache Spark au niveau du service à un espace de travail Analytique des journaux Azure. Azure Databricks est un service basé sur Apache Spark, qui inclut un ensemble d’API structurées pour les données à l’aide de SQL, les trames de données et les jeux de données de traitement par lots. Avec Apache Spark 2.0, la prise en charge a été ajoutée pour [Structured Streaming](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html), une API basée sur les API de traitement de lot de Spark de traitement de flux de données.

Le **spark-écouteurs-loganalytics** répertoire inclut un récepteur pour les écouteurs, un récepteur pour DropWizard et un client pour un espace de travail Analytique de journal Azure Spark. Ce répertoire inclut également un Appender log4j pour les journaux des applications Apache Spark.

Le **spark-écouteurs-loganalytics** et **spark-écouteurs** répertoires contiennent le code pour générer les deux fichiers JAR qui sont déployés sur le cluster Databricks. Le **spark-écouteurs** répertoire inclut un **scripts** répertoire qui contient un script de l’initialisation de nœud de cluster pour copier le fichier JAR de fichiers à partir d’un répertoire de transit dans le système de fichiers Azure Databricks nœuds d’exécution.

Le **pom.xml** fichier est le principal fichier de build Maven pour l’ensemble du projet.

## <a name="prerequisites"></a>Conditions préalables

Pour commencer, déployez les ressources Azure suivantes :

- Un espace de travail Azure Databricks. Consultez [Démarrage rapide : Exécuter un travail Spark sur Azure Databricks à l’aide du portail Azure](/azure/azure-databricks/quickstart-create-databricks-workspace-portal).
- Un espace de travail Log Analytics. Consultez [créer un espace de travail Analytique de journal dans le portail Azure](/azure/azure-monitor/learn/quick-create-workspace).

Ensuite, installez le [Azure Databricks CLI](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html#install-the-cli). Un jeton d’accès personnel Azure Databricks est nécessaire pour utiliser l’interface CLI. Pour obtenir des instructions, consultez [générer un jeton](https://docs.azuredatabricks.net/api/latest/authentication.html#token-management).

Trouver votre ID d’espace de travail Analytique de journal et la clé. Pour obtenir des instructions, consultez [obtenir l’ID de l’espace de travail et la clé](/azure/azure-monitor/platform/agent-windows#obtain-workspace-id-and-key). Vous devez ces valeurs lorsque vous configurez le cluster Databricks.

Pour générer la bibliothèque d’analyse, vous devrez un IDE Java avec les ressources suivantes :

- [Kit de développement Java (JDK) version 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
- [Langage Scala 2.11 du Kit de développement logiciel](https://www.scala-lang.org/download/)
- [Apache Maven 3.5.4](http://maven.apache.org/download.cgi)

## <a name="build-the-azure-databricks-monitoring-library"></a>Générer la bibliothèque d’analyse de Databricks Azure

Pour générer la bibliothèque d’analyse Azure Databricks, procédez comme suit :

1. Clonez, dupliquez ou téléchargez le [référentiel GitHub](https://github.com/mspnp/spark-monitoring).

1. Importer le fichier de modèle objet de projet Maven, _pom.xml_, situé dans le **/src** dossier dans votre projet. Cette opération va importer les trois projets :

    - travaux Spark
    - écouteurs de Spark
    - spark-listeners-loganalytics

1. Exécuter le Maven **package** phase de génération dans votre IDE Java pour générer les fichiers JAR pour chacun de ces projets :

    |Projet| Fichier JAR|
    |-------|---------|
    |travaux Spark|Spark-tâches-1.0-snapshot.jar|
    |écouteurs de Spark|Spark-écouteurs-1.0-snapshot.jar|
    |spark-listeners-loganalytics|Spark-écouteurs-loganalytics-1.0-snapshot.jar|

1. Utiliser Azure Databricks CLI pour créer un répertoire nommé **dbfs : / databricks/surveillance-mise en lots**:  

    ```bash
    dbfs mkdirs dbfs:/databricks/monitoring-staging
    ```

1. Permet de copier l’interface CLI Databricks **/src/spark-listeners/scripts/listeners.sh** dans le répertoire précédemment :

    ```bash
    dbfs cp ./src/spark-listeners/scripts/listeners.sh dbfs:/databricks/monitoring-staging/listeners.sh
    ```

1. Permet de copier l’interface CLI Databricks **/src/spark-listeners/scripts/metrics.properties** au répertoire créé précédemment :

    ```bash
    dbfs cp <local path to metrics.properties> dbfs:/databricks/monitoring-staging/metrics.properties
    ```

1. Permet de copier l’interface CLI Databricks **spark-écouteurs-1.0-snapshot.jar** et **spark-écouteurs-loganalytics-1.0-snapshot.jar** au répertoire créé précédemment :

    ```bash
    dbfs cp ./src/spark-listeners/target/spark-listeners-1.0-SNAPSHOT.jar dbfs:/databricks/monitoring-staging/spark-listeners-1.0-SNAPSHOT.jar
    dbfs cp ./src/spark-listeners-loganalytics/target/spark-listeners-loganalytics-1.0-SNAPSHOT.jar dbfs:/databricks/monitoring-staging/spark-listeners-loganalytics-1.0-SNAPSHOT.jar
    ```

## <a name="create-and-configure-an-azure-databricks-cluster"></a>Créer et configurer un cluster Azure Databricks

Pour créer et configurer un cluster Azure Databricks, procédez comme suit :

1. Accédez à votre espace de travail Azure Databricks dans le portail Azure.
1. Dans la page d’accueil, cliquez sur **nouveau Cluster**.
1. Entrez un nom pour votre cluster dans le **nom du Cluster** zone de texte.
1. Dans le **Version du Runtime Databricks** liste déroulante, sélectionnez **4.3** ou supérieur.
1. Sous **Options avancées**, cliquez sur le **Spark** onglet. Entrez les paires nom-valeur suivantes dans le **Spark configuration** zone de texte :

    ```
    spark.extraListeners com.databricks.backend.daemon.driver.DBCEventLoggingListener,org.apache.spark.listeners.UnifiedSparkListener
    spark.unifiedListener.sink org.apache.spark.listeners.sink.loganalytics.LogAnalyticsListenerSink
    spark.unifiedListener.logBlockUpdates false
    ```

1. Bien que toujours sous le **Spark** , entrez les valeurs suivantes dans le **Variables d’environnement** zone de texte :

    ```
    LOG_ANALYTICS_WORKSPACE_ID=[your Azure Log Analytics workspace ID]
    LOG_ANALYTICS_WORKSPACE_KEY=[your Azure Log Analytics workspace key]
    ```

    ![Capture d’écran de l’interface utilisateur de Databricks](./_images/create-cluster1.png)

1. Bien que toujours sous le **Options avancées** section, cliquez sur le **Scripts Init** onglet.
1. Dans le **Destination** liste déroulante, sélectionnez **DBFS**.
1. Sous **chemin d’accès du Script Init**, entrez `dbfs:/databricks/monitoring-staging/listeners.sh`.
1. Cliquez sur **Add**.

    ![Capture d’écran de l’interface utilisateur de Databricks](./_images/create-cluster2.png)

1. Cliquez sur **création d’un Cluster** pour créer le cluster.

## <a name="view-metrics"></a>Afficher les mesures

Après avoir effectué ces étapes, votre cluster Databricks transmet en continu des données métriques sur le cluster lui-même pour Azure Monitor. Données de journal soient disponibles dans votre espace de travail Analytique de journal Azure sous la « Active | Journaux personnalisés | Schéma de SparkMetric_CL ». Pour plus d’informations sur les types de mesures qui sont consignés, consultez [métriques Core](https://metrics.dropwizard.io/4.0.0/manual/core.html) dans la documentation Dropwizard. Le type de mesure, telles que la jauge, compteur ou d’histogramme, est écrit dans le champ de Type.

En outre, la bibliothèque diffuse les événements de niveau d’Apache Spark et Spark Structured Streaming les métriques à partir de vos travaux à Azure Monitor. Vous n’avez pas besoin d’apporter des modifications à votre code d’application pour ces événements et les mesures. Spark Structured Streaming de données de journal sont disponibles sous les « Active | Journaux personnalisés | Schéma de SparkListenerEvent_CL ».

![Capture d’écran d’un espace de travail Analytique de journal](./_images/workspace.png)

Pour plus d’informations sur l’affichage des journaux, consultez [visualiser et analyser les données de journal dans Azure Monitor](/azure/azure-monitor/log-query/portals).

## <a name="next-steps"></a>Étapes suivantes

Cet article décrit comment configurer votre cluster pour envoyer des métriques Azure Monitor. L’article suivant montre comment utiliser la bibliothèque d’analyse Azure Databricks pour envoyer des journaux et des métriques de l’application.

> [!div class="nextstepaction"]
> [Envoyer les journaux des applications à Azure Monitor](./application-logs.md)
