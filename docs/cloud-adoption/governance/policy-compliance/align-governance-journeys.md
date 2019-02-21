---
title: "Framework d'adoption du cloud : Aligner les guides de conception sur la stratégie."
description: Comment aligner les guides de conception sur la stratégie ?
author: BrianBlanchard
ms.date: 01/04/2019
ms.openlocfilehash: 77a35597585e5f58967ea79d3c7b0fa17b6ab80e
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900734"
---
<!---
I've established policies. How to help developers adopt these policies?
Draft an architecture design guide.

[Aspirational statement] If you're using Azure, you can use one of ours as a starting point. The choose one of the following 6 as a starting point and mold it to fit your policies.
--->

<!-- markdownlint-disable MD026 -->

# <a name="how-do-you-align-design-guides-with-policy"></a><span data-ttu-id="ee543-103">Comment aligner les guides de conception sur la stratégie ?</span><span class="sxs-lookup"><span data-stu-id="ee543-103">How do you align design guides with policy?</span></span>

<span data-ttu-id="ee543-104">Après avoir [défini des stratégies cloud](define-policy.md) basées sur les [risques que vous avez identifiés](understanding-business-risk.md), vous devez générer des conseils exploitables, conformes à ces stratégies et auxquels le personnel informatique et les développeurs pourront se référer.</span><span class="sxs-lookup"><span data-stu-id="ee543-104">After you've [defined cloud policies](define-policy.md) based on your [identified risks](understanding-business-risk.md), you'll need to generate actionable guidance that aligns with these policies for your IT staff and developers to refer to.</span></span> <span data-ttu-id="ee543-105">La rédaction d'un guide de conception de gouvernance cloud vous permet de spécifier des choix spécifiques en matière de structure, de technologie et de processus conformément aux déclarations de stratégie que vous avez générées pour chacune des cinq disciplines de gouvernance.</span><span class="sxs-lookup"><span data-stu-id="ee543-105">Drafting a cloud governance design guide allows you to specify specific structural, technological, and process choices based on the policy statements you generated for each of the five governance disciplines.</span></span>

<span data-ttu-id="ee543-106">Pour chacun des composants d'infrastructure de base des déploiements cloud, un guide de conception de gouvernance cloud doit établir les choix d'architecture et les modèles de conception qui répondent le mieux à vos exigences en matière de stratégie.</span><span class="sxs-lookup"><span data-stu-id="ee543-106">A cloud governance design guide should establish the architecture choices and design patterns for each of the core infrastructure components of cloud deployments that best meet your policy requirements.</span></span> <span data-ttu-id="ee543-107">En outre, vous devez fournir une explication détaillée sur les technologies, outils et processus qui soutiendront chacune de ces décisions de conception.</span><span class="sxs-lookup"><span data-stu-id="ee543-107">Alongside these you should provide a high-level explanation of the technology, tools, and processes that will support each of these design decisions.</span></span>

<span data-ttu-id="ee543-108">Même si votre analyse des risques et vos déclarations de stratégie sont plus ou moins indépendantes de la plateforme cloud, votre guide de conception doit fournir des détails d'implémentation spécifiques à la plateforme pour votre personnel informatique et vos développeurs.</span><span class="sxs-lookup"><span data-stu-id="ee543-108">Although your risk analysis and policy statements may, to some degree, be cloud platform agnostic, your design guide should provide platform-specific implementation details that your IT and dev.</span></span> <span data-ttu-id="ee543-109">Concentrez-vous sur l'architecture, les outils et les fonctionnalités de la plateforme que vous avez choisie pour prendre des décisions de conception et fournir des conseils.</span><span class="sxs-lookup"><span data-stu-id="ee543-109">Focus on the architecture, tools, and features of your chosen platform when making design decision and providing guidance.</span></span>

<span data-ttu-id="ee543-110">Les guides de conception cloud doivent tenir compte de certains détails techniques associés à chaque composant d'infrastructure, mais il ne s'agit pas pour autant de spécifications ou de documents techniques exhaustifs.</span><span class="sxs-lookup"><span data-stu-id="ee543-110">While cloud design guides should take into account some of the technical details associated with each infrastructure component, they are not meant to be extensive technical documents or specifications.</span></span> <span data-ttu-id="ee543-111">Veillez à ce que vos guides traitent de toutes vos déclarations de stratégie et énoncent clairement les décisions de conception dans un format facile à comprendre et à consulter pour le personnel.</span><span class="sxs-lookup"><span data-stu-id="ee543-111">Make sure your guides address all of your policy statements and clearly state design decisions in a format easy for staff to understand and reference.</span></span>

<!-- markdownlint-enable MD033 -->

## <a name="using-the-actionable-governance-journeys"></a><span data-ttu-id="ee543-112">Utiliser les parcours de gouvernance exploitables</span><span class="sxs-lookup"><span data-stu-id="ee543-112">Using the actionable governance journeys</span></span>

<span data-ttu-id="ee543-113">Si vous envisagez d'utiliser la plateforme Azure pour votre adoption du cloud, le Framework d'adoption du cloud (CAF) fournit des [parcours de gouvernance](../journeys/overview.md) illustrant l'approche incrémentielle du modèle de gouvernance CAF.</span><span class="sxs-lookup"><span data-stu-id="ee543-113">If you're planning to use the Azure platform for your cloud adoption, the CAF provides [governance journeys](../journeys/overview.md) illustrating the incremental approach of the CAF governance model.</span></span> <span data-ttu-id="ee543-114">Ces parcours narratifs couvrent un ensemble de scénarios d'adoption courants, notamment sur les risques métier, les exigences de tolérance et les déclarations de stratégie ayant permis de créer un produit minimum viable (MVP).</span><span class="sxs-lookup"><span data-stu-id="ee543-114">These narrative journeys cover a range of common adoption scenarios, including the business risks, tolerance requirements, and policy statements that went into creating a governance minimum viable product (MVP).</span></span> <span data-ttu-id="ee543-115">Ces parcours représentent une synthèse du processus d'adoption du cloud tel qu'il est vécu par un client réel dans Azure.</span><span class="sxs-lookup"><span data-stu-id="ee543-115">These journeys represent a synthesis of real-world customer experience of the cloud adoption process in Azure.</span></span>

<span data-ttu-id="ee543-116">Bien que chaque adoption de cloud présente des objectifs, des priorités et des enjeux uniques, ces exemples constituent un bon modèle pour convertir votre stratégie en conseils.</span><span class="sxs-lookup"><span data-stu-id="ee543-116">While every cloud adoption has unique goals, priorities, and challenges, these samples should provide a good template for converting your policy into guidance.</span></span> <span data-ttu-id="ee543-117">Choisissez le scénario le plus proche de votre situation comme point de départ et adaptez-le à vos besoins spécifiques en termes de stratégie.</span><span class="sxs-lookup"><span data-stu-id="ee543-117">Pick the closest scenario to your situation as a starting point, and mold it to fit your specific policy needs.</span></span>

## <a name="next-steps"></a><span data-ttu-id="ee543-118">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="ee543-118">Next steps</span></span>

<span data-ttu-id="ee543-119">Une fois les conseils de conception en place, développez des processus d'adhésion à la stratégie pour garantir la mise en conformité avec celle-ci.</span><span class="sxs-lookup"><span data-stu-id="ee543-119">With design guidance in place, establish policy adherence processes to ensure policy compliance.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="ee543-120">Développer des processus d'adhésion à la stratégie</span><span class="sxs-lookup"><span data-stu-id="ee543-120">Policy adherence processes</span></span>](processes.md)