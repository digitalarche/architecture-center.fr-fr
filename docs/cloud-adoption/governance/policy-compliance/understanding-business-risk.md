---
title: 'Framework d’adoption du cloud : Comment le risque métier change dans le cloud ?'
description: Présentation du risque métier pendant la migration
author: BrianBlanchard
ms.date: 10/10/2018
ms.openlocfilehash: 458474f3c94c5df4f7ffef439095adf138f33d78
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901154"
---
<!-- markdownlint-disable MD026 -->

# <a name="how-does-business-risk-change-in-the-cloud"></a>Comment le risque métier change dans le cloud ?

Une bonne compréhension du risque métier est l’un des éléments les plus importants de toute transformation cloud. Le risque entraîne une stratégie. Il influence la supervision et l’application des exigences. Le risque influence fortement la façon dont nous gérons le patrimoine numérique, localement ou dans le cloud.

<!-- markdownlint-enable MD026 -->

## <a name="relativity-of-risk"></a>Relativité du risque

Le risque est relatif. Une petite entreprise disposant de quelques ressources informatiques dans un bâtiment fermé est à faible risque. L’ajout d’utilisateurs et d’une connexion Internet avec un accès à ces ressources accroît le risque. Quand cette petite entreprise se développe au point de figurer dans le classement Fortune 500, les risques augmentent de manière exponentielle. À mesure que le chiffre d’affaires, les processus métier, les nombres d’employés et les ressources informatiques s’accumulent, les risques s’accroissent et fusionnent. Les ressources informatiques qui favorisent la génération de revenus sont exposées à un risque tangible d’arrêt de cette source de revenus en cas de panne. Chaque moment d’arrêt se traduit par des pertes. De même, à mesure que les données s’accumulent, le risque de préjudice pour les clients augmente.

Dans le monde local traditionnel, les équipes de gouvernance informatique se concentrent sur l’évaluation de ces risques. Elles créent des processus permettant d’atténuer le risque. Elles déploient des systèmes pour garantir que les mesures d’atténuation sont efficaces et implémentées. Cela équilibre les risques nécessaires pour opérer dans un environnement d’activité moderne et connecté.

## <a name="understanding-business-risks-in-the-cloud"></a>Comprendre les risques métier dans le cloud

Lors d’une transformation, les mêmes risques relatifs peuvent être observés.

* Pendant une expérimentation anticipée, quelques ressources sont déployées avec peu ou pas de données pertinentes. Le risque est faible.
* Quand la première charge de travail est déployée, le risque augmente un peu. Ce risque est facilement atténué en choisissant une application à faible risque par nature avec une base d’utilisateurs réduite.
* À mesure que le nombre de charges de travail mises en ligne augmente, les risques changent à chaque mise en production. La publication de nouvelles applications fait évoluer les risques.
* Quand une entreprise met en ligne les 10 à 20 premières applications, le profil de risque est très différent de celui de la mise en production de la 1 000ème application dans le cloud.

L’accumulation des ressources dans le patrimoine local traditionnel s’est probablement effectuée au fil du temps. La maturité des équipes commerciales et informatiques se développait vraisemblablement de la même manière. Cette croissance parallèle peut avoir tendance à créer un bagage de stratégies inutiles.

Lors d’une transformation cloud, les équipes commerciales et informatiques ont toutes les deux la possibilité de réinitialiser ces stratégies et d’en générer de nouvelles avec un esprit évolué.

<!-- markdownlint-disable MD026 -->

## <a name="what-is-a-business-risk-mvp"></a>Qu’est un MVP de risque métier ?

**Produit minimum viable** (MVP, Minimum Viable Product) est un terme standard utilisé pour définir la plus petite unité de quelque chose qui peut produire une valeur tangible. Dans un MVP de risque professionnel, l’équipe part du principe que certaines ressources seront déployées dans un environnement cloud. À ce moment-là, ces ressources ne sont pas connues. Les types de données qui seront traités par ces ressources sont également inconnus.

L’équipe de gouvernance cloud pourrait générer le scénario le plus défavorable en mappant chaque stratégie possible au cloud. Cela n’est pas conseillé, mais c’est une option.

À l’inverse, l’équipe pourrait adopter une approche de MVP et définir un point de départ et un ensemble d’hypothèses qui seraient vraies pour la plupart/la totalité des ressources.
Voici quelques exemples extrêmement simples :

* Toutes les ressources sont exposées au risque d’être arrêtées (à cause d’une erreur, d’une méprise ou d’une maintenance)
* Toutes les ressources sont exposées au risque de générer trop de dépenses
* Toutes les ressources pourraient être compromises par des mots de passe faibles
* Toute ressource dont tous les ports ouverts sont exposés à Internet court un risque de compromission

Les exemples ci-dessus sont destinés à établir les risques métier de MVP comme une théorie. La liste réelle sera propre à chaque environnement.
Une fois les MVP de risque métier établis, ils peuvent être convertis en [stratégies](overview.md) afin d’atténuer chaque risque.

<!-- markdownlint-enable MD026 -->

## <a name="incremental-risk-mitigation"></a>Atténuation incrémentielle des risques

En supposant qu’un MVP de risque métier est le point de départ, la gouvernance peut évoluer en parallèle du déploiement planifié (au lieu de croître parallèlement à la croissance de l’entreprise). Il s’agit d’un modèle beaucoup plus stable pour la maturité de la gouvernance. À chaque itération, de nouvelles ressources sont répliquées et envoyées en préproduction. À chaque mise en production, les charges de travail sont préparées pour la promotion vers la production. Bien entendu, le risque relatif peut augmenter avec chaque cycle.

Quand l’équipe de gouvernance cloud opère en parallèle des équipes de l’adoption du cloud, la croissance des risques métier peut également être prise en compte. Chaque ressource en préproduction peut facilement être classée en fonction du risque. Les documents de classification des données peuvent être générés ou créés en parallèle de la préproduction. Le profil de risque et les points d’exposition peuvent également être documentés. Au fil du temps, une vision très claire du risque métier se précisera dans l’organisation.

Avec chaque itération, l’équipe de gouvernance cloud peut travailler avec l’équipe de stratégie cloud pour communiquer rapidement les nouveaux risques, les stratégies d’atténuation, les compromis et les coûts potentiels. Cette option permet aux participants de l’entreprise et aux responsables informatiques d’être partenaires dans des décisions éclairées et matures. Ces décisions informent alors sur la maturité de la stratégie. Si nécessaire, les changements de stratégie produisent de nouveaux éléments de travail pour la maturité de systèmes d’infrastructure centrale. Quand il est nécessaire d’apporter des changements à des systèmes en préproduction, les équipes d’adoption du cloud ont suffisamment de temps pour cette opération, pendant que l’entreprise teste les systèmes en préproduction et développe un plan d’adoption par les utilisateurs.

Cette approche limite les risques, tout en permettant à l’équipe de migrer rapidement. Elle garantit également que les risques sont identifiés et résolus rapidement avant le déploiement.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Évaluer la tolérance au risque](./risk-tolerance.md)
