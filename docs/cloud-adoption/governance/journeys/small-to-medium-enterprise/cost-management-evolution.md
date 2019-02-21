---
title: "Framework d'adoption du cloud : Petites et moyennes entreprises - Évolution de la gestion des coûts  "
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 2/11/2019
description: Explication - Petites et moyennes entreprises - Évolution de la gestion des coûts
author: BrianBlanchard
ms.openlocfilehash: ca070bfca3efbf9548faa8cf28a1adffefd4a33a
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900727"
---
# <a name="small-to-medium-enterprise-cost-management-evolution"></a>Petites et moyennes entreprises : Évolution de la gestion des coûts

Cet article fait évoluer le scénario en ajoutant des contrôles des coûts au produit minimum viable (MVP) de la gouvernance.

## <a name="evolution-of-the-narrative"></a>Évolution du scénario

L'adoption a dépassé l'indicateur de tolérance des coûts défini dans le MVP de la gouvernance. Il s'agit d'une bonne chose car cela correspond aux migrations entreprises depuis le centre de données de récupération d'urgence. L'augmentation des dépenses justifie désormais un investissement en temps de la part de l'équipe de gouvernance cloud.

### <a name="evolution-of-the-current-state"></a>Évolution de l'état actuel

Lors de la phase précédente de ce scénario, le service informatique avait mis hors service 100 % du centre de données de récupération d'urgence. Les équipes de développement d'applications et BI étaient prêtes pour le trafic de production.

Depuis, certains changements ont eu lieu et ceux-ci vont avoir un impact sur la gouvernance :

- L'équipe de migration a commencé à migrer les machines virtuelles hors du centre de données de production.
- Les équipes de développement d'applications envoient (push) activement les applications de production vers le cloud par le biais de pipelines CI/CD. Ces applications peuvent s'adapter de manière réactive aux demandes des utilisateurs.
- L'équipe décisionnelle du service informatique a fourni un certain nombre d'outils d'analyse prédictive dans le cloud. Les volumes de données agrégées dans le cloud continuent de croître.
- Cette croissance va dans le sens des engagements pris en matière de résultats opérationnels. En revanche, les coûts ont commencé à s'envoler. Les projections budgétaires augmentent plus rapidement que prévu. Le directeur financier a besoin de revoir les approches en matière de gestion des coûts.

### <a name="evolution-of-the-future-state"></a>Évolution de l'état futur

Des fonctionnalités de contrôle des coûts et de reporting doivent être ajoutées à la solution cloud. Le service informatique fait toujours office de chambre de compensation des coûts. Cela signifie que les achats informatiques continuent à financer les services cloud. Le reporting devrait cependant lier les dépenses opérationnelles directes aux fonctions qui absorbent les coûts du cloud. Ce modèle est appelé « Show Back » en termes de comptabilité cloud.

Les modifications apportées aux états actuel et futur exposent à de nouveaux risques qui nécessiteront de nouvelles déclarations de stratégie.

## <a name="evolution-of-tangible-risks"></a>Évolution des risques tangibles

**Contrôle du budget** : Les fonctionnalités en libre-service risquent de générer des coûts excessifs et inattendus sur la nouvelle plateforme. Des processus de gouvernance permettant de contrôler les coûts et d'atténuer les risques permanents liés à ceux-ci doivent être en place pour garantir un alignement continu avec le budget prévu.

Ce risque métier peut être étendu à quelques risques techniques :

- Les coûts réels peuvent dépasser le plan.
- La situation commerciale évolue. Dans ce cas, il peut arriver qu'une fonction métier utilise plus de services cloud que prévu, ce qui se traduit par des anomalies en termes de dépenses. Ces dépenses supplémentaires risquent alors d'être considérées comme des dépassements, par opposition à un ajustement nécessaire du plan.
- Les systèmes peuvent être surapprovisionnés et, par conséquent, générer des dépenses excessives.

## <a name="evolution-of-the-policy-statements"></a>Évolution des déclarations de stratégie

Les modifications suivantes apportées à la stratégie contribueront à atténuer les nouveaux risques et à guider l'implémentation.

1. Chaque semaine, l'équipe de gouvernance doit s'assurer que tous les coûts liés au cloud sont conformes au plan. Chaque mois, des rapports sur les écarts entre les coûts du cloud et le plan doivent être remis aux responsables informatiques et au service financier. Chaque mois, toutes les mises à jour des coûts liés au cloud et du plan doivent être contrôlées par les responsables informatiques et le service financier.
2. Tous les coûts doivent être affectés à une fonction opérationnelle à des fins de responsabilisation.
3. Les ressources du cloud doivent être contrôlées en permanence afin de déceler les possibilités d'optimisation.
4. Les outils de gouvernance du cloud doivent limiter les options de dimensionnement des ressources à une liste approuvée de configurations. Les outils doivent permettre à la solution de contrôle des coûts de détecter et de suivre toutes les ressources.
5. Lors de la planification du déploiement, toutes les ressources cloud requises associées à l'hébergement de charges de travail de production doivent faire l'objet d'une documentation. Cette documentation permettra d'affiner les budgets et de préparer une automatisation supplémentaire pour éviter l'utilisation d'options plus coûteuses. Au cours de ce processus, il convient d'envisager l'adoption de différents outils de remise proposés par le fournisseur de cloud, tels que des instances réservées ou des réductions sur le coût des licences.
6. Tous les propriétaires d'applications sont tenus de suivre une formation sur les pratiques d'optimisation des charges de travail afin de mieux contrôler les coûts liés au cloud.

## <a name="evolution-of-the-best-practices"></a>Évolution des meilleures pratiques

Cette section de l'article va faire évoluer la conception du produit minimum viable (MVP) de la gouvernance afin d'inclure de nouvelles stratégies Azure, ainsi qu'une implémentation d'Azure Cost Management. Ensemble, ces deux modifications de la conception permettront de répondre aux nouvelles déclarations de la stratégie d'entreprise.

1. Implémenter Azure Cost Management
    1. Établissez la bonne étendue d'accès pour l'aligner sur le modèle d'abonnement et la discipline de cohérence des ressources. Un alignement sur le MVP de la gouvernance défini dans les articles précédents nécessitera un accès à l'**étendue Compte d'inscription** pour l'équipe de gouvernance cloud qui exécutera le reporting de haut niveau. D'autres équipes extérieures à la gouvernance pourront avoir besoin d'un accès à l'**étendue Groupe de ressources**.
    2. Établissez un budget dans Azure Cost Management.
    3. Consultez les recommandations initiales et agissez en fonction. Prévoyez un processus récurrent pour le reporting.
    4. Configurez et exécutez le Reporting d'Azure Cost Management, à la fois initial et récurrent.
2. Mettre à jour Azure Policy
    1. Vérifiez les valeurs de balisage, de groupe de gestion, d'abonnement et de groupe de ressources pour identifier tout écart.
    2. Définissez des options de taille pour les références SKU afin de limiter les déploiements aux références SKU répertoriées dans la documentation de planification du déploiement.

## <a name="conclusion"></a>Conclusion

L'ajout de ces processus et modifications au MVP de la gouvernance permet d'atténuer de nombreux risques associés à la gouvernance des coûts. Ensemble, ils fournissent la visibilité, la responsabilisation et l'optimisation nécessaires pour contrôler les coûts.

## <a name="next-steps"></a>Étapes suivantes

À mesure que l'adoption du cloud évolue et offre davantage de valeur aux activités métier, les risques et besoins en matière de gouvernance du cloud doivent aussi évoluer. Pour notre entreprise fictive, l'étape suivante consiste à utiliser cet investissement en termes de gouvernance pour gérer plusieurs clouds.

> [!div class="nextstepaction"]
> [Évolution multicloud](./multi-cloud-evolution.md)
