---
title: Style d’architecture Web-File d’attente-Worker
titleSuffix: Azure Application Architecture Guide
description: Décrit les avantages, les inconvénients et les bonnes pratiques des architectures Web-File d’attente-Worker sur Azure.
author: MikeWasson
ms.date: 08/30/2018
ms.custom: seojan19
ms.openlocfilehash: 0b478344c4b64808d30156bd563917d9d8d7ec30
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/08/2019
ms.locfileid: "54113278"
---
# <a name="web-queue-worker-architecture-style"></a><span data-ttu-id="6e57a-103">Style d’architecture Web-File d’attente-Worker</span><span class="sxs-lookup"><span data-stu-id="6e57a-103">Web-Queue-Worker architecture style</span></span>

<span data-ttu-id="6e57a-104">Les principaux composants de cette architecture sont un **serveur web frontal** qui traite les requêtes des clients et un **Worker** qui s’occupe des tâches gourmandes en ressources, des flux de travail à exécution longue ou des programmes de traitement par lots.</span><span class="sxs-lookup"><span data-stu-id="6e57a-104">The core components of this architecture are a **web front end** that serves client requests, and a **worker** that performs resource-intensive tasks, long-running workflows, or batch jobs.</span></span>  <span data-ttu-id="6e57a-105">Le serveur web frontal communique avec le Worker via une **file d’attente de messages**.</span><span class="sxs-lookup"><span data-stu-id="6e57a-105">The web front end communicates with the worker through a **message queue**.</span></span>

![Diagramme logique du style d’architecture Web-File d’attente-Worker](./images/web-queue-worker-logical.svg)

<span data-ttu-id="6e57a-107">Les autres composants généralement intégrés dans cette architecture sont :</span><span class="sxs-lookup"><span data-stu-id="6e57a-107">Other components that are commonly incorporated into this architecture include:</span></span>

- <span data-ttu-id="6e57a-108">Une ou plusieurs bases de données.</span><span class="sxs-lookup"><span data-stu-id="6e57a-108">One or more databases.</span></span>
- <span data-ttu-id="6e57a-109">Un cache pour stocker des valeurs à partir de la base de données pour des lectures rapides.</span><span class="sxs-lookup"><span data-stu-id="6e57a-109">A cache to store values from the database for quick reads.</span></span>
- <span data-ttu-id="6e57a-110">Un réseau de distribution de contenu (CDN) pour traiter du contenu statique.</span><span class="sxs-lookup"><span data-stu-id="6e57a-110">CDN to serve static content</span></span>
- <span data-ttu-id="6e57a-111">Des services à distance, tels qu’un service de messagerie ou SMS.</span><span class="sxs-lookup"><span data-stu-id="6e57a-111">Remote services, such as email or SMS service.</span></span> <span data-ttu-id="6e57a-112">Souvent, ils sont fournis par des tiers.</span><span class="sxs-lookup"><span data-stu-id="6e57a-112">Often these are provided by third parties.</span></span>
- <span data-ttu-id="6e57a-113">Un fournisseur d’identité pour l’authentification.</span><span class="sxs-lookup"><span data-stu-id="6e57a-113">Identity provider for authentication.</span></span>

<span data-ttu-id="6e57a-114">Le serveur web et le Worker sont sans état.</span><span class="sxs-lookup"><span data-stu-id="6e57a-114">The web and worker are both stateless.</span></span> <span data-ttu-id="6e57a-115">L’état de session peut être stocké dans un cache distribué.</span><span class="sxs-lookup"><span data-stu-id="6e57a-115">Session state can be stored in a distributed cache.</span></span> <span data-ttu-id="6e57a-116">N’importe quel travail à exécution longue est traité de façon asynchrone par le Worker.</span><span class="sxs-lookup"><span data-stu-id="6e57a-116">Any long-running work is done asynchronously by the worker.</span></span> <span data-ttu-id="6e57a-117">Le Worker peut être déclenché par des messages dans la file d’attente ou s’exécuter selon une planification pour le traitement par lots.</span><span class="sxs-lookup"><span data-stu-id="6e57a-117">The worker can be triggered by messages on the queue, or run on a schedule for batch processing.</span></span> <span data-ttu-id="6e57a-118">Le Worker est un composant facultatif.</span><span class="sxs-lookup"><span data-stu-id="6e57a-118">The worker is an optional component.</span></span> <span data-ttu-id="6e57a-119">S’il n’y a aucune opération à exécution longue, le Worker peut être omis.</span><span class="sxs-lookup"><span data-stu-id="6e57a-119">If there are no long-running operations, the worker can be omitted.</span></span>

<span data-ttu-id="6e57a-120">Le serveur frontal peut être une API web.</span><span class="sxs-lookup"><span data-stu-id="6e57a-120">The front end might consist of a web API.</span></span> <span data-ttu-id="6e57a-121">Du côté client, l’API web peut être utilisée par une application de page unique qui effectue les appels AJAX ou par une application cliente native.</span><span class="sxs-lookup"><span data-stu-id="6e57a-121">On the client side, the web API can be consumed by a single-page application that makes AJAX calls, or by a native client application.</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="6e57a-122">Quand utiliser cette architecture</span><span class="sxs-lookup"><span data-stu-id="6e57a-122">When to use this architecture</span></span>

<span data-ttu-id="6e57a-123">L’architecture Web-File d’attente-Worker est généralement implémentée à l’aide des services de calcul gérés d’Azure App Service ou d’Azure Cloud Services.</span><span class="sxs-lookup"><span data-stu-id="6e57a-123">The Web-Queue-Worker architecture is typically implemented using managed compute services, either Azure App Service or Azure Cloud Services.</span></span>

<span data-ttu-id="6e57a-124">Envisagez ce style d’architecture pour :</span><span class="sxs-lookup"><span data-stu-id="6e57a-124">Consider this architecture style for:</span></span>

- <span data-ttu-id="6e57a-125">Des applications avec un domaine relativement simple.</span><span class="sxs-lookup"><span data-stu-id="6e57a-125">Applications with a relatively simple domain.</span></span>
- <span data-ttu-id="6e57a-126">Des applications ayant des flux de travail à exécution longue ou des opérations par lots.</span><span class="sxs-lookup"><span data-stu-id="6e57a-126">Applications with some long-running workflows or batch operations.</span></span>
- <span data-ttu-id="6e57a-127">Lorsque vous souhaitez utiliser des services gérés plutôt que l’infrastructure as a service (IaaS).</span><span class="sxs-lookup"><span data-stu-id="6e57a-127">When you want to use managed services, rather than infrastructure as a service (IaaS).</span></span>

## <a name="benefits"></a><span data-ttu-id="6e57a-128">Avantages</span><span class="sxs-lookup"><span data-stu-id="6e57a-128">Benefits</span></span>

- <span data-ttu-id="6e57a-129">Architecture relativement simple, facile à comprendre.</span><span class="sxs-lookup"><span data-stu-id="6e57a-129">Relatively simple architecture that is easy to understand.</span></span>
- <span data-ttu-id="6e57a-130">Gestion et déploiement faciles.</span><span class="sxs-lookup"><span data-stu-id="6e57a-130">Easy to deploy and manage.</span></span>
- <span data-ttu-id="6e57a-131">Séparation nette des problèmes.</span><span class="sxs-lookup"><span data-stu-id="6e57a-131">Clear separation of concerns.</span></span>
- <span data-ttu-id="6e57a-132">Le serveur frontal est dissocié du Worker à l’aide d’une messagerie asynchrone.</span><span class="sxs-lookup"><span data-stu-id="6e57a-132">The front end is decoupled from the worker using asynchronous messaging.</span></span>
- <span data-ttu-id="6e57a-133">Le serveur frontal et le Worker peuvent être mis à l’échelle indépendamment.</span><span class="sxs-lookup"><span data-stu-id="6e57a-133">The front end and the worker can be scaled independently.</span></span>

## <a name="challenges"></a><span data-ttu-id="6e57a-134">Défis</span><span class="sxs-lookup"><span data-stu-id="6e57a-134">Challenges</span></span>

- <span data-ttu-id="6e57a-135">Sans une conception minutieuse, le serveur frontal et le Worker peuvent devenir des composants monolithiques volumineux, difficiles à gérer et à mettre à jour.</span><span class="sxs-lookup"><span data-stu-id="6e57a-135">Without careful design, the front end and the worker can become large, monolithic components that are difficult to maintain and update.</span></span>
- <span data-ttu-id="6e57a-136">Des dépendances masquées peuvent être présentes si le serveur frontal et le Worker partagent des schémas de données ou des modules de code.</span><span class="sxs-lookup"><span data-stu-id="6e57a-136">There may be hidden dependencies, if the front end and worker share data schemas or code modules.</span></span>

## <a name="best-practices"></a><span data-ttu-id="6e57a-137">Bonnes pratiques</span><span class="sxs-lookup"><span data-stu-id="6e57a-137">Best practices</span></span>

- <span data-ttu-id="6e57a-138">Exposez une API bien conçue au client.</span><span class="sxs-lookup"><span data-stu-id="6e57a-138">Expose a well-designed API to the client.</span></span> <span data-ttu-id="6e57a-139">Consultez [API design][api-design] (Conception d’API).</span><span class="sxs-lookup"><span data-stu-id="6e57a-139">See [API design best practices][api-design].</span></span>
- <span data-ttu-id="6e57a-140">Effectuez une mise à l’échelle automatique pour gérer les évolutions au niveau de la charge.</span><span class="sxs-lookup"><span data-stu-id="6e57a-140">Autoscale to handle changes in load.</span></span> <span data-ttu-id="6e57a-141">Consultez [Autoscaling][autoscaling] (Mise à l’échelle automatique).</span><span class="sxs-lookup"><span data-stu-id="6e57a-141">See [Autoscaling best practices][autoscaling].</span></span>
- <span data-ttu-id="6e57a-142">Mettez en cache les données semi-statiques.</span><span class="sxs-lookup"><span data-stu-id="6e57a-142">Cache semi-static data.</span></span> <span data-ttu-id="6e57a-143">Consultez [Caching][caching] (Mise en cache).</span><span class="sxs-lookup"><span data-stu-id="6e57a-143">See [Caching best practices][caching].</span></span>
- <span data-ttu-id="6e57a-144">Utilisez un réseau de distribution de contenu (CDN) pour héberger le contenu statique.</span><span class="sxs-lookup"><span data-stu-id="6e57a-144">Use a CDN to host static content.</span></span> <span data-ttu-id="6e57a-145">Consultez [Content Delivery Network][cdn] (Réseau de distribution de contenu).</span><span class="sxs-lookup"><span data-stu-id="6e57a-145">See [CDN best practices][cdn].</span></span>
- <span data-ttu-id="6e57a-146">Utilisez la persistance polyglotte lorsque cela est approprié.</span><span class="sxs-lookup"><span data-stu-id="6e57a-146">Use polyglot persistence when appropriate.</span></span> <span data-ttu-id="6e57a-147">Consultez [Use the best data store for the job][polyglot] (Utiliser le meilleur magasin de données pour le travail).</span><span class="sxs-lookup"><span data-stu-id="6e57a-147">See [Use the best data store for the job][polyglot].</span></span>
- <span data-ttu-id="6e57a-148">Partitionnez des données pour améliorer l’extensibilité, réduire la contention et optimiser les performances.</span><span class="sxs-lookup"><span data-stu-id="6e57a-148">Partition data to improve scalability, reduce contention, and optimize performance.</span></span> <span data-ttu-id="6e57a-149">Consultez [Data partitioning][data-partition] (Partitionnement des données).</span><span class="sxs-lookup"><span data-stu-id="6e57a-149">See [Data partitioning best practices][data-partition].</span></span>

## <a name="web-queue-worker-on-azure-app-service"></a><span data-ttu-id="6e57a-150">Web-File d’attente-Worker sur Azure App Service</span><span class="sxs-lookup"><span data-stu-id="6e57a-150">Web-Queue-Worker on Azure App Service</span></span>

<span data-ttu-id="6e57a-151">Cette section décrit une architecture Web-File d’attente-Worker recommandée qui utilise Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="6e57a-151">This section describes a recommended Web-Queue-Worker architecture that uses Azure App Service.</span></span>

![Diagramme physique du style d’architecture Web-File d’attente-Worker](./images/web-queue-worker-physical.png)

<span data-ttu-id="6e57a-153">Le serveur frontal est implémenté en tant qu’application web Azure App Service et le Worker est implémenté en tant que WebJob.</span><span class="sxs-lookup"><span data-stu-id="6e57a-153">The front end is implemented as an Azure App Service web app, and the worker is implemented as a WebJob.</span></span> <span data-ttu-id="6e57a-154">L’application web et WebJob sont associés à un plan App Service qui fournit les instances de machine virtuelle.</span><span class="sxs-lookup"><span data-stu-id="6e57a-154">The web app and the WebJob are both associated with an App Service plan that provides the VM instances.</span></span>

<span data-ttu-id="6e57a-155">Vous pouvez utiliser les files d’attente Azure Service Bus ou de stockage Azure pour la file d’attente de messages.</span><span class="sxs-lookup"><span data-stu-id="6e57a-155">You can use either Azure Service Bus or Azure Storage queues for the message queue.</span></span> <span data-ttu-id="6e57a-156">(Le schéma montre une file d’attente de stockage Azure.)</span><span class="sxs-lookup"><span data-stu-id="6e57a-156">(The diagram shows an Azure Storage queue.)</span></span>

<span data-ttu-id="6e57a-157">Cache Redis Azure stocke l’état de session et d’autres données qui nécessitent un accès à faible latence.</span><span class="sxs-lookup"><span data-stu-id="6e57a-157">Azure Redis Cache stores session state and other data that needs low latency access.</span></span>

<span data-ttu-id="6e57a-158">Azure CDN est utilisé pour mettre en cache le contenu statique comme des images, des CSS ou des HTML.</span><span class="sxs-lookup"><span data-stu-id="6e57a-158">Azure CDN is used to cache static content such as images, CSS, or HTML.</span></span>

<span data-ttu-id="6e57a-159">Pour le stockage, choisissez les technologies de stockage qui répondent le mieux aux besoins de l’application.</span><span class="sxs-lookup"><span data-stu-id="6e57a-159">For storage, choose the storage technologies that best fit the needs of the application.</span></span> <span data-ttu-id="6e57a-160">Vous pouvez utiliser plusieurs technologies de stockage (persistance polyglotte).</span><span class="sxs-lookup"><span data-stu-id="6e57a-160">You might use multiple storage technologies (polyglot persistence).</span></span> <span data-ttu-id="6e57a-161">Pour illustrer cette idée, le schéma montre Azure SQL Database et Azure Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="6e57a-161">To illustrate this idea, the diagram shows Azure SQL Database and Azure Cosmos DB.</span></span>

<span data-ttu-id="6e57a-162">Pour plus d’informations, consultez [Improve scalability in a web application][scalable-web-app] (Améliorer l’évolutivité d’une application web).</span><span class="sxs-lookup"><span data-stu-id="6e57a-162">For more details, see [App Service web application reference architecture][scalable-web-app].</span></span>

### <a name="additional-considerations"></a><span data-ttu-id="6e57a-163">Considérations supplémentaires</span><span class="sxs-lookup"><span data-stu-id="6e57a-163">Additional considerations</span></span>

- <span data-ttu-id="6e57a-164">Toutes les transactions n’ont pas à passer par la file d’attente et le Worker avant d’arriver au stockage.</span><span class="sxs-lookup"><span data-stu-id="6e57a-164">Not every transaction has to go through the queue and worker to storage.</span></span> <span data-ttu-id="6e57a-165">Le serveur web frontal peut directement effectuer des opérations de lecture/d’écriture simples.</span><span class="sxs-lookup"><span data-stu-id="6e57a-165">The web front end can perform simple read/write operations directly.</span></span> <span data-ttu-id="6e57a-166">Les Workers sont conçus pour les tâches gourmandes en ressources ou les flux de travail à exécution longue.</span><span class="sxs-lookup"><span data-stu-id="6e57a-166">Workers are designed for resource-intensive tasks or long-running workflows.</span></span> <span data-ttu-id="6e57a-167">Dans certains cas, aucun Worker ne vous sera nécessaire.</span><span class="sxs-lookup"><span data-stu-id="6e57a-167">In some cases, you might not need a worker at all.</span></span>

- <span data-ttu-id="6e57a-168">Utilisez la fonctionnalité de mise à l’échelle intégrée d’App Service pour augmenter la taille des instances de machine virtuelle.</span><span class="sxs-lookup"><span data-stu-id="6e57a-168">Use the built-in autoscale feature of App Service to scale out the number of VM instances.</span></span> <span data-ttu-id="6e57a-169">Si la charge sur l’application suit les modèles prévisibles, utilisez la mise à l’échelle automatique basée sur la planification.</span><span class="sxs-lookup"><span data-stu-id="6e57a-169">If the load on the application follows predictable patterns, use schedule-based autoscale.</span></span> <span data-ttu-id="6e57a-170">Si la charge est imprévisible, utilisez des règles de mise à l’échelle automatique basée sur des métriques.</span><span class="sxs-lookup"><span data-stu-id="6e57a-170">If the load is unpredictable, use metrics-based autoscaling rules.</span></span>

- <span data-ttu-id="6e57a-171">Envisagez de placer l’application web et WebJob dans des plans App Service distincts.</span><span class="sxs-lookup"><span data-stu-id="6e57a-171">Consider putting the web app and the WebJob into separate App Service plans.</span></span> <span data-ttu-id="6e57a-172">De cette manière, ils sont hébergés sur des instances distinctes de machine virtuelle et peuvent être mis à l’échelle indépendamment.</span><span class="sxs-lookup"><span data-stu-id="6e57a-172">That way, they are hosted on separate VM instances and can be scaled independently.</span></span>

- <span data-ttu-id="6e57a-173">Utilisez des plans App Service distincts pour la production et le test.</span><span class="sxs-lookup"><span data-stu-id="6e57a-173">Use separate App Service plans for production and testing.</span></span> <span data-ttu-id="6e57a-174">Sinon, si vous utilisez le même plan pour la production et le test, cela signifie que l’exécution des tests a lieu sur vos machines virtuelles de production.</span><span class="sxs-lookup"><span data-stu-id="6e57a-174">Otherwise, if you use the same plan for production and testing, it means your tests are running on your production VMs.</span></span>

- <span data-ttu-id="6e57a-175">Utilisez des emplacements de déploiement pour gérer les déploiements.</span><span class="sxs-lookup"><span data-stu-id="6e57a-175">Use deployment slots to manage deployments.</span></span> <span data-ttu-id="6e57a-176">Cela vous permet de déployer une version mise à jour sur un emplacement de préproduction, puis de basculer sur la nouvelle version.</span><span class="sxs-lookup"><span data-stu-id="6e57a-176">This lets you to deploy an updated version to a staging slot, then swap over to the new version.</span></span> <span data-ttu-id="6e57a-177">Cela vous permet également de revenir à la version précédente en cas de problème avec la mise à jour.</span><span class="sxs-lookup"><span data-stu-id="6e57a-177">It also lets you swap back to the previous version, if there was a problem with the update.</span></span>

<!-- links -->

[api-design]: ../../best-practices/api-design.md
[autoscaling]: ../../best-practices/auto-scaling.md
[caching]: ../../best-practices/caching.md
[cdn]: ../../best-practices/cdn.md
[data-partition]: ../../best-practices/data-partitioning.md
[polyglot]: ../design-principles/use-the-best-data-store.md
[scalable-web-app]: ../../reference-architectures/app-service-web-app/scalable-web-app.md