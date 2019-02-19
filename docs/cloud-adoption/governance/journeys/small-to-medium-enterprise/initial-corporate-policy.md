---
title: 'Framework d’adoption du cloud : Petite ou moyenne entreprise – Stratégie d’entreprise initiale sous-tendant la stratégie de gouvernance'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 2/11/2019
description: Petite ou moyenne entreprise – Stratégie d’entreprise initiale sous-tendant la stratégie de gouvernance
author: BrianBlanchard
ms.openlocfilehash: 4f49d0aae2c6ab5a5c8feba669cbbb904af2773b
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900782"
---
# <a name="small-to-medium-enterprise-initial-corporate-policy-behind-the-governance-strategy"></a><span data-ttu-id="1c21f-103">Petite ou moyenne entreprise Stratégie d’entreprise initiale sous-tendant la stratégie de gouvernance</span><span class="sxs-lookup"><span data-stu-id="1c21f-103">Small-to-medium enterprise: Initial corporate policy behind the governance strategy</span></span>

<span data-ttu-id="1c21f-104">La stratégie d’entreprise suivante définit une position de gouvernance initiale qui constitue le point de départ de ce parcours.</span><span class="sxs-lookup"><span data-stu-id="1c21f-104">The following corporate policy defines an initial governance position, which is the starting point for this journey.</span></span> <span data-ttu-id="1c21f-105">Cet article définit les risques à un stade précoce, les déclarations de stratégie initiales et les processus à mettre en œuvre dès le début pour appliquer des déclarations de stratégie.</span><span class="sxs-lookup"><span data-stu-id="1c21f-105">This article defines early-stage risks, initial policy statements, and early processes to enforce policy statements.</span></span>

> [!NOTE]
><span data-ttu-id="1c21f-106">La stratégie d’entreprise n’est pas un document technique, mais oriente de nombreuses décisions techniques.</span><span class="sxs-lookup"><span data-stu-id="1c21f-106">The corporate policy is not a technical document, but it drives many technical decisions.</span></span> <span data-ttu-id="1c21f-107">Le MVP de gouvernance décrit dans cette [vue d’ensemble](./overview.md) découle de cette stratégie.</span><span class="sxs-lookup"><span data-stu-id="1c21f-107">The governance MVP described in the [overview](./overview.md) ultimately derives from this policy.</span></span> <span data-ttu-id="1c21f-108">Avant d’implémenter un MVP de gouvernance, votre organisation doit développer une stratégie d’entreprise basée sur ses propres objectifs et risques.</span><span class="sxs-lookup"><span data-stu-id="1c21f-108">Before implementing a governance MVP, your organization should develop a corporate policy based on your own objectives and business risks.</span></span>

## <a name="cloud-governance-team"></a><span data-ttu-id="1c21f-109">Équipe de gouvernance cloud</span><span class="sxs-lookup"><span data-stu-id="1c21f-109">Cloud Governance team</span></span>

<span data-ttu-id="1c21f-110">Dans ce récit, l’équipe de gouvernance cloud est composée de deux administrateurs système qui ont reconnu le besoin de gouvernance.</span><span class="sxs-lookup"><span data-stu-id="1c21f-110">In this narrative, the Cloud Governance team is comprised of two systems administrators who have recognized the need for governance.</span></span> <span data-ttu-id="1c21f-111">Au cours des prochains mois, ils seront chargés d’assainir la gouvernance de la présence cloud de la société, ce qui leur vaudra le titre de Responsables du cloud.</span><span class="sxs-lookup"><span data-stu-id="1c21f-111">Over the next several months, they will inherit the job of cleaning up the governance of the company’s cloud presence, earning them the title of Cloud Custodians.</span></span> <span data-ttu-id="1c21f-112">Dans des évolutions ultérieures ce titre pourrait changer.</span><span class="sxs-lookup"><span data-stu-id="1c21f-112">In subsequent evolutions, this title will likely change.</span></span>

[!INCLUDE [business-risk](../../../../../includes/cloud-adoption/governance/business-risks.md)]

## <a name="tolerance-indicators"></a><span data-ttu-id="1c21f-113">Indicateurs de tolérance</span><span class="sxs-lookup"><span data-stu-id="1c21f-113">Tolerance indicators</span></span>

<span data-ttu-id="1c21f-114">La tolérance au risque actuelle est élevée et l’envie d’investir dans la gouvernance de cloud est faible.</span><span class="sxs-lookup"><span data-stu-id="1c21f-114">The current tolerance for risk is high and the appetite for investing in cloud governance is low.</span></span> <span data-ttu-id="1c21f-115">Par conséquent, les indicateurs de tolérance agissent comme un système d’avertissement précoce pour déclencher des investissements de temps et d’énergie.</span><span class="sxs-lookup"><span data-stu-id="1c21f-115">As such, the tolerance indicators act as an early warning system to trigger more investment of time and energy.</span></span> <span data-ttu-id="1c21f-116">Si les indicateurs suivants sont observés, vous devez faire évoluer la stratégie de gouvernance.</span><span class="sxs-lookup"><span data-stu-id="1c21f-116">If and when the following indicators are observed, you should evolve the governance strategy.</span></span>

- <span data-ttu-id="1c21f-117">Gestion des coûts : La mise à l’échelle du déploiement dépasse 100 ressources dans le cloud, ou les dépenses dépassent 1 000 USD par mois.</span><span class="sxs-lookup"><span data-stu-id="1c21f-117">Cost Management: The scale of deployment exceeds 100 assets to the cloud, or monthly spending exceeds $1,000 USD per month.</span></span>
- <span data-ttu-id="1c21f-118">Base de référence de la sécurité : Inclusion de données protégées dans des plans définis d’adoption du cloud.</span><span class="sxs-lookup"><span data-stu-id="1c21f-118">Security Baseline: Inclusion of protected data in defined cloud adoption plans.</span></span>
- <span data-ttu-id="1c21f-119">Cohérence des ressources : Inclusion de toutes les applications stratégiques dans des plans définis d’adoption du cloud.</span><span class="sxs-lookup"><span data-stu-id="1c21f-119">Resource Consistency: Inclusion of any mission-critical applications in defined cloud adoption plans.</span></span>

[!INCLUDE [policy-statements](../../../../../includes/cloud-adoption/governance/policy-statements.md)]

## <a name="next-steps"></a><span data-ttu-id="1c21f-120">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="1c21f-120">Next steps</span></span>

<span data-ttu-id="1c21f-121">Cette stratégie d’entreprise prépare l’équipe de gouvernance du cloud à implémenter le MVP de gouvernance, qui constituera la base de l’adoption.</span><span class="sxs-lookup"><span data-stu-id="1c21f-121">This corporate policy prepares the Cloud Governance team to implement the governance MVP, which will be the foundation for adoption.</span></span> <span data-ttu-id="1c21f-122">L’étape suivante consiste à implémenter ce MVP.</span><span class="sxs-lookup"><span data-stu-id="1c21f-122">The next step is to implement this MVP.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="1c21f-123">Bonne pratique, explication</span><span class="sxs-lookup"><span data-stu-id="1c21f-123">Best practice explained</span></span>](./best-practice-explained.md)