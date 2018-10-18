---
title: Serveur frontal e-commerce sur Azure
description: Hébergez un site d’e-commerce sur Azure.
author: masonch
ms.date: 7/13/18
ms.openlocfilehash: 6ca85665a5bf63bf71f5badc16406db5df2a34c2
ms.sourcegitcommit: b2a4eb132857afa70201e28d662f18458865a48e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2018
ms.locfileid: "48819089"
---
# <a name="an-e-commerce-front-end-on-azure"></a>Serveur frontal e-commerce sur Azure

Cet exemple de scénario vous guide tout au long d’une implémentation d’un serveur frontal e-commerce à l’aide des outils PaaS Azure. De nombreux sites Web de e-commerce sont confrontés à des variations de la saisonnalité et du trafic au fil du temps. Quand la demande en produits et services augmente, de façon prévisible ou non, l’utilisation des outils PaaS vous permet de gérer davantage de clients et de transactions automatiquement. De plus, ce scénario tire parti de l’approche économique du cloud en payant uniquement la capacité que vous utilisez.

Ce document vous aide à découvrir les différents composants PaaS Azure et les considérations utilisées pour mettre ensemble et pour déployer un exemple d’application de e-commerce, *Relecloud Concerts*, une plateforme de création de tickets de concert en ligne.

## <a name="relevant-use-cases"></a>Cas d’usage appropriés

Pensez à ce scénario pour les cas d’usage suivants :

* La création d’une application nécessitant une mise à l’échelle élastique pour gérer les pics d’utilisateurs à des moments différents.
* La création d’une application conçue pour fonctionner avec une haute disponibilité dans différentes régions Azure du monde entier.

## <a name="architecture"></a>Architecture

![Exemple d’architecture de scénario pour une application de e-commerce][architecture]

Ce scénario couvre l’achat de tickets à partir d’un site de e-commerce, les données transitent par le scénario comme suit :

1. Azure Traffic Manager achemine la demande d’un utilisateur vers le site de e-commerce hébergé dans Azure App Service.
2. Azure CDN sert des images statiques et le contenu à l’utilisateur.
3. L’utilisateur se connecte à l’application via un client Azure Active Directory B2C.
4. L’utilisateur recherche des concerts à l’aide d’Azure Search.
5. Le site web extrait les détails du concert à partir d’Azure SQL Database. 
6. Le site web fait référence aux images de tickets achetés dans le stockage Blob.
7. Les résultats de requête de la base de données sont mis en cache dans Cache Redis Azure pour offrir de meilleures performances.
8. L’utilisateur envoie des commandes de billets et des avis sur des concerts, qui sont placés dans la file d’attente.
9. Azure Functions traite le paiement de la commande et les revues de concert.
10. Cognitives services fournit une analyse de la revue du concert afin de déterminer le sentiment (positif ou négatif).
11. Application Insights fournit des métriques de performances pour surveiller l’intégrité de l’application web.

### <a name="components"></a>Composants

* [Azure CDN][docs-cdn] fournit le contenu statique, mis en cache à partir d’emplacements proches des utilisateurs pour réduire la latence.
* [Azure Traffic Manager][docs-traffic-manager] contrôle la distribution du trafic utilisateur pour les points de terminaison de service dans différentes régions Azure.
* [App Services - Web Apps][docs-webapps] héberge des applications web permettant une mise à l'échelle automatique et une haute disponibilité sans avoir à gérer l’infrastructure.
* [Azure Active Directory - B2C][docs-b2c] est un service de gestion des identités qui vous permet de personnaliser et de contrôler la façon dont les clients s’inscrivent, se connectent et gèrent leurs profils dans une application.
* [Files d’attente de stockage][docs-storage-queues] stocke un grand nombre de messages de file d’attente accessibles par une application.
* [Functions][docs-functions] constitue les options de calcul sans serveur qui permettent aux applications d’être exécutées à la demande sans avoir à gérer l’infrastructure.
* [Cognitive Services – Analyse des sentiments][docs-sentiment-analysis] utilise des API de Machine Learning et permet aux développeurs d’ajouter facilement des fonctionnalités intelligentes dans des applications, comme la reconnaissance des émotions et la détection vidéo, la reconnaissance faciale, vocale et visuelle ainsi que la compréhension de la parole et du langage.
* La [Recherche Azure][docs-search] est une solution cloud de recherche en tant que service, qui offre une expérience de recherche riche concernant du contenu privé et hétérogène dans les applications web, mobiles et d’entreprise.
* Le [stockage Blob][docs-storage-blobs] est optimisé pour stocker de grandes quantités de données non structurées, telles que des données texte ou binaires.
* [Cache Redis][docs-redis-cache] améliore les performances et l’extensibilité des systèmes qui s’appuient sur les magasins de données principaux en copiant temporairement des données fréquemment utilisées dans un stockage rapide situé près de l’application.
* [SQL Database][docs-sql-database] est un service administré de bases de données relationnelles à usage général de Microsoft Azure qui prend en charge des structures telles que les données relationnelles, JSON, les données spatiales et XML.
* [Application Insights][docs-application-insights] est conçu pour vous aider à améliorer en permanence les performances et la convivialité en détectant automatiquement les anomalies de performances via des outils d’analytique intégrés pour aider à comprendre ce que font les utilisateurs avec une application.

### <a name="alternatives"></a>Autres solutions

Bien d’autres technologies sont disponibles pour créer une application visible par le client consacrée au e-commerce à grande échelle. Elles couvrent le front-end de l’application, ainsi que la couche Données.

Les autres options pour la couche Web et les fonctions incluent :

* [Service Fabric][docs-service-fabric] : une plateforme concentrée autour de la création des composants distribués qui bénéficient d’un déploiement et d’une exécution sur un cluster avec un degré élevé de contrôle. Service Fabric peut également être utilisé pour héberger des conteneurs.
* [Azure Kubernetes Service][docs-kubernetes-service] : une plateforme pour la création et le déploiement de solutions à conteneurs utilisables comme une implémentation d’une architecture de microservices. Cela offre une agilité aux différents composants de l’application, pour leur permettre d’être mis à l’échelle indépendamment à la demande.
* [Azure Container Instances][docs-container-instances] : une façon rapide de déployer et d’exécuter des conteneurs avec un cycle de vie court. Les conteneurs sont alors déployés pour exécuter une tâche de traitement rapide, par exemple d’un message ou d’un calcul, puis ils sont déprovisionnés une fois l’opération terminée.
* [Service Bus][service bus] peut être utilisé à la place de la file d'attente de stockage.

D’autres options pour la couche Données incluent :

* [Cosmos DB](/azure/cosmos-db/introduction) : service de base de données multimodèle mondialement distribué de Microsoft. Cette plateforme permet d’exécuter d’autres modèles de données comme des données MongoDB, Cassandra ou Graph, ou un Stockage Table simple.

## <a name="considerations"></a>Considérations

### <a name="availability"></a>Disponibilité

* Envisagez l’utilisation des [modèles de conception standards pour la disponibilité][design-patterns-availability] lorsque vous générez votre application cloud.
* Passez en revue les considérations sur la disponibilité dans [l’architecture de référence d’application web de App Service][app-service-reference-architecture] appropriée
* Pour plus d’informations sur la disponibilité, voir la [liste de contrôle de la disponibilité][availability] dans le Centre des architectures Azure.

### <a name="scalability"></a>Extensibilité

* Lors de la création d’une application cloud, soyez conscient des [modèles de conception standards pour l’évolutivité][design-patterns-scalability].
* Passez en revue les considérations sur l’extensibilité dans [l’architecture de référence d’application web de App Service][app-service-reference-architecture] appropriée
* Pour plus d’informations sur l’extensibilité, voir la [liste de contrôle de l’extensibilité][scalability] dans le Centre des architectures Azure.

### <a name="security"></a>Sécurité

* Envisagez l’utilisation des [modèles de conception standards pour la sécurité][design-patterns-security] le cas échéant.
* Passez en revue les considérations sur la sécurité dans [l’architecture de référence d’application web de App Service][app-service-reference-architecture] appropriée.
* Prenez en compte le suivi d’un processus de [cycle de vie de développement sécurisé][secure-development] pour aider les développeurs à créer des logiciels plus sécurisés et gérer les exigences de conformité de sécurité, tout en réduisant les coûts de développement.
* Passez en revue le plan d’architecture pour [la conformité avec Azure PCI DSS][pci-dss-blueprint].

### <a name="resiliency"></a>Résilience

* Envisagez l’utilisation du [modèle disjoncteur][circuit-breaker] pour permettre la gestion des erreurs sans perte de données dans le cas où une partie de l’application ne serait pas disponible.
* Examinez les [modèles de conception standards pour la résilience][design-patterns-resiliency] et envisagez de les implémenter le cas échéant.
* Vous trouverez différentes [pratiques recommandées pour App Service][resiliency-app-service] dans le Centre des architectures Azure.
* Envisagez d’utiliser la [géo-réplication][sql-geo-replication] active pour la couche Données et le stockage [géoredondant][storage-geo-redudancy] des images et des files d’attente.
* Pour plus d’informations sur la [résilience][resiliency], voir l’article correspondant dans le Centre des architectures Azure.

## <a name="deploy-the-scenario"></a>Déployez le scénario

Pour déployer ce scénario, vous pouvez suivre ce [didacticiel pas à pas][end-to-end-walkthrough] montrant comment déployer manuellement chaque composant. Ce didacticiel fournit également un exemple d’application .NET qui exécute une simple application d’achat de tickets. Il existe un autre modèle Resource Manager permettant d’automatiser le déploiement de la plupart des ressources Azure.

## <a name="pricing"></a>Tarifs

Explorez le coût d’exécution de ce scénario, tous les services sont préconfigurés dans le calculateur de coûts. Pour pouvoir observer l’évolution de la tarification pour votre cas d’usage particulier, modifiez les variables appropriées en fonction du trafic que vous escomptez.

Nous proposons trois exemples de profils de coût basés sur la quantité de trafic que vous escomptez :

* [Petit][small-pricing] : les composants nécessaires pour générer la sortie pour une instance d’un niveau de production minimal. Nous supposons ici la présence d’un petit nombre d’utilisateurs, de l’ordre de quelques milliers par mois. L’application utilise une seule instance d’une application web standard, suffisante pour permettre une mise à l’échelle automatique. Tous les autres composants sont mis à l’échelle à un niveau de base qui, pour un coût minimal, assure le respect du contrat de niveau de service et une capacité suffisante pour gérer une charge de travail de production.
* [Moyen][medium-pricing] : les composants représentatifs d’un déploiement de taille moyenne. Ici, nous estimons environ 100 000 utilisateurs du système au cours d’un mois. Le trafic attendu est géré dans une instance de service d’application unique avec un niveau standard modéré. En outre, des niveaux modérés des services Cognitive et de recherche sont ajoutés à la calculatrice.
* [Grand][large-pricing] : une application destinée à un déploiement à grande échelle, de l’ordre de plusieurs millions d’utilisateurs par mois et de plusieurs téraoctets de données. À ce niveau de performances d’utilisation élevées, un niveau Premium est requis pour les applications web déployées dans plusieurs régions exposées par Traffic Manager. Les données incluent : le stockage, les bases de données et un réseau de distribution de contenu, qui sont configurés pour plusieurs téraoctets de données.

## <a name="related-resources"></a>Ressources associées

* [Architecture de référence pour une application web multirégion][multi-region-web-app]
* [eShop sur l’exemple de référence de conteneurs][microservices-ecommerce]

<!-- links -->
[architecture]: ./media/architecture-ecommerce-scenario.png
[small-pricing]: https://azure.com/e/90fbb6a661a04888a57322985f9b34ac
[medium-pricing]: https://azure.com/e/38d5d387e3234537b6859660db1c9973
[large-pricing]: https://azure.com/e/f07f99b6c3134803a14c9b43fcba3e2f
[app-service-reference-architecture]: ../../reference-architectures/app-service-web-app/basic-web-app.md
[availability]: /azure/architecture/checklist/availability
[circuit-breaker]: /azure/architecture/patterns/circuit-breaker
[design-patterns-availability]: /azure/architecture/patterns/category/availability
[design-patterns-resiliency]: /azure/architecture/patterns/category/resiliency
[design-patterns-scalability]: /azure/architecture/patterns/category/performance-scalability
[design-patterns-security]: /azure/architecture/patterns/category/security
[docs-application-insights]: /azure/application-insights/app-insights-overview
[docs-b2c]: /azure/active-directory-b2c/active-directory-b2c-overview
[docs-cdn]: /azure/cdn/cdn-overview
[docs-container-instances]: /azure/container-instances/
[docs-kubernetes-service]: /azure/aks/
[docs-functions]: /azure/azure-functions/functions-overview
[docs-redis-cache]: /azure/redis-cache/cache-overview
[docs-search]: /azure/search/search-what-is-azure-search
[docs-service-fabric]: /azure/service-fabric/
[docs-sentiment-analysis]: /azure/cognitive-services/welcome
[docs-sql-database]: /azure/sql-database/sql-database-technical-overview
[docs-storage-blobs]: /azure/storage/blobs/storage-blobs-introduction
[docs-storage-queues]: /azure/storage/queues/storage-queues-introduction
[docs-traffic-manager]: /azure/traffic-manager/traffic-manager-overview
[docs-webapps]: /azure/app-service/app-service-web-overview
[end-to-end-walkthrough]: https://github.com/Azure/fta-customerfacingapps/tree/master/ecommerce/articles
[microservices-ecommerce]: https://github.com/dotnet-architecture/eShopOnContainers
[multi-region-web-app]: /azure/architecture/reference-architectures/app-service-web-app/multi-region
[pci-dss-blueprint]: /azure/security/blueprints/payment-processing-blueprint
[resiliency-app-service]: /azure/architecture/checklist/resiliency-per-service#app-service
[resiliency]: /azure/architecture/checklist/resiliency
[scalability]: /azure/architecture/checklist/scalability
[secure-development]: https://www.microsoft.com/SDL/process/design.aspx
[sql-geo-replication]: /azure/sql-database/sql-database-geo-replication-overview
[storage-geo-redudancy]: /azure/storage/common/storage-redundancy-grs
