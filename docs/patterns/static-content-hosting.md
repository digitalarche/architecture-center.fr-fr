---
title: Hébergement de contenu statique
description: Déployez un contenu statique dans un service de stockage cloud qui peut distribuer ce contenu directement au client.
keywords: modèle de conception
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- design-implementation
- performance-scalability
ms.openlocfilehash: 450d0c4c08098c1ba48e4c0dac3d058a46e3709b
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428208"
---
# <a name="static-content-hosting-pattern"></a><span data-ttu-id="8b550-104">Modèle d’hébergement de contenu statique</span><span class="sxs-lookup"><span data-stu-id="8b550-104">Static Content Hosting pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="8b550-105">Déployez un contenu statique dans un service de stockage cloud qui peut distribuer ce contenu directement au client.</span><span class="sxs-lookup"><span data-stu-id="8b550-105">Deploy static content to a cloud-based storage service that can deliver them directly to the client.</span></span> <span data-ttu-id="8b550-106">Cela peut réduire le besoin en instances de calcul éventuellement coûteuses.</span><span class="sxs-lookup"><span data-stu-id="8b550-106">This can reduce the need for potentially expensive compute instances.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="8b550-107">Contexte et problème</span><span class="sxs-lookup"><span data-stu-id="8b550-107">Context and problem</span></span>

<span data-ttu-id="8b550-108">En règle générale, les applications web comprennent certains éléments de contenu statique.</span><span class="sxs-lookup"><span data-stu-id="8b550-108">Web applications typically include some elements of static content.</span></span> <span data-ttu-id="8b550-109">Ce contenu statique peut inclure des pages HTML et d’autres ressources telles que des images et des documents qui sont disponibles pour le client, que ce soit en tant que composant d’une page HTML (par exemple, images incorporées, feuilles de style et fichiers JavaScript côté client) ou sous forme de téléchargements distincts (par exemple, documents PDF).</span><span class="sxs-lookup"><span data-stu-id="8b550-109">This static content might include HTML pages and other resources such as images and documents that are available to the client, either as part of an HTML page (such as inline images, style sheets, and client-side JavaScript files) or as separate downloads (such as PDF documents).</span></span>

<span data-ttu-id="8b550-110">Même si les serveurs web sont bien réglés pour optimiser les requêtes via la mise en cache de sortie et l’exécution de code de page dynamique efficaces, ils doivent tout de même gérer les requêtes pour télécharger du contenu statique.</span><span class="sxs-lookup"><span data-stu-id="8b550-110">Although web servers are well tuned to optimize requests through efficient dynamic page code execution and output caching, they still have to handle requests to download static content.</span></span> <span data-ttu-id="8b550-111">Cela fait appel à des cycles de traitement qui peuvent souvent être mieux utilisés ailleurs.</span><span class="sxs-lookup"><span data-stu-id="8b550-111">This consumes processing cycles that could often be put to better use.</span></span>

## <a name="solution"></a><span data-ttu-id="8b550-112">Solution</span><span class="sxs-lookup"><span data-stu-id="8b550-112">Solution</span></span>

<span data-ttu-id="8b550-113">Dans la plupart des environnements d’hébergement cloud, il est possible de réduire la nécessité d’instances de calcul (par exemple, utiliser une instance plus petite ou moins d’instances) en recherchant les pages statiques et les ressources d’une application dans un service de stockage.</span><span class="sxs-lookup"><span data-stu-id="8b550-113">In most cloud hosting environments it's possible to minimize the need for compute instances (for example, use a smaller instance or fewer instances), by locating some of an application’s resources and static pages in a storage service.</span></span> <span data-ttu-id="8b550-114">Le coût du stockage hébergé sur le cloud est généralement moindre par rapport à celui des instances de calcul.</span><span class="sxs-lookup"><span data-stu-id="8b550-114">The cost for cloud-hosted storage is typically much less than for compute instances.</span></span>

<span data-ttu-id="8b550-115">Lors de l’hébergement de certaines parties d’une application dans un service de stockage, les principales considérations à prendre en compte sont celles liées au déploiement de l’application et à la sécurisation des ressources qui ne sont pas destinées à être accessibles pour les utilisateurs anonymes.</span><span class="sxs-lookup"><span data-stu-id="8b550-115">When hosting some parts of an application in a storage service, the main considerations are related to deployment of the application and to securing resources that aren't intended to be available to anonymous users.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="8b550-116">Problèmes et considérations</span><span class="sxs-lookup"><span data-stu-id="8b550-116">Issues and considerations</span></span>

<span data-ttu-id="8b550-117">Prenez en compte les points suivants lorsque vous choisissez comment implémenter ce modèle :</span><span class="sxs-lookup"><span data-stu-id="8b550-117">Consider the following points when deciding how to implement this pattern:</span></span>

- <span data-ttu-id="8b550-118">Le service de stockage hébergé doit exposer un point de terminaison HTTP auquel les utilisateurs peuvent accéder pour télécharger les ressources statiques.</span><span class="sxs-lookup"><span data-stu-id="8b550-118">The hosted storage service must expose an HTTP endpoint that users can access to download the static resources.</span></span> <span data-ttu-id="8b550-119">Certains services de stockage prenant également en charge HTTPS, il est possible d’héberger des ressources dans les services de stockage qui exigent SSL.</span><span class="sxs-lookup"><span data-stu-id="8b550-119">Some storage services also support HTTPS, so it's possible to host resources in storage services that require SSL.</span></span>

- <span data-ttu-id="8b550-120">Pour optimiser les performances et la disponibilité, envisagez d’utiliser un réseau de distribution de contenu (CDN) pour mettre en cache le contenu du conteneur de stockage dans plusieurs centres de données dans le monde entier.</span><span class="sxs-lookup"><span data-stu-id="8b550-120">For maximum performance and availability, consider using a content delivery network (CDN) to cache the contents of the storage container in multiple datacenters around the world.</span></span> <span data-ttu-id="8b550-121">Toutefois, vous devrez probablement payer pour utiliser le CDN.</span><span class="sxs-lookup"><span data-stu-id="8b550-121">However, you'll likely have to pay for using the CDN.</span></span>

- <span data-ttu-id="8b550-122">Les comptes de stockage sont souvent géorépliqués par défaut pour assurer la résilience par rapport aux événements susceptibles d’affecter un centre de données.</span><span class="sxs-lookup"><span data-stu-id="8b550-122">Storage accounts are often geo-replicated by default to provide resiliency against events that might affect a datacenter.</span></span> <span data-ttu-id="8b550-123">Cela signifie que l’adresse IP peut être changée, mais que l’URL restera la même.</span><span class="sxs-lookup"><span data-stu-id="8b550-123">This means that the IP address might change, but the URL will remain the same.</span></span>

- <span data-ttu-id="8b550-124">Lorsque du contenu se trouve dans un compte de stockage et qu’un autre contenu se trouve dans une instance de calcul hébergée, il devient plus difficile de déployer une application et de la mettre à jour.</span><span class="sxs-lookup"><span data-stu-id="8b550-124">When some content is located in a storage account and other content is in a hosted compute instance it becomes more challenging to deploy an application and to update it.</span></span> <span data-ttu-id="8b550-125">Vous devrez peut-être effectuer des déploiements distincts et contrôler la version de l’application et du contenu pour les gérer de façon plus simple&mdash;particulièrement quand le contenu statique inclut des fichiers de script ou des composants d’interface utilisateur.</span><span class="sxs-lookup"><span data-stu-id="8b550-125">You might have to perform separate deployments, and version the application and content to manage it more easily&mdash;especially when the static content includes script files or UI components.</span></span> <span data-ttu-id="8b550-126">Toutefois, si seules les ressources statiques doivent être mises à jour, elles peuvent simplement être chargées sur le compte de stockage sans avoir à redéployer le package d’application.</span><span class="sxs-lookup"><span data-stu-id="8b550-126">However, if only static resources have to be updated, they can simply be uploaded to the storage account without needing to redeploy the application package.</span></span>

- <span data-ttu-id="8b550-127">Les services de stockage peuvent ne pas prendre en charge l’utilisation de noms de domaine personnalisés.</span><span class="sxs-lookup"><span data-stu-id="8b550-127">Storage services might not support the use of custom domain names.</span></span> <span data-ttu-id="8b550-128">Dans ce cas, il est nécessaire de spécifier l’URL complète des ressources dans les liens, car ils seront dans un autre domaine que le contenu généré dynamiquement qui contient les liens.</span><span class="sxs-lookup"><span data-stu-id="8b550-128">In this case it's necessary to specify the full URL of the resources in links because they'll be in a different domain from the dynamically-generated content containing the links.</span></span>

- <span data-ttu-id="8b550-129">Les conteneurs de stockage doivent être configurés pour un accès en lecture public, mais il est essentiel de vous assurer qu’ils ne sont pas configurés pour l’accès en écriture public, afin d’empêcher les utilisateurs de charger du contenu.</span><span class="sxs-lookup"><span data-stu-id="8b550-129">The storage containers must be configured for public read access, but it's vital to ensure that they aren't configured for public write access to prevent users being able to upload content.</span></span> <span data-ttu-id="8b550-130">Envisagez d’utiliser une clé de valet ou un jeton pour contrôler l’accès aux ressources qui ne doivent pas être disponibles de façon anonyme&mdash;, consultez [Valet Key pattern](valet-key.md) (Modèle de clé de valet) pour plus d’informations.</span><span class="sxs-lookup"><span data-stu-id="8b550-130">Consider using a valet key or token to control access to resources that shouldn't be available anonymously&mdash;see the [Valet Key pattern](valet-key.md) for more information.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="8b550-131">Quand utiliser ce modèle</span><span class="sxs-lookup"><span data-stu-id="8b550-131">When to use this pattern</span></span>

<span data-ttu-id="8b550-132">Ce modèle est utile dans les situations suivantes :</span><span class="sxs-lookup"><span data-stu-id="8b550-132">This pattern is useful for:</span></span>

- <span data-ttu-id="8b550-133">Pour réduire le coût d’hébergement des sites web et des applications qui contiennent certaines ressources statiques.</span><span class="sxs-lookup"><span data-stu-id="8b550-133">Minimizing the hosting cost for websites and applications that contain some static resources.</span></span>

- <span data-ttu-id="8b550-134">Pour réduire le coût d’hébergement des sites web qui comprennent uniquement des ressources et du contenu statiques.</span><span class="sxs-lookup"><span data-stu-id="8b550-134">Minimizing the hosting cost for websites that consist of only static content and resources.</span></span> <span data-ttu-id="8b550-135">Selon les capacités du système de stockage du fournisseur d’hébergement, il peut être possible d’héberger totalement un site web complètement statique dans un compte de stockage.</span><span class="sxs-lookup"><span data-stu-id="8b550-135">Depending on the capabilities of the hosting provider’s storage system, it might be possible to entirely host a fully static website in a storage account.</span></span>

- <span data-ttu-id="8b550-136">Pour exposer du contenu et des ressources statiques pour les applications s’exécutant dans d’autres environnements d’hébergement ou sur des serveurs locaux.</span><span class="sxs-lookup"><span data-stu-id="8b550-136">Exposing static resources and content for applications running in other hosting environments or on-premises servers.</span></span>

- <span data-ttu-id="8b550-137">Pour localiser du contenu dans plus d’une zone géographique à l’aide d’un CDN qui met en cache les contenus du compte de stockage dans plusieurs centres de données partout dans le monde.</span><span class="sxs-lookup"><span data-stu-id="8b550-137">Locating content in more than one geographical area using a content delivery network that caches the contents of the storage account in multiple datacenters around the world.</span></span>

- <span data-ttu-id="8b550-138">Pour surveiller les coûts et l’utilisation de la bande passante.</span><span class="sxs-lookup"><span data-stu-id="8b550-138">Monitoring costs and bandwidth usage.</span></span> <span data-ttu-id="8b550-139">Utiliser un compte de stockage distinct pour certains ou l’ensemble des contenus statiques permet de séparer plus facilement les coûts d’exécution et d’hébergement.</span><span class="sxs-lookup"><span data-stu-id="8b550-139">Using a separate storage account for some or all of the static content allows the costs to be more easily separated from hosting and runtime costs.</span></span>

<span data-ttu-id="8b550-140">Ce modèle peut s’avérer inutile dans les situations suivantes :</span><span class="sxs-lookup"><span data-stu-id="8b550-140">This pattern might not be useful in the following situations:</span></span>

- <span data-ttu-id="8b550-141">L’application doit effectuer une partie du traitement du contenu statique avant de le fournir au client.</span><span class="sxs-lookup"><span data-stu-id="8b550-141">The application needs to perform some processing on the static content before delivering it to the client.</span></span> <span data-ttu-id="8b550-142">Par exemple, il peut être nécessaire d’ajouter un timestamp à un document.</span><span class="sxs-lookup"><span data-stu-id="8b550-142">For example, it might be necessary to add a timestamp to a document.</span></span>

- <span data-ttu-id="8b550-143">Le volume du contenu statique est très faible.</span><span class="sxs-lookup"><span data-stu-id="8b550-143">The volume of static content is very small.</span></span> <span data-ttu-id="8b550-144">La surcharge liée à la récupération de ce contenu à partir du stockage distinct peut annuler l’avantage en termes de coûts de sa séparation de la ressource de calcul.</span><span class="sxs-lookup"><span data-stu-id="8b550-144">The overhead of retrieving this content from separate storage can outweigh the cost benefit of separating it out from the compute resource.</span></span>

## <a name="example"></a><span data-ttu-id="8b550-145">Exemples</span><span class="sxs-lookup"><span data-stu-id="8b550-145">Example</span></span>

<span data-ttu-id="8b550-146">Le contenu statique se trouvant dans le stockage d’objets blob Azure est accessible directement via un navigateur web.</span><span class="sxs-lookup"><span data-stu-id="8b550-146">Static content located in Azure Blob storage can be accessed directly by a web browser.</span></span> <span data-ttu-id="8b550-147">Azure fournit une interface basée sur HTTP pour le stockage qui peut être exposé publiquement aux clients.</span><span class="sxs-lookup"><span data-stu-id="8b550-147">Azure provides an HTTP-based interface over storage that can be publicly exposed to clients.</span></span> <span data-ttu-id="8b550-148">Par exemple, le contenu d’un conteneur de stockage d’objets blob Azure est exposé à l’aide d’une URL sous la forme suivante :</span><span class="sxs-lookup"><span data-stu-id="8b550-148">For example, content in an Azure Blob storage container is exposed using a URL with the following form:</span></span>

`https://[ storage-account-name ].blob.core.windows.net/[ container-name ]/[ file-name ]`


<span data-ttu-id="8b550-149">Lors du chargement du contenu, il est nécessaire de créer un ou plusieurs conteneurs blob pour stocker les fichiers et les documents.</span><span class="sxs-lookup"><span data-stu-id="8b550-149">When uploading the content it's necessary to create one or more blob containers to hold the files and documents.</span></span> <span data-ttu-id="8b550-150">Notez que l’autorisation par défaut d’un nouveau conteneur est Privée et que vous devez la remplacer par Publique pour permettre aux clients d’accéder aux contenus.</span><span class="sxs-lookup"><span data-stu-id="8b550-150">Note that the default permission for a new container is Private, and you must change this to Public to allow clients to access the contents.</span></span> <span data-ttu-id="8b550-151">S’il est nécessaire de protéger le contenu des accès anonymes, vous pouvez implémenter le [modèle de clé de valet](valet-key.md), afin que les utilisateurs présentent un jeton valide pour télécharger les ressources.</span><span class="sxs-lookup"><span data-stu-id="8b550-151">If it's necessary to protect the content from anonymous access, you can implement the [Valet Key pattern](valet-key.md) so users must present a valid token to download the resources.</span></span>

> <span data-ttu-id="8b550-152">L’article [Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx) (Concepts de service blob) contient des informations sur le stockage d’objets blob et sur les méthodes vous permettant d’y accéder et de l’utiliser.</span><span class="sxs-lookup"><span data-stu-id="8b550-152">[Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx) has information about blob storage, and the ways that you can access and use it.</span></span>

<span data-ttu-id="8b550-153">Les liens de chaque page spécifieront l’URL de la ressource et le client y accédera directement à partir du service de stockage.</span><span class="sxs-lookup"><span data-stu-id="8b550-153">The links in each page will specify the URL of the resource and the client will access it directly from the storage service.</span></span> <span data-ttu-id="8b550-154">La figure illustre la distribution des parties statiques d’une application directement à partir d’un service de stockage.</span><span class="sxs-lookup"><span data-stu-id="8b550-154">The figure illustrates delivering static parts of an application directly from a storage service.</span></span>

![Figure 1 - Distribution des parties statiques d’une application directement à partir d’un service de stockage](./_images/static-content-hosting-pattern.png)


<span data-ttu-id="8b550-156">Les liens des pages distribuées au client doivent spécifier l’URL complète de la ressource et du conteneur d’objets blob.</span><span class="sxs-lookup"><span data-stu-id="8b550-156">The links in the pages delivered to the client must specify the full URL of the blob container and resource.</span></span> <span data-ttu-id="8b550-157">Par exemple, une page qui contient un lien vers une image dans un conteneur public peut contenir le code HTML suivant.</span><span class="sxs-lookup"><span data-stu-id="8b550-157">For example, a page that contains a link to an image in a public container might contain the following HTML.</span></span>

```html
<img src="https://mystorageaccount.blob.core.windows.net/myresources/image1.png"
     alt="My image" />
```

> <span data-ttu-id="8b550-158">Si les ressources sont protégées à l’aide d’une clé de valet, par exemple une signature d’accès partagé Azure, cette signature doit être incluse dans les URL des liens.</span><span class="sxs-lookup"><span data-stu-id="8b550-158">If the resources are protected by using a valet key, such as an Azure shared access signature, this signature must be included in the URLs in the links.</span></span>

<span data-ttu-id="8b550-159">Une solution nommée StaticContentHosting qui illustre l’utilisation du stockage externe pour les ressources statiques est disponible à partir de [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span><span class="sxs-lookup"><span data-stu-id="8b550-159">A solution named StaticContentHosting that demonstrates using external storage for static resources is available from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span></span> <span data-ttu-id="8b550-160">Le projet StaticContentHosting.Cloud contient les fichiers de configuration qui spécifient le compte de stockage et le conteneur qui détient le contenu statique.</span><span class="sxs-lookup"><span data-stu-id="8b550-160">The StaticContentHosting.Cloud project contains configuration files that specify the storage account and container that holds the static content.</span></span>

```xml
<Setting name="StaticContent.StorageConnectionString"
         value="UseDevelopmentStorage=true" />
<Setting name="StaticContent.Container" value="static-content" />
```

<span data-ttu-id="8b550-161">La classe `Settings` dans le fichier Settings.cs du projet StaticContentHosting.Web contient des méthodes pour extraire ces valeurs et générer une valeur de chaîne contenant l’URL de conteneur du compte de stockage cloud.</span><span class="sxs-lookup"><span data-stu-id="8b550-161">The `Settings` class in the file Settings.cs of the StaticContentHosting.Web project contains methods to extract these values and build a string value containing the cloud storage account container URL.</span></span>

```csharp
public class Settings
{
  public static string StaticContentStorageConnectionString {
    get
    {
      return RoleEnvironment.GetConfigurationSettingValue(
                              "StaticContent.StorageConnectionString");
    }
  }

  public static string StaticContentContainer
  {
    get
    {
      return RoleEnvironment.GetConfigurationSettingValue("StaticContent.Container");
    }
  }

  public static string StaticContentBaseUrl
  {
    get
    {
      var account = CloudStorageAccount.Parse(StaticContentStorageConnectionString);

      return string.Format("{0}/{1}", account.BlobEndpoint.ToString().TrimEnd('/'),
                                      StaticContentContainer.TrimStart('/'));
    }
  }
}
```

<span data-ttu-id="8b550-162">La classe `StaticContentUrlHtmlHelper` dans le fichier StaticContentUrlHtmlHelper.cs expose une méthode nommée `StaticContentUrl` qui génère une URL contenant le chemin d’accès au compte de stockage cloud si l’URL transmise à ce dernier commence par le caractère de chemin d’accès ASP.NET racine (~).</span><span class="sxs-lookup"><span data-stu-id="8b550-162">The `StaticContentUrlHtmlHelper` class in the file StaticContentUrlHtmlHelper.cs exposes a method named `StaticContentUrl` that generates a URL containing the path to the cloud storage account if the URL passed to it starts with the ASP.NET root path character (~).</span></span>

```csharp
public static class StaticContentUrlHtmlHelper
{
  public static string StaticContentUrl(this HtmlHelper helper, string contentPath)
  {
    if (contentPath.StartsWith("~"))
    {
      contentPath = contentPath.Substring(1);
    }

    contentPath = string.Format("{0}/{1}", Settings.StaticContentBaseUrl.TrimEnd('/'),
                                contentPath.TrimStart('/'));

    var url = new UrlHelper(helper.ViewContext.RequestContext);

    return url.Content(contentPath);
  }
}
```

<span data-ttu-id="8b550-163">Le fichier Index.cshtml dans le dossier Views\Home contient un élément d’image qui utilise la méthode `StaticContentUrl` pour créer l’URL pour son attribut `src`.</span><span class="sxs-lookup"><span data-stu-id="8b550-163">The file Index.cshtml in the Views\Home folder contains an image element that uses the `StaticContentUrl` method to create the URL for its `src` attribute.</span></span>

```html
<img src="@Html.StaticContentUrl("~/media/orderedList1.png")" alt="Test Image" />
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="8b550-164">Conseils et modèles connexes</span><span class="sxs-lookup"><span data-stu-id="8b550-164">Related patterns and guidance</span></span>

- <span data-ttu-id="8b550-165">Un exemple illustrant ce modèle est disponible sur [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span><span class="sxs-lookup"><span data-stu-id="8b550-165">A sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span></span>
- <span data-ttu-id="8b550-166">[Valet Key pattern](valet-key.md) (Modèle de clé de valet).</span><span class="sxs-lookup"><span data-stu-id="8b550-166">[Valet Key pattern](valet-key.md).</span></span> <span data-ttu-id="8b550-167">Si les ressources cibles ne sont pas censées être accessibles aux utilisateurs anonymes, il est nécessaire d’implémenter la sécurité sur le magasin qui contient le contenu statique.</span><span class="sxs-lookup"><span data-stu-id="8b550-167">If the target resources aren't supposed to be available to anonymous users it's necessary to implement security over the store that holds the static content.</span></span> <span data-ttu-id="8b550-168">Décrit l’utilisation d’un jeton ou d’une clé qui fournit aux clients un accès direct limité à une ressource ou un service spécifique comme un service de stockage hébergé sur le cloud.</span><span class="sxs-lookup"><span data-stu-id="8b550-168">Describes how to use a token or key that provides clients with restricted direct access to a specific resource or service such as a cloud-hosted storage service.</span></span>
- <span data-ttu-id="8b550-169">[Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx) (Concepts de service blob)</span><span class="sxs-lookup"><span data-stu-id="8b550-169">[Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx)</span></span>
