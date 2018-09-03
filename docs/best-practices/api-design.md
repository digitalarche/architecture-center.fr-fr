---
title: Guide de conception d’API
description: Conseils sur la création et la conception efficace d’une API web.
author: dragon119
ms.date: 01/12/2018
pnp.series.title: Best Practices
ms.openlocfilehash: 68ed3f59e1fd63ae754ceabf27a182daa0de0e5d
ms.sourcegitcommit: c4106b58ad08f490e170e461009a4693578294ea
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/10/2018
ms.locfileid: "43016074"
---
# <a name="api-design"></a>Conception d’API

Les applications web plus modernes exposent les API que les clients peuvent utiliser pour interagir avec l’application. Une API web bien conçue doit prendre en charge les capacités suivantes :

* **Indépendance de la plateforme**. Tous les clients doivent être en mesure d’appeler l’API, quelle que soit la façon dont l’API est implémentée en interne. Cela requiert l’utilisation de protocoles standard et un mécanisme où le client et le service web peuvent convenir du format des données à échanger.

* **Évolution des services**. L’API web doit être en mesure d’évoluer et ses fonctionnalités doivent pouvoir être ajoutées indépendamment des applications clientes. À mesure que l’API évolue, les applications clientes existantes doivent continuer de fonctionner sans modifications. Toutes les fonctionnalités doivent être détectables, de sorte que les applications clientes puissent les utiliser pleinement.

Ce guide décrit les problèmes à prendre en compte lorsque vous concevez une API web.

## <a name="introduction-to-rest"></a>Présentation de REST

En 2000, Roy Fielding a présenté Representational State Transfer (REST) en tant qu’approche d’architecture pour concevoir des services web. REST est un style d’architecture pour la création de systèmes distribués basés sur l’hypermédia. REST est indépendant de tout protocole sous-jacent et n’est pas nécessairement lié à HTTP. Toutefois, les implémentations REST les plus courantes utilisent HTTP comme protocole d’application, et ce guide s’intéresse à la conception d’API REST pour HTTP.

L’un des principaux avantages de REST par rapport à HTTP est qu’il utilise des normes ouvertes et qu’il ne lie pas l’implémentation de l’API ou des applications clientes à une implémentation spécifique. Par exemple, un service web REST peut être écrit dans ASP.NET, et les applications clientes peuvent utiliser n’importe quel langage ou ensemble d’outils qui peut générer des requêtes HTTP et analyser des réponses HTTP.

Voici quelques-uns des principes de conception clés d’API RESTful à l’aide de HTTP :

- Les API REST sont conçues autour de *ressources*, qui peuvent être tout type d’objet, de données ou de service accessibles par le client. 

- Une ressource possède un *identificateur*, qui est un URI qui identifie de façon unique cette ressource. Par exemple, l’URI d’une commande client spécifique peut être : 
 
    ```http
    http://adventure-works.com/orders/1
    ```
 
- Les clients interagissent avec un service en échangeant des *représentations* de ressources. De nombreuses API web utilisent JSON comme format d’échange. Par exemple, une requête GET envoyée à l’URI ci-dessus peut renvoyer ce corps de réponse :

    ```json
    {"orderId":1,"orderValue":99.90,"productId":1,"quantity":1}
    ```

- Les API REST utilisent une interface uniforme, ce qui vous permet de séparer les implémentations du client et du service. Pour les API REST qui reposent sur HTTP, l’interface uniforme inclut l’utilisation de verbes HTTP standard pour effectuer des opérations sur les ressources. Les opérations les plus courantes sont GET, POST, PUT, PATCH et DELETE. 

- Les API REST utilisent un modèle de requête sans état. Les requêtes HTTP doivent être indépendantes et peuvent être générées dans n’importe quel ordre. Par conséquent, les informations d’état temporaires entre les requêtes ne peuvent pas être conservées. Les ressources elles-mêmes représentent l’unique emplacement de stockage de ces informations. Chaque requête doit être une opération atomique. Cette contrainte permet aux services web d’être extrêmement évolutifs, car il est inutile de conserver des affinités entre les clients et les serveurs spécifiques. Tous les serveurs peuvent gérer des demandes provenant de n’importe quel client. Cela étant dit, d’autres facteurs peuvent limiter l’extensibilité. Par exemple, de nombreux services web écrivent dans un magasin de données principal, dont vous pouvez difficilement augmenter la taille des instances. (L’article [Partitionnement des données](./data-partitioning.md) décrit les stratégies d’augmentation de la taille des instances d’un magasin de données.)

- Les API REST sont pilotées par des liens hypermédias qui figurent dans la représentation. L’exemple suivant montre une représentation JSON d’une commande. Il contient des liens permettant d’obtenir ou de mettre à jour le client associé à la commande. 
 
    ```json
    {
        "orderID":3,
        "productID":2,
        "quantity":4,
        "orderValue":16.60,
        "links": [
            {"rel":"product","href":"http://adventure-works.com/customers/3", "action":"GET" },
            {"rel":"product","href":"http://adventure-works.com/customers/3", "action":"PUT" } 
        ]
    } 
    ```


C’est en 2008 que Leonard Richardson a proposé le [ modèle de maturité](https://martinfowler.com/articles/richardsonMaturityModel.html) suivant pour les API web :

- Niveau 0 : Définir un URI, et toutes les opérations sont des requêtes POST envoyées à cet URI.
- Niveau 1 : Créer des URI distincts pour des ressources individuelles.
- Niveau 2 : Utiliser des méthodes HTTP pour définir des opérations sur des ressources.
- Niveau 3 : Utiliser l’hypermédia (HATEOAS, décrit ci-dessous).

Le niveau 3 correspond à une véritable API RESTful, telle que définie par Roy Fielding. Dans la pratique, un grand nombre d’API web publiées se trouvent quelque part autour du niveau 2.  

## <a name="organize-the-api-around-resources"></a>Organiser l’API autour des ressources

Concentrez-vous sur les entités métier exposées par l’API web. Par exemple, dans un système d’e-commerce, les entités principales peuvent être des clients et des commandes. La création d’une commande peut être obtenue en envoyant une requête POST HTTP qui contient les informations de la commande. La réponse HTTP indique si la commande a été passée avec succès ou non. Lorsque cela est possible, les URI de ressource doivent être basées sur des noms (la ressource) et non sur des verbes (les opérations sur la ressource). 

```HTTP
http://adventure-works.com/orders // Good

http://adventure-works.com/create-order // Avoid
```

Une ressource ne doit pas nécessairement être basée sur un élément de données physique unique. Par exemple, une ressource de commande peut être implémentée en interne sous la forme de plusieurs tables d’une base de données relationnelle, mais présentée au client comme une entité unique. Évitez de créer des API qui reflètent simplement la structure interne d’une base de données. L’objectif de REST est de modéliser des entités et les opérations qu’une application peut effectuer sur ces entités. Un client ne doit pas être exposé à l’implémentation interne.

Les entités sont souvent regroupées dans des collections (commandes, clients). Une collection est une ressource distincte de l’élément dans la collection et doit avoir son propre URI. Par exemple, l’URI suivant peut représenter la collection de commandes : 

```HTTP
http://adventure-works.com/orders
```

L’envoi d’une requête GET HTTP à l’URI de la collection récupère une liste d’éléments dans la collection. Chaque élément de la collection possède également son propre URI unique. Une requête GET HTTP envoyée à l’URI de l’élément renvoie les détails de cet élément. 

Adoptez une convention d’affectation de noms cohérente pour les URI. En général, il est recommandé d’utiliser des noms au pluriel pour les URI qui référencent des collections. Il est judicieux d’organiser les URI pour les collections et les éléments dans une hiérarchie. Par exemple, `/customers` est le chemin d’accès à la collection des clients, et `/customers/5` est le chemin d’accès au client avec un ID égal à 5. Cette approche contribue à préserver l’intuitivité de l’API web. En outre, plusieurs infrastructures d’API web peuvent acheminer les requêtes en fonction de chemins d’accès paramétrables pour que vous puissiez définir un itinéraire pour le chemin d’accès `/customers/{id}`.

Prenez également en compte les relations entre les différents types de ressources et la façon dont vous pouvez exposer ces associations. Par exemple, le `/customers/5/orders` peut représenter toutes les commandes du client 5. Vous pouvez également aller dans l’autre sens et représenter l’association d’une commande vers un client avec un URI tel que `/orders/99/customer`. Cependant, développer ce modèle de façon trop poussée peut complexifier son implémentation. Une meilleure solution consiste à fournir des liens navigables vers des ressources associées dans le corps du message de réponse HTTP. Ce mécanisme est décrit plus en détail dans la section [Utilisation de l’approche HATEOAS pour autoriser la navigation vers des ressources associées](#using-the-hateoas-approach-to-enable-navigation-to-related-resources) plus loin dans cet article.

Dans les systèmes plus complexes, il peut être tentant de fournir des URI qui permettent à un client de naviguer parmi différents niveaux de relations, tels que `/customers/1/orders/99/products`. Toutefois, ce niveau de complexité peut être difficile à gérer et n’offre aucune flexibilité si les relations entre les ressources changent ultérieurement. Au lieu de cela, optez pour des URI aussi simples que possible. Une fois qu’une application a une référence à une ressource, il doit être possible d’utiliser cette référence pour rechercher des éléments liés à cette ressource. La requête précédente peut être remplacée par l’URI `/customers/1/orders` pour rechercher toutes les commandes pour le client 1, puis par `/orders/99/products` pour rechercher les produits dans cette commande.

> [!TIP]
> Évitez d’imposer des URI de ressource plus complexes que *collection/item/collection*.

Vous devez également garder à l’esprit que toutes les requêtes web imposent une charge sur le serveur web. Plus le nombre de requêtes est important, plus la charge est élevée. Par conséquent, essayez d’éviter les API web effectuant de nombreux échanges et qui exposent un grand nombre de ressources de petite taille. Une telle API peut nécessiter l’envoi de plusieurs requêtes par une application cliente pour rechercher toutes les données requises. À la place, vous souhaitez peut-être dénormaliser les données et combiner des informations connexes en ressources plus volumineuses, qui peuvent être récupérées à l’aide d’une seule requête. Toutefois, vous devez mesurer cette approche par rapport à la surcharge imposée par l’extraction de données inutiles pour le client. La récupération d’objets volumineux peut augmenter la latence d’une requête et entraîner des coûts supplémentaires liés à la bande passante. Pour plus d’informations sur ces antimodèles de performance, consultez [Antimodèle E/S bavardes](../antipatterns/chatty-io/index.md) et [Antimodèle de récupération superflue](../antipatterns/extraneous-fetching/index.md).

Évitez d’introduire des dépendances entre l’API web et les sources de données sous-jacentes. Par exemple, si vos données sont stockées dans une base de données relationnelle, l’API web n’a pas besoin d’exposer chaque table comme une collection de ressources. En fait, il s’agit probablement d’une conception médiocre. Au lieu de cela, considérez l’API web comme une abstraction de la base de données. Si nécessaire, ajoutez une couche de mappage entre la base de données et l’API web. De cette façon, les applications clientes sont isolées des modifications apportées au schéma de base de données sous-jacent.

Enfin, il n’est pas toujours possible de mapper chaque opération implémentée par une API web à une ressource spécifique. Vous pouvez gérer ces scénarios *sans ressource* par le biais de requêtes HTTP qui appellent une fonction et renvoient les résultats dans un message de réponse HTTP. Par exemple, une API web qui implémente des opérations de calcul simples, comme une addition ou une soustraction, peut fournir des URI qui exposent ces opérations en tant que pseudo-ressources et utilisent la chaîne de requête pour spécifier les paramètres requis. Par exemple, une requête GET envoyée à l’URI */add?operand1=99&operand2=1* renvoie un message de réponse dont le corps contient la valeur 100. Toutefois, utilisez ces formes d’URI avec parcimonie.

## <a name="define-operations-in-terms-of-http-methods"></a>Définir des opérations en termes de méthodes HTTP

Le protocole HTTP définit un certain nombre de méthodes qui affectent une signification sémantique à une requête. Les méthodes HTTP utilisées par la plupart des API web RESTful sont les suivantes :

* **GET** récupère une représentation de la ressource à l’URI spécifié. Le corps du message de réponse contient les détails de la ressource demandée.
* **POST** crée une ressource à l’URI spécifié. Le corps du message de requête fournit les détails de la nouvelle ressource. Notez que POST permet également de déclencher des opérations qui ne créent pas réellement de ressources.
* **PUT** crée ou remplace la ressource à l’URI spécifié. Le corps du message de requête spécifie la ressource à créer ou mettre à jour.
* **PATCH** effectue une mise à jour partielle d’une ressource. Le corps de la requête spécifie l’ensemble de modifications à appliquer à la ressource.
* **DELETE** supprime la ressource à l’URI spécifié.

L’effet d’une requête spécifique sera différent, selon que la ressource est une collection ou un élément individuel. Le tableau suivant résume les conventions courantes adoptées par la plupart des implémentations RESTful suivant l’exemple du commerce électronique. Notez que certaines de ces requêtes ne peuvent pas être implémentées ; cela dépend du scénario spécifique.

| **Ressource** | **POST** | **GET** | **PUT** | **DELETE** |
| --- | --- | --- | --- | --- |
| /customers |Créer un client |Récupérer tous les clients |Mettre à jour des clients en bloc |Supprimer tous les clients |
| /customers/1 |Error |Récupérer les détails du client 1 |Mettre à jour les détails du client 1 s’il existe |Supprimer le client 1 |
| /customers/1/orders |Créer une commande pour le client 1 |Récupérer toutes les commandes pour le client 1 |Mettre à jour des commandes en bloc pour le client 1 |Supprimer toutes les commandes pour le client 1 |

Les différences entre POST, PUT et PATCH peuvent prêter à confusion.

- Une requête POST crée une ressource. Le serveur assigne un URI à la nouvelle ressource et renvoie cet URI au client. Dans le modèle REST, vous appliquez fréquemment les requêtes POST aux collections. La nouvelle ressource est ajoutée à la collection. Une requête POST peut également être utilisée pour envoyer des données pour le traitement d’une ressource existante, sans qu’une nouvelle ressource ne soit créée.

- Une requête PUT crée une ressource *ou* met à jour une ressource existante. Le client spécifie l’URI de la ressource. Le corps de la requête contient une représentation complète de la ressource. Si une ressource avec cet URI existe déjà, elle est remplacée. Sinon, une ressource est créée si le serveur peut prendre en charge cette opération. Les requêtes PUT sont plus fréquemment appliquées à des ressources qui sont des éléments individuels, par exemple un client spécifique, plutôt qu’à des collections. Un serveur peut prendre en charge les mises à jour, mais pas la création via la méthode PUT. Prendre en charge la création via la méthode PUT dépend de la capacité du client à assigner de façon explicite un URI à une ressource avant qu’elle existe. S’il n’est pas en mesure de le faire, utilisez la méthode POST pour créer des ressources et les méthodes PUT ou PATCH pour effectuer une mise à jour.

- Une requête PATCH effectue une *mise à jour partielle* vers une ressource existante. Le client spécifie l’URI de la ressource. Le corps de la requête spécifie un ensemble de *modifications* à appliquer à la ressource. Cela peut être plus efficace que l’utilisation de la méthode PUT, étant donné que le client envoie uniquement les modifications, et non pas la représentation entière de la ressource. Techniquement, la méthode PATCH peut également créer une ressource (en spécifiant un ensemble de mises à jour vers une ressource « null »), si le serveur prend en charge cette opération. 

Les requêtes PUT doivent être idempotentes. Si un client envoie la même requête PUT plusieurs fois, les résultats doivent toujours être identiques (la même ressource sera modifiée avec les mêmes valeurs). Le caractère idempotent des requêtes POST et PATCH n’est pas garanti.

## <a name="conform-to-http-semantics"></a>Être en conformité par rapport à la sémantique HTTP

Cette section décrit certaines considérations classiques liées à la conception d’une API conforme à la spécification HTTP. Toutefois, elle n’aborde pas tous les détails ou scénarios possibles. En cas de doute, consultez les spécifications HTTP.

### <a name="media-types"></a>Types de médias

Comme mentionné précédemment, les clients et les serveurs échangent des représentations de ressources. Par exemple, dans une requête POST, le corps de la requête contient une représentation de la ressource à créer. Dans une requête GET, le corps de la réponse contient une représentation de la ressource extraite.

Dans le protocole HTTP, les formats sont spécifiés à l’aide de *types de médias*, également appelés types MIME. Pour les données non binaires, la plupart des API web prennent en charge JSON (type de média = application/json) et éventuellement XML (type de média = application/xml). 

L’en-tête Content-Type dans une requête ou une réponse spécifie le format de la représentation. Voici un exemple d’une requête POST qui comprend des données JSON :

```HTTP
POST http://adventure-works.com/orders HTTP/1.1
Content-Type: application/json; charset=utf-8
Content-Length: 57

{"Id":1,"Name":"Gizmo","Category":"Widgets","Price":1.99}
```

Si le serveur ne prend pas en charge le type de média, il doit renvoyer le code d’état HTTP 415 (Type de support non pris en charge).

Une requête cliente peut inclure un en-tête Accept qui contient une liste des types de médias acceptés par le client à partir du serveur dans le message de réponse. Par exemple : 

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
Accept: application/json
```

Si le serveur ne trouve pas de correspondances avec l’un des types de médias répertoriés, il doit renvoyer le code d’état HTTP 406 (Non acceptable). 

### <a name="get-methods"></a>Méthodes GET

En général, une méthode GET réussie renvoie le code d’état HTTP 200 (OK). Si la ressource est introuvable, la méthode doit renvoyer 404 (Introuvable).

### <a name="post-methods"></a>Méthodes POST

Si une méthode POST crée une ressource, elle renvoie le code d’état HTTP 201 (Créé). L’URI de la nouvelle ressource est inclus dans l’en-tête Location de la réponse. Le corps de la réponse contient une représentation de la ressource.

Si la méthode effectue des opérations de traitement, mais ne crée pas de ressource, elle peut renvoyer le code d’état HTTP 200 et inclure le résultat de l’opération dans le corps de la réponse. Ou bien, en l’absence de résultat à renvoyer, la méthode peut renvoyer le code d’état HTTP 204 (Pas de contenu) sans corps de réponse.

Si le client place des données non valides dans la requête, le serveur doit renvoyer le code d’état HTTP 400 (Demande incorrecte). Le corps de la réponse peut contenir des informations supplémentaires sur l’erreur ou un lien vers un URI qui fournit plus de détails.

### <a name="put-methods"></a>Méthodes PUT

Si une méthode PUT crée une ressource, elle renvoie le code d’état HTTP 201 (Créé), comme pour la méthode POST. Si la méthode met à jour une ressource existante, elle renvoie 200 (OK) ou 204 (Pas de contenu). Dans certains cas, mettre à jour une ressource existante peut se révéler impossible. Dans ce cas, envisagez de renvoyer le code d’état HTTP 409 (Conflit). 

Envisagez d’implémenter des opérations HTTP PUT en bloc, qui peuvent regrouper des mises à jour destinées à plusieurs ressources d’une collection. La requête PUT doit spécifier l’URI de la collection, et le corps de la requête doit spécifier les détails des ressources à modifier. Cette approche peut aider à réduire les échanges excessifs et à améliorer les performances.

### <a name="patch-methods"></a>Méthodes PATCH

Avec une requête PATCH, le client envoie un ensemble de mises à jour à une ressource existante, sous la forme d’un *document de correctif*. Le serveur traite le document de correctif pour effectuer la mise à jour. Le document de correctif ne décrit pas la ressource entière, mais uniquement un ensemble de modifications à appliquer. La spécification de la méthode PATCH ([RFC 5789](https://tools.ietf.org/html/rfc5789)) ne définit pas un format particulier pour les documents de correctif. Le format doit être déduit à partir du type de média dans la requête.

JSON est probablement le format de données le plus courant pour les API web. Il existe deux formats de correctif principaux basés sur JSON, appelés *correctif JSON* et *correctif de fusion JSON*.

Le correctif de fusion JSON est un peu plus simple. Le document de correctif a la même structure que la ressource JSON d’origine, mais inclut uniquement le sous-ensemble de champs à modifier ou à ajouter. En outre, un champ peut être supprimé en spécifiant `null` pour la valeur du champ dans le document de correctif. (Cela signifie que le correctif de fusion n’est pas adapté si la ressource d’origine peut avoir des valeurs null explicites.)

Par exemple, supposons que la ressource d’origine ait la représentation JSON suivante :

```json
{ 
    "name":"gizmo",
    "category":"widgets",
    "color":"blue",
    "price":10
}
```

Voici un correctif de fusion JSON possible pour cette ressource :

```json
{ 
    "price":12,
    "color":null,
    "size":"small"
}
```

Cela indique au serveur de mettre à jour « price », de supprimer « color » et d’ajouter « size ». « Name » et « category » ne sont pas modifiés. Pour des informations plus précises sur le correctif de fusion JSON, consultez [RFC 7396](https://tools.ietf.org/html/rfc7396). Le type de média pour le correctif de fusion JSON est « application/merge-patch+json ».

Le correctif de fusion n’est pas adapté si la ressource d’origine peut contenir des valeurs null explicites, en raison du sens particulier de `null` dans le document de correctif. En outre, le document de correctif ne spécifie pas l’ordre selon lequel le serveur doit appliquer les mises à jour. Cela peut, ou non, avoir son importance, selon le domaine et les données concernées. Le correctif JSON, défini dans [RFC 6902](https://tools.ietf.org/html/rfc6902), est plus souple. Il spécifie les modifications comme une séquence d’opérations à appliquer. Les opérations incluent l’ajout, la suppression, le remplacement, la copie et le test (pour valider les valeurs). Le type de média pour le correctif JSON est « application/json-patch+json ».

Vous trouverez ci-dessous des états d’erreur classiques pouvant être rencontrés lors du traitement d’une requête PATCH, accompagnés du code d’état HTTP approprié.

| État d’erreur | Code d'état HTTP |
|-----------|------------|
| Le format de document de correctif n’est pas pris en charge. | 415 (Type de support non pris en charge) |
| Document de correctif incorrect. | 400 (Demande incorrecte) |
| Le document de correctif est valide, mais les modifications ne peuvent pas être appliquées à la ressource dans son état actuel. | 409 (Conflit)

### <a name="delete-methods"></a>Méthodes DELETE

Si l’opération de suppression est réussie, le serveur web doit répondre avec un code d’état HTTP 204 indiquant que le processus a été géré correctement mais que le corps de la réponse ne contient aucune information supplémentaire. Si la ressource n’existe pas, le serveur web peut renvoyer HTTP 404 (Introuvable).

### <a name="asynchronous-operations"></a>Opérations asynchrones

Parfois, un processus de traitement un peu long peut avoir lieu pour une opération POST, PUT, PATCH ou DELETE. Si vous attendez la fin du traitement avant d’envoyer une réponse au client, cela peut entraîner une latence inacceptable. Dans ce cas, envisagez de rendre l’opération asynchrone. Renvoyez le code d’état HTTP 202 (Accepté) pour indiquer que la requête a été acceptée pour traitement, mais que celui-ci n’est pas terminé. 

Vous devez exposer un point de terminaison qui renvoie l’état d’une requête asynchrone ; le client peut ainsi surveiller l’état en interrogeant le point de terminaison de l’état. Ajoutez l’URI du point de terminaison de l’état dans l’en-tête Location de la réponse 202. Par exemple : 

```http
HTTP/1.1 202 Accepted
Location: /api/status/12345
```

Si le client envoie une requête GET à ce point de terminaison, la réponse doit contenir l’état actuel de la requête. Sinon, elle peut également inclure une durée estimée pour la fin du traitement ou un lien permettant d’annuler l’opération. 

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "status":"In progress",
    "link": { "rel":"cancel", "method":"delete", "href":"/api/status/12345" }
}
```

Si l’opération asynchrone crée une ressource, le point de terminaison d’état doit renvoyer le code d’état 303 (See Other) (Voir autre) une fois l’opération terminée. Dans la réponse 303, ajoutez un en-tête Location qui spécifie l’URI de la nouvelle ressource :

```http
HTTP/1.1 303 See Other
Location: /api/orders/12345
```

Pour plus d’informations, consultez [Asynchronous operations in REST](https://www.adayinthelifeof.nl/2011/06/02/asynchronous-operations-in-rest/) (Opérations asynchrones dans REST).

## <a name="filter-and-paginate-data"></a>Filtrer et paginer des données

L’exposition d’une collection de ressources par le biais d’un seul URI peut conduire les applications à extraire d’importantes quantités de données alors que seul un sous-ensemble d’informations est requis. Par exemple, imaginons qu’une application cliente ait besoin de rechercher toutes les commandes dont le coût est supérieur à une valeur spécifique. Elle peut récupérer toutes les commandes à partir de l’URI */commandes*, puis filtrer ces commandes sur le côté client. Clairement, ce processus est très peu efficace. Il gaspille la bande passante réseau et la puissance de traitement du serveur hébergeant l’API web.

Au lieu de cela, l’API peut autoriser la transmission d’un filtre dans la chaîne de requête de l’URI, tel que */orders?minCost=n*. L’API web est alors responsable de l’analyse et de la gestion du paramètre `minCost` dans la chaîne de requête et du renvoi des résultats filtrés sur le côté serveur. 

Les requêtes GET sur les ressources de la collection peuvent renvoyer un grand nombre d’éléments. Vous devez concevoir une API web pour limiter la quantité de données renvoyées par chaque requête. Envisagez de prendre en charge les chaînes de requête qui spécifient le nombre maximal d’éléments à récupérer et un décalage de départ dans la collection. Par exemple : 

```
/orders?limit=25&offset=50
```

Envisagez également d’imposer une limite supérieure du nombre d’éléments renvoyés, afin d’empêcher les attaques par déni de service. Pour aider les applications clientes, les requêtes GET qui renvoient les données paginées doivent également inclure des métadonnées qui indiquent le nombre total de ressources disponibles dans la collection. Vous pouvez également envisager d’autres stratégies de pagination intelligente. Pour plus d’informations, consultez [API Design Notes: Smart Paging](http://bizcoder.com/api-design-notes-smart-paging) (Notes de conception d’API : pagination intelligente).

Vous pouvez utiliser une stratégie similaire pour trier les données au fil de leur extraction en fournissant un paramètre de tri qui utilise un nom de champ comme valeur, tel que */orders?sort=ProductID*. Toutefois, cette approche peut avoir un effet négatif sur la mise en cache puisque les paramètres de chaîne de requête font partie de l’identificateur de ressource utilisé par de nombreuses implémentations de cache comme clé pour les données mises en cache.

Vous pouvez étendre cette approche pour limiter les champs renvoyés pour chaque élément, si ceux-ci contiennent une grande quantité de données. Vous pouvez notamment utiliser un paramètre de chaîne de requête qui accepte une liste de champs séparés par des virgules, par exemple */orders?fields=ProductID,Quantity*. 

Affectez des valeurs par défaut significatives à tous les paramètres facultatifs des chaînes de requête. Par exemple, définissez le paramètre `limit` sur 10 et le paramètre `offset` sur 0 si vous implémentez la pagination, définissez le paramètre de tri sur la clé de la ressource si vous implémentez le classement et définissez le paramètre `fields` pour tous les champs de la ressource si vous prenez en charge des projections.

## <a name="support-partial-responses-for-large-binary-resources"></a>Prendre en charge des réponses partielles pour les ressources binaires volumineuses

Une ressource peut contenir des champs binaires volumineux, tels que des fichiers ou des images. Pour remédier aux problèmes provoqués par des connexions non fiables et intermittentes et pour améliorer les temps de réponse, envisagez la récupération de ces ressources en blocs. Pour ce faire, l’API web doit prendre en charge l’en-tête Accept-Ranges pour les requêtes GET en présence de ressources volumineuses. Cet en-tête indique que l’opération GET prend en charge les requêtes partielles. L’application cliente peut envoyer des requêtes GET qui renvoient un sous-ensemble d’une ressource, spécifié sous la forme d’une plage d’octets. 

En outre, envisagez d’implémenter les requêtes HEAD HTTP pour ces ressources. Une requête HEAD est similaire à une requête GET, à ceci près qu’elle renvoie uniquement des en-têtes HTTP qui décrivent la ressource et un corps de message vide. Une application cliente peut émettre une requête HEAD pour déterminer s’il faut extraire une ressource à l’aide de requêtes GET partielles. Par exemple : 

```HTTP
HEAD http://adventure-works.com/products/10?fields=productImage HTTP/1.1
```

Voici un exemple de message de réponse : 

```HTTP
HTTP/1.1 200 OK

Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 4580
```

L’en-tête Content-Length donne la taille totale de la ressource et l’en-tête Accept-Ranges indique que l’opération GET correspondante prend en charge les résultats partiels. L’application cliente peut utiliser ces informations pour récupérer l’image en blocs plus petits. La première requête extrait les 2500 premiers octets à l’aide de l’en-tête Range :

```HTTP
GET http://adventure-works.com/products/10?fields=productImage HTTP/1.1
Range: bytes=0-2499
```

Le message de réponse indique qu’il s’agit d’une réponse partielle en renvoyant le code d’état HTTP 206. L’en-tête Content-Length indique le nombre réel d’octets renvoyés dans le corps du message (pas la taille de la ressource), et l’en-tête Content-Range indique de quelle partie de la ressource il s’agit (octets 0-2499 sur 4580) :

```HTTP
HTTP/1.1 206 Partial Content

Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2500
Content-Range: bytes 0-2499/4580

[...]
```

Une requête ultérieure de l’application cliente peut récupérer le reste de la ressource.

## <a name="use-hateoas-to-enable-navigation-to-related-resources"></a>Utilisation de l’approche HATEOAS pour autoriser la navigation vers des ressources associées

L’utilisation de REST est principalement motivée par la possibilité de naviguer dans l’ensemble des ressources sans connaissance préalable du modèle d’URI. Chaque requête HTTP GET doit renvoyer les informations nécessaires pour trouver les ressources liées directement à l’objet demandé par le biais de liens hypertexte inclus dans la réponse. Des informations décrivant les opérations disponibles sur chacune de ces ressources doivent également être fournies. Il s’agit du principe HATEOAS (Hypertext as the Engine of Application State). Le système est effectivement une machine à états finis, et la réponse à chaque requête contient les informations nécessaires pour passer d’un état à l’autre ; aucune autre information n’est nécessaire.

> [!NOTE]
> Il n’existe actuellement aucune norme ou spécification définissant comment modéliser le principe HATEOAS. Les exemples présentés dans cette section illustrent une solution possible.
>
>

Par exemple, pour gérer la relation entre une commande et un client, la représentation d’une commande peut inclure des liens qui identifient les opérations disponibles pour le client de la commande. Voici une représentation possible : 

```json
{
  "orderID":3,
  "productID":2,
  "quantity":4,
  "orderValue":16.60,
  "links":[
    {
      "rel":"customer",
      "href":"http://adventure-works.com/customers/3", 
      "action":"GET",
      "types":["text/xml","application/json"] 
    },
    {
      "rel":"customer",
      "href":"http://adventure-works.com/customers/3", 
      "action":"PUT",
      "types":["application/x-www-form-urlencoded"]
    },
    {
      "rel":"customer",
      "href":"http://adventure-works.com/customers/3",
      "action":"DELETE",
      "types":[]
    },
    {
      "rel":"self",
      "href":"http://adventure-works.com/orders/3", 
      "action":"GET",
      "types":["text/xml","application/json"]
    },
    {
      "rel":"self",
      "href":"http://adventure-works.com/orders/3", 
      "action":"PUT",
      "types":["application/x-www-form-urlencoded"]
    },
    {
      "rel":"self",
      "href":"http://adventure-works.com/orders/3", 
      "action":"DELETE",
      "types":[]
    }]
}
```

Dans cet exemple, le tableau `links` présente un ensemble de liens. Chaque lien correspond à une opération sur une entité associée. Les données de chaque lien incluent la relation (« customer »), l’URI (`http://adventure-works.com/customers/3`), la méthode HTTP et les types MIME pris en charge. Il s’agit de toutes les informations nécessaires à une application cliente pour être en mesure d’appeler l’opération. 

Le tableau `links` inclut également des informations avec référence circulaire sur la ressource qui a été récupérée. La relation de ces éléments est *self*.

L’ensemble de liens renvoyé peut varier en fonction de l’état de la ressource. C’est la signification de « Hypertext as the Engine of Application State » (hypertexte en tant que moteur de l’état de l’application).

## <a name="versioning-a-restful-web-api"></a>Contrôle de version d’une API web RESTful

Il est très improbable qu’une API web reste statique. Au fil de l’évolution des besoins de l’entreprise, de nouvelles collections de ressources peuvent être ajoutées, les relations entre les ressources peuvent changer et la structure des données des ressources peut être modifiée. La mise à jour d’une API web pour gérer les exigences nouvelles ou différentes est un processus relativement simple. Cependant, vous devez considérer les effets de ces modifications sur les applications clientes utilisant l’API web. Le problème est le suivant : même si le développeur qui conçoit et implémente une API web bénéficie d’un contrôle total sur cette API, il n’a pas le même degré de contrôle sur les applications clientes qui peuvent être créées par des organisations tierces distantes. L’impératif principal consiste à faire en sorte que les applications clientes existantes puissent continuer à fonctionner sans modification tout en autorisant les nouvelles applications clientes à tirer parti des nouvelles fonctionnalités et ressources.

Le contrôle de version permet à une API web d’indiquer les fonctionnalités et ressources qu’elle expose. Une application cliente peut alors envoyer des requêtes destinées à une version spécifique d’une fonctionnalité ou d’une ressource. Les sections suivantes décrivent différentes approches présentant chacune des avantages et des inconvénients.

### <a name="no-versioning"></a>Aucun contrôle de version
Il s’agit de l’approche la plus simple. Elle peut être acceptable pour certaines API internes. De nouvelles ressources ou de nouveaux liens peuvent représenter des changements importants.  L’ajout de contenu à des ressources existantes ne représente pas nécessairement une modification avec rupture, dans la mesure où les applications clientes qui n’attendent pas ce contenu l’ignoreront simplement.

Par exemple, une requête à l’URI *http://adventure-works.com/customers/3* doit renvoyer les détails d’un client unique contenant les champs `id`, `name` et `address` attendus par l’application cliente :

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

> [!NOTE]
> À des fins de simplicité, les exemples de réponses présentés dans cette section n’incluent pas les liens HATEOAS.
>
>

Si le champ `DateCreated` est ajouté au schéma de la ressource du client, la réponse se présentera comme suit :

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":"1 Microsoft Way Redmond WA 98053"}
```

Les applications clientes existantes peuvent continuer à fonctionner correctement si elles peuvent ignorer les champs non reconnus. Les nouvelles applications clientes peuvent être conçues pour gérer ce nouveau champ. Cependant, des modifications plus importantes affectant le schéma des ressources (par exemple, la suppression ou la modification de nom de champs), ou un changement des relations entre les ressources peuvent constituer des modifications avec rupture empêchant les applications clientes existantes de fonctionner correctement. Dans ces situations, vous devez envisager l’une des approches suivantes.

### <a name="uri-versioning"></a>Contrôle de version d’URI
Dès lors que vous modifiez l’API web ou le schéma des ressources, vous ajoutez un numéro de version à l’URI pour chaque ressource. Les URI existants doivent continuer à fonctionner comme avant et à renvoyer les ressources conformes à leur schéma d’origine.

Si l’on étend l’exemple précédent, si le champ `address` est restructuré dans des sous-champs contenant chaque partie de l’adresse (par exemple, `streetAddress`, `city`, `state` et `zipCode`), cette version de la ressource peut être exposée par le biais d’un URI qui contient un numéro de version, tel que http://adventure-works.com/v2/customers/3:

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

Ce mécanisme de contrôle de version est très simple, mais dépend du routage de la requête par le serveur au point de terminaison approprié. Toutefois, il peut devenir complexe à mesure que l’API web arrive à maturité par le biais de plusieurs itérations et que le nombre de versions différentes prises en charge par le serveur évolue. En outre, d’un point de vue puriste, les applications clientes extraient les mêmes données (client 3) dans tous les cas. L’URI ne devrait donc pas être différent selon la version. Ce modèle complique également l’implémentation de HATEOAS, dans la mesure où tous les liens devront inclure le numéro de version dans leurs URI.

### <a name="query-string-versioning"></a>Contrôle de version de chaîne de requête
Plutôt que de fournir plusieurs URI, vous pouvez spécifier la version de la ressource en utilisant un paramètre dans la chaîne de requête ajoutée à la requête HTTP, par exemple *http://adventure-works.com/customers/3?version=2*. La valeur par défaut du paramètre de version doit être significative, par exemple 1, s’il est omis par des applications clientes plus anciennes.

D’un point de vue sémantique, cette approche présente l’avantage suivant : la même ressource est toujours extraite du même URI, mais cela dépend du code qui gère la demande d’analyse de la chaîne de requête et de renvoi de la réponse HTTP appropriée. D’autre part, cette approche présente les mêmes inconvénients que le mécanisme de contrôle de version d’URI concernant l’implémentation de HATEOAS.

> [!NOTE]
> Certains proxys et navigateurs web anciens ne mettent pas en cache les réponses aux requêtes qui incluent une chaîne de requête dans l’URI. Cela peut avoir un impact négatif sur les performances des applications web qui utilisent une API web et qui s’exécutent à partir des navigateurs web.
>
>

### <a name="header-versioning"></a>Contrôle de version d’en-tête
Plutôt que d’ajouter le numéro de version en tant que paramètre dans la chaîne de requête, vous pouvez implémenter un en-tête personnalisé qui indique la version de la ressource. Cette approche nécessite que l’application cliente ajoute l’en-tête approprié à toutes les requêtes, même si le code qui gère la requête client peut utiliser une valeur par défaut (version 1) si l’en-tête de version est omis. Les exemples suivants utilisent un en-tête personnalisé nommé *Custom-Header*. La valeur de cet en-tête indique la version de l’API web.

Version 1 :

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
Custom-Header: api-version=1
```

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

Version 2 :

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
Custom-Header: api-version=2
```

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

Notez que, comme avec les deux approches précédentes, l’implémentation de HATEOAS nécessite l’inclusion de l’en-tête personnalisé approprié dans tous les liens.

### <a name="media-type-versioning"></a>Contrôle de version du type de média
Lorsqu’une application cliente envoie une requête HTTP GET à un serveur web, elle doit indiquer le format du contenu qu’elle peut traiter à l’aide d’un en-tête Accept, comme décrit précédemment dans ce guide. Souvent, l’en-tête *Accept* a pour rôle de permettre à l’application cliente de spécifier si le corps de la réponse doit être au format XML ou JSON, ou dans un autre format courant que le client peut analyser. Toutefois, il est possible de définir des types de médias personnalisés qui incluent des informations permettant à l’application cliente d’indiquer la version de ressource attendue. L’exemple suivant illustre une requête qui spécifie un en-tête *Accept* avec la valeur *application/vnd.adventure-works.v1+json*. L’élément *vnd.adventure-works.v1* indique au serveur web qu’il doit renvoyer la version 1 de la ressource, tandis que l’élément *json* spécifie que le format du corps de la réponse doit être JSON :

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
Accept: application/vnd.adventure-works.v1+json
```

Le code qui gère la requête est responsable du traitement de l’en-tête *Accept* et doit le respecter autant que possible (l’application cliente peut spécifier plusieurs formats dans l’en-tête *Accept*, auquel cas le serveur web peut choisir le format le plus approprié pour le corps de la réponse). Le serveur web vérifie le format des données dans le corps de la réponse à l’aide de l’en-tête Content-Type :

```HTTP
HTTP/1.1 200 OK
Content-Type: application/vnd.adventure-works.v1+json; charset=utf-8

{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

Si l’en-tête Accept ne spécifie aucun type de média connu, le serveur web peut générer un message de réponse HTTP 406 (Non acceptable) ou renvoyer un message avec un type de média par défaut.

Cette approche représente sans doute le mécanisme de contrôle de version le plus pur et se prête naturellement à l’utilisation du principe HATEOAS, qui peut inclure le type MIME des données connexes dans les liens de ressources.

> [!NOTE]
> Lorsque vous choisissez une stratégie de contrôle de version, vous devez également tenir compte des implications sur les performances et en particulier de la mise en cache sur le serveur web. À cet égard, les modèles de contrôle de version d’URI et de chaîne de requête sont particulièrement efficaces, dans la mesure où la même combinaison URI/chaîne de requête fait référence aux mêmes données à chaque fois.
>
> Les mécanismes de contrôle de version d’en-tête et de type de média requièrent généralement une logique supplémentaire pour examiner les valeurs dans l’en-tête personnalisé ou l’en-tête Accept. Dans un environnement à grande échelle, l’utilisation de différentes versions d’une API web par de nombreux clients peut entraîner une quantité importante de données dupliquées dans un cache côté serveur. Ce problème peut s’aggraver si une application cliente communique avec un serveur web par l’intermédiaire d’un proxy qui implémente la mise en cache et qui transmet une requête au serveur web uniquement s’il ne contient pas une copie des données demandées dans son cache.
>
>

## <a name="open-api-initiative"></a>Open API Initiative
Le programme [Open API Initiative](https://www.openapis.org/) a été créé par un consortium industriel afin de normaliser les descriptions d’API REST entre les différents fournisseurs. Dans le cadre de ce programme, la spécification Swagger 2.0 a été renommée en spécification OpenAPI (OAS) et placée sous le programme Open API Initiative.

Vous pouvez adopter la spécification OpenAPI pour vos API Web. Éléments à prendre en considération :

- La spécification OpenAPI est fournie avec un ensemble de consignes strictes sur la façon dont une API REST doit être conçue. Ceci présente des avantages en termes d’interopérabilité, mais nécessite un surcroît d’attention de votre part lorsque vous concevez votre API pour qu’elle soit conforme à la spécification.
- OpenAPI promeut une approche qui commence par le contrat plutôt que par l’implémentation. Cela signifie que vous commencez par concevoir le contrat d’API (l’interface), puis que vous écrivez le code qui implémente ce contrat. 
- Des outils tels que Swagger peuvent générer des bibliothèques clientes ou une documentation à partir des contrats d’API. Par exemple, consultez l’article [ASP.NET Web API Help Pages using Swagger (Pages d’aide sur l’API Web ASP.NET à l’aide de Swagger)](/aspnet/core/tutorials/web-api-help-pages-using-swagger).

## <a name="more-information"></a>Plus d’informations
* [Microsoft REST API Guidelines](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md) (Directives relatives aux API REST Microsoft). Recommandations détaillées pour la conception d’API REST publiques.
* [The REST Cookbook](http://restcookbook.com/) (Le livre de recettes spécial REST). Introduction à la génération d’API RESTful.
* [Web API Checklist](https://mathieu.fenniak.net/the-api-checklist/) (Liste de contrôle pour les API web). Une liste d’éléments à prendre en compte lors de la conception et de l’implémentation d’une API web.
* [Open API Initiative](https://www.openapis.org/). Documentation et informations d’implémentation sur Open API.
