---
title: Analyse du mode d’échec
description: Instructions pour effectuer l’analyse du mode d’échec pour les solutions cloud basées sur Azure.
author: MikeWasson
ms.date: 03/24/2017
ms.custom: resiliency
pnp.series.title: Design for Resiliency
ms.openlocfilehash: 95068bf8b1f5b559255e27819aaddb454d3427bc
ms.sourcegitcommit: d08f6ee27e1e8a623aeee32d298e616bc9bb87ff
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/07/2018
---
# <a name="failure-mode-analysis"></a>Analyse du mode d’échec
[!INCLUDE [header](../_includes/header.md)]

L’analyse du mode d’échec (FMA) est un processus de création de résilience dans un système, en identifiant les points de défaillance possibles. La FMA doit faire partie des phases de conception et d’architecture, afin que vous puissiez générer une récupération après une défaillance dans le système depuis le début.

Voici le processus général pour effectuer une FMA :

1. Identifiez tous les composants dans le système. Incluez les dépendances externes, telles que les fournisseurs d’identité, les services tiers et ainsi de suite.   
2. Pour chaque composant, identifiez les défaillances qui peuvent se produire. Un seul composant peut avoir plusieurs modes d’échec. Par exemple, vous devriez prendre en compte séparément les échecs de lecture et d’écriture, car les solutions et les impacts possibles seront différents.
3. Évaluez chaque mode d’échec en fonction de son niveau de risque global. Tenez compte de ces facteurs :  

   * Quelle est la probabilité de l’échec. Est-il relativement courant ? Très rare ? Vous n’avez pas besoin de chiffres exacts, le but est d’aider à classer par ordre priorité.
   * Quel est l’impact sur l’application, en termes de disponibilité, de pertes de données, de coût monétaire et d’interruption de service ?
4. Pour chaque mode de défaillance, déterminez comment l’application va répondre et récupérer. Étudiez les compromis au niveau du le coût et de la complexité de l’application.   

Point de départ pour votre processus FMA, cet article contient un catalogue des modes de défaillance potentiels et les mesures de corrections associées. Le catalogue est organisé par technologie ou service Azure, ainsi qu’une catégorie générale pour la conception au niveau de l’application. Le catalogue n’est pas exhaustif, mais traite beaucoup des principaux services Azure.

## <a name="app-service"></a>App Service
### <a name="app-service-app-shuts-down"></a>L’application App Service s’arrête.
**Détection**. Causes possibles :

* Arrêt planifié

  * Un opérateur arrête l’application ; via le portail Azure par exemple.
  * L’application a été déchargée, car elle était inactive. (Uniquement si le paramètre `Always On` est désactivé.)
* Arrêt inattendu

  * L’application se bloque.
  * Une instance de machine virtuelle de App Service est indisponible.

L’enregistrement Application_End interceptera l’arrêt du domaine d’application (panne de processus logicielle) et il s’agit de la seule façon d’intercepter les arrêts de domaine d’application.

**Récupération**

* Si l’arrêt était attendu, utilisez l’événement d’arrêt de l’application pour arrêter l’application de manière appropriée. Par exemple, dans ASP.NET, utilisez la méthode `Application_End`.
* Si l’application a été déchargée une fois inactive, elle est automatiquement redémarrée lors de la prochaine requête. Toutefois, cela occasionne le coût du « démarrage à froid ».
* Pour empêcher le déchargement de l’application une fois inactive, vous devez activer la paramètre `Always On` dans l’application web. Voir [Configurer des applications web dans Azure App Service][app-service-configure].
* Pour empêcher un opérateur d’arrêter l’application, définissez un verrou de ressource avec un niveau `ReadOnly`. Consultez [Verrouiller des ressources avec Azure Resource Manager][rm-locks].
* En cas de blocage de l’application ou si une machine virtuelle de App Service devient indisponible, App Service redémarre automatiquement l’application.

**Diagnostics**. Journaux des serveurs web et d'applications. Consultez la page [Activer la journalisation des diagnostics pour les applications web dans Azure App Service][app-service-logging].

### <a name="a-particular-user-repeatedly-makes-bad-requests-or-overloads-the-system"></a>Un utilisateur particulier effectue des requêtes incorrectes ou surcharge le système à plusieurs reprises.
**Détection**. Authentifiez les utilisateurs et incluez l’ID d’utilisateur dans les journaux d’applications.

**Récupération**

* Utilisez [Gestion des API Azure][api-management] pour limiter les requêtes de l’utilisateur. Consultez [Limitation de requêtes avancée avec Gestion des API Azure][api-management-throttling]
* Bloquez l’utilisateur.

**Diagnostics**. Journalisation de toutes les requêtes d’authentification.

### <a name="a-bad-update-was-deployed"></a>Une mise à jour incorrecte a été déployée.
**Détection**. Surveillez l’intégrité de l’application via le portail Azure (consultez [Surveiller les performances de l’application web Azure][app-insights-web-apps]) ou implémentez le [modèle de contrôle d’intégrité de point de terminaison][health-endpoint-monitoring-pattern].

**Récupération**. Utilisez plusieurs [emplacements de déploiement][app-service-slots] et l’annulation du déploiement vers le dernier déploiement correct. Pour plus d’informations, voir la rubrique [Authentification web de base][ra-web-apps-basic].

## <a name="azure-active-directory"></a>Azure Active Directory
### <a name="openid-connect-oidc-authentication-fails"></a>L’authentification OpenID Connect (OIDC) échoue.
**Détection**. Les modes d’échec possibles sont les suivantes :

1. Azure AD n’est pas disponible ou ne peut pas être atteint en raison d’un problème réseau. La redirection vers le point de terminaison de l’authentification échoue et l’intergiciel (middleware) OIDC lève une exception.
2. Le locataire Azure AD n’existe pas. La redirection vers le point de terminaison de l’authentification retourne un code d’erreur HTTP, et l’intergiciel (middleware) OIDC lève une exception.
3. L’utilisateur ne peut pas s’authentifier. Aucune stratégie de détection n’est nécessaire ; Azure AD gère les échecs de connexion.

**Récupération**

1. Interceptez des exceptions non gérées à partir de l’intergiciel (middleware).
2. Gérez les événements `AuthenticationFailed`.
3. Redirigez l’utilisateur vers une page d’erreur.
4. L’utilisateur essaye de nouveau.

## <a name="azure-search"></a>Recherche Azure
### <a name="writing-data-to-azure-search-fails"></a>L’écriture des données sur Recherche Azure échoue.
**Détection**. Interceptez les erreurs `Microsoft.Rest.Azure.CloudException`.

**Récupération**

Le [kit de développement logiciel .NET][search-sdk] tente automatiquement après des échecs temporaires. Toutes les exceptions levées par le kit de développement logiciel client doivent être traitées comme des erreurs non temporaires.

La stratégie de nouvelles tentatives par défaut utilise une temporisation exponentielle. Pour utiliser une stratégie de nouvelles tentatives différente, appelez `SetRetryPolicy` sur la classe `SearchIndexClient` ou `SearchServiceClient`. Pour plus d’informations, voir [Nouvelles tentatives automatiques][auto-rest-client-retry].

**Diagnostics**. Utilisez [Rechercher l’analyse du trafic][search-analytics].

### <a name="reading-data-from-azure-search-fails"></a>La lecture des données à partir de Recherche Azure échoue.
**Détection**. Interceptez les erreurs `Microsoft.Rest.Azure.CloudException`.

**Récupération**

Le [kit de développement logiciel .NET][search-sdk] tente automatiquement après des échecs temporaires. Toutes les exceptions levées par le kit de développement logiciel client doivent être traitées comme des erreurs non temporaires.

La stratégie de nouvelles tentatives par défaut utilise une temporisation exponentielle. Pour utiliser une stratégie de nouvelles tentatives différente, appelez `SetRetryPolicy` sur la classe `SearchIndexClient` ou `SearchServiceClient`. Pour plus d’informations, voir [Nouvelles tentatives automatiques][auto-rest-client-retry].

**Diagnostics**. Utilisez [Rechercher l’analyse du trafic][search-analytics].

## <a name="cassandra"></a>Cassandra
### <a name="reading-or-writing-to-a-node-fails"></a>La lecture ou l’écriture dans un nœud échoue.
**Détection**. Interceptez l’exception. Pour les clients .NET, il s’agira généralement de `System.Web.HttpException`. Les autres clients peuvent être sujets à d’autres types d’exceptions.  Pour plus d’informations, consultez [Gestion correcte des erreurs Cassandra](http://www.datastax.com/dev/blog/cassandra-error-handling-done-right).

**Récupération**

* Chaque [client de Cassandra](https://wiki.apache.org/cassandra/ClientOptions) possède ses propres fonctionnalités et stratégies de nouvelles tentatives. Pour plus d’informations, consultez [Gestion correcte des erreurs Cassandra][cassandra-error-handling].
* Utilisez un déploiement en mode rack, avec les nœuds de données répartis entre les domaines d’erreur.
* Déployez dans plusieurs régions avec une cohérence de quorum locale. Si une défaillance non temporaire se produit, basculez vers une autre région.

**Diagnostics**. Journaux d’application

## <a name="cloud-service"></a>Service cloud
### <a name="web-or-worker-roles-are-unexpectedlybeing-shut-down"></a>Les rôles web ou de travail sont arrêtés de façon inattendue.
**Détection**. L’événement [RoleEnvironment.Stopping][RoleEnvironment.Stopping] est déclenché.

<strong>Récupération</strong>. Remplacez la méthode [RoleEntryPoint.OnStop][RoleEntryPoint.OnStop] pour nettoyer de manière appropriée. Pour plus d’informations, consultez [La bonne manière de gérer les événements de Azure OnStop][onstop-events] (blog).

## <a name="cosmos-db"></a>Cosmos DB 
### <a name="reading-data-fails"></a>Échec de la lecture des données.
**Détection**. Interceptez `System.Net.Http.HttpRequestException` ou `Microsoft.Azure.Documents.DocumentClientException`.

**Récupération**

* Le kit de développement logiciel retente automatiquement les tentatives ayant échoué. Pour définir le nombre de nouvelles tentatives et le délai d’attente maximal, configurez `ConnectionPolicy.RetryOptions`. Les exceptions déclenchées par le client sont au-delà de la stratégie de nouvelle tentative ou ne sont pas des erreurs temporaires.
* Si Cosmos DB limite le client, il renvoie une erreur HTTP 429. Vérifiez le code d’état dans `DocumentClientException`. Si l’erreur 429 se produit régulièrement, envisagez d’augmenter la valeur de débit de la collection.
    * Si vous utilisez l’API MongoDB, le service retourne le code d’erreur 16500 lors de la limitation.
* Répliquez la base de données Cosmos DB entre deux ou plusieurs régions. Tous les réplicas sont lisibles. À l’aide des kits de développement logiciel, spécifiez le paramètre `PreferredLocations`. Il s’agit d’une liste ordonnée des régions Azure. Toutes les lectures sont envoyées vers la première région disponible dans la liste. Si la requête échoue, le client va essayer les autres régions dans la liste, dans l’ordre. Pour en savoir plus, voir [Comment configurer la distribution mondiale Azure Cosmos DB à l’aide de l’API SQL][cosmosdb-multi-region].

**Diagnostics**. Journalisation de toutes les erreurs côté client.

### <a name="writing-data-fails"></a>Échec de l’écriture des données.
**Détection**. Interceptez `System.Net.Http.HttpRequestException` ou `Microsoft.Azure.Documents.DocumentClientException`.

**Récupération**

* Le kit de développement logiciel retente automatiquement les tentatives ayant échoué. Pour définir le nombre de nouvelles tentatives et le délai d’attente maximal, configurez `ConnectionPolicy.RetryOptions`. Les exceptions déclenchées par le client sont au-delà de la stratégie de nouvelle tentative ou ne sont pas des erreurs temporaires.
* Si Cosmos DB limite le client, il renvoie une erreur HTTP 429. Vérifiez le code d’état dans `DocumentClientException`. Si l’erreur 429 se produit régulièrement, envisagez d’augmenter la valeur de débit de la collection.
* Répliquez la base de données Cosmos DB entre deux ou plusieurs régions. Si la région principale échoue, une autre région sera promue pour l’écriture. Vous pouvez également déclencher le basculement manuellement. Le kit de développement logiciel effectue une découverte et un routage automatique, le code de l’application continue donc de fonctionner après un basculement. Pendant la période de basculement (généralement quelques minutes), les opérations d’écriture ont une latence plus élevée, étant donné que le kit de développement logiciel est en train de trouver la nouvelle zone d’écriture.
  Pour en savoir plus, voir [Comment configurer la distribution mondiale Azure Cosmos DB à l’aide de l’API SQL][cosmosdb-multi-region].
* Comme solution de secours, conservez le document dans une file d’attente de sauvegarde et traitez cette file plus tard.

**Diagnostics**. Journalisation de toutes les erreurs côté client.

## <a name="elasticsearch"></a>Elasticsearch
### <a name="reading-data-from-elasticsearch-fails"></a>La lecture des données à partir de Elasticsearch échoue.
**Détection**. Interceptez l’exception appropriée pour le [client Elasticsearch][elasticsearch-client] utilisé.

**Récupération**

* Utilisez le mécanisme de nouvelle tentative. Chaque client possède ses propres stratégies de nouvelle tentative.
* Déployez plusieurs nœuds Elasticsearch et utilisez la réplication pour obtenir une haute disponibilité.

Pour plus d’informations, voir [Exécution de Elasticsearch sur Azure][elasticsearch-azure].

**Diagnostics**. Vous pouvez utiliser les outils de surveillance pour Elasticsearch ou enregistrer toutes les erreurs du côté client avec la charge utile. Consultez la section « Surveillance » dans [Exécution de Elasticsearch sur Azure][elasticsearch-azure].

### <a name="writing-data-to-elasticsearch-fails"></a>L’écriture des données sur Elasticsearch échoue.
**Détection**. Interceptez l’exception appropriée pour le [client Elasticsearch][elasticsearch-client] utilisé.  

**Récupération**

* Utilisez le mécanisme de nouvelle tentative. Chaque client possède ses propres stratégies de nouvelle tentative.
* Si l’application peut tolérer un niveau de cohérence réduit, envisagez l’écriture avec le paramètre `write_consistency` de `quorum`.

Pour plus d’informations, voir [Exécution de Elasticsearch sur Azure][elasticsearch-azure].

**Diagnostics**. Vous pouvez utiliser les outils de surveillance pour Elasticsearch ou enregistrer toutes les erreurs du côté client avec la charge utile. Consultez la section « Surveillance » dans [Exécution de Elasticsearch sur Azure][elasticsearch-azure].

## <a name="queue-storage"></a>Stockage de files d'attente
### <a name="writing-a-message-to-azure-queue-storage-fails-consistently"></a>L’écriture d’un message sur le Stockage File d’attente Azure échoue régulièrement.
**Détection**. Après *N* nouvelles tentatives, l’opération d’écriture échoue encore.

**Récupération**

* Stockez les données dans un cache local et transférez les écritures dans le stockage plus tard, lorsque le service devient disponible.
* Créez une file d’attente secondaire et écrivez dans cette file d’attente si la file d’attente principale n’est pas disponible.

**Diagnostics**. Utilisez les [métriques de stockage][storage-metrics].

### <a name="the-application-cannot-process-a-particular-message-from-the-queue"></a>L’application ne peut pas traiter un message particulier à partir de la file d’attente.
**Détection**. Spécifique à l’application. Par exemple, le message contient des données non valides, ou la logique métier échoue pour une raison quelconque.

**Récupération**

Déplacez le message vers une file d’attente distincte. Exécutez un processus séparé pour examiner les messages dans cette file d’attente.

Considérez l’utilisation des files d’attente de messagerie Azure Service Bus, fournissant une fonctionnalité de [file d’attente de lettres mortes][sb-dead-letter-queue] à cet effet.

> [!NOTE]
> Si vous utilisez des files d’attente de stockage avec WebJobs, le kit de développement logiciel WebJobs fournit la gestion intégrée des messages incohérents. Voir [Utilisation du stockage de file d’attente Azure avec le Kit de développement logiciel (SDK) WebJobs][sb-poison-message].

**Diagnostics**. Utilisez la journalisation des applications.

## <a name="redis-cache"></a>Cache Redis
### <a name="reading-from-the-cache-fails"></a>La lecture à partir du cache échoue.
**Détection**. Interceptez `StackExchange.Redis.RedisConnectionException`.

**Récupération**

1. Relance en cas d’échecs temporaires. Cache Redis Azure prend en charge l’intégration des nouvelles tentatives Voir [Instructions des nouvelles tentatives de Cache Redis][redis-retry].
2. Traitez les échecs non temporaires comme une absence dans le cache et revenez à la source de données d’origine.

**Diagnostics**. Utilisez [Diagnostics de Cache Redis][redis-monitor].

### <a name="writing-to-the-cache-fails"></a>L’écriture dans le cache échoue.
**Détection**. Interceptez `StackExchange.Redis.RedisConnectionException`.

**Récupération**

1. Relance en cas d’échecs temporaires. Cache Redis Azure prend en charge l’intégration des nouvelles tentatives Voir [Instructions des nouvelles tentatives de Cache Redis][redis-retry].
2. Si l’erreur est non temporaire, ignorez-la et laissez les autres transactions écrire ultérieurement dans le cache.

**Diagnostics**. Utilisez [Diagnostics de Cache Redis][redis-monitor].

## <a name="sql-database"></a>Base de données SQL
### <a name="cannot-connect-to-the-database-in-the-primary-region"></a>Impossible de se connecter à la base de données dans la région principale.
**Détection**. La connexion échoue.

**Récupération**

Condition préalable : la base de données doit être configurée pour la géo-réplication active. Voir [Géo-réplication active de SQL Database][sql-db-replication].

* Pour les requêtes, lisez à partir d’un réplica secondaire.
* Pour les insertions et les mises à jour, basculez manuellement vers un réplica secondaire. Voir [Lancer un basculement planifié ou non planifié pour une Azure SQL Database][sql-db-failover].

Le réplica utilise une chaîne de connexion différente, vous devrez donc la mettre à jour dans votre application.

### <a name="client-runs-out-of-connections-in-the-connection-pool"></a>Client à court de connexions dans le pool de connexions.
**Détection**. Interceptez les erreurs `System.InvalidOperationException`.

**Récupération**

* Retentez l’opération.
* En tant qu’un plan d’atténuation, isolez les pools de connexions pour chaque cas d’usage, de sorte qu’un seul cas d’usage ne puisse pas dominer toutes les connexions.
* Augmentez les pools de connexions maximales.

**Diagnostics**. Journaux d’application.

### <a name="database-connection-limit-is-reached"></a>La limite de connexion de base de données est atteinte.
**Détection**. Azure SQL Database limite le nombre de rôles de travail, de connexions et de sessions simultanés. Les limites dépendent du niveau de service. Pour en savoir plus, voir [Limites de ressources Microsoft Azure SQL Database][sql-db-limits].

Pour détecter ces erreurs, interceptez `System.Data.SqlClient.SqlException` et vérifiez la valeur de `SqlException.Number` pour le code d’erreur SQL. Pour connaître les codes d’erreur concernés, consultez la page [Codes d’erreur SQL pour les applications clientes SQL Database : erreur de connexion à la base de données et autres problèmes][sql-db-errors].

**Récupération**. Ces erreurs sont considérées comme transitoires, une nouvelle tentative peut donc résoudre le problème. Si vous rencontrez régulièrement ces erreurs, envisagez une mise à l’échelle de la base de données.

**Diagnostics**. - La requête [sys.event_log][sys.event_log] renvoie les interblocages, les échecs de connexion et les réussites de connexion de base de données.

* Créer une [règle d’alerte][azure-alerts] pour les connexions qui ont échoué.
* Activez [l’audit de SQL Database][sql-db-audit] et recherchez les échecs de connexion.

## <a name="service-bus-messaging"></a>Messagerie Service Bus 
### <a name="reading-a-message-from-a-service-bus-queue-fails"></a>Échec de la lecture d’un message à partir d’une file d’attente Service Bus.
**Détection**. Interceptez les exceptions à partir du kit de développement logiciel client. La classe de base pour les exceptions Service Bus est [MessagingException][sb-messagingexception-class]. Si l’erreur est temporaire, la propriété `IsTransient` a la valeur True.

Pour plus d’informations, consultez [Exceptions de la messagerie Service Bus][sb-messaging-exceptions].

**Récupération**

1. Relance en cas d’échecs temporaires. Voir [Instructions relatives aux nouvelles tentatives de Service Bus][sb-retry].
2. Les messages ne pouvant être remis au récepteur sont placés dans une *file d’attente de lettres mortes*. Utilisez cette file d’attente pour voir les messages non reçus. Notez que la file d’attente de lettres mortes n’est pas nettoyée automatiquement. Les messages y restent jusqu'à ce que vous les récupériez. Voir [Vue d’ensemble des files d’attente de lettres mortes Service Bus][sb-dead-letter-queue].

### <a name="writing-a-message-to-a-service-bus-queue-fails"></a>Échec de l’écriture d’un message vers une file d’attente Service Bus.
**Détection**. Interceptez les exceptions à partir du kit de développement logiciel client. La classe de base pour les exceptions Service Bus est [MessagingException][sb-messagingexception-class]. Si l’erreur est temporaire, la propriété `IsTransient` a la valeur True.

Pour plus d’informations, consultez [Exceptions de la messagerie Service Bus][sb-messaging-exceptions].

**Récupération**

1. Le client Service Bus fait automatiquement une nouvelle tentative après les erreurs transitoires. Par défaut, il utilise une temporisation exponentielle. Après avoir atteint le nombre de tentatives ou le délai d’attente maximal, le client lève une exception. Pour plus d’informations, consultez [Instructions relatives aux nouvelles tentatives Service Bus][sb-retry].
2. Si le quota de file d’attente est dépassé, le client lève [QuotaExceededException][QuotaExceededException]. Le message de l'exception donne plus d'informations. Retirez des messages de la file d’attente avant une nouvelle tentative et envisagez l’utilisation du modèle disjoncteur pour éviter les tentatives continues lorsque le quota est dépassé. En outre, assurez-vous que la propriété [BrokeredMessage.TimeToLive] n’est pas définie sur une valeur trop élevée.
3. Dans une région, la résilience peut être améliorée en utilisant des [files d’attente ou rubriques partitionnées][sb-partition]. Une file d’attente ou une rubrique non partitionnée est affectée à une banque de messagerie. Si cette banque de messagerie n’est pas disponible, toutes les opérations sur cette file d’attente ou rubrique échoueront. Une file d’attente ou une rubrique partitionnée est partitionnée entre plusieurs banques de messagerie.
4. Pour assurer une meilleure résilience, créez deux espaces de noms Service Bus dans différentes régions et répliquez les messages. Vous pouvez utiliser une réplication active ou passive.

   * Réplication active : le client envoie tous les messages à chaque file d’attente. Le récepteur écoute chaque file d’attente. Les messages ont une balise avec un identificateur unique, afin que le client puisse ignorer les messages en double.
   * Réplication passive : le client envoie les messages à une seule file d’attente. En cas d’erreur, le client effectue un envoi vers une autre file d’attente. Le récepteur écoute chaque file d’attente. Cette approche réduit le nombre de messages en double envoyés. Toutefois, le récepteur doit toujours gérer les messages en double.

     Pour plus d’informations, consultez la section [Exemple de géoréplication][sb-georeplication-sample] et [Meilleures pratiques pour protéger les applications contre les pannes de Service Bus et les sinistres](/azure/service-bus-messaging/service-bus-outages-disasters/).

### <a name="duplicate-message"></a>Messages en double.
**Détection**. Examinez les propriété `MessageId` et `DeliveryCount` du message.

**Récupération**

* Si possible, concevez vos opérations de traitement des messages de sorte qu’elles soient idempotentes. Dans le cas contraire, stockez les ID des messages déjà traités et vérifiez l’ID avant de traiter un message.
* Activez la détection des doublons, en créant la file d’attente avec `RequiresDuplicateDetection` défini sur True. Avec ce paramètre, Service Bus supprime automatiquement n’importe quel message envoyé avec le même `MessageId` qu’un message précédent.  Notez les points suivants :

  * Ce paramètre empêche les messages en double d’être placés dans la file d’attente. Il n’empêche pas un récepteur de traiter le même message plusieurs fois.
  * La détection des doublons est dotée d’une fenêtre de temps. Si un doublon est envoyé au-delà de cette fenêtre, il ne sera pas détecté.

**Diagnostics**. Journalisation des messages en double.

### <a name="the-application-cannot-process-a-particular-message-from-the-queue"></a>L’application ne peut pas traiter un message particulier à partir de la file d’attente.
**Détection**. Spécifique à l’application. Par exemple, le message contient des données non valides, ou la logique métier échoue pour une raison quelconque.

**Récupération**

Il existe deux modes d’échec à prendre en compte.

* Le récepteur détecte l’échec. Dans un tel cas, déplacez le message vers la file d'attente de lettres mortes. Plus tard, exécutez un processus séparé pour examiner les messages dans cette file d’attente.
* Le récepteur échoue au milieu du traitement du message &mdash;, par exemple, en raison d’une exception non prise en charge. Pour gérer ce cas, utilisez le mode `PeekLock`. Dans ce mode, si le verrou expire, le message est rendu disponible aux autres récepteurs. Si le message dépasse le nombre maximal de diffusions ou la durée de vie, il est automatiquement déplacé vers la file d’attente de lettres mortes.

Pour plus d’informations, voir [Vue d’ensemble des files d’attente de lettres mortes Service Bus][sb-dead-letter-queue].

**Diagnostics**. Chaque fois que l’application déplace un message dans la file d’attente de lettres mortes, écrivez un événement dans les journaux des applications.

## <a name="service-fabric"></a>Service Fabric
### <a name="a-request-to-a-service-fails"></a>Une requête vers un service échoue.
**Détection**. Le service retourne une erreur.

**Récupération**

* Recherchez de nouveau un proxy (`ServiceProxy` ou `ActorProxy`) et appelez de nouveau la méthode de service/acteur.
* **Service avec état**. Wrappez les opérations sur des collections fiables dans une transaction. En cas d’erreur, la transaction sera restaurée. La requête, si elle est extraite d’une file d’attente, sera traitée à nouveau.
* **Service sans état**. Si le service conserve des données dans un magasin externe, toutes les opérations doivent être idempotentes.

**Diagnostics**. Journal des application

### <a name="service-fabric-node-is-shut-down"></a>Le nœud de Service Fabric est arrêté.
**Détection**. Un jeton d’annulation est passé à la méthode `RunAsync` du service. Service Fabric annule la tâche avant l’arrêt du nœud.

**Récupération**. Utilisez le jeton d’annulation pour détecter l’arrêt. Lorsque Service Fabric demande l’annulation, terminez votre travail et quittez `RunAsync` aussi rapidement que possible.

**Diagnostics**. Journaux d’application

## <a name="storage"></a>Stockage
### <a name="writing-data-to-azure-storage-fails"></a>L’écriture de données sur Stockage Azure échoue
**Détection**. Le client reçoit des erreurs lors de l’écriture.

**Récupération**

1. Recommencez l’opération pour récupérer après des défaillances temporaires. La [stratégie de nouvelles tentatives][Storage.RetryPolicies] dans le kit de développement logiciel client gère cela automatiquement.
2. Implémentez le modèle disjoncteur pour éviter une surcharge de stockage.
3. Si N tentatives échouent, exécutez une procédure de secours appropriée. Par exemple : 

   * Stockez les données dans un cache local et transférez les écritures dans le stockage plus tard, lorsque le service devient disponible.
   * Si l’action d’écriture se trouvait dans une étendue transactionnelle, compensez la transaction.

**Diagnostics**. Utilisez les [métriques de stockage][storage-metrics].

### <a name="reading-data-from-azure-storage-fails"></a>La lecture des données à partir de Stockage Azure échoue.
**Détection**. Le client reçoit des erreurs lors de la lecture.

**Récupération**

1. Recommencez l’opération pour récupérer après des défaillances temporaires. La [stratégie de nouvelles tentatives][Storage.RetryPolicies] dans le kit de développement logiciel client gère cela automatiquement.
2. Pour le stockage RA-GRS, si la lecture depuis le point de terminaison principal échoue, essayez de lire à partir du point de terminaison secondaire. Le kit de développement logiciel client permet de gérer cela automatiquement. Voir [Réplication Stockage Azure][storage-replication].
3. Si *N* nouvelles tentatives échouent, effectuez une action de secours pour une dégradation normale. Par exemple, si une image du produit ne peut pas être récupérée à partir du stockage, affichez une image générique d’espace réservé.

**Diagnostics**. Utilisez les [métriques de stockage][storage-metrics].

## <a name="virtual-machine"></a>Machine virtuelle
### <a name="connection-to-a-backend-vm-fails"></a>Échec de la connexion à une machine virtuelle principale.
**Détection**. Erreurs de connexion réseau.

**Récupération**

* Déployez au moins deux machines virtuelles principales dans un groupe à haute disponibilité, derrière un équilibreur de charge.
* Si l’erreur de connexion est temporaire, il est possible que TCP arrive à envoyer le message correctement lors d’une nouvelle tentative.
* Implémentez une stratégie de nouvelles tentatives dans l’application.
* Pour les erreurs persistantes ou non temporaires, vous devez implémenter le modèle [disjoncteur][circuit-breaker].
* Si la machine virtuelle appelante dépasse sa limite de sortie réseau, la file d’attente sortante va saturer. Si la file d’attente sortante est constamment pleine, envisagez une montée en charge.

**Diagnostics**. Journalisation des événements au niveau des limites de service.

### <a name="vm-instance-becomes-unavailable-or-unhealthy"></a>Une instance de machine virtuelle est indisponible ou défectueuse.
**Détection**. Configurez une [sonde d’intégrité][lb-probe] de l’équilibreur de charge qui signale si l’instance de machine virtuelle est saine. La sonde devrait vérifier si les fonctions essentielles répondent correctement.

**Récupération**. Pour chaque couche d’application, placez plusieurs instances de machine virtuelle dans le même groupe à haute disponibilité et placez un équilibreur de charge devant les machines virtuelles. Si une sonde ne répond pas, l’équilibreur de charge n’envoie plus de nouvelles connexions à l’instance défectueuse.

**Diagnostics**. - Utilisez [l’analyse des journaux][lb-monitor] de l’équilibreur de charge.

* Configurer votre système de surveillance pour surveiller tous les points de terminaison d’analyse du fonctionnement.

### <a name="operator-accidentally-shuts-down-a-vm"></a>Un opérateur arrête accidentellement une machine virtuelle.
**Détection**. N/A

**Récupération**. Définissez un verrou de ressource avec un niveau `ReadOnly`. Consultez [Verrouiller des ressources avec Azure Resource Manager][rm-locks].

**Diagnostics**. Utilisez les [Journaux d’activité Azure][azure-activity-logs].

## <a name="webjobs"></a>WebJobs
### <a name="continuous-job-stops-running-when-the-scm-host-is-idle"></a>La tâche en continu s’arrête lorsque l’hôte SCM est inactif.
**Détection**. Passez un jeton d’annulation à la fonction WebJob. Pour plus d’informations, consultez [Arrêt correct][web-jobs-shutdown].

**Récupération**. Activez le paramètre `Always On` dans l’application web. Pour plus d’informations, consultez [Exécuter des tâches en arrière-plan avec Webjobs][web-jobs].

## <a name="application-design"></a>Conception des applications
### <a name="application-cant-handle-a-spike-in-incoming-requests"></a>L’application ne peut pas gérer un pic de requêtes entrantes.
**Détection**. Dépend de l’application. Symptômes courants :

* Le site Web démarre en renvoyant des codes d’erreur HTTP 5xx.
* Les services dépendants, tels que la base de données ou le stockage, commencent à limiter les requêtes. Recherchez des erreurs HTTP tels que HTTP 429 (trop de demandes), en fonction du service.
* La longueur de la file d’attente HTTP augmente.

**Récupération**

* Montez en charge pour traiter une charge accrue.
* Atténuez les échecs pour éviter que les pannes en cascade interrompent toute l’application. Les stratégies d’atténuation comprennent :

  * Implémentez le [Modèle de limitation][throttling-pattern] afin d’éviter de déborder les systèmes principaux.
  * Utilisez le [nivellement de la charge basé sur une file d’attente][queue-based-load-leveling] pour mettre les demandes en mémoire tampon et les traiter à un rythme approprié.
  * Classez certains clients par priorité. Par exemple, si l’application possède des niveaux gratuits et payants, limitez les clients disposant du niveau gratuit, mais pas ceux disposant d’un niveau payant. Voir [Modèle de file d’attente de priorité][priority-queue-pattern].

**Diagnostics**. Utilisez la [Journalisation des diagnostics de App Service][app-service-logging]. Utilisez un service tel que [Azure Log Analytics][azure-log-analytics], [Application Insights][app-insights], ou [New Relic] [ new-relic] pour aider à comprendre les journaux de diagnostic.

### <a name="one-of-the-operations-in-a-workflow-or-distributed-transaction-fails"></a>L’une des opérations d’un flux de travail ou d’une transaction distribuée a échoué.
**Détection**. Après *N* nouvelles tentatives, elle échoue toujours.

**Récupération**

* En tant qu’un plan d’atténuation, implémentez le modèle [Superviseur de l’agent du planificateur][scheduler-agent-supervisor] pour gérer l’intégralité du workflow.
* N’effectuez pas de nouvelles tentatives après un délai d’expiration. Cette erreur a un faible taux de réussite.
* La file d’attente fonctionne, pour réessayer ultérieurement.

**Diagnostics**. Journalisation de toutes les opérations (réussies et échouées), y compris les actions de compensation. Utilisez les ID de corrélation pour suivre toutes les opérations situées dans une même transaction.

### <a name="a-call-to-a-remote-service-fails"></a>Un appel vers un service distant a échoué.
**Détection**. Code d'erreur HTTP.

**Récupération**

1. Relance en cas d’échecs temporaires.
2. Si l’appel échoue après *N* tentatives, effectuez une action de secours. (Spécifique à l’application.)
3. Implémentez le [modèle disjoncteur][circuit-breaker] pour éviter des erreurs en cascade.

**Diagnostics**. Journalisation de tous les échecs d’appel distant.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur le processus FMA, consultez [Résilience par la conception pour les services de cloud computing][resilience-by-design-pdf] (téléchargement PDF).

<!-- links -->

[api-management]: https://azure.microsoft.com/documentation/services/api-management/
[api-management-throttling]: /azure/api-management/api-management-sample-flexible-throttling/
[app-insights]: /azure/application-insights/app-insights-overview/
[app-insights-web-apps]: /azure/application-insights/app-insights-azure-web-apps/
[app-service-configure]: /azure/app-service-web/web-sites-configure/
[app-service-logging]: /azure/app-service-web/web-sites-enable-diagnostic-log/
[app-service-slots]: /azure/app-service-web/web-sites-staged-publishing/
[auto-rest-client-retry]: https://github.com/Azure/autorest/tree/master/docs
[azure-activity-logs]: /azure/monitoring-and-diagnostics/monitoring-overview-activity-logs/
[azure-alerts]: /azure/monitoring-and-diagnostics/insights-alerts-portal/
[azure-log-analytics]: /azure/log-analytics/log-analytics-overview/
[BrokeredMessage.TimeToLive]: https://msdn.microsoft.com/library/microsoft.servicebus.messaging.brokeredmessage.timetolive.aspx
[cassandra-error-handling]: http://www.datastax.com/dev/blog/cassandra-error-handling-done-right
[circuit-breaker]: https://msdn.microsoft.com/library/dn589784.aspx
[cosmosdb-multi-region]: /azure/cosmos-db/tutorial-global-distribution-sql-api
[elasticsearch-azure]: ../elasticsearch/index.md
[elasticsearch-client]: https://www.elastic.co/guide/en/elasticsearch/client/index.html
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[onstop-events]: https://azure.microsoft.com/blog/the-right-way-to-handle-azure-onstop-events/
[lb-monitor]: /azure/load-balancer/load-balancer-monitor-log/
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview/#learn-about-the-types-of-probes
[new-relic]: https://newrelic.com/
[priority-queue-pattern]: https://msdn.microsoft.com/library/dn589794.aspx
[queue-based-load-leveling]: https://msdn.microsoft.com/library/dn589783.aspx
[QuotaExceededException]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.quotaexceededexception.aspx
[ra-web-apps-basic]: ../reference-architectures/app-service-web-app/basic-web-app.md
[redis-monitor]: /azure/redis-cache/cache-how-to-monitor/
[redis-retry]: ../best-practices/retry-service-specific.md#azure-redis-cache
[resilience-by-design-pdf]: http://download.microsoft.com/download/D/8/C/D8C599A4-4E8A-49BF-80EE-FE35F49B914D/Resilience_by_Design_for_Cloud_Services_White_Paper.pdf
[RoleEntryPoint.OnStop]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.onstop.aspx
[RoleEnvironment.Stopping]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.stopping.aspx
[rm-locks]: /azure/azure-resource-manager/resource-group-lock-resources/
[sb-dead-letter-queue]: /azure/service-bus-messaging/service-bus-dead-letter-queues/
[sb-georeplication-sample]: https://github.com/Azure-Samples/azure-servicebus-messaging-samples/tree/master/GeoReplication
[sb-messagingexception-class]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.messagingexception.aspx
[sb-messaging-exceptions]: /azure/service-bus-messaging/service-bus-messaging-exceptions/
[sb-outages]: /azure/service-bus-messaging/service-bus-outages-disasters/#protecting-queues-and-topics-against-datacenter-outages-or-disasters
[sb-partition]: /azure/service-bus-messaging/service-bus-partitioning/
[sb-poison-message]: /azure/app-service-web/websites-dotnet-webjobs-sdk-storage-queues-how-to/#poison
[sb-retry]: ../best-practices/retry-service-specific.md#service-bus
[search-sdk]: https://msdn.microsoft.com/library/dn951165.aspx
[scheduler-agent-supervisor]: https://msdn.microsoft.com/library/dn589780.aspx
[search-analytics]: /azure/search/search-traffic-analytics/
[sql-db-audit]: /azure/sql-database/sql-database-auditing-get-started/
[sql-db-errors]: /azure/sql-database/sql-database-develop-error-messages/#resource-governance-errors
[sql-db-failover]: /azure/sql-database/sql-database-geo-replication-failover-portal/
[sql-db-limits]: /azure/sql-database/sql-database-resource-limits/
[sql-db-replication]: /azure/sql-database/sql-database-geo-replication-overview/
[storage-metrics]: https://msdn.microsoft.com/library/dn782843.aspx
[storage-replication]: /azure/storage/storage-redundancy/
[Storage.RetryPolicies]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.aspx
[sys.event_log]: https://msdn.microsoft.com/library/dn270018.aspx
[throttling-pattern]: https://msdn.microsoft.com/library/dn589798.aspx
[web-jobs]: /azure/app-service-web/web-sites-create-web-jobs/
[web-jobs-shutdown]: /azure/app-service-web/websites-dotnet-webjobs-sdk-storage-queues-how-to/#graceful
