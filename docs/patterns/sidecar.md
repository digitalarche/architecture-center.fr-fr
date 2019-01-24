---
title: Modèle side-car
titleSuffix: Cloud Design Patterns
description: Déployez les composants d’une application sur un processus ou conteneur distinct pour fournir l’isolation et l’encapsulation.
keywords: modèle de conception
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 447400c543c27d8655ca2d8b1dbadd8d0fdb4ba1
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54486782"
---
# <a name="sidecar-pattern"></a>Modèle side-car

Déployez les composants d’une application sur un processus ou conteneur distinct pour fournir l’isolation et l’encapsulation. Ce modèle permet également aux applications d’être constituées de technologies et de composants hétérogènes.

Ce modèle est nommé *side-car*, car il ressemble à un side-car attaché à une moto. Dans ce modèle, le side-car est attaché à une application parente et fournit des fonctionnalités de prise en charge pour l’application. De plus, le side-car partage le même cycle de vie que l’application parente, étant créé et mis hors service en même temps que celle-ci. Le modèle side-car est parfois appelé « modèle assistant » et est un modèle de décomposition.

## <a name="context-and-problem"></a>Contexte et problème

Les applications et services nécessitent souvent des fonctionnalités connexes, telles que la surveillance, la journalisation, la configuration et les services réseau. Ces tâches périphériques peuvent être implémentées en tant que composants ou services distincts.

Si elles sont étroitement intégrées à l’application, elles peuvent s’exécuter dans le même processus que l’application, optimisant l’utilisation des ressources partagées. Toutefois, cela signifie également qu’elles ne sont pas bien isolées ; une panne dans un de ces composants peut affecter les autres composants ou l’ensemble de l’application. De plus, elles doivent généralement être implémentées à l’aide du même langage que l’application parente. Ainsi, le composant et l’application sont étroitement interdépendants.

Si l’application est décomposée en services, chaque service peut être généré à l’aide de technologies et de langages différents. Bien que cela procure davantage de souplesse, chaque composant a ses propres dépendances et nécessite des bibliothèques spécifiques au langage pour accéder à la plateforme sous-jacente et à toutes les ressources partagées avec l’application parente. Par ailleurs, le déploiement de ces fonctionnalités en tant que services distincts peut ajouter de la latence à l’application. La gestion du code et des dépendances pour ces interfaces spécifiques au langage peut également ajouter une part importante de complexité, en particulier pour l’hébergement, le déploiement et la gestion.

## <a name="solution"></a>Solution

Colocalisez un ensemble cohérent de tâches avec l’application principale, mais placez-les à l’intérieur de leur propre processus ou conteneur, en fournissant une interface homogène pour les services de plateforme entre les langages.

![Diagramme du modèle Sidecar](./_images/sidecar.png)

Un service side-car ne fait pas nécessairement partie de l’application, mais il est connecté à celle-ci. Il est présent partout où l’application parente est présente. Les side-cars sont des processus ou services de prise en charge qui sont déployés avec l’application principale. Dans le cas d’une motocyclette, le side-car est attaché à celle-ci, et chaque motocyclette peut avoir son propre side-car. De la même façon, un service side-car partage le devenir de son application parente. Pour chaque instance de l’application, une instance du side-car est déployée et hébergée à ses côtés.

Avantages de l’utilisation d’un modèle side-car :

- Un side-car étant indépendant de son application principale en termes d’environnement d’exécution et de langage de programmation, vous n’avez pas besoin de développer un side-car par langage.

- Le side-car peut accéder aux mêmes ressources que l’application principale. Par exemple, un side-car peut surveiller les ressources système utilisées par lui-même et l’application principale.

- En raison de sa proximité avec l’application principale, il n’existe aucune latence importante quand l’un et l’autre communiquent.

- Même pour les applications qui ne fournissent pas de mécanisme d’extensibilité, vous pouvez utiliser un side-car pour étendre les fonctionnalités en le joignant en tant que processus propre dans le même hôte ou sous-conteneur que l’application principale.

Le modèle side-car est souvent utilisé avec des conteneurs et est appelé « conteneur side-car » ou « conteneur assistant ».

## <a name="issues-and-considerations"></a>Problèmes et considérations

- Envisagez le format de déploiement et d’empaquetage à utiliser pour déployer des services, processus ou conteneurs. Les conteneurs sont particulièrement bien adaptés au modèle side-car.
- Quand vous concevez un service side-car, choisissez avec soin le mécanisme de communication entre processus. Essayez d’utiliser des technologies indépendantes du langage ou du framework, sauf si des exigences de niveau de performance rendent cela peu pratique.
- Avant de placer des fonctionnalités dans un side-car, déterminez si elles ne seraient pas plus efficaces sous la forme d’un service distinct ou d’un démon plus traditionnel.
- Déterminez également si les fonctionnalités peuvent être implémentées en tant que bibliothèque ou à l’aide d’un mécanisme d’extension traditionnel. Les bibliothèques spécifiques au langage peuvent offrir un niveau d’intégration plus profond et une surcharge réseau inférieure.

## <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle

Utilisez ce modèle dans les situations suivantes :

- Votre application principale utilise un ensemble hétérogène de langages et de frameworks. Un composant situé dans un service side-car peut être utilisé par des applications écrites dans des langages différents à l’aide de différents frameworks.
- Un composant est détenu par une équipe à distance ou une autre organisation.
- Un composant ou fonctionnalité doit coexister sur le même hôte que l’application.
- Vous avez besoin d’un service qui partage le cycle de vie global de votre application principale, mais qui peut être mis à jour indépendamment.
- Vous avez besoin d’un contrôle affiné des limites de ressources pour une ressource ou un composant particulier. Par exemple, vous souhaitez limiter la quantité de mémoire utilisée par un composant spécifique. Vous pouvez déployer le composant en tant que side-car et gérer l’utilisation de la mémoire indépendamment de l’application principale.

Ce modèle peut ne pas convenir :

- Quand la communication entre processus doit être optimisée. La communication entre une application parente et des services side-car engendre une surcharge, notamment de la latence dans les appels. Ce compromis peut ne pas convenir pour les interfaces gourmandes en communication.
- Pour les petites applications où l’avantage procuré par l’isolation ne contrebalance pas le coût en ressources lié au déploiement d’un service side-car pour chaque instance.
- Quand le service doit se mettre à l’échelle différemment ou indépendamment des applications principales. Dans ce cas, il peut être préférable de déployer la fonctionnalité en tant que service distinct.

## <a name="example"></a>Exemples

Ce modèle side-car s’applique à de nombreux scénarios. Voici quelques exemples courants :

- API d’infrastructure. L’équipe de développement de l’infrastructure crée un service qui est déployé en même temps que chaque application, au lieu d’une bibliothèque cliente spécifique au langage pour accéder à l’infrastructure. Le service est chargé en tant que side-car et fournit une couche commune pour les services de l’infrastructure, notamment la journalisation, les données de l’environnement, le magasin de configuration, la découverte, les contrôles d’intégrité et les services de surveillance. De plus, le side-car surveille le processus (ou conteneur) et l’environnement hôte de l’application parente et journalise les informations dans un service centralisé.
- Gérer NGINX/HAProxy. Déployer NGINX avec un service side-car qui surveille l’état de l’environnement, puis met à jour le fichier de configuration NGINX et recycle le processus quand une modification d’état est nécessaire.
- Side-car ambassadeur. Déployez un service [Ambassadeur](./ambassador.md) en tant que side-car. L’application fait appel à l’ambassadeur, qui gère la journalisation des demandes, le routage, la rupture de circuit (circuit breaking) et autres fonctionnalités liées à la connectivité.
- Proxy de déchargement. Placer un proxy NGINX devant une instance de service node.js afin de gérer la fourniture de contenu de fichier statique pour le service.

## <a name="related-guidance"></a>Aide connexe

- [Modèle ambassadeur](./ambassador.md)
