---
title: Traitement évolutif des commandes
titleSuffix: Azure Example Scenarios
description: Créez un pipeline de traitement de commande hautement évolutif à l’aide d’Azure Cosmos DB.
author: alexbuckgit
ms.date: 07/10/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.openlocfilehash: ff71697969ba9fd85ff49c38458e59fc3447f905
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54481029"
---
# <a name="scalable-order-processing-on-azure"></a>Traitement évolutif des commandes sur Azure

Cet exemple de scénario s’applique aux entreprises ayant besoin d’une architecture hautement évolutive et résiliente pour le traitement de commandes en ligne. Les applications potentielles incluent le e-commerce et les points de vente au détail, le traitement des commandes ainsi que la réservation et le suivi d’un stock.

Ce scénario prend une approche d’approvisionnement en événements, à l’aide d’un modèle de programmation fonctionnel implémenté par le biais de microservices. Chaque microservice est traité comme un processeur de flux de données, et toute la logique métier est implémentée au moyen de microservices. Cette approche permet une disponibilité et une résilience élevées, une géo-réplication et des performances rapides.

L’utilisation de services gérés par Azure tels que Cosmos DB et HDInsight peut aider à réduire les coûts en tirant parti de l’expertise de Microsoft dans le stockage et la récupération de données mondialement distribuées à l’échelle du cloud. Ce scénario concerne plus spécialement un scénario de e-commerce ou de vente au détail. Si vous avez d’autres besoins en matière de services de données, vous devez examiner la liste des [services de base de données intelligente entièrement gérée dans Azure][product-category] qui sont disponibles.

## <a name="relevant-use-cases"></a>Cas d’usage appropriés

Les autres cas d’usage appropriés sont les suivants :

- Systèmes principaux de e-commerce ou de point de vente au détail.
- Systèmes de gestion des stocks.
- Systèmes de traitement de commande.
- Autres scénarios d’intégration concernant un pipeline de traitement de commande.

## <a name="architecture"></a>Architecture

![Exemple d’architecture pour un pipeline de traitement évolutif des commandes][architecture]

Cette architecture décrit en détail les composants clés d’un pipeline de traitement des commandes. Les données circulent dans le scénario comme suit :

1. Les messages d’événements entrent dans le système via des applications orientées client (de façon synchrone sur HTTP) et divers systèmes back-end (de façon asynchrone via Apache Kafka). Ces messages sont passés dans un pipeline de traitement des commandes.
2. Chaque message d’événement est ingéré et mappé à un ensemble défini de commandes par un microservice de traitement des commandes. Le traitement des commandes récupère n’importe quel état actuel pertinent pour l’exécution de la commande à partir d’une base de données de capture instantanée de flux d’événements. La commande est ensuite exécutée, et la sortie de la commande est émise comme un nouvel événement.
3. Chaque événement émis en tant que sortie d’une commande est affecté à une base de données des flux d’événement à l’aide de Cosmos DB.
4. Pour chaque insertion de base de données ou mise à jour affectée dans la base de données de flux d’événements, un événement est déclenché par le flux de modification de Cosmos DB. Les systèmes en aval peuvent s’abonner aux rubriques d’événement qui s’appliquent à ce système.
5. Tous les événements à partir du flux de modification de Cosmos DB sont également envoyés à un microservice de flux d’événements de capture instantanée, qui calcule les modifications d’état provoquées par des événements qui se sont produits. Le nouvel état est ensuite validé dans la base de données de capture instantanée de flux événement stockée dans Cosmos DB. La base de données de capture instantanée fournit une source de données mondialement distribuée et à faible latence pour l’état actuel de tous les éléments de données. La base de données de flux d’événements fournit un enregistrement complet de tous les messages d’événement passés à travers l’architecture, ce qui permet des scénarios robustes de récupération d’urgence, de dépannage et de test.

### <a name="components"></a>Composants

- [Cosmos DB](/azure/cosmos-db/introduction) est une base de données Microsoft multimodèle et distribuée mondialement qui vous permet de faire évoluer à votre guise et de manière indépendante le débit et le stockage de vos solutions dans le nombre de régions géographiques de votre choix. Il offre des garanties en termes de débit, de latence, de disponibilité et de cohérence avec des contrats SLA complets. Ce scénario utilise Cosmos DB pour le stockage de flux d’événements et le stockage de capture instantanée et tire parti des fonctionnalités du [flux de modification de Cosmos DB][docs-cosmos-db-change-feed] pour fournir une cohérence des données et une récupération après incident.
- [Apache Kafka sur HDInsight](/azure/hdinsight/kafka/apache-kafka-introduction) est une implémentation de service géré d’Apache Kafka, une plateforme de diffusion en continu distribuée en open source pour la création d’applications et de pipelines de diffusion de données en continu et en temps réel. Kafka fournit également des fonctionnalités de courtier de messages semblables à une file d’attente, pour la publication et l’abonnement aux flux de données nommés. Ce scénario utilise Kafka pour traiter les événements entrants et en aval dans le pipeline de traitement des commandes.

## <a name="considerations"></a>Considérations

De nombreuses options technologiques sont disponibles pour l’ingestion des messages en temps réel, le stockage de données, le traitement de flux, le stockage des données d’analyse et l’analyse et la création de rapports. Pour une vue d’ensemble de ces options, de leurs fonctionnalités et des principaux critères de sélection, consultez [Architectures de Big Data : Traitement en temps réel](/azure/architecture/data-guide/technology-choices/real-time-ingestion) dans le [Guide d’architecture des données Azure](/azure/architecture/data-guide).

Les microservices sont devenus un style architectural populaire pour générer des applications cloud résilientes, hautement évolutives, capables d’évoluer rapidement et pouvant être déployées indépendamment. Les microservices requièrent une approche différente pour concevoir et générer des applications. Des outils tels que Docker, Kubernetes, Azure Service Fabric et Nomad facilitent le développement d’architectures basées sur des microservices. Pour obtenir des conseils sur la création et l’exécution d’une architecture basée sur des microservices, consultez [Conception des microservices sur Azure](/azure/architecture/microservices) dans le Centre des architectures Azure.

### <a name="availability"></a>Disponibilité

L’approche d’approvisionnement en événements de ce scénario permet aux composants de système faiblement couplés et déployés indépendamment les uns des autres. Cosmos DB offre une [haute disponibilité][docs-cosmos-db-regional-failover] et permet à l’organisation de gérer les compromis associés à la cohérence, la disponibilité et les performances, avec [les garanties correspondantes][docs-cosmos-db-guarantees]. Apache Kafka sur HDInsight est également conçu pour une [haute disponibilité][docs-kafka-high-availability].

Azure Monitor fournit des interfaces utilisateur unifiées pour la surveillance entre divers services Azure. Pour plus d’informations, consultez l’article [Monitoring in Microsoft Azure](/azure/monitoring-and-diagnostics/monitoring-overview) (Surveillance dans Microsoft Azure). Event Hubs et Stream Analytics sont intégrés à Azure Monitor.

Pour d’autres considérations relatives à la disponibilité, consultez la [check-list de disponibilité][availability].

### <a name="scalability"></a>Extensibilité

Kafka sur HDInsight permet la [configuration du stockage et de l’extensibilité](/azure/hdinsight/kafka/apache-kafka-scalability) pour des clusters Kafka. Cosmos DB offre des performances rapides et prévisibles et [se met à l’échelle de façon transparente](/azure/cosmos-db/partition-data) à mesure que votre application se développe.
L’architecture basée sur les microservices d’approvisionnement en événements de ce scénario simplifie la mise à l'échelle de votre système et l’extension de ses fonctionnalités.

Pour voir d’autres considérations relatives à l’extensibilité, consultez la [liste de contrôle de l’extensibilité][scalability] disponible dans le Centre des architectures Azure.

### <a name="security"></a>Sécurité

Le [modèle de sécurité de Cosmos DB](/azure/cosmos-db/secure-access-to-data) authentifie les utilisateurs et fournit l’accès à ses données et les ressources. Pour plus d’informations, consultez la [sécurité de base de données Cosmos DB](/azure/cosmos-db/database-security).

Pour obtenir des conseils d’ordre général sur la conception de solutions sécurisées, consultez la [documentation sur la sécurité Azure][security].

### <a name="resiliency"></a>Résilience

L’architecture d’approvisionnement en événements et les technologies associées dans cet exemple de scénario rendent ce scénario hautement résilient lorsque des échecs se produisent. Pour obtenir des conseils d’ordre général sur la conception de solutions résilientes, consultez l’article [Conception d’applications résilientes pour Azure][resiliency].

## <a name="pricing"></a>Tarifs

Pour examiner le coût d’exécution de ce scénario, tous les services sont préconfigurés dans le calculateur de coûts. Pour pouvoir observer l’évolution de la tarification pour votre scénario particulier, modifiez les variables appropriées en fonction du volume de données que vous escomptez. Pour ce scénario, la tarification de l’exemple inclut uniquement Cosmos DB et un cluster Kafka pour le traitement des événements déclenchés à partir du flux de modification Cosmos DB. Les processeurs et les microservices d’événement pour des systèmes origineant et d’autres systèmes en aval ne sont pas inclus, et leur coût dépend beaucoup de la quantité et de la mise à l’échelle de ces services, de même que les technologies choisies pour les mettre en œuvre.

La devise d’Azure Cosmos DB est l’unité de requête (RU). Avec les unités de requête, vous n’avez pas besoin de réserver de capacités en lecture et en écriture, ni de configurer les ressources de processeur, de mémoire et d’E/S par seconde. Azure Cosmos DB prend en charge des API variées avec différentes opérations allant des lectures et des écritures simples aux requêtes de graphe complexes. Toutes les requêtes n’étant pas égales, la quantité normalisée d’unités de requête qui leur est affectée est fonction de la quantité de calcul requise pour traiter chaque requête. Le nombre d’unités de requête requis par votre solution dépend de la taille d’élément de données et du nombre d’opérations de lecture et d’écriture de la base de données, par seconde. Pour en savoir plus, voir [Unités de requête dans Azure Cosmos DB](/azure/cosmos-db/request-units). Ces prix estimés sont basés sur Cosmos DB en cours d’exécution dans deux régions Azure.

Nous proposons trois exemples de profils de coût basés sur la quantité d’activité que vous escomptez :

- [Petite][small-pricing] : 5 unités de requête réservées avec un magasin de données de 1 To dans Cosmos DB et un petit cluster Kafka (D3 v2).
- [Moyenne][medium-pricing] : 50 unités de requête réservées avec un magasin de données de 10 To dans Cosmos DB et un cluster Kafka de taille moyenne (D4 v2).
- [Grande][large-pricing] : 500 unités de requête réservées avec un magasin de données de 30 To dans Cosmos DB et un gros cluster Kafka (D5 v2).

## <a name="related-resources"></a>Ressources associées

Cet exemple de scénario est basé sur une version plus étendue de cette architecture générée par [Jet.com](https://jet.com) pour son pipeline de traitement de commandes de bout en bout. Pour plus d’informations, consultez le [profil de client technique jet.com][source-document] et la [présentation de jet.com au Build 2018][source-presentation].

Les autres ressources liées incluent :

- *[Designing Data-Intensive Applications](https://dataintensive.net)* par Martin Kleppmann (O'Reilly Media, 2017).
- *[Domain Modeling Made Functional: Tackle Software Complexity with Domain-Driven Design and F#](https://pragprog.com/book/swdddf/domain-modeling-made-functional)* par Scott Wlaschin (Pragmatic Programmers LLC, 2018).
- Autres [cas d’utilisation de Cosmos DB][docs-cosmos-db-use-cases]
- [Traitement en temps réel](/azure/architecture/data-guide/big-data/real-time-processing) dans le [Guide d’architecture des données Azure](/azure/architecture/data-guide).

<!-- links -->

[architecture]: ./media/architecture-ecommerce-order-processing.png
[product-category]: https://azure.microsoft.com/product-categories/databases/
[source-document]: https://customers.microsoft.com/story/jet-com-powers-innovative-e-commerce-engine-on-azure-in-less-than-12-months
[source-presentation]: https://channel9.msdn.com/events/Build/2018/BRK3602
[small-pricing]: https://azure.com/e/3d43949ffbb945a88cc0a126dc3a0e6e
[medium-pricing]: https://azure.com/e/1f1e7bf2a6ad4f7799581211f4369b9b
[large-pricing]: https://azure.com/e/75207172ece94cf6b5fb354a2252b333
[docs-cosmos-db-change-feed]: /azure/cosmos-db/change-feed
[docs-cosmos-db-regional-failover]: /azure/cosmos-db/regional-failover
[docs-cosmos-db-guarantees]: /azure/cosmos-db/distribute-data-globally#AvailabilityGuarantees
[docs-cosmos-db-use-cases]: /azure/cosmos-db/use-cases
[docs-kafka-high-availability]: /azure/hdinsight/kafka/apache-kafka-high-availability
[docs-event-hubs]: /azure/event-hubs/event-hubs-what-is-event-hubs
[docs-stream-analytics]: /azure/stream-analytics/stream-analytics-introduction
[availability]: /azure/architecture/checklist/availability
[scalability]: /azure/architecture/checklist/scalability
[resiliency]: /azure/architecture/patterns/category/resiliency/
[security]: /azure/security/
