---
title: 'Framework d’adoption du cloud : Exemples d’instructions de la stratégie de gestion des coûts'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 1/4/2019
description: Exemples d’instructions de la stratégie de gestion des coûts
author: BrianBlanchard
ms.openlocfilehash: 1c8b94ae5285fa26cdf9760892beaced2487af8b
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901113"
---
# <a name="cost-management-sample-policy-statements"></a>Exemples d’instructions de la stratégie de gestion des coûts

Les instructions de stratégie cloud individuelles sont des recommandations destinées à traiter des risques spécifiques identifiés lors de votre processus d’évaluation des risques. Ces instructions doivent fournir un bref récapitulatif des risques, ainsi que des plans établis pour les gérer. Chaque définition d’instruction doit inclure les informations suivantes :

- Risque métier : récapitulatif du risque à gérer par cette stratégie.
- Instruction de stratégie : explication succincte et claire des exigences de la stratégie.
- Options de conception : recommandations exploitables, spécifications ou autres conseils que des équipes informatiques et des développeurs peuvent utiliser lors de l’implémentation de la stratégie.

Les exemples d’instructions de stratégie suivants gèrent un certain nombre des risques métier courants liés aux coûts. Ils sont fournis à titre d’exemples que vous pouvez consulter lors de l’élaboration d’instructions de stratégie réelles répondant aux besoins de votre propre organisation. Ces exemples ne sont pas destinés à être normatifs, et il existe potentiellement plusieurs options de stratégie pour gérer un seul risque identifié. Travaillez en étroite collaboration avec les équipes métier et informatiques pour identifier les meilleures solutions de stratégie pour vos risques liés aux coûts.  

## <a name="future-proofing"></a>Pérennité

**Risque métier**. Critères actuels qui ne garantissent pas que l’équipe de gouvernance investisse dans une discipline Gestion des coûts. Toutefois, vous prévoyez un tel investissement à l’avenir.

**Instruction de stratégie**. Vous devez associer toutes les ressources déployées dans le cloud à une unité de facturation, une application/charge de travail et une norme d’affectation de noms. Cette stratégie garantit que les futurs efforts de gestion des coûts seront efficaces.

**Options de conception**. Pour en savoir plus sur la manière d’établir une base pérenne, consultez les discussions sur la création d’un MVP de gouvernance dans les [guides de conception exploitables](../journeys/overview.md), qui font partie des recommandations CAF.

## <a name="budget-overruns"></a>Dépassements de budget

**Risque métier :** Le déploiement en libre-service crée un risque de dépassement du budget.

**Instruction de stratégie :** Tout déploiement cloud doit être affecté à une unité de facturation avec un budget approuvé et un mécanisme de limites budgétaires.

**Options de conception :** Dans Azure, le budget peut être contrôlé avec [Azure Cost Management](/azure/cost-management/manage-budgets)

## <a name="underutilization"></a>Sous-utilisation

**Risque métier :** L’entreprise a prépayé les services cloud ou a signé un contrat annuel l’engageant à dépenser un montant spécifique. Il existe un risque que le montant convenu ne soit pas utilisé, et que ce soit donc un investissement perdu.

**Instruction de stratégie :** Chaque unité de facturation qui dispose d’un budget cloud alloué se réunit tous les ans pour définir les budgets, tous les trimestres pour ajuster les budgets, et tous les mois pour analyser les dépenses réelles par rapport aux dépenses planifiées. Chaque mois, discutez des écarts supérieurs à 20 % avec le responsable de l’unité de facturation. À des fins de suivi, affectez toutes les ressources à une unité de facturation.

**Options de conception :**

- Dans Azure, les dépenses planifiées versus les dépenses réelles peuvent être gérées dans [Azure Cost Management](/azure/cost-management/quick-acm-cost-analysis)
- Il existe plusieurs options pour regrouper les ressources par unité de facturation. Dans Azure, un [modèle de cohérence des ressources](../../decision-guides/resource-consistency/overview.md) doit être choisi en collaboration avec l’équipe de gouvernance et appliqué à toutes les ressources.

## <a name="overprovisioned-assets"></a>Ressources surapprovisionnées

**Risque métier :** Dans les centres de données locaux traditionnels, il est courant de déployer des ressources avec une planification de capacité supplémentaire afin d’anticiper la croissance à venir. Le cloud peut effectuer une mise à l’échelle plus rapide que les équipements traditionnels. Les ressources dans le cloud sont également facturées en fonction de leur capacité technique. Il existe un risque que l’ancienne pratique en local fasse augmenter artificiellement les dépenses cloud.

**Instruction de stratégie :** Toute ressource déployée dans le cloud doit être inscrite dans un programme permettant de superviser l’utilisation et de signaler toute capacité dépassant 50 % d’utilisation. Toute ressource déployée dans le cloud doit être regroupée ou balisée de manière logique. De cette manière, les membres de l’équipe de gouvernance peuvent s’adresser au propriétaire de la charge de travail concernant toute optimisation des ressources surapprovisionnées.

**Options de conception :**

- Dans Azure, [Azure Advisor](/azure/advisor/advisor-cost-recommendations) peut fournir des recommandations vis-à-vis de l’optimisation.
- Il existe plusieurs options pour regrouper les ressources par unité de facturation. Dans Azure, un [modèle de cohérence des ressources](../../decision-guides/resource-consistency/overview.md) doit être choisi en collaboration avec l’équipe de gouvernance et appliqué à toutes les ressources.

## <a name="overoptimization"></a>Sur-optimisation

**Risque métier :** Une gestion des coûts efficace peut créer de nouveaux risques. L’optimisation des dépenses peut nuire aux performances du système. En réduisant les coûts, il existe le risque de trop restreindre les dépenses et d’offrir des expériences utilisateur de qualité médiocre.

**Instruction de stratégie :** Toute ressource qui a un impact direct sur les expériences client doit être identifiée via le regroupement ou le balisage. Avant d’optimiser une ressource ayant un impact sur l’expérience client, l’équipe de gouvernance cloud doit ajuster l’optimisation en se basant sur une tendance d’utilisation supérieure à 90 jours. Lors de l’optimisation des ressources, documentez les pics saisonniers ou basés sur un événement qui ont été pris en compte.

**Options de conception :**

- Dans Azure, les [fonctionnalités d’insights d’Azure Monitor](/azure/azure-monitor/insights/vminsights-performance) peuvent aider à analyser l’utilisation du système.
- Il existe plusieurs options pour regrouper et baliser les ressources en fonction des rôles. Dans Azure, vous devez choisir un [modèle de cohérence des ressources](../../decision-guides/resource-consistency/overview.md) en collaboration avec l’équipe de gouvernance et l’appliquer à toutes les ressources.

## <a name="next-steps"></a>Étapes suivantes

Utilisez les exemples mentionnés dans cet article comme point de départ pour développer des stratégies qui répondent à des risques métier spécifiques, dans la continuité de vos plans d’adoption du cloud.

Pour commencer à développer vos instructions de stratégie personnalisées pour la gestion des coûts, téléchargez le [modèle de gestion des coûts](template.md).

Pour accélérer l’adoption de cette discipline, choisissez le [parcours de gouvernance exploitable](../journeys/overview.md) correspondant le mieux à votre environnement. Modifiez ensuite la conception pour intégrer vos décisions spécifiques en matière de stratégie d’entreprise.

> [!div class="nextstepaction"]
> [Parcours de gouvernance exploitables](../journeys/overview.md)
