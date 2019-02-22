---
title: 'Framework d’adoption du cloud : Réseaux à définition logicielle – PaaS uniquement'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Discussion à propos du modèle « PaaS uniquement » pour la fonctionnalité de mise en réseau basée sur le cloud
author: rotycenh
ms.openlocfilehash: 2f3f82d781ddb6544721e82e7b7d795222a2f8ff
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900946"
---
# <a name="software-defined-networks-paas-only"></a><span data-ttu-id="70fc5-103">Réseaux à définition logicielle : PaaS uniquement</span><span class="sxs-lookup"><span data-stu-id="70fc5-103">Software Defined Networks: PaaS-only</span></span>

<span data-ttu-id="70fc5-104">Lorsque vous implémentez une ressource PaaS (platform as a service), le processus de déploiement crée automatiquement un réseau sous-jacent supposé, qui comprend un nombre limité de contrôles sur ce réseau. Ces contrôles incluent, entre autres, l’équilibrage de charge, le blocage des ports et les connexions à d’autres services PaaS.</span><span class="sxs-lookup"><span data-stu-id="70fc5-104">When you implement a platform as a service (PaaS) resource, the deployment process automatically creates an assumed underlying network with a limited number of controls over that network, including load balancing, port blocking, and connections to other PaaS services.</span></span>

<span data-ttu-id="70fc5-105">Dans Azure, différents types de ressources PaaS peuvent être [déployés dans](/azure/virtual-network/virtual-network-for-azure-services) ou [connectés à](/azure/virtual-network/virtual-network-service-endpoints-overview) un réseau virtuel. De cette manière, ces ressources peuvent s’intégrer à votre infrastructure existante de mise en réseau virtuelle.</span><span class="sxs-lookup"><span data-stu-id="70fc5-105">In Azure, several PaaS resource types can be [deployed into](/azure/virtual-network/virtual-network-for-azure-services) or [connected to](/azure/virtual-network/virtual-network-service-endpoints-overview) a virtual network, allowing these resources to integrate with your existing virtual networking infrastructure.</span></span> <span data-ttu-id="70fc5-106">La plupart du temps, une architecture de mise en réseau « PaaS uniquement », qui ne s’appuie que sur ces capacités de mise en réseau par défaut fournies par les ressources PaaS, est suffisante pour répondre aux exigences des charges de travail.</span><span class="sxs-lookup"><span data-stu-id="70fc5-106">However, in many cases a PaaS only networking architecture, relying only on these default networking capabilities natively provided by PaaS resources, is sufficient to meet workload requirements.</span></span>

<span data-ttu-id="70fc5-107">Si vous envisagez d’utiliser une architecture de mise en réseau « PaaS uniquement », vérifiez bien que les postulats requis s’alignent sur vos exigences.</span><span class="sxs-lookup"><span data-stu-id="70fc5-107">If you are considering a PaaS only networking architecture, be sure you validate that the required assumptions align with your requirements.</span></span>

## <a name="paas-only-assumptions"></a><span data-ttu-id="70fc5-108">Postulats concernant « PaaS uniquement »</span><span class="sxs-lookup"><span data-stu-id="70fc5-108">PaaS-only assumptions</span></span>

<span data-ttu-id="70fc5-109">Les postulats suivants sont admis lorsque l’on déploie une architecture de mise en réseau « PaaS uniquement » :</span><span class="sxs-lookup"><span data-stu-id="70fc5-109">Deploying a PaaS-only networking architecture assumes the following:</span></span>

- <span data-ttu-id="70fc5-110">L’application déployée est une application autonome OU dépendante d’autres ressources PaaS uniquement.</span><span class="sxs-lookup"><span data-stu-id="70fc5-110">The application being deployed is a standalone application OR is dependent on only other PaaS resources.</span></span>
- <span data-ttu-id="70fc5-111">L’équipe qui s’occupe des opérations informatiques peut mettre à jour les outils, les formations et les processus afin de prendre en charge la gestion, la configuration et le déploiement d’applications PaaS autonomes.</span><span class="sxs-lookup"><span data-stu-id="70fc5-111">Your IT operations teams can update their tools, training, and processes to support management, configuration, and deployment of standalone PaaS applications.</span></span>
- <span data-ttu-id="70fc5-112">L’application PaaS ne fait pas partie d’un effort de migration cloud plus large incluant des ressources IaaS.</span><span class="sxs-lookup"><span data-stu-id="70fc5-112">The PaaS application is not part of a broader cloud migration effort that will include IaaS resources.</span></span>

<span data-ttu-id="70fc5-113">Ces postulats sont des qualificateurs minimaux qui s’alignent sur le déploiement d’un réseau « PaaS uniquement ».</span><span class="sxs-lookup"><span data-stu-id="70fc5-113">These assumptions are minimum qualifiers aligned to deploying a PaaS-only network.</span></span> <span data-ttu-id="70fc5-114">Bien que cette approche soit conforme aux exigences d’un déploiement pour une seule application, votre équipe d’adoption du cloud doit réfléchir aux questions suivantes portant sur le long terme :</span><span class="sxs-lookup"><span data-stu-id="70fc5-114">While this approach may align with the requirements of a single application deployment, your Cloud Adoption Team should examine these long-term questions:</span></span>

- <span data-ttu-id="70fc5-115">L’échelle ou la portée de ce déploiement s’étendra-t-elle et demandera-t-elle d’avoir accès à des ressources non PaaS ?</span><span class="sxs-lookup"><span data-stu-id="70fc5-115">Will this deployment expand in scope or scale to require access to other non-PaaS resources?</span></span>
- <span data-ttu-id="70fc5-116">En plus de la solution actuelle, d’autres déploiements PaaS sont-ils prévus ?</span><span class="sxs-lookup"><span data-stu-id="70fc5-116">Are other PaaS deployments planned beyond the current solution?</span></span>
- <span data-ttu-id="70fc5-117">L’organisation a-t-elle planifié d’autres migrations cloud à l’avenir ?</span><span class="sxs-lookup"><span data-stu-id="70fc5-117">Does the organization have plans for other future cloud migrations?</span></span>

<span data-ttu-id="70fc5-118">Les réponses à ces questions ne doivent pas empêcher une équipe de choisir une solution « PaaS uniquement », mais doivent être prises en compte avant de prendre une décision finale.</span><span class="sxs-lookup"><span data-stu-id="70fc5-118">The answers to these questions would not preclude a team from choosing a PaaS only option but should be considered before making a final decision.</span></span>
