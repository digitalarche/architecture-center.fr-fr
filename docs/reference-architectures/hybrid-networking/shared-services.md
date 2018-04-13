---
title: Implémentation d’une topologie de réseau hub-and-spoke avec des services partagés dans Azure
description: Procédure d’implémentation d’une topologie de réseau hub-and-spoke avec des services partagés dans Azure.
author: telmosampaio
ms.date: 02/25/2018
pnp.series.title: Implement a hub-spoke network topology with shared services in Azure
pnp.series.prev: hub-spoke
ms.openlocfilehash: b492427f12e026be97629ccdc2b8d19c8c66f47d
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/06/2018
---
# <a name="implement-a-hub-spoke-network-topology-with-shared-services-in-azure"></a><span data-ttu-id="05c49-103">Implémentation d’une topologie de réseau hub-and-spoke avec des services partagés dans Azure</span><span class="sxs-lookup"><span data-stu-id="05c49-103">Implement a hub-spoke network topology with shared services in Azure</span></span>

<span data-ttu-id="05c49-104">Cette architecture de référence s’appuie sur l’architecture de référence [hub-and-spoke][guidance-hub-spoke] de manière à inclure dans le hub des services partagés qui peuvent être utilisés par tous les spokes.</span><span class="sxs-lookup"><span data-stu-id="05c49-104">This reference architecture builds on the [hub-spoke][guidance-hub-spoke] reference architecture to include shared services in the hub that can be consumed by all spokes.</span></span> <span data-ttu-id="05c49-105">Les premiers services que vous devez partager, en tant que première étape de la migration d’un centre de données vers le cloud et la création d’un [centre de données virtuel], sont l’identité et la sécurité.</span><span class="sxs-lookup"><span data-stu-id="05c49-105">As a first step toward migrating a datacenter to the cloud, and building a [virtual datacenter], the first services you need to share are identity and security.</span></span> <span data-ttu-id="05c49-106">Cette architecture de référence vous montre comment étendre vos services Active Directory à partir de votre centre de données local vers Azure, et comment ajouter une appliance virtuelle réseau qui peut jouer le rôle de pare-feu dans une topologie hub-and-spoke.</span><span class="sxs-lookup"><span data-stu-id="05c49-106">This reference archiecture shows you how to extend your Active Directory services from your on-premises datacenter to Azure, and how to add a network virtual appliance (NVA) that can act as a firewall, in a hub-spoke topology.</span></span>  <span data-ttu-id="05c49-107">[**Déployez cette solution**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="05c49-107">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="05c49-108">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="05c49-108">![[0]][0]</span></span>

<span data-ttu-id="05c49-109">*Téléchargez un [fichier Visio][visio-download] de cette architecture.*</span><span class="sxs-lookup"><span data-stu-id="05c49-109">*Download a [Visio file][visio-download] of this architecture*</span></span>

<span data-ttu-id="05c49-110">Cette topologie présente les avantages suivants :</span><span class="sxs-lookup"><span data-stu-id="05c49-110">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="05c49-111">**Réduction les coûts** en centralisant les services qui peuvent être partagés par plusieurs charges de travail, comme les appliances virtuelles réseau et les serveurs DNS.</span><span class="sxs-lookup"><span data-stu-id="05c49-111">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="05c49-112">**Surmonter les limites des abonnements** en appairant les réseaux virtuels de différents abonnements avec le hub central.</span><span class="sxs-lookup"><span data-stu-id="05c49-112">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="05c49-113">**Séparation des préoccupations** entre le service informatique central (SecOps, InfraOps) et les charges de travail (DevOps).</span><span class="sxs-lookup"><span data-stu-id="05c49-113">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="05c49-114">Utilisations courantes de cette architecture :</span><span class="sxs-lookup"><span data-stu-id="05c49-114">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="05c49-115">Charges de travail déployées sur des environnements différents, tels que les environnements de développement, de test et de production, qui requièrent des services partagés tels que DNS, IDS, NTP ou AD DS.</span><span class="sxs-lookup"><span data-stu-id="05c49-115">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="05c49-116">Les services partagés sont placés dans le réseau virtuel hub, tandis que chaque environnement est déployé sur un membre spoke pour garantir l’isolation.</span><span class="sxs-lookup"><span data-stu-id="05c49-116">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="05c49-117">Charges de travail ne nécessitant pas de connectivité entre elles, mais nécessitant un accès aux services partagés.</span><span class="sxs-lookup"><span data-stu-id="05c49-117">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="05c49-118">Entreprises nécessitant un contrôle centralisé des aspects de la sécurité, tel qu’un pare-feu dans le hub en tant que zone DMZ et une gestion séparée des charges de travail dans chaque membre spoke.</span><span class="sxs-lookup"><span data-stu-id="05c49-118">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="05c49-119">Architecture</span><span class="sxs-lookup"><span data-stu-id="05c49-119">Architecture</span></span>

<span data-ttu-id="05c49-120">L’architecture est constituée des composants suivants.</span><span class="sxs-lookup"><span data-stu-id="05c49-120">The architecture consists of the following components.</span></span>

* <span data-ttu-id="05c49-121">**Réseau local**.</span><span class="sxs-lookup"><span data-stu-id="05c49-121">**On-premises network**.</span></span> <span data-ttu-id="05c49-122">Un réseau local privé qui s’exécute au sein d’une organisation.</span><span class="sxs-lookup"><span data-stu-id="05c49-122">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="05c49-123">**Périphérique VPN**.</span><span class="sxs-lookup"><span data-stu-id="05c49-123">**VPN device**.</span></span> <span data-ttu-id="05c49-124">Périphérique ou service qui assure la connectivité externe au réseau local.</span><span class="sxs-lookup"><span data-stu-id="05c49-124">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="05c49-125">Le périphérique VPN peut être un périphérique matériel ou une solution logicielle telle que le service RRAS (Routing and Remote Access Service) dans Windows Server 2012.</span><span class="sxs-lookup"><span data-stu-id="05c49-125">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="05c49-126">Pour obtenir la liste des périphériques VPN pris en charge et des informations sur la configuration de certains périphériques VPN pour la connexion à Azure, consultez [À propos des périphériques VPN pour les connexions de la passerelle VPN de site à site][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="05c49-126">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="05c49-127">**Passerelle de réseau virtuel VPN ou passerelle ExpressRoute**.</span><span class="sxs-lookup"><span data-stu-id="05c49-127">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="05c49-128">La passerelle de réseau virtuel permet au réseau virtuel de se connecter au périphérique VPN, ou circuit ExpressRoute, utilisé pour la connectivité avec votre réseau local.</span><span class="sxs-lookup"><span data-stu-id="05c49-128">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="05c49-129">Pour plus d’informations, consultez [Connecter un réseau local à Microsoft Azure Virtual Network][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="05c49-129">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="05c49-130">Les scripts de déploiement pour cette architecture de référence utilisent une passerelle VPN pour la connectivité et un réseau virtuel dans Azure pour simuler votre réseau local.</span><span class="sxs-lookup"><span data-stu-id="05c49-130">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="05c49-131">**Réseau virtuel hub**.</span><span class="sxs-lookup"><span data-stu-id="05c49-131">**Hub VNet**.</span></span> <span data-ttu-id="05c49-132">Réseau virtuel Azure utilisé comme hub dans la topologie hub-and-spoke.</span><span class="sxs-lookup"><span data-stu-id="05c49-132">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="05c49-133">Le hub est le point central de la connectivité à votre réseau local et un emplacement pour héberger les services que peuvent utiliser les différentes charges de travail hébergées dans les réseaux virtuels spokes.</span><span class="sxs-lookup"><span data-stu-id="05c49-133">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="05c49-134">**Sous-réseau de passerelle**.</span><span class="sxs-lookup"><span data-stu-id="05c49-134">**Gateway subnet**.</span></span> <span data-ttu-id="05c49-135">Les passerelles de réseau virtuel sont conservées dans le même sous-réseau.</span><span class="sxs-lookup"><span data-stu-id="05c49-135">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="05c49-136">**Sous-réseau de services partagés**.</span><span class="sxs-lookup"><span data-stu-id="05c49-136">**Shared services subnet**.</span></span> <span data-ttu-id="05c49-137">Sous-réseau dans le réseau virtuel hub utilisé pour héberger les services que peuvent partager tous les membres spokes, tels que DNS ou AD DS.</span><span class="sxs-lookup"><span data-stu-id="05c49-137">A subnet in the hub VNet used to host services that can be shared among all spokes, such as DNS or AD DS.</span></span>

* <span data-ttu-id="05c49-138">**Sous-réseau de DMZ**.</span><span class="sxs-lookup"><span data-stu-id="05c49-138">**DMZ subnet**.</span></span> <span data-ttu-id="05c49-139">Sous-réseau du réseau virtuel du hub utilisé pour héberger les appliances virtuelles réseau qui peuvent jouer le rôle d’appliances de sécurité, comme les pare-feu.</span><span class="sxs-lookup"><span data-stu-id="05c49-139">A subnet in the hub VNet used to host NVAs that can act as security appliances, such as firewalls.</span></span>

* <span data-ttu-id="05c49-140">**Réseaux virtuels spokes**.</span><span class="sxs-lookup"><span data-stu-id="05c49-140">**Spoke VNets**.</span></span> <span data-ttu-id="05c49-141">Un ou plusieurs réseaux virtuels Azure qui sont utilisés comme membres spokes dans la topologie hub-and-spoke.</span><span class="sxs-lookup"><span data-stu-id="05c49-141">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="05c49-142">Les membres spokes peuvent servir à isoler les charges de travail dans leurs propres réseaux virtuels, qui sont alors gérées séparément des autres membres spokes.</span><span class="sxs-lookup"><span data-stu-id="05c49-142">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="05c49-143">Chaque charge de travail peut inclure plusieurs niveaux, avec plusieurs sous-réseaux connectés à l’aide d’équilibreurs de charge Azure.</span><span class="sxs-lookup"><span data-stu-id="05c49-143">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="05c49-144">Pour plus d’informations sur l’infrastructure d’application, consultez [Running Windows VM workloads][windows-vm-ra] (Exécution de charges de travail de machine virtuelle Windows) et [Exécution de charges de travail de machine virtuelle Linux][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="05c49-144">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="05c49-145">**Appairage de réseaux virtuels**.</span><span class="sxs-lookup"><span data-stu-id="05c49-145">**VNet peering**.</span></span> <span data-ttu-id="05c49-146">Deux réseaux virtuels dans la même région Azure peuvent être connectés à l’aide d’une [connexion d’appairage][vnet-peering].</span><span class="sxs-lookup"><span data-stu-id="05c49-146">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="05c49-147">Les connexions d’appairage sont des connexions non transitives et à faible latence entre des réseaux virtuels.</span><span class="sxs-lookup"><span data-stu-id="05c49-147">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="05c49-148">Une fois appairés, les réseaux virtuels échangent le trafic à l’aide de la dorsale principale d’Azure, sans avoir besoin d’un routeur.</span><span class="sxs-lookup"><span data-stu-id="05c49-148">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="05c49-149">Dans une topologie de réseau hub-and-spoke, vous utilisez l’appairage de réseaux virtuels pour connecter le hub à chaque membre spoke.</span><span class="sxs-lookup"><span data-stu-id="05c49-149">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="05c49-150">Cet article couvre uniquement les déploiements [Resource Manager](/azure/azure-resource-manager/resource-group-overview), mais vous pouvez également connecter un réseau virtuel classique à un réseau virtuel Resource Manager dans un même abonnement.</span><span class="sxs-lookup"><span data-stu-id="05c49-150">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="05c49-151">De cette façon, vos membres spokes peuvent héberger des déploiements classiques tout en tirant parti des services partagés dans le hub.</span><span class="sxs-lookup"><span data-stu-id="05c49-151">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="05c49-152">Recommandations</span><span class="sxs-lookup"><span data-stu-id="05c49-152">Recommendations</span></span>

<span data-ttu-id="05c49-153">Toutes les recommandations de l’architecture de référence [hub-and-spoke][guidance-hub-spoke] concernent également celle des services partagés.</span><span class="sxs-lookup"><span data-stu-id="05c49-153">All the recommendations for the [hub-spoke][guidance-hub-spoke] reference architecture also apply to the shared services reference architecture.</span></span> 

<span data-ttu-id="05c49-154">De plus, les recommandations suivantes s’appliquent à la plupart des scénarios relatifs aux services partagés.</span><span class="sxs-lookup"><span data-stu-id="05c49-154">ALso, the following recommendations apply for most scenarios under shared services.</span></span> <span data-ttu-id="05c49-155">Suivez ces recommandations, sauf si vous avez un besoin spécifique qui vous oblige à les ignorer.</span><span class="sxs-lookup"><span data-stu-id="05c49-155">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="identity"></a><span data-ttu-id="05c49-156">Identité</span><span class="sxs-lookup"><span data-stu-id="05c49-156">Identity</span></span>

<span data-ttu-id="05c49-157">La plupart des organisations incluent un environnement Active Directory Directory Services (AD DS) dans leur centre de données local.</span><span class="sxs-lookup"><span data-stu-id="05c49-157">Most enterprise organizations have an Active Directory Directory Services (ADDS) environment in their on-premises datacenter.</span></span> <span data-ttu-id="05c49-158">Pour faciliter la gestion des ressources déplacées vers Azure à partir de votre réseau local et qui dépendent d’AD DS, il est recommandé d’héberger les contrôleurs de domaine AD DS dans Azure.</span><span class="sxs-lookup"><span data-stu-id="05c49-158">To facilitate management of assets moved to Azure from your on-premises network that depend on ADDS, it is recommended to host ADDS domain controllers in Azure.</span></span>

<span data-ttu-id="05c49-159">Si vous devez utiliser des objets de stratégie de groupe, que vous souhaitez contrôler séparément pour Azure et votre environnement local, utilisez un site Active Directory différent pour chaque région Azure.</span><span class="sxs-lookup"><span data-stu-id="05c49-159">If you make use of Group Policy Objects, that you want to control separately for Azure and your on-premises environment, use a different AD site for each Azure region.</span></span> <span data-ttu-id="05c49-160">Placez vos contrôleurs de domaine dans un réseau virtuel central (hub) auquel les charges de travail dépendantes peuvent accéder.</span><span class="sxs-lookup"><span data-stu-id="05c49-160">Place your domain controllers in a central VNet (hub) that dependent workloads can access.</span></span>

### <a name="security"></a><span data-ttu-id="05c49-161">Sécurité</span><span class="sxs-lookup"><span data-stu-id="05c49-161">Security</span></span>

<span data-ttu-id="05c49-162">Lorsque vous déplacez des charges de travail depuis votre environnement local vers Azure, certaines d’entre elles doivent être hébergées sur des machines virtuelles.</span><span class="sxs-lookup"><span data-stu-id="05c49-162">As you move workloads from your on-premises environment to Azure, some of these workloads will require to be hosted in VMs.</span></span> <span data-ttu-id="05c49-163">Pour des raisons de conformité, vous devrez peut-être appliquer des restrictions sur le trafic qui traverse ces charges de travail.</span><span class="sxs-lookup"><span data-stu-id="05c49-163">For compliance reasons, you may need to enforce restrictions on traffic traversing those workloads.</span></span> 

<span data-ttu-id="05c49-164">Vous pouvez utiliser les appliances virtuelles réseau dans Azure pour héberger différents types de services de gestion de la sécurité et des performances.</span><span class="sxs-lookup"><span data-stu-id="05c49-164">You can use network virtual appliances (NVAs) in Azure to host different types of security and performance services.</span></span> <span data-ttu-id="05c49-165">Si vous êtes familiarisé avec un ensemble donné d’appliances locales, il est recommandé d’utiliser les mêmes appliances virtualisées dans Azure, le cas échéant.</span><span class="sxs-lookup"><span data-stu-id="05c49-165">If you are familiar with a given set of appliances on-premises today, it is recommended to use the same virtualized appliances in Azure, where applicable.</span></span>

> [!NOTE]
> <span data-ttu-id="05c49-166">Les scripts de déploiement de cette architecture de référence utilisent une machine virtuelle Ubuntu avec transfert d’IP activé pour imiter une appliance virtuelle réseau.</span><span class="sxs-lookup"><span data-stu-id="05c49-166">The deployment scripts for this reference architecture use an Ubuntu VM with IP forwarding enbaled to mimic a network virtual appliance.</span></span>

## <a name="considerations"></a><span data-ttu-id="05c49-167">Considérations</span><span class="sxs-lookup"><span data-stu-id="05c49-167">Considerations</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="05c49-168">Surmonter les limites d’appairage de réseaux virtuels</span><span class="sxs-lookup"><span data-stu-id="05c49-168">Overcoming VNet peering limits</span></span>

<span data-ttu-id="05c49-169">Veillez à prendre en compte le [nombre maximal d’appairages de réseaux virtuels par réseau virtuel][vnet-peering-limit] dans Azure.</span><span class="sxs-lookup"><span data-stu-id="05c49-169">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="05c49-170">Si vous décidez que vous avez besoin d’un nombre de membres spokes supérieur à celui autorisé par la limite, envisagez de créer une topologie hub-and-spoke/hub-and-spoke, où les membres spokes du premier niveau de membres spokes font également office de hubs.</span><span class="sxs-lookup"><span data-stu-id="05c49-170">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="05c49-171">Le diagramme qui suit montre cette topologie.</span><span class="sxs-lookup"><span data-stu-id="05c49-171">The following diagram shows this approach.</span></span>

<span data-ttu-id="05c49-172">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="05c49-172">![[3]][3]</span></span>

<span data-ttu-id="05c49-173">Déterminez également les services qui sont partagés dans le hub, afin que ce dernier puisse prendre en charge un plus grand nombre de membres spokes.</span><span class="sxs-lookup"><span data-stu-id="05c49-173">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="05c49-174">Par exemple, si votre hub fournit des services de pare-feu, tenez compte des limites de bande passante de votre solution de pare-feu quand vous ajoutez plusieurs membres spokes.</span><span class="sxs-lookup"><span data-stu-id="05c49-174">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="05c49-175">Vous pourriez souhaiter transférer certains de ces services partagés vers un second niveau de hubs.</span><span class="sxs-lookup"><span data-stu-id="05c49-175">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="05c49-176">Déployer la solution</span><span class="sxs-lookup"><span data-stu-id="05c49-176">Deploy the solution</span></span>

<span data-ttu-id="05c49-177">Un déploiement pour cette architecture est disponible sur [GitHub][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="05c49-177">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="05c49-178">Il utilise des machines virtuelles Ubuntu dans chaque réseau virtuel pour tester la connectivité.</span><span class="sxs-lookup"><span data-stu-id="05c49-178">It uses Ubuntu VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="05c49-179">Aucun service réel n’est hébergé dans le sous-réseau **shared-services** du **réseau virtuel hub**.</span><span class="sxs-lookup"><span data-stu-id="05c49-179">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="05c49-180">Prérequis
</span><span class="sxs-lookup"><span data-stu-id="05c49-180">Prerequisites</span></span>

<span data-ttu-id="05c49-181">Avant de pouvoir déployer l’architecture de référence sur votre propre abonnement, vous devez effectuer les étapes suivantes.</span><span class="sxs-lookup"><span data-stu-id="05c49-181">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="05c49-182">Clonez, dupliquez ou téléchargez le fichier zip pour le référentiel GitHub des [architectures de référence][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="05c49-182">Clone, fork, or download the zip file for the [reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="05c49-183">Vérifiez qu’Azure CLI 2.0 est installé sur votre ordinateur.</span><span class="sxs-lookup"><span data-stu-id="05c49-183">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="05c49-184">Pour des instructions d’installation de l’interface de ligne de commande, consultez [Installer Azure CLI 2.0][azure-cli-2].</span><span class="sxs-lookup"><span data-stu-id="05c49-184">For CLI installation instructions, see [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="05c49-185">Installez le package npm des [modules Azure][azbb].</span><span class="sxs-lookup"><span data-stu-id="05c49-185">Install the [Azure building blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="05c49-186">À partir d’une invite de commandes, d’une invite bash ou de l’invite de commandes PowerShell, connectez-vous à votre compte Azure à l’aide de la commande ci-dessous et suivez les invites.</span><span class="sxs-lookup"><span data-stu-id="05c49-186">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using the command below, and follow the prompts.</span></span>

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a><span data-ttu-id="05c49-187">Déployer le centre de données local simulé à l’aide d’azbb</span><span class="sxs-lookup"><span data-stu-id="05c49-187">Deploy the simulated on-premises datacenter using azbb</span></span>

<span data-ttu-id="05c49-188">Pour déployer le centre de données local simulé en tant que réseau virtuel Azure, procédez comme suit :</span><span class="sxs-lookup"><span data-stu-id="05c49-188">To deploy the simulated on-premises datacenter as an Azure VNet, follow these steps:</span></span>

1. <span data-ttu-id="05c49-189">Accédez au dossier `hybrid-networking\shared-services-stack\` pour le dépôt que vous avez téléchargé à l’étape des prérequis ci-dessus.</span><span class="sxs-lookup"><span data-stu-id="05c49-189">Navigate to the `hybrid-networking\shared-services-stack\` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="05c49-190">Ouvrez le fichier `onprem.json` et entrez un nom d’utilisateur et un mot de passe entre les guillemets aux lignes 45 et 46, comme illustré ci-dessous, puis enregistrez le fichier.</span><span class="sxs-lookup"><span data-stu-id="05c49-190">Open the `onprem.json` file and enter a username and password between the quotes in line 45 and 46, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

3. <span data-ttu-id="05c49-191">Exécutez `azbb` pour déployer l’environnement local simulé, comme indiqué ci-dessous.</span><span class="sxs-lookup"><span data-stu-id="05c49-191">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g onprem-vnet-rg - l <location> -p onoprem.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="05c49-192">Si vous décidez d’utiliser un nom de groupe de ressources différent (autre que `onprem-vnet-rg`), veillez à rechercher tous les fichiers de paramètre qui utilisent ce nom et à les modifier afin qu’ils utilisent votre propre nom de groupe de ressources.</span><span class="sxs-lookup"><span data-stu-id="05c49-192">If you decide to use a different resource group name (other than `onprem-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="05c49-193">Attendez que le déploiement se termine.</span><span class="sxs-lookup"><span data-stu-id="05c49-193">Wait for the deployment to finish.</span></span> <span data-ttu-id="05c49-194">Ce déploiement crée un réseau virtuel, une machine virtuelle exécutant Windows et une passerelle VPN.</span><span class="sxs-lookup"><span data-stu-id="05c49-194">This deployment creates a virtual network, a virtual machine running Windows, and a VPN gateway.</span></span> <span data-ttu-id="05c49-195">La création de la passerelle VPN peut prendre plus de 40 minutes.</span><span class="sxs-lookup"><span data-stu-id="05c49-195">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="azure-hub-vnet"></a><span data-ttu-id="05c49-196">Réseau virtuel hub Azure</span><span class="sxs-lookup"><span data-stu-id="05c49-196">Azure hub VNet</span></span>

<span data-ttu-id="05c49-197">Pour déployer le réseau virtuel hub et vous connecter au réseau virtuel local simulé créé ci-dessus, effectuez les étapes suivantes.</span><span class="sxs-lookup"><span data-stu-id="05c49-197">To deploy the hub VNet, and connect to the simulated on-premises VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="05c49-198">Ouvrez le fichier `hub-vnet.json` et entrez un nom d’utilisateur et un mot de passe entre les guillemets aux lignes 50 et 51, comme illustré ci-dessous.</span><span class="sxs-lookup"><span data-stu-id="05c49-198">Open the `hub-vnet.json` file and enter a username and password between the quotes in line 50 and 51, as shown below.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

2. <span data-ttu-id="05c49-199">À la ligne 52, pour `osType`, saisissez `Windows` ou `Linux` pour installer le système d’exploitation Windows Server 2016 Datacenter ou Ubuntu 16.04 sur le serveur de rebond.</span><span class="sxs-lookup"><span data-stu-id="05c49-199">On line 52, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="05c49-200">Saisissez une clé partagée entre les guillemets à la ligne 83, comme illustré ci-dessous, puis enregistrez le fichier.</span><span class="sxs-lookup"><span data-stu-id="05c49-200">Enter a shared key between the quotes in line 83, as shown below, then save the file.</span></span>

   ```bash
   "sharedKey": "",
   ```

4. <span data-ttu-id="05c49-201">Exécutez `azbb` pour déployer l’environnement local simulé, comme indiqué ci-dessous.</span><span class="sxs-lookup"><span data-stu-id="05c49-201">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="05c49-202">Si vous décidez d’utiliser un nom de groupe de ressources différent (autre que `hub-vnet-rg`), veillez à rechercher tous les fichiers de paramètre qui utilisent ce nom et à les modifier afin qu’ils utilisent votre propre nom de groupe de ressources.</span><span class="sxs-lookup"><span data-stu-id="05c49-202">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="05c49-203">Attendez que le déploiement se termine.</span><span class="sxs-lookup"><span data-stu-id="05c49-203">Wait for the deployment to finish.</span></span> <span data-ttu-id="05c49-204">Ce déploiement crée un réseau virtuel, une machine virtuelle, une passerelle VPN et une connexion à la passerelle créée à la section précédente.</span><span class="sxs-lookup"><span data-stu-id="05c49-204">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="05c49-205">La création de la passerelle VPN peut prendre plus de 40 minutes.</span><span class="sxs-lookup"><span data-stu-id="05c49-205">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="adds-in-azure"></a><span data-ttu-id="05c49-206">ADDS dans Azure</span><span class="sxs-lookup"><span data-stu-id="05c49-206">ADDS in Azure</span></span>

<span data-ttu-id="05c49-207">Pour déployer des contrôleurs de domaine AD DS dans Azure, procédez comme suit.</span><span class="sxs-lookup"><span data-stu-id="05c49-207">To deploy the ADDS domain controllers in Azure, perform the following steps.</span></span>

1. <span data-ttu-id="05c49-208">Ouvrez le fichier `hub-adds.json` et entrez un nom d’utilisateur et un mot de passe entre les guillemets aux lignes 14 et 15, comme illustré ci-dessous, puis enregistrez le fichier.</span><span class="sxs-lookup"><span data-stu-id="05c49-208">Open the `hub-adds.json` file and enter a username and password between the quotes in lines 14 and 15, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

2. <span data-ttu-id="05c49-209">Exécutez `azbb` pour déployer les contrôleurs de domaine AD DS, comme indiqué ci-dessous.</span><span class="sxs-lookup"><span data-stu-id="05c49-209">Run `azbb` to deploy the ADDS domain controllers as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-adds-rg - l <location> -p hub-adds.json --deploy
   ```
  
   > [!NOTE]
   > <span data-ttu-id="05c49-210">Si vous décidez d’utiliser un nom de groupe de ressources différent (autre que `hub-adds-rg`), veillez à rechercher tous les fichiers de paramètre qui utilisent ce nom et à les modifier afin qu’ils utilisent votre propre nom de groupe de ressources.</span><span class="sxs-lookup"><span data-stu-id="05c49-210">If you decide to use a different resource group name (other than `hub-adds-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

   > [!NOTE]
   > <span data-ttu-id="05c49-211">Cette partie du déploiement peut prendre plusieurs minutes, car elle requiert la jonction de deux machines virtuelles au domaine hébergé dans le centre de données local simulé, puis l’installation d’AD DS sur ces dernières.</span><span class="sxs-lookup"><span data-stu-id="05c49-211">This part of the deployment may take several minutes, since it requires joining the two VMs to the domain hosted int he simulated on-premises datacenter, then installing AD DS on them.</span></span>

### <a name="nva"></a><span data-ttu-id="05c49-212">Appliances virtuelles réseau</span><span class="sxs-lookup"><span data-stu-id="05c49-212">NVA</span></span>

<span data-ttu-id="05c49-213">Pour déployer une appliance virtuelle réseau dans le sous-réseau `dmz`, effectuez les opérations suivantes :</span><span class="sxs-lookup"><span data-stu-id="05c49-213">To deploy an NVA in the `dmz` subnet, perform the following steps:</span></span>

1. <span data-ttu-id="05c49-214">Ouvrez le fichier `hub-nva.json` et entrez un nom d’utilisateur et un mot de passe entre les guillemets aux lignes 13 et 14, comme illustré ci-dessous, puis enregistrez le fichier.</span><span class="sxs-lookup"><span data-stu-id="05c49-214">Open the `hub-nva.json` file and enter a username and password between the quotes in lines 13 and 14, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```
2. <span data-ttu-id="05c49-215">Exécutez `azbb` pour déployer la VM de l’appliance virtuelle réseau et les itinéraires définis par l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="05c49-215">Run `azbb` to deploy the NVA VM and user defined routes.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg - l <location> -p hub-nva.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="05c49-216">Si vous décidez d’utiliser un nom de groupe de ressources différent (autre que `hub-nva-rg`), veillez à rechercher tous les fichiers de paramètre qui utilisent ce nom et à les modifier afin qu’ils utilisent votre propre nom de groupe de ressources.</span><span class="sxs-lookup"><span data-stu-id="05c49-216">If you decide to use a different resource group name (other than `hub-nva-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="azure-spoke-vnets"></a><span data-ttu-id="05c49-217">Réseaux virtuels spokes Azure</span><span class="sxs-lookup"><span data-stu-id="05c49-217">Azure spoke VNets</span></span>

<span data-ttu-id="05c49-218">Pour déployer les réseaux virtuels spokes, procédez comme suit.</span><span class="sxs-lookup"><span data-stu-id="05c49-218">To deploy the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="05c49-219">Ouvrez le fichier `spoke1.json` et entrez un nom d’utilisateur et un mot de passe entre les guillemets aux lignes 52 et 53, comme illustré ci-dessous, puis enregistrez le fichier.</span><span class="sxs-lookup"><span data-stu-id="05c49-219">Open the `spoke1.json` file and enter a username and password between the quotes in lines 52 and 53, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

2. <span data-ttu-id="05c49-220">À la ligne 54, pour `osType`, saisissez `Windows` ou `Linux` pour installer le système d’exploitation Windows Server 2016 Datacenter ou Ubuntu 16.04 sur le serveur de rebond.</span><span class="sxs-lookup"><span data-stu-id="05c49-220">On line 54, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="05c49-221">Exécutez `azbb` pour déployer le premier environnement de réseau virtuel spoke, comme indiqué ci-dessous.</span><span class="sxs-lookup"><span data-stu-id="05c49-221">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg - l <location> -p spoke1.json --deploy
   ```
  
   > [!NOTE]
   > <span data-ttu-id="05c49-222">Si vous décidez d’utiliser un nom de groupe de ressources différent (autre que `spoke1-vnet-rg`), veillez à rechercher tous les fichiers de paramètre qui utilisent ce nom et à les modifier afin qu’ils utilisent votre propre nom de groupe de ressources.</span><span class="sxs-lookup"><span data-stu-id="05c49-222">If you decide to use a different resource group name (other than `spoke1-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="05c49-223">Répétez l’étape 1 ci-dessus pour le fichier `spoke2.json`.</span><span class="sxs-lookup"><span data-stu-id="05c49-223">Repeat step 1 above for file `spoke2.json`.</span></span>

5. <span data-ttu-id="05c49-224">Exécutez `azbb` pour déployer le deuxième environnement de réseau virtuel spoke, comme indiqué ci-dessous.</span><span class="sxs-lookup"><span data-stu-id="05c49-224">Run `azbb` to deploy the second spoke VNet environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg - l <location> -p spoke2.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="05c49-225">Si vous décidez d’utiliser un nom de groupe de ressources différent (autre que `spoke2-vnet-rg`), veillez à rechercher tous les fichiers de paramètre qui utilisent ce nom et à les modifier afin qu’ils utilisent votre propre nom de groupe de ressources.</span><span class="sxs-lookup"><span data-stu-id="05c49-225">If you decide to use a different resource group name (other than `spoke2-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a><span data-ttu-id="05c49-226">Appairage de réseaux virtuels hub Azure avec des réseaux virtuels spokes</span><span class="sxs-lookup"><span data-stu-id="05c49-226">Azure hub VNet peering to spoke VNets</span></span>

<span data-ttu-id="05c49-227">Procédez comme suit pour créer une connexion d’homologation entre le réseau virtuel du hub et les réseaux virtuels spokes.</span><span class="sxs-lookup"><span data-stu-id="05c49-227">To create a peering connection from the hub VNet to the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="05c49-228">Ouvrez le fichier `hub-vnet-peering.json` et vérifiez l’exactitude du nom du groupe de ressources et de celui du réseau virtuel de chaque homologation de réseau virtuel, à compter de la ligne 29.</span><span class="sxs-lookup"><span data-stu-id="05c49-228">Open the `hub-vnet-peering.json` file and verify that the resource group name, and virtual network name for each of the virtual network peerings starting in line 29 are correct.</span></span>

2. <span data-ttu-id="05c49-229">Exécutez `azbb` pour déployer le premier environnement de réseau virtuel spoke, comme indiqué ci-dessous.</span><span class="sxs-lookup"><span data-stu-id="05c49-229">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet-peering.json --deploy
   ```

   > [!NOTE]
   > <span data-ttu-id="05c49-230">Si vous décidez d’utiliser un nom de groupe de ressources différent (autre que `hub-vnet-rg`), veillez à rechercher tous les fichiers de paramètre qui utilisent ce nom et à les modifier afin qu’ils utilisent votre propre nom de groupe de ressources.</span><span class="sxs-lookup"><span data-stu-id="05c49-230">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[guidance-hub-spoke]: ./hub-spoke.md
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[centre de données virtuel]: https://aka.ms/vdc
[virtual datacenter]: https://aka.ms/vdc
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/shared-services.png "Topologie de service partagé dans Azure"
[3]: ./images/hub-spokehub-spoke.svg "Topologie hub-and-spoke/hub-and-spoke dans Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
