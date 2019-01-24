---
title: Entreposage et analyse des données pour les ventes et le marketing
titleSuffix: Azure Example Scenarios
description: Consolidez les données provenant de plusieurs sources et optimisez l’analyse des données.
author: alexbuckgit
ms.date: 09/15/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: data-analytics
ms.openlocfilehash: ddf9935d534b6902c407c22ba765e8c84558547b
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54488441"
---
# <a name="data-warehousing-and-analytics-for-sales-and-marketing"></a>Entreposage et analyse des données pour les ventes et le marketing

Cet exemple de scénario illustre un pipeline de données qui intègre de grandes quantités de données provenant de plusieurs sources en une plateforme d’analyse unifiée dans Azure. Ce scénario particulier repose sur une solution de vente et de marketing, mais les modèles de conception sont appropriés à de nombreux secteurs nécessitant une analyse avancée des jeux de données volumineux comme ceux de l’e-commerce, de la vente au détail et de la santé.

Cet exemple présente une entreprise de ventes et de marketing qui crée des programmes d’offres incitatives. Ces programmes récompensent les clients, les fournisseurs, les vendeurs et les employés. Les données sont essentielles pour ces programmes, et l’entreprise souhaite améliorer les informations obtenues via l’analyse des données à l’aide d’Azure.

L’entreprise recherche une approche moderne en matière de données d’analyse, afin que les décisions soient prises à l’aide des données appropriées au moment opportun. Les objectifs de l’entreprise sont les suivants :

- combinaison de différents types de sources de données en une plateforme à l’échelle du cloud ;
- transformation des données sources en une structure et une taxonomie communes afin de rendre les données cohérentes et facilement comparables ;
- chargement des données à l’aide d’une approche hautement parallélisée pouvant prendre en charge des milliers de programmes d’offres incitatives, sans les coûts élevés de déploiement et de gestion d’une infrastructure locale ;
- réduction considérable du temps nécessaire pour collecter et transformer des données afin que vous puissiez vous concentrer sur l’analyse des données.

## <a name="relevant-use-cases"></a>Cas d’usage appropriés

Cette approche peut également servir à :

- établir un entrepôt de données pour qu’il soit une source unique et fiable de vos données ;
- intégrer des sources de données relationnelles à d’autres jeux de données non structurées ;
- utiliser de puissants outils de visualisation et de modélisation sémantique pour simplifier l’analyse des données.

## <a name="architecture"></a>Architecture

![Architecture correspondant à un scénario d’analyse et d’entreposage de données dans Azure][architecture]

Les données circulent dans la solution comme suit :

1. Pour chaque source de données, des mises à jour sont exportées régulièrement dans une zone de transit du stockage d’objets blob Azure.
2. Data Factory charge de façon incrémentielle les données provenant du stockage d’objets blob dans des tables de mise en lots de SQL Data Warehouse. Les données sont nettoyées et transformées pendant ce processus. La technologie PolyBase peut paralléliser le processus pour des jeux de données volumineux.
3. À l’issue du chargement d’un nouveau lot de données dans l’entrepôt, un modèle tabulaire Analysis Services créé précédemment est actualisé. Ce modèle sémantique simplifie l’analyse des données d’entreprise et des relations.
4. Les analystes d’entreprise utilisent Microsoft Power BI pour analyser les données en entrepôt via le modèle sémantique Analysis Services.

### <a name="components"></a>Composants

L’entreprise dispose de sources de données sur différentes plateformes :

- Instance SQL Server locale
- Instance Oracle locale
- Azure SQL Database
- Stockage de table Azure
- Cosmos DB

Les données sont chargées à partir de ces différentes sources de données à l’aide de plusieurs composants Azure :

- [Stockage Blob](/azure/storage/blobs/storage-blobs-introduction) est utilisé pour organiser les données sources avant leur chargement dans SQL Data Warehouse.
- [Data Factory](/azure/data-factory) orchestre la transformation des données mises en lots en une structure commune dans SQL Data Warehouse. Data Factory [utilise PolyBase lors du chargement des données dans SQL Data Warehouse](/azure/data-factory/connector-azure-sql-data-warehouse#use-polybase-to-load-data-into-azure-sql-data-warehouse) pour optimiser le débit.
- [SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is) est un système distribué permettant de stocker et d’analyser des jeux de données volumineux. Son recours à un traitement parallèle massif (MPP) lui permet d’exécuter des analyses hautes performances. SQL Data Warehouse peut utiliser [PolyBase](/sql/relational-databases/polybase/polybase-guide) pour charger rapidement des données à partir du stockage d’objets blob.
- [Analysis Services](/azure/analysis-services) fournit un modèle sémantique de vos données. Ce composant permet également d’augmenter les performances du système lors de l’analyse de vos données.
- [Power BI](/power-bi) est une suite d’outils d’analyse métier pour analyser les données et partager les informations. Power BI peut interroger un modèle sémantique stocké dans Analysis Services ou interroger directement SQL Data Warehouse.
- [Azure Active Directory (Azure AD)](/azure/active-directory) authentifie les utilisateurs qui se connectent au serveur Analysis Services via Power BI. Data Factory peut également utiliser Azure AD pour s’authentifier auprès de SQL Data Warehouse via un principal de service ou une [identité managée pour des ressources Azure](/azure/active-directory/managed-identities-azure-resources/overview).

### <a name="alternatives"></a>Autres solutions

- L’exemple de pipeline inclut plusieurs types de sources de données. Cette architecture peut gérer un large éventail de sources de données relationnelles et non relationnelles.
- Data Factory orchestre les flux de travail pour votre pipeline de données. Si vous souhaitez charger des données une seule fois ou à la demande, vous pouvez utiliser des outils tels que la copie en bloc (bcp) et AzCopy de SQL Server pour copier des données dans le stockage d’objets blob. Vous pouvez alors charger les données dans SQL Data Warehouse à l’aide de PolyBase
- Si vous disposez de jeux de données très volumineux, pensez à utiliser [Data Lake Storage](/azure/storage/data-lake-storage/introduction), qui fournit un stockage illimité pour les données d’analyse.
- Une appliance [SQL Server Parallel Data Warehouse](/sql/analytics-platform-system) locale peut également être utilisée pour le traitement du Big Data. Toutefois, les coûts d’exploitation sont souvent bien inférieurs avec une solution managée basée sur le cloud telle que SQL Data Warehouse.
- SQL Data Warehouse ne convient pas à des charges de travail OLTP ni à des jeux de données inférieurs à 250 Go. Pour ces cas, vous devez utiliser Azure SQL Database ou SQL Server.
- Pour procéder à des comparaisons avec d’autres solutions, consultez les articles suivants :

  - [Sélection d’une technologie d’orchestration de pipeline de données dans Azure](/azure/architecture/data-guide/technology-choices/pipeline-orchestration-data-movement)
  - [Choix d’une technologie de traitement par lots dans Azure](/azure/architecture/data-guide/technology-choices/batch-processing)
  - [Sélection d’un magasin de données analytiques dans Azure](/azure/architecture/data-guide/technology-choices/analytical-data-stores)
  - [Sélection d’une technologie d’analytique de données dans Azure](/azure/architecture/data-guide/technology-choices/analysis-visualizations-reporting)

## <a name="considerations"></a>Considérations

Les technologies appliquées dans cette architecture ont été choisies, car elles remplissent les exigences de l’entreprise en matière d’extensibilité et de disponibilité tout en les aidant à maîtriser les coûts.

- [L’architecture de traitement massivement parallèle](/azure/sql-data-warehouse/massively-parallel-processing-mpp-architecture) de SQL Data Warehouse assure extensibilité et hautes performances.
- SQL Data Warehouse vous [garantit des SLA](https://azure.microsoft.com/support/legal/sla/sql-data-warehouse) et des [pratiques recommandées pour assurer une haute disponibilité](/azure/sql-data-warehouse/sql-data-warehouse-best-practices).
- Lorsque l’activité d’analyse est faible, l’entreprise peut [mettre à l’échelle SQL Data Warehouse à la demande](/azure/sql-data-warehouse/sql-data-warehouse-manage-compute-overview) de façon à réduire ou même à interrompre le calcul afin de réduire les coûts.
- La taille des instances Azure Analysis Services peut être [augmentée](/azure/analysis-services/analysis-services-scale-out) afin de réduire les temps de réponse au cours des charges de travail élevées pour les requêtes. Vous pouvez aussi séparer le traitement du pool de requêtes afin que les requêtes des clients ne soient pas ralenties par les opérations de traitement.
- Azure Analysis Services vous [garantit également des SLA](https://azure.microsoft.com/support/legal/sla/analysis-services) et des [pratiques recommandées pour assurer une haute disponibilité](/azure/analysis-services/analysis-services-bcdr).
- Le [modèle de sécurité de SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-manage-security) assure la sécurité des connexions, [l’authentification et l’autorisation](/azure/sql-data-warehouse/sql-data-warehouse-authentication) via l’authentification Azure AD ou SQL Server, et le chiffrement. [Azure Analysis Services](/azure/analysis-services/analysis-services-manage-users) utilise Azure AD pour la gestion des identités et l’authentification utilisateur.

## <a name="pricing"></a>Tarifs

Examinez un [exemple de tarification pour un scénario d’entreposage de données][calculator] via la calculatrice de prix Azure. Ajustez les valeurs pour déterminer l’incidence de vos besoins sur vos coûts.

- [SQL Data Warehouse](https://azure.microsoft.com/pricing/details/sql-data-warehouse/gen2) vous permet de mettre à l’échelle vos niveaux de calcul et de stockage de façon indépendante. Les ressources de calcul sont facturées à l’heure, et vous pouvez mettre ces ressources à l’échelle ou en pause à la demande. Les ressources de stockage sont facturées au téraoctet. Vos coûts augmentent donc en fonction du volume de données ingéré.
- Les coûts [Data Factory](https://azure.microsoft.com/pricing/details/data-factory) sont basés sur le nombre d’opérations de lecture/écriture et de surveillance et sur les activités d’orchestration effectuées dans une charge de travail. Vos coûts Data Factory augmentent avec chaque flux de données supplémentaire et la quantité de données traitées par chacun d’eux.
- [Analysis Services](https://azure.microsoft.com/pricing/details/analysis-services) est disponible pour les niveaux développeur, de base et standard. La tarification des instances est établie en fonction des unités de traitement des requêtes (QPU) et de la mémoire disponible. Pour diminuer vos coûts, réduisez le nombre de requêtes exécutées, la quantité de données traitées et leur fréquence d’exécution.
- [Power BI](https://powerbi.microsoft.com/pricing) offre différentes options de produit selon les besoins. [Power BI Embedded](https://azure.microsoft.com/pricing/details/power-bi-embedded) offre une option Azure permettant d’intégrer la fonctionnalité Power BI dans vos applications. L’exemple de tarification ci-dessus comprend une instance Power BI Embedded.

## <a name="next-steps"></a>Étapes suivantes

- Passez en revue l’article [BI d’entreprise automatisée avec SQL Data Warehouse et Azure Data Factory](/azure/architecture/reference-architectures/data/enterprise-bi-adf), qui inclut des instructions pour déployer une instance de cette architecture dans Azure.
- Lisez le [témoignage de Maritz Motivation Solutions][source-document]. Ce témoignage décrit une approche similaire pour la gestion des données client.
- Retrouvez des conseils architecturaux complets sur les pipelines de données, l’entreposage des données, le traitement analytique en ligne (OLAP) et le Big Data dans le [Guide d’architecture des données Azure](/azure/architecture/data-guide).

<!-- links -->

[source-document]: https://customers.microsoft.com/story/maritz
[calculator]: https://azure.com/e/b798fb70c53e4dd19fdeacea4db78276
[architecture]: ./media/architecture-data-warehouse.png
