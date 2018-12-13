---
title: Traitement de flux de données avec Azure Databricks
titleSuffix: Azure Reference Architectures
description: Créez un pipeline de traitement de flux de bout en bout dans Azure à l’aide d’Azure Databricks.
author: petertaylor9999
ms.date: 11/30/2018
ms.custom: seodec18
ms.openlocfilehash: 822a3c448dcc2bdd4ae77ef2a2b7a9ffad633440
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2018
ms.locfileid: "53120300"
---
# <a name="create-a-stream-processing-pipeline-with-azure-databricks"></a>Créer un pipeline de traitement de flux avec Azure Databricks

Cette architecture de référence présente un pipeline de [traitement de flux](/azure/architecture/data-guide/big-data/real-time-processing) de bout en bout. Ce type de pipeline comprend quatre étapes : ingestion, traitement, stockage, analyse et création de rapports. Pour cette architecture de référence, le pipeline ingère des données issues de deux sources, effectue une jonction sur les enregistrements apparentés de chaque flux, enrichit le résultat et calcule une moyenne en temps réel. Les résultats sont stockés en vue d’une analyse plus approfondie. [**Déployez cette solution**](#deploy-the-solution).

![Architecture de référence pour le traitement de flux avec Azure Databricks](./images/stream-processing-databricks.png)

**Scénario** : Une compagnie de taxis collecte des données sur chaque trajet effectué en taxi. Pour ce scénario, nous partons du principe que deux périphériques distincts envoient des données. Le taxi est équipé d’un compteur qui envoie les informations suivantes sur chaque course : durée, distance et lieux de prise en charge et de dépose. Un autre périphérique accepte les paiements des clients et envoie des données sur les tarifs. Afin de déterminer les tendances des usagers, la compagnie de taxis souhaite calculer le pourboire moyen par mile parcouru, en temps réel, pour chaque quartier.

## <a name="architecture"></a>Architecture

L’architecture est constituée des composants suivants.

**Sources de données**. Dans cette architecture, deux sources de données génèrent des flux de données en temps réel. Le premier flux de données contient des informations sur les courses, tandis que le second contient des informations sur les tarifs. L’architecture de référence comprend un générateur de données simulées qui lit le contenu d’un ensemble de fichiers statiques et envoie (push) les données vers Event Hubs. Les sources de données dans une application réelle seraient les appareils installés dans les taxis.

**Azure Event Hubs**. [Event Hubs](/azure/event-hubs/) est un service d’ingestion d’événements. Cette architecture utilise deux instances d’Event Hub, à savoir une par source de données. Chaque source de données envoie un flux de données à l’Event Hub associé.

**Azure Databricks**. [Databricks](/azure/azure-databricks/) est une plateforme d’analytique basée sur Apache Spark et optimisée pour la plateforme de services cloud Microsoft Azure. Databricks est utilisé pour mettre en corrélation les données de courses et de tarifs des taxis, mais aussi pour enrichir les données mises en corrélation avec les données de quartiers stockées dans le système de fichiers Databricks.

**Cosmos DB**. La sortie d’un travail Azure Databricks est une série d’enregistrements écrits dans [Cosmos DB](/azure/cosmos-db/) à l’aide de l’API Cassandra. Si l’API Cassandra est utilisée, c’est parce qu’elle prend en charge la modélisation de données de séries chronologiques.

**Azure Log Analytics**. Les données de journal d’application collectées par [Azure Monitor](/azure/monitoring-and-diagnostics/) sont stockées dans un [espace de travail Log Analytics](/azure/log-analytics). Il est possible d’utiliser des requêtes Log Analytics pour analyser et visualiser les métriques et inspecter les messages de journal pour identifier les problèmes au sein de l’application.

## <a name="data-ingestion"></a>Ingestion de données

Pour simuler une source de données, cette architecture de référence utilise les [données des taxis de la ville de New York](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) <sup>[[1]](#note1)</sup>. Ce jeu de données contient des données sur les courses de taxis à New York sur une période de quatre ans (2010 &ndash; 2013). Il contient deux types d’informations : Données de course et données de tarif. Les premières incluent la durée du trajet, la distance et les lieux de prise en charge et de dépose. Les secondes incluent le montant des tarifs des courses, des taxes et des pourboires. Les champs communs aux deux types d’enregistrement sont le numéro de médaillon (« taxi jaune »), le permis spécial et l’ID fournisseur. Ensemble ces trois champs identifient un taxi ainsi qu’un chauffeur. Les données sont stockées au format CSV.

Le générateur de données est une application .NET Core qui lit les enregistrements et les envoie à Azure Event Hubs. Le générateur envoie les données des courses au format JSON et les données relatives aux tarifs au format CSV.

Le service Event Hubs utilise des [partitions](/azure/event-hubs/event-hubs-features#partitions) pour segmenter les données. Ce système de partition permet à un consommateur de lire chaque partition en parallèle. Lorsque vous envoyez des données à Event Hubs, vous pouvez spécifier explicitement la clé de partition. Sinon, les enregistrements sont affectés aux partitions de manière alternée.

Dans ce scénario, les données de courses et de tarifs doivent se retrouver avec le même ID de partition pour un taxi donné. Cela permet à Databricks d’appliquer un degré de parallélisme quand il met en corrélation les deux flux. Un enregistrement dans la partition *n* des données des courses correspond à un enregistrement de la partition *n* des données relatives aux tarifs.

![Diagramme du traitement de flux de données avec Azure Databricks et Event Hubs](./images/stream-processing-databricks-eh.png)

Dans le générateur de données, le modèle de données commun pour les deux types d’enregistrement comprend une propriété `PartitionKey`, qui est la concaténation de `Medallion`, `HackLicense` et `VendorId`.

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

Cette propriété est utilisée pour fournir une clé de partition explicite lors de l’envoi des données vers Event Hubs :

```csharp
using (var client = pool.GetObject())
{
    return client.Value.SendAsync(new EventData(Encoding.UTF8.GetBytes(
        t.GetData(dataFormat))), t.PartitionKey);
}
```

### <a name="event-hubs"></a>Event Hubs

La capacité de débit du service Event Hubs est mesurée par les [unités de débit](/azure/event-hubs/event-hubs-features#throughput-units). Vous pouvez mettre automatiquement à l’échelle un Event Hub en activant [l’augmentation automatique](/azure/event-hubs/event-hubs-auto-inflate), qui ajuste automatiquement les unités de débit en fonction du trafic, jusqu’à la limite configurée.

## <a name="stream-processing"></a>Traitement des flux de données

Dans Azure Databricks, le traitement des données est assuré par un travail. Le travail est attribué à un cluster, qui en assure l’exécution. Le travail peut être du code personnalisé écrit en Java ou un [notebook](https://docs.databricks.com/user-guide/notebooks/index.html) Spark.

Dans cette architecture de référence, le travail est une archive Java avec des classes écrites en Java et Scala. Au moment de spécifier l’archive Java pour un travail Databricks, la classe doit être spécifiée en vue d’une exécution par le cluster Databricks. Ici, la méthode **main** de la classe **com.microsoft.pnp.TaxiCabReader** contient la logique de traitement des données.

### <a name="reading-the-stream-from-the-two-event-hub-instances"></a>Lecture du flux à partir des deux instances de hub d’événements

La logique de traitement des données utilise [Spark Structured Streaming](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html) pour lire dans les deux instances de hub d’événements Azure :

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

### <a name="enriching-the-data-with-the-neighborhood-information"></a>Enrichissement des données avec les informations de quartiers

Les données de courses comprennent les coordonnées de latitude et de longitude des lieux de prise en charge et de dépôt des clients. Même si ces coordonnées sont utiles, elles ne sont pas facilement exploitables pour l’analyse. Par conséquent, ces données sont enrichies avec les données de quartiers lues dans un [fichier de forme](https://en.wikipedia.org/wiki/Shapefile).

De format binaire, le fichier de forme ne s’analyse pas facilement, mais la bibliothèque [GeoTools](http://geotools.org/) propose des outils pour les données géospatiales qui utilisent le format de fichier de forme. Cette bibliothèque est utilisée dans la classe **com.microsoft.pnp.GeoFinder** pour déterminer le nom du quartier en fonction des coordonnées de prise en charge et de dépôt.

```scala
val neighborhoodFinder = (lon: Double, lat: Double) => {
      NeighborhoodFinder.getNeighborhood(lon, lat).get()
    }
```

### <a name="joining-the-ride-and-fare-data"></a>Jonction des données de courses et de tarifs

Dans un premier temps, les données de courses et de tarifs sont transformées :

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

Les données de courses sont ensuite jointes aux données de tarifs :

```scala
val mergedTaxiTrip = rides.join(fares, Seq("medallion", "hackLicense", "vendorId", "pickupTime"))
```

### <a name="processing-the-data-and-inserting-into-cosmos-db"></a>Traitement des données et insertion dans Cosmos DB

Le montant du tarif moyen pour chaque quartier est calculé pour un intervalle de temps donné :

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

Il est ensuite inséré dans Cosmos DB :

```scala
maxAvgFarePerNeighborhood
      .writeStream
      .queryName("maxAvgFarePerNeighborhood_cassandra_insert")
      .outputMode(OutputMode.Append())
      .foreach(new CassandraSinkForeach(connector))
      .start()
      .awaitTermination()
```

## <a name="security-considerations"></a>Considérations relatives à la sécurité

L’accès à l’espace de travail Azure Database est contrôlé à l’aide de la [console administrateur](https://docs.databricks.com/administration-guide/admin-settings/index.html). La console administrateur comprend une fonctionnalité qui permet d’ajouter des utilisateurs, de gérer les autorisations utilisateur et de configurer l’authentification unique. Le contrôle d’accès pour les espaces de travail, les clusters, les travaux et les tables peut aussi être défini via la console administrateur.

### <a name="managing-secrets"></a>Gestion des secrets

Azure Databricks comprend un [magasin des secrets](https://docs.azuredatabricks.net/user-guide/secrets/index.html) dans lequel sont stockés les secrets, notamment les chaînes de connexion, les clés d’accès, les noms d’utilisateur et les mots de passe. Les secrets contenus dans le magasin des secrets Azure Databricks sont partitionnés par **étendues** (scope) :

```bash
databricks secrets create-scope --scope "azure-databricks-job"
```

Les secrets sont ajoutés au niveau de l’étendue :

```bash
databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
```

> [!NOTE]
> Il est possible d’utiliser une étendue adossée à Azure Key Vault à la place de l’étendue Azure Databricks native. Pour en savoir plus, consultez la page relative aux [étendues adossées à Azure Key Vault](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes).

Dans le code, les secrets sont accessibles via les [utilitaires de secrets](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities) Azure Databricks.

## <a name="monitoring-considerations"></a>Surveillance - Éléments à prendre en compte

Azure Databricks est basé sur Apache Spark et tous deux utilisent [log4j](https://logging.apache.org/log4j/2.x/) comme bibliothèque standard pour la journalisation. En plus de la journalisation par défaut fournie par Apache Spark, cette architecture de référence envoie des journaux et des métriques à [Azure Log Analytics](/azure/log-analytics/).

La classe **com.microsoft.pnp.TaxiCabReader** configure le système de journalisation Apache Spark pour envoyer ses journaux à Azure Log Analytics en utilisant les valeurs contenues dans le fichier **log4j.properties**. Si les messages du journaliseur Apache Spark sont des chaînes, Azure Log Analytics nécessite de son côté des messages de journal au format JSON. La classe **com.microsoft.pnp.log4j.LogAnalyticsAppender** transforme ces messages au format JSON :

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

Comme la classe **com.microsoft.pnp.TaxiCabReader** traite les messages relatifs aux courses et aux tarifs, il est possible que l’un des deux soit mal formé et donc non valide. Dans un environnement de production, il est important d’analyser ces messages mal formés pour identifier un problème au niveau des sources de données et le résoudre rapidement pour éviter toute perte de données. La classe **com.microsoft.pnp.TaxiCabReader** inscrit un accumulateur Apache Spark qui assure le suivi du nombre d’enregistrements relatifs aux courses et aux tarifs mal formés :

```scala
    @transient val appMetrics = new AppMetrics(spark.sparkContext)
    appMetrics.registerGauge("metrics.malformedrides", AppAccumulators.getRideInstance(spark.sparkContext))
    appMetrics.registerGauge("metrics.malformedfares", AppAccumulators.getFareInstance(spark.sparkContext))
    SparkEnv.get.metricsSystem.registerSource(appMetrics)
```

Apache Spark utilise la bibliothèque Dropwizard pour envoyer des métriques, et certains champs de métriques Dropwizard natifs sont incompatibles avec Azure Log Analytics. Par conséquent, cette architecture de référence comprend un récepteur et un rapporteur Dropwizard personnalisés. Elle met en forme les métriques dans le format attendu par Azure Log Analytics. Quand Apache Spark transmet des métriques, les métriques personnalisées pour les données de course et de tarif malformées sont aussi envoyées.

La dernière métrique à être journalisée dans l’espace de travail Azure Log Analytics est la progression cumulative du travail Spark Structured Streaming. Cette opération s’effectue via un écouteur StreamingQuery personnalisé implémenté dans la classe **com.microsoft.pnp.StreamingMetricsListener**. Cette classe est inscrite dans la session Apache Spark au moment où le travail s’exécute :

```scala
spark.streams.addListener(new StreamingMetricsListener())
```

Les méthodes contenues dans StreamingMetricsListener sont appelées par le runtime Apache Spark chaque fois qu’un événement Structured Streaming se produit ; les messages de journal et les métriques sont alors envoyés à l’espace de travail Azure Log Analytics. Vous pouvez utiliser les requêtes suivantes dans votre espace de travail pour superviser l’application :

### <a name="latency-and-throughput-for-streaming-queries"></a>Latence et débit pour les requêtes de streaming

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| project  mdc_inputRowsPerSecond_d, mdc_durationms_triggerExecution_d
| render timechart
```

### <a name="exceptions-logged-during-stream-query-execution"></a>Exceptions journalisées pendant l’exécution de requête de flux

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| where Level contains "Error"
```

### <a name="accumulation-of-malformed-fare-and-ride-data"></a>Accumulation de données de course et de tarif mal formées

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

### <a name="job-execution-to-trace-resiliency"></a>Exécution du travail pour tracer la résilience

```shell
SparkMetric_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart
| where name_s contains "driver.DAGScheduler.job.allJobs"
```

## <a name="deploy-the-solution"></a>Déployer la solution

Un déploiement de cette architecture de référence est disponible sur [GitHub](https://github.com/mspnp/azure-databricks-streaming-analytics).

### <a name="prerequisites"></a>Prérequis

1. Clonez, dupliquez (fork) ou téléchargez le dépôt GitHub de [traitement de flux avec Azure Databricks](https://github.com/mspnp/azure-databricks-streaming-analytics).

2. Installez [Docker](https://www.docker.com/) pour exécuter le générateur de données.

3. Installez [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).

4. Installez l’interface [CLI Databricks](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html).

5. À partir d’une invite de commandes, d’une invite bash ou de l’invite de commandes PowerShell, connectez-vous à votre compte Azure, comme suit :
    ```shell
    az login
    ```
6. Installez un IDE Java, avec les ressources suivantes :
    - JDK 1.8
    - SDK Scala 2.11
    - Maven 3.5.4

### <a name="download-the-new-york-city-taxi-and-neighborhood-data-files"></a>Télécharger les fichiers de données sur les taxis et les quartiers de New York

1. Créez un répertoire nommé `DataFile` à la racine du dépôt GitHub cloné dans votre système de fichiers local.

2. Ouvrez un navigateur web et accédez à https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935.

3. Cliquez sur le bouton **Télécharger** de cette page pour télécharger un fichier zip de toutes les données portant sur les taxis pour cette année.

4. Extrayez le fichier zip dans le répertoire `DataFile`.

    > [!NOTE]
    > Ce fichier zip contient d’autres fichiers zip. N’extrayez pas les fichiers zip enfants.

    La structure des répertoires doit se présenter comme ceci :

    ```shell
    /DataFile
        /FOIL2013
            trip_data_1.zip
            trip_data_2.zip
            trip_data_3.zip
            ...
    ```

5. Ouvrez un navigateur web et accédez à https://www.zillow.com/howto/api/neighborhood-boundaries.htm.

6. Cliquez sur **New York Neighborhood Boundaries** pour télécharger le fichier.

7. Copiez le fichier **ZillowNeighborhoods-NY.zip** du répertoire de **téléchargements** de votre navigateur vers le répertoire `DataFile`.

### <a name="deploy-the-azure-resources"></a>Déployer les ressources Azure

1. À partir d’une invite de commandes Windows ou d’un interpréteur de commandes, exécutez la commande suivante et suivez l’invite de connexion :

    ```bash
    az login
    ```

2. Accédez au dossier nommé `azure` dans le dépôt GitHub :

    ```bash
    cd azure
    ```

3. Exécutez les commandes suivantes pour déployer les ressources Azure :

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
        --template-file deployresources.json --parameters \
        eventHubNamespace=$eventHubNamespace \
        databricksWorkspaceName=$databricksWorkspaceName \
        cosmosDatabaseAccount=$cosmosDatabaseAccount \
        logAnalyticsWorkspaceName=$logAnalyticsWorkspaceName \
        logAnalyticsWorkspaceRegion=$logAnalyticsWorkspaceRegion
    ```

4. La sortie du déploiement est écrite dans la console à l’issue de l’opération. Recherchez dans la sortie le code JSON suivant :

```json
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

Ces valeurs correspondent aux secrets qui seront ajoutés aux secrets Databricks dans les prochaines sections. Gardez-les en lieu sûr en attendant de les ajouter dans ces sections.

### <a name="add-a-cassandra-table-to-the-cosmos-db-account"></a>Ajouter une table Cassandra au compte Cosmos DB

1. Sur le portail Azure, accédez au groupe de ressources créé dans la section **Déployer les ressources Azure** ci-dessus. Cliquez sur **Compte Azure Cosmos DB**. Créez une table avec l’API Cassandra.

2. Dans le panneau **overview** (vue d’ensemble), cliquez sur **add table** (ajouter une table).

3. Quand le panneau **add table** s’ouvre, entrez `newyorktaxi` dans la zone de texte **Keyspace name** (Nom de l’espace de clés).

4. Dans la section **enter CQL command to create the table** (entrer la commande CQL pour créer la table), entrez `neighborhoodstats` dans la zone de texte en regard de `newyorktaxi`.

5. Dans la zone de texte ci-dessous, entrez ce qui suit :
    ```shell
    (neighborhood text, window_end timestamp, number_of_rides bigint,total_fare_amount double, primary key(neighborhood, window_end))
    ```
6. Dans la zone de texte **Throughput (1,000 - 1,000,000 RU/s)** (Débit (1 000 - 1 000 000 de RU/s)), entrez la valeur `4000`.

7. Cliquez sur **OK**.

### <a name="add-the-databricks-secrets-using-the-databricks-cli"></a>Ajouter les secrets Databricks à l’aide de l’interface CLI Databricks

Pour commencer, entrez les secrets pour EventHub :

1. À partir de l’interface **CLI Azure Databricks** installée à l’étape 2 des prérequis, créez l’étendue des secrets Azure Databricks :
    ```shell
    databricks secrets create-scope --scope "azure-databricks-job"
    ```
2. Ajoutez le secret pour le hub d’événements des courses de taxis (taxi-ride) :
    ```shell
    databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
    ```
    Une fois exécutée, cette commande ouvre l’éditeur vi. Entrez la valeur de **taxi-tour-eh** de la section de sortie **eventHubs** de l’étape 4 de la section *Déployer les ressources Azure*. Enregistrez et quittez vi.

3. Ajoutez le secret pour le hub d’événements des tarifs de taxi (taxi-fare) :
    ```shell
    databricks secrets put --scope "azure-databricks-job" --key "taxi-fare"
    ```
    Une fois exécutée, cette commande ouvre l’éditeur vi. Entrez la valeur de **taxi-fare-eh** de la section de sortie **eventHubs** de l’étape 4 de la section *Déployer les ressources Azure*. Enregistrez et quittez vi.

Ensuite, entrez les secrets pour Cosmos DB :

1. Ouvrez le portail Azure, puis accédez au groupe de ressources spécifié à l’étape 3 de la section **Déployer les ressources Azure**. Cliquez sur le compte Azure Cosmos DB.

2. À partir de l’interface **CLI Azure Databricks**, ajoutez le secret pour le nom d’utilisateur Cosmos DB :
    ```shell
    databricks secrets put --scope azure-databricks-job --key "cassandra-username"
    ```
Une fois exécutée, cette commande ouvre l’éditeur vi. Entrez la valeur de **username** de la section de sortie **CosmosDb** de l’étape 4 de la section *Déployer les ressources Azure*. Enregistrez et quittez vi.

3. Ensuite, ajoutez le secret pour le mot de passe Cosmos DB :
    ```shell
    databricks secrets put --scope azure-databricks-job --key "cassandra-password"
    ```

Une fois exécutée, cette commande ouvre l’éditeur vi. Entrez la valeur de **secret** de la section de sortie **CosmosDb** de l’étape 4 de la section *Déployer les ressources Azure*. Enregistrez et quittez vi.

> [!NOTE]
> Si vous utilisez une [étendue de secrets adossée à Azure Key Vault](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes), elle doit se nommer **azure-databricks-job** et les secrets doivent avoir exactement les mêmes noms que ceux mentionnés ci-dessus.

### <a name="add-the-zillow-neighborhoods-data-file-to-the-databricks-file-system"></a>Ajouter le fichier de données Zillow Neighborhoods au système de fichiers Databricks

1. Créez un répertoire dans le système de fichiers Databricks :
    ```bash
    dbfs mkdirs dbfs:/azure-databricks-jobs
    ```

2. Accédez au répertoire `DataFile` et entrez les informations suivantes :
    ```bash
    dbfs cp ZillowNeighborhoods-NY.zip dbfs:/azure-databricks-jobs
    ```

### <a name="add-the-azure-log-analytics-workspace-id-and-primary-key-to-configuration-files"></a>Ajouter l’ID d’espace de travail Azure Log Analytics et la clé primaire aux fichiers de configuration

Pour cette section, vous avez besoin de l’ID et de la clé primaire de l’espace de travail Log Analytics. L’ID de l’espace de travail est la valeur de **workspaceId** dans la section de sortie **logAnalytics** de l’étape 4 de la section *Déployer les ressources Azure*. La clé primaire est le **secret** de la section de sortie.

1. Pour configurer la journalisation log4j, ouvrez `\azure\AzureDataBricksJob\src\main\resources\com\microsoft\pnp\azuredatabricksjob\log4j.properties`. Modifiez les deux valeurs suivantes :
    ```shell
    log4j.appender.A1.workspaceId=<Log Analytics workspace ID>
    log4j.appender.A1.secret=<Log Analytics primary key>
    ```

2. Pour configurer une journalisation personnalisée, ouvrez `\azure\azure-databricks-monitoring\scripts\metrics.properties`. Modifiez les deux valeurs suivantes :
    ```shell
    *.sink.loganalytics.workspaceId=<Log Analytics workspace ID>
    *.sink.loganalytics.secret=<Log Analytics primary key>
    ```

### <a name="build-the-jar-files-for-the-databricks-job-and-databricks-monitoring"></a>Générer les fichiers .jar pour le travail Databricks et la supervision Databricks

1. Utilisez votre IDE Java pour importer le fichier projet Maven nommé **pom.xml** qui se trouve dans le répertoire racine.

2. Optez pour une build propre. La sortie de cette build correspond aux fichiers nommés **azure-databricks-job-1.0-SNAPSHOT.jar** et **azure-databricks-monitoring-0.9.jar**.

### <a name="configure-custom-logging-for-the-databricks-job"></a>Configurer la journalisation personnalisée pour le travail Databricks

1. Copiez le fichier **azure-databricks-surveillance-0.9.jar** dans le système de fichiers Databricks en entrant la commande suivante dans l’interface **CLI Databricks** :
    ```shell
    databricks fs cp --overwrite azure-databricks-monitoring-0.9.jar dbfs:/azure-databricks-job/azure-databricks-monitoring-0.9.jar
    ```

2. Copiez les propriétés de la journalisation personnalisée depuis `\azure\azure-databricks-monitoring\scripts\metrics.properties` vers le système de fichiers Databricks en entrant la commande suivante :
    ```shell
    databricks fs cp --overwrite metrics.properties dbfs:/azure-databricks-job/metrics.properties
    ```

3. À ce stade, vous n’avez pas encore nommé votre cluster Databricks ; attribuez-lui un nom maintenant. Vous devez entrer le nom ci-dessous dans le chemin du système de fichiers Databricks de votre cluster. Copiez le script d’initialisation depuis `\azure\azure-databricks-monitoring\scripts\spark.metrics` vers le système de fichiers Databricks en entrant la commande suivante :
    ```shell
    databricks fs cp --overwrite spark-metrics.sh dbfs:/databricks/init/<cluster-name>/spark-metrics.sh
    ```

### <a name="create-a-databricks-cluster"></a>Créer un cluster Databricks

1. Dans l’espace de travail Databricks, cliquez sur Clusters, puis sur create cluster. Entrez le nom du cluster que vous avez créé à l’étape 3 de la section **Configurer la journalisation personnalisée pour le travail Databricks** ci-dessus.

2. Sélectionnez un mode de cluster **standard**.

3. Définissez **Databricks runtime version** (Version du runtime Databricks) sur **4.3 (comprend Apache Spark 2.3.1, Scala 2.11)**

4. Définissez **Python version** (Version de Python) sur **2**.

5. Définissez **Driver Type** (Type de pilote) sur **Same as worker** (Identique au Worker).

6. Définissez **Worker Type** (Type de Worker) sur **Standard_DS3_v2**.

7. Définissez **Min Workers** (Nombre minimum de Workers) sur **2**.

8. Désélectionnez **Enable autoscaling** (Activer la mise à l’échelle automatique).

9. En dessous de la boîte de dialogue **Auto Termination** (Arrêt automatique), cliquez sur **Init Scripts** (Scripts d’initialisation).

10. Entrez **dbfs:/databricks/init/<cluster-name>/spark-metrics.sh** en remplaçant <cluster-name> par le nom de cluster créé à l’étape 1.

11. Cliquez sur le bouton **Add** .

12. Cliquez sur le bouton **Create Cluster**.

### <a name="create-a-databricks-job"></a>Créer un travail Databricks

1. Dans l’espace de travail Databricks, cliquez sur Jobs et sur create job.

2. Entrez un nom de travail.

3. Cliquez sur « set jar » pour accéder à la boîte de dialogue Upload JAR to Run (Charger le JAR à exécuter).

4. Faites glisser le fichier **azure-databricks-job-1.0-SNAPSHOT.jar** que vous avez créé dans la section **Générer le fichier .jar pour le travail Databricks** vers la zone **Drop JAR here to upload** (Déposer le JAR à charger ici).

5. Entrez **com.microsoft.pnp.TaxiCabReader** dans le champ **Main Class** (Classe principale).

6. Dans le champ arguments, entrez ce qui suit :
    ```shell
    -n jar:file:/dbfs/azure-databricks-jobs/ZillowNeighborhoods-NY.zip!/ZillowNeighborhoods-NY.shp --taxi-ride-consumer-group taxi-ride-eh-cg --taxi-fare-consumer-group taxi-fare-eh-cg --window-interval "1 minute" --cassandra-host <Cosmos DB Cassandra host name from above>
    ```

7. Installez les bibliothèques dépendantes en suivant ces étapes :

    1. Dans l’interface utilisateur Databricks, cliquez sur le bouton **home** (accueil).

    2. Dans la liste déroulante **Users** (Utilisateurs), cliquez sur le nom de votre compte d’utilisateur pour ouvrir les paramètres d’espace de travail de votre compte.

    3. Cliquez sur la flèche déroulante en regard du nom de votre compte, cliquez sur **create**, puis sur **Library** (Bibliothèque) pour ouvrir la boîte de dialogue **New Library** (Nouvelle bibliothèque).

    4. Dans la liste déroulante **Source**, sélectionnez **Maven Coordinate** (Coordonnée Maven).

    5. Sous l’en-tête **Install Maven Artifacts** (Installer les artefacts Maven), entrez `com.microsoft.azure:azure-eventhubs-spark_2.11:2.3.5` dans la zone de texte **Coordinate**.

    6. Cliquez sur **Create Library** (Créer une bibliothèque) pour ouvrir la fenêtre **Artifacts**.

    7. Sous **Status on running clusters** (État sur les clusters en cours d’exécution), cochez la case **Attach automatically to all clusters** (Joindre automatiquement à tous les clusters).

    8. Répétez les étapes 1 à 7 pour la coordonnée Maven `com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.0.0`.

    9. Répétez les étapes 1 à 6 pour la coordonnée Maven `org.geotools:gt-shapefile:19.2`.

    10. Cliquez sur **Advanced Options** (Options avancées).

    11. Entrez `http://download.osgeo.org/webdav/geotools/` dans la zone de texte **Repository** (Référentiel).

    12. Cliquez sur **Create Library** (Créer une bibliothèque) pour ouvrir la fenêtre **Artifacts**. 

    13. Sous **Status on running clusters** (État sur les clusters en cours d’exécution), cochez la case **Attach automatically to all clusters** (Joindre automatiquement à tous les clusters).

8. Ajoutez les bibliothèques dépendantes ajoutées à l’étape 7 au travail créé à la fin de l’étape 6 :

    1. Dans l’espace de travail Azure Databricks, cliquez sur **Jobs** (Travaux).

    2. Cliquez sur le nom du travail créé à l’étape 2 de la section **Créer un travail Databricks**.

    3. En regard de la section **Dependent Libraries** (Bibliothèques dépendantes), cliquez sur **Add** pour ouvrir la boîte de dialogue **Add Dependent Library** (Ajouter une bibliothèque dépendante).

    4. Sous **Library From** (Bibliothèque de), sélectionnez **Workspace** (Espace de travail).

    5. Cliquez successivement sur **users** (utilisateurs), sur votre nom d’utilisateur, puis sur `azure-eventhubs-spark_2.11:2.3.5`.

    6. Cliquez sur **OK**.

    7. Répétez les étapes 1 à 6 pour `spark-cassandra-connector_2.11:2.3.1` et `gt-shapefile:19.2`.

9. En regard de **Cluster:**, cliquez sur **Edit** (Modifier). La boîte de dialogue **Configurer Cluster** (Configurer le cluster) s’ouvre. Dans la liste déroulante **Cluster Type** (Type de cluster), sélectionnez **Existing Cluster** (Cluster existant). Dans la liste déroulante **Select Cluster**, sélectionnez le cluster créé dans la section **Créer un cluster Databricks**. Cliquez sur **confirm**.

10. Cliquez sur **run now** (exécuter maintenant).

### <a name="run-the-data-generator"></a>Exécuter le générateur de données

1. Accédez au répertoire nommé `onprem` dans le dépôt GitHub.

2. Mettez à jour les valeurs du fichier **main.env** comme suit :

    ```shell
    RIDE_EVENT_HUB=[Connection string for the taxi-ride event hub]
    FARE_EVENT_HUB=[Connection string for the taxi-fare event hub]
    RIDE_DATA_FILE_PATH=/DataFile/FOIL2013
    MINUTES_TO_LEAD=0
    PUSH_RIDE_DATA_FIRST=false
    ```
    La chaîne de connexion pour le hub d’événements taxi-ride est la valeur de **taxi-ride-eh** dans la section de sortie **eventHubs** de l’étape 4 de la section *Déployer les ressources Azure*. La chaîne de connexion pour le hub d’événements taxi-fare est la valeur de **taxi-fare-eh** dans la section de sortie **eventHubs** de l’étape 4 de la section *Déployer les ressources Azure*.

3. Pour générer l’image Docker, exécutez la commande suivante :

    ```bash
    docker build --no-cache -t dataloader .
    ```

4. Revenez au répertoire parent.

    ```bash
    cd ..
    ```

5. Pour exécuter l’image Docker, exécutez la commande suivante :

    ```bash
    docker run -v `pwd`/DataFile:/DataFile --env-file=onprem/main.env dataloader:latest
    ```

La sortie doit se présenter comme suit :

```
Created 10000 records for TaxiFare
Created 10000 records for TaxiRide
Created 20000 records for TaxiFare
Created 20000 records for TaxiRide
Created 30000 records for TaxiFare
...
```

Pour vérifier que le travail Databricks s’exécute correctement, ouvrez le portail Azure et accédez à la base de données Cosmos DB. Ouvrez le panneau **Explorateur de données** et examinez les données dans la table **taxi records** (enregistrements taxi). 

[1] <span id="note1">Donovan, Brian; Work, Dan (2016) : Données de trajet des taxis de New York (2010-2013). Université de l’Illinois, Urbana-Champaign. https://doi.org/10.13012/J8PN93H8
