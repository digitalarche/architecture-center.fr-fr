---
title: Récupérer suite à une altération de données ou à une suppression accidentelle
description: Article dédié à la description de la récupération suite à une corruption de données ou à une suppression accidentelle de données et à la conception d’applications résilientes, hautement disponibles et tolérantes aux pannes, ainsi qu’à la planification de la récupération d’urgence
author: MikeWasson
ms.date: 01/10/2018
ms.openlocfilehash: b0716de39fe69d607b9a63e51356d28bbcdbfeae
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/16/2018
ms.locfileid: "31012424"
---
# <a name="recover-from-data-corruption-or-accidental-deletion"></a>Récupérer suite à une altération de données ou à une suppression accidentelle 

Un plan solide de continuité d’activité englobe une portion dédiée à la récupération suite aux corruptions ou aux suppressions de données. Vous trouverez ci-après des informations sur la récupération suite à une corruption ou à une suppression accidentelle de vos données en raison d’erreurs applicatives ou d’erreurs de l’opérateur.

## <a name="virtual-machines"></a>Virtual Machines

Pour protéger les machines virtuelles Azure contre les erreurs applicatives ou les suppressions accidentelles, utilisez [Sauvegarde Azure](/azure/backup/). Azure Backup prend en charge la création de sauvegardes cohérentes sur plusieurs disques de machine virtuelle. En outre, l’archivage de sauvegarde peut être répliqué entre les régions, afin d’assurer la prise en charge de la récupération suite à une perte de région.

## <a name="storage"></a>Stockage

Le stockage Azure fournit la résilience des données via des réplicas automatisés. Toutefois, cela n’empêche pas le code d’application ou les utilisateurs d’altérer des données de façon volontaire ou accidentelle. La conservation de l’intégrité des données sous la menace d’erreurs des utilisateurs et des applications nécessite des techniques plus avancées, comme la copie des données vers un emplacement de stockage secondaire avec un journal d’audit. 

- **Objets blob de blocs**. Créez un instantané à un point dans le temps de chaque objet blob de blocs. Pour plus d’informations, consultez [Création d’un instantané d’objet blob](/rest/api/storageservices/creating-a-snapshot-of-a-blob). Pour chaque instantané, vous êtes facturé pour le stockage requis pour stocker les différences identifiées sur l’objet blob depuis le dernier état d’instantané. Les instantanés dépendant de l’existence de l’objet blob d’origine sur lequel ils sont basés, nous vous recommandons de copier les données sur un autre objet blob voire sur un autre compte de stockage. Cela garantit la protection des données de sauvegarde contre tout risque de suppression accidentelle. Vous pouvez utiliser [AzCopy](/azure/storage/common/storage-use-azcopy) ou [Azure PowerShell](/azure/storage/common/storage-powershell-guide-full) pour copier les objets blob vers un autre compte de stockage.

- **Fichiers**. Utilisez les [instantanés de partage](/azure/storage/files/storage-snapshots-files), AzCopy ou PowerShell pour copier vos fichiers vers un autre compte de stockage.

- **Tables**. Utilisez AzCopy pour exporter les données de table vers un autre compte de stockage dans une autre région.

## <a name="database"></a>Base de données

### <a name="azure-sql-database"></a>Base de données SQL Azure 

SQL Database effectue automatiquement une combinaison de sauvegardes de bases de données complètes (toutes les semaines), de sauvegardes de bases de données différentielles (toutes les heures), et de sauvegardes de journaux de transactions (toutes les cinq à dix minutes) pour protéger votre entreprise contre la perte de données. Utilisez la restauration dans le temps pour restaurer une base de données à une heure antérieure. Pour plus d'informations, consultez les pages suivantes :

- [Récupérer une base de données SQL Azure à l’aide des sauvegardes automatisées d’une base de données](/azure/sql-database/sql-database-recovery-using-backups)

- [Vue d’ensemble de la continuité de l’activité avec la base de données Azure SQL](/azure/sql-database/sql-database-business-continuity)

### <a name="sql-server-on-vms"></a>SQL Server sur une machine virtuelle

Pour SQL Server exécuté sur une machine virtuelle, il existe deux options : les sauvegardes traditionnelles et la copie des journaux de transaction. Les sauvegardes traditionnelles vous permettent de restaurer les données à un point spécifique dans le temps, mais le processus de récupération est lent. Avec la restauration de sauvegardes traditionnelles, vous devez démarrer par une sauvegarde initiale complète, puis appliquer toutes les sauvegardes effectuées ultérieurement. La seconde option consiste en la configuration d’une session de copie des journaux de transaction pour reporter la restauration des sauvegardes des journaux (par exemple, de deux heures). Vous disposez ainsi d’une fenêtre permettant de récupérer des erreurs identifiées sur la sauvegarde primaire.

### <a name="azure-cosmos-db"></a>Azure Cosmos DB

Azure Cosmos DB effectue des sauvegardes automatiques à des intervalles réguliers. Les sauvegardes sont stockées séparément dans un autre service de stockage, et ces sauvegardes sont répliquées globalement pour garantir la résilience contre les sinistres régionaux. En cas de suppression accidentelle de votre base de données ou collection, vous pouvez émettre un ticket de support ou appeler le support technique Azure pour restaurer les données à partir de la dernière sauvegarde automatique. Pour plus d’informations, consultez [Sauvegarde et restauration en ligne automatiques avec Azure Cosmos DB](/azure/cosmos-db/online-backup-and-restore).

### <a name="azure-database-for-mysql-azure-database-for-postresql"></a>Azure Database pour MySQL, Azure Database pour PostgreSQL

Lorsque vous utilisez Azure Database pour MySQL ou Azure Database pour PostgreSQL, le service de base de données crée automatiquement une sauvegarde du service toutes les cinq minutes. À l’aide de cette fonctionnalité de sauvegarde automatique, vous pouvez restaurer le serveur et toutes ses bases de données dans un nouveau serveur à un moment antérieur. Pour plus d'informations, consultez les pages suivantes :

- [Sauvegarde et restauration d’un serveur Azure Database pour MySQL à l’aide du portail Azure](/azure/mysql/howto-restore-server-portal)

- [Comment sauvegarder et restaurer un serveur Azure Database pour PostgreSQL à l’aide du portail Azure](/azure/postgresql/howto-restore-server-portal)

