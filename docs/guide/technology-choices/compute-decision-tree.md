---
title: Arbre de décision des services de calcul Azure
description: Un organigramme pour sélectionner un service de calcul
author: MikeWasson
ms.date: 11/03/2018
ms.openlocfilehash: cb074272b8d00a71223d8c5755ef8cde3a3f2592
ms.sourcegitcommit: 225251ee2dd669432a9c9abe3aa8cd84d9e020b7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2018
ms.locfileid: "50981978"
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="4196d-103">Arbre de décision des services de calcul Azure</span><span class="sxs-lookup"><span data-stu-id="4196d-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="4196d-104">Azure offre de nombreuses manières d’héberger votre code d’application.</span><span class="sxs-lookup"><span data-stu-id="4196d-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="4196d-105">Le terme *calcul* fait référence au modèle d’hébergement des ressources de calcul utilisées par votre application.</span><span class="sxs-lookup"><span data-stu-id="4196d-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="4196d-106">L’organigramme suivant vous aidera à choisir un service de calcul pour votre application.</span><span class="sxs-lookup"><span data-stu-id="4196d-106">The following flowchart will help you to choose a compute service for your application.</span></span> <span data-ttu-id="4196d-107">Il vous guide au travers d’un ensemble de critères de décisions clé pour trouver une recommandation.</span><span class="sxs-lookup"><span data-stu-id="4196d-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span> 

<span data-ttu-id="4196d-108">**Traitez cet organigramme comme un point de départ.**</span><span class="sxs-lookup"><span data-stu-id="4196d-108">**Treat this flowchart as a starting point.**</span></span> <span data-ttu-id="4196d-109">Chaque application dispose de ces exigences propres, utilisez donc la recommandation pour commencer.</span><span class="sxs-lookup"><span data-stu-id="4196d-109">Every application has unique requirements, so use the recommendation as a starting point.</span></span> <span data-ttu-id="4196d-110">Réalisez ensuite une évaluation plus détaillée, en considérant les aspects suivants :</span><span class="sxs-lookup"><span data-stu-id="4196d-110">Then perform a more detailed evaluation, looking at aspects such as:</span></span>
 
- <span data-ttu-id="4196d-111">Ensemble des fonctionnalités</span><span class="sxs-lookup"><span data-stu-id="4196d-111">Feature set</span></span>
- [<span data-ttu-id="4196d-112">Limites du service</span><span class="sxs-lookup"><span data-stu-id="4196d-112">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="4196d-113">Coût</span><span class="sxs-lookup"><span data-stu-id="4196d-113">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="4196d-114">CONTRAT SLA</span><span class="sxs-lookup"><span data-stu-id="4196d-114">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="4196d-115">Disponibilité régionale</span><span class="sxs-lookup"><span data-stu-id="4196d-115">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="4196d-116">Écosystème de développement et compétences en équipe</span><span class="sxs-lookup"><span data-stu-id="4196d-116">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="4196d-117">Table de comparaison des calculs</span><span class="sxs-lookup"><span data-stu-id="4196d-117">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="4196d-118">Si votre application comprend plusieurs charges de travail, évaluez-les séparément.</span><span class="sxs-lookup"><span data-stu-id="4196d-118">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="4196d-119">Une solution complète peut incorporer deux services de calcul ou plus.</span><span class="sxs-lookup"><span data-stu-id="4196d-119">A complete solution may incorporate two or more compute services.</span></span>

<span data-ttu-id="4196d-120">Pour plus d’informations sur les options d’hébergement des conteneurs dans Azure, consultez https://azure.microsoft.com/overview/containers/.</span><span class="sxs-lookup"><span data-stu-id="4196d-120">For more information about your options for hosting containers in Azure, see https://azure.microsoft.com/overview/containers/.</span></span>

## <a name="flowchart"></a><span data-ttu-id="4196d-121">Organigramme</span><span class="sxs-lookup"><span data-stu-id="4196d-121">Flowchart</span></span>

![](../images/compute-decision-tree.svg)

## <a name="definitions"></a><span data-ttu-id="4196d-122">Définitions</span><span class="sxs-lookup"><span data-stu-id="4196d-122">Definitions</span></span>

- <span data-ttu-id="4196d-123">L’**opération « lift-and-shift »** est une stratégie visant à migrer une charge de travail vers le cloud sans reconcevoir l’application ou modifier le code.</span><span class="sxs-lookup"><span data-stu-id="4196d-123">**Lift and shift** is a strategy for migrating a workload to the cloud without redesigning the application or making code changes.</span></span> <span data-ttu-id="4196d-124">Elle est également appelée *ré-hébergement*.</span><span class="sxs-lookup"><span data-stu-id="4196d-124">Also called *rehosting*.</span></span> <span data-ttu-id="4196d-125">Pour plus d’informations, consultez [Centre de migration Azure](https://azure.microsoft.com/migration/).</span><span class="sxs-lookup"><span data-stu-id="4196d-125">For more information, see [Azure migration center](https://azure.microsoft.com/migration/).</span></span>

- <span data-ttu-id="4196d-126">**Optimisé pour le cloud** est une stratégie visant à migrer vers le cloud par la refactorisation d’une application, pour tirer parti des fonctionnalités cloud natives.</span><span class="sxs-lookup"><span data-stu-id="4196d-126">**Cloud optimized** is a strategy for migrating to the cloud by refactoring an application to take advantage of cloud-native features and capabilities.</span></span>

## <a name="next-steps"></a><span data-ttu-id="4196d-127">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="4196d-127">Next steps</span></span>

<span data-ttu-id="4196d-128">Pour connaître les autres critères à prendre en compte, consultez [Critères de sélection d’un service de calcul Azure](./compute-comparison.md).</span><span class="sxs-lookup"><span data-stu-id="4196d-128">For additional criteria to consider, see [Criteria for choosing an Azure compute service](./compute-comparison.md).</span></span>
