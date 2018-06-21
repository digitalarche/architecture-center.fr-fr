---
title: Identification des limites de microservice
description: Identification des limites de microservice
author: MikeWasson
ms.date: 12/08/2017
ms.openlocfilehash: d35b92ffd97c4fda5d6599340925ce3dfea7f15b
ms.sourcegitcommit: a5e549c15a948f6fb5cec786dbddc8578af3be66
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/06/2018
ms.locfileid: "33673400"
---
# <a name="designing-microservices-identifying-microservice-boundaries"></a>Conception de microservices : identification des limites de microservice

Quelle est la taille adaptée pour un microservice ? Vous entendez souvent « ni trop gros, ni trop petit » &mdash; et, bien que certainement correct, ce n’est pas très utile dans la pratique. Mais si vous partez d’un modèle de domaine bien conçu, il est beaucoup plus facile d’aborder les microservices.

![](./images/bounded-contexts.png)

## <a name="from-domain-model-to-microservices"></a>Du modèle de domaine aux microservices

Dans le [chapitre précédent](./domain-analysis.md), nous avons défini un ensemble de contextes limités pour l’application de livraison par drone, Drone Delivery. Ensuite, nous avons examiné plus en détail un de ces contextes limités (celui de l’expédition) et identifié un ensemble d’entités, d’agrégats et de services de domaine pour ce contexte limité.

Nous sommes maintenant prêts à passer du modèle de domaine à la conception de l’application. Il s’agit d’une approche que vous pouvez utiliser pour dériver des microservices à partir du modèle de domaine.

1. Commencez avec un contexte limité. En général, la fonctionnalité dans un microservice ne doit pas s’étendre à plusieurs contextes limités. Par définition, un contexte limité marque la limite d’un modèle de domaine particulier. S’il s’avère qu’un microservice associe des modèles de domaines différents, vous devrez peut-être revenir en arrière et affiner votre analyse de domaine.

2. Ensuite, examinez les agrégats dans votre modèle de domaine. Les agrégats font souvent de bons candidats pour les microservices. Un agrégat bien conçu présente de nombreuses caractéristiques d’un microservice bien conçu. Entre autres :

    - Un agrégat est dérivé des besoins de l’entreprise, plutôt que des problèmes techniques, tels que les accès aux données ou la messagerie.  
    - Un agrégat doit avoir une cohésion fonctionnelle élevée.
    - Un agrégat est une limite de persistance.
    - Les agrégats doivent être couplés en laissant une marge de manœuvre. 
    
3. Les services de domaine font également de bons candidats pour les microservices. Les services de domaine sont des opérations sans état entre plusieurs agrégats. Un workflow impliquant plusieurs microservices en est un exemple type. Nous en verrons un exemple dans l’application Drone Delivery.

4. Enfin, tenez compte des exigences non fonctionnelles. Examinez les facteurs tels que la taille de l’équipe, les types de données, les technologies, ainsi que les exigences d’extensibilité, de disponibilité et de sécurité. Ces facteurs peuvent vous amener à décomposer un microservice en plusieurs services plus petits ou, à l’inverse, à combiner plusieurs microservices en un seul. 

Après avoir identifié le microservices dans votre application, vérifiez la conception en vous aidant des critères suivants :

- Chaque service a une seule responsabilité.
- Il n’existe aucun appel bavard entre les services. Si le fractionnement des fonctionnalités entre deux services entraîne trop de bavardage, cela peut indiquer que ces fonctions appartiennent au même service.
- Chaque service est assez petit pour permettre à une petite équipe travaillant indépendamment de le générer.
- Il n’existe aucune interdépendance qui nécessite le déploiement de plusieurs services avec un verrouillage. Il doit toujours être possible de déployer un service sans avoir à redéployer les autres services.
- Les services ne sont pas strictement couplés et peuvent évoluer indépendamment.
- Les limites de service ne créeront pas de problèmes liés à l’intégrité ou à la cohérence des données. Parfois, il est important de maintenir la cohérence des données en plaçant des fonctionnalités dans un seul microservice. Ceci dit, déterminez si vous avez vraiment besoin d’une forte cohérence. Il existe des stratégies de résolution de la cohérence éventuelle dans un système distribué, et les avantages de la décomposition des services l’emportent souvent sur les défis de gestion de la cohérence éventuelle.

Avant tout, il est important d’être pragmatique et de ne pas oublier que la conception orientée domaine est un processus itératif. En cas de doute, commencez avec des microservices généraux. Il est plus facile de fractionner un microservice en deux services plus petits que de refactoriser une fonctionnalité entre plusieurs microservices existants.
  
## <a name="drone-delivery-defining-the-microservices"></a>Drone Delivery : définition des microservices

Rappelez-vous que l’équipe de développement a identifié les quatre agrégats &mdash; Livraison, Colis, Drone et Compte &mdash; et deux services de domaine (le planificateur et le superviseur). 

Les agrégats Livraison et Colis sont des candidats évidents pour les microservices. Le planificateur et le superviseur coordonnent les activités effectuées par les autres microservices. Il est donc judicieux de mettre en œuvre ces services de domaine en tant que microservices.  

Les agrégats Drone et Compte sont intéressants, car ils appartiennent à d’autres contextes limités. Première option : le planificateur appelle directement les contextes limités Drone et Compte. Seconde option : créer des microservices Drone et Compte dans le contexte limité Expédition. Ces microservices serviraient de médiateurs entre les contextes limités, en exposant des API ou des schémas de données plus adaptés au contexte Expédition.

Les détails des contextes limités Drone et Compte dépassent le cadre de ce guide. Nous avons donc créé des services fictifs pour eux dans notre implémentation de référence. Mais voici quelques facteurs à prendre en compte dans cette situation :

- Quelle charge réseau représente un appel direct dans l’autre contexte limité ? 

- Le schéma de données pour l’autre contexte limité est-il approprié pour ce contexte, ou vaut-il mieux avoir un schéma sur mesure pour ce contexte limité ? 

- L’autre contexte limité est-il un système hérité ? Dans ce cas, vous pouvez créer un service qui agit comme une [couche anti-corruption](../patterns/anti-corruption-layer.md) pour traduire entre le système hérité et l’application moderne. 

- Quelle est la structure de l’équipe ? Est-il facile de communiquer avec l’équipe responsable de l’autre contexte limité ? Si ce n’est pas le cas, créez un service qui sert d’intermédiaire entre les deux contextes pour les frais de communication entre les équipes.

Jusqu'à présent, nous n’avons pas considéré les exigences non fonctionnelles. Avec à l’esprit les exigences de débit de l’application, l’équipe de développement décidé créer un microservice Ingestion distinct qui est chargé d’ingérer les requêtes des clients. Ce microservice implémentera le [nivellement de charge](../patterns/queue-based-load-leveling.md) en plaçant les requêtes entrantes dans une mémoire tampon en vue de leur traitement. Le planificateur lira les requêtes à partir de la mémoire tampon et exécutera le workflow. 

Les exigences non fonctionnelles ont conduit l’équipe à créer un service supplémentaire. À ce stade, tous les services concernent le processus de planification et de livraison des colis en temps réel. Mais le système doit stocker l’historique de chaque livraison dans un emplacement de stockage à long terme pour l’analyse de données. L’équipe a envisagé d’en donner la responsabilité au service Livraison. Toutefois, les exigences de stockage de données sont très différentes pour une analyse d’historique et des opérations en cours (voir [Considérations relatives aux données](./data-considerations.md)). Par conséquent, l’équipe a décidé de créer un service Historique des livraisons distinct, qui écoute les événements DeliveryTracking à partir du service Livraison et les écrit dans l’emplacement de stockage à long terme.

Le schéma suivant présente la conception à ce stade :
 
![](./images/microservices.png)

## <a name="choosing-a-compute-option"></a>Choix d'une option de calcul

Le terme *calcul* fait référence au modèle d’hébergement des ressources de calcul utilisées par votre application. Pour une architecture de microservices, deux approches sont particulièrement prisées :

- Orchestrateur de service qui gère les services s’exécutant sur des nœuds dédiés (machines virtuelles).
- Architecture sans serveur utilisant des fonctions en tant que service (FaaS). 

Bien qu’il ne s’agisse pas des seules options, ce sont deux approches éprouvées pour créer des microservices. Une application peut inclure les deux approches.

### <a name="service-orchestrators"></a>Orchestrateurs de services

Un orchestrateur gère les tâches liées au déploiement et à la gestion d’un ensemble de services. Ces tâches incluent le placement des services sur des nœuds, la surveillance de l’intégrité des services, le redémarrage des services non intègres, l’équilibrage de charge du trafic réseau entre les instances de service, la détection des services, la mise à l’échelle du nombre d’instances d’un service et l’application des mises à jour de configuration. Les orchestrateurs les plus courants sont Kubernetes, DC/OS, Docker Swarm et Service Fabric. 

- [Azure Container Service](/azure/container-service/) (ACS) est un service Azure qui permet de déployer un cluster Kubernetes, DC/OS ou Docker Swarm prêt pour la production.

- [AKS (Azure Container Service)](/azure/aks/) est un service managé Kubernetes. AKS configure Kubernetes et expose les points de terminaison d’API Kubernetes, mais il héberge et gère le plan de contrôle Kubernetes, en exécutant les mises à niveau automatisées, l’application automatisée de correctifs, la mise à l’échelle automatique et d’autres tâches de gestion. Vous pouvez considérer AKS comme « des API Kubernetes en tant que service ». Au moment de la rédaction de ce document, AKS est toujours en version préliminaire. Toutefois, AKS devrait devenir la méthode à privilégier pour exécuter Kubernetes dans Azure. 

- [Service Fabric](/azure/service-fabric/) est une plateforme de systèmes distribués pour le packaging, le déploiement et la gestion de microservices. Les microservices peuvent être déployés vers Service Fabric comme des conteneurs, des exécutables binaires ou des [Reliable Services](/azure/service-fabric/service-fabric-reliable-services-introduction). À l’aide du modèle de programmation des Reliable Services, les services peuvent directement utiliser les API de programmation Service Fabric pour interroger le système, vérifier l’intégrité, recevoir des notifications sur la configuration et les modifications de code, et détecter d’autres services. La singularité de Service Fabric tient au fait qu’il est axé sur la création de services avec état à l’aide de [Reliable Collections](/azure/service-fabric/service-fabric-reliable-services-reliable-collections).

### <a name="containers"></a>Containers

On fait parfois l’amalgame entre conteneurs et microservices. Or, il s’agit de deux choses différentes &mdash; vous n’avez pas besoin de conteneurs pour générer des microservices &mdash; et les conteneurs ont des avantages particulièrement intéressants pour les microservices, tels que :

- **Portabilité** : Une image de conteneur est un package autonome qui s’exécute sans nécessiter l’installation des bibliothèques ou autres dépendances. Cela la rend facile à déployer. Les conteneurs peuvent être démarrés et arrêtés rapidement, afin de lancer des instances pour gérer plus de charge ou pour effectuer une récupération après la défaillance d’un nœud. 

- **Densité**. Les conteneurs sont légers par rapport à l’exécution d’une machine virtuelle, car ils partagent les ressources du système d’exploitation. Cela permet de packager plusieurs conteneurs sur un seul nœud, ce qui est particulièrement utile lorsque l’application se compose de nombreux petits services.

- **Isolation des ressources**. Vous pouvez limiter la quantité de mémoire et d’UC disponible pour un conteneur, ce qui contribue à garantir qu’un processus de perte de contrôle n’épuise pas les ressources de l’hôte. Pour plus d’informations, consultez le [modèle de cloisonnement](../patterns/bulkhead.md).

### <a name="serverless-functions-as-a-service"></a>Sans serveur (fonctions en tant que service)

Avec une architecture sans serveur, vous ne gérez ni les machines virtuelles, ni l’infrastructure de réseau virtuel. Par contre, vous déployez le code et le service d’hébergement se charge de placer ce code sur une machine virtuelle et de l’exécuter. Cette approche a tendance à favoriser les petites fonctions granulaires coordonnées à l’aide de déclencheurs d’événements. Par exemple, un message placé dans une file d’attente peut déclencher une fonction qui lit le contenu de la file d’attente et traite le message.

[Azure Functions][functions] est un service de calcul sans serveur qui prend en charge plusieurs déclencheurs de fonction, y compris les requêtes HTTP, les files d’attente Service Bus et les événements Event Hubs. Pour obtenir la liste complète, voir [Concepts des déclencheurs et liaisons Azure Functions][functions-triggers]. Envisagez également [Azure Event Grid][event-grid], un service de routage d’événement managé dans Azure.

### <a name="orchestrator-or-serverless"></a>Orchestrateur ou sans serveur ?

Voici quelques éléments à prendre en compte lors du choix entre une approche d’orchestrateur et une approche sans serveur.

**Facilité de gestion** : une application sans serveur est facile à gérer, car la plate-forme gère toutes les ressources de calcul à votre place. Alors qu’un orchestrateur rend abstraits certains aspects de la gestion et de la configuration d’un cluster, il ne masque pas complètement les machines virtuelles sous-jacentes. Avec un orchestrateur, vous devez envisager des problèmes tels que l’équilibrage de charge, l’utilisation de l’UC et de la mémoire, et la mise en réseau.

**Flexibilité et contrôle** : un orchestrateur vous donne un grand contrôle sur la configuration et la gestion de vos services et du cluster. L’inconvénient est la complexité accrue. Avec une architecture sans serveur, vous perdez en contrôle, car ces détails sont rendus abstraits.

**Portabilité** : tous les orchestrateurs répertoriés ici (Kubernetes, DC/OS, Docker Swarm et Service Fabric) peuvent s’exécuter localement ou dans plusieurs clouds publics. 

**Intégration d’application** : il peut être difficile de créer une application complexe à l’aide d’une architecture sans serveur. Une option dans Azure consiste à utiliser [Azure Logic Apps](/azure/logic-apps/) pour coordonner un ensemble de fonctions Azure. Pour obtenir un exemple de cette approche, consultez [Créer une fonction qui s’intègre avec Azure Logic Apps](/azure/azure-functions/functions-twitter-email).

**Coût** : avec un orchestrateur, vous payez pour les machines virtuelles qui s’exécutent dans le cluster. Avec une application sans serveur, vous payez uniquement pour les ressources de calcul réellement consommées. Dans les deux cas, vous devez tenir compte du coût des services supplémentaires, tels que le stockage, les bases de données et les services de messagerie.

**Extensibilité**. Azure Functions est automatiquement mis à l’échelle afin de répondre à la demande, en fonction du nombre d’événements entrants. Avec un orchestrateur, vous pouvez augmenter le nombre d’instances de service en cours d’exécution dans le cluster. Vous pouvez également effectuer une mise à l’échelle en ajoutant des machines virtuelles au cluster.

Notre implémentation de référence utilise principalement Kubernetes, mais nous avons utilisé Azure Functions pour un service, à savoir le service Historique des livraisons. Azure Functions est bien adapté à ce service particulier, car il s’agit d’une charge de travail pilotée par des événements. En utilisant un déclencheur Event Hubs pour appeler la fonction, le service avait besoin d’une quantité minimale de code. En outre, le service Historique des livraisons ne fait pas partie du workflow principal. Son exécution en dehors du cluster Kubernetes n’a donc aucune incidence sur la latence de bout en bout des opérations initiées par l’utilisateur. 

> [!div class="nextstepaction"]
> [Considérations sur les données](./data-considerations.md)

<!-- links -->

[acs-engine]: https://github.com/Azure/acs-engine
[acs-faq]: /azure/container-service/dcos-swarm/container-service-faq
[event-grid]: /azure/event-grid/
[functions]: /azure/azure-functions/functions-overview
[functions-triggers]: /azure/azure-functions/functions-triggers-bindings
