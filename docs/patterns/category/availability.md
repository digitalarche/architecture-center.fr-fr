---
title: Modèles de disponibilité
titleSuffix: Cloud Design Patterns
description: La disponibilité définit la durée pendant laquelle le système est fonctionnel. Elle sera affectée par des erreurs système, des problèmes d’infrastructure, des attaques malveillantes ou la charge du système. Elle est habituellement mesurée en pourcentage du temps d’activité. En général, les applications cloud fournissent aux utilisateurs un contrat de niveau de service (SLA), ce qui signifie que les applications doivent être conçues et implémentées de façon à optimiser la disponibilité.
keywords: modèle de conception
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: d256ddd39ca3e8a452162b7b75d67f26f7bb22c2
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/04/2019
ms.locfileid: "54009812"
---
# <a name="availability-patterns"></a><span data-ttu-id="ace43-107">Modèles de disponibilité</span><span class="sxs-lookup"><span data-stu-id="ace43-107">Availability patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="ace43-108">La disponibilité définit la durée pendant laquelle le système est fonctionnel.</span><span class="sxs-lookup"><span data-stu-id="ace43-108">Availability defines the proportion of time that the system is functional and working.</span></span> <span data-ttu-id="ace43-109">Elle sera affectée par des erreurs système, des problèmes d’infrastructure, des attaques malveillantes ou la charge du système.</span><span class="sxs-lookup"><span data-stu-id="ace43-109">It will be affected by system errors, infrastructure problems, malicious attacks, and system load.</span></span> <span data-ttu-id="ace43-110">Elle est habituellement mesurée en pourcentage du temps d’activité.</span><span class="sxs-lookup"><span data-stu-id="ace43-110">It is usually measured as a percentage of uptime.</span></span> <span data-ttu-id="ace43-111">En général, les applications cloud fournissent aux utilisateurs un contrat de niveau de service (SLA), ce qui signifie que les applications doivent être conçues et implémentées de façon à optimiser la disponibilité.</span><span class="sxs-lookup"><span data-stu-id="ace43-111">Cloud applications typically provide users with a service level agreement (SLA), which means that applications must be designed and implemented in a way that maximizes availability.</span></span>

|                            <span data-ttu-id="ace43-112">Modèle</span><span class="sxs-lookup"><span data-stu-id="ace43-112">Pattern</span></span>                             |                                                           <span data-ttu-id="ace43-113">Résumé</span><span class="sxs-lookup"><span data-stu-id="ace43-113">Summary</span></span>                                                            |
|----------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------|
| [<span data-ttu-id="ace43-114">Surveillance de point de terminaison d’intégrité</span><span class="sxs-lookup"><span data-stu-id="ace43-114">Health Endpoint Monitoring</span></span>](../health-endpoint-monitoring.md) | <span data-ttu-id="ace43-115">Implémentez des contrôles fonctionnels dans une application à laquelle des outils externes peuvent accéder par le biais de points de terminaison exposés à intervalles réguliers.</span><span class="sxs-lookup"><span data-stu-id="ace43-115">Implement functional checks in an application that external tools can access through exposed endpoints at regular intervals.</span></span> |
|  [<span data-ttu-id="ace43-116">Nivellement de la charge basé sur une file d’attente</span><span class="sxs-lookup"><span data-stu-id="ace43-116">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md)  | <span data-ttu-id="ace43-117">Utilisez une file d’attente qui agit comme mémoire tampon entre une tâche et un service qu’elle appelle, afin d’atténuer les surcharges intermittentes.</span><span class="sxs-lookup"><span data-stu-id="ace43-117">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span>  |
|                 [<span data-ttu-id="ace43-118">Limitation</span><span class="sxs-lookup"><span data-stu-id="ace43-118">Throttling</span></span>](../throttling.md)                 |   <span data-ttu-id="ace43-119">Contrôlez la consommation des ressources utilisées par une instance d’une application, un locataire ou un service entier.</span><span class="sxs-lookup"><span data-stu-id="ace43-119">Control the consumption of resources used by an instance of an application, an individual tenant, or an entire service.</span></span>    |
