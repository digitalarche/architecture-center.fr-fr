---
title: 'Framework d’adoption du cloud : Processus de conformité à la stratégie Accélération du déploiement'
description: Processus de conformité à la stratégie Accélération du déploiement
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
author: alexbuckgit
ms.openlocfilehash: 08046a4ae199a8714393d2aba3841dd84008d836
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900866"
---
# <a name="deployment-acceleration-policy-compliance-processes"></a>Processus de conformité à la stratégie Accélération du déploiement

Cet article décrit les processus d’adhésion à la stratégie, qui gouvernent [l’Accélération du déploiement](./overview.md). Pour être efficace, une gouvernance de configuration du cloud implique plusieurs processus manuels récurrents qui visent à détecter les problèmes et à imposer des stratégies d’atténuation de ces risques. Cependant, vous pouvez automatiser ces processus et les compléter par des outils pour réduire la charge de gouvernance et accélérer la réponse en cas de manquement.

## <a name="planning-review-and-reporting-processes"></a>Processus de planification, de révision et de génération de rapports

Les meilleurs outils Accélération de déploiement du cloud ne peuvent pas surpasser les processus et stratégies qu’ils appuient. Les éléments suivants représentent un ensemble d’exemples de processus généralement utilisés dans le cadre de la discipline Base de référence de la sécurité. Ces exemples constituent des points de départ pour planifier les processus qui vous permettront de mettre à jour la stratégie de sécurité à mesure que l’entreprise évolue et selon les commentaires des équipes informatiques chargés d’implémenter les mesures de gouvernance.

**Évaluation des risques et planification initiales** : Dans le cadre de l’adoption initiale de la discipline Accélération du déploiement, identifiez vos principaux risques métier et tolérances liés au déploiement de vos applications métier. Utilisez ces informations pour débattre des risques techniques particuliers avec les membres de l’équipe d’exploitation informatique et de l’équipe de sécurité, puis développer un ensemble de stratégies de base de déploiement et de configuration pour atténuer ces risques et ainsi établir votre stratégie de gouvernance initiale.

**Planification de déploiement** : Avant le déploiement de n’importe quelle ressource, effectuez une révision de la sécurité afin d’identifier tout nouveau risque et de vous assurer que toutes les exigences de la stratégie d’accès et de sécurité des données sont respectées.

**Test de déploiement** : Dans le cadre du processus de déploiement de n’importe quelle ressource, l’équipe de gouvernance du cloud, en collaboration avec les équipes de sécurité de votre entreprise, est chargée de passer en revue le déploiement pour valider la conformité à la stratégie de sécurité.

**Planification annuelle :** Tous les ans, effectuez une révision de la stratégie Accélération du déploiement. Étudiez les priorités de l’entreprise pour l’avenir et les stratégies d’adoption du cloud mises à jour afin d’identifier l’augmentation des risques potentiels ou d’autres opportunités et besoins de configuration émergents. Utilisez ce temps de révision annuelle pour examiner les dernières meilleures pratiques liées à l’Accélération du déploiement et intégrez-les dans vos stratégies et processus de révision.

**Planification et révision trimestrielles** : Tous les trimestres, effectuez une révision des données d’audit de sécurité et des rapports d’incident afin d’identifier les modifications requises dans la stratégie Accélération du déploiement. Dans le cadre de ce processus, passez en revue les ressources de cybersécurité afin d’anticiper de manière proactive les nouvelles menaces et mettre à jour la stratégie, le cas échéant. Une fois l’examen terminé, assurez-vous de la conformité des mesures de conception des applications et des systèmes à la stratégie mise à jour.

Ce processus de planification constitue également un moment idéal pour évaluer les lacunes des membres de votre équipe de gouvernance cloud en lien avec les risques, nouveaux ou évolutifs, encourus par les DevOps et la stratégie Accélération du déploiement. Invitez le personnel informatique concerné à participer aux révisions et à la planification en tant que conseillers techniques temporaires ou membres permanents de votre équipe.

**Apprentissage et formation** : Deux fois par mois, offrez des sessions de formation pour vous assurer que le personnel informatique et les développeurs connaissent les dernières exigences stratégiques Accélération du déploiement. Dans le cadre de ce processus, relisez et mettez à jour toute la documentation, les guides et les autres ressources de formation pour vous assurer qu’ils correspondent aux dernières instructions stratégiques de l’entreprise.

**Révisions d’audit et création de rapports mensuelles** : Tous les mois, effectuez un audit sur tous les déploiements cloud afin de garantir leur alignement continu sur la stratégie de configuration. Examinez les activités de sécurité avec le personnel informatique, puis identifiez les problèmes de conformité qui n’ont pas encore été pris en charge par le processus d’application et de supervision continues. Cet examen produit un rapport pour l’équipe de la stratégie cloud et chaque équipe d’adoption du cloud afin de communiquer sur l’adhésion générale des parties prenantes à la stratégie. Le rapport est également conservé à des fins d’audit et juridiques.

## <a name="ongoing-monitoring-processes"></a>Processus de supervision continue

Votre stratégie de gouvernance Accélération du déploiement réussit si l’état actuel et passé de votre infrastructure cloud est clairement identifiable. Si vous ne pouvez pas analyser les mesures et données pertinentes au sujet de l’intégrité et de l’activité de vos ressources cloud, vous ne pourrez pas identifier les changements de risques ni détecter les violations de vos tolérances aux risques. Les processus de gouvernance continus décrits ci-dessus exigent que vous fournissiez des données de qualité afin d’être capable de modifier la stratégie pour protéger votre infrastructure face à l’évolution des menaces et des risques induits par des ressources mal configurées.

Assurez-vous que vos équipes de sécurité et informatiques ont mis en place des systèmes de supervision automatisés pour votre infrastructure cloud recueillant les données de journaux pertinentes dont vous avez besoin pour évaluer les risques. Soyez proactif dans le cadre de la surveillance de ces systèmes afin d’assurer la détection et l’atténuation rapides des violations potentielles des stratégies, et assurez-vous que votre stratégie de supervision est conforme aux besoins de déploiement et de configuration.

## <a name="violation-triggers-and-enforcement-actions"></a>Déclencheurs relatifs aux violations et actions de mise en conformité

Du fait que la non-conformité aux stratégies de configuration peut entraîner de sérieux risques d’interruption de service, l’équipe de gouvernance du cloud doit disposer d’une visibilité totale sur les violations graves des stratégies. Veillez à ce que le personnel informatique dispose de procédures d’escalade claires pour signaler les problèmes de conformité à la configuration aux membres de l’équipe de gouvernance les mieux placés pour identifier et vérifier que les problèmes de stratégie sont atténués.  

Lorsque des violations sont détectées, vous devez prendre des actions conformes à la stratégie dès que possible. Votre équipe informatique peut automatiser la plupart des déclencheurs relatifs aux violations à l’aide des outils présentés dans la [chaîne d’outils Accélération du déploiement pour Azure](toolchain.md).

Les déclencheurs et mesures d’exécution suivants fournissent des exemples utilisables lorsque vous débattez de la façon d’utiliser les données de supervision pour résoudre les violations des stratégies :

- **Changements inattendus détectés dans la configuration**. Si la configuration d’une ressource change de façon inattendue, collaborez avec le personnel informatique et les responsables de la charge de travail pour identifier les causes profondes et élaborer un plan d’atténuation.
- **La configuration de nouvelles ressources n’est pas conforme à la stratégie.** Collaborez avec les équipes DevOps et les responsables de la charge de travail pour passer en revue les stratégies Accélération du déploiement lors du démarrage du projet afin que toutes les personnes impliquées comprennent les exigences de stratégie pertinentes.
- **Les échecs de déploiement ou les problèmes de configuration entraînent des retards dans les échéances du projet.** Collaborez avec les équipes de développement et les responsables de la charge de travail pour vous assurer que l’équipe comprend comment automatiser le déploiement des ressources sur le cloud par souci de cohérence et de répétabilité des tâches. Des déploiements entièrement automatisés sont nécessaires au début du cycle de développement &mdash;, ce qui entraîne habituellement des problèmes et des retards imprévus.

## <a name="next-steps"></a>Étapes suivantes

À l’aide du [modèle de gestion cloud](./template.md), documentez les processus et les déclencheurs qui correspondent au plan d’adoption du cloud actuel.

Pour obtenir des conseils sur l’exécution des stratégies de gestion cloud en harmonie avec les plans d’adoption, consultez l’article sur l’amélioration de la discipline.

> [!div class="nextstepaction"]
> [Amélioration de la discipline Accélération du déploiement](./discipline-improvement.md)
