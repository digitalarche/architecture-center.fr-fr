---
title: 'Framework d’adoption du cloud : Petites et moyennes entreprises - Évolution de la cohérence des ressources '
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: 'Explication : Petites et moyennes entreprises - Évolution de la cohérence des ressources'
author: BrianBlanchard
ms.openlocfilehash: 402bb3ff4182123cdc8825522f965f7cf8637980
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900950"
---
# <a name="small-to-medium-enterprise-resource-consistency-evolution"></a>Petites et moyennes entreprises : Évolution de la cohérence des ressources

Cet article fait évoluer le scénario en ajoutant des contrôles de cohérence des ressources à des fins de prise en charge des applications critiques.

## <a name="evolution-of-the-narrative"></a>Évolution du scénario

Les nouvelles expériences client, les nouveaux outils de prédiction et les infrastructures migrées continuent de progresser. L’entreprise est maintenant prête pour commencer à utiliser ces ressources en production.

### <a name="evolution-of-the-current-state"></a>Évolution de l’état actuel

Dans la phase précédente de ce scénario, les équipes Business Intelligence et de développement d’applications étaient pratiquement prêtes à intégrer des données financières et client dans des charges de travail de production. L’équipe informatique procédait à la mise hors service du centre de données de récupération d’urgence.

Depuis, certains changements ont eu lieu et ceux-ci vont avoir un impact sur la gouvernance :

- L’équipe informatique a mis hors service l’intégralité du centre de données de récupération d’urgence plus tôt que prévu. Dans le processus, de nombreuses ressources du centre de données de production ont été identifiées comme de bonnes candidates pour la migration cloud.
- Les équipes de développement d’applications sont désormais prêtes pour le trafic de production.
- L’équipe Business Intelligence est prête à injecter des prédictions et des insights dans les systèmes d’exploitation du centre de données de production.

### <a name="evolution-of-the-future-state"></a>Évolution de l’état futur

Avant d’utiliser des déploiements Azure dans des processus métier de production, les opérations cloud doivent gagner en maturité. Dans le même temps, une évolution supplémentaire de la gouvernance est nécessaire pour garantir la bonne gestion des ressources.

Les modifications apportées aux états actuel et futur exposent à de nouveaux risques qui nécessiteront de nouvelles instructions de stratégie.

## <a name="evolution-of-tangible-risks"></a>Évolution des risques tangibles

**Interruption de l’activité** : chaque nouvelle plateforme risque de provoquer des interruptions de processus métier critiques. L’équipe responsable des opérations informatiques et les équipes travaillant sur diverses adoptions cloud sont relativement peu expérimentées en matière d’opérations cloud. Cela augmente le risque d’interruption, qui doit être atténué et régi.

Ce risque métier peut être associé à quelques risques techniques :

- Des intrusions externes ou des attaques par déni de service peuvent conduire à une interruption de l’activité.
- Des ressources critiques peuvent ne pas être détectées correctement, et donc ne pas être gérées comme il se doit.
- Des ressources non détectées ou étiquetées de façon incorrecte peuvent ne pas être prises en charge pas les processus de gestion opérationnelle existants.
- La configuration des ressources déployées peut ne pas répondre aux attentes en termes de performances.
- La journalisation peut ne pas être correctement enregistrée et centralisée pour permettre la résolution des problèmes de performances.
- Les stratégies de récupération peuvent échouer ou durer plus longtemps que prévu.
- Des processus de déploiement incohérents peuvent entraîner des failles de sécurité pouvant être à l’origine de fuites de données ou d’interruptions.
- Des changements de configuration ou des correctifs manqués peuvent entraîner des failles de sécurité pouvant être à l’origine de fuites de données ou d’interruptions.
- La configuration peut ne pas appliquer les exigences des SLA définis ou les exigences de récupération approuvées.
- Des applications ou systèmes d’exploitation déployés peuvent ne pas répondre aux exigences de renforcement de la sécurité.
- En raison du nombre d’équipes travaillant dans le cloud, il existe un risque d’incohérence.

## <a name="evolution-of-the-policy-statements"></a>Évolution des instructions de stratégie

Les modifications suivantes apportées à la stratégie permettront de réduire les nouveaux risques et de guider l’implémentation. La liste peut sembler longue, mais l’adoption de ces stratégies est plus simple qu’il n’y paraît.

1. Toutes les ressources déployées doivent être classées par criticité et classification des données. Les classifications doivent être examinées par l’équipe de gouvernance cloud et le propriétaire de l’application avant le déploiement sur le cloud.
2. Les sous-réseaux hébergeant des applications critiques doivent être protégés par un pare-feu capable de détecter les intrusions et de répondre aux attaques.
3. Les outils de gouvernance doivent auditer et appliquer les exigences de configuration réseau définies par l’équipe de gestion de la sécurité.
4. Les outils de gouvernance doivent vérifier que toutes les ressources associées aux applications critiques ou aux données protégées sont incluses dans la surveillance pour l’optimisation et l’épuisement des ressources.
5. Les outils de gouvernance doivent valider que le niveau approprié de données de journalisation est collecté pour toutes les applications critiques ou données protégées.
6. Le processus de gouvernance doit valider que la sauvegarde, la restauration et le respect des SLA sont implémentés correctement pour les applications critiques et les données protégées.
7. Les outils de gouvernance doivent limiter les déploiements des machines virtuelles aux images approuvées uniquement.
8. Les outils de gouvernance doivent faire en sorte d’empêcher les mises à jour automatiques sur toutes les ressources déployées qui prennent en charge les applications critiques. Les violations doivent être examinées avec les équipes de gestion opérationnelle et corrigées conformément aux stratégies liées aux opérations. Les ressources qui ne sont pas mises à jour automatiquement doivent être incluses dans les processus détenus par l’équipe responsable des opérations informatiques.
9. Les outils de gouvernance doivent valider le marquage associé aux coûts, à la criticité, aux SLA, aux applications et aux classifications des données. Toutes les valeurs doivent être alignées avec les valeurs prédéfinies gérées par l’équipe de gouvernance.
10. Les processus de gouvernance doivent inclure des audits, à intervalles réguliers, au point de déploiement pour garantir la cohérence entre toutes les ressources.
11. Les tendances et les attaques susceptibles d’affecter les déploiements cloud doivent être régulièrement examinées par l’équipe de sécurité de façon à fournir des mises à jour aux outils de gestion de la sécurité utilisés dans le cloud.
12. Avant la mise en production, toutes les applications critiques et données protégées doivent être ajoutées à la solution de surveillance opérationnelle désignée. Les ressources qui ne peuvent pas être détectées par les outils choisis par l’équipe responsable des opérations informatiques ne pourront pas être utilisées en production. Les modifications nécessaires pour que ces ressources puissent être détectées doivent être apportées dans les processus de déploiement appropriés pour garantir la bonne détection des ressources dans les prochains déploiements.
13. Suite à la détection des ressources, les équipes de gestion opérationnelle dimensionneront les ressources pour s’assurer qu’elles répondent aux exigences en termes de performances.
14. Les outils de déploiement doivent être approuvés par l’équipe de gouvernance cloud pour s’assurer de la gouvernance en continu des ressources déployées.
15. Les scripts de déploiement doivent être conservés dans un référentiel central, accessible par l’équipe de gouvernance cloud pour effectuer des audits et des révisions périodiques.
16. Les processus de révision de la gouvernance doivent valider que les ressources déployées sont correctement configurées selon les exigences des SLA et de récupération.

## <a name="evolution-of-the-best-practices"></a>Évolution des meilleures pratiques

Cette section de l’article va faire évoluer la conception du produit minimum viable pour la gouvernance afin d’inclure de nouvelles stratégies Azure, ainsi qu’une implémentation d’Azure Cost Management. Ensemble, ces deux modifications de la conception permettront de répondre aux nouvelles instructions de la stratégie d’entreprise.

1. L’équipe responsable des opérations cloud définira des outils de surveillance opérationnelle et de correction automatisée. L’équipe de gouvernance cloud prendra en charge ces processus de détection. Ici, l’équipe responsable des opérations cloud a choisi Azure Monitor en tant qu’outil principal pour la surveillance des applications critiques.
2. Créez un référentiel dans Azure DevOps pour stocker et gérer les versions de tous les modèles Resource Manager pertinents et des configurations utilisant des scripts.
3. Implémentation d’Azure Vault :
    1. Définissez et déployez Azure Vault pour les processus de sauvegarde et de récupération.
    2. Créez un modèle Resource Manager pour mettre en place un coffre dans chaque abonnement.
4. Mettez à jour Azure Policy pour tous les abonnements :
    1. Auditez et appliquez la classification des données et la criticité sur tous les abonnements, de façon à identifier les abonnements avec des ressources critiques.
    2. Auditez et appliquez l’utilisation exclusive d’images approuvées.
5. Implémentation d’Azure Monitor :
    1. Une fois qu’un abonnement critique est identifié, créez un espace de travail Azure Monitor à l’aide de PowerShell. Il s’agit d’un processus de prédéploiement.
    2. Pendant les tests de déploiement, l’équipe responsable des opérations cloud déploie les agents nécessaires et teste la détection.
6. Mettez à jour Azure Policy pour tous les abonnements qui contiennent des applications critiques.
    1. Auditez et imposez l’application d’un groupe de sécurité réseau à toutes les cartes réseau et à tous les sous-réseaux. Les équipes responsables de la sécurité informatique et de la mise en réseau définissent le groupe de sécurité réseau.
    2. Auditez et imposez l’utilisation de sous-réseaux et réseaux virtuels approuvés pour chaque interface réseau.
    3. Auditez et appliquez la limitation de tables de routage définies par l’utilisateur.
    4. Auditez et appliquez le déploiement des agents Azure Monitor pour toutes les machines virtuelles.
    5. Auditez et imposez la présence d’Azure Vault dans l’abonnement.
7. Configuration du pare-feu :
    1. Identifiez une configuration de Pare-feu Azure qui répond aux exigences de sécurité. Vous pouvez également identifier une appliance tierce qui est compatible avec Azure.
    2. Créez un modèle Resource Manager pour déployer le pare-feu avec les configurations nécessaires.
8. Blueprint Azure :
    1. Créez un blueprint Azure nommé `protected-data`.
    2. Ajoutez les modèles de pare-feu et Azure Vault au blueprint.
    3. Ajoutez les nouvelles stratégies pour les abonnements de données protégées.
    4. Publiez le blueprint dans les groupes d’administration destinés à héberger les applications critiques.
    5. Appliquez le nouveau blueprint à chaque abonnement affecté, en plus des blueprints existants.

## <a name="conclusion"></a>Conclusion

Ces processus et modifications supplémentaires du produit minimum viable pour la gouvernance permettent d’atténuer de nombreux risques associés à la gouvernance des ressources. Ensemble, ils apportent les contrôles de récupération, de taille et de surveillance nécessaires aux opérations orientées cloud.

## <a name="next-steps"></a>Étapes suivantes

À mesure que l’adoption du cloud évolue et offre davantage de valeur aux activités métier, les risques et besoins en matière de gouvernance cloud doivent aussi évoluer. Pour l’entreprise fictive dont il est question dans cet article, la prochaine échéance sera lorsque le déploiement comptera plus de 100 ressources sur le cloud ou lorsque les dépenses mensuelles s’élèveront à plus de 1 000 USD. Une fois ces seuils atteints, l’équipe de gouvernance cloud ajoutera des contrôles de gestion des coûts.

> [!div class="nextstepaction"]
> [Évolution de la gestion des coûts](./cost-management-evolution.md)