---
title: Anti-modèle E/S synchrone
description: Bloquer le thread appelant lorsque l’E/S se termine peut réduire les performances et affecter l’extensibilité verticale.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 961eacb82344ec7e71aaa96fb4cd8bc530721e96
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/28/2018
ms.locfileid: "47429007"
---
# <a name="synchronous-io-antipattern"></a><span data-ttu-id="92d4b-103">Anti-modèle E/S synchrone</span><span class="sxs-lookup"><span data-stu-id="92d4b-103">Synchronous I/O antipattern</span></span>

<span data-ttu-id="92d4b-104">Bloquer le thread appelant lorsque l’E/S se termine peut réduire les performances et affecter l’extensibilité verticale.</span><span class="sxs-lookup"><span data-stu-id="92d4b-104">Blocking the calling thread while I/O completes can reduce performance and affect vertical scalability.</span></span>

## <a name="problem-description"></a><span data-ttu-id="92d4b-105">Description du problème</span><span class="sxs-lookup"><span data-stu-id="92d4b-105">Problem description</span></span>

<span data-ttu-id="92d4b-106">Une opération d’E/S synchrone bloque le thread appelant pendant que l’E/S se termine.</span><span class="sxs-lookup"><span data-stu-id="92d4b-106">A synchronous I/O operation blocks the calling thread while the I/O completes.</span></span> <span data-ttu-id="92d4b-107">Le thread appelant entre dans un état d’attente et ne peut pas effectuer de travail utile pendant cet intervalle, gaspillant des ressources de traitement.</span><span class="sxs-lookup"><span data-stu-id="92d4b-107">The calling thread enters a wait state and is unable to perform useful work during this interval, wasting processing resources.</span></span>

<span data-ttu-id="92d4b-108">Voici quelques exemples courants d’E/S :</span><span class="sxs-lookup"><span data-stu-id="92d4b-108">Common examples of I/O include:</span></span>

- <span data-ttu-id="92d4b-109">Récupération ou conservation de données dans une base de données ou n’importe quel type de stockage permanent.</span><span class="sxs-lookup"><span data-stu-id="92d4b-109">Retrieving or persisting data to a database or any type of persistent storage.</span></span>
- <span data-ttu-id="92d4b-110">Envoi d’une demande à un service web.</span><span class="sxs-lookup"><span data-stu-id="92d4b-110">Sending a request to a web service.</span></span>
- <span data-ttu-id="92d4b-111">Publication d’un message de validation ou récupération d’un message à partir d’une file d’attente.</span><span class="sxs-lookup"><span data-stu-id="92d4b-111">Posting a message or retrieving a message from a queue.</span></span>
- <span data-ttu-id="92d4b-112">Lecture sur un fichier local ou à partir de ce dernier.</span><span class="sxs-lookup"><span data-stu-id="92d4b-112">Writing to or reading from a local file.</span></span>

<span data-ttu-id="92d4b-113">Cet antimodèle survient généralement pour les raisons suivantes :</span><span class="sxs-lookup"><span data-stu-id="92d4b-113">This antipattern typically occurs because:</span></span>

- <span data-ttu-id="92d4b-114">Cela semble être la manière la plus intuitive d’effectuer une opération.</span><span class="sxs-lookup"><span data-stu-id="92d4b-114">It appears to be the most intuitive way to perform an operation.</span></span> 
- <span data-ttu-id="92d4b-115">L’application requiert une réponse d’une requête.</span><span class="sxs-lookup"><span data-stu-id="92d4b-115">The application requires a response from a request.</span></span>
- <span data-ttu-id="92d4b-116">L’application utilise une bibliothèque qui fournit uniquement des méthodes synchrones pour E/S.</span><span class="sxs-lookup"><span data-stu-id="92d4b-116">The application uses a library that only provides synchronous methods for I/O.</span></span> 
- <span data-ttu-id="92d4b-117">Une bibliothèque externe effectue les opérations d’E/S synchrones en interne.</span><span class="sxs-lookup"><span data-stu-id="92d4b-117">An external library performs synchronous I/O operations internally.</span></span> <span data-ttu-id="92d4b-118">Un seul appel d’E/S synchrone peut bloquer une chaîne d’appel entière.</span><span class="sxs-lookup"><span data-stu-id="92d4b-118">A single synchronous I/O call can block an entire call chain.</span></span>

<span data-ttu-id="92d4b-119">Le code suivant télécharge un fichier vers le stockage d’objets blob Azure.</span><span class="sxs-lookup"><span data-stu-id="92d4b-119">The following code uploads a file to Azure blob storage.</span></span> <span data-ttu-id="92d4b-120">Il existe deux emplacements où les blocs de code sont en attente d’E/S synchrones, la méthode `CreateIfNotExists` et la méthode `UploadFromStream`.</span><span class="sxs-lookup"><span data-stu-id="92d4b-120">There are two places where the code blocks waiting for synchronous I/O, the `CreateIfNotExists` method and the `UploadFromStream` method.</span></span>

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

container.CreateIfNotExists();
var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    blockBlob.UploadFromStream(fileStream);
}
```

<span data-ttu-id="92d4b-121">Voici un exemple d’attente de réponse d’un service externe.</span><span class="sxs-lookup"><span data-stu-id="92d4b-121">Here's an example of waiting for a response from an external service.</span></span> <span data-ttu-id="92d4b-122">La méthode `GetUserProfile` appelle un service distant qui retourne un `UserProfile`.</span><span class="sxs-lookup"><span data-stu-id="92d4b-122">The `GetUserProfile` method calls a remote service that returns a `UserProfile`.</span></span>

```csharp
public interface IUserProfileService
{
    UserProfile GetUserProfile();
}

public class SyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public SyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is a synchronous method that calls the synchronous GetUserProfile method.
    public UserProfile GetUserProfile()
    {
        return _userProfileService.GetUserProfile();
    }
}
```

<span data-ttu-id="92d4b-123">Vous pouvez trouver le code complet pour ces deux exemples [ici][sample-app].</span><span class="sxs-lookup"><span data-stu-id="92d4b-123">You can find the complete code for both of these examples [here][sample-app].</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="92d4b-124">Comment corriger le problème</span><span class="sxs-lookup"><span data-stu-id="92d4b-124">How to fix the problem</span></span>

<span data-ttu-id="92d4b-125">Remplacez les opérations d’E/S synchrones avec des opérations asynchrones.</span><span class="sxs-lookup"><span data-stu-id="92d4b-125">Replace synchronous I/O operations with asynchronous operations.</span></span> <span data-ttu-id="92d4b-126">Cela libère le thread actuel pour continuer à effectuer un travail pertinent plutôt que de le bloquer et contribue à améliorer l’utilisation des ressources de calcul.</span><span class="sxs-lookup"><span data-stu-id="92d4b-126">This frees the current thread to continue performing meaningful work rather than blocking, and helps improve the utilization of compute resources.</span></span> <span data-ttu-id="92d4b-127">L’exécution d’E/S de façon asynchrone est particulièrement efficace pour gérer un bond inattendu du nombre de demandes des applications clientes.</span><span class="sxs-lookup"><span data-stu-id="92d4b-127">Performing I/O asynchronously is particularly efficient for handling an unexpected surge in requests from client applications.</span></span> 

<span data-ttu-id="92d4b-128">De nombreuses bibliothèques fournissent des versions synchrones et asynchrones des méthodes.</span><span class="sxs-lookup"><span data-stu-id="92d4b-128">Many libraries provide both synchronous and asynchronous versions of methods.</span></span> <span data-ttu-id="92d4b-129">Lorsque c’est possible, utilisez les versions asynchrones.</span><span class="sxs-lookup"><span data-stu-id="92d4b-129">Whenever possible, use the asynchronous versions.</span></span> <span data-ttu-id="92d4b-130">Voici la version asynchrone de l’exemple précédent qui télécharge un fichier sur le stockage d’objets blob Azure.</span><span class="sxs-lookup"><span data-stu-id="92d4b-130">Here is the asynchronous version of the previous example that uploads a file to Azure blob storage.</span></span>

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

await container.CreateIfNotExistsAsync();

var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    await blockBlob.UploadFromStreamAsync(fileStream);
}
```

<span data-ttu-id="92d4b-131">L’opérateur `await` retourne le contrôle à l’environnement appelant pendant que l’opération asynchrone est effectuée.</span><span class="sxs-lookup"><span data-stu-id="92d4b-131">The `await` operator returns control to the calling environment while the asynchronous operation is performed.</span></span> <span data-ttu-id="92d4b-132">Le code après cette instruction se comporte comme une continuation qui s’exécute lorsque l’opération asynchrone est terminée.</span><span class="sxs-lookup"><span data-stu-id="92d4b-132">The code after this statement acts as a continuation that runs when the asynchronous operation has completed.</span></span>

<span data-ttu-id="92d4b-133">Un service bien conçu doit également fournir des opérations asynchrones.</span><span class="sxs-lookup"><span data-stu-id="92d4b-133">A well designed service should also provide asynchronous operations.</span></span> <span data-ttu-id="92d4b-134">Voici une version asynchrone du service web qui retourne les profils utilisateur.</span><span class="sxs-lookup"><span data-stu-id="92d4b-134">Here is an asynchronous version of the web service that returns user profiles.</span></span> <span data-ttu-id="92d4b-135">La méthode `GetUserProfileAsync` dépend d’une version asynchrone du service de profil utilisateur.</span><span class="sxs-lookup"><span data-stu-id="92d4b-135">The `GetUserProfileAsync` method depends on having an asynchronous version of the User Profile service.</span></span>

```csharp
public interface IUserProfileService
{
    Task<UserProfile> GetUserProfileAsync();
}

public class AsyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public AsyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is an synchronous method that calls the Task based GetUserProfileAsync method.
    public Task<UserProfile> GetUserProfileAsync()
    {
        return _userProfileService.GetUserProfileAsync();
    }
}
```

<span data-ttu-id="92d4b-136">Pour les bibliothèques qui ne fournissent pas de versions asynchrones des opérations, il peut être possible de créer des wrappers asynchrones autour des méthodes synchrones sélectionnées.</span><span class="sxs-lookup"><span data-stu-id="92d4b-136">For libraries that don't provide asynchronous versions of operations, it may be possible to create asynchronous wrappers around selected synchronous methods.</span></span> <span data-ttu-id="92d4b-137">Suivez cette approche avec précaution.</span><span class="sxs-lookup"><span data-stu-id="92d4b-137">Follow this approach with caution.</span></span> <span data-ttu-id="92d4b-138">Bien qu’elle puisse améliorer les temps de réponse sur le thread qui appelle le wrapper asynchrone, elle consomme réellement davantage de ressources.</span><span class="sxs-lookup"><span data-stu-id="92d4b-138">While it may improve responsiveness on the thread that invokes the asynchronous wrapper, it actually consumes more resources.</span></span> <span data-ttu-id="92d4b-139">Un thread supplémentaire peut être créé, résultant en une surcharge associée à la synchronisation du travail effectué par ce thread.</span><span class="sxs-lookup"><span data-stu-id="92d4b-139">An extra thread may be created, and there is overhead associated with synchronizing the work done by this thread.</span></span> <span data-ttu-id="92d4b-140">Certains inconvénients sont présentés dans ce billet de blog : [Should I expose asynchronous wrappers for synchronous methods?][async-wrappers] (Dois-je exposer les wrappers asynchrones pour les méthodes synchrones ?)</span><span class="sxs-lookup"><span data-stu-id="92d4b-140">Some tradeoffs are discussed in this blog post: [Should I expose asynchronous wrappers for synchronous methods?][async-wrappers]</span></span>

<span data-ttu-id="92d4b-141">Voici un exemple d’un wrapper asynchrone autour d’une méthode synchrone.</span><span class="sxs-lookup"><span data-stu-id="92d4b-141">Here is an example of an asynchronous wrapper around a synchronous method.</span></span>

```csharp
// Asynchronous wrapper around synchronous library method
private async Task<int> LibraryIOOperationAsync()
{
    return await Task.Run(() => LibraryIOOperation());
}
```

<span data-ttu-id="92d4b-142">Le code appelant peut désormais attendre le wrapper :</span><span class="sxs-lookup"><span data-stu-id="92d4b-142">Now the calling code can await on the wrapper:</span></span>

```csharp
// Invoke the asynchronous wrapper using a task
await LibraryIOOperationAsync();
```

## <a name="considerations"></a><span data-ttu-id="92d4b-143">Considérations</span><span class="sxs-lookup"><span data-stu-id="92d4b-143">Considerations</span></span>

- <span data-ttu-id="92d4b-144">Les opérations d’E/S qui sont censées être de très courte durée et sont peu susceptibles de provoquer des contentions peuvent être plus performantes que les opérations synchrones.</span><span class="sxs-lookup"><span data-stu-id="92d4b-144">I/O operations that are expected to be very short lived and are unlikely to cause contention might be more performant as synchronous operations.</span></span> <span data-ttu-id="92d4b-145">La lecture de petits fichiers sur un disque SSD en est un exemple.</span><span class="sxs-lookup"><span data-stu-id="92d4b-145">An example might be reading small files on an SSD drive.</span></span> <span data-ttu-id="92d4b-146">La surcharge de la distribution d’une tâche à un autre thread et la synchronisation avec ce thread lorsque la tâche se termine, peuvent dépasser les avantages d’E/S asynchrones.</span><span class="sxs-lookup"><span data-stu-id="92d4b-146">The overhead of dispatching a task to another thread, and synchronizing with that thread when the task completes, might outweigh the benefits of asynchronous I/O.</span></span> <span data-ttu-id="92d4b-147">Toutefois, ces cas sont relativement rares, et la plupart des opérations d’E/S doivent être effectuées de façon asynchrone.</span><span class="sxs-lookup"><span data-stu-id="92d4b-147">However, these cases are relatively rare, and most I/O operations should be done asynchronously.</span></span>

- <span data-ttu-id="92d4b-148">L’amélioration des performances d’E/S peut entraîner la transformation d’autres parties du système en goulots d’étranglement.</span><span class="sxs-lookup"><span data-stu-id="92d4b-148">Improving I/O performance may cause other parts of the system to become bottlenecks.</span></span> <span data-ttu-id="92d4b-149">Par exemple, le déblocage de threads peut entraîner un plus grand volume de demandes simultanées à des ressources partagées, conduisant à son tour à une limitation ou à une insuffisance de ressources.</span><span class="sxs-lookup"><span data-stu-id="92d4b-149">For example, unblocking threads might result in a higher volume of concurrent requests to shared resources, leading in turn to resource starvation or throttling.</span></span> <span data-ttu-id="92d4b-150">Si cela devient un problème, vous devrez peut-être faire évoluer le nombre de serveurs web ou partitionner les magasins de données pour réduire la contention.</span><span class="sxs-lookup"><span data-stu-id="92d4b-150">If that becomes a problem, you might need to scale out the number of web servers or partition data stores to reduce contention.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="92d4b-151">Comment détecter le problème</span><span class="sxs-lookup"><span data-stu-id="92d4b-151">How to detect the problem</span></span>

<span data-ttu-id="92d4b-152">Pour les utilisateurs, l’application peut sembler inactive ou se bloquer régulièrement.</span><span class="sxs-lookup"><span data-stu-id="92d4b-152">For users, the application may seem unresponsive or appear to hang periodically.</span></span> <span data-ttu-id="92d4b-153">L’application peut échouer avec les exceptions de délai d’expiration.</span><span class="sxs-lookup"><span data-stu-id="92d4b-153">The application might fail with timeout exceptions.</span></span> <span data-ttu-id="92d4b-154">Ces échecs peuvent également renvoyer des erreurs HTTP 500 (Erreur interne du serveur).</span><span class="sxs-lookup"><span data-stu-id="92d4b-154">These failures could also return HTTP 500 (Internal Server) errors.</span></span> <span data-ttu-id="92d4b-155">Sur le serveur, les demandes client entrantes peuvent être bloquées jusqu’à ce qu’un thread soit disponible, ce qui entraîne des files d’attente de demande excessives, représentées par des erreurs HTTP 503 (service indisponible).</span><span class="sxs-lookup"><span data-stu-id="92d4b-155">On the server, incoming client requests might be blocked until a thread becomes available, resulting in excessive request queue lengths, manifested as HTTP 503 (Service Unavailable) errors.</span></span>

<span data-ttu-id="92d4b-156">Vous pouvez procéder de la manière suivante pour identifier le problème :</span><span class="sxs-lookup"><span data-stu-id="92d4b-156">You can perform the following steps to help identify the problem:</span></span>

1. <span data-ttu-id="92d4b-157">Surveiller le système de production et déterminer si les threads de travail bloqués limitent le débit.</span><span class="sxs-lookup"><span data-stu-id="92d4b-157">Monitor the production system and determine whether blocked worker threads are constraining throughput.</span></span>

2. <span data-ttu-id="92d4b-158">Si les demandes sont bloquées en raison d’un manque de threads, passez en revue l’application pour déterminer quelles opérations peuvent effectuer des E/S de façon synchrone.</span><span class="sxs-lookup"><span data-stu-id="92d4b-158">If requests are being blocked due to lack of threads, review the application to determine which operations may be performing I/O synchronously.</span></span>

3. <span data-ttu-id="92d4b-159">Effectuez un test de charge contrôlé de chaque opération qui effectue des E/S synchrones, pour savoir si ces opérations affectent les performances du système.</span><span class="sxs-lookup"><span data-stu-id="92d4b-159">Perform controlled load testing of each operation that is performing synchronous I/O, to find out whether those operations are affecting system performance.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="92d4b-160">Exemple de diagnostic</span><span class="sxs-lookup"><span data-stu-id="92d4b-160">Example diagnosis</span></span>

<span data-ttu-id="92d4b-161">Les sections suivantes appliquent ces étapes à l’exemple d’application décrit précédemment.</span><span class="sxs-lookup"><span data-stu-id="92d4b-161">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="monitor-web-server-performance"></a><span data-ttu-id="92d4b-162">Surveiller les performances du serveur web</span><span class="sxs-lookup"><span data-stu-id="92d4b-162">Monitor web server performance</span></span>

<span data-ttu-id="92d4b-163">Pour les rôles web et les applications web Azure, il est intéressant d’analyser les performances du serveur web IIS.</span><span class="sxs-lookup"><span data-stu-id="92d4b-163">For Azure web applications and web roles, it's worth monitoring the performance of the IIS web server.</span></span> <span data-ttu-id="92d4b-164">Faites particulièrement attention à la longueur de file d’attente de la demande pour établir si les demandes sont bloquées en attente des threads disponibles pendant les périodes de forte activité.</span><span class="sxs-lookup"><span data-stu-id="92d4b-164">In particular, pay attention to the request queue length to establish whether requests are being blocked waiting for available threads during periods of high activity.</span></span> <span data-ttu-id="92d4b-165">Vous pouvez collecter ces informations en activant les diagnostics Azure.</span><span class="sxs-lookup"><span data-stu-id="92d4b-165">You can gather this information by enabling Azure diagnostics.</span></span> <span data-ttu-id="92d4b-166">Pour plus d'informations, consultez les pages suivantes :</span><span class="sxs-lookup"><span data-stu-id="92d4b-166">For more information, see:</span></span>

- <span data-ttu-id="92d4b-167">[Surveiller les applications dans Azure App Service][web-sites-monitor]</span><span class="sxs-lookup"><span data-stu-id="92d4b-167">[Monitor Apps in Azure App Service][web-sites-monitor]</span></span>
- <span data-ttu-id="92d4b-168">[Créer et utiliser des compteurs de performances dans une application Azure][performance-counters]</span><span class="sxs-lookup"><span data-stu-id="92d4b-168">[Create and use performance counters in an Azure application][performance-counters]</span></span>

<span data-ttu-id="92d4b-169">Instrumentez l’application pour voir comment les demandes sont traitées une fois qu’ils ont été acceptées.</span><span class="sxs-lookup"><span data-stu-id="92d4b-169">Instrument the application to see how requests are handled once they have been accepted.</span></span> <span data-ttu-id="92d4b-170">Suivre le flux d’une demande peut aider à identifier si elle effectue des appels à exécution lente et si cela bloque le thread actuel.</span><span class="sxs-lookup"><span data-stu-id="92d4b-170">Tracing the flow of a request can help to identify whether it is performing slow-running calls and blocking the current thread.</span></span> <span data-ttu-id="92d4b-171">Le profilage du thread permet également de mettre en surbrillance les demandes qui sont bloquées.</span><span class="sxs-lookup"><span data-stu-id="92d4b-171">Thread profiling can also highlight requests that are being blocked.</span></span>

### <a name="load-test-the-application"></a><span data-ttu-id="92d4b-172">Effectuer un test de charge de l’application</span><span class="sxs-lookup"><span data-stu-id="92d4b-172">Load test the application</span></span>

<span data-ttu-id="92d4b-173">Le graphique suivant montre les performances de la méthode synchrone `GetUserProfile` présentée précédemment, sous différentes charges d’utilisateurs simultanés (jusqu’à 4 000).</span><span class="sxs-lookup"><span data-stu-id="92d4b-173">The following graph shows the performance of the synchronous `GetUserProfile` method shown earlier, under varying loads of up to 4000 concurrent users.</span></span> <span data-ttu-id="92d4b-174">L’application est une application ASP.NET en cours d’exécution dans un rôle web de service cloud Azure.</span><span class="sxs-lookup"><span data-stu-id="92d4b-174">The application is an ASP.NET application running in an Azure Cloud Service web role.</span></span>

![Graphique de performances pour l’exemple d’application effectuant des opérations d’E/S synchrones][sync-performance]

<span data-ttu-id="92d4b-176">L’opération synchrone est codée en dur pour la mise en veille pendant 2 secondes, pour simuler des E/S synchrones, par conséquent, le temps de réponse minimal est légèrement supérieur à 2 secondes.</span><span class="sxs-lookup"><span data-stu-id="92d4b-176">The synchronous operation is hard-coded to sleep for 2 seconds, to simulate synchronous I/O, so the minimum response time is slightly over 2 seconds.</span></span> <span data-ttu-id="92d4b-177">Lorsque la charge atteint environ 2 500 utilisateurs simultanés, le temps de réponse moyen commence à stagner, bien que le volume de demandes par seconde continue d’augmenter.</span><span class="sxs-lookup"><span data-stu-id="92d4b-177">When the load reaches approximately 2500 concurrent users, the average response time reaches a plateau, although the volume of requests per second continues to increase.</span></span> <span data-ttu-id="92d4b-178">Notez que l’échelle pour ces deux mesures est logarithmique.</span><span class="sxs-lookup"><span data-stu-id="92d4b-178">Note that the scale for these two measures is logarithmic.</span></span> <span data-ttu-id="92d4b-179">Le nombre de demandes par seconde double entre ce point et la fin du test.</span><span class="sxs-lookup"><span data-stu-id="92d4b-179">The number of requests per second doubles between this point and the end of the test.</span></span>

<span data-ttu-id="92d4b-180">En isolation, il n’est pas nécessairement clair à partir de ce test de définir si les E/S synchrones sont un problème.</span><span class="sxs-lookup"><span data-stu-id="92d4b-180">In isolation, it's not necessarily clear from this test whether the synchronous I/O is a problem.</span></span> <span data-ttu-id="92d4b-181">Sous une charge plus importante, l’application peut atteindre un point de basculement où le serveur web ne peut plus traiter les demandes en temps voulu, causant l’envoi d’exceptions de délai d’attente aux applications clientes.</span><span class="sxs-lookup"><span data-stu-id="92d4b-181">Under heavier load, the application may reach a tipping point where the web server can no longer process requests in a timely manner, causing client applications to receive time-out exceptions.</span></span>

<span data-ttu-id="92d4b-182">Les demandes entrantes sont mises en file d’attente par le serveur web IIS et transmises à un thread en cours d’exécution dans le pool de thread ASP.NET.</span><span class="sxs-lookup"><span data-stu-id="92d4b-182">Incoming requests are queued by the IIS web server and handed to a thread running in the ASP.NET thread pool.</span></span> <span data-ttu-id="92d4b-183">Étant donné que chaque opération effectue des opérations d’E/S de manière synchrone, le thread est bloqué jusqu’à ce que l’opération se termine.</span><span class="sxs-lookup"><span data-stu-id="92d4b-183">Because each operation performs I/O synchronously, the thread is blocked until the operation completes.</span></span> <span data-ttu-id="92d4b-184">À mesure que la charge de travail augmente, tous les threads ASP.NET dans le pool de threads sont finalement alloués et bloqués.</span><span class="sxs-lookup"><span data-stu-id="92d4b-184">As the workload increases, eventually all of the ASP.NET threads in the thread pool are allocated and blocked.</span></span> <span data-ttu-id="92d4b-185">À ce stade, toute demande entrante supplémentaire doit attendre dans la file d’attente qu’un thread soit disponible.</span><span class="sxs-lookup"><span data-stu-id="92d4b-185">At that point, any further incoming requests must wait in the queue for an available thread.</span></span> <span data-ttu-id="92d4b-186">Lorsque la longueur de file d’attente augmente, les requêtes commencent à expirer.</span><span class="sxs-lookup"><span data-stu-id="92d4b-186">As the queue length grows, requests start to time out.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="92d4b-187">Implémenter la solution et vérifier le résultat</span><span class="sxs-lookup"><span data-stu-id="92d4b-187">Implement the solution and verify the result</span></span>

<span data-ttu-id="92d4b-188">Le graphique suivant affiche les résultats du test de charge de la version asynchrone du code.</span><span class="sxs-lookup"><span data-stu-id="92d4b-188">The next graph shows the results from load testing the asynchronous version of the code.</span></span>

![Graphique de performances pour l’exemple d’application effectuant des opérations d’E/S asynchrones][async-performance]

<span data-ttu-id="92d4b-190">Le débit est beaucoup plus élevé.</span><span class="sxs-lookup"><span data-stu-id="92d4b-190">Throughput is far higher.</span></span> <span data-ttu-id="92d4b-191">Sur la même durée que le test précédent, le système gère correctement un débit presque dix fois supérieur, exprimé en nombre de requêtes par seconde.</span><span class="sxs-lookup"><span data-stu-id="92d4b-191">Over the same duration as the previous test, the system successfully handles a nearly tenfold increase in throughput, as measured in requests per second.</span></span> <span data-ttu-id="92d4b-192">En outre, le temps de réponse moyen est relativement constant et reste environ 25 fois plus court que pour le test précédent.</span><span class="sxs-lookup"><span data-stu-id="92d4b-192">Moreover, the average response time is relatively constant and remains approximately 25 times smaller than the previous test.</span></span>


[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/SynchronousIO


[async-wrappers]: https://blogs.msdn.microsoft.com/pfxteam/2012/03/24/should-i-expose-asynchronous-wrappers-for-synchronous-methods/
[performance-counters]: /azure/cloud-services/cloud-services-dotnet-diagnostics-performance-counters
[web-sites-monitor]: /azure/app-service-web/web-sites-monitor

[sync-performance]: _images/SyncPerformance.jpg
[async-performance]: _images/AsyncPerformance.jpg



