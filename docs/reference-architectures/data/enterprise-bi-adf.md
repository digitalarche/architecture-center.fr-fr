---
title: BI d’entreprise automatisée avec SQL Data Warehouse et Azure Data Factory
description: Automatiser un flux de travail ELT sur Azure à l’aide d’Azure Data Factory
author: MikeWasson
ms.date: 07/01/2018
ms.openlocfilehash: ffd75ba8c57a9afbc6abad61f21f738c644c9bc8
ms.sourcegitcommit: 58d93e7ac9a6d44d5668a187a6827d7cd4f5a34d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/02/2018
ms.locfileid: "37142271"
---
# <a name="automated-enterprise-bi-with-sql-data-warehouse-and-azure-data-factory"></a>BI d’entreprise automatisée avec SQL Data Warehouse et Azure Data Factory

Cette architecture de référence indique comment effectuer un chargement incrémentiel dans un pipeline [ELT](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt) (Extract-Load-Transform). Elle utilise Azure Data Factory pour automatiser le pipeline ELT. Ce pipeline déplace de façon incrémentielle les données OLTP les plus récentes d’une base de données SQL Server locale vers une instance SQL Data Warehouse. Les données transactionnelles sont transformées en un modèle tabulaire à des fins d’analyse. [**Déployez cette solution**.](#deploy-the-solution)

![](./images/enterprise-bi-sqldw-adf.png)

Cette architecture repose sur celle qui est décrite dans la section [Enterprise BI avec SQL Data Warehouse](./enterprise-bi-sqldw.md), mais ajoute des fonctionnalités qui sont importantes pour les scénarios d’entreposage de données d’entreprise.

-   Automatisation du pipeline à l’aide de Data Factory.
-   Chargement incrémentiel.
-   Intégration de plusieurs sources de données.
-   Chargement des données binaires comme les données géospatiales et les images.

## <a name="architecture"></a>Architecture

L’architecture est constituée des composants suivants.

### <a name="data-sources"></a>Sources de données

**Serveur SQL Server local**. Les données source sont situées dans une base de données SQL Server locale. Pour simuler l’environnement local, les scripts de déploiement de cette architecture approvisionnent une machine virtuelle dans Azure disposant de SQL Server. [L’exemple de base de données OLTP Wide World Importers] [wwi] est utilisé comme base de données source.

**Données externes**. Un scénario courant de gestion des entrepôts de données consiste à intégrer plusieurs sources de données. Cette architecture de référence charge un jeu de données externe qui contient la population par ville et par année, et l’intègre dans les données de la base de données OLTP. Vous pouvez utiliser ces données pour répondre à des questions du type : « La croissance des ventes dans chaque région correspond-elle, voire dépasse-t-elle la croissance de la population ? »

### <a name="ingestion-and-data-storage"></a>Ingestion et stockage de données

**Stockage d'objets blob**. Le stockage d’objets blob est utilisé en tant que zone de processus de site pour les données sources, avant leur chargement dans SQL Data Warehouse.

**Azure SQL Data Warehouse**. [SQL Data Warehouse](/azure/sql-data-warehouse/) est un système distribué conçu pour réaliser des analyses sur de grandes quantités de données. Il prend en charge le traitement MPP (Massive Parallel Processing), le rendant ainsi adapté à l’exécution d’analyses hautes performances. 

**Azure Data Factory**. [Data Factory] [adf] est un service géré qui orchestre et automatise le déplacement et la transformation des données. Dans cette architecture, il coordonne les différentes étapes du processus ELT.

### <a name="analysis-and-reporting"></a>Analyse et rapports

**Azure Analysis Services**. [Analysis Services](/azure/analysis-services/) est un service entièrement géré qui fournit des capacités de modélisation des données. Le modèle sémantique est chargé dans Analysis Services.

**Power BI**. Power BI est une suite d’outils d’analyse métier pour analyser les données et obtenir des informations métier. Dans cette architecture, il demande le modèle sémantique stockée dans Analysis Services.

### <a name="authentication"></a>Authentification

**Azure Active Directory** (Azure AD) authentifie les utilisateurs qui se connectent au serveur Analysis Services via Power BI.

Data Factory peut également utiliser Azure AD pour s’authentifier auprès de SQL Data Warehouse, en utilisant un principal de service ou une identité MSI (Managed Service Identity). Par souci de simplicité, l’exemple de déploiement utilise l’authentification SQL Server.

## <a name="data-pipeline"></a>Pipeline de données

Dans [Azure Data Factory] [adf], un pipeline est un regroupement logique d’activités qui permet de coordonner une tâche. Dans ce cas, il s’agit du chargement et de la transformation des données dans SQL Data Warehouse. 

Cette architecture de référence définit un pipeline principal qui exécute une suite de pipelines enfants. Chaque pipeline enfant charge les données dans une ou plusieurs tables d’entrepôt de données.

![](./images/adf-pipeline.png)

## <a name="incremental-loading"></a>Chargement incrémentiel

Lorsque vous exécutez un processus ETL ou ELT automatisé, il s’avère plus efficace de charger uniquement les données modifiées depuis l’exécution précédente. Il s’agit d’un *chargement incrémentiel*, par opposition à un chargement complet, qui porte sur toutes les données. Pour effectuer un chargement incrémentiel, vous avez besoin d’une méthode d’identification des données modifiées. L’approche la plus courante consiste à utiliser une valeur *limite supérieure*, qui se traduit par le suivi de la valeur la plus récente d’une colonne de la table source, soit une colonne DateHeure, soit une colonne d’entier unique. 

Depuis SQL Server 2016, vous pouvez utiliser des [tables temporelles](/sql/relational-databases/tables/temporal-tables). Il s’agit de tables de système par version qui conservent un historique complet des modifications apportées aux données. Le moteur de base de données enregistre automatiquement l’historique de chaque modification dans une table d’historique distincte. Vous pouvez interroger les données d’historique en ajoutant une clause FOR SYSTEM_TIME à une requête. En interne, le moteur de base de données interroge la table d’historique, mais cette opération est transparente pour l’application. 

> [!NOTE]
> Pour les versions antérieures de SQL Server, vous pouvez utiliser la fonction [Capture des changements de données](/sql/relational-databases/track-changes/about-change-data-capture-sql-server) (CDC). Cette approche est moins pratique que les tables temporelles, car vous devez interroger une table de modifications distincte, et les modifications font l’objet d’un suivi par numéro séquentiel dans le journal plutôt que par horodatage. 

Les tables temporelles sont utiles pour les données de dimension, qui peuvent changer au fil du temps. Les tables de faits représentent généralement une transaction immuable, comme une vente. De ce fait, il n’est pas utile de conserver l’historique des versions du système. Au lieu de cela, les transactions incluent généralement une colonne qui représente la date de transaction, qui peut être utilisée en tant que la valeur de filigrane. Par exemple, dans la base de données OLTP Wide World Importers, les tables Sales.Invoices et Sales.InvoiceLines incluent un champ `LastEditedWhen` dont la valeur par défaut est `sysdatetime()`. 

Voici le flux général du pipeline ELT :

1. Pour chaque table de la base de données source, effectuez le suivi de l’heure de coupure lors de l’exécution du dernier travail ELT. Stockez ces informations dans l’entrepôt de données. (Lors de la configuration initiale, toutes les heures sont définies sur « 1-1-1900 ».)

2. Lors de l’étape d’exportation des données, l’heure de coupure est transmise sous la forme d’un paramètre à un ensemble de procédures stockées dans la base de données source. Ces procédures stockées demandent au système tous les enregistrements ayant été modifiés ou créés après l’heure de coupure. Pour la table de faits Sales, la colonne `LastEditedWhen` est utilisée. Dans le cas des données de dimension, les tables temporelles de système par version sont utilisées.

3. Lorsque la migration des données est terminée, mettez à jour la table qui stocke les heures de coupure.

Il est également utile d’enregistrer un *lignage* pour chaque processus ELT exécuté. Pour un enregistrement donné, le lignage associe cet enregistrement avec le processus ELT exécuté qui a produit les données. Pour chaque exécution du processus ETL, un enregistrement de lignage est créé pour chaque table, montrant les heures de début et de fin du chargement. Les clés de lignage pour chaque enregistrement sont stockées dans les tables de dimension et de faits.

![](./images/city-dimension-table.png)

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

![](./images/city-dimension-table.png)

L’avantage de cette approche est qu’elle conserve les données d’historique, qui peuvent être utiles pour l’analyse. Toutefois, cela signifie également que la même entité sera associée à plusieurs lignes. Par exemple, voici les enregistrements qui correspondent à la valeur `WWI City ID` = 28561 :

![](./images/city-dimension-table-2.png)

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

Soyez conscient des limitations suivantes :

- Au moment où cette architecture de référence a été créée, les points de terminaison de service de réseau virtuel étaient pris en charge pour le stockage Azure et Azure SQL Data Warehouse, mais non pour le service Azure Analysis. Vérifiez l’état le plus récent [ici](https://azure.microsoft.com/updates/?product=virtual-network). 

- Si les points de terminaison de service sont activés pour le stockage Azure, PolyBase ne peut pas copier des données à partir du stockage vers SQL Data Warehouse. Il existe une atténuation pour ce problème. Pour en savoir plus, voir [Impact de l’utilisation des points de terminaison de service de réseau virtuel avec le stockage Azure](/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview?toc=%2fazure%2fvirtual-network%2ftoc.json#impact-of-using-vnet-service-endpoints-with-azure-storage). 

- Pour déplacer des données locales vers le stockage Azure, vous devrez créer une liste verte regroupant les adresses IP publiques de votre système local ou associées à ExpressRoute. Pour en savoir plus, voir [Sécurisation des services Azure pour des réseaux virtuels](/azure/virtual-network/virtual-network-service-endpoints-overview#securing-azure-services-to-virtual-networks).

- Pour permettre à Analysis Services de lire les données à partir de SQL Data Warehouse, déployez une machine virtuelle Windows sur le réseau virtuel qui contient le point de terminaison de service SQL Data Warehouse. Installez la [passerelle de données locale Azure](/azure/analysis-services/analysis-services-gateway) sur cette machine virtuelle. Ensuite, connectez votre service Azure Analysis à la passerelle de données.

## <a name="deploy-the-solution"></a>Déployer la solution

Un déploiement de cette architecture de référence est disponible sur [GitHub][ref-arch-repo-folder]. Il déploie les éléments suivants :

  * Une machine virtuelle pour simuler un serveur de base de données local. Sont inclus SQL Server 2017 et les outils associés, et Power BI Desktop.
  * Un compte de stockage Azure qui fournit le stockage d’objets blob pour conserver des données exportées de la base de données SQL Server.
  * Une instance Azure SQL Data Warehouse.
  * Une instance Azure Analysis Services.
  * Azure Data Factory et le pipeline Data Factory associé au travail ELT.

### <a name="prerequisites"></a>Prérequis

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="variables"></a>variables

Les étapes qui suivent incluent des variables définies par l’utilisateur. Vous devez les remplacer par les valeurs que vous définissez.

- `<data_factory_name>`. Nom de l’instance Data Factory.
- `<analysis_server_name>`. Nom du serveur Analysis Services.
- `<active_directory_upn>`. Votre nom d’utilisateur principal (UPN) Azure Active Directory. Par exemple : `user@contoso.com`.
- `<data_warehouse_server_name>`. Nom du serveur SQL Data Warehouse.
- `<data_warehouse_password>`. Mot de passe administrateur de SQL Data Warehouse.
- `<resource_group_name>`. Nom du groupe de ressources.
- `<region>`. Région Azure dans laquelle les ressources seront déployées.
- `<storage_account_name>`. Nom du compte de stockage. Il doit respecter les [règles de dénomination](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) des comptes de stockage.
- `<sql-db-password>`. Mot de passe de connexion SQL Server.

### <a name="deploy-azure-data-factory"></a>Déployer Azure Data Factory

1. Accédez au dossier `data\enterprise_bi_sqldw_advanced\azure\templates` du [référentiel GitHub][ref-arch-repo].

2. Exécutez la commande Azure CLI suivante pour créer un groupe de ressources.  

    ```bash
    az group create --name <resource_group_name> --location <region>  
    ```

    Spécifiez une région qui prend en charge SQL Data Warehouse, Azure Analysis Services et Data Factory version 2. Voir [Produits Azure par région](https://azure.microsoft.com/global-infrastructure/services/).

3. Exécutez la commande suivante

    ```
    az group deployment create --resource-group <resource_group_name> \
        --template-file adf-create-deploy.json \
        --parameters factoryName=<data_factory_name> location=<location>
    ```

Ensuite, utilisez le portail Azure pour obtenir la clé d’authentification du [runtime d’intégration](/azure/data-factory/concepts-integration-runtime) Azure Data Factory, comme suit :

1. Dans le [portail Azure](https://portal.azure.com/), accédez à l’instance Data Factory.

2. Dans le panneau Data Factory, cliquez sur **Créer et surveiller**. Le portail Azure Data Factory s’ouvre dans une autre fenêtre du navigateur.

    ![](./images/adf-blade.png)

3. Dans le portail Azure Data Factory, sélectionnez l’icône de crayon (« Author »). 

4. Cliquez sur **Connexions**, puis sélectionnez **Runtimes d’intégration**.

5. Sous **sourceIntegrationRuntime**, cliquez sur l’icône de crayon (« Modifier »).

    > [!NOTE]
    > Le portail affiche l’état « Non disponible ». Ce comportement est attendu tant que vous n’avez pas déployé le serveur local.

6. Recherchez le paramètre **Key1** et copiez la valeur de la clé d’authentification.

Il vous faut la clé d’authentification pour l’étape suivante.

### <a name="deploy-the-simulated-on-premises-server"></a>Déployer le serveur local simulé

Cette étape déploie une machine virtuelle en tant que serveur local simulé, qui inclut SQL Server 2017 et les outils associés. Elle charge également [l’exemple de base de données OLTP Wide World Importers][wwi] dans SQL Server.

1. Accédez au dossier `data\enterprise_bi_sqldw_advanced\onprem\templates` du référentiel.

2. Dans le fichier `onprem.parameters.json`, recherchez `adminPassword`. Il s’agit du mot de passe vous permettant de vous connecter à la machine virtuelle SQL Server. Remplacez la valeur par un autre mot de passe.

3. Dans le même fichier, recherchez l’élément `SqlUserCredentials`. Cette propriété spécifie les informations d’identification du compte SQL Server. Remplacez le mot de passe par une autre valeur.

4. Dans le même fichier, collez la clé d’authentification du runtime d’intégration dans le paramètre `IntegrationRuntimeGatewayKey`, comme indiqué ci-dessous :

    ```json
    "protectedSettings": {
        "configurationArguments": {
            "SqlUserCredentials": {
                "userName": ".\\adminUser",
                "password": "<sql-db-password>"
            },
            "IntegrationRuntimeGatewayKey": "<authentication key>"
        }
    ```

5. Exécutez la commande ci-dessous.

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.parameters.json --deploy
    ```

Cette étape peut prendre 20 à 30 minutes. Elle inclut l’exécution d’un script [DSC](/powershell/dsc/overview) pour installer les outils et restaurer la base de données. 

### <a name="deploy-azure-resources"></a>Déployer des ressources Azure

Cette étape déploie SQL Data Warehouse, Azure Analysis Services et Data Factory.

1. Accédez au dossier `data\enterprise_bi_sqldw_advanced\azure\templates` du [référentiel GitHub][ref-arch-repo].

2. Exécutez la commande d’interface de ligne de commande Azure suivante. Remplacez les valeurs de paramètres affichées entre crochets.

    ```bash
    az group deployment create --resource-group <resource_group_name> \
     --template-file azure-resources-deploy.json \
     --parameters "dwServerName"="<data_warehouse_server_name>" \
     "dwAdminLogin"="adminuser" "dwAdminPassword"="<data_warehouse_password>" \ 
     "storageAccountName"="<storage_account_name>" \
     "analysisServerName"="<analysis_server_name>" \
     "analysisServerAdmin"="<user@contoso.com>"
    ```

    - Le paramètre `storageAccountName` doit respecter les [règles d’attribution de noms](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) des comptes de stockage. 
    - Pour le paramètre `analysisServerAdmin`, utilisez votre nom d’utilisateur principal (UPN) Azure Active Directory.

3. Exécutez la commande Azure CLI suivante pour obtenir la clé d’accès du compte de stockage. Vous aurez besoin de cette clé lors de l’étape suivante.

    ```bash
    az storage account keys list -n <storage_account_name> -g <resource_group_name> --query [0].value
    ```

4. Exécutez la commande d’interface de ligne de commande Azure suivante. Remplacez les valeurs de paramètres affichées entre crochets. 

    ```bash
    az group deployment create --resource-group <resource_group_name> \
    --template-file adf-pipeline-deploy.json \
    --parameters "factoryName"="<data_factory_name>" \
    "sinkDWConnectionString"="Server=tcp:<data_warehouse_server_name>.database.windows.net,1433;Initial Catalog=wwi;Persist Security Info=False;User ID=adminuser;Password=<data_warehouse_password>;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;" \
    "blobConnectionString"="DefaultEndpointsProtocol=https;AccountName=<storage_account_name>;AccountKey=<storage_account_key>;EndpointSuffix=core.windows.net" \
    "sourceDBConnectionString"="Server=sql1;Database=WideWorldImporters;User Id=adminuser;Password=<sql-db-password>;Trusted_Connection=True;"
    ```

    Les chaînes de connexion incluent des sous-chaînes affichées entre crochets, qui doivent être remplacées. Pour le paramètre `<storage_account_key>`, utilisez la clé que vous avez obtenue à l’étape précédente. Pour l’élément `<sql-db-password>`, utilisez le mot de passe du compte SQL Server que vous avez précédemment spécifié dans le fichier `onprem.parameters.json`.

### <a name="run-the-data-warehouse-scripts"></a>Exécuter les scripts d’entrepôt de données

1. Dans le [portail Azure](https://portal.azure.com/), recherchez la machine virtuelle locale, appelée `sql-vm1`. Le nom d’utilisateur et le mot de passe de la machine virtuelle sont spécifiés dans le fichier `onprem.parameters.json`.

2. Cliquez sur **Connect** et utilisez le Bureau à distance pour vous connecter à la machine virtuelle.

3. À partir de votre session Bureau à distance, ouvrez une invite de commandes et accédez au dossier suivant sur la machine virtuelle :

    ```
    cd C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw_advanced\azure\sqldw_scripts
    ```

4. Exécutez la commande suivante :

    ```
    deploy_database.cmd -S <data_warehouse_server_name>.database.windows.net -d wwi -U adminuser -P <data_warehouse_password> -N -I
    ```

    Pour les éléments `<data_warehouse_server_name>` et `<data_warehouse_password>`, utilisez le nom et le mot de passe du serveur de l’entrepôt de données précédemment utilisés.

Vous pouvez utiliser SQL Server Management Studio (SSMS) pour vous connecter à une base de données Azure SQL Data Warehouse afin de vérifier cette étape. Vous devez voir apparaître les schémas de table de base de données.

### <a name="run-the-data-factory-pipeline"></a>Exécuter le pipeline Data Factory

1. À partir de la même session Bureau à distance, ouvrez une fenêtre PowerShell.

2. Exécutez la commande PowerShell suivante. Cliquez sur **Oui** lorsque vous y êtes invité.

    ```powershell
    Install-Module -Name AzureRM -AllowClobber
    ```

3. Exécutez la commande PowerShell suivante. À l’invite, entrez vos informations d’identification Azure.

    ```powershell
    Connect-AzureRmAccount 
    ```

4. Exécutez les commandes PowerShell suivantes. Remplacez les valeurs figurant entre crochets.

    ```powershell
    Set-AzureRmContext -SubscriptionId <subscription id>

    Invoke-AzureRmDataFactoryV2Pipeline -DataFactory <data-factory-name> -PipelineName "MasterPipeline" -ResourceGroupName <resource_group_name>

5. In the Azure Portal, navigate to the Data Factory instance that was created earlier.

6. In the Data Factory blade, click **Author & Monitor**. This opens the Azure Data Factory portal in another browser window.

    ![](./images/adf-blade.png)

7. In the Azure Data Factory portal, click the **Monitor** icon. 

8. Verify that the pipeline completes successfully. It can take a few minutes.

    ![](./images/adf-pipeline-progress.png)


## Build the Analysis Services model

In this step, you will create a tabular model that imports data from the data warehouse. Then you will deploy the model to Azure Analysis Services.

**Create a new tabular project**

1. From your Remote Desktop session, launch SQL Server Data Tools 2015.

2. Select **File** > **New** > **Project**.

3. In the **New Project** dialog, under **Templates**, select  **Business Intelligence** > **Analysis Services** > **Analysis Services Tabular Project**. 

4. Name the project and click **OK**.

5. In the **Tabular model designer** dialog, select **Integrated workspace**  and set **Compatibility level** to `SQL Server 2017 / Azure Analysis Services (1400)`. 

6. Click **OK**.


**Import data**

1. In the **Tabular Model Explorer** window, right-click the project and select **Import from Data Source**.

2. Select **Azure SQL Data Warehouse** and click **Connect**.

3. For **Server**, enter the fully qualified name of your Azure SQL Data Warehouse server. You can get this value from the Azure Portal. For **Database**, enter `wwi`. Click **OK**.

4. In the next dialog, choose **Database** authentication and enter your Azure SQL Data Warehouse user name and password, and click **OK**.

5. In the **Navigator** dialog, select the checkboxes for the **Fact.\*** and **Dimension.\*** tables.

    ![](./images/analysis-services-import-2.png)

6. Click **Load**. When processing is complete, click **Close**. You should now see a tabular view of the data.

**Create measures**

1. In the model designer, select the **Fact Sale** table.

2. Click a cell in the the measure grid. By default, the measure grid is displayed below the table. 

    ![](./images/tabular-model-measures.png)

3. In the formula bar, enter the following and press ENTER:

    ```
    Total des ventes :=SUM('Fact Sale'[total incluant les taxes])
    ```

4. Repeat these steps to create the following measures:

    ```
    Nombre d’années =(MAX('Fact CityPopulation'[NumAnnée])-MIN('Fact CityPopulation'[NumAnnée]))+1
    
    Population de départ :=CALCULATE(SUM('Fact CityPopulation'[Population]),FILTER('Fact CityPopulation','Fact CityPopulation'[NumAnnée]=MIN('Fact CityPopulation'[NumAnnée])))
    
    Population finale :=CALCULATE(SUM('Fact CityPopulation'[Population]),FILTER('Fact CityPopulation','Fact CityPopulation'[NumAnnée]=MIN('Fact CityPopulation'[NumAnnée])))
    
    Croissance :=IFERROR((([Population finale]/[Population de départ])^(1/[Nombre d’années]))-1,0)
    ```

    ![](./images/analysis-services-measures.png)

For more information about creating measures in SQL Server Data Tools, see [Measures](/sql/analysis-services/tabular-models/measures-ssas-tabular).

**Create relationships**

1. In the **Tabular Model Explorer** window, right-click the project and select **Model View** > **Diagram View**.

2. Drag the **[Fact Sale].[City Key]** field to the **[Dimension City].[City Key]** field to create a relationship.  

3. Drag the **[Face CityPopulation].[City Key]** field to the **[Dimension City].[City Key]** field.  

    ![](./images/analysis-services-relations-2.png)

**Deploy the model**

1. From the **File** menu, choose **Save All**.

2. In **Solution Explorer**, right-click the project and select **Properties**. 

3. Under **Server**, enter the URL of your Azure Analysis Services instance. You can get this value from the Azure Portal. In the portal, select the Analysis Services resource, click the Overview pane, and look for the **Server Name** property. It will be similar to `asazure://westus.asazure.windows.net/contoso`. Click **OK**.

    ![](./images/analysis-services-properties.png)

4. In **Solution Explorer**, right-click the project and select **Deploy**. Sign into Azure if prompted. When processing is complete, click **Close**.

5. In the Azure portal, view the details for your Azure Analysis Services instance. Verify that your model appears in the list of models.

    ![](./images/analysis-services-models.png)

## Analyze the data in Power BI Desktop

In this step, you will use Power BI to create a report from the data in Analysis Services.

1. From your Remote Desktop session, launch Power BI Desktop.

2. In the Welcome Scren, click **Get Data**.

3. Select **Azure** > **Azure Analysis Services database**. Click **Connect**

    ![](./images/power-bi-get-data.png)

4. Enter the URL of your Analysis Services instance, then click **OK**. Sign into Azure if prompted.

5. In the **Navigator** dialog, expand the tabular project, select the model, and click **OK**.

2. In the **Visualizations** pane, select the **Table** icon. In the Report view, resize the visualization to make it larger.

6. In the **Fields** pane, expand **Dimension City**.

7. From **Dimension City**, drag **City** and **State Province** to the **Values** well.

9. In the **Fields** pane, expand **Fact Sale**.

10. From **Fact Sale**, drag **CAGR**, **Ending Population**,  and **Total Sales** to the **Value** well.

11. Under **Visual Level Filters**, select **Ending Population**. Set the filter to "is greater than 100000" and click **Apply filter**.

12. Under **Visual Level Filters**, select **Total Sales**. Set the filter to "is 0" and click **Apply filter**.

![](./images/power-bi-report-2.png)

The table now shows cities with population greater than 100,000 and zero sales. CAGR  stands for Compounded Annual Growth Rate and measures the rate of population growth per city. You could use this value to find cities with high growth rates, for example. However, note that the values for CAGR in the model aren't accurate, because they are derived from sample data.

To learn more about Power BI Desktop, see [Getting started with Power BI Desktop](/power-bi/desktop-getting-started).


[adf]: //azure/data-factory
[azure-cli-2]: //azure/install-azure-cli
[azbb-repo]: https://github.com/mspnp/template-building-blocks
[azbb-wiki]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[MergeLocation]: https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/city/%5BIntegration%5D.%5BMergeLocation%5D.sql
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[ref-arch-repo-folder]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw_advanced
[wwi]: //sql/sample/world-wide-importers/wide-world-importers-oltp-database
