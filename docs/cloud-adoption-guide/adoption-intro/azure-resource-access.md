---
title: Présentation de la gestion des accès aux ressources dans Azure
description: 'Explication de la structure de gestion de l’accès aux ressources dans Azure : gestion des ressources Azure, abonnements, groupes de ressources et ressources'
author: petertay
ms.openlocfilehash: 02e25e943822ba065d360d687d34283d86a805bd
ms.sourcegitcommit: c704d5d51c8f9bbab26465941ddcf267040a8459
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/24/2018
ms.locfileid: "39229502"
---
# <a name="understanding-resource-access-management-in-azure"></a>Présentation de la gestion des accès aux ressources dans Azure

Comme vous l’avez appris dans l’article [Qu’est-ce que la gouvernance de ressource ?](governance-explainer.md), la gouvernance consiste à gérer, surveiller et contrôler en continu les ressources Azure afin d’atteindre les objectifs et de satisfaire aux exigences de votre organisation. Avant de découvrir comment concevoir un modèle de gouvernance, il est important de comprendre les contrôles de gestion des accès aux ressources dans Azure. La configuration de ces contrôles de gestion des accès aux ressources constitue la base de votre modèle de gouvernance.

Nous allons commencer en examinant plus en détail comment les ressources sont déployées dans Azure. 

## <a name="what-is-an-azure-resource"></a>Qu’est-ce qu’une ressource Azure ?

Dans Azure, le terme **ressource** fait référence à une entité gérée par Azure. Par exemple, les machines virtuelles, les réseaux virtuels et les comptes de stockage sont tous considérés comme des ressources Azure.

![](../_images/governance-1-9.png)   
*Figure 1 : Une ressource.*

## <a name="what-is-an-azure-resource-group"></a>Qu’est-ce qu’un groupe de ressources Azure ?

Chaque ressource dans Azure doit appartenir à un [groupe de ressources](/azure/azure-resource-manager/resource-group-overview#resource-groups). Un groupe de ressources est simplement une construction logique qui regroupe plusieurs ressources afin qu’elles puissent être gérées en tant qu’entité unique. Par exemple, des ressources qui partagent un cycle de vie similaire, tels que les ressources d’une [application multi-niveau](/azure/architecture/guide/architecture-styles/n-tier), peuvent être créées ou supprimées en tant que groupe. 

![](../_images/governance-1-10.png)   
*Figure 2 : Un groupe de ressources contient une ressource.* 

Les groupes de ressources et les ressources qu’ils contiennent sont associés à un **abonnement** Azure. 

## <a name="what-is-an-azure-subscription"></a>Qu’est-ce qu’un abonnement Azure ?

Un abonnement Azure est similaire à un groupe de ressources : il s’agit d’une construction logique qui regroupe des groupes de ressources et leurs ressources. Toutefois, un abonnement Azure est également associé aux contrôles utilisés par Azure Resource Manager. Qu’est-ce que cela signifie ? Examinons plus en détail Azure Resource Manager pour en savoir plus sur sa relation avec un abonnement Azure.

![](../_images/governance-1-11.png)   
*Figure 3 : Un abonnement Azure.*

## <a name="what-is-azure-resource-manager"></a>Qu’est-ce qu’Azure Resource Manager ?

Dans [Comment fonctionne Azure ?](azure-explainer.md), vous avez appris qu’Azure comprend un « serveur frontal » avec de nombreux services qui orchestrent toutes les fonctions d’Azure. Un de ces services est le gestionnaire de ressources [Azure Resource Manager](/azure/azure-resource-manager/), et ce service héberge l’API RESTful utilisée par les clients pour gérer les ressources. 

![](../_images/governance-1-12.png)   
*Figure 4 : Azure Resource Manager.*

L’illustration suivante montre trois clients : [Powershell](/powershell/azure/overview), [le portail Azure](https://portal.azure.com) et l’[interface de ligne de commande (CLI) Azure](/cli/azure) :

![](../_images/governance-1-13.png)   
*Figure 5 : Les clients Azure se connectent à l’API RESTful d’Azure Resource Manager.*

Bien que ces clients se connectent à Azure Resource Manager à l’aide de l’API RESTful, Azure Resource Manager n’inclut pas de fonctionnalités pour gérer directement les ressources. Au lieu de cela, la plupart des types de ressources Azure ont leur propre [**fournisseur de ressources**](/azure/azure-resource-manager/resource-group-overview#terminology). 

![](../_images/governance-1-14.png)   
*Figure 6 : Fournisseurs de ressources Azure.*

Quand un client envoie une requête pour gérer une ressource spécifique, le gestionnaire de ressources Azure Resource Manager se connecte au fournisseur de ressources de ce type de ressources pour terminer la requête. Par exemple, si un client envoie une requête pour gérer une ressource de machine virtuelle, Azure Resource Manager se connecte au fournisseur de ressources **Microsoft.compute**. 

![](../_images/governance-1-15.png)   
*Figure 7 : Azure Resource Manager se connecte au fournisseur de ressources **Microsoft.compute** pour gérer la ressource spécifiée dans la requête du client.*

Notez que le gestionnaire de ressources Azure Resource Manager demande au client de spécifier un identificateur pour l’abonnement et le groupe de ressources pour gérer la ressource de machine virtuelle. 

Maintenant que vous comprenez le fonctionnement d’Azure Resource Manager, revenons à notre discussion sur la façon dont un abonnement Azure est associé aux contrôles utilisés par Azure Resource Manager. Avant qu’une requête de gestion des ressources ne soit exécutée par Azure Resource Manager, plusieurs contrôles sont vérifiés. 

Le premier contrôle est qu’une requête doit être effectuée par un utilisateur validé, et le gestionnaire de ressources Azure Resource Manager a une relation de confiance avec [Azure Active Directory (Azure AD)](/azure/active-directory/) pour fournir des fonctionnalités d’identité utilisateur.

![](../_images/governance-1-16.png)   
*Figure 8 : Azure Active Directory.*

Dans Azure AD, les utilisateurs sont segmentés sous forme de **clients**. Un client est une construction logique qui représente une instance d’Azure AD dédiée et sécurisée, généralement associée à une organisation. Chaque abonnement est associé à un client Azure AD.

![](../_images/governance-1-17.png)   
*Figure 9 : Un client Azure AD associé à un abonnement.*

Chaque requête du client pour gérer une ressource dans un abonnement spécifique requiert que l’utilisateur dispose d’un compte dans le client Azure AD associé. 

Le contrôle suivant consiste à vérifier que l’utilisateur dispose des autorisations suffisantes pour effectuer la requête. Les autorisations sont attribuées aux utilisateurs à l’aide du [contrôle d’accès en fonction du rôle (RBAC)](/azure/role-based-access-control/).

![](../_images/governance-1-18.png)   
*Figure 10 : Un ou plusieurs rôles RBAC sont attribués à chaque utilisateur du client.*

Un rôle RBAC spécifie un ensemble d’autorisations qu’un utilisateur peut utiliser pour une ressource spécifique. Lorsque le rôle est attribué à l’utilisateur, ces autorisations s’appliquent. Par exemple, le [rôle de **propriétaire** intégré](/azure/role-based-access-control/built-in-roles#owner) autorise un utilisateur à effectuer une action sur une ressource.

Le contrôle suivant vérifie que la requête est autorisée sous les paramètres spécifiés dans la [stratégie de ressources Azure](/azure/azure-policy/). Les stratégies de ressources Azure spécifient les opérations autorisées pour une ressource spécifique. Par exemple, une stratégie de ressources Azure peut spécifier que les utilisateurs sont uniquement autorisés à déployer un type spécifique de machine virtuelle.

![](../_images/governance-1-19.png)   
*Figure 11 : Stratégie de ressource Azure.*

Le contrôle suivant consiste à vérifier que la requête ne dépasse pas une [limite d’abonnement Azure](/azure/azure-subscription-service-limits). Par exemple, chaque abonnement dispose d’une limite de 980 groupes de ressources par abonnement. Si une requête pour déployer un groupe de ressources supplémentaire est reçue une fois la limite atteinte, elle est refusée.

![](../_images/governance-1-20.png)   
*Figure 12 : Limites des ressources Azure.* 

Le contrôle final vérifie que la requête est comprise dans l’engagement financier associé à l’abonnement. Par exemple, si la requête requiert le déploiement d’une machine virtuelle, Azure Resource Manager vérifie que l’abonnement dispose des informations de paiement suffisantes.

![](../_images/governance-1-21.png)   
*Figure 13 : Un engagement financier est associé à un abonnement.*

# <a name="summary"></a>Résumé

Dans cet article, vous avez découvert comment l’accès aux ressources est géré dans Azure à l’aide d’Azure Resource Manager.

# <a name="next-steps"></a>Étapes suivantes

Maintenant que vous comprenez comment l’accès aux ressources est géré dans Azure, poursuivez pour en savoir plus sur la [conception d’un modèle de gouvernance](governance-how-to.md) à l’aide de ces services.