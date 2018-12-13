---
title: Implémenter une topologie de réseau hub-and-spoke dans Azure
titleSuffix: Azure Reference Architectures
description: Implémentez une topologie de réseau hub-and-spoke dans Azure.
author: telmosampaio
ms.date: 10/08/2018
ms.custom: seodec18
ms.openlocfilehash: fe56630b621f02fe71b864642b75688ba1965862
ms.sourcegitcommit: 8d951fd7e9534054b160be48a1881ae0857561ef
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2018
ms.locfileid: "53329430"
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a><span data-ttu-id="d811a-103">Implémenter une topologie de réseau hub-and-spoke dans Azure</span><span class="sxs-lookup"><span data-stu-id="d811a-103">Implement a hub-spoke network topology in Azure</span></span>

<span data-ttu-id="d811a-104">Cette architecture de référence montre comment implémenter une topologie hub-and-spoke dans Azure.</span><span class="sxs-lookup"><span data-stu-id="d811a-104">This reference architecture shows how to implement a hub-spoke topology in Azure.</span></span> <span data-ttu-id="d811a-105">Le *hub* est un réseau virtuel (VNet) dans Azure qui centralise la connectivité à votre réseau local.</span><span class="sxs-lookup"><span data-stu-id="d811a-105">The *hub* is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="d811a-106">Les *membres spokes* sont des réseaux virtuels appairés avec le hub et qui peuvent être utilisés pour isoler les charges de travail.</span><span class="sxs-lookup"><span data-stu-id="d811a-106">The *spokes* are VNets that peer with the hub, and can be used to isolate workloads.</span></span> <span data-ttu-id="d811a-107">Le trafic circule entre le centre de données local et le hub via une connexion de passerelle ExpressRoute ou VPN.</span><span class="sxs-lookup"><span data-stu-id="d811a-107">Traffic flows between the on-premises datacenter and the hub through an ExpressRoute or VPN gateway connection.</span></span> <span data-ttu-id="d811a-108">[**Déployez cette solution**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="d811a-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="d811a-109">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="d811a-109">![[0]][0]</span></span>

<span data-ttu-id="d811a-110">*Téléchargez un [fichier Visio][visio-download] de cette architecture.*</span><span class="sxs-lookup"><span data-stu-id="d811a-110">*Download a [Visio file][visio-download] of this architecture*</span></span>

<span data-ttu-id="d811a-111">Cette topologie présente les avantages suivants :</span><span class="sxs-lookup"><span data-stu-id="d811a-111">The benefits of this toplogy include:</span></span>

- <span data-ttu-id="d811a-112">**Réduction les coûts** en centralisant les services qui peuvent être partagés par plusieurs charges de travail, comme les appliances virtuelles réseau et les serveurs DNS.</span><span class="sxs-lookup"><span data-stu-id="d811a-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
- <span data-ttu-id="d811a-113">**Surmonter les limites des abonnements** en appairant les réseaux virtuels de différents abonnements avec le hub central.</span><span class="sxs-lookup"><span data-stu-id="d811a-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
- <span data-ttu-id="d811a-114">**Séparation des préoccupations** entre le service informatique central (SecOps, InfraOps) et les charges de travail (DevOps).</span><span class="sxs-lookup"><span data-stu-id="d811a-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="d811a-115">Utilisations courantes de cette architecture :</span><span class="sxs-lookup"><span data-stu-id="d811a-115">Typical uses for this architecture include:</span></span>

- <span data-ttu-id="d811a-116">Charges de travail déployées sur des environnements différents, tels que les environnements de développement, de test et de production, qui requièrent des services partagés tels que DNS, IDS, NTP ou AD DS.</span><span class="sxs-lookup"><span data-stu-id="d811a-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="d811a-117">Les services partagés sont placés dans le réseau virtuel hub, tandis que chaque environnement est déployé sur un membre spoke pour garantir l’isolation.</span><span class="sxs-lookup"><span data-stu-id="d811a-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
- <span data-ttu-id="d811a-118">Charges de travail ne nécessitant pas de connectivité entre elles, mais nécessitant un accès aux services partagés.</span><span class="sxs-lookup"><span data-stu-id="d811a-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
- <span data-ttu-id="d811a-119">Entreprises nécessitant un contrôle centralisé des aspects de la sécurité, tel qu’un pare-feu dans le hub en tant que zone DMZ et une gestion séparée des charges de travail dans chaque membre spoke.</span><span class="sxs-lookup"><span data-stu-id="d811a-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="d811a-120">Architecture</span><span class="sxs-lookup"><span data-stu-id="d811a-120">Architecture</span></span>

<span data-ttu-id="d811a-121">L’architecture est constituée des composants suivants.</span><span class="sxs-lookup"><span data-stu-id="d811a-121">The architecture consists of the following components.</span></span>

- <span data-ttu-id="d811a-122">**Réseau local**.</span><span class="sxs-lookup"><span data-stu-id="d811a-122">**On-premises network**.</span></span> <span data-ttu-id="d811a-123">Un réseau local privé qui s’exécute au sein d’une organisation.</span><span class="sxs-lookup"><span data-stu-id="d811a-123">A private local-area network running within an organization.</span></span>

- <span data-ttu-id="d811a-124">**Périphérique VPN**.</span><span class="sxs-lookup"><span data-stu-id="d811a-124">**VPN device**.</span></span> <span data-ttu-id="d811a-125">Périphérique ou service qui assure la connectivité externe au réseau local.</span><span class="sxs-lookup"><span data-stu-id="d811a-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="d811a-126">Le périphérique VPN peut être un périphérique matériel ou une solution logicielle telle que le service RRAS (Routing and Remote Access Service) dans Windows Server 2012.</span><span class="sxs-lookup"><span data-stu-id="d811a-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="d811a-127">Pour obtenir la liste des périphériques VPN pris en charge et des informations sur la configuration de certains périphériques VPN pour la connexion à Azure, consultez [À propos des périphériques VPN pour les connexions de la passerelle VPN de site à site][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="d811a-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

- <span data-ttu-id="d811a-128">**Passerelle de réseau virtuel VPN ou passerelle ExpressRoute**.</span><span class="sxs-lookup"><span data-stu-id="d811a-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="d811a-129">La passerelle de réseau virtuel permet au réseau virtuel de se connecter au périphérique VPN, ou circuit ExpressRoute, utilisé pour la connectivité avec votre réseau local.</span><span class="sxs-lookup"><span data-stu-id="d811a-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="d811a-130">Pour plus d’informations, consultez [Connecter un réseau local à Microsoft Azure Virtual Network][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="d811a-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="d811a-131">Les scripts de déploiement pour cette architecture de référence utilisent une passerelle VPN pour la connectivité et un réseau virtuel dans Azure pour simuler votre réseau local.</span><span class="sxs-lookup"><span data-stu-id="d811a-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

- <span data-ttu-id="d811a-132">**Réseau virtuel hub**.</span><span class="sxs-lookup"><span data-stu-id="d811a-132">**Hub VNet**.</span></span> <span data-ttu-id="d811a-133">Réseau virtuel Azure utilisé comme hub dans la topologie hub-and-spoke.</span><span class="sxs-lookup"><span data-stu-id="d811a-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="d811a-134">Le hub est le point central de la connectivité à votre réseau local et un emplacement pour héberger les services que peuvent utiliser les différentes charges de travail hébergées dans les réseaux virtuels spokes.</span><span class="sxs-lookup"><span data-stu-id="d811a-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

- <span data-ttu-id="d811a-135">**Sous-réseau de passerelle**.</span><span class="sxs-lookup"><span data-stu-id="d811a-135">**Gateway subnet**.</span></span> <span data-ttu-id="d811a-136">Les passerelles de réseau virtuel sont conservées dans le même sous-réseau.</span><span class="sxs-lookup"><span data-stu-id="d811a-136">The virtual network gateways are held in the same subnet.</span></span>

- <span data-ttu-id="d811a-137">**Réseaux virtuels spokes**.</span><span class="sxs-lookup"><span data-stu-id="d811a-137">**Spoke VNets**.</span></span> <span data-ttu-id="d811a-138">Un ou plusieurs réseaux virtuels Azure qui sont utilisés comme membres spokes dans la topologie hub-and-spoke.</span><span class="sxs-lookup"><span data-stu-id="d811a-138">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="d811a-139">Les membres spokes peuvent servir à isoler les charges de travail dans leurs propres réseaux virtuels, qui sont alors gérées séparément des autres membres spokes.</span><span class="sxs-lookup"><span data-stu-id="d811a-139">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="d811a-140">Chaque charge de travail peut inclure plusieurs niveaux, avec plusieurs sous-réseaux connectés à l’aide d’équilibreurs de charge Azure.</span><span class="sxs-lookup"><span data-stu-id="d811a-140">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="d811a-141">Pour plus d’informations sur l’infrastructure d’application, consultez [Running Windows VM workloads][windows-vm-ra] (Exécution de charges de travail de machine virtuelle Windows) et [Exécution de charges de travail de machine virtuelle Linux][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="d811a-141">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

- <span data-ttu-id="d811a-142">**Appairage de réseaux virtuels**.</span><span class="sxs-lookup"><span data-stu-id="d811a-142">**VNet peering**.</span></span> <span data-ttu-id="d811a-143">Deux réseaux virtuels peuvent être connectés à l’aide d’une [connexion d’appairage][vnet-peering].</span><span class="sxs-lookup"><span data-stu-id="d811a-143">Two VNets can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="d811a-144">Les connexions d’appairage sont des connexions non transitives et à faible latence entre des réseaux virtuels.</span><span class="sxs-lookup"><span data-stu-id="d811a-144">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="d811a-145">Une fois appairés, les réseaux virtuels échangent le trafic à l’aide de la dorsale principale d’Azure, sans avoir besoin d’un routeur.</span><span class="sxs-lookup"><span data-stu-id="d811a-145">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="d811a-146">Dans une topologie de réseau hub-and-spoke, vous utilisez l’appairage de réseaux virtuels pour connecter le hub à chaque membre spoke.</span><span class="sxs-lookup"><span data-stu-id="d811a-146">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span> <span data-ttu-id="d811a-147">Vous pouvez appairer des réseaux virtuels dans la même région ou dans différentes régions.</span><span class="sxs-lookup"><span data-stu-id="d811a-147">You can peer virtual networks in the same region, or different regions.</span></span> <span data-ttu-id="d811a-148">Pour plus d’informations, consultez la [configuration requise et les contraintes][vnet-peering-requirements].</span><span class="sxs-lookup"><span data-stu-id="d811a-148">For more information, see [Requirements and constraints][vnet-peering-requirements].</span></span>

> [!NOTE]
> <span data-ttu-id="d811a-149">Cet article couvre uniquement les déploiements [Resource Manager](/azure/azure-resource-manager/resource-group-overview), mais vous pouvez également connecter un réseau virtuel classique à un réseau virtuel Resource Manager dans un même abonnement.</span><span class="sxs-lookup"><span data-stu-id="d811a-149">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="d811a-150">De cette façon, vos membres spokes peuvent héberger des déploiements classiques tout en tirant parti des services partagés dans le hub.</span><span class="sxs-lookup"><span data-stu-id="d811a-150">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="d811a-151">Recommandations</span><span class="sxs-lookup"><span data-stu-id="d811a-151">Recommendations</span></span>

<span data-ttu-id="d811a-152">Les recommandations suivantes s’appliquent à la plupart des scénarios.</span><span class="sxs-lookup"><span data-stu-id="d811a-152">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="d811a-153">Suivez ces recommandations, sauf si vous avez un besoin spécifique qui vous oblige à les ignorer.</span><span class="sxs-lookup"><span data-stu-id="d811a-153">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="d811a-154">Groupes de ressources</span><span class="sxs-lookup"><span data-stu-id="d811a-154">Resource groups</span></span>

<span data-ttu-id="d811a-155">Le réseau virtuel hub et chaque réseau virtuel spoke peuvent être implémentés dans des groupes de ressources et différentes, voire dans des abonnements différents.</span><span class="sxs-lookup"><span data-stu-id="d811a-155">The hub VNet, and each spoke VNet, can be implemented in different resource groups, and even different subscriptions.</span></span> <span data-ttu-id="d811a-156">Quand vous appairez des réseaux virtuels de différents abonnements, les deux abonnements peuvent être associés au même locataire Azure Active Directory ou à un locataire différent.</span><span class="sxs-lookup"><span data-stu-id="d811a-156">When you peer virtual networks in different subscriptions, both subscriptions can be associated to the same or different Azure Active Directory tenant.</span></span> <span data-ttu-id="d811a-157">Cela permet de décentraliser la gestion de chaque charge de travail, tout en partageant les services gérés dans le réseau virtuel hub.</span><span class="sxs-lookup"><span data-stu-id="d811a-157">This allows for a decentralized management of each workload, while sharing services maintained in the hub VNet.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="d811a-158">Réseau virtuel et sous réseau GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="d811a-158">VNet and GatewaySubnet</span></span>

<span data-ttu-id="d811a-159">Créez un sous-réseau nommé *GatewaySubnet*, avec une plage d’adresses de /27.</span><span class="sxs-lookup"><span data-stu-id="d811a-159">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="d811a-160">La passerelle de réseau virtuel requiert ce sous-réseau.</span><span class="sxs-lookup"><span data-stu-id="d811a-160">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="d811a-161">En allouant 32 adresses à ce sous-réseau, vous devriez éviter les limitations de taille de passerelle.</span><span class="sxs-lookup"><span data-stu-id="d811a-161">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span>

<span data-ttu-id="d811a-162">Pour plus d’informations sur la configuration de la passerelle, consultez les architectures de référence suivantes, en fonction de votre type de connexion :</span><span class="sxs-lookup"><span data-stu-id="d811a-162">For more information about setting up the gateway, see the following reference architectures, depending on your connection type:</span></span>

- <span data-ttu-id="d811a-163">[Réseau hybride utilisant ExpressRoute][guidance-expressroute]</span><span class="sxs-lookup"><span data-stu-id="d811a-163">[Hybrid network using ExpressRoute][guidance-expressroute]</span></span>
- <span data-ttu-id="d811a-164">[Réseau hybride utilisant une passerelle VPN][guidance-vpn]</span><span class="sxs-lookup"><span data-stu-id="d811a-164">[Hybrid network using a VPN gateway][guidance-vpn]</span></span>

<span data-ttu-id="d811a-165">Pour accroître la disponibilité, vous pouvez utiliser ExpressRoute et un VPN pour le basculement.</span><span class="sxs-lookup"><span data-stu-id="d811a-165">For higher availability, you can use ExpressRoute plus a VPN for failover.</span></span> <span data-ttu-id="d811a-166">Consultez [Connecter un réseau local à Azure à l’aide d’ExpressRoute avec basculement VPN][hybrid-ha].</span><span class="sxs-lookup"><span data-stu-id="d811a-166">See [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha].</span></span>

<span data-ttu-id="d811a-167">Une topologie hub-and-spoke peut également être utilisée sans passerelle, si vous n’avez pas besoin de connectivité avec votre réseau local.</span><span class="sxs-lookup"><span data-stu-id="d811a-167">A hub-spoke topology can also be used without a gateway, if you don't need connectivity with your on-premises network.</span></span>

### <a name="vnet-peering"></a><span data-ttu-id="d811a-168">Homologation de réseaux virtuels</span><span class="sxs-lookup"><span data-stu-id="d811a-168">VNet peering</span></span>

<span data-ttu-id="d811a-169">L’appairage de réseaux virtuels est une relation non transitive entre deux réseaux virtuels.</span><span class="sxs-lookup"><span data-stu-id="d811a-169">VNet peering is a non-transitive relationship between two VNets.</span></span> <span data-ttu-id="d811a-170">Si des membres spokes doivent se connecter les uns aux autres, envisagez d’ajouter une connexion d’appairage distincte entre ces membres spokes.</span><span class="sxs-lookup"><span data-stu-id="d811a-170">If you require spokes to connect to each other, consider adding a separate peering connection between those spokes.</span></span>

<span data-ttu-id="d811a-171">Toutefois, cette situation vous prive très rapidement de connexions d’appairage en raison du [nombre maximal d’appairages de réseaux virtuels par réseau virtuel][vnet-peering-limit].</span><span class="sxs-lookup"><span data-stu-id="d811a-171">However, if you have several spokes that need to connect with each other, you will run out of possible peering connections very quickly due to the [limitation on number of VNets peerings per VNet][vnet-peering-limit].</span></span> <span data-ttu-id="d811a-172">Dans ce scénario, envisagez d’utiliser des itinéraires définis par l’utilisateur afin que le trafic destiné à un membre spoke soit envoyé à une appliance virtuelle réseau faisant office de routeur sur le réseau virtuel hub.</span><span class="sxs-lookup"><span data-stu-id="d811a-172">In this scenario, consider using user defined routes (UDRs) to force traffic destined to a spoke to be sent to an NVA acting as a router at the hub VNet.</span></span> <span data-ttu-id="d811a-173">Ainsi, les membres spokes peuvent se connecter les uns aux autres.</span><span class="sxs-lookup"><span data-stu-id="d811a-173">This will allow the spokes to connect to each other.</span></span>

<span data-ttu-id="d811a-174">Vous pouvez également configurer les membres spokes afin qu’ils utilisent la passerelle de réseau virtuel hub pour communiquer avec des réseaux distants.</span><span class="sxs-lookup"><span data-stu-id="d811a-174">You can also configure spokes to use the hub VNet gateway to communicate with remote networks.</span></span> <span data-ttu-id="d811a-175">Pour que le trafic de la passerelle circule entre le membre spoke et le hub, puis se connecte à des réseaux distants, vous devez effectuer les opérations suivantes :</span><span class="sxs-lookup"><span data-stu-id="d811a-175">To allow gateway traffic to flow from spoke to hub, and connect to remote networks, you must:</span></span>

- <span data-ttu-id="d811a-176">Configurer la connexion d’appairage de réseaux virtuels dans le hub pour **autoriser le transit par passerelle**.</span><span class="sxs-lookup"><span data-stu-id="d811a-176">Configure the VNet peering connection in the hub to **allow gateway transit**.</span></span>
- <span data-ttu-id="d811a-177">Configurer la connexion d’appairage de réseaux virtuels dans chaque membre spoke pour **utiliser des passerelles distantes**.</span><span class="sxs-lookup"><span data-stu-id="d811a-177">Configure the VNet peering connection in each spoke to **use remote gateways**.</span></span>
- <span data-ttu-id="d811a-178">Configurer toutes les connexions d’appairages de réseaux virtuels pour **autoriser le trafic transféré**.</span><span class="sxs-lookup"><span data-stu-id="d811a-178">Configure all VNet peering connections to **allow forwarded traffic**.</span></span>

## <a name="considerations"></a><span data-ttu-id="d811a-179">Considérations</span><span class="sxs-lookup"><span data-stu-id="d811a-179">Considerations</span></span>

### <a name="spoke-connectivity"></a><span data-ttu-id="d811a-180">Connectivité entre membres spokes</span><span class="sxs-lookup"><span data-stu-id="d811a-180">Spoke connectivity</span></span>

<span data-ttu-id="d811a-181">Si vous avez besoin de connectivité entre des membres spokes, envisagez d’implémenter une appliance virtuelle réseau pour le routage dans le hub et d’utiliser des itinéraires définis par l’utilisateur dans le membre spoke pour transférer le trafic vers le hub.</span><span class="sxs-lookup"><span data-stu-id="d811a-181">If you require connectivity between spokes, consider implementing an NVA for routing in the hub, and using UDRs in the spoke to forward traffic to the hub.</span></span>

<span data-ttu-id="d811a-182">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="d811a-182">![[2]][2]</span></span>

<span data-ttu-id="d811a-183">Dans ce scénario, vous devez configurer les connexions d’appairage pour **autoriser le trafic transféré**.</span><span class="sxs-lookup"><span data-stu-id="d811a-183">In this scenario, you must configure the peering connections to **allow forwarded traffic**.</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="d811a-184">Surmonter les limites d’appairage de réseaux virtuels</span><span class="sxs-lookup"><span data-stu-id="d811a-184">Overcoming VNet peering limits</span></span>

<span data-ttu-id="d811a-185">Veillez à prendre en compte le [nombre maximal d’appairages de réseaux virtuels par réseau virtuel][vnet-peering-limit] dans Azure.</span><span class="sxs-lookup"><span data-stu-id="d811a-185">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="d811a-186">Si vous décidez que vous avez besoin d’un nombre de membres spokes supérieur à celui autorisé par la limite, envisagez de créer une topologie hub-and-spoke/hub-and-spoke, où les membres spokes du premier niveau de membres spokes font également office de hubs.</span><span class="sxs-lookup"><span data-stu-id="d811a-186">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="d811a-187">Le diagramme qui suit montre cette topologie.</span><span class="sxs-lookup"><span data-stu-id="d811a-187">The following diagram shows this approach.</span></span>

<span data-ttu-id="d811a-188">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="d811a-188">![[3]][3]</span></span>

<span data-ttu-id="d811a-189">Déterminez également les services qui sont partagés dans le hub, afin que ce dernier puisse prendre en charge un plus grand nombre de membres spokes.</span><span class="sxs-lookup"><span data-stu-id="d811a-189">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="d811a-190">Par exemple, si votre hub fournit des services de pare-feu, tenez compte des limites de bande passante de votre solution de pare-feu quand vous ajoutez plusieurs membres spokes.</span><span class="sxs-lookup"><span data-stu-id="d811a-190">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="d811a-191">Vous pourriez souhaiter transférer certains de ces services partagés vers un second niveau de hubs.</span><span class="sxs-lookup"><span data-stu-id="d811a-191">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="d811a-192">Déployer la solution</span><span class="sxs-lookup"><span data-stu-id="d811a-192">Deploy the solution</span></span>

<span data-ttu-id="d811a-193">Un déploiement pour cette architecture est disponible sur [GitHub][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="d811a-193">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="d811a-194">Il utilise des machines virtuelles dans chaque réseau virtuel pour tester la connectivité.</span><span class="sxs-lookup"><span data-stu-id="d811a-194">It uses VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="d811a-195">Aucun service réel n’est hébergé dans le sous-réseau **shared-services** du **réseau virtuel hub**.</span><span class="sxs-lookup"><span data-stu-id="d811a-195">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

<span data-ttu-id="d811a-196">Le déploiement crée les groupes de ressources suivants dans votre abonnement :</span><span class="sxs-lookup"><span data-stu-id="d811a-196">The deployment creates the following resource groups in your subscription:</span></span>

- <span data-ttu-id="d811a-197">hub-nva-rg</span><span class="sxs-lookup"><span data-stu-id="d811a-197">hub-nva-rg</span></span>
- <span data-ttu-id="d811a-198">hub-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="d811a-198">hub-vnet-rg</span></span>
- <span data-ttu-id="d811a-199">onprem-jb-rg</span><span class="sxs-lookup"><span data-stu-id="d811a-199">onprem-jb-rg</span></span>
- <span data-ttu-id="d811a-200">onprem-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="d811a-200">onprem-vnet-rg</span></span>
- <span data-ttu-id="d811a-201">spoke1-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="d811a-201">spoke1-vnet-rg</span></span>
- <span data-ttu-id="d811a-202">spoke2-vent-rg</span><span class="sxs-lookup"><span data-stu-id="d811a-202">spoke2-vent-rg</span></span>

<span data-ttu-id="d811a-203">Les fichiers de paramètre modèle font référence à ces noms. Si vous les modifiez, mettez à jour les fichiers de paramètres afin qu’ils correspondent.</span><span class="sxs-lookup"><span data-stu-id="d811a-203">The template parameter files refer to these names, so if you change them, update the parameter files to match.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="d811a-204">Prérequis</span><span class="sxs-lookup"><span data-stu-id="d811a-204">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="d811a-205">Déployer le centre de données local simulé</span><span class="sxs-lookup"><span data-stu-id="d811a-205">Deploy the simulated on-premises datacenter</span></span>

<span data-ttu-id="d811a-206">Pour déployer le centre de données local simulé en tant que réseau virtuel Azure, procédez comme suit :</span><span class="sxs-lookup"><span data-stu-id="d811a-206">To deploy the simulated on-premises datacenter as an Azure VNet, follow these steps:</span></span>

1. <span data-ttu-id="d811a-207">Accédez au dossier `hybrid-networking/hub-spoke` du référentiel des architectures de référence.</span><span class="sxs-lookup"><span data-stu-id="d811a-207">Navigate to the `hybrid-networking/hub-spoke` folder of the reference architectures repository.</span></span>

2. <span data-ttu-id="d811a-208">Ouvrez le fichier `onprem.json` .</span><span class="sxs-lookup"><span data-stu-id="d811a-208">Open the `onprem.json` file.</span></span> <span data-ttu-id="d811a-209">Remplacez les valeurs de `adminUsername` et `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="d811a-209">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```json
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

3. <span data-ttu-id="d811a-210">Pour un déploiement Linux, définissez `osType` sur `Linux` (facultatif).</span><span class="sxs-lookup"><span data-stu-id="d811a-210">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

4. <span data-ttu-id="d811a-211">Exécutez la commande suivante :</span><span class="sxs-lookup"><span data-stu-id="d811a-211">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
    ```

5. <span data-ttu-id="d811a-212">Attendez que le déploiement se termine.</span><span class="sxs-lookup"><span data-stu-id="d811a-212">Wait for the deployment to finish.</span></span> <span data-ttu-id="d811a-213">Ce déploiement crée un réseau virtuel, une machine virtuelle et une passerelle VPN.</span><span class="sxs-lookup"><span data-stu-id="d811a-213">This deployment creates a virtual network, a virtual machine, and a VPN gateway.</span></span> <span data-ttu-id="d811a-214">La création de la passerelle VPN peut prendre environ 40 minutes.</span><span class="sxs-lookup"><span data-stu-id="d811a-214">It can take about 40 minutes to create the VPN gateway.</span></span>

### <a name="deploy-the-hub-vnet"></a><span data-ttu-id="d811a-215">Déployer le réseau virtuel du hub</span><span class="sxs-lookup"><span data-stu-id="d811a-215">Deploy the hub VNet</span></span>

<span data-ttu-id="d811a-216">Pour déployer réseau virtuel du hub, procédez comme suit.</span><span class="sxs-lookup"><span data-stu-id="d811a-216">To deploy the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="d811a-217">Ouvrez le fichier `hub-vnet.json` .</span><span class="sxs-lookup"><span data-stu-id="d811a-217">Open the `hub-vnet.json` file.</span></span> <span data-ttu-id="d811a-218">Remplacez les valeurs de `adminUsername` et `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="d811a-218">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```json
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="d811a-219">Pour un déploiement Linux, définissez `osType` sur `Linux` (facultatif).</span><span class="sxs-lookup"><span data-stu-id="d811a-219">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

3. <span data-ttu-id="d811a-220">Recherchez les deux instances de `sharedKey`, puis entrez une clé partagée pour la connexion VPN.</span><span class="sxs-lookup"><span data-stu-id="d811a-220">Find both instances of `sharedKey` and enter a shared key for the VPN connection.</span></span> <span data-ttu-id="d811a-221">Les valeurs doivent correspondre.</span><span class="sxs-lookup"><span data-stu-id="d811a-221">The values must match.</span></span>

    ```json
    "sharedKey": "",
    ```

4. <span data-ttu-id="d811a-222">Exécutez la commande suivante :</span><span class="sxs-lookup"><span data-stu-id="d811a-222">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
    ```

5. <span data-ttu-id="d811a-223">Attendez que le déploiement se termine.</span><span class="sxs-lookup"><span data-stu-id="d811a-223">Wait for the deployment to finish.</span></span> <span data-ttu-id="d811a-224">Ce déploiement crée un réseau virtuel, une machine virtuelle, une passerelle VPN et une connexion à la passerelle.</span><span class="sxs-lookup"><span data-stu-id="d811a-224">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway.</span></span>  <span data-ttu-id="d811a-225">La création de la passerelle VPN peut prendre environ 40 minutes.</span><span class="sxs-lookup"><span data-stu-id="d811a-225">It can take about 40 minutes to create the VPN gateway.</span></span>

### <a name="test-connectivity-with-the-hub"></a><span data-ttu-id="d811a-226">Tester la connectivité avec le hub</span><span class="sxs-lookup"><span data-stu-id="d811a-226">Test connectivity with the hub</span></span>

<span data-ttu-id="d811a-227">Testez la connectivité entre l’environnement local simulé et le réseau virtuel du hub.</span><span class="sxs-lookup"><span data-stu-id="d811a-227">Test conectivity from the simulated on-premises environment to the hub VNet.</span></span>

<span data-ttu-id="d811a-228">**Déploiement sous Windows**</span><span class="sxs-lookup"><span data-stu-id="d811a-228">**Windows deployment**</span></span>

1. <span data-ttu-id="d811a-229">Utilisez le portail Azure pour trouver la machine virtuelle appelée `jb-vm1` dans le groupe de ressources `onprem-jb-rg`.</span><span class="sxs-lookup"><span data-stu-id="d811a-229">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="d811a-230">Cliquez sur `Connect` pour ouvrir une session Bureau à distance vers la machine virtuelle.</span><span class="sxs-lookup"><span data-stu-id="d811a-230">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="d811a-231">Utilisez le mot de passe spécifié dans le fichier de paramètre `onprem.json`.</span><span class="sxs-lookup"><span data-stu-id="d811a-231">Use the password that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="d811a-232">Ouvrez une console PowerShell dans la machine virtuelle et utilisez la cmdlet `Test-NetConnection` pour vérifier que vous pouvez vous connecter à la machine virtuelle du serveur de rebond dans le réseau virtuel du hub.</span><span class="sxs-lookup"><span data-stu-id="d811a-232">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the jumpbox VM in the hub VNet.</span></span>

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```

<span data-ttu-id="d811a-233">Le résultat doit être semblable à ce qui suit :</span><span class="sxs-lookup"><span data-stu-id="d811a-233">The output should look similar to the following:</span></span>

```powershell
ComputerName     : 10.0.0.68
RemoteAddress    : 10.0.0.68
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.1.000
TcpTestSucceeded : True
```

> [!NOTE]
> <span data-ttu-id="d811a-234">Par défaut, les machines virtuelles Windows Server n’autorisent pas les réponses ICMP dans Azure.</span><span class="sxs-lookup"><span data-stu-id="d811a-234">By default, Windows Server VMs do not allow ICMP responses in Azure.</span></span> <span data-ttu-id="d811a-235">Si vous souhaitez utiliser `ping` pour tester la connectivité, vous devez activer le trafic ICMP dans le pare-feu Windows avancé et ce, pour chaque machine virtuelle.</span><span class="sxs-lookup"><span data-stu-id="d811a-235">If you want to use `ping` to test connectivity, you need to enable ICMP traffic in the Windows Advanced Firewall for each VM.</span></span>

<span data-ttu-id="d811a-236">**Déploiement sous Linux**</span><span class="sxs-lookup"><span data-stu-id="d811a-236">**Linux deployment**</span></span>

1. <span data-ttu-id="d811a-237">Utilisez le portail Azure pour trouver la machine virtuelle appelée `jb-vm1` dans le groupe de ressources `onprem-jb-rg`.</span><span class="sxs-lookup"><span data-stu-id="d811a-237">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="d811a-238">Cliquez sur `Connect` et copiez la commande `ssh` qui s’affiche dans le portail.</span><span class="sxs-lookup"><span data-stu-id="d811a-238">Click `Connect` and copy the `ssh` command shown in the portal.</span></span> 

3. <span data-ttu-id="d811a-239">Depuis un invite Linux, exécutez `ssh` pour vous connecter à l’environnement local simulé.</span><span class="sxs-lookup"><span data-stu-id="d811a-239">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment.</span></span> <span data-ttu-id="d811a-240">Utilisez le mot de passe spécifié dans le fichier de paramètre `onprem.json`.</span><span class="sxs-lookup"><span data-stu-id="d811a-240">Use the password that you specified in the `onprem.json` parameter file.</span></span>

4. <span data-ttu-id="d811a-241">Utilisez la commande `ping` pour tester la connectivité avec la machine virtuelle du serveur de rebond dans le réseau virtuel du hub :</span><span class="sxs-lookup"><span data-stu-id="d811a-241">Use the `ping` command to test connectivity to the jumpbox VM in the hub VNet:</span></span>

   ```shell
   ping 10.0.0.68
   ```

### <a name="deploy-the-spoke-vnets"></a><span data-ttu-id="d811a-242">Déployer les réseaux virtuels spokes</span><span class="sxs-lookup"><span data-stu-id="d811a-242">Deploy the spoke VNets</span></span>

<span data-ttu-id="d811a-243">Pour déployer les réseaux virtuels spokes, procédez comme suit.</span><span class="sxs-lookup"><span data-stu-id="d811a-243">To deploy the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="d811a-244">Ouvrez le fichier `spoke1.json` .</span><span class="sxs-lookup"><span data-stu-id="d811a-244">Open the `spoke1.json` file.</span></span> <span data-ttu-id="d811a-245">Remplacez les valeurs de `adminUsername` et `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="d811a-245">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```json
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="d811a-246">Pour un déploiement Linux, définissez `osType` sur `Linux` (facultatif).</span><span class="sxs-lookup"><span data-stu-id="d811a-246">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

3. <span data-ttu-id="d811a-247">Exécutez la commande suivante :</span><span class="sxs-lookup"><span data-stu-id="d811a-247">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```

4. <span data-ttu-id="d811a-248">Répétez les étapes 1-2 pour le fichier `spoke2.json`.</span><span class="sxs-lookup"><span data-stu-id="d811a-248">Repeat steps 1-2 for the `spoke2.json` file.</span></span>

5. <span data-ttu-id="d811a-249">Exécutez la commande suivante :</span><span class="sxs-lookup"><span data-stu-id="d811a-249">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

6. <span data-ttu-id="d811a-250">Exécutez la commande suivante :</span><span class="sxs-lookup"><span data-stu-id="d811a-250">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
   ```

### <a name="test-connectivity"></a><span data-ttu-id="d811a-251">Tester la connectivité</span><span class="sxs-lookup"><span data-stu-id="d811a-251">Test connectivity</span></span>

<span data-ttu-id="d811a-252">Testez la connectivité entre l’environnement local simulé et les réseaux virtuels spokes.</span><span class="sxs-lookup"><span data-stu-id="d811a-252">Test conectivity from the simulated on-premises environment to the spoke VNets.</span></span>

<span data-ttu-id="d811a-253">**Déploiement sous Windows**</span><span class="sxs-lookup"><span data-stu-id="d811a-253">**Windows deployment**</span></span>

1. <span data-ttu-id="d811a-254">Utilisez le portail Azure pour trouver la machine virtuelle appelée `jb-vm1` dans le groupe de ressources `onprem-jb-rg`.</span><span class="sxs-lookup"><span data-stu-id="d811a-254">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="d811a-255">Cliquez sur `Connect` pour ouvrir une session Bureau à distance vers la machine virtuelle.</span><span class="sxs-lookup"><span data-stu-id="d811a-255">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="d811a-256">Utilisez le mot de passe spécifié dans le fichier de paramètre `onprem.json`.</span><span class="sxs-lookup"><span data-stu-id="d811a-256">Use the password that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="d811a-257">Ouvrez une console PowerShell dans la machine virtuelle et utilisez l’applet de commande `Test-NetConnection` pour vérifier que vous pouvez vous connecter aux machines virtuelles du serveur de rebond dans les réseaux virtuels spokes.</span><span class="sxs-lookup"><span data-stu-id="d811a-257">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the jumpbox VMs in the spoke VNets.</span></span>

   ```powershell
   Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
   Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
   ```

<span data-ttu-id="d811a-258">**Déploiement sous Linux**</span><span class="sxs-lookup"><span data-stu-id="d811a-258">**Linux deployment**</span></span>

<span data-ttu-id="d811a-259">Pour tester la connectivité entre l’environnement local simulé et le réseau virtuel spoke à l’aide de machines virtuelles Linux, procédez comme suit :</span><span class="sxs-lookup"><span data-stu-id="d811a-259">To test conectivity from the simulated on-premises environment to the spoke VNets using Linux VMs, perform the following steps:</span></span>

1. <span data-ttu-id="d811a-260">Utilisez le portail Azure pour trouver la machine virtuelle appelée `jb-vm1` dans le groupe de ressources `onprem-jb-rg`.</span><span class="sxs-lookup"><span data-stu-id="d811a-260">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="d811a-261">Cliquez sur `Connect` et copiez la commande `ssh` qui s’affiche dans le portail.</span><span class="sxs-lookup"><span data-stu-id="d811a-261">Click `Connect` and copy the `ssh` command shown in the portal.</span></span>

3. <span data-ttu-id="d811a-262">Depuis un invite Linux, exécutez `ssh` pour vous connecter à l’environnement local simulé.</span><span class="sxs-lookup"><span data-stu-id="d811a-262">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment.</span></span> <span data-ttu-id="d811a-263">Utilisez le mot de passe spécifié dans le fichier de paramètre `onprem.json`.</span><span class="sxs-lookup"><span data-stu-id="d811a-263">Use the password that you specified in the `onprem.json` parameter file.</span></span>

4. <span data-ttu-id="d811a-264">Utilisez la commande `ping` pour tester la connectivité avec les machines virtuelles du serveur de rebond dans chaque spoke :</span><span class="sxs-lookup"><span data-stu-id="d811a-264">Use the `ping` command to test connectivity to the jumpbox VMs in each spoke:</span></span>

   ```bash
   ping 10.1.0.68
   ping 10.2.0.68
   ```

### <a name="add-connectivity-between-spokes"></a><span data-ttu-id="d811a-265">Ajouter la connectivité entre les membres spokes</span><span class="sxs-lookup"><span data-stu-id="d811a-265">Add connectivity between spokes</span></span>

<span data-ttu-id="d811a-266">Cette étape est facultative.</span><span class="sxs-lookup"><span data-stu-id="d811a-266">This step is optional.</span></span> <span data-ttu-id="d811a-267">Si vous souhaitez autoriser les spokes à se connecter les uns aux autres, vous devez utiliser une appliance virtuelle réseau en tant que routeur dans le réseau virtuel du hub, et forcer le trafic entre les spokes et le routeur lors de la tentative de connexion à un autre spoke.</span><span class="sxs-lookup"><span data-stu-id="d811a-267">If you want to allow spokes to connect to each other, you must use a network virtual appliance (NVA) as a router in the hub VNet, and force traffic from spokes to the router when trying to connect to another spoke.</span></span> <span data-ttu-id="d811a-268">Pour déployer une appliance virtuelle réseau de base en tant que machine virtuelle unique, ainsi que des itinéraires définis par l’utilisateur pour permettre la connexion des réseaux virtuels spokes, procédez comme suit :</span><span class="sxs-lookup"><span data-stu-id="d811a-268">To deploy a basic sample NVA as a single VM, along with user-defined routes (UDRs) to allow the two spoke VNets to connect, perform the following steps:</span></span>

1. <span data-ttu-id="d811a-269">Ouvrez le fichier `hub-nva.json` .</span><span class="sxs-lookup"><span data-stu-id="d811a-269">Open the `hub-nva.json` file.</span></span> <span data-ttu-id="d811a-270">Remplacez les valeurs de `adminUsername` et `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="d811a-270">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```json
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="d811a-271">Exécutez la commande suivante :</span><span class="sxs-lookup"><span data-stu-id="d811a-271">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

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
[vnet-peering-requirements]: /azure/virtual-network/virtual-network-manage-peering#requirements-and-constraints
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures

[0]: ./images/hub-spoke.png "Topologie hub-and-spoke dans Azure"
[1]: ./images/hub-spoke-gateway-routing.svg "Topologie hub-and-spoke dans Azure avec routage transitif"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Topologie hub-and-spoke dans Azure avec routage transitif utilisant une appliance virtuelle réseau"
[3]: ./images/hub-spokehub-spoke.svg "Topologie hub-and-spoke/hub-and-spoke dans Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
