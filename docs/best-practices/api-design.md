---
title: "Guide de conception d’API"
description: "Conseils sur la création et la conception efficace d’une API."
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: 3ffadce1b0c4a4da808e52d61cff0b7f0b27de11
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2017
---
# <a name="api-design"></a>Conception d’API
[!INCLUDE [header](../_includes/header.md)]

De nombreuses solutions modernes basées sur le web utilisent des services web, hébergés par des serveurs web, pour fournir des fonctionnalités aux applications clientes distantes. Les opérations exposées par un service web constituent une API web. Une API web bien conçue doit prendre en charge les capacités suivantes :

* **Indépendance de la plateforme**. Les applications clientes doivent être en mesure d’utiliser l’API fournie par le service web quelle que soit la façon dont les données ou opérations exposées par l’API sont implémentées physiquement. Pour cela, l’API doit respecter des normes communes permettant à une application cliente et à un service web de s’accorder sur les formats de données à utiliser et sur la structure des données échangées entre les applications clientes et le service web.
* **Évolution des services**. Le service web doit être en mesure d’évoluer et ses fonctionnalités doivent pouvoir être modifiées (ajout ou suppression) indépendamment des applications clientes. Les applications clientes existantes doivent être en mesure de continuer à fonctionner sans modification au fil de l’évolution des fonctionnalités fournies par le service web. Toutes les fonctionnalités doivent également être détectables, de sorte que les applications clientes puissent les utiliser pleinement.

L’objectif de ce guide est de décrire les problèmes à prendre en compte lorsque vous concevez une API web.

## <a name="introduction-to-representational-state-transfer-rest"></a>Introduction à REST (Representational State Transfer)
Dans son étude de 2000, Roy Fielding propose une approche architecturale alternative à la structuration des opérations exposées par les services web : REST. REST est un style d’architecture pour la création de systèmes distribués basés sur l’hypermédia. L’un des principaux avantages du modèle REST est qu’il est basé sur des normes ouvertes et qu’il ne lie pas l’implémentation du modèle ou les applications clientes qui accèdent au modèle à une implémentation spécifique. Par exemple, un service web REST peut être implémenté à l’aide de l’API web Microsoft ASP.NET, et les applications clientes peuvent être développées à l’aide de n’importe quel langage et n’importe quel ensemble d’outils pouvant générer des requêtes HTTP et analyser les réponses HTTP.

> [!NOTE]
> REST est indépendant de tout protocole sous-jacent et n’est pas nécessairement lié à HTTP. Toutefois, la plupart des implémentations courantes de systèmes basés sur REST utilisent HTTP comme protocole d’application pour l’envoi et la réception de requêtes. Ce document est axé sur le mappage des principes REST aux systèmes conçus pour fonctionner à l’aide de HTTP.
>
>

Le modèle REST utilise un schéma de navigation pour représenter les objets et les services sur un réseau (les *ressources*). De nombreux systèmes qui implémentent REST utilisent généralement le protocole HTTP pour transmettre les demandes d’accès à ces ressources. Dans ces systèmes, une application cliente envoie une requête sous la forme d’un URI qui identifie une ressource, avec une méthode HTTP (les plus courantes étant GET, POST, PUT et DELETE) qui indique l’opération à effectuer sur cette ressource.  Le corps de la requête HTTP contient les données requises pour effectuer l’opération. Il est essentiel de bien comprendre que REST définit un modèle de requête sans état. Les requêtes HTTP doivent être indépendantes et peuvent être générées dans n’importe quel ordre. Par conséquent, les informations d’état temporaires entre les requêtes ne peuvent pas être conservées.  Les ressources elles-mêmes représentent l’unique emplacement de stockage de ces informations. Chaque requête doit être une opération atomique. En effet, un modèle REST implémente une machine à états finis où une requête fait passer une ressource d’un état non temporaire bien défini à un autre.

> [!NOTE]
> De par la nature sans état des requêtes individuelles du modèle REST, un système construit selon ces principes peut être hautement évolutif. Il est inutile de conserver une affinité entre une application cliente effectuant une série de requêtes et les serveurs web spécifiques qui gèrent ces requêtes.
>
>

Pour implémenter un modèle REST efficace, il est également crucial de comprendre les relations entre les différentes ressources auxquelles le modèle fournit l’accès. Ces ressources sont généralement organisées sous forme de collections et relations. Par exemple, supposons qu’une analyse rapide d’un système de commerce électronique montre que deux collections sont susceptibles d’intéresser des applications clientes : les commandes et les clients. Chaque commande et chaque client doivent posséder leur propre clé pour l’identification. L’URI permettant d’accéder à la collection de commandes peut être aussi simple que */orders*. De même, l’URI pour la récupération de tous les clients peut être */customers*. L’émission d’une requête HTTP GET à l’URI */orders* doit renvoyer une liste de toutes les commandes de la collection, encodée sous la forme d’une réponse HTTP :

```HTTP
GET http://adventure-works.com/orders HTTP/1.1
...
```

La réponse ci-dessous encode les commandes sous la forme d’une structure de liste JSON :

```HTTP
HTTP/1.1 200 OK
...
Date: Fri, 22 Aug 2014 08:49:02 GMT
Content-Length: ...
[{"orderId":1,"orderValue":99.90,"productId":1,"quantity":1},{"orderId":2,"orderValue":10.00,"productId":4,"quantity":2},{"orderId":3,"orderValue":16.60,"productId":2,"quantity":4},{"orderId":4,"orderValue":25.90,"productId":3,"quantity":1},{"orderId":5,"orderValue":99.90,"productId":1,"quantity":1}]
```
L’extraction d’une commande individuelle nécessite de spécifier l’identificateur de la commande à partir de la ressource *orders*, par exemple */orders/2* :

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
...
```

```HTTP
HTTP/1.1 200 OK
...
Date: Fri, 22 Aug 2014 08:49:02 GMT
Content-Length: ...
{"orderId":2,"orderValue":10.00,"productId":4,"quantity":2}
```

> [!NOTE]
> Par souci de simplicité, ces exemples illustrent les informations dans des réponses renvoyées sous forme de données de texte JSON. Toutefois, des ressources peuvent contenir d’autres types de données pris en charge par HTTP, par exemple des informations binaires ou chiffrées. L’en-tête Content-Type de la réponse HTTP doit spécifier le type. En outre, un modèle REST peut renvoyer les mêmes données dans différents formats, tels que XML ou JSON. Dans ce cas, le service web doit être en mesure d’effectuer la négociation de contenu avec le client qui effectue la requête. La requête peut inclure un en-tête *Accept* qui spécifie le format de réception privilégié par le client. Dans la mesure du possible, le service web doit alors tenter de respecter ce format.
>
>

Notez que la réponse liée à une requête REST utilise les codes d’état HTTP standard. Par exemple, une requête qui renvoie des données valides doit inclure le code de réponse HTTP 200 (OK), alors qu’une requête qui ne parvient pas à trouver ou supprimer une ressource spécifique doit renvoyer une réponse qui inclut le code d’état HTTP 404 (Introuvable).

## <a name="design-and-structure-of-a-restful-web-api"></a>Conception et structure d’une API web RESTful
La simplicité et la cohérence sont les clés de la réussite en matière de conception d’API web. Une API web offrant ces deux qualités simplifie la création des applications clientes qui devront l’utiliser.

Une API web RESTful repose principalement sur l’exposition d’un ensemble de ressources connectées et la fourniture des opérations principales qui permettent à une application de manipuler ces ressources et de naviguer facilement entre ces ressources. Par conséquent, les URI qui constituent une API web RESTful classique doivent être orientées vers les données qu’elle expose et utiliser les fonctionnalités fournies par HTTP pour agir sur ces données. Cette approche requiert une mentalité différente de celle généralement adoptée pour la conception d’un ensemble de classes dans une API orientée objet, dont la motivation porte plutôt sur le comportement des objets et des classes. En outre, une API web RESTful doit être sans état et indépendante des opérations appelées dans une séquence particulière. Les sections suivantes résument les points à prendre en compte lors de la conception d’une API web RESTful.

### <a name="organizing-the-web-api-around-resources"></a>Organisation de l’API web autour des ressources
> [!TIP]
> Les URI exposés par un service web REST doivent être basés sur des noms (les données auxquelles l’API web permet d’accéder) et non des verbes (ce qu’une application peut faire avec les données).
>
>

Concentrez-vous sur les entités métier exposées par l’API web. Par exemple, avec une API web conçue pour prendre en charge le système de commerce électronique décrit précédemment, les entités principales sont les clients et les commandes. Les processus tels que le passage d’une commande peuvent être assurés grâce à la fourniture d’une opération HTTP POST qui extrait les informations de commande et les ajoute à la liste de commandes du client. En interne, cette opération POST peut effectuer des tâches telles que la vérification des niveaux de stock et la facturation du client. La réponse HTTP peut indiquer si la commande a été passée avec succès ou non. Notez également qu’une ressource ne doit pas nécessairement être basée sur un élément de données physique unique. Par exemple, une ressource de commande peut être implémentée en interne à l’aide des informations agrégées à partir de nombreuses lignes réparties sur plusieurs tables d’une base de données relationnelle, mais présentée au client comme une entité unique.

> [!TIP]
> Évitez de concevoir une interface REST qui reflète ou dépend de la structure interne des données qu’elle expose. REST offre plus que l’implémentation de simples opérations CRUD (Create, Retrieve, Update, Delete) sur des tables distinctes d’une base de données relationnelle. REST a pour objet de mapper des entités métier et les opérations qu’une application peut effectuer sur ces entités à l’implémentation physique de ces entités, mais un client ne doit pas être exposé à ces détails physiques.
>
>

Les entités métier individuelles existent rarement de façon isolée (même s’il peut exister certains objets singleton). Elles sont généralement regroupées dans des collections. Dans le cadre de REST, toutes les entités et collections sont des ressources. Dans une API web RESTful, chaque collection a son propre URI dans le service web. Une requête HTTP GET sur l’URI d’une collection récupère une liste d’éléments de cette collection. Chaque élément individuel possède également son propre URI. Une application peut envoyer une autre requête HTTP GET avec cet URI pour récupérer les détails de l’élément. Vous devez organiser les URI des collections et éléments de façon hiérarchique. Dans le système de commerce électronique, l’URI */customers* désigne la collection du client, et */customers/5* récupère les détails du client unique avec l’ID 5 de cette collection. Cette approche contribue à préserver l’intuitivité de l’API web.

> [!TIP]
> Adoptez une convention d’affectation de noms cohérente pour les URI. En général, il est recommandé d’utiliser des noms au pluriel pour les URI qui référencent des collections.
>
>

Vous devez également prendre en compte les relations entre les différents types de ressources et la façon dont vous pouvez exposer ces associations. Par exemple, les clients peuvent ne passer aucune commande ou en passer plusieurs. L’utilisation d’un URI tel que */customers/5/orders* pour rechercher toutes les commandes du client 5 offre une façon naturelle de représenter cette relation. Vous pouvez également envisager de représenter l’association d’une commande associée à un client spécifique par le biais d’un URI tel que */orders/99/customer* pour rechercher le client auquel est associée la commande 99. Cependant, il n’est pas judicieux d’étendre ce modèle de façon excessive, car son implémentation deviendrait trop complexe. Une meilleure solution consiste à fournir des liens navigables vers des ressources associées, telles que le client, dans le corps du message de réponse HTTP renvoyé lorsque la commande fait l’objet d’une interrogation. Ce mécanisme est décrit plus en détail dans la section Utilisation de l’approche HATEOAS pour autoriser la navigation vers des ressources associées, plus loin dans ce guide.

Les systèmes plus complexes peuvent inclure des types d’entité beaucoup plus nombreux, et il peut être tentant de fournir des URI qui permettent à une application cliente de naviguer dans plusieurs niveaux de relations, par exemple */customers/1/orders/99/products*, pour obtenir la liste des produits de la commande 99 passée par le client 1. Toutefois, ce niveau de complexité peut être difficile à gérer et n’offre aucune flexibilité si les relations entre les ressources changent ultérieurement. Il est donc préférable de conserver des URI relativement simples. N’oubliez pas qu’une fois qu’une application a une référence à une ressource, il doit être possible d’utiliser cette référence pour rechercher des éléments liés à cette ressource. La requête précédente peut être remplacée par l’URI */customers/1/orders* pour rechercher toutes les commandes du client 1, puis interroger l’URI */orders/99/products* pour rechercher les produits de cette commande (en supposant que la commande 99 a été passée par le client 1).

> [!TIP]
> Évitez d’imposer des URI de ressource plus complexes que *collection/item/collection*.
>
>

Un autre point à prendre en considération est la charge imposée au serveur web par toutes les requêtes web. Plus le nombre de requêtes est élevé, plus la charge est importante. Vous devez tenter de définir vos ressources afin d’éviter les API web effectuant de nombreux échanges et qui exposent un grand nombre de ressources de petite taille. Une telle API peut nécessiter l’envoi de plusieurs requêtes par une application cliente pour rechercher toutes les données requises. Il peut être avantageux de dénormaliser des données et de combiner des informations connexes en ressources plus volumineuses, qui peuvent être récupérées à l’aide d’une seule requête. Toutefois, vous devez mesurer cette approche par rapport à la surcharge imposée par l’extraction de données qui ne sont peut-être pas fréquemment requises par le client. La récupération d’objets volumineux peut augmenter la latence d’une requête et entraîner des coûts supplémentaires liés à la bande passante, et ce, pour un avantage modeste si les données supplémentaires ne sont pas souvent utilisées.

Évitez d’introduire des dépendances entre l’API web et la structure, le type ou l’emplacement des sources de données sous-jacentes. Par exemple, si vos données se trouvent dans une base de données relationnelle, l’API web n’a pas besoin d’exposer chaque table comme une collection de ressources. Considérez l’API web comme une abstraction de la base de données et, si nécessaire, introduisez une couche de mappage entre la base de données et l’API web. Ainsi, si la conception ou l’implémentation de la base de données évolue (par exemple, si vous passez d’une base de données relationnelle contenant une collection de tables normalisées à un système de stockage NoSQL dénormalisé tel qu’une base de données de documents), les applications clientes ne sont pas affectées par ces modifications.

> [!TIP]
> La source de données sur laquelle s’appuie une API web ne doit pas nécessairement être un magasin de données. Il peut s’agir d’une autre application de service ou métier, ou même d’une application héritée exécutée localement dans une organisation.
>
>

Enfin, il n’est pas toujours possible de mapper chaque opération implémentée par une API web à une ressource spécifique. Vous pouvez gérer ces scénarios *sans ressource* par le biais de requêtes HTTP GET qui appellent une partie des fonctionnalités et renvoient les résultats dans un message de réponse HTTP. Une API web qui implémente des opérations de calcul simples, par exemple d’addition ou de soustraction, peut fournir des URI qui exposent ces opérations en tant que pseudo-ressources et utilisent la chaîne de requête pour spécifier les paramètres requis. Par exemple une requête GET à l’URI */add?operand1=99&operand2=1* peut renvoyer un message de réponse dont le corps contenant la valeur 100, et une requête GET à l’URI */subtract?operand1=50&operand2=20* peut renvoyer un message de réponse dont le corps contient la valeur 30. Toutefois, utilisez ces formes d’URI avec parcimonie.

### <a name="defining-operations-in-terms-of-http-methods"></a>Définition d’opérations en termes de méthodes HTTP
Le protocole HTTP définit un certain nombre de méthodes qui affectent une signification sémantique à une requête. Les méthodes HTTP utilisées par la plupart des API web RESTful sont les suivantes :

* **GET**pour récupérer une copie de la ressource à l’URI spécifié. Le corps du message de réponse contient les détails de la ressource demandée.
* **POST**pour créer une ressource à l’URI spécifié. Le corps du message de requête fournit les détails de la nouvelle ressource. Notez que POST permet également de déclencher des opérations qui ne créent pas réellement de ressources.
* **PUT**pour remplacer ou mettre à jour la ressource à l’URI spécifié. Le corps du message de requête spécifie la ressource à modifier et les valeurs à appliquer.
* **DELETE**pour supprimer la ressource à l’URI spécifié.

> [!NOTE]
> Le protocole HTTP définit également d’autres méthodes moins couramment utilisées, notamment la méthode PATCH pour demander des mises à jour sélectives pour une ressource, la méthode HEAD, pour demander une description d’une ressource, la méthode OPTIONS qui permet à une application cliente d’obtenir des informations sur les options de communication prises en charge par le serveur et la méthode TRACE, qui permet à un client de demander des informations qu’il peut utiliser à des fins de test et de diagnostic.
>
>

L’effet d’une requête spécifique sera différent, selon que la ressource à laquelle elle est appliquée est une collection ou un élément individuel. Le tableau suivant résume les conventions courantes adoptées par la plupart des implémentations RESTful suivant l’exemple du commerce électronique. Notez que certaines de ces requêtes ne peuvent pas être implémentées ; cela dépend du scénario spécifique.

| **Ressource** | **POST** | **GET** | **PUT** | **DELETE** |
| --- | --- | --- | --- | --- |
| /customers |Créer un client |Récupérer tous les clients |Mettre à jour des clients en bloc (*si implémentée*) |Supprimer tous les clients |
| /customers/1 |Erreur |Récupérer les détails du client 1 |Mettre à jour les détails du client 1 s’il existe, sinon renvoyer une erreur |Supprimer le client 1 |
| /customers/1/orders |Créer une commande pour le client 1 |Récupérer toutes les commandes pour le client 1 |Mettre à jour des commandes en bloc pour le client 1 (*si implémentée*) |Supprimer toutes les commandes du client 1 (*si implémentée*) |

Le rôle des requêtes GET et DELETE est relativement simple. Cependant, le rôle et les effets des requêtes POST et PUT peuvent prêter à confusion.

Une requête POST doit créer une ressource avec les données fournies dans le corps de la requête. Dans le cadre du modèle REST, vous appliquez fréquemment des requêtes POST à des ressources qui sont des collections ; la nouvelle ressource est ajoutée à la collection.

> [!NOTE]
> Vous pouvez également définir des requêtes POST qui déclenchent certaines fonctionnalités (et qui ne renvoient pas nécessairement des données). Ces types de requête peuvent être appliqués aux collections. Par exemple, vous pouvez utiliser une requête POST pour transmettre une feuille de temps à un service de traitement de paie et obtenir en réponse les taxes calculées.
>
>

Une requête PUT est destinée à modifier une ressource existante. Si la ressource spécifiée n’existe pas, la requête PUT peut renvoyer une erreur (dans certains cas, elle peut créer la ressource). Les requêtes PUT sont généralement appliquées aux ressources qui sont des éléments individuels (par exemple, un client ou une commande spécifique). Elles peuvent cependant être appliquées à des collections, même si ce type d’implémentation est moins fréquent. Notez que les requêtes PUT sont idempotentes, à la différence des requêtes POST ; si une application envoie la même requête PUT plusieurs fois, les résultats seront toujours les mêmes (la même ressource sera modifiée avec les mêmes valeurs), mais si une application répète la même requête POST, plusieurs ressources seront créées.

> [!NOTE]
> À proprement parler, une requête HTTP PUT remplace une ressource existante par la ressource spécifiée dans le corps de la requête. Si l’intention est de modifier une sélection de propriétés d’une ressource tout en conservant les autres propriétés inchangées, une requête HTTP PATCH doit être utilisée. Toutefois, de nombreuses implémentations RESTful assouplissent cette règle et utilisent une requête PUT dans les deux cas.
>
>

### <a name="processing-http-requests"></a>Traitement des requêtes HTTP
Les données incluses par une application cliente dans de nombreuses requêtes HTTP et les messages de réponse correspondants envoyés par le serveur web peuvent être présentés dans divers formats (ou types de médias). Par exemple, les données qui spécifient les détails d’une commande ou d’un client peuvent être fournies au format XML ou JSON, ou dans un autre format encodé et compressé. Une API web RESTful doit également prendre en charge les différents types de média demandés par l’application cliente qui envoie une requête.

Lorsqu’une application cliente envoie une requête qui renvoie des données dans le corps d’un message, elle peut spécifier les types de média qu’elle peut gérer dans l’en-tête Accept de la requête. Le code ci-après illustre une requête HTTP GET qui récupère les détails de la commande 2 et indique que le résultat doit être renvoyé au format JSON (le client doit toujours examiner le type de média des données dans la réponse pour vérifier le format des données renvoyées) :

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
...
Accept: application/json
...
```

Si le serveur web prend en charge ce type de média, il peut renvoyer une réponse dont l’en-tête Content-Type spécifie le format des données dans le corps du message :

> [!NOTE]
> Pour une interopérabilité maximale, les types de média référencés dans les en-têtes Accept et Content-Type doivent être des types MIME reconnus plutôt que des types de média personnalisés.
>
>

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"orderID":2,"productID":4,"quantity":2,"orderValue":10.00}
```

Si le serveur web ne prend pas en charge le type de média demandé, il peut envoyer les données dans un format différent. Dans tous les cas, il doit spécifier le type de média (par exemple, *application/json*) dans l’en-tête Content-Type. Il incombe à l’application cliente d’analyser le message de réponse et d’interpréter les résultats dans le corps du message de manière appropriée.

Notez que dans cet exemple, le serveur web récupère correctement les données demandées et indique la réussite en repassant un code d’état 200 dans l’en-tête de la réponse. Si aucune donnée correspondante n’est trouvée, il doit renvoyer un code d’état 404 (Introuvable), et le corps du message de réponse peut contenir des informations supplémentaires. Le format de ces informations est spécifié par l’en-tête Content-Type, comme illustré dans l’exemple suivant :

```HTTP
GET http://adventure-works.com/orders/222 HTTP/1.1
...
Accept: application/json
...
```

La commande 222 n’existe pas. Le message de réponse ressemble donc à ceci :

```HTTP
HTTP/1.1 404 Not Found
...
Content-Type: application/json; charset=utf-8
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"message":"No such order"}
```

Lorsqu’une application envoie une requête HTTP PUT pour mettre à jour une ressource, elle spécifie l’URI de la ressource et fournit les données à modifier dans le corps du message de requête. Elle doit également spécifier le format de ces données à l’aide de l’en-tête Content-Type. Le format *application/x-www-form-urlencoded* est souvent utilisé pour les informations textuelles. Il comprend un ensemble de paires nom/valeur séparées par le caractère &. L’exemple suivant illustre une requête HTTP PUT qui modifie les informations de la commande 1 :

```HTTP
PUT http://adventure-works.com/orders/1 HTTP/1.1
...
Content-Type: application/x-www-form-urlencoded
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
ProductID=3&Quantity=5&OrderValue=250
```

Si la modification est réussie, le serveur doit idéalement répondre avec un code d’état HTTP 204 indiquant que le processus a été géré correctement mais que le corps de la réponse ne contient aucune information supplémentaire. L’en-tête Location de la réponse contient l’URI de la ressource récemment mise à jour :

```HTTP
HTTP/1.1 204 No Content
...
Location: http://adventure-works.com/orders/1
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
```

> [!TIP]
> Si les données d’un message de requête HTTP PUT incluent des informations de date et d’heure, assurez-vous que votre service web accepte les dates et heures au format de la norme ISO 8601.
>
>

Si la ressource à mettre à jour n’existe pas, le serveur web peut répondre avec une réponse Introuvable, comme décrit précédemment. Si le serveur crée l’objet lui-même, il peut renvoyer les codes d’état HTTP 200 (OK) ou HTTP 201 (Créé) et le corps de la réponse peut contenir les données de la nouvelle ressource. Si l’en-tête Content-Type de la requête spécifie un format de données que le serveur web ne peut pas gérer, il doit répondre avec le code d’état HTTP 415 (Type de support non pris en charge).

> [!TIP]
> Envisagez d’implémenter des opérations HTTP PUT en bloc, qui peuvent regrouper des mises à jour destinées à plusieurs ressources d’une collection. La requête PUT doit spécifier l’URI de la collection, et le corps de la requête doit spécifier les détails des ressources à modifier. Cette approche peut aider à réduire les échanges excessifs et à améliorer les performances.
>
>

Le format d’une requête HTTP POST créant des ressources est semblable à celui des requêtes PUT ; le corps du message contient les détails de la nouvelle ressource à ajouter. Toutefois, l’URI spécifie généralement la collection à laquelle la ressource doit être ajoutée. L’exemple suivant crée une commande et l’ajoute à la collection de commandes :

```HTTP
POST http://adventure-works.com/orders HTTP/1.1
...
Content-Type: application/x-www-form-urlencoded
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
productID=5&quantity=15&orderValue=400
```

Si la requête réussit, le serveur web doit répondre avec un message possédant le code d’état HTTP 201 (Créé). L’en-tête Location doit contenir l’URI de la ressource nouvellement créée, et le corps de la réponse doit contenir une copie de la nouvelle ressource. L’en-tête Content-Type spécifie le format de ces données :

```HTTP
HTTP/1.1 201 Created
...
Content-Type: application/json; charset=utf-8
Location: http://adventure-works.com/orders/99
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"orderID":99,"productID":5,"quantity":15,"orderValue":400}
```

> [!TIP]
> Si les données fournies par une requête PUT ou POST ne sont pas valides, le serveur web doit répondre avec un message possédant le code d’état HTTP 400 (Requête incorrecte). Le corps de ce message peut contenir des informations supplémentaires sur le problème avec la requête et les formats attendus ou un lien vers une URL qui fournit plus de détails.
>
>

Pour supprimer une ressource, une requête HTTP DELETE fournit simplement l’URI de la ressource à supprimer. L’exemple suivant tente de supprimer la commande 99 :

```HTTP
DELETE http://adventure-works.com/orders/99 HTTP/1.1
...
```

Si l’opération de suppression réussit, le serveur web doit répondre avec un code d’état HTTP 204, indiquant que le processus a été géré correctement, mais que le corps de la réponse ne contient aucune information supplémentaire (il s’agit de la même réponse renvoyée par une opération PUT ayant réussi, mais sans en-tête Location car la ressource n’existe plus.) Une requête DELETE peut également renvoyer le code d’état HTTP 200 (OK) ou 202 (Accepté) si la suppression est exécutée de façon asynchrone.

```HTTP
HTTP/1.1 204 No Content
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
```

Si la ressource est introuvable, le serveur web doit renvoyer un message 404 (Introuvable).

> [!TIP]
> Si toutes les ressources d’une collection doivent être supprimées, autorisez la spécification d’une requête HTTP DELETE pour l’URI de la collection, et ne forcez pas une application à supprimer successivement chaque ressource de la collection.
>
>

### <a name="filtering-and-paginating-data"></a>Filtrage et pagination des données
Vous devez vous efforcer de garder les URI simples et intuitifs. L’exposition d’une collection de ressources par le biais d’un seul URI peut y contribuer. Cependant, dans ce cas, il se peut que les applications extraient de grandes quantités de données alors que seul un sous-ensemble d’informations est requis. La génération d’un grand volume de trafic affecte non seulement les performances et l’évolutivité du serveur web mais également la réactivité des applications clientes qui demandent les données.

Par exemple, si les commandes contiennent le prix d’achat de la commande, une application cliente qui a besoin de récupérer toutes les commandes dont le coût est supérieur à une valeur spécifique devra peut-être récupérer toutes les commandes à partir de l’URI */orders*, puis filtrer ces commandes localement. Ce processus est très peu efficace. Il gaspille la bande passante réseau et la puissance de traitement du serveur hébergeant l’API web.

Une solution consiste à fournir un modèle d’URI tel que */orders/ordervalue_greater_than_n* où *n* est le prix de la commande. Toutefois, hormis pour un nombre limité de prix, une telle approche présente des inconvénients. En outre, si vous devez interroger des commandes selon d’autres critères, vous pouvez être tenu de fournir une longue liste d’URI contenant des noms potentiellement non intuitifs.

Pour le filtrage des données, une meilleure stratégie consiste à fournir des critères de filtre dans la chaîne de requête qui est passée à l’API web, par exemple */orders?ordervaluethreshold=n*. Dans cet exemple, l’opération correspondante dans l’API web est responsable de l’analyse et de la gestion du paramètre `ordervaluethreshold` dans la chaîne de requête et du renvoi des résultats filtrés dans la réponse HTTP.

Certaines requêtes HTTP GET simples sur les ressources de la collection peuvent renvoyer un grand nombre d’éléments. Pour réduire ce risque, vous devez concevoir l’API web de façon à limiter la quantité de données renvoyées par chaque requête. Pour cela, vous pouvez prendre en charge les chaînes de requête permettant aux utilisateurs de spécifier le nombre maximal d’éléments à récupérer (qui peut lui-même être soumis à une limite supérieure pour contribuer à empêcher les attaques par déni de service) et un décalage de départ dans la collection. Par exemple, la chaîne de requête dans l’URI */orders?limit=25&offset=50* doit récupérer 25 commandes à partir de la cinquantième commande trouvée dans la collection. Comme avec le filtrage des données, l’opération qui implémente la requête GET dans l’API web est responsable de l’analyse et de la gestion des paramètres `limit` et `offset` dans la chaîne de requête. Pour aider les applications clientes, les requêtes GET qui renvoient les données paginées doivent également inclure des métadonnées qui indiquent le nombre total de ressources disponibles dans la collection. Vous pouvez également envisager d’autres stratégies de pagination intelligente. Pour plus d’informations, consultez [API Design Notes: Smart Paging](http://bizcoder.com/api-design-notes-smart-paging) (Notes de conception d’API : pagination intelligente).

Vous pouvez suivre une stratégie similaire pour trier les données au fil de leur extraction. Vous pouvez fournir un paramètre de tri qui utilise un nom de champ comme valeur, tel que */orders?sort=ProductID*. Toutefois, notez que cette approche peut avoir un effet nuisible sur la mise en cache (les paramètres de chaîne de requête font partie de l’identificateur de ressource utilisé par de nombreuses implémentations de cache comme clé pour les données mises en cache).

Vous pouvez étendre cette approche pour limiter (projeter) les champs renvoyés si un élément de ressource unique contient une grande quantité de données. Vous pouvez notamment utiliser un paramètre de chaîne de requête qui accepte une liste de champs séparés par des virgules, par exemple */orders?fields=ProductID,Quantity*.

> [!TIP]
> Affectez des valeurs par défaut significatives à tous les paramètres facultatifs des chaînes de requête. Par exemple, définissez le paramètre `limit` sur 10 et le paramètre `offset` sur 0 si vous implémentez la pagination, définissez le paramètre de tri sur la clé de la ressource si vous implémentez le classement et définissez le paramètre `fields` pour tous les champs de la ressource si vous prenez en charge des projections.
>
>

### <a name="handling-large-binary-resources"></a>Gestion de ressources binaires volumineuses
Une ressource unique peut contenir des champs binaires volumineux, tels que des fichiers ou des images. Pour remédier aux problèmes de transmission provoqués par des connexions non fiables et intermittentes et pour améliorer les temps de réponse, envisagez de fournir des opérations qui permettent la récupération de ces ressources en blocs par l’application cliente. Pour ce faire, l’API web doit prendre en charge l’en-tête Accept-Ranges pour les requêtes GET en présence de ressources volumineuses et, dans l’idéal, implémenter les requêtes HTTP HEAD pour ces ressources. L’en-tête Accept-Ranges indique que l’opération GET prend en charge les résultats partiels et qu’une application cliente peut envoyer des requêtes GET qui renvoient un sous-ensemble d’une ressource spécifiée sous forme de plage d’octets. Une requête HEAD est similaire à une requête GET à ceci près qu’elle renvoie uniquement un en-tête qui décrit la ressource et un corps de message vide. Une application cliente peut émettre une requête HEAD pour déterminer s’il faut extraire une ressource à l’aide de requêtes GET partielles. L’exemple suivant illustre une requête HEAD qui obtient des informations sur une image de produit :

```HTTP
HEAD http://adventure-works.com/products/10?fields=productImage HTTP/1.1
...
```

Le message de réponse contient un en-tête qui inclut la taille de la ressource (4580 octets), et l’en-tête Accept-Ranges indiquant que l’opération GET correspondante prend en charge les résultats partiels :

```HTTP
HTTP/1.1 200 OK
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 4580
...
```

L’application cliente peut utiliser ces informations pour construire une série d’opérations GET afin de récupérer l’image en blocs plus petits. La première requête extrait les 2500 premiers octets à l’aide de l’en-tête Range :

```HTTP
GET http://adventure-works.com/products/10?fields=productImage HTTP/1.1
Range: bytes=0-2499
...
```

Le message de réponse indique qu’il s’agit d’une réponse partielle en renvoyant le code d’état HTTP 206. L’en-tête Content-Length indique le nombre réel d’octets renvoyés dans le corps du message (pas la taille de la ressource), et l’en-tête Content-Range indique de quelle partie de la ressource il s’agit (octets 0-2499 sur 4580) :

```HTTP
HTTP/1.1 206 Partial Content
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2500
Content-Range: bytes 0-2499/4580
...
_{binary data not shown}_
```

Une demande ultérieure de l’application cliente peut extraire le reste de la ressource à l’aide d’un en-tête Range approprié :

```HTTP
GET http://adventure-works.com/products/10?fields=productImage HTTP/1.1
Range: bytes=2500-
...
```

Le message de résultat correspondant doit ressembler à ceci :

```HTTP
HTTP/1.1 206 Partial Content
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2080
Content-Range: bytes 2500-4580/4580
...
```

## <a name="using-the-hateoas-approach-to-enable-navigation-to-related-resources"></a>Utilisation de l’approche HATEOAS pour autoriser la navigation vers des ressources associées
L’utilisation de REST est principalement motivée par la possibilité de naviguer dans l’ensemble des ressources sans connaissance préalable du modèle d’URI. Chaque requête HTTP GET doit renvoyer les informations nécessaires pour trouver les ressources liées directement à l’objet demandé par le biais de liens hypertexte inclus dans la réponse. Des informations décrivant les opérations disponibles sur chacune de ces ressources doivent également être fournies. Il s’agit du principe HATEOAS (Hypertext as the Engine of Application State). Le système est effectivement une machine à états finis, et la réponse à chaque requête contient les informations nécessaires pour passer d’un état à l’autre ; aucune autre information n’est nécessaire.

> [!NOTE]
> Il n’existe actuellement aucune norme ou spécification définissant comment modéliser le principe HATEOAS. Les exemples présentés dans cette section illustrent une solution possible.
>
>

Par exemple, pour gérer la relation entre les clients et les commandes, les données renvoyées dans la réponse pour une commande spécifique doivent contenir les URI sous la forme d’un lien hypertexte qui identifie le client qui a passé la commande et les opérations qui peuvent être effectuées sur ce client.

```HTTP
GET http://adventure-works.com/orders/3 HTTP/1.1
Accept: application/json
...
```

Le corps du message de réponse contient un tableau `links` (mis en évidence dans l’exemple de code) qui spécifie la nature de la relation (*Customer*), l’URI du client (*http://adventure-works.com/customers/3*), le mode de récupération des détails de ce client (*GET*) et les types MIME que le serveur Web prend en charge pour la récupération de ces informations (*text/xml* et *application/json*). Il s’agit de toutes les informations nécessaires à une application cliente pour être en mesure d’extraire les détails du client. En outre, le tableau Links inclut également des liens pour les autres opérations qui peuvent être effectuées, telles que PUT (pour modifier le client, ainsi que le format que le serveur web attend du client) et DELETE.

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"orderID":3,"productID":2,"quantity":4,"orderValue":16.60,"links":[(some links omitted){"rel":"customer","href":" http://adventure-works.com/customers/3", "action":"GET","types":["text/xml","application/json"]},{"rel":"
customer","href":" http://adventure-works.com /customers/3", "action":"PUT","types":["application/x-www-form-urlencoded"]},{"rel":"customer","href":" http://adventure-works.com /customers/3","action":"DELETE","types":[]}]}
```

Pour être complet, le tableau Links doit également inclure des informations d’auto-référencement se rapportant à la ressource récupérée. Ces liens ont été omis dans l’exemple précédent, mais sont mis en évidence dans le code suivant. Notez que dans ces liens, la relation *self* a été utilisée pour indiquer qu’il s’agit d’une référence à la ressource renvoyée par l’opération :

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"orderID":3,"productID":2,"quantity":4,"orderValue":16.60,"links":[{"rel":"self","href":" http://adventure-works.com/orders/3", "action":"GET","types":["text/xml","application/json"]},{"rel":" self","href":" http://adventure-works.com /orders/3", "action":"PUT","types":["application/x-www-form-urlencoded"]},{"rel":"self","href":" http://adventure-works.com /orders/3", "action":"DELETE","types":[]},{"rel":"customer",
"href":" http://adventure-works.com /customers/3", "action":"GET","types":["text/xml","application/json"]},{"rel":" customer" (customer links omitted)}]}
```

Pour que cette approche soit efficace, les applications clientes doivent être préparées pour récupérer et analyser ces informations supplémentaires.

## <a name="versioning-a-restful-web-api"></a>Contrôle de version d’une API web RESTful
Il est peu probable qu’une API web reste statique, hormis dans les situations les plus simples. Au fil de l’évolution des besoins de l’entreprise, de nouvelles collections de ressources peuvent être ajoutées, les relations entre les ressources peuvent changer et la structure des données des ressources peut être modifiée. La mise à jour d’une API web pour gérer les exigences nouvelles ou différentes est un processus relativement simple. Cependant, vous devez considérer les effets de ces modifications sur les applications clientes utilisant l’API web. Le problème est le suivant : même si le développeur qui conçoit et implémente une API web bénéficie d’un contrôle total sur cette API, il n’a pas le même degré de contrôle sur les applications clientes qui peuvent être créées par des organisations tierces distantes. L’impératif principal consiste à faire en sorte que les applications clientes existantes puissent continuer à fonctionner sans modification tout en autorisant les nouvelles applications clientes à tirer parti des nouvelles fonctionnalités et ressources.

Le contrôle de version permet à une API web d’indiquer les fonctionnalités et ressources qu’elle expose. Une application cliente peut alors envoyer des requêtes destinées à une version spécifique d’une fonctionnalité ou d’une ressource. Les sections suivantes décrivent différentes approches présentant chacune des avantages et des inconvénients.

### <a name="no-versioning"></a>Aucun contrôle de version
Il s’agit de l’approche la plus simple. Elle peut être acceptable pour certaines API internes. De nouvelles ressources ou de nouveaux liens peuvent représenter des changements importants.  L’ajout de contenu à des ressources existantes ne représente pas nécessairement une modification avec rupture, dans la mesure où les applications clientes qui n’attendent pas ce contenu l’ignoreront simplement.

Par exemple, une requête à l’URI *http://adventure-works.com/customers/3* doit renvoyer les détails d’un client unique contenant les champs `id`, `name` et `address` attendus par l’application cliente :

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

> [!NOTE]
> À des fins de simplicité et de clarté, les exemples de réponses présentés dans cette section n’incluent pas les liens HATEOAS.
>
>

Si le champ `DateCreated` est ajouté au schéma de la ressource du client, la réponse se présentera comme suit :

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":"1 Microsoft Way Redmond WA 98053"}
```

Les applications clientes existantes peuvent continuer à fonctionner correctement si elles peuvent ignorer les champs non reconnus. Les nouvelles applications clientes peuvent être conçues pour gérer ce nouveau champ. Cependant, des modifications plus importantes affectant le schéma des ressources (par exemple, la suppression ou la modification de nom de champs), ou un changement des relations entre les ressources peuvent constituer des modifications avec rupture empêchant les applications clientes existantes de fonctionner correctement. Dans ces situations, vous devez envisager l’une des approches suivantes.

### <a name="uri-versioning"></a>Contrôle de version d’URI
Dès lors que vous modifiez l’API web ou le schéma des ressources, vous ajoutez un numéro de version à l’URI pour chaque ressource. Les URI existants doivent continuer à fonctionner comme avant et à renvoyer les ressources conformes à leur schéma d’origine.

Si l’on étend l’exemple précédent, si le champ `address` est restructuré dans des sous-champs contenant chaque partie de l’adresse (par exemple, `streetAddress`, `city`, `state` et `zipCode`), cette version de la ressource peut être exposée par le biais d’un URI qui contient un numéro de version, tel que http://adventure-works.com/v2/customers/3 :

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

Ce mécanisme de contrôle de version est très simple, mais dépend du routage de la requête par le serveur au point de terminaison approprié. Toutefois, il peut devenir complexe à mesure que l’API web arrive à maturité par le biais de plusieurs itérations et que le nombre de versions différentes prises en charge par le serveur évolue. En outre, d’un point de vue puriste, les applications clientes extraient les mêmes données (client 3) dans tous les cas. L’URI ne devrait donc pas être différent selon la version. Ce modèle complique également l’implémentation de HATEOAS, dans la mesure où tous les liens devront inclure le numéro de version dans leurs URI.

### <a name="query-string-versioning"></a>Contrôle de version de chaîne de requête
Plutôt que de fournir plusieurs URI, vous pouvez spécifier la version de la ressource en utilisant un paramètre dans la chaîne de requête ajoutée à la requête HTTP, par exemple *http://adventure-works.com/customers/3?version=2*. La valeur par défaut du paramètre de version doit être significative, par exemple 1, s’il est omis par des applications clientes plus anciennes.

D’un point de vue sémantique, cette approche présente l’avantage suivant : la même ressource est toujours extraite du même URI, mais cela dépend du code qui gère la demande d’analyse de la chaîne de requête et de renvoi de la réponse HTTP appropriée. D’autre part, cette approche présente les mêmes inconvénients que le mécanisme de contrôle de version d’URI concernant l’implémentation de HATEOAS.

> [!NOTE]
> Certains proxys et navigateurs web anciens ne mettent pas en cache les réponses aux requêtes qui incluent une chaîne de requête dans l’URL. Cela peut avoir un impact négatif sur les performances des applications web qui utilisent une API web et qui s’exécutent à partir des navigateurs web.
>
>

### <a name="header-versioning"></a>Contrôle de version d’en-tête
Plutôt que d’ajouter le numéro de version en tant que paramètre dans la chaîne de requête, vous pouvez implémenter un en-tête personnalisé qui indique la version de la ressource. Cette approche nécessite que l’application cliente ajoute l’en-tête approprié à toutes les requêtes, même si le code qui gère la requête client peut utiliser une valeur par défaut (version 1) si l’en-tête de version est omis. Les exemples suivants utilisent un en-tête personnalisé nommé *Custom-Header*. La valeur de cet en-tête indique la version de l’API web.

Version 1 :

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Custom-Header: api-version=1
...
```

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

Version 2 :

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Custom-Header: api-version=2
...
```

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

Notez que, comme avec les deux approches précédentes, l’implémentation de HATEOAS nécessite l’inclusion de l’en-tête personnalisé approprié dans tous les liens.

### <a name="media-type-versioning"></a>Contrôle de version du type de média
Lorsqu’une application cliente envoie une requête HTTP GET à un serveur web, elle doit indiquer le format du contenu qu’elle peut traiter à l’aide d’un en-tête Accept, comme décrit précédemment dans ce guide. Souvent, l’en-tête *Accept* a pour rôle de permettre à l’application cliente de spécifier si le corps de la réponse doit être au format XML ou JSON, ou dans un autre format courant que le client peut analyser. Toutefois, il est possible de définir des types de médias personnalisés qui incluent des informations permettant à l’application cliente d’indiquer la version de ressource attendue. L’exemple suivant illustre une requête qui spécifie un en-tête *Accept* avec la valeur *application/vnd.adventure-works.v1+json*. L’élément *vnd.adventure-works.v1* indique au serveur web qu’il doit renvoyer la version 1 de la ressource, tandis que l’élément *json* spécifie que le format du corps de la réponse doit être JSON :

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Accept: application/vnd.adventure-works.v1+json
...
```

Le code qui gère la requête est responsable du traitement de l’en-tête *Accept* et doit le respecter autant que possible (l’application cliente peut spécifier plusieurs formats dans l’en-tête *Accept*, auquel cas le serveur web peut choisir le format le plus approprié pour le corps de la réponse). Le serveur web vérifie le format des données dans le corps de la réponse à l’aide de l’en-tête Content-Type :

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/vnd.adventure-works.v1+json; charset=utf-8
...
Content-Length: ...
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
* L’article [Microsoft REST API Guidelines (Directives relatives aux API REST Microsoft)](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md) contient des recommandations détaillées concernant la conception d’API REST publiques.
* Le site [RESTful Cookbook](http://restcookbook.com/) (en anglais) offre une introduction à la création d’API RESTful.
* L’article [Web API Checklist (Liste de contrôle pour les API Web)](https://mathieu.fenniak.net/the-api-checklist/) présente une liste d’éléments à prendre en compte lors de la conception et de l’implémentation d’une API Web.
* Le site [Open API Initiative](https://www.openapis.org/) contient l’ensemble de la documentation associée et tous les détails d’implémentation sur Open API.
