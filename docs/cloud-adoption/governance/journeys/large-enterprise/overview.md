---
title: 'Framework d’adoption du cloud : Parcours de gouvernance pour les grandes entreprises'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Parcours de gouvernance pour les grandes entreprises
author: BrianBlanchard
ms.openlocfilehash: 689b600524c3be6c505ade8c5aaa447d772c6e35
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900763"
---
# <a name="caf-large-enterprise-governance-journey"></a>Framework d’adoption du cloud : Parcours de gouvernance pour les grandes entreprises

## <a name="best-practice-overview"></a>Présentation des meilleures pratiques

Ce parcours de gouvernance suit les expériences d’une entreprise fictive à différents stades de maturité. Il est basé sur le parcours de clients réels. Les meilleures pratiques recommandées se fondent sur les contraintes et les besoins de l’entreprise fictive.

Comme point de départ rapide, cette présentation définit un produit minimum viable (MVP) pour la gouvernance, basé sur les meilleures pratiques. Elle fournit également des liens vers des évolutions de gouvernance, qui ajoutent des meilleures pratiques à mesure que de nouveaux risques métier et techniques émergent.

> [!WARNING]
> Ce MVP est un point de départ de base, qui se fonde sur un ensemble de postulats. Même cet ensemble minimal de meilleures pratiques se fonde sur des stratégies d’entreprise, qui sont axées sur des risques métier et des tolérances aux risques uniques. Pour déterminer si ces postulats s’appliquent à vous, lisez la [plus longue description](./narrative.md) qui suit cet article.

### <a name="governance-best-practice"></a>Meilleures pratiques de gouvernance

Ces meilleures pratiques servent de bases sur lesquelles une organisation peut s’appuyer pour ajouter de manière cohérente et rapide des protections de gouvernance à plusieurs abonnements Azure.

### <a name="resource-organization"></a>Organisation des ressources

Le schéma suivant montre la hiérarchie MVP de gouvernance pour organiser les ressources.

![Schéma d’organisation des ressources](../../../_images/governance/resource-organization.png)

Chaque application doit être déployée dans la zone appropriée du groupe d’administration, de l’abonnement et de la hiérarchie des groupes de ressources. Lors de la planification du déploiement, l’équipe de gouvernance cloud crée les nœuds nécessaires dans la hiérarchie pour donner aux équipes d’adoption du cloud les moyens d’agir.

1. Un groupe de gestion pour chaque unité commerciale avec une hiérarchie détaillée qui reflète la géographie puis le type d’environnement (Production, Non-Production).
2. Un abonnement pour chaque combinaison unique d’unité commerciale, de géographie, d’environnement et de « Catégorisation d’applications ».
3. Un groupe de ressources séparé pour chaque application.
4. Une nomenclature cohérente doit être appliquée à chaque niveau de cette hiérarchie de regroupement.

![Schéma d’organisation des ressources des grandes entreprises](../../../_images/governance/large-enterprise-resource-organization.png)

Ces modèles laissent de l’espace pour la croissance sans compliquer la hiérarchie inutilement.

[!INCLUDE [governance-of-resources](../../../../../includes/cloud-adoption/governance/governance-of-resources.md)]

## <a name="governance-evolutions"></a>Évolutions de la gouvernance

Une fois que ce MVP a été déployé, des couches supplémentaires de gouvernance peuvent être rapidement intégrées à l’environnement. Voici quelques méthodes permettant de faire évoluer le MVP afin de répondre aux besoins spécifiques de l’entreprise :

- [Base de référence de sécurité pour les données protégées](./security-baseline-evolution.md)
- [Configurations des ressources pour les applications critiques](./resource-consistency-evolution.md)
- [Contrôles pour la gestion des coûts](./cost-management-evolution.md)
- [Contrôles pour l’évolution multi-cloud](./multi-cloud-evolution.md)

<!-- markdownlint-disable MD026 -->

## <a name="what-does-this-best-practice-do"></a>Concrètement, que font les meilleures pratiques ?

Dans le MVP, les pratiques et les outils venant de la discipline [Accélération du déploiement](../../deployment-acceleration/overview.md) ont été conçus pour appliquer rapidement la stratégie d’entreprise. Plus précisément, le MVP utilise Azure Blueprints, Azure Policy et les groupes d’administration Azure pour appliquer quelques stratégies d’entreprise basiques, comme le définit le scénario pour cette entreprise fictive. Ces stratégies d’entreprise sont appliquées à l’aide de modèles Azure Resource Manager et de stratégies Azure afin d’établir une petite base de référence pour l’identité et la sécurité.

![Exemple de MVP de gouvernance incrémentielle](../../../_images/governance/governance-mvp.png)

## <a name="evolving-the-best-practice"></a>Évolution des meilleures pratiques

Au fil du temps, ce MVP de gouvernance servira à faire évoluer les pratiques de gouvernance. À mesure que le processus d’adoption avance, les risques métier augmentent. Différentes disciplines au sein du modèle de framework d’adoption du cloud évolueront pour atténuer ces risques. D’autres articles de cette série traitent de l’évolution de la stratégie d’entreprise et de son impact sur l’entreprise fictive. Ces évolutions se produisent dans trois disciplines :

- Base de référence des identités, à mesure que la migration des dépendances évolue dans le scénario
- La gestion des coûts, à mesure que l’adoption avance.
- La base de référence de la sécurité, à mesure que les données protégées sont déployées.
- La cohérence des ressources, à mesure que les opérations informatiques commencent à gérer les charges de travail critiques.

![Exemple de MVP de gouvernance incrémentielle](../../../_images/governance/governance-evolution-large.png)

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous connaissez le MVP de gouvernance et que vous avez une idée des évolutions de gouvernance à suivre, lisez le scénario correspondant pour en savoir plus.

> [!div class="nextstepaction"]
> [Lire le scénario correspondant](./narrative.md)
