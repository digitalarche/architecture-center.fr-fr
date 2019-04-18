---
title: Intégration et livraison continues pour les microservices
description: Intégration continue et livraison continue pour les microservices.
author: MikeWasson
ms.date: 03/27/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: c52ff3d0a330f564e5f7e9b0b07f0ba84c328c8b
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59639781"
---
# <a name="cicd-for-microservices-architectures"></a>CI/CD pour les architectures de microservices

Les cycles de mise en production plus rapides sont un des principaux avantages des architectures de microservices. Mais sans un bon processus CI/CD, vous n’atteindrez pas l’agilité qui promettent de microservices. Cet article décrit ces obstacles et recommande certaines stratégies pour le problème.

## <a name="what-is-cicd"></a>Qu’est CI/CD ?

Lorsque nous parlons de CI/CD, nous parlons vraiment plusieurs processus associés : intégration continue, livraison continue et déploiement continu.

- **Intégration continue**. Modifications du code sont fréquemment fusionnées dans la branche principale. Automatisée de build et le processus de test vous assurer que le code dans la branche principale est toujours de qualité production.

- **Livraison continue**. Modification du code qui passent le processus d’intégration continue est automatiquement publiées dans un environnement de type production. Si le déploiement au sein de l’environnement de production réelle peut nécessiter une approbation manuelle, il est automatisé. L’objectif est de toujours disposer d’un code *prêt* à être déployé en production.

- **Déploiement continu**. Modifications du code qui passent les deux étapes précédentes sont déployés automatiquement *en production*.

Voici certains objectifs d’un processus CI/CD robuste pour une architecture de microservices :

- Chaque équipe peut générer et déployer ses propres services de manière indépendante, sans affecter ou interrompre d’autres équipes.

- Avant d’être déployée en production, une nouvelle version d’un service est déployée sur les environnements de développement/test/assurance qualité à des fins de validation. Des critères de qualité sont appliqués à chaque étape.

- Une nouvelle version d’un service peut être déployée côte à côte avec la version précédente.

- Des stratégies de contrôle d’accès suffisantes sont en place.

- Pour les charges de travail, vous pouvez approuver les images de conteneur qui sont déployés en production.

## <a name="why-a-robust-cicd-pipeline-matters"></a>Importance d’un pipeline CI/CD robust

Dans une application monolithique traditionnelle, il existe un pipeline de build unique dont la sortie est l’exécutable de l’application. L’ensemble des tâches de développement alimentent ce pipeline. Si un bogue de priorité élevée est identifié, un correctif doit être intégré, testé puis publié, ce qui peut retarder la sortie des nouvelles fonctionnalités. Vous pouvez atténuer ces problèmes en ayant des modules bien factorisées et à l’aide des branches de fonctionnalité pour minimiser l’impact des modifications du code. Toutefois, à mesure que l’application devient plus complexe, en parallèle de l’ajout des fonctionnalités, le processus de publication d’un monolithe a tendance à se fragiliser et présente des risques de rupture.

Si une approche de microservices est suivie, vous ne déplorez jamais une longue chaîne de publication au sein de laquelle les équipes interviennent tour à tour. L’équipe qui développe le service « A » peut publier une mise à jour à tout moment, sans attendre que les modifications de l’équipe du service « B » soient fusionnées, testées et déployées.

![Diagramme d’un monolithe CI/CD](./images/cicd-monolith.png)

Pour obtenir une grande rapidité de diffusion, votre pipeline de versions doit être automatisé et hautement fiable, afin de minimiser les risques. Si vous libérez en production tous les jours ou plusieurs fois par jour, les régressions ou les interruptions de service doivent être très rares. En même temps, en cas de déploiement d’une mauvaise mise à jour, vous devez disposer d’un moyen fiable pour revenir vers une version antérieure du service.

## <a name="challenges"></a>Défis

- **De nombreuses bases de code indépendantes et réduites**. Chaque équipe est responsable de la création de son propre service, associé à un pipeline de génération spécifique. Dans certaines organisations, les équipes peuvent utiliser des référentiels de code séparés. Dépôts distincts peuvent entraîner une situation où la base de connaissances de la génération du système est réparti entre les équipes et quiconque au sein de l’organisation sait comment déployer l’application entière. Par exemple, que se passe-t-il en cas de récupération d’urgence, si vous devez déployer rapidement vers un nouveau cluster ?

    **Atténuation** : Disposez d’un pipeline unifié et automatisé pour générer et déployer des services, afin que cette base de connaissances n’est pas « masqué » au sein de chaque équipe.

- **Langues et infrastructures multiples**. Si chaque équipe utilise sa propre combinaison de technologies, il peut s’avérer difficile de créer un processus de génération unique fonctionnant dans tous les services de l’organisation. Le processus de génération doit être suffisamment souple pour que chaque équipe puisse opter pour la langue ou l’infrastructure de son choix.

    **Atténuation** : Mettre en conteneur le processus de génération pour chaque service. De cette façon, le système de génération uniquement doit être en mesure d’exécuter les conteneurs.

- **Intégration et test de charge**. Chaque équipe publiant les mises à jour à son propre rythme, la mise en œuvre d’une approche de test robuste de bout en bout peut présenter des difficultés, plus particulièrement quand des interdépendances entre services sont identifiées. En outre, un cluster de production complet en cours d’exécution peut être coûteuse, il est peu probable que chaque équipe s’exécutera son propre cluster complet à des échelles de production, juste pour le test.

- **Gestion des mises en production**. Chaque équipe doit être en mesure de déployer une mise à jour en production. Cela ne signifie pas que tous les membres doivent être autorisés à le faire. Cependant, une organisation disposant d’un rôle centralisé de gestionnaire des versions peut constater un ralentissement de ses déploiements.

    **Atténuation** : Plus le processus d’intégration et de livraison continue est automatisé et fiable, moins la nécessité d’une autorité centrale se fait sentir. Ceci dit, vous pouvez avoir plusieurs stratégies, adaptées à la fois à la publication des modifications majeures de fonctionnalités et des correctifs mineurs. La décentralisation ne signifie pas aucune trace de gouvernance.

- **Mises à jour de service**. Lorsque vous mettez à jour un service vers une nouvelle version, il ne doit pas perturber les autres services qui en dépendent.

    **Atténuation** : Utiliser des techniques de déploiement telles que la mise en production bleu-vert ou contrôle de validité pour des modifications sans rupture. Les dernières modifications de l’API, déployez la nouvelle version côte à côte avec la version précédente. De cette façon, les services qui utilisent l’API précédente peuvent être mis à jour et testés pour la nouvelle API. Consultez [mise à jour des services](#updating-services), ci-dessous.

## <a name="monorepo-vs-multi-repo"></a>Plusieurs dépôts de Monorepo vs

Avant de créer un workflow CI/CD, vous devez savoir comment la base de code est structurée et gérée.

- Équipes fonctionnent dans les référentiels séparés ou dans un monorepo (référentiel unique) ?
- Quelle est votre stratégie de création de branche ?
- Qui peut envoyer (push) du code en production ? Existe-t-il un rôle de gestionnaire de mise en production ?

L’approche « dépôt unique » est de plus en plus utilisée, mais les deux approches ont des avantages et des inconvénients.

| &nbsp; | Dépôt unique | Dépôts multiples |
|--------|----------|----------------|
| **Avantages** | Partage du code<br/>Code et outils plus faciles à standardiser<br/>Code plus facile à refactoriser<br/>Découvrabilité - une même vue du code<br/> | Propriété clairement établie par équipe<br/>Potentiellement moins de conflits de fusion<br/>Aide à appliquer le découplage des microservices |
| **Défis** | Les modifications apportées au code partagé peuvent affecter plusieurs microservices<br/>Plus grand risque de conflits de fusion<br/>Les outils doivent être adaptés pour une grande base de code<br/>Contrôle d’accès<br/>Processus de déploiement plus complexe | Partage du code plus difficile<br/>Standards de codage plus difficiles à appliquer<br/>Gestion des dépendances<br/>Base de code diffuse, découvrabilité faible<br/>Manque d’infrastructure partagée

## <a name="updating-services"></a>Mise à jour des services

Il existe différentes stratégies de mise à jour d’un service déjà en production. Nous présentons ici trois options courantes : mise à jour propagée, déploiement bleu-vert et contrôle de la validité de la mise en production.

### <a name="rolling-updates"></a>Mises à jour propagées

Dans une mise à jour propagée, vous déployez de nouvelles instances d’un service ; ces nouvelles instances reçoivent immédiatement les requêtes. À mesure que les nouvelles instances arrivent, les anciennes instances sont supprimées.

**Exemple.** Dans Kubernetes, les mises à jour propagées sont le comportement par défaut lorsque vous mettez à jour les spécifications des PODS pour un déploiement. Le contrôleur de déploiement crée un nouveau jeu de réplicas pour les pods mis à jour. Ensuite, il promeut le nouveau jeu en rétrogradant l’ancien, ceci pour garantir le maintien d’un nombre souhaité de réplicas. Les anciens pods ne sont pas supprimés avant que les nouveaux ne soient prêts. Kubernetes conserve un historique de la mise à jour, donc vous pouvez restaurer une mise à jour si nécessaire.

L’une des difficultés liées aux mises à jour propagées est la confusion régnant durant le processus de mise à jour. À ce stade, une combinaison de versions anciennes et nouvelles exécute et reçoit le trafic. Pendant cette période, une requête peut être dirigée vers l’une ou l’autre des versions.

Les dernières modifications de l’API, une bonne pratique est de prendre en charge les deux versions côte à côte, jusqu'à ce que tous les clients de la version précédente sont mis à jour. Consultez [version de l’API](./design/api-design.md#api-versioning).

### <a name="blue-green-deployment"></a>Déploiement bleu-vert

Dans un déploiement bleu-vert, il est possible de déployer la nouvelle version en parallèle de la version précédente. Une fois la nouvelle version validée, vous basculez simultanément l’ensemble du trafic de l’ancienne version vers la nouvelle. Après le basculement, vous vérifiez l’absence de problèmes dans l’application. Si une erreur survient, vous pouvez revenir à l’ancienne version. En supposant qu’il n’existe aucun problème, vous pouvez supprimer l’ancienne version.

Avec une application monolithique ou multiniveau plus traditionnelle, un déploiement bleu-vert est généralement synonyme d’approvisionnement de deux environnements identiques. Vous déployez alors la nouvelle version dans un environnement intermédiaire, avant de rediriger le trafic client vers l’environnement intermédiaire, par exemple, en échangeant les adresses IP virtuelles. Dans une architecture de microservices, les mises à jour se produisent au niveau du microservice, qui serait généralement déployer la mise à jour dans le même environnement et d’utiliser un mécanisme de découverte de service pour le remplacement.

**Exemple**. Dans Kubernetes, il n’est pas nécessaire d’approvisionner un cluster distinct pour procéder à des déploiements bleu-vert. Ici, vous pouvez valoriser des sélecteurs. Créez une nouvelle ressource de déploiement avec une nouvelle spécification de pods et un jeu différent d’étiquettes. Créez ce déploiement, sans supprimer le déploiement précédent ni modifier le service pointant vers lui. Une fois que les nouveaux pods sont exécutés, vous pouvez mettre à jour le sélecteur du service en fonction du nouveau déploiement.

L’inconvénient du déploiement bleu-vert est pendant la mise à jour, vous devez exécuter deux fois plus pods pour le service (actuel et suivant). Si les pods nécessitent un volume important d’UC ou de ressources de mémoire, il vous faudra éventuellement augmenter temporairement la taille du cluster pour prendre en charge efficacement la consommation de ressources.

### <a name="canary-release"></a>Contrôle la validité de la mise en production

Durant un contrôle de la validité de la mise en production, vous déployez une version mise à jour à un nombre réduit de clients. Ensuite, vous analysez le comportement du nouveau service avant de procéder à un déploiement plus large vers l’ensemble des clients. Ainsi, vous contrôlez la mise en œuvre progressive du processus, observez les données réelles et identifiez les problèmes avant que l’ensemble des clients ne soient affectés.

La gestion de cette version est plus complexe que celle des versions bleu-vert et propagée, dans la mesure où vous devez diriger dynamiquement les requêtes vers différentes versions du service.

**Exemple**. Dans Kubernetes, vous pouvez configurer un service pour une prise en charge de deux jeux de réplicas (un pour chaque version), avant de modifier manuellement au besoin la quantité de réplicas. Toutefois, cette approche présente une granularité plutôt grossière, en raison de la manière dont la charge de Kubernetes s’équilibre entre les pods. Par exemple, si vous avez un total de 10 réplicas, vous pouvez décaler uniquement le trafic en incréments de 10 %. Si vous avez recours à une maille de service, vous pouvez solliciter les règles d’acheminement dédiées pour implémenter une stratégie de publication plus sophistiquée.

## <a name="next-steps"></a>Étapes suivantes

Découvrez les pratiques de CI/CD spécifiques pour les microservices s’exécutant sur Kubernetes.

- [CI/CD pour les microservices sur Kubernetes](./ci-cd-kubernetes.md)