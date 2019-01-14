---
title: Récupération de l’environnement local vers Azure
titleSuffix: Azure Resiliency Technical Guidance
description: Présentation et conception de systèmes de récupération de l’infrastructure locale vers Azure.
author: adamglick
ms.date: 08/18/2016
ms.custom: seojan19
ms.openlocfilehash: 5e4c4ea4eede5f11e787b9957b8de47736645672
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/08/2019
ms.locfileid: "54111357"
---
[!INCLUDE [header](../_includes/header.md)]

# <a name="azure-resiliency-technical-guidance-recovery-from-on-premises-to-azure"></a>Guide technique de la résilience Azure : Récupération de l’environnement local vers Azure

Azure offre un ensemble complet de services conçus pour étendre un centre de données local vers Azure afin de faciliter la récupération d’urgence et de garantir une haute disponibilité :

- **Réseau** : L’utilisation d’un réseau privé virtuel vous permet d’étendre de manière sécurisée votre réseau local vers le cloud.
- **Calcul** : Les clients qui utilisent Hyper-V en local peuvent effectuer un « lift-and-shift » des machines virtuelles existantes vers Azure.
- **Stockage** : StorSimple étend votre système de fichiers au Stockage Azure. Le service Azure Backup assure la sauvegarde des fichiers et des bases de données SQL dans Azure Storage.
- **Réplication de la base de données** : Avec les groupes de disponibilité SQL Server 2014 (ou version ultérieure), vous pouvez implémenter une haute disponibilité et une reprise d’activité après sinistre pour vos données locales.

## <a name="networking"></a>Mise en réseau

Vous pouvez utiliser le réseau virtuel Azure pour créer une section isolée logiquement dans Azure et la connecter en toute sécurité à votre centre de données local ou à un seul ordinateur client à l’aide d’une connexion IPsec. Grâce au réseau virtuel, vous pouvez tirer parti de l’infrastructure évolutive et à la demande d’Azure tout en assurant la connectivité aux données et aux applications locales, y compris aux systèmes s’exécutant sur Windows Server, les ordinateurs centraux et les systèmes UNIX. Pour plus d’informations, consultez la [documentation réseau d’Azure](/azure/virtual-network/virtual-networks-overview/) .

## <a name="compute"></a>Calcul

Si vous utilisez Hyper-V en local, vous pouvez « élever et déplacer » des machines virtuelles existantes vers Azure et vers des fournisseurs de services exécutant Windows Server 2012 (ou version ultérieure), sans avoir à apporter de modifications à la machine virtuelle ni à la convertir dans un autre format. Pour plus d’informations, consultez l’article [À propos des disques et des VHD pour les machines virtuelles Azure](/azure/virtual-machines/virtual-machines-linux-about-disks-vhds/?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

## <a name="azure-site-recovery"></a>Azure Site Recovery

Si vous souhaitez bénéficier d’une récupération d’urgence en tant que service (DRaaS), vous pouvez utiliser [Azure Site Recovery](https://azure.microsoft.com/services/site-recovery/). Azure Site Recovery offre une protection complète pour les serveurs VMware, Hyper-V et physiques. Avec Azure Site Recovery, vous pouvez utiliser un autre serveur local ou Azure comme site de récupération. Pour plus d’informations sur Azure Site Recovery, consultez la [Documentation d’Azure Site Recovery](https://azure.microsoft.com/documentation/services/site-recovery/).

## <a name="storage"></a>Stockage

Pour utiliser Azure comme site de sauvegarde pour les données locales, vous disposez de plusieurs options.

### <a name="storsimple"></a>StorSimple

StorSimple intègre le stockage cloud en toute sécurité et en toute transparence pour les applications locales. Il propose également une appliance unique conçue pour garantir un stockage en local et dans le cloud hiérarchisé et performant, tout en offrant des fonctions d’archivage dynamique, de protection des données dans le cloud et de récupération d’urgence. Pour plus d’informations, consultez la [page produit StorSimple](https://azure.microsoft.com/services/storsimple/).

### <a name="azure-backup"></a>Sauvegarde Azure

Azure Backup exécute des sauvegardes dans le cloud à l’aide des outils de sauvegarde classiques de Windows Server 2012 (ou version ultérieure), de Windows Server 2012 Essentials (ou version ultérieure) et de System Center 2012 Data Protection Manager (ou version ultérieure). Ces outils établissent pour la gestion des sauvegarde un workflow totalement indépendant de l’emplacement de stockage des sauvegardes, qu’il s’agisse d’un disque local ou d’Azure Storage. Une fois les données sauvegardées dans le cloud, les utilisateurs autorisés peuvent facilement restaurer les sauvegardes sur un serveur.

Avec les sauvegardes incrémentielles, seules les modifications des fichiers sont transférées dans le cloud. Cela permet d’utiliser efficacement le stockage, de réduire la consommation de bande passante et d’assurer une récupération à tout moment des différentes versions des données. Vous pouvez également choisir d’utiliser des fonctionnalités supplémentaires, telles que les stratégies de rétention des données, la compression des données et la limitation du transfert de données. L’utilisation d’Azure comme emplacement de sauvegarde a l’avantage d’offrir des sauvegardes qui sont automatiquement exécutées « hors site ». Cette approche vous évite d’avoir à sécuriser et protéger vos supports de sauvegarde sur site.

Pour plus d’informations, consultez [Qu’est-ce qu’Azure Backup ?](/azure/backup/backup-introduction-to-azure-backup/) et [Configuration d’une sauvegarde Azure pour des données DPM](https://technet.microsoft.com/library/jj728752.aspx).

## <a name="database"></a>Base de données

Vous pouvez disposer d’une solution de récupération d’urgence pour vos bases de données SQL Server dans un environnement informatique hybride utilisant des groupes de disponibilité AlwaysOn, la mise en miroir de bases de données, la copie des journaux de transaction et la sauvegarde et la restauration avec le stockage d’objets blob Azure. Toutes ces solutions utilisent une implémentation SQL Server exécutée sur Azure Virtual Machines.

Les groupes de disponibilité AlwaysOn peuvent être utilisés dans un environnement informatique hybride comportant des réplicas de base de données à la fois en local et dans le cloud. Cette situation est présentée dans le diagramme suivant.

![Groupes de disponibilité SQL Server AlwaysOn dans une architecture de cloud hybride](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-3.png)

La mise en miroir de base de données peut également s’étendre à la fois à des serveurs locaux et au cloud dans une installation basée sur un certificat. Le schéma suivant illustre ce concept.

![Mise en miroir de la base de données SQL Server dans une architecture de cloud hybride](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-4.png)

L’envoi de journaux peut être utilisé pour synchroniser une base de données locale avec une base de données SQL Server dans une machine virtuelle Azure.

![Envoi de journaux SQL Server dans une architecture de cloud hybride](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-5.png)

Vous pouvez enfin sauvegarder une base de données locale directement dans Azure Blob Storage.

![Sauvegarde de SQL Server sur Azure Blob Storage dans une architecture de cloud hybride](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-6.png)

Pour plus d’informations, consultez [Haute disponibilité et récupération d’urgence pour SQL Server sur des machines virtuelles Azure](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/) et [Sauvegarde et restauration de SQL Server dans les machines virtuelles Azure](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-backup-recovery/).

## <a name="checklists-for-on-premises-recovery-in-microsoft-azure"></a>Listes de vérification pour la récupération des données locales dans Microsoft Azure

<!-- markdownlint-disable MD024 -->

### <a name="networking"></a>Mise en réseau

1. Consultez la section Mise en réseau de ce document.
2. Utilisez un réseau virtuel pour connecter en toute sécurité votre environnement local au cloud.

### <a name="compute"></a>Calcul

1. Consultez la section Calcul de ce document.
2. Déplacez des machines virtuelles entre Hyper-V et Azure.

### <a name="storage"></a>Stockage

1. Consultez la section Stockage de ce document.
2. Utilisez les services StorSimple pour tirer parti du stockage cloud.
3. Utilisez le service Azure Backup.

### <a name="database"></a>Base de données

1. Consultez la section Base de données de ce document.
2. Envisagez d’utiliser comme sauvegarde une instance SQL Server exécutée sur des machines virtuelles Azure.
3. Configurez les groupes de disponibilité AlwaysOn.
4. Configurez la mise en miroir de base de données basée sur un certificat.
5. Utilisez l’envoi de journaux.
6. Sauvegardez les bases de données locales dans Azure Blob Storage.

<!-- markdownlint-enable MD024 -->
