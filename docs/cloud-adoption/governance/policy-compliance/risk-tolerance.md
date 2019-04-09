---
title: "Framework d'adoption du cloud : Quels sont les risques commerciaux associés à une transformation cloud ?"
description: Explication des risques commerciaux associés à une transformation cloud
author: BrianBlanchard
ms.date: 10/10/2018
ms.openlocfilehash: 774a6703b353a3c670b3764505185aecb794784f
ms.sourcegitcommit: 0a8a60d782facc294f7f78ec0e9033e3ee16bf4a
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/08/2019
ms.locfileid: "59068937"
---
# <a name="evaluating-risk-tolerance"></a>Évaluer la tolérance au risque

Chaque décision commerciale engendre de nouveaux risques. Tout investissement s'accompagne d'un risque de perte. Les nouveaux produits ou services engendrent des risques de défaillance du marché. Toute modification apportée aux produits ou services actuels peut se traduire par une réduction de la part de marché. La transformation cloud n'apporte aucune solution magique aux risques encourus quotidiennement par les entreprises. Au contraire, les solutions connectées (cloud ou locales) introduisent de nouveaux risques. Le déploiement de ressources dans une installation connectée au réseau accroît également les menaces potentielles en exposant les points faibles de la sécurité à une communauté mondiale beaucoup plus large. Heureusement, les fournisseurs de services cloud sont conscients de l'évolution et de l'augmentation de ces risques. Ils investissent de manière importante pour réduire et gérer ces risques de la part de leurs clients.

Cet article n'est pas consacré aux risques liés au cloud. Il se concentre sur les risques commerciaux associés aux différentes formes de transformation cloud. Plus bas, nous nous pencherons sur les moyens dont disposent les entreprises pour évaluer la tolérance au risque.

<!-- markdownlint-disable MD026 -->

## <a name="what-business-risks-are-associated-with-a-cloud-transformation"></a>Quels sont les risques commerciaux associés à une transformation cloud ?

Les risques commerciaux true doit reposer sur la page qu’et derrière des transformations spécifiques. Mais heureusement, un certain nombre de risques très courants peuvent être utilisés lorsqu'une conversation s'engage pour identifier des risques spécifiques et personnalisés.

** Avant de lire les rubriques suivantes, prenez garde que chacun de ces risques peut être géré. L’objectif de cet article consiste à informer et préparer les lecteurs, afin que la discussion sur la gestion des risques peut être plus efficace. **

- **Violation des données** : en matière de transformation, la protection des données représente indiscutablement le risque numéro un. Les fuites de données peut causer des dommages importants pour votre entreprise, ce qui conduit à une perte de clients, diminution de l’entreprise, ou même de poursuites. Tout changement apporté à la façon dont les données sont stockées, traitées ou utilisées engendre un risque. Les transformations cloud entraînent de nombreux changements en matière de gestion des données. Le risque ne doit donc pas être pris à la légère. [Sécurité de base](../security-baseline/overview.md), [Classification des données](./what-is-data-classification.md), et [rationalisation incrémentielle](../../digital-estate/rationalize.md#incremental-rationalization) chacun peut aider à gérer ce risque.

- **Interruption de service**: Opérations de l’entreprise et des expériences client dépendent fortement opérations techniques. Les transformations cloud entraînent des modifications au niveau des opérations techniques (TechOps). Dans certaines organisations, que la modification est petite et facilement ajustée. Dans d'autres, elles peuvent nécessiter un réoutillage, une requalification ou de nouvelles approches de prise en charge. La modification de plus, plus l’impact potentiel sur les opérations de l’entreprise et l’expérience client. La gestion de ce risque nécessite l’intervention de l’entreprise dans la planification de la transformation. Les sections Planification de la mise en production et Première charge de travail de l'article [Rationalisation incrémentielle](../../digital-estate/rationalize.md#incremental-rationalization) expliquent comment choisir les charges de travail pour les projets de transformation. Rôle de l’entreprise dans la mesure où l’activité doit communiquer le risque d’opérations de l’entreprise de la modification par ordre de priorité des charges de travail. Aider informatique choisissez les charges de travail qui ont un impact plus faible sur les opérations réduira le risque global.

- **Contrôle du budget** : les modèles de coûts changent dans le cloud. Ce changement peut engendrer des risques associés à des dépassements de coûts ou à une augmentation du COGS (Cost of Goods Sold - coût des marchandises vendues), en particulier des dépenses opérationnelles directement imputables. Lorsque l’entreprise travaille en étroite collaboration avec l’informatique, il est possible de créer la transparence des coûts et les services consommés par les différentes divisions, programmes, projets, etc.... [Gestion des coûts](../cost-management/overview.md) fournit des exemples d’entreprise et le service informatique peut partenaire sur cette rubrique.

Voici quelques-uns des risques les plus fréquemment mentionnés par les clients. L'équipe de gouvernance cloud et les équipes d'adoption du cloud peuvent commencer à développer un profil de risque à mesure que les charges de travail sont migrées et préparées pour la mise en production. Préparez-vous pour les conversations définir, d’affiner et de gérer les risques selon les résultats de votre entreprise souhaité et les efforts de transformation.

## <a name="understanding-risk-tolerance"></a>Appréhender la tolérance au risque

L'identification des risques est un processus relativement direct. Risques liés à l’informatique sont généralement standard pour différents secteurs. Toutefois, la tolérance de panne pour ces risques est spécifique à chaque organisation. C'est là que les conversations commerciales et informatiques ont tendance à s'enliser. Chacune des parties représentées parle une langue différente. Les comparaisons et les questions suivantes sont conçues pour engager la conversation afin d'aider chaque partie à mieux appréhender et évaluer la tolérance au risque.

## <a name="simple-use-case-for-comparison"></a>Cas d'usage simple à des fins de comparaison

Pour aider à comprendre la tolérance au risque, nous allons examiner les données client. Si une entreprise dans n’importe quel secteur publie les données du client sur un serveur non sécurisé, le risque technique de ces données compromises ou de vol est à peu près le même. Cependant, la tolérance de panne d’une société de ce risque varie grandement selon la valeur de caractère et le potentiel des données.

- Aux États-Unis, les entreprises du secteur de la santé et de la finance sont soumises à des obligations de conformité strictes imposées par des tiers. On part du principe que les informations d'identification personnelle (PII) ou les données médicales sont extrêmement confidentielles. Les conséquences pour les entreprises de ce type sont graves si elles sont impliquées dans le scénario de risque ci-dessus. Leur tolérance est extrêmement faible. Toutes les données client publiées à l'intérieur ou à l'extérieur du réseau doivent être régies par ces stratégies de conformité tierces.
- Une société dont les données client sont limitées à un nom d’utilisateur, durée d’exécution et les meilleurs scores n’est pas aussi susceptible de subir des conséquences significatives, si elles s’engager dans les comportements à risques ci-dessus. Bien que toutes les données non sécurisées sont menacée, l’impact de ce risque est faible. Par conséquent, la tolérance de risque dans ce cas est élevée.
- Une entreprise de taille moyenne qui fournit des services à des milliers de clients de nettoyage de tapis comprises entre ces extrêmes de tolérance de panne de deux. Là, les données client peuvent être plus robustes, contenant des détails tels que l’adresse ou numéro de téléphone. Les deux pourraient être considérées comme des informations d’identification personnelle et doivent être protégées. Cela dit, il se peut qu'aucune obligation de gouvernance spécifique n'impose de sécuriser les données. D'un point de vue informatique, la réponse est simple : sécuriser les données. D'un point de vue commercial, ce n'est peut-être pas aussi simple. L'équipe commerciale aurait besoin de détails supplémentaires avant de pouvoir déterminer un niveau de tolérance pour ce risque.

La section suivante partage quelques exemples de questions qui peuvent aider l’entreprise de déterminer un niveau de tolérance au risque pour le cas d’usage ou d’autres utilisateurs.

## <a name="risk-tolerance-questions"></a>Questions relatives à la tolérance au risque

Cette section répertorie les questions dans trois catégories de fou de conversation : impact de la perte, la probabilité de perte et les coûts de mise à jour. Lorsque business et partenaire informatique pour traiter chacun de ces domaines, la décision d’étendre l’effort sur la gestion des risques et la tolérance globale à un risque particulier peuvent être déterminées facilement.

**Impact de la perte**: Questions visant à déterminer l'impact d'un risque. Il est souvent difficile (voire impossible) de répondre à ce type de questions. Il est préférable de quantifier l'impact, mais parfois la conversation seule suffit à identifier la tolérance. Des fourchettes sont également acceptables, surtout si elles incluent les hypothèses qui les ont définies.

- Ce risque enfreint-il les obligations de conformité de tiers ?
- Ce risque enfreint-il les stratégies internes de l'entreprise ?
- Ce risque pourrait-il coûter des clients ou des parts de marché ? Si, par conséquent, ces coûts peuvent être évalués ?
- Ce risque pourrait-il nuire à l'expérience client ? Ces expériences sont susceptibles d’affecter les ventes ou la réalisation de chiffre d’affaires ?
- Ce risque pourrait-il engager une nouvelle responsabilité juridique ? Si oui, existe-t-il un précédent en matière d'attribution de dommages-intérêts dans ce genre de cas ?
- Ce risque pourrait-il entraîner une interruption d'activité pour l'entreprise ? Si oui, pendant combien de temps l'activité serait-elle interrompue ?
- Ce risque pourrait-il ralentir les activités de l'entreprise ? Dans ce cas, quelle vitesse et la durée ?
- À ce stade de la transformation, s'agit-il d'un risque ponctuel ou est-il susceptible de se répéter ?
- Le risque augmente-t-il ou diminue-t-il au fil de la progression de la transformation ?
- Le risque est-il susceptible d'augmenter ou de diminuer avec le temps ?
- Le risque est-il lié à une échéance ? Le risque s'estompera-t-il ou s'aggravera-t-il si rien n'est fait ?

Ces questions de base soulèveront beaucoup d'autres interrogations. Après un dialogue sain, il est conseillé de consigner les risques pertinents et, si possible, de les quantifier.

**Les coûts de mise à jour des risques**: Questions pour déterminer le coût de la suppression ou sinon en minimisant les risques. Ces questions peuvent être assez directes, surtout lorsqu'elles sont représentées dans une fourchette.

- Existe-t-il une solution claire ? Combien coûte-t-elle ?
- Existe-t-il des options pour prévenir ou à réduire ce risque ? Quelle est la fourchette de coûts de ces solutions ?
- Comment l'entreprise doit-elle s'y prendre pour choisir la solution la plus efficace et la plus claire ?
- Comment l'entreprise doit-elle s'y prendre pour valider les coûts ?
- Les autres avantages peuvent provenir de la solution qui supprimerait ce risque ?

Ces questions sur simplifient les solutions techniques nécessaires pour gérer ou supprimer les risques. Toutefois, elles communiquent ces solutions de manière à ce que l'entreprise puisse les intégrer rapidement dans un processus décisionnel.

**Probabilité de perte** : Questions visant à déterminer la probabilité que le risque devienne une réalité. Il s'agit du domaine le plus difficile à quantifier. Nous suggérons plutôt à l'équipe de gouvernance cloud de créer des catégories pour la communication de la probabilité, sur la base de données justificatives. Les questions suivantes peuvent vous aider à créer des catégories pertinentes pour l'équipe.

- Des recherches ont-elles été menées sur la probabilité que ce risque se réalise ?
- Le fournisseur peut-il fournir des références ou des statistiques sur la probabilité de l'impact ?
- D'autres entreprises du secteur ou de la branche concernés ont-elles été touchées par ce risque ?
- D'autres entreprises en général ont-elles été touchées par ce risque ?
- Ce risque est-il propre à quelque chose que cette entreprise a mal fait ?

Après avoir répondu à ces questions, ainsi que des questions comme déterminé par l’équipe de gouvernance du Cloud, les regroupements de probabilité seront probablement émergent. Voici quelques exemples de regroupements pour vous aider à démarrer :

- Aucune indication : N’est pas suffisant research a été effectuée pour déterminer la probabilité.
- Risque faible : Recherche en cours suggère que le risque n’est pas susceptible d’être mis en œuvre.
- Risque futur : actuellement, le risque est faible. Toutefois, l’adoption continue déclenche une nouvelle analyse.
- Risque moyen : Il est probable que le risque aura un impact sur l’entreprise.
- Risque élevé : Au fil du temps, il est d’augmenter moins probable que l’entreprise, élimine l’impact de ce risque.
- Diminution du risque : le risque est moyen à élevé. Toutefois, les mesures prises par les équipes informatiques ou commerciales réduisent la probabilité d'un impact.

**Détermination de la tolérance de panne :**

Les trois catégories de questions ci-dessus doivent fournir suffisamment de données pour déterminer les tolérances initiales. Lorsque le risque et probabilité sont faibles, et les coûts de mise à jour risque sont élevés, l’entreprise est peu probable d’investir dans la mise à jour. Lorsque risque et probabilité sont élevés, l’entreprise est susceptible de prendre en compte un investissement, tant les coûts ne dépassent pas les risques potentiels.

## <a name="next-steps"></a>Étapes suivantes

Ce type de conversation peut aider les équipes commerciales et informatiques à évaluer plus efficacement la tolérance. Ces conversations peuvent être utilisées lors de la création des stratégies MVP et lors des révisions incrémentielles des stratégies.

> [!div class="nextstepaction"]
> [Définir la stratégie de l’entreprise](./define-policy.md)
