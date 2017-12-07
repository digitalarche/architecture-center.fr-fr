---
title: "Exécuter des machines virtuelles à charge équilibrée dans Azure à des fins de scalabilité et de disponibilité"
description: "Découvrez comment exécuter plusieurs machines virtuelles Linux dans Azure à des fins de scalabilité et de disponibilité."
author: telmosampaio
ms.date: 11/16/2017
pnp.series.title: Linux VM workloads
pnp.series.next: n-tier
pnp.series.prev: single-vm
ms.openlocfilehash: b1b3c94524d50d05c90b46d26cab54fea8c8061a
ms.sourcegitcommit: 115db7ee008a0b1f2b0be50a26471050742ddb04
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2017
---
# <a name="run-load-balanced-vms-for-scalability-and-availability"></a>Exécuter des machines virtuelles à charge équilibrée à des fins de scalabilité et de disponibilité

Cette architecture de référence présente un ensemble de pratiques éprouvées pour exécuter plusieurs machines virtuelles Linux dans un groupe identique derrière un équilibreur de charge, afin d’améliorer la disponibilité et l’extensibilité. Vous pouvez utiliser cette architecture pour toute charge de travail sans état, telle qu’un serveur web. Elle représente une base pour le déploiement d’applications multiniveaux. [**Déployez cette solution**.](#deploy-the-solution)

![[0]][0]

*Téléchargez un [fichier Visio][visio-download] de cette architecture.*

## <a name="architecture"></a>Architecture

Cette architecture repose sur [l’architecture de référence de machine virtuelle unique][single-vm]. Ces recommandations s’appliquent également à cette architecture.

Dans cette architecture, une charge de travail est répartie entre plusieurs instances de machine virtuelle. Il existe une seule adresse IP publique, et le trafic Internet est réparti entre les machines virtuelles à l’aide d’un équilibreur de charge. Vous pouvez utiliser cette architecture pour une application à un seul niveau, telle qu’une application web sans état.

Elle comporte les composants suivants :

* **Groupe de ressources.** Les [groupes de ressources][resource-manager-overview] servent à regrouper des ressources afin de pouvoir les gérer en fonction de la durée de vie, du propriétaire ou d’autres critères.
* **Réseau virtuel (VNet) et sous-réseau.** Chaque machine virtuelle Azure est déployée dans un réseau virtuel qui peut être segmenté en plusieurs sous-réseaux.
* **Équilibreur de charge Azure**. [L’équilibreur de charge][load-balancer] répartit les requêtes Internet entrantes entre les instances de machine virtuelle. 
* **Adresse IP publique**. Une adresse IP publique est nécessaire pour que l’équilibreur de charge puisse recevoir le trafic Internet.
* **Groupe de machines virtuelles identiques**. Un [groupe de machines virtuelles identiques][vm-scaleset] est un ensemble de machines virtuelles identiques utilisé pour héberger une charge de travail. Les groupes identiques permettent d’augmenter ou de réduire le nombre de machines virtuelles manuellement ou automatiquement en fonction de règles prédéfinies.
* **Groupe à haute disponibilité**. Le [groupe à haute disponibilité][availability-set] contient les machines virtuelles, ce qui leur permet d’être éligibles pour un [niveau contrat de service (SLA)][vm-sla] supérieur. Pour appliquer le contrat SLA supérieur, vous devez ajouter au moins deux machines virtuelles dans le groupe à haute disponibilité. Les groupes à haute disponibilité sont implicites dans les groupes identiques. Si vous créez des machines virtuelles en dehors d’un groupe identique, vous devez créer le groupe à haute disponibilité indépendamment.
* **Disques managés**. Les disques managés Azure gèrent les fichiers de disque dur virtuel (VHD) des disques de machine virtuelle. 
* **Stockage**. Créez un compte de stockage Azure pour stocker les journaux de diagnostic des machines virtuelles.

## <a name="recommendations"></a>Recommandations

Vos exigences peuvent ne pas s’aligner complètement avec l’architecture décrite ici. Utilisez ces recommandations comme point de départ. 

### <a name="availability-and-scalability-recommendations"></a>Recommandations en matière de disponibilité et de scalabilité

Une option pour la disponibilité et la scalabilité consiste à utiliser un [groupe de machines virtuelles identiques][vmss]. Les groupes de machines virtuelles identiques vous aident à déployer et à gérer un ensemble de machines virtuelles identiques. Les groupes identiques prennent en charge la mise à l’échelle automatique basée sur des métriques de performances. À mesure que la charge sur les machines virtuelles augmente, des machines virtuelles supplémentaires sont ajoutées automatiquement à l’équilibreur de charge. Les groupes identiques sont parfaits si vous devez rapidement faire monter en puissance des machines virtuelles, ou si vous avez besoin d’une mise à l’échelle automatique.

Par défaut, les groupes identiques utilisent le « surprovisionnement », ce qui signifie qu’ils configurent initialement plus de machines virtuelles que vous ne le demandez, puis il supprime les machines virtuelles supplémentaires. Cela permet d’améliorer le taux de réussite global lors du provisionnement des machines virtuelles. Si vous n’utilisez pas de [disques gérés](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-managed-disks), nous vous recommandons de ne pas utiliser plus de 20 machines virtuelles par compte de stockage quand le sur-approvisionnement est activé, et plus de 40 machines virtuelles quand le sur-approvisionnement est désactivé.

Il existe deux façons de configurer des machines virtuelles déployées dans un groupe identique :

- Utiliser des extensions pour configurer la machine virtuelle après son provisionnement. Avec cette approche, le démarrage des nouvelles instances de machine virtuelle peut être plus long que pour une machine virtuelle sans extension.

- Déployer un [disque managé](/azure/storage/storage-managed-disks-overview) avec une image de disque personnalisée. Cette option peut être plus rapide à déployer. Toutefois, elle vous oblige à tenir l’image à jour.

Pour plus d’informations, consultez [Considérations relatives à la conception des groupes de machines virtuelles identiques][vmss-design].

> [!TIP]
> Quand vous utilisez une solution de mise à l’échelle automatique, testez-la bien à l’avance avec des charges de travail de niveau production.

Si vous n’utilisez pas de groupe identique, utilisez au moins un groupe à haute disponibilité. Créez au moins deux machines virtuelles dans le groupe à haute disponibilité, afin de prendre en charge le [contrat SLA de disponibilité pour les machines virtuelles Azure][vm-sla]. L’équilibreur de charge Azure exige également que les machines virtuelles avec équilibrage de charge appartiennent au même groupe à haute disponibilité.

Chaque abonnement Azure a des limites par défaut, notamment une quantité maximale de machines virtuelles par région. Vous pouvez augmenter la limite en créant une demande de support. Pour plus d’informations, consultez [Abonnement Azure et limites, quotas et contraintes de service][subscription-limits].

### <a name="network-recommendations"></a>Recommandations pour le réseau

Déployez les machines virtuelles sur le même sous-réseau. N’exposez pas les machines virtuelles directement à Internet. Attribuez plutôt à chaque machine virtuelle une adresse IP privée. Les clients se connectent à l’aide de l’adresse IP publique de l’équilibreur de charge.

Si vous avez besoin de vous connecter aux machines virtuelles derrière l’équilibreur de charge, ajoutez une machine virtuelle comme serveur de rebond (également appelé hôte bastion) avec une adresse IP publique à laquelle vous pouvez vous connecter. Ensuite, connectez-vous aux machines virtuelles derrière l’équilibreur de charge à partir du serveur de rebond. Autrement, vous pouvez configurer les règles de traduction d’adresse réseau (NAT) de trafic entrant SSH de l’équilibreur de charge. Toutefois, le serveur de rebond est une meilleure solution quand vous hébergez des charges de travail multiniveaux ou plusieurs charges de travail.

### <a name="load-balancer-recommendations"></a>Recommandations relatives à l’équilibreur de charge

Ajoutez toutes les machines virtuelles du groupe à haute disponibilité dans le pool d’adresses backend de l’équilibreur de charge.

Définissez des règles d’équilibreur de charge pour diriger le trafic réseau vers les machines virtuelles. Par exemple, pour activer le trafic HTTP, créez une règle qui mappe le port 80 de la configuration frontend au port 80 dans le pool d’adresses backend. Quand un client envoie une requête HTTP au port 80, l’équilibreur de charge sélectionne une adresse IP backend en utilisant un [algorithme de hachage][load-balancer-hashing] qui inclut l’adresse IP source. Ainsi, les requêtes des clients sont réparties entre toutes les machines virtuelles.

Pour acheminer le trafic vers une machine virtuelle spécifique, utilisez des règles NAT. Par exemple, pour autoriser les communications RDP sur les machines virtuelles, créez une règle NAT distincte pour chaque machine virtuelle. Chaque règle doit mapper un numéro de port distinct au port 3389, le port par défaut pour le protocole RDP. Par exemple, utilisez le port 50001 pour « VM1 », le port 50002 pour « VM2 », et ainsi de suite. Affectez les règles NAT aux cartes réseau sur les machines virtuelles.

### <a name="storage-account-recommendations"></a>Recommandations relatives aux comptes de stockage

Nous vous recommandons d’utiliser des [disques managés](/azure/storage/storage-managed-disks-overview) avec le [stockage premium][premium]. Les disques managés ne nécessitent pas de compte de stockage. Il vous suffit de spécifier leur taille et leur type, puis de les déployer en tant que ressource hautement disponible.

Si vous utilisez des disques non gérés, créez des comptes de stockage Azure distincts pour chaque machine virtuelle pour stocker les disques durs virtuels (VHD), afin d’éviter d’atteindre les [limites d’opérations d’entrée/sortie par seconde][vm-disk-limits] des comptes de stockage.

Créez un compte de stockage pour les journaux de diagnostic. Ce compte de stockage peut être partagé par toutes les machines virtuelles. Il peut s’agir d’un compte de stockage non géré utilisant des disques standards.

## <a name="availability-considerations"></a>Considérations relatives à la disponibilité

Le groupe à haute disponibilité rend votre application plus résiliente aux événements de maintenance planifiés et non planifiés.

* La *maintenance planifiée* se produit quand Microsoft met à jour la plateforme sous-jacente, ce qui provoque parfois le redémarrage des machines virtuelles. Azure garantit que les machines virtuelles d’un groupe à haute disponibilité ne sont pas toutes redémarrées en même temps. Au moins l’une d’elles continue à s’exécuter pendant que les autres redémarrent.
* La *maintenance non planifiée* se produit en cas de défaillance matérielle. Azure garantit que les machines virtuelles d’un groupe à haute disponibilité sont provisionnées parmi plusieurs racks de serveurs. Cela aide à réduire l’impact des défaillances matérielles, pannes réseau, coupures d’alimentation, et ainsi de suite.

Pour plus d’informations, consultez [Gestion de la disponibilité des machines virtuelles][availability-set]. La vidéo suivante offre également une bonne vue d’ensemble des groupes à haute disponibilité : [How Do I Configure an Availability Set to Scale VMs (Guide pratique pour configurer un groupe à haute disponibilité pour la mise à l’échelle des machines virtuelles)][availability-set-ch9].

> [!WARNING]
> N’oubliez pas de configurer le groupe à haute disponibilité quand vous provisionnez la machine virtuelle. Actuellement, il n’existe aucun moyen d’ajouter une machine virtuelle Resource Manager à un groupe à haute disponibilité après le provisionnement de la machine virtuelle.

L’équilibreur de charge utilise des [sondes d’intégrité][health-probes] pour surveiller la disponibilité des instances de machine virtuelle. Si une sonde ne peut pas atteindre une instance dans le délai imparti, l’équilibreur de charge cesse d’envoyer le trafic vers cette machine virtuelle. Toutefois, il continue à sonder, et si la machine virtuelle redevient disponible il reprend l’envoi du trafic vers cette machine virtuelle.

Voici quelques recommandations concernant les sondes d’intégrité d’équilibreur de charge :

* Les sondes peuvent tester le protocole TCP ou HTTP. Si vos machines virtuelles exécutent un serveur HTTP, créez une sonde HTTP. Sinon, créez une sonde TCP.
* Pour une sonde HTTP, spécifiez le chemin d’un point de terminaison HTTP. La sonde vérifie la présence d’une réponse HTTP 200 à partir de ce chemin. Il peut s’agir du chemin racine (« / ») ou d’un point de terminaison de surveillance de l’intégrité qui implémente une logique personnalisée afin de vérifier l’intégrité de l’application. Le point de terminaison doit autoriser les requêtes HTTP anonymes.
* La sonde est envoyée à partir d’une [adresse IP connue][health-probe-ip], 168.63.129.16. Veillez à ne bloquer le trafic vers ou à partir de cette adresse IP dans aucune stratégie de pare-feu ou règle de groupe de sécurité réseau.
* Utilisez des [journaux de sonde d’intégrité][health-probe-log] pour afficher l’état des sondes d’intégrité. Activez la journalisation dans le portail Azure pour chaque équilibreur de charge. Les journaux sont écrits dans le Stockage Blob Azure. Ils indiquent combien de machines virtuelles du côté backend ne reçoivent pas de trafic réseau à cause d’échecs de réponses de sonde.

## <a name="manageability-considerations"></a>Considérations relatives à la facilité de gestion

Avec plusieurs machines virtuelles, il est important d’automatiser les processus afin qu’ils soient fiables et reproductibles. Vous pouvez utiliser [Azure Automation][azure-automation] pour automatiser le déploiement, les mises à jour correctives du système d’exploitation et d’autres tâches.

## <a name="security-considerations"></a>Considérations relatives à la sécurité

Les réseaux virtuels sont une limite d’isolation du trafic dans Azure. Les machines virtuelles d’un réseau virtuel ne peuvent pas communiquer directement avec celles d’un autre réseau virtuel. Les machines virtuelles situées sur un même réseau virtuel peuvent communiquer, sauf si vous créez des [groupes de sécurité réseau][nsg] pour limiter le trafic. Pour plus d’informations, consultez [Services cloud Microsoft et sécurité réseau][network-security].

Pour le trafic Internet entrant, les règles d’équilibreur de charge définissent le trafic qui peut atteindre le backend. Toutefois, les règles d’équilibreur de charge ne prennent pas en charge les listes de sécurité IP. Par conséquent, si vous souhaitez ajouter certaines adresses IP publiques à une liste verte, ajoutez un groupe de sécurité réseau au sous-réseau.

## <a name="deploy-the-solution"></a>Déployer la solution

Un déploiement pour cette architecture est disponible sur [GitHub][github-folder]. Il déploie les éléments suivants :

  * Un réseau virtuel avec un seul sous-réseau nommé **web** contenant les machines virtuelles.
  * Un groupe de machines virtuelles identiques qui contient des machines virtuelles exécutant la version la plus récente d’Ubuntu 16.04.3 LTS. La mise à l’échelle automatique est activée.
  * Un équilibreur de charge placé devant le groupe de machines virtuelles identiques.
  * Un groupe de sécurité réseau avec des règles de trafic entrant qui autorisent le trafic HTTP vers le groupe de machines virtuelles identiques.

### <a name="prerequisites"></a>Composants requis

Avant de pouvoir déployer l’architecture de référence sur votre propre abonnement, vous devez effectuer les étapes suivantes.

1. Clonez, dupliquez ou téléchargez le fichier zip pour le dépôt GitHub des [architectures de référence AzureCAT][ref-arch-repo].

2. Vérifiez qu’Azure CLI 2.0 est installé sur votre ordinateur. Pour des instructions d’installation de l’interface de ligne de commande, consultez [Installer Azure CLI 2.0][azure-cli-2].

3. Installez le package npm des [modules Azure][azbb].

4. À partir d’une invite de commandes, d’une invite bash ou de l’invite de commandes PowerShell, connectez-vous à votre compte Azure à l’aide de l’une des commandes ci-dessous et suivez les invites.

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a>Déployer la solution à l’aide d’azbb

Pour déployer l’exemple de charge de travail de machine virtuelle unique, effectuez les étapes suivantes :

1. Accédez au dossier `virtual-machines\multi-vm\parameters\linux` pour le dépôt que vous avez téléchargé à l’étape des prérequis ci-dessus.

2. Ouvrez le fichier `multi-vm-v2.json` et entrez un nom d’utilisateur et une clé SSH entre les guillemets, comme illustré ci-dessous, puis enregistrez le fichier.

  ```bash
  "adminUsername": "",
  "sshPublicKey": "",
  ```

3. Exécutez `azbb` pour déployer les machines virtuelles comme illustré ci-dessous.

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p multi-vm-v2.json --deploy
  ```

Pour plus d’informations sur le déploiement de cet exemple d’architecture de référence, visitez notre [dépôt GitHub][git].

<!-- links -->

[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[availability-set-ch9]: https://channel9.msdn.com/Series/Microsoft-Azure-Fundamentals-Virtual-Machines/08
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-automation]: /azure/automation/automation-intro
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-cli-2]: /azure/install-azure-cli?view=azure-cli-latest
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[health-probe-log]: /azure/load-balancer/load-balancer-monitor-log
[health-probes]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[health-probe-ip]: /azure/virtual-network/virtual-networks-nsg#special-rules
[load-balancer]: /azure/load-balancer/load-balancer-get-started-internet-arm-cli
[load-balancer-hashing]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[naming-conventions]: ../../best-practices/naming-conventions.md
[network-security]: /azure/best-practices-network-security
[nsg]: /azure/virtual-network/virtual-networks-nsg
[premium]: /azure/storage/common/storage-premium-storage
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview 
[runbook-gallery]: /azure/automation/automation-runbook-gallery#runbooks-in-runbook-gallery
[single-vm]: single-vm.md
[subscription-limits]: /azure/azure-subscription-service-limits
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-scaleset]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[vm-sizes]: https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_2/
[vmss]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[vmss-design]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview
[vmss-quickstart]: https://azure.microsoft.com/documentation/templates/?term=scale+set
[0]: ./images/multi-vm-diagram.png "Architecture d’une solution à plusieurs machines virtuelles dans Azure comprenant un groupe à haute disponibilité avec deux machines virtuelles et un équilibreur de charge"
