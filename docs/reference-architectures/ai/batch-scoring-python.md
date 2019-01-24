---
title: Scoring par lots des modèles Python sur Azure
description: Générez une solution scalable pour le scoring par lots des modèles selon une planification en parallèle à l’aide d’Azure Batch AI.
author: njray
ms.date: 12/13/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: azcat-ai, AI
ms.openlocfilehash: a291821860a8e503ba4c6173ac6d8fd449d6ebf3
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54485364"
---
# <a name="batch-scoring-of-python-models-on-azure"></a>Scoring par lots des modèles Python sur Azure

Cette architecture de référence montre comment générer une solution scalable pour le scoring par lots de nombreux modèles selon une planification en parallèle à l’aide d’Azure Batch AI. La solution peut être utilisée comme modèle et appliquée à différents problèmes.

Une implémentation de référence pour cette architecture est disponible sur [GitHub][github].

![Scoring par lots des modèles Python sur Azure](./_images/batch-scoring-python.png)

**Scénario** : Cette solution supervise le fonctionnement d’un grand nombre d’appareils dans un paramètre IoT où chaque appareil envoie des lectures de capteurs en permanence. Il est supposé que chaque appareil dispose de modèles de détection des anomalies préentraînés qui doivent être utilisés pour prédire si une série de mesures, agrégées sur un intervalle de temps prédéfini, correspondent ou non à une anomalie. Dans les scénarios réels, il peut s’agir d’un flux de lectures de capteurs qui doivent être filtrées et agrégées avant d’être utilisées dans l’entraînement ou le scoring en temps réel. Par souci de simplicité, la solution utilise le même fichier de données lors de l’exécution des travaux de scoring.

## <a name="architecture"></a>Architecture

Cette architecture est constituée des composants suivants :

[Azure Event Hubs][event-hubs]. Ce service d’ingestion de messages peut recevoir des millions de messages d’événement par seconde. Dans cette architecture, les capteurs envoient un flux de données au hub d’événements.

[Azure Stream Analytics][stream-analytics]. Moteur de traitement des événements. Un travail Stream Analytics lit les flux de données provenant du hub d’événements et effectue le traitement des flux.

[Azure Batch AI][batch-ai]. Ce moteur de calcul distribué est utilisé pour entraîner et tester les modèles de machine learning et d’intelligence artificielle à grande échelle dans Azure. Batch AI crée des machines virtuelles à la demande avec une option de mise à l’échelle automatique, où chaque nœud du cluster Batch AI exécute un travail de scoring pour un capteur spécifique. Le [script][python-script] Python de scoring s’exécute dans des conteneurs Docker qui sont créés sur chaque nœud du cluster, où il lit les données de capteurs pertinentes, génère des prédictions et les stocke dans le stockage Blob.

[Stockage Blob Azure][storage]. Les conteneurs d’objets blob sont utilisés pour stocker les modèles préentraînés, les données et les prédictions de sortie. Les modèles sont chargés sur le stockage Blob dans le notebook [create\_resources.ipynb][create-resources]. Ces modèles [SVM à une classe][one-class-svm] sont entraînés sur des données qui représentent les valeurs de différents capteurs pour différents appareils. Cette solution part du principe que les valeurs de données sont agrégées sur un intervalle de temps fixe.

[Azure Logic Apps][logic-apps]. Cette solution crée une application logique qui exécute toutes les heures des travaux Batch AI. Logic Apps offre un moyen facile de créer le workflow du runtime et la planification de la solution. Les travaux Batch AI sont envoyés à l’aide d’un [script][script] Python qui s’exécute également dans un conteneur Docker.

[Azure Container Registry][acr]. Les images Docker sont utilisées à la fois dans Batch AI et Logic Apps, et sont créées dans le notebook [create\_resources.ipynb][create-resources], puis envoyées (push) à Container Registry. Cela représente un moyen pratique d’héberger des images et d’instancier des conteneurs via d’autres services Azure, Logic Apps et Batch AI dans cette solution.

## <a name="performance-considerations"></a>Considérations relatives aux performances

Pour les modèles Python standard, il est généralement admis que les processeurs sont suffisants pour gérer la charge de travail. Cette architecture utilise des processeurs. Toutefois, pour les [charges de travail d’apprentissage profond][deep], les GPU sont généralement beaucoup plus performants que les CPU, dans la mesure où un cluster de CPU important est nécessaire pour obtenir des performances comparables.

### <a name="parallelizing-across-vms-vs-cores"></a>Parallélisation dans les machines virtuelles et les cœurs

Lors de l’exécution des processus de scoring de nombreux modèles en mode Batch, le travail doit être mis en parallèle sur les machines virtuelles. Deux approches sont possibles :

* Créer un cluster plus grand avec des machines virtuelles de faible coût.

* Créer un cluster plus petit avec des machines virtuelles hautes performances et plus de cœurs disponibles sur chacune.

En général, le scoring des modèles Python standard n’est pas aussi exigeant que celui des modèles d’apprentissage profond, et un petit cluster doit être en mesure de gérer efficacement un grand nombre de modèles en file d’attente. Vous pouvez accroître le nombre de nœuds de cluster à mesure que les tailles des jeux de données augmentent.

Pour des raisons pratiques dans ce scénario, une tâche de scoring est envoyée au sein d’un travail Batch AI unique. Toutefois, il peut être plus efficace de scorer plusieurs blocs de données au sein du même travail Batch AI. Dans ce cas, écrivez un code personnalisé à lire dans plusieurs jeux de données et exécutez le script de scoring pour ceux-ci lors d’un travail Batch AI unique.

### <a name="file-servers"></a>Serveurs de fichiers

Quand vous utilisez Batch AI, vous pouvez choisir plusieurs options de stockage, selon le débit nécessaire à votre scénario. Pour les charges de travail qui demandent peu de débit, l’utilisation du stockage Blob doit suffire. Sinon, Batch AI prend aussi en charge un [serveur de fichiers Batch AI][bai-file-server], un système NFS à un seul nœud géré, qui peut être monté automatiquement sur les nœuds du cluster pour fournir un emplacement de stockage accessible de façon centralisée pour les travaux. Dans la plupart des cas, un seul serveur de fichiers s’avère nécessaire dans un espace de travail, et vous pouvez répartir les données de vos travaux d’entraînement dans différents répertoires.

Si un système NFS à un seul nœud ne convient pas pour vos charges de travail, Batch AI prend en charge d’autres options de stockage, dont [Azure Files][azure-files] et les solutions personnalisées que sont le système de fichiers Gluster ou Lustre.

## <a name="management-considerations"></a>Considérations relatives à la gestion

### <a name="monitoring-batch-ai-jobs"></a>Supervision des tâches Batch AI

Il est important de superviser la progression de l’exécution de travaux, mais cela peut s’avérer ardu sur un cluster de nœuds actifs. Pour vous faire une idée de l’état global du cluster, accédez au panneau **Batch AI** du [portail Azure][portal] pour inspecter l’état des nœuds du cluster. Si un nœud est inactif ou si un travail a échoué, les journaux d’erreurs sont enregistrés dans le stockage Blob et sont aussi accessibles dans le panneau **Travaux** du portail.

Pour une supervision plus complète, connectez les journaux à [Application Insights][ai] ou exécutez des processus distincts pour demander l’état du cluster Batch AI et de ses travaux.

### <a name="logging-in-batch-ai"></a>Journalisation dans Batch AI

Batch AI journalise tous les stdout/stderr dans le compte de stockage Azure associé. L’utilisation d’un outil de navigation de stockage comme l’[Explorateur Stockage Azure][explorer] facilite la navigation dans les fichiers journaux.

Quand vous déployez cette architecture de référence, vous pouvez configurer un système de journalisation plus simple. Avec cette option, tous les journaux des différents travaux sont enregistrés dans un même répertoire de votre conteneur d’objets blob, comme illustré ci-dessous. Ces journaux s’avèrent utiles pour superviser la durée de traitement de chaque travail et de chaque image, et vous pouvez ainsi vous faire une idée plus précise de la façon d’optimiser le processus.

![Explorateur de stockage Azure](./_images/batch-scoring-python-monitor.png)

## <a name="cost-considerations"></a>Considérations relatives au coût

Les composants les plus coûteux utilisés dans cette architecture de référence sont les ressources de calcul.

La taille du cluster Batch AI peut être mise à l’échelle à la hausse ou à la baisse en fonction des travaux présents dans la file d’attente. Avec Batch AI, vous pouvez activer la [mise à l’échelle automatique][automatic-scaling] de deux façons différentes. Vous pouvez le faire par programmation, ce que vous pouvez configurer dans le fichier .env lors des [étapes de déploiement][github], ou vous pouvez changer la formule de mise à l’échelle directement dans le portail une fois le cluster créé.

Pour les tâches qui ne nécessitent pas un traitement immédiat, configurez la formule de mise à l’échelle automatique de sorte que l’état par défaut (minimum) soit un cluster sans nœud. Avec cette configuration, le cluster démarre sans nœud et ne monte en puissance que s’il détecte des tâches dans la file d’attente. Si le processus de scoring par lots ne s’enclenche que quelques fois par jour, ce paramètre permet de réaliser des économies significatives.

La mise à l’échelle automatique peut ne pas convenir pour les traitements par lots trop rapprochés les uns des autres. Le temps nécessaire au lancement et à l’arrêt d’un cluster a aussi un coût. De ce fait, si une charge de travail Batch commence seulement quelques minutes après la fin de la tâche précédente, il peut être plus rentable de laisser le cluster s’exécuter entre les tâches. Cela dépend si les processus de scoring sont planifiés pour s’exécuter très fréquemment (toutes les heures, par exemple) ou moins fréquemment (une fois par mois, par exemple).

## <a name="deploy-the-solution"></a>Déployer la solution

L’implémentation de référence de cette architecture est disponible sur [GitHub][github]. Suivez les étapes de configuration ici afin de générer une solution scalable pour le scoring de nombreux modèles en parallèle à l’aide de Batch AI.

[acr]: /azure/container-registry/container-registry-intro
[ai]: /azure/application-insights/app-insights-overview
[automatic-scaling]: /azure/batch/batch-automatic-scaling
[azure-files]: /azure/storage/files/storage-files-introduction
[batch-ai]: /azure/batch-ai/
[bai-file-server]: /azure/batch-ai/resource-concepts#file-server
[create-resources]: https://github.com/Azure/BatchAIAnomalyDetection/blob/master/create_resources.ipynb
[deep]: /azure/architecture/reference-architectures/ai/batch-scoring-deep-learning
[event-hubs]: /azure/event-hubs/event-hubs-geo-dr
[explorer]: https://azure.microsoft.com/en-us/features/storage-explorer/
[github]: https://github.com/Azure/BatchAIAnomalyDetection
[logic-apps]: /azure/logic-apps/logic-apps-overview
[one-class-svm]: http://scikit-learn.org/stable/modules/generated/sklearn.svm.OneClassSVM.html
[portal]: https://portal.azure.com
[python-script]: https://github.com/Azure/BatchAIAnomalyDetection/blob/master/batchai/predict.py
[script]: https://github.com/Azure/BatchAIAnomalyDetection/blob/master/sched/submit_jobs.py
[storage]: /azure/storage/blobs/storage-blobs-overview
[stream-analytics]: /azure/stream-analytics/
