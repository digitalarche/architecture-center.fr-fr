---
title: Rendu vidéo 3D sur Azure
description: Exécution des charges de travail HPC natives dans Azure à l’aide du service Azure Batch
author: adamboeglin
ms.date: 07/13/2018
ms.openlocfilehash: b3af0641642d7ec4b022e8c96f51693eeb0adee4
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/14/2018
ms.locfileid: "39061000"
---
# <a name="3d-video-rendering-on-azure"></a><span data-ttu-id="492b7-103">Rendu vidéo 3D sur Azure</span><span class="sxs-lookup"><span data-stu-id="492b7-103">3D video rendering on Azure</span></span>

<span data-ttu-id="492b7-104">Le rendu 3D est un processus qui prend beaucoup de temps et nécessitant une quantité importante de temps processeur pour être terminé.</span><span class="sxs-lookup"><span data-stu-id="492b7-104">3D rendering is a time consuming process that requires a significant amount of CPU time co complete.</span></span>  <span data-ttu-id="492b7-105">Sur une seule machine, le processus de génération d’un fichier vidéo à partir de ressources statiques peut prendre plusieurs heures, voire plusieurs jours selon la longueur et la complexité de la vidéo que vous générez.</span><span class="sxs-lookup"><span data-stu-id="492b7-105">On a single machine, the process of generating a video file from static assets can take hours or even days depending on the length and complexity of the video you are producing.</span></span>  <span data-ttu-id="492b7-106">De nombreuses entreprises achètent des ordinateurs haut de gamme coûteux pour effectuer ces tâches, ou bien elles investissent dans des groupes de rendu volumineux auxquels ils peuvent envoyer des travaux.</span><span class="sxs-lookup"><span data-stu-id="492b7-106">Many companies will purchase either expensive high end desktop computers to perform these tasks, or invest in large render farms that they can submit jobs to.</span></span>  <span data-ttu-id="492b7-107">Toutefois, en tirant parti d’Azure Batch, cette puissance est disponible lorsque vous en avez besoin et s’arrête lorsqu’elle n’est plus nécessaire, tout cela sans avoir à ne consentir aucun investissement.</span><span class="sxs-lookup"><span data-stu-id="492b7-107">However, by taking advantage of Azure Batch, that power is available to you when you need it and shuts itself down when you don't, all without any capital investment.</span></span>

<span data-ttu-id="492b7-108">Batch fournit une expérience cohérente de gestion et de planification des travaux pour les nœuds de calcul Windows Server ou Linux.</span><span class="sxs-lookup"><span data-stu-id="492b7-108">Batch gives you a consistent management experience and job scheduling, whether you select Windows Server or Linux compute nodes.</span></span> <span data-ttu-id="492b7-109">Avec Batch, vous pouvez utiliser vos applications Windows ou Linux existantes, notamment AutoDesk Maya et Blender, pour exécuter des travaux de rendu à grande échelle dans Azure.</span><span class="sxs-lookup"><span data-stu-id="492b7-109">With Batch, you can use your existing Windows or Linux applications, including AutoDesk Maya and Blender, to run large-scale render jobs in Azure.</span></span>

## <a name="related-use-cases"></a><span data-ttu-id="492b7-110">Cas d’usage connexes</span><span class="sxs-lookup"><span data-stu-id="492b7-110">Related use cases</span></span>

<span data-ttu-id="492b7-111">Pensez à ce scénario pour ces cas d’usage similaires :</span><span class="sxs-lookup"><span data-stu-id="492b7-111">Consider this scenario for these similar use cases:</span></span>

* <span data-ttu-id="492b7-112">Modélisation 3D</span><span class="sxs-lookup"><span data-stu-id="492b7-112">3D Modeling</span></span>
* <span data-ttu-id="492b7-113">Rendu FX Visual (VFX)</span><span class="sxs-lookup"><span data-stu-id="492b7-113">Visual FX (VFX) Rendering</span></span>
* <span data-ttu-id="492b7-114">Transcodage vidéo</span><span class="sxs-lookup"><span data-stu-id="492b7-114">Video transcoding</span></span>
* <span data-ttu-id="492b7-115">Traitement d’image, correction des couleurs et redimensionnement</span><span class="sxs-lookup"><span data-stu-id="492b7-115">Image processing, color correction, & resizing</span></span>

## <a name="architecture"></a><span data-ttu-id="492b7-116">Architecture</span><span class="sxs-lookup"><span data-stu-id="492b7-116">Architecture</span></span>

![Présentation de l’architecture des composants impliqués dans une solution HPC native Cloud à l’aide d’Azure Batch][architecture]

<span data-ttu-id="492b7-118">Cet exemple de scénario décrit le flux de travail lors de l’utilisation d’Azure Batch, les données circulent comme suit :</span><span class="sxs-lookup"><span data-stu-id="492b7-118">This sample scenario covers the workflow when using Azure Batch, the data flows as follows:</span></span>

1. <span data-ttu-id="492b7-119">Charger les fichiers d’entrée et les applications nécessaires pour le traitement des fichiers dans votre compte Stockage Azure</span><span class="sxs-lookup"><span data-stu-id="492b7-119">Upload input files and the applications to process those files to your Azure Storage account</span></span>
2. <span data-ttu-id="492b7-120">Créer un pool Batch de nœuds de calcul dans votre compte Batch, un travail pour exécuter la charge de travail sur le pool et des tâches dans le travail.</span><span class="sxs-lookup"><span data-stu-id="492b7-120">Create a Batch pool of compute nodes in your Batch account, a job to run the workload on the pool, and tasks in the job.</span></span>
3. <span data-ttu-id="492b7-121">Télécharger des fichiers d’entrée et des applications vers Azure Batch</span><span class="sxs-lookup"><span data-stu-id="492b7-121">Download input files and the applications to Batch</span></span>
4. <span data-ttu-id="492b7-122">Surveiller l’exécution d’une tâche</span><span class="sxs-lookup"><span data-stu-id="492b7-122">Monitor task execution</span></span>
5. <span data-ttu-id="492b7-123">Charger les sorties des tâches</span><span class="sxs-lookup"><span data-stu-id="492b7-123">Upload task output</span></span>
6. <span data-ttu-id="492b7-124">Télécharger les fichiers de sortie</span><span class="sxs-lookup"><span data-stu-id="492b7-124">Download output files</span></span>

<span data-ttu-id="492b7-125">Pour simplifier ce processus, vous pouvez également utiliser les [plug-ins de Batch pour Maya et 3ds Max][batch-plugins]</span><span class="sxs-lookup"><span data-stu-id="492b7-125">To simplify this process, you could also use the [Batch Plugins for Maya & 3ds Max][batch-plugins]</span></span>

### <a name="components"></a><span data-ttu-id="492b7-126">Composants</span><span class="sxs-lookup"><span data-stu-id="492b7-126">Components</span></span>

<span data-ttu-id="492b7-127">Azure Batch s’appuie sur les technologies Azure suivantes :</span><span class="sxs-lookup"><span data-stu-id="492b7-127">Azure Batch builds upon the following Azure technologies:</span></span>

* <span data-ttu-id="492b7-128">Les [groupes de ressources][resource-groups] sont des conteneurs logiques pour des ressources Azure.</span><span class="sxs-lookup"><span data-stu-id="492b7-128">[Resource Groups][resource-groups] is a logical container for Azure resources.</span></span>
* <span data-ttu-id="492b7-129">Les [réseaux virtuels][vnet] sont utilisés pour le nœud principal et les ressources de calcul</span><span class="sxs-lookup"><span data-stu-id="492b7-129">[Virtual Networks][vnet] are used to for both the Head Node and Compute resources</span></span>
* <span data-ttu-id="492b7-130">Les comptes de [stockage][storage] sont utilisés pour la conservation des données et la synchronisation</span><span class="sxs-lookup"><span data-stu-id="492b7-130">[Storage][storage] accounts are used for the synchronisation and data retention</span></span>
* <span data-ttu-id="492b7-131">Les [groupes de machines virtuelles identiques][vmss] sont utilisés par CycleCloud pour des ressources de calcul</span><span class="sxs-lookup"><span data-stu-id="492b7-131">[Virtual Machine Scale Sets][vmss] are utilised by CycleCloud for compute resources</span></span>

## <a name="considerations"></a><span data-ttu-id="492b7-132">Considérations</span><span class="sxs-lookup"><span data-stu-id="492b7-132">Considerations</span></span>

### <a name="machine-sizes-available-for-azure-batch"></a><span data-ttu-id="492b7-133">Tailles de machine disponibles pour Azure Batch</span><span class="sxs-lookup"><span data-stu-id="492b7-133">Machine Sizes available for Azure Batch</span></span>
<span data-ttu-id="492b7-134">Bien que la plupart des clients effectuant un rendu aillent choisir les ressources avec une grande puissance de processeur, d’autres charges de travail utilisant des groupes de machines virtuelles identiques peuvent choisir des machines virtuelles différemment et dépendent de plusieurs facteurs :</span><span class="sxs-lookup"><span data-stu-id="492b7-134">While most rendering customers whill choose resources with high CPU power, other workloads using VM Scale Sets may choose VM's differently and will depend on a number of factors:</span></span>
  - <span data-ttu-id="492b7-135">Est-ce que l’application en cours d’exécution est liée à la mémoire ?</span><span class="sxs-lookup"><span data-stu-id="492b7-135">Is the Application being run memory bound?</span></span>
  - <span data-ttu-id="492b7-136">Est-ce que l’application a besoin d’utiliser un GPU ?</span><span class="sxs-lookup"><span data-stu-id="492b7-136">Does the Application need to use GPU's?</span></span> 
  - <span data-ttu-id="492b7-137">Est-ce que les types de travaux sont extrêmement parallèles ou nécessitent une connectivité Infiniband pour les travaux étroitement couplés ?</span><span class="sxs-lookup"><span data-stu-id="492b7-137">Are the job types embarassingly parallel or require Infiniband connectivity for tightly coupled jobs?</span></span>
  - <span data-ttu-id="492b7-138">Exiger des E/S rapides vers le stockage sur les nœuds de calcul</span><span class="sxs-lookup"><span data-stu-id="492b7-138">Require fast I/O to Storage on the Compute Nodes</span></span>

<span data-ttu-id="492b7-139">Azure propose une large plage de tailles de machine virtuelle qui peuvent répondre à chacune des exigences ci-dessus d’une application, certaines sont spécifiques à HPC, mais même les plus petites tailles peuvent être utilisées pour fournir une implémentation de grille efficace :</span><span class="sxs-lookup"><span data-stu-id="492b7-139">Azure has a wide range of VM sizes that can address each and every one of the above application requirements, some are specific to HPC, but even the smallest sizes can be utilised to provide an effective grid implementation:</span></span>

  - <span data-ttu-id="492b7-140">[Tailles de machine virtuelle HPC][compute-hpc] En raison de la nature du rendu qui est liée à l’UC, Microsoft suggère généralement les machines virtuelles Azure de série H.</span><span class="sxs-lookup"><span data-stu-id="492b7-140">[HPC VM sizes][compute-hpc] Due to the CPU bound nature of rendering, Microsoft typically suggests the Azure H-Series VM's.</span></span>  <span data-ttu-id="492b7-141">Elles sont spécifiquement générées pour les besoins d’un calcul haut de gamme, possèdent des formats de processeur virtuel de 8 et 16 cœurs, une mémoire DDR4, un stockage temporaire SSD et la technologie Intel Haswell E5.</span><span class="sxs-lookup"><span data-stu-id="492b7-141">These are built specifically available for high end computational needs, have 8 and 16 core vCPU sizes available, and feature DDR4 memory, SSD temporary storage and Haswell E5 Intel technology.</span></span>
  - <span data-ttu-id="492b7-142">[Tailles de machine virtuelle de GPU][compute-gpu] Les tailles de machine virtuelle au GPU optimisé sont des machines virtuelles spécialisées disponibles avec des GPU NVIDIA uniques ou multiples.</span><span class="sxs-lookup"><span data-stu-id="492b7-142">[GPU VM sizes][compute-gpu] GPU optimized VM sizes are specialized virtual machines available with single or multiple NVIDIA GPUs.</span></span> <span data-ttu-id="492b7-143">Ces tailles sont conçues pour des charges de travail de visualisation, mais également de calcul et d’affichage graphique intensifs.</span><span class="sxs-lookup"><span data-stu-id="492b7-143">These sizes are designed for compute-intensive, graphics-intensive, and visualization workloads.</span></span>
    - <span data-ttu-id="492b7-144">Les tailles des cœurs NC, NCv2, NCv3 et ND sont optimisées pour les applications nécessitant des ressources réseau et de calcul importantes, les algorithmes, notamment les applications et simulations CUDA et OpenCL, l’intelligence artificielle et le Deep Learning.</span><span class="sxs-lookup"><span data-stu-id="492b7-144">NC, NCv2, NCv3, and ND sizes are optimized for compute-intensive and network-intensive applications and algorithms, including CUDA and OpenCL-based applications and simulations, AI, and Deep Learning.</span></span> <span data-ttu-id="492b7-145">Les tailles des cœurs NV sont optimisées et conçues pour la visualisation à distance, la diffusion en continu, les jeux, le codage et les scénarios VDI utilisant des infrastructures comme OpenGL et DirectX.</span><span class="sxs-lookup"><span data-stu-id="492b7-145">NV sizes are optimized and designed for remote visualization, streaming, gaming, encoding, and VDI scenarios utilizing frameworks such as OpenGL and DirectX.</span></span>
  - <span data-ttu-id="492b7-146">[Tailles de machine virtuelle à mémoire optimisée][compute-memory] Lorsque davantage de mémoire est requise, les tailles de machines virtuelles à mémoire optimisée offrent un ratio mémoire/processeur supérieur.</span><span class="sxs-lookup"><span data-stu-id="492b7-146">[Memory optmised VM sizes][compute-memory] When more memory is required, the memory optimized VM sizes offer a higher memory-to-CPU ratio.</span></span>
  - <span data-ttu-id="492b7-147">[Tailles de machine virtuelle à usage général][compute-general] Les tailles de machines virtuelles à usage général sont également disponibles et fournissent un ratio mémoire/processeur équilibré.</span><span class="sxs-lookup"><span data-stu-id="492b7-147">[General purposes VM sizes][compute-general] General purpose VM sizes are also available and provide balanced CPU-to-memory ratio.</span></span>

### <a name="alternatives"></a><span data-ttu-id="492b7-148">Autres solutions</span><span class="sxs-lookup"><span data-stu-id="492b7-148">Alternatives</span></span>

<span data-ttu-id="492b7-149">Si vous souhaitez contrôler plus étroitement votre environnement de rendu dans Azure ou si vous avez besoin d’une implémentation hybride, le calcul CycleCloud peut aider à orchestrer une grille IaaS dans le cloud.</span><span class="sxs-lookup"><span data-stu-id="492b7-149">If you require more control over your rendering environment in Azure or need a hybrid implementation, then CycleCloud computing can help orchestrate an IaaS grid in the cloud.</span></span> <span data-ttu-id="492b7-150">En utilisant les mêmes technologies Azure sous-jacentes comme Azure Batch, la création et la maintenance d’une grille IaaS est un processus efficace.</span><span class="sxs-lookup"><span data-stu-id="492b7-150">Using the same underlying Azure technologies as Azure Batch, it makes building and maintaining an IaaS grid an efficient process.</span></span> <span data-ttu-id="492b7-151">Pour plus d’informations et en savoir plus sur les principes de conception, utilisez le lien suivant :</span><span class="sxs-lookup"><span data-stu-id="492b7-151">To find out more and learn about the design principles, please use the following link:</span></span>

<span data-ttu-id="492b7-152">Pour obtenir une présentation complète de toutes les solutions HPC disponibles dans Azure, consultez l’article [Solutions HPC, Batch et Big Compute à l’aide de machines virtuelles Azure][hpc-alt-solutions]</span><span class="sxs-lookup"><span data-stu-id="492b7-152">For a complete overview of all the HPC solutions that are available to you in Azure, please see the article [HPC, Batch, and Big Compute solutions using Azure VMs][hpc-alt-solutions]</span></span>

### <a name="availability"></a><span data-ttu-id="492b7-153">Disponibilité</span><span class="sxs-lookup"><span data-stu-id="492b7-153">Availability</span></span>

<span data-ttu-id="492b7-154">La surveillance des composants Azure Batch est disponible via une suite de services, d’outils et d’API.</span><span class="sxs-lookup"><span data-stu-id="492b7-154">Monitoring of the Azure Batch components is available through a range of services, tools and APIs.</span></span> <span data-ttu-id="492b7-155">Ce sujet est abordé plus en détail dans l’article [Surveiller les solutions Batch][batch-monitor].</span><span class="sxs-lookup"><span data-stu-id="492b7-155">This is discussed further in the [Monitor Batch solutions][batch-monitor] article.</span></span>

### <a name="scalability"></a><span data-ttu-id="492b7-156">Extensibilité</span><span class="sxs-lookup"><span data-stu-id="492b7-156">Scalability</span></span>

<span data-ttu-id="492b7-157">Les pools au sein d’un compte Azure Batch peuvent être mis à l’échelle via une intervention manuelle ou bien ils peuvent l’être automatiquement en utilisant une formule basée sur des métriques d’Azure Batch.</span><span class="sxs-lookup"><span data-stu-id="492b7-157">Pools within an Azure Batch account can either scale through manual intervention or, by using an formula based on Azure Batch metrics, be scaled automatically.</span></span> <span data-ttu-id="492b7-158">Pour plus d’informations, consultez l’article [Créer une formule de mise à l’échelle automatique pour la mise à l’échelle des nœuds dans un pool Batch][batch-scaling].</span><span class="sxs-lookup"><span data-stu-id="492b7-158">For more information on this, please see the article [Create an automatic scaling formula for scaling nodes in a Batch pool][batch-scaling].</span></span>

### <a name="security"></a><span data-ttu-id="492b7-159">Sécurité</span><span class="sxs-lookup"><span data-stu-id="492b7-159">Security</span></span>

<span data-ttu-id="492b7-160">Pour obtenir des conseils d’ordre général sur la conception de solutions sécurisées, consultez la [documentation sur la sécurité Azure][security].</span><span class="sxs-lookup"><span data-stu-id="492b7-160">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="492b7-161">Résilience</span><span class="sxs-lookup"><span data-stu-id="492b7-161">Resiliency</span></span>

<span data-ttu-id="492b7-162">Il n’existe actuellement aucune fonctionnalité de basculement dans Azure Batch, nous recommandons d’utiliser les étapes suivantes pour garantir la disponibilité en cas de panne non planifiée :</span><span class="sxs-lookup"><span data-stu-id="492b7-162">While there is currently no failover capability in Azure Batch, we recommend using the following steps to ensure availbility in case of an unplanned outage:</span></span>

* <span data-ttu-id="492b7-163">Créer un compte Azure Batch dans un autre emplacement Azure avec un autre compte de stockage</span><span class="sxs-lookup"><span data-stu-id="492b7-163">Create a Azure Batch account in an alternate Azure location with an alternate Storage Account</span></span>
* <span data-ttu-id="492b7-164">Créer les mêmes pools de nœud portant le même nom, avec 0 nœud alloué</span><span class="sxs-lookup"><span data-stu-id="492b7-164">Create the same node pools with the same name, with 0 nodes allocated</span></span>
* <span data-ttu-id="492b7-165">S’assurer que les applications sont créées et mises à jour vers le compte de stockage secondaire</span><span class="sxs-lookup"><span data-stu-id="492b7-165">Ensure Applications are created and updated to the alternate Storage Account</span></span>
* <span data-ttu-id="492b7-166">Charger des fichiers d’entrée et envoyer des travaux vers l’autre compte Azure Batch</span><span class="sxs-lookup"><span data-stu-id="492b7-166">Upload input files and submit jobs to the alternate Azure Batch account</span></span>

## <a name="deploy-this-scenario"></a><span data-ttu-id="492b7-167">Déployer ce scénario</span><span class="sxs-lookup"><span data-stu-id="492b7-167">Deploy this scenario</span></span>

### <a name="creating-an-azure-batch-account-and-pools-manually"></a><span data-ttu-id="492b7-168">Création manuelle d’un compte Azure Batch et des pools</span><span class="sxs-lookup"><span data-stu-id="492b7-168">Creating an Azure Batch account and pools manually</span></span>

<span data-ttu-id="492b7-169">Cet exemple de scénario vous aidera à apprendre le fonctionnement d’Azure Batch tout en utilisant Azure Batch Labs comme exemple de solution SaaS pouvant être développée pour vos propres clients :</span><span class="sxs-lookup"><span data-stu-id="492b7-169">This sample scenario will provide help in learning how Azure Batch works while showcasing Azure Batch Labs as an example SaaS solution that can be developed for your own customers:</span></span>

<span data-ttu-id="492b7-170">[Azure Batch Masterclass][batch-labs-masterclass]</span><span class="sxs-lookup"><span data-stu-id="492b7-170">[Azure Batch Masterclass][batch-labs-masterclass]</span></span>

### <a name="deploying-the-sample-scenario-using-an-azure-resource-manager-arm-template"></a><span data-ttu-id="492b7-171">Déploiement de l’exemple de scénario à l’aide d’un modèle Azure Resource Manager (ARM)</span><span class="sxs-lookup"><span data-stu-id="492b7-171">Deploying the sample scenario using an Azure Resource Manager (ARM) template</span></span>

<span data-ttu-id="492b7-172">Le modèle déploiera :</span><span class="sxs-lookup"><span data-stu-id="492b7-172">The template will deploy:</span></span>
  - <span data-ttu-id="492b7-173">Un nouveau compte Azure Batch</span><span class="sxs-lookup"><span data-stu-id="492b7-173">A new Azure Batch account</span></span>
  - <span data-ttu-id="492b7-174">Un compte de stockage</span><span class="sxs-lookup"><span data-stu-id="492b7-174">A storage account</span></span>
  - <span data-ttu-id="492b7-175">Un pool de nœuds associé au compte Batch</span><span class="sxs-lookup"><span data-stu-id="492b7-175">A node pool associated with the batch account</span></span>
  - <span data-ttu-id="492b7-176">Le pool de nœuds sera configuré pour utiliser des machines virtuelles A2 v2 avec les images de Ubuntu Canonical</span><span class="sxs-lookup"><span data-stu-id="492b7-176">The node pool will be configured to use A2 v2 VMs with Canonical Ubuntu images</span></span>
  - <span data-ttu-id="492b7-177">Le pool de nœuds ne contient initialement aucune machine virtuelle et nécessite une mise à l’échelle manuelle pour ajouter des machines virtuelles</span><span class="sxs-lookup"><span data-stu-id="492b7-177">The node pool will contain 0 VMs initially and will require scaling manually to add VMs</span></span>

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fhpc%2Fbatchcreatewithpools.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

<span data-ttu-id="492b7-178">[En savoir plus sur les modèles ARM][azure-arm-templates]</span><span class="sxs-lookup"><span data-stu-id="492b7-178">[Learn more about ARM templates][azure-arm-templates]</span></span>

## <a name="pricing"></a><span data-ttu-id="492b7-179">Tarifs</span><span class="sxs-lookup"><span data-stu-id="492b7-179">Pricing</span></span>

<span data-ttu-id="492b7-180">Le coût d’utilisation d’Azure Batch varie selon les tailles des machines virtuelles utilisées pour les pools et la durée pendant laquelle elles sont allouées et en cours d’exécution, aucun coût n’est associé à la création d’un compte Azure Batch.</span><span class="sxs-lookup"><span data-stu-id="492b7-180">The cost of using Azure Batch will depend on the VM sizes that are used for the pools and how long these are allocated and running, there is no cost associated with an Azure Batch account creation.</span></span> <span data-ttu-id="492b7-181">Le stockage et la sortie des de données doivent également être pris en compte car cela occasionnera des frais supplémentaires.</span><span class="sxs-lookup"><span data-stu-id="492b7-181">Storage and data egress should also be taken into account as these will apply additional costs.</span></span>

<span data-ttu-id="492b7-182">Voici quelques exemples des coûts qui pourraient être engagés pour une tâche se terminant en 8 heures à l’aide de différents nombres de serveurs :</span><span class="sxs-lookup"><span data-stu-id="492b7-182">The following are examples of costs that could be incurred for a job that completes in 8 hours using a different number of servers:</span></span>


- <span data-ttu-id="492b7-183">100 machines virtuelles avec un processeur hautes performances : [Estimation des coûts][hpc-est-high]</span><span class="sxs-lookup"><span data-stu-id="492b7-183">100 High Performance CPU VMs: [Cost Estimate][hpc-est-high]</span></span>

  <span data-ttu-id="492b7-184">100 x H16m (16 cœurs, 225 Go de RAM, stockage Premium de 512 Go), stockage d’objets Blob de 2 To, sortie de 1 To</span><span class="sxs-lookup"><span data-stu-id="492b7-184">100 x H16m (16 cores, 225GB RAM, Premium Storage 512GB), 2 TB Blob Storage, 1 TB egress</span></span>

- <span data-ttu-id="492b7-185">50 machines virtuelles avec un processeur hautes performances : [Estimation des coûts][hpc-est-med]</span><span class="sxs-lookup"><span data-stu-id="492b7-185">50 High Performance CPU VMs: [Cost Estimate][hpc-est-med]</span></span>

  <span data-ttu-id="492b7-186">50 x H16m (16 cœurs, 225 Go de RAM, stockage Premium de 512 Go), stockage d’objets Blob de 2 To, sortie de 1 To</span><span class="sxs-lookup"><span data-stu-id="492b7-186">50 x H16m (16 cores, 225GB RAM, Premium Storage 512GB), 2 TB Blob Storage, 1 TB egress</span></span>

- <span data-ttu-id="492b7-187">10 machines virtuelles avec un processeur hautes performances : [Estimation des coûts][hpc-est-low]</span><span class="sxs-lookup"><span data-stu-id="492b7-187">10 High Performance CPU VMs: [Cost Estimate][hpc-est-low]</span></span>
  
  <span data-ttu-id="492b7-188">10 x H16m (16 cœurs, 225 Go de RAM, stockage Premium de 512 Go), stockage d’objets Blob de 2 To, sortie de 1 To</span><span class="sxs-lookup"><span data-stu-id="492b7-188">10 x H16m (16 cores, 225GB RAM, Premium Storage 512GB), 2 TB Blob Storage, 1 TB egress</span></span>

### <a name="low-priority-vm-pricing"></a><span data-ttu-id="492b7-189">Tarification des machines virtuelles basse priorité</span><span class="sxs-lookup"><span data-stu-id="492b7-189">Low Priority VM Pricing</span></span>

<span data-ttu-id="492b7-190">Azure Batch prend également en charge l’utilisation de machines virtuelles basse priorité\* dans les pools de nœuds, ce qui permet de profiter d’une économie substantielle.</span><span class="sxs-lookup"><span data-stu-id="492b7-190">Azure Batch also supports the use of Low Priority VMs\* in the node pools, which can potentially provide a substantial cost saving.</span></span> <span data-ttu-id="492b7-191">Pour une comparaison de prix entre les machines virtuelles standards et les machines virtuelles basse priorité et pour en savoir plus sur les machines virtuelles basse priorité, consultez [Tarification Batch][batch-pricing].</span><span class="sxs-lookup"><span data-stu-id="492b7-191">For a price comparison between standard VMs and Low Priority VMs, and to find out more about Low Priority VMs, please see [Batch Pricing][batch-pricing].</span></span>

<span data-ttu-id="492b7-192">\* Veuillez noter que seules certaines applications et charges de travail pourront être exécutées sur des machines virtuelles basse priorité.</span><span class="sxs-lookup"><span data-stu-id="492b7-192">\* Please note that only certain applications and workloads will be suitable to run on Low Priority VMs.</span></span>

## <a name="related-resources"></a><span data-ttu-id="492b7-193">Ressources associées</span><span class="sxs-lookup"><span data-stu-id="492b7-193">Related Resources</span></span>

<span data-ttu-id="492b7-194">[Vue d'ensemble de Azure Batch][batch-overview]</span><span class="sxs-lookup"><span data-stu-id="492b7-194">[Azure Batch Overview][batch-overview]</span></span>

<span data-ttu-id="492b7-195">[Documentation de Azure Batch][batch-doc]</span><span class="sxs-lookup"><span data-stu-id="492b7-195">[Azure Batch Documentation][batch-doc]</span></span>

<span data-ttu-id="492b7-196">[Utilisation des conteneurs Docker sur Azure Batch][batch-containers]</span><span class="sxs-lookup"><span data-stu-id="492b7-196">[Using containers on Azure Batch][batch-containers]</span></span>

<!-- links -->
[architecture]: ./media/native-hpc-ref-arch.png
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[resiliency]: /azure/architecture/resiliency/
[scalability]: /azure/architecture/checklist/scalability
[vmss]: /azure/virtual-machine-scale-sets/overview
[vnet]: /azure/virtual-network/virtual-networks-overview
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
[batch-pricing]: https://azure.microsoft.com/en-gb/pricing/details/batch/
[batch-doc]: /azure/batch/
[batch-overview]: https://azure.microsoft.com/services/batch/
[batch-containers]: https://github.com/Azure/batch-shipyard
[azure-arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[batch-plugins]: /azure/batch/batch-rendering-service#options-for-submitting-a-render-job