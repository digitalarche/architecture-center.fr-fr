---
title: 'Framework d’adoption du cloud : Comment la politique informatique de l’entreprise peut devenir prête pour le cloud ?'
description: Explication du concept de stratégie d’entreprise par rapport à la gouvernance cloud
author: BrianBlanchard
ms.date: 2/8/2019
ms.openlocfilehash: e15f4ef4e6fa9a64ea2bb78cd1df6c5867528383
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901177"
---
<!-- markdownlint-disable MD026 -->

# <a name="caf-how-can-corporate-it-policy-become-cloud-ready"></a>Framework d’adoption du cloud : Comment la politique informatique de l’entreprise peut devenir prête pour le cloud ?

La gouvernance cloud est le fruit d’un travail d’adoption sur le long terme, car une vraie transformation durable ne se fait pas du jour au lendemain. Vous avez peu de chances d’obtenir des résultats satisfaisants si vous essayez de fournir une gouvernance cloud complète avant de prendre en charge les modifications clés de la stratégie d’entreprise, et si vous utilisez une méthode rapide et agressive. Nous vous recommandons plutôt d’opter pour une approche incrémentielle.

Ce qui est différent dans notre framework d’adoption du cloud, c’est le cycle d’achat et l’impact de ce dernier sur l’authenticité de la transformation. Comme il n’existe pas d’obligation à acquérir des dépenses d’investissement (CapEx) importantes, les ingénieurs peuvent commencer plus tôt l’expérimentation et l’adoption. Dans la plupart des cultures d’entreprise, le fait que la barrière CapEx ne soit plus un obstacle à l’adoption peut entraîner des boucles de rétroaction plus serrées, une croissance organique et une exécution incrémentielle.

Le passage au cloud implique de changer la gouvernance. Dans beaucoup d’organisations, la transformation de la stratégie d’entreprise permet d’améliorer la gouvernance et d’augmenter les taux de conformité. Ces optimisations sont possibles grâce aux changements incrémentiels de stratégie et à l’application automatique de ces derniers. Pour ce faire, vous disposez de capacités nouvellement définies, que vous pouvez configurer avec votre fournisseur de services cloud.

Cet article décrit les principales activités qui peuvent vous aider à façonner vos stratégies d’entreprise, afin d’obtenir un modèle de gouvernance étendu.

## <a name="define-corporate-policy-to-mature-cloud-governance"></a>Définition de la stratégie d’entreprise pour faire évoluer la gouvernance cloud

Dans la gouvernance traditionnelle et incrémentielle, la stratégie d’entreprise crée la définition pratique de la gouvernance. La plupart des actions de gouvernance informatique cherchent à implémenter la technologie afin de superviser, appliquer, exécuter et automatiser ces stratégies d’entreprise. La gouvernance cloud s’appuie sur des concepts similaires.

![Gouvernance d’entreprise et disciplines de gouvernance](../../_images/operational-transformation-govern.png)

*Figure 1 : Gouvernance d’entreprise et disciplines de gouvernance.*

L’image ci-dessus montre les interactions qu’il existe entre les risques métier, la stratégie, la conformité, la supervision et l’application afin de créer une stratégie de gouvernance. Ensuite, vous sont présentées les cinq disciplines de la gouvernance cloud pour élaborer votre stratégie.

## <a name="review-existing-policies"></a>Révision des stratégies existantes

Dans l’image ci-dessus, la stratégie de gouvernance (risque, stratégie et conformité, supervision et application) commence par l’identification des risques métier. La première étape pour créer une stratégie de gouvernance cloud durable consiste à comprendre en quoi les [risques métier](understanding-business-risk.md) changent dans le cloud. En travaillant avec vos unités commerciales afin de [jauger précisément la tolérance de l’entreprise aux risques](risk-tolerance.md), vous saurez quel niveau de risques doit être atténué. Votre compréhension des nouveaux risques et de la tolérance acceptable peut vous permettre d’effectuer une [révision des stratégies existantes](what-is-a-cloud-policy-review.md), et de déterminer le niveau de gouvernance requis pour votre organisation.

> [!TIP]
> Si votre organisation est soumise à une conformité tierce, un des plus gros risques métier à prendre en compte est l’adhésion à la [conformité réglementaire](what-is-regulatory-compliance.md). Bien souvent, ce risque ne peut pas être atténué et requiert une adhésion stricte. Veillez à comprendre les exigences de conformité tierces avant de commencer une révision de la stratégie.

## <a name="an-incremental-approach-to-cloud-governance"></a>Approche incrémentielle de la gouvernance cloud

Une approche incrémentielle de la gouvernance cloud part du principe qu’il est inacceptable de dépasser le [seuil de tolérance aux risques de l’entreprise](risk-tolerance.md). Pour elle, le rôle de la gouvernance est d’accélérer les changements de l’entreprise, d’aider les ingénieurs à comprendre les consignes de l’architecture, et de s’assurer que les [risques métier](understanding-business-risk.md) sont régulièrement communiqués et atténués. À l’inverse, le rôle traditionnel de la gouvernance peut devenir un obstacle à l’adoption par les ingénieurs ou par l’entreprise en général.

Avec l’approche incrémentielle de la gouvernance cloud, il existe parfois une opposition naturelle entre les équipes qui développent de nouvelles solutions métier et les équipes qui protègent l’entreprise des risques. Toutefois, dans ce modèle, les deux équipes peuvent devenir des pairs travaillant par incréments ou sprints. De cette manière, l’équipe de gouvernance cloud et les équipes d’adoption du cloud peuvent travailler ensemble à l’exposition, l’évaluation et l’atténuation des risques métier. Grâce à cela, il est possible de réduire naturellement l’opposition entre les équipes et établir la collaboration.

## <a name="minimum-viable-product-mvp-for-policy"></a>Produit minimum viable (MVP) pour la stratégie

Pour faire émerger le partenariat entre l’équipe de gouvernance cloud et les équipes d’adoption du cloud, la première étape est un accord concernant le MVP de stratégie. Votre MVP de gouvernance cloud doit reconnaître que les risques métier sont faibles au début, mais vont probablement croître à mesure que votre organisation adopte plus de services cloud au fil du temps.

Par exemple :  Pour une entreprise qui déploie 5 machines virtuelles ne contenant pas de données HBI (High Business Impact, ou Impact élevé pour l’entreprise), le risque métier est faible. Après plusieurs incréments, lorsque l’entreprise a atteint 1 000 machines virtuelles et qu’elle commence à déplacer les données HBI, le risque métier augmente.

Le MVP de stratégie vise à définir une base requise pour les stratégies. Cette base requise permet de déployer les premières « x » machines virtuelles ou applications. Le chiffre « x » correspond à une quantité peu élevée mais à fort impact d’unités en cours d’adoption. Cet ensemble stratégique requiert peu de contraintes, mais contient les aspects fondamentaux permettant de passer rapidement d’un incrément de travail au prochain. Grâce au développement incrémentiel de la stratégie, la gouvernance se développe au fil du temps. Grâce à des changements lents et subtils, le MVP de stratégie atteint la parité des fonctionnalités avec les résultats de l’exercice de révision de la stratégie.

## <a name="incremental-policy-growth"></a>Croissance incrémentielle de la stratégie

La croissance incrémentielle de la stratégie est le principal mécanisme permettant de faire évoluer la stratégie et la gouvernance cloud au fil du temps. Elle est également essentielle à l’adoption d’un modèle incrémentiel pour la gouvernance. Pour que ce modèle fonctionne bien, l’équipe de gouvernance doit s’engager à allouer du temps en continu à chaque sprint, de manière à évaluer et implémenter les disciplines de gouvernance changeantes.

**Exigences de temps pour les sprints :** Au début d’une itération, chaque équipe d’adoption du cloud dresse une liste des ressources à migrer ou adopter dans l’incrément actuel. L’équipe de gouvernance cloud est supposée laisser le temps nécessaire aux activités suivantes : examen de la liste, validation de la classification des données pour les ressources, évaluation des nouveaux risques associés à chaque ressource, mise à jour des consignes de l’architecture, et explication des changements à l’équipe. Ces activités nécessitent en général entre 10 et 30 heures par sprint. Pour ce niveau d’implication, il est également préférable d’avoir au moins un employé dédié qui gère la gouvernance au sein du processus plus large d’adoption du cloud.

**Exigences de temps pour la mise en production :** Au début de chaque mise en production, les équipes d’adoption du cloud et l’équipe de stratégie cloud doivent placer en priorité une liste d’applications ou de charges de travail à migrer dans l’itération actuelle, ainsi que des changements de l’entreprise. Ces points de données permettent à l’équipe de gouvernance cloud de comprendre de manière précoce les nouveaux risques métier. Grâce à cela, elle a le temps de s’aligner sur l’entreprise et de jauger la tolérance aux risques de l’entreprise.