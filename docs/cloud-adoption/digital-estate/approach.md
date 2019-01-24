---
title: Approches de la planification du patrimoine numérique
titleSuffix: Enterprise Cloud Adoption
description: Décrit les approches de planification du patrimoine numérique
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.openlocfilehash: 59c9e208254eaa0e8beb365718ef59d0694094f3
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54482219"
---
# <a name="enterprise-cloud-adoption-approaches-to-digital-estate-planning"></a>Adoption du cloud d’entreprise : Approches de la planification du patrimoine numérique

Le patrimoine numérique peut prendre plusieurs formes, selon les résultats escomptés et la taille du patrimoine existant. Chaque approche propose plusieurs options. Il est important de définir ses propres attentes relativement tôt dans le cycle de planification. Des attentes qui n’ont pas été clairement définies mènent souvent à des retards dus à la nécessité d’effectuer des inventaires supplémentaires. Cet article présente trois approches pour l’analyse.

## <a name="workload-driven-approach"></a>Approche basée sur les charges de travail

L’approche d’évaluation descendante évalue certains aspects de la sécurité, comme la catégorisation des données (impact métier élevé, moyen ou faible), la conformité, la souveraineté et les exigences liées aux risques de sécurité. Cette approche évalue ensuite la complexité architecturale générale, en évaluant des aspects comme l’authentification, la structure des données, les exigences de latence, les dépendances et l’espérance de vie des applications. Ensuite, l’approche descendante mesure les exigences opérationnelles de l’application, comme les niveaux de service, l’intégration, les fenêtres de maintenance, la supervision et les insights. Lorsque tous ces aspects ont été analysés et pris en considération, vous obtenez un score qui reflète la difficulté relative qui sera associée à la migration de cette application vers chacune des plateformes cloud : IaaS, PaaS et SaaS.

Deuxièmement, l’évaluation descendante évalue les avantages financiers de l’application, comme l’efficacité opérationnelle, le coût total de possession, le retour sur investissement et autres métriques financières. L’évaluation examine aussi le caractère saisonnier de l’application (existe-t-il des périodes de l’année où la demande connaît des pics ?) ainsi que la charge de calcul globale. De plus, elle examine les types d’utilisateurs pris en charge (normal/expert, régulier/occasionnel), et donc, la scalabilité et l’élasticité nécessaires. Enfin, l’évaluation se termine par l’examen des exigences de continuité de l’activité et de résilience de l’application, ainsi que des dépendances permettant d’exécuter l’application en cas d’interruption du service.

> [!TIP]
> Cette approche nécessite des entretiens avec les parties prenantes technico-commerciales afin de rassembler leurs commentaires. La disponibilité des parties prenantes clés est la principale difficulté pour le timing. La nature anecdotique des sources de données rend plus difficile la production d’estimations exactes concernant les coûts ou le timing. Effectuez les planifications à l’avance et validez les données collectées.

## <a name="asset-driven-approach"></a>Approche basée sur les ressources

Cette approche fournit un plan basé sur les ressources qui prennent en charge une application devant faire l’objet d’une migration. Les données d’utilisation statistiques sont tirées (pull) à partir d’une base de données de gestion de la configuration ou d’autres outils d’évaluation d’infrastructure. Cette approche a généralement comme ligne de base un modèle IaaS de déploiement. Dans ce processus, l’analyse évalue les attributs de chaque ressource : mémoire, nombre de processeurs (cœurs de CPU), espace de stockage du système d’exploitation, lecteurs de données, cartes d’interface réseau (NIC), IPv6, équilibrage de charge réseau, clustering, version du système d’exploitation, version de la base de données (si nécessaire), domaines pris en charge, composants ou packages logiciels tiers, entre autres. Les ressources inventoriées dans cette approche sont ensuite alignées sur les charges de travail ou les applications à des fins de regroupement et de mappage des dépendances.

> [!TIP]
> Cette approche nécessite une source de données d’utilisation statistiques riche. Le temps nécessaire à l’analyse de l’inventaire et à la collecte des données peut constituer une difficulté importante pour le timing. Les sources de données de bas niveau peuvent ne pas avoir les dépendances nécessaires aux ressources ou aux applications. Prévoyez au moins un mois pour l’analyse de l’inventaire. Validez les dépendances avant le déploiement.

## <a name="incremental-approach"></a>Approche incrémentielle

Comme souvent avec l’adoption du cloud en entreprise, une approche incrémentielle est fortement recommandée. Dans le cas de la planification du patrimoine numérique, cela équivaut à un processus multiphase :

- Analyse des coûts initiale : Si une validation financière est nécessaire, commencez par une approche basée sur les ressources (décrite ci-dessus) afin d’obtenir un premier calcul des coûts pour l’intégralité du patrimoine numérique, sans rationalisation. Ceci établit un banc d’essai du scénario le plus défavorable.

- Planification de la migration : Une fois que vous avez désigné l’équipe de stratégies cloud, créez un premier backlog de migration à l’aide d’une approche basée sur les charges de travail, en vous basant uniquement sur vos connaissances collectives et sur les entretiens limités avec les parties prenantes. Avec cette approche, vous obtenez rapidement une évaluation simple des charges de travail pour favoriser la collaboration.

- Planification de la mise en production : Lors de chaque mise en production, le backlog de migration est nettoyé et à nouveau hiérarchisé pour se focaliser sur les impacts métier les plus importants. Pendant ce processus, les 5&ndash;10 charges de travail suivantes sont sélectionnées comme des versions prioritaires. À ce stade, l’équipe chargée des stratégies cloud consacre du temps à l’exécution d’une approche complète basée sur les charges de travail. Le fait de différer cette évaluation jusqu’à l’alignement d’une mise en production permet de mieux respecter l’emploi du temps des parties prenantes. Cela diffère également l’analyse complète jusqu’au moment où l’entreprise commence à voir les résultats de ses précédents efforts.

- Analyse de l’exécution : Avant de pouvoir effectuer la migration, la modernisation ou la réplication d’une ressource, celle-ci doit être évaluée individuellement et collectivement dans le cadre de la mise en production. À ce stade, les données obtenues avec l’approche basée sur les ressources peuvent être examinées pour garantir l’exactitude de leur taille et de leurs contraintes opérationnelles.

> [!TIP]
> Cette approche incrémentielle permet une planification simplifiée et des résultats accélérés. Il est très important que toutes les parties concernées comprennent la raison de cette prise de décision différée. Il est également important que les hypothèses émises à chaque étape soient documentées pour éviter toute perte d’informations.

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez sélectionné une approche, vous pouvez réaliser l’inventaire.

> [!div class="nextstepaction"]
> [Collecter des données d’inventaire](inventory.md)