---
title: Rendu vidéo 3D sur Azure
description: Exécution des charges de travail HPC natives dans Azure à l’aide du service Azure Batch
author: adamboeglin
ms.date: 07/13/2018
ms.openlocfilehash: 1d51a9812c890a4877ec0edfdcbf344b27049812
ms.sourcegitcommit: b7e521ba317f4fcd3253c80ac0c0a355eaaa56c5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/21/2018
ms.locfileid: "46533930"
---
# <a name="3d-video-rendering-on-azure"></a>Rendu vidéo 3D sur Azure

Le rendu 3D est un processus qui prend beaucoup de temps et nécessitant une quantité importante de temps processeur pour être terminé.  Sur une seule machine, le processus de génération d’un fichier vidéo à partir de ressources statiques peut prendre plusieurs heures, voire plusieurs jours selon la longueur et la complexité de la vidéo que vous générez.  De nombreuses entreprises achètent des ordinateurs haut de gamme coûteux pour effectuer ces tâches, ou bien elles investissent dans des groupes de rendu volumineux auxquels ils peuvent envoyer des travaux.  Toutefois, en tirant parti d’Azure Batch, cette puissance est disponible lorsque vous en avez besoin et s’arrête lorsqu’elle n’est plus nécessaire, tout cela sans avoir à ne consentir aucun investissement.

Batch fournit une expérience cohérente de gestion et de planification des travaux pour les nœuds de calcul Windows Server ou Linux. Avec Batch, vous pouvez utiliser vos applications Windows ou Linux existantes, notamment AutoDesk Maya et Blender, pour exécuter des travaux de rendu à grande échelle dans Azure.

## <a name="related-use-cases"></a>Cas d’usage connexes

Pensez à ce scénario pour ces cas d’usage similaires :

* Modélisation 3D
* Rendu FX Visual (VFX)
* Transcodage vidéo
* Traitement d’image, correction des couleurs et redimensionnement

## <a name="architecture"></a>Architecture

![Présentation de l’architecture des composants impliqués dans une solution HPC native Cloud à l’aide d’Azure Batch][architecture]

Cet exemple de scénario décrit le flux de travail lors de l’utilisation d’Azure Batch, les données circulent comme suit :

1. Charger les fichiers d’entrée et les applications nécessaires pour le traitement des fichiers dans votre compte Stockage Azure
2. Créer un pool Batch de nœuds de calcul dans votre compte Batch, un travail pour exécuter la charge de travail sur le pool et des tâches dans le travail.
3. Télécharger des fichiers d’entrée et des applications vers Azure Batch
4. Surveiller l’exécution d’une tâche
5. Charger les sorties des tâches
6. Télécharger les fichiers de sortie

Pour simplifier ce processus, vous pouvez également utiliser les [plug-ins de Batch pour Maya et 3ds Max][batch-plugins]

### <a name="components"></a>Composants

Azure Batch s’appuie sur les technologies Azure suivantes :

* Les [groupes de ressources][resource-groups] sont des conteneurs logiques pour des ressources Azure.
* Les [réseaux virtuels][vnet] sont utilisés pour le nœud principal et les ressources de calcul
* Les comptes de [stockage][storage] sont utilisés pour la synchronisation et la conservation des données
* Les [groupes de machines virtuelles identiques][vmss] sont utilisés par CycleCloud pour les ressources de calcul

## <a name="considerations"></a>Considérations

### <a name="machine-sizes-available-for-azure-batch"></a>Tailles de machine disponibles pour Azure Batch

Bien que la plupart des clients optent pour des ressources qui possèdent une grande puissance de processeur, d’autres charges de travail utilisant des groupes de machines virtuelles identiques peuvent impliquer des critères de choix des machines virtuelles différents, selon plusieurs facteurs :

* L’application qui s’exécute est-elle memory-bound ?
* A-t-elle besoin d’utiliser des GPU ? 
* Les types de travaux sont-ils extrêmement parallèles ? Réclament-ils une connectivité InfiniBand pour les travaux étroitement couplés ?
* Des E/S rapides vers le stockage sur les nœuds de calcul sont-elles nécessaires ?

Azure propose un large éventail de tailles de machine virtuelle qui répondent à chacune des exigences des applications ci-dessus ; certaines sont spécifiquement adaptées au calcul haute performance (HPC), mais même les plus petites tailles peuvent être utilisées pour fournir une implémentation de grille efficace :

* [Tailles de machine virtuelle HPC][compute-hpc] Le rendu étant CPU-bound, Microsoft suggère généralement les machines virtuelles Azure de série H.  Les machines virtuelles de ce type, spécialement conçues pour les besoins élevés de calcul, proposent des processeurs virtuels de 8 ou 16 cœurs, une mémoire DDR4, du stockage temporaire SSD et la technologie Intel Haswell E5.
* [Tailles de machine virtuelle de GPU][compute-gpu] Les tailles de machine virtuelle au GPU optimisé sont des machines virtuelles spécialisées disponibles avec des GPU NVIDIA uniques ou multiples. Ces tailles sont conçues pour des charges de travail de visualisation, mais également de calcul et d’affichage graphique intensifs.
* Les tailles des cœurs NC, NCv2, NCv3 et ND sont optimisées pour les applications nécessitant des ressources réseau et de calcul importantes, les algorithmes, notamment les applications et simulations CUDA et OpenCL, l’intelligence artificielle et le Deep Learning. Les tailles des cœurs NV sont optimisées et conçues pour la visualisation à distance, la diffusion en continu, les jeux, le codage et les scénarios VDI utilisant des infrastructures comme OpenGL et DirectX.
* [Tailles de machine virtuelle à mémoire optimisée][compute-memory] Lorsque le besoin de mémoire augmente, les tailles de machines virtuelles à mémoire optimisée offrent un rapport mémoire/processeur supérieur.
* [Tailles de machine virtuelle à usage général][compute-general] Des tailles de machines virtuelles à usage général sont également disponibles ; elles offrent un ratio mémoire/processeur équilibré.

### <a name="alternatives"></a>Autres solutions

Si vous souhaitez contrôler plus étroitement votre environnement de rendu dans Azure ou si vous avez besoin d’une implémentation hybride, le calcul CycleCloud peut aider à orchestrer une grille IaaS dans le cloud. En utilisant les mêmes technologies Azure sous-jacentes comme Azure Batch, la création et la maintenance d’une grille IaaS est un processus efficace. Pour plus d’informations, notamment sur les principes de conception, suivez le lien ci-dessous :

Pour une vue d’ensemble complète de toutes les solutions HPC disponibles dans Azure, voir l’article [Solutions HPC, Batch et Big Compute avec les machines virtuelles Azure][hpc-alt-solutions].

### <a name="availability"></a>Disponibilité

Le monitoring des composants Azure Batch est possible avec différents services, outils et API. Pour plus d’informations, voir l’article [Effectuer le monitoring des solutions Batch][batch-monitor].

### <a name="scalability"></a>Extensibilité

Les pools qui se trouvent dans un compte Azure Batch peuvent être mis à l’échelle par intervention manuelle ou bien automatiquement avec une formule qui s’appuie sur les mesures d’Azure Batch. Pour plus d’informations sur l’évolutivité, voir l’article [Créer une formule de mise à l’échelle automatique pour les nœuds d’un pool Batch][batch-scaling].

### <a name="security"></a>Sécurité

Pour obtenir des conseils d’ordre général sur la conception de solutions sécurisées, consultez la [documentation sur la sécurité Azure][security].

### <a name="resiliency"></a>Résilience

Il n’existe pour le moment aucune fonctionnalité de basculement dans Azure Batch. Nous vous recommandons de suivre les étapes ci-dessous pour garantir la disponibilité en cas de panne non planifiée :

* Créer un compte Azure Batch dans un autre emplacement Azure avec un autre compte de stockage
* Créer les mêmes pools de nœud avec le même nom et aucun nœud alloué
* S’assurer que les applications sont créées et mises à jour vers le compte de stockage secondaire
* Charger des fichiers d’entrée et envoyer des travaux vers l’autre compte Azure Batch

## <a name="deploy-this-scenario"></a>Déployer ce scénario

### <a name="creating-an-azure-batch-account-and-pools-manually"></a>Création manuelle d’un compte Azure Batch et des pools

Cet exemple de scénario aide à apprendre le fonctionnement d’Azure Batch tout en présentant Azure Batch Labs comme un exemple de solution SaaS que vous pouvez développer pour vos propres clients :

[Azure Batch Masterclass][batch-labs-masterclass]

### <a name="deploying-the-sample-scenario-using-an-azure-resource-manager-template"></a>Déploiement de l’exemple de scénario suivant un modèle Azure Resource Manager

Le modèle déploiera :

* Un nouveau compte Azure Batch
* Un compte de stockage
* Un pool de nœuds associé au compte Batch
* Le pool de nœuds sera configuré pour utiliser des machines virtuelles A2 v2 avec les images de Ubuntu Canonical
* Le pool de nœuds ne contient initialement aucune machine virtuelle ; une mise à l’échelle manuelle sera nécessaire pour en ajouter

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fhpc%2Fbatchcreatewithpools.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

[Plus d’informations sur les modèles Resource Manager][azure-arm-templates]

## <a name="pricing"></a>Tarifs

Le coût d’utilisation d’Azure Batch varie selon les tailles de machine virtuelle utilisées pour les pools et la durée d’allocation et d’exécution de ces machines virtuelles ; la création d’un compte Azure Batch est gratuite. Le stockage et la sortie des données doivent être pris en compte, car ils occasionnent des frais supplémentaires.

Voici quelques exemples des coûts qui pourraient être engagés pour une tâche se terminant en 8 heures à l’aide de différents nombres de serveurs :

* 100 machines virtuelles avec processeur hautes performances : [Devis estimatif][hpc-est-high]

  100 H16m (16 cœurs, 225 Go de RAM, 512 Go de Stockage Premium), 2 To de Stockage Blob, 1 To de sortie

* 50 machines virtuelles avec processeur hautes performances : [Devis estimatif][hpc-est-med]

  50 H16m (16 cœurs, 225 Go de RAM, 512 Go de Stockage Premium), 2 To de Stockage Blob, 1 To de sortie

* 10 machines virtuelles avec processeur hautes performances : [Devis estimatif][hpc-est-low]
  
  10 H16m (16 cœurs, 225 Go de RAM, 512 Go de Stockage Premium), 2 To de Stockage Blob, 1 To de sortie

### <a name="low-priority-vm-pricing"></a>Prix des machines virtuelles basse priorité

Azure Batch prend également en charge les machines virtuelles basse priorité* dans les pools de nœuds, qui peuvent représenter des économies substantielles. Pour plus d’informations sur les machines virtuelles basse priorité, et notamment une comparaison de prix avec les machines virtuelles standard, voir [Prix Batch][batch-pricing].

\* Veuillez noter que les machines virtuelles basse priorité ne sont adaptées qu’à certaines applications et charges de travail.

## <a name="related-resources"></a>Ressources associées

[Vue d'ensemble de Azure Batch][batch-overview]

[Documentation de Azure Batch][batch-doc]

[Utilisation des conteneurs Docker sur Azure Batch][batch-containers]

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