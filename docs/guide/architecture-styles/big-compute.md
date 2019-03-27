---
title: Style d’architecture Big Compute
titleSuffix: Azure Application Architecture Guide
description: Décrit les avantages, les inconvénients et les bonnes pratiques des architectures Big Compute sur Azure.
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19, HPC
ms.openlocfilehash: 56bd2ce010b56880e769ada4c6397391a73bbd1e
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58244620"
---
# <a name="big-compute-architecture-style"></a><span data-ttu-id="00a89-103">Style d’architecture Big Compute</span><span class="sxs-lookup"><span data-stu-id="00a89-103">Big compute architecture style</span></span>

<span data-ttu-id="00a89-104">Le terme *Big Compute* décrit les charges de travail à grande échelle qui nécessitent plusieurs centaines ou milliers de cœurs.</span><span class="sxs-lookup"><span data-stu-id="00a89-104">The term *big compute* describes large-scale workloads that require a large number of cores, often numbering in the hundreds or thousands.</span></span> <span data-ttu-id="00a89-105">Les scénarios d’utilisation possibles incluent notamment le rendu d’images, la dynamique des fluides, la modélisation des risques financiers, la prospection pétrolière, la conception de médicaments et l’analyse technique des contraintes.</span><span class="sxs-lookup"><span data-stu-id="00a89-105">Scenarios include image rendering, fluid dynamics, financial risk modeling, oil exploration, drug design, and engineering stress analysis, among others.</span></span>

![Diagramme logique pour le style d’architecture Big Compute](./images/big-compute-logical.png)

<span data-ttu-id="00a89-107">Voici quelques caractéristiques types des applications Big Compute :</span><span class="sxs-lookup"><span data-stu-id="00a89-107">Here are some typical characteristics of big compute applications:</span></span>

- <span data-ttu-id="00a89-108">Le travail peut être fractionné en tâches discrètes qui sont exécutables sur plusieurs cœurs simultanément.</span><span class="sxs-lookup"><span data-stu-id="00a89-108">The work can be split into discrete tasks, which can be run across many cores simultaneously.</span></span>
- <span data-ttu-id="00a89-109">Chaque tâche est un processus fini.</span><span class="sxs-lookup"><span data-stu-id="00a89-109">Each task is finite.</span></span> <span data-ttu-id="00a89-110">Elle accepte une entrée, effectue un traitement spécifique, puis génère la sortie correspondante.</span><span class="sxs-lookup"><span data-stu-id="00a89-110">It takes some input, does some processing, and produces output.</span></span> <span data-ttu-id="00a89-111">L’ensemble de l’application s’exécute pendant une durée limitée (nombre de minutes ou de jours).</span><span class="sxs-lookup"><span data-stu-id="00a89-111">The entire application runs for a finite amount of time (minutes to days).</span></span> <span data-ttu-id="00a89-112">Un modèle courant consiste à approvisionner un grand nombre de cœurs en rafale, puis à revenir à zéro une fois l’application terminée.</span><span class="sxs-lookup"><span data-stu-id="00a89-112">A common pattern is to provision a large number of cores in a burst, and then spin down to zero once the application completes.</span></span>
- <span data-ttu-id="00a89-113">Il n’est pas nécessaire que l’application reste opérationnelle 24 h sur 24, 7 jours sur 7.</span><span class="sxs-lookup"><span data-stu-id="00a89-113">The application does not need to stay up 24/7.</span></span> <span data-ttu-id="00a89-114">Toutefois, le système doit gérer les défaillances de nœud ou les incidents d’application.</span><span class="sxs-lookup"><span data-stu-id="00a89-114">However, the system must handle node failures or application crashes.</span></span>
- <span data-ttu-id="00a89-115">Dans le cas de certaines applications, les tâches sont indépendantes et peuvent s’exécuter en parallèle.</span><span class="sxs-lookup"><span data-stu-id="00a89-115">For some applications, tasks are independent and can run in parallel.</span></span> <span data-ttu-id="00a89-116">Dans d’autres cas, les tâches sont étroitement liées, ce qui signifie qu’elles doivent interagir ou échanger des résultats intermédiaires.</span><span class="sxs-lookup"><span data-stu-id="00a89-116">In other cases, tasks are tightly coupled, meaning they must interact or exchange intermediate results.</span></span> <span data-ttu-id="00a89-117">Dans ce cas, envisagez d’utiliser des technologies de réseau à haut débit comme InfiniBand et l’Accès direct à la mémoire à distance (RDMA).</span><span class="sxs-lookup"><span data-stu-id="00a89-117">In that case, consider using high-speed networking technologies such as InfiniBand and remote direct memory access (RDMA).</span></span>
- <span data-ttu-id="00a89-118">Selon votre charge de travail, vous pouvez utiliser des tailles de machine virtuelle nécessitant beaucoup de ressources système (H16r, H16mr et A9).</span><span class="sxs-lookup"><span data-stu-id="00a89-118">Depending on your workload, you might use compute-intensive VM sizes (H16r, H16mr, and A9).</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="00a89-119">Quand utiliser cette architecture</span><span class="sxs-lookup"><span data-stu-id="00a89-119">When to use this architecture</span></span>

- <span data-ttu-id="00a89-120">Opérations nécessitant beaucoup de ressources système telles que les simulations et les calculs ultrarapides.</span><span class="sxs-lookup"><span data-stu-id="00a89-120">Computationally intensive operations such as simulation and number crunching.</span></span>
- <span data-ttu-id="00a89-121">Simulations utilisant de nombreuses ressources de calcul et devant être réparties entre les unités centrales de quelques dizaines à plusieurs milliers d’ordinateurs.</span><span class="sxs-lookup"><span data-stu-id="00a89-121">Simulations that are computationally intensive and must be split across CPUs in multiple computers (10-1000s).</span></span>
- <span data-ttu-id="00a89-122">Simulations qui nécessitent une quantité de mémoire trop importante pour un seul ordinateur et qui doivent donc être réparties entre plusieurs ordinateurs.</span><span class="sxs-lookup"><span data-stu-id="00a89-122">Simulations that require too much memory for one computer, and must be split across multiple computers.</span></span>
- <span data-ttu-id="00a89-123">Calculs de longue durée qui mettraient trop de temps à s’exécuter sur un seul ordinateur.</span><span class="sxs-lookup"><span data-stu-id="00a89-123">Long-running computations that would take too long to complete on a single computer.</span></span>
- <span data-ttu-id="00a89-124">Calculs plus modestes devant être exécutés plusieurs centaines ou milliers de fois, comme les méthodes de Monte Carlo.</span><span class="sxs-lookup"><span data-stu-id="00a89-124">Smaller computations that must be run 100s or 1000s of times, such as Monte Carlo simulations.</span></span>

## <a name="benefits"></a><span data-ttu-id="00a89-125">Avantages</span><span class="sxs-lookup"><span data-stu-id="00a89-125">Benefits</span></span>

- <span data-ttu-id="00a89-126">Obtention de hautes performances avec un traitement de type « [massivement parallèle][embarrassingly-parallel] ».</span><span class="sxs-lookup"><span data-stu-id="00a89-126">High performance with "[embarrassingly parallel][embarrassingly-parallel]" processing.</span></span>
- <span data-ttu-id="00a89-127">Possibilité d’exploiter des centaines, voire des milliers de cœurs d’ordinateur pour résoudre plus rapidement les problèmes complexes.</span><span class="sxs-lookup"><span data-stu-id="00a89-127">Can harness hundreds or thousands of computer cores to solve large problems faster.</span></span>
- <span data-ttu-id="00a89-128">Accès à un matériel hautes performances spécialisé, avec des réseaux InfiniBand à haut débit dédiés.</span><span class="sxs-lookup"><span data-stu-id="00a89-128">Access to specialized high-performance hardware, with dedicated high-speed InfiniBand networks.</span></span>
- <span data-ttu-id="00a89-129">Possibilité d’approvisionnement et de suppression de machines virtuelles en fonction de la tâche à accomplir.</span><span class="sxs-lookup"><span data-stu-id="00a89-129">You can provision VMs as needed to do work, and then tear them down.</span></span>

## <a name="challenges"></a><span data-ttu-id="00a89-130">Défis</span><span class="sxs-lookup"><span data-stu-id="00a89-130">Challenges</span></span>

- <span data-ttu-id="00a89-131">Gestion de l’infrastructure de machines virtuelles.</span><span class="sxs-lookup"><span data-stu-id="00a89-131">Managing the VM infrastructure.</span></span>
- <span data-ttu-id="00a89-132">Gestion du volume de calculs ultrarapides</span><span class="sxs-lookup"><span data-stu-id="00a89-132">Managing the volume of number crunching</span></span>
- <span data-ttu-id="00a89-133">Approvisionnement de milliers de cœurs au moment opportun.</span><span class="sxs-lookup"><span data-stu-id="00a89-133">Provisioning thousands of cores in a timely manner.</span></span>
- <span data-ttu-id="00a89-134">Dans le cas des tâches étroitement liées, l’ajout de cœurs peut entraîner une baisse de rendement.</span><span class="sxs-lookup"><span data-stu-id="00a89-134">For tightly coupled tasks, adding more cores can have diminishing returns.</span></span> <span data-ttu-id="00a89-135">Vous devrez peut-être effectuer des essais pour déterminer le nombre de cœurs optimal.</span><span class="sxs-lookup"><span data-stu-id="00a89-135">You may need to experiment to find the optimum number of cores.</span></span>

## <a name="big-compute-using-azure-batch"></a><span data-ttu-id="00a89-136">Big Compute à l’aide d’Azure Batch</span><span class="sxs-lookup"><span data-stu-id="00a89-136">Big compute using Azure Batch</span></span>

<span data-ttu-id="00a89-137">[Azure Batch][batch] est un service géré qui permet d’exécuter des applications de calcul haute performance (HPC) à grande échelle.</span><span class="sxs-lookup"><span data-stu-id="00a89-137">[Azure Batch][batch] is a managed service for running large-scale high-performance computing (HPC) applications.</span></span>

<span data-ttu-id="00a89-138">Avec Azure Batch, vous configurez un pool de machines virtuelles, puis vous chargez les applications et les fichiers de données.</span><span class="sxs-lookup"><span data-stu-id="00a89-138">Using Azure Batch, you configure a VM pool, and upload the applications and data files.</span></span> <span data-ttu-id="00a89-139">Le service Batch approvisionne ensuite les machines virtuelles, leur affecte des tâches, puis exécute ces dernières et surveille l’état d’avancement de l’exécution.</span><span class="sxs-lookup"><span data-stu-id="00a89-139">Then the Batch service provisions the VMs, assign tasks to the VMs, runs the tasks, and monitors the progress.</span></span> <span data-ttu-id="00a89-140">Batch peut également augmenter automatiquement la taille des machines virtuelles afin de traiter la charge de travail.</span><span class="sxs-lookup"><span data-stu-id="00a89-140">Batch can automatically scale out the VMs in response to the workload.</span></span> <span data-ttu-id="00a89-141">Enfin, Batch offre une fonctionnalité de planification des travaux.</span><span class="sxs-lookup"><span data-stu-id="00a89-141">Batch also provides job scheduling.</span></span>

![Diagramme de Big Compute avec Azure Batch](./images/big-compute-batch.png)

## <a name="big-compute-running-on-virtual-machines"></a><span data-ttu-id="00a89-143">Exécution de Big Compute sur des machines virtuelles</span><span class="sxs-lookup"><span data-stu-id="00a89-143">Big compute running on Virtual Machines</span></span>

<span data-ttu-id="00a89-144">Vous pouvez utiliser [Microsoft HPC Pack][hpc-pack] pour administrer un cluster de machines virtuelles, puis planifier et surveiller des travaux HPC.</span><span class="sxs-lookup"><span data-stu-id="00a89-144">You can use [Microsoft HPC Pack][hpc-pack] to administer a cluster of VMs, and schedule and monitor HPC jobs.</span></span> <span data-ttu-id="00a89-145">Dans le cadre de cette approche, vous devez approvisionner et gérer les machines virtuelles et l’infrastructure réseau.</span><span class="sxs-lookup"><span data-stu-id="00a89-145">With this approach, you must provision and manage the VMs and network infrastructure.</span></span> <span data-ttu-id="00a89-146">Envisagez d’adopter cette approche si vous devez traiter des charges de travail HPC et que vous souhaitez en déplacer une partie ou la totalité vers Azure.</span><span class="sxs-lookup"><span data-stu-id="00a89-146">Consider this approach if you have existing HPC workloads and want to move some or all it to Azure.</span></span> <span data-ttu-id="00a89-147">Vous pouvez déplacer l’intégralité du cluster HPC vers Azure, ou conserver votre cluster HPC au niveau local, mais utiliser Azure pour sa capacité de débordement.</span><span class="sxs-lookup"><span data-stu-id="00a89-147">You can move the entire HPC cluster to Azure, or keep your HPC cluster on-premises but use Azure for burst capacity.</span></span> <span data-ttu-id="00a89-148">Pour plus d’informations, consultez l’article [Solutions Batch et HPC pour les charges de travail de calcul à grande échelle][batch-hpc-solutions].</span><span class="sxs-lookup"><span data-stu-id="00a89-148">For more information, see [Batch and HPC solutions for large-scale computing workloads][batch-hpc-solutions].</span></span>

### <a name="hpc-pack-deployed-to-azure"></a><span data-ttu-id="00a89-149">HPC Pack déployé sur Azure</span><span class="sxs-lookup"><span data-stu-id="00a89-149">HPC Pack deployed to Azure</span></span>

<span data-ttu-id="00a89-150">Dans ce scénario, le cluster HPC est entièrement créé dans Azure.</span><span class="sxs-lookup"><span data-stu-id="00a89-150">In this scenario, the HPC cluster is created entirely within Azure.</span></span>

![Diagramme de HPC Pack déployé sur Azure](./images/big-compute-iaas.png)

<span data-ttu-id="00a89-152">Le nœud principal fournit les services de gestion et de planification des travaux au cluster.</span><span class="sxs-lookup"><span data-stu-id="00a89-152">The head node provides management and job scheduling services to the cluster.</span></span> <span data-ttu-id="00a89-153">Pour les tâches étroitement liées, utilisez un réseau RDMA garantissant des communications à très haut débit et à faible latence entre les machines virtuelles.</span><span class="sxs-lookup"><span data-stu-id="00a89-153">For tightly coupled tasks, use an RDMA network that provides very high bandwidth, low latency communication between VMs.</span></span> <span data-ttu-id="00a89-154">Pour plus d’informations, consultez l’article [Déployer un cluster HPC Pack 2016 dans Azure][deploy-hpc-azure].</span><span class="sxs-lookup"><span data-stu-id="00a89-154">For more information see [Deploy an HPC Pack 2016 cluster in Azure][deploy-hpc-azure].</span></span>

### <a name="burst-an-hpc-cluster-to-azure"></a><span data-ttu-id="00a89-155">Débordement d’un cluster HPC vers Azure</span><span class="sxs-lookup"><span data-stu-id="00a89-155">Burst an HPC cluster to Azure</span></span>

<span data-ttu-id="00a89-156">Dans ce scénario, une organisation exécute HPC Pack en local et utilise des machines virtuelles Azure pour leur capacité de débordement.</span><span class="sxs-lookup"><span data-stu-id="00a89-156">In this scenario, an organization is running HPC Pack on-premises, and uses Azure VMs for burst capacity.</span></span> <span data-ttu-id="00a89-157">Le nœud principal du cluster est local.</span><span class="sxs-lookup"><span data-stu-id="00a89-157">The cluster head node is on-premises.</span></span> <span data-ttu-id="00a89-158">ExpressRoute ou une passerelle VPN connectent le réseau local au réseau virtuel Azure.</span><span class="sxs-lookup"><span data-stu-id="00a89-158">ExpressRoute or VPN Gateway connects the on-premises network to the Azure VNet.</span></span>

![Diagramme d’un cluster hybride Big Compute](./images/big-compute-hybrid.png)

<!-- links -->

[batch]: /azure/batch/
[batch-hpc-solutions]: /azure/batch/batch-hpc-solutions
[deploy-hpc-azure]: /azure/virtual-machines/windows/hpcpack-2016-cluster
[embarrassingly-parallel]: https://en.wikipedia.org/wiki/Embarrassingly_parallel
[hpc-pack]: https://technet.microsoft.com/library/cc514029
