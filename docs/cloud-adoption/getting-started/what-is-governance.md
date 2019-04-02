---
title: 'Framework d’adoption du cloud : Qu’est-ce que la gouvernance des ressources cloud ?'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.date: 02/11/2019
description: Fonctionnement de la gouvernance des ressources cloud sur Azure
author: petertaylor9999
ms.openlocfilehash: 602ac81f2b2201c77746df971d282582ceee23f7
ms.sourcegitcommit: 9854bd27fb5cf92041bbfb743d43045cd3552a69
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/27/2019
ms.locfileid: "58503194"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-is-cloud-resource-governance"></a>Qu’est-ce que la gouvernance des ressources cloud ?

Dans l’article [Comment fonctionne Azure ?](what-is-azure.md), vous avez appris qu’Azure est une collection de serveurs et d’équipements de mise en réseau sur lesquels fonctionnent du matériel virtualisé et des logiciels pour le compte des utilisateurs. Azure garantit l’agilité des développeurs et des services informatiques de votre organisation en les aidant à créer, lire, mettre à jour et supprimer des ressources en fonction de leurs besoins.

Mais si le fait d’accorder aux développeurs un accès illimité aux ressources les fait considérablement gagner en agilité, une telle approche peut également entraîner des coûts inattendus. Par exemple, une équipe de développement peut être autorisée à déployer un ensemble de ressources à des fins de test mais oublier de les supprimer une fois les tests terminés. Ces ressources continueront d’engendrer des coûts, même si leur utilisation n’est plus nécessaire ou autorisée.

La **gouvernance** de l’accès aux ressources offre un moyen de résoudre ce problème. La gouvernance consiste à gérer, surveiller et contrôler en continu les ressources Azure afin d’atteindre les objectifs et de satisfaire aux exigences de votre organisation.

<!-- markdownlint-disable MD034 -->

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2ii94]

<!-- markdownlint-enable MD034 -->

Ces objectifs et exigences étant propres à chaque organisation, il est impossible de définir une approche unique de la gouvernance. C’est pourquoi Azure implémente deux principaux outils de gouvernance : le **contrôle d’accès en fonction du rôle (RBAC)** et la **stratégie de ressources**. Il appartient à chaque organisation d’utiliser ces outils pour concevoir son propre modèle de gouvernance.

Le RBAC définit des rôles, qui eux-mêmes définissent les fonctionnalités dont dispose un utilisateur à qui est attribué le rôle. Par exemple, le rôle de **propriétaire** active toutes les fonctionnalités (création, lecture, mise à jour et suppression) pour une ressource, tandis que les rôles de **lecteur** activent uniquement la fonction de lecture. Les rôles peuvent couvrir une vaste étendue qui s’applique à de nombreux types de ressources. Ils peuvent aussi avoir une portée étroite qui s’applique à seulement quelques ressources.

Les stratégies de ressources définissent des règles pour la création de ressources. Par exemple, une stratégie de ressources peut limiter la référence SKU d’une machine virtuelle à une taille donnée pré-approuvée. Une stratégie de ressources peut également appliquer l’ajout d’une étiquette auprès d’un centre de coûts au moment de la demande de création de la ressource.

Lors de la configuration de ces outils, il est important de veiller à garantir le meilleur compromis entre gouvernance et agilité organisationnelle. Autrement dit, plus votre stratégie de gouvernance sera restrictive, moins vos développeurs et vos ingénieurs informatiques seront agiles. Il s’agit, car une stratégie de gouvernance restrictif peut nécessiter plus d’étapes manuelles, telles que le développeur d’avoir à remplir un formulaire ou envoyez un e-mail à une personne de l’équipe de gouvernance pour créer manuellement une ressource. L’équipe de gouvernance possède des fonctionnalités limitées et peut être retardé, ce qui entraîne des équipes de développement non productives en attente de leurs ressources comptabiliser les coûts en attente de suppression de ressources créé et inutiles.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous comprenez le concept de gouvernance des ressources de cloud, en savoir plus sur la gestion des accès aux ressources dans Azure.

> [!div class="nextstepaction"]
> [En savoir plus sur l’accès aux ressources dans Azure](azure-resource-access.md)
