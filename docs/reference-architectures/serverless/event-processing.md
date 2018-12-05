---
title: Traitement d’événements serverless à l’aide d’Azure Functions
description: Architecture de référence qui montre le traitement et l’ingestion d’événements serverless
author: MikeWasson
ms.date: 10/16/2018
ms.openlocfilehash: 76c8b9c1244c987c96e38e50ecad7814cc49cd88
ms.sourcegitcommit: 19a517a2fb70768b3edb9a7c3c37197baa61d9b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2018
ms.locfileid: "52295648"
---
# <a name="serverless-event-processing-using-azure-functions"></a><span data-ttu-id="f077e-103">Traitement d’événements serverless à l’aide d’Azure Functions</span><span class="sxs-lookup"><span data-stu-id="f077e-103">Serverless event processing using Azure Functions</span></span>

<span data-ttu-id="f077e-104">Cette architecture de référence montre une architecture [serverless](https://azure.microsoft.com/solutions/serverless/) basée sur des événements qui ingère un flux de données, traite les données et écrit les résultats dans une base de données back-end.</span><span class="sxs-lookup"><span data-stu-id="f077e-104">This reference architecture shows a [serverless](https://azure.microsoft.com/solutions/serverless/), event-driven architecture that ingests a stream of data, processes the data, and writes the results to a back-end database.</span></span> <span data-ttu-id="f077e-105">Une implémentation de référence pour cette architecture est disponible sur [GitHub][github].</span><span class="sxs-lookup"><span data-stu-id="f077e-105">A reference implementation for this architecture is available on [GitHub][github].</span></span>

![](./_images/serverless-event-processing.png)

## <a name="architecture"></a><span data-ttu-id="f077e-106">Architecture</span><span class="sxs-lookup"><span data-stu-id="f077e-106">Architecture</span></span>

<span data-ttu-id="f077e-107">**Event Hubs** ingère le flux de données.</span><span class="sxs-lookup"><span data-stu-id="f077e-107">**Event Hubs** ingests the data stream.</span></span> <span data-ttu-id="f077e-108">[Event Hubs] [ eh] est conçu pour les scénarios de diffusion de données à débit élevé.</span><span class="sxs-lookup"><span data-stu-id="f077e-108">[Event Hubs][eh] is designed for high-throughput data streaming scenarios.</span></span>

> [!NOTE]
> <span data-ttu-id="f077e-109">Pour des scénarios IoT, nous vous recommandons IoT Hub.</span><span class="sxs-lookup"><span data-stu-id="f077e-109">For IoT scenarios, we recommend IoT Hub.</span></span> <span data-ttu-id="f077e-110">IoT Hub a un point de terminaison intégré qui est compatible avec l’API Azure Event Hubs, vous pouvez donc utiliser ces services dans cette architecture sans aucune modification majeure dans le traitement principal.</span><span class="sxs-lookup"><span data-stu-id="f077e-110">IoT Hub has a built-in endpoint that’s compatible with the Azure Event Hubs API, so you can use either service in this architecture with no major changes in the backend processing.</span></span> <span data-ttu-id="f077e-111">Pour plus d’informations, consultez [Connexion des appareils IoT à Azure : IoT Hub et Event Hubs][iot].</span><span class="sxs-lookup"><span data-stu-id="f077e-111">For more information, see [Connecting IoT Devices to Azure: IoT Hub and Event Hubs][iot].</span></span>

<span data-ttu-id="f077e-112">**Function App**.</span><span class="sxs-lookup"><span data-stu-id="f077e-112">**Function App**.</span></span> <span data-ttu-id="f077e-113">[Azure Functions][functions] est une solution de calcul serverless.</span><span class="sxs-lookup"><span data-stu-id="f077e-113">[Azure Functions][functions] is a serverless compute option.</span></span> <span data-ttu-id="f077e-114">Cette application utilise un modèle événementiel, dans lequel un morceau de code (une « fonction ») est invoqué par un déclencheur.</span><span class="sxs-lookup"><span data-stu-id="f077e-114">It uses an event-driven model, where a piece of code (a “function”) is invoked by a trigger.</span></span> <span data-ttu-id="f077e-115">Dans cette architecture, lorsque des événements arrivent dans Event Hubs, ils déclenchent une fonction qui traite les événements et écrit les résultats dans le stockage.</span><span class="sxs-lookup"><span data-stu-id="f077e-115">In this architecture, when events arrive at Event Hubs, they trigger a function that processes the events and writes the results to storage.</span></span>

<span data-ttu-id="f077e-116">Function App ne convient pas pour le traitement des enregistrements individuels à partir d’Event Hubs.</span><span class="sxs-lookup"><span data-stu-id="f077e-116">Function Apps are suitable for processing individual records from Event Hubs.</span></span> <span data-ttu-id="f077e-117">Pour les scénarios de traitement de flux de données plus complexes, envisagez Apache Spark en utilisant Azure Databricks et Azure Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="f077e-117">For more complex stream processing scenarios, consider Apache Spark using Azure Databricks, or Azure Stream Analytics.</span></span>

<span data-ttu-id="f077e-118">**Cosmos DB**.</span><span class="sxs-lookup"><span data-stu-id="f077e-118">**Cosmos DB**.</span></span> <span data-ttu-id="f077e-119">[Cosmos DB][cosmosdb] est un service de base de données multimodèle.</span><span class="sxs-lookup"><span data-stu-id="f077e-119">[Cosmos DB][cosmosdb] is a multi-model database service.</span></span> <span data-ttu-id="f077e-120">Pour ce scénario, la fonction de traitement d’événements stocke les enregistrements JSON, à l’aide de l’[API SQL][cosmosdb-sql]Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="f077e-120">For this scenario, the event-processing function stores JSON records, using the Cosmos DB [SQL API][cosmosdb-sql].</span></span>

<span data-ttu-id="f077e-121">**Stockage de files d’attente**.</span><span class="sxs-lookup"><span data-stu-id="f077e-121">**Queue storage**.</span></span> <span data-ttu-id="f077e-122">[Stockage de files d’attente] [ queue] est utilisé pour les messages de lettres mortes.</span><span class="sxs-lookup"><span data-stu-id="f077e-122">[Queue storage][queue] is used for dead letter messages.</span></span> <span data-ttu-id="f077e-123">Si une erreur se produit lors du traitement d’un événement, la fonction stocke les données d’événement dans une file d’attente de lettres mortes pour un traitement ultérieur.</span><span class="sxs-lookup"><span data-stu-id="f077e-123">If an error occurs while processing an event, the function stores the event data in a dead letter queue for later processing.</span></span> <span data-ttu-id="f077e-124">Pour plus d’informations, consultez [Résilience et considérations](#resiliency-considerations).</span><span class="sxs-lookup"><span data-stu-id="f077e-124">For more information, see [Resiliency Considerations](#resiliency-considerations).</span></span>

<span data-ttu-id="f077e-125">**Azure Monitor**.</span><span class="sxs-lookup"><span data-stu-id="f077e-125">**Azure Monitor**.</span></span> <span data-ttu-id="f077e-126">[Monitor][monitor] collecte les mesures de performances concernant les services Azure déployés dans la solution.</span><span class="sxs-lookup"><span data-stu-id="f077e-126">[Monitor][monitor] collects performance metrics about the Azure services deployed in the solution.</span></span> <span data-ttu-id="f077e-127">En les visualisant dans un tableau de bord, vous pouvez obtenir des informations sur l’intégrité de la solution.</span><span class="sxs-lookup"><span data-stu-id="f077e-127">By visualizing these in a dashboard, you can get visibility  into the health of the solution.</span></span>

<span data-ttu-id="f077e-128">**Azure Pipelines**.</span><span class="sxs-lookup"><span data-stu-id="f077e-128">**Azure Pipelines**.</span></span> <span data-ttu-id="f077e-129">[Pipelines] [ pipelines] est un service d’intégration continue (CI) et de livraison continue (CD) qui génère, teste et déploie l’application.</span><span class="sxs-lookup"><span data-stu-id="f077e-129">[Pipelines][pipelines] is a continuous integration (CI) and continuous delivery (CD) service that builds, tests, and deploys the application.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="f077e-130">Considérations relatives à l’extensibilité</span><span class="sxs-lookup"><span data-stu-id="f077e-130">Scalability considerations</span></span>

### <a name="event-hubs"></a><span data-ttu-id="f077e-131">Event Hubs</span><span class="sxs-lookup"><span data-stu-id="f077e-131">Event Hubs</span></span>

<span data-ttu-id="f077e-132">La capacité de débit du service Event Hubs est mesurée par les [unités de débit][eh-throughput].</span><span class="sxs-lookup"><span data-stu-id="f077e-132">The throughput capacity of Event Hubs is measured in [throughput units][eh-throughput].</span></span> <span data-ttu-id="f077e-133">Vous pouvez mettre automatiquement à l’échelle un Event Hub en activant [l’augmentation automatique][eh-autoscale], qui ajuste automatiquement les unités de débit en fonction du trafic, jusqu’à la limite configurée.</span><span class="sxs-lookup"><span data-stu-id="f077e-133">You can autoscale an event hub by enabling [auto-inflate][eh-autoscale], which automatically scales the throughput units based on traffic, up to a configured maximum.</span></span>

<span data-ttu-id="f077e-134">Le [déclencheur Event Hub] [ eh-trigger] dans la fonction application met à l’échelle en fonction du nombre de partitions dans l’event hub.</span><span class="sxs-lookup"><span data-stu-id="f077e-134">The [Event Hub trigger][eh-trigger] in the function app scales according to the number of partitions in the event hub.</span></span> <span data-ttu-id="f077e-135">Une instance de la fonction est assignée à chaque partition à la fois.</span><span class="sxs-lookup"><span data-stu-id="f077e-135">Each partition is assigned one function instance at a time.</span></span> <span data-ttu-id="f077e-136">Pour maximiser le débit, recevez les événements en un traitement, plutôt qu’un seul à la fois.</span><span class="sxs-lookup"><span data-stu-id="f077e-136">To maximize throughput, receive the events in a batch, instead of one at a time.</span></span>

### <a name="cosmos-db"></a><span data-ttu-id="f077e-137">Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="f077e-137">Cosmos DB</span></span>

<span data-ttu-id="f077e-138">La capacité de débit pour Cosmos DB est mesurée en [unités de requête][ru] (RU).</span><span class="sxs-lookup"><span data-stu-id="f077e-138">Throughput capacity for Cosmos DB is measured in [Request Units][ru] (RU).</span></span> <span data-ttu-id="f077e-139">Pour mettre à l’échelle un conteneur Cosmos DB au-delà de 10 000 RU, vous devez spécifier une [clé de partition][partition-key] lorsque vous créez le conteneur, puis inclure la clé de partition dans chaque document que vous créez.</span><span class="sxs-lookup"><span data-stu-id="f077e-139">In order to scale a Cosmos DB container past 10,000 RU, you must specify a [partition key][partition-key] when you create the container, and include the partition key in every document that you create.</span></span>

<span data-ttu-id="f077e-140">Voici certaines caractéristiques d’une bonne clé de partition :</span><span class="sxs-lookup"><span data-stu-id="f077e-140">Here are some characteristics of a good partition key:</span></span>

- <span data-ttu-id="f077e-141">L’espace de valeur de clé est volumineux.</span><span class="sxs-lookup"><span data-stu-id="f077e-141">The key value space is large.</span></span> 
- <span data-ttu-id="f077e-142">Il y aura une répartition uniforme des lectures/écritures par valeur de clé, en évitant les clés sensibles.</span><span class="sxs-lookup"><span data-stu-id="f077e-142">There will be an even distribution of reads/writes per key value, avoiding hot keys.</span></span>
- <span data-ttu-id="f077e-143">Le maximum des données stockées pour toute valeur de clé unique ne dépasse pas la taille maximale de la partition physique (10 Go).</span><span class="sxs-lookup"><span data-stu-id="f077e-143">The maximum data stored for any single key value will not exceed the maximum physical partition size (10 GB).</span></span> 
- <span data-ttu-id="f077e-144">La clé de partition pour un document ne change pas.</span><span class="sxs-lookup"><span data-stu-id="f077e-144">The partition key for a document won't change.</span></span> <span data-ttu-id="f077e-145">Vous ne pouvez pas mettre à jour la clé de partition sur un document existant.</span><span class="sxs-lookup"><span data-stu-id="f077e-145">You can't update the partition key on an existing document.</span></span> 

<span data-ttu-id="f077e-146">Dans le scénario pour cette architecture de référence, la fonction stocke un seul document par appareil qui envoie des données.</span><span class="sxs-lookup"><span data-stu-id="f077e-146">In the scenario for this reference architecture, the function stores exactly one document per device that is sending data.</span></span> <span data-ttu-id="f077e-147">La fonction met continuellement à jour les documents avec le dernier état de l’appareil, à l’aide d’une opération upsert.</span><span class="sxs-lookup"><span data-stu-id="f077e-147">The function continually updates the documents with latest device status, using an upsert operation.</span></span> <span data-ttu-id="f077e-148">L’ID de l’appareil est une bonne clé de partition pour ce scénario, car les écritures seront réparties uniformément entre les clés et la taille de chaque partition sera strictement limitée, étant donné qu’il existe un document unique pour chaque valeur de clé.</span><span class="sxs-lookup"><span data-stu-id="f077e-148">Device ID is a good partition key for this scenario, because writes will be evenly distributed across the keys, and the size of each partition will be strictly bounded, because there is a single document for each key value.</span></span> <span data-ttu-id="f077e-149">Pour plus d’informations sur les clés de partition, voir [Partition et mise à l’échelle dans Azure Cosmos DB][cosmosdb-scale].</span><span class="sxs-lookup"><span data-stu-id="f077e-149">For more information about partition keys, see [Partition and scale in Azure Cosmos DB][cosmosdb-scale].</span></span>

## <a name="resiliency-considerations"></a><span data-ttu-id="f077e-150">Remarques relatives à la résilience</span><span class="sxs-lookup"><span data-stu-id="f077e-150">Resiliency considerations</span></span>

<span data-ttu-id="f077e-151">Lorsque vous utilisez le déclencheur Event Hubs avec Functions, intercepter les exceptions dans votre boucle de traitement.</span><span class="sxs-lookup"><span data-stu-id="f077e-151">When using the Event Hubs trigger with Functions, catch exceptions within your processing loop.</span></span> <span data-ttu-id="f077e-152">Si une exception non prise en charge se produit, le runtime Functions ne retente pas les messages.</span><span class="sxs-lookup"><span data-stu-id="f077e-152">If an unhandled exception occurs, the Functions runtime does not retry the messages.</span></span> <span data-ttu-id="f077e-153">Si un message ne peut pas être traité, placez le message dans une file d’attente de lettres mortes.</span><span class="sxs-lookup"><span data-stu-id="f077e-153">If a message cannot be processed, put the message into a dead letter queue.</span></span> <span data-ttu-id="f077e-154">Utilisez un processus hors-bande pour examiner les messages et de déterminer l’action corrective.</span><span class="sxs-lookup"><span data-stu-id="f077e-154">Use an out-of-band process to examine the messages and determine corrective action.</span></span> 

<span data-ttu-id="f077e-155">Le code suivant montre comment la fonction d’ingestion intercepte les exceptions et place les messages non traités dans une file d’attente de lettres mortes.</span><span class="sxs-lookup"><span data-stu-id="f077e-155">The following code shows how the ingestion function catches exceptions and puts unprocessed messages onto a dead letter queue.</span></span>

```csharp
[FunctionName("RawTelemetryFunction")]
[StorageAccount("DeadLetterStorage")]
public static async Task RunAsync(
    [EventHubTrigger("%EventHubName%", Connection = "EventHubConnection", ConsumerGroup ="%EventHubConsumerGroup%")]EventData[] messages,
    [Queue("deadletterqueue")] IAsyncCollector<DeadLetterMessage> deadLetterMessages,
    ILogger logger)
{
    foreach (var message in messages)
    {
        DeviceState deviceState = null;

        try
        {
            deviceState = telemetryProcessor.Deserialize(message.Body.Array, logger);
        }
        catch (Exception ex)
        {
            logger.LogError(ex, "Error deserializing message", message.SystemProperties.PartitionKey, message.SystemProperties.SequenceNumber);
            await deadLetterMessages.AddAsync(new DeadLetterMessage { Issue = ex.Message, EventData = message });
        }

        try
        {
            await stateChangeProcessor.UpdateState(deviceState, logger);
        }
        catch (Exception ex)
        {
            logger.LogError(ex, "Error updating status document", deviceState);
            await deadLetterMessages.AddAsync(new DeadLetterMessage { Issue = ex.Message, EventData = message, DeviceState = deviceState });
        }
    }
}
```

<span data-ttu-id="f077e-156">Notez que la fonction utilise la [liaison de sortie stockage de files d’attente] [ queue-binding] pour placer des éléments dans la file d’attente.</span><span class="sxs-lookup"><span data-stu-id="f077e-156">Notice that the function uses the [Queue storage output binding][queue-binding] to put items in the queue.</span></span>

<span data-ttu-id="f077e-157">Le code ci-dessus enregistre également les exceptions dans Application Insights.</span><span class="sxs-lookup"><span data-stu-id="f077e-157">The code shown above also logs exceptions to Application Insights.</span></span> <span data-ttu-id="f077e-158">Vous pouvez utiliser le numéro de séquence et de clé de partition pour mettre en corrélation des messages de lettres mortes avec les exceptions dans les journaux.</span><span class="sxs-lookup"><span data-stu-id="f077e-158">You can use the partition key and sequence number to correlate dead letter messages with the exceptions in the logs.</span></span> 

<span data-ttu-id="f077e-159">Les messages dans la file d’attente de lettres mortes doivent posséder suffisamment d’informations pour que vous puissiez comprendre le contexte d’erreur.</span><span class="sxs-lookup"><span data-stu-id="f077e-159">Messages in the dead letter queue should have enough information so that you can understand the context of error.</span></span> <span data-ttu-id="f077e-160">Dans cet exemple, la `DeadLetterMessage` classe contient le message d’exception, les données d’événement d’origine et le message d’événement désérialisé (si disponible).</span><span class="sxs-lookup"><span data-stu-id="f077e-160">In this example, the `DeadLetterMessage` class contains the exception message, the original event data, and the deserialized event message (if available).</span></span> 

```csharp
public class DeadLetterMessage
{
    public string Issue { get; set; }
    public EventData EventData { get; set; }
    public DeviceState DeviceState { get; set; }
}
```

<span data-ttu-id="f077e-161">Utilisation d’[Azure Monitor][monitor] pour surveiller Event Hub.</span><span class="sxs-lookup"><span data-stu-id="f077e-161">Use [Azure Monitor][monitor] to monitor the event hub.</span></span> <span data-ttu-id="f077e-162">Si vous voyez qu’il y a des entrées, mais aucune sortie, cela signifie que les messages ne sont pas traités.</span><span class="sxs-lookup"><span data-stu-id="f077e-162">If you see there is input but no output, it means that messages are not being processed.</span></span> <span data-ttu-id="f077e-163">Dans ce cas, accédez à [Log Analytics] [ log-analytics] et recherchez les exceptions ou d’autres erreurs.</span><span class="sxs-lookup"><span data-stu-id="f077e-163">In that case, go into [Log Analytics][log-analytics] and look for exceptions or other errors.</span></span>

## <a name="disaster-recovery-considerations"></a><span data-ttu-id="f077e-164">Considérations relatives à la récupération d’urgence</span><span class="sxs-lookup"><span data-stu-id="f077e-164">Disaster recovery considerations</span></span>

<span data-ttu-id="f077e-165">Le déploiement illustré ici se trouve dans une seule région Azure.</span><span class="sxs-lookup"><span data-stu-id="f077e-165">The deployment shown here resides in a single Azure region.</span></span> <span data-ttu-id="f077e-166">Pour une approche plus résiliente à la récupération d’urgence, tirer parti des fonctionnalités de géo-distribution dans les différents services :</span><span class="sxs-lookup"><span data-stu-id="f077e-166">For a more resilient approach to disaster-recovery, take advantage of geo-distribution features in the various services:</span></span>

- <span data-ttu-id="f077e-167">**Event Hubs**.</span><span class="sxs-lookup"><span data-stu-id="f077e-167">**Event Hubs**.</span></span> <span data-ttu-id="f077e-168">Créez deux espaces de noms Event Hubs, un espace de noms principal (actif) et un espace de noms secondaire (passif).</span><span class="sxs-lookup"><span data-stu-id="f077e-168">Create two Event Hubs namespaces, a primary (active) namespace and a secondary (passive) namespace.</span></span> <span data-ttu-id="f077e-169">Les messages sont automatiquement acheminés vers l’espace de noms actif, sauf si vous basculez vers l’espace de noms secondaire.</span><span class="sxs-lookup"><span data-stu-id="f077e-169">Messages are automatically routed to the active namespace unless you fail over to the secondary namespace.</span></span> <span data-ttu-id="f077e-170">Pour en savoir plus, voir [Géo-reprise d’activité après sinistre Azure Event Hubs][eh-dr].</span><span class="sxs-lookup"><span data-stu-id="f077e-170">For more information, see [Azure Event Hubs Geo-disaster recovery][eh-dr].</span></span>

- <span data-ttu-id="f077e-171">**Function App**.</span><span class="sxs-lookup"><span data-stu-id="f077e-171">**Function App**.</span></span> <span data-ttu-id="f077e-172">Déployez une deuxième application de fonction qui est en attente de lecture à partir de l’espace de noms Event Hubs secondaire.</span><span class="sxs-lookup"><span data-stu-id="f077e-172">Deploy a second function app that is waiting to read from the secondary Event Hubs namespace.</span></span> <span data-ttu-id="f077e-173">Cette fonction écrit dans un compte de stockage secondaire pour la file d’attente de lettres mortes.</span><span class="sxs-lookup"><span data-stu-id="f077e-173">This function writes to a secondary storage account for dead letter queue.</span></span>

- <span data-ttu-id="f077e-174">**Cosmos DB**.</span><span class="sxs-lookup"><span data-stu-id="f077e-174">**Cosmos DB**.</span></span> <span data-ttu-id="f077e-175">COSMOS DB prend en charge [plusieurs régions principales][cosmosdb-geo], ce qui permet des écritures sur n’importe quelle région que vous ajoutez à votre compte Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="f077e-175">Cosmos DB supports [multiple master regions][cosmosdb-geo], which enables writes to any region that you add to your Cosmos DB account.</span></span> <span data-ttu-id="f077e-176">Si vous n’activez pas multimaître, vous pouvez toujours basculer vers la région d’écriture primaire.</span><span class="sxs-lookup"><span data-stu-id="f077e-176">If you don’t enable multi-master, you can still fail over the primary write region.</span></span> <span data-ttu-id="f077e-177">Les kits de développement client Cosmos DB et les liaisons d’Azure Function gèrent automatiquement le basculement, vous n’avez pas besoin de mettre à jour les paramètres de configuration d’application.</span><span class="sxs-lookup"><span data-stu-id="f077e-177">The Cosmos DB client SDKs and the Azure Function bindings automatically handle the failover, so you don’t need to update any application configuration settings.</span></span>

- <span data-ttu-id="f077e-178">**Stockage Azure**.</span><span class="sxs-lookup"><span data-stu-id="f077e-178">**Azure Storage**.</span></span> <span data-ttu-id="f077e-179">Utilisez le stockage [RA-GRS] [ ra-grs] pour la file d’attente de lettres mortes.</span><span class="sxs-lookup"><span data-stu-id="f077e-179">Use [RA-GRS][ra-grs] storage for the dead letter queue.</span></span> <span data-ttu-id="f077e-180">Cette opération crée un réplica en lecture seule dans une autre région.</span><span class="sxs-lookup"><span data-stu-id="f077e-180">This creates a read-only replica in another region.</span></span> <span data-ttu-id="f077e-181">Si la région primaire devient indisponible, vous pouvez lire les éléments actuellement présents dans la file d’attente.</span><span class="sxs-lookup"><span data-stu-id="f077e-181">If the primary region becomes unavailable, you can read the items currently in the queue.</span></span> <span data-ttu-id="f077e-182">En outre, configurez un autre compte de stockage dans la région secondaire à laquelle la fonction peut écrire après un basculement.</span><span class="sxs-lookup"><span data-stu-id="f077e-182">In addition, provision another storage account in the secondary region that the function can write to after a fail-over.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="f077e-183">Déployer la solution</span><span class="sxs-lookup"><span data-stu-id="f077e-183">Deploy the solution</span></span>

<span data-ttu-id="f077e-184">Pour déployer cette architecture de référence, affichez le [fichier readme de GitHub][readme].</span><span class="sxs-lookup"><span data-stu-id="f077e-184">To deploy this reference architecture, view the [GitHub readme][readme].</span></span> 

<!-- links -->

[cosmosdb]: /azure/cosmos-db/introduction
[cosmosdb-geo]: /azure/cosmos-db/distribute-data-globally
[cosmosdb-scale]: /azure/cosmos-db/partition-data
[cosmosdb-sql]: /azure/cosmos-db/sql-api-introduction
[eh]: /azure/event-hubs/
[eh-autoscale]: /azure/event-hubs/event-hubs-auto-inflate
[eh-dr]: /azure/event-hubs/event-hubs-geo-dr
[eh-throughput]: /azure/event-hubs/event-hubs-features#throughput-units
[eh-trigger]: /azure/azure-functions/functions-bindings-event-hubs
[functions]: /azure/azure-functions/functions-overview
[iot]: /azure/iot-hub/iot-hub-compare-event-hubs
[log-analytics]: /azure/log-analytics/log-analytics-queries
[monitor]: /azure/azure-monitor/overview
[partition-key]: /azure/cosmos-db/partition-data
[pipelines]: /azure/devops/pipelines/index
[queue]: /azure/storage/queues/storage-queues-introduction
[queue-binding]: /azure/azure-functions/functions-bindings-storage-queue#output
[ra-grs]: /azure/storage/common/storage-redundancy-grs
[ru]: /azure/cosmos-db/request-units

[github]: https://github.com/mspnp/serverless-reference-implementation
[readme]: https://github.com/mspnp/serverless-reference-implementation/blob/master/README.md
