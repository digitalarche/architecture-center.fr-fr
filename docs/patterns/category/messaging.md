---
title: Modèles de messagerie
titleSuffix: Cloud Design Patterns
description: La nature distribuée des applications cloud nécessite une infrastructure de messagerie qui connecte les composants et les services, dans l’idéal, d’une manière faiblement couplée afin d’optimiser l’extensibilité. La messagerie asynchrone est largement utilisée et offre de nombreux avantages, mais elle apporte également des défis, tels que le classement des messages, la gestion des messages incohérents, l’idempotence et bien plus encore.
keywords: modèle de conception
author: dragon119
ms.date: 03/01/2018
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: f838288e10acf9af5776c93972028b6b878bd7b0
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640666"
---
# <a name="messaging-patterns"></a>Modèles de messagerie

La nature distribuée des applications cloud nécessite une infrastructure de messagerie qui connecte les composants et les services, dans l’idéal, d’une manière faiblement couplée afin d’optimiser l’extensibilité. La messagerie asynchrone est largement utilisée et offre de nombreux avantages, mais elle apporte également des défis, tels que le classement des messages, la gestion des messages incohérents, l’idempotence et bien plus encore.

| Modèle | Résumé |
| ------- | ------- |
| [Vérification des revendications](../claim-check.md) | Diviser un message volumineux en une vérification des revendications et une charge utile pour éviter de surcharger un bus de messages. |
| [Consommateurs concurrents](../competing-consumers.md) | Activez plusieurs consommateurs simultanés pour traiter les messages reçus sur le même canal de messagerie. |
| [Canaux et filtres](../pipes-and-filters.md) | Divisez une tâche qui exécute un traitement complexe en une série d’éléments séparés qui peuvent être réutilisés. |
| [File d’attente de priorité](../priority-queue.md) | Classez par ordre de priorité les requêtes envoyées aux services, de telle sorte que les demandes ayant une priorité plus élevée soient reçues et traitées plus rapidement que celles de moindre priorité. |
| [Éditeur-abonné](../publisher-subscriber.md) | Permettre à une application annoncer les événements à plusieurs consommateurs intéressées de façon asynchrone, sans couplage les expéditeurs aux récepteurs. |
| [Nivellement de la charge basé sur une file d’attente](../queue-based-load-leveling.md) | Utilisez une file d’attente qui agit comme mémoire tampon entre une tâche et un service qu’elle appelle, afin d’atténuer les surcharges intermittentes. |
| [Superviseur de l’agent du planificateur](../scheduler-agent-supervisor.md) | Coordonnez un ensemble d’actions sur un ensemble distribué de services et d’autres ressources à distance. |
