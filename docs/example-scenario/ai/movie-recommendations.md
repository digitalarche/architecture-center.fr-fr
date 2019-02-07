---
title: Suggestions de films sur Azure
description: Automatisez des suggestions de films, de produits et autres en utilisant le machine learning et une instance Azure Data Science Virtual Machine (DSVM) pour entraîner un modèle sur Azure.
author: njray
ms.date: 1/9/2019
ms.custom: azcat-ai, AI
social_image_url: /azure/architecture/example-scenario/ai/media/architecture-movie-recommender.png
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.openlocfilehash: 9387ab7989695df29df53d7aa4a437010cdd9fdf
ms.sourcegitcommit: 14226018a058e199523106199be9c07f6a3f8592
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/31/2019
ms.locfileid: "55482992"
---
# <a name="movie-recommendations-on-azure"></a><span data-ttu-id="0da75-103">Suggestions de films sur Azure</span><span class="sxs-lookup"><span data-stu-id="0da75-103">Movie recommendations on Azure</span></span>

<span data-ttu-id="0da75-104">Cet exemple de scénario montre comment une entreprise peut utiliser le machine learning pour automatiser des suggestions de produits pour ses clients.</span><span class="sxs-lookup"><span data-stu-id="0da75-104">This example scenario shows how a business can use machine learning to automate product recommendations for their customers.</span></span> <span data-ttu-id="0da75-105">Une instance Azure DSVM est employée pour entraîner un modèle sur Azure qui suggère des films aux utilisateurs en fonction des notes qui ont été données sur les films.</span><span class="sxs-lookup"><span data-stu-id="0da75-105">An Azure Data Science Virtual Machine (DSVM) is used to train a model on Azure that recommends movies to users based on ratings that have been given to movies.</span></span>

<span data-ttu-id="0da75-106">Les suggestions peuvent être utiles dans de nombreux secteurs d’activité, par exemple dans les commerces, les infos et les médias.</span><span class="sxs-lookup"><span data-stu-id="0da75-106">Recommendations can be useful in various industries from retail to news to media.</span></span> <span data-ttu-id="0da75-107">Par exemple, vous pouvez fournir des suggestions de produits dans un magasin virtuel, des suggestions d’articles d’actualité ou de posts, ou des suggestions de musique.</span><span class="sxs-lookup"><span data-stu-id="0da75-107">Potential applications include providing  product recommendations in a virtual store, providing news or post recommendations, or providing music recommendations.</span></span> <span data-ttu-id="0da75-108">Avant, les entreprises devaient employer et former des assistants pour rédiger des recommandations personnalisées à l’intention des clients.</span><span class="sxs-lookup"><span data-stu-id="0da75-108">Traditionally, businesses had to hire and train assistants to make personalized recommendations to customers.</span></span> <span data-ttu-id="0da75-109">Aujourd’hui, nous pouvons fournir des suggestions personnalisées à grande échelle en nous servant d’Azure pour entraîner des modèles qui comprennent les préférences des clients.</span><span class="sxs-lookup"><span data-stu-id="0da75-109">Today, we can  provide customized recommendations at scale by utilizing Azure to train models to understand customer preferences.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="0da75-110">Cas d’usage appropriés</span><span class="sxs-lookup"><span data-stu-id="0da75-110">Relevant use cases</span></span>

<span data-ttu-id="0da75-111">Pensez à ce scénario pour les cas d’usage suivants :</span><span class="sxs-lookup"><span data-stu-id="0da75-111">Consider this scenario for the following use cases:</span></span>

* <span data-ttu-id="0da75-112">Suggestions de films sur un site web.</span><span class="sxs-lookup"><span data-stu-id="0da75-112">Movie recommendations on a website.</span></span>
* <span data-ttu-id="0da75-113">Suggestions de biens de consommation dans une application mobile.</span><span class="sxs-lookup"><span data-stu-id="0da75-113">Consumer product recommendations in a mobile app.</span></span>
* <span data-ttu-id="0da75-114">Suggestions d’articles d’actualité sur des médias de streaming.</span><span class="sxs-lookup"><span data-stu-id="0da75-114">News recommendations on streaming media.</span></span>

## <a name="architecture"></a><span data-ttu-id="0da75-115">Architecture</span><span class="sxs-lookup"><span data-stu-id="0da75-115">Architecture</span></span>

![Architecture d’un modèle Machine Learning pour l’entraînement de recommandations sur des films][architecture]

<span data-ttu-id="0da75-117">Ce scénario couvre l’entraînement et l’évaluation du modèle Machine Learning à l’aide de l’algorithme Spark [des moindres carrés alternés][als] (ALS) sur un jeu de données de notes de films.</span><span class="sxs-lookup"><span data-stu-id="0da75-117">This scenario covers the training and evaluating of the machine learning model using the Spark [alternating least squares][als] (ALS) algorithm on a dataset of movie ratings.</span></span> <span data-ttu-id="0da75-118">Les étapes de ce scénario sont les suivantes :</span><span class="sxs-lookup"><span data-stu-id="0da75-118">The steps for this scenario are as following:</span></span>

1. <span data-ttu-id="0da75-119">Le service d’application ou le site web front-end collecte les données de l’historique des interactions utilisateurs-films qui sont représentées dans une table de tuples de classement numérique, d’éléments et d’utilisateurs.</span><span class="sxs-lookup"><span data-stu-id="0da75-119">The front-end website or app service collects historical data of user-movie interactions, which are represented in a table of user, item, and numerical rating tuples.</span></span>

2. <span data-ttu-id="0da75-120">Les données historiques collectées sont stockées dans un stockage blob.</span><span class="sxs-lookup"><span data-stu-id="0da75-120">The collected historical data is stored in a blob storage.</span></span>

3. <span data-ttu-id="0da75-121">Une DSVM est souvent utilisée pour faire des essais ou mettre en production un modèle de système de recommandations ALS Spark.</span><span class="sxs-lookup"><span data-stu-id="0da75-121">A DSVM is often used to experiment with or productize a Spark ALS recommender model.</span></span> <span data-ttu-id="0da75-122">Le modèle ALS est entraîné au moyen d’un jeu de données d’entraînement, lui-même produit à partir du jeu de données global, en appliquant la stratégie appropriée de division des données.</span><span class="sxs-lookup"><span data-stu-id="0da75-122">The ALS model is trained using a training dataset, which is produced from the overall dataset by applying the appropriate data splitting strategy.</span></span> <span data-ttu-id="0da75-123">Par exemple, le jeu de données peut être divisé en plusieurs jeux de façon aléatoire, chronologique ou stratifiée, en fonction des exigences de l’entreprise.</span><span class="sxs-lookup"><span data-stu-id="0da75-123">For example, the dataset can be split into sets randomly, chronologically, or stratified, depending on the business requirement.</span></span> <span data-ttu-id="0da75-124">Comme pour les autres tâches de machine learning, un système de recommandations est validé en utilisant des métriques d’évaluation (par exemple, précision\@*k*, rappel\@*k*, [moyenne de la précision moyenne (MAP)][map], [nDCG\@k][ndcg]).</span><span class="sxs-lookup"><span data-stu-id="0da75-124">Similar to other machine learning tasks, a recommender is validated by using evaluation metrics (for example, precision\@*k*, recall\@*k*, [MAP][map], [nDCG\@k][ndcg]).</span></span>

4. <span data-ttu-id="0da75-125">Azure Machine Learning service sert à coordonner l’expérimentation, comme le balayage d’hyperparamètres et la gestion du modèle.</span><span class="sxs-lookup"><span data-stu-id="0da75-125">Azure Machine Learning service is used for coordinating the experimentation, such as hyperparameter sweeping and model management.</span></span>

5. <span data-ttu-id="0da75-126">Un modèle entraîné est conservé sur Azure Cosmos DB et peut ensuite être appliqué pour suggérer les *k* meilleurs films à un utilisateur donné.</span><span class="sxs-lookup"><span data-stu-id="0da75-126">A trained model is preserved on Azure Cosmos DB, which can then be applied for recommending the top *k* movies for a given user.</span></span>

6. <span data-ttu-id="0da75-127">Le modèle est alors déployé sur un service web ou un service d’application par le biais d’Azure Container Instances ou d’Azure Kubernetes Service.</span><span class="sxs-lookup"><span data-stu-id="0da75-127">The model is then deployed onto a web or app service by using Azure Container Instances or Azure Kubernetes Service.</span></span>

<span data-ttu-id="0da75-128">Pour des instructions détaillées sur la création et la mise à l’échelle d’un service de recommandations, consultez [Créer une API de recommandation en temps réel sur Azure][ref-arch].</span><span class="sxs-lookup"><span data-stu-id="0da75-128">For an in-depth guide to building and scaling a recommender service, see [Build a real-time recommendation API on Azure][ref-arch].</span></span>

### <a name="components"></a><span data-ttu-id="0da75-129">Composants</span><span class="sxs-lookup"><span data-stu-id="0da75-129">Components</span></span>

* <span data-ttu-id="0da75-130">[Data Science Virtual Machine][dsvm] (DSVM) est une machine virtuelle Azure dotée d’outils et de frameworks de deep learning pour le machine learning et la science des données.</span><span class="sxs-lookup"><span data-stu-id="0da75-130">[Data Science Virtual Machine][dsvm] (DSVM) is an Azure virtual machine with deep learning frameworks and tools for machine learning and data science.</span></span> <span data-ttu-id="0da75-131">La machine virtuelle DSVM dispose d’un environnement Spark autonome qui peut servir à exécuter ALS.</span><span class="sxs-lookup"><span data-stu-id="0da75-131">The DSVM has a standalone Spark environment that can be used to run ALS.</span></span>

* <span data-ttu-id="0da75-132">Le [stockage Blob Azure][blob] stocke le jeu de données pour les suggestions de films.</span><span class="sxs-lookup"><span data-stu-id="0da75-132">[Azure Blob storage][blob] stores the dataset for movie recommendations.</span></span>

* <span data-ttu-id="0da75-133">[Azure Machine Learning service][mls] est utilisé pour accélérer la création, la gestion et le déploiement de modèles Machine Learning.</span><span class="sxs-lookup"><span data-stu-id="0da75-133">[Azure Machine Learning service][mls] is used to accelerate the building, managing, and deploying of machine learning models.</span></span>

* <span data-ttu-id="0da75-134">[Azure Cosmos DB][cosmosdb] assure le stockage de bases de données multimodèles distribuées à l’échelle mondiale.</span><span class="sxs-lookup"><span data-stu-id="0da75-134">[Azure Cosmos DB][cosmosdb] enables globally distributed and multi-model database storage.</span></span>

* <span data-ttu-id="0da75-135">[Azure Container Instances][aci] sert à déployer les modèles entraînés sur les services web ou d’application, en utilisant au besoin [Azure Kubernetes Service][aks].</span><span class="sxs-lookup"><span data-stu-id="0da75-135">[Azure Container Instances][aci] is used to deploy the trained models to web or app services, optionally using [Azure Kubernetes Service][aks].</span></span>

### <a name="alternatives"></a><span data-ttu-id="0da75-136">Autres solutions</span><span class="sxs-lookup"><span data-stu-id="0da75-136">Alternatives</span></span>

<span data-ttu-id="0da75-137">[Azure Databricks][databricks] est un cluster Spark managé dans lequel l’entraînement et l’évaluation de modèles sont effectués.</span><span class="sxs-lookup"><span data-stu-id="0da75-137">[Azure Databricks][databricks] is a managed Spark cluster where model training and evaluating is performed.</span></span> <span data-ttu-id="0da75-138">Vous pouvez configurer un environnement Spark managé en quelques minutes, et le [mettre à l’échelle automatiquement][autoscale] afin de réduire les ressources et les coûts associés à une mise à l’échelle manuelle des clusters.</span><span class="sxs-lookup"><span data-stu-id="0da75-138">You can set up a managed Spark environment in minutes, and [autoscale][autoscale] up and down to help reduce the resources and costs associated with scaling clusters manually.</span></span> <span data-ttu-id="0da75-139">Une autre option visant à économiser des ressources consiste à configurer des [clusters][clusters] inactifs pour un arrêt automatique.</span><span class="sxs-lookup"><span data-stu-id="0da75-139">Another resource-saving option is to configure inactive [clusters][clusters] to terminate automatically.</span></span>

## <a name="considerations"></a><span data-ttu-id="0da75-140">Considérations</span><span class="sxs-lookup"><span data-stu-id="0da75-140">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="0da75-141">Disponibilité</span><span class="sxs-lookup"><span data-stu-id="0da75-141">Availability</span></span>

<span data-ttu-id="0da75-142">Les applications créés pour le machine learning sont divisées en deux composants de ressources : les ressources destinées à l’entraînement et celles destinées au traitement.</span><span class="sxs-lookup"><span data-stu-id="0da75-142">Machine-learning-built apps are split into two resource components: resources for training, and resources for serving.</span></span> <span data-ttu-id="0da75-143">Les ressources nécessaires à l’entraînement n’ont généralement pas besoin de haute disponibilité, car les requêtes de production en temps réel ne touchent pas directement ces ressources.</span><span class="sxs-lookup"><span data-stu-id="0da75-143">Resources required for training generally do not need  high availability, as live production requests do not directly hit these resources.</span></span> <span data-ttu-id="0da75-144">Par contre, les ressources nécessaires au traitement nécessitent une haute disponibilité pour gérer les requêtes clientes.</span><span class="sxs-lookup"><span data-stu-id="0da75-144">Resources required for serving need to have high availability to serve customer requests.</span></span>

<span data-ttu-id="0da75-145">Pour l’entraînement, la machine DSVM est disponible dans [plusieurs régions][regions] du monde entier et remplit le [contrat de niveau de service][sla] (SLA) des machines virtuelles.</span><span class="sxs-lookup"><span data-stu-id="0da75-145">For training, the DSVM is available in [multiple regions][regions] around the globe and meets the [service level agreement][sla] (SLA) for virtual machines.</span></span> <span data-ttu-id="0da75-146">Pour le traitement, Azure Kubernetes Service fournit une infrastructure [hautement disponible][ha].</span><span class="sxs-lookup"><span data-stu-id="0da75-146">For serving, Azure Kubernetes Service provides a [highly available][ha] infrastructure.</span></span> <span data-ttu-id="0da75-147">Les nœuds d’agent suivent également le [SLA][sla-aks] des machines virtuelles.</span><span class="sxs-lookup"><span data-stu-id="0da75-147">Agent nodes also follow the [SLA][sla-aks] for virtual machines.</span></span>

### <a name="scalability"></a><span data-ttu-id="0da75-148">Extensibilité</span><span class="sxs-lookup"><span data-stu-id="0da75-148">Scalability</span></span>

<span data-ttu-id="0da75-149">Si vous avez un volume de données important, vous pouvez mettre à l’échelle votre DSVM afin de raccourcir le temps d’entraînement.</span><span class="sxs-lookup"><span data-stu-id="0da75-149">If you have a large data size, you can scale your DSVM to shorten training time.</span></span> <span data-ttu-id="0da75-150">Vous pouvez effectuer un scale-up ou un scale-down d’une machine virtuelle en changeant sa [taille][vm-size].</span><span class="sxs-lookup"><span data-stu-id="0da75-150">You can scale a VM up or down by changing the [VM size][vm-size].</span></span> <span data-ttu-id="0da75-151">Choisissez une taille de mémoire adaptée à votre jeu de données en mémoire, et un nombre plus élevé de processeurs virtuels afin de réduire le temps nécessaire à l’entraînement.</span><span class="sxs-lookup"><span data-stu-id="0da75-151">Choose a memory size large enough to fit your dataset in-memory and a higher vCPU count in order to decrease the amount of time that training takes.</span></span>

### <a name="security"></a><span data-ttu-id="0da75-152">Sécurité</span><span class="sxs-lookup"><span data-stu-id="0da75-152">Security</span></span>

<span data-ttu-id="0da75-153">Ce scénario peut utiliser Azure Active Directory pour authentifier les utilisateurs qui [accèdent à la machine DSVM][dsvm-id] contenant votre code, vos modèles et vos données (en mémoire).</span><span class="sxs-lookup"><span data-stu-id="0da75-153">This scenario can use Azure Active Directory to authenticate users for [access to the DSVM][dsvm-id], which contains your code, models, and (in-memory) data.</span></span> <span data-ttu-id="0da75-154">Les données sont stockées dans Stockage Azure avant d’être chargées sur une machine DSVM où elles sont automatiquement chiffrées à l’aide de [Storage Service Encryption][storage-security].</span><span class="sxs-lookup"><span data-stu-id="0da75-154">Data is stored in Azure Storage prior to being loaded on a DSVM, where it is automatically encrypted using [Storage Service Encryption][storage-security].</span></span> <span data-ttu-id="0da75-155">Les autorisations peuvent être managées via l’authentification Azure Active Directory ou le contrôle d’accès en fonction du rôle.</span><span class="sxs-lookup"><span data-stu-id="0da75-155">Permissions can be managed via Azure Active Directory authentication or role-based access control.</span></span>

## <a name="deploy-this-scenario"></a><span data-ttu-id="0da75-156">Déployer ce scénario</span><span class="sxs-lookup"><span data-stu-id="0da75-156">Deploy this scenario</span></span>

<span data-ttu-id="0da75-157">**Conditions préalables**: Vous devez disposer d’un compte Azure existant.</span><span class="sxs-lookup"><span data-stu-id="0da75-157">**Prerequisites**: You must have an existing Azure account.</span></span> <span data-ttu-id="0da75-158">Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit][free] avant de commencer.</span><span class="sxs-lookup"><span data-stu-id="0da75-158">If you don't have an Azure subscription, create a [free account][free] before you begin.</span></span>

<span data-ttu-id="0da75-159">L’intégralité du code de ce scénario est disponible dans le [dépôt Microsoft Recommenders][github].</span><span class="sxs-lookup"><span data-stu-id="0da75-159">All the code for this scenario is available in the [Microsoft Recommenders repository][github].</span></span>

<span data-ttu-id="0da75-160">Suivez ces étapes pour exécuter le [notebook de démarrage rapide ALS][notebook] :</span><span class="sxs-lookup"><span data-stu-id="0da75-160">Follow these steps to run the [ALS quickstart notebook][notebook]:</span></span>

1. <span data-ttu-id="0da75-161">[Créez une machine DSVM][dsvm-ubuntu] à partir du portail Azure.</span><span class="sxs-lookup"><span data-stu-id="0da75-161">[Create a DSVM][dsvm-ubuntu] from the Azure portal.</span></span>

2. <span data-ttu-id="0da75-162">Clonez le dépôt dans le dossier Notebooks :</span><span class="sxs-lookup"><span data-stu-id="0da75-162">Clone the repo in the Notebooks folder:</span></span>

    ```shell
    cd notebooks
    git clone https://github.com/Microsoft/Recommenders
    ```

3. <span data-ttu-id="0da75-163">Installez les dépendances conda en suivant les étapes décrites dans le fichier [SETUP.md][setup].</span><span class="sxs-lookup"><span data-stu-id="0da75-163">Install the conda dependencies following the steps described in the [SETUP.md][setup] file.</span></span>

4. <span data-ttu-id="0da75-164">Dans un navigateur, affichez votre machine virtuelle jupyterlab et accédez à `notebooks/00_quick_start/als_pyspark_movielens.ipynb`.</span><span class="sxs-lookup"><span data-stu-id="0da75-164">In a browser, go to your jupyterlab VM and navigate to `notebooks/00_quick_start/als_pyspark_movielens.ipynb`.</span></span>

5. <span data-ttu-id="0da75-165">Exécutez le notebook.</span><span class="sxs-lookup"><span data-stu-id="0da75-165">Execute the notebook.</span></span>

## <a name="related-resources"></a><span data-ttu-id="0da75-166">Ressources associées</span><span class="sxs-lookup"><span data-stu-id="0da75-166">Related resources</span></span>

<span data-ttu-id="0da75-167">Pour des instructions détaillées sur la création et la mise à l’échelle d’un service de recommandations, consultez [Créer une API de recommandation en temps réel sur Azure][ref-arch].</span><span class="sxs-lookup"><span data-stu-id="0da75-167">For an in-depth guide to building and scaling a recommender service, see [Build a real-time recommendation API on Azure][ref-arch].</span></span> <span data-ttu-id="0da75-168">Pour des tutoriels et des exemples de systèmes de recommandations, consultez le [Dépôt Microsoft Recommenders][github].</span><span class="sxs-lookup"><span data-stu-id="0da75-168">For tutorials and examples of recommendation systems, see [Microsoft Recommenders repository][github].</span></span>

[architecture]: ./media/architecture-movie-recommender.png
[aci]: /azure/container-instances/container-instances-overview
[aad]: /azure/active-directory-b2c/active-directory-b2c-overview
[aks]: /azure/aks/intro-kubernetes
[als]: https://spark.apache.org/docs/latest/ml-collaborative-filtering.html
[autoscale]: https://docs.azuredatabricks.net/user-guide/clusters/sizing.html#autoscaling
[blob]: /azure/storage/blobs/storage-blobs-introduction
[clusters]: https://docs.azuredatabricks.net/user-guide/clusters/configure.html
[cosmosdb]: /azure/cosmos-db/introduction
[databricks]: /azure/azure-databricks/what-is-azure-databricks
[dsvm]: /azure/machine-learning/data-science-virtual-machine/overview
[dsvm-id]: /azure/machine-learning/data-science-virtual-machine/dsvm-common-identity
[dsvm-ubuntu]: /azure/machine-learning/data-science-virtual-machine/dsvm-ubuntu-intro
[free]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[github]: https://github.com/Microsoft/Recommenders
[ha]: /azure/aks/container-service-quotas
[map]: https://en.wikipedia.org/wiki/Evaluation_measures_(information_retrieval)
[mls]: /azure/machine-learning/service/
[n-tier]: /azure/architecture/reference-architectures/n-tier/n-tier-cassandra
[ndcg]: https://en.wikipedia.org/wiki/Discounted_cumulative_gain
[notebook]: https://github.com/Microsoft/Recommenders/notebooks/00_quick_start/als_pyspark_movielens.ipynb
[ref-arch]: /azure/architecture/reference-architectures/ai/real-time-recommendation
[regions]: https://azure.microsoft.com/en-us/global-infrastructure/services/?products=virtual-machines&regions=all
[resiliency]: /azure/architecture/resiliency/
[sec-docs]: /azure/security/
[setup]: https://github.com/Microsoft/Recommenders/blob/master/SETUP.md%60
[sla]: https://azure.microsoft.com/en-us/support/legal/sla/virtual-machines/v1_8/
[sla-aks]: https://azure.microsoft.com/en-us/support/legal/sla/kubernetes-service/v1_0/
[storage-security]: /azure/storage/common/storage-service-encryption
[vm-size]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
