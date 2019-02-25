---
title: 'Framework d’adoption du cloud : Gestion de l’accès aux ressources dans Azure'
description: Explication de la structure de gestion des accès aux ressources dans Azure – Azure Resource Manager, abonnements, groupes de ressources et ressources.
author: petertaylor9999
ms.date: 2/11/2019
ms.openlocfilehash: f23540a03c82fbc46872645ac0fd82d574db353a
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55898117"
---
# <a name="resource-access-management-in-azure"></a><span data-ttu-id="aa595-103">Gestion de l’accès aux ressources dans Azure</span><span class="sxs-lookup"><span data-stu-id="aa595-103">Resource access management in Azure</span></span>

<span data-ttu-id="aa595-104">Comme vous l’avez appris dans l’article [Qu’est-ce que la gouvernance de ressource ?](what-is-governance.md), la gouvernance consiste à gérer, surveiller et contrôler en continu les ressources Azure afin d’atteindre les objectifs et de satisfaire aux exigences de votre organisation.</span><span class="sxs-lookup"><span data-stu-id="aa595-104">In [what is resource governance?](what-is-governance.md), you learned that governance refers to the ongoing process of managing, monitoring, and auditing the use of Azure resources to meet the goals and requirements of your organization.</span></span> <span data-ttu-id="aa595-105">Avant de découvrir comment concevoir un modèle de gouvernance, il est important de comprendre les contrôles de gestion des accès aux ressources dans Azure.</span><span class="sxs-lookup"><span data-stu-id="aa595-105">Before you move on to learn how to design a governance model, it's important to understand the resource access management controls in Azure.</span></span> <span data-ttu-id="aa595-106">La configuration de ces contrôles de gestion des accès aux ressources constitue la base de votre modèle de gouvernance.</span><span class="sxs-lookup"><span data-stu-id="aa595-106">The configuration of these resource access management controls forms the basis of your governance model.</span></span>

<span data-ttu-id="aa595-107">Commençons par examiner de plus près le déploiement des ressources dans Azure.</span><span class="sxs-lookup"><span data-stu-id="aa595-107">Let's begin by taking a closer look at how resources are deployed in Azure.</span></span>

<!-- markdownlint-disable MD026 -->

## <a name="what-is-an-azure-resource"></a><span data-ttu-id="aa595-108">Qu’est-ce qu’une ressource Azure ?</span><span class="sxs-lookup"><span data-stu-id="aa595-108">What is an Azure resource?</span></span>

<span data-ttu-id="aa595-109">Dans Azure, le terme **ressource** fait référence à une entité gérée par Azure.</span><span class="sxs-lookup"><span data-stu-id="aa595-109">In Azure, the term **resource** refers to an entity managed by Azure.</span></span> <span data-ttu-id="aa595-110">Par exemple, les machines virtuelles, les réseaux virtuels et les comptes de stockage sont tous considérés comme des ressources Azure.</span><span class="sxs-lookup"><span data-stu-id="aa595-110">For example, virtual machines, virtual networks, and storage accounts are all referred to as Azure resources.</span></span>

<span data-ttu-id="aa595-111">![](../_images/governance-1-9.png)
*Figure 1. Une ressource.*</span><span class="sxs-lookup"><span data-stu-id="aa595-111">![](../_images/governance-1-9.png)
*Figure 1. A resource.*</span></span>

## <a name="what-is-an-azure-resource-group"></a><span data-ttu-id="aa595-112">Qu’est-ce qu’un groupe de ressources Azure ?</span><span class="sxs-lookup"><span data-stu-id="aa595-112">What is an Azure resource group?</span></span>

<span data-ttu-id="aa595-113">Chaque ressource dans Azure doit appartenir à un [groupe de ressources](/azure/azure-resource-manager/resource-group-overview#resource-groups).</span><span class="sxs-lookup"><span data-stu-id="aa595-113">Each resource in Azure must belong to a [resource group](/azure/azure-resource-manager/resource-group-overview#resource-groups).</span></span> <span data-ttu-id="aa595-114">Un groupe de ressources est simplement une construction logique qui regroupe plusieurs ressources afin qu’elles puissent être gérées en tant qu’entité unique.</span><span class="sxs-lookup"><span data-stu-id="aa595-114">A resource group is simply a logical construct that groups multiple resources together so they can be managed as a single entity.</span></span> <span data-ttu-id="aa595-115">Par exemple, des ressources qui partagent un cycle de vie similaire, tels que les ressources d’une [application multi-niveau](/azure/architecture/guide/architecture-styles/n-tier), peuvent être créées ou supprimées en tant que groupe.</span><span class="sxs-lookup"><span data-stu-id="aa595-115">For example, resources that share a similar lifecycle, such as the resources for an [n-tier application](/azure/architecture/guide/architecture-styles/n-tier) may be created or deleted as a group.</span></span>

<span data-ttu-id="aa595-116">![](../_images/governance-1-10.png)
*Figure 2. Un groupe de ressources contient une ressource.*</span><span class="sxs-lookup"><span data-stu-id="aa595-116">![](../_images/governance-1-10.png)
*Figure 2. A resource group contains a resource.*</span></span>

<span data-ttu-id="aa595-117">Les groupes de ressources et les ressources qu’ils contiennent sont associés à un **abonnement** Azure.</span><span class="sxs-lookup"><span data-stu-id="aa595-117">Resource groups and the resources they contain are associated with an Azure **subscription**.</span></span>

## <a name="what-is-an-azure-subscription"></a><span data-ttu-id="aa595-118">Qu’est-ce qu’un abonnement Azure ?</span><span class="sxs-lookup"><span data-stu-id="aa595-118">What is an Azure subscription?</span></span>

<span data-ttu-id="aa595-119">Un abonnement Azure est similaire à un groupe de ressources : il s’agit d’une construction logique qui regroupe des groupes de ressources et leurs ressources.</span><span class="sxs-lookup"><span data-stu-id="aa595-119">An Azure subscription is similar to a resource group in that it's a logical construct that groups together resource groups and their resources.</span></span> <span data-ttu-id="aa595-120">Toutefois, un abonnement Azure est également associé aux contrôles utilisés par Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="aa595-120">However, an Azure subscription is also associated with the controls used by Azure Resource Manager.</span></span> <span data-ttu-id="aa595-121">Qu’est-ce que cela signifie ?</span><span class="sxs-lookup"><span data-stu-id="aa595-121">What does this mean?</span></span> <span data-ttu-id="aa595-122">Examinons plus en détail Resource Manager pour en savoir plus sur sa relation avec un abonnement Azure.</span><span class="sxs-lookup"><span data-stu-id="aa595-122">Let's take a closer look at Resource Manager to learn about the relationship between it and an Azure subscription.</span></span>

<span data-ttu-id="aa595-123">![](../_images/governance-1-11.png)
*Figure 3. Un abonnement Azure.*</span><span class="sxs-lookup"><span data-stu-id="aa595-123">![](../_images/governance-1-11.png)
*Figure 3. An Azure subscription.*</span></span>

## <a name="what-is-azure-resource-manager"></a><span data-ttu-id="aa595-124">Qu’est-ce qu’Azure Resource Manager ?</span><span class="sxs-lookup"><span data-stu-id="aa595-124">What is Azure Resource Manager?</span></span>

<span data-ttu-id="aa595-125">Dans [Comment fonctionne Azure ?](what-is-azure.md), vous avez appris qu’Azure comprend un « serveur frontal » avec de nombreux services qui orchestrent toutes les fonctions d’Azure.</span><span class="sxs-lookup"><span data-stu-id="aa595-125">In [how does Azure work?](what-is-azure.md) you learned that Azure includes a "front end" with many services that orchestrate all the functions of Azure.</span></span> <span data-ttu-id="aa595-126">Un de ces services est le gestionnaire de ressources [Resource Manager](/azure/azure-resource-manager/), et ce service héberge l’API RESTful utilisée par les clients pour gérer les ressources.</span><span class="sxs-lookup"><span data-stu-id="aa595-126">One of these services is [Resource Manager](/azure/azure-resource-manager/), and this service hosts the RESTful API used by clients to manage resources.</span></span>

<span data-ttu-id="aa595-127">![](../_images/governance-1-12.png)
*Figure 4. Gestionnaire de ressources Azure Resource Manager.*</span><span class="sxs-lookup"><span data-stu-id="aa595-127">![](../_images/governance-1-12.png)
*Figure 4. Azure resource manager.*</span></span>

<span data-ttu-id="aa595-128">La figure suivante montre trois clients : [PowerShell](/powershell/azure/overview), le [Portail Azure](https://portal.azure.com) et l’[interface CLI (interface de ligne de commande) Azure](/cli/azure) :</span><span class="sxs-lookup"><span data-stu-id="aa595-128">The following figure shows three clients: [PowerShell](/powershell/azure/overview), the [Azure portal](https://portal.azure.com), and the [Azure command line interface (CLI)](/cli/azure):</span></span>

<span data-ttu-id="aa595-129">![](../_images/governance-1-13.png)
*Figure 5. Les clients Azure se connectent à l’API RESTful de Resource Manager.*</span><span class="sxs-lookup"><span data-stu-id="aa595-129">![](../_images/governance-1-13.png)
*Figure 5. Azure clients connect to the Resource Manager RESTful API.*</span></span>

<span data-ttu-id="aa595-130">Bien que ces clients se connectent à Resource Manager à l’aide de l’API RESTful, Resource Manager n’inclut pas de fonctionnalités pour gérer directement les ressources.</span><span class="sxs-lookup"><span data-stu-id="aa595-130">While these clients connect to Resource Manager using the RESTful API, Resource Manager does not include functionality to manage resources directly.</span></span> <span data-ttu-id="aa595-131">Au lieu de cela, la plupart des types de ressources Azure ont leur propre [**fournisseur de ressources**](/azure/azure-resource-manager/resource-group-overview#terminology).</span><span class="sxs-lookup"><span data-stu-id="aa595-131">Rather, most resource types in Azure have their own [**resource provider**](/azure/azure-resource-manager/resource-group-overview#terminology).</span></span>

<span data-ttu-id="aa595-132">\*
*Figure 6. Fournisseurs de ressources Azure.*</span><span class="sxs-lookup"><span data-stu-id="aa595-132">![](../_images/governance-1-14.png)
*Figure 6. Azure resource providers.*</span></span>

<span data-ttu-id="aa595-133">Quand un client envoie une requête pour gérer une ressource spécifique, le gestionnaire de ressources Resource Manager se connecte au fournisseur de ressources de ce type de ressources pour terminer la requête.</span><span class="sxs-lookup"><span data-stu-id="aa595-133">When a client makes a request to manage a specific resource, Resource Manager connects to the resource provider for that resource type to complete the request.</span></span> <span data-ttu-id="aa595-134">Par exemple, si un client envoie une requête pour gérer une ressource de machine virtuelle, Resource Manager se connecte au fournisseur de ressources **Microsoft.Compute**.</span><span class="sxs-lookup"><span data-stu-id="aa595-134">For example, if a client makes a request to manage a virtual machine resource, Resource Manager connects to the **Microsoft.Compute** resource provider.</span></span>

<span data-ttu-id="aa595-135">![](../_images/governance-1-15.png)
*Figure 7. Resource Manager se connecte au fournisseur de ressources **Microsoft.Compute** pour gérer la ressource spécifiée dans la requête du client.*</span><span class="sxs-lookup"><span data-stu-id="aa595-135">![](../_images/governance-1-15.png)
*Figure 7. Resource Manager connects to the **Microsoft.Compute** resource provider to manage the resource specified in the client request.*</span></span>

<span data-ttu-id="aa595-136">Le gestionnaire de ressources Resource Manager demande au client de spécifier un identificateur pour l’abonnement et le groupe de ressources afin de gérer la ressource de machine virtuelle.</span><span class="sxs-lookup"><span data-stu-id="aa595-136">Resource Manager requires the client to specify an identifier for both the subscription and the resource group in order to manage the virtual machine resource.</span></span>

<span data-ttu-id="aa595-137">Maintenant que vous comprenez le fonctionnement de Resource Manager, revenons à notre discussion sur l’association entre un abonnement Azure et les contrôles utilisés par Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="aa595-137">Now that you have an understanding of how Resource Manager works, let's return to our discussion of how an Azure subscription is associated with the controls used by Azure resource manager.</span></span> <span data-ttu-id="aa595-138">Avant qu’une requête de gestion des ressources ne soit exécutée par Resource Manager, plusieurs contrôles sont vérifiés.</span><span class="sxs-lookup"><span data-stu-id="aa595-138">Before any resource management request can be executed by Resource Manager, a set of controls are checked.</span></span>

<span data-ttu-id="aa595-139">Pour le premier contrôle, une requête doit être effectuée par un utilisateur validé, et le gestionnaire de ressources Resource Manager entretient une relation de confiance avec [Azure Active Directory](/azure/active-directory/) (Azure AD) pour fournir des fonctionnalités d’identité utilisateur.</span><span class="sxs-lookup"><span data-stu-id="aa595-139">The first control is that a request must be made by a validated user, and Resource Manager has a trusted relationship with [Azure Active Directory](/azure/active-directory/) (Azure AD) to provide user identity functionality.</span></span>

<span data-ttu-id="aa595-140">![](../_images/governance-1-16.png)
*Figure 8. Azure Active Directory.*</span><span class="sxs-lookup"><span data-stu-id="aa595-140">![](../_images/governance-1-16.png)
*Figure 8. Azure Active Directory.*</span></span>

<span data-ttu-id="aa595-141">Dans Azure AD, les utilisateurs sont segmentés sous forme de **clients**.</span><span class="sxs-lookup"><span data-stu-id="aa595-141">In Azure AD, users are segmented into **tenants**.</span></span> <span data-ttu-id="aa595-142">Un client est une construction logique qui représente une instance d’Azure AD dédiée et sécurisée, généralement associée à une organisation.</span><span class="sxs-lookup"><span data-stu-id="aa595-142">A tenant is a logical construct that represents a secure, dedicated instance of Azure AD typically associated with an organization.</span></span> <span data-ttu-id="aa595-143">Chaque abonnement est associé à un client Azure AD.</span><span class="sxs-lookup"><span data-stu-id="aa595-143">Each subscription is associated with an Azure AD tenant.</span></span>

<span data-ttu-id="aa595-144">![](../_images/governance-1-17.png)
*Figure 9. Un client Azure AD associé à un abonnement.*</span><span class="sxs-lookup"><span data-stu-id="aa595-144">![](../_images/governance-1-17.png)
*Figure 9. An Azure AD tenant associated with a subscription.*</span></span>

<span data-ttu-id="aa595-145">Chaque requête du client pour gérer une ressource dans un abonnement spécifique requiert que l’utilisateur dispose d’un compte dans le client Azure AD associé.</span><span class="sxs-lookup"><span data-stu-id="aa595-145">Each client request to manage a resource in a particular subscription requires that the user has an account in the associated Azure AD tenant.</span></span>

<span data-ttu-id="aa595-146">Le contrôle suivant consiste à vérifier que l’utilisateur dispose des autorisations suffisantes pour effectuer la requête.</span><span class="sxs-lookup"><span data-stu-id="aa595-146">The next control is a check that the user has sufficient permission to make the request.</span></span> <span data-ttu-id="aa595-147">Les autorisations sont attribuées aux utilisateurs à l’aide du [contrôle d’accès en fonction du rôle (RBAC)](/azure/role-based-access-control/).</span><span class="sxs-lookup"><span data-stu-id="aa595-147">Permissions are assigned to users using [role-based access control (RBAC)](/azure/role-based-access-control/).</span></span>

<span data-ttu-id="aa595-148">![](../_images/governance-1-18.png)
*Figure 10. Un ou plusieurs rôles RBAC sont attribués à chaque utilisateur du client.*</span><span class="sxs-lookup"><span data-stu-id="aa595-148">![](../_images/governance-1-18.png)
*Figure 10. Each user in the tenant is assigned one or more RBAC roles.*</span></span>

<span data-ttu-id="aa595-149">Un rôle RBAC spécifie un ensemble d’autorisations qu’un utilisateur peut utiliser pour une ressource spécifique.</span><span class="sxs-lookup"><span data-stu-id="aa595-149">An RBAC role specifies a set of permissions a user may take on a specific resource.</span></span> <span data-ttu-id="aa595-150">Lorsque le rôle est attribué à l’utilisateur, ces autorisations s’appliquent.</span><span class="sxs-lookup"><span data-stu-id="aa595-150">When the role is assigned to the user, those permissions are applied.</span></span> <span data-ttu-id="aa595-151">Par exemple, le [rôle de **propriétaire** intégré](/azure/role-based-access-control/built-in-roles#owner) autorise un utilisateur à effectuer une action sur une ressource.</span><span class="sxs-lookup"><span data-stu-id="aa595-151">For example, the [built-in **owner** role](/azure/role-based-access-control/built-in-roles#owner) allows a user to perform any action on a resource.</span></span>

<span data-ttu-id="aa595-152">Le contrôle suivant vérifie que la requête est autorisée sous les paramètres spécifiés dans la [stratégie de ressources Azure](/azure/governance/policy/).</span><span class="sxs-lookup"><span data-stu-id="aa595-152">The next control is a check that the request is allowed under the settings specified for [Azure resource policy](/azure/governance/policy/).</span></span> <span data-ttu-id="aa595-153">Les stratégies de ressources Azure spécifient les opérations autorisées pour une ressource spécifique.</span><span class="sxs-lookup"><span data-stu-id="aa595-153">Azure resource policies specify the operations allowed for a specific resource.</span></span> <span data-ttu-id="aa595-154">Par exemple, une stratégie de ressources Azure peut spécifier que les utilisateurs sont uniquement autorisés à déployer un type spécifique de machine virtuelle.</span><span class="sxs-lookup"><span data-stu-id="aa595-154">For example, an Azure resource policy can specify that users are only allowed to deploy a specific type of virtual machine.</span></span>

<span data-ttu-id="aa595-155">![](../_images/governance-1-19.png)
*Figure 11. Stratégie de ressource Azure.*</span><span class="sxs-lookup"><span data-stu-id="aa595-155">![](../_images/governance-1-19.png)
*Figure 11. Azure resource policy.*</span></span>

<span data-ttu-id="aa595-156">Le contrôle suivant consiste à vérifier que la requête ne dépasse pas une [limite d’abonnement Azure](/azure/azure-subscription-service-limits).</span><span class="sxs-lookup"><span data-stu-id="aa595-156">The next control is a check that the request does not exceed an [Azure subscription limit](/azure/azure-subscription-service-limits).</span></span> <span data-ttu-id="aa595-157">Par exemple, chaque abonnement dispose d’une limite de 980 groupes de ressources par abonnement.</span><span class="sxs-lookup"><span data-stu-id="aa595-157">For example, each subscription has a limit of 980 resource groups per subscription.</span></span> <span data-ttu-id="aa595-158">Si une requête pour déployer un groupe de ressources supplémentaire est reçue une fois la limite atteinte, elle est refusée.</span><span class="sxs-lookup"><span data-stu-id="aa595-158">If a request is received to deploy an additional resource group once the limit has been reached, it is denied.</span></span>

<span data-ttu-id="aa595-159">![](../_images/governance-1-20.png)
*Figure 12. Limites des ressources Azure.*</span><span class="sxs-lookup"><span data-stu-id="aa595-159">![](../_images/governance-1-20.png)
*Figure 12. Azure resource limits.*</span></span>

<span data-ttu-id="aa595-160">Le contrôle final vérifie que la requête est comprise dans l’engagement financier associé à l’abonnement.</span><span class="sxs-lookup"><span data-stu-id="aa595-160">The final control is a check that the request is within the financial commitment associated with the subscription.</span></span> <span data-ttu-id="aa595-161">Par exemple, si la requête requiert le déploiement d’une machine virtuelle, Resource Manager vérifie que l’abonnement dispose d’informations de paiement suffisantes.</span><span class="sxs-lookup"><span data-stu-id="aa595-161">For example, if the request is to deploy a virtual machine, Resource Manager verifies that the subscription has sufficient payment information.</span></span>

<span data-ttu-id="aa595-162">![](../_images/governance-1-21.png)
*Figure 13. Un engagement financier est associé à un abonnement.*</span><span class="sxs-lookup"><span data-stu-id="aa595-162">![](../_images/governance-1-21.png)
*Figure 13. A financial commitment is associated with a subscription.*</span></span>

## <a name="summary"></a><span data-ttu-id="aa595-163">Résumé</span><span class="sxs-lookup"><span data-stu-id="aa595-163">Summary</span></span>

<span data-ttu-id="aa595-164">Dans cet article, vous avez étudié la gestion des accès aux ressources dans Azure à l’aide d’Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="aa595-164">In this article, you learned about how resource access is managed in Azure using Azure Resource Manager.</span></span>

## <a name="next-steps"></a><span data-ttu-id="aa595-165">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="aa595-165">Next steps</span></span>

<span data-ttu-id="aa595-166">Maintenant que vous comprenez la gestion des accès aux ressources dans Azure, poursuivez votre apprentissage avec la conception d’un modèle de gouvernance.</span><span class="sxs-lookup"><span data-stu-id="aa595-166">Now that you understand how resource access is managed in Azure, move on to learn how to design a governance model.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="aa595-167">Gouvernance cloud</span><span class="sxs-lookup"><span data-stu-id="aa595-167">Cloud governance</span></span>](../governance/overview.md)
