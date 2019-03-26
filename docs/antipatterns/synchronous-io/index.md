---
title: Anti-modèle E/S synchrone
titleSuffix: Performance antipatterns for cloud apps
description: Bloquer le thread appelant lorsque l’E/S se termine peut réduire les performances et affecter l’extensibilité verticale.
author: dragon119
ms.date: 06/05/2017
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
---

# <a name="synchronous-io-antipattern"></a>Anti-modèle E/S synchrone

Bloquer le thread appelant lorsque l’E/S se termine peut réduire les performances et affecter l’extensibilité verticale.

## <a name="problem-description"></a>Description du problème

Une opération d’E/S synchrone bloque le thread appelant pendant que l’E/S se termine. Le thread appelant entre dans un état d’attente et ne peut pas effectuer de travail utile pendant cet intervalle, gaspillant des ressources de traitement.

Voici quelques exemples courants d’E/S :

- Récupération ou conservation de données dans une base de données ou n’importe quel type de stockage permanent.
- Envoi d’une demande à un service web.
- Publication d’un message de validation ou récupération d’un message à partir d’une file d’attente.
- Lecture sur un fichier local ou à partir de ce dernier.

Cet antimodèle survient généralement pour les raisons suivantes :

- Cela semble être la manière la plus intuitive d’effectuer une opération.
- L’application requiert une réponse d’une requête.
- L’application utilise une bibliothèque qui fournit uniquement des méthodes synchrones pour E/S.
- Une bibliothèque externe effectue les opérations d’E/S synchrones en interne. Un seul appel d’E/S synchrone peut bloquer une chaîne d’appel entière.

Le code suivant télécharge un fichier vers le stockage d’objets blob Azure. Il existe deux emplacements où les blocs de code sont en attente d’E/S synchrones, la méthode `CreateIfNotExists` et la méthode `UploadFromStream`.

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

Voici un exemple d’attente de réponse d’un service externe. La méthode `GetUserProfile` appelle un service distant qui retourne un `UserProfile`.

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

Vous pouvez trouver le code complet pour ces deux exemples [ici][sample-app].

## <a name="how-to-fix-the-problem"></a>Comment corriger le problème

Remplacez les opérations d’E/S synchrones avec des opérations asynchrones. Cela libère le thread actuel pour continuer à effectuer un travail pertinent plutôt que de le bloquer et contribue à améliorer l’utilisation des ressources de calcul. L’exécution d’E/S de façon asynchrone est particulièrement efficace pour gérer un bond inattendu du nombre de demandes des applications clientes.

De nombreuses bibliothèques fournissent des versions synchrones et asynchrones des méthodes. Lorsque c’est possible, utilisez les versions asynchrones. Voici la version asynchrone de l’exemple précédent qui télécharge un fichier sur le stockage d’objets blob Azure.

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

L’opérateur `await` retourne le contrôle à l’environnement appelant pendant que l’opération asynchrone est effectuée. Le code après cette instruction se comporte comme une continuation qui s’exécute lorsque l’opération asynchrone est terminée.

Un service bien conçu doit également fournir des opérations asynchrones. Voici une version asynchrone du service web qui retourne les profils utilisateur. La méthode `GetUserProfileAsync` dépend d’une version asynchrone du service de profil utilisateur.

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

Pour les bibliothèques qui ne fournissent pas de versions asynchrones des opérations, il peut être possible de créer des wrappers asynchrones autour des méthodes synchrones sélectionnées. Suivez cette approche avec précaution. Bien qu’elle puisse améliorer les temps de réponse sur le thread qui appelle le wrapper asynchrone, elle consomme réellement davantage de ressources. Un thread supplémentaire peut être créé, résultant en une surcharge associée à la synchronisation du travail effectué par ce thread. Certains inconvénients sont présentés dans ce billet de blog : [Should I expose asynchronous wrappers for synchronous methods?][async-wrappers]

Voici un exemple d’un wrapper asynchrone autour d’une méthode synchrone.

```csharp
// Asynchronous wrapper around synchronous library method
private async Task<int> LibraryIOOperationAsync()
{
    return await Task.Run(() => LibraryIOOperation());
}
```

Le code appelant peut désormais attendre le wrapper :

```csharp
// Invoke the asynchronous wrapper using a task
await LibraryIOOperationAsync();
```

## <a name="considerations"></a>Considérations

- Les opérations d’E/S qui sont censées être de très courte durée et sont peu susceptibles de provoquer des contentions peuvent être plus performantes que les opérations synchrones. La lecture de petits fichiers sur un disque SSD en est un exemple. La surcharge de la distribution d’une tâche à un autre thread et la synchronisation avec ce thread lorsque la tâche se termine, peuvent dépasser les avantages d’E/S asynchrones. Toutefois, ces cas sont relativement rares, et la plupart des opérations d’E/S doivent être effectuées de façon asynchrone.

- L’amélioration des performances d’E/S peut entraîner la transformation d’autres parties du système en goulots d’étranglement. Par exemple, le déblocage de threads peut entraîner un plus grand volume de demandes simultanées à des ressources partagées, conduisant à son tour à une limitation ou à une insuffisance de ressources. Si cela devient un problème, vous devrez peut-être faire évoluer le nombre de serveurs web ou partitionner les magasins de données pour réduire la contention.

## <a name="how-to-detect-the-problem"></a>Comment détecter le problème

Pour les utilisateurs, l’application peut sembler inactive ou se bloquer régulièrement. L’application peut échouer avec les exceptions de délai d’expiration. Ces échecs peuvent également renvoyer des erreurs HTTP 500 (Erreur interne du serveur). Sur le serveur, les demandes client entrantes peuvent être bloquées jusqu’à ce qu’un thread soit disponible, ce qui entraîne des files d’attente de demande excessives, représentées par des erreurs HTTP 503 (service indisponible).

Vous pouvez procéder de la manière suivante pour identifier le problème :

1. Surveiller le système de production et déterminer si les threads de travail bloqués limitent le débit.

2. Si les demandes sont bloquées en raison d’un manque de threads, passez en revue l’application pour déterminer quelles opérations peuvent effectuer des E/S de façon synchrone.

3. Effectuez un test de charge contrôlé de chaque opération qui effectue des E/S synchrones, pour savoir si ces opérations affectent les performances du système.

## <a name="example-diagnosis"></a>Exemple de diagnostic

Les sections suivantes appliquent ces étapes à l’exemple d’application décrit précédemment.

### <a name="monitor-web-server-performance"></a>Surveiller les performances du serveur web

Pour les rôles web et les applications web Azure, il est intéressant d’analyser les performances du serveur web IIS. Faites particulièrement attention à la longueur de file d’attente de la demande pour établir si les demandes sont bloquées en attente des threads disponibles pendant les périodes de forte activité. Vous pouvez collecter ces informations en activant les diagnostics Azure. Pour plus d'informations, consultez les pages suivantes :

- [Surveiller les applications dans Azure App Service][web-sites-monitor]
- [Créer et utiliser des compteurs de performances dans une application Azure][performance-counters]

Instrumentez l’application pour voir comment les demandes sont traitées une fois qu’ils ont été acceptées. Suivre le flux d’une demande peut aider à identifier si elle effectue des appels à exécution lente et si cela bloque le thread actuel. Le profilage du thread permet également de mettre en surbrillance les demandes qui sont bloquées.

### <a name="load-test-the-application"></a>Effectuer un test de charge de l’application

Le graphique suivant montre les performances de la méthode synchrone `GetUserProfile` présentée précédemment, sous différentes charges d’utilisateurs simultanés (jusqu’à 4 000). L’application est une application ASP.NET en cours d’exécution dans un rôle web de service cloud Azure.

![Graphique de performances pour l’exemple d’application effectuant des opérations d’E/S synchrones][sync-performance]

L’opération synchrone est codée en dur pour la mise en veille pendant 2 secondes, pour simuler des E/S synchrones, par conséquent, le temps de réponse minimal est légèrement supérieur à 2 secondes. Lorsque la charge atteint environ 2 500 utilisateurs simultanés, le temps de réponse moyen commence à stagner, bien que le volume de demandes par seconde continue d’augmenter. Notez que l’échelle pour ces deux mesures est logarithmique. Le nombre de demandes par seconde double entre ce point et la fin du test.

En isolation, il n’est pas nécessairement clair à partir de ce test de définir si les E/S synchrones sont un problème. Sous une charge plus importante, l’application peut atteindre un point de basculement où le serveur web ne peut plus traiter les demandes en temps voulu, causant l’envoi d’exceptions de délai d’attente aux applications clientes.

Les demandes entrantes sont mises en file d’attente par le serveur web IIS et transmises à un thread en cours d’exécution dans le pool de thread ASP.NET. Étant donné que chaque opération effectue des opérations d’E/S de manière synchrone, le thread est bloqué jusqu’à ce que l’opération se termine. À mesure que la charge de travail augmente, tous les threads ASP.NET dans le pool de threads sont finalement alloués et bloqués. À ce stade, toute demande entrante supplémentaire doit attendre dans la file d’attente qu’un thread soit disponible. Lorsque la longueur de file d’attente augmente, les requêtes commencent à expirer.

### <a name="implement-the-solution-and-verify-the-result"></a>Implémenter la solution et vérifier le résultat

Le graphique suivant affiche les résultats du test de charge de la version asynchrone du code.

![Graphique de performances pour l’exemple d’application effectuant des opérations d’E/S asynchrones][async-performance]

Le débit est beaucoup plus élevé. Sur la même durée que le test précédent, le système gère correctement un débit presque dix fois supérieur, exprimé en nombre de requêtes par seconde. En outre, le temps de réponse moyen est relativement constant et reste environ 25 fois plus court que pour le test précédent.

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/SynchronousIO
[async-wrappers]: https://blogs.msdn.microsoft.com/pfxteam/2012/03/24/should-i-expose-asynchronous-wrappers-for-synchronous-methods/
[performance-counters]: /azure/cloud-services/cloud-services-dotnet-diagnostics-performance-counters
[web-sites-monitor]: /azure/app-service-web/web-sites-monitor

[sync-performance]: _images/SyncPerformance.jpg
[async-performance]: _images/AsyncPerformance.jpg
