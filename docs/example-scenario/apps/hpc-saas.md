---
title: Un service d’ingénierie assistée par ordinateur
titleSuffix: Azure Example Scenarios
description: Offrez une plateforme de software as a service (SaaS) pour l’ingénierie assistée par ordinateur (IAO) sur Azure.
author: alexbuckgit
ms.date: 08/22/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: HPC
social_image_url: /azure/architecture/example-scenario/apps/media/architecture-hpc-saas.png
ms.openlocfilehash: bd38bd0042fceeab6efe04d7b6d1477d17ada7f5
ms.sourcegitcommit: 3b15d65e7c35a19506e562c444343f8467b6a073
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/25/2019
ms.locfileid: "54908475"
---
# <a name="a-computer-aided-engineering-service-on-azure"></a>Un service d’ingénierie assistée par ordinateur sur Azure

Cet exemple de scénario illustre la livraison d’une plateforme software-as-a-service (SaaS) basée sur les fonctionnalités de calcul haute performance (HPC) d’Azure. Ce scénario repose sur une solution logicielle d’ingénierie. Toutefois, l’architecture est adaptée à d’autres secteurs nécessitant des ressources HPC comme le rendu d’images, la modélisation complexe et le calcul des risques financiers.

Cet exemple présente un fournisseur de logiciels d’ingénierie qui propose des applications d’ingénierie assistée par ordinateur (IAO) aux entreprises d’ingénierie et de fabrication. Les solutions IAO encouragent l’innovation, réduisent les durées de développement et diminuent les coûts tout au long de la durée de vie de la conception d’un produit. Ces solutions requièrent des ressources de calcul importantes et traitent souvent des volumes élevés de données. Les coûts élevés d’une appliance HPC locale ou des stations de travail haut de gamme placent souvent ces technologies hors de portée des étudiants, des entrepreneurs et des petites entreprises d’ingénierie.

L’entreprise souhaite développer le marché de ses applications en créant une plateforme SaaS s’appuyant sur les technologies HPC basées sur le cloud. Ses clients doivent être en mesure de payer pour des ressources de calcul variant en fonction des besoins et pour accéder à l’immense puissance de calcul qui serait autrement hors de prix.

Les objectifs de l’entreprise sont les suivants :

- tirer parti des capacités HPC d’Azure pour accélérer la conception du produit et le processus de test ;
- utiliser les dernières innovations matérielles pour exécuter des simulations complexes, tout en réduisant les coûts des simulations plus simples ;
- obtenir un rendu et une visualisation réalistes dans un navigateur web sans nécessiter une station de travail d’ingénierie haut de gamme.

## <a name="relevant-use-cases"></a>Cas d’usage appropriés

Les autres cas d’usage appropriés sont les suivants :

- Recherche sur le génome
- Simulation météorologique
- Applications de chimie par modélisation numérique

## <a name="architecture"></a>Architecture

![Architecture d’une solution SaaS activant les fonctionnalités HPC][architecture]

- Les utilisateurs peuvent accéder à des machines virtuelles de série NV via un navigateur à l’aide d’une connexion RDP HTML5 utilisant le [service Apache Guacamole](https://guacamole.apache.org/). Ces instances de machine virtuelle fournissent de puissantes unités GPU destinées aux tâches de rendu et de collaboration. Les utilisateurs peuvent modifier leurs conceptions et afficher leurs résultats sans avoir besoin d’accéder aux appareils informatiques mobiles ni aux portables haut de gamme. Le planificateur prépare des machines virtuelles supplémentaires basées sur une méthode heuristique définie par l’utilisateur.
- À partir d’une session de CAO sur un Bureau, les utilisateurs peuvent envoyer des charges de travail pour exécution sur les nœuds de cluster HPC disponibles. Ces charges de travail effectuent des tâches comme l’analyse des contraintes ou les calculs de mécanique des fluides, éliminant ainsi la nécessité d’avoir recours à des clusters de calcul locaux dédiés. Ces nœuds de cluster peuvent être configurés sur une mise à l’échelle automatique basée sur la longueur de la file d’attente ou la profondeur de la charge en fonction de la demande d’utilisateur active en ressources de calcul.
- Azure Kubernetes Service (AKS) permet d’héberger les ressources web disponibles pour les utilisateurs finaux.

### <a name="components"></a>Composants

- Les [machines virtuelles de série H](/azure/virtual-machines/linux/sizes-hpc) sont utilisées pour exécuter des simulations nécessitant beaucoup de ressources système telles que la modélisation moléculaire et la mécanique des fluides numérique. La solution tire également parti de technologies telles que la connectivité RDMA et la mise en réseau InfiniBand.
- Les [machines virtuelles de série NV](/azure/virtual-machines/windows/sizes-gpu) fournissent aux ingénieurs des fonctionnalités de station de travail haut de gamme à partir d’un navigateur web standard. Ces machines virtuelles disposent de processeurs GPU NVIDIA Tesla M60 qui prennent en charge un rendu avancé et peuvent exécuter des charges de travail précises individuelles.
- Les [machines virtuelles à usage général](/azure/virtual-machines/linux/sizes-general) exécutant CentOS gèrent des charges de travail plus traditionnelles, par exemple des applications web.
- [Application Gateway](/azure/application-gateway/overview) équilibre la charge des requêtes entrant sur les serveurs web.
- [Azure Kubernetes Service (AKS)](/azure/aks/intro-kubernetes) permet d’exécuter des charges de travail évolutives à moindre coût pour les simulations qui ne nécessitent pas les fonctionnalités haut de gamme des machines virtuelles équipées de fonctionnalités HPC ou de processeurs GPU.
- [Altair PBS Works Suite](https://www.pbsworks.com/PBSProduct.aspx?n=PBS-Works-Suite&c=Overview-and-Capabilities) orchestre le flux de travail HPC, s’assurant ainsi que suffisamment d’instances de machine virtuelle sont disponibles pour gérer la charge actuelle. Ce service libère également des machines virtuelles lorsque la demande est plus faible pour réduire les coûts.
- Le [stockage d’objets blob](/azure/storage/blobs/storage-blobs-introduction) stocke les fichiers qui prennent en charge les travaux planifiés.

### <a name="alternatives"></a>Autres solutions

- L’outil [Azure CycleCloud](/azure/cyclecloud/overview) simplifie la création, la gestion, le fonctionnement et l’optimisation des clusters HPC. Il offre des fonctionnalités de gouvernance et de stratégie avancées. CycleCloud prend en charge le planificateur de travaux ou la pile logicielle de votre choix.
- [HPC Pack](/azure/virtual-machines/windows/hpcpack-cluster-options) peut créer et gérer un cluster HPC Azure pour les charges de travail basées sur Windows Server. HPC Pack n’est pas une option pour les charges de travail basées sur Linux.
- Le service [Azure Automation State Configuration](/azure/automation/automation-dsc-overview) fournit une approche de type infrastructure en tant que code pour définir les machines virtuelles et les logiciels à déployer. Les machines virtuelles peuvent être déployées dans le cadre d’un groupe de machines virtuelles identiques, avec des règles de mise à l’échelle automatique pour les nœuds de calcul basées sur le nombre de travaux soumis à la file d’attente des travaux. Lorsqu’une nouvelle machine virtuelle est nécessaire, elle est approvisionnée à l’aide de la dernière image corrigée provenant de la galerie d’images Azure, puis le logiciel requis est installé et configuré via un script de configuration DSC PowerShell.
- [Azure Functions](/azure/azure-functions/functions-overview)

## <a name="considerations"></a>Considérations

- Si l’utilisation d’une approche de type infrastructure en tant que code est un excellent moyen de gérer les définitions de build des machines virtuelles, l’approvisionnement d’une nouvelle machine virtuelle à l’aide d’un script peut prendre du temps. Cette solution constitue un juste milieu en utilisant le script DSC pour créer régulièrement une image de référence, qui peut ensuite être utilisée pour approvisionner une nouvelle machine virtuelle plus rapidement qu’en générant complètement une machine virtuelle à la demande à l’aide de DSC. Azure DevOps Services ou d’autres outils CI/CD peuvent actualiser périodiquement les images de référence à l’aide de scripts DSC.
- L’équilibrage des coûts de la solution globale avec la disponibilité rapide des ressources de calcul est élément clé à prendre en compte. L’approvisionnement d’un pool d’instances de machine virtuelle de série N et le placement de celles-ci dans un état désalloué permettent de réduire les coûts d’exploitation. Lorsqu’une machine virtuelle supplémentaire est nécessaire, la réallocation d’une instance existante implique la mise sous tension de la machine virtuelle sur un hôte différent, mais le temps de détection du bus PCI requis par le système d’exploitation pour identifier et installer des pilotes pour le processeur GPU est éliminé, car une machine virtuelle déprovisionnée, puis réapprovisionnée conserve le même bus PCI pour le processeur GPU dès le redémarrage.
- L’architecture d’origine reposait entièrement sur les machines virtuelles Azure pour exécuter des simulations. Pour réduire les coûts des charges de travail ne nécessitant pas toutes les fonctionnalités d’une machine virtuelle, ces charges de travail ont été mises en conteneur et déployées vers Azure Kubernetes Service (AKS).
- Le personnel de l’entreprise disposait de compétences dans les technologies open source. Il peut tirer parti de ces compétences en s’appuyant sur des technologies telles que Linux et Kubernetes.

## <a name="pricing"></a>Tarifs

Pour vous aider à explorer le coût d’exécution de ce scénario, nombre des services requis sont préconfigurés dans un [exemple de calculateur de coûts][calculator]. Les coûts de votre solution dépendent du nombre et de l’échelle des services nécessaires pour répondre à vos besoins.

Les considérations ci-après ont une incidence notable sur les coûts de cette solution :

- Les coûts de machine virtuelle Azure augmentent de manière linéaire à mesure que des instances supplémentaires sont approvisionnées. Les machines virtuelles qui sont libérées entraînent uniquement des coûts de stockage, et non des coûts de calcul. Ces machines libérées peuvent être réaffectées par la suite en cas d’augmentation de la demande.
- Les coûts d’Azure Kubernetes Service sont basés sur le type de machine virtuelle choisi pour prendre en charge la charge de travail. Ces coûts augmentent de façon linéaire en fonction du nombre de machines virtuelles contenues dans le cluster.

## <a name="next-steps"></a>Étapes suivantes

- Lisez le [témoignage client d’Altair][source-document]. Cet exemple de scénario est basé sur une version de son architecture.
- Passez en revue d’autres [solutions Big Compute](https://azure.microsoft.com/solutions/big-compute) disponibles dans Azure.

<!-- links -->
[architecture]: ./media/architecture-hpc-saas.png
[source-document]: https://customers.microsoft.com/story/altair-manufacturing-azure
[calculator]: https://azure.com/e/3cb9ccdc893f41ffbcdb00c328178ccf
