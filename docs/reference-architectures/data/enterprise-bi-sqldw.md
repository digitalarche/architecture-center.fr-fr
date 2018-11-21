---
title: Enterprise BI avec SQL Data Warehouse
description: Utiliser Azure pour obtenir des analyses détaillées des données relationnelles stockées en local
author: MikeWasson
ms.date: 11/06/2018
ms.openlocfilehash: 2822cf6d2a75d521f182c267f4bf2bac462d2b7f
ms.sourcegitcommit: 877777094b554559dc9cb1f0d9214d6d38197439
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2018
ms.locfileid: "51527709"
---
# <a name="enterprise-bi-in-azure-with-sql-data-warehouse"></a>Enterprise BI dans Azure avec SQL Data Warehouse

Cette architecture de référence implémente un pipeline [ELT](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt) (extract-load-transform) qui déplace des données d’une base de données SQL Server vers SQL Data Warehouse et qui transforme les données pour les analyser. 

Une implémentation de référence pour cette architecture est disponible sur [GitHub][github-folder]

![](./images/enterprise-bi-sqldw.png)

**Scénario**: Une organisation dispose d’un jeu de données OLTP important stocké dans une base de données SQL Server locale. L’organisation souhaite utiliser SQL Data Warehouse pour réaliser une analyse avec Power BI. 

Cette architecture de référence est conçue pour des tâches uniques ou à la demande. Si vous devez déplacer des données continuellement (toutes les heures ou tous les jours), nous vous recommandons d’utiliser Azure Data Factory pour définir un flux de travail automatisé. Pour accéder à une architecture de référence qui utilise Data Factory, consultez [BI d’entreprise automatisée avec SQL Data Warehouse et Azure Data Factory][adf-ra].

## <a name="architecture"></a>Architecture

L’architecture est constituée des composants suivants.

### <a name="data-source"></a>Source de données

**SQL Server**. Les données sources sont situées dans une base de données SQL Server locale. Pour simuler l’environnement local, les scripts de déploiement de cette architecture approvisionnent une machine virtuelle dans Azure disposant de SQL Server. [L’exemple de base de données OLTP Wide World Importers][wwi] est utilisé comme base de données source.

### <a name="ingestion-and-data-storage"></a>Ingestion et stockage de données

**Stockage d'objets blob**. Le stockage d’objets blob est utilisé comme zone de préparation à la copie des données, avant de les charger dans SQL Data Warehouse.

**Azure SQL Data Warehouse**. [SQL Data Warehouse](/azure/sql-data-warehouse/) est un système distribué conçu pour réaliser des analyses sur de grandes quantités de données. Il prend en charge le traitement MPP (Massive Parallel Processing), le rendant ainsi adapté à l’exécution d’analyses hautes performances. 

### <a name="analysis-and-reporting"></a>Analyse et rapports

**Azure Analysis Services**. [Analysis Services](/azure/analysis-services/) est un service entièrement géré qui fournit des capacités de modélisation des données. Utilisez Analysis Services pour créer un modèle sémantique que les utilisateurs peuvent demander. Analysis Services est particulièrement utile dans un scénario de tableau de bord BI. Dans cette architecture, Analysis Services lit les données de l’entrepôt de données pour traiter le modèle sémantique, et délivrer efficacement les requêtes du tableau de bord. Il prend aussi en charge la concurrence élastique, en adaptant les réplicas en vue d’un traitement des requêtes plus rapide.

À l’heure actuelle, Azure Analysis Services prend en charge les modèles tabulaires, mais pas les modèles multidimensionnels. Les modèles tabulaires utilisent des constructions de modélisation relationnelle (tables et colonnes), tandis que les modèles multidimensionnels utilisent des constructions de modélisation de traitement analytique en ligne (cubes, dimensions, et mesures). Si vous avez besoin de modèles multidimensionnels, utilisez SQL Server Analysis Services (SSAS). Pour en savoir plus, consultez [Comparaison des solutions tabulaires et multidimensionnelles](/sql/analysis-services/comparing-tabular-and-multidimensional-solutions-ssas).

**Power BI**. Power BI est une suite d’outils d’analyse métier pour analyser les données et obtenir des informations métier. Dans cette architecture, il demande le modèle sémantique stocké dans Analysis Services.

### <a name="authentication"></a>Authentification

**Azure Active Directory** (Azure AD) authentifie les utilisateurs qui se connectent au serveur Analysis Services via Power BI.

## <a name="data-pipeline"></a>Pipeline de données
 
Cette architecture de référence utilise l’exemple de base de données [WorldWideImporters](/sql/sample/world-wide-importers/wide-world-importers-oltp-database) en tant que source de données. Le pipeline de données comporte les étapes suivantes :

1. Exportez les données de SQL Server vers des fichiers plats (utilitaire BCP).
2. Copiez les fichiers plats dans le Stockage Blob Azure (AzCopy).
3. Chargez les données dans SQL Data Warehouse à (PolyBase).
4. Transformez les données en schéma en étoile (T-SQL).
5. Chargez un modèle sémantique dans Analysis Services (SQL Server Data Tools).

![](./images/enterprise-bi-sqldw-pipeline.png)
 
> [!NOTE]
> Pour les étapes 1 &ndash; 3, préférez l’utilisation de Redgate Data Platform Studio. Data Platform Studio applique les correctifs de compatibilité et optimisations les plus appropriés. Il s’agit donc du moyen le plus rapide pour vous familiariser avec SQL Data Warehouse. Pour en savoir plus, consultez [Chargement de données avec Redgate Data Platform Studio](/azure/sql-data-warehouse/sql-data-warehouse-load-with-redgate). 

Les sections suivantes décrivent ces étapes plus en détail.

### <a name="export-data-from-sql-server"></a>Exporter des données depuis SQL Server

L’utilitaire [BCP](/sql/tools/bcp-utility) (programme de copie en bloc) constitue un moyen rapide de créer des fichiers texte plats à partir des tables SQL. Dans cette étape, vous sélectionnez les colonnes que vous souhaitez exporter, mais vous ne transformez pas les données. Les transformations de données doivent se faire dans SQL Data Warehouse.

**Recommandations**

Si possible, prévoyez l’extraction des données lors des heures creuses afin de minimiser la contention des ressources dans l’environnement de production. 

N’exécutez pas l’utilitaire BCP sur le serveur de base de données. À la place, exécutez-le depuis une autre machine. Écrivez les fichiers sur un disque local. Veillez à disposer de suffisamment de ressources d’E/S pour gérer les écritures simultanées. Pour de meilleures performances, exportez les fichiers vers des disques de stockage rapides dédiés.

Vous pouvez accélérer le transfert réseau en enregistrant les données exportées dans un format compressé Gzip. Toutefois, le chargement de fichiers compressés dans l’entrepôt est plus long qu’un chargement de fichiers décompressés. Il faut donc choisir entre un transfert réseau rapide et un chargement rapide. Si vous choisissez d’utiliser la compression Gzip, ne créez pas qu’un seul fichier Gzip. À la place, divisez les données en plusieurs fichiers compressés.

### <a name="copy-flat-files-into-blob-storage"></a>Copier des fichiers plats dans le stockage d’objets blob

L’utilitaire [AzCopy](/azure/storage/common/storage-use-azcopy) est conçu pour la copie hautes performances des données dans le Stockage Blob Azure.

**Recommandations**

Créez un compte de stockage dans une région proche de l’emplacement des données source. Déployez le compte de stockage et l’instance SQL Data Warehouse dans la même région. 

N’exécutez pas AzCopy sur la machine qui exécute vos charges de travail de production, car l’unité centrale et la consommation d’E/S peuvent interférer avec elles. 

Testez le chargement afin de déterminer la vitesse. Vous pouvez utiliser l’option /NC dans AzCopy pour spécifier le nombre d’opérations de copie simultanées. Commencez par la valeur par défaut, puis expérimentez avec ce paramètre pour ajuster les performances. Remarque : un trop grand nombre d’opérations simultanées dans un environnement à faible bande passante peut surcharger la connexion réseau et entraver la réussite des opérations.  

AzCopy déplace les données vers le stockage via l’Internet public. Si ce n’est pas assez rapide, envisagez la configuration d’un circuit [ExpressRoute](/azure/expressroute/). ExpressRoute est un service qui achemine vos données via une connexion privée dédiée vers Azure. Si votre connexion réseau est trop lente, une autre option consiste à envoyer les données physiquement sur disque vers un centre de données Azure. Pour plus d’informations, consultez l’article [Transférer des données vers et à partir d’Azure](/azure/architecture/data-guide/scenarios/data-transfer).

Lors d’une opération de copie, AzCopy crée un fichier journal temporaire, qui permet à AzCopy de redémarrer l’opération si elle est interrompue (à cause d’une erreur réseau, par exemple). Veillez à disposer de suffisamment d’espace disque pour stocker les fichiers journaux. Vous pouvez utiliser l’option /Z pour spécifier où sont écrits les fichiers journaux.

### <a name="load-data-into-sql-data-warehouse"></a>Chargement de données dans SQL Data Warehouse

Utilisez [PolyBase](/sql/relational-databases/polybase/polybase-guide) pour charger des fichiers du stockage d’objets blob vers l’entrepôt de données. PolyBase est conçu pour tirer parti de l’architecture MPP (Massively Parallel Processing) de SQL Data Warehouse, ce qui en fait le moyen le plus rapide pour y charger des données. 

Le chargement des données est un processus en deux étapes :

1. Créez un ensemble de tables externes pour les données. Une table externe est une définition de table qui pointe vers des données stockées à l’extérieur de l’entrepôt &mdash;. Dans notre cas, il s’agit des fichiers plats dans le stockage d’objets blob. Cette étape ne déplace aucune donnée dans l’entrepôt.
2. Créez des tables de mise en lots, et chargez-y les données. Cette étape copie les données dans l’entrepôt.

**Recommandations**

Préférez utiliser SQL Data Warehouse lorsque vous disposez d’une importante quantité de données (supérieure à 1 To) et que vous exécutez une charge de travail d’analyse qui profiterait de ce parallélisme. SQL Data Warehouse ne convient pas à des charges de travail OLTP ni à des jeux de données moins importants (inférieurs à 250 Go). Pour les jeux de données inférieurs à 250 Go, préférez utiliser Azure SQL Database ou SQL Server. Pour plus d’informations, consultez la page [Entreposage des données](../../data-guide/relational-data/data-warehousing.md).

Créez les tables de mise en lots comme tables de segments de mémoire, qui ne sont pas indexées. Les requêtes qui créent les tables de production créeront une analyse de table complète. Il n’y a donc aucune raison d’indexer les tables de mise en lots.

PolyBase tire automatiquement parti du parallélisme dans l’entrepôt. Les performances de chargement s’adaptent à mesure que vous augmentez les DWU (Data Warehouse Units). Pour de meilleures performances, utilisez une seule opération de chargement. Il n’y a pas d’améliorations de performances pour diviser les données entrantes et exécuter plusieurs chargements simultanés.

PolyBase peut lire les fichiers compressés au format Gzip. Toutefois, seul un lecteur unique est utilisé par fichier compressé, car la décompression du fichier est une opération à un seul thread. Évitez donc de charger un seul gros fichier compressé. À la place, divisez les données en plusieurs fichiers compressés, afin de tirer parti du parallélisme. 

Notez les limitations suivantes :

- PolyBase prend en charge une taille de colonne maximale de `varchar(8000)`, `nvarchar(4000)` ou `varbinary(8000)`. Si vos données dépassent ces limites, une option consiste à les diviser au moment de les exporter, puis de les réassembler après les avoir importées. 

- PolyBase utilise un terminateur de ligne fixe \n ou un renvoi à la ligne. Cela peut entraîner des problèmes si les caractères de renvoi à la ligne apparaissent dans les données source.

- Votre schéma des données source peut contenir des types de données qui ne sont pas pris en charge dans SQL Data Warehouse.

Pour contourner ces limitations, vous pouvez créer une procédure stockée qui réalise les conversions nécessaires. Référencez cette procédure stockée lorsque vous exécutez l’utilitaire BCP. Autrement, [Redgate Data Platform Studio](/azure/sql-data-warehouse/sql-data-warehouse-load-with-redgate) convertit automatiquement les types de données qui ne sont pas pris en charge dans SQL Data Warehouse.

Pour plus d’informations, consultez les articles suivants :

- [Meilleures pratiques de chargement de données dans Azure SQL Data Warehouse](/azure/sql-data-warehouse/guidance-for-loading-data).
- [Migration de votre schéma vers SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-migrate-schema)
- [Conseils relatifs à la définition des types de données pour tables dans SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-tables-data-types)

### <a name="transform-the-data"></a>Transformer les données

Transformez les données et les déplacer dans des tables de production. Dans cette étape, les données sont transformées en schéma en étoile avec des tables de dimension et de faits, adaptées à la modélisation sémantique.

Créez les tables de production avec des index columstore en cluster, ce qui offre les meilleures performances globales de requête. Les index columnstore sont optimisés pour les requêtes qui analysent de nombreux enregistrements. Les index columnstore ne sont pas aussi efficaces pour les recherches singleton (rechercher une seule ligne). Si vous avez besoin d’effectuer fréquemment des recherches singleton, vous pouvez ajouter un index non cluster à une table. Les recherches singleton peuvent s’exécuter bien plus rapidement avec un index non cluster. Toutefois, elles sont généralement moins fréquentes dans des scénarios d’entrepôt de données que des charges de travail OLTP. Pour plus d’informations, consultez [Indexage de tables dans SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-tables-index).

> [!NOTE]
> Les tables columnstore cluster ne prennent pas en charge les types de données `varchar(max)`, `nvarchar(max)` ou `varbinary(max)`. Dans ces cas, préférez utiliser un segment de mémoire ou un index cluster. Vous pouvez placer ces colonnes dans une table distincte.

Comme l’exemple de base de données n’est pas très important, nous avons créé des tables répliquées sans partition. Pour les charges de travail de production, l’utilisation de tables distribuées a des chances d’améliorer les performances de requête. Consultez le [Guide de conception des tables distribuées dans Azure SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-tables-distribute). Nos exemples de scripts exécutent les requêtes avec une [classe de ressources](/azure/sql-data-warehouse/resource-classes-for-workload-management) statique.

### <a name="load-the-semantic-model"></a>Charger le modèle sémantique

Chargez les données dans un modèle tabulaire dans Azure Analysis Services. Dans cette étape, vous créez un modèle de données sémantique avec SQL Server Data Tools (SSDT). Vous pouvez aussi créer un modèle en l’important depuis un fichier Power BI Desktop. Comme SQL Data Warehouse ne prend pas en charge les clés étrangères, vous devez ajouter les relations au modèle sémantique afin de joindre les tables.

### <a name="use-power-bi-to-visualize-the-data"></a>Utiliser Power BI pour visualiser les données

Power BI prend en charge deux options pour la connexion à Azure Analysis Services :

- Importation. Les données sont importées dans le modèle Power BI.
- Connexion active. Les données sont extraites directement depuis Analysis Services.

Nous vous recommandons l’option Connexion active car elle ne nécessite pas de copier des données dans le modèle Power BI. De plus, DirectQuery veille à ce que les résultats soient toujours cohérents avec les données sources les plus récentes. Pour plus d’informations, consultez [Connexion avec Power BI](/azure/analysis-services/analysis-services-connect-pbi).

**Recommandations**

N’exécutez pas des requêtes de tableau de bord BI directement dans l’entrepôt de données. Les tableaux de bord BI nécessitent des temps de réponse très lents. Les requêtes directes dans l’entrepôt de données peuvent ne pas être adaptées. De plus, l’actualisation du tableau de bord comptera dans le nombre de requêtes simultanées, ce qui peut impacter les performances. 

Azure Analysis Services est conçu pour gérer les exigences de requête d’un tableau de bord BI. Il est donc recommandé d’effectuer des requêtes Analysis Services depuis Power BI.

## <a name="scalability-considerations"></a>Considérations relatives à l’extensibilité

### <a name="sql-data-warehouse"></a>SQL Data Warehouse

Avec SQL Data Warehouse, vous pouvez augmenter la taille de vos ressources de calcul à la demande. Le moteur de requête optimise les requêtes pour des traitements simultanés basés sur le nombre de nœuds de calcul, et déplace les données entre nœuds si nécessaire. Pour plus d’informations, consultez [Gérer le calcul dans Azure SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-manage-compute-overview).

### <a name="analysis-services"></a>Analysis Services

Pour les charges de travail de production, nous recommandons le niveau Standard pour Azure Analysis Services, car il prend en charge le partitionnement et DirectQuery. Dans un niveau, la taille de l’instance détermine la mémoire et la puissance de traitement. La puissance de traitement se mesure en QPU (Unité de traitement des requêtes). Surveillez votre utilisation QPU pour sélectionner la taille appropriée. Pour plus d’informations, voir [Surveiller les mesures du serveur](/azure/analysis-services/analysis-services-monitor).

Sous une charge importante, les performances de requête peuvent se dégrader en raison de la simultanéité des requêtes. Vous pouvez augmenter la taille d’Analyse Services en créant un pool de réplicas pour traiter des requêtes, dans le but de pouvoir réaliser plus de requêtes simultanément. Le traitement du modèle des données se fait toujours sur le serveur principal. Par défaut, le serveur principal gère aussi les requêtes. Optionnellement, vous pouvez désigner le serveur principal pour qu’il n’exécute que le traitement, afin que le pool de requêtes gère toutes les requêtes. Si vous avez des exigences de traitement élevées, vous devriez séparer le traitement et le pool de requêtes. Si vous avez des charges de requêtes importantes, et un traitement relativement léger, vous pouvez inclure le serveur principal dans le pool de requêtes. Pour en savoir plus, consultez [Extensibilité d’Azure Analysis Services](/azure/analysis-services/analysis-services-scale-out). 

Pour réduire la quantité de traitement inutile, préférez utiliser des partitions pour diviser le modèle tabulaire en plusieurs parties logiques. Chaque partition peut être traitée séparément. Pour plus d'informations, consultez [Partitions](/sql/analysis-services/tabular-models/partitions-ssas-tabular).

## <a name="security-considerations"></a>Considérations relatives à la sécurité

### <a name="ip-whitelisting-of-analysis-services-clients"></a>Liste verte IP des clients Analysis Services

Utilisez la fonctionnalité de pare-feu Analysis Services pour mettre les adresses IP client sur liste verte. S’il est activé, le pare-feu bloque toutes les connexions clients autres que celles spécifiées dans les règles du pare-feu. Les règles par défaut mettent le service Power BI sur liste verte, mais vous pouvez désactiver cette règle si vous le souhaitez. Pour plus d’informations, consultez [Hardening Azure Analysis Services with the new firewall capability](https://azure.microsoft.com/blog/hardening-azure-analysis-services-with-the-new-firewall-capability/)(Consolidation d’Azure Analysis Services avec la nouvelle capacité de pare-feu).

### <a name="authorization"></a>Authorization

Azure Analysis Services utilise Azure Active Directory (Azure AD) pour authentifier les utilisateurs qui se connectent à un serveur Analysis Services. Vous pouvez limiter les données qu’un utilisateur spécifique peut consulter en créant des rôles et en les assignant à des utilisateurs ou groupes Azure AD. Pour chaque rôle, vous pouvez : 

- Protégez des tables ou des colonnes individuelles. 
- Protégez des lignes individuelles basées sur des expressions filtrées. 

Pour en savoir plus, consultez [Gérer les rôles et les utilisateurs de bases de données](/azure/analysis-services/analysis-services-database-users).

## <a name="deploy-the-solution"></a>Déployer la solution

Pour déployer et exécuter l’implémentation de référence, suivez les étapes du [fichier Readme de GitHub][github-folder]. Il déploie les éléments suivants :

  * Une machine virtuelle pour simuler un serveur de base de données local. Sont inclus SQL Server 2017 et les outils associés, et Power BI Desktop.
  * Un compte de stockage Azure qui fournit le stockage d’objets blob pour conserver des données exportées de la base de données SQL Server.
  * Une instance Azure SQL Data Warehouse.
  * Une instance Azure Analysis Services.


## <a name="next-steps"></a>Étapes suivantes

- Utilisez Azure Data Factory pour automatiser le pipeline ELT. Voir [BI d’entreprise automatisée avec SQL Data Warehouse et Azure Data Factory][adf-ra].

<!-- links -->

[adf-ra]: ./enterprise-bi-adf.md
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw
[wwi]: /sql/sample/world-wide-importers/wide-world-importers-oltp-database

