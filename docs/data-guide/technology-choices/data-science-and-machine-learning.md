---
title: Sélectionner une technologie Machine Learning
description: Comparez les options de création, de déploiement et de gestion de vos modèles Machine Learning. Décidez quels produits Microsoft choisir pour votre solution.
author: MikeWasson
ms.date: 03/06/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: 1020e938a04c6a82e6cc831e6620ec17c9a10cc7
ms.sourcegitcommit: 9854bd27fb5cf92041bbfb743d43045cd3552a69
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/27/2019
ms.locfileid: "58503228"
---
# <a name="what-are-the-machine-learning-products-at-microsoft"></a>Quels sont les produits de machine learning proposés par Microsoft ?

Machine Learning est une technique de science des données qui permet aux ordinateurs d’utiliser des données existantes afin de prévoir les tendances, les résultats et les comportements futurs. Grâce au machine learning, les ordinateurs apprennent sans programmation explicite.

Solutions machine learning sont générées de manière itérative et ont des phases distinctes :

- Préparation des données
- Expérimentation et apprentissage de modèles
- Déploiement de modèles formés
- La gestion déployé des modèles

Microsoft fournit un éventail d’options de produit pour la préparation, générer, déployer et gérer vos modèles machine learning. Comparez ces produits et choisissez celui qu’il vous faut pour optimiser le développement de vos solutions de machine learning.

## <a name="cloud-based-options"></a>Options de cloud

Les options suivantes sont disponibles pour le machine learning dans le cloud Azure.

| Options&nbsp;cloud | Présentation | Fonctionnalités |
|-|-|-|
| [Service Azure Machine Learning](#azure-machine-learning-service) | Service cloud géré pour l’apprentissage  | Entraîner, déployer et gérer des modèles dans Azure à l’aide de Python et de l’interface CLI |
| [Azure Machine Learning Studio](#azure-machine-learning-studio) | Faites glisser&ndash;et&ndash;interface visuelle de dépôt pour l’apprentissage | Générer, tester et déployer des modèles à l’aide d’algorithmes préconfigurés |

Si vous souhaitez utiliser préconçues intelligence artificielle et les modèles machine learning, [Azure Cognitive Services](#azure-cognitive-services) vous permet d’ajouter facilement des fonctionnalités intelligentes à vos applications.

## <a name="on-premises-options"></a>Options en local

Les options suivantes sont disponibles pour le machine learning local. Les serveurs locaux peuvent également être exécutés sur une machine virtuelle dans le cloud.

| Options&nbsp;locales | Présentation | Fonctionnalités |
|-|-|-|
| [Services de Machine Learning SQL Server](#sql-server-machine-learning-services) | Moteur d’analytique incorporé à SQL | Générer et déployer des modèles dans SQL Server |
| [Microsoft Machine Learning Server](#microsoft-machine-learning-server) | Serveur d’entreprise autonome pour l’analyse prédictive | Générer et déployer des modèles sur des données prétraitées |

## <a name="development-platforms-and-tools"></a>Outils et plateformes de développement

Les outils et les plateformes de développement suivantes sont disponibles pour l’apprentissage.

| Plateformes/tools | Présentation | Fonctionnalités |
|-|-|-|
| [Azure Data Science Virtual Machine](#azure-data-science-virtual-machine) | Machine virtuelle avec des outils de science des données préinstallés | Développer des solutions machine learning dans un environnement préconfiguré |
| [Azure Databricks](#azure-databricks) | Plateforme d’analytique basée sur Spark | Générer et déployer des modèles et des workflows de données |
| [ML.NET](#mlnet) | Open source, multiplateforme apprentissage SDK | Développer des solutions machine learning pour les applications .NET |
| [Windows ML](#windows-ml) | Plateforme d’apprentissage automatique Windows 10 | Évaluer des modèles entraînés sur un appareil Windows 10 |

## <a name="azure-machine-learning-service"></a>Service Azure Machine Learning

[Le service Azure Machine Learning](/azure/machine-learning/service/overview-what-is-azure-ml.md) est un service cloud entièrement géré utilisé pour former, déployer et gérer des modèles d’apprentissage automatique à grande échelle. Il prend entièrement en charge les technologies open source. Vous pouvez donc utiliser des dizaines de milliers de packages Python open source tels que TensorFlow, PyTorch et scikit-learn. Avec des outils puissants comme [Azure Notebooks](https://notebooks.azure.com/), [Jupyter Notebook](http://jupyter.org) ou l’extension [Azure Machine Learning pour Visual Studio Code](https://aka.ms/vscodetoolsforai), vous pouvez facilement explorer et transformer des données, puis entraîner et déployer des modèles. Le service Azure Machine Learning comprend des fonctionnalités qui automatisent la génération de modèles et qui vous permettent de les régler de manière simple, efficace et précise.

Utiliser le service Azure Machine Learning pour former, déployer et gérer des modèles d’apprentissage automatique à l’aide de Python et l’interface CLI à l’échelle du cloud.

Essayez la [version gratuite ou payante d’Azure Machine Learning service](http://aka.ms/AMLFree).

|||
|-|-|
|**Type**                   |Basé sur le cloud solution d’apprentissage|
|**Langues prises en charge**    |Python|
|**Phases de machine learning**|Préparation des données<br>Apprentissage du modèle<br>Déploiement<br>gestion|
|**Principaux avantages**           |Gestion centralisée des scripts et historique d’exécution, ce qui facilite la comparaison des versions du modèle.<br/><br/>Déploiement et gestion simples des modèles sur le cloud ou sur des appareils de périphérie.|
|**Considérations**         |Nécessite des connaissances relatives au modèle de Gestion des modèles.|

## <a name="azure-machine-learning-studio"></a>Azure Machine Learning Studio

[Azure Machine Learning Studio](/azure/machine-learning/studio/) est un espace de travail visuel et interactif que vous pouvez utiliser pour générer, tester et déployer avec facilité et rapidité des modèles à l’aide d’algorithmes de machine learning prédéfinis. Machine Learning Studio publie des modèles en tant que services web pouvant facilement être consommés par des applications personnalisées ou des outils décisionnels tels qu'Excel.
Aucune programmation n’est requise : vous construisez votre modèle Machine Learning en connectant des jeux de données et des modules d’analyse sur un canevas interactif, puis vous le déployez en quelques clics.

Utilisez Machine Learning Studio quand vous souhaitez développer et déployer des modèles sans écrire de code.

Essayez [Azure Machine Learning Studio](https://studio.azureml.net/?selectAccess=true&o=2&target=_blank), disponible en version payante ou gratuite.

|||
|-|-|
|**Type**                   |Basé sur le cloud, glissez-déposez solution d’apprentissage|
|**Langues prises en charge**    |Python, R|
|**Phases de machine learning**|Préparation des données<br>Apprentissage du modèle<br>Déploiement<br>gestion|
|**Principaux avantages**           |Interface visuelle interactive pour la modélisation Machine Learning avec un minimum de code.<br/><br/>Jupyter Notebooks intégrés pour l’exploration de données.<br/><br/>Déploiement direct de modèles formés en tant que services web Azure.|
|**Considérations**         |Évolutivité limitée. La taille maximale d’un jeu de données d’apprentissage est de 10 Go.<br/><br/>En ligne uniquement. Aucun environnement de développement en mode hors connexion.|

## <a name="azure-cognitive-services"></a>Azure Cognitive Services

[Azure Cognitive Services](/azure/cognitive-services/welcome) regroupe un ensemble d’API permettant de créer des applications qui utilisent des méthodes de communication naturelles. Ces API permettent à vos applications de voir, d’entendre, de parler, de comprendre et d’interpréter les besoins des utilisateurs avec seulement quelques lignes de code. Ajoutez facilement des caractéristiques intelligentes à vos applications, par exemple :

- Émotion et détection de sentiments
- Vision et reconnaissance vocale
- Language understanding (LUIS)
- Connaissances et recherche

Utilisez Cognitive Services pour développer des applications sur divers appareils et plateformes. Les API ne cessent de s’améliorer et sont faciles à configurer.

|||
|-|-|
|**Type**                   |API permettant de créer des applications intelligentes|
|**Langues prises en charge**    |nombreuses options en fonction du service|
|**Phases de machine learning**|Déploiement|
|**Principaux avantages**           |Incorporation des fonctionnalités d’apprentissage automatique dans les applications à l’aide de modèles préentraînés.<br/><br/>Choix de modèles pour les méthodes de communication naturelle avec visuelle et vocale.|
|**Considérations**         |Les modèles ont été préformés et ne sont pas personnalisables.|

## <a name="sql-server-machine-learning-services"></a>Services de Machine Learning SQL Server

[SQL Server Microsoft Machine Learning Services](https://docs.microsoft.com/sql/advanced-analytics/r/r-services) ajoute l’analyse statistique, la visualisation des données et l’analytique prédictive dans R et Python pour les données relationnelles dans les bases de données SQL Server. Les bibliothèques R et Python à partir de Microsoft incluent les avancées de modélisation et les algorithmes d’apprentissage automatique, qui peuvent s’exécuter en parallèle et à grande échelle, dans SQL Server.

Utilisez SQL Server Machine Learning Services quand vous souhaitez bénéficier d’une intelligence artificielle intégrée et d’une analytique prédictive des données relationnelles dans SQL Server.

|||
|-|-|
|**Type**                   |Analytique prédictive pour les données relationnelles locales|
|**Langues prises en charge**    |Python, R|
|**Phases de machine learning**|Préparation des données<br>Apprentissage du modèle<br>Déploiement|
|**Principaux avantages**           |Encapsuler la logique prédictive dans une fonction de base de données, ce qui permet d’inclure facilement dans une logique de couche de données.|
|**Considérations**         |Considère une base de données SQL Server en tant que couche de données pour votre application.|

## <a name="microsoft-machine-learning-server"></a>Microsoft Machine Learning Server

[Microsoft Machine Learning Server](https://docs.microsoft.com/machine-learning-server/what-is-machine-learning-server) est un serveur d’entreprise pour l’hébergement et la gestion des charges de travail parallèles et distribuées des processus R et Python. Microsoft Machine Learning Server s’exécute sur Linux, Windows, Hadoop et Apache Spark. Il est également disponible sur [HDInsight](https://azure.microsoft.com/services/hdinsight/r-server/). Il fournit un moteur d’exécution pour les solutions créées à l’aide des packages [RevoScaleR](https://docs.microsoft.com/machine-learning-server/r-reference/revoscaler/revoscaler), [revoscalepy](https://docs.microsoft.com/machine-learning-server/python-reference/revoscalepy/revoscalepy-package) et [MicrosoftML](https://docs.microsoft.com/r-server/r/concept-what-is-the-microsoftml-package). Il étend également le code R et Python open source en prenant en charge l’analytique hautes performances, l’analyse statistique, le machine learning et les jeux de données volumineux. Cette fonctionnalité est fournie par le biais de packages propriétaires qui sont installés avec le serveur. Pour le développement, vous pouvez utiliser comme IDE [Outils R pour Visual Studio](https://www.visualstudio.com/vs/rtvs/) et [Python Tools pour Visual Studio](https://www.visualstudio.com/vs/python/).

Utilisez Microsoft Machine Learning Server pour créer des modèles créés avec R et Python sur un serveur et les rendre opérationnels, ou distribuer un entraînement R et Python à grande échelle sur un cluster Hadoop ou Spark.

|||
|-|-|
|**Type**                   |Serveur d’entreprise en local pour l’analytique prédictive|
|**Langues prises en charge**    |Python, R|
|**Phases de machine learning**|Apprentissage du modèle<br>Déploiement|
|**Principaux avantages**           |Évolutivité élevée.|
|**Considérations**         |Vous devez déployer et gérer Machine Learning Server dans votre entreprise.|

## <a name="azure-data-science-virtual-machine"></a>Machine virtuelle DSVM (Data Science Virtual Machine) Azure

[Azure Data Science Virtual Machine](https://docs.microsoft.com/azure/machine-learning/data-science-virtual-machine/overview) est un environnement de machine virtuelle personnalisé sur le cloud Microsoft Azure qui est spécialement conçu pour la science des données. Elle inclut de nombreux outils de science des données populaires et d’autres outils sont préinstallés et préconfigurés afin d’accélérer la création d’applications intelligentes à des fins d’analyse avancée.

Data Science Virtual Machine est pris en charge en tant que cible pour le service Azure Machine Learning.
Elle est disponible dans des versions pour Windows et Linux Ubuntu (le service Azure Machine Learning n’est pas pris en charge sur Linux CentOS).
Pour obtenir des informations de version spécifiques et la liste des fonctionnalités incluses, consultez [Présentation d’Azure Data Science Virtual Machine](/azure/machine-learning/data-science-virtual-machine/overview.md).

Utilisez la machine virtuelle DSVM quand vous devez exécuter ou héberger vos tâches sur un même nœud, ou si vous devez monter en puissance à distance le traitement sur un seul ordinateur.

|||
|-|-|
|**Type**                   |Environnement de machine virtuelle personnalisée pour la science des données|
|**Principaux avantages**           |Réduction du temps d’installation, de gestion et de résolution des problèmes liés aux outils et aux frameworks de science des données.<br/><br/>Les dernières versions de tous les outils et frameworks couramment utilisés sont incluses.<br/><br/>Les options de machine virtuelle incluent des images hautement évolutives avec des fonctionnalités GPU pour la modélisation intensive des données.|
|**Considérations**         |La machine virtuelle n’est pas accessible en mode hors connexion.<br/><br/>L’exécution d’une machine virtuelle entraîne des frais Azure, donc vous devez veiller à l’exécuter uniquement si nécessaire.|

## <a name="azure-databricks"></a>Azure Databricks

[Azure Databricks](/azure/azure-databricks/what-is-azure-databricks) est une plateforme d’analytique basée sur Apache Spark et optimisée pour la plateforme de services cloud Microsoft Azure. Databricks est intégré à Azure pour offrir une configuration en un clic, des workflows simplifiés et un espace de travail interactif permettant aux scientifiques des données, aux ingénieurs des données et aux analystes métier de collaborer.
Utilisez du code Python, R, Scala et SQL dans des notebooks basés sur le web pour interroger, visualiser et modéliser des données.

Utilisez Databricks quand vous souhaitez travailler à plusieurs sur la conception de solutions de machine learning sur Apache Spark.

|||
|-|-|
|**Type**                   |Plateforme d’analytique basée sur Apache Spark|
|**Langues prises en charge**    |Python, R, Scala, SQL|
|**Phases de machine learning**|Requête de données<br>Apprentissage du modèle|

## <a name="mlnet"></a>ML.NET

[ML.NET](https://docs.microsoft.com/dotnet/machine-learning/) est un framework de machine learning gratuit, open source et multiplateforme qui vous permet de créer des solutions de machine learning personnalisées et de les intégrer à vos applications .NET.

Utilisez ML.NET quand vous avez besoin d’intégrer des solutions de machine learning à vos applications .NET.

|||
|-|-|
|**Type**                   |Infrastructure open source pour le développement d’applications d’apprentissage automatique personnalisé|
|**Langues prises en charge**    |.NET|

## <a name="windows-ml"></a>Windows ML

[Windows ML](https://docs.microsoft.com/windows/uwp/machine-learning/) moteur d’inférence vous permet d’utiliser la machine entraînée modèles dans vos applications d’apprentissage, l’évaluation des modèles formés localement sur les appareils Windows 10.

Utilisez Windows ML quand vous avez besoin d’utiliser des modèles Machine Learning entraînés dans vos applications Windows.

|||
|-|-|
|**Type**                   |Moteur d’inférence pour les modèles formés sur les appareils Windows|
|**Langues prises en charge**    |C#/C++, JavaScript|

## <a name="next-steps"></a>Étapes suivantes

- Pour découvrir l’ensemble des produits de développement Microsoft axés sur l’intelligence artificielle, consultez [Plateforme Microsoft AI](https://www.microsoft.com/ai)
- Pour apprendre à développer des solutions d’intelligence artificielle, consultez [Microsoft AI School](https://aischool.microsoft.com/learning-paths)
