---
title: Exécution de simulations CFD
titleSuffix: Azure Example Scenarios
description: Exécutez des simulations de diagramme de flux cumulé (CFD) sur Azure.
author: mikewarr
ms.date: 09/20/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: fasttrack
social_image_url: /azure/architecture/example-scenario/infrastructure/media/architecture-hpc-cfd.png
ms.openlocfilehash: 6972b701a608351104c23459bc2c38029a5c8d3d
ms.sourcegitcommit: 3b15d65e7c35a19506e562c444343f8467b6a073
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/25/2019
ms.locfileid: "54908418"
---
# <a name="running-computational-fluid-dynamics-cfd-simulations-on-azure"></a>Exécution des simulations de diagramme de flux cumulé (CFD) sur Azure

Les simulations de mécanique des fluides numérique (CFD) nécessitent un temps de calcul considérable ainsi que du matériel spécialisé. À mesure que le recours aux clusters s’intensifie, les temps de simulation et l’utilisation totale des grilles augmentent, entraînant ainsi des problèmes de capacité de réserve et des temps d’attente excessifs. L’ajout de matériel physique peut se révéler coûteux et ne cadre pas toujours avec les pics d’utilisation et les creux d’activité auxquels sont confrontées les entreprises. Azure vous permet de surmonter la plupart de ces difficultés sans engager de dépenses en capital.

Azure vous fournit le matériel dont vous avez besoin pour exécuter vos travaux CFD sur des machines virtuelles avec capacités GPU et UC. Les tailles de machine virtuelle prenant en charge la fonctionnalité RDMA (Accès direct à la mémoire à distance) présentent une mise en réseau basée sur InfiniBand FDR qui autorise une communication MPI (Message Passing Interface, passage de messages) à faible latence. En combinant ces fonctionnalités avec Avere vFXT, un système de fichiers d’entreprise en cluster, les clients sont en mesure de garantir un débit maximal pour les opérations de lecture dans Azure.

Afin de simplifier la création, la gestion et l’optimisation des clusters HPC (calcul haute performance), vous pouvez utiliser Azure CycleCloud pour approvisionner les clusters et orchestrer les données dans les scénarios hybrides et cloud. En supervisant les travaux en attente, CycleCloud lance automatiquement les calculs à la demande, et vous ne payez que ce que vous utilisez, en vous connectant au planificateur de charge de travail de votre choix.

## <a name="relevant-use-cases"></a>Cas d’usage appropriés

Voici plusieurs autres secteurs d’activité utilisant le CFD :

- Aéronautique
- Industrie automobile
- Chauffage, ventilation et climatisation (HVAC) de bâtiments
- Hydrocarbures
- Sciences de la vie

## <a name="architecture"></a>Architecture

![Diagramme de l’architecture][architecture]

Ce diagramme présente une vue d’ensemble générale d’une conception hybride traditionnelle assurant la supervision des travaux des nœuds à la demande dans Azure :

1. Connexion au cluster Azure CycleCloud pour configurer le cluster.
2. Configuration et création du nœud principal du cluster, à l’aide de machines prenant en charge RDMA pour MPI.
3. Ajout et configuration du nœud principal local.
4. Si les ressources sont insuffisantes, Azure CycleCloud augmente (ou réduit) les ressources de calcul dans Azure. Il est possible de définir une limite prédéterminée pour empêcher une surutilisation.
5. Allocation de tâches aux nœuds d’exécution.
6. Mise en cache des données dans Azure à partir du serveur NFS local.
7. Lecture des données d’Avere vFXT pour le cache Azure.
8. Transfert des informations de travail et de tâche vers le serveur Azure CycleCloud.

### <a name="components"></a>Composants

- [Azure CycleCloud][cyclecloud] est un outil de création, de gestion, d’exploitation et d’optimisation de clusters HPC et Big Compute dans Azure.
- [Avere vFXT sur Azure][avere] fournit un système de fichiers d’entreprise en cluster conçu pour le cloud.
- [Machines virtuelles Azure][vms] permet de créer un ensemble statique d’instances de calcul.
- [Virtual Machine Scale Sets][vmss] fournit un groupe de machines virtuelles identiques pouvant faire l’objet d’une montée ou d’une descente en puissance par Azure CycleCloud.
- Les [comptes Stockage Azure](/azure/storage/common/storage-introduction) sont utilisés pour la synchronisation et la rétention des données.
- [Réseau virtuel](/azure/virtual-network/virtual-networks-overview) permet à de nombreux types de ressources Azure, comme Machines virtuelles Azure, de communiquer en toute sécurité entre elles, avec Internet et avec les réseaux locaux.

### <a name="alternatives"></a>Autres solutions

Les clients peuvent également utiliser Azure CycleCloud pour créer une grille entièrement dans Azure. Dans cette configuration, le serveur Azure CycleCloud est exécuté au sein de votre abonnement Azure.

Dans le cadre d’une approche moderne ne nécessitant pas la gestion d’un planificateur de charge de travail, [Azure Batch][batch] peut se révéler utile. Azure Batch peut exécuter efficacement des applications de calcul haute performance (HPC) en parallèle et à grande échelle dans le cloud. Azure Batch vous permet de définir les ressources de calcul Azure pour exécuter vos applications en parallèle ou à grande échelle sans avoir à configurer ou gérer manuellement votre infrastructure. Azure Batch planifie les tâches nécessitant beaucoup de ressources système et ajoute ou supprime des ressources de calcul de manière dynamique en fonction de vos besoins.

### <a name="scalability-and-security"></a>Extensibilité et sécurité

La mise à l’échelle des nœuds d’exécution sur Azure CycleCloud peut être effectuée manuellement ou automatiquement. Pour plus d’informations, consultez l’article [CycleCloud Autoscaling][cycle-scale] (Mise à l’échelle automatique CycleCloud).

Pour obtenir des conseils d’ordre général sur la conception de solutions sécurisées, consultez la [documentation sur la sécurité Azure][security].

## <a name="deploy-the-scenario"></a>Déployez le scénario

### <a name="prerequisites"></a>Prérequis

Avant de déployer le modèle Resource Manager, procédez comme suit :

1. Créez un [principal de service][cycle-svcprin] pour la récupération des valeurs d’ID d’application, de nom d’affichage, de nom, de mot de passe et de locataire.
2. Générez une [paire de clés SSH][cycle-ssh] pour vous connecter en toute sécurité au serveur CycleCloud.

    <!-- markdownlint-disable MD033 -->

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FCycleCloudCommunity%2Fcyclecloud_arm%2Fmaster%2Fazuredeploy.json" target="_blank">
        <img src="https://azuredeploy.net/deploybutton.png"/>
    </a>

    <!-- markdownlint-enable MD033 -->

3. [Connectez-vous au serveur CycleCloud][cycle-login] pour configurer et créer un nouveau cluster.
4. [Créez un cluster][cycle-create].

Avere Cache est une solution facultative pouvant augmenter considérablement le débit de lecture des données de travail d’application. Avere vFXT pour Azure résout le problème de l’exécution de ces applications HPC d’entreprise dans le cloud tout en tirant profit des données stockées localement ou dans le service Stockage Blob Azure.

Pour les organisations qui planifient l’utilisation d’une infrastructure hybride combinant stockage local et cloud computing, les applications HPC peuvent s’intégrer à Azure en utilisant les données stockées dans des périphériques de stockage NAS et démarrer des processeurs virtuels en fonction des besoins. Le jeu de données n’est jamais déplacé intégralement vers le cloud. Les octets requis sont temporairement mis en cache à l’aide d’un cluster Avere pendant le traitement.

Pour installer et configurer une installation Avere vFXT, suivez les instructions du [guide d’installation et de configuration d’Avere][avere].

## <a name="pricing"></a>Tarifs

Les coûts d’exécution d’une implémentation HPC avec un serveur CycleCloud dépendent d’un certain nombre de facteurs. Par exemple, CycleCloud est facturé en fonction du temps de calcul utilisé, lorsque le serveur maître et le serveur CycleCloud sont alloués et exécutés en continu. Les coûts d’exécution des nœuds d’exécution dépendent de la durée pendant laquelle ces nœuds sont opérationnels, ainsi que de la taille utilisée. Les frais de stockage et de mise en réseau Azure standard s’appliquent également.

Ce scénario indique comment exécuter des applications CFD dans Azure. Par conséquent, les machines virtuelles nécessitent la fonctionnalité RDMA, qui est uniquement disponible sur des tailles de machine virtuelle spécifiques. Vous trouverez ci-après des exemples de coûts induits par l’utilisation d’un groupe identique alloué en continu huit heures par jour pendant un mois, avec une sortie de données de 1 To. La tarification du serveur Azure CycleCloud et de l’installation d’Avere vFXT pour Azure est également incluse :

- Région : Europe Nord
- Serveur Azure CycleCloud : 1 x Standard D3 (4 UC, 14 Go de mémoire, HDD Standard 32 Go)
- Serveur maître Azure CycleCloud : 1 x Standard D12 (4 UC, 28 Go de mémoire, HDD Standard 32 Go)
- Tableau de nœuds Azure CycleCloud : 10 x Standard H16r (16 UC, 112 Go de mémoire)
- Avere vFXT sur un cluster Azure : 3 x D16s v3 (système d’exploitation avec 200 Go, 1 disque SSD Premium de 1 To)
- Sortie de données : 1 To

Examinez cette [estimation de prix][pricing] pour le matériel répertorié ci-dessus.

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez déployé l’exemple, apprenez-en davantage sur [Azure CycleCloud][cyclecloud].

## <a name="related-resources"></a>Ressources associées

- [Instances prenant en charge RDMA][rdma]
- [Personnalisation d’une machine virtuelle d’instance RDMA][rdma-custom]

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
