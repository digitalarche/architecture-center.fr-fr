---
title: Traitement de flux de données avec Azure Stream Analytics
titleSuffix: Azure Reference Architectures
description: Créez un pipeline de traitement de flux de données de bout en bout dans Azure.
author: MikeWasson
ms.date: 11/06/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18
ms.openlocfilehash: e891c0fe490f57e0723b1ec133e33e70d2e23554
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640411"
---
# <a name="create-a-stream-processing-pipeline-with-azure-stream-analytics"></a>Créer un pipeline de traitement de flux avec Azure Stream Analytics

Cette architecture de référence présente un pipeline de [traitement de flux](/azure/architecture/data-guide/big-data/real-time-processing) de bout en bout. Le pipeline ingère les données provenant des deux sources, met en corrélation les enregistrements dans les deux flux, puis calcule une moyenne mobile dans une fenêtre de temps. Les résultats sont stockés en vue d’une analyse plus approfondie.

Une implémentation de référence pour cette architecture est disponible sur [GitHub][github].

![Architecture de référence pour la création d’un pipeline de traitement de flux de données avec Azure Stream Analytics](./images/stream-processing-asa/stream-processing-asa.png)

**Scénario** : Une compagnie de taxis collecte des données sur chaque trajet effectué en taxi. Pour ce scénario, nous partons du principe que deux périphériques distincts envoient des données. Le taxi est équipé d’un compteur qui envoie les informations suivantes sur chaque course : durée, distance et lieux de prise en charge et de dépose. Un autre périphérique accepte les paiements des clients et envoie des données sur les tarifs. La compagnie de taxis souhaite calculer le pourboire moyen par mile (1,6 km) parcouru, en temps réel, afin de déterminer les tendances.

## <a name="architecture"></a>Architecture

L’architecture est constituée des composants suivants.

**Sources de données**. Dans cette architecture, deux sources de données génèrent des flux de données en temps réel. Le premier flux de données contient des informations sur les courses, tandis que le second contient des informations sur les tarifs. L’architecture de référence comprend un générateur de données simulées qui lit le contenu d’un ensemble de fichiers statiques et envoie (push) les données vers Event Hubs. Dans une application réelle, les sources de données seraient des périphériques installés dans les taxis.

**Azure Event Hubs**. [Event Hubs](/azure/event-hubs/) est un service d’ingestion d’événements. Cette architecture utilise deux instances d’Event Hub, à savoir une par source de données. Chaque source de données envoie un flux de données à l’Event Hub associé.

**Azure Stream Analytics**. [Stream Analytics](/azure/stream-analytics/) est un moteur de traitement des événements. Un travail Stream Analytics lit les flux de données provenant des deux instances d’Event Hub et effectue le traitement des flux de données.

**Cosmos DB**. La sortie du travail Stream Analytics est une série d’enregistrements, qui sont écrits sous forme de documents JSON dans une base de données de documents Cosmos DB.

**Microsoft Power BI**. Power BI est une suite d’outils d’analyse métier pour analyser les données et obtenir des informations métier. Dans cette architecture, elle charge les données de Cosmos DB. Les utilisateurs sont ainsi en mesure d’analyser l’ensemble complet des données historiques qui ont été collectées. Vous pouvez également transmettre en continu les résultats directement de Stream Analytics à Power BI pour disposer d’une vue en temps réel des données. Pour plus d’informations, voir [Streaming en temps réel dans Power BI](/power-bi/service-real-time-streaming).

**Azure Monitor**. [Azure Monitor](/azure/monitoring-and-diagnostics/) collecte les mesures de performances sur les services Azure déployés dans la solution. En les visualisant dans un tableau de bord, vous pouvez obtenir des informations sur l’intégrité de la solution.

## <a name="data-ingestion"></a>Ingestion de données

<!-- markdownlint-disable MD033 -->

Pour simuler une source de données, cette architecture de référence utilise les [données des taxis de la ville de New York](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) <sup>[[1]](#note1)</sup>. Ce jeu de données contient des données sur les courses de taxi effectuées à New York sur une période de 4 ans (2010 à 2013). Il contient deux types d’informations : Données de course et données de tarif. Les premières incluent la durée du trajet, la distance et les lieux de prise en charge et de dépose. Les secondes incluent le montant des tarifs des courses, des taxes et des pourboires. Les champs communs aux deux types d’enregistrement sont le numéro de médaillon (« taxi jaune »), le permis spécial et l’ID fournisseur. Ensemble ces trois champs identifient un taxi ainsi qu’un chauffeur. Les données sont stockées au format CSV.

<span id="note1">[1]</span> Donovan, Brian; Work, Dan (2016) : Données de trajet des taxis de New York (2010-2013). Université de l’Illinois, Urbana-Champaign. https://doi.org/10.13012/J8PN93H8

<!-- markdownlint-enable MD033 -->

Le générateur de données est une application .NET Core qui lit les enregistrements et les envoie à Azure Event Hubs. Le générateur envoie les données des courses au format JSON et les données relatives aux tarifs au format CSV.

Le service Event Hubs utilise des [partitions](/azure/event-hubs/event-hubs-features#partitions) pour segmenter les données. Ce système de partition permet à un consommateur de lire chaque partition en parallèle. Lorsque vous envoyez des données à Event Hubs, vous pouvez spécifier explicitement la clé de partition. Sinon, les enregistrements sont affectés aux partitions de manière alternée.

Dans ce scénario particulier, les données relatives aux courses et aux tarifs doivent se retrouver avec le même ID de partition pour un taxi donné. Cela permet à Stream Analytics d’appliquer un degré de parallélisme lorsqu’il met en corrélation les deux flux de données. Un enregistrement dans la partition *n* des données des courses correspond à un enregistrement de la partition *n* des données relatives aux tarifs.

![Diagramme du traitement de flux de données avec Azure Stream Analytics et Event Hubs](./images/stream-processing-asa/stream-processing-eh.png)

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

## <a name="stream-processing"></a>Traitement des flux de données

Le travail de traitement des flux de données est défini à l’aide d’une requête SQL comprenant plusieurs étapes distinctes. Les deux premières étapes sélectionnent simplement les enregistrements dans les deux flux d’entrée.

```sql
WITH
Step1 AS (
    SELECT PartitionId,
           TRY_CAST(Medallion AS nvarchar(max)) AS Medallion,
           TRY_CAST(HackLicense AS nvarchar(max)) AS HackLicense,
           VendorId,
           TRY_CAST(PickupTime AS datetime) AS PickupTime,
           TripDistanceInMiles
    FROM [TaxiRide] PARTITION BY PartitionId
),
Step2 AS (
    SELECT PartitionId,
           medallion AS Medallion,
           hack_license AS HackLicense,
           vendor_id AS VendorId,
           TRY_CAST(pickup_datetime AS datetime) AS PickupTime,
           tip_amount AS TipAmount
    FROM [TaxiFare] PARTITION BY PartitionId
),
```

L’étape suivante joint les deux flux d’entrée pour sélectionner des enregistrements correspondants dans chaque flux.

```sql
Step3 AS (
  SELECT
         tr.Medallion,
         tr.HackLicense,
         tr.VendorId,
         tr.PickupTime,
         tr.TripDistanceInMiles,
         tf.TipAmount
    FROM [Step1] tr
    PARTITION BY PartitionId
    JOIN [Step2] tf PARTITION BY PartitionId
      ON tr.Medallion = tf.Medallion
     AND tr.HackLicense = tf.HackLicense
     AND tr.VendorId = tf.VendorId
     AND tr.PickupTime = tf.PickupTime
     AND tr.PartitionId = tf.PartitionId
     AND DATEDIFF(minute, tr, tf) BETWEEN 0 AND 15
)
```

Cette requête joint des enregistrements dans un ensemble de champs qui identifient de manière unique les enregistrements correspondants (Medallion, HackLicense, VendorId et PickupTime). L’instruction `JOIN` inclut également l’ID de partition. Comme mentionné, ce procédé tire parti du fait que les enregistrements correspondants ont toujours le même ID de partition dans ce scénario.

Dans Stream Analytics, les jointures sont *temporelles*, ce qui signifie que les enregistrements sont joints au sein d’une fenêtre de temps donnée. Dans le cas contraire, le travail devrait peut-être attendre indéfiniment une correspondance. La fonction [DATEDIFF](https://msdn.microsoft.com/azure/stream-analytics/reference/join-azure-stream-analytics) spécifie la durée pouvant séparer deux enregistrements correspondants.

La dernière étape du travail calcule le pourboire moyen par mile, groupé par fenêtre récurrente de 5 minutes.

```sql
SELECT System.Timestamp AS WindowTime,
       SUM(tr.TipAmount) / SUM(tr.TripDistanceInMiles) AS AverageTipPerMile
  INTO [TaxiDrain]
  FROM [Step3] tr
  GROUP BY HoppingWindow(Duration(minute, 5), Hop(minute, 1))
```

Stream Analytics fournit plusieurs [fonctions de fenêtrage](/azure/stream-analytics/stream-analytics-window-functions). Une fenêtre récurrente avance dans le temps en fonction d’une période fixe, dans ce cas 1 minute par saut. Le résultat revient à calculer une moyenne mobile sur les 5 dernières minutes.

Dans l’architecture illustrée ici, seuls les résultats du travail Stream Analytics sont enregistrés dans Cosmos DB. Pour un scénario de type Big Data, pensez également à utiliser [Event Hubs Capture](/azure/event-hubs/event-hubs-capture-overview) pour enregistrer les données d’événement brutes dans le stockage Blob Azure. En conservant les données brutes, vous pourrez exécuter des requêtes par lot sur vos données historiques ultérieurement afin de dériver les nouvelles informations à partir des données.

## <a name="scalability-considerations"></a>Considérations relatives à l’extensibilité

### <a name="event-hubs"></a>Event Hubs

La capacité de débit du service Event Hubs est mesurée par les [unités de débit](/azure/event-hubs/event-hubs-features#throughput-units). Vous pouvez mettre automatiquement à l’échelle un Event Hub en activant [l’augmentation automatique](/azure/event-hubs/event-hubs-auto-inflate), qui ajuste automatiquement les unités de débit en fonction du trafic, jusqu’à la limite configurée.

### <a name="stream-analytics"></a>Stream Analytics

Pour Stream Analytics, les ressources de calcul allouées à un travail sont mesurées en unités de streaming. La mise à l’échelle des travaux Stream Analytics est améliorée si ceux-ci peuvent être parallélisés. De cette façon, Stream Analytics peut distribuer le travail entre plusieurs nœuds de calcul.

Pour l’entrée du service Event Hubs, utilisez le mot clé `PARTITION BY` pour partitionner le travail Stream Analytics. Les données sont divisées en sous-ensembles basés sur les partitions du service Event Hubs.

Les fonctions de fenêtrage et les jointures temporelles requièrent des unités de streaming supplémentaires. Si possible, utilisez `PARTITION BY` afin que chaque partition soit traitée séparément. Pour plus d’informations, voir [Comprendre et ajuster les unités de streaming](/azure/stream-analytics/stream-analytics-streaming-unit-consumption#windowed-aggregates).

S’il n’est pas possible de paralléliser l’intégralité du travail Stream Analytics, essayez de le fractionner en plusieurs étapes, en commençant par une ou plusieurs étapes parallèles. De cette façon, les premières étapes peuvent s’exécuter en parallèle. Par exemple, dans cette architecture de référence :

- Les étapes 1 et 2 sont des instructions `SELECT` simples qui sélectionnent des enregistrements dans une seule et même partition.
- L’étape 3 effectue une jointure partitionnée entre deux flux d’entrée. Cette étape tire parti du fait que les enregistrements correspondants partagent la même clé de partition et qu’ils sont donc assurés de disposer du même ID de partition dans chaque flux d’entrée.
- L’étape 4 est agrégée dans toutes les partitions. Elle ne peut pas être parallélisée.

Consultez le [diagramme de travail](/azure/stream-analytics/stream-analytics-job-diagram-with-metrics) Stream Analytics pour voir le nombre de partitions affectées à chaque étape du travail. Le diagramme de travail suivant illustre cette architecture de référence :

![Diagramme de travail](./images/stream-processing-asa/job-diagram.png)

### <a name="cosmos-db"></a>Cosmos DB

La capacité de débit pour Cosmos DB est mesurée en [unités de requête](/azure/cosmos-db/request-units) (RU). Pour mettre à l’échelle un conteneur Cosmos DB au-delà de 10 000 RU, vous devez spécifier une [clé de partition](/azure/cosmos-db/partition-data) lorsque vous créez le conteneur, puis inclure la clé de partition dans chaque document.

Dans cette architecture de référence, les nouveaux documents sont créés une seule fois par minute (intervalle de fenêtre récurrente). Les besoins en débit sont donc assez faibles. C’est pour cette raison qu’il est inutile d’affecter une clé de partition dans ce scénario.

## <a name="monitoring-considerations"></a>Surveillance - Éléments à prendre en compte

Avec des solutions de traitement des flux de données, il est important de surveiller les performances et l’intégrité du système. [Azure Monitor](/azure/monitoring-and-diagnostics/) collecte des journaux d’activité de métriques et de diagnostics pour les services Azure utilisés dans l’architecture. Azure Monitor est intégré à la plateforme Azure et ne nécessite pas de code supplémentaire dans votre application.

Signaux d’avertissement indiquant que vous devez faire évoluer la ressource Azure appropriée :

- Le service Event Hubs limite les requêtes ou est proche du quota quotidien des messages.
- Le travail Stream Analytics utilise toujours plus de 80 % des unités de streaming allouées.
- Cosmos DB commence à limiter les requêtes.

L’architecture de référence inclut un tableau de bord personnalisé, qui est déployé sur le portail Azure. Après avoir déployé l’architecture, vous pouvez afficher le tableau de bord en ouvrant le [portail Azure](https://portal.azure.com) et en sélectionnant `TaxiRidesDashboard` dans la liste des tableaux de bord. Pour plus d’informations sur la création et le déploiement de tableaux de bord personnalisés dans le portail Azure, voir [Créer par programmation des tableaux de bord Azure](/azure/azure-portal/azure-portal-dashboards-create-programmatically).

L’illustration suivante présente le tableau de bord une fois que le travail Stream Analytics a été exécuté pendant environ une heure.

![Capture d’écran du tableau de bord des courses de taxi](./images/stream-processing-asa/asa-dashboard.png)

Le panneau affiché dans l’angle inférieur gauche indique que la consommation d’unités de streaming pour le travail Stream Analytics augmente pendant les 15 premières minutes, puis se stabilise. Il s’agit d’un modèle standard, car le travail atteint un état stable.

Notez que le service Event Hubs limite les requêtes (illustré dans le panneau supérieur droit). Une demande limitée occasionnelle n’est pas un problème, car le Kit de développement logiciel (SDK) client du service Event Hubs exécute une nouvelle tentative à la réception d’une erreur de limitation. Toutefois, si vous observez régulièrement des erreurs de limitation, cela signifie que l’Event Hub a besoin de davantage d’unités de débit. Le graphique suivant montre une série de tests utilisant la fonction d’augmentation automatique d’Event Hubs, qui ajuste automatiquement les unités de débit en fonction des besoins.

![Capture d’écran de la mise à l’échelle automatique Event Hubs](./images/stream-processing-asa/stream-processing-eh-autoscale.png)

La fonction d’augmentation automatique a été activée au niveau de la marque de 06:35. Vous pouvez observer la chute p du nombre de requêtes limitées, car le service Event Hubs est automatiquement mis à l’échelle jusqu’à 3 unités de débit.

L’effet secondaire de ce processus est l’augmentation de l’utilisation des unités de streaming dans le travail Stream Analytics. Via la limitation des requêtes, le service Event Hubs réduit artificiellement le taux d’ingestion des données pour le travail Stream Analytics. Il est assez courant que la résolution d’un problème de goulot d’étranglement de performances en révèle un autre. Dans ce cas, l’allocation d’unités de streaming supplémentaires pour le travail Stream Analytics a résolu le problème.

## <a name="deploy-the-solution"></a>Déployer la solution

Pour déployer et exécuter l’implémentation de référence, suivez les étapes du [fichier Readme de GitHub][github].

## <a name="related-resources"></a>Ressources associées

Vous pouvez consulter les [exemples de scénarios Azure](/azure/architecture/example-scenario) suivants, qui décrivent des solutions spécifiques utilisant certaines de ces technologies :

- [IoT et analyse de données dans le secteur de la construction](/azure/architecture/example-scenario/data/big-data-with-iot)
- [Détection des fraudes en temps réel](/azure/architecture/example-scenario/data/fraud-detection)

<!-- links -->

[github]: https://github.com/mspnp/azure-stream-analytics-data-pipeline
