---
title: 'Framework d’adoption du cloud : Déclarations de stratégie de cohérence des ressources'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Déclarations de stratégie de cohérence des ressources
author: BrianBlanchard
ms.openlocfilehash: 9217ae73b0edf5b40bedac1cdc961b7a34da87d1
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900837"
---
# <a name="resource-consistency-sample-policy-statements"></a>Déclarations de stratégie de cohérence des ressources

Les déclarations de stratégie cloud individuelles sont des recommandations destinées à traiter des risques spécifiques identifiés lors de votre processus d'évaluation des risques. Ces déclarations doivent fournir un bref récapitulatif des risques, ainsi que des plans établis pour les gérer. Chaque définition de déclaration doit inclure les informations suivantes :

- Risque technique : récapitulatif du risque à gérer par cette stratégie.
- Déclaration de stratégie : explication succincte et claire des exigences de la stratégie.
- Options de conception : recommandations exploitables, spécifications ou autres conseils que des équipes informatiques et des développeurs peuvent utiliser lors de l'implémentation de la stratégie.

Les exemples de déclarations de stratégie suivants gèrent un certain nombre des risques métier courants liés à l'identité. Ils sont fournis à des fins de référence lors de l'élaboration de déclarations de stratégie répondant aux besoins de votre organisation. Notez que ces exemples ne sont pas destinés à être normatifs, et qu’il existe potentiellement plusieurs options de stratégie pour la gestion de tout risque spécifique. Travaillez en étroite collaboration avec les équipes métier et informatiques pour identifier les meilleures solutions de stratégie pour votre ensemble de risques unique.

## <a name="tagging"></a>Marquage

**Risque technique** : Sans marquage approprié des métadonnées associées aux ressources déployées, les opérations informatiques ne peuvent pas hiérarchiser la prise en charge ou l’optimisation des ressources selon le contrat SLA requis, l'importance pour les opérations de l’entreprise ou les coûts d'exploitation. Cela peut entraîner une mauvaise allocation des ressources informatiques, ainsi que de possibles retards en termes de résolution des incidents.

**Déclaration de stratégie** : les stratégies suivantes seront implémentées :

- Les ressources déployées doivent être étiquetées avec les valeurs suivantes : coûts, criticité, SLA et environnement.
- Les outils de gouvernance doivent valider le marquage associé aux coûts, à la criticité, aux SLA, aux applications et aux environnements. Toutes les valeurs doivent être alignées avec les valeurs prédéfinies gérées par l’équipe de gouvernance.

**Options de conception potentielles** : Dans Azure, les [balises de métadonnées de nom-valeur standard](/azure/azure-resource-manager/resource-group-using-tags) sont prises en charge pour la plupart des types de ressources. [Azure Policy](/azure/governance/policy/overview) est utilisé pour appliquer des balises spécifiques lors de la création de ressources.

## <a name="ungoverned-subscriptions"></a>Abonnements non régis

**Risque technique** : La création arbitraire d'abonnements et de groupes d’administration peut générer des sections isolées au sein de votre parc cloud, sections auxquelles les stratégies de gouvernance ne seront pas correctement appliquées.

**Déclaration de stratégie** : La création de nouveaux abonnements ou de groupes d’administration pour les applications critiques ou les données protégées nécessite un examen par l’équipe de gouvernance du cloud. Les changements approuvés seront intégrés dans une affectation de blueprint appropriée.

**Options de conception potentielles** : Réservez l’accès administratif aux [groupes d’administration Azure](/azure/governance/management-groups/) de votre organisation aux seuls membres approuvés de l'équipe de gouvernance en charge de la création de l’abonnement et du processus de contrôle d’accès.

## <a name="manage-updates-to-virtual-machines"></a>Gérer les mises à jour des machines virtuelles

**Risque technique** : Les machines virtuelles (VM) ne disposant pas des dernières mises à jour et derniers correctifs logiciels peuvent faire l'objet de problèmes de sécurité ou de performances susceptibles d'entraîner des interruptions de service.

**Déclaration de stratégie** : Les outils de gouvernance doivent veiller à ce que les mises à jour automatiques soient activées sur toutes les machines virtuelles déployées. Les violations doivent être examinées avec les équipes de gestion opérationnelle et corrigées conformément aux stratégies d’exploitation. Les ressources qui ne sont pas mises à jour automatiquement doivent être incluses dans les processus détenus par l’équipe des opérations informatiques.

**Options de conception potentielles** : Les machines virtuelles hébergées sur Azure peuvent faire l'objet d'une gestion cohérente des mises à jour à l'aide de la [solution Update Management dans Azure Automation](/azure/automation/automation-update-management).

## <a name="deployment-compliance"></a>Conformité du déploiement

**Risque technique** : Des scripts de déploiement et outils d’automatisation non vérifiés par l'équipe de gouvernance cloud peuvent donner lieu à des déploiements de ressources contraires à la stratégie.

**Déclaration de stratégie** : les stratégies suivantes seront implémentées :

- Les outils de déploiement doivent être approuvés par l’équipe de gouvernance du cloud pour garantir une gouvernance continue des ressources déployées.
- Les scripts de déploiement doivent être conservés dans un référentiel central, accessible par l’équipe de gouvernance du cloud pour effectuer des audits et révisions périodiques.

**Options de conception potentielles** : Une utilisation cohérente d'[Azure Blueprints](/azure/governance/blueprints/) pour gérer les déploiements automatisés permet des déploiements cohérents des ressources Azure, respectant les normes et stratégies de gouvernance de votre organisation.

## <a name="monitoring"></a>Surveillance

**Risque technique** : Incorrectement implémentée ou instrumentée, la surveillance peut empêcher la détection de problèmes d’intégrité liés aux charges de travail ou autres violations des stratégies.

**Déclaration de stratégie** : les stratégies suivantes seront implémentées :

- Les outils de gouvernance doivent vérifier que toutes les ressources associés aux applications critiques ou aux données protégées sont incluses dans la surveillance pour l’optimisation et l’épuisement des ressources.
- Les outils de gouvernance doivent valider que le niveau approprié de données de journalisation est collecté pour toutes les applications critiques ou données protégées.

**Options de conception potentielles** : [Azure Monitor](/azure/azure-monitor/overview) est le service de surveillance par défaut de la plateforme Azure, et une surveillance cohérente peut être appliquée à l’aide d'[Azure Blueprints](/azure/governance/blueprints/) lors du déploiement de ressources.

## <a name="disaster-recovery"></a>Récupération d'urgence

**Risque technique** : La défaillance, la suppression ou la corruption de ressources peut entraîner une interruption des applications ou services stratégiques, ainsi qu'une perte de données sensibles.

**Déclaration de stratégie** : Toutes les applications stratégiques et données protégées doivent s'accompagner de solutions de sauvegarde et de récupération mises en œuvre pour limiter l'impact des pannes et défaillances système sur l'entreprise.

**Options de conception potentielles** : Le service [Azure Site Recovery] propose des fonctionnalités de sauvegarde, de récupération et de réplication conçues pour limiter la durée des pannes dans des scénarios BCDR (continuité d'activité et reprise d'activité).

## <a name="next-steps"></a>Étapes suivantes

Utilisez les exemples mentionnés dans cet article comme point de départ pour développer des stratégies qui répondent à des risques métier spécifiques, dans la continuité de vos plans d'adoption du cloud.

Pour commencer à développer vos déclarations de stratégie personnalisées liées à la cohérence des ressources, téléchargez le [modèle Cohérence des ressources](template.md).

Pour accélérer l'adoption de cette discipline, choisissez le [parcours de gouvernance exploitable](../journeys/overview.md) correspondant le mieux à votre environnement. Modifiez ensuite la conception pour intégrer vos décisions spécifiques en matière de stratégie d'entreprise.

> [!div class="nextstepaction"]
> [Parcours de gouvernance exploitables](../journeys/overview.md)
