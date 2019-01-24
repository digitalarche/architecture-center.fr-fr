---
title: Environnements de développement/test pour les charges de travail SAP
titleSuffix: Azure Example Scenarios
description: Créez un environnement de développement/test pour les charges de travail SAP.
author: AndrewDibbins
ms.date: 07/11/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: fasttrack, SAP
ms.openlocfilehash: c06e81f72e264b545aef4bca2c795bb662a16a13
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54488203"
---
# <a name="devtest-environments-for-sap-workloads-on-azure"></a><span data-ttu-id="0a39e-103">Environnements de développement/test pour les charges de travail SAP sur Azure</span><span class="sxs-lookup"><span data-stu-id="0a39e-103">Dev/test environments for SAP workloads on Azure</span></span>

<span data-ttu-id="0a39e-104">Cet exemple montre comment créer un environnement de développement/test pour SAP NetWeaver dans un environnement Windows ou Linux sur Azure.</span><span class="sxs-lookup"><span data-stu-id="0a39e-104">This example shows how to establish a dev/test environment for SAP NetWeaver in a Windows or Linux environment on Azure.</span></span> <span data-ttu-id="0a39e-105">La base de données utilisée est AnyDB, le terme SAP pour tout SGBD pris en charge (qui n’est pas SAP HANA).</span><span class="sxs-lookup"><span data-stu-id="0a39e-105">The database used is AnyDB, the SAP term for any supported DBMS (that isn't SAP HANA).</span></span> <span data-ttu-id="0a39e-106">Étant donné que cette architecture est conçue pour les environnements hors production, elle est déployée avec une seule machine virtuelle et sa taille peut être modifiée pour prendre en compte les besoins de votre organisation.</span><span class="sxs-lookup"><span data-stu-id="0a39e-106">Because this architecture is designed for non-production environments, it's deployed with just a single virtual machine (VM) and it's size can be changed to accommodate your organization's needs.</span></span>

<span data-ttu-id="0a39e-107">Pour les cas d'usage de production, examinez les architectures de référence SAP disponibles ci-dessous :</span><span class="sxs-lookup"><span data-stu-id="0a39e-107">For production use cases review the SAP reference architectures available below:</span></span>

- <span data-ttu-id="0a39e-108">[SAP NetWeaver pour AnyDB][sap-netweaver]</span><span class="sxs-lookup"><span data-stu-id="0a39e-108">[SAP NetWeaver for AnyDB][sap-netweaver]</span></span>
- <span data-ttu-id="0a39e-109">[SAP S/4HANA][sap-hana]</span><span class="sxs-lookup"><span data-stu-id="0a39e-109">[SAP S/4HANA][sap-hana]</span></span>
- <span data-ttu-id="0a39e-110">[SAP sur des Instances de grande taille Azure][sap-large]</span><span class="sxs-lookup"><span data-stu-id="0a39e-110">[SAP on Azure large instances][sap-large]</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="0a39e-111">Cas d’usage appropriés</span><span class="sxs-lookup"><span data-stu-id="0a39e-111">Relevant use cases</span></span>

<span data-ttu-id="0a39e-112">Les autres cas d’usage appropriés sont les suivants :</span><span class="sxs-lookup"><span data-stu-id="0a39e-112">Other relevant use cases include:</span></span>

- <span data-ttu-id="0a39e-113">Charges de travail SAP non productives non critiques (bac à sable, développement, test, assurance qualité)</span><span class="sxs-lookup"><span data-stu-id="0a39e-113">Non-critical SAP non-productive workloads (sandbox, development, test, quality assurance)</span></span>
- <span data-ttu-id="0a39e-114">Charges de travail SAP Business non critiques</span><span class="sxs-lookup"><span data-stu-id="0a39e-114">Non-critical SAP business workloads</span></span>

## <a name="architecture"></a><span data-ttu-id="0a39e-115">Architecture</span><span class="sxs-lookup"><span data-stu-id="0a39e-115">Architecture</span></span>

![Diagramme d’architecture pour les environnements de développement/test des charges de travail SAP](./media/architecture-sap-dev-test.png)

<span data-ttu-id="0a39e-117">Ce scénario illustre le provisionnement d’une base de données système SAP et d’un serveur d’application SAP uniques sur une seule machine virtuelle.</span><span class="sxs-lookup"><span data-stu-id="0a39e-117">This scenario demonstrates provisioning a single SAP system database and SAP application server on a single virtual machine.</span></span> <span data-ttu-id="0a39e-118">Les données circulent dans le scénario comme suit :</span><span class="sxs-lookup"><span data-stu-id="0a39e-118">The data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="0a39e-119">Les clients utilisent l’interface utilisateur SAP ou d’autres outils clients (Excel, un navigateur Web ou une autre application Web) pour accéder au système SAP basé sur Azure.</span><span class="sxs-lookup"><span data-stu-id="0a39e-119">Customers use the SAP user interface or other client tools (Excel, a web browser, or other web application) to access the Azure-based SAP system.</span></span>
2. <span data-ttu-id="0a39e-120">La connectivité est assurée par une connexion ExpressRoute établie</span><span class="sxs-lookup"><span data-stu-id="0a39e-120">Connectivity is provided through the use of an established ExpressRoute.</span></span> <span data-ttu-id="0a39e-121">qui se termine dans Azure au niveau de la passerelle ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="0a39e-121">The ExpressRoute connection is terminated in Azure at the ExpressRoute gateway.</span></span> <span data-ttu-id="0a39e-122">Le trafic réseau est routé via la passerelle ExpressRoute vers le sous-réseau de passerelle, et depuis le sous-réseau de passerelle vers le sous-réseau spoke de niveau application (consultez les informations sur la [topologie réseau hub-and-spoke][hub-spoke]), et via une passerelle de sécurité réseau vers la machine virtuelle de l’application SAP.</span><span class="sxs-lookup"><span data-stu-id="0a39e-122">Network traffic routes through the ExpressRoute gateway to the gateway subnet, and from the gateway subnet to the application-tier spoke subnet (see the [hub-spoke network topology][hub-spoke]) and via a Network Security Gateway to the SAP application virtual machine.</span></span>
3. <span data-ttu-id="0a39e-123">Les serveurs de gestion d’identité fournissent des services d’authentification.</span><span class="sxs-lookup"><span data-stu-id="0a39e-123">The identity management servers provide authentication services.</span></span>
4. <span data-ttu-id="0a39e-124">Le serveur jumpbox offre des fonctionnalités de gestion locale.</span><span class="sxs-lookup"><span data-stu-id="0a39e-124">The jump box provides local management capabilities.</span></span>

### <a name="components"></a><span data-ttu-id="0a39e-125">Composants</span><span class="sxs-lookup"><span data-stu-id="0a39e-125">Components</span></span>

- <span data-ttu-id="0a39e-126">Les [réseaux virtuels](/azure/virtual-network/virtual-networks-overview) constituent la base de la communication réseau dans Azure.</span><span class="sxs-lookup"><span data-stu-id="0a39e-126">[Virtual Networks](/azure/virtual-network/virtual-networks-overview) are the basis of network communication within Azure.</span></span>
- <span data-ttu-id="0a39e-127">Les [machines virtuelles Azure](/azure/virtual-machines/windows/overview) fournissent une infrastructure sécurisée et virtualisée à la demande et à grande échelle avec un serveur Windows ou Linux.</span><span class="sxs-lookup"><span data-stu-id="0a39e-127">[Virtual Machine](/azure/virtual-machines/windows/overview) Azure Virtual Machines provides on-demand, high-scale, secure, virtualized infrastructure using Windows or Linux Server.</span></span>
- <span data-ttu-id="0a39e-128">[ExpressRoute](/azure/expressroute/expressroute-introduction) vous permet d’étendre vos réseaux locaux au cloud de Microsoft via une connexion privée assurée par un fournisseur de connectivité.</span><span class="sxs-lookup"><span data-stu-id="0a39e-128">[ExpressRoute](/azure/expressroute/expressroute-introduction) lets you extend your on-premises networks into the Microsoft cloud over a private connection facilitated by a connectivity provider.</span></span>
- <span data-ttu-id="0a39e-129">Les [groupes de sécurité réseau](/azure/virtual-network/security-overview) vous permettent de limiter le trafic réseau vers les ressources d’un réseau virtuel.</span><span class="sxs-lookup"><span data-stu-id="0a39e-129">[Network Security Group](/azure/virtual-network/security-overview) lets you limit network traffic to resources in a virtual network.</span></span> <span data-ttu-id="0a39e-130">Un groupe de sécurité réseau contient une liste de règles de sécurité qui autorisent ou refusent le trafic réseau entrant ou sortant en fonction de l’adresse IP source ou de destination, du port et du protocole.</span><span class="sxs-lookup"><span data-stu-id="0a39e-130">A network security group contains a list of security rules that allow or deny inbound or outbound network traffic based on source or destination IP address, port, and protocol.</span></span>
- <span data-ttu-id="0a39e-131">Les [groupes de ressources](/azure/azure-resource-manager/resource-group-overview#resource-groups) jouent le rôle de conteneurs logiques pour des ressources Azure.</span><span class="sxs-lookup"><span data-stu-id="0a39e-131">[Resource Groups](/azure/azure-resource-manager/resource-group-overview#resource-groups) act as logical containers for Azure resources.</span></span>

## <a name="considerations"></a><span data-ttu-id="0a39e-132">Considérations</span><span class="sxs-lookup"><span data-stu-id="0a39e-132">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="0a39e-133">Disponibilité</span><span class="sxs-lookup"><span data-stu-id="0a39e-133">Availability</span></span>

<span data-ttu-id="0a39e-134">Microsoft propose un contrat de niveau de service (SLA) pour les instances de machine virtuelle uniques.</span><span class="sxs-lookup"><span data-stu-id="0a39e-134">Microsoft offers a service level agreement (SLA) for single VM instances.</span></span> <span data-ttu-id="0a39e-135">Pour plus d’informations sur le contrat de niveau de service Microsoft Azure pour les machines virtuelles [Contrat SLA pour les machines virtuelles](https://azure.microsoft.com/support/legal/sla/virtual-machines)</span><span class="sxs-lookup"><span data-stu-id="0a39e-135">For more information on Microsoft Azure Service Level Agreement for Virtual Machines [SLA For Virtual Machines](https://azure.microsoft.com/support/legal/sla/virtual-machines)</span></span>

### <a name="scalability"></a><span data-ttu-id="0a39e-136">Extensibilité</span><span class="sxs-lookup"><span data-stu-id="0a39e-136">Scalability</span></span>

<span data-ttu-id="0a39e-137">Pour obtenir des conseils d’ordre général sur la conception de solutions évolutives, consultez la [liste de contrôle de l’extensibilité][scalability] dans le Centre des architectures Azure.</span><span class="sxs-lookup"><span data-stu-id="0a39e-137">For general guidance on designing scalable solutions, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="0a39e-138">Sécurité</span><span class="sxs-lookup"><span data-stu-id="0a39e-138">Security</span></span>

<span data-ttu-id="0a39e-139">Pour obtenir des conseils d’ordre général sur la conception de solutions sécurisées, consultez la [documentation sur la sécurité Azure][security].</span><span class="sxs-lookup"><span data-stu-id="0a39e-139">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="0a39e-140">Résilience</span><span class="sxs-lookup"><span data-stu-id="0a39e-140">Resiliency</span></span>

<span data-ttu-id="0a39e-141">Pour obtenir des conseils d’ordre général sur la conception de solutions résilientes, consultez l’article [Conception d’applications résilientes pour Azure][resiliency].</span><span class="sxs-lookup"><span data-stu-id="0a39e-141">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="pricing"></a><span data-ttu-id="0a39e-142">Tarifs</span><span class="sxs-lookup"><span data-stu-id="0a39e-142">Pricing</span></span>

<span data-ttu-id="0a39e-143">Pour vous aider à explorer le coût d’exécution de ce scénario, tous les services sont préconfigurés dans les exemples du calculateur de coûts ci-après.</span><span class="sxs-lookup"><span data-stu-id="0a39e-143">To help you explore the cost of running this scenario, all of the services are pre-configured in the cost calculator examples below.</span></span> <span data-ttu-id="0a39e-144">Pour pouvoir observer l’évolution de la tarification pour votre cas d’usage particulier, modifiez les variables appropriées en fonction du trafic que vous escomptez.</span><span class="sxs-lookup"><span data-stu-id="0a39e-144">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="0a39e-145">Nous proposons quatre exemples de profils de coût basés sur la quantité de trafic que vous prévoyez de recevoir :</span><span class="sxs-lookup"><span data-stu-id="0a39e-145">We have provided four sample cost profiles based on amount of traffic you expect to receive:</span></span>

|<span data-ttu-id="0a39e-146">Taille</span><span class="sxs-lookup"><span data-stu-id="0a39e-146">Size</span></span>|<span data-ttu-id="0a39e-147">SAP</span><span class="sxs-lookup"><span data-stu-id="0a39e-147">SAPs</span></span>|<span data-ttu-id="0a39e-148">Type de machine virtuelle</span><span class="sxs-lookup"><span data-stu-id="0a39e-148">VM Type</span></span>|<span data-ttu-id="0a39e-149">Stockage</span><span class="sxs-lookup"><span data-stu-id="0a39e-149">Storage</span></span>|<span data-ttu-id="0a39e-150">Calculatrice de tarification Azure</span><span class="sxs-lookup"><span data-stu-id="0a39e-150">Azure Pricing Calculator</span></span>|
|----|----|-------|-------|---------------|
|<span data-ttu-id="0a39e-151">Petite</span><span class="sxs-lookup"><span data-stu-id="0a39e-151">Small</span></span>|<span data-ttu-id="0a39e-152">8000</span><span class="sxs-lookup"><span data-stu-id="0a39e-152">8000</span></span>|<span data-ttu-id="0a39e-153">D8s_v3</span><span class="sxs-lookup"><span data-stu-id="0a39e-153">D8s_v3</span></span>|<span data-ttu-id="0a39e-154">2xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="0a39e-154">2xP20, 1xP10</span></span>|[<span data-ttu-id="0a39e-155">Petite</span><span class="sxs-lookup"><span data-stu-id="0a39e-155">Small</span></span>](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1)|
|<span data-ttu-id="0a39e-156">Moyenne</span><span class="sxs-lookup"><span data-stu-id="0a39e-156">Medium</span></span>|<span data-ttu-id="0a39e-157">16000</span><span class="sxs-lookup"><span data-stu-id="0a39e-157">16000</span></span>|<span data-ttu-id="0a39e-158">D16s_v3</span><span class="sxs-lookup"><span data-stu-id="0a39e-158">D16s_v3</span></span>|<span data-ttu-id="0a39e-159">3xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="0a39e-159">3xP20, 1xP10</span></span>|[<span data-ttu-id="0a39e-160">Moyenne</span><span class="sxs-lookup"><span data-stu-id="0a39e-160">Medium</span></span>](https://azure.com/e/465bd07047d148baab032b2f461550cd)|
<span data-ttu-id="0a39e-161">grand</span><span class="sxs-lookup"><span data-stu-id="0a39e-161">Large</span></span>|<span data-ttu-id="0a39e-162">32000</span><span class="sxs-lookup"><span data-stu-id="0a39e-162">32000</span></span>|<span data-ttu-id="0a39e-163">E32s_v3</span><span class="sxs-lookup"><span data-stu-id="0a39e-163">E32s_v3</span></span>|<span data-ttu-id="0a39e-164">3xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="0a39e-164">3xP20, 1xP10</span></span>|[<span data-ttu-id="0a39e-165">Grande</span><span class="sxs-lookup"><span data-stu-id="0a39e-165">Large</span></span>](https://azure.com/e/ada2e849d68b41c3839cc976000c6931)|
<span data-ttu-id="0a39e-166">Très grande</span><span class="sxs-lookup"><span data-stu-id="0a39e-166">Extra Large</span></span>|<span data-ttu-id="0a39e-167">64 000</span><span class="sxs-lookup"><span data-stu-id="0a39e-167">64000</span></span>|<span data-ttu-id="0a39e-168">M64s</span><span class="sxs-lookup"><span data-stu-id="0a39e-168">M64s</span></span>|<span data-ttu-id="0a39e-169">4xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="0a39e-169">4xP20, 1xP10</span></span>|[<span data-ttu-id="0a39e-170">Très grande</span><span class="sxs-lookup"><span data-stu-id="0a39e-170">Extra Large</span></span>](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef)|

> [!NOTE]
> <span data-ttu-id="0a39e-171">Cette tarification, fournie à titre informatif, indique uniquement les coûts relatifs au stockage et aux machines virtuelles.</span><span class="sxs-lookup"><span data-stu-id="0a39e-171">This pricing is a guide that only indicates the VMs and storage costs.</span></span> <span data-ttu-id="0a39e-172">Elle ne tient pas compte des frais associés à la mise en réseau, au stockage de sauvegarde et à l’entrée/la sortie des données.</span><span class="sxs-lookup"><span data-stu-id="0a39e-172">It excludes networking, backup storage, and data ingress/egress charges.</span></span>

- <span data-ttu-id="0a39e-173">[Petit](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1) : machine virtuelle de type D8s_v3 avec 8 processeurs virtuels, 32 Go de RAM et 200 Go de stockage temporaire, en plus de deux disques de Stockage Premium de 512 Go et un de 128 Go.</span><span class="sxs-lookup"><span data-stu-id="0a39e-173">[Small](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1): A small system consists of VM type D8s_v3 with 8x vCPUs, 32 GB RAM and 200 GB temp storage, additionally two 512 GB and one 128 GB premium storage disks.</span></span>
- <span data-ttu-id="0a39e-174">[Moyen](https://azure.com/e/465bd07047d148baab032b2f461550cd) : machine virtuelle de type D16s_v3 avec 16 processeurs virtuels, 64 Go de RAM et 400 Go de stockage temporaire, en plus de trois disques de Stockage Premium de 512 Go et un de 128 Go.</span><span class="sxs-lookup"><span data-stu-id="0a39e-174">[Medium](https://azure.com/e/465bd07047d148baab032b2f461550cd): A medium system consists of VM type D16s_v3 with 16x vCPUs, 64 GB RAM and 400 GB temp storage, additionally three 512 GB and one 128 GB premium storage disks.</span></span>
- <span data-ttu-id="0a39e-175">[Grand](https://azure.com/e/ada2e849d68b41c3839cc976000c6931) : machine virtuelle de type E32s_v3 avec 32 processeurs virtuels, 256 Go de RAM et 512 Go de stockage temporaire, en plus de trois disques de Stockage Premium de 512 Go et un de 128 Go.</span><span class="sxs-lookup"><span data-stu-id="0a39e-175">[Large](https://azure.com/e/ada2e849d68b41c3839cc976000c6931): A large system consists of VM type E32s_v3 with 32x vCPUs, 256 GB RAM and 512 GB temp storage, additionally three 512GB and one 128GB premium storage disks.</span></span>
- <span data-ttu-id="0a39e-176">[Très grand](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef) : machine virtuelle de type M64s avec 64 processeurs virtuels, 1 024 Go de RAM et 2 000 Go de stockage temporaire, en plus de quatre disques de Stockage Premium de 512 Go et un de 128 Go.</span><span class="sxs-lookup"><span data-stu-id="0a39e-176">[Extra Large](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef): An extra large system consists of a VM type M64s with 64x vCPUs, 1024 GB RAM and 2000 GB temp storage, additionally four 512 GB and one 128 GB premium storage disks.</span></span>

## <a name="deployment"></a><span data-ttu-id="0a39e-177">Déploiement</span><span class="sxs-lookup"><span data-stu-id="0a39e-177">Deployment</span></span>

<span data-ttu-id="0a39e-178">Cliquez ici pour déployer l’infrastructure sous-jacente à ce scénario.</span><span class="sxs-lookup"><span data-stu-id="0a39e-178">Click here to deploy the underlying infrastructure for this scenario.</span></span>

<!-- markdownlint-disable MD033 -->

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fsap-2tier%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

<!-- markdownlint-enable MD033 -->

> [!NOTE]
> <span data-ttu-id="0a39e-179">SAP et Oracle ne sont pas installés au cours de ce déploiement.</span><span class="sxs-lookup"><span data-stu-id="0a39e-179">SAP and Oracle are not installed during this deployment.</span></span> <span data-ttu-id="0a39e-180">Vous devez déployer ces composants séparément.</span><span class="sxs-lookup"><span data-stu-id="0a39e-180">You will need to deploy these components separately.</span></span>

<!-- links -->
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[sap-netweaver]: /azure/architecture/reference-architectures/sap/sap-netweaver
[sap-hana]: /azure/architecture/reference-architectures/sap/sap-s4hana
[sap-large]: /azure/architecture/reference-architectures/sap/hana-large-instances
[hub-spoke]: /azure/architecture/reference-architectures/hybrid-networking/hub-spoke
