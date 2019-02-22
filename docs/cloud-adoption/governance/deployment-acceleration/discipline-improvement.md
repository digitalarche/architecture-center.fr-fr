---
title: 'Framework d’adoption du cloud : Amélioration de la discipline Accélération du déploiement'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Amélioration de la discipline Accélération du déploiement
author: alexbuckgit
ms.openlocfilehash: 98192c777d8866efb01544737e8cabea6354c4d7
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901181"
---
# <a name="deployment-acceleration-discipline-improvement"></a>Amélioration de la discipline Accélération du déploiement

La discipline Accélération du déploiement se concentre sur l’élaboration de stratégies qui garantissent que les ressources sont déployées et configurées de façon cohérente et répétée, et qu’elles restent conformes tout au long de leur cycle de vie. Au sein des cinq disciplines de la gouvernance cloud, la discipline Accélération du déploiement inclut des décisions en matière d’automatisation du déploiement, de contrôle du code source des artefacts du déploiement, de surveillance des ressources déployées pour maintenir l’état désiré et d’audit des problèmes de conformité.

Cet article décrit certaines tâches potentielles que votre entreprise peut entreprendre pour mieux développer et faire mûrir la discipline Accélération du déploiement. Ces tâches peuvent être décomposées en phases de planification, de construction, d’adoption et d’exploitation de l’implémentation d’une solution cloud, qui sont ensuite répétées pour permettre le développement d’une [approche incrémentielle de la gouvernance cloud](../journeys/overview.md#an-incremental-approach-to-cloud-governance).

![Quatre phases d’adoption](../../_images/adoption-phases.png)

*Figure 1 : Phases d’adoption de l’approche incrémentielle de la gouvernance cloud.*

Il est impossible pour un même document de prendre en compte les exigences de toutes les organisations. Par conséquent, cet article présente des exemples d’activités minimales et potentielles suggérés pour chaque phase du processus de maturation de la gouvernance. L’objectif initial de ces activités est de vous aider à créer un [produit minimum viable de stratégie](../journeys/overview.md#an-incremental-approach-to-cloud-governance) et à établir une infrastructure pour une évolution de stratégie incrémentielle. Votre équipe de gouvernance cloud devra décider combien investir dans ces activités pour améliorer vos fonctionnalités de gouvernance de base de référence des identités.

> [!CAUTION]
> Les activités minimales ou potentielles décrites dans cet article ne sont pas alignées sur des stratégies d’entreprise spécifiques ou des exigences de conformité de tiers. Ces conseils visent à favoriser les échanges qui conduiront à un alignement des deux exigences avec un modèle de gouvernance cloud.

## <a name="planning-and-readiness"></a>Planification et préparation

Cette phase de maturité de la gouvernance comble le fossé entre les résultats métier et les stratégies concrètes. Au cours de ce processus, l’équipe de direction définit des métriques spécifiques, les mappe au patrimoine numérique, et commence à planifier l’effort global de migration.

**Activités minimales suggérées :**

- Évaluez vos options de [chaîne d’outils d’accélération du déploiement](toolchain.md) et implémentez une stratégie hybride appropriée pour votre organisation.
- Développez un brouillon de document d’instructions relatives à l’architecture et distribuez-le aux principales parties prenantes.
- Formez et impliquez les personnes et les équipes affectées par le développement des instructions relatives à l’architecture.
- Formez le personnel informatique et les équipes de développement pour qu’elles comprennent les stratégies et les principes DevSecOps, ainsi que l’importance des déploiements intégralement automatisés dans la discipline Accélération du déploiement.

**Activités potentielles :**

- Définissez les rôles et attributions qui régiront la discipline Accélération du déploiement dans le cloud.

## <a name="build-and-pre-deployment"></a>Génération et prédéploiement

**Activités minimales suggérées :**

- Pour les nouvelles applications cloud, introduisez au tout début du processus de développement des déploiements intégralement automatisés. Cet investissement permettra d’améliorer la fiabilité de vos processus de tests, tout en garantissant la cohérence dans l’ensemble de vos environnements de production, QA et de développement.
- Stockez tous les artefacts du déploiement (modèles de déploiement ou scripts de configuration) à l’aide d’une plateforme de contrôle de code source telles que GitHub ou Azure DevOps.
- Envisagez de procéder à un test pilote avant d’implémenter votre [chaîne d’outils d’accélération du déploiement](toolchain.md), pour vous assurer que vos déploiements sont bien rationalisés. Tenez compte des commentaires reçus lors des tests pilotes pendant la phase de prédéploiement et répétez si nécessaire.
- Évaluez l’architecture logique et physique de vos applications, et identifiez les opportunités d’automatisation du déploiement des ressources d’application ou d’améliorations de parties de l’architecture avec d’autres ressources cloud.
- Mettez à jour le document d’instructions relatives à l’architecture pour inclure des plans de déploiement et d’adoption par les utilisateurs, puis distribuez-le aux principales parties prenantes.
- Continuez de former les personnes et les équipes les plus concernées par les instructions relatives à l’architecture.

**Activités potentielles :**

- Définissez un pipeline de déploiement continu et d’intégration continue pour gérer en intégralité les versions des mises à jour de vos applications via vos environnements de production, QA et de développement.

## <a name="adopt-and-migrate"></a>Adoption et migration

La migration est un processus incrémentiel qui se concentre sur le déplacement, le test et l’adoption d’applications ou de charges de travail dans un patrimoine numérique existant.

**Activités minimales suggérées :**

- Migrez votre [chaîne d’outils d’accélération du déploiement](toolchain.md) du développement vers la production.
- Mettez à jour le document d’instructions relatives à l’architecture et distribuez-le aux principales parties prenantes.
- Développez une documentation et des ressources pédagogiques, des communications de sensibilisation, des incitations et d’autres programmes afin d’encourager l’adoption par les équipes informatiques et de développement.

**Activités potentielles :**

- Vérifiez que les meilleures pratiques définies pendant les phases de génération/prédéploiement sont correctement mises en œuvre.
- Assurez-vous que chaque application ou charge de travail est alignée avec la stratégie d’accélération du déploiement avant la mise en production.

## <a name="operate-and-post-implementation"></a>Exploitation et post-implémentation

Une fois la transformation terminée, la gouvernance et les opérations doivent rester actives pendant le cycle de vie naturel d’une application ou d’une charge de travail. Cette phase de maturité de la gouvernance se concentre sur les activités couramment conduites une fois que la solution est implémentée et que le cycle de transformation commence à se stabiliser.

**Activités minimales suggérées :**

- Personnalisez votre [chaîne d’outils d’accélération du déploiement](toolchain.md) en fonction des changements résultant de l’évolution des besoins en matière d’identité de votre organisation.
- Automatisez les notifications et les rapports pour vous avertir des problèmes de configuration ou menaces potentiels.
- Analysez et créez des rapports sur l’utilisation des ressources et des applications.
- Générez des rapports sur les métriques post-déploiement et distribuez-les aux parties prenantes.
- Révisez les instructions relatives à l’architecture pour guider les futurs processus d’adoption.
- Continuez à communiquer avec les équipes et personnes concernées et formez-les régulièrement pour vous assurer de leur adhésion continue aux instructions relatives à l’architecture.

**Activités potentielles :**

- Configurez un outil de création de rapports et de surveillance de la configuration de l’état souhaité.
- Passez régulièrement en revue des outils et scripts de configuration pour améliorer les processus et identifier les problèmes courants.
- Travailler avec les équipes responsables du développement, des opérations et de la sécurité pour faire mûrir les pratiques DevSecOps et supprimer les silos organisationnels à l’origine du manque d’efficacité.

## <a name="next-steps"></a>Étapes suivantes

À présent que vous avez compris le concept de gouvernance des identités cloud, examinez la [chaîne d’outils de base de référence des identités](toolchain.md) pour identifier les outils et fonctionnalités Azure dont vous aurez besoin lors du développement de la discipline de gouvernance Base de référence des identités sur la plateforme Azure.

> [!div class="nextstepaction"]
> [Chaîne d’outils de base de référence des identités pour Azure](toolchain.md)
