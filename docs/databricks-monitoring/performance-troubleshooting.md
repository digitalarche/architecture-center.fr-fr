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
ms.sourcegitcommit: 1a3cc91530d56731029ea091db1f15d41ac056af
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/03/2019
ms.locfileid: "58888013"
---
# <a name="troubleshoot-performance-bottlenecks-in-azure-databricks"></a><span data-ttu-id="d1aef-103">Résoudre les goulots d’étranglement de performances dans Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="d1aef-103">Troubleshoot performance bottlenecks in Azure Databricks</span></span>

<span data-ttu-id="d1aef-104">Cet article décrit comment utiliser des tableaux de bord de surveillance pour trouver les goulots d’étranglement de performances dans des travaux Spark sur Azure Databricks.</span><span class="sxs-lookup"><span data-stu-id="d1aef-104">This article describes how to use monitoring dashboards to find performance bottlenecks in Spark jobs on Azure Databricks.</span></span>

<span data-ttu-id="d1aef-105">[Azure Databricks](/azure/azure-databricks/) est un [Apache Spark](https://spark.apache.org/)– en fonction du service d’analytique qui facilite la rapidement développer et déployer l’analytique des données volumineuses.</span><span class="sxs-lookup"><span data-stu-id="d1aef-105">[Azure Databricks](/azure/azure-databricks/) is an [Apache Spark](https://spark.apache.org/)–based analytics service that makes it easy to rapidly develop and deploy big data analytics.</span></span> <span data-ttu-id="d1aef-106">Surveillance et dépannage des problèmes de performances sont une critique lors de l’exploitation des charges de travail de production Azure Databricks.</span><span class="sxs-lookup"><span data-stu-id="d1aef-106">Monitoring and troubleshooting performance issues is a critical when operating production Azure Databricks workloads.</span></span> <span data-ttu-id="d1aef-107">Pour identifier les problèmes de performances courants, il est conseillé d’utiliser la surveillance de visualisations basées sur les données de télémétrie.</span><span class="sxs-lookup"><span data-stu-id="d1aef-107">To identify common performance issues, it's helpful to use monitoring visualizations based on telemetry data.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="d1aef-108">Conditions préalables</span><span class="sxs-lookup"><span data-stu-id="d1aef-108">Prerequisites</span></span>

<span data-ttu-id="d1aef-109">Pour configurer les tableaux de bord Grafana présenté dans cet article :</span><span class="sxs-lookup"><span data-stu-id="d1aef-109">To set up the Grafana dashboards shown in this article:</span></span>

- <span data-ttu-id="d1aef-110">Configurer votre cluster Databricks pour envoyer la télémétrie à un espace de travail Analytique de journal, à l’aide de la bibliothèque d’analyse Azure Databricks.</span><span class="sxs-lookup"><span data-stu-id="d1aef-110">Configure your Databricks cluster to send telemetry to a Log Analytics workspace, using the Azure Databricks Monitoring Library.</span></span> <span data-ttu-id="d1aef-111">Pour plus d’informations, consultez [configurer Azure Databricks pour envoyer des métriques à Azure Monitor](./configure-cluster.md).</span><span class="sxs-lookup"><span data-stu-id="d1aef-111">For details, see [Configure Azure Databricks to send metrics to Azure Monitor](./configure-cluster.md).</span></span>

- <span data-ttu-id="d1aef-112">Déployez Grafana dans une machine virtuelle.</span><span class="sxs-lookup"><span data-stu-id="d1aef-112">Deploy Grafana in a virtual machine.</span></span> <span data-ttu-id="d1aef-113">Consultez [utiliser des tableaux de bord pour visualiser les métriques d’Azure Databricks](./dashboards.md).</span><span class="sxs-lookup"><span data-stu-id="d1aef-113">See [Use dashboards to visualize Azure Databricks metrics](./dashboards.md).</span></span>

<span data-ttu-id="d1aef-114">Le tableau de bord Grafana est déployé comprend un ensemble de visualisations de série chronologique.</span><span class="sxs-lookup"><span data-stu-id="d1aef-114">The Grafana dashboard that is deployed includes a set of time-series visualizations.</span></span> <span data-ttu-id="d1aef-115">Chaque graphique est tracé de séries chronologiques des mesures relatives à une tâche Apache Spark, les étapes de travail et des tâches qui composent chaque étape.</span><span class="sxs-lookup"><span data-stu-id="d1aef-115">Each graph is time-series plot of metrics related to an Apache Spark job, the stages of the job, and tasks that make up each stage.</span></span>

## <a name="azure-databricks-performance-overview"></a><span data-ttu-id="d1aef-116">Vue d’ensemble des performances Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="d1aef-116">Azure Databricks performance overview</span></span>

<span data-ttu-id="d1aef-117">Azure Databricks est basée sur Apache Spark, un système informatique distribué à usage général.</span><span class="sxs-lookup"><span data-stu-id="d1aef-117">Azure Databricks is based on Apache Spark, a general-purpose distributed computing system.</span></span> <span data-ttu-id="d1aef-118">Code d’application, appelé un **travail**, s’exécute sur un cluster Apache Spark, coordonné par le Gestionnaire du cluster.</span><span class="sxs-lookup"><span data-stu-id="d1aef-118">Application code, known as a **job**, executes on an Apache Spark cluster, coordinated by the cluster manager.</span></span> <span data-ttu-id="d1aef-119">En règle générale, un travail est l’unité de plus haut niveau de calcul.</span><span class="sxs-lookup"><span data-stu-id="d1aef-119">In general, a job is the highest-level unit of computation.</span></span> <span data-ttu-id="d1aef-120">Une tâche représente l’opération complète effectuée par l’application Spark.</span><span class="sxs-lookup"><span data-stu-id="d1aef-120">A job represents the complete operation performed by the Spark application.</span></span> <span data-ttu-id="d1aef-121">Une opération standard inclut la lecture des données à partir d’une source de, appliquant des transformations de données et écrire les résultats vers le stockage ou une autre destination.</span><span class="sxs-lookup"><span data-stu-id="d1aef-121">A typical operation includes reading data from a source, applying data transformations, and writing the results to storage or another destination.</span></span>

<span data-ttu-id="d1aef-122">Travaux sont décomposent en **étapes**.</span><span class="sxs-lookup"><span data-stu-id="d1aef-122">Jobs are broken down into **stages**.</span></span> <span data-ttu-id="d1aef-123">Le travail progresse vers les étapes séquentiellement, ce qui signifie que les phases ultérieures doivent attendre les étapes précédentes terminer.</span><span class="sxs-lookup"><span data-stu-id="d1aef-123">The job advances through the stages sequentially, which means that later stages must wait for earlier stages to complete.</span></span> <span data-ttu-id="d1aef-124">Étapes contiennent des groupes d’identiques **tâches** qui peuvent être exécutées en parallèle sur plusieurs nœuds du cluster Spark.</span><span class="sxs-lookup"><span data-stu-id="d1aef-124">Stages contain groups of identical **tasks** that can be executed in parallel on multiple nodes of the Spark cluster.</span></span> <span data-ttu-id="d1aef-125">Les tâches sont l’unité plus granulaire d’exécution qui ont lieu sur un sous-ensemble des données.</span><span class="sxs-lookup"><span data-stu-id="d1aef-125">Tasks are the most granular unit of execution taking place on a subset of the data.</span></span>

<span data-ttu-id="d1aef-126">Les sections suivantes décrivent certaines visualisations de tableau de bord qui sont utiles pour la résolution des problèmes de performances.</span><span class="sxs-lookup"><span data-stu-id="d1aef-126">The next sections describe some dashboard visualizations that are useful for performance troubleshooting.</span></span>

## <a name="job-and-stage-latency"></a><span data-ttu-id="d1aef-127">Latence de travail et de la scène</span><span class="sxs-lookup"><span data-stu-id="d1aef-127">Job and stage latency</span></span>

<span data-ttu-id="d1aef-128">Latence du travail est la durée d’une exécution du travail à partir de lorsqu’il démarre jusqu'à la fin.</span><span class="sxs-lookup"><span data-stu-id="d1aef-128">Job latency is the duration of a job execution from when it starts until it completes.</span></span> <span data-ttu-id="d1aef-129">Elle est affichée en tant que centiles d’une exécution du travail par ID de cluster et des applications, pour permettre la visualisation des valeurs hors norme.</span><span class="sxs-lookup"><span data-stu-id="d1aef-129">It is shown as percentiles of a job execution per cluster and application ID, to allow the visualization of outliers.</span></span> <span data-ttu-id="d1aef-130">Le graphique suivant montre un historique des travaux où le 90e centile atteint 50 secondes, même si la 50e centile a été constamment environ 10 secondes.</span><span class="sxs-lookup"><span data-stu-id="d1aef-130">The following graph shows a job history where the 90th percentile reached 50 seconds, even though the 50th percentile was consistently around 10 seconds.</span></span>

![Graphique illustrant la latence du travail](./_images/grafana-job-latency.png)

<span data-ttu-id="d1aef-132">Examiner l’exécution du travail en cluster et des applications, recherchez des pics de latence.</span><span class="sxs-lookup"><span data-stu-id="d1aef-132">Investigate job execution by cluster and application, looking for spikes in latency.</span></span> <span data-ttu-id="d1aef-133">Une fois que les clusters et des applications avec une latence élevée sont identifiées, passer à examiner la latence de la scène.</span><span class="sxs-lookup"><span data-stu-id="d1aef-133">Once clusters and applications with high latency are identified, move on to investigate stage latency.</span></span>

<span data-ttu-id="d1aef-134">Latence de l’étape est également affichée en tant que centiles pour permettre la visualisation des valeurs hors norme.</span><span class="sxs-lookup"><span data-stu-id="d1aef-134">Stage latency is also shown as percentiles to allow the visualization of outliers.</span></span> <span data-ttu-id="d1aef-135">Latence de la scène est répartie en cluster, l’application et nom de la phase.</span><span class="sxs-lookup"><span data-stu-id="d1aef-135">Stage latency is broken out by cluster, application, and stage name.</span></span> <span data-ttu-id="d1aef-136">Identifier les pics de latence de tâche dans le graphique pour déterminer quelles tâches sont empêchaient l’achèvement de l’étape.</span><span class="sxs-lookup"><span data-stu-id="d1aef-136">Identify spikes in task latency in the graph to determine which tasks are holding back completion of the stage.</span></span>

![Graphique affichant la latence de phase](./_images/grafana-stage-latency.png)

<span data-ttu-id="d1aef-138">Le graphique de débit de cluster affiche le nombre de travaux, des étapes et des tâches sont terminées par minute.</span><span class="sxs-lookup"><span data-stu-id="d1aef-138">The cluster throughput graph shows the number of jobs, stages, and tasks completed per minute.</span></span> <span data-ttu-id="d1aef-139">Cela vous aide à comprendre la charge de travail en termes de nombre relatif des étapes et des tâches par travail.</span><span class="sxs-lookup"><span data-stu-id="d1aef-139">This helps you to understand the workload in terms of the relative number of stages and tasks per job.</span></span> <span data-ttu-id="d1aef-140">Ici, vous pouvez voir que le nombre de tâches par minute plages entre 2 et 6, tandis que le nombre d’étapes est environ 12 &ndash; 24 par minute.</span><span class="sxs-lookup"><span data-stu-id="d1aef-140">Here you can see that the number of jobs per minute ranges between 2 and 6, while the number of stages is about 12 &ndash; 24 per minute.</span></span>

![Débit de cluster graphique montrant](./_images/grafana-cluster-throughput.png)

## <a name="sum-of-task-execution-latency"></a><span data-ttu-id="d1aef-142">Somme de la latence de l’exécution de tâches</span><span class="sxs-lookup"><span data-stu-id="d1aef-142">Sum of task execution latency</span></span>

<span data-ttu-id="d1aef-143">Cette visualisation affiche la somme de la latence de l’exécution de tâches par hôte en cours d’exécution sur un cluster.</span><span class="sxs-lookup"><span data-stu-id="d1aef-143">This visualization shows the sum of task execution latency per host running on a cluster.</span></span> <span data-ttu-id="d1aef-144">Utilisez ce graphique pour détecter les tâches qui s’exécutent lentement en raison de l’hôte ralentit le sur un cluster ou une misallocation de tâches par exécuteur.</span><span class="sxs-lookup"><span data-stu-id="d1aef-144">Use this graph to detect tasks that run slowly due to the host slowing down on a cluster, or a misallocation of tasks per executor.</span></span> <span data-ttu-id="d1aef-145">Dans le graphique suivant, la plupart des ordinateurs hôtes ont une somme d’environ 30 secondes.</span><span class="sxs-lookup"><span data-stu-id="d1aef-145">In the following graph, most of the hosts have a sum of about 30 seconds.</span></span> <span data-ttu-id="d1aef-146">Toutefois, deux des ordinateurs hôtes ont des sommes pointez environ 10 minutes.</span><span class="sxs-lookup"><span data-stu-id="d1aef-146">However, two of the hosts have sums that hover around 10 minutes.</span></span> <span data-ttu-id="d1aef-147">Les hôtes sont lentes ou le nombre de tâches par exécuteur est misallocated.</span><span class="sxs-lookup"><span data-stu-id="d1aef-147">Either the hosts are running slow or the number of tasks per executor is misallocated.</span></span>

![Somme de montrant graphique l’exécution des tâches par hôte](./_images/grafana-sum-task-exec.png)

<span data-ttu-id="d1aef-149">Le nombre de tâches par exécuteur montre que deux exécuteurs sont affectés un nombre disproportionné de tâches, provoquant un goulet d’étranglement.</span><span class="sxs-lookup"><span data-stu-id="d1aef-149">The number of tasks per executor shows that two executors are assigned a disproportionate number of tasks, causing a bottleneck.</span></span>

![Graphique illustrant les tâches par exécuteur](./_images/grafana-tasks-per-exec.png)

## <a name="task-metrics-per-stage"></a><span data-ttu-id="d1aef-151">Métriques de tâche par phase</span><span class="sxs-lookup"><span data-stu-id="d1aef-151">Task metrics per stage</span></span>

<span data-ttu-id="d1aef-152">La visualisation de métriques de tâche donne la répartition des coûts pour une exécution de tâches.</span><span class="sxs-lookup"><span data-stu-id="d1aef-152">The task metrics visualization gives the cost breakdown for a task execution.</span></span> <span data-ttu-id="d1aef-153">Vous pouvez l’utiliser voir dans le temps passé sur des tâches telles que la sérialisation et la désérialisation.</span><span class="sxs-lookup"><span data-stu-id="d1aef-153">You can use it see the relative time spent on tasks such as serialization and deserialization.</span></span> <span data-ttu-id="d1aef-154">Ces données peuvent afficher des opportunités d’optimisation &mdash; , par exemple, en utilisant [diffusez les variables](https://spark.apache.org/docs/2.2.0/rdd-programming-guide.html#broadcast-variables) afin d’éviter l’envoi de données.</span><span class="sxs-lookup"><span data-stu-id="d1aef-154">This data might show opportunities to optimize &mdash; for example, by using [broadcast variables](https://spark.apache.org/docs/2.2.0/rdd-programming-guide.html#broadcast-variables) to avoid shipping data.</span></span> <span data-ttu-id="d1aef-155">Les métriques de tâche affichent également la taille des données aléatoires pour une tâche, et le mélange lire et écrire des heures.</span><span class="sxs-lookup"><span data-stu-id="d1aef-155">The task metrics also show the shuffle data size for a task, and the shuffle read and write times.</span></span> <span data-ttu-id="d1aef-156">Si ces valeurs sont élevées, cela signifie qu’une grande quantité de données se déplace sur le réseau.</span><span class="sxs-lookup"><span data-stu-id="d1aef-156">If these values are high, it means that a lot of data is moving across the network.</span></span>

<span data-ttu-id="d1aef-157">Une autre métrique de tâche est le délai de planificateur, qui mesure le temps nécessaire pour planifier une tâche.</span><span class="sxs-lookup"><span data-stu-id="d1aef-157">Another task metric is the scheduler delay, which measures how long it takes to schedule a task.</span></span> <span data-ttu-id="d1aef-158">Dans l’idéal, cette valeur doit être faible par rapport à l’heure de calcul exécuteur, qui est le temps passé à réellement l’exécution de la tâche.</span><span class="sxs-lookup"><span data-stu-id="d1aef-158">Ideally, this value should be low compared to the executor compute time, which is the time spent actually executing the task.</span></span>

<span data-ttu-id="d1aef-159">Le graphique suivant un délai de planificateur (3.7 s) qui dépasse le temps de calcul exécuteur (1.1 s).</span><span class="sxs-lookup"><span data-stu-id="d1aef-159">The following graph shows a scheduler delay time (3.7 s) that exceeds the executor compute time (1.1 s).</span></span> <span data-ttu-id="d1aef-160">Cela signifie que plus de temps est passé en attente pour être planifiée que de faire le travail réel des tâches.</span><span class="sxs-lookup"><span data-stu-id="d1aef-160">That means more time is spent waiting for tasks to be scheduled than doing the actual work.</span></span>

![Graphique montrant les métriques de tâche par phase](./_images/grafana-metrics-per-stage.png)

<span data-ttu-id="d1aef-162">Dans ce cas, le problème a été provoqué par un trop grand nombre de partitions, qui a provoqué une charge importante.</span><span class="sxs-lookup"><span data-stu-id="d1aef-162">In this case, the problem was caused by having too many partitions, which caused a lot of overhead.</span></span> <span data-ttu-id="d1aef-163">Réduction du nombre de partitions réduit le temps de retard du planificateur.</span><span class="sxs-lookup"><span data-stu-id="d1aef-163">Reducing the number of partitions lowered the scheduler delay time.</span></span> <span data-ttu-id="d1aef-164">Le montre le graphique suivant que la plupart du temps passé dans l’exécution de la tâche.</span><span class="sxs-lookup"><span data-stu-id="d1aef-164">The next graph shows that most of the time is spent executing the task.</span></span>

![Graphique montrant les métriques de tâche par phase](./_images/grafana-metrics-per-stage2.png)

## <a name="streaming-throughput-and-latency"></a><span data-ttu-id="d1aef-166">Débit et la latence de la diffusion en continu</span><span class="sxs-lookup"><span data-stu-id="d1aef-166">Streaming throughput and latency</span></span>

<span data-ttu-id="d1aef-167">Débit de diffusion en continu est directement liée à la diffusion en continu structurée.</span><span class="sxs-lookup"><span data-stu-id="d1aef-167">Streaming throughput is directly related to structured streaming.</span></span> <span data-ttu-id="d1aef-168">Il existe deux métriques importantes associées à la vitesse de streaming : Lignes d’entrée par secondes et traités les lignes par seconde.</span><span class="sxs-lookup"><span data-stu-id="d1aef-168">There are two important metrics associated with streaming throughput: Input rows per second and processed rows per second.</span></span> <span data-ttu-id="d1aef-169">Si les lignes d’entrée par seconde distance les autres secteurs lignes traitées par seconde, cela signifie que le système de traitement de flux est en retard.</span><span class="sxs-lookup"><span data-stu-id="d1aef-169">If input rows per second outpaces processed rows per second, it means the stream processing system is falling behind.</span></span> <span data-ttu-id="d1aef-170">En outre, si les données d’entrée provient d’Event Hubs ou Kafka, puis les lignes d’entrée par seconde doivent suivre le taux d’ingestion de données au niveau frontal.</span><span class="sxs-lookup"><span data-stu-id="d1aef-170">Also, if the input data comes from Event Hubs or Kafka, then input rows per second should keep up with the data ingestion rate at the front end.</span></span>

<span data-ttu-id="d1aef-171">Deux travaux peut avoir un débit cluster similaire mais très différentes mesures de diffusion en continu.</span><span class="sxs-lookup"><span data-stu-id="d1aef-171">Two jobs can have similar cluster throughput but very different streaming metrics.</span></span> <span data-ttu-id="d1aef-172">La capture d’écran suivante montre deux différentes charges de travail.</span><span class="sxs-lookup"><span data-stu-id="d1aef-172">The following screenshot shows two different workloads.</span></span> <span data-ttu-id="d1aef-173">Elles sont similaires en termes de débit de cluster (des travaux, des étapes et des tâches par minute).</span><span class="sxs-lookup"><span data-stu-id="d1aef-173">They are similar in terms of cluster throughput (jobs, stages, and tasks per minute).</span></span> <span data-ttu-id="d1aef-174">Mais la seconde exécution traite 12 000 lignes par seconde contre 4 000 lignes par seconde.</span><span class="sxs-lookup"><span data-stu-id="d1aef-174">But the second run processes 12,000 rows/sec versus 4,000 rows/sec.</span></span>

![Vitesse de streaming montrant graphique](./_images/grafana-streaming-throughput.png)

<span data-ttu-id="d1aef-176">Débit de diffusion en continu est souvent une meilleure métrique d’entreprise que le débit de cluster, car il mesure le nombre d’enregistrements de données sont traitées.</span><span class="sxs-lookup"><span data-stu-id="d1aef-176">Streaming throughput is often a better business metric than cluster throughput, because it measures the number of data records that are processed.</span></span>

## <a name="resource-consumption-per-executor"></a><span data-ttu-id="d1aef-177">Consommation des ressources par exécuteur</span><span class="sxs-lookup"><span data-stu-id="d1aef-177">Resource consumption per executor</span></span>

<span data-ttu-id="d1aef-178">Ces mesures vous aider à comprendre le travail qui exécute chaque exécuteur.</span><span class="sxs-lookup"><span data-stu-id="d1aef-178">These metrics help to understand the work that each executor performs.</span></span>

<span data-ttu-id="d1aef-179">**Métriques de pourcentage** mesurer combien de temps un exécuteur est occupé sur divers éléments, exprimée sous la forme d’un ratio de temps passé par rapport à l’heure de calcul exécuteur globale.</span><span class="sxs-lookup"><span data-stu-id="d1aef-179">**Percentage metrics** measure how much time an executor spends on various things, expressed as a ratio of time spent versus the overall executor compute time.</span></span> <span data-ttu-id="d1aef-180">Les mesures sont :</span><span class="sxs-lookup"><span data-stu-id="d1aef-180">The metrics are:</span></span>

- <span data-ttu-id="d1aef-181">% Temps de sérialiser</span><span class="sxs-lookup"><span data-stu-id="d1aef-181">% Serialize time</span></span>
- <span data-ttu-id="d1aef-182">% Temps de désérialiser</span><span class="sxs-lookup"><span data-stu-id="d1aef-182">% Deserialize time</span></span>
- <span data-ttu-id="d1aef-183">% Exécuteur temps UC</span><span class="sxs-lookup"><span data-stu-id="d1aef-183">% CPU executor time</span></span>
- <span data-ttu-id="d1aef-184">Pourcentage du temps JVM</span><span class="sxs-lookup"><span data-stu-id="d1aef-184">% JVM time</span></span>

<span data-ttu-id="d1aef-185">Ces visualisations indiquent la quantité chacune de ces métriques contribue à l’exécuteur globale de traitement.</span><span class="sxs-lookup"><span data-stu-id="d1aef-185">These visualizations show how much each of these metrics contributes to overall executor processing.</span></span>

![Graphique montrant les métriques de pourcentage](./_images/grafana-percentage.png)

<span data-ttu-id="d1aef-187">**Lecture aléatoire des mesures** sont de mesures liées aux données mélange entre les exécuteurs.</span><span class="sxs-lookup"><span data-stu-id="d1aef-187">**Shuffle metrics** are metrics related to data shuffling across the executors.</span></span>

- <span data-ttu-id="d1aef-188">E/s de lecture aléatoire</span><span class="sxs-lookup"><span data-stu-id="d1aef-188">Shuffle I/O</span></span>
- <span data-ttu-id="d1aef-189">La réorganisation de la mémoire</span><span class="sxs-lookup"><span data-stu-id="d1aef-189">Shuffle memory</span></span>
- <span data-ttu-id="d1aef-190">Utilisation du système de fichiers</span><span class="sxs-lookup"><span data-stu-id="d1aef-190">File system usage</span></span>
- <span data-ttu-id="d1aef-191">Utilisation du disque</span><span class="sxs-lookup"><span data-stu-id="d1aef-191">Disk usage</span></span>

## <a name="common-performance-bottlenecks"></a><span data-ttu-id="d1aef-192">Goulots d’étranglement de performances courants</span><span class="sxs-lookup"><span data-stu-id="d1aef-192">Common performance bottlenecks</span></span>

<span data-ttu-id="d1aef-193">Deux courants les goulots d’étranglement dans Spark sont *tâches délais a* et un *nombre de partitions de lecture aléatoire non optimales*.</span><span class="sxs-lookup"><span data-stu-id="d1aef-193">Two common performance bottlenecks in Spark are *task stragglers* and a *non-optimal shuffle partition count*.</span></span>

### <a name="task-stragglers"></a><span data-ttu-id="d1aef-194">Délais a tâche</span><span class="sxs-lookup"><span data-stu-id="d1aef-194">Task stragglers</span></span>

<span data-ttu-id="d1aef-195">Les étapes d’un travail sont exécutées séquentiellement, avec des étapes antérieures bloque les étapes ultérieures.</span><span class="sxs-lookup"><span data-stu-id="d1aef-195">The stages in a job are executed sequentially, with earlier stages blocking later stages.</span></span> <span data-ttu-id="d1aef-196">Si une tâche s’exécute une partition aléatoire plus lentement que d’autres tâches, toutes les tâches dans le cluster doivent attendre la tâche lente à rattraper avant l’étape peut se terminer.</span><span class="sxs-lookup"><span data-stu-id="d1aef-196">If one task executes a shuffle partition more slowly than other tasks, all tasks in the cluster must wait for the slow task to catch up before the stage can end.</span></span> <span data-ttu-id="d1aef-197">Cela peut se produire pour les raisons suivantes :</span><span class="sxs-lookup"><span data-stu-id="d1aef-197">This can happen for the following reasons:</span></span>

1. <span data-ttu-id="d1aef-198">Un hôte ou un groupe d’hôtes exécutent trop lentement.</span><span class="sxs-lookup"><span data-stu-id="d1aef-198">A host or group of hosts are running slow.</span></span> <span data-ttu-id="d1aef-199">Symptômes suivants : Tâche haute, étape, ou latence du travail et un débit faible de cluster.</span><span class="sxs-lookup"><span data-stu-id="d1aef-199">Symptoms: High task, stage, or job latency and low cluster throughput.</span></span> <span data-ttu-id="d1aef-200">La somme des latences de tâches par ordinateur hôte ne sont pas distribuée uniformément.</span><span class="sxs-lookup"><span data-stu-id="d1aef-200">The summation of tasks latencies per host won't be evenly distributed.</span></span> <span data-ttu-id="d1aef-201">Toutefois, la consommation des ressources est distribuée uniformément entre les exécuteurs.</span><span class="sxs-lookup"><span data-stu-id="d1aef-201">However, resource consumption will be evenly distributed across executors.</span></span>

1. <span data-ttu-id="d1aef-202">Tâches ont une agrégation coûteuse à exécuter (inclinaison de données).</span><span class="sxs-lookup"><span data-stu-id="d1aef-202">Tasks have an expensive aggregation to execute (data skewing).</span></span> <span data-ttu-id="d1aef-203">Symptômes suivants : Tâche haute, étape, ou travail et un débit faible de cluster, mais la somme de latences par hôte est distribué uniformément.</span><span class="sxs-lookup"><span data-stu-id="d1aef-203">Symptoms: High task, stage, or job and low cluster throughput, but the summation of latencies per host is evenly distributed.</span></span> <span data-ttu-id="d1aef-204">La consommation des ressources est distribuée uniformément entre les exécuteurs.</span><span class="sxs-lookup"><span data-stu-id="d1aef-204">Resource consumption will be evenly distributed across executors.</span></span>

1. <span data-ttu-id="d1aef-205">Si les partitions sont de taille inégale, une plus grande partition peut entraîner l’exécution de tâche non équilibrées (partition inclinaison).</span><span class="sxs-lookup"><span data-stu-id="d1aef-205">If partitions are of unequal size, a larger partition may cause unbalanced task execution (partition skewing).</span></span> <span data-ttu-id="d1aef-206">Symptômes suivants : La consommation des ressources exécuteur est élevée par rapport aux autres exécuteurs en cours d’exécution sur le cluster.</span><span class="sxs-lookup"><span data-stu-id="d1aef-206">Symptoms: Executor resource consumption is high compared to other executors running on the cluster.</span></span> <span data-ttu-id="d1aef-207">Toutes les tâches en cours d’exécution sur cet exécuteur exécutera lentement et maintenez l’exécution de l’étape dans le pipeline.</span><span class="sxs-lookup"><span data-stu-id="d1aef-207">All tasks running on that executor will run slow and hold the stage execution in the pipeline.</span></span> <span data-ttu-id="d1aef-208">Ces étapes sont dits se situer *les barrières à l’étape*.</span><span class="sxs-lookup"><span data-stu-id="d1aef-208">Those stages are said to be *stage barriers*.</span></span>

### <a name="non-optimal-shuffle-partition-count"></a><span data-ttu-id="d1aef-209">Nombre de partitions de lecture aléatoire non optimales</span><span class="sxs-lookup"><span data-stu-id="d1aef-209">Non-optimal shuffle partition count</span></span>

<span data-ttu-id="d1aef-210">Pendant une requête de diffusion en continu structurée, l’attribution d’une tâche à un exécuteur est une opération nécessitant de nombreuses ressources pour le cluster.</span><span class="sxs-lookup"><span data-stu-id="d1aef-210">During a structured streaming query, the assignment of a task to an executor is a resource-intensive operation for the cluster.</span></span> <span data-ttu-id="d1aef-211">Si les données de lecture aléatoire n’est pas la taille optimale, la durée du délai d’une tâche ralentiront débit et la latence.</span><span class="sxs-lookup"><span data-stu-id="d1aef-211">If the shuffle data isn't the optimal size, the amount of delay for a task will negatively impact throughput and latency.</span></span> <span data-ttu-id="d1aef-212">S’il existe trop peu de partitions, les cœurs dans le cluster qui peuvent entraîner l’inefficacité de traitement sont sous-utilisés.</span><span class="sxs-lookup"><span data-stu-id="d1aef-212">If there are too few partitions, the cores in the cluster will be underutilized which can result in processing inefficiency.</span></span> <span data-ttu-id="d1aef-213">À l’inverse, s’il y a trop de partitions, il est beaucoup de temps de gestion à un petit nombre de tâches.</span><span class="sxs-lookup"><span data-stu-id="d1aef-213">Conversely, if there are too many partitions, there's a great deal of management overhead for a small number of tasks.</span></span>

<span data-ttu-id="d1aef-214">Utiliser les mesures de consommation de ressources pour résoudre les problèmes d’inclinaison de la partition et misallocation d’exécuteurs sur le cluster.</span><span class="sxs-lookup"><span data-stu-id="d1aef-214">Use the resource consumption metrics to troubleshoot partition skewing and misallocation of executors on the cluster.</span></span> <span data-ttu-id="d1aef-215">Si une partition est déréglée, les ressources de l’exécuteur va être élevés par rapport aux autres exécuteurs en cours d’exécution sur le cluster.</span><span class="sxs-lookup"><span data-stu-id="d1aef-215">If a partition is skewed, executor resources will be elevated in comparison to other executors running on the cluster.</span></span>

<span data-ttu-id="d1aef-216">Par exemple, le graphique suivant montre que la mémoire utilisée par le mélange sur les deux premières exécuteurs est 90 X plus grand que les autres exécuteurs :</span><span class="sxs-lookup"><span data-stu-id="d1aef-216">For example, the following graph shows that the memory used by shuffling on the first two executors is 90X bigger than the other executors:</span></span>

![Graphique montrant les métriques de pourcentage](./_images/grafana-shuffle-memory.png)