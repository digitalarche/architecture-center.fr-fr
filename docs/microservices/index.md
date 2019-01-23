---
title: Conception, génération et exploitation de microservices sur Azure avec Kubernetes
description: Conception, génération et exploitation de microservices sur Azure.
author: MikeWasson
ms.date: 10/23/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: ffd42679d0e04c2283cd78b8b03d7b5c695578be
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54481948"
---
# <a name="designing-building-and-operating-microservices-on-azure"></a>Conception, génération et exploitation de microservices sur Azure

![Diagramme d’un service de livraison par drone](./images/drone.svg)

Les microservices sont devenus un style architectural populaire pour générer des applications cloud résilientes, hautement évolutives, capables d’évoluer rapidement et pouvant être déployées indépendamment. Toutefois, au-delà du concept à la mode, les microservices requièrent une approche différente pour concevoir et générer des applications.

Dans cet ensemble d’articles, nous explorons comment générer et exécuter une architecture de microservices sur Azure. Les sujets abordés sont les suivants :

- Utilisation d’une conception pilotée par le domaine pour concevoir une architecture de microservices.
- Choix des technologies Azure adaptées pour le calcul, le stockage, la messagerie et autres éléments de la conception.
- Présentation des microservices à l’aide de modèles.
- Conception orientée performances, extensibilité et résilience.
- Création d’un pipeline d'intégration continue/de déploiement continu.

Tout au long de cet article, nous nous concentrons sur un scénario de bout en bout : Un service de livraison par drone qui permet aux clients de planifier l’enlèvement et la livraison de colis via un drone. Vous trouverez le code de notre implémentation de référence sur GitHub.

[![GitHub](../_images/github.png) - Implémentation de référence][drone-ri]

Mais tout d’abord, commençons par les notions de base. Que sont les microservices, et quels sont les avantages d’une architecture de microservices ?

<!-- markdownlint-disable MD026 -->

## <a name="why-build-microservices"></a>Pourquoi générer des microservices ?

<!-- markdownlint-enable MD026 -->

Dans une architecture de microservices, l’application est composée de services indépendants et de petite taille. Voici certaines caractéristiques qui définissent un microservice :

- Chaque microservice implémente une fonctionnalité métier unique.
- Un microservice est suffisamment petit pour qu’une équipe réduite de développeurs puisse l’écrire et en assurer la maintenance.
- Les microservices s’exécutent dans des processus distincts, en communiquant via des API bien définies ou des modèles de messagerie.
- Les microservices ne partagent pas de magasins de données ou de schémas de données. Chaque microservice est chargé de gérer ses propres données.
- Les microservices ont des bases de code distinctes et ne partagent pas de code source. Ils peuvent néanmoins utiliser des bibliothèques d’utilitaires communes.
- Chaque microservice peut être déployé et mis à jour indépendamment des autres services.

Implémentés correctement, les microservices présentent de nombreux avantages :

- **Agilité.** Les microservices sont déployés de manière indépendante, ce qui facilite la gestion des correctifs de bogues et des nouvelles fonctionnalités. Vous pouvez mettre à jour un service sans avoir à redéployer l’application entière et effectuer une restauration si une mise à jour est source de problèmes. Dans de nombreuses applications traditionnelles, s’il existe un bogue dans une partie de l’application, il peut bloquer tout le processus de mise en production. Par conséquent, les nouvelles fonctionnalités peuvent être mises en attente pendant que la résolution du bogue est intégrée, testée et publiée.

- **Code réduit, petites équipes.** Un microservice doit être suffisamment petit pour qu’une seule équipe de fonctionnalité puisse le générer, le tester et le déployer. Les petites bases de code sont plus faciles à comprendre. Dans une grande application monolithique, les dépendances de code ont tendance à s’entremêler au fil du temps, auquel cas l’ajout d’une nouvelle fonctionnalité nécessite de modifier le code à beaucoup d’endroits. En ne partageant ni code, ni magasins de données, une architecture de microservices minimise les dépendances, ce qui facilite l’ajout de nouvelles fonctionnalités. Les petites équipes permettent également une plus grande souplesse. Selon la règle des « deux pizzas », une équipe doit être suffisamment réduite pour que deux pizzas suffisent à rassasier tous ses membres. Évidemment, ce n’est pas une mesure exacte et tout dépend de l’appétit de l’équipe ! Mais il faut en retenir que les grands groupes ont tendance à être moins productifs, car la communication est plus lente, la charge de gestion augmente et l’agilité diminue.

- **Pot-pourri de technologies**. Les équipes peuvent choisir la technologie qui convient le mieux à leur service à l’aide de diverses piles technologiques.

- **Résilience**. Si un microservice individuel devient indisponible, il n’interrompt pas l’application toute entière, tant qu’un microservice en amont est conçus pour gérer correctement les erreurs (par exemple, en implémentant une coupure du circuit).

- **Extensibilité**. Une architecture de microservices permet de mettre à l’échelle chaque microservice indépendamment des autres. Cela permet de faire monter en charge les sous-systèmes qui nécessitent plus de ressources, sans appliquer ce changement à toute l’application. Si vous déployez des services à l’intérieur de conteneurs, vous pouvez également packager une densité plus élevée de microservices sur un seul hôte, ce qui permet une utilisation plus efficace des ressources.

- **Isolation des données**. Il est beaucoup plus facile d’effectuer des mises à jour de schéma, car un seul microservice est concerné. Dans une application monolithique, les mises à jour de schéma peuvent devenir très difficiles, car les différentes parties de l’application impliquer les mêmes données, d’où le risque de modifier le schéma.

## <a name="no-free-lunch"></a>L’envers de la médaille

Ces avantages ont néanmoins un prix. Cette série d’articles est conçue pour résoudre certains problèmes rencontrés lors de la génération de microservices résilients, évolutifs et facile à gérer.

- **Limites de service** : lorsque vous générez des microservices, vous devez examiner soigneusement où placer les limites entre les services. Une fois que les services sont générés et déployés en production, il peut être difficile de refactoriser ces limites. Le choix de limites de service adéquates constitue un des plus grands défis lors de la conception d’une architecture de microservices. Quelle doit être la taille de chaque service ? Quand une fonctionnalité doit-elle être répartie entre plusieurs services, et quand est-il recommandé de la conserver à l’intérieur du même service ? Dans ce guide, nous décrivons une approche qui utilise la conception pilotée par le domaine pour déterminer les limites de service. Cette approche commence par l’[analyse de domaine](./domain-analysis.md) pour rechercher les contextes limités, puis elle applique un ensemble de [modèles DDD tactiques](./microservice-boundaries.md) selon les exigences fonctionnelles et non fonctionnelles.

- **Cohérence et intégrité des données** : un principe fondamental des microservices est que chaque service gère ses propres données. Cela sépare les services, mais peut entraîner des problèmes en matière d’intégrité ou de redondance des données. Nous abordons certains de ces problèmes dans les [considérations relatives aux données](./data-considerations.md).

- **Surcharge du réseau et latence** : l’utilisation de nombreux services granulaires de petite taille peut se traduire par une intensification des communications entre les services et une plus longue latence de bout en bout. Le chapitre sur la [communication entre les services](./interservice-communication.md) décrit les considérations relatives aux messages entre les services. Dans les architectures de microservices, la communication est à la fois synchrone et asynchrone. Une [conception d’API](./api-design.md) appropriée est importante afin de conserver une liaison souple entre les services, et pouvoir ainsi les déployer et les mettre à jour indépendamment.

- **Complexité** : une application de microservices possède plus de parties mobiles. Chaque service peut être simple, mais les services doivent fonctionner ensemble comme un tout. Une même opération utilisateur peut impliquer plusieurs services. Dans le chapitre [Ingestion et workflow](./ingestion-workflow.md), nous examinons certains problèmes relatifs à l’ingestion de requêtes à haut débit, à la coordination d’un workflow et à la gestion des erreurs.

- **Communication entre les clients et l’application** :  lorsque vous décomposez une application en de nombreux petits services, comment les clients communiquent-ils avec ces services ? Un client doit-il appeler directement chaque service ou acheminer les requêtes via une [passerelle d’API](./gateway.md) ?

- **Surveillance** : la surveillance d’une application distribuée peut s’avérer beaucoup plus difficile que celle d’une application monolithique, car vous devez corréler les données de télémétrie de plusieurs services. Le chapitre [Enregistrement et surveillance](./logging-monitoring.md) aborde ces problèmes.

- **Intégration et livraison continues** : un des principaux objectifs des microservices est l’agilité. Pour ce faire, vous devez disposer d’un processus [d’intégration et de livraison continues](./ci-cd.md) automatisé et robuste, afin de déployer rapidement et de façon fiable des services individuels dans les environnements de test et de production.

## <a name="the-drone-delivery-application"></a>Application de livraison par drone (Drone Delivery)

Pour explorer ces problèmes et illustrer certaines des meilleures pratiques pour une architecture de microservices, nous avons créé une implémentation de référence que nous appelons l’application Drone Delivery. Vous trouverez l’implémentation de référence sur [GitHub][drone-ri].

Fabrikam, Inc. veut lancer un service de livraison par drone. La société gère un parc de drones. Les entreprises souscrivent au service, et les utilisateurs peuvent demander à ce qu’un drone vienne récupérer les marchandises à livrer. Lorsqu’un client planifie un enlèvement de colis, un système principal assigne un drone et donne à l’utilisateur un délai de livraison estimé. Une fois la livraison en cours, le client peut suivre la position du drone, avec un ATE mis à jour en permanence.

Ce scénario implique un domaine assez complexe. Certaines entreprises veulent programmer des drones, suivre des colis, gérer les comptes d’utilisateur, ainsi que stocker et analyser les données d’historique. En outre, Fabrikam souhaite commercialiser ce service et effectuer une itération rapidement, en ajoutant des nouvelles fonctionnalités. L’application doit fonctionner à l’échelle du cloud, avec un objectif de niveau de service (SLO) élevé. Fabrikam prévoit également que les diverses parties du système auront des besoins très différents pour le stockage et l’interrogation des données. En s’appuyant sur toutes ces considérations, Fabrikam opte pour une architecture de microservices pour l’application Drone Delivery.

> [!NOTE]
> Pour éclairer votre choix entre une architecture de microservices et d’autres styles d’architecture, consultez le [Guide de l’architecture des applications Azure](../guide/index.md).

Notre implémentation de référence utilise Kubernetes avec [Azure Kubernetes Service](/azure/aks/) (AKS). Toutefois, la plupart des principaux défis et décisions architecturales s’appliquent à n’importe quel orchestrateur de conteneur, y compris [Azure Service Fabric](/azure/service-fabric/).

> [!div class="nextstepaction"]
> [Analyse de domaine](./domain-analysis.md)

<!-- links -->

[drone-ri]: https://github.com/mspnp/microservices-reference-implementation
