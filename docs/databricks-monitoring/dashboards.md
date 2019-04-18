---
title: Utiliser des tableaux de bord pour visualiser les métriques Azure Databricks
description: Comment déployer un tableau de bord Grafana pour analyser les performances dans Azure Databricks
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: a84203a9188848e6363a80ac455332e8f6a73cda
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640309"
---
# <a name="use-dashboards-to-visualize-azure-databricks-metrics"></a><span data-ttu-id="6e3f3-103">Utiliser des tableaux de bord pour visualiser les métriques Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="6e3f3-103">Use dashboards to visualize Azure Databricks metrics</span></span>

<span data-ttu-id="6e3f3-104">Cet article explique comment configurer un tableau de bord Grafana pour surveiller les travaux Azure Databricks pour les problèmes de performances.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-104">This article shows how to set up a Grafana dashboard to monitor Azure Databricks jobs for performance issues.</span></span>

<span data-ttu-id="6e3f3-105">[Azure Databricks](/azure/azure-databricks/) est rapide, puissante et de collaboration [Apache Spark](https://spark.apache.org/)– en fonction du service d’analytique qui facilite la rapidement développer et déployer l’analytique de big data et les solutions d’intelligence artificielle (IA).</span><span class="sxs-lookup"><span data-stu-id="6e3f3-105">[Azure Databricks](/azure/azure-databricks/) is a fast, powerful, and collaborative [Apache Spark](https://spark.apache.org/)–based analytics service that makes it easy to rapidly develop and deploy big data analytics and artificial intelligence (AI) solutions.</span></span> <span data-ttu-id="6e3f3-106">La surveillance est un composant essentiel de l’utilisation de charges de travail Azure Databricks en production.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-106">Monitoring is a critical component of operating Azure Databricks workloads in production.</span></span> <span data-ttu-id="6e3f3-107">La première étape consiste à collecter des mesures dans un espace de travail pour l’analyse.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-107">The first step is to gather metrics into a workspace for analysis.</span></span> <span data-ttu-id="6e3f3-108">Dans Azure, la meilleure solution pour la gestion des données de journal est [Azure Monitor](/azure/azure-monitor/).</span><span class="sxs-lookup"><span data-stu-id="6e3f3-108">In Azure, the best solution for managing log data is [Azure Monitor](/azure/azure-monitor/).</span></span> <span data-ttu-id="6e3f3-109">Databricks Azure ne prend pas en charge envoi de données de journal à Azure monitor, mais un [bibliothèque pour cette fonctionnalité](https://github.com/mspnp/spark-monitoring) est disponible dans [Github](https://github.com).</span><span class="sxs-lookup"><span data-stu-id="6e3f3-109">Azure Databricks does not natively support sending log data to Azure monitor, but a [library for this functionality](https://github.com/mspnp/spark-monitoring) is available in [Github](https://github.com).</span></span>

<span data-ttu-id="6e3f3-110">Cette bibliothèque Active la journalisation des métriques de service Azure Databricks aussi structure Apache Spark streaming des métriques d’événement de requête.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-110">This library enables logging of Azure Databricks service metrics as well as Apache Spark structure streaming query event metrics.</span></span> <span data-ttu-id="6e3f3-111">Une fois que vous avez correctement déployé cette bibliothèque dans un cluster Azure Databricks, vous pouvez plus déployer un ensemble de [Grafana](https://granfana.com) des tableaux de bord que vous pouvez déployer dans le cadre de votre environnement de production.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-111">Once you've successfully deployed this library to an Azure Databricks cluster, you can further deploy a set of [Grafana](https://granfana.com) dashboards that you can deploy as part of your production environment.</span></span>

![Capture d’écran du tableau de bord](./_images/dashboard-screenshot.png)

## <a name="prerequisites"></a><span data-ttu-id="6e3f3-113">Conditions préalables</span><span class="sxs-lookup"><span data-stu-id="6e3f3-113">Prerequisites</span></span>

<span data-ttu-id="6e3f3-114">Clone le [référentiel Github](https://github.com/mspnp/spark-monitoring) et [suivez les instructions de déploiement](./configure-cluster.md) pour générer et configurer la journalisation d’Azure Monitor pour la bibliothèque Azure Databricks envoyer les journaux à votre espace de travail Analytique des journaux Azure.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-114">Clone the [Github repository](https://github.com/mspnp/spark-monitoring) and [follow the deployment instructions](./configure-cluster.md) to build and configure the Azure Monitor logging for Azure Databricks library to send logs to your Azure Log Analytics workspace.</span></span>

## <a name="deploy-the-azure-log-analytics-workspace"></a><span data-ttu-id="6e3f3-115">Déployer l’espace de travail Analytique des journaux Azure</span><span class="sxs-lookup"><span data-stu-id="6e3f3-115">Deploy the Azure Log Analytics workspace</span></span>

<span data-ttu-id="6e3f3-116">Pour déployer l’espace de travail Analytique des journaux Azure, procédez comme suit :</span><span class="sxs-lookup"><span data-stu-id="6e3f3-116">To deploy the Azure Log Analytics workspace, follow these steps:</span></span>

1. <span data-ttu-id="6e3f3-117">Accédez à la `/perftools/deployment/loganalytics` directory.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-117">Navigate to the `/perftools/deployment/loganalytics` directory.</span></span>
1. <span data-ttu-id="6e3f3-118">Déployer le **logAnalyticsDeploy.json** modèle Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-118">Deploy the **logAnalyticsDeploy.json** Azure Resource Manager template.</span></span> <span data-ttu-id="6e3f3-119">Pour plus d’informations sur le déploiement de modèles Resource Manager, consultez [déployer des ressources avec des modèles Resource Manager et Azure CLI][rm-cli].</span><span class="sxs-lookup"><span data-stu-id="6e3f3-119">For more information about deploying Resource Manager templates, see [Deploy resources with Resource Manager templates and Azure CLI][rm-cli].</span></span> <span data-ttu-id="6e3f3-120">Le modèle possède les paramètres suivants :</span><span class="sxs-lookup"><span data-stu-id="6e3f3-120">The template has the following parameters:</span></span>

    * <span data-ttu-id="6e3f3-121">**location** : La région où l’espace de travail Analytique de journal et les tableaux de bord est déployés.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-121">**location**: The region where the Log Analytics workspace and dashboards are deployed.</span></span>
    * <span data-ttu-id="6e3f3-122">**serviceTier**: Niveau tarifaire de l’espace de travail Rhe.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-122">**serviceTier**: Rhe workspace pricing tier.</span></span> <span data-ttu-id="6e3f3-123">Consultez [ici] [ sku] pour obtenir la liste des valeurs valides.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-123">See [here][sku] for a list of valid values.</span></span>
    * <span data-ttu-id="6e3f3-124">**dataRetention** (facultatif) : Le nombre de jours de données de journal est conservé dans l’espace de travail Analytique de journal.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-124">**dataRetention** (optional): The number of days log data is retained in the Log Analytics workspace.</span></span> <span data-ttu-id="6e3f3-125">La valeur par défaut est de 30 jours.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-125">The default value is 30 days.</span></span> <span data-ttu-id="6e3f3-126">Si le niveau tarifaire est `Free`, la rétention des données doit être de 7 jours.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-126">If the pricing tier is `Free`, the data retention must be 7 days.</span></span>
    * <span data-ttu-id="6e3f3-127">**WorkspaceName** (facultatif) : Nom de l’espace de travail.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-127">**workspaceName** (optional): A name for the workspace.</span></span> <span data-ttu-id="6e3f3-128">Si non spécifié, le modèle génère un nom.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-128">If not specified, the template generates a name.</span></span>

    ```bash
    az group deployment create --resource-group <resource-group-name> --template-file logAnalyticsDeploy.json --parameters location='East US' serviceTier='Standalone'
    ```

<span data-ttu-id="6e3f3-129">Ce modèle crée l’espace de travail et crée également un ensemble de requêtes prédéfinies qui sont utilisés par le tableau de bord.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-129">This template creates the workspace and also creates a set of predefined queries that are used by dashboard.</span></span>

## <a name="deploy-grafana-in-a-virtual-machine"></a><span data-ttu-id="6e3f3-130">Déployer Grafana dans une machine virtuelle</span><span class="sxs-lookup"><span data-stu-id="6e3f3-130">Deploy Grafana in a virtual machine</span></span>

<span data-ttu-id="6e3f3-131">Grafana est un projet open source que vous pouvez déployer pour visualiser les métriques de série de temps stockées dans votre espace de travail Analytique des journaux Azure à l’aide du plug-in Grafana pour Azure Monitor.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-131">Grafana is an open source project you can deploy to visualize the time series metrics stored in your Azure Log Analytics workspace using the Grafana plugin for Azure Monitor.</span></span> <span data-ttu-id="6e3f3-132">Grafana s’exécute sur une machine virtuelle (VM) et nécessite un compte de stockage, de réseau virtuel et d’autres ressources.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-132">Grafana executes on a virtual machine (VM) and requires a storage account, virtual network, and other resources.</span></span> <span data-ttu-id="6e3f3-133">Pour déployer une machine virtuelle avec le bitnami certifié Grafana image et les ressources associées, procédez comme suit :</span><span class="sxs-lookup"><span data-stu-id="6e3f3-133">To deploy a virtual machine with the bitnami certified Grafana image and associated resources, follow these steps:</span></span>

1. <span data-ttu-id="6e3f3-134">Utilisez l’interface CLI pour accepter les termes d’image de place de marché Azure de Grafana.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-134">Use the Azure CLI to accept the Azure Marketplace image terms for Grafana.</span></span>

    ```bash
    az vm image accept-terms --publisher bitnami --offer grafana --plan default
    ```

1. <span data-ttu-id="6e3f3-135">Accédez à la `/spark-monitoring/perftools/deployment/grafana` répertoire dans votre copie locale du référentiel GitHub.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-135">Navigate to the `/spark-monitoring/perftools/deployment/grafana` directory in your local copy of the GitHub repo.</span></span>
1. <span data-ttu-id="6e3f3-136">Déployer le **grafanaDeploy.json** modèle Resource Manager comme suit :</span><span class="sxs-lookup"><span data-stu-id="6e3f3-136">Deploy the **grafanaDeploy.json** Resource Manager template as follows:</span></span>

    ```bash
    export DATA_SOURCE="https://raw.githubusercontent.com/mspnp/spark-monitoring/master/perftools/deployment/grafana/AzureDataSource.sh"
    az group deployment create \
        --resource-group <resource-group-name> \
        --template-file grafanaDeploy.json \
        --parameters adminPass='<vm password>' dataSource=$DATA_SOURCE
    ```

<span data-ttu-id="6e3f3-137">Une fois le déploiement terminé, l’image bitnami de Grafana est installé sur l’ordinateur virtuel.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-137">Once the deployment is complete, the bitnami image of Grafana is installed on the virtual machine.</span></span>

## <a name="update-the-grafana-password"></a><span data-ttu-id="6e3f3-138">Mettre à jour le mot de passe de Grafana</span><span class="sxs-lookup"><span data-stu-id="6e3f3-138">Update the Grafana password</span></span>

<span data-ttu-id="6e3f3-139">Dans le cadre du processus d’installation, le script d’installation Grafana génère un mot de passe temporaire pour le **administrateur** utilisateur.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-139">As part of the setup process, the Grafana installation script outputs a temporary password for the **admin** user.</span></span> <span data-ttu-id="6e3f3-140">Vous avez besoin de ce mot de passe temporaire pour vous connecter.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-140">You need this temporary password to sign in.</span></span> <span data-ttu-id="6e3f3-141">Pour obtenir le mot de passe temporaire, procédez comme suit :</span><span class="sxs-lookup"><span data-stu-id="6e3f3-141">To obtain the temporary password, follow these steps:</span></span>  

1. <span data-ttu-id="6e3f3-142">Connectez-vous au portail Azure.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-142">Log in to the Azure portal.</span></span>  
1. <span data-ttu-id="6e3f3-143">Sélectionnez le groupe de ressources dans lequel les ressources ont été déployées.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-143">Select the resource group where the resources were deployed.</span></span>
1. <span data-ttu-id="6e3f3-144">Sélectionnez la machine virtuelle où Grafana a été installé.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-144">Select the VM where Grafana was installed.</span></span> <span data-ttu-id="6e3f3-145">Si vous avez utilisé le nom de paramètre par défaut dans le modèle de déploiement, le nom de la machine virtuelle est précédé **sparkmonitoring-vm-grafana**.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-145">If you used the default parameter name in the deployment template, the VM name is prefaced with **sparkmonitoring-vm-grafana**.</span></span>
1. <span data-ttu-id="6e3f3-146">Dans le **Support + dépannage** , cliquez sur **diagnostics de démarrage** pour ouvrir la page de diagnostics de démarrage.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-146">In the **Support + troubleshooting** section, click **Boot diagnostics** to open the boot diagnostics page.</span></span>
1. <span data-ttu-id="6e3f3-147">Cliquez sur **journal série** dans la page de diagnostics de démarrage.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-147">Click **Serial log** on the boot diagnostics page.</span></span>
1. <span data-ttu-id="6e3f3-148">Recherchez la chaîne suivante : « Configuration de mot de passe applications Bitnami. »</span><span class="sxs-lookup"><span data-stu-id="6e3f3-148">Search for the following string: "Setting Bitnami application password to".</span></span>
1. <span data-ttu-id="6e3f3-149">Copiez le mot de passe dans un emplacement sûr.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-149">Copy the password to a safe location.</span></span>

<span data-ttu-id="6e3f3-150">Ensuite, modifiez le mot de passe administrateur Grafana en suivant ces étapes :</span><span class="sxs-lookup"><span data-stu-id="6e3f3-150">Next, change the Grafana administrator password by following these steps:</span></span>

1. <span data-ttu-id="6e3f3-151">Dans le portail Azure, sélectionnez la machine virtuelle, puis cliquez sur **vue d’ensemble**.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-151">In the Azure portal, select the VM and click **Overview**.</span></span>
1. <span data-ttu-id="6e3f3-152">Copiez l’adresse IP publique.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-152">Copy the public IP address.</span></span>
1. <span data-ttu-id="6e3f3-153">Ouvrez un navigateur web et accédez à l’URL suivante : `http://<IP address>:3000`.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-153">Open a web browser and navigate to the following URL: `http://<IP address>:3000`.</span></span>
1. <span data-ttu-id="6e3f3-154">À l’écran de connexion Grafana, entrez **administrateur** le nom d’utilisateur et utilisez le mot de passe Grafana lors des étapes précédentes.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-154">At the Grafana log in screen, enter **admin** for the user name, and use the Grafana password from the previous steps.</span></span>
1. <span data-ttu-id="6e3f3-155">Une fois connecté, sélectionnez **Configuration** (l’icône d’engrenage).</span><span class="sxs-lookup"><span data-stu-id="6e3f3-155">Once logged in, select **Configuration** (the gear icon).</span></span>
1. <span data-ttu-id="6e3f3-156">Sélectionnez **administrateur du serveur**.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-156">Select **Server Admin**.</span></span>
1. <span data-ttu-id="6e3f3-157">Sur le **utilisateurs** onglet, sélectionnez le **administrateur** connexion.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-157">On the **Users** tab, select the **admin** login.</span></span>
1. <span data-ttu-id="6e3f3-158">Mettre à jour le mot de passe.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-158">Update the password.</span></span>

## <a name="create-an-azure-monitor-data-source"></a><span data-ttu-id="6e3f3-159">Créer une source de données Azure Monitor</span><span class="sxs-lookup"><span data-stu-id="6e3f3-159">Create an Azure Monitor data source</span></span>

1. <span data-ttu-id="6e3f3-160">Créer un service principal qui permet de Grafana gérer l’accès à votre espace de travail Analytique de journal.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-160">Create a service principal that allows Grafana to manage access to your Log Analytics workspace.</span></span> <span data-ttu-id="6e3f3-161">Pour plus d’informations, consultez [créer un principal du service avec Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)</span><span class="sxs-lookup"><span data-stu-id="6e3f3-161">For more information, see [Create an Azure service principal with Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)</span></span>

    ```bash
    az ad sp create-for-rbac --name http://<service principal name> --role "Log Analytics Reader"
    ```

1. <span data-ttu-id="6e3f3-162">Notez les valeurs de l’appId, password et tenant dans la sortie de cette commande :</span><span class="sxs-lookup"><span data-stu-id="6e3f3-162">Note the values for appId, password, and tenant in the output from this command:</span></span>

    ```json
    {
        "appId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "displayName": "azure-cli-2019-03-27-00-33-39",
        "name": "http://<service principal name>",
        "password": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "tenant": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
    ```

1. <span data-ttu-id="6e3f3-163">Connectez-vous à Grafana comme décrit précédemment.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-163">Log into Grafana as described earlier.</span></span> <span data-ttu-id="6e3f3-164">Sélectionnez **Configuration** (l’icône d’engrenage), puis **des Sources de données**.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-164">Select **Configuration** (the gear icon) and then **Data Sources**.</span></span>
1. <span data-ttu-id="6e3f3-165">Dans le **des Sources de données** , cliquez sur **ajouter une source de données**.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-165">In the **Data Sources** tab, click **Add data source**.</span></span>
1. <span data-ttu-id="6e3f3-166">Sélectionnez **Azure Monitor** comme type de source de données.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-166">Select **Azure Monitor** as the data source type.</span></span>
1. <span data-ttu-id="6e3f3-167">Dans le **paramètres** section, entrez un nom pour la source de données dans le **nom** zone de texte.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-167">In the **Settings** section, enter a name for the data source in the **Name** textbox.</span></span>
1. <span data-ttu-id="6e3f3-168">Dans le **détails de l’API Azure Monitor** section, entrez les informations suivantes :</span><span class="sxs-lookup"><span data-stu-id="6e3f3-168">In the **Azure Monitor API Details** section, enter the following information:</span></span>

    * <span data-ttu-id="6e3f3-169">ID d'abonnement : ID de votre abonnement Azure.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-169">Subscription Id: Your Azure subscription ID.</span></span>
    * <span data-ttu-id="6e3f3-170">Id de locataire : L’ID de client précédemment.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-170">Tenant Id: The tenant ID from earlier.</span></span>
    * <span data-ttu-id="6e3f3-171">Id de client : La valeur de « appId » indiquée précédemment.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-171">Client Id: The value of "appId" from earlier.</span></span>
    * <span data-ttu-id="6e3f3-172">Clé secrète client : La valeur de « password » indiquée précédemment.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-172">Client Secret: The value of "password" from earlier.</span></span>

1. <span data-ttu-id="6e3f3-173">Dans le **détails de l’API Azure Log Analytique** section, vérifiez le **même détails comme les API Azure Monitor** case à cocher.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-173">In the **Azure Log Analytics API Details** section, check the **Same Details as Azure Monitor API** checkbox.</span></span>
1. <span data-ttu-id="6e3f3-174">Cliquez sur **enregistrer et tester**.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-174">Click **Save & Test**.</span></span> <span data-ttu-id="6e3f3-175">Si la source de données Analytique de journal est configurée correctement, un message de réussite s’affiche.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-175">If the Log Analytics data source is correctly configured, a success message is displayed.</span></span>

## <a name="create-the-dashboard"></a><span data-ttu-id="6e3f3-176">Créer le tableau de bord</span><span class="sxs-lookup"><span data-stu-id="6e3f3-176">Create the dashboard</span></span>

<span data-ttu-id="6e3f3-177">Créer des tableaux de bord dans Grafana en suivant ces étapes :</span><span class="sxs-lookup"><span data-stu-id="6e3f3-177">Create the dashboards in Grafana by following these steps:</span></span>

1. <span data-ttu-id="6e3f3-178">Accédez à la `/perftools/dashboards/grafana` répertoire dans votre copie locale du référentiel GitHub.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-178">Navigate to the `/perftools/dashboards/grafana` directory in your local copy of the GitHub repo.</span></span>
1. <span data-ttu-id="6e3f3-179">Exécutez le script qui suit :</span><span class="sxs-lookup"><span data-stu-id="6e3f3-179">Run the following script:</span></span>

    ```bash
    export WORKSPACE=<your Azure Log Analytics workspace ID>
    export LOGTYPE=SparkListenerEvent_CL

    sh DashGen.sh
    ```

    <span data-ttu-id="6e3f3-180">La sortie du script est un fichier nommé **SparkMonitoringDash.json**.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-180">The output from the script is a file named **SparkMonitoringDash.json**.</span></span>

1. <span data-ttu-id="6e3f3-181">Revenez au tableau de bord Grafana et sélectionnez **créer** (l’icône plus (+)).</span><span class="sxs-lookup"><span data-stu-id="6e3f3-181">Return to the Grafana dashboard and select **Create** (the plus icon).</span></span>
1. <span data-ttu-id="6e3f3-182">Sélectionnez **Importer**.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-182">Select **Import**.</span></span>
1. <span data-ttu-id="6e3f3-183">Cliquez sur **télécharger le fichier .json**.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-183">Click **Upload .json File**.</span></span>
1. <span data-ttu-id="6e3f3-184">Sélectionnez le **SparkMonitoringDash.json** fichier créé à l’étape 2.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-184">Select the **SparkMonitoringDash.json** file created in step 2.</span></span>
1. <span data-ttu-id="6e3f3-185">Dans le **Options** section sous **ALA**, sélectionnez la source de données Azure Monitor créée précédemment.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-185">In the **Options** section, under **ALA**, select the Azure Monitor data source created earlier.</span></span>
1. <span data-ttu-id="6e3f3-186">Cliquez sur **Importer**.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-186">Click **Import**.</span></span>

## <a name="visualizations-in-the-dashboards"></a><span data-ttu-id="6e3f3-187">Visualisations dans les tableaux de bord</span><span class="sxs-lookup"><span data-stu-id="6e3f3-187">Visualizations in the dashboards</span></span>

<span data-ttu-id="6e3f3-188">Les tableaux de bord Azure Log Analytique et de Grafana comprennent un ensemble de visualisations de série chronologique.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-188">Both the Azure Log Analytics and Grafana dashboards include a set of time-series visualizations.</span></span> <span data-ttu-id="6e3f3-189">Chaque graphique est tracé de séries chronologiques des données métriques associées à un Apache Spark [travail](https://spark.apache.org/docs/latest/job-scheduling.html), étapes de travail et des tâches qui composent chaque étape.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-189">Each graph is time-series plot of metric data related to an Apache Spark [job](https://spark.apache.org/docs/latest/job-scheduling.html), stages of the job, and tasks that make up each stage.</span></span>

<span data-ttu-id="6e3f3-190">Les visualisations sont les suivantes :</span><span class="sxs-lookup"><span data-stu-id="6e3f3-190">The visualizations are as follows:</span></span>

### <a name="job-latency"></a><span data-ttu-id="6e3f3-191">Latence du travail</span><span class="sxs-lookup"><span data-stu-id="6e3f3-191">Job latency</span></span>

<span data-ttu-id="6e3f3-192">Cette visualisation présente une latence d’exécution d’une tâche, qui est une vue grossière sur les performances globales d’un travail.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-192">This visualization shows execution latency for a job, which is a coarse view on the overall performance of a job.</span></span> <span data-ttu-id="6e3f3-193">Affiche la durée d’exécution du travail à partir du début jusqu'à la fin.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-193">Displays the job execution duration from start to completion.</span></span> <span data-ttu-id="6e3f3-194">Remarque heure de début de la tâche n’est pas identique à l’heure d’envoi de travail.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-194">Note that the job start time is not the same as the job submission time.</span></span> <span data-ttu-id="6e3f3-195">Latence est représentée en tant que centiles (10 %, 30 %, 50 %, 90 %) de l’exécution du travail indexée par ID de cluster et les ID d’application.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-195">Latency is represented as percentiles (10%, 30%, 50%, 90%) of job execution indexed by cluster ID and application ID.</span></span>

### <a name="stage-latency"></a><span data-ttu-id="6e3f3-196">Latence de la scène</span><span class="sxs-lookup"><span data-stu-id="6e3f3-196">Stage latency</span></span>

<span data-ttu-id="6e3f3-197">La visualisation montre la latence de chaque étape par étape individuel par cluster et par application.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-197">The visualization shows the latency of each stage per cluster, per application, and per individual stage.</span></span> <span data-ttu-id="6e3f3-198">Cette visualisation est utile pour identifier une phase spécifique qui s’exécute lentement.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-198">This visualization is useful for identifying a particular stage that is running slowly.</span></span>

### <a name="task-latency"></a><span data-ttu-id="6e3f3-199">Latence de la tâche</span><span class="sxs-lookup"><span data-stu-id="6e3f3-199">Task latency</span></span>

<span data-ttu-id="6e3f3-200">Cette visualisation montre la latence d’exécution de tâches.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-200">This visualization shows task execution latency.</span></span> <span data-ttu-id="6e3f3-201">Latence est représentée comme un centile de l’exécution de tâches par cluster, le nom de la scène et application.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-201">Latency is represented as a percentile of task execution per cluster, stage name, and application.</span></span>

### <a name="sum-task-execution-per-host"></a><span data-ttu-id="6e3f3-202">Exécution de la tâche somme par hôte</span><span class="sxs-lookup"><span data-stu-id="6e3f3-202">Sum Task Execution per host</span></span>

<span data-ttu-id="6e3f3-203">Cette visualisation affiche la somme de la latence de l’exécution de tâches par hôte en cours d’exécution sur un cluster.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-203">This visualization shows the sum of task execution latency per host running on a cluster.</span></span> <span data-ttu-id="6e3f3-204">Affichage de latence d’exécution de tâches par hôte identifie hôtes qui ont beaucoup globale tâche une latence plus élevée que les autres hôtes.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-204">Viewing task execution latency per host identifies hosts that have much higher overall task latency than other hosts.</span></span> <span data-ttu-id="6e3f3-205">Cela peut signifier que les tâches ont été inefficace ou inégalement distribuées aux ordinateurs hôtes.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-205">This may mean that tasks have been inefficiently or unevenly distributed to hosts.</span></span>

### <a name="task-metrics"></a><span data-ttu-id="6e3f3-206">Mesures de tâches</span><span class="sxs-lookup"><span data-stu-id="6e3f3-206">Task metrics</span></span>

<span data-ttu-id="6e3f3-207">Cette visualisation présente un ensemble de métriques d’exécution pour l’exécution d’une tâche donnée.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-207">This visualization shows a set of the execution metrics for a given task's execution.</span></span> <span data-ttu-id="6e3f3-208">Ces mesures comprennent la taille et la durée d’un mélange de données et durée des opérations de sérialisation et la désérialisation.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-208">These metrics include the size and duration of a data shuffle, duration of serialization and deserialization operations, and others.</span></span> <span data-ttu-id="6e3f3-209">Pour l’ensemble des mesures, afficher la requête Analytique de journal pour le panneau.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-209">For the full set of metrics, view the Log Analytics query for the panel.</span></span> <span data-ttu-id="6e3f3-210">Cette visualisation est utile pour comprendre les opérations qui constituent une tâche et l’identification la consommation des ressources de chaque opération.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-210">This visualization is useful for understanding the operations that make up a task and identifying resource consumption of each operation.</span></span> <span data-ttu-id="6e3f3-211">Les pointes du graphique représentent des opérations coûteuses qui doivent être examinées.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-211">Spikes in the graph represent costly operations that should be investigated.</span></span>

### <a name="cluster-throughput"></a><span data-ttu-id="6e3f3-212">Débit de cluster</span><span class="sxs-lookup"><span data-stu-id="6e3f3-212">Cluster throughput</span></span>

<span data-ttu-id="6e3f3-213">Cette visualisation est une vue de haut niveau d’éléments de travail indexées par le cluster et des applications pour représenter la quantité de travail effectué par le cluster et des applications.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-213">This visualization is a high level view of work items indexed by cluster and application to represent the amount of work done per cluster and application.</span></span> <span data-ttu-id="6e3f3-214">Il indique le nombre de travaux, tâches et étapes réussies par cluster, l’application et étape par incréments d’une minute.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-214">It shows the number of jobs, tasks, and stages completed per cluster, application, and stage in one minute increments.</span></span> 

### <a name="streaming-throughputlatency"></a><span data-ttu-id="6e3f3-215">Diffusion en continu de débit et de latence</span><span class="sxs-lookup"><span data-stu-id="6e3f3-215">Streaming Throughput/Latency</span></span>

<span data-ttu-id="6e3f3-216">Cette visualisation concerne les mesures associées à une requête de diffusion en continu structurée.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-216">This visualization is related to the metrics associated with a structured streaming query.</span></span> <span data-ttu-id="6e3f3-217">Les graphiques présente le nombre de lignes d’entrée par seconde et le nombre de lignes traitées par seconde.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-217">The graphs shows the number of input rows per second and the number of rows processed per second.</span></span> <span data-ttu-id="6e3f3-218">Les mesures de diffusion en continu sont également représentés par l’application.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-218">The streaming metrics are also represented per application.</span></span> <span data-ttu-id="6e3f3-219">Ces mesures sont envoyés lorsque l’événement OnQueryProgress est généré comme traitement de la requête de diffusion en continu structurée et la visualisation représente latence en tant que la quantité de temps de diffusion en continu, en millisecondes, nécessaire pour exécuter un traitement de requêtes.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-219">These metrics are sent when the OnQueryProgress event is generated as the structured streaming query is processed and the visualization represents streaming latency as the amount of time, in milliseconds, taken to execute a query batch.</span></span>

### <a name="resource-consumption-per-executor"></a><span data-ttu-id="6e3f3-220">Consommation des ressources par exécuteur</span><span class="sxs-lookup"><span data-stu-id="6e3f3-220">Resource consumption per executor</span></span>

<span data-ttu-id="6e3f3-221">Voici un ensemble de visualisations de tableau de bord afficher le type de ressource particulier et la façon dont elle est consommée par exécuteur sur chaque cluster.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-221">Next is a set of visualizations for the dashboard show the particular type of resource and how it is consumed per executor on each cluster.</span></span> <span data-ttu-id="6e3f3-222">Ces visualisations pour identifier les valeurs hors norme la consommation de ressources par exécuteur.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-222">These visualizations help identify outliers in resource consumption per executor.</span></span> <span data-ttu-id="6e3f3-223">Par exemple, si la répartition du travail pour un exécuteur particulier est déréglée, la consommation de ressources va être élevée par rapport aux autres exécuteurs en cours d’exécution sur le cluster.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-223">For example, if the work allocation for a particular executor is skewed, resource consumption will be elevated in relation to other executors running on the cluster.</span></span> <span data-ttu-id="6e3f3-224">Cela peut être identifiée par les pics de la consommation des ressources pour un exécuteur.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-224">This can be identified by spikes in the resource consumption for an executor.</span></span>

### <a name="executor-compute-time-metrics"></a><span data-ttu-id="6e3f3-225">Mesures de temps de calcul exécuteur</span><span class="sxs-lookup"><span data-stu-id="6e3f3-225">Executor compute time metrics</span></span>

<span data-ttu-id="6e3f3-226">Voici un ensemble de visualisations du tableau de bord qui afficher le rapport de l’exécuteur sérialiser le temps, désérialiser le temps, le temps processeur et l’heure de machine virtuelle Java pour les temps de calcul exécuteur globale.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-226">Next is a set of visualizations for the dashboard that show the ratio of executor serialize time, deserialize time, CPU time, and Java virtual machine time to overall executor compute time.</span></span> <span data-ttu-id="6e3f3-227">Cet exemple montre visuellement la quantité chacune de ces quatre mesures contribuent à l’exécuteur globale de traitement.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-227">This demonstrates visually how much each of these four metrics are contributing to overall executor processing.</span></span>

### <a name="shuffle-metrics"></a><span data-ttu-id="6e3f3-228">Métriques de lecture aléatoire</span><span class="sxs-lookup"><span data-stu-id="6e3f3-228">Shuffle metrics</span></span>

<span data-ttu-id="6e3f3-229">L’ensemble final d’afficher des visualisations mesures associées à une requête de diffusion en continu structurée entre tous les exécuteurs de lecture aléatoire les données.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-229">The final set of visualizations show the data shuffle metrics associated with a structured streaming query across all executors.</span></span> <span data-ttu-id="6e3f3-230">Ceux-ci incluent la lecture aléatoire octets de lecture, de lecture aléatoire d’octets écrits, de lecture aléatoire de mémoire et l’utilisation du disque dans les requêtes où le système de fichiers est utilisé.</span><span class="sxs-lookup"><span data-stu-id="6e3f3-230">These include shuffle bytes read, shuffle bytes written, shuffle memory, and disk usage in queries where the file system is used.</span></span>

## <a name="next-steps"></a><span data-ttu-id="6e3f3-231">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="6e3f3-231">Next steps</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="6e3f3-232">Résoudre les goulots d’étranglement de performances</span><span class="sxs-lookup"><span data-stu-id="6e3f3-232">Troubleshoot performance bottlenecks</span></span>](./performance-troubleshooting.md)

<!-- links -->

[rm-cli]: /azure/azure-resource-manager/resource-group-template-deploy-cli
[sku]: /azure/templates/Microsoft.OperationalInsights/2015-11-01-preview/workspaces#sku-object