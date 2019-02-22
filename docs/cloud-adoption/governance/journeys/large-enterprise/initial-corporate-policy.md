---
title: 'Framework d’adoption du cloud : Grandes entreprises - Stratégie d’entreprise initiale sous-tendant la stratégie de gouvernance'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 2/11/2019
description: Grandes entreprises - Stratégie d’entreprise initiale sous-tendant la stratégie de gouvernance.
author: BrianBlanchard
ms.openlocfilehash: d3bc31f95229a82cbc2f6ac6e684a87107783c07
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901089"
---
# <a name="large-enterprise-initial-corporate-policy-behind-the-governance-strategy"></a><span data-ttu-id="6a91b-103">Grandes entreprises : Stratégie d’entreprise initiale sous-tendant la stratégie de gouvernance</span><span class="sxs-lookup"><span data-stu-id="6a91b-103">Large enterprise: Initial corporate policy behind the governance strategy</span></span>

<span data-ttu-id="6a91b-104">La stratégie d’entreprise suivante définit la position de gouvernance initiale qui constitue le point de départ de ce parcours.</span><span class="sxs-lookup"><span data-stu-id="6a91b-104">The following corporate policy defines the initial governance position, which is the starting point for this journey.</span></span> <span data-ttu-id="6a91b-105">Cet article définit les risques à un stade précoce, les déclarations de stratégie initiales et les processus à mettre en œuvre dès le début pour appliquer des déclarations de stratégie.</span><span class="sxs-lookup"><span data-stu-id="6a91b-105">This article defines early-stage risks, initial policy statements, and early processes to enforce policy statements.</span></span>

> [!NOTE]
><span data-ttu-id="6a91b-106">La stratégie d’entreprise n’est pas un document technique, mais oriente de nombreuses décisions techniques.</span><span class="sxs-lookup"><span data-stu-id="6a91b-106">The corporate policy is not a technical document, but it drives many technical decisions.</span></span> <span data-ttu-id="6a91b-107">Le MVP de gouvernance décrit dans cette [vue d’ensemble](./overview.md) découle de cette stratégie.</span><span class="sxs-lookup"><span data-stu-id="6a91b-107">The governance MVP described in the [overview](./overview.md) ultimately derives from this policy.</span></span> <span data-ttu-id="6a91b-108">Avant d’implémenter un MVP de gouvernance, votre organisation doit développer une stratégie d’entreprise basée sur ses propres objectifs et risques.</span><span class="sxs-lookup"><span data-stu-id="6a91b-108">Before implementing a governance MVP, your organization should develop a corporate policy based on your own objectives and business risks.</span></span>

## <a name="cloud-governance-team"></a><span data-ttu-id="6a91b-109">Équipe de gouvernance cloud</span><span class="sxs-lookup"><span data-stu-id="6a91b-109">Cloud Governance team</span></span>

<span data-ttu-id="6a91b-110">La directrice informatique a récemment rencontré l'équipe de gouvernance informatique pour comprendre l'évolution des stratégies liées aux informations d'identification personnelle notamment, et examiner l'impact d'une modification de ces stratégies.</span><span class="sxs-lookup"><span data-stu-id="6a91b-110">The CIO recently held a meeting with the IT Governance team to understand the history of the PII and mission-critical policies and review the impact of changing those policies.</span></span> <span data-ttu-id="6a91b-111">Elle a également évoqué le potentiel global du cloud pour le service informatique et l'entreprise.</span><span class="sxs-lookup"><span data-stu-id="6a91b-111">She also discussed the overall potential of the cloud for IT and the company.</span></span>

<span data-ttu-id="6a91b-112">Après la réunion, deux membres de l'équipe de gouvernance informatique ont demandé l'autorisation de participer aux efforts de planification sur le cloud.</span><span class="sxs-lookup"><span data-stu-id="6a91b-112">After the meeting, two members of the IT Governance team requested permission to research and support the cloud planning efforts.</span></span> <span data-ttu-id="6a91b-113">Conscient du besoin de gouvernance et voyant là l'occasion de limiter l'informatique fantôme, le directeur de la gouvernance informatique a appuyé cette idée.</span><span class="sxs-lookup"><span data-stu-id="6a91b-113">Recognizing the need for governance and an opportunity to limit shadow IT, the Director of IT Governance supported this idea.</span></span> <span data-ttu-id="6a91b-114">C'est ainsi qu'a vu le jour l'équipe de gouvernance cloud.</span><span class="sxs-lookup"><span data-stu-id="6a91b-114">With that, the Cloud Governance team was born.</span></span> <span data-ttu-id="6a91b-115">Au cours des prochains mois, ses membres corrigeront les nombreuses erreurs commises lors de l'exploration du cloud en termes de gouvernance.</span><span class="sxs-lookup"><span data-stu-id="6a91b-115">Over the next several months, they will inherit the cleanup of many mistakes made during exploration in the cloud from a governance perspective.</span></span> <span data-ttu-id="6a91b-116">Cela leur vaudra le moniker des gardiens du cloud.</span><span class="sxs-lookup"><span data-stu-id="6a91b-116">This will earn them the moniker of Cloud Custodians.</span></span> <span data-ttu-id="6a91b-117">Au cours des évolutions à venir, ce parcours démontrera la manière dont les rôles changent au fil du temps.</span><span class="sxs-lookup"><span data-stu-id="6a91b-117">In later evolutions, this journey will show how their roles change over time.</span></span>

[!INCLUDE [business-risk](../../../../../includes/cloud-adoption/governance/business-risks.md)]

## <a name="tolerance-indicators"></a><span data-ttu-id="6a91b-118">Indicateurs de tolérance</span><span class="sxs-lookup"><span data-stu-id="6a91b-118">Tolerance indicators</span></span>

<span data-ttu-id="6a91b-119">La tolérance au risque actuelle est élevée et l’envie d’investir dans la gouvernance de cloud est faible.</span><span class="sxs-lookup"><span data-stu-id="6a91b-119">The current risk tolerance is high and the appetite for investing in cloud governance is low.</span></span> <span data-ttu-id="6a91b-120">Par conséquent, les indicateurs de tolérance agissent comme un système d’avertissement précoce pour déclencher les investissements de temps et d’énergie.</span><span class="sxs-lookup"><span data-stu-id="6a91b-120">As such, the tolerance indicators act as an early warning system to trigger the investment of time and energy.</span></span> <span data-ttu-id="6a91b-121">Si les indicateurs suivants sont observés, il apparaît judicieux de faire évoluer la stratégie de gouvernance.</span><span class="sxs-lookup"><span data-stu-id="6a91b-121">If the following indicators are observed, it would be wise to evolve the governance strategy.</span></span>

- <span data-ttu-id="6a91b-122">Gestion des coûts : La mise à l’échelle du déploiement dépasse 1 000 ressources dans le cloud, ou les dépenses dépassent 10 000 USD par mois.</span><span class="sxs-lookup"><span data-stu-id="6a91b-122">Cost Management: Scale of deployment exceeds 1,000 assets to the cloud, or monthly spending exceeds $10,000 USD per month.</span></span>
- <span data-ttu-id="6a91b-123">Base de référence des identités : Inclusion d’applications avec exigences d’authentification multifacteur tierces ou héritées.</span><span class="sxs-lookup"><span data-stu-id="6a91b-123">Identity Baseline: Inclusion of applications with legacy or third-party multifactor authentication (MFA) requirements.</span></span>
- <span data-ttu-id="6a91b-124">Base de référence de la sécurité : Inclusion de données protégées dans des plans définis d’adoption du cloud.</span><span class="sxs-lookup"><span data-stu-id="6a91b-124">Security Baseline: Inclusion of protected data in defined cloud adoption plans.</span></span>
- <span data-ttu-id="6a91b-125">Cohérence des ressources : Inclusion de toutes les applications stratégiques dans des plans définis d’adoption du cloud.</span><span class="sxs-lookup"><span data-stu-id="6a91b-125">Resource Consistency: Inclusion of any mission-critical applications in defined cloud adoption plans.</span></span>

[!INCLUDE [policy-statements](../../../../../includes/cloud-adoption/governance/policy-statements.md)]

## <a name="next-steps"></a><span data-ttu-id="6a91b-126">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="6a91b-126">Next steps</span></span>

<span data-ttu-id="6a91b-127">Cette stratégie d’entreprise prépare l’équipe de gouvernance du cloud à implémenter le MVP de gouvernance, qui constituera la base de l’adoption.</span><span class="sxs-lookup"><span data-stu-id="6a91b-127">This corporate policy prepares the Cloud Governance team to implement the governance MVP, which will be the foundation for adoption.</span></span> <span data-ttu-id="6a91b-128">L’étape suivante consiste à implémenter ce MVP.</span><span class="sxs-lookup"><span data-stu-id="6a91b-128">The next step is to implement this MVP.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="6a91b-129">Bonne pratique, explication</span><span class="sxs-lookup"><span data-stu-id="6a91b-129">Best practice explained</span></span>](./best-practice-explained.md)