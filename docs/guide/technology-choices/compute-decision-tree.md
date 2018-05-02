---
title: Arbre de décision des services de calcul Azure
description: Un organigramme pour sélectionner un service de calcul
author: MikeWasson
ms.date: 04/21/2018
ms.openlocfilehash: f5703f4906ca2ea6f825b383710eb4bd335f5043
ms.sourcegitcommit: d702b4d27e96e7a5a248dc4f2f0e25cf6e82c134
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/23/2018
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="2abd0-103">Arbre de décision des services de calcul Azure</span><span class="sxs-lookup"><span data-stu-id="2abd0-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="2abd0-104">Azure offre de nombreuses manières d’héberger votre code d’application.</span><span class="sxs-lookup"><span data-stu-id="2abd0-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="2abd0-105">Le terme *calcul* fait référence au modèle d’hébergement des ressources de calcul utilisées par votre application.</span><span class="sxs-lookup"><span data-stu-id="2abd0-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="2abd0-106">L’organigramme suivant vous aidera à choisir un service de calcul pour votre application.</span><span class="sxs-lookup"><span data-stu-id="2abd0-106">The following flowchart will help you to choose a compute service for your application.</span></span>
 
![](../images/compute-decision-tree.svg)

<span data-ttu-id="2abd0-107">Il vous guide au travers d’un ensemble de critères de décisions clé pour trouver une recommandation.</span><span class="sxs-lookup"><span data-stu-id="2abd0-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span> <span data-ttu-id="2abd0-108">Chaque application dispose de ces exigences propres. Vous devez donc vous baser sur la recommandation pour commencer.</span><span class="sxs-lookup"><span data-stu-id="2abd0-108">Every application has unique requirements, so you should treat the recommendation as a starting point.</span></span> <span data-ttu-id="2abd0-109">Réalisez ensuite une analyse plus détaillée, en considérant les aspects suivants :</span><span class="sxs-lookup"><span data-stu-id="2abd0-109">Then perform a more detailed analysis, looking at aspects such as:</span></span>
 
- <span data-ttu-id="2abd0-110">Ensemble des fonctionnalités</span><span class="sxs-lookup"><span data-stu-id="2abd0-110">Feature set</span></span>
- [<span data-ttu-id="2abd0-111">Limites du service</span><span class="sxs-lookup"><span data-stu-id="2abd0-111">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="2abd0-112">Coût</span><span class="sxs-lookup"><span data-stu-id="2abd0-112">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="2abd0-113">Contrat SLA</span><span class="sxs-lookup"><span data-stu-id="2abd0-113">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="2abd0-114">Disponibilité régionale</span><span class="sxs-lookup"><span data-stu-id="2abd0-114">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="2abd0-115">Écosystème de développement et compétences en équipe</span><span class="sxs-lookup"><span data-stu-id="2abd0-115">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="2abd0-116">Table de comparaison des calculs</span><span class="sxs-lookup"><span data-stu-id="2abd0-116">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="2abd0-117">Si votre application comprend plusieurs charges de travail, évaluez-les séparément.</span><span class="sxs-lookup"><span data-stu-id="2abd0-117">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="2abd0-118">Une solution complète peut incorporer deux services de calcul ou plus.</span><span class="sxs-lookup"><span data-stu-id="2abd0-118">A complete solution may incorporate two or more compute services.</span></span>

