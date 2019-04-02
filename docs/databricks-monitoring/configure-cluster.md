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

# <a name="configure-azure-databricks-to-send-metrics-to-azure-monitor"></a><span data-ttu-id="e5df5-103">Configurer Azure Databricks pour envoyer des métriques Azure Monitor</span><span class="sxs-lookup"><span data-stu-id="e5df5-103">Configure Azure Databricks to send metrics to Azure Monitor</span></span>

<span data-ttu-id="e5df5-104">Cet article explique comment configurer un cluster Azure Databricks pour envoyer des mesures à un [espace de travail Analytique de journal](/azure/azure-monitor/platform/manage-access).</span><span class="sxs-lookup"><span data-stu-id="e5df5-104">This article shows how to configure an Azure Databricks cluster to send metrics to a [Log Analytics workspace](/azure/azure-monitor/platform/manage-access).</span></span> <span data-ttu-id="e5df5-105">Il utilise le [bibliothèque d’analyse Azure Databricks](https://github.com/mspnp/spark-monitoring), qui est disponible sur GitHub.</span><span class="sxs-lookup"><span data-stu-id="e5df5-105">It uses the [Azure Databricks Monitoring Library](https://github.com/mspnp/spark-monitoring), which is available on GitHub.</span></span> <span data-ttu-id="e5df5-106">Présentation de Java, Scala et Maven sont recommandées comme prerequisistes.</span><span class="sxs-lookup"><span data-stu-id="e5df5-106">Understanding of Java, Scala, and Maven are recommended as prerequisistes.</span></span>

## <a name="about-the-azure-databricks-monitoring-library"></a><span data-ttu-id="e5df5-107">À propos de la bibliothèque d’analyse de Databricks Azure</span><span class="sxs-lookup"><span data-stu-id="e5df5-107">About the Azure Databricks Monitoring Library</span></span>

<span data-ttu-id="e5df5-108">Le [référentiel GitHub](https://github.com/mspnp/spark-monitoring) pour la bibliothèque d’analyse Azure Databricks a suivant la structure de répertoires :</span><span class="sxs-lookup"><span data-stu-id="e5df5-108">The [GitHub repo](https://github.com/mspnp/spark-monitoring) for the Azure Databricks Monitoring Library has following directory structure:</span></span>

```
/src  
    /spark-jobs  
    /spark-listeners-loganalytics  
    /spark-listeners  
    /pom.xml  
```

<span data-ttu-id="e5df5-109">Le **des travaux spark** directory est un exemple d’application Spark qui montre comment implémenter un compteur de métrique application Spark.</span><span class="sxs-lookup"><span data-stu-id="e5df5-109">The **spark-jobs** directory is a sample Spark application demonstrating how to implement a Spark application metric counter.</span></span>

<span data-ttu-id="e5df5-110">Le **spark-écouteurs** directory inclut une fonctionnalité qui permet à Azure Databricks envoyer des événements de Apache Spark au niveau du service à un espace de travail Analytique des journaux Azure.</span><span class="sxs-lookup"><span data-stu-id="e5df5-110">The **spark-listeners** directory includes functionality that enables Azure Databricks to send Apache Spark events at the service level to an Azure Log Analytics workspace.</span></span> <span data-ttu-id="e5df5-111">Azure Databricks est un service basé sur Apache Spark, qui inclut un ensemble d’API structurées pour les données à l’aide de SQL, les trames de données et les jeux de données de traitement par lots.</span><span class="sxs-lookup"><span data-stu-id="e5df5-111">Azure Databricks is a service based on Apache Spark, which includes a set of structured APIs for batch processing data using Datasets, DataFrames, and SQL.</span></span> <span data-ttu-id="e5df5-112">Avec Apache Spark 2.0, la prise en charge a été ajoutée pour [Structured Streaming](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html), une API basée sur les API de traitement de lot de Spark de traitement de flux de données.</span><span class="sxs-lookup"><span data-stu-id="e5df5-112">With Apache Spark 2.0, support was added for [Structured Streaming](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html), a data stream processing API built on Spark's batch processing APIs.</span></span>

<span data-ttu-id="e5df5-113">Le **spark-écouteurs-loganalytics** répertoire inclut un récepteur pour les écouteurs, un récepteur pour DropWizard et un client pour un espace de travail Analytique de journal Azure Spark.</span><span class="sxs-lookup"><span data-stu-id="e5df5-113">The **spark-listeners-loganalytics** directory includes a sink for Spark listeners, a sink for DropWizard, and a client for an Azure Log Analytics Workspace.</span></span> <span data-ttu-id="e5df5-114">Ce répertoire inclut également un Appender log4j pour les journaux des applications Apache Spark.</span><span class="sxs-lookup"><span data-stu-id="e5df5-114">This directory also includes a log4j Appender for your Apache Spark application logs.</span></span>

<span data-ttu-id="e5df5-115">Le **spark-écouteurs-loganalytics** et **spark-écouteurs** répertoires contiennent le code pour générer les deux fichiers JAR qui sont déployés sur le cluster Databricks.</span><span class="sxs-lookup"><span data-stu-id="e5df5-115">The **spark-listeners-loganalytics** and **spark-listeners** directories contain the code for building the two JAR files that are deployed to the Databricks cluster.</span></span> <span data-ttu-id="e5df5-116">Le **spark-écouteurs** répertoire inclut un **scripts** répertoire qui contient un script de l’initialisation de nœud de cluster pour copier le fichier JAR de fichiers à partir d’un répertoire de transit dans le système de fichiers Azure Databricks nœuds d’exécution.</span><span class="sxs-lookup"><span data-stu-id="e5df5-116">The **spark-listeners** directory includes a **scripts** directory that contains a cluster node initialization script to copy the JAR files from a staging directory in the Azure Databricks file system to execution nodes.</span></span>

<span data-ttu-id="e5df5-117">Le **pom.xml** fichier est le principal fichier de build Maven pour l’ensemble du projet.</span><span class="sxs-lookup"><span data-stu-id="e5df5-117">The **pom.xml** file is the main Maven build file for the entire project.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="e5df5-118">Conditions préalables</span><span class="sxs-lookup"><span data-stu-id="e5df5-118">Prerequisites</span></span>

<span data-ttu-id="e5df5-119">Pour commencer, déployez les ressources Azure suivantes :</span><span class="sxs-lookup"><span data-stu-id="e5df5-119">To get started, deploy the following Azure resources:</span></span>

- <span data-ttu-id="e5df5-120">Un espace de travail Azure Databricks.</span><span class="sxs-lookup"><span data-stu-id="e5df5-120">An Azure Databricks workspace.</span></span> <span data-ttu-id="e5df5-121">Consultez [Démarrage rapide : Exécuter un travail Spark sur Azure Databricks à l’aide du portail Azure](/azure/azure-databricks/quickstart-create-databricks-workspace-portal).</span><span class="sxs-lookup"><span data-stu-id="e5df5-121">See [Quickstart: Run a Spark job on Azure Databricks using the Azure portal](/azure/azure-databricks/quickstart-create-databricks-workspace-portal).</span></span>
- <span data-ttu-id="e5df5-122">Un espace de travail Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="e5df5-122">A Log Analytics workspace.</span></span> <span data-ttu-id="e5df5-123">Consultez [créer un espace de travail Analytique de journal dans le portail Azure](/azure/azure-monitor/learn/quick-create-workspace).</span><span class="sxs-lookup"><span data-stu-id="e5df5-123">See [Create a Log Analytics workspace in the Azure portal](/azure/azure-monitor/learn/quick-create-workspace).</span></span>

<span data-ttu-id="e5df5-124">Ensuite, installez le [Azure Databricks CLI](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html#install-the-cli).</span><span class="sxs-lookup"><span data-stu-id="e5df5-124">Next, install the [Azure Databricks CLI](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html#install-the-cli).</span></span> <span data-ttu-id="e5df5-125">Un jeton d’accès personnel Azure Databricks est nécessaire pour utiliser l’interface CLI.</span><span class="sxs-lookup"><span data-stu-id="e5df5-125">An Azure Databricks personal access token is required to use the CLI.</span></span> <span data-ttu-id="e5df5-126">Pour obtenir des instructions, consultez [générer un jeton](https://docs.azuredatabricks.net/api/latest/authentication.html#token-management).</span><span class="sxs-lookup"><span data-stu-id="e5df5-126">For instructions, see [Generate a token](https://docs.azuredatabricks.net/api/latest/authentication.html#token-management).</span></span>

<span data-ttu-id="e5df5-127">Trouver votre ID d’espace de travail Analytique de journal et la clé.</span><span class="sxs-lookup"><span data-stu-id="e5df5-127">Find your Log Analytics workspace ID and key.</span></span> <span data-ttu-id="e5df5-128">Pour obtenir des instructions, consultez [obtenir l’ID de l’espace de travail et la clé](/azure/azure-monitor/platform/agent-windows#obtain-workspace-id-and-key).</span><span class="sxs-lookup"><span data-stu-id="e5df5-128">For instructions, see [Obtain workspace ID and key](/azure/azure-monitor/platform/agent-windows#obtain-workspace-id-and-key).</span></span> <span data-ttu-id="e5df5-129">Vous devez ces valeurs lorsque vous configurez le cluster Databricks.</span><span class="sxs-lookup"><span data-stu-id="e5df5-129">You will need these values when you configure the Databricks cluster.</span></span>

<span data-ttu-id="e5df5-130">Pour générer la bibliothèque d’analyse, vous devrez un IDE Java avec les ressources suivantes :</span><span class="sxs-lookup"><span data-stu-id="e5df5-130">To build the monitoring library, you will need a Java IDE with the following resources:</span></span>

- [<span data-ttu-id="e5df5-131">Kit de développement Java (JDK) version 1.8</span><span class="sxs-lookup"><span data-stu-id="e5df5-131">Java Development Kit (JDK) version 1.8</span></span>](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
- [<span data-ttu-id="e5df5-132">Langage Scala 2.11 du Kit de développement logiciel</span><span class="sxs-lookup"><span data-stu-id="e5df5-132">Scala language SDK 2.11</span></span>](https://www.scala-lang.org/download/)
- [<span data-ttu-id="e5df5-133">Apache Maven 3.5.4</span><span class="sxs-lookup"><span data-stu-id="e5df5-133">Apache Maven 3.5.4</span></span>](http://maven.apache.org/download.cgi)

## <a name="build-the-azure-databricks-monitoring-library"></a><span data-ttu-id="e5df5-134">Générer la bibliothèque d’analyse de Databricks Azure</span><span class="sxs-lookup"><span data-stu-id="e5df5-134">Build the Azure Databricks Monitoring Library</span></span>

<span data-ttu-id="e5df5-135">Pour générer la bibliothèque d’analyse Azure Databricks, procédez comme suit :</span><span class="sxs-lookup"><span data-stu-id="e5df5-135">To build the Azure Databricks Monitoring Library, perform the following steps:</span></span>

1. <span data-ttu-id="e5df5-136">Clonez, dupliquez ou téléchargez le [référentiel GitHub](https://github.com/mspnp/spark-monitoring).</span><span class="sxs-lookup"><span data-stu-id="e5df5-136">Clone, fork, or download the [GitHub repository](https://github.com/mspnp/spark-monitoring).</span></span>

1. <span data-ttu-id="e5df5-137">Importer le fichier de modèle objet de projet Maven, _pom.xml_, situé dans le **/src** dossier dans votre projet.</span><span class="sxs-lookup"><span data-stu-id="e5df5-137">Import the Maven project object model file, _pom.xml_, located in the **/src** folder into your project.</span></span> <span data-ttu-id="e5df5-138">Cette opération va importer les trois projets :</span><span class="sxs-lookup"><span data-stu-id="e5df5-138">This will import three projects:</span></span>

    - <span data-ttu-id="e5df5-139">travaux Spark</span><span class="sxs-lookup"><span data-stu-id="e5df5-139">spark-jobs</span></span>
    - <span data-ttu-id="e5df5-140">écouteurs de Spark</span><span class="sxs-lookup"><span data-stu-id="e5df5-140">spark-listeners</span></span>
    - <span data-ttu-id="e5df5-141">spark-listeners-loganalytics</span><span class="sxs-lookup"><span data-stu-id="e5df5-141">spark-listeners-loganalytics</span></span>

1. <span data-ttu-id="e5df5-142">Exécuter le Maven **package** phase de génération dans votre IDE Java pour générer les fichiers JAR pour chacun de ces projets :</span><span class="sxs-lookup"><span data-stu-id="e5df5-142">Execute the Maven **package** build phase in your Java IDE to build the JAR files for each of these projects:</span></span>

    |<span data-ttu-id="e5df5-143">Projet</span><span class="sxs-lookup"><span data-stu-id="e5df5-143">Project</span></span>| <span data-ttu-id="e5df5-144">Fichier JAR</span><span class="sxs-lookup"><span data-stu-id="e5df5-144">JAR file</span></span>|
    |-------|---------|
    |<span data-ttu-id="e5df5-145">travaux Spark</span><span class="sxs-lookup"><span data-stu-id="e5df5-145">spark-jobs</span></span>|<span data-ttu-id="e5df5-146">Spark-tâches-1.0-snapshot.jar</span><span class="sxs-lookup"><span data-stu-id="e5df5-146">spark-jobs-1.0-SNAPSHOT.jar</span></span>|
    |<span data-ttu-id="e5df5-147">écouteurs de Spark</span><span class="sxs-lookup"><span data-stu-id="e5df5-147">spark-listeners</span></span>|<span data-ttu-id="e5df5-148">Spark-écouteurs-1.0-snapshot.jar</span><span class="sxs-lookup"><span data-stu-id="e5df5-148">spark-listeners-1.0-SNAPSHOT.jar</span></span>|
    |<span data-ttu-id="e5df5-149">spark-listeners-loganalytics</span><span class="sxs-lookup"><span data-stu-id="e5df5-149">spark-listeners-loganalytics</span></span>|<span data-ttu-id="e5df5-150">Spark-écouteurs-loganalytics-1.0-snapshot.jar</span><span class="sxs-lookup"><span data-stu-id="e5df5-150">spark-listeners-loganalytics-1.0-SNAPSHOT.jar</span></span>|

1. <span data-ttu-id="e5df5-151">Utiliser Azure Databricks CLI pour créer un répertoire nommé **dbfs : / databricks/surveillance-mise en lots**:</span><span class="sxs-lookup"><span data-stu-id="e5df5-151">Use the Azure Databricks CLI to create a directory named **dbfs:/databricks/monitoring-staging**:</span></span>  

    ```bash
    dbfs mkdirs dbfs:/databricks/monitoring-staging
    ```

1. <span data-ttu-id="e5df5-152">Permet de copier l’interface CLI Databricks **/src/spark-listeners/scripts/listeners.sh** dans le répertoire précédemment :</span><span class="sxs-lookup"><span data-stu-id="e5df5-152">Use the Azure Databricks CLI to copy **/src/spark-listeners/scripts/listeners.sh** to the directory previously:</span></span>

    ```bash
    dbfs cp ./src/spark-listeners/scripts/listeners.sh dbfs:/databricks/monitoring-staging/listeners.sh
    ```

1. <span data-ttu-id="e5df5-153">Permet de copier l’interface CLI Databricks **/src/spark-listeners/scripts/metrics.properties** au répertoire créé précédemment :</span><span class="sxs-lookup"><span data-stu-id="e5df5-153">Use the Azure Databricks CLI to copy **/src/spark-listeners/scripts/metrics.properties** to the directory created previously:</span></span>

    ```bash
    dbfs cp <local path to metrics.properties> dbfs:/databricks/monitoring-staging/metrics.properties
    ```

1. <span data-ttu-id="e5df5-154">Permet de copier l’interface CLI Databricks **spark-écouteurs-1.0-snapshot.jar** et **spark-écouteurs-loganalytics-1.0-snapshot.jar** au répertoire créé précédemment :</span><span class="sxs-lookup"><span data-stu-id="e5df5-154">Use the Azure Databricks CLI to copy **spark-listeners-1.0-SNAPSHOT.jar** and **spark-listeners-loganalytics-1.0-SNAPSHOT.jar** to the directory created previously:</span></span>

    ```bash
    dbfs cp ./src/spark-listeners/target/spark-listeners-1.0-SNAPSHOT.jar dbfs:/databricks/monitoring-staging/spark-listeners-1.0-SNAPSHOT.jar
    dbfs cp ./src/spark-listeners-loganalytics/target/spark-listeners-loganalytics-1.0-SNAPSHOT.jar dbfs:/databricks/monitoring-staging/spark-listeners-loganalytics-1.0-SNAPSHOT.jar
    ```

## <a name="create-and-configure-an-azure-databricks-cluster"></a><span data-ttu-id="e5df5-155">Créer et configurer un cluster Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="e5df5-155">Create and configure an Azure Databricks cluster</span></span>

<span data-ttu-id="e5df5-156">Pour créer et configurer un cluster Azure Databricks, procédez comme suit :</span><span class="sxs-lookup"><span data-stu-id="e5df5-156">To create and configure an Azure Databricks cluster, follow these steps:</span></span>

1. <span data-ttu-id="e5df5-157">Accédez à votre espace de travail Azure Databricks dans le portail Azure.</span><span class="sxs-lookup"><span data-stu-id="e5df5-157">Navigate to your Azure Databricks workspace in the Azure portal.</span></span>
1. <span data-ttu-id="e5df5-158">Dans la page d’accueil, cliquez sur **nouveau Cluster**.</span><span class="sxs-lookup"><span data-stu-id="e5df5-158">On the home page, click **New Cluster**.</span></span>
1. <span data-ttu-id="e5df5-159">Entrez un nom pour votre cluster dans le **nom du Cluster** zone de texte.</span><span class="sxs-lookup"><span data-stu-id="e5df5-159">Enter a name for your cluster in the **Cluster Name** text box.</span></span>
1. <span data-ttu-id="e5df5-160">Dans le **Version du Runtime Databricks** liste déroulante, sélectionnez **4.3** ou supérieur.</span><span class="sxs-lookup"><span data-stu-id="e5df5-160">In the **Databricks Runtime Version** dropdown, select **4.3** or greater.</span></span>
1. <span data-ttu-id="e5df5-161">Sous **Options avancées**, cliquez sur le **Spark** onglet. Entrez les paires nom-valeur suivantes dans le **Spark configuration** zone de texte :</span><span class="sxs-lookup"><span data-stu-id="e5df5-161">Under **Advanced Options**, click on the **Spark** tab. Enter the following name-value pairs in the **Spark Config** text box:</span></span>

    ```
    spark.extraListeners com.databricks.backend.daemon.driver.DBCEventLoggingListener,org.apache.spark.listeners.UnifiedSparkListener
    spark.unifiedListener.sink org.apache.spark.listeners.sink.loganalytics.LogAnalyticsListenerSink
    spark.unifiedListener.logBlockUpdates false
    ```

1. <span data-ttu-id="e5df5-162">Bien que toujours sous le **Spark** , entrez les valeurs suivantes dans le **Variables d’environnement** zone de texte :</span><span class="sxs-lookup"><span data-stu-id="e5df5-162">While still under the **Spark** tab, enter the following values in the **Environment Variables** text box:</span></span>

    ```
    LOG_ANALYTICS_WORKSPACE_ID=[your Azure Log Analytics workspace ID]
    LOG_ANALYTICS_WORKSPACE_KEY=[your Azure Log Analytics workspace key]
    ```

    ![Capture d’écran de l’interface utilisateur de Databricks](./_images/create-cluster1.png)

1. <span data-ttu-id="e5df5-164">Bien que toujours sous le **Options avancées** section, cliquez sur le **Scripts Init** onglet.</span><span class="sxs-lookup"><span data-stu-id="e5df5-164">While still under the **Advanced Options** section, click on the **Init Scripts** tab.</span></span>
1. <span data-ttu-id="e5df5-165">Dans le **Destination** liste déroulante, sélectionnez **DBFS**.</span><span class="sxs-lookup"><span data-stu-id="e5df5-165">In the **Destination** dropdown, select **DBFS**.</span></span>
1. <span data-ttu-id="e5df5-166">Sous **chemin d’accès du Script Init**, entrez `dbfs:/databricks/monitoring-staging/listeners.sh`.</span><span class="sxs-lookup"><span data-stu-id="e5df5-166">Under **Init Script Path**, enter `dbfs:/databricks/monitoring-staging/listeners.sh`.</span></span>
1. <span data-ttu-id="e5df5-167">Cliquez sur **Add**.</span><span class="sxs-lookup"><span data-stu-id="e5df5-167">Click **Add**.</span></span>

    ![Capture d’écran de l’interface utilisateur de Databricks](./_images/create-cluster2.png)

1. <span data-ttu-id="e5df5-169">Cliquez sur **création d’un Cluster** pour créer le cluster.</span><span class="sxs-lookup"><span data-stu-id="e5df5-169">Click **Create Cluster** to create the cluster.</span></span>

## <a name="view-metrics"></a><span data-ttu-id="e5df5-170">Afficher les mesures</span><span class="sxs-lookup"><span data-stu-id="e5df5-170">View metrics</span></span>

<span data-ttu-id="e5df5-171">Après avoir effectué ces étapes, votre cluster Databricks transmet en continu des données métriques sur le cluster lui-même pour Azure Monitor.</span><span class="sxs-lookup"><span data-stu-id="e5df5-171">After you complete these steps, your Databricks cluster streams some metric data about the cluster itself to Azure Monitor.</span></span> <span data-ttu-id="e5df5-172">Données de journal soient disponibles dans votre espace de travail Analytique de journal Azure sous la « Active | Journaux personnalisés | Schéma de SparkMetric_CL ».</span><span class="sxs-lookup"><span data-stu-id="e5df5-172">This log data is available in your Azure Log Analytics workspace under the "Active | Custom Logs | SparkMetric_CL" schema.</span></span> <span data-ttu-id="e5df5-173">Pour plus d’informations sur les types de mesures qui sont consignés, consultez [métriques Core](https://metrics.dropwizard.io/4.0.0/manual/core.html) dans la documentation Dropwizard.</span><span class="sxs-lookup"><span data-stu-id="e5df5-173">For more information about the types of metrics that are logged, see [Metrics Core](https://metrics.dropwizard.io/4.0.0/manual/core.html) in the Dropwizard documentation.</span></span> <span data-ttu-id="e5df5-174">Le type de mesure, telles que la jauge, compteur ou d’histogramme, est écrit dans le champ de Type.</span><span class="sxs-lookup"><span data-stu-id="e5df5-174">The metric type, such as gauge, counter, or histogram, is written to the Type field.</span></span>

<span data-ttu-id="e5df5-175">En outre, la bibliothèque diffuse les événements de niveau d’Apache Spark et Spark Structured Streaming les métriques à partir de vos travaux à Azure Monitor.</span><span class="sxs-lookup"><span data-stu-id="e5df5-175">In addition, the library streams Apache Spark level events and Spark Structured Streaming metrics from your jobs to Azure Monitor.</span></span> <span data-ttu-id="e5df5-176">Vous n’avez pas besoin d’apporter des modifications à votre code d’application pour ces événements et les mesures.</span><span class="sxs-lookup"><span data-stu-id="e5df5-176">You don't need to make any changes to your application code for these events and metrics.</span></span> <span data-ttu-id="e5df5-177">Spark Structured Streaming de données de journal sont disponibles sous les « Active | Journaux personnalisés | Schéma de SparkListenerEvent_CL ».</span><span class="sxs-lookup"><span data-stu-id="e5df5-177">Spark Structured Streaming log data is available under the "Active | Custom Logs | SparkListenerEvent_CL" schema.</span></span>

![Capture d’écran d’un espace de travail Analytique de journal](./_images/workspace.png)

<span data-ttu-id="e5df5-179">Pour plus d’informations sur l’affichage des journaux, consultez [visualiser et analyser les données de journal dans Azure Monitor](/azure/azure-monitor/log-query/portals).</span><span class="sxs-lookup"><span data-stu-id="e5df5-179">For more information about viewing logs, see [Viewing and analyzing log data in Azure Monitor](/azure/azure-monitor/log-query/portals).</span></span>

## <a name="next-steps"></a><span data-ttu-id="e5df5-180">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="e5df5-180">Next steps</span></span>

<span data-ttu-id="e5df5-181">Cet article décrit comment configurer votre cluster pour envoyer des métriques Azure Monitor.</span><span class="sxs-lookup"><span data-stu-id="e5df5-181">This article described how to configure your cluster to send metrics to Azure Monitor.</span></span> <span data-ttu-id="e5df5-182">L’article suivant montre comment utiliser la bibliothèque d’analyse Azure Databricks pour envoyer des journaux et des métriques de l’application.</span><span class="sxs-lookup"><span data-stu-id="e5df5-182">The next article shows how to use the Azure Databricks Monitoring Library to send application metrics and logs.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="e5df5-183">Envoyer les journaux des applications à Azure Monitor</span><span class="sxs-lookup"><span data-stu-id="e5df5-183">Send application logs to Azure Monitor</span></span>](./application-logs.md)
