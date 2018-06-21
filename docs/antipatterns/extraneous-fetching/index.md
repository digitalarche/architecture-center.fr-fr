---
title: Antimodèle de récupération superflue
description: Récupérer plus de données que nécessaire pour une opération d’entreprise peut provoquer une surcharge d’E/S inutile et réduire les temps de réponse.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 7a72bfd3e4b2e206f3266a046fac2083224ecb4f
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/06/2018
ms.locfileid: "30846601"
---
# <a name="extraneous-fetching-antipattern"></a><span data-ttu-id="f8ef4-103">Antimodèle de récupération superflue</span><span class="sxs-lookup"><span data-stu-id="f8ef4-103">Extraneous Fetching antipattern</span></span>

<span data-ttu-id="f8ef4-104">Récupérer plus de données que nécessaire pour une opération d’entreprise peut provoquer une surcharge d’E/S inutile et réduire les temps de réponse.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-104">Retrieving more data than needed for a business operation can result in unnecessary I/O overhead and reduce responsiveness.</span></span> 

## <a name="problem-description"></a><span data-ttu-id="f8ef4-105">Description du problème</span><span class="sxs-lookup"><span data-stu-id="f8ef4-105">Problem description</span></span>

<span data-ttu-id="f8ef4-106">Cet antimodèle peut se produire si l’application essaie de minimiser les demandes d’E/S en extrayant toutes les données dont elle *peut* avoir besoin.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-106">This antipattern can occur if the application tries to minimize I/O requests by retrieving all of the data that it *might* need.</span></span> <span data-ttu-id="f8ef4-107">Ceci est souvent la conséquence de la surcompensation pour l’antimodèle d’[E/S bavardes][chatty-io].</span><span class="sxs-lookup"><span data-stu-id="f8ef4-107">This is often a result of overcompensating for the [Chatty I/O][chatty-io] antipattern.</span></span> <span data-ttu-id="f8ef4-108">Par exemple, une application peut extraire les détails de chaque produit dans une base de données.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-108">For example, an application might fetch the details for every product in a database.</span></span> <span data-ttu-id="f8ef4-109">Mais l’utilisateur peut n’avoir besoin que d’un sous-ensemble de ces informations (certaines peuvent ne pas être pertinentes pour les clients) et n’a probablement pas besoin de voir *la totalité* des produits en même temps.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-109">But the user may need just a subset of the details (some may not be relevant to customers), and probably doesn't need to see *all* of the products at once.</span></span> <span data-ttu-id="f8ef4-110">Même si l’utilisateur parcourt l’intégralité du catalogue, il serait logique de paginer les résultats &mdash; en montrant 20 résultats à la fois, par exemple.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-110">Even if the user is browsing the entire catalog, it would make sense to paginate the results &mdash; showing 20 at a time, for example.</span></span>

<span data-ttu-id="f8ef4-111">Ce problème trouve aussi sa source dans des pratiques de programmation ou de conception médiocres.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-111">Another source of this problem is following poor programming or design practices.</span></span> <span data-ttu-id="f8ef4-112">Par exemple, le code suivant utilise Entity Framework pour extraire les informations détaillées de chaque produit.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-112">For example, the following code uses Entity Framework to fetch the complete details for every product.</span></span> <span data-ttu-id="f8ef4-113">Il filtre ensuite les résultats pour retourner uniquement un sous-ensemble des champs, en ignorant le reste.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-113">Then it filters the results to return only a subset of the fields, discarding the rest.</span></span> <span data-ttu-id="f8ef4-114">Vous trouverez l’exemple complet [ici][sample-app].</span><span class="sxs-lookup"><span data-stu-id="f8ef4-114">You can find the complete sample [here][sample-app].</span></span>

```csharp
public async Task<IHttpActionResult> GetAllFieldsAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Execute the query. This happens at the database.
        var products = await context.Products.ToListAsync();

        // Project fields from the query results. This happens in application memory.
        var result = products.Select(p => new ProductInfo { Id = p.ProductId, Name = p.Name });
        return Ok(result);
    }
}
```

<span data-ttu-id="f8ef4-115">Dans l’exemple suivant, l’application récupère les données pour effectuer une agrégation qui aurait pu être effectuée par la base de données.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-115">In the next example, the application retrieves data to perform an aggregation that could be done by the database instead.</span></span> <span data-ttu-id="f8ef4-116">L’application calcule le total des ventes en obtenant tous les enregistrements pour toutes les commandes, puis calcule la somme sur ces enregistrements.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-116">The application calculates total sales by getting every record for all orders sold, and then computing the sum over those records.</span></span> <span data-ttu-id="f8ef4-117">Vous trouverez l’exemple complet [ici][sample-app].</span><span class="sxs-lookup"><span data-stu-id="f8ef4-117">You can find the complete sample [here][sample-app].</span></span>

```csharp
public async Task<IHttpActionResult> AggregateOnClientAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Fetch all order totals from the database.
        var orderAmounts = await context.SalesOrderHeaders.Select(soh => soh.TotalDue).ToListAsync();

        // Sum the order totals in memory.
        var total = orderAmounts.Sum();
        return Ok(total);
    }
}
```

<span data-ttu-id="f8ef4-118">L’exemple suivant montre un problème subtil, provoqué par la façon dont Entity Framework utilise LINQ to Entities.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-118">The next example shows a subtle problem caused by the way Entity Framework uses LINQ to Entities.</span></span> 

```csharp
var query = from p in context.Products.AsEnumerable()
            where p.SellStartDate < DateTime.Now.AddDays(-7) // AddDays cannot be mapped by LINQ to Entities
            select ...;

List<Product> products = query.ToList();
```

<span data-ttu-id="f8ef4-119">L’application essaie de trouver les produits avec une `SellStartDate` de plus d’une semaine.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-119">The application is trying to find products with a `SellStartDate` more than a week old.</span></span> <span data-ttu-id="f8ef4-120">Dans la plupart des cas, LINQ to Entities traduit une clause `where` pour une instruction SQL qui est exécutée par la base de données.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-120">In most cases, LINQ to Entities would translate a `where` clause to a SQL statement that is executed by the database.</span></span> <span data-ttu-id="f8ef4-121">Dans ce cas, toutefois, LINQ to Entities ne peut pas mapper la méthode `AddDays` à SQL.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-121">In this case, however, LINQ to Entities cannot map the `AddDays` method to SQL.</span></span> <span data-ttu-id="f8ef4-122">Au lieu de cela, chaque ligne à partir de la table `Product` est retournée et les résultats sont filtrés dans la mémoire.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-122">Instead, every row from the `Product` table is returned, and the results are filtered in memory.</span></span> 

<span data-ttu-id="f8ef4-123">L’appel à `AsEnumerable` est une indication qu’il existe un problème.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-123">The call to `AsEnumerable` is a hint that there is a problem.</span></span> <span data-ttu-id="f8ef4-124">Cette méthode convertit les résultats en une interface `IEnumerable`.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-124">This method converts the results to an `IEnumerable` interface.</span></span> <span data-ttu-id="f8ef4-125">Bien que `IEnumerable` prend en charge le filtrage, le filtrage est effectué côté *client* et non côté base de données.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-125">Although `IEnumerable` supports filtering, the filtering is done on the *client* side, not the database.</span></span> <span data-ttu-id="f8ef4-126">Par défaut, LINQ to Entities utilise `IQueryable`, qui passe la responsabilité de filtrage à la source de données.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-126">By default, LINQ to Entities uses `IQueryable`, which passes the responsibility for filtering to the data source.</span></span> 

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="f8ef4-127">Comment corriger le problème</span><span class="sxs-lookup"><span data-stu-id="f8ef4-127">How to fix the problem</span></span>

<span data-ttu-id="f8ef4-128">Évitez l’extraction de grands volumes de données qui peuvent rapidement devenir obsolètes ou peuvent être ignorées et n’extrayez que les données nécessaires pour l’opération en cours.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-128">Avoid fetching large volumes of data that may quickly become outdated or might be discarded, and only fetch the data needed for the operation being performed.</span></span> 

<span data-ttu-id="f8ef4-129">Au lieu d’obtenir toutes les colonnes d’une table, puis de les filtrer, sélectionnez les colonnes dont vous avez besoin à partir de la base de données.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-129">Instead of getting every column from a table and then filtering them, select the columns that you need from the database.</span></span>

```csharp
public async Task<IHttpActionResult> GetRequiredFieldsAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Project fields as part of the query itself
        var result = await context.Products
            .Select(p => new ProductInfo {Id = p.ProductId, Name = p.Name})
            .ToListAsync();
        return Ok(result);
    }
}
```

<span data-ttu-id="f8ef4-130">De même, effectuez une agrégation dans la base de données et non dans la mémoire de l’application.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-130">Similarly, perform aggregation in the database and not in application memory.</span></span>

```csharp
public async Task<IHttpActionResult> AggregateOnDatabaseAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Sum the order totals as part of the database query.
        var total = await context.SalesOrderHeaders.SumAsync(soh => soh.TotalDue);
        return Ok(total);
    }
}
```

<span data-ttu-id="f8ef4-131">Lorsque vous utilisez Entity Framework, vérifiez que les requêtes LINQ sont résolues à l’aide de l’interface `IQueryable` et non `IEnumerable`.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-131">When using Entity Framework, ensure that LINQ queries are resolved using the `IQueryable`interface and not `IEnumerable`.</span></span> <span data-ttu-id="f8ef4-132">Vous devrez peut-être ajuster la requête pour utiliser uniquement les fonctions qui peuvent être mappées à la source de données.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-132">You may need to adjust the query to use only functions that can be mapped to the data source.</span></span> <span data-ttu-id="f8ef4-133">L’exemple précédent peut être refactorisé pour supprimer la méthode `AddDays` de la requête, ce qui permet d’effectuer le filtrage depuis la base de données.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-133">The earlier example can be refactored to remove the `AddDays` method from the query, allowing filtering to be done by the database.</span></span>

```csharp
DateTime dateSince = DateTime.Now.AddDays(-7); // AddDays has been factored out. 
var query = from p in context.Products
            where p.SellStartDate < dateSince // This criterion can be passed to the database by LINQ to Entities
            select ...;

List<Product> products = query.ToList();
```

## <a name="considerations"></a><span data-ttu-id="f8ef4-134">Considérations</span><span class="sxs-lookup"><span data-stu-id="f8ef4-134">Considerations</span></span>

- <span data-ttu-id="f8ef4-135">Dans certains cas, vous pouvez améliorer les performances en partitionnant horizontalement les données.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-135">In some cases, you can improve performance by partitioning data horizontally.</span></span> <span data-ttu-id="f8ef4-136">Si différentes opérations accèdent aux différents attributs des données, le partitionnement horizontal peut réduire la contention.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-136">If different operations access different attributes of the data, horizontal partitioning may reduce contention.</span></span> <span data-ttu-id="f8ef4-137">Souvent, la plupart des opérations sont exécutées par rapport à un petit sous-ensemble des données, ainsi répartir cette charge peut améliorer les performances.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-137">Often, most operations are run against a small subset of the data, so spreading this load may improve performance.</span></span> <span data-ttu-id="f8ef4-138">Voir [Partitionnement des données][data-partitioning].</span><span class="sxs-lookup"><span data-stu-id="f8ef4-138">See [Data partitioning][data-partitioning].</span></span>

- <span data-ttu-id="f8ef4-139">Pour les opérations qui doivent prendre en charge les requêtes non liées, implémentez la pagination et n’extrayez qu’un nombre limité d’entités à la fois.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-139">For operations that have to support unbounded queries, implement pagination and only fetch a limited number of entities at a time.</span></span> <span data-ttu-id="f8ef4-140">Par exemple, si un client parcourt un catalogue de produits, vous pouvez afficher une page de résultats à la fois.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-140">For example, if a customer is browsing a product catalog, you can show one page of results at a time.</span></span>

- <span data-ttu-id="f8ef4-141">Lorsque cela est possible, tirez parti des fonctionnalités intégrées dans le magasin de données.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-141">When possible, take advantage of features built into the data store.</span></span> <span data-ttu-id="f8ef4-142">Par exemple, les bases de données SQL fournissent généralement des fonctions d’agrégation.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-142">For example, SQL databases typically provide aggregate functions.</span></span> 

- <span data-ttu-id="f8ef4-143">Si vous utilisez un magasin de données qui ne prend pas en charge une fonction particulière, telle que l’agrégation, vous pouvez stocker le résultat calculé ailleurs et mettre à jour les valeurs lorsque les enregistrements sont ajoutés ou mis à jour, afin que l’application n’ait pas à recalculer la valeur chaque fois que cela est nécessaire.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-143">If you're using a data store that doesn't support a particular function, such as aggregration, you could store the calculated result elsewhere, updating the value as records are added or updated, so the application doesn't have to recalculate the value each time it's needed.</span></span>

- <span data-ttu-id="f8ef4-144">Si vous voyez que les demandes extraient un grand nombre de champs, examinez le code source pour déterminer si tous ces champs sont réellement nécessaires.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-144">If you see that requests are retrieving a large number of fields, examine the source code to determine whether all of these fields are actually necessary.</span></span> <span data-ttu-id="f8ef4-145">Parfois, ces requêtes sont le résultat de requêtes `SELECT *` de mauvaise conception.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-145">Sometimes these requests are the result of poorly designed `SELECT *` query.</span></span> 

- <span data-ttu-id="f8ef4-146">De même, les requêtes qui récupèrent un grand nombre d’entités peuvent démontrer que l’application ne filtre pas correctement les données.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-146">Similarly, requests that retrieve a large number of entities may be sign that the application is not filtering data correctly.</span></span> <span data-ttu-id="f8ef4-147">Vérifiez que toutes ces entités sont réellement nécessaires.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-147">Verify that all of these entities are actually needed.</span></span> <span data-ttu-id="f8ef4-148">Utilisez le filtrage côté base de données dans la mesure du possible, par exemple, à l’aide de clauses `WHERE` dans SQL.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-148">Use database-side filtering if possible, for example, by using `WHERE` clauses in SQL.</span></span> 

- <span data-ttu-id="f8ef4-149">Le déchargement du traitement sur la base de données n’est pas toujours la meilleure option.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-149">Offloading processing to the database is not always the best option.</span></span> <span data-ttu-id="f8ef4-150">N’utilisez cette stratégie que lorsque la base de données est conçue ou optimisée pour cela.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-150">Only use this strategy when the database is designed or optimized to do so.</span></span> <span data-ttu-id="f8ef4-151">La plupart des systèmes de base de données sont optimisés pour certaines fonctions, mais ne sont pas conçus pour servir de moteurs d’applications à usage général.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-151">Most database systems are highly optimized for certain functions, but are not designed to act as general-purpose application engines.</span></span> <span data-ttu-id="f8ef4-152">Pour plus d’informations, consultez l’article [Antimodèle de base de données occupé][BusyDatabase].</span><span class="sxs-lookup"><span data-stu-id="f8ef4-152">For more information, see the [Busy Database antipattern][BusyDatabase].</span></span>


## <a name="how-to-detect-the-problem"></a><span data-ttu-id="f8ef4-153">Comment détecter le problème</span><span class="sxs-lookup"><span data-stu-id="f8ef4-153">How to detect the problem</span></span>

<span data-ttu-id="f8ef4-154">Une récupération superflue se traduit par une latence élevée et un débit faible.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-154">Symptoms of extraneous fetching include high latency and low throughput.</span></span> <span data-ttu-id="f8ef4-155">Si les données sont récupérées à partir d’un magasin de données, une contention accrue est également probable.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-155">If the data is retrieved from a data store, increased contention is also probable.</span></span> <span data-ttu-id="f8ef4-156">Les utilisateurs finals sont susceptibles de signaler des temps de réponse plus longs que d’habitude ou des échecs causés par l’expiration du délai d’attente des services. Ces échecs peuvent renvoyer des erreurs HTTP 500 (serveur interne) ou HTTP 503 (service indisponible).</span><span class="sxs-lookup"><span data-stu-id="f8ef4-156">End users are likely to report extended response times or failures caused by services timing out. These failures could return HTTP 500 (Internal Server) errors or HTTP 503 (Service Unavailable) errors.</span></span> <span data-ttu-id="f8ef4-157">Examinez les journaux d’événements du serveur web, qui contiennent probablement des informations plus détaillées sur les causes et les circonstances de ces erreurs.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-157">Examine the event logs for the web server, which are likely to contain more detailed information about the causes and circumstances of the errors.</span></span>

<span data-ttu-id="f8ef4-158">Les symptômes de cet antimodèle ainsi qu’une partie de la télémétrie obtenue peuvent être très similaires à ceux de l’[antimodèle de persistance monolithique][MonolithicPersistence].</span><span class="sxs-lookup"><span data-stu-id="f8ef4-158">The symptoms of this antipattern and some of the telemetry obtained might be very similar to those of the [Monolithic Persistence antipattern][MonolithicPersistence].</span></span> 

<span data-ttu-id="f8ef4-159">Vous pouvez procédez de la manière suivante pour identifier la cause :</span><span class="sxs-lookup"><span data-stu-id="f8ef4-159">You can perform the following steps to help identify the cause:</span></span>

1. <span data-ttu-id="f8ef4-160">Identifiez les charges de travail ou les transactions lentes en effectuant des tests de charge, des analyses de processus ou d’autres méthodes de capture de données d’instrumentation.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-160">Identify slow workloads or transactions by performing load-testing, process monitoring, or other methods of capturing instrumentation data.</span></span>
2. <span data-ttu-id="f8ef4-161">Observez les modèles de comportement exposés par le système.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-161">Observe any behavioral patterns exhibited by the system.</span></span> <span data-ttu-id="f8ef4-162">Existe-t-il des limites particulières en termes de transactions par seconde ou de volume d’utilisateurs ?</span><span class="sxs-lookup"><span data-stu-id="f8ef4-162">Are there particular limits in terms of transactions per second or volume of users?</span></span>
3. <span data-ttu-id="f8ef4-163">Mettez en corrélation les instances des charges de travail lentes avec les modèles de comportement.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-163">Correlate the instances of slow workloads with behavioral patterns.</span></span>
4. <span data-ttu-id="f8ef4-164">Identifiez les banques de données utilisées.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-164">Identify the data stores being used.</span></span> <span data-ttu-id="f8ef4-165">Pour chaque source de données, exécutez les données de télémétrie de niveau inférieur pour observer le comportement des opérations.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-165">For each data source, run lower level telemetry to observe the behavior of operations.</span></span>
6. <span data-ttu-id="f8ef4-166">Identifiez les requêtes à exécution lente qui font référence à ces sources de données.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-166">Identify any slow-running queries that reference these data sources.</span></span>
7. <span data-ttu-id="f8ef4-167">Effectuez une analyse de ressource spécifique des requêtes à exécution lente et déterminez comment les données sont utilisées et consommées.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-167">Perform a resource-specific analysis of the slow-running queries and ascertain how the data is used and consumed.</span></span>

<span data-ttu-id="f8ef4-168">Recherchez la présence des symptômes suivants :</span><span class="sxs-lookup"><span data-stu-id="f8ef4-168">Look for any of these symptoms:</span></span>

- <span data-ttu-id="f8ef4-169">Demandes d’E/S fréquentes et de grande taille effectuées à la même ressource ou base de données.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-169">Frequent, large I/O requests made to the same resource or data store.</span></span>
- <span data-ttu-id="f8ef4-170">Contention dans un magasin de données ou dans une ressource partagée.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-170">Contention in a shared resource or data store.</span></span>
- <span data-ttu-id="f8ef4-171">Opération qui reçoit fréquemment des volumes importants de données sur le réseau.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-171">An operation that frequently receives large volumes of data over the network.</span></span>
- <span data-ttu-id="f8ef4-172">Applications et services attendant longtemps que les E/S se terminent.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-172">Applications and services spending significant time waiting for I/O to complete.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="f8ef4-173">Exemple de diagnostic</span><span class="sxs-lookup"><span data-stu-id="f8ef4-173">Example diagnosis</span></span>    

<span data-ttu-id="f8ef4-174">Les sections suivantes appliquent ces étapes pour les exemples précédents.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-174">The following sections apply these steps to the previous examples.</span></span>

### <a name="identify-slow-workloads"></a><span data-ttu-id="f8ef4-175">Identifier les charges de travail lentes</span><span class="sxs-lookup"><span data-stu-id="f8ef4-175">Identify slow workloads</span></span>

<span data-ttu-id="f8ef4-176">Ce graphique montre les résultats de performances d’un test de charge simulant jusqu’à 400 utilisateurs simultanés exécutant la méthode `GetAllFieldsAsync` illustrée précédemment.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-176">This graph shows performance results from a load test that simulated up to 400 concurrent users running the `GetAllFieldsAsync` method shown earlier.</span></span> <span data-ttu-id="f8ef4-177">Le débit diminue lentement à mesure que la charge augmente.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-177">Throughput diminishes slowly as the load increases.</span></span> <span data-ttu-id="f8ef4-178">Le temps de réponse moyen augmente à mesure que la charge de travail augmente.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-178">Average response time goes up as the workload increases.</span></span> 

![Résultats de test de charge pour la méthode GetAllFieldsAsync][Load-Test-Results-Client-Side1]

<span data-ttu-id="f8ef4-180">Un test de charge pour l’opération `AggregateOnClientAsync` montre un modèle semblable.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-180">A load test for the `AggregateOnClientAsync` operation shows a similar pattern.</span></span> <span data-ttu-id="f8ef4-181">Le volume de requêtes est relativement stable.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-181">The volume of requests is reasonably stable.</span></span> <span data-ttu-id="f8ef4-182">Le temps de réponse moyen augmente avec la charge de travail, bien que plus lentement que le graphique précédent.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-182">The average response time increases with the workload, although more slowly than the previous graph.</span></span>

![Résultats de test de charge pour la méthode AggregateOnClientAsync][Load-Test-Results-Client-Side2]

### <a name="correlate-slow-workloads-with-behavioral-patterns"></a><span data-ttu-id="f8ef4-184">Mettre en corrélation les charges de travail lentes avec les modèles de comportement</span><span class="sxs-lookup"><span data-stu-id="f8ef4-184">Correlate slow workloads with behavioral patterns</span></span>

<span data-ttu-id="f8ef4-185">Toute corrélation entre périodes régulières d’utilisation intensive et ralentissement des performances peut indiquer les zones posant problème.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-185">Any correlation between regular periods of high usage and slowing performance can indicate areas of concern.</span></span> <span data-ttu-id="f8ef4-186">Examinez le profil de performance de fonctionnalités qui semble s’exécuter lentement, pour déterminer s’il correspond au test de charge effectué précédemment.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-186">Closely examine the performance profile of functionality that is suspected to be slow running, to determine whether it matches the load testing performed earlier.</span></span>

<span data-ttu-id="f8ef4-187">Effectuez un test de charge sur les mêmes fonctionnalités à l’aide des charges utilisateur basées sur les étapes, pour trouver le point où les performances diminuent de manière significative ou échouent complètement.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-187">Load test the same functionality using step-based user loads, to find the point where performance drops significantly or fails completely.</span></span> <span data-ttu-id="f8ef4-188">Si ce point se situe dans les limites de votre utilisation réelle attendue, examinez la manière dont la fonctionnalité est implémentée.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-188">If that point falls within the bounds of your expected real-world usage, examine how the functionality is implemented.</span></span>

<span data-ttu-id="f8ef4-189">Une opération lente n’est pas nécessairement un problème, si elle n’est pas en cours lorsque le système est sous tension, si elle n’est pas urgente et qu’elle n’affecte pas négativement les performances d’autres opérations importantes.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-189">A slow operation is not necessarily a problem, if it is not being performed when the system is under stress, is not time critical, and does not negatively affect the performance of other important operations.</span></span> <span data-ttu-id="f8ef4-190">Par exemple, la génération de statistiques opérationnelles mensuelles peut être une opération longue, mais elle peut probablement s’effectuer en traitement par lots et s’exécuter en tant que travail de priorité basse.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-190">For example, generating monthly operational statistics might be a long-running operation, but it can probably be performed as a batch process and run as a low priority job.</span></span> <span data-ttu-id="f8ef4-191">En revanche, l’interrogation du catalogue de produits par les clients est une opération métier critique.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-191">On the other hand, customers querying the product catalog is a critical business operation.</span></span> <span data-ttu-id="f8ef4-192">Concentrez-vous sur la télémétrie générée par ces opérations critiques pour déterminer comment les performances varient au cours des périodes d’utilisation intensive.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-192">Focus on the telemetry generated by these critical operations to see how the performance varies during periods of high usage.</span></span>

### <a name="identify-data-sources-in-slow-workloads"></a><span data-ttu-id="f8ef4-193">Identifier les sources de données dans les charges de travail lentes</span><span class="sxs-lookup"><span data-stu-id="f8ef4-193">Identify data sources in slow workloads</span></span>

<span data-ttu-id="f8ef4-194">Si vous pensez qu’un service est médiocre en raison de la façon dont il récupère les données, examinez la façon dont l’application interagit avec les référentiels qu’elle utilise.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-194">If you suspect that a service is performing poorly because of the way it retrieves data, investigate how the application interacts with the repositories it uses.</span></span> <span data-ttu-id="f8ef4-195">Analysez le système réel pour voir quelles sources sont accessibles pendant les périodes de performance médiocre.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-195">Monitor the live system to see which sources are accessed during periods of poor performance.</span></span> 

<span data-ttu-id="f8ef4-196">Pour chaque source de données, instrumentez le système afin de capturer les éléments suivants :</span><span class="sxs-lookup"><span data-stu-id="f8ef4-196">For each data source, instrument the system to capture the following:</span></span>

- <span data-ttu-id="f8ef4-197">La fréquence à laquelle chaque magasin de données est accessible.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-197">The frequency that each data store is accessed.</span></span>
- <span data-ttu-id="f8ef4-198">Le volume de données entrant et sortant de la banque de données.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-198">The volume of data entering and exiting the data store.</span></span>
- <span data-ttu-id="f8ef4-199">Le minutage de ces opérations, notamment la latence des demandes.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-199">The timing of these operations, especially the latency of requests.</span></span>
- <span data-ttu-id="f8ef4-200">La nature et la fréquence des erreurs qui se produisent pendant l’accès à chaque magasin de données sous une charge normale.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-200">The nature and rate of any errors that occur while accessing each data store under typical load.</span></span>

<span data-ttu-id="f8ef4-201">Comparez ces informations avec le volume de données renvoyé par l’application au client.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-201">Compare this information against the volume of data being returned by the application to the client.</span></span> <span data-ttu-id="f8ef4-202">Suivez le rapport entre le volume des données retournées par la banque de données et le volume de données retournées au client.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-202">Track the ratio of the volume of data returned by the data store against the volume of data returned to the client.</span></span> <span data-ttu-id="f8ef4-203">S’il existe une disparité volumineuse, faites des recherches pour déterminer si l’application extrait des données dont elle n’a pas besoin.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-203">If there is any large disparity, investigate to determine whether the application is fetching data that it doesn't need.</span></span>

<span data-ttu-id="f8ef4-204">Vous pourrez peut-être capturer ces données par l’observation du système réel et le suivi du cycle de vie de chaque demande de l’utilisateur, ou vous pouvez modéliser une série de charges de travail synthétiques et les exécuter sur un système de test.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-204">You may be able to capture this data by observing the live system and tracing the lifecycle of each user request, or you can model a series of synthetic workloads and run them against a test system.</span></span>

<span data-ttu-id="f8ef4-205">Les graphiques suivants montrent les données de télémétrie capturées à l’aide de l’[APM New Relic][new-relic] pendant un test de charge de la méthode `GetAllFieldsAsync`.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-205">The following graphs show telemetry captured using [New Relic APM][new-relic] during a load test of the `GetAllFieldsAsync` method.</span></span> <span data-ttu-id="f8ef4-206">Notez la différence entre les volumes de données reçues à partir de la base de données et les réponses HTTP correspondantes.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-206">Note the difference between the volumes of data received from the database and the corresponding HTTP responses.</span></span>

![Données de télémétrie pour la méthode « GetAllFieldsAsync »][TelemetryAllFields]

<span data-ttu-id="f8ef4-208">Pour chaque demande, la base de données a retourné 80 503 octets, mais la réponse au client ne contenait que 19 855 octets, environ 25 % de la taille de la réponse de la base de données.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-208">For each request, the database returned 80,503 bytes, but the response to the client only contained 19,855 bytes, about 25% of the size of the database response.</span></span> <span data-ttu-id="f8ef4-209">La taille des données retournées au client peut varier selon le format.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-209">The size of the data returned to the client can vary depending on the format.</span></span> <span data-ttu-id="f8ef4-210">Pour ce test de charge, le client a demandé des données JSON.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-210">For this load test, the client requested JSON data.</span></span> <span data-ttu-id="f8ef4-211">Le test individuel à l’aide de XML (non affiché) a obtenu une taille de réponse de 35 655 octets, ou 44 % de la taille de la réponse de la base de données.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-211">Separate testing using XML (not shown) had a response size of 35,655 bytes, or 44% of the size of the database response.</span></span>

<span data-ttu-id="f8ef4-212">Le test de charge pour la méthode `AggregateOnClientAsync` affiche des résultats plus extrêmes.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-212">The load test for the `AggregateOnClientAsync` method shows more extreme results.</span></span> <span data-ttu-id="f8ef4-213">Dans ce cas, chaque test a exécuté une requête permettant d’extraire plus de 280 Ko de données de la base de données, mais la réponse JSON était seulement de 14 octets.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-213">In this case, each test performed a query that retrieved over 280Kb of data from the database, but the JSON response was a mere 14 bytes.</span></span> <span data-ttu-id="f8ef4-214">Les écarts importants constatés sont dus à la méthode qui calcule un résultat agrégé à partir d’un gros volume de données.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-214">The wide disparity is because the method calculates an aggregated result from a large volume of data.</span></span>

![Données de télémétrie pour la méthode « AggregateOnClientAsync »][TelemetryAggregateOnClient]

### <a name="identify-and-analyze-slow-queries"></a><span data-ttu-id="f8ef4-216">Identifier et analyser les requêtes lentes</span><span class="sxs-lookup"><span data-stu-id="f8ef4-216">Identify and analyze slow queries</span></span>

<span data-ttu-id="f8ef4-217">Recherchez les requêtes de base de données qui consomment le plus de ressources et prennent le plus de temps à s’exécuter.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-217">Look for database queries that consume the most resources and take the most time to execute.</span></span> <span data-ttu-id="f8ef4-218">Vous pouvez ajouter la fonctionnalité d’instrumentation pour rechercher les heures de début et de fin pour de nombreuses opérations de base de données.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-218">You can add instrumentation to find the start and completion times for many database operations.</span></span> <span data-ttu-id="f8ef4-219">De nombreux magasins de données fournissent également des informations détaillées sur la façon dont les requêtes sont effectuées et optimisées.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-219">Many data stores also provide in-depth information on how queries are performed and optimized.</span></span> <span data-ttu-id="f8ef4-220">Par exemple, le volet de performance des requêtes dans le portail de gestion Azure SQL Database permet de sélectionner une requête et d’afficher les informations de performances d’exécution détaillées.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-220">For example, the Query Performance pane in the Azure SQL Database management portal lets you select a query and view detailed runtime performance information.</span></span> <span data-ttu-id="f8ef4-221">Voici la requête générée par l’opération `GetAllFieldsAsync` :</span><span class="sxs-lookup"><span data-stu-id="f8ef4-221">Here is the query generated by the `GetAllFieldsAsync` operation:</span></span>

![Le volet détails de la requête dans le portail de gestion Windows Azure SQL Database][QueryDetails]


## <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="f8ef4-223">Implémenter la solution et vérifier le résultat</span><span class="sxs-lookup"><span data-stu-id="f8ef4-223">Implement the solution and verify the result</span></span>

<span data-ttu-id="f8ef4-224">Après avoir modifié la méthode `GetRequiredFieldsAsync` pour utiliser une instruction SELECT côté base de données, le test de charge a montré les résultats suivants.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-224">After changing the `GetRequiredFieldsAsync` method to use a SELECT statement on the database side, load testing showed the following results.</span></span>

![Résultats de test de charge pour la méthode GetRequiredFieldsAsync][Load-Test-Results-Database-Side1]

<span data-ttu-id="f8ef4-226">Ce test de charge a utilisé le même déploiement et la même charge de travail simulée de 400 utilisateurs simultanés que pour l’exemple précédant.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-226">This load test used the same deployment and the same simulated workload of 400 concurrent users as before.</span></span> <span data-ttu-id="f8ef4-227">Ce graphique indique une latence plus faible.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-227">The graph shows much lower latency.</span></span> <span data-ttu-id="f8ef4-228">Le temps de réponse augmente avec la charge d’environ 1,3 secondes, par rapport à 4 secondes dans le cas précédent.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-228">Response time rises with load to approximately 1.3 seconds, compared to 4 seconds in the previous case.</span></span> <span data-ttu-id="f8ef4-229">Le débit est également plus élevé avec 350 demandes par seconde par rapport à 100 plus tôt.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-229">The throughput is also higher at 350 requests per second compared to 100 earlier.</span></span> <span data-ttu-id="f8ef4-230">Le volume des données récupérées à partir de la base de données correspond mieux à la taille des messages de réponse HTTP.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-230">The volume of data retrieved from the database now closely matches the size of the HTTP response messages.</span></span>

![Données de télémétrie pour la méthode « GetRequiredFieldsAsync »][TelemetryRequiredFields]

<span data-ttu-id="f8ef4-232">Le test de charge effectué à l’aide de la méthode `AggregateOnDatabaseAsync` génère les résultats suivants :</span><span class="sxs-lookup"><span data-stu-id="f8ef4-232">Load testing using the `AggregateOnDatabaseAsync` method generates the following results:</span></span>

![Résultats de test de charge pour la méthode AggregateOnDatabaseAsync][Load-Test-Results-Database-Side2]

<span data-ttu-id="f8ef4-234">Le temps de réponse moyen est désormais minimal.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-234">The average response time is now minimal.</span></span> <span data-ttu-id="f8ef4-235">Il s’agit d’une amélioration importante des performances, principalement par la réduction des E/S volumineuses à partir de la base de données.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-235">This is an order of magnitude improvement in performance, caused primarily by the large reduction in I/O from the database.</span></span> 

<span data-ttu-id="f8ef4-236">Voici les données de télémétrie correspondantes pour la méthode `AggregateOnDatabaseAsync`.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-236">Here is the corresponding telemetry for the `AggregateOnDatabaseAsync` method.</span></span> <span data-ttu-id="f8ef4-237">La quantité de données récupérées à partir de la base de données a été considérablement réduite de plus de 280 Ko par transaction à 53 octets.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-237">The amount of data retrieved from the database was vastly reduced, from over 280Kb per transaction to 53 bytes.</span></span> <span data-ttu-id="f8ef4-238">Par conséquent, le nombre maximal de demandes soutenues par minute a été augmenté, d’environ 2 000 à plus de 25 000.</span><span class="sxs-lookup"><span data-stu-id="f8ef4-238">As a result, the maximum sustained number of requests per minute was raised from around 2,000 to over 25,000.</span></span>

![Données de télémétrie pour la méthode « AggregateOnDatabaseAsync »][TelemetryAggregateInDatabaseAsync]


## <a name="related-resources"></a><span data-ttu-id="f8ef4-240">Ressources associées</span><span class="sxs-lookup"><span data-stu-id="f8ef4-240">Related resources</span></span>

- <span data-ttu-id="f8ef4-241">[Antimodèle de base de données occupé][BusyDatabase]</span><span class="sxs-lookup"><span data-stu-id="f8ef4-241">[Busy Database antipattern][BusyDatabase]</span></span>
- <span data-ttu-id="f8ef4-242">[Antimodèle E/S bavardes][chatty-io]</span><span class="sxs-lookup"><span data-stu-id="f8ef4-242">[Chatty I/O antipattern][chatty-io]</span></span>
- <span data-ttu-id="f8ef4-243">[Bonnes pratiques de partitionnement des données][data-partitioning]</span><span class="sxs-lookup"><span data-stu-id="f8ef4-243">[Data partitioning best practices][data-partitioning]</span></span>


[BusyDatabase]: ../busy-database/index.md
[data-partitioning]: ../../best-practices/data-partitioning.md
[new-relic]: https://newrelic.com/application-monitoring

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/ExtraneousFetching

[chatty-io]: ../chatty-io/index.md
[MonolithicPersistence]: ../monolithic-persistence/index.md
[Load-Test-Results-Client-Side1]:_images/LoadTestResultsClientSide1.jpg
[Load-Test-Results-Client-Side2]:_images/LoadTestResultsClientSide2.jpg
[Load-Test-Results-Database-Side1]:_images/LoadTestResultsDatabaseSide1.jpg
[Load-Test-Results-Database-Side2]:_images/LoadTestResultsDatabaseSide2.jpg
[QueryPerformanceZoomed]: _images/QueryPerformanceZoomed.jpg
[QueryDetails]: _images/QueryDetails.jpg
[TelemetryAllFields]: _images/TelemetryAllFields.jpg
[TelemetryAggregateOnClient]: _images/TelemetryAggregateOnClient.jpg
[TelemetryRequiredFields]: _images/TelemetryRequiredFields.jpg
[TelemetryAggregateInDatabaseAsync]: _images/TelemetryAggregateInDatabase.jpg
