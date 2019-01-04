---
title: Anti-modèles de performance pour les applications cloud
titleSuffix: Azure Architecture Center
description: Pratiques courantes susceptibles de provoquer des problèmes d’extensibilité.
author: dragon119
ms.date: 06/05/2017
ms.custom: seodec18
ms.openlocfilehash: 9ce1177bac2c93139faf6bc757f2866d6b7eac2d
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/04/2019
ms.locfileid: "54009727"
---
# <a name="performance-antipatterns-for-cloud-applications"></a>Anti-modèles de performance pour les applications cloud

Un *anti-modèle de performance* est une pratique courante susceptible de causer des problèmes d’extensibilité lorsqu’une application est sous pression.

Voici un scénario courant : une application se comporte correctement lors du test de performance. Elle est mise en production et commence à gérer des charges de travail réelles. À ce stade, elle commence à fonctionner difficilement &mdash; en rejetant les demandes d’utilisateur, en se bloquant ou en lançant des exceptions. L’équipe de développement est alors confrontée à deux questions :

- Pourquoi ce comportement n’est-il pas apparu pendant les tests ?
- Comment le corriger ?

La réponse à la première question est simple. Il est très difficile de simuler de vrais utilisateurs dans un environnement de test,de même que leurs modèles de comportement et leurs volumes de travail possibles. La seule façon permettant de comprendre avec certitude comment un système se comporte sous charge consiste à l’observer en production. Pour être clair, nous ne vous suggérons pas d’ignorer les tests de performances. Les tests de performances sont essentiels pour obtenir des mesures de performances de référence. Mais vous devez être prêt à observer et corriger des problèmes de performances lorsqu’ils se produisent dans le système réel.

La réponse à la deuxième question (comment résoudre le problème) est moins simple. Un grand nombre de facteurs peuvent y contribuer, et il arrive que le problème ne se manifeste que dans certaines circonstances. L’instrumentation et la journalisation sont essentielles pour rechercher la cause racine, mais vous devez également savoir quoi rechercher.

À partir de nos engagements avec les clients de Microsoft Azure, nous avons identifié certains des problèmes de performances les plus courants auxquels ils sont confrontés en production. Pour chaque anti-modèle, nous décrivons pourquoi il se produit généralement, ses symptômes et les techniques pour résoudre le problème. Nous fournissons également des exemples de code illustrant l’anti-modèle et une suggestion de solution.

Certains de ces anti-modèles peuvent sembler évidents lorsque vous lisez les descriptions, mais ils se produisent plus souvent que vous ne le pensez. Parfois, une application hérite d’une conception qui a fonctionné localement, mais qui ne s’étend pas dans le cloud. Ou alors une application peut démarrer avec une conception très claire mais l’ajout de nouvelles fonctionnalités peut faire apparaître un ou plusieurs de ces anti-modèles. Dans tous les cas, ce guide vous aidera à identifier et résoudre ces anti-modèles.

Voici la liste des anti-modèles que nous avons identifiés :

| Anti-modèle | Description |
|-------------|-------------|
| [Base de données occupée][BusyDatabase] | Déchargement d’un traitement excessif dans un magasin de données. |
| [Serveur frontal occupé][BusyFrontEnd] | Déplacement de tâches gourmandes en ressources vers des threads d’arrière-plan. |
| [E/S bavardes][ChattyIO] | Envoi continu de nombreuses petites requêtes réseau. |
| [Récupération superflus][ExtraneousFetching] | Récupération de données plus que nécessaire, entraînant des E/S inutiles. |
| [Instanciation incorrecte][ImproperInstantiation] | Création et destruction répétées d’objets conçus pour être partagés et réutilisés. |
| [Persistance monolithique][MonolithicPersistence] | Utilisation d’un même magasin de données pour des données ayant des modèles d’utilisation très différents. |
| [Pas de mise en cache][NoCaching] | Échec de la mise en cache des données. |
| [E/S synchrone][SynchronousIO] | Blocage du thread appelant lorsque l’E/S se termine. |

[BusyDatabase]: ./busy-database/index.md
[BusyFrontEnd]: ./busy-front-end/index.md
[ChattyIO]: ./chatty-io/index.md
[ExtraneousFetching]: ./extraneous-fetching/index.md
[ImproperInstantiation]: ./improper-instantiation/index.md
[MonolithicPersistence]: ./monolithic-persistence/index.md
[NoCaching]: ./no-caching/index.md
[SynchronousIO]: ./synchronous-io/index.md
