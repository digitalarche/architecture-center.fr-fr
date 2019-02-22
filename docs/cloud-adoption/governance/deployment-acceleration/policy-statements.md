---
title: 'Framework d’adoption du cloud : Exemples d’instructions de stratégie pour l’accélération du déploiement'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Exemples d’instructions de stratégie pour l’accélération du déploiement
author: alexbuckgit
ms.openlocfilehash: 4f7d59e6653c29db03f966d1c7105524b72586fc
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900997"
---
# <a name="deployment-acceleration-sample-policy-statements"></a>Exemples d’instructions de stratégie pour l’accélération du déploiement

Les instructions de stratégie cloud individuelles sont des recommandations pour la gestion de risques spécifiques identifiés lors de votre processus d’évaluation des risques. Ces instructions doivent fournir un bref récapitulatif des risques, ainsi que des plans établis pour les gérer. Chaque définition d’instruction doit inclure les informations suivantes :

- **Risque technique.** Un récapitulatif du risque que cette stratégie gérera.
- **Instruction de stratégie.** Une explication succincte claire des exigences de la stratégie.
- **Options de conception.** Recommandations exploitables, spécifications ou autres conseils que des équipes informatiques et des développeurs peuvent utiliser lors de l’implémentation de la stratégie.

Les exemples d’instructions de stratégie suivants gèrent un certain nombre des risques métier courants liés à la configuration. Ils sont fournis à des fins de référence lors de l’élaboration d’instructions de stratégie répondant aux besoins de votre organisation. Notez que ces exemples ne sont pas destinés à être normatifs, et qu’il existe potentiellement plusieurs options de stratégie pour la gestion de tout risque spécifique. Travaillez en étroite collaboration avec les équipes métier et informatiques pour identifier les meilleures solutions de stratégie pour votre ensemble de risques unique.

## <a name="reliance-on-manual-deployment-or-configuration-of-systems"></a>Dépendance aux déploiements ou configurations manuels des systèmes

**Risque technique** : dépendre des interventions humaines lors d’un déploiement ou d’une configuration augmente le risque d’erreur humaine et réduit la répétabilité et la prévisibilité des configurations et déploiements des systèmes. En règle générale, cela ralentit le déploiement des ressources du système.

**Instruction de stratégie** : dans la mesure du possible, toutes les ressources déployées dans le cloud doivent être déployées à l’aide de modèles ou de scripts d’automatisation.

**Options de conception potentielles** : les [modèles Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview#template-deployment) proposent une approche de type « infrastructure en tant que code » pour déployer vos ressources sur Azure. [Azure Building Blocks](https://github.com/mspnp/template-building-blocks/wiki) fournit un outil en ligne de commande et un ensemble de modèles Resource Manager conçus pour simplifier le déploiement des ressources Azure.

## <a name="lack-of-visibility-into-system-issues"></a>Manque de visibilité sur les problèmes du système

**Risque technique** : une surveillance et des diagnostics insuffisants pour les systèmes métier empêchent le personnel responsable des opérations d’identifier et de corriger les problèmes avant une panne du système. En outre, cela peut augmenter de façon significative le temps nécessaire à la résolution d’une panne.

**Instruction de stratégie** : les stratégies suivantes seront implémentées :

- Des mesures de diagnostics et des métriques clés seront identifiées pour tous les composants et systèmes de production, et des outils de diagnostics et de surveillance seront appliqués à ces systèmes et surveillés de façon régulière par le personnel responsable des opérations.
- L’équipe responsable des opérations envisagera d’utiliser les outils de surveillance et de diagnostics dans les environnements autres que ceux de production, comme les environnements intermédiaires et AQ, pour identifier les problèmes des systèmes avant qu’ils ne se répercutent dans l’environnement de production.

**Options de conception potentielles** : [Azure Monitor](/azure/azure-monitor/), qui inclut également Log Analytics et Application Insights, fournit des outils pour la collecte et l’analyse des données de télémétrie afin de vous aider à comprendre le fonctionnement de vos applications et d’identifier de manière proactive les problèmes qu’elles rencontrent et les ressources dont elles dépendent.

## <a name="configuration-security-reviews"></a>Révisions de la sécurité des configurations

**Risque technique** : au fil du temps, de nouvelles problématiques ou menaces de sécurité peuvent augmenter les risques liés à un accès non autorisé aux ressources sécurisées.

**Instruction de stratégie** : les processus de gouvernance cloud doivent inclure des révisions trimestrielles avec les équipes de gestion de la configuration. L’objectif est d’identifier les modèles d’utilisation ou les acteurs malveillants devant être bloqués par la configuration des ressources cloud.

**Options de conception potentielles** : tous les trimestres, organisez une réunion destinée à la révision de la sécurité avec les membres de l’équipe de gouvernance et le personnel informatique responsable de la configuration des ressources et applications cloud. Passez en revue les métriques et données de sécurité existantes pour déterminer les écarts dans les outils et la stratégie d’accélération du déploiement actuels, puis mettez à jour la stratégie pour atténuer les nouveaux risques.

## <a name="next-steps"></a>Étapes suivantes

Servez-vous des exemples mentionnés dans cet article pour commencer à développer des stratégies qui répondent à des risques métier spécifiques, dans la continuité de vos plans d’adoption du cloud.

Pour commencer à développer vos instructions de stratégie personnalisées pour la gestion des identités, téléchargez le [modèle de base de référence des identités](template.md).

Pour accélérer l’adoption de cette discipline, choisissez le [parcours de gouvernance actionnable](../journeys/overview.md), qui est le plus adapté à votre environnement. Modifiez ensuite la conception pour intégrer vos décisions en matière de stratégie d’entreprise.

> [!div class="nextstepaction"]
> [Parcours de gouvernance actionnables](../journeys/overview.md)