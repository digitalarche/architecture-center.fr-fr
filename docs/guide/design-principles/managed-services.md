---
title: Utiliser les services gérés
titleSuffix: Azure Application Architecture Guide
description: Dans la mesure du possible, privilégiez l’approche PaaS (platform as a service) plutôt que l’approche IaaS (infrastructure as a service).
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: f08914e1eacc4f02fdb16093fe590f46249e3da3
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58249534"
---
# <a name="use-managed-services"></a><span data-ttu-id="71d67-103">Utiliser les services gérés</span><span class="sxs-lookup"><span data-stu-id="71d67-103">Use managed services</span></span>

## <a name="when-possible-use-platform-as-a-service-paas-rather-than-infrastructure-as-a-service-iaas"></a><span data-ttu-id="71d67-104">Dans la mesure du possible, privilégiez l’approche platform as a service (PaaS) plutôt que l’approche infrastructure as a service (IaaS)</span><span class="sxs-lookup"><span data-stu-id="71d67-104">When possible, use platform as a service (PaaS) rather than infrastructure as a service (IaaS)</span></span>

<span data-ttu-id="71d67-105">L’approche IaaS équivaut à disposer d’un carton rempli de pièces.</span><span class="sxs-lookup"><span data-stu-id="71d67-105">IaaS is like having a box of parts.</span></span> <span data-ttu-id="71d67-106">Vous pouvez construire tout ce que vous voulez, mais vous devez tout assembler vous-même.</span><span class="sxs-lookup"><span data-stu-id="71d67-106">You can build anything, but you have to assemble it yourself.</span></span> <span data-ttu-id="71d67-107">Les services gérés sont plus faciles à configurer et à administrer.</span><span class="sxs-lookup"><span data-stu-id="71d67-107">Managed services are easier to configure and administer.</span></span> <span data-ttu-id="71d67-108">Vous n’avez pas besoin d’approvisionner de machines virtuelles, de gérer les correctifs et les mises à jour, ni de prendre en charge les autres surcoûts inhérents à l’exécution de logiciels sur une machine virtuelle.</span><span class="sxs-lookup"><span data-stu-id="71d67-108">You don't need to provision VMs, set up VNets, manage patches and updates, and all of the other overhead associated with running software on a VM.</span></span>

<span data-ttu-id="71d67-109">Par exemple, supposons que votre application ait besoin d’une file d’attente de messages.</span><span class="sxs-lookup"><span data-stu-id="71d67-109">For example, suppose your application needs a message queue.</span></span> <span data-ttu-id="71d67-110">Vous pouvez configurer votre propre service de messagerie sur une machine virtuelle à l’aide d’un logiciel tel que RabbitMQ.</span><span class="sxs-lookup"><span data-stu-id="71d67-110">You could set up your own messaging service on a VM, using something like RabbitMQ.</span></span> <span data-ttu-id="71d67-111">Toutefois, Azure Service Bus fournit déjà une messagerie fiable sous la forme d’un service, et se révèle plus simple à configurer.</span><span class="sxs-lookup"><span data-stu-id="71d67-111">But Azure Service Bus already provides reliable messaging as service, and it's simpler to set up.</span></span> <span data-ttu-id="71d67-112">Il vous suffit de créer un espace de noms Service Bus (ce qui peut être effectué dans le cadre d’un script de déploiement), puis d’appeler Service Bus à l’aide du Kit de développement logiciel (SDK) client.</span><span class="sxs-lookup"><span data-stu-id="71d67-112">Just create a Service Bus namespace (which can be done as part of a deployment script) and then call Service Bus using the client SDK.</span></span>

<span data-ttu-id="71d67-113">Bien entendu, il est possible que votre application présente certaines exigences auxquelles une approche IaaS peut répondre de façon plus adéquate.</span><span class="sxs-lookup"><span data-stu-id="71d67-113">Of course, your application may have specific requirements that make an IaaS approach more suitable.</span></span> <span data-ttu-id="71d67-114">Toutefois, même si votre application repose sur IaaS, recherchez les emplacements où vous pouvez incorporer naturellement des services gérés.</span><span class="sxs-lookup"><span data-stu-id="71d67-114">However, even if your application is based on IaaS, look for places where it may be natural to incorporate managed services.</span></span> <span data-ttu-id="71d67-115">Ces emplacements comprennent le cache, les files d’attente et le stockage des données.</span><span class="sxs-lookup"><span data-stu-id="71d67-115">These include cache, queues, and data storage.</span></span>

| <span data-ttu-id="71d67-116">Au lieu d’exécuter...</span><span class="sxs-lookup"><span data-stu-id="71d67-116">Instead of running...</span></span> | <span data-ttu-id="71d67-117">Utilisez plutôt...</span><span class="sxs-lookup"><span data-stu-id="71d67-117">Consider using...</span></span> |
|-----------------------|-------------|
| <span data-ttu-id="71d67-118">Active Directory</span><span class="sxs-lookup"><span data-stu-id="71d67-118">Active Directory</span></span> | <span data-ttu-id="71d67-119">Azure Active Directory Domain Services</span><span class="sxs-lookup"><span data-stu-id="71d67-119">Azure Active Directory Domain Services</span></span> |
| <span data-ttu-id="71d67-120">Elasticsearch</span><span class="sxs-lookup"><span data-stu-id="71d67-120">Elasticsearch</span></span> | <span data-ttu-id="71d67-121">Recherche Azure</span><span class="sxs-lookup"><span data-stu-id="71d67-121">Azure Search</span></span> |
| <span data-ttu-id="71d67-122">Hadoop</span><span class="sxs-lookup"><span data-stu-id="71d67-122">Hadoop</span></span> | <span data-ttu-id="71d67-123">HDInsight</span><span class="sxs-lookup"><span data-stu-id="71d67-123">HDInsight</span></span> |
| <span data-ttu-id="71d67-124">IIS</span><span class="sxs-lookup"><span data-stu-id="71d67-124">IIS</span></span> | <span data-ttu-id="71d67-125">App Service</span><span class="sxs-lookup"><span data-stu-id="71d67-125">App Service</span></span> |
| <span data-ttu-id="71d67-126">MongoDB</span><span class="sxs-lookup"><span data-stu-id="71d67-126">MongoDB</span></span> | <span data-ttu-id="71d67-127">Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="71d67-127">Cosmos DB</span></span> |
| <span data-ttu-id="71d67-128">Redis</span><span class="sxs-lookup"><span data-stu-id="71d67-128">Redis</span></span> | <span data-ttu-id="71d67-129">Cache Redis Azure</span><span class="sxs-lookup"><span data-stu-id="71d67-129">Azure Redis Cache</span></span> |
| <span data-ttu-id="71d67-130">SQL Server</span><span class="sxs-lookup"><span data-stu-id="71d67-130">SQL Server</span></span> | <span data-ttu-id="71d67-131">Azure SQL Database</span><span class="sxs-lookup"><span data-stu-id="71d67-131">Azure SQL Database</span></span> |
