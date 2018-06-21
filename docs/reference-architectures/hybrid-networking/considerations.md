---
title: Choisir une solution pour connecter un réseau local à Azure
description: Comparatif des architectures de référence pour connecter un réseau local à Azure.
author: telmosampaio
ms.date: 04/06/2017
ms.openlocfilehash: 274b9df1817632a7f3eaafa8bf02e965fdc3feea
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2017
ms.locfileid: "24541047"
---
# <a name="choose-a-solution-for-connecting-an-on-premises-network-to-azure"></a><span data-ttu-id="b3582-103">Choisir une solution pour connecter un réseau local à Azure</span><span class="sxs-lookup"><span data-stu-id="b3582-103">Choose a solution for connecting an on-premises network to Azure</span></span>

<span data-ttu-id="b3582-104">Cet article compare les options disponibles pour connecter un réseau local à un réseau virtuel (VNet) Azure.</span><span class="sxs-lookup"><span data-stu-id="b3582-104">This article compares options for connecting an on-premises network to an Azure Virtual Network (VNet).</span></span> <span data-ttu-id="b3582-105">Pour chaque option, nous fournissons une architecture de référence et une solution pouvant être déployée.</span><span class="sxs-lookup"><span data-stu-id="b3582-105">We provide a reference architecture and a deployable solution for each option.</span></span>

## <a name="vpn-connection"></a><span data-ttu-id="b3582-106">Connexion VPN</span><span class="sxs-lookup"><span data-stu-id="b3582-106">VPN connection</span></span>

<span data-ttu-id="b3582-107">Utilisez un réseau privé virtuel (VPN) pour connecter votre réseau local à un réseau virtuel Azure via un tunnel VPN IPSec.</span><span class="sxs-lookup"><span data-stu-id="b3582-107">Use a virtual private network (VPN) to connect your on-premises network with an Azure VNet through an IPSec VPN tunnel.</span></span>

<span data-ttu-id="b3582-108">Cette architecture est idéale pour les applications hybrides avec un trafic peu volumineux entre le matériel sur site et le cloud, ou dans le cas où vous êtes disposé à accepter une latence un peu plus élevée pour bénéficier de la flexibilité et de la puissance de la technologie cloud.</span><span class="sxs-lookup"><span data-stu-id="b3582-108">This architecture is suitable for hybrid applications where the traffic between on-premises hardware and the cloud is likely to be light, or you are willing to trade slightly extended latency for the flexibility and processing power of the cloud.</span></span>

<span data-ttu-id="b3582-109">**Avantages**</span><span class="sxs-lookup"><span data-stu-id="b3582-109">**Benefits**</span></span>

- <span data-ttu-id="b3582-110">Cette méthode est facile à configurer.</span><span class="sxs-lookup"><span data-stu-id="b3582-110">Simple to configure.</span></span>

<span data-ttu-id="b3582-111">**Défis**</span><span class="sxs-lookup"><span data-stu-id="b3582-111">**Challenges**</span></span>

- <span data-ttu-id="b3582-112">Vous avez besoin d’un appareil VPN local.</span><span class="sxs-lookup"><span data-stu-id="b3582-112">Requires an on-premises VPN device.</span></span>
- <span data-ttu-id="b3582-113">Bien que Microsoft garantit une disponibilité de 99,9 % pour chaque passerelle VPN, ce contrat SLA couvre uniquement la passerelle VPN, et pas votre connexion réseau à la passerelle.</span><span class="sxs-lookup"><span data-stu-id="b3582-113">Although Microsoft guarantees 99.9% availability for each VPN Gateway, this SLA only covers the VPN gateway, and not your network connection to the gateway.</span></span>
- <span data-ttu-id="b3582-114">Une connexion VPN via la passerelle VPN Azure prend actuellement en charge une bande passante maximale de 200 Mbits/s.</span><span class="sxs-lookup"><span data-stu-id="b3582-114">A VPN connection over Azure VPN Gateway currently supports a maximum of 200 Mbps bandwidth.</span></span> <span data-ttu-id="b3582-115">Vous devrez peut-être partitionner votre réseau virtuel Azure sur plusieurs connexions VPN si vous pensez dépasser ce débit.</span><span class="sxs-lookup"><span data-stu-id="b3582-115">You may need to partition your Azure virtual network across multiple VPN connections if you expect to exceed this throughput.</span></span>

<span data-ttu-id="b3582-116">**[En savoir plus...][vpn]**</span><span class="sxs-lookup"><span data-stu-id="b3582-116">**[Read more...][vpn]**</span></span>

## <a name="azure-expressroute-connection"></a><span data-ttu-id="b3582-117">Connexion Azure ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="b3582-117">Azure ExpressRoute connection</span></span>

<span data-ttu-id="b3582-118">Les connexions ExpressRoute utilisent une connexion privée et dédiée via un fournisseur de connectivité tiers.</span><span class="sxs-lookup"><span data-stu-id="b3582-118">ExpressRoute connections use a private, dedicated connection through a third-party connectivity provider.</span></span> <span data-ttu-id="b3582-119">La connexion privée étend votre réseau local à Azure.</span><span class="sxs-lookup"><span data-stu-id="b3582-119">The private connection extends your on-premises network into Azure.</span></span> 

<span data-ttu-id="b3582-120">Cette architecture est idéale pour les applications hybrides exécutant des charges de travail critiques et volumineuses qui nécessitent un niveau élevé d’évolutivité.</span><span class="sxs-lookup"><span data-stu-id="b3582-120">This architecture is suitable for hybrid applications running large-scale, mission-critical workloads that require a high degree of scalability.</span></span> 

<span data-ttu-id="b3582-121">**Avantages**</span><span class="sxs-lookup"><span data-stu-id="b3582-121">**Benefits**</span></span>

- <span data-ttu-id="b3582-122">La bande passante disponible est plus élevée, avec jusqu’à 10 Gbits/s selon le fournisseur de connectivité choisi.</span><span class="sxs-lookup"><span data-stu-id="b3582-122">Much higher bandwidth available; up to 10 Gbps depending on the connectivity provider.</span></span>
- <span data-ttu-id="b3582-123">Cette méthode prend en charge la mise à l’échelle dynamique de la bande passante afin de réduire les coûts pendant les périodes de faible demande.</span><span class="sxs-lookup"><span data-stu-id="b3582-123">Supports dynamic scaling of bandwidth to help reduce costs during periods of lower demand.</span></span> <span data-ttu-id="b3582-124">Toutefois, tous les fournisseurs de connectivité ne proposent pas cette option.</span><span class="sxs-lookup"><span data-stu-id="b3582-124">However, not all connectivity providers have this option.</span></span>
- <span data-ttu-id="b3582-125">En fonction du fournisseur de connectivité choisi, votre organisation peut bénéficier d’un accès direct aux clouds nationaux.</span><span class="sxs-lookup"><span data-stu-id="b3582-125">May allow your organization direct access to national clouds, depending on the connectivity provider.</span></span>
- <span data-ttu-id="b3582-126">Vous bénéficiez d’un contrat SLA garantissant une disponibilité de 99,9 % sur l’ensemble de la connexion.</span><span class="sxs-lookup"><span data-stu-id="b3582-126">99.9% availability SLA across the entire connection.</span></span>

<span data-ttu-id="b3582-127">**Défis**</span><span class="sxs-lookup"><span data-stu-id="b3582-127">**Challenges**</span></span>

- <span data-ttu-id="b3582-128">Cette méthode peut être difficile à configurer.</span><span class="sxs-lookup"><span data-stu-id="b3582-128">Can be complex to set up.</span></span> <span data-ttu-id="b3582-129">La création d’une connexion ExpressRoute nécessite l’intervention d’un fournisseur de connectivité tiers.</span><span class="sxs-lookup"><span data-stu-id="b3582-129">Creating an ExpressRoute connection requires working with a third-party connectivity provider.</span></span> <span data-ttu-id="b3582-130">Le fournisseur est responsable de l’approvisionnement de la connexion réseau.</span><span class="sxs-lookup"><span data-stu-id="b3582-130">The provider is responsible for provisioning the network connection.</span></span>
- <span data-ttu-id="b3582-131">Cette méthode requiert des routeurs locaux offrant une large bande passante.</span><span class="sxs-lookup"><span data-stu-id="b3582-131">Requires high-bandwidth routers on-premises.</span></span>

<span data-ttu-id="b3582-132">**[En savoir plus...][expressroute]**</span><span class="sxs-lookup"><span data-stu-id="b3582-132">**[Read more...][expressroute]**</span></span>

## <a name="expressroute-with-vpn-failover"></a><span data-ttu-id="b3582-133">ExpressRoute avec basculement VPN</span><span class="sxs-lookup"><span data-stu-id="b3582-133">ExpressRoute with VPN failover</span></span>

<span data-ttu-id="b3582-134">Cette option combine les deux méthodes précédentes : elle utilise ExpressRoute dans des conditions normales, mais bascule vers une connexion VPN en cas de perte de connectivité au niveau du circuit ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="b3582-134">This options combines the previous two, using ExpressRoute in normal conditions, but failing over to a VPN connection if there is a loss of connectivity in the ExpressRoute circuit.</span></span>

<span data-ttu-id="b3582-135">Cette architecture est idéale pour les applications hybrides qui nécessitent une bande passante ExpressRoute plus élevée, ainsi qu’une connectivité réseau hautement disponible.</span><span class="sxs-lookup"><span data-stu-id="b3582-135">This architecture is suitable for hybrid applications that need the higher bandwidth of ExpressRoute, and also require highly available network connectivity.</span></span> 

<span data-ttu-id="b3582-136">**Avantages**</span><span class="sxs-lookup"><span data-stu-id="b3582-136">**Benefits**</span></span>

- <span data-ttu-id="b3582-137">Haute disponibilité si le circuit ExpressRoute échoue, même lorsque la connexion de secours intervient sur un réseau de bande passante inférieure.</span><span class="sxs-lookup"><span data-stu-id="b3582-137">High availability if the ExpressRoute circuit fails, although the fallback connection is on a lower bandwidth network.</span></span>

<span data-ttu-id="b3582-138">**Défis**</span><span class="sxs-lookup"><span data-stu-id="b3582-138">**Challenges**</span></span>

- <span data-ttu-id="b3582-139">Cette méthode est difficile à configurer.</span><span class="sxs-lookup"><span data-stu-id="b3582-139">Complex to configure.</span></span> <span data-ttu-id="b3582-140">Vous devez configurer à la fois une connexion VPN et un circuit ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="b3582-140">You need to set up both a VPN connection and an ExpressRoute circuit.</span></span>
- <span data-ttu-id="b3582-141">Cette méthode requiert un matériel redondant (appliances VPN) et une connexion de passerelle VPN Azure redondante pour laquelle vous devrez payer des frais supplémentaires.</span><span class="sxs-lookup"><span data-stu-id="b3582-141">Requires redundant hardware (VPN appliances), and a redundant Azure VPN Gateway connection for which you pay charges.</span></span>

<span data-ttu-id="b3582-142">**[En savoir plus...][expressroute-vpn-failover]**</span><span class="sxs-lookup"><span data-stu-id="b3582-142">**[Read more...][expressroute-vpn-failover]**</span></span>

<!-- links -->
[expressroute]: ./expressroute.md
[expressroute-vpn-failover]: ./expressroute-vpn-failover.md
[vpn]: ./vpn.md