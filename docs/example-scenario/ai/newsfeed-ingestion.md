---
title: Ingestion et analyse en masse de flux d’actualités sur Azure
description: Créez un pipeline pour ingérer et analyser du texte, des images, des sentiments et autres données de flux d’actualités RSS en utilisant seulement des services Azure, notamment Azure Cosmos DB et Azure Cognitive Services.
author: njray
ms.date: 2/1/2019
ms.custom: azcat-ai, AI
social_image_url: /azure/architecture/example-scenario/ai/media/mass-ingestion-newsfeeds-architecture.png
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.openlocfilehash: e1bc636103753d474b545a6e3e9118b2ef302b34
ms.sourcegitcommit: ea97ac004c38c6b456794c1a8eef29f8d2b77d50
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/26/2019
ms.locfileid: "58489256"
---
# <a name="mass-ingestion-and-analysis-of-news-feeds-on-azure"></a><span data-ttu-id="23409-103">Ingestion et analyse en masse de flux d’actualités sur Azure</span><span class="sxs-lookup"><span data-stu-id="23409-103">Mass ingestion and analysis of news feeds on Azure</span></span>

<span data-ttu-id="23409-104">Cet exemple de scénario décrit un pipeline pour l’ingestion massive et près de l’analyse en temps réel de documents à l’aide de canaux RSS publiques.</span><span class="sxs-lookup"><span data-stu-id="23409-104">This example scenario describes a pipeline for mass ingestion and near real-time analysis of documents using public RSS news feeds.</span></span>  <span data-ttu-id="23409-105">Elle utilise Azure Cognitive Services pour fournissent des informations utiles, y compris la traduction de texte, la reconnaissance des visages et la détection de sentiments.</span><span class="sxs-lookup"><span data-stu-id="23409-105">It uses Azure Cognitive Services to offer useful insights including text translation, facial recognition, and sentiment detection.</span></span>

<span data-ttu-id="23409-106">Ce scénario contient des exemples pour [anglais][english], [russe][russian], et [allemand] [ german] news flux, mais vous pouvez l’étendre facilement aux autres flux RSS.</span><span class="sxs-lookup"><span data-stu-id="23409-106">This scenario contains examples for [English][english], [Russian][russian], and [German][german] news feeds, but you can easily extend it to other RSS feeds.</span></span> <span data-ttu-id="23409-107">Pour faciliter le déploiement, la collecte de données, le traitement et l’analyse reposent entièrement sur les services Azure.</span><span class="sxs-lookup"><span data-stu-id="23409-107">For ease of deployment, the data collection, processing, and analysis are based entirely on Azure services.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="23409-108">Cas d’usage appropriés</span><span class="sxs-lookup"><span data-stu-id="23409-108">Relevant use cases</span></span>

<span data-ttu-id="23409-109">Tandis que ce scénario est basé sur le traitement de flux RSS, elle s’applique à n’importe quel document, un site Web ou un article dans lequel vous devez :</span><span class="sxs-lookup"><span data-stu-id="23409-109">While this scenario is based on processing of RSS feeds, it's relevant to any document, website, or article where you would need to:</span></span>

* <span data-ttu-id="23409-110">Traduire un texte pour le langage de choix.</span><span class="sxs-lookup"><span data-stu-id="23409-110">Translate any text to the language of choice.</span></span>
* <span data-ttu-id="23409-111">Rechercher des expressions clés, les entités et les sentiments de l’utilisateur dans le contenu numérique.</span><span class="sxs-lookup"><span data-stu-id="23409-111">Find key phrases, entities, and user sentiment in digital content.</span></span>
* <span data-ttu-id="23409-112">Détecter des objets, le texte et points de repère dans des images associés à un article numérique.</span><span class="sxs-lookup"><span data-stu-id="23409-112">Detect objects, text, and landmarks in images associated with a digital article.</span></span>
* <span data-ttu-id="23409-113">Détecter des personnes par leur type et un âge dans n’importe quelle image associée à contenu numérique.</span><span class="sxs-lookup"><span data-stu-id="23409-113">Detect people by their gender and age in any image associated with digital content.</span></span>

## <a name="architecture"></a><span data-ttu-id="23409-114">Architecture</span><span class="sxs-lookup"><span data-stu-id="23409-114">Architecture</span></span>

![][architecture]

<span data-ttu-id="23409-115">Les données circulent dans la solution comme suit :</span><span class="sxs-lookup"><span data-stu-id="23409-115">The data flows through the solution as follows:</span></span>

1. <span data-ttu-id="23409-116">Un flux d’actualités RSS agit comme le générateur qui obtient des données à partir d’un document ou un article.</span><span class="sxs-lookup"><span data-stu-id="23409-116">An RSS news feed acts as the generator that obtains data from a document or article.</span></span> <span data-ttu-id="23409-117">Par exemple, avec un article, les données comprend généralement un titre, un résumé de l’organisme d’origine de l’article et parfois d’images.</span><span class="sxs-lookup"><span data-stu-id="23409-117">For example, with an article, data typically includes a title, a summary of the original body of the news item, and sometimes images.</span></span>

2. <span data-ttu-id="23409-118">Un processus de générateur ou ingestion insère l’article et les images associées dans Azure Cosmos DB [Collection][collection].</span><span class="sxs-lookup"><span data-stu-id="23409-118">A generator or ingestion process inserts the article and any associated images into an Azure Cosmos DB [Collection][collection].</span></span>

3. <span data-ttu-id="23409-119">Une notification déclenche une fonction de réception dans Azure Functions qui enregistre le texte de l’article dans CosmosDB et les images de l’article (le cas échéant) dans le stockage Blob Azure.</span><span class="sxs-lookup"><span data-stu-id="23409-119">A notification triggers an ingest function in Azure Functions that stores the article text in CosmosDB and the article images (if any) in Azure Blob Storage.</span></span>  <span data-ttu-id="23409-120">L’article est ensuite transmis à la file d’attente suivante.</span><span class="sxs-lookup"><span data-stu-id="23409-120">The article is then passed to the next queue.</span></span>

4. <span data-ttu-id="23409-121">Une fonction de traduction est déclenchée par l’événement de la file d’attente.</span><span class="sxs-lookup"><span data-stu-id="23409-121">A translate function is triggered by the queue event.</span></span> <span data-ttu-id="23409-122">Il utilise le [traduire les API de texte] [ translate-text] d’Azure Cognitive Services pour détecter la langue, traduire, si nécessaire et y recueillir les sentiments, les expressions clés et les entités le corps et le titre.</span><span class="sxs-lookup"><span data-stu-id="23409-122">It uses the [Translate Text API][translate-text] of Azure Cognitive Services to detect the language, translate if necessary, and collect the sentiment, key phrases, and entities from the body and the title.</span></span> <span data-ttu-id="23409-123">Il transmet ensuite l’article à la file d’attente suivante.</span><span class="sxs-lookup"><span data-stu-id="23409-123">Then it passes the article to the next queue.</span></span>

5. <span data-ttu-id="23409-124">Une fonction de détection est déclenchée à partir de l’article en file d’attente.</span><span class="sxs-lookup"><span data-stu-id="23409-124">A detect function is triggered from the queued article.</span></span> <span data-ttu-id="23409-125">Il utilise le [vision par ordinateur] [ vision] service pour détecter des objets, les points de repère et les mots écrits dans l’image associée, puis transmet l’article à la file d’attente suivante.</span><span class="sxs-lookup"><span data-stu-id="23409-125">It uses the [Computer Vision][vision] service to detect objects, landmarks, and written words in the associated image, then passes the article to the next queue.</span></span>

6. <span data-ttu-id="23409-126">Une fonction face est déclenchée est déclenchée à partir de l’article en file d’attente.</span><span class="sxs-lookup"><span data-stu-id="23409-126">A face function is triggered is triggered from the queued article.</span></span> <span data-ttu-id="23409-127">Il utilise le [API visage de Azure] [ face] service pour détecter les faces pour sexe et l’âge dans l’image associée, puis transmet l’article à la file d’attente suivante.</span><span class="sxs-lookup"><span data-stu-id="23409-127">It uses the [Azure Face API][face] service to detect faces for gender and age in the associated image, then passes the article to the next queue.</span></span>

7. <span data-ttu-id="23409-128">Lorsque toutes les fonctions sont terminées, la fonction de notification est déclenchée.</span><span class="sxs-lookup"><span data-stu-id="23409-128">When all functions are complete, the notify function is triggered.</span></span> <span data-ttu-id="23409-129">Il charge les enregistrements traités pour l’article et les analyse pour les résultats souhaités.</span><span class="sxs-lookup"><span data-stu-id="23409-129">It loads the processed records for the article and scans them for any results you want.</span></span> <span data-ttu-id="23409-130">Si trouvée, le contenu est marqué et une notification est envoyée au système de votre choix.</span><span class="sxs-lookup"><span data-stu-id="23409-130">If found, the content is flagged and a notification is sent to the system of your choice.</span></span>

<span data-ttu-id="23409-131">À chaque étape du processus, la fonction écrit les résultats dans Azure Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="23409-131">At each processing step, the function writes the results to Azure Cosmos DB.</span></span> <span data-ttu-id="23409-132">Au final, les données peuvent être utilisées comme vous le souhaitez.</span><span class="sxs-lookup"><span data-stu-id="23409-132">Ultimately, the data can be used as desired.</span></span> <span data-ttu-id="23409-133">Par exemple, vous pouvez l’utiliser pour améliorer les processus d’entreprise, recherchez les nouveaux clients ou identifier les problèmes de satisfaction client.</span><span class="sxs-lookup"><span data-stu-id="23409-133">For example, you can use it to enhance business processes, locate new customers, or identify customer satisfaction issues.</span></span>

### <a name="components"></a><span data-ttu-id="23409-134">Composants</span><span class="sxs-lookup"><span data-stu-id="23409-134">Components</span></span>

<span data-ttu-id="23409-135">La liste suivante des composants Azure est utilisée dans cet exemple.</span><span class="sxs-lookup"><span data-stu-id="23409-135">The following list of Azure components is used in this example.</span></span>

* <span data-ttu-id="23409-136">[Stockage Azure] [ storage] est utilisé pour contenir des images bruts et des fichiers vidéo associés à un article.</span><span class="sxs-lookup"><span data-stu-id="23409-136">[Azure Storage][storage] is used to hold raw image and video files associated with an article.</span></span> <span data-ttu-id="23409-137">Un compte de stockage secondaire est créé avec Azure App Service et est utilisé pour héberger le code de fonction Azure et les journaux.</span><span class="sxs-lookup"><span data-stu-id="23409-137">A secondary storage account is created with Azure App Service and is used to host the Azure Function code and logs.</span></span>

* <span data-ttu-id="23409-138">[Azure Cosmos DB] [ cosmos-db] contient l’article texte, image et vidéo des informations de suivi.</span><span class="sxs-lookup"><span data-stu-id="23409-138">[Azure Cosmos DB][cosmos-db] holds article text, image, and video tracking information.</span></span> <span data-ttu-id="23409-139">Les résultats des étapes de Cognitive Services sont également stockés ici.</span><span class="sxs-lookup"><span data-stu-id="23409-139">The results of the Cognitive Services steps are also stored here.</span></span>

* <span data-ttu-id="23409-140">[Azure Functions] [ functions] exécute le code de fonction utilisé pour répondre aux messages de file d’attente et de transformer le contenu entrant.</span><span class="sxs-lookup"><span data-stu-id="23409-140">[Azure Functions][functions] executes the function code used to respond to queue messages and transform the incoming content.</span></span> <span data-ttu-id="23409-141">[Azure App Service] [ aas] héberge le code de fonction et traite les enregistrements en série.</span><span class="sxs-lookup"><span data-stu-id="23409-141">[Azure App Service][aas] hosts the function code and processes the records serially.</span></span> <span data-ttu-id="23409-142">Ce scénario comprend cinq fonctions : Ingérer, transformer, détecter l’objet, visage et en informer.</span><span class="sxs-lookup"><span data-stu-id="23409-142">This scenario includes five functions: Ingest, Transform, Detect Object, Face, and Notify.</span></span>

* <span data-ttu-id="23409-143">[Azure Service Bus] [ service-bus] héberge les files d’attente Azure Service Bus utilisés par les fonctions.</span><span class="sxs-lookup"><span data-stu-id="23409-143">[Azure Service Bus][service-bus] hosts the Azure Service Bus queues used by the functions.</span></span>

* <span data-ttu-id="23409-144">[Azure Cognitive Services] [ acs] fournit l’intelligence artificielle pour le pipeline selon les implémentations de la [vision par ordinateur] [ vision] service, [Face API][face], et [traduire le texte] [ translate-text] service de traduction automatique.</span><span class="sxs-lookup"><span data-stu-id="23409-144">[Azure Cognitive Services][acs] provides the AI for the pipeline based on implementations of the [Computer Vision][vision] service, [Face API][face], and [Translate Text][translate-text] machine translation service.</span></span>

* <span data-ttu-id="23409-145">[Azure Application Insights] [ aai] fournit analytique pour vous aider à diagnostiquer les problèmes et à comprendre les fonctionnalités de votre application.</span><span class="sxs-lookup"><span data-stu-id="23409-145">[Azure Application Insights][aai] provides analytics to help you diagnose issues and to understand functionality of your application.</span></span>

### <a name="alternatives"></a><span data-ttu-id="23409-146">Autres solutions</span><span class="sxs-lookup"><span data-stu-id="23409-146">Alternatives</span></span>

* <span data-ttu-id="23409-147">Au lieu d’utiliser un modèle basé sur la notification de la file d’attente et Azure Functions, utilisez un autre modèle pour ce flux de données.</span><span class="sxs-lookup"><span data-stu-id="23409-147">Instead of using a pattern based on queue notification and Azure Functions, use another pattern for this data flow.</span></span> <span data-ttu-id="23409-148">Par exemple, [rubriques Azure Service Bus] [ topics] peut être utilisé pour les processus et non les différentes parties de l’article en parallèle en tant que le traitement en série effectué dans cet exemple.</span><span class="sxs-lookup"><span data-stu-id="23409-148">For example, [Azure Service Bus Topics][topics] can be used to processes the various parts of the article in parallel as opposed to the serial processing done in this example.</span></span> <span data-ttu-id="23409-149">Pour plus d’informations, comparer [files d’attente et rubriques][queues-topics].</span><span class="sxs-lookup"><span data-stu-id="23409-149">For more information, compare [queues and topics][queues-topics].</span></span>

* <span data-ttu-id="23409-150">Utilisez [application logique Azure] [ logic-app] pour implémenter le code de fonction et d’implémenter le verrouillage des enregistrements comme [Redlock] [ redlock] (nécessaire en parallèle traitement jusqu'à ce qu’Azure Cosmos DB prend en charge [mises à jour partielles de documents][partial]).</span><span class="sxs-lookup"><span data-stu-id="23409-150">Use [Azure Logic App][logic-app] to implement the function code and implement record-level locking such as [Redlock][redlock] (needed for parallel processing until Azure Cosmos DB supports [partial document updates][partial]).</span></span> <span data-ttu-id="23409-151">Pour plus d’informations, [comparer Functions et Logic Apps][compare].</span><span class="sxs-lookup"><span data-stu-id="23409-151">For more information, [compare Functions and Logic Apps][compare].</span></span>

* <span data-ttu-id="23409-152">Implémentation de cette architecture à l’aide de composants d’intelligence artificielle personnalisés plutôt que les services Azure existants.</span><span class="sxs-lookup"><span data-stu-id="23409-152">Implement this architecture using customized AI components rather than existing Azure services.</span></span> <span data-ttu-id="23409-153">Par exemple, étendre le pipeline à l’aide d’un modèle personnalisé qui détecte certaines personnes dans une image plutôt que le nombre de personnes générique, sexe et âge des données collectées dans cet exemple.</span><span class="sxs-lookup"><span data-stu-id="23409-153">For example, extend the pipeline using a customized model that detects certain people in an image as opposed to the generic people count, gender, and age data collected in this example.</span></span> <span data-ttu-id="23409-154">Pour utiliser l’apprentissage automatique personnalisés ou des modèles d’intelligence artificielle avec cette architecture, créer les modèles en tant que points de terminaison RESTful peuvent donc être appelés à partir d’Azure Functions.</span><span class="sxs-lookup"><span data-stu-id="23409-154">To use customized machine learning or AI models with this architecture, build the models as RESTful endpoints so they can be called from Azure Functions.</span></span>

* <span data-ttu-id="23409-155">Utilisez un autre mécanisme d’entrée au lieu de flux RSS.</span><span class="sxs-lookup"><span data-stu-id="23409-155">Use a different input mechanism instead of RSS feeds.</span></span> <span data-ttu-id="23409-156">Permet de plusieurs générateurs ou processus d’ingestion de flux Azure Cosmos DB et le stockage Azure.</span><span class="sxs-lookup"><span data-stu-id="23409-156">Use multiple generators or ingestion processes to feed Azure Cosmos DB and Azure Storage.</span></span>

## <a name="considerations"></a><span data-ttu-id="23409-157">Considérations</span><span class="sxs-lookup"><span data-stu-id="23409-157">Considerations</span></span>

<span data-ttu-id="23409-158">Par souci de simplicité, cet exemple de scénario utilise uniquement quelques exemples des API disponibles et des services d’Azure Cognitive Services.</span><span class="sxs-lookup"><span data-stu-id="23409-158">For simplicity, this example scenario uses only a few of the available APIs and services from Azure Cognitive Services.</span></span> <span data-ttu-id="23409-159">Par exemple, le texte dans les images peut être analysé à l’aide de la [API d’Analytique de texte][text-analytics].</span><span class="sxs-lookup"><span data-stu-id="23409-159">For example, text in images can be analyzed using the [Text Analytics API][text-analytics].</span></span> <span data-ttu-id="23409-160">La langue cible dans ce scénario est supposée pour être l’anglais, mais vous pouvez modifier l’entrée à tout [langue prise en charge] [ language] de votre choix.</span><span class="sxs-lookup"><span data-stu-id="23409-160">The target language in this scenario is assumed to be English, but you can change the input to any [supported language][language] of your choice.</span></span>

### <a name="scalability"></a><span data-ttu-id="23409-161">Extensibilité</span><span class="sxs-lookup"><span data-stu-id="23409-161">Scalability</span></span>

<span data-ttu-id="23409-162">Les fonctions Azure mise à l’échelle varie selon le [plan d’hébergement] [ plan] vous utilisez.</span><span class="sxs-lookup"><span data-stu-id="23409-162">Azure Functions scaling depends on the [hosting plan][plan] you use.</span></span> <span data-ttu-id="23409-163">Cette solution suppose un [plan de consommation][plan-c], dans le calcul power est alloué automatiquement aux fonctions si nécessaire.</span><span class="sxs-lookup"><span data-stu-id="23409-163">This solution assumes a [Consumption plan][plan-c], in which compute power is automatically allocated to the functions when required.</span></span> <span data-ttu-id="23409-164">Vous payez uniquement lorsque vos fonctions sont en cours d’exécution.</span><span class="sxs-lookup"><span data-stu-id="23409-164">You pay only when your functions are running.</span></span> <span data-ttu-id="23409-165">Une autre option consiste à utiliser un [Azure App Service] [ plan-aas] plan, ce qui vous permet de mettre à l’échelle les niveaux pour allouer une quantité différente de ressources.</span><span class="sxs-lookup"><span data-stu-id="23409-165">Another option is to use an [Azure App Service][plan-aas] plan, which allows you to scale between tiers to allocate a different amount of resources.</span></span>

<span data-ttu-id="23409-166">Avec Azure Cosmos DB, la clé est de distribuer votre charge de travail à peu près uniformément entre un nombre suffisant de [clés de partition][keys].</span><span class="sxs-lookup"><span data-stu-id="23409-166">With Azure Cosmos DB, the key is to distribute your workload roughly evenly among a sufficiently large number of [partition keys][keys].</span></span> <span data-ttu-id="23409-167">Il n’existe aucune limite à la quantité totale de données qu’un conteneur peut stocker ou à la quantité totale de [débit] [ throughput] un conteneur pouvant prendre en charge.</span><span class="sxs-lookup"><span data-stu-id="23409-167">There's no limit to the total amount of data that a container can store or to the total amount of [throughput][throughput] that a container can support.</span></span>

### <a name="management-and-logging"></a><span data-ttu-id="23409-168">Gestion et la journalisation</span><span class="sxs-lookup"><span data-stu-id="23409-168">Management and logging</span></span>

<span data-ttu-id="23409-169">Cette solution utilise [Application Insights] [ aai] pour collecter des informations de journalisation et de performances.</span><span class="sxs-lookup"><span data-stu-id="23409-169">This solution uses [Application Insights][aai] to collect performance and logging information.</span></span> <span data-ttu-id="23409-170">Une instance d’Application Insights est créée avec le déploiement dans le même groupe de ressources que les autres services nécessaires pour ce déploiement.</span><span class="sxs-lookup"><span data-stu-id="23409-170">An instance of Application Insights is created with the deployment in the same resource group as the other services needed for this deployment.</span></span>

<span data-ttu-id="23409-171">Pour afficher les journaux générés par la solution :</span><span class="sxs-lookup"><span data-stu-id="23409-171">To view the logs generated by the solution:</span></span>

1. <span data-ttu-id="23409-172">Accédez à [Azure portal] [ portal] et accédez au groupe de ressources créé pour le déploiement.</span><span class="sxs-lookup"><span data-stu-id="23409-172">Go to [Azure portal][portal] and navigate to the resource group created for the deployment.</span></span>

2. <span data-ttu-id="23409-173">Cliquez sur le **Application Insights** instance.</span><span class="sxs-lookup"><span data-stu-id="23409-173">Click the **Application Insights** instance.</span></span>

3. <span data-ttu-id="23409-174">À partir de la **Application Insights** section, accédez à **examiner\\recherche** et rechercher les données.</span><span class="sxs-lookup"><span data-stu-id="23409-174">From the **Application Insights** section, navigate to **Investigate\\Search** and search the data.</span></span>

### <a name="security"></a><span data-ttu-id="23409-175">Sécurité</span><span class="sxs-lookup"><span data-stu-id="23409-175">Security</span></span>

<span data-ttu-id="23409-176">Azure Cosmos DB utilise une connexion sécurisée et la signature d’accès partagé via le C\# SDK fourni par Microsoft.</span><span class="sxs-lookup"><span data-stu-id="23409-176">Azure Cosmos DB uses a secured connection and shared access signature through the C\# SDK provided by Microsoft.</span></span> <span data-ttu-id="23409-177">Il n’existe aucun autres surfaces d’exposition exposés en externe.</span><span class="sxs-lookup"><span data-stu-id="23409-177">There are no other externally facing surface areas.</span></span> <span data-ttu-id="23409-178">En savoir plus sur la sécurité [meilleures pratiques] [ db-practices] pour Azure Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="23409-178">Learn more about security [best practices][db-practices] for Azure Cosmos DB.</span></span>

## <a name="pricing"></a><span data-ttu-id="23409-179">Tarifs</span><span class="sxs-lookup"><span data-stu-id="23409-179">Pricing</span></span>

<span data-ttu-id="23409-180">Le coût quotidien estimé pour conserver ce déploiement disponible est d’environ \$20 US sans données circulant dans le système.</span><span class="sxs-lookup"><span data-stu-id="23409-180">The estimated daily cost to keep this deployment available is approximately \$20 U.S. with no data moving through the system.</span></span>

<span data-ttu-id="23409-181">Azure Cosmos DB est puissante, mais entraîne le plus grand [coût] [ db-cost] dans ce déploiement.</span><span class="sxs-lookup"><span data-stu-id="23409-181">Azure Cosmos DB is powerful but incurs the greatest [cost][db-cost] in this deployment.</span></span> <span data-ttu-id="23409-182">Vous pouvez utiliser une autre solution de stockage par la refactorisation du code Azure Functions fourni.</span><span class="sxs-lookup"><span data-stu-id="23409-182">You can use another storage solution by refactoring the Azure Functions code provided.</span></span>

<span data-ttu-id="23409-183">La tarification pour Azure Functions peut varier selon le [plan] [ function-plan] elle s’exécute dans.</span><span class="sxs-lookup"><span data-stu-id="23409-183">Pricing for Azure Functions varies depending on the [plan][function-plan] it runs in.</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="23409-184">Déployez le scénario</span><span class="sxs-lookup"><span data-stu-id="23409-184">Deploy the scenario</span></span>

> [!NOTE]
> <span data-ttu-id="23409-185">Vous devez disposer d’un compte Azure existant.</span><span class="sxs-lookup"><span data-stu-id="23409-185">You must have an existing Azure account.</span></span> <span data-ttu-id="23409-186">Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit][free] avant de commencer.</span><span class="sxs-lookup"><span data-stu-id="23409-186">If you don't have an Azure subscription, create a [free account][free] before you begin.</span></span>

<span data-ttu-id="23409-187">Tout le code pour ce scénario est disponible dans le [GitHub] [ github] référentiel.</span><span class="sxs-lookup"><span data-stu-id="23409-187">All the code for this scenario is available in the [GitHub][github] repository.</span></span> <span data-ttu-id="23409-188">Ce référentiel contient le code source utilisé pour générer l’application de générateur qui alimente le pipeline pour cette démonstration.</span><span class="sxs-lookup"><span data-stu-id="23409-188">This repository contains the source code used to build the generator application that feeds the pipeline for this demo.</span></span>

[architecture]: ./media/mass-ingestion-newsfeeds-architecture.png
[aai]: /azure/azure-monitor/app/app-insights-overview
[aas]: https://azure.microsoft.com/try/app-service/
[acs]: https://azure.microsoft.com/services/cognitive-services/directory/
[collection]: /rest/api/cosmos-db/collections
[compare]: /azure/azure-functions/functions-compare-logic-apps-ms-flow-webjobs#compare-azure-functions-and-azure-logic-apps
[cosmos-db]: /azure/cosmos-db/introduction
[db-cost]: https://azure.microsoft.com/pricing/details/cosmos-db/
[db-practices]: /azure/cosmos-db/database-security
[db-collection]: /azure/cosmos-db/databases-containers-items
[english]: https://www.nasa.gov/rss/dyn/breaking_news.rss
[face]: /azure/cognitive-services/face/overview
[free]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[functions]: /azure/azure-functions/functions-overview
[function-plan]: /azure/azure-functions/functions-scale
[german]: http://www.bamf.de/SiteGlobals/Functions/RSS/DE/Feed/RSSNewsfeed_Meldungen
[github]: https://github.com/Azure/cognitive-services
[keys]: /azure/cosmos-db/partition-data
[language]: /azure/cognitive-services/translator/reference/v3-0-languages
[logic-app]: /azure/logic-apps/logic-apps-overview
[queues-topics]: /azure/service-bus-messaging/service-bus-queues-topics-subscriptions
[partial]: https://feedback.azure.com/forums/263030-azure-cosmos-db/suggestions/6693091-be-able-to-do-partial-updates-on-document
[plan]: /azure/azure-functions/functions-scale
[plan-aas]: /azure/azure-functions/functions-scale#app-service-plan
[plan-c]: /azure/azure-functions/functions-scale#consumption-plan
[portal]: http://portal.azure.com
[redlock]: https://redis.io/topics/distlock
[russian]: http://government.ru/all/rss/
[service-bus]: /azure/service-bus-messaging/
[storage]: /azure/storage/common/storage-account-overview 
[throughput]: /azure/cosmos-db/scaling-throughput
[topics]: /azure/service-bus-messaging/service-bus-dotnet-how-to-use-topics-subscriptions
[text-analytics]: /azure/cognitive-services/text-analytics/
[translate-text]: /azure/cognitive-services/translator/translator-info-overview
[vision]: /azure/cognitive-services/computer-vision/home
