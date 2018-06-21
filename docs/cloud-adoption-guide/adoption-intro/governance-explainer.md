---
title: 'Explicatif : qu’est-ce que la gouvernance cloud ?'
description: Explique le concept de gouvernance des ressources pour Azure et le cloud
author: petertay
ms.openlocfilehash: 3404beaa719177ee7638feed8a8442b5c3b455c6
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206478"
---
# <a name="explainer-what-is-cloud-resource-governance"></a>Explicatif : qu’est-ce que la gouvernance des ressources cloud ?

Dans l’article explicatif [Comment fonctionne Azure ?](azure-explainer.md), vous avez appris qu’Azure est une collection de serveurs et d’équipements de mise en réseau sur lesquels fonctionnent du matériel virtualisé et des logiciels pour le compte des utilisateurs. Azure garantit l’agilité des développeurs et des services informatiques de votre organisation en les aidant à créer, lire, mettre à jour et supprimer des ressources en fonction de leurs besoins.

Mais si le fait d’accorder aux développeurs un accès illimité aux ressources les fait considérablement gagner en agilité, une telle approche peut également entraîner des coûts inattendus. Par exemple, une équipe de développement peut être autorisée à déployer un ensemble de ressources à des fins de test mais oublier de les supprimer une fois les tests terminés. Ces ressources continueront d’engendrer des coûts, même si leur utilisation n’est plus nécessaire ou autorisée. 

La **gouvernance** de l’accès aux ressources offre un moyen de résoudre ce problème. La gouvernance consiste à gérer, surveiller et contrôler en continu les ressources Azure afin d’atteindre les objectifs et de satisfaire aux exigences de votre organisation. 

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2ii94] 

Ces objectifs et exigences étant propres à chaque organisation, il est impossible de définir une approche unique de la gouvernance. C’est pourquoi Azure implémente deux principaux outils de gouvernance : le **contrôle d’accès basée sur les ressources (RBAC)** et la **stratégie de ressources**. Il appartient à chaque organisation d’utiliser ces outils pour concevoir son propre modèle de gouvernance.

Le RBAC définit des rôles, qui eux-mêmes définissent les fonctionnalités dont dispose un utilisateur à qui est attribué le rôle. Par exemple, le rôle de **propriétaire** active toutes les fonctionnalités (création, lecture, mise à jour et suppression) pour une ressource, tandis que les rôles de **lecteur** activent uniquement la fonction de lecture. Les rôles peuvent couvrir une vaste étendue qui s’applique à de nombreux types de ressources. Ils peuvent aussi avoir une portée étroite qui s’applique à seulement quelques ressources. 

Les stratégies de ressources définissent des règles pour la création de ressources. Par exemple, une stratégie de ressources peut limiter la référence SKU d’une machine virtuelle à une taille donnée pré-approuvée. Une stratégie de ressources peut également appliquer l’ajout d’une étiquette auprès d’un centre de coûts au moment de la demande de création de la ressource. 

Lors de la configuration de ces outils, il est important de veiller à garantir le meilleur compromis entre gouvernance et agilité organisationnelle. Autrement dit, plus votre stratégie de gouvernance sera restrictive, moins vos développeurs et vos ingénieurs informatiques seront agiles. Une stratégie de gouvernance restrictive peut en effet impliquer davantage d’étapes manuelles, par exemple exiger d’un développeur de remplir un formulaire ou d’envoyer un e-mail à une personne de l’équipe de gouvernance pour créer manuellement une ressource. L’équipe de gouvernance a des capacités limitées et peut être retardée. Résultat : en attendant que leurs ressources soient créées, les équipes de développement ne sont plus productives et les ressources inutiles entraînent une augmentation des coûts tant qu’elles n’ont pas été supprimées.

## <a name="next-steps"></a>Étapes suivantes

Les étapes suivantes de votre adoption d’Azure consistent à [comprendre l’identité numérique dans Azure](tenant-explainer.md) et à [créer votre premier utilisateur dans Azure AD][docs-add-users-to-aad].

<!-- Links -->

[docs-add-users-to-aad]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json