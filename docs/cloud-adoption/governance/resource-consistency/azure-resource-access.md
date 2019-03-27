---
title: 'Framework d’adoption du cloud : Gestion de l’accès aux ressources dans Azure'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: 'Présentation des constructions de gestion des accès aux ressources dans Azure : Azure Resource Manager, abonnements, groupes de ressources et ressources'
author: petertaylor9999
ms.openlocfilehash: b98cdc94d6d3a37c1e65da1d4de35d5d9520d6eb
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58243200"
---
# <a name="resource-access-management-in-azure"></a><span data-ttu-id="80dbb-103">Gestion de l’accès aux ressources dans Azure</span><span class="sxs-lookup"><span data-stu-id="80dbb-103">Resource access management in Azure</span></span>

<span data-ttu-id="80dbb-104">La rubrique [Gouvernance cloud](../overview.md) présente les cinq disciplines de la gouvernance cloud, ce qui inclut la gestion des ressources.</span><span class="sxs-lookup"><span data-stu-id="80dbb-104">[Cloud Governance](../overview.md) outlines the five disciplines of Cloud Governance, which includes Resource Management.</span></span>  <span data-ttu-id="80dbb-105">La rubrique [Présentation de la gouvernance des accès aux ressources](overview.md) explique comment la gestion de l’accès aux ressources s’inscrit dans la discipline de gestion des ressources.</span><span class="sxs-lookup"><span data-stu-id="80dbb-105">[What is resource access governance](overview.md) furthers explains how resource access management fits into the resource management discipline.</span></span> <span data-ttu-id="80dbb-106">Avant de découvrir comment concevoir un modèle de gouvernance, il est important de comprendre les contrôles de gestion des accès aux ressources dans Azure.</span><span class="sxs-lookup"><span data-stu-id="80dbb-106">Before you move on to learn how to design a governance model, it's important to understand the resource access management controls in Azure.</span></span> <span data-ttu-id="80dbb-107">La configuration de ces contrôles de gestion des accès aux ressources constitue la base de votre modèle de gouvernance.</span><span class="sxs-lookup"><span data-stu-id="80dbb-107">The configuration of these resource access management controls forms the basis of your governance model.</span></span>

<span data-ttu-id="80dbb-108">Commencez par examiner de plus près le déploiement des ressources dans Azure.</span><span class="sxs-lookup"><span data-stu-id="80dbb-108">Begin by taking a closer look at how resources are deployed in Azure.</span></span>

<!-- markdownlint-disable MD026 -->

## <a name="what-is-an-azure-resource"></a><span data-ttu-id="80dbb-109">Qu’est-ce qu’une ressource Azure ?</span><span class="sxs-lookup"><span data-stu-id="80dbb-109">What is an Azure resource?</span></span>

<span data-ttu-id="80dbb-110">Dans Azure, le terme **ressource** fait référence à une entité gérée par Azure.</span><span class="sxs-lookup"><span data-stu-id="80dbb-110">In Azure, the term **resource** refers to an entity managed by Azure.</span></span> <span data-ttu-id="80dbb-111">Par exemple, les machines virtuelles, les réseaux virtuels et les comptes de stockage sont tous considérés comme des ressources Azure.</span><span class="sxs-lookup"><span data-stu-id="80dbb-111">For example, virtual machines, virtual networks, and storage accounts are all referred to as Azure resources.</span></span>

<span data-ttu-id="80dbb-112">![Diagramme d’une ressource](../../_images/governance-1-9.png)
*Figure 1. Une ressource.*</span><span class="sxs-lookup"><span data-stu-id="80dbb-112">![Diagram of a resource](../../_images/governance-1-9.png)
*Figure 1. A resource.*</span></span>

## <a name="what-is-an-azure-resource-group"></a><span data-ttu-id="80dbb-113">Qu’est-ce qu’un groupe de ressources Azure ?</span><span class="sxs-lookup"><span data-stu-id="80dbb-113">What is an Azure resource group?</span></span>

<span data-ttu-id="80dbb-114">Chaque ressource dans Azure doit appartenir à un [groupe de ressources](/azure/azure-resource-manager/resource-group-overview#resource-groups).</span><span class="sxs-lookup"><span data-stu-id="80dbb-114">Each resource in Azure must belong to a [resource group](/azure/azure-resource-manager/resource-group-overview#resource-groups).</span></span> <span data-ttu-id="80dbb-115">Un groupe de ressources est simplement une construction logique qui regroupe plusieurs ressources afin qu’elles puissent être gérées en tant qu’entité unique.</span><span class="sxs-lookup"><span data-stu-id="80dbb-115">A resource group is simply a logical construct that groups multiple resources together so they can be managed as a single entity.</span></span> <span data-ttu-id="80dbb-116">Par exemple, des ressources qui partagent un cycle de vie similaire, tels que les ressources d’une [application multi-niveau](/azure/architecture/guide/architecture-styles/n-tier), peuvent être créées ou supprimées en tant que groupe.</span><span class="sxs-lookup"><span data-stu-id="80dbb-116">For example, resources that share a similar lifecycle, such as the resources for an [n-tier application](/azure/architecture/guide/architecture-styles/n-tier) may be created or deleted as a group.</span></span>

<span data-ttu-id="80dbb-117">![Diagramme d’un groupe de ressources contenant une ressource](../../_images/governance-1-10.png)
*Figure 2. Un groupe de ressources contient une ressource.*</span><span class="sxs-lookup"><span data-stu-id="80dbb-117">![Diagram of a resource group containing a resource](../../_images/governance-1-10.png)
*Figure 2. A resource group contains a resource.*</span></span>

<span data-ttu-id="80dbb-118">Les groupes de ressources et les ressources qu’ils contiennent sont associés à un **abonnement** Azure.</span><span class="sxs-lookup"><span data-stu-id="80dbb-118">Resource groups and the resources they contain are associated with an Azure **subscription**.</span></span>

## <a name="what-is-an-azure-subscription"></a><span data-ttu-id="80dbb-119">Qu’est-ce qu’un abonnement Azure ?</span><span class="sxs-lookup"><span data-stu-id="80dbb-119">What is an Azure subscription?</span></span>

<span data-ttu-id="80dbb-120">Un abonnement Azure est similaire à un groupe de ressources : il s’agit d’une construction logique qui regroupe des groupes de ressources et leurs ressources.</span><span class="sxs-lookup"><span data-stu-id="80dbb-120">An Azure subscription is similar to a resource group in that it's a logical construct that groups together resource groups and their resources.</span></span> <span data-ttu-id="80dbb-121">Toutefois, un abonnement Azure est également associé aux contrôles utilisés par Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="80dbb-121">However, an Azure subscription is also associated with the controls used by Azure Resource Manager.</span></span> <span data-ttu-id="80dbb-122">Qu’est-ce que cela signifie ?</span><span class="sxs-lookup"><span data-stu-id="80dbb-122">What does this mean?</span></span> <span data-ttu-id="80dbb-123">Examinez plus en détail Azure Resource Manager pour en savoir plus sur sa relation avec un abonnement Azure.</span><span class="sxs-lookup"><span data-stu-id="80dbb-123">Take a closer look at Azure Resource Manager to learn about the relationship between it and an Azure subscription.</span></span>

<span data-ttu-id="80dbb-124">![Diagramme d’un abonnement Azure](../../_images/governance-1-11.png)
*Figure 3. Un abonnement Azure.*</span><span class="sxs-lookup"><span data-stu-id="80dbb-124">![Diagram of an Azure subscription](../../_images/governance-1-11.png)
*Figure 3. An Azure subscription.*</span></span>

## <a name="what-is-azure-resource-manager"></a><span data-ttu-id="80dbb-125">Qu’est-ce qu’Azure Resource Manager ?</span><span class="sxs-lookup"><span data-stu-id="80dbb-125">What is Azure Resource Manager?</span></span>

<span data-ttu-id="80dbb-126">Dans [Comment fonctionne Azure ?](../../getting-started/what-is-azure.md), vous avez appris qu’Azure comprend un « serveur frontal » avec de nombreux services qui orchestrent toutes les fonctions d’Azure.</span><span class="sxs-lookup"><span data-stu-id="80dbb-126">In [how does Azure work?](../../getting-started/what-is-azure.md) you learned that Azure includes a "front end" with many services that orchestrate all the functions of Azure.</span></span> <span data-ttu-id="80dbb-127">L’un de ces services, [Azure Resource Manager](/azure/azure-resource-manager/), héberge l’API RESTful utilisée par les clients pour gérer les ressources.</span><span class="sxs-lookup"><span data-stu-id="80dbb-127">One of these services is [Azure Resource Manager](/azure/azure-resource-manager/), and this service hosts the RESTful API used by clients to manage resources.</span></span>

<span data-ttu-id="80dbb-128">![Diagramme d’Azure Resource Manager](../../_images/governance-1-12.png)
*Figure 4. Azure Resource Manager.*</span><span class="sxs-lookup"><span data-stu-id="80dbb-128">![Diagram of Azure Resource Manager](../../_images/governance-1-12.png)
*Figure 4. Azure Resource Manager.*</span></span>

<span data-ttu-id="80dbb-129">La figure suivante montre trois clients : [PowerShell](/powershell/azure/overview), le [portail Azure](https://portal.azure.com) et l’[interface de ligne de commande (CLI) Azure](/cli/azure) :</span><span class="sxs-lookup"><span data-stu-id="80dbb-129">The following figure shows three clients: [PowerShell](/powershell/azure/overview), the [Azure portal](https://portal.azure.com), and the [Azure command-line interface (CLI)](/cli/azure):</span></span>

<span data-ttu-id="80dbb-130">![Diagramme de clients Azure se connectant à l’API Azure Resource Manager](../../_images/governance-1-13.png)
*Figure 5. Les clients Azure se connectent à l’API RESTful Azure Resource Manager.*</span><span class="sxs-lookup"><span data-stu-id="80dbb-130">![Diagram of Azure clients connecting to the Azure Resource Manager API](../../_images/governance-1-13.png)
*Figure 5. Azure clients connect to the Azure Resource Manager RESTful API.*</span></span>

<span data-ttu-id="80dbb-131">Bien que ces clients se connectent à Azure Resource Manager à l’aide de l’API RESTful, Azure Resource Manager n’inclut pas de fonctionnalités permettant de gérer directement les ressources.</span><span class="sxs-lookup"><span data-stu-id="80dbb-131">While these clients connect to Azure Resource Manager using the RESTful API, Azure Resource Manager does not include functionality to manage resources directly.</span></span> <span data-ttu-id="80dbb-132">Au lieu de cela, la plupart des types de ressources Azure ont leur propre [**fournisseur de ressources**](/azure/azure-resource-manager/resource-group-overview#terminology).</span><span class="sxs-lookup"><span data-stu-id="80dbb-132">Rather, most resource types in Azure have their own [**resource provider**](/azure/azure-resource-manager/resource-group-overview#terminology).</span></span>

<span data-ttu-id="80dbb-133">![Fournisseurs de ressources Azure](../../_images/governance-1-14.png)
*Figure 6. Fournisseurs de ressources Azure.*</span><span class="sxs-lookup"><span data-stu-id="80dbb-133">![Azure resource providers](../../_images/governance-1-14.png)
*Figure 6. Azure resource providers.*</span></span>

<span data-ttu-id="80dbb-134">Quand un client effectue une requête pour gérer une ressource spécifique, Azure Resource Manager se connecte au fournisseur de ressources pour ce type de ressources afin d’exécuter la requête.</span><span class="sxs-lookup"><span data-stu-id="80dbb-134">When a client makes a request to manage a specific resource, Azure Resource Manager connects to the resource provider for that resource type to complete the request.</span></span> <span data-ttu-id="80dbb-135">Par exemple, si un client effectue une requête pour gérer une ressource de machine virtuelle, Azure Resource Manager se connecte au fournisseur de ressources **Microsoft.Compute**.</span><span class="sxs-lookup"><span data-stu-id="80dbb-135">For example, if a client makes a request to manage a virtual machine resource, Azure Resource Manager connects to the **Microsoft.Compute** resource provider.</span></span>

<span data-ttu-id="80dbb-136">![Azure Resource Manager se connectant au fournisseur de ressources Microsoft.Compute](../../_images/governance-1-15.png)
*Figure 7. Azure Resource Manager se connecte au fournisseur de ressources **Microsoft.Compute** pour gérer la ressource spécifiée dans la requête du client.*</span><span class="sxs-lookup"><span data-stu-id="80dbb-136">![Azure Resource Manager connecting to the Microsoft.Compute resource provider](../../_images/governance-1-15.png)
*Figure 7. Azure Resource Manager connects to the **Microsoft.Compute** resource provider to manage the resource specified in the client request.*</span></span>

<span data-ttu-id="80dbb-137">Azure Resource Manager exige du client qu’il spécifie un identificateur pour l’abonnement et le groupe de ressources, afin de gérer la ressource de machine virtuelle.</span><span class="sxs-lookup"><span data-stu-id="80dbb-137">Azure Resource Manager requires the client to specify an identifier for both the subscription and the resource group in order to manage the virtual machine resource.</span></span>

<span data-ttu-id="80dbb-138">Maintenant que vous comprenez le fonctionnement d’Azure Resource Manager, revenez à la présentation de la façon dont un abonnement Azure est associé aux contrôles utilisés par Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="80dbb-138">Now that you have an understanding of how Azure Resource Manager works, return to the discussion of how an Azure subscription is associated with the controls used by Azure Resource Manager.</span></span> <span data-ttu-id="80dbb-139">Avant qu’une requête de gestion des ressources ne soit exécutée par Azure Resource Manager, plusieurs contrôles sont vérifiés.</span><span class="sxs-lookup"><span data-stu-id="80dbb-139">Before any resource management request can be executed by Azure Resource Manager, a set of controls are checked.</span></span>

<span data-ttu-id="80dbb-140">Le premier contrôle est qu’une requête doit être effectuée par un utilisateur validé, et qu’Azure Resource Manager a une relation de confiance avec [Azure Active Directory (Azure AD)](/azure/active-directory/) pour fournir des fonctionnalités d’identité utilisateur.</span><span class="sxs-lookup"><span data-stu-id="80dbb-140">The first control is that a request must be made by a validated user, and Azure Resource Manager has a trusted relationship with [Azure Active Directory (Azure AD)](/azure/active-directory/) to provide user identity functionality.</span></span>

<span data-ttu-id="80dbb-141">![Azure Active Directory](../../_images/governance-1-16.png)
*Figure 8. Azure Active Directory.*</span><span class="sxs-lookup"><span data-stu-id="80dbb-141">![Azure Active Directory](../../_images/governance-1-16.png)
*Figure 8. Azure Active Directory.*</span></span>

<span data-ttu-id="80dbb-142">Dans Azure AD, les utilisateurs sont segmentés sous forme de **clients**.</span><span class="sxs-lookup"><span data-stu-id="80dbb-142">In Azure AD, users are segmented into **tenants**.</span></span> <span data-ttu-id="80dbb-143">Un client est une construction logique qui représente une instance d’Azure AD dédiée et sécurisée, généralement associée à une organisation.</span><span class="sxs-lookup"><span data-stu-id="80dbb-143">A tenant is a logical construct that represents a secure, dedicated instance of Azure AD typically associated with an organization.</span></span> <span data-ttu-id="80dbb-144">Chaque abonnement est associé à un client Azure AD.</span><span class="sxs-lookup"><span data-stu-id="80dbb-144">Each subscription is associated with an Azure AD tenant.</span></span>

<span data-ttu-id="80dbb-145">![Un locataire Azure AD associé à un abonnement](../../_images/governance-1-17.png)
*Figure 9. Un client Azure AD associé à un abonnement.*</span><span class="sxs-lookup"><span data-stu-id="80dbb-145">![An Azure AD tenant associated with a subscription](../../_images/governance-1-17.png)
*Figure 9. An Azure AD tenant associated with a subscription.*</span></span>

<span data-ttu-id="80dbb-146">Chaque requête du client pour gérer une ressource dans un abonnement spécifique requiert que l’utilisateur dispose d’un compte dans le client Azure AD associé.</span><span class="sxs-lookup"><span data-stu-id="80dbb-146">Each client request to manage a resource in a particular subscription requires that the user has an account in the associated Azure AD tenant.</span></span>

<span data-ttu-id="80dbb-147">Le contrôle suivant consiste à vérifier que l’utilisateur dispose des autorisations suffisantes pour effectuer la requête.</span><span class="sxs-lookup"><span data-stu-id="80dbb-147">The next control is a check that the user has sufficient permission to make the request.</span></span> <span data-ttu-id="80dbb-148">Les autorisations sont attribuées aux utilisateurs à l’aide du [contrôle d’accès en fonction du rôle (RBAC)](/azure/role-based-access-control/).</span><span class="sxs-lookup"><span data-stu-id="80dbb-148">Permissions are assigned to users using [role-based access control (RBAC)](/azure/role-based-access-control/).</span></span>

<span data-ttu-id="80dbb-149">![Des utilisateurs affectés aux rôles RBAC](../../_images/governance-1-18.png)
*Figure 10. Un ou plusieurs rôles RBAC sont attribués à chaque utilisateur du client.*</span><span class="sxs-lookup"><span data-stu-id="80dbb-149">![Users assigned to RBAC roles](../../_images/governance-1-18.png)
*Figure 10. Each user in the tenant is assigned one or more RBAC roles.*</span></span>

<span data-ttu-id="80dbb-150">Un rôle RBAC spécifie un ensemble d’autorisations qu’un utilisateur peut utiliser pour une ressource spécifique.</span><span class="sxs-lookup"><span data-stu-id="80dbb-150">An RBAC role specifies a set of permissions a user may take on a specific resource.</span></span> <span data-ttu-id="80dbb-151">Lorsque le rôle est attribué à l’utilisateur, ces autorisations s’appliquent.</span><span class="sxs-lookup"><span data-stu-id="80dbb-151">When the role is assigned to the user, those permissions are applied.</span></span> <span data-ttu-id="80dbb-152">Par exemple, le [rôle de **propriétaire** intégré](/azure/role-based-access-control/built-in-roles#owner) autorise un utilisateur à effectuer une action sur une ressource.</span><span class="sxs-lookup"><span data-stu-id="80dbb-152">For example, the [built-in **owner** role](/azure/role-based-access-control/built-in-roles#owner) allows a user to perform any action on a resource.</span></span>

<span data-ttu-id="80dbb-153">Le contrôle suivant vérifie que la requête est autorisée sous les paramètres spécifiés dans la [stratégie de ressources Azure](/azure/governance/policy/).</span><span class="sxs-lookup"><span data-stu-id="80dbb-153">The next control is a check that the request is allowed under the settings specified for [Azure resource policy](/azure/governance/policy/).</span></span> <span data-ttu-id="80dbb-154">Les stratégies de ressources Azure spécifient les opérations autorisées pour une ressource spécifique.</span><span class="sxs-lookup"><span data-stu-id="80dbb-154">Azure resource policies specify the operations allowed for a specific resource.</span></span> <span data-ttu-id="80dbb-155">Par exemple, une stratégie de ressources Azure peut spécifier que les utilisateurs sont uniquement autorisés à déployer un type spécifique de machine virtuelle.</span><span class="sxs-lookup"><span data-stu-id="80dbb-155">For example, an Azure resource policy can specify that users are only allowed to deploy a specific type of virtual machine.</span></span>

<span data-ttu-id="80dbb-156">![Stratégie de ressource Azure](../../_images/governance-1-19.png)
*Figure 11. Stratégie de ressource Azure.*</span><span class="sxs-lookup"><span data-stu-id="80dbb-156">![Azure resource policy](../../_images/governance-1-19.png)
*Figure 11. Azure resource policy.*</span></span>

<span data-ttu-id="80dbb-157">Le contrôle suivant consiste à vérifier que la requête ne dépasse pas une [limite d’abonnement Azure](/azure/azure-subscription-service-limits).</span><span class="sxs-lookup"><span data-stu-id="80dbb-157">The next control is a check that the request does not exceed an [Azure subscription limit](/azure/azure-subscription-service-limits).</span></span> <span data-ttu-id="80dbb-158">Par exemple, chaque abonnement dispose d’une limite de 980 groupes de ressources par abonnement.</span><span class="sxs-lookup"><span data-stu-id="80dbb-158">For example, each subscription has a limit of 980 resource groups per subscription.</span></span> <span data-ttu-id="80dbb-159">Si une requête pour déployer un groupe de ressources supplémentaire est reçue une fois la limite atteinte, elle est refusée.</span><span class="sxs-lookup"><span data-stu-id="80dbb-159">If a request is received to deploy an additional resource group once the limit has been reached, it is denied.</span></span>

<span data-ttu-id="80dbb-160">![Limites des ressources Azure](../../_images/governance-1-20.png)
*Figure 12. Limites des ressources Azure.*</span><span class="sxs-lookup"><span data-stu-id="80dbb-160">![Azure resource limits](../../_images/governance-1-20.png)
*Figure 12. Azure resource limits.*</span></span>

<span data-ttu-id="80dbb-161">Le contrôle final vérifie que la requête est comprise dans l’engagement financier associé à l’abonnement.</span><span class="sxs-lookup"><span data-stu-id="80dbb-161">The final control is a check that the request is within the financial commitment associated with the subscription.</span></span> <span data-ttu-id="80dbb-162">Par exemple, si la requête doit déployer une machine virtuelle, Azure Resource Manager vérifie que l’abonnement dispose des informations de paiement suffisantes.</span><span class="sxs-lookup"><span data-stu-id="80dbb-162">For example, if the request is to deploy a virtual machine, Azure Resource Manager verifies that the subscription has sufficient payment information.</span></span>

<span data-ttu-id="80dbb-163">![Un engagement financier associé à un abonnement](../../_images/governance-1-21.png)
*Figure 13. Un engagement financier est associé à un abonnement.*</span><span class="sxs-lookup"><span data-stu-id="80dbb-163">![A financial commitment associated with a subscription](../../_images/governance-1-21.png)
*Figure 13. A financial commitment is associated with a subscription.*</span></span>

## <a name="summary"></a><span data-ttu-id="80dbb-164">Résumé</span><span class="sxs-lookup"><span data-stu-id="80dbb-164">Summary</span></span>

<span data-ttu-id="80dbb-165">Dans cet article, vous avez étudié la gestion des accès aux ressources dans Azure à l’aide d’Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="80dbb-165">In this article, you learned about how resource access is managed in Azure using Azure Resource Manager.</span></span>

## <a name="next-steps"></a><span data-ttu-id="80dbb-166">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="80dbb-166">Next steps</span></span>

<span data-ttu-id="80dbb-167">Maintenant que vous comprenez comment gérer l’accès aux ressources dans Azure, poursuivez pour en savoir plus sur la conception d’un modèle de gouvernance [pour une charge de travail simple](governance-simple-workload.md) ou pour [plusieurs équipes](governance-multiple-teams.md) à l’aide de ces services.</span><span class="sxs-lookup"><span data-stu-id="80dbb-167">Now that you understand how to manage resource access in Azure, move on to learn how to design a governance model [for a simple workload](governance-simple-workload.md) or for [multiple teams](governance-multiple-teams.md) using these services.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="80dbb-168">Une vue d’ensemble de la gouvernance</span><span class="sxs-lookup"><span data-stu-id="80dbb-168">An overview of governance</span></span>](../overview.md)
