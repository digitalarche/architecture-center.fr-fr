---
title: 'Guide : conception d’un abonné Azure AD'
description: Aide à la conception d’un client Azure dans le cadre d’une stratégie préparatoire à l’adoption du cloud
author: telmosampaio
ms.openlocfilehash: 9ac52e9fd44bd8b9c777625002d5960f4f269be2
ms.sourcegitcommit: 29fbcb1eec44802d2c01b6d3bcf7d7bd0bae65fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/27/2018
---
# <a name="guidance-azure-ad-tenant-design"></a>Guide : conception d’un abonné Azure AD

Un client Azure AD fournit les espaces de noms et les services d’identité numérique utilisés par un ou plusieurs [abonnements Azure](subscription-explainer.md). Si vous suivez le plan préparatoire à l’adoption, vous savez déjà [comment obtenir un client Azure AD][how-to-get-aad-tenant]. 

## <a name="design-considerations"></a>Remarques relatives à la conception

- Pendant la phase préparatoire à l’adoption, vous pouvez commencer avec un seul client Azure AD. Si votre organisation dispose déjà d’un abonnement Office 365 ou d’un abonnement Azure, vous pouvez utiliser le client Azure AD fourni. Sinon, renseignez-vous pour savoir [comment obtenir un client Azure AD][how-to-get-aad-tenant]. 
- Aux stades d’adoption intermédiaire et avancé, vous apprendrez à synchroniser ou à fédérer des annuaires locaux avec Azure AD. Cela vous permettra d’utiliser l’identité numérique locale dans Azure AD. Toutefois, au cours de la phase préparatoire à l’adoption, vous allez ajouter de nouveaux utilisateurs ayant pour seule identité votre unique client Azure AD. Votre rôle sera de gérer ces identités. Vous devrez par exemple intégrer de nouveaux utilisateurs Azure AD, débarquer des utilisateurs Azure AD à qui vous ne souhaitez plus accorder l’accès aux ressources Azure, et apporter d’autres modifications aux autorisations des utilisateurs.

## <a name="next-steps"></a>Étapes suivantes

* Maintenant que vous disposez d’un client Azure AD, découvrez [comment ajouter un utilisateur][azure-ad-add-user]. Une fois que vous aurez ajouté un ou plusieurs autres utilisateurs à votre client Azure AD, renseignez-vous sur les [Abonnements Azure](subscription-explainer.md).

<!-- Links -->

[azure-ad-add-user]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-manage-azure-ad]: /azure/active-directory/active-directory-administer?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-associate-subscription]: /azure/active-directory/active-directory-how-subscriptions-associated-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[how-to-get-aad-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
