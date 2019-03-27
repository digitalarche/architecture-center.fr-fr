---
title: Choisir une technologie de traitement par lots
description: ''
author: zoinerTejada
ms.date: 11/03/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: 53f8b233b0e0c1ff83a72a04b2707caa528d6f6b
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58248514"
---
# <a name="choosing-a-batch-processing-technology-in-azure"></a>Choisir une technologie de traitement par lots dans Azure

Les solutions de Big Data utilisent souvent de longs traitements par lots pour filtrer, agréger et, plus généralement, préparer les données pour l’analyse. Généralement, ces tâches impliquent de lire les fichiers sources dans un système de stockage évolutif (comme HDFS, Azure Data Lake Store et le Stockage Azure), de les traiter et d’écrire la sortie dans de nouveaux fichiers au sein du système de stockage évolutif.

La condition principale de ces moteurs de traitement par lots est la capacité à faire monter en charge les calculs, afin de gérer de gros volumes de données. Contrairement au traitement en temps réel, cependant, le traitement par lots est censé afficher des latences (temps entre l’ingestion des données et le calcul du résultat) qui se mesurent en minutes, voire en heures.

## <a name="technology-choices-for-batch-processing"></a>Choix de technologie pour le traitement par lots

### <a name="azure-sql-data-warehouse"></a>Azure SQL Data Warehouse

[SQL Data Warehouse](/azure/sql-data-warehouse/) est un système distribué conçu pour réaliser des analyses sur de grandes quantités de données. Il prend en charge le traitement MPP (Massive Parallel Processing), le rendant ainsi adapté à l’exécution d’analyses hautes performances. Préférez utiliser SQL Data Warehouse lorsque vous disposez d’une importante quantité de données (supérieure à 1 To) et que vous exécutez une charge de travail d’analyse qui profiterait de ce parallélisme.

### <a name="azure-data-lake-analytics"></a>Service Analytique Azure Data Lake

[Data Lake Analytics](/azure/data-lake-analytics/data-lake-analytics-overview) est un service de travaux d’analyse à la demande. Il est optimisé pour le traitement distribué de jeux de données volumineux stockés dans Azure Data Lake Storage.

- Langages : [U-SQL](/azure/data-lake-analytics/data-lake-analytics-u-sql-get-started) (notamment les extensions Python, R et C#).
- S’intègre à Azure Data Lake Storage, aux objets blob de stockage Azure, à Azure SQL Database et à SQL Data Warehouse.
- Le modèle de tarification est calculé en fonction du nombre de travaux.

### <a name="hdinsight"></a>HDInsight

HDInsight est un service géré Hadoop. Utilisez-le pour déployer et gérer des clusters Hadoop dans Azure. Pour le traitement par lots, vous pouvez utiliser [Spark](/azure/hdinsight/spark/apache-spark-overview), [Hive](/azure/hdinsight/hadoop/hdinsight-use-hive), [Hive LLAP](/azure/hdinsight/interactive-query/apache-interactive-query-get-started), [MapReduce](/azure/hdinsight/hadoop/hdinsight-use-mapreduce).

- Langages : R, Python, Java, Scala, SQL
- Authentification Kerberos avec Active Directory, contrôle d’accès basé sur Apache Ranger
- Vous offre un contrôle total sur le cluster Hadoop

### <a name="azure-databricks"></a>Azure Databricks

[Azure Databricks](/azure/azure-databricks/) est une plateforme d’analyse basée sur Apache Spark. Celle-ci peut-être considérée comme « Spark en tant que service ». Il s’agit du moyen le plus simple d’utiliser Spark sur la plateforme Azure.

- Langages : R, Python, Java, Scala, Spark SQL
- Heures de début du cluster rapides, arrêt et mise à l’échelle automatiques.
- Gère le cluster Spark à votre place.
- Intégration au stockage Blob Azure, Azure Data Lake Storage (ADLS), Azure SQL Data Warehouse (SQL DW) et d’autres services. Consultez [Source de données](https://docs.azuredatabricks.net/spark/latest/data-sources/index.html).
- Authentification utilisateur avec Azure Active Directory.
- [Notebooks](https://docs.azuredatabricks.net/user-guide/notebooks/index.html) basés sur le web pour la collaboration et l’exploration de données.
- Prend en charge les [clusters compatibles GPU](https://docs.azuredatabricks.net/user-guide/clusters/gpu.html)

### <a name="azure-distributed-data-engineering-toolkit"></a>Azure Distributed Data Engineering Toolkit

Le [Distributed Data Engineering Toolkit](https://github.com/azure/aztk) (AZTK) est un outil qui permet de configurer des travaux Spark à la demande sur des clusters Docker dans Azure.

AZTK n’est pas un service Azure. Il s’agit plutôt d’un outil côté client avec une interface CLI et SDK Python, basé sur Azure Batch. C’est l’option qui vous offre le contrôle le plus poussé sur l’infrastructure lors du déploiement d’un cluster Spark.

- Apportez votre propre image Docker.
- Utilisez des machines virtuelles de faible priorité pour une remise de 80 %.
- Clusters en mode mixte qui utilisent des machines virtuelles de faible priorité et dédiées.
- Prise en charge intégrée de Stockage Blob Azure et de la connexion Azure Data Lake.

## <a name="key-selection-criteria"></a>Critères de sélection principaux

Pour restreindre les choix, commencez par répondre aux questions suivantes :

- Préférez-vous opter pour un service géré plutôt que de gérer vos propres serveurs ?

- Souhaitez-vous créer la logique de traitement par lots de manière déclarative ou impérative ?

- Effectuerez-vous des traitements par lots en rafales ? Si oui, tournez-vous vers des solutions permettant l’arrêt automatique du cluster ou suivant un modèle tarifaire par programme de traitement par lots.

- Avez-vous besoin d’interroger des magasins de données relationnels en parallèle de vos traitements par lots, par exemple, pour rechercher des données de référence ? Si oui, optez pour des solutions permettant d’interroger des magasins relationnels externes.

## <a name="capability-matrix"></a>Matrice des fonctionnalités

Les tableaux suivants résument les principales différences entre les fonctionnalités.

### <a name="general-capabilities"></a>Fonctionnalités générales

<!-- markdownlint-disable MD033 -->

| | Service Analytique Azure Data Lake | Azure SQL Data Warehouse | HDInsight | Azure Databricks |
| --- | --- | --- | --- | --- | --- |
| Est un service géré | OUI | OUI | Oui <sup>1</sup> | OUI |
| Magasin de données relationnel | OUI | OUI | Non  | Non  |
| Modèle de tarification | Par programme de traitement par lots | Par heure de cluster | Par heure de cluster | Unité Databricks<sup>2</sup> + heure de cluster |

[1] Avec mise à l’échelle et configuration manuelles.

[2] Une unité Databricks (DBU) est une unité de capabilité de processus par heure.

### <a name="capabilities"></a>Fonctionnalités

| | Service Analytique Azure Data Lake | SQL Data Warehouse | HDInsight avec Spark | HDInsight avec Hive | HDInsight avec Hive LLAP | Azure Databricks |
| --- | --- | --- | --- | --- | --- | --- |
| Mise à l’échelle automatique | Non  | Non  | Non  | Non  | Non  | OUI |
| Granularité de la montée en charge  | Par tâche | Par cluster | Par cluster | Par cluster | Par cluster | Par cluster |
| Mise en cache des données en mémoire | Non  | OUI | OUI | Non  | OUI | OUI |
| Interrogation à partir de magasins relationnels externes | OUI | Non  | OUI | Non  | Non  | OUI |
| Authentication  | Azure AD | SQL / Azure AD | Non  | Azure AD<sup>1</sup> | Azure AD<sup>1</sup> | Azure AD |
| Audit  | OUI | OUI | Non  | Oui <sup>1</sup> | Oui <sup>1</sup> | OUI |
| Sécurité au niveau des lignes | Non  | Non  | Non  | Oui <sup>1</sup> | Oui <sup>1</sup> | Non  |
| Prend en charge les pare-feu | OUI | OUI | OUI | Oui <sup>2</sup> | Oui <sup>2</sup> | Non  |
| Masquage des données dynamiques | Non  | Non  | Non  | Oui <sup>1</sup> | Oui <sup>1</sup> | Non  |

<!-- markdownlint-enable MD033 -->

[1] Suppose d’utiliser un [cluster HDInsight joint à un domaine](/azure/hdinsight/domain-joined/apache-domain-joined-introduction).

[2] Pris en charge si [utilisé au sein d’un Réseau virtuel Azure](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network).
