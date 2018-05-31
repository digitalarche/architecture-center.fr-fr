---
title: Guide spécifique relatif au service de nouvelle tentative
description: Guide spécifique relatif au service pour définir le mécanisme de nouvelle tentative.
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: 65206c5f39a74d228c7eaa0fea0c5b1b0710b22f
ms.sourcegitcommit: bb348bd3a8a4e27ef61e8eee74b54b07b65dbf98
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/21/2018
ms.locfileid: "34423015"
---
# <a name="retry-guidance-for-specific-services"></a>Guide du mécanisme de nouvelle tentative relatif aux différents services

La plupart des services Azure et des kits de développement logiciel (SDK) client incluent un mécanisme de nouvelle tentative. Cependant, ceux-ci diffèrent, car chaque service a des exigences et des caractéristiques différentes, et donc chaque mécanisme de nouvelle tentative est réglé sur un service spécifique. Ce guide résume les fonctionnalités de mécanisme de nouvelle tentative pour la majorité des services Azure et inclut des informations pour vous aider à utiliser, adapter ou étendre le mécanisme de nouvelle tentative pour ce service.

Pour obtenir des instructions générales sur la gestion des erreurs temporaires, des nouvelles tentatives de connexion et des opérations en fonction des services et des ressources, consultez [Guide sur les nouvelles tentatives](./transient-faults.md).

Le tableau suivant récapitule les fonctionnalités de nouvelle tentative pour les services Azure décrits dans ce guide.

| **Service** | **Fonctionnalités de nouvelle tentative** | **Configuration de la stratégie** | **Portée** | **Fonctionnalités de télémétrie** |
| --- | --- | --- | --- | --- |
| **[Azure Active Directory](#azure-active-directory)** |Native dans la bibliothèque ADAL |Incorporée dans la bibliothèque ADAL |Interne |Aucun |
| **[Cosmos DB](#cosmos-db)** |Native dans le service |Non configurable |Globale |TraceSource |
| **[Event Hubs](#azure-event-hubs)** |Native dans le client |Par programme |Client |Aucun |
| **[Cache Redis](#azure-redis-cache)** |Native dans le client |Par programme |Client |TextWriter |
| **[Recherche](#azure-search)** |Native dans le client |Par programme |Client |ETW ou personnalisé |
| **[Service Bus](#service-bus)** |Native dans le client |Par programme |Gestionnaire d’espace de noms, fabrique de messagerie et client |ETW |
| **[Service Fabric](#service-fabric)** |Native dans le client |Par programme |Client |Aucun | 
| **[Base de données SQL avec ADO.NET](#sql-database-using-adonet)** |[Polly](#transient-fault-handling-with-polly) |Déclarative et par programme |Instructions uniques ou blocs de code |Personnalisée |
| **[Base de données SQL avec Entity Framework](#sql-database-using-entity-framework-6)** |Native dans le client |Par programme |Globale par domaine d’application |Aucun |
| **[SQL Database avec Entity Framework Core](#sql-database-using-entity-framework-core)** |Native dans le client |Par programme |Globale par domaine d’application |Aucun |
| **[Stockage](#azure-storage)** |Native dans le client |Par programme |Opérations individuelles et du client |TraceSource |

> [!NOTE]
> Pour la plupart des mécanismes de nouvelle tentative intégrés Azure, il n’existe actuellement aucune manière d’appliquer une autre stratégie de nouvelle tentative selon le type d’erreur ou d’exception. Vous devez configurer une stratégie qui fournit les performances et la disponibilité moyennes optimales. Une façon d’ajuster la stratégie consiste à analyser les fichiers journaux pour déterminer le type d’erreurs temporaires qui se produisent. 

## <a name="azure-active-directory"></a>Azure Active Directory
Azure Active Directory (AD) est une solution cloud complète de gestion des accès et des identités qui combine des services d’annuaire essentiels, une gouvernance avancée des identités, des services de sécurité et une gestion des accès aux applications. Azure AD offre également aux développeurs une plate-forme de gestion des identités pour permettre un contrôle d’accès à leurs applications, en fonction d’une stratégie et de règles centralisés.

### <a name="retry-mechanism"></a>Mécanisme de nouvelle tentative
La bibliothèque d’authentification d’Active Directory (ADAL) intègre un mécanisme de nouvelle tentative pour Azure Active Directory. Pour éviter les verrouillages imprévus, il est recommandé que les bibliothèques et codes d’application tiers ne procèdent **pas** à de nouvelles tentatives pour les connexions ayant échoué, mais autorisent la bibliothèque ADAL à gérer les nouvelles tentatives. 

### <a name="retry-usage-guidance"></a>Guide d’utilisation des nouvelles tentatives
Respectez les consignes suivantes lors de l’utilisation d’Azure Active Directory :

* Dans la mesure du possible, utilisez la bibliothèque ADAL et la prise en charge intégrée des nouvelles tentatives.
* Si vous utilisez l’API REST pour Azure Active Directory, réessayez l’opération si le code de résultat est 429 (trop de demandes) ou si vous rencontrez une erreur dans la plage 5xx. N’effectuez pas de nouvelle tentative pour toute autre erreur.
* Une stratégie de temporisation exponentielle est recommandée pour une utilisation dans des scénarios de traitement par lots avec Azure Active Directory.

Pensez à commencer par les paramètres ci-après pour les opérations liées aux nouvelles tentatives. Il s’agit de paramètres généraux. Vous devez par conséquent surveiller les opérations et optimiser les valeurs en fonction de votre propre scénario.

| **Contexte** | **Exemple de cible E2E<br />latence max** | **Stratégie de nouvelle tentative** | **Paramètres** | **Valeurs** | **Fonctionnement** |
| --- | --- | --- | --- | --- | --- |
| Interactif, interface utilisateur,<br />ou premier plan |2 secondes |FixedInterval |Nombre de tentatives<br />Intervalle avant nouvelle tentative<br />Première nouvelle tentative rapide |3<br />500 ms<br />true |Tentative 1 - délai 0 s<br />Tentative 2 - délai 500 ms<br />Tentative 3 - délai 500 ms |
| Arrière-plan ou <br />lot |60 secondes |ExponentialBackoff |Nombre de tentatives<br />Temporisation min<br />Temporisation max<br />Temporisation delta<br />Première nouvelle tentative rapide |5.<br />0 seconde<br />60 secondes<br />2 secondes<br />false |Tentative 1 - délai 0 s<br />Tentative 2 - délai ~2 s<br />Tentative 3 - délai ~6 s<br />Tentative 4 - délai ~14 s<br />Tentative 5 - délai ~30 s |

### <a name="more-information"></a>Plus d’informations
* [Bibliothèques d’authentification d’Azure Active Directory][adal]

## <a name="cosmos-db"></a>Cosmos DB

Cosmos DB est une base de données multimodèle entièrement gérée qui prend en charge les données JSON sans schéma. Elle offre des performances configurables et fiables, un traitement transactionnel JavaScript natif et est conçue pour le cloud avec une mise à l’échelle élastique.

### <a name="retry-mechanism"></a>Mécanisme de nouvelle tentative
La classe `DocumentClient` retente automatiquement les tentatives ayant échoué. Pour définir le nombre de nouvelles tentatives et le délai d’attente maximal, configurez [ConnectionPolicy.RetryOptions]. Les exceptions déclenchées par le client sont au-delà de la stratégie de nouvelle tentative ou ne sont pas des erreurs temporaires.

Si Cosmos DB limite le client, il renvoie une erreur HTTP 429. Vérifiez le code d’état dans `DocumentClientException`.

### <a name="policy-configuration"></a>Configuration de la stratégie
Le tableau suivant présente les paramètres par défaut pour la classe `RetryOptions`.

| Paramètre | Valeur par défaut | Description |
| --- | --- | --- |
| MaxRetryAttemptsOnThrottledRequests |9 |Nombre maximal de nouvelles tentatives en cas d’échec de la requête, car Cosmos DB a appliqué une limite du débit au client. |
| MaxRetryWaitTimeInSeconds |30 |La durée maximale de nouvelle tentative en secondes. |

### <a name="example"></a>Exemples
```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey); ;
var options = client.ConnectionPolicy.RetryOptions;
options.MaxRetryAttemptsOnThrottledRequests = 5;
options.MaxRetryWaitTimeInSeconds = 15;
```

### <a name="telemetry"></a>Télémétrie
Les nouvelles tentatives sont enregistrées sous forme de messages de trace non structurés via un **TraceSource**.NET. Vous devez configurer un **TraceListener** pour capturer les événements et les écrire dans un journal de destination approprié.

Par exemple, si vous ajoutez ce qui suit à votre fichier App.config, les traces seront générées dans un fichier texte dans le même emplacement que le fichier exécutable :

```xml
<configuration>
  <system.diagnostics>
    <switches>
      <add name="SourceSwitch" value="Verbose"/>
    </switches>
    <sources>
      <source name="DocDBTrace" switchName="SourceSwitch" switchType="System.Diagnostics.SourceSwitch" >
        <listeners>
          <add name="MyTextListener" type="System.Diagnostics.TextWriterTraceListener" traceOutputOptions="DateTime,ProcessId,ThreadId" initializeData="CosmosDBTrace.txt"></add>
        </listeners>
      </source>
    </sources>
  </system.diagnostics>
</configuration>
```

## <a name="event-hubs"></a>Event Hubs

Azure Event Hubs est un service d’ingestion de données de télémétrie à très grande échelle qui collecte, transforme et stocke des millions d’événements.

### <a name="retry-mechanism"></a>Mécanisme de nouvelle tentative
Le comportement du mécanisme de nouvelle tentative dans la bibliothèque de client Azure Event Hubs est contrôlé par la propriété `RetryPolicy` sur la classe `EventHubClient`. La stratégie par défaut effectue de nouvelles tentatives avec une interruption exponentielle lorsqu’Azure Event Hubs renvoie une exception temporaire `EventHubsException` ou une exception `OperationCanceledException`.

### <a name="example"></a>Exemples
```csharp
EventHubClient client = EventHubClient.CreateFromConnectionString("[event_hub_connection_string]");
client.RetryPolicy = RetryPolicy.Default;
```

### <a name="more-information"></a>Plus d’informations
[Bibliothèque de client .NET Standard pour Azure Event Hubs](https://github.com/Azure/azure-event-hubs-dotnet)

## <a name="azure-redis-cache"></a>Cache Redis Azure
Cache Redis Azure est un service de cache de faible latence et d’accès aux données rapide basé sur le cache Redis open source connu. Il est sécurisé, géré par Microsoft et accessible à partir de n’importe quelle application dans Azure.

Les instructions de cette section sont basées sur l’utilisation du client StackExchange.Redis pour accéder au cache. Vous trouverez une liste des autres clients appropriés sur le [site web de Redis](http://redis.io/clients), qui peuvent avoir des mécanismes de nouvelle tentative différents.

Notez que le client StackExchange.Redis utilise le multiplexage via une connexion unique. L’utilisation recommandée consiste à créer une instance du client au démarrage de l’application et à utiliser cette instance pour toutes les opérations sur le cache. Pour cette raison, la connexion au cache est effectuée une seule fois. Ainsi, toutes les instructions de cette section sont associées à la stratégie de nouvelle tentative pour cette connexion initiale et non pour chaque opération qui accède au cache.

### <a name="retry-mechanism"></a>Mécanisme de nouvelle tentative
Le client StackExchange.Redis utilise une classe de gestionnaire de connexions qui est configurée par le biais d’un ensemble d’options, incluant :

- **ConnectRetry**. Nombre de fois où une connexion au cache ayant échoué fera l’objet d’une nouvelle tentative.
- **ReconnectRetryPolicy**. Stratégie de nouvelle tentative à utiliser.
- **ConnectTimeout**. Temps d’attente maximal en millisecondes.

### <a name="policy-configuration"></a>Configuration de la stratégie
Les stratégies de nouvelle tentative sont configurées par programme en définissant les options pour le client avant de se connecter au cache. Cela peut être effectué en créant une instance de la classe **ConfigurationOptions**, en remplissant ses propriétés et en la transmettant à la méthode **Connect**.

Les classes intégrées prennent en charge un délai linéaire (constant) et une interruption exponentielle avec des intervalles avant nouvelle tentative aléatoires. Vous pouvez également créer une stratégie de nouvelle tentative personnalisée en implémentant l’interface **IReconnectRetryPolicy**.

L’exemple ci-après configure une stratégie de nouvelle tentative à l’aide d’une interruption exponentielle.

```csharp
var deltaBackOffInMilliseconds = TimeSpan.FromSeconds(5).Milliseconds;
var maxDeltaBackOffInMilliseconds = TimeSpan.FromSeconds(20).Milliseconds;
var options = new ConfigurationOptions
{
    EndPoints = {"localhost"},
    ConnectRetry = 3,
    ReconnectRetryPolicy = new ExponentialRetry(deltaBackOffInMilliseconds, maxDeltaBackOffInMilliseconds),
    ConnectTimeout = 2000
};
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

Vous pouvez également spécifier les options sous forme de chaîne et la transmettre à la méthode **Connect** . Notez que la propriété **ReconnectRetryPolicy** n’est pas définissable de cette façon, mais uniquement au moyen du code.

```csharp
var options = "localhost,connectRetry=3,connectTimeout=2000";
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

Vous pouvez également spécifier des options directement lorsque vous vous connectez au cache.

```csharp
var conn = ConnectionMultiplexer.Connect("redis0:6380,redis1:6380,connectRetry=3");
```

Pour plus d’informations, consultez la section concernant la [configuration de StackExchange.Redis](https://stackexchange.github.io/StackExchange.Redis/Configuration) dans la documentation de StackExchange.Redis.

Le tableau suivant présente les paramètres par défaut pour la stratégie de nouvelle tentative intégrée.

| **Contexte** | **Paramètre** | **Valeur par défaut**<br />(v 1.2.2) | **Signification** |
| --- | --- | --- | --- |
| ConfigurationOptions |ConnectRetry<br /><br />ConnectTimeout<br /><br />SyncTimeout<br /><br />ReconnectRetryPolicy |3<br /><br />5 000 ms maximum plus SyncTimeout<br />1 000<br /><br />LinearRetry 5 000 ms |Le nombre de répétitions de tentatives de connexion pendant l’opération de connexion initiale.<br />Délai d’attente (ms) pour les opérations de connexion. Pas un délai entre chaque tentative.<br />Temps (ms) pour permettre des opérations synchrones.<br /><br />Nouvelle tentative toutes les 5 000 ms.|

> [!NOTE]
> Dans le cas des opérations synchrones, le paramètre `SyncTimeout` peut augmenter la latence de bout en bout, mais la définition d’une valeur trop faible risque d’entraîner des délais d’expiration excessifs. Consultez l’article [Résolution des problèmes du cache Redis Azure][redis-cache-troubleshoot]. En règle générale, utilisez des opérations asynchrones plutôt que des opérations synchrones. Pour plus d’informations, consultez [Pipelines et multiplexeurs](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).
>
>

### <a name="retry-usage-guidance"></a>Guide d’utilisation des nouvelles tentatives
Respectez les consignes suivantes lors de l’utilisation du cache Redis Azure :

* Le client StackExchange Redis gère ses propres nouvelles tentatives, mais uniquement lors de l’établissement d’une connexion au cache au premier démarrage de l’application. Vous pouvez configurer le délai de connexion, le nombre de nouvelles tentatives et l’intervalle avant nouvelle tentative qui sont nécessaires pour établir cette connexion. Toutefois, la stratégie de nouvelle tentative ne s’applique pas aux opérations sur le cache.
* Au lieu d’utiliser un grand nombre de nouvelles tentatives, envisagez une solution de secours en accédant plutôt à la source de données d’origine.

### <a name="telemetry"></a>Télémétrie
Vous pouvez collecter des informations sur les connexions (mais pas sur d’autres opérations) à l’aide d’un élément **TextWriter**.

```csharp
var writer = new StringWriter();
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

Vous trouverez ci-dessous un exemple de la sortie générée.

```text
localhost:6379,connectTimeout=2000,connectRetry=3
1 unique nodes specified
Requesting tie-break from localhost:6379 > __Booksleeve_TieBreak...
Allowing endpoints 00:00:02 to respond...
localhost:6379 faulted: SocketFailure on PING
localhost:6379 failed to nominate (Faulted)
> UnableToResolvePhysicalConnection on GET
No masters detected
localhost:6379: Standalone v2.0.0, master; keep-alive: 00:01:00; int: Connecting; sub: Connecting; not in use: DidNotRespond
localhost:6379: int ops=0, qu=0, qs=0, qc=1, wr=0, sync=1, socks=2; sub ops=0, qu=0, qs=0, qc=0, wr=0, socks=2
Circular op-count snapshot; int: 0 (0.00 ops/s; spans 10s); sub: 0 (0.00 ops/s; spans 10s)
Sync timeouts: 0; fire and forget: 0; last heartbeat: -1s ago
resetting failing connections to retry...
retrying; attempts left: 2...
...
```

### <a name="examples"></a>Exemples
L’exemple de code ci-après configure un délai (linéaire) constant entre les nouvelles tentatives lors de l’initialisation du client StackExchange.Redis. Cet exemple indique comment définir la configuration à l’aide d’une instance **ConfigurationOptions**.

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();
            {
                try
                {
                    var retryTimeInMilliseconds = TimeSpan.FromSeconds(4).Milliseconds; // delay between retries

                    // Using object-based configuration.
                    var options = new ConfigurationOptions
                                        {
                                            EndPoints = { "localhost" },
                                            ConnectRetry = 3,
                                            ReconnectRetryPolicy = new LinearRetry(retryTimeInMilliseconds)
                                        };
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

L’exemple ci-après définit la configuration en spécifiant les options sous forme de chaîne. Le délai de connexion correspond au temps d’attente maximal autorisé pour la connexion au cache. Il ne s’agit pas du délai entre chaque nouvelle tentative. Notez que la propriété **ReconnectRetryPolicy** est uniquement définissable par le biais du code.

```csharp
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();
            {
                try
                {
                    // Using string-based configuration.
                    var options = "localhost,connectRetry=3,connectTimeout=2000";
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

Pour plus d’exemples, consultez [Configuration](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) sur le site web de projet.

### <a name="more-information"></a>Plus d’informations
* [Site web Redis](http://redis.io/)

## <a name="azure-search"></a>Recherche Azure
Azure Search permet d’ajouter des fonctionnalités de recherche puissantes et sophistiquées à un site web ou une application, d’ajuster rapidement et aisément les résultats de recherche, et de construire des modèles de classement enrichis et optimisés.

### <a name="retry-mechanism"></a>Mécanisme de nouvelle tentative
Le comportement de nouvelle tentative dans le Kit de développement logiciel (SDK) Recherche Azure est contrôlé par la méthode `SetRetryPolicy` sur les classes [SearchServiceClient] et [SearchIndexClient]. La stratégie par défaut effectue une nouvelle tentative avec temporisation exponentielle lorsque Recherche Azure renvoie une réponse 5xx ou 408 (Délai d’expiration de la demande).

### <a name="telemetry"></a>Télémétrie
Effectuez le suivi avec ETW ou via l’inscription d’un fournisseur de suivi personnalisé. Pour plus d’informations, consultez la [documentation AutoRest][autorest].

## <a name="service-bus"></a>Service Bus
Service Bus est une plate-forme de messagerie cloud qui propose l’échange de messages d’une façon faiblement couplée avec une amélioration de la mise à l’échelle et de la résilience pour les composants d’une application, si cette dernière est hébergée dans le cloud ou sur site.

### <a name="retry-mechanism"></a>Mécanisme de nouvelle tentative
Service Bus met en œuvre des nouvelles tentatives à l’aide d’implémentations de la classe de base [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) . Tous les clients Service Bus exposent une propriété **RetryPolicy** qui peut être définie sur une des implémentations de la classe de base **RetryPolicy**. Les implémentations intégrées sont :

* La [classe RetryExponential](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx). Elle expose les propriétés qui contrôlent l’intervalle de temporisation, le nombre de nouvelles tentatives et la propriété **TerminationTimeBuffer** utilisée pour limiter la durée totale nécessaire pour terminer l’opération.
* La [classe NoRetry](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx). Elle est utilisée lorsque de nouvelles tentatives au niveau de l’API Service Bus ne sont pas requises, notamment lorsque les nouvelles tentatives sont gérées par un autre processus dans le cadre d’un lot ou d’une opération en plusieurs étapes.

Les actions de Service Bus peuvent renvoyer une plage d’exceptions, comme celles répertoriées dans [Annexe : Exceptions de messagerie](http://msdn.microsoft.com/library/hh418082.aspx). La liste fournit plus d’informations sur ces exceptions si celles-ci indiquent qu’une nouvelle tentative de l’opération est requise. Par exemple, une exception [ServerBusyException](http://msdn.microsoft.com/library/microsoft.servicebus.messaging.serverbusyexception.aspx) indique que le client doit attendre un certain temps, puis recommencer l’opération. La survenue d’une exception **ServerBusyException** fait également basculer le Service Bus vers un mode différent, dans lequel un délai supplémentaire de 10 secondes est ajouté aux délais de nouvelle tentative calculés. Ce mode est réinitialisé au bout d’une courte période.

Les exceptions renvoyées à partir du Service Bus exposent la propriété **IsTransient** qui indique si le client doit retenter l’opération. La stratégie **RetryExponential** intégrée repose sur la propriété **IsTransient** de la classe **MessagingException**, qui est la classe de base pour toutes les exceptions Service Bus. Si vous créez des implémentations personnalisées de la classe de base **RetryPolicy**, vous pouvez combiner le type d’exception et la propriété **IsTransient** pour permettre un contrôle plus précis des actions liées aux nouvelles tentatives. Par exemple, vous pouvez détecter une exception **QuotaExceededException** et prendre des mesures pour vider la file d’attente avant de tenter de lui renvoyer un message.

### <a name="policy-configuration"></a>Configuration de la stratégie
Les stratégies de nouvelle tentative sont définies par programme et peuvent être considérées comme stratégie par défaut pour un **NamespaceManager** et un **MessagingFactory**, ou individuellement pour chaque client de messagerie. Pour définir la stratégie de nouvelle tentative par défaut pour une session de messagerie, vous définissez la propriété **RetryPolicy** de l’élément **NamespaceManager**.

```csharp
namespaceManager.Settings.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                                maxBackoff: TimeSpan.FromSeconds(30),
                                                                maxRetryCount: 3);
```

Pour définir la stratégie de nouvelle tentative par défaut pour tous les clients créée à partir d’une fabrique de messagerie, vous définissez l’élément **RetryPolicy** de **MessagingFactory**.

```csharp
messagingFactory.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                    maxBackoff: TimeSpan.FromSeconds(30),
                                                    maxRetryCount: 3);
```

Pour définir la stratégie de nouvelle tentative pour un client de messagerie ou remplacer sa stratégie par défaut, vous définissez sa propriété **RetryPolicy** à l’aide d’une instance de la classe de stratégie requise :

```csharp
client.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                            maxBackoff: TimeSpan.FromSeconds(30),
                                            maxRetryCount: 3);
```

Impossible de définir la stratégie de nouvelle tentative au niveau de l’opération individuelle. Elle s’applique à toutes les opérations pour le client de messagerie.
Le tableau suivant présente les paramètres par défaut pour la stratégie de nouvelle tentative intégrée.

| Paramètre | Valeur par défaut | Signification |
|---------|---------------|---------|
| Stratégie | Exponentielle | Temporisation exponentielle. |
| MinimalBackoff | 0 | Intervalle de temporisation minimal. Cette valeur est ajoutée à l’intervalle avant nouvelle tentative calculé à partir de deltaBackoff. |
| MaximumBackoff | 30 secondes | Intervalle de temporisation maximal. Le paramètre MaximumBackoff est utilisé si l’intervalle avant nouvelle tentative calculé est supérieur à la valeur MaxBackoff. |
| DeltaBackoff | 3 secondes | Intervalle de temporisation entre les tentatives. Des multiples de cette durée seront utilisés pour les tentatives suivantes. |
| TimeBuffer | 5 secondes | Mémoire tampon de délai d’expiration associé à la nouvelle tentative. Les nouvelles tentatives seront abandonnées si le temps restant est inférieur à la valeur TimeBuffer. |
| MaxRetryCount | 10 | Nombre maximal de nouvelles tentatives. |
| ServerBusyBaseSleepTime | 10 secondes | Si la dernière exception rencontrée était **ServerBusyException**, cette valeur sera ajoutée à l’intervalle avant nouvelle tentative calculé. Cette valeur ne peut pas être modifiée. |

### <a name="retry-usage-guidance"></a>Guide d’utilisation des nouvelles tentatives
Respectez les consignes suivantes lors de l’utilisation de Service Bus :

* Lors de l’utilisation de l’implémentation intégrée de l’élément **RetryExponential** , ne mettez pas en œuvre d’opération de secours dès que la stratégie réagit aux exceptions liées à un serveur occupé et bascule automatiquement vers un mode de nouvelle tentative approprié
* Service Bus prend en charge une fonctionnalité appelée Espaces de noms associés, qui met en œuvre le basculement automatique vers une file d’attente de sauvegarde dans un espace de noms séparé en cas d’échec de la file d’attente dans l’espace de noms principal. Les messages de la file d’attente secondaire peuvent être renvoyés à la file d’attente principale lors de la récupération Cette fonctionnalité permet de traiter les erreurs temporaires. Pour plus d’informations, consultez [Modèles de messagerie asynchrone et haute disponibilité](http://msdn.microsoft.com/library/azure/dn292562.aspx).

Pensez à commencer par les paramètres suivants pour les opérations liées aux nouvelles tentatives. Il s’agit de paramètres généraux. Vous devez par conséquent surveiller les opérations et optimiser les valeurs en fonction de votre propre scénario.

| Context | Exemple de latence maximale | Stratégie de nouvelle tentative | Paramètres | Fonctionnement |
|---------|---------|---------|---------|---------|
| Interactif, interface utilisateur ou premier plan | 2 secondes*  | Exponentielle | MinimumBackoff = 0 <br/> MaximumBackoff = 30 s <br/> DeltaBackoff = 300 ms <br/> TimeBuffer = 300 ms <br/> MaxRetryCount = 2 | Tentative 1 : délai 0 s <br/> Tentative 2 : délai ~ 300 ms <br/> Tentative 3 : délai ~ 900 ms |
| Arrière-plan ou lot | 30 secondes | Exponentielle | MinimumBackoff = 1 <br/> MaximumBackoff = 30 s <br/> DeltaBackoff = 1,75 s <br/> TimeBuffer = 5 s <br/> MaxRetryCount = 3 | Tentative 1 : délai ~ 1 s <br/> Tentative 2 : délai ~ 3 s <br/> Tentative 3 : délai ~ 6 ms <br/> Tentative 4 : délai ~ 13 ms |

\* N’inclut pas le délai supplémentaire qui s’ajoute en cas de réception d’une réponse du type Serveur occupé.

### <a name="telemetry"></a>Télémétrie
Service Bus enregistre les nouvelles tentatives sous forme d’événements ETW à l’aide d’ **EventSource**. Vous devez associer un élément **EventListener** à la source d’événement pour capturer les événements et les afficher dans la visionneuse de performances ou les écrire dans un journal de destination approprié. Vous pouvez utiliser le [bloc applicatif de journalisation sémantique](http://msdn.microsoft.com/library/dn775006.aspx) pour effectuer cette opération. Les événements de nouvelle tentative se présentent sous la forme suivante :

```text
Microsoft-ServiceBus-Client/RetryPolicyIteration
ThreadID="14,500"
FormattedMessage="[TrackingId:] RetryExponential: Operation Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05 at iteration 0 is retrying after 00:00:00.1000000 sleep because of Microsoft.ServiceBus.Messaging.MessagingCommunicationException: The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3, TimeStamp:9/5/2014 10:00:13 PM."
trackingId=""
policyType="RetryExponential"
operation="Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05"
iteration="0"
iterationSleep="00:00:00.1000000"
lastExceptionType="Microsoft.ServiceBus.Messaging.MessagingCommunicationException"
exceptionMessage="The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3,TimeStamp:9/5/2014 10:00:13 PM"
```

### <a name="examples"></a>Exemples
L’exemple de code suivant montre comment définir la stratégie de nouvelle tentative pour :

* Un gestionnaire d’espace de noms. La stratégie s’applique à toutes les opérations sur ce gestionnaire et ne peut pas être remplacée pour des opérations individuelles.
* Une fabrique de messagerie. La stratégie s’applique à tous les clients créés à partir de cette fabrique et ne peut pas être remplacée lors de la création de clients individuels.
* Un client de messagerie individuel. Après avoir créé un client, vous pouvez définir la stratégie de nouvelle tentative pour ce client. La stratégie s’applique à toutes les opérations sur ce client.

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Messaging;

namespace RetryCodeSamples
{
    class ServiceBusCodeSamples
    {
        private const string connectionString =
            @"Endpoint=sb://[my-namespace].servicebus.windows.net/;
                SharedAccessKeyName=RootManageSharedAccessKey;
                SharedAccessKey=C99..........Mk=";

        public async static Task Samples()
        {
            const string QueueName = "TestQueue";

            ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.Http;

            var namespaceManager = NamespaceManager.CreateFromConnectionString(connectionString);

            // The namespace manager will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for all operations on the namespace manager.
                namespaceManager.Settings.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);

                // Policies cannot be specified on a per-operation basis.
                if (!await namespaceManager.QueueExistsAsync(QueueName))
                {
                    await namespaceManager.CreateQueueAsync(QueueName);
                }
            }

            var messagingFactory = MessagingFactory.Create(
                namespaceManager.Address, namespaceManager.Settings.TokenProvider);
            // The messaging factory will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for clients created from it.
                messagingFactory.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);


                // Policies cannot be specified on a per-operation basis.
                var session = await messagingFactory.AcceptMessageSessionAsync();
            }

            {
                var client = messagingFactory.CreateQueueClient(QueueName);
                // The client inherits the policy from the factory that created it.


                // Set different values for the retry policy on the client.
                client.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0.1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);

                // Policies cannot be specified on a per-operation basis.
                var session = await client.AcceptMessageSessionAsync();
            }
        }
    }
}
```

### <a name="more-information"></a>Plus d’informations
* [Modèles de messagerie asynchrone et haute disponibilité](http://msdn.microsoft.com/library/azure/dn292562.aspx)

## <a name="service-fabric"></a>Service Fabric

La distribution de services fiables dans un cluster Service Fabric offre une protection contre la plupart des erreurs temporaires potentielles décrites dans cet article. Toutefois, certaines erreurs temporaires restent possibles. Par exemple, si le service d’affectation de noms reçoit une requête au moment précis où il fait l’objet d’un changement de routage, il lève une exception. En revanche, si la même requête arrive 100 millisecondes plus tard, elle aboutira probablement.

Service Fabric gère ce type d’erreur temporaire en interne. Vous pouvez configurer certains paramètres à l’aide de la classe `OperationRetrySettings` lorsque vous configurez vos services.  Le code ci-après présente un exemple. Dans la plupart des cas, cette opération n’est pas nécessaire, et les paramètres par défaut font l’affaire.

```csharp
FabricTransportRemotingSettings transportSettings = new FabricTransportRemotingSettings
{
    OperationTimeout = TimeSpan.FromSeconds(30)
};

var retrySettings = new OperationRetrySettings(TimeSpan.FromSeconds(15), TimeSpan.FromSeconds(1), 5);

var clientFactory = new FabricTransportServiceRemotingClientFactory(transportSettings);

var serviceProxyFactory = new ServiceProxyFactory((c) => clientFactory, retrySettings);

var client = serviceProxyFactory.CreateServiceProxy<ISomeService>(
    new Uri("fabric:/SomeApp/SomeStatefulReliableService"),
    new ServicePartitionKey(0));
```

### <a name="more-information"></a>Plus d’informations

* [Gestion des exceptions à distance](https://github.com/Microsoft/azure-docs/blob/master/articles/service-fabric/service-fabric-reliable-services-communication-remoting.md#remoting-exception-handling)

## <a name="sql-database-using-adonet"></a>Base de données SQL utilisant ADO.NET
La base de données SQL est une base de données SQL hébergée disponible en différentes tailles et sous forme de service standard (partagé) et premium (non partagé).

### <a name="retry-mechanism"></a>Mécanisme de nouvelle tentative
Les bases de données SQL n’intègrent aucune prise en charge pour les nouvelles tentatives en cas d’accès à l’aide d’ADO.NET. Toutefois, les codes de retour issus des demandes peuvent être utilisés pour déterminer le motif d’échec de ces dernières. Pour plus d’informations sur la limitation de SQL Database, consultez l’article [Limites de ressources de base de données SQL Azure](/azure/sql-database/sql-database-resource-limits). Pour obtenir la liste des codes d’erreur pertinents, consultez l’article [Codes d’erreur SQL pour les applications clientes SQL Database](/azure/sql-database/sql-database-develop-error-messages).

Vous pouvez utiliser la bibliothèque Polly afin d’implémenter le mécanisme de nouvelles tentatives pour SQL Database. Consultez la section [Gestion des erreurs temporaires avec Polly](#transient-fault-handling-with-polly).

### <a name="retry-usage-guidance"></a>Guide d’utilisation des nouvelles tentatives
Respectez les consignes suivantes lorsque vous accédez à la base de données SQL à l’aide d’ADO.NET :

* Choisissez l’option de service appropriée (partagée ou premium). Une instance partagée peut être soumise à des délais de connexion plus longs que d’ordinaire et à des limitations en raison de l’utilisation du serveur partagé par d’autres locataires. Si des performances prévisibles et des opérations fiables à faible latence sont requises, pensez à choisir l’option premium.
* Veillez à effectuer de nouvelles tentatives au niveau ou à l’étendue qui convient pour éviter les opérations non idempotent à l’origine d’une incohérence dans les données. Idéalement, toutes les opérations doivent être idempotent afin qu’elles puissent être répétées sans entraîner d’incohérence. Lorsque cela n’est pas le cas, la nouvelle tentative doit être effectuée à un niveau ou à une étendue qui autorise l’annulation de toutes les modifications associées en cas d’échec d’une opération ; par exemple, depuis une étendue transactionnelle. Pour plus d’informations, consultez [Couche d’accès aux données de Cloud Service Fundamentals - Gestion des erreurs temporaires](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).
* Une stratégie d’intervalle fixe n’est pas recommandée pour une utilisation avec la base de données SQL Azure à l’exception des scénarios interactifs comportant seulement quelques nouvelles tentatives à intervalles très courts. Envisagez plutôt d’utiliser une stratégie de temporisation exponentielle pour la majorité des scénarios.
* Choisissez une valeur appropriée pour les délais d’attente de connexion et de commande lors de la définition des connexions. Un délai d’attente trop court peut entraîner des échecs de connexion prématurés lorsque la base de données est occupée. Un délai d’attente trop long peut empêcher la logique de nouvelle tentative de fonctionner correctement en attendant trop longtemps avant la détection d’un échec de connexion. La valeur du délai d’attente est un composant de la latence de bout en bout ; Il est en fait ajouté au délai de nouvelle tentative spécifié dans la stratégie de nouvelle tentative pour chaque nouvelle tentative.
* Fermez la connexion après un certain nombre de nouvelles tentatives, même lorsque vous utilisez une logique de nouvelle tentative à temporisation exponentielle et renouvelez l’opération sur une nouvelle connexion. Le fait d’effectuer la même opération à plusieurs reprises sur la même connexion peut être un facteur entraînant des problèmes de connexion. Pour voir un exemple de cette technique, consultez [Couche d’accès aux données de Cloud Service Fundamentals - Gestion des erreurs temporaires](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).
* Lorsque le regroupement de connexions est en cours d’utilisation (par défaut), il est probable que la même connexion sera choisie à partir du pool, même après la fermeture et la réouverture d’une connexion. Si c’est le cas, une technique permettant de résoudre ce problème consiste à appeler la méthode **ClearPool** de la classe **SqlConnection** pour marquer la connexion comme n’étant pas réutilisable. Toutefois, vous ne devez effectuer cette opération qu’après l’échec de plusieurs tentatives de connexion, et uniquement lorsque vous rencontrez la classe spécifique d’erreurs temporaires, relative notamment à des délais d’attente SQL (code d’erreur -2) liés à des connexions défectueuses.
* Si le code d’accès aux données utilise des transactions lancées en tant qu’instances **TransactionScope** , la logique de nouvelle tentative doit ouvrir de nouveau la connexion et lancer une nouvelle étendue de transaction. Pour cette raison, le bloc de code renouvelable doit englober l’ensemble de la portée de la transaction.

Pensez à commencer par les paramètres suivants pour les opérations liées aux nouvelles tentatives. Il s’agit de paramètres généraux. Vous devez par conséquent surveiller les opérations et optimiser les valeurs en fonction de votre propre scénario.

| **Contexte** | **Exemple de cible E2E<br />latence max** | **Stratégie de nouvelle tentative** | **Paramètres** | **Valeurs** | **Fonctionnement** |
| --- | --- | --- | --- | --- | --- |
| Interactif, interface utilisateur,<br />ou premier plan |2 secondes |FixedInterval |Nombre de tentatives<br />Intervalle avant nouvelle tentative<br />Première nouvelle tentative rapide |3<br />500 ms<br />true |Tentative 1 - délai 0 s<br />Tentative 2 - délai 500 ms<br />Tentative 3 - délai 500 ms |
| Arrière-plan<br />ou lot |30 secondes |ExponentialBackoff |Nombre de tentatives<br />Temporisation min<br />Temporisation max<br />Temporisation delta<br />Première nouvelle tentative rapide |5.<br />0 seconde<br />60 secondes<br />2 secondes<br />false |Tentative 1 - délai 0 s<br />Tentative 2 - délai ~2 s<br />Tentative 3 - délai ~6 s<br />Tentative 4 - délai ~14 s<br />Tentative 5 - délai ~30 s |

> [!NOTE]
> Les cibles de latence de bout en bout supposent le délai d’attente par défaut pour les connexions au service. Si vous spécifiez des délais de connexion, la latence de bout en bout sera prolongée de ce temps supplémentaire pour toute nouvelle tentative.
>
>

### <a name="examples"></a>Exemples
Cette section vous indique comment utiliser Polly pour accéder à Azure SQL Database à l’aide d’un ensemble de stratégies de nouvelle tentative configurées dans la classe `Policy`.

Le code ci-après présente l’utilisation d’une méthode d’extension sur la classe `SqlCommand` qui appelle `ExecuteAsync` avec une interruption exponentielle.

```csharp
public async static Task<SqlDataReader> ExecuteReaderWithRetryAsync(this SqlCommand command)
{
    GuardConnectionIsNotNull(command);

    var policy = Policy.Handle<Exception>().WaitAndRetryAsync(
        retryCount: 3, // Retry 3 times
        sleepDurationProvider: attempt => TimeSpan.FromMilliseconds(200 * Math.Pow(2, attempt - 1)), // Exponential backoff based on an initial 200ms delay.
        onRetry: (exception, attempt) => 
        {
            // Capture some info for logging/telemetry.  
            logger.LogWarn($"ExecuteReaderWithRetryAsync: Retry {attempt} due to {exception}.");
        });

    // Retry the following call according to the policy.
    await policy.ExecuteAsync<SqlDataReader>(async token =>
    {
        // This code is executed within the Policy 

        if (conn.State != System.Data.ConnectionState.Open) await conn.OpenAsync(token);
        return await command.ExecuteReaderAsync(System.Data.CommandBehavior.Default, token);

    }, cancellationToken);
}
```

Cette méthode d’extension asynchrone est utilisable comme suit.

```csharp
var sqlCommand = sqlConnection.CreateCommand();
sqlCommand.CommandText = "[some query]";

using (var reader = await sqlCommand.ExecuteReaderWithRetryAsync())
{
    // Do something with the values
}
```

### <a name="more-information"></a>Plus d’informations
* [Couche d’accès aux données de Cloud Service Fundamentals - Gestion des erreurs temporaires](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)

Pour obtenir des conseils généraux sur le moyen de tirer le meilleur parti de SQL Database, consultez l’article [Azure SQL Database Performance and Elasticity Guide (Guide relatif à l’élasticité et aux performances d’Azure SQL Database)](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).

## <a name="sql-database-using-entity-framework-6"></a>Base de données SQL utilisant Entity Framework 6
La base de données SQL est une base de données SQL hébergée disponible en différentes tailles et sous forme de service standard (partagé) et premium (non partagé). Entity Framework est un mappeur relationnel objet qui permet aux développeurs .NET de travailler avec des données relationnelles à l’aide d’objets spécifiques de domaine. Il élimine le recours à la plupart du code d’accès aux données que les développeurs doivent généralement écrire.

### <a name="retry-mechanism"></a>Mécanisme de nouvelle tentative
La prise en charge de la fonctionnalité de nouvelle tentative est assurée lors de l’accès à la base de données SQL à l’aide d’Entity Framework 6.0 et version supérieure via un mécanisme appelé [Résilience des connexions/logique de nouvelle tentative](http://msdn.microsoft.com/data/dn456835.aspx). Les principales fonctionnalités de ce mécanisme sont :

* L’abstraction principale est l’interface **IDbExecutionStrategy** . Cette interface :
  * Définit les méthodes synchrones et asynchrones **Execute** \*.
  * Définit les classes qui peuvent être utilisées directement ou configurées sur un contexte de base de données comme une stratégie par défaut mappée sur un nom de fournisseur ou un nom de fournisseur et un nom de serveur. En cas de configuration sur un contexte, les nouvelles tentatives se produisent au niveau des opérations de base de données individuelles. Il peut y en avoir plusieurs pour une opération de contexte donnée.
  * Définit à quel moment et comment proposer une nouvelle tentative en cas d’échec de connexion.
* Il inclut plusieurs implémentations intégrées de l’interface **IDbExecutionStrategy** :
  * Par défaut : aucune nouvelle tentative.
  * Par défaut pour la base de données SQL (automatique) : aucune nouvelle tentative, mais inspecte les exceptions et y inclut la suggestion d’utiliser la stratégie de base de données SQL.
  * Par défaut pour la base de données SQL : exponentielle (héritée de la classe de base) et logique de détection de base de données SQL.
* Il implémente une stratégie de temporisation exponentielle qui inclut la répartition aléatoire.
* Les classes de nouvelle tentative intégrées sont avec état et ne sont pas thread-safe. Toutefois, elles peuvent être réutilisées une fois l’opération en cours terminée.
* Si le nombre de tentatives spécifié est dépassé, les résultats sont inclus dans une nouvelle exception. Il ne se propage pas dans l’exception en cours.

### <a name="policy-configuration"></a>Configuration de la stratégie
La prise en charge de la nouvelle tentative est assurée lors de l’accès à la base de données SQL à l’aide d’Entity Framework 6.0 et versions ultérieures. Les stratégies de nouvelle tentative sont configurées par programme. La configuration ne peut pas être modifiée opération par opération.

Lorsque vous configurez une stratégie sur le contexte comme stratégie par défaut, vous définissez une fonction qui crée une nouvelle stratégie à la demande. Le code suivant montre comment vous pouvez créer une classe de configuration de nouvelle tentative qui étend la classe de base **DbConfiguration** .

```csharp
public class BloggingContextConfiguration : DbConfiguration
{
  public BlogConfiguration()
  {
    // Set up the execution strategy for SQL Database (exponential) with 5 retries and 4 sec delay
    this.SetExecutionStrategy(
         "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4)));
  }
}
```

Vous pouvez alors la spécifier comme stratégie de nouvelle tentative par défaut pour toutes les opérations à l’aide de la méthode **SetConfiguration** de l’instance **DbConfiguration** lorsque l’application démarre. Par défaut, Entity Framework détecte et utilise automatiquement la classe de configuration.

```csharp
DbConfiguration.SetConfiguration(new BloggingContextConfiguration());
```

Vous pouvez spécifier la classe de configuration de nouvelle tentative pour un contexte en annotant la classe de contexte avec un attribut **DbConfigurationType** . Toutefois, si vous avez une seule classe de configuration, Entity Framework l’utilisera sans avoir à annoter le contexte.

```csharp
[DbConfigurationType(typeof(BloggingContextConfiguration))]
public class BloggingContext : DbContext
```

Si vous devez utiliser des stratégies de nouvelle tentative différentes pour des opérations spécifiques ou désactiver des nouvelles tentatives pour des opérations spécifiques, vous pouvez créer une classe de configuration qui vous permet de suspendre ou d’échanger des stratégies en définissant un indicateur dans le **CallContext**. La classe de configuration peut utiliser cet indicateur pour changer de stratégie ou désactiver la stratégie indiquée et utilisée comme stratégie par défaut. Pour plus d’informations, consultez [Suspendre la stratégie d’exécution](http://msdn.microsoft.com/dn307226#transactions_workarounds) dans la page de Restrictions liées aux stratégies d’exécution de nouvelle tentative (Entity Framework 6 et versions supérieures).

Une autre technique d’utilisation de stratégies de nouvelle tentative spécifiques pour des opérations individuelles consiste à créer une instance de la classe de stratégie requise et à fournir la configuration requise au moyen de paramètres. Vous pouvez ensuite faire appel à sa méthode **ExecuteAsync** .

```csharp
var executionStrategy = new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4));
var blogs = await executionStrategy.ExecuteAsync(
    async () =>
    {
        using (var db = new BloggingContext("Blogs"))
        {
            // Acquire some values asynchronously and return them
        }
    },
    new CancellationToken()
);
```

La façon la plus simple d’utiliser une classe **DbConfiguration** consiste à la localiser dans le même assembly que la classe **DbContext**. Toutefois, cela n’est pas approprié lorsque le même contexte est requis dans différents scénarios, comme dans le cas de stratégies de nouvelle tentative en arrière-plan et interactives. Si les différents contextes s’exécutent dans un AppDomain séparé, vous pouvez utiliser la prise en charge intégrée pour spécifier des classes de configuration dans le fichier de configuration ou la définir explicitement à l’aide de code. Si les différents contextes doivent s’exécuter dans le même AppDomain, une solution personnalisée sera nécessaire.

Pour plus d’informations, consultez [Configuration basée sur le code (Entity Framework 6 et versions supérieures)](http://msdn.microsoft.com/data/jj680699.aspx).

Le tableau suivant présente les paramètres par défaut pour la stratégie de nouvelle tentative intégrée lors de l’utilisation d’Entity Framework 6.

| Paramètre | Valeur par défaut | Signification |
|---------|---------------|---------|
| Stratégie | Exponentielle | Temporisation exponentielle. |
| MaxRetryCount | 5. | Nombre maximal de nouvelles tentatives. |
| MaxDelay | 30 secondes | Délai maximal entre les nouvelles tentatives. Cette valeur n’affecte pas le mode de calcul de la série de délais. Elle définit uniquement une limite supérieure. |
| DefaultCoefficient | 1 seconde | Coefficient du calcul de temporisation exponentielle. Cette valeur ne peut pas être modifiée. |
| DefaultRandomFactor | 1.1 | Multiplicateur utilisé pour l’ajout d’un délai aléatoire pour chaque entrée. Cette valeur ne peut pas être modifiée. |
| DefaultExponentialBase | 2 | Multiplicateur utilisé pour le calcul du délai suivant. Cette valeur ne peut pas être modifiée. |

### <a name="retry-usage-guidance"></a>Guide d’utilisation des nouvelles tentatives
Respectez les consignes suivantes lorsque vous accédez à la base de données SQL à l’aide d’Entity Framework 6 :

* Choisissez l’option de service appropriée (partagée ou premium). Une instance partagée peut être soumise à des délais de connexion plus longs que d’ordinaire et à des limitations en raison de l’utilisation du serveur partagé par d’autres locataires. Si des performances prévisibles et des opérations fiables à faible latence sont requises, pensez à choisir l’option premium.
* Une stratégie d’intervalle fixe n’est pas recommandée pour une utilisation avec la base de données SQL Azure. Utilisez plutôt une stratégie de temporisation exponentielle car le service peut être surchargé et des délais plus longs laissent davantage de temps de récupération.
* Choisissez une valeur appropriée pour les délais d’attente de connexion et de commande lors de la définition des connexions. Basez le délai d’attente à la fois sur la conception de la logique métier et sur les tests. Vous devrez peut-être modifier cette valeur au fil du temps en fonction de l’évolution des volumes de données ou des processus d’entreprise. Un délai d’attente trop court peut entraîner des échecs de connexion prématurés lorsque la base de données est occupée. Un délai d’attente trop long peut empêcher la logique de nouvelle tentative de fonctionner correctement en attendant trop longtemps avant la détection d’un échec de connexion. La valeur d’expiration est un composant de la latence de bout en bout, bien que vous ne puissiez pas aisément déterminer le nombre de commandes qui seront exécutées lors de l’enregistrement du contexte. Vous pouvez modifier le délai d’attente par défaut en définissant la propriété **CommandTimeout** de l’instance **DbContext**.
* Entity Framework prend en charge les configurations de nouvelle tentative définies dans les fichiers de configuration. Toutefois, pour une flexibilité maximale sur Azure, vous devez envisager la création de la configuration par programme au sein de l’application. Les paramètres spécifiques pour les stratégies de nouvelle tentative, tels que le nombre de nouvelles tentatives et les intervalles, peuvent être stockés dans le fichier de configuration du service et être utilisés en cours d’exécution pour créer les stratégies appropriées. Cela permet de modifier les paramètres sans avoir à redémarrer l’application.

Pensez à commencer par les paramètres ci-après pour les opérations liées aux nouvelles tentatives. Vous ne pouvez pas spécifier le délai entre chaque nouvelle tentative (il est fixe et généré sous la forme d’une séquence exponentielle). Vous ne pouvez spécifier que les valeurs maximales, comme indiqué ici, sauf si vous créez une stratégie de nouvelle tentative personnalisée. Il s’agit de paramètres généraux. Vous devez par conséquent surveiller les opérations et optimiser les valeurs en fonction de votre propre scénario.

| **Contexte** | **Exemple de cible E2E<br />latence max** | **Stratégie de nouvelle tentative** | **Paramètres** | **Valeurs** | **Fonctionnement** |
| --- | --- | --- | --- | --- | --- |
| Interactif, interface utilisateur,<br />ou premier plan |2 secondes |Exponentielle |MaxRetryCount<br />MaxDelay |3<br />750 ms |Tentative 1 - délai 0 s<br />Tentative 2 - délai 750 ms<br />Tentative 3 - délai 750 ms |
| Arrière-plan<br /> ou lot |30 secondes |Exponentielle |MaxRetryCount<br />MaxDelay |5.<br />12 secondes |Tentative 1 - délai 0 s<br />Tentative 2 - délai ~1 s<br />Tentative 3 - délai ~3 s<br />Tentative 4 - délai ~7 s<br />Tentative 5 - délai 12 s |

> [!NOTE]
> Les cibles de latence de bout en bout supposent le délai d’attente par défaut pour les connexions au service. Si vous spécifiez des délais de connexion, la latence de bout en bout sera prolongée de ce temps supplémentaire pour toute nouvelle tentative.
>
>

### <a name="examples"></a>Exemples
L’exemple de code suivant définit une solution d’accès aux données simple qui utilise Entity Framework. Il spécifie une stratégie de nouvelle tentative spécifique par la définition de l’instance d’une classe nommée **BlogConfiguration** qui étend **DbConfiguration**.

```csharp
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Data.Entity.SqlServer;
using System.Threading.Tasks;

namespace RetryCodeSamples
{
    public class BlogConfiguration : DbConfiguration
    {
        public BlogConfiguration()
        {
            // Set up the execution strategy for SQL Database (exponential) with 5 retries and 12 sec delay.
            // These values could be loaded from configuration rather than being hard-coded.
            this.SetExecutionStrategy(
                    "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(12)));
        }
    }

    // Specify the configuration type if more than one has been defined.
    // [DbConfigurationType(typeof(BlogConfiguration))]
    public class BloggingContext : DbContext
    {
        // Definition of content goes here.
    }

    class EF6CodeSamples
    {
        public async static Task Samples()
        {
            // Execution strategy configured by DbConfiguration subclass, discovered automatically or
            // or explicitly indicated through configuration or with an attribute. Default is no retries.
            using (var db = new BloggingContext("Blogs"))
            {
                // Add, edit, delete blog items here, then:
                await db.SaveChangesAsync();
            }
        }
    }
}
```

Vous trouverez plusieurs exemples d’utilisation du mécanisme de nouvelle tentative d’Entity Framework dans [Résilience des connexions/logique de nouvelle tentative](http://msdn.microsoft.com/data/dn456835.aspx).

### <a name="more-information"></a>Plus d’informations
* [Guide relatif à l’élasticité et aux performances de la base de données SQL Azure](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)

## <a name="sql-database-using-entity-framework-core"></a>Base de données SQL utilisant Entity Framework Core
[Entity Framework Core](/ef/core/) est un mappeur Objet Relationnel qui permet aux développeurs .NET Core de travailler avec des données à l’aide d’objets spécifiques du domaine. Il élimine le recours à la plupart du code d’accès aux données que les développeurs doivent généralement écrire. Cette version d’Entity Framework a été écrite à partir de zéro et n’hérite pas automatiquement de toutes les fonctionnalités d’EF6.x.

### <a name="retry-mechanism"></a>Mécanisme de nouvelle tentative
La prise en charge du mécanisme de nouvelle tentative est assurée lors de l’accès à SQL Database utilisant Entity Framework Core par le biais d’un mécanisme appelé [Résilience de connexion](/ef/core/miscellaneous/connection-resiliency). La résilience de connexion a été introduite dans EF Core 1.1.0.

L’abstraction principale est l’interface `IExecutionStrategy`. La stratégie d’exécution de SQL Server, incluant SQL Azure, tient compte des types d’exceptions qui peuvent faire l’objet de nouvelles tentatives et présente des valeurs par défaut sensibles pour le nombre maximal de nouvelles tentatives, le délai entre deux tentatives, et ainsi de suite.

### <a name="examples"></a>Exemples

Le code ci-après autorise les nouvelles tentatives automatiques lors de la configuration de l’objet DbContext, qui représente une session avec la base de données. 

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(
            @"Server=(localdb)\mssqllocaldb;Database=EFMiscellanous.ConnectionResiliency;Trusted_Connection=True;",
            options => options.EnableRetryOnFailure());
}
```

Le code ci-après indique comment exécuter une transaction avec de nouvelles tentatives automatiques à l’aide d’une stratégie d’exécution. La transaction est définie dans un délégué. En cas d’échec passager, la stratégie d’exécution appelle de nouveau le délégué.

```csharp
using (var db = new BloggingContext())
{
    var strategy = db.Database.CreateExecutionStrategy();

    strategy.Execute(() =>
    {
        using (var transaction = db.Database.BeginTransaction())
        {
            db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/dotnet" });
            db.SaveChanges();

            db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/visualstudio" });
            db.SaveChanges();

            transaction.Commit();
        }
    });
}
```

## <a name="azure-storage"></a>Stockage Azure
Les services de stockage Azure incluent le stockage des tables et des objets blob, les fichiers et les files d’attente de stockage.

### <a name="retry-mechanism"></a>Mécanisme de nouvelle tentative
Les nouvelles tentatives se produisent au niveau de chaque opération REST et font partie intégrante de l’implémentation de l’API client. Le kit de développement logiciel (SDK) de stockage client utilise des classes qui implémentent l’ [interface IExtendedRetryPolicy](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx).

Il existe différentes implémentations de l’interface. Les clients de stockage peuvent choisir parmi les stratégies spécialement conçues pour l’accès à des tables, des objets BLOB et des files d’attente. Chaque implémentation utilise une stratégie de nouvelle tentative différente qui définit essentiellement l’intervalle avant une nouvelle tentative et d’autres détails.

Les classes intégrées prennent en charge la stratégie linéaire (délai constant) et exponentielle avec répartition aléatoire des intervalles entre chaque nouvelle tentative. Il n’existe également aucune stratégie de nouvelle tentative à utiliser lorsqu’un autre processus gère les nouvelles tentatives à un niveau supérieur. Toutefois, vous pouvez implémenter vos propres classes de nouvelle tentative si vous avez des exigences spécifiques non fournies par les classes intégrées.

Les autres alternatives passent d’un emplacement de service de stockage principal à un emplacement de service de stockage secondaire si vous utilisez le stockage géo-redondant avec accès en lecture (RA-GRS) et que le résultat de la demande est une erreur renouvelable. Pour plus d’informations, consultez [Options de redondance du stockage Azure](http://msdn.microsoft.com/library/azure/dn727290.aspx) .

### <a name="policy-configuration"></a>Configuration de la stratégie
Les stratégies de nouvelle tentative sont configurées par programme. Une procédure classique consiste à créer et remplir une instance **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions** ou **QueueRequestOptions**.

```csharp
TableRequestOptions interactiveRequestOption = new TableRequestOptions()
{
  RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
  // For Read-access geo-redundant storage, use PrimaryThenSecondary.
  // Otherwise set this to PrimaryOnly.
  LocationMode = LocationMode.PrimaryThenSecondary,
  // Maximum execution time based on the business use case. 
  MaximumExecutionTime = TimeSpan.FromSeconds(2)
};
```

L’instance d’options de demande peut ensuite être définie sur le client et toutes les opérations avec le client utiliseront les options de demande spécifiées.

```csharp
client.DefaultRequestOptions = interactiveRequestOption;
var stats = await client.GetServiceStatsAsync();
```

Vous pouvez remplacer les options de demande client en transmettant une instance remplie de la classe d’options de demande en tant que paramètre aux méthodes d’opération.

```csharp
var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
```

Vous utilisez une instance **OperationContext** pour spécifier le code à exécuter lorsqu’une nouvelle tentative se produit et qu’une opération est terminée. Ce code peut collecter des informations sur l’opération à utiliser dans les journaux et la télémétrie

```csharp
// Set up notifications for an operation
var context = new OperationContext();
context.ClientRequestID = "some request id";
context.Retrying += (sender, args) =>
{
    /* Collect retry information */
};
context.RequestCompleted += (sender, args) =>
{
    /* Collect operation completion information */
};
var stats = await client.GetServiceStatsAsync(null, context);
```

En plus d’indiquer si un échec donne droit à une nouvelle tentative, les stratégies de nouvelle tentative étendues renvoient un objet **RetryContext** indiquant le nombre de nouvelles tentatives, les résultats de la dernière demande, et si la tentative suivante se déroulera dans l’emplacement principal ou secondaire (voir le tableau ci-dessous pour plus d’informations). Les propriétés de l’objet **RetryContext** peuvent être utilisées pour décider de l’éventualité d’une nouvelle tentative et du moment où elle doit se dérouler. Pour plus d’informations, reportez-vous à la [méthode IExtendedRetryPolicy.Evaluate](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx).

Les tableaux ci-après présentent les paramètres par défaut pour les stratégies de nouvelle tentative intégrées.

**Options de requête**

| **Paramètre** | **Valeur par défaut** | **Signification** |
| --- | --- | --- |
| MaximumExecutionTime | Aucun | Durée d’exécution maximale pour la demande, y compris toutes les nouvelles tentatives potentielles. Si elle n’est pas indiquée, la durée d’exécution autorisée d’une demande est illimitée. En d’autres termes, la demande peut se bloquer. |
| ServerTimeout | Aucun | Intervalle du délai d’attente du serveur pour la demande (la valeur est arrondie aux secondes). Si non spécifié, il utilisera la valeur par défaut pour toutes les demandes au serveur. En règle générale, la meilleure option consiste à omettre ce paramètre afin que le serveur par défaut soit utilisé. | 
| LocationMode | Aucun | Si le compte de stockage est créé avec l’option de duplication du stockage géo-redondant avec accès en lecture (RA-GRS), vous pouvez utiliser le mode de d’emplacement pour indiquer l’emplacement devant recevoir la demande. Par exemple, si **PrimaryThenSecondary** est spécifié, les demandes sont dans un premier temps toujours envoyées vers l’emplacement principal. En cas d’échec, la demande est envoyée vers l’emplacement secondaire. |
| RetryPolicy | ExponentialPolicy | Voir ci-dessous pour obtenir plus d’informations sur chaque option. |

**Stratégie exponentielle** 

| **Paramètre** | **Valeur par défaut** | **Signification** |
| --- | --- | --- |
| maxAttempt | 3 | Nombre de nouvelles tentatives. |
| deltaBackoff | 4 secondes | Intervalle de temporisation entre les tentatives. Multiples de ce laps de temps, y compris un élément aléatoire, seront utilisés pour les tentatives suivantes. |
| MinBackoff | 3 secondes | Ajouté à tous les intervalles des tentatives calculés à partir de deltaBackoff. Cette valeur ne peut pas être modifiée.
| MaxBackoff | 120 secondes | MaxBackoff est utilisé si l’intervalle des tentatives calculé est supérieur à MaxBackoff. Cette valeur ne peut pas être modifiée. |

**Stratégie linéaire**

| **Paramètre** | **Valeur par défaut** | **Signification** |
| --- | --- | --- |
| maxAttempt | 3 | Nombre de nouvelles tentatives. |
| deltaBackoff | 30 secondes | Intervalle de temporisation entre les tentatives. |

### <a name="retry-usage-guidance"></a>Guide d’utilisation des nouvelles tentatives
Respectez les consignes suivantes lorsque vous accédez aux services de stockage Azure à l’aide de l’API client de stockage :

* Utilisez les stratégies de nouvelle tentative intégrées de l’espace de noms Microsoft.WindowsAzure.Storage.RetryPolicies, lorsqu’elles sont adaptées à vos besoins. Dans la plupart des cas, ces stratégies suffisent.
* Utilisez la stratégie **ExponentialRetry** dans les opérations de traitement par lots, les tâches en arrière-plan ou les scénarios non interactifs. Dans ces scénarios, vous pouvez généralement accorder plus de temps à la récupération du service, ce qui par conséquent accroît les chances de réussite de l’opération.
* Vous pouvez spécifier la propriété **MaximumExecutionTime** du paramètre **RequestOptions** pour limiter la durée d’exécution totale. Prenez en compte le type et la taille de l’opération lorsque vous choisissez une valeur de délai d’attente.
* Si vous avez besoin d’implémenter une nouvelle tentative personnalisée, évitez de créer des wrappers autour des classes de client de stockage. Utilisez plutôt les fonctionnalités permettant d’étendre les stratégies existantes via l’interface **IExtendedRetryPolicy** .
* Si vous utilisez le stockage géo-redondant avec accès en lecture (RA-GRS), vous pouvez utiliser **LocationMode** pour indiquer que les nouvelles tentatives accéderont à la copie en lecture seule secondaire du magasin en cas d’échec de l’accès principal. Toutefois, lorsque vous utilisez cette option vous devez vous assurer que votre application peut fonctionner correctement avec des données qui peuvent être obsolètes si la réplication à partir du magasin principal n’est pas encore terminée.

Pensez à commencer par les paramètres suivants pour les opérations liées aux nouvelles tentatives. Il s’agit de paramètres généraux. Vous devez par conséquent surveiller les opérations et optimiser les valeurs en fonction de votre propre scénario.  

| **Contexte** | **Exemple de cible E2E<br />latence max** | **Stratégie de nouvelle tentative** | **Paramètres** | **Valeurs** | **Fonctionnement** |
| --- | --- | --- | --- | --- | --- |
| Interactif, interface utilisateur,<br />ou premier plan |2 secondes |Linéaire |maxAttempt<br />deltaBackoff |3<br />500 ms |Tentative 1 - délai 500 ms<br />Tentative 2 - délai 500 ms<br />Tentative 3 - délai 500 ms |
| Arrière-plan<br />ou lot |30 secondes |Exponentielle |maxAttempt<br />deltaBackoff |5.<br />4 secondes |Tentative 1 - délai ~3 sec<br />Tentative 2 - délai ~7 sec<br />Tentative 3 - délai ~15 sec |

### <a name="telemetry"></a>Télémétrie
Les nouvelles tentatives sont enregistrées dans un **TraceSource**. Vous devez configurer un **TraceListener** pour capturer les événements et les écrire dans un journal de destination approprié. Vous pouvez utiliser le **TextWriterTraceListener** ou **XmlWriterTraceListener** pour écrire les données dans un fichier journal, le **EventLogTraceListener** pour écrire dans le journal des événements Windows, ou le **EventProviderTraceListener** pour écrire les données de trace dans le sous-système ETW. Vous pouvez également configurer le vidage automatique de la mémoire tampon et le niveau de détail des événements qui seront enregistrés (par exemple, erreur, avertissement, information et informations détaillées). Pour plus d’informations, consultez [Journalisation côté client avec la bibliothèque cliente de stockage .NET](http://msdn.microsoft.com/library/azure/dn782839.aspx).

Les opérations peuvent recevoir une instance **OperationContext**, qui expose un événement **Retrying** pouvant être utilisé pour associer la logique personnalisée de télémétrie. Pour plus d’informations, consultez [OperationContext.Retrying Event](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).

### <a name="examples"></a>Exemples
L’exemple de code suivant montre comment créer deux instances **TableRequestOptions** avec des paramètres de nouvelle tentative différents ; une pour les demandes interactives et une pour les demandes d’arrière-plan. L’exemple définit ensuite ces deux jeux stratégies de nouvelle tentative sur le client de sorte qu’elles s’appliquent à toutes les demandes et définit également la stratégie interactive sur une demande spécifique afin qu’elle remplace les paramètres par défaut appliqués au client.

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.RetryPolicies;
using Microsoft.WindowsAzure.Storage.Table;

namespace RetryCodeSamples
{
    class AzureStorageCodeSamples
    {
        private const string connectionString = "UseDevelopmentStorage=true";

        public async static Task Samples()
        {
            var storageAccount = CloudStorageAccount.Parse(connectionString);

            TableRequestOptions interactiveRequestOption = new TableRequestOptions()
            {
                RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
                // For Read-access geo-redundant storage, use PrimaryThenSecondary.
                // Otherwise set this to PrimaryOnly.
                LocationMode = LocationMode.PrimaryThenSecondary,
                // Maximum execution time based on the business use case. 
                MaximumExecutionTime = TimeSpan.FromSeconds(2)
            };

            TableRequestOptions backgroundRequestOption = new TableRequestOptions()
            {
                // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
                // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
                MaximumExecutionTime = TimeSpan.FromSeconds(30),
                // PrimaryThenSecondary in case of Read-access geo-redundant storage, else set this to PrimaryOnly
                LocationMode = LocationMode.PrimaryThenSecondary
            };

            var client = storageAccount.CreateCloudTableClient();
            // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
            // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
            // ServerTimeout and MaximumExecutionTime are not set

            {
                // Set properties for the client (used on all requests unless overridden)
                // Different exponential policy parameters for background scenarios
                client.DefaultRequestOptions = backgroundRequestOption;
                // Linear policy for interactive scenarios
                client.DefaultRequestOptions = interactiveRequestOption;
            }

            {
                // set properties for a specific request
                var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
            }

            {
                // Set up notifications for an operation
                var context = new OperationContext();
                context.ClientRequestID = "some request id";
                context.Retrying += (sender, args) =>
                {
                    /* Collect retry information */
                };
                context.RequestCompleted += (sender, args) =>
                {
                    /* Collect operation completion information */
                };
                var stats = await client.GetServiceStatsAsync(null, context);
            }
        }
    }
}
```

### <a name="more-information"></a>Plus d’informations
* [Recommandations de stratégie de nouvelle tentative de Bibliothèque cliente de stockage Azure](https://azure.microsoft.com/blog/2014/05/22/azure-storage-client-library-retry-policy-recommendations/)
* [Bibliothèque cliente de stockage 2.0 - Mise en œuvre de stratégies de nouvelle tentative](http://gauravmantri.com/2012/12/30/storage-client-library-2-0-implementing-retry-policies/)

## <a name="general-rest-and-retry-guidelines"></a>Instructions générales relatives aux nouvelles tentatives et à REST
Prenez en compte les éléments suivants lors de l’accès aux services Azure ou tiers :

* Utilisez une approche systématique pour gérer les nouvelles tentatives, peut-être en tant que code réutilisable, afin de pouvoir appliquer une méthodologie cohérente sur tous les clients et toutes les solutions.
* Envisagez d’utiliser une infrastructure de nouvelle tentative, telle que le bloc applicatif de gestion des erreurs temporaires pour gérer les nouvelles tentatives si le service cible ou le client ne dispose d’aucun mécanisme de nouvelle tentative intégré. Cela vous permettra d’implémenter un comportement cohérent de nouvelle tentative, et peut fournir une stratégie de nouvelle tentative par défaut appropriée pour le service cible. Toutefois, vous devrez peut-être créer un code personnalisé de nouvelle tentative pour les services ne disposant pas d’un comportement standard, qui ne reposent pas sur les exceptions pour indiquer les erreurs temporaires, ou si vous souhaitez utiliser une réponse **Retry-Response** pour gérer le comportement de la nouvelle tentative.
* La logique de détection temporaire dépend de l’API client réelle utilisée pour appeler les appels REST. Certains clients, tels que la nouvelle classe **HttpClient** , ne lanceront pas d’exceptions pour les demandes terminées avec un code d’état HTTP indiquant un échec. Cela améliore les performances, mais vous empêche d’utiliser le bloc applicatif de gestion des erreurs temporaires. Dans ce cas, vous pourriez encapsuler l’appel à l’API REST avec le code qui génère des exceptions pour les codes d’état HTTP indiquant un échec, qui peuvent ensuite être traités par le bloc. Vous pouvez également utiliser un mécanisme différent pour piloter les nouvelles tentatives.
* Le code d’état HTTP renvoyé par le service peut permettre d’indiquer si l’échec est temporaire. Vous devrez peut-être examiner les exceptions générées par un client ou l’infrastructure de nouvelle tentative pour accéder au code d’état ou pour déterminer le type d’exception équivalent. Les codes HTTP suivants indiquent habituellement qu’une nouvelle tentative est appropriée :
  * 408 Délai d’expiration de la requête
  * 429 Trop de demandes
  * 500 Erreur interne du serveur
  * 502 Passerelle incorrecte
  * 503 Service indisponible
  * 504 Dépassement du délai de la passerelle
* Si vous basez votre logique de nouvelle tentative sur les exceptions, ce qui suit indique généralement un échec temporaire lorsqu’aucune connexion n’a pu être établie :
  * WebExceptionStatus.ConnectionClosed
  * WebExceptionStatus.ConnectFailure
  * WebExceptionStatus.Timeout
  * WebExceptionStatus.RequestCanceled
* Dans le cas d’un service à l’état non disponible, le service peut indiquer le délai approprié avant l’exécution d’une nouvelle tentative dans l’en-tête de réponse **Retry-After** ou dans un en-tête personnalisé différent. Les services peuvent également envoyer des informations supplémentaires sous la forme d’en-têtes personnalisés ou en les intégrant au contenu de la réponse. Le bloc applicatif de gestion des erreurs temporaires ne peut pas utiliser les en-têtes « retry-after » personnalisés ou standard.
* N’effectuez pas de nouvelle tentative pour les codes d’état représentant des erreurs du client (erreurs comprises dans la plage 4xx), excepté pour le code 408 Délai d’expiration de la requête.
* Testez minutieusement vos stratégies et mécanismes de nouvelle tentative sous différentes conditions, en utilisant différents états de réseau et en variant les charges du système.

### <a name="retry-strategies"></a>Stratégie de nouvelle tentative
Voici les types d’intervalles de stratégie de nouvelle tentative classiques :

* **Exponentielle**. Une stratégie de nouvelle tentative qui effectue un nombre spécifié de tentatives en utilisant une approche de secours exponentielle répartie de manière aléatoire pour déterminer l’intervalle entre deux tentatives. Par exemple : 

    ```csharp
    var random = new Random();

    var delta = (int)((Math.Pow(2.0, currentRetryCount) - 1.0) *
                random.Next((int)(this.deltaBackoff.TotalMilliseconds * 0.8),
                (int)(this.deltaBackoff.TotalMilliseconds * 1.2)));
    var interval = (int)Math.Min(checked(this.minBackoff.TotalMilliseconds + delta),
                    this.maxBackoff.TotalMilliseconds);
    retryInterval = TimeSpan.FromMilliseconds(interval);
    ```

* **Incrémentielle**. Une stratégie de nouvelle tentative avec un nombre spécifié de nouvelles tentatives et un intervalle de temps incrémentiel entre deux tentatives. Par exemple : 

    ```csharp
    retryInterval = TimeSpan.FromMilliseconds(this.initialInterval.TotalMilliseconds +
                    (this.increment.TotalMilliseconds * currentRetryCount));
    ```

* **LinearRetry**. Une stratégie de nouvelle tentative qui effectue un nombre spécifié de nouvelles tentatives en utilisant un intervalle fixe spécifique entre deux tentatives. Par exemple : 

    ```csharp
    retryInterval = this.deltaBackoff;
    ```

### <a name="transient-fault-handling-with-polly"></a>Gestion des erreurs temporaires avec Polly
[Polly] [polly]est une bibliothèque permettant de gérer par programme les nouvelles tentatives et les stratégies de [disjoncteur][circuit-breaker]. Le projet Polly est un membre de l’organisation [.NET Foundation][dotnet-foundation]. Dans le cas des services pour lesquels le client ne prend pas en charge les nouvelles tentatives en mode natif, Polly constitue une alternative valide et évite d’avoir à écrire un code de nouvelle tentative personnalisé, qui peut être difficile à implémenter correctement. Polly vous offre également un moyen de tracer les erreurs lorsqu’elles se produisent, ce qui vous permet de journaliser les nouvelles tentatives.


<!-- links -->

[adal]: /azure/active-directory/develop/active-directory-authentication-libraries
[autorest]: https://github.com/Azure/autorest/tree/master/docs
[circuit-breaker]: ../patterns/circuit-breaker.md
[ConnectionPolicy.RetryOptions]: https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.connectionpolicy.retryoptions.aspx
[dotnet-foundation]: https://dotnetfoundation.org/
[polly]: http://www.thepollyproject.org
[redis-cache-troubleshoot]: /azure/redis-cache/cache-how-to-troubleshoot
[SearchIndexClient]: https://msdn.microsoft.com/library/azure/microsoft.azure.search.searchindexclient.aspx
[SearchServiceClient]: https://msdn.microsoft.com/library/microsoft.azure.search.searchserviceclient.aspx


### <a name="more-information"></a>Plus d’informations
* [Résilience de connexion](/ef/core/miscellaneous/connection-resiliency)
* [Data Points - EF Core 1.1 (Points de données - EF Core 1.1)](https://msdn.microsoft.com/magazine/mt745093.aspx)


