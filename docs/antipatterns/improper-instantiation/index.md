---
title: Antimodèle d’instanciation incorrect
titleSuffix: Performance antipatterns for cloud apps
description: 'Évitez de créer continuellement de nouvelles instances d’un objet destiné à être créé une seule fois, puis partagé.'
author: dragon119
ms.date: 06/05/2017
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
---

# <a name="improper-instantiation-antipattern"></a><span data-ttu-id="819de-103">Antimodèle d’instanciation incorrect</span><span class="sxs-lookup"><span data-stu-id="819de-103">Improper Instantiation antipattern</span></span>

<span data-ttu-id="819de-104">Le fait de créer continuellement de nouvelles instances d’un objet destiné à être créé une seule fois, puis partagé, peut affecter les performances.</span><span class="sxs-lookup"><span data-stu-id="819de-104">It can hurt performance to continually create new instances of an object that is meant to be created once and then shared.</span></span>

## <a name="problem-description"></a><span data-ttu-id="819de-105">Description du problème</span><span class="sxs-lookup"><span data-stu-id="819de-105">Problem description</span></span>

<span data-ttu-id="819de-106">De nombreuses bibliothèques fournissent des abstractions de ressources externes.</span><span class="sxs-lookup"><span data-stu-id="819de-106">Many libraries provide abstractions of external resources.</span></span> <span data-ttu-id="819de-107">En interne, ces classes gèrent généralement leurs propres connexions à la ressource, en agissant comme des répartiteurs que les clients peuvent utiliser pour accéder à la ressource.</span><span class="sxs-lookup"><span data-stu-id="819de-107">Internally, these classes typically manage their own connections to the resource, acting as brokers that clients can use to access the resource.</span></span> <span data-ttu-id="819de-108">Voici quelques exemples de classes de répartiteurs appropriées pour les applications Azure :</span><span class="sxs-lookup"><span data-stu-id="819de-108">Here are some examples of broker classes that are relevant to Azure applications:</span></span>

- <span data-ttu-id="819de-109">`System.Net.Http.HttpClient`.</span><span class="sxs-lookup"><span data-stu-id="819de-109">`System.Net.Http.HttpClient`.</span></span> <span data-ttu-id="819de-110">Communique avec un service web à l’aide du protocole HTTP.</span><span class="sxs-lookup"><span data-stu-id="819de-110">Communicates with a web service using HTTP.</span></span>
- <span data-ttu-id="819de-111">`Microsoft.ServiceBus.Messaging.QueueClient`.</span><span class="sxs-lookup"><span data-stu-id="819de-111">`Microsoft.ServiceBus.Messaging.QueueClient`.</span></span> <span data-ttu-id="819de-112">Publie et reçoit des messages sur une file d’attente Service Bus.</span><span class="sxs-lookup"><span data-stu-id="819de-112">Posts and receives messages to a Service Bus queue.</span></span>
- <span data-ttu-id="819de-113">`Microsoft.Azure.Documents.Client.DocumentClient`.</span><span class="sxs-lookup"><span data-stu-id="819de-113">`Microsoft.Azure.Documents.Client.DocumentClient`.</span></span> <span data-ttu-id="819de-114">Se connecte à une instance Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="819de-114">Connects to a Cosmos DB instance</span></span>
- <span data-ttu-id="819de-115">`StackExchange.Redis.ConnectionMultiplexer`.</span><span class="sxs-lookup"><span data-stu-id="819de-115">`StackExchange.Redis.ConnectionMultiplexer`.</span></span> <span data-ttu-id="819de-116">Se connecte à Redis, y compris le Cache Redis Azure.</span><span class="sxs-lookup"><span data-stu-id="819de-116">Connects to Redis, including Azure Redis Cache.</span></span>

<span data-ttu-id="819de-117">Ces classes sont destinées à être instanciées une seule fois et réutilisées tout au long de la durée de vie d’une application.</span><span class="sxs-lookup"><span data-stu-id="819de-117">These classes are intended to be instantiated once and reused throughout the lifetime of an application.</span></span> <span data-ttu-id="819de-118">Toutefois, on pense à tort que ces classes doivent être acquises uniquement en cas de besoin et publiées rapidement.</span><span class="sxs-lookup"><span data-stu-id="819de-118">However, it's a common misunderstanding that these classes should be acquired only as necessary and released quickly.</span></span> <span data-ttu-id="819de-119">Celles répertoriées ici sont des bibliothèques .NET, mais le modèle n’est pas propre à .NET. L’exemple ASP.NET suivant crée une instance de `HttpClient` pour communiquer avec un service distant.</span><span class="sxs-lookup"><span data-stu-id="819de-119">(The ones listed here happen to be .NET libraries, but the pattern is not unique to .NET.) The following ASP.NET example creates an instance of `HttpClient` to communicate with a remote service.</span></span> <span data-ttu-id="819de-120">Vous trouverez l’exemple complet [ici][sample-app].</span><span class="sxs-lookup"><span data-stu-id="819de-120">You can find the complete sample [here][sample-app].</span></span>

```csharp
public class NewHttpClientInstancePerRequestController : ApiController
{
    // This method creates a new instance of HttpClient and disposes it for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        using (var httpClient = new HttpClient())
        {
            var hostName = HttpContext.Current.Request.Url.Host;
            var result = await httpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
            return new Product { Name = result };
        }
    }
}
```

<span data-ttu-id="819de-121">Dans une application web, cette technique n’est pas évolutive.</span><span class="sxs-lookup"><span data-stu-id="819de-121">In a web application, this technique is not scalable.</span></span> <span data-ttu-id="819de-122">Un nouvel objet `HttpClient` est créé pour chaque requête de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="819de-122">A new `HttpClient` object is created for each user request.</span></span> <span data-ttu-id="819de-123">Sous une charge importante, le serveur web peut épuiser le nombre de sockets disponibles, ce qui entraîne des erreurs `SocketException`.</span><span class="sxs-lookup"><span data-stu-id="819de-123">Under heavy load, the web server may exhaust the number of available sockets, resulting in `SocketException` errors.</span></span>

<span data-ttu-id="819de-124">Ce problème n’est pas limité à la classe `HttpClient`.</span><span class="sxs-lookup"><span data-stu-id="819de-124">This problem is not restricted to the `HttpClient` class.</span></span> <span data-ttu-id="819de-125">Les autres classes qui encapsulent des ressources ou qui sont coûteuses à créer peuvent entraîner des problèmes similaires.</span><span class="sxs-lookup"><span data-stu-id="819de-125">Other classes that wrap resources or are expensive to create might cause similar issues.</span></span> <span data-ttu-id="819de-126">L’exemple suivant crée une instance de la classe `ExpensiveToCreateService`.</span><span class="sxs-lookup"><span data-stu-id="819de-126">The following example creates an instances of the `ExpensiveToCreateService` class.</span></span> <span data-ttu-id="819de-127">Ici le problème n’est pas nécessairement l’épuisement des sockets, mais simplement la durée de création de chaque instance.</span><span class="sxs-lookup"><span data-stu-id="819de-127">Here the issue is not necessarily socket exhaustion, but simply how long it takes to create each instance.</span></span> <span data-ttu-id="819de-128">La création et la destruction continuelles des instances de cette classe peuvent nuire à l’évolutivité du système.</span><span class="sxs-lookup"><span data-stu-id="819de-128">Continually creating and destroying instances of this class might adversely affect the scalability of the system.</span></span>

```csharp
public class NewServiceInstancePerRequestController : ApiController
{
    public async Task<Product> GetProductAsync(string id)
    {
        var expensiveToCreateService = new ExpensiveToCreateService();
        return await expensiveToCreateService.GetProductByIdAsync(id);
    }
}

public class ExpensiveToCreateService
{
    public ExpensiveToCreateService()
    {
        // Simulate delay due to setup and configuration of ExpensiveToCreateService
        Thread.SpinWait(Int32.MaxValue / 100);
    }
    ...
}
```

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="819de-129">Comment corriger le problème</span><span class="sxs-lookup"><span data-stu-id="819de-129">How to fix the problem</span></span>

<span data-ttu-id="819de-130">Si la classe qui encapsule la ressource externe est partageable et thread-safe, créez une instance de singleton partagée ou un pool d’instances réutilisables de la classe.</span><span class="sxs-lookup"><span data-stu-id="819de-130">If the class that wraps the external resource is shareable and thread-safe, create a shared singleton instance or a pool of reusable instances of the class.</span></span>

<span data-ttu-id="819de-131">L’exemple suivant utilise une instance `HttpClient` statique, ce qui entraîne le partage de la connexion sur l’ensemble des requêtes.</span><span class="sxs-lookup"><span data-stu-id="819de-131">The following example uses a static `HttpClient` instance, thus sharing the connection across all requests.</span></span>

```csharp
public class SingleHttpClientInstanceController : ApiController
{
    private static readonly HttpClient httpClient;

    static SingleHttpClientInstanceController()
    {
        httpClient = new HttpClient();
    }

    // This method uses the shared instance of HttpClient for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        var hostName = HttpContext.Current.Request.Url.Host;
        var result = await httpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
        return new Product { Name = result };
    }
}
```

## <a name="considerations"></a><span data-ttu-id="819de-132">Considérations</span><span class="sxs-lookup"><span data-stu-id="819de-132">Considerations</span></span>

- <span data-ttu-id="819de-133">L’élément clé de cet antimodèle est la création et la destruction répétée d’instances d’un objet *partageable*.</span><span class="sxs-lookup"><span data-stu-id="819de-133">The key element of this antipattern is repeatedly creating and destroying instances of a *shareable* object.</span></span> <span data-ttu-id="819de-134">Si une classe n’est pas partageable (pas thread-safe), cet antimodèle ne s’applique pas.</span><span class="sxs-lookup"><span data-stu-id="819de-134">If a class is not shareable (not thread-safe), then this antipattern does not apply.</span></span>

- <span data-ttu-id="819de-135">Le type de ressource partagée peut déterminer si vous devez utiliser un singleton ou créer un pool.</span><span class="sxs-lookup"><span data-stu-id="819de-135">The type of shared resource might dictate whether you should use a singleton or create a pool.</span></span> <span data-ttu-id="819de-136">La classe `HttpClient` a été conçue pour être partagée plutôt que regroupée.</span><span class="sxs-lookup"><span data-stu-id="819de-136">The `HttpClient` class is designed to be shared rather than pooled.</span></span> <span data-ttu-id="819de-137">D’autres objets peuvent prendre en charge le regroupement pour permettre au système de répartir la charge de travail sur plusieurs instances.</span><span class="sxs-lookup"><span data-stu-id="819de-137">Other objects might support pooling, enabling the system to spread the workload across multiple instances.</span></span>

- <span data-ttu-id="819de-138">Les objets que vous partagez sur plusieurs requêtes *doivent* être thread-safe.</span><span class="sxs-lookup"><span data-stu-id="819de-138">Objects that you share across multiple requests *must* be thread-safe.</span></span> <span data-ttu-id="819de-139">La classe `HttpClient` est conçue pour être utilisée de cette manière, mais les autres classes ne peuvent pas en charge des requêtes simultanées. Par conséquent, reportez-vous à la documentation disponible.</span><span class="sxs-lookup"><span data-stu-id="819de-139">The `HttpClient` class is designed to be used in this manner, but other classes might not support concurrent requests, so check the available documentation.</span></span>

- <span data-ttu-id="819de-140">Attention lors de la définition des propriétés sur les objets partagés, car cette opération peut entraîner des conditions de concurrence.</span><span class="sxs-lookup"><span data-stu-id="819de-140">Be careful about setting properties on shared objects, as this can lead to race conditions.</span></span> <span data-ttu-id="819de-141">Par exemple, la définition de `DefaultRequestHeaders` sur la classe `HttpClient` avant chaque demande peut créer une condition de concurrence.</span><span class="sxs-lookup"><span data-stu-id="819de-141">For example, setting `DefaultRequestHeaders` on the `HttpClient` class before each request can create a race condition.</span></span> <span data-ttu-id="819de-142">Définissez ces propriétés une seule fois (par exemple, au démarrage) et créez des instances distinctes, si vous avez besoin de configurer des paramètres différents.</span><span class="sxs-lookup"><span data-stu-id="819de-142">Set such properties once (for example, during startup), and create separate instances if you need to configure different settings.</span></span>

- <span data-ttu-id="819de-143">Certains types de ressources sont rares et ne doivent pas y être conservés,</span><span class="sxs-lookup"><span data-stu-id="819de-143">Some resource types are scarce and should not be held onto.</span></span> <span data-ttu-id="819de-144">comme les connexions de base de données.</span><span class="sxs-lookup"><span data-stu-id="819de-144">Database connections are an example.</span></span> <span data-ttu-id="819de-145">Le maintien d’une connexion de base de données qui n’est pas requise peut empêcher les autres utilisateurs simultanés d’accéder à la base de données.</span><span class="sxs-lookup"><span data-stu-id="819de-145">Holding an open database connection that is not required may prevent other concurrent users from gaining access to the database.</span></span>

- <span data-ttu-id="819de-146">Dans le .NET Framework, de nombreux objets établissant des connexions à des ressources externes sont créés à l’aide de méthodes de fabrique statiques d’autres classes qui gèrent ces connexions.</span><span class="sxs-lookup"><span data-stu-id="819de-146">In the .NET Framework, many objects that establish connections to external resources are created by using static factory methods of other classes that manage these connections.</span></span> <span data-ttu-id="819de-147">Ces objets sont destinés à être enregistrés et réutilisés, plutôt que supprimés et recréés.</span><span class="sxs-lookup"><span data-stu-id="819de-147">These objects are intended to be saved and reused, rather than disposed and recreated.</span></span> <span data-ttu-id="819de-148">Par exemple, dans Microsoft Azure Service Bus, l’objet `QueueClient` est créé via un objet `MessagingFactory`.</span><span class="sxs-lookup"><span data-stu-id="819de-148">For example, in Azure Service Bus, the `QueueClient` object is created through a `MessagingFactory` object.</span></span> <span data-ttu-id="819de-149">En interne, `MessagingFactory` gère les connexions.</span><span class="sxs-lookup"><span data-stu-id="819de-149">Internally, the `MessagingFactory` manages connections.</span></span> <span data-ttu-id="819de-150">Pour plus d’informations, consultez [Meilleures pratiques relatives aux améliorations de performances à l’aide de la messagerie Service Bus][service-bus-messaging].</span><span class="sxs-lookup"><span data-stu-id="819de-150">For more information, see [Best Practices for performance improvements using Service Bus Messaging][service-bus-messaging].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="819de-151">Comment détecter le problème</span><span class="sxs-lookup"><span data-stu-id="819de-151">How to detect the problem</span></span>

<span data-ttu-id="819de-152">Ce problème inclut une baisse de débit et une augmentation du taux d’erreur, ainsi qu’un ou plusieurs symptômes parmi les suivants :</span><span class="sxs-lookup"><span data-stu-id="819de-152">Symptoms of this problem include a drop in throughput or an increased error rate, along with one or more of the following:</span></span>

- <span data-ttu-id="819de-153">Une augmentation des exceptions indiquant l’insuffisance de ressources, telles que les sockets, les connexions de base de données, les descripteurs de fichiers etc.</span><span class="sxs-lookup"><span data-stu-id="819de-153">An increase in exceptions that indicate exhaustion of resources such as sockets, database connections, file handles, and so on.</span></span>
- <span data-ttu-id="819de-154">Une utilisation de la mémoire et un nettoyage de la mémoire accrus.</span><span class="sxs-lookup"><span data-stu-id="819de-154">Increased memory use and garbage collection.</span></span>
- <span data-ttu-id="819de-155">Une augmentation de l’activité du réseau, du disque ou de la base de données.</span><span class="sxs-lookup"><span data-stu-id="819de-155">An increase in network, disk, or database activity.</span></span>

<span data-ttu-id="819de-156">Vous pouvez procéder de la manière suivante pour identifier ce problème :</span><span class="sxs-lookup"><span data-stu-id="819de-156">You can perform the following steps to help identify this problem:</span></span>

1. <span data-ttu-id="819de-157">Analysez le processus du système de production afin d’identifier les points où les temps de réponse augmentent ou ceux où le système échoue en raison d’un manque de ressources.</span><span class="sxs-lookup"><span data-stu-id="819de-157">Performing process monitoring of the production system, to identify points when response times slow down or the system fails due to lack of resources.</span></span>
2. <span data-ttu-id="819de-158">Examinez les données de télémétrie capturées à ces points pour déterminer les opérations pouvant créer et détruire des objets consommateurs de ressources.</span><span class="sxs-lookup"><span data-stu-id="819de-158">Examine the telemetry data captured at these points to determine which operations might be creating and destroying resource-consuming objects.</span></span>
3. <span data-ttu-id="819de-159">Effectuez un test de charge de chaque opération suspectée au sein d’un environnement de test contrôlé plutôt que dans le système de production.</span><span class="sxs-lookup"><span data-stu-id="819de-159">Load test each suspected operation, in a controlled test environment rather than the production system.</span></span>
4. <span data-ttu-id="819de-160">Passez en revue le code source et examinez la façon dont les objets du répartiteur sont gérés.</span><span class="sxs-lookup"><span data-stu-id="819de-160">Review the source code and examine the how broker objects are managed.</span></span>

<span data-ttu-id="819de-161">Examinez l’arborescence des appels de procédure pour les opérations lentes ou celles qui génèrent des exceptions lorsque le système est sous charge.</span><span class="sxs-lookup"><span data-stu-id="819de-161">Look at stack traces for operations that are slow-running or that generate exceptions when the system is under load.</span></span> <span data-ttu-id="819de-162">Ces informations peuvent aider à identifier la manière dont ces opérations utilisent les ressources.</span><span class="sxs-lookup"><span data-stu-id="819de-162">This information can help to identify how these operations are using resources.</span></span> <span data-ttu-id="819de-163">Les exceptions peuvent permettre de déterminer si les erreurs sont provoquées par l’épuisement des ressources partagées.</span><span class="sxs-lookup"><span data-stu-id="819de-163">Exceptions can help to determine whether errors are caused by shared resources being exhausted.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="819de-164">Exemple de diagnostic</span><span class="sxs-lookup"><span data-stu-id="819de-164">Example diagnosis</span></span>

<span data-ttu-id="819de-165">Les sections suivantes appliquent ces étapes à l’exemple d’application décrit précédemment.</span><span class="sxs-lookup"><span data-stu-id="819de-165">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="identify-points-of-slow-down-or-failure"></a><span data-ttu-id="819de-166">Identifier les points de ralentissement ou d’échec</span><span class="sxs-lookup"><span data-stu-id="819de-166">Identify points of slow down or failure</span></span>

<span data-ttu-id="819de-167">L’illustration suivante montre les résultats générés à l’aide de l’[APM New Relic][new-relic], affichant les opérations dont le temps de réponse est médiocre.</span><span class="sxs-lookup"><span data-stu-id="819de-167">The following image shows results generated using [New Relic APM][new-relic], showing operations that have a poor response time.</span></span> <span data-ttu-id="819de-168">Dans ce cas, la méthode `GetProductAsync` du contrôleur `NewHttpClientInstancePerRequest` mérite un examen plus approfondi.</span><span class="sxs-lookup"><span data-stu-id="819de-168">In this case, the `GetProductAsync` method in the `NewHttpClientInstancePerRequest` controller is worth investigating further.</span></span> <span data-ttu-id="819de-169">Notez que le taux d’erreur augmente également lorsque ces opérations sont en cours d’exécution.</span><span class="sxs-lookup"><span data-stu-id="819de-169">Notice that the error rate also increases when these operations are running.</span></span>

![Tableau de bord d’analyse New Relic montrant l’exemple d’application créant une nouvelle instance d’un objet HttpClient pour chaque requête][dashboard-new-HTTPClient-instance]

### <a name="examine-telemetry-data-and-find-correlations"></a><span data-ttu-id="819de-171">Examiner les données de télémétrie et rechercher les corrélations</span><span class="sxs-lookup"><span data-stu-id="819de-171">Examine telemetry data and find correlations</span></span>

<span data-ttu-id="819de-172">L’image suivante montre les données capturées à l’aide du thread de profilage, sur la même période correspondant à l’image précédente.</span><span class="sxs-lookup"><span data-stu-id="819de-172">The next image shows data captured using thread profiling, over the same period corresponding as the previous image.</span></span> <span data-ttu-id="819de-173">Le système consacre un temps important à l’ouverture de connexions aux sockets et davantage pour leur fermeture et le traitement des exceptions de socket.</span><span class="sxs-lookup"><span data-stu-id="819de-173">The system spends a significant time opening socket connections, and even more time closing them and handling socket exceptions.</span></span>

![Profileur de thread New Relic montrant l’exemple d’application créant une nouvelle instance d’un objet HttpClient pour chaque requête][thread-profiler-new-HTTPClient-instance]

### <a name="performing-load-testing"></a><span data-ttu-id="819de-175">Effectuer des tests de charge</span><span class="sxs-lookup"><span data-stu-id="819de-175">Performing load testing</span></span>

<span data-ttu-id="819de-176">Pour simuler les opérations courantes que les utilisateurs peuvent effectuer, utilisez le test de charge.</span><span class="sxs-lookup"><span data-stu-id="819de-176">Use load testing to simulate the typical operations that users might perform.</span></span> <span data-ttu-id="819de-177">Cela peut vous aider à identifier les parties d’un système subissant un épuisement des ressources sous différentes charges.</span><span class="sxs-lookup"><span data-stu-id="819de-177">This can help to identify which parts of a system suffer from resource exhaustion under varying loads.</span></span> <span data-ttu-id="819de-178">Effectuer ces tests dans un environnement contrôlé plutôt que dans le système de production.</span><span class="sxs-lookup"><span data-stu-id="819de-178">Perform these tests in a controlled environment rather than the production system.</span></span> <span data-ttu-id="819de-179">Le graphique suivant montre le débit des requêtes traitées par le contrôleur `NewHttpClientInstancePerRequest` alors que la charge utilisateur augmente pour atteindre 100 utilisateurs simultanés.</span><span class="sxs-lookup"><span data-stu-id="819de-179">The following graph shows the throughput of requests handled by the `NewHttpClientInstancePerRequest` controller as the user load increases to 100 concurrent users.</span></span>

![Débit de l’exemple d’application créant une nouvelle instance d’un objet HttpClient pour chaque requête][throughput-new-HTTPClient-instance]

<span data-ttu-id="819de-181">Dans un premier temps, le volume des requêtes traitées par seconde augmente à mesure que la charge de travail s’accroît.</span><span class="sxs-lookup"><span data-stu-id="819de-181">At first, the volume of requests handled per second increases as the workload increases.</span></span> <span data-ttu-id="819de-182">Toutefois, à environ 30 utilisateurs, le volume des requêtes réussies atteint une limite et le système commence à générer des exceptions.</span><span class="sxs-lookup"><span data-stu-id="819de-182">At about 30 users, however, the volume of successful requests reaches a limit, and the system starts to generate exceptions.</span></span> <span data-ttu-id="819de-183">Dès lors, le volume d’exceptions augmente progressivement avec la charge utilisateur.</span><span class="sxs-lookup"><span data-stu-id="819de-183">From then on, the volume of exceptions gradually increases with the user load.</span></span>

<span data-ttu-id="819de-184">Le test de charge a signalé ces échecs sous forme d’erreurs HTTP 500 (serveur interne).</span><span class="sxs-lookup"><span data-stu-id="819de-184">The load test reported these failures as HTTP 500 (Internal Server) errors.</span></span> <span data-ttu-id="819de-185">L’examen de la télémétrie a montré que ces erreurs ont été causées par l’exécution du système en dehors des ressources de socket, au fur et à mesure de l’augmentation du nombre d’objets `HttpClient` créés.</span><span class="sxs-lookup"><span data-stu-id="819de-185">Reviewing the telemetry showed that these errors were caused by the system running out of socket resources, as more and more `HttpClient` objects were created.</span></span>

<span data-ttu-id="819de-186">Le graphique suivant montre un test similaire pour un contrôleur qui crée l’objet `ExpensiveToCreateService` personnalisé.</span><span class="sxs-lookup"><span data-stu-id="819de-186">The next graph shows a similar test for a controller that creates the custom `ExpensiveToCreateService` object.</span></span>

![Débit de l’exemple d’application créant une nouvelle instance de ExpensiveToCreateService pour chaque requête][throughput-new-ExpensiveToCreateService-instance]

<span data-ttu-id="819de-188">Cette fois-ci, le contrôleur ne génère pas d’exceptions. Toutefois, le débit se stabilise, tandis que le temps de réponse moyen augmente selon un facteur de 20.</span><span class="sxs-lookup"><span data-stu-id="819de-188">This time, the controller does not generate any exceptions, but throughput still reaches a plateau, while the average response time increases by a factor of 20.</span></span> <span data-ttu-id="819de-189">Le graphique utilise une échelle logarithmique pour le débit et le temps de réponse. La télémétrie a montré que la création de nouvelles instances du `ExpensiveToCreateService` était la cause principale du problème.</span><span class="sxs-lookup"><span data-stu-id="819de-189">(The graph uses a logarithmic scale for response time and throughput.) Telemetry showed that creating new instances of the `ExpensiveToCreateService` was the main cause of the problem.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="819de-190">Implémenter la solution et vérifier le résultat</span><span class="sxs-lookup"><span data-stu-id="819de-190">Implement the solution and verify the result</span></span>

<span data-ttu-id="819de-191">Après avoir changé la méthode `GetProductAsync` pour partager une seule instance `HttpClient`, un deuxième test de charge a montré une amélioration des performances.</span><span class="sxs-lookup"><span data-stu-id="819de-191">After switching the `GetProductAsync` method to share a single `HttpClient` instance, a second load test showed improved performance.</span></span> <span data-ttu-id="819de-192">Aucune erreur n’a été signalée, et le système a été en mesure de gérer une charge croissante pouvant atteindre 500 requêtes par seconde.</span><span class="sxs-lookup"><span data-stu-id="819de-192">No errors were reported, and the system was able to handle an increasing load of up to 500 requests per second.</span></span> <span data-ttu-id="819de-193">Le temps de réponse moyen a été réduit de moitié, comparé au test précédent.</span><span class="sxs-lookup"><span data-stu-id="819de-193">The average response time was cut in half, compared with the previous test.</span></span>

![Débit de l’exemple d’application réutilisant la même instance d’un objet HttpClient pour chaque requête][throughput-single-HTTPClient-instance]

<span data-ttu-id="819de-195">À des fins de comparaison, l’illustration suivante montre la télémétrie de trace de pile.</span><span class="sxs-lookup"><span data-stu-id="819de-195">For comparison, the following image shows the stack trace telemetry.</span></span> <span data-ttu-id="819de-196">Cette fois-ci, le système passe la majorité de son temps à effectuer un travail réel, plutôt qu’à ouvrir et à fermer des sockets.</span><span class="sxs-lookup"><span data-stu-id="819de-196">This time, the system spends most of its time performing real work, rather than opening and closing sockets.</span></span>

![Profileur de thread New Relic montrant l’exemple d’application créant une seule instance d’un objet HttpClient pour toutes les requêtes][thread-profiler-single-HTTPClient-instance]

<span data-ttu-id="819de-198">Le graphique suivant montre un test de charge similaire utilisant une instance partagée de l’objet `ExpensiveToCreateService`.</span><span class="sxs-lookup"><span data-stu-id="819de-198">The next graph shows a similar load test using a shared instance of the `ExpensiveToCreateService` object.</span></span> <span data-ttu-id="819de-199">Là encore, le volume de requêtes traitées augmente en fonction de la charge utilisateur, tandis que le temps de réponse moyen reste faible.</span><span class="sxs-lookup"><span data-stu-id="819de-199">Again, the volume of handled requests increases in line with the user load, while the average response time remains low.</span></span>

![Débit de l’exemple d’application réutilisant la même instance d’un objet HttpClient pour chaque requête][throughput-single-ExpensiveToCreateService-instance]

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/ImproperInstantiation
[service-bus-messaging]: /azure/service-bus-messaging/service-bus-performance-improvements
[new-relic]: https://newrelic.com/application-monitoring
[throughput-new-HTTPClient-instance]: _images/HttpClientInstancePerRequest.jpg
[dashboard-new-HTTPClient-instance]: _images/HttpClientInstancePerRequestWebTransactions.jpg
[thread-profiler-new-HTTPClient-instance]: _images/HttpClientInstancePerRequestThreadProfile.jpg
[throughput-new-ExpensiveToCreateService-instance]: _images/ServiceInstancePerRequest.jpg
[throughput-single-HTTPClient-instance]: _images/SingleHttpClientInstance.jpg
[throughput-single-ExpensiveToCreateService-instance]: _images/SingleServiceInstance.jpg
[thread-profiler-single-HTTPClient-instance]: _images/SingleHttpClientInstanceThreadProfile.jpg
