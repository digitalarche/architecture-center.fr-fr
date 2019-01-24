---
title: 'Adoption du cloud d’entreprise : Gestion de l’accès aux ressources dans Azure'
description: 'Présentation des constructions de gestion des accès aux ressources dans Azure : Azure Resource Manager, abonnements, groupes de ressources et ressources'
author: petertaylor9999
ms.date: 09/10/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.openlocfilehash: 9f1d28cfeb062f44b2c58cd52c58f163d6de5b9d
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54483069"
---
# <a name="enterprise-cloud-adoption-resource-access-management-in-azure"></a><span data-ttu-id="ecba9-103">Adoption du cloud d’entreprise : Gestion de l’accès aux ressources dans Azure</span><span class="sxs-lookup"><span data-stu-id="ecba9-103">Enterprise Cloud Adoption: Resource access management in Azure</span></span>

<span data-ttu-id="ecba9-104">Comme vous l’avez appris dans l’article [Qu’est-ce que la gouvernance de ressource ?](what-is-governance.md), la gouvernance consiste à gérer, surveiller et contrôler en continu les ressources Azure afin d’atteindre les objectifs et de satisfaire aux exigences de votre organisation.</span><span class="sxs-lookup"><span data-stu-id="ecba9-104">In [what is resource governance?](what-is-governance.md), you learned that governance refers to the ongoing process of managing, monitoring, and auditing the use of Azure resources to meet the goals and requirements of your organization.</span></span> <span data-ttu-id="ecba9-105">Avant de découvrir comment concevoir un modèle de gouvernance, il est important de comprendre les contrôles de gestion des accès aux ressources dans Azure.</span><span class="sxs-lookup"><span data-stu-id="ecba9-105">Before you move on to learn how to design a governance model, it's important to understand the resource access management controls in Azure.</span></span> <span data-ttu-id="ecba9-106">La configuration de ces contrôles de gestion des accès aux ressources constitue la base de votre modèle de gouvernance.</span><span class="sxs-lookup"><span data-stu-id="ecba9-106">The configuration of these resource access management controls forms the basis of your governance model.</span></span>

<span data-ttu-id="ecba9-107">Commençons par examiner de plus près le déploiement des ressources dans Azure.</span><span class="sxs-lookup"><span data-stu-id="ecba9-107">Let's begin by taking a closer look at how resources are deployed in Azure.</span></span> 

## <a name="what-is-an-azure-resource"></a><span data-ttu-id="ecba9-108">Qu’est-ce qu’une ressource Azure ?</span><span class="sxs-lookup"><span data-stu-id="ecba9-108">What is an Azure resource?</span></span>

<span data-ttu-id="ecba9-109">Dans Azure, le terme **ressource** fait référence à une entité gérée par Azure.</span><span class="sxs-lookup"><span data-stu-id="ecba9-109">In Azure, the term **resource** refers to an entity managed by Azure.</span></span> <span data-ttu-id="ecba9-110">Par exemple, les machines virtuelles, les réseaux virtuels et les comptes de stockage sont tous considérés comme des ressources Azure.</span><span class="sxs-lookup"><span data-stu-id="ecba9-110">For example, virtual machines, virtual networks, and storage accounts are all referred to as Azure resources.</span></span>

![](../_images/governance-1-9.png)   
<span data-ttu-id="ecba9-111">*Figure 1 : Une ressource.*</span><span class="sxs-lookup"><span data-stu-id="ecba9-111">*Figure 1. A resource.*</span></span>

## <a name="what-is-an-azure-resource-group"></a><span data-ttu-id="ecba9-112">Qu’est-ce qu’un groupe de ressources Azure ?</span><span class="sxs-lookup"><span data-stu-id="ecba9-112">What is an Azure resource group?</span></span>

<span data-ttu-id="ecba9-113">Chaque ressource dans Azure doit appartenir à un [groupe de ressources](/azure/azure-resource-manager/resource-group-overview#resource-groups).</span><span class="sxs-lookup"><span data-stu-id="ecba9-113">Each resource in Azure must belong to a [resource group](/azure/azure-resource-manager/resource-group-overview#resource-groups).</span></span> <span data-ttu-id="ecba9-114">Un groupe de ressources est simplement une construction logique qui regroupe plusieurs ressources afin qu’elles puissent être gérées en tant qu’entité unique.</span><span class="sxs-lookup"><span data-stu-id="ecba9-114">A resource group is simply a logical construct that groups multiple resources together so they can be managed as a single entity.</span></span> <span data-ttu-id="ecba9-115">Par exemple, des ressources qui partagent un cycle de vie similaire, tels que les ressources d’une [application multi-niveau](/azure/architecture/guide/architecture-styles/n-tier), peuvent être créées ou supprimées en tant que groupe.</span><span class="sxs-lookup"><span data-stu-id="ecba9-115">For example, resources that share a similar lifecycle, such as the resources for an [n-tier application](/azure/architecture/guide/architecture-styles/n-tier) may be created or deleted as a group.</span></span> 

![](../_images/governance-1-10.png)   
<span data-ttu-id="ecba9-116">*Figure 2 : Un groupe de ressources contient une ressource.*</span><span class="sxs-lookup"><span data-stu-id="ecba9-116">*Figure 2. A resource group contains a resource.*</span></span> 

<span data-ttu-id="ecba9-117">Les groupes de ressources et les ressources qu’ils contiennent sont associés à un **abonnement** Azure.</span><span class="sxs-lookup"><span data-stu-id="ecba9-117">Resource groups and the resources they contain are associated with an Azure **subscription**.</span></span> 

## <a name="what-is-an-azure-subscription"></a><span data-ttu-id="ecba9-118">Qu’est-ce qu’un abonnement Azure ?</span><span class="sxs-lookup"><span data-stu-id="ecba9-118">What is an Azure subscription?</span></span>

<span data-ttu-id="ecba9-119">Un abonnement Azure est similaire à un groupe de ressources : il s’agit d’une construction logique qui regroupe des groupes de ressources et leurs ressources.</span><span class="sxs-lookup"><span data-stu-id="ecba9-119">An Azure subscription is similar to a resource group in that it's a logical construct that groups together resource groups and their resources.</span></span> <span data-ttu-id="ecba9-120">Toutefois, un abonnement Azure est également associé aux contrôles utilisés par Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="ecba9-120">However, an Azure subscription is also associated with the controls used by Azure resource manager.</span></span> <span data-ttu-id="ecba9-121">Qu’est-ce que cela signifie ?</span><span class="sxs-lookup"><span data-stu-id="ecba9-121">What does this mean?</span></span> <span data-ttu-id="ecba9-122">Examinons plus en détail Azure Resource Manager pour en savoir plus sur sa relation avec un abonnement Azure.</span><span class="sxs-lookup"><span data-stu-id="ecba9-122">Let's take a closer look at Azure resource manager to learn about the relationship between it and an Azure subscription.</span></span>

![](../_images/governance-1-11.png)   
<span data-ttu-id="ecba9-123">*Figure 3 : Un abonnement Azure.*</span><span class="sxs-lookup"><span data-stu-id="ecba9-123">*Figure 3. An Azure subscription.*</span></span>

## <a name="what-is-azure-resource-manager"></a><span data-ttu-id="ecba9-124">Qu’est-ce qu’Azure Resource Manager ?</span><span class="sxs-lookup"><span data-stu-id="ecba9-124">What is Azure resource manager?</span></span>

<span data-ttu-id="ecba9-125">Dans [Comment fonctionne Azure ?](what-is-azure.md), vous avez appris qu’Azure comprend un « serveur frontal » avec de nombreux services qui orchestrent toutes les fonctions d’Azure.</span><span class="sxs-lookup"><span data-stu-id="ecba9-125">In [how does Azure work?](what-is-azure.md) you learned that Azure includes a "front end" with many services that orchestrate all the functions of Azure.</span></span> <span data-ttu-id="ecba9-126">Un de ces services est le gestionnaire de ressources [Azure Resource Manager](/azure/azure-resource-manager/), et ce service héberge l’API RESTful utilisée par les clients pour gérer les ressources.</span><span class="sxs-lookup"><span data-stu-id="ecba9-126">One of these services is [Azure resource manager](/azure/azure-resource-manager/), and this service hosts the RESTful API used by clients to manage resources.</span></span> 

![](../_images/governance-1-12.png)   
<span data-ttu-id="ecba9-127">*Figure 4 : Azure Resource Manager.*</span><span class="sxs-lookup"><span data-stu-id="ecba9-127">*Figure 4. Azure resource manager.*</span></span>

<span data-ttu-id="ecba9-128">La figure suivante montre trois clients : [PowerShell](/powershell/azure/overview), le [Portail Azure](https://portal.azure.com) et l’[interface CLI (interface de ligne de commande) Azure](/cli/azure) :</span><span class="sxs-lookup"><span data-stu-id="ecba9-128">The following figure shows three clients: [Powershell](/powershell/azure/overview), [the Azure portal](https://portal.azure.com), and the [Azure command line interface (CLI)](/cli/azure):</span></span>

![](../_images/governance-1-13.png)   
<span data-ttu-id="ecba9-129">*Figure 5 : Les clients Azure se connectent à l’API RESTful d’Azure Resource Manager.*</span><span class="sxs-lookup"><span data-stu-id="ecba9-129">*Figure 5. Azure clients connect to the Azure resource manager RESTful API.*</span></span>

<span data-ttu-id="ecba9-130">Bien que ces clients se connectent à Azure Resource Manager à l’aide de l’API RESTful, Azure Resource Manager n’inclut pas de fonctionnalités pour gérer directement les ressources.</span><span class="sxs-lookup"><span data-stu-id="ecba9-130">While these clients connect to Azure resource manager using the RESTful API, Azure resource manager does not include functionality to manage resources directly.</span></span> <span data-ttu-id="ecba9-131">Au lieu de cela, la plupart des types de ressources Azure ont leur propre [**fournisseur de ressources**](/azure/azure-resource-manager/resource-group-overview#terminology).</span><span class="sxs-lookup"><span data-stu-id="ecba9-131">Rather, most resource types in Azure have their own [**resource provider**](/azure/azure-resource-manager/resource-group-overview#terminology).</span></span> 

![](../_images/governance-1-14.png)   
<span data-ttu-id="ecba9-132">*Figure 6 : Fournisseurs de ressources Azure.*</span><span class="sxs-lookup"><span data-stu-id="ecba9-132">*Figure 6. Azure resource providers.*</span></span>

<span data-ttu-id="ecba9-133">Quand un client envoie une requête pour gérer une ressource spécifique, le gestionnaire de ressources Azure Resource Manager se connecte au fournisseur de ressources de ce type de ressources pour terminer la requête.</span><span class="sxs-lookup"><span data-stu-id="ecba9-133">When a client makes a request to manage a specific resource, Azure resource manager connects to the resource provider for that resource type to complete the request.</span></span> <span data-ttu-id="ecba9-134">Par exemple, si un client envoie une requête pour gérer une ressource de machine virtuelle, Azure Resource Manager se connecte au fournisseur de ressources **Microsoft.compute**.</span><span class="sxs-lookup"><span data-stu-id="ecba9-134">For example, if a client makes a request to manage a virtual machine resource, Azure resource manager connects to the **Microsoft.compute** resource provider.</span></span> 

![](../_images/governance-1-15.png)   
<span data-ttu-id="ecba9-135">*Figure 7 : Azure Resource Manager se connecte au fournisseur de ressources **Microsoft.compute** pour gérer la ressource spécifiée dans la requête du client.*</span><span class="sxs-lookup"><span data-stu-id="ecba9-135">*Figure 7. Azure resource manager connects to the **Microsoft.compute** resource provider to manage the resource specified in the client request.*</span></span>

<span data-ttu-id="ecba9-136">Le gestionnaire de ressources Azure Resource Manager demande au client de spécifier un identificateur pour l’abonnement et le groupe de ressources, afin de gérer la ressource de machine virtuelle.</span><span class="sxs-lookup"><span data-stu-id="ecba9-136">Azure resource manager requires the client to specify an identifier for both the subscription and the resource group in order to manage the virtual machine resource.</span></span> 

<span data-ttu-id="ecba9-137">Maintenant que vous comprenez le fonctionnement d’Azure Resource Manager, revenons à notre discussion sur la façon dont un abonnement Azure est associé aux contrôles utilisés par Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="ecba9-137">Now that you have an understanding of how Azure resource manager works, let's return to our discussion of how an Azure subscription is associated with the controls used by Azure resource manager.</span></span> <span data-ttu-id="ecba9-138">Avant qu’une requête de gestion des ressources ne soit exécutée par Azure Resource Manager, plusieurs contrôles sont vérifiés.</span><span class="sxs-lookup"><span data-stu-id="ecba9-138">Before any resource management request can be executed by Azure resource manager, a set of controls are checked.</span></span> 

<span data-ttu-id="ecba9-139">Le premier contrôle est qu’une requête doit être effectuée par un utilisateur validé, et le gestionnaire de ressources Azure Resource Manager a une relation de confiance avec [Azure Active Directory (Azure AD)](/azure/active-directory/) pour fournir des fonctionnalités d’identité utilisateur.</span><span class="sxs-lookup"><span data-stu-id="ecba9-139">The first control is that a request must be made by a validated user, and Azure resource manager has a trusted relationship with [Azure Active Directory (Azure AD)](/azure/active-directory/) to provide user identity functionality.</span></span>

![](../_images/governance-1-16.png)   
<span data-ttu-id="ecba9-140">*Figure 8 : Azure Active Directory.*</span><span class="sxs-lookup"><span data-stu-id="ecba9-140">*Figure 8. Azure Active Directory.*</span></span>

<span data-ttu-id="ecba9-141">Dans Azure AD, les utilisateurs sont segmentés sous forme de **clients**.</span><span class="sxs-lookup"><span data-stu-id="ecba9-141">In Azure AD, users are segmented into **tenants**.</span></span> <span data-ttu-id="ecba9-142">Un client est une construction logique qui représente une instance d’Azure AD dédiée et sécurisée, généralement associée à une organisation.</span><span class="sxs-lookup"><span data-stu-id="ecba9-142">A tenant is a logical construct that represents a secure, dedicated instance of Azure AD typically associated with an organization.</span></span> <span data-ttu-id="ecba9-143">Chaque abonnement est associé à un client Azure AD.</span><span class="sxs-lookup"><span data-stu-id="ecba9-143">Each subscription is associated with an Azure AD tenant.</span></span>

![](../_images/governance-1-17.png)   
<span data-ttu-id="ecba9-144">*Figure 9 : Un client Azure AD associé à un abonnement.*</span><span class="sxs-lookup"><span data-stu-id="ecba9-144">*Figure 9. An Azure AD tenant associated with a subscription.*</span></span>

<span data-ttu-id="ecba9-145">Chaque requête du client pour gérer une ressource dans un abonnement spécifique requiert que l’utilisateur dispose d’un compte dans le client Azure AD associé.</span><span class="sxs-lookup"><span data-stu-id="ecba9-145">Each client request to manage a resource in a particular subscription requires that the user has an account in the associated Azure AD tenant.</span></span> 

<span data-ttu-id="ecba9-146">Le contrôle suivant consiste à vérifier que l’utilisateur dispose des autorisations suffisantes pour effectuer la requête.</span><span class="sxs-lookup"><span data-stu-id="ecba9-146">The next control is a check that the user has sufficient permission to make the request.</span></span> <span data-ttu-id="ecba9-147">Les autorisations sont attribuées aux utilisateurs à l’aide du [contrôle d’accès en fonction du rôle (RBAC)](/azure/role-based-access-control/).</span><span class="sxs-lookup"><span data-stu-id="ecba9-147">Permissions are assigned to users using [role-based access control (RBAC)](/azure/role-based-access-control/).</span></span>

![](../_images/governance-1-18.png)   
<span data-ttu-id="ecba9-148">*Figure 10 : Un ou plusieurs rôles RBAC sont attribués à chaque utilisateur du client.*</span><span class="sxs-lookup"><span data-stu-id="ecba9-148">*Figure 10. Each user in the tenant is assigned one or more RBAC roles.*</span></span>

<span data-ttu-id="ecba9-149">Un rôle RBAC spécifie un ensemble d’autorisations qu’un utilisateur peut utiliser pour une ressource spécifique.</span><span class="sxs-lookup"><span data-stu-id="ecba9-149">An RBAC role specifies a set of permissions a user may take on a specific resource.</span></span> <span data-ttu-id="ecba9-150">Lorsque le rôle est attribué à l’utilisateur, ces autorisations s’appliquent.</span><span class="sxs-lookup"><span data-stu-id="ecba9-150">When the role is assigned to the user, those permissions are applied.</span></span> <span data-ttu-id="ecba9-151">Par exemple, le [rôle de **propriétaire** intégré](/azure/role-based-access-control/built-in-roles#owner) autorise un utilisateur à effectuer une action sur une ressource.</span><span class="sxs-lookup"><span data-stu-id="ecba9-151">For example, the [built-in **owner** role](/azure/role-based-access-control/built-in-roles#owner) allows a user to perform any action on a resource.</span></span>

<span data-ttu-id="ecba9-152">Le contrôle suivant vérifie que la requête est autorisée sous les paramètres spécifiés dans la [stratégie de ressources Azure](/azure/governance/policy/).</span><span class="sxs-lookup"><span data-stu-id="ecba9-152">The next control is a check that the request is allowed under the settings specified for [Azure resource policy](/azure/governance/policy/).</span></span> <span data-ttu-id="ecba9-153">Les stratégies de ressources Azure spécifient les opérations autorisées pour une ressource spécifique.</span><span class="sxs-lookup"><span data-stu-id="ecba9-153">Azure resource policies specify the operations allowed for a specific resource.</span></span> <span data-ttu-id="ecba9-154">Par exemple, une stratégie de ressources Azure peut spécifier que les utilisateurs sont uniquement autorisés à déployer un type spécifique de machine virtuelle.</span><span class="sxs-lookup"><span data-stu-id="ecba9-154">For example, an Azure resource policy can specify that users are only allowed to deploy a specific type of virtual machine.</span></span>

![](../_images/governance-1-19.png)   
<span data-ttu-id="ecba9-155">*Figure 11 : Stratégie de ressource Azure.*</span><span class="sxs-lookup"><span data-stu-id="ecba9-155">*Figure 11. Azure resource policy.*</span></span>

<span data-ttu-id="ecba9-156">Le contrôle suivant consiste à vérifier que la requête ne dépasse pas une [limite d’abonnement Azure](/azure/azure-subscription-service-limits).</span><span class="sxs-lookup"><span data-stu-id="ecba9-156">The next control is a check that the request does not exceed an [Azure subscription limit](/azure/azure-subscription-service-limits).</span></span> <span data-ttu-id="ecba9-157">Par exemple, chaque abonnement dispose d’une limite de 980 groupes de ressources par abonnement.</span><span class="sxs-lookup"><span data-stu-id="ecba9-157">For example, each subscription has a limit of 980 resource groups per subscription.</span></span> <span data-ttu-id="ecba9-158">Si une requête pour déployer un groupe de ressources supplémentaire est reçue une fois la limite atteinte, elle est refusée.</span><span class="sxs-lookup"><span data-stu-id="ecba9-158">If a request is received to deploy an additional resource group once the limit has been reached, it is denied.</span></span>

![](../_images/governance-1-20.png)   
<span data-ttu-id="ecba9-159">*Figure 12 : Limites des ressources Azure.*</span><span class="sxs-lookup"><span data-stu-id="ecba9-159">*Figure 12. Azure resource limits.*</span></span> 

<span data-ttu-id="ecba9-160">Le contrôle final vérifie que la requête est comprise dans l’engagement financier associé à l’abonnement.</span><span class="sxs-lookup"><span data-stu-id="ecba9-160">The final control is a check that the request is within the financial commitment associated with the subscription.</span></span> <span data-ttu-id="ecba9-161">Par exemple, si la requête requiert le déploiement d’une machine virtuelle, Azure Resource Manager vérifie que l’abonnement dispose des informations de paiement suffisantes.</span><span class="sxs-lookup"><span data-stu-id="ecba9-161">For example, if the request is to deploy a virtual machine, Azure resource manager verifies that the subscription has sufficient payment information.</span></span>

![](../_images/governance-1-21.png)   
<span data-ttu-id="ecba9-162">*Figure 13 : Un engagement financier est associé à un abonnement.*</span><span class="sxs-lookup"><span data-stu-id="ecba9-162">*Figure 13. A financial commitment is associated with a subscription.*</span></span>

## <a name="summary"></a><span data-ttu-id="ecba9-163">Résumé</span><span class="sxs-lookup"><span data-stu-id="ecba9-163">Summary</span></span>

<span data-ttu-id="ecba9-164">Dans cet article, vous avez découvert comment l’accès aux ressources est géré dans Azure à l’aide d’Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="ecba9-164">In this article, you learned about how resource access is managed in Azure using Azure resource manager.</span></span>

## <a name="next-steps"></a><span data-ttu-id="ecba9-165">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="ecba9-165">Next steps</span></span>

<span data-ttu-id="ecba9-166">Maintenant que vous comprenez comment l’accès aux ressources est géré dans Azure, poursuivez pour en savoir plus sur la conception d’un modèle de gouvernance pour [une seule](../governance/governance-single-team.md) ou [plusieurs équipes](../governance/governance-multiple-teams.md) à l’aide de ces services.</span><span class="sxs-lookup"><span data-stu-id="ecba9-166">Now that you understand how resource access is managed in Azure, move on to learn how to design a governance model [for a single team](../governance/governance-single-team.md) or [multiple teams](../governance/governance-multiple-teams.md) using these services.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="ecba9-167">Une vue d’ensemble de la gouvernance</span><span class="sxs-lookup"><span data-stu-id="ecba9-167">An overview of governance</span></span>](../governance/overview.md)
