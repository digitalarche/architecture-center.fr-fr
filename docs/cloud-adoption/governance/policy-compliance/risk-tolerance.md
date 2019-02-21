---
title: "Framework d'adoption du cloud : Quels sont les risques commerciaux associés à une transformation cloud ?"
description: Explication des risques commerciaux associés à une transformation cloud
author: BrianBlanchard
ms.date: 10/10/2018
ms.openlocfilehash: bfd91da42d20a85004debc6b767a1482ba3e158d
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901125"
---
# <a name="evaluating-risk-tolerance"></a>Évaluer la tolérance au risque

Chaque décision commerciale engendre de nouveaux risques. Tout investissement s'accompagne d'un risque de perte. Les nouveaux produits ou services engendrent des risques de défaillance du marché. Toute modification apportée aux produits ou services actuels peut se traduire par une réduction de la part de marché. La transformation cloud n'apporte aucune solution magique aux risques encourus quotidiennement par les entreprises. Au contraire, les solutions connectées (cloud ou locales) introduisent de nouveaux risques. Le déploiement de ressources dans une installation connectée au réseau accroît également les menaces potentielles en exposant les points faibles de la sécurité à une communauté mondiale beaucoup plus large. Heureusement, les fournisseurs de services cloud sont conscients de l'évolution et de l'augmentation de ces risques. Ils investissent massivement afin d'atténuer les risques auxquels leurs clients sont exposés.

Cet article n'est pas consacré aux risques liés au cloud. Il se concentre sur les risques commerciaux associés aux différentes formes de transformation cloud. Plus bas, nous nous pencherons sur les moyens dont disposent les entreprises pour évaluer la tolérance au risque.

<!-- markdownlint-disable MD026 -->

## <a name="what-business-risks-are-associated-with-a-cloud-transformation"></a>Quels sont les risques commerciaux associés à une transformation cloud ?

Malheureusement, les véritables risques commerciaux reposent sur les tenants et les aboutissants des transformations spécifiques. Mais heureusement, un certain nombre de risques très courants peuvent être utilisés lorsqu'une conversation s'engage pour identifier des risques spécifiques et personnalisés.

** Avant de lire ce qui suit, sachez que chacun de ces risques peut être corrigé. Le but de cet article est d'informer et de préparer les lecteurs afin de leur permettre de prendre des mesures efficaces d'atténuation les risques. **

* Risque lié à la protection des données : en matière de transformation, la protection des données représente indiscutablement le risque numéro un. À l'ère du numérique, les données constituent le nouveau pétrole dans le monde des affaires. Elles alimentent l'économie, chauffent le bureau et ravissent les clients. Mais en cas de fuite, le résultat est tout aussi dévastateur. Tout changement apporté à la façon dont les données sont stockées, traitées ou utilisées engendre un risque. Les transformations cloud entraînent de nombreux changements en matière de gestion des données. Le risque ne doit donc pas être pris à la légère. Les [règles de sécurité de base](../security-baseline/overview.md), la [classification des données](./what-is-data-classification.md) et la [rationalisation incrémentielle](../../digital-estate/rationalize.md#incremental-rationalization) peuvent toutes contribuer à atténuer ce risque.

* Risque lié aux opérations et à l'expérience client : les opérations commerciales et l'expérience client reposent largement sur les opérations techniques. Les transformations cloud entraînent des modifications au niveau des opérations techniques (TechOps). Dans certaines organisations, ces modifications sont mineures et faciles à ajuster. Dans d'autres, elles peuvent nécessiter un réoutillage, une requalification ou de nouvelles approches de prise en charge. Plus les modifications sont importantes, plus l'impact potentiel sur les opérations commerciales et l'expérience client le sont également. L'atténuation de ce risque viendra de la participation de l'entreprise à la planification de la transformation. Les sections Planification de la mise en production et Première charge de travail de l'article [Rationalisation incrémentielle](../../digital-estate/rationalize.md#incremental-rationalization) expliquent comment choisir les charges de travail pour les projets de transformation. Le rôle de l'entreprise dans le cadre de cette activité consiste à communiquer le risque découlant des modifications apportées aux charges de travail classées par ordre de priorité. Le fait d'aider l'équipe informatique à choisir des charges de travail à impact réduit sur les opérations diminuera le risque global.

* Risque lié aux coûts : les modèles de coûts changent dans le cloud. Ce changement peut engendrer des risques associés à des dépassements de coûts ou à une augmentation du COGS (Cost of Goods Sold - coût des marchandises vendues), en particulier des dépenses opérationnelles directement imputables. Lorsque l'équipe commerciale travaille en étroite collaboration avec l'équipe informatique, une véritable transparence peut être atteinte en matière de coûts et d'utilisation des services par les différents programmes, projets, unités opérationnelles, etc. [Cost Management] fournit des exemples de partenariats entre les équipes commerciales et informatiques sur ce sujet.

Voici quelques-uns des risques les plus fréquemment mentionnés par les clients. L'équipe de gouvernance cloud et les équipes d'adoption du cloud peuvent commencer à développer un profil de risque à mesure que les charges de travail sont migrées et préparées pour la mise en production. Préparez-vous à devoir discuter pour définir, affiner et atténuer les risques en fonction des résultats opérationnels et des efforts de transformation souhaités.

## <a name="understanding-risk-tolerance"></a>Appréhender la tolérance au risque

L'identification des risques est un processus relativement direct. Les risques sont plutôt courants dans tous les secteurs. En revanche, la tolérance au risque est extrêmement variable. C'est là que les conversations commerciales et informatiques ont tendance à s'enliser. Chacune des parties représentées parle une langue différente. Les comparaisons et les questions suivantes sont conçues pour engager la conversation afin d'aider chaque partie à mieux appréhender et évaluer la tolérance au risque.

## <a name="simple-use-case-for-comparison"></a>Cas d'usage simple à des fins de comparaison

Pour appréhender la tolérance au risque, examinons des données client. Si une entreprise d'un secteur particulier publie des données client sur un serveur non sécurisé, le risque de compromission ou de vol de ces données est à peu près identique. En revanche, la nature des données et la tolérance des entreprises vis-à-vis de ce risque varient énormément.

* Aux États-Unis, les entreprises du secteur de la santé et de la finance sont soumises à des obligations de conformité strictes imposées par des tiers. On part du principe que les informations d'identification personnelle (PII) ou les données médicales sont extrêmement confidentielles. Les conséquences pour les entreprises de ce type sont graves si elles sont impliquées dans le scénario de risque ci-dessus. Leur tolérance est extrêmement faible. Toutes les données client publiées à l'intérieur ou à l'extérieur du réseau doivent être régies par ces stratégies de conformité tierces.
* Une société de jeux dont les données client se limitent à un nom d'utilisateur, à des temps de jeu et à des scores est moins exposée à des conséquences graves si elle adopte le comportement à risque décrit ci-dessus. Si un risque pèse sur des données non sécurisées, l'impact de ce risque est faible. Par conséquent, dans ce cas, la tolérance au risque est élevée.
* La tolérance d'une PME fournissant des services de nettoyage de tapis à des milliers de clients se situerait entre ces deux extrémités. Les données client peuvent être plus robustes et contenir des informations comme l'adresse ou le numéro de téléphone. Dans la mesure où ces informations peuvent être considérées comme des informations d'identification personnelle, elles doivent être protégées. Cela dit, il se peut qu'aucune obligation de gouvernance spécifique n'impose de sécuriser les données. D'un point de vue informatique, la réponse est simple : sécuriser les données. D'un point de vue commercial, ce n'est peut-être pas aussi simple. L'équipe commerciale aurait besoin de détails supplémentaires avant de pouvoir déterminer un niveau de tolérance pour ce risque.

La section suivante contient des exemples de questions qui peuvent aider l'entreprise à déterminer un niveau de tolérance au risque pour le cas d'usage ci-dessus, entre autres.

## <a name="risk-tolerance-questions"></a>Questions relatives à la tolérance au risque

Cette section présente trois catégories de questions : Impact sur les pertes, Probabilité de perte et Coûts d'atténuation. Lorsque les équipes commerciales et informatiques s'associent pour réfléchir à chacun de ces thèmes, les décisions relatives à l'atténuation du risque et à la tolérance globale à un risque particulier sont plus faciles à prendre.

**Impact sur les pertes** : Questions visant à déterminer l'impact d'un risque. Il est souvent difficile (voire impossible) de répondre à ce type de questions. Il est préférable de quantifier l'impact, mais parfois la conversation seule suffit à identifier la tolérance. Des fourchettes sont également acceptables, surtout si elles incluent les hypothèses qui les ont définies.

* Ce risque enfreint-il les obligations de conformité de tiers ?
* Ce risque enfreint-il les stratégies internes de l'entreprise ?
* Ce risque pourrait-il coûter des clients ou des parts de marché ? Si oui, cet impact peut-il être quantifié ?
* Ce risque pourrait-il nuire à l'expérience client ? Cette expérience peut-elle avoir un impact sur les ventes ou sur les recettes ?
* Ce risque pourrait-il engager une nouvelle responsabilité juridique ? Si oui, existe-t-il un précédent en matière d'attribution de dommages-intérêts dans ce genre de cas ?
* Ce risque pourrait-il entraîner une interruption d'activité pour l'entreprise ? Si oui, pendant combien de temps l'activité serait-elle interrompue ?
* Ce risque pourrait-il ralentir les activités de l'entreprise ? Si oui, dans quelle mesure et combien de temps ?
* À ce stade de la transformation, s'agit-il d'un risque ponctuel ou est-il susceptible de se répéter ?
* Le risque augmente-t-il ou diminue-t-il au fil de la progression de la transformation ?
* Le risque est-il susceptible d'augmenter ou de diminuer avec le temps ?
* Le risque est-il lié à une échéance ? Le risque s'estompera-t-il ou s'aggravera-t-il si rien n'est fait ?

Ces questions de base soulèveront beaucoup d'autres interrogations. Après un dialogue sain, il est conseillé de consigner les risques pertinents et, si possible, de les quantifier.

**Coûts d'atténuation** : Questions visant à déterminer le coût de l'atténuation ou de l'élimination du risque. Ces questions peuvent être assez directes, surtout lorsqu'elles sont représentées dans une fourchette.

* Existe-t-il une solution claire ? Combien coûte-t-elle ?
* Des solutions sont-elles disponibles pour résoudre ou atténuer ce risque ? Quelle est la fourchette de coûts de ces solutions ?
* Comment l'entreprise doit-elle s'y prendre pour choisir la solution la plus efficace et la plus claire ?
* Comment l'entreprise doit-elle s'y prendre pour valider les coûts ?
* Quels autres avantages peuvent découler de la solution d'atténuation de ce risque ?

Ces questions simplifient les solutions techniques nécessaires à l'atténuation des risques. Toutefois, elles communiquent ces solutions de manière à ce que l'entreprise puisse les intégrer rapidement dans un processus décisionnel.

**Probabilité de perte** : Questions visant à déterminer la probabilité que le risque devienne une réalité. Il s'agit du domaine le plus difficile à quantifier. Nous suggérons plutôt à l'équipe de gouvernance cloud de créer des catégories pour la communication de la probabilité, sur la base de données justificatives. Les questions suivantes peuvent vous aider à créer des catégories pertinentes pour l'équipe.

* Des recherches ont-elles été menées sur la probabilité que ce risque se réalise ?
* Le fournisseur peut-il fournir des références ou des statistiques sur la probabilité de l'impact ?
* D'autres entreprises du secteur ou de la branche concernés ont-elles été touchées par ce risque ?
* D'autres entreprises en général ont-elles été touchées par ce risque ?
* Ce risque est-il propre à quelque chose que cette entreprise a mal fait ?

Après avoir répondu à ces questions et à d'autres questions posées par l'équipe de gouvernance cloud, des regroupements de probabilités émergeront probablement. Voici quelques exemples de regroupements pour vous aider à démarrer :

* Aucune indication : les recherches n'ont pas été suffisantes pour déterminer la probabilité.
* Risque faible : en l'état actuel des recherches, il est peu probable que le risque se concrétise.
* Risque futur : actuellement, le risque est faible. Toutefois, la poursuite de l'adoption déclencherait une nouvelle analyse.
* Risque moyen : il est probable que le risque ait un impact sur l'entreprise.
* Risque élevé : avec le temps, il est de moins en moins probable que ce risque épargne l'entreprise.
* Diminution du risque : le risque est moyen à élevé. Toutefois, les mesures prises par les équipes informatiques ou commerciales réduisent la probabilité d'un impact.

**Détermination de la tolérance :**

Les trois catégories de questions ci-dessus doivent fournir suffisamment de données pour déterminer les tolérances initiales. Si le risque et la probabilité sont faibles, et que le coût d'atténuation est élevé, l'entreprise sera peu encline à investir dans des mesures correctives. Si le risque et la probabilité sont élevés, l'entreprise envisagera probablement d'investir, à condition que les coûts ne dépassent pas les risques potentiels.

## <a name="next-steps"></a>Étapes suivantes

Ce type de conversation peut aider les équipes commerciales et informatiques à évaluer plus efficacement la tolérance. Ces conversations peuvent être utilisées lors de la création des stratégies MVP et lors des révisions incrémentielles des stratégies.

> [!div class="nextstepaction"]
> [Définir la stratégie de l'entreprise](./define-policy.md)
