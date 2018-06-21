---
title: Sélectionner une technologie Machine Learning
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 995349c795066ec3067b20ad2615e40b0fb152db
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/14/2018
ms.locfileid: "29288931"
---
# <a name="choosing-a-machine-learning-technology-in-azure"></a>Sélectionner une technologie Machine Learning dans Azure

La science des données et Machine Learning est une charge de travail généralement effectuée par les scientifiques de données. Elle requiert des outils spécialisés, qui sont conçus spécifiquement pour le type de tâches d’exploration interactive des données et de modélisation qu’un scientifique de données doit effectuer.

Les solutions Machine Learning sont générées de manière itérative et comportent deux phases distinctes :
* Préparation et modélisation des données.
* Déploiement et consommation des services prédictifs.

## <a name="tools-and-services-for-data-preparation-and-modeling"></a>Outils et services pour la préparation et la modélisation des données
En général, les scientifiques de données préfèrent travailler avec des données utilisant un code personnalisé écrit dans Python ou R. Ce code est généralement exécuté de manière interactive. Les scientifiques de données l’utilise pour interroger et explorer les données, générer des visualisations et des statistiques afin de déterminer les relations avec le code. Il existe de nombreux environnements interactifs pour R et Python que les scientifiques de données peuvent utiliser. L’un des favoris, **Jupyter Notebooks**, fournit un shell basé sur navigateur qui permet aux scientifiques de données de créer des fichiers de *bloc-notes* qui contiennent du code R ou Python et du texte Markdown. Cela constitue un moyen efficace de collaborer en partageant et en documentant le code et des résultats dans un seul document.

Autres outils couramment utilisés :
* **Spyder** : environnement de développement interactif (IDE) pour Python fourni avec la distribution Anaconda Python.
* **R Studio** : IDE pour le langage de programmation R.
* **Visual Studio Code** : environnement de codage léger et multiplateforme qui prend en charge Python ainsi que les frameworks couramment utilisés pour Machine Learning et le développement IA.

Outre ces outils, les scientifiques de données peuvent tirer parti des services Azure pour simplifier la gestion du code et du modèle.

### <a name="azure-notebooks"></a>Azure Notebooks
Azure Notebooks est un service Jupyter Notebooks en ligne qui permet aux scientifiques de données de créer, d’exécuter et de partager Jupyter Notebooks dans des bibliothèques cloud.

Principaux avantages :

* Service gratuit&mdash;pas d’abonnement Azure requis.
* Inutile d’installer Jupyter et les distributions R ou Python connexes localement&mdash;utilisez simplement un navigateur.
* Gérez vos propres bibliothèques en ligne et accédez-y sur n’importe quel appareil.
* Partagez vos blocs-notes avec des collaborateurs.

Considérations :

* Vous ne pourrez pas accéder à vos blocs-notes en mode hors connexion.
* Les capacités de traitement limitées du service de bloc-notes gratuit peuvent ne pas suffire pour l’apprentissage de modèles volumineux ou complexes.

### <a name="data-science-virtual-machine"></a>Machine virtuelle de science des données
La machine virtuelle de science des données est une image de machine virtuelle Azure qui inclut les outils et les frameworks couramment utilisés par les scientifiques de données, notamment R, Python, Jupyter Notebooks, Visual Studio Code, et les bibliothèques pour la modélisation Machine Learning comme Microsoft Cognitive Toolkit. Ces outils peuvent être complexes et longs à installer, et contiennent de nombreuses interdépendances qui mènent souvent à des problèmes de gestion de version. Le fait de disposer d’une image pré-installée peut réduire le temps que les scientifiques de données passent à résoudre les problèmes d’environnement, ce qui leur permet de se concentrer sur les tâches d’exploration et de modélisation des données qu’ils doivent exécuter.

Principaux avantages :
* Réduction du temps d’installation, de gestion et de résolution des problèmes liés aux outils et aux frameworks de science des données.
* Les dernières versions de tous les outils et frameworks couramment utilisés sont incluses.
* Les options de machine virtuelle incluent des images hautement évolutives avec des fonctionnalités GPU pour la modélisation intensive des données.

Considérations :
* La machine virtuelle n’est pas accessible en mode hors connexion.
* L’exécution d’une machine virtuelle entraîne des frais Azure, donc vous devez veiller à l’exécuter uniquement si nécessaire.

### <a name="azure-machine-learning"></a>Azure Machine Learning

Azure Machine Learning est un service cloud pour la gestion des modèles et des expériences Machine Learning. Il inclut un service d’expérimentation qui effectue le suivi des scripts d’apprentissage de préparation et de modélisation des données, en conservant un historique de toutes les exécutions, qui vous permet de comparer les performances du modèle entre les itérations. Un outil client multiplateforme nommé Azure Machine Learning Workbench fournit une interface centrale pour la gestion et l’historique des scripts, tout en permettant aux scientifiques de données de créer des scripts dans leur outil préféré, comme Jupyter Notebooks ou Visual Studio Code.

Dans Azure Machine Learning Workbench, vous pouvez utiliser les outils interactifs de préparation des données pour simplifier les tâches courantes de transformation des données, et vous pouvez configurer l’environnement d’exécution du script pour exécuter localement des scripts de formation du modèle, dans un conteneur Docker évolutif, ou dans Spark.

Lorsque vous êtes prêt à déployer votre modèle, utilisez l’environnement Workbench pour packager le modèle et le déployer en tant que service web dans un conteneur Docker, Spark sur Azure HDinsight, Microsoft Machine Learning Server ou SQL Server. Le service Gestion des modèles Azure Machine Learning vous permet d’effectuer le suivi et de gérer les déploiements de modèle dans le cloud, sur des appareils de périphérie ou à l’échelle de l’entreprise.

Principaux avantages :

* Gestion centralisée des scripts et historique d’exécution, ce qui facilite la comparaison des versions du modèle.
* Transformation interactive des données via un éditeur visuel.
* Déploiement et gestion simples des modèles sur le cloud ou sur des appareils de périphérie.

Considérations :
* Nécessite des connaissances sur le modèle de gestion de modèle et l’environnement de l’outil Workbench.

### <a name="azure-batch-ai"></a>Azure Batch AI

Azure Batch AI vous permet d’exécuter vos expériences Machine Learning en parallèle et d’effectuer l’apprentissage du modèle à l’échelle sur un cluster de machines virtuelles avec des GPU. Batch AI Training vous permet de monter en charge des tâches d’apprentissage profond sur des GPU en cluster, en utilisant des frameworks comme Cognitive Toolkit, Caffe, Chainer et TensorFlow. 

Le service Gestion des modèles Azure Machine Learning peut servir à retirer des modèles de Batch AI Training pour les déployer, gérer et surveiller. 

### <a name="azure-machine-learning-studio"></a>Azure Machine Learning Studio

Azure Machine Learning Studio est un environnement de développement visuel, basé sur le cloud, pour la création d’expériences de données, la formation de modèles Machine Learning et leur publication en tant que services web dans Azure. Son interface de type glisser-déposer visuelle permet aux scientifiques de données et aux utilisateurs avancés de créer rapidement des solutions Machine Learning, tout en prenant en charge la logique R et Python personnalisée, une grande variété de techniques et d’algorithmes statistiques établis pour les tâches de modélisation Machine Learning, et la prise en charge intégrée de Jupyter Notebooks.

Principaux avantages :

* Interface visuelle interactive pour la modélisation Machine Learning avec un minimum de code.
* Jupyter Notebooks intégrés pour l’exploration de données.
* Déploiement direct de modèles formés en tant que services web Azure.

Considérations :

* Évolutivité limitée. La taille maximale d’un jeu de données d’apprentissage est de 10 Go.
* En ligne uniquement. Aucun environnement de développement en mode hors connexion.

## <a name="tools-and-services-for-deploying-machine-learning-models"></a>Outils et services pour le déploiement de modèles Machine Learning

Une fois qu’un scientifique de données a créé un modèle Machine Learning, vous devrez en général le déployer et le consommer à partir d’applications ou dans d’autres flux de données. Il existe plusieurs cibles potentielles de déploiement pour les modèles Machine Learning.

### <a name="spark-on-azure-hdinsight"></a>Spark sur Azure HDInsight

Apache Spark inclut Spark MLlib, un framework et une bibliothèque de modèles Machine Learning. La bibliothèque Microsoft Machine Learning pour Spark (MMLSpark) fournit également la prise en charge d’un algorithme d’apprentissage profond pour les modèles prédictifs dans Spark.

Principaux avantages :

* Spark est une plateforme distribuée qui offre une haute évolutivité pour les processus Machine Learning à volume élevé.
* Vous pouvez déployer des modèles directement sur Spark dans HDinsight à partir d’Azure Machine Learning Workbench et les gérer à l’aide du service Gestion des modèles Azure Machine Learning.

Considérations :

* Spark s’exécute dans un cluster HDinsght qui entraîne des frais pendant toute la durée de son exécution. Si le service Machine Learning ne sera utilisé qu’occasionnellement, cela peut entraîner des frais inutiles.

### <a name="web-service-in-a-container"></a>Service web dans un conteneur

Vous pouvez déployer un modèle Machine Learning en tant que service web Python dans un conteneur Docker. Vous pouvez déployer le modèle sur Azure ou sur un appareil de périphérie, où il peut être utilisé localement avec les données sur lesquelles il opère.

Principaux avantages :

* Les conteneurs sont légers et constituent un moyen généralement économique d’empaqueter et de déployer des services.
* La capacité à déployer sur un appareil de périphérie vous permet de déplacer votre logique prédictive plus proche des données.
* Vous pouvez déployer sur un conteneur directement à partir d’Azure Machine Learning Workbench.

Considérations :

* Ce modèle de déploiement est basé sur des conteneurs Docker, donc vous devez être familiarisé avec cette technologie avant de déployer un service web de cette manière.

### <a name="microsoft-machine-learning-server"></a>Microsoft Machine Learning Server

Machine Learning Server (anciennement Microsoft R Server) est une plateforme évolutive pour le code R et Python, spécifiquement conçue pour les scénarios Machine Learning.

Principaux avantages :

* Évolutivité élevée.
* Déploiement direct à partir d’Azure Machine Learning Workbench.

Considérations :

* Vous devez déployer et gérer Machine Learning Server dans votre entreprise.

### <a name="microsoft-sql-server"></a>Microsoft SQL Server

Microsoft SQL Server prend en charge R et Python en mode natif, ce qui vous permet d’encapsuler des modèles Machine Learning générés dans ces langages en tant que fonctions Transact-SQL dans une base de données.

Principaux avantages :

* Encapsuler la logique prédictive dans une fonction de base de données, ce qui permet d’inclure facilement dans une logique de couche de données.

Considérations :

* Considère une base de données SQL Server en tant que couche de données pour votre application.

### <a name="azure-machine-learning-web-service"></a>Service web Microsoft Azure Machine Learning

Lorsque vous créez un modèle Machine Learning à l’aide d’Azure Machine Learning Studio, vous pouvez le déployer en tant que service web. Il peut ensuite être consommé via une interface REST à partir de toute application cliente capable de communiquer via le protocole HTTP.

Principaux avantages :

* Facilité de déploiement et de développement.
* Portail de gestion de service web avec métriques de monitoring de base.
* Prise en charge intégrée pour appeler les services web Azure Machine Learning à partir d’Azure Data Lake Analytics, Azure Data Factory et Azure Stream Analytics.

Considérations :

* Disponible uniquement pour les modèles générés à l’aide d’Azure Machine Learning Studio.
* Accès web uniquement, les modèles formés ne peuvent pas s’exécuter sur site ou hors connexion.

