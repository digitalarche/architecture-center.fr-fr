---
title: À propos de l’application Tailspin Surveys
description: Présentation de l’application Tailspin Surveys
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: index
pnp.series.next: authenticate
ms.openlocfilehash: 028f7940d2e3cd7e8e629554f8af290ec5fdd184
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2017
ms.locfileid: "24540055"
---
# <a name="the-tailspin-scenario"></a><span data-ttu-id="8c6da-103">Le scénario Tailspin</span><span class="sxs-lookup"><span data-stu-id="8c6da-103">The Tailspin scenario</span></span>

<span data-ttu-id="8c6da-104">[![GitHub](../_images/github.png) Exemple de code][sample application]</span><span class="sxs-lookup"><span data-stu-id="8c6da-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="8c6da-105">Tailspin est une société fictive qui développe une application SaaS nommée Surveys.</span><span class="sxs-lookup"><span data-stu-id="8c6da-105">Tailspin is a fictitious company that is developing a SaaS application named Surveys.</span></span> <span data-ttu-id="8c6da-106">Cette application permet aux organisations de créer et de publier des enquêtes en ligne.</span><span class="sxs-lookup"><span data-stu-id="8c6da-106">This application enables organizations to create and publish online surveys.</span></span>

* <span data-ttu-id="8c6da-107">Une organisation peut s’inscrire auprès de l’application.</span><span class="sxs-lookup"><span data-stu-id="8c6da-107">An organization can sign up for the application.</span></span>
* <span data-ttu-id="8c6da-108">Une fois que l’organisation est inscrite, les utilisateurs peuvent se connecter à l’application avec leurs informations d’identification professionnelles.</span><span class="sxs-lookup"><span data-stu-id="8c6da-108">After the organization is signed up, users can sign into the application with their organizational credentials.</span></span>
* <span data-ttu-id="8c6da-109">Les utilisateurs peuvent créer, modifier et publier des enquêtes.</span><span class="sxs-lookup"><span data-stu-id="8c6da-109">Users can create, edit, and publish surveys.</span></span>

> [!NOTE]
> <span data-ttu-id="8c6da-110">Pour vous familiariser avec l’application, consultez [Run the Surveys application] (Exécution de l’application Surveys).</span><span class="sxs-lookup"><span data-stu-id="8c6da-110">To get started with the application, see [Run the Surveys application].</span></span>
> 
> 

## <a name="users-can-create-edit-and-view-surveys"></a><span data-ttu-id="8c6da-111">Les utilisateurs peuvent créer, modifier et consulter des enquêtes.</span><span class="sxs-lookup"><span data-stu-id="8c6da-111">Users can create, edit, and view surveys</span></span>
<span data-ttu-id="8c6da-112">Un utilisateur authentifié peut consulter toutes les enquêtes qu’il a créées ou pour lesquelles il détient des droits de contributeur. Il peut également créer des enquêtes.</span><span class="sxs-lookup"><span data-stu-id="8c6da-112">An authenticated user can view all the surveys that he or she has created or has contributor rights to, and create new surveys.</span></span> <span data-ttu-id="8c6da-113">Notez que l’utilisateur est connecté à l’aide de son identité professionnelle, `bob@contoso.com`.</span><span class="sxs-lookup"><span data-stu-id="8c6da-113">Notice that the user is signed in with his organizational identity, `bob@contoso.com`.</span></span>

![Application Surveys](./images/surveys-screenshot.png)

<span data-ttu-id="8c6da-115">Cette capture d’écran illustre la page Edit Survey :</span><span class="sxs-lookup"><span data-stu-id="8c6da-115">This screenshot shows the Edit Survey page:</span></span>

![Modifier l’enquête](./images/edit-survey.png)

<span data-ttu-id="8c6da-117">Les utilisateurs peuvent aussi consulter les enquêtes créées par d’autres utilisateurs au sein du même locataire.</span><span class="sxs-lookup"><span data-stu-id="8c6da-117">Users can also view any surveys created by other users within the same tenant.</span></span>

![Enquêtes client](./images/tenant-surveys.png)

## <a name="survey-owners-can-invite-contributors"></a><span data-ttu-id="8c6da-119">Les propriétaires d’enquêtes peuvent inviter des contributeurs.</span><span class="sxs-lookup"><span data-stu-id="8c6da-119">Survey owners can invite contributors</span></span>
<span data-ttu-id="8c6da-120">Lorsqu’un utilisateur crée une enquête, il peut inviter d’autres personnes à devenir des collaborateurs sur l’enquête.</span><span class="sxs-lookup"><span data-stu-id="8c6da-120">When a user creates a survey, he or she can invite other people to be contributors on the survey.</span></span> <span data-ttu-id="8c6da-121">Les collaborateurs peuvent modifier l’enquête, mais ils ne peuvent pas la supprimer, ni la publier.</span><span class="sxs-lookup"><span data-stu-id="8c6da-121">Contributors can edit the survey, but cannot delete or publish it.</span></span>  

![Ajouter un collaborateur](./images/add-contributor.png)

<span data-ttu-id="8c6da-123">Un utilisateur peut ajouter des collaborateurs à partir d’autres clients, ce qui permet le partage des ressources entre locataires.</span><span class="sxs-lookup"><span data-stu-id="8c6da-123">A user can add contributors from other tenants, which enables cross-tenant sharing of resources.</span></span> <span data-ttu-id="8c6da-124">Dans cette capture d’écran, Bob (`bob@contoso.com`) ajoute Alice (`alice@fabrikam.com`) en tant que contributeur à une enquête que Bob a créée.</span><span class="sxs-lookup"><span data-stu-id="8c6da-124">In this screenshot, Bob (`bob@contoso.com`) is adding Alice (`alice@fabrikam.com`) as a contributor to a survey that Bob created.</span></span>

<span data-ttu-id="8c6da-125">Lorsqu’Alice se connecte, elle voit l’enquête répertoriée sous « Surveys I can contribute to » (Enquêtes auxquelles je peux contribuer).</span><span class="sxs-lookup"><span data-stu-id="8c6da-125">When Alice logs in, she sees the survey listed under "Surveys I can contribute to".</span></span>

![Collaborateur de l’enquête](./images/contributor.png)

<span data-ttu-id="8c6da-127">Notez qu’Alice se connecte à son propre locataire et non en tant qu’invité du locataire Contoso.</span><span class="sxs-lookup"><span data-stu-id="8c6da-127">Note that Alice signs into her own tenant, not as a guest of the Contoso tenant.</span></span> <span data-ttu-id="8c6da-128">Alice a des autorisations de contributeur uniquement pour cette enquête &mdash; elle ne peut pas consulter les autres enquêtes du locataire Contoso.</span><span class="sxs-lookup"><span data-stu-id="8c6da-128">Alice has contributor permissions only for that survey &mdash; she cannot view other surveys from the Contoso tenant.</span></span>

## <a name="architecture"></a><span data-ttu-id="8c6da-129">Architecture</span><span class="sxs-lookup"><span data-stu-id="8c6da-129">Architecture</span></span>
<span data-ttu-id="8c6da-130">L’application Surveys se compose d’un serveur web frontal et d’un serveur principal d’API web.</span><span class="sxs-lookup"><span data-stu-id="8c6da-130">The Surveys application consists of a web front end and a web API backend.</span></span> <span data-ttu-id="8c6da-131">Les deux sont implémentés à l’aide [d’ASP.NET Core].</span><span class="sxs-lookup"><span data-stu-id="8c6da-131">Both are implemented using [ASP.NET Core].</span></span>

<span data-ttu-id="8c6da-132">L’application web utilise Azure Active Directory (Azure AD) pour authentifier les utilisateurs.</span><span class="sxs-lookup"><span data-stu-id="8c6da-132">The web application uses Azure Active Directory (Azure AD) to authenticate users.</span></span> <span data-ttu-id="8c6da-133">L’application web appelle également Azure AD pour obtenir des jetons d’accès OAuth 2 pour l’API web.</span><span class="sxs-lookup"><span data-stu-id="8c6da-133">The web application also calls Azure AD to get OAuth 2 access tokens for the Web API.</span></span> <span data-ttu-id="8c6da-134">Les jetons d’accès sont mis en cache dans le Cache Redis Azure.</span><span class="sxs-lookup"><span data-stu-id="8c6da-134">Access tokens are cached in Azure Redis Cache.</span></span> <span data-ttu-id="8c6da-135">Le cache permet à plusieurs instances de partager le même cache de jeton (par exemple, dans une batterie de serveurs).</span><span class="sxs-lookup"><span data-stu-id="8c6da-135">The cache enables multiple instances to share the same token cache (e.g., in a server farm).</span></span>

![Architecture](./images/architecture.png)

<span data-ttu-id="8c6da-137">[**Suivant**][authentication]</span><span class="sxs-lookup"><span data-stu-id="8c6da-137">[**Next**][authentication]</span></span>

<!-- Links -->

[authentication]: authenticate.md

[Run the Surveys application]: ./run-the-app.md
[d’ASP.NET Core]: /aspnet/core
[ASP.NET Core]: /aspnet/core
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
