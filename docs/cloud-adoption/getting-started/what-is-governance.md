---
title: 'Framework d’adoption du cloud : Qu’est-ce que la gouvernance des ressources cloud ?'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.date: 02/11/2019
description: Fonctionnement de la gouvernance des ressources cloud sur Azure
author: petertaylor9999
ms.openlocfilehash: 0989a5aad8a6290cce07fd71771c690bbd615e0d
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59068852"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-is-cloud-resource-governance"></a>Qu’est-ce que la gouvernance des ressources cloud ?

Dans [comment fonctionne Azure ?](what-is-azure.md), vous avez appris qu’Azure est une collection de serveurs et du matériel en cours d’exécution un matériel virtualisé et logiciels pour le compte utilisateurs réseau. Azure permet le développement d’applications et services informatiques à être agile en facilitant la créer, lire, mettre à jour et supprimer des ressources en fonction des besoins de votre organisation.

Toutefois, pendant un accès illimité aux ressources peut rendre les développeurs très souple, il peut également entraîner les coûts inattendus. Par exemple, une équipe de développement peut être autorisée à déployer un ensemble de ressources à des fins de test mais oublier de les supprimer une fois les tests terminés. Ces ressources continueront à s’accumuler les coûts, même si elles ne sont plus approuvées ou nécessaires.

La solution est la gouvernance des accès aux ressources. **Gouvernance** est le processus en cours de la gestion, la surveillance et l’audit de l’utilisation des ressources Azure pour satisfaire les besoins de votre organisation.

<!-- markdownlint-disable MD034 -->

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2ii94]

<!-- markdownlint-enable MD034 -->

Ces exigences sont uniques à chaque organisation, donc une approche unique de la gouvernance n’est pas utile. Au lieu de cela, c’est à chaque organisation pour concevoir leur modèle de gouvernance à l’aide de deux outils de gouvernance principale d’Azure : **contrôle d’accès en fonction du rôle (RBAC)** et **stratégie de ressource**.

Définit les rôles RBAC et les rôles définissent les fonctionnalités de chaque utilisateur affecté ce rôle. Par exemple, le **propriétaire** rôle permet à toutes les fonctionnalités (créer, lire, mettre à jour et supprimer) pour une ressource, tandis que le **lecteur** rôle permet uniquement la fonctionnalité de lecture. Les rôles peuvent être définis avec une large étendue s’applique à de nombreux types de ressources ou d’une portée étroite s’applique à quelques.

Les stratégies de ressources définissent des règles pour la création de ressources. Par exemple, une stratégie de ressource peut limiter la référence (SKU) d’un ordinateur virtuel à une taille particulière pré-approuvée. Une autre stratégie de ressource peut imposer l’application d’une balise pour un centre de coût affecté lors de la demande est faite pour créer la ressource.

Lorsque vous configurez ces outils, il est important d’équilibrer la gouvernance et réactivité de l’entreprise. Le plus restrictif votre stratégie de gouvernance, la moins souple seront de vos développeurs et le personnel des services informatiques. Une stratégie de gouvernance restrictif peut nécessiter plus d’étapes manuelles par exemple, exigez un développeur remplir un formulaire ou envoyez un e-mail à un membre de l’équipe de gouvernance pour créer manuellement une ressource. L’équipe de gouvernance a une capacité finie et peut devenir un goulot d’étranglement, ce qui entraîne des équipes de développement en attente unproductively pour leurs ressources soient créées ou inutiles ressources comptabiliser des coûts avant d’être supprimés.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous comprenez le concept de gouvernance des ressources de cloud, en savoir plus sur la gestion des accès aux ressources dans Azure.

> [!div class="nextstepaction"]
> [En savoir plus sur l’accès aux ressources dans Azure](azure-resource-access.md)
