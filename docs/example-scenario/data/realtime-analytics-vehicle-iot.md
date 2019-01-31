---
title: Ingestion et traitement de données IoT automobiles en temps réel
titleSuffix: Azure Example Scenarios
description: Ingérez et traitez les données de véhicule en temps réel à l’aide de l’IoT.
author: msdpalam
ms.date: 09/12/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: fasttrack, IoT
social_image_url: /azure/architecture/example-scenario/data/media/architecture-realtime-analytics-vehicle-data1.png
ms.openlocfilehash: 95a59ed84870c9ce9d3c6637d9ba56fcd6935b53
ms.sourcegitcommit: 3b15d65e7c35a19506e562c444343f8467b6a073
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/25/2019
ms.locfileid: "54908398"
---
# <a name="ingestion-and-processing-of-real-time-automotive-iot-data"></a>Ingestion et traitement de données IoT automobiles en temps réel

Cet exemple de scénario crée un pipeline permettant l’ingestion et le traitement en temps réel des données des appareils IoT (en général des capteurs) sur une plateforme d’analyse Big Data dans Azure. Les plateformes d’ingestion et de traitement télématiques pour les véhicules sont essentielles à la création de solutions automobiles connectées. Ce scénario particulier a été conçu pour les systèmes d’ingestion et de traitement de la télématique automobile. Cependant, les modèles de conception sont applicables à de nombreux domaines qui se servent de capteurs pour gérer et surveiller des systèmes complexes dans des secteurs tels que les bâtiments intelligents, les communications, la fabrication, la vente au détail et la santé.

Cet exemple illustre un pipeline d’ingestion et de traitement de données en temps réel pour les messages provenant des appareils IoT installés dans les véhicules. Les appareils et les capteurs IoT génèrent des millions de messages (ou d’événements). En capturant et en analysant ces messages, nous pouvons identifier des informations utiles et prendre les mesures appropriées. Par exemple, en capturant les messages en temps réel des appareils télématiques des véhicules qui en sont équipés, nous pouvons surveiller en temps réel l’emplacement des véhicules, planifier des itinéraires optimisés, aider les conducteurs et appuyer des secteurs liés à la télématique comme les assurances automobiles.

Dans cet exemple d’illustration, une entreprise de construction automobile souhaite créer un système en temps réel d’ingestion et de traitement des messages provenant des appareils télématiques. Les objectifs de l’entreprise sont les suivants :

- Ingérer et stocker les données en temps réel des capteurs et des appareils des véhicules.
- Analyser les messages pour comprendre l’emplacement des véhicules et d’autres informations provenant de différents types de capteurs (comme les capteurs liés au moteur et aux conditions environnementales).
- Stocker les données après analyse en vue de traitements ultérieurs afin de fournir des informations exploitables (par exemple, les agences d’assurance peuvent souhaiter savoir ce qui s’est passé au cours d’un accident)

## <a name="relevant-use-cases"></a>Cas d’usage appropriés

Les autres cas d’usage appropriés sont les suivants :

- Rappels et alertes de maintenance des véhicules.
- Services basés sur l’emplacement des passagers du véhicule (c’est-à-dire, secours).
- Véhicules autonomes (conduite automatique).

## <a name="architecture"></a>Architecture

![Capture d'écran](media/architecture-realtime-analytics-vehicle-data1.png)

Dans une implémentation classique de pipeline de traitement de Big Data, les données circulent de gauche à droite. Dans ce pipeline de traitement de Big Data en temps réel, les données circulent dans la solution de la manière suivante :

1. Les événements générés à partir des sources de données IoT sont envoyés à la couche d’ingestion de flux via Azure HDInsight Kafka en tant que flux de messages. HDInsight Kafka stocke les flux de données dans les rubriques pour une durée configurable.
2. Le consommateur Kafka, Azure Databricks, récupère le message en temps réel dans la rubrique Kafka afin de traiter les données en fonction de la logique métier et ensuite de les envoyer à la couche Serving à des fins de stockage.
3. Les services de stockage en aval, comme Azure Cosmos DB, Azure SQL Data Warehouse ou Azure SQL DB, constituent alors une source de données pour la couche de présentation et d’action.
4. Les analystes métier peuvent utiliser Microsoft Power BI pour analyser les données stockées. D’autres applications peuvent également être créées sur la couche de service. Par exemple, nous pouvons exposer des API basées sur les données de la couche de service pour des usages tiers.

### <a name="components"></a>Composants

Les événements générés par les appareils IoT (données ou messages) sont ingérés, traités, puis stockés à des fins d’analyse, de présentation et d’action ultérieures à l’aide des composants Azure suivants :

- [Apache Kafka sur HDInsight](/azure/hdinsight/kafka/apache-kafka-introduction) se trouve dans la couche d’ingestion. Les données sont enregistrées dans la rubrique Kafka à l’aide d’une API de producteur Kafka.
- [Azure Databricks](/services/databricks) se trouve dans la couche de transformation et d’analyse. Les blocs-notes Databricks implémentent une API consommateur Kafka afin de lire les données de la rubrique Kafka.
- [Azure Cosmos DB](/services/cosmos-db), [Azure SQL Database](/azure/sql-database/sql-database-technical-overview), et Azure SQL Data Warehouse se trouvent dans la couche de stockage Serving, où Azure Databricks peut écrire les données avec des connecteurs de données.
- [Azure SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is) est un système distribué permettant de stocker et d’analyser de grands ensembles de données. Son recours à un traitement parallèle massif (MPP) lui permet d’exécuter des analyses hautes performances.
- [Power BI](https://docs.microsoft.com/power-bi) est une suite d’outils d’analyse métier pour analyser les données et partager les informations. Power BI peut interroger un modèle sémantique stocké dans Analysis Services ou interroger directement SQL Data Warehouse.
- [Azure Active Directory (Azure AD)](/azure/active-directory) authentifie les utilisateurs lors de la connexion à [Azure Databricks](https://azure.microsoft.com/services/databricks). Si nous créons un cube dans [Analysis Services](/azure/analysis-services) basé sur le modèle Azure de données SQL Data Warehouse, nous pouvons nous servir d’AAD pour établir la connexion au serveur Analysis Services par le biais de Power BI. Data Factory peut également utiliser Azure AD pour s’authentifier auprès de SQL Data Warehouse via un principal de service ou une identité MSI (Managed Service Identity).
- Il est possible d’utiliser [Azure App Services](/azure/app-service/app-service-web-overview), notamment [API App](/services/app-service/api) pour présenter des données à des tiers en fonction des données stockées dans la couche Serving.

## <a name="alternatives"></a>Autres solutions

![Capture d'écran](media/architecture-realtime-analytics-vehicle-data2.png)

Il est possible d’implémenter un pipeline de Big Data plus généralisé avec d’autres composants Azure.

- Dans la couche d’ingestion de flux, nous pouvons utiliser [IoT Hub](https://azure.microsoft.com/services/iot-hub) ou [Event Hub](https://azure.microsoft.com/services/event-hubs), au lieu de [HDInsight Kafka](/azure/hdinsight/kafka/apache-kafka-introduction) pour ingérer les données.
- Dans la couche de transformation et d’analyse, nous pouvons utiliser [HDInsight Storm](/azure/hdinsight/storm/apache-storm-overview), [HDInsight Spark](/azure/hdinsight/spark/apache-spark-overview) ou [Azure Stream Analytics](https://azure.microsoft.com/services/stream-analytics).
- La fonctionnalité [Analysis Services](/azure/analysis-services) fournit un modèle sémantique de vos données. Elle permet également de renforcer les performances du système lors de l’analyse de vos données. Vous pouvez créer le modèle à partir des données Azure DW.

## <a name="considerations"></a>Considérations

Les technologies de cette architecture ont été choisies en fonction de l’échelle nécessaire au traitement des événements, du SLA des services, de la gestion des coûts et de la facilité de la gestion des composants.

- La fonctionnalité [HDInsight Kafka](/azure/hdinsight/kafka/apache-kafka-introduction) managée est livrée avec un SLA à 99,9 % intégré à Azure Managed Disks
- La fonctionnalité [Azure Databricks](/azure/azure-databricks/what-is-azure-databricks) est optimisée de A à Z pour assurer des performances et une rentabilité optimales dans le cloud. Databricks Runtime ajoute plusieurs fonctionnalités essentielles aux charges de travail Apache Spark qui peuvent augmenter les performances et réduire les coûts d’un facteur allant de 10 à 100 lorsque vous utilisez Azure, notamment :
- Azure Databricks s’intègre en profondeur aux magasins et aux bases de données Azure : [Azure SQL Data Warehouse](/azure/sql-data-warehouse), [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db), [Azure Data Lake Storage](https://azure.microsoft.com/services/storage/data-lake-storage) et [Stockage Blob Azure](https://azure.microsoft.com/services/storage/blobs)
  - Mise à l’échelle et arrêt automatique des clusters Spark pour réduire les coûts de façon automatique.
  - Optimisations des performances, notamment la mise en cache, l’indexation et l’optimisation avancée des requêtes, permettant d’améliorer les performances de 10 à 100 fois par rapport aux déploiements traditionnels Apache Spark dans les environnements cloud ou locaux.
  - L’intégration à Azure Active Directory vous permet d’appliquer des solutions Azure complètes avec Azure Databricks.
  - L’accès basé sur les rôles dans Azure Databricks permet d’obtenir des autorisations d’utilisateur fines pour les blocs-notes, les clusters, les travaux et les données.
  - Fourni avec des SLA de classe Entreprise.
- Azure Cosmos DB est un service de base de données multimodèle mondialement distribué de Microsoft. Le service Azure Cosmos DB repose sur une distribution mondiale et sur une scalabilité horizontale. Il offre une distribution mondiale clé en main sur un nombre illimité de régions Azure grâce à une scalabilité et à une réplication transparentes de vos données, quel que soit l’emplacement de vos utilisateurs. Bénéficiez d’une scalabilité élastique du débit et du stockage dans le monde entier et payez uniquement le débit et le stockage dont vous avez besoin.
- L’architecture de traitement massivement parallèle de SQL Data Warehouse assure évolutivité et hautes performances.
- Azure SQL Data Warehouse vous garantit des SLA et des pratiques recommandées pour assurer une haute disponibilité.
- Lorsque l’activité d’analyse est faible, l’entreprise peut mettre à l’échelle Azure SQL Data Warehouse à la demande de façon à réduire ou même à interrompre le calcul afin de réduire les coûts.
- Le modèle de sécurité d’Azure SQL Data Warehouse assure la sécurité des connexions, l’authentification et l’autorisation via l’authentification Azure AD ou SQL Server, et le chiffrement.

## <a name="pricing"></a>Tarifs

Consultez la [tarification Azure Databricks](https://azure.microsoft.com/pricing/details/databricks), la [tarification Azure HDInsight](https://azure.microsoft.com/pricing/details/hdinsight) et l’[exemple de tarification d’un scénario d’entreposage de données](https://azure.com/e/b798fb70c53e4dd19fdeacea4db78276) via la  calculatrice de tarification Azure. Ajustez les valeurs pour déterminer l’incidence de vos besoins sur vos coûts.

- [Azure HDInsight](/azure/hdinsight) est un service cloud entièrement géré, qui simplifie, accélère et rentabilise le traitement de quantités importantes de données
- [Azure Databricks](https://azure.microsoft.com/services/databricks) offre deux charges de travail distinctes sur plusieurs [instances de machines virtuelles](https://azure.microsoft.com/pricing/details/databricks/#instances) adaptées à votre flux de travail Analytique données. Avec la charge de travail Engineering données, les ingénieurs de données peuvent facilement créer et exécuter des travaux. De plus, la charge de travail Analytique données aide les scientifiques des données à explorer, à visualiser, à manipuler et à partager facilement des données et informations de façon interactive.
- [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db) garantit des latences en millisecondes à un chiffre au 99e centile partout dans le monde, offre de [multiples modèles de cohérence bien définis](/azure/cosmos-db/consistency-levels) afin d’affiner les performances, et garantit une haute disponibilité grâce à des fonctionnalités d’hébergement multiple, le tout régi par des [contrats SLA](https://azure.microsoft.com/support/legal/sla/cosmos-db) comptant parmi les plus complets et les plus pertinents du secteur.
- [Azure SQL Data Warehouse](https://azure.microsoft.com/pricing/details/sql-data-warehouse/gen2) vous permet de mettre à l’échelle vos niveaux de calcul et de stockage de façon indépendante. Les ressources de calcul sont facturées à l’heure, et vous pouvez mettre ces ressources à l’échelle ou en pause à la demande. Les ressources de stockage sont facturées au téraoctet. Vos coûts augmentent donc en fonction du volume de données ingéré.
- [Analysis Services](https://azure.microsoft.com/pricing/details/analysis-services) est disponible pour les niveaux développeur, de base et standard. La tarification des instances est établie en fonction des unités de traitement des requêtes (QPU) et de la mémoire disponible. Pour diminuer vos coûts, réduisez le nombre de requêtes exécutées, la quantité de données traitées et leur fréquence d’exécution.
- [Power BI](https://powerbi.microsoft.com/pricing) offre différentes options de produit selon les besoins. [Power BI Embedded](https://azure.microsoft.com/pricing/details/power-bi-embedded) offre une option Azure permettant d’intégrer la fonctionnalité Power BI dans vos applications. L’exemple de tarification ci-dessus comprend une instance Power BI Embedded.

## <a name="next-steps"></a>Étapes suivantes

- Consultez l’architecture de référence [Analyses en temps réel](https://azure.microsoft.com/solutions/architecture/real-time-analytics) qui comprend un flux de pipeline Big Data.
- Consultez l’architecture de référence [Analyses avancées en Big Data](https://azure.microsoft.com/solutions/architecture/advanced-analytics-on-big-data)pour avoir un aperçu de la façon dont les différents composants Azure permettent de créer un pipeline Big Data.
- Consultez la documentation [Traitement en temps réel](/azure/architecture/data-guide/big-data/real-time-processing) pour obtenir un aperçu rapide de la façon dont les différents composants Azure permettent de traiter les flux de données en temps réel.
- Retrouvez des conseils architecturaux complets sur les pipelines de données, l’entreposage des données, le traitement analytique en ligne (OLAP) et le Big Data dans le [guide sur l’architecture des données Azure](/azure/architecture/data-guide).
