---
title: Calcul haute performance (HPC) sur Azure
description: Guide pour la génération de charges de travail HPC en cours d’exécution sur Azure
author: adamboeglin
ms.date: 2/4/2019
ms.openlocfilehash: 5263dd3a06e5244bf804df4be6ec57d789574f76
ms.sourcegitcommit: ea97ac004c38c6b456794c1a8eef29f8d2b77d50
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/26/2019
ms.locfileid: "58489196"
---
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD026 -->

# <a name="high-performance-computing-hpc-on-azure"></a><span data-ttu-id="fd306-103">Calcul haute performance (HPC) sur Azure</span><span class="sxs-lookup"><span data-stu-id="fd306-103">High Performance Computing (HPC) on Azure</span></span>

## <a name="introduction-to-hpc"></a><span data-ttu-id="fd306-104">Introduction à HPC</span><span class="sxs-lookup"><span data-stu-id="fd306-104">Introduction to HPC</span></span>

<!-- markdownlint-disable MD034 -->

> [!VIDEO https://www.youtube.com/embed/rKURT32faJk]

<!-- markdownlint-enable MD034 -->

<span data-ttu-id="fd306-105">HPC (High-Performance Computing, calcul haute performance), également appelé « Big Compute », utilise un grand nombre d’ordinateurs basés sur le processeur ou sur GPU pour résoudre des tâches mathématiques complexes.</span><span class="sxs-lookup"><span data-stu-id="fd306-105">High Performance Computing (HPC), also called "Big Compute", uses a large number of CPU or GPU-based computers to solve complex mathematical tasks.</span></span>

<span data-ttu-id="fd306-106">De nombreux secteurs d’activité utilisent HPC pour résoudre certains de leurs problèmes les plus complexes.</span><span class="sxs-lookup"><span data-stu-id="fd306-106">Many industries use HPC to solve some of their most difficult problems.</span></span>  <span data-ttu-id="fd306-107">Ces problèmes incluent notamment des charges de travail telles que les suivantes :</span><span class="sxs-lookup"><span data-stu-id="fd306-107">These include workloads such as:</span></span>

- <span data-ttu-id="fd306-108">Genomics</span><span class="sxs-lookup"><span data-stu-id="fd306-108">Genomics</span></span>
- <span data-ttu-id="fd306-109">Simulations relatives aux hydrocarbures</span><span class="sxs-lookup"><span data-stu-id="fd306-109">Oil and gas simulations</span></span>
- <span data-ttu-id="fd306-110">Finances</span><span class="sxs-lookup"><span data-stu-id="fd306-110">Finance</span></span>
- <span data-ttu-id="fd306-111">Conception de semiconducteurs</span><span class="sxs-lookup"><span data-stu-id="fd306-111">Semiconductor design</span></span>
- <span data-ttu-id="fd306-112">Ingénierie</span><span class="sxs-lookup"><span data-stu-id="fd306-112">Engineering</span></span>
- <span data-ttu-id="fd306-113">Modélisation de phénomènes météorologiques</span><span class="sxs-lookup"><span data-stu-id="fd306-113">Weather modeling</span></span>

### <a name="how-is-hpc-different-on-the-cloud"></a><span data-ttu-id="fd306-114">En quoi HPC est différent sur le cloud ?</span><span class="sxs-lookup"><span data-stu-id="fd306-114">How is HPC different on the cloud?</span></span>

<span data-ttu-id="fd306-115">L’une des principales différences entre un système HPC local et un système HPC dans le cloud est la possibilité d’ajouter et de supprimer dynamiquement des ressources en fonction des besoins.</span><span class="sxs-lookup"><span data-stu-id="fd306-115">One of the primary differences between an on-premise HPC system and one in the cloud is the ability for resources to dynamically be added and removed as they're needed.</span></span>  <span data-ttu-id="fd306-116">La mise à l’échelle dynamique supprime la capacité de calcul comme goulot d’étranglement et, à la place, permet aux clients de trouver la taille adaptée pour leur infrastructure pour satisfaire les exigences de leurs travaux.</span><span class="sxs-lookup"><span data-stu-id="fd306-116">Dynamic scaling removes compute capacity as a bottleneck and instead allow customers to right size their infrastructure for the requirements of their jobs.</span></span>

<span data-ttu-id="fd306-117">Les articles suivants fournissent davantage de détails sur cette fonctionnalité de mise à l’échelle dynamique.</span><span class="sxs-lookup"><span data-stu-id="fd306-117">The following articles provide more detail about this dynamic scaling capability.</span></span>

- [<span data-ttu-id="fd306-118">Style d’architecture Big Compute</span><span class="sxs-lookup"><span data-stu-id="fd306-118">Big Compute Architecture Style</span></span>](/azure/architecture/guide/architecture-styles/big-compute?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="fd306-119">Bonnes pratiques concernant la mise à l’échelle automatique</span><span class="sxs-lookup"><span data-stu-id="fd306-119">Autoscaling best practices</span></span>](/azure/architecture/best-practices/auto-scaling?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="implementation-checklist"></a><span data-ttu-id="fd306-120">Liste de vérification de l’implémentation</span><span class="sxs-lookup"><span data-stu-id="fd306-120">Implementation checklist</span></span>

<span data-ttu-id="fd306-121">Comme vous cherchez à implémenter votre propre solution HPC sur Azure, vérifiez que vous avez consulté les rubriques suivantes :</span><span class="sxs-lookup"><span data-stu-id="fd306-121">As you're looking to implement your own HPC solution on Azure, ensure you're reviewed the following topics:</span></span>

<!-- markdownlint-disable MD032 -->

> [!div class="checklist"]
> - <span data-ttu-id="fd306-122">Choisir l’[architecture](#infrastructure) appropriée en fonction de vos exigences</span><span class="sxs-lookup"><span data-stu-id="fd306-122">Choose the appropriate [architecture](#infrastructure) based on your requirements</span></span>
> - <span data-ttu-id="fd306-123">Déterminer quelles options de [calcul](#compute) conviennent pour votre charge de travail</span><span class="sxs-lookup"><span data-stu-id="fd306-123">Know which [compute](#compute) options is right for your workload</span></span>
> - <span data-ttu-id="fd306-124">Identifier la solution de [stockage](#storage) appropriée qui répond à vos besoins</span><span class="sxs-lookup"><span data-stu-id="fd306-124">Identify the right [storage](#storage) solution that meets your needs</span></span>
> - <span data-ttu-id="fd306-125">Décider comment vous allez [gérer](#management) toutes vos ressources</span><span class="sxs-lookup"><span data-stu-id="fd306-125">Decide how you're going to [manage](#management) all your resources</span></span>
> - <span data-ttu-id="fd306-126">Optimiser votre [application](#hpc-applications) pour le cloud</span><span class="sxs-lookup"><span data-stu-id="fd306-126">Optimize your [application](#hpc-applications) for the cloud</span></span>
> - <span data-ttu-id="fd306-127">[Sécuriser](#security) votre infrastructure</span><span class="sxs-lookup"><span data-stu-id="fd306-127">[Secure](#security) your Infrastructure</span></span>

<!-- markdownlint-enable MD032 -->

## <a name="infrastructure"></a><span data-ttu-id="fd306-128">Infrastructure</span><span class="sxs-lookup"><span data-stu-id="fd306-128">Infrastructure</span></span>

<span data-ttu-id="fd306-129">Un certain nombre de composants d’infrastructure sont nécessaires à la génération d’un système HPC.</span><span class="sxs-lookup"><span data-stu-id="fd306-129">There are a number of infrastructure components necessary to build an HPC system.</span></span>  <span data-ttu-id="fd306-130">Calcul, stockage et réseau fournissent les composants sous-jacents, quelle que soit la façon dont vous choisissez de gérer vos charges de travail HPC.</span><span class="sxs-lookup"><span data-stu-id="fd306-130">Compute, Storage, and Networking provide the underlying components, no matter how you choose to manage your HPC workloads.</span></span>

### <a name="example-hpc-architectures"></a><span data-ttu-id="fd306-131">Exemples d’architectures HPC</span><span class="sxs-lookup"><span data-stu-id="fd306-131">Example HPC architectures</span></span>

<span data-ttu-id="fd306-132">Il existe différentes façons de concevoir et d’implémenter votre architecture HPC sur Azure.</span><span class="sxs-lookup"><span data-stu-id="fd306-132">There are a number of different ways to design and implement your HPC architecture on Azure.</span></span>  <span data-ttu-id="fd306-133">Les applications HPC peuvent être mises à l’échelle pour plusieurs milliers de cœurs de calcul, étendre des clusters locaux ou s’exécuter en tant que solutions natives entièrement dans le cloud.</span><span class="sxs-lookup"><span data-stu-id="fd306-133">HPC applications can scale to thousands of compute cores, extend on-premises clusters, or run as a 100% cloud native solution.</span></span>

<span data-ttu-id="fd306-134">Les scénarios suivants décrivent quelques-unes des façons courantes de générer des solutions HPC.</span><span class="sxs-lookup"><span data-stu-id="fd306-134">The following scenarios outline a few of the common ways HPC solutions are built.</span></span>

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/apps/hpc-saas?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/apps/media/architecture-hpc-saas.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="fd306-135">Services d’ingénierie assistée par ordinateur sur Azure</span><span class="sxs-lookup"><span data-stu-id="fd306-135">Computer-aided engineering services on Azure</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="fd306-136">Offrez une plateforme de software as a service (SaaS) pour l’ingénierie assistée par ordinateur (IAO) sur Azure.</span><span class="sxs-lookup"><span data-stu-id="fd306-136">Provide a software-as-a-service (SaaS) platform for computer-aided engineering (CAE) on Azure.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/infrastructure/hpc-cfd?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/infrastructure/media/architecture-hpc-cfd.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="fd306-137">Simulations de diagramme de flux cumulé (CFD) sur Azure</span><span class="sxs-lookup"><span data-stu-id="fd306-137">Computational fluid dynamics (CFD) simulations on Azure</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="fd306-138">Exécutez des simulations de diagramme de flux cumulé (CFD) sur Azure.</span><span class="sxs-lookup"><span data-stu-id="fd306-138">Execute computational fluid dynamics (CFD) simulations on Azure.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/infrastructure/video-rendering?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/infrastructure/media/architecture-video-rendering.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="fd306-139">Rendu vidéo 3D sur Azure</span><span class="sxs-lookup"><span data-stu-id="fd306-139">3D video rendering on Azure</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="fd306-140">Exécutez des charges de travail HPC natives dans Azure à l’aide du service Azure Batch.</span><span class="sxs-lookup"><span data-stu-id="fd306-140">Run native HPC workloads in Azure using the Azure Batch service</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

### <a name="compute"></a><span data-ttu-id="fd306-141">Calcul</span><span class="sxs-lookup"><span data-stu-id="fd306-141">Compute</span></span>

<span data-ttu-id="fd306-142">Azure propose une gamme de tailles qui sont optimisées pour les charges de travail intensives processeur et GPU.</span><span class="sxs-lookup"><span data-stu-id="fd306-142">Azure offers a range of sizes that are optimized for both CPU & GPU intensive workloads.</span></span>

#### <a name="cpu-based-virtual-machines"></a><span data-ttu-id="fd306-143">Machines virtuelles basées sur le processeur</span><span class="sxs-lookup"><span data-stu-id="fd306-143">CPU-based virtual machines</span></span>

- [<span data-ttu-id="fd306-144">Machines virtuelles Linux</span><span class="sxs-lookup"><span data-stu-id="fd306-144">Linux VMs</span></span>](/azure/virtual-machines/linux/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- <span data-ttu-id="fd306-145">[Machines virtuelles Windows](/azure/virtual-machines/windows/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)</span><span class="sxs-lookup"><span data-stu-id="fd306-145">[Windows VM's](/azure/virtual-machines/windows/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) VMs</span></span>
  
#### <a name="gpu-enabled-virtual-machines"></a><span data-ttu-id="fd306-146">Machines virtuelles compatibles GPU</span><span class="sxs-lookup"><span data-stu-id="fd306-146">GPU-enabled virtual machines</span></span>

<span data-ttu-id="fd306-147">Les machines virtuelles de série N comportent des processeurs graphiques NVIDIA conçus pour des applications graphiques ou de calcul nécessitant beaucoup de ressources système, y compris la visualisation et l’apprentissage de l’intelligence artificielle (AI).</span><span class="sxs-lookup"><span data-stu-id="fd306-147">N-series VMs feature NVIDIA GPUs designed for compute-intensive or graphics-intensive applications including artificial intelligence (AI) learning and visualization.</span></span>

- [<span data-ttu-id="fd306-148">Machines virtuelles Linux</span><span class="sxs-lookup"><span data-stu-id="fd306-148">Linux VMs</span></span>](/azure/virtual-machines/linux/sizes-gpu?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="fd306-149">Machines virtuelles Windows</span><span class="sxs-lookup"><span data-stu-id="fd306-149">Windows VMs</span></span>](/azure/virtual-machines/windows/sizes-gpu?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

### <a name="storage"></a><span data-ttu-id="fd306-150">Stockage</span><span class="sxs-lookup"><span data-stu-id="fd306-150">Storage</span></span>

<span data-ttu-id="fd306-151">Les charges de travail HPC et Batch à grande échelle nécessitent un stockage des données et un accès à ces dernières dépassant les capacités des systèmes de fichiers cloud classiques.</span><span class="sxs-lookup"><span data-stu-id="fd306-151">Large-scale Batch and HPC workloads have demands for data storage and access that exceed the capabilities of traditional cloud file systems.</span></span>  <span data-ttu-id="fd306-152">Un certain nombre de solutions permettent de gérer les besoins de vitesse et de capacité des applications HPC sur Azure</span><span class="sxs-lookup"><span data-stu-id="fd306-152">There are a number of solutions to manage both the speed and capacity needs of HPC applications on Azure</span></span>

- <span data-ttu-id="fd306-153">[Avere vFXT](https://azure.microsoft.com/services/storage/avere-vfxt/) pour un stockage de données plus rapide et plus accessible pour un calcul haute performance à la périphérie</span><span class="sxs-lookup"><span data-stu-id="fd306-153">[Avere vFXT](https://azure.microsoft.com/services/storage/avere-vfxt/) for faster, more accessible data storage for high-performance computing at the edge</span></span>
- [<span data-ttu-id="fd306-154">BeeGFS</span><span class="sxs-lookup"><span data-stu-id="fd306-154">BeeGFS</span></span>](https://azure.microsoft.com/resources/implement-glusterfs-on-azure/)
- [<span data-ttu-id="fd306-155">Machines virtuelles à stockage optimisé</span><span class="sxs-lookup"><span data-stu-id="fd306-155">Storage Optimized Virtual Machines</span></span>](/azure/virtual-machines/windows/sizes-storage?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="fd306-156">Stockage Objet BLOB, Table et File d’attente</span><span class="sxs-lookup"><span data-stu-id="fd306-156">Blob, table, and queue storage</span></span>](/azure/storage/storage-introduction?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="fd306-157">Stockage de fichiers SMB Azure</span><span class="sxs-lookup"><span data-stu-id="fd306-157">Azure SMB File storage</span></span>](/azure/storage/files/storage-files-introduction?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="fd306-158">Intel Cloud Edition pour Lustre</span><span class="sxs-lookup"><span data-stu-id="fd306-158">Intel Cloud Edition Lustre</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/intel.intel-cloud-edition-gs)

<span data-ttu-id="fd306-159">Pour plus d’informations sur la comparaison de Lustre, de GlusterFS et de BeeGFS sur Azure, consultez le [livre électronique Systèmes de fichiers parallèles sur Azure](https://blogs.msdn.microsoft.com/azurecat/2018/06/11/azurecat-ebook-parallel-virtual-file-systems-on-microsoft-azure/)</span><span class="sxs-lookup"><span data-stu-id="fd306-159">For more information comparing Lustre, GlusterFS, and BeeGFS on Azure, review the [Parallel Files Systems on Azure eBook](https://blogs.msdn.microsoft.com/azurecat/2018/06/11/azurecat-ebook-parallel-virtual-file-systems-on-microsoft-azure/)</span></span>

### <a name="networking"></a><span data-ttu-id="fd306-160">Mise en réseau</span><span class="sxs-lookup"><span data-stu-id="fd306-160">Networking</span></span>

<span data-ttu-id="fd306-161">Les machines virtuelles H16r, H16mr, A8 et A9 peuvent se connecter à un réseau RDMA back-end à débit élevé.</span><span class="sxs-lookup"><span data-stu-id="fd306-161">H16r, H16mr, A8, and A9 VMs can connect to a high throughput back-end RDMA network.</span></span> <span data-ttu-id="fd306-162">Ce réseau peut améliorer les performances d’applications parallèles étroitement liées s’exécutant sous Microsoft MPI ou Intel MPI.</span><span class="sxs-lookup"><span data-stu-id="fd306-162">This network can improve the performance of tightly coupled parallel applications running under Microsoft MPI or Intel MPI.</span></span>

- [<span data-ttu-id="fd306-163">Instances compatibles RDMA</span><span class="sxs-lookup"><span data-stu-id="fd306-163">RDMA Capable Instances</span></span>](/azure/virtual-machines/windows/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context#rdma-capable-instances)
- [<span data-ttu-id="fd306-164">Réseau virtuel</span><span class="sxs-lookup"><span data-stu-id="fd306-164">Virtual Network</span></span>](/azure/virtual-network/virtual-networks-overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="fd306-165">ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="fd306-165">ExpressRoute</span></span>](/azure/expressroute/expressroute-introduction?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="management"></a><span data-ttu-id="fd306-166">gestion</span><span class="sxs-lookup"><span data-stu-id="fd306-166">Management</span></span>

### <a name="do-it-yourself"></a><span data-ttu-id="fd306-167">À faire soi-même</span><span class="sxs-lookup"><span data-stu-id="fd306-167">Do-it-yourself</span></span>

<span data-ttu-id="fd306-168">La génération d’un système HPC à partir de zéro sur Azure offre une grande flexibilité, mais elle nécessite souvent beaucoup de maintenance.</span><span class="sxs-lookup"><span data-stu-id="fd306-168">Building an HPC system from scratch on Azure offers a significant amount of flexibility, but is often very maintenance intensive.</span></span>  

1. <span data-ttu-id="fd306-169">Configurez votre environnement de cluster dans des machines virtuelles Azure ou dans des [groupes de machines virtuelles identiques](/azure/virtual-machine-scale-sets/overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context).</span><span class="sxs-lookup"><span data-stu-id="fd306-169">Set up your own cluster environment in Azure virtual machines or [virtual machine scale sets](/azure/virtual-machine-scale-sets/overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context).</span></span>
2. <span data-ttu-id="fd306-170">Utilisez les modèles Azure Resource Manager pour déployer des [gestionnaires de charge de travail](#workload-managers) principaux, des infrastructures et des [applications](#hpc-applications).</span><span class="sxs-lookup"><span data-stu-id="fd306-170">Use Azure Resource Manager templates to deploy leading [workload managers](#workload-managers), infrastructure, and [applications](#hpc-applications).</span></span>
3. <span data-ttu-id="fd306-171">Choisissez des [tailles de machine virtuelle](#compute) HPC et GPU qui comprennent du matériel spécialisé et des connexions réseau pour les charges de travail MPI ou GPU.</span><span class="sxs-lookup"><span data-stu-id="fd306-171">Choose HPC and GPU [VM sizes](#compute) that include specialized hardware and network connections for MPI or GPU workloads.</span></span>
4. <span data-ttu-id="fd306-172">Ajoutez du [stockage de haute performance](#storage) aux charges de travail intensives d’E/S.</span><span class="sxs-lookup"><span data-stu-id="fd306-172">Add [high performance storage](#storage) for I/O-intensive workloads.</span></span>

### <a name="hybrid-and-cloud-bursting"></a><span data-ttu-id="fd306-173">Éclatement hybride et cloud bursting</span><span class="sxs-lookup"><span data-stu-id="fd306-173">Hybrid and cloud Bursting</span></span>

<span data-ttu-id="fd306-174">Si vous avez un système HPC local existant que vous voulez connecter à Azure, il existe de nombreuses ressources pour vous aider à commencer.</span><span class="sxs-lookup"><span data-stu-id="fd306-174">If you have an existing on-premise HPC system that you'd like to connect to Azure, there are a number of resources to help get you started.</span></span>

<span data-ttu-id="fd306-175">Tout d’abord, consultez l’article [Options pour la connexion d’un réseau local à Azure](/azure/architecture/reference-architectures/hybrid-networking/?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) de la documentation.</span><span class="sxs-lookup"><span data-stu-id="fd306-175">First, review the [Options for connecting an on-premises network to Azure](/azure/architecture/reference-architectures/hybrid-networking/?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) article in the documentation.</span></span>  <span data-ttu-id="fd306-176">Alors, vous voudrez peut-être des informations sur les options de connectivité suivantes :</span><span class="sxs-lookup"><span data-stu-id="fd306-176">From there, you may want information on these connectivity options:</span></span>

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/vpn?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/vpn.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="fd306-177">Connecter un réseau local à Azure à l’aide d’une passerelle VPN</span><span class="sxs-lookup"><span data-stu-id="fd306-177">Connect an on-premises network to Azure using a VPN gateway</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="fd306-178">Cette architecture de référence montre comment étendre un réseau local à Azure à l’aide d’un réseau privé virtuel (VPN) site à site.</span><span class="sxs-lookup"><span data-stu-id="fd306-178">This reference architecture shows how to extend an on-premises network to Azure, using a site-to-site virtual private network (VPN).</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/expressroute?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/expressroute.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="fd306-179">Connecter un réseau local à Azure à l’aide d’ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="fd306-179">Connect an on-premises network to Azure using ExpressRoute</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="fd306-180">Les connexions ExpressRoute utilisent une connexion privée et dédiée via un fournisseur de connectivité tiers.</span><span class="sxs-lookup"><span data-stu-id="fd306-180">ExpressRoute connections use a private, dedicated connection through a third-party connectivity provider.</span></span> <span data-ttu-id="fd306-181">La connexion privée étend votre réseau local à Azure.</span><span class="sxs-lookup"><span data-stu-id="fd306-181">The private connection extends your on-premises network into Azure.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/expressroute-vpn-failover?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/expressroute-vpn-failover.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="fd306-182">Connecter un réseau local à Azure à l’aide d’ExpressRoute avec basculement VPN</span><span class="sxs-lookup"><span data-stu-id="fd306-182">Connect an on-premises network to Azure using ExpressRoute with VPN failover</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="fd306-183">Implémentez une architecture réseau de site à site sécurisée et hautement disponible qui s’étend sur un réseau virtuel Azure et un réseau local connecté à l’aide d’ExpressRoute avec basculement de passerelle VPN.</span><span class="sxs-lookup"><span data-stu-id="fd306-183">Implement a highly available and secure site-to-site network architecture that spans an Azure virtual network and an on-premises network connected using ExpressRoute with VPN gateway failover.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

<span data-ttu-id="fd306-184">Une fois la connectivité réseau établie de manière sécurisée, vous pouvez commencer à utiliser des ressources de calcul cloud à la demande avec les fonctionnalités d’éclatement de votre [gestionnaire de charges de travail](#workload-managers) existant.</span><span class="sxs-lookup"><span data-stu-id="fd306-184">Once network connectivity is securely established, you can start using cloud compute resources on-demand with the bursting capabilities of your existing [workload manager](#workload-managers).</span></span>

### <a name="marketplace-solutions"></a><span data-ttu-id="fd306-185">Solutions de la Place de marché</span><span class="sxs-lookup"><span data-stu-id="fd306-185">Marketplace solutions</span></span>

<span data-ttu-id="fd306-186">Un certain nombre de gestionnaires de charges de travail sont proposés dans la [Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/).</span><span class="sxs-lookup"><span data-stu-id="fd306-186">There are a number of workload managers offered in the [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/).</span></span>

- [<span data-ttu-id="fd306-187">HPC basé sur RogueWave CentOS</span><span class="sxs-lookup"><span data-stu-id="fd306-187">RogueWave CentOS-based HPC</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/RogueWave.CentOSbased73HPC?tab=Overview)
- [<span data-ttu-id="fd306-188">SUSE Linux Enterprise Server (SLES)</span><span class="sxs-lookup"><span data-stu-id="fd306-188">SUSE Linux Enterprise Server for HPC</span></span>](https://azure.microsoft.com/marketplace/partners/suse/suselinuxenterpriseserver12optimizedforhighperformancecompute/)
- [<span data-ttu-id="fd306-189">TIBCO Grid Server Engine</span><span class="sxs-lookup"><span data-stu-id="fd306-189">TIBCO Grid Server Engine</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/tibco-software.gridserverlinuxengine?tab=Overview)
- [<span data-ttu-id="fd306-190">Machine virtuelle de science des données Azure pour Windows et Linux</span><span class="sxs-lookup"><span data-stu-id="fd306-190">Azure Data Science VM for Windows and Linux</span></span>](/azure/machine-learning/data-science-virtual-machine/overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="fd306-191">D3View</span><span class="sxs-lookup"><span data-stu-id="fd306-191">D3View</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/xfinityinc.d3view-v5?tab=Overview)
- [<span data-ttu-id="fd306-192">UberCloud</span><span class="sxs-lookup"><span data-stu-id="fd306-192">UberCloud</span></span>](https://azure.microsoft.com/search/marketplace/?q=ubercloud)

### <a name="azure-batch"></a><span data-ttu-id="fd306-193">Azure Batch</span><span class="sxs-lookup"><span data-stu-id="fd306-193">Azure Batch</span></span>

<span data-ttu-id="fd306-194">[Azure Batch](/azure/batch/batch-technical-overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) est une plateforme qui permet d’exécuter efficacement des applications de calcul haute performance (HPC) en parallèle et à grande échelle dans le cloud.</span><span class="sxs-lookup"><span data-stu-id="fd306-194">[Azure Batch](/azure/batch/batch-technical-overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) is a platform service for running large-scale parallel and high-performance computing (HPC) applications efficiently in the cloud.</span></span> <span data-ttu-id="fd306-195">Azure Batch planifie les travaux nécessitant une grande quantité de ressources système à exécuter sur un pool géré de machines virtuelles. Il peut mettre automatiquement à l’échelle les ressources de calcul pour répondre aux besoins de vos travaux.</span><span class="sxs-lookup"><span data-stu-id="fd306-195">Azure Batch schedules compute-intensive work to run on a managed pool of virtual machines, and can automatically scale compute resources to meet the needs of your jobs.</span></span>

<span data-ttu-id="fd306-196">Les fournisseurs et développeurs SaaS peuvent utiliser les outils et kits de développement logiciel (SDK) pour intégrer des applications HPC ou des charges de travail de conteneur dans Azure, stocker des données dans Azure et générer des pipelines d’exécution du travail.</span><span class="sxs-lookup"><span data-stu-id="fd306-196">SaaS providers or developers can use the Batch SDKs and tools to integrate HPC applications or container workloads with Azure, stage data to Azure, and build job execution pipelines.</span></span>

### <a name="azure-cyclecloud"></a><span data-ttu-id="fd306-197">Azure CycleCloud</span><span class="sxs-lookup"><span data-stu-id="fd306-197">Azure CycleCloud</span></span>

<span data-ttu-id="fd306-198">[Azure CycleCloud](/azure/cyclecloud/?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) La manière la plus simple de gérer les charges de travail HPC à l’aide d’un planificateur (tel que Slurm, Grid Engine, HPC Pack, HTCondor, LSF, PBS Pro ou Symphony), sur Azure</span><span class="sxs-lookup"><span data-stu-id="fd306-198">[Azure CycleCloud](/azure/cyclecloud/?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) Provides the simplest way to manage HPC workloads using any scheduler (like Slurm, Grid Engine, HPC Pack, HTCondor, LSF, PBS Pro, or Symphony), on Azure</span></span>

<span data-ttu-id="fd306-199">CycleCloud vous permet d’effectuer les opérations suivantes :</span><span class="sxs-lookup"><span data-stu-id="fd306-199">CycleCloud allows you to:</span></span>

- <span data-ttu-id="fd306-200">Déployer des clusters complets et d’autres ressources, notamment le planificateur, les machines virtuelles de calcul, le stockage, la mise en réseau et le cache</span><span class="sxs-lookup"><span data-stu-id="fd306-200">Deploy full clusters and other resources, including scheduler, compute VMs, storage, networking, and cache</span></span>
- <span data-ttu-id="fd306-201">Orchestrer des workflows liés aux travaux, aux données et au cloud</span><span class="sxs-lookup"><span data-stu-id="fd306-201">Orchestrate job, data, and cloud workflows</span></span>
- <span data-ttu-id="fd306-202">Donner aux administrateurs un contrôle total sur les utilisateurs qui peuvent exécuter des travaux, l’emplacement de ces travaux et leur coût</span><span class="sxs-lookup"><span data-stu-id="fd306-202">Give admins full control over which users can run jobs, as well as where and at what cost</span></span>
- <span data-ttu-id="fd306-203">Personnaliser et optimiser les clusters par le biais des fonctionnalités de stratégie et de gouvernance avancées, notamment le contrôle des coûts, l’intégration d’Active Directory, la supervision et la création de rapports</span><span class="sxs-lookup"><span data-stu-id="fd306-203">Customize and optimize clusters through advanced policy and governance features, including cost controls, Active Directory integration, monitoring, and reporting</span></span>
- <span data-ttu-id="fd306-204">Utiliser votre planificateur de travaux et vos applications actuels sans modification</span><span class="sxs-lookup"><span data-stu-id="fd306-204">Use your current job scheduler and applications without modification</span></span>
- <span data-ttu-id="fd306-205">Tirer parti de la mise à l’échelle automatique intégrée et des architectures de référence testées sur le terrain pour un large éventail de secteurs et de charges de travail HPC</span><span class="sxs-lookup"><span data-stu-id="fd306-205">Take advantage of built-in autoscaling and battle-tested reference architectures for a wide range of HPC workloads and industries</span></span>

### <a name="workload-managers"></a><span data-ttu-id="fd306-206">Gestionnaires de charges de travail</span><span class="sxs-lookup"><span data-stu-id="fd306-206">Workload managers</span></span>

<span data-ttu-id="fd306-207">Voici quelques exemples de cluster et de gestionnaires de charges de travail qui peuvent s’exécuter dans une architecture Azure.</span><span class="sxs-lookup"><span data-stu-id="fd306-207">The following are examples of cluster and workload managers that can run in Azure infrastructure.</span></span> <span data-ttu-id="fd306-208">Créer des clusters autonomes dans des machines virtuelles Azure ou effectuez un burst dans les machines virtuelles Azure depuis un cluster local.</span><span class="sxs-lookup"><span data-stu-id="fd306-208">Create stand-alone clusters in Azure VMs or burst to Azure VMs from an on-premises cluster.</span></span>

- [<span data-ttu-id="fd306-209">Alces Flight Compute</span><span class="sxs-lookup"><span data-stu-id="fd306-209">Alces Flight Compute</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/alces-flight-limited.alces-flight-compute-solo?tab=Overview)
- [<span data-ttu-id="fd306-210">TIBCO DataSynapse GridServer</span><span class="sxs-lookup"><span data-stu-id="fd306-210">TIBCO DataSynapse GridServer</span></span>](https://azure.microsoft.com/blog/tibco-datasynapse-comes-to-the-azure-marketplace/)
- [<span data-ttu-id="fd306-211">Bright Cluster Manager</span><span class="sxs-lookup"><span data-stu-id="fd306-211">Bright Cluster Manager</span></span>](http://www.brightcomputing.com/technology-partners/microsoft)
- [<span data-ttu-id="fd306-212">IBM Spectrum Symphony and Symphony LSF</span><span class="sxs-lookup"><span data-stu-id="fd306-212">IBM Spectrum Symphony and Symphony LSF</span></span>](https://azure.microsoft.com/blog/ibm-and-microsoft-azure-support-spectrum-symphony-and-spectrum-lsf/)
- [<span data-ttu-id="fd306-213">PBS Pro</span><span class="sxs-lookup"><span data-stu-id="fd306-213">PBS Pro</span></span>](http://pbspro.org)
- [<span data-ttu-id="fd306-214">Altair</span><span class="sxs-lookup"><span data-stu-id="fd306-214">Altair</span></span>](http://www.altair.com/)
- [<span data-ttu-id="fd306-215">Rescale</span><span class="sxs-lookup"><span data-stu-id="fd306-215">Rescale</span></span>](https://www.rescale.com/azure/)
- [<span data-ttu-id="fd306-216">Microsoft HPC Pack</span><span class="sxs-lookup"><span data-stu-id="fd306-216">Microsoft HPC Pack</span></span>](https://technet.microsoft.com/library/mt744885.aspx)
  - [<span data-ttu-id="fd306-217">HPC Pack pour Windows</span><span class="sxs-lookup"><span data-stu-id="fd306-217">HPC Pack for Windows</span></span>](/azure/virtual-machines/windows/hpcpack-cluster-options?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
  - [<span data-ttu-id="fd306-218">HPC Pack pour Linux</span><span class="sxs-lookup"><span data-stu-id="fd306-218">HPC Pack for Linux</span></span>](/azure/virtual-machines/linux/hpcpack-cluster-options?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

#### <a name="containers"></a><span data-ttu-id="fd306-219">Containers</span><span class="sxs-lookup"><span data-stu-id="fd306-219">Containers</span></span>

<span data-ttu-id="fd306-220">Les conteneurs peuvent également être utilisés pour gérer certaines charges de travail HPC.</span><span class="sxs-lookup"><span data-stu-id="fd306-220">Containers can also be used to manage some HPC workloads.</span></span>  <span data-ttu-id="fd306-221">Des services comme Azure Kubernetes Service (AKS) simplifie le déploiement d’un cluster Kubernetes managé dans Azure.</span><span class="sxs-lookup"><span data-stu-id="fd306-221">Services like the Azure Kubernetes Service (AKS) makes it simple to deploy a managed Kubernetes cluster in Azure.</span></span>

- [<span data-ttu-id="fd306-222">Azure Kubernetes Service (AKS)</span><span class="sxs-lookup"><span data-stu-id="fd306-222">Azure Kubernetes Service (AKS)</span></span>](/azure/aks/intro-kubernetes?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="fd306-223">Container Registry</span><span class="sxs-lookup"><span data-stu-id="fd306-223">Container Registry</span></span>](/azure/container-registry/container-registry-intro?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="cost-management"></a><span data-ttu-id="fd306-224">la gestion des coûts ;</span><span class="sxs-lookup"><span data-stu-id="fd306-224">Cost management</span></span>

<span data-ttu-id="fd306-225">Il existe différentes manières de gérer vos coûts HPC sur Azure.</span><span class="sxs-lookup"><span data-stu-id="fd306-225">Managing your HPC cost on Azure can be done through a few different ways.</span></span>  <span data-ttu-id="fd306-226">Vérifiez que vous avez consulté les [options d’achat Azure](https://azure.microsoft.com/pricing/purchase-options/) afin de trouver la méthode qui convient le mieux à votre organisation.</span><span class="sxs-lookup"><span data-stu-id="fd306-226">Ensure you've reviewed the [Azure purchasing options](https://azure.microsoft.com/pricing/purchase-options/) to find the method that works best for your organization.</span></span>

<span data-ttu-id="fd306-227">Les [machines virtuelles basse priorité](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-use-low-priority?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) vous permettent de disposer de notre capacité inutilisée en réalisant des économies significatives.</span><span class="sxs-lookup"><span data-stu-id="fd306-227">[Low priority VMs](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-use-low-priority?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) allow you to take advantage of our unutilized capacity at a significant cost savings.</span></span>

## <a name="security"></a><span data-ttu-id="fd306-228">Sécurité</span><span class="sxs-lookup"><span data-stu-id="fd306-228">Security</span></span>

<span data-ttu-id="fd306-229">Pour obtenir une vue d’ensemble des bonnes pratiques de sécurité sur Azure, consultez la [documentation sur la sécurité Azure](/azure/security/azure-security?context=/azure/architecture/topics/high-performance-computing/context/hpc-context).</span><span class="sxs-lookup"><span data-stu-id="fd306-229">For an overview of security best practices on Azure, review the [Azure Security Documentation](/azure/security/azure-security?context=/azure/architecture/topics/high-performance-computing/context/hpc-context).</span></span>  

<span data-ttu-id="fd306-230">Outre les configurations réseau disponibles dans la section [Cloud bursting](#hybrid-and-cloud-bursting), vous pouvez souhaiter implémenter une configuration hub/spoke pour isoler vos ressources de calcul :</span><span class="sxs-lookup"><span data-stu-id="fd306-230">In addition to the network configurations available in the [Cloud Bursting](#hybrid-and-cloud-bursting) section, you may want to implement a hub/spoke configuration to isolate your compute resources:</span></span>

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/hub-spoke.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="fd306-231">Implémenter une topologie de réseau hub-and-spoke dans Azure</span><span class="sxs-lookup"><span data-stu-id="fd306-231">Implement a hub-spoke network topology in Azure</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="fd306-232">Le hub est un réseau virtuel (VNet) dans Azure qui centralise la connectivité à votre réseau local.</span><span class="sxs-lookup"><span data-stu-id="fd306-232">The hub is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="fd306-233">Les rayons (spokes) sont des réseaux virtuels qui s’homologuent avec le hub et qui peuvent être utilisés pour isoler les charges de travail.</span><span class="sxs-lookup"><span data-stu-id="fd306-233">The spokes are VNets that peer with the hub, and can be used to isolate workloads.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/shared-services?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/shared-services.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="fd306-234">Implémentation d’une topologie de réseau hub-and-spoke avec des services partagés dans Azure</span><span class="sxs-lookup"><span data-stu-id="fd306-234">Implement a hub-spoke network topology with shared services in Azure</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="fd306-235">Cette architecture de référence s’appuie sur l’architecture de référence hub-and-spoke de manière à inclure dans le hub des services partagés qui peuvent être utilisés par tous les spokes.</span><span class="sxs-lookup"><span data-stu-id="fd306-235">This reference architecture builds on the hub-spoke reference architecture to include shared services in the hub that can be consumed by all spokes.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="hpc-applications"></a><span data-ttu-id="fd306-236">Applications HPC</span><span class="sxs-lookup"><span data-stu-id="fd306-236">HPC applications</span></span>

<span data-ttu-id="fd306-237">Exécuter des applications HPC commerciales ou personnalisées dans Azure.</span><span class="sxs-lookup"><span data-stu-id="fd306-237">Run custom or commercial HPC applications in Azure.</span></span> <span data-ttu-id="fd306-238">Plusieurs exemples dans cette section ont été testés et se montrent efficaces pour la mise à l’échelle avec des machines virtuelles ou des cœurs de calcul supplémentaires.</span><span class="sxs-lookup"><span data-stu-id="fd306-238">Several examples in this section are benchmarked to scale efficiently with additional VMs or compute cores.</span></span> <span data-ttu-id="fd306-239">Visitez la [Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace) pour obtenir des solutions prêtes au déploiement.</span><span class="sxs-lookup"><span data-stu-id="fd306-239">Visit the [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace) for ready-to-deploy solutions.</span></span>

> [!NOTE]
> <span data-ttu-id="fd306-240">Vérifiez auprès du fournisseur de toute application commerciale les questions de licence ou toute autre restriction relative à l’exécution dans le cloud.</span><span class="sxs-lookup"><span data-stu-id="fd306-240">Check with the vendor of any commercial application for licensing or other restrictions for running in the cloud.</span></span> <span data-ttu-id="fd306-241">Tous les fournisseurs ne proposent pas le paiement à l'utilisation pour les licences.</span><span class="sxs-lookup"><span data-stu-id="fd306-241">Not all vendors offer pay-as-you-go licensing.</span></span> <span data-ttu-id="fd306-242">Vous aurez peut-être besoin d’un serveur de licences dans le cloud pour votre solution ou de vous connecter à un serveur de licences sur site.</span><span class="sxs-lookup"><span data-stu-id="fd306-242">You might need a licensing server in the cloud for your solution, or connect to an on-premises license server.</span></span>

### <a name="engineering-applications"></a><span data-ttu-id="fd306-243">Applications d’ingénierie</span><span class="sxs-lookup"><span data-stu-id="fd306-243">Engineering applications</span></span>

- [<span data-ttu-id="fd306-244">Altair RADIOSS</span><span class="sxs-lookup"><span data-stu-id="fd306-244">Altair RADIOSS</span></span>](https://azure.microsoft.com/blog/availability-of-altair-radioss-rdma-on-microsoft-azure/)
- [<span data-ttu-id="fd306-245">ANSYS CFD</span><span class="sxs-lookup"><span data-stu-id="fd306-245">ANSYS CFD</span></span>](https://azure.microsoft.com/blog/ansys-cfd-and-microsoft-azure-perform-the-best-hpc-scalability-in-the-cloud/)
- [<span data-ttu-id="fd306-246">MATLAB Distributed Computing Server</span><span class="sxs-lookup"><span data-stu-id="fd306-246">MATLAB Distributed Computing Server</span></span>](/azure/virtual-machines/windows/matlab-mdcs-cluster?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="fd306-247">StarCCM +</span><span class="sxs-lookup"><span data-stu-id="fd306-247">StarCCM+</span></span>](https://blogs.msdn.microsoft.com/azurecat/2017/07/07/run-star-ccm-in-an-azure-hpc-cluster/)
- [<span data-ttu-id="fd306-248">OpenFOAM</span><span class="sxs-lookup"><span data-stu-id="fd306-248">OpenFOAM</span></span>](https://simulation.azure.com/casestudies/Team-182-ABB-UC-Final.pdf)

### <a name="graphics-and-rendering"></a><span data-ttu-id="fd306-249">Graphiques et rendu</span><span class="sxs-lookup"><span data-stu-id="fd306-249">Graphics and rendering</span></span>

- <span data-ttu-id="fd306-250">[Autodesk Maya, 3ds Max et Arnold](/azure/batch/batch-rendering-service?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) sur Azure Batch</span><span class="sxs-lookup"><span data-stu-id="fd306-250">[Autodesk Maya, 3ds Max, and Arnold](/azure/batch/batch-rendering-service?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) on Azure Batch</span></span>

### <a name="ai-and-deep-learning"></a><span data-ttu-id="fd306-251">Intelligence artificielle et apprentissage approfondi</span><span class="sxs-lookup"><span data-stu-id="fd306-251">AI and deep learning</span></span>

- [<span data-ttu-id="fd306-252">Microsoft Cognitive Toolkit</span><span class="sxs-lookup"><span data-stu-id="fd306-252">Microsoft Cognitive Toolkit</span></span>](/cognitive-toolkit/cntk-on-azure)
- [<span data-ttu-id="fd306-253">Machine virtuelle d’apprentissage profond</span><span class="sxs-lookup"><span data-stu-id="fd306-253">Deep Learning VM</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-ads.dsvm-deep-learning)
- [<span data-ttu-id="fd306-254">Recettes Batch Shipyard pour l’apprentissage approfondi</span><span class="sxs-lookup"><span data-stu-id="fd306-254">Batch Shipyard recipes for deep learning</span></span>](https://github.com/Azure/batch-shipyard/tree/master/recipes#deeplearning)

### <a name="mpi-providers"></a><span data-ttu-id="fd306-255">Fournisseurs MPI</span><span class="sxs-lookup"><span data-stu-id="fd306-255">MPI Providers</span></span>

- [<span data-ttu-id="fd306-256">Microsoft MPI</span><span class="sxs-lookup"><span data-stu-id="fd306-256">Microsoft MPI</span></span>](/message-passing-interface/microsoft-mpi)

## <a name="remote-visualization"></a><span data-ttu-id="fd306-257">Visualisation à distance</span><span class="sxs-lookup"><span data-stu-id="fd306-257">Remote visualization</span></span>

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/infrastructure/linux-vdi-citrix?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/infrastructure/media/azure-citrix-sample-diagram.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="fd306-258">Bureaux virtuels Linux avec Citrix</span><span class="sxs-lookup"><span data-stu-id="fd306-258">Linux virtual desktops with Citrix</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="fd306-259">Générez un environnement VDI pour les bureaux Linux à l’aide de Citrix sur Azure.</span><span class="sxs-lookup"><span data-stu-id="fd306-259">Build a VDI environment for Linux Desktops using Citrix on Azure.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="performance-benchmarks"></a><span data-ttu-id="fd306-260">Test d’évaluation des performances</span><span class="sxs-lookup"><span data-stu-id="fd306-260">Performance Benchmarks</span></span>

- [<span data-ttu-id="fd306-261">Test d’évaluation des calculs</span><span class="sxs-lookup"><span data-stu-id="fd306-261">Compute benchmarks</span></span>](/azure/virtual-machines/windows/compute-benchmark-scores?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="customer-stories"></a><span data-ttu-id="fd306-262">Témoignages client</span><span class="sxs-lookup"><span data-stu-id="fd306-262">Customer stories</span></span>

<span data-ttu-id="fd306-263">Un certain nombre de clients ont connu beaucoup de succès grâce à l’utilisation d’Azure pour leurs charges de travail HPC.</span><span class="sxs-lookup"><span data-stu-id="fd306-263">There are a number of customers who have seen great success by using Azure for their HPC workloads.</span></span>  <span data-ttu-id="fd306-264">Voici quelques-unes de ces études de cas clients :</span><span class="sxs-lookup"><span data-stu-id="fd306-264">You can find a few of these customer case studies below:</span></span>

- [<span data-ttu-id="fd306-265">ANEO</span><span class="sxs-lookup"><span data-stu-id="fd306-265">ANEO</span></span>](https://customers.microsoft.com/story/it-provider-finds-highly-scalable-cloud-based-hpc-redu)
- [<span data-ttu-id="fd306-266">AXA Global P&amp;C</span><span class="sxs-lookup"><span data-stu-id="fd306-266">AXA Global P&C</span></span>](https://customers.microsoft.com/story/axa-global-p-and-c)
- [<span data-ttu-id="fd306-267">Axioma</span><span class="sxs-lookup"><span data-stu-id="fd306-267">Axioma</span></span>](https://customers.microsoft.com/story/axioma-delivers-fintechs-first-born-in-the-cloud-multi-asset-class-enterprise-risk-solution)
- [<span data-ttu-id="fd306-268">d3View</span><span class="sxs-lookup"><span data-stu-id="fd306-268">d3View</span></span>](https://customers.microsoft.com/story/big-data-solution-provider-adopts-new-cloud-gains-thou)
- [<span data-ttu-id="fd306-269">EFS</span><span class="sxs-lookup"><span data-stu-id="fd306-269">EFS</span></span>](https://customers.microsoft.com/story/efs-professionalservices-azure)
- [<span data-ttu-id="fd306-270">Hymans Robertson</span><span class="sxs-lookup"><span data-stu-id="fd306-270">Hymans Robertson</span></span>](https://customers.microsoft.com/story/hymans-robertson)
- [<span data-ttu-id="fd306-271">MetLife</span><span class="sxs-lookup"><span data-stu-id="fd306-271">MetLife</span></span>](https://enterprise.microsoft.com/customer-story/industries/insurance/metlife/)
- [<span data-ttu-id="fd306-272">Microsoft Research</span><span class="sxs-lookup"><span data-stu-id="fd306-272">Microsoft Research</span></span>](https://customers.microsoft.com/doclink/fast-lmm-and-windows-azure-put-genetics-research-on-fa)
- [<span data-ttu-id="fd306-273">Milliman</span><span class="sxs-lookup"><span data-stu-id="fd306-273">Milliman</span></span>](https://customers.microsoft.com/story/actuarial-firm-works-to-transform-insurance-industry-w)
- [<span data-ttu-id="fd306-274">Mitsubishi UFJ Securities International</span><span class="sxs-lookup"><span data-stu-id="fd306-274">Mitsubishi UFJ Securities International</span></span>](https://customers.microsoft.com/story/powering-risk-compute-grids-in-the-cloud)
- [<span data-ttu-id="fd306-275">NeuroInitiative</span><span class="sxs-lookup"><span data-stu-id="fd306-275">NeuroInitiative</span></span>](https://customers.microsoft.com/story/neuroinitiative-health-provider-azure)
- [<span data-ttu-id="fd306-276">Schlumberger</span><span class="sxs-lookup"><span data-stu-id="fd306-276">Schlumberger</span></span>](https://azure.microsoft.com/blog/big-compute-for-large-engineering-simulations)
- [<span data-ttu-id="fd306-277">Towers Watson</span><span class="sxs-lookup"><span data-stu-id="fd306-277">Towers Watson</span></span>](https://customers.microsoft.com/story/insurance-tech-provider-delivers-disruptive-solutions)

## <a name="other-important-information"></a><span data-ttu-id="fd306-278">Autres informations importantes</span><span class="sxs-lookup"><span data-stu-id="fd306-278">Other important information</span></span>

- <span data-ttu-id="fd306-279">Vérifiez que votre [quotas de processeurs virtuels](/azure/virtual-machines/linux/quotas?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) a été augmentée avant de tenter d’exécuter des charges de travail à grande échelle.</span><span class="sxs-lookup"><span data-stu-id="fd306-279">Ensure your [vCPU quota](/azure/virtual-machines/linux/quotas?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) has been increased before attempting to run large-scale workloads.</span></span>

## <a name="next-steps"></a><span data-ttu-id="fd306-280">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="fd306-280">Next steps</span></span>

<span data-ttu-id="fd306-281">Pour connaître les dernières annonces, consultez :</span><span class="sxs-lookup"><span data-stu-id="fd306-281">For the latest announcements, see:</span></span>

- [<span data-ttu-id="fd306-282">Blog de l’équipe Microsoft HPC et Batch</span><span class="sxs-lookup"><span data-stu-id="fd306-282">Microsoft HPC and Batch team blog</span></span>](http://blogs.technet.com/b/windowshpc/)
- <span data-ttu-id="fd306-283">Consultez le [blog Azure](https://azure.microsoft.com/blog/tag/hpc/).</span><span class="sxs-lookup"><span data-stu-id="fd306-283">Visit the [Azure blog](https://azure.microsoft.com/blog/tag/hpc/).</span></span>

### <a name="microsoft-batch-examples"></a><span data-ttu-id="fd306-284">Exemples Microsoft Batch</span><span class="sxs-lookup"><span data-stu-id="fd306-284">Microsoft Batch Examples</span></span>

<span data-ttu-id="fd306-285">Les tutoriels suivants vous fourniront des informations détaillées sur l’exécution d’applications sur Microsoft Batch</span><span class="sxs-lookup"><span data-stu-id="fd306-285">These tutorials will provide you with details on running applications on Microsoft Batch</span></span>

- [<span data-ttu-id="fd306-286">Commencer à développer avec Batch</span><span class="sxs-lookup"><span data-stu-id="fd306-286">Get started developing with Batch</span></span>](/azure/batch/quick-run-dotnet?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="fd306-287">Utiliser les exemples de code Azure Batch</span><span class="sxs-lookup"><span data-stu-id="fd306-287">Use Azure Batch code samples</span></span>](https://github.com/Azure/azure-batch-samples)
- [<span data-ttu-id="fd306-288">Utiliser des machines virtuelles de faible priorité avec Batch</span><span class="sxs-lookup"><span data-stu-id="fd306-288">Use low-priority VMs with Batch</span></span>](/azure/batch/batch-low-pri-vms?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="fd306-289">Exécuter des charges de travail HPC en conteneur avec Batch Shipyard</span><span class="sxs-lookup"><span data-stu-id="fd306-289">Run containerized HPC workloads with Batch Shipyard</span></span>](https://github.com/Azure/batch-shipyard)
- [<span data-ttu-id="fd306-290">Exécuter des charges de travail R parallèles sur Batch</span><span class="sxs-lookup"><span data-stu-id="fd306-290">Run parallel R workloads on Batch</span></span>](https://github.com/Azure/doAzureParallel)
- [<span data-ttu-id="fd306-291">Exécuter des travaux Spark à la demande sur Batch</span><span class="sxs-lookup"><span data-stu-id="fd306-291">Run on-demand Spark jobs on Batch</span></span>](https://github.com/Azure/aztk)
- [<span data-ttu-id="fd306-292">Utiliser des machines virtuelles nécessitant beaucoup de ressources système dans des pools Batch</span><span class="sxs-lookup"><span data-stu-id="fd306-292">Use compute-intensive VMs in Batch pools</span></span>](/azure/batch/batch-pool-compute-intensive-sizes?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)