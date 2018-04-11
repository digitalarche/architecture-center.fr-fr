---
title: Absence d’antimodèle de mise en cache
description: Extraire plusieurs fois les mêmes données peut réduire les performances et l’évolutivité.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 8a2bc3b473a30536cc1bef9e1dcad87acb46c4a9
ms.sourcegitcommit: c93f1b210b3deff17cc969fb66133bc6399cfd10
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/05/2018
---
# <a name="no-caching-antipattern"></a>Absence d’antimodèle de mise en cache

Dans une application cloud gérant de nombreuses requêtes simultanées, l’extraction répétée de mêmes données peut réduire l’évolutivité et les performances. 

## <a name="problem-description"></a>Description du problème

Un certain nombre de comportements indésirables peuvent survenir lorsque les données ne sont pas mises en cache, notamment :

- L’extraction répétée des mêmes informations à partir d’une ressource coûteuse pour l’accès en termes de latence ou de surcharge d’E/S.
- La construction répétée des mêmes objets ou structures de données pour plusieurs requêtes.
- Nombre excessif d’appels à un service distant disposant d’un quota de service et qui limite les clients après une certaine limite.

Ces problèmes peuvent à leur tour entraîner un allongement du temps de réponse, une contention accrue dans le magasin de données et une faible évolutivité.

L’exemple suivant utilise Entity Framework pour se connecter à une base de données. Chaque requête du client entraîne un appel à la base de données, même si plusieurs requêtes extraient exactement les mêmes données. Le coût des requêtes répétées, en termes de frais de surcharge d’E/S et d’accès aux données, peut rapidement augmenter.

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

Vous trouverez l’exemple complet [ici][sample-app].

Cet antimodèle survient généralement pour les raisons suivantes :

- Le fait de ne pas utiliser de cache est plus simple à implémenter et fonctionne bien en cas de faible charge. La mise en cache rend le code plus complexe. 
- Les avantages et les inconvénients liés à l’utilisation d’un cache ne sont pas clairement compris.
- Un problème se pose quant à la surcharge liée à la gestion de l’exactitude et de l’actualisation des données mises en cache.
- Une application a été migrée depuis un système local, où la latence du réseau n’était pas un problème. De plus, le système s’exécutait sur un matériel coûteux très performant, si bien que la mise en cache n’a pas été prise en compte dans de la conception d’origine.
- Les développeurs ne savent pas que la mise en cache est une possibilité dans un scénario donné. Par exemple, les développeurs ne pensent pas à utiliser les ETag lors de l’implémentation d’une API web.

## <a name="how-to-fix-the-problem"></a>Comment corriger le problème

La stratégie de mise en cache la plus populaire est la stratégie *à la demande* ou *cache-aside*.

- Lors de la lecture, l’application tente de lire les données à partir du cache. Si les données ne sont pas dans le cache, l’application les récupère à partir de la source de données et les ajoute au cache.
- Lors de l’écriture, l’application écrit la modification directement dans la source de données et supprime l’ancienne valeur du cache. Elle est récupérée et ajoutée au cache dès qu’elle est nécessaire.

Cette approche est appropriée pour les données changeant fréquemment. Voici l’exemple précédent mis à jour pour utiliser le modèle [Cache-Aside][cache-aside].  

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

Notez que la méthode `GetAsync` appelle désormais la classe `CacheService` au lieu d’appeler directement la base de données. La classe `CacheService` tente d’abord d’obtenir l’élément à partir du Cache Redis Microsoft Azure. Si la valeur est introuvable dans le Cache Redis, la méthode `CacheService` appelle une fonction lambda qui lui a été transmise par l’appelant. La fonction lambda est responsable de la récupération des données à partir de la base de données. Cette implémentation dissocie le référentiel à partir de la solution de mise en cache spécifique et l’élément `CacheService` de la base de données. 

## <a name="considerations"></a>Considérations

- Si le cache n’est pas disponible, peut-être en raison d’une défaillance passagère, il ne renvoie aucune erreur au client. Au lieu de cela, il extrait les données de la source de données d’origine. Toutefois, prenez en compte le fait que lors de la récupération du cache, le magasin d’origine peut être submergé de requêtes, ce qui entraîne des délais d’attente et des échecs de connexion. Après tout, ceci est une des raisons pour lesquelles on utilise un cache en premier lieu. Utilisez une technique telle que le [modèle de disjoncteur] [ circuit-breaker] pour éviter de surcharger la source de données.

- Les applications qui mettent en cache les données non statiques doivent être conçues pour prendre en charge une cohérence éventuelle.

- Pour les API web, vous pouvez prendre en charge la mise en cache côté client, y compris un en-tête Cache-Control dans les messages de requête et de réponse, et l’utilisation des ETags pour identifier les versions des objets. Pour plus d’informations, consultez [API implementation][api-implementation] (Implémentation de l’API).

- Vous n’êtes pas obligé de mettre en cache des entités complètes. Si la majeure partie d’une entité est statique, et que seule une petite partie est modifiée fréquemment, mettez en cache les éléments statiques et récupérez les éléments dynamiques de la source de données. Cette approche peut contribuer à réduire le volume d’E/S en cours d’exécution par rapport à la source de données.

- Dans certains cas, si des données volatiles sont de courte durée, il peut être utile de les mettre en cache. Prenons par exemple un appareil qui envoie continuellement des mises à jour. Il peut être judicieux de mettre en cache ces informations dès leur arrivée et de ne pas les écrire dans un magasin persistant.  

- Pour empêcher les données de devenir obsolètes, de nombreuses solutions de mise en cache prennent en charge des périodes d’expiration configurables, afin que les données soient supprimées automatiquement du cache après un intervalle spécifique. Vous devrez peut-être régler l’heure d’expiration de votre scénario. Les données hautement statiques peuvent rester dans le cache plus longtemps que les données volatiles qui deviennent rapidement obsolètes.

- Si la solution de mise en cache ne propose pas d’expiration intégrée, vous devrez peut-être implémenter un processus en arrière-plan qui balaie occasionnellement le cache et l’empêche de croître sans limite. 

- Outre la mise en cache des données à partir d’une source de données externe, vous pouvez utiliser la mise en cache pour enregistrer les résultats de calculs complexes. Avant cela, il convient toutefois d’instrumenter l’application pour déterminer si elle est réellement liée à l’UC.

- Il peut être utile d’amorcer le cache au démarrage de l’application. Alimentez le cache avec les données les plus susceptibles d’être utilisées.

- Incluez toujours une instrumentation qui détecte les correspondances et les absences dans le cache. Ces informations permettent de paramétrer des stratégies de mise en cache, comme pour définir les données à mettre en cache et la durée de maintien en cache avant leur expiration.

- Si l’absence de mise en cache entraîne un goulot d’étranglement, l’ajout de mise en cache peut augmenter le volume de requêtes au point de surcharger le serveur web frontal. Les clients peuvent commencer à recevoir des erreurs HTTP 503 (Service indisponible). Cela indique que vous devez augmenter la taille des instances du serveur frontal.

## <a name="how-to-detect-the-problem"></a>Comment détecter le problème

Vous pouvez effectuer les étapes suivantes afin de déterminer si l’absence de mise en cache est à l’origine des problèmes de performances :

1. Vérifiez la conception de l’application. Effectuez un inventaire de tous les magasins de données utilisés par l’application. Pour chacun, déterminez si l’application utilise un cache. Si possible, déterminez la fréquence à laquelle les données sont modifiées. Les données qui évoluent lentement et les données de référence statiques fréquemment lues sont dans un premier temps de bonnes candidates à la mise en cache. 

2. Instrumentez l’application et analysez le système pour déterminer la fréquence à laquelle l’application récupère des données ou calcule des informations.

3. Profilez l’application dans un environnement de test pour capturer des métriques de bas niveau concernant la surcharge associée aux opérations d’accès aux données ou aux calculs effectués fréquemment.

4. Effectuez un test de charge dans un environnement de test pour identifier la manière dont le système répond sous une charge de travail normale et sous une charge importante. Le test de charge doit simuler le modèle d’accès aux données observé dans l’environnement de production à l’aide de charges de travail réalistes. 

5. Examinez les statistiques d’accès aux données pour les magasins de données sous-jacents et vérifiez la fréquence à laquelle les mêmes requêtes de données sont répétées. 


## <a name="example-diagnosis"></a>Exemple de diagnostic

Les sections suivantes appliquent ces étapes à l’exemple d’application décrit précédemment.

### <a name="instrument-the-application-and-monitor-the-live-system"></a>Instrumenter l’application et analyser le système dynamique

Instrumentez l’application et analysez-la pour obtenir des informations sur les requêtes spécifiques effectuées par les utilisateurs lorsque l’application est en production. 

L’illustration suivante montre les données d’analyse capturées par [New Relic] [ NewRelic] pendant un test de charge. Dans ce cas, la seule opération HTTP GET effectuée est `Person/GetAsync`. Toutefois, dans un environnement de production dynamique, le fait de connaître la fréquence relative à laquelle chaque requête est effectuée peut vous donner un aperçu des ressources à mettre en cache.

![New Relic montrant des requêtes de serveur pour l’application CachingDemo][NewRelic-server-requests]

Si vous avez besoin d’une analyse plus approfondie, vous pouvez utiliser un générateur de profils pour capturer les données dont les performances sont faibles dans un environnement de test (et non le système de production). Examinez les métriques, telles que le taux de demandes d’E/S, l’utilisation de la mémoire et du processeur. Ces dernières peuvent montrer un grand nombre de requêtes vers un magasin de données ou un service ou un traitement répété qui effectue le même calcul. 

### <a name="load-test-the-application"></a>Effectuer un test de charge de l’application

Le graphique suivant montre les résultats du test de charge de l’exemple d’application. Le test de charge simule une charge par étape pouvant atteindre 800 utilisateurs effectuant une série classique d’opérations. 

![Résultats du test de charge portant sur les performances pour le scénario de non mise en cache][Performance-Load-Test-Results-Uncached]

Le nombre de tests réussis exécutés chaque seconde se stabilise et les requêtes supplémentaires sont ralenties en conséquence. La durée moyenne du test augmente régulièrement avec la charge de travail. Le temps de réponse se stabilise dès que la charge utilisateur atteint un pic.

### <a name="examine-data-access-statistics"></a>Examiner les statistiques d’accès aux données

Les statistiques d’accès aux données et les autres informations fournies par un magasin de données peuvent donner des informations utiles, comme savoir quelles sont les requêtes les plus fréquemment répétées. Par exemple, dans Microsoft SQL Server, la vue de gestion `sys.dm_exec_query_stats` comporte des informations statistiques pour les requêtes récemment exécutées. Le texte de chaque requête est disponible dans la vue `sys.dm_exec-query_plan`. Vous pouvez utiliser un outil tel que SQL Server Management Studio pour exécuter la requête SQL suivante et déterminer la fréquence à laquelle les requêtes sont exécutées.

```SQL
SELECT UseCounts, Text, Query_Plan
FROM sys.dm_exec_cached_plans
CROSS APPLY sys.dm_exec_sql_text(plan_handle)
CROSS APPLY sys.dm_exec_query_plan(plan_handle)
```

La colonne `UseCount` dans les résultats indique la fréquence d’exécution de chaque requête. L’illustration suivante montre que la troisième requête a été exécutée plus de 250 000 fois, ce qui est beaucoup plus que toute autre requête.

![Résultats de l’interrogation des vues de gestion dynamique dans SQL Server Management Server][Dynamic-Management-Views]

Voici la requête SQL à l’origine des si nombreuses requêtes de base de données : 

```SQL
(@p__linq__0 int)SELECT TOP (2)
[Extent1].[BusinessEntityId] AS [BusinessEntityId],
[Extent1].[FirstName] AS [FirstName],
[Extent1].[LastName] AS [LastName]
FROM [Person].[Person] AS [Extent1]
WHERE [Extent1].[BusinessEntityId] = @p__linq__0
```

Il s’agit de la requête qu’Entity Framework génère dans la méthode `GetByIdAsync` illustrée précédemment.

### <a name="implement-the-solution-and-verify-the-result"></a>Implémenter la solution et vérifier le résultat

Après avoir intégré un cache, répétez les tests de charge et comparez les résultats avec les tests de charge précédents sans un cache. Voici les résultats du test de charge après l’ajout d’un cache à l’exemple d’application.

![Résultats du test de charge portant sur les performances pour le scénario de mise en cache][Performance-Load-Test-Results-Cached]

Le volume de tests réussis se stabilise, mais à une charge utilisateur plus élevée. Le taux de requêtes à cette charge est beaucoup plus important que le précédent. La durée moyenne du test augmente sans cesse avec la charge, toutefois le temps de réponse maximal est de 0,05 ms, comparé à 1 ms précédemment &mdash;, soit une amélioration de 20&times;. 

## <a name="related-resources"></a>Ressources associées

- [Meilleures pratiques de mise en œuvre des API][api-implementation]
- [Modèle de type cache-Aside][cache-aside-pattern]
- [Meilleures pratiques en matière de mise en cache][caching-guidance]
- [Modèle Disjoncteur][circuit-breaker]

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/NoCaching
[cache-aside-pattern]: /azure/architecture/patterns/cache-aside
[caching-guidance]: ../../best-practices/caching.md
[circuit-breaker]: ../../patterns/circuit-breaker.md
[api-implementation]: ../../best-practices/api-implementation.md#optimizing-client-side-data-access
[NewRelic]: http://newrelic.com/azure
[NewRelic-server-requests]: _images/New-Relic.jpg
[Performance-Load-Test-Results-Uncached]:_images/InitialLoadTestResults.jpg
[Dynamic-Management-Views]: _images/SQLServerManagementStudio.jpg
[Performance-Load-Test-Results-Cached]: _images/CachedLoadTestResults.jpg