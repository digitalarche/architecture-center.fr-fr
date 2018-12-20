---
title: Modèles de messagerie
description: La nature distribuée des applications cloud nécessite une infrastructure de messagerie qui connecte les composants et les services, dans l’idéal, d’une manière faiblement couplée afin d’optimiser l’extensibilité. La messagerie asynchrone est largement utilisée et offre de nombreux avantages, mais elle apporte également des défis, tels que le classement des messages, la gestion des messages incohérents, l’idempotence et bien plus encore.
keywords: modèle de conception
author: dragon119
ms.date: 12/07/2018
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 754e9a7dcad20dc1c4471af00f3f142f18022d62
ms.sourcegitcommit: a0a9981e7586bed8d876a54e055dea1e392118f8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2018
ms.locfileid: "53233878"
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
