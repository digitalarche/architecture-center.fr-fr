---
title: Implémentation d’une zone DMZ entre Azure et Internet
description: Comment implémenter une architecture réseau hybride sécurisée avec accès Internet dans Azure.
author: telmosampaio
ms.date: 07/02/2018
pnp.series.title: Network DMZ
pnp.series.next: nva-ha
pnp.series.prev: secure-vnet-hybrid
cardTitle: DMZ between Azure and the Internet
ms.openlocfilehash: 7a062d2394ae8b3bd1b17c19cbdf512327f9a766
ms.sourcegitcommit: 9b459f75254d97617e16eddd0d411d1f80b7fe90
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/03/2018
ms.locfileid: "37403145"
---
# <a name="dmz-between-azure-and-the-internet"></a><span data-ttu-id="e4f1b-103">Zone DMZ entre Azure et Internet</span><span class="sxs-lookup"><span data-stu-id="e4f1b-103">DMZ between Azure and the Internet</span></span>

<span data-ttu-id="e4f1b-104">Cette architecture de référence montre un réseau sécurisé hybride qui étend un réseau local sur Azure et accepte également le trafic Internet.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-104">This reference architecture shows a secure hybrid network that extends an on-premises network to Azure and also accepts Internet traffic.</span></span> [<span data-ttu-id="e4f1b-105">**Déployez cette solution**.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-105">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="e4f1b-106">[![0]][0]</span><span class="sxs-lookup"><span data-stu-id="e4f1b-106">[![0]][0]</span></span> 

<span data-ttu-id="e4f1b-107">*Téléchargez un [fichier Visio][visio-download] de cette architecture.*</span><span class="sxs-lookup"><span data-stu-id="e4f1b-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="e4f1b-108">Cette architecture de référence étend l’architecture décrite dans [Implementing a DMZ between Azure and your on-premises datacenter][implementing-a-secure-hybrid-network-architecture] (Implémenter une zone DMZ entre Azure et votre centre de données local).</span><span class="sxs-lookup"><span data-stu-id="e4f1b-108">This reference architecture extends the architecture described in [Implementing a DMZ between Azure and your on-premises datacenter][implementing-a-secure-hybrid-network-architecture].</span></span> <span data-ttu-id="e4f1b-109">Elle ajoute une zone DMZ qui gère le trafic Internet, en plus de la zone DMZ privée qui gère le trafic issu du réseau local</span><span class="sxs-lookup"><span data-stu-id="e4f1b-109">It adds a public DMZ that handles Internet traffic, in addition to the private DMZ that handles traffic from the on-premises network</span></span> 

<span data-ttu-id="e4f1b-110">Utilisations courantes de cette architecture :</span><span class="sxs-lookup"><span data-stu-id="e4f1b-110">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="e4f1b-111">Les applications hybrides où les charges de travail s’exécutent en partie en local et dans Azure.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-111">Hybrid applications where workloads run partly on-premises and partly in Azure.</span></span>
* <span data-ttu-id="e4f1b-112">Infrastructure Azure qui achemine le trafic entrant en provenance du réseau local et d’Internet.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-112">Azure infrastructure that routes incoming traffic from on-premises and the Internet.</span></span>

## <a name="architecture"></a><span data-ttu-id="e4f1b-113">Architecture</span><span class="sxs-lookup"><span data-stu-id="e4f1b-113">Architecture</span></span>

<span data-ttu-id="e4f1b-114">L’architecture est constituée des composants suivants.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-114">The architecture consists of the following components.</span></span>

* <span data-ttu-id="e4f1b-115">**Adresse IP publique**.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-115">**Public IP address (PIP)**.</span></span> <span data-ttu-id="e4f1b-116">Adresse IP publique du point de terminaison public.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-116">The IP address of the public endpoint.</span></span> <span data-ttu-id="e4f1b-117">Les utilisateurs externes connectés à Internet peuvent accéder au système via cette adresse.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-117">External users connected to the Internet can access the system through this address.</span></span>
* <span data-ttu-id="e4f1b-118">**Appliance virtuelle réseau**.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-118">**Network virtual appliance (NVA)**.</span></span> <span data-ttu-id="e4f1b-119">Cette architecture inclut un pool distinct d’appliances virtuelles réseau pour le trafic provenant d’Internet.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-119">This architecture includes a separate pool of NVAs for traffic originating on the Internet.</span></span>
* <span data-ttu-id="e4f1b-120">**Équilibreur de charge Azure**.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-120">**Azure load balancer**.</span></span> <span data-ttu-id="e4f1b-121">Toutes les demandes entrantes provenant d’Internet passent par l’équilibreur de charge et sont distribuées aux appliances virtuelles réseau dans la zone DMZ publique.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-121">All incoming requests from the Internet pass through the load balancer and are distributed to the NVAs in the public DMZ.</span></span>
* <span data-ttu-id="e4f1b-122">**Sous-réseau entrant de zone DMZ publique**.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-122">**Public DMZ inbound subnet**.</span></span> <span data-ttu-id="e4f1b-123">Ce sous-réseau accepte les requêtes issues de l’équilibreur de charge Azure.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-123">This subnet accepts requests from the Azure load balancer.</span></span> <span data-ttu-id="e4f1b-124">Les requêtes entrantes sont transmises à une des appliances virtuelles réseau de la zone DMZ publique.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-124">Incoming requests are passed to one of the NVAs in the public DMZ.</span></span>
* <span data-ttu-id="e4f1b-125">**Sous-réseau sortant de zone DMZ publique**.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-125">**Public DMZ outbound subnet**.</span></span> <span data-ttu-id="e4f1b-126">Les requêtes approuvées par l’appliance virtuelle réseau passent à travers ce sous-réseau vers l’équilibreur de charge interne pour la couche web.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-126">Requests that are approved by the NVA pass through this subnet to the internal load balancer for the web tier.</span></span>

## <a name="recommendations"></a><span data-ttu-id="e4f1b-127">Recommandations</span><span class="sxs-lookup"><span data-stu-id="e4f1b-127">Recommendations</span></span>

<span data-ttu-id="e4f1b-128">Les recommandations suivantes s’appliquent à la plupart des scénarios.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-128">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="e4f1b-129">Suivez ces recommandations, sauf si vous avez un besoin spécifique qui vous oblige à les ignorer.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-129">Follow these recommendations unless you have a specific requirement that overrides them.</span></span> 

### <a name="nva-recommendations"></a><span data-ttu-id="e4f1b-130">Recommandations en matière d’appliances virtuelles réseau</span><span class="sxs-lookup"><span data-stu-id="e4f1b-130">NVA recommendations</span></span>

<span data-ttu-id="e4f1b-131">Utilisez un ensemble d’appliances virtuelles réseau pour le trafic provenant d’Internet, et un autre ensemble pour le trafic émanant du niveau local.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-131">Use one set of NVAs for traffic originating on the Internet, and another for traffic originating on-premises.</span></span> <span data-ttu-id="e4f1b-132">L’utilisation d’un seul ensemble d’appliances virtuelles réseau pour ces deux types de trafics réseau constitue un risque pour la sécurité, car elle n’offre aucun périmètre de sécurité entre les deux.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-132">Using only one set of NVAs for both is a security risk, because it provides no security perimeter between the two sets of network traffic.</span></span> <span data-ttu-id="e4f1b-133">L’utilisation d’appliances virtuelles réseau distinctes simplifie la vérification des règles de sécurité et permet d’identifier clairement les règles auxquelles correspondent les différentes requêtes réseau entrantes.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-133">Using separate NVAs reduces the complexity of checking security rules, and makes it clear which rules correspond to each incoming network request.</span></span> <span data-ttu-id="e4f1b-134">Un ensemble d’appliances virtuelles réseau implémente les règles pour le trafic Internet uniquement, et l’autre celles du trafic local uniquement.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-134">One set of NVAs implements rules for Internet traffic only, while another set of NVAs implement rules for on-premises traffic only.</span></span>

<span data-ttu-id="e4f1b-135">Incluez une appliance virtuelle réseau de couche 7 pour mettre fin aux connexions d’application au niveau de l’appliance virtuelle réseau et assurer la compatibilité avec les couches principales.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-135">Include a layer-7 NVA to terminate application connections at the NVA level and maintain compatibility with the backend tiers.</span></span> <span data-ttu-id="e4f1b-136">Cela garantit une connectivité symétrique où le trafic de réponse issu des couches principales est renvoyé via l’appliance virtuelle réseau.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-136">This guarantees symmetric connectivity where response traffic from the backend tiers returns through the NVA.</span></span>  

### <a name="public-load-balancer-recommendations"></a><span data-ttu-id="e4f1b-137">Recommandations relatives à l’équilibreur de charge public</span><span class="sxs-lookup"><span data-stu-id="e4f1b-137">Public load balancer recommendations</span></span>

<span data-ttu-id="e4f1b-138">À des fins d’évolutivité et de disponibilité, déployez les appliances virtuelles réseau de la zone DMZ publique dans un [groupe à haute disponibilité][availability-set] et utilisez un [équilibreur de charge accessible sur Internet][load-balancer]pour répartir les requêtes Internet sur les appliances virtuelles réseau du groupe à haute disponibilité.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-138">For scalability and availability, deploy the public DMZ NVAs in an [availability set][availability-set] and use an [Internet facing load balancer][load-balancer] to distribute Internet requests across the NVAs in the availability set.</span></span>  

<span data-ttu-id="e4f1b-139">Configurer l’équilibreur de charge pour accepter les requêtes uniquement sur les ports nécessaires au trafic Internet.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-139">Configure the load balancer to accept requests only on the ports necessary for Internet traffic.</span></span> <span data-ttu-id="e4f1b-140">Par exemple, limitez les requêtes HTTP entrantes vers le port 80 et les requêtes HTTPS entrantes vers le port 443.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-140">For example, restrict inbound HTTP requests to port 80 and inbound HTTPS requests to port 443.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="e4f1b-141">Considérations relatives à l’extensibilité</span><span class="sxs-lookup"><span data-stu-id="e4f1b-141">Scalability considerations</span></span>

<span data-ttu-id="e4f1b-142">Même si votre architecture requiert initialement une appliance virtuelle réseau unique dans la zone DMZ publique, nous recommandons de placer un équilibreur de charge devant la zone DMZ publique dès le début.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-142">Even if your architecture initially requires a single NVA in the public DMZ, we recommend putting a load balancer in front of the public DMZ from the beginning.</span></span> <span data-ttu-id="e4f1b-143">Cela facilite l’évolution ultérieure vers plusieurs appliances virtuelles réseau, si nécessaire.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-143">That will make it easier to scale to multiple NVAs in the future, if needed.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="e4f1b-144">Considérations relatives à la disponibilité</span><span class="sxs-lookup"><span data-stu-id="e4f1b-144">Availability considerations</span></span>

<span data-ttu-id="e4f1b-145">L’équilibreur de charge accessible sur Internet requiert que chaque appliance virtuelle réseau de la zone DMZ du sous-réseau entrant de la zone publique DMZ implémente une [sonde d’intégrité][lb-probe].</span><span class="sxs-lookup"><span data-stu-id="e4f1b-145">The Internet facing load balancer requires each NVA in the public DMZ inbound subnet to implement a [health probe][lb-probe].</span></span> <span data-ttu-id="e4f1b-146">Une sonde d’intégrité qui ne répond pas sur ce point de terminaison est considérée comme étant indisponible. Par conséquent, l’équilibreur de charge doit diriger les requêtes vers d’autres appliances virtuelles réseau du même groupe à haute disponibilité.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-146">A health probe that fails to respond on this endpoint is considered to be unavailable, and the load balancer will direct requests to other NVAs in the same availability set.</span></span> <span data-ttu-id="e4f1b-147">Notez que si toutes les appliances virtuelles réseau ne répondent pas, votre application échoue. Il est donc important de configurer l’analyse afin d’alerter DevOps lorsque le nombre d’instances d’appliances virtuelles réseau saines tombe en dessous d’un seuil défini.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-147">Note that if all NVAs fail to respond, your application will fail, so it's important to have monitoring configured to alert DevOps when the number of healthy NVA instances falls below a defined threshold.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="e4f1b-148">Considérations relatives à la facilité de gestion</span><span class="sxs-lookup"><span data-stu-id="e4f1b-148">Manageability considerations</span></span>

<span data-ttu-id="e4f1b-149">L’ensemble de l’analyse et de la gestion des appliances virtuelles réseau de la zone publique DMZ doit être effectuée par la jumpbox dans le sous-réseau de gestion.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-149">All monitoring and management for the NVAs in the public DMZ should be performed by the jumpbox in the management subnet.</span></span> <span data-ttu-id="e4f1b-150">Comme indiqué dans [Implementing a DMZ between Azure and your on-premises datacenter][implementing-a-secure-hybrid-network-architecture] (Implémenter une zone DMZ entre Azure et votre centre de données local), définissez un itinéraire de réseau unique à partir du réseau local via la passerelle menant au jumpbox afin de restreindre accès.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-150">As discussed in [Implementing a DMZ between Azure and your on-premises datacenter][implementing-a-secure-hybrid-network-architecture], define a single network route from the on-premises network through the gateway to the jumpbox, in order to restrict access.</span></span>

<span data-ttu-id="e4f1b-151">Si la connectivité de la passerelle de votre réseau local vers Azure est interrompue, vous pouvez toujours atteindre le jumpbox en déployant une adresse IP publique, en l’ajoutant au jumpbox et en vous connectant à partir d’Internet.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-151">If gateway connectivity from your on-premises network to Azure is down, you can still reach the jumpbox by deploying a public IP address, adding it to the jumpbox, and logging in from the Internet.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="e4f1b-152">Considérations relatives à la sécurité</span><span class="sxs-lookup"><span data-stu-id="e4f1b-152">Security considerations</span></span>

<span data-ttu-id="e4f1b-153">Cette architecture de référence implémente plusieurs niveaux de sécurité :</span><span class="sxs-lookup"><span data-stu-id="e4f1b-153">This reference architecture implements multiple levels of security:</span></span>

* <span data-ttu-id="e4f1b-154">L’équilibreur de charge accessible sur Internet dirige les requêtes vers les appliances virtuelles réseau au sein du sous-réseau DMZ public et uniquement sur les ports nécessaires à l’application.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-154">The Internet facing load balancer directs requests to the NVAs in the inbound public DMZ subnet, and only on the ports necessary for the application.</span></span>
* <span data-ttu-id="e4f1b-155">Les règles du groupe de sécurité réseau pour les sous-réseaux de la zone DMZ publique entrants et sortants empêchent la compromission des appliances virtuelles réseau par le blocage des requêtes se trouvant en dehors des règles du groupe de sécurité réseau.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-155">The NSG rules for the inbound and outbound public DMZ subnets prevent the NVAs from being compromised, by blocking requests that fall outside of the NSG rules.</span></span>
* <span data-ttu-id="e4f1b-156">La configuration du routage NAT pour les appliances virtuelles réseau dirige les requêtes entrantes sur les ports 80 et 443 vers l’équilibreur de charge de la couche web, mais ignore les requêtes sur tous les autres ports.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-156">The NAT routing configuration for the NVAs directs incoming requests on port 80 and port 443 to the web tier load balancer, but ignores requests on all other ports.</span></span>

<span data-ttu-id="e4f1b-157">Vous devez journaliser toutes les requêtes entrantes sur tous les ports.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-157">You should log all incoming requests on all ports.</span></span> <span data-ttu-id="e4f1b-158">Effectuez un audit régulier des journaux en accordant une attention particulière aux requêtes qui se trouvent en dehors des paramètres attendus, comme cela peut indiquer des tentatives d’intrusion.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-158">Regularly audit the logs, paying attention to requests that fall outside of expected parameters, as these may indicate intrusion attempts.</span></span>


## <a name="deploy-the-solution"></a><span data-ttu-id="e4f1b-159">Déployer la solution</span><span class="sxs-lookup"><span data-stu-id="e4f1b-159">Deploy the solution</span></span>

<span data-ttu-id="e4f1b-160">Un déploiement pour une architecture de référence implémentant ces recommandations est disponible sur [GitHub][github-folder].</span><span class="sxs-lookup"><span data-stu-id="e4f1b-160">A deployment for a reference architecture that implements these recommendations is available on [GitHub][github-folder].</span></span> 

### <a name="prerequisites"></a><span data-ttu-id="e4f1b-161">Prérequis</span><span class="sxs-lookup"><span data-stu-id="e4f1b-161">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-resources"></a><span data-ttu-id="e4f1b-162">Déployer des ressources</span><span class="sxs-lookup"><span data-stu-id="e4f1b-162">Deploy resources</span></span>

1. <span data-ttu-id="e4f1b-163">Accédez au dossier `/dmz/secure-vnet-hybrid` du référentiel GitHub des architectures de référence.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-163">Navigate to the `/dmz/secure-vnet-hybrid` folder of the reference architectures GitHub repository.</span></span>

2. <span data-ttu-id="e4f1b-164">Exécutez la commande suivante :</span><span class="sxs-lookup"><span data-stu-id="e4f1b-164">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.json --deploy
    ```

3. <span data-ttu-id="e4f1b-165">Exécutez la commande suivante :</span><span class="sxs-lookup"><span data-stu-id="e4f1b-165">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p secure-vnet-hybrid.json --deploy
    ```

### <a name="connect-the-on-premises-and-azure-gateways"></a><span data-ttu-id="e4f1b-166">Connecter les passerelles Azure et locales</span><span class="sxs-lookup"><span data-stu-id="e4f1b-166">Connect the on-premises and Azure gateways</span></span>

<span data-ttu-id="e4f1b-167">Lors de cette étape, vous allez connecter les deux passerelles réseau local.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-167">In this step, you will connect the two local network gateways.</span></span>

1. <span data-ttu-id="e4f1b-168">Dans le portail Azure, accédez au groupe de ressources que vous avez créé.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-168">In the Azure Portal, navigate to the resource group that you created.</span></span> 

2. <span data-ttu-id="e4f1b-169">Recherchez la ressource appelée `ra-vpn-vgw-pip` et copiez l’adresse IP affichée dans le panneau **Vue d’ensemble**.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-169">Find the resource named `ra-vpn-vgw-pip` and copy the IP address shown in the **Overview** blade.</span></span>

3. <span data-ttu-id="e4f1b-170">Recherchez la ressource appelée `onprem-vpn-lgw`.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-170">Find the resource named `onprem-vpn-lgw`.</span></span>

4. <span data-ttu-id="e4f1b-171">Cliquez sur le panneau **Configuration**.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-171">Click the **Configuration** blade.</span></span> <span data-ttu-id="e4f1b-172">Sous **Adresse IP**, collez l’adresse IP identifiée à l’étape 2.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-172">Under **IP address**, paste in the IP address from step 2.</span></span>

    ![](./images/local-net-gw.png)

5. <span data-ttu-id="e4f1b-173">Cliquez sur **Enregistrer** et attendez que l’opération soit terminée.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-173">Click **Save** and wait for the operation to complete.</span></span> <span data-ttu-id="e4f1b-174">Elle peut durer environ 5 minutes.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-174">It can take about 5 minutes.</span></span>

6. <span data-ttu-id="e4f1b-175">Recherchez la ressource appelée `onprem-vpn-gateway1-pip`.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-175">Find the resource named `onprem-vpn-gateway1-pip`.</span></span> <span data-ttu-id="e4f1b-176">Copiez l’adresse IP indiquée dans le panneau **Vue d’ensemble**.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-176">Copy the IP address shown in the **Overview** blade.</span></span>

7. <span data-ttu-id="e4f1b-177">Recherchez la ressource appelée `ra-vpn-lgw`.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-177">Find the resource named `ra-vpn-lgw`.</span></span> 

8. <span data-ttu-id="e4f1b-178">Cliquez sur le panneau **Configuration**.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-178">Click the **Configuration** blade.</span></span> <span data-ttu-id="e4f1b-179">Sous **Adresse IP**, collez l’adresse IP identifiée à l’étape 6.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-179">Under **IP address**, paste in the IP address from step 6.</span></span>

9. <span data-ttu-id="e4f1b-180">Cliquez sur **Enregistrer** et attendez que l’opération soit terminée.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-180">Click **Save** and wait for the operation to complete.</span></span>

10. <span data-ttu-id="e4f1b-181">Pour vérifier la connexion, accédez au panneau **Connexions** de chaque passerelle.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-181">To verify the connection, go to the **Connections** blade for each gateway.</span></span> <span data-ttu-id="e4f1b-182">L’état doit être **Connecté**.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-182">The status should be **Connected**.</span></span>

### <a name="verify-that-network-traffic-reaches-the-web-tier"></a><span data-ttu-id="e4f1b-183">Vérifiez que le trafic réseau atteint le niveau web.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-183">Verify that network traffic reaches the web tier</span></span>

1. <span data-ttu-id="e4f1b-184">Dans le portail Azure, accédez au groupe de ressources que vous avez créé.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-184">In the Azure Portal, navigate to the resource group that you created.</span></span> 

2. <span data-ttu-id="e4f1b-185">Recherchez la ressource appelée `pub-dmz-lb`, qui correspond à l’équilibreur de charge placé devant la zone DMZ publique.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-185">Find the resource named `pub-dmz-lb`, which is the load balancer in front of the public DMZ.</span></span> 

3. <span data-ttu-id="e4f1b-186">Copiez l’adresse IP publique du panneau **Vue d’ensemble** et ouvrez cette adresse dans un navigateur web.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-186">Copy the public IP addess from the **Overview** blade and open this address in a web browser.</span></span> <span data-ttu-id="e4f1b-187">Vous devez voir apparaître la page d’accueil du serveur Apache2 par défaut.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-187">You should see the default Apache2 server home page.</span></span>

4. <span data-ttu-id="e4f1b-188">Recherchez la ressource appelée `int-dmz-lb`, qui correspond à l’équilibreur de charge placé devant la zone DMZ privée.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-188">Find the resource named `int-dmz-lb`, which is the load balancer in front of the private DMZ.</span></span> <span data-ttu-id="e4f1b-189">Copiez l’adresse IP privée dans le panneau **Vue d’ensemble**.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-189">Copy the private IP address from the **Overview** blade.</span></span>

5. <span data-ttu-id="e4f1b-190">Trouvez la machine virtuelle nommée `jb-vm1`.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-190">Find the VM named `jb-vm1`.</span></span> <span data-ttu-id="e4f1b-191">Cliquez sur **Connect** et utilisez le Bureau à distance pour vous connecter à la machine virtuelle.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-191">Click **Connect** and use Remote Desktop to connect to the VM.</span></span> <span data-ttu-id="e4f1b-192">Le nom d’utilisateur et le mot de passe sont spécifiés dans le fichier onprem.json.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-192">The user name and password are specified in the onprem.json file.</span></span>

6. <span data-ttu-id="e4f1b-193">À partir de la session Bureau à distance, ouvrez un navigateur web et accédez à l’adresse IP identifiée à l’étape 4.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-193">From the Remote Desktop Session, open a web browser and navigate to the IP address from step 4.</span></span> <span data-ttu-id="e4f1b-194">Vous devez voir apparaître la page d’accueil du serveur Apache2 par défaut.</span><span class="sxs-lookup"><span data-stu-id="e4f1b-194">You should see the default Apache2 server home page.</span></span>

[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-dmz

[implementing-a-secure-hybrid-network-architecture]: ./secure-vnet-hybrid.md
[iptables]: https://help.ubuntu.com/community/IptablesHowTo
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview
[load-balancer]: /azure/load-balancer/load-balancer-Internet-overview
[network-security-group]: /azure/virtual-network/virtual-networks-nsg

[visio-download]: https://archcenter.blob.core.windows.net/cdn/dmz-reference-architectures.vsdx


[0]: ./images/dmz-public.png "Architecture réseau hybride sécurisée"
