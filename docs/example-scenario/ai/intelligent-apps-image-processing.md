---
title: Classification d’images pour les déclarations de sinistre sur Azure
description: Scénario éprouvé pour générer le traitement d’images dans vos applications Azure.
author: david-stanford
ms.date: 07/05/2018
ms.openlocfilehash: 361a88234fd9ed918ab7664893f86666b4328b8c
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060827"
---
# <a name="image-classification-for-insurance-claims-on-azure"></a><span data-ttu-id="080a3-103">Classification d’images pour les déclarations de sinistre sur Azure</span><span class="sxs-lookup"><span data-stu-id="080a3-103">Image classification for insurance claims on Azure</span></span>

<span data-ttu-id="080a3-104">Cet exemple de scénario s’applique aux entreprises qui doivent traiter des images.</span><span class="sxs-lookup"><span data-stu-id="080a3-104">This example scenario is applicable for businesses that need to process images.</span></span>

<span data-ttu-id="080a3-105">Applications potentielles : classification d’images pour un site web de mode, analyse de texte et d’images pour les déclarations de sinistre ou compréhension des données de télémétrie issues des captures d’écran de jeux.</span><span class="sxs-lookup"><span data-stu-id="080a3-105">Potential applications include classifying images for a fashion website, analyzing text and images for insurance claims, or understanding telemetry data from game screenshots.</span></span> <span data-ttu-id="080a3-106">Traditionnellement, les entreprises devaient développer une expertise en matière de modèles Machine Learning, effectuer l’apprentissage des modèles et enfin exécuter les images via leur processus personnalisé pour obtenir les données des images.</span><span class="sxs-lookup"><span data-stu-id="080a3-106">Traditionally, companies would need to develop expertise in machine learning models, train the models, and finally run the images through their custom process to get the data out of the images.</span></span>

<span data-ttu-id="080a3-107">En utilisant des services Azure tels que l’API Vision par ordinateur et Azure Functions, les sociétés peuvent éliminer la nécessité de gérer des serveurs individuels, tout en réduisant les coûts et en tirant parti de l’expertise que Microsoft a déjà développée en matière de traitement d’images avec Cognitives Services.</span><span class="sxs-lookup"><span data-stu-id="080a3-107">By using Azure services such as the Computer Vision API and Azure Functions, companies can eliminate the need to manage individual servers, while reducing costs and leveraging the expertise that Microsoft has already developed around processing images with Cognitive services.</span></span> <span data-ttu-id="080a3-108">Ce scénario concerne plus spécialement un scénario de traitement d’images.</span><span class="sxs-lookup"><span data-stu-id="080a3-108">This scenario specifically addresses an image processing scenario.</span></span> <span data-ttu-id="080a3-109">Si vos besoins en termes d’intelligence artificielle sont variés, pensez à la suite complète de [Cognitive Services][cognitive-docs].</span><span class="sxs-lookup"><span data-stu-id="080a3-109">If you have different AI needs, consider the full suite of [Cognitive Services][cognitive-docs].</span></span>

## <a name="related-use-cases"></a><span data-ttu-id="080a3-110">Cas d’usage connexes</span><span class="sxs-lookup"><span data-stu-id="080a3-110">Related use cases</span></span>

<span data-ttu-id="080a3-111">Pensez à ce scénario pour les cas d’usage suivants :</span><span class="sxs-lookup"><span data-stu-id="080a3-111">Consider this scenario for the following use cases:</span></span>

* <span data-ttu-id="080a3-112">Classer des images sur un site web de mode</span><span class="sxs-lookup"><span data-stu-id="080a3-112">Classify images on a fashion website.</span></span>
* <span data-ttu-id="080a3-113">Classer des données de télémétrie provenant de captures d’écran de jeux</span><span class="sxs-lookup"><span data-stu-id="080a3-113">Classify telemetry data from screenshots of games.</span></span>

## <a name="architecture"></a><span data-ttu-id="080a3-114">Architecture</span><span class="sxs-lookup"><span data-stu-id="080a3-114">Architecture</span></span>

![Architecture des applications intelligentes : Vision par ordinateur][architecture-computer-vision]

<span data-ttu-id="080a3-116">Ce scénario couvre les composants principaux d’une application web ou mobile.</span><span class="sxs-lookup"><span data-stu-id="080a3-116">This scenario covers the back-end components of a web or mobile application.</span></span> <span data-ttu-id="080a3-117">Les données circulent dans le scénario comme suit :</span><span class="sxs-lookup"><span data-stu-id="080a3-117">Data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="080a3-118">Azure Functions fait office de couche API.</span><span class="sxs-lookup"><span data-stu-id="080a3-118">Azure Functions acts as the API layer.</span></span> <span data-ttu-id="080a3-119">Ces API permettent à l’application de charger des images et de récupérer des données à partir de Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="080a3-119">These APIs enable the application to upload images and retrieve data from Cosmos DB.</span></span>

2. <span data-ttu-id="080a3-120">Quand une image est chargée via un appel d’API, elle est stockée dans le stockage Blob.</span><span class="sxs-lookup"><span data-stu-id="080a3-120">When an image is uploaded via an API call, it's stored in Blob storage.</span></span>

3. <span data-ttu-id="080a3-121">L’ajout de nouveaux fichiers dans le stockage Blob déclenche l’envoi d’une notification EventGrid à une instance Azure Functions.</span><span class="sxs-lookup"><span data-stu-id="080a3-121">Adding new files to Blob storage triggers an EventGrid notification to be sent to an Azure Function.</span></span>

4. <span data-ttu-id="080a3-122">Azure Functions envoie un lien vers le fichier qui vient d’être chargé à l’API Vision par ordinateur à des fins d’analyse.</span><span class="sxs-lookup"><span data-stu-id="080a3-122">Azure Functions sends a link to the newly uploaded file to the Computer Vision API to analyze.</span></span>

5. <span data-ttu-id="080a3-123">Une fois que les données ont été retournées à partir de l’API Vision par ordinateur, Azure Functions crée une entrée dans Cosmos DB pour conserver les résultats de l’analyse en même temps que les métadonnées d’images.</span><span class="sxs-lookup"><span data-stu-id="080a3-123">Once the data has been returned from the Computer Vision API, Azure Functions makes an entry in Cosmos DB to persist the results of the analysis alongside the image metadata.</span></span>

### <a name="components"></a><span data-ttu-id="080a3-124">Composants</span><span class="sxs-lookup"><span data-stu-id="080a3-124">Components</span></span>

* <span data-ttu-id="080a3-125">[L’API Vision par ordinateur][computer-vision-docs] fait partie de la suite Cognitive Services et est utilisée pour récupérer des informations sur chaque image.</span><span class="sxs-lookup"><span data-stu-id="080a3-125">[Computer Vision API][computer-vision-docs] is part of the Cognitive Services suite and is used to retrieve information about each image.</span></span>

* <span data-ttu-id="080a3-126">[Azure Functions][functions-docs] fournit l’API principale pour l’application web et assure le traitement des événements correspondant aux images chargées.</span><span class="sxs-lookup"><span data-stu-id="080a3-126">[Azure Functions][functions-docs] provides the backend API for the web application, as well as the event processing for uploaded images.</span></span>

* <span data-ttu-id="080a3-127">[Event Grid][eventgrid-docs] déclenche un événement quand une nouvelle image est chargée dans le stockage Blob.</span><span class="sxs-lookup"><span data-stu-id="080a3-127">[Event Grid][eventgrid-docs] triggers an event when a new image is uploaded to blob storage.</span></span> <span data-ttu-id="080a3-128">L’image est ensuite traitée avec Azure Functions.</span><span class="sxs-lookup"><span data-stu-id="080a3-128">The image is then processed with Azure functions.</span></span>

* <span data-ttu-id="080a3-129">[Stockage Blob][storage-docs] stocke tous les fichiers image qui sont chargés dans l’application web, ainsi que les fichiers statiques qui sont consommés par l’application web.</span><span class="sxs-lookup"><span data-stu-id="080a3-129">[Blob Storage][storage-docs] stores all of the image files that are uploaded into the web application, as well any static files that the web application consumes.</span></span>

* <span data-ttu-id="080a3-130">[Cosmos DB][cosmos-docs] stocke des métadonnées sur chaque image chargée, notamment les résultats du traitement de l’API Vision par ordinateur.</span><span class="sxs-lookup"><span data-stu-id="080a3-130">[Cosmos DB][cosmos-docs] stores metadata about each image that is uploaded, including the results of the processing from Computer Vision API.</span></span>

## <a name="alternatives"></a><span data-ttu-id="080a3-131">Autres solutions</span><span class="sxs-lookup"><span data-stu-id="080a3-131">Alternatives</span></span>

* <span data-ttu-id="080a3-132">[Service Vision personnalisée][custom-vision-docs].</span><span class="sxs-lookup"><span data-stu-id="080a3-132">[Custom Vision Service][custom-vision-docs].</span></span> <span data-ttu-id="080a3-133">L’API Vision par ordinateur retourne un ensemble de [catégories basées sur la taxonomie][cv-categories].</span><span class="sxs-lookup"><span data-stu-id="080a3-133">The Computer Vision API returns a set of [taxonomy-based categories][cv-categories].</span></span> <span data-ttu-id="080a3-134">Si vous devez traiter des informations qui ne sont pas retournées par l’API Vision par ordinateur, pensez au service Vision personnalisée, qui vous permet de créer des classifieurs d’images personnalisés.</span><span class="sxs-lookup"><span data-stu-id="080a3-134">If you need to process information that isn't returned by the Computer Vision API, consider the Custom Vision Service, which lets you build custom image classifiers.</span></span>

* <span data-ttu-id="080a3-135">[Recherche Azure][azure-search-docs].</span><span class="sxs-lookup"><span data-stu-id="080a3-135">[Azure Search][azure-search-docs].</span></span> <span data-ttu-id="080a3-136">Si votre cas d’usage implique l’interrogation de métadonnées pour rechercher des images qui répondent à des critères spécifiques, pensez à utiliser Recherche Azure.</span><span class="sxs-lookup"><span data-stu-id="080a3-136">If your use case involves querying the metadata to find images that meet specific criteria, consider using Azure Search.</span></span> <span data-ttu-id="080a3-137">Actuellement en préversion, la [recherche cognitive][cognitive-search] intègre parfaitement ce flux de travail.</span><span class="sxs-lookup"><span data-stu-id="080a3-137">Currently in preview, [Cognitive search][cognitive-search] seamlessly integrates this workflow.</span></span>

## <a name="considerations"></a><span data-ttu-id="080a3-138">Considérations</span><span class="sxs-lookup"><span data-stu-id="080a3-138">Considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="080a3-139">Extensibilité</span><span class="sxs-lookup"><span data-stu-id="080a3-139">Scalability</span></span>

<span data-ttu-id="080a3-140">Pour leur majeure partie, les composants de ce scénario sont des services gérés qui sont mis automatiquement à l’échelle.</span><span class="sxs-lookup"><span data-stu-id="080a3-140">For the most part all of the components of this scenario are managed services that will automatically scale.</span></span> <span data-ttu-id="080a3-141">Deux exceptions notables : la solution Azure Functions est limitée à un maximum de 200 instances.</span><span class="sxs-lookup"><span data-stu-id="080a3-141">A couple notable exceptions: Azure Functions has a limit of a maximum of 200 instances.</span></span> <span data-ttu-id="080a3-142">Si vous devez procéder à une mise à l’échelle et atteindre un nombre supérieur d’instances, pensez à utiliser plusieurs régions ou plans d’application.</span><span class="sxs-lookup"><span data-stu-id="080a3-142">If you need to scale beyond, consider multiple regions or app plans.</span></span>

<span data-ttu-id="080a3-143">Cosmos DB n’effectue pas de mise à l’échelle automatique en termes d’unités de requête approvisionnées (RU).</span><span class="sxs-lookup"><span data-stu-id="080a3-143">Cosmos DB doesn’t auto-scale in terms of provisioned request units (RUs).</span></span>  <span data-ttu-id="080a3-144">Pour obtenir des conseils sur l’estimation de vos besoins, consultez la section relative aux [unités de requête][request-units] dans notre documentation.</span><span class="sxs-lookup"><span data-stu-id="080a3-144">For guidance on estimating your requirements see [request units][request-units] in our documentation.</span></span> <span data-ttu-id="080a3-145">Pour tirer pleinement parti de la mise à l’échelle dans Cosmos DB, vous devez également jeter un coup d’œil sur les [clés de partition][partition-key].</span><span class="sxs-lookup"><span data-stu-id="080a3-145">To fully take advantage of the scaling in Cosmos DB you should also take a look at [partition keys][partition-key].</span></span>

<span data-ttu-id="080a3-146">Les bases de données NoSQL échangent fréquemment la cohérence (au sens du théorème CAP) contre la disponibilité, l’extensibilité et la partition.</span><span class="sxs-lookup"><span data-stu-id="080a3-146">NoSQL databases frequently trade consistency (in the sense of the CAP theorem) for availability, scalability and partition.</span></span>  <span data-ttu-id="080a3-147">Toutefois, dans le cas des modèles de données clé-valeur utilisés dans ce scénario, la cohérence des transactions est rarement nécessaire, car la plupart des opérations sont par définition atomiques.</span><span class="sxs-lookup"><span data-stu-id="080a3-147">However, in the case of key-value data models which is used in this scenario, transaction consistency is rarely needed as most operations are by definition atomic.</span></span> <span data-ttu-id="080a3-148">Des conseils supplémentaires pour [choisir le magasin de données approprié](../../guide/technology-choices/data-store-overview.md) sont disponibles dans le Centre des architectures.</span><span class="sxs-lookup"><span data-stu-id="080a3-148">Additional guidance to [Choose the right data store](../../guide/technology-choices/data-store-overview.md) is available in the architecture center.</span></span>

<span data-ttu-id="080a3-149">Pour obtenir des conseils d’ordre général sur la conception de solutions évolutives, consultez la [liste de contrôle de l’extensibilité][scalability] dans le Centre des architectures Azure.</span><span class="sxs-lookup"><span data-stu-id="080a3-149">For general guidance on designing scalable solutions, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="080a3-150">Sécurité</span><span class="sxs-lookup"><span data-stu-id="080a3-150">Security</span></span>

<span data-ttu-id="080a3-151">Des instances [Managed Service Identity][msi] (MSI) permettent à d’autres ressources internes d’accéder à votre compte et de les affecter à vos instances Azure Functions.</span><span class="sxs-lookup"><span data-stu-id="080a3-151">[Managed service identities][msi] (MSI) are used to provide access to other resources internal to your account and then assigned to your Azure Functions.</span></span> <span data-ttu-id="080a3-152">Autorisez uniquement l’accès aux ressources requises dans ces identités pour vous assurer qu’aucun élément supplémentaire n’est exposé à vos fonctions (et potentiellement à vos clients).</span><span class="sxs-lookup"><span data-stu-id="080a3-152">Only allow access to the requisite resources in those identities to ensure that nothing extra is exposed to your functions (and potentially to your customers).</span></span>  

<span data-ttu-id="080a3-153">Pour obtenir des conseils d’ordre général sur la conception de solutions sécurisées, consultez la [documentation sur la sécurité Azure][security].</span><span class="sxs-lookup"><span data-stu-id="080a3-153">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="080a3-154">Résilience</span><span class="sxs-lookup"><span data-stu-id="080a3-154">Resiliency</span></span>

<span data-ttu-id="080a3-155">Comme tous les composants de ce scénario sont gérés, à un niveau régional, ils sont résilients automatiquement.</span><span class="sxs-lookup"><span data-stu-id="080a3-155">All of the components in this scenario are managed, so at a regional level they are all resilient automatically.</span></span>

<span data-ttu-id="080a3-156">Pour obtenir des conseils d’ordre général sur la conception de solutions résilientes, consultez l’article [Conception d’applications résilientes pour Azure][resiliency].</span><span class="sxs-lookup"><span data-stu-id="080a3-156">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="pricing"></a><span data-ttu-id="080a3-157">Tarifs</span><span class="sxs-lookup"><span data-stu-id="080a3-157">Pricing</span></span>

<span data-ttu-id="080a3-158">Pour explorer le coût d’exécution de ce scénario, tous les services sont préconfigurés dans le calculateur de coûts.</span><span class="sxs-lookup"><span data-stu-id="080a3-158">To explore the cost of running this scenario, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="080a3-159">Pour pouvoir observer l’évolution de la tarification pour votre cas d’usage particulier, modifiez les variables appropriées en fonction du trafic que vous escomptez.</span><span class="sxs-lookup"><span data-stu-id="080a3-159">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="080a3-160">Nous proposons trois exemples de profils de coût basés sur la quantité de trafic (nous supposons que toutes les images ont une taille de 100 Ko) :</span><span class="sxs-lookup"><span data-stu-id="080a3-160">We have provided three sample cost profiles based on amount of traffic (we assume all images are 100kb in size):</span></span>

* <span data-ttu-id="080a3-161">[Petit][pricing] : ce profil correspond au traitement de &lt; 5 000 images par mois.</span><span class="sxs-lookup"><span data-stu-id="080a3-161">[Small][pricing]: this correlates to processing &lt; 5000 images a month.</span></span>
* <span data-ttu-id="080a3-162">[Moyen][medium-pricing] : ce profil correspond au traitement de 500 000 images par mois.</span><span class="sxs-lookup"><span data-stu-id="080a3-162">[Medium][medium-pricing]: this correlates to processing 500,000 images a month.</span></span>
* <span data-ttu-id="080a3-163">[Grand][large-pricing] : ce profil correspond au traitement de 50 millions d’images par mois.</span><span class="sxs-lookup"><span data-stu-id="080a3-163">[Large][large-pricing]: this correlates to processing 50 million images a month.</span></span>

## <a name="related-resources"></a><span data-ttu-id="080a3-164">Ressources associées</span><span class="sxs-lookup"><span data-stu-id="080a3-164">Related Resources</span></span>

<span data-ttu-id="080a3-165">Pour un parcours d’apprentissage interactif de ce scénario, consultez l’article [Build a serverless web app in Azure][serverless] (Créer une application web serverless dans Azure).</span><span class="sxs-lookup"><span data-stu-id="080a3-165">For a guided learning path of this scenario, see [Build a serverless web app in Azure][serverless].</span></span>  

<span data-ttu-id="080a3-166">Avant de passer à un environnement de production, passez en revue les [meilleures pratiques][functions-best-practices] relatives à Azure Functions.</span><span class="sxs-lookup"><span data-stu-id="080a3-166">Before putting this in a production environment, review the Azure Functions [best practices][functions-best-practices].</span></span>

<!-- links -->
[pricing]: https://azure.com/e/f9b59d238b43423683db73f4a31dc380
[medium-pricing]: https://azure.com/e/7c7fc474db344b87aae93bc29ae27108
[large-pricing]: https://azure.com/e/cbadbca30f8640d6a061f8457a74ba7d
[functions-docs]: /azure/azure-functions/
[computer-vision-docs]: /azure/cognitive-services/computer-vision/home
[storage-docs]: /azure/storage/
[azure-search-docs]: /azure/search/
[cognitive-search]: /azure/search/cognitive-search-concept-intro
[architecture-computer-vision]: ./media/architecture-computer-vision.png
[serverless]: /azure/functions/tutorial-static-website-serverless-api-with-database
[cosmos-docs]: /azure/cosmos-db/
[eventgrid-docs]: /azure/event-grid/
[cognitive-docs]: /azure/#pivot=products&panel=ai
[custom-vision-docs]: /azure/cognitive-services/Custom-Vision-Service/home
[cv-categories]: /azure/cognitive-services/computer-vision/home#the-86-category-concept
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[functions-best-practices]: /azure/azure-functions/functions-best-practices
[msi]: /azure/app-service/app-service-managed-service-identity
[request-units]: /azure/cosmos-db/request-units
[partition-key]: /azure/cosmos-db/partition-data