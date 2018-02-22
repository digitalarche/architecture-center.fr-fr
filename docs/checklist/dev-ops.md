---
title: "Liste de contrôle DevOps"
description: "Liste de contrôle qui fournit des conseils liés à DevOps."
author: dragon119
ms.date: 01/10/2018
ms.custom: checklist
ms.openlocfilehash: c435ea0aed9571cb6508d7d23f93414a138998fe
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/14/2018
---
# <a name="devops-checklist"></a>Liste de contrôle DevOps

Le DevOps consiste à intégrer le développement, l’assurance qualité et les opérations informatiques dans une culture unifiée et dans un ensemble de processus pour la distribution de logiciels. Utilisez cette liste de contrôle comme point de départ pour évaluer votre culture et vos processus DevOps.

## <a name="culture"></a>Culture

**Vérifiez l’alignement des objectifs au sein des organisations et des équipes.** Les conflits au sujet des ressources, des objectifs et des priorités au sein d’une organisation peuvent compromettre la réussite des opérations. Assurez-vous que les objectifs des équipes commerciales, de développement et des opérations sont alignés.

**Assurez-vous que toute l’équipe comprend bien le cycle de vie du logiciel.** Votre équipe doit comprendre le cycle de vie complet de l’application et savoir dans quelle phase de son cycle de vie l’application se trouve. Cela permet à tous les membres de l’équipe de savoir ce qu’ils doivent faire maintenant et ce qu’ils doivent planifier et préparer pour plus tard.

**Réduisez la durée du cycle.** Essayez de réduire le temps nécessaire pour passer de l’idée d’origine au logiciel fini. Limitez la taille et la portée des différentes versions pour réduire la charge de travail imposée aux équipes de test. Automatisez aussi souvent que possible les processus de génération, de test, de configuration et de déploiement. Supprimez tous les obstacles à la communication entre les développeurs et entre les développeurs et l’équipe chargée des opérations. 

**Révisez et améliorez les processus.** Vos processus et procédures, qu’ils soient automatisés ou manuels, ne sont jamais définitifs. Prévoyez des révisions régulières du workflow, des procédures et de la documentation existants, avec pour objectif de les améliorer.

**Anticipez.** Anticipez les pannes. Mettez en place des processus pour identifier rapidement les problèmes lorsqu’ils se produisent, pour les faire remonter aux membres de l’équipe chargée de les résoudre et pour confirmer leur résolution.

**Exploitez les informations issues des pannes.** Les pannes sont inévitables, mais il est important de savoir en tirer des informations pour éviter qu’elles ne se répètent. En cas de défaillance opérationnelle, triez le problème, documentez sa cause et sa solution et partagez les leçons que vous en tirez. Quand c’est possible, mettez à jour votre processus de génération pour détecter automatiquement ce type de défaillance à l’avenir.

**Accélérez vos processus et collectez des données.** Chaque amélioration planifiée est une hypothèse. Faites des incréments aussi petits que possible. Considérez les nouvelles idées comme des expériences. Instrumentez les expériences afin de pouvoir collecter des données de production pour évaluer leur efficacité. Préparez-vous à effectuer un Fail-fast si l’hypothèse est incorrecte.

**Prenez le temps d’apprendre.** Vous pouvez apprendre de vos échecs aussi bien que de vos réussites. Avant de passer à de nouveaux projets, prenez le temps de tirer les leçons importantes de vos résultats et assurez-vous que votre équipe les intègre correctement. Donnez aussi à votre équipe le temps d’acquérir de nouvelles compétences, d’expérimenter et d’en apprendre plus sur les nouveaux outils et techniques. 

**Documentez vos opérations** Documentez tous les outils, les processus et les tâches automatisées avec le même niveau de qualité que le code de votre produit. Documentez la conception et l’architecture actuelles de tous les systèmes que vous prenez en charge, ainsi que les processus de récupération et les autres procédures de maintenance. Concentrez-vous sur les étapes que vous effectuez réellement et non sur les processus théoriquement optimaux. Révisez et mettez à jour régulièrement votre documentation. Pour le code, assurez-vous d’inclure des commentaires pertinents, en particulier dans les API publiques, et utilisez autant que possible des outils pour générer automatiquement la documentation du code. 

**Partagez vos connaissances.** La documentation n’est utile que si les utilisateurs connaissent son existence et savent où la trouver. Vérifiez que la documentation est bien organisée et facile à trouver. Soyez créatif : partagez votre documentation de façon informelle lors d’un déjeuner, créez des vidéos ou envoyez des bulletins d’informations.

## <a name="development"></a>Développement

**Offrez aux développeurs des environnements qui reproduisent l’environnement de production.** Si les environnements de développement et de test ne correspondent pas à l’environnement de production, il est difficile de tester et de diagnostiquer les problèmes. Par conséquent, essayez de fournir des environnements de développement et de test aussi similaires que possible à l’environnement de production. Assurez-vous que les données de test sont cohérentes avec les données utilisées en production, même s’il s’agit d’exemples de données et non de vraies données de production (pour des raisons de confidentialité ou de conformité). Prévoyez de générer et d’anonymiser des données de test.

**Assurez-vous que tous les membres de l’équipe autorisés peuvent approvisionner l’infrastructure et déployer l’application.** La configuration de ressources similaires à celles utilisées en production et le déploiement de l’application ne doivent pas impliquer de tâches manuelles complexes ou des connaissances techniques détaillées sur le système. Toute personne disposant des autorisations appropriées doit être en mesure de créer ou de déployer des ressources similaires à celles utilisées en production sans passer par l’équipe chargée des opérations. 

> Cette recommandation ne signifie pas que tout le monde peut envoyer des mises à jour automatiques pour le déploiement en production. Elle a pour but de réduire les difficultés que les équipes de développement et d’assurance qualité rencontrent pour créer des environnements qui reproduisent l’environnement de production.

**Instrumentez l’application pour obtenir des informations.** Pour vérifier l’intégrité de votre application, vous devez savoir comment elle s’exécute et pouvoir reconnaître les erreurs ou les problèmes qui surviennent. Il faut toujours considérer l’instrumentation comme une exigence de conception et l’intégrer dans l’application dès le début. L’instrumentation doit inclure la journalisation des événements pour l’analyse de la cause racine, mais également des données de télémétrie et des métriques pour surveiller l’intégrité globale et l’utilisation de l’application.

**Surveillez votre dette technique.** Dans de nombreux projets, le calendrier de mise en production devient, à un certain degré, prioritaire sur la qualité du code. Essayez de déterminer quand cela se produit et prenez note de tous les raccourcis et autres implémentations non optimales afin de planifier du temps à l’avenir pour réexaminer ces problèmes.

**Envisagez d’envoyer des mises à jour directement en production.** Pour réduire la durée globale du cycle de mise en production, envisagez d’envoyer la validation des codes dument testés directement en production. Utilisez les [bascules de fonctionnalité][feature-toggles] pour contrôler l’activation des fonctionnalités. Cela vous permet de passer rapidement du développement à la mise en production, en utilisant les bascules pour activer ou désactiver des fonctionnalités. Les bascules sont également utiles lors de l’exécution de tests tels que les [versions de contrôle de validité][canary-release], où une fonctionnalité particulière est déployée sur un sous-ensemble de l’environnement de production.

## <a name="testing"></a>Test

**Automatisez les tests.** Le test manuel des logiciels est une tâche fastidieuse et sujette aux erreurs. Automatisez les tâches de test communes et intégrez les tests dans vos processus de génération. Les tests automatisés garantissent la cohérence et la reproductibilité des tests. Les tests de l’interface utilisateur intégrée doivent également être effectués par un outil automatisé. Azure propose des ressources de développement et de test qui peuvent vous aider à configurer et exécuter vos tests. Pour plus d’informations, consultez la page [Développement et test][dev-test].

**Test de détection des défaillances.** Si un système ne peut pas se connecter à un service, comment réagit-il ? Peut-il reprendre son fonctionnement une fois le service à nouveau disponible ? Intégrez systématiquement des tests par injection d’erreurs à la phase de révision dans les environnements de test et intermédiaires. Lorsque vos processus et pratiques de test sont bien rodés, envisagez d’exécuter ces tests en production. 

**Testez en production.** Le processus de mise en production ne s’arrête pas avec le déploiement en production. Mettez en place des tests pour vérifier que le code déployé fonctionne comme prévu. Pour les déploiements rarement mis à jour, intégrez des tests en production réguliers à votre procédure de maintenance.

**Automatisez les tests de performance afin d’identifier rapidement les problèmes de performances.** L’impact d’un grave problème de performances peut être aussi important que celui d’un bogue dans le code. Si les tests fonctionnels automatisés peuvent détecter les bogues applicatifs, il arrive qu’ils ne détectent pas les problèmes de performances. Il est donc important de définir des objectifs de performances acceptables pour les métriques telles que la latence, le temps de chargement et l’utilisation des ressources. Pensez à inclure des tests de performance automatisés dans votre pipeline de mise en production, pour vous assurer que l’application répond à ces objectifs.

**Effectuez un test de capacité.** Une application peut fonctionner parfaitement dans des conditions de test, puis avoir des problèmes une fois en production à cause du changement d’échelle ou des limitations de ressources. Veillez à toujours définir les limites maximales de capacité et d’utilisation attendues. Effectuez des tests pour vérifier que l’application peut gérer ces limites et pour savoir ce qu’il se passe lorsque ces limites sont dépassées. Il est conseillé d’effectuer des tests de capacité à intervalles réguliers.

Après la mise en production initiale, vous devez effectuer des tests de capacité et de performance à chaque mise à jour du code de production. Utilisez les données de l’historique pour affiner les tests et pour déterminer les types de tests à exécuter.

**Effectuez des tests de pénétration automatisés.** La sécurité de votre application est aussi importante que le bon fonctionnement de ses fonctionnalités. Intégrez des tests de pénétration automatisés à vos processus de génération et de déploiement. Planifiez régulièrement des tests de sécurité et des analyses de vulnérabilité sur les applications déployées, et surveillez les ports ouverts, les points de terminaison et les attaques. Les tests automatisés ne vous dispensent pas d’effectuer des révisions de sécurité approfondies à intervalles réguliers.

**Effectuez des tests automatisés de continuité de l’activité.** Développez des tests pour vérifier la continuité de l’activité à grande échelle, notamment le basculement et la récupération des sauvegardes. Configurez des processus automatisés pour effectuer ces tests régulièrement.

## <a name="release"></a>Libérer

**Automatisez les déploiements.** Automatisez le déploiement des applications pour les environnements de test, intermédiaires et de production. L’automatisation accélère les déploiements et augmente leur fiabilité tout en garantissant leur cohérence sur tous les environnements pris en charge. Elle supprime le risque d’erreur humaine due aux déploiements manuels. Elle simplifie aussi la planification de la mise en production, afin de minimiser les effets des éventuels temps d’arrêt.

**Utilisez l’intégration continue.** L’intégration continue (CI) consiste à fusionner le code du développeur dans un codebase centralisé selon un planning régulier, puis à effectuer automatiquement les processus de génération et de test. L’intégration continue permet à une équipe tout entière de travailler sur un codebase en même temps sans générer de conflits. Elle permet également de détecter plus rapidement les erreurs de code. Idéalement, le processus d’intégration continue doit s’exécuter chaque fois que le code est validé ou archivé. Au minimum, il doit s’exécuter une fois par jour.

> Vous pouvez envisager l’adoption d’un [modèle de développement « trunk »][trunk-based]. Dans ce modèle, les développeurs n’effectuent leurs validations que sur une seule branche (le « trunk »). Les validations ne doivent jamais arrêter la génération. Ce modèle facilite l’intégration continue, car tout le travail sur les fonctionnalités est effectué sur le « trunk » et les conflits de fusion sont ensuite résolus lors de la validation.

**Envisagez la livraison continue.** La livraison continue (CD) consiste à s’assurer que le code est toujours prêt à être déployé, en automatisant la génération, le test et le déploiement du code dans des environnements similaires à l’environnement de production. L’intégration de la livraison continue pour créer un pipeline de CI/CD complet vous permet de détecter plus rapidement les erreurs de code et d’assurer la mise en production rapide des mises à jour dument testées.

> Le *déploiement* continu est un processus supplémentaire qui déploie automatiquement en production toutes les mises à jour qui ont traversé le pipeline de CI/CD. Le déploiement continu requiert un processus de test robuste et automatisé ainsi qu’un processus de planification avancé. Il ne convient pas forcément à toutes les équipes.

**Effectuez de petites modifications incrémentielles.** Plus la modification apportée au code est importante, plus vous risquez d’introduire des bogues. Privilégiez les petites modifications. Cela limite les effets potentiels de chaque modification et simplifie la compréhension et le débogage des problèmes.

**Gardez le contrôle sur la publication des modifications.** Gardez le contrôle sur la publication des mises à jour pour vos utilisateurs finaux. Vous pouvez utiliser des bascules de fonctionnalité pour décider de l’activation des fonctionnalités pour les utilisateurs finaux.

**Implémentez des stratégies de gestion des mises en production afin de réduire les risques liés au déploiement.** Le déploiement d’une mise à jour applicative en production implique toujours des risques. Pour les réduire au maximum, vous pouvez recourir à des stratégies telles que les [versions de contrôle de validité][canary-release] ou les [déploiements bleu-vert][blue-green] pour déployer des mises à jour dans un sous-ensemble d’utilisateurs. Vérifiez que la mise à jour fonctionne comme prévu, puis déployez la mise à jour sur le reste du système.

**Documentez toutes les modifications.** Les modifications de configuration et les mises à jour mineures peuvent vite devenir sources de confusion et de conflit de version. C’est pourquoi il est important de toujours conserver une trace de toutes les modifications, quelle que soit leur importance. Prenez note de tout ce qui a été modifié, y compris les correctifs et les modifications apportées aux stratégies et à la configuration. (Évitez d’inclure des données sensibles dans ces notes. Par exemple, vous pouvez noter que des informations d’identification ont été mises à jour et indiquer le nom de la personne qui a apporté la modification, mais ne précisez pas de quelles informations d’identification il s’agit.) Ces notes sur les modifications doivent être accessibles à toute l’équipe. 

**Automatisez les déploiements.** Automatisez tous les déploiements et mettez en place des systèmes pour détecter les problèmes lors du lancement. Prévoyez un processus d’atténuation pour conserver le code et les données qui existent en production, avant que la mise à jour ne les remplace dans toutes les instances de production. Mettez en place un moyen de restaurer les correctifs par progression ou de restaurer les modifications.

**Envisagez une infrastructure immuable.** L’infrastructure immuable est le principe selon lequel vous ne devez pas modifier l’infrastructure après son déploiement en production. Si vous le faites, vous risquez de parvenir à un état dans lequel les modifications ad hoc ont été appliquées. Il est alors difficile de savoir avec exactitude ce qui a changé. Une infrastructure immuable fonctionne en remplaçant des serveurs entiers lors de tout nouveau déploiement. Ainsi, le code et l’environnement d’hébergement peuvent être testés et déployés en bloc. Une fois déployés, les composants de l’infrastructure ne sont plus modifiés jusqu’à la nouvelle version et au nouveau cycle de déploiement. 

## <a name="monitoring"></a>Surveillance

**Améliorez la visibilité sur vos systèmes.** L’équipe chargée des opérations doit toujours bénéficier d’une bonne visibilité sur l’intégrité et sur l’état d’un système ou d’un service. Vous pouvez définir des points de terminaison d’intégrité externes pour contrôler l’état et vérifier que les applications sont codées de façon à instrumenter les métriques opérationnelles. Utilisez un schéma commun et cohérent qui vous permet de mettre en corrélation les événements entre les systèmes. [Azure Diagnostics][azure-diagnostics] et [Application Insights][app-insights] représentent la méthode standard pour le suivi de l’intégrité et de l’état des ressources Azure. Microsoft [Operation Management Suite][oms] fournit également des fonctions de surveillance et de gestion centralisées pour les solutions cloud ou hybrides.

**Agrégez et mettez en corrélation les métriques et journaux**. Un système de télémétrie correctement instrumenté fournit une grande quantité de données de performances brutes ainsi que des journaux d’événements. Assurez-vous que les données de télémétrie et les données issues des journaux sont traitées et mises en corrélation rapidement, afin que l’équipe chargée des opérations puisse toujours avoir un aperçu pertinent de l’intégrité du système. Organisez et affichez les données de manière à obtenir une vue cohérente de tous les problèmes, afin que vous puissiez clairement savoir lorsque plusieurs événements sont liés.

> Consultez la stratégie de rétention de votre entreprise pour connaître les exigences en matière de traitement des données et de durée de stockage. 

**Implémentez des alertes et notifications automatiques.** Configurez des outils de surveillance tels qu’[Azure Monitor][azure-monitor] pour détecter les modèles et les conditions qui indiquent des problèmes potentiels ou actuels et envoyer des alertes aux membres de l’équipe chargée de résoudre les problèmes. Vous pouvez configurer vos alertes pour éviter les faux positifs.

**Surveillez le délai d’expiration des ressources.** Certaines ressources, comme les certificats, expirent au bout d’un certain temps. Assurez-vous de suivre les dates d’expiration des ressources et de savoir quels services ou fonctionnalités en dépendent. Utilisez des processus automatisés pour surveiller ces ressources. Informez l’équipe chargée des opérations avant l’expiration d’une ressource et réaffectez-la si son expiration risque de perturber l’application.

## <a name="management"></a>gestion

**Automatisez les tâches opérationnelles.** L’exécution manuelle des processus opérationnels répétitifs est sujette aux erreurs. Automatisez ces tâches aussi souvent que possible pour assurer la qualité et des exécutions cohérentes. Le code qui implémente l’automation doit être versionné dans le contrôle de code source. Comme pour tous les autres codes, les outils d’automatisation doivent être testés.

**Adoptez une approche d’approvisionnement de type infrastructure en tant que code.** Réduisez le nombre de configurations manuelles nécessaires à l’approvisionnement des ressources. À la place, utilisez des scripts et des modèles [Azure Resource Manager][resource-manager]. Conservez les scripts et les modèles dans le contrôle de code source, comme tous les autres codes que vous gérez. 

**Envisagez l’utilisation de conteneurs.** Les conteneurs fournissent une interface standard basée sur le package pour déployer des applications. Avec les conteneurs, une application est déployée à l’aide des packages qu’elle contient. Ceux-ci incluent des logiciels, les dépendances et les fichiers nécessaires pour exécuter l’application, ce qui simplifie considérablement le processus de déploiement. 

Les conteneurs créent également une couche d’abstraction entre l’application et le système d’exploitation sous-jacent. Vous bénéficiez ainsi d’environnements cohérents. Cette couche d’abstraction permet aussi d’isoler un conteneur des autres processus ou applications qui s’exécutent sur un ordinateur hôte. 

**Implémentez des stratégies de résilience et la réparation spontanée.** La résilience d’une application est sa capacité à récupérer après une défaillance. Exemples de stratégies pour assurer la résilience : relancer les échecs temporaires, basculer vers une instance secondaire ou même une autre région. Pour plus d’informations, consultez l’article [Conception d’applications résilientes pour Azure][resiliency]. Instrumentez vos applications afin que les problèmes soient signalés immédiatement et que vous puissiez gérer les pannes ou autres défaillances du système.

**Compilez un manuel des opérations.** Un manuel des opérations, ou *runbook*, documente les procédures et les informations de gestion nécessaires pour aider l’équipe chargée des opérations à mettre à jour un système. Documentez-y également tous les scénarios d’opérations et les plans d’atténuation qui peuvent entrer en jeu lors d’une défaillance ou de toute autre interruption de votre service. Créez cette documentation pendant le processus de développement et assurez-vous de la tenir à jour par la suite. Ce document doit être vérifié, testé et amélioré régulièrement selon l’évolution de vos processus. 

Il est essentiel de partager cette documentation. Encouragez les membres de votre équipe à collaborer et à partager leurs connaissances. Toute l’équipe doit avoir accès à la documentation. Faites en sorte que tous les membres de l’équipe puissent mettre à jour ces documents très facilement.

**Documentez les procédures d’appel.** Assurez-vous que toutes les procédures, planifications et tâches d’appel sont documentées et partagées avec tous les membres de l’équipe. Mettez à jour ces informations aussi souvent que possible.

**Procédures d’escalade de document pour les dépendances tierces.** Si votre application dépend de services tiers externes que vous ne contrôlez pas directement, vous devez prévoir un plan pour gérer les pannes. Créez une documentation pour vos processus d’atténuation planifiés. N’oubliez pas d’y inclure les informations de contact du support technique et les chemins d’escalade.

**Utilisez la gestion de la configuration.** Les modifications de configuration doivent être planifiées, visibles pour l’équipe chargée des opérations et enregistrées. Cette opération peut prendre la forme d’une base de données de gestion de la configuration ou d’une approche de type configuration en tant que code. La configuration doit être auditée régulièrement pour vérifier que ce que vous aviez demandé a réellement été mis en place.

**Prévoyez un plan de support Azure et étudiez le processus.** Azure offre plusieurs [plans de support][azure-support-plans]. Identifiez le plan qui correspond à vos besoins et assurez-vous que toute votre équipe sait comment l’utiliser. Chaque membre de l’équipe doit comprendre tous les détails du plan, comment fonctionne le processus de support et comment ouvrir un ticket de support avec Azure. Si vous prévoyez un événement à grande échelle, le support Azure peut vous aider à augmenter les limites de votre service. Pour plus d’informations, consultez le [Forum Aux Questions sur le support technique Azure](https://azure.microsoft.com/en-us/support/faq/).

**Suivez les principes de séparation des privilèges lorsque vous octroyez un accès aux ressources.** Gérez prudemment l'accès aux ressources. L’accès doit être refusé par défaut, sauf si un utilisateur obtient explicitement l’accès à une ressource. N’accordez aux utilisateurs que les accès dont ils ont besoin pour effectuer leurs tâches. Effectuez un suivi des privilèges accordés aux utilisateurs et réalisez régulièrement des audits de sécurité.

**Utilisez le contrôle d’accès en fonction du rôle.** L’assignation des comptes utilisateur et des accès aux ressources ne doit pas être un processus manuel. Utilisez le [contrôle d’accès en fonction du rôle][rbac] pour accorder les accès en fonction des identités et des groupes [Azure Active Directory][azure-ad]. 

**Utilisez un système de suivi des bogues pour suivre les problèmes.** Si vous ne disposez pas d’un bon moyen pour effectuer le suivi des problèmes, vous risquez d’égarer des éléments, de dupliquer certaines tâches ou même de provoquer des problèmes supplémentaires. Ne vous fiez pas aux communications orales informelles pour suivre l’état des bogues. Utilisez un outil de suivi des bogues pour enregistrer tous les détails sur les différents problèmes, assigner des ressources pour les résoudre et créer un journal d’audit contenant la progression et l’état de chaque problème. 

**Gérez toutes les ressources dans un système de gestion des changements.** Tous les aspects de votre processus DevOps doivent être inclus dans un système de gestion et de contrôle de version, afin que vous puissiez facilement suivre et auditer les modifications. Les aspects en question incluent le code, l’infrastructure, la configuration, la documentation et les scripts. Traitez tous ces types de ressources comme du code tout au long des processus de test, génération, révision. 

**Utilisez des listes de contrôle** Créez des listes de contrôle des opérations pour assurer le suivi des processus. Il n’est pas rare de passer à côté d’un élément dans un manuel volumineux. Si vous suivez une liste de contrôle, vous avez plus de chances de remarquer les détails que vous auriez risqué de manquer. Mettez à jour vos listes de contrôle et cherchez en permanence des méthodes pour automatiser les tâches et simplifier les processus.

Pour plus d’informations sur le DevOps, consultez l’article [Qu’est-ce que DevOps ?][what-is-devops] sur le site de Visual Studio.

<!-- links -->

[app-insights]: /azure/application-insights/
[azure-ad]: https://azure.microsoft.com/services/active-directory/
[azure-diagnostics]: /azure/monitoring-and-diagnostics/azure-diagnostics
[azure-monitor]: /azure/monitoring-and-diagnostics/monitoring-overview
[azure-support-plans]: https://azure.microsoft.com/support/plans/
[blue-green]: https://martinfowler.com/bliki/BlueGreenDeployment.html
[canary-release]:https://martinfowler.com/bliki/CanaryRelease.html
[dev-test]: https://azure.microsoft.com/solutions/dev-test/
[feature-toggles]: https://www.martinfowler.com/articles/feature-toggles.html
[oms]: https://www.microsoft.com/cloud-platform/operations-management-suite
[rbac]: /azure/active-directory/role-based-access-control-what-is
[resiliency]: ../resiliency/index.md
[resource-manager]: /azure/azure-resource-manager/
[trunk-based]: https://trunkbaseddevelopment.com/
[what-is-devops]: https://www.visualstudio.com/learn/what-is-devops/
