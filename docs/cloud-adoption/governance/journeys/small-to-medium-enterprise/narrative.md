---
title: 'Framework d’adoption du cloud : Petite ou moyenne entreprise – Scénario initial sous-tendant la stratégie de gouvernance'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Ce scénario définit un cas d’usage pour un cheminement de la gouvernance des petites et moyennes entreprises.
author: BrianBlanchard
ms.openlocfilehash: 6173b01f310169017761110d6641acfa51d45df8
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901189"
---
# <a name="small-to-medium-enterprise-the-narrative-behind-the-governance-strategy"></a>Petites et moyennes entreprises : Le scénario sous-tendant la stratégie de gouvernance

Le scénario suivant décrit le cas d’usage pour le [cheminement de la gouvernance des petites et moyennes entreprises](./overview.md). Avant d’implémenter le cheminement, il est important de comprendre les hypothèses et le raisonnement qui sont reflétés dans ce scénario. Vous pouvez ensuite mieux aligner la stratégie de gouvernance au cheminement de votre propre organisation.

## <a name="back-story"></a>Ce qui s’est passé

Le conseil d’administration a démarré l’année avec des plans pour dynamiser l’activité de plusieurs façons. Ils incitent la direction à améliorer l’expérience client de façon à gagner des parts de marché. Ils veulent aussi offrir de nouveaux produits et services qui vont positionner l’entreprise comme leader dans le secteur. Ils ont également initié un travail en parallèle pour réduire le gaspillage et réduire les coûts inutiles. Bien qu’intimidantes, les actions du conseil d’administration et de la direction montrent que ce travail doit être consacré autant que possible à une croissance future.

Dans le passé, le directeur informatique de la société était exclu de ces discussions stratégiques. Cependant, comme la vision du futur est intrinsèquement liée à une croissance technique, l’informatique a maintenant un siège à la table de réunion pour aider à réaliser ces grands projets. Le département informatique doit désormais fournir ses services selon de nouveaux modes. L’équipe n’est pas vraiment préparée pour ces changements et elle aura probablement des difficultés avec la courbe d’apprentissage.

## <a name="business-characteristics"></a>Caractéristiques de l’entreprise

L’entreprise a le profil d’activité suivant :

- Toutes les ventes et les opérations se font dans un même pays, avec un faible pourcentage de clients du monde entier.
- L’entreprise fonctionne comme une seule division, avec un budget aligné sur les fonctions, notamment Ventes, Marketing, Exploitation et Informatique.
- L’entreprise voit l’informatique principalement comme des investissements à perte ou un centre de coûts.

## <a name="current-state"></a>État actuel

Voici l’état actuel de l’informatique et de l’exploitation du cloud dans l’entreprise :

- Le département informatique exploite deux environnements d’infrastructure hébergés. Un environnement contient des ressources de production. Le deuxième environnement contient les ressources de reprise d’activité et certaines ressources de développement/test. Ces environnements sont hébergés par deux fournisseurs différents. Le département informatique appelle ces deux centres Prod (Production) et DR (Disaster Recovery, Reprise d’activité), respectivement.
- Le département informatique est entré dans le cloud en migrant tous les comptes de messagerie des utilisateurs finaux vers Office 365. Cette migration s’est terminée il y a six mois. Quelques autres ressources informatiques ont été déployés sur le cloud.
- Les équipes de développement d’applications travaillent dans une capacité de développement/test pour découvrir les fonctionnalités natives du cloud.
- L’équipe Décisionnel (BI) expérimente le Big Data dans le cloud et la collecte des données sur de nouvelles plateformes.
- L’entreprise a une stratégie peu contraignante indiquant que les informations d’identification personnelle information et les données financières ne peuvent pas être hébergées dans le cloud, ce qui limite les applications critiques dans les déploiements actuels.
- Les investissements informatiques sont contrôlés principalement par les dépenses d’investissement. Ces investissements sont planifiés chaque année. Au cours des dernières années, les investissements ont très peu porté sur autre chose que les besoins en maintenance de base.

## <a name="future-state"></a>État futur

Les changements suivants sont prévus dans les prochaines années :

- Le directeur informatique examine la stratégie relative aux informations d’identification personnelle et aux données financières de façon à pouvoir atteindre les objectifs fixés pour le futur.
- Les équipes Développement d’applications et les équipes Décisionnel veulent passer en production les solutions cloud au cours des 24 prochains mois, conformément aux perspectives souhaitées pour l’engagement client et les nouveaux produits.
- Cette année, l’équipe informatique achèvera la mise hors service des charges de travail de reprise d’activité du centre de données DR en migrant 2 000 machines virtuelles vers le cloud. Ceci doit normalement induire des économies de 25 millions de dollars sur les cinq prochaines années.
    ![Les coûts locaux par rapport aux coûts Azure montrent des économies de 25 millions de dollars au cours des cinq prochaines années](../../../_images/governance/calculator-small-to-medium-enterprise.png)
- L’entreprise prévoit de changer la façon dont elle investit dans l’informatique en repositionnant les dépenses d’investissement validées comme dépenses d’exploitation au sein du département informatique. Ce changement aboutira à un meilleur contrôle des coûts et permettra au département informatique d’accélérer les autres travaux planifiés.

## <a name="next-steps"></a>Étapes suivantes

L’entreprise a développé une stratégie d’entreprise pour modeler l’implémentation de la gouvernance. La stratégie d’entreprise pilote de nombreuses décisions techniques.

> [!div class="nextstepaction"]
> [Passer en revue la stratégie d’entreprise initiale](./initial-corporate-policy.md)
