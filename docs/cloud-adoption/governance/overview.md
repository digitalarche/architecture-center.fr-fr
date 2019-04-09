---
title: 'Framework d’adoption du cloud : Gouvernance dans le CAF Microsoft pour Azure'
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Gouvernance dans le CAF Microsoft pour Azure
author: BrianBlanchard
ms.openlocfilehash: ce407de0daa590e767382346692c80e0a113bb3c
ms.sourcegitcommit: 0a8a60d782facc294f7f78ec0e9033e3ee16bf4a
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/08/2019
ms.locfileid: "59068835"
---
# <a name="governance-in-the-microsoft-caf-for-azure"></a>Gouvernance dans le CAF Microsoft pour Azure

Le cloud crée de nouveaux paradigmes à propos des technologies sur lesquelles repose l’entreprise. Ces nouveaux paradigmes transforment également la façon dont ces technologies sont adoptées, gérées et régies. Alors qu’il est désormais possible de détruire et de recréer intégralement un centre de données grâce à une seule ligne de code exécutée à partir d’un processus sans assistance, nous devons repenser les approches traditionnelles. Cette affirmation est également vraie pour tout ce qui touche à la gouvernance.

Pour les organisations qui disposent déjà de stratégies régissant les environnements informatiques locaux, la gouvernance cloud vient en complément de ces stratégies. Cependant, le niveau d’intégration entre les stratégies d’entreprise locales et cloud varie en fonction de la maturité de la gouvernance cloud et du patrimoine numérique dans le cloud. Les processus de gouvernance et les stratégies cloud évoluent au même rythme que le patrimoine cloud.

Les instructions de cette section du framework d’adoption du cloud visent deux objectifs :

* Proposer des parcours client immédiatement actionnables qui illustrent des expériences communes rencontrées par les clients. Chacun de ces parcours inclut les risques commerciaux, les stratégies d’entreprise pour l’atténuation des risques et des conseils de conception pour implémenter des solutions techniques. Par définition, les conseils de conception sont propres à Azure. Tous les autres conseils de ces parcours peuvent être appliqués dans le cadre d’une approche indépendante du cloud ou multicloud.
* Aider les lecteurs à créer des solutions de gouvernance personnalisées capables de satisfaire plusieurs des besoins de l’entreprise, notamment la gouvernance de plusieurs clouds publics grâce à des instructions détaillées sur le développement de stratégies, processus et outils appliqués à l’entreprise.

Ce contenu s’adresse à l’équipe de gouvernance cloud. Il peut également servir aux architectes du cloud qui cherchent à développer des fondements solides pour la gouvernance cloud.

## <a name="audience"></a>Audience

Le contenu du framework d’adoption du cloud touche l’activité, la technologie et la culture des entreprises. Cette section du framework d’adoption du cloud implique des interactions avec les équipes chargées de la sécurité informatique, la gouvernance informatique et la finance, avec les responsables des applications cœur de métier et avec les équipes chargées de la mise en réseau, l’identité et l’adoption du cloud. Plusieurs codépendances sont à noter entre ces acteurs et nécessitent une approche facilitante de la part des architectes du cloud qui font usage de ce guide. Il se peut que la facilitation entre les équipes découle d’une action ponctuelle, mais dans certains cas, elle requiert des interactions récurrentes entre les acteurs.

L’architecte du cloud joue le rôle de facilitateur et de leader d’opinion pour rassembler tous ces publics. Le contenu figurant dans cette collection de guides est conçu pour aider l’architecte cloud à adapter les discussions en fonction des publics et ainsi prendre les décisions nécessaires. La transformation de l’entreprise qu’engendre le cloud repose sur le rôle de l’architecte du cloud qui oriente les décisions des équipes commerciales et informatiques.

**Spécialisation Architecte du cloud dans cette section :** Chaque section du framework d’adoption du cloud représente une spécialisation ou une variante différente du rôle de l’architecte du cloud. Cette section de la CAF est conçue pour les architectes cloud avec une passion pour atténuer ou en réduisant les risques techniques. De nombreux fournisseurs de cloud nomment ces spécialistes des « gardiens du cloud ». Nous préférons nous y référer en tant « qu’équipe de gouvernance cloud ». Chacun des parcours client immédiatement actionnables comporte des articles détaillant la composition et le rôle de l’équipe de gouvernance cloud et son évolution au fil du temps.

## <a name="using-this-guide"></a>Comment utiliser ce guide

Pour les lecteurs qui souhaitent suivre ce guide du début à la fin, ce contenu les aidera à développer une stratégie de gouvernance cloud fiable en même temps qu’ils implémentent le cloud. Les différentes instructions guident le lecteur à travers la théorie et la mise en pratique de cette stratégie.

Pour un cours intensif sur la théorie et un accès rapide à l’implémentation Azure, prise en main le [vue d’ensemble de parcours de gouvernance actionnables](./journeys/overview.md). Grâce à ce guide, le lecteur peut commencer pas à pas et faire évoluer ses besoins de gouvernance en même temps que ses efforts d’adoption cloud.

## <a name="next-steps"></a>Étapes suivantes

Passez en revue les parcours de gouvernance exploitables.

> [!div class="nextstepaction"]
> [Parcours de gouvernance actionnables](./journeys/overview.md)
