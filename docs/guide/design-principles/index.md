---
title: Principes de conception pour les applications Azure
titleSuffix: Azure Application Architecture Guide
description: Principes de conception pour les applications Azure.
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
---

# <a name="ten-design-principles-for-azure-applications"></a><span data-ttu-id="de482-103">Dix principes de conception pour les applications Azure</span><span class="sxs-lookup"><span data-stu-id="de482-103">Ten design principles for Azure applications</span></span>

<span data-ttu-id="de482-104">Appliquez ces principes de conception pour rendre votre application plus évolutive, résiliente et gérable.</span><span class="sxs-lookup"><span data-stu-id="de482-104">Follow these design principles to make your application more scalable, resilient, and manageable.</span></span>

<span data-ttu-id="de482-105">**[Penser la conception des applications pour la réparation spontanée](self-healing.md)**.</span><span class="sxs-lookup"><span data-stu-id="de482-105">**[Design for self healing](self-healing.md)**.</span></span> <span data-ttu-id="de482-106">Dans un système distribué, des défaillances peuvent se produire.</span><span class="sxs-lookup"><span data-stu-id="de482-106">In a distributed system, failures happen.</span></span> <span data-ttu-id="de482-107">Concevez votre application pour qu’elle se répare spontanément en cas de défaillance.</span><span class="sxs-lookup"><span data-stu-id="de482-107">Design your application to be self healing when failures occur.</span></span>

<span data-ttu-id="de482-108">**[Rendre tous les éléments redondant](redundancy.md)**.</span><span class="sxs-lookup"><span data-stu-id="de482-108">**[Make all things redundant](redundancy.md)**.</span></span> <span data-ttu-id="de482-109">Intégrez la redondance à votre application, pour éviter les points de défaillance uniques.</span><span class="sxs-lookup"><span data-stu-id="de482-109">Build redundancy into your application, to avoid having single points of failure.</span></span>

<span data-ttu-id="de482-110">**[Réduire la coordination](minimize-coordination.md)**.</span><span class="sxs-lookup"><span data-stu-id="de482-110">**[Minimize coordination](minimize-coordination.md)**.</span></span> <span data-ttu-id="de482-111">Réduisez la coordination entre les services d’application pour atteindre une certaine évolutivité.</span><span class="sxs-lookup"><span data-stu-id="de482-111">Minimize coordination between application services to achieve scalability.</span></span>

<span data-ttu-id="de482-112">**[Penser la conception des applications pour augmenter la taille des instances](scale-out.md)**. Concevez votre application afin qu’elle puisse évoluer horizontalement, par l’ajout ou la suppression de nouvelles instances en fonction de la demande.</span><span class="sxs-lookup"><span data-stu-id="de482-112">**[Design to scale out](scale-out.md)**. Design your application so that it can scale horizontally, adding or removing new instances as demand requires.</span></span>

<span data-ttu-id="de482-113">**[Partitionner autour des limites](partition.md)**.</span><span class="sxs-lookup"><span data-stu-id="de482-113">**[Partition around limits](partition.md)**.</span></span> <span data-ttu-id="de482-114">Utilisez le partitionnement pour contourner les limites liées à la base de données, au réseau et au calcul.</span><span class="sxs-lookup"><span data-stu-id="de482-114">Use partitioning to work around database, network, and compute limits.</span></span>

<span data-ttu-id="de482-115">**[Penser la conception des applications pour les opérations](design-for-operations.md)**.</span><span class="sxs-lookup"><span data-stu-id="de482-115">**[Design for operations](design-for-operations.md)**.</span></span> <span data-ttu-id="de482-116">Concevez votre application pour que l’équipe des opérations dispose des outils dont elle a besoin.</span><span class="sxs-lookup"><span data-stu-id="de482-116">Design your application so that the operations team has the tools they need.</span></span>

<span data-ttu-id="de482-117">**[Utiliser les services managés Azure](managed-services.md)**.</span><span class="sxs-lookup"><span data-stu-id="de482-117">**[Use managed services](managed-services.md)**.</span></span> <span data-ttu-id="de482-118">Si possible, utilisez platform as a service (PaaS) au lieu de infrastructure as a service (IaaS).</span><span class="sxs-lookup"><span data-stu-id="de482-118">When possible, use platform as a service (PaaS) rather than infrastructure as a service (IaaS).</span></span>

<span data-ttu-id="de482-119">**[Utiliser la meilleure banque de données pour le travail](use-the-best-data-store.md)**.</span><span class="sxs-lookup"><span data-stu-id="de482-119">**[Use the best data store for the job](use-the-best-data-store.md)**.</span></span> <span data-ttu-id="de482-120">Choisissez la technologie de stockage la mieux adaptée à vos données et son mode d’utilisation.</span><span class="sxs-lookup"><span data-stu-id="de482-120">Pick the storage technology that is the best fit for your data and how it will be used.</span></span>

<span data-ttu-id="de482-121">**[Penser la conception des applications en vue de leur évolution](design-for-evolution.md)**.</span><span class="sxs-lookup"><span data-stu-id="de482-121">**[Design for evolution](design-for-evolution.md)**.</span></span> <span data-ttu-id="de482-122">Mêmes les applications les plus réussies changent au fil du temps.</span><span class="sxs-lookup"><span data-stu-id="de482-122">All successful applications change over time.</span></span> <span data-ttu-id="de482-123">Une conception évolutive est essentielle une innovation continue.</span><span class="sxs-lookup"><span data-stu-id="de482-123">An evolutionary design is key for continuous innovation.</span></span>

<span data-ttu-id="de482-124">**[Créer pour les besoins de l’entreprise](build-for-business.md)**.</span><span class="sxs-lookup"><span data-stu-id="de482-124">**[Build for the needs of business](build-for-business.md)**.</span></span> <span data-ttu-id="de482-125">Chaque décision de conception doit être justifiée par un besoin de l’entreprise.</span><span class="sxs-lookup"><span data-stu-id="de482-125">Every design decision must be justified by a business requirement.</span></span>
