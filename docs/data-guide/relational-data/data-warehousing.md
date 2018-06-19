---
title: Entreposage de données et mini-Data Warehouses
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 9b90d77ce1a81cd4a7532f5d4230ada8b4991d13
ms.sourcegitcommit: 85334ab0ccb072dac80de78aa82bcfa0f0044d3f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/11/2018
ms.locfileid: "35252803"
---
# <a name="data-warehousing-and-data-marts"></a>Entreposage de données et mini-Data Warehouses

Un entrepôt de données est un référentiel central, organisationnel et relationnel de données intégrées provenant d’une ou de plusieurs sources hétérogènes, à travers de nombreuses ou toutes les zones de sujet. Les entrepôts de données stockent des données d’historique et actuelles, et permettent de créer des rapports et d’analyser des données de différentes façons.

![Entreposage de données dans Azure](../images/data-warehousing.png)

Les données à déplacer dans un entrepôt de données sont régulièrement extraites de différentes sources qui contiennent d’importantes informations d’entreprise. Durant leur déplacement, les données peuvent être mises en forme, nettoyées, validées, synthétisées et réorganisées. Elles peuvent également être stockées avec le plus bas niveau de détail, avec des vues agrégées fournies dans l’entrepôt pour la création de rapports. Dans tous les cas, l’entrepôt de données devient un espace de stockage permanent pour les données utilisées pour la création de rapports, l’analyse et la prise d’importantes décisions commerciales à l’aide d’outils de Business Intelligence (BI).

## <a name="data-marts-and-operational-data-stores"></a>Mini-Data Warehouses et magasins de données opérationnels

La gestion des données à l’échelle est complexe, et disposer d’un entrepôt de données unique qui représente l’ensemble des données au sein de toute l’entreprise est une pratique de moins en moins courante. À la place, les organisations créent des entrepôts de données plus petits et plus spécifiques, appelés *mini-Data Warehouses*, qui exposent les données souhaitées à des fins d’analyse. Un processus d’orchestration renseigne les mini-Data Warehouses à partir des données conservées dans un magasin de données opérationnel. Le magasin de données opérationnel agit en tant qu’intermédiaire entre le système transactionnel source et le mini-Data Warehouse. Les données gérées par le magasin de données opérationnel sont une version nettoyée des données présentes dans le système transactionnel source, et représentent généralement un sous-ensemble des données d’historique gérées par l’entrepôt de données ou le mini-Data Warehouse. 

## <a name="when-to-use-this-solution"></a>Quand utiliser cette solution ?

Optez pour un entrepôt de données si vous avez besoin de convertir un grand nombre de données de systèmes d’exploitation dans un format actuel, précis et facile à comprendre. Les entrepôts de données ne doivent pas forcément suivre la même structure de données laconique que vous pouvez utiliser dans vos bases de données opérationnelles/OLTP. Vous pouvez utiliser des noms de colonne pertinents pour les utilisateurs professionnels et les analystes, restructurer le schéma pour simplifier les relations entre les données, et consolider plusieurs tables en une seule. Ces étapes guident les utilisateurs qui ont besoin de créer des rapports ad hoc, ou de créer des rapports et d’analyser les données dans des systèmes de BI, sans l’aide d’un administrateur de base de données (DBA) ou d’un développeur de données.

Vous pouvez utiliser un entrepôt de données si vous avez besoin de conserver vos données d’historique dans un emplacement autre que les systèmes transactionnels sources pour des raisons de performances. Les entrepôts de données facilitent l’accès aux données d’historique à partir de plusieurs emplacements, en fournissant un emplacement centralisé utilisant des formats, des clés, des méthodes d’accès et des modèles de données courants.

L’accès en lecture des entrepôts de données est optimisé, ce qui accélère la génération de rapports en comparaison avec l’exécution de rapports via le système transactionnel source. Les entrepôts de données présentent en outre les avantages suivants :

* Toutes les données d’historique provenant de différentes sources peuvent être stockées et sont accessibles à partir d’un entrepôt de données en tant que source de vérité unique.
* Vous pouvez améliorer la qualité des données en les nettoyant au cours de leur importation dans l’entrepôt de données, afin de disposer de données plus précises, ainsi que de descriptions et codes cohérents.
* Les outils de création de rapports ne sont pas comparables aux systèmes transactionnels sources pour ce qui est des cycles de traitement des requêtes. Un entrepôt de données permet au système transactionnel de se centrer principalement sur la gestion des écritures, tandis que l’entrepôt lui-même répond à la plupart des demandes de lecture.
* Un entrepôt de données peut aider à consolider des données provenant de différents logiciels.
* Les outils d’exploration de données peuvent vous aider à rechercher des modèles masqués à l’aide de méthodologies automatiques appliquées aux données stockées dans votre entrepôt.
* Les entrepôts de données facilitent la mise en place d’un accès sécurisé aux utilisateurs autorisés, tout en limitant l’accès aux autres utilisateurs. Il n’est pas nécessaire d’accorder aux utilisateurs professionnels l’accès aux données sources, ce qui élimine un vecteur d’attaque potentiel par rapport à un ou plusieurs systèmes transactionnels de production.
* Les entrepôts de données facilitent la création de solutions de Business Intelligence applicables aux données, telles que les [cubes OLAP](online-analytical-processing.md).

## <a name="challenges"></a>Défis

La configuration correcte d’un entrepôt de données en fonction des besoins de votre entreprise peut présenter les difficultés suivantes :

* Consacrer le temps nécessaire à la définition correcte des concepts de votre entreprise : il s’agit d’une étape importante, car les entrepôts de données sont axés sur les informations, où le reste du projet dépend du mappage des concepts. Cela implique la normalisation des termes et formats professionnels courants (tels que les devises et les dates) et la restructuration du schéma qui soit pertinente pour les utilisateurs professionnels, mais qui garantisse toujours l’exactitude des relations et agrégats de données.
* Planifier et configurer l’orchestration de vos données : ces étapes doivent notamment prendre en compte la façon dont les données doivent être copiées du système transactionnel source vers l’entrepôt de données, et le moment auquel les données d’historique doivent être déplacées de vos magasins de données opérationnels vers l’entrepôt.
* Préserver ou améliorer la qualité des données en les nettoyant au cours de leur importation dans l’entrepôt.

## <a name="data-warehousing-in-azure"></a>Entreposage de données dans Azure

Dans Azure, vous pouvez avoir une ou plusieurs sources de données, provenant de transactions de clients ou de diverses applications métier utilisées par différents services. Ces données sont généralement stockées dans une ou plusieurs bases de données [OLTP](online-transaction-processing.md). Les données peuvent être persistantes dans d’autres supports de stockage tels que des partages réseau, des objets blob de stockage Azure ou un Data Lake. Elles peuvent également être stockées dans l’entrepôt de données lui-même ou dans une base de données relationnelle comme Azure SQL Database. L’objectif de la couche du magasin de données analytique est de satisfaire les requêtes émises par les outils d’analyse et de création de rapports au niveau de l’entrepôt de données ou du mini-Data Warehouse. Dans Azure, cette fonctionnalité de magasin analytique est disponible avec Azure SQL Data Warehouse, ou avec HDInsight Azure à l’aide d’une requête Hive ou interactive. Vous avez également besoin d’un certain niveau d’orchestration pour déplacer ou copier régulièrement des données du stockage de données vers l’entrepôt de données, ce qui peut être effectué à l’aide d’Azure Data Factory ou d’Oozie sur Azure HDInsight.

Il existe plusieurs façons d’implémenter un entrepôt de données dans Azure, en fonction des besoins. Les listes suivantes sont divisées en deux catégories, [multitraitement symétrique](https://en.wikipedia.org/wiki/Symmetric_multiprocessing) (SMP) et [traitement massivement parallèle](https://en.wikipedia.org/wiki/Massively_parallel) (MPP). 

SMP :

- [Azure SQL Database](/azure/sql-database/)
- [SQL Server dans une machine virtuelle](/sql/sql-server/sql-server-technical-documentation)

MPP :

- [Azure Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [Apache Hive sur HDInsight](/azure/hdinsight/hadoop/hdinsight-use-hive)
- [Interactive Query (Hive LLAP) sur HDInsight](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)

En règle générale, les entrepôts SMP sont idéaux pour les jeux de données de taille petite ou moyenne (jusqu'à 4-100 To), tandis que les entrepôts MPP sont souvent utilisés pour le Big Data. La frontière entre le Big Data et les volumes petits/moyens de données est en partie liée à la définition et à l’infrastructure sous-jacente de votre organisation. (Voir [Choisir un magasin de données OLTP](online-transaction-processing.md#scalability-capabilities).) 

Au-delà de la taille des données, le type de modèle de charge de travail a toutes les chances d’être un facteur plus déterminant. Par exemple, les requêtes complexes risquent d’être trop lentes pour une solution SMP et d’exiger plutôt une solution MPP. Les systèmes MPP sont susceptibles d’imposer une pénalité de performances aux données de petite taille, en raison de la façon dont les tâches sont distribuées et consolidées sur les différents nœuds. Si vos données dépassent déjà 1 To et qu’elles seront amenées à croître en permanence, envisagez une solution MPP. Toutefois, si elles sont de plus petite taille, mais que vos charges de travail dépassent les ressources disponibles de votre solution SMP, le meilleur choix reste peut-être là aussi un système MPP.

Les données lues et stockées par l’entrepôt de données peuvent provenir de différentes sources de données, y compris un lac de données comme [Azure Data Lake Store](/azure/data-lake-store/). Vous trouverez une session vidéo comparant les différents avantages offerts par les services MPP qui peuvent utiliser Azure Data Lake sur la page [Azure Data Lake et Azure Data Warehouse : appliquer des pratiques modernes à une application](https://azure.microsoft.com/resources/videos/build-2016-azure-data-lake-and-azure-data-warehouse-applying-modern-practices-to-your-app/).

Les systèmes SMP sont caractérisés par une seule instance d’un système de gestion de base de données relationnelle, partageant toutes les ressources (UC / mémoire / disque). Un système SMP peut monter en puissance. Si SQL Server est exécuté sur une machine virtuelle, vous pouvez augmenter la taille de cette dernière. Dans le cas d’Azure SQL Database, la montée en puissance passe par le choix d’un autre niveau de service. 

Pour monter en charge des systèmes MPP, il faut ajouter des nœuds de calcul (qui ont leurs propres sous-systèmes d’UC, de mémoire et d’E/S). La montée en puissance d’un serveur est soumise à des limites physiques, au-delà desquelles la montée en charge est plus judicieuse, en fonction de la charge de travail. Toutefois, les solutions MPP requièrent des compétences différentes, en raison des écarts dans la manière d’interroger, de modéliser et de partitionner les données, ainsi que d’autres facteurs propres au traitement parallèle. 

Pour choisir quelle solution SMP utiliser, consultez la page [Retour sur Azure SQL Database et SQL Server sur des machines virtuelles Azure](/azure/sql-database/sql-database-paas-vs-sql-server-iaas#a-closer-look-at-azure-sql-database-and-sql-server-on-azure-vms). 

Azure SQL Data Warehouse est également utilisable pour des jeux de données de taille petite ou moyenne, dont la charge de travail nécessite de nombreuses ressources de calcul et de mémoire. Pour plus d’informations sur les modèles SQL Data Warehouse et les scénarios courants :

- [Modèles et antimodèles SQL Data Warehouse](https://blogs.msdn.microsoft.com/sqlcat/2017/09/05/azure-sql-data-warehouse-workload-patterns-and-anti-patterns/)
- [Stratégies et modèles de charge Azure SQL Data Warehouse](https://blogs.msdn.microsoft.com/sqlcat/2017/05/17/azure-sql-data-warehouse-loading-patterns-and-strategies/)
- [Migrating Data to Azure SQL Data Warehouse](https://blogs.msdn.microsoft.com/sqlcat/2016/08/18/migrating-data-to-azure-sql-data-warehouse-in-practice/) (Migration de données vers Microsoft Azure SQL Data Warehouse)
- [Modèles d’application courants des ISV avec Azure SQL Data Warehouse](https://blogs.msdn.microsoft.com/sqlcat/2017/09/05/common-isv-application-patterns-using-azure-sql-data-warehouse/)

## <a name="key-selection-criteria"></a>Critères de sélection principaux

Pour restreindre les choix, commencez par répondre aux questions suivantes :

- Préférez-vous opter pour un service géré plutôt que de gérer vos propres serveurs ?

- Travaillez-vous avec des jeux de données extrêmement volumineux, ou des requêtes longues et très complexes ? Si oui, envisagez un système MPP. 

- Dans le cas d’un jeu de données volumineux, la source de données est-elle structurée ou non structurée ? Le traitement des données non structurées doit parfois s’effectuer dans un environnement Big Data comme Spark sur HDInsight, Azure Databricks, Hive LLAP sur HDInsight ou Azure Data Lake Analytics. Tous ces outils peuvent servir de moteurs ELT (extraction, chargement, transformation) et ETL (extraction, transformation, chargement). À partir des données traitées, ils peuvent produire des données structurées, plus faciles à charger dans SQL Data Warehouse ou les autres solutions. Pour les données structurées, SQL Data Warehouse a un niveau de performances nommé Optimisé pour le calcul, adapté aux charges de travail nécessitant de nombreuses ressources de calcul et des performances extrêmement élevées.

- Souhaitez-vous séparer vos données historiques de vos données opérationnelles actuelles ? Si oui, sélectionnez l’une des solutions pour lesquelles [l’orchestration](../technology-choices/pipeline-orchestration-data-movement.md) est requise. Ces sont des entrepôts autonomes optimisés pour un accès en lecture intensif et idéaux comme magasins de données historiques distincts.

- Avez-vous besoin d’intégrer des données provenant de plusieurs sources, au-delà de votre magasin de données OLTP ? Si oui, choisissez des solutions qui intègrent facilement plusieurs sources de données. 

- Avez-vous une exigence d’architecture mutualisée ? Si oui, SQL Data Warehouse n’est pas idéal pour y répondre. Pour plus d’informations, consultez la page [Modèles et antimodèles SQL Data Warehouse](https://blogs.msdn.microsoft.com/sqlcat/2017/09/05/azure-sql-data-warehouse-workload-patterns-and-anti-patterns/).

- Préférez-vous un magasin de données relationnel ? Si oui, concentrez-vous sur les solutions possédant un magasin de données relationnel, mais sachez également que vous pouvez utiliser un outil comme PolyBase pour interroger des magasins de données non relationnels si nécessaire. Si vous décidez d’utiliser PolyBase, toutefois, exécutez des tests de performances sur vos jeux de données non structurées pour votre charge de travail.

- Avez-vous des exigences de création de rapports en temps réel ? Si vous avez besoin de faibles temps de réponse aux requêtes sur des volumes élevés d’insertions de singletons, limitez vos recherches aux solutions prenant en charge les rapports en temps réel.

- Devez-vous prendre en charge un grand nombre de connexions et d’utilisateurs simultanés ? La capacité à prendre en charge de nombreux utilisateurs/connexions simultanés dépend de plusieurs facteurs. 

    - Pour Azure SQL Database, référez-vous aux [limites de ressources documentées](/azure/sql-database/sql-database-resource-limits) selon votre niveau de service. 
    
    - SQL Server autorise un maximum de 32 767 connexions utilisateurs. En cas d’exécution sur une machine virtuelle, les performances dépendront de la taille de cette machine virtuelle et d’autres facteurs. 
    
    - SQL Data Warehouse est soumis à des limites sur les requêtes et les connexions simultanées. Pour plus d’informations, consultez la page [Gestion de la concurrence et des charges de travail dans SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-develop-concurrency). Vous pouvez utiliser des services complémentaires, par exemple, [Azure Analysis Services](/azure/analysis-services/analysis-services-overview), pour surmonter les limites de SQL Data Warehouse.

- De quel type est votre charge de travail ? En général, les solutions d’entrepôts MPP sont idéales pour les charges de travail analytiques par lots. Si les vôtres sont de nature transactionnelle, avec beaucoup de petites opérations de lecture/écriture ou d’opérations ligne par ligne, regardez du côté des solutions SMP. Il existe une exception à cette recommandation : si vous utilisez le traitement des flux de données sur un cluster HDInsight, par exemple, Spark Streaming, et que vous stockez les données dans une table Hive.

## <a name="capability-matrix"></a>Matrice des fonctionnalités

Les tableaux suivants résument les principales différences de fonctionnalités.

### <a name="general-capabilities"></a>Fonctionnalités générales

| | Azure SQL Database | SQL Server (machine virtuelle) | SQL Data Warehouse | Apache Hive sur HDInsight | Hive LLAP sur HDInsight |
| --- | --- | --- | --- | --- | --- | -- |
| Est un service géré | OUI | Non  | OUI | Oui <sup>1</sup> | Oui<sup>1</sup> |
| Nécessite l’orchestration des données (conserve une copie des données/données historiques) | Non  | Non  | OUI | OUI | OUI |
| Intègre facilement plusieurs sources de données | Non  | Non  | OUI | OUI | OUI |
| Prend en charge l’interruption du calcul | Non  | Non  | OUI | Non<sup>2</sup> | Non<sup>2</sup> |
| Magasin de données relationnel | OUI | OUI |  OUI | Non  | Non  |
| Rapports en temps réel | OUI | OUI | Non  | Non  | OUI |
| Points de restauration de sauvegarde flexibles | OUI | OUI | Non<sup>3</sup> | Oui<sup>4</sup> | Oui<sup>4</sup> |
| SMP/MPP | SMP | SMP | MPP | MPP | MPP |

[1] Mise à l’échelle et configuration manuelles.

[2] Les clusters HDInsight peuvent être supprimés s’ils ne sont pas nécessaires, puis recréés. Attachez un magasin de données externe à votre cluster afin de conserver vos données si vous supprimez votre cluster. Vous pouvez utiliser Azure Data Factory pour automatiser le cycle de vie de votre cluster en créant un cluster HDInsight à la demande pour traiter votre charge de travail, puis en le supprimant une fois le traitement terminé.

[3] Avec SQL Data Warehouse, vous pouvez restaurer une base de données sur n’importe quel point de restauration disponible au cours des sept derniers jours. Les captures instantanées démarrent toutes les quatre à huit heures et sont disponibles pendant sept jours. Quand une capture instantanée a une ancienneté supérieure à sept jours, elle expire et son point de restauration n’est plus disponible.

[4] Envisagez d’utiliser un [metastore Hive externe](/azure/hdinsight/hdinsight-hadoop-provision-linux-clusters#use-hiveoozie-metastore) qui peut être sauvegardé et restauré en fonction des besoins. Il est possible d’utiliser les options de sauvegarde et de restauration standard qui s’appliquent au Stockage Blob ou à Data Lake Store pour les données, ou bien des solutions de sauvegarde et de restauration HDInsight tierces, comme [Imanis Data](https://azure.microsoft.com/blog/imanis-data-cloud-migration-backup-for-your-big-data-applications-on-azure-hdinsight/), pour plus de flexibilité et de facilité d’utilisation.

### <a name="scalability-capabilities"></a>Fonctionnalités d’évolutivité

| | Azure SQL Database | SQL Server (machine virtuelle) |  SQL Data Warehouse | Apache Hive sur HDInsight | Hive LLAP sur HDInsight |
| --- | --- | --- | --- | --- | --- | -- |
| Serveurs régionaux redondants pour assurer une haute disponibilité  | OUI | OUI | OUI | Non  | Non  |
| Prend en charge les requêtes avec montée en charge (requêtes distribuées)  | Non  | Non  | OUI | OUI | OUI |
| Évolutivité dynamique | OUI | Non  | Oui <sup>1</sup> | Non  | Non  |
| Prend en charge la mise en cache en mémoire des données | OUI |  OUI | Non  | OUI | OUI |

[1] SQL Data Warehouse permet de monter ou de descendre en puissance en ajustant le nombre d’unités DWU (Data Warehouse Unit). Voir [Gérer la puissance de calcul dans Azure SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-manage-compute-overview).

### <a name="security-capabilities"></a>Fonctionnalités de sécurité

|                         |           Azure SQL Database            |  SQL Server dans une machine virtuelle  | SQL Data Warehouse |   Apache Hive sur HDInsight    |    Hive LLAP sur HDInsight     |
|-------------------------|-----------------------------------------|-----------------------------------|--------------------|-------------------------------|-------------------------------|
|     Authentification      | SQL / Azure Active Directory (Azure AD) | SQL / Azure AD / Active Directory |   SQL / Azure AD   | local / Azure AD<sup>1</sup> | local / Azure AD <sup>1</sup> |
|      Authorization      |                   OUI                   |                OUI                |        OUI         |              OUI              |       Oui <sup>1</sup>        |
|        Audit         |                   OUI                   |                OUI                |        OUI         |              OUI              |       Oui <sup>1</sup>        |
| Chiffrement des données au repos |            Oui <sup>2</sup>             |         Oui<sup>2</sup>          |  Oui<sup>2</sup>  |       Oui <sup>2</sup>        |       Oui <sup>1</sup>        |
|   Sécurité au niveau des lignes    |                   OUI                   |                OUI                |        OUI         |              Non                |       Oui <sup>1</sup>        |
|   Prend en charge les pare-feu    |                   OUI                   |                OUI                |        OUI         |              OUI              |       Oui <sup>3</sup>        |
|  Masquage des données dynamiques   |                   OUI                   |                OUI                |        OUI         |              Non                |       Oui <sup>1</sup>        |

[1] Suppose d’utiliser un [cluster HDInsight joint à un domaine](/azure/hdinsight/domain-joined/apache-domain-joined-introduction).

[2] Suppose d’utiliser le chiffrement TDE (Transparent Data Encryption) pour chiffrer et déchiffrer les données au repos.

[3] Pris en charge si [utilisé au sein d’un Réseau virtuel Azure](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network).

Pour plus d’informations sur la sécurisation de l’entrepôt de données :

* [Sécurisation de votre base de données SQL](/azure/sql-database/sql-database-security-overview#connection-security)
* [Sécuriser une base de données dans SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-manage-security)
* [Étendre HDInsight à l’aide d’un Réseau virtuel Azure](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network)
* [Sécurité Hadoop au niveau de l’entreprise avec des clusters HDInsight joints à un domaine](/azure/hdinsight/domain-joined/apache-domain-joined-introduction)

