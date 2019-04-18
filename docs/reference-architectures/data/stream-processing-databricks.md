---
title: Traitement de flux de données avec Azure Databricks
titleSuffix: Azure Reference Architectures
description: Créez un pipeline de traitement de flux de bout en bout dans Azure à l’aide d’Azure Databricks.
author: petertaylor9999
ms.date: 11/30/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18
ms.openlocfilehash: 3d109cb830b7dfc8c3d4de0e654f9d8667acf101
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/18/2019
ms.locfileid: "59740457"
---
# <a name="create-a-stream-processing-pipeline-with-azure-databricks"></a><span data-ttu-id="ad94d-103">Créer un pipeline de traitement de flux avec Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="ad94d-103">Create a stream processing pipeline with Azure Databricks</span></span>

<span data-ttu-id="ad94d-104">Cette architecture de référence présente un pipeline de [traitement de flux](/azure/architecture/data-guide/big-data/real-time-processing) de bout en bout.</span><span class="sxs-lookup"><span data-stu-id="ad94d-104">This reference architecture shows an end-to-end [stream processing](/azure/architecture/data-guide/big-data/real-time-processing) pipeline.</span></span> <span data-ttu-id="ad94d-105">Ce type de pipeline comprend quatre étapes : ingestion, traitement, stockage, analyse et création de rapports.</span><span class="sxs-lookup"><span data-stu-id="ad94d-105">This type of pipeline has four stages: ingest, process, store, and analysis and reporting.</span></span> <span data-ttu-id="ad94d-106">Pour cette architecture de référence, le pipeline ingère des données issues de deux sources, effectue une jonction sur les enregistrements apparentés de chaque flux, enrichit le résultat et calcule une moyenne en temps réel.</span><span class="sxs-lookup"><span data-stu-id="ad94d-106">For this reference architecture, the pipeline ingests data from two sources, performs a join on related records from each stream, enriches the result, and calculates an average in real time.</span></span> <span data-ttu-id="ad94d-107">Les résultats sont stockés en vue d’une analyse plus approfondie.</span><span class="sxs-lookup"><span data-stu-id="ad94d-107">The results are stored for further analysis.</span></span> <span data-ttu-id="ad94d-108">[**Déployez cette solution**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="ad94d-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

![Architecture de référence pour le traitement de flux avec Azure Databricks](./images/stream-processing-databricks.png)

<span data-ttu-id="ad94d-110">**Scénario** : Une compagnie de taxis collecte des données sur chaque trajet effectué en taxi.</span><span class="sxs-lookup"><span data-stu-id="ad94d-110">**Scenario**: A taxi company collects data about each taxi trip.</span></span> <span data-ttu-id="ad94d-111">Pour ce scénario, nous partons du principe que deux périphériques distincts envoient des données.</span><span class="sxs-lookup"><span data-stu-id="ad94d-111">For this scenario, we assume there are two separate devices sending data.</span></span> <span data-ttu-id="ad94d-112">Le taxi est équipé d’un compteur qui envoie les informations suivantes sur chaque course : durée, distance et lieux de prise en charge et de dépose.</span><span class="sxs-lookup"><span data-stu-id="ad94d-112">The taxi has a meter that sends information about each ride &mdash; the duration, distance, and pickup and dropoff locations.</span></span> <span data-ttu-id="ad94d-113">Un autre périphérique accepte les paiements des clients et envoie des données sur les tarifs.</span><span class="sxs-lookup"><span data-stu-id="ad94d-113">A separate device accepts payments from customers and sends data about fares.</span></span> <span data-ttu-id="ad94d-114">Afin de déterminer les tendances des usagers, la compagnie de taxis souhaite calculer le pourboire moyen par mile parcouru, en temps réel, pour chaque quartier.</span><span class="sxs-lookup"><span data-stu-id="ad94d-114">To spot ridership trends, the taxi company wants to calculate the average tip per mile driven, in real time, for each neighborhood.</span></span>

## <a name="architecture"></a><span data-ttu-id="ad94d-115">Architecture</span><span class="sxs-lookup"><span data-stu-id="ad94d-115">Architecture</span></span>

<span data-ttu-id="ad94d-116">L’architecture est constituée des composants suivants.</span><span class="sxs-lookup"><span data-stu-id="ad94d-116">The architecture consists of the following components.</span></span>

<span data-ttu-id="ad94d-117">**Sources de données**.</span><span class="sxs-lookup"><span data-stu-id="ad94d-117">**Data sources**.</span></span> <span data-ttu-id="ad94d-118">Dans cette architecture, deux sources de données génèrent des flux de données en temps réel.</span><span class="sxs-lookup"><span data-stu-id="ad94d-118">In this architecture, there are two data sources that generate data streams in real time.</span></span> <span data-ttu-id="ad94d-119">Le premier flux de données contient des informations sur les courses, tandis que le second contient des informations sur les tarifs.</span><span class="sxs-lookup"><span data-stu-id="ad94d-119">The first stream contains ride information, and the second contains fare information.</span></span> <span data-ttu-id="ad94d-120">L’architecture de référence comprend un générateur de données simulées qui lit le contenu d’un ensemble de fichiers statiques et envoie (push) les données vers Event Hubs.</span><span class="sxs-lookup"><span data-stu-id="ad94d-120">The reference architecture includes a simulated data generator that reads from a set of static files and pushes the data to Event Hubs.</span></span> <span data-ttu-id="ad94d-121">Les sources de données dans une application réelle seraient les appareils installés dans les taxis.</span><span class="sxs-lookup"><span data-stu-id="ad94d-121">The data sources in a real application would be devices installed in the taxi cabs.</span></span>

<span data-ttu-id="ad94d-122">**Azure Event Hubs**.</span><span class="sxs-lookup"><span data-stu-id="ad94d-122">**Azure Event Hubs**.</span></span> <span data-ttu-id="ad94d-123">[Event Hubs](/azure/event-hubs/) est un service d’ingestion d’événements.</span><span class="sxs-lookup"><span data-stu-id="ad94d-123">[Event Hubs](/azure/event-hubs/) is an event ingestion service.</span></span> <span data-ttu-id="ad94d-124">Cette architecture utilise deux instances d’Event Hub, à savoir une par source de données.</span><span class="sxs-lookup"><span data-stu-id="ad94d-124">This architecture uses two event hub instances, one for each data source.</span></span> <span data-ttu-id="ad94d-125">Chaque source de données envoie un flux de données à l’Event Hub associé.</span><span class="sxs-lookup"><span data-stu-id="ad94d-125">Each data source sends a stream of data to the associated event hub.</span></span>

<span data-ttu-id="ad94d-126">**Azure Databricks**.</span><span class="sxs-lookup"><span data-stu-id="ad94d-126">**Azure Databricks**.</span></span> <span data-ttu-id="ad94d-127">[Databricks](/azure/azure-databricks/) est une plateforme d’analytique basée sur Apache Spark et optimisée pour la plateforme de services cloud Microsoft Azure.</span><span class="sxs-lookup"><span data-stu-id="ad94d-127">[Databricks](/azure/azure-databricks/) is an Apache Spark-based analytics platform optimized for the Microsoft Azure cloud services platform.</span></span> <span data-ttu-id="ad94d-128">Databricks est utilisé pour mettre en corrélation les données de courses et de tarifs des taxis, mais aussi pour enrichir les données mises en corrélation avec les données de quartiers stockées dans le système de fichiers Databricks.</span><span class="sxs-lookup"><span data-stu-id="ad94d-128">Databricks is used to correlate of the taxi ride and fare data, and also to enrich the correlated data with neighborhood data stored in the Databricks file system.</span></span>

<span data-ttu-id="ad94d-129">**Cosmos DB**.</span><span class="sxs-lookup"><span data-stu-id="ad94d-129">**Cosmos DB**.</span></span> <span data-ttu-id="ad94d-130">La sortie d’un travail Azure Databricks est une série d’enregistrements écrits dans [Cosmos DB](/azure/cosmos-db/) à l’aide de l’API Cassandra.</span><span class="sxs-lookup"><span data-stu-id="ad94d-130">The output from Azure Databricks job is a series of records, which are written to [Cosmos DB](/azure/cosmos-db/) using the Cassandra API.</span></span> <span data-ttu-id="ad94d-131">Si l’API Cassandra est utilisée, c’est parce qu’elle prend en charge la modélisation de données de séries chronologiques.</span><span class="sxs-lookup"><span data-stu-id="ad94d-131">The Cassandra API is used because it supports time series data modeling.</span></span>

<span data-ttu-id="ad94d-132">**Azure Log Analytics**.</span><span class="sxs-lookup"><span data-stu-id="ad94d-132">**Azure Log Analytics**.</span></span> <span data-ttu-id="ad94d-133">Les données de journal d’application collectées par [Azure Monitor](/azure/monitoring-and-diagnostics/) sont stockées dans un [espace de travail Log Analytics](/azure/log-analytics).</span><span class="sxs-lookup"><span data-stu-id="ad94d-133">Application log data collected by [Azure Monitor](/azure/monitoring-and-diagnostics/) is stored in a [Log Analytics workspace](/azure/log-analytics).</span></span> <span data-ttu-id="ad94d-134">Il est possible d’utiliser des requêtes Log Analytics pour analyser et visualiser les métriques et inspecter les messages de journal pour identifier les problèmes au sein de l’application.</span><span class="sxs-lookup"><span data-stu-id="ad94d-134">Log Analytics queries can be used to analyze and visualize metrics and inspect log messages to identify issues within the application.</span></span>

## <a name="data-ingestion"></a><span data-ttu-id="ad94d-135">Ingestion de données</span><span class="sxs-lookup"><span data-stu-id="ad94d-135">Data ingestion</span></span>

<!-- markdownlint-disable MD033 -->

<span data-ttu-id="ad94d-136">Pour simuler une source de données, cette architecture de référence utilise les [données des taxis de la ville de New York](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) <sup>[[1]](#note1)</sup>.</span><span class="sxs-lookup"><span data-stu-id="ad94d-136">To simulate a data source, this reference architecture uses the [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) dataset<sup>[[1]](#note1)</sup>.</span></span> <span data-ttu-id="ad94d-137">Ce jeu de données contient des données sur les courses de taxis à New York sur une période de quatre ans (2010 &ndash; 2013).</span><span class="sxs-lookup"><span data-stu-id="ad94d-137">This dataset contains data about taxi trips in New York City over a four-year period (2010 &ndash; 2013).</span></span> <span data-ttu-id="ad94d-138">Il contient deux types d’informations : Données de course et données de tarif.</span><span class="sxs-lookup"><span data-stu-id="ad94d-138">It contains two types of record: Ride data and fare data.</span></span> <span data-ttu-id="ad94d-139">Les premières incluent la durée du trajet, la distance et les lieux de prise en charge et de dépose.</span><span class="sxs-lookup"><span data-stu-id="ad94d-139">Ride data includes trip duration, trip distance, and pickup and dropoff location.</span></span> <span data-ttu-id="ad94d-140">Les secondes incluent le montant des tarifs des courses, des taxes et des pourboires.</span><span class="sxs-lookup"><span data-stu-id="ad94d-140">Fare data includes fare, tax, and tip amounts.</span></span> <span data-ttu-id="ad94d-141">Les champs communs aux deux types d’enregistrement sont le numéro de médaillon (« taxi jaune »), le permis spécial et l’ID fournisseur.</span><span class="sxs-lookup"><span data-stu-id="ad94d-141">Common fields in both record types include medallion number, hack license, and vendor ID.</span></span> <span data-ttu-id="ad94d-142">Ensemble ces trois champs identifient un taxi ainsi qu’un chauffeur.</span><span class="sxs-lookup"><span data-stu-id="ad94d-142">Together these three fields uniquely identify a taxi plus a driver.</span></span> <span data-ttu-id="ad94d-143">Les données sont stockées au format CSV.</span><span class="sxs-lookup"><span data-stu-id="ad94d-143">The data is stored in CSV format.</span></span>

> <span data-ttu-id="ad94d-144">[1] <span id="note1">Donovan, Brian; Work, Dan (2016) : Données de trajet des taxis de New York (2010-2013).</span><span class="sxs-lookup"><span data-stu-id="ad94d-144">[1] <span id="note1">Donovan, Brian; Work, Dan (2016): New York City Taxi Trip Data (2010-2013).</span></span> <span data-ttu-id="ad94d-145">Université de l’Illinois, Urbana-Champaign.</span><span class="sxs-lookup"><span data-stu-id="ad94d-145">University of Illinois at Urbana-Champaign.</span></span> <https://doi.org/10.13012/J8PN93H8>

<!-- markdownlint-enable MD033 -->

<span data-ttu-id="ad94d-146">Le générateur de données est une application .NET Core qui lit les enregistrements et les envoie à Azure Event Hubs.</span><span class="sxs-lookup"><span data-stu-id="ad94d-146">The data generator is a .NET Core application that reads the records and sends them to Azure Event Hubs.</span></span> <span data-ttu-id="ad94d-147">Le générateur envoie les données des courses au format JSON et les données relatives aux tarifs au format CSV.</span><span class="sxs-lookup"><span data-stu-id="ad94d-147">The generator sends ride data in JSON format and fare data in CSV format.</span></span>

<span data-ttu-id="ad94d-148">Le service Event Hubs utilise des [partitions](/azure/event-hubs/event-hubs-features#partitions) pour segmenter les données.</span><span class="sxs-lookup"><span data-stu-id="ad94d-148">Event Hubs uses [partitions](/azure/event-hubs/event-hubs-features#partitions) to segment the data.</span></span> <span data-ttu-id="ad94d-149">Ce système de partition permet à un consommateur de lire chaque partition en parallèle.</span><span class="sxs-lookup"><span data-stu-id="ad94d-149">Partitions allow a consumer to read each partition in parallel.</span></span> <span data-ttu-id="ad94d-150">Lorsque vous envoyez des données à Event Hubs, vous pouvez spécifier explicitement la clé de partition.</span><span class="sxs-lookup"><span data-stu-id="ad94d-150">When you send data to Event Hubs, you can specify the partition key explicitly.</span></span> <span data-ttu-id="ad94d-151">Sinon, les enregistrements sont affectés aux partitions de manière alternée.</span><span class="sxs-lookup"><span data-stu-id="ad94d-151">Otherwise, records are assigned to partitions in round-robin fashion.</span></span>

<span data-ttu-id="ad94d-152">Dans ce scénario, les données de courses et de tarifs doivent se retrouver avec le même ID de partition pour un taxi donné.</span><span class="sxs-lookup"><span data-stu-id="ad94d-152">In this scenario, ride data and fare data should end up with the same partition ID for a given taxi cab.</span></span> <span data-ttu-id="ad94d-153">Cela permet à Databricks d’appliquer un degré de parallélisme quand il met en corrélation les deux flux.</span><span class="sxs-lookup"><span data-stu-id="ad94d-153">This enables Databricks to apply a degree of parallelism when it correlates the two streams.</span></span> <span data-ttu-id="ad94d-154">Un enregistrement dans la partition *n* des données des courses correspond à un enregistrement de la partition *n* des données relatives aux tarifs.</span><span class="sxs-lookup"><span data-stu-id="ad94d-154">A record in partition *n* of the ride data will match a record in partition *n* of the fare data.</span></span>

![Diagramme du traitement de flux de données avec Azure Databricks et Event Hubs](./images/stream-processing-databricks-eh.png)

<span data-ttu-id="ad94d-156">Dans le générateur de données, le modèle de données commun pour les deux types d’enregistrement comprend une propriété `PartitionKey`, qui est la concaténation de `Medallion`, `HackLicense` et `VendorId`.</span><span class="sxs-lookup"><span data-stu-id="ad94d-156">In the data generator, the common data model for both record types has a `PartitionKey` property that is the concatenation of `Medallion`, `HackLicense`, and `VendorId`.</span></span>

```csharp
public abstract class TaxiData
{
    public TaxiData()
    {
    }

    [JsonProperty]
    public long Medallion { get; set; }

    [JsonProperty]
    public long HackLicense { get; set; }

    [JsonProperty]
    public string VendorId { get; set; }

    [JsonProperty]
    public DateTimeOffset PickupTime { get; set; }

    [JsonIgnore]
    public string PartitionKey
    {
        get => $"{Medallion}_{HackLicense}_{VendorId}";
    }
```

<span data-ttu-id="ad94d-157">Cette propriété est utilisée pour fournir une clé de partition explicite lors de l’envoi des données vers Event Hubs :</span><span class="sxs-lookup"><span data-stu-id="ad94d-157">This property is used to provide an explicit partition key when sending to Event Hubs:</span></span>

```csharp
using (var client = pool.GetObject())
{
    return client.Value.SendAsync(new EventData(Encoding.UTF8.GetBytes(
        t.GetData(dataFormat))), t.PartitionKey);
}
```

### <a name="event-hubs"></a><span data-ttu-id="ad94d-158">Event Hubs</span><span class="sxs-lookup"><span data-stu-id="ad94d-158">Event Hubs</span></span>

<span data-ttu-id="ad94d-159">La capacité de débit du service Event Hubs est mesurée par les [unités de débit](/azure/event-hubs/event-hubs-features#throughput-units).</span><span class="sxs-lookup"><span data-stu-id="ad94d-159">The throughput capacity of Event Hubs is measured in [throughput units](/azure/event-hubs/event-hubs-features#throughput-units).</span></span> <span data-ttu-id="ad94d-160">Vous pouvez mettre automatiquement à l’échelle un Event Hub en activant [l’augmentation automatique](/azure/event-hubs/event-hubs-auto-inflate), qui ajuste automatiquement les unités de débit en fonction du trafic, jusqu’à la limite configurée.</span><span class="sxs-lookup"><span data-stu-id="ad94d-160">You can autoscale an event hub by enabling [auto-inflate](/azure/event-hubs/event-hubs-auto-inflate), which automatically scales the throughput units based on traffic, up to a configured maximum.</span></span>

## <a name="stream-processing"></a><span data-ttu-id="ad94d-161">Traitement des flux de données</span><span class="sxs-lookup"><span data-stu-id="ad94d-161">Stream processing</span></span>

<span data-ttu-id="ad94d-162">Dans Azure Databricks, le traitement des données est assuré par un travail.</span><span class="sxs-lookup"><span data-stu-id="ad94d-162">In Azure Databricks, data processing is performed by a job.</span></span> <span data-ttu-id="ad94d-163">Le travail est attribué à un cluster, qui en assure l’exécution.</span><span class="sxs-lookup"><span data-stu-id="ad94d-163">The job is assigned to and runs on a cluster.</span></span> <span data-ttu-id="ad94d-164">Le travail peut être du code personnalisé écrit en Java ou un [notebook](https://docs.databricks.com/user-guide/notebooks/index.html) Spark.</span><span class="sxs-lookup"><span data-stu-id="ad94d-164">The job can either be custom code written in Java, or a Spark [notebook](https://docs.databricks.com/user-guide/notebooks/index.html).</span></span>

<span data-ttu-id="ad94d-165">Dans cette architecture de référence, le travail est une archive Java avec des classes écrites en Java et Scala.</span><span class="sxs-lookup"><span data-stu-id="ad94d-165">In this reference architecture, the job is a Java archive with classes written in both Java and Scala.</span></span> <span data-ttu-id="ad94d-166">Au moment de spécifier l’archive Java pour un travail Databricks, la classe doit être spécifiée en vue d’une exécution par le cluster Databricks.</span><span class="sxs-lookup"><span data-stu-id="ad94d-166">When specifying the Java archive for a Databricks job, the class is specified for execution by the Databricks cluster.</span></span> <span data-ttu-id="ad94d-167">Ici, la méthode **main** de la classe **com.microsoft.pnp.TaxiCabReader** contient la logique de traitement des données.</span><span class="sxs-lookup"><span data-stu-id="ad94d-167">Here, the **main** method of the **com.microsoft.pnp.TaxiCabReader** class contains the data processing logic.</span></span>

### <a name="reading-the-stream-from-the-two-event-hub-instances"></a><span data-ttu-id="ad94d-168">Lecture du flux à partir des deux instances de hub d’événements</span><span class="sxs-lookup"><span data-stu-id="ad94d-168">Reading the stream from the two event hub instances</span></span>

<span data-ttu-id="ad94d-169">La logique de traitement des données utilise [Spark Structured Streaming](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html) pour lire dans les deux instances de hub d’événements Azure :</span><span class="sxs-lookup"><span data-stu-id="ad94d-169">The data processing logic uses [Spark structured streaming](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html) to read from the two Azure event hub instances:</span></span>

```scala
val rideEventHubOptions = EventHubsConf(rideEventHubConnectionString)
      .setConsumerGroup(conf.taxiRideConsumerGroup())
      .setStartingPosition(EventPosition.fromStartOfStream)
    val rideEvents = spark.readStream
      .format("eventhubs")
      .options(rideEventHubOptions.toMap)
      .load

    val fareEventHubOptions = EventHubsConf(fareEventHubConnectionString)
      .setConsumerGroup(conf.taxiFareConsumerGroup())
      .setStartingPosition(EventPosition.fromStartOfStream)
    val fareEvents = spark.readStream
      .format("eventhubs")
      .options(fareEventHubOptions.toMap)
      .load
```

### <a name="enriching-the-data-with-the-neighborhood-information"></a><span data-ttu-id="ad94d-170">Enrichissement des données avec les informations de quartiers</span><span class="sxs-lookup"><span data-stu-id="ad94d-170">Enriching the data with the neighborhood information</span></span>

<span data-ttu-id="ad94d-171">Les données de courses comprennent les coordonnées de latitude et de longitude des lieux de prise en charge et de dépôt des clients.</span><span class="sxs-lookup"><span data-stu-id="ad94d-171">The ride data includes the latitude and longitude coordinates of the pick up and drop off locations.</span></span> <span data-ttu-id="ad94d-172">Même si ces coordonnées sont utiles, elles ne sont pas facilement exploitables pour l’analyse.</span><span class="sxs-lookup"><span data-stu-id="ad94d-172">While these coordinates are useful, they are not easily consumed for analysis.</span></span> <span data-ttu-id="ad94d-173">Par conséquent, ces données sont enrichies avec les données de quartiers lues dans un [fichier de forme](https://en.wikipedia.org/wiki/Shapefile).</span><span class="sxs-lookup"><span data-stu-id="ad94d-173">Therefore, this data is enriched with neighborhood data that is read from a [shapefile](https://en.wikipedia.org/wiki/Shapefile).</span></span>

<span data-ttu-id="ad94d-174">De format binaire, le fichier de forme ne s’analyse pas facilement, mais la bibliothèque [GeoTools](http://geotools.org/) propose des outils pour les données géospatiales qui utilisent le format de fichier de forme.</span><span class="sxs-lookup"><span data-stu-id="ad94d-174">The shapefile format is binary and not easily parsed, but the [GeoTools](http://geotools.org/) library provides tools for geospatial data that use the shapefile format.</span></span> <span data-ttu-id="ad94d-175">Cette bibliothèque est utilisée dans la classe **com.microsoft.pnp.GeoFinder** pour déterminer le nom du quartier en fonction des coordonnées de prise en charge et de dépôt.</span><span class="sxs-lookup"><span data-stu-id="ad94d-175">This library is used in the **com.microsoft.pnp.GeoFinder** class to determine the neighborhood name based on the pick up and drop off coordinates.</span></span>

```scala
val neighborhoodFinder = (lon: Double, lat: Double) => {
      NeighborhoodFinder.getNeighborhood(lon, lat).get()
    }
```

### <a name="joining-the-ride-and-fare-data"></a><span data-ttu-id="ad94d-176">Jonction des données de courses et de tarifs</span><span class="sxs-lookup"><span data-stu-id="ad94d-176">Joining the ride and fare data</span></span>

<span data-ttu-id="ad94d-177">Dans un premier temps, les données de courses et de tarifs sont transformées :</span><span class="sxs-lookup"><span data-stu-id="ad94d-177">First the ride and fare data is transformed:</span></span>

```scala
    val rides = transformedRides
      .filter(r => {
        if (r.isNullAt(r.fieldIndex("errorMessage"))) {
          true
        }
        else {
          malformedRides.add(1)
          false
        }
      })
      .select(
        $"ride.*",
        to_neighborhood($"ride.pickupLon", $"ride.pickupLat")
          .as("pickupNeighborhood"),
        to_neighborhood($"ride.dropoffLon", $"ride.dropoffLat")
          .as("dropoffNeighborhood")
      )
      .withWatermark("pickupTime", conf.taxiRideWatermarkInterval())

    val fares = transformedFares
      .filter(r => {
        if (r.isNullAt(r.fieldIndex("errorMessage"))) {
          true
        }
        else {
          malformedFares.add(1)
          false
        }
      })
      .select(
        $"fare.*",
        $"pickupTime"
      )
      .withWatermark("pickupTime", conf.taxiFareWatermarkInterval())
```

<span data-ttu-id="ad94d-178">Les données de courses sont ensuite jointes aux données de tarifs :</span><span class="sxs-lookup"><span data-stu-id="ad94d-178">And then the ride data is joined with the fare data:</span></span>

```scala
val mergedTaxiTrip = rides.join(fares, Seq("medallion", "hackLicense", "vendorId", "pickupTime"))
```

### <a name="processing-the-data-and-inserting-into-cosmos-db"></a><span data-ttu-id="ad94d-179">Traitement des données et insertion dans Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="ad94d-179">Processing the data and inserting into Cosmos DB</span></span>

<span data-ttu-id="ad94d-180">Le montant du tarif moyen pour chaque quartier est calculé pour un intervalle de temps donné :</span><span class="sxs-lookup"><span data-stu-id="ad94d-180">The average fare amount for each neighborhood is calculated for a given time interval:</span></span>

```scala
val maxAvgFarePerNeighborhood = mergedTaxiTrip.selectExpr("medallion", "hackLicense", "vendorId", "pickupTime", "rateCode", "storeAndForwardFlag", "dropoffTime", "passengerCount", "tripTimeInSeconds", "tripDistanceInMiles", "pickupLon", "pickupLat", "dropoffLon", "dropoffLat", "paymentType", "fareAmount", "surcharge", "mtaTax", "tipAmount", "tollsAmount", "totalAmount", "pickupNeighborhood", "dropoffNeighborhood")
      .groupBy(window($"pickupTime", conf.windowInterval()), $"pickupNeighborhood")
      .agg(
        count("*").as("rideCount"),
        sum($"fareAmount").as("totalFareAmount"),
        sum($"tipAmount").as("totalTipAmount")
      )
      .select($"window.start", $"window.end", $"pickupNeighborhood", $"rideCount", $"totalFareAmount", $"totalTipAmount")
```

<span data-ttu-id="ad94d-181">Il est ensuite inséré dans Cosmos DB :</span><span class="sxs-lookup"><span data-stu-id="ad94d-181">Which is then inserted into Cosmos DB:</span></span>

```scala
maxAvgFarePerNeighborhood
      .writeStream
      .queryName("maxAvgFarePerNeighborhood_cassandra_insert")
      .outputMode(OutputMode.Append())
      .foreach(new CassandraSinkForeach(connector))
      .start()
      .awaitTermination()
```

## <a name="security-considerations"></a><span data-ttu-id="ad94d-182">Considérations relatives à la sécurité</span><span class="sxs-lookup"><span data-stu-id="ad94d-182">Security considerations</span></span>

<span data-ttu-id="ad94d-183">L’accès à l’espace de travail Azure Database est contrôlé à l’aide de la [console administrateur](https://docs.databricks.com/administration-guide/admin-settings/index.html).</span><span class="sxs-lookup"><span data-stu-id="ad94d-183">Access to the Azure Database workspace is controlled using the [administrator console](https://docs.databricks.com/administration-guide/admin-settings/index.html).</span></span> <span data-ttu-id="ad94d-184">La console administrateur comprend une fonctionnalité qui permet d’ajouter des utilisateurs, de gérer les autorisations utilisateur et de configurer l’authentification unique.</span><span class="sxs-lookup"><span data-stu-id="ad94d-184">The administrator console includes functionality to add users, manage user permissions, and set up single sign-on.</span></span> <span data-ttu-id="ad94d-185">Le contrôle d’accès pour les espaces de travail, les clusters, les travaux et les tables peut aussi être défini via la console administrateur.</span><span class="sxs-lookup"><span data-stu-id="ad94d-185">Access control for workspaces, clusters, jobs, and tables can also be set through the administrator console.</span></span>

### <a name="managing-secrets"></a><span data-ttu-id="ad94d-186">Gestion des secrets</span><span class="sxs-lookup"><span data-stu-id="ad94d-186">Managing secrets</span></span>

<span data-ttu-id="ad94d-187">Azure Databricks comprend un [magasin des secrets](https://docs.azuredatabricks.net/user-guide/secrets/index.html) dans lequel sont stockés les secrets, notamment les chaînes de connexion, les clés d’accès, les noms d’utilisateur et les mots de passe.</span><span class="sxs-lookup"><span data-stu-id="ad94d-187">Azure Databricks includes a [secret store](https://docs.azuredatabricks.net/user-guide/secrets/index.html) that is used to store secrets, including connection strings, access keys, user names, and passwords.</span></span> <span data-ttu-id="ad94d-188">Les secrets contenus dans le magasin des secrets Azure Databricks sont partitionnés par **étendues** (scope) :</span><span class="sxs-lookup"><span data-stu-id="ad94d-188">Secrets within the Azure Databricks secret store are partitioned by **scopes**:</span></span>

```bash
databricks secrets create-scope --scope "azure-databricks-job"
```

<span data-ttu-id="ad94d-189">Les secrets sont ajoutés au niveau de l’étendue :</span><span class="sxs-lookup"><span data-stu-id="ad94d-189">Secrets are added at the scope level:</span></span>

```bash
databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
```

> [!NOTE]
> <span data-ttu-id="ad94d-190">Il est possible d’utiliser une étendue adossée à Azure Key Vault à la place de l’étendue Azure Databricks native.</span><span class="sxs-lookup"><span data-stu-id="ad94d-190">An Azure Key Vault-backed scope can be used instead of the native Azure Databricks scope.</span></span> <span data-ttu-id="ad94d-191">Pour en savoir plus, consultez la page relative aux [étendues adossées à Azure Key Vault](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes).</span><span class="sxs-lookup"><span data-stu-id="ad94d-191">To learn more, see [Azure Key Vault-backed scopes](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes).</span></span>

<span data-ttu-id="ad94d-192">Dans le code, les secrets sont accessibles via les [utilitaires de secrets](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities) Azure Databricks.</span><span class="sxs-lookup"><span data-stu-id="ad94d-192">In code, secrets are accessed via the Azure Databricks [secrets utilities](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities).</span></span>

## <a name="monitoring-considerations"></a><span data-ttu-id="ad94d-193">Surveillance - Éléments à prendre en compte</span><span class="sxs-lookup"><span data-stu-id="ad94d-193">Monitoring considerations</span></span>

<span data-ttu-id="ad94d-194">Azure Databricks est basé sur Apache Spark et tous deux utilisent [log4j](https://logging.apache.org/log4j/2.x/) comme bibliothèque standard pour la journalisation.</span><span class="sxs-lookup"><span data-stu-id="ad94d-194">Azure Databricks is based on Apache Spark, and both use [log4j](https://logging.apache.org/log4j/2.x/) as the standard library for logging.</span></span> <span data-ttu-id="ad94d-195">En plus de la journalisation par défaut fournie par Apache Spark, cette architecture de référence envoie des journaux d’activité et des métriques à [Azure Log Analytics](/azure/log-analytics/).</span><span class="sxs-lookup"><span data-stu-id="ad94d-195">In addition to the default logging provided by Apache Spark, this reference architecture sends logs and metrics to [Azure Log Analytics](/azure/log-analytics/).</span></span>

<span data-ttu-id="ad94d-196">La classe **com.microsoft.pnp.TaxiCabReader** configure le système de journalisation Apache Spark pour envoyer ses journaux d’activité à Azure Log Analytics en utilisant les valeurs contenues dans le fichier **log4j.properties**.</span><span class="sxs-lookup"><span data-stu-id="ad94d-196">The **com.microsoft.pnp.TaxiCabReader** class configures the Apache Spark logging system to send its logs to Azure Log Analytics using the values in the **log4j.properties** file.</span></span> <span data-ttu-id="ad94d-197">Si les messages du journaliseur Apache Spark sont des chaînes, Azure Log Analytics nécessite de son côté des messages de journal au format JSON.</span><span class="sxs-lookup"><span data-stu-id="ad94d-197">While the Apache Spark logger messages are strings, Azure Log Analytics requires log messages to be formatted as JSON.</span></span> <span data-ttu-id="ad94d-198">La classe **com.microsoft.pnp.log4j.LogAnalyticsAppender** transforme ces messages au format JSON :</span><span class="sxs-lookup"><span data-stu-id="ad94d-198">The **com.microsoft.pnp.log4j.LogAnalyticsAppender** class transforms these messages to JSON:</span></span>

```scala

    @Override
    protected void append(LoggingEvent loggingEvent) {
        if (this.layout == null) {
            this.setLayout(new JSONLayout());
        }

        String json = this.getLayout().format(loggingEvent);
        try {
            this.client.send(json, this.logType);
        } catch(IOException ioe) {
            LogLog.warn("Error sending LoggingEvent to Log Analytics", ioe);
        }
    }

```

<span data-ttu-id="ad94d-199">Comme la classe **com.microsoft.pnp.TaxiCabReader** traite les messages relatifs aux courses et aux tarifs, il est possible que l’un des deux soit mal formé et donc non valide.</span><span class="sxs-lookup"><span data-stu-id="ad94d-199">As the **com.microsoft.pnp.TaxiCabReader** class processes ride and fare messages, it's possible that either one may be malformed and therefore not valid.</span></span> <span data-ttu-id="ad94d-200">Dans un environnement de production, il est important d’analyser ces messages mal formés pour identifier un problème au niveau des sources de données et le résoudre rapidement pour éviter toute perte de données.</span><span class="sxs-lookup"><span data-stu-id="ad94d-200">In a production environment, it's important to analyze these malformed messages to identify a problem with the data sources so it can be fixed quickly to prevent data loss.</span></span> <span data-ttu-id="ad94d-201">La classe **com.microsoft.pnp.TaxiCabReader** inscrit un accumulateur Apache Spark qui assure le suivi du nombre d’enregistrements relatifs aux courses et aux tarifs mal formés :</span><span class="sxs-lookup"><span data-stu-id="ad94d-201">The **com.microsoft.pnp.TaxiCabReader** class registers an Apache Spark Accumulator that keeps track of the number of malformed fare and ride records:</span></span>

```scala
    @transient val appMetrics = new AppMetrics(spark.sparkContext)
    appMetrics.registerGauge("metrics.malformedrides", AppAccumulators.getRideInstance(spark.sparkContext))
    appMetrics.registerGauge("metrics.malformedfares", AppAccumulators.getFareInstance(spark.sparkContext))
    SparkEnv.get.metricsSystem.registerSource(appMetrics)
```

<span data-ttu-id="ad94d-202">Apache Spark utilise la bibliothèque Dropwizard pour envoyer des métriques, et certains champs de métriques Dropwizard natifs sont incompatibles avec Azure Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="ad94d-202">Apache Spark uses the Dropwizard library to send metrics, and some of the native Dropwizard metrics fields are incompatible with Azure Log Analytics.</span></span> <span data-ttu-id="ad94d-203">Par conséquent, cette architecture de référence comprend un récepteur et un rapporteur Dropwizard personnalisés.</span><span class="sxs-lookup"><span data-stu-id="ad94d-203">Therefore, this reference architecture includes a custom Dropwizard sink and reporter.</span></span> <span data-ttu-id="ad94d-204">Elle met en forme les métriques dans le format attendu par Azure Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="ad94d-204">It formats the metrics in the format expected by Azure Log Analytics.</span></span> <span data-ttu-id="ad94d-205">Quand Apache Spark transmet des métriques, les métriques personnalisées pour les données de course et de tarif malformées sont aussi envoyées.</span><span class="sxs-lookup"><span data-stu-id="ad94d-205">When Apache Spark reports metrics, the custom metrics for the malformed ride and fare data are also sent.</span></span>

<span data-ttu-id="ad94d-206">La dernière métrique à être journalisée dans l’espace de travail Azure Log Analytics est la progression cumulative du travail Spark Structured Streaming.</span><span class="sxs-lookup"><span data-stu-id="ad94d-206">The last metric to be logged to the Azure Log Analytics workspace is the cumulative progress of the Spark Structured Streaming job progress.</span></span> <span data-ttu-id="ad94d-207">Cette opération s’effectue via un écouteur StreamingQuery personnalisé implémenté dans la classe **com.microsoft.pnp.StreamingMetricsListener**.</span><span class="sxs-lookup"><span data-stu-id="ad94d-207">This is done using a custom StreamingQuery listener implemented in the **com.microsoft.pnp.StreamingMetricsListener** class.</span></span> <span data-ttu-id="ad94d-208">Cette classe est inscrite dans la session Apache Spark au moment où le travail s’exécute :</span><span class="sxs-lookup"><span data-stu-id="ad94d-208">This class is registered to the Apache Spark Session when the job runs:</span></span>

```scala
spark.streams.addListener(new StreamingMetricsListener())
```

<span data-ttu-id="ad94d-209">Les méthodes contenues dans StreamingMetricsListener sont appelées par le runtime Apache Spark chaque fois qu’un événement Structured Streaming se produit ; les messages de journal d’activité et les métriques sont alors envoyés à l’espace de travail Azure Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="ad94d-209">The methods in the StreamingMetricsListener are called by the Apache Spark runtime whenever a structured steaming event occurs, sending log messages and metrics to the Azure Log Analytics workspace.</span></span> <span data-ttu-id="ad94d-210">Vous pouvez utiliser les requêtes suivantes dans votre espace de travail pour superviser l’application :</span><span class="sxs-lookup"><span data-stu-id="ad94d-210">You can use the following queries in your workspace to monitor the application:</span></span>

### <a name="latency-and-throughput-for-streaming-queries"></a><span data-ttu-id="ad94d-211">Latence et débit pour les requêtes de streaming</span><span class="sxs-lookup"><span data-stu-id="ad94d-211">Latency and throughput for streaming queries</span></span>

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| project  mdc_inputRowsPerSecond_d, mdc_durationms_triggerExecution_d
| render timechart
```

### <a name="exceptions-logged-during-stream-query-execution"></a><span data-ttu-id="ad94d-212">Exceptions journalisées pendant l’exécution de requête de flux</span><span class="sxs-lookup"><span data-stu-id="ad94d-212">Exceptions logged during stream query execution</span></span>

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| where Level contains "Error"
```

### <a name="accumulation-of-malformed-fare-and-ride-data"></a><span data-ttu-id="ad94d-213">Accumulation de données de course et de tarif mal formées</span><span class="sxs-lookup"><span data-stu-id="ad94d-213">Accumulation of malformed fare and ride data</span></span>

```shell
SparkMetric_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart
| where name_s contains "metrics.malformedrides"

SparkMetric_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart
| where name_s contains "metrics.malformedfares"
```

### <a name="job-execution-to-trace-resiliency"></a><span data-ttu-id="ad94d-214">Exécution du travail pour tracer la résilience</span><span class="sxs-lookup"><span data-stu-id="ad94d-214">Job execution to trace resiliency</span></span>

```shell
SparkMetric_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart
| where name_s contains "driver.DAGScheduler.job.allJobs"
```

<span data-ttu-id="ad94d-215">Pour plus d’informations, consultez [surveillance Azure Databricks](../../databricks-monitoring/index.md).</span><span class="sxs-lookup"><span data-stu-id="ad94d-215">For more information, see [Monitoring Azure Databricks](../../databricks-monitoring/index.md).</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="ad94d-216">Déployer la solution</span><span class="sxs-lookup"><span data-stu-id="ad94d-216">Deploy the solution</span></span>

<span data-ttu-id="ad94d-217">Pour déployer et exécuter l’implémentation de référence, suivez les étapes du [fichier Readme de GitHub](https://github.com/mspnp/azure-databricks-streaming-analytics).</span><span class="sxs-lookup"><span data-stu-id="ad94d-217">To the deploy and run the reference implementation, follow the steps in the [GitHub readme](https://github.com/mspnp/azure-databricks-streaming-analytics).</span></span>
