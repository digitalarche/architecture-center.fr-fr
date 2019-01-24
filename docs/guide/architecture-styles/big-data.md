---
title: Style d’architecture Big Data
titleSuffix: Azure Application Architecture Guide
description: Décrit les avantages, les inconvénients et les bonnes pratiques des architectures Big Data sur Azure.
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19, IoT
ms.openlocfilehash: 07fb955fdc5475020097554070753689d1515462
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54488018"
---
# <a name="big-data-architecture-style"></a>Style d’architecture Big Data

Une architecture Big Data est conçue pour gérer l’ingestion, le traitement et l’analyse de données trop volumineuses ou complexes pour les systèmes de base de données traditionnels.

![Diagramme logique d’un style d’architecture Big Data](./images/big-data-logical.svg)

Les solutions Big Data impliquent généralement un ou plusieurs des types de charges de travail suivants :

- Traitement par lots des sources Big Data au repos.
- Traitement en temps réel des Big Data en mouvement.
- Exploration interactive des Big Data.
- Analyse prédictive et apprentissage machine.

La plupart des architectures Big Data incluent tout ou partie des composants suivants :

- **Sources de données** : Toutes les solutions Big Data reposent sur une ou plusieurs sources de données. Voici quelques exemples :

  - Magasins de données d’application, tels que des bases de données relationnelles.
  - Fichiers statiques produits par les applications, tels que les fichiers journaux de serveur web.
  - Sources de données en temps réel, tels que les appareils IoT.

- **Stockage des données** : Les données destinées aux opérations de traitement par lots sont généralement stockées dans un magasin de fichiers distribués, qui peut contenir de vastes volumes de fichiers volumineux dans divers formats. Ce type de magasin est souvent appelé « *lac de données* ». Les options pour l’implémentation de ce stockage incluent des conteneurs Azure Data Lake Store ou les conteneurs blob dans le stockage Azure.

- **Traitement par lots** : Étant donné que les jeux de données sont trop lourds, une solution Big Data doit souvent traiter les fichiers de données à l’aide de traitements par lots à longue durée d’exécution pour filtrer, agréger et préparer les données en vue de l’analyse. Généralement, ces travaux impliquent la lecture des fichiers source, leur traitement et l’écriture de la sortie dans de nouveaux fichiers. Les options incluent l’exécution de travaux U-SQL dans Azure Data Lake Analytics, à l’aide de travaux personnalisés de mappage/réduction, Pig ou Hive dans un cluster HDInsight Hadoop ou à l’aide des programmes Java, Scala ou Python dans un cluster HDInsight Spark.

- **Ingestion de messages en temps réel** : Si la solution inclut des sources en temps réel, l’architecture doit inclure un moyen pour capturer et stocker des messages en temps réel pour le traitement de flux de données. Il peut s’agir d’un simple magasin de données, où les messages entrants sont déposés dans un dossier en vue du traitement. Toutefois, de nombreuses solutions besoin d’un magasin d’ingestion des messages qui agit comme une mémoire tampon pour les messages et qui prend en charge un traitement de montée en puissance, une remise fiable et d’autres sémantiques de files d’attente de message. Les options incluent Azure Event Hubs, Azure IoT Hubs et Kafka.

- **Traitement de flux** : Après avoir capturé les messages en temps réel, la solution doit les traiter en filtrant, en agrégeant et, plus généralement, en préparant les données pour l’analyse. Les données de flux traitées sont ensuite écrites dans un récepteur de sortie. Azure Stream Analytics fournit un service de traitement de flux managé reposant sur des requêtes SQL à l’exécution permanente qui fonctionnent sur les flux de données indépendants. Vous pouvez également utiliser des technologies de flux Apache open source comme Storm et Spark dans un cluster HDInsight.

- **Magasin de données analytiques** : De nombreuses solutions Big Data préparent les données pour l’analyse, puis fournissent les données traitées dans un format structuré qui peut être interrogé à l’aide des outils d’analyse. Le magasin de données analytique utilisé pour répondre à ces requêtes peut être un entrepôt de données relationnelles de type Kimball, comme indiqué dans les solutions décisionnelles (BI) plus traditionnelles. Les données peuvent également être présentées via une technologie NoSQL à faible latence, telle que HBase, ou via une base de données Hive interactif qui fournit une abstraction de métadonnées sur les fichiers de données dans le magasin de données distribuées. Azure SQL Data Warehouse fournit un service managé pour l’entreposage cloud des données à grande échelle. HDInsight prend en charge les formats Hive interactif, HBase et Spark SQL, qui peuvent également servir à préparer les données en vue de l’analyse.

- **Analyse et rapports** : La plupart des solutions Big Data ont pour but de fournir des informations sur les données par le biais de l’analyse et des rapports. Pour permettre aux utilisateurs d’analyser les données, l’architecture peut inclure une couche de modélisation des données, comme un cube OLAP multidimensionnel ou un modèle de données tabulaire dans Azure Analysis Services. Elle peut également prendre en charge le décisionnel libre-service, en utilisant les technologies de modélisation et de visualisation de Microsoft Power BI ou Microsoft Excel. L’analyse et les rapports peuvent aussi prendre la forme d’une exploration interactive des données par les scientifiques de données ou les analystes de données. Pour ces scénarios, plusieurs services Azure prennent en charge les blocs-notes analytiques, tels que Jupyter, ce qui permet à ces utilisateurs de tirer parti de leurs connaissances avec Python ou R. Pour l’exploration de données à grande échelle, vous pouvez utiliser Microsoft R Server seul ou avec Spark.

- **Orchestration** : La plupart des solutions Big Data consistent en des opérations de traitement de données répétées, encapsulées dans des workflows, qui transforment les données source, déplacent les données entre plusieurs sources et récepteurs, chargent les données traitées dans un magasin de données analytique, ou envoient les résultats directement à un rapport ou à un tableau de bord. Pour automatiser ces workflows, vous pouvez utiliser une technologie d’orchestration telle qu’Azure Data Factory ou Apache Oozie avec Sqoop.

Azure inclut de nombreux services qui peuvent être utilisés dans une architecture Big Data. Il existe deux grandes catégories :

- Services managés, y compris Azure Data Lake Store, Azure Data Lake Analytics, Azure Data Warehouse, Azure Stream Analytics, Azure Event Hub, Azure IoT Hub et Azure Data Factory.
- Technologies open source basées sur une plateforme Apache Hadoop, y compris HDFS, HBase, Hive, Pig, Spark, Storm, Oozie, Sqoop et Kafka. Ces technologies sont disponibles sur Azure dans le service Azure HDInsight.

Ces options ne sont pas mutuellement exclusives, et de nombreuses solutions combinent les technologies open source avec les services Azure.

## <a name="when-to-use-this-architecture"></a>Quand utiliser cette architecture

Envisagez ce style d’architecture pour répondre aux besoins suivants :

- Stocker et traiter des données dans des volumes trop vastes pour une base de données traditionnelle.
- Transformer des données non structurées en vue d’une analyse et de la création de rapports.
- Capturer, traiter et analyser des flux de données indépendants en temps réel ou avec une faible latence.
- Utilisez Azure Machine Learning ou Microsoft Cognitive Services.

## <a name="benefits"></a>Avantages

- **Choix de technologie**. Vous pouvez combiner des services managés Azure et des technologies Apache dans des clusters HDInsight afin de tirer parti des compétences ou des investissements technologiques existants.
- **Performances via le parallélisme**. Les solutions Big Data tirent parti du parallélisme, ce qui en fait des solutions hautes performances qui s’adaptent à des volumes importants de données.
- **Mise à l’échelle élastique**. Tous les composants de l’architecture Big Data prennent en charge l’approvisionnement de montée en puissance, afin que vous puissiez ajuster votre solution aux petites ou aux grandes charges de travail et ne payer que pour les ressources que vous utilisez.
- **Interopérabilité avec les solutions existantes**. Les composants de l’architecture Big Data sont également utilisés pour les solutions de décisionnel d’entreprise et de traitement IoT, ce qui vous permet de créer une solution intégrée pour toutes les charges de données.

## <a name="challenges"></a>Défis

- **Complexité** : Les solutions Big Data peuvent être extrêmement complexes, avec nombreux composants pour gérer l’ingestion de données à partir de plusieurs sources de données. Il peut être difficile de générer, tester et dépanner des processus Big Data. En outre, pour optimiser les performances, il faut parfois utiliser de nombreux paramètres de configuration sur plusieurs systèmes.
- **Compétences**. De nombreuses technologies Big Data sont hautement spécialisées et utilisent des infrastructures et des langues assez rares pour des architectures d’applications plus générales. En revanche, les technologies Big Data font évoluer de nouvelles API qui s’appuient sur plusieurs langages établis. Par exemple, le langage U-SQL dans Azure Data Lake Analytics repose sur une combinaison de Transact-SQL et C#. De même, les API basées sur SQL sont disponibles pour Hive, HBase et Spark.
- **Maturité de la technologie**. La plupart des technologies utilisées en Big Data sont en pleine évolution. Tandis que les technologies Hadoop de base comme Hive et Pig se sont stabilisées, de nouvelles technologies, telles que Spark, introduisent des modifications et des améliorations importantes à chaque nouvelle version. Les services managés tels qu’Azure Data Lake Analytics et Azure Data Factory sont relativement récents par rapport aux autres services Azure et évolueront probablement au fil du temps.
- **Sécurité**. Les solutions Big Data s’appuient généralement sur le stockage de toutes les données statiques dans un lac de données centralisé. La sécurisation de l’accès à ces données peut être difficile, en particulier lorsque les données doivent être ingérées et consommées par plusieurs applications et plateformes.

## <a name="best-practices"></a>Bonnes pratiques

- **Tirer parti du parallélisme**. La plupart des technologies de traitement Big Data distribuent la charge sur plusieurs unités de traitement. Cela nécessite la création et le stockage de fichiers de données statiques dans un format fractionnable. Les systèmes de fichiers distribués tels que HDFS permettent d’optimiser les performances de lecture et d’écriture. Le traitement réel est effectué par plusieurs nœuds de cluster en parallèle, ce qui réduit globalement les temps de travail.

- **Partitionner les données**. En général, le traitement par lots se produit selon une planification périodique &mdash;par exemple, selon une fréquence hebdomadaire ou mensuelle. Partitionnez les fichiers de données et les structures de données telles que les tables en fonction de périodes qui correspondent à la planification du traitement. Cela simplifie l’ingestion de données et la planification des travaux, et facilite la résolution des problèmes. En outre, le partitionnement des tables utilisées dans les requêtes Hive, U-SQL ou SQL peut améliorer considérablement les performances des requêtes.

- **Appliquer une sémantique de schéma à la lecture**. Un lac de données permet de combiner le stockage de fichiers dans plusieurs formats, qu’ils soient structurés, semi-structurés ou non structurés. Utilisez une sémantique de *schéma à la lecture*, qui projette un schéma sur les données lors de leur traitement, et non lors du stockage. Cela confère une grande souplesse à la solution et empêche les goulots d’étranglement pendant l’ingestion des données, causés par la validation des données et la vérification du type.

- **Traiter les données sur place**. Les solutions BI traditionnelles utilisent souvent un processus ETL (extraction, transformation et chargement) pour déplacer des données dans un entrepôt de données. Avec de plus grands volumes de données et une plus grande variété de formats, les solutions Big Data utilisent généralement des variantes de processus ETL, comme le processus TEL (transformation, extraction et chargement). Avec cette approche, les données sont traitées dans le magasin de données distribué, en leur donnant la structure requise avant de déplacer les données transformées dans un magasin de données analytique.

- **Équilibrer les coûts d’utilisation et de temps**. Pour les travaux de traitement par lots, il est important de prendre en compte deux facteurs : le coût unitaire de nœuds de calcul et le coût par minute de l’utilisation de ces nœuds pour effectuer le travail. Par exemple, un programme de traitement par lots peut prendre les huit heures avec quatre nœuds de cluster. Toutefois, il peut arriver que le programme utilise les quatre nœuds uniquement pendant les deux premières heures et, par la suite, qu’il n’ait besoin que de deux nœuds. Dans ce cas, exécuter l’intégralité du programme sur deux nœuds augmenterait la durée totale du travail, sans pour autant la doubler. Le coût total serait donc inférieur. Dans certains scénarios métier, une durée de traitement plus longue peut être préférable au coût plus élevé lié à l’utilisation de ressources de cluster sous-exploitées.

- **Séparer les ressources de cluster**. Lorsque vous déployez des clusters HDInsight, vous obtenez normalement de meilleures performances en approvisionnant des ressources de cluster distinctes pour chaque type de charge de travail. Par exemple, bien que les clusters Spark incluent Hive, si vous avez besoin d’effectuer un traitement intensif avec Hive et Spark, vous devez envisager de déployer des clusters dédiés distincts pour Spark et Hadoop. De même, si vous utilisez HBase et Storm pour le traitement de flux de données à faible latence et Hive pour le traitement par lots, envisagez des clusters distincts pour Storm, HBase et Hadoop.

- **Orchestrer l’ingestion de données**. Dans certains cas, les applications métier existantes peuvent écrire des fichiers de données destinés à un traitement par lots directement dans les conteneurs d’objets blob de stockage Azure, où ils peuvent être consommés par HDInsight ou Azure Data Lake Analytics. Toutefois, vous devez souvent orchestrer l’ingestion de données dans le lac de données à partir de sources de données externes ou locales. Utilisez un pipeline ou un workflow d’orchestration, tels que ceux pris en charge par Azure Data Factory ou Oozie, pour effectuer cette opération de manière prévisible et facile à gérer de manière centralisée.

- **Nettoyer les données sensibles tôt**. Le workflow d’ingestion de données doit nettoyer les données sensibles tôt dans le processus pour éviter de les stocker dans le lac de données.

## <a name="iot-architecture"></a>Architecture IoT

L’Internet des objets (IoT) est un sous-ensemble spécialisé des solutions big data. Le diagramme suivant présente une architecture logique possible pour IoT. Le diagramme met en avant les composants de diffusion d’événements de l’architecture.

![Diagramme d’une architecture IoT](./images/iot.png)

La **passerelle cloud** ingère les événements d’appareils à la limite du cloud en utilisant un système de messagerie fiable et à faible latence.

Les appareils peuvent envoyer les événements directement à la passerelle cloud ou via une **passerelle de champ**. Une passerelle de champ est un appareil ou un logiciel spécialisé, généralement colocalisée avec les appareils, qui reçoit les événements et les transfère à la passerelle cloud. La passerelle de champ peut aussi prétraiter les événements d’appareils bruts, remplissant des fonctions de filtrage, d’agrégation ou de transformation de protocole.

Après ingestion, les événements transitent par un ou plusieurs **processeurs de flux** qui peuvent acheminer les données (par exemple, vers le stockage) ou procéder à l’analytique et autres traitements.

Voici quelques types de traitement courants. (Cette liste n’est certainement pas exhaustive.)

- Écriture de données d’événement dans un stockage froid pour archivage ou traitement analytique par lots.

- Analytique de séquence à chaud (« hot path analytics »), avec une analyse du flux d’événements en (quasi) temps réel, pour détecter les anomalies, reconnaître les modèles dans des fenêtres de temps glissantes ou déclencher des alertes quand une condition spécifique est rencontrée dans le flux.

- Gestion de types de messages d’appareils non liés à la télémétrie, tels que les notifications et les alarmes.

- Apprentissage automatique.

Les cadres gris représentent les composants d’un système IoT qui ne sont pas directement liés à la diffusion d’événements, mais qui sont inclus ici par souci d’exhaustivité.

- Le **registre d’appareils** est une base de données qui recense les appareils provisionnés, avec notamment leur ID et les métadonnées associées usuelles, telles que l’emplacement.

- L’**API de provisionnement** est une interface externe commune pour provisionner et inscrire de nouveaux appareils.

- Certaines solutions IoT autorisent l’envoi de **messages de commande et de contrôle** aux appareils.

> Cette section a fourni une vue d’ensemble globale d’IoT et bon nombre de subtilités et d’écueils sont à prendre en considération. Pour obtenir une description et une présentation plus détaillées d’une architecture de référence, consultez [Microsoft Azure IoT Reference Architecture][iot-ref-arch] (PDF téléchargeable).

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les [architectures big data](../../data-guide/big-data/index.md).

 <!-- links -->

[iot-ref-arch]: https://azure.microsoft.com/updates/microsoft-azure-iot-reference-architecture-available/
