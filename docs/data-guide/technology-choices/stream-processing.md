---
title: Sélectionner une technologie de traitement de flux
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: 342e44d960682c72901a7482caaf328514eb73d8
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54486113"
---
# <a name="choosing-a-stream-processing-technology-in-azure"></a>Sélectionner une technologie de traitement de flux dans Azure

Cet article compare les choix technologiques pour le traitement de flux en temps réel dans Azure.

Le traitement de flux en temps réel utilise les messages provenant d’une file d’attente ou d’un stockage de fichiers, traite les messages, puis transfère le résultat à une autre file d’attente de messages, magasin de fichiers ou base de données. Ce traitement peut inclure l’interrogation, le filtrage et l’agrégation de messages. Les moteurs de traitement de flux doivent être en mesure d’utiliser des flux infinis de données et de produire des résultats avec une latence minimale. Pour plus d’informations, consultez [Traitement en temps réel](../big-data/real-time-processing.md).

<!-- markdownlint-disable MD026 -->

## <a name="what-are-your-options-when-choosing-a-technology-for-real-time-processing"></a>Quelles sont vos options quant au choix d’une technologie de traitement en temps réel ?

<!-- markdownlint-enable MD026 -->

Dans Azure, tous les magasins de données suivants répondent aux principales exigences de prise en charge du traitement en temps réel :

- [Azure Stream Analytics](/azure/stream-analytics/)
- [HDInsight avec Spark Streaming](/azure/hdinsight/spark/apache-spark-streaming-overview)
- [Apache Spark dans Azure Databricks](/azure/azure-databricks/)
- [HDInsight avec Storm](/azure/hdinsight/storm/apache-storm-overview)
- [Azure Functions](/azure/azure-functions/functions-overview)
- [Azure App Service WebJobs](/azure/app-service/web-sites-create-web-jobs)

## <a name="key-selection-criteria"></a>Critères de sélection principaux

Pour les scénarios de traitement en temps réel, commencez par choisir le service adapté à vos besoins en répondant à ces questions :

- Préférez-vous une approche déclarative ou impérative pour élaborer la logique de traitement du flux ?

- Avez-vous besoin d’une prise en charge intégrée pour le traitement temporel ou le fenêtrage ?

- Vos données arrivent-elles dans des formats autres que Avro, JSON ou CSV ? Si oui, choisissez des options prenant en charge n’importe quel format à base d’un code personnalisé.

- Devez-vous mettre à l’échelle votre traitement au-delà de 1 Go/s ? Si oui, choisissez des options qui évoluent avec la taille du cluster.

## <a name="capability-matrix"></a>Matrice des fonctionnalités

Les tableaux suivants résument les principales différences entre les fonctionnalités.

### <a name="general-capabilities"></a>Fonctionnalités générales

| | Azure Stream Analytics | HDInsight avec Spark Streaming | Apache Spark dans Azure Databricks | HDInsight avec Storm | Azure Functions | Azure App Service WebJobs |
| --- | --- | --- | --- | --- | --- | --- |
| Programmabilité | Langage de requête Stream Analytics, JavaScript | Scala, Python, Java | Scala, Python, Java, R | Java, C# | C#, F#, Node.js | C#, Node.js, PHP, Java, Python |
| Paradigme de programmation | Déclarative | À la fois déclaratif et impératif | À la fois déclaratif et impératif | Impérative | Impérative | Impérative |
| Modèle de tarification | [Unités de streaming](https://azure.microsoft.com/pricing/details/stream-analytics/) | Par heure de cluster | [Unités Databricks](https://azure.microsoft.com/pricing/details/databricks/) | Par heure de cluster | Par exécution de fonction et consommation de ressources | Par heure de plan App Service |  

### <a name="integration-capabilities"></a>Fonctionnalités d’intégration

| | Azure Stream Analytics | HDInsight avec Spark Streaming | Apache Spark dans Azure Databricks | HDInsight avec Storm | Azure Functions | Azure App Service WebJobs |
| --- | --- | --- | --- | --- | --- | --- |
| Entrées | Azure Event Hubs, Azure IoT Hub, stockage d’objets blob Azure  | Event Hubs, IoT Hub, Kafka, HDFS, Storage Blobs, Azure Data Lake Store  | Event Hubs, IoT Hub, Kafka, HDFS, Storage Blobs, Azure Data Lake Store  | Event Hubs, IoT Hub, Storage Blobs, Azure Data Lake Store  | [Liaisons prises en charge](/azure/azure-functions/functions-triggers-bindings#supported-bindings) | Service Bus, files d’attente de stockage, Storage Blobs, Event Hubs, WebHooks, Cosmos DB, fichiers |
| Récepteurs |  Azure Data Lake Store, Azure SQL Database, objets blob de stockage, Event Hubs, Power BI, service de Table, files d’attente Service Bus, rubriques Service Bus, Cosmos DB, Azure Functions  | HDFS, Kafka, Storage Blobs, Azure Data Lake Store, Cosmos DB | HDFS, Kafka, Storage Blobs, Azure Data Lake Store, Cosmos DB | Event Hubs, Service Bus, Kafka | [Liaisons prises en charge](/azure/azure-functions/functions-triggers-bindings#supported-bindings) | Service Bus, files d’attente de stockage, Storage Blobs, Event Hubs, WebHooks, Cosmos DB, fichiers |

### <a name="processing-capabilities"></a>Fonctionnalités de traitement

| | Azure Stream Analytics | HDInsight avec Spark Streaming | Apache Spark dans Azure Databricks | HDInsight avec Storm | Azure Functions | Azure App Service WebJobs |
| --- | --- | --- | --- | --- | --- | --- |
| Prise en charge temporelle/fenêtrage intégrée | Oui | OUI | OUI | Oui | Non  | Non  |
| Formats de données d’entrée | Avro, JSON ou CSV, encodage UTF-8 | Tout format à base de code personnalisé | Tout format à base de code personnalisé | Tout format à base de code personnalisé | Tout format à base de code personnalisé | Tout format à base de code personnalisé |
| Extensibilité | [Interroger des partitions](/azure/stream-analytics/stream-analytics-parallelization) | Limitée par la taille du cluster | Limitée par la configuration de la mise à l’échelle du cluster Databricks | Limitée par la taille du cluster | Jusqu'à 200 instances d’application de fonction traitées en parallèle | Limitée par la capacité du plan App Service |
| Prise en charge de l’arrivée tardive et de la gestion des événements de manière désordonnée | Oui | OUI | OUI | Oui | Non  | Non  |

Voir aussi :

- [Choisir une technologie d’ingestion de messages en temps réel](./real-time-ingestion.md)
- [Traitement en temps réel](../big-data/real-time-processing.md)
