---
title: Connecter un réseau local à Azure à l’aide d’un VPN
description: Comment implémenter une architecture réseau de site à site sécurisée qui s’étend sur un réseau virtuel Azure et un réseau local connecté à l’aide d’un VPN.
author: RohitSharma-pnp
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute
pnp.series.prev: ./index
cardTitle: VPN
ms.openlocfilehash: ef89cdd3e2a175f82929b613159a99557560cc7a
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/31/2018
ms.locfileid: "43325386"
---
# <a name="connect-an-on-premises-network-to-azure-using-a-vpn-gateway"></a><span data-ttu-id="9aeef-103">Connecter un réseau local à Azure à l’aide d’une passerelle VPN</span><span class="sxs-lookup"><span data-stu-id="9aeef-103">Connect an on-premises network to Azure using a VPN gateway</span></span>

<span data-ttu-id="9aeef-104">Cette architecture de référence montre comment étendre un réseau local à Azure à l’aide d’un réseau privé virtuel (VPN) site à site.</span><span class="sxs-lookup"><span data-stu-id="9aeef-104">This reference architecture shows how to extend an on-premises network to Azure, using a site-to-site virtual private network (VPN).</span></span> <span data-ttu-id="9aeef-105">Le trafic circule entre le réseau local et un réseau virtuel Azure (VNet) via un tunnel VPN IPSec.</span><span class="sxs-lookup"><span data-stu-id="9aeef-105">Traffic flows between the on-premises network and an Azure Virtual Network (VNet) through an IPSec VPN tunnel.</span></span> [<span data-ttu-id="9aeef-106">**Déployez cette solution**.</span><span class="sxs-lookup"><span data-stu-id="9aeef-106">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="9aeef-107">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="9aeef-107">![[0]][0]</span></span>

<span data-ttu-id="9aeef-108">*Téléchargez un [fichier Visio][visio-download] de cette architecture.*</span><span class="sxs-lookup"><span data-stu-id="9aeef-108">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="9aeef-109">Architecture</span><span class="sxs-lookup"><span data-stu-id="9aeef-109">Architecture</span></span> 

<span data-ttu-id="9aeef-110">L’architecture est constituée des composants suivants.</span><span class="sxs-lookup"><span data-stu-id="9aeef-110">The architecture consists of the following components.</span></span>

* <span data-ttu-id="9aeef-111">**Réseau local**.</span><span class="sxs-lookup"><span data-stu-id="9aeef-111">**On-premises network**.</span></span> <span data-ttu-id="9aeef-112">Un réseau local privé s’exécutant au sein d’une organisation.</span><span class="sxs-lookup"><span data-stu-id="9aeef-112">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="9aeef-113">**Appliance VPN**.</span><span class="sxs-lookup"><span data-stu-id="9aeef-113">**VPN appliance**.</span></span> <span data-ttu-id="9aeef-114">Périphérique ou service qui assure la connectivité externe au réseau local.</span><span class="sxs-lookup"><span data-stu-id="9aeef-114">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="9aeef-115">L’appliance VPN peut être un périphérique matériel ou une solution logicielle telle que le service RRAS (Routing and Remote Access Service) dans Windows Server 2012.</span><span class="sxs-lookup"><span data-stu-id="9aeef-115">The VPN appliance may be a hardware device, or it can be a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="9aeef-116">Pour obtenir la liste des appareils VPN pris en charge et des informations sur la configuration pour les connecter à une passerelle VPN Azure, consultez les instructions pour l’appareil sélectionné dans l’article [À propos des périphériques VPN pour les connexions de la passerelle VPN de site à site][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="9aeef-116">For a list of supported VPN appliances and information on configuring them to connect to an Azure VPN gateway, see the instructions for the selected device in the article [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="9aeef-117">**Réseau virtuel**.</span><span class="sxs-lookup"><span data-stu-id="9aeef-117">**Virtual network (VNet)**.</span></span> <span data-ttu-id="9aeef-118">L’application cloud et les composants de la passerelle VPN Azure résident dans le même [réseau virtuel][azure-virtual-network].</span><span class="sxs-lookup"><span data-stu-id="9aeef-118">The cloud application and the components for the Azure VPN gateway reside in the same [VNet][azure-virtual-network].</span></span>

* <span data-ttu-id="9aeef-119">**Passerelle VPN Azure**.</span><span class="sxs-lookup"><span data-stu-id="9aeef-119">**Azure VPN gateway**.</span></span> <span data-ttu-id="9aeef-120">Le service de [passerelle VPN][azure-vpn-gateway] vous permet de connecter le réseau virtuel au réseau local via une appliance VPN.</span><span class="sxs-lookup"><span data-stu-id="9aeef-120">The [VPN gateway][azure-vpn-gateway] service enables you to connect the VNet to the on-premises network through a VPN appliance.</span></span> <span data-ttu-id="9aeef-121">Pour plus d’informations, consultez [Connecter un réseau local à Microsoft Azure Virtual Network][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="9aeef-121">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span> <span data-ttu-id="9aeef-122">La passerelle VPN inclut les éléments suivants :</span><span class="sxs-lookup"><span data-stu-id="9aeef-122">The VPN gateway includes the following elements:</span></span>
  
  * <span data-ttu-id="9aeef-123">**Passerelle de réseau virtuel**.</span><span class="sxs-lookup"><span data-stu-id="9aeef-123">**Virtual network gateway**.</span></span> <span data-ttu-id="9aeef-124">Ressource qui fournit une appliance VPN virtuelle pour le réseau virtuel.</span><span class="sxs-lookup"><span data-stu-id="9aeef-124">A resource that provides a virtual VPN appliance for the VNet.</span></span> <span data-ttu-id="9aeef-125">Elle achemine le trafic du réseau local vers le réseau virtuel.</span><span class="sxs-lookup"><span data-stu-id="9aeef-125">It is responsible for routing traffic from the on-premises network to the VNet.</span></span>
  * <span data-ttu-id="9aeef-126">**Passerelle de réseau local**.</span><span class="sxs-lookup"><span data-stu-id="9aeef-126">**Local network gateway**.</span></span> <span data-ttu-id="9aeef-127">Abstraction de l’appliance VPN locale.</span><span class="sxs-lookup"><span data-stu-id="9aeef-127">An abstraction of the on-premises VPN appliance.</span></span> <span data-ttu-id="9aeef-128">Le trafic réseau allant de l’application cloud au réseau local est acheminé via cette passerelle.</span><span class="sxs-lookup"><span data-stu-id="9aeef-128">Network traffic from the cloud application to the on-premises network is routed through this gateway.</span></span>
  * <span data-ttu-id="9aeef-129">**Connexion**.</span><span class="sxs-lookup"><span data-stu-id="9aeef-129">**Connection**.</span></span> <span data-ttu-id="9aeef-130">La connexion a des propriétés qui spécifient le type de connexion (IPSec) et la clé partagée avec l’appliance VPN locale pour chiffrer le trafic.</span><span class="sxs-lookup"><span data-stu-id="9aeef-130">The connection has properties that specify the connection type (IPSec) and the key shared with the on-premises VPN appliance to encrypt traffic.</span></span>
  * <span data-ttu-id="9aeef-131">**Sous-réseau de passerelle**.</span><span class="sxs-lookup"><span data-stu-id="9aeef-131">**Gateway subnet**.</span></span> <span data-ttu-id="9aeef-132">La passerelle de réseau virtuel est conservée dans son propre sous-réseau, qui est soumis à des exigences diverses, décrites dans la section Recommandations ci-dessous.</span><span class="sxs-lookup"><span data-stu-id="9aeef-132">The virtual network gateway is held in its own subnet, which is subject to various requirements, described in the Recommendations section below.</span></span>

* <span data-ttu-id="9aeef-133">**Application cloud**.</span><span class="sxs-lookup"><span data-stu-id="9aeef-133">**Cloud application**.</span></span> <span data-ttu-id="9aeef-134">L’application hébergée dans Azure.</span><span class="sxs-lookup"><span data-stu-id="9aeef-134">The application hosted in Azure.</span></span> <span data-ttu-id="9aeef-135">Elle peut inclure plusieurs niveaux, avec plusieurs sous-réseaux connectés à l’aide d’équilibreurs de charge Azure.</span><span class="sxs-lookup"><span data-stu-id="9aeef-135">It might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="9aeef-136">Pour plus d’informations sur l’infrastructure d’application, consultez [Running Windows VM workloads][windows-vm-ra] (Exécution de charges de travail de machine virtuelle Windows) et [Exécution de charges de travail de machine virtuelle Linux][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="9aeef-136">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="9aeef-137">**Équilibreur de charge interne**.</span><span class="sxs-lookup"><span data-stu-id="9aeef-137">**Internal load balancer**.</span></span> <span data-ttu-id="9aeef-138">Le trafic réseau de la passerelle VPN est acheminé vers l’application cloud via un équilibreur de charge interne.</span><span class="sxs-lookup"><span data-stu-id="9aeef-138">Network traffic from the VPN gateway is routed to the cloud application through an internal load balancer.</span></span> <span data-ttu-id="9aeef-139">L’équilibreur de charge se trouve dans le sous-réseau frontal de l’application.</span><span class="sxs-lookup"><span data-stu-id="9aeef-139">The load balancer is located in the front-end subnet of the application.</span></span>

## <a name="recommendations"></a><span data-ttu-id="9aeef-140">Recommandations</span><span class="sxs-lookup"><span data-stu-id="9aeef-140">Recommendations</span></span>

<span data-ttu-id="9aeef-141">Les recommandations suivantes s’appliquent à la plupart des scénarios.</span><span class="sxs-lookup"><span data-stu-id="9aeef-141">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="9aeef-142">Suivez ces recommandations, sauf si vous avez un besoin spécifique qui vous oblige à les ignorer.</span><span class="sxs-lookup"><span data-stu-id="9aeef-142">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="vnet-and-gateway-subnet"></a><span data-ttu-id="9aeef-143">Réseau virtuel et sous-réseau de passerelle</span><span class="sxs-lookup"><span data-stu-id="9aeef-143">VNet and gateway subnet</span></span>

<span data-ttu-id="9aeef-144">Créez un réseau virtuel Azure avec un espace d’adressage suffisant pour toutes les ressources requises.</span><span class="sxs-lookup"><span data-stu-id="9aeef-144">Create an Azure VNet with an address space large enough for all of your required resources.</span></span> <span data-ttu-id="9aeef-145">Assurez-vous que l’espace d’adressage du réseau virtuel puisse évoluer au cas où vous auriez besoin de machines virtuelles supplémentaires à l’avenir.</span><span class="sxs-lookup"><span data-stu-id="9aeef-145">Ensure that the VNet address space has sufficient room for growth if additional VMs are likely to be needed in the future.</span></span> <span data-ttu-id="9aeef-146">L’espace d’adressage du réseau virtuel ne doit pas chevaucher le réseau local.</span><span class="sxs-lookup"><span data-stu-id="9aeef-146">The address space of the VNet must not overlap with the on-premises network.</span></span> <span data-ttu-id="9aeef-147">Par exemple, le diagramme ci-dessus utilise l’espace d’adressage 10.20.0.0/16 pour le réseau virtuel.</span><span class="sxs-lookup"><span data-stu-id="9aeef-147">For example, the diagram above uses the address space 10.20.0.0/16 for the VNet.</span></span>

<span data-ttu-id="9aeef-148">Créez un sous-réseau nommé *GatewaySubnet*, avec une plage d’adresses de /27.</span><span class="sxs-lookup"><span data-stu-id="9aeef-148">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="9aeef-149">La passerelle de réseau virtuel requiert ce sous-réseau.</span><span class="sxs-lookup"><span data-stu-id="9aeef-149">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="9aeef-150">En allouant 32 adresses à ce sous-réseau, vous devriez éviter les limitations de taille de passerelle.</span><span class="sxs-lookup"><span data-stu-id="9aeef-150">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span> <span data-ttu-id="9aeef-151">En outre, évitez de placer ce sous-réseau au milieu de l’espace d’adressage.</span><span class="sxs-lookup"><span data-stu-id="9aeef-151">Also, avoid placing this subnet in the middle of the address space.</span></span> <span data-ttu-id="9aeef-152">La meilleure pratique consiste à définir l’espace d’adressage du sous-réseau de passerelle à l’extrémité supérieure de l’espace d’adressage du réseau virtuel.</span><span class="sxs-lookup"><span data-stu-id="9aeef-152">A good practice is to set the address space for the gateway subnet at the upper end of the VNet address space.</span></span> <span data-ttu-id="9aeef-153">L’exemple du diagramme utilise 10.20.255.224/27.</span><span class="sxs-lookup"><span data-stu-id="9aeef-153">The example shown in the diagram uses 10.20.255.224/27.</span></span>  <span data-ttu-id="9aeef-154">Voici une procédure rapide pour calculer le [CIDR] :</span><span class="sxs-lookup"><span data-stu-id="9aeef-154">Here is a quick procedure to calculate the [CIDR]:</span></span>

1. <span data-ttu-id="9aeef-155">Définissez les bits variables dans l’espace d’adressage du réseau virtuel sur 1, jusqu'aux bits utilisés par le sous-réseau de passerelle, puis les bits restants sur 0.</span><span class="sxs-lookup"><span data-stu-id="9aeef-155">Set the variable bits in the address space of the VNet to 1, up to the bits being used by the gateway subnet, then set the remaining bits to 0.</span></span>
2. <span data-ttu-id="9aeef-156">Convertissez les bits obtenus en nombre décimal et exprimez celui-ci sous la forme d’un espace d’adressage dont la longueur de préfixe correspond à la taille du sous-réseau de passerelle.</span><span class="sxs-lookup"><span data-stu-id="9aeef-156">Convert the resulting bits to decimal and express it as an address space with the prefix length set to the size of the gateway subnet.</span></span>

<span data-ttu-id="9aeef-157">Par exemple, pour un réseau virtuel avec une plage d’adresses IP de 10.20.0.0/16, l’étape 1 ci-dessus donne le résultat 10.20.0b11111111.0b11100000.</span><span class="sxs-lookup"><span data-stu-id="9aeef-157">For example, for a VNet with an IP address range of 10.20.0.0/16, applying step #1 above becomes 10.20.0b11111111.0b11100000.</span></span>  <span data-ttu-id="9aeef-158">La conversion en nombre décimal et l’expression sous la forme d’un espace d’adressage génère 10.20.255.224/27.</span><span class="sxs-lookup"><span data-stu-id="9aeef-158">Converting that to decimal and expressing it as an address space yields 10.20.255.224/27.</span></span> 

> [!WARNING]
> <span data-ttu-id="9aeef-159">Ne déployez pas de machines virtuelles pour le sous-réseau de passerelle.</span><span class="sxs-lookup"><span data-stu-id="9aeef-159">Do not deploy any VMs to the gateway subnet.</span></span> <span data-ttu-id="9aeef-160">N’affectez pas non plus de groupe de sécurité réseau à ce sous-réseau, car la passerelle cesserait de fonctionner.</span><span class="sxs-lookup"><span data-stu-id="9aeef-160">Also, do not assign an NSG to this subnet, as it will cause the gateway to stop functioning.</span></span>
> 
> 

### <a name="virtual-network-gateway"></a><span data-ttu-id="9aeef-161">Passerelle de réseau virtuel</span><span class="sxs-lookup"><span data-stu-id="9aeef-161">Virtual network gateway</span></span>

<span data-ttu-id="9aeef-162">Allouez une adresse IP publique à la passerelle de réseau virtuel.</span><span class="sxs-lookup"><span data-stu-id="9aeef-162">Allocate a public IP address for the virtual network gateway.</span></span>

<span data-ttu-id="9aeef-163">Créez la passerelle de réseau virtuel dans le sous-réseau de passerelle et affectez-lui l’adresse IP publique nouvellement allouée.</span><span class="sxs-lookup"><span data-stu-id="9aeef-163">Create the virtual network gateway in the gateway subnet and assign it the newly allocated public IP address.</span></span> <span data-ttu-id="9aeef-164">Utilisez le type de passerelle qui correspond le mieux à vos besoins et qui est activé par votre appliance VPN :</span><span class="sxs-lookup"><span data-stu-id="9aeef-164">Use the gateway type that most closely matches your requirements and that is enabled by your VPN appliance:</span></span>

- <span data-ttu-id="9aeef-165">Créez une [passerelle basée sur des stratégies][policy-based-routing] si vous avez besoin contrôler avec précision la façon dont les requêtes sont acheminées en fonction de critères de stratégie tels que les préfixes d’adresse.</span><span class="sxs-lookup"><span data-stu-id="9aeef-165">Create a [policy-based gateway][policy-based-routing] if you need to closely control how requests are routed based on policy criteria such as address prefixes.</span></span> <span data-ttu-id="9aeef-166">Les passerelles basées sur des stratégies utilisent le routage statique et fonctionnent uniquement avec les connexions de site à site.</span><span class="sxs-lookup"><span data-stu-id="9aeef-166">Policy-based gateways use static routing, and only work with site-to-site connections.</span></span>

- <span data-ttu-id="9aeef-167">Créez une [passerelle basée sur des itinéraires][route-based-routing] si vous vous connectez au réseau local via RRAS, prenez en charge les connexions entre plusieurs régions ou sites, ou implémentez des connexions de réseau virtuel à réseau virtuel (y compris des itinéraires qui traversent plusieurs réseaux virtuels).</span><span class="sxs-lookup"><span data-stu-id="9aeef-167">Create a [route-based gateway][route-based-routing] if you connect to the on-premises network using RRAS, support multi-site or cross-region connections, or implement VNet-to-VNet connections (including routes that traverse multiple VNets).</span></span> <span data-ttu-id="9aeef-168">Les passerelles basées sur des itinéraires utilisent le routage dynamique pour diriger le trafic entre les réseaux.</span><span class="sxs-lookup"><span data-stu-id="9aeef-168">Route-based gateways use dynamic routing to direct traffic between networks.</span></span> <span data-ttu-id="9aeef-169">Elles tolèrent mieux les incidents dans le chemin d’accès réseau que les itinéraires statiques, car elles peuvent essayer d’autres itinéraires.</span><span class="sxs-lookup"><span data-stu-id="9aeef-169">They can tolerate failures in the network path better than static routes because they can try alternative routes.</span></span> <span data-ttu-id="9aeef-170">Les passerelles basées sur des itinéraires peuvent également réduire la charge de gestion, car les itinéraires ne doivent pas nécessairement être mis à jour manuellement en cas de changement des adresses réseau.</span><span class="sxs-lookup"><span data-stu-id="9aeef-170">Route-based gateways can also reduce the management overhead because routes might not need to be updated manually when network addresses change.</span></span>

<span data-ttu-id="9aeef-171">Pour obtenir la liste des appliances VPN prises en charge, voir [À propos des périphériques VPN pour les connexions de la passerelle VPN de site à site][vpn-appliances].</span><span class="sxs-lookup"><span data-stu-id="9aeef-171">For a list of supported VPN appliances, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliances].</span></span>

> [!NOTE]
> <span data-ttu-id="9aeef-172">Si vous souhaitez changer de type de passerelle une fois celle-ci créée, il vous faudra la supprimer et en recréer une autre.</span><span class="sxs-lookup"><span data-stu-id="9aeef-172">After the gateway has been created, you cannot change between gateway types without deleting and re-creating the gateway.</span></span>
> 
> 

<span data-ttu-id="9aeef-173">Sélectionnez la référence SKU de passerelle VPN Azure qui correspond le mieux à vos besoins en débit.</span><span class="sxs-lookup"><span data-stu-id="9aeef-173">Select the Azure VPN gateway SKU that most closely matches your throughput requirements.</span></span> <span data-ttu-id="9aeef-174">Pour plus d’informations, consultez [SKU de passerelle][azure-gateway-skus]</span><span class="sxs-lookup"><span data-stu-id="9aeef-174">For more informayion, see [Gateway SKUs][azure-gateway-skus]</span></span>

> [!NOTE]
> <span data-ttu-id="9aeef-175">La référence SKU de base n’est pas compatible avec Azure ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="9aeef-175">The Basic SKU is not compatible with Azure ExpressRoute.</span></span> <span data-ttu-id="9aeef-176">Vous pouvez [modifier la référence SKU][changing-SKUs] une fois la passerelle créée.</span><span class="sxs-lookup"><span data-stu-id="9aeef-176">You can [change the SKU][changing-SKUs] after the gateway has been created.</span></span>
> 
> 

<span data-ttu-id="9aeef-177">Le coût facturé est calculé en fonction de la durée pendant laquelle une passerelle est configurée et disponible.</span><span class="sxs-lookup"><span data-stu-id="9aeef-177">You are charged based on the amount of time that the gateway is provisioned and available.</span></span> <span data-ttu-id="9aeef-178">Voir la [tarification de la passerelle VPN][azure-gateway-charges].</span><span class="sxs-lookup"><span data-stu-id="9aeef-178">See [VPN Gateway Pricing][azure-gateway-charges].</span></span>

<span data-ttu-id="9aeef-179">Créez des règles de routage pour le sous-réseau de passerelle qui dirigent le trafic entrant de l’application de la passerelle vers l’équilibreur de charge interne, au lieu d’autoriser les requêtes à passer directement jusqu’aux machines virtuelles d’application.</span><span class="sxs-lookup"><span data-stu-id="9aeef-179">Create routing rules for the gateway subnet that direct incoming application traffic from the gateway to the internal load balancer, rather than allowing requests to pass directly to the application VMs.</span></span>

### <a name="on-premises-network-connection"></a><span data-ttu-id="9aeef-180">Connexion au réseau local</span><span class="sxs-lookup"><span data-stu-id="9aeef-180">On-premises network connection</span></span>

<span data-ttu-id="9aeef-181">Créer une passerelle de réseau local.</span><span class="sxs-lookup"><span data-stu-id="9aeef-181">Create a local network gateway.</span></span> <span data-ttu-id="9aeef-182">Spécifiez l’adresse IP publique de l’appliance VPN locale et l’espace d’adressage du réseau local.</span><span class="sxs-lookup"><span data-stu-id="9aeef-182">Specify the public IP address of the on-premises VPN appliance, and the address space of the on-premises network.</span></span> <span data-ttu-id="9aeef-183">Notez que l’appliance VPN locale doit avoir une adresse IP publique accessible par la passerelle de réseau local dans la passerelle VPN Azure.</span><span class="sxs-lookup"><span data-stu-id="9aeef-183">Note that the on-premises VPN appliance must have a public IP address that can be accessed by the local network gateway in Azure VPN Gateway.</span></span> <span data-ttu-id="9aeef-184">Le périphérique VPN ne peut pas se trouver derrière un périphérique de traduction d’adresses réseau (NAT).</span><span class="sxs-lookup"><span data-stu-id="9aeef-184">The VPN device cannot be located behind a network address translation (NAT) device.</span></span>

<span data-ttu-id="9aeef-185">Créez une connexion de site à site pour la passerelle de réseau virtuel et la passerelle de réseau local.</span><span class="sxs-lookup"><span data-stu-id="9aeef-185">Create a site-to-site connection for the virtual network gateway and the local network gateway.</span></span> <span data-ttu-id="9aeef-186">Sélectionnez le type de connexion de site à site (IPSec) et spécifiez la clé partagée.</span><span class="sxs-lookup"><span data-stu-id="9aeef-186">Select the site-to-site (IPSec) connection type, and specify the shared key.</span></span> <span data-ttu-id="9aeef-187">Le chiffrement de site à site avec la passerelle VPN Azure est basé sur le protocole IPSec, à l’aide de clés prépartagées pour l’authentification.</span><span class="sxs-lookup"><span data-stu-id="9aeef-187">Site-to-site encryption with the Azure VPN gateway is based on the IPSec protocol, using preshared keys for authentication.</span></span> <span data-ttu-id="9aeef-188">Vous spécifiez la clé lorsque vous créez la passerelle VPN Azure.</span><span class="sxs-lookup"><span data-stu-id="9aeef-188">You specify the key when you create the Azure VPN gateway.</span></span> <span data-ttu-id="9aeef-189">Vous devez configurer l’appliance VPN exécutée en local avec la même clé.</span><span class="sxs-lookup"><span data-stu-id="9aeef-189">You must configure the VPN appliance running on-premises with the same key.</span></span> <span data-ttu-id="9aeef-190">Les autres mécanismes d’authentification ne sont pas pris en charge actuellement.</span><span class="sxs-lookup"><span data-stu-id="9aeef-190">Other authentication mechanisms are not currently supported.</span></span>

<span data-ttu-id="9aeef-191">Vérifiez que l’infrastructure de routage locale est configurée pour transférer les requêtes destinées aux adresses de réseau virtuel Azure au périphérique VPN.</span><span class="sxs-lookup"><span data-stu-id="9aeef-191">Ensure that the on-premises routing infrastructure is configured to forward requests intended for addresses in the Azure VNet to the VPN device.</span></span>

<span data-ttu-id="9aeef-192">Ouvrez les ports requis par l’application cloud dans le réseau local.</span><span class="sxs-lookup"><span data-stu-id="9aeef-192">Open any ports required by the cloud application in the on-premises network.</span></span>

<span data-ttu-id="9aeef-193">Testez la connexion pour vérifier les points suivants :</span><span class="sxs-lookup"><span data-stu-id="9aeef-193">Test the connection to verify that:</span></span>

* <span data-ttu-id="9aeef-194">L’appliance VPN locale achemine correctement le trafic vers l’application cloud via la passerelle VPN Azure.</span><span class="sxs-lookup"><span data-stu-id="9aeef-194">The on-premises VPN appliance correctly routes traffic to the cloud application through the Azure VPN gateway.</span></span>
* <span data-ttu-id="9aeef-195">Le réseau virtuel réachemine correctement le trafic vers le réseau local.</span><span class="sxs-lookup"><span data-stu-id="9aeef-195">The VNet correctly routes traffic back to the on-premises network.</span></span>
* <span data-ttu-id="9aeef-196">Le trafic interdit dans les deux directions est correctement bloqué.</span><span class="sxs-lookup"><span data-stu-id="9aeef-196">Prohibited traffic in both directions is blocked correctly.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="9aeef-197">Considérations relatives à l’extensibilité</span><span class="sxs-lookup"><span data-stu-id="9aeef-197">Scalability considerations</span></span>

<span data-ttu-id="9aeef-198">Vous pouvez obtenir une évolutivité verticale limitée en passant des références SKU de passerelle VPN basique ou standard à la référence SKU de VPN hautes performances.</span><span class="sxs-lookup"><span data-stu-id="9aeef-198">You can achieve limited vertical scalability by moving from the Basic or Standard VPN Gateway SKUs to the High Performance VPN SKU.</span></span>

<span data-ttu-id="9aeef-199">Pour les réseaux virtuels qui attendent un gros volume de trafic VPN, envisagez de distribuer les différentes charges de travail dans des réseaux virtuels distincts plus petits et de configurer une passerelle VPN pour chacun d’entre eux.</span><span class="sxs-lookup"><span data-stu-id="9aeef-199">For VNets that expect a large volume of VPN traffic, consider distributing the different workloads into separate smaller VNets and configuring a VPN gateway for each of them.</span></span>

<span data-ttu-id="9aeef-200">Vous pouvez partitionner le réseau virtuel horizontalement ou verticalement.</span><span class="sxs-lookup"><span data-stu-id="9aeef-200">You can partition the VNet either horizontally or vertically.</span></span> <span data-ttu-id="9aeef-201">Pour un partitionnement horizontal, déplacez certaines instances de machine virtuelle de chaque couche vers des sous-réseaux du nouveau réseau virtuel.</span><span class="sxs-lookup"><span data-stu-id="9aeef-201">To partition horizontally, move some VM instances from each tier into subnets of the new VNet.</span></span> <span data-ttu-id="9aeef-202">Ainsi, chaque réseau virtuel a la même structure et les mêmes fonctionnalités.</span><span class="sxs-lookup"><span data-stu-id="9aeef-202">The result is that each VNet has the same structure and functionality.</span></span> <span data-ttu-id="9aeef-203">Pour un partitionnement vertical, réorganisez chaque couche pour diviser les fonctionnalités en différentes zones logiques (par exemple, la gestion des commandes, la facturation, la gestion des comptes client, etc.).</span><span class="sxs-lookup"><span data-stu-id="9aeef-203">To partition vertically, redesign each tier to divide the functionality into different logical areas (such as handling orders, invoicing, customer account management, and so on).</span></span> <span data-ttu-id="9aeef-204">Chaque zone fonctionnelle peut alors être placée dans son propre réseau virtuel.</span><span class="sxs-lookup"><span data-stu-id="9aeef-204">Each functional area can then be placed in its own VNet.</span></span>

<span data-ttu-id="9aeef-205">La réplication d’un contrôleur de domaine Active Directory local dans le réseau virtuel et l’implémentation d’un DNS dans le réseau virtuel contribuent à réduire la part du trafic relative à la sécurité et à l’administration qui passe du site local au cloud.</span><span class="sxs-lookup"><span data-stu-id="9aeef-205">Replicating an on-premises Active Directory domain controller in the VNet, and implementing DNS in the VNet, can help to reduce some of the security-related and administrative traffic flowing from on-premises to the cloud.</span></span> <span data-ttu-id="9aeef-206">Pour plus d’informations, voir [Extension d’Active Directory Domain Services (AD DS) à Azure][adds-extend-domain].</span><span class="sxs-lookup"><span data-stu-id="9aeef-206">For more information, see [Extending Active Directory Domain Services (AD DS) to Azure][adds-extend-domain].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="9aeef-207">Considérations relatives à la disponibilité</span><span class="sxs-lookup"><span data-stu-id="9aeef-207">Availability considerations</span></span>

<span data-ttu-id="9aeef-208">Si vous devez vous assurer que le réseau local reste disponible pour la passerelle VPN Azure, implémentez un cluster de basculement pour la passerelle VPN locale.</span><span class="sxs-lookup"><span data-stu-id="9aeef-208">If you need to ensure that the on-premises network remains available to the Azure VPN gateway, implement a failover cluster for the on-premises VPN gateway.</span></span>

<span data-ttu-id="9aeef-209">Si votre organisation possède plusieurs sites locaux, créez des [connexions multisites][vpn-gateway-multi-site] vers un ou plusieurs réseaux virtuels Azure.</span><span class="sxs-lookup"><span data-stu-id="9aeef-209">If your organization has multiple on-premises sites, create [multi-site connections][vpn-gateway-multi-site] to one or more Azure VNets.</span></span> <span data-ttu-id="9aeef-210">Cette approche requiert un routage dynamique (basé sur des itinéraires). Veillez donc à ce que la passerelle VPN locale prenne en charge cette fonctionnalité.</span><span class="sxs-lookup"><span data-stu-id="9aeef-210">This approach requires dynamic (route-based) routing, so make sure that the on-premises VPN gateway supports this feature.</span></span>

<span data-ttu-id="9aeef-211">Pour plus d’informations sur les contrats de niveau de service, voir le [contrat de niveau de service pour la passerelle VPN][sla-for-vpn-gateway].</span><span class="sxs-lookup"><span data-stu-id="9aeef-211">For details about service level agreements, see [SLA for VPN Gateway][sla-for-vpn-gateway].</span></span> 

## <a name="manageability-considerations"></a><span data-ttu-id="9aeef-212">Considérations relatives à la facilité de gestion</span><span class="sxs-lookup"><span data-stu-id="9aeef-212">Manageability considerations</span></span>

<span data-ttu-id="9aeef-213">Surveillez les informations de diagnostic provenant des appliances VPN locales.</span><span class="sxs-lookup"><span data-stu-id="9aeef-213">Monitor diagnostic information from on-premises VPN appliances.</span></span> <span data-ttu-id="9aeef-214">Ce processus dépend des fonctionnalités fournies par l’appliance VPN.</span><span class="sxs-lookup"><span data-stu-id="9aeef-214">This process depends on the features provided by the VPN appliance.</span></span> <span data-ttu-id="9aeef-215">Par exemple, si vous utilisez le service de routage et d’accès distant sur Windows Server 2012, [journalisation RRAS][rras-logging].</span><span class="sxs-lookup"><span data-stu-id="9aeef-215">For example, if you are using the Routing and Remote Access Service on Windows Server 2012, [RRAS logging][rras-logging].</span></span>

<span data-ttu-id="9aeef-216">Utilisez les [diagnostics de la passerelle VPN Azure][gateway-diagnostic-logs] pour capturer des informations sur les problèmes de connectivité.</span><span class="sxs-lookup"><span data-stu-id="9aeef-216">Use [Azure VPN gateway diagnostics][gateway-diagnostic-logs] to capture information about connectivity issues.</span></span> <span data-ttu-id="9aeef-217">Ces journaux contiennent des informations telles que la source et les destinations des requêtes de connexion, le protocole utilisé et la méthode d’établissement de la connexion (ou la raison de l’échec).</span><span class="sxs-lookup"><span data-stu-id="9aeef-217">These logs can be used to track information such as the source and destinations of connection requests, which protocol was used, and how the connection was established (or why the attempt failed).</span></span>

<span data-ttu-id="9aeef-218">Analysez les journaux des opérations de la passerelle VPN Azure à l’aide des journaux d’audit disponibles dans le portail Azure.</span><span class="sxs-lookup"><span data-stu-id="9aeef-218">Monitor the operational logs of the Azure VPN gateway using the audit logs available in the Azure portal.</span></span> <span data-ttu-id="9aeef-219">Des journaux distincts sont disponibles pour la passerelle de réseau local, la passerelle de réseau Azure et la connexion.</span><span class="sxs-lookup"><span data-stu-id="9aeef-219">Separate logs are available for the local network gateway, the Azure network gateway, and the connection.</span></span> <span data-ttu-id="9aeef-220">Ces informations permettent de suivre toutes les modifications apportées à la passerelle et peuvent être utiles si une passerelle cesse de fonctionner pour une raison quelconque.</span><span class="sxs-lookup"><span data-stu-id="9aeef-220">This information can be used to track any changes made to the gateway, and can be useful if a previously functioning gateway stops working for some reason.</span></span>

<span data-ttu-id="9aeef-221">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="9aeef-221">![[2]][2]</span></span>

<span data-ttu-id="9aeef-222">Contrôlez la connectivité et suivez les événements d’échec de connectivité.</span><span class="sxs-lookup"><span data-stu-id="9aeef-222">Monitor connectivity, and track connectivity failure events.</span></span> <span data-ttu-id="9aeef-223">Vous pouvez utiliser un package de contrôle comme [Nagios][nagios] pour capturer ces informations et les consigner dans des rapports.</span><span class="sxs-lookup"><span data-stu-id="9aeef-223">You can use a monitoring package such as [Nagios][nagios] to capture and report this information.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="9aeef-224">Considérations relatives à la sécurité</span><span class="sxs-lookup"><span data-stu-id="9aeef-224">Security considerations</span></span>

<span data-ttu-id="9aeef-225">Générez une clé partagée différente pour chaque passerelle VPN.</span><span class="sxs-lookup"><span data-stu-id="9aeef-225">Generate a different shared key for each VPN gateway.</span></span> <span data-ttu-id="9aeef-226">Utilisez une clé partagée forte qui résiste aux attaques en force brute.</span><span class="sxs-lookup"><span data-stu-id="9aeef-226">Use a strong shared key to help resist brute-force attacks.</span></span>

> [!NOTE]
> <span data-ttu-id="9aeef-227">Actuellement, vous ne pouvez pas utiliser Azure Key Vault afin de prépartager des clés pour la passerelle VPN Azure.</span><span class="sxs-lookup"><span data-stu-id="9aeef-227">Currently, you cannot use Azure Key Vault to preshare keys for the Azure VPN gateway.</span></span>
> 
> 

<span data-ttu-id="9aeef-228">Assurez-vous que l’appliance VPN locale utilise une méthode de chiffrement [compatible avec la passerelle VPN Azure][vpn-appliance-ipsec].</span><span class="sxs-lookup"><span data-stu-id="9aeef-228">Ensure that the on-premises VPN appliance uses an encryption method that is [compatible with the Azure VPN gateway][vpn-appliance-ipsec].</span></span> <span data-ttu-id="9aeef-229">Pour le routage basé sur des stratégies, la passerelle VPN Azure prend en charge les algorithmes de chiffrement AES256, AES128 et 3DES.</span><span class="sxs-lookup"><span data-stu-id="9aeef-229">For policy-based routing, the Azure VPN gateway supports the AES256, AES128, and 3DES encryption algorithms.</span></span> <span data-ttu-id="9aeef-230">Les passerelles basées sur des itinéraires prennent en charge AES256 et 3DES.</span><span class="sxs-lookup"><span data-stu-id="9aeef-230">Route-based gateways support AES256 and 3DES.</span></span>

<span data-ttu-id="9aeef-231">Si votre appliance VPN locale se trouve sur un réseau de périmètre (DMZ) qui a un pare-feu entre le réseau de périmètre et Internet, vous devrez peut-être configurer des [règles de pare-feu supplémentaires][additional-firewall-rules] pour permettre la connexion VPN de site à site.</span><span class="sxs-lookup"><span data-stu-id="9aeef-231">If your on-premises VPN appliance is on a perimeter network (DMZ) that has a firewall between the perimeter network and the Internet, you might have to configure [additional firewall rules][additional-firewall-rules] to allow the site-to-site VPN connection.</span></span>

<span data-ttu-id="9aeef-232">Si l’application du réseau virtuel envoie des données à Internet, envisagez de [mettre en œuvre le tunneling forcé][forced-tunneling] pour acheminer tout le trafic Internet via le réseau local.</span><span class="sxs-lookup"><span data-stu-id="9aeef-232">If the application in the VNet sends data to the Internet, consider [implementing forced tunneling][forced-tunneling] to route all Internet-bound traffic through the on-premises network.</span></span> <span data-ttu-id="9aeef-233">Cette approche vous permet d’auditer les requêtes sortantes effectuées par l’application à partir de l’infrastructure locale.</span><span class="sxs-lookup"><span data-stu-id="9aeef-233">This approach enables you to audit outgoing requests made by the application from the on-premises infrastructure.</span></span>

> [!NOTE]
> <span data-ttu-id="9aeef-234">Le tunneling forcé peut influer sur la connectivité aux services Azure (le service de stockage, par exemple) et le gestionnaire de licences de Windows.</span><span class="sxs-lookup"><span data-stu-id="9aeef-234">Forced tunneling can impact connectivity to Azure services (the Storage Service, for example) and the Windows license manager.</span></span>
> 
> 


## <a name="troubleshooting"></a><span data-ttu-id="9aeef-235">Résolution de problèmes</span><span class="sxs-lookup"><span data-stu-id="9aeef-235">Troubleshooting</span></span> 

<span data-ttu-id="9aeef-236">Pour obtenir des informations générales sur la résolution des problèmes courants liés au VPN, voir la page correspondante ([Troubleshooting common VPN related errors][troubleshooting-vpn-errors]).</span><span class="sxs-lookup"><span data-stu-id="9aeef-236">For general information on troubleshooting common VPN-related errors, see [Troubleshooting common VPN related errors][troubleshooting-vpn-errors].</span></span>

<span data-ttu-id="9aeef-237">Les recommandations suivantes sont utiles pour déterminer si votre appliance VPN locale fonctionne correctement.</span><span class="sxs-lookup"><span data-stu-id="9aeef-237">The following recommendations are useful for determining if your on-premises VPN appliance is functioning correctly.</span></span>

- <span data-ttu-id="9aeef-238">**Vérifiez les fichiers journaux générés par l’appliance VPN pour les erreurs ou les échecs.**</span><span class="sxs-lookup"><span data-stu-id="9aeef-238">**Check any log files generated by the VPN appliance for errors or failures.**</span></span>

    <span data-ttu-id="9aeef-239">Cela vous aidera à déterminer si l’appliance VPN fonctionne correctement.</span><span class="sxs-lookup"><span data-stu-id="9aeef-239">This will help you determine if the VPN appliance is functioning correctly.</span></span> <span data-ttu-id="9aeef-240">L’emplacement de ces informations varie en fonction de votre application.</span><span class="sxs-lookup"><span data-stu-id="9aeef-240">The location of this information will vary according to your appliance.</span></span> <span data-ttu-id="9aeef-241">Par exemple, avec RRAS sur Windows Server 2012, vous pouvez utiliser la commande PowerShell suivante pour afficher des informations sur les événements d’erreur du service RRAS :</span><span class="sxs-lookup"><span data-stu-id="9aeef-241">For example, if you are using RRAS on Windows Server 2012, you can use the following PowerShell command to display error event information for the RRAS service:</span></span>

    ```PowerShell
    Get-EventLog -LogName System -EntryType Error -Source RemoteAccess | Format-List -Property *
    ```

    <span data-ttu-id="9aeef-242">La propriété *Message* de chaque entrée fournit une description de l’erreur.</span><span class="sxs-lookup"><span data-stu-id="9aeef-242">The *Message* property of each entry provides a description of the error.</span></span> <span data-ttu-id="9aeef-243">Voici quelques exemples courants :</span><span class="sxs-lookup"><span data-stu-id="9aeef-243">Some common examples are:</span></span>

        - Inability to connect, possibly due to an incorrect IP address specified for the Azure VPN gateway in the RRAS VPN network interface configuration.

        ```
        EventID            : 20111
        MachineName        : on-prem-vm
        Data               : {41, 3, 0, 0}
        Index              : 14231
        Category           : (0)
        CategoryNumber     : 0
        EntryType          : Error
        Message            : RoutingDomainID- {00000000-0000-0000-0000-000000000000}: A demand dial connection to the remote
                             interface AzureGateway on port VPN2-4 was successfully initiated but failed to complete
                             successfully because of the  following error: The network connection between your computer and
                             the VPN server could not be established because the remote server is not responding. This could
                             be because one of the network devices (for example, firewalls, NAT, routers, and so on) between your computer
                             and the remote server is not configured to allow VPN connections. Please contact your
                             Administrator or your service provider to determine which device may be causing the problem.
        Source             : RemoteAccess
        ReplacementStrings : {{00000000-0000-0000-0000-000000000000}, AzureGateway, VPN2-4, The network connection between
                             your computer and the VPN server could not be established because the remote server is not
                             responding. This could be because one of the network devices (for example, firewalls, NAT, routers, and so on)
                             between your computer and the remote server is not configured to allow VPN connections. Please
                             contact your Administrator or your service provider to determine which device may be causing the
                             problem.}
        InstanceId         : 20111
        TimeGenerated      : 3/18/2016 1:26:02 PM
        TimeWritten        : 3/18/2016 1:26:02 PM
        UserName           :
        Site               :
        Container          :
        ```

        - The wrong shared key being specified in the RRAS VPN network interface configuration.

        ```
        EventID            : 20111
        MachineName        : on-prem-vm
        Data               : {233, 53, 0, 0}
        Index              : 14245
        Category           : (0)
        CategoryNumber     : 0
        EntryType          : Error
        Message            : RoutingDomainID- {00000000-0000-0000-0000-000000000000}: A demand dial connection to the remote
                             interface AzureGateway on port VPN2-4 was successfully initiated but failed to complete
                             successfully because of the  following error: Internet key exchange (IKE) authentication credentials are unacceptable.

        Source             : RemoteAccess
        ReplacementStrings : {{00000000-0000-0000-0000-000000000000}, AzureGateway, VPN2-4, IKE authentication credentials are
                             unacceptable.
                             }
        InstanceId         : 20111
        TimeGenerated      : 3/18/2016 1:34:22 PM
        TimeWritten        : 3/18/2016 1:34:22 PM
        UserName           :
        Site               :
        Container          :
        ```

    <span data-ttu-id="9aeef-244">Vous pouvez également obtenir des informations de journal des événements sur les tentatives de connexion via le service RRAS à l’aide de la commande PowerShell suivante :</span><span class="sxs-lookup"><span data-stu-id="9aeef-244">You can also obtain event log information about attempts to connect through the RRAS service using the following PowerShell command:</span></span> 

    ```
    Get-EventLog -LogName Application -Source RasClient | Format-List -Property *
    ```

    <span data-ttu-id="9aeef-245">En cas d’échec de connexion, ce fichier journal contient des erreurs semblables à ce qui suit :</span><span class="sxs-lookup"><span data-stu-id="9aeef-245">In the event of a failure to connect, this log will contain errors that look similar to the following:</span></span>

    ```
    EventID            : 20227
    MachineName        : on-prem-vm
    Data               : {}
    Index              : 4203
    Category           : (0)
    CategoryNumber     : 0
    EntryType          : Error
    Message            : CoId={B4000371-A67F-452F-AA4C-3125AA9CFC78}: The user SYSTEM dialed a connection named
                         AzureGateway that has failed. The error code returned on failure is 809.
    Source             : RasClient
    ReplacementStrings : {{B4000371-A67F-452F-AA4C-3125AA9CFC78}, SYSTEM, AzureGateway, 809}
    InstanceId         : 20227
    TimeGenerated      : 3/18/2016 1:29:21 PM
    TimeWritten        : 3/18/2016 1:29:21 PM
    UserName           :
    Site               :
    Container          :
    ```

- <span data-ttu-id="9aeef-246">**Vérifiez la connectivité et le routage sur la passerelle VPN.**</span><span class="sxs-lookup"><span data-stu-id="9aeef-246">**Verify connectivity and routing across the VPN gateway.**</span></span>

    <span data-ttu-id="9aeef-247">L’appliance VPN n’achemine peut-être pas correctement le trafic via la passerelle VPN Azure.</span><span class="sxs-lookup"><span data-stu-id="9aeef-247">The VPN appliance may not be correctly routing traffic through the Azure VPN Gateway.</span></span> <span data-ttu-id="9aeef-248">Utilisez un outil tel que [PsPing][psping] pour vérifier la connectivité et le routage sur la passerelle VPN.</span><span class="sxs-lookup"><span data-stu-id="9aeef-248">Use a tool such as [PsPing][psping] to verify connectivity and routing across the VPN gateway.</span></span> <span data-ttu-id="9aeef-249">Par exemple, pour tester la connectivité d’un ordinateur local à un serveur web situé sur le réseau virtuel, exécutez la commande suivante (en remplaçant `<<web-server-address>>` par l’adresse du serveur web) :</span><span class="sxs-lookup"><span data-stu-id="9aeef-249">For example, to test connectivity from an on-premises machine to a web server located on the VNet, run the following command (replacing `<<web-server-address>>` with the address of the web server):</span></span>

    ```
    PsPing -t <<web-server-address>>:80
    ```

    <span data-ttu-id="9aeef-250">Si l’ordinateur local peut acheminer le trafic vers le serveur web, vous devriez obtenir une sortie de ce type :</span><span class="sxs-lookup"><span data-stu-id="9aeef-250">If the on-premises machine can route traffic to the web server, you should see output similar to the following:</span></span>

    ```
    D:\PSTools>psping -t 10.20.0.5:80

    PsPing v2.01 - PsPing - ping, latency, bandwidth measurement utility
    Copyright (C) 2012-2014 Mark Russinovich
    Sysinternals - www.sysinternals.com

    TCP connect to 10.20.0.5:80:
    Infinite iterations (warmup 1) connecting test:
    Connecting to 10.20.0.5:80 (warmup): 6.21ms
    Connecting to 10.20.0.5:80: 3.79ms
    Connecting to 10.20.0.5:80: 3.44ms
    Connecting to 10.20.0.5:80: 4.81ms

      Sent = 3, Received = 3, Lost = 0 (0% loss),
      Minimum = 3.44ms, Maximum = 4.81ms, Average = 4.01ms
    ```

    <span data-ttu-id="9aeef-251">Si l’ordinateur local ne peut pas communiquer avec la destination spécifiée, vous verrez des messages de ce type :</span><span class="sxs-lookup"><span data-stu-id="9aeef-251">If the on-premises machine cannot communicate with the specified destination, you will see messages like this:</span></span>

    ```
    D:\PSTools>psping -t 10.20.1.6:80

    PsPing v2.01 - PsPing - ping, latency, bandwidth measurement utility
    Copyright (C) 2012-2014 Mark Russinovich
    Sysinternals - www.sysinternals.com

    TCP connect to 10.20.1.6:80:
    Infinite iterations (warmup 1) connecting test:
    Connecting to 10.20.1.6:80 (warmup): This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80:
      Sent = 3, Received = 0, Lost = 3 (100% loss),
      Minimum = 0.00ms, Maximum = 0.00ms, Average = 0.00ms
    ```

- <span data-ttu-id="9aeef-252">**Vérifiez que le pare-feu local autorise le passage du trafic VPN et que les ports appropriés sont ouverts.**</span><span class="sxs-lookup"><span data-stu-id="9aeef-252">**Verify that the on-premises firewall allows VPN traffic to pass and that the correct ports are opened.**</span></span>

- <span data-ttu-id="9aeef-253">**Vérifiez que l’appliance VPN locale utilise une méthode de chiffrement [compatible avec la passerelle VPN Azure][vpn-appliance].**</span><span class="sxs-lookup"><span data-stu-id="9aeef-253">**Verify that the on-premises VPN appliance uses an encryption method that is [compatible with the Azure VPN gateway][vpn-appliance].**</span></span> <span data-ttu-id="9aeef-254">Pour le routage basé sur des stratégies, la passerelle VPN Azure prend en charge les algorithmes de chiffrement AES256, AES128 et 3DES.</span><span class="sxs-lookup"><span data-stu-id="9aeef-254">For policy-based routing, the Azure VPN gateway supports the AES256, AES128, and 3DES encryption algorithms.</span></span> <span data-ttu-id="9aeef-255">Les passerelles basées sur des itinéraires prennent en charge AES256 et 3DES.</span><span class="sxs-lookup"><span data-stu-id="9aeef-255">Route-based gateways support AES256 and 3DES.</span></span>

<span data-ttu-id="9aeef-256">Les recommandations suivantes sont utiles pour déterminer s’il existe un problème avec la passerelle VPN Azure :</span><span class="sxs-lookup"><span data-stu-id="9aeef-256">The following recommendations are useful for determining if there is a problem with the Azure VPN gateway:</span></span>

- <span data-ttu-id="9aeef-257">**Examinez les [journaux de diagnostic de la passerelle VPN Azure][gateway-diagnostic-logs] à la recherche de problèmes éventuels.**</span><span class="sxs-lookup"><span data-stu-id="9aeef-257">**Examine [Azure VPN gateway diagnostic logs][gateway-diagnostic-logs] for potential issues.**</span></span>

- <span data-ttu-id="9aeef-258">**Vérifiez que la passerelle VPN Azure et l’appliance VPN locale sont configurées avec la même clé d’authentification partagée.**</span><span class="sxs-lookup"><span data-stu-id="9aeef-258">**Verify that the Azure VPN gateway and on-premises VPN appliance are configured with the same shared authentication key.**</span></span>

    <span data-ttu-id="9aeef-259">Vous pouvez afficher la clé partagée stockée par la passerelle VPN Azure à l’aide de la commande CLI Azure suivante :</span><span class="sxs-lookup"><span data-stu-id="9aeef-259">You can view the shared key stored by the Azure VPN gateway using the following Azure CLI command:</span></span>

    ```
    azure network vpn-connection shared-key show <<resource-group>> <<vpn-connection-name>>
    ```

    <span data-ttu-id="9aeef-260">Utilisez la commande appropriée pour votre appliance VPN locale afin d’afficher la clé partagée configurée pour cette application.</span><span class="sxs-lookup"><span data-stu-id="9aeef-260">Use the command appropriate for your on-premises VPN appliance to show the shared key configured for that appliance.</span></span>

    <span data-ttu-id="9aeef-261">Vérifiez que le sous-réseau *GatewaySubnet* contenant la passerelle VPN Azure n’est pas associé à un groupe de sécurité réseau.</span><span class="sxs-lookup"><span data-stu-id="9aeef-261">Verify that the *GatewaySubnet* subnet holding the Azure VPN gateway is not associated with an NSG.</span></span>

    <span data-ttu-id="9aeef-262">Vous pouvez afficher les détails du sous-réseau à l’aide de la commande CLI Azure suivante :</span><span class="sxs-lookup"><span data-stu-id="9aeef-262">You can view the subnet details using the following Azure CLI command:</span></span>

    ```
    azure network vnet subnet show -g <<resource-group>> -e <<vnet-name>> -n GatewaySubnet
    ```

    <span data-ttu-id="9aeef-263">Vérifiez qu’il n’existe aucun champ de données nommé *Network Security Group id*. L’exemple suivant montre les résultats pour une instance du sous-réseau *GatewaySubnet* à laquelle est affectée un groupe de sécurité réseau (*VPN-Gateway-Group*).</span><span class="sxs-lookup"><span data-stu-id="9aeef-263">Ensure there is no data field named *Network Security Group id*. The following example shows the results for an instance of the *GatewaySubnet* that has an assigned NSG (*VPN-Gateway-Group*).</span></span> <span data-ttu-id="9aeef-264">Cela peut empêcher la passerelle de fonctionner correctement si les règles sont définies pour ce groupe de sécurité réseau.</span><span class="sxs-lookup"><span data-stu-id="9aeef-264">This can prevent the gateway from working correctly if there are any rules defined for this NSG.</span></span>

    ```
    C:\>azure network vnet subnet show -g profx-prod-rg -e profx-vnet -n GatewaySubnet
        info:    Executing command network vnet subnet show
        + Looking up virtual network "profx-vnet"
        + Looking up the subnet "GatewaySubnet"
        data:    Id                              : /subscriptions/########-####-####-####-############/resourceGroups/profx-prod-rg/providers/Microsoft.Network/virtualNetworks/profx-vnet/subnets/GatewaySubnet
        data:    Name                            : GatewaySubnet
        data:    Provisioning state              : Succeeded
        data:    Address prefix                  : 10.20.3.0/27
        data:    Network Security Group id       : /subscriptions/########-####-####-####-############/resourceGroups/profx-prod-rg/providers/Microsoft.Network/networkSecurityGroups/VPN-Gateway-Group
        info:    network vnet subnet show command OK
    ```

- <span data-ttu-id="9aeef-265">**Vérifiez que les machines virtuelles du réseau virtuel Azure sont configurés pour autoriser le trafic provenant de l’extérieur du réseau virtuel.**</span><span class="sxs-lookup"><span data-stu-id="9aeef-265">**Verify that the virtual machines in the Azure VNet are configured to permit traffic coming in from outside the VNet.**</span></span>

    <span data-ttu-id="9aeef-266">Vérifiez toutes les règles du groupe de sécurité réseau associées aux sous-réseaux contenant ces machines virtuelles.</span><span class="sxs-lookup"><span data-stu-id="9aeef-266">Check any NSG rules associated with subnets containing these virtual machines.</span></span> <span data-ttu-id="9aeef-267">Vous pouvez afficher toutes les règles du groupe de sécurité réseau à l’aide de la commande CLI Azure suivante :</span><span class="sxs-lookup"><span data-stu-id="9aeef-267">You can view all NSG rules using the following Azure CLI command:</span></span>

    ```
    azure network nsg show -g <<resource-group>> -n <<nsg-name>>
    ```

- <span data-ttu-id="9aeef-268">**Vérifiez que la passerelle VPN Azure est connectée.**</span><span class="sxs-lookup"><span data-stu-id="9aeef-268">**Verify that the Azure VPN gateway is connected.**</span></span>

    <span data-ttu-id="9aeef-269">Vous pouvez utiliser la commande Azure PowerShell suivante pour vérifier l’état actuel de la connexion VPN Azure.</span><span class="sxs-lookup"><span data-stu-id="9aeef-269">You can use the following Azure PowerShell command to check the current status of the Azure VPN connection.</span></span> <span data-ttu-id="9aeef-270">Le paramètre `<<connection-name>>` est le nom de la connexion VPN Azure qui relie la passerelle du réseau virtuel et la passerelle locale.</span><span class="sxs-lookup"><span data-stu-id="9aeef-270">The `<<connection-name>>` parameter is the name of the Azure VPN connection that links the virtual network gateway and the local gateway.</span></span>

    ```
    Get-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> - ResourceGroupName <<resource-group>>
    ```

    <span data-ttu-id="9aeef-271">Les extraits de code suivants montrent la sortie générée si la passerelle est connectée (premier exemple) et déconnectée (second exemple) :</span><span class="sxs-lookup"><span data-stu-id="9aeef-271">The following snippets highlight the output generated if the gateway is connected (the first example), and disconnected (the second example):</span></span>

    ```
    PS C:\> Get-AzureRmVirtualNetworkGatewayConnection -Name profx-gateway-connection -ResourceGroupName profx-prod-rg

    AuthorizationKey           :
    VirtualNetworkGateway1     : Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
    VirtualNetworkGateway2     :
    LocalNetworkGateway2       : Microsoft.Azure.Commands.Network.Models.PSLocalNetworkGateway
    Peer                       :
    ConnectionType             : IPsec
    RoutingWeight              : 0
    SharedKey                  : ####################################
    ConnectionStatus           : Connected
    EgressBytesTransferred     : 55254803
    IngressBytesTransferred    : 32227221
    ProvisioningState          : Succeeded
    ...
    ```

    ```
    PS C:\> Get-AzureRmVirtualNetworkGatewayConnection -Name profx-gateway-connection2 -ResourceGroupName profx-prod-rg

    AuthorizationKey           :
    VirtualNetworkGateway1     : Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
    VirtualNetworkGateway2     :
    LocalNetworkGateway2       : Microsoft.Azure.Commands.Network.Models.PSLocalNetworkGateway
    Peer                       :
    ConnectionType             : IPsec
    RoutingWeight              : 0
    SharedKey                  : ####################################
    ConnectionStatus           : NotConnected
    EgressBytesTransferred     : 0
    IngressBytesTransferred    : 0
    ProvisioningState          : Succeeded
    ...
    ```

<span data-ttu-id="9aeef-272">Les recommandations suivantes sont utiles pour déterminer s’il existe un problème avec la configuration de machine virtuelle hôte, l’utilisation de la bande passante réseau ou les performances des applications :</span><span class="sxs-lookup"><span data-stu-id="9aeef-272">The following recommendations are useful for determining if there is an issue with Host VM configuration, network bandwidth utilization, or application performance:</span></span>

- <span data-ttu-id="9aeef-273">**Vérifiez que le pare-feu du système d’exploitation invité s’exécutant sur des machines virtuelles Azure dans le sous-réseau est correctement configuré pour permettre le trafic autorisé provenant des plages d’adresses IP locales.**</span><span class="sxs-lookup"><span data-stu-id="9aeef-273">**Verify that the firewall in the guest operating system running on the Azure VMs in the subnet is configured correctly to allow permitted traffic from the on-premises IP ranges.**</span></span>

- <span data-ttu-id="9aeef-274">**Vérifiez que le volume de trafic n’est pas proche de la limite de la bande passante disponible pour la passerelle VPN Azure.**</span><span class="sxs-lookup"><span data-stu-id="9aeef-274">**Verify that the volume of traffic is not close to the limit of the bandwidth available to the Azure VPN gateway.**</span></span>

    <span data-ttu-id="9aeef-275">La façon de vérifier cela dépend de l’appliance VPN exécutée localement.</span><span class="sxs-lookup"><span data-stu-id="9aeef-275">How to verify this depends on the VPN appliance running on-premises.</span></span> <span data-ttu-id="9aeef-276">Par exemple, avec RRAS sur Windows Server 2012, vous pouvez utiliser l’Analyseur de performances pour suivre le volume de données reçues et transmises sur la connexion VPN.</span><span class="sxs-lookup"><span data-stu-id="9aeef-276">For example, if you are using RRAS on Windows Server 2012, you can use Performance Monitor to track the volume of data being received and transmitted over the VPN connection.</span></span> <span data-ttu-id="9aeef-277">À l’aide de l’objet *RAS Total*, sélectionnez les compteurs d’*octets reçus par seconde* et d’*octets transmis par seconde* :</span><span class="sxs-lookup"><span data-stu-id="9aeef-277">Using the *RAS Total* object, select the *Bytes Received/Sec* and *Bytes Transmitted/Sec* counters:</span></span>

    <span data-ttu-id="9aeef-278">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="9aeef-278">![[3]][3]</span></span>

    <span data-ttu-id="9aeef-279">Vous devez comparer les résultats à la bande passante disponible pour la passerelle VPN (100 Mbits/s pour les références SKU basiques et standard, et 200 Mbits/s pour la référence SKU hautes performances) :</span><span class="sxs-lookup"><span data-stu-id="9aeef-279">You should compare the results with the bandwidth available to the VPN gateway (100 Mbps for the Basic and Standard SKUs, and 200 Mbps for the High Performance SKU):</span></span>

    <span data-ttu-id="9aeef-280">![[4]][4]</span><span class="sxs-lookup"><span data-stu-id="9aeef-280">![[4]][4]</span></span>

- <span data-ttu-id="9aeef-281">**Vérifiez que vous avez déployé le nombre approprié de machines virtuelles, avec la taille correcte, pour votre charge applicative.**</span><span class="sxs-lookup"><span data-stu-id="9aeef-281">**Verify that you have deployed the right number and size of VMs for your application load.**</span></span>

    <span data-ttu-id="9aeef-282">Déterminez si toutes les machines virtuelles du réseau virtuel Azure s’exécutent lentement.</span><span class="sxs-lookup"><span data-stu-id="9aeef-282">Determine if any of the virtual machines in the Azure VNet are running slowly.</span></span> <span data-ttu-id="9aeef-283">Si c’est le cas, elles peuvent être surchargées, c’est-à-dire pas assez nombreuses pour gérer la charge, ou les équilibreurs de charge ne sont pas configurés correctement.</span><span class="sxs-lookup"><span data-stu-id="9aeef-283">If so, they may be overloaded, there may be too few to handle the load, or the load-balancers may not be configured correctly.</span></span> <span data-ttu-id="9aeef-284">Pour le déterminer, [capturez et analysez les informations de diagnostic][azure-vm-diagnostics].</span><span class="sxs-lookup"><span data-stu-id="9aeef-284">To determine this, [capture and analyze diagnostic information][azure-vm-diagnostics].</span></span> <span data-ttu-id="9aeef-285">Vous pouvez examiner les résultats à l’aide du portail Azure, mais de nombreux outils tiers sont également disponibles pour obtenir des informations détaillées sur les données de performances.</span><span class="sxs-lookup"><span data-stu-id="9aeef-285">You can examine the results using the Azure portal, but many third-party tools are also available that can provide detailed insights into the performance data.</span></span>

- <span data-ttu-id="9aeef-286">**Vérifiez que l’application utilise efficacement les ressources cloud.**</span><span class="sxs-lookup"><span data-stu-id="9aeef-286">**Verify that the application is making efficient use of cloud resources.**</span></span>

    <span data-ttu-id="9aeef-287">Servez-vous du code d’application en cours d’exécution sur chaque machine virtuelle afin de déterminer si les applications font un usage optimal des ressources.</span><span class="sxs-lookup"><span data-stu-id="9aeef-287">Instrument application code running on each VM to determine whether applications are making the best use of resources.</span></span> <span data-ttu-id="9aeef-288">Vous pouvez utiliser des outils tels qu’[Application Insights][application-insights].</span><span class="sxs-lookup"><span data-stu-id="9aeef-288">You can use tools such as [Application Insights][application-insights].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="9aeef-289">Déployer la solution</span><span class="sxs-lookup"><span data-stu-id="9aeef-289">Deploy the solution</span></span>


<span data-ttu-id="9aeef-290">**Configuration requise.**</span><span class="sxs-lookup"><span data-stu-id="9aeef-290">**Prequisites.**</span></span> <span data-ttu-id="9aeef-291">Vous devez disposer d’une infrastructure locale existante déjà configurée avec une appliance réseau appropriée.</span><span class="sxs-lookup"><span data-stu-id="9aeef-291">You must have an existing on-premises infrastructure already configured with a suitable network appliance.</span></span>

<span data-ttu-id="9aeef-292">Pour déployer la solution, procédez comme suit :</span><span class="sxs-lookup"><span data-stu-id="9aeef-292">To deploy the solution, perform the following steps.</span></span>

1. <span data-ttu-id="9aeef-293">Cliquez sur le bouton ci-dessous :</span><span class="sxs-lookup"><span data-stu-id="9aeef-293">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fvpn%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="9aeef-294">Attendez que le lien s’ouvre dans le portail Azure, puis procédez comme suit :</span><span class="sxs-lookup"><span data-stu-id="9aeef-294">Wait for the link to open in the Azure portal, then follow these steps:</span></span> 
   * <span data-ttu-id="9aeef-295">Le nom du **groupe de ressources** est déjà défini dans le fichier de paramètres ; sélectionnez **Créer nouveau** et entrez `ra-hybrid-vpn-rg` dans la zone de texte.</span><span class="sxs-lookup"><span data-stu-id="9aeef-295">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-hybrid-vpn-rg` in the text box.</span></span>
   * <span data-ttu-id="9aeef-296">Sélectionnez la région à partir de la zone déroulante **Emplacement**.</span><span class="sxs-lookup"><span data-stu-id="9aeef-296">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="9aeef-297">Ne modifiez pas les zones de texte **Template Root Uri** (Uri racine de modèle) ou **Parameter Root Uri** (Uri racine de paramètre).</span><span class="sxs-lookup"><span data-stu-id="9aeef-297">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="9aeef-298">Passez en revue les termes et conditions, puis cochez la case **J’accepte les termes et conditions mentionnés ci-dessus**.</span><span class="sxs-lookup"><span data-stu-id="9aeef-298">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="9aeef-299">Cliquez sur le bouton **Acheter**.</span><span class="sxs-lookup"><span data-stu-id="9aeef-299">Click the **Purchase** button.</span></span>
3. <span data-ttu-id="9aeef-300">Attendez la fin du déploiement.</span><span class="sxs-lookup"><span data-stu-id="9aeef-300">Wait for the deployment to complete.</span></span>



<!-- links -->

[adds-extend-domain]: ../identity/adds-extend-domain.md
[expressroute]: ../hybrid-networking/expressroute.md
[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md

[naming conventions]: /azure/guidance/guidance-naming-conventions

[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[arm-templates]: /azure/resource-group-authoring-templates
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-portal]: /azure/azure-portal/resource-group-portal
[azure-powershell]: /azure/powershell-azure-resource-manager
[azure-virtual-network]: /azure/virtual-network/virtual-networks-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: https://azure.microsoft.com/services/vpn-gateway/
[azure-gateway-charges]: https://azure.microsoft.com/pricing/details/vpn-gateway/
[azure-gateway-skus]: /azure/vpn-gateway/vpn-gateway-about-vpngateways#gwsku
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[vpn-gateway-multi-site]: /azure/vpn-gateway/vpn-gateway-multi-site
[policy-based-routing]: https://en.wikipedia.org/wiki/Policy-based_routing
[route-based-routing]: https://en.wikipedia.org/wiki/Static_routing
[network-security-group]: /azure/virtual-network/virtual-networks-nsg
[sla-for-vpn-gateway]: https://azure.microsoft.com/support/legal/sla/vpn-gateway/v1_2/
[additional-firewall-rules]: https://technet.microsoft.com/library/dn786406.aspx#firewall
[nagios]: https://www.nagios.org/
[azure-vpn-gateway-diagnostics]: http://blogs.technet.com/b/keithmayer/archive/2014/12/18/diagnose-azure-virtual-network-vpn-connectivity-issues-with-powershell.aspx
[ping]: https://technet.microsoft.com/library/ff961503.aspx
[tracert]: https://technet.microsoft.com/library/ff961507.aspx
[psping]: http://technet.microsoft.com/sysinternals/jj729731.aspx
[nmap]: http://nmap.org
[changing-SKUs]: https://azure.microsoft.com/blog/azure-virtual-network-gateway-improvements/
[gateway-diagnostic-logs]: http://blogs.technet.com/b/keithmayer/archive/2015/12/07/step-by-step-capturing-azure-resource-manager-arm-vnet-gateway-diagnostic-logs.aspx
[troubleshooting-vpn-errors]: https://blogs.technet.microsoft.com/rrasblog/2009/08/12/troubleshooting-common-vpn-related-errors/
[rras-logging]: https://www.petri.com/enable-diagnostic-logging-in-windows-server-2012-r2-routing-and-remote-access
[create-on-prem-network]: https://technet.microsoft.com/library/dn786406.aspx#routing
[azure-vm-diagnostics]: https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/
[application-insights]: /azure/application-insights/app-insights-overview-usage
[forced-tunneling]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-forced-tunneling/
[vpn-appliances]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[vpn-appliance-ipsec]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices#ipsec-parameters
<span data-ttu-id="9aeef-302"><!--[solution-script]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/Deploy-ReferenceArchitecture.ps1-->
<!--[solution-script-bash]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/deploy-reference-architecture.sh-->
<!--[virtualNetworkGateway-parameters]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/parameters/virtualNetworkGateway.parameters.json-->
[azure-cli] : https://azure.microsoft.com/documentation/articles/xplat-cli-install/ [CIDR] : https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing [0] : ./images/vpn.png « Réseau hybride s’étendant sur des infrastructures Azure et locales » [2] : ../_images/guidance-hybrid-network-vpn/audit-logs.png « Journaux d’audit dans le Portail Azure » [3] : ../_images/guidance-hybrid-network-vpn/RRAS-perf-counters.png « Compteur de performances pour la supervision du trafic réseau VPN » [4] : ../_images/guidance-hybrid-network-vpn/RRAS-perf-graph.png « Exemple de graphique de performances de réseau VPN »</span><span class="sxs-lookup"><span data-stu-id="9aeef-302"><!--[solution-script]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/Deploy-ReferenceArchitecture.ps1-->
<!--[solution-script-bash]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/deploy-reference-architecture.sh-->
<!--[virtualNetworkGateway-parameters]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/parameters/virtualNetworkGateway.parameters.json-->
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/ [CIDR]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing [0]: ./images/vpn.png "Hybrid network spanning on-premises and Azure infrastructures" [2]: ../_images/guidance-hybrid-network-vpn/audit-logs.png "Audit logs in the Azure portal" [3]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-counters.png "Performance counters for monitoring VPN network traffic" [4]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-graph.png "Example VPN network performance graph"</span></span>"
