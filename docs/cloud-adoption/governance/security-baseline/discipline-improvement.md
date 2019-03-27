---
title: "Framework d'adoption du cloud : Amélioration de la discipline Base de référence de la sécurité"
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Amélioration de la discipline Base de référence de la sécurité
author: BrianBlanchard
ms.openlocfilehash: 28a971f56c9f8ada1d184bdc1cb3dbb9a17c3507
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58246450"
---
# <a name="security-baseline-discipline-improvement"></a>Amélioration de la discipline Base de référence de la sécurité

La discipline Base de référence de la sécurité se concentre sur l'établissement de stratégies qui protègent le réseau, les ressources et, plus important encore, les données qui résideront sur la solution d'un fournisseur de services cloud. Parmi les cinq disciplines de la gouvernance cloud, la discipline Base de référence de la sécurité inclut la classification du parc numérique et des données. Elle inclut également la documentation relative aux risques, à la tolérance métier et aux stratégies d'atténuation associées à la sécurité des données, des ressources et du réseau. D'un point de vue technique, cela englobe également la participation aux décisions concernant le [chiffrement](../../decision-guides/encryption/overview.md), la [configuration réseau requise](../../decision-guides/software-defined-network/overview.md), les [stratégies d'identités hybrides](../../decision-guides/identity/overview.md) et les [processus](compliance-processes.md) utilisés pour développer les stratégies Base de référence de la sécurité du cloud.

Cet article décrit certaines tâches potentielles que votre entreprise peut entreprendre pour mieux développer et faire mûrir la discipline de Base de référence de la sécurité. Ces tâches peuvent être décomposées en phases de planification, de construction, d'adoption et d'exploitation de l'implémentation d'une solution cloud, qui sont ensuite répétées pour permettre le développement d'une [approche incrémentielle de la gouvernance cloud](../journeys/overview.md#an-incremental-approach-to-cloud-governance).

![Quatre phases d’adoption](../../_images/adoption-phases.png)

*Figure 1 : Phases d’adoption de l’approche incrémentielle de la gouvernance cloud.*

Il est impossible pour un même document de prendre en compte les exigences de toutes les organisations. Par conséquent, cet article présente des exemples d’activités minimales et potentielles suggérés pour chaque phase du processus de maturation de la gouvernance. L'objectif initial de ces activités est de vous aider à créer un [MVP de stratégie](../journeys/overview.md#an-incremental-approach-to-cloud-governance) et à établir une infrastructure pour une évolution de stratégie incrémentielle. Votre équipe de gouvernance cloud devra décider combien investir dans ces activités pour améliorer vos fonctionnalités de gouvernance de Base de référence de la sécurité.

> [!CAUTION]
> Les activités minimales ou potentielles décrites dans cet article ne sont pas alignées sur des stratégies d'entreprise spécifiques ou des exigences de conformité de tiers. Ces conseils visent à favoriser les échanges qui conduiront à un alignement des deux exigences avec un modèle de gouvernance cloud.

## <a name="planning-and-readiness"></a>Planification et préparation

Cette phase de maturité de la gouvernance comble le fossé entre les résultats opérationnels et les stratégies concrètes. Au cours de ce processus, l’équipe de direction définit des métriques spécifiques, les mappe au patrimoine numérique, et commence à planifier l’effort global de migration.

**Activités minimales suggérées :**

- Évaluez les options de votre [chaîne d'outils Base de référence de la sécurité](toolchain.md).
- Élaborez un brouillon des instructions relatives à l'architecture et distribuez ce document aux principales parties prenantes.
- Formez et impliquez les personnes et les équipes concernées par le développement des instructions relatives à l'architecture.
- Ajoutez des tâches de sécurité classées par ordre de priorité à votre backlog de migration.

**Activités potentielles :**

- Définissez un schéma de classification des données.
- Menez un processus de planification du parc numérique pour dresser l'inventaire des ressources informatiques qui alimentent actuellement vos processus métier et les opérations associées.
- Menez une [révision des stratégies](../../governance/policy-compliance/what-is-a-cloud-policy-review.md) pour entamer le processus de modernisation des stratégies de sécurité informatique existantes, et définissez des stratégies MVP pour répondre aux risques connus.
- Passez en revue les consignes de sécurité de votre plateforme cloud. Pour Azure, vous les trouverez sur la [Plateforme d'approbation de services Microsoft](https://www.microsoft.com/trustcenter/stp/default.aspx).
- Déterminez si votre stratégie Base de référence de la sécurité inclut un [Cycle de vie de développement de la sécurité](https://www.microsoft.com/securityengineering/sdl/).
- Évaluez les risques métier liés au réseau, aux données et aux ressources en fonction des trois versions suivantes et déterminez la tolérance de votre organisation vis-à-vis de ces risques.
- Consultez le rapport Microsoft consacré aux [principales tendances en matière de cybersécurité](https://www.microsoft.com/security/operations/security-intelligence-report) pour obtenir un aperçu du paysage de la sécurité actuel.
- Envisagez de développer un rôle [Security DevOps](https://www.microsoft.com/en-us/securityengineering/devsecops) au sein de votre organisation.

<!-- "en-us" location is required for the URL above. -->

## <a name="build-and-pre-deployment"></a>Génération et prédéploiement

Un certain nombre de conditions techniques et non techniques sont requises pour migrer un environnement correctement. Ce processus se concentre sur les décisions, l’état de préparation et l’infrastructure de base qui permettent une migration.

**Activités minimales suggérées :**

- Implémentez votre [chaîne d'outils Base de référence de la sécurité](toolchain.md) en la déployant dans le cadre d'une phase de prédéploiement.
- Mettez à jour les instructions relatives à l'architecture et distribuez le document aux principales parties prenantes.
- Implémentez des tâches de sécurité sur votre backlog de migration classé par ordre de priorité.
- Développez une documentation et des ressources pédagogiques, des communications de sensibilisation, des incitations et d'autres programmes afin d'encourager l'adoption par les utilisateurs.

**Activités potentielles :**

- Déterminez la stratégie de [chiffrement](../../decision-guides/encryption/overview.md) de votre organisation pour les données hébergées dans le cloud.
- Évaluez la stratégie d'[identité](../../decision-guides/identity/overview.md) de votre déploiement cloud. Déterminez comment votre solution d'identité basée sur le cloud coexistera avec les fournisseurs d'identité locaux ou s'intégrera à ceux-ci.
- Déterminez les stratégies de délimitation du réseau pour la conception de votre [réseau à définition logicielle](../../decision-guides/software-defined-network/overview.md) afin de bénéficier de capacités de mise en réseau sécurisées et virtualisées.
- Évaluez les stratégies d'[accès à privilège minimum](/azure/active-directory/users-groups-roles/roles-delegate-by-task) de votre organisation et utilisez des rôles basés sur des tâches pour fournir un accès à des ressources spécifiques.
- Appliquez des mécanismes de sécurité et de supervision à tous les services cloud et à toutes les machines virtuelles.
- Si possible, automatisez les [stratégies de sécurité](../../decision-guides/policy-enforcement/overview.md).
- Passez en revue votre stratégie Base de référence de la sécurité et déterminez si vous devez modifier vos plans en fonction des meilleures pratiques, comme celles décrites dans le [Cycle de vie de développement de la sécurité](https://www.microsoft.com/securityengineering/sdl/).

## <a name="adopt-and-migrate"></a>Adoption et migration

La migration est un processus incrémentiel qui porte essentiellement sur le déplacement, le test et l’adoption d’applications ou de charges de travail dans un patrimoine numérique.

**Activités minimales suggérées :**

- Migrez votre [chaîne d'outils Base de référence de la sécurité](toolchain.md) de la phase de prédéploiement vers la phase de production.
- Mettez à jour les instructions relatives à l'architecture et distribuez le document aux principales parties prenantes.
- Développez une documentation et des ressources pédagogiques, des communications de sensibilisation, des incitations et d'autres programmes afin d'encourager l'adoption par les utilisateurs.

**Activités potentielles :**

- Consultez les dernières informations relatives à la Base de référence de la sécurité et aux menaces pour identifier tout nouveau risque.
- Évaluez la tolérance de votre organisation pour faire face aux nouveaux risques liés à la sécurité.
- Identifiez les écarts par rapport la stratégie et procédez à des corrections.
- Ajustez la sécurité et l'automatisation du contrôle d'accès afin d'assurer une conformité maximale avec la stratégie.  
- Vérifiez que les meilleures pratiques définies pendant les phases de génération/prédéploiement sont correctement mises en œuvre.
- Passez en revue vos stratégies d'accès à privilège minimum et ajustez les contrôles d'accès pour optimiser la sécurité.
- Testez votre chaîne d'outils Base de référence de la sécurité sur vos charges de travail pour identifier et résoudre les vulnérabilités éventuelles.

## <a name="operate-and-post-implementation"></a>Exploitation et post-implémentation

Une fois la transformation terminée, la gouvernance et les opérations doivent perdurer tout au long du cycle de vie naturel d’une application ou d’une charge de travail. Cette phase de maturité de la gouvernance est centrée sur les activités qui interviennent généralement après l’implémentation de la solution et le début de stabilisation du cycle de transformation.

**Activités minimales suggérées :**

- Validez et/ou affinez votre [chaîne d'outils Base de référence de la sécurité](toolchain.md).
- Personnalisez les notifications et les rapports pour vous avertir des problèmes de sécurité potentiels.
- Affinez les instructions relatives à l'architecture pour guider les futurs processus d'adoption.
- Communiquez avec les équipes concernées et formez-les régulièrement pour vous assurer de leur adhésion continue aux instructions relatives à l'architecture.

**Activités potentielles :**

- Découvrez des modèles et comportements pour vos charges de travail, et configurez vos outils de supervision et de création de rapports de manière à identifier et à vous avertir des activités, accès ou utilisations anormaux des ressources.
- Mettez continuellement à jour vos stratégies de supervision et de création de rapports pour détecter les dernières vulnérabilités et attaques.
- Mettez des procédures en place pour arrêter rapidement les accès non autorisés et désactiver les ressources susceptibles d'avoir été compromises par un attaquant.
- Passez régulièrement en revue les meilleures pratiques de sécurité et, si possible, appliquez les recommandations à votre stratégie de sécurité ainsi qu'à vos fonctionnalités d'automatisation et de supervision.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous vous êtes familiarisé avec le concept de gouvernance de la sécurité dans le cloud, découvrez les [consignes de sécurité et les meilleures pratiques fournies par Microsoft](azure-security-guidance.md) pour Azure.

> [!div class="nextstepaction"]
> [En savoir plus sur les consignes de sécurité relatives à Azure](azure-security-guidance.md)
> [Présentation d'Azure Security](/azure/security/azure-security)
> [En savoir plus sur la journalisation, la création de rapports et la supervision](../../decision-guides/log-and-report/overview.md)
