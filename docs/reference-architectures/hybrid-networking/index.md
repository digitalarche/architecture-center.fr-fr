---
title: Choisir une solution pour connecter un réseau local à Azure
description: Comparatif des architectures de référence pour connecter un réseau local à Azure.
author: telmosampaio
ms.date: 07/02/2018
ms.openlocfilehash: 0cc07d3b7d45accf9f99ce32914b0ef065d62f32
ms.sourcegitcommit: 776b8c1efc662d42273a33de3b82ec69e3cd80c5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/12/2018
ms.locfileid: "38987476"
---
# <a name="connect-an-on-premises-network-to-azure"></a><span data-ttu-id="b1efd-103">Connecter un réseau local à Azure</span><span class="sxs-lookup"><span data-stu-id="b1efd-103">Connect an on-premises network to Azure</span></span>

<span data-ttu-id="b1efd-104">Cet article compare les options disponibles pour connecter un réseau local à un réseau virtuel (VNet) Azure.</span><span class="sxs-lookup"><span data-stu-id="b1efd-104">This article compares options for connecting an on-premises network to an Azure Virtual Network (VNet).</span></span> <span data-ttu-id="b1efd-105">Pour chaque option, une architecture de référence plus détaillée est disponible.</span><span class="sxs-lookup"><span data-stu-id="b1efd-105">For each option, a more detailed reference architecture is available.</span></span>

## <a name="vpn-connection"></a><span data-ttu-id="b1efd-106">Connexion VPN</span><span class="sxs-lookup"><span data-stu-id="b1efd-106">VPN connection</span></span>

<span data-ttu-id="b1efd-107">Une [passerelle VPN](/azure/vpn-gateway/vpn-gateway-about-vpngateways) est un type de passerelle de réseau virtuel qui envoie le trafic chiffré entre un réseau virtuel Azure et un emplacement local.</span><span class="sxs-lookup"><span data-stu-id="b1efd-107">A [VPN gateway](/azure/vpn-gateway/vpn-gateway-about-vpngateways) is a type of virtual network gateway that sends encrypted traffic between an Azure virtual network and an on-premises location.</span></span> <span data-ttu-id="b1efd-108">Le trafic chiffré est envoyé via le réseau Internet public.</span><span class="sxs-lookup"><span data-stu-id="b1efd-108">The encrypted traffic goes over the public Internet.</span></span>

<span data-ttu-id="b1efd-109">Cette architecture est idéale pour les applications hybrides avec un trafic peu volumineux entre le matériel sur site et le cloud, ou dans le cas où vous êtes disposé à accepter une latence un peu plus élevée pour bénéficier de la flexibilité et de la puissance de la technologie cloud.</span><span class="sxs-lookup"><span data-stu-id="b1efd-109">This architecture is suitable for hybrid applications where the traffic between on-premises hardware and the cloud is likely to be light, or you are willing to trade slightly extended latency for the flexibility and processing power of the cloud.</span></span>

<span data-ttu-id="b1efd-110">**Avantages**</span><span class="sxs-lookup"><span data-stu-id="b1efd-110">**Benefits**</span></span>

- <span data-ttu-id="b1efd-111">Cette méthode est facile à configurer.</span><span class="sxs-lookup"><span data-stu-id="b1efd-111">Simple to configure.</span></span>

<span data-ttu-id="b1efd-112">**Défis**</span><span class="sxs-lookup"><span data-stu-id="b1efd-112">**Challenges**</span></span>

- <span data-ttu-id="b1efd-113">Vous avez besoin d’un appareil VPN local.</span><span class="sxs-lookup"><span data-stu-id="b1efd-113">Requires an on-premises VPN device.</span></span>
- <span data-ttu-id="b1efd-114">Même si Microsoft garantit une disponibilité de 99,9 % pour chaque passerelle VPN, ce contrat [SLA](https://azure.microsoft.com/support/legal/sla/vpn-gateway/) couvre uniquement la passerelle VPN, et non votre connexion réseau à la passerelle.</span><span class="sxs-lookup"><span data-stu-id="b1efd-114">Although Microsoft guarantees 99.9% availability for each VPN Gateway, this [SLA](https://azure.microsoft.com/support/legal/sla/vpn-gateway/) only covers the VPN gateway, and not your network connection to the gateway.</span></span>
- <span data-ttu-id="b1efd-115">Une connexion VPN via la passerelle VPN Azure prend actuellement en charge une bande passante maximale de 200 Mbits/s.</span><span class="sxs-lookup"><span data-stu-id="b1efd-115">A VPN connection over Azure VPN Gateway currently supports a maximum of 200 Mbps bandwidth.</span></span> <span data-ttu-id="b1efd-116">Vous devrez peut-être partitionner votre réseau virtuel Azure sur plusieurs connexions VPN si vous pensez dépasser ce débit.</span><span class="sxs-lookup"><span data-stu-id="b1efd-116">You may need to partition your Azure virtual network across multiple VPN connections if you expect to exceed this throughput.</span></span>

<span data-ttu-id="b1efd-117">**Architecture de référence**</span><span class="sxs-lookup"><span data-stu-id="b1efd-117">**Reference architecture**</span></span>

- [<span data-ttu-id="b1efd-118">Réseau hybride avec une passerelle VPN</span><span class="sxs-lookup"><span data-stu-id="b1efd-118">Hybrid network with VPN gateway</span></span>](./vpn.md)

## <a name="azure-expressroute-connection"></a><span data-ttu-id="b1efd-119">Connexion Azure ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="b1efd-119">Azure ExpressRoute connection</span></span>

<span data-ttu-id="b1efd-120">Les connexions [ExpressRoute](/azure/expressroute/) utilisent une connexion privée et dédiée via un fournisseur de connectivité tiers.</span><span class="sxs-lookup"><span data-stu-id="b1efd-120">[ExpressRoute](/azure/expressroute/) connections use a private, dedicated connection through a third-party connectivity provider.</span></span> <span data-ttu-id="b1efd-121">La connexion privée étend votre réseau local à Azure.</span><span class="sxs-lookup"><span data-stu-id="b1efd-121">The private connection extends your on-premises network into Azure.</span></span> 

<span data-ttu-id="b1efd-122">Cette architecture est idéale pour les applications hybrides exécutant des charges de travail critiques et volumineuses qui nécessitent un niveau élevé d’évolutivité.</span><span class="sxs-lookup"><span data-stu-id="b1efd-122">This architecture is suitable for hybrid applications running large-scale, mission-critical workloads that require a high degree of scalability.</span></span> 

<span data-ttu-id="b1efd-123">**Avantages**</span><span class="sxs-lookup"><span data-stu-id="b1efd-123">**Benefits**</span></span>

- <span data-ttu-id="b1efd-124">La bande passante disponible est plus élevée, avec jusqu’à 10 Gbits/s selon le fournisseur de connectivité choisi.</span><span class="sxs-lookup"><span data-stu-id="b1efd-124">Much higher bandwidth available; up to 10 Gbps depending on the connectivity provider.</span></span>
- <span data-ttu-id="b1efd-125">Cette méthode prend en charge la mise à l’échelle dynamique de la bande passante afin de réduire les coûts pendant les périodes de faible demande.</span><span class="sxs-lookup"><span data-stu-id="b1efd-125">Supports dynamic scaling of bandwidth to help reduce costs during periods of lower demand.</span></span> <span data-ttu-id="b1efd-126">Toutefois, tous les fournisseurs de connectivité ne proposent pas cette option.</span><span class="sxs-lookup"><span data-stu-id="b1efd-126">However, not all connectivity providers have this option.</span></span>
- <span data-ttu-id="b1efd-127">En fonction du fournisseur de connectivité choisi, votre organisation peut bénéficier d’un accès direct aux clouds nationaux.</span><span class="sxs-lookup"><span data-stu-id="b1efd-127">May allow your organization direct access to national clouds, depending on the connectivity provider.</span></span>
- <span data-ttu-id="b1efd-128">Vous bénéficiez d’un contrat SLA garantissant une disponibilité de 99,9 % sur l’ensemble de la connexion.</span><span class="sxs-lookup"><span data-stu-id="b1efd-128">99.9% availability SLA across the entire connection.</span></span>

<span data-ttu-id="b1efd-129">**Défis**</span><span class="sxs-lookup"><span data-stu-id="b1efd-129">**Challenges**</span></span>

- <span data-ttu-id="b1efd-130">Cette méthode peut être difficile à configurer.</span><span class="sxs-lookup"><span data-stu-id="b1efd-130">Can be complex to set up.</span></span> <span data-ttu-id="b1efd-131">La création d’une connexion ExpressRoute nécessite l’intervention d’un fournisseur de connectivité tiers.</span><span class="sxs-lookup"><span data-stu-id="b1efd-131">Creating an ExpressRoute connection requires working with a third-party connectivity provider.</span></span> <span data-ttu-id="b1efd-132">Le fournisseur est responsable de l’approvisionnement de la connexion réseau.</span><span class="sxs-lookup"><span data-stu-id="b1efd-132">The provider is responsible for provisioning the network connection.</span></span>
- <span data-ttu-id="b1efd-133">Cette méthode requiert des routeurs locaux offrant une large bande passante.</span><span class="sxs-lookup"><span data-stu-id="b1efd-133">Requires high-bandwidth routers on-premises.</span></span>

<span data-ttu-id="b1efd-134">**Architecture de référence**</span><span class="sxs-lookup"><span data-stu-id="b1efd-134">**Reference architecture**</span></span>

- [<span data-ttu-id="b1efd-135">Réseau hybride avec ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="b1efd-135">Hybrid network with ExpressRoute</span></span>](./expressroute.md)

## <a name="expressroute-with-vpn-failover"></a><span data-ttu-id="b1efd-136">ExpressRoute avec basculement VPN</span><span class="sxs-lookup"><span data-stu-id="b1efd-136">ExpressRoute with VPN failover</span></span>

<span data-ttu-id="b1efd-137">Cette option combine les deux méthodes précédentes : elle utilise ExpressRoute dans des conditions normales, mais bascule vers une connexion VPN en cas de perte de connectivité au niveau du circuit ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="b1efd-137">This options combines the previous two, using ExpressRoute in normal conditions, but failing over to a VPN connection if there is a loss of connectivity in the ExpressRoute circuit.</span></span>

<span data-ttu-id="b1efd-138">Cette architecture est idéale pour les applications hybrides qui nécessitent une bande passante ExpressRoute plus élevée, ainsi qu’une connectivité réseau hautement disponible.</span><span class="sxs-lookup"><span data-stu-id="b1efd-138">This architecture is suitable for hybrid applications that need the higher bandwidth of ExpressRoute, and also require highly available network connectivity.</span></span> 

<span data-ttu-id="b1efd-139">**Avantages**</span><span class="sxs-lookup"><span data-stu-id="b1efd-139">**Benefits**</span></span>

- <span data-ttu-id="b1efd-140">Haute disponibilité si le circuit ExpressRoute échoue, même lorsque la connexion de secours intervient sur un réseau de bande passante inférieure.</span><span class="sxs-lookup"><span data-stu-id="b1efd-140">High availability if the ExpressRoute circuit fails, although the fallback connection is on a lower bandwidth network.</span></span>

<span data-ttu-id="b1efd-141">**Défis**</span><span class="sxs-lookup"><span data-stu-id="b1efd-141">**Challenges**</span></span>

- <span data-ttu-id="b1efd-142">Cette méthode est difficile à configurer.</span><span class="sxs-lookup"><span data-stu-id="b1efd-142">Complex to configure.</span></span> <span data-ttu-id="b1efd-143">Vous devez configurer à la fois une connexion VPN et un circuit ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="b1efd-143">You need to set up both a VPN connection and an ExpressRoute circuit.</span></span>
- <span data-ttu-id="b1efd-144">Cette méthode requiert un matériel redondant (appliances VPN) et une connexion de passerelle VPN Azure redondante pour laquelle vous devrez payer des frais supplémentaires.</span><span class="sxs-lookup"><span data-stu-id="b1efd-144">Requires redundant hardware (VPN appliances), and a redundant Azure VPN Gateway connection for which you pay charges.</span></span>

<span data-ttu-id="b1efd-145">**Architecture de référence**</span><span class="sxs-lookup"><span data-stu-id="b1efd-145">**Reference architecture**</span></span>

- [<span data-ttu-id="b1efd-146">Réseau hybride avec ExpressRoute et basculement du VPN</span><span class="sxs-lookup"><span data-stu-id="b1efd-146">Hybrid network with ExpressRoute and VPN failover</span></span>](./expressroute-vpn-failover.md)


## <a name="hub-spoke-network-topology"></a><span data-ttu-id="b1efd-147">Topologie de réseau hub-and-spoke</span><span class="sxs-lookup"><span data-stu-id="b1efd-147">Hub-spoke network topology</span></span>

<span data-ttu-id="b1efd-148">Une topologie de réseau hub-and-spoke consiste à isoler des charges de travail tout en partageant des services tels que l’identité et la sécurité.</span><span class="sxs-lookup"><span data-stu-id="b1efd-148">A hub-spoke network topology is a way to isolate workloads while sharing services such as identity and security.</span></span> <span data-ttu-id="b1efd-149">Le hub est un réseau virtuel (VNet) dans Azure qui centralise la connectivité à votre réseau local.</span><span class="sxs-lookup"><span data-stu-id="b1efd-149">The hub is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="b1efd-150">Les rayons sont des réseaux virtuels qui s’associent au hub.</span><span class="sxs-lookup"><span data-stu-id="b1efd-150">The spokes are VNets that peer with the hub.</span></span> <span data-ttu-id="b1efd-151">Les services partagés sont déployés dans le hub, tandis que les charges de travail individuelles sont déployées en tant que rayons.</span><span class="sxs-lookup"><span data-stu-id="b1efd-151">Shared services are deployed in the hub, while individual workloads are deployed as spokes.</span></span>


<span data-ttu-id="b1efd-152">**Architectures de référence**</span><span class="sxs-lookup"><span data-stu-id="b1efd-152">**Reference architectures**</span></span>

- [<span data-ttu-id="b1efd-153">Topologie hub-and-spoke</span><span class="sxs-lookup"><span data-stu-id="b1efd-153">Hub-spoke topology</span></span>](./hub-spoke.md)
- [<span data-ttu-id="b1efd-154">Topologie hub-and-spoke avec des services partagés</span><span class="sxs-lookup"><span data-stu-id="b1efd-154">Hub-spoke with shared services</span></span>](./shared-services.md)
