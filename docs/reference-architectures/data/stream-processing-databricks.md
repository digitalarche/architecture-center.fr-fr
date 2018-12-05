---
title: Traitement de flux avec Azure Databricks
description: Créer un pipeline de traitement de flux de bout en bout dans Azure à l’aide d’Azure Databricks
author: petertaylor9999
ms.date: 11/01/2018
ms.openlocfilehash: a7e9df57572c9b3a3b0e4f418f148449aa40b04c
ms.sourcegitcommit: 19a517a2fb70768b3edb9a7c3c37197baa61d9b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2018
ms.locfileid: "52295736"
---
# <a name="stream-processing-with-azure-databricks"></a><span data-ttu-id="5db6c-103">Traitement de flux avec Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="5db6c-103">Stream processing with Azure Databricks</span></span>

<span data-ttu-id="5db6c-104">Cette architecture de référence présente un pipeline de [traitement de flux](/azure/architecture/data-guide/big-data/real-time-processing) de bout en bout.</span><span class="sxs-lookup"><span data-stu-id="5db6c-104">This reference architecture shows an end-to-end [stream processing](/azure/architecture/data-guide/big-data/real-time-processing) pipeline.</span></span> <span data-ttu-id="5db6c-105">Ce type de pipeline comprend quatre étapes : ingestion, traitement, stockage, analyse et création de rapports.</span><span class="sxs-lookup"><span data-stu-id="5db6c-105">This type of pipeline has four stages: ingest, process, store, and analysis and reporting.</span></span> <span data-ttu-id="5db6c-106">Pour cette architecture de référence, le pipeline ingère des données issues de deux sources, effectue une jonction sur les enregistrements apparentés de chaque flux, enrichit le résultat et calcule une moyenne en temps réel.</span><span class="sxs-lookup"><span data-stu-id="5db6c-106">For this reference architecture, the pipeline ingests data from two sources, performs a join on related records from each stream, enriches the result, and calculates an average in real time.</span></span> <span data-ttu-id="5db6c-107">Les résultats sont stockés en vue d’une analyse plus approfondie.</span><span class="sxs-lookup"><span data-stu-id="5db6c-107">The results are stored for further analysis.</span></span> [<span data-ttu-id="5db6c-108">**Déployez cette solution**.</span><span class="sxs-lookup"><span data-stu-id="5db6c-108">**Deploy this solution**.</span></span>](#deploy-the-solution)

![](./images/stream-processing-databricks.png)

<span data-ttu-id="5db6c-109">**Scénario** : une compagnie de taxis collecte des données sur chaque trajet effectué en taxi.</span><span class="sxs-lookup"><span data-stu-id="5db6c-109">**Scenario**: A taxi company collects data about each taxi trip.</span></span> <span data-ttu-id="5db6c-110">Pour ce scénario, nous partons du principe que deux appareils distincts envoient des données.</span><span class="sxs-lookup"><span data-stu-id="5db6c-110">For this scenario, we assume there are two separate devices sending data.</span></span> <span data-ttu-id="5db6c-111">Le taxi est équipé d’un compteur qui envoie les informations suivantes sur chaque course : durée, distance et lieux de prise en charge et de dépose.</span><span class="sxs-lookup"><span data-stu-id="5db6c-111">The taxi has a meter that sends information about each ride &mdash; the duration, distance, and pickup and dropoff locations.</span></span> <span data-ttu-id="5db6c-112">Un autre appareil accepte les paiements des clients et envoie des données sur les tarifs.</span><span class="sxs-lookup"><span data-stu-id="5db6c-112">A separate device accepts payments from customers and sends data about fares.</span></span> <span data-ttu-id="5db6c-113">Afin de déterminer les tendances des usagers, la compagnie de taxis souhaite calculer le pourboire moyen par mile parcouru, en temps réel, pour chaque quartier.</span><span class="sxs-lookup"><span data-stu-id="5db6c-113">To spot ridership trends, the taxi company wants to calculate the average tip per mile driven, in real time, for each neighborhood.</span></span>

## <a name="architecture"></a><span data-ttu-id="5db6c-114">Architecture</span><span class="sxs-lookup"><span data-stu-id="5db6c-114">Architecture</span></span>

<span data-ttu-id="5db6c-115">L’architecture est constituée des composants suivants.</span><span class="sxs-lookup"><span data-stu-id="5db6c-115">The architecture consists of the following components.</span></span>

<span data-ttu-id="5db6c-116">**Sources de données**.</span><span class="sxs-lookup"><span data-stu-id="5db6c-116">**Data sources**.</span></span> <span data-ttu-id="5db6c-117">Dans cette architecture, deux sources de données génèrent des flux de données en temps réel.</span><span class="sxs-lookup"><span data-stu-id="5db6c-117">In this architecture, there are two data sources that generate data streams in real time.</span></span> <span data-ttu-id="5db6c-118">Le premier flux de données contient des informations sur les courses, tandis que le second contient des informations sur les tarifs.</span><span class="sxs-lookup"><span data-stu-id="5db6c-118">The first stream contains ride information, and the second contains fare information.</span></span> <span data-ttu-id="5db6c-119">L’architecture de référence comprend un générateur de données simulées qui lit le contenu d’un ensemble de fichiers statiques et envoie (push) les données vers Event Hubs.</span><span class="sxs-lookup"><span data-stu-id="5db6c-119">The reference architecture includes a simulated data generator that reads from a set of static files and pushes the data to Event Hubs.</span></span> <span data-ttu-id="5db6c-120">Les sources de données dans une application réelle seraient les appareils installés dans les taxis.</span><span class="sxs-lookup"><span data-stu-id="5db6c-120">The data sources in a real application would be devices installed in the taxi cabs.</span></span>

<span data-ttu-id="5db6c-121">**Azure Event Hubs**.</span><span class="sxs-lookup"><span data-stu-id="5db6c-121">**Azure Event Hubs**.</span></span> <span data-ttu-id="5db6c-122">[Event Hubs](/azure/event-hubs/) est un service d’ingestion d’événements.</span><span class="sxs-lookup"><span data-stu-id="5db6c-122">[Event Hubs](/azure/event-hubs/) is an event ingestion service.</span></span> <span data-ttu-id="5db6c-123">Cette architecture utilise deux instances de hub d’événements, à savoir une par source de données.</span><span class="sxs-lookup"><span data-stu-id="5db6c-123">This architecture uses two event hub instances, one for each data source.</span></span> <span data-ttu-id="5db6c-124">Chaque source de données envoie un flux de données au hub d’événements associé.</span><span class="sxs-lookup"><span data-stu-id="5db6c-124">Each data source sends a stream of data to the associated event hub.</span></span>

<span data-ttu-id="5db6c-125">**Azure Databricks**.</span><span class="sxs-lookup"><span data-stu-id="5db6c-125">**Azure Databricks**.</span></span> <span data-ttu-id="5db6c-126">[Databricks](/azure/azure-databricks/) est une plateforme d’analytique basée sur Apache Spark et optimisée pour la plateforme de services cloud Microsoft Azure.</span><span class="sxs-lookup"><span data-stu-id="5db6c-126">[Databricks](/azure/azure-databricks/) is an Apache Spark-based analytics platform optimized for the Microsoft Azure cloud services platform.</span></span> <span data-ttu-id="5db6c-127">Databricks est utilisé pour mettre en corrélation les données de courses et de tarifs des taxis, mais aussi pour enrichir les données mises en corrélation avec les données de quartiers stockées dans le système de fichiers Databricks.</span><span class="sxs-lookup"><span data-stu-id="5db6c-127">Databricks is used to correlate of the taxi ride and fare data, and also to enrich the correlated data with neighborhood data stored in the Databricks file system.</span></span>

<span data-ttu-id="5db6c-128">**Cosmos DB**.</span><span class="sxs-lookup"><span data-stu-id="5db6c-128">**Cosmos DB**.</span></span> <span data-ttu-id="5db6c-129">La sortie d’un travail Azure Databricks est une série d’enregistrements écrits dans [Cosmos DB](/azure/cosmos-db/) à l’aide de l’API Cassandra.</span><span class="sxs-lookup"><span data-stu-id="5db6c-129">The output from Azure Databricks job is a series of records, which are written to [Cosmos DB](/azure/cosmos-db/) using the Cassandra API.</span></span> <span data-ttu-id="5db6c-130">Si l’API Cassandra est utilisée, c’est parce qu’elle prend en charge la modélisation de données de séries chronologiques.</span><span class="sxs-lookup"><span data-stu-id="5db6c-130">The Cassandra API is used because it supports time series data modeling.</span></span>

<span data-ttu-id="5db6c-131">**Azure Log Analytics**.</span><span class="sxs-lookup"><span data-stu-id="5db6c-131">**Azure Log Analytics**.</span></span> <span data-ttu-id="5db6c-132">Les données de journal d’application collectées par [Azure Monitor](/azure/monitoring-and-diagnostics/) sont stockées dans un [espace de travail Log Analytics](/azure/log-analytics).</span><span class="sxs-lookup"><span data-stu-id="5db6c-132">Application log data collected by [Azure Monitor](/azure/monitoring-and-diagnostics/) is stored in a [Log Analytics workspace](/azure/log-analytics).</span></span> <span data-ttu-id="5db6c-133">Il est possible d’utiliser des requêtes Log Analytics pour analyser et visualiser les métriques et inspecter les messages de journal pour identifier les problèmes au sein de l’application.</span><span class="sxs-lookup"><span data-stu-id="5db6c-133">Log Analytics queries can be used to analyze and visualize metrics and inspect log messages to identify issues within the application.</span></span>

## <a name="data-ingestion"></a><span data-ttu-id="5db6c-134">Ingestion de données</span><span class="sxs-lookup"><span data-stu-id="5db6c-134">Data ingestion</span></span>

<span data-ttu-id="5db6c-135">Pour simuler une source de données, cette architecture de référence utilise les [données des taxis de la ville de New York](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) <sup>[[1]](#note1)</sup>.</span><span class="sxs-lookup"><span data-stu-id="5db6c-135">To simulate a data source, this reference architecture uses the [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) dataset<sup>[[1]](#note1)</sup>.</span></span> <span data-ttu-id="5db6c-136">Ce jeu de données contient des données sur les courses de taxis à New York sur une période de quatre ans (2010 &ndash; 2013).</span><span class="sxs-lookup"><span data-stu-id="5db6c-136">This dataset contains data about taxi trips in New York City over a four-year period (2010 &ndash; 2013).</span></span> <span data-ttu-id="5db6c-137">Il contient deux types d’enregistrement : les données sur les courses et les données sur les tarifs.</span><span class="sxs-lookup"><span data-stu-id="5db6c-137">It contains two types of record: Ride data and fare data.</span></span> <span data-ttu-id="5db6c-138">Les premières incluent la durée du trajet, la distance et les lieux de prise en charge et de dépose.</span><span class="sxs-lookup"><span data-stu-id="5db6c-138">Ride data includes trip duration, trip distance, and pickup and dropoff location.</span></span> <span data-ttu-id="5db6c-139">Les secondes incluent le montant des tarifs des courses, des taxes et des pourboires.</span><span class="sxs-lookup"><span data-stu-id="5db6c-139">Fare data includes fare, tax, and tip amounts.</span></span> <span data-ttu-id="5db6c-140">Les champs communs aux deux types d’enregistrement sont le numéro de médaillon (« taxi jaune »), le permis spécial et l’ID fournisseur.</span><span class="sxs-lookup"><span data-stu-id="5db6c-140">Common fields in both record types include medallion number, hack license, and vendor ID.</span></span> <span data-ttu-id="5db6c-141">Ensemble ces trois champs identifient un taxi ainsi qu’un chauffeur.</span><span class="sxs-lookup"><span data-stu-id="5db6c-141">Together these three fields uniquely identify a taxi plus a driver.</span></span> <span data-ttu-id="5db6c-142">Les données sont stockées au format CSV.</span><span class="sxs-lookup"><span data-stu-id="5db6c-142">The data is stored in CSV format.</span></span> 

<span data-ttu-id="5db6c-143">Le générateur de données est une application .NET Core qui lit les enregistrements et les envoie à Azure Event Hubs.</span><span class="sxs-lookup"><span data-stu-id="5db6c-143">The data generator is a .NET Core application that reads the records and sends them to Azure Event Hubs.</span></span> <span data-ttu-id="5db6c-144">Le générateur envoie les données des courses au format JSON et les données relatives aux tarifs au format CSV.</span><span class="sxs-lookup"><span data-stu-id="5db6c-144">The generator sends ride data in JSON format and fare data in CSV format.</span></span> 

<span data-ttu-id="5db6c-145">Le service Event Hubs utilise des [partitions](/azure/event-hubs/event-hubs-features#partitions) pour segmenter les données.</span><span class="sxs-lookup"><span data-stu-id="5db6c-145">Event Hubs uses [partitions](/azure/event-hubs/event-hubs-features#partitions) to segment the data.</span></span> <span data-ttu-id="5db6c-146">Ce système de partition permet à un consommateur de lire chaque partition en parallèle.</span><span class="sxs-lookup"><span data-stu-id="5db6c-146">Partitions allow a consumer to read each partition in parallel.</span></span> <span data-ttu-id="5db6c-147">Lorsque vous envoyez des données à Event Hubs, vous pouvez spécifier explicitement la clé de partition.</span><span class="sxs-lookup"><span data-stu-id="5db6c-147">When you send data to Event Hubs, you can specify the partition key explicitly.</span></span> <span data-ttu-id="5db6c-148">Sinon, les enregistrements sont affectés aux partitions de manière alternée.</span><span class="sxs-lookup"><span data-stu-id="5db6c-148">Otherwise, records are assigned to partitions in round-robin fashion.</span></span> 

<span data-ttu-id="5db6c-149">Dans ce scénario, les données de courses et de tarifs doivent se retrouver avec le même ID de partition pour un taxi donné.</span><span class="sxs-lookup"><span data-stu-id="5db6c-149">In this scenario, ride data and fare data should end up with the same partition ID for a given taxi cab.</span></span> <span data-ttu-id="5db6c-150">Cela permet à Databricks d’appliquer un degré de parallélisme quand il met en corrélation les deux flux.</span><span class="sxs-lookup"><span data-stu-id="5db6c-150">This enables Databricks to apply a degree of parallelism when it correlates the two streams.</span></span> <span data-ttu-id="5db6c-151">Un enregistrement dans la partition *n* des données des courses correspond à un enregistrement de la partition *n* des données relatives aux tarifs.</span><span class="sxs-lookup"><span data-stu-id="5db6c-151">A record in partition *n* of the ride data will match a record in partition *n* of the fare data.</span></span>

![](./images/stream-processing-databricks-eh.png)

<span data-ttu-id="5db6c-152">Dans le générateur de données, le modèle de données commun pour les deux types d’enregistrement comprend une propriété `PartitionKey`, qui est la concaténation de `Medallion`, `HackLicense` et `VendorId`.</span><span class="sxs-lookup"><span data-stu-id="5db6c-152">In the data generator, the common data model for both record types has a `PartitionKey` property that is the concatenation of `Medallion`, `HackLicense`, and `VendorId`.</span></span>

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

<span data-ttu-id="5db6c-153">Cette propriété est utilisée pour fournir une clé de partition explicite lors de l’envoi des données vers Event Hubs :</span><span class="sxs-lookup"><span data-stu-id="5db6c-153">This property is used to provide an explicit partition key when sending to Event Hubs:</span></span>

```csharp
using (var client = pool.GetObject())
{
    return client.Value.SendAsync(new EventData(Encoding.UTF8.GetBytes(
        t.GetData(dataFormat))), t.PartitionKey);
}
```

### <a name="event-hubs"></a><span data-ttu-id="5db6c-154">Event Hubs</span><span class="sxs-lookup"><span data-stu-id="5db6c-154">Event Hubs</span></span>

<span data-ttu-id="5db6c-155">La capacité de débit du service Event Hubs est mesurée par les [unités de débit](/azure/event-hubs/event-hubs-features#throughput-units).</span><span class="sxs-lookup"><span data-stu-id="5db6c-155">The throughput capacity of Event Hubs is measured in [throughput units](/azure/event-hubs/event-hubs-features#throughput-units).</span></span> <span data-ttu-id="5db6c-156">Vous pouvez mettre automatiquement à l’échelle un hub d’événements en activant [l’augmentation automatique](/azure/event-hubs/event-hubs-auto-inflate), qui ajuste automatiquement les unités de débit en fonction du trafic, jusqu’à la limite configurée.</span><span class="sxs-lookup"><span data-stu-id="5db6c-156">You can autoscale an event hub by enabling [auto-inflate](/azure/event-hubs/event-hubs-auto-inflate), which automatically scales the throughput units based on traffic, up to a configured maximum.</span></span> 

## <a name="stream-processing"></a><span data-ttu-id="5db6c-157">Traitement des flux de données</span><span class="sxs-lookup"><span data-stu-id="5db6c-157">Stream processing</span></span>

<span data-ttu-id="5db6c-158">Dans Azure Databricks, le traitement des données est assuré par un travail.</span><span class="sxs-lookup"><span data-stu-id="5db6c-158">In Azure Databricks, data processing is performed by a job.</span></span> <span data-ttu-id="5db6c-159">Le travail est attribué à un cluster, qui en assure l’exécution.</span><span class="sxs-lookup"><span data-stu-id="5db6c-159">The job is assigned to and runs on a cluster.</span></span> <span data-ttu-id="5db6c-160">Le travail peut être du code personnalisé écrit en Java ou un [notebook](https://docs.databricks.com/user-guide/notebooks/index.html) Spark.</span><span class="sxs-lookup"><span data-stu-id="5db6c-160">The job can either be custom code written in Java, or a Spark [notebook](https://docs.databricks.com/user-guide/notebooks/index.html).</span></span>

<span data-ttu-id="5db6c-161">Dans cette architecture de référence, le travail est une archive Java avec des classes écrites en Java et Scala.</span><span class="sxs-lookup"><span data-stu-id="5db6c-161">In this reference architecture, the job is a Java archive with classes written in both Java and Scala.</span></span> <span data-ttu-id="5db6c-162">Au moment de spécifier l’archive Java pour un travail Databricks, la classe doit être spécifiée en vue d’une exécution par le cluster Databricks.</span><span class="sxs-lookup"><span data-stu-id="5db6c-162">When specifying the Java archive for a Databricks job, the class is specified for execution by the Databricks cluster.</span></span> <span data-ttu-id="5db6c-163">Ici, la méthode **main** de la classe **com.microsoft.pnp.TaxiCabReader** contient la logique de traitement des données.</span><span class="sxs-lookup"><span data-stu-id="5db6c-163">Here, the **main** method of the **com.microsoft.pnp.TaxiCabReader** class contains the data processing logic.</span></span> 

### <a name="reading-the-stream-from-the-two-event-hub-instances"></a><span data-ttu-id="5db6c-164">Lecture du flux à partir des deux instances de hub d’événements</span><span class="sxs-lookup"><span data-stu-id="5db6c-164">Reading the stream from the two event hub instances</span></span>

<span data-ttu-id="5db6c-165">La logique de traitement des données utilise [Spark Structured Streaming](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html) pour lire dans les deux instances de hub d’événements Azure :</span><span class="sxs-lookup"><span data-stu-id="5db6c-165">The data processing logic uses [Spark structured streaming](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html) to read from the two Azure event hub instances:</span></span>

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

### <a name="enriching-the-data-with-the-neighborhood-information"></a><span data-ttu-id="5db6c-166">Enrichissement des données avec les informations de quartiers</span><span class="sxs-lookup"><span data-stu-id="5db6c-166">Enriching the data with the neighborhood information</span></span>

<span data-ttu-id="5db6c-167">Les données de courses comprennent les coordonnées de latitude et de longitude des lieux de prise en charge et de dépôt des clients.</span><span class="sxs-lookup"><span data-stu-id="5db6c-167">The ride data includes the latitude and longitude coordinates of the pick up and drop off locations.</span></span> <span data-ttu-id="5db6c-168">Même si ces coordonnées sont utiles, elles ne sont pas facilement exploitables pour l’analyse.</span><span class="sxs-lookup"><span data-stu-id="5db6c-168">While these coordinates are useful, they are not easily consumed for analysis.</span></span> <span data-ttu-id="5db6c-169">Par conséquent, ces données sont enrichies avec les données de quartiers lues dans un [fichier de forme](https://en.wikipedia.org/wiki/Shapefile).</span><span class="sxs-lookup"><span data-stu-id="5db6c-169">Therefore, this data is enriched with neighborhood data that is read from a [shapefile](https://en.wikipedia.org/wiki/Shapefile).</span></span> 

<span data-ttu-id="5db6c-170">De format binaire, le fichier de forme ne s’analyse pas facilement, mais la bibliothèque [GeoTools](http://geotools.org/) propose des outils pour les données géospatiales qui utilisent le format de fichier de forme.</span><span class="sxs-lookup"><span data-stu-id="5db6c-170">The shapefile format is binary and not easily parsed, but the [GeoTools](http://geotools.org/) library provides tools for geospatial data that use the shapefile format.</span></span> <span data-ttu-id="5db6c-171">Cette bibliothèque est utilisée dans la classe **com.microsoft.pnp.GeoFinder** pour déterminer le nom du quartier en fonction des coordonnées de prise en charge et de dépôt.</span><span class="sxs-lookup"><span data-stu-id="5db6c-171">This library is used in the **com.microsoft.pnp.GeoFinder** class to determine the neighborhood name based on the pick up and drop off coordinates.</span></span> 

```scala
val neighborhoodFinder = (lon: Double, lat: Double) => {
      NeighborhoodFinder.getNeighborhood(lon, lat).get()
    }
```

### <a name="joining-the-ride-and-fare-data"></a><span data-ttu-id="5db6c-172">Jonction des données de courses et de tarifs</span><span class="sxs-lookup"><span data-stu-id="5db6c-172">Joining the ride and fare data</span></span>

<span data-ttu-id="5db6c-173">Dans un premier temps, les données de courses et de tarifs sont transformées :</span><span class="sxs-lookup"><span data-stu-id="5db6c-173">First the ride and fare data is transformed:</span></span>

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

<span data-ttu-id="5db6c-174">Les données de courses sont ensuite jointes aux données de tarifs :</span><span class="sxs-lookup"><span data-stu-id="5db6c-174">And then the ride data is joined with the fare data:</span></span>

```scala
val mergedTaxiTrip = rides.join(fares, Seq("medallion", "hackLicense", "vendorId", "pickupTime"))
```

### <a name="processing-the-data-and-inserting-into-cosmos-db"></a><span data-ttu-id="5db6c-175">Traitement des données et insertion dans Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="5db6c-175">Processing the data and inserting into Cosmos DB</span></span>

<span data-ttu-id="5db6c-176">Le montant du tarif moyen pour chaque quartier est calculé pour un intervalle de temps donné :</span><span class="sxs-lookup"><span data-stu-id="5db6c-176">The average fare amount for each neighborhood is calculated for a given time interval:</span></span>

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

<span data-ttu-id="5db6c-177">Il est ensuite inséré dans Cosmos DB :</span><span class="sxs-lookup"><span data-stu-id="5db6c-177">Which is then inserted into Cosmos DB:</span></span>

```scala
maxAvgFarePerNeighborhood
      .writeStream
      .queryName("maxAvgFarePerNeighborhood_cassandra_insert")
      .outputMode(OutputMode.Append())
      .foreach(new CassandraSinkForeach(connector))
      .start()
      .awaitTermination()
```

## <a name="security-considerations"></a><span data-ttu-id="5db6c-178">Considérations relatives à la sécurité</span><span class="sxs-lookup"><span data-stu-id="5db6c-178">Security considerations</span></span>

<span data-ttu-id="5db6c-179">L’accès à l’espace de travail Azure Database est contrôlé à l’aide de la [console administrateur](https://docs.databricks.com/administration-guide/admin-settings/index.html).</span><span class="sxs-lookup"><span data-stu-id="5db6c-179">Access to the Azure Database workspace is controlled using the [administrator console](https://docs.databricks.com/administration-guide/admin-settings/index.html).</span></span> <span data-ttu-id="5db6c-180">La console administrateur comprend une fonctionnalité qui permet d’ajouter des utilisateurs, de gérer les autorisations utilisateur et de configurer l’authentification unique.</span><span class="sxs-lookup"><span data-stu-id="5db6c-180">The administrator console includes functionality to add users, manage user permissions, and set up single sign-on.</span></span> <span data-ttu-id="5db6c-181">Le contrôle d’accès pour les espaces de travail, les clusters, les travaux et les tables peut aussi être défini via la console administrateur.</span><span class="sxs-lookup"><span data-stu-id="5db6c-181">Access control for workspaces, clusters, jobs, and tables can also be set through the administrator console.</span></span>

### <a name="managing-secrets"></a><span data-ttu-id="5db6c-182">Gestion des secrets</span><span class="sxs-lookup"><span data-stu-id="5db6c-182">Managing secrets</span></span>

<span data-ttu-id="5db6c-183">Azure Databricks comprend un [magasin des secrets](https://docs.azuredatabricks.net/user-guide/secrets/index.html) dans lequel sont stockés les secrets, notamment les chaînes de connexion, les clés d’accès, les noms d’utilisateur et les mots de passe.</span><span class="sxs-lookup"><span data-stu-id="5db6c-183">Azure Databricks includes a [secret store](https://docs.azuredatabricks.net/user-guide/secrets/index.html) that is used to store secrets, including connection strings, access keys, user names, and passwords.</span></span> <span data-ttu-id="5db6c-184">Les secrets contenus dans le magasin des secrets Azure Databricks sont partitionnés par **étendues** (scope) :</span><span class="sxs-lookup"><span data-stu-id="5db6c-184">Secrets within the Azure Databricks secret store are partitioned by **scopes**:</span></span>

```bash
databricks secrets create-scope --scope "azure-databricks-job"
```

<span data-ttu-id="5db6c-185">Les secrets sont ajoutés au niveau de l’étendue :</span><span class="sxs-lookup"><span data-stu-id="5db6c-185">Secrets are added at the scope level:</span></span>

```bash
databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
```

> [!NOTE]
> <span data-ttu-id="5db6c-186">Il est possible d’utiliser une étendue adossée à Azure Key Vault à la place de l’étendue Azure Databricks native.</span><span class="sxs-lookup"><span data-stu-id="5db6c-186">An Azure Key Vault-backed scope can be used instead of the native Azure Databricks scope.</span></span> <span data-ttu-id="5db6c-187">Pour en savoir plus, consultez la page relative aux [étendues adossées à Azure Key Vault](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes).</span><span class="sxs-lookup"><span data-stu-id="5db6c-187">To learn more, see [Azure Key Vault-backed scopes](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes).</span></span>

<span data-ttu-id="5db6c-188">Dans le code, les secrets sont accessibles via les [utilitaires de secrets](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities) Azure Databricks.</span><span class="sxs-lookup"><span data-stu-id="5db6c-188">In code, secrets are accessed via the Azure Databricks [secrets utilities](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities).</span></span>


## <a name="monitoring-considerations"></a><span data-ttu-id="5db6c-189">Éléments à prendre en compte sur la supervision</span><span class="sxs-lookup"><span data-stu-id="5db6c-189">Monitoring considerations</span></span>

<span data-ttu-id="5db6c-190">Azure Databricks est basé sur Apache Spark et tous deux utilisent [log4j](https://logging.apache.org/log4j/2.x/) comme bibliothèque standard pour la journalisation.</span><span class="sxs-lookup"><span data-stu-id="5db6c-190">Azure Databricks is based on Apache Spark, and both use [log4j](https://logging.apache.org/log4j/2.x/) as the standard library for logging.</span></span> <span data-ttu-id="5db6c-191">En plus de la journalisation par défaut fournie par Apache Spark, cette architecture de référence envoie des journaux et des métriques à [Azure Log Analytics](/azure/log-analytics/).</span><span class="sxs-lookup"><span data-stu-id="5db6c-191">In addition to the default logging provided by Apache Spark, this reference architecture sends logs and metrics to [Azure Log Analytics](/azure/log-analytics/).</span></span>

<span data-ttu-id="5db6c-192">La classe **com.microsoft.pnp.TaxiCabReader** configure le système de journalisation Apache Spark pour envoyer ses journaux à Azure Log Analytics en utilisant les valeurs contenues dans le fichier **log4j.properties**.</span><span class="sxs-lookup"><span data-stu-id="5db6c-192">The **com.microsoft.pnp.TaxiCabReader** class configures the Apache Spark logging system to send its logs to Azure Log Analytics using the values in the **log4j.properties** file.</span></span> <span data-ttu-id="5db6c-193">Si les messages du journaliseur Apache Spark sont des chaînes, Azure Log Analytics nécessite de son côté des messages de journal au format JSON.</span><span class="sxs-lookup"><span data-stu-id="5db6c-193">While the Apache Spark logger messages are strings, Azure Log Analytics requires log messages to be formatted as JSON.</span></span> <span data-ttu-id="5db6c-194">La classe **com.microsoft.pnp.log4j.LogAnalyticsAppender** transforme ces messages au format JSON :</span><span class="sxs-lookup"><span data-stu-id="5db6c-194">The **com.microsoft.pnp.log4j.LogAnalyticsAppender** class transforms these messages to JSON:</span></span>

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

<span data-ttu-id="5db6c-195">Comme la classe **com.microsoft.pnp.TaxiCabReader** traite les messages relatifs aux courses et aux tarifs, il est possible que l’un des deux soit mal formé et donc non valide.</span><span class="sxs-lookup"><span data-stu-id="5db6c-195">As the **com.microsoft.pnp.TaxiCabReader** class processes ride and fare messages, it's possible that either one may be malformed and therefore not valid.</span></span> <span data-ttu-id="5db6c-196">Dans un environnement de production, il est important d’analyser ces messages mal formés pour identifier un problème au niveau des sources de données et le résoudre rapidement pour éviter toute perte de données.</span><span class="sxs-lookup"><span data-stu-id="5db6c-196">In a production environment, it's important to analyze these malformed messages to identify a problem with the data sources so it can be fixed quickly to prevent data loss.</span></span> <span data-ttu-id="5db6c-197">La classe **com.microsoft.pnp.TaxiCabReader** inscrit un accumulateur Apache Spark qui assure le suivi du nombre d’enregistrements relatifs aux courses et aux tarifs mal formés :</span><span class="sxs-lookup"><span data-stu-id="5db6c-197">The **com.microsoft.pnp.TaxiCabReader** class registers an Apache Spark Accumulator that keeps track of the number of malformed fare and ride records:</span></span>

```scala
    @transient val appMetrics = new AppMetrics(spark.sparkContext)
    appMetrics.registerGauge("metrics.malformedrides", AppAccumulators.getRideInstance(spark.sparkContext))
    appMetrics.registerGauge("metrics.malformedfares", AppAccumulators.getFareInstance(spark.sparkContext))
    SparkEnv.get.metricsSystem.registerSource(appMetrics)
```

<span data-ttu-id="5db6c-198">Apache Spark utilise la bibliothèque Dropwizard pour envoyer des métriques, et certains champs de métriques Dropwizard natifs sont incompatibles avec Azure Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="5db6c-198">Apache Spark uses the Dropwizard library to send metrics, and some of the native Dropwizard metrics fields are incompatible with Azure Log Analytics.</span></span> <span data-ttu-id="5db6c-199">Par conséquent, cette architecture de référence comprend un récepteur et un rapporteur Dropwizard personnalisés.</span><span class="sxs-lookup"><span data-stu-id="5db6c-199">Therefore, this reference architecture includes a custom Dropwizard sink and reporter.</span></span> <span data-ttu-id="5db6c-200">Elle met en forme les métriques dans le format attendu par Azure Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="5db6c-200">It formats the metrics in the format expected by Azure Log Analytics.</span></span> <span data-ttu-id="5db6c-201">Quand Apache Spark transmet des métriques, les métriques personnalisées pour les données de course et de tarif malformées sont aussi envoyées.</span><span class="sxs-lookup"><span data-stu-id="5db6c-201">When Apache Spark reports metrics, the custom metrics for the malformed ride and fare data are also sent.</span></span>

<span data-ttu-id="5db6c-202">La dernière métrique à être journalisée dans l’espace de travail Azure Log Analytics est la progression cumulative du travail Spark Structured Streaming.</span><span class="sxs-lookup"><span data-stu-id="5db6c-202">The last metric to be logged to the Azure Log Analytics workspace is the cumulative progress of the Spark Structured Streaming job progress.</span></span> <span data-ttu-id="5db6c-203">Cette opération s’effectue via un écouteur StreamingQuery personnalisé implémenté dans la classe **com.microsoft.pnp.StreamingMetricsListener**.</span><span class="sxs-lookup"><span data-stu-id="5db6c-203">This is done using a custom StreamingQuery listener implemented in the **com.microsoft.pnp.StreamingMetricsListener** class.</span></span> <span data-ttu-id="5db6c-204">Cette classe est inscrite dans la session Apache Spark au moment où le travail s’exécute :</span><span class="sxs-lookup"><span data-stu-id="5db6c-204">This class is registered to the Apache Spark Session when the job runs:</span></span>

```scala
spark.streams.addListener(new StreamingMetricsListener())
```

<span data-ttu-id="5db6c-205">Les méthodes contenues dans StreamingMetricsListener sont appelées par le runtime Apache Spark chaque fois qu’un événement Structured Streaming se produit ; les messages de journal et les métriques sont alors envoyés à l’espace de travail Azure Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="5db6c-205">The methods in the StreamingMetricsListener are called by the Apache Spark runtime whenever a structured steaming event occurs, sending log messages and metrics to the Azure Log Analytics workspace.</span></span> <span data-ttu-id="5db6c-206">Vous pouvez utiliser les requêtes suivantes dans votre espace de travail pour superviser l’application :</span><span class="sxs-lookup"><span data-stu-id="5db6c-206">You can use the following queries in your workspace to monitor the application:</span></span>

### <a name="latency-and-throughput-for-streaming-queries"></a><span data-ttu-id="5db6c-207">Latence et débit pour les requêtes de streaming</span><span class="sxs-lookup"><span data-stu-id="5db6c-207">Latency and throughput for streaming queries</span></span> 

```
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| project  mdc_inputRowsPerSecond_d, mdc_durationms_triggerExecution_d  
| render timechart
``` 
### <a name="exceptions-logged-during-stream-query-execution"></a><span data-ttu-id="5db6c-208">Exceptions journalisées pendant l’exécution de requête de flux</span><span class="sxs-lookup"><span data-stu-id="5db6c-208">Exceptions logged during stream query execution</span></span>

```
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| where Level contains "Error" 
```

### <a name="accumulation-of-malformed-fare-and-ride-data"></a><span data-ttu-id="5db6c-209">Accumulation de données de course et de tarif mal formées</span><span class="sxs-lookup"><span data-stu-id="5db6c-209">Accumulation of malformed fare and ride data</span></span>

```
SparkMetric_CL 
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart 
| where name_s contains "metrics.malformedrides"

SparkMetric_CL 
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart 
| where name_s contains "metrics.malformedfares" 
```

### <a name="job-execution-to-trace-resiliency"></a><span data-ttu-id="5db6c-210">Exécution du travail pour tracer la résilience</span><span class="sxs-lookup"><span data-stu-id="5db6c-210">Job execution to trace resiliency</span></span>
```
SparkMetric_CL 
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart 
| where name_s contains "driver.DAGScheduler.job.allJobs" 
```

## <a name="deploy-the-solution"></a><span data-ttu-id="5db6c-211">Déployer la solution</span><span class="sxs-lookup"><span data-stu-id="5db6c-211">Deploy the solution</span></span>

<span data-ttu-id="5db6c-212">Un déploiement de cette architecture de référence est disponible sur [GitHub](https://github.com/mspnp/reference-architectures/tree/master/data).</span><span class="sxs-lookup"><span data-stu-id="5db6c-212">A deployment for this reference architecture is available on [GitHub](https://github.com/mspnp/reference-architectures/tree/master/data).</span></span> 

### <a name="prerequisites"></a><span data-ttu-id="5db6c-213">Prérequis</span><span class="sxs-lookup"><span data-stu-id="5db6c-213">Prerequisites</span></span>

1. <span data-ttu-id="5db6c-214">Clonez, dupliquez ou téléchargez le fichier zip pour le dépôt GitHub des [architectures de référence](https://github.com/mspnp/reference-architectures).</span><span class="sxs-lookup"><span data-stu-id="5db6c-214">Clone, fork, or download the zip file for the [reference architectures](https://github.com/mspnp/reference-architectures) GitHub repository.</span></span>

2. <span data-ttu-id="5db6c-215">Installez [Docker](https://www.docker.com/) pour exécuter le générateur de données.</span><span class="sxs-lookup"><span data-stu-id="5db6c-215">Install [Docker](https://www.docker.com/) to run the data generator.</span></span>

3. <span data-ttu-id="5db6c-216">Installez [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).</span><span class="sxs-lookup"><span data-stu-id="5db6c-216">Install [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).</span></span>

4. <span data-ttu-id="5db6c-217">Installez l’interface [CLI Databricks](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html).</span><span class="sxs-lookup"><span data-stu-id="5db6c-217">Install [Databricks CLI](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html).</span></span>

5. <span data-ttu-id="5db6c-218">À partir d’une invite de commandes, d’une invite bash ou de l’invite de commandes PowerShell, connectez-vous à votre compte Azure, comme suit :</span><span class="sxs-lookup"><span data-stu-id="5db6c-218">From a command prompt, bash prompt, or PowerShell prompt, sign into your Azure account as follows:</span></span>
    ```
    az login
    ```
6. <span data-ttu-id="5db6c-219">Installez un IDE Java, avec les ressources suivantes :</span><span class="sxs-lookup"><span data-stu-id="5db6c-219">Install a Java IDE, with the following resources:</span></span>
    - <span data-ttu-id="5db6c-220">JDK 1.8</span><span class="sxs-lookup"><span data-stu-id="5db6c-220">JDK 1.8</span></span>
    - <span data-ttu-id="5db6c-221">SDK Scala 2.11</span><span class="sxs-lookup"><span data-stu-id="5db6c-221">Scala SDK 2.11</span></span>
    - <span data-ttu-id="5db6c-222">Maven 3.5.4</span><span class="sxs-lookup"><span data-stu-id="5db6c-222">Maven 3.5.4</span></span>

### <a name="download-the-new-york-city-taxi-and-neighborhood-data-files"></a><span data-ttu-id="5db6c-223">Télécharger les fichiers de données sur les taxis et les quartiers de New York</span><span class="sxs-lookup"><span data-stu-id="5db6c-223">Download the New York City taxi and neighborhood data files</span></span>

1. <span data-ttu-id="5db6c-224">Créez un répertoire nommé `DataFile` sous le répertoire `data/streaming_azuredatabricks` dans votre système de fichiers local.</span><span class="sxs-lookup"><span data-stu-id="5db6c-224">Create a directory named `DataFile` under the `data/streaming_azuredatabricks` directory in your local file system.</span></span>

2. <span data-ttu-id="5db6c-225">Ouvrez un navigateur web et accédez à https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935.</span><span class="sxs-lookup"><span data-stu-id="5db6c-225">Open a web browser and navigate to https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935.</span></span>

3. <span data-ttu-id="5db6c-226">Cliquez sur le bouton **Télécharger** de cette page pour télécharger un fichier zip de toutes les données portant sur les taxis pour cette année.</span><span class="sxs-lookup"><span data-stu-id="5db6c-226">Click the **Download** button on this page to download a zip file of all the taxi data for that year.</span></span>

4. <span data-ttu-id="5db6c-227">Extrayez le fichier zip dans le répertoire `DataFile`.</span><span class="sxs-lookup"><span data-stu-id="5db6c-227">Extract the zip file to the `DataFile` directory.</span></span>

    > [!NOTE]
    > <span data-ttu-id="5db6c-228">Ce fichier zip contient d’autres fichiers zip.</span><span class="sxs-lookup"><span data-stu-id="5db6c-228">This zip file contains other zip files.</span></span> <span data-ttu-id="5db6c-229">N’extrayez pas les fichiers zip enfants.</span><span class="sxs-lookup"><span data-stu-id="5db6c-229">Don't extract the child zip files.</span></span>

    <span data-ttu-id="5db6c-230">La structure de répertoire doit ressembler à ce qui suit :</span><span class="sxs-lookup"><span data-stu-id="5db6c-230">The directory structure should look like the following:</span></span>

    ```
    /data
        /streaming_azuredatabricks
            /DataFile
                /FOIL2013
                    trip_data_1.zip
                    trip_data_2.zip
                    trip_data_3.zip
                    ...
    ```

5. <span data-ttu-id="5db6c-231">Ouvrez un navigateur web et accédez à https://www.zillow.com/howto/api/neighborhood-boundaries.htm.</span><span class="sxs-lookup"><span data-stu-id="5db6c-231">Open a web browser and navigate to https://www.zillow.com/howto/api/neighborhood-boundaries.htm.</span></span> 

6. <span data-ttu-id="5db6c-232">Cliquez sur **New York Neighborhood Boundaries** pour télécharger le fichier.</span><span class="sxs-lookup"><span data-stu-id="5db6c-232">Click on **New York Neighborhood Boundaries** to download the file.</span></span>

7. <span data-ttu-id="5db6c-233">Copiez le fichier **ZillowNeighborhoods-NY.zip** du répertoire de **téléchargements** de votre navigateur vers le répertoire `DataFile`.</span><span class="sxs-lookup"><span data-stu-id="5db6c-233">Copy the **ZillowNeighborhoods-NY.zip** file from your browser's **downloads** directory to the `DataFile` directory.</span></span>

### <a name="deploy-the-azure-resources"></a><span data-ttu-id="5db6c-234">Déployer les ressources Azure</span><span class="sxs-lookup"><span data-stu-id="5db6c-234">Deploy the Azure resources</span></span>

1. <span data-ttu-id="5db6c-235">À partir d’une invite de commandes Windows ou d’un interpréteur de commandes, exécutez la commande suivante et suivez l’invite de connexion :</span><span class="sxs-lookup"><span data-stu-id="5db6c-235">From a shell or Windows Command Prompt, run the following command and follow the sign-in prompt:</span></span>

    ```bash
    az login
    ```

2. <span data-ttu-id="5db6c-236">Accédez au dossier `data/streaming_azuredatabricks` du dépôt GitHub.</span><span class="sxs-lookup"><span data-stu-id="5db6c-236">Navigate to the folder `data/streaming_azuredatabricks` in the GitHub repository</span></span>

    ```bash
    cd data/streaming_azuredatabricks
    ```

3. <span data-ttu-id="5db6c-237">Exécutez les commandes suivantes pour déployer les ressources Azure :</span><span class="sxs-lookup"><span data-stu-id="5db6c-237">Run the following commands to deploy the Azure resources:</span></span>

    ```bash
    export resourceGroup='[Resource group name]'
    export resourceLocation='[Region]'
    export eventHubNamespace='[Event Hubs namespace name]'
    export databricksWorkspaceName='[Azure Databricks workspace name]'
    export cosmosDatabaseAccount='[Cosmos DB database name]'
    export logAnalyticsWorkspaceName='[Log Analytics workspace name]'
    export logAnalyticsWorkspaceRegion='[Log Analytics region]'

    # Create a resource group
    az group create --name $resourceGroup --location $resourceLocation

    # Deploy resources
    az group deployment create --resource-group $resourceGroup \
        --template-file ./azure/deployresources.json --parameters \
        eventHubNamespace=$eventHubNamespace \
        databricksWorkspaceName=$databricksWorkspaceName \
        cosmosDatabaseAccount=$cosmosDatabaseAccount \
        logAnalyticsWorkspaceName=$logAnalyticsWorkspaceName \
        logAnalyticsWorkspaceRegion=$logAnalyticsWorkspaceRegion
    ```

4. <span data-ttu-id="5db6c-238">La sortie du déploiement est écrite dans la console à l’issue de l’opération.</span><span class="sxs-lookup"><span data-stu-id="5db6c-238">The output of the deployment is written to the console once complete.</span></span> <span data-ttu-id="5db6c-239">Recherchez dans la sortie le code JSON suivant :</span><span class="sxs-lookup"><span data-stu-id="5db6c-239">Search the output for the following JSON:</span></span>

```JSON
"outputs": {
        "cosmosDb": {
          "type": "Object",
          "value": {
            "hostName": <value>,
            "secret": <value>,
            "username": <value>
          }
        },
        "eventHubs": {
          "type": "Object",
          "value": {
            "taxi-fare-eh": <value>,
            "taxi-ride-eh": <value>
          }
        },
        "logAnalytics": {
          "type": "Object",
          "value": {
            "secret": <value>,
            "workspaceId": <value>
          }
        }
},
```
<span data-ttu-id="5db6c-240">Ces valeurs correspondent aux secrets qui seront ajoutés aux secrets Databricks dans les prochaines sections.</span><span class="sxs-lookup"><span data-stu-id="5db6c-240">These values are the secrets that will be added to Databricks secrets in upcoming sections.</span></span> <span data-ttu-id="5db6c-241">Gardez-les en lieu sûr en attendant de les ajouter dans ces sections.</span><span class="sxs-lookup"><span data-stu-id="5db6c-241">Keep them secure until you add them in those sections.</span></span>

### <a name="add-a-cassandra-table-to-the-cosmos-db-account"></a><span data-ttu-id="5db6c-242">Ajouter une table Cassandra au compte Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="5db6c-242">Add a Cassandra table to the Cosmos DB Account</span></span>

1. <span data-ttu-id="5db6c-243">Sur le portail Azure, accédez au groupe de ressources créé dans la section **Déployer les ressources Azure** ci-dessus.</span><span class="sxs-lookup"><span data-stu-id="5db6c-243">In the Azure portal, navigate to the resource group created in the **deploy the Azure resources** section above.</span></span> <span data-ttu-id="5db6c-244">Cliquez sur **Compte Azure Cosmos DB**.</span><span class="sxs-lookup"><span data-stu-id="5db6c-244">Click on **Azure Cosmos DB Account**.</span></span> <span data-ttu-id="5db6c-245">Créez une table avec l’API Cassandra.</span><span class="sxs-lookup"><span data-stu-id="5db6c-245">Create a table with the Cassandra API.</span></span>

2. <span data-ttu-id="5db6c-246">Dans le panneau **overview** (vue d’ensemble), cliquez sur **add table** (ajouter une table).</span><span class="sxs-lookup"><span data-stu-id="5db6c-246">In the **overview** blade, click **add table**.</span></span>

3. <span data-ttu-id="5db6c-247">Quand le panneau **add table** s’ouvre, entrez `newyorktaxi` dans la zone de texte **Keyspace name** (Nom de l’espace de clés).</span><span class="sxs-lookup"><span data-stu-id="5db6c-247">When the **add table** blade opens, enter `newyorktaxi` in the **Keyspace name** text box.</span></span> 

4. <span data-ttu-id="5db6c-248">Dans la section **enter CQL command to create the table** (entrer la commande CQL pour créer la table), entrez `neighborhoodstats` dans la zone de texte en regard de `newyorktaxi`.</span><span class="sxs-lookup"><span data-stu-id="5db6c-248">In the **enter CQL command to create the table** section, enter `neighborhoodstats` in the text box beside `newyorktaxi`.</span></span>

5. <span data-ttu-id="5db6c-249">Dans la zone de texte ci-dessous, entrez ce qui suit :</span><span class="sxs-lookup"><span data-stu-id="5db6c-249">In the text box below, enter the following::</span></span>
```
(neighborhood text, window_end timestamp, number_of_rides bigint,total_fare_amount double, primary key(neighborhood, window_end))
```
6. <span data-ttu-id="5db6c-250">Dans la zone de texte **Throughput (1,000 - 1,000,000 RU/s)** (Débit (1 000 - 1 000 000 de RU/s)), entrez la valeur `4000`.</span><span class="sxs-lookup"><span data-stu-id="5db6c-250">In the **Throughput (1,000 - 1,000,000 RU/s)** text box enter the value `4000`.</span></span>

7. <span data-ttu-id="5db6c-251">Cliquez sur **OK**.</span><span class="sxs-lookup"><span data-stu-id="5db6c-251">Click **OK**.</span></span>

### <a name="add-the-databricks-secrets-using-the-databricks-cli"></a><span data-ttu-id="5db6c-252">Ajouter les secrets Databricks à l’aide de l’interface CLI Databricks</span><span class="sxs-lookup"><span data-stu-id="5db6c-252">Add the Databricks secrets using the Databricks CLI</span></span>

<span data-ttu-id="5db6c-253">Pour commencer, entrez les secrets pour EventHub :</span><span class="sxs-lookup"><span data-stu-id="5db6c-253">First, enter the secrets for EventHub:</span></span>

1. <span data-ttu-id="5db6c-254">À partir de l’interface **CLI Azure Databricks** installée à l’étape 2 des prérequis, créez l’étendue des secrets Azure Databricks :</span><span class="sxs-lookup"><span data-stu-id="5db6c-254">Using the **Azure Databricks CLI** installed in step 2 of the prerequisites, create the Azure Databricks secret scope:</span></span>
    ```
    databricks secrets create-scope --scope "azure-databricks-job"
    ```
2. <span data-ttu-id="5db6c-255">Ajoutez le secret pour le hub d’événements des courses de taxis (taxi-ride) :</span><span class="sxs-lookup"><span data-stu-id="5db6c-255">Add the secret for the taxi ride EventHub:</span></span>
    ```
    databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
    ```
    <span data-ttu-id="5db6c-256">Une fois exécutée, cette commande ouvre l’éditeur vi.</span><span class="sxs-lookup"><span data-stu-id="5db6c-256">Once executed, this command opens the vi editor.</span></span> <span data-ttu-id="5db6c-257">Entrez la valeur de **taxi-tour-eh** de la section de sortie **eventHubs** de l’étape 4 de la section *Déployer les ressources Azure*.</span><span class="sxs-lookup"><span data-stu-id="5db6c-257">Enter the **taxi-ride-eh** value from the **eventHubs** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="5db6c-258">Enregistrez et quittez vi.</span><span class="sxs-lookup"><span data-stu-id="5db6c-258">Save and exit vi.</span></span>

3. <span data-ttu-id="5db6c-259">Ajoutez le secret pour le hub d’événements des tarifs de taxi (taxi-fare) :</span><span class="sxs-lookup"><span data-stu-id="5db6c-259">Add the secret for the taxi fare EventHub:</span></span>
    ```
    databricks secrets put --scope "azure-databricks-job" --key "taxi-fare"
    ```
    <span data-ttu-id="5db6c-260">Une fois exécutée, cette commande ouvre l’éditeur vi.</span><span class="sxs-lookup"><span data-stu-id="5db6c-260">Once executed, this command opens the vi editor.</span></span> <span data-ttu-id="5db6c-261">Entrez la valeur de **taxi-fare-eh** de la section de sortie **eventHubs** de l’étape 4 de la section *Déployer les ressources Azure*.</span><span class="sxs-lookup"><span data-stu-id="5db6c-261">Enter the **taxi-fare-eh** value from the **eventHubs** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="5db6c-262">Enregistrez et quittez vi.</span><span class="sxs-lookup"><span data-stu-id="5db6c-262">Save and exit vi.</span></span>

<span data-ttu-id="5db6c-263">Ensuite, entrez les secrets pour Cosmos DB :</span><span class="sxs-lookup"><span data-stu-id="5db6c-263">Next, enter the secrets for Cosmos DB:</span></span>

1. <span data-ttu-id="5db6c-264">Ouvrez le portail Azure, puis accédez au groupe de ressources spécifié à l’étape 3 de la section **Déployer les ressources Azure**.</span><span class="sxs-lookup"><span data-stu-id="5db6c-264">Open the Azure portal, and navigate to the resource group specified in step 3 of the **deploy the Azure resources** section.</span></span> <span data-ttu-id="5db6c-265">Cliquez sur le compte Azure Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="5db6c-265">Click on the Azure Cosmos DB Account.</span></span>

2. <span data-ttu-id="5db6c-266">À partir de l’interface **CLI Azure Databricks**, ajoutez le secret pour le nom d’utilisateur Cosmos DB :</span><span class="sxs-lookup"><span data-stu-id="5db6c-266">Using the **Azure Databricks CLI**, add the secret for the Cosmos DB user name:</span></span>
    ```
    databricks secrets put --scope azure-databricks-job --key "cassandra-username"
    ```
<span data-ttu-id="5db6c-267">Une fois exécutée, cette commande ouvre l’éditeur vi.</span><span class="sxs-lookup"><span data-stu-id="5db6c-267">Once executed, this command opens the vi editor.</span></span> <span data-ttu-id="5db6c-268">Entrez la valeur de **username** de la section de sortie **CosmosDb** de l’étape 4 de la section *Déployer les ressources Azure*.</span><span class="sxs-lookup"><span data-stu-id="5db6c-268">Enter the **username** value from the **CosmosDb** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="5db6c-269">Enregistrez et quittez vi.</span><span class="sxs-lookup"><span data-stu-id="5db6c-269">Save and exit vi.</span></span>

3. <span data-ttu-id="5db6c-270">Ensuite, ajoutez le secret pour le mot de passe Cosmos DB :</span><span class="sxs-lookup"><span data-stu-id="5db6c-270">Next, add the secret for the Cosmos DB password:</span></span>
    ```
    databricks secrets put --scope azure-databricks-job --key "cassandra-password"
    ```

<span data-ttu-id="5db6c-271">Une fois exécutée, cette commande ouvre l’éditeur vi.</span><span class="sxs-lookup"><span data-stu-id="5db6c-271">Once executed, this command opens the vi editor.</span></span> <span data-ttu-id="5db6c-272">Entrez la valeur de **secret** de la section de sortie **CosmosDb** de l’étape 4 de la section *Déployer les ressources Azure*.</span><span class="sxs-lookup"><span data-stu-id="5db6c-272">Enter the **secret** value from the **CosmosDb** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="5db6c-273">Enregistrez et quittez vi.</span><span class="sxs-lookup"><span data-stu-id="5db6c-273">Save and exit vi.</span></span>

> [!NOTE]
> <span data-ttu-id="5db6c-274">Si vous utilisez une [étendue de secrets adossée à Azure Key Vault](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes), elle doit se nommer **azure-databricks-job** et les secrets doivent avoir exactement les mêmes noms que ceux mentionnés ci-dessus.</span><span class="sxs-lookup"><span data-stu-id="5db6c-274">If using an [Azure Key Vault-backed secret scope](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes), the scope must be named **azure-databricks-job** and the secrets must have the exact same names as those above.</span></span>

### <a name="add-the-zillow-neighborhoods-data-file-to-the-databricks-file-system"></a><span data-ttu-id="5db6c-275">Ajouter le fichier de données Zillow Neighborhoods au système de fichiers Databricks</span><span class="sxs-lookup"><span data-stu-id="5db6c-275">Add the Zillow Neighborhoods data file to the Databricks file system</span></span>

1. <span data-ttu-id="5db6c-276">Créez un répertoire dans le système de fichiers Databricks :</span><span class="sxs-lookup"><span data-stu-id="5db6c-276">Create a directory in the Databricks file system:</span></span>
    ```bash
    dbfs mkdirs dbfs:/azure-databricks-jobs
    ```

2. <span data-ttu-id="5db6c-277">Accédez à data/streaming_azuredatabricks/DateFile et entrez ce qui suit :</span><span class="sxs-lookup"><span data-stu-id="5db6c-277">Navigate to data/streaming_azuredatabricks/DataFile and enter the following:</span></span>
    ```bash
    dbfs cp ZillowNeighborhoods-NY.zip dbfs:/azure-databricks-jobs
    ```

### <a name="add-the-azure-log-analytics-workspace-id-and-primary-key-to-configuration-files"></a><span data-ttu-id="5db6c-278">Ajouter l’ID d’espace de travail Azure Log Analytics et la clé primaire aux fichiers de configuration</span><span class="sxs-lookup"><span data-stu-id="5db6c-278">Add the Azure Log Analytics workspace ID and primary key to configuration files</span></span>

<span data-ttu-id="5db6c-279">Pour cette section, vous avez besoin de l’ID et de la clé primaire de l’espace de travail Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="5db6c-279">For this section, you require the Log Analytics workspace ID and primary key.</span></span> <span data-ttu-id="5db6c-280">L’ID de l’espace de travail est la valeur de **workspaceId** dans la section de sortie **logAnalytics** de l’étape 4 de la section *Déployer les ressources Azure*.</span><span class="sxs-lookup"><span data-stu-id="5db6c-280">The workspace ID is the **workspaceId** value from the **logAnalytics** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="5db6c-281">La clé primaire est le **secret** de la section de sortie.</span><span class="sxs-lookup"><span data-stu-id="5db6c-281">The primary key is the **secret** from the output section.</span></span> 

1. <span data-ttu-id="5db6c-282">Pour configurer la journalisation log4j, ouvrez data\streaming_azuredatabricks\azure\AzureDataBricksJob\src\main\resources\com\microsoft\pnp\azuredatabricksjob\log4j.properties.</span><span class="sxs-lookup"><span data-stu-id="5db6c-282">To configure log4j logging, open data\streaming_azuredatabricks\azure\AzureDataBricksJob\src\main\resources\com\microsoft\pnp\azuredatabricksjob\log4j.properties.</span></span> <span data-ttu-id="5db6c-283">Modifiez les deux valeurs suivantes :</span><span class="sxs-lookup"><span data-stu-id="5db6c-283">Edit the following two values:</span></span>
    ```
    log4j.appender.A1.workspaceId=<Log Analytics workspace ID>
    log4j.appender.A1.secret=<Log Analytics primary key>
    ```

2. <span data-ttu-id="5db6c-284">Pour configurer la journalisation personnalisée, ouvrez data\streaming_azuredatabricks\azure\azure-databricks-monitoring\scripts\metrics.properties.</span><span class="sxs-lookup"><span data-stu-id="5db6c-284">To configure custom logging, open data\streaming_azuredatabricks\azure\azure-databricks-monitoring\scripts\metrics.properties.</span></span> <span data-ttu-id="5db6c-285">Modifiez les deux valeurs suivantes :</span><span class="sxs-lookup"><span data-stu-id="5db6c-285">Edit the following two values:</span></span>
    ``` 
    *.sink.loganalytics.workspaceId=<Log Analytics workspace ID>
    *.sink.loganalytics.secret=<Log Analytics primary key>
    ```

### <a name="build-the-jar-files-for-the-databricks-job-and-databricks-monitoring"></a><span data-ttu-id="5db6c-286">Générer les fichiers .jar pour le travail Databricks et la supervision Databricks</span><span class="sxs-lookup"><span data-stu-id="5db6c-286">Build the .jar files for the Databricks job and Databricks monitoring</span></span>

1. <span data-ttu-id="5db6c-287">Utilisez votre environnement de développement Java pour importer le fichier projet Maven nommé **pom.xml** situé à la racine du répertoire **data/streaming_azuredatabricks**.</span><span class="sxs-lookup"><span data-stu-id="5db6c-287">Use your Java IDE to import the Maven project file named **pom.xml** located in the root of the **data/streaming_azuredatabricks** directory.</span></span> 

2. <span data-ttu-id="5db6c-288">Optez pour une build propre.</span><span class="sxs-lookup"><span data-stu-id="5db6c-288">Perform a clean build.</span></span> <span data-ttu-id="5db6c-289">La sortie de cette build correspond aux fichiers nommés **azure-databricks-job-1.0-SNAPSHOT.jar** et **azure-databricks-monitoring-0.9.jar**.</span><span class="sxs-lookup"><span data-stu-id="5db6c-289">The output of this build is files named **azure-databricks-job-1.0-SNAPSHOT.jar** and **azure-databricks-monitoring-0.9.jar**.</span></span> 

### <a name="configure-custom-logging-for-the-databricks-job"></a><span data-ttu-id="5db6c-290">Configurer la journalisation personnalisée pour le travail Databricks</span><span class="sxs-lookup"><span data-stu-id="5db6c-290">Configure custom logging for the Databricks job</span></span>

1. <span data-ttu-id="5db6c-291">Copiez le fichier **azure-databricks-surveillance-0.9.jar** dans le système de fichiers Databricks en entrant la commande suivante dans l’interface **CLI Databricks** :</span><span class="sxs-lookup"><span data-stu-id="5db6c-291">Copy the **azure-databricks-monitoring-0.9.jar** file to the Databricks file system by entering the following command in the **Databricks CLI**:</span></span>
    ```
    databricks fs cp --overwrite azure-databricks-monitoring-0.9.jar dbfs:/azure-databricks-job/azure-databricks-monitoring-0.9.jar
    ```

2. <span data-ttu-id="5db6c-292">Copiez les propriétés de journalisation personnalisée de data\streaming_azuredatabricks\azure\azure-databricks-monitoring\scripts\metrics.properties vers le système de fichiers Databricks en entrant la commande suivante :</span><span class="sxs-lookup"><span data-stu-id="5db6c-292">Copy the custom logging properties from data\streaming_azuredatabricks\azure\azure-databricks-monitoring\scripts\metrics.properties to the Databricks file system by entering the following command:</span></span>
    ```
    databricks fs cp --overwrite metrics.properties dbfs:/azure-databricks-job/metrics.properties
    ```

3. <span data-ttu-id="5db6c-293">À ce stade, vous n’avez pas encore nommé votre cluster Databricks ; attribuez-lui un nom maintenant.</span><span class="sxs-lookup"><span data-stu-id="5db6c-293">While you haven't yet decided on a name for your Databricks cluster, select one now.</span></span> <span data-ttu-id="5db6c-294">Vous devez entrer le nom ci-dessous dans le chemin du système de fichiers Databricks de votre cluster.</span><span class="sxs-lookup"><span data-stu-id="5db6c-294">You'll enter the name below in the Databricks file system path for your cluster.</span></span> <span data-ttu-id="5db6c-295">Copiez le script d’initialisation de data\streaming_azuredatabricks\azure\azure-databricks-monitoring\scripts\sparks.properties vers le système de fichiers Databricks en entrant la commande suivante :</span><span class="sxs-lookup"><span data-stu-id="5db6c-295">Copy the initialization script from data\streaming_azuredatabricks\azure\azure-databricks-monitoring\scripts\spark.metrics to the Databricks file system by entering the following command:</span></span>
    ```
    databricks fs cp --overwrite spark-metrics.sh dbfs:/databricks/init/<cluster-name>/spark-metrics.sh
    ```

### <a name="create-a-databricks-cluster"></a><span data-ttu-id="5db6c-296">Créer un cluster Databricks</span><span class="sxs-lookup"><span data-stu-id="5db6c-296">Create a Databricks cluster</span></span>

1. <span data-ttu-id="5db6c-297">Dans l’espace de travail Databricks, cliquez sur Clusters, puis sur create cluster.</span><span class="sxs-lookup"><span data-stu-id="5db6c-297">In the Databricks workspace, click "Clusters", then click "create cluster".</span></span> <span data-ttu-id="5db6c-298">Entrez le nom du cluster que vous avez créé à l’étape 3 de la section **Configurer la journalisation personnalisée pour le travail Databricks** ci-dessus.</span><span class="sxs-lookup"><span data-stu-id="5db6c-298">Enter the cluster name you created in step 3 of the **configure custom logging for the Databricks job** section above.</span></span> 

2. <span data-ttu-id="5db6c-299">Sélectionnez un mode de cluster **standard**.</span><span class="sxs-lookup"><span data-stu-id="5db6c-299">Select a **standard** cluster mode.</span></span>

3. <span data-ttu-id="5db6c-300">Définissez **Databricks runtime version** (Version du runtime Databricks) sur **4.3 (comprend Apache Spark 2.3.1, Scala 2.11)**</span><span class="sxs-lookup"><span data-stu-id="5db6c-300">Set **Databricks runtime version** to **4.3 (includes Apache Spark 2.3.1, Scala 2.11)**</span></span>

4. <span data-ttu-id="5db6c-301">Définissez **Python version** (Version de Python) sur **2**.</span><span class="sxs-lookup"><span data-stu-id="5db6c-301">Set **Python version** to **2**.</span></span>

5. <span data-ttu-id="5db6c-302">Définissez **Driver Type** (Type de pilote) sur **Same as worker** (Identique au Worker).</span><span class="sxs-lookup"><span data-stu-id="5db6c-302">Set **Driver Type** to **Same as worker**</span></span>

6. <span data-ttu-id="5db6c-303">Définissez **Worker Type** (Type de Worker) sur **Standard_DS3_v2**.</span><span class="sxs-lookup"><span data-stu-id="5db6c-303">Set **Worker Type** to **Standard_DS3_v2**.</span></span>

7. <span data-ttu-id="5db6c-304">Définissez **Min Workers** (Nombre minimum de Workers) sur **2**.</span><span class="sxs-lookup"><span data-stu-id="5db6c-304">Set **Min Workers** to **2**.</span></span>

8. <span data-ttu-id="5db6c-305">Désélectionnez **Enable autoscaling** (Activer la mise à l’échelle automatique).</span><span class="sxs-lookup"><span data-stu-id="5db6c-305">Deselect **Enable autoscaling**.</span></span> 

9. <span data-ttu-id="5db6c-306">En dessous de la boîte de dialogue **Auto Termination** (Arrêt automatique), cliquez sur **Init Scripts** (Scripts d’initialisation).</span><span class="sxs-lookup"><span data-stu-id="5db6c-306">Below the **Auto Termination** dialog box, click on **Init Scripts**.</span></span> 

10. <span data-ttu-id="5db6c-307">Entrez **dbfs:/databricks/init/<cluster-name>/spark-metrics.sh** en remplaçant <cluster-name> par le nom de cluster créé à l’étape 1.</span><span class="sxs-lookup"><span data-stu-id="5db6c-307">Enter **dbfs:/databricks/init/<cluster-name>/spark-metrics.sh**, substituting the cluster name created in step 1 for <cluster-name>.</span></span>

11. <span data-ttu-id="5db6c-308">Cliquez sur le bouton **Add**.</span><span class="sxs-lookup"><span data-stu-id="5db6c-308">Click the **Add** button.</span></span>

12. <span data-ttu-id="5db6c-309">Cliquez sur le bouton **Create Cluster**.</span><span class="sxs-lookup"><span data-stu-id="5db6c-309">Click the **Create Cluster** button.</span></span>

### <a name="create-a-databricks-job"></a><span data-ttu-id="5db6c-310">Créer un travail Databricks</span><span class="sxs-lookup"><span data-stu-id="5db6c-310">Create a Databricks job</span></span>

1. <span data-ttu-id="5db6c-311">Dans l’espace de travail Databricks, cliquez sur Jobs et sur create job.</span><span class="sxs-lookup"><span data-stu-id="5db6c-311">In the Databricks workspace, click "Jobs", "create job".</span></span>

2. <span data-ttu-id="5db6c-312">Entrez un nom de travail.</span><span class="sxs-lookup"><span data-stu-id="5db6c-312">Enter a job name.</span></span>

3. <span data-ttu-id="5db6c-313">Cliquez sur « set jar » pour accéder à la boîte de dialogue Upload JAR to Run (Charger le JAR à exécuter).</span><span class="sxs-lookup"><span data-stu-id="5db6c-313">Click "set jar", this opens the "Upload JAR to Run" dialog box.</span></span>

4. <span data-ttu-id="5db6c-314">Faites glisser le fichier **azure-databricks-job-1.0-SNAPSHOT.jar** que vous avez créé dans la section **Générer le fichier .jar pour le travail Databricks** vers la zone **Drop JAR here to upload** (Déposer le JAR à charger ici).</span><span class="sxs-lookup"><span data-stu-id="5db6c-314">Drag the **azure-databricks-job-1.0-SNAPSHOT.jar** file you created in the **build the .jar for the Databricks job** section to the **Drop JAR here to upload** box.</span></span>

5. <span data-ttu-id="5db6c-315">Entrez **com.microsoft.pnp.TaxiCabReader** dans le champ **Main Class** (Classe principale).</span><span class="sxs-lookup"><span data-stu-id="5db6c-315">Enter **com.microsoft.pnp.TaxiCabReader** in the **Main Class** field.</span></span>

6. <span data-ttu-id="5db6c-316">Dans le champ arguments, entrez ce qui suit :</span><span class="sxs-lookup"><span data-stu-id="5db6c-316">In the arguments field, enter the following:</span></span>
    ```
    -n jar:file:/dbfs/azure-databricks-jobs/ZillowNeighborhoods-NY.zip!/ZillowNeighborhoods-NY.shp --taxi-ride-consumer-group taxi-ride-eh-cg --taxi-fare-consumer-group taxi-fare-eh-cg --window-interval "1 minute" --cassandra-host <Cosmos DB Cassandra host name from above> 
    ``` 

7. <span data-ttu-id="5db6c-317">Installez les bibliothèques dépendantes en suivant ces étapes :</span><span class="sxs-lookup"><span data-stu-id="5db6c-317">Install the dependent libraries by following these steps:</span></span>
    
    1. <span data-ttu-id="5db6c-318">Dans l’interface utilisateur Databricks, cliquez sur le bouton **home** (accueil).</span><span class="sxs-lookup"><span data-stu-id="5db6c-318">In the Databricks user interface, click on the **home** button.</span></span>
    
    2. <span data-ttu-id="5db6c-319">Dans la liste déroulante **Users** (Utilisateurs), cliquez sur le nom de votre compte d’utilisateur pour ouvrir les paramètres d’espace de travail de votre compte.</span><span class="sxs-lookup"><span data-stu-id="5db6c-319">In the **Users** drop-down, click on your user account name to open your account workspace settings.</span></span>
    
    3. <span data-ttu-id="5db6c-320">Cliquez sur la flèche déroulante en regard du nom de votre compte, cliquez sur **create**, puis sur **Library** (Bibliothèque) pour ouvrir la boîte de dialogue **New Library** (Nouvelle bibliothèque).</span><span class="sxs-lookup"><span data-stu-id="5db6c-320">Click on the drop-down arrow beside your account name, click on **create**, and click on **Library** to open the **New Library** dialog.</span></span>
    
    4. <span data-ttu-id="5db6c-321">Dans la liste déroulante **Source**, sélectionnez **Maven Coordinate** (Coordonnée Maven).</span><span class="sxs-lookup"><span data-stu-id="5db6c-321">In the **Source** drop-down control, select **Maven Coordinate**.</span></span>
    
    5. <span data-ttu-id="5db6c-322">Sous l’en-tête **Install Maven Artifacts** (Installer les artefacts Maven), entrez `com.microsoft.azure:azure-eventhubs-spark_2.11:2.3.5` dans la zone de texte **Coordinate**.</span><span class="sxs-lookup"><span data-stu-id="5db6c-322">Under the **Install Maven Artifacts** heading, enter `com.microsoft.azure:azure-eventhubs-spark_2.11:2.3.5` in the **Coordinate** text box.</span></span> 
    
    6. <span data-ttu-id="5db6c-323">Cliquez sur **Create Library** (Créer une bibliothèque) pour ouvrir la fenêtre **Artifacts**.</span><span class="sxs-lookup"><span data-stu-id="5db6c-323">Click on **Create Library** to open the **Artifacts** window.</span></span>
    
    7. <span data-ttu-id="5db6c-324">Sous **Status on running clusters** (État sur les clusters en cours d’exécution), cochez la case **Attach automatically to all clusters** (Joindre automatiquement à tous les clusters).</span><span class="sxs-lookup"><span data-stu-id="5db6c-324">Under **Status on running clusters** check the **Attach automatically to all clusters** checkbox.</span></span>
    
    8. <span data-ttu-id="5db6c-325">Répétez les étapes 1 à 7 pour la coordonnée Maven `com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.0.0`.</span><span class="sxs-lookup"><span data-stu-id="5db6c-325">Repeat steps 1 - 7 for the `com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.0.0` Maven coordinate.</span></span>
    
    9. <span data-ttu-id="5db6c-326">Répétez les étapes 1 à 6 pour la coordonnée Maven `org.geotools:gt-shapefile:19.2`.</span><span class="sxs-lookup"><span data-stu-id="5db6c-326">Repeat steps 1 - 6 for the `org.geotools:gt-shapefile:19.2` Maven coordinate.</span></span>
    
    10. <span data-ttu-id="5db6c-327">Cliquez sur **Advanced Options** (Options avancées).</span><span class="sxs-lookup"><span data-stu-id="5db6c-327">Click on **Advanced Options**.</span></span>
    
    11. <span data-ttu-id="5db6c-328">Entrez `http://download.osgeo.org/webdav/geotools/` dans la zone de texte **Repository** (Référentiel).</span><span class="sxs-lookup"><span data-stu-id="5db6c-328">Enter `http://download.osgeo.org/webdav/geotools/` in the **Repository** text box.</span></span> 
    
    12. <span data-ttu-id="5db6c-329">Cliquez sur **Create Library** (Créer une bibliothèque) pour ouvrir la fenêtre **Artifacts**.</span><span class="sxs-lookup"><span data-stu-id="5db6c-329">Click **Create Library** to open the **Artifacts** window.</span></span> 
    
    13. <span data-ttu-id="5db6c-330">Sous **Status on running clusters** (État sur les clusters en cours d’exécution), cochez la case **Attach automatically to all clusters** (Joindre automatiquement à tous les clusters).</span><span class="sxs-lookup"><span data-stu-id="5db6c-330">Under **Status on running clusters** check the **Attach automatically to all clusters** checkbox.</span></span>

8. <span data-ttu-id="5db6c-331">Ajoutez les bibliothèques dépendantes ajoutées à l’étape 7 au travail créé à la fin de l’étape 6 :</span><span class="sxs-lookup"><span data-stu-id="5db6c-331">Add the dependent libraries added in step 7 to the job created at the end of step 6:</span></span>
    1. <span data-ttu-id="5db6c-332">Dans l’espace de travail Azure Databricks, cliquez sur **Jobs** (Travaux).</span><span class="sxs-lookup"><span data-stu-id="5db6c-332">In the Azure Databricks workspace, click on **Jobs**.</span></span>

    2. <span data-ttu-id="5db6c-333">Cliquez sur le nom du travail créé à l’étape 2 de la section **Créer un travail Databricks**.</span><span class="sxs-lookup"><span data-stu-id="5db6c-333">Click on the job name created in step 2 of the **create a Databricks job** section.</span></span> 
    
    3. <span data-ttu-id="5db6c-334">En regard de la section **Dependent Libraries** (Bibliothèques dépendantes), cliquez sur **Add** pour ouvrir la boîte de dialogue **Add Dependent Library** (Ajouter une bibliothèque dépendante).</span><span class="sxs-lookup"><span data-stu-id="5db6c-334">Beside the **Dependent Libraries** section, click on **Add** to open the **Add Dependent Library** dialog.</span></span> 
    
    4. <span data-ttu-id="5db6c-335">Sous **Library From** (Bibliothèque de), sélectionnez **Workspace** (Espace de travail).</span><span class="sxs-lookup"><span data-stu-id="5db6c-335">Under **Library From** select **Workspace**.</span></span>
    
    5. <span data-ttu-id="5db6c-336">Cliquez successivement sur **users** (utilisateurs), sur votre nom d’utilisateur, puis sur `azure-eventhubs-spark_2.11:2.3.5`.</span><span class="sxs-lookup"><span data-stu-id="5db6c-336">Click on **users**, then your username, then click on `azure-eventhubs-spark_2.11:2.3.5`.</span></span> 
    
    6. <span data-ttu-id="5db6c-337">Cliquez sur **OK**.</span><span class="sxs-lookup"><span data-stu-id="5db6c-337">Click **OK**.</span></span>
    
    7. <span data-ttu-id="5db6c-338">Répétez les étapes 1 à 6 pour `spark-cassandra-connector_2.11:2.3.1` et `gt-shapefile:19.2`.</span><span class="sxs-lookup"><span data-stu-id="5db6c-338">Repeat steps 1 - 6 for `spark-cassandra-connector_2.11:2.3.1` and `gt-shapefile:19.2`.</span></span>

9. <span data-ttu-id="5db6c-339">En regard de **Cluster:**, cliquez sur **Edit** (Modifier).</span><span class="sxs-lookup"><span data-stu-id="5db6c-339">Beside **Cluster:**, click on **Edit**.</span></span> <span data-ttu-id="5db6c-340">La boîte de dialogue **Configurer Cluster** (Configurer le cluster) s’ouvre.</span><span class="sxs-lookup"><span data-stu-id="5db6c-340">This opens the **Configure Cluster** dialog.</span></span> <span data-ttu-id="5db6c-341">Dans la liste déroulante **Cluster Type** (Type de cluster), sélectionnez **Existing Cluster** (Cluster existant).</span><span class="sxs-lookup"><span data-stu-id="5db6c-341">In the **Cluster Type** drop-down, select **Existing Cluster**.</span></span> <span data-ttu-id="5db6c-342">Dans la liste déroulante **Select Cluster**, sélectionnez le cluster créé dans la section **Créer un cluster Databricks**.</span><span class="sxs-lookup"><span data-stu-id="5db6c-342">In the **Select Cluster** drop-down, select the cluster created the **create a Databricks cluster** section.</span></span> <span data-ttu-id="5db6c-343">Cliquez sur **confirm**.</span><span class="sxs-lookup"><span data-stu-id="5db6c-343">Click **confirm**.</span></span>

10. <span data-ttu-id="5db6c-344">Cliquez sur **run now** (exécuter maintenant).</span><span class="sxs-lookup"><span data-stu-id="5db6c-344">Click **run now**.</span></span>

### <a name="run-the-data-generator"></a><span data-ttu-id="5db6c-345">Exécuter le générateur de données</span><span class="sxs-lookup"><span data-stu-id="5db6c-345">Run the data generator</span></span>

1. <span data-ttu-id="5db6c-346">Accédez au répertoire `data/streaming_azuredatabricks/onprem` dans le dépôt GitHub.</span><span class="sxs-lookup"><span data-stu-id="5db6c-346">Navigate to the directory `data/streaming_azuredatabricks/onprem` in the GitHub repository.</span></span>

2. <span data-ttu-id="5db6c-347">Mettez à jour les valeurs du fichier **main.env** comme suit :</span><span class="sxs-lookup"><span data-stu-id="5db6c-347">Update the values in the file **main.env** as follows:</span></span>

    ```
    RIDE_EVENT_HUB=[Connection string for the taxi-ride event hub]
    FARE_EVENT_HUB=[Connection string for the taxi-fare event hub]
    RIDE_DATA_FILE_PATH=/DataFile/FOIL2013
    MINUTES_TO_LEAD=0
    PUSH_RIDE_DATA_FIRST=false
    ```
    <span data-ttu-id="5db6c-348">La chaîne de connexion pour le hub d’événements taxi-ride est la valeur de **taxi-ride-eh** dans la section de sortie **eventHubs** de l’étape 4 de la section *Déployer les ressources Azure*.</span><span class="sxs-lookup"><span data-stu-id="5db6c-348">The connection string for the taxi-ride event hub is the **taxi-ride-eh** value from the **eventHubs** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="5db6c-349">La chaîne de connexion pour le hub d’événements taxi-fare est la valeur de **taxi-fare-eh** dans la section de sortie **eventHubs** de l’étape 4 de la section *Déployer les ressources Azure*.</span><span class="sxs-lookup"><span data-stu-id="5db6c-349">The connection string for the taxi-fare event hub the **taxi-fare-eh** value from the **eventHubs** output section in step 4 of the *deploy the Azure resources* section.</span></span>

3. <span data-ttu-id="5db6c-350">Pour générer l’image Docker, exécutez la commande suivante :</span><span class="sxs-lookup"><span data-stu-id="5db6c-350">Run the following command to build the Docker image.</span></span>

    ```bash
    docker build --no-cache -t dataloader .
    ```

4. <span data-ttu-id="5db6c-351">Revenez vers le répertoire parent, `data/stream_azuredatabricks`.</span><span class="sxs-lookup"><span data-stu-id="5db6c-351">Navigate back to the parent directory, `data/stream_azuredatabricks`.</span></span>

    ```bash
    cd ..
    ```

5. <span data-ttu-id="5db6c-352">Pour exécuter l’image Docker, exécutez la commande suivante :</span><span class="sxs-lookup"><span data-stu-id="5db6c-352">Run the following command to run the Docker image.</span></span>

    ```bash
    docker run -v `pwd`/DataFile:/DataFile --env-file=onprem/main.env dataloader:latest
    ```

<span data-ttu-id="5db6c-353">La sortie doit se présenter comme suit :</span><span class="sxs-lookup"><span data-stu-id="5db6c-353">The output should look like the following:</span></span>

```
Created 10000 records for TaxiFare
Created 10000 records for TaxiRide
Created 20000 records for TaxiFare
Created 20000 records for TaxiRide
Created 30000 records for TaxiFare
...
```

<span data-ttu-id="5db6c-354">Pour vérifier que le travail Databricks s’exécute correctement, ouvrez le portail Azure et accédez à la base de données Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="5db6c-354">To verify the Databricks job is running correctly, open the Azure portal and navigate to the Cosmos DB database.</span></span> <span data-ttu-id="5db6c-355">Ouvrez le panneau **Explorateur de données** et examinez les données dans la table **taxi records** (enregistrements taxi).</span><span class="sxs-lookup"><span data-stu-id="5db6c-355">Open the **Data Explorer** blade and examine the data in the **taxi records** table.</span></span> 

<span data-ttu-id="5db6c-356">[1] <span id="note1">Donovan, Brian ; Work, Dan (2016): New York City Taxi Trip Data (2010-2013) (Données relatives aux courses des taxis de New York).</span><span class="sxs-lookup"><span data-stu-id="5db6c-356">[1] <span id="note1">Donovan, Brian; Work, Dan (2016): New York City Taxi Trip Data (2010-2013).</span></span> <span data-ttu-id="5db6c-357">Université de l’Illinois, Urbana-Champaign.</span><span class="sxs-lookup"><span data-stu-id="5db6c-357">University of Illinois at Urbana-Champaign.</span></span> <span data-ttu-id="5db6c-358">https://doi.org/10.13012/J8PN93H8</span><span class="sxs-lookup"><span data-stu-id="5db6c-358">https://doi.org/10.13012/J8PN93H8</span></span>
