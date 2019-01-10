---
title: Modèles de messagerie
titleSuffix: Cloud Design Patterns
description: La nature distribuée des applications cloud nécessite une infrastructure de messagerie qui connecte les composants et les services, dans l’idéal, d’une manière faiblement couplée afin d’optimiser l’extensibilité. La messagerie asynchrone est largement utilisée et offre de nombreux avantages, mais elle apporte également des défis, tels que le classement des messages, la gestion des messages incohérents, l’idempotence et bien plus encore.
keywords: modèle de conception
author: dragon119
ms.date: 12/07/2018
ms.custom: seodec18
ms.openlocfilehash: 4619d30c152f050f3f95aee3f9983b8fe85911ed
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011325"
---
# <a name="messaging-patterns"></a><span data-ttu-id="2a471-105">Modèles de messagerie</span><span class="sxs-lookup"><span data-stu-id="2a471-105">Messaging patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="2a471-106">La nature distribuée des applications cloud nécessite une infrastructure de messagerie qui connecte les composants et les services, dans l’idéal, d’une manière faiblement couplée afin d’optimiser l’extensibilité.</span><span class="sxs-lookup"><span data-stu-id="2a471-106">The distributed nature of cloud applications requires a messaging infrastructure that connects the components and services, ideally in a loosely coupled manner in order to maximize scalability.</span></span> <span data-ttu-id="2a471-107">La messagerie asynchrone est largement utilisée et offre de nombreux avantages, mais elle apporte également des défis, tels que le classement des messages, la gestion des messages incohérents, l’idempotence et bien plus encore.</span><span class="sxs-lookup"><span data-stu-id="2a471-107">Asynchronous messaging is widely used, and provides many benefits, but also brings challenges such as the ordering of messages, poison message management, idempotency, and more.</span></span>

| <span data-ttu-id="2a471-108">Modèle</span><span class="sxs-lookup"><span data-stu-id="2a471-108">Pattern</span></span> | <span data-ttu-id="2a471-109">Résumé</span><span class="sxs-lookup"><span data-stu-id="2a471-109">Summary</span></span> |
| ------- | ------- |
| [<span data-ttu-id="2a471-110">Consommateurs concurrents</span><span class="sxs-lookup"><span data-stu-id="2a471-110">Competing Consumers</span></span>](../competing-consumers.md) | <span data-ttu-id="2a471-111">Activez plusieurs consommateurs simultanés pour traiter les messages reçus sur le même canal de messagerie.</span><span class="sxs-lookup"><span data-stu-id="2a471-111">Enable multiple concurrent consumers to process messages received on the same messaging channel.</span></span> |
| [<span data-ttu-id="2a471-112">Canaux et filtres</span><span class="sxs-lookup"><span data-stu-id="2a471-112">Pipes and Filters</span></span>](../pipes-and-filters.md) | <span data-ttu-id="2a471-113">Divisez une tâche qui exécute un traitement complexe en une série d’éléments séparés qui peuvent être réutilisés.</span><span class="sxs-lookup"><span data-stu-id="2a471-113">Break down a task that performs complex processing into a series of separate elements that can be reused.</span></span> |
| [<span data-ttu-id="2a471-114">File d’attente de priorité</span><span class="sxs-lookup"><span data-stu-id="2a471-114">Priority Queue</span></span>](../priority-queue.md) | <span data-ttu-id="2a471-115">Classez par ordre de priorité les requêtes envoyées aux services, de telle sorte que les demandes ayant une priorité plus élevée soient reçues et traitées plus rapidement que celles de moindre priorité.</span><span class="sxs-lookup"><span data-stu-id="2a471-115">Prioritize requests sent to services so that requests with a higher priority are received and processed more quickly than those with a lower priority.</span></span> |
| [<span data-ttu-id="2a471-116">Éditeur-abonné</span><span class="sxs-lookup"><span data-stu-id="2a471-116">Publisher-Subscriber</span></span>](../publisher-subscriber.md) | <span data-ttu-id="2a471-117">Activez une application pour annoncer des événements à plusieurs consommateurs intéressés de manière asynchrone, sans coupler les expéditeurs aux destinataires.</span><span class="sxs-lookup"><span data-stu-id="2a471-117">Enable an application to announce events to multiple interested consumers aynchronously, without coupling the senders to the receivers.</span></span> |
| [<span data-ttu-id="2a471-118">Nivellement de la charge basé sur une file d’attente</span><span class="sxs-lookup"><span data-stu-id="2a471-118">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md) | <span data-ttu-id="2a471-119">Utilisez une file d’attente qui agit comme mémoire tampon entre une tâche et un service qu’elle appelle, afin d’atténuer les surcharges intermittentes.</span><span class="sxs-lookup"><span data-stu-id="2a471-119">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span> |
| [<span data-ttu-id="2a471-120">Superviseur de l’agent du planificateur</span><span class="sxs-lookup"><span data-stu-id="2a471-120">Scheduler Agent Supervisor</span></span>](../scheduler-agent-supervisor.md) | <span data-ttu-id="2a471-121">Coordonnez un ensemble d’actions sur un ensemble distribué de services et d’autres ressources à distance.</span><span class="sxs-lookup"><span data-stu-id="2a471-121">Coordinate a set of actions across a distributed set of services and other remote resources.</span></span> |
