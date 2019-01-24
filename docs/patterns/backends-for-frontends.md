---
title: Modèle de services principaux destinés aux frontaux
titleSuffix: Cloud Design Patterns
description: Créez différents services principaux destinés à être utilisés par des applications ou interfaces frontales spécifiques.
keywords: modèle de conception
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 7a58da4c249250eaa789c39222e965e1cdf84002
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54487880"
---
# <a name="backends-for-frontends-pattern"></a>Modèle de services principaux destinés aux frontaux

Créez différents services principaux destinés à être utilisés par des applications ou interfaces frontales spécifiques. Ce modèle se révèle utile lorsque vous souhaitez personnaliser un même service principal pour plusieurs interfaces. Ce modèle a d’abord été décrit par Sam Newman.

## <a name="context-and-problem"></a>Contexte et problème

À l’origine, une application peut être destinée à une interface utilisateur web de bureau. Un service principal est généralement développé en parallèle et fournit les fonctionnalités requises par cette interface utilisateur. À mesure que la base d’utilisateurs de l’application augmente, une application mobile est développée et doit interagir avec le même service principal. Le service principal devient un service principal à usage général, répondant aussi bien aux exigences des interfaces de bureau qu’à celles des interfaces mobiles.

Toutefois, les fonctionnalités d’un appareil mobile diffèrent sensiblement de celles d’un navigateur de bureau en termes de taille d’écran, de performances et de limitations d’affichage. Par conséquent, les impératifs d’un service principal d’application mobile sont distincts de ceux d’une interface utilisateur web de bureau.

Le service principal doit donc satisfaire à des exigences contradictoires. Le service principal nécessite des modifications régulières et significatives pour desservir à la fois l’interface utilisateur web de bureau et l’application mobile. Étant donné que des équipes d’interface distinctes travaillent souvent sur chaque frontal, le service principal devient un goulot d’étranglement dans le processus de développement. Les exigences de mise à jour conflictuelles et l’obligation de maintenir le service opérationnel pour les deux frontaux peuvent nécessiter un travail considérable sur une seule ressource déployable.

![Diagramme de contexte et problème du modèle de services back-end destinés aux services front-end](./_images/backend-for-frontend.png)

L’activité de développement étant axée sur le service principal, il est possible qu’une équipe distincte soit créée pour gérer et mettre à jour le service principal. Les équipes de développement d’interface se retrouvent alors déconnectées de l’équipe de service principal, contraignant ainsi cette dernière à trouver un équilibre entre les exigences contradictoires des différentes équipes d’interface utilisateur. Lorsqu’une équipe d’interface requiert l’apport de modifications au service principal, ces modifications doivent être validées auprès des autres équipes d’interface avant de pouvoir être intégrées au service principal.

## <a name="solution"></a>Solution

Créez un service principal par interface utilisateur. Ajustez le comportement et les performances de chaque service principal pour qu’ils répondent au mieux aux besoins de l’environnement frontal, sans vous soucier d’affecter l’expérience des autres frontaux.

![Diagramme du modèle de services back-end destinés aux services front-end](./_images/backend-for-frontend-example.png)

Étant donné que chaque service principal est propre à une interface, il peut être optimisé pour cette dernière. Par conséquent, il sera moins volumineux, moins complexe et probablement plus rapide qu’un service principal générique qui tente de répondre aux exigences de toutes les interfaces. Chaque équipe d’interface a la possibilité de contrôler son propre service principal et ne dépend pas d’une équipe de développement de service principal centralisée. L’équipe d’interface dispose ainsi d’une réelle flexibilité en matière de sélection de langue, de cadence de mise en production, de hiérarchisation des charges de travail et d’intégration de fonctionnalités à son service principal.

Pour plus d’informations, consultez l’article [Pattern: Backends For Frontends](https://samnewman.io/patterns/architectural/bff/).

## <a name="issues-and-considerations"></a>Problèmes et considérations

- Considérez le nombre de services principaux à déployer.
- Si différentes interfaces (telles que des clients mobiles) doivent exécuter les mêmes requêtes, déterminez s’il est nécessaire d’implémenter un service principal pour chaque interface, ou si un seul service principal suffit.
- L’implémentation de ce modèle implique le plus souvent une duplication de code entre les services.
- Les services principaux axés sur un frontal doivent uniquement contenir la logique et le comportement propres au client. La logique métier générale et les autres fonctionnalités globales doivent être gérées à un autre emplacement de votre application.
- Pensez à la façon dont ce modèle peut se refléter dans les responsabilités d’une équipe de développement.
- Prenez en compte le temps que prendra l’implémentation de ce modèle. La tâche de création d’autres services principaux occasionnera-t-elle une dette technique, pendant que vous continuerez à prendre en charge le service principal générique existant ?

## <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle

Utilisez ce modèle dans les situations suivantes :

- Un service principal partagé ou à usage général doit être entretenu avec des coûts de développement substantiels.
- Vous devez optimiser le service principal pour les exigences d’interfaces clientes spécifiques.
- Des personnalisations sont appliquées à un service principal à usage général pour prendre en charge plusieurs interfaces.
- Une autre langue se révèle mieux adaptée au service principal d’une interface utilisateur différente.

Ce modèle peut ne pas convenir dans les cas suivants :

- Les interfaces envoient des requêtes identiques ou similaires au service principal.
- Une seule interface est utilisée pour interagir avec le service principal.

## <a name="related-guidance"></a>Aide connexe

- [Gateway Aggregation pattern](./gateway-aggregation.md) (Modèle d’agrégation de passerelle)
- [Modèle de déchargement de passerelle](./gateway-offloading.md)
- [Gateway Routing pattern](./gateway-routing.md) (Modèle de routage de passerelle)
