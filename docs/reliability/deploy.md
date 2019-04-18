---
title: Déploiement d’applications Azure pour la résilience et disponibilité
description: Une vue d’ensemble des techniques pour créer des déploiements fiables dans Azure
author: MikeWasson
ms.date: 04/10/2019
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.openlocfilehash: d8df3fe1c3353c60462959a625ba13ae47b96cf8
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59646554"
---
# <a name="deploying-azure-applications-for-resiliency-and-availability"></a>Déploiement d’applications Azure pour la résilience et disponibilité

Lorsque vous configurez et mettre à jour des ressources Azure, code d’application et les paramètres de configuration, un processus reproductible et prévisible vous aidera à éviter des erreurs et des temps d’arrêt. Nous vous recommandons les processus automatisés de déploiement que vous pouvez exécuter à la demande et exécutez à nouveau en cas de problème. Une fois que vos processus de déploiement sont en cours d’exécution sans heurts, documentation sur le processus peut conservez-les de cette façon.

## <a name="automate-processes"></a>Automatiser les processus

Pour activer des ressources à la demande, déployer des solutions rapidement, réduire les erreurs humaines et produire des résultats cohérents et reproductibles, veillez à automatiser les déploiements et mises à jour.

### <a name="automate-as-many-processes-as-possible"></a>Automatiser les processus autant que possible

Les processus de déploiement plus fiables sont automatisées et *idempotent* &mdash; , autrement dit, reproductibles pour produire les mêmes résultats.

- Pour automatiser l’approvisionnement des ressources Azure, vous pouvez utiliser [Terraform](/azure/virtual-machines/windows/infrastructure-automation#terraform), [Ansible](/azure/virtual-machines/windows/infrastructure-automation#ansible), [Chef](/azure/virtual-machines/windows/infrastructure-automation#chef), [Puppet](/azure/virtual-machines/windows/infrastructure-automation#puppet), [Azure PowerShell](/powershell/azure/overview), [interface de ligne de commande Azure (CLI)](/cli/azure), ou [modèles Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview#template-deployment).
- Pour configurer des machines virtuelles, vous pouvez utiliser [cloud-init](/azure/virtual-machines/windows/infrastructure-automation#cloud-init) (pour les machines virtuelles Linux) ou [Configuration d’état Azure Automation](/azure/automation/automation-dsc-overview) (DSC).
- Pour automatiser le déploiement d’applications, vous pouvez utiliser [Azure DevOps Services](/azure/virtual-machines/windows/infrastructure-automation#azure-devops-services), [Jenkins](/azure/virtual-machines/windows/infrastructure-automation#jenkins), ou d’autres solutions de CI/CD.

Comme meilleure pratique, créez un référentiel de scripts d’automatisation par catégorie pour un accès rapide, documentée avec des explications des paramètres et des exemples d’utilisation de script. Synchroniser cette documentation avec vos déploiements Azure et désigner une personne principale pour gérer le référentiel.

Scripts d’automatisation peuvent également activer des ressources à la demande de récupération d’urgence.

### <a name="use-code-to-provision-and-configure-infrastructure"></a>Utiliser le code pour approvisionner et configurer l’infrastructure

Cette pratique appelée *infrastructure en tant que code,* peut utiliser une approche déclarative ou une approche impérative (ou une combinaison des deux).

- [Modèles Resource Manager](/azure/azure-resource-manager/resource-group-overview#template-deployment) sont un exemple d’une approche déclarative.
- [PowerShell](/powershell/azure/overview) scripts sont un exemple d’approche impérative.

### <a name="practice-immutable-infrastructure"></a>Infrastructure immuable pratique

En d’autres termes, ne modifiez pas infrastructure après son déploiement en production. Une fois les modifications ad hoc ont été appliquées, vous ne savez pas exactement ce qui a changé, il peut être difficile à dépanner le système.

### <a name="automate-and-test-deployment-and-maintenance-tasks"></a>Automatisez et testez les tâches de déploiement et maintenance

Les applications distribuées se composent de plusieurs parties qui doivent fonctionner ensemble. Déploiement doit tirer parti des mécanismes ayant fait leurs preuves, telles que les scripts, ce qui peuvent mettre à jour et valider la configuration et pour automatiser le processus de déploiement. Testez tous les processus entièrement pour vous assurer que les erreurs ne provoquent des temps d’arrêt supplémentaires.

### <a name="implement-deployment-security-measures"></a>Implémenter des mesures de sécurité de déploiement

Tous les outils de déploiement doivent intégrer des restrictions de sécurité pour protéger l’application déployée. Définir et appliquer des stratégies de déploiement avec soin et minimiser la nécessité d’une intervention humaine.

## <a name="deploy-to-multiple-regions-and-instances"></a>Déployer sur plusieurs régions et les instances

Une application qui repose sur une seule instance d’un service crée un point de défaillance unique. Pour améliorer la résilience et l’évolutivité, configurez plusieurs instances.

- Pour [Azure App Service](/azure/app-service/app-service-value-prop-what-is/), sélectionnez un [plan App Service](/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview/) qui offre plusieurs instances.
- Pour [Azure Cloud Services](/azure/cloud-services/cloud-services-choose-me), configurez chacun de vos rôles pour utiliser [plusieurs instances](/azure/cloud-services/cloud-services-choose-me/#scaling-and-management).
- Pour [Machines virtuelles](/azure/virtual-machines/virtual-machines-windows-about/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json), vérifiez que votre architecture inclut plusieurs machines virtuelles et que chaque machine virtuelle est incluse dans un [à haute disponibilité](/azure/virtual-machines/virtual-machines-windows-manage-availability/).

### <a name="consider-deploying-across-multiple-regions"></a>Envisagez de déployer dans plusieurs régions

Nous vous recommandons de déployer toutes, mais les applications moins critiques et les services d’application dans plusieurs régions. Si votre application est déployée sur une seule région, dans les rares cas que la région entière deviendrait indisponible, l’application sera également être indisponible.

Si vous choisissez de déployer à une seule région, envisagez de préparation de la redéployer vers une région secondaire en réponse à une défaillance inattendue.

### <a name="redeploy-to-a-secondary-region"></a>Redéployer vers une région secondaire

Si vous exécutez des applications et bases de données dans une région unique, primaire sans aucune réplication, votre stratégie de récupération peut consister à redéployer vers une autre région. Cette solution est abordable, mais plus approprié pour les applications non critiques qui peuvent tolérer des temps de récupération.

Si vous choisissez cette stratégie, automatiser le processus de redéploiement autant que possible et incluent des scénarios de redéploiement dans votre test de réponse d’urgence.

Pour automatiser votre processus de redéploiement, envisagez d’utiliser [Azure Site Recovery](/azure/site-recovery/).

## <a name="use-staging-and-production-environments"></a>Utiliser des environnements intermédiaire et de production

Avec bon usage des environnements intermédiaire et de production, vous pouvez envoyer des mises à jour à l’environnement de production de manière hautement contrôlée et limiter les perturbations de problèmes de déploiement imprévus.

- [*Déploiement bleu-vert*](https://martinfowler.com/bliki/BlueGreenDeployment.html) implique le déploiement d’une mise à jour dans un environnement de production qui est distinct de l’application en direct. Après avoir validé le déploiement, passez le routage du trafic vers la version mise à jour. Une façon de faire cela consiste à utiliser le [emplacements intermédiaires](/azure/app-service/web-sites-staged-publishing) disponibles dans Azure App Service pour planifier un déploiement avant de passer à la production.
- [*Versions « Canary »*](https://martinfowler.com/bliki/CanaryRelease.html) sont similaires aux déploiements bleu-vert. Au lieu de basculer tout le trafic à l’application mise à jour, vous acheminez uniquement une petite partie du trafic vers le nouveau déploiement. S’il existe un problème, revenir à l’ancien déploiement. Si ce n’est pas le cas, Router progressivement plus de trafic vers la nouvelle version. Si vous utilisez Azure App Service, vous pouvez utiliser le test dans la fonctionnalité de production pour gérer une version de contrôle de validité.

## <a name="create-a-rollback-plan-for-deployment-and-updates"></a>Créer un plan de restauration pour le déploiement et les mises à jour

Si un déploiement échoue, votre application peut être indisponible. Pour réduire le temps mort, concevoir un processus de restauration pour revenir à une version de bonne connu. Inclure une stratégie pour restaurer les modifications pour les bases de données et d’autres services que dépend de votre application.

Si vous utilisez Azure App Service, vous pouvez configurer un emplacement connu bon site et l’utiliser pour restaurer à partir d’un site ou d’un déploiement de l’application API.

## <a name="log-and-audit-deployments"></a>Déploiements de journaux et d’audit

Pour capturer en tant que beaucoup spécifique à la version d’informations que possible, implémentez une stratégie de journalisation fiable. Si vous utilisez des techniques de déploiement intermédiaire, plusieurs versions de votre application seront exécuteront en production. Si un problème se produit, de déterminer quelle version est le responsable.

## <a name="document-release-processes"></a>Processus de mise en production de documents

Sans documentation du processus de publication détaillées, un opérateur peut déployer une mise à jour incorrecte ou mal peut configurer les paramètres de votre application. Définissez et documentez clairement votre processus de mise en production et assurez-vous que toute l’équipe chargée des opérations a accès à cette documentation.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Surveiller l’intégrité d’application pour la fiabilité](./monitoring.md)
