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
# <a name="resource-access-management-in-azure"></a>Gestion de l’accès aux ressources dans Azure

La rubrique [Gouvernance cloud](../overview.md) présente les cinq disciplines de la gouvernance cloud, ce qui inclut la gestion des ressources.  La rubrique [Présentation de la gouvernance des accès aux ressources](overview.md) explique comment la gestion de l’accès aux ressources s’inscrit dans la discipline de gestion des ressources. Avant de découvrir comment concevoir un modèle de gouvernance, il est important de comprendre les contrôles de gestion des accès aux ressources dans Azure. La configuration de ces contrôles de gestion des accès aux ressources constitue la base de votre modèle de gouvernance.

Commencez par examiner de plus près le déploiement des ressources dans Azure.

<!-- markdownlint-disable MD026 -->

## <a name="what-is-an-azure-resource"></a>Qu’est-ce qu’une ressource Azure ?

Dans Azure, le terme **ressource** fait référence à une entité gérée par Azure. Par exemple, les machines virtuelles, les réseaux virtuels et les comptes de stockage sont tous considérés comme des ressources Azure.

![Diagramme d’une ressource](../../_images/governance-1-9.png)
*Figure 1. Une ressource.*

## <a name="what-is-an-azure-resource-group"></a>Qu’est-ce qu’un groupe de ressources Azure ?

Chaque ressource dans Azure doit appartenir à un [groupe de ressources](/azure/azure-resource-manager/resource-group-overview#resource-groups). Un groupe de ressources est simplement une construction logique qui regroupe plusieurs ressources afin qu’elles puissent être gérées en tant qu’entité unique. Par exemple, des ressources qui partagent un cycle de vie similaire, tels que les ressources d’une [application multi-niveau](/azure/architecture/guide/architecture-styles/n-tier), peuvent être créées ou supprimées en tant que groupe.

![Diagramme d’un groupe de ressources contenant une ressource](../../_images/governance-1-10.png)
*Figure 2. Un groupe de ressources contient une ressource.*

Les groupes de ressources et les ressources qu’ils contiennent sont associés à un **abonnement** Azure.

## <a name="what-is-an-azure-subscription"></a>Qu’est-ce qu’un abonnement Azure ?

Un abonnement Azure est similaire à un groupe de ressources : il s’agit d’une construction logique qui regroupe des groupes de ressources et leurs ressources. Toutefois, un abonnement Azure est également associé aux contrôles utilisés par Azure Resource Manager. Qu’est-ce que cela signifie ? Examinez plus en détail Azure Resource Manager pour en savoir plus sur sa relation avec un abonnement Azure.

![Diagramme d’un abonnement Azure](../../_images/governance-1-11.png)
*Figure 3. Un abonnement Azure.*

## <a name="what-is-azure-resource-manager"></a>Qu’est-ce qu’Azure Resource Manager ?

Dans [Comment fonctionne Azure ?](../../getting-started/what-is-azure.md), vous avez appris qu’Azure comprend un « serveur frontal » avec de nombreux services qui orchestrent toutes les fonctions d’Azure. L’un de ces services, [Azure Resource Manager](/azure/azure-resource-manager/), héberge l’API RESTful utilisée par les clients pour gérer les ressources.

![Diagramme d’Azure Resource Manager](../../_images/governance-1-12.png)
*Figure 4. Azure Resource Manager.*

La figure suivante montre trois clients : [PowerShell](/powershell/azure/overview), le [portail Azure](https://portal.azure.com) et l’[interface de ligne de commande (CLI) Azure](/cli/azure) :

![Diagramme de clients Azure se connectant à l’API Azure Resource Manager](../../_images/governance-1-13.png)
*Figure 5. Les clients Azure se connectent à l’API RESTful Azure Resource Manager.*

Bien que ces clients se connectent à Azure Resource Manager à l’aide de l’API RESTful, Azure Resource Manager n’inclut pas de fonctionnalités permettant de gérer directement les ressources. Au lieu de cela, la plupart des types de ressources Azure ont leur propre [**fournisseur de ressources**](/azure/azure-resource-manager/resource-group-overview#terminology).

![Fournisseurs de ressources Azure](../../_images/governance-1-14.png)
*Figure 6. Fournisseurs de ressources Azure.*

Quand un client effectue une requête pour gérer une ressource spécifique, Azure Resource Manager se connecte au fournisseur de ressources pour ce type de ressources afin d’exécuter la requête. Par exemple, si un client effectue une requête pour gérer une ressource de machine virtuelle, Azure Resource Manager se connecte au fournisseur de ressources **Microsoft.Compute**.

![Azure Resource Manager se connectant au fournisseur de ressources Microsoft.Compute](../../_images/governance-1-15.png)
*Figure 7. Azure Resource Manager se connecte au fournisseur de ressources **Microsoft.Compute** pour gérer la ressource spécifiée dans la requête du client.*

Azure Resource Manager exige du client qu’il spécifie un identificateur pour l’abonnement et le groupe de ressources, afin de gérer la ressource de machine virtuelle.

Maintenant que vous comprenez le fonctionnement d’Azure Resource Manager, revenez à la présentation de la façon dont un abonnement Azure est associé aux contrôles utilisés par Azure Resource Manager. Avant qu’une requête de gestion des ressources ne soit exécutée par Azure Resource Manager, plusieurs contrôles sont vérifiés.

Le premier contrôle est qu’une requête doit être effectuée par un utilisateur validé, et qu’Azure Resource Manager a une relation de confiance avec [Azure Active Directory (Azure AD)](/azure/active-directory/) pour fournir des fonctionnalités d’identité utilisateur.

![Azure Active Directory](../../_images/governance-1-16.png)
*Figure 8. Azure Active Directory.*

Dans Azure AD, les utilisateurs sont segmentés sous forme de **clients**. Un client est une construction logique qui représente une instance d’Azure AD dédiée et sécurisée, généralement associée à une organisation. Chaque abonnement est associé à un client Azure AD.

![Un locataire Azure AD associé à un abonnement](../../_images/governance-1-17.png)
*Figure 9. Un client Azure AD associé à un abonnement.*

Chaque requête du client pour gérer une ressource dans un abonnement spécifique requiert que l’utilisateur dispose d’un compte dans le client Azure AD associé.

Le contrôle suivant consiste à vérifier que l’utilisateur dispose des autorisations suffisantes pour effectuer la requête. Les autorisations sont attribuées aux utilisateurs à l’aide du [contrôle d’accès en fonction du rôle (RBAC)](/azure/role-based-access-control/).

![Des utilisateurs affectés aux rôles RBAC](../../_images/governance-1-18.png)
*Figure 10. Un ou plusieurs rôles RBAC sont attribués à chaque utilisateur du client.*

Un rôle RBAC spécifie un ensemble d’autorisations qu’un utilisateur peut utiliser pour une ressource spécifique. Lorsque le rôle est attribué à l’utilisateur, ces autorisations s’appliquent. Par exemple, le [rôle de **propriétaire** intégré](/azure/role-based-access-control/built-in-roles#owner) autorise un utilisateur à effectuer une action sur une ressource.

Le contrôle suivant vérifie que la requête est autorisée sous les paramètres spécifiés dans la [stratégie de ressources Azure](/azure/governance/policy/). Les stratégies de ressources Azure spécifient les opérations autorisées pour une ressource spécifique. Par exemple, une stratégie de ressources Azure peut spécifier que les utilisateurs sont uniquement autorisés à déployer un type spécifique de machine virtuelle.

![Stratégie de ressource Azure](../../_images/governance-1-19.png)
*Figure 11. Stratégie de ressource Azure.*

Le contrôle suivant consiste à vérifier que la requête ne dépasse pas une [limite d’abonnement Azure](/azure/azure-subscription-service-limits). Par exemple, chaque abonnement dispose d’une limite de 980 groupes de ressources par abonnement. Si une requête pour déployer un groupe de ressources supplémentaire est reçue une fois la limite atteinte, elle est refusée.

![Limites des ressources Azure](../../_images/governance-1-20.png)
*Figure 12. Limites des ressources Azure.*

Le contrôle final vérifie que la requête est comprise dans l’engagement financier associé à l’abonnement. Par exemple, si la requête doit déployer une machine virtuelle, Azure Resource Manager vérifie que l’abonnement dispose des informations de paiement suffisantes.

![Un engagement financier associé à un abonnement](../../_images/governance-1-21.png)
*Figure 13. Un engagement financier est associé à un abonnement.*

## <a name="summary"></a>Résumé

Dans cet article, vous avez étudié la gestion des accès aux ressources dans Azure à l’aide d’Azure Resource Manager.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous comprenez comment gérer l’accès aux ressources dans Azure, poursuivez pour en savoir plus sur la conception d’un modèle de gouvernance [pour une charge de travail simple](governance-simple-workload.md) ou pour [plusieurs équipes](governance-multiple-teams.md) à l’aide de ces services.

> [!div class="nextstepaction"]
> [Une vue d’ensemble de la gouvernance](../overview.md)
