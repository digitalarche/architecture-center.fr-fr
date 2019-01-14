---
title: Sélectionner une technologie Machine Learning
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: e4b8ecb64afbacc8a4e24e9c6274455db0451d62
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/08/2019
ms.locfileid: "54112123"
---
# <a name="choosing-a-machine-learning-technology-in-azure"></a><span data-ttu-id="bf95c-102">Sélectionner une technologie Machine Learning dans Azure</span><span class="sxs-lookup"><span data-stu-id="bf95c-102">Choosing a machine learning technology in Azure</span></span>

<span data-ttu-id="bf95c-103">La science des données et Machine Learning est une charge de travail généralement effectuée par les scientifiques de données.</span><span class="sxs-lookup"><span data-stu-id="bf95c-103">Data science and machine learning is a workload that is usually undertaken by data scientists.</span></span> <span data-ttu-id="bf95c-104">Elle requiert des outils spécialisés, qui sont conçus spécifiquement pour le type de tâches d’exploration interactive des données et de modélisation qu’un scientifique de données doit effectuer.</span><span class="sxs-lookup"><span data-stu-id="bf95c-104">It requires specialist tools, many of which are designed specifically for the type of interactive data exploration and modeling tasks that a data scientist must perform.</span></span>

<span data-ttu-id="bf95c-105">Les solutions Machine Learning sont générées de manière itérative et comportent deux phases distinctes :</span><span class="sxs-lookup"><span data-stu-id="bf95c-105">Machine learning solutions are built iteratively, and have two distinct phases:</span></span>

- <span data-ttu-id="bf95c-106">Préparation et modélisation des données.</span><span class="sxs-lookup"><span data-stu-id="bf95c-106">Data preparation and modeling.</span></span>
- <span data-ttu-id="bf95c-107">Déploiement et consommation des services prédictifs.</span><span class="sxs-lookup"><span data-stu-id="bf95c-107">Deployment and consumption of predictive services.</span></span>

## <a name="tools-and-services-for-data-preparation-and-modeling"></a><span data-ttu-id="bf95c-108">Outils et services pour la préparation et la modélisation des données</span><span class="sxs-lookup"><span data-stu-id="bf95c-108">Tools and services for data preparation and modeling</span></span>

<span data-ttu-id="bf95c-109">En général, les scientifiques de données préfèrent travailler avec des données utilisant un code personnalisé écrit dans Python ou R. Ce code est généralement exécuté de manière interactive. Les scientifiques de données l’utilise pour interroger et explorer les données, générer des visualisations et des statistiques afin de déterminer les relations avec le code.</span><span class="sxs-lookup"><span data-stu-id="bf95c-109">Data scientists typically prefer to work with data using custom code written in Python or R. This code is generally run interactively, with the data scientists using it to query and explore the data, generating visualizations and statistics to help determine the relationships with it.</span></span> <span data-ttu-id="bf95c-110">Il existe de nombreux environnements interactifs pour R et Python que les scientifiques de données peuvent utiliser.</span><span class="sxs-lookup"><span data-stu-id="bf95c-110">There are many interactive environments for R and Python that data scientists can use.</span></span> <span data-ttu-id="bf95c-111">L’un des favoris, **Jupyter Notebooks**, fournit un shell basé sur navigateur qui permet aux scientifiques de données de créer des fichiers de *bloc-notes* qui contiennent du code R ou Python et du texte Markdown.</span><span class="sxs-lookup"><span data-stu-id="bf95c-111">A particular favorite is **Jupyter Notebooks** that provides a browser-based shell that enables data scientists to create *notebook* files that contain R or Python code and markdown text.</span></span> <span data-ttu-id="bf95c-112">Cela constitue un moyen efficace de collaborer en partageant et en documentant le code et des résultats dans un seul document.</span><span class="sxs-lookup"><span data-stu-id="bf95c-112">This is an effective way to collaborate by sharing and documenting code and results in a single document.</span></span>

<span data-ttu-id="bf95c-113">Autres outils couramment utilisés :</span><span class="sxs-lookup"><span data-stu-id="bf95c-113">Other commonly used tools include:</span></span>

- <span data-ttu-id="bf95c-114">**Spyder** : IDE (environnement de développement interactif) pour Python, fourni avec la distribution Anaconda Python.</span><span class="sxs-lookup"><span data-stu-id="bf95c-114">**Spyder**: The interactive development environment (IDE) for Python provided with the Anaconda Python distribution.</span></span>
- <span data-ttu-id="bf95c-115">**R Studio** : IDE pour le langage de programmation R.</span><span class="sxs-lookup"><span data-stu-id="bf95c-115">**R Studio**: An IDE for the R programming language.</span></span>
- <span data-ttu-id="bf95c-116">**Visual Studio Code** : Environnement de codage léger et multiplateforme qui prend en charge Python ainsi que les frameworks couramment utilisés pour l’apprentissage automatique et le développement de l’IA.</span><span class="sxs-lookup"><span data-stu-id="bf95c-116">**Visual Studio Code**: A lightweight, cross-platform coding environment that supports Python as well as commonly used frameworks for machine learning and AI development.</span></span>

<span data-ttu-id="bf95c-117">Outre ces outils, les scientifiques de données peuvent tirer parti des services Azure pour simplifier la gestion du code et du modèle.</span><span class="sxs-lookup"><span data-stu-id="bf95c-117">In addition to these tools, data scientists can leverage Azure services to simplify code and model management.</span></span>

### <a name="azure-notebooks"></a><span data-ttu-id="bf95c-118">Azure Notebooks</span><span class="sxs-lookup"><span data-stu-id="bf95c-118">Azure Notebooks</span></span>

<span data-ttu-id="bf95c-119">Azure Notebooks est un service Jupyter Notebooks en ligne qui permet aux scientifiques de données de créer, d’exécuter et de partager Jupyter Notebooks dans des bibliothèques cloud.</span><span class="sxs-lookup"><span data-stu-id="bf95c-119">Azure Notebooks is an online Jupyter Notebooks service that enables data scientists to create, run, and share Jupyter Notebooks in cloud-based libraries.</span></span>

<span data-ttu-id="bf95c-120">Principaux avantages :</span><span class="sxs-lookup"><span data-stu-id="bf95c-120">Key benefits:</span></span>

- <span data-ttu-id="bf95c-121">Service gratuit&mdash;pas d’abonnement Azure requis.</span><span class="sxs-lookup"><span data-stu-id="bf95c-121">Free service&mdash;no Azure subscription required.</span></span>
- <span data-ttu-id="bf95c-122">Inutile d’installer Jupyter et les distributions R ou Python connexes localement&mdash;utilisez simplement un navigateur.</span><span class="sxs-lookup"><span data-stu-id="bf95c-122">No need to install Jupyter and the supporting R or Python distributions locally&mdash;just use a browser.</span></span>
- <span data-ttu-id="bf95c-123">Gérez vos propres bibliothèques en ligne et accédez-y sur n’importe quel appareil.</span><span class="sxs-lookup"><span data-stu-id="bf95c-123">Manage your own online libraries and access them from any device.</span></span>
- <span data-ttu-id="bf95c-124">Partagez vos blocs-notes avec des collaborateurs.</span><span class="sxs-lookup"><span data-stu-id="bf95c-124">Share your notebooks with collaborators.</span></span>

<span data-ttu-id="bf95c-125">Considérations :</span><span class="sxs-lookup"><span data-stu-id="bf95c-125">Considerations:</span></span>

- <span data-ttu-id="bf95c-126">Vous ne pourrez pas accéder à vos blocs-notes en mode hors connexion.</span><span class="sxs-lookup"><span data-stu-id="bf95c-126">You will be unable to access your notebooks when offline.</span></span>
- <span data-ttu-id="bf95c-127">Les capacités de traitement limitées du service de bloc-notes gratuit peuvent ne pas suffire pour l’apprentissage de modèles volumineux ou complexes.</span><span class="sxs-lookup"><span data-stu-id="bf95c-127">Limited processing capabilities of the free notebook service may not be enough to train large or complex models.</span></span>

### <a name="data-science-virtual-machine"></a><span data-ttu-id="bf95c-128">Machine virtuelle de science des données</span><span class="sxs-lookup"><span data-stu-id="bf95c-128">Data science virtual machine</span></span>

<span data-ttu-id="bf95c-129">La machine virtuelle de science des données est une image de machine virtuelle Azure qui inclut les outils et les frameworks couramment utilisés par les scientifiques de données, notamment R, Python, Jupyter Notebooks, Visual Studio Code, et les bibliothèques pour la modélisation Machine Learning comme Microsoft Cognitive Toolkit.</span><span class="sxs-lookup"><span data-stu-id="bf95c-129">The data science virtual machine is an Azure virtual machine image that includes the tools and frameworks commonly used by data scientists, including R, Python, Jupyter Notebooks, Visual Studio Code, and libraries for machine learning modeling such as the Microsoft Cognitive Toolkit.</span></span> <span data-ttu-id="bf95c-130">Ces outils peuvent être complexes et longs à installer, et contiennent de nombreuses interdépendances qui mènent souvent à des problèmes de gestion de version.</span><span class="sxs-lookup"><span data-stu-id="bf95c-130">These tools can be complex and time consuming to install, and contain many interdependencies that often lead to version management issues.</span></span> <span data-ttu-id="bf95c-131">Le fait de disposer d’une image pré-installée peut réduire le temps que les scientifiques de données passent à résoudre les problèmes d’environnement, ce qui leur permet de se concentrer sur les tâches d’exploration et de modélisation des données qu’ils doivent exécuter.</span><span class="sxs-lookup"><span data-stu-id="bf95c-131">Having a preinstalled image can reduce the time data scientists spend troubleshooting environment issues, allowing them to focus on the data exploration and modeling tasks they need to perform.</span></span>

<span data-ttu-id="bf95c-132">Principaux avantages :</span><span class="sxs-lookup"><span data-stu-id="bf95c-132">Key benefits:</span></span>

- <span data-ttu-id="bf95c-133">Réduction du temps d’installation, de gestion et de résolution des problèmes liés aux outils et aux frameworks de science des données.</span><span class="sxs-lookup"><span data-stu-id="bf95c-133">Reduced time to install, manage, and troubleshoot data science tools and frameworks.</span></span>
- <span data-ttu-id="bf95c-134">Les dernières versions de tous les outils et frameworks couramment utilisés sont incluses.</span><span class="sxs-lookup"><span data-stu-id="bf95c-134">The latest versions of all commonly used tools and frameworks are included.</span></span>
- <span data-ttu-id="bf95c-135">Les options de machine virtuelle incluent des images hautement évolutives avec des fonctionnalités GPU pour la modélisation intensive des données.</span><span class="sxs-lookup"><span data-stu-id="bf95c-135">Virtual machine options include highly scalable images with GPU capabilities for intensive data modeling.</span></span>

<span data-ttu-id="bf95c-136">Considérations :</span><span class="sxs-lookup"><span data-stu-id="bf95c-136">Considerations:</span></span>

- <span data-ttu-id="bf95c-137">La machine virtuelle n’est pas accessible en mode hors connexion.</span><span class="sxs-lookup"><span data-stu-id="bf95c-137">The virtual machine cannot be accessed when offline.</span></span>
- <span data-ttu-id="bf95c-138">L’exécution d’une machine virtuelle entraîne des frais Azure, donc vous devez veiller à l’exécuter uniquement si nécessaire.</span><span class="sxs-lookup"><span data-stu-id="bf95c-138">Running a virtual machine incurs Azure charges, so you must be careful to have it running only when required.</span></span>

### <a name="azure-machine-learning"></a><span data-ttu-id="bf95c-139">Azure Machine Learning</span><span class="sxs-lookup"><span data-stu-id="bf95c-139">Azure Machine Learning</span></span>

<span data-ttu-id="bf95c-140">Azure Machine Learning est un service cloud pour la gestion des modèles et des expériences Machine Learning.</span><span class="sxs-lookup"><span data-stu-id="bf95c-140">Azure Machine Learning is a cloud-based service for managing machine learning experiments and models.</span></span> <span data-ttu-id="bf95c-141">Il inclut un service d’expérimentation qui effectue le suivi des scripts d’apprentissage de préparation et de modélisation des données, en conservant un historique de toutes les exécutions, qui vous permet de comparer les performances du modèle entre les itérations.</span><span class="sxs-lookup"><span data-stu-id="bf95c-141">It includes an experimentation service that tracks data preparation and modeling training scripts, maintaining a history of all executions so you can compare model performance across iterations.</span></span> <span data-ttu-id="bf95c-142">Les scientifiques des données peuvent créer des scripts dans l’outil de leur choix, par exemple Jupyter Notebooks ou Visual Studio Code, puis les déployer sur différentes [ressources de calcul](/azure/machine-learning/service/how-to-set-up-training-targets) dans Azure.</span><span class="sxs-lookup"><span data-stu-id="bf95c-142">Data scientists can create scripts in their tool of choice, such as Jupyter Notebooks or Visual Studio Code, and then deploy to a variety of different [compute resources](/azure/machine-learning/service/how-to-set-up-training-targets) in Azure.</span></span>

<span data-ttu-id="bf95c-143">Vous pouvez déployer des modèles sous forme de services web sur un conteneur Docker, Spark sur Azure HDinsight, Microsoft Machine Learning Server ou SQL Server.</span><span class="sxs-lookup"><span data-stu-id="bf95c-143">Models can be deployed as a web service to a Docker container, Spark on Azure HDinsight, Microsoft Machine Learning Server, or SQL Server.</span></span> <span data-ttu-id="bf95c-144">Le service Gestion des modèles Azure Machine Learning vous permet d’effectuer le suivi et de gérer les déploiements de modèle dans le cloud, sur des appareils de périphérie ou à l’échelle de l’entreprise.</span><span class="sxs-lookup"><span data-stu-id="bf95c-144">The Azure Machine Learning Model Management service then enables you to track and manage model deployments in the cloud, on edge devices, or across the enterprise.</span></span>

<span data-ttu-id="bf95c-145">Principaux avantages :</span><span class="sxs-lookup"><span data-stu-id="bf95c-145">Key benefits:</span></span>

- <span data-ttu-id="bf95c-146">Gestion centralisée des scripts et historique d’exécution, ce qui facilite la comparaison des versions du modèle.</span><span class="sxs-lookup"><span data-stu-id="bf95c-146">Central management of scripts and run history, making it easy to compare model versions.</span></span>
- <span data-ttu-id="bf95c-147">Transformation interactive des données via un éditeur visuel.</span><span class="sxs-lookup"><span data-stu-id="bf95c-147">Interactive data transformation through a visual editor.</span></span>
- <span data-ttu-id="bf95c-148">Déploiement et gestion simples des modèles sur le cloud ou sur des appareils de périphérie.</span><span class="sxs-lookup"><span data-stu-id="bf95c-148">Easy deployment and management of models to the cloud or edge devices.</span></span>

<span data-ttu-id="bf95c-149">Considérations :</span><span class="sxs-lookup"><span data-stu-id="bf95c-149">Considerations:</span></span>

- <span data-ttu-id="bf95c-150">Nécessite des connaissances relatives au modèle de Gestion des modèles.</span><span class="sxs-lookup"><span data-stu-id="bf95c-150">Requires some familiarity with the model management model.</span></span>

### <a name="azure-batch-ai"></a><span data-ttu-id="bf95c-151">Azure Batch AI</span><span class="sxs-lookup"><span data-stu-id="bf95c-151">Azure Batch AI</span></span>

<span data-ttu-id="bf95c-152">Azure Batch AI vous permet d’exécuter vos expériences Machine Learning en parallèle et d’effectuer l’apprentissage du modèle à l’échelle sur un cluster de machines virtuelles avec des GPU.</span><span class="sxs-lookup"><span data-stu-id="bf95c-152">Azure Batch AI enables you to run your machine learning experiments in parallel, and perform model training at scale across a cluster of virtual machines with GPUs.</span></span> <span data-ttu-id="bf95c-153">Batch AI Training vous permet de monter en charge des tâches d’apprentissage profond sur des GPU en cluster, en utilisant des frameworks comme Cognitive Toolkit, Caffe, Chainer et TensorFlow.</span><span class="sxs-lookup"><span data-stu-id="bf95c-153">Batch AI training enables you to scale out deep learning jobs across clustered GPUs, using frameworks such as Cognitive Toolkit, Caffe, Chainer, and TensorFlow.</span></span>

<span data-ttu-id="bf95c-154">Le service Gestion des modèles Azure Machine Learning peut servir à retirer des modèles de Batch AI Training pour les déployer, gérer et surveiller.</span><span class="sxs-lookup"><span data-stu-id="bf95c-154">Azure Machine Learning Model Management can be used to take models from Batch AI training to deploy, manage, and monitor them.</span></span>

### <a name="azure-machine-learning-studio"></a><span data-ttu-id="bf95c-155">Azure Machine Learning Studio</span><span class="sxs-lookup"><span data-stu-id="bf95c-155">Azure Machine Learning Studio</span></span>

<span data-ttu-id="bf95c-156">Azure Machine Learning Studio est un environnement de développement visuel, basé sur le cloud, pour la création d’expériences de données, la formation de modèles Machine Learning et leur publication en tant que services web dans Azure.</span><span class="sxs-lookup"><span data-stu-id="bf95c-156">Azure Machine Learning Studio is a cloud-based, visual development environment for creating data experiments, training machine learning models, and publishing them as web services in Azure.</span></span> <span data-ttu-id="bf95c-157">Son interface de type glisser-déposer visuelle permet aux scientifiques de données et aux utilisateurs avancés de créer rapidement des solutions Machine Learning, tout en prenant en charge la logique R et Python personnalisée, une grande variété de techniques et d’algorithmes statistiques établis pour les tâches de modélisation Machine Learning, et la prise en charge intégrée de Jupyter Notebooks.</span><span class="sxs-lookup"><span data-stu-id="bf95c-157">Its visual drag-and-drop interface lets data scientists and power users create machine learning solutions quickly, while supporting custom R and Python logic, a wide range of established statistical algorithms and techniques for machine learning modeling tasks, and built-in support for Jupyter Notebooks.</span></span>

<span data-ttu-id="bf95c-158">Principaux avantages :</span><span class="sxs-lookup"><span data-stu-id="bf95c-158">Key benefits:</span></span>

- <span data-ttu-id="bf95c-159">Interface visuelle interactive pour la modélisation Machine Learning avec un minimum de code.</span><span class="sxs-lookup"><span data-stu-id="bf95c-159">Interactive visual interface enables machine learning modeling with minimal code.</span></span>
- <span data-ttu-id="bf95c-160">Jupyter Notebooks intégrés pour l’exploration de données.</span><span class="sxs-lookup"><span data-stu-id="bf95c-160">Built-in Jupyter Notebooks for data exploration.</span></span>
- <span data-ttu-id="bf95c-161">Déploiement direct de modèles formés en tant que services web Azure.</span><span class="sxs-lookup"><span data-stu-id="bf95c-161">Direct deployment of trained models as Azure web services.</span></span>

<span data-ttu-id="bf95c-162">Considérations :</span><span class="sxs-lookup"><span data-stu-id="bf95c-162">Considerations:</span></span>

- <span data-ttu-id="bf95c-163">Évolutivité limitée.</span><span class="sxs-lookup"><span data-stu-id="bf95c-163">Limited scalability.</span></span> <span data-ttu-id="bf95c-164">La taille maximale d’un jeu de données d’apprentissage est de 10 Go.</span><span class="sxs-lookup"><span data-stu-id="bf95c-164">The maximum size of a training dataset is 10 GB.</span></span>
- <span data-ttu-id="bf95c-165">En ligne uniquement.</span><span class="sxs-lookup"><span data-stu-id="bf95c-165">Online only.</span></span> <span data-ttu-id="bf95c-166">Aucun environnement de développement en mode hors connexion.</span><span class="sxs-lookup"><span data-stu-id="bf95c-166">No offline development environment.</span></span>

## <a name="tools-and-services-for-deploying-machine-learning-models"></a><span data-ttu-id="bf95c-167">Outils et services pour le déploiement de modèles Machine Learning</span><span class="sxs-lookup"><span data-stu-id="bf95c-167">Tools and services for deploying machine learning models</span></span>

<span data-ttu-id="bf95c-168">Une fois qu’un scientifique de données a créé un modèle Machine Learning, vous devrez en général le déployer et le consommer à partir d’applications ou dans d’autres flux de données.</span><span class="sxs-lookup"><span data-stu-id="bf95c-168">After a data scientist has created a machine learning model, you will typically need to deploy it and consume it from applications or in other data flows.</span></span> <span data-ttu-id="bf95c-169">Il existe plusieurs cibles potentielles de déploiement pour les modèles Machine Learning.</span><span class="sxs-lookup"><span data-stu-id="bf95c-169">There are a number of potential deployment targets for machine learning models.</span></span>

### <a name="spark-on-azure-hdinsight"></a><span data-ttu-id="bf95c-170">Spark sur Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="bf95c-170">Spark on Azure HDInsight</span></span>

<span data-ttu-id="bf95c-171">Apache Spark inclut Spark MLlib, un framework et une bibliothèque de modèles Machine Learning.</span><span class="sxs-lookup"><span data-stu-id="bf95c-171">Apache Spark includes Spark MLlib, a framework and library for machine learning models.</span></span> <span data-ttu-id="bf95c-172">La bibliothèque Microsoft Machine Learning pour Spark (MMLSpark) fournit également la prise en charge d’un algorithme d’apprentissage profond pour les modèles prédictifs dans Spark.</span><span class="sxs-lookup"><span data-stu-id="bf95c-172">The Microsoft Machine Learning library for Spark (MMLSpark) also provides deep learning algorithm support for predictive models in Spark.</span></span>

<span data-ttu-id="bf95c-173">Principaux avantages :</span><span class="sxs-lookup"><span data-stu-id="bf95c-173">Key benefits:</span></span>

- <span data-ttu-id="bf95c-174">Spark est une plateforme distribuée qui offre une haute évolutivité pour les processus Machine Learning à volume élevé.</span><span class="sxs-lookup"><span data-stu-id="bf95c-174">Spark is a distributed platform that offers high scalability for high-volume machine learning processes.</span></span>
- <span data-ttu-id="bf95c-175">Vous pouvez déployer des modèles directement sur Spark dans HDinsight et les gérer à l’aide du service Gestion des modèles Azure Machine Learning.</span><span class="sxs-lookup"><span data-stu-id="bf95c-175">You can deploy models directly to Spark in HDinsight and manage them using the Azure Machine Learning Model Management service.</span></span>

<span data-ttu-id="bf95c-176">Considérations :</span><span class="sxs-lookup"><span data-stu-id="bf95c-176">Considerations:</span></span>

- <span data-ttu-id="bf95c-177">Spark s’exécute dans un cluster HDinsght qui entraîne des frais pendant toute la durée de son exécution.</span><span class="sxs-lookup"><span data-stu-id="bf95c-177">Spark runs in an HDinsght cluster that incurs charges the whole time it is running.</span></span> <span data-ttu-id="bf95c-178">Si le service Machine Learning ne sera utilisé qu’occasionnellement, cela peut entraîner des frais inutiles.</span><span class="sxs-lookup"><span data-stu-id="bf95c-178">If the machine learning service will only be used occasionally, this may result in unnecessary costs.</span></span>

### <a name="azure-databricks"></a><span data-ttu-id="bf95c-179">Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="bf95c-179">Azure Databricks</span></span>

<span data-ttu-id="bf95c-180">[Azure Databricks](/azure/azure-databricks/) est une plateforme d’analyse basée sur Apache Spark.</span><span class="sxs-lookup"><span data-stu-id="bf95c-180">[Azure Databricks](/azure/azure-databricks/) is an Apache Spark-based analytics platform.</span></span> <span data-ttu-id="bf95c-181">Celle-ci peut-être considérée comme « Spark en tant que service ».</span><span class="sxs-lookup"><span data-stu-id="bf95c-181">You can think of it as "Spark as a service."</span></span> <span data-ttu-id="bf95c-182">Il s’agit du moyen le plus simple d’utiliser Spark sur la plateforme Azure.</span><span class="sxs-lookup"><span data-stu-id="bf95c-182">It's the easiest way to use Spark on the Azure platform.</span></span> <span data-ttu-id="bf95c-183">Pour l’apprentissage, vous pouvez utiliser [MLFlow](https://www.mlflow.org/), [Databricks Runtime ML](https://docs.azuredatabricks.net/user-guide/clusters/mlruntime.html), Apache Spark MLlib et d’autres.</span><span class="sxs-lookup"><span data-stu-id="bf95c-183">For machine learning, you can use [MLFlow](https://www.mlflow.org/), [Databricks Runtime ML](https://docs.azuredatabricks.net/user-guide/clusters/mlruntime.html), Apache Spark MLlib, and others.</span></span> <span data-ttu-id="bf95c-184">Pour plus d’informations, consultez [Azure Databricks : Machine Learning](https://docs.azuredatabricks.net/spark/latest/mllib/index.html).</span><span class="sxs-lookup"><span data-stu-id="bf95c-184">For more information, see [Azure Databricks: Machine Learning](https://docs.azuredatabricks.net/spark/latest/mllib/index.html).</span></span>

### <a name="web-service-in-a-container"></a><span data-ttu-id="bf95c-185">Service web dans un conteneur</span><span class="sxs-lookup"><span data-stu-id="bf95c-185">Web service in a container</span></span>

<span data-ttu-id="bf95c-186">Vous pouvez déployer un modèle Machine Learning en tant que service web Python dans un conteneur Docker.</span><span class="sxs-lookup"><span data-stu-id="bf95c-186">You can deploy a machine learning model as a Python web service in a Docker container.</span></span> <span data-ttu-id="bf95c-187">Vous pouvez déployer le modèle sur Azure ou sur un appareil de périphérie, où il peut être utilisé localement avec les données sur lesquelles il opère.</span><span class="sxs-lookup"><span data-stu-id="bf95c-187">You can deploy the model to Azure or to an edge device, where it can be used locally with the data on which it operates.</span></span>

<span data-ttu-id="bf95c-188">Principaux avantages :</span><span class="sxs-lookup"><span data-stu-id="bf95c-188">Key Benefits:</span></span>

- <span data-ttu-id="bf95c-189">Les conteneurs sont légers et constituent un moyen généralement économique d’empaqueter et de déployer des services.</span><span class="sxs-lookup"><span data-stu-id="bf95c-189">Containers are a lightweight and generally cost effective way to package and deploy services.</span></span>
- <span data-ttu-id="bf95c-190">La capacité à déployer sur un appareil de périphérie vous permet de déplacer votre logique prédictive plus proche des données.</span><span class="sxs-lookup"><span data-stu-id="bf95c-190">The ability to deploy to an edge device enables you to move your predictive logic closer to the data.</span></span>

<span data-ttu-id="bf95c-191">Considérations :</span><span class="sxs-lookup"><span data-stu-id="bf95c-191">Considerations:</span></span>

- <span data-ttu-id="bf95c-192">Ce modèle de déploiement est basé sur des conteneurs Docker, donc vous devez être familiarisé avec cette technologie avant de déployer un service web de cette manière.</span><span class="sxs-lookup"><span data-stu-id="bf95c-192">This deployment model is based on Docker containers, so you should be familiar with this technology before deploying a web service this way.</span></span>

### <a name="microsoft-machine-learning-server"></a><span data-ttu-id="bf95c-193">Microsoft Machine Learning Server</span><span class="sxs-lookup"><span data-stu-id="bf95c-193">Microsoft Machine Learning Server</span></span>

<span data-ttu-id="bf95c-194">Machine Learning Server (anciennement Microsoft R Server) est une plateforme évolutive pour le code R et Python, spécifiquement conçue pour les scénarios Machine Learning.</span><span class="sxs-lookup"><span data-stu-id="bf95c-194">Machine Learning Server (formerly Microsoft R Server) is a scalable platform for R and Python code, specifically designed for machine learning scenarios.</span></span>

<span data-ttu-id="bf95c-195">Principaux avantages :</span><span class="sxs-lookup"><span data-stu-id="bf95c-195">Key benefits:</span></span>

- <span data-ttu-id="bf95c-196">Évolutivité élevée.</span><span class="sxs-lookup"><span data-stu-id="bf95c-196">High scalability.</span></span>

<span data-ttu-id="bf95c-197">Considérations :</span><span class="sxs-lookup"><span data-stu-id="bf95c-197">Considerations:</span></span>

- <span data-ttu-id="bf95c-198">Vous devez déployer et gérer Machine Learning Server dans votre entreprise.</span><span class="sxs-lookup"><span data-stu-id="bf95c-198">You need to deploy and manage Machine Learning Server in your enterprise.</span></span>

### <a name="microsoft-sql-server"></a><span data-ttu-id="bf95c-199">Microsoft SQL Server</span><span class="sxs-lookup"><span data-stu-id="bf95c-199">Microsoft SQL Server</span></span>

<span data-ttu-id="bf95c-200">Microsoft SQL Server prend en charge R et Python en mode natif, ce qui vous permet d’encapsuler des modèles Machine Learning générés dans ces langages en tant que fonctions Transact-SQL dans une base de données.</span><span class="sxs-lookup"><span data-stu-id="bf95c-200">Microsoft SQL Server supports R and Python natively, enabling you to encapsulate machine learning models built in these languages as Transact-SQL functions in a database.</span></span>

<span data-ttu-id="bf95c-201">Principaux avantages :</span><span class="sxs-lookup"><span data-stu-id="bf95c-201">Key benefits:</span></span>

- <span data-ttu-id="bf95c-202">Encapsuler la logique prédictive dans une fonction de base de données, ce qui permet d’inclure facilement dans une logique de couche de données.</span><span class="sxs-lookup"><span data-stu-id="bf95c-202">Encapsulate predictive logic in a database function, making it easy to include in data-tier logic.</span></span>

<span data-ttu-id="bf95c-203">Considérations :</span><span class="sxs-lookup"><span data-stu-id="bf95c-203">Considerations:</span></span>

- <span data-ttu-id="bf95c-204">Considère une base de données SQL Server en tant que couche de données pour votre application.</span><span class="sxs-lookup"><span data-stu-id="bf95c-204">Assumes a SQL Server database as the data tier for your application.</span></span>

### <a name="azure-machine-learning-web-service"></a><span data-ttu-id="bf95c-205">Service web Microsoft Azure Machine Learning</span><span class="sxs-lookup"><span data-stu-id="bf95c-205">Azure Machine Learning web service</span></span>

<span data-ttu-id="bf95c-206">Lorsque vous créez un modèle Machine Learning à l’aide d’Azure Machine Learning Studio, vous pouvez le déployer en tant que service web.</span><span class="sxs-lookup"><span data-stu-id="bf95c-206">When you create a machine learning model using Azure Machine Learning Studio, you can deploy it as a web service.</span></span> <span data-ttu-id="bf95c-207">Il peut ensuite être consommé via une interface REST à partir de toute application cliente capable de communiquer via le protocole HTTP.</span><span class="sxs-lookup"><span data-stu-id="bf95c-207">This can then be consumed through a REST interface from any client applications capable of communicating by HTTP.</span></span>

<span data-ttu-id="bf95c-208">Principaux avantages :</span><span class="sxs-lookup"><span data-stu-id="bf95c-208">Key benefits:</span></span>

- <span data-ttu-id="bf95c-209">Facilité de déploiement et de développement.</span><span class="sxs-lookup"><span data-stu-id="bf95c-209">Ease of development and deployment.</span></span>
- <span data-ttu-id="bf95c-210">Portail de gestion de service web avec métriques de monitoring de base.</span><span class="sxs-lookup"><span data-stu-id="bf95c-210">Web service management portal with basic monitoring metrics.</span></span>
- <span data-ttu-id="bf95c-211">Prise en charge intégrée pour appeler les services web Azure Machine Learning à partir d’Azure Data Lake Analytics, Azure Data Factory et Azure Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="bf95c-211">Built-in support for calling Azure Machine Learning web services from Azure Data Lake Analytics, Azure Data Factory, and Azure Stream Analytics.</span></span>

<span data-ttu-id="bf95c-212">Considérations :</span><span class="sxs-lookup"><span data-stu-id="bf95c-212">Considerations:</span></span>

- <span data-ttu-id="bf95c-213">Disponible uniquement pour les modèles générés à l’aide d’Azure Machine Learning Studio.</span><span class="sxs-lookup"><span data-stu-id="bf95c-213">Only available for models built using Azure Machine Learning Studio.</span></span>
- <span data-ttu-id="bf95c-214">Accès web uniquement, les modèles formés ne peuvent pas s’exécuter sur site ou hors connexion.</span><span class="sxs-lookup"><span data-stu-id="bf95c-214">Web-based access only, trained models cannot run on-premises or offline.</span></span>
