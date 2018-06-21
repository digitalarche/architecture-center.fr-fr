---
title: 'Explicatif : qu’est-ce qu’un abonnement Azure ?'
description: Décrit les abonnements, les comptes et les offres Azure
author: alexbuckgit
ms.openlocfilehash: 1650d90d6f78b46b7fe4128d2dab6a80bd6cca78
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/23/2018
ms.locfileid: "29478304"
---
# <a name="explainer-what-is-an-azure-subscription"></a>Explicatif : qu’est-ce qu’un abonnement Azure ?

Dans l’article explicatif [Qu’est un locataire Azure Active Directory ?](tenant-explainer.md), vous avez appris que l’identité numérique de votre organisation est stockée dans un locataire Azure Active Directory. Vous avez également appris qu’Azure approuve Azure Active Directory pour authentifier les demandes de création, de lecture, de mise à jour et de suppression de ressources des utilisateurs. 

Nous comprenons fondamentalement pourquoi il est nécessaire de restreindre l’accès à ces opérations sur une ressource, à savoir pour empêcher les utilisateurs non authentifiés et non autorisés d’accéder à nos ressources. Mais ces opérations sur les ressources ont d’autres propriétés qu’une organisation souhaiterait contrôler, telles que le nombre de ressources qu’un utilisateur ou qu’un groupe d’utilisateurs est autorisé à créer, ainsi que le coût d’exécution de ces ressources. 

Azure implémente ce contrôle, qui s’appelle un **abonnement**. Un abonnement regroupe les utilisateurs et les ressources qui ont été créées par ces utilisateurs. Chacune de ces ressources contribue à une [limite globale][subscription-service-limits] sur cette ressource particulière.

Les organisations peuvent utiliser des abonnements pour gérer les coûts et la création de ressources par les utilisateurs, les équipes et les projets, ou utiliser de nombreuses autres stratégies. Ces stratégies seront abordées dans les articles relatifs aux phases d’adoption intermédiaire et avancée. 

## <a name="next-steps"></a>étapes suivantes

* Maintenant que vous avez en savez davantage sur les abonnements Azure, découvrez-en plus sur la [création d’un abonnement](subscription.md) avant de créer vos premières ressources Azure.

<!-- Links -->
[azure-get-started]: https://azure.microsoft.com/get-started/
[azure-offers]: https://azure.microsoft.com/support/legal/offer-details/
[azure-free-trial]: https://azure.microsoft.com/offers/ms-azr-0044p/
[azure-change-subscription-offer]: /azure/billing/billing-how-to-switch-azure-offer
[microsoft-account]: https://account.microsoft.com/account
[subscription-service-limits]: /azure/azure-subscription-service-limits
[docs-organizational-account]: https://docs.microsoft.com/azure/active-directory/sign-up-organization
