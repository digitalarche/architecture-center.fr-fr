---
title: 'Framework d’adoption du cloud : Parcours de gouvernance pour les petites et moyennes entreprises'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Parcours de gouvernance pour les petites et moyennes entreprises
author: BrianBlanchard
ms.openlocfilehash: a3e078845038a12977e7be5affbf22708411069f
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900938"
---
# <a name="small-to-medium-enterprise-governance-journey"></a><span data-ttu-id="80749-103">Parcours de gouvernance pour les petites et moyennes entreprises</span><span class="sxs-lookup"><span data-stu-id="80749-103">Small-to-medium enterprise governance journey</span></span>

## <a name="best-practice-overview"></a><span data-ttu-id="80749-104">Présentation des meilleures pratiques</span><span class="sxs-lookup"><span data-stu-id="80749-104">Best practice overview</span></span>

<span data-ttu-id="80749-105">Ce parcours de gouvernance suit les expériences d’une entreprise fictive à différents stades de maturité.</span><span class="sxs-lookup"><span data-stu-id="80749-105">This governance journey follows the experiences of a fictional company through various stages of governance maturity.</span></span> <span data-ttu-id="80749-106">Il est basé sur le parcours de clients réels.</span><span class="sxs-lookup"><span data-stu-id="80749-106">It is based on real customer journeys.</span></span> <span data-ttu-id="80749-107">Les meilleures pratiques recommandées se fondent sur les contraintes et les besoins de l’entreprise fictive.</span><span class="sxs-lookup"><span data-stu-id="80749-107">The suggested best practices are based on the constraints and needs of the fictional company.</span></span>

<span data-ttu-id="80749-108">Comme point de départ rapide, cette présentation définit un produit minimum viable (MVP) pour la gouvernance, basé sur les meilleures pratiques.</span><span class="sxs-lookup"><span data-stu-id="80749-108">As a quick starting point, this overview defines a minimum viable product (MVP) for governance based on best practices.</span></span> <span data-ttu-id="80749-109">Elle fournit également des liens vers des évolutions de gouvernance, qui ajoutent des meilleures pratiques à mesure que de nouveaux risques métier et techniques émergent.</span><span class="sxs-lookup"><span data-stu-id="80749-109">It also provides links to some governance evolutions that add further best practices as new business or technical risks emerge.</span></span>

> [!WARNING]
> <span data-ttu-id="80749-110">Ce MVP est un point de départ de base, qui se fonde sur un ensemble de postulats.</span><span class="sxs-lookup"><span data-stu-id="80749-110">This MVP is a baseline starting point, based on a set of assumptions.</span></span> <span data-ttu-id="80749-111">Même cet ensemble minimal de meilleures pratiques se fonde sur des stratégies d’entreprise, qui sont axées sur des risques métier et des tolérances aux risques uniques.</span><span class="sxs-lookup"><span data-stu-id="80749-111">Even this minimal set of best practices is based on corporate policies driven by unique business risks and risk tolerances.</span></span> <span data-ttu-id="80749-112">Pour déterminer si ces postulats s’appliquent à vous, lisez la [plus longue description](./narrative.md) qui suit cet article.</span><span class="sxs-lookup"><span data-stu-id="80749-112">To see if these assumptions apply to you, read the [longer narrative](./narrative.md) that follows this article.</span></span>

## <a name="governance-best-practice"></a><span data-ttu-id="80749-113">Meilleures pratiques de gouvernance</span><span class="sxs-lookup"><span data-stu-id="80749-113">Governance best practice</span></span>

<span data-ttu-id="80749-114">Ces meilleures pratiques servent de bases sur lesquelles une organisation peut s’appuyer pour ajouter de manière cohérente et rapide des protections de gouvernance à plusieurs abonnements Azure.</span><span class="sxs-lookup"><span data-stu-id="80749-114">This best practice serves as a foundation that an organization can use to quickly and consistently add governance guardrails across multiple Azure subscriptions.</span></span>

### <a name="resource-organization"></a><span data-ttu-id="80749-115">Organisation des ressources</span><span class="sxs-lookup"><span data-stu-id="80749-115">Resource organization</span></span>

<span data-ttu-id="80749-116">Le schéma suivant montre la hiérarchie MVP de gouvernance pour organiser les ressources.</span><span class="sxs-lookup"><span data-stu-id="80749-116">The following diagram shows the governance MVP hierarchy for organizing resources.</span></span>

![Schéma d’organisation des ressources](../../../_images/governance/resource-organization.png)

<span data-ttu-id="80749-118">Chaque application doit être déployée dans la zone appropriée du groupe d’administration, de l’abonnement et de la hiérarchie des groupes de ressources.</span><span class="sxs-lookup"><span data-stu-id="80749-118">Every application should be deployed in the proper area of the management group, subscription, and resource group hierarchy.</span></span> <span data-ttu-id="80749-119">Lors de la planification du déploiement, l’équipe de gouvernance cloud crée les nœuds nécessaires dans la hiérarchie pour donner aux équipes d’adoption du cloud les moyens d’agir.</span><span class="sxs-lookup"><span data-stu-id="80749-119">During deployment planning, the Cloud Governance team will create the necessary nodes in the hierarchy to empower the cloud adoption teams.</span></span>  

1. <span data-ttu-id="80749-120">Un groupe d’administration pour chaque type d’environnement (par exemple, Production, Développement et Test).</span><span class="sxs-lookup"><span data-stu-id="80749-120">A management group for each type of environment (such as Production, Development, and Test).</span></span>
2. <span data-ttu-id="80749-121">Un abonnement pour chaque « catégorisation d’application ».</span><span class="sxs-lookup"><span data-stu-id="80749-121">A subscription for each "application categorization".</span></span>
3. <span data-ttu-id="80749-122">Un groupe de ressources séparé pour chaque application.</span><span class="sxs-lookup"><span data-stu-id="80749-122">A separate resource group for each application.</span></span>
4. <span data-ttu-id="80749-123">Une nomenclature cohérente doit être appliquée à chaque niveau de cette hiérarchie de regroupement.</span><span class="sxs-lookup"><span data-stu-id="80749-123">Consistent nomenclature should be applied at each level of this grouping hierarchy.</span></span>

<span data-ttu-id="80749-124">Voici un exemple de modèle utilisé :</span><span class="sxs-lookup"><span data-stu-id="80749-124">Here is an example of this pattern in use:</span></span>

![Exemple d’organisation des ressources pour une entreprise de taille intermédiaire](../../../_images/governance/mid-market-resource-organization.png)

<span data-ttu-id="80749-126">Ces modèles laissent de l’espace pour la croissance sans compliquer la hiérarchie inutilement.</span><span class="sxs-lookup"><span data-stu-id="80749-126">These patterns provide room for growth without complicating the hierarchy unnecessarily.</span></span>

[!INCLUDE [governance-of-resources](../../../../../includes/cloud-adoption/governance/governance-of-resources.md)]

## <a name="governance-evolutions"></a><span data-ttu-id="80749-127">Évolutions de la gouvernance</span><span class="sxs-lookup"><span data-stu-id="80749-127">Governance evolutions</span></span>

<span data-ttu-id="80749-128">Une fois que ce MVP a été déployé, des couches supplémentaires de gouvernance peuvent être rapidement intégrées à l’environnement.</span><span class="sxs-lookup"><span data-stu-id="80749-128">Once this MVP has been deployed, additional layers of governance can be quickly incorporated into the environment.</span></span> <span data-ttu-id="80749-129">Voici quelques méthodes permettant de faire évoluer le MVP afin de répondre aux besoins spécifiques de l’entreprise :</span><span class="sxs-lookup"><span data-stu-id="80749-129">Here are some ways to evolve the MVP to meet specific business needs:</span></span>

- [<span data-ttu-id="80749-130">Base de référence de sécurité pour les données protégées</span><span class="sxs-lookup"><span data-stu-id="80749-130">Security Baseline for protected data</span></span>](./security-baseline-evolution.md)
- [<span data-ttu-id="80749-131">Configurations des ressources pour les applications critiques</span><span class="sxs-lookup"><span data-stu-id="80749-131">Resource configurations for mission-critical applications</span></span>](./resource-consistency-evolution.md)
- [<span data-ttu-id="80749-132">Contrôles pour la gestion des coûts</span><span class="sxs-lookup"><span data-stu-id="80749-132">Controls for Cost Management</span></span>](./cost-management-evolution.md)
- [<span data-ttu-id="80749-133">Contrôles pour l’évolution multi-cloud</span><span class="sxs-lookup"><span data-stu-id="80749-133">Controls for multi-cloud evolution</span></span>](./multi-cloud-evolution.md)

<!-- markdownlint-disable MD026 -->

## <a name="what-does-this-best-practice-do"></a><span data-ttu-id="80749-134">Concrètement, que font les meilleures pratiques ?</span><span class="sxs-lookup"><span data-stu-id="80749-134">What does this best practice do?</span></span>

<span data-ttu-id="80749-135">Dans le MVP, les pratiques et les outils venant de la discipline [Accélération du déploiement](../../deployment-acceleration/overview.md) ont été conçus pour appliquer rapidement la stratégie d’entreprise.</span><span class="sxs-lookup"><span data-stu-id="80749-135">In the MVP, practices and tools from the [Deployment Acceleration](../../deployment-acceleration/overview.md) discipline are established to quickly apply corporate policy.</span></span> <span data-ttu-id="80749-136">Plus précisément, le MVP utilise Azure Blueprints, Azure Policy et les groupes d’administration Azure pour appliquer quelques stratégies d’entreprise basiques, comme le définit le scénario pour cette entreprise fictive.</span><span class="sxs-lookup"><span data-stu-id="80749-136">In particular, the MVP uses Azure Blueprints, Azure Policy, and Azure management groups to apply a few basic corporate policies, as defined in the narrative for this fictional company.</span></span> <span data-ttu-id="80749-137">Ces stratégies d’entreprise sont appliquées à l’aide de modèles Resource Manager et de stratégies Azure afin d’établir une petite base de référence pour l’identité et la sécurité.</span><span class="sxs-lookup"><span data-stu-id="80749-137">Those corporate policies are applied using Resource Manager templates and Azure policies to establish a very small baseline for identity and security.</span></span>

![Exemple de MVP de gouvernance incrémentielle](../../../_images/governance/governance-mvp.png)

## <a name="evolving-the-best-practice"></a><span data-ttu-id="80749-139">Évolution des meilleures pratiques</span><span class="sxs-lookup"><span data-stu-id="80749-139">Evolving the best practice</span></span>

<span data-ttu-id="80749-140">Au fil du temps, ce MVP de gouvernance servira à faire évoluer les pratiques de gouvernance.</span><span class="sxs-lookup"><span data-stu-id="80749-140">Over time, this governance MVP will be used to evolve the governance practices.</span></span> <span data-ttu-id="80749-141">À mesure que le processus d’adoption avance, les risques métier augmentent.</span><span class="sxs-lookup"><span data-stu-id="80749-141">As adoption advances, business risk grows.</span></span> <span data-ttu-id="80749-142">Différentes disciplines au sein du modèle de framework d’adoption du cloud évolueront pour atténuer ces risques.</span><span class="sxs-lookup"><span data-stu-id="80749-142">Various disciplines within the CAF governance model will evolve to mitigate those risks.</span></span> <span data-ttu-id="80749-143">D’autres articles de cette série traitent de l’évolution de la stratégie d’entreprise et de son impact sur l’entreprise fictive.</span><span class="sxs-lookup"><span data-stu-id="80749-143">Later articles in this series discuss the evolution of corporate policy affecting the fictional company.</span></span> <span data-ttu-id="80749-144">Ces évolutions se produisent dans trois disciplines :</span><span class="sxs-lookup"><span data-stu-id="80749-144">These evolutions happen across three disciplines:</span></span>

- <span data-ttu-id="80749-145">La gestion des coûts, à mesure que l’adoption avance.</span><span class="sxs-lookup"><span data-stu-id="80749-145">Cost Management, as adoption scales.</span></span>
- <span data-ttu-id="80749-146">La base de référence de la sécurité, à mesure que les données protégées sont déployées.</span><span class="sxs-lookup"><span data-stu-id="80749-146">Security Baseline, as protected data is deployed.</span></span>
- <span data-ttu-id="80749-147">La cohérence des ressources, à mesure que les opérations informatiques commencent à gérer les charges de travail critiques.</span><span class="sxs-lookup"><span data-stu-id="80749-147">Resource Consistency, as IT Operations begins supporting mission-critical workloads.</span></span>

![Exemple de MVP de gouvernance incrémentielle](../../../_images/governance/governance-evolution.png)

## <a name="next-steps"></a><span data-ttu-id="80749-149">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="80749-149">Next steps</span></span>

<span data-ttu-id="80749-150">Maintenant que vous connaissez le MVP de gouvernance et que vous avez une idée des évolutions de gouvernance à suivre, lisez le scénario correspondant pour en savoir plus.</span><span class="sxs-lookup"><span data-stu-id="80749-150">Now that you’re familiar with the governance MVP and have an idea of the governance evolutions to follow, read the supporting narrative for additional context.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="80749-151">Lire le scénario correspondant</span><span class="sxs-lookup"><span data-stu-id="80749-151">Read the supporting narrative</span></span>](./narrative.md)
