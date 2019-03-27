---
title: Modèles de disponibilité
titleSuffix: Cloud Design Patterns
description: La disponibilité définit la durée pendant laquelle le système est fonctionnel. Elle sera affectée par des erreurs système, des problèmes d’infrastructure, des attaques malveillantes ou la charge du système. Elle est habituellement mesurée en pourcentage du temps d’activité. En général, les applications cloud fournissent aux utilisateurs un contrat de niveau de service (SLA), ce qui signifie que les applications doivent être conçues et implémentées de façon à optimiser la disponibilité.
keywords: modèle de conception
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 897bc3c87beb23770220ef0fc3c5c4394d192f2a
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58243040"
---
# <a name="availability-patterns"></a>Modèles de disponibilité

[!INCLUDE [header](../../_includes/header.md)]

La disponibilité définit la durée pendant laquelle le système est fonctionnel. Elle sera affectée par des erreurs système, des problèmes d’infrastructure, des attaques malveillantes ou la charge du système. Elle est habituellement mesurée en pourcentage du temps d’activité. En général, les applications cloud fournissent aux utilisateurs un contrat de niveau de service (SLA), ce qui signifie que les applications doivent être conçues et implémentées de façon à optimiser la disponibilité.

|                            Modèle                             |                                                           Résumé                                                            |
|----------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------|
| [Surveillance de point de terminaison d’intégrité](../health-endpoint-monitoring.md) | Implémentez des contrôles fonctionnels dans une application à laquelle des outils externes peuvent accéder par le biais de points de terminaison exposés à intervalles réguliers. |
|  [Nivellement de la charge basé sur une file d’attente](../queue-based-load-leveling.md)  | Utilisez une file d’attente qui agit comme mémoire tampon entre une tâche et un service qu’elle appelle, afin d’atténuer les surcharges intermittentes.  |
|                 [Limitation](../throttling.md)                 |   Contrôlez la consommation des ressources utilisées par une instance d’une application, un locataire ou un service entier.    |
