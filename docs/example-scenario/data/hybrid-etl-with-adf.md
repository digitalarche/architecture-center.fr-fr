---
title: ETL hybride avec une instance SSIS locale existante et Azure Data Factory
titleSuffix: Azure Example Scenarios
description: ETL hybride avec des déploiements SSIS (SQL Server Integration Services) locaux existants et Azure Data Factory.
author: alhieng
ms.date: 09/20/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: tsp-team
ms.openlocfilehash: 2fc5a94391b2e7e24209e884a790fe72070c4fc2
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54485109"
---
# <a name="hybrid-etl-with-existing-on-premises-ssis-and-azure-data-factory"></a>ETL hybride avec une instance SSIS locale existante et Azure Data Factory

Les organisations qui migrent leurs bases de données SQL Server vers le cloud peuvent réaliser des économies considérables, augmenter les performances, gagner en flexibilité et améliorer la scalabilité. Toutefois, le fait de retravailler des processus ETL (extraire, transformer et charger) existants conçus avec SQL Server Integration Services (SSIS) peut être un obstacle pour la migration. Dans d’autres cas, le processus de chargement des données nécessite une logique complexe et/ou des composants d’outil de données spécifiques qui ne sont pas encore pris en charge par Azure Data Factory v2. Les fonctionnalités SSIS couramment utilisées comprennent des transformations Recherche floue et Regroupement probable, la capture des changements de données (CDC), les dimensions à variation lente (SCD) et Data Quality Services (DQS).

L’approche ETL hybride peut être l’option la plus adaptée pour faciliter une migration lift-and-shift d’une base de données SQL existante. Une approche hybride utilise Data Factory comme moteur d’orchestration principal, mais continue à tirer parti des packages SSIS existants pour nettoyer les données et utiliser les ressources locales. Cette approche utilise SQL Server Integrated Runtime (IR) de Data Factory pour effectuer un lift-and-shift des bases de données existantes dans le cloud, tout en utilisant le code existant et les packages SSIS.

Cet exemple de scénario s’applique aux organisations qui migrent des bases de données vers le cloud et envisagent d’utiliser Data Factory comme moteur ETL principal dans le cloud, tout en incorporant des packages SSIS existants dans leur nouveau workflow de données cloud. De nombreuses organisations ont beaucoup investi dans le développement de packages SSIS ETL pour des tâches de données spécifiques. La réécriture de ces packages peut être décourageante. Par ailleurs, de nombreux packages de code existants ont des dépendances sur des ressources locales qui empêchent la migration vers le cloud.

Data Factory permet aux clients de tirer parti de leurs packages ETL existants sans investir outre mesure dans le développement d’ETL locaux. Cet exemple présente des cas d’usage potentiels pour tirer parti des packages SSIS existants dans le cadre d’un nouveau workflow de données cloud à l’aide d’Azure Data Factory v2.

## <a name="potential-use-cases"></a>Cas d’usage potentiels

À l’origine, SSIS était l’outil ETL de choix pour de nombreux professionnels des données SQL Server dans le cadre de la transformation de données et du chargement. Parfois, des fonctionnalités SSIS spécifiques ou des composants enfichables tiers étaient utilisés pour accélérer les efforts de développement. Le remplacement ou le redéveloppement de ces packages n’est pas toujours possible et empêche les clients de migrer leurs bases de données vers le cloud. Les clients cherchent une approche qui leur permette de migrer leurs bases de données existantes dans le cloud et de tirer parti de leurs packages SSIS existants avec le moins d’impact possible.

Plusieurs cas d’usage locaux potentiels sont listés ci-dessous :

- Chargement des journaux de routeur réseau dans une base de données pour analyse.
- Préparation des données d’emploi des ressources humaines pour les rapports analytiques.
- Chargement des données de produit et de ventes dans un entrepôt de données pour la prévision des ventes.
- Automatisation du chargement des magasins de données opérationnelles ou des entrepôts de données pour la finance et la comptabilité.

## <a name="architecture"></a>Architecture

![Vue d’ensemble de l’architecture d’un processus ETL hybride avec Azure Data Factory][architecture-diagram]

1. Les données proviennent d’un stockage Blob dans Data Factory.
2. Le pipeline Data Factory appelle une procédure stockée pour exécuter un travail SSIS hébergé localement via Integrated Runtime.
3. Les travaux de nettoyage de données sont exécutés pour préparer les données à la consommation en aval.
4. Une fois effectuée la tâche de nettoyage des données, une tâche de copie est exécutée pour charger les données propres dans Azure.
5. Les données propres sont ensuite chargées dans des tables dans SQL Data Warehouse.

### <a name="components"></a>Composants

- Le [Stockage Blob][docs-blob-storage] est utilisé pour stocker des fichiers et comme source de Data Factory pour récupérer des données.
- [SQL Server Integration Services][docs-ssis] contient les packages ETL locaux utilisés pour exécuter les charges de travail propres à la tâche.
- [Azure Data Factory][docs-data-factory] est le moteur d’orchestration cloud qui prend des données de plusieurs sources, et les combine, les orchestre et les charge dans un entrepôt de données.
- [SQL Data Warehouse][docs-sql-data-warehouse] centralise les données dans le cloud pour pouvoir y accéder facilement à l’aide de requêtes SQL ANSI standard.

### <a name="alternatives"></a>Autres solutions

Data Factory peut appeler des procédures de nettoyage de données implémentées à l’aide d’autres technologies, comme un notebook Databricks, un script Python ou une instance SSIS s’exécutant sur une machine virtuelle. [L’installation d’une version payante ou sous licence des composants personnalisés pour le runtime d’intégration Azure-SSIS](/azure/data-factory/how-to-develop-azure-ssis-ir-licensed-components) peut être une alternative viable à l’approche hybride.

## <a name="considerations"></a>Considérations

Le runtime intégré (IR) prend en charge deux modèles : IR auto-hébergé ou IR hébergé par Azure. Vous devez d’abord choisir l’une de ces deux options. L’auto-hébergement est plus économique, mais représente davantage de gestion et de maintenance. Pour plus d'informations, consultez [IR auto-hébergé](/azure/data-factory/concepts-integration-runtime#self-hosted-integration-runtime). Si vous avez besoin d’aide pour déterminer l’IR à utiliser, consultez [Choix du runtime d’intégration à utiliser](/azure/data-factory/concepts-integration-runtime#determining-which-ir-to-use).

Pour l’approche d’hébergement sur Azure, vous devez choisir la quantité d’énergie nécessaire pour traiter vos données. La configuration hébergée sur Azure vous permet de sélectionner la taille de machine virtuelle dans le cadre des étapes de configuration. Pour en savoir plus sur la sélection des tailles de machine virtuelle, consultez [Considérations sur les performances de machine virtuelle](/azure/cloud-services/cloud-services-sizes-specs#performance-considerations).

Le choix est plus facile quand vous avez déjà des packages SSIS existants avec des dépendances locales comme des sources de données ou des fichiers qui ne sont pas accessibles à partir d’Azure. Dans ce scénario, votre seule option est l’IR auto-hébergé. Cette approche offre davantage de souplesse pour tirer parti du cloud comme moteur d’orchestration, sans avoir à réécrire les packages existants.

L’objectif est de déplacer les données traitées dans le cloud pour les affiner ou les combiner avec d’autres données stockées dans le cloud. Dans le cadre du processus de conception, effectuez le suivi du nombre d’activités utilisées dans les pipelines Data Factory. Pour plus d’informations, consultez [Pipelines et activités dans Azure Data Factory](/azure/data-factory/concepts-pipelines-activities).

## <a name="pricing"></a>Tarifs

Data Factory est un moyen économique d’orchestrer le déplacement des données dans le cloud. Le coût dépend de plusieurs facteurs.

- Nombre d’exécutions de pipeline
- Nombre d’entités/d’activités utilisées dans le pipeline
- Nombre d’opérations de supervision
- Nombre d’exécutions d’intégration (IR hébergé dans Azure ou IR auto-hébergé)

Data Factory utilise la facturation à l’utilisation. Par conséquent, les frais sont facturés uniquement pendant les exécutions de pipeline et la supervision. L’exécution d’un pipeline de base ne coûte pas plus de 50 cents et la supervision pas plus de 25 cents. Le [calculateur de coût Azure](https://azure.microsoft.com/pricing/calculator/) peut vous aider à élaborer une estimation plus précise en fonction de votre charge de travail spécifique.

Quand vous exécutez une charge de travail ETL hybride, vous devez tenir compte du coût de la machine virtuelle qui héberge vos packages SSIS. Ce coût est basé sur la taille de la machine virtuelle : de D1v2 (1 cœur, 3,5 Go de RAM, disque de 50 Go) à E64V3 (64 cœurs, 432 Go de RAM, disque de 1 600 Go). Si vous avez besoin d’aide pour choisir la taille appropriée de la machine virtuelle, consultez [Considérations sur les performances de machine virtuelle](/azure/cloud-services/cloud-services-sizes-specs#performance-considerations).

## <a name="next-steps"></a>Étapes suivantes

- Découvrez plus d’informations sur [Azure Data Factory](https://azure.microsoft.com/services/data-factory/).
- Démarrez avec Azure Data Factory en suivant le [Tutoriel pas à pas](/azure/data-factory/#step-by-step-tutorials).
- [Provisionnez Azure-SSIS Integration Runtime dans Azure Data Factory](/azure/data-factory/tutorial-deploy-ssis-packages-azure).

<!-- links -->
[architecture-diagram]: ./media/architecture-diagram-hybrid-etl-with-adf.png
[small-pricing]: https://azure.com/e/
[medium-pricing]: https://azure.com/e/
[large-pricing]: https://azure.com/e/
[availability]: /azure/architecture/checklist/availability
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[docs-blob-storage]: /azure/storage/blobs/
[docs-data-factory]: /azure/data-factory/introduction
[docs-resource-groups]: /azure/azure-resource-manager/resource-group-overview
[docs-ssis]: /sql/integration-services/sql-server-integration-services
[docs-sql-data-warehouse]: /azure/sql-data-warehouse/sql-data-warehouse-overview-what-is
