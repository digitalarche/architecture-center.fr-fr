---
title: Accélérer la modélisation basée sur des images numériques dans Azure
titleSuffix: Azure Example Scenarios
description: Accélérer la modélisation basée sur des images numériques dans Azure en utilisant Avere et Agisoft PhotoScan
author: adamboeglin
ms.date: 1/11/2019
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: cat-team, Linux, HPC
social_image_url: /azure/architecture/example-scenario/infrastructure/media/architecture-image-modeling.png
ms.openlocfilehash: 87b43347fb5f4baec0081a67c8b003dccd2fdf0d
ms.sourcegitcommit: 14226018a058e199523106199be9c07f6a3f8592
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/31/2019
ms.locfileid: "55483009"
---
# <a name="accelerate-digital-image-based-modeling-on-azure"></a>Accélérer la modélisation basée sur des images numériques dans Azure

Cet exemple de scénario fournit des conseils sur la conception et l’architecture aux organisations qui veulent effectuer une modélisation basée sur des images sur l’infrastructure IaaS Azure. Le scénario est conçu pour l’exécution de logiciels de photogrammétrie sur Machines virtuelles Azure en utilisant un stockage hautes performances qui accélère le temps de traitement. L’environnement peut faire l’objet d’un scale-up ou d’un scale-down en fonction des besoins et prend en charge plusieurs téraoctets de stockage sans sacrifier les performances.

## <a name="relevant-use-cases"></a>Cas d’usage appropriés

Les cas d’usage appropriés sont les suivants :

- Modélisation et mesure de bâtiments, ingénierie des structures et investigation scientifique des lieux d’accident.
- Création d’effets visuels pour les jeux sur ordinateur et les films.
- Utilisation d’images numériques pour générer indirectement des mesures d’objets de différentes échelles, comme dans la planification urbaine et d’autres applications.

## <a name="architecture"></a>Architecture

Cet exemple décrit l’utilisation du logiciel de photogrammétrie Agisoft PhotoScan avec le stockage Avere vFXT. PhotoScan a été choisi pour sa popularité dans les applications de système d’information géographique, pour la documentation de patrimoine culturel, pour le développement de jeux et pour la production d’effets visuels. Il convient pour la photogrammétrie de proximité et aérienne.

Les concepts de cet article s’appliquent aux charges de travail de computing haute performance (HPC) basées sur un planificateur et aux nœuds worker gérés comme infrastructure.  Pour cette charge de travail, Avere vFXT a été sélectionné pour ses performances supérieures lors de tests d’évaluation.  Cependant, le scénario découple le stockage du traitement : d’autres solutions de stockage peuvent donc être utilisées (consultez [alternatives](#alternatives) plus loin dans ce document).

Cette architecture comprend également des contrôleurs de domaine Active Directory pour contrôler l’accès aux ressources Azure et fournir une résolution de noms interne via le DNS (Domain Name System). Des ordinateurs Jump Box fournissent aux administrateurs un accès aux machines virtuelles Windows et Linux qui exécutent la solution.

![Diagramme d'architecture](./media/architecture-image-modeling.png)

1. Un utilisateur soumet un certain nombre d’images à PhotoScan.
2. Le planificateur PhotoScan s’exécute sur une machine virtuelle Windows qui sert de nœud principal et pilote le traitement des images de l’utilisateur.
3. PhotoScan recherche des points communs sur les photographies et construit la géométrie (maillage) en utilisant les nœuds de traitement PhotoScan qui s’exécutent sur des machines virtuelles avec des GPU (Graphics Processing Unit).
4. Avere vFXT fournit une solution de stockage haute performance sur Azure basée sur NFSv3 et constituée d’au moins quatre machines virtuelles.
5. PhotoScan effectue le rendu du modèle.

### <a name="components"></a>Composants

- [Agisoft PhotoScan](http://www.agisoft.com/) : Le planificateur PhotoScan s’exécute sur une machine virtuelle Windows Server 2016 et les nœuds de traitement utilisent cinq machines virtuelles avec des GPU qui exécutent CentOS Linux 7.5.
- [Avere vFXT](/azure/avere-vfxt/avere-vfxt-overview) est une solution de mise en cache de fichiers qui utilise le stockage d’objets et un stockage NAS traditionnel pour optimiser le stockage de jeux de données volumineux.  Il inclut :
  - Contrôleur Avere. Cette machine virtuelle exécute le script qui installe le cluster Avere vFXT et exécute Ubuntu 18.04 LTS. La machine virtuelle peut être utilisée ultérieurement pour ajouter ou supprimer des nœuds de cluster ainsi que pour détruire le cluster.
  - Cluster vFXT. Au moins trois machines virtuelles sont utilisées, une pour chacun des nœuds Avere vFXT basés sur Avere OS 5.0.2.1. Ces machines virtuelles forment le cluster vFXT, qui est attaché à Stockage Blob Azure.
- Les [contrôleurs de domaine Microsoft Active Directory](/windows/desktop/ad/active-directory-domain-services) permettent à l’hôte d’accéder aux ressources du domaine et fournissent la résolution de noms DNS. Avere vFXT ajoute un certain nombre d’enregistrements A : par exemple, chaque enregistrement A dans un cluster vFXT pointe vers l’adresse IP de chaque nœud Avere vFXT. Dans cette configuration, toutes les machines virtuelles utilisent le modèle de tourniquet (round robin) pour accéder aux exportations de vFXT.
- Les [autres machines virtuelles](/azure/virtual-machines/) servent d’ordinateurs Jump Box utilisées par l’administrateur pour accéder au nœud du planificateur et aux nœuds de traitement. L’ordinateur Jump Box Windows est obligatoire pour permettre à l’administrateur d’accéder au nœud principal via le protocole RDP. Le deuxième ordinateur Jump Box est facultatif et exécute Linux pour l’administration des nœuds worker.
- Les [groupes de sécurité réseau](/azure/virtual-network/manage-network-security-group) limitent l’accès à l’adresse IP publique et autorisent les ports 3389 et 22 pour l’accès aux machines virtuelles attachées au sous-réseau de l’ordinateur Jump Box.
- Le [peering de réseau virtuel](/azure/virtual-network/virtual-network-peering-overview) connecte un réseau virtuel PhotoScan à un réseau virtuel Avere.
- [Stockage Blob Azure](/azure/storage/blobs/storage-blobs-introduction) fonctionne avec Avere vFXT comme serveur de fichiers principal pour stocker les données validées à traiter. Avere vFXT identifie les données actives stockées dans Stockage Blob Azure et les place dans les disques SSD utilisés pour la mise en cache dans ses nœuds de calcul quand un travail PhotoScan s’exécute. Si des modifications sont apportées, les données sont validées de façon asynchrone dans le serveur de fichiers principal.
- [Azure Key Vault](/azure/key-vault/key-vault-overview) est utilisé pour stocker les mots de passe de l’administrateur et le code d’activation de PhotoScan.

### <a name="alternatives"></a>Autres solutions

- Pour tirer parti des services Azure dans la gestion d’un cluster HPC, utilisez des outils comme Azure CycleCloud ou Azure Batch, au lieu de gérer les ressources via des modèles ou des scripts.
- Déployez le système de fichiers virtuel parallèle BeeGFS comme stockage back-end sur Azure au lieu d’Avere vFXT. Utilisez le [modèle BeeGFS](https://github.com/paulomarquesc/beegfs-template) pour déployer cette solution de bout en bout sur Azure.
- Déployez la solution de stockage de votre choix, comme GlusterFS, Lustre ou les espaces de stockage direct de Windows. Pour cela, modifiez le [modèle PhotoScan](https://github.com/paulomarquesc/photoscan-template) pour qu’il utilise la solution de stockage souhaitée.
- Déployez les nœuds worker avec le système d’exploitation Windows au lieu de Linux, qui est l’option par défaut. Si vous choisissez des nœuds Windows, les options d’intégration du stockage ne sont pas exécutées par les modèles de déploiement. Vous devez intégrer manuellement l’environnement à une solution de stockage, ou personnaliser le modèle PhotoScan pour fournir cette automatisation, comme décrit dans le [dépôt](https://github.com/paulomarquesc/photoscan-template/blob/master/docs/AverePostDeploymentSteps.md).

## <a name="considerations"></a>Considérations

Ce scénario est conçu spécifiquement pour fournir un stockage haute performance pour une charge de travail HPC, qu’elle soit déployée sur Windows ou sur Linux. D’une façon générale, la configuration du stockage de la charge de travail HPC doit correspondre aux bonnes pratiques appropriées utilisées pour les déploiements locaux.

Les considérations relatives au déploiement dépendent des applications et des services utilisés, mais il faut prendre en compte quelques remarques :

- Lors de la création d’applications haute performance, utilisez Stockage Azure Premium et [optimisez la couche Application](/azure/virtual-machines/windows/premium-storage-performance). Optimisez le stockage pour les accès fréquents en utilisant le [niveau d’accès chaud](/azure/storage/blobs/storage-blob-storage-tiers) de Stockage Blob Azure.
- Utilisez une [option de réplication](/azure/storage/common/storage-redundancy) du stockage qui répond à vos besoins de disponibilité et de performances. Dans cet exemple, Avere vFXT est configuré par défaut pour la haute disponibilité, avec un stockage localement redondant (LRS). Pour l’équilibrage de charge, toutes les machines virtuelles utilisent le modèle de tourniquet (round robin) pour accéder aux exportations de vFXT.
- Si le stockage back-end est utilisé à la fois par des clients Windows et des clients Linux, utilisez des serveurs Samba pour prendre en charge les nœuds Windows. Une [version](https://github.com/paulomarquesc/beegfs-template) de cet exemple de scénario basé sur BeeGFS utilise Samba pour prendre en charge le nœud du planificateur de la charge de travail HPC (PhotoScan) s’exécutant sur Windows. Un équilibreur de charge est déployé pour agir comme remplacement intelligent pour le tourniquet (round robin) DNS.
- Exécutez les applications HPC en utilisant le type de machine virtuelle le mieux adapté à votre charge de travail [Windows](/azure/virtual-machines/windows/sizes-hpc) ou [Linux](/azure/virtual-machines/linux/sizes?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
- Pour isoler la charge de travail HPC des ressources de stockage, déployez chaque ressource dans son propre réseau virtuel, puis utilisez le [peering](/azure/virtual-network/virtual-network-peering-overview) de réseau virtuel pour connecter les deux réseaux. Le peering crée une connexion haut débit à faible latence entre les ressources dans différents réseaux virtuels et route le trafic à travers l’infrastructure principale Microsoft via des adresses IP privées uniquement.

### <a name="security"></a>Sécurité

Cet exemple se concentre sur le déploiement d’une solution de stockage haute performance pour une charge de travail HPC et n’est pas une solution de sécurité. Veillez à impliquer votre équipe de sécurité pour les éventuelles modifications.

Pour renforcer la sécurité, cet exemple d’infrastructure permet à toutes les machines virtuelles Windows d’être jointes au domaine et utilise Active Directory pour l’authentification centralisée. Il fournit également des services DNS personnalisés pour toutes les machines virtuelles. Pour aider à protéger l’environnement, ce modèle s’appuie sur des [groupes de sécurité réseau](/azure/virtual-network/security-overview). Les groupes de sécurité réseau offrent des filtres de trafic et des règles de sécurité de base.

Considérez les options suivantes pour améliorer la sécurité dans ce scénario :

- Utilisez des appliances de réseau virtuel, comme Fortinet, Checkpoint et Juniper.
- Appliquez le [contrôle d’accès en fonction du rôle](/azure/role-based-access-control/overview) aux groupes de ressources.
- Activez l’accès [JIT](/azure/security-center/security-center-just-in-time) aux machines virtuelles si les ordinateurs Jump Box sont accessibles via Internet.
- Utilisez [Azure Key Vault](/azure/key-vault/quick-create-portal) pour stocker les mots de passe utilisés par les comptes d’administrateur.

## <a name="pricing"></a>Tarifs

Le coût d’exécution de ce scénario peut varier considérablement en fonction de plusieurs facteurs.  Le nombre et la taille des machines virtuelles, la quantité de stockage nécessaire et le temps nécessaire pour effectuer un travail détermineront le coût.

L’exemple de profil de coûts suivant dans la [calculatrice de prix Azure](https://azure.com/e/42362ddfd2e245a28a8e78bc609c80f3) est basé sur une configuration classique pour Avere vFXT et PhotoScan :

- 1 machine virtuelle Ubuntu A1\_v2 pour exécuter le contrôleur Avere.
- 3 machines virtuelles Avere OS D16s\_v3 : une pour chacun des nœuds Avere vFXT qui forment le cluster vFXT.
- 5 machines virtuelles Linux NC24\_v2 pour fournir les GPU nécessaires aux nœuds de traitement PhotoScan.
- 1 machine virtuelle CentOS D8s\_v3 pour le nœud du planificateur PhotoScan.
- 1 machine virtuelle CentOS DS2\_v2 utilisée comme ordinateur Jump Box de l’administrateur.
- 2 machines virtuelles DS2\_v2 pour les contrôleurs de domaine Active Directory.
- Disques managés Premium.
- Stockage Blob v2 universel (GPv2) avec LRS et niveau d’accès chaud (seuls les comptes de stockage GPv2 exposent l’attribut Niveau d’accès).
- Réseau virtuel prenant en charge le transfert de 10 To de données.

Pour plus d’informations sur cette architecture, consultez le [livre électronique](https://azure.microsoft.com/en-us/resources/deploy-agisoft-photoscan-on-azure-with-azere-vfxt-for-azure-or-beegfs/). Pour voir quel serait le tarif de votre cas d’usage particulier, choisissez des tailles de machines virtuelles différentes dans la calculatrice de prix en fonction du déploiement que vous prévoyez.

## <a name="deployment"></a>Déploiement

Pour obtenir des instructions pas à pas sur le déploiement de cette architecture, notamment tous les prérequis de l’utilisation d’Avere FxT ou BeeGFS, téléchargez le livre électronique : [Deploy Agisoft PhotoScan on Azure With Avere vFXT for Azure or BeeGFS](https://azure.microsoft.com/en-us/resources/deploy-agisoft-photoscan-on-azure-with-azere-vfxt-for-azure-or-beegfs/).

## <a name="related-resources"></a>Ressources associées

Les ressources suivantes fournissent plus d’informations sur les composants utilisés dans ce scénario, ainsi que des approches alternatives pour le traitement par lots sur Azure.

- Vue d’ensemble d’[Avere vFXT pour Azure](/azure/avere-vfxt/avere-vfxt-overview)
- Page d’accueil d’[Agisoft PhotoScan](https://www.agisoft.com/)
- [Liste de contrôle des performances et de l’extensibilité du Stockage Microsoft Azure](/azure/storage/common/storage-performance-checklist)
- [Systèmes de fichiers virtuels parallèles sur Microsoft Azure : Tests de performances de Lustre, GlusterFS et BeeGFS](https://azure.microsoft.com/mediahandler/files/resourcefiles/parallel-virtual-file-systems-on-microsoft-azure/Parallel_Virtual_File_Systems_on_Microsoft_Azure.pdf) (PDF)
- Un exemple de scénario pour l’[ingénierie assistée par ordinateur (IAO) sur Azure](/azure/architecture/example-scenario/apps/hpc-saas)
- Page d’accueil de [HPC sur Azure](https://azure.microsoft.com/en-us/solutions/high-performance-computing/)
- Vue d’ensemble de [Big Compute : HPC &amp; Microsoft Batch](https://azure.microsoft.com/en-us/solutions/big-compute/)