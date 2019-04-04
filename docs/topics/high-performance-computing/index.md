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

# <a name="high-performance-computing-hpc-on-azure"></a>Calcul haute performance (HPC) sur Azure

## <a name="introduction-to-hpc"></a>Introduction à HPC

<!-- markdownlint-disable MD034 -->

> [!VIDEO https://www.youtube.com/embed/rKURT32faJk]

<!-- markdownlint-enable MD034 -->

HPC (High-Performance Computing, calcul haute performance), également appelé « Big Compute », utilise un grand nombre d’ordinateurs basés sur le processeur ou sur GPU pour résoudre des tâches mathématiques complexes.

De nombreux secteurs d’activité utilisent HPC pour résoudre certains de leurs problèmes les plus complexes.  Ces problèmes incluent notamment des charges de travail telles que les suivantes :

- Genomics
- Simulations relatives aux hydrocarbures
- Finances
- Conception de semiconducteurs
- Ingénierie
- Modélisation de phénomènes météorologiques

### <a name="how-is-hpc-different-on-the-cloud"></a>En quoi HPC est différent sur le cloud ?

L’une des principales différences entre un système HPC local et un système HPC dans le cloud est la possibilité d’ajouter et de supprimer dynamiquement des ressources en fonction des besoins.  La mise à l’échelle dynamique supprime la capacité de calcul comme goulot d’étranglement et, à la place, permet aux clients de trouver la taille adaptée pour leur infrastructure pour satisfaire les exigences de leurs travaux.

Les articles suivants fournissent davantage de détails sur cette fonctionnalité de mise à l’échelle dynamique.

- [Style d’architecture Big Compute](/azure/architecture/guide/architecture-styles/big-compute?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [Bonnes pratiques concernant la mise à l’échelle automatique](/azure/architecture/best-practices/auto-scaling?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="implementation-checklist"></a>Liste de vérification de l’implémentation

Comme vous cherchez à implémenter votre propre solution HPC sur Azure, vérifiez que vous avez consulté les rubriques suivantes :

<!-- markdownlint-disable MD032 -->

> [!div class="checklist"]
> - Choisir l’[architecture](#infrastructure) appropriée en fonction de vos exigences
> - Déterminer quelles options de [calcul](#compute) conviennent pour votre charge de travail
> - Identifier la solution de [stockage](#storage) appropriée qui répond à vos besoins
> - Décider comment vous allez [gérer](#management) toutes vos ressources
> - Optimiser votre [application](#hpc-applications) pour le cloud
> - [Sécuriser](#security) votre infrastructure

<!-- markdownlint-enable MD032 -->

## <a name="infrastructure"></a>Infrastructure

Un certain nombre de composants d’infrastructure sont nécessaires à la génération d’un système HPC.  Calcul, stockage et réseau fournissent les composants sous-jacents, quelle que soit la façon dont vous choisissez de gérer vos charges de travail HPC.

### <a name="example-hpc-architectures"></a>Exemples d’architectures HPC

Il existe différentes façons de concevoir et d’implémenter votre architecture HPC sur Azure.  Les applications HPC peuvent être mises à l’échelle pour plusieurs milliers de cœurs de calcul, étendre des clusters locaux ou s’exécuter en tant que solutions natives entièrement dans le cloud.

Les scénarios suivants décrivent quelques-unes des façons courantes de générer des solutions HPC.

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
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Services d’ingénierie assistée par ordinateur sur Azure</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Offrez une plateforme de software as a service (SaaS) pour l’ingénierie assistée par ordinateur (IAO) sur Azure.</p>
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
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Simulations de diagramme de flux cumulé (CFD) sur Azure</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Exécutez des simulations de diagramme de flux cumulé (CFD) sur Azure.</p>
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
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Rendu vidéo 3D sur Azure</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Exécutez des charges de travail HPC natives dans Azure à l’aide du service Azure Batch.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

### <a name="compute"></a>Calcul

Azure propose une gamme de tailles qui sont optimisées pour les charges de travail intensives processeur et GPU.

#### <a name="cpu-based-virtual-machines"></a>Machines virtuelles basées sur le processeur

- [Machines virtuelles Linux](/azure/virtual-machines/linux/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [Machines virtuelles Windows](/azure/virtual-machines/windows/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
  
#### <a name="gpu-enabled-virtual-machines"></a>Machines virtuelles compatibles GPU

Les machines virtuelles de série N comportent des processeurs graphiques NVIDIA conçus pour des applications graphiques ou de calcul nécessitant beaucoup de ressources système, y compris la visualisation et l’apprentissage de l’intelligence artificielle (AI).

- [Machines virtuelles Linux](/azure/virtual-machines/linux/sizes-gpu?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [Machines virtuelles Windows](/azure/virtual-machines/windows/sizes-gpu?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

### <a name="storage"></a>Stockage

Les charges de travail HPC et Batch à grande échelle nécessitent un stockage des données et un accès à ces dernières dépassant les capacités des systèmes de fichiers cloud classiques.  Un certain nombre de solutions permettent de gérer les besoins de vitesse et de capacité des applications HPC sur Azure

- [Avere vFXT](https://azure.microsoft.com/services/storage/avere-vfxt/) pour un stockage de données plus rapide et plus accessible pour un calcul haute performance à la périphérie
- [BeeGFS](https://azure.microsoft.com/resources/implement-glusterfs-on-azure/)
- [Machines virtuelles à stockage optimisé](/azure/virtual-machines/windows/sizes-storage?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [Stockage Objet BLOB, Table et File d’attente](/azure/storage/storage-introduction?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [Stockage de fichiers SMB Azure](/azure/storage/files/storage-files-introduction?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [Intel Cloud Edition pour Lustre](https://azuremarketplace.microsoft.com/marketplace/apps/intel.intel-cloud-edition-gs)

Pour plus d’informations sur la comparaison de Lustre, de GlusterFS et de BeeGFS sur Azure, consultez le [livre électronique Systèmes de fichiers parallèles sur Azure](https://blogs.msdn.microsoft.com/azurecat/2018/06/11/azurecat-ebook-parallel-virtual-file-systems-on-microsoft-azure/)

### <a name="networking"></a>Mise en réseau

Les machines virtuelles H16r, H16mr, A8 et A9 peuvent se connecter à un réseau RDMA back-end à débit élevé. Ce réseau peut améliorer les performances d’applications parallèles étroitement liées s’exécutant sous Microsoft MPI ou Intel MPI.

- [Instances compatibles RDMA](/azure/virtual-machines/windows/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context#rdma-capable-instances)
- [Réseau virtuel](/azure/virtual-network/virtual-networks-overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [ExpressRoute](/azure/expressroute/expressroute-introduction?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="management"></a>gestion

### <a name="do-it-yourself"></a>À faire soi-même

La génération d’un système HPC à partir de zéro sur Azure offre une grande flexibilité, mais elle nécessite souvent beaucoup de maintenance.  

1. Configurez votre environnement de cluster dans des machines virtuelles Azure ou dans des [groupes de machines virtuelles identiques](/azure/virtual-machine-scale-sets/overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context).
2. Utilisez les modèles Azure Resource Manager pour déployer des [gestionnaires de charge de travail](#workload-managers) principaux, des infrastructures et des [applications](#hpc-applications).
3. Choisissez des [tailles de machine virtuelle](#compute) HPC et GPU qui comprennent du matériel spécialisé et des connexions réseau pour les charges de travail MPI ou GPU.
4. Ajoutez du [stockage de haute performance](#storage) aux charges de travail intensives d’E/S.

### <a name="hybrid-and-cloud-bursting"></a>Éclatement hybride et cloud bursting

Si vous avez un système HPC local existant que vous voulez connecter à Azure, il existe de nombreuses ressources pour vous aider à commencer.

Tout d’abord, consultez l’article [Options pour la connexion d’un réseau local à Azure](/azure/architecture/reference-architectures/hybrid-networking/?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) de la documentation.  Alors, vous voudrez peut-être des informations sur les options de connectivité suivantes :

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
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Connecter un réseau local à Azure à l’aide d’une passerelle VPN</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Cette architecture de référence montre comment étendre un réseau local à Azure à l’aide d’un réseau privé virtuel (VPN) site à site.</p>
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
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Connecter un réseau local à Azure à l’aide d’ExpressRoute</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Les connexions ExpressRoute utilisent une connexion privée et dédiée via un fournisseur de connectivité tiers. La connexion privée étend votre réseau local à Azure.</p>
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
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Connecter un réseau local à Azure à l’aide d’ExpressRoute avec basculement VPN</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Implémentez une architecture réseau de site à site sécurisée et hautement disponible qui s’étend sur un réseau virtuel Azure et un réseau local connecté à l’aide d’ExpressRoute avec basculement de passerelle VPN.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

Une fois la connectivité réseau établie de manière sécurisée, vous pouvez commencer à utiliser des ressources de calcul cloud à la demande avec les fonctionnalités d’éclatement de votre [gestionnaire de charges de travail](#workload-managers) existant.

### <a name="marketplace-solutions"></a>Solutions de la Place de marché

Un certain nombre de gestionnaires de charges de travail sont proposés dans la [Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/).

- [HPC basé sur RogueWave CentOS](https://azuremarketplace.microsoft.com/marketplace/apps/RogueWave.CentOSbased73HPC?tab=Overview)
- [SUSE Linux Enterprise Server (SLES)](https://azure.microsoft.com/marketplace/partners/suse/suselinuxenterpriseserver12optimizedforhighperformancecompute/)
- [TIBCO Grid Server Engine](https://azuremarketplace.microsoft.com/marketplace/apps/tibco-software.gridserverlinuxengine?tab=Overview)
- [Machine virtuelle de science des données Azure pour Windows et Linux](/azure/machine-learning/data-science-virtual-machine/overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [D3View](https://azuremarketplace.microsoft.com/marketplace/apps/xfinityinc.d3view-v5?tab=Overview)
- [UberCloud](https://azure.microsoft.com/search/marketplace/?q=ubercloud)

### <a name="azure-batch"></a>Azure Batch

[Azure Batch](/azure/batch/batch-technical-overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) est une plateforme qui permet d’exécuter efficacement des applications de calcul haute performance (HPC) en parallèle et à grande échelle dans le cloud. Azure Batch planifie les travaux nécessitant une grande quantité de ressources système à exécuter sur un pool géré de machines virtuelles. Il peut mettre automatiquement à l’échelle les ressources de calcul pour répondre aux besoins de vos travaux.

Les fournisseurs et développeurs SaaS peuvent utiliser les outils et kits de développement logiciel (SDK) pour intégrer des applications HPC ou des charges de travail de conteneur dans Azure, stocker des données dans Azure et générer des pipelines d’exécution du travail.

### <a name="azure-cyclecloud"></a>Azure CycleCloud

[Azure CycleCloud](/azure/cyclecloud/?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) La manière la plus simple de gérer les charges de travail HPC à l’aide d’un planificateur (tel que Slurm, Grid Engine, HPC Pack, HTCondor, LSF, PBS Pro ou Symphony), sur Azure

CycleCloud vous permet d’effectuer les opérations suivantes :

- Déployer des clusters complets et d’autres ressources, notamment le planificateur, les machines virtuelles de calcul, le stockage, la mise en réseau et le cache
- Orchestrer des workflows liés aux travaux, aux données et au cloud
- Donner aux administrateurs un contrôle total sur les utilisateurs qui peuvent exécuter des travaux, l’emplacement de ces travaux et leur coût
- Personnaliser et optimiser les clusters par le biais des fonctionnalités de stratégie et de gouvernance avancées, notamment le contrôle des coûts, l’intégration d’Active Directory, la supervision et la création de rapports
- Utiliser votre planificateur de travaux et vos applications actuels sans modification
- Tirer parti de la mise à l’échelle automatique intégrée et des architectures de référence testées sur le terrain pour un large éventail de secteurs et de charges de travail HPC

### <a name="workload-managers"></a>Gestionnaires de charges de travail

Voici quelques exemples de cluster et de gestionnaires de charges de travail qui peuvent s’exécuter dans une architecture Azure. Créer des clusters autonomes dans des machines virtuelles Azure ou effectuez un burst dans les machines virtuelles Azure depuis un cluster local.

- [Alces Flight Compute](https://azuremarketplace.microsoft.com/marketplace/apps/alces-flight-limited.alces-flight-compute-solo?tab=Overview)
- [TIBCO DataSynapse GridServer](https://azure.microsoft.com/blog/tibco-datasynapse-comes-to-the-azure-marketplace/)
- [Bright Cluster Manager](http://www.brightcomputing.com/technology-partners/microsoft)
- [IBM Spectrum Symphony and Symphony LSF](https://azure.microsoft.com/blog/ibm-and-microsoft-azure-support-spectrum-symphony-and-spectrum-lsf/)
- [PBS Pro](http://pbspro.org)
- [Altair](http://www.altair.com/)
- [Rescale](https://www.rescale.com/azure/)
- [Microsoft HPC Pack](https://technet.microsoft.com/library/mt744885.aspx)
  - [HPC Pack pour Windows](/azure/virtual-machines/windows/hpcpack-cluster-options?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
  - [HPC Pack pour Linux](/azure/virtual-machines/linux/hpcpack-cluster-options?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

#### <a name="containers"></a>Containers

Les conteneurs peuvent également être utilisés pour gérer certaines charges de travail HPC.  Des services comme Azure Kubernetes Service (AKS) simplifie le déploiement d’un cluster Kubernetes managé dans Azure.

- [Azure Kubernetes Service (AKS)](/azure/aks/intro-kubernetes?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [Container Registry](/azure/container-registry/container-registry-intro?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="cost-management"></a>la gestion des coûts ;

Il existe différentes manières de gérer vos coûts HPC sur Azure.  Vérifiez que vous avez consulté les [options d’achat Azure](https://azure.microsoft.com/pricing/purchase-options/) afin de trouver la méthode qui convient le mieux à votre organisation.

Les [machines virtuelles basse priorité](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-use-low-priority?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) vous permettent de disposer de notre capacité inutilisée en réalisant des économies significatives.

## <a name="security"></a>Sécurité

Pour obtenir une vue d’ensemble des bonnes pratiques de sécurité sur Azure, consultez la [documentation sur la sécurité Azure](/azure/security/azure-security?context=/azure/architecture/topics/high-performance-computing/context/hpc-context).  

Outre les configurations réseau disponibles dans la section [Cloud bursting](#hybrid-and-cloud-bursting), vous pouvez souhaiter implémenter une configuration hub/spoke pour isoler vos ressources de calcul :

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
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Implémenter une topologie de réseau hub-and-spoke dans Azure</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Le hub est un réseau virtuel (VNet) dans Azure qui centralise la connectivité à votre réseau local. Les rayons (spokes) sont des réseaux virtuels qui s’homologuent avec le hub et qui peuvent être utilisés pour isoler les charges de travail.</p>
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
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Implémentation d’une topologie de réseau hub-and-spoke avec des services partagés dans Azure</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Cette architecture de référence s’appuie sur l’architecture de référence hub-and-spoke de manière à inclure dans le hub des services partagés qui peuvent être utilisés par tous les spokes.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="hpc-applications"></a>Applications HPC

Exécuter des applications HPC commerciales ou personnalisées dans Azure. Plusieurs exemples dans cette section ont été testés et se montrent efficaces pour la mise à l’échelle avec des machines virtuelles ou des cœurs de calcul supplémentaires. Visitez la [Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace) pour obtenir des solutions prêtes au déploiement.

> [!NOTE]
> Vérifiez auprès du fournisseur de toute application commerciale les questions de licence ou toute autre restriction relative à l’exécution dans le cloud. Tous les fournisseurs ne proposent pas le paiement à l'utilisation pour les licences. Vous aurez peut-être besoin d’un serveur de licences dans le cloud pour votre solution ou de vous connecter à un serveur de licences sur site.

### <a name="engineering-applications"></a>Applications d’ingénierie

- [Altair RADIOSS](https://azure.microsoft.com/blog/availability-of-altair-radioss-rdma-on-microsoft-azure/)
- [ANSYS CFD](https://azure.microsoft.com/blog/ansys-cfd-and-microsoft-azure-perform-the-best-hpc-scalability-in-the-cloud/)
- [MATLAB Distributed Computing Server](/azure/virtual-machines/windows/matlab-mdcs-cluster?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [StarCCM +](https://blogs.msdn.microsoft.com/azurecat/2017/07/07/run-star-ccm-in-an-azure-hpc-cluster/)
- [OpenFOAM](https://simulation.azure.com/casestudies/Team-182-ABB-UC-Final.pdf)

### <a name="graphics-and-rendering"></a>Graphiques et rendu

- [Autodesk Maya, 3ds Max et Arnold](/azure/batch/batch-rendering-service?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) sur Azure Batch

### <a name="ai-and-deep-learning"></a>Intelligence artificielle et apprentissage approfondi

- [Microsoft Cognitive Toolkit](/cognitive-toolkit/cntk-on-azure)
- [Machine virtuelle d’apprentissage profond](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-ads.dsvm-deep-learning)
- [Recettes Batch Shipyard pour l’apprentissage approfondi](https://github.com/Azure/batch-shipyard/tree/master/recipes#deeplearning)

### <a name="mpi-providers"></a>Fournisseurs MPI

- [Microsoft MPI](/message-passing-interface/microsoft-mpi)

## <a name="remote-visualization"></a>Visualisation à distance

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
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Bureaux virtuels Linux avec Citrix</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Générez un environnement VDI pour les bureaux Linux à l’aide de Citrix sur Azure.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="performance-benchmarks"></a>Test d’évaluation des performances

- [Test d’évaluation des calculs](/azure/virtual-machines/windows/compute-benchmark-scores?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="customer-stories"></a>Témoignages client

Un certain nombre de clients ont connu beaucoup de succès grâce à l’utilisation d’Azure pour leurs charges de travail HPC.  Voici quelques-unes de ces études de cas clients :

- [ANEO](https://customers.microsoft.com/story/it-provider-finds-highly-scalable-cloud-based-hpc-redu)
- [AXA Global P&amp;C](https://customers.microsoft.com/story/axa-global-p-and-c)
- [Axioma](https://customers.microsoft.com/story/axioma-delivers-fintechs-first-born-in-the-cloud-multi-asset-class-enterprise-risk-solution)
- [d3View](https://customers.microsoft.com/story/big-data-solution-provider-adopts-new-cloud-gains-thou)
- [EFS](https://customers.microsoft.com/story/efs-professionalservices-azure)
- [Hymans Robertson](https://customers.microsoft.com/story/hymans-robertson)
- [MetLife](https://enterprise.microsoft.com/customer-story/industries/insurance/metlife/)
- [Microsoft Research](https://customers.microsoft.com/doclink/fast-lmm-and-windows-azure-put-genetics-research-on-fa)
- [Milliman](https://customers.microsoft.com/story/actuarial-firm-works-to-transform-insurance-industry-w)
- [Mitsubishi UFJ Securities International](https://customers.microsoft.com/story/powering-risk-compute-grids-in-the-cloud)
- [NeuroInitiative](https://customers.microsoft.com/story/neuroinitiative-health-provider-azure)
- [Schlumberger](https://azure.microsoft.com/blog/big-compute-for-large-engineering-simulations)
- [Towers Watson](https://customers.microsoft.com/story/insurance-tech-provider-delivers-disruptive-solutions)

## <a name="other-important-information"></a>Autres informations importantes

- Vérifiez que votre [quotas de processeurs virtuels](/azure/virtual-machines/linux/quotas?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) a été augmentée avant de tenter d’exécuter des charges de travail à grande échelle.

## <a name="next-steps"></a>Étapes suivantes

Pour connaître les dernières annonces, consultez :

- [Blog de l’équipe Microsoft HPC et Batch](http://blogs.technet.com/b/windowshpc/)
- Consultez le [blog Azure](https://azure.microsoft.com/blog/tag/hpc/).

### <a name="microsoft-batch-examples"></a>Exemples Microsoft Batch

Les tutoriels suivants vous fourniront des informations détaillées sur l’exécution d’applications sur Microsoft Batch

- [Commencer à développer avec Batch](/azure/batch/quick-run-dotnet?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [Utiliser les exemples de code Azure Batch](https://github.com/Azure/azure-batch-samples)
- [Utiliser des machines virtuelles de faible priorité avec Batch](/azure/batch/batch-low-pri-vms?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [Exécuter des charges de travail HPC en conteneur avec Batch Shipyard](https://github.com/Azure/batch-shipyard)
- [Exécuter des charges de travail R parallèles sur Batch](https://github.com/Azure/doAzureParallel)
- [Exécuter des travaux Spark à la demande sur Batch](https://github.com/Azure/aztk)
- [Utiliser des machines virtuelles nécessitant beaucoup de ressources système dans des pools Batch](/azure/batch/batch-pool-compute-intensive-sizes?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)