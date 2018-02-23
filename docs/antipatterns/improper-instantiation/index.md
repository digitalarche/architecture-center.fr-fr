---
title: "Antimodèle d’instanciation incorrect"
description: "Évitez de créer continuellement de nouvelles instances d’un objet destiné à être créé une seule fois, puis partagé."
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 4d5ef9ad9e675b46df94b51e81d7a4bd4c1b25e9
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/23/2018
---
# <a name="improper-instantiation-antipattern"></a>Antimodèle d’instanciation incorrect

Le fait de créer continuellement de nouvelles instances d’un objet destiné à être créé une seule fois, puis partagé, peut affecter les performances. 

## <a name="problem-description"></a>Description du problème

De nombreuses bibliothèques fournissent des abstractions de ressources externes. En interne, ces classes gèrent généralement leurs propres connexions à la ressource, en agissant comme des répartiteurs que les clients peuvent utiliser pour accéder à la ressource. Voici quelques exemples de classes de répartiteurs appropriées pour les applications Azure :

- `System.Net.Http.HttpClient`. Communique avec un service web à l’aide du protocole HTTP.
- `Microsoft.ServiceBus.Messaging.QueueClient`. Publie et reçoit des messages sur une file d’attente Service Bus. 
- `Microsoft.Azure.Documents.Client.DocumentClient`. Se connecte à une instance Cosmos DB
- `StackExchange.Redis.ConnectionMultiplexer`. Se connecte à Redis, y compris le Cache Redis Azure.

Ces classes sont destinées à être instanciées une seule fois et réutilisées tout au long de la durée de vie d’une application. Toutefois, on pense à tort que ces classes doivent être acquises uniquement en cas de besoin et publiées rapidement. Celles répertoriées ici sont des bibliothèques .NET, mais le modèle n’est pas propre à .NET. L’exemple ASP.NET suivant crée une instance de `HttpClient` pour communiquer avec un service distant. Vous trouverez l’exemple complet [ici][sample-app].

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

Dans une application web, cette technique n’est pas évolutive. Un nouvel objet `HttpClient` est créé pour chaque requête de l’utilisateur. Sous une charge importante, le serveur web peut épuiser le nombre de sockets disponibles, ce qui entraîne des erreurs `SocketException`.

Ce problème n’est pas limité à la classe `HttpClient`. Les autres classes qui encapsulent des ressources ou qui sont coûteuses à créer peuvent entraîner des problèmes similaires. L’exemple suivant crée une instance de la classe `ExpensiveToCreateService`. Ici le problème n’est pas nécessairement l’épuisement des sockets, mais simplement la durée de création de chaque instance. La création et la destruction continuelles des instances de cette classe peuvent nuire à l’évolutivité du système.

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

## <a name="how-to-fix-the-problem"></a>Comment corriger le problème

Si la classe qui encapsule la ressource externe est partageable et thread-safe, créez une instance de singleton partagée ou un pool d’instances réutilisables de la classe.

L’exemple suivant utilise une instance `HttpClient` statique, ce qui entraîne le partage de la connexion sur l’ensemble des requêtes.

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

## <a name="considerations"></a>Considérations

- L’élément clé de cet antimodèle est la création et la destruction répétée d’instances d’un objet *partageable*. Si une classe n’est pas partageable (pas thread-safe), cet antimodèle ne s’applique pas.

- Le type de ressource partagée peut déterminer si vous devez utiliser un singleton ou créer un pool. La classe `HttpClient` a été conçue pour être partagée plutôt que regroupée. D’autres objets peuvent prendre en charge le regroupement pour permettre au système de répartir la charge de travail sur plusieurs instances.

- Les objets que vous partagez sur plusieurs requêtes *doivent* être thread-safe. La classe `HttpClient` est conçue pour être utilisée de cette manière, mais les autres classes ne peuvent pas en charge des requêtes simultanées. Par conséquent, reportez-vous à la documentation disponible.

- Certains types de ressources sont rares et ne doivent pas y être conservés, comme les connexions de base de données. Le maintien d’une connexion de base de données qui n’est pas requise peut empêcher les autres utilisateurs simultanés d’accéder à la base de données.

- Dans le .NET Framework, de nombreux objets établissant des connexions à des ressources externes sont créés à l’aide de méthodes de fabrique statiques d’autres classes qui gèrent ces connexions. Ces objets de fabrique sont destinés à être enregistrés et réutilisés, plutôt que supprimés et recréés. Par exemple, dans Microsoft Azure Service Bus, l’objet `QueueClient` est créé via un objet `MessagingFactory`. En interne, `MessagingFactory` gère les connexions. Pour plus d’informations, consultez [Meilleures pratiques relatives aux améliorations de performances à l’aide de la messagerie Service Bus][service-bus-messaging].

## <a name="how-to-detect-the-problem"></a>Comment détecter le problème

Ce problème inclut une baisse de débit et une augmentation du taux d’erreur, ainsi qu’un ou plusieurs symptômes parmi les suivants : 

- Une augmentation des exceptions indiquant l’insuffisance de ressources, telles que les sockets, les connexions de base de données, les descripteurs de fichiers etc. 
- Une utilisation de la mémoire et un nettoyage de la mémoire accrus.
- Une augmentation de l’activité du réseau, du disque ou de la base de données.

Vous pouvez procéder de la manière suivante pour identifier ce problème :

1. Analysez le processus du système de production afin d’identifier les points où les temps de réponse augmentent ou ceux où le système échoue en raison d’un manque de ressources.
2. Examinez les données de télémétrie capturées à ces points pour déterminer les opérations pouvant créer et détruire des objets consommateurs de ressources.
3. Effectuez un test de charge de chaque opération suspectée au sein d’un environnement de test contrôlé plutôt que dans le système de production.
4. Passez en revue le code source et examinez la façon dont les objets du répartiteur sont gérés.

Examinez l’arborescence des appels de procédure pour les opérations lentes ou celles qui génèrent des exceptions lorsque le système est sous charge. Ces informations peuvent aider à identifier la manière dont ces opérations utilisent les ressources. Les exceptions peuvent permettre de déterminer si les erreurs sont provoquées par l’épuisement des ressources partagées. 

## <a name="example-diagnosis"></a>Exemple de diagnostic

Les sections suivantes appliquent ces étapes à l’exemple d’application décrit précédemment.

### <a name="identify-points-of-slow-down-or-failure"></a>Identifier les points de ralentissement ou d’échec

L’illustration suivante montre les résultats générés à l’aide de l’[APM New Relic][new-relic], affichant les opérations dont le temps de réponse est médiocre. Dans ce cas, la méthode `GetProductAsync` du contrôleur `NewHttpClientInstancePerRequest` mérite un examen plus approfondi. Notez que le taux d’erreur augmente également lorsque ces opérations sont en cours d’exécution. 

![Tableau de bord d’analyse New Relic montrant l’exemple d’application créant une nouvelle instance d’un objet HttpClient pour chaque requête][dashboard-new-HTTPClient-instance]

### <a name="examine-telemetry-data-and-find-correlations"></a>Examiner les données de télémétrie et rechercher les corrélations

L’image suivante montre les données capturées à l’aide du thread de profilage, sur la même période correspondant à l’image précédente. Le système consacre un temps important à l’ouverture de connexions aux sockets et davantage pour leur fermeture et le traitement des exceptions de socket.

![Profileur de thread New Relic montrant l’exemple d’application créant une nouvelle instance d’un objet HttpClient pour chaque requête][thread-profiler-new-HTTPClient-instance]

### <a name="performing-load-testing"></a>Effectuer des tests de charge

Pour simuler les opérations courantes que les utilisateurs peuvent effectuer, utilisez le test de charge. Cela peut vous aider à identifier les parties d’un système subissant un épuisement des ressources sous différentes charges. Effectuer ces tests dans un environnement contrôlé plutôt que dans le système de production. Le graphique suivant montre le débit des requêtes traitées par le contrôleur `NewHttpClientInstancePerRequest` alors que la charge utilisateur augmente pour atteindre 100 utilisateurs simultanés.

![Débit de l’exemple d’application créant une nouvelle instance d’un objet HttpClient pour chaque requête][throughput-new-HTTPClient-instance]

Dans un premier temps, le volume des requêtes traitées par seconde augmente à mesure que la charge de travail s’accroît. Toutefois, à environ 30 utilisateurs, le volume des requêtes réussies atteint une limite et le système commence à générer des exceptions. Dès lors, le volume d’exceptions augmente progressivement avec la charge utilisateur. 

Le test de charge a signalé ces échecs sous forme d’erreurs HTTP 500 (serveur interne). L’examen de la télémétrie a montré que ces erreurs ont été causées par l’exécution du système en dehors des ressources de socket, au fur et à mesure de l’augmentation du nombre d’objets `HttpClient` créés.

Le graphique suivant montre un test similaire pour un contrôleur qui crée l’objet `ExpensiveToCreateService` personnalisé.

![Débit de l’exemple d’application créant une nouvelle instance de ExpensiveToCreateService pour chaque requête][throughput-new-ExpensiveToCreateService-instance]

Cette fois-ci, le contrôleur ne génère pas d’exceptions. Toutefois, le débit se stabilise, tandis que le temps de réponse moyen augmente selon un facteur de 20. Le graphique utilise une échelle logarithmique pour le débit et le temps de réponse. La télémétrie a montré que la création de nouvelles instances du `ExpensiveToCreateService` était la cause principale du problème.

### <a name="implement-the-solution-and-verify-the-result"></a>Implémenter la solution et vérifier le résultat

Après avoir changé la méthode `GetProductAsync` pour partager une seule instance `HttpClient`, un deuxième test de charge a montré une amélioration des performances. Aucune erreur n’a été signalée, et le système a été en mesure de gérer une charge croissante pouvant atteindre 500 requêtes par seconde. Le temps de réponse moyen a été réduit de moitié, comparé au test précédent.

![Débit de l’exemple d’application réutilisant la même instance d’un objet HttpClient pour chaque requête][throughput-single-HTTPClient-instance]

À des fins de comparaison, l’illustration suivante montre la télémétrie de trace de pile. Cette fois-ci, le système passe la majorité de son temps à effectuer un travail réel, plutôt qu’à ouvrir et à fermer des sockets.

![Profileur de thread New Relic montrant l’exemple d’application créant une seule instance d’un objet HttpClient pour toutes les requêtes][thread-profiler-single-HTTPClient-instance]

Le graphique suivant montre un test de charge similaire utilisant une instance partagée de l’objet `ExpensiveToCreateService`. Là encore, le volume de requêtes traitées augmente en fonction de la charge utilisateur, tandis que le temps de réponse moyen reste faible. 

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
