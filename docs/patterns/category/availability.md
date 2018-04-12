---
title: Modèles de disponibilité
description: La disponibilité définit la durée pendant laquelle le système est fonctionnel. Elle sera affectée par des erreurs système, des problèmes d’infrastructure, des attaques malveillantes ou la charge du système. Elle est habituellement mesurée en pourcentage du temps d’activité. En général, les applications cloud fournissent aux utilisateurs un contrat de niveau de service (SLA), ce qui signifie que les applications doivent être conçues et implémentées de façon à optimiser la disponibilité.
keywords: modèle de conception
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: eac1d192fe83f8501d11dce02d9bec16e939ba55
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/06/2018
---
# <a name="availability-patterns"></a>Modèles de disponibilité

[!INCLUDE [header](../../_includes/header.md)]

La disponibilité définit la durée pendant laquelle le système est fonctionnel. Elle sera affectée par des erreurs système, des problèmes d’infrastructure, des attaques malveillantes ou la charge du système. Elle est habituellement mesurée en pourcentage du temps d’activité. En général, les applications cloud fournissent aux utilisateurs un contrat de niveau de service (SLA), ce qui signifie que les applications doivent être conçues et implémentées de façon à optimiser la disponibilité.


|                            Modèle                             |                                                           Résumé                                                            |
|----------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------|
| [Surveillance de point de terminaison d’intégrité](../health-endpoint-monitoring.md) | Implémentez des contrôles fonctionnels dans une application à laquelle des outils externes peuvent accéder par le biais de points de terminaison exposés à intervalles réguliers. |
|  [Nivellement de la charge basé sur une file d’attente](../queue-based-load-leveling.md)  | Utilisez une file d’attente qui agit comme mémoire tampon entre une tâche et un service qu’elle appelle, afin d’atténuer les surcharges intermittentes.  |
|                 [Limitation](../throttling.md)                 |   Contrôlez la consommation des ressources utilisées par une instance d’une application, un locataire ou un service entier.    |

