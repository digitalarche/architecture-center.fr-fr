---
title: Exécution de simulations CFD
titleSuffix: Azure Example Scenarios
description: Exécutez des simulations de diagramme de flux cumulé (CFD) sur Azure.
author: mikewarr
ms.date: 09/20/2018
ms.custom: fasttrack
ms.openlocfilehash: af43a60e952d75f84b4c7903a1567b0c76b9f4c4
ms.sourcegitcommit: bb7fcffbb41e2c26a26f8781df32825eb60df70c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/20/2018
ms.locfileid: "53643932"
---
# <a name="running-computational-fluid-dynamics-cfd-simulations-on-azure"></a><span data-ttu-id="5e6dd-103">Exécution des simulations de diagramme de flux cumulé (CFD) sur Azure</span><span class="sxs-lookup"><span data-stu-id="5e6dd-103">Running computational fluid dynamics (CFD) simulations on Azure</span></span>

<span data-ttu-id="5e6dd-104">Les simulations de mécanique des fluides numérique (CFD) nécessitent un temps de calcul considérable ainsi que du matériel spécialisé.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-104">Computational Fluid Dynamics (CFD) simulations require significant compute time along with specialized hardware.</span></span> <span data-ttu-id="5e6dd-105">À mesure que le recours aux clusters s’intensifie, les temps de simulation et l’utilisation totale des grilles augmentent, entraînant ainsi des problèmes de capacité de réserve et des temps d’attente excessifs.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-105">As cluster usage increases, simulation times and overall grid use grow, leading to issues with spare capacity and long queue times.</span></span> <span data-ttu-id="5e6dd-106">L’ajout de matériel physique peut se révéler coûteux et ne cadre pas toujours avec les pics d’utilisation et les creux d’activité auxquels sont confrontées les entreprises.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-106">Adding physical hardware can be expensive, and may not align to the usage peaks and valleys that a business goes through.</span></span> <span data-ttu-id="5e6dd-107">Azure vous permet de surmonter la plupart de ces difficultés sans engager de dépenses en capital.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-107">By taking advantage of Azure, many of these challenges can be overcome with no capital expenditure.</span></span>

<span data-ttu-id="5e6dd-108">Azure vous fournit le matériel dont vous avez besoin pour exécuter vos travaux CFD sur des machines virtuelles avec capacités GPU et UC.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-108">Azure provides the hardware you need to run your CFD jobs on both GPU and CPU virtual machines.</span></span> <span data-ttu-id="5e6dd-109">Les tailles de machine virtuelle prenant en charge la fonctionnalité RDMA (Accès direct à la mémoire à distance) présentent une mise en réseau basée sur InfiniBand FDR qui autorise une communication MPI (Message Passing Interface, passage de messages) à faible latence.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-109">RDMA (Remote Direct Memory Access) enabled VM sizes have FDR InfiniBand-based networking which allows for low latency MPI (Message Passing Interface) communication.</span></span> <span data-ttu-id="5e6dd-110">En combinant ces fonctionnalités avec Avere vFXT, un système de fichiers d’entreprise en cluster, les clients sont en mesure de garantir un débit maximal pour les opérations de lecture dans Azure.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-110">Combined with the Avere vFXT, which provides an enterprise-scale clustered file system, customers can ensure maximum throughput for read operations in Azure.</span></span>

<span data-ttu-id="5e6dd-111">Afin de simplifier la création, la gestion et l’optimisation des clusters HPC (calcul haute performance), vous pouvez utiliser Azure CycleCloud pour approvisionner les clusters et orchestrer les données dans les scénarios hybrides et cloud.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-111">To simplify the creation, management, and optimization of HPC clusters, Azure CycleCloud can be used to provision clusters and orchestrate data in both hybrid and cloud scenarios.</span></span> <span data-ttu-id="5e6dd-112">En supervisant les travaux en attente, CycleCloud lance automatiquement les calculs à la demande, et vous ne payez que ce que vous utilisez, en vous connectant au planificateur de charge de travail de votre choix.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-112">By monitoring the pending jobs, CycleCloud will automatically launch on-demand compute, where you only pay for what you use, connected to the workload scheduler of your choice.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="5e6dd-113">Cas d’usage appropriés</span><span class="sxs-lookup"><span data-stu-id="5e6dd-113">Relevant use cases</span></span>

<span data-ttu-id="5e6dd-114">Voici plusieurs autres secteurs d’activité utilisant le CFD :</span><span class="sxs-lookup"><span data-stu-id="5e6dd-114">Other relevant industries for CFD applications include:</span></span>

- <span data-ttu-id="5e6dd-115">Aéronautique</span><span class="sxs-lookup"><span data-stu-id="5e6dd-115">Aeronautics</span></span>
- <span data-ttu-id="5e6dd-116">Industrie automobile</span><span class="sxs-lookup"><span data-stu-id="5e6dd-116">Automotive</span></span>
- <span data-ttu-id="5e6dd-117">Chauffage, ventilation et climatisation (HVAC) de bâtiments</span><span class="sxs-lookup"><span data-stu-id="5e6dd-117">Building HVAC</span></span>
- <span data-ttu-id="5e6dd-118">Hydrocarbures</span><span class="sxs-lookup"><span data-stu-id="5e6dd-118">Oil and gas</span></span>
- <span data-ttu-id="5e6dd-119">Sciences de la vie</span><span class="sxs-lookup"><span data-stu-id="5e6dd-119">Life sciences</span></span>

## <a name="architecture"></a><span data-ttu-id="5e6dd-120">Architecture</span><span class="sxs-lookup"><span data-stu-id="5e6dd-120">Architecture</span></span>

![Diagramme de l’architecture][architecture]

<span data-ttu-id="5e6dd-122">Ce diagramme présente une vue d’ensemble générale d’une conception hybride traditionnelle assurant la supervision des travaux des nœuds à la demande dans Azure :</span><span class="sxs-lookup"><span data-stu-id="5e6dd-122">This diagram shows a high-level overview of a typical hybrid design providing job monitoring of the on-demand nodes in Azure:</span></span>

1. <span data-ttu-id="5e6dd-123">Connexion au cluster Azure CycleCloud pour configurer le cluster.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-123">Connect to the Azure CycleCloud server to configure the cluster.</span></span>
2. <span data-ttu-id="5e6dd-124">Configuration et création du nœud principal du cluster, à l’aide de machines prenant en charge RDMA pour MPI.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-124">Configure and create the cluster head node, using RDMA enabled machines for MPI.</span></span>
3. <span data-ttu-id="5e6dd-125">Ajout et configuration du nœud principal local.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-125">Add and configure the on-premises head node.</span></span>
4. <span data-ttu-id="5e6dd-126">Si les ressources sont insuffisantes, Azure CycleCloud augmente (ou réduit) les ressources de calcul dans Azure.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-126">If there are insufficient resources, Azure CycleCloud will scale up (or down) compute resources in Azure.</span></span> <span data-ttu-id="5e6dd-127">Il est possible de définir une limite prédéterminée pour empêcher une surutilisation.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-127">A predetermined limit can be defined to prevent over allocation.</span></span>
5. <span data-ttu-id="5e6dd-128">Allocation de tâches aux nœuds d’exécution.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-128">Tasks allocated to the execute nodes.</span></span>
6. <span data-ttu-id="5e6dd-129">Mise en cache des données dans Azure à partir du serveur NFS local.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-129">Data cached in Azure from on-premises NFS server.</span></span>
7. <span data-ttu-id="5e6dd-130">Lecture des données d’Avere vFXT pour le cache Azure.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-130">Data read in from the Avere vFXT for Azure cache.</span></span>
8. <span data-ttu-id="5e6dd-131">Transfert des informations de travail et de tâche vers le serveur Azure CycleCloud.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-131">Job and task information relayed to the Azure CycleCloud server.</span></span>

### <a name="components"></a><span data-ttu-id="5e6dd-132">Composants</span><span class="sxs-lookup"><span data-stu-id="5e6dd-132">Components</span></span>

- <span data-ttu-id="5e6dd-133">[Azure CycleCloud][cyclecloud] est un outil de création, de gestion, d’exploitation et d’optimisation de clusters HPC et Big Compute dans Azure.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-133">[Azure CycleCloud][cyclecloud] a tool for creating, managing, operating, and optimizing HPC and Big Compute clusters in Azure.</span></span>
- <span data-ttu-id="5e6dd-134">[Avere vFXT sur Azure][avere] fournit un système de fichiers d’entreprise en cluster conçu pour le cloud.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-134">[Avere vFXT on Azure][avere] is used to provide an enterprise-scale clustered file system built for the cloud.</span></span>
- <span data-ttu-id="5e6dd-135">[Machines virtuelles Azure][vms] permet de créer un ensemble statique d’instances de calcul.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-135">[Azure Virtual Machines (VMs)][vms] are used to create a static set of compute instances.</span></span>
- <span data-ttu-id="5e6dd-136">[Virtual Machine Scale Sets][vmss] fournit un groupe de machines virtuelles identiques pouvant faire l’objet d’une montée ou d’une descente en puissance par Azure CycleCloud.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-136">[Virtual Machine Scale Sets (virtual machine scale set)][vmss] provide a group of identical VMs capable of being scaled up or down by Azure CycleCloud.</span></span>
- <span data-ttu-id="5e6dd-137">Les [comptes Stockage Azure](/azure/storage/common/storage-introduction) sont utilisés pour la synchronisation et la rétention des données.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-137">[Azure Storage accounts](/azure/storage/common/storage-introduction) are used for synchronization and data retention.</span></span>
- <span data-ttu-id="5e6dd-138">[Réseau virtuel](/azure/virtual-network/virtual-networks-overview) permet à de nombreux types de ressources Azure, comme Machines virtuelles Azure, de communiquer en toute sécurité entre elles, avec Internet et avec les réseaux locaux.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-138">[Virtual Networks](/azure/virtual-network/virtual-networks-overview) enable many types of Azure resources, such as Azure Virtual Machines (VMs), to securely communicate with each other, the internet, and on-premises networks.</span></span>

### <a name="alternatives"></a><span data-ttu-id="5e6dd-139">Autres solutions</span><span class="sxs-lookup"><span data-stu-id="5e6dd-139">Alternatives</span></span>

<span data-ttu-id="5e6dd-140">Les clients peuvent également utiliser Azure CycleCloud pour créer une grille entièrement dans Azure.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-140">Customers can also use Azure CycleCloud to create a grid entirely in Azure.</span></span> <span data-ttu-id="5e6dd-141">Dans cette configuration, le serveur Azure CycleCloud est exécuté au sein de votre abonnement Azure.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-141">In this setup, the Azure CycleCloud server is run within your Azure subscription.</span></span>

<span data-ttu-id="5e6dd-142">Dans le cadre d’une approche moderne ne nécessitant pas la gestion d’un planificateur de charge de travail, [Azure Batch][batch] peut se révéler utile.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-142">For a modern application approach where management of a workload scheduler is not needed, [Azure Batch][batch] can help.</span></span> <span data-ttu-id="5e6dd-143">Azure Batch peut exécuter efficacement des applications de calcul haute performance (HPC) en parallèle et à grande échelle dans le cloud.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-143">Azure Batch can run large-scale parallel and high-performance computing (HPC) applications efficiently in the cloud.</span></span> <span data-ttu-id="5e6dd-144">Azure Batch vous permet de définir les ressources de calcul Azure pour exécuter vos applications en parallèle ou à grande échelle sans avoir à configurer ou gérer manuellement votre infrastructure.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-144">Azure Batch allows you to define the Azure compute resources to execute your applications in parallel or at scale without manually configuring or managing infrastructure.</span></span> <span data-ttu-id="5e6dd-145">Azure Batch planifie les tâches nécessitant beaucoup de ressources système et ajoute ou supprime des ressources de calcul de manière dynamique en fonction de vos besoins.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-145">Azure Batch schedules compute-intensive tasks and dynamically adds and removes compute resources based on your requirements.</span></span>

### <a name="scalability-and-security"></a><span data-ttu-id="5e6dd-146">Extensibilité et sécurité</span><span class="sxs-lookup"><span data-stu-id="5e6dd-146">Scalability, and Security</span></span>

<span data-ttu-id="5e6dd-147">La mise à l’échelle des nœuds d’exécution sur Azure CycleCloud peut être effectuée manuellement ou automatiquement.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-147">Scaling the execute nodes on Azure CycleCloud can be accomplished either manually or using autoscaling.</span></span> <span data-ttu-id="5e6dd-148">Pour plus d’informations, consultez l’article [CycleCloud Autoscaling][cycle-scale] (Mise à l’échelle automatique CycleCloud).</span><span class="sxs-lookup"><span data-stu-id="5e6dd-148">For more information, see [CycleCloud Autoscaling][cycle-scale].</span></span>

<span data-ttu-id="5e6dd-149">Pour obtenir des conseils d’ordre général sur la conception de solutions sécurisées, consultez la [documentation sur la sécurité Azure][security].</span><span class="sxs-lookup"><span data-stu-id="5e6dd-149">For general guidance on designing secure solutions, see the [Azure security documentation][security].</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="5e6dd-150">Déployez le scénario</span><span class="sxs-lookup"><span data-stu-id="5e6dd-150">Deploy the scenario</span></span>

### <a name="prerequisites"></a><span data-ttu-id="5e6dd-151">Prérequis</span><span class="sxs-lookup"><span data-stu-id="5e6dd-151">Prerequisites</span></span>

<span data-ttu-id="5e6dd-152">Avant de déployer le modèle Resource Manager, procédez comme suit :</span><span class="sxs-lookup"><span data-stu-id="5e6dd-152">Follow these steps before deploying the Resource Manager template:</span></span>

1. <span data-ttu-id="5e6dd-153">Créez un [principal de service][cycle-svcprin] pour la récupération des valeurs d’ID d’application, de nom d’affichage, de nom, de mot de passe et de locataire.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-153">Create a [service principal][cycle-svcprin] for retrieving the appId, displayName, name, password, and tenant.</span></span>
2. <span data-ttu-id="5e6dd-154">Générez une [paire de clés SSH][cycle-ssh] pour vous connecter en toute sécurité au serveur CycleCloud.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-154">Generate an [SSH key pair][cycle-ssh] to sign in securely to the CycleCloud server.</span></span>

    <!-- markdownlint-disable MD033 -->

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FCycleCloudCommunity%2Fcyclecloud_arm%2Fmaster%2Fazuredeploy.json" target="_blank">
        <img src="https://azuredeploy.net/deploybutton.png"/>
    </a>

    <!-- markdownlint-enable MD033 -->

3. <span data-ttu-id="5e6dd-155">[Connectez-vous au serveur CycleCloud][cycle-login] pour configurer et créer un nouveau cluster.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-155">[Log into the CycleCloud server][cycle-login] to configure and create a new cluster.</span></span>
4. <span data-ttu-id="5e6dd-156">[Créez un cluster][cycle-create].</span><span class="sxs-lookup"><span data-stu-id="5e6dd-156">[Create a cluster][cycle-create].</span></span>

<span data-ttu-id="5e6dd-157">Avere Cache est une solution facultative pouvant augmenter considérablement le débit de lecture des données de travail d’application.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-157">The Avere Cache is an optional solution that can drastically increase read throughput for the application job data.</span></span> <span data-ttu-id="5e6dd-158">Avere vFXT pour Azure résout le problème de l’exécution de ces applications HPC d’entreprise dans le cloud tout en tirant profit des données stockées localement ou dans le service Stockage Blob Azure.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-158">Avere vFXT for Azure solves the problem of running these enterprise HPC applications in the cloud while leveraging data stored on-premises or in Azure Blob storage.</span></span>

<span data-ttu-id="5e6dd-159">Pour les organisations qui planifient l’utilisation d’une infrastructure hybride combinant stockage local et cloud computing, les applications HPC peuvent s’intégrer à Azure en utilisant les données stockées dans des périphériques de stockage NAS et démarrer des processeurs virtuels en fonction des besoins.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-159">For organizations that are planning for a hybrid infrastructure with both on-premises storage and cloud computing, HPC applications can “burst” into Azure using data stored in NAS devices and spin up virtual CPUs as needed.</span></span> <span data-ttu-id="5e6dd-160">Le jeu de données n’est jamais déplacé intégralement vers le cloud.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-160">The data set is never moved completely into the cloud.</span></span> <span data-ttu-id="5e6dd-161">Les octets requis sont temporairement mis en cache à l’aide d’un cluster Avere pendant le traitement.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-161">The requested bytes are temporarily cached using an Avere cluster during processing.</span></span>

<span data-ttu-id="5e6dd-162">Pour installer et configurer une installation Avere vFXT, suivez les instructions du [guide d’installation et de configuration d’Avere][avere].</span><span class="sxs-lookup"><span data-stu-id="5e6dd-162">To set up and configure an Avere vFXT installation, follow the [Avere Setup and Configuration guide][avere].</span></span>

## <a name="pricing"></a><span data-ttu-id="5e6dd-163">Tarifs</span><span class="sxs-lookup"><span data-stu-id="5e6dd-163">Pricing</span></span>

<span data-ttu-id="5e6dd-164">Les coûts d’exécution d’une implémentation HPC avec un serveur CycleCloud dépendent d’un certain nombre de facteurs.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-164">The cost of running an HPC implementation using CycleCloud server will vary depending on a number of factors.</span></span> <span data-ttu-id="5e6dd-165">Par exemple, CycleCloud est facturé en fonction du temps de calcul utilisé, lorsque le serveur maître et le serveur CycleCloud sont alloués et exécutés en continu.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-165">For example, CycleCloud is charged by the amount of compute time that is used, with the Master and CycleCloud server typically being constantly allocated and running.</span></span> <span data-ttu-id="5e6dd-166">Les coûts d’exécution des nœuds d’exécution dépendent de la durée pendant laquelle ces nœuds sont opérationnels, ainsi que de la taille utilisée.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-166">The cost of running the Execute nodes will depend on how long these are up and running as well as what size is used.</span></span> <span data-ttu-id="5e6dd-167">Les frais de stockage et de mise en réseau Azure standard s’appliquent également.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-167">The normal Azure charges for storage and networking also apply.</span></span>

<span data-ttu-id="5e6dd-168">Ce scénario indique comment exécuter des applications CFD dans Azure. Par conséquent, les machines virtuelles nécessitent la fonctionnalité RDMA, qui est uniquement disponible sur des tailles de machine virtuelle spécifiques.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-168">This scenario shows how CFD applications can be run in Azure, so the machines will require RDMA functionality, which is only available on specific VM sizes.</span></span> <span data-ttu-id="5e6dd-169">Vous trouverez ci-après des exemples de coûts induits par l’utilisation d’un groupe identique alloué en continu huit heures par jour pendant un mois, avec une sortie de données de 1 To.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-169">The following are examples of costs that could be incurred for a scale set that is allocated continuously for eight hours per day for one month, with data egress of 1 TB.</span></span> <span data-ttu-id="5e6dd-170">La tarification du serveur Azure CycleCloud et de l’installation d’Avere vFXT pour Azure est également incluse :</span><span class="sxs-lookup"><span data-stu-id="5e6dd-170">It also includes pricing for the Azure CycleCloud server and the Avere vFXT for Azure install:</span></span>

- <span data-ttu-id="5e6dd-171">Région : Europe Nord</span><span class="sxs-lookup"><span data-stu-id="5e6dd-171">Region: North Europe</span></span>
- <span data-ttu-id="5e6dd-172">Serveur Azure CycleCloud : 1 x Standard D3 (4 UC, 14 Go de mémoire, HDD Standard 32 Go)</span><span class="sxs-lookup"><span data-stu-id="5e6dd-172">Azure CycleCloud Server: 1 x Standard D3 (4 x CPUs, 14 GB Memory, Standard HDD 32 GB)</span></span>
- <span data-ttu-id="5e6dd-173">Serveur maître Azure CycleCloud : 1 x Standard D12 (4 UC, 28 Go de mémoire, HDD Standard 32 Go)</span><span class="sxs-lookup"><span data-stu-id="5e6dd-173">Azure CycleCloud Master Server: 1 x Standard D12 v (4 x CPUs, 28 GB Memory, Standard HDD 32 GB)</span></span>
- <span data-ttu-id="5e6dd-174">Tableau de nœuds Azure CycleCloud : 10 x Standard H16r (16 UC, 112 Go de mémoire)</span><span class="sxs-lookup"><span data-stu-id="5e6dd-174">Azure CycleCloud Node Array: 10 x Standard H16r (16 x CPUs, 112 GB Memory)</span></span>
- <span data-ttu-id="5e6dd-175">Avere vFXT sur un cluster Azure : 3 x D16s v3 (système d’exploitation avec 200 Go, 1 disque SSD Premium de 1 To)</span><span class="sxs-lookup"><span data-stu-id="5e6dd-175">Avere vFXT on Azure Cluster: 3 x D16s v3 (200 GB OS, Premium SSD 1-TB data disk)</span></span>
- <span data-ttu-id="5e6dd-176">Sortie de données : 1 To</span><span class="sxs-lookup"><span data-stu-id="5e6dd-176">Data Egress: 1 TB</span></span>

<span data-ttu-id="5e6dd-177">Examinez cette [estimation de prix][pricing] pour le matériel répertorié ci-dessus.</span><span class="sxs-lookup"><span data-stu-id="5e6dd-177">Review this [price estimate][pricing] for the hardware listed above.</span></span>

## <a name="next-steps"></a><span data-ttu-id="5e6dd-178">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="5e6dd-178">Next Steps</span></span>

<span data-ttu-id="5e6dd-179">Une fois que vous avez déployé l’exemple, apprenez-en davantage sur [Azure CycleCloud][cyclecloud].</span><span class="sxs-lookup"><span data-stu-id="5e6dd-179">Once you've deployed the sample, learn more about [Azure CycleCloud][cyclecloud].</span></span>

## <a name="related-resources"></a><span data-ttu-id="5e6dd-180">Ressources associées</span><span class="sxs-lookup"><span data-stu-id="5e6dd-180">Related resources</span></span>

- <span data-ttu-id="5e6dd-181">[Instances prenant en charge RDMA][rdma]</span><span class="sxs-lookup"><span data-stu-id="5e6dd-181">[RDMA Capable Machine Instances][rdma]</span></span>
- <span data-ttu-id="5e6dd-182">[Personnalisation d’une machine virtuelle d’instance RDMA][rdma-custom]</span><span class="sxs-lookup"><span data-stu-id="5e6dd-182">[Customizing an RDMA Instance VM][rdma-custom]</span></span>

<!-- links -->
[architecture]: ./media/architecture-hpc-cfd.png
[calculator]: https://azure.com/e/
[availability]: /azure/architecture/checklist/availability
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[vmss]: /azure/virtual-machine-scale-sets/overview
[cyclecloud]: /azure/cyclecloud/
[rdma]: /azure/virtual-machines/windows/sizes-hpc#rdma-capable-instances
[gpu]: /azure/virtual-machines/windows/sizes-gpu
[hpcsizes]: /azure/virtual-machines/windows/sizes-hpc
[vms]: /azure/virtual-machines/
[low-pri]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-use-low-priority
[batch]: /azure/batch/
[avere]: https://github.com/Azure/Avere/blob/master/README.md
[cycle-prereq]: /azure/cyclecloud/quickstart-install-cyclecloud#prerequisites
[cycle-svcprin]: /azure/cyclecloud/quickstart-install-cyclecloud#service-principal
[cycle-ssh]: /azure/cyclecloud/quickstart-install-cyclecloud#ssh-keypair
[cycle-login]: /azure/cyclecloud/quickstart-install-cyclecloud#log-into-the-cyclecloud-application-server
[cycle-create]: /azure/cyclecloud/quickstart-create-and-run-cluster
[rdma]: /azure/virtual-machines/windows/sizes-hpc#rdma-capable-instances
[rdma-custom]: /azure/virtual-machines/linux/classic/rdma-cluster#customize-the-vm
[pricing]: https://azure.com/e/53030a04a2ab47a289156e2377a4247a
[cycle-scale]: /azure/cyclecloud/autoscale
