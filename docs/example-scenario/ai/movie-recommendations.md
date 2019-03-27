---
title: Suggestions de films sur Azure
description: Automatisez des suggestions de films, de produits et autres en utilisant le machine learning et une instance Azure Data Science Virtual Machine (DSVM) pour entraîner un modèle sur Azure.
author: njray
ms.date: 01/09/2019
ms.custom: azcat-ai, AI
social_image_url: /azure/architecture/example-scenario/ai/media/architecture-movie-recommender.png
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.openlocfilehash: be7959c1201e3a2aaf92c192d73a0ca47a990934
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58249644"
---
# <a name="movie-recommendations-on-azure"></a>Suggestions de films sur Azure

Cet exemple de scénario montre comment une entreprise peut utiliser le machine learning pour automatiser des suggestions de produits pour ses clients. Une instance Azure DSVM est employée pour entraîner un modèle sur Azure qui suggère des films aux utilisateurs en fonction des notes qui ont été données sur les films.

Les suggestions peuvent être utiles dans de nombreux secteurs d’activité, par exemple dans les commerces, les infos et les médias. Par exemple, vous pouvez fournir des suggestions de produits dans un magasin virtuel, des suggestions d’articles d’actualité ou de posts, ou des suggestions de musique. Avant, les entreprises devaient employer et former des assistants pour rédiger des recommandations personnalisées à l’intention des clients. Aujourd’hui, nous pouvons fournir des suggestions personnalisées à grande échelle en nous servant d’Azure pour entraîner des modèles qui comprennent les préférences des clients.

## <a name="relevant-use-cases"></a>Cas d’usage appropriés

Pensez à ce scénario pour les cas d’usage suivants :

* Suggestions de films sur un site web.
* Suggestions de biens de consommation dans une application mobile.
* Suggestions d’articles d’actualité sur des médias de streaming.

## <a name="architecture"></a>Architecture

![Architecture d’un modèle Machine Learning pour l’entraînement de recommandations sur des films][architecture]

Ce scénario couvre l’entraînement et l’évaluation du modèle Machine Learning à l’aide de l’algorithme Spark [des moindres carrés alternés][als] (ALS) sur un jeu de données de notes de films. Les étapes de ce scénario sont les suivantes :

1. Le service d’application ou le site web front-end collecte les données de l’historique des interactions utilisateurs-films qui sont représentées dans une table de tuples de classement numérique, d’éléments et d’utilisateurs.

2. Les données historiques collectées sont stockées dans un stockage blob.

3. Une DSVM est souvent utilisée pour faire des essais ou mettre en production un modèle de système de recommandations ALS Spark. Le modèle ALS est entraîné au moyen d’un jeu de données d’entraînement, lui-même produit à partir du jeu de données global, en appliquant la stratégie appropriée de division des données. Par exemple, le jeu de données peut être divisé en plusieurs jeux de façon aléatoire, chronologique ou stratifiée, en fonction des exigences de l’entreprise. Comme pour les autres tâches de machine learning, un système de recommandations est validé en utilisant des métriques d’évaluation (par exemple, précision\@*k*, rappel\@*k*, [moyenne de la précision moyenne (MAP)][map], [nDCG\@k][ndcg]).

4. Azure Machine Learning service sert à coordonner l’expérimentation, comme le balayage d’hyperparamètres et la gestion du modèle.

5. Un modèle entraîné est conservé sur Azure Cosmos DB et peut ensuite être appliqué pour suggérer les *k* meilleurs films à un utilisateur donné.

6. Le modèle est alors déployé sur un service web ou un service d’application par le biais d’Azure Container Instances ou d’Azure Kubernetes Service.

Pour des instructions détaillées sur la création et la mise à l’échelle d’un service de recommandations, consultez [Créer une API de recommandation en temps réel sur Azure][ref-arch].

### <a name="components"></a>Composants

* [Data Science Virtual Machine][dsvm] (DSVM) est une machine virtuelle Azure dotée d’outils et de frameworks de deep learning pour le machine learning et la science des données. La machine virtuelle DSVM dispose d’un environnement Spark autonome qui peut servir à exécuter ALS.

* Le [stockage Blob Azure][blob] stocke le jeu de données pour les suggestions de films.

* [Azure Machine Learning service][mls] est utilisé pour accélérer la création, la gestion et le déploiement de modèles Machine Learning.

* [Azure Cosmos DB][cosmosdb] assure le stockage de bases de données multimodèles distribuées à l’échelle mondiale.

* [Azure Container Instances][aci] sert à déployer les modèles entraînés sur les services web ou d’application, en utilisant au besoin [Azure Kubernetes Service][aks].

### <a name="alternatives"></a>Autres solutions

[Azure Databricks][databricks] est un cluster Spark managé dans lequel l’entraînement et l’évaluation de modèles sont effectués. Vous pouvez configurer un environnement Spark managé en quelques minutes, et le [mettre à l’échelle automatiquement][autoscale] afin de réduire les ressources et les coûts associés à une mise à l’échelle manuelle des clusters. Une autre option visant à économiser des ressources consiste à configurer des [clusters][clusters] inactifs pour un arrêt automatique.

## <a name="considerations"></a>Considérations

### <a name="availability"></a>Disponibilité

Les applications créés pour le machine learning sont divisées en deux composants de ressources : les ressources destinées à l’entraînement et celles destinées au traitement. Les ressources nécessaires à l’entraînement n’ont généralement pas besoin de haute disponibilité, car les requêtes de production en temps réel ne touchent pas directement ces ressources. Par contre, les ressources nécessaires au traitement nécessitent une haute disponibilité pour gérer les requêtes clientes.

Pour l’entraînement, la machine DSVM est disponible dans [plusieurs régions][regions] du monde entier et remplit le [contrat de niveau de service][sla] (SLA) des machines virtuelles. Pour le traitement, Azure Kubernetes Service fournit une infrastructure [hautement disponible][ha]. Les nœuds d’agent suivent également le [SLA][sla-aks] des machines virtuelles.

### <a name="scalability"></a>Extensibilité

Si vous avez un volume de données important, vous pouvez mettre à l’échelle votre DSVM afin de raccourcir le temps d’entraînement. Vous pouvez effectuer un scale-up ou un scale-down d’une machine virtuelle en changeant sa [taille][vm-size]. Choisissez une taille de mémoire adaptée à votre jeu de données en mémoire, et un nombre plus élevé de processeurs virtuels afin de réduire le temps nécessaire à l’entraînement.

### <a name="security"></a>Sécurité

Ce scénario peut utiliser Azure Active Directory pour authentifier les utilisateurs qui [accèdent à la machine DSVM][dsvm-id] contenant votre code, vos modèles et vos données (en mémoire). Les données sont stockées dans Stockage Azure avant d’être chargées sur une machine DSVM où elles sont automatiquement chiffrées à l’aide de [Storage Service Encryption][storage-security]. Les autorisations peuvent être managées via l’authentification Azure Active Directory ou le contrôle d’accès en fonction du rôle.

## <a name="deploy-this-scenario"></a>Déployer ce scénario

**Conditions préalables**: Vous devez disposer d’un compte Azure existant. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit][free] avant de commencer.

L’intégralité du code de ce scénario est disponible dans le [dépôt Microsoft Recommenders][github].

Suivez ces étapes pour exécuter le [notebook de démarrage rapide ALS][notebook] :

1. [Créez une machine DSVM][dsvm-ubuntu] à partir du portail Azure.

2. Clonez le dépôt dans le dossier Notebooks :

    ```shell
    cd notebooks
    git clone https://github.com/Microsoft/Recommenders
    ```

3. Installez les dépendances conda en suivant les étapes décrites dans le fichier [SETUP.md][setup].

4. Dans un navigateur, affichez votre machine virtuelle jupyterlab et accédez à `notebooks/00_quick_start/als_pyspark_movielens.ipynb`.

5. Exécutez le notebook.

## <a name="related-resources"></a>Ressources associées

Pour des instructions détaillées sur la création et la mise à l’échelle d’un service de recommandations, consultez [Créer une API de recommandation en temps réel sur Azure][ref-arch]. Pour des tutoriels et des exemples de systèmes de recommandations, consultez le [Dépôt Microsoft Recommenders][github].

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
