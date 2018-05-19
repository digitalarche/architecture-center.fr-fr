---
title: Arbre de décision des services de calcul Azure
description: Un organigramme pour sélectionner un service de calcul
author: MikeWasson
ms.date: 04/21/2018
ms.openlocfilehash: 3dcfbd156d4fced863a56bcc8bb74483aa665f9f
ms.sourcegitcommit: 7ced70ebc11aa0df0dc0104092d3cc6ad5c28bd6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/11/2018
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="57328-103">Arbre de décision des services de calcul Azure</span><span class="sxs-lookup"><span data-stu-id="57328-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="57328-104">Azure offre de nombreuses manières d’héberger votre code d’application.</span><span class="sxs-lookup"><span data-stu-id="57328-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="57328-105">Le terme *calcul* fait référence au modèle d’hébergement des ressources de calcul utilisées par votre application.</span><span class="sxs-lookup"><span data-stu-id="57328-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="57328-106">L’organigramme suivant vous aidera à choisir un service de calcul pour votre application.</span><span class="sxs-lookup"><span data-stu-id="57328-106">The following flowchart will help you to choose a compute service for your application.</span></span> <span data-ttu-id="57328-107">Il vous guide au travers d’un ensemble de critères de décisions clé pour trouver une recommandation.</span><span class="sxs-lookup"><span data-stu-id="57328-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span> 

<span data-ttu-id="57328-108">**Traitez cet organigramme comme un point de départ.**</span><span class="sxs-lookup"><span data-stu-id="57328-108">**Treat this flowchart as a starting point.**</span></span> <span data-ttu-id="57328-109">Chaque application dispose de ces exigences propres, utilisez donc la recommandation pour commencer.</span><span class="sxs-lookup"><span data-stu-id="57328-109">Every application has unique requirements, so use the recommendation as a starting point.</span></span> <span data-ttu-id="57328-110">Réalisez ensuite une évaluation plus détaillée, en considérant les aspects suivants :</span><span class="sxs-lookup"><span data-stu-id="57328-110">Then perform a more detailed evaluation, looking at aspects such as:</span></span>
 
- <span data-ttu-id="57328-111">Ensemble des fonctionnalités</span><span class="sxs-lookup"><span data-stu-id="57328-111">Feature set</span></span>
- [<span data-ttu-id="57328-112">Limites du service</span><span class="sxs-lookup"><span data-stu-id="57328-112">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="57328-113">Coût</span><span class="sxs-lookup"><span data-stu-id="57328-113">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="57328-114">Contrat SLA</span><span class="sxs-lookup"><span data-stu-id="57328-114">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="57328-115">Disponibilité régionale</span><span class="sxs-lookup"><span data-stu-id="57328-115">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="57328-116">Écosystème de développement et compétences en équipe</span><span class="sxs-lookup"><span data-stu-id="57328-116">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="57328-117">Table de comparaison des calculs</span><span class="sxs-lookup"><span data-stu-id="57328-117">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="57328-118">Si votre application comprend plusieurs charges de travail, évaluez-les séparément.</span><span class="sxs-lookup"><span data-stu-id="57328-118">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="57328-119">Une solution complète peut incorporer deux services de calcul ou plus.</span><span class="sxs-lookup"><span data-stu-id="57328-119">A complete solution may incorporate two or more compute services.</span></span>

![](../images/compute-decision-tree.svg)

