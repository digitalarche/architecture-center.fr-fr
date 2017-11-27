---
title: "Antimodèle Serveur frontal occupé"
description: "L’exécution d’un travail asynchrone sur un grand nombre de threads d’arrière-plan peut priver les tâches de premier plan de ressources."
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: cedb80ddac5ceb1eb901455df3165993fd28a138
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2017
---
# <a name="busy-front-end-antipattern"></a><span data-ttu-id="f45bd-103">Antimodèle Serveur frontal occupé</span><span class="sxs-lookup"><span data-stu-id="f45bd-103">Busy Front End antipattern</span></span>

<span data-ttu-id="f45bd-104">L’exécution d’un travail asynchrone sur un grand nombre de threads d’arrière-plan peut priver les tâches de premier plan simultanées de ressources, donnant lieu à des temps de réponse inacceptables.</span><span class="sxs-lookup"><span data-stu-id="f45bd-104">Performing asynchronous work on a large number of background threads can starve other concurrent foreground tasks of resources, decreasing response times to unacceptable levels.</span></span>

## <a name="problem-description"></a><span data-ttu-id="f45bd-105">Description du problème</span><span class="sxs-lookup"><span data-stu-id="f45bd-105">Problem description</span></span>

<span data-ttu-id="f45bd-106">Les tâches nécessitant de nombreuses ressources peuvent augmenter les temps de réponse pour les requêtes utilisateur et entraîner une latence élevée.</span><span class="sxs-lookup"><span data-stu-id="f45bd-106">Resource-intensive tasks can increase the response times for user requests and cause high latency.</span></span> <span data-ttu-id="f45bd-107">Pour améliorer les temps de réponse, une solution consiste à décharger une tâche nécessitant de nombreuses ressources vers un thread distinct.</span><span class="sxs-lookup"><span data-stu-id="f45bd-107">One way to improve response times is to offload a resource-intensive task to a separate thread.</span></span> <span data-ttu-id="f45bd-108">Cette approche permet à l’application de rester réactive pendant que le traitement est réalisé en arrière-plan.</span><span class="sxs-lookup"><span data-stu-id="f45bd-108">This approach lets the application stay responsive while processing happens in the background.</span></span> <span data-ttu-id="f45bd-109">Cependant, les tâches exécutées sur un thread d’arrière-plan consomment toujours des ressources.</span><span class="sxs-lookup"><span data-stu-id="f45bd-109">However, tasks that run on a background thread still consume resources.</span></span> <span data-ttu-id="f45bd-110">Si ces tâches sont trop nombreuses, elles peuvent priver les threads qui traitent les requêtes de ressources.</span><span class="sxs-lookup"><span data-stu-id="f45bd-110">If there are too many of them, they can starve the threads that are handling requests.</span></span>

> [!NOTE]
> <span data-ttu-id="f45bd-111">Le terme *ressource* peut englober un grand nombre de choses, notamment l’utilisation du processeur, l’occupation de la mémoire et les E/S réseau ou de disque.</span><span class="sxs-lookup"><span data-stu-id="f45bd-111">The term *resource* can encompass many things, such as CPU utilization, memory occupancy, and network or disk I/O.</span></span>

<span data-ttu-id="f45bd-112">Ce problème se produit généralement quand une application est développée en tant que bloc de code monolithique, avec toute la logique métier regroupée dans un niveau unique partagé avec la couche de présentation.</span><span class="sxs-lookup"><span data-stu-id="f45bd-112">This problem typically occurs when an application is developed as monolithic piece of code, with all of the business logic combined into a single tier shared with the presentation layer.</span></span>

<span data-ttu-id="f45bd-113">Voici un exemple faisant appel à ASP.NET qui illustre le problème.</span><span class="sxs-lookup"><span data-stu-id="f45bd-113">Here’s an example using ASP.NET that demonstrates the problem.</span></span> <span data-ttu-id="f45bd-114">Vous trouverez l’exemple complet [ici][code-sample].</span><span class="sxs-lookup"><span data-stu-id="f45bd-114">You can find the complete sample [here][code-sample].</span></span>

```csharp
public class WorkInFrontEndController : ApiController
{
    [HttpPost]
    [Route("api/workinfrontend")]
    public HttpResponseMessage Post()
    {
        new Thread(() =>
        {
            //Simulate processing
            Thread.SpinWait(Int32.MaxValue / 100);
        }).Start();

        return Request.CreateResponse(HttpStatusCode.Accepted);
    }
}

public class UserProfileController : ApiController
{
    [HttpGet]
    [Route("api/userprofile/{id}")]
    public UserProfile Get(int id)
    {
        //Simulate processing
        return new UserProfile() { FirstName = "Alton", LastName = "Hudgens" };
    }
}
```

- <span data-ttu-id="f45bd-115">La méthode `Post` dans le contrôleur `WorkInFrontEnd` implémente une opération HTTP POST.</span><span class="sxs-lookup"><span data-stu-id="f45bd-115">The `Post` method in the `WorkInFrontEnd` controller implements an HTTP POST operation.</span></span> <span data-ttu-id="f45bd-116">Cette opération simule une tâche de longue durée nécessitant une utilisation importante du processeur.</span><span class="sxs-lookup"><span data-stu-id="f45bd-116">This operation simulates a long-running, CPU-intensive task.</span></span> <span data-ttu-id="f45bd-117">Le travail est réalisé sur un thread distinct afin de favoriser l’exécution rapide de l’opération POST.</span><span class="sxs-lookup"><span data-stu-id="f45bd-117">The work is performed on a separate thread, in an attempt to enable the POST operation to complete quickly.</span></span>

- <span data-ttu-id="f45bd-118">La méthode `Get` dans le contrôleur `UserProfile` implémente une opération HTTP GET.</span><span class="sxs-lookup"><span data-stu-id="f45bd-118">The `Get` method in the `UserProfile` controller implements an HTTP GET operation.</span></span> <span data-ttu-id="f45bd-119">Cette méthode nécessite une utilisation beaucoup moins importante du processeur.</span><span class="sxs-lookup"><span data-stu-id="f45bd-119">This method is much less CPU intensive.</span></span>

<span data-ttu-id="f45bd-120">La principale préoccupation concerne les besoins en ressources de la méthode `Post`.</span><span class="sxs-lookup"><span data-stu-id="f45bd-120">The primary concern is the resource requirements of the `Post` method.</span></span> <span data-ttu-id="f45bd-121">Bien que cette méthode place le travail sur un thread d’arrière-plan, le travail est toujours susceptible de consommer un nombre considérable de ressources processeur.</span><span class="sxs-lookup"><span data-stu-id="f45bd-121">Although it puts the work onto a background thread, the work can still consume considerable CPU resources.</span></span> <span data-ttu-id="f45bd-122">Ces ressources sont partagées avec les autres opérations exécutées par les autres utilisateurs simultanés.</span><span class="sxs-lookup"><span data-stu-id="f45bd-122">These resources are shared with other operations being performed by other concurrent users.</span></span> <span data-ttu-id="f45bd-123">Si un nombre modéré d’utilisateurs envoient cette requête en même temps, les performances globales risquent d’être dégradées, entraînant un ralentissement de toutes les opérations.</span><span class="sxs-lookup"><span data-stu-id="f45bd-123">If a moderate number of users send this request at the same time, overall performance is likely to suffer, slowing down all operations.</span></span> <span data-ttu-id="f45bd-124">Les utilisateurs peuvent par exemple être confrontés à une latence importante pour la méthode `Get`.</span><span class="sxs-lookup"><span data-stu-id="f45bd-124">Users might experience significant latency in the `Get` method, for example.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="f45bd-125">Comment corriger le problème</span><span class="sxs-lookup"><span data-stu-id="f45bd-125">How to fix the problem</span></span>

<span data-ttu-id="f45bd-126">Déplacez les processus qui consomment de nombreuses ressources sur un serveur principal distinct.</span><span class="sxs-lookup"><span data-stu-id="f45bd-126">Move processes that consume significant resources to a separate back end.</span></span> 

<span data-ttu-id="f45bd-127">Avec cette approche, le serveur frontal place les tâches nécessitant de nombreuses ressources dans une file d’attente.</span><span class="sxs-lookup"><span data-stu-id="f45bd-127">With this approach, the front end puts resource-intensive tasks onto a message queue.</span></span> <span data-ttu-id="f45bd-128">Le serveur principal récupère les tâches à des fins de traitement asynchrone.</span><span class="sxs-lookup"><span data-stu-id="f45bd-128">The back end picks up the tasks for asynchronous processing.</span></span> <span data-ttu-id="f45bd-129">La file d’attente assure également le nivellement de la charge, mettant les requêtes en tampon pour le serveur principal.</span><span class="sxs-lookup"><span data-stu-id="f45bd-129">The queue also acts as a load leveler, buffering requests for the back end.</span></span> <span data-ttu-id="f45bd-130">Si la file d’attente devient trop longue, vous pouvez configurer la mise à l’échelle automatique pour augmenter la taille des instances du serveur principal.</span><span class="sxs-lookup"><span data-stu-id="f45bd-130">If the queue length becomes too long, you can configure autoscaling to scale out the back end.</span></span>

<span data-ttu-id="f45bd-131">Voici une version révisée du code précédent.</span><span class="sxs-lookup"><span data-stu-id="f45bd-131">Here is a revised version of the previous code.</span></span> <span data-ttu-id="f45bd-132">Dans cette version, la méthode `Post` place un message dans une file d’attente Service Bus.</span><span class="sxs-lookup"><span data-stu-id="f45bd-132">In this version, the `Post` method puts a message on a Service Bus queue.</span></span> 

```csharp
public class WorkInBackgroundController : ApiController
{
    private static readonly QueueClient QueueClient;
    private static readonly string QueueName;
    private static readonly ServiceBusQueueHandler ServiceBusQueueHandler;

    public WorkInBackgroundController()
    {
        var serviceBusConnectionString = ...;
        QueueName = ...;
        ServiceBusQueueHandler = new ServiceBusQueueHandler(serviceBusConnectionString);
        QueueClient = ServiceBusQueueHandler.GetQueueClientAsync(QueueName).Result;
    }

    [HttpPost]
    [Route("api/workinbackground")]
    public async Task<long> Post()
    {
        return await ServiceBusQueuehandler.AddWorkLoadToQueueAsync(QueueClient, QueueName, 0);
    }
}
```

<span data-ttu-id="f45bd-133">Le serveur principal extrait les messages de la file d’attente Service Bus et effectue le traitement.</span><span class="sxs-lookup"><span data-stu-id="f45bd-133">The back end pulls messages from the Service Bus queue and does the processing.</span></span>

```csharp
public async Task RunAsync(CancellationToken cancellationToken)
{
    this._queueClient.OnMessageAsync(
        // This lambda is invoked for each message received.
        async (receivedMessage) =>
        {
            try
            {
                // Simulate processing of message
                Thread.SpinWait(Int32.Maxvalue / 1000);

                await receivedMessage.CompleteAsync();
            }
            catch
            {
                receivedMessage.Abandon();
            }
        });
}
```

## <a name="considerations"></a><span data-ttu-id="f45bd-134">Considérations</span><span class="sxs-lookup"><span data-stu-id="f45bd-134">Considerations</span></span>

- <span data-ttu-id="f45bd-135">Cette approche rend l’application un peu plus complexe.</span><span class="sxs-lookup"><span data-stu-id="f45bd-135">This approach adds some additional complexity to the application.</span></span> <span data-ttu-id="f45bd-136">Vous devez gérer la mise en file d’attente et le retrait de la file d’attente de manière sûre pour éviter la perte de requêtes en cas de défaillance.</span><span class="sxs-lookup"><span data-stu-id="f45bd-136">You must handle queuing and dequeuing safely to avoid losing requests in the event of a failure.</span></span>
- <span data-ttu-id="f45bd-137">L’application devient dépendante d’un service supplémentaire pour la file d’attente de messages.</span><span class="sxs-lookup"><span data-stu-id="f45bd-137">The application takes a dependency on an additional service for the message queue.</span></span>
- <span data-ttu-id="f45bd-138">L’environnement de traitement doit être suffisamment évolutif pour gérer la charge de travail attendue et atteindre les objectifs de débit requis.</span><span class="sxs-lookup"><span data-stu-id="f45bd-138">The processing environment must be sufficiently scalable to handle the expected workload and meet the required throughput targets.</span></span>
- <span data-ttu-id="f45bd-139">Même si cette approche devrait normalement améliorer la réactivité globale, l’exécution des tâches déplacées sur le serveur principal peut prendre plus de temps.</span><span class="sxs-lookup"><span data-stu-id="f45bd-139">While this approach should improve overall responsiveness, the tasks that are moved to the back end may take longer to complete.</span></span> 

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="f45bd-140">Comment détecter le problème</span><span class="sxs-lookup"><span data-stu-id="f45bd-140">How to detect the problem</span></span>

<span data-ttu-id="f45bd-141">Les symptômes d’un serveur frontal occupé incluent une latence élevée pendant l’exécution des tâches nécessitant de nombreuses ressources.</span><span class="sxs-lookup"><span data-stu-id="f45bd-141">Symptoms of a busy front end include high latency when resource-intensive tasks are being performed.</span></span> <span data-ttu-id="f45bd-142">Les utilisateurs finals sont susceptibles de signaler des temps de réponse plus longs que d’habitude ou des échecs causés par l’expiration du délai d’attente des services. Ces échecs peuvent également renvoyer des erreurs HTTP 500 (serveur interne) ou HTTP 503 (service indisponible).</span><span class="sxs-lookup"><span data-stu-id="f45bd-142">End users are likely to report extended response times or failures caused by services timing out. These failures could also return HTTP 500 (Internal Server) errors or HTTP 503 (Service Unavailable) errors.</span></span> <span data-ttu-id="f45bd-143">Examinez les journaux d’événements du serveur web, qui contiennent probablement des informations plus détaillées sur les causes et les circonstances de ces erreurs.</span><span class="sxs-lookup"><span data-stu-id="f45bd-143">Examine the event logs for the web server, which are likely to contain more detailed information about the causes and circumstances of the errors.</span></span>

<span data-ttu-id="f45bd-144">Vous pouvez procéder de la manière suivante pour identifier ce problème :</span><span class="sxs-lookup"><span data-stu-id="f45bd-144">You can perform the following steps to help identify this problem:</span></span>

1. <span data-ttu-id="f45bd-145">Analysez le processus du système de production afin d’identifier les points où les temps de réponse augmentent.</span><span class="sxs-lookup"><span data-stu-id="f45bd-145">Perform process monitoring of the production system, to identify points when response times slow down.</span></span>
2. <span data-ttu-id="f45bd-146">Examinez les données de télémétrie enregistrées à ces points pour déterminer les opérations exécutées en même temps et les ressources utilisées.</span><span class="sxs-lookup"><span data-stu-id="f45bd-146">Examine the telemetry data captured at these points to determine the mix of operations being performed and the resources being used.</span></span> 
3. <span data-ttu-id="f45bd-147">Recherchez les corrélations entre les temps de réponse médiocres et les volumes et combinaisons d’opérations observés à ces moments-là.</span><span class="sxs-lookup"><span data-stu-id="f45bd-147">Find any correlations between poor response times and the volumes and combinations of operations that were happening at those times.</span></span>
4. <span data-ttu-id="f45bd-148">Procédez à un test de charge de chaque opération suspectée pour identifier les opérations qui consomment de nombreuses ressources et nuisent ainsi aux autres opérations.</span><span class="sxs-lookup"><span data-stu-id="f45bd-148">Load test each suspected operation to identify which operations are consuming resources and starving other operations.</span></span> 
5. <span data-ttu-id="f45bd-149">Révisez le code source de ces opérations afin de déterminer les raisons potentielles de cette consommation excessive de ressources.</span><span class="sxs-lookup"><span data-stu-id="f45bd-149">Review the source code for those operations to determine why they might cause excessive resource consumption.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="f45bd-150">Exemple de diagnostic</span><span class="sxs-lookup"><span data-stu-id="f45bd-150">Example diagnosis</span></span> 

<span data-ttu-id="f45bd-151">Les sections suivantes appliquent ces étapes à l’exemple d’application décrit précédemment.</span><span class="sxs-lookup"><span data-stu-id="f45bd-151">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="identify-points-of-slowdown"></a><span data-ttu-id="f45bd-152">Identifier les points de ralentissement</span><span class="sxs-lookup"><span data-stu-id="f45bd-152">Identify points of slowdown</span></span>

<span data-ttu-id="f45bd-153">Instrumentez chaque méthode pour suivre la durée des différentes requêtes et les ressources consommées par celles-ci.</span><span class="sxs-lookup"><span data-stu-id="f45bd-153">Instrument each method to track the duration and resources consumed by each request.</span></span> <span data-ttu-id="f45bd-154">Ensuite, analysez l’application en production.</span><span class="sxs-lookup"><span data-stu-id="f45bd-154">Then monitor the application in production.</span></span> <span data-ttu-id="f45bd-155">Vous bénéficierez ainsi d’une vue d’ensemble de la manière dont les requêtes s’opposent.</span><span class="sxs-lookup"><span data-stu-id="f45bd-155">This can provide an overall view of how requests compete with each other.</span></span> <span data-ttu-id="f45bd-156">Pendant les périodes de forte activité, les requêtes gourmandes en ressources et dont l’exécution est lente sont susceptibles d’avoir un impact négatif sur les autres opérations, un comportement qu’il est possible de constater en analysant le système et en observant la baisse des performances.</span><span class="sxs-lookup"><span data-stu-id="f45bd-156">During periods of stress, slow-running resource-hungry requests will likely impact other operations, and this behavior can be observed by monitoring the system and noting the drop off in performance.</span></span>

<span data-ttu-id="f45bd-157">L’image suivante illustre un tableau de bord d’analyse</span><span class="sxs-lookup"><span data-stu-id="f45bd-157">The following image shows a monitoring dashboard.</span></span> <span data-ttu-id="f45bd-158">(nous avons utilisé [AppDynamics] pour nos tests). Au départ, le système présente une charge faible.</span><span class="sxs-lookup"><span data-stu-id="f45bd-158">(We used [AppDynamics] for our tests.) Initially, the system has light load.</span></span> <span data-ttu-id="f45bd-159">Ensuite, les utilisateurs commencent à envoyer des requêtes pour la méthode GET au contrôleur `UserProfile`.</span><span class="sxs-lookup"><span data-stu-id="f45bd-159">Then users start requesting the `UserProfile` GET method.</span></span> <span data-ttu-id="f45bd-160">Les performances sont relativement bonnes jusqu'à ce que d’autres utilisateurs commencent à émettre des requêtes pour la méthode POST à destination du contrôleur `WorkInFrontEnd`.</span><span class="sxs-lookup"><span data-stu-id="f45bd-160">The performance is reasonably good until other users start issuing requests to the `WorkInFrontEnd` POST method.</span></span> <span data-ttu-id="f45bd-161">À ce moment-là, les temps de réponse augmentent considérablement (première flèche).</span><span class="sxs-lookup"><span data-stu-id="f45bd-161">At that point, response times increase dramatically (first arrow).</span></span> <span data-ttu-id="f45bd-162">Les temps de réponse s’améliorent uniquement après la diminution du volume de requêtes envoyées au contrôleur `WorkInFrontEnd` (seconde flèche).</span><span class="sxs-lookup"><span data-stu-id="f45bd-162">Response times only improve after the volume of requests to the `WorkInFrontEnd` controller diminishes (second arrow).</span></span>

![Volet des transactions commerciales d’AppDynamics illustrant l’effet de l’utilisation du contrôleur WorkInFrontEnd sur les temps de réponse de l’ensemble des requêtes][AppDynamics-Transactions-Front-End-Requests]

### <a name="examine-telemetry-data-and-find-correlations"></a><span data-ttu-id="f45bd-164">Examiner les données de télémétrie et rechercher les corrélations</span><span class="sxs-lookup"><span data-stu-id="f45bd-164">Examine telemetry data and find correlations</span></span>

<span data-ttu-id="f45bd-165">L’image suivante illustre quelques-unes des mesures collectées pour surveiller l’utilisation des ressources pendant le même intervalle de temps.</span><span class="sxs-lookup"><span data-stu-id="f45bd-165">The next image shows some of the metrics gathered to monitor resource utilization during the same interval.</span></span> <span data-ttu-id="f45bd-166">Au départ, peu d’utilisateurs accèdent au système.</span><span class="sxs-lookup"><span data-stu-id="f45bd-166">At first, few users are accessing the system.</span></span> <span data-ttu-id="f45bd-167">À mesure que d’autres utilisateurs se connectent, l’utilisation du processeur devient très élevée (100 %).</span><span class="sxs-lookup"><span data-stu-id="f45bd-167">As more users connect, CPU utilization becomes very high (100%).</span></span> <span data-ttu-id="f45bd-168">Par ailleurs, vous remarquerez qu’initialement, le taux d’E/S réseau augmente parallèlement à l’utilisation du processeur.</span><span class="sxs-lookup"><span data-stu-id="f45bd-168">Also notice that the network I/O rate initially goes up as CPU usage rises.</span></span> <span data-ttu-id="f45bd-169">Cependant, une fois que l’utilisation du processeur est au plus haut, les E/S réseau diminuent.</span><span class="sxs-lookup"><span data-stu-id="f45bd-169">But once CPU usage peaks, network I/O actually goes down.</span></span> <span data-ttu-id="f45bd-170">Cela s’explique par le fait que le système peut seulement traiter un nombre relativement restreint de requêtes une fois que le processeur a atteint sa capacité maximale.</span><span class="sxs-lookup"><span data-stu-id="f45bd-170">That's because the system can only handle a relatively small number of requests once the CPU is at capacity.</span></span> <span data-ttu-id="f45bd-171">À mesure que les utilisateurs se déconnectent, la charge du processeur diminue.</span><span class="sxs-lookup"><span data-stu-id="f45bd-171">As users disconnect, the CPU load tails off.</span></span>

![Mesures d’AppDynamics reflétant l’utilisation du processeur et du réseau][AppDynamics-Metrics-Front-End-Requests]

<span data-ttu-id="f45bd-173">À ce stade, la méthode `Post` dans le contrôleur `WorkInFrontEnd` semble être un candidat de choix pour un examen plus approfondi.</span><span class="sxs-lookup"><span data-stu-id="f45bd-173">At this point, it appears the `Post` method in the `WorkInFrontEnd` controller is a prime candidate for closer examination.</span></span> <span data-ttu-id="f45bd-174">Un travail supplémentaire dans un environnement contrôlé est nécessaire pour confirmer cette hypothèse.</span><span class="sxs-lookup"><span data-stu-id="f45bd-174">Further work in a controlled environment is needed to confirm the hypothesis.</span></span>

### <a name="perform-load-testing"></a><span data-ttu-id="f45bd-175">Effectuer des tests de charge</span><span class="sxs-lookup"><span data-stu-id="f45bd-175">Perform load testing</span></span> 

<span data-ttu-id="f45bd-176">L’étape suivante consiste à effectuer des tests dans un environnement contrôlé.</span><span class="sxs-lookup"><span data-stu-id="f45bd-176">The next step is to perform tests in a controlled environment.</span></span> <span data-ttu-id="f45bd-177">Par exemple, vous pouvez exécuter une série de tests de charge qui incluent puis omettent successivement chaque requête pour observer les effets.</span><span class="sxs-lookup"><span data-stu-id="f45bd-177">For example, run a series of load tests that include and then omit each request in turn to see the effects.</span></span>

<span data-ttu-id="f45bd-178">Le graphique ci-dessous montre les résultats d’un test de charge effectué sur un déploiement identique du service cloud utilisé dans les tests précédents.</span><span class="sxs-lookup"><span data-stu-id="f45bd-178">The graph below shows the results of a load test performed against an identical deployment of the cloud service used in the previous tests.</span></span> <span data-ttu-id="f45bd-179">Pour ce test, une charge constante de 500 utilisateurs exécutant l’opération `Get` dans le contrôleur `UserProfile` et une charge progressive d’utilisateurs exécutant l’opération `Post` dans le contrôleur `WorkInFrontEnd` ont été utilisées.</span><span class="sxs-lookup"><span data-stu-id="f45bd-179">The test used a constant load of 500 users performing the `Get` operation in the `UserProfile` controller, along with a step load of users performing the `Post` operation in the `WorkInFrontEnd` controller.</span></span> 

![Résultats initiaux du test de charge pour le contrôleur WorkInFrontEnd][Initial-Load-Test-Results-Front-End]

<span data-ttu-id="f45bd-181">Au départ, la charge progressive est de 0, ce qui signifie que les seuls utilisateurs actifs envoient des requêtes `UserProfile`.</span><span class="sxs-lookup"><span data-stu-id="f45bd-181">Initially, the step load is 0, so the only active users are performing the `UserProfile` requests.</span></span> <span data-ttu-id="f45bd-182">Le système est capable de répondre à environ 500 requêtes par seconde.</span><span class="sxs-lookup"><span data-stu-id="f45bd-182">The system is able to respond to approximately 500 requests per second.</span></span> <span data-ttu-id="f45bd-183">Après 60 secondes, une charge de 100 utilisateurs supplémentaires, qui commencent à envoyer des requêtes POST au contrôleur `WorkInFrontEnd`, est ajoutée.</span><span class="sxs-lookup"><span data-stu-id="f45bd-183">After 60 seconds, a load of 100 additional users starts sending POST requests to the `WorkInFrontEnd` controller.</span></span> <span data-ttu-id="f45bd-184">Presque immédiatement, la charge de travail envoyée au contrôleur `UserProfile` descend à environ 150 requêtes par seconde.</span><span class="sxs-lookup"><span data-stu-id="f45bd-184">Almost immediately, the workload sent to the `UserProfile` controller drops to about 150 requests per second.</span></span> <span data-ttu-id="f45bd-185">Cela est dû au mode de fonctionnement de l’exécuteur de tests de charge.</span><span class="sxs-lookup"><span data-stu-id="f45bd-185">This is due to the way the load-test runner functions.</span></span> <span data-ttu-id="f45bd-186">Celui-ci attend une réponse avant d’envoyer la requête suivante. Par conséquent, plus la réponse met de temps à arriver, plus le taux de requêtes est faible.</span><span class="sxs-lookup"><span data-stu-id="f45bd-186">It waits for a response before sending the next request, so the longer it takes to receive a response, the lower the request rate.</span></span>

<span data-ttu-id="f45bd-187">À mesure que des utilisateurs supplémentaires envoient des requêtes POST au contrôleur `WorkInFrontEnd`, le taux de requêtes du contrôleur `UserProfile` continue à diminuer.</span><span class="sxs-lookup"><span data-stu-id="f45bd-187">As more users send POST requests to the `WorkInFrontEnd` controller, the response rate of the `UserProfile` controller continues to drop.</span></span> <span data-ttu-id="f45bd-188">Cependant, vous remarquerez que le volume de requêtes traitées par le contrôleur `WorkInFrontEnd` reste relativement constant.</span><span class="sxs-lookup"><span data-stu-id="f45bd-188">But note that the volume of requests handled by the `WorkInFrontEnd`controller remains relatively constant.</span></span> <span data-ttu-id="f45bd-189">La saturation du système devient apparente à mesure que le taux global des deux requêtes tend vers une limite constante mais faible.</span><span class="sxs-lookup"><span data-stu-id="f45bd-189">The saturation of the system becomes apparent as the overall rate of both requests tends towards a steady but low limit.</span></span>


### <a name="review-the-source-code"></a><span data-ttu-id="f45bd-190">Réviser le code source</span><span class="sxs-lookup"><span data-stu-id="f45bd-190">Review the source code</span></span>

<span data-ttu-id="f45bd-191">La dernière étape consiste à examiner le code source.</span><span class="sxs-lookup"><span data-stu-id="f45bd-191">The final step is to look at the source code.</span></span> <span data-ttu-id="f45bd-192">L’équipe de développement était consciente que la méthode `Post` pouvait prendre beaucoup de temps, c’est pourquoi l’implémentation d’origine utilisait un thread distinct.</span><span class="sxs-lookup"><span data-stu-id="f45bd-192">The development team was aware that the `Post` method could take a considerable amount of time, which is why the original implementation used a separate thread.</span></span> <span data-ttu-id="f45bd-193">Cela a permis de résoudre le problème immédiat, car la méthode `Post` ne se bloquait pas en attendant qu’une tâche de longue durée se termine.</span><span class="sxs-lookup"><span data-stu-id="f45bd-193">That solved the immediate problem, because the `Post` method did not block waiting for a long-running task to complete.</span></span>

<span data-ttu-id="f45bd-194">Cependant, le travail exécuté par cette méthode consomme toujours des ressources processeur, mémoire et autres.</span><span class="sxs-lookup"><span data-stu-id="f45bd-194">However, the work performed by this method still consumes CPU, memory, and other resources.</span></span> <span data-ttu-id="f45bd-195">Permettre l’exécution asynchrone de ce processus est en fait susceptible de dégrader les performances, car les utilisateurs peuvent déclencher un grand nombre de ces opérations simultanément, de manière non contrôlée.</span><span class="sxs-lookup"><span data-stu-id="f45bd-195">Enabling this process to run asynchronously might actually damage performance, as users can trigger a large number of these operations simultaneously, in an uncontrolled manner.</span></span> <span data-ttu-id="f45bd-196">Le nombre de threads qu’un serveur peut exécuter est limité.</span><span class="sxs-lookup"><span data-stu-id="f45bd-196">There is a limit to the number of threads that a server can run.</span></span> <span data-ttu-id="f45bd-197">Au-delà de cette limite, l’application risque d’obtenir une exception en essayant de démarrer un nouveau thread.</span><span class="sxs-lookup"><span data-stu-id="f45bd-197">Past this limit, the application is likely to get an exception when it tries to start a new thread.</span></span>

> [!NOTE]
> <span data-ttu-id="f45bd-198">Vous ne devez pas pour autant éviter les opérations asynchrones.</span><span class="sxs-lookup"><span data-stu-id="f45bd-198">This doesn't mean you should avoid asynchronous operations.</span></span> <span data-ttu-id="f45bd-199">L’exécution d’une opération await asynchrone sur un appel réseau est une pratique recommandée</span><span class="sxs-lookup"><span data-stu-id="f45bd-199">Performing an asynchronous await on a network call is a recommended practice.</span></span> <span data-ttu-id="f45bd-200">(voir l’antimodèle [E/S synchrones][sync-io]). Le problème ici est que le travail nécessitant une utilisation importante du processeur a été engendré sur un autre thread.</span><span class="sxs-lookup"><span data-stu-id="f45bd-200">(See the [Synchronous I/O][sync-io] antipattern.) The problem here is that CPU-intensive work was spawned on another thread.</span></span> 

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="f45bd-201">Implémenter la solution et vérifier le résultat</span><span class="sxs-lookup"><span data-stu-id="f45bd-201">Implement the solution and verify the result</span></span>

<span data-ttu-id="f45bd-202">L’image suivante montre l’analyse des performances après l’implémentation de la solution.</span><span class="sxs-lookup"><span data-stu-id="f45bd-202">The following image shows performance monitoring after the solution was implemented.</span></span> <span data-ttu-id="f45bd-203">La charge était similaire à celle illustrée précédemment, mais les temps de réponse pour le contrôleur `UserProfile` sont maintenant beaucoup plus courts.</span><span class="sxs-lookup"><span data-stu-id="f45bd-203">The load was similar to that shown earlier, but the response times for the `UserProfile` controller are now much faster.</span></span> <span data-ttu-id="f45bd-204">Sur la même durée, le volume de requêtes est passé de 2 759 à 23 565.</span><span class="sxs-lookup"><span data-stu-id="f45bd-204">The volume of requests increased over the same duration, from 2,759 to 23,565.</span></span> 

![Volet des transactions commerciales d’AppDynamics illustrant l’effet de l’utilisation du contrôleur WorkInBackground sur les temps de réponse de l’ensemble des requêtes][AppDynamics-Transactions-Background-Requests]

<span data-ttu-id="f45bd-206">Vous remarquerez que le contrôleur `WorkInBackground` a également traité un volume beaucoup plus important de requêtes.</span><span class="sxs-lookup"><span data-stu-id="f45bd-206">Note that the `WorkInBackground` controller also handled a much larger volume of requests.</span></span> <span data-ttu-id="f45bd-207">Cependant, il n’est pas possible de faire de comparaison directe dans ce cas, car le travail exécuté dans ce contrôleur est très différent de celui du code d’origine.</span><span class="sxs-lookup"><span data-stu-id="f45bd-207">However, you can't make a direct comparison in this case, because the work being performed in this controller is very different from the original code.</span></span> <span data-ttu-id="f45bd-208">En effet, au lieu d’exécuter un long calcul, la nouvelle version met simplement une requête en file d’attente.</span><span class="sxs-lookup"><span data-stu-id="f45bd-208">The new version simply queues a request, rather than performing a time consuming calculation.</span></span> <span data-ttu-id="f45bd-209">Le point à retenir est que cette méthode ne ralentit plus l’ensemble du système sous charge.</span><span class="sxs-lookup"><span data-stu-id="f45bd-209">The main point is that this method no longer drags down the entire system under load.</span></span>

<span data-ttu-id="f45bd-210">L’utilisation du processeur et l’utilisation du réseau témoignent également de l’amélioration des performances.</span><span class="sxs-lookup"><span data-stu-id="f45bd-210">CPU and network utilization also show the improved performance.</span></span> <span data-ttu-id="f45bd-211">L’utilisation du processeur n’a jamais atteint 100 %, et le volume de requêtes réseau traitées a été bien plus important que précédemment, ne diminuant qu’au moment où la charge de travail a baissé.</span><span class="sxs-lookup"><span data-stu-id="f45bd-211">The CPU utilization never reached 100%, and the volume of handled network requests was far greater than earlier, and did not tail off until the workload dropped.</span></span>

![Mesures d’AppDynamics reflétant l’utilisation du processeur et du réseau pour le contrôleur WorkInBackground][AppDynamics-Metrics-Background-Requests]

<span data-ttu-id="f45bd-213">Le graphique suivant présente les résultats d’un test de charge.</span><span class="sxs-lookup"><span data-stu-id="f45bd-213">The following graph shows the results of a load test.</span></span> <span data-ttu-id="f45bd-214">Le volume total de requêtes traitées est en nette hausse par rapport aux tests précédents.</span><span class="sxs-lookup"><span data-stu-id="f45bd-214">The overall volume of requests serviced is greatly improved compared to the the earlier tests.</span></span>

![Résultats du test de charge pour le contrôleur BackgroundImageProcessing][Load-Test-Results-Background]

## <a name="related-guidance"></a><span data-ttu-id="f45bd-216">Aide connexe</span><span class="sxs-lookup"><span data-stu-id="f45bd-216">Related guidance</span></span>

- <span data-ttu-id="f45bd-217">[Meilleures pratiques en matière de mise à l’échelle automatique][autoscaling]</span><span class="sxs-lookup"><span data-stu-id="f45bd-217">[Autoscaling best practices][autoscaling]</span></span>
- <span data-ttu-id="f45bd-218">[Meilleures pratiques en matière de tâches en arrière-plan][background-jobs]</span><span class="sxs-lookup"><span data-stu-id="f45bd-218">[Background jobs best practices][background-jobs]</span></span>
- <span data-ttu-id="f45bd-219">[Modèle de nivellement de charge basé sur une file d’attente][load-leveling]</span><span class="sxs-lookup"><span data-stu-id="f45bd-219">[Queue-Based Load Leveling pattern][load-leveling]</span></span>
- <span data-ttu-id="f45bd-220">[Style d’architecture Web-File d’attente-Agent de travail][web-queue-worker]</span><span class="sxs-lookup"><span data-stu-id="f45bd-220">[Web Queue Worker architecture style][web-queue-worker]</span></span>

[AppDyanamics]: https://www.appdynamics.com/
[autoscaling]: /azure/architecture/best-practices/auto-scaling
[background-jobs]: /azure/architecture/best-practices/background-jobs
[code-sample]: https://github.com/mspnp/performance-optimization/tree/master/BusyFrontEnd
[fullDemonstrationOfSolution]: https://github.com/mspnp/performance-optimization/tree/master/BusyFrontEnd
[load-leveling]: /azure/architecture/patterns/queue-based-load-leveling
[sync-io]: ../synchronous-io/index.md
[web-queue-worker]: /azure/architecture/guide/architecture-styles/web-queue-worker

[WebJobs]: http://www.hanselman.com/blog/IntroducingWindowsAzureWebJobs.aspx
[ComputePartitioning]: https://msdn.microsoft.com/library/dn589773.aspx
[ServiceBusQueues]: https://msdn.microsoft.com/library/azure/hh367516.aspx
[AppDynamics-Transactions-Front-End-Requests]: ./_images/AppDynamicsPerformanceStats.jpg
[AppDynamics-Metrics-Front-End-Requests]: ./_images/AppDynamicsFrontEndMetrics.jpg
[Initial-Load-Test-Results-Front-End]: ./_images/InitialLoadTestResultsFrontEnd.jpg
[AppDynamics-Transactions-Background-Requests]: ./_images/AppDynamicsBackgroundPerformanceStats.jpg
[AppDynamics-Metrics-Background-Requests]: ./_images/AppDynamicsBackgroundMetrics.jpg
[Load-Test-Results-Background]: ./_images/LoadTestResultsBackground.jpg


