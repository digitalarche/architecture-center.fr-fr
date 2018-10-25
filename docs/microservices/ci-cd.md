---
title: Intégration et livraison continues pour les microservices
description: Intégration et livraison continues pour les microservices
author: MikeWasson
ms.date: 10/23/2018
ms.openlocfilehash: b411e687a111e55a5821d4fdc66975e80f73584b
ms.sourcegitcommit: fdcacbfdc77370532a4dde776c5d9b82227dff2d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2018
ms.locfileid: "49962855"
---
# <a name="designing-microservices-continuous-integration"></a>Conception des microservices : Intégration continue

L’intégration et la livraison continues sont une condition essentielle à la réussite des microservices. Sans processus efficace d’intégration et de livraison continues, vous n’obtiendrez pas l’agilité attendue des microservices. Certaines des difficultés du processus appliqué aux microservices proviennent de la multiplication des bases de code et de l’hétérogénéité des environnements de génération pour les divers services. Ce chapitre décrit ces obstacles et recommande certaines stratégies pour les surmonter.

![](./images/ci-cd.png)

L’accélération des cycles de version constitue l’un des motifs principaux d’adoption d’une architecture de microservices. 

Dans une application purement monolithique, un pipeline unique de conception génère le fichier exécutable de l’application. L’ensemble des tâches de développement alimentent ce pipeline. Si un bogue de priorité élevée est identifié, un correctif doit être intégré, testé puis publié, ce qui peut retarder la sortie des nouvelles fonctionnalités. Il est vrai que vous pouvez atténuer ces problèmes en vous dotant de modules correctement rationalisés et en valorisant des branches de fonctionnalités destinées à réduire l’incidence des modifications de code. Toutefois, à mesure que l’application devient plus complexe, en parallèle de l’ajout des fonctionnalités, le processus de publication d’un monolithe a tendance à se fragiliser et présente des risques de rupture. 

Si une approche de microservices est suivie, vous ne déplorez jamais une longue chaîne de publication au sein de laquelle les équipes interviennent tour à tour. L’équipe qui développe le service « A » peut publier une mise à jour à tout moment, sans attendre que les modifications de l’équipe du service « B » soient fusionnées, testées et déployées. Le processus d’intégration et de livraison continues est essentiel à cette approche. Votre pipeline de publication doit être automatisé et hautement fiable, de manière à réduire au maximum les risques liés au déploiement des mises à jour. Si vous publiez une production tous les jours ou plusieurs fois par jour, les régressions et les interruptions de service doivent être très rares. En même temps, en cas de déploiement d’une mauvaise mise à jour, vous devez disposer d’un moyen fiable pour revenir vers une version antérieure du service.

![](./images/cicd-monolith.png)

Le processus dont nous parlons traite de plusieurs processus associés : intégration continue, livraison continue et déploiement continu.

- L’intégration continue implique que les modifications du code sont fréquemment fusionnées dans la branche principale, à l’aide de processus automatisés de génération et de test qui garantissent que le code de la branche principale présente toujours une qualité acceptable pour la production.

- Avec la livraison continue, les modifications de code sont automatiquement publiées sur un environnement de type production. Si le déploiement au sein de l’environnement de production réelle peut nécessiter une approbation manuelle, il est automatisé. L’objectif est de toujours disposer d’un code *prêt* à être déployé en production.

- Avec le déploiement continu, les modifications de code traitées par le processus sont automatiquement déployées en production.

Dans le contexte de Kubernetes et des microservices, l’intégration continue consiste en la génération et le test d’images de conteneurs, et en leur transmission vers un registre de conteneurs. Durant la phase de déploiement, les spécifications de pod sont mises à jour pour récupérer l’image de production la plus récente.

## <a name="challenges"></a>Défis

- **De nombreuses bases de code indépendantes et réduites**. Chaque équipe est responsable de la création de son propre service, associé à un pipeline de génération spécifique. Dans certaines organisations, les équipes peuvent utiliser des référentiels de code séparés. Ainsi, parfois, l’expertise en matière de génération du système est partagée entre les équipes, sans que quiconque au sein de l’organisation ne sache comment déployer l’application entière. Par exemple, que se passe-t-il en cas de récupération d’urgence, si vous devez déployer rapidement vers un nouveau cluster ?   

- **Langues et infrastructures multiples**. Si chaque équipe utilise sa propre combinaison de technologies, il peut s’avérer difficile de créer un processus de génération unique fonctionnant dans tous les services de l’organisation. Le processus de génération doit être suffisamment souple pour que chaque équipe puisse opter pour la langue ou l’infrastructure de son choix. 

- **Intégration et test de charge**. Chaque équipe publiant les mises à jour à son propre rythme, la mise en œuvre d’une approche de test robuste de bout en bout peut présenter des difficultés, plus particulièrement quand des interdépendances entre services sont identifiées. En outre, l’exécution d’un cluster de production intégral peut s’avérer onéreuse. Aussi, il est peu probable que l’ensemble des équipes soient en mesure d’exécuter leur propre cluster à l’échelle de la production, simplement pour la phase de test. 

- **Gestion des mises en production**. Chaque équipe doit être capable de déployer une mise à jour en production. Cela ne signifie pas que tous les membres doivent être autorisés à le faire. Cependant, une organisation disposant d’un rôle centralisé de gestionnaire des versions peut constater un ralentissement de ses déploiements. Plus le processus d’intégration et de livraison continue est automatisé et fiable, moins la nécessité d’une autorité centrale se fait sentir. Ceci dit, vous pouvez avoir plusieurs stratégies, adaptées à la fois à la publication des modifications majeures de fonctionnalités et des correctifs mineurs. La décentralisation ne signifie pas qu’il n’existe plus aucune trace de gouvernance.

- **Gestion des versions d’images de conteneur**. Durant le cycle de développement et de test, le processus d’intégration et de livraison continue génère de nombreuses images de conteneur. Seulement une partie de ces instances sont candidates à la publication, puis un nouvel écrémage filtre les éléments transmis en phase de production. Vous devez disposer d’une stratégie claire de gestion des versions, qui vous permettent de savoir quelles images sont déployées à un instant t en production, et ainsi de revenir à une version antérieure si nécessaire. 

- **Mises à jour de service**. Lorsque vous mettez à jour un service vers une nouvelle version, il ne doit pas perturber les autres services qui en dépendent. Si vous procédez à une mise à jour propagée, une combinaison de versions différentes sera exécutée sur un intervalle ponctuel. 
 
Ces enjeux reflètent une problématique centrale. D’une part, les équipes doivent travailler aussi indépendamment que possible. Malgré tout, une certaine coordination est requise, de manière à ce qu’une personne donnée puisse effectuer des tâches variées, comme l’exécution du test d’intégration, le redéploiement de la solution entière vers un nouveau cluster ou l’annulation d’une mauvaise mise à jour. 
 
## <a name="cicd-approaches-for-microservices"></a>Approches d’intégration/livraison continues pour les microservices

Il est recommandé que chaque équipe de service héberge son environnement de génération au sein d’un conteneur. Ce conteneur doit comporter tous les outils nécessaires pour le développement des artefacts de code dédiés au service considéré. Bien souvent, vous pouvez trouver une image Docker officielle pour votre langue et votre infrastructure. Vous pouvez ensuite recourir à `docker run` ou à Docker Compose pour exécuter la version. 

Avec cette approche, il est très facile de configurer un nouvel environnement de génération. Un développeur souhaitant générer votre code n’a pas besoin d’installer un jeu d’outils de génération ; il lui suffit d’exécuter l’image de conteneur. Plus important encore peut-être, votre serveur de builds peut être configuré pour effectuer la même chose. De cette manière, vous n’avez pas à installer ces outils sur le serveur de builds, ni à gérer des versions en conflit des outils. 

Pour les opérations locales de développement et de test, utilisez Docker pour exécuter le service au sein d’un conteneur. Dans le cadre de ce processus, vous devrez peut-être exécuter d’autres conteneurs prenant en charge des services fictifs ou des bases de données test nécessaires pour le test en local. Vous pourriez utiliser Docker Compose pour coordonner ces conteneurs, ou Minikube pour exécuter Kubernetes en local. 

Lorsque le code est prêt, ouvrez une requête d’extraction et procédez à une fusion dans les données de références. Cela déclenche une tâche sur le serveur de builds :

1. Générez les ressources du code. 
2. Exécutez des tests unitaires sur le code.
3. Générez l’image de conteneur.
4. Testez l’image de conteneur en exécutant des tests fonctionnels sur un conteneur en exécution. Durant cette phase, des erreurs peuvent être identifiées dans le fichier Docker, comme un mauvais point d’entrée.
5. Envoyez l’image dans un registre de conteneurs.
6. Mettez à jour le cluster de test avec la nouvelle image avant d’exécuter les tests d’intégration.

Lorsque l’image est prête à passer en production, mettez à jour les fichiers de déploiement en fonction de l’image la plus récente, en tenant compte des éventuels fichiers de configuration Kubernetes. Appliquez ensuite la mise à jour sur le cluster de production.

Voici quelques recommandations pour optimiser la fiabilité des déploiements :
 
- Définissez des conventions d’organisation pour les balises de conteneurs, la gestion des versions et des conventions de d’affectation de noms pour les ressources déployées sur le cluster (pod, services, etc.). Cela peut simplifier le diagnostic des problèmes de déploiement. 

- Créez 2 registres distincts de conteneur, un dédié au développement/au test et un à la production. N’envoyez pas l’image dans le registre de production avant d’être prêt à la déployer en phase de production. Si vous combinez cette pratique à la gestion sémantique des versions des images de conteneurs, vous pouvez réduire les risques de déploiement accidentel d’une version non approuvée pour publication.

## <a name="updating-services"></a>Mise à jour des services

Il existe différentes stratégies de mise à jour d’un service déjà en production. Nous présentons ici 3 options courantes : mise à jour propagée, déploiement bleu-vert et contrôle de la validité de la mise en production.

### <a name="rolling-update"></a>Mise à jour propagée 

Dans une mise à jour propagée, vous déployez de nouvelles instances d’un service ; ces nouvelles instances reçoivent immédiatement les requêtes. À mesure que les nouvelles instances arrivent, les anciennes instances sont supprimées.

Les mises à jour propagées correspondent au comportement par défaut dans Kubernetes, quand vous mettez à jour les spécifications des pods pour un déploiement. Le contrôleur de déploiement crée un nouveau jeu de réplicas pour les pods mis à jour. Ensuite, il promeut le nouveau jeu en rétrogradant l’ancien, ceci pour garantir le maintien d’un nombre souhaité de réplicas. Les anciens pods ne sont pas supprimés avant que les nouveaux ne soient prêts. Kubernetes conserve un historique de la mise à jour, ce qui vous permet d’utiliser kubectl pour annuler une opération, au besoin. 

Si votre service effectue une tâche de démarrage longue, vous pouvez définir une détection de préparation. Cette sonde de préparation signale quand le conteneur est prêt à recevoir le trafic. Kubernetes n’envoie aucun trafic au pod avant l’aval de la sonde. 

L’une des difficultés liées aux mises à jour propagées est la confusion régnant durant le processus de mise à jour. À ce stade, une combinaison de versions anciennes et nouvelles exécute et reçoit le trafic. Pendant cette période, une requête peut être dirigée vers l’une ou l’autre des versions. Cela peut provoquer des problèmes, en fonction de l’ampleur des changements entre les deux versions. 

### <a name="blue-green-deployment"></a>Déploiement bleu-vert

Dans un déploiement bleu-vert, il est possible de déployer la nouvelle version en parallèle de la version précédente. Une fois la nouvelle version validée, vous basculez simultanément l’ensemble du trafic de l’ancienne version vers la nouvelle. Après le basculement, vous vérifiez l’absence de problèmes dans l’application. Si une erreur survient, vous pouvez revenir à l’ancienne version. En supposant qu’il n’existe aucun problème, vous pouvez supprimer l’ancienne version.

Avec une application monolithique ou multiniveau plus traditionnelle, un déploiement bleu-vert est généralement synonyme d’approvisionnement de deux environnements identiques. Vous déployez alors la nouvelle version dans un environnement intermédiaire, avant de rediriger le trafic client vers l’environnement intermédiaire, par exemple, en échangeant les adresses IP virtuelles.

Dans Kubernetes, il n’est pas nécessaire d’approvisionner un cluster distinct pour procéder à des déploiements bleu-vert. Ici, vous pouvez valoriser des sélecteurs. Créez une nouvelle ressource de déploiement avec une nouvelle spécification de pods et un jeu différent d’étiquettes. Créez ce déploiement, sans supprimer le déploiement précédent ni modifier le service pointant vers lui. Une fois que les nouveaux pods sont exécutés, vous pouvez mettre à jour le sélecteur du service en fonction du nouveau déploiement. 

Avec les déploiements bleu-vert, le service bascule l’ensemble des pods au même moment. Une fois que le service est mis à jour, toutes les nouvelles requêtes sont acheminées vers la nouvelle version. L’un des inconvénients est que pendant la mise à jour, vous exécutez deux fois plus de pods pour le service (l’actuel et le suivant). Si les pods nécessitent un volume important d’UC ou de ressources de mémoire, il vous faudra éventuellement augmenter temporairement la taille du cluster pour prendre en charge efficacement la consommation de ressources. 

### <a name="canary-release"></a>Contrôle la validité de la mise en production

Durant un contrôle de la validité de la mise en production, vous déployez une version mise à jour à un nombre réduit de clients. Ensuite, vous analysez le comportement du nouveau service avant de procéder à un déploiement plus large vers l’ensemble des clients. Ainsi, vous contrôlez la mise en œuvre progressive du processus, observez les données réelles et identifiez les problèmes avant que l’ensemble des clients ne soient affectés.

La gestion de cette version est plus complexe que celle des versions bleu-vert et propagée, dans la mesure où vous devez diriger dynamiquement les requêtes vers différentes versions du service. Dans Kubernetes, vous pouvez configurer un service pour une prise en charge de deux jeux de réplicas (un pour chaque version), avant de modifier manuellement au besoin la quantité de réplicas. Toutefois, cette approche présente une granularité plutôt grossière, en raison de la manière dont la charge de Kubernetes s’équilibre entre les pods. Par exemple, si vous disposez d’au total dix réplicas, vous pouvez modifier le trafic en incrément de 10 % uniquement. Si vous avez recours à une maille de service, vous pouvez solliciter les règles d’acheminement dédiées pour implémenter une stratégie de publication plus sophistiquée. Voici quelques ressources qui peuvent vous être utiles :

- Kubernetes sans maille de service : [contrôles de la validité de la mise en production](https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/#canary-deployments)
- Linkerd : [acheminement dynamique des requêtes](https://linkerd.io/features/routing/)
- Istio : [contrôles de la validité de la mise en production à l’aide d’Istio](https://istio.io/blog/canary-deployments-using-istio.html)

## <a name="conclusion"></a>Conclusion

Au cours des récentes années, le marché a subi une refonte majeure, en passant d’un modèle de développement de *systèmes d’enregistrement* à un modèle de mise en œuvre de *systèmes d’engagement*.

Les systèmes d’enregistrement sont des applications traditionnelles de gestion des données back-office. Au cœur de ces systèmes est bien souvent hébergé un système SGBDR, qui est le cas échéant la source unique de vérité. L’expression « système d’engagement » a été inventée par Geoffrey Moore, dans son article paru en 2011 *Systems of Engagement and the Future of Enterprise IT* (Systèmes d’engagement et l’avenir de l’informatique d’entreprise). Les systèmes d’engagement sont des applications axées sur la communication et la collaboration. Ils connectent les personnes en temps réel. Ils doivent être disponibles 24h/24 et 7j/7. De nouvelles fonctionnalités sont régulièrement introduites, sans mise hors connexion des applications. Les utilisateurs sont davantage exigeants et plus impatients vis-à-vis des délais inattendus ou des périodes d’interruption.

Au sein du marché de la consommation, une amélioration de l’expérience utilisateur peut constituer une valeur métier quantifiable. La durée pendant laquelle un utilisateur interagit avec une application peut se traduire directement en revenus. Dans l’univers des systèmes d’entreprise, les attentes des utilisateurs ont évolué. Si ces systèmes visent à encourager la communication et la collaboration, ils doivent être inspirés des applications orientées consommateur.

Les microservices sont une réponse à cette évolution. En décomposant une application monolithique en groupe de services sommairement liés, nous sommes en mesure de réguler la publication de chaque service et de mettre en œuvre des mises à jour fréquentes sans déplorer de périodes d’interruption ni bouleversement fondamental. Les microservices, par ailleurs, présentent des avantages en matière d’évolutivité, d’isolation des défaillances et de résilience. Parallèlement, les plateformes cloud simplifient le développement et l’exécution des microservices, avec un approvisionnement automatisé des ressources de calcul, des orchestrateurs de conteneur en tant que service et des environnements sans serveur basés sur les événements.

Cependant, comme nous l’avons vu, les architectures de microservices présentent de nombreuses difficultés. Pour réussir, vous devez partir d’une conception robuste. Il est nécessaire d’analyser soigneusement le domaine, et de prendre soin de bien choisir les technologies, modéliser les données et concevoir les API tout en veillant à instaurer une culture DevOps mature. Nous espérons que ce guide et l’[implémentation de référence](https://github.com/mspnp/microservices-reference-implementation) associée vous ont éclairé la voie sur le chemin de la réussite. 

