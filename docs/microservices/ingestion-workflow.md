---
title: Ingestion et workflow dans les microservices
description: Ingestion et workflow dans les microservices
author: MikeWasson
ms.date: 10/23/2018
ms.openlocfilehash: 8a6d2d3209ca61e0588c96ed92862c1a7b91109f
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/08/2019
ms.locfileid: "54112802"
---
# <a name="designing-microservices-ingestion-and-workflow"></a>Conception de microservices : Ingestion et workflow

La plupart du temps, les microservices présentent un workflow qui s’étend sur plusieurs services pour une même transaction. Ce workflow doit être fiable ; autrement dit, il ne peut pas perdre de transactions ni les laisser dans un état incomplet. Il est également primordial de contrôler le taux d’ingestion des requêtes entrantes. Étant donné qu’une multitude de petits services communiquent entre eux, il existe un risque qu’une rafale de requêtes entrantes submerge la communication interservice.

![Schéma du workflow d’ingestion](./images/ingestion-workflow.png)

## <a name="the-drone-delivery-workflow"></a>Workflow de livraison par drone

Dans l’application de livraison par drone Drone Delivery, la planification d’une livraison requiert l’exécution des opérations suivantes :

1. Vérifiez l’état du compte du client (service Account).
2. Créez une entité de package (service Package).
3. Vérifiez si cette livraison requiert un transport tiers en fonction des emplacements de prélèvement et de livraison (service Third-party Transportation).
4. Planifiez un drone pour le prélèvement (service Drone).
5. Créez une entité de livraison (service Delivery).

Ce workflow constitue le fondement de toute l’application. Le processus de bout en bout doit donc se révéler aussi performant que fiable. Vous devez relever certains défis spécifiques :

- **Nivellement de charge**. Un nombre excessif de requêtes de client risque de submerger le système avec un trafic réseau interservice. Ce volume de requêtes est également susceptible de surcharger les dépendances du serveur principal, comme les services de stockage ou les services distants. Ces derniers peuvent réagir en limitant les services qui les appellent, engendrant ainsi une régulation de flux dans le système. Par conséquent, il est important de niveler la charge des requêtes qui entrent dans le système en plaçant ces dernières dans une mémoire tampon ou dans une file d’attente pour traitement.

- **Garantie de remise**. Pour éviter l’abandon d’une quelconque requête de client, le composant d’ingestion doit garantir le fait que les messages sont remis au moins une fois.

- **Gestion des erreurs**. Si l’un des services renvoie un code d’erreur ou rencontre un échec non temporaire, la livraison ne peut pas être planifiée. Un code d’erreur peut signaler une condition d’erreur attendue (par exemple, suspension du compte du client) ou une erreur de serveur inattendue (HTTP 5xx). Il est également possible qu’un service ne soit pas disponible et entraîne alors l’expiration de l’appel réseau.

Nous allons commencer par examiner le côté ingestion de l’équation, autrement dit, la façon dont le système peut ingérer les requêtes utilisateur entrantes à haut débit. Ensuite, nous étudierons la manière dont l’application de livraison par drone peut implémenter un workflow fiable. Il s’avère que la conception du sous-système d’ingestion affecte le serveur principal du workflow.

## <a name="ingestion"></a>Ingestion

En se basant sur les exigences métiers, l’équipe de développement a identifié les impératifs non fonctionnels ci-après en matière d’ingestion :

- débit soutenu de 10 000 requêtes par seconde ;
- capacité de prise en charge de pics atteignant jusqu’à 50 000 requêtes par seconde sans abandon de requêtes de client ni expiration du délai d’attente ;
- latence inférieure à 500 ms au 99e centile.

L’obligation de gérer les pics de trafic occasionnels représente un véritable défi de conception. Il serait théoriquement possible d’augmenter la taille des instances du système afin de prendre en charge le trafic attendu maximal. Toutefois, l’approvisionnement d’un tel volume de ressources se révélerait hautement inefficace. En effet, dans la majorité des cas, l’application n’aura pas besoin d’une telle capacité, de sorte que certains cœurs resteront inactifs, occasionnant ainsi des dépenses inutiles sans offrir de valeur ajoutée.

Une meilleure approche consiste à placer les requêtes entrantes dans une mémoire tampon et à permettre à cette dernière de niveler la charge. Dans le cadre de cette conception, le service Ingestion doit être en mesure de prendre en charge le taux d’ingestion maximal sur de courtes périodes, alors que les services principaux ont seulement besoin de gérer la charge soutenue maximale. Grâce à la mise en mémoire tampon au niveau du frontal, il n’est pas nécessaire que les services principaux prennent en charge les pics de trafic importants. À l’échelle requise pour l’application Drone Delivery, [Azure Event Hubs](/azure/event-hubs/) constitue un choix judicieux pour le nivellement de charge. Event Hubs offre une latence faible et un débit élevé, et constitue une solution rentable en cas de volumes d’ingestion élevés.

Pour nos tests, nous avons utilisé un Event Hub de niveau Standard avec 32 partitions et 100 unités de débit. Nous avons observé un taux d’ingestion de 32 000 événements/seconde environ, avec une latence approximative de 90 ms. Pour l’instant, la limite par défaut est de 20 unités de débit, mais les clients Azure peuvent demander des unités de débit supplémentaires en créant une demande de support. Pour plus d’informations, consultez l’article [Quotas Event Hubs](/azure/event-hubs/event-hubs-quotas). Comme pour toutes les mesures de performances, de nombreux facteurs peuvent affecter les performances, comme la taille de charge utile des messages ; par conséquent, n’interprétez pas ces valeurs comme un point de référence. Si un débit supplémentaire est nécessaire, le service Ingestion peut partitionner les données entre plusieurs Event Hubs. Pour des débits encore plus élevés, [Event Hubs Dedicated](/azure/event-hubs/event-hubs-dedicated-overview) offre des déploiements à locataire unique qui peuvent accepter plus de 2 millions d’événements par seconde.

Il est important de comprendre de quelle façon Event Hubs peut atteindre un débit aussi élevé, car ceci a une incidence sur le mode de consommation des messages d’Event Hubs par un client. Le service Event Hubs n’implémente pas de *file d’attente*. À la place, il met en œuvre un *flux d’événements*.

Dans le cadre de l’utilisation d’une file d’attente, un consommateur spécifique peut supprimer un message de la file d’attente, auquel cas le consommateur suivant ne verra pas ce message. Les files d’attente vous permettent donc d’utiliser un [modèle de consommateurs concurrents](../patterns/competing-consumers.md) pour traiter les messages en parallèle et pour améliorer l’extensibilité. En vue d’accroître la résilience, le consommateur maintient un verrou sur le message et libère ce verrou une fois qu’il a fini de traiter le message. Si le consommateur échoue &mdash; par exemple, si le nœud sur lequel il s’exécute cesse de fonctionner &mdash; le verrou expire et le message réintègre la file d’attente.

![Diagramme de la sémantique de file d’attente](./images/queue-semantics.png)

En revanche, Event Hubs utilise une sémantique de diffusion en continu. Les consommateurs lisent le flux de manière indépendante à leur propre rythme. Chaque consommateur est chargé d’assurer le suivi de sa position actuelle dans le flux. Un consommateur doit écrire sa position actuelle dans le stockage persistant à intervalle régulier prédéfini. De cette façon, si le consommateur rencontre une erreur (par exemple, blocage du consommateur ou échec de l’hôte), une nouvelle instance peut reprendre la lecture du flux à partir de la dernière position enregistrée. Ce processus est désigné sous le terme de *création de points de contrôle*.

Pour des raisons de performances, un consommateur ne crée généralement pas de point de contrôle après chaque message. À la place, il effectue cette opération à intervalle régulier, par exemple après le traitement de *n* messages ou toutes les *n* secondes. Par conséquent, en cas d’échec d’un consommateur, il est possible que certains événements soient traités deux fois, car une nouvelle instance repart toujours du dernier point de contrôle. Il existe un compromis : Des points de contrôle fréquents peuvent nuire aux performances, mais des points de contrôle trop rares nécessitent la relecture d’un plus grand nombre d’événements après un échec.

![Diagramme de la sémantique de flux](./images/stream-semantics.png)

Le service Event Hubs n’est pas conçu pour prendre en charge des consommateurs concurrents. Bien que plusieurs consommateurs puissent lire un flux, chacun d’eux parcourt le flux indépendamment. À la place, Event Hubs utilise un modèle de consommateur partitionné. Un Event Hub comporte jusqu’à 32 partitions. La mise à l’échelle horizontale est obtenue par l’attribution d’un consommateur distinct à chaque partition.

Qu’est-ce que cela signifie pour le workflow de livraison par drone ? Pour tirer pleinement parti d’Event Hubs, le service Delivery Scheduler ne peut pas attendre que chaque message soit traité avant de passer au message suivant. S’il procède de cette façon, il passera la plupart de son temps à attendre que les appels réseau soient terminés. À la place, il doit traiter des lots de messages en parallèle à l’aide d’appels asynchrones vers les services principaux. Comme nous allons le voir, il est également important de choisir une stratégie de création de points de contrôle adéquate.

## <a name="workflow"></a>Workflow

Nous avons examiné trois options pour la lecture et le traitement des messages : hôte du processeur d’événements, files d’attente Service Bus et bibliothèque IoTHub React. Nous avons opté pour IoTHub React, mais pour que vous en compreniez la raison, il est utile de commencer par examiner l’option d’utilisation de l’hôte du processeur d’événements.

### <a name="event-processor-host"></a>Hôte du processeur d’événements

L’hôte du processeur d’événements est conçu pour le traitement de messages par lot. L’application implémente l’interface `IEventProcessor`, et l’hôte du processeur crée une instance de processeur d’événements pour chaque partition dans l’Event Hub. Ensuite, l’hôte du processeur d’événements appelle la méthode `ProcessEventsAsync` de chaque processeur d’événements avec les lots de messages d’événement. L’application détermine quand créer des points de contrôle dans la méthode `ProcessEventsAsync`, et l’hôte du processeur d’événements écrit ces points de contrôle dans le stockage Azure.

Dans une partition, l’hôte du processeur d’événements attend que la méthode `ProcessEventsAsync` soit renvoyée avant de l’appeler de nouveau avec le lot suivant. Cette approche simplifie le modèle de programmation, car votre code de traitement des événements n’a pas besoin d’être réentrant. Toutefois, cela signifie également que le processeur d’événements ne gère qu’un lot à la fois, et cela restreint la vitesse à laquelle l’hôte du processeur peut pomper les messages.

> [!NOTE]
> L’hôte du processeur *n’attend* pas à proprement parler, dans le sens où il bloquerait un thread. La méthode `ProcessEventsAsync` est asynchrone, de sorte que l’hôte du processeur peut accomplir d’autres tâches pendant que la méthode s’exécute. Mais il ne remettra aucun autre lot de messages pour cette partition tant que la méthode n’aura pas été renvoyée.

Dans l’application de livraison par drone, un lot de messages peut être traité en parallèle. Toutefois, l’attente du traitement de la totalité du lot est toujours susceptible d’entraîner un goulot d’étranglement. La vitesse du traitement est tributaire de celle du message le plus lent d’un lot. Toute variation des temps de réponse peut engendrer une « longue file » dans le cadre de laquelle quelques réponses lentes ralentissent la totalité du système. Nos tests de performances ont montré que cette approche ne nous permettait pas d’atteindre notre débit cible. Cela ne signifie *pas* qu’il vous faille éviter d’utiliser l’hôte du processeur d’événements. Mais dans le cas d’un débit élevé, évitez d’exécuter des tâches de longue durée à l’intérieur de la méthode `ProcesssEventsAsync`. Traitez chaque lot rapidement.

### <a name="iothub-react"></a>IotHub React

[IotHub React](https://github.com/Azure/toketi-iothubreact) est une bibliothèque Akka Streams pour la lecture d’événements issus d’Event Hubs. Akka Streams est une infrastructure de programmation basée sur des flux qui implémente la spécification [Reactive Streams](https://www.reactive-streams.org/). Elle permet de générer des pipelines de diffusion en continu efficaces, dans le cadre desquels toutes les opérations de diffusion en continu sont exécutées de manière asynchrone, et où les pipelines gèrent la régulation de flux de manière appropriée. Une régulation de flux se produit lorsqu’une source d’événements produit des événements plus rapidement que les consommateurs en aval ne peuvent les recevoir &mdash; ce qui est précisément le cas lorsque le système de livraison par drone rencontre un pic de trafic. Si les services principaux fonctionnent moins rapidement, IoTHub React ralentira. Si la capacité est accrue, IoTHub React transmettra (push) davantage de messages par le biais du pipeline.

Akka Streams constitue également un modèle de programmation très naturel pour la diffusion en continu d’événements à partir d’Event Hubs. Au lieu d’effectuer une boucle sur un lot d’événements, vous définissez un ensemble d’opérations qui seront appliquées à chaque événement, et vous laissez Akka Streams gérer la diffusion en continu. Akka Streams définit un pipeline de diffusion en continu en termes de *sources*, de *flux* et de *récepteurs*. Une source génère un flux de sortie, un flux traite un flux d’entrée et produit un flux de sortie, et un récepteur consomme un flux sans produire de sortie.

Voici le code du service Scheduler qui configure le pipeline Akka Streams :

```java
IoTHub iotHub = new IoTHub();
Source<MessageFromDevice, NotUsed> messages = iotHub.source(options);

messages.map(msg -> DeliveryRequestEventProcessor.parseDeliveryRequest(msg))
        .filter(ad -> ad.getDelivery() != null).via(deliveryProcessor()).to(iotHub.checkpointSink())
        .run(streamMaterializer);
```

Ce code configure Event Hubs en tant que source. L’instruction `map` désérialise chaque message d’événement dans une classe Java qui représente une requête de livraison. L’instruction `filter` supprime tous les objets `null` du flux ; ceci évite les situations dans lesquelles un message ne peut pas être désérialisé. L’instruction `via` joint la source à un flux qui traite chaque requête de livraison. La méthode `to` joint le flux au récepteur de points de contrôle, qui est intégré à IoTHub React.

IoTHub React utilise une autre stratégie de points de contrôle que l’hôte du processeur d’événements. Les points de contrôle sont écrits par le récepteur de points de contrôle, qui constitue la phase finale du pipeline. La conception d’Akka Streams permet au pipeline de poursuivre la diffusion en continu des données pendant que le récepteur écrit le point de contrôle. Cela signifie que les phases de traitement en amont n’ont pas besoin d’attendre que des points de contrôle surviennent. Vous pouvez configurer la création de points de contrôle pour qu’elle se produise après un délai d’expiration ou après le traitement d’un certain nombre de messages.

La méthode `deliveryProcessor` crée le flux Akka Streams :

```java
private static Flow<AkkaDelivery, MessageFromDevice, NotUsed> deliveryProcessor() {
    return Flow.of(AkkaDelivery.class).map(delivery -> {
        CompletableFuture<DeliverySchedule> completableSchedule = DeliveryRequestEventProcessor
                .processDeliveryRequestAsync(delivery.getDelivery(),
                        delivery.getMessageFromDevice().properties());

        completableSchedule.whenComplete((deliverySchedule,error) -> {
            if (error!=null){
                Log.info("failed delivery" + error.getStackTrace());
            }
            else{
                Log.info("Completed Delivery",deliverySchedule.toString());
            }

        });
        completableSchedule = null;
        return delivery.getMessageFromDevice();
    });
}
```

Le flux appelle une méthode `processDeliveryRequestAsync` statique qui procède au traitement proprement dit de chaque message.

### <a name="scaling-with-iothub-react"></a>Mise à l’échelle avec IoTHub React

Le service Scheduler est conçu pour que chaque instance de conteneur lise les événements à partir d’une seule partition. Par exemple, si l’Event Hub comporte 32 partitions, le service Scheduler est déployé avec 32 réplicas. Ceci offre une grande flexibilité en termes de mise à l’échelle horizontale.

Selon la taille du cluster, il est possible que plusieurs pods du service Scheduler s’exécutent sur un même nœud du cluster. Mais si le service Scheduler a besoin de ressources supplémentaires, vous pouvez augmenter la taille des instances dans le cluster afin de répartir les pods entre un plus grand nombre de nœuds. Nos tests de performances ont démontré que le service Scheduler est lié à la mémoire et aux threads ; par conséquent, les performances dépendaient considérablement de la taille de la machine virtuelle et du nombre de pods par nœud.

Chaque instance a besoin de connaître la partition Event Hubs à partir de laquelle elle doit lire les événements. Pour configurer le nombre de partitions, nous avons tiré avantage du type de ressource [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) dans Kubernetes. Les pods d’une ressource StatefulSet comportent un identificateur persistant incluant un index numérique. Plus précisément, le nom du pod correspond à `<statefulset name>-<index>`, et cette valeur est mise à la disposition du conteneur par le biais de [l’API Downward](https://kubernetes.io/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/) Kubernetes. Au moment de l’exécution, le service Scheduler lit le nom du pod et utilise l’index du pod en tant qu’ID de partition.

Si vous avez besoin d’augmenter encore davantage la taille des instances du service Scheduler, vous pouvez attribuer plusieurs pods par partition Event Hub, afin que chaque partition soit lue par différents pods. Toutefois, dans ce cas, chaque instance lira tous les événements de la partition attribuée. Pour éviter tout traitement en double, vous devrez donc utiliser un algorithme de hachage afin que chaque instance ignore une partie des messages. De cette façon, plusieurs lecteurs peuvent consommer le flux, mais chaque message n’est traité que par une seule instance.

![Diagramme du hachage de hub d’événements](./images/eventhub-hashing.png)

### <a name="service-bus-queues"></a>Files d’attente Service Bus

La troisième option que nous avons envisagée consistait à copier les messages d’Event Hubs dans une file d’attente Service Bus, puis à faire en sorte que le service Scheduler lise les messages à partir de Service Bus. Il peut sembler étrange d’écrire les requêtes entrantes dans Event Hubs uniquement dans le but de les copier dans Service Bus.  Toutefois, l’idée était de tirer parti des différentes forces de chaque service : L’utilisation d’Event Hubs permet d’absorber les pics de trafic dense, tout en tirant parti de la sémantique de file d’attente de Service Bus pour traiter la charge de travail avec un modèle de consommateurs concurrents. Souvenez-vous que le débit soutenu que nous ciblons est inférieur à notre charge maximale attendue ; par conséquent, le traitement de la file d’attente Service Bus n’a pas besoin d’être aussi rapide que l’ingestion des messages.

Dans le cadre de cette approche, notre implémentation de preuve de concept a atteint environ 4 000 opérations par seconde. Ces tests portaient sur des services principaux fictifs qui n’accomplissaient aucune tâche réelle, mais ajoutaient simplement un temps de latence fixe par service. Notez que nos valeurs de performances étaient sensiblement inférieures à la valeur maximale théorique pour Service Bus. Les raisons possibles de cet écart sont les suivantes :

- utilisation de valeurs non optimales pour différents paramètres de client, tels que la limite du pool de connexions, le degré de parallélisation, le nombre de prérécupérations et la taille de lot ;

- goulots d’étranglement des E/S réseau ;

- utilisation du mode [PeekLock](/rest/api/servicebus/peek-lock-message-non-destructive-read) au lieu du mode [ReceiveAndDelete](/rest/api/servicebus/receive-and-delete-message-destructive-read), qui était nécessaire pour garantir le fait que les messages avaient été remis au moins une fois.

L’exécution de tests de performances supplémentaires aurait pu nous aider à découvrir la cause racine de ces problèmes et à résoudre ces derniers. Toutefois, étant donné qu’IotHub React nous a permis d’atteindre notre niveau de performance cible, nous avons opté pour cette solution. Cela dit, Service Bus constitue une option viable pour ce scénario.

## <a name="handling-failures"></a>Gestion des échecs

Il existe trois classes générales d’échec à prendre en compte.

1. Un service en aval peut présenter un échec non temporaire, autrement dit, peu susceptible de disparaître de lui-même. Les échecs non temporaires comprennent les conditions d’erreur normales, telles qu’une entrée non valide dans une méthode. Les échecs non temporaires englobent également les exceptions non prises en charge dans le code d’application ou le blocage d’un processus. Si ce type d’erreur se produit, la totalité de la transaction métier doit être marquée en tant qu’échec. Il peut se révéler nécessaire d’annuler d’autres étapes réussies de la même transaction. (Voir la section « Transactions de compensation » ci-dessous.)

2. Un service en aval peut rencontrer un échec temporaire, tel qu’un délai d’expiration réseau. Il est souvent possible de résoudre ces erreurs en procédant simplement à une nouvelle tentative d’appel. Si l’opération continue d’échouer après un certain nombre de tentatives, elle est considérée comme un échec non temporaire.

3. Le service Scheduler proprement dit peut faire l’objet d’une défaillance (par exemple, en raison du blocage d’un nœud). Dans ce cas, Kubernetes présentera une nouvelle instance du service. Toutefois, toutes les transactions qui étaient déjà en cours d’exécution au moment de l’incident devront être reprises.

## <a name="compensating-transactions"></a>Transactions de compensation

Si un échec non temporaire se produit, la transaction actuelle peut se trouver dans un état *d’échec partiel*, dans lequel une ou plusieurs étapes ont déjà été correctement exécutées. Par exemple, si le service Drone avait déjà planifié un drone, le drone doit être annulé. Dans ce cas, l’application doit annuler les étapes qui ont réussi en utilisant une [transaction de compensation](../patterns/compensating-transaction.md). Dans certains cas, cette opération doit être effectuée par un système externe ou même par un processus manuel.

Si la logique des transactions de compensation se révèle complexe, envisagez de créer un service distinct qui sera responsable de ce processus. Dans l’application Drone Delivery, le service Scheduler place les opérations ayant échoué dans une file d’attente dédiée. Un microservice distinct, appelé Supervisor, lit les données de cette file d’attente et appelle une API d’annulation sur les services qui nécessitent une compensation. Ceci constitue une variante du [modèle de superviseur de l’agent du planificateur][scheduler-agent-supervisor]. Le service Supervisor peut également exécuter d’autres actions, telles qu’avertir l’utilisateur par texte ou par e-mail, ou envoyer une alerte à un tableau de bord des opérations.

![Diagramme illustrant le microservice Supervisor](./images/supervisor.png)

## <a name="idempotent-vs-non-idempotent-operations"></a>Opérations idempotentes et non idempotentes

Pour éviter toute perte de requêtes, le service Scheduler doit garantir que tous les messages sont traités au moins une fois. Event Hubs peut garantir que les messages ont été remis au moins une fois si le client applique une stratégie de points de contrôle adéquate.

Il est possible que le service Scheduler cesse de fonctionner alors qu’il était en train de traiter une ou plusieurs requêtes de client. Dans ce cas, les messages correspondants seront prélevés par une autre instance du service Scheduler et feront l’objet d’un nouveau traitement. Que se passe-t-il si une requête est traitée deux fois ? Il est important d’éviter tout travail en double. En effet, nous ne voulons pas que le système envoie deux drones pour le même package.

Une approche possible consiste à concevoir toutes les opérations comme étant idempotentes. Une opération est idempotente si elle peut être appelée plusieurs fois sans produire d’effets secondaires supplémentaires après le premier appel. En d’autres termes, le résultat sera le même, qu’un client appelle l’opération une fois, deux fois ou à différentes reprises. Fondamentalement, le service doit ignorer les appels en double. Pour qu’une méthode avec des effets secondaires soit idempotente, le service doit être en mesure de détecter les appels en double. Par exemple, vous pouvez faire en sorte que l’appelant attribue l’ID, au lieu que ce soit le service qui génère un nouvel ID. Le service peut alors rechercher l’existence d’ID en double.

> [!NOTE]
> La spécification HTTP stipule que les méthodes GET, PUT et DELETE doivent être idempotentes. Le caractère idempotent des méthodes POST n’est pas garanti. Si une méthode POST crée une ressource, il n’existe généralement aucune garantie que cette opération soit idempotente.

Il n’est pas toujours simple d’écrire une méthode idempotente. Une autre possibilité consiste à faire en sorte que le service Scheduler suive la progression de chaque transaction dans un magasin durable. Chaque fois qu’il traite un message, il recherche l’état dans ce magasin durable. Au terme de chaque étape, il écrit le résultat correspondant dans le magasin. Cette approche peut avoir des répercussions sur les performances.

## <a name="example-idempotent-operations"></a>Exemple : Opérations idempotentes

La spécification HTTP stipule que les méthodes PUT doivent être idempotentes. La spécification définit le terme « idempotent » de cette façon :

> Une méthode de demande est dite « idempotente » si l’exécution de plusieurs requêtes identiques à l’aide de cette méthode est censée produire le même effet sur le serveur que l’exécution d’une seule de ces requêtes. ([RFC 7231](https://tools.ietf.org/html/rfc7231#section-4))

Il est important de bien comprendre la différence entre les sémantiques PUT et POST lors de la création d’une entité. Dans les deux cas, le client envoie une représentation d’une entité dans le corps de la requête. Toutefois, la signification de l’URI diffère.

- Dans le cas d’une méthode POST, l’URI représente une ressource parente de la nouvelle entité, telle qu’une collection. Par exemple, l’URI utilisé pour la création d’une livraison pourrait être `/api/deliveries`. Le serveur crée l’entité et lui attribue un nouvel URI, tel que `/api/deliveries/39660`. Cet URI est renvoyé dans l’en-tête Location de la réponse. Chaque fois que le client envoie une requête, le serveur crée une entité avec un nouvel URI.

- Dans le cas d’une méthode PUT, l’URI identifie l’entité. Si une entité présente déjà cet URI, le serveur remplace l’entité existante par la version dans la requête. Si aucune entité ne présente cet URI, le serveur en crée une. Par exemple, supposons que le client envoie une requête PUT à `api/deliveries/39660`. Si aucune livraison ne présente cet URI, le serveur en crée une. Si le client envoie de nouveau la même requête par la suite, le serveur remplacera l’entité existante.

Voici l’implémentation de la méthode PUT dans le service Delivery.

```csharp
[HttpPut("{id}")]
[ProducesResponseType(typeof(Delivery), 201)]
[ProducesResponseType(typeof(void), 204)]
public async Task<IActionResult> Put([FromBody]Delivery delivery, string id)
{
    logger.LogInformation("In Put action with delivery {Id}: {@DeliveryInfo}", id, delivery.ToLogInfo());
    try
    {
        var internalDelivery = delivery.ToInternal();

        // Create the new delivery entity.
        await deliveryRepository.CreateAsync(internalDelivery);

        // Create a delivery status event.
        var deliveryStatusEvent = new DeliveryStatusEvent { DeliveryId = delivery.Id, Stage = DeliveryEventType.Created };
        await deliveryStatusEventRepository.AddAsync(deliveryStatusEvent);

        // Return HTTP 201 (Created)
        return CreatedAtRoute("GetDelivery", new { id= delivery.Id }, delivery);
    }
    catch (DuplicateResourceException)
    {
        // This method is mainly used to create deliveries. If the delivery already exists then update it.
        logger.LogInformation("Updating resource with delivery id: {DeliveryId}", id);

        var internalDelivery = delivery.ToInternal();
        await deliveryRepository.UpdateAsync(id, internalDelivery);

        // Return HTTP 204 (No Content)
        return NoContent();
    }
}
```

En principe, la plupart des requêtes créeront une entité ; par conséquent, la méthode anticipe les choses en appelant `CreateAsync` sur l’objet de référentiel, puis traite toutes les exceptions de ressources en double en mettant à jour la ressource à la place.

> [!div class="nextstepaction"]
> [Passerelles d’API](./gateway.md)

<!-- links -->

[scheduler-agent-supervisor]: ../patterns/scheduler-agent-supervisor.md