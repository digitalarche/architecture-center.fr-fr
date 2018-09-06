---
title: Style d’architecture Big Compute
description: Décrit les avantages, les inconvénients et les bonnes pratiques relatifs aux architectures Big Compute sur Azure
author: MikeWasson
ms.date: 08/30/2018
ms.openlocfilehash: aca2221faf1fbf47de2fd81c8909dfe8aef46bea
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/31/2018
ms.locfileid: "43326171"
---
# <a name="big-compute-architecture-style"></a><span data-ttu-id="54f75-103">Style d’architecture Big Compute</span><span class="sxs-lookup"><span data-stu-id="54f75-103">Big compute architecture style</span></span>

<span data-ttu-id="54f75-104">Le terme *Big Compute* décrit les charges de travail à grande échelle qui nécessitent plusieurs centaines ou milliers de cœurs.</span><span class="sxs-lookup"><span data-stu-id="54f75-104">The term *big compute* describes large-scale workloads that require a large number of cores, often numbering in the hundreds or thousands.</span></span> <span data-ttu-id="54f75-105">Les scénarios d’utilisation possibles incluent notamment le rendu d’images, la dynamique des fluides, la modélisation des risques financiers, la prospection pétrolière, la conception de médicaments et l’analyse technique des contraintes.</span><span class="sxs-lookup"><span data-stu-id="54f75-105">Scenarios include image rendering, fluid dynamics, financial risk modeling, oil exploration, drug design, and engineering stress analysis, among others.</span></span>

![](./images/big-compute-logical.png)

<span data-ttu-id="54f75-106">Voici quelques caractéristiques types des applications Big Compute :</span><span class="sxs-lookup"><span data-stu-id="54f75-106">Here are some typical characteristics of big compute applications:</span></span>

- <span data-ttu-id="54f75-107">Le travail peut être fractionné en tâches discrètes qui sont exécutables sur plusieurs cœurs simultanément.</span><span class="sxs-lookup"><span data-stu-id="54f75-107">The work can be split into discrete tasks, which can be run across many cores simultaneously.</span></span>
- <span data-ttu-id="54f75-108">Chaque tâche est un processus fini.</span><span class="sxs-lookup"><span data-stu-id="54f75-108">Each task is finite.</span></span> <span data-ttu-id="54f75-109">Elle accepte une entrée, effectue un traitement spécifique, puis génère la sortie correspondante.</span><span class="sxs-lookup"><span data-stu-id="54f75-109">It takes some input, does some processing, and produces output.</span></span> <span data-ttu-id="54f75-110">L’ensemble de l’application s’exécute pendant une durée limitée (nombre de minutes ou de jours).</span><span class="sxs-lookup"><span data-stu-id="54f75-110">The entire application runs for a finite amount of time (minutes to days).</span></span> <span data-ttu-id="54f75-111">Un modèle courant consiste à approvisionner un grand nombre de cœurs en rafale, puis à revenir à zéro une fois l’application terminée.</span><span class="sxs-lookup"><span data-stu-id="54f75-111">A common pattern is to provision a large number of cores in a burst, and then spin down to zero once the application completes.</span></span> 
- <span data-ttu-id="54f75-112">Il n’est pas nécessaire que l’application reste opérationnelle 24 h sur 24, 7 jours sur 7.</span><span class="sxs-lookup"><span data-stu-id="54f75-112">The application does not need to stay up 24/7.</span></span> <span data-ttu-id="54f75-113">Toutefois, le système doit gérer les défaillances de nœud ou les incidents d’application.</span><span class="sxs-lookup"><span data-stu-id="54f75-113">However, the system must handle node failures or application crashes.</span></span>
- <span data-ttu-id="54f75-114">Dans le cas de certaines applications, les tâches sont indépendantes et peuvent s’exécuter en parallèle.</span><span class="sxs-lookup"><span data-stu-id="54f75-114">For some applications, tasks are independent and can run in parallel.</span></span> <span data-ttu-id="54f75-115">Dans d’autres cas, les tâches sont étroitement liées, ce qui signifie qu’elles doivent interagir ou échanger des résultats intermédiaires.</span><span class="sxs-lookup"><span data-stu-id="54f75-115">In other cases, tasks are tightly coupled, meaning they must interact or exchange intermediate results.</span></span> <span data-ttu-id="54f75-116">Dans ce cas, envisagez d’utiliser des technologies de réseau à haut débit comme InfiniBand et l’Accès direct à la mémoire à distance (RDMA).</span><span class="sxs-lookup"><span data-stu-id="54f75-116">In that case, consider using high-speed networking technologies such as InfiniBand and remote direct memory access (RDMA).</span></span> 
- <span data-ttu-id="54f75-117">Selon votre charge de travail, vous pouvez utiliser des tailles de machine virtuelle nécessitant beaucoup de ressources système (H16r, H16mr et A9).</span><span class="sxs-lookup"><span data-stu-id="54f75-117">Depending on your workload, you might use compute-intensive VM sizes (H16r, H16mr, and A9).</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="54f75-118">Quand utiliser cette architecture</span><span class="sxs-lookup"><span data-stu-id="54f75-118">When to use this architecture</span></span>

- <span data-ttu-id="54f75-119">Opérations nécessitant beaucoup de ressources système telles que les simulations et les calculs ultrarapides.</span><span class="sxs-lookup"><span data-stu-id="54f75-119">Computationally intensive operations such as simulation and number crunching.</span></span>
- <span data-ttu-id="54f75-120">Simulations utilisant de nombreuses ressources de calcul et devant être réparties entre les unités centrales de quelques dizaines à plusieurs milliers d’ordinateurs.</span><span class="sxs-lookup"><span data-stu-id="54f75-120">Simulations that are computationally intensive and must be split across CPUs in multiple computers (10-1000s).</span></span>
- <span data-ttu-id="54f75-121">Simulations qui nécessitent une quantité de mémoire trop importante pour un seul ordinateur et qui doivent donc être réparties entre plusieurs ordinateurs.</span><span class="sxs-lookup"><span data-stu-id="54f75-121">Simulations that require too much memory for one computer, and must be split across multiple computers.</span></span>
- <span data-ttu-id="54f75-122">Calculs de longue durée qui mettraient trop de temps à s’exécuter sur un seul ordinateur.</span><span class="sxs-lookup"><span data-stu-id="54f75-122">Long-running computations that would take too long to complete on a single computer.</span></span>
- <span data-ttu-id="54f75-123">Calculs plus modestes devant être exécutés plusieurs centaines ou milliers de fois, comme les méthodes de Monte Carlo.</span><span class="sxs-lookup"><span data-stu-id="54f75-123">Smaller computations that must be run 100s or 1000s of times, such as Monte Carlo simulations.</span></span>

## <a name="benefits"></a><span data-ttu-id="54f75-124">Avantages</span><span class="sxs-lookup"><span data-stu-id="54f75-124">Benefits</span></span>

- <span data-ttu-id="54f75-125">Obtention de hautes performances avec un traitement de type « [massivement parallèle][embarrassingly-parallel] ».</span><span class="sxs-lookup"><span data-stu-id="54f75-125">High performance with "[embarrassingly parallel][embarrassingly-parallel]" processing.</span></span>
- <span data-ttu-id="54f75-126">Possibilité d’exploiter des centaines, voire des milliers de cœurs d’ordinateur pour résoudre plus rapidement les problèmes complexes.</span><span class="sxs-lookup"><span data-stu-id="54f75-126">Can harness hundreds or thousands of computer cores to solve large problems faster.</span></span>
- <span data-ttu-id="54f75-127">Accès à un matériel hautes performances spécialisé, avec des réseaux InfiniBand à haut débit dédiés.</span><span class="sxs-lookup"><span data-stu-id="54f75-127">Access to specialized high-performance hardware, with dedicated high-speed InfiniBand networks.</span></span>
- <span data-ttu-id="54f75-128">Possibilité d’approvisionnement et de suppression de machines virtuelles en fonction de la tâche à accomplir.</span><span class="sxs-lookup"><span data-stu-id="54f75-128">You can provision VMs as needed to do work, and then tear them down.</span></span> 

## <a name="challenges"></a><span data-ttu-id="54f75-129">Défis</span><span class="sxs-lookup"><span data-stu-id="54f75-129">Challenges</span></span>

- <span data-ttu-id="54f75-130">Gestion de l’infrastructure de machines virtuelles.</span><span class="sxs-lookup"><span data-stu-id="54f75-130">Managing the VM infrastructure.</span></span>
- <span data-ttu-id="54f75-131">Gestion du volume de calculs ultrarapides.</span><span class="sxs-lookup"><span data-stu-id="54f75-131">Managing the volume of number crunching.</span></span> 
- <span data-ttu-id="54f75-132">Approvisionnement de milliers de cœurs au moment opportun.</span><span class="sxs-lookup"><span data-stu-id="54f75-132">Provisioning thousands of cores in a timely manner.</span></span>
- <span data-ttu-id="54f75-133">Dans le cas des tâches étroitement liées, l’ajout de cœurs peut entraîner une baisse de rendement.</span><span class="sxs-lookup"><span data-stu-id="54f75-133">For tightly coupled tasks, adding more cores can have diminishing returns.</span></span> <span data-ttu-id="54f75-134">Vous devrez peut-être effectuer des essais pour déterminer le nombre de cœurs optimal.</span><span class="sxs-lookup"><span data-stu-id="54f75-134">You may need to experiment to find the optimum number of cores.</span></span>

## <a name="big-compute-using-azure-batch"></a><span data-ttu-id="54f75-135">Big Compute à l’aide d’Azure Batch</span><span class="sxs-lookup"><span data-stu-id="54f75-135">Big compute using Azure Batch</span></span>

<span data-ttu-id="54f75-136">[Azure Batch][batch] est un service géré qui permet d’exécuter des applications de calcul haute performance (HPC) à grande échelle.</span><span class="sxs-lookup"><span data-stu-id="54f75-136">[Azure Batch][batch] is a managed service for running large-scale high-performance computing (HPC) applications.</span></span>

<span data-ttu-id="54f75-137">Avec Azure Batch, vous configurez un pool de machines virtuelles, puis vous chargez les applications et les fichiers de données.</span><span class="sxs-lookup"><span data-stu-id="54f75-137">Using Azure Batch, you configure a VM pool, and upload the applications and data files.</span></span> <span data-ttu-id="54f75-138">Le service Batch approvisionne ensuite les machines virtuelles, leur affecte des tâches, puis exécute ces dernières et surveille l’état d’avancement de l’exécution.</span><span class="sxs-lookup"><span data-stu-id="54f75-138">Then the Batch service provisions the VMs, assign tasks to the VMs, runs the tasks, and monitors the progress.</span></span> <span data-ttu-id="54f75-139">Batch peut également augmenter automatiquement la taille des machines virtuelles afin de traiter la charge de travail.</span><span class="sxs-lookup"><span data-stu-id="54f75-139">Batch can automatically scale out the VMs in response to the workload.</span></span> <span data-ttu-id="54f75-140">Enfin, Batch offre une fonctionnalité de planification des travaux.</span><span class="sxs-lookup"><span data-stu-id="54f75-140">Batch also provides job scheduling.</span></span>

![](./images/big-compute-batch.png) 

## <a name="big-compute-running-on-virtual-machines"></a><span data-ttu-id="54f75-141">Exécution de Big Compute sur des machines virtuelles</span><span class="sxs-lookup"><span data-stu-id="54f75-141">Big compute running on Virtual Machines</span></span>

<span data-ttu-id="54f75-142">Vous pouvez utiliser [Microsoft HPC Pack][hpc-pack] pour administrer un cluster de machines virtuelles, puis planifier et surveiller des travaux HPC.</span><span class="sxs-lookup"><span data-stu-id="54f75-142">You can use [Microsoft HPC Pack][hpc-pack] to administer a cluster of VMs, and schedule and monitor HPC jobs.</span></span> <span data-ttu-id="54f75-143">Dans le cadre de cette approche, vous devez approvisionner et gérer les machines virtuelles et l’infrastructure réseau.</span><span class="sxs-lookup"><span data-stu-id="54f75-143">With this approach, you must provision and manage the VMs and network infrastructure.</span></span> <span data-ttu-id="54f75-144">Envisagez d’adopter cette approche si vous devez traiter des charges de travail HPC et que vous souhaitez en déplacer une partie ou la totalité vers Azure.</span><span class="sxs-lookup"><span data-stu-id="54f75-144">Consider this approach if you have existing HPC workloads and want to move some or all it to Azure.</span></span> <span data-ttu-id="54f75-145">Vous pouvez déplacer l’intégralité du cluster HPC vers Azure, ou conserver votre cluster HPC au niveau local, mais utiliser Azure pour sa capacité de débordement.</span><span class="sxs-lookup"><span data-stu-id="54f75-145">You can move the entire HPC cluster to Azure, or keep your HPC cluster on-premises but use Azure for burst capacity.</span></span> <span data-ttu-id="54f75-146">Pour plus d’informations, consultez l’article [Solutions Batch et HPC pour les charges de travail de calcul à grande échelle][batch-hpc-solutions].</span><span class="sxs-lookup"><span data-stu-id="54f75-146">For more information, see [Batch and HPC solutions for large-scale computing workloads][batch-hpc-solutions].</span></span>

### <a name="hpc-pack-deployed-to-azure"></a><span data-ttu-id="54f75-147">HPC Pack déployé sur Azure</span><span class="sxs-lookup"><span data-stu-id="54f75-147">HPC Pack deployed to Azure</span></span>

<span data-ttu-id="54f75-148">Dans ce scénario, le cluster HPC est entièrement créé dans Azure.</span><span class="sxs-lookup"><span data-stu-id="54f75-148">In this scenario, the HPC cluster is created entirely within Azure.</span></span>

![](./images/big-compute-iaas.png) 
 
<span data-ttu-id="54f75-149">Le nœud principal fournit les services de gestion et de planification des travaux au cluster.</span><span class="sxs-lookup"><span data-stu-id="54f75-149">The head node provides management and job scheduling services to the cluster.</span></span> <span data-ttu-id="54f75-150">Pour les tâches étroitement liées, utilisez un réseau RDMA garantissant des communications à très haut débit et à faible latence entre les machines virtuelles.</span><span class="sxs-lookup"><span data-stu-id="54f75-150">For tightly coupled tasks, use an RDMA network that provides very high bandwidth, low latency communication between VMs.</span></span> <span data-ttu-id="54f75-151">Pour plus d’informations, consultez l’article [Déployer un cluster HPC Pack 2016 dans Azure][deploy-hpc-azure].</span><span class="sxs-lookup"><span data-stu-id="54f75-151">For more information see [Deploy an HPC Pack 2016 cluster in Azure][deploy-hpc-azure].</span></span>

### <a name="burst-an-hpc-cluster-to-azure"></a><span data-ttu-id="54f75-152">Débordement d’un cluster HPC vers Azure</span><span class="sxs-lookup"><span data-stu-id="54f75-152">Burst an HPC cluster to Azure</span></span>

<span data-ttu-id="54f75-153">Dans ce scénario, une organisation exécute HPC Pack en local et utilise des machines virtuelles Azure pour leur capacité de débordement.</span><span class="sxs-lookup"><span data-stu-id="54f75-153">In this scenario, an organization is running HPC Pack on-premises, and uses Azure VMs for burst capacity.</span></span> <span data-ttu-id="54f75-154">Le nœud principal du cluster est local.</span><span class="sxs-lookup"><span data-stu-id="54f75-154">The cluster head node is on-premises.</span></span> <span data-ttu-id="54f75-155">ExpressRoute ou une passerelle VPN connectent le réseau local au réseau virtuel Azure.</span><span class="sxs-lookup"><span data-stu-id="54f75-155">ExpressRoute or VPN Gateway connects the on-premises network to the Azure VNet.</span></span>

![](./images/big-compute-hybrid.png) 


[batch]: /azure/batch/
[batch-hpc-solutions]: /azure/batch/batch-hpc-solutions
[deploy-hpc-azure]: /azure/virtual-machines/windows/hpcpack-2016-cluster
[embarrassingly-parallel]: https://en.wikipedia.org/wiki/Embarrassingly_parallel
[hpc-pack]: https://technet.microsoft.com/library/cc514029

 
