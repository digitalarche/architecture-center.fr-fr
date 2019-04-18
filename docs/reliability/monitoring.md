---
title: Analyse l’intégrité des applications pour la fiabilité dans Azure
description: Comment utiliser la surveillance pour améliorer la fiabilité des applications dans Azure
author: MikeWasson
ms.date: 04/10/2019
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.openlocfilehash: 46f9f3fb623a2facf4b9f9c6ec57376ae59e41dc
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59646568"
---
# <a name="monitoring-application-health-for-reliability-in-azure"></a>Analyse l’intégrité des applications pour la fiabilité dans Azure

La surveillance et les diagnostics sont cruciaux pour assurer la résilience. En cas de problème, vous devez savoir *qui* a échoué, *lorsque* défaillante &mdash; et *pourquoi*.

*Surveillance* n’est pas le même que *détection des défaillances*. Par exemple, votre application peut détecter un temporaire erreur et nouvelle tentative, éviter les temps d’arrêt. Mais il doit également connecter l’opération de nouvelle tentative afin que vous puissiez surveiller le taux d’erreur pour obtenir une vue d’ensemble de l’intégrité des applications.

Considérez le processus de surveillance et de diagnostic comme un pipeline avec quatre phases distinctes : Instrumentation, collecte et stockage, analyse et diagnostic et de visualisation et alertes.

## <a name="instrumentation"></a>Instrumentation

Il n’est pas pratique surveiller votre application directement, instrumentation est donc primordial. Un système distribué à grande échelle peut s’exécuter sur des dizaines de machines de virtuelles (VM), qui sont ajoutés et supprimés au fil du temps. De même, une application de cloud peut utiliser un nombre de magasins de données et une action utilisateur unique peut couvrir plusieurs sous-systèmes.

Fournir l’instrumentation riche :

- Pour les défaillances qui sont probablement pas encore fournir suffisamment de données pour déterminer la cause, d’atténuer le problème s’est produite et vous assurer que le système reste disponible.
- Pour les défaillances qui ont déjà eu lieu, l’application doit renvoyer un message d’erreur à l’utilisateur, mais doit tenter de continuer à exécuter, avec une fonctionnalité réduite.

Systèmes de surveillance doivent collecter des détails complets afin que les applications peuvent être restaurées de manière efficace et, si nécessaire, les concepteurs et développeurs peuvent modifier le système pour éviter la réapparition.

Les données brutes de surveillance peuvent provenir de diverses sources, y compris :

- Journaux des applications, tels que ceux générés par le [Azure Application Insights](/azure/azure-monitor/app/app-insights-overview) service.
- Système d’exploitation les indicateurs de performances collectés par [Azure agents de surveillance](/azure/azure-monitor/platform/agents-overview).
- [Ressources Azure](/azure/azure-monitor/platform/metrics-supported), y compris les mesures collectées par Azure Monitor.
- [Azure Service Health](/azure/service-health/service-health-overview), qui offre un tableau de bord pour vous aider à suivre les événements actifs.
- [Journaux Azure AD](/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics) intégrées à la plateforme Azure.

Plupart des services Azure ont des métriques et diagnostics que vous pouvez configurer pour analyser et de déterminer la cause des problèmes. Pour plus d’informations, consultez [analyse les données collectées par Azure Monitor](/azure/azure-monitor/platform/data-collection).

## <a name="collection-and-storage"></a>Collecte et stockage

Données d’instrumentation brutes peuvent être conservées dans divers emplacements et les formats, notamment :

- Journaux de trace d’application
- Journaux d’activité IIS
- Compteurs de performances

Ces sources distinctes sont collectées, consolidées et placées dans des magasins de données fiables dans Azure, tels que Application Insights, les métriques Azure Monitor, l’intégrité du Service, comptes de stockage et Analytique de journal Azure.

## <a name="analysis-and-diagnosis"></a>Analyse et diagnostic

Analyser les données consolidées dans ces magasins de données pour résoudre les problèmes et obtenir une vue d’ensemble de l’intégrité des applications. En règle générale, vous pouvez [rechercher et analyser des](/azure/azure-monitor/log-query/log-query-overview) les données dans Application Insights et Analytique de journal à l’aide de requêtes de Kusto ou afficher préconfigurée des graphiques à l’aide de [solutions de gestion](/azure/azure-monitor/insights/solutions-inventory). Ou utilisez le conseiller Azure pour afficher les recommandations en se concentrant sur [résilience](/azure/advisor/advisor-high-availability-recommendations) et [performances](/azure/advisor/advisor-performance-recommendations).

## <a name="visualization-and-alerts"></a>Visualisation et alertes

Données de télémétrie présents dans un format qui facilite un opérateur à remarquer des problèmes ou tendances répétitives rapidement, telles que d’une alerte de tableau de bord ou un e-mail.

Obtenir une vue de la pile complète de l’état de l’application à l’aide de [tableaux de bord Azure](/azure/azure-portal/azure-portal-dashboards) pour créer une vue consolidée des graphiques à partir d’Application Insights, Analytique de journal, les métriques Azure Monitor et l’intégrité du Service de surveillance. Et utilisez [alertes Azure Monitor](/azure/azure-monitor/platform/alerts-overview) pour créer des notifications sur l’intégrité du Service, l’intégrité des ressources, les métriques Azure Monitor, journaux dans Log Analytique et Application Insights.

Pour plus d’informations sur la surveillance et diagnostics, consultez [surveillance et diagnostics](../best-practices/monitoring.md).

## <a name="health-probes-and-check-functions"></a>Sondes d’intégrité et les fonctions de vérification

L’intégrité et les performances d’une application peuvent se dégrader au fil du temps, et une dégradation peut ne pas être visible jusqu'à ce que l’application échoue.

Implémentez des sondes ou vérifier les fonctions et exécutez-les régulièrement à partir de l’extérieur de l’application. Ces vérifications peuvent être aussi simples que de mesurer le temps de réponse pour l’application dans son ensemble, les parties individuelles de l’application, des services spécifiques que l’application utilise ou composants distincts.

Vérifiez que les fonctions peuvent exécuter des processus pour vous assurer qu’ils produisent des résultats valides, mesurent la latence et vérifier la disponibilité et extraient des informations à partir du système.

## <a name="long-running-workflow-failures"></a>Échecs de flux de travail de longue

Flux de travail de longue durée inclure souvent plusieurs étapes, chacune d’elles doit être indépendant.

Suivre la progression du processus à long terme pour réduire la probabilité que l’intégralité du workflow devront être restaurée ou que les transactions de compensation multiples doivent être exécutées.

>[!TIP]
> Surveiller et gérer la progression des longs workflows en implémentant un modèle tel que [superviseur de l’Agent du planificateur](../patterns/scheduler-agent-supervisor.md).

## <a name="application-logs"></a>Journaux d’activité d’application

Les journaux d’activité d’application sont une source importante de données de diagnostic. Pour obtenir des informations lorsque vous en avez besoin la plupart, suivez les meilleures pratiques pour la journalisation de l’application.

### <a name="log-data-in-the-production-environment"></a>Données de journal dans l’environnement de production

Capturer des données de télémétrie fiables pendant que l’application est en cours d’exécution dans l’environnement de production, afin d’avoir suffisamment d’informations pour diagnostiquer la cause des problèmes de l’état de production.

### <a name="log-events-at-service-boundaries"></a>Journaux d’événements au niveau des limites de service

Vous devez inclure un ID de corrélation qui franchit les limites de service. Si une transaction passe par plusieurs services et un d’eux échoue, la corrélation ID vous permet de suivre les demandes entre votre application et la mise en évidence la raison pour laquelle la transaction a échoué.

### <a name="use-semantic-structured-logging"></a>Utilisez la journalisation sémantique (structurée)

Avec les journaux structurées, il est plus facile automatiser la consommation et l’analyse des données du journal, ce qui est particulièrement importantes à l’échelle du cloud. En règle générale, nous vous recommandons de stocker des données de métriques et diagnostics de ressources Azure dans un espace de travail Analytique de journal plutôt que dans un compte de stockage. De cette façon, vous pouvez utiliser des requêtes de Kusto pour obtenir les données que vous souhaitez rapidement et dans un format structuré. Vous pouvez également utiliser les API d’analyse Azure et Azure Log Analytique.

### <a name="use-asynchronous-logging"></a>Utilisez la journalisation asynchrone

Opérations de journalisation synchrone bloquent parfois votre code d’application, à l’origine des demandes de sauvegarder les journaux sont écrits. Utilisez la journalisation asynchrone pour préserver la disponibilité au cours de la journalisation de l’application.

### <a name="separate-application-logging-from-auditing"></a>Séparer la journalisation de l’application à partir de l’audit

Enregistrements d’audit sont couramment conservées de conformité ou les exigences réglementaires et doivent être terminées. Pour éviter des transactions perdues, mettre à jour les journaux d’audit séparément à partir des journaux de diagnostic.

## <a name="remote-call-statistics"></a>Statistiques d’appels distants

Suivre et rapports à distance en temps réel, les statistiques d’appels et fournissent un moyen simple de consulter ces informations, par conséquent, l’équipe d’exploitation a une vue instantanée de l’intégrité de votre application. Résumer les métriques d’appel distant, telles que la latence, débit et les erreurs dans les centiles 99 et 95.

## <a name="transient-exceptions-and-retries"></a>Nouvelles tentatives et les exceptions temporaires

Suivre le nombre d’exceptions temporaires et les nouvelles tentatives au fil du temps pour découvrir des problèmes ou des échecs dans la logique de nouvelle tentative de votre application. Une tendance à l’augmentation des exceptions au fil du temps peut indiquer que le service rencontre un problème et risque d’échouer. Pour plus d’informations, consultez [Guide spécifique relatif au service de nouvelle tentative](../best-practices/retry-service-specific.md).

## <a name="early-warning-system"></a>Système d’avertissement anticipé

Surveillez votre application pour trouver des signes précurseurs peut nécessiter une intervention proactive. Outils qui évaluent l’intégrité globale de l’application et ses dépendances vous aident à reconnaître rapidement quand un système ou ses composants deviennent subitement inaccessibles. Utilisez-les pour implémenter un système d’avertissement anticipé.

1. Identifier les indicateurs de performance clés d’intégrité de votre application, telles que les exceptions temporaires et la latence de l’appel distant.
1. Définir des seuils à niveaux qui identifient les problèmes avant qu’ils deviennent critiques et exigent une réponse de récupération.
1. Envoyez une alerte à l’équipe chargée des opérations quand une valeur de seuil est atteinte.

Envisagez [Microsoft System Center 2016](https://www.microsoft.com/cloud-platform/system-center) ou des outils tiers pour fournir des fonctionnalités de surveillance. La plupart des solutions d’analyse suivent les principaux compteurs de performance et de disponibilité des services. [L’intégrité des ressources Azure](/azure/service-health/resource-health-checks-resource-types) fournit certains d’intégrité intégrés vérifications d’état, ce qui peuvent aider à diagnostiquer la limitation des services Azure.

## <a name="subscription-and-service-limitations"></a>Limitations de service et d’abonnement

Les abonnements Azure ont des limites sur certains types de ressources, comme le nombre de groupes de ressources, les noyaux et les comptes de stockage. Pour vous assurer que votre application ne s’exécute pas sur les limites d’abonnement Azure, créer des alertes qui interrogent de services arrivent à leurs limites et les quotas.

Résoudre les limites d’abonnement suivantes avec des alertes.

### <a name="individual-services"></a>Services individuels

Les services Azure individuels ont des limites de consommation de stockage, le débit, nombre de connexions, les demandes par seconde et les autres mesures. Votre application échoue si elle tente d’utiliser des ressources au-delà de ces limites, ce qui entraîne une interruption de limitation et possible du service.

Selon le service spécifique et les exigences de votre application, vous pouvez souvent rester sous ces limites par montée en puissance (en choisissant un autre niveau tarifaire, par exemple) ou la montée en charge (ajout de nouvelles instances).

### <a name="azure-storage-scalability-and-performance-targets"></a>Objectifs d’évolutivité et les performances de stockage Azure

Azure autorise un nombre maximal de comptes de stockage par abonnement. Si votre application nécessite plus de comptes de stockage actuellement disponibles dans votre abonnement, créez un nouvel abonnement avec les comptes de stockage supplémentaires. Pour plus d’informations, consultez [Abonnement Azure et limites, quotas et contraintes de service](/azure/azure-subscription-service-limits/#storage-limits).

Si vous dépassez les objectifs d’évolutivité et les performances de stockage Azure, votre application subira de limitation du stockage. Pour plus d’informations, consultez [Objectifs d’extensibilité et de performances de stockage Azure](/azure/storage/storage-scalability-targets/).

### <a name="scalability-targets-for-virtual-machine-disks"></a>Objectifs d'évolutivité pour les disques de machines virtuelles

Une infrastructure en tant que service (IaaS) de machine virtuelle Azure prend en charge l’attachement d’un nombre de disques de données, en fonction de plusieurs facteurs, notamment la taille de machine virtuelle et le type de compte de stockage. Si votre application dépasse les objectifs d’extensibilité pour les disques de machine virtuelle, approvisionnez des comptes de stockage supplémentaires et créez-y les disques de machine virtuelle. Pour plus d’informations, consultez [Objectifs d’extensibilité et de performances de stockage Azure](/azure/storage/storage-scalability-targets/#scalability-targets-for-virtual-machine-disks).

### <a name="vm-size"></a>Taille de la machine virtuelle

Si le processeur, mémoire, disque et e/s de vos machines virtuelles réel approchent des limites de la taille de machine virtuelle, votre application peut rencontrer des problèmes de capacité. Pour corriger les problèmes, augmentez la taille de machine virtuelle. Tailles de machine virtuelle sont décrits dans [tailles des machines virtuelles dans Azure](/azure/virtual-machines/virtual-machines-windows-sizes/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

Si votre charge de travail fluctue au fil du temps, envisagez l’utilisation de l’échelle de machine virtuelle définit automatiquement à l’échelle le nombre d’instances de machine virtuelle. Sinon, vous devez augmenter ou diminuer le nombre de machines virtuelles manuellement. Pour plus d’informations, consultez le [vue d’ensemble de machines virtuelles identiques](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview/).

### <a name="azure-sql-database"></a>Azure SQL Database

Si votre niveau de base de données SQL Azure n’est pas suffisante pour gérer les exigences d’unité de Transaction de base de données (DTU) de votre application, votre utilisation de données est limitée. Pour plus d’informations sur la sélection du plan de service approprié, consultez [modèles d’achat Azure SQL Database](/azure/sql-database/sql-database-service-tiers/).

## <a name="third-party-services"></a>Services tiers

Si votre application a des dépendances sur les services tiers, identifiez comment ces services peuvent échouer et ce que les échecs effet aura sur votre application.

Un service tiers n’inclure pas de surveillance et diagnostic. Consigner les appels à ces services et les mettre en corrélation avec l’intégrité et la journalisation des diagnostics à l’aide d’un identificateur unique de votre application. Pour plus d’informations sur des pratiques éprouvées pour la surveillance et diagnostics, consultez [Guide de surveillance et diagnostics](../best-practices/monitoring.md).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Plan de récupération d’urgence](./disaster-recovery.md)
