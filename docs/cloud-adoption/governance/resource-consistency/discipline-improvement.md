---
title: 'Framework d’adoption du cloud : Amélioration de la discipline Cohérence des ressources'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Amélioration de la discipline Cohérence des ressources
author: BrianBlanchard
ms.openlocfilehash: bc81b894d46266c52291c53dba5532ab2ab6b860
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900769"
---
# <a name="resource-consistency-discipline-improvement"></a>Amélioration de la discipline Cohérence des ressources

La discipline Cohérence des ressources se concentre sur les moyens à utiliser pour établir des stratégies en lien avec la gestion opérationnelle d'un environnement, d'une application ou d'une charge de travail. Parmi les cinq disciplines de la gouvernance cloud, la Cohérence des ressources inclut la supervision des performances des applications, des charges de travail et des ressources. Elle inclut également les tâches nécessaires pour répondre aux demandes de mise à l'échelle, corriger les violations de contrat de niveau de service et prévenir de façon proactive les violations de contrat de niveau de service via la correction automatisée.

Cet article décrit certaines tâches potentielles que votre entreprise peut entreprendre pour mieux développer et faire mûrir la discipline de Cohérence des ressources. Ces tâches peuvent être décomposées en phases de planification, de construction, d’adoption et d’exploitation de l’implémentation d’une solution cloud, qui sont ensuite répétées pour permettre le développement d’une [approche incrémentielle de la gouvernance cloud](../journeys/overview.md#an-incremental-approach-to-cloud-governance).

![Quatre phases d’adoption](../../_images/adoption-phases.png)

*Figure 1 : Phases d’adoption de l’approche incrémentielle de la gouvernance cloud.*

Il est impossible pour tout un document de prendre en compte les exigences de toutes les organisations. En tant que tel, cet article présente des exemples d’activités minimales et potentielles suggérés pour chaque phase du processus de maturation de la gouvernance. L’objectif initial de ces activités est de vous aider à créer un [MVP de stratégie](../journeys/overview.md#an-incremental-approach-to-cloud-governance) et à établir une infrastructure pour une évolution de stratégie incrémentielle. Votre équipe de gouvernance cloud devra décider combien investir dans ces activités pour améliorer vos fonctionnalités de gouvernance de Cohérence des ressources.

> [!CAUTION]
> Les activités minimales ou potentielles décrites dans cet article ne sont pas alignées sur des stratégies d’entreprise spécifiques ou des exigences de conformité de tiers. Ces conseils sont conçus pour faciliter les conversations qui conduiront à un alignement des deux exigences avec un modèle de gouvernance cloud.

## <a name="planning-and-readiness"></a>Planification et préparation

Cette phase de maturité de la gouvernance comble le fossé entre les résultats opérationnels et les stratégies concrètes. Au cours de ce processus, l’équipe de direction définit des métriques spécifiques, les mappe au parc numérique, et commence à planifier l’effort global de migration.

**Activités minimales suggérées :**

* Évaluez votre options de [chaîne d’outils en matière de cohérence des données](toolchain.md).
* Appréhendez au mieux les conditions de licence de votre stratégie cloud.
* Développez un brouillon de document d’instructions relatives à l’architecture et distribuez-le aux principales parties prenantes.
* Familiarisez-vous avec le gestionnaire de ressources que vous utilisez pour déployer, gérer et surveiller toutes les ressources de votre solution en tant que groupe.
* Formez et impliquez les personnes et les équipes concernées par le développement des instructions relatives à l'architecture.
* Ajoutez des tâches de déploiement de ressources classées par ordre de priorité à votre backlog de migration.

**Activités potentielles :**

* Collaborez avec les parties prenantes de l'entreprise et/ou l'équipe des stratégies cloud pour connaître l'approche de gestion des comptes cloud souhaitée, les pratiques de contrôle de gestion de vos unités commerciales et de votre organisation dans son ensemble.
* Définissez vos exigences en matière de [surveillance et d'application des stratégies](compliance-processes.md).
* Examinez la valeur métier et le coût des interruptions pour définir une stratégie de remédiation et des exigences de contrat SLA.
* Déterminez si vous déploierez une [charge de travail simple](./governance-simple-workload.md) ou une stratégie de gouvernance [multi-équipe](./governance-multiple-teams.md) pour vos ressources.
* Déterminez les exigences d’extensibilité correspondant à vos charges de travail prévues.

## <a name="build-and-pre-deployment"></a>Génération et prédéploiement

Un certain nombre de conditions techniques et non techniques sont requises pour migrer un environnement correctement. Ce processus se concentre sur les décisions, l’état de préparation et l’infrastructure de base qui permettent une migration.

**Activités minimales suggérées :**

* Implémentez votre [chaîne d'outils Cohérence des ressources](toolchain.md) en la déployant dans le cadre d'une phase de prédéploiement.
* Mettez à jour le document d’instructions relatives à l’architecture et distribuez-le aux principales parties prenantes.
* Implémentez des tâches de déploiement de ressources sur votre backlog de migration classé par ordre de priorité.
* Développez une documentation et des ressources pédagogiques, des communications de sensibilisation, des incitations et d’autres programmes afin d’encourager l’adoption par les utilisateurs.

**Activités potentielles :**

* Choisissez une [stratégie de conception d’abonnement](../../decision-guides/subscriptions/overview.md), en optant pour les modèles d'abonnement les mieux adaptés aux besoins de votre entreprise et de vos charges de travail.
* Utilisez une stratégie de [cohérence des ressources](../../decision-guides/resource-consistency/overview.md) pour appliquer les recommandations relatives à l’architecture dans le temps.
* Implémentez des [normes d'attribution de noms et de marquage de vos ressources](../../decision-guides/resource-tagging/overview.md) en adéquation avec vos exigences en matière d'organisation et de gestion des comptes.
* Pour créer une gouvernance durable proactive, utilisez des modèles de déploiement et une automatisation afin d'appliquer des configurations communes et une structure de regroupement cohérente lors du déploiement de ressources et de groupes de ressources.
* Établissez un modèle d’autorisations de privilège minimal dans lequel les utilisateurs ne disposent d’aucune autorisation par défaut.
* Au sein de votre organisation, déterminez les propriétaires de chaque charge de travail et compte, ainsi que les personnes qui pourront y accéder à des fins de maintenance et de modification de ces ressources. Définissez des rôles et responsabilités cloud correspondant à ces besoins et utilisez ces rôles comme base pour le contrôle d’accès.
* Définissez les dépendances entre les ressources.
* Implémentez la mise à l’échelle automatisée des ressources conformément aux exigences définies à l'étape de planification.
* Mettez en place des performances d'accès pour mesurer la qualité des services reçus.
* Envisagez de déployer une [stratégie](/azure/governance/policy/overview) pour gérer la mise en œuvre du contrat SLA à l’aide de paramètres de configuration et de règles de création de ressources.

## <a name="adopt-and-migrate"></a>Adoption et migration

Une migration est un processus incrémentiel qui se concentre sur le déplacement, le test et l’adoption d’applications ou de charges de travail dans un parc numérique existant.

**Activités minimales suggérées :**

* Migrez votre [chaîne d'outils Cohérence des ressources](toolchain.md) de la phase de prédéploiement vers la phase de production.
* Mettez à jour le document d’instructions relatives à l’architecture et distribuez-le aux principales parties prenantes.
* Développez une documentation et des ressources pédagogiques, des communications de sensibilisation, des incitations et d’autres programmes afin d’encourager l’adoption par les utilisateurs.
* Migrez tous les scripts de correction automatisés ou outils pour prendre en charge les exigences du contrat SLA.

**Activités potentielles :**

* Terminez et testez la surveillance et la création de rapports ayant trait aux données à l'aide de votre passerelle cloud locale ou d'une solution hybride.
* Déterminez si des modifications sont à apporter au contrat SLA ou à la stratégie de gestion des ressources.
* Améliorez les tâches opérationnelles en implémentant des fonctionnalités d'interrogation pour trouver efficacement des ressources dans votre parc cloud.
* Alignez les ressources à l’évolution des besoins métier et des exigences de gouvernance.
* Assurez-vous que vos machines virtuelles, réseaux virtuels et comptes de stockage reflètent les besoins d’accès aux ressources à chaque version, et procédez à des ajustements, si besoin.
* Vérifiez que la mise à l’échelle automatique des ressources répond aux exigences d’accès.
* Examinez l'accès des utilisateurs aux ressources, groupes de ressources et abonnements Azure, et ajustez les contrôles d'accès, si besoin.
* Surveillez les modifications apportées aux plans d'accès aux ressources et les valider avec les parties prenantes si des approbations supplémentaires sont nécessaires.
* Mettez à jour les recommandations relatives à l’architecture en fonction des coûts réels.
* Déterminez si, d’un point de vue financier, les abonnements des unités commerciales sont dans la lignée des comptes de résultats.
* Pour les organisations internationales, implémentez vos exigences en matière de conformité SLA et de souveraineté.
* Pour l’agrégation cloud, déployez une solution de passerelle vers un fournisseur de cloud.
* Pour les outils n'autorisant pas les options de passerelle ou hybrides, associez la surveillance à un outil de surveillance opérationnelle.

## <a name="operate-and-post-implementation"></a>Exploitation et post-implémentation

Une fois la transformation terminée, la gouvernance et les opérations doivent rester actives pendant le cycle de vie naturel d’une application ou d’une charge de travail. Cette phase de maturité de la gouvernance se concentre sur les activités couramment conduites une fois que la solution est implémentée et que le cycle de transformation commence à se stabiliser.

**Activités minimales suggérées :**

* Personnalisez votre [chaîne d’outils de cohérence des ressources](toolchain.md) selon les besoins évolutifs de votre entreprise en matière de gestion des coûts.
* Envisagez l’automatisation des notifications et des rapports à mesure que l'utilisation des ressources évolue.
* Affinez les instructions relatives à l’architecture pour guider les futurs processus d’adoption.
* Formez régulièrement les équipes concernées pour vous assurer de leur adhésion continue aux instructions relatives à l'architecture.

**Activités potentielles :**

* Ajustez les plans tous les trimestres en fonction de l’évolution des ressources réelles.
* Appliquez automatiquement les exigences de gouvernance lors de déploiements futurs.
* Évaluez les ressources sous-utilisées et déterminez s’il est intéressant de les conserver.
* Détectez les distorsions et les anomalies entre l'utilisation prévue et réelle des ressources.
* Aidez les équipes d’adoption du cloud et l’équipe de stratégie cloud à comprendre et à résoudre ces anomalies.
* Déterminez si des modifications doivent être apportées à la cohérence des ressources en termes de facturation et de contrats SLA.
* Évaluez les outils de journalisation et de surveillance pour déterminer si votre solution locale, de passerelle cloud ou hybride doit être ajustée.
* Pour les unités commerciales et groupes dispersés géographiquement, déterminez si votre organisation doit envisager d'utiliser des fonctionnalités de gestion cloud supplémentaires ([groupes d’administration Azure](/azure/governance/management-groups/), par exemple) afin de mieux appliquer la stratégie centralisée et de répondre aux exigences de contrat SLA.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous comprenez le concept de gouvernance des ressources cloud, poursuivez pour en apprendre davantage sur [la gestion de l’accès aux ressources](azure-resource-access.md) dans Azure en vue de découvrir la conception d’un modèle de gouvernance pour une [charge de travail simple](governance-simple-workload.md) ou [plusieurs équipes](governance-multiple-teams.md).

> [!div class="nextstepaction"]
> [En savoir plus sur l’accès aux ressources dans Azure](azure-resource-access.md)
> [En savoir plus sur les contrats SLA pour Azure](https://azure.microsoft.com/support/legal/sla/)
> [En savoir plus sur la journalisation, la création de rapports et la surveillance](../../decision-guides/log-and-report/overview.md)
