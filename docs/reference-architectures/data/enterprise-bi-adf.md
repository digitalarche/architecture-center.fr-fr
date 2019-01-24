---
title: Décisionnel d’entreprise automatisé
titleSuffix: Azure Reference Architectures
description: Automatisez un workflow ELT (Extract-Load-Transform) dans Azure à l’aide d’Azure Data Factory avec SQL Data Warehouse.
author: MikeWasson
ms.date: 11/06/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18
ms.openlocfilehash: 020c401e9db85b76fd48c6df9be9c80d2ba5c7e4
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54481471"
---
# <a name="automated-enterprise-bi-with-sql-data-warehouse-and-azure-data-factory"></a>BI d’entreprise automatisée avec SQL Data Warehouse et Azure Data Factory

Cette architecture de référence indique comment effectuer un chargement incrémentiel dans un pipeline [ELT (Extract-Load-Transform)](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt). Elle utilise Azure Data Factory pour automatiser le pipeline ELT. Ce pipeline déplace de façon incrémentielle les données OLTP les plus récentes d’une base de données SQL Server locale vers une instance SQL Data Warehouse. Les données transactionnelles sont transformées en un modèle tabulaire à des fins d’analyse.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2Gnz2]

Une implémentation de référence pour cette architecture est disponible sur [GitHub][github].

![Diagramme d’architecture pour le décisionnel d’entreprise automatisé avec SQL Data Warehouse et Azure Data Factory](./images/enterprise-bi-sqldw-adf.png)

Cette architecture repose sur celle qui est décrite dans la section [Enterprise BI avec SQL Data Warehouse](./enterprise-bi-sqldw.md), mais ajoute des fonctionnalités qui sont importantes pour les scénarios d’entreposage de données d’entreprise.

- Automatisation du pipeline à l’aide de Data Factory.
- Chargement incrémentiel.
- Intégration de plusieurs sources de données.
- Chargement des données binaires comme les données géospatiales et les images.

## <a name="architecture"></a>Architecture

L’architecture est constituée des composants suivants.

### <a name="data-sources"></a>Sources de données

**Serveur SQL Server local**. Les données sources sont situées dans une base de données SQL Server locale. Pour simuler l’environnement local, les scripts de déploiement de cette architecture approvisionnent une machine virtuelle dans Azure disposant de SQL Server. [L’exemple de base de données OLTP Wide World Importers][wwi] est utilisé comme base de données source.

**Données externes**. Un scénario courant de gestion des entrepôts de données consiste à intégrer plusieurs sources de données. Cette architecture de référence charge un jeu de données externe qui contient la population par ville et par année, et l’intègre dans les données de la base de données OLTP. Vous pouvez utiliser ces données pour obtenir des insights du type : « La croissance des ventes dans chaque région correspond-elle, voire dépasse-t-elle la croissance de la population ? »

### <a name="ingestion-and-data-storage"></a>Ingestion et stockage de données

**Stockage d'objets blob**. Le stockage d’objets blob est utilisé en tant que zone de processus de site pour les données sources, avant leur chargement dans SQL Data Warehouse.

**Azure SQL Data Warehouse**. [SQL Data Warehouse](/azure/sql-data-warehouse/) est un système distribué conçu pour réaliser des analyses sur de grandes quantités de données. Il prend en charge le traitement MPP (Massive Parallel Processing), le rendant ainsi adapté à l’exécution d’analyses hautes performances.

**Azure Data Factory**. [Data Factory][adf] est un service géré qui orchestre et automatise le déplacement et la transformation des données. Dans cette architecture, il coordonne les différentes étapes du processus ELT.

### <a name="analysis-and-reporting"></a>Analyse et rapports

**Azure Analysis Services**. [Analysis Services](/azure/analysis-services/) est un service entièrement géré qui fournit des capacités de modélisation des données. Le modèle sémantique est chargé dans Analysis Services.

**Power BI**. Power BI est une suite d’outils d’analyse métier pour analyser les données et obtenir des informations métier. Dans cette architecture, il demande le modèle sémantique stocké dans Analysis Services.

### <a name="authentication"></a>Authentification

**Azure Active Directory** (Azure AD) authentifie les utilisateurs qui se connectent au serveur Analysis Services via Power BI.

Data Factory peut également utiliser Azure AD pour s’authentifier auprès de SQL Data Warehouse, en utilisant un principal de service ou une identité MSI (Managed Service Identity). Par souci de simplicité, l’exemple de déploiement utilise l’authentification SQL Server.

## <a name="data-pipeline"></a>Pipeline de données

Dans [Azure Data Factory][adf], un pipeline est un regroupement logique d’activités qui permet de coordonner une tâche &mdash;. Dans ce cas, il s’agit du chargement et de la transformation des données dans SQL Data Warehouse.

Cette architecture de référence définit un pipeline principal qui exécute une suite de pipelines enfants. Chaque pipeline enfant charge les données dans une ou plusieurs tables d’entrepôt de données.

![Capture d’écran du pipeline dans Azure Data Factory](./images/adf-pipeline.png)

## <a name="incremental-loading"></a>Chargement incrémentiel

Lorsque vous exécutez un processus ETL ou ELT automatisé, il s’avère plus efficace de charger uniquement les données modifiées depuis l’exécution précédente. Il s’agit d’un *chargement incrémentiel*, par opposition à un chargement complet, qui porte sur toutes les données. Pour effectuer un chargement incrémentiel, vous avez besoin d’une méthode d’identification des données modifiées. L’approche la plus courante consiste à utiliser une valeur *limite supérieure*, qui se traduit par le suivi de la valeur la plus récente d’une colonne de la table source, soit une colonne DateHeure, soit une colonne d’entier unique.

Depuis SQL Server 2016, vous pouvez utiliser des [tables temporelles](/sql/relational-databases/tables/temporal-tables). Il s’agit de tables de système par version qui conservent un historique complet des modifications apportées aux données. Le moteur de base de données enregistre automatiquement l’historique de chaque modification dans une table d’historique distincte. Vous pouvez interroger les données d’historique en ajoutant une clause FOR SYSTEM_TIME à une requête. En interne, le moteur de base de données interroge la table d’historique, mais cette opération est transparente pour l’application.

> [!NOTE]
> Pour les versions antérieures de SQL Server, vous pouvez utiliser la fonction [Capture des changements de données](/sql/relational-databases/track-changes/about-change-data-capture-sql-server) (CDC). Cette approche est moins pratique que les tables temporelles, car vous devez interroger une table de modifications distincte, et les modifications font l’objet d’un suivi par numéro séquentiel dans le journal plutôt que par horodatage.
>

Les tables temporelles sont utiles pour les données de dimension, qui peuvent changer au fil du temps. Les tables de faits représentent généralement une transaction immuable, comme une vente. De ce fait, il n’est pas utile de conserver l’historique des versions du système. Au lieu de cela, les transactions incluent généralement une colonne qui représente la date de transaction, qui peut être utilisée en tant que la valeur de filigrane. Par exemple, dans la base de données OLTP Wide World Importers, les tables Sales.Invoices et Sales.InvoiceLines incluent un champ `LastEditedWhen` dont la valeur par défaut est `sysdatetime()`.

Voici le flux général du pipeline ELT :

1. Pour chaque table de la base de données source, effectuez le suivi de l’heure de coupure lors de l’exécution du dernier travail ELT. Stockez ces informations dans l’entrepôt de données. (Lors de la configuration initiale, toutes les heures sont définies sur « 1-1-1900 ».)

2. Lors de l’étape d’exportation des données, l’heure de coupure est transmise sous la forme d’un paramètre à un ensemble de procédures stockées dans la base de données source. Ces procédures stockées demandent au système tous les enregistrements ayant été modifiés ou créés après l’heure de coupure. Pour la table de faits Sales, la colonne `LastEditedWhen` est utilisée. Dans le cas des données de dimension, les tables temporelles de système par version sont utilisées.

3. Lorsque la migration des données est terminée, mettez à jour la table qui stocke les heures de coupure.

Il est également utile d’enregistrer un *lignage* pour chaque processus ELT exécuté. Pour un enregistrement donné, le lignage associe cet enregistrement avec le processus ELT exécuté qui a produit les données. Pour chaque exécution du processus ETL, un enregistrement de lignage est créé pour chaque table, montrant les heures de début et de fin du chargement. Les clés de lignage pour chaque enregistrement sont stockées dans les tables de dimension et de faits.

![Capture d’écran de la table de dimension des villes](./images/city-dimension-table.png)

Lorsqu’un nouveau lot de données est chargé dans l’entrepôt de données, actualisez le modèle Analysis Services tabulaire. Voir [Actualisation asynchrone avec l’API REST](/azure/analysis-services/analysis-services-async-refresh).

## <a name="data-cleansing"></a>Nettoyage des données

Le nettoyage des données doit faire partie du processus ELT. Dans cette architecture de référence, la table incluant la population urbaine fournit des données incorrectes, car certaines villes sont associées à une population égale à zéro, sans doute parce qu’aucune donnée n’est disponible. Lors du traitement, le pipeline ELT supprime ces villes de la table incluant la population urbaine. Effectuez le nettoyage des données sur les tables de mise en lots plutôt que sur les tables externes.

Voici la procédure stockée qui supprime les villes dont la population est égale à zéro de la table incluant la population urbaine. (Vous trouverez le fichier source [ici](https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/citypopulation/%5BIntegration%5D.%5BMigrateExternalCityPopulationData%5D.sql).)

```sql
DELETE FROM [Integration].[CityPopulation_Staging]
WHERE RowNumber in (SELECT DISTINCT RowNumber
FROM [Integration].[CityPopulation_Staging]
WHERE POPULATION = 0
GROUP BY RowNumber
HAVING COUNT(RowNumber) = 4)
```

## <a name="external-data-sources"></a>Sources de données externes

Souvent, les entrepôts de données consolident les données provenant de plusieurs sources. Cette architecture de référence charge une source de données externe qui contient des données démographiques. Ce jeu de données est disponible dans le stockage d’objets blob Azure associé à l’exemple [WorldWideImportersDW](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/wide-world-importers/sample-scripts/polybase).

Azure Data Factory peut copier directement les données depuis le stockage d’objets blob, à l’aide du [connecteur de stockage d’objets blob](/azure/data-factory/connector-azure-blob-storage). Toutefois, ce connecteur nécessite une chaîne de connexion ou d’une signature d’accès partagé, afin qu’elle ne puisse être utilisée pour copier un objet blob avec un accès en lecture public. Pour résoudre ce problème, vous pouvez utiliser PolyBase pour créer une table externe sur le stockage d’objets blob, puis copier les tables externes dans SQL Data Warehouse.

## <a name="handling-large-binary-data"></a>Gestion de données binaires volumineuses

Dans la base de données source, la table Cities inclut une colonne Location qui présente un type de données spatiales [geography](/sql/t-sql/spatial-geography/spatial-types-geography). SQL Data Warehouse ne prend pas en charge le type **geography** en mode natif. Ce champ est donc converti en type **varbinary** pendant le chargement. (Voir [Utilisation de solutions de contournement pour les types de données non pris en charge](/azure/sql-data-warehouse/sql-data-warehouse-tables-data-types#unsupported-data-types).)

Toutefois, PolyBase prend en charge une taille de colonne maximale de `varbinary(8000)`, ce qui signifie que certaines données peuvent être tronquées. Pour résoudre ce problème, vous pouvez scinder les données en segments pendant l’exportation, puis les réassembler, comme suit :

1. Créez une table de mise en lots temporaire pour la colonne Location.

2. Pour chaque ville, fractionnez les données de localisation en segments de 8 000 octets, ce qui entraîne la création de 1 &ndash; N lignes pour chaque ville.

3. Pour les réassembler, convertissez les lignes en colonnes avec l’opérateur T-SQL [PIVOT](/sql/t-sql/queries/from-using-pivot-and-unpivot), puis concaténez les valeurs des colonnes pour chaque ville.

La difficulté, c’est que chaque ville peut être scindée en un nombre de lignes spécifique, selon la taille des données géographiques. Pour que l’opérateur PIVOT fonctionne, chaque ville doit présenter le même nombre de lignes. À cette fin, la requête T-SQL (que vous pouvez afficher [ici][MergeLocation]) s’efforce de remplir les lignes avec des valeurs vides, afin que chaque ville présente le même nombre de colonnes après l’exécution de l’opérateur PIVOT. La requête obtenue s’avère beaucoup plus rapide qu’un bouclage dans les lignes, au cas par cas.

La même approche est utilisée pour les données d’image.

## <a name="slowly-changing-dimensions"></a>Dimensions à variation lente

Les données de dimension sont relativement statiques, mais elle peuvent changer. Par exemple, un produit peut être réaffecté à une autre catégorie. Il existe plusieurs approches permettant de gérer les dimensions à variation lente. Une technique courante, appelée [Type 2](https://wikipedia.org/wiki/Slowly_changing_dimension#Type_2:_add_new_row), consiste à ajouter un nouvel enregistrement chaque fois qu’une dimension est modifiée.

Pour implémenter l’approche Type 2, les tables de dimension requièrent des colonnes supplémentaires, qui spécifient la plage de dates effective d’un enregistrement donné. En outre, les clés primaires de la base de données source sont dupliquées, donc la table de dimension doit avoir une clé primaire artificielle.

L’illustration suivante représente la table Dimension.City. La colonne `WWI City ID` est la clé primaire provenant de la base de données source. La colonne `City Key` est une clé artificielle générée lors du pipeline ETL. Notez également que la table inclut des colonnes `Valid From` et `Valid To`, qui définissent la plage lorsque chaque ligne était valide. Les valeurs actuelles incluent un paramètre `Valid To` égal à « 9999-12-31 ».

![Capture d’écran de la table de dimension des villes](./images/city-dimension-table.png)

L’avantage de cette approche est qu’elle conserve les données d’historique, qui peuvent être utiles pour l’analyse. Toutefois, cela signifie également que la même entité sera associée à plusieurs lignes. Par exemple, voici les enregistrements qui correspondent à la valeur `WWI City ID` = 28561 :

![Deuxième capture d’écran de la table de dimension des villes](./images/city-dimension-table-2.png)

Associez chaque fait de la table Sales avec une ligne unique de la table de dimension City correspondant à la date de la facture. Dans le cadre du processus ETL, créez une colonne supplémentaire. 

La requête T-SQL suivante crée une table temporaire qui associe chaque facture à la clé City Key correcte de la table de dimension City.

```sql
CREATE TABLE CityHolder
WITH (HEAP , DISTRIBUTION = HASH([WWI Invoice ID]))
AS
SELECT DISTINCT s1.[WWI Invoice ID] AS [WWI Invoice ID],
                c.[City Key] AS [City Key]
    FROM [Integration].[Sale_Staging] s1
    CROSS APPLY (
                SELECT TOP 1 [City Key]
                    FROM [Dimension].[City]
                WHERE [WWI City ID] = s1.[WWI City ID]
                    AND s1.[Last Modified When] > [Valid From]
                    AND s1.[Last Modified When] <= [Valid To]
                ORDER BY [Valid From], [City Key] DESC
                ) c

```

Cette table est utilisée pour remplir une colonne dans la table de faits Sales :

```sql
UPDATE [Integration].[Sale_Staging]
SET [Integration].[Sale_Staging].[WWI Customer ID] =  CustomerHolder.[WWI Customer ID]
```

Cette colonne permet à une requête Power BI de rechercher l’enregistrement City correct pour une facture client donnée.

## <a name="security-considerations"></a>Considérations relatives à la sécurité

Pour renforcer la sécurité, vous pouvez utiliser des [points de terminaison de service réseau virtuel](/azure/virtual-network/virtual-network-service-endpoints-overview) pour sécuriser les ressources du service Azure à votre réseau virtuel. Cela supprime complètement tout accès Internet public à ces ressources, en autorisant le trafic uniquement à partir de votre réseau virtuel.

Grâce à cette approche, vous créez un réseau virtuel dans Azure, puis créez des points de terminaison de service privés pour les services Azure. Ces services sont ensuite limités au trafic à partir de ce réseau virtuel. Vous pouvez également y accéder à partir de votre réseau local, via une passerelle.

Notez les limitations suivantes :

- Au moment où cette architecture de référence a été créée, les points de terminaison de service de réseau virtuel étaient pris en charge pour le stockage Azure et Azure SQL Data Warehouse, mais non pour le service Azure Analysis. Vérifiez l’état le plus récent [ici](https://azure.microsoft.com/updates/?product=virtual-network).

- Si les points de terminaison de service sont activés pour le stockage Azure, PolyBase ne peut pas copier des données à partir du stockage vers SQL Data Warehouse. Il existe une atténuation pour ce problème. Pour en savoir plus, voir [Impact de l’utilisation des points de terminaison de service de réseau virtuel avec le stockage Azure](/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview?toc=%2fazure%2fvirtual-network%2ftoc.json#impact-of-using-vnet-service-endpoints-with-azure-storage).

- Pour déplacer des données locales vers le stockage Azure, vous devrez créer une liste verte regroupant les adresses IP publiques de votre système local ou associées à ExpressRoute. Pour en savoir plus, voir [Sécurisation des services Azure pour des réseaux virtuels](/azure/virtual-network/virtual-network-service-endpoints-overview#securing-azure-services-to-virtual-networks).

- Pour permettre à Analysis Services de lire les données à partir de SQL Data Warehouse, déployez une machine virtuelle Windows sur le réseau virtuel qui contient le point de terminaison de service SQL Data Warehouse. Installez la [passerelle de données locale Azure](/azure/analysis-services/analysis-services-gateway) sur cette machine virtuelle. Ensuite, connectez votre service Azure Analysis à la passerelle de données.

## <a name="deploy-the-solution"></a>Déployer la solution

Pour déployer et exécuter l’implémentation de référence, suivez les étapes du [fichier Readme de GitHub][github]. Il déploie les éléments suivants :

- Une machine virtuelle pour simuler un serveur de base de données local. Sont inclus SQL Server 2017 et les outils associés, et Power BI Desktop.
- Un compte de stockage Azure qui fournit le stockage d’objets blob pour conserver des données exportées de la base de données SQL Server.
- Une instance Azure SQL Data Warehouse.
- Une instance Azure Analysis Services.
- Azure Data Factory et le pipeline Data Factory associé au travail ELT.

## <a name="related-resources"></a>Ressources associées

Vous pouvez consulter les [exemples de scénarios Azure](/azure/architecture/example-scenario) suivants, qui décrivent des solutions spécifiques utilisant certaines de ces technologies :

- [Entreposage et analyse des données pour les ventes et le marketing](/azure/architecture/example-scenario/data/data-warehouse)
- [ETL hybride avec des instances SSIS locales existantes et Azure Data Factory](/azure/architecture/example-scenario/data/hybrid-etl-with-adf)

<!-- links -->

[adf]: /azure/data-factory
[github]: https://github.com/mspnp/azure-data-factory-sqldw-elt-pipeline
[MergeLocation]: https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/city/%5BIntegration%5D.%5BMergeLocation%5D.sql
[wwi]: /sql/sample/world-wide-importers/wide-world-importers-oltp-database
