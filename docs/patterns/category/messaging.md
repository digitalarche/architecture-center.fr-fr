---
title: Modèles de messagerie
titleSuffix: Cloud Design Patterns
description: La nature distribuée des applications cloud nécessite une infrastructure de messagerie qui connecte les composants et les services, dans l’idéal, d’une manière faiblement couplée afin d’optimiser l’extensibilité. La messagerie asynchrone est largement utilisée et offre de nombreux avantages, mais elle apporte également des défis, tels que le classement des messages, la gestion des messages incohérents, l’idempotence et bien plus encore.
keywords: modèle de conception
author: dragon119
ms.date: 12/07/2018
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 2f8a34afc5e6c427bf5635b7a3be316719211112
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54485160"
---
# <a name="messaging-patterns"></a>Modèles de messagerie

[!INCLUDE [header](../../_includes/header.md)]

La nature distribuée des applications cloud nécessite une infrastructure de messagerie qui connecte les composants et les services, dans l’idéal, d’une manière faiblement couplée afin d’optimiser l’extensibilité. La messagerie asynchrone est largement utilisée et offre de nombreux avantages, mais elle apporte également des défis, tels que le classement des messages, la gestion des messages incohérents, l’idempotence et bien plus encore.

| Modèle | Résumé |
| ------- | ------- |
| [Consommateurs concurrents](../competing-consumers.md) | Activez plusieurs consommateurs simultanés pour traiter les messages reçus sur le même canal de messagerie. |
| [Canaux et filtres](../pipes-and-filters.md) | Divisez une tâche qui exécute un traitement complexe en une série d’éléments séparés qui peuvent être réutilisés. |
| [File d’attente de priorité](../priority-queue.md) | Classez par ordre de priorité les requêtes envoyées aux services, de telle sorte que les demandes ayant une priorité plus élevée soient reçues et traitées plus rapidement que celles de moindre priorité. |
| [Éditeur-abonné](../publisher-subscriber.md) | Activez une application pour annoncer des événements à plusieurs consommateurs intéressés de manière asynchrone, sans coupler les expéditeurs aux destinataires. |
| [Nivellement de la charge basé sur une file d’attente](../queue-based-load-leveling.md) | Utilisez une file d’attente qui agit comme mémoire tampon entre une tâche et un service qu’elle appelle, afin d’atténuer les surcharges intermittentes. |
| [Superviseur de l’agent du planificateur](../scheduler-agent-supervisor.md) | Coordonnez un ensemble d’actions sur un ensemble distribué de services et d’autres ressources à distance. |
