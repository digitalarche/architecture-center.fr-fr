---
title: Grandes entreprises - Évolution de la cohérence des ressources
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Grandes entreprises - Évolution de la cohérence des ressources
author: BrianBlanchard
ms.openlocfilehash: 9fc6ca015310c02338463d3c8aa53ddfc74699df
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900902"
---
# <a name="large-enterprise-resource-consistency-evolution"></a>Grandes entreprises : Évolution de la cohérence des ressources

Cet article fait évoluer le scénario en ajoutant des contrôles de cohérence des ressources au produit minimum viable pour la gouvernance à des fins de prise en charge des applications critiques.

## <a name="evolution-of-the-narrative"></a>Évolution du scénario

Les équipes d’adoption du cloud ont respecté toutes les conditions pour déplacer les données protégées. Ces applications s’accompagnent d’engagements en matière de SLA pour l’entreprise et d’exigences de prise en charge de l’équipe responsable des opérations informatiques. Juste derrière l’équipe chargée de la migration des deux centres de données, plusieurs équipes Business Intelligence et de développement d’applications sont prêtes à lancer de nouvelles solutions en production. L’équipe responsable des opérations informatiques ne connaît pas encore les opérations cloud et a besoin d’un outil pour intégrer rapidement les processus opérationnels existants.

### <a name="evolution-of-current-state"></a>Évolution de l’état actuel

- L’équipe informatique procède au déplacement des charges de travail de production avec des données protégées dans Azure. Plusieurs charges de travail de basse priorité assurent le trafic de production. Bien d’autres peuvent être transférées une fois que l’équipe responsable des opérations informatiques sera prête à prendre en charge les charges de travail.
- Les équipes de développement d’applications sont prêtes pour le trafic de production.
- L’équipe Business Intelligence est prête à intégrer les prédictions et insights dans les systèmes qui exécutent des opérations pour les trois unités commerciales.

### <a name="evolution-of-the-future-state"></a>Évolution de l’état futur

- L’équipe responsable des opérations informatiques ne connaît pas encore les opérations cloud et a besoin d’un outil pour intégrer rapidement les processus opérationnels existants.

Les modifications apportées aux états actuel et futur exposent à de nouveaux risques qui nécessiteront de nouvelles instructions de stratégie.

## <a name="evolution-of-tangible-risks"></a>Évolution des risques tangibles

**Interruption de l’activité** : chaque nouvelle plateforme risque de provoquer des interruptions de processus métier critiques. L’équipe responsable des opérations informatiques et les équipes travaillant sur diverses adoptions cloud sont relativement peu expérimentées en matière d’opérations cloud. Cela augmente le risque d’interruption, qui doit être atténué et régi.

Ce risque métier peut être lié à plusieurs risques techniques :

- Les processus opérationnels mal alignés peuvent conduire à des pannes impossibles à détecter et corriger rapidement.
- Des intrusions externes ou des attaques par déni de service peuvent conduire à une interruption de l’activité.
- Des ressources critiques peuvent ne pas être détectées correctement, et donc ne pas être gérées comme il se doit.
- Des ressources non détectées ou étiquetées de façon incorrecte peuvent ne pas être prises en charge pas les processus de gestion opérationnelle existants.
- La configuration des ressources déployées peut ne pas répondre aux attentes en termes de performances.
- La journalisation peut ne pas être correctement enregistrée et centralisée pour permettre la résolution des problèmes de performances.
- Les stratégies de récupération peuvent échouer ou durer plus longtemps que prévu.
- Des processus de déploiement incohérents peuvent entraîner des failles de sécurité pouvant être à l’origine de fuites de données ou d’interruptions.
- Des changements de configuration ou des correctifs manqués peuvent entraîner des failles de sécurité pouvant être à l’origine de fuites de données ou d’interruptions.
- La configuration peut ne pas appliquer les exigences des SLA définis ou les exigences de récupération approuvées.
- Des applications ou systèmes d’exploitation déployés peuvent ne pas répondre aux exigences de renforcement de la sécurité des applications et des systèmes d’exploitation.
- Il existe un risque d’incohérence en raison du nombre d’équipes travaillant dans le cloud.

## <a name="evolution-of-the-policy-statements"></a>Évolution des instructions de stratégie

Les modifications suivantes apportées à la stratégie permettront de réduire les nouveaux risques et de guider l’implémentation. La liste peut sembler longue, mais l’adoption de ces stratégies est plus simple qu’il n’y paraît.

1. Toutes les ressources déployées doivent être classées par criticité et classification des données. Les classifications doivent être examinées par l’équipe de gouvernance cloud et le propriétaire de l’application avant le déploiement sur le cloud.
2. Les sous-réseaux hébergeant des applications critiques doivent être protégés par un pare-feu capable de détecter les intrusions et de répondre aux attaques.
3. Les outils de gouvernance doivent auditer et appliquer les exigences de configuration réseau définies par l’équipe de base de référence de sécurité.
4. Les outils de gouvernance doivent vérifier que toutes les ressources associées aux applications critiques ou aux données protégées sont incluses dans la surveillance pour l’optimisation et l’épuisement des ressources.
5. Les outils de gouvernance doivent valider que le niveau approprié de données de journalisation est collecté pour toutes les applications critiques ou données protégées.
6. Le processus de gouvernance doit valider que la sauvegarde, la restauration et le respect des SLA sont implémentés correctement pour les applications critiques et les données protégées.
7. Les outils de gouvernance doivent limiter le déploiement des machines virtuelles aux images approuvées uniquement.
8. Les outils de gouvernance doivent faire en sorte **d’empêcher** les mises à jour automatiques sur toutes les ressources déployées qui prennent en charge les applications critiques. Les violations doivent être examinées avec les équipes de gestion opérationnelle et corrigées conformément aux stratégies liées aux opérations. Les ressources qui ne sont pas automatiquement mises à jour doivent être incluses dans les processus détenus par l’équipe responsable des opérations informatiques.
9. Les outils de gouvernance doivent valider le marquage associé aux coûts, à la criticité, aux SLA, aux applications et aux classifications des données. Toutes les valeurs doivent être alignées avec les valeurs prédéfinies gérées par l’équipe de gouvernance cloud.
10. Les processus de gouvernance doivent inclure des audits, à intervalles réguliers, au point de déploiement pour garantir la cohérence entre toutes les ressources.
11. Les tendances et les attaques susceptibles d’affecter les déploiements cloud doivent être régulièrement examinées par l’équipe de sécurité pour fournir des mises à jour aux outils de base de référence de sécurité utilisés dans le cloud.
12. Avant la mise en production, toutes les applications critiques et données protégées doivent être ajoutées à la solution de surveillance opérationnelle désignée. Les ressources qui ne peuvent pas être détectées par les outils choisis par l’équipe responsable des opérations informatiques ne pourront pas être utilisées en production. Les modifications nécessaires pour que ces ressources puissent être détectées doivent être apportées dans les processus de déploiement appropriés pour garantir la bonne détection des ressources dans les prochains déploiements.
13. Suite à leur détection, la taille des ressources doit être validée par les équipes de gestion opérationnelle pour garantir qu’elles respectent les exigences en matière de performances.
14. Les outils de déploiement doivent être approuvés par l’équipe de gouvernance cloud pour s’assurer de la gouvernance en continu des ressources déployées.
15. Les scripts de déploiement doivent être conservés dans un référentiel central, accessible par l’équipe de gouvernance cloud pour effectuer des audits et révisions périodiques.
16. Les processus de révision de la gouvernance doivent valider que les ressources déployées sont correctement configurées selon les exigences des SLA et de récupération.

## <a name="evolution-of-the-best-practices"></a>Évolution des meilleures pratiques

Cette section de l’article va faire évoluer la conception du produit minimum viable pour la gouvernance afin d’inclure de nouvelles stratégies Azure, ainsi qu’une implémentation d’Azure Cost Management. Ensemble, ces deux modifications de la conception permettront de répondre aux nouvelles instructions de la stratégie d’entreprise.

En reprenant l’exemple utilisé dans ce document, nous partons du principe que l’évolution des données protégées a déjà eu lieu. Basée sur cette meilleure pratique, la procédure suivante vous permettra d’ajouter des exigences de surveillance opérationnelle pour préparer un abonnement aux applications critiques.

**Abonnement équipe informatique de l’entreprise** : Ajoutez ce qui suit à l’abonnement équipe informatique de l’entreprise, qui joue le rôle d’un hub.

1. En tant que dépendance externe, l’équipe responsable des opérations cloud devra définir des outils de surveillance opérationnelle, des outils de continuité d’activité et reprise d’activité et des outils de correction automatisée. L’équipe de gouvernance cloud peut ensuite prendre en charge les processus de détection nécessaires.
    1. Ici, l’équipe responsable des opérations cloud a choisi Azure Monitor en tant qu’outil principal pour la surveillance des applications critiques.
    2. Elle a également sélectionné Azure Site Recovery en tant qu’outil principal pour la continuité d’activité et reprise d’activité.
2. Implémentation d’Azure Site Recovery
    1. Définissez et déployez Azure Vault pour les processus de sauvegarde et de récupération
    2. Créez un modèle de gestion des ressources Azure pour mettre en place un coffre dans chaque abonnement
3. Implémentation d’Azure Monitor
    1. Une fois qu’un abonnement critique est identifié, un espace de travail Log Analytics peut être cré à l’aide de PowerShell. Il s’agit d’un processus de prédéploiement.

**Abonnement équipe adoption du cloud individuel** : ce qui suit garantira que chaque abonnement peut être détecté par la solution de surveillance et prêt pour être inclus dans les pratiques de continuité d’activité et reprise d’activité.

1. Azure Policy pour les nœuds critiques
    1. Auditez et appliquez l’utilisation de rôles standard uniquement.
    2. Auditez et appliquez le chiffrement pour tous les comptes de stockage.
    3. Auditez et appliquez l’utilisation d’un réseau virtuel et sous-réseau de réseau approuvé par interface réseau.
    4. Auditez et appliquez la limitation de tables de routage définies par l’utilisateur.
    5. Auditez et appliquez le déploiement d’agents Log Analytics pour les machines virtuelles Windows et Linux.
2. Blueprint Azure
    1. Créez un blueprint nommé `mission-critical-workloads-and-protected-data`. Ce blueprint appliquera des ressources en plus du blueprint de données protégées.
    2. Ajoutez les nouvelles stratégies Azure au blueprint.
    3. Appliquez le blueprint à tous les abonnements devant héberger une application critique.

## <a name="conclusion"></a>Conclusion

L’ajout de ces processus et modifications au produit minimum viable pour la gouvernance permet d’atténuer de nombreux risques associés à la gouvernance des ressources. Ensemble, ils apportent les contrôles de récupération, de taille et de surveillance nécessaires aux opérations orientées cloud.

## <a name="next-steps"></a>Étapes suivantes

À mesure que l’adoption du cloud évolue et offre davantage de valeur aux activités métier, les risques et besoins en matière de gouvernance cloud doivent aussi évoluer. Pour l’entreprise fictive dont il est question dans cet article, la prochaine échéance sera lorsque le déploiement comptera plus de 1 000 ressources sur le cloud ou lorsque les dépenses mensuelles s’élèveront à plus de 10 000 USD. Une fois ces seuils atteints, l’équipe de gouvernance cloud ajoutera des contrôles de gestion des coûts.

> [!div class="nextstepaction"]
> [Évolution de la gestion des coûts](./cost-management-evolution.md)