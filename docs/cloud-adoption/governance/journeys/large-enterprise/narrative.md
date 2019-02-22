---
title: 'Framework d’adoption du cloud : Scénario de gouvernance pour les grandes entreprises'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Ce scénario définit un cas d’usage pour un cheminement de la gouvernance des grandes entreprises.
author: BrianBlanchard
ms.openlocfilehash: 2f8e18a7a845d39fd5aa1632a9d025a502518a94
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901150"
---
# <a name="large-enterprise-the-narrative-behind-the-governance-strategy"></a>Grande entreprise : Le scénario sous-tendant la stratégie de gouvernance

Le scénario suivant définit un cas d’usage pour un [cheminement de la gouvernance des grandes entreprises](./overview.md). Avant d’implémenter le cheminement, il est important de comprendre les hypothèses et le raisonnement qui sont reflétés dans ce scénario. Vous pouvez ensuite mieux aligner la stratégie de gouvernance au cheminement de votre propre organisation.

## <a name="back-story"></a>Ce qui s’est passé

Les clients veulent une meilleure expérience lors des interactions avec cette entreprise. L’expérience actuelle a provoqué une érosion du marché et a conduit la direction à embaucher un directeur de la transformation numérique (CDO, Chief Digital Officer). Le directeur de la transformation numérique collabore avec les départements Marketing et Ventes pour piloter une transformation numérique qui permettra d’améliorer les expériences. En outre, plusieurs divisions ont récemment embauché des scientifiques des données pour gérer et exploiter les données, et pour améliorer de nombreuses expériences manuelles via l’apprentissage et la prédiction. Le département informatique fournit un support pour ces travaux là où il peut le faire. Cependant, des activités de type « shadow IT » se font en dehors des contrôles nécessaires en matière de gouvernance et de sécurité.

L’organisation du département informatique doit également faire face à ses propres impératifs. Le département Finance prévoit des réductions continues dans le budget informatique pour les cinq prochaines années, ce qui impose des coupes dans les dépenses à partir de cette année. À l’inverse, le RGPD et d’autres règles de souveraineté des données obligent le département informatique à investir dans des ressources pour placer les données dans d’autres pays. Deux des centres de données existants ont pris du retard dans l’actualisation nécessaire de leur matériel, ce qui provoque des problèmes de satisfaction des employés et des clients. Trois autres centres de données nécessitent des actualisations de leur matériel dans le cadre de l’exécution du plan à cinq ans. Le directeur financier pousse la directrice informatique à envisager le cloud en guise d’alternative pour ces centres de données, afin de diminuer les dépenses d’investissement.

La directrice informatique a des idées novatrices qui peuvent aider l’entreprise, mais elle-même et ses équipes doivent se limiter à parer au plus pressé et à contrôler les coûts. Lors d’un déjeuner avec le directeur de la transformation numérique et un des responsables d’une division, la conversation sur la migration vers le cloud a intéressé les collègues de la directrice informatique. Les trois responsables ont l’intention de se soutenir mutuellement dans l’utilisation du cloud pour atteindre leurs objectifs métier, et ils ont commencé à explorer et à planifier les phases d’adoption du cloud.

## <a name="business-characteristics"></a>Caractéristiques de l’entreprise

L’entreprise a le profil d’activité suivant :

- Les ventes et l’exploitation s’étendent sur plusieurs zones géographiques avec des clients internationaux dans plusieurs marchés.
- L’entreprise a grandi via des acquisitions et elle fonctionne avec trois divisions établies selon la base des clients cibles. La budgétisation se fait via une matrice complexe entre les divisions et les fonctions.
- L’entreprise voit l’informatique principalement comme des investissements à perte ou un centre de coûts.

## <a name="current-state"></a>État actuel

Voici l’état actuel de l’informatique et de l’exploitation du cloud dans l’entreprise :

- Le département informatique exploite plus de 20 centres de données privés dans le monde.
- Chaque centre de données est connecté par une série de lignes louées régionales, créant un réseau WAN faiblement couplé.
- Le département informatique est entré dans le cloud en migrant tous les comptes de messagerie des utilisateurs finaux vers Office 365. Cette migration s’est terminée il y a plus de six mois. Depuis lors, seules quelques autres ressources informatiques ont été déployés sur le cloud.
- L’équipe de développement principale du directeur de la transformation numérique travaille dans une capacité de développement/test pour découvrir les fonctionnalités natives du cloud.
- Une division expérimente le Big Data dans le cloud. L’équipe Décisionnel, qui fait partie du département informatique, participe à ce travail.
- La stratégie de gouvernance informatique existante stipule que les informations d’identification personnelle et les données financières doivent être hébergées sur des ressources détenues directement par l’entreprise. Cette stratégie bloque l’adoption du cloud pour toutes les applications critiques ou les données protégées.
- Les investissements informatiques sont contrôlés principalement par les dépenses d’investissement. Ces investissements sont planifiés annuellement et incluent souvent des plans pour la maintenance continue ainsi que des cycles d’actualisation établis de trois à cinq années, selon le centre de données.
- La plupart des investissements dans la technologie qui ne sont pas alignés sur le plan annuel sont relatifs aux travaux du shadow IT. Ces travaux sont généralement gérés par les différentes divisions et sont financés via les dépenses de fonctionnement des divisions.

## <a name="future-state"></a>État futur

Les changements suivants sont prévus dans les prochaines années :

- Le directeur informatique conduit un travail visant à moderniser la stratégie sur les données financières et les informations d’identification personnelle pour prendre en charge les objectifs futurs. Deux membres de l’équipe Gouvernance informatique ont une visibilité sur ce travail.
- Si les premières expérimentations dans le développement des applications et le décisionnel montrent des indicateurs de réussite, chacun souhaiterait mettre en œuvre des solutions de production à petite échelle dans le cloud au cours des 24 prochains mois.
- La directrice informatique et le directeur financier ont demandé à un architecte et au vice-président en charge de l’infrastructure de faire une analyse des coûts et une étude de faisabilité. Ces travaux détermineront si l’entreprise peut et doit déplacer 5 000 ressources dans le cloud au cours des 36 prochains mois. Une migration réussie permettrait à la directrice informatique d’éliminer deux centres de données, ce qui réduirait les coûts de plus de 100 millions de dollars sur le plan à cinq ans. Si trois ou quatre centres de données peuvent obtenir des résultats similaires, le budget reviendra dans le vert, donnant à la directrice informatique le budget nécessaire pour soutenir plus d’initiatives innovantes.
    ![Les coûts locaux par rapport aux coûts Azure montrent des économies de 100 millions de dollars au cours des cinq prochaines années](../../../_images/governance/calculator-enterprise.png)
- En plus de ces économies, l’entreprise prévoit de changer la gestion de certains investissements informatiques en repositionnant les dépenses d’investissement validées comme dépenses d’exploitation au sein du département informatique. Ce changement aboutira à un meilleur contrôle des coûts, ce qui permettra au département informatique d’accélérer les autres travaux planifiés.

## <a name="next-steps"></a>Étapes suivantes

L’entreprise a développé une stratégie d’entreprise pour modeler l’implémentation de la gouvernance. La stratégie d’entreprise pilote de nombreuses décisions techniques.

> [!div class="nextstepaction"]
> [Passer en revue la stratégie d’entreprise initiale](./initial-corporate-policy.md)
