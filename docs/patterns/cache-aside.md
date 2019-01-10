---
title: Modèle Cache-Aside
titleSuffix: Cloud Design Patterns
description: Chargez les données à la demande dans un cache à partir d’une banque de données.
keywords: modèle de conception
author: dragon119
ms.date: 11/01/2018
ms.custom: seodec18
ms.openlocfilehash: 96dee3ca766414a3a17ea161f13c9fcd15001b4d
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/08/2019
ms.locfileid: "54114162"
---
# <a name="cache-aside-pattern"></a>Modèle Cache-Aside

[!INCLUDE [header](../_includes/header.md)]

Chargez les données à la demande dans un cache à partir d’une banque de données. Cela peut améliorer les performances et aider à maintenir la cohérence entre les données contenues dans le cache et les données résidant dans la banque de données sous-jacente.

## <a name="context-and-problem"></a>Contexte et problème

Les applications utilisent un cache pour optimiser l’accès répété aux informations contenues dans une banque de données. Toutefois, on peut difficilement s’attendre à ce que les données mises en cache soient toujours entièrement cohérentes avec les données contenues dans la banque de données. Les applications doivent implémenter une stratégie pour s’assurer que les données contenues dans le cache soient le plus à jour possible, mais également pour identifier et gérer les situations dans lesquelles les données du cache sont devenues obsolètes.

## <a name="solution"></a>Solution

De nombreux systèmes de mise en cache disponibles sur le marché intègrent des opérations de double lecture et de double écriture/d’écriture différée. Dans ces systèmes, une application récupère des données en établissant une référence au cache. Si les données ne se trouvent pas dans le cache, elles sont récupérées dans la banque de données et ajoutées au cache. Les modifications apportées aux données contenues dans le cache sont automatiquement répercutées dans la banque de données.

Pour les caches qui n’offrent pas cette fonctionnalité, il revient aux applications qui utilisent le cache d’assurer la mise à jour des données.

Une application peut émuler la fonctionnalité de mise en cache avec double lecture en implémentant la stratégie Cache-Aside. Cette stratégie charge des données dans le cache à la demande. La figure montre comment utiliser le modèle Cache-Aside pour stocker des données dans le cache.

![Utilisation du modèle Cache-Aside pour stocker des données dans le cache](./_images/cache-aside-diagram.png)

Si une application met à jour les informations, elle peut appliquer la stratégie de double écriture en répercutant la modification dans la banque de données et en invalidant l’élément correspondant dans le cache.

Lorsque l’application aura à nouveau besoin de cet élément, la stratégie Cache-Aside permettra de récupérer les données mises à jour dans la banque de données et de les ajouter au cache.

## <a name="issues-and-considerations"></a>Problèmes et considérations

Prenez en compte les points suivants lorsque vous choisissez comment implémenter ce modèle :

**Durée de vie des données mises en cache**. De nombreux caches intègrent une stratégie d’expiration qui invalide les données et les supprime du cache si elles n’ont pas été consultées pendant une certaine période. Pour que la stratégie Cache-Aside soit efficace, assurez-vous que la stratégie d’expiration correspond au modèle d’accès pour les applications qui utilisent les données. Ne configurez pas un délai d’expiration trop court, car les applications pourraient alors récupérer des données en continu dans la banque de données pour les ajouter au cache. De même, ne configurez pas un délai d’expiration trop long pour éviter que les données en cache deviennent obsolètes. N’oubliez pas que la mise en cache est particulièrement efficace pour des données relativement statiques ou des données qui sont fréquemment lues.

**Éviction des données**. La plupart des caches étant de plus petite taille que les banques de données d’où proviennent les données, ils n’ont parfois d’autre choix que de supprimer des données. La plupart des caches adoptent une stratégie qui consiste à supprimer les données les moins récemment utilisées. Il est néanmoins possible de personnaliser cette stratégie. Configurez la propriété d’expiration globale ainsi que d’autres propriétés du cache et la propriété d’expiration de chaque élément mis en cache pour limiter les coûts associés à la mise en cache. Il n’est pas toujours judicieux d’appliquer une stratégie d’éviction globale pour chaque élément présent dans le cache. Par exemple, si la récupération d’un élément mis en cache à partir de la banque de données s’avère très coûteuse, il peut être judicieux de conserver cet élément dans le cache au détriment d’autres éléments fréquemment lus mais moins coûteux.

**Amorçage du cache**. De nombreuses solutions préremplissent le cache avec des données dont une application est susceptible d’avoir besoin au moment du démarrage. Le modèle Cache-Aside peut toujours être utile si certaines de ces données expirent ou sont supprimées.

**Cohérence**. L’implémentation du modèle Cache-Aside ne garantit pas la cohérence entre la banque de données et le cache. Un élément de la banque de données peut être modifié à tout moment par un processus externe, et cette modification peut ne pas être répercutée dans le cache jusqu’au prochain chargement de l’élément. Dans un système qui réplique les données dans différentes banques de données, ce problème peut s’aggraver si la synchronisation intervient fréquemment.

**Mise en cache locale (en mémoire)**. Un cache peut être localement associé à une instance d’application et stocké en mémoire. Le modèle Cache-Aside peut s’avérer utile dans cet environnement si une application accède aux mêmes données de manière répétée. Toutefois, un cache local est privé, ce qui permet aux différentes instances d’application d’avoir chacune une copie des mêmes données en cache. Ces données peuvent rapidement devenir incohérentes entre les caches, et il peut être nécessaire de faire expirer les données conservées dans un cache privé et de les actualiser plus fréquemment. Dans ces scénarios, analysez les avantages d’utiliser un mécanisme de mise en cache partagé ou distribué.

## <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle

Utilisez ce modèle dans les situations suivantes :

- Un cache ne fournit pas d’opérations de double lecture et de double écriture natives.
- La demande en ressources n’est pas prévisible. Ce modèle permet aux applications de charger des données à la demande. Il n’émet aucune hypothèse sur les données dont aura besoin une application.

Ce modèle peut ne pas convenir :

- Lorsque le jeu de données mises en cache est statique. Si les données ne tiennent pas dans l’espace de cache disponible, amorcez le cache avec les données au démarrage et appliquez une stratégie qui empêche l’expiration des données.
- Pour mettre en cache les informations d’état de session dans une application web hébergée dans une batterie de serveurs web. Dans cet environnement, vous devez éviter d’introduire des dépendances basées sur l’affinité client-serveur.

## <a name="example"></a>Exemples

Dans Microsoft Azure, vous pouvez utiliser le Cache Redis Azure pour créer un cache distribué qui peut être partagé par plusieurs instances d’une application.

Les exemples de code suivants utilisent le client [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis). Il s’agit d’une bibliothèque de client Redis écrite pour .NET. Pour vous connecter à une instance de Cache Redis Azure, appelez la méthode statique `ConnectionMultiplexer.Connect` et transmettez la chaîne de connexion. La méthode renvoie un élément `ConnectionMultiplexer` qui représente la connexion. Pour partager une instance `ConnectionMultiplexer` dans votre application, vous pouvez utiliser une propriété statique qui renvoie une instance connectée, comme dans l’exemple suivant. Cette approche fournit une méthode thread-safe permettant d’initialiser une seule instance connectée.

```csharp
private static ConnectionMultiplexer Connection;

// Redis Connection string info
private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
{
    string cacheConnection = ConfigurationManager.AppSettings["CacheConnection"].ToString();
    return ConnectionMultiplexer.Connect(cacheConnection);
});

public static ConnectionMultiplexer Connection => lazyConnection.Value;
```

La méthode `GetMyEntityAsync` dans l’exemple de code suivant illustre une implémentation du modèle Cache-Aside. Cette méthode récupère un objet à partir du cache à l’aide de l’approche en double lecture.

Un objet est identifié en utilisant un identifiant entier comme clé. La méthode `GetMyEntityAsync` tente de récupérer un élément avec cette clé à partir du cache. Si un élément correspondant est trouvé, celui-ci est renvoyé. Si aucune correspondance n’est trouvée dans le cache, la méthode `GetMyEntityAsync` récupère l’objet à partir d’une banque de données, l’ajoute au cache, puis le renvoie. Le code qui lit les données de la banque de données n’est pas présenté ici, car il varie selon la banque de données. Notez que l’élément mis en cache est configuré pour expirer afin qu’il ne devienne pas obsolète dans le cas où il serait mis à jour à un autre endroit.

```csharp
// Set five minute expiration as a default
private const double DefaultExpirationTimeInMinutes = 5.0;

public async Task<MyEntity> GetMyEntityAsync(int id)
{
  // Define a unique key for this method and its parameters.
  var key = $"MyEntity:{id}";
  var cache = Connection.GetDatabase();

  // Try to get the entity from the cache.
  var json = await cache.StringGetAsync(key).ConfigureAwait(false);
  var value = string.IsNullOrWhiteSpace(json)
                ? default(MyEntity)
                : JsonConvert.DeserializeObject<MyEntity>(json);

  if (value == null) // Cache miss
  {
    // If there's a cache miss, get the entity from the original store and cache it.
    // Code has been omitted because it is data store dependent.
    value = ...;

    // Avoid caching a null value.
    if (value != null)
    {
      // Put the item in the cache with a custom expiration time that
      // depends on how critical it is to have stale data.
      await cache.StringSetAsync(key, JsonConvert.SerializeObject(value)).ConfigureAwait(false);
      await cache.KeyExpireAsync(key, TimeSpan.FromMinutes(DefaultExpirationTimeInMinutes)).ConfigureAwait(false);
    }
  }

  return value;
}
```

> Ces exemples utilisent Cache Redis pour accéder au magasin de données et récupérer des informations depuis le cache. Pour plus d’informations, consultez les articles [Using Microsoft Azure Redis Cache](https://docs.microsoft.com/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache) (Utilisation du Cache Redis Microsoft Azure) et [Création d’une application web avec le Cache Redis](https://docs.microsoft.com/azure/redis-cache/cache-web-app-howto).

La méthode `UpdateEntityAsync` illustrée ci-dessous montre comment invalider un objet dans le cache lorsque sa valeur est modifiée par l’application. Le code met à jour la banque de données d’origine, puis supprime l’élément en cache du cache.

```csharp
public async Task UpdateEntityAsync(MyEntity entity)
{
    // Update the object in the original data store.
    await this.store.UpdateEntityAsync(entity).ConfigureAwait(false);

    // Invalidate the current cache object.
    var cache = Connection.GetDatabase();
    var id = entity.Id;
    var key = $"MyEntity:{id}"; // The key for the cached object.
    await cache.KeyDeleteAsync(key).ConfigureAwait(false); // Delete this key from the cache.
}
```

> [!NOTE]
> L’ordre dans lequel sont effectuées les étapes est important. Mettez à jour la banque de données *avant* de supprimer l’élément du cache. Si vous supprimez d’abord l’élément mis en cache, un client pourrait avoir le temps d’extraire l’élément avant que la banque de données ne soit mise à jour. En raison de cette absence dans le cache (l’élément a été supprimé du cache), la version antérieure de l’élément serait extraite de la banque de données et ajoutée au cache. On obtiendrait donc des données en cache obsolètes.

## <a name="related-guidance"></a>Aide connexe

Les informations suivantes peuvent également être pertinentes durant l’implémentation de ce modèle :

- [Recommandations en matière de cache](https://docs.microsoft.com/azure/architecture/best-practices/caching). Fournit des informations supplémentaires sur la façon dont vous pouvez mettre en cache des données dans une solution cloud, ainsi que les points à prendre en compte lorsque vous implémentez un cache.

- [Manuel d’introduction à la cohérence des données](https://msdn.microsoft.com/library/dn589800.aspx). Les applications cloud utilisent généralement des données réparties dans plusieurs banques de données. La gestion et la maintenance de la cohérence des données dans cet environnement constituent un aspect essentiel du système, notamment par rapport aux problèmes de concurrence et de disponibilité pouvant survenir. Ce manuel décrit les problèmes de cohérence des données distribuées et explique comment une application peut implémenter la cohérence éventuelle pour garantir la disponibilité des données.
