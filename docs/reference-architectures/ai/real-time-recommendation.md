---
title: Générer une API de recommandation en temps réel sur Azure
description: Utilisez le machine learning pour automatiser des recommandations à l’aide d’Azure Databricks et d’Azure Data Science Virtual Machine (DSVM) pour entraîner un modèle sur Azure.
author: njray
ms.date: 12/12/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: azcat-ai
ms.openlocfilehash: 8fa15c82275f18af78d1f42ec1750ab9b28119fb
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54487109"
---
# <a name="build-a-real-time-recommendation-api-on-azure"></a><span data-ttu-id="d9e10-103">Générer une API de recommandation en temps réel sur Azure</span><span class="sxs-lookup"><span data-stu-id="d9e10-103">Build a real-time recommendation API on Azure</span></span>

<span data-ttu-id="d9e10-104">Cette architecture de référence montre comment entraîner un modèle de recommandation à l’aide d’Azure Databricks et le déployer en tant qu’API en utilisant Azure Cosmos DB, Azure Machine Learning et Azure Kubernetes Service (AKS).</span><span class="sxs-lookup"><span data-stu-id="d9e10-104">This reference architecture shows how to train a recommendation model using Azure Databricks and deploy it as an API by using Azure Cosmos DB, Azure Machine Learning, and Azure Kubernetes Service (AKS).</span></span> <span data-ttu-id="d9e10-105">Cette architecture peut être étendue à la plupart des scénarios de moteur de recommandation, notamment des recommandations pour des produits, films et actualités.</span><span class="sxs-lookup"><span data-stu-id="d9e10-105">This architecture can be generalized for most recommendation engine scenarios, including recommendations for products, movies, and news.</span></span>

<span data-ttu-id="d9e10-106">Une implémentation de référence pour cette architecture est disponible sur [GitHub](https://github.com/Microsoft/Recommenders/blob/master/notebooks/04_operationalize/als_movie_o16n.ipynb).</span><span class="sxs-lookup"><span data-stu-id="d9e10-106">A reference implementation for this architecture is available on [GitHub](https://github.com/Microsoft/Recommenders/blob/master/notebooks/04_operationalize/als_movie_o16n.ipynb).</span></span>

![Architecture d’un modèle Machine Learning pour l’entraînement de recommandations sur des films](./_images/recommenders-architecture.png)

<span data-ttu-id="d9e10-108">**Scénario** : Une société de multimédia souhaite fournir des recommandations sur des vidéos ou des films à ses utilisateurs.</span><span class="sxs-lookup"><span data-stu-id="d9e10-108">**Scenario**: A media organization wants to provide movie or video recommendations to its users.</span></span> <span data-ttu-id="d9e10-109">En proposant des recommandations personnalisées, la société répond à plusieurs objectifs de l’entreprise, dont des taux de clic plus élevés, un niveau d’engagement accru sur site et une plus grande satisfaction des utilisateurs.</span><span class="sxs-lookup"><span data-stu-id="d9e10-109">By providing personalized recommendations, the organization meets several business goals, including increased click-through rates, increased engagement on site, and higher user satisfaction.</span></span>

<span data-ttu-id="d9e10-110">Cette architecture de référence concerne l’entraînement et le déploiement d’une API de service de recommandation en temps réel qui peut fournir les 10 meilleures recommandations de films pour un utilisateur donné.</span><span class="sxs-lookup"><span data-stu-id="d9e10-110">This reference architecture is for training and deploying a real-time recommender service API that can provide the top 10 movie recommendations for a given user.</span></span>

<span data-ttu-id="d9e10-111">Le flux de données pour ce modèle de recommandation se présente comme suit :</span><span class="sxs-lookup"><span data-stu-id="d9e10-111">The data flow for this recommendation model is as follows:</span></span>

1. <span data-ttu-id="d9e10-112">Suivre les comportements des utilisateurs.</span><span class="sxs-lookup"><span data-stu-id="d9e10-112">Track user behaviors.</span></span> <span data-ttu-id="d9e10-113">Par exemple, un service back-end peut enregistrer quand un utilisateur évalue un film, ou clique sur un produit ou un article de presse.</span><span class="sxs-lookup"><span data-stu-id="d9e10-113">For example, a backend service might log when a user rates a movie or clicks a product or news article.</span></span>

2. <span data-ttu-id="d9e10-114">Charger les données dans Azure Databricks à partir d’une [source de données][data-source] disponible.</span><span class="sxs-lookup"><span data-stu-id="d9e10-114">Load the data into Azure Databricks from an available [data source][data-source].</span></span>

3. <span data-ttu-id="d9e10-115">Préparer les données et les diviser en jeux d’entraînement et de test pour entraîner le modèle.</span><span class="sxs-lookup"><span data-stu-id="d9e10-115">Prepare the data and split it into training and testing sets to train the model.</span></span> <span data-ttu-id="d9e10-116">([Ce guide][guide] décrit les options de division des données.)</span><span class="sxs-lookup"><span data-stu-id="d9e10-116">([This guide][guide] describes options for splitting data.)</span></span>

4. <span data-ttu-id="d9e10-117">Adapter le modèle [Spark Collaborative Filtering][als] aux données.</span><span class="sxs-lookup"><span data-stu-id="d9e10-117">Fit the [Spark Collaborative Filtering][als] model to the data.</span></span>

5. <span data-ttu-id="d9e10-118">Évaluer la qualité du modèle à l’aide de métriques d’évaluation et de classement.</span><span class="sxs-lookup"><span data-stu-id="d9e10-118">Evaluate the quality of the model using rating and ranking metrics.</span></span> <span data-ttu-id="d9e10-119">([Ce guide][eval-guide] fournit des détails sur les métriques selon lesquelles vous pouvez évaluer votre recommandation.)</span><span class="sxs-lookup"><span data-stu-id="d9e10-119">([This guide][eval-guide] provides details about the metrics you can evaluate your recommender on.)</span></span>

6. <span data-ttu-id="d9e10-120">Précalculer les 10 meilleures recommandations par utilisateur et les stocker en tant que cache dans Azure Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="d9e10-120">Precompute the top 10 recommendations per user and store as a cache in Azure Cosmos DB.</span></span>

7. <span data-ttu-id="d9e10-121">Déployer un service API sur AKS à l’aide des API Azure Machine Learning pour conteneuriser et déployer l’API.</span><span class="sxs-lookup"><span data-stu-id="d9e10-121">Deploy an API service to AKS using the Azure Machine Learning APIs to containerize and deploy the API.</span></span>

8. <span data-ttu-id="d9e10-122">Quand le service back-end reçoit une demande d’un utilisateur, appeler l’API de recommandation hébergée dans AKS pour obtenir les 10 meilleures recommandations et les afficher à l’intention de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="d9e10-122">When the backend service gets a request from a user, call the recommendations API hosted in AKS to get the top 10 recommendations and display them to the user.</span></span>

## <a name="architecture"></a><span data-ttu-id="d9e10-123">Architecture</span><span class="sxs-lookup"><span data-stu-id="d9e10-123">Architecture</span></span>

<span data-ttu-id="d9e10-124">Cette architecture est constituée des composants suivants :</span><span class="sxs-lookup"><span data-stu-id="d9e10-124">This architecture consists of the following components:</span></span>

<span data-ttu-id="d9e10-125">[Azure Databricks][databricks].</span><span class="sxs-lookup"><span data-stu-id="d9e10-125">[Azure Databricks][databricks].</span></span> <span data-ttu-id="d9e10-126">Databricks est un environnement de développement utilisé pour préparer des données d’entrée et entraîner le modèle de recommandation sur un cluster Spark.</span><span class="sxs-lookup"><span data-stu-id="d9e10-126">Databricks is a development environment used to prepare input data and train the recommender model on a Spark cluster.</span></span> <span data-ttu-id="d9e10-127">Azure Databricks offre également un espace de travail interactif afin d’exécuter des notebooks et de collaborer sur ceux-ci pour toute tâche de traitement de données ou de machine learning.</span><span class="sxs-lookup"><span data-stu-id="d9e10-127">Azure Databricks also provides an interactive workspace to run and collaborate on notebooks for any data processing or machine learning tasks.</span></span>

<span data-ttu-id="d9e10-128">[Azure Kubernetes Service][aks] (AKS).</span><span class="sxs-lookup"><span data-stu-id="d9e10-128">[Azure Kubernetes Service][aks] (AKS).</span></span> <span data-ttu-id="d9e10-129">AKS est utilisé pour déployer et rendre opérationnelle une API de service de modèle Machine Learning sur un cluster Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="d9e10-129">AKS is used to deploy and operationalize a machine learning model service API on a Kubernetes cluster.</span></span> <span data-ttu-id="d9e10-130">AKS héberge le modèle conteneurisé, en offrant une scalabilité qui répond à vos besoins en débit, la gestion des identités et des accès ainsi que la journalisation et la supervision de l’intégrité.</span><span class="sxs-lookup"><span data-stu-id="d9e10-130">AKS hosts the containerized model, providing scalability that meets your throughput requirements, identity and access management, and logging and health monitoring.</span></span>

<span data-ttu-id="d9e10-131">[Azure Cosmos DB][cosmosdb].</span><span class="sxs-lookup"><span data-stu-id="d9e10-131">[Azure Cosmos DB][cosmosdb].</span></span> <span data-ttu-id="d9e10-132">Cosmos DB est un service de base de données mondialement distribué qui permet de stocker les 10 premiers films recommandés pour chaque utilisateur.</span><span class="sxs-lookup"><span data-stu-id="d9e10-132">Cosmos DB is a globally distributed database service used to store the top 10 recommended movies for each user.</span></span> <span data-ttu-id="d9e10-133">Azure Cosmos DB convient bien pour ce scénario, car il fournit une faible latence (10 ms au 99e centile) pour lire les premiers articles recommandés pour un utilisateur donné.</span><span class="sxs-lookup"><span data-stu-id="d9e10-133">Azure Cosmos DB is well-suited for this scenario, because it provides low latency (10 ms at 99th percentile) to read the top recommended items for a given user.</span></span>

<span data-ttu-id="d9e10-134">[Service Azure Machine Learning][mls].</span><span class="sxs-lookup"><span data-stu-id="d9e10-134">[Azure Machine Learning Service][mls].</span></span> <span data-ttu-id="d9e10-135">Ce service est utilisé pour suivre et gérer des modèles Machine Learning, puis empaqueter et déployer ces modèles sur un environnement AKS scalable.</span><span class="sxs-lookup"><span data-stu-id="d9e10-135">This service is used to track and manage machine learning models, and then package and deploy these models to a scalable AKS environment.</span></span>

<span data-ttu-id="d9e10-136">[Microsoft Recommenders][github].</span><span class="sxs-lookup"><span data-stu-id="d9e10-136">[Microsoft Recommenders][github].</span></span> <span data-ttu-id="d9e10-137">Ce dépôt open source contient le code des utilitaires et des exemples permettant aux utilisateurs de prendre en main la génération, l’évaluation et la mise en place d’un système de recommandation.</span><span class="sxs-lookup"><span data-stu-id="d9e10-137">This open-source repository contains utility code and samples to help users get started in building, evaluating, and operationalizing a recommender system.</span></span>

## <a name="performance-considerations"></a><span data-ttu-id="d9e10-138">Considérations relatives aux performances</span><span class="sxs-lookup"><span data-stu-id="d9e10-138">Performance considerations</span></span>

<span data-ttu-id="d9e10-139">Les performances représentent un aspect primordial pour les recommandations en temps réel, car celles-ci se situent généralement sur le chemin critique de la demande effectuée par un utilisateur sur votre site.</span><span class="sxs-lookup"><span data-stu-id="d9e10-139">Performance is a primary consideration for real-time recommendations, because recommendations usually fall in the critical path of the request a user makes on your site.</span></span>

<span data-ttu-id="d9e10-140">L’association d’AKS et d’Azure Cosmos DB permet à cette architecture de fournir un bon point de départ pour proposer des recommandations destinées à une charge de travail de taille moyenne avec une surcharge minimale.</span><span class="sxs-lookup"><span data-stu-id="d9e10-140">The combination of AKS and Azure Cosmos DB enables this architecture to provide a good starting point to provide recommendations for a medium-sized workload with minimal overhead.</span></span> <span data-ttu-id="d9e10-141">Sous un test de charge avec 200 utilisateurs simultanés, cette architecture fournit des recommandations à une latence moyenne d’environ 60 ms et fonctionne à un débit de 180 demandes par seconde.</span><span class="sxs-lookup"><span data-stu-id="d9e10-141">Under a load test with 200 concurrent users, this architecture provides recommendations at a median latency of about 60 ms and performs at a throughput of 180 requests per second.</span></span> <span data-ttu-id="d9e10-142">Le test de charge a été exécuté sur la configuration de déploiement par défaut (un cluster AKS 3x D3 v2 avec 12 processeurs virtuels, 42 Go de mémoire et 11 000 [unités de requête par seconde][ru] provisionnées pour Azure Cosmos DB).</span><span class="sxs-lookup"><span data-stu-id="d9e10-142">The load test was run against the default deployment configuration (a 3x D3 v2 AKS cluster with 12 vCPUs, 42 GB of memory, and 11,000 [Request Units (RUs) per second][ru] provisioned for Azure Cosmos DB).</span></span>

![Graphique de performances](./_images/recommenders-performance.png)

![Graphique de débit](./_images/recommenders-throughput.png)

<span data-ttu-id="d9e10-145">Azure Cosmos DB est recommandé pour sa distribution mondiale clés en main et son utilité pour répondre à toutes les exigences en matière de base de données de votre application.</span><span class="sxs-lookup"><span data-stu-id="d9e10-145">Azure Cosmos DB is recommended for its turnkey global distribution and usefulness in meeting any database requirements your app has.</span></span> <span data-ttu-id="d9e10-146">Pour une [latence légèrement plus rapide][latency], envisagez d’utiliser le [Cache Redis Azure][redis] au lieu d’Azure Cosmos DB pour traiter les recherches.</span><span class="sxs-lookup"><span data-stu-id="d9e10-146">For slightly [faster latency][latency], consider using [Azure Redis Cache][redis] instead of Azure Cosmos DB to serve lookups.</span></span> <span data-ttu-id="d9e10-147">Le Cache Redis peut améliorer les performances des systèmes qui reposent fortement sur les données dans les magasins back-end.</span><span class="sxs-lookup"><span data-stu-id="d9e10-147">Redis Cache can improve performance of systems that rely highly on data in back-end stores.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="d9e10-148">Considérations relatives à l’extensibilité</span><span class="sxs-lookup"><span data-stu-id="d9e10-148">Scalability considerations</span></span>

<span data-ttu-id="d9e10-149">Si vous n’envisagez pas d’utiliser Spark ou si vous avez une charge de travail plus petite où vous n’avez pas besoin de distribution, pensez à utiliser [Data Science Virtual Machine][dsvm] (DSVM) au lieu d’Azure Databricks.</span><span class="sxs-lookup"><span data-stu-id="d9e10-149">If you don't plan to use Spark, or you have a smaller workload where you don't need distribution, consider using [Data Science Virtual Machine][dsvm] (DSVM) instead of Azure Databricks.</span></span> <span data-ttu-id="d9e10-150">DSVM est une machine virtuelle Azure avec des outils et frameworks d’apprentissage profond pour le machine learning et la science des données.</span><span class="sxs-lookup"><span data-stu-id="d9e10-150">DSVM is an Azure virtual machine with deep learning frameworks and tools for machine learning and data science.</span></span> <span data-ttu-id="d9e10-151">Comme avec Azure Databricks, n’importe quel modèle que vous créez dans une machine virtuelle DSVM peut être rendu opérationnel en tant que service sur AKS via Azure Machine Learning.</span><span class="sxs-lookup"><span data-stu-id="d9e10-151">As with Azure Databricks, any model you create in a DSVM can be operationalized as a service on AKS via Azure Machine Learning.</span></span>

<span data-ttu-id="d9e10-152">Pendant l’entraînement, provisionnez un cluster Spark de taille fixe plus grand dans Azure Databricks ou configurez la [mise à l’échelle automatique][autoscaling].</span><span class="sxs-lookup"><span data-stu-id="d9e10-152">During training, provision a larger fixed-size Spark cluster in Azure Databricks or configure [autoscaling][autoscaling].</span></span> <span data-ttu-id="d9e10-153">Quand la mise à l’échelle automatique est activée, Databricks supervise la charge sur votre cluster et effectue un scale-up et un scale-down si nécessaire.</span><span class="sxs-lookup"><span data-stu-id="d9e10-153">When autoscaling is enabled, Databricks monitors the load on your cluster and scales up and downs when required.</span></span> <span data-ttu-id="d9e10-154">Provisionnez un cluster plus grand ou effectuez un scale-out de celui-ci si vous avez une taille de données importante et que vous souhaitez réduire la quantité de temps nécessaire pour la préparation des données ou les tâches de modélisation.</span><span class="sxs-lookup"><span data-stu-id="d9e10-154">Provision or scale out a larger cluster if you have a large data size and you want to reduce the amount of time it takes for data preparation or modeling tasks.</span></span>

<span data-ttu-id="d9e10-155">Mettez le cluster AKS à l’échelle pour répondre à vos besoins en termes de performances et de débit.</span><span class="sxs-lookup"><span data-stu-id="d9e10-155">Scale the AKS cluster to meet your performance and throughput requirements.</span></span> <span data-ttu-id="d9e10-156">Prenez soin d’effectuer un scale-up du nombre de [pods][scale] pour utiliser pleinement le cluster et de mettre à l’échelle les [nœuds][nodes] du cluster pour répondre à la demande de votre service.</span><span class="sxs-lookup"><span data-stu-id="d9e10-156">Take care to scale up the number of [pods][scale] to fully utilize the cluster, and to scale the [nodes][nodes] of the cluster to meet the demand of your service.</span></span> <span data-ttu-id="d9e10-157">Pour plus d’informations sur la façon de mettre à l’échelle votre cluster afin de répondre aux besoins en termes de performances et de débit de votre service de recommandation, consultez [Scaling Azure Container Service Clusters][blog].</span><span class="sxs-lookup"><span data-stu-id="d9e10-157">For more information on how to scale your cluster to meet the performance and throughput requirements of your recommender service, see [Scaling Azure Container Service Clusters][blog].</span></span>

<span data-ttu-id="d9e10-158">Pour gérer les performances Azure Cosmos DB, estimez le nombre de lectures nécessaires par seconde et provisionnez le nombre d’[unités de requête par seconde][ru] (débit) nécessaire.</span><span class="sxs-lookup"><span data-stu-id="d9e10-158">To manage Azure Cosmos DB performance, estimate the number of reads required per second, and provision the number of [RUs per second][ru] (throughput) needed.</span></span> <span data-ttu-id="d9e10-159">Utilisez les bonnes pratiques pour [le partitionnement et la mise à l’échelle horizontale][partition-data].</span><span class="sxs-lookup"><span data-stu-id="d9e10-159">Use best practices for [partitioning and horizontal scaling][partition-data].</span></span>

## <a name="cost-considerations"></a><span data-ttu-id="d9e10-160">Considérations relatives au coût</span><span class="sxs-lookup"><span data-stu-id="d9e10-160">Cost considerations</span></span>

<span data-ttu-id="d9e10-161">Les principaux facteurs de coût dans ce scénario sont :</span><span class="sxs-lookup"><span data-stu-id="d9e10-161">The main drivers of cost in this scenario are:</span></span>

- <span data-ttu-id="d9e10-162">Taille du cluster Azure Databricks nécessaire pour l’entraînement.</span><span class="sxs-lookup"><span data-stu-id="d9e10-162">The Azure Databricks cluster size required for training.</span></span>
- <span data-ttu-id="d9e10-163">Taille du cluster AKS nécessaire pour répondre à vos besoins de performance.</span><span class="sxs-lookup"><span data-stu-id="d9e10-163">The AKS cluster size required to meet your performance requirements.</span></span>
- <span data-ttu-id="d9e10-164">Unités de requête Azure Cosmos DB provisionnées pour répondre à vos besoins de performance.</span><span class="sxs-lookup"><span data-stu-id="d9e10-164">Azure Cosmos DB RUs provisioned to meet your performance requirements.</span></span>

<span data-ttu-id="d9e10-165">Gérez les coûts Azure Databricks en réentraînant moins fréquemment et en désactivant le cluster Spark quand il n’est pas utilisé.</span><span class="sxs-lookup"><span data-stu-id="d9e10-165">Manage the Azure Databricks costs by retraining less frequently and turning off the Spark cluster when not in use.</span></span> <span data-ttu-id="d9e10-166">Les coûts AKS et Azure Cosmos DB sont liés aux performances et au débit requis par votre site, et augmentent et baissent selon le volume de trafic sur votre site.</span><span class="sxs-lookup"><span data-stu-id="d9e10-166">The AKS and Azure Cosmos DB costs are tied to the throughput and performance required by your site and will scale up and down depending on the volume of traffic to your site.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="d9e10-167">Déployer la solution</span><span class="sxs-lookup"><span data-stu-id="d9e10-167">Deploy the solution</span></span>

<span data-ttu-id="d9e10-168">Pour déployer cette architecture, commencez par créer un environnement Azure Databricks pour préparer des données et entraîner un modèle de recommandation :</span><span class="sxs-lookup"><span data-stu-id="d9e10-168">To deploy this architecture, first create an Azure Databricks environment to prepare data and train a recommender model:</span></span>

1. <span data-ttu-id="d9e10-169">Créez un [espace de travail Azure Databricks][workspace].</span><span class="sxs-lookup"><span data-stu-id="d9e10-169">Create an [Azure Databricks workspace][workspace].</span></span>

2. <span data-ttu-id="d9e10-170">Créez un cluster dans Azure Databricks.</span><span class="sxs-lookup"><span data-stu-id="d9e10-170">Create a new cluster in Azure Databricks.</span></span> <span data-ttu-id="d9e10-171">La configuration suivante est requise :</span><span class="sxs-lookup"><span data-stu-id="d9e10-171">The following configuration is required:</span></span>

    - <span data-ttu-id="d9e10-172">Mode de cluster : standard</span><span class="sxs-lookup"><span data-stu-id="d9e10-172">Cluster mode: Standard</span></span>
    - <span data-ttu-id="d9e10-173">Version de Databricks Runtime : 4.1 (comprend Apache Spark 2.3.0, Scala 2.11)</span><span class="sxs-lookup"><span data-stu-id="d9e10-173">Databricks Runtime Version: 4.1 (includes Apache Spark 2.3.0, Scala 2.11)</span></span>
    - <span data-ttu-id="d9e10-174">Version de Python : 3</span><span class="sxs-lookup"><span data-stu-id="d9e10-174">Python Version: 3</span></span>
    - <span data-ttu-id="d9e10-175">Type de pilote : Standard\_DS3\_v2</span><span class="sxs-lookup"><span data-stu-id="d9e10-175">Driver Type: Standard\_DS3\_v2</span></span>
    - <span data-ttu-id="d9e10-176">Type de traitement web : Standard\_DS3\_v2 (min et max en fonction des besoins)</span><span class="sxs-lookup"><span data-stu-id="d9e10-176">Worker Type: Standard\_DS3\_v2 (min and max as required)</span></span>
    - <span data-ttu-id="d9e10-177">Arrêt automatique : (en fonction des besoins)</span><span class="sxs-lookup"><span data-stu-id="d9e10-177">Auto Termination: (as required)</span></span>
    - <span data-ttu-id="d9e10-178">Configuration Spark : (en fonction des besoins)</span><span class="sxs-lookup"><span data-stu-id="d9e10-178">Spark Config: (as required)</span></span>
    - <span data-ttu-id="d9e10-179">Variables d’environnement : (en fonction des besoins)</span><span class="sxs-lookup"><span data-stu-id="d9e10-179">Environment Variables: (as required)</span></span>

3. <span data-ttu-id="d9e10-180">Clonez le dépôt [Microsoft Recommenders][github] sur votre ordinateur local.</span><span class="sxs-lookup"><span data-stu-id="d9e10-180">Clone the [Microsoft Recommenders][github] repository on your local computer.</span></span>

4. <span data-ttu-id="d9e10-181">Compressez le contenu dans le dossier Recommenders :</span><span class="sxs-lookup"><span data-stu-id="d9e10-181">Zip the content inside the Recommenders folder:</span></span>

    ```console
    cd Recommenders
    zip -r Recommenders.zip
    ```

5. <span data-ttu-id="d9e10-182">Attachez la bibliothèque Recommenders à votre cluster comme suit :</span><span class="sxs-lookup"><span data-stu-id="d9e10-182">Attach the Recommenders library to your cluster as follows:</span></span>

    1. <span data-ttu-id="d9e10-183">Dans le menu suivant, utilisez l’option pour importer une bibliothèque (« pour importer une bibliothèque, comme un fichier jar ou egg, cliquez ici ») et appuyez sur **cliquer ici**.</span><span class="sxs-lookup"><span data-stu-id="d9e10-183">In the next menu, use the option to import a library ("To import a library, such as a jar or egg, click here") and press **click here**.</span></span>

    2. <span data-ttu-id="d9e10-184">Dans le premier menu déroulant, sélectionnez l’option **Upload Python egg or PyPI** (Charger Python Egg ou PyPi).</span><span class="sxs-lookup"><span data-stu-id="d9e10-184">At the first drop-down menu, select the **Upload Python egg or PyPI** option.</span></span>

    3. <span data-ttu-id="d9e10-185">Sélectionnez **Drop library egg here to upload** (Déposer la bibliothèque Egg ici à charger) et le fichier Recommenders.zip que vous venez de créer.</span><span class="sxs-lookup"><span data-stu-id="d9e10-185">Select **Drop library egg here to upload** and select the Recommenders.zip file you just created.</span></span>

    4. <span data-ttu-id="d9e10-186">Sélectionnez **Créer une bibliothèque** pour charger le fichier .zip et le rendre disponible dans votre espace de travail.</span><span class="sxs-lookup"><span data-stu-id="d9e10-186">Select **Create library** to upload the .zip file and make it available in your workspace.</span></span>

    5. <span data-ttu-id="d9e10-187">Dans le menu suivant, attachez la bibliothèque à votre cluster.</span><span class="sxs-lookup"><span data-stu-id="d9e10-187">In the next menu, attach the library to your cluster.</span></span>

6. <span data-ttu-id="d9e10-188">Dans votre espace de travail, importez l’[exemple ALS Movie Operationalization][als-example].</span><span class="sxs-lookup"><span data-stu-id="d9e10-188">In your workspace, import the [ALS Movie Operationalization example][als-example].</span></span>

7. <span data-ttu-id="d9e10-189">Exécutez le notebook ALS Movie Operationalization afin de générer les ressources requises pour créer une API de recommandation qui fournit les 10 meilleures recommandations de films pour un utilisateur donné.</span><span class="sxs-lookup"><span data-stu-id="d9e10-189">Run the ALS Movie Operationalization notebook to create the resources required to create a recommendation API that provides the top-10 movie recommendations for a given user.</span></span>

<!-- links -->
[aci]: /azure/container-instances/container-instances-overview
[aad]: /azure/active-directory-b2c/active-directory-b2c-overview
[aks]: /azure/aks/intro-kubernetes
[als]: https://spark.apache.org/docs/latest/ml-collaborative-filtering.html
[als-example]: https://github.com/Microsoft/Recommenders/blob/master/notebooks/04_operationalize/als_movie_o16n.ipynb
[autoscaling]: https://docs.azuredatabricks.net/user-guide/clusters/sizing.html
[autoscale]: https://docs.azuredatabricks.net/user-guide/clusters/sizing.html#autoscaling
[availability]: /azure/architecture/checklist/availability
[blob]: /azure/storage/blobs/storage-blobs-introduction
[blog]: https://blogs.technet.microsoft.com/machinelearning/2018/03/20/scaling-azure-container-service-cluster/
[clusters]: https://docs.azuredatabricks.net/user-guide/clusters/configure.html
[cosmosdb]: /azure/cosmos-db/introduction
[data-source]: https://docs.azuredatabricks.net/spark/latest/data-sources/index.html
[databricks]: /azure/azure-databricks/what-is-azure-databricks
[dsvm]: /azure/machine-learning/data-science-virtual-machine/overview
[dsvm-ubuntu]: /azure/machine-learning/data-science-virtual-machine/dsvm-ubuntu-intro
[eval-guide]: https://github.com/Microsoft/Recommenders/blob/master/notebooks/03_evaluate/evaluation.ipynb
[free]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[github]: https://github.com/Microsoft/Recommenders
[guide]: https://github.com/Microsoft/Recommenders/blob/master/notebooks/01_prepare_data/data_split.ipynb
[latency]: https://github.com/jessebenson/azure-performance
[mls]: /azure/machine-learning/service/
[n-tier]: /azure/architecture/reference-architectures/n-tier/n-tier-cassandra
[ndcg]: https://en.wikipedia.org/wiki/Discounted_cumulative_gain
[nodes]: /azure/aks/scale-cluster
[notebook]: https://github.com/Microsoft/Recommenders/notebooks/00_quick_start/als_pyspark_movielens.ipynb
[partition-data]: /azure/cosmos-db/partition-data
[redis]: /azure/redis-cache/cache-overview
[regions]: https://azure.microsoft.com/en-us/global-infrastructure/services/?products=virtual-machines&regions=all
[resiliency]: /azure/architecture/resiliency/
[ru]: /azure/cosmos-db/request-units
[sec-docs]: /azure/security/
[setup]: https://github.com/Microsoft/Recommenders/blob/master/SETUP.md%60
[scale]: /azure/aks/tutorial-kubernetes-scale
[sla]: https://azure.microsoft.com/en-us/support/legal/sla/virtual-machines/v1_8/
[vm-size]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[workspace]: https://docs.azuredatabricks.net/getting-started/index.html
