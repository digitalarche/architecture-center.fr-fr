---
title: Implémentation de référence des microservices pour Azure Kubernetes Service
description: Cette implémentation de référence présente les bonnes pratiques pour une architecture de microservices
author: MikeWasson
ms.date: 02/26/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: 15e9aa16c0e2cfccecbfb84d217c275cc99a66fd
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58344408"
---
# <a name="designing-a-microservices-architecture"></a><span data-ttu-id="15bd9-103">Conception d’une architecture de microservices</span><span class="sxs-lookup"><span data-stu-id="15bd9-103">Designing a microservices architecture</span></span>

<span data-ttu-id="15bd9-104">Les microservices sont devenus un style architectural populaire pour générer des applications cloud résilientes, hautement évolutives, capables d’évoluer rapidement et pouvant être déployées indépendamment.</span><span class="sxs-lookup"><span data-stu-id="15bd9-104">Microservices have become a popular architectural style for building cloud applications that are resilient, highly scalable, independently deployable, and able to evolve quickly.</span></span> <span data-ttu-id="15bd9-105">Toutefois, au-delà du concept à la mode, les microservices requièrent une approche différente pour concevoir et générer des applications.</span><span class="sxs-lookup"><span data-stu-id="15bd9-105">To be more than just a buzzword, however, microservices require a different approach to designing and building applications.</span></span>

<span data-ttu-id="15bd9-106">Dans cet ensemble d’articles, nous explorons comment générer et exécuter une architecture de microservices sur Azure.</span><span class="sxs-lookup"><span data-stu-id="15bd9-106">In this set of articles, we explore how to build and run a microservices architecture on Azure.</span></span> <span data-ttu-id="15bd9-107">Les sujets abordés sont les suivants :</span><span class="sxs-lookup"><span data-stu-id="15bd9-107">Topics include:</span></span>

- [<span data-ttu-id="15bd9-108">Communication entre les services</span><span class="sxs-lookup"><span data-stu-id="15bd9-108">Interservice communication</span></span>](./interservice-communication.md)
- [<span data-ttu-id="15bd9-109">Conception d’API</span><span class="sxs-lookup"><span data-stu-id="15bd9-109">API design</span></span>](./api-design.md)
- [<span data-ttu-id="15bd9-110">Passerelles d’API</span><span class="sxs-lookup"><span data-stu-id="15bd9-110">API gateways</span></span>](./gateway.md)
- [<span data-ttu-id="15bd9-111">Considérations sur les données</span><span class="sxs-lookup"><span data-stu-id="15bd9-111">Data considerations</span></span>](./data-considerations.md)
- [<span data-ttu-id="15bd9-112">Modèles de conception</span><span class="sxs-lookup"><span data-stu-id="15bd9-112">Design patterns</span></span>](./patterns.md)

## <a name="prerequisites"></a><span data-ttu-id="15bd9-113">Prérequis</span><span class="sxs-lookup"><span data-stu-id="15bd9-113">Prerequisites</span></span>

<span data-ttu-id="15bd9-114">Avant de lire ces articles, vous pouvez commencer par ce qui suit :</span><span class="sxs-lookup"><span data-stu-id="15bd9-114">Before reading these articles, you might start with the following:</span></span>

- <span data-ttu-id="15bd9-115">[Introduction aux architectures de microservices](../introduction.md).</span><span class="sxs-lookup"><span data-stu-id="15bd9-115">[Introduction to microservices architectures](../introduction.md).</span></span> <span data-ttu-id="15bd9-116">Comprendre les avantages et les problématiques des microservices, et quand utiliser ce style d’architecture.</span><span class="sxs-lookup"><span data-stu-id="15bd9-116">Understand the benefits and challenges of microservices, and when to use this style of architecture.</span></span>
- <span data-ttu-id="15bd9-117">[Utilisation de l’analyse de domaine pour modéliser les microservices](../model/domain-analysis.md).</span><span class="sxs-lookup"><span data-stu-id="15bd9-117">[Using domain analysis to model microservices](../model/domain-analysis.md).</span></span> <span data-ttu-id="15bd9-118">Découvrir une approche orientée domaine pour la modélisation des microservices.</span><span class="sxs-lookup"><span data-stu-id="15bd9-118">Learn a domain-driven approach to modeling microservices.</span></span>

## <a name="reference-implementation"></a><span data-ttu-id="15bd9-119">Implémentation de référence</span><span class="sxs-lookup"><span data-stu-id="15bd9-119">Reference implementation</span></span>

<span data-ttu-id="15bd9-120">Pour illustrer les bonnes pratiques concernant une architecture de microservices, nous avons créé une implémentation de référence que nous appelons « application de livraison par drone (Drone Delivery) ».</span><span class="sxs-lookup"><span data-stu-id="15bd9-120">To illustrate best practices for a microservices architecture, we created a reference implementation that we call the Drone Delivery application.</span></span> <span data-ttu-id="15bd9-121">Cette implémentation s’exécute sur Kubernetes à l’aide d’Azure Kubernetes Service (AKS).</span><span class="sxs-lookup"><span data-stu-id="15bd9-121">This implementation runs on Kubernetes using Azure Kubernetes Service (AKS).</span></span> <span data-ttu-id="15bd9-122">Vous trouverez l’implémentation de référence sur [GitHub][drone-ri].</span><span class="sxs-lookup"><span data-stu-id="15bd9-122">You can find the reference implementation on [GitHub][drone-ri].</span></span>

![Architecture de l’application de livraison par drone (Drone Delivery)](../images/drone-delivery.png)

## <a name="scenario"></a><span data-ttu-id="15bd9-124">Scénario</span><span class="sxs-lookup"><span data-stu-id="15bd9-124">Scenario</span></span>

<span data-ttu-id="15bd9-125">Fabrikam, Inc. veut lancer un service de livraison par drone.</span><span class="sxs-lookup"><span data-stu-id="15bd9-125">Fabrikam, Inc. is starting a drone delivery service.</span></span> <span data-ttu-id="15bd9-126">La société gère un parc de drones.</span><span class="sxs-lookup"><span data-stu-id="15bd9-126">The company manages a fleet of drone aircraft.</span></span> <span data-ttu-id="15bd9-127">Les entreprises souscrivent au service, et les utilisateurs peuvent demander à ce qu’un drone vienne récupérer les marchandises à livrer.</span><span class="sxs-lookup"><span data-stu-id="15bd9-127">Businesses register with the service, and users can request a drone to pick up goods for delivery.</span></span> <span data-ttu-id="15bd9-128">Lorsqu’un client planifie un enlèvement de colis, un système principal assigne un drone et donne à l’utilisateur un délai de livraison estimé.</span><span class="sxs-lookup"><span data-stu-id="15bd9-128">When a customer schedules a pickup, a backend system assigns a drone and notifies the user with an estimated delivery time.</span></span> <span data-ttu-id="15bd9-129">Une fois la livraison en cours, le client peut suivre la position du drone, avec un ATE mis à jour en permanence.</span><span class="sxs-lookup"><span data-stu-id="15bd9-129">While the delivery is in progress, the customer can track the location of the drone, with a continuously updated ETA.</span></span>

<span data-ttu-id="15bd9-130">Ce scénario implique un domaine assez complexe.</span><span class="sxs-lookup"><span data-stu-id="15bd9-130">This scenario involves a fairly complicated domain.</span></span> <span data-ttu-id="15bd9-131">Certaines entreprises veulent programmer des drones, suivre des colis, gérer les comptes d’utilisateur, ainsi que stocker et analyser les données d’historique.</span><span class="sxs-lookup"><span data-stu-id="15bd9-131">Some of the business concerns include scheduling drones, tracking packages, managing user accounts, and storing and analyzing historical data.</span></span> <span data-ttu-id="15bd9-132">En outre, Fabrikam souhaite commercialiser ce service et effectuer une itération rapidement, en ajoutant des nouvelles fonctionnalités.</span><span class="sxs-lookup"><span data-stu-id="15bd9-132">Moreover, Fabrikam wants to get to market quickly and then iterate quickly, adding new functionality and capabilities.</span></span> <span data-ttu-id="15bd9-133">L’application doit fonctionner à l’échelle du cloud, avec un objectif de niveau de service (SLO) élevé.</span><span class="sxs-lookup"><span data-stu-id="15bd9-133">The application needs to operate at cloud scale, with a high service level objective (SLO).</span></span> <span data-ttu-id="15bd9-134">Fabrikam prévoit également que les diverses parties du système auront des besoins très différents pour le stockage et l’interrogation des données.</span><span class="sxs-lookup"><span data-stu-id="15bd9-134">Fabrikam also expects that different parts of the system will have very different requirements for data storage and querying.</span></span> <span data-ttu-id="15bd9-135">En s’appuyant sur toutes ces considérations, Fabrikam opte pour une architecture de microservices pour l’application Drone Delivery.</span><span class="sxs-lookup"><span data-stu-id="15bd9-135">All of these considerations lead Fabrikam to choose a microservices architecture for the Drone Delivery application.</span></span>

> [!NOTE]
> <span data-ttu-id="15bd9-136">Pour éclairer votre choix entre une architecture de microservices et d’autres styles d’architecture, consultez le [Guide de l’architecture des applications Azure](../../guide/index.md).</span><span class="sxs-lookup"><span data-stu-id="15bd9-136">For help in choosing between a microservices architecture and other architectural styles, see the [Azure Application Architecture Guide](../../guide/index.md).</span></span>

<span data-ttu-id="15bd9-137">Notre implémentation de référence utilise Kubernetes avec [Azure Kubernetes Service](/azure/aks/) (AKS).</span><span class="sxs-lookup"><span data-stu-id="15bd9-137">Our reference implementation uses Kubernetes with [Azure Kubernetes Service](/azure/aks/) (AKS).</span></span> <span data-ttu-id="15bd9-138">Toutefois, la plupart des principaux défis et décisions architecturales s’appliquent à n’importe quel orchestrateur de conteneur, y compris [Azure Service Fabric](/azure/service-fabric/).</span><span class="sxs-lookup"><span data-stu-id="15bd9-138">However, many of the high-level architectural decisions and challenges will apply to any container orchestrator, including [Azure Service Fabric](/azure/service-fabric/).</span></span>

<!-- links -->

[drone-ri]: https://github.com/mspnp/microservices-reference-implementation

## <a name="next-steps"></a><span data-ttu-id="15bd9-139">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="15bd9-139">Next steps</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="15bd9-140">Choisir une option de calcul</span><span class="sxs-lookup"><span data-stu-id="15bd9-140">Choose a compute option</span></span>](./compute-options.md)