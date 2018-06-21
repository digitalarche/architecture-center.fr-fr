---
title: Modèle de cloisonnement
description: Isolez les éléments d’une application sous forme de pools afin qu’en cas de défaillance de l’un d’eux, les autres continuent à fonctionner
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: 9917870e1dcbed87aaa41e051f1622ad4950456a
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/16/2018
ms.locfileid: "31012703"
---
# <a name="bulkhead-pattern"></a><span data-ttu-id="6bf29-103">Modèle de cloisonnement</span><span class="sxs-lookup"><span data-stu-id="6bf29-103">Bulkhead pattern</span></span>

<span data-ttu-id="6bf29-104">Isolez les éléments d’une application sous forme de pools afin qu’en cas de défaillance de l’un d’eux, les autres continuent à fonctionner.</span><span class="sxs-lookup"><span data-stu-id="6bf29-104">Isolate elements of an application into pools so that if one fails, the others will continue to function.</span></span>

<span data-ttu-id="6bf29-105">Ce modèle est nommé *cloisonnement*, car il est comparable aux cloisons qui séparent les différents compartiments d’un navire.</span><span class="sxs-lookup"><span data-stu-id="6bf29-105">This pattern is named *Bulkhead* because it resembles the sectioned partitions of a ship's hull.</span></span> <span data-ttu-id="6bf29-106">Si la coque d’un navire subit une avarie, seule la section endommagée se remplit d’eau, empêchant ainsi le navire de couler.</span><span class="sxs-lookup"><span data-stu-id="6bf29-106">If the hull of a ship is compromised, only the damaged section fills with water, which prevents the ship from sinking.</span></span> 

## <a name="context-and-problem"></a><span data-ttu-id="6bf29-107">Contexte et problème</span><span class="sxs-lookup"><span data-stu-id="6bf29-107">Context and problem</span></span>

<span data-ttu-id="6bf29-108">Une application basée sur le cloud peut inclure plusieurs services comptant chacun un ou plusieurs consommateurs.</span><span class="sxs-lookup"><span data-stu-id="6bf29-108">A cloud-based application may include multiple services, with each service having one or more consumers.</span></span> <span data-ttu-id="6bf29-109">Une charge excessive ou une défaillance dans un service auront une incidence sur tous les consommateurs du service.</span><span class="sxs-lookup"><span data-stu-id="6bf29-109">Excessive load or failure in a service will impact all consumers of the service.</span></span>

<span data-ttu-id="6bf29-110">En outre, un consommateur peut envoyer des requêtes à plusieurs services simultanément en utilisant des ressources pour chaque requête.</span><span class="sxs-lookup"><span data-stu-id="6bf29-110">Moreover, a consumer may send requests to multiple services simultaneously, using resources for each request.</span></span> <span data-ttu-id="6bf29-111">Lorsque le consommateur adresse une requête à un service qui est mal configuré ou qui ne répond pas, il est possible que les ressources utilisées par cette requête ne soient pas libérées suffisamment vite.</span><span class="sxs-lookup"><span data-stu-id="6bf29-111">When the consumer sends a request to a service that is misconfigured or not responding, the resources used by the client's request may not be freed in a timely manner.</span></span> <span data-ttu-id="6bf29-112">À mesure que l’envoi de requêtes au service se poursuit, ces ressources risquent de finir par s’épuiser.</span><span class="sxs-lookup"><span data-stu-id="6bf29-112">As requests to the service continue, those resources may be exhausted.</span></span> <span data-ttu-id="6bf29-113">Par exemple, il est possible que le pool de connexions du client ait été consommé en totalité.</span><span class="sxs-lookup"><span data-stu-id="6bf29-113">For example, the client's connection pool may be exhausted.</span></span> <span data-ttu-id="6bf29-114">Lorsque cette situation se produit, elle affecte également les requêtes que le consommateur envoie à d’autres services.</span><span class="sxs-lookup"><span data-stu-id="6bf29-114">At that point, requests by the consumer to other services are impacted.</span></span> <span data-ttu-id="6bf29-115">En fin de compte, le consommateur ne peut plus adresser de requêtes au service qui ne répondait pas à l’origine, ni à aucun autre service.</span><span class="sxs-lookup"><span data-stu-id="6bf29-115">Eventually the consumer can no longer send requests to other services, not just the original unresponsive service.</span></span>

<span data-ttu-id="6bf29-116">Ce même problème d’épuisement des ressources affecte les services desservant plusieurs consommateurs.</span><span class="sxs-lookup"><span data-stu-id="6bf29-116">The same issue of resource exhaustion affects services with multiple consumers.</span></span> <span data-ttu-id="6bf29-117">Un grand nombre de requêtes émanant d’un même client peut épuiser les ressources disponibles dans le service.</span><span class="sxs-lookup"><span data-stu-id="6bf29-117">A large number of requests originating from one client may exhaust available resources in the service.</span></span> <span data-ttu-id="6bf29-118">Les autres consommateurs ne sont plus en mesure de consommer le service, ce qui entraîne un effet de défaillances en cascade.</span><span class="sxs-lookup"><span data-stu-id="6bf29-118">Other consumers are no longer able to consume the service, causing a cascading failure effect.</span></span>

## <a name="solution"></a><span data-ttu-id="6bf29-119">Solution</span><span class="sxs-lookup"><span data-stu-id="6bf29-119">Solution</span></span>

<span data-ttu-id="6bf29-120">Partitionnez les instances de service en différents groupes basés sur les exigences des consommateurs en matière de charge et de disponibilité.</span><span class="sxs-lookup"><span data-stu-id="6bf29-120">Partition service instances into different groups, based on consumer load and availability requirements.</span></span> <span data-ttu-id="6bf29-121">Cette conception vous permet d’isoler les défaillances et de préserver les fonctionnalités des services auprès de certains consommateurs, même dans l’éventualité d’une défaillance.</span><span class="sxs-lookup"><span data-stu-id="6bf29-121">This design helps to isolate failures, and allows you to sustain service functionality for some consumers, even during a failure.</span></span>

<span data-ttu-id="6bf29-122">Un consommateur peut également partitionner les ressources pour garantir que celles qui sont utilisées pour appeler un service n’affectent pas celles qui permettent d’appeler un autre service.</span><span class="sxs-lookup"><span data-stu-id="6bf29-122">A consumer can also partition resources, to ensure that resources used to call one service don't affect the resources used to call another service.</span></span> <span data-ttu-id="6bf29-123">Par exemple, un consommateur qui appelle plusieurs services peut se voir attribuer un pool de connexions pour chaque service.</span><span class="sxs-lookup"><span data-stu-id="6bf29-123">For example, a consumer that calls multiple services may be assigned a connection pool for each service.</span></span> <span data-ttu-id="6bf29-124">Si un service devient défaillant, cela affecte uniquement le pool de connexions attribué à ce service, permettant ainsi au consommateur de continuer à utiliser les autres services.</span><span class="sxs-lookup"><span data-stu-id="6bf29-124">If a service begins to fail, it only affects the connection pool assigned for that service, allowing the consumer to continue using the other services.</span></span>

<span data-ttu-id="6bf29-125">Ce modèle offre les avantages suivants :</span><span class="sxs-lookup"><span data-stu-id="6bf29-125">The benefits of this pattern include:</span></span>

- <span data-ttu-id="6bf29-126">Les consommateurs et les services sont isolés des défaillances en cascade.</span><span class="sxs-lookup"><span data-stu-id="6bf29-126">Isolates consumers and services from cascading failures.</span></span> <span data-ttu-id="6bf29-127">Un problème affectant un consommateur ou un service peut être isolé dans sa propre cloison, empêchant ainsi la défaillance de la totalité de la solution.</span><span class="sxs-lookup"><span data-stu-id="6bf29-127">An issue affecting a consumer or service can be isolated within its own bulkhead, preventing the entire solution from failing.</span></span>
- <span data-ttu-id="6bf29-128">Vous pouvez préserver certaines fonctionnalités en cas de défaillance d’un service.</span><span class="sxs-lookup"><span data-stu-id="6bf29-128">Allows you to preserve some functionality in the event of a service failure.</span></span> <span data-ttu-id="6bf29-129">Les autres services et fonctionnalités de l’application continueront à fonctionner.</span><span class="sxs-lookup"><span data-stu-id="6bf29-129">Other services and features of the application will continue to work.</span></span>
- <span data-ttu-id="6bf29-130">Vous pouvez déployer des services qui offrent différentes qualités de service pour les applications consommatrices.</span><span class="sxs-lookup"><span data-stu-id="6bf29-130">Allows you to deploy services that offer a different quality of service for consuming applications.</span></span> <span data-ttu-id="6bf29-131">Vous avez la possibilité de configurer un pool de consommateurs hautement prioritaire pour l’utilisation des services à priorité élevée.</span><span class="sxs-lookup"><span data-stu-id="6bf29-131">A high-priority consumer pool can be configured to use high-priority services.</span></span> 

<span data-ttu-id="6bf29-132">Le diagramme ci-après illustre des cloisons structurées autour de pools de connexions qui appellent des services individuels.</span><span class="sxs-lookup"><span data-stu-id="6bf29-132">The following diagram shows bulkheads structured around connection pools that call individual services.</span></span> <span data-ttu-id="6bf29-133">Si le Service A devient défaillant ou pose un autre problème, le pool de connexions est isolé, de sorte que seules les charges de travail qui utilisent le pool de threads attribué au Service A sont affectées par le problème.</span><span class="sxs-lookup"><span data-stu-id="6bf29-133">If Service A fails or causes some other issue, the connection pool is isolated, so only workloads using the thread pool assigned to Service A are affected.</span></span> <span data-ttu-id="6bf29-134">Les charges de travail qui utilisent les Services B et C ne sont pas touchées par le problème et peuvent se poursuivre sans interruption.</span><span class="sxs-lookup"><span data-stu-id="6bf29-134">Workloads that use Service B and C are not affected and can continue working without interruption.</span></span>

![](./_images/bulkhead-1.png) 

<span data-ttu-id="6bf29-135">Le diagramme ci-après représente plusieurs clients appelant un seul service.</span><span class="sxs-lookup"><span data-stu-id="6bf29-135">The next diagram shows multiple clients calling a single service.</span></span> <span data-ttu-id="6bf29-136">Une instance de service distincte est attribuée à chaque client.</span><span class="sxs-lookup"><span data-stu-id="6bf29-136">Each client is assigned a separate service instance.</span></span> <span data-ttu-id="6bf29-137">Le Client 1 a effectué trop de requêtes et a submergé son instance.</span><span class="sxs-lookup"><span data-stu-id="6bf29-137">Client 1 has made too many requests and overwhelmed its instance.</span></span> <span data-ttu-id="6bf29-138">Étant donné que chaque instance de service est isolée des autres, les autres clients peuvent continuer à effectuer des appels.</span><span class="sxs-lookup"><span data-stu-id="6bf29-138">Because each service instance is isolated from the others, the other clients can continue making calls.</span></span>

![](./_images/bulkhead-2.png)
     
## <a name="issues-and-considerations"></a><span data-ttu-id="6bf29-139">Problèmes et considérations</span><span class="sxs-lookup"><span data-stu-id="6bf29-139">Issues and considerations</span></span>

- <span data-ttu-id="6bf29-140">Définissez des partitions basées sur les exigences métiers et techniques de l’application.</span><span class="sxs-lookup"><span data-stu-id="6bf29-140">Define partitions around the business and technical requirements of the application.</span></span>
- <span data-ttu-id="6bf29-141">Lorsque vous partitionnez des services ou des consommateurs en cloisons, tenez compte du niveau d’isolation offert par la technologie, ainsi que des gains en termes de coût, de performances et de facilité de gestion.</span><span class="sxs-lookup"><span data-stu-id="6bf29-141">When partitioning services or consumers into bulkheads, consider the level of isolation offered by the technology as well as the overhead in terms of cost, performance and manageability.</span></span>
- <span data-ttu-id="6bf29-142">Envisagez de combiner les cloisons avec les modèles de nouvelle tentative, de disjoncteur et de limitation afin d’assurer une gestion des erreurs plus élaborée.</span><span class="sxs-lookup"><span data-stu-id="6bf29-142">Consider combining bulkheads with retry, circuit breaker, and throttling patterns to provide more sophisticated fault handling.</span></span>
- <span data-ttu-id="6bf29-143">Lorsque vous partitionnez les consommateurs en cloisons, pensez à utiliser des processus, des pools de threads et des sémaphores.</span><span class="sxs-lookup"><span data-stu-id="6bf29-143">When partitioning consumers into bulkheads, consider using processes, thread pools, and semaphores.</span></span> <span data-ttu-id="6bf29-144">Les projets tels que [Netflix Hystrix][hystrix] et [Polly][polly] offrent une infrastructure pour la création de cloisons de consommateurs.</span><span class="sxs-lookup"><span data-stu-id="6bf29-144">Projects like [Netflix Hystrix][hystrix] and [Polly][polly] offer a framework for creating consumer bulkheads.</span></span>
- <span data-ttu-id="6bf29-145">Lorsque vous partitionnez des services en cloisons, envisagez de les déployer dans des machines virtuelles, des conteneurs ou des processus distincts.</span><span class="sxs-lookup"><span data-stu-id="6bf29-145">When partitioning services into bulkheads, consider deploying them into separate virtual machines, containers, or processes.</span></span> <span data-ttu-id="6bf29-146">Les conteneurs offrent un bon niveau d’isolation des ressources pour un coût relativement faible.</span><span class="sxs-lookup"><span data-stu-id="6bf29-146">Containers offer a good balance of resource isolation with fairly low overhead.</span></span>
- <span data-ttu-id="6bf29-147">Les services qui communiquent à l’aide de messages asynchrones peuvent être isolés par le biais de différents ensembles de files d’attente.</span><span class="sxs-lookup"><span data-stu-id="6bf29-147">Services that communicate using asynchronous messages can be isolated through different sets of queues.</span></span> <span data-ttu-id="6bf29-148">Chaque file d’attente peut disposer d’un ensemble dédié d’instances traitant les messages de la file d’attente, ou d’un groupe d’instances unique utilisant un algorithme pour le traitement des opérations d’enlèvement de la file d’attente et de répartition.</span><span class="sxs-lookup"><span data-stu-id="6bf29-148">Each queue can have a dedicated set of instances processing messages on the queue, or a single group of instances using an algorithm to dequeue and dispatch processing.</span></span>
- <span data-ttu-id="6bf29-149">Déterminez le niveau de granularité des cloisons.</span><span class="sxs-lookup"><span data-stu-id="6bf29-149">Determine the level of granularity for the bulkheads.</span></span> <span data-ttu-id="6bf29-150">Par exemple, si vous souhaitez répartir des clients entre plusieurs partitions, vous pouvez placer chaque client dans une partition distincte, ou placer plusieurs clients dans une seule partition.</span><span class="sxs-lookup"><span data-stu-id="6bf29-150">For example, if you want to distribute tenants across partitions, you could place each tenant into a separate partition, or put several tenants into one partition.</span></span>
- <span data-ttu-id="6bf29-151">Surveillez les performances et le Contrat de niveau de service de chaque partition.</span><span class="sxs-lookup"><span data-stu-id="6bf29-151">Monitor each partition’s performance and SLA.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="6bf29-152">Quand utiliser ce modèle</span><span class="sxs-lookup"><span data-stu-id="6bf29-152">When to use this pattern</span></span>

<span data-ttu-id="6bf29-153">Utilisez ce modèle pour :</span><span class="sxs-lookup"><span data-stu-id="6bf29-153">Use this pattern to:</span></span>

- <span data-ttu-id="6bf29-154">isoler les ressources utilisées pour la consommation d’un ensemble de services principaux, notamment si l’application peut assurer un certain niveau de fonctionnalité, même si l’un des services ne répond pas ;</span><span class="sxs-lookup"><span data-stu-id="6bf29-154">Isolate resources used to consume a set of backend services, especially if the application can provide some level of functionality even when one of the services is not responding.</span></span>
- <span data-ttu-id="6bf29-155">isoler les consommateurs critiques des consommateurs standard ;</span><span class="sxs-lookup"><span data-stu-id="6bf29-155">Isolate critical consumers from standard consumers.</span></span>
- <span data-ttu-id="6bf29-156">protéger l’application des défaillances en cascade.</span><span class="sxs-lookup"><span data-stu-id="6bf29-156">Protect the application from cascading failures.</span></span>

<span data-ttu-id="6bf29-157">Ce modèle peut ne pas convenir dans les cas suivants :</span><span class="sxs-lookup"><span data-stu-id="6bf29-157">This pattern may not be suitable when:</span></span>

- <span data-ttu-id="6bf29-158">Une utilisation moins efficace des ressources n’est pas acceptable dans le projet.</span><span class="sxs-lookup"><span data-stu-id="6bf29-158">Less efficient use of resources may not be acceptable in the project.</span></span>
- <span data-ttu-id="6bf29-159">Ce surcroît de complexité n’est pas nécessaire.</span><span class="sxs-lookup"><span data-stu-id="6bf29-159">The added complexity is not necessary</span></span>

## <a name="example"></a><span data-ttu-id="6bf29-160">Exemples</span><span class="sxs-lookup"><span data-stu-id="6bf29-160">Example</span></span>

<span data-ttu-id="6bf29-161">Le fichier de configuration Kubernetes ci-après crée un conteneur isolé pour l’exécution d’un seul service, doté de ses propres ressources et limites d’UC et de mémoire.</span><span class="sxs-lookup"><span data-stu-id="6bf29-161">The following Kubernetes configuration file creates an isolated container to run a single service, with its own CPU and memory resources and limits.</span></span>

```yml
apiVersion: v1
kind: Pod
metadata:
  name: drone-management
spec:
  containers:
  - name: drone-management-container
    image: drone-service
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "1"
```

## <a name="related-guidance"></a><span data-ttu-id="6bf29-162">Aide connexe</span><span class="sxs-lookup"><span data-stu-id="6bf29-162">Related guidance</span></span>

- [<span data-ttu-id="6bf29-163">Modèle Disjoncteur</span><span class="sxs-lookup"><span data-stu-id="6bf29-163">Circuit Breaker pattern</span></span>](./circuit-breaker.md)
- [<span data-ttu-id="6bf29-164">Conception d’applications résilientes pour Azure</span><span class="sxs-lookup"><span data-stu-id="6bf29-164">Designing resilient applications for Azure</span></span>](../resiliency/index.md)
- [<span data-ttu-id="6bf29-165">Modèle Nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="6bf29-165">Retry pattern</span></span>](./retry.md)
- [<span data-ttu-id="6bf29-166">Modèle de limitation</span><span class="sxs-lookup"><span data-stu-id="6bf29-166">Throttling pattern</span></span>](./throttling.md)


<!-- links -->

[hystrix]: https://github.com/Netflix/Hystrix
[polly]: https://github.com/App-vNext/Polly