---
title: "Mise en cache de jetons d’accès dans une application multi-locataire"
description: "Mise en cache des jetons d’accès utilisés pour appeler une API web de serveur principal"
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: web-api
pnp.series.next: adfs
ms.openlocfilehash: ed0def244f3229bbd3fdd0976574d13a345f9aee
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2017
---
# <a name="cache-access-tokens"></a><span data-ttu-id="4ece7-103">Mettre en cache des jetons d’accès</span><span class="sxs-lookup"><span data-stu-id="4ece7-103">Cache access tokens</span></span>

<span data-ttu-id="4ece7-104">[![GitHub](../_images/github.png) Exemple de code][sample application]</span><span class="sxs-lookup"><span data-stu-id="4ece7-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="4ece7-105">Il est relativement coûteux d’obtenir un jeton d’accès OAuth, car cette opération requiert une requête HTTP au point de terminaison de jeton.</span><span class="sxs-lookup"><span data-stu-id="4ece7-105">It's relatively expensive to get an OAuth access token, because it requires an HTTP request to the token endpoint.</span></span> <span data-ttu-id="4ece7-106">Par conséquent, il est bon de mettre en cache les jetons lorsque cela est possible.</span><span class="sxs-lookup"><span data-stu-id="4ece7-106">Therefore, it's good to cache tokens whenever possible.</span></span> <span data-ttu-id="4ece7-107">La [bibliothèque d’authentification Azure AD][ADAL] (ADAL) met automatiquement en cache les jetons obtenus à partir d’Azure AD, notamment des jetons d’actualisation.</span><span class="sxs-lookup"><span data-stu-id="4ece7-107">The [Azure AD Authentication Library][ADAL] (ADAL)  automatically caches tokens obtained from Azure AD, including refresh tokens.</span></span>

<span data-ttu-id="4ece7-108">ADAL fournit une implémentation de cache de jeton par défaut.</span><span class="sxs-lookup"><span data-stu-id="4ece7-108">ADAL provides a default token cache implementation.</span></span> <span data-ttu-id="4ece7-109">Toutefois, ce cache de jeton est destiné aux applications clientes natives et n’est *pas* approprié pour les applications web :</span><span class="sxs-lookup"><span data-stu-id="4ece7-109">However, this token cache is intended for native client apps, and is *not* suitable for web apps:</span></span>

* <span data-ttu-id="4ece7-110">Il s’agit d’une instance statique, qui n’est pas thread-safe.</span><span class="sxs-lookup"><span data-stu-id="4ece7-110">It is a static instance, and not thread safe.</span></span>
* <span data-ttu-id="4ece7-111">Elle ne convient pas pour un grand nombre d’utilisateurs, car les jetons de tous les utilisateurs sont placés dans le même dictionnaire.</span><span class="sxs-lookup"><span data-stu-id="4ece7-111">It doesn't scale to large numbers of users, because tokens from all users go into the same dictionary.</span></span>
* <span data-ttu-id="4ece7-112">Elle ne peut pas être partagée entre les serveurs web d’une batterie de serveurs.</span><span class="sxs-lookup"><span data-stu-id="4ece7-112">It can't be shared across web servers in a farm.</span></span>

<span data-ttu-id="4ece7-113">Au lieu de cela, vous devez implémenter un cache de jeton personnalisé qui dérive de la classe `TokenCache` ADAL mais qui est adapté à un environnement serveur et fournit le niveau d’isolation souhaité entre les jetons de différents utilisateurs.</span><span class="sxs-lookup"><span data-stu-id="4ece7-113">Instead, you should implement a custom token cache that derives from the ADAL `TokenCache` class but is suitable for a server environment and provides the desirable level of isolation between tokens for different users.</span></span>

<span data-ttu-id="4ece7-114">La classe `TokenCache` stocke un dictionnaire de jetons, indexées par émetteur, ressource, ID de client et utilisateur.</span><span class="sxs-lookup"><span data-stu-id="4ece7-114">The `TokenCache` class stores a dictionary of tokens, indexed by issuer, resource, client ID, and user.</span></span> <span data-ttu-id="4ece7-115">Un cache de jeton personnalisé doit écrire ce dictionnaire dans un magasin de stockage, comme un cache Redis.</span><span class="sxs-lookup"><span data-stu-id="4ece7-115">A custom token cache should write this dictionary to a backing store, such as a Redis cache.</span></span>

<span data-ttu-id="4ece7-116">Dans l’application Surveys de Tailspin, la classe `DistributedTokenCache` implémente le cache de jeton.</span><span class="sxs-lookup"><span data-stu-id="4ece7-116">In the Tailspin Surveys application, the `DistributedTokenCache` class implements the token cache.</span></span> <span data-ttu-id="4ece7-117">Cette implémentation utilise l’abstraction [IDistributedCache][distributed-cache] d’ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="4ece7-117">This implementation uses the [IDistributedCache][distributed-cache] abstraction from ASP.NET Core.</span></span> <span data-ttu-id="4ece7-118">De cette façon, toute implémentation d’ `IDistributedCache` peut être utilisée comme magasin de stockage.</span><span class="sxs-lookup"><span data-stu-id="4ece7-118">That way, any `IDistributedCache` implementation can be used as a backing store.</span></span>

* <span data-ttu-id="4ece7-119">Par défaut, l’application Surveys utilise un cache Redis.</span><span class="sxs-lookup"><span data-stu-id="4ece7-119">By default, the Surveys app uses a Redis cache.</span></span>
* <span data-ttu-id="4ece7-120">Pour un serveur web à instance unique, vous pouvez utiliser le cache en mémoire [ASP.NET Core][in-memory-cache].</span><span class="sxs-lookup"><span data-stu-id="4ece7-120">For a single-instance web server, you could use the ASP.NET Core [in-memory cache][in-memory-cache].</span></span> <span data-ttu-id="4ece7-121">(Cette option est également idéale pour exécuter l’application localement pendant le développement.)</span><span class="sxs-lookup"><span data-stu-id="4ece7-121">(This is also a good option for running the app locally during development.)</span></span>

<span data-ttu-id="4ece7-122">`DistributedTokenCache` stocke les données du cache en tant que paires clé/valeur dans le magasin de stockage.</span><span class="sxs-lookup"><span data-stu-id="4ece7-122">`DistributedTokenCache` stores the cache data as key/value pairs in the backing store.</span></span> <span data-ttu-id="4ece7-123">La clé est l’ID d’utilisateur, plus l’ID client. Ainsi, le magasin de stockage conserve des données de cache distinctes pour chaque combinaison unique utilisateur/client.</span><span class="sxs-lookup"><span data-stu-id="4ece7-123">The key is the user ID plus client ID, so the backing store holds separate cache data for each unique combination of user/client.</span></span>

![Cache de jeton](./images/token-cache.png)

<span data-ttu-id="4ece7-125">Le magasin de stockage est partitionné par utilisateur.</span><span class="sxs-lookup"><span data-stu-id="4ece7-125">The backing store is partitioned by user.</span></span> <span data-ttu-id="4ece7-126">Pour chaque requête HTTP, les jetons de cet utilisateur sont lus à partir du magasin de stockage et chargés dans le dictionnaire `TokenCache` .</span><span class="sxs-lookup"><span data-stu-id="4ece7-126">For each HTTP request, the tokens for that user are read from the backing store and loaded into the `TokenCache` dictionary.</span></span> <span data-ttu-id="4ece7-127">Si Redis est utilisé comme magasin de stockage, chaque instance de serveur d’une batterie de serveurs lit/écrit dans le même cache, et cette approche est adaptée pour de nombreux utilisateurs.</span><span class="sxs-lookup"><span data-stu-id="4ece7-127">If Redis is used as the backing store, every server instance in a server farm reads/writes to the same cache, and this approach scales to many users.</span></span>

## <a name="encrypting-cached-tokens"></a><span data-ttu-id="4ece7-128">Chiffrement des jetons mis en cache</span><span class="sxs-lookup"><span data-stu-id="4ece7-128">Encrypting cached tokens</span></span>
<span data-ttu-id="4ece7-129">Les jetons sont des données sensibles, car elles donnent accès aux ressources d’un utilisateur.</span><span class="sxs-lookup"><span data-stu-id="4ece7-129">Tokens are sensitive data, because they grant access to a user's resources.</span></span> <span data-ttu-id="4ece7-130">(En outre, contrairement à un mot de passe d’utilisateur, vous ne pouvez pas simplement stocker un code de hachage du jeton.) Par conséquent, il est essentiel de protéger les jetons.</span><span class="sxs-lookup"><span data-stu-id="4ece7-130">(Moreover, unlike a user's password, you can't just store a hash of the token.) Therefore, it's critical to protect tokens from being compromised.</span></span> <span data-ttu-id="4ece7-131">Le cache Redis est protégé par un mot de passe, mais si quelqu'un obtient le mot de passe, cette personne peut accéder à tous les jetons d’accès mis en cache.</span><span class="sxs-lookup"><span data-stu-id="4ece7-131">The Redis-backed cache is protected by a password, but if someone obtains the password, they could get all of the cached access tokens.</span></span> <span data-ttu-id="4ece7-132">Pour cette raison, `DistributedTokenCache` chiffre toutes les données qu’il écrit dans le magasin de stockage.</span><span class="sxs-lookup"><span data-stu-id="4ece7-132">For that reason, the `DistributedTokenCache` encrypts everything that it writes to the backing store.</span></span> <span data-ttu-id="4ece7-133">Le chiffrement est effectué à l’aide des API de [protection des données][data-protection] ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="4ece7-133">Encryption is done using the ASP.NET Core [data protection][data-protection] APIs.</span></span>

> [!NOTE]
> <span data-ttu-id="4ece7-134">Si vous effectuez un déploiement sur des sites web Azure, les clés de chiffrement sont sauvegardées dans le stockage réseau et synchronisées sur tous les ordinateurs (voir [Durée de vie et de gestion de clés][key-management]).</span><span class="sxs-lookup"><span data-stu-id="4ece7-134">If you deploy to Azure Web Sites, the encryption keys are backed up to network storage and synchronized across all machines (see [Key management and lifetime][key-management]).</span></span> <span data-ttu-id="4ece7-135">Par défaut, les clés ne sont pas chiffrées lors de leur exécution dans les sites web Azure, mais vous pouvez [activer le chiffrement à l’aide d’un certificat X.509][x509-cert-encryption].</span><span class="sxs-lookup"><span data-stu-id="4ece7-135">By default, keys are not encrypted when running in Azure Web Sites, but you can [enable encryption using an X.509 certificate][x509-cert-encryption].</span></span>
> 
> 

## <a name="distributedtokencache-implementation"></a><span data-ttu-id="4ece7-136">Implémentation de DistributedTokenCache</span><span class="sxs-lookup"><span data-stu-id="4ece7-136">DistributedTokenCache implementation</span></span>
<span data-ttu-id="4ece7-137">La classe `DistributedTokenCache` dérive de la classe [TokenCache][tokencache-class] ADAL.</span><span class="sxs-lookup"><span data-stu-id="4ece7-137">The `DistributedTokenCache` class derives from the ADAL [TokenCache][tokencache-class] class.</span></span>

<span data-ttu-id="4ece7-138">Dans le constructeur, la classe `DistributedTokenCache` crée une clé pour l’utilisateur actuel et charge le cache à partir du magasin de stockage :</span><span class="sxs-lookup"><span data-stu-id="4ece7-138">In the constructor, the `DistributedTokenCache` class creates a key for the current user and loads the cache from the backing store:</span></span>

```csharp
public DistributedTokenCache(
    ClaimsPrincipal claimsPrincipal,
    IDistributedCache distributedCache,
    ILoggerFactory loggerFactory,
    IDataProtectionProvider dataProtectionProvider)
    : base()
{
    _claimsPrincipal = claimsPrincipal;
    _cacheKey = BuildCacheKey(_claimsPrincipal);
    _distributedCache = distributedCache;
    _logger = loggerFactory.CreateLogger<DistributedTokenCache>();
    _protector = dataProtectionProvider.CreateProtector(typeof(DistributedTokenCache).FullName);
    AfterAccess = AfterAccessNotification;
    LoadFromCache();
}
```

<span data-ttu-id="4ece7-139">La clé est créée en concaténant l’identifiant utilisateur et l’ID client.</span><span class="sxs-lookup"><span data-stu-id="4ece7-139">The key is created by concatenating the user ID and client ID.</span></span> <span data-ttu-id="4ece7-140">Ces deux éléments proviennent de revendications se trouvant dans le `ClaimsPrincipal`de l’utilisateur :</span><span class="sxs-lookup"><span data-stu-id="4ece7-140">Both of these are taken from claims found in the user's `ClaimsPrincipal`:</span></span>

```csharp
private static string BuildCacheKey(ClaimsPrincipal claimsPrincipal)
{
    string clientId = claimsPrincipal.FindFirstValue("aud", true);
    return string.Format(
        "UserId:{0}::ClientId:{1}",
        claimsPrincipal.GetObjectIdentifierValue(),
        clientId);
}
```

<span data-ttu-id="4ece7-141">Pour charger les données du cache, lisez l’objet blob sérialisé à partir du magasin de stockage, puis appelez `TokenCache.Deserialize` pour convertir l’objet blob en données de cache.</span><span class="sxs-lookup"><span data-stu-id="4ece7-141">To load the cache data, read the serialized blob from the backing store, and call `TokenCache.Deserialize` to convert the blob into cache data.</span></span>

```csharp
private void LoadFromCache()
{
    byte[] cacheData = _distributedCache.Get(_cacheKey);
    if (cacheData != null)
    {
        this.Deserialize(_protector.Unprotect(cacheData));
    }
}
```

<span data-ttu-id="4ece7-142">Chaque fois que la bibliothèque ADAL accède au cache, elle déclenche un événement `AfterAccess` .</span><span class="sxs-lookup"><span data-stu-id="4ece7-142">Whenever ADAL access the cache, it fires an `AfterAccess` event.</span></span> <span data-ttu-id="4ece7-143">Si les données du cache ont changé, la propriété `HasStateChanged` a la valeur true.</span><span class="sxs-lookup"><span data-stu-id="4ece7-143">If the cache data has changed, the `HasStateChanged` property is true.</span></span> <span data-ttu-id="4ece7-144">Dans ce cas, mettez à jour le magasin de stockage pour refléter la modification et affectez à `HasStateChanged` la valeur false.</span><span class="sxs-lookup"><span data-stu-id="4ece7-144">In that case, update the backing store to reflect the change, and then set `HasStateChanged` to false.</span></span>

```csharp
public void AfterAccessNotification(TokenCacheNotificationArgs args)
{
    if (this.HasStateChanged)
    {
        try
        {
            if (this.Count > 0)
            {
                _distributedCache.Set(_cacheKey, _protector.Protect(this.Serialize()));
            }
            else
            {
                // There are no tokens for this user/client, so remove the item from the cache.
                _distributedCache.Remove(_cacheKey);
            }
            this.HasStateChanged = false;
        }
        catch (Exception exp)
        {
            _logger.WriteToCacheFailed(exp);
            throw;
        }
    }
}
```

<span data-ttu-id="4ece7-145">TokenCache envoie deux autres événements :</span><span class="sxs-lookup"><span data-stu-id="4ece7-145">TokenCache sends two other events:</span></span>

* <span data-ttu-id="4ece7-146">`BeforeWrite`.</span><span class="sxs-lookup"><span data-stu-id="4ece7-146">`BeforeWrite`.</span></span> <span data-ttu-id="4ece7-147">Appelé juste avant que la bibliothèque ADAL effectue une opération d’écriture dans le cache.</span><span class="sxs-lookup"><span data-stu-id="4ece7-147">Called immediately before ADAL writes to the cache.</span></span> <span data-ttu-id="4ece7-148">Vous pouvez l’utiliser pour implémenter une stratégie d’accès concurrentiel.</span><span class="sxs-lookup"><span data-stu-id="4ece7-148">You can use this to implement a concurrency strategy</span></span>
* <span data-ttu-id="4ece7-149">`BeforeAccess`.</span><span class="sxs-lookup"><span data-stu-id="4ece7-149">`BeforeAccess`.</span></span> <span data-ttu-id="4ece7-150">Appelé juste avant que la bibliothèque ADAL effectue une opération de lecture à partir du cache.</span><span class="sxs-lookup"><span data-stu-id="4ece7-150">Called immediately before ADAL reads from the cache.</span></span> <span data-ttu-id="4ece7-151">Vous pouvez recharger ici le cache pour obtenir la version la plus récente.</span><span class="sxs-lookup"><span data-stu-id="4ece7-151">Here you can reload the cache to get the latest version.</span></span>

<span data-ttu-id="4ece7-152">Dans le cas présent, nous avons décidé de ne pas gérer ces deux événements.</span><span class="sxs-lookup"><span data-stu-id="4ece7-152">In our case, we decided not to handle these two events.</span></span>

* <span data-ttu-id="4ece7-153">Pour l’accès concurrentiel, la dernière écriture prévaut.</span><span class="sxs-lookup"><span data-stu-id="4ece7-153">For concurrency, last write wins.</span></span> <span data-ttu-id="4ece7-154">Cela ne pose pas de problème, car les jetons sont stockés séparément pour chaque utilisateur/client. Un conflit ne peut survenir que si le même utilisateur a deux sessions de connexion simultanées.</span><span class="sxs-lookup"><span data-stu-id="4ece7-154">That's OK, because tokens are stored independently for each user + client, so a conflict would only happen if the same user had two concurrent login sessions.</span></span>
* <span data-ttu-id="4ece7-155">Pour la lecture, nous chargeons le cache à chaque requête.</span><span class="sxs-lookup"><span data-stu-id="4ece7-155">For reading, we load the cache on every request.</span></span> <span data-ttu-id="4ece7-156">Les requêtes sont à durée de vie limitée.</span><span class="sxs-lookup"><span data-stu-id="4ece7-156">Requests are short lived.</span></span> <span data-ttu-id="4ece7-157">Si le cache est modifié pendant cette période, la requête suivante utilise la nouvelle valeur.</span><span class="sxs-lookup"><span data-stu-id="4ece7-157">If the cache gets modified in that time, the next request will pick up the new value.</span></span>

<span data-ttu-id="4ece7-158">[**Suivant**][client-assertion]</span><span class="sxs-lookup"><span data-stu-id="4ece7-158">[**Next**][client-assertion]</span></span>

<!-- links -->
[ADAL]: https://msdn.microsoft.com/library/azure/jj573266.aspx
[client-assertion]: ./client-assertion.md
[data-protection]: /aspnet/core/security/data-protection/
[distributed-cache]: /aspnet/core/performance/caching/distributed
[key-management]: /aspnet/core/security/data-protection/configuration/default-settings
[in-memory-cache]: /aspnet/core/performance/caching/memory
[tokencache-class]: https://msdn.microsoft.com/library/azure/microsoft.identitymodel.clients.activedirectory.tokencache.aspx
[x509-cert-encryption]: /aspnet/core/security/data-protection/implementation/key-encryption-at-rest#x509-certificate
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
