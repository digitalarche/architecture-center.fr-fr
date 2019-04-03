---
title: Scoring par lots des modèles Python sur Azure
description: Créez une solution scalable pour le scoring par lots de modèles selon une planification en parallèle avec Azure Machine Learning Service.
author: njray
ms.date: 01/30/2019
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: azcat-ai, AI
ms.openlocfilehash: b7607984bcf2c4bd046421aeb6e9d52dd8e7c18e
ms.sourcegitcommit: 1a3cc91530d56731029ea091db1f15d41ac056af
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/03/2019
ms.locfileid: "58887741"
---
# <a name="batch-scoring-of-python-machine-learning-models-on-azure"></a>Modèles de notation par lot de l’apprentissage de Python sur Azure

Cette architecture de référence montre comment créer une solution scalable pour le scoring par lots d’un grand nombre de modèles selon une planification en parallèle avec Azure Machine Learning Service. La solution peut être utilisée comme modèle et appliquée à différents problèmes.

Une implémentation de référence pour cette architecture est disponible sur [GitHub][github].

![Scoring par lots des modèles Python sur Azure](./_images/batch-scoring-python.png)

**Scénario** : Cette solution supervise le fonctionnement d’un grand nombre d’appareils dans un paramètre IoT où chaque appareil envoie des lectures de capteurs en permanence. Chaque appareil est supposé être associé à des modèles de détection des anomalies préentraînés, qui doivent être utilisés pour prédire si une série de mesures, agrégées sur un intervalle de temps prédéfini, correspondent ou non à une anomalie. Dans les scénarios réels, il peut s’agir d’un flux de lectures de capteurs qui doivent être filtrées et agrégées avant d’être utilisées dans l’entraînement ou le scoring en temps réel. Pour simplifier, cette solution utilise le même fichier de données lors de l’exécution des travaux de scoring.

Cette architecture de référence est conçue pour des charges de travail déclenchées selon une planification. Le traitement est constitué des étapes suivantes :
1.  Envoyer des lectures de capteurs pour ingestion à Azure Event Hubs.
2.  Effectuer le traitement des flux et stocker les données brutes.
3.  Envoyer les données à un cluster Machine Learning prêt à travailler. Chaque nœud du cluster exécute un travail de scoring pour un capteur spécifique. 
4.  Exécuter le pipeline de scoring, qui exécute les travaux de scoring en parallèle avec des scripts Python Machine Learning. Le pipeline est créé et publié, et son exécution est planifiée à un intervalle de temps prédéfini.
5.  Générer des prédictions et les stocker dans Stockage Blob pour les utiliser plus tard.

## <a name="architecture"></a>Architecture

Cette architecture est constituée des composants suivants :

[Azure Event Hubs][event-hubs]. Ce service d’ingestion de messages peut recevoir des millions de messages d’événement par seconde. Dans cette architecture, les capteurs envoient un flux de données au hub d’événements.

[Azure Stream Analytics][stream-analytics]. Moteur de traitement des événements. Un travail Stream Analytics lit les flux de données provenant du hub d’événements et effectue le traitement des flux.

[Azure SQL Database][sql-database]. Les données des lectures des capteurs sont chargées dans SQL Database. SQL est un moyen bien connu de stocker les données en flux traitées (qui sont tabulaires et structurées), mais d’autres magasins de données peuvent être utilisés.

[Service Azure Machine Learning][amls]. Machine Learning est un service cloud pour l’entraînement, le scoring, le déploiement et la gestion des modèles de machine learning à grande échelle. Dans le contexte du scoring par lots, Machine Learning crée un cluster de machines virtuelles à la demande avec une option de mise à l’échelle automatique, où chaque nœud du cluster exécute un travail de scoring pour un capteur spécifique. Les travaux de scoring sont exécutés en parallèle sous forme d’étapes de script Python qui sont placés en file d’attente et gérés par Machine Learning. Ces étapes font partie d’un pipeline Machine Learning qui est créé et publié, et dont l’exécution est planifiée à un intervalle de temps prédéfini.

[Stockage Blob Azure][storage]. Les conteneurs d’objets blob sont utilisés pour stocker les modèles préentraînés, les données et les prédictions de sortie. Les modèles sont chargés sur Stockage Blob dans le notebook [01_create_resources.ipynb][create-resources]. Ces modèles [SVM à une classe][one-class-svm] sont entraînés sur des données qui représentent les valeurs de différents capteurs pour différents appareils. Cette solution part du principe que les valeurs de données sont agrégées sur un intervalle de temps fixe.

[Azure Container Registry][acr]. Le [script][pyscript] Python de scoring s’exécute dans des conteneurs Docker qui sont créés sur chaque nœud du cluster, où il lit les données des capteurs appropriés, génère des prédictions et les stocke dans Stockage Blob.

## <a name="performance-considerations"></a>Considérations relatives aux performances

Pour les modèles Python standard, il est généralement admis que les processeurs sont suffisants pour gérer la charge de travail. Cette architecture utilise des processeurs. Toutefois, pour [charges de travail d’apprentissage approfondi][deep], GPU généralement plus performantes que les unités centrales par une quantité considérable &mdash; un cluster important d’UC est généralement nécessaire pour obtenir des performances comparables.

### <a name="parallelizing-across-vms-versus-cores"></a>Parallélisation entre machines virtuelles par rapport aux cœurs

Lors de l’exécution des processus de scoring de nombreux modèles en mode Batch, le travail doit être mis en parallèle sur les machines virtuelles. Deux approches sont possibles :

* Créer un cluster plus grand avec des machines virtuelles de faible coût.

* Créer un cluster plus petit avec des machines virtuelles hautes performances et plus de cœurs disponibles sur chacune.

En général, le scoring des modèles Python standard n’est pas aussi exigeant que celui des modèles d’apprentissage profond, et un petit cluster doit être en mesure de gérer efficacement un grand nombre de modèles en file d’attente. Vous pouvez accroître le nombre de nœuds de cluster à mesure que les tailles des jeux de données augmentent.

Pour des raisons pratiques, dans ce scénario, une seule tâche de scoring est envoyée dans une étape du pipeline Machine Learning. Il peut cependant être plus efficace d’effectuer le scoring de plusieurs blocs de données dans la même étape de pipeline. Dans ce cas, écrivez un code personnalisé pour lire dans plusieurs jeux de données et exécutez le script de scoring pour ceux-ci au cours d’une même étape.

## <a name="management-considerations"></a>Considérations relatives à la gestion

- **Superviser les travaux**. Il est important de superviser la progression de l’exécution de travaux, mais cela peut s’avérer ardu sur un cluster de nœuds actifs. Pour examiner l’état des nœuds du cluster, utilisez le [portail Azure][portal] pour gérer l’[espace de travail Machine Learning][ml-workspace]. Si un nœud est inactif ou si un travail a échoué, vous pouvez consulter les journaux d’erreurs enregistrés dans Stockage Blob. Ils sont également accessibles dans la section Pipelines. Pour une supervision approfondie, connectez les journaux à [Application Insights][app-insights], ou exécutez des processus distincts pour interroger l’état du cluster et de ses travaux.
-   **Journalisation**. Machine Learning Service journalise tous les stdout/stderr dans le compte de stockage Azure associé. Pour consulter facilement les fichiers journaux, utilisez un outil de navigation dans le stockage comme [Explorateur Stockage Azure][explorer].

## <a name="cost-considerations"></a>Considérations relatives au coût

Les composants les plus coûteux utilisés dans cette architecture de référence sont les ressources de calcul. La taille du cluster de calcul peut faire l’objet d’un scale-up ou d’un scale-down en fonction des travaux présents dans la file d’attente. Activez la mise à l’échelle automatique programmatiquement via le SDK Python en modifiant la configuration du provisionnement de la capacité de calcul. Vous pouvez aussi utiliser [Azure CLI][cli] pour définir les paramètres de mise à l’échelle automatique du cluster.

Pour les tâches qui ne nécessitent pas un traitement immédiat, configurez la formule de mise à l’échelle automatique de sorte que l’état par défaut (minimum) soit un cluster sans nœud. Avec cette configuration, le cluster démarre sans nœud et ne monte en puissance que s’il détecte des tâches dans la file d’attente. Si le processus de scoring par lots ne se produit que quelques fois par jour ou moins, ce paramètre permet de réaliser des économies significatives.

La mise à l’échelle automatique peut ne pas convenir pour les traitements par lots trop rapprochés les uns des autres. Le temps nécessaire au lancement et à l’arrêt d’un cluster a aussi un coût. De ce fait, si une charge de travail par lots commence seulement quelques minutes après la fin du travail précédent, il peut être plus rentable de laisser le cluster en fonctionnement entre les travaux. Cela dépend si les processus de scoring sont planifiés pour s’exécuter très fréquemment (toutes les heures, par exemple) ou moins fréquemment (une fois par mois, par exemple).


## <a name="deployment"></a>Déploiement

Pour déployer cette architecture de référence, suivez les étapes décrites dans le [dépôt GitHub][github].

[acr]: /azure/container-registry/container-registry-intro
[ai]: /azure/application-insights/app-insights-overview
[aml-compute]: /azure/machine-learning/service/how-to-set-up-training-targets#amlcompute
[amls]: /azure/machine-learning/service/overview-what-is-azure-ml
[automatic-scaling]: /azure/batch/batch-automatic-scaling
[azure-files]: /azure/storage/files/storage-files-introduction
[cli]: /cli/azure
[create-resources]: https://github.com/Microsoft/AMLBatchScoringPipeline/blob/master/01_create_resources.ipynb
[deep]: /azure/architecture/reference-architectures/ai/batch-scoring-deep-learning
[event-hubs]: /azure/event-hubs/event-hubs-geo-dr
[explorer]: https://azure.microsoft.com/en-us/features/storage-explorer/
[github]: https://github.com/Microsoft/AMLBatchScoringPipeline
[one-class-svm]: http://scikit-learn.org/stable/modules/generated/sklearn.svm.OneClassSVM.html
[portal]: https://portal.azure.com
[ml-workspace]: /azure/machine-learning/studio/create-workspace
[python-script]: https://github.com/Azure/BatchAIAnomalyDetection/blob/master/batchai/predict.py
[pyscript]: https://github.com/Microsoft/AMLBatchScoringPipeline/blob/master/scripts/predict.py
[storage]: /azure/storage/blobs/storage-blobs-overview
[stream-analytics]: /azure/stream-analytics/
[sql-database]: /azure/sql-database/
[app-insights]: /azure/application-insights/app-insights-overview
