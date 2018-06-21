---
title: Antimodèle de persistance monolithique
description: Rassembler toutes les données d’une application dans un magasin de données unique peut dégrader les performances.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 7f04b9f0805c281068b6b2edaf040683773e6f6e
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2017
ms.locfileid: "24538679"
---
# <a name="monolithic-persistence-antipattern"></a><span data-ttu-id="c7911-103">Antimodèle de persistance monolithique</span><span class="sxs-lookup"><span data-stu-id="c7911-103">Monolithic Persistence antipattern</span></span>

<span data-ttu-id="c7911-104">Rassembler toutes les données d’une application dans un magasin de données unique peut nuire aux performances car cela provoque un conflit de ressources ou bien parce que le magasin de données n’est pas adapté à toutes les données.</span><span class="sxs-lookup"><span data-stu-id="c7911-104">Putting all of an application's data into a single data store can hurt performance, either because it leads to resource contention, or because the data store is not a good fit for some of the data.</span></span>

## <a name="problem-description"></a><span data-ttu-id="c7911-105">Description du problème</span><span class="sxs-lookup"><span data-stu-id="c7911-105">Problem description</span></span>

<span data-ttu-id="c7911-106">Historiquement, les applications ont souvent utilisé un magasin de données unique, quels que soit les types de données qu’elles devaient stocker.</span><span class="sxs-lookup"><span data-stu-id="c7911-106">Historically, applications have often used a single data store, regardless of the different types of data that the application might need to store.</span></span> <span data-ttu-id="c7911-107">Généralement, le but était de simplifier la conception de l’application, ou bien de s’adapter aux compétences de l’équipe de développement.</span><span class="sxs-lookup"><span data-stu-id="c7911-107">Usually this was done to simplify the application design, or else to match the existing skill set of the development team.</span></span> 

<span data-ttu-id="c7911-108">Les systèmes cloud modernes ont souvent des exigences fonctionnelles et non fonctionnelles supplémentaires et ils doivent stocker de nombreux types de données hétérogènes, tels que des documents, des images, des données mises en cache, des messages en attente, des journaux d’applications et des données de télémétrie.</span><span class="sxs-lookup"><span data-stu-id="c7911-108">Modern cloud-based systems often have additional functional and nonfunctional requirements, and need to store many heterogenous types of data, such as documents, images, cached data, queued messages, application logs, and telemetry.</span></span> <span data-ttu-id="c7911-109">Suivre l’approche traditionnelle et placer toutes ces informations dans un même magasin de données peut nuire aux performances, principalement pour deux raisons :</span><span class="sxs-lookup"><span data-stu-id="c7911-109">Following the traditional approach and putting all of this information into the same data store can hurt performance, for two main reasons:</span></span>

- <span data-ttu-id="c7911-110">Stocker et récupérer de grandes quantités de données non liées dans le même magasin de données peut provoquer des conflits, conduisant à un ralentissement du temps de réponse et à des échecs de connexion.</span><span class="sxs-lookup"><span data-stu-id="c7911-110">Storing and retrieving large amounts of unrelated data in the same data store can cause contention, which in turn leads to slow response times and connection failures.</span></span>
- <span data-ttu-id="c7911-111">Quel que soit le magasin de données choisi, il est possible qu’il ne soit le plus adapté à tous les types de données, ou bien il peut ne pas être optimisé pour les opérations réalisées par l’application.</span><span class="sxs-lookup"><span data-stu-id="c7911-111">Whichever data store is chosen, it might not be the best fit for all of the different types of data, or it might not be optimized for the operations that the application performs.</span></span> 

<span data-ttu-id="c7911-112">L’exemple suivant montre un contrôleur d’API Web ASP.NET qui ajoute un nouvel enregistrement à une base de données et enregistre également le résultat dans un journal.</span><span class="sxs-lookup"><span data-stu-id="c7911-112">The following example shows an ASP.NET Web API controller that adds a new record to a database and also records the result to a log.</span></span> <span data-ttu-id="c7911-113">Le journal est conservé dans la même base de données que les données d’entreprise.</span><span class="sxs-lookup"><span data-stu-id="c7911-113">The log is held in the same database as the business data.</span></span> <span data-ttu-id="c7911-114">Vous pouvez trouver l’exemple complet [ici][sample-app].</span><span class="sxs-lookup"><span data-stu-id="c7911-114">You can find the complete sample [here][sample-app].</span></span>

```csharp
public class MonoController : ApiController
{
    private static readonly string ProductionDb = ...;

    public async Task<IHttpActionResult> PostAsync([FromBody]string value)
    {
        await DataAccess.InsertPurchaseOrderHeaderAsync(ProductionDb);
        await DataAccess.LogAsync(ProductionDb, LogTableName);
        return Ok();
    }
}
```

<span data-ttu-id="c7911-115">La vitesse à laquelle les enregistrements de journal sont générés va probablement affecter les performances des opérations d’exploitation.</span><span class="sxs-lookup"><span data-stu-id="c7911-115">The rate at which log records are generated will probably affect the performance of the business operations.</span></span> <span data-ttu-id="c7911-116">Et si un autre composant, par exemple un moniteur de processus d’application, lit et traite régulièrement les données de journal, qui peuvent également affecter les opérations d’exploitation.</span><span class="sxs-lookup"><span data-stu-id="c7911-116">And if another component, such as an application process monitor, regularly reads and processes the log data, that can also affect the business operations.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="c7911-117">Comment corriger le problème</span><span class="sxs-lookup"><span data-stu-id="c7911-117">How to fix the problem</span></span>

<span data-ttu-id="c7911-118">Séparez les données en fonction de leur utilisation.</span><span class="sxs-lookup"><span data-stu-id="c7911-118">Separate data according to its use.</span></span> <span data-ttu-id="c7911-119">Pour chaque jeu de données, sélectionnez un magasin de données correspondant le mieux à leur future utilisation.</span><span class="sxs-lookup"><span data-stu-id="c7911-119">For each data set, select a data store that best matches how that data set will be used.</span></span> <span data-ttu-id="c7911-120">Dans l’exemple précédent, l’application doit se connecter à un magasin distinct de la base de données contenant les données d’entreprise :</span><span class="sxs-lookup"><span data-stu-id="c7911-120">In the previous example, the application should be logging to a separate store from the database that holds business data:</span></span> 

```csharp
public class PolyController : ApiController
{
    private static readonly string ProductionDb = ...;
    private static readonly string LogDb = ...;

    public async Task<IHttpActionResult> PostAsync([FromBody]string value)
    {
        await DataAccess.InsertPurchaseOrderHeaderAsync(ProductionDb);
        // Log to a different data store.
        await DataAccess.LogAsync(LogDb, LogTableName);
        return Ok();
    }
}
```

## <a name="considerations"></a><span data-ttu-id="c7911-121">Considérations</span><span class="sxs-lookup"><span data-stu-id="c7911-121">Considerations</span></span>

- <span data-ttu-id="c7911-122">Séparez les données selon leur accès et la manière dont elles sont utilisées.</span><span class="sxs-lookup"><span data-stu-id="c7911-122">Separate data by the way it is used and how it is accessed.</span></span> <span data-ttu-id="c7911-123">Par exemple, ne stockez pas des informations de journal et des données d’entreprise dans le même magasin de données.</span><span class="sxs-lookup"><span data-stu-id="c7911-123">For example, don't store log information and business data in the same data store.</span></span> <span data-ttu-id="c7911-124">Ces types de données ont des exigences et des modèles d’accès considérablement différentes.</span><span class="sxs-lookup"><span data-stu-id="c7911-124">These types of data have significantly different requirements and patterns of access.</span></span> <span data-ttu-id="c7911-125">Les enregistrements de journal sont séquentiels par nature, tandis que les données d’entreprise sont plus susceptibles de demander un accès aléatoire et sont souvent relationnelles.</span><span class="sxs-lookup"><span data-stu-id="c7911-125">Log records are inherently sequential, while business data is more likely to require random access, and is often relational.</span></span>

- <span data-ttu-id="c7911-126">Prenez en compte le modèle d’accès aux données pour chaque type de données.</span><span class="sxs-lookup"><span data-stu-id="c7911-126">Consider the data access pattern for each type of data.</span></span> <span data-ttu-id="c7911-127">Par exemple, stockez les rapports et documents formatés dans une base de données de documents, telle que [Cosmos DB][CosmosDB], mais utilisez [Azure Redis Cache Azure][Azure-cache] pour mettre en cache les données temporaires.</span><span class="sxs-lookup"><span data-stu-id="c7911-127">For example, store formatted reports and documents in a document database such as [Cosmos DB][CosmosDB], but use [Azure Redis Cache][Azure-cache] to cache temporary data.</span></span>

- <span data-ttu-id="c7911-128">Si vous suivez ces instructions mais que vous atteignez toujours les limites de la base de données, vous devez peut-être mettre à l’échelle la base de données.</span><span class="sxs-lookup"><span data-stu-id="c7911-128">If you follow this guidance but still reach the limits of the database, you may need to scale up the database.</span></span> <span data-ttu-id="c7911-129">Envisagez également la mise à l’échelle horizontale et le partitionnement de la charge entre les serveurs de base de données.</span><span class="sxs-lookup"><span data-stu-id="c7911-129">Also consider scaling horizontally and partitioning the load across database servers.</span></span> <span data-ttu-id="c7911-130">Toutefois, le partitionnement peut nécessiter une reconception de l’application.</span><span class="sxs-lookup"><span data-stu-id="c7911-130">However, partitioning may require redesigning the application.</span></span> <span data-ttu-id="c7911-131">Pour plus d’informations, consultez la page [Partitionnement des données][DataPartitioningGuidance].</span><span class="sxs-lookup"><span data-stu-id="c7911-131">For more information, see [Data partitioning][DataPartitioningGuidance].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="c7911-132">Comment détecter le problème</span><span class="sxs-lookup"><span data-stu-id="c7911-132">How to detect the problem</span></span>

<span data-ttu-id="c7911-133">Il est probable que le système ralentisse considérablement et échoue, car il manque de ressources, telles que les connexions de base de données.</span><span class="sxs-lookup"><span data-stu-id="c7911-133">The system will likely slow down dramatically and eventually fail, as the system runs out of resources such as database connections.</span></span>

<span data-ttu-id="c7911-134">Vous pouvez procédez de la manière suivante pour identifier la cause.</span><span class="sxs-lookup"><span data-stu-id="c7911-134">You can perform the following steps to help identify the cause.</span></span>

1. <span data-ttu-id="c7911-135">Instrumentez le système pour enregistrer les statistiques de performance clés.</span><span class="sxs-lookup"><span data-stu-id="c7911-135">Instrument the system to record the key performance statistics.</span></span> <span data-ttu-id="c7911-136">Capturez les informations de minutage pour chaque opération, ainsi que les points où l’application lit et écrit des données.</span><span class="sxs-lookup"><span data-stu-id="c7911-136">Capture timing information for each operation, as well as the points where the application reads and writes data.</span></span>
1. <span data-ttu-id="c7911-137">Si possible, surveillez le système en cours d’exécution pendant quelques jours dans un environnement de production pour obtenir une vision réelle de l’utilisation du système.</span><span class="sxs-lookup"><span data-stu-id="c7911-137">If possible, monitor the system running for a few days in a production environment to get a real-world view of how the system is used.</span></span> <span data-ttu-id="c7911-138">Si ce n’est pas possible, exécutez des tests de charge scriptés avec un volume réaliste d’utilisateurs virtuels exécutant une série classique d’opérations.</span><span class="sxs-lookup"><span data-stu-id="c7911-138">If this is not possible, run scripted load tests with a realistic volume of virtual users performing a typical series of operations.</span></span>
2. <span data-ttu-id="c7911-139">Utilisez les données de télémétrie pour identifier les périodes présentant des problèmes de performance.</span><span class="sxs-lookup"><span data-stu-id="c7911-139">Use the telemetry data to identify periods of poor performance.</span></span>
3. <span data-ttu-id="c7911-140">Identifiez les magasins de données consultés durant ces périodes.</span><span class="sxs-lookup"><span data-stu-id="c7911-140">Identify which data stores were accessed during those periods.</span></span>
4. <span data-ttu-id="c7911-141">Identifiez les ressources de stockage de données pouvant rencontrer des problèmes de contention.</span><span class="sxs-lookup"><span data-stu-id="c7911-141">Identify data storage resources that might be experiencing contention.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="c7911-142">Exemple de diagnostic</span><span class="sxs-lookup"><span data-stu-id="c7911-142">Example diagnosis</span></span>

<span data-ttu-id="c7911-143">Les sections suivantes appliquent ces étapes à l’exemple d’application décrit précédemment.</span><span class="sxs-lookup"><span data-stu-id="c7911-143">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="instrument-and-monitor-the-system"></a><span data-ttu-id="c7911-144">Instrumenter et surveiller le système</span><span class="sxs-lookup"><span data-stu-id="c7911-144">Instrument and monitor the system</span></span>

<span data-ttu-id="c7911-145">Le graphique suivant montre les résultats du test de charge de l’exemple d’application décrit précédemment.</span><span class="sxs-lookup"><span data-stu-id="c7911-145">The following graph shows the results of load testing the sample application described earlier.</span></span> <span data-ttu-id="c7911-146">Le test a utilisé une charge d’étape allant jusqu'à 1 000 utilisateurs simultanés.</span><span class="sxs-lookup"><span data-stu-id="c7911-146">The test used a step load of up to 1000 concurrent users.</span></span>

![Résultats des performances du test de charge pour le contrôleur basé sur SQL][MonolithicScenarioLoadTest]

<span data-ttu-id="c7911-148">À mesure que la charge augmente jusqu’à 700 utilisateurs, le débit augmente également.</span><span class="sxs-lookup"><span data-stu-id="c7911-148">As the load increases to 700 users, so does the throughput.</span></span> <span data-ttu-id="c7911-149">À ce stade, sans considérer les niveaux de débit, le système semble être en cours d’exécution à sa capacité maximale.</span><span class="sxs-lookup"><span data-stu-id="c7911-149">But at that point, throughput levels off, and the system appears to be running at its maximum capacity.</span></span> <span data-ttu-id="c7911-150">La réponse moyenne augmente progressivement avec une charge d’utilisateur, indiquant que le système ne peut pas suivre la demande.</span><span class="sxs-lookup"><span data-stu-id="c7911-150">The average response gradually increases with user load, showing that the system can't keep up with demand.</span></span>

### <a name="identify-periods-of-poor-performance"></a><span data-ttu-id="c7911-151">Identifier les périodes présentant des problèmes de performance</span><span class="sxs-lookup"><span data-stu-id="c7911-151">Identify periods of poor performance</span></span>

<span data-ttu-id="c7911-152">Si vous surveillez le système de production, vous pouvez remarquer des modèles.</span><span class="sxs-lookup"><span data-stu-id="c7911-152">If you are monitoring the production system, you might notice patterns.</span></span> <span data-ttu-id="c7911-153">Par exemple, les temps de réponse s’allonge de façon significative au même moment chaque jour.</span><span class="sxs-lookup"><span data-stu-id="c7911-153">For example, response times might drop off significantly at the same time each day.</span></span> <span data-ttu-id="c7911-154">Cela peut être dû à une charge de travail normale ou à une tâche planifiée, ou simplement parce que le système accueille davantage d’utilisateurs à certains moments.</span><span class="sxs-lookup"><span data-stu-id="c7911-154">This could be caused by a regular workload or scheduled batch job, or just because the system has more users at certain times.</span></span> <span data-ttu-id="c7911-155">Vous devriez vous concentrer sur les données de télémétrie de ces événements.</span><span class="sxs-lookup"><span data-stu-id="c7911-155">You should focus on the telemetry data for these events.</span></span>

<span data-ttu-id="c7911-156">Recherchez des corrélations entre les temps de réponse allongés et l’augmentation de l’activité de la base de données ou des E/S vers des ressources partagées.</span><span class="sxs-lookup"><span data-stu-id="c7911-156">Look for correlations between increased response times and increased database activity or I/O to shared resources.</span></span> <span data-ttu-id="c7911-157">S’il existe des corrélations, cela signifie que la base de données est susceptible d’être un goulot d’étranglement.</span><span class="sxs-lookup"><span data-stu-id="c7911-157">If there are correlations, it means the database might be a bottleneck.</span></span>

### <a name="identify-which-data-stores-are-accessed-during-those-periods"></a><span data-ttu-id="c7911-158">Identifier les magasins de données consultés durant ces périodes</span><span class="sxs-lookup"><span data-stu-id="c7911-158">Identify which data stores are accessed during those periods</span></span>

<span data-ttu-id="c7911-159">Le graphique suivant illustre l’utilisation des unités de débit de base de données (UDBD) pendant le test de charge.</span><span class="sxs-lookup"><span data-stu-id="c7911-159">The next graph shows the utilization of database throughput units (DTU) during the load test.</span></span> <span data-ttu-id="c7911-160">(Une UDBD est une mesure de la capacité disponible, et une combinaison de l’utilisation du processeur, de l’allocation de mémoire et du taux d’E/S.) L’utilisation des UDBD atteint rapidement 100 %.</span><span class="sxs-lookup"><span data-stu-id="c7911-160">(A DTU is a measure of available capacity, and is a combination of CPU utilization, memory allocation, I/O rate.) Utilization of DTUs quickly reached 100%.</span></span> <span data-ttu-id="c7911-161">C’est à peu près le point où le débit a atteint un pic dans le graphique précédent.</span><span class="sxs-lookup"><span data-stu-id="c7911-161">This is roughly the point where throughput peaked in the previous graph.</span></span> <span data-ttu-id="c7911-162">L’utilisation de la base de données est restée très élevée jusqu'à la fin du test.</span><span class="sxs-lookup"><span data-stu-id="c7911-162">Database utilization remained very high until the test finished.</span></span> <span data-ttu-id="c7911-163">Il y a une légère baisse vers la fin, peut être due à la limitation, la concurrence pour les connexions de base de données ou d’autres facteurs.</span><span class="sxs-lookup"><span data-stu-id="c7911-163">There is a slight drop toward the end, which could be caused by throttling, competition for database connections, or other factors.</span></span>

![La surveillance de la base de données dans le portail Azure classique montrant l’utilisation des ressources de la base de données][MonolithicDatabaseUtilization]

### <a name="examine-the-telemetry-for-the-data-stores"></a><span data-ttu-id="c7911-165">Examiner les données de télémétrie pour les magasins de données</span><span class="sxs-lookup"><span data-stu-id="c7911-165">Examine the telemetry for the data stores</span></span>

<span data-ttu-id="c7911-166">Instrumentez les magasins de données pour capturer les détails de bas niveau de l’activité.</span><span class="sxs-lookup"><span data-stu-id="c7911-166">Instrument the data stores to capture the low-level details of the activity.</span></span> <span data-ttu-id="c7911-167">Dans l’exemple d’application, les statistiques d’accès aux données ont montré un grand nombre d’opérations d’insertion effectuées sur les tables `PurchaseOrderHeader` et `MonoLog`.</span><span class="sxs-lookup"><span data-stu-id="c7911-167">In the sample application, the data access statistics showed a high volume of insert operations performed against both the `PurchaseOrderHeader` table and the `MonoLog` table.</span></span> 

![Les statistiques d’accès aux données pour l’exemple d’application][MonolithicDataAccessStats]

### <a name="identify-resource-contention"></a><span data-ttu-id="c7911-169">Identifier les contentions de ressources</span><span class="sxs-lookup"><span data-stu-id="c7911-169">Identify resource contention</span></span>

<span data-ttu-id="c7911-170">À ce stade, vous pouvez examiner le code source, en mettant l’accent sur les points où les ressources objets d’une contention sont accessibles par l’application.</span><span class="sxs-lookup"><span data-stu-id="c7911-170">At this point, you can review the source code, focusing on the points where contended resources are accessed by the application.</span></span> <span data-ttu-id="c7911-171">Recherchez des situations telles que :</span><span class="sxs-lookup"><span data-stu-id="c7911-171">Look for situations such as:</span></span>

- <span data-ttu-id="c7911-172">Des données logiquement séparées et écrites dans le même magasin.</span><span class="sxs-lookup"><span data-stu-id="c7911-172">Data that is logically separate being written to the same store.</span></span> <span data-ttu-id="c7911-173">Des données telles que des journaux, rapports et messages en attente ne devraient pas être conservées dans la même base de données en tant qu’informations d’entreprise.</span><span class="sxs-lookup"><span data-stu-id="c7911-173">Data such as logs, reports, and queued messages should not be held in the same database as business information.</span></span>
- <span data-ttu-id="c7911-174">Une incompatibilité entre le choix du magasin de données et le type de données, tels que des objets BLOB volumineux ou des documents XML dans une base de données relationnelle.</span><span class="sxs-lookup"><span data-stu-id="c7911-174">A mismatch between the choice of data store and the type of data, such as large blobs or XML documents in a relational database.</span></span>
- <span data-ttu-id="c7911-175">Des données avec des modèles d’utilisation très différents qui partagent le même magasin, telles que des données d’écriture élevée-lecture basse stockées avec des données d’écriture basse-lecture élevée.</span><span class="sxs-lookup"><span data-stu-id="c7911-175">Data with significantly different usage patterns that share the same store, such as high-write/low-read data being stored with low-write/high-read data.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="c7911-176">Implémenter la solution et vérifier le résultat</span><span class="sxs-lookup"><span data-stu-id="c7911-176">Implement the solution and verify the result</span></span>

<span data-ttu-id="c7911-177">L’application a été modifiée pour écrire des journaux dans un magasin de données distinct.</span><span class="sxs-lookup"><span data-stu-id="c7911-177">The application was changed to write logs to a separate data store.</span></span> <span data-ttu-id="c7911-178">Voici les résultats des tests de charge :</span><span class="sxs-lookup"><span data-stu-id="c7911-178">Here are the load test results:</span></span>

![Résultats des performances du test de charge à l’aide du contrôleur polyglotte][PolyglotScenarioLoadTest]

<span data-ttu-id="c7911-180">Le modèle de débit est similaire au graphique antérieur, mais le point du pic de performances est plus élevé d’environ 500 demandes par seconde.</span><span class="sxs-lookup"><span data-stu-id="c7911-180">The pattern of throughput is similar to the earlier graph, but the point at which performance peaks is approximately 500 requests per second higher.</span></span> <span data-ttu-id="c7911-181">Le temps de réponse moyen est légèrement plus court.</span><span class="sxs-lookup"><span data-stu-id="c7911-181">The average response time is marginally lower.</span></span> <span data-ttu-id="c7911-182">Toutefois, ces statistiques ne disent pas tout.</span><span class="sxs-lookup"><span data-stu-id="c7911-182">However, these statistics don't tell the full story.</span></span> <span data-ttu-id="c7911-183">Les données de télémétrie pour la base de données d’entreprise montrent que la UDBD a un pic d’utilisation à environ 75 %, au lieu de 100 %.</span><span class="sxs-lookup"><span data-stu-id="c7911-183">Telemetry for the business database shows that DTU utilization peaks at around 75%, rather than 100%.</span></span>

![La surveillance de la base de données dans le portail Azure classique montrant l’utilisation des ressources de la base de données dans le scénario polyglotte][PolyglotDatabaseUtilization]

<span data-ttu-id="c7911-185">De même, l’utilisation maximale de la UDBD de la base de données de journaux n'atteint environ que 70 %.</span><span class="sxs-lookup"><span data-stu-id="c7911-185">Similarly, the maximum DTU utilization of the log database only reaches about 70%.</span></span> <span data-ttu-id="c7911-186">Les bases de données ne sont plus le facteur de limitation pour les performances du système.</span><span class="sxs-lookup"><span data-stu-id="c7911-186">The databases are no longer the limiting factor in the performance of the system.</span></span>

![La surveillance de la base de données dans le portail Azure classique montrant l’utilisation des ressources de la base de données dans le scénario polyglotte][LogDatabaseUtilization]


## <a name="related-resources"></a><span data-ttu-id="c7911-188">Ressources associées</span><span class="sxs-lookup"><span data-stu-id="c7911-188">Related resources</span></span>

- <span data-ttu-id="c7911-189">[Choisir le magasin de données correct][data-store-overview]</span><span class="sxs-lookup"><span data-stu-id="c7911-189">[Choose the right data store][data-store-overview]</span></span>
- <span data-ttu-id="c7911-190">[Critères de sélection d’un magasin de données][data-store-comparison]</span><span class="sxs-lookup"><span data-stu-id="c7911-190">[Criteria for choosing a data store][data-store-comparison]</span></span>
- <span data-ttu-id="c7911-191">[Accès aux données pour les solutions hautement évolutives : Utilisation de la persistance SQL, NoSQL et polyglotte][Data-Access-Guide]</span><span class="sxs-lookup"><span data-stu-id="c7911-191">[Data Access for Highly-Scalable Solutions: Using SQL, NoSQL, and Polyglot Persistence][Data-Access-Guide]</span></span>
- <span data-ttu-id="c7911-192">[Partitionnement des données][DataPartitioningGuidance]</span><span class="sxs-lookup"><span data-stu-id="c7911-192">[Data partitioning][DataPartitioningGuidance]</span></span>

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/MonolithicPersistence
[CosmosDB]: http://azure.microsoft.com/services/cosmos-db/
[Azure-cache]: /azure/redis-cache/
[Data-Access-Guide]: https://msdn.microsoft.com/library/dn271399.aspx
[DataPartitioningGuidance]: ../../best-practices/data-partitioning.md
[data-store-overview]: ../../guide/technology-choices/data-store-overview.md
[data-store-comparison]: ../../guide/technology-choices/data-store-comparison.md

[MonolithicScenarioLoadTest]: _images/MonolithicScenarioLoadTest.jpg
[MonolithicDatabaseUtilization]: _images/MonolithicDatabaseUtilization.jpg
[MonolithicDataAccessStats]: _images/MonolithicDataAccessStats.jpg
[PolyglotScenarioLoadTest]: _images/PolyglotScenarioLoadTest.jpg
[PolyglotDatabaseUtilization]: _images/PolyglotDatabaseUtilization.jpg
[LogDatabaseUtilization]: _images/LogDatabaseUtilization.jpg
