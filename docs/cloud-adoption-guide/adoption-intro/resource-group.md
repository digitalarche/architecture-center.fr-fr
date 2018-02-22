---
title: "Guide : conception d’un groupe de ressources Azure"
description: "Guide pour la conception d’un groupe de ressources Azure dans le cadre d’une stratégie d’adoption cloud fondamentale"
author: petertay
ms.openlocfilehash: ac6cbb03be8cdba020641d3b9034ad9d20101acf
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/09/2018
---
# <a name="guidance-azure-resource-group-design"></a>Guide : conception d’un groupe de ressources Azure

Dans Azure, un [groupe de ressources](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview#resource-groups) est un conteneur logique dans lequel les ressources sont regroupées. Chaque ressource déployée dans Azure doit être déployée dans un seul groupe de ressources.

## <a name="design-considerations"></a>Remarques relatives à la conception

- Toutes les ressources d’un groupe de ressources doivent avoir le même cycle de vie. Autrement dit, vous devez déployer, mettre à jour et supprimer des ressources avec le même cycle de vie qu’un groupe. Par exemple, les ressources de calcul d’une application web unique sont généralement déployées comme une seule unité. Toutefois, une base de données partagée avec d’autres applications web serait très probablement gérée dans un autre cycle de vie, et doit se trouver dans son propre groupe de ressources.
- Un groupe de ressources peut contenir des ressources figurant dans différentes régions.
- Toutes les ressources d’un seul groupe de ressources doivent être associées à un abonnement unique. 
- Une ressource peut être déplacée entre des groupes de ressources, mais pas vers un groupe de ressources contenant une ressource déployée dans un autre abonnement.
- L’affectation d’un groupe de ressources n’a pas d’impact sur la connectivité ou l’interaction avec les ressources d’autres groupes de ressources. Par exemple, une machine virtuelle affectée à un groupe de ressources peut se connecter à une base de données affectée à un autre groupe de ressources s’il existe une connectivité réseau entre les deux.
- Un groupe de ressources peut être utilisé pour définir l’étendue du contrôle d’accès des actions administratives. Vous pouvez appliquer des autorisations de contrôle d’accès en fonction du rôle (RBAC) au niveau de l’abonnement ou au niveau du groupe de ressources. Toutes les autorisations affectées au niveau de l’abonnement sont héritées au niveau du groupe de ressources. Vous en apprendrez davantage sur RBAC et les autorisations liées aux ressources dans les étapes d’adoption intermédiaire et avancée.

## <a name="proven-practices"></a>Pratiques éprouvées

- Durant la phase fondamentale, vous gérez probablement uniquement des projets de type preuve de concept (POC), chacun avec un petit nombre de ressources. Étant donné que les ressources POC partagent généralement le même cycle de vie, vous pouvez créer un seul groupe de ressources pour chacun de ces projets.
- Durant la phase d’adoption intermédiaire, vous allez gérer plusieurs projets. Différents types de projets peuvent bénéficier d’autres conceptions en termes de groupes de ressources. Si vous envisagez de faire passer l’un de vos projets POC initiaux en production, vous pouvez déplacer des ressources vers un autre groupe de ressources s’il appartient au même abonnement. Par conséquent, à ce stade vous devez déployer ces ressources dans le même abonnement afin de pouvoir réorganiser les ressources à l’avenir.

## <a name="next-steps"></a>étapes suivantes

* À présent que vous avez découvert les pratiques éprouvées pour la phase d’adoption fondamentale, vous êtes en mesure de créer des groupes de ressources et d’y ajouter des ressources. Même si vous ne gérez qu’un petit nombre de ressources à ce stade, leur gestion devient plus complexe au fur et à mesure que vous ajoutez davantage de ressources. Apprenez-en plus sur [Conventions d’affectation de noms et balisage Azure](/azure/architecture/best-practices/naming-conventions?toc=/azure/architecture/cloud-adoption-guide/toc.json) pour nommer et baliser vos ressources en préparation de la phase d’adoption intermédiaire.
