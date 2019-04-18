# <a name="choosing-a-compute-option-for-microservices"></a>En choisissant une option de calcul pour les microservices

Le terme *calcul* fait référence au modèle d’hébergement des ressources de calcul utilisées par votre application. Pour une architecture de microservices, deux approches sont particulièrement prisées :

- Orchestrateur de service qui gère les services s’exécutant sur des nœuds dédiés (machines virtuelles).
- Architecture sans serveur utilisant des fonctions en tant que service (FaaS).

Bien qu’il ne s’agisse pas des seules options, ce sont deux approches éprouvées pour créer des microservices. Une application peut inclure les deux approches.

## <a name="service-orchestrators"></a>Orchestrateurs de services

Un orchestrateur gère les tâches liées au déploiement et à la gestion d’un ensemble de services. Ces tâches incluent le placement des services sur des nœuds, la surveillance de l’intégrité des services, le redémarrage des services non intègres, l’équilibrage de charge du trafic réseau entre les instances de service, la détection des services, la mise à l’échelle du nombre d’instances d’un service et l’application des mises à jour de configuration. Les orchestrateurs les plus courants sont Kubernetes, Service Fabric, DC/OS et Docker Swarm.

Sur la plateforme Azure, envisagez les options suivantes :

- [Azure Kubernetes Service](/azure/aks/) (AKS) est un service Kubernetes géré. AKS configure Kubernetes et expose les points de terminaison d’API Kubernetes, mais il héberge et gère le plan de contrôle Kubernetes, en exécutant les mises à niveau automatisées, l’application automatisée de correctifs, la mise à l’échelle automatique et d’autres tâches de gestion. Vous pouvez considérer AKS comme « des API Kubernetes en tant que service ».

- [Service Fabric](/azure/service-fabric/) est une plateforme de systèmes distribués pour le packaging, le déploiement et la gestion de microservices. Les microservices peuvent être déployés vers Service Fabric comme des conteneurs, des exécutables binaires ou des [Reliable Services](/azure/service-fabric/service-fabric-reliable-services-introduction). À l’aide du modèle de programmation des Reliable Services, les services peuvent directement utiliser les API de programmation Service Fabric pour interroger le système, vérifier l’intégrité, recevoir des notifications sur la configuration et les modifications de code, et détecter d’autres services. La singularité de Service Fabric tient au fait qu’il est axé sur la création de services avec état à l’aide de [Reliable Collections](/azure/service-fabric/service-fabric-reliable-services-reliable-collections).

- Autres options telles que Docker Enterprise Edition et Mesosphere DC/OS peuvent s’exécuter dans un environnement IaaS sur Azure. Vous pouvez trouver des modèles de déploiement sur [place de marché Azure](https://azuremarketplace.microsoft.com).

## <a name="containers"></a>Containers

On fait parfois l’amalgame entre conteneurs et microservices. Or, il s’agit de deux choses différentes &mdash; vous n’avez pas besoin de conteneurs pour générer des microservices &mdash; et les conteneurs ont des avantages particulièrement intéressants pour les microservices, tels que :

- **Portabilité** : Une image de conteneur est un package autonome qui s’exécute sans nécessiter l’installation des bibliothèques ou autres dépendances. Cela la rend facile à déployer. Les conteneurs peuvent être démarrés et arrêtés rapidement, afin de lancer des instances pour gérer plus de charge ou pour effectuer une récupération après la défaillance d’un nœud.

- **Densité**. Les conteneurs sont légers par rapport à l’exécution d’une machine virtuelle, car ils partagent les ressources du système d’exploitation. Cela permet de packager plusieurs conteneurs sur un seul nœud, ce qui est particulièrement utile lorsque l’application se compose de nombreux petits services.

- **Isolation des ressources**. Vous pouvez limiter la quantité de mémoire et d’UC disponible pour un conteneur, ce qui contribue à garantir qu’un processus de perte de contrôle n’épuise pas les ressources de l’hôte. Pour plus d’informations, consultez [Modèle de cloisonnement](../../patterns/bulkhead.md).

## <a name="serverless-functions-as-a-service"></a>Sans serveur (fonctions en tant que service)

Avec une architecture [sans serveur](https://azure.microsoft.com/solutions/serverless/), vous ne gérez ni les machines virtuelles, ni l’infrastructure de réseau virtuel. Par contre, vous déployez le code et le service d’hébergement se charge de placer ce code sur une machine virtuelle et de l’exécuter. Cette approche a tendance à favoriser les petites fonctions granulaires coordonnées à l’aide de déclencheurs d’événements. Par exemple, un message placé dans une file d’attente peut déclencher une fonction qui lit le contenu de la file d’attente et traite le message.

[Azure Functions](/azure/azure-functions/) est un service de calcul sans serveur qui prend en charge plusieurs déclencheurs de fonction, y compris les requêtes HTTP, les files d’attente Service Bus et les événements Event Hubs. Pour obtenir la liste complète, consultez [Azure Functions concepts des déclencheurs et liaisons](/azure/azure-functions/functions-triggers-bindings). Envisagez également [Azure Event Grid](/azure/event-grid/), qui est un service de routage d’événement managé dans Azure.

<!-- markdownlint-disable MD026 -->

## <a name="orchestrator-or-serverless"></a>Orchestrateur ou sans serveur ?

<!-- markdownlint-enable MD026 -->

Voici quelques éléments à prendre en compte lors du choix entre une approche d’orchestrateur et une approche sans serveur.

**Facilité de gestion** : une application sans serveur est facile à gérer, car la plate-forme gère toutes les ressources de calcul à votre place. Alors qu’un orchestrateur rend abstraits certains aspects de la gestion et de la configuration d’un cluster, il ne masque pas complètement les machines virtuelles sous-jacentes. Avec un orchestrateur, vous devez envisager des problèmes tels que l’équilibrage de charge, l’utilisation de l’UC et de la mémoire, et la mise en réseau.

**Flexibilité et contrôle** : un orchestrateur vous donne un grand contrôle sur la configuration et la gestion de vos services et du cluster. L’inconvénient est la complexité accrue. Avec une architecture sans serveur, vous perdez en contrôle, car ces détails sont rendus abstraits.

**Portabilité** : tous les orchestrateurs répertoriés ici (Kubernetes, DC/OS, Docker Swarm et Service Fabric) peuvent s’exécuter localement ou dans plusieurs clouds publics.

**Intégration d’application** : il peut être difficile de créer une application complexe à l’aide d’une architecture sans serveur. Une option dans Azure consiste à utiliser [Azure Logic Apps](/azure/logic-apps/) pour coordonner un ensemble de fonctions Azure. Pour obtenir un exemple de cette approche, consultez [Créer une fonction qui s’intègre avec Azure Logic Apps](/azure/azure-functions/functions-twitter-email).

**Coût** : avec un orchestrateur, vous payez pour les machines virtuelles qui s’exécutent dans le cluster. Avec une application sans serveur, vous payez uniquement pour les ressources de calcul réellement consommées. Dans les deux cas, vous devez tenir compte du coût des services supplémentaires, tels que le stockage, les bases de données et les services de messagerie.

**Extensibilité**. Azure Functions est automatiquement mis à l’échelle afin de répondre à la demande, en fonction du nombre d’événements entrants. Avec un orchestrateur, vous pouvez augmenter le nombre d’instances de service en cours d’exécution dans le cluster. Vous pouvez également effectuer une mise à l’échelle en ajoutant des machines virtuelles au cluster.

Notre implémentation de référence utilise principalement Kubernetes, mais nous avons utilisé Azure Functions pour un service, à savoir le service Historique des livraisons. Azure Functions est bien adapté à ce service particulier, car il s’agit d’une charge de travail pilotée par des événements. En utilisant un déclencheur Event Hubs pour appeler la fonction, le service avait besoin d’une quantité minimale de code. En outre, le service Historique des livraisons ne fait pas partie du workflow principal. Son exécution en dehors du cluster Kubernetes n’a donc aucune incidence sur la latence de bout en bout des opérations initiées par l’utilisateur.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Communication entre les services](./interservice-communication.md)
