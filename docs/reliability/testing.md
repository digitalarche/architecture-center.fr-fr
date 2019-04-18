---
title: Tester la résilience et la disponibilité des applications Azure
description: Approches de test de la fiabilité des applications dans Azure
author: MikeWasson
ms.date: 04/10/2019
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.openlocfilehash: cf33a859540810279b1cb85be5a6fb2ebe4500dc
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59646551"
---
# <a name="testing-azure-applications-for-resiliency-and-availability"></a>Tester la résilience et la disponibilité des applications Azure

Pour tester la résilience, vous devez vérifier le fonctionnement de la charge de travail de bout en bout dans les conditions de défaillance intermittente.

Exécuter des tests en production à l’aide des données synthétiques et réelles de l’utilisateur. Test et production sont rarement identiques, il est donc important de valider votre application à l’aide de la production une [bleu-vert](https://martinfowler.com/bliki/BlueGreenDeployment.html) ou [déploiement canary](https://martinfowler.com/bliki/CanaryRelease.html). De cette façon, vous testez l’application dans les conditions réelles, afin de vous assurer qu’elle pourra fonctionner comme prévu quand le déploiement complet.

Dans le cadre de votre plan de test, sont les suivantes :

- Les aspects de tests automatisés
- Test d’injection d’erreur
- Test de charge de pointe
- Test de récupération d’urgence
- Test du service tiers

Les tests sont un processus itératif. Testez l’application, mesurez le résultat, analysez et résolvez tous les échecs résultants, puis répétez le processus.

## <a name="perform-fault-injection-testing"></a>Effectuer le test d’injection d’erreur

Pour le test d’injection d’erreur, vérifiez la résilience du système lors de pannes, en déclenchant des échecs réels ou en simulant les. Voici quelques stratégies de provoquer des défaillances :

- Arrêter les instances de machine virtuelle (VM).
- Blocage des processus.
- Expiration des certificats.
- Changement des clés d’accès.
- Arrêt du service DNS sur les contrôleurs de domaine.
- Limitation des ressources système disponibles, comme la RAM ou le nombre de threads.
- Démontage des disques.
- Redéploiement d’une machine virtuelle.

Votre plan de test doit incorporer des points de défaillance possibles identifiées pendant la phase de conception, en plus des scénarios courants de défaillance :

- Tester votre application dans un environnement aussi proche de production que possible.
- Échecs de test dans la combinaison.
- Mesurez les temps de récupération et vérifiez que les besoins de votre entreprise sont remplies.
- Vérifiez que les échecs n’en cascade et sont gérées de manière isolée.

Pour plus d’informations sur les scénarios d’échec, consultez [échec et récupération d’urgence pour les applications Azure](./disaster-recovery.md).

## <a name="test-under-peak-loads"></a>Tester sous les pics de charge

Test de charge est essentiel pour identifier les défaillances qui se produisent uniquement en charge, tels que la base de données back-end saturé ou la limitation du service. Test de charge maximale et augmentation anticipée des pics de charge à l’aide des données de production ou des données synthétiques qui soient aussi proches que possible, les données de production. Votre objectif est de voir comment l’application se comporte dans des conditions réelles.

## <a name="conduct-disaster-recovery-drills"></a>Effectuer des exercices de récupération d’urgence

Démarrez avec un plan de récupération d’urgence satisfaisante et tester régulièrement pour vérifier qu’elle fonctionne.

Pour les Machines virtuelles Azure, vous pouvez utiliser [Azure Site Recovery](/azure/site-recovery/azure-to-azure-quickstart/) pour répliquer et effectuer [exercices de récupération d’urgence](/azure/site-recovery/azure-to-azure-tutorial-dr-drill/) sans affecter les applications de production ni la réplication continue.

### <a name="failover-and-failback-testing"></a>Basculement et le test de la restauration automatique

Tester le basculement et la restauration pour vérifier que les services dépendants de votre application redeviendront actifs de manière synchronisée au cours de la récupération d’urgence. Modifications apportées aux systèmes et aux opérations susceptibles d’affecter les fonctions de basculement et la restauration automatique, mais l’impact peuvent ne pas être détecté jusqu'à ce que le système principal tombe en panne ou est surchargé. Tester les fonctionnalités de basculement *avant* ils sont requis afin de compenser un problème en direct. Assurez-vous également que les services dépendants basculent et restaurer dans l’ordre correct.

Si vous utilisez [Azure Site Recovery](/azure/site-recovery/) pour répliquer des machines virtuelles, exécutez après sinistre exercices de récupération régulièrement en effectuant des tests de basculement pour valider votre stratégie de réplication. Un test de basculement n’affecte pas la réplication de machine virtuelle en cours ou de votre environnement de production. Pour plus d’informations, consultez [exécuter une simulation de récupération d’urgence vers Azure](/azure/site-recovery/site-recovery-test-failover-to-azure).

### <a name="simulation-testing"></a>Le test de simulation

Test de simulation implique la création d’un petit et réelles. Simulations illustrent l’efficacité des solutions dans le plan de récupération et mettre en évidence les problèmes qui n’ont pas été traités correctement.

Quand vous effectuez le test de simulation, suivez les meilleures pratiques :

- Effectuez des simulations d’une manière qui ne perturbent l’activité réelle, mais semble une situation réelle.
- Assurez-vous que les scénarios simulés sont entièrement contrôlables. Si le plan de récupération a échoué, vous pouvez restaurer la situation normal sans provoquer des dommages.
- Informer de gestion sur quand et comment les exercices de simulation seront effectuées. Votre plan doit préciser l’intervalle de temps et les ressources impactés pendant la simulation.

## <a name="test-health-probes-for-load-balancers-and-traffic-managers"></a>Tester les sondes d’intégrité pour les équilibreurs de charge et les gestionnaires de trafic

Configurez et testez les sondes d’intégrité pour vos équilibreurs de charge et les gestionnaires de trafic. Assurez-vous que votre point de terminaison d’intégrité vérifie les composants critiques du système et répond correctement.

- Pour [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview/), la sonde d’intégrité détermine s’il faut basculer vers une autre région. Votre point de terminaison d’intégrité doit vérifier les dépendances critiques qui sont déployés au sein de la même région.
- Pour [Azure Load Balancer](/azure/load-balancer/load-balancer-overview/), la sonde d’intégrité détermine s’il faut supprimer une machine virtuelle à partir de la rotation. Le point de terminaison d’intégrité doit signaler l’intégrité de la machine virtuelle. N’incluez pas d’autres niveaux ou de services externes. Autrement, si un échec se produit en dehors de la machine virtuelle, l’équilibreur de charge supprimera la machine virtuelle de la rotation.

Pour savoir comment implémenter la surveillance de l’intégrité dans votre application, consultez [Modèle Surveillance de point de terminaison](../patterns/health-endpoint-monitoring.md).

## <a name="test-monitoring-systems"></a>Tester des systèmes de surveillance

Inclure des systèmes de surveillance dans votre plan de test. Systèmes automatisés de basculement et la restauration automatique varient selon le bon fonctionnement de la surveillance et l’instrumentation. Tableaux de bord pour visualiser les alertes d’intégrité et l’opérateur système dépendre également ayant suivi précis et l’instrumentation. Si ces éléments subissent des défaillances, laissent échapper des informations critiques ou signalent des données inexactes, l’opérateur peut ne pas avoir connaissance du défaut d’intégrité ou de la défaillance du système.

## <a name="include-third-party-services-in-test-scenarios"></a>Inclure des services tiers dans les scénarios de test

Si votre application a des dépendances sur les services tiers, identifier l’emplacement où et comment ces services peuvent échouer et les effets que ces échecs auront sur votre application. N’oubliez pas le contrat de niveau de service (SLA) pour le service tiers et l’effet qu'est peut-être sur votre plan de récupération d’urgence.

Un service tiers peut ne pas fournit des fonctionnalités de surveillance et de diagnostic, par conséquent, il est important de se connecter à vos appels de ces et de les mettre en corrélation avec l’intégrité de votre application et de diagnostic de journalisation à l’aide d’un identificateur unique. Pour plus d’informations sur des pratiques éprouvées pour la surveillance et diagnostics, consultez [Guide de surveillance et diagnostics](../best-practices/monitoring.md).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Déployer pour la résilience et disponibilité](./deploy.md)
