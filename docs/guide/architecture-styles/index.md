---
title: Styles d’architecture
description: Styles d’architecture courants pour les applications cloud
layout: LandingPage
ms.date: 08/30/2018
ms.openlocfilehash: b192cfc4168cc6f73e191a4ec8a5a83f1986aee7
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/31/2018
ms.locfileid: "43326270"
---
# <a name="architecture-styles"></a>Styles d’architecture

Un *style d’architecture* est une famille d’architectures qui partagent certaines caractéristiques. Par exemple, [multiniveau][n-tier] est un style d’architecture courant. Plus récemment, [les architectures de microservice][microservices] ont commencé à obtenir une place de choix. Les styles d’architecture ne nécessitent pas l’utilisation de technologies particulières, mais certaines technologies sont bien adaptées pour certaines architectures. Par exemple, les conteneurs sont naturellement adaptés aux microservices.  

Nous avons identifié un ensemble de styles d’architecture qui sont couramment utilisés dans les applications cloud. L’article inclut pour chaque style :

- Une description et un diagramme logique du style.
- Des recommandations concernant le moment approprié pour choisir le style.
- Les avantages, les défis et les meilleures pratiques.
- Un déploiement recommandé à l’aide des services Azure appropriés.


## <a name="a-quick-tour-of-the-styles"></a>Une présentation rapide des styles.   

Cette section fournit une présentation rapide des styles d’architecture que nous avons identifiés, ainsi que certaines considérations de haut niveau pour leur utilisation. Obtenez plus de détails dans les rubriques associées.

### <a name="n-tier"></a>Multiniveau

<img src="./images/n-tier-sketch.svg" style="float:left; margin-top:6px;"/>

**[Multiniveau][n-tier]** est une architecture classique pour les applications d’entreprise. Les dépendances sont gérées en divisant l’application en *couches* qui exécutent des fonctions logiques, telles que la présentation, la logique métier et l’accès aux données. Une couche ne peut appeler que des couches inférieures. Toutefois, cette superposition horizontale peut être une responsabilité. Il peut être difficile d’introduire des modifications dans une partie de l’application sans toucher le reste de l’application. Ainsi, les mises à jour fréquentes sont un défi, elles limitent la rapidité avec laquelle de nouvelles fonctionnalités peuvent être ajoutées.

L’architecture multiniveau convient parfaitement pour la migration d’applications existantes qui utilisent déjà une architecture en couches. Pour cette raison, l’architecture multiniveau est le plus souvent vue dans les solutions de type infrastructure as a service (IaaS), ou comme une application qui utilise une combinaison de IaaS et de services gérés. 

### <a name="web-queue-worker"></a>Web-File d’attente-Worker

<img src="./images/web-queue-worker-sketch.svg" style="float:left; margin-top:6px;"/>

Pour une solution purement PaaS, envisagez une architecture **[Web-File d’attente-Worker](./web-queue-worker.md)**. Dans ce style, l’application possède un serveur web frontal qui gère les requêtes HTTP et un Worker principal qui exécute les tâches sollicitant beaucoup le processeur ou les opérations longues. Le serveur frontal communique avec le Worker via une file d’attente de messages asynchrones. 

L’architecture Web-File d’attente-Worker est adaptée aux domaines relativement simples avec certaines tâches gourmandes en ressources. Comme pour l’architecture multiniveau, cette architecture est facile à comprendre. L’utilisation de services gérés simplifie le déploiement et les opérations. Mais avec des domaines complexes, il peut être difficile de gérer les dépendances. Le serveur frontal et le Worker peuvent facilement devenir des composants monolithiques volumineux, difficiles à gérer et à mettre à jour. Comme avec l’architecture multicouche, cela peut réduire la fréquence des mises à jour et limiter l’innovation.

### <a name="microservices"></a>Microservices

<img src="./images/microservices-sketch.svg" style="float:left; margin-top:6px;"/>

Si votre application possède un domaine plus complexe, envisagez de passer à une architecture **[microservices][microservices]**. Une application de microservices est composée de nombreux services indépendants et de petite taille. Chaque service implémente une fonctionnalité unique. Les services sont faiblement couplés et communiquent via des contrats d’API.

Chaque service peut être généré par une équipe de développement petite et ciblée. Les services individuels peuvent être déployés sans beaucoup de coordination entre les équipes, ce qui permet des mises à jour fréquentes. Une architecture de microservices est plus complexe à créer et à gérer que les architectures multiniveau ou Web-File d’attente-Worker. Elle nécessite un développement mature et une culture DevOps. Mais bien mis en place, ce style peut libérer plus de rapidité, accélérer l’innovation et permettre une architecture plus résiliente. 

### <a name="cqrs"></a>CQRS

<img src="./images/cqrs-sketch.svg" style="float:left; margin-top:6px;"/>

Le style **[CQRS](./cqrs.md)** (Séparation des responsabilités en matière de commande et de requête) sépare les opérations de lecture et d’écriture en plusieurs modèles distincts. Cela permet d’isoler les parties du système qui mettent à jour les données des parties qui les lisent. En outre, les lectures peuvent être exécutées sur une vue matérialisée qui est physiquement séparée de la base de données d’écriture. Cela vous permet de mettre à l’échelle les charges de travail de lecture et d’écriture indépendamment et d’optimiser la vue matérialisée pour les requêtes.

Le style CQRS est le plus approprié lorsqu’il est appliqué à un sous-système d’une architecture plus vaste. En règle générale, vous ne devriez pas l’imposer à toute l’application, cela ne ferait que créer une complexité inutile. Considérez-la comme pour les domaines de collaboration où de nombreux utilisateurs accèdent aux mêmes données.

### <a name="event-driven-architecture"></a>Architecture basée sur les événements 

<img src="./images/event-driven-sketch.svg" style="float:left; margin-top:6px;"/>

Les **[architectures basées sur les événements](./event-driven.md)** utilisent un modèle de publication-abonnement (pub-sub), où les producteurs publient des événements et les consommateurs s’y abonnent. Les producteurs sont indépendants des consommateurs et les consommateurs sont indépendants les uns des autres. 

Envisagez une architecture basée sur les événements pour les applications qui reçoivent et traitent un gros volume de données avec une latence très faible, telles que les solutions IoT. Le style est également utile lorsque différents sous-systèmes doivent effectuer différents types de traitement sur les mêmes données d’événement.

<br />

### <a name="big-data-big-compute"></a>Big Data et Big Compute

**[Big Data](./big-data.md)** et **[Big Compute](./big-compute.md)** sont des styles d’architecture spécialisée pour les charges de travail qui correspondent à certains profils spécifiques. Big Data divise un jeu de données très volumineux en plusieurs blocs, exécutant un traitement parallèle sur l’ensemble du jeu de données, permettant l’analyse et la création de rapports. Big compute, également appelé High-Performance Computing, calcul haute performance (HPC), effectue des calculs parallèles sur un grand nombre (des milliers) de cœurs. Les domaines incluent des simulations, des modélisations et un rendu 3D.

## <a name="architecture-styles-as-constraints"></a>Les styles d’architecture en tant que contraintes

Un style d’architecture implique des contraintes de conception, y compris l’ensemble d’éléments qui peuvent s’afficher et les relations autorisées entre ces éléments. Les contraintes encadrent la « forme » d’une architecture en limitant les choix possibles. Lorsqu’une architecture se conforme aux limites d’un style particulier, certaines propriétés souhaitables émergent. 

Les contraintes de microservices incluent, par exemple : 

- Un service représente une seule responsabilité. 
- Chaque service est indépendant des autres. 
- Les données sont privées et accessibles uniquement au service auquel elles appartiennent. Les services ne partagent pas de données.

En adhérant à ces contraintes, ce qui apparaît est un système où les services peuvent être déployés indépendamment, les erreurs sont isolées, les mises à jour fréquentes sont possibles et il est facile d’introduire de nouvelles technologies dans l’application.

Avant de choisir un style d’architecture, assurez-vous de comprendre les principes sous-jacents et les contraintes de ce style. Dans le cas contraire, vous pouvez vous retrouver avec une conception qui est conforme au style à un niveau superficiel, mais qui n’atteint pas tout le potentiel de ce style. Il est également important d’être pragmatique. Il est parfois préférable d’assouplir une contrainte, plutôt que d’insister sur la pureté de l’architecture.


Le tableau suivant résume la façon dont chaque style gère les dépendances et les types de domaine qui sont le mieux adaptés pour chacun.

| Style d’architecture |  Gestion des dépendances | Type de domaine |
|--------------------|------------------------|-------------|
| Multiniveau | Couches horizontales divisées par un sous-réseau. | Domaine d’entreprise classique. La fréquence des mises à jour est faible. |
| Web-File d’attente-Worker | Travaux frontaux et principaux, découplés par messagerie asynchrone. | Domaine relativement simple avec des tâches gourmandes en ressources. |
| Microservices | Services verticalement (fonctionnellement) décomposés qui s’appellent mutuellement via des API. | Domaine complexe. Mises à jour fréquentes. |
| CQRS | Répartition en lecture/écriture. Le schéma et l’échelle sont optimisés séparément. | Domaine de collaboration où un grand nombre d’utilisateurs accèdent aux mêmes données. |
| Architecture basée sur les événements. | Producteur/consommateur. Vue indépendante par sous-système. | IoT et systèmes en temps réel. |
| Données volumineuses (« Big Data ») | Divise un jeu de données volumineux en petits blocs. Traitement parallèle sur les jeux de données locaux. | Analyse des données en mode batch et en temps réel Analyse prédictive à l’aide de ML. |
| Big Compute| Allocation des données à des milliers de cœurs. | Domaines nécessitant beaucoup de ressources système, tels que la simulation. |


## <a name="consider-challenges-and-benefits"></a>Prenez en compte les avantages et les défis

Les contraintes créent également des défis, il est donc important de comprendre les inconvénients lors de l’adoption d’un de ces styles. Les avantages peuvent l’emporter sur les défis liés au style architecture, *pour ce sous-domaine et le contexte limité*. 

Voici quelques types de problèmes à prendre en compte lors de la sélection d’un style d’architecture :

- **Complexité**. La complexité de l’architecture est-elle justifiée pour votre domaine ? À l’inverse, le style est-il trop simpliste pour votre domaine ? Dans ce cas, vous risquez de vous retrouver avec une «[boule de boue][ball-of-mud]» (ball of mud), car l’architecture ne vous aide pas à gérer correctement les dépendances.

- **Messagerie asynchrone et cohérence éventuelle**. La messagerie asynchrone peut être utilisée pour séparer des services et améliorer la fiabilité (étant donné que les messages peuvent être tentés à nouveau) et l’extensibilité. Toutefois, cela crée également des défis comme la sémantique Toujours une fois et la cohérence éventuelle.

- **Communication interservice** Lorsque vous décomposez une application en services distincts, il est possible que la communication entre services provoque une latence inacceptable ou crée une congestion du réseau (par exemple, dans une architecture microservices). 

- **Facilité de gestion**. Est-il difficile de gérer l’application, analyser, déployer des mises à jour et ainsi de suite ?


[ball-of-mud]: https://en.wikipedia.org/wiki/Big_ball_of_mud
[microservices]: ./microservices.md
[n-tier]: ./n-tier.md
