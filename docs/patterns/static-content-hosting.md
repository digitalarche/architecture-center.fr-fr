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
ms.openlocfilehash: deb15001bea2598d56a2793be78bbc3e7473bdf3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2017
---
# <a name="static-content-hosting-pattern"></a>Modèle d’hébergement de contenu statique

[!INCLUDE [header](../_includes/header.md)]

Déployez un contenu statique dans un service de stockage cloud qui peut distribuer ce contenu directement au client. Cela peut réduire le besoin en instances de calcul éventuellement coûteuses.

## <a name="context-and-problem"></a>Contexte et problème

En règle générale, les applications web comprennent certains éléments de contenu statique. Ce contenu statique peut inclure des pages HTML et d’autres ressources telles que des images et des documents qui sont disponibles pour le client, que ce soit en tant que composant d’une page HTML (par exemple, images incorporées, feuilles de style et fichiers JavaScript côté client) ou sous forme de téléchargements distincts (par exemple, documents PDF).

Même si les serveurs web sont bien réglés pour optimiser les requêtes via la mise en cache de sortie et l’exécution de code de page dynamique efficaces, ils doivent tout de même gérer les requêtes pour télécharger du contenu statique. Cela fait appel à des cycles de traitement qui peuvent souvent être mieux utilisés ailleurs.

## <a name="solution"></a>Solution

Dans la plupart des environnements d’hébergement cloud, il est possible de réduire la nécessité d’instances de calcul (par exemple, utiliser une instance plus petite ou moins d’instances) en recherchant les pages statiques et les ressources d’une application dans un service de stockage. Le coût du stockage hébergé sur le cloud est généralement moindre par rapport à celui des instances de calcul.

Lors de l’hébergement de certaines parties d’une application dans un service de stockage, les principales considérations à prendre en compte sont celles liées au déploiement de l’application et à la sécurisation des ressources qui ne sont pas destinées à être accessibles pour les utilisateurs anonymes.

## <a name="issues-and-considerations"></a>Problèmes et considérations

Prenez en compte les points suivants lorsque vous choisissez comment implémenter ce modèle :

- Le service de stockage hébergé doit exposer un point de terminaison HTTP auquel les utilisateurs peuvent accéder pour télécharger les ressources statiques. Certains services de stockage prenant également en charge HTTPS, il est possible d’héberger des ressources dans les services de stockage qui exigent SSL.

- Pour optimiser les performances et la disponibilité, envisagez d’utiliser un réseau de distribution de contenu (CDN) pour mettre en cache le contenu du conteneur de stockage dans plusieurs centres de données dans le monde entier. Toutefois, vous devrez probablement payer pour utiliser le CDN.

- Les comptes de stockage sont souvent géorépliqués par défaut pour assurer la résilience par rapport aux événements susceptibles d’affecter un centre de données. Cela signifie que l’adresse IP peut être changée, mais que l’URL restera la même.

- Lorsque du contenu se trouve dans un compte de stockage et qu’un autre contenu se trouve dans une instance de calcul hébergée, il devient plus difficile de déployer une application et de la mettre à jour. Vous devrez peut-être effectuer des déploiements distincts et contrôler la version de l’application et du contenu pour les gérer de façon plus simple&mdash;particulièrement quand le contenu statique inclut des fichiers de script ou des composants d’interface utilisateur. Toutefois, si seules les ressources statiques doivent être mises à jour, elles peuvent simplement être chargées sur le compte de stockage sans avoir à redéployer le package d’application.

- Les services de stockage peuvent ne pas prendre en charge l’utilisation de noms de domaine personnalisés. Dans ce cas, il est nécessaire de spécifier l’URL complète des ressources dans les liens, car ils seront dans un autre domaine que le contenu généré dynamiquement qui contient les liens.

- Les conteneurs de stockage doivent être configurés pour un accès en lecture public, mais il est essentiel de vous assurer qu’ils ne sont pas configurés pour l’accès en écriture public, afin d’empêcher les utilisateurs de charger du contenu. Envisagez d’utiliser une clé de valet ou un jeton pour contrôler l’accès aux ressources qui ne doivent pas être disponibles de façon anonyme&mdash;, consultez [Valet Key pattern](valet-key.md) (Modèle de clé de valet) pour plus d’informations.

## <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle

Ce modèle est utile dans les situations suivantes :

- Pour réduire le coût d’hébergement des sites web et des applications qui contiennent certaines ressources statiques.

- Pour réduire le coût d’hébergement des sites web qui comprennent uniquement des ressources et du contenu statiques. Selon les capacités du système de stockage du fournisseur d’hébergement, il peut être possible d’héberger totalement un site web complètement statique dans un compte de stockage.

- Pour exposer du contenu et des ressources statiques pour les applications s’exécutant dans d’autres environnements d’hébergement ou sur des serveurs locaux.

- Pour localiser du contenu dans plus d’une zone géographique à l’aide d’un CDN qui met en cache les contenus du compte de stockage dans plusieurs centres de données partout dans le monde.

- Pour surveiller les coûts et l’utilisation de la bande passante. Utiliser un compte de stockage distinct pour certains ou l’ensemble des contenus statiques permet de séparer plus facilement les coûts d’exécution et d’hébergement.

Ce modèle peut s’avérer inutile dans les situations suivantes :

- L’application doit effectuer une partie du traitement du contenu statique avant de le fournir au client. Par exemple, il peut être nécessaire d’ajouter un timestamp à un document.

- Le volume du contenu statique est très faible. La surcharge liée à la récupération de ce contenu à partir du stockage distinct peut annuler l’avantage en termes de coûts de sa séparation de la ressource de calcul.

## <a name="example"></a>Exemple

Le contenu statique se trouvant dans le stockage d’objets blob Azure est accessible directement via un navigateur web. Azure fournit une interface basée sur HTTP pour le stockage qui peut être exposé publiquement aux clients. Par exemple, le contenu d’un conteneur de stockage d’objets blob Azure est exposé à l’aide d’une URL sous la forme suivante :

`http://[ storage-account-name ].blob.core.windows.net/[ container-name ]/[ file-name ]`


Lors du chargement du contenu, il est nécessaire de créer un ou plusieurs conteneurs blob pour stocker les fichiers et les documents. Notez que l’autorisation par défaut d’un nouveau conteneur est Privée et que vous devez la remplacer par Publique pour permettre aux clients d’accéder aux contenus. S’il est nécessaire de protéger le contenu des accès anonymes, vous pouvez implémenter le [modèle de clé de valet](valet-key.md), afin que les utilisateurs présentent un jeton valide pour télécharger les ressources.

> L’article [Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx) (Concepts de service blob) contient des informations sur le stockage d’objets blob et sur les méthodes vous permettant d’y accéder et de l’utiliser.

Les liens de chaque page spécifieront l’URL de la ressource et le client y accédera directement à partir du service de stockage. La figure illustre la distribution des parties statiques d’une application directement à partir d’un service de stockage.

![Figure 1 - Distribution des parties statiques d’une application directement à partir d’un service de stockage](./_images/static-content-hosting-pattern.png)


Les liens des pages distribuées au client doivent spécifier l’URL complète de la ressource et du conteneur d’objets blob. Par exemple, une page qui contient un lien vers une image dans un conteneur public peut contenir le code HTML suivant.

```html
<img src="http://mystorageaccount.blob.core.windows.net/myresources/image1.png"
     alt="My image" />
```

> Si les ressources sont protégées à l’aide d’une clé de valet, par exemple une signature d’accès partagé Azure, cette signature doit être incluse dans les URL des liens.

Une solution nommée StaticContentHosting qui illustre l’utilisation du stockage externe pour les ressources statiques est disponible à partir de [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting). Le projet StaticContentHosting.Cloud contient les fichiers de configuration qui spécifient le compte de stockage et le conteneur qui détient le contenu statique.

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

- Un exemple illustrant ce modèle est disponible sur [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).
- [Valet Key pattern](valet-key.md) (Modèle de clé de valet). Si les ressources cibles ne sont pas censées être accessibles aux utilisateurs anonymes, il est nécessaire d’implémenter la sécurité sur le magasin qui contient le contenu statique. Décrit l’utilisation d’un jeton ou d’une clé qui fournit aux clients un accès direct limité à une ressource ou un service spécifique comme un service de stockage hébergé sur le cloud.
- [An efficient way of deploying a static web site on Azure](http://www.infosysblogs.com/microsoft/2010/06/an_efficient_way_of_deploying.html) (Un moyen efficace pour déployer un site web statique sur Azure) sur le blog Infosys.
- [Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx) (Concepts de service blob)
