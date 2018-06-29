---
title: Arbre de décision des services de calcul Azure
description: Un organigramme pour sélectionner un service de calcul
author: MikeWasson
ms.date: 06/13/2018
ms.openlocfilehash: 689ec3f265e563273a75ad98268d03624a7b4536
ms.sourcegitcommit: ce2fa8ac2d310f7078317cade12f1b89db1ffe06
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/23/2018
ms.locfileid: "36338181"
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="4dee9-103">Arbre de décision des services de calcul Azure</span><span class="sxs-lookup"><span data-stu-id="4dee9-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="4dee9-104">Azure offre de nombreuses manières d’héberger votre code d’application.</span><span class="sxs-lookup"><span data-stu-id="4dee9-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="4dee9-105">Le terme *calcul* fait référence au modèle d’hébergement des ressources de calcul utilisées par votre application.</span><span class="sxs-lookup"><span data-stu-id="4dee9-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="4dee9-106">L’organigramme suivant vous aidera à choisir un service de calcul pour votre application.</span><span class="sxs-lookup"><span data-stu-id="4dee9-106">The following flowchart will help you to choose a compute service for your application.</span></span> <span data-ttu-id="4dee9-107">Il vous guide au travers d’un ensemble de critères de décisions clé pour trouver une recommandation.</span><span class="sxs-lookup"><span data-stu-id="4dee9-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span> 

<span data-ttu-id="4dee9-108">**Traitez cet organigramme comme un point de départ.**</span><span class="sxs-lookup"><span data-stu-id="4dee9-108">**Treat this flowchart as a starting point.**</span></span> <span data-ttu-id="4dee9-109">Chaque application dispose de ces exigences propres, utilisez donc la recommandation pour commencer.</span><span class="sxs-lookup"><span data-stu-id="4dee9-109">Every application has unique requirements, so use the recommendation as a starting point.</span></span> <span data-ttu-id="4dee9-110">Réalisez ensuite une évaluation plus détaillée, en considérant les aspects suivants :</span><span class="sxs-lookup"><span data-stu-id="4dee9-110">Then perform a more detailed evaluation, looking at aspects such as:</span></span>
 
- <span data-ttu-id="4dee9-111">Ensemble des fonctionnalités</span><span class="sxs-lookup"><span data-stu-id="4dee9-111">Feature set</span></span>
- [<span data-ttu-id="4dee9-112">Limites du service</span><span class="sxs-lookup"><span data-stu-id="4dee9-112">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="4dee9-113">Coût</span><span class="sxs-lookup"><span data-stu-id="4dee9-113">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="4dee9-114">Contrat SLA</span><span class="sxs-lookup"><span data-stu-id="4dee9-114">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="4dee9-115">Disponibilité régionale</span><span class="sxs-lookup"><span data-stu-id="4dee9-115">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="4dee9-116">Écosystème de développement et compétences en équipe</span><span class="sxs-lookup"><span data-stu-id="4dee9-116">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="4dee9-117">Table de comparaison des calculs</span><span class="sxs-lookup"><span data-stu-id="4dee9-117">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="4dee9-118">Si votre application comprend plusieurs charges de travail, évaluez-les séparément.</span><span class="sxs-lookup"><span data-stu-id="4dee9-118">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="4dee9-119">Une solution complète peut incorporer deux services de calcul ou plus.</span><span class="sxs-lookup"><span data-stu-id="4dee9-119">A complete solution may incorporate two or more compute services.</span></span>

<span data-ttu-id="4dee9-120">Pour plus d’informations sur les options d’hébergement des conteneurs dans Azure, consultez https://azure.microsoft.com/overview/containers/.</span><span class="sxs-lookup"><span data-stu-id="4dee9-120">For more information about your options for hosting containers in Azure, see https://azure.microsoft.com/overview/containers/.</span></span>

## <a name="flowchart"></a><span data-ttu-id="4dee9-121">Organigramme</span><span class="sxs-lookup"><span data-stu-id="4dee9-121">Flowchart</span></span>

![](../images/compute-decision-tree.svg)

## <a name="definitions"></a><span data-ttu-id="4dee9-122">Définitions</span><span class="sxs-lookup"><span data-stu-id="4dee9-122">Definitions</span></span>

- <span data-ttu-id="4dee9-123">**Greenfield** décrit un projet de logiciel complètement nouveau et créé à partir de zéro.</span><span class="sxs-lookup"><span data-stu-id="4dee9-123">**Greenfield** describes a software project that is completely new and built from scratch.</span></span> <span data-ttu-id="4dee9-124">Il n’inclut pas les codes hérités.</span><span class="sxs-lookup"><span data-stu-id="4dee9-124">It does not include legacy code.</span></span> 

- <span data-ttu-id="4dee9-125">**Brownfield** décrit un projet de logiciel qui s’appuie sur une application existante.</span><span class="sxs-lookup"><span data-stu-id="4dee9-125">**Brownfield** describes a software project that builds on an existing application.</span></span> <span data-ttu-id="4dee9-126">Il peut hériter d’infrastructures ou de codes hérités.</span><span class="sxs-lookup"><span data-stu-id="4dee9-126">It may inherit legacy code or frameworks.</span></span>

- <span data-ttu-id="4dee9-127">L’**opération « lift-and-shift »** est une stratégie visant à migrer une charge de travail vers le cloud sans reconcevoir l’application ou modifier le code.</span><span class="sxs-lookup"><span data-stu-id="4dee9-127">**Lift and shift** is a strategy for migrating a workload to the cloud without redesigning the application or making code changes.</span></span> <span data-ttu-id="4dee9-128">Elle est également appelée *ré-hébergement*.</span><span class="sxs-lookup"><span data-stu-id="4dee9-128">Also called *rehosting*.</span></span> <span data-ttu-id="4dee9-129">Pour plus d’informations, consultez [Centre de migration Azure](https://azure.microsoft.com/migration/).</span><span class="sxs-lookup"><span data-stu-id="4dee9-129">For more information, see [Azure migration center](https://azure.microsoft.com/migration/).</span></span>

- <span data-ttu-id="4dee9-130">**Optimisé pour le cloud** est une stratégie visant à migrer vers le cloud par la refactorisation d’une application, pour tirer parti des fonctionnalités cloud natives.</span><span class="sxs-lookup"><span data-stu-id="4dee9-130">**Cloud optimized** is a strategy for migrating to the cloud by refactoring an application to take advantage of cloud-native features and capabilities.</span></span>

## <a name="next-steps"></a><span data-ttu-id="4dee9-131">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="4dee9-131">Next steps</span></span>

<span data-ttu-id="4dee9-132">Pour connaître les autres critères à prendre en compte, consultez [Critères de sélection d’un service de calcul Azure](./compute-comparison.md).</span><span class="sxs-lookup"><span data-stu-id="4dee9-132">For additional criteria to consider, see [Criteria for choosing an Azure compute service](./compute-comparison.md).</span></span>
