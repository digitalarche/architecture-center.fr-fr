---
title: "Sélectionner une technologie de traitement de flux"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 23d9849c14964b0905300f191a41084b589fd127
ms.sourcegitcommit: 943e671a8d522cef5ddc8c6e04848134b03c2de4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/05/2018
---
# <a name="choosing-a-stream-processing-technology-in-azure"></a>Sélectionner une technologie de traitement de flux dans Azure

Cet article compare les choix technologiques pour le traitement de flux en temps réel dans Azure.

Le traitement de flux en temps réel utilise les messages provenant d’une file d’attente ou d’un stockage de fichiers, traite les messages, puis transfère le résultat à une autre file d’attente de messages, magasin de fichiers ou base de données. Ce traitement peut inclure l’interrogation, le filtrage et l’agrégation de messages. Les moteurs de traitement de flux doivent être en mesure d’utiliser des flux infinis de données et de produire des résultats avec une latence minimale. Pour plus d’informations, consultez [Traitement en temps réel](../scenarios/real-time-processing.md).

## <a name="what-are-your-options-when-choosing-a-technology-for-real-time-processing"></a>Quelles sont vos options quant au choix d’une technologie de traitement en temps réel ?
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
| Entrées | [Entrées Stream Analytics](/azure/stream-analytics/stream-analytics-define-inputs)  | Event Hubs, IoT Hub, Kafka, HDFS, Storage Blobs, Azure Data Lake Store  | Event Hubs, IoT Hub, Kafka, HDFS, Storage Blobs, Azure Data Lake Store  | Event Hubs, IoT Hub, Storage Blobs, Azure Data Lake Store  | [Liaisons prises en charge](/azure/azure-functions/functions-triggers-bindings#supported-bindings) | Service Bus, files d’attente de stockage, Storage Blobs, Event Hubs, WebHooks, Cosmos DB, fichiers |
| Récepteurs |  [Sorties Stream Analytics](/azure/stream-analytics/stream-analytics-define-outputs) | HDFS, Kafka, Storage Blobs, Azure Data Lake Store, Cosmos DB | HDFS, Kafka, Storage Blobs, Azure Data Lake Store, Cosmos DB | Event Hubs, Service Bus, Kafka | [Liaisons prises en charge](/azure/azure-functions/functions-triggers-bindings#supported-bindings) | Service Bus, files d’attente de stockage, Storage Blobs, Event Hubs, WebHooks, Cosmos DB, fichiers | 

### <a name="processing-capabilities"></a>Fonctionnalités de traitement
| | Azure Stream Analytics | HDInsight avec Spark Streaming | Apache Spark dans Azure Databricks | HDInsight avec Storm | Azure Functions | Azure App Service WebJobs |
| --- | --- | --- | --- | --- | --- | --- | 
| Prise en charge temporelle/fenêtrage intégrée | OUI | OUI | OUI | OUI | Non  | Non  |
| Formats de données d’entrée | Avro, JSON ou CSV, encodage UTF-8 | Tout format à base de code personnalisé | Tout format à base de code personnalisé | Tout format à base de code personnalisé | Tout format à base de code personnalisé | Tout format à base de code personnalisé |
| Extensibilité | [Interroger des partitions](/azure/stream-analytics/stream-analytics-parallelization) | Limitée par la taille du cluster | Limitée par la configuration de la mise à l’échelle du cluster Databricks | Limitée par la taille du cluster | Jusqu'à 200 instances d’application de fonction traitées en parallèle | Limitée par la capacité du plan App Service | 
| Prise en charge de l’arrivée tardive et de la gestion des événements de manière désordonnée | OUI | OUI | OUI | OUI | Non  | Non  |

Voir aussi :

- [Choisir une technologie d’ingestion de messages en temps réel](./real-time-ingestion.md)
- [Comparaison d’Apache Storm et d’Azure Stream Analytics](/azure/stream-analytics/stream-analytics-comparison-storm)
- [Traitement en temps réel](../scenarios/real-time-processing.md)
