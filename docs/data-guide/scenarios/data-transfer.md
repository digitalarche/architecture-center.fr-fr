---
title: Choisir une technologie de transfert de données
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: bb0732b0f771a4c9e1a4e565875576c08484490a
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/14/2018
---
# <a name="transferring-data-to-and-from-azure"></a>Transférer des données vers et à partir d’Azure

Il existe plusieurs manières de transférer des données vers et à partir d’Azure, en fonction des besoins.

## <a name="physical-transfer"></a>Transfert physique

Il est intéressant d’utiliser du matériel physique pour transférer des données vers Azure si :

- Votre réseau est lent ou peu fiable.
- Il est coûteux d’ajouter de la bande passante réseau.
- Les directives organisationnelles ou les stratégies de sécurité n’autorisent pas les connexions sortantes en cas de traitement de données sensibles. 

Si votre préoccupation principale est le temps que prendra le transfert de données, il peut être utile d’effectuer un test pour vérifier si le transfert réseau est réellement plus lent que le transport physique.

Il existe deux moyens de transporter physiquement les données vers Azure :
- **Azure Import/Export**. Le service [Azure Import/Export](/azure/storage/common/storage-import-export-service) permet de transférer en toute sécurité de gros volumes de données vers le Stockage Blob Azure ou Azure Files en expédiant des disques durs ou des SSD SATA vers un centre de données Azure. Vous pouvez également utiliser ce service pour transférer des données du Stockage Azure vers des disques durs et vous les faire expédier pour les charger sur place.

- **Azure Data Box**. [Azure Data Box](https://azure.microsoft.com/services/storage/databox/) est une appliance fournie par Microsoft qui fonctionne à peu près comme le service Azure Import/Export. Microsoft vous envoie une appliance de transfert propriétaire, sécurisée et inviolable, et gère la logistique de bout en bout, que vous pouvez suivre sur le portail. L’un des avantages du service Azure Data Box est sa facilité d’utilisation. Vous n’avez pas besoin d’acheter plusieurs disques durs, de les préparer et de transférer des fichiers dessus. Azure Data Box est pris en charge par de nombreux partenaires Azure de premier plan ; il est donc d’autant plus facile de tirer parti en toute transparence du transport hors connexion de leurs produits vers le cloud. 

## <a name="command-line-tools-and-apis"></a>API et outils en ligne de commande

Choisissez ces solutions si vous souhaitez transférer les données par script et par programme.

- **Azure CLI**. [Azure CLI](/azure/hdinsight/hdinsight-upload-data#commandline) est un outil multiplateforme permettant de gérer les services Azure et de charger des données sur le Stockage Azure. 

- **AzCopy**. Utilisez AzCopy dans une interface de ligne de commande [Windows](/azure/storage/common/storage-use-azcopy?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) ou [Linux](/azure/storage/common/storage-use-azcopy-linux?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) pour copier facilement des données vers et à partir du Stockage Blob, Fichier et Table Azure avec des performances optimales. Il prend en charge la concurrence et le parallélisme, ainsi que la possibilité de reprendre les opérations de copie après une interruption. Il est également plus rapide que la plupart des autres solutions. Pour un accès par programme, la [bibliothèque Mouvement de données du Stockage Microsoft Azure](/azure/storage/common/storage-use-data-movement-library) est l’infrastructure de base d’AzCopy. Elle est proposée sous forme de bibliothèque .NET Core. 

- **PowerShell**. La [cmdlet PowerShell `Start-AzureStorageBlobCopy`](/powershell/module/azure.storage/start-azurestorageblobcopy?view=azurermps-5.0.0) est une possibilité pour les administrateurs de Windows qui sont habitués à PowerShell.  

- **AdlCopy**. [AdlCopy](/azure/data-lake-store/data-lake-store-copy-data-azure-storage-blob) permet de copier des données d’Azure Storage Blob vers Data Lake Store. Il peut également servir à copier des données entre deux comptes Azure Data Lake Store. En revanche, il ne permet pas de copier des données de Data Lake Store vers Storage Blob.

- **Distcp**. Si vous disposez d’un cluster HDInsight ayant accès à Data Lake Store, vous pouvez utiliser des outils de l’écosystème Hadoop, comme [Distcp](/azure/data-lake-store/data-lake-store-copy-data-wasb-distcp), pour copier des données vers et à partir d’un espace de stockage en cluster HDInsight (WASB) dans un compte Data Lake Store.

- **Sqoop**. [Sqoop](/azure/hdinsight/hadoop/hdinsight-use-sqoop) est un projet Apache qui fait partie de l’écosystème Hadoop. Il est préinstallé sur tous les clusters HDInsight. Il permet le transfert de données entre un cluster HDInsight et des bases de données relationnelles comme SQL, Oracle, MySQL, etc. Sqoop est une collection d’outils connexes, notamment d’importation et d’exportation. Il fonctionne avec des clusters HDInsight, à l’aide soit d’Azure Storage Blob, soit du stockage attaché Data Lake Store.

- **PolyBase**. [PolyBase](/sql/relational-databases/polybase/get-started-with-polybase) est une technologie qui accède aux données extérieures à la base de données avec le langage T-SQL. Dans SQL Server 2016, elle permet d’exécuter des requêtes sur des données externes dans Hadoop ou d’importer/exporter des données à partir du Stockage Blob Azure. Dans Azure SQL Data Warehouse, vous pouvez importer/exporter des données à partir du Stockage Blob Azure et d’Azure Data Lake Store. Actuellement, PolyBase est le moyen le plus rapide d’importer des données dans SQL Data Warehouse.

- **Ligne de commande Hadoop**. Si vous avez des données qui résident sur le nœud principal d’un cluster HDInsight, vous pouvez utiliser la commande `hadoop -copyFromLocal` pour les copier vers le stockage attaché de votre cluster, par exemple, Azure Storage Blob ou Azure Data Lake Store. Pour pouvoir utiliser la commande Hadoop, vous devez d'abord vous connecter au nœud principal. Vous pourrez alors charger un fichier dans le stockage.

## <a name="graphical-interface"></a>Interface graphique

Envisagez les solutions suivantes si vous transférez uniquement quelques fichiers ou quelques objets de données et que vous n’avez pas besoin d’automatiser le processus.

- **Explorateur Stockage Azure**. [L’Explorateur Stockage Azure](https://azure.microsoft.com/features/storage-explorer/) est un outil multiplateforme qui vous permet de gérer le contenu de vos comptes de stockage Azure. Avec lui, vous pouvez charger, télécharger et gérer des objets blob, des fichiers, des files d’attente, des tables et des entités Azure Cosmos DB. Utilisez-le avec le Stockage Blob pour gérer des objets blob et des dossiers, ainsi que charger et télécharger des objets blob entre votre système de fichiers local et le Stockage Blob, ou entre deux comptes de stockage.

- **Portail Azure**. Le Stockage Blob et Data Lake Store fournissent tous deux une interface web permettant d’explorer les fichiers et d’en charger de nouveaux un par un. C’est une bonne solution si vous ne souhaitez pas installer d’outils ni lancer des commandes pour explorer rapidement vos fichiers, ou simplement pour en charger de nouveaux en petite quantité.

## <a name="data-pipeline"></a>Pipeline de données

**Azure Data Factory**. [Azure Data Factory](/azure/data-factory/) est un service géré idéal pour transférer régulièrement des fichiers entre différents services Azure, locaux ou une combinaison des deux. Il permet de créer et de planifier des workflows pilotés par les données (nommés pipelines) qui ingèrent des données provenant de différents magasins de données. Il peut traiter et transformer les données à l’aide de services de calcul tels que Azure HDInsight Hadoop, Spark, Azure Data Lake Analytics et Azure Machine Learning. Créez des workflows pilotés par les données pour [orchestrer](../technology-choices/pipeline-orchestration-data-movement.md) et automatiser le déplacement et la transformation des données.

## <a name="key-selection-criteria"></a>Critères de sélection principaux

Dans les scénarios de transfert de données, choisissez le système adapté à vos besoins en répondant à ces questions :

- Avez-vous besoin de transférer de très grandes quantités de données, ce qui prendrait trop longtemps, ne serait pas assez fiable ou coûterait trop cher avec une connexion Internet ? Si oui, optez pour le transfert physique.

- Préférez-vous créer un script réutilisable pour vos tâches de transfert de données ? Si oui, sélectionnez l’une des solutions en ligne de commande, ou bien Azure Data Factory.

- Avez-vous besoin de transférer de très grandes quantités de données sur une connexion réseau ? Si oui, sélectionnez une solution optimisée pour le Big Data.

- Avez-vous besoin de transférer des données vers ou à partir d’une base de données relationnelle ? Si oui, choisissez une solution qui prend en charge une ou plusieurs bases de données relationnelles. Sachez que certaines de ces options nécessitent également un cluster Hadoop.

- Avez-vous besoin d’une orchestration automatisée du workflow ou du pipeline de données ? Si oui, tournez-vous vers Azure Data Factory.

## <a name="capability-matrix"></a>Matrice des fonctionnalités

Les tableaux suivants résument les principales différences de fonctionnalités.

### <a name="physical-transfer"></a>Transfert physique

| | Service Azure Import/Export | Azure Data Box |
| --- | --- | --- |
| Facteur de forme | Disques durs ou SSD SATA internes | Appliance matérielle unique, sécurisée et inviolable |
| Microsoft gère la logistique d’expédition | Non  | OUI |
| S’intègre avec les produits partenaires | Non  | OUI |
| Appliance personnalisée | Non  | OUI |

### <a name="command-line-tools"></a>Outils en ligne de commande

**Hadoop/HDInsight**

| | Distcp | Sqoop | Interface CLI Hadoop |
| --- | --- | --- | --- |
| Optimisé pour le Big Data | OUI | OUI |  OUI |
| Copie vers une base de données relationnelle |  Non  | OUI | Non  |
| Copie à partir d’une base de données relationnelle |  Non  | OUI | Non  |
| Copie vers le Stockage Blob |  OUI | OUI | OUI |
| Copie à partir du Stockage Blob | OUI |  OUI | Non  |
| Copie vers Data Lake Store | OUI | OUI | OUI |
| Copie à partir de Data Lake Store | OUI | OUI | Non  |

**Autres**

| | Azure CLI | AzCopy | PowerShell | AdlCopy | PolyBase |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Plateformes compatibles | Linux, OS X, Windows | Linux, Windows | Windows | Linux, OS X, Windows | SQL Server, Azure SQL Data Warehouse | 
| Optimisé pour le Big Data | Non  | Non  | Non  | Oui<sup>1</sup> | Oui<sup>2</sup> |
| Copie vers une base de données relationnelle | Non  | Non  | Non  | Non  | OUI | 
| Copie à partir d’une base de données relationnelle | Non  | Non  | Non  | Non  | OUI | 
| Copie vers le Stockage Blob | OUI | OUI | OUI | Non  | OUI | 
| Copie à partir du Stockage Blob | OUI | OUI | OUI | OUI | OUI |
| Copie vers Data Lake Store | Non  | Non  | OUI | OUI |  OUI | 
| Copie à partir de Data Lake Store | Non  | Non  | OUI | OUI | OUI | 


[1] AdlCopy est optimisé pour le transfert de données volumineuses lorsqu’il est utilisé avec un compte Data Lake Analytics.

[2] Il est possible [d’améliorer les performances](/sql/relational-databases/polybase/polybase-guide#performance) de PolyBase en envoyant les calculs sur Hadoop et en utilisant des [groupes de scale-out PolyBase](/sql/relational-databases/polybase/polybase-scale-out-groups) pour permettre le transfert de données en parallèle entre les instances SQL Server et les nœuds Hadoop.

### <a name="graphical-interface-and-azure-data-factory"></a>Interface graphique et Azure Data Factory

| | Explorateur de stockage Azure | Portail Azure* | Azure Data Factory |
| --- | --- | --- | --- |
| Optimisé pour le Big Data | Non  | Non  | OUI | 
| Copie vers une base de données relationnelle | Non  | Non  | OUI |
| Copie vers une base de données relationnelle | Non  | Non  | OUI |
| Copie vers le Stockage Blob | OUI | Non  | OUI |
| Copie à partir du Stockage Blob | OUI | Non  | OUI |
| Copie vers Data Lake Store | Non  | Non  | OUI |
| Copie à partir de Data Lake Store | Non  | Non  | OUI |
| Chargement vers le Stockage Blob | OUI | OUI | OUI |
| Chargement vers Data Lake Store | OUI | OUI | OUI |
| Orchestration des transferts de données | Non  | Non  | OUI |
| Transformations de données personnalisées | Non  | Non  | OUI |
| Modèle de tarification | Gratuit | Gratuit | Paiement à l’utilisation |

\* Le Portail Azure signifie dans ce cas utiliser les outils web d’exploration pour le Stockage Blob et Data Lake Store.

