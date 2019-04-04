---
title: Gestion des identités pour les applications multilocataires
description: Meilleures pratiques pour la gestion de l’authentification, de l’autorisation et de l’identité dans les applications multi-locataires.
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.openlocfilehash: be906106fb12c381d57ad40ae22e748dcff9722f
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58346074"
---
# <a name="manage-identity-in-multitenant-applications"></a><span data-ttu-id="705d9-103">Gérer l’identité dans les applications mutualisées</span><span class="sxs-lookup"><span data-stu-id="705d9-103">Manage identity in multitenant applications</span></span>

<span data-ttu-id="705d9-104">Cette série d’articles décrit les meilleures pratiques pour les applications multi-locataires, lors de l’utilisation d’Azure AD pour l’authentification et la gestion des identités.</span><span class="sxs-lookup"><span data-stu-id="705d9-104">This series of articles describes best practices for multitenancy, when using Azure AD for authentication and identity management.</span></span>

<span data-ttu-id="705d9-105">[![GitHub](../_images/github.png) Exemple de code][sample-application]</span><span class="sxs-lookup"><span data-stu-id="705d9-105">[![GitHub](../_images/github.png) Sample code][sample-application]</span></span>

<span data-ttu-id="705d9-106">Lorsque vous générez une les application multi-locataire, l’une des premières difficultés est la gestion des identités utilisateur, car désormais chaque utilisateur appartient à un client.</span><span class="sxs-lookup"><span data-stu-id="705d9-106">When you're building a multitenant application, one of the first challenges is managing user identities, because now every user belongs to a tenant.</span></span> <span data-ttu-id="705d9-107">Par exemple : </span><span class="sxs-lookup"><span data-stu-id="705d9-107">For example:</span></span>

- <span data-ttu-id="705d9-108">Les utilisateurs se connectent avec les informations d’identification de leur organisation.</span><span class="sxs-lookup"><span data-stu-id="705d9-108">Users sign in with their organizational credentials.</span></span>
- <span data-ttu-id="705d9-109">Les utilisateurs doivent avoir accès aux données de leur organisation, mais pas aux données appartenant à d’autres clients.</span><span class="sxs-lookup"><span data-stu-id="705d9-109">Users should have access to their organization's data, but not data that belongs to other tenants.</span></span>
- <span data-ttu-id="705d9-110">Une organisation peut s’inscrire auprès de l’application, puis affecter des rôles d’application à ses membres.</span><span class="sxs-lookup"><span data-stu-id="705d9-110">An organization can sign up for the application, and then assign application roles to its members.</span></span>

<span data-ttu-id="705d9-111">Azure Active Directory (Azure AD) possède des fonctionnalités qui prennent en charge tous ces scénarios.</span><span class="sxs-lookup"><span data-stu-id="705d9-111">Azure Active Directory (Azure AD) has some great features that support all of these scenarios.</span></span>

<span data-ttu-id="705d9-112">Pour accompagner cette série d’articles, nous avons créé une [implémentation de bout en bout] [ sample-application] d’une application multi-locataire.</span><span class="sxs-lookup"><span data-stu-id="705d9-112">To accompany this series of articles, we created a complete [end-to-end implementation][sample-application] of a multitenant application.</span></span> <span data-ttu-id="705d9-113">Les articles reflètent ce que nous avons appris au cours de la création de l’application.</span><span class="sxs-lookup"><span data-stu-id="705d9-113">The articles reflect what we learned in the process of building the application.</span></span> <span data-ttu-id="705d9-114">Pour vous familiariser avec l’application, consultez [Exécution de l’application Surveys][running-the-app].</span><span class="sxs-lookup"><span data-stu-id="705d9-114">To get started with the application, see [Run the Surveys application][running-the-app].</span></span>

## <a name="introduction"></a><span data-ttu-id="705d9-115">Introduction</span><span class="sxs-lookup"><span data-stu-id="705d9-115">Introduction</span></span>

<span data-ttu-id="705d9-116">Supposons que vous écriviez une application SaaS d’entreprise pour qu’elle soit hébergée dans le cloud.</span><span class="sxs-lookup"><span data-stu-id="705d9-116">Let's say you're writing an enterprise SaaS application to be hosted in the cloud.</span></span> <span data-ttu-id="705d9-117">Bien sûr, cette application aura des utilisateurs :</span><span class="sxs-lookup"><span data-stu-id="705d9-117">Of course, the application will have users:</span></span>

![Utilisateurs](./images/users.png)

<span data-ttu-id="705d9-119">Mais ces utilisateurs appartiennent à des organisations :</span><span class="sxs-lookup"><span data-stu-id="705d9-119">But those users belong to organizations:</span></span>

![Utilisateurs d’organisation](./images/org-users.png)

<span data-ttu-id="705d9-121">Exemple : Tailspin vend des abonnements à son application SaaS.</span><span class="sxs-lookup"><span data-stu-id="705d9-121">Example: Tailspin sells subscriptions to its SaaS application.</span></span> <span data-ttu-id="705d9-122">Contoso et Fabrikam s’inscrivent à l’application.</span><span class="sxs-lookup"><span data-stu-id="705d9-122">Contoso and Fabrikam sign up for the app.</span></span> <span data-ttu-id="705d9-123">Quand Alice (`alice@contoso`) s’inscrit, l’application doit savoir qu’Alice fait partie de Contoso.</span><span class="sxs-lookup"><span data-stu-id="705d9-123">When Alice (`alice@contoso`) signs in, the application should know that Alice is part of Contoso.</span></span>

- <span data-ttu-id="705d9-124">Alice *doit* avoir accès aux données Contoso.</span><span class="sxs-lookup"><span data-stu-id="705d9-124">Alice *should* have access to Contoso data.</span></span>
- <span data-ttu-id="705d9-125">Alice *ne doit pas* avoir accès aux données Fabrikam.</span><span class="sxs-lookup"><span data-stu-id="705d9-125">Alice *should not* have access to Fabrikam data.</span></span>

<span data-ttu-id="705d9-126">Ce guide vous explique comment gérer les identités des utilisateurs dans une application multilocataire, en utilisant [Azure Active Directory (Azure AD)](/azure/active-directory) pour gérer la connexion et l’authentification.</span><span class="sxs-lookup"><span data-stu-id="705d9-126">This guidance will show you how to manage user identities in a multitenant application, using [Azure Active Directory (Azure AD)](/azure/active-directory) to handle sign-in and authentication.</span></span>

<!-- markdownlint-disable MD026 -->

## <a name="what-is-multitenancy"></a><span data-ttu-id="705d9-127">Qu’est-ce que l’architecture mutualisée ?</span><span class="sxs-lookup"><span data-stu-id="705d9-127">What is multitenancy?</span></span>

<!-- markdownlint-enable MD026 -->

<span data-ttu-id="705d9-128">Un *locataire* est un groupe d’utilisateurs.</span><span class="sxs-lookup"><span data-stu-id="705d9-128">A *tenant* is a group of users.</span></span> <span data-ttu-id="705d9-129">Dans une application SaaS, le locataire est un abonné ou un client de l’application.</span><span class="sxs-lookup"><span data-stu-id="705d9-129">In a SaaS application, the tenant is a subscriber or customer of the application.</span></span> <span data-ttu-id="705d9-130">*architecture mutualisée* est une architecture où plusieurs locataires partagent la même instance physique de l’application.</span><span class="sxs-lookup"><span data-stu-id="705d9-130">*Multitenancy* is an architecture where multiple tenants share the same physical instance of the app.</span></span> <span data-ttu-id="705d9-131">Bien que les locataires partagent des ressources physiques (par exemple, les machines virtuelles ou le stockage), chaque locataire obtient sa propre instance logique de l’application.</span><span class="sxs-lookup"><span data-stu-id="705d9-131">Although tenants share physical resources (such as VMs or storage), each tenant gets its own logical instance of the app.</span></span>

<span data-ttu-id="705d9-132">Normalement, les données d’application sont partagées entre les utilisateurs d’un locataire, mais pas avec d’autres locataires.</span><span class="sxs-lookup"><span data-stu-id="705d9-132">Typically, application data is shared among the users within a tenant, but not with other tenants.</span></span>

![Multi-locataire](./images/multitenant.png)

<span data-ttu-id="705d9-134">Comparez cette architecture avec une architecture à locataire unique, où chaque locataire a une instance physique dédiée.</span><span class="sxs-lookup"><span data-stu-id="705d9-134">Compare this architecture with a single-tenant architecture, where each tenant has a dedicated physical instance.</span></span> <span data-ttu-id="705d9-135">Dans une architecture à locataire unique, vous ajoutez des locataires mettant en place de nouvelles instances de l’application.</span><span class="sxs-lookup"><span data-stu-id="705d9-135">In a single-tenant architecture, you add tenants by spinning up new instances of the app.</span></span>

![Locataire unique](./images/single-tenant.png)

### <a name="multitenancy-and-horizontal-scaling"></a><span data-ttu-id="705d9-137">Architecture mutualisée et mise à l’échelle horizontale</span><span class="sxs-lookup"><span data-stu-id="705d9-137">Multitenancy and horizontal scaling</span></span>

<span data-ttu-id="705d9-138">Pour atteindre une mise à l’échelle dans le cloud, il est courant d’ajouter des instances physiques supplémentaires.</span><span class="sxs-lookup"><span data-stu-id="705d9-138">To achieve scale in the cloud, it’s common to add more physical instances.</span></span> <span data-ttu-id="705d9-139">Il s’agit de la *mise à l’échelle horizontale* ou de l’*augmentation de la taille des instances*. Prenons l’exemple d’une application web.</span><span class="sxs-lookup"><span data-stu-id="705d9-139">This is known as *horizontal scaling* or *scaling out*. Consider a web app.</span></span> <span data-ttu-id="705d9-140">Pour gérer davantage de trafic, vous pouvez ajouter des machines virtuelles serveurs supplémentaires et les placer derrière un équilibreur de charge.</span><span class="sxs-lookup"><span data-stu-id="705d9-140">To handle more traffic, you can add more server VMs and put them behind a load balancer.</span></span> <span data-ttu-id="705d9-141">Chaque machine virtuelle exécute une instance physique distincte de l’application web.</span><span class="sxs-lookup"><span data-stu-id="705d9-141">Each VM runs a separate physical instance of the web app.</span></span>

![Équilibrage de la charge d’un site web](./images/load-balancing.png)

<span data-ttu-id="705d9-143">Toute requête peut être acheminée vers n’importe quelle instance.</span><span class="sxs-lookup"><span data-stu-id="705d9-143">Any request can be routed to any instance.</span></span> <span data-ttu-id="705d9-144">Globalement, le système fonctionne comme une instance logique unique.</span><span class="sxs-lookup"><span data-stu-id="705d9-144">Together, the system functions as a single logical instance.</span></span> <span data-ttu-id="705d9-145">Vous pouvez supprimer une machine virtuelle ou en ajouter une nouvelle, sans affecter les utilisateurs.</span><span class="sxs-lookup"><span data-stu-id="705d9-145">You can tear down a VM or spin up a new VM, without affecting users.</span></span> <span data-ttu-id="705d9-146">Dans cette architecture, chaque instance physique est mutualisée, et vous effectuez la mise à l’échelle en ajoutant des instances supplémentaires.</span><span class="sxs-lookup"><span data-stu-id="705d9-146">In this architecture, each physical instance is multi-tenant, and you scale by adding more instances.</span></span> <span data-ttu-id="705d9-147">Si une instance subit une défaillance, cela ne doit affecter aucun locataire.</span><span class="sxs-lookup"><span data-stu-id="705d9-147">If one instance goes down, it should not affect any tenant.</span></span>

## <a name="identity-in-a-multitenant-app"></a><span data-ttu-id="705d9-148">Identité dans une application multi-locataire</span><span class="sxs-lookup"><span data-stu-id="705d9-148">Identity in a multitenant app</span></span>

<span data-ttu-id="705d9-149">Dans une application multi-locataire, vous devez considérer les utilisateurs dans le contexte des locataires.</span><span class="sxs-lookup"><span data-stu-id="705d9-149">In a multitenant app, you must consider users in the context of tenants.</span></span>

### <a name="authentication"></a><span data-ttu-id="705d9-150">Authentication</span><span class="sxs-lookup"><span data-stu-id="705d9-150">Authentication</span></span>

- <span data-ttu-id="705d9-151">Les utilisateurs se connectent à l’application avec les informations d’identification de leur organisation.</span><span class="sxs-lookup"><span data-stu-id="705d9-151">Users sign into the app with their organization credentials.</span></span> <span data-ttu-id="705d9-152">Ils n’ont pas à créer de nouveaux profils utilisateur pour l’application.</span><span class="sxs-lookup"><span data-stu-id="705d9-152">They don't have to create new user profiles for the app.</span></span>
- <span data-ttu-id="705d9-153">Les utilisateurs appartenant à la même organisation font partie du même locataire.</span><span class="sxs-lookup"><span data-stu-id="705d9-153">Users within the same organization are part of the same tenant.</span></span>
- <span data-ttu-id="705d9-154">Quand un utilisateur se connecte, l’application sait à quel locataire l’utilisateur appartient.</span><span class="sxs-lookup"><span data-stu-id="705d9-154">When a user signs in, the application knows which tenant the user belongs to.</span></span>

### <a name="authorization"></a><span data-ttu-id="705d9-155">Authorization</span><span class="sxs-lookup"><span data-stu-id="705d9-155">Authorization</span></span>

- <span data-ttu-id="705d9-156">Lors de l’autorisation d’actions d’un utilisateur (par exemple, l’affichage d’une ressource), l’application doit prendre en compte le locataire de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="705d9-156">When authorizing a user's actions (say, viewing a resource), the app must take into account the user's tenant.</span></span>
- <span data-ttu-id="705d9-157">Des rôles peuvent être affectés aux utilisateurs de dans l’application, comme « Admin » ou « Utilisateur Standard ».</span><span class="sxs-lookup"><span data-stu-id="705d9-157">Users might be assigned roles within the application, such as "Admin" or "Standard User".</span></span> <span data-ttu-id="705d9-158">Les attributions de rôles doivent être gérées par le client, et non par le fournisseur SaaS.</span><span class="sxs-lookup"><span data-stu-id="705d9-158">Role assignments should be managed by the customer, not by the SaaS provider.</span></span>

<span data-ttu-id="705d9-159">**Exemple.**</span><span class="sxs-lookup"><span data-stu-id="705d9-159">**Example.**</span></span> <span data-ttu-id="705d9-160">Alice, une employé de Contoso, navigue vers l’application dans son navigateur et clique sur le bouton « Se connecter ».</span><span class="sxs-lookup"><span data-stu-id="705d9-160">Alice, an employee at Contoso, navigates to the application in her browser and clicks the “Log in” button.</span></span> <span data-ttu-id="705d9-161">Elle est redirigée vers un écran de connexion où elle entre ses informations d’identification d’entreprise (nom d’utilisateur et mot de passe).</span><span class="sxs-lookup"><span data-stu-id="705d9-161">She is redirected to a login screen where she enters her corporate credentials (username and password).</span></span> <span data-ttu-id="705d9-162">À ce stade, elle est connectée à l’application en tant que `alice@contoso.com`.</span><span class="sxs-lookup"><span data-stu-id="705d9-162">At this point, she is logged into the app as `alice@contoso.com`.</span></span> <span data-ttu-id="705d9-163">L’application sait également qu’Alice est une utilisatrice administratrice de cette application.</span><span class="sxs-lookup"><span data-stu-id="705d9-163">The application also knows that Alice is an admin user for this application.</span></span> <span data-ttu-id="705d9-164">Son statut d’administratrice lui permet de voir une liste de toutes les ressources appartenant à Contoso.</span><span class="sxs-lookup"><span data-stu-id="705d9-164">Because she is an admin, she can see a list of all the resources that belong to Contoso.</span></span> <span data-ttu-id="705d9-165">Toutefois, elle ne peut pas afficher les ressources de Fabrikam, car elle est administratrice uniquement au sein de son client.</span><span class="sxs-lookup"><span data-stu-id="705d9-165">However, she cannot view Fabrikam's resources, because she is an admin only within her tenant.</span></span>

<span data-ttu-id="705d9-166">Dans ce guide, nous examinerons plus particulièrement l’utilisation d’Azure AD pour la gestion des identités.</span><span class="sxs-lookup"><span data-stu-id="705d9-166">In this guidance, we'll look specifically at using Azure AD for identity management.</span></span>

- <span data-ttu-id="705d9-167">Nous supposons que le client stocke ses profils utilisateur dans Azure AD (notamment les locataires Office 365 et Dynamics CRM).</span><span class="sxs-lookup"><span data-stu-id="705d9-167">We assume the customer stores their user profiles in Azure AD (including Office365 and Dynamics CRM tenants)</span></span>
- <span data-ttu-id="705d9-168">Les clients disposant d’Active Directory local peuvent utiliser [Azure AD Connect](/azure/active-directory/hybrid/whatis-hybrid-identity) pour synchroniser leur Active Directory local avec Azure AD.</span><span class="sxs-lookup"><span data-stu-id="705d9-168">Customers with on-premises Active Directory can use [Azure AD Connect](/azure/active-directory/hybrid/whatis-hybrid-identity) to sync their on-premises Active Directory with Azure AD.</span></span> <span data-ttu-id="705d9-169">Si un client disposant d’Active Directory local ne peut pas utiliser Azure AD Connect (en raison d’une stratégie informatique d’entreprise ou pour d’autres raisons), le fournisseur SaaS peut fédérer avec l’annuaire du client par le biais des services AD FS (Active Directory Federation Services).</span><span class="sxs-lookup"><span data-stu-id="705d9-169">If a customer with on-premises Active Directory cannot use Azure AD Connect (due to corporate IT policy or other reasons), the SaaS provider can federate with the customer's directory through Active Directory Federation Services (AD FS).</span></span> <span data-ttu-id="705d9-170">Cette option est décrite dans [Fédération avec les services ADFS d’un client](adfs.md).</span><span class="sxs-lookup"><span data-stu-id="705d9-170">This option is described in [Federating with a customer's AD FS](adfs.md).</span></span>

<span data-ttu-id="705d9-171">Ce guide ne prend pas en considération les autres aspects d’une architecture mutualisée, comme le partitionnement des données, la configuration par locataire, etc.</span><span class="sxs-lookup"><span data-stu-id="705d9-171">This guidance does not consider other aspects of multitenancy such as data partitioning, per-tenant configuration, and so forth.</span></span>

[<span data-ttu-id="705d9-172">**Suivant**</span><span class="sxs-lookup"><span data-stu-id="705d9-172">**Next**</span></span>](./tailspin.md)

<!-- links -->

[sample-application]: https://github.com/mspnp/multitenant-saas-guidance
[running-the-app]: ./run-the-app.md
