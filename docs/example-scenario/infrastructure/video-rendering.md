---
title: Rendu vidéo 3D sur Azure
description: Exécutez des charges de travail HPC natives dans Azure à l’aide du service Azure Batch.
author: adamboeglin
ms.date: 07/13/2018
ms.custom: fasttrack
ms.openlocfilehash: 7dacefd5179c426912dd97af9af7b5a39505392d
ms.sourcegitcommit: a0e8d11543751d681953717f6e78173e597ae207
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2018
ms.locfileid: "53004827"
---
# <a name="3d-video-rendering-on-azure"></a><span data-ttu-id="9a426-103">Rendu vidéo 3D sur Azure</span><span class="sxs-lookup"><span data-stu-id="9a426-103">3D video rendering on Azure</span></span>

<span data-ttu-id="9a426-104">Le rendu vidéo 3D est un processus qui prend beaucoup de temps et nécessitant une quantité importante de temps processeur pour être terminé.</span><span class="sxs-lookup"><span data-stu-id="9a426-104">3D video rendering is a time consuming process that requires a significant amount of CPU time to complete.</span></span> <span data-ttu-id="9a426-105">Sur une seule machine, le processus de génération d’un fichier vidéo à partir de ressources statiques peut prendre plusieurs heures, voire plusieurs jours selon la longueur et la complexité de la vidéo que vous générez.</span><span class="sxs-lookup"><span data-stu-id="9a426-105">On a single machine, the process of generating a video file from static assets can take hours or even days depending on the length and complexity of the video you are producing.</span></span> <span data-ttu-id="9a426-106">De nombreuses entreprises achètent des ordinateurs haut de gamme coûteux pour effectuer ces tâches, ou bien elles investissent dans des groupes de rendu volumineux auxquels ils peuvent envoyer des travaux.</span><span class="sxs-lookup"><span data-stu-id="9a426-106">Many companies will purchase either expensive high end desktop computers to perform these tasks, or invest in large render farms that they can submit jobs to.</span></span> <span data-ttu-id="9a426-107">Toutefois, en tirant parti d’Azure Batch, cette puissance est disponible lorsque vous en avez besoin et s’arrête lorsqu’elle n’est plus nécessaire, tout cela sans avoir à ne consentir aucun investissement.</span><span class="sxs-lookup"><span data-stu-id="9a426-107">However, by taking advantage of Azure Batch, that power is available to you when you need it and shuts itself down when you don't, all without any capital investment.</span></span>

<span data-ttu-id="9a426-108">Batch fournit une expérience cohérente de gestion et de planification des travaux pour les nœuds de calcul Windows Server ou Linux.</span><span class="sxs-lookup"><span data-stu-id="9a426-108">Batch gives you a consistent management experience and job scheduling, whether you select Windows Server or Linux compute nodes.</span></span> <span data-ttu-id="9a426-109">Avec Batch, vous pouvez utiliser vos applications Windows ou Linux existantes, notamment AutoDesk Maya et Blender, pour exécuter des travaux de rendu à grande échelle dans Azure.</span><span class="sxs-lookup"><span data-stu-id="9a426-109">With Batch, you can use your existing Windows or Linux applications, including AutoDesk Maya and Blender, to run large-scale render jobs in Azure.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="9a426-110">Cas d’usage appropriés</span><span class="sxs-lookup"><span data-stu-id="9a426-110">Relevant use cases</span></span>

<span data-ttu-id="9a426-111">Les autres cas d’usage appropriés sont les suivants :</span><span class="sxs-lookup"><span data-stu-id="9a426-111">Other relevant use cases include:</span></span>

* <span data-ttu-id="9a426-112">Modélisation 3D</span><span class="sxs-lookup"><span data-stu-id="9a426-112">3D modeling</span></span>
* <span data-ttu-id="9a426-113">Rendu FX Visual (VFX)</span><span class="sxs-lookup"><span data-stu-id="9a426-113">Visual FX (VFX) rendering</span></span>
* <span data-ttu-id="9a426-114">Transcodage vidéo</span><span class="sxs-lookup"><span data-stu-id="9a426-114">Video transcoding</span></span>
* <span data-ttu-id="9a426-115">Traitement d’image, correction des couleurs et redimensionnement</span><span class="sxs-lookup"><span data-stu-id="9a426-115">Image processing, color correction, and resizing</span></span>

## <a name="architecture"></a><span data-ttu-id="9a426-116">Architecture</span><span class="sxs-lookup"><span data-stu-id="9a426-116">Architecture</span></span>

![Présentation de l’architecture des composants impliqués dans une solution HPC native Cloud à l’aide d’Azure Batch][architecture]

<span data-ttu-id="9a426-118">Ce scénario présente un flux de travail qui utilise Azure Batch.</span><span class="sxs-lookup"><span data-stu-id="9a426-118">This scenario shows a workflow that uses Azure Batch.</span></span> <span data-ttu-id="9a426-119">Les données circulent comme suit :</span><span class="sxs-lookup"><span data-stu-id="9a426-119">The data flows as follows:</span></span>

1. <span data-ttu-id="9a426-120">Charger les fichiers d’entrée et les applications nécessaires pour le traitement des fichiers dans votre compte Stockage Azure.</span><span class="sxs-lookup"><span data-stu-id="9a426-120">Upload input files and the applications to process those files to your Azure Storage account.</span></span>
2. <span data-ttu-id="9a426-121">Créer un pool Batch de nœuds de calcul dans votre compte Batch, un travail pour exécuter la charge de travail sur le pool et des tâches dans le travail.</span><span class="sxs-lookup"><span data-stu-id="9a426-121">Create a Batch pool of compute nodes in your Batch account, a job to run the workload on the pool, and tasks in the job.</span></span>
3. <span data-ttu-id="9a426-122">Télécharger des fichiers d’entrée et des applications vers Azure Batch.</span><span class="sxs-lookup"><span data-stu-id="9a426-122">Download input files and the applications to Batch.</span></span>
4. <span data-ttu-id="9a426-123">Surveiller l’exécution d’une tâche.</span><span class="sxs-lookup"><span data-stu-id="9a426-123">Monitor task execution.</span></span>
5. <span data-ttu-id="9a426-124">Charger les sorties des tâches.</span><span class="sxs-lookup"><span data-stu-id="9a426-124">Upload task output.</span></span>
6. <span data-ttu-id="9a426-125">Télécharger les fichiers de sortie.</span><span class="sxs-lookup"><span data-stu-id="9a426-125">Download output files.</span></span>

<span data-ttu-id="9a426-126">Pour simplifier ce processus, vous pouvez également utiliser les [plug-ins de Batch pour Maya et 3ds Max][batch-plugins]</span><span class="sxs-lookup"><span data-stu-id="9a426-126">To simplify this process, you could also use the [Batch Plugins for Maya and 3ds Max][batch-plugins]</span></span>

### <a name="components"></a><span data-ttu-id="9a426-127">Composants</span><span class="sxs-lookup"><span data-stu-id="9a426-127">Components</span></span>

<span data-ttu-id="9a426-128">Azure Batch s’appuie sur les technologies Azure suivantes :</span><span class="sxs-lookup"><span data-stu-id="9a426-128">Azure Batch builds upon the following Azure technologies:</span></span>

* <span data-ttu-id="9a426-129">Les [réseaux virtuels](/azure/virtual-network/virtual-networks-overview) sont utilisés pour le nœud principal et les ressources de calcul.</span><span class="sxs-lookup"><span data-stu-id="9a426-129">[Virtual Networks](/azure/virtual-network/virtual-networks-overview) are used for both the head node and the compute resources.</span></span>
* <span data-ttu-id="9a426-130">Les [comptes de stockage Azure](/azure/storage/common/storage-introduction) sont utilisés pour la synchronisation et la conservation des données.</span><span class="sxs-lookup"><span data-stu-id="9a426-130">[Azure Storage accounts](/azure/storage/common/storage-introduction) are used for synchronization and data retention.</span></span>
* <span data-ttu-id="9a426-131">Les [Microsoft Azure Virtual Machine Scale Sets][vmss] sont utilisés par CycleCloud pour les ressources de calcul.</span><span class="sxs-lookup"><span data-stu-id="9a426-131">[Virtual Machine Scale Sets][vmss] are used by CycleCloud for compute resources.</span></span>

## <a name="considerations"></a><span data-ttu-id="9a426-132">Considérations</span><span class="sxs-lookup"><span data-stu-id="9a426-132">Considerations</span></span>

### <a name="machine-sizes-available-for-azure-batch"></a><span data-ttu-id="9a426-133">Tailles de machine disponibles pour Azure Batch</span><span class="sxs-lookup"><span data-stu-id="9a426-133">Machine Sizes available for Azure Batch</span></span>

<span data-ttu-id="9a426-134">Bien que la plupart des clients optent pour des ressources qui possèdent une grande puissance de processeur, d’autres charges de travail utilisant des groupes de machines virtuelles identiques peuvent impliquer des critères de choix des machines virtuelles différents, selon plusieurs facteurs :</span><span class="sxs-lookup"><span data-stu-id="9a426-134">While most rendering customers will choose resources with high CPU power, other workloads using virtual machine scale sets may choose VMs differently and will depend on a number of factors:</span></span>

* <span data-ttu-id="9a426-135">L’application qui s’exécute est-elle memory-bound ?</span><span class="sxs-lookup"><span data-stu-id="9a426-135">Is the application being run memory bound?</span></span>
* <span data-ttu-id="9a426-136">A-t-elle besoin d’utiliser des GPU ?</span><span class="sxs-lookup"><span data-stu-id="9a426-136">Does the application need to use GPUs?</span></span> 
* <span data-ttu-id="9a426-137">Les types de travaux sont-ils extrêmement parallèles ? Réclament-ils une connectivité InfiniBand pour les travaux étroitement couplés ?</span><span class="sxs-lookup"><span data-stu-id="9a426-137">Are the job types embarrassingly parallel or require infiniband connectivity for tightly coupled jobs?</span></span>
* <span data-ttu-id="9a426-138">Nécessitent des E/S rapides pour accéder au stockage sur les nœuds de calcul.</span><span class="sxs-lookup"><span data-stu-id="9a426-138">Require fast I/O to access storage on the compute Nodes.</span></span>

<span data-ttu-id="9a426-139">Azure propose un large éventail de tailles de machine virtuelle qui répondent à chacune des exigences des applications ci-dessus ; certaines sont spécifiquement adaptées au calcul haute performance (HPC), mais même les plus petites tailles peuvent être utilisées pour fournir une implémentation de grille efficace :</span><span class="sxs-lookup"><span data-stu-id="9a426-139">Azure has a wide range of VM sizes that can address each and every one of the above application requirements, some are specific to HPC, but even the smallest sizes can be utilized to provide an effective grid implementation:</span></span>

* <span data-ttu-id="9a426-140">[Tailles de machine virtuelle HPC][compute-hpc] Le rendu étant CPU-bound, Microsoft suggère généralement les machines virtuelles Azure de série H.</span><span class="sxs-lookup"><span data-stu-id="9a426-140">[HPC VM sizes][compute-hpc] Due to the CPU bound nature of rendering, Microsoft typically suggests the Azure H-Series VMs.</span></span> <span data-ttu-id="9a426-141">Les machines virtuelles de ce type, spécialement conçues pour les besoins élevés de calcul, proposent des processeurs virtuels de 8 ou 16 cœurs, une mémoire DDR4, du stockage temporaire SSD et la technologie Intel Haswell E5.</span><span class="sxs-lookup"><span data-stu-id="9a426-141">This type of VM is built specifically for high end computational needs, they have 8 and 16 core vCPU sizes available, and features DDR4 memory, SSD temporary storage, and Haswell E5 Intel technology.</span></span>
* <span data-ttu-id="9a426-142">[Tailles de machine virtuelle de GPU][compute-gpu] Les tailles de machine virtuelle au GPU optimisé sont des machines virtuelles spécialisées disponibles avec des GPU NVIDIA uniques ou multiples.</span><span class="sxs-lookup"><span data-stu-id="9a426-142">[GPU VM sizes][compute-gpu] GPU optimized VM sizes are specialized virtual machines available with single or multiple NVIDIA GPUs.</span></span> <span data-ttu-id="9a426-143">Ces tailles sont conçues pour des charges de travail de visualisation, mais également de calcul et d’affichage graphique intensifs.</span><span class="sxs-lookup"><span data-stu-id="9a426-143">These sizes are designed for compute-intensive, graphics-intensive, and visualization workloads.</span></span>
* <span data-ttu-id="9a426-144">Les tailles des cœurs NC, NCv2, NCv3 et ND sont optimisées pour les applications nécessitant des ressources réseau et de calcul importantes, les algorithmes, notamment les applications et simulations CUDA et OpenCL, l’intelligence artificielle et le Deep Learning.</span><span class="sxs-lookup"><span data-stu-id="9a426-144">NC, NCv2, NCv3, and ND sizes are optimized for compute-intensive and network-intensive applications and algorithms, including CUDA and OpenCL-based applications and simulations, AI, and Deep Learning.</span></span> <span data-ttu-id="9a426-145">Les tailles des cœurs NV sont optimisées et conçues pour la visualisation à distance, la diffusion en continu, les jeux, le codage et les scénarios VDI utilisant des infrastructures comme OpenGL et DirectX.</span><span class="sxs-lookup"><span data-stu-id="9a426-145">NV sizes are optimized and designed for remote visualization, streaming, gaming, encoding, and VDI scenarios utilizing frameworks such as OpenGL and DirectX.</span></span>
* <span data-ttu-id="9a426-146">[Tailles de machine virtuelle à mémoire optimisée][compute-memory] Lorsque le besoin de mémoire augmente, les tailles de machines virtuelles à mémoire optimisée offrent un rapport mémoire/processeur supérieur.</span><span class="sxs-lookup"><span data-stu-id="9a426-146">[Memory optimized VM sizes][compute-memory] When more memory is required, the memory optimized VM sizes offer a higher memory-to-CPU ratio.</span></span>
* <span data-ttu-id="9a426-147">[Tailles de machine virtuelle à usage général][compute-general] Des tailles de machines virtuelles à usage général sont également disponibles ; elles offrent un ratio mémoire/processeur équilibré.</span><span class="sxs-lookup"><span data-stu-id="9a426-147">[General purposes VM sizes][compute-general] General-purpose VM sizes are also available and provide balanced CPU-to-memory ratio.</span></span>

### <a name="alternatives"></a><span data-ttu-id="9a426-148">Autres solutions</span><span class="sxs-lookup"><span data-stu-id="9a426-148">Alternatives</span></span>

<span data-ttu-id="9a426-149">Si vous souhaitez contrôler plus étroitement votre environnement de rendu dans Azure ou si vous avez besoin d’une implémentation hybride, le calcul CycleCloud peut aider à orchestrer une grille IaaS dans le cloud.</span><span class="sxs-lookup"><span data-stu-id="9a426-149">If you require more control over your rendering environment in Azure or need a hybrid implementation, then CycleCloud computing can help orchestrate an IaaS grid in the cloud.</span></span> <span data-ttu-id="9a426-150">En utilisant les mêmes technologies Azure sous-jacentes comme Azure Batch, la création et la maintenance d’une grille IaaS est un processus efficace.</span><span class="sxs-lookup"><span data-stu-id="9a426-150">Using the same underlying Azure technologies as Azure Batch, it makes building and maintaining an IaaS grid an efficient process.</span></span> <span data-ttu-id="9a426-151">Pour plus d’informations, notamment sur les principes de conception, suivez le lien ci-dessous :</span><span class="sxs-lookup"><span data-stu-id="9a426-151">To find out more and learn about the design principles use the following link:</span></span>

<span data-ttu-id="9a426-152">Pour une vue d’ensemble complète de toutes les solutions HPC disponibles dans Azure, voir l’article [Solutions HPC, Batch et Big Compute avec les machines virtuelles Azure][hpc-alt-solutions].</span><span class="sxs-lookup"><span data-stu-id="9a426-152">For a complete overview of all the HPC solutions that are available to you in Azure, see the article [HPC, Batch, and Big Compute solutions using Azure VMs][hpc-alt-solutions]</span></span>

### <a name="availability"></a><span data-ttu-id="9a426-153">Disponibilité</span><span class="sxs-lookup"><span data-stu-id="9a426-153">Availability</span></span>

<span data-ttu-id="9a426-154">Le monitoring des composants Azure Batch est possible avec différents services, outils et API.</span><span class="sxs-lookup"><span data-stu-id="9a426-154">Monitoring of the Azure Batch components is available through a range of services, tools, and APIs.</span></span> <span data-ttu-id="9a426-155">Pour plus d’informations, voir l’article [Effectuer le monitoring des solutions Batch][batch-monitor].</span><span class="sxs-lookup"><span data-stu-id="9a426-155">Monitoring is discussed further in the [Monitor Batch solutions][batch-monitor] article.</span></span>

### <a name="scalability"></a><span data-ttu-id="9a426-156">Extensibilité</span><span class="sxs-lookup"><span data-stu-id="9a426-156">Scalability</span></span>

<span data-ttu-id="9a426-157">Les pools qui se trouvent dans un compte Azure Batch peuvent être mis à l’échelle par intervention manuelle ou bien automatiquement avec une formule qui s’appuie sur les mesures d’Azure Batch.</span><span class="sxs-lookup"><span data-stu-id="9a426-157">Pools within an Azure Batch account can either scale through manual intervention or, by using a formula based on Azure Batch metrics, be scaled automatically.</span></span> <span data-ttu-id="9a426-158">Pour plus d’informations sur l’évolutivité, voir l’article [Créer une formule de mise à l’échelle automatique pour les nœuds d’un pool Batch][batch-scaling].</span><span class="sxs-lookup"><span data-stu-id="9a426-158">For more information on scalability, see the article [Create an automatic scaling formula for scaling nodes in a Batch pool][batch-scaling].</span></span>

### <a name="security"></a><span data-ttu-id="9a426-159">Sécurité</span><span class="sxs-lookup"><span data-stu-id="9a426-159">Security</span></span>

<span data-ttu-id="9a426-160">Pour obtenir des conseils d’ordre général sur la conception de solutions sécurisées, consultez la [documentation sur la sécurité Azure][security].</span><span class="sxs-lookup"><span data-stu-id="9a426-160">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="9a426-161">Résilience</span><span class="sxs-lookup"><span data-stu-id="9a426-161">Resiliency</span></span>

<span data-ttu-id="9a426-162">Il n’existe pour le moment aucune fonctionnalité de basculement dans Azure Batch. Nous vous recommandons de suivre les étapes ci-dessous pour garantir la disponibilité en cas de panne non planifiée :</span><span class="sxs-lookup"><span data-stu-id="9a426-162">While there is currently no failover capability in Azure Batch, we recommend using the following steps to ensure availability if there is an unplanned outage:</span></span>

* <span data-ttu-id="9a426-163">Créer un compte Azure Batch dans un autre emplacement Azure avec un autre compte de stockage</span><span class="sxs-lookup"><span data-stu-id="9a426-163">Create an Azure Batch account in an alternate Azure location with an alternate Storage Account</span></span>
* <span data-ttu-id="9a426-164">Créer les mêmes pools de nœud avec le même nom et aucun nœud alloué</span><span class="sxs-lookup"><span data-stu-id="9a426-164">Create the same node pools with the same name, with zero nodes allocated</span></span>
* <span data-ttu-id="9a426-165">S’assurer que les applications sont créées et mises à jour vers le compte de stockage secondaire</span><span class="sxs-lookup"><span data-stu-id="9a426-165">Ensure Applications are created and updated to the alternate Storage Account</span></span>
* <span data-ttu-id="9a426-166">Charger des fichiers d’entrée et envoyer des travaux vers l’autre compte Azure Batch</span><span class="sxs-lookup"><span data-stu-id="9a426-166">Upload input files and submit jobs to the alternate Azure Batch account</span></span>

## <a name="deploy-this-scenario"></a><span data-ttu-id="9a426-167">Déployer ce scénario</span><span class="sxs-lookup"><span data-stu-id="9a426-167">Deploy this scenario</span></span>

### <a name="creating-an-azure-batch-account-and-pools-manually"></a><span data-ttu-id="9a426-168">Création manuelle d’un compte Azure Batch et des pools</span><span class="sxs-lookup"><span data-stu-id="9a426-168">Creating an Azure Batch account and pools manually</span></span>

<span data-ttu-id="9a426-169">Ce scénario illustre le fonctionnement d’Azure Batch tout en présentant Azure Batch Labs comme un exemple de solution SaaS que vous pouvez développer pour vos propres clients :</span><span class="sxs-lookup"><span data-stu-id="9a426-169">This scenario demonstrates how Azure Batch works while showcasing Azure Batch Labs as an example SaaS solution that can be developed for your own customers:</span></span>

<span data-ttu-id="9a426-170">[Azure Batch Masterclass][batch-labs-masterclass]</span><span class="sxs-lookup"><span data-stu-id="9a426-170">[Azure Batch Masterclass][batch-labs-masterclass]</span></span>

### <a name="deploying-the-example-scenario-using-an-azure-resource-manager-template"></a><span data-ttu-id="9a426-171">Déploiement de l’exemple de scénario suivant un modèle Azure Resource Manager</span><span class="sxs-lookup"><span data-stu-id="9a426-171">Deploying the example scenario using an Azure Resource Manager template</span></span>

<span data-ttu-id="9a426-172">Le modèle déploiera :</span><span class="sxs-lookup"><span data-stu-id="9a426-172">The template will deploy:</span></span>

* <span data-ttu-id="9a426-173">Un nouveau compte Azure Batch</span><span class="sxs-lookup"><span data-stu-id="9a426-173">A new Azure Batch account</span></span>
* <span data-ttu-id="9a426-174">Un compte de stockage</span><span class="sxs-lookup"><span data-stu-id="9a426-174">A storage account</span></span>
* <span data-ttu-id="9a426-175">Un pool de nœuds associé au compte Batch</span><span class="sxs-lookup"><span data-stu-id="9a426-175">A node pool associated with the batch account</span></span>
* <span data-ttu-id="9a426-176">Le pool de nœuds sera configuré pour utiliser des machines virtuelles A2 v2 avec les images de Ubuntu Canonical</span><span class="sxs-lookup"><span data-stu-id="9a426-176">The node pool will be configured to use A2 v2 VMs with Canonical Ubuntu images</span></span>
* <span data-ttu-id="9a426-177">Le pool de nœuds ne contient initialement aucune machine virtuelle ; une mise à l’échelle manuelle sera nécessaire pour en ajouter</span><span class="sxs-lookup"><span data-stu-id="9a426-177">The node pool will contain zero VMs initially and will require you to manually scale to add VMs</span></span>

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fhpc%2Fbatchcreatewithpools.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

<span data-ttu-id="9a426-178">[Plus d’informations sur les modèles Resource Manager][azure-arm-templates]</span><span class="sxs-lookup"><span data-stu-id="9a426-178">[Learn more about Resource Manager templates][azure-arm-templates]</span></span>

## <a name="pricing"></a><span data-ttu-id="9a426-179">Tarifs</span><span class="sxs-lookup"><span data-stu-id="9a426-179">Pricing</span></span>

<span data-ttu-id="9a426-180">Le coût d’utilisation d’Azure Batch varie selon les tailles de machine virtuelle utilisées pour les pools et la durée d’allocation et d’exécution de ces machines virtuelles ; la création d’un compte Azure Batch est gratuite.</span><span class="sxs-lookup"><span data-stu-id="9a426-180">The cost of using Azure Batch will depend on the VM sizes that are used for the pools and how long these VMs are allocated and running, there is no cost associated with an Azure Batch account creation.</span></span> <span data-ttu-id="9a426-181">Le stockage et la sortie des données doivent être pris en compte, car ils occasionnent des frais supplémentaires.</span><span class="sxs-lookup"><span data-stu-id="9a426-181">Storage and data egress should be taken into account as these will apply additional costs.</span></span>

<span data-ttu-id="9a426-182">Voici quelques exemples des coûts qui pourraient être engagés pour une tâche se terminant en 8 heures à l’aide de différents nombres de serveurs :</span><span class="sxs-lookup"><span data-stu-id="9a426-182">The following are examples of costs that could be incurred for a job that completes in 8 hours using a different number of servers:</span></span>

* <span data-ttu-id="9a426-183">100 machines virtuelles avec processeur hautes performances : [Estimation du coût][hpc-est-high]</span><span class="sxs-lookup"><span data-stu-id="9a426-183">100 High-Performance CPU VMs: [Cost Estimate][hpc-est-high]</span></span>

  <span data-ttu-id="9a426-184">100 H16m (16 cœurs, 225 Go de RAM, 512 Go de Stockage Premium), 2 To de Stockage Blob, 1 To de sortie</span><span class="sxs-lookup"><span data-stu-id="9a426-184">100 x H16m (16 cores, 225 GB RAM, Premium Storage 512 GB), 2 TB Blob Storage, 1-TB egress</span></span>

* <span data-ttu-id="9a426-185">50 machines virtuelles avec processeur hautes performances : [Estimation du coût][hpc-est-med]</span><span class="sxs-lookup"><span data-stu-id="9a426-185">50 High-Performance CPU VMs: [Cost Estimate][hpc-est-med]</span></span>

  <span data-ttu-id="9a426-186">50 H16m (16 cœurs, 225 Go de RAM, 512 Go de Stockage Premium), 2 To de Stockage Blob, 1 To de sortie</span><span class="sxs-lookup"><span data-stu-id="9a426-186">50 x H16m (16 cores, 225 GB RAM, Premium Storage 512 GB), 2 TB Blob Storage, 1-TB egress</span></span>

* <span data-ttu-id="9a426-187">10 machines virtuelles avec processeur hautes performances : [Estimation du coût][hpc-est-low]</span><span class="sxs-lookup"><span data-stu-id="9a426-187">10 High-Performance CPU VMs: [Cost Estimate][hpc-est-low]</span></span>

  <span data-ttu-id="9a426-188">10 H16m (16 cœurs, 225 Go de RAM, 512 Go de Stockage Premium), 2 To de Stockage Blob, 1 To de sortie</span><span class="sxs-lookup"><span data-stu-id="9a426-188">10 x H16m (16 cores, 225 GB RAM, Premium Storage 512 GB), 2 TB Blob Storage, 1-TB egress</span></span>

### <a name="pricing-for-low-priority-vms"></a><span data-ttu-id="9a426-189">Tarification des machines virtuelles de faible priorité</span><span class="sxs-lookup"><span data-stu-id="9a426-189">Pricing for low-priority VMs</span></span>

<span data-ttu-id="9a426-190">Azure Batch prend également en charge les machines virtuelles basse priorité dans les pools de nœuds, qui peuvent représenter des économies substantielles.</span><span class="sxs-lookup"><span data-stu-id="9a426-190">Azure Batch also supports the use of low-priority VMs in the node pools, which can potentially provide a substantial cost saving.</span></span> <span data-ttu-id="9a426-191">Pour plus d’informations, notamment sur la comparaison des prix entre les machines virtuelles standard et les machines virtuelles de faible priorité, consultez [Azure Batch Pricing][batch-pricing].</span><span class="sxs-lookup"><span data-stu-id="9a426-191">For more information, including a price comparison between standard VMs and low-priority VMs, see [Azure Batch Pricing][batch-pricing].</span></span>

> [!NOTE] 
> <span data-ttu-id="9a426-192">Les machines virtuelles de faible priorité ne conviennent qu’à certaines applications et charges de travail.</span><span class="sxs-lookup"><span data-stu-id="9a426-192">Low-priority VMs are only suitable for certain applications and workloads.</span></span>

## <a name="related-resources"></a><span data-ttu-id="9a426-193">Ressources associées</span><span class="sxs-lookup"><span data-stu-id="9a426-193">Related resources</span></span>

<span data-ttu-id="9a426-194">[Vue d'ensemble de Azure Batch][batch-overview]</span><span class="sxs-lookup"><span data-stu-id="9a426-194">[Azure Batch Overview][batch-overview]</span></span>

<span data-ttu-id="9a426-195">[Documentation de Azure Batch][batch-doc]</span><span class="sxs-lookup"><span data-stu-id="9a426-195">[Azure Batch Documentation][batch-doc]</span></span>

<span data-ttu-id="9a426-196">[Utilisation des conteneurs Docker sur Azure Batch][batch-containers]</span><span class="sxs-lookup"><span data-stu-id="9a426-196">[Using containers on Azure Batch][batch-containers]</span></span>

<!-- links -->
[architecture]: ./media/architecture-video-rendering.png
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[resiliency]: /azure/architecture/resiliency/
[scalability]: /azure/architecture/checklist/scalability
[vmss]: /azure/virtual-machine-scale-sets/overview
[storage]: https://azure.microsoft.com/services/storage/
[batch]: https://azure.microsoft.com/services/batch/
[batch-arch]: https://azure.microsoft.com/solutions/architecture/big-compute-with-azure-batch/
[compute-hpc]: /azure/virtual-machines/windows/sizes-hpc
[compute-gpu]: /azure/virtual-machines/windows/sizes-gpu
[compute-compute]: /azure/virtual-machines/windows/sizes-compute
[compute-memory]: /azure/virtual-machines/windows/sizes-memory
[compute-general]: /azure/virtual-machines/windows/sizes-general
[compute-storage]: /azure/virtual-machines/windows/sizes-storage
[compute-acu]: /azure/virtual-machines/windows/acu
[compute=benchmark]: /azure/virtual-machines/windows/compute-benchmark-scores
[hpc-est-high]: https://azure.com/e/9ac25baf44ef49c3a6b156935ee9544c
[hpc-est-med]: https://azure.com/e/0286f1d6f6784310af4dcda5aec8c893
[hpc-est-low]: https://azure.com/e/e39afab4e71949f9bbabed99b428ba4a
[batch-labs-masterclass]: https://github.com/azurebigcompute/BigComputeLabs/tree/master/Azure%20Batch%20Masterclass%20Labs
[batch-scaling]: /azure/batch/batch-automatic-scaling
[hpc-alt-solutions]: /azure/virtual-machines/linux/high-performance-computing?toc=%2fazure%2fbatch%2ftoc.json
[batch-monitor]: /azure/batch/monitoring-overview
[batch-pricing]: https://azure.microsoft.com/pricing/details/batch/
[batch-doc]: /azure/batch/
[batch-overview]: https://azure.microsoft.com/services/batch/
[batch-containers]: https://github.com/Azure/batch-shipyard
[azure-arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[batch-plugins]: /azure/batch/batch-rendering-service#options-for-submitting-a-render-job