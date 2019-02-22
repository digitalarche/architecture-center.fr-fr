---
title: 'Framework d’adoption du cloud : Qu’est-ce qu’une révision de stratégie cloud ?'
description: Qu’est-ce qu’une révision de stratégie cloud ?
author: BrianBlanchard
ms.date: 2/8/2019
ms.openlocfilehash: b879f261e16ffc72180417e2e0f533e2eaa435ba
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900993"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-is-a-cloud-policy-review"></a>Qu’est-ce qu’une révision de politique cloud ?

Une **révision de stratégie cloud** constitue la première étape vers la [maturité de la gouvernance](../overview.md) dans le cloud. L’objectif de ce processus est de moderniser les stratégies informatiques existantes de l’entreprise. À la fin de ce processus, les stratégies mises à jour fournissent un niveau équivalent de réduction des risques pour les ressources cloud. Cet article explique le processus de révision de stratégie de cloud et son importance.

## <a name="why-perform-a-cloud-policy-review"></a>Pourquoi procéder à une révision de politique cloud ?

La plupart des entreprises gèrent les services informatiques via l’exécution de processus alignés avec des stratégies directrices. Dans les petites entreprises, ces stratégies peuvent sembler anecdotiques et les processus sont vaguement définis. À mesure qu’une entreprise grandit, les stratégies et les processus ont tendance à être documentés de façon plus précise et exécutés de manière plus cohérente.

Tandis que les stratégies informatiques d’entreprise gagnent en maturité, les dépendances aux décisions techniques prises par le passé ont tendance à s’infiltrer dans les stratégies directrices. Par exemple, il est courant de voir que les processus de récupération d’urgence incluent une stratégie qui impose des sauvegardes sur bande hors site. Cette inclusion suppose une dépendance à un type de technologie (sauvegardes sur bande) qui n’est peut-être plus la solution la plus pertinente.

Les transformations cloud créent un point d’inflexion naturel pour réexaminer les décisions en matière de stratégie prises par le passé. Les fonctionnalités techniques et les processus par défaut changent considérablement dans le cloud, comme les risques inhérents. En reprenant l’exemple précédent, le choix d’une stratégie de sauvegarde sur bande est motivé par le risque d’avoir un point de défaillance unique, où les données sont conservées dans un seul emplacement, et par le besoin métier de réduire le profil de risque en atténuant ce risque. Dans un déploiement cloud, plusieurs options sont proposées pour atténuer le risque, avec des objectifs de temps de récupération bien moins élevés. Exemple :

- Une solution cloud peut fournir des fonctionnalités de géoréplication de la base de données SQL Azure
- Une solution hybride peut tirer parti d’Azure Site Recovery pour répliquer une charge de travail IaaS dans plusieurs centres de données

Lors d’une transformation cloud, les stratégies régissent souvent les nombreux outils, services et processus disponibles pour les équipes d’adoption du cloud. Si ces stratégies reposent sur des technologies héritées, elles peuvent entraver les efforts de l’équipe pour mettre en place les changements. Dans le pire des cas, ce sont des stratégies importantes qui sont complètement ignorées par l’équipe de migration pour appliquer des solutions de contournement. Tout cela n’est pas acceptable.

## <a name="the-cloud-policy-review-process"></a>Le processus de révision de stratégie cloud

Les révisions de stratégie cloud alignent les stratégies de sécurité et de gouvernance informatiques existantes avec les [cinq disciplines de la gouvernance cloud](../overview.md) : [Gestion des coûts](../cost-management/overview.md), [Base de référence de sécurité](../security-baseline/overview.md), [Base de référence des identités](../identity-baseline/overview.md), [Cohérence des ressources](../resource-consistency/overview.md) et [Accélération du déploiement](../deployment-acceleration/overview.md).

Pour chacune de ces disciplines, le processus de révision suit les étapes ci-dessous :

1. Révision des stratégies locales existantes en lien avec la discipline concernée pour rechercher deux points de données clés : les dépendances héritées et les risques métier identifiés.
2. Évaluation de chacun des risques métier grâce à une question simple : « Le risque existe-t-il toujours dans un modèle cloud ? »
3. Si le risque existe toujours, réécriture de la stratégie en documentant l’atténuation nécessaire plutôt que la solution technique.
4. Révision de la stratégie mise à jour avec les équipes d’adoption du cloud pour comprendre les solutions potentielles associées à l’atténuation nécessaire.

## <a name="example-of-a-policy-review-for-a-legacy-policy"></a>Exemple d’une révision de stratégie héritée

Pour illustrer le processus de révision, reprenons la stratégie de sauvegarde sur bande de la section précédente :

- Une stratégie d’entreprise exige que des sauvegardes sur bande hors site soient effectuées pour tous les systèmes de production. Dans cette stratégie, vous pouvez voir deux points de données intéressants :
  - Une dépendance héritée sur une solution de sauvegarde sur bande.
  - Un risque métier supposé en lien avec le stockage des sauvegardes dans le même emplacement physique que l’équipement de production.
- Le risque existe-t-il encore ? Oui. Même dans le cloud, une dépendance sur une seule installation présente des risques. La probabilité que ce risque affecte l’activité est inférieure à celle liée à l’utilisation d’une solution en local, mais le risque existe tout de même.
- Réécrivez la stratégie. En cas de sinistre à l’échelle du centre de données, une solution de restauration (dans les 24 heures suivant la panne) des systèmes de production dans un autre centre de données et emplacement géographique doit être en place.
- Procédez à la révision avec les équipes d’adoption du cloud. En fonction de la solution en cours d’implémentation, plusieurs moyens vous permettent de respecter cette stratégie de cohérence des ressources.

## <a name="tools-to-help-create-modern-policies"></a>Outils d’aide à la création de stratégies modernes

Pour accélérer la création de stratégies modernes, un ensemble d’exemples de stratégies est disponible dans chacune des cinq disciplines de la gouvernance cloud. Ces exemples de stratégies reposent sur l’une des trois hypothèses de conception suivantes :

- **Native cloud** : la solution en cours de déploiement est native cloud et peut exploiter les solutions par défaut disponibles dans Azure, avec une configuration minimale.
- **D’entreprise** : la solution en cours de déploiement est complexe et nécessite un modèle de déploiement cloud hybride. Des implémentations plus complexes de certaines disciplines de la gouvernance sont nécessaires.
- **Conforme au principe de conception cloud** : la solution en cours de déploiement doit respecter les axes architecturaux définis dans le principe de conception cloud, qui requiert un niveau beaucoup plus élevé de gouvernance.  

Pour chaque discipline, un exemple de stratégie doit être créé à chacun de ces niveaux. Le but de chaque exemple est de générer des idées et des discussions au sein de l’environnement de l’entreprise. Notez que ces exemples ne sont pas destinés à être utilisés comme alternatives à une stratégie informatique d’entreprise correctement construite.
