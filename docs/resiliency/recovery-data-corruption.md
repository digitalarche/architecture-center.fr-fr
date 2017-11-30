---
title: "Récupérer suite à une altération de données ou à une suppression accidentelle"
description: "Article dédié à la description de la récupération suite à une altération de données ou à une suppression accidentelle de données et à la conception d’applications résilientes, hautement disponibles et tolérantes aux pannes, ainsi qu’à la planification de la récupération d’urgence"
author: adamglick
ms.date: 08/18/2016
ms.openlocfilehash: b75c774f85c42f64472167897f08a7302ab50a3f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2017
---
[!INCLUDE [header](../_includes/header.md)]
# <a name="azure-resiliency-technical-guidance-recovery-from-data-corruption-or-accidental-deletion"></a>Guide technique de la résilience Azure : récupération suite à une altération de données ou à une suppression accidentelle
Un plan solide de continuité d’activité englobe une portion dédiée à la récupération suite aux altérations ou aux suppressions de données. Vous trouverez ci-après des informations sur la récupération suite à une altération ou à une suppression accidentelle de vos données en raison d’erreurs applicatives ou d’erreurs de l’opérateur.

## <a name="virtual-machines"></a>Machines virtuelles
Pour protéger votre système de machines virtuelles Azure, parfois appelé machines virtuelles IaaS (infrastructure-as-a-service), contre les erreurs applicatives ou les suppressions accidentelles, utilisez [Sauvegarde Azure](https://azure.microsoft.com/services/backup/). Le service Sauvegarde Azure prend en charge la création de sauvegardes cohérentes sur plusieurs disques de machine virtuelle. En outre, l’archivage de sauvegarde peut être répliqué entre les régions, afin d’assurer la prise en charge de la récupération suite à une perte de région.

## <a name="storage"></a>Stockage
Notez que si Stockage Azure assure la résilience des données via des réplicas automatisés, cela n’empêche aucunement votre code d’application (ou les développeurs/utilisateurs) d’altérer des données lors d’une suppression accidentelle ou involontaire, une mise à jour, etc. La conservation de l’intégrité des données sous la menace d’erreurs des utilisateurs et des applications nécessite des techniques plus avancées, comme la copie des données vers un emplacement de stockage secondaire avec un journal d’audit. Les développeurs peuvent tirer parti de la [fonctionnalité d’instantané d’objet blob](https://msdn.microsoft.com/library/azure/ee691971.aspx), qui peut créer des instantanés en lecture seule d’objets blob. Elle peut être utilisée comme solution de base dédiée à l’intégrité des données des objets blob de stockage Azure.

### <a name="blob-and-table-storage-backup"></a>Sauvegarde de stockage d’objets blob et de tables
Les objets blob et les tables sont hautement durables, mais ils représentent toujours l’état actuel des données. La récupération de modifications ou de suppressions non souhaitées des données peut nécessiter leur restauration vers un état précédent. Pour ce faire, vous pouvez profiter des fonctionnalités fournies par Azure pour stocker et conserver des copies ponctuelles.

Pour les objets blob Azure, vous pouvez effectuer des sauvegardes ponctuelles à l’aide de la [fonctionnalité d’instantané blob](https://msdn.microsoft.com/library/ee691971.aspx). Pour chaque instantané, vous êtes facturé pour le stockage requis pour stocker les différences identifiées sur l’objet blob depuis le dernier état d’instantané. Les instantanés dépendant de l’existence de l’objet blob d’origine sur lequel ils sont basés, nous vous recommandons de copier les données sur un autre objet blob voire sur un autre compte de stockage. Cela garantit la protection des données de sauvegarde contre tout risque de suppression accidentelle. Pour les tables Azure, vous pouvez générer des copies ponctuelles vers une autre table ou vers des objets blob Azure. Pour consulter des recommandations détaillées et des exemples de sauvegardes de tables et d’objets blob au niveau de l’application, consultez la page :

* [Protecting Your Tables Against Application Errors](https://blogs.msdn.microsoft.com/windowsazurestorage/2010/05/03/protecting-your-tables-against-application-errors/)
* [Protecting Your Blobs Against Application Errors](https://blogs.msdn.microsoft.com/windowsazurestorage/2010/04/29/protecting-your-blobs-against-application-errors/)

## <a name="database"></a>Base de données
Plusieurs options de [continuité d’activité](/azure/sql-database/sql-database-business-continuity/) (sauvegarde, restauration) sont disponibles pour Azure SQL Database. Les bases de données peuvent être copiées avec la fonctionnalité de [copie de base de données](/azure/sql-database/sql-database-copy/) ou avec [l’exportation](/azure/sql-database/sql-database-export/) et [l’importation](https://msdn.microsoft.com/library/hh710052.aspx) d’un fichier .bacpac SQL Server. La fonctionnalité de copie de base de données offre des résultats cohérents d’un point de vue transactionnel, ce qui n’est pas le cas du fichier bacpac (par le biais du service d’importation/d’exportation). Ces deux options sont exécutées en tant que services basés sur file d’attente au sein du centre de données. Aucun contrat de niveau de service de durée d’exécution n’est actuellement fourni.

> [!NOTE]
> Les options de copie de base de données et d’importation/exportation placent un niveau de charge considérable sur la base de données source. Elles peuvent déclencher la contention des ressources ou la limitation des événements.
> 
> 

### <a name="sql-database-backup"></a>Sauvegarde de base de données SQL
Les sauvegardes dans le temps pour Microsoft Azure SQL Database sont accomplies via la [copie de votre base de données Azure SQL](/azure/sql-database/sql-database-copy/). Vous pouvez utiliser cette commande pour créer une copie cohérente sur le plan transactionnel d’une base de données sur le même serveur logique de base de données ou sur un serveur différent. Dans les deux cas, la copie de base de données est pleinement fonctionnelle et entièrement indépendante de la base de données source. Chaque copie créée représente une option de récupération dans le temps. Pour récupérer intégralement l’état de base de données, attribuez à la nouvelle base de données le nom de la base de données source. Sinon, vous pouvez récupérer un sous-ensemble spécifique des données de la nouvelle base. Pour ce faire, exécutez des requêtes Transact-SQL. Pour plus d’informations sur SQL Database, consultez [Vue d’ensemble de la continuité de l’activité avec Azure SQL Database](/azure/sql-database/sql-database-business-continuity/).

### <a name="sql-server-on-virtual-machines-backup"></a>Sauvegarde SQL Server sur les machines virtuelles
Quand SQL Server est utilisé avec l’infrastructure Azure sous la forme de machines virtuelles de service (souvent appelées modèles IaaS ou machines virtuelles IaaS), vous disposez de deux options : les sauvegardes traditionnelles et la copie des journaux de transaction. Le recours aux sauvegardes traditionnelles vous permet de restaurer les données à un point spécifique dans le temps, mais le processus de récupération est lent. Avec la restauration de sauvegardes traditionnelles, vous devez démarrer par une sauvegarde initiale complète, puis appliquer toutes les sauvegardes effectuées ultérieurement. La seconde option consiste en la configuration d’une session de copie des journaux de transaction pour reporter la restauration des sauvegardes des journaux (par exemple, de deux heures). Vous disposez ainsi d’une fenêtre permettant de récupérer des erreurs identifiées sur la sauvegarde primaire.

## <a name="other-azure-platform-services"></a>Autres services de plateforme Azure
Certains services de plateforme Azure stockent des informations dans un compte de stockage contrôlé par l’utilisateur ou Azure SQL Database. Si le compte ou la ressource de stockage est supprimé ou corrompu, de graves erreurs peuvent être signalées sur le service. Dans ces cas, il est important de conserver des sauvegardes qui peuvent être utilisées pour recréer ces ressources en cas de suppression ou d’altération.

Pour les Sites Web Azure et Azure Mobile Services, vous devez sauvegarder et mettre à jour les bases de données associées. Pour Azure Media Service et Machines virtuelles, vous devez gérer le compte Stockage Azure associé et l’ensemble des ressources de ce compte. Par exemple, pour Machines virtuelles, vous devez sauvegarder et gérer les disques de machines virtuelles dans le stockage blob Azure.

## <a name="checklists-for-data-corruption-or-accidental-deletion"></a>Listes de contrôle pour l’altération ou la suppression accidentelle des données
## <a name="virtual-machines-checklist"></a>Liste de contrôle de Machines virtuelles
1. Consulter la section Machines virtuelles de ce document.
2. Sauvegardez et gérer les disques de machines virtuelles avec Sauvegarde Azure (ou votre propre système de sauvegarde à l’aide du stockage Blob Azure et d’instantanés VHD).

## <a name="storage-checklist"></a>Liste de contrôle du stockage
1. Consultez la section Stockage de ce document.
2. Sauvegardez régulièrement les ressources de stockage critiques.
3. Envisagez d’utiliser la fonctionnalité d’instantané pour les objets blob.

## <a name="database-checklist"></a>Liste de contrôle de la base de données
1. Consultez la section Base de données de ce document.
2. Créez des sauvegardes dans le temps à l’aide de la commande de copie de base de données.

## <a name="sql-server-on-virtual-machines-backup-checklist"></a>Liste de contrôle de la sauvegarde SQL Server sur les machines virtuelles
1. Consultez la section Sauvegarde SQL Server sur les machines virtuelles de ce document.
2. Utilisez des techniques traditionnelles de sauvegarde et de restauration.
3. Créez une session reportée de copie des journaux de transaction.

## <a name="web-apps-checklist"></a>Liste de contrôle Web Apps
1. Sauvegardez et gérez la base de données associée, le cas échéant.

## <a name="media-services-checklist"></a>Liste de contrôle Media Services
1. Sauvegardez et gérez les ressources de stockage associées.

## <a name="more-information"></a>Plus d’informations
Pour plus d’informations sur les fonctionnalités de sauvegarde et de restauration d’Azure, consultez la rubrique [Scénarios de stockage, sauvegarde et récupération](https://azure.microsoft.com/documentation/scenarios/storage-backup-recovery/).


