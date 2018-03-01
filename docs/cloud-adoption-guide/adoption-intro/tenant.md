---
title: "Guide : conception d’un abonné Azure AD"
description: "Aide à la conception d’un client Azure dans le cadre d’une stratégie préparatoire à l’adoption du cloud"
author: telmosampaio
ms.openlocfilehash: 9ac52e9fd44bd8b9c777625002d5960f4f269be2
ms.sourcegitcommit: 29fbcb1eec44802d2c01b6d3bcf7d7bd0bae65fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/27/2018
---
# <a name="guidance-azure-ad-tenant-design"></a><span data-ttu-id="430e2-103">Guide : conception d’un abonné Azure AD</span><span class="sxs-lookup"><span data-stu-id="430e2-103">Guidance: Azure AD tenant design</span></span>

<span data-ttu-id="430e2-104">Un client Azure AD fournit les espaces de noms et les services d’identité numérique utilisés par un ou plusieurs [abonnements Azure](subscription-explainer.md).</span><span class="sxs-lookup"><span data-stu-id="430e2-104">An Azure AD tenant provides digital identity services and namespaces used for one or more [Azure subscriptions](subscription-explainer.md).</span></span> <span data-ttu-id="430e2-105">Si vous suivez le plan préparatoire à l’adoption, vous savez déjà [comment obtenir un client Azure AD][how-to-get-aad-tenant].</span><span class="sxs-lookup"><span data-stu-id="430e2-105">If you are following the foundational adoption outline, you have already learned [how to get an Azure AD tenant][how-to-get-aad-tenant].</span></span> 

## <a name="design-considerations"></a><span data-ttu-id="430e2-106">Remarques relatives à la conception</span><span class="sxs-lookup"><span data-stu-id="430e2-106">Design considerations</span></span>

- <span data-ttu-id="430e2-107">Pendant la phase préparatoire à l’adoption, vous pouvez commencer avec un seul client Azure AD.</span><span class="sxs-lookup"><span data-stu-id="430e2-107">At the foundational adoption stage, you can begin with a single Azure AD tenant.</span></span> <span data-ttu-id="430e2-108">Si votre organisation dispose déjà d’un abonnement Office 365 ou d’un abonnement Azure, vous pouvez utiliser le client Azure AD fourni.</span><span class="sxs-lookup"><span data-stu-id="430e2-108">If your organization has an existing Office 365 subscription or an Azure subscription, you already have an Azure AD tenant that you can use.</span></span> <span data-ttu-id="430e2-109">Sinon, renseignez-vous pour savoir [comment obtenir un client Azure AD][how-to-get-aad-tenant].</span><span class="sxs-lookup"><span data-stu-id="430e2-109">If you do not have either or of these, you can learn more about [how to get an Azure AD tenant][how-to-get-aad-tenant].</span></span> 
- <span data-ttu-id="430e2-110">Aux stades d’adoption intermédiaire et avancé, vous apprendrez à synchroniser ou à fédérer des annuaires locaux avec Azure AD.</span><span class="sxs-lookup"><span data-stu-id="430e2-110">In the intermediate and advanced adoption stages, you will learn how to synchronize or federate on-premises directories with Azure AD.</span></span> <span data-ttu-id="430e2-111">Cela vous permettra d’utiliser l’identité numérique locale dans Azure AD.</span><span class="sxs-lookup"><span data-stu-id="430e2-111">This will allow you to use on-premises digital identity in Azure AD.</span></span> <span data-ttu-id="430e2-112">Toutefois, au cours de la phase préparatoire à l’adoption, vous allez ajouter de nouveaux utilisateurs ayant pour seule identité votre unique client Azure AD.</span><span class="sxs-lookup"><span data-stu-id="430e2-112">However, at the foundational stage, you will be adding new users that only have identity your single Azure AD tenant.</span></span> <span data-ttu-id="430e2-113">Votre rôle sera de gérer ces identités.</span><span class="sxs-lookup"><span data-stu-id="430e2-113">You will be responsible for managing those identities.</span></span> <span data-ttu-id="430e2-114">Vous devrez par exemple intégrer de nouveaux utilisateurs Azure AD, débarquer des utilisateurs Azure AD à qui vous ne souhaitez plus accorder l’accès aux ressources Azure, et apporter d’autres modifications aux autorisations des utilisateurs.</span><span class="sxs-lookup"><span data-stu-id="430e2-114">For example, you will have to on-board new Azure AD users, off-board Azure AD users that you no longer wish to have access to Azure resources, and other changes to user permissions.</span></span>

## <a name="next-steps"></a><span data-ttu-id="430e2-115">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="430e2-115">Next Steps</span></span>

* <span data-ttu-id="430e2-116">Maintenant que vous disposez d’un client Azure AD, découvrez [comment ajouter un utilisateur][azure-ad-add-user].</span><span class="sxs-lookup"><span data-stu-id="430e2-116">Now that you have an Azure AD tenant, learn [how to add a user][azure-ad-add-user].</span></span> <span data-ttu-id="430e2-117">Une fois que vous aurez ajouté un ou plusieurs autres utilisateurs à votre client Azure AD, renseignez-vous sur les [Abonnements Azure](subscription-explainer.md).</span><span class="sxs-lookup"><span data-stu-id="430e2-117">After you have added one or more new users to your Azure AD tenant, your next step is learning about [Azure subscriptions](subscription-explainer.md).</span></span>

<!-- Links -->

[azure-ad-add-user]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-manage-azure-ad]: /azure/active-directory/active-directory-administer?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-associate-subscription]: /azure/active-directory/active-directory-how-subscriptions-associated-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[how-to-get-aad-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
