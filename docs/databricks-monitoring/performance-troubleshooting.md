---
title: Problèmes de performances de Azure Databricks à l’aide d’Azure Monitor
titleSuffix: ''
description: Utiliser des tableaux de bord Grafana pour résoudre les problèmes de performances dans Azure Databricks
author: petertaylor9999
ms.date: 04/02/2019
ms.topic: ''
ms.service: ''
ms.subservice: ''
ms.openlocfilehash: 49ec63d0c45ab388ca83b3ab0562428327539619
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/18/2019
ms.locfileid: "59740372"
---
# <a name="troubleshoot-performance-bottlenecks-in-azure-databricks"></a>Résoudre les goulots d’étranglement de performances dans Azure Databricks

Cet article décrit comment utiliser des tableaux de bord de surveillance pour trouver les goulots d’étranglement de performances dans des travaux Spark sur Azure Databricks.

[Azure Databricks](/azure/azure-databricks/) est un [Apache Spark](https://spark.apache.org/)– en fonction du service d’analytique qui facilite la rapidement développer et déployer l’analytique des données volumineuses. Surveillance et dépannage des problèmes de performances sont une critique lors de l’exploitation des charges de travail de production Azure Databricks. Pour identifier les problèmes de performances courants, il est conseillé d’utiliser la surveillance de visualisations basées sur les données de télémétrie.

## <a name="prerequisites"></a>Conditions préalables

Pour configurer les tableaux de bord Grafana présenté dans cet article :

- Configurer votre cluster Databricks pour envoyer la télémétrie à un espace de travail Analytique de journal, à l’aide de la bibliothèque d’analyse Azure Databricks. Pour plus d’informations, consultez [configurer Azure Databricks pour envoyer des métriques à Azure Monitor](./configure-cluster.md).

- Déployez Grafana dans une machine virtuelle. Consultez [utiliser des tableaux de bord pour visualiser les métriques d’Azure Databricks](./dashboards.md).

Le tableau de bord Grafana est déployé comprend un ensemble de visualisations de série chronologique. Chaque graphique est tracé de séries chronologiques des mesures relatives à une tâche Apache Spark, les étapes de travail et des tâches qui composent chaque étape.

## <a name="azure-databricks-performance-overview"></a>Vue d’ensemble des performances Azure Databricks

Azure Databricks est basée sur Apache Spark, un système informatique distribué à usage général. Code d’application, appelé un **travail**, s’exécute sur un cluster Apache Spark, coordonné par le Gestionnaire du cluster. En règle générale, un travail est l’unité de plus haut niveau de calcul. Une tâche représente l’opération complète effectuée par l’application Spark. Une opération standard inclut la lecture des données à partir d’une source de, appliquant des transformations de données et écrire les résultats vers le stockage ou une autre destination.

Travaux sont décomposent en **étapes**. Le travail progresse vers les étapes séquentiellement, ce qui signifie que les phases ultérieures doivent attendre les étapes précédentes terminer. Étapes contiennent des groupes d’identiques **tâches** qui peuvent être exécutées en parallèle sur plusieurs nœuds du cluster Spark. Les tâches sont l’unité plus granulaire d’exécution qui ont lieu sur un sous-ensemble des données.

Les sections suivantes décrivent certaines visualisations de tableau de bord qui sont utiles pour la résolution des problèmes de performances.

## <a name="job-and-stage-latency"></a>Latence de travail et de la scène

Latence du travail est la durée d’une exécution du travail à partir de lorsqu’il démarre jusqu'à la fin. Elle est affichée en tant que centiles d’une exécution du travail par ID de cluster et des applications, pour permettre la visualisation des valeurs hors norme. Le graphique suivant montre un historique des travaux où le 90e centile atteint 50 secondes, même si la 50e centile a été constamment environ 10 secondes.

![Graphique illustrant la latence du travail](./_images/grafana-job-latency.png)

Examiner l’exécution du travail en cluster et des applications, recherchez des pics de latence. Une fois que les clusters et des applications avec une latence élevée sont identifiées, passer à examiner la latence de la scène.

Latence de l’étape est également affichée en tant que centiles pour permettre la visualisation des valeurs hors norme. Latence de la scène est répartie en cluster, l’application et nom de la phase. Identifier les pics de latence de tâche dans le graphique pour déterminer quelles tâches sont empêchaient l’achèvement de l’étape.

![Graphique affichant la latence de phase](./_images/grafana-stage-latency.png)

Le graphique de débit de cluster affiche le nombre de travaux, des étapes et des tâches sont terminées par minute. Cela vous aide à comprendre la charge de travail en termes de nombre relatif des étapes et des tâches par travail. Ici, vous pouvez voir que le nombre de tâches par minute plages entre 2 et 6, tandis que le nombre d’étapes est environ 12 &ndash; 24 par minute.

![Débit de cluster graphique montrant](./_images/grafana-cluster-throughput.png)

## <a name="sum-of-task-execution-latency"></a>Somme de la latence de l’exécution de tâches

Cette visualisation affiche la somme de la latence de l’exécution de tâches par hôte en cours d’exécution sur un cluster. Utilisez ce graphique pour détecter les tâches qui s’exécutent lentement en raison de l’hôte ralentit le sur un cluster ou une misallocation de tâches par exécuteur. Dans le graphique suivant, la plupart des ordinateurs hôtes ont une somme d’environ 30 secondes. Toutefois, deux des ordinateurs hôtes ont des sommes pointez environ 10 minutes. Les hôtes sont lentes ou le nombre de tâches par exécuteur est misallocated.

![Somme de montrant graphique l’exécution des tâches par hôte](./_images/grafana-sum-task-exec.png)

Le nombre de tâches par exécuteur montre que deux exécuteurs sont affectés un nombre disproportionné de tâches, provoquant un goulet d’étranglement.

![Graphique illustrant les tâches par exécuteur](./_images/grafana-tasks-per-exec.png)

## <a name="task-metrics-per-stage"></a>Métriques de tâche par phase

La visualisation de métriques de tâche donne la répartition des coûts pour une exécution de tâches. Vous pouvez l’utiliser voir dans le temps passé sur des tâches telles que la sérialisation et la désérialisation. Ces données peuvent afficher des opportunités d’optimisation &mdash; , par exemple, en utilisant [diffusez les variables](https://spark.apache.org/docs/2.2.0/rdd-programming-guide.html#broadcast-variables) afin d’éviter l’envoi de données. Les métriques de tâche affichent également la taille des données aléatoires pour une tâche, et le mélange lire et écrire des heures. Si ces valeurs sont élevées, cela signifie qu’une grande quantité de données se déplace sur le réseau.

Une autre métrique de tâche est le délai de planificateur, qui mesure le temps nécessaire pour planifier une tâche. Dans l’idéal, cette valeur doit être faible par rapport à l’heure de calcul exécuteur, qui est le temps passé à réellement l’exécution de la tâche.

Le graphique suivant un délai de planificateur (3.7 s) qui dépasse le temps de calcul exécuteur (1.1 s). Cela signifie que plus de temps est passé en attente pour être planifiée que de faire le travail réel des tâches.

![Graphique montrant les métriques de tâche par phase](./_images/grafana-metrics-per-stage.png)

Dans ce cas, le problème a été provoqué par un trop grand nombre de partitions, qui a provoqué une charge importante. Réduction du nombre de partitions réduit le temps de retard du planificateur. Le montre le graphique suivant que la plupart du temps passé dans l’exécution de la tâche.

![Graphique montrant les métriques de tâche par phase](./_images/grafana-metrics-per-stage2.png)

## <a name="streaming-throughput-and-latency"></a>Débit et la latence de la diffusion en continu

Débit de diffusion en continu est directement liée à la diffusion en continu structurée. Il existe deux métriques importantes associées à la vitesse de streaming : Lignes d’entrée par secondes et traités les lignes par seconde. Si les lignes d’entrée par seconde distance les autres secteurs lignes traitées par seconde, cela signifie que le système de traitement de flux est en retard. En outre, si les données d’entrée provient d’Event Hubs ou Kafka, puis les lignes d’entrée par seconde doivent suivre le taux d’ingestion de données au niveau frontal.

Deux travaux peut avoir un débit cluster similaire mais très différentes mesures de diffusion en continu. La capture d’écran suivante montre deux différentes charges de travail. Elles sont similaires en termes de débit de cluster (des travaux, des étapes et des tâches par minute). Mais la seconde exécution traite 12 000 lignes par seconde contre 4 000 lignes par seconde.

![Vitesse de streaming montrant graphique](./_images/grafana-streaming-throughput.png)

Débit de diffusion en continu est souvent une meilleure métrique d’entreprise que le débit de cluster, car il mesure le nombre d’enregistrements de données sont traitées.

## <a name="resource-consumption-per-executor"></a>Consommation des ressources par exécuteur

Ces mesures vous aider à comprendre le travail qui exécute chaque exécuteur.

**Métriques de pourcentage** mesurer combien de temps un exécuteur est occupé sur divers éléments, exprimée sous la forme d’un ratio de temps passé par rapport à l’heure de calcul exécuteur globale. Les mesures sont :

- % Temps de sérialiser
- % Temps de désérialiser
- % Exécuteur temps UC
- Pourcentage du temps JVM

Ces visualisations indiquent la quantité chacune de ces métriques contribue à l’exécuteur globale de traitement.

![Graphique montrant les métriques de pourcentage](./_images/grafana-percentage.png)

**Lecture aléatoire des mesures** sont de mesures liées aux données mélange entre les exécuteurs.

- E/s de lecture aléatoire
- La réorganisation de la mémoire
- Utilisation du système de fichiers
- Utilisation du disque

## <a name="common-performance-bottlenecks"></a>Goulots d’étranglement de performances courants

Deux courants les goulots d’étranglement dans Spark sont *tâches délais a* et un *nombre de partitions de lecture aléatoire non optimales*.

### <a name="task-stragglers"></a>Délais a tâche

Les étapes d’un travail sont exécutées séquentiellement, avec des étapes antérieures bloque les étapes ultérieures. Si une tâche s’exécute une partition aléatoire plus lentement que d’autres tâches, toutes les tâches dans le cluster doivent attendre la tâche lente à rattraper avant l’étape peut se terminer. Cela peut se produire pour les raisons suivantes :

1. Un hôte ou un groupe d’hôtes exécutent trop lentement. Symptômes suivants : Tâche haute, étape, ou latence du travail et un débit faible de cluster. La somme des latences de tâches par ordinateur hôte ne sont pas distribuée uniformément. Toutefois, la consommation des ressources est distribuée uniformément entre les exécuteurs.

1. Tâches ont une agrégation coûteuse à exécuter (inclinaison de données). Symptômes suivants : Tâche haute, étape, ou travail et un débit faible de cluster, mais la somme de latences par hôte est distribué uniformément. La consommation des ressources est distribuée uniformément entre les exécuteurs.

1. Si les partitions sont de taille inégale, une plus grande partition peut entraîner l’exécution de tâche non équilibrées (partition inclinaison). Symptômes suivants : La consommation des ressources exécuteur est élevée par rapport aux autres exécuteurs en cours d’exécution sur le cluster. Toutes les tâches en cours d’exécution sur cet exécuteur exécutera lentement et maintenez l’exécution de l’étape dans le pipeline. Ces étapes sont dits se situer *les barrières à l’étape*.

### <a name="non-optimal-shuffle-partition-count"></a>Nombre de partitions de lecture aléatoire non optimales

Pendant une requête de diffusion en continu structurée, l’attribution d’une tâche à un exécuteur est une opération nécessitant de nombreuses ressources pour le cluster. Si les données de lecture aléatoire n’est pas la taille optimale, la durée du délai d’une tâche ralentiront débit et la latence. S’il existe trop peu de partitions, les cœurs dans le cluster qui peuvent entraîner l’inefficacité de traitement sont sous-utilisés. À l’inverse, s’il y a trop de partitions, il est beaucoup de temps de gestion à un petit nombre de tâches.

Utiliser les mesures de consommation de ressources pour résoudre les problèmes d’inclinaison de la partition et misallocation d’exécuteurs sur le cluster. Si une partition est déréglée, les ressources de l’exécuteur va être élevés par rapport aux autres exécuteurs en cours d’exécution sur le cluster.

Par exemple, le graphique suivant montre que la mémoire utilisée par le mélange sur les deux premières exécuteurs est 90 X plus grand que les autres exécuteurs :

![Graphique montrant les métriques de pourcentage](./_images/grafana-shuffle-memory.png)