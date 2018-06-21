---
title: Antimodèle Serveur frontal occupé
description: L’exécution d’un travail asynchrone sur un grand nombre de threads d’arrière-plan peut priver les tâches de premier plan de ressources.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: cedb80ddac5ceb1eb901455df3165993fd28a138
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2017
ms.locfileid: "24538727"
---
# <a name="busy-front-end-antipattern"></a>Antimodèle Serveur frontal occupé

L’exécution d’un travail asynchrone sur un grand nombre de threads d’arrière-plan peut priver les tâches de premier plan simultanées de ressources, donnant lieu à des temps de réponse inacceptables.

## <a name="problem-description"></a>Description du problème

Les tâches nécessitant de nombreuses ressources peuvent augmenter les temps de réponse pour les requêtes utilisateur et entraîner une latence élevée. Pour améliorer les temps de réponse, une solution consiste à décharger une tâche nécessitant de nombreuses ressources vers un thread distinct. Cette approche permet à l’application de rester réactive pendant que le traitement est réalisé en arrière-plan. Cependant, les tâches exécutées sur un thread d’arrière-plan consomment toujours des ressources. Si ces tâches sont trop nombreuses, elles peuvent priver les threads qui traitent les requêtes de ressources.

> [!NOTE]
> Le terme *ressource* peut englober un grand nombre de choses, notamment l’utilisation du processeur, l’occupation de la mémoire et les E/S réseau ou de disque.

Ce problème se produit généralement quand une application est développée en tant que bloc de code monolithique, avec toute la logique métier regroupée dans un niveau unique partagé avec la couche de présentation.

Voici un exemple faisant appel à ASP.NET qui illustre le problème. Vous trouverez l’exemple complet [ici][code-sample].

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

- La méthode `Post` dans le contrôleur `WorkInFrontEnd` implémente une opération HTTP POST. Cette opération simule une tâche de longue durée nécessitant une utilisation importante du processeur. Le travail est réalisé sur un thread distinct afin de favoriser l’exécution rapide de l’opération POST.

- La méthode `Get` dans le contrôleur `UserProfile` implémente une opération HTTP GET. Cette méthode nécessite une utilisation beaucoup moins importante du processeur.

La principale préoccupation concerne les besoins en ressources de la méthode `Post`. Bien que cette méthode place le travail sur un thread d’arrière-plan, le travail est toujours susceptible de consommer un nombre considérable de ressources processeur. Ces ressources sont partagées avec les autres opérations exécutées par les autres utilisateurs simultanés. Si un nombre modéré d’utilisateurs envoient cette requête en même temps, les performances globales risquent d’être dégradées, entraînant un ralentissement de toutes les opérations. Les utilisateurs peuvent par exemple être confrontés à une latence importante pour la méthode `Get`.

## <a name="how-to-fix-the-problem"></a>Comment corriger le problème

Déplacez les processus qui consomment de nombreuses ressources sur un serveur principal distinct. 

Avec cette approche, le serveur frontal place les tâches nécessitant de nombreuses ressources dans une file d’attente. Le serveur principal récupère les tâches à des fins de traitement asynchrone. La file d’attente assure également le nivellement de la charge, mettant les requêtes en tampon pour le serveur principal. Si la file d’attente devient trop longue, vous pouvez configurer la mise à l’échelle automatique pour augmenter la taille des instances du serveur principal.

Voici une version révisée du code précédent. Dans cette version, la méthode `Post` place un message dans une file d’attente Service Bus. 

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

Le serveur principal extrait les messages de la file d’attente Service Bus et effectue le traitement.

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

## <a name="considerations"></a>Considérations

- Cette approche rend l’application un peu plus complexe. Vous devez gérer la mise en file d’attente et le retrait de la file d’attente de manière sûre pour éviter la perte de requêtes en cas de défaillance.
- L’application devient dépendante d’un service supplémentaire pour la file d’attente de messages.
- L’environnement de traitement doit être suffisamment évolutif pour gérer la charge de travail attendue et atteindre les objectifs de débit requis.
- Même si cette approche devrait normalement améliorer la réactivité globale, l’exécution des tâches déplacées sur le serveur principal peut prendre plus de temps. 

## <a name="how-to-detect-the-problem"></a>Comment détecter le problème

Les symptômes d’un serveur frontal occupé incluent une latence élevée pendant l’exécution des tâches nécessitant de nombreuses ressources. Les utilisateurs finals sont susceptibles de signaler des temps de réponse plus longs que d’habitude ou des échecs causés par l’expiration du délai d’attente des services. Ces échecs peuvent également renvoyer des erreurs HTTP 500 (serveur interne) ou HTTP 503 (service indisponible). Examinez les journaux d’événements du serveur web, qui contiennent probablement des informations plus détaillées sur les causes et les circonstances de ces erreurs.

Vous pouvez procéder de la manière suivante pour identifier ce problème :

1. Analysez le processus du système de production afin d’identifier les points où les temps de réponse augmentent.
2. Examinez les données de télémétrie enregistrées à ces points pour déterminer les opérations exécutées en même temps et les ressources utilisées. 
3. Recherchez les corrélations entre les temps de réponse médiocres et les volumes et combinaisons d’opérations observés à ces moments-là.
4. Procédez à un test de charge de chaque opération suspectée pour identifier les opérations qui consomment de nombreuses ressources et nuisent ainsi aux autres opérations. 
5. Révisez le code source de ces opérations afin de déterminer les raisons potentielles de cette consommation excessive de ressources.

## <a name="example-diagnosis"></a>Exemple de diagnostic 

Les sections suivantes appliquent ces étapes à l’exemple d’application décrit précédemment.

### <a name="identify-points-of-slowdown"></a>Identifier les points de ralentissement

Instrumentez chaque méthode pour suivre la durée des différentes requêtes et les ressources consommées par celles-ci. Ensuite, analysez l’application en production. Vous bénéficierez ainsi d’une vue d’ensemble de la manière dont les requêtes s’opposent. Pendant les périodes de forte activité, les requêtes gourmandes en ressources et dont l’exécution est lente sont susceptibles d’avoir un impact négatif sur les autres opérations, un comportement qu’il est possible de constater en analysant le système et en observant la baisse des performances.

L’image suivante illustre un tableau de bord d’analyse (nous avons utilisé [AppDynamics] pour nos tests). Au départ, le système présente une charge faible. Ensuite, les utilisateurs commencent à envoyer des requêtes pour la méthode GET au contrôleur `UserProfile`. Les performances sont relativement bonnes jusqu'à ce que d’autres utilisateurs commencent à émettre des requêtes pour la méthode POST à destination du contrôleur `WorkInFrontEnd`. À ce moment-là, les temps de réponse augmentent considérablement (première flèche). Les temps de réponse s’améliorent uniquement après la diminution du volume de requêtes envoyées au contrôleur `WorkInFrontEnd` (seconde flèche).

![Volet des transactions commerciales d’AppDynamics illustrant l’effet de l’utilisation du contrôleur WorkInFrontEnd sur les temps de réponse de l’ensemble des requêtes][AppDynamics-Transactions-Front-End-Requests]

### <a name="examine-telemetry-data-and-find-correlations"></a>Examiner les données de télémétrie et rechercher les corrélations

L’image suivante illustre quelques-unes des mesures collectées pour surveiller l’utilisation des ressources pendant le même intervalle de temps. Au départ, peu d’utilisateurs accèdent au système. À mesure que d’autres utilisateurs se connectent, l’utilisation du processeur devient très élevée (100 %). Par ailleurs, vous remarquerez qu’initialement, le taux d’E/S réseau augmente parallèlement à l’utilisation du processeur. Cependant, une fois que l’utilisation du processeur est au plus haut, les E/S réseau diminuent. Cela s’explique par le fait que le système peut seulement traiter un nombre relativement restreint de requêtes une fois que le processeur a atteint sa capacité maximale. À mesure que les utilisateurs se déconnectent, la charge du processeur diminue.

![Mesures d’AppDynamics reflétant l’utilisation du processeur et du réseau][AppDynamics-Metrics-Front-End-Requests]

À ce stade, la méthode `Post` dans le contrôleur `WorkInFrontEnd` semble être un candidat de choix pour un examen plus approfondi. Un travail supplémentaire dans un environnement contrôlé est nécessaire pour confirmer cette hypothèse.

### <a name="perform-load-testing"></a>Effectuer des tests de charge 

L’étape suivante consiste à effectuer des tests dans un environnement contrôlé. Par exemple, vous pouvez exécuter une série de tests de charge qui incluent puis omettent successivement chaque requête pour observer les effets.

Le graphique ci-dessous montre les résultats d’un test de charge effectué sur un déploiement identique du service cloud utilisé dans les tests précédents. Pour ce test, une charge constante de 500 utilisateurs exécutant l’opération `Get` dans le contrôleur `UserProfile` et une charge progressive d’utilisateurs exécutant l’opération `Post` dans le contrôleur `WorkInFrontEnd` ont été utilisées. 

![Résultats initiaux du test de charge pour le contrôleur WorkInFrontEnd][Initial-Load-Test-Results-Front-End]

Au départ, la charge progressive est de 0, ce qui signifie que les seuls utilisateurs actifs envoient des requêtes `UserProfile`. Le système est capable de répondre à environ 500 requêtes par seconde. Après 60 secondes, une charge de 100 utilisateurs supplémentaires, qui commencent à envoyer des requêtes POST au contrôleur `WorkInFrontEnd`, est ajoutée. Presque immédiatement, la charge de travail envoyée au contrôleur `UserProfile` descend à environ 150 requêtes par seconde. Cela est dû au mode de fonctionnement de l’exécuteur de tests de charge. Celui-ci attend une réponse avant d’envoyer la requête suivante. Par conséquent, plus la réponse met de temps à arriver, plus le taux de requêtes est faible.

À mesure que des utilisateurs supplémentaires envoient des requêtes POST au contrôleur `WorkInFrontEnd`, le taux de requêtes du contrôleur `UserProfile` continue à diminuer. Cependant, vous remarquerez que le volume de requêtes traitées par le contrôleur `WorkInFrontEnd` reste relativement constant. La saturation du système devient apparente à mesure que le taux global des deux requêtes tend vers une limite constante mais faible.


### <a name="review-the-source-code"></a>Réviser le code source

La dernière étape consiste à examiner le code source. L’équipe de développement était consciente que la méthode `Post` pouvait prendre beaucoup de temps, c’est pourquoi l’implémentation d’origine utilisait un thread distinct. Cela a permis de résoudre le problème immédiat, car la méthode `Post` ne se bloquait pas en attendant qu’une tâche de longue durée se termine.

Cependant, le travail exécuté par cette méthode consomme toujours des ressources processeur, mémoire et autres. Permettre l’exécution asynchrone de ce processus est en fait susceptible de dégrader les performances, car les utilisateurs peuvent déclencher un grand nombre de ces opérations simultanément, de manière non contrôlée. Le nombre de threads qu’un serveur peut exécuter est limité. Au-delà de cette limite, l’application risque d’obtenir une exception en essayant de démarrer un nouveau thread.

> [!NOTE]
> Vous ne devez pas pour autant éviter les opérations asynchrones. L’exécution d’une opération await asynchrone sur un appel réseau est une pratique recommandée (voir l’antimodèle [E/S synchrones][sync-io]). Le problème ici est que le travail nécessitant une utilisation importante du processeur a été engendré sur un autre thread. 

### <a name="implement-the-solution-and-verify-the-result"></a>Implémenter la solution et vérifier le résultat

L’image suivante montre l’analyse des performances après l’implémentation de la solution. La charge était similaire à celle illustrée précédemment, mais les temps de réponse pour le contrôleur `UserProfile` sont maintenant beaucoup plus courts. Sur la même durée, le volume de requêtes est passé de 2 759 à 23 565. 

![Volet des transactions commerciales d’AppDynamics illustrant l’effet de l’utilisation du contrôleur WorkInBackground sur les temps de réponse de l’ensemble des requêtes][AppDynamics-Transactions-Background-Requests]

Vous remarquerez que le contrôleur `WorkInBackground` a également traité un volume beaucoup plus important de requêtes. Cependant, il n’est pas possible de faire de comparaison directe dans ce cas, car le travail exécuté dans ce contrôleur est très différent de celui du code d’origine. En effet, au lieu d’exécuter un long calcul, la nouvelle version met simplement une requête en file d’attente. Le point à retenir est que cette méthode ne ralentit plus l’ensemble du système sous charge.

L’utilisation du processeur et l’utilisation du réseau témoignent également de l’amélioration des performances. L’utilisation du processeur n’a jamais atteint 100 %, et le volume de requêtes réseau traitées a été bien plus important que précédemment, ne diminuant qu’au moment où la charge de travail a baissé.

![Mesures d’AppDynamics reflétant l’utilisation du processeur et du réseau pour le contrôleur WorkInBackground][AppDynamics-Metrics-Background-Requests]

Le graphique suivant présente les résultats d’un test de charge. Le volume total de requêtes traitées est en nette hausse par rapport aux tests précédents.

![Résultats du test de charge pour le contrôleur BackgroundImageProcessing][Load-Test-Results-Background]

## <a name="related-guidance"></a>Aide connexe

- [Meilleures pratiques en matière de mise à l’échelle automatique][autoscaling]
- [Meilleures pratiques en matière de tâches en arrière-plan][background-jobs]
- [Modèle de nivellement de charge basé sur une file d’attente][load-leveling]
- [Style d’architecture Web-File d’attente-Agent de travail][web-queue-worker]

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


