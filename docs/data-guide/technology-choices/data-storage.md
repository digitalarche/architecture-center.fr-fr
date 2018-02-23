---
title: "Sélectionner une technologie de stockage de données"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: d8f831e758ddc8604758392644a68b56dc51cf57
ms.sourcegitcommit: 475064f0a3c2fac23e1286ba159aaded287eec86
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/19/2018
---
# <a name="choosing-a-big-data-storage-technology-in-azure"></a>Sélectionner une technologie de stockage de Big Data dans Azure

Cette rubrique compare les options de stockage de données pour les solutions de Big Data, &mdash; plus précisément, le stockage de données pour l’ingestion de données en bloc et le traitement par lots, par opposition aux [magasins de données analytiques](./analytical-data-stores.md) ou à l’[ingestion en continu en temps réel](./real-time-ingestion.md).

## <a name="what-are-your-options-when-choosing-data-storage-in-azure"></a>Quelles sont vos options à l’heure de choisir un stockage de données dans Azure ?

Il existe plusieurs options disponibles pour l’ingestion de données dans Azure, en fonction de vos besoins :

**Stockage Fichier**

- [Stockage d’objets blob Azure](/azure/storage/blobs/storage-blobs-introduction)
- [Azure Data Lake Store](/azure/data-lake-store/)

**Bases de données NoSQL**

- [Azure Cosmos DB](/azure/cosmos-db/)
- [HBase sur HDInsight](http://hbase.apache.org/)

## <a name="azure-storage-blobs"></a>Objets blob de stockage Azure

Le stockage Azure est un service de stockage managé hautement disponible, sécurisé, durable, évolutif et redondant. Microsoft prend en charge la maintenance et gère les problèmes critiques pour vous. Le stockage Azure est la solution de stockage la plus omniprésente d’'Azure, en raison du nombre de services et d’outils qu’elle permet d’utiliser.

Vous pouvez utiliser divers services de stockage Azure pour stocker vos données. L’option la plus flexible pour le stockage d’objets blob à partir de plusieurs sources de données est le [stockage d’objets blob](/azure/storage/blobs/storage-blobs-introduction). Les objets blob sont en fait des fichiers. Ils stockent des images, des documents, des fichiers HTML, des disques durs virtuels (VHD), du Big Data tel que des journaux et des sauvegardes de base de données, autrement dit, pratiquement tout. Les objets blob sont stockés dans des conteneurs, équivalents à des dossiers. Un conteneur regroupe un ensemble d’objets blob. Un compte de stockage peut contenir un nombre illimité de conteneurs, et un conteneur peut stocker un nombre illimité d’objets blob.

Le stockage Azure est un choix judicieux pour les solutions de Big Data et d’analyse, en raison de sa flexibilité, sa haute disponibilité et son faible coût. Il fournit des niveaux de stockage chaud, froid et archive pour différents cas d’usage. Pour plus d’informations, consultez [Stockage Blob Azure : niveaux de stockage chaud, froid et archive](/azure/storage/blobs/storage-blob-storage-tiers).

Le stockage d’objets blob Azure est accessible à partir de Hadoop (disponible via HDInsight). HDInsight peut utiliser un conteneur d’objets blob dans le stockage Azure comme système de fichiers par défaut pour le cluster. Grâce à une interface HDFS (Hadoop Distributed File System) fournie par un pilote WASB, l’ensemble des composants de HDInsight peut fonctionner directement sur les données structurées ou non structurées en tant qu’objets blob. Le stockage d’objets blob Azure est également accessible via Azure SQL Data Warehouse à l’aide de sa fonctionnalité PolyBase.

Parmi les autres fonctionnalités qui font du stockage Azure un choix idéal, citons les suivantes :

- [Stratégies d’accès concurrentiel multiples](/azure/storage/common/storage-concurrency?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
- [Options de récupération d’urgence et de haute disponibilité](/azure/storage/common/storage-disaster-recovery-guidance?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
- [Chiffrement au repos](/azure/storage/common/storage-service-encryption?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
- [Contrôle d’accès en fonction du rôle (RBAC)](/azure/storage/common/storage-security-guide?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#management-plane-security) pour contrôler l’accès à l’aide d’utilisateurs et de groupes Azure Active Directory

## <a name="azure-data-lake-store"></a>Azure Data Lake Store

[Azure Data Lake Store](/azure/data-lake-store/) est un référentiel d'entreprise à très grande échelle pour les charges de travail d'analyse du Big Data. Data Lake vous permet de capturer des données de toute taille, de tout type et à toute vitesse d'ingestion dans un emplacement [sécurisé](/azure/data-lake-store/data-lake-store-overview#DataLakeStoreSecurity) unique en vue d'une analyse opérationnelle et exploratoire.

Data Lake Store n'impose aucune limite de taille de compte, de taille de fichier ou de quantité de données stockées dans un Data Lake. Les données sont stockées durablement en créant des copies multiples. De plus, il n'existe aucune limite de durée de stockage des données dans le Data Lake. En plus de créer des copies de sauvegarde multiples de fichiers que vous pouvez utiliser en cas de défaillances inattendues, Data Lake répartit les parties d’un fichier sur plusieurs serveurs de stockage individuels. Cela améliore le débit de lecture lors de la lecture du fichier en parallèle de l'analyse de données.

Data Lake Store est accessible à partir de Hadoop (disponible avec via HDInsight) à l’aide des API REST compatibles WebHDFS. Vous pouvez envisager cette solution comme une alternative au stockage Azure quand la taille de vos fichiers individuels ou combinés dépasse la taille maximale prise en charge par le stockage Azure. Il existe toutefois des [recommandations sur le paramétrage des performances](/azure/data-lake-store/data-lake-store-performance-tuning-guidance#optimizing-io-intensive-jobs-on-hadoop-and-spark-workloads-on-hdinsight) à suivre si vous utilisez Data Lake Store en tant que stockage principal pour un cluster HDInsight, avec des instructions spécifiques pour [Spark](/azure/data-lake-store/data-lake-store-performance-tuning-spark), [Hive](/azure/data-lake-store/data-lake-store-performance-tuning-hive), [MapReduce](/azure/data-lake-store/data-lake-store-performance-tuning-mapreduce) et [Storm](/azure/data-lake-store/data-lake-store-performance-tuning-storm). Veillez également à vérifier la [disponibilité régionale](https://azure.microsoft.com/regions/#services) de Data Lake Store, car il n’est pas disponible dans toutes les régions prises en charge par le stockage Azure, et il doit se trouver dans la même région que votre cluster HDInsight.

Combiné à Azure Data Lake Analytics, Data Lake Store est spécialement conçu pour permettre l'analyse des données stockées et met l'accent sur les performances des scénarios d'analyse des données. Data Lake Store est également accessible via Azure SQL Data Warehouse à l’aide de sa fonctionnalité PolyBase.

## <a name="azure-cosmos-db"></a>Azure Cosmos DB

[Azure Cosmos DB](/azure/cosmos-db/) est un service de base de données multimodèle mondialement distribué de Microsoft. Cosmos DB garantit des latences en millisecondes à un chiffre au 99e centile partout dans le monde, offre de multiples modèles de cohérence bien définis pour affiner les performances, et garantit une disponibilité optimale grâce à des fonctionnalités d’hébergement multiple.

Azure Cosmos DB est sans schéma. Il indexe automatiquement toutes les données sans avoir à s’occuper de la gestion des schémas et des index. Il est également multimodèle. Les modèles de données de types documents, valeurs clés, graphiques et colonnes sont pris en charge de manière native. 

Fonctionnalités d’Azure Cosmos DB :

- [Géoréplication](/azure/cosmos-db/distribute-data-globally)
- [Mise à l’échelle élastique du débit et du stockage](/azure/cosmos-db/partition-data) à l’échelon mondial
- [Cinq niveaux de cohérence bien définis](/azure/cosmos-db/consistency-levels)

## <a name="hbase-on-hdinsight"></a>HBase sur HDInsight

[Apache HBase](http://hbase.apache.org/) est une base de données NoSQL open source, basée sur Hadoop et modélisée d'après Google BigTable. HBase fournit un accès aléatoire et une forte cohérence pour de vastes quantités de données non structurées et semi-structurées, dans une base de données sans schéma, organisée par familles de colonnes.

Les données sont stockées dans les lignes d'une table et les données au sein d'une ligne sont regroupées par familles de colonnes. HBase est sans schéma dans le sens où ni les colonnes ni le type de données qui y sont stockées ne doivent être définis avant de pouvoir les utiliser. Le code open source peut être mis à l'échelle de façon linéaire pour gérer des pétaoctets de données dans des milliers de nœuds. Il peut reposer sur la redondance des données, le traitement par lots et d'autres fonctionnalités qui sont fournies par des applications distribuées dans l'écosystème Hadoop.

La [mise en œuvre de HDInsight](/azure/hdinsight/hbase/apache-hbase-overview) exploite l'architecture de montée en charge de HBase pour fournir un partitionnement automatique des tables, une cohérence forte pour les lectures et les écritures, et un basculement automatique. Les performances sont optimisées par la mise en cache en mémoire des lectures et par des écritures en diffusion à débit élevé. Dans la plupart des cas, vous souhaiterez certainement [créer le cluster HBase à l’intérieur d’un réseau virtuel](/azure/hdinsight/hbase/apache-hbase-provision-vnet) pour permettre aux autres applications et clusters HDInsight d’accéder directement aux tables.

## <a name="key-selection-criteria"></a>Principaux critères de sélection

Pour restreindre les choix, commencez par répondre aux questions suivantes :

- Avez-vous besoin d’un stockage managé, rapide et basé sur le cloud pour tout type de données texte ou binaires ? Si oui, choisissez l’une des options de stockage de fichiers.

- Avez-vous besoin d’un stockage de fichiers optimisé pour des charges de travail d’analyse parallèles et un haut débit ou un nombre élevé d’E/S par seconde ? Si oui, choisissez une option privilégiant les performances des charges de travail d’analyse.

- Avez-vous besoin de stocker des données non structurées ou semi-structurées dans une base de données sans schéma ? Si oui, sélectionnez l’une des options non relationnelles. Comparez les options disponibles pour l’indexation et les modèles de base de données. Selon le type de données que vous devez stocker, les modèles de base de données primaire peuvent être le facteur le plus important.

- Pouvez-vous utiliser le service dans votre région ? Vérifiez la disponibilité régionale de chaque service Azure. Consultez la [disponibilité des produits par région](https://azure.microsoft.com/regions/services/).

## <a name="capability-matrix"></a>Matrice des fonctionnalités

Les tableaux suivants résument les principales différences entre les fonctionnalités.

### <a name="file-storage-capabilities"></a>Fonctionnalités de stockage de fichiers

|  | Azure Data Lake Store | Conteneurs de stockage d’objets blob Azure |
| --- | --- | --- |
| Objectif | Stockage optimisé pour les charges de travail d’analyse de données volumineuses |Magasin d’objets polyvalent adapté à un large éventail de scénarios de stockage |
| Cas d'utilisation | Données par lots, d’analyse de diffusion en continu et d’apprentissage machine (par exemple, fichiers journaux, données IoT, données sur le parcours de navigation, jeux de données volumineux) | N’importe quel type de données texte ou binaires, par exemple données d’application principale, de sauvegarde, de stockage de médias pour la diffusion en continu, et d’usage général |
| Structure | Système de fichiers hiérarchique | Magasin d’objets avec espace de noms plat |
| Authentification | Basées sur les [Identités Azure Active Directory](/azure/active-directory/active-directory-authentication-scenarios) | Basées sur les secrets partagés : [clés d’accès au compte](/azure/storage/common/storage-create-storage-account#manage-your-storage-account), [clés de signature d’accès partagé](/azure/storage/common/storage-dotnet-shared-access-signature-part-1) et [contrôle d’accès en fonction du rôle (RBAC)](/azure/security/security-storage-overview) |
| Protocole d’authentification | OAuth 2.0. Les appels doivent contenir un JWT (jeton web JSON) valide émis par Azure Active Directory | Code d’authentification de message basé sur le hachage (HMAC). Les appels doivent contenir un hachage SHA-256 codé en Base64 sur une partie de la requête HTTP. |
| Authorization | Listes de contrôle d’accès (ACL) POSIX. Les listes ACL basées sur les identités Azure Active Directory peuvent être définies aux niveaux fichier et dossier. | Pour l’autorisation au niveau des comptes, utilisez des [clés d’accès au compte](/azure/storage/common/storage-create-storage-account#manage-your-storage-account). Pour l’autorisation au niveau d'un compte, d'un conteneur ou d'un objet blob, utilisez des [clés de signature d’accès partagé](/azure/storage/common/storage-dotnet-shared-access-signature-part-1). |
| Audit | Disponible.  |Disponible |
| Chiffrement au repos | Transparent, côté serveur | Transparent, côté serveur ; chiffrement côté client |
| Kits de développement logiciel pour développeur | .NET, Java, Python, Node.js | .NET, Java, Python, Node.js, C++, Ruby |
| Performances des charges de travail d’analyse | Optimisation des performances pour les charges de travail d’analyse parallèles, haut débit et nombre élevé d’E/S par seconde | Non optimisé pour les charges de travail d’analyse |
| Limites de taille | Aucune limite de taille pour les comptes, les fichiers ou le nombre de fichiers | Limites spécifiques documentées [ici](/azure/azure-subscription-service-limits#storage-limits) |
| Géo-redondance | Localement redondant (plusieurs copies des données dans une région Azure) | Localement redondant (LRS), globalement redondant (GRS), accès en lecture redondant globalement (RA-GRS). Pour plus d’informations, voir [ici](/azure/storage/common/storage-redundancy) |

### <a name="nosql-database-capabilities"></a>Fonctionnalités de base de données NoSQL

| | Azure Cosmos DB | HBase sur HDInsight |
| --- | --- | --- |
| Modèle de base de données primaire | Stockage de documents, graphiques, stockage de valeurs clés, stockage de colonnes larges | Stockage de colonnes larges |
| Index secondaires | OUI | Non  |
| Prise en charge du langage SQL | OUI | Oui (à l’aide du pilote JDBC [Phoenix](http://phoenix.apache.org/)) |
| Cohérence | Fort, Obsolescence limitée, Session, Préfixe cohérent et Éventuel | Remarque |
| Intégration native à Azure Functions | [Oui](/azure/cosmos-db/serverless-computing-database) | Non  |
| Distribution mondiale automatique | [Oui](/azure/cosmos-db/distribute-data-globally) | Aucune [réplication de cluster HBase ne peut être configurée ](/azure/hdinsight/hbase/apache-hbase-replication) dans les régions avec une cohérence éventuelle |
| Modèle de tarification | Unités de requête (RU) avec mise à l’échelle élastique facturées par seconde en fonction des besoins, stockage avec mise à l’échelle élastique | Prix par minute du cluster HDInsight (mise à l’échelle horizontale des nœuds), stockage |
