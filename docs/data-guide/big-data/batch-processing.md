---
title: Traitement par lots
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: d6843bf4e20c3eb26e61cfa09300ad533e969c2e
ms.sourcegitcommit: 51f49026ec46af0860de55f6c082490e46792794
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/03/2018
ms.locfileid: "30298657"
---
# <a name="batch-processing"></a>Traitement par lots

L’un des scénarios les plus courants pour le Big Data est le traitement par lots de données au repos. Dans ce cas de figure, la source de données est chargé dans le stockage de données, soit par l’application de la source elle-même, soit par un workflow d’orchestration. Les données sont alors traitées sur place par une tâche parallélisée, qui peut également être lancée par le workflow d’orchestration. Le traitement peut passer par plusieurs étapes itératives avant que les résultats transformés ne soient chargés dans un magasin de données analytiques, qui peut être interrogé par des composants d’analytique et de création de rapports.

Par exemple, les journaux provenant d’un serveur web peuvent être copiés dans un dossier, puis traités pendant la nuit pour générer les rapports quotidiens d’activités web.

![](./images/batch-pipeline.png)

## <a name="when-to-use-this-solution"></a>Quand utiliser cette solution

Le traitement par lots est utilisé dans différents scénarios, qui vont des transformations de données simples à un pipeline ETL (extraction, transformation et chargement) plus complet. Dans un contexte de Big Data, le traitement par lots peut fonctionner sur des jeux de données très volumineux, pour lesquels le calcul prend beaucoup de temps. (Voir par exemple [Architecture lambda](../big-data/index.md#lambda-architecture).) En général, il aboutit à une exploration interactive plus poussée, fournit des données modélisables pour le Machine Learning ou écrit les données dans un magasin de données optimisé pour l’analytique et la visualisation.

Il peut s’agir par exemple de transformer un grand ensemble de fichiers CSV ou JSON plats et semi-structurées dans un format schématisé et structuré afin de pousser plus loin les requêtes. La plupart du temps, les données sont converties d’un format brut utilisé pour l’ingestion (par exemple, CSV) dans un format binaire, plus performant pour l’interrogation, car stockant les données en colonnes et fournissant souvent des index et des statistiques incluses sur les données.

## <a name="challenges"></a>Défis

- **Encodage et format de données**. Certains des problèmes plus difficiles à déboguer se produisent lorsque les fichiers utilisent un encodage ou un format inattendu. Par exemple, les fichiers sources peuvent utiliser un mélange d’encodage UTF-16 et UTF-8, ou comporter des caractères ou des délimiteurs inattendus (espace au lieu de tabulation). On peut également citer l’exemple de champs de texte contenant des tabulations, des espaces ou des virgules qui sont interprétés comme des délimiteurs. La logique de chargement et d’analyse des données doit être suffisamment souple pour détecter et gérer ces problèmes.

- **Orchestration de tranches horaires**. Les données sources sont souvent placées dans une arborescence de dossiers qui reflète les fenêtres de traitement, organisées par année, par mois, par jour, par heure, etc. Dans certains cas, les données peuvent arrivent en retard. Supposons par exemple que, à cause d’une défaillance sur un serveur web, les journaux du 7 mars ne se retrouvent pas dans le dossier de traitement avant le 9 mars. Sont-elles simplement ignorées du fait de leur retard ? La logique de traitement en aval est-elle capable de gérer les enregistrements non ordonnés ?

## <a name="architecture"></a>Architecture

Une architecture de traitement par lots comporte les éléments logiques suivants, illustrés dans le diagramme ci-dessus.

- **Stockage des données**. En général, un magasin de fichiers distribués pouvant servir de référentiel pour de gros volumes de fichiers dans différents formats. Ce type de magasin est communément appelé lac de données. 

- **Traitement par lots**. En raison des gros volumes en jeu dans le Big Data, les solutions doivent gérer les fichiers de données en appliquant des traitements par lots de longue durée pour filtrer, agréger et, plus généralement, préparer les données à des fins d’analyse. Généralement, ces travaux impliquent la lecture des fichiers source, leur traitement et l’écriture de la sortie dans de nouveaux fichiers. 

- **Magasin de données analytiques**. De nombreuses solutions Big Data sont conçues pour préparer les données à des fins d’analyse, puis fournir les données traitées dans un format structuré et interrogeable à l’aide d’outils d’analyse. 

- **Analyse et rapports**. La plupart des solutions Big Data ont pour but de fournir des informations sur les données par le biais de l’analyse et des rapports. 

- **Orchestration**. Avec le traitement par lots, une orchestration est en général nécessaire pour migrer ou copier les données dans les couches de stockage, de traitement, de magasin de données analytiques et de création de rapports.

## <a name="technology-choices"></a>Choix de technologie

Les technologies suivantes sont recommandées pour les solutions de traitement par lots dans Azure.

### <a name="data-storage"></a>Stockage des données

- **Conteneurs Azure Storage Blob**. De nombreux processus d’entreprise Azure utilisent déjà le Stockage Blob Azure, ce qui en fait un bon choix pour un magasin de Big Data.
- **Azure Data Lake Store**. Azure Data Lake Store offre un stockage quasiment illimité pour des fichiers de toutes tailles, ainsi que des options de sécurité étendues, ce qui en fait un bon choix pour les solutions de Big Data à très grande échelle qui nécessitent un magasin centralisé de données de formats hétérogènes.

Pour plus d’informations, consultez la page [Stockage de données](../technology-choices/data-storage.md).

### <a name="batch-processing"></a>Traitement par lots

- **U-SQL**. U-SQL est le langage de traitement des requêtes utilisé par Azure Data Lake Analytics. Il combine la nature déclarative de SQL avec l’extensibilité procédurale de C#, et tire parti du parallélisme pour permettre un traitement efficace des données à très grande échelle.
- **Hive**. Hive est un langage de type SQL pris en charge dans la plupart des distributions Hadoop, y compris HDInsight. Il permet de traiter des données provenant de n’importe quel magasin compatible HDFS, notamment le Stockage Blob Azure et Azure Data Lake Store.
- **Pig**. Pig est un langage de traitement du Big Data déclaratif utilisé dans de nombreuses distributions Hadoop, notamment HDInsight. Il est particulièrement utile pour traiter des données non structurées ou semi-structurées.
- **Spark**. Le moteur Spark prend en charge les programmes de traitement par lots écrits dans différents langages, notamment Python, Java et Scala. Il utilise une architecture distribuée pour traiter les données en parallèle sur plusieurs nœuds de travail.

Pour plus d’informations, consultez la section [Traitement par lots](../technology-choices/batch-processing.md).

### <a name="analytical-data-store"></a>Magasin de données analytiques

- **SQL Data Warehouse**. Azure SQL Data Warehouse est un service géré fondé sur les technologies de base de données SQL Server et optimisé pour prendre en charge les grosses charges de travail d’entreposage de données.
- **Spark SQL**. Spark SQL est une API qui repose sur Spark et prend en charge la création de trames de données et de tables interrogeables avec la syntaxe SQL.
- **HBase**. HBase est un magasin NoSQL à faible latence qui permet d’interroger des données structurées et semi-structurées d’une manière souple et très performante.
- **Hive**. En plus d’être utile pour le traitement par lots, Hive offre une architecture de base de données semblable d’un point de vue conceptuel à celle d’un système de gestion de base de données relationnelle classique. Grâce aux améliorations apportées aux performances des requêtes Hive par des innovations comme le moteur Tez et l’initiative Stinger, les tables Hive représentent des sources efficaces pour les requêtes analytiques dans certains scénarios.

Pour plus d’informations, consultez la section [Magasins de données analytiques](../technology-choices/analytical-data-stores.md).

### <a name="analytics-and-reporting"></a>Analytique et création de rapports

- **Azure Analysis Services**. De nombreuses solutions de Big Data émulent des architectures d’informatique décisionnelle d’entreprise classiques en intégrant un modèle de données OLAP (traitement analytique en ligne) centralisé (communément appelé cube) qui peut servir de base aux rapports, aux tableaux de bord et à l’analyse interactive par permutation d'axes. Azure Analysis Services prend en charge la création de modèles multidimensionnels et tabulaires pour répondre à ce besoin.
- **Power BI**. Power BI permet aux analystes de données de créer des visualisations de données interactives sur la base de modèles de données dans un modèle OLAP ou directement à partir d’un magasin de données analytiques.
- **Microsoft Excel**. Microsoft Excel est l’une des applications logicielles les plus largement utilisées dans le monde ; elle offre un large éventail de fonctionnalités d’analyse et de visualisation de données. Les analystes de données peuvent se servir d’Excel pour générer des modèles de données de documents à partir de magasins de données analytiques, ou pour récupérer des données provenant de modèles de données OLAP dans des graphiques et des tableaux croisés dynamiques interactifs.

Pour plus d'informations, consultez la section [Analytique et création de rapports](../technology-choices/analysis-visualizations-reporting.md).

### <a name="orchestration"></a>Orchestration

- **Azure Data Factory**. Les pipelines Azure Data Factory peuvent servir à définir une séquence d’activités, planifiée pour des fenêtres temporelles périodiques. Ces activités peuvent lancer des opérations de copie de données ainsi que des tâches Hive, Pig, MapReduce ou Spark dans des clusters HDInsight à la demande ; des tâches U-SQL dans Azure Date Lake Analytics ; et des procédures stockées dans Azure SQL Data Warehouse ou Azure SQL Database.
- **Oozie** et **Sqoop**. Oozie est un moteur d’automatisation des tâches pour l’écosystème Apache Hadoop qui permet de lancer des opérations de copie de données, ainsi que des tâches Hive, Pig et MapReduce pour traiter les données, et Sqoop pour les copier entre des bases de données HDFS et SQL.

Pour plus d’informations, consultez la section [Orchestration de pipelines](../technology-choices/pipeline-orchestration-data-movement.md).
