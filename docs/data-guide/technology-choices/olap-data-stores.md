---
title: "Choix d’une banque de données OLAP"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: f3041b95696c9408a2c9ab747fe1ec3041db0743
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-an-olap-data-store-in-azure"></a>Choix d’une banque de données OLAP dans Azure

Le traitement analytique en ligne (OLAP) est une technologie qui organise les bases de données de grandes entreprises et prend en charge les analyses complexes. Cette rubrique compare les options des solutions OLAP dans Azure.

> [!NOTE]
> Pour plus d’informations, sur l’utilisation d’une banque de données OLAP, consultez [Traitement analytique en ligne (OLAP)](../scenarios/online-analytical-processing.md).

## <a name="what-are-your-options-when-choosing-an-olap-data-store"></a>Quelles sont vos options lors du choix d’une banque de données OLAP ?

Dans Azure, toutes les banques de données suivantes répondent aux principales exigences d’OLAP :

- [SQL Server avec index Columnstore](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics)
- [Azure Analysis Services](/azure/analysis-services/analysis-services-overview)
- [SQL Server Analysis Services (SSAS)](/sql/analysis-services/analysis-services)

SQL Server Analysis Services (SSAS) offre la fonctionnalité OLAP et l’exploration de données pour les applications intelligentes métier. Vous pouvez installer SSAS sur des serveurs locaux ou l’héberger dans une machine virtuelle dans Azure. Azure Analysis Services est un service entièrement géré qui fournit les mêmes fonctionnalités principales que SSAS. Azure Analysis Services prend en charge la connexion aux [différentes sources de données](/azure/analysis-services/analysis-services-datasource)dans le cloud et localement dans votre organisation.

Les index Columnstore en cluster sont disponibles dans SQL Server 2014 et versions ultérieures, ainsi que de la base de données SQL Azure et sont idéales pour les charges de travail OLAP. Toutefois, à partir de SQL Server 2016 (y compris SQL Database Azure), vous pouvez bénéficier du traitement hybride transactionnel/analytique (HTAP) via l’utilisation des index columnstore non cluster actualisables. HTAP vous permet d’effectuer le traitement OLTP et OLAP sur la même plateforme, ce qui supprime la nécessité de stocker plusieurs copies de vos données et élimine la nécessité de systèmes OLTP et OLAP distincts. Pour plus d’informations, consultez [Prise en main de Columnstore pour l’analytique opérationnelle en temps réel](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics).

## <a name="key-selection-criteria"></a>Critères de sélection principaux

Pour restreindre les choix, commencez par répondre aux questions suivantes :

- Préférez-vous un service géré plutôt que de gérer vos propres serveurs ?

- Avez-vous besoin de l’authentification sécurisée à l’aide de Azure Active Directory (Azure AD) ?

- Souhaitez-vous effectuer l’analytique en temps réel ? Dans ce cas, limitez vos options à celles qui prennent en charge d’analytique en temps réel. 

    *Analytique en temps réel* dans ce contexte s’applique à une source de données unique, par exemple une application Enterprise Resource Planning (ERP), qui exécute à la fois une charge de travail opérationnel et analytique. Si vous avez besoin d’intégrer des données provenant de plusieurs sources ou nécessitez des performances d’analytique extrêmes à l’aide de données préagrégées telles que des cubes, vous pouvez toujours avoir besoin d’un entrepôt de données distinct.

- Devez-vous utiliser des données préagrégées, par exemple, pour fournir des modèles sémantiques qui rendent l’analytique plus conviviale pour les entreprises ? Si tel est le cas, choisissez une option qui prend en charge des cubes multidimensionnels ou des modèles sémantiques tabulaires. 

    Fournir des agrégats permet aux utilisateurs de calculer de manière cohérente des agrégats de données. Des données préagrégées peuvent également fournir un sérieux renforcement des performances lors du traitement de plusieurs colonnes sur plusieurs lignes. Les données peuvent être préagrégées dans des cubes multidimensionnels ou des modèles sémantiques tabulaires.

- Devez-vous intégrer des données provenant de plusieurs sources, au-delà de votre banque de données OLTP ? Dans ce cas, envisagez les options qui s’intègrent facilement à plusieurs sources de données.

## <a name="capability-matrix"></a>Matrice des fonctionnalités

Les tableaux suivants résument les principales différences entre les fonctionnalités.

### <a name="general-capabilities"></a>Fonctionnalités générales

| | Azure Analysis Services | SQL Server Analysis Services | SQL Server avec index Columnstore | SQL Database Azure avec des index Columnstore |
| --- | --- | --- | --- | --- |
| Est un service géré | OUI | Non  | Non  | OUI |
| Prend en charge des cubes multidimensionnels | Non  | OUI | Non  | Non  |
| Prend en charge les modèles sémantiques tabulaires | OUI | OUI | Non  | Non  |
| Intégrer facilement plusieurs sources de données | OUI | OUI | Nonn <sup>1</sup> | Nonn <sup>1</sup> |
| Prend en charge l’analytique en temps réel | Non  | Non  | OUI | OUI |
| Exige un processus pour copier des données à partir de sources | OUI | OUI | Non  | Non  |
| Intégration Azure AD | OUI | Non  | Non <sup>2</sup> | OUI |

[1] bien que SQL Server et SQL Database Azure ne peuvent pas être utilisés pour interroger et intégrer plusieurs sources de données externes, vous pouvez toujours créer un pipeline qui effectue cette opération à l’aide de [SSIS](/sql/integration-services/sql-server-integration-services) ou [Azure Data Factory](/azure/data-factory/). SQL Server hébergé dans une machine virtuelle Azure, propose des options supplémentaires, telles que les serveurs liés et [PolyBase](/sql/relational-databases/polybase/polybase-guide). Pour plus d’informations, consultez [Orchestration, flux de contrôle et déplacement des données de pipeline](../technology-choices/pipeline-orchestration-data-movement.md).

[2] La connexion à SQL Server s’exécutant sur une machine virtuelle Azure n’est pas prise en charge à l’aide d’un compte Azure AD. Utilisez plutôt un compte Active Directory du domaine.

### <a name="scalability-capabilities"></a>Fonctionnalités d’extensibilité

| | Azure Analysis Services | SQL Server Analysis Services | SQL Server avec index Columnstore | SQL Database Azure avec des index Columnstore |
| --- | --- | --- | --- | --- |
| Serveurs régionaux redondants pour une haute disponibilité  | OUI | Non  | OUI | OUI |
| Prend en charge l’augmentation de la taille des instances de la requête  | OUI | Non  | OUI | Non  |
| Évolutivité dynamique (monter en puissance)  | OUI | Non  | OUI | Non  |

