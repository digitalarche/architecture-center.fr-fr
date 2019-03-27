---
title: Modèle d’étranglement
titleSuffix: Cloud Design Patterns
description: Faites migrer un système hérité de façon incrémentielle en remplaçant progressivement des parties spécifiques des fonctionnalités par de nouveaux services et applications.
keywords: modèle de conception
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: f139d368c98256c0190753930983a47df81a5134
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58241450"
---
# <a name="strangler-pattern"></a>Modèle d’étranglement

Faites migrer un système hérité de façon incrémentielle en remplaçant progressivement des parties spécifiques des fonctionnalités par de nouveaux services et applications. Lorsque les fonctionnalités d’un système hérité sont remplacées, le nouveau système finit par remplacer toutes les fonctionnalités de l’ancien système. Il étrangle alors l’ancien système et vous permet ainsi de le désactiver.

## <a name="context-and-problem"></a>Contexte et problème

À mesure que les systèmes vieillissent, les outils de développement, la technologie d’hébergement et même les architectures système sur lesquels reposent ces systèmes peuvent devenir de plus en plus obsolètes. L’ajout progressif de nouvelles fonctions et fonctionnalités peut considérablement accroître la complexité de ces applications et compliquer ainsi la mise à jour ou l’enrichissement de ces dernières.

Le remplacement complet d’un système complexe peut constituer un défi de taille. Vous serez généralement contraint d’effectuer une migration progressive vers un nouveau système, tout en conservant l’ancien système pour gérer les fonctionnalités qui n’ont pas encore fait l’objet d’une migration. Toutefois, l’exécution de deux versions distinctes d’une application signifie que les clients doivent connaître l’emplacement des différentes fonctionnalités. Chaque fois que vous faites migrer une fonctionnalité ou un service, vous devez donc mettre à jour les clients pour qu’ils pointent vers le nouvel emplacement.

## <a name="solution"></a>Solution

Remplacez progressivement des parties spécifiques des fonctionnalités par de nouveaux services et applications. Créez une façade qui intercepte les requêtes destinées au système hérité principal. La façade route ces requêtes soit vers l’application héritée, soit vers les nouveaux services. Les fonctionnalités existantes peuvent faire l’objet d’une migration progressive vers le nouveau système, et les clients peuvent continuer à utiliser la même interface sans savoir qu’une migration a été effectuée.

![Diagramme du modèle d’étranglement](./_images/strangler.png)

Ce modèle contribue à minimiser les risques liés à la migration et à étaler l’effort de développement dans le temps. Étant donné que la façade route les utilisateurs en toute sécurité vers l’application adéquate, vous pouvez ajouter des fonctionnalités au nouveau système à votre rythme, tout en ayant l’assurance que l’application héritée continue à fonctionner. Avec le temps, à mesure que les fonctionnalités font l’objet d’une migration vers le nouveau système, le système hérité finit par être « étranglé » et par devenir inutile. Une fois ce processus terminé, le système hérité peut alors être mis hors service sans problème.

## <a name="issues-and-considerations"></a>Problèmes et considérations

- Déterminez comment gérer les services et les magasins de données potentiellement utilisés à la fois par un nouveau système et par un système hérité. Assurez-vous que les deux systèmes peuvent accéder à ces ressources côte à côte.
- Structurez les nouveaux services et applications de façon à faciliter leur interception et leur remplacement dans les futures migrations d’étranglement.
- À un moment donné, une fois la migration terminée, la façade d’étranglement finit soit par disparaître, soit par évoluer sous la forme d’un adaptateur pour les clients hérités.
- Assurez-vous que la façade suive le rythme de la migration.
- Assurez-vous que la façade ne devienne pas un point de défaillance unique ou un goulot d’étranglement.

## <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle

Utilisez ce modèle lorsque vous faites migrer progressivement une application principale vers une nouvelle architecture.

Ce modèle peut ne pas convenir :

- lorsque les requêtes adressées au système principal ne peuvent pas être interceptées ;
- pour les systèmes plus modestes dont le remplacement global présente peu de difficultés.

## <a name="related-guidance"></a>Aide connexe

- Billet de blog de Martin Fowler sur [StranglerApplication](https://www.martinfowler.com/bliki/StranglerApplication.html)
