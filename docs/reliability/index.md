---
title: Concevoir des applications Azure fiables
description: Introduction à la conception d’applications Azure fiables et à haute disponibilité
author: MikeWasson
ms.date: 04/10/2019
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.custom: ''
ms.openlocfilehash: 16c1daeed9b1d79f33051eb69571f1574bf9be4d
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59644008"
---
# <a name="designing-reliable-azure-applications"></a>Conception d’applications Azure fiables

Concevoir une application fiable dans le cloud est un processus différent du développement d’applications traditionnelles. Par le passé, vous avez peut-être acheté du matériel plus performant pour augmenter votre puissance de traitement, selon une approche de scale-up. Dans un environnement cloud, cela se fait par un scale-out. Au lieu d’essayer d’empêcher toutes les défaillances, l’objectif est de réduire les répercussions d’une défaillance potentielle au niveau de chaque composant.

Les applications fiables présentent les caractéristiques suivantes :

- Elles sont **résilientes**, ce qui leur permet de reprendre correctement après une défaillance et de continuer à fonctionner avec un temps d’arrêt et une perte de données minimum avant la récupération complète.
- Elles offrent une **haute disponibilité** et s’exécutent comme prévu dans un état intègre et sans temps d’arrêt significatif.

Pour concevoir une application fiable, il est essentiel de bien comprendre comment ces éléments interagissent ainsi que leur impact sur les coûts. Vous pourrez alors mieux évaluer le temps d’arrêt acceptable, les répercussions financières potentielles pour votre entreprise et les fonctions requises pendant une récupération.

Cet article offre une vue d’ensemble des différentes étapes du processus de conception d’une application Azure qui doivent être effectuées dans une optique de fiabilité. Chaque section contient un lien vers un article qui explique plus en détail comment rendre l’application plus fiable à chaque étape du processus. Pour obtenir des informations sur la vérification de la fiabilité des différents services Azure, passez en revue la [liste de contrôle de la résilience des services Azure](../checklist/resiliency-per-service.md).

## <a name="build-for-reliability"></a>Concevoir dans un souci de fiabilité

Cette section décrit six étapes à respecter pour concevoir une application Azure fiable. Chaque étape renvoie à une section qui explique de manière plus précise le processus et les conditions.

1. [**Définissez les exigences.**](#define-requirements) Établissez les exigences de disponibilité et de récupération pour chaque charge de travail et besoin métier.
1. [**Respectez les bonnes pratiques en matière d’architecture.**](#use-architectural-best-practices) Suivez les pratiques éprouvées, identifiez les points de défaillance possibles dans l’architecture et déterminez de quelle façon l’application répondra aux défaillances.
1. [**Testez la conception avec des simulations et des basculements forcés.**](#test-with-simulations-and-forced-failovers) Simulez des erreurs et déclenchez des basculements forcés afin de tester la détection et la récupération après ces défaillances.
1. [**Déployez l’application de manière cohérente.**](#deploy-the-application-consistently) Mettez l’application en production au moyen de processus fiables et reproductibles.
1. [**Supervisez l’intégrité des applications.**](#monitor-application-health) Détectez les défaillances, supervisez les indicateurs de défaillances potentielles et évaluez l’intégrité de vos applications.
1. [**Planifiez le traitement des défaillances et des sinistres.**](#respond-to-failures-and-disasters) Identifiez les situations de défaillance et déterminez comment les traiter en fonction des stratégies établies.

## <a name="define-requirements"></a>Définir les conditions requises

Identifiez vos besoins métier et créez un plan de fiabilité approprié pour y répondre. Tenez compte des éléments suivants :

- **Identifiez les charges de travail et l’utilisation.** Une *charge de travail* est une fonctionnalité ou une tâche distincte qui est séparée logiquement des autres tâches sur les plans de la logique métier et des exigences de stockage des données. Chaque charge de travail a des exigences différentes concernant la disponibilité, la scalabilité, la cohérence des données et la reprise d’activité.
- **Planifiez les modèles d’utilisation.** Les *modèles d’utilisation* jouent également un rôle dans la définition des exigences. Identifiez les différences dans les exigences pendant les périodes critiques et non critiques. Par exemple, une application de déclaration de revenus ne doit pas échouer le jour de la date limite de la déclaration. Pour garantir la disponibilité de l’application, planifiez sa redondance dans plusieurs régions, en cas d’échec de l’une d’elles. À l’inverse, pour réduire les coûts durant les périodes non critiques, vous pouvez exécuter votre application dans une seule région.
- **Établissez des métriques de disponibilité&mdash; : le *temps moyen de récupération* (MTTR) et le *temps moyen entre défaillances* (MTBF).** Le MTTR est la durée moyenne nécessaire à la restauration d’un composant après une défaillance. Le MTBF indique la durée pendant laquelle un composant peut raisonnablement fonctionner entre deux pannes. Utilisez ces métriques pour déterminer à quelles étapes vous devez prévoir plus de redondance et quels contrats de niveau de service (SLA) conviennent pour les clients.  
- **Établissez des métriques de récupération&mdash; : l’objectif de délai de récupération (RTO) et l’objectif de point de récupération (RPO).** Le *RTO* est la durée maximale acceptable pendant laquelle une application peut rester indisponible après un incident. Le *RPO* est la durée maximale de perte de données acceptable pendant une panne. Pour déterminer ces valeurs, effectuez une évaluation des risques, en prenant en compte le coût et le risque que représente un temps d’arrêt ou une perte de données pour votre organisation.
    > [!NOTE]
    > Si le MTTR de *n’importe lequel* des composants critiques dans une configuration à haute disponibilité dépasse le RTO système, une défaillance du système risque d’entraîner une interruption inacceptable de l’activité. Vous ne pouvez alors pas restaurer le système dans l’objectif RTO défini.
- **Déterminez les objectifs de disponibilité des charges de travail.** Pour vous assurer que l’architecture de l’application répond à vos besoins métier, définissez des contrats SLA cibles pour chaque charge de travail. Prenez en compte le coût et la complexité de répondre aux exigences de disponibilité, en plus des dépendances d’application.
- **Soyez sûr de bien comprendre les contrats de niveau de service (SLA).** Dans Azure, un contrat SLA décrit les engagements de Microsoft en matière de disponibilité et de connectivité. Si le contrat SLA lié à un service particulier est de 99,9 pour cent, le service doit être disponible 99,9 pour cent du temps.  

    Définissez vos propres contrats SLA cibles pour chaque charge de travail dans votre solution, afin de déterminer si l’architecture répond aux besoins métier. Par exemple, si une charge de travail doit avoir une disponibilité de 99,99 pour cent, mais qu’elle dépend d’un service avec un contrat SLA de 99,9 pour cent, ce service ne peut pas être un point de défaillance unique dans le système.

Pour plus d’informations sur la définition des exigences pour des applications fiables, consultez [Définition des exigences pour des applications Azure résilientes](./requirements.md).

## <a name="use-architectural-best-practices"></a>Respecter les bonnes pratiques en matière d’architecture

Pendant la phase de la conception architecturale, veillez à implémenter des pratiques qui respectent les exigences de votre entreprise, identifient les points de défaillance et limitent l’impact des défaillances.

- **Effectuez une analyse du mode d’échec (FMA).** L’analyse du mode d’échec intègre la résilience dans une application dès le début de la phase de conception. Elle vous aide à identifier les types de défaillances que votre application peut rencontrer, les effets potentiels de chaque type de défaillance et les stratégies de récupération possibles.
- **Créez un plan de redondance.** Le niveau de redondance à prévoir pour chaque charge de travail dépend de vos besoins métier et des facteurs impactant le coût global de votre application. 
- **Concevez en pensant scalabilité.** Une application cloud doit pouvoir être redimensionnée pour s’adapter aux changements dans son utilisation. Commencez avec des composants discrets et concevez l’application pour qu’elle s’adapte automatiquement aux variations de charge chaque fois que possible. Au moment de la conception, gardez à l’esprit les limites du dimensionnement afin de pouvoir facilement étendre l’application plus tard.
- **Planifiez les abonnements et services nécessaires.** Vous devrez peut-être utiliser des abonnements supplémentaires afin de provisionner toutes les ressources nécessaires pour répondre aux besoins de votre entreprise en matière de stockage, de connectivité, de débit, etc. 
- **Utilisez l’équilibrage de charge pour distribuer les requêtes.** L’équilibrage de charge distribue les requêtes de votre application à des instances de service intègres, en retirant de la rotation les instances non intègres.
- **Implémentez des stratégies de résilience.** *La résilience* est la capacité d’un système à récupérer après des défaillances et à continuer de fonctionner. Implémentez des [modèles de conception de la résilience](../patterns/category/resiliency.md), comme l’isolation des ressources critiques, l’utilisation de transactions de compensation et l’exécution d’opérations asynchrones dès que possible.
- **Intégrez les exigences de disponibilité à votre conception.** La *disponibilité* est la proportion de temps durant laquelle le système est opérationnel et fonctionne. Prenez des mesures pour vous assurer que la disponibilité de l’application est conforme à votre contrat de niveau de service. Par exemple, évitez les points de défaillance uniques, décomposez les charges de travail par objectif de niveau de service et limitez les utilisateurs consommant beaucoup de ressources.
- **Gérez vos données.** Le choix des méthodes de stockage, de sauvegarde et de réplication des données est capital.

  - **Choisissez des méthodes de réplication appropriées pour les données de votre application.** Vos données d’application sont stockées dans divers magasins de données et peuvent avoir des exigences de disponibilité différentes. Évaluez les méthodes et emplacements de réplication pour chaque type de magasin de données afin de vous assurer qu’ils remplissent vos exigences.
  - **Documentez et testez vos processus de basculement et de restauration automatique.** Documentez clairement les instructions de basculement vers un nouveau magasin de données et vérifiez-les régulièrement pour vous assurer qu’elles sont toujours à jour et simples à suivre.
  - **Protégez vos données.** Sauvegardez et validez régulièrement les données, et assurez-vous qu’aucun compte d’utilisateur n’a accès à la fois aux données de production et aux données de sauvegarde.
  - **Planifiez la récupération des données.** Assurez-vous que votre stratégie de sauvegarde et de réplication présente des délais de récupération des données qui respectent les exigences de votre contrat de niveau de service. Prenez en compte tous les types de données que votre application utilise, y compris les bases de données et les données de référence.

Pour plus d’informations sur la conception de l’architecture d’applications fiables, consultez [Concevoir l’architecture des applications Azure dans une optique de résilience et de disponibilité](./architect.md).

## <a name="test-with-simulations-and-forced-failovers"></a>Tester la conception avec des simulations et des basculements forcés

Tester la fiabilité implique de mesurer les performances de la charge de travail de bout en bout dans des conditions d’échec qui se produisent seulement par intermittence.

- **Testez des scénarios de défaillances courants en déclenchant des défaillances réelles ou simulées.** Utilisez l’injection d’erreurs pour tester des scénarios courants (y compris des combinaisons de défaillances) et le délai de récupération.
- **Identifiez les défaillances qui se produisent uniquement en situation de charge importante.** Testez la charge maximale, en utilisant des données de production ou des données synthétiques qui soient aussi proches que possible des données de production. L’objectif est de voir comment se comporte l’application dans des conditions réelles.
- **Effectuez des essais de reprise d’activité après sinistre**. Mettez en place un plan de reprise d’activité après sinistre et testez-le régulièrement pour vous assurer qu’il fonctionne.
- **Effectuez des tests de basculement et de restauration automatique.** Assurez-vous que le basculement et la restauration automatique des services dépendants de votre application s’effectuent dans le bon ordre.
- **Effectuez des tests de simulation.** Tester des scénarios réels permet parfois de mettre en évidence certains problèmes devant être traités. Les scénarios doivent être contrôlables et non perturbateurs pour l’activité de l’entreprise. Informez la direction des plans de tests de simulation.
- **Testez les sondes d’intégrité.** Configurez des sondes d’intégrité afin que les équilibreurs de charge et les gestionnaires de trafic vérifient les composants système critiques. Testez-les pour vous assurer qu’elles répondent de manière appropriée.
- **Testez les systèmes de supervision.** Vérifiez que les systèmes de supervision rapportent les informations critiques et les données précises de manière fiable pour faciliter l’identification des défaillances potentielles.
- **Incluez les services tiers dans vos scénarios de test.** Testez les points de défaillance possibles en raison d’une interruption d’un service tiers, et testez également la récupération.

Les tests sont un processus itératif. Testez l’application, mesurez le résultat, analysez et résolvez les défaillances, puis répétez le processus.

Pour plus d’informations sur le test de la fiabilité des applications, consultez [Tester les applications Azure dans une optique de résilience et de disponibilité](./testing.md).

## <a name="deploy-the-application-consistently"></a>Déployer l’application de manière cohérente

Le *déploiement* inclut le provisionnement des ressources Azure, le déploiement du code de l’application ainsi que l’implémentation des paramètres de configuration. Une mise à jour peut impliquer ces trois tâches ou seulement certaines d’entre elles.

Après qu’une application a été déployée en production, les mises à jour deviennent une source possible d’erreurs. Limitez les erreurs en mettant en place des processus de déploiement prévisibles et reproductibles.

- **Automatisez votre processus de déploiement d’application.** Automatisez le plus de processus possible.
- **Concevez votre processus de mise en production de manière à optimiser la disponibilité.** Si votre processus de mise en production nécessite la mise hors connexion des services pendant le déploiement, votre application reste indisponible jusqu’à ce que ces services soient remis en ligne. Tirez parti des fonctionnalités de préproduction et de production de la plateforme. Utilisez des versions bleu/vert ou de contrôle de validité pour déployer des mises à jour. De cette façon, en cas de défaillance, vous pourrez rapidement restaurer la mise à jour.
- **Mettez en place un plan de restauration pour le déploiement.** Concevez un processus de restauration pour revenir à la dernière version valide connue et réduire ainsi le temps d’arrêt en cas d’échec d’un déploiement.
- **Journalisez et auditez les déploiements.** Si vous utilisez des techniques de déploiement en préproduction, plusieurs versions de votre application seront exécutées en production. Implémentez une stratégie de journalisation fiable pour capturer le plus d’informations possible concernant la version.
- **Documentez le processus de mise en production de l’application.** Définissez et documentez clairement votre processus de mise en production et assurez-vous que toute l’équipe chargée des opérations a accès à cette documentation.

Pour plus d’informations sur la fiabilité et le déploiement des applications, consultez [Déployer des applications Azure dans une optique de résilience et de disponibilité](./deploy.md).

## <a name="monitor-application-health"></a>Superviser l’intégrité de l’application

Implémentez les bonnes pratiques de supervision et d’alerte dans votre application afin de pouvoir facilement détecter les défaillances et avertir un technicien pour les corriger.

- **Implémentez des sondes d’intégrité et vérifiez le fonctionnement.** Exécutez les sondes régulièrement en dehors de l’application pour identifier une éventuelle dégradation des performances et de l’intégrité de l’application.
- **Vérifiez les workflows de longue durée.** Interceptez les problèmes tôt pour réduire le risque d’avoir à restaurer le workflow entier ou exécuter plusieurs transactions de compensation.
- **Conservez les journaux des applications.**

  - Journalisez les applications en production et au niveau des limites de service.
  - Utilisez la journalisation sémantique et asynchrone.
  - Séparez les journaux des applications des journaux d’audit.

- **Mesurez les statistiques des appels distants et partagez les résultats avec l’équipe travaillant sur l’application.** Pour donner à l’équipe des opérations une vue instantanée de l’intégrité de l’application, faites un récapitulatif des statistiques d’appels distants, comme la latence, le débit et les erreurs dans les centiles 99 et 95. Effectuez une analyse statistique sur les mesures pour détecter les erreurs qui se produisent dans chaque centile.
- **Suivez les exceptions temporaires et les nouvelles tentatives sur une période d’exécution pertinente.** Une augmentation des exceptions au fil du temps indique que le service rencontre un problème et risque d’échouer.
- **Configurez un système d’avertissement précoce.** Identifiez les indicateurs de performance clés (KPI) de l’intégrité de votre application, tels que les exceptions temporaires et la latence des appels distants, et définissez des valeurs de seuil appropriées pour chacun d’eux. Envoyez une alerte à l’équipe chargée des opérations quand une valeur de seuil est atteinte.
- **Tenez compte des limites d’abonnement Azure.** Les abonnements Azure présentent des limites pour certains types de ressources, par exemple le nombre de groupes de ressources, de cœurs et de comptes de stockage. Surveillez votre utilisation des types de ressources.
- **Surveillez les services tiers.** Journalisez vos appels et mettez-les en corrélation avec la journalisation de l’intégrité et des diagnostics de votre application à l’aide d’un identificateur unique.
- **Formez plusieurs opérateurs pour leur apprendre à superviser l’application et à effectuer manuellement les étapes de récupération.** Il doit toujours y avoir au moins un opérateur formé disponible.

Pour plus d’informations sur la supervision de la fiabilité des applications, consultez [Supervision de l’intégrité des applications Azure](./monitoring.md).

## <a name="respond-to-failures-and-disasters"></a>Planifier le traitement des défaillances et des sinistres

Créez un plan de récupération, en vous assurant qu’il couvre la restauration des données, les pannes de réseau, les défaillances des services dépendants et les interruptions de service à l’échelle de la région. Incluez les machines virtuelles, le stockage, les bases de données et les autres services de plateforme Azure dans votre stratégie de récupération.

- **Planifiez les interactions avec le support Azure.** Établissez en amont le processus à suivre pour contacter le support Azure.
- **Documentez et testez votre plan de reprise d’activité après sinistre.** Rédigez un plan de reprise d’activité après sinistre qui reflète l’impact financier des défaillances de l’application. Automatisez le plus possible le processus de reprise d’activité et documentez toutes les étapes manuelles. Testez régulièrement votre processus de récupération d’urgence pour valider et améliorer le plan.
- **Effectuez un basculement manuel quand cela est nécessaire.** Certains systèmes ne peuvent pas basculer automatiquement et nécessitent un basculement manuel. Si une application est basculée vers une région secondaire, effectuez un test de disponibilité opérationnelle. Vérifiez que la région principale est intègre et qu’elle est de nouveau prête à recevoir du trafic avant sa restauration automatique. Identifiez les fonctionnalités de l’application qui sont réduites et déterminez de quelle manière l’application doit informer les utilisateurs des problèmes temporaires.
- **Anticipez les défaillances possibles de l’application.** Anticipez les différents types de défaillances, comme les erreurs gérées automatiquement, celles qui entraînent une réduction de fonctionnalité et celles qui rendent l’application indisponible. L’application doit toujours informer les utilisateurs des problèmes temporaires.
- **Récupérez après une altération des données.** Si une défaillance se produit dans un magasin de données, recherchez les incohérences de données éventuelles quand le magasin redevient disponible, en particulier si les données ont été répliquées. Restaurez les données endommagées à partir d’une sauvegarde.
- **Récupérez après une panne du réseau.** Vous serez peut-être en mesure d’utiliser les données mises en cache pour exécuter l’application localement avec des fonctionnalités réduites. Si ce n’est pas le cas, envisagez d’arrêter l’application ou de basculer vers une autre région. Stockez vos données à un autre emplacement jusqu’au rétablissement de la connectivité.
- **Récupérez après une défaillance d’un service dépendant.** Déterminez quelle fonctionnalité est encore disponible et la façon dont l’application doit répondre.
- **Établissez une stratégie de reprise après une interruption de service à l’échelle de la région.** Même si les interruptions de service à l’échelle de la région sont rares, vous devez établir une stratégie de reprise, en particulier pour les applications critiques. Vous pouvez essayer de redéployer l’application vers une autre région ou de redistribuer le trafic.

Pour plus d’informations sur le traitement des défaillances et la reprise d’activité après sinistre, consultez [Défaillances et reprise d’activité après sinistre pour les applications Azure](./disaster-recovery.md).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Établir les exigences pour des applications résilientes](./requirements.md)
