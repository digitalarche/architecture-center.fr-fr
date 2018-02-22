---
title: "Explicatif : qu’est-ce qu’un abonnement Azure ?"
description: "Décrit les abonnements, les comptes et les offres Azure"
author: abuck
ms.openlocfilehash: 7b88e9489b40b100eecb76602b45901566b3f37f
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/09/2018
---
# <a name="explainer-what-is-an-azure-subscription"></a><span data-ttu-id="3d321-103">Explicatif : qu’est-ce qu’un abonnement Azure ?</span><span class="sxs-lookup"><span data-stu-id="3d321-103">Explainer: What is an Azure subscription?</span></span>

<span data-ttu-id="3d321-104">Dans l’article explicatif [Qu’est un locataire Azure Active Directory ?](tenant-explainer.md), vous avez appris que l’identité numérique de votre organisation est stockée dans un locataire Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="3d321-104">In the [what is an Azure Active Directory tenant?](tenant-explainer.md) explainer article, you learned that digital identity for your organization is stored in an Azure Active Directory tenant.</span></span> <span data-ttu-id="3d321-105">Vous avez également appris qu’Azure approuve Azure Active Directory pour authentifier les demandes de création, de lecture, de mise à jour et de suppression de ressources des utilisateurs.</span><span class="sxs-lookup"><span data-stu-id="3d321-105">You also learned that Azure trusts Azure Active Directory to authenticate user requests to create, read, update, or delete a resource.</span></span> 

<span data-ttu-id="3d321-106">Nous comprenons fondamentalement pourquoi il est nécessaire de restreindre l’accès à ces opérations sur une ressource, à savoir pour empêcher les utilisateurs non authentifiés et non autorisés d’accéder à nos ressources.</span><span class="sxs-lookup"><span data-stu-id="3d321-106">We fundamentally understand why it's necessary to restrict access to these operations on a resource - to prevent unauthenticated and unauthorized users from accessing our resources.</span></span> <span data-ttu-id="3d321-107">Mais ces opérations sur les ressources ont d’autres propriétés qu’une organisation souhaiterait contrôler, telles que le nombre de ressources qu’un utilisateur ou qu’un groupe d’utilisateurs est autorisé à créer, ainsi que le coût d’exécution de ces ressources.</span><span class="sxs-lookup"><span data-stu-id="3d321-107">However, these resource operations have other properties that an organization would like to control, such as the number of resources a user or group of users is allowed to create, and, the cost to run those resources.</span></span> 

<span data-ttu-id="3d321-108">Azure implémente ce contrôle, qui s’appelle un **abonnement**.</span><span class="sxs-lookup"><span data-stu-id="3d321-108">Azure implements this control, and it is named a **subscription**.</span></span> <span data-ttu-id="3d321-109">Un abonnement regroupe les utilisateurs et les ressources qui ont été créées par ces utilisateurs.</span><span class="sxs-lookup"><span data-stu-id="3d321-109">A subscription groups together users and the resources that have been created by those users.</span></span> <span data-ttu-id="3d321-110">Chacune de ces ressources contribue à une [limite globale][subscription-service-limits] sur cette ressource particulière.</span><span class="sxs-lookup"><span data-stu-id="3d321-110">Each of those resources contributes to an [overall limit][subscription-service-limits] on that particular resource.</span></span>

<span data-ttu-id="3d321-111">Les organisations peuvent utiliser des abonnements pour gérer les coûts et la création de ressources par les utilisateurs, les équipes et les projets, ou utiliser de nombreuses autres stratégies.</span><span class="sxs-lookup"><span data-stu-id="3d321-111">Organizations can use subscriptions to manage costs and creation of resource by users, teams, projects, or using many other strategies.</span></span> <span data-ttu-id="3d321-112">Ces stratégies seront abordées dans les articles relatifs aux phases d’adoption intermédiaire et avancée.</span><span class="sxs-lookup"><span data-stu-id="3d321-112">These strategies will be discussed in the intermediate and advanced adoption stage articles.</span></span> 

## <a name="next-steps"></a><span data-ttu-id="3d321-113">étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="3d321-113">Next steps</span></span>

* <span data-ttu-id="3d321-114">Maintenant que vous avez en savez davantage sur les abonnements Azure, découvrez-en plus sur la [création d’un abonnement](subscription.md) avant de créer vos premières ressources Azure.</span><span class="sxs-lookup"><span data-stu-id="3d321-114">Now that you have learned about Azure subscriptions, learn more about [creating a subscription](subscription.md) before you create your first Azure resources..</span></span>

<!-- Links -->
[azure-get-started]: https://azure.microsoft.com/en-us/get-started/
[azure-offers]: https://azure.microsoft.com/en-us/support/legal/offer-details/
[azure-free-trial]: https://azure.microsoft.com/en-us/offers/ms-azr-0044p/
[azure-change-subscription-offer]: /azure/billing/billing-how-to-switch-azure-offer
[microsoft-account]: https://account.microsoft.com/account
[subscription-service-limits]: /azure/azure-subscription-service-limits
[docs-organizational-account]: https://docs.microsoft.com/en-us/azure/active-directory/sign-up-organization
