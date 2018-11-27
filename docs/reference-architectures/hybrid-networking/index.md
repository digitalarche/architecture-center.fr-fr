---
title: Choisir une solution pour connecter un réseau local à Azure
description: Comparatif des architectures de référence pour connecter un réseau local à Azure.
author: telmosampaio
ms.date: 07/02/2018
ms.openlocfilehash: a9e2a212d65530e714635bbfae3a57766e77c3a6
ms.sourcegitcommit: 19a517a2fb70768b3edb9a7c3c37197baa61d9b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2018
ms.locfileid: "52295483"
---
# <a name="connect-an-on-premises-network-to-azure"></a><span data-ttu-id="b9993-103">Connecter un réseau local à Azure</span><span class="sxs-lookup"><span data-stu-id="b9993-103">Connect an on-premises network to Azure</span></span>

<span data-ttu-id="b9993-104">Cet article compare les options disponibles pour connecter un réseau local à un réseau virtuel (VNet) Azure.</span><span class="sxs-lookup"><span data-stu-id="b9993-104">This article compares options for connecting an on-premises network to an Azure Virtual Network (VNet).</span></span> <span data-ttu-id="b9993-105">Pour chaque option, une architecture de référence plus détaillée est disponible.</span><span class="sxs-lookup"><span data-stu-id="b9993-105">For each option, a more detailed reference architecture is available.</span></span>

## <a name="vpn-connection"></a><span data-ttu-id="b9993-106">Connexion VPN</span><span class="sxs-lookup"><span data-stu-id="b9993-106">VPN connection</span></span>

<span data-ttu-id="b9993-107">Une [passerelle VPN](/azure/vpn-gateway/vpn-gateway-about-vpngateways) est un type de passerelle de réseau virtuel qui envoie le trafic chiffré entre un réseau virtuel Azure et un emplacement local.</span><span class="sxs-lookup"><span data-stu-id="b9993-107">A [VPN gateway](/azure/vpn-gateway/vpn-gateway-about-vpngateways) is a type of virtual network gateway that sends encrypted traffic between an Azure virtual network and an on-premises location.</span></span> <span data-ttu-id="b9993-108">Le trafic chiffré est envoyé via le réseau Internet public.</span><span class="sxs-lookup"><span data-stu-id="b9993-108">The encrypted traffic goes over the public Internet.</span></span>

<span data-ttu-id="b9993-109">Cette architecture est idéale pour les applications hybrides avec un trafic peu volumineux entre le matériel sur site et le cloud, ou dans le cas où vous êtes disposé à accepter une latence un peu plus élevée pour bénéficier de la flexibilité et de la puissance de la technologie cloud.</span><span class="sxs-lookup"><span data-stu-id="b9993-109">This architecture is suitable for hybrid applications where the traffic between on-premises hardware and the cloud is likely to be light, or you are willing to trade slightly extended latency for the flexibility and processing power of the cloud.</span></span>

<span data-ttu-id="b9993-110">**Avantages**</span><span class="sxs-lookup"><span data-stu-id="b9993-110">**Benefits**</span></span>

- <span data-ttu-id="b9993-111">Cette méthode est facile à configurer.</span><span class="sxs-lookup"><span data-stu-id="b9993-111">Simple to configure.</span></span>

<span data-ttu-id="b9993-112">**Défis**</span><span class="sxs-lookup"><span data-stu-id="b9993-112">**Challenges**</span></span>

- <span data-ttu-id="b9993-113">Vous avez besoin d’un appareil VPN local.</span><span class="sxs-lookup"><span data-stu-id="b9993-113">Requires an on-premises VPN device.</span></span>
- <span data-ttu-id="b9993-114">Même si Microsoft garantit une disponibilité de 99,9 % pour chaque passerelle VPN, ce contrat [SLA](https://azure.microsoft.com/support/legal/sla/vpn-gateway/) couvre uniquement la passerelle VPN, et non votre connexion réseau à la passerelle.</span><span class="sxs-lookup"><span data-stu-id="b9993-114">Although Microsoft guarantees 99.9% availability for each VPN Gateway, this [SLA](https://azure.microsoft.com/support/legal/sla/vpn-gateway/) only covers the VPN gateway, and not your network connection to the gateway.</span></span>
- <span data-ttu-id="b9993-115">Une connexion VPN via la passerelle VPN Azure prend actuellement en charge une bande passante maximale de 1,25 Mbits/s.</span><span class="sxs-lookup"><span data-stu-id="b9993-115">A VPN connection over Azure VPN Gateway currently supports a maximum of 1.25 Gbps bandwidth.</span></span> <span data-ttu-id="b9993-116">Vous devrez peut-être partitionner votre réseau virtuel Azure sur plusieurs connexions VPN si vous pensez dépasser ce débit.</span><span class="sxs-lookup"><span data-stu-id="b9993-116">You may need to partition your Azure virtual network across multiple VPN connections if you expect to exceed this throughput.</span></span>

<span data-ttu-id="b9993-117">**Architecture de référence**</span><span class="sxs-lookup"><span data-stu-id="b9993-117">**Reference architecture**</span></span>

- [<span data-ttu-id="b9993-118">Réseau hybride avec une passerelle VPN</span><span class="sxs-lookup"><span data-stu-id="b9993-118">Hybrid network with VPN gateway</span></span>](./vpn.md)

## <a name="azure-expressroute-connection"></a><span data-ttu-id="b9993-119">Connexion Azure ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="b9993-119">Azure ExpressRoute connection</span></span>

<span data-ttu-id="b9993-120">Les connexions [ExpressRoute](/azure/expressroute/) utilisent une connexion privée et dédiée via un fournisseur de connectivité tiers.</span><span class="sxs-lookup"><span data-stu-id="b9993-120">[ExpressRoute](/azure/expressroute/) connections use a private, dedicated connection through a third-party connectivity provider.</span></span> <span data-ttu-id="b9993-121">La connexion privée étend votre réseau local à Azure.</span><span class="sxs-lookup"><span data-stu-id="b9993-121">The private connection extends your on-premises network into Azure.</span></span> 

<span data-ttu-id="b9993-122">Cette architecture est idéale pour les applications hybrides exécutant des charges de travail critiques et volumineuses qui nécessitent un niveau élevé d’évolutivité.</span><span class="sxs-lookup"><span data-stu-id="b9993-122">This architecture is suitable for hybrid applications running large-scale, mission-critical workloads that require a high degree of scalability.</span></span> 

<span data-ttu-id="b9993-123">**Avantages**</span><span class="sxs-lookup"><span data-stu-id="b9993-123">**Benefits**</span></span>

- <span data-ttu-id="b9993-124">La bande passante disponible est plus élevée, avec jusqu’à 10 Gbits/s selon le fournisseur de connectivité choisi.</span><span class="sxs-lookup"><span data-stu-id="b9993-124">Much higher bandwidth available; up to 10 Gbps depending on the connectivity provider.</span></span>
- <span data-ttu-id="b9993-125">Cette méthode prend en charge la mise à l’échelle dynamique de la bande passante afin de réduire les coûts pendant les périodes de faible demande.</span><span class="sxs-lookup"><span data-stu-id="b9993-125">Supports dynamic scaling of bandwidth to help reduce costs during periods of lower demand.</span></span> <span data-ttu-id="b9993-126">Toutefois, tous les fournisseurs de connectivité ne proposent pas cette option.</span><span class="sxs-lookup"><span data-stu-id="b9993-126">However, not all connectivity providers have this option.</span></span>
- <span data-ttu-id="b9993-127">En fonction du fournisseur de connectivité choisi, votre organisation peut bénéficier d’un accès direct aux clouds nationaux.</span><span class="sxs-lookup"><span data-stu-id="b9993-127">May allow your organization direct access to national clouds, depending on the connectivity provider.</span></span>
- <span data-ttu-id="b9993-128">Vous bénéficiez d’un contrat SLA garantissant une disponibilité de 99,9 % sur l’ensemble de la connexion.</span><span class="sxs-lookup"><span data-stu-id="b9993-128">99.9% availability SLA across the entire connection.</span></span>

<span data-ttu-id="b9993-129">**Défis**</span><span class="sxs-lookup"><span data-stu-id="b9993-129">**Challenges**</span></span>

- <span data-ttu-id="b9993-130">Cette méthode peut être difficile à configurer.</span><span class="sxs-lookup"><span data-stu-id="b9993-130">Can be complex to set up.</span></span> <span data-ttu-id="b9993-131">La création d’une connexion ExpressRoute nécessite l’intervention d’un fournisseur de connectivité tiers.</span><span class="sxs-lookup"><span data-stu-id="b9993-131">Creating an ExpressRoute connection requires working with a third-party connectivity provider.</span></span> <span data-ttu-id="b9993-132">Le fournisseur est responsable de l’approvisionnement de la connexion réseau.</span><span class="sxs-lookup"><span data-stu-id="b9993-132">The provider is responsible for provisioning the network connection.</span></span>
- <span data-ttu-id="b9993-133">Cette méthode requiert des routeurs locaux offrant une large bande passante.</span><span class="sxs-lookup"><span data-stu-id="b9993-133">Requires high-bandwidth routers on-premises.</span></span>

<span data-ttu-id="b9993-134">**Architecture de référence**</span><span class="sxs-lookup"><span data-stu-id="b9993-134">**Reference architecture**</span></span>

- [<span data-ttu-id="b9993-135">Réseau hybride avec ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="b9993-135">Hybrid network with ExpressRoute</span></span>](./expressroute.md)

## <a name="expressroute-with-vpn-failover"></a><span data-ttu-id="b9993-136">ExpressRoute avec basculement VPN</span><span class="sxs-lookup"><span data-stu-id="b9993-136">ExpressRoute with VPN failover</span></span>

<span data-ttu-id="b9993-137">Cette option combine les deux méthodes précédentes : elle utilise ExpressRoute dans des conditions normales, mais bascule vers une connexion VPN en cas de perte de connectivité au niveau du circuit ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="b9993-137">This options combines the previous two, using ExpressRoute in normal conditions, but failing over to a VPN connection if there is a loss of connectivity in the ExpressRoute circuit.</span></span>

<span data-ttu-id="b9993-138">Cette architecture est idéale pour les applications hybrides qui nécessitent une bande passante ExpressRoute plus élevée, ainsi qu’une connectivité réseau hautement disponible.</span><span class="sxs-lookup"><span data-stu-id="b9993-138">This architecture is suitable for hybrid applications that need the higher bandwidth of ExpressRoute, and also require highly available network connectivity.</span></span> 

<span data-ttu-id="b9993-139">**Avantages**</span><span class="sxs-lookup"><span data-stu-id="b9993-139">**Benefits**</span></span>

- <span data-ttu-id="b9993-140">Haute disponibilité si le circuit ExpressRoute échoue, même lorsque la connexion de secours intervient sur un réseau de bande passante inférieure.</span><span class="sxs-lookup"><span data-stu-id="b9993-140">High availability if the ExpressRoute circuit fails, although the fallback connection is on a lower bandwidth network.</span></span>

<span data-ttu-id="b9993-141">**Défis**</span><span class="sxs-lookup"><span data-stu-id="b9993-141">**Challenges**</span></span>

- <span data-ttu-id="b9993-142">Cette méthode est difficile à configurer.</span><span class="sxs-lookup"><span data-stu-id="b9993-142">Complex to configure.</span></span> <span data-ttu-id="b9993-143">Vous devez configurer à la fois une connexion VPN et un circuit ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="b9993-143">You need to set up both a VPN connection and an ExpressRoute circuit.</span></span>
- <span data-ttu-id="b9993-144">Cette méthode requiert un matériel redondant (appliances VPN) et une connexion de passerelle VPN Azure redondante pour laquelle vous devrez payer des frais supplémentaires.</span><span class="sxs-lookup"><span data-stu-id="b9993-144">Requires redundant hardware (VPN appliances), and a redundant Azure VPN Gateway connection for which you pay charges.</span></span>

<span data-ttu-id="b9993-145">**Architecture de référence**</span><span class="sxs-lookup"><span data-stu-id="b9993-145">**Reference architecture**</span></span>

- [<span data-ttu-id="b9993-146">Réseau hybride avec ExpressRoute et basculement du VPN</span><span class="sxs-lookup"><span data-stu-id="b9993-146">Hybrid network with ExpressRoute and VPN failover</span></span>](./expressroute-vpn-failover.md)


## <a name="hub-spoke-network-topology"></a><span data-ttu-id="b9993-147">Topologie de réseau hub-and-spoke</span><span class="sxs-lookup"><span data-stu-id="b9993-147">Hub-spoke network topology</span></span>

<span data-ttu-id="b9993-148">Une topologie de réseau hub-and-spoke consiste à isoler des charges de travail tout en partageant des services tels que l’identité et la sécurité.</span><span class="sxs-lookup"><span data-stu-id="b9993-148">A hub-spoke network topology is a way to isolate workloads while sharing services such as identity and security.</span></span> <span data-ttu-id="b9993-149">Le hub est un réseau virtuel (VNet) dans Azure qui centralise la connectivité à votre réseau local.</span><span class="sxs-lookup"><span data-stu-id="b9993-149">The hub is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="b9993-150">Les rayons sont des réseaux virtuels qui s’associent au hub.</span><span class="sxs-lookup"><span data-stu-id="b9993-150">The spokes are VNets that peer with the hub.</span></span> <span data-ttu-id="b9993-151">Les services partagés sont déployés dans le hub, tandis que les charges de travail individuelles sont déployées en tant que rayons.</span><span class="sxs-lookup"><span data-stu-id="b9993-151">Shared services are deployed in the hub, while individual workloads are deployed as spokes.</span></span>


<span data-ttu-id="b9993-152">**Architectures de référence**</span><span class="sxs-lookup"><span data-stu-id="b9993-152">**Reference architectures**</span></span>

- [<span data-ttu-id="b9993-153">Topologie hub-and-spoke</span><span class="sxs-lookup"><span data-stu-id="b9993-153">Hub-spoke topology</span></span>](./hub-spoke.md)
- [<span data-ttu-id="b9993-154">Topologie hub-and-spoke avec des services partagés</span><span class="sxs-lookup"><span data-stu-id="b9993-154">Hub-spoke with shared services</span></span>](./shared-services.md)
