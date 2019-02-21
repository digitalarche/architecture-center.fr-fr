---
title: "Framework d'adoption du cloud : Aligner les guides de conception sur la stratégie."
description: Comment aligner les guides de conception sur la stratégie ?
author: BrianBlanchard
ms.date: 01/04/2019
ms.openlocfilehash: 77a35597585e5f58967ea79d3c7b0fa17b6ab80e
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900734"
---
<!---
I've established policies. How to help developers adopt these policies?
Draft an architecture design guide.

[Aspirational statement] If you're using Azure, you can use one of ours as a starting point. The choose one of the following 6 as a starting point and mold it to fit your policies.
--->

<!-- markdownlint-disable MD026 -->

# <a name="how-do-you-align-design-guides-with-policy"></a>Comment aligner les guides de conception sur la stratégie ?

Après avoir [défini des stratégies cloud](define-policy.md) basées sur les [risques que vous avez identifiés](understanding-business-risk.md), vous devez générer des conseils exploitables, conformes à ces stratégies et auxquels le personnel informatique et les développeurs pourront se référer. La rédaction d'un guide de conception de gouvernance cloud vous permet de spécifier des choix spécifiques en matière de structure, de technologie et de processus conformément aux déclarations de stratégie que vous avez générées pour chacune des cinq disciplines de gouvernance.

Pour chacun des composants d'infrastructure de base des déploiements cloud, un guide de conception de gouvernance cloud doit établir les choix d'architecture et les modèles de conception qui répondent le mieux à vos exigences en matière de stratégie. En outre, vous devez fournir une explication détaillée sur les technologies, outils et processus qui soutiendront chacune de ces décisions de conception.

Même si votre analyse des risques et vos déclarations de stratégie sont plus ou moins indépendantes de la plateforme cloud, votre guide de conception doit fournir des détails d'implémentation spécifiques à la plateforme pour votre personnel informatique et vos développeurs. Concentrez-vous sur l'architecture, les outils et les fonctionnalités de la plateforme que vous avez choisie pour prendre des décisions de conception et fournir des conseils.

Les guides de conception cloud doivent tenir compte de certains détails techniques associés à chaque composant d'infrastructure, mais il ne s'agit pas pour autant de spécifications ou de documents techniques exhaustifs. Veillez à ce que vos guides traitent de toutes vos déclarations de stratégie et énoncent clairement les décisions de conception dans un format facile à comprendre et à consulter pour le personnel.

<!-- markdownlint-enable MD033 -->

## <a name="using-the-actionable-governance-journeys"></a>Utiliser les parcours de gouvernance exploitables

Si vous envisagez d'utiliser la plateforme Azure pour votre adoption du cloud, le Framework d'adoption du cloud (CAF) fournit des [parcours de gouvernance](../journeys/overview.md) illustrant l'approche incrémentielle du modèle de gouvernance CAF. Ces parcours narratifs couvrent un ensemble de scénarios d'adoption courants, notamment sur les risques métier, les exigences de tolérance et les déclarations de stratégie ayant permis de créer un produit minimum viable (MVP). Ces parcours représentent une synthèse du processus d'adoption du cloud tel qu'il est vécu par un client réel dans Azure.

Bien que chaque adoption de cloud présente des objectifs, des priorités et des enjeux uniques, ces exemples constituent un bon modèle pour convertir votre stratégie en conseils. Choisissez le scénario le plus proche de votre situation comme point de départ et adaptez-le à vos besoins spécifiques en termes de stratégie.

## <a name="next-steps"></a>Étapes suivantes

Une fois les conseils de conception en place, développez des processus d'adhésion à la stratégie pour garantir la mise en conformité avec celle-ci.

> [!div class="nextstepaction"]
> [Développer des processus d'adhésion à la stratégie](processes.md)