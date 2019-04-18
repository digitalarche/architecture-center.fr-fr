---
title: Notation avec des modèles R sur Azure par lot
description: Exécuter avec des modèles R à l’aide d’Azure Batch et un jeu de données selon les prévisions de vente de magasin de vente au détail de notation par lots.
author: njray
ms.date: 03/29/2019
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: azcat-ai
ms.openlocfilehash: 72769cf078596f0312a1f4293205dda5a086ef41
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59639901"
---
# <a name="batch-scoring-of-r-machine-learning-models-on-azure"></a>Modèles de notation par lot de l’apprentissage de R sur Azure

Cette architecture de référence montre comment effectuer des modèles R à l’aide d’Azure Batch de notation par lots. Le scénario est basé sur les prévisions de vente de magasin de vente au détail, mais cette architecture peut être étendue pour n’importe quel scénario nécessitant la génération de prédictions sur un grand scaler à l’aide des modèles R. Une implémentation de référence pour cette architecture est disponible sur [GitHub][github].

![Diagramme de l’architecture][0]

**Scénario** : Une chaîne de supermarchés doit prévoir les ventes de produits au cours du trimestre à venir. La prévision permet à mieux gérer sa chaîne d’approvisionnement et de vous assurer qu’il peut répondre à la demande pour les produits à chacune de ses magasins de l’entreprise. La société met à jour ses prévisions chaque semaine que de nouvelles données de ventes à partir de la semaine précédente devient disponibles et la commercialisation de stratégie pour le trimestre suivant du produit est défini. Prévisions de Quantile sont générées pour estimer l’incertitude des prévisions de ventes individuelles.

Le traitement est constitué des étapes suivantes :

1. Une application logique Azure déclenche le processus de génération de prévision une fois par semaine.

1. L’application logique démarre une Instance de conteneur Azure qui exécute le Planificateur de conteneur Docker, ce qui déclenche les travaux de calcul de score sur le cluster de traitement par lots.

1. Calcul de score exécutés en parallèle entre les nœuds du cluster de traitement par lots. Chaque nœud :

    1. Extrait le worker image Docker à partir de Docker Hub et démarre un conteneur.

    1. Lit les données d’entrée et préformés modèles R à partir du stockage d’objets Blob Azure.

    1. Évalue les données pour produire les prévisions.

    1. Écrit les résultats de prévisions pour le stockage d’objets blob.

La figure ci-dessous illustre les prévisions de ventes pour les quatre produits (références SKU) dans un magasin. La ligne noire est l’historique des ventes, la ligne en pointillés est la valeur médiane (q50) de prévision, la bande rose représente les centiles vingt-cinq et soixante-quinze cinquième et la bande bleu représente les centiles cinquième et cinquième 90.

![Prévisions de ventes][1]

## <a name="architecture"></a>Architecture

Cette architecture est constituée des composants suivants.

[Azure Batch] [ batch] est utilisé pour exécuter des tâches de génération de prévision en parallèle sur un cluster de machines virtuelles. Prédictions sont faites à l’aide de la machine PRÉFORMÉE implémentés dans R. Azure Batch des modèles d’apprentissage peuvent automatiquement mettre à l’échelle le nombre de machines virtuelles en fonction du nombre de travaux soumis au cluster. Sur chaque nœud, un script R s’exécute dans un conteneur Docker à noter les données et générer des prévisions.

[Stockage d’objets Blob Azure] [ blob] est utilisé pour stocker les données d’entrée, les modèles d’apprentissage automatique préformé et les résultats de prévision. Il offre un stockage très économique pour les performances nécessitant cette charge de travail.

[Azure Container Instances] [ aci] fournir le calcul sans serveur à la demande. Dans ce cas, une instance de conteneur est déployée selon une planification pour déclencher les tâches de traitement par lots qui génèrent les prévisions. Les traitements par lots sont déclenchés à partir d’un script R à l’aide du [doAzureParallel][doAzureParallel] package. L’instance de conteneur s’arrête automatiquement une fois les travaux terminés.

[Azure Logic Apps] [ logic-apps] déclencher l’intégralité du workflow en déployant les instances de conteneur selon une planification. Un connecteur dans Logic Apps d’Azure Container Instances permet à une instance à être déployé sur une plage de déclencher des événements.

## <a name="performance-considerations"></a>Considérations relatives aux performances

### <a name="containerized-deployment"></a>Déploiement en conteneur

Avec cette architecture, tous les scripts R s’exécutent dans [Docker](https://www.docker.com/) conteneurs. Cela garantit que les scripts exécutés dans un environnement cohérent, avec la même version de R et les versions de packages, à chaque fois. Les images Docker distinctes sont utilisés pour les conteneurs du planificateur et de travail, car chacun possède un ensemble de dépendances de package R.

Azure Container Instances fournit un environnement sans serveur pour exécuter le Planificateur de conteneur. Le conteneur de planificateur exécute un script R qui déclenche les travaux de calcul de score individuels s’exécutant sur un cluster Azure Batch.

Chaque nœud du cluster de traitement par lots s’exécute le conteneur de travail, qui exécute le script de notation.

### <a name="parallelizing-the-workload"></a>Parallélisation de la charge de travail

Lorsque batch notation des données avec des modèles R, déterminez comment paralléliser la charge de travail. Les données d’entrée doivent être partitionnées d’une certaine manière afin que l’opération d’évaluation peut être répartie sur les nœuds du cluster. Expérimentez des approches différentes pour découvrir le meilleur choix pour la distribution de votre charge de travail. Cas par cas, considérez les points suivants :

- La quantité de données permettre être chargé et traité dans la mémoire d’un seul nœud.
- La surcharge de démarrage de chaque traitement par lots.
- La surcharge de chargement des modèles R.

Dans le scénario utilisé pour cet exemple, les objets de modèle sont volumineux, et il prend quelques secondes seulement pour générer des prévisions pour les produits individuels. Pour cette raison, vous pouvez regrouper les produits et exécuter un traitement par lots unique par nœud. Une boucle au sein de chaque tâche génère des prévisions pour les produits séquentiellement. Cette méthode s’avère pour être le moyen le plus efficace de paralléliser cette charge de travail particulier. Il permet d’éviter la surcharge liée à partir de nombreux traitements par lots plus petits et à plusieurs reprises le chargement des modèles R.

Une autre approche consiste à déclencher un traitement par lots par produit. Traitement par lots Azure constitue une file d’attente des travaux automatiquement et les soumet à exécuter sur le cluster comme nœuds deviennent disponibles. Utilisez [mise à l’échelle automatique] [ autoscale] pour ajuster le nombre de nœuds du cluster en fonction du nombre de travaux. Cette approche est plus judicieux de temps relativement long pour terminer chaque opération de calcul de score, justifiant la surcharge de démarrer les travaux et de recharger les objets de modèle. Cette approche est également plus simple à implémenter et vous donne la possibilité d’utiliser la mise à l’échelle automatique, un facteur important si la taille de la charge de travail totale n’est pas connue à l’avance.

## <a name="monitoring-and-logging-considerations"></a>Supervision et enregistrement des considérations

### <a name="monitoring-azure-batch-jobs"></a>Surveillance des travaux Azure Batch

Surveiller et arrêter le traitement par lots à partir de la **travaux** volet du compte Batch dans le portail Azure. Surveiller le cluster de lot, y compris l’état des nœuds individuels, à partir de la **Pools** volet.

### <a name="logging-with-doazureparallel"></a>Journalisation avec doAzureParallel

Le package doAzureParallel collecte automatiquement des journaux de tous les stdout/stderr pour chaque tâche envoyée sur Azure Batch. Vous la trouverez dans le compte de stockage créé lors de l’installation. Pour les afficher, utilisez un outil de navigation de stockage comme [Azure Storage Explorer] [ storage-explorer] ou le portail Azure.

Pour déboguer rapidement des traitements par lots pendant le développement, imprimer des journaux dans votre session R locale à l’aide du [getJobFiles][getJobFiles] de doAzureParallel (fonction).

## <a name="cost-considerations"></a>Considérations relatives au coût

Les ressources de calcul utilisées dans cette architecture de référence sont les composants plus coûteuses. Pour ce scénario, un cluster de taille fixe est créé chaque fois que la tâche est déclenchée, puis arrêtez une fois la tâche terminée. Des frais sont facturés uniquement pendant que les nœuds de cluster sont démarrage, en cours d’exécution ou en cours d’arrêt. Cette approche est appropriée pour un scénario où les ressources de calcul requises pour générer les prévisions demeurent relativement constants à partir d’un travail à un travail.

Dans les scénarios où la quantité de calcul requis pour terminer la tâche n’est pas connue d’avance, il peut être plus judicieux d’utiliser la mise à l’échelle automatique. Avec cette approche, la taille du cluster est mis à l’échelle vers le haut ou vers le bas en fonction de la taille de la tâche. Azure Batch prend en charge une plage de formules à l’échelle automatique que vous pouvez définir lors de la définition du cluster à l’aide de la [doAzureParallel][doAzureParallel] API.

Dans certains scénarios, le temps entre les travaux peut être trop court pour arrêter et démarrer le cluster. Dans ce cas, conserver le cluster en cours d’exécution entre les travaux si nécessaire.

Azure Batch et doAzureParallel prend en charge l’utilisation de machines virtuelles de faible priorité. Ces machines virtuelles sont fournis avec une remise significative, mais risque d’être affectés par les autres charges de travail de priorité plus élevées. L’utilisation de ces machines virtuelles ne sont donc pas recommandé pour les charges de travail de production critiques. Toutefois, elles sont très utiles pour expérimentales ou charges de travail de développement.

## <a name="deployment"></a>Déploiement

Pour déployer cette architecture de référence, suivez les étapes décrites dans le [GitHub][github] dépôt.

[0]: ./_images/batch-scoring-r-models.png
[1]: ./_images/sales-forecasts.png
[aci]: /azure/container-instances/container-instances-overview
[autoscale]: /azure/batch/batch-automatic-scaling
[batch]: /azure/batch/batch-technical-overview
[blob]: /azure/storage/blobs/storage-blobs-introduction
[doAzureParallel]: https://github.com/Azure/doAzureParallel/blob/master/docs/32-autoscale.md
[getJobFiles]: /azure/machine-learning/service/how-to-train-ml-models
[github]: https://github.com/Azure/RBatchScoring
[logic-apps]: /azure/logic-apps/logic-apps-overview
[storage-explorer]: /azure/vs-azure-tools-storage-manage-with-storage-explorer?tabs=windows