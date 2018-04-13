---
title: Implémentation d’une topologie de réseau hub-and-spoke dans Azure
description: Comment implémenter une topologie de réseau hub-and-spoke dans Azure.
author: telmosampaio
ms.date: 02/23/2018
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: 243ad026c7c9703d9659cbef6815131fcdaa8a11
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/06/2018
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a><span data-ttu-id="4d047-103">Implémenter une topologie de réseau hub-and-spoke dans Azure</span><span class="sxs-lookup"><span data-stu-id="4d047-103">Implement a hub-spoke network topology in Azure</span></span>

<span data-ttu-id="4d047-104">Cette architecture de référence montre comment implémenter une topologie hub-and-spoke dans Azure.</span><span class="sxs-lookup"><span data-stu-id="4d047-104">This reference architecture shows how to implement a hub-spoke topology in Azure.</span></span> <span data-ttu-id="4d047-105">Le *hub* est un réseau virtuel (VNet) dans Azure qui centralise la connectivité à votre réseau local.</span><span class="sxs-lookup"><span data-stu-id="4d047-105">The *hub* is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="4d047-106">Les *membres spokes* sont des réseaux virtuels appairés avec le hub et qui peuvent être utilisés pour isoler les charges de travail.</span><span class="sxs-lookup"><span data-stu-id="4d047-106">The *spokes* are VNets that peer with the hub, and can be used to isolate workloads.</span></span> <span data-ttu-id="4d047-107">Le trafic circule entre le centre de données local et le hub via une connexion de passerelle ExpressRoute ou VPN.</span><span class="sxs-lookup"><span data-stu-id="4d047-107">Traffic flows between the on-premises datacenter and the hub through an ExpressRoute or VPN gateway connection.</span></span>  <span data-ttu-id="4d047-108">[**Déployez cette solution**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="4d047-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="4d047-109">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="4d047-109">![[0]][0]</span></span>

<span data-ttu-id="4d047-110">*Téléchargez un [fichier Visio][visio-download] de cette architecture.*</span><span class="sxs-lookup"><span data-stu-id="4d047-110">*Download a [Visio file][visio-download] of this architecture*</span></span>


<span data-ttu-id="4d047-111">Cette topologie présente les avantages suivants :</span><span class="sxs-lookup"><span data-stu-id="4d047-111">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="4d047-112">**Réduction les coûts** en centralisant les services qui peuvent être partagés par plusieurs charges de travail, comme les appliances virtuelles réseau et les serveurs DNS.</span><span class="sxs-lookup"><span data-stu-id="4d047-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="4d047-113">**Surmonter les limites des abonnements** en appairant les réseaux virtuels de différents abonnements avec le hub central.</span><span class="sxs-lookup"><span data-stu-id="4d047-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="4d047-114">**Séparation des préoccupations** entre le service informatique central (SecOps, InfraOps) et les charges de travail (DevOps).</span><span class="sxs-lookup"><span data-stu-id="4d047-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="4d047-115">Utilisations courantes de cette architecture :</span><span class="sxs-lookup"><span data-stu-id="4d047-115">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="4d047-116">Charges de travail déployées sur des environnements différents, tels que les environnements de développement, de test et de production, qui requièrent des services partagés tels que DNS, IDS, NTP ou AD DS.</span><span class="sxs-lookup"><span data-stu-id="4d047-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="4d047-117">Les services partagés sont placés dans le réseau virtuel hub, tandis que chaque environnement est déployé sur un membre spoke pour garantir l’isolation.</span><span class="sxs-lookup"><span data-stu-id="4d047-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="4d047-118">Charges de travail ne nécessitant pas de connectivité entre elles, mais nécessitant un accès aux services partagés.</span><span class="sxs-lookup"><span data-stu-id="4d047-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="4d047-119">Entreprises nécessitant un contrôle centralisé des aspects de la sécurité, tel qu’un pare-feu dans le hub en tant que zone DMZ et une gestion séparée des charges de travail dans chaque membre spoke.</span><span class="sxs-lookup"><span data-stu-id="4d047-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="4d047-120">Architecture</span><span class="sxs-lookup"><span data-stu-id="4d047-120">Architecture</span></span>

<span data-ttu-id="4d047-121">L’architecture est constituée des composants suivants.</span><span class="sxs-lookup"><span data-stu-id="4d047-121">The architecture consists of the following components.</span></span>

* <span data-ttu-id="4d047-122">**Réseau local**.</span><span class="sxs-lookup"><span data-stu-id="4d047-122">**On-premises network**.</span></span> <span data-ttu-id="4d047-123">Un réseau local privé qui s’exécute au sein d’une organisation.</span><span class="sxs-lookup"><span data-stu-id="4d047-123">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="4d047-124">**Périphérique VPN**.</span><span class="sxs-lookup"><span data-stu-id="4d047-124">**VPN device**.</span></span> <span data-ttu-id="4d047-125">Périphérique ou service qui assure la connectivité externe au réseau local.</span><span class="sxs-lookup"><span data-stu-id="4d047-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="4d047-126">Le périphérique VPN peut être un périphérique matériel ou une solution logicielle telle que le service RRAS (Routing and Remote Access Service) dans Windows Server 2012.</span><span class="sxs-lookup"><span data-stu-id="4d047-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="4d047-127">Pour obtenir la liste des périphériques VPN pris en charge et des informations sur la configuration de certains périphériques VPN pour la connexion à Azure, consultez [À propos des périphériques VPN pour les connexions de la passerelle VPN de site à site][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="4d047-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="4d047-128">**Passerelle de réseau virtuel VPN ou passerelle ExpressRoute**.</span><span class="sxs-lookup"><span data-stu-id="4d047-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="4d047-129">La passerelle de réseau virtuel permet au réseau virtuel de se connecter au périphérique VPN, ou circuit ExpressRoute, utilisé pour la connectivité avec votre réseau local.</span><span class="sxs-lookup"><span data-stu-id="4d047-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="4d047-130">Pour plus d’informations, consultez [Connecter un réseau local à Microsoft Azure Virtual Network][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="4d047-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="4d047-131">Les scripts de déploiement pour cette architecture de référence utilisent une passerelle VPN pour la connectivité et un réseau virtuel dans Azure pour simuler votre réseau local.</span><span class="sxs-lookup"><span data-stu-id="4d047-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="4d047-132">**Réseau virtuel hub**.</span><span class="sxs-lookup"><span data-stu-id="4d047-132">**Hub VNet**.</span></span> <span data-ttu-id="4d047-133">Réseau virtuel Azure utilisé comme hub dans la topologie hub-and-spoke.</span><span class="sxs-lookup"><span data-stu-id="4d047-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="4d047-134">Le hub est le point central de la connectivité à votre réseau local et un emplacement pour héberger les services que peuvent utiliser les différentes charges de travail hébergées dans les réseaux virtuels spokes.</span><span class="sxs-lookup"><span data-stu-id="4d047-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="4d047-135">**Sous-réseau de passerelle**.</span><span class="sxs-lookup"><span data-stu-id="4d047-135">**Gateway subnet**.</span></span> <span data-ttu-id="4d047-136">Les passerelles de réseau virtuel sont conservées dans le même sous-réseau.</span><span class="sxs-lookup"><span data-stu-id="4d047-136">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="4d047-137">**Réseaux virtuels spokes**.</span><span class="sxs-lookup"><span data-stu-id="4d047-137">**Spoke VNets**.</span></span> <span data-ttu-id="4d047-138">Un ou plusieurs réseaux virtuels Azure qui sont utilisés comme membres spokes dans la topologie hub-and-spoke.</span><span class="sxs-lookup"><span data-stu-id="4d047-138">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="4d047-139">Les membres spokes peuvent servir à isoler les charges de travail dans leurs propres réseaux virtuels, qui sont alors gérées séparément des autres membres spokes.</span><span class="sxs-lookup"><span data-stu-id="4d047-139">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="4d047-140">Chaque charge de travail peut inclure plusieurs niveaux, avec plusieurs sous-réseaux connectés à l’aide d’équilibreurs de charge Azure.</span><span class="sxs-lookup"><span data-stu-id="4d047-140">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="4d047-141">Pour plus d’informations sur l’infrastructure d’application, consultez [Running Windows VM workloads][windows-vm-ra] (Exécution de charges de travail de machine virtuelle Windows) et [Exécution de charges de travail de machine virtuelle Linux][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="4d047-141">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="4d047-142">**Appairage de réseaux virtuels**.</span><span class="sxs-lookup"><span data-stu-id="4d047-142">**VNet peering**.</span></span> <span data-ttu-id="4d047-143">Deux réseaux virtuels dans la même région Azure peuvent être connectés à l’aide d’une [connexion d’appairage][vnet-peering].</span><span class="sxs-lookup"><span data-stu-id="4d047-143">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="4d047-144">Les connexions d’appairage sont des connexions non transitives et à faible latence entre des réseaux virtuels.</span><span class="sxs-lookup"><span data-stu-id="4d047-144">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="4d047-145">Une fois appairés, les réseaux virtuels échangent le trafic à l’aide de la dorsale principale d’Azure, sans avoir besoin d’un routeur.</span><span class="sxs-lookup"><span data-stu-id="4d047-145">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="4d047-146">Dans une topologie de réseau hub-and-spoke, vous utilisez l’appairage de réseaux virtuels pour connecter le hub à chaque membre spoke.</span><span class="sxs-lookup"><span data-stu-id="4d047-146">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="4d047-147">Cet article couvre uniquement les déploiements [Resource Manager](/azure/azure-resource-manager/resource-group-overview), mais vous pouvez également connecter un réseau virtuel classique à un réseau virtuel Resource Manager dans un même abonnement.</span><span class="sxs-lookup"><span data-stu-id="4d047-147">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="4d047-148">De cette façon, vos membres spokes peuvent héberger des déploiements classiques tout en tirant parti des services partagés dans le hub.</span><span class="sxs-lookup"><span data-stu-id="4d047-148">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="4d047-149">Recommandations</span><span class="sxs-lookup"><span data-stu-id="4d047-149">Recommendations</span></span>

<span data-ttu-id="4d047-150">Les recommandations suivantes s’appliquent à la plupart des scénarios.</span><span class="sxs-lookup"><span data-stu-id="4d047-150">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="4d047-151">Suivez ces recommandations, sauf si vous avez un besoin spécifique qui vous oblige à les ignorer.</span><span class="sxs-lookup"><span data-stu-id="4d047-151">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="4d047-152">Groupes de ressources</span><span class="sxs-lookup"><span data-stu-id="4d047-152">Resource groups</span></span>

<span data-ttu-id="4d047-153">Le réseau virtuel hub et chaque réseau virtuel spoke peuvent être implémentés dans différents groupes de ressources, voire dans différents abonnements, tant qu’ils appartiennent au même locataire Azure Active Directory (Azure AD) dans la même région Azure.</span><span class="sxs-lookup"><span data-stu-id="4d047-153">The hub VNet, and each spoke VNet, can be implemented in different resource groups, and even different subscriptions, as long as they belong to the same Azure Active Directory (Azure AD) tenant in the same Azure region.</span></span> <span data-ttu-id="4d047-154">Cela permet de décentraliser la gestion de chaque charge de travail, tout en partageant les services gérés dans le réseau virtuel hub.</span><span class="sxs-lookup"><span data-stu-id="4d047-154">This allows for a decentralized management of each workload, while sharing services maintained in the hub VNet.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="4d047-155">Réseau virtuel et sous réseau GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="4d047-155">VNet and GatewaySubnet</span></span>

<span data-ttu-id="4d047-156">Créez un sous-réseau nommé *GatewaySubnet*, avec une plage d’adresses de /27.</span><span class="sxs-lookup"><span data-stu-id="4d047-156">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="4d047-157">La passerelle de réseau virtuel requiert ce sous-réseau.</span><span class="sxs-lookup"><span data-stu-id="4d047-157">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="4d047-158">En allouant 32 adresses à ce sous-réseau, vous devriez éviter les limitations de taille de passerelle.</span><span class="sxs-lookup"><span data-stu-id="4d047-158">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span>

<span data-ttu-id="4d047-159">Pour plus d’informations sur la configuration de la passerelle, consultez les architectures de référence suivantes, en fonction de votre type de connexion :</span><span class="sxs-lookup"><span data-stu-id="4d047-159">For more information about setting up the gateway, see the following reference architectures, depending on your connection type:</span></span>

- <span data-ttu-id="4d047-160">[Réseau hybride utilisant ExpressRoute][guidance-expressroute]</span><span class="sxs-lookup"><span data-stu-id="4d047-160">[Hybrid network using ExpressRoute][guidance-expressroute]</span></span>
- <span data-ttu-id="4d047-161">[Réseau hybride utilisant une passerelle VPN][guidance-vpn]</span><span class="sxs-lookup"><span data-stu-id="4d047-161">[Hybrid network using a VPN gateway][guidance-vpn]</span></span>

<span data-ttu-id="4d047-162">Pour accroître la disponibilité, vous pouvez utiliser ExpressRoute et un VPN pour le basculement.</span><span class="sxs-lookup"><span data-stu-id="4d047-162">For higher availability, you can use ExpressRoute plus a VPN for failover.</span></span> <span data-ttu-id="4d047-163">Consultez [Connecter un réseau local à Azure à l’aide d’ExpressRoute avec basculement VPN][hybrid-ha].</span><span class="sxs-lookup"><span data-stu-id="4d047-163">See [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha].</span></span>

<span data-ttu-id="4d047-164">Une topologie hub-and-spoke peut également être utilisée sans passerelle, si vous n’avez pas besoin de connectivité avec votre réseau local.</span><span class="sxs-lookup"><span data-stu-id="4d047-164">A hub-spoke topology can also be used without a gateway, if you don't need connectivity with your on-premises network.</span></span> 

### <a name="vnet-peering"></a><span data-ttu-id="4d047-165">Homologation de réseaux virtuels</span><span class="sxs-lookup"><span data-stu-id="4d047-165">VNet peering</span></span>

<span data-ttu-id="4d047-166">L’appairage de réseaux virtuels est une relation non transitive entre deux réseaux virtuels.</span><span class="sxs-lookup"><span data-stu-id="4d047-166">VNet peering is a non-transitive relationship between two VNets.</span></span> <span data-ttu-id="4d047-167">Si des membres spokes doivent se connecter les uns aux autres, envisagez d’ajouter une connexion d’appairage distincte entre ces membres spokes.</span><span class="sxs-lookup"><span data-stu-id="4d047-167">If you require spokes to connect to each other, consider adding a separate peering connection between those spokes.</span></span>

<span data-ttu-id="4d047-168">Toutefois, cette situation vous prive très rapidement de connexions d’appairage en raison du [nombre maximal d’appairages de réseaux virtuels par réseau virtuel][vnet-peering-limit].</span><span class="sxs-lookup"><span data-stu-id="4d047-168">However, if you have several spokes that need to connect with each other, you will run out of possible peering connections very quickly due to the [limitation on number of VNets peerings per VNet][vnet-peering-limit].</span></span> <span data-ttu-id="4d047-169">Dans ce scénario, envisagez d’utiliser des itinéraires définis par l’utilisateur afin que le trafic destiné à un membre spoke soit envoyé à une appliance virtuelle réseau faisant office de routeur sur le réseau virtuel hub.</span><span class="sxs-lookup"><span data-stu-id="4d047-169">In this scenario, consider using user defined routes (UDRs) to force traffic destined to a spoke to be sent to an NVA acting as a router at the hub VNet.</span></span> <span data-ttu-id="4d047-170">Ainsi, les membres spokes peuvent se connecter les uns aux autres.</span><span class="sxs-lookup"><span data-stu-id="4d047-170">This will allow the spokes to connect to each other.</span></span>

<span data-ttu-id="4d047-171">Vous pouvez également configurer les membres spokes afin qu’ils utilisent la passerelle de réseau virtuel hub pour communiquer avec des réseaux distants.</span><span class="sxs-lookup"><span data-stu-id="4d047-171">You can also configure spokes to use the hub VNet gateway to communicate with remote networks.</span></span> <span data-ttu-id="4d047-172">Pour que le trafic de la passerelle circule entre le membre spoke et le hub, puis se connecte à des réseaux distants, vous devez effectuer les opérations suivantes :</span><span class="sxs-lookup"><span data-stu-id="4d047-172">To allow gateway traffic to flow from spoke to hub, and connect to remote networks, you must:</span></span>

  - <span data-ttu-id="4d047-173">Configurer la connexion d’appairage de réseaux virtuels dans le hub pour **autoriser le transit par passerelle**.</span><span class="sxs-lookup"><span data-stu-id="4d047-173">Configure the VNet peering connection in the hub to **allow gateway transit**.</span></span>
  - <span data-ttu-id="4d047-174">Configurer la connexion d’appairage de réseaux virtuels dans chaque membre spoke pour **utiliser des passerelles distantes**.</span><span class="sxs-lookup"><span data-stu-id="4d047-174">Configure the VNet peering connection in each spoke to **use remote gateways**.</span></span>
  - <span data-ttu-id="4d047-175">Configurer toutes les connexions d’appairages de réseaux virtuels pour **autoriser le trafic transféré**.</span><span class="sxs-lookup"><span data-stu-id="4d047-175">Configure all VNet peering connections to **allow forwarded traffic**.</span></span>

## <a name="considerations"></a><span data-ttu-id="4d047-176">Considérations</span><span class="sxs-lookup"><span data-stu-id="4d047-176">Considerations</span></span>

### <a name="spoke-connectivity"></a><span data-ttu-id="4d047-177">Connectivité entre membres spokes</span><span class="sxs-lookup"><span data-stu-id="4d047-177">Spoke connectivity</span></span>

<span data-ttu-id="4d047-178">Si vous avez besoin de connectivité entre des membres spokes, envisagez d’implémenter une appliance virtuelle réseau pour le routage dans le hub et d’utiliser des itinéraires définis par l’utilisateur dans le membre spoke pour transférer le trafic vers le hub.</span><span class="sxs-lookup"><span data-stu-id="4d047-178">If you require connectivity between spokes, consider implementing an NVA for routing in the hub, and using UDRs in the spoke to forward traffic to the hub.</span></span>

<span data-ttu-id="4d047-179">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="4d047-179">![[2]][2]</span></span>

<span data-ttu-id="4d047-180">Dans ce scénario, vous devez configurer les connexions d’appairage pour **autoriser le trafic transféré**.</span><span class="sxs-lookup"><span data-stu-id="4d047-180">In this scenario, you must configure the peering connections to **allow forwarded traffic**.</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="4d047-181">Surmonter les limites d’appairage de réseaux virtuels</span><span class="sxs-lookup"><span data-stu-id="4d047-181">Overcoming VNet peering limits</span></span>

<span data-ttu-id="4d047-182">Veillez à prendre en compte le [nombre maximal d’appairages de réseaux virtuels par réseau virtuel][vnet-peering-limit] dans Azure.</span><span class="sxs-lookup"><span data-stu-id="4d047-182">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="4d047-183">Si vous décidez que vous avez besoin d’un nombre de membres spokes supérieur à celui autorisé par la limite, envisagez de créer une topologie hub-and-spoke/hub-and-spoke, où les membres spokes du premier niveau de membres spokes font également office de hubs.</span><span class="sxs-lookup"><span data-stu-id="4d047-183">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="4d047-184">Le diagramme qui suit montre cette topologie.</span><span class="sxs-lookup"><span data-stu-id="4d047-184">The following diagram shows this approach.</span></span>

<span data-ttu-id="4d047-185">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="4d047-185">![[3]][3]</span></span>

<span data-ttu-id="4d047-186">Déterminez également les services qui sont partagés dans le hub, afin que ce dernier puisse prendre en charge un plus grand nombre de membres spokes.</span><span class="sxs-lookup"><span data-stu-id="4d047-186">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="4d047-187">Par exemple, si votre hub fournit des services de pare-feu, tenez compte des limites de bande passante de votre solution de pare-feu quand vous ajoutez plusieurs membres spokes.</span><span class="sxs-lookup"><span data-stu-id="4d047-187">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="4d047-188">Vous pourriez souhaiter transférer certains de ces services partagés vers un second niveau de hubs.</span><span class="sxs-lookup"><span data-stu-id="4d047-188">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="4d047-189">Déployer la solution</span><span class="sxs-lookup"><span data-stu-id="4d047-189">Deploy the solution</span></span>

<span data-ttu-id="4d047-190">Un déploiement pour cette architecture est disponible sur [GitHub][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="4d047-190">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="4d047-191">Il utilise des machines virtuelles Ubuntu dans chaque réseau virtuel pour tester la connectivité.</span><span class="sxs-lookup"><span data-stu-id="4d047-191">It uses Ubuntu VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="4d047-192">Aucun service réel n’est hébergé dans le sous-réseau **shared-services** du **réseau virtuel hub**.</span><span class="sxs-lookup"><span data-stu-id="4d047-192">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="4d047-193">Prérequis
</span><span class="sxs-lookup"><span data-stu-id="4d047-193">Prerequisites</span></span>

<span data-ttu-id="4d047-194">Avant de pouvoir déployer l’architecture de référence sur votre propre abonnement, vous devez effectuer les étapes suivantes.</span><span class="sxs-lookup"><span data-stu-id="4d047-194">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="4d047-195">Clonez, dupliquez ou téléchargez le fichier zip pour le référentiel GitHub des [architectures de référence][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="4d047-195">Clone, fork, or download the zip file for the [reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="4d047-196">Vérifiez qu’Azure CLI 2.0 est installé sur votre ordinateur.</span><span class="sxs-lookup"><span data-stu-id="4d047-196">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="4d047-197">Pour des instructions d’installation de l’interface de ligne de commande, consultez [Installer Azure CLI 2.0][azure-cli-2].</span><span class="sxs-lookup"><span data-stu-id="4d047-197">For CLI installation instructions, see [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="4d047-198">Installez le package npm des [blocs de construction Azure][azbb].</span><span class="sxs-lookup"><span data-stu-id="4d047-198">Install the [Azure buulding blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="4d047-199">À partir d’une invite de commandes, d’une invite bash ou de l’invite de commandes PowerShell, connectez-vous à votre compte Azure à l’aide de la commande ci-dessous et suivez les invites.</span><span class="sxs-lookup"><span data-stu-id="4d047-199">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using the command below, and follow the prompts.</span></span>

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a><span data-ttu-id="4d047-200">Déployer le centre de données local simulé à l’aide d’azbb</span><span class="sxs-lookup"><span data-stu-id="4d047-200">Deploy the simulated on-premises datacenter using azbb</span></span>

<span data-ttu-id="4d047-201">Pour déployer le centre de données local simulé en tant que réseau virtuel Azure, procédez comme suit :</span><span class="sxs-lookup"><span data-stu-id="4d047-201">To deploy the simulated on-premises datacenter as an Azure VNet, follow these steps:</span></span>

1. <span data-ttu-id="4d047-202">Accédez au dossier `hybrid-networking\hub-spoke\` pour le dépôt que vous avez téléchargé à l’étape des prérequis ci-dessus.</span><span class="sxs-lookup"><span data-stu-id="4d047-202">Navigate to the `hybrid-networking\hub-spoke\` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="4d047-203">Ouvrez le fichier `onprem.json` et entrez un nom d’utilisateur et un mot de passe entre les guillemets aux lignes 36 et 37, comme illustré ci-dessous, puis enregistrez le fichier.</span><span class="sxs-lookup"><span data-stu-id="4d047-203">Open the `onprem.json` file and enter a username and password between the quotes in line 36 and 37, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

3. <span data-ttu-id="4d047-204">À la ligne 38, pour `osType`, saisissez `Windows` ou `Linux` pour installer le système d’exploitation Windows Server 2016 Datacenter ou Ubuntu 16.04 sur le serveur de rebond.</span><span class="sxs-lookup"><span data-stu-id="4d047-204">On line 38, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

4. <span data-ttu-id="4d047-205">Exécutez `azbb` pour déployer l’environnement local simulé, comme indiqué ci-dessous.</span><span class="sxs-lookup"><span data-stu-id="4d047-205">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g onprem-vnet-rg - l <location> -p onoprem.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="4d047-206">Si vous décidez d’utiliser un nom de groupe de ressources différent (autre que `onprem-vnet-rg`), veillez à rechercher tous les fichiers de paramètre qui utilisent ce nom et à les modifier afin qu’ils utilisent votre propre nom de groupe de ressources.</span><span class="sxs-lookup"><span data-stu-id="4d047-206">If you decide to use a different resource group name (other than `onprem-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="4d047-207">Attendez que le déploiement se termine.</span><span class="sxs-lookup"><span data-stu-id="4d047-207">Wait for the deployment to finish.</span></span> <span data-ttu-id="4d047-208">Ce déploiement crée un réseau virtuel, une machine virtuelle et une passerelle VPN.</span><span class="sxs-lookup"><span data-stu-id="4d047-208">This deployment creates a virtual network, a virtual machine, and a VPN gateway.</span></span> <span data-ttu-id="4d047-209">La création de la passerelle VPN peut prendre plus de 40 minutes.</span><span class="sxs-lookup"><span data-stu-id="4d047-209">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="azure-hub-vnet"></a><span data-ttu-id="4d047-210">Réseau virtuel hub Azure</span><span class="sxs-lookup"><span data-stu-id="4d047-210">Azure hub VNet</span></span>

<span data-ttu-id="4d047-211">Pour déployer le réseau virtuel hub et vous connecter au réseau virtuel local simulé créé ci-dessus, effectuez les étapes suivantes.</span><span class="sxs-lookup"><span data-stu-id="4d047-211">To deploy the hub VNet, and connect to the simulated on-premises VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="4d047-212">Ouvrez le fichier `hub-vnet.json` et entrez un nom d’utilisateur et un mot de passe entre les guillemets aux lignes 39 et 40, comme illustré ci-dessous.</span><span class="sxs-lookup"><span data-stu-id="4d047-212">Open the `hub-vnet.json` file and enter a username and password between the quotes in line 39 and 40, as shown below.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

2. <span data-ttu-id="4d047-213">À la ligne 41, pour `osType`, saisissez `Windows` ou `Linux` pour installer le système d’exploitation Windows Server 2016 Datacenter ou Ubuntu 16.04 sur le serveur de rebond.</span><span class="sxs-lookup"><span data-stu-id="4d047-213">On line 41, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="4d047-214">Saisissez une clé partagée entre les guillemets à la ligne 72, comme illustré ci-dessous, puis enregistrez le fichier.</span><span class="sxs-lookup"><span data-stu-id="4d047-214">Enter a shared key between the quotes in line 72, as shown below, then save the file.</span></span>

   ```bash
   "sharedKey": "",
   ```

4. <span data-ttu-id="4d047-215">Exécutez `azbb` pour déployer l’environnement local simulé, comme indiqué ci-dessous.</span><span class="sxs-lookup"><span data-stu-id="4d047-215">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="4d047-216">Si vous décidez d’utiliser un nom de groupe de ressources différent (autre que `hub-vnet-rg`), veillez à rechercher tous les fichiers de paramètre qui utilisent ce nom et à les modifier afin qu’ils utilisent votre propre nom de groupe de ressources.</span><span class="sxs-lookup"><span data-stu-id="4d047-216">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="4d047-217">Attendez que le déploiement se termine.</span><span class="sxs-lookup"><span data-stu-id="4d047-217">Wait for the deployment to finish.</span></span> <span data-ttu-id="4d047-218">Ce déploiement crée un réseau virtuel, une machine virtuelle, une passerelle VPN et une connexion à la passerelle créée à la section précédente.</span><span class="sxs-lookup"><span data-stu-id="4d047-218">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="4d047-219">La création de la passerelle VPN peut prendre plus de 40 minutes.</span><span class="sxs-lookup"><span data-stu-id="4d047-219">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="optional-test-connectivity-from-onprem-to-hub"></a><span data-ttu-id="4d047-220">(Facultatif) Tester la connectivité entre l’instance locale et le hub</span><span class="sxs-lookup"><span data-stu-id="4d047-220">(Optional) Test connectivity from onprem to hub</span></span>

<span data-ttu-id="4d047-221">Pour tester la connectivité entre l’environnement local simulé et le réseau virtuel du hub à l’aide de machines virtuelles Windows, procédez comme suit.</span><span class="sxs-lookup"><span data-stu-id="4d047-221">To test conectivity from the simulated on-premises environment to the hub VNet using Windows VMs, perform the following steps.</span></span>

1. <span data-ttu-id="4d047-222">À partir du portail Azure, accédez au groupe de ressources `onprem-jb-rg`, puis cliquez sur la ressource de machine virtuelle `jb-vm1`.</span><span class="sxs-lookup"><span data-stu-id="4d047-222">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2. <span data-ttu-id="4d047-223">Dans l’angle supérieur gauche du volet de la machine virtuelle, dans le portail, cliquez sur `Connect`, puis suivez les invites afin d’utiliser le Bureau à distance pour vous connecter à la machine virtuelle.</span><span class="sxs-lookup"><span data-stu-id="4d047-223">On the top left hand corner of your VM blade in the portal, click `Connect`, and follow the prompts to use remote desktop to connect to the VM.</span></span> <span data-ttu-id="4d047-224">Assurez-vous d’utiliser le nom d’utilisateur et le mot de passe spécifiés aux lignes 36 et 37 du fichier `onprem.json`.</span><span class="sxs-lookup"><span data-stu-id="4d047-224">Make sure to use the username and password you specified in lines 36 and 37 in the `onprem.json` file.</span></span>

3. <span data-ttu-id="4d047-225">Ouvrez une console PowerShell dans la machine virtuelle et utilisez la cmdlet `Test-NetConnection` pour vérifier que vous pouvez vous connecter à la machine virtuelle du serveur de rebond dans le hub, comme indiqué ci-dessous.</span><span class="sxs-lookup"><span data-stu-id="4d047-225">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the hub jumpbox VM as shown below.</span></span>

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```
   > [!NOTE]
   > <span data-ttu-id="4d047-226">Par défaut, les machines virtuelles Windows Server n’autorisent pas les réponses ICMP dans Azure.</span><span class="sxs-lookup"><span data-stu-id="4d047-226">By default, Windows Server VMs do not allow ICMP responses in Azure.</span></span> <span data-ttu-id="4d047-227">Si vous souhaitez utiliser `ping` pour tester la connectivité, vous devez activer le trafic ICMP dans le pare-feu Windows avancé et ce, pour chaque machine virtuelle.</span><span class="sxs-lookup"><span data-stu-id="4d047-227">If you want to use `ping` to test connectivity, you need to enable ICMP traffic in the Windows Advanced Firewall for each VM.</span></span>

<span data-ttu-id="4d047-228">Pour tester la connectivité entre l’environnement local simulé et le réseau virtuel du hub à l’aide de machines virtuelles Linux, procédez comme suit :</span><span class="sxs-lookup"><span data-stu-id="4d047-228">To test conectivity from the simulated on-premises environment to the hub VNet using Linux VMs, perform the following steps:</span></span>

1. <span data-ttu-id="4d047-229">À partir du portail Azure, accédez au groupe de ressources `onprem-jb-rg`, puis cliquez sur la ressource de machine virtuelle `jb-vm1`.</span><span class="sxs-lookup"><span data-stu-id="4d047-229">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2. <span data-ttu-id="4d047-230">Dans l’angle supérieur gauche du panneau de votre machine virtuelle, dans le portail, cliquez sur `Connect`, puis copiez la commande `ssh` affichée sur le portail.</span><span class="sxs-lookup"><span data-stu-id="4d047-230">On the top left hand corner of your VM blade in the portal, click `Connect`, and then copy the `ssh` command shown on the portal.</span></span> 

3. <span data-ttu-id="4d047-231">À partir d’une invite Linux, exécutez `ssh` pour vous connecter au serveur de rebond de l’environnement local simulé en utilisant les informations copiées à l’étape 2 (ci-dessus), comme indiqué ici.</span><span class="sxs-lookup"><span data-stu-id="4d047-231">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment jumpbox witht the information you copied in step 2 above, as shown below.</span></span>

   ```bash
   ssh <your_user>@<public_ip_address>
   ```

4. <span data-ttu-id="4d047-232">Utilisez le mot de passe que vous avez spécifié à la ligne 37 dans le fichier `onprem.json` pour vous connecter à la machine virtuelle.</span><span class="sxs-lookup"><span data-stu-id="4d047-232">Use the password you specified in line 37 in the `onprem.json` file to the connect to the VM.</span></span>

5. <span data-ttu-id="4d047-233">La commande `ping` vous permet de tester la connectivité au serveur de rebond du hub, comme indiqué ci-dessous.</span><span class="sxs-lookup"><span data-stu-id="4d047-233">Use the `ping` command to test connectivity to the hub jumpbox, as shown below.</span></span>

   ```bash
   ping 10.0.0.68
   ```

### <a name="azure-spoke-vnets"></a><span data-ttu-id="4d047-234">Réseaux virtuels spokes Azure</span><span class="sxs-lookup"><span data-stu-id="4d047-234">Azure spoke VNets</span></span>

<span data-ttu-id="4d047-235">Pour déployer les réseaux virtuels spokes, procédez comme suit.</span><span class="sxs-lookup"><span data-stu-id="4d047-235">To deploy the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="4d047-236">Ouvrez le fichier `spoke1.json` et entrez un nom d’utilisateur et un mot de passe entre les guillemets aux lignes 47 et 48, comme illustré ci-dessous, puis enregistrez le fichier.</span><span class="sxs-lookup"><span data-stu-id="4d047-236">Open the `spoke1.json` file and enter a username and password between the quotes in lines 47 and 48, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

2. <span data-ttu-id="4d047-237">À la ligne 49, pour `osType`, saisissez `Windows` ou `Linux` pour installer le système d’exploitation Windows Server 2016 Datacenter ou Ubuntu 16.04 sur le serveur de rebond.</span><span class="sxs-lookup"><span data-stu-id="4d047-237">On line 49, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="4d047-238">Exécutez `azbb` pour déployer le premier environnement de réseau virtuel spoke, comme indiqué ci-dessous.</span><span class="sxs-lookup"><span data-stu-id="4d047-238">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg - l <location> -p spoke1.json --deploy
   ```
  
   > [!NOTE]
   > <span data-ttu-id="4d047-239">Si vous décidez d’utiliser un nom de groupe de ressources différent (autre que `spoke1-vnet-rg`), veillez à rechercher tous les fichiers de paramètre qui utilisent ce nom et à les modifier afin qu’ils utilisent votre propre nom de groupe de ressources.</span><span class="sxs-lookup"><span data-stu-id="4d047-239">If you decide to use a different resource group name (other than `spoke1-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="4d047-240">Répétez l’étape 1 ci-dessus pour le fichier `spoke2.json`.</span><span class="sxs-lookup"><span data-stu-id="4d047-240">Repeat step 1 above for file `spoke2.json`.</span></span>

5. <span data-ttu-id="4d047-241">Exécutez `azbb` pour déployer le deuxième environnement de réseau virtuel spoke, comme indiqué ci-dessous.</span><span class="sxs-lookup"><span data-stu-id="4d047-241">Run `azbb` to deploy the second spoke VNet environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg - l <location> -p spoke2.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="4d047-242">Si vous décidez d’utiliser un nom de groupe de ressources différent (autre que `spoke2-vnet-rg`), veillez à rechercher tous les fichiers de paramètre qui utilisent ce nom et à les modifier afin qu’ils utilisent votre propre nom de groupe de ressources.</span><span class="sxs-lookup"><span data-stu-id="4d047-242">If you decide to use a different resource group name (other than `spoke2-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a><span data-ttu-id="4d047-243">Appairage de réseaux virtuels hub Azure avec des réseaux virtuels spokes</span><span class="sxs-lookup"><span data-stu-id="4d047-243">Azure hub VNet peering to spoke VNets</span></span>

<span data-ttu-id="4d047-244">Procédez comme suit pour créer une connexion d’homologation entre le réseau virtuel du hub et les réseaux virtuels spokes.</span><span class="sxs-lookup"><span data-stu-id="4d047-244">To create a peering connection from the hub VNet to the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="4d047-245">Ouvrez le fichier `hub-vnet-peering.json` et vérifiez l’exactitude du nom du groupe de ressources et de celui du réseau virtuel de chaque homologation de réseau virtuel, à compter de la ligne 29.</span><span class="sxs-lookup"><span data-stu-id="4d047-245">Open the `hub-vnet-peering.json` file and verify that the resource group name, and virtual network name for each of the virtual network peerings starting in line 29 are correct.</span></span>

2. <span data-ttu-id="4d047-246">Exécutez `azbb` pour déployer le premier environnement de réseau virtuel spoke, comme indiqué ci-dessous.</span><span class="sxs-lookup"><span data-stu-id="4d047-246">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet-peering.json --deploy
   ```

   > [!NOTE]
   > <span data-ttu-id="4d047-247">Si vous décidez d’utiliser un nom de groupe de ressources différent (autre que `hub-vnet-rg`), veillez à rechercher tous les fichiers de paramètre qui utilisent ce nom et à les modifier afin qu’ils utilisent votre propre nom de groupe de ressources.</span><span class="sxs-lookup"><span data-stu-id="4d047-247">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="test-connectivity"></a><span data-ttu-id="4d047-248">Tester la connectivité</span><span class="sxs-lookup"><span data-stu-id="4d047-248">Test connectivity</span></span>

<span data-ttu-id="4d047-249">Pour tester la connectivité entre l’environnement local simulé et le réseau virtuel spoke à l’aide de machines virtuelles Windows, procédez comme suit.</span><span class="sxs-lookup"><span data-stu-id="4d047-249">To test conectivity from the simulated on-premises environment to the spoke VNets using Windows VMs, perform the following steps.</span></span>

1. <span data-ttu-id="4d047-250">À partir du portail Azure, accédez au groupe de ressources `onprem-jb-rg`, puis cliquez sur la ressource de machine virtuelle `jb-vm1`.</span><span class="sxs-lookup"><span data-stu-id="4d047-250">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2. <span data-ttu-id="4d047-251">Dans l’angle supérieur gauche du volet de la machine virtuelle, dans le portail, cliquez sur `Connect`, puis suivez les invites afin d’utiliser le Bureau à distance pour vous connecter à la machine virtuelle.</span><span class="sxs-lookup"><span data-stu-id="4d047-251">On the top left hand corner of your VM blade in the portal, click `Connect`, and follow the prompts to use remote desktop to connect to the VM.</span></span> <span data-ttu-id="4d047-252">Assurez-vous d’utiliser le nom d’utilisateur et le mot de passe spécifiés aux lignes 36 et 37 du fichier `onprem.json`.</span><span class="sxs-lookup"><span data-stu-id="4d047-252">Make sure to use the username and password you specified in lines 36 and 37 in the `onprem.json` file.</span></span>

3. <span data-ttu-id="4d047-253">Ouvrez une console PowerShell dans la machine virtuelle et utilisez la cmdlet `Test-NetConnection` pour vérifier que vous pouvez vous connecter à la machine virtuelle du serveur de rebond dans le hub, comme indiqué ci-dessous.</span><span class="sxs-lookup"><span data-stu-id="4d047-253">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the hub jumpbox VM as shown below.</span></span>

   ```powershell
   Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
   Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
   ```

<span data-ttu-id="4d047-254">Pour tester la connectivité entre l’environnement local simulé et le réseau virtuel spoke à l’aide de machines virtuelles Linux, procédez comme suit :</span><span class="sxs-lookup"><span data-stu-id="4d047-254">To test conectivity from the simulated on-premises environment to the spoke VNets using Linux VMs, perform the following steps:</span></span>

1. <span data-ttu-id="4d047-255">À partir du portail Azure, accédez au groupe de ressources `onprem-jb-rg`, puis cliquez sur la ressource de machine virtuelle `jb-vm1`.</span><span class="sxs-lookup"><span data-stu-id="4d047-255">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2. <span data-ttu-id="4d047-256">Dans l’angle supérieur gauche du panneau de votre machine virtuelle, dans le portail, cliquez sur `Connect`, puis copiez la commande `ssh` affichée sur le portail.</span><span class="sxs-lookup"><span data-stu-id="4d047-256">On the top left hand corner of your VM blade in the portal, click `Connect`, and then copy the `ssh` command shown on the portal.</span></span> 

3. <span data-ttu-id="4d047-257">À partir d’une invite Linux, exécutez `ssh` pour vous connecter au serveur de rebond de l’environnement local simulé en utilisant les informations copiées à l’étape 2 (ci-dessus), comme indiqué ici.</span><span class="sxs-lookup"><span data-stu-id="4d047-257">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment jumpbox witht the information you copied in step 2 above, as shown below.</span></span>

   ```bash
   ssh <your_user>@<public_ip_address>
   ```

4. <span data-ttu-id="4d047-258">Utilisez le mot de passe que vous avez spécifié à la ligne 37 dans le fichier `onprem.json` pour vous connecter à la machine virtuelle.</span><span class="sxs-lookup"><span data-stu-id="4d047-258">Use the password you specified in line 37 in the `onprem.json` file to the connect to the VM.</span></span>

5. <span data-ttu-id="4d047-259">La commande `ping` vous permet de tester la connectivité avec les machines virtuelles du serveur de rebond dans chaque spoke, comme indiqué ci-dessous.</span><span class="sxs-lookup"><span data-stu-id="4d047-259">Use the `ping` command to test connectivity to the jumpbox VMs in each spoke, as shown below.</span></span>

   ```bash
   ping 10.1.0.68
   ping 10.2.0.68
   ```

### <a name="add-connectivity-between-spokes"></a><span data-ttu-id="4d047-260">Ajouter la connectivité entre les membres spokes</span><span class="sxs-lookup"><span data-stu-id="4d047-260">Add connectivity between spokes</span></span>

<span data-ttu-id="4d047-261">Si vous souhaitez autoriser les spokes à se connecter les uns aux autres, vous devez utiliser une appliance virtuelle réseau en tant que routeur dans le réseau virtuel du hub, et forcer le trafic entre les spokes et le routeur lors de la tentative de connexion à un autre spoke.</span><span class="sxs-lookup"><span data-stu-id="4d047-261">If you want to allow spokes to connect to each other, you need to use a newtwork virtual appliance (NVA) as a router in the hub virtual netowrk, and force traffic from spokes to the router when trying to connect to another spoke.</span></span> <span data-ttu-id="4d047-262">Pour déployer une appliance virtuelle réseau de base en tant que machine virtuelle unique, ainsi que les itinéraires définis par l’utilisateur qui sont requis pour permettre la connexion des réseaux virtuels spokes, procédez comme suit :</span><span class="sxs-lookup"><span data-stu-id="4d047-262">To deploy a basic sample NVA as a single VM, and the necessary uder defined routes to allow the two spoke VNets to connect, perform the following steps:</span></span>

1. <span data-ttu-id="4d047-263">Ouvrez le fichier `hub-nva.json` et entrez un nom d’utilisateur et un mot de passe entre les guillemets aux lignes 13 et 14, comme illustré ci-dessous, puis enregistrez le fichier.</span><span class="sxs-lookup"><span data-stu-id="4d047-263">Open the `hub-nva.json` file and enter a username and password between the quotes in lines 13 and 14, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```
2. <span data-ttu-id="4d047-264">Exécutez `azbb` pour déployer la VM de l’appliance virtuelle réseau et les itinéraires définis par l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="4d047-264">Run `azbb` to deploy the NVA VM and user defined routes.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg - l <location> -p hub-nva.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="4d047-265">Si vous décidez d’utiliser un nom de groupe de ressources différent (autre que `hub-nva-rg`), veillez à rechercher tous les fichiers de paramètre qui utilisent ce nom et à les modifier afin qu’ils utilisent votre propre nom de groupe de ressources.</span><span class="sxs-lookup"><span data-stu-id="4d047-265">If you decide to use a different resource group name (other than `hub-nva-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
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

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Topologie hub-and-spoke dans Azure"
[1]: ./images/hub-spoke-gateway-routing.svg "Topologie hub-and-spoke dans Azure avec routage transitif"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Topologie hub-and-spoke dans Azure avec routage transitif utilisant une appliance virtuelle réseau"
[3]: ./images/hub-spokehub-spoke.svg "Topologie hub-and-spoke/hub-and-spoke dans Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
