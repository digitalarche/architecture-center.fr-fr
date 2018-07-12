---
title: Application web scalable
description: Amélioration de la scalabilité dans une application web s’exécutant dans Microsoft Azure.
author: MikeWasson
pnp.series.title: Azure App Service
pnp.series.prev: basic-web-app
pnp.series.next: multi-region-web-app
ms.date: 11/23/2016
cardTitle: Improve scalability
ms.openlocfilehash: 6459acebfa25491332e2118b9e8fe51d5fc79ff3
ms.sourcegitcommit: 5d99b195388b7cabba383c49a81390ac48f86e8a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/10/2018
ms.locfileid: "37958804"
---
# <a name="improve-scalability-in-a-web-application"></a>Améliorer la scalabilité dans une application web

Cette architecture de référence présente des pratiques éprouvées pour améliorer la scalabilité et le niveau de performance d’une application web Azure App Service.

![[0]][0]

*Téléchargez un [fichier Visio][visio-download] de cette architecture.*

## <a name="architecture"></a>Architecture  

Cette architecture repose sur celle décrite dans l’article [Application web de base][basic-web-app]. Ses composants sont les suivants :

* **Groupe de ressources**. Un [groupe de ressources][resource-group] est un conteneur logique pour les ressources Azure.
* **[Application web][app-service-web-app]** et **[application API][app-service-api-app]**. Une application moderne classique peut inclure un site web et une ou plusieurs API web RESTful. Une API web peut être consommée par les clients de navigateur par le biais d’AJAX, par les applications clientes natives ou par les applications côté serveur. Pour des considérations sur la conception d’API web, consultez [Guide de conception d’API][api-guidance].    
* **Tâche web**. Utilisez des [tâches web Azure][webjobs] pour exécuter des tâches longues en arrière-plan. Les tâches web peuvent s’exécuter selon une planification, en continu ou en réponse à un déclencheur, tel que le placement d’un message dans une file d’attente. Une tâche web s’exécute en tant que processus en arrière-plan dans le contexte d’une application App Service.
* **File d’attente**. Dans l’architecture illustrée ici, l’application met en file d’attente des tâches exécutées en arrière-plan en plaçant un message dans la file d’attente [Stockage File d’attente Azure][queue-storage]. Le message déclenche une fonction dans la tâche web. Vous pouvez également utiliser des files d’attente Service Bus. Pour obtenir une comparaison, consultez [Files d’attente Azure et files d’attente Service Bus : comparaison et différences][queues-compared].
* **Cache**. Stockez les données semi-statiques dans [Cache Redis Azure][azure-redis].  
* <strong>CDN</strong>. Utilisez [Azure Content Delivery Network][azure-cdn] (CDN) pour mettre en cache le contenu disponible publiquement afin de réduire la latence et d’accélérer la remise du contenu.
* **Stockage des données**. Utilisez [Azure SQL Database][sql-db] pour les données relationnelles. Pour les données non relationnelles, envisagez un magasin NoSQL, tel que [Cosmos DB][cosmosdb].
* **Recherche Azure**. Utilisez [Recherche Azure][azure-search] pour ajouter des fonctionnalités de recherche telles que les suggestions de recherche, la recherche partielle et la recherche spécifique à une langue. Recherche Azure est généralement utilisé conjointement avec un autre magasin de données, surtout si le magasin de données principal requiert la cohérence stricte. Dans cette approche, stockez les données faisant autorité dans l’autre magasin de données et l’index de recherche dans Recherche Azure. Recherche Azure peut également servir à consolider un index de recherche unique à partir de plusieurs magasins de données.  
* **E-mail/SMS**. Utilisez un service tiers tel que SendGrid ou Twilio pour envoyer des e-mails ou SMS au lieu de générer cette fonctionnalité directement dans l’application.
* **Azure DNS**. [Azure DNS][azure-dns] est un service d’hébergement pour les domaines DNS qui offre une résolution de noms à l’aide de l’infrastructure Microsoft Azure. En hébergeant vos domaines dans Azure, vous pouvez gérer vos enregistrements DNS avec les mêmes informations d’identification, les mêmes API, les mêmes outils et la même facturation que vos autres services Azure.

## <a name="recommendations"></a>Recommandations

Vos exigences peuvent différer de celles de l’architecture décrite ici. Utilisez les recommandations de cette section comme point de départ.

### <a name="app-service-apps"></a>Applications App Service
Nous vous recommandons de créer l’application web et l’API web en tant qu’applications App Service distinctes. Vous pouvez ainsi les exécuter dans des plans App Service distincts et donc les mettre à l’échelle indépendamment l’une de l’autre. Si vous n’avez pas besoin de ce niveau de scalabilité dès le départ, vous pouvez déployer les applications sur le même plan et les déplacer ultérieurement dans des plans distincts si cela est nécessaire.

> [!NOTE]
> Pour les plans De base, Standard et Premium, vous êtes facturé pour les instances de machine virtuelle dans le plan, pas par application. Consultez [Tarification App Service][app-service-pricing].
> 
> 

Si vous envisagez d’utiliser les fonctionnalités *Tables faciles* ou *API faciles* d’App Service Mobile Apps, créez une application App Service distincte à cet effet.  L’activation de ces fonctionnalités repose sur une infrastructure d’application spécifique.

### <a name="webjobs"></a>WebJobs
Envisagez de déployer les tâches web gourmandes en ressources sur une application App Service vide au sein d’un plan App Service distinct. Les tâches web disposent ainsi d’instances dédiées. Consultez [Conseils sur les travaux en arrière-plan][webjobs-guidance].  

### <a name="cache"></a>Cache
Vous pouvez améliorer le niveau de performance et la scalabilité en utilisant [Cache Redis Azure][azure-redis] pour mettre en cache des données. Envisagez d’utiliser Cache Redis pour :

* Les données de transaction semi-statiques.
* L’état de la session.
* La sortie HTML. Cela peut être utile dans les applications qui restituent une sortie HTML complexe.

Pour plus d’informations sur la conception d’une stratégie de mise en cache, consultez [Conseils sur la mise en cache][caching-guidance].

### <a name="cdn"></a>CDN
Utilisez [Azure CDN][azure-cdn] pour mettre en cache du contenu statique. L’avantage principal d’un CDN est qu’il réduit la latence pour l’utilisateur ; en effet, le contenu est mis en cache sur un serveur Edge de périphérie qui est géographiquement proche de l’utilisateur. CDN peut également réduire la charge qui pèse sur l’application, car ce trafic n’est pas géré par l’application.

Si votre application se compose principalement de pages statiques, envisagez d’utiliser [CDN pour mettre en cache la totalité de l’application][cdn-app-service]. Sinon, placez le contenu statique, tel que les images, les feuilles de style en cascade et les fichiers HTML, dans [Stockage Azure et utilisez CDN pour mettre en cache ces fichiers][cdn-storage-account].

> [!NOTE]
> Azure CDN ne peut pas servir du contenu qui nécessite une authentification.
> 
> 

Pour plus d’informations, consultez [Aide relative au réseau de distribution de contenu (CDN)][cdn-guidance].

### <a name="storage"></a>Stockage
Les applications modernes traitent souvent de grandes quantités de données. Afin de trouver une bonne adaptation pour le cloud, il est important de choisir le type de stockage adéquat. Voici quelques recommandations de base : 

| Éléments à stocker | Exemples | Stockage recommandé |
| --- | --- | --- |
| Fichiers |Images, documents, fichiers PDF |un stockage Azure Blob |
| Paires clé/valeur |Données de profil utilisateur recherchées par ID d’utilisateur |Stockage de tables Azure |
| Messages courts destinés à déclencher un traitement supplémentaire |Demandes de commande |Stockage File d’attente Azure, file d’attente Service Bus ou rubrique Service Bus |
| Données non relationnelles avec un schéma flexible nécessitant une interrogation de base |Catalogue produits |Base de données de document, comme Azure Cosmos DB, MongoDB ou Apache CouchDB |
| Données relationnelles nécessitant une prise en charge des requêtes enrichie, un schéma strict et/ou une cohérence forte |Inventaire de produits |Azure SQL Database |

## <a name="scalability-considerations"></a>Considérations relatives à l’extensibilité

Le principal avantage de Azure App Service est la possibilité de mettre à l’échelle votre application en fonction de la charge. Voici quelques considérations à prendre en compte pendant la planification de la mise à l’échelle de votre application.

### <a name="app-service-app"></a>application App Service
Si votre solution inclut plusieurs applications App Service, envisagez de les déployer sur des plans App Service distincts. Cette approche vous permet de les mettre à l’échelle indépendamment, car elles s’exécutent sur des instances distinctes. 

De même, envisagez de placer une tâche web dans son propre plan afin que les tâches en arrière-plan ne s’exécutent pas sur les mêmes instances qui gèrent les requêtes HTTP.  

### <a name="sql-database"></a>Base de données SQL
Augmentez la scalabilité d’une base de données SQL en *partitionnant* celle-ci. Le partitionnement fait référence au partitionnement horizontal de la base de données. Le partitionnement vous permet de faire évoluer horizontalement la base de données à l’aide [d’outils de base de données élastique][sql-elastic]. Les avantages du partitionnement peuvent être les suivants :

- Meilleur débit de transactions.
- Les requêtes peuvent s’exécuter plus rapidement sur un sous-ensemble des données.

### <a name="azure-search"></a>Recherche Azure
Recherche Azure supprime la surcharge liées aux recherches de données complexes à partir du magasin de données principal, et peut évoluer pour gérer la charge. Consultez [Mettre à l’échelle les niveaux de ressources pour interroger et indexer les charges de travail dans Recherche Azure][azure-search-scaling].

## <a name="security-considerations"></a>Considérations relatives à la sécurité
Cette section répertorie les considérations relatives à la sécurité qui sont spécifiques aux services Azure décrits dans cet article. Il ne s’agit pas d’une liste complète de bonnes pratiques relatives à la sécurité. Pour certaines considérations supplémentaires relatives à la sécurité, consultez [Sécuriser une application dans Azure App Service][app-service-security].

### <a name="cross-origin-resource-sharing-cors"></a>Partage des ressources cross-origin (CORS)
Si vous créez un site web et une API en tant qu’applications distinctes, le site web ne peut pas effectuer d’appels AJAX côté client à l’API, sauf si vous activez CORS.

> [!NOTE]
> La sécurité des navigateurs empêche une page web d’adresser des demandes AJAX à un autre domaine. Cette restriction est appelée stratégie de même origine et empêche un site malveillant de lire des données sensibles à partir d’un autre site. CORS est une norme W3C qui permet à un serveur d’assouplir la stratégie de même origine et d’autoriser certaines requêtes cross-origin tout en en rejetant d’autres.
> 
> 

App Services prend en charge CORS, sans que vous ayez besoin d’écrire un code d’application. Consultez [Consommer une application API à partir de JavaScript à l’aide de CORS][cors]. Ajoutez le site web à la liste des origines autorisées pour l’API.

### <a name="sql-database-encryption"></a>Chiffrement de la base de données SQL
Utilisez [Transparent Data Encryption][sql-encryption] si vous avez besoin de chiffrer les données au repos dans la base de données. Cette fonctionnalité effectue le chiffrement et le déchiffrement en temps réel d’une base de données entière (y compris les sauvegardes et les fichiers journaux des transactions) et ne nécessite aucune modification de l’application. Le chiffrement étant source de latence, il est judicieux de placer les données à sécuriser dans leur propre base de données et d’activer le chiffrement uniquement pour cette base de données.  
  

<!-- links -->

[api-guidance]: ../../best-practices/api-design.md
[app-service-security]: /azure/app-service-web/web-sites-security
[app-service-web-app]: /azure/app-service-web/app-service-web-overview
[app-service-api-app]: /azure/app-service-api/app-service-api-apps-why-best-platform
[app-service-pricing]: https://azure.microsoft.com/pricing/details/app-service/
[azure-cdn]: https://azure.microsoft.com/services/cdn/
[azure-dns]: /azure/dns/dns-overview
[azure-redis]: https://azure.microsoft.com/services/cache/
[azure-search]: https://azure.microsoft.com/documentation/services/search/
[azure-search-scaling]: /azure/search/search-capacity-planning
[background-jobs]: ../../best-practices/background-jobs.md
[basic-web-app]: basic-web-app.md
[basic-web-app-scalability]: basic-web-app.md#scalability-considerations
[caching-guidance]: ../../best-practices/caching.md
[cdn-app-service]: /azure/app-service-web/cdn-websites-with-cdn
[cdn-storage-account]: /azure/cdn/cdn-create-a-storage-account-with-cdn
[cdn-guidance]: ../../best-practices/cdn.md
[cors]: /azure/app-service-api/app-service-api-cors-consume-javascript
[cosmosdb]: /azure/cosmos-db/
[queue-storage]: /azure/storage/storage-dotnet-how-to-use-queues
[queues-compared]: /azure/service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted
[resource-group]: /azure/azure-resource-manager/resource-group-overview#resource-groups
[sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[sql-elastic]: /azure/sql-database/sql-database-elastic-scale-introduction
[sql-encryption]: https://msdn.microsoft.com/library/dn948096.aspx
[tm]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.blob.core.windows.net/cdn/app-service-reference-architectures.vsdx
[web-app-multi-region]: ./multi-region.md
[webjobs-guidance]: ../../best-practices/background-jobs.md
[webjobs]: /azure/app-service/app-service-webjobs-readme
[0]: ./images/scalable-web-app.png "Application web dans Azure avec une scalabilité améliorée"
