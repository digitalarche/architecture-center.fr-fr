---
title: Moteur de recherche de produit intelligent pour l’e-commerce
titleSuffix: Azure Example Scenarios
description: Offrez une expérience de recherche de haute qualité dans une application d’e-commerce.
author: jelledruyts
ms.date: 09/14/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: fasttrack
ms.openlocfilehash: fe67c891a9e42d5216fe6fd81de6ea1333d5bd37
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54488237"
---
# <a name="intelligent-product-search-engine-for-e-commerce"></a>Moteur de recherche de produit intelligent pour l’e-commerce

Cet exemple de scénario montre comment l’utilisation d’un service de recherche dédié peut considérablement augmenter la pertinence des résultats de la recherche pour vos clients de commerce électronique.

La recherche est le mécanisme principal grâce auquel les clients retrouvent et achètent des produits. Il est donc essentiel que les résultats de la recherche soient pertinents par rapport à l’_intention_ de la requête de recherche et que l’expérience de recherche de bout en bout corresponde aux géants de la recherche en proposant des résultats quasi instantanés, une analyse linguistique, une correspondance géographique, un filtrage, des facettes, une saisie semi-automatique, une mise en surbrillance des correspondances, etc.

Imaginez une application web de commerce électronique classique avec des données produit stockées dans une base de données relationnelle telle que SQL Server ou Azure SQL Database. Les requêtes de recherche sont généralement gérées dans la base de données à l’aide de requêtes `LIKE` ou de fonctionnalités [Recherche en texte intégral][docs-sql-fts]. Grâce à [Recherche Azure][docs-search], vous libérez la base de données opérationnelle du traitement des requêtes et vous pouvez facilement tirer profit de ces fonctionnalités difficiles à implémenter qui offrent à vos clients la meilleure expérience de recherche possible. En outre, Recherche Azure étant un composant de plateforme en tant que service (PaaS), vous n’avez pas à vous soucier de la gestion de l’infrastructure ni d’être un expert de la recherche.

## <a name="relevant-use-cases"></a>Cas d’usage appropriés

Les autres cas d’usage appropriés sont les suivants :

- Recherche d’annonces immobilières ou d’agences proches de l’emplacement physique de l’utilisateur.
- Recherche d’articles sur un site d’actualités ou de résultats sportifs, avec une préférence pour des informations plus _récentes_.
- Recherche dans de grands référentiels pour des organisations _axées sur les documents_ comme les décideurs politiques et les notaires.

Enfin, _toute_ application incluant une quelconque forme de fonctionnalité de recherche peut tirer parti d’un service de recherche dédié.

## <a name="architecture"></a>Architecture

![Vue d’ensemble de l’architecture des composants Azure impliqués dans un moteur de recherche de produit intelligent pour le commerce électronique][architecture]

Ce scénario couvre une solution de commerce électronique dans laquelle les clients peuvent rechercher via un catalogue de produits.

1. Les clients accèdent à l’**application web de commerce électronique** depuis n’importe quel appareil.
2. Le catalogue de produits se trouve dans une **Azure SQL Database** pour le traitement transactionnel.
3. Recherche Azure utilise un **indexeur de recherche** pour maintenir automatiquement son index de recherche à jour via le suivi des modifications intégré.
4. Les requêtes de recherche du client sont dirigées sur le service **Recherche Azure** qui traite la requête et retourne les résultats les plus pertinents.
5. Outre l’expérience de recherche basée sur le web, les clients peuvent également utiliser un **bot conversationnel** dans un réseau social ou directement à partir d’Assistants numériques pour rechercher des produits et affiner de manière incrémentielle leur requête de recherche et les résultats.
6. La fonctionnalité **Recherche cognitive** peut éventuellement être utilisée pour appliquer l’intelligence artificielle pour un traitement encore plus intelligent.

### <a name="components"></a>Composants

- [App Services - Web Apps][docs-webapps] héberge des applications web permettant une mise à l’échelle automatique et une haute disponibilité sans avoir à gérer l’infrastructure.
- [SQL Database][docs-sql-database] est un service administré de bases de données relationnelles à usage général de Microsoft Azure qui prend en charge des structures telles que les données relationnelles, JSON, les données spatiales et XML.
- La [Recherche Azure][docs-search] est une solution cloud de recherche en tant que service, qui offre une expérience de recherche riche concernant du contenu privé et hétérogène dans les applications web, mobiles et d’entreprise.
- [Bot Service][docs-botservice] fournit des outils pour créer, tester, déployer et gérer des robots intelligents.
- [Cognitive Services][docs-cognitive] permet d’utiliser des algorithmes intelligents pour voir, écouter, énoncer, comprendre et interpréter les besoins des utilisateurs selon des modes de communication naturels.

### <a name="alternatives"></a>Autres solutions

- Vous pouvez utiliser des fonctionnalités de **recherche dans la base de données**, par exemple, via la recherche en texte intégral de SQL Server, mais votre magasin transactionnel traite alors également des requêtes (augmentant ainsi la puissance de traitement), et les fonctionnalités de recherche dans la base de données sont plus limitées.
- Vous pouvez héberger [Apache Lucene][apache-lucene] open source (sur lequel Recherche Azure s’appuie) sur des machines virtuelles Azure, mais vous revenez alors à la gestion d’infrastructure en tant que service (IaaS) et ne bénéficiez pas des nombreuses fonctionnalités offertes par Recherche Azure sur Lucene.
- Vous pouvez également envisager de déployer [ElasticSearch][elastic-marketplace] à partir de la Place de marché Azure, qui est un produit de recherche alternatif d’un fournisseur tiers, mais vous rencontrez dans ce cas une charge de travail IaaS.

D’autres options pour la couche Données incluent :

- [Cosmos DB](/azure/cosmos-db/introduction) : un service de base de données multimodèle mondialement distribué de Microsoft. Cosmos DB fournit une plateforme pour exécuter d’autres modèles de données tels que Mongo DB, Cassandra, des données graphiques ou un stockage de tables simple. Recherche Azure prend également en charge l’indexation des données directement à partir de Cosmos DB.

## <a name="considerations"></a>Considérations

### <a name="scalability"></a>Extensibilité

Le [niveau tarifaire][search-tier] du service Recherche Azure ne détermine pas les fonctionnalités disponibles, mais est principalement utilisé pour la [planification de la capacité][search-capacity] car il définit le stockage maximum que vous obtenez et le nombre de partitions et de réplicas que vous pouvez configurer. Les **partitions** vous permettent d’indexer plus de documents et d’obtenir des débits d’écriture plus importants, tandis que les **réplicas** fournissent davantage de requêtes par seconde (RPS) et la haute disponibilité.

Vous pouvez modifier dynamiquement le nombre de partitions et de réplicas, mais vous ne pouvez pas modifier le niveau tarifaire. Étudiez donc rigoureusement le niveau approprié à votre charge de travail cible. Si vous devez changer de niveau, vous devrez configurer un nouveau service en parallèle et recharger vos index. Vous dirigerez ensuite vos applications vers le nouveau service.

### <a name="availability"></a>Disponibilité

Recherche Azure fournit un [SLA de disponibilité de 99,9 %][search-sla] pour les _lectures_ (c’est-à-dire les requêtes) si vous disposez d’au moins deux réplicas, et pour les _mises à jour_ (c’est-à-dire la mise à jour de l’index de recherche) si vous disposez d’au moins trois réplicas. Vous devez donc configurer deux réplicas minimum si vous souhaitez que vos clients puissent effectuer une _recherche_ fiable, et en configurer 3 si des _modifications réelles apportées à l’index_ doivent également être prises en compte dans les opérations de haute disponibilité.

Si des changements cassants doivent être apportés à l’index sans impliquer un temps d’arrêt (par exemple, modifier des types de données, supprimer ou renommer des champs), l’index devra être reconstruit. Comme pour le changement du niveau de service, cela implique de créer un nouvel index, de le renseigner à nouveau avec des données, puis de mettre à jour vos applications pour qu’elles pointent vers le nouvel index.

### <a name="security"></a>Sécurité

Recherche Azure est compatible avec de nombreuses [normes de sécurité et de confidentialité des données][search-security], et peut donc être utilisé dans la plupart des secteurs d’activité.

Pour sécuriser l’accès au service, Recherche Azure utilise deux types de clés : des **clés d’administration** qui vous permettent d’effectuer _n’importe quelle_ tâche sur le service, et des **clés de requête** qui ne peuvent être utilisées que dans des opérations en lecture seule comme l’interrogation. En règle générale, l’application qui effectue la recherche ne met pas à jour l’index. Il ne doit donc être configuré qu’avec une clé de requête et non une clé d’administration (et plus particulièrement si la recherche est effectuée à partir d’un appareil utilisateur final comme un script exécuté dans un navigateur web).

### <a name="search-relevance"></a>Pertinence de la recherche

L’efficacité votre application de commerce électronique dépend en grande partie de la pertinence des résultats de la recherche pour vos clients. Un paramétrage rigoureux de votre service de recherche pour fournir des résultats optimaux en fonction de la recherche de l’utilisateur, ou s’appuyer sur des fonctionnalités intégrées telles que l’[analyse du trafic de recherche][search-analysis] afin de comprendre les modèles de recherche de votre client, vous permettent de prendre des décisions basées sur les données.

Les méthodes classiques de paramétrage de votre service de recherche incluent les suivantes :

- Utilisation de [profils de scoring][search-scoring] pour influencer la pertinence des résultats de la recherche, par exemple, en fonction du champ correspondant à la requête, de l’ancienneté des données, de la distance géographique par rapport à l’utilisateur, ...
- Utilisation d’[analyseurs de langage fournis par Microsoft][search-languages] qui utilisent une pile de traitement en langage naturel (NLP) avancée pour mieux interpréter les requêtes
- Utilisation d’[analyseurs personnalisés][search-analyzers] pour garantir que vos produits sont correctement trouvés, en particulier si vous souhaitez rechercher sur des informations non basées sur un langage comme la marque et le modèle d’un produit.

## <a name="deploy-the-scenario"></a>Déployez le scénario

Pour déployer une version de commerce électronique plus complète de ce scénario, vous pouvez suivre ce [didacticiel pas à pas][end-to-end-walkthrough] qui fournit un exemple d’application .NET qui exécute une application d’achat de tickets simple. Il inclut également Recherche Azure et utilise un grand nombre des fonctionnalités abordées. Il existe un autre modèle Resource Manager permettant d’automatiser le déploiement de la plupart des ressources Azure.

## <a name="pricing"></a>Tarifs

Pour explorer le coût d’exécution de ce scénario, tous les services mentionnés ci-dessus sont préconfigurés dans le calculateur de coûts. Pour pouvoir observer l’évolution de la tarification pour votre cas d’usage particulier, modifiez les variables appropriées en fonction de l’utilisation que vous escomptez.

Nous proposons trois exemples de profils de coût basés sur la quantité de trafic que vous escomptez :

- [Petit][small-pricing] : dans ce profil, nous utilisons une seule application web `Standard S1` pour héberger le site web, le niveau gratuit du service Azure Bot, un seul service Recherche Azure `Basic` et une base de données SQL Database `Standard S2`.
- [Moyen][medium-pricing] : ici, nous effectuons un scale-up de l’application web vers deux instances du niveau `Standard S3`, nous mettons à niveau Search Service vers le niveau `Standard S1` et nous utilisons une base de données SQL Database `Standard S6`.
- [Grand][large-pricing] : dans le profil le plus grand, nous utilisons quatre instances d’une application web `Premium P2V2`, nous mettons à niveau le service Azure Bot vers le niveau `Standard S1` (avec 1 000 000 messages dans les canaux Premium), nous utilisons deux unités du service Recherche Azure `Standard S3` ainsi qu’une base de données SQL Database `Premium P6`.

## <a name="related-resources"></a>Ressources associées

Pour en savoir plus sur Recherche Azure, consultez le [centre de documentation][docs-search], consultez les [exemples][search-samples], ou un [site de démonstration][search-demo] complet en action.

<!-- links -->
[architecture]: ./media/architecture-ecommerce-search.png
[docs-sql-fts]: /sql/relational-databases/search/query-with-full-text-search
[docs-search]: /azure/search/search-what-is-azure-search
[docs-sql-database]: /azure/sql-database/sql-database-technical-overview
[docs-webapps]: /azure/app-service/app-service-web-overview
[docs-botservice]: /azure/bot-service/
[docs-cognitive]: /azure/cognitive-services/
[apache-lucene]: https://lucene.apache.org/
[elastic-marketplace]: https://azuremarketplace.microsoft.com/marketplace/apps/elastic.elasticsearch
[end-to-end-walkthrough]: https://github.com/Azure/fta-customerfacingapps/tree/master/ecommerce/articles
[search-sla]: https://go.microsoft.com/fwlink/?LinkId=716855
[search-tier]: /azure/search/search-sku-tier
[search-capacity]: /azure/search/search-capacity-planning
[search-security]: /azure/search/search-security-overview
[search-analysis]: /azure/search/search-traffic-analytics
[search-languages]: /rest/api/searchservice/language-support
[search-analyzers]: /rest/api/searchservice/custom-analyzers-in-azure-search
[search-scoring]: /rest/api/searchservice/add-scoring-profiles-to-a-search-index
[search-samples]: https://azure.microsoft.com/resources/samples/?service=search&sort=0
[search-demo]: https://azjobsdemo.azurewebsites.net/
[small-pricing]: https://azure.com/e/db2672a55b6b4d768ef0060a8d9759bd
[medium-pricing]: https://azure.com/e/a5ad0706c9e74add811e83ef83766a1c
[large-pricing]: https://azure.com/e/57f95a898daa487795bd305599973ee6
