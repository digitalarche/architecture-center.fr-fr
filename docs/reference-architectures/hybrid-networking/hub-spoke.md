---
title: "Implémentation d’une topologie de réseau hub-and-spoke dans Azure"
description: "Comment implémenter une topologie de réseau hub-and-spoke dans Azure."
author: telmosampaio
ms.date: 05/05/2017
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: e6f07a7962dd5728226b023700268340590d97a3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2017
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a><span data-ttu-id="f2a2d-103">Implémenter une topologie de réseau hub-and-spoke dans Azure</span><span class="sxs-lookup"><span data-stu-id="f2a2d-103">Implement a hub-spoke network topology in Azure</span></span>

<span data-ttu-id="f2a2d-104">Cette architecture de référence montre comment implémenter une topologie hub-and-spoke dans Azure.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-104">This reference architecture shows how to implement a hub-spoke topology in Azure.</span></span> <span data-ttu-id="f2a2d-105">Le *hub* est un réseau virtuel (VNet) dans Azure qui centralise la connectivité à votre réseau local.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-105">The *hub* is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="f2a2d-106">Les *membres spokes* sont des réseaux virtuels appairés avec le hub et qui peuvent être utilisés pour isoler les charges de travail.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-106">The *spokes* are VNets that peer with the hub, and can be used to isolate workloads.</span></span> <span data-ttu-id="f2a2d-107">Le trafic circule entre le centre de données local et le hub via une connexion de passerelle ExpressRoute ou VPN.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-107">Traffic flows between the on-premises datacenter and the hub through an ExpressRoute or VPN gateway connection.</span></span>  <span data-ttu-id="f2a2d-108">[**Déployez cette solution**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="f2a2d-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="f2a2d-109">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="f2a2d-109">![[0]][0]</span></span>

<span data-ttu-id="f2a2d-110">*Téléchargez un [fichier Visio][visio-download] de cette architecture.*</span><span class="sxs-lookup"><span data-stu-id="f2a2d-110">*Download a [Visio file][visio-download] of this architecture*</span></span>


<span data-ttu-id="f2a2d-111">Cette topologie présente les avantages suivants :</span><span class="sxs-lookup"><span data-stu-id="f2a2d-111">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="f2a2d-112">**Réduction les coûts** en centralisant les services qui peuvent être partagés par plusieurs charges de travail, comme les appliances virtuelles réseau et les serveurs DNS.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="f2a2d-113">**Surmonter les limites des abonnements** en appairant les réseaux virtuels de différents abonnements avec le hub central.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="f2a2d-114">**Séparation des préoccupations** entre le service informatique central (SecOps, InfraOps) et les charges de travail (DevOps).</span><span class="sxs-lookup"><span data-stu-id="f2a2d-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="f2a2d-115">Utilisations courantes de cette architecture :</span><span class="sxs-lookup"><span data-stu-id="f2a2d-115">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="f2a2d-116">Charges de travail déployées sur des environnements différents, tels que les environnements de développement, de test et de production, qui requièrent des services partagés tels que DNS, IDS, NTP ou AD DS.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="f2a2d-117">Les services partagés sont placés dans le réseau virtuel hub, tandis que chaque environnement est déployé sur un membre spoke pour garantir l’isolation.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="f2a2d-118">Charges de travail ne nécessitant pas de connectivité entre elles, mais nécessitant un accès aux services partagés.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="f2a2d-119">Entreprises nécessitant un contrôle centralisé des aspects de la sécurité, tel qu’un pare-feu dans le hub en tant que zone DMZ et une gestion séparée des charges de travail dans chaque membre spoke.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="f2a2d-120">Architecture</span><span class="sxs-lookup"><span data-stu-id="f2a2d-120">Architecture</span></span>

<span data-ttu-id="f2a2d-121">L’architecture est constituée des composants suivants :</span><span class="sxs-lookup"><span data-stu-id="f2a2d-121">The architecture consists of the following components.</span></span>

* <span data-ttu-id="f2a2d-122">**Réseau local**.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-122">**On-premises network**.</span></span> <span data-ttu-id="f2a2d-123">Un réseau local privé s’exécutant au sein d’une organisation.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-123">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="f2a2d-124">**Périphérique VPN**.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-124">**VPN device**.</span></span> <span data-ttu-id="f2a2d-125">Périphérique ou service qui assure la connectivité externe au réseau local.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="f2a2d-126">Le périphérique VPN peut être un périphérique matériel ou une solution logicielle telle que le service RRAS (Routing and Remote Access Service) dans Windows Server 2012.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="f2a2d-127">Pour obtenir la liste des périphériques VPN pris en charge et des informations sur la configuration de certains périphériques VPN pour la connexion à Azure, consultez [À propos des périphériques VPN pour les connexions de la passerelle VPN de site à site][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="f2a2d-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="f2a2d-128">**Passerelle de réseau virtuel VPN ou passerelle ExpressRoute**.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="f2a2d-129">La passerelle de réseau virtuel permet au réseau virtuel de se connecter au périphérique VPN, ou circuit ExpressRoute, utilisé pour la connectivité avec votre réseau local.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="f2a2d-130">Pour plus d’informations, consultez [Connecter un réseau local à Microsoft Azure Virtual Network][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="f2a2d-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="f2a2d-131">Les scripts de déploiement pour cette architecture de référence utilisent une passerelle VPN pour la connectivité et un réseau virtuel dans Azure pour simuler votre réseau local.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="f2a2d-132">**Réseau virtuel hub**.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-132">**Hub VNet**.</span></span> <span data-ttu-id="f2a2d-133">Réseau virtuel Azure utilisé comme hub dans la topologie hub-and-spoke.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="f2a2d-134">Le hub est le point central de la connectivité à votre réseau local et un emplacement pour héberger les services que peuvent utiliser les différentes charges de travail hébergées dans les réseaux virtuels spokes.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="f2a2d-135">**Sous-réseau de passerelle**.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-135">**Gateway subnet**.</span></span> <span data-ttu-id="f2a2d-136">Les passerelles de réseau virtuel sont conservées dans le même sous-réseau.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-136">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="f2a2d-137">**Sous-réseau de services partagés**.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-137">**Shared services subnet**.</span></span> <span data-ttu-id="f2a2d-138">Sous-réseau dans le réseau virtuel hub utilisé pour héberger les services que peuvent partager tous les membres spokes, tels que DNS ou AD DS.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-138">A subnet in the hub VNet used to host services that can be shared among all spokes, such as DNS or AD DS.</span></span>

* <span data-ttu-id="f2a2d-139">**Réseaux virtuels spokes**.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-139">**Spoke VNets**.</span></span> <span data-ttu-id="f2a2d-140">Un ou plusieurs réseaux virtuels Azure qui sont utilisés comme membres spokes dans la topologie hub-and-spoke.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-140">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="f2a2d-141">Les membres spokes peuvent servir à isoler les charges de travail dans leurs propres réseaux virtuels, qui sont alors gérées séparément des autres membres spokes.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-141">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="f2a2d-142">Chaque charge de travail peut inclure plusieurs niveaux, avec plusieurs sous-réseaux connectés à l’aide d’équilibreurs de charge Azure.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-142">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="f2a2d-143">Pour plus d’informations sur l’infrastructure d’application, consultez [Running Windows VM workloads][windows-vm-ra] (Exécution de charges de travail de machine virtuelle Windows) et [Exécution de charges de travail de machine virtuelle Linux][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="f2a2d-143">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="f2a2d-144">**Appairage de réseaux virtuels**.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-144">**VNet peering**.</span></span> <span data-ttu-id="f2a2d-145">Deux réseaux virtuels dans la même région Azure peuvent être connectés à l’aide d’une [connexion d’appairage][vnet-peering].</span><span class="sxs-lookup"><span data-stu-id="f2a2d-145">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="f2a2d-146">Les connexions d’appairage sont des connexions non transitives et à faible latence entre des réseaux virtuels.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-146">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="f2a2d-147">Une fois appairés, les réseaux virtuels échangent le trafic à l’aide de la dorsale principale d’Azure, sans avoir besoin d’un routeur.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-147">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="f2a2d-148">Dans une topologie de réseau hub-and-spoke, vous utilisez l’appairage de réseaux virtuels pour connecter le hub à chaque membre spoke.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-148">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="f2a2d-149">Cet article couvre uniquement les déploiements [Resource Manager](/azure/azure-resource-manager/resource-group-overview), mais vous pouvez également connecter un réseau virtuel classique à un réseau virtuel Resource Manager dans un même abonnement.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-149">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="f2a2d-150">De cette façon, vos membres spokes peuvent héberger des déploiements classiques tout en tirant parti des services partagés dans le hub.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-150">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>


## <a name="recommendations"></a><span data-ttu-id="f2a2d-151">Recommandations</span><span class="sxs-lookup"><span data-stu-id="f2a2d-151">Recommendations</span></span>

<span data-ttu-id="f2a2d-152">Les recommandations suivantes s’appliquent à la plupart des scénarios.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-152">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="f2a2d-153">Suivez ces recommandations, sauf si vous avez un besoin spécifique qui vous oblige à les ignorer.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-153">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="f2a2d-154">Groupes de ressources</span><span class="sxs-lookup"><span data-stu-id="f2a2d-154">Resource groups</span></span>

<span data-ttu-id="f2a2d-155">Le réseau virtuel hub et chaque réseau virtuel spoke peuvent être implémentés dans différents groupes de ressources, voire dans différents abonnements, tant qu’ils appartiennent au même locataire Azure Active Directory (Azure AD) dans la même région Azure.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-155">The hub VNet, and each spoke VNet, can be implemented in different resource groups, and even different subscriptions, as long as they belong to the same Azure Active Directory (Azure AD) tenant in the same Azure region.</span></span> <span data-ttu-id="f2a2d-156">Cela permet de décentraliser la gestion de chaque charge de travail, tout en partageant les services gérés dans le réseau virtuel hub.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-156">This allows for a decentralized management of each workload, while sharing services maintained in the hub VNet.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="f2a2d-157">Réseau virtuel et sous réseau GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="f2a2d-157">VNet and GatewaySubnet</span></span>

<span data-ttu-id="f2a2d-158">Créez un sous-réseau nommé *GatewaySubnet*, avec une plage d’adresses de /27.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-158">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="f2a2d-159">La passerelle de réseau virtuel requiert ce sous-réseau.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-159">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="f2a2d-160">En allouant 32 adresses à ce sous-réseau, vous devriez éviter les limitations de taille de passerelle.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-160">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span>

<span data-ttu-id="f2a2d-161">Pour plus d’informations sur la configuration de la passerelle, consultez les architectures de référence suivantes, en fonction de votre type de connexion :</span><span class="sxs-lookup"><span data-stu-id="f2a2d-161">For more information about setting up the gateway, see the following reference architectures, depending on your connection type:</span></span>

- <span data-ttu-id="f2a2d-162">[Réseau hybride utilisant ExpressRoute][guidance-expressroute]</span><span class="sxs-lookup"><span data-stu-id="f2a2d-162">[Hybrid network using ExpressRoute][guidance-expressroute]</span></span>
- <span data-ttu-id="f2a2d-163">[Réseau hybride utilisant une passerelle VPN][guidance-vpn]</span><span class="sxs-lookup"><span data-stu-id="f2a2d-163">[Hybrid network using a VPN gateway][guidance-vpn]</span></span>

<span data-ttu-id="f2a2d-164">Pour accroître la disponibilité, vous pouvez utiliser ExpressRoute et un VPN pour le basculement.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-164">For higher availability, you can use ExpressRoute plus a VPN for failover.</span></span> <span data-ttu-id="f2a2d-165">Consultez [Connecter un réseau local à Azure à l’aide d’ExpressRoute avec basculement VPN][hybrid-ha].</span><span class="sxs-lookup"><span data-stu-id="f2a2d-165">See [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha].</span></span>

<span data-ttu-id="f2a2d-166">Une topologie hub-and-spoke peut également être utilisée sans passerelle, si vous n’avez pas besoin de connectivité avec votre réseau local.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-166">A hub-spoke topology can also be used without a gateway, if you don't need connectivity with your on-premises network.</span></span> 

### <a name="vnet-peering"></a><span data-ttu-id="f2a2d-167">Homologation de réseaux virtuels</span><span class="sxs-lookup"><span data-stu-id="f2a2d-167">VNet peering</span></span>

<span data-ttu-id="f2a2d-168">L’appairage de réseaux virtuels est une relation non transitive entre deux réseaux virtuels.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-168">VNet peering is a non-transitive relationship between two VNets.</span></span> <span data-ttu-id="f2a2d-169">Si des membres spokes doivent se connecter les uns aux autres, envisagez d’ajouter une connexion d’appairage distincte entre ces membres spokes.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-169">If you require spokes to connect to each other, consider adding a separate peering connection between those spokes.</span></span>

<span data-ttu-id="f2a2d-170">Toutefois, cette situation vous prive très rapidement de connexions d’appairage en raison du [nombre maximal d’appairages de réseaux virtuels par réseau virtuel][vnet-peering-limit].</span><span class="sxs-lookup"><span data-stu-id="f2a2d-170">However, if you have several spokes that need to connect with each other, you will run out of possible peering connections very quickly due to the [limitation on number of VNets peerings per VNet][vnet-peering-limit].</span></span> <span data-ttu-id="f2a2d-171">Dans ce scénario, envisagez d’utiliser des itinéraires définis par l’utilisateur afin que le trafic destiné à un membre spoke soit envoyé à une appliance virtuelle réseau faisant office de routeur sur le réseau virtuel hub.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-171">In this scenario, consider using user defined routes (UDRs) to force traffic destined to a spoke to be sent to an NVA acting as a router at the hub VNet.</span></span> <span data-ttu-id="f2a2d-172">Ainsi, les membres spokes peuvent se connecter les uns aux autres.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-172">This will allow the spokes to connect to each other.</span></span>

<span data-ttu-id="f2a2d-173">Vous pouvez également configurer les membres spokes afin qu’ils utilisent la passerelle de réseau virtuel hub pour communiquer avec des réseaux distants.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-173">You can also configure spokes to use the hub VNet gateway to communicate with remote networks.</span></span> <span data-ttu-id="f2a2d-174">Pour que le trafic de la passerelle circule entre le membre spoke et le hub, puis se connecte à des réseaux distants, vous devez effectuer les opérations suivantes :</span><span class="sxs-lookup"><span data-stu-id="f2a2d-174">To allow gateway traffic to flow from spoke to hub, and connect to remote networks, you must:</span></span>

  - <span data-ttu-id="f2a2d-175">Configurer la connexion d’appairage de réseaux virtuels dans le hub pour **autoriser le transit par passerelle**.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-175">Configure the VNet peering connection in the hub to **allow gateway transit**.</span></span>
  - <span data-ttu-id="f2a2d-176">Configurer la connexion d’appairage de réseaux virtuels dans chaque membre spoke pour **utiliser des passerelles distantes**.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-176">Configure the VNet peering connection in each spoke to **use remote gateways**.</span></span>
  - <span data-ttu-id="f2a2d-177">Configurer toutes les connexions d’appairages de réseaux virtuels pour **autoriser le trafic transféré**.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-177">Configure all VNet peering connections to **allow forwarded traffic**.</span></span>

## <a name="considerations"></a><span data-ttu-id="f2a2d-178">Considérations</span><span class="sxs-lookup"><span data-stu-id="f2a2d-178">Considerations</span></span>

### <a name="spoke-connectivity"></a><span data-ttu-id="f2a2d-179">Connectivité entre membres spokes</span><span class="sxs-lookup"><span data-stu-id="f2a2d-179">Spoke connectivity</span></span>

<span data-ttu-id="f2a2d-180">Si vous avez besoin de connectivité entre des membres spokes, envisagez d’implémenter une appliance virtuelle réseau pour le routage dans le hub et d’utiliser des itinéraires définis par l’utilisateur dans le membre spoke pour transférer le trafic vers le hub.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-180">If you require connectivity between spokes, consider implementing an NVA for routing in the hub, and using UDRs in the spoke to forward traffic to the hub.</span></span>

<span data-ttu-id="f2a2d-181">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="f2a2d-181">![[2]][2]</span></span>

<span data-ttu-id="f2a2d-182">Dans ce scénario, vous devez configurer les connexions d’appairage pour **autoriser le trafic transféré**.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-182">In this scenario, you must configure the peering connections to **allow forwarded traffic**.</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="f2a2d-183">Surmonter les limites d’appairage de réseaux virtuels</span><span class="sxs-lookup"><span data-stu-id="f2a2d-183">Overcoming VNet peering limits</span></span>

<span data-ttu-id="f2a2d-184">Veillez à prendre en compte le [nombre maximal d’appairages de réseaux virtuels par réseau virtuel][vnet-peering-limit] dans Azure.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-184">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="f2a2d-185">Si vous décidez que vous avez besoin d’un nombre de membres spokes supérieur à celui autorisé par la limite, envisagez de créer une topologie hub-and-spoke/hub-and-spoke, où les membres spokes du premier niveau de membres spokes font également office de hubs.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-185">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="f2a2d-186">Le diagramme qui suit montre cette topologie.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-186">The following diagram shows this approach.</span></span>

<span data-ttu-id="f2a2d-187">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="f2a2d-187">![[3]][3]</span></span>

<span data-ttu-id="f2a2d-188">Déterminez également les services qui sont partagés dans le hub, afin que ce dernier puisse prendre en charge un plus grand nombre de membres spokes.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-188">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="f2a2d-189">Par exemple, si votre hub fournit des services de pare-feu, tenez compte des limites de bande passante de votre solution de pare-feu quand vous ajoutez plusieurs membres spokes.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-189">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="f2a2d-190">Vous pourriez souhaiter transférer certains de ces services partagés vers un second niveau de hubs.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-190">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="f2a2d-191">Déployer la solution</span><span class="sxs-lookup"><span data-stu-id="f2a2d-191">Deploy the solution</span></span>

<span data-ttu-id="f2a2d-192">Un déploiement pour cette architecture est disponible sur [GitHub][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="f2a2d-192">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="f2a2d-193">Il utilise des machines virtuelles Ubuntu dans chaque réseau virtuel pour tester la connectivité.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-193">It uses Ubuntu VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="f2a2d-194">Aucun service réel n’est hébergé dans le sous-réseau **shared-services** du **réseau virtuel hub**.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-194">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="f2a2d-195">Composants requis</span><span class="sxs-lookup"><span data-stu-id="f2a2d-195">Prerequisites</span></span>

<span data-ttu-id="f2a2d-196">Avant de pouvoir déployer l’architecture de référence sur votre propre abonnement, vous devez effectuer les étapes suivantes.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-196">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="f2a2d-197">Clonez, dupliquez ou téléchargez le fichier zip pour le dépôt GitHub des [architectures de référence AzureCAT][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="f2a2d-197">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="f2a2d-198">Si vous préférez utiliser l’interface de ligne de commande Azure, vérifiez qu’Azure CLI 2.0 est installé sur votre ordinateur.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-198">If you prefer to use the Azure CLI, make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="f2a2d-199">Pour installer l’interface CLI, suivez les instructions fournies dans [Installer Azure CLI 2.0][azure-cli-2].</span><span class="sxs-lookup"><span data-stu-id="f2a2d-199">To install the CLI, follow the instructions in [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="f2a2d-200">Si vous préférez utiliser PowerShell, vérifiez que le dernier module PowerShell pour Azure est installé sur votre ordinateur.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-200">If you prefer to use PowerShell, make sure you have the latest PowerShell module for Azure installed on you computer.</span></span> <span data-ttu-id="f2a2d-201">Pour installer le dernier module Azure PowerShell, suivez les instructions dans [Installer PowerShell pour Azure][azure-powershell].</span><span class="sxs-lookup"><span data-stu-id="f2a2d-201">To install the latest Azure PowerShell module, follow the instructions in [Install PowerShell for Azure][azure-powershell].</span></span>

4. <span data-ttu-id="f2a2d-202">À partir d’une invite de commandes, d’une invite bash ou de l’invite de commandes PowerShell, connectez-vous à votre compte Azure à l’aide de l’une des commandes ci-dessous et suivez les invites.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-202">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using one of the commands below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

  ```powershell
  Login-AzureRmAccount
  ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="f2a2d-203">Déployer le centre de données local simulé</span><span class="sxs-lookup"><span data-stu-id="f2a2d-203">Deploy the simulated on-premises datacenter</span></span>

<span data-ttu-id="f2a2d-204">Pour déployer le centre de données local simulé en tant que réseau virtuel Azure, effectuez les étapes suivantes.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-204">To deploy the simulated on-premises datacenter as an Azure VNet, perform the following steps.</span></span>

1. <span data-ttu-id="f2a2d-205">Accédez au dossier `hybrid-networking\hub-spoke\onprem` pour le dépôt que vous avez téléchargé à l’étape des prérequis ci-dessus.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-205">Navigate to the `hybrid-networking\hub-spoke\onprem` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="f2a2d-206">Ouvrez le fichier `onprem.vm.parameters.json` et entrez un nom d’utilisateur et un mot de passe entre les guillemets aux lignes 11 et 12, comme illustré ci-dessous, puis enregistrez le fichier.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-206">Open the `onprem.vm.parameters.json` file and enter a username and password between the quotes in line 11 and 12, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="f2a2d-207">Exécutez la commande bash ou PowerShell suivante pour déployer l’environnement local simulé en tant que réseau virtuel dans Azure.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-207">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="f2a2d-208">Remplacez les valeurs par l’abonnement, le nom de groupe de ressources et la région Azure.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-208">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./onprem.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > <span data-ttu-id="f2a2d-209">Si vous décidez d’utiliser un nom de groupe de ressources différent (autre que `ra-onprem-rg`), veillez à rechercher tous les fichiers de paramètre qui utilisent ce nom et à les modifier afin qu’ils utilisent votre propre nom de groupe de ressources.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-209">If you decide to use a different resource group name (other than `ra-onprem-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="f2a2d-210">Attendez que le déploiement se termine.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-210">Wait for the deployment to finish.</span></span> <span data-ttu-id="f2a2d-211">Ce déploiement crée un réseau virtuel, une machine virtuelle exécutant Ubuntu et une passerelle VPN.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-211">This deployment creates a virtual network, a virtual machine running Ubuntu, and a VPN gateway.</span></span> <span data-ttu-id="f2a2d-212">La création de la passerelle VPN peut prendre plus de 40 minutes.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-212">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="azure-hub-vnet"></a><span data-ttu-id="f2a2d-213">Réseau virtuel hub Azure</span><span class="sxs-lookup"><span data-stu-id="f2a2d-213">Azure hub VNet</span></span>

<span data-ttu-id="f2a2d-214">Pour déployer le réseau virtuel hub et vous connecter au réseau virtuel local simulé créé ci-dessus, effectuez les étapes suivantes.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-214">To deploy the hub VNet, and connect to the simulated on-premises VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="f2a2d-215">Accédez au dossier `hybrid-networking\hub-spoke\hub` pour le dépôt que vous avez téléchargé à l’étape des prérequis ci-dessus.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-215">Navigate to the `hybrid-networking\hub-spoke\hub` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="f2a2d-216">Ouvrez le fichier `hub.vm.parameters.json` et entrez un nom d’utilisateur et un mot de passe entre les guillemets aux lignes 11 et 12, comme illustré ci-dessous, puis enregistrez le fichier.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-216">Open the `hub.vm.parameters.json` file and enter a username and password between the quotes in line 11 and 12, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="f2a2d-217">Ouvrez le fichier `hub.gateway.parameters.json` et entrez une clé partagée entre les guillemets à la ligne 23, comme illustré ci-dessous, puis enregistrez le fichier.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-217">Open the `hub.gateway.parameters.json` file and enter a shared key between the quotes in line 23, as shown below, then save the file.</span></span> <span data-ttu-id="f2a2d-218">Notez cette valeur, car vous devrez l’utiliser ultérieurement dans le déploiement.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-218">Keep a note of this value, you will need to use it later in the deployment.</span></span>

  ```bash
  "sharedKey": "",
  ```

4. <span data-ttu-id="f2a2d-219">Exécutez la commande bash ou PowerShell suivante pour déployer l’environnement local simulé en tant que réseau virtuel dans Azure.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-219">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="f2a2d-220">Remplacez les valeurs par l’abonnement, le nom de groupe de ressources et la région Azure.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-220">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./hub.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus
  ```

  ```powershell
  ./hub.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus
  ```
  > [!NOTE]
  > <span data-ttu-id="f2a2d-221">Si vous décidez d’utiliser un nom de groupe de ressources différent (autre que `ra-hub-rg`), veillez à rechercher tous les fichiers de paramètre qui utilisent ce nom et à les modifier afin qu’ils utilisent votre propre nom de groupe de ressources.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-221">If you decide to use a different resource group name (other than `ra-hub-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="f2a2d-222">Attendez que le déploiement se termine.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-222">Wait for the deployment to finish.</span></span> <span data-ttu-id="f2a2d-223">Ce déploiement crée un réseau virtuel, une machine virtuelle exécutant Ubuntu, une passerelle VPN et une connexion à la passerelle créée à la section précédente.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-223">This deployment creates a virtual network, a virtual machine running Ubuntu, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="f2a2d-224">La création de la passerelle VPN peut prendre plus de 40 minutes.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-224">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="connection-from-on-premises-to-the-hub"></a><span data-ttu-id="f2a2d-225">Connexion au hub à partir de l’environnement local</span><span class="sxs-lookup"><span data-stu-id="f2a2d-225">Connection from on-premises to the hub</span></span>

<span data-ttu-id="f2a2d-226">Pour vous connecter au réseau virtuel hub à partir du centre de données local simulé, effectuez les étapes suivantes.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-226">To connect from the simulated on-premises datacenter to the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="f2a2d-227">Accédez au dossier `hybrid-networking\hub-spoke\onprem` pour le dépôt que vous avez téléchargé à l’étape des prérequis ci-dessus.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-227">Navigate to the `hybrid-networking\hub-spoke\onprem` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="f2a2d-228">Ouvrez le fichier `onprem.connection.parameters.json` et entrez une clé partagée entre les guillemets à la ligne 9, comme illustré ci-dessous, puis enregistrez le fichier.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-228">Open the `onprem.connection.parameters.json` file and enter a shared key between the quotes in line 9, as shown below, then save the file.</span></span> <span data-ttu-id="f2a2d-229">Cette valeur de clé partagée doit être la même que celle utilisée dans la passerelle locale que vous avez déployée.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-229">This shared key value must be the same used in the on-premises gateway you deployed previously.</span></span>

  ```bash
  "sharedKey": "",
  ```

3. <span data-ttu-id="f2a2d-230">Exécutez la commande bash ou PowerShell suivante pour déployer l’environnement local simulé en tant que réseau virtuel dans Azure.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-230">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="f2a2d-231">Remplacez les valeurs par l’abonnement, le nom de groupe de ressources et la région Azure.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-231">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./onprem.connection.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.connection.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > <span data-ttu-id="f2a2d-232">Si vous décidez d’utiliser un nom de groupe de ressources différent (autre que `ra-onprem-rg`), veillez à rechercher tous les fichiers de paramètre qui utilisent ce nom et à les modifier afin qu’ils utilisent votre propre nom de groupe de ressources.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-232">If you decide to use a different resource group name (other than `ra-onprem-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="f2a2d-233">Attendez que le déploiement se termine.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-233">Wait for the deployment to finish.</span></span> <span data-ttu-id="f2a2d-234">Ce déploiement crée une connexion entre le réseau virtuel utilisé pour simuler un centre de données local et le réseau virtuel hub.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-234">This deployment creates a connection between the VNet used to simulate an on-premises datacenter, and the hub VNet.</span></span>

### <a name="azure-spoke-vnets"></a><span data-ttu-id="f2a2d-235">Réseaux virtuels spokes Azure</span><span class="sxs-lookup"><span data-stu-id="f2a2d-235">Azure spoke VNets</span></span>

<span data-ttu-id="f2a2d-236">Pour déployer les réseaux virtuels spokes et vous connecter au réseau virtuel hub créé ci-dessus, effectuez les étapes suivantes.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-236">To deploy the spoke VNets, and connect to the hub VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="f2a2d-237">Accédez au dossier `hybrid-networking\hub-spoke\spokes` pour le dépôt que vous avez téléchargé à l’étape des prérequis ci-dessus.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-237">Switch to the `hybrid-networking\hub-spoke\spokes` folder for the repository you download in the pre-requisites step above.</span></span>

2. <span data-ttu-id="f2a2d-238">Ouvrez le fichier `spoke1.web.parameters.json` et entrez un nom d’utilisateur et un mot de passe entre les guillemets aux lignes 53 et 54, comme illustré ci-dessous, puis enregistrez le fichier.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-238">Open the `spoke1.web.parameters.json` file and enter a username and password between the quotes in line 53 and 54, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="f2a2d-239">Répétez l’étape précédente pour le fichier `spoke2.web.parameters.json`.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-239">Repeat the previous step for file `spoke2.web.parameters.json`.</span></span>

4. <span data-ttu-id="f2a2d-240">Exécutez la commande bash ou PowerShell ci-dessous pour déployer le premier membre spoke et le connecter au hub.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-240">Run the bash or PowerShell command below to deploy the first spoke and connect it to the hub.</span></span> <span data-ttu-id="f2a2d-241">Remplacez les valeurs par l’abonnement, le nom de groupe de ressources et la région Azure.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-241">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke1-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke1-rg `
    -Location westus `
    -Spoke 1
  ```
  > [!NOTE]
  > <span data-ttu-id="f2a2d-242">Si vous décidez d’utiliser un nom de groupe de ressources différent (autre que `ra-spoke1-rg`), veillez à rechercher tous les fichiers de paramètre qui utilisent ce nom et à les modifier afin qu’ils utilisent votre propre nom de groupe de ressources.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-242">If you decide to use a different resource group name (other than `ra-spoke1-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="f2a2d-243">Attendez que le déploiement se termine.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-243">Wait for the deployment to finish.</span></span> <span data-ttu-id="f2a2d-244">Ce déploiement crée un réseau virtuel, un équilibreur de charge avec trois machines virtuelles exécutant Ubuntu et Apache et une connexion d’appairage de réseaux virtuels au réseau virtuel hub créé à la section précédente.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-244">This deployment creates a virtual network, a load balancer with three virtual machine running Ubuntu and Apache, and a VNet peering connection to the hub VNet created in the previous section.</span></span> <span data-ttu-id="f2a2d-245">Ce déploiement peut prendre plus de 20 minutes.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-245">This deployment may take over 20 minutes.</span></span>

6. <span data-ttu-id="f2a2d-246">Exécutez la commande bash ou PowerShell ci-dessous pour déployer le premier membre spoke et le connecter au hub.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-246">Run the bash or PowerShell command below to deploy the first spoke and connect it to the hub.</span></span> <span data-ttu-id="f2a2d-247">Remplacez les valeurs par l’abonnement, le nom de groupe de ressources et la région Azure.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-247">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke2-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke2-rg `
    -Location westus `
    -Spoke 2
  ```
  > [!NOTE]
  > <span data-ttu-id="f2a2d-248">Si vous décidez d’utiliser un nom de groupe de ressources différent (autre que `ra-spoke2-rg`), veillez à rechercher tous les fichiers de paramètre qui utilisent ce nom et à les modifier afin qu’ils utilisent votre propre nom de groupe de ressources.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-248">If you decide to use a different resource group name (other than `ra-spoke2-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="f2a2d-249">Attendez que le déploiement se termine.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-249">Wait for the deployment to finish.</span></span> <span data-ttu-id="f2a2d-250">Ce déploiement crée un réseau virtuel, un équilibreur de charge avec trois machines virtuelles exécutant Ubuntu et Apache et une connexion d’appairage de réseaux virtuels au réseau virtuel hub créé à la section précédente.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-250">This deployment creates a virtual network, a load balancer with three virtual machine running Ubuntu and Apache, and a VNet peering connection to the hub VNet created in the previous section.</span></span> <span data-ttu-id="f2a2d-251">Ce déploiement peut prendre plus de 20 minutes.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-251">This deployment may take over 20 minutes.</span></span>

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a><span data-ttu-id="f2a2d-252">Appairage de réseaux virtuels hub Azure avec des réseaux virtuels spokes</span><span class="sxs-lookup"><span data-stu-id="f2a2d-252">Azure hub VNet peering to spoke VNets</span></span>

<span data-ttu-id="f2a2d-253">Pour déployer les connexions d’appairage de réseaux virtuels pour le réseau virtuel hub, effectuez les étapes suivantes.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-253">To deploy the VNet peering connections for the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="f2a2d-254">Accédez au dossier `hybrid-networking\hub-spoke\hub` pour le dépôt que vous avez téléchargé à l’étape des prérequis ci-dessus.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-254">Switch to the `hybrid-networking\hub-spoke\hub` folder for the repository you download in the pre-requisites step above.</span></span>

2. <span data-ttu-id="f2a2d-255">Exécutez la commande bash ou PowerShell ci-dessous pour déployer la connexion d’appairage sur le premier membre spoke.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-255">Run the bash or PowerShell command below to deploy the peering connection to the first spoke.</span></span> <span data-ttu-id="f2a2d-256">Remplacez les valeurs par l’abonnement, le nom de groupe de ressources et la région Azure.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-256">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 1
  ```

2. <span data-ttu-id="f2a2d-257">Exécutez la commande bash ou PowerShell ci-dessous pour déployer la connexion d’appairage sur le second membre spoke.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-257">Run the bash or PowerShell command below to deploy the peering connection to the second spoke.</span></span> <span data-ttu-id="f2a2d-258">Remplacez les valeurs par l’abonnement, le nom de groupe de ressources et la région Azure.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-258">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 2
  ```

### <a name="test-connectivity"></a><span data-ttu-id="f2a2d-259">Tester la connectivité</span><span class="sxs-lookup"><span data-stu-id="f2a2d-259">Test connectivity</span></span>

<span data-ttu-id="f2a2d-260">Pour vérifier que la topologie hub-and-spoke connectée à un déploiement de centre de données local a fonctionné, effectuez les étapes suivantes.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-260">To verify that the hub-spoke topology connected to an on-premises datacenter deployment worked, follow these steps.</span></span>

1. <span data-ttu-id="f2a2d-261">À partir du [portail Azure] [portail], connectez-vous à votre abonnement, puis accédez à la machine virtuelle `ra-onprem-vm1` dans le groupe de ressources `ra-onprem-rg`.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-261">From the [Azure portal][portal], connect to your subscription, and navigate to the `ra-onprem-vm1` virtual machine in the `ra-onprem-rg` resource group.</span></span>

2. <span data-ttu-id="f2a2d-262">Dans le panneau `Overview`, notez l’adresse `Public IP address` de la machine virtuelle.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-262">In the `Overview` blade, note the `Public IP address` for the VM.</span></span>

3. <span data-ttu-id="f2a2d-263">Utilisez un client SSH pour vous connecter à l’adresse IP que vous avez notée ci-dessus à l’aide du nom d’utilisateur et du mot de passe que vous avez spécifiés pendant le déploiement.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-263">Use an SSH client to connect to the IP address you noted above using the user name and password you specified during deployment.</span></span>

4. <span data-ttu-id="f2a2d-264">À partir de l’invite de commandes sur la machine virtuelle à laquelle vous vous êtes connecté, exécutez la commande suivante pour tester la connectivité entre le réseau virtuel local et le réseau virtuel Spoke1.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-264">From the command prompt on the VM you connected to, run the command below to test connectivity from the on-premises VNet to the Spoke1 VNet.</span></span>

  ```bash
  ping 10.1.1.37
  ```

### <a name="add-connectivity-between-spokes"></a><span data-ttu-id="f2a2d-265">Ajouter la connectivité entre les membres spokes</span><span class="sxs-lookup"><span data-stu-id="f2a2d-265">Add connectivity between spokes</span></span>

<span data-ttu-id="f2a2d-266">Pour autoriser les membres spokes à se connecter les uns aux autres, vous devez déployer des itinéraires définis par l’utilisateur sur chaque membre spoke qui transfère du trafic destiné à d’autres membres spokes vers la passerelle dans le réseau virtuel hub.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-266">If you want to allow spokes to connect to each other, you must deploy UDRs to each spoke that forward traffic destined to other spokes to the gateway in the hub VNet.</span></span> <span data-ttu-id="f2a2d-267">Effectuez les étapes suivantes pour vérifier que vous n’êtes pas en mesure de vous connecter à partir d’un membre spoke à un autre, puis déployer les itinéraires définis par l’utilisateur et retester la connectivité.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-267">Perform the following steps to verify that currently you are not able to connect from a spoke to another, then deploy the UDRs and test connectivity again.</span></span>

1. <span data-ttu-id="f2a2d-268">Répétez les étapes 1 à 4 ci-dessus, si vous n’êtes plus connecté à la machine virtuelle du serveur de rebond.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-268">Repeat steps 1 to 4 above, if you are not connected to the jumpbox VM any longer.</span></span>

2. <span data-ttu-id="f2a2d-269">Connectez-vous à un des serveurs web dans le membre spoke 1.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-269">Connect to one of the web servers in spoke 1.</span></span>

  ```bash
  ssh 10.1.1.37
  ```

3. <span data-ttu-id="f2a2d-270">Testez la connectivité entre le membre spoke 1 et le membre spoke 2.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-270">Test the connectivity between spoke 1 and spoke 2.</span></span> <span data-ttu-id="f2a2d-271">Le test doit se solder par un échec.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-271">It should fail.</span></span>

  ```bash
  ping 10.1.2.37
  ```

4. <span data-ttu-id="f2a2d-272">Revenez à l’invite de commandes de votre ordinateur.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-272">Switch back to your computer's command prompt.</span></span>

5. <span data-ttu-id="f2a2d-273">Accédez au dossier `hybrid-networking\hub-spoke\spokes` pour le dépôt que vous avez téléchargé à l’étape des prérequis ci-dessus.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-273">Switch to the `hybrid-networking\hub-spoke\spokes` folder for the repository you downloaded in the pre-requisites step above.</span></span>

6. <span data-ttu-id="f2a2d-274">Exécutez la commande bash ou PowerShell ci-dessous pour déployer un itinéraire défini par l’utilisateur sur le premier membre spoke.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-274">Run the bash or PowerShell command below to deploy an UDR to the first spoke.</span></span> <span data-ttu-id="f2a2d-275">Remplacez les valeurs par l’abonnement, le nom de groupe de ressources et la région Azure.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-275">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./spoke.udr.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke1-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./spoke.udr.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke1-rg `
    -Location westus `
    -Spoke 1
  ```

7. <span data-ttu-id="f2a2d-276">Exécutez la commande bash ou PowerShell ci-dessous pour déployer un itinéraire défini par l’utilisateur sur le second membre spoke.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-276">Run the bash or PowerShell command below to deploy an UDR to the second spoke.</span></span> <span data-ttu-id="f2a2d-277">Remplacez les valeurs par l’abonnement, le nom de groupe de ressources et la région Azure.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-277">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./spoke.udr.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke2-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./spoke.udr.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke2-rg `
    -Location westus `
    -Spoke 2
  ```

8. <span data-ttu-id="f2a2d-278">Revenez au terminal ssh.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-278">Switch back to the ssh terminal.</span></span>

9. <span data-ttu-id="f2a2d-279">Testez la connectivité entre le membre spoke 1 et le membre spoke 2.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-279">Test the connectivity between spoke 1 and spoke 2.</span></span> <span data-ttu-id="f2a2d-280">Le test doit réussir.</span><span class="sxs-lookup"><span data-stu-id="f2a2d-280">It should succeed.</span></span>

  ```bash
  ping 10.1.2.37
  ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azure-powershell]: /powershell/azure/install-azure-ps?view=azuresmps-3.7.0
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Topologie hub-and-spoke dans Azure"
[1]: ./images/hub-spoke-gateway-routing.svg "Topologie hub-and-spoke dans Azure avec routage transitif"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Topologie hub-and-spoke dans Azure avec routage transitif utilisant une appliance virtuelle réseau"
[3]: ./images/hub-spokehub-spoke.svg "Topologie hub-and-spoke/hub-and-spoke dans Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
