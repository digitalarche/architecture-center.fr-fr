---
title: Modèle d’hébergement de contenu statique
titleSuffix: Cloud Design Patterns
description: Déployez un contenu statique dans un service de stockage cloud qui peut distribuer ce contenu directement au client.
keywords: modèle de conception
author: dragon119
ms.date: 01/04/2019
ms.custom: seodec18
ms.openlocfilehash: cf4f65e935a01e4d84b3cc82b5779edb729bd80e
ms.sourcegitcommit: 036cd03c39f941567e0de4bae87f4e2aa8c84cf8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/05/2019
ms.locfileid: "54058180"
---
# <a name="static-content-hosting-pattern"></a>Modèle d’hébergement de contenu statique

Déployez un contenu statique dans un service de stockage cloud qui peut distribuer ce contenu directement au client. Cela peut réduire le besoin en instances de calcul éventuellement coûteuses.

## <a name="context-and-problem"></a>Contexte et problème

En règle générale, les applications web comprennent certains éléments de contenu statique. Ce contenu statique peut inclure des pages HTML et d’autres ressources telles que des images et des documents qui sont disponibles pour le client, que ce soit en tant que composant d’une page HTML (par exemple, images incorporées, feuilles de style et fichiers JavaScript côté client) ou sous forme de téléchargements distincts (par exemple, documents PDF).

Bien que les serveurs web soient optimisés pour le rendu dynamique et la mise en cache de sortie, ils doivent néanmoins gérer les requêtes pour le téléchargement de contenu statique. Cela fait appel à des cycles de traitement qui peuvent souvent être mieux utilisés ailleurs.

## <a name="solution"></a>Solution

Dans la plupart des environnements d’hébergement cloud, vous pouvez placer certaines des ressources et des pages statiques d’une application dans un service de stockage. Le service de stockage peut traiter les requêtes de ces ressources, ce qui permet ainsi de réduire la charge des ressources de calcul qui gèrent d’autres requêtes web. Le coût du stockage hébergé sur le cloud est généralement moindre par rapport à celui des instances de calcul.

Lors de l’hébergement de certaines parties d’une application dans un service de stockage, les principales considérations à prendre en compte sont celles liées au déploiement de l’application et à la sécurisation des ressources qui ne sont pas destinées à être accessibles pour les utilisateurs anonymes.

## <a name="issues-and-considerations"></a>Problèmes et considérations

Prenez en compte les points suivants lorsque vous choisissez comment implémenter ce modèle :

- Le service de stockage hébergé doit exposer un point de terminaison HTTP auquel les utilisateurs peuvent accéder pour télécharger les ressources statiques. Certains services de stockage prenant également en charge HTTPS, il est possible d’héberger des ressources dans les services de stockage qui exigent SSL.

- Pour optimiser les performances et la disponibilité, envisagez d’utiliser un réseau de distribution de contenu (CDN) pour mettre en cache le contenu du conteneur de stockage dans plusieurs centres de données dans le monde entier. Toutefois, vous devrez probablement payer pour utiliser le CDN.

- Les comptes de stockage sont souvent géorépliqués par défaut pour assurer la résilience par rapport aux événements susceptibles d’affecter un centre de données. Cela signifie que l’adresse IP peut être changée, mais que l’URL restera la même.

- Quand un contenu se trouve dans un compte de stockage et qu’un autre contenu se trouve dans une instance de calcul hébergée, il devient plus difficile de déployer une application et de la mettre à jour. Vous devrez peut-être effectuer des déploiements distincts et contrôler la version de l’application et du contenu pour les gérer de façon plus simple&mdash;particulièrement quand le contenu statique inclut des fichiers de script ou des composants d’interface utilisateur. Toutefois, si seules les ressources statiques doivent être mises à jour, elles peuvent simplement être chargées sur le compte de stockage sans avoir à redéployer le package d’application.

- Les services de stockage peuvent ne pas prendre en charge l’utilisation de noms de domaine personnalisés. Dans ce cas, il est nécessaire de spécifier l’URL complète des ressources dans les liens, car ils seront dans un autre domaine que le contenu généré dynamiquement qui contient les liens.

- Les conteneurs de stockage doivent être configurés pour un accès en lecture public, mais il est essentiel de vous assurer qu’ils ne sont pas configurés pour l’accès en écriture public, afin d’empêcher les utilisateurs de charger du contenu.

- Utilisez une clé de valet ou un jeton pour contrôler l’accès aux ressources qui ne doivent pas être disponibles de façon anonyme. Pour plus d’informations, consultez [Modèle de clé de valet](./valet-key.md).

## <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle

Ce modèle est utile dans les situations suivantes :

- Pour réduire le coût d’hébergement des sites web et des applications qui contiennent certaines ressources statiques.

- Pour réduire le coût d’hébergement des sites web qui comprennent uniquement des ressources et du contenu statiques. En fonction des capacités du système de stockage du fournisseur d’hébergement, il est possible d’héberger intégralement un site web complètement statique dans un compte de stockage.

- Pour exposer du contenu et des ressources statiques pour les applications s’exécutant dans d’autres environnements d’hébergement ou sur des serveurs locaux.

- Pour localiser du contenu dans plus d’une zone géographique à l’aide d’un CDN qui met en cache les contenus du compte de stockage dans plusieurs centres de données partout dans le monde.

- Pour surveiller les coûts et l’utilisation de la bande passante. Utiliser un compte de stockage distinct pour certains ou l’ensemble des contenus statiques permet de séparer plus facilement les coûts d’exécution et d’hébergement.

Ce modèle peut s’avérer inutile dans les situations suivantes :

- L’application doit effectuer une partie du traitement du contenu statique avant de le fournir au client. Par exemple, il peut être nécessaire d’ajouter un timestamp à un document.

- Le volume du contenu statique est très faible. La surcharge liée à la récupération de ce contenu à partir du stockage distinct peut annuler l’avantage en termes de coûts de sa séparation de la ressource de calcul.

## <a name="example"></a>Exemples

Le Stockage Azure prend en charge la fourniture de contenu statique directement à partir d’un conteneur de stockage. Les fichiers sont traités via des requêtes d’accès anonymes. Par défaut, les fichiers ont une URL dans un sous-domaine de `core.windows.net`, par exemple `https://contoso.z4.web.core.windows.net/image.png`. Vous pouvez configurer un nom de domaine personnalisé et utiliser Azure CDN pour accéder aux fichiers via une connexion HTTPS. Pour plus d’informations, consultez [Hébergement de sites web statiques dans le service Stockage Azure](/azure/storage/blobs/storage-blob-static-website).

![Distribution de parties statiques d’une application directement à partir d’un service de stockage](./_images/static-content-hosting-pattern.png)

L’hébergement de sites web statiques permet l’accès anonyme aux fichiers. Si vous devez contrôler l’accès aux fichiers de manière individualisée, stockez ces derniers dans le Stockage Blob Azure, puis générez des [signatures d’accès partagé](/azure/storage/common/storage-dotnet-shared-access-signature-part-1) pour limiter l’accès.

Les liens des pages distribuées au client doivent spécifier l’URL complète de la ressource. Si la ressource est protégée à l’aide d’une clé de valet, par exemple une signature d’accès partagé Azure, cette signature doit être incluse dans l’URL.

Un exemple d’application illustrant l’utilisation du stockage externe pour les ressources statiques est disponible sur [GitHub][sample-app]. Cet exemple utilise des fichiers config pour spécifier le compte de stockage ainsi que le conteneur qui détient le contenu statique.

```xml
<Setting name="StaticContent.StorageConnectionString"
         value="UseDevelopmentStorage=true" />
<Setting name="StaticContent.Container" value="static-content" />
```

La classe `Settings` dans le fichier Settings.cs du projet StaticContentHosting.Web contient des méthodes pour extraire ces valeurs et générer une valeur de chaîne contenant l’URL de conteneur du compte de stockage cloud.

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

La classe `StaticContentUrlHtmlHelper` dans le fichier StaticContentUrlHtmlHelper.cs expose une méthode nommée `StaticContentUrl` qui génère une URL contenant le chemin d’accès au compte de stockage cloud si l’URL transmise à ce dernier commence par le caractère de chemin d’accès ASP.NET racine (~).

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

Le fichier Index.cshtml dans le dossier Views\Home contient un élément d’image qui utilise la méthode `StaticContentUrl` pour créer l’URL pour son attribut `src`.

```html
<img src="@Html.StaticContentUrl("~/media/orderedList1.png")" alt="Test Image" />
```

## <a name="related-patterns-and-guidance"></a>Conseils et modèles connexes

- [Exemple d’hébergement de contenu statique][sample-app]. Exemple d’application qui illustre ce modèle.
- [Valet Key pattern](./valet-key.md) (Modèle de clé de valet). Si les ressources cibles ne sont pas censées être accessibles aux utilisateurs anonymes, utilisez ce modèle pour restreindre l’accès direct.
- [Application web serverless sur Azure](../reference-architectures/serverless/web-app.md). Architecture de référence utilisant un hébergement web statique avec Azure Functions pour implémenter une application web serverless.

[sample-app]: https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting
