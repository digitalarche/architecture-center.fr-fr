---
title: Choisir une technologie de traitement par lots
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 0117798af82f2caa6704dc86e88be57f09c381ea
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/06/2018
---
# <a name="choosing-a-batch-processing-technology-in-azure"></a>Choisir une technologie de traitement par lots dans Azure

Les solutions de Big Data utilisent souvent de longs traitements par lots pour filtrer, agréger et, plus généralement, préparer les données pour l’analyse. Généralement, ces tâches impliquent de lire les fichiers sources dans un système de stockage évolutif (comme HDFS, Azure Data Lake Store et le Stockage Azure), de les traiter et d’écrire la sortie dans de nouveaux fichiers au sein du système de stockage évolutif. 

La condition principale de ces moteurs de traitement par lots est la capacité à faire monter en charge les calculs, afin de gérer de gros volumes de données. Contrairement au traitement en temps réel, cependant, le traitement par lots est censé afficher des latences (temps entre l’ingestion des données et le calcul du résultat) qui se mesurent en minutes, voire en heures.

## <a name="what-are-your-options-when-choosing-a-batch-processing-technology"></a>Quels sont les choix qui s’offrent à vous en matière de technologie de traitement par lots ?

Dans Azure, tous les magasins de données suivants répondront aux principales exigences du traitement par lots :

- [Service Analytique Azure Data Lake](/azure/data-lake-analytics/)
- [Azure SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [HDInsight avec Spark](/azure/hdinsight/spark/apache-spark-overview)
- [HDInsight avec Hive](/azure/hdinsight/hadoop/hdinsight-use-hive)
- [HDInsight avec Hive LLAP](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)

## <a name="key-selection-criteria"></a>Critères de sélection principaux

Pour restreindre les choix, commencez par répondre aux questions suivantes :

- Préférez-vous opter pour un service géré plutôt que de gérer vos propres serveurs ?

- Souhaitez-vous créer la logique de traitement par lots de manière déclarative ou impérative ?

- Effectuerez-vous des traitements par lots en rafales ? Si oui, tournez-vous vers des solutions permettant d’interrompre le cluster ou suivant un modèle tarifaire par programme de traitement par lots.

- Avez-vous besoin d’interroger des magasins de données relationnels en parallèle de vos traitements par lots, par exemple, pour rechercher des données de référence ? Si oui, optez pour des solutions permettant d’interroger des magasins relationnels externes.

## <a name="capability-matrix"></a>Matrice des fonctionnalités

Les tableaux suivants résument les principales différences entre les fonctionnalités. 

### <a name="general-capabilities"></a>Fonctionnalités générales

| | Service Analytique Azure Data Lake | Azure SQL Data Warehouse | HDInsight avec Spark | HDInsight avec Hive | HDInsight avec Hive LLAP |
| --- | --- | --- | --- | --- | --- |
| Est un service géré | OUI | OUI | Oui <sup>1</sup> | Oui <sup>1</sup> | Oui<sup>1</sup> |
| Prend en charge l’interruption du calcul | Non  | OUI | Non  | Non  | Non  |
| Magasin de données relationnel | OUI | OUI | Non  | Non  | Non  |
| Programmabilité | U-SQL | T-SQL | Python, Scala, Java, R | HiveQL | HiveQL |
| Paradigme de programmation | À la fois déclaratif et impératif  | Déclaratif | À la fois déclaratif et impératif | Déclaratif | Déclaratif | 
| Modèle de tarification | Par programme de traitement par lots | Par heure de cluster | Par heure de cluster | Par heure de cluster | Par heure de cluster |  

[1] Avec mise à l’échelle et configuration manuelles.

### <a name="integration-capabilities"></a>Fonctionnalités d’intégration

| | Service Analytique Azure Data Lake | SQL Data Warehouse | HDInsight avec Spark | HDInsight avec Hive | HDInsight avec Hive LLAP |
| --- | --- | --- | --- | --- | --- |
| Accès à partir d’Azure Data Lake Store | OUI | OUI | OUI | OUI | OUI |
| Interrogation à partir du Stockage Azure | OUI | OUI | OUI | OUI | OUI |
| Interrogation à partir de magasins relationnels externes | OUI | Non  | OUI | Non  | Non  |

### <a name="scalability-capabilities"></a>Fonctionnalités d’évolutivité

| | Service Analytique Azure Data Lake | SQL Data Warehouse | HDInsight avec Spark | HDInsight avec Hive | HDInsight avec Hive LLAP |
| --- | --- | --- | --- | --- | --- |
| Granularité de la montée en charge  | Par tâche | Par cluster | Par cluster | Par cluster | Par cluster |
| Montée en charge rapide (moins d’une minute) | OUI | OUI | Non  | Non  | Non  |
| Mise en cache des données en mémoire | Non  | OUI | OUI | Non  | OUI | 

### <a name="security-capabilities"></a>Fonctionnalités de sécurité

| | Service Analytique Azure Data Lake | SQL Data Warehouse | HDInsight avec Spark | Apache Hive sur HDInsight | Hive LLAP sur HDInsight |
| --- | --- | --- | --- | --- | --- |
| Authentification  | Azure Active Directory (Azure AD) | SQL / Azure AD | Non  | local / Azure AD <sup>1</sup> | local / Azure AD <sup>1</sup> |
| Authorization  | OUI | OUI| Non  | Oui <sup>1</sup> | Oui <sup>1</sup> |
| Audit  | OUI | OUI | Non  | Oui <sup>1</sup> | Oui <sup>1</sup> |
| Chiffrement des données au repos | OUI| Oui <sup>2</sup> | OUI | OUI | OUI |
| Sécurité au niveau des lignes | Non  | OUI | Non  | Oui <sup>1</sup> | Oui<sup>1</sup> |
| Prend en charge les pare-feu | OUI | OUI | OUI | Oui <sup>3</sup> | Oui <sup>3</sup> |
| Masquage des données dynamiques | Non  | Non  | Non  | Oui <sup>1</sup> | Oui<sup>1</sup> |

[1] Suppose d’utiliser un [cluster HDInsight joint à un domaine](/azure/hdinsight/domain-joined/apache-domain-joined-introduction).

[2] Suppose d’utiliser le chiffrement TDE (Transparent Data Encryption) pour chiffrer et déchiffrer les données au repos.

[3] Pris en charge si [utilisé au sein d’un Réseau virtuel Azure](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network).
