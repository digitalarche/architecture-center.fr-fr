---
title: Absence d’antimodèle de mise en cache
titleSuffix: Performance antipatterns for cloud apps
description: Extraire plusieurs fois les mêmes données peut réduire les performances et l’évolutivité.
author: dragon119
ms.date: 06/05/2017
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 5c3062c0a17de708ada83ba81dcb111e6b1f8440
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54488050"
---
# <a name="no-caching-antipattern"></a><span data-ttu-id="5da5e-103">Absence d’antimodèle de mise en cache</span><span class="sxs-lookup"><span data-stu-id="5da5e-103">No Caching antipattern</span></span>

<span data-ttu-id="5da5e-104">Dans une application cloud gérant de nombreuses requêtes simultanées, l’extraction répétée de mêmes données peut réduire l’évolutivité et les performances.</span><span class="sxs-lookup"><span data-stu-id="5da5e-104">In a cloud application that handles many concurrent requests, repeatedly fetching the same data can reduce performance and scalability.</span></span>

## <a name="problem-description"></a><span data-ttu-id="5da5e-105">Description du problème</span><span class="sxs-lookup"><span data-stu-id="5da5e-105">Problem description</span></span>

<span data-ttu-id="5da5e-106">Un certain nombre de comportements indésirables peuvent survenir lorsque les données ne sont pas mises en cache, notamment :</span><span class="sxs-lookup"><span data-stu-id="5da5e-106">When data is not cached, it can cause a number of undesirable behaviors, including:</span></span>

- <span data-ttu-id="5da5e-107">L’extraction répétée des mêmes informations à partir d’une ressource coûteuse pour l’accès en termes de latence ou de surcharge d’E/S.</span><span class="sxs-lookup"><span data-stu-id="5da5e-107">Repeatedly fetching the same information from a resource that is expensive to access, in terms of I/O overhead or latency.</span></span>
- <span data-ttu-id="5da5e-108">La construction répétée des mêmes objets ou structures de données pour plusieurs requêtes.</span><span class="sxs-lookup"><span data-stu-id="5da5e-108">Repeatedly constructing the same objects or data structures for multiple requests.</span></span>
- <span data-ttu-id="5da5e-109">Nombre excessif d’appels à un service distant disposant d’un quota de service et qui limite les clients après une certaine limite.</span><span class="sxs-lookup"><span data-stu-id="5da5e-109">Making excessive calls to a remote service that has a service quota and throttles clients past a certain limit.</span></span>

<span data-ttu-id="5da5e-110">Ces problèmes peuvent à leur tour entraîner un allongement du temps de réponse, une contention accrue dans le magasin de données et une faible évolutivité.</span><span class="sxs-lookup"><span data-stu-id="5da5e-110">In turn, these problems can lead to poor response times, increased contention in the data store, and poor scalability.</span></span>

<span data-ttu-id="5da5e-111">L’exemple suivant utilise Entity Framework pour se connecter à une base de données.</span><span class="sxs-lookup"><span data-stu-id="5da5e-111">The following example uses Entity Framework to connect to a database.</span></span> <span data-ttu-id="5da5e-112">Chaque requête du client entraîne un appel à la base de données, même si plusieurs requêtes extraient exactement les mêmes données.</span><span class="sxs-lookup"><span data-stu-id="5da5e-112">Every client request results in a call to the database, even if multiple requests are fetching exactly the same data.</span></span> <span data-ttu-id="5da5e-113">Le coût des requêtes répétées, en termes de frais de surcharge d’E/S et d’accès aux données, peut rapidement augmenter.</span><span class="sxs-lookup"><span data-stu-id="5da5e-113">The cost of repeated requests, in terms of I/O overhead and data access charges, can accumulate quickly.</span></span>

```csharp
public class PersonRepository : IPersonRepository
{
    public async Task<Person> GetAsync(int id)
    {
        using (var context = new AdventureWorksContext())
        {
            return await context.People
                .Where(p => p.Id == id)
                .FirstOrDefaultAsync()
                .ConfigureAwait(false);
        }
    }
}
```

<span data-ttu-id="5da5e-114">Vous trouverez l’exemple complet [ici][sample-app].</span><span class="sxs-lookup"><span data-stu-id="5da5e-114">You can find the complete sample [here][sample-app].</span></span>

<span data-ttu-id="5da5e-115">Cet antimodèle survient généralement pour les raisons suivantes :</span><span class="sxs-lookup"><span data-stu-id="5da5e-115">This antipattern typically occurs because:</span></span>

- <span data-ttu-id="5da5e-116">Le fait de ne pas utiliser de cache est plus simple à implémenter et fonctionne bien en cas de faible charge.</span><span class="sxs-lookup"><span data-stu-id="5da5e-116">Not using a cache is simpler to implement, and it works fine under low loads.</span></span> <span data-ttu-id="5da5e-117">La mise en cache rend le code plus complexe.</span><span class="sxs-lookup"><span data-stu-id="5da5e-117">Caching makes the code more complicated.</span></span>
- <span data-ttu-id="5da5e-118">Les avantages et les inconvénients liés à l’utilisation d’un cache ne sont pas clairement compris.</span><span class="sxs-lookup"><span data-stu-id="5da5e-118">The benefits and drawbacks of using a cache are not clearly understood.</span></span>
- <span data-ttu-id="5da5e-119">Un problème se pose quant à la surcharge liée à la gestion de l’exactitude et de l’actualisation des données mises en cache.</span><span class="sxs-lookup"><span data-stu-id="5da5e-119">There is concern about the overhead of maintaining the accuracy and freshness of cached data.</span></span>
- <span data-ttu-id="5da5e-120">Une application a été migrée depuis un système local, où la latence du réseau n’était pas un problème. De plus, le système s’exécutait sur un matériel coûteux très performant, si bien que la mise en cache n’a pas été prise en compte dans de la conception d’origine.</span><span class="sxs-lookup"><span data-stu-id="5da5e-120">An application was migrated from an on-premises system, where network latency was not an issue, and the system ran on expensive high-performance hardware, so caching wasn't considered in the original design.</span></span>
- <span data-ttu-id="5da5e-121">Les développeurs ne savent pas que la mise en cache est une possibilité dans un scénario donné.</span><span class="sxs-lookup"><span data-stu-id="5da5e-121">Developers aren't aware that caching is a possibility in a given scenario.</span></span> <span data-ttu-id="5da5e-122">Par exemple, les développeurs ne pensent pas à utiliser les ETag lors de l’implémentation d’une API web.</span><span class="sxs-lookup"><span data-stu-id="5da5e-122">For example, developers may not think of using ETags when implementing a web API.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="5da5e-123">Comment corriger le problème</span><span class="sxs-lookup"><span data-stu-id="5da5e-123">How to fix the problem</span></span>

<span data-ttu-id="5da5e-124">La stratégie de mise en cache la plus populaire est la stratégie *à la demande* ou *cache-aside*.</span><span class="sxs-lookup"><span data-stu-id="5da5e-124">The most popular caching strategy is the *on-demand* or *cache-aside* strategy.</span></span>

- <span data-ttu-id="5da5e-125">Lors de la lecture, l’application tente de lire les données à partir du cache.</span><span class="sxs-lookup"><span data-stu-id="5da5e-125">On read, the application tries to read the data from the cache.</span></span> <span data-ttu-id="5da5e-126">Si les données ne sont pas dans le cache, l’application les récupère à partir de la source de données et les ajoute au cache.</span><span class="sxs-lookup"><span data-stu-id="5da5e-126">If the data isn't in the cache, the application retrieves it from the data source and adds it to the cache.</span></span>
- <span data-ttu-id="5da5e-127">Lors de l’écriture, l’application écrit la modification directement dans la source de données et supprime l’ancienne valeur du cache.</span><span class="sxs-lookup"><span data-stu-id="5da5e-127">On write, the application writes the change directly to the data source and removes the old value from the cache.</span></span> <span data-ttu-id="5da5e-128">Elle est récupérée et ajoutée au cache dès qu’elle est nécessaire.</span><span class="sxs-lookup"><span data-stu-id="5da5e-128">It will be retrieved and added to the cache the next time it is required.</span></span>

<span data-ttu-id="5da5e-129">Cette approche est appropriée pour les données changeant fréquemment.</span><span class="sxs-lookup"><span data-stu-id="5da5e-129">This approach is suitable for data that changes frequently.</span></span> <span data-ttu-id="5da5e-130">Voici l’exemple précédent mis à jour pour utiliser le modèle [Cache-Aside][cache-aside-pattern].</span><span class="sxs-lookup"><span data-stu-id="5da5e-130">Here is the previous example updated to use the [Cache-Aside][cache-aside-pattern] pattern.</span></span>

```csharp
public class CachedPersonRepository : IPersonRepository
{
    private readonly PersonRepository _innerRepository;

    public CachedPersonRepository(PersonRepository innerRepository)
    {
        _innerRepository = innerRepository;
    }

    public async Task<Person> GetAsync(int id)
    {
        return await CacheService.GetAsync<Person>("p:" + id, () => _innerRepository.GetAsync(id)).ConfigureAwait(false);
    }
}

public class CacheService
{
    private static ConnectionMultiplexer _connection;

    public static async Task<T> GetAsync<T>(string key, Func<Task<T>> loadCache, double expirationTimeInMinutes)
    {
        IDatabase cache = Connection.GetDatabase();
        T value = await GetAsync<T>(cache, key).ConfigureAwait(false);
        if (value == null)
        {
            // Value was not found in the cache. Call the lambda to get the value from the database.
            value = await loadCache().ConfigureAwait(false);
            if (value != null)
            {
                // Add the value to the cache.
                await SetAsync(cache, key, value, expirationTimeInMinutes).ConfigureAwait(false);
            }
        }
        return value;
    }
}
```

<span data-ttu-id="5da5e-131">Notez que la méthode `GetAsync` appelle désormais la classe `CacheService` au lieu d’appeler directement la base de données.</span><span class="sxs-lookup"><span data-stu-id="5da5e-131">Notice that the `GetAsync` method now calls the `CacheService` class, rather than calling the database directly.</span></span> <span data-ttu-id="5da5e-132">La classe `CacheService` tente d’abord d’obtenir l’élément à partir du Cache Redis Microsoft Azure.</span><span class="sxs-lookup"><span data-stu-id="5da5e-132">The `CacheService` class first tries to get the item from Azure Redis Cache.</span></span> <span data-ttu-id="5da5e-133">Si la valeur est introuvable dans le Cache Redis, la méthode `CacheService` appelle une fonction lambda qui lui a été transmise par l’appelant.</span><span class="sxs-lookup"><span data-stu-id="5da5e-133">If the value isn't found in Redis Cache, the `CacheService` invokes a lambda function that was passed to it by the caller.</span></span> <span data-ttu-id="5da5e-134">La fonction lambda est responsable de la récupération des données à partir de la base de données.</span><span class="sxs-lookup"><span data-stu-id="5da5e-134">The lambda function is responsible for fetching the data from the database.</span></span> <span data-ttu-id="5da5e-135">Cette implémentation dissocie le référentiel à partir de la solution de mise en cache spécifique et l’élément `CacheService` de la base de données.</span><span class="sxs-lookup"><span data-stu-id="5da5e-135">This implementation decouples the repository from the particular caching solution, and decouples the `CacheService` from the database.</span></span>

## <a name="considerations"></a><span data-ttu-id="5da5e-136">Considérations</span><span class="sxs-lookup"><span data-stu-id="5da5e-136">Considerations</span></span>

- <span data-ttu-id="5da5e-137">Si le cache n’est pas disponible, peut-être en raison d’une défaillance passagère, il ne renvoie aucune erreur au client.</span><span class="sxs-lookup"><span data-stu-id="5da5e-137">If the cache is unavailable, perhaps because of a transient failure, don't return an error to the client.</span></span> <span data-ttu-id="5da5e-138">Au lieu de cela, il extrait les données de la source de données d’origine.</span><span class="sxs-lookup"><span data-stu-id="5da5e-138">Instead, fetch the data from the original data source.</span></span> <span data-ttu-id="5da5e-139">Toutefois, prenez en compte le fait que lors de la récupération du cache, le magasin d’origine peut être submergé de requêtes, ce qui entraîne des délais d’attente et des échecs de connexion.</span><span class="sxs-lookup"><span data-stu-id="5da5e-139">However, be aware that while the cache is being recovered, the original data store could be swamped with requests, resulting in timeouts and failed connections.</span></span> <span data-ttu-id="5da5e-140">Après tout, ceci est une des raisons pour lesquelles on utilise un cache en premier lieu. Utilisez une technique telle que le [modèle de disjoncteur] [ circuit-breaker] pour éviter de surcharger la source de données.</span><span class="sxs-lookup"><span data-stu-id="5da5e-140">(After all, this is one of the motivations for using a cache in the first place.) Use a technique such as the [Circuit Breaker pattern][circuit-breaker] to avoid overwhelming the data source.</span></span>

- <span data-ttu-id="5da5e-141">Les applications qui mettent en cache les données non statiques doivent être conçues pour prendre en charge une cohérence éventuelle.</span><span class="sxs-lookup"><span data-stu-id="5da5e-141">Applications that cache nonstatic data should be designed to support eventual consistency.</span></span>

- <span data-ttu-id="5da5e-142">Pour les API web, vous pouvez prendre en charge la mise en cache côté client, y compris un en-tête Cache-Control dans les messages de requête et de réponse, et l’utilisation des ETags pour identifier les versions des objets.</span><span class="sxs-lookup"><span data-stu-id="5da5e-142">For web APIs, you can support client-side caching by including a Cache-Control header in request and response messages, and using ETags to identify versions of objects.</span></span> <span data-ttu-id="5da5e-143">Pour plus d’informations, consultez [API implementation][api-implementation] (Implémentation de l’API).</span><span class="sxs-lookup"><span data-stu-id="5da5e-143">For more information, see [API implementation][api-implementation].</span></span>

- <span data-ttu-id="5da5e-144">Vous n’êtes pas obligé de mettre en cache des entités complètes.</span><span class="sxs-lookup"><span data-stu-id="5da5e-144">You don't have to cache entire entities.</span></span> <span data-ttu-id="5da5e-145">Si la majeure partie d’une entité est statique, et que seule une petite partie est modifiée fréquemment, mettez en cache les éléments statiques et récupérez les éléments dynamiques de la source de données.</span><span class="sxs-lookup"><span data-stu-id="5da5e-145">If most of an entity is static but only a small piece changes frequently, cache the static elements and retrieve the dynamic elements from the data source.</span></span> <span data-ttu-id="5da5e-146">Cette approche peut contribuer à réduire le volume d’E/S en cours d’exécution par rapport à la source de données.</span><span class="sxs-lookup"><span data-stu-id="5da5e-146">This approach can help to reduce the volume of I/O being performed against the data source.</span></span>

- <span data-ttu-id="5da5e-147">Dans certains cas, si des données volatiles sont de courte durée, il peut être utile de les mettre en cache.</span><span class="sxs-lookup"><span data-stu-id="5da5e-147">In some cases, if volatile data is short-lived, it can be useful to cache it.</span></span> <span data-ttu-id="5da5e-148">Prenons par exemple un appareil qui envoie continuellement des mises à jour.</span><span class="sxs-lookup"><span data-stu-id="5da5e-148">For example, consider a device that continually sends status updates.</span></span> <span data-ttu-id="5da5e-149">Il peut être judicieux de mettre en cache ces informations dès leur arrivée et de ne pas les écrire dans un magasin persistant.</span><span class="sxs-lookup"><span data-stu-id="5da5e-149">It might make sense to cache this information as it arrives, and not write it to a persistent store at all.</span></span>

- <span data-ttu-id="5da5e-150">Pour empêcher les données de devenir obsolètes, de nombreuses solutions de mise en cache prennent en charge des périodes d’expiration configurables, afin que les données soient supprimées automatiquement du cache après un intervalle spécifique.</span><span class="sxs-lookup"><span data-stu-id="5da5e-150">To prevent data from becoming stale, many caching solutions support configurable expiration periods, so that data is automatically removed from the cache after a specified interval.</span></span> <span data-ttu-id="5da5e-151">Vous devrez peut-être régler l’heure d’expiration de votre scénario.</span><span class="sxs-lookup"><span data-stu-id="5da5e-151">You may need to tune the expiration time for your scenario.</span></span> <span data-ttu-id="5da5e-152">Les données hautement statiques peuvent rester dans le cache plus longtemps que les données volatiles qui deviennent rapidement obsolètes.</span><span class="sxs-lookup"><span data-stu-id="5da5e-152">Data that is highly static can stay in the cache for longer periods than volatile data that may become stale quickly.</span></span>

- <span data-ttu-id="5da5e-153">Si la solution de mise en cache ne propose pas d’expiration intégrée, vous devrez peut-être implémenter un processus en arrière-plan qui balaie occasionnellement le cache et l’empêche de croître sans limite.</span><span class="sxs-lookup"><span data-stu-id="5da5e-153">If the caching solution doesn't provide built-in expiration, you may need to implement a background process that occasionally sweeps the cache, to prevent it from growing without limits.</span></span>

- <span data-ttu-id="5da5e-154">Outre la mise en cache des données à partir d’une source de données externe, vous pouvez utiliser la mise en cache pour enregistrer les résultats de calculs complexes.</span><span class="sxs-lookup"><span data-stu-id="5da5e-154">Besides caching data from an external data source, you can use caching to save the results of complex computations.</span></span> <span data-ttu-id="5da5e-155">Avant cela, il convient toutefois d’instrumenter l’application pour déterminer si elle est réellement liée à l’UC.</span><span class="sxs-lookup"><span data-stu-id="5da5e-155">Before you do that, however, instrument the application to determine whether the application is really CPU bound.</span></span>

- <span data-ttu-id="5da5e-156">Il peut être utile d’amorcer le cache au démarrage de l’application.</span><span class="sxs-lookup"><span data-stu-id="5da5e-156">It might be useful to prime the cache when the application starts.</span></span> <span data-ttu-id="5da5e-157">Alimentez le cache avec les données les plus susceptibles d’être utilisées.</span><span class="sxs-lookup"><span data-stu-id="5da5e-157">Populate the cache with the data that is most likely to be used.</span></span>

- <span data-ttu-id="5da5e-158">Incluez toujours une instrumentation qui détecte les correspondances et les absences dans le cache.</span><span class="sxs-lookup"><span data-stu-id="5da5e-158">Always include instrumentation that detects cache hits and cache misses.</span></span> <span data-ttu-id="5da5e-159">Ces informations permettent de paramétrer des stratégies de mise en cache, comme pour définir les données à mettre en cache et la durée de maintien en cache avant leur expiration.</span><span class="sxs-lookup"><span data-stu-id="5da5e-159">Use this information to tune caching policies, such what data to cache, and how long to hold data in the cache before it expires.</span></span>

- <span data-ttu-id="5da5e-160">Si l’absence de mise en cache entraîne un goulot d’étranglement, l’ajout de mise en cache peut augmenter le volume de requêtes au point de surcharger le serveur web frontal.</span><span class="sxs-lookup"><span data-stu-id="5da5e-160">If the lack of caching is a bottleneck, then adding caching may increase the volume of requests so much that the web front end becomes overloaded.</span></span> <span data-ttu-id="5da5e-161">Les clients peuvent commencer à recevoir des erreurs HTTP 503 (Service indisponible).</span><span class="sxs-lookup"><span data-stu-id="5da5e-161">Clients may start to receive HTTP 503 (Service Unavailable) errors.</span></span> <span data-ttu-id="5da5e-162">Cela indique que vous devez augmenter la taille des instances du serveur frontal.</span><span class="sxs-lookup"><span data-stu-id="5da5e-162">These are an indication that you should scale out the front end.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="5da5e-163">Comment détecter le problème</span><span class="sxs-lookup"><span data-stu-id="5da5e-163">How to detect the problem</span></span>

<span data-ttu-id="5da5e-164">Vous pouvez effectuer les étapes suivantes afin de déterminer si l’absence de mise en cache est à l’origine des problèmes de performances :</span><span class="sxs-lookup"><span data-stu-id="5da5e-164">You can perform the following steps to help identify whether lack of caching is causing performance problems:</span></span>

1. <span data-ttu-id="5da5e-165">Vérifiez la conception de l’application.</span><span class="sxs-lookup"><span data-stu-id="5da5e-165">Review the application design.</span></span> <span data-ttu-id="5da5e-166">Effectuez un inventaire de tous les magasins de données utilisés par l’application.</span><span class="sxs-lookup"><span data-stu-id="5da5e-166">Take an inventory of all the data stores that the application uses.</span></span> <span data-ttu-id="5da5e-167">Pour chacun, déterminez si l’application utilise un cache.</span><span class="sxs-lookup"><span data-stu-id="5da5e-167">For each, determine whether the application is using a cache.</span></span> <span data-ttu-id="5da5e-168">Si possible, déterminez la fréquence à laquelle les données sont modifiées.</span><span class="sxs-lookup"><span data-stu-id="5da5e-168">If possible, determine how frequently the data changes.</span></span> <span data-ttu-id="5da5e-169">Les données qui évoluent lentement et les données de référence statiques fréquemment lues sont dans un premier temps de bonnes candidates à la mise en cache.</span><span class="sxs-lookup"><span data-stu-id="5da5e-169">Good initial candidates for caching include data that changes slowly, and static reference data that is read frequently.</span></span>

2. <span data-ttu-id="5da5e-170">Instrumentez l’application et analysez le système pour déterminer la fréquence à laquelle l’application récupère des données ou calcule des informations.</span><span class="sxs-lookup"><span data-stu-id="5da5e-170">Instrument the application and monitor the live system to find out how frequently the application retrieves data or calculates information.</span></span>

3. <span data-ttu-id="5da5e-171">Profilez l’application dans un environnement de test pour capturer des métriques de bas niveau concernant la surcharge associée aux opérations d’accès aux données ou aux calculs effectués fréquemment.</span><span class="sxs-lookup"><span data-stu-id="5da5e-171">Profile the application in a test environment to capture low-level metrics about the overhead associated with data access operations or other frequently performed calculations.</span></span>

4. <span data-ttu-id="5da5e-172">Effectuez un test de charge dans un environnement de test pour identifier la manière dont le système répond sous une charge de travail normale et sous une charge importante.</span><span class="sxs-lookup"><span data-stu-id="5da5e-172">Perform load testing in a test environment to identify how the system responds under a normal workload and under heavy load.</span></span> <span data-ttu-id="5da5e-173">Le test de charge doit simuler le modèle d’accès aux données observé dans l’environnement de production à l’aide de charges de travail réalistes.</span><span class="sxs-lookup"><span data-stu-id="5da5e-173">Load testing should simulate the pattern of data access observed in the production environment using realistic workloads.</span></span>

5. <span data-ttu-id="5da5e-174">Examinez les statistiques d’accès aux données pour les magasins de données sous-jacents et vérifiez la fréquence à laquelle les mêmes requêtes de données sont répétées.</span><span class="sxs-lookup"><span data-stu-id="5da5e-174">Examine the data access statistics for the underlying data stores and review how often the same data requests are repeated.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="5da5e-175">Exemple de diagnostic</span><span class="sxs-lookup"><span data-stu-id="5da5e-175">Example diagnosis</span></span>

<span data-ttu-id="5da5e-176">Les sections suivantes appliquent ces étapes à l’exemple d’application décrit précédemment.</span><span class="sxs-lookup"><span data-stu-id="5da5e-176">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="instrument-the-application-and-monitor-the-live-system"></a><span data-ttu-id="5da5e-177">Instrumenter l’application et analyser le système dynamique</span><span class="sxs-lookup"><span data-stu-id="5da5e-177">Instrument the application and monitor the live system</span></span>

<span data-ttu-id="5da5e-178">Instrumentez l’application et analysez-la pour obtenir des informations sur les requêtes spécifiques effectuées par les utilisateurs lorsque l’application est en production.</span><span class="sxs-lookup"><span data-stu-id="5da5e-178">Instrument the application and monitor it to get information about the specific requests that users make while the application is in production.</span></span>

<span data-ttu-id="5da5e-179">L’illustration suivante montre les données d’analyse capturées par [New Relic] [ NewRelic] pendant un test de charge.</span><span class="sxs-lookup"><span data-stu-id="5da5e-179">The following image shows monitoring data captured by [New Relic][NewRelic] during a load test.</span></span> <span data-ttu-id="5da5e-180">Dans ce cas, la seule opération HTTP GET effectuée est `Person/GetAsync`.</span><span class="sxs-lookup"><span data-stu-id="5da5e-180">In this case, the only HTTP GET operation performed is `Person/GetAsync`.</span></span> <span data-ttu-id="5da5e-181">Toutefois, dans un environnement de production dynamique, le fait de connaître la fréquence relative à laquelle chaque requête est effectuée peut vous donner un aperçu des ressources à mettre en cache.</span><span class="sxs-lookup"><span data-stu-id="5da5e-181">But in a live production environment, knowing the relative frequency that each request is performed can give you insight into which resources should be cached.</span></span>

![New Relic montrant des requêtes de serveur pour l’application CachingDemo][NewRelic-server-requests]

<span data-ttu-id="5da5e-183">Si vous avez besoin d’une analyse plus approfondie, vous pouvez utiliser un générateur de profils pour capturer les données dont les performances sont faibles dans un environnement de test (et non le système de production).</span><span class="sxs-lookup"><span data-stu-id="5da5e-183">If you need a deeper analysis, you can use a profiler to capture low-level performance data in a test environment (not the production system).</span></span> <span data-ttu-id="5da5e-184">Examinez les métriques, telles que le taux de demandes d’E/S, l’utilisation de la mémoire et du processeur.</span><span class="sxs-lookup"><span data-stu-id="5da5e-184">Look at metrics such as I/O request rates, memory usage, and CPU utilization.</span></span> <span data-ttu-id="5da5e-185">Ces dernières peuvent montrer un grand nombre de requêtes vers un magasin de données ou un service ou un traitement répété qui effectue le même calcul.</span><span class="sxs-lookup"><span data-stu-id="5da5e-185">These metrics may show a large number of requests to a data store or service, or repeated processing that performs the same calculation.</span></span>

### <a name="load-test-the-application"></a><span data-ttu-id="5da5e-186">Effectuer un test de charge de l’application</span><span class="sxs-lookup"><span data-stu-id="5da5e-186">Load test the application</span></span>

<span data-ttu-id="5da5e-187">Le graphique suivant montre les résultats du test de charge de l’exemple d’application.</span><span class="sxs-lookup"><span data-stu-id="5da5e-187">The following graph shows the results of load testing the sample application.</span></span> <span data-ttu-id="5da5e-188">Le test de charge simule une charge par étape pouvant atteindre 800 utilisateurs effectuant une série classique d’opérations.</span><span class="sxs-lookup"><span data-stu-id="5da5e-188">The load test simulates a step load of up to 800 users performing a typical series of operations.</span></span>

![Résultats du test de charge portant sur les performances pour le scénario de non mise en cache][Performance-Load-Test-Results-Uncached]

<span data-ttu-id="5da5e-190">Le nombre de tests réussis exécutés chaque seconde se stabilise et les requêtes supplémentaires sont ralenties en conséquence.</span><span class="sxs-lookup"><span data-stu-id="5da5e-190">The number of successful tests performed each second reaches a plateau, and additional requests are slowed as a result.</span></span> <span data-ttu-id="5da5e-191">La durée moyenne du test augmente régulièrement avec la charge de travail.</span><span class="sxs-lookup"><span data-stu-id="5da5e-191">The average test time steadily increases with the workload.</span></span> <span data-ttu-id="5da5e-192">Le temps de réponse se stabilise dès que la charge utilisateur atteint un pic.</span><span class="sxs-lookup"><span data-stu-id="5da5e-192">The response time levels off once the user load peaks.</span></span>

### <a name="examine-data-access-statistics"></a><span data-ttu-id="5da5e-193">Examiner les statistiques d’accès aux données</span><span class="sxs-lookup"><span data-stu-id="5da5e-193">Examine data access statistics</span></span>

<span data-ttu-id="5da5e-194">Les statistiques d’accès aux données et les autres informations fournies par un magasin de données peuvent donner des informations utiles, comme savoir quelles sont les requêtes les plus fréquemment répétées.</span><span class="sxs-lookup"><span data-stu-id="5da5e-194">Data access statistics and other information provided by a data store can give useful information, such as which queries are repeated most frequently.</span></span> <span data-ttu-id="5da5e-195">Par exemple, dans Microsoft SQL Server, la vue de gestion `sys.dm_exec_query_stats` comporte des informations statistiques pour les requêtes récemment exécutées.</span><span class="sxs-lookup"><span data-stu-id="5da5e-195">For example, in Microsoft SQL Server, the `sys.dm_exec_query_stats` management view has statistical information for recently executed queries.</span></span> <span data-ttu-id="5da5e-196">Le texte de chaque requête est disponible dans la vue `sys.dm_exec-query_plan`.</span><span class="sxs-lookup"><span data-stu-id="5da5e-196">The text for each query is available in the `sys.dm_exec-query_plan` view.</span></span> <span data-ttu-id="5da5e-197">Vous pouvez utiliser un outil tel que SQL Server Management Studio pour exécuter la requête SQL suivante et déterminer la fréquence à laquelle les requêtes sont exécutées.</span><span class="sxs-lookup"><span data-stu-id="5da5e-197">You can use a tool such as SQL Server Management Studio to run the following SQL query and determine how frequently queries are performed.</span></span>

```SQL
SELECT UseCounts, Text, Query_Plan
FROM sys.dm_exec_cached_plans
CROSS APPLY sys.dm_exec_sql_text(plan_handle)
CROSS APPLY sys.dm_exec_query_plan(plan_handle)
```

<span data-ttu-id="5da5e-198">La colonne `UseCount` dans les résultats indique la fréquence d’exécution de chaque requête.</span><span class="sxs-lookup"><span data-stu-id="5da5e-198">The `UseCount` column in the results indicates how frequently each query is run.</span></span> <span data-ttu-id="5da5e-199">L’illustration suivante montre que la troisième requête a été exécutée plus de 250 000 fois, ce qui est beaucoup plus que toute autre requête.</span><span class="sxs-lookup"><span data-stu-id="5da5e-199">The following image shows that the third query was run more than 250,000 times, significantly more than any other query.</span></span>

![Résultats de l’interrogation des vues de gestion dynamique dans SQL Server Management Server][Dynamic-Management-Views]

<span data-ttu-id="5da5e-201">Voici la requête SQL à l’origine des si nombreuses requêtes de base de données :</span><span class="sxs-lookup"><span data-stu-id="5da5e-201">Here is the SQL query that is causing so many database requests:</span></span>

```SQL
(@p__linq__0 int)SELECT TOP (2)
[Extent1].[BusinessEntityId] AS [BusinessEntityId],
[Extent1].[FirstName] AS [FirstName],
[Extent1].[LastName] AS [LastName]
FROM [Person].[Person] AS [Extent1]
WHERE [Extent1].[BusinessEntityId] = @p__linq__0
```

<span data-ttu-id="5da5e-202">Il s’agit de la requête qu’Entity Framework génère dans la méthode `GetByIdAsync` illustrée précédemment.</span><span class="sxs-lookup"><span data-stu-id="5da5e-202">This is the query that Entity Framework generates in `GetByIdAsync` method shown earlier.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="5da5e-203">Implémenter la solution et vérifier le résultat</span><span class="sxs-lookup"><span data-stu-id="5da5e-203">Implement the solution and verify the result</span></span>

<span data-ttu-id="5da5e-204">Après avoir intégré un cache, répétez les tests de charge et comparez les résultats avec les tests de charge précédents sans un cache.</span><span class="sxs-lookup"><span data-stu-id="5da5e-204">After you incorporate a cache, repeat the load tests and compare the results to the earlier load tests without a cache.</span></span> <span data-ttu-id="5da5e-205">Voici les résultats du test de charge après l’ajout d’un cache à l’exemple d’application.</span><span class="sxs-lookup"><span data-stu-id="5da5e-205">Here are the load test results after adding a cache to the sample application.</span></span>

![Résultats du test de charge portant sur les performances pour le scénario de mise en cache][Performance-Load-Test-Results-Cached]

<span data-ttu-id="5da5e-207">Le volume de tests réussis se stabilise, mais à une charge utilisateur plus élevée.</span><span class="sxs-lookup"><span data-stu-id="5da5e-207">The volume of successful tests still reaches a plateau, but at a higher user load.</span></span> <span data-ttu-id="5da5e-208">Le taux de requêtes à cette charge est beaucoup plus important que le précédent.</span><span class="sxs-lookup"><span data-stu-id="5da5e-208">The request rate at this load is significantly higher than earlier.</span></span> <span data-ttu-id="5da5e-209">La durée moyenne du test augmente sans cesse avec la charge, toutefois le temps de réponse maximal est de 0,05 ms, comparé à 1 ms précédemment &mdash;, soit une amélioration de 20&times;.</span><span class="sxs-lookup"><span data-stu-id="5da5e-209">Average test time still increases with load, but the maximum response time is 0.05 ms, compared with 1ms earlier &mdash; a 20&times; improvement.</span></span>

## <a name="related-resources"></a><span data-ttu-id="5da5e-210">Ressources associées</span><span class="sxs-lookup"><span data-stu-id="5da5e-210">Related resources</span></span>

- <span data-ttu-id="5da5e-211">[Meilleures pratiques de mise en œuvre des API][api-implementation]</span><span class="sxs-lookup"><span data-stu-id="5da5e-211">[API implementation best practices][api-implementation]</span></span>
- <span data-ttu-id="5da5e-212">[Modèle Cache-Aside][cache-aside-pattern]</span><span class="sxs-lookup"><span data-stu-id="5da5e-212">[Cache-Aside pattern][cache-aside-pattern]</span></span>
- <span data-ttu-id="5da5e-213">[Meilleures pratiques en matière de mise en cache][caching-guidance]</span><span class="sxs-lookup"><span data-stu-id="5da5e-213">[Caching best practices][caching-guidance]</span></span>
- <span data-ttu-id="5da5e-214">[Modèle Disjoncteur][circuit-breaker]</span><span class="sxs-lookup"><span data-stu-id="5da5e-214">[Circuit Breaker pattern][circuit-breaker]</span></span>

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/NoCaching
[cache-aside-pattern]: /azure/architecture/patterns/cache-aside
[caching-guidance]: ../../best-practices/caching.md
[circuit-breaker]: ../../patterns/circuit-breaker.md
[api-implementation]: ../../best-practices/api-implementation.md#optimizing-client-side-data-access
[NewRelic]: https://newrelic.com/partner/azure
[NewRelic-server-requests]: _images/New-Relic.jpg
[Performance-Load-Test-Results-Uncached]:_images/InitialLoadTestResults.jpg
[Dynamic-Management-Views]: _images/SQLServerManagementStudio.jpg
[Performance-Load-Test-Results-Cached]: _images/CachedLoadTestResults.jpg
