---
title: Analyse du domaine pour les microservices
description: Analyse du domaine pour les microservices
author: MikeWasson
ms.date: 12/08/2017
ms.openlocfilehash: c3c353a6b30507369357af4b520a51f8afc2fb8d
ms.sourcegitcommit: 3d6dba524cc7661740bdbaf43870de7728d60a01
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/11/2018
---
# <a name="designing-microservices-domain-analysis"></a>Conception des microservices : Analyse du domaine 

L’un des plus grandes problématiques des microservices est la définition des limites de chacun des services isolés. Le principe de base est qu’un service doit exécuter « une action » &mdash; ; toutefois, l’application concrète de cette règle nécessite une réflexion approfondie. Aucun processus mécanique ne produira la conception idéale. Vous devez analyser en profondeur le domaine, les exigences et les objectifs de votre entreprise. Sinon, vous pouvez avoir à gérer une conception incohérente présentant des caractéristiques indésirables, comme des dépendances masquées entre les services, un couplage étroit ou des interfaces mal développées. Dans ce chapitre, nous appliquons une approche orientée domaine de la conception des microservices. 

Les microservices doivent être alignés sur les activités commerciales, pas développés comme des couches horizontales de données, telles que l’accès aux données ou la messagerie. Par ailleurs, ils doivent présenter un couplage souple et une haute cohérence fonctionnelle. Les microservices sont *couplés souplement* si vous êtes en mesure de modifier un service sans avoir à mettre à jour d’autres services au même moment. Un microservice est *cohérent* s’il a une vocation unique et précisément définie, comme la gestion des comptes utilisateurs ou le suivi de l’historique de livraison. Un service a vocation à encapsuler la connaissance du domaine et à extraire cette connaissance des clients. Par exemple, un client doit pouvoir programmer un drone sans connaître en détail l’algorithme de programmation ni maîtriser la gestion de la flotte de drones.

La conception pilotée par domaine fournit une infrastructure constituant la base d’un ensemble efficacement organisé de microservices. Cette approche comporte deux phases distinctes, l’une stratégique, l’autre tactique. Dans la phase stratégique de la conception pilotée par domaine, vous définissez la structure à grande échelle du système. Cette phase garantit que votre architecture demeure axée sur les fonctionnalités commerciales. La phase tactique, quant à elle, offre un jeu de modèles de conception à valoriser pour créer le modèle du domaine. Ces modèles comprennent les entités, les agrégats et les services de domaine. Ces modèles tactiques vous aident à concevoir des microservices à la fois cohérents et souplement couplés.

![](./images/ddd-process.png)

Dans ce chapitre et dans le suivant, nous allons nous intéresser aux phases suivantes, en les exécutant avec l’application Drone Delivery : 

1. Commencez par analyser le domaine d’entreprise afin d’appréhender les exigences fonctionnelles de l’application. Cette étape produit une description informelle du domaine, qui peut être affinée en un jeu plus formel de modèles de domaine. 

2. Ensuite, définissez les *limites de contexte* du domaine. Chaque limite de contexte comporte un modèle de domaine représentant un sous-domaine spécifique de l’application dans son ensemble. 

3. Au sein d’une limite de contexte, appliquez des modèles tactiques de la conception pilotée par domaine afin de définir les entités, les agrégats et les services de domaine. 
 
4. Utilisez les résultats de l’étape précédente pour identifier les microservices dans votre application.

Dans ce chapitre, nous abordons les trois premières étapes, qui sont principalement liées à la conception pilotée par domaine. Dans le chapitre suivant, nous identifierons les microservices. Toutefois, il est essentiel de se rappeler que la conception pilotée par domaine est un processus continu, itératif. Les limites de services ne sont pas fixes. À mesure de l’évolution d’une application, vous pouvez décider de décomposer un service en sous-services de taille plus réduite.

> [!NOTE]
> Ce chapitre n’a pas vocation à apporter une analyse complète et exhaustive du domaine. Nous tâcherons d’apporter un exemple concis, illustrant les points principaux. Pour plus d’informations sur la conception pilotée par domaine, nous vous recommandons de lire l’ouvrage d’Eric Evans, *Domain-Driven Design* (Conception pilotée par domaine), dans lequel le concept fut pour la première fois évoqué. Une autre référence importante est le livre *Implementing Domain-Driven Design* (Implémentation de la conception pilotée par domaine) de Vaughn Vernon. 

## <a name="analyze-the-domain"></a>Analyser le domaine

L’application d’une approche de conception pilotée par domaine vous permet de configurer des microservices de manière à ce que chacune des composantes constitue une niche dédiée à une exigence fonctionnelle de l’entreprise. Cela peut vous aider à éviter le piège consistant à développer une conception dictée par les limites organisationnelles ou les choix technologiques.

Avant d’écrire le code, vous devez bénéficier d’une visibilité exhaustive sur le système en cours de création. La conception pilotée par domaine vise dans un premier temps à modeler le domaine d’entreprise et à créer un *modèle de domaine*. Le modèle de domaine est un modèle théorique du modèle d’entreprise. Il répartit et organise la connaissance du domaine, en fournissant un langage commun pour les développeurs et les experts du domaine. 

Commencez par mapper l’ensemble des fonctions d’entreprise et leurs connexions. Il s’agira probablement d’un travail collaboratif impliquant les experts du domaine, les architectes logiciels et d’autres parties prenantes. Il n’est pas nécessaire de recourir à aucun formalisme particulier.  Dessinez un schéma ou établissez une esquisse sur tableau blanc.

À mesure que vous enrichissez le schéma, vous pouvez commencer à identifier des sous-domaines distincts. Quelles fonctions sont étroitement liées ? Quelles sont les fonctions au cœur de l’entreprise, quelles sont celles prenant en charge les services auxiliaires ? À quoi ressemble le graphique de dépendance ? Durant cette phase initiale, vous ne vous intéressez pas aux technologies ni aux détails de l’implémentation. Ceci dit, vous devez identifier l’emplacement d’intégration de l’application avec les systèmes externes, tel que le processus CRM, le système de traitement des paiements ou les architectures de facturation. 

## <a name="drone-delivery-analyzing-the-business-domain"></a>Drone Delivery : Analyse du domaine d’entreprise.

Après quelques activités d’analyse initiale de domaine, l’équipe Fabrikam est parvenue à établir une esquisse brute décrivant le domaine de l’application Drone Delivery.

![](./images/ddd1.svg) 

- L’**Expédition**, au cœur de l’activité, est placée au centre du schéma. Elle est tributaire de l’ensemble des autres éléments du schéma.
- La **Gestion des drones** est également cruciale. Une fonctionnalité étroitement liée à la gestion des drones est la **réparation des drones** ; combinée à l’**analyse prédictive**, elle permet d’établir un échéancier des périodes d’entretien et de maintenance des drones. 
- L’**analyse de l’heure prévue** fournit des estimations des horaires de collecte et de livraison. 
- Le modèle de **transport tiers** permet à l’application de planifier des modes de transport alternatifs en cas d’incapacité du drone à livrer l’intégralité d’un colis.
- Le **partage de drones** est une extension possible de l’activité principale. La société peut disposer d’une capacité supérieure de drones sur des intervalles spécifiques, et louer des appareils qui seraient inutilisés sans sa sollicitation. Cette fonctionnalité ne sera pas disponible dans la version initiale.
- La **surveillance vidéo** est un autre secteur que l’entreprise pourrait choisir de développer à l’avenir.
- Les **comptes utilisateurs**, la **facturation** et le **centre d’appels** sont des sous-domaines qui prennent en charge l’activité fondamentale.
 
Notez qu’à ce stade du processus, nous n’avons encore pris aucune décision ayant trait à l’implémentation ou aux technologies. Certains des sous-systèmes peuvent recourir à des systèmes logiciels externes ou des services tiers. Même dans ce cas, l’application doit interagir avec ces systèmes et services. Aussi, il est important de les inclure dans le modèle de domaine. 

> [!NOTE]
> Lorsqu’une application dépend d’un système externe, il existe un risque que le schéma de données ou l’API de ce dernier soient répercutés dans votre application, et compromettent le cas échéant la conception architecturale. Ce cas de figure est recensé en particulier avec les systèmes hérités ne respectant pas forcément les meilleures pratiques modernes et pouvant valoriser des schémas de données complexes ou des API obsolètes. Dans ce cas, il est important de définir une limite précise entre ces systèmes externes et l’application. Ici, vous pouvez envisager le recours au [modèle d’étranglement](../patterns/strangler.md) ou au [modèle de couche de lutte contre la corruption](../patterns/anti-corruption-layer.md).

## <a name="define-bounded-contexts"></a>Définir les limites de contexte

Le modèle de domaine comprend des représentations des composantes réelles (utilisateurs, drones, colis, etc.). Cela ne signifie pas pour autant que chacune des portions du système doive utiliser la même représentation pour une composante donnée. 

Par exemple, les sous-systèmes dédiés à la réparation des drones et à l’analyse prédictive doivent représenter de nombreuses caractéristiques physiques des drones, comme l’historique de maintenance, le kilométrage, l’âge, le numéro de modèle, les caractéristiques de performance, etc. Toutefois, lors de la planification d’une livraison, ces éléments importent peu. Le sous-système de planification doit simplement connaître la disponibilité des drones, ainsi que les horaires prévus de collecte et de livraison. 

Si nous essayions de développer un modèle unique pour chacun de ces sous-systèmes, cela pourrait s’avérer inutilement complexe. Par ailleurs, l’évolution au fil du temps du modèle serait ralentie, dans la mesure où les modifications devraient être validées par plusieurs équipes travaillant sur des sous-systèmes séparés. Par conséquent, il est souvent préférable de concevoir des modèles séparés qui représentent la même entité réelle (dans ce cas, un drone) dans deux contextes différents. Chaque modèle comporte les fonctions et attributs qui sont nécessaires au sein du contexte considéré.

C’est là que le concept de *limites de contexte* de la conception pilotée par domaine entre en jeu. Une limite de contexte correspond tout simplement au périmètre, au sein d’un domaine, dans lequel s’applique un modèle de domaine particulier. En considérant le schéma précédent, nous pouvons rassembler les fonctionnalités par modèle de domaine. 

![](./images/ddd2.svg) 
 
Les limites de contexte ne sont pas forcément isolées les unes des autres. Dans ce schéma, les lignes pleines reliant les limites de contexte représentent les points d’interaction entre deux limites de contexte. Par exemple, l’Expédition dépend des Comptes utilisateur au niveau de la collecte des informations sur les clients, et de la Gestion des drones en matière de sollicitation des drones disponibles de la flotte.

Dans son ouvrage *Domain Driven Design* (Conception pilotée par domaine), Eric Evans décrit plusieurs approches utilisées pour maintenir l’intégrité d’un modèle de domaine interagissant avec une autre limite de contexte. L’un des principes fondamentaux des microservices concerne la communication des services, qui s’effectue via des API précisément définies. Cette approche correspond à deux modèles que M. Evans appelle Open Host Service (Service hôte ouvert) et Published Language (Langage publié). L’idée derrière le Service hôte ouvert, c’est qu’un sous-système définit un protocole formel (API) valorisé par les autres sous-systèmes souhaitant communiquer avec lui. Le modèle de Langage publié prolonge cette philosophie. Il consiste en la publication de l’API dans une forme que d’autres équipes peuvent utiliser pour écrire aux clients. Dans le chapitre sur la [Conception d’API](./api-design.md), nous évoquons l’utilisation de la [Spécification OpenAPI](https://www.openapis.org/specification/repo) (auparavant connue en tant que Swagger) pour définir les descriptions d’interface indépendantes du langage pour les API REST, exprimées au format JSON ou YAML.

Pour le reste de cette activité, nous nous intéresserons à la limite de contexte Expédition. 

## <a name="tactical-ddd"></a>Phase tactique de la conception pilotée par domaine

Pendant la phase stratégique de la conception, vous mappez le domaine d’entreprise et définissez les limites de contexte de vos modèles de domaine. La phase tactique consiste à définir vos modèles de domaine avec davantage de précision. Les modèles tactiques sont appliqués au sein d’une seule limite de contexte. Au sein d’une architecture de microservices, nous nous intéressons particulièrement aux modèles d’entités et d’agrégats. En appliquant ces modèles, nous nous donnons les moyens d’identifier les limites naturelles des services au sein de notre application (voir le [chapitre suivant](./microservice-boundaries.md)). En règle générale, un microservice doit être plus petit qu’un agrégat, et pas plus grand qu’une limite de contexte. Nous allons dans un premier temps examiner les modèles tactiques. Ensuite, nous les appliquerons à la limite du contexte Expédition dans l’application Drone Delivery. 

### <a name="overview-of-the-tactical-patterns"></a>Vue d’ensemble des modèles tactiques

Cette section propose une brève synthèse des modèles tactiques de la conception pilotée par domaine. Aussi, si vous maîtrisez bien cette solution, vous pouvez passer cette section. Les modèles sont décrits en détail dans les chapitres 5 &ndash; 6 de l’ouvrage d’Eric Evans, et dans le livre *Implementing Domain-Driven Design* (Implémentation de la conception pilotée par domaine) de Vaughn Vernon. 

![](./images/ddd-patterns.png)

**Entités**. Une entité est un objet présentant une identité unique et permanente. Par exemple, dans une application bancaire, les clients et les comptes sont des entités. 

- Une entité présente un identifiant unique dans le système, pouvant être utilisé pour la rechercher ou la récupérer. Cela ne signifie pas que les identifiants sont toujours exposés directement aux utilisateurs. Il peut s’agit d’un GUID ou de la clé primaire dans une base de données. 
- Une identité peut être associée à plusieurs limites de contexte et persister au-delà de la durée de vie de l’application. Par exemple, les numéros de comptes bancaires ou les numéros d’identification nationaux n’arrivent pas à expiration à la disparition d’une application spécifique.
- Les attributs d’une entité peuvent évoluer au cours du temps. Par exemple, le nom ou l’adresse d’une personne peuvent évoluer. 
- Une entité peut comporter des références à d’autres entités.
 
**Objets de valeur**. Un objet de valeur n’a aucune identité. Il est défini uniquement par les valeurs de ses attributs. Les objets de valeur sont également immuables. Pour modifier un objet de valeur, vous devez toujours créer une nouvelle instance de remplacement. Les objets de valeur peuvent comporter des méthodes encapsulant la logique du domaine. Toutefois, ces méthodes ne doivent avoir aucun effet sur l’état de l’objet. Comme exemples typiques des objets de valeur, citons les couleurs, les dates et les heures, ainsi que les valeurs de devise. 

**Agrégats**. Un agrégat définit la limite de cohérence autour d’une ou plusieurs entités. Une seule des entités d’un agrégat est la racine. La recherche s’effectue à l’aide de l’identifiant de l’entité racine. Toutes les autres entités de l’agrégat sont des enfants de la racine ; elles sont référencées en suivant les pointeurs de la racine. 

La finalité d’un agrégat est de modéliser les éléments transactionnels invariants. Les composantes du monde réel présentent des réseaux complexes de relations. Les clients créent des commandes, les commandes contiennent des produits, les produits ont des fournisseurs, etc. Si l’application modifie plusieurs objets connexes, comment garantit-elle la cohérence ? Comment effectuer le suivi des éléments invariants ? Comment les appliquer ?  

Les applications traditionnelles ont bien souvent valorisé les transactions de base de données pour garantir les principes de cohérence. Dans une application distribuée, toutefois, ce n’est pas toujours possible. Une seule transaction commerciale peut être associée à plusieurs magasins de données, peut être à long terme ou encore impliquer le recours à des services tiers. Finalement, c’est à l’application, non à la couche de données, d’appliquer les éléments invariants requis pour le domaine. C’est ce que les agrégats ont vocation à modéliser.

> [!NOTE]
> Un agrégat peut être constitué d’une entité unique, sans entité enfant. Il s’agit d’un agrégat parce qu’une limite transactionnelle est identifiée.

**Services de domaine et d’application**. Dans la terminologie de la conception pilotée par domaine, un service est un objet implémentant une certaine logique sans avoir aucun effet sur l’état. Evans fait la différence entre les *services de domaine*, qui encapsulent la logique du domaine, et les *services d’application*, qui offrent une fonctionnalité technique, comme l’authentification utilisateur ou l’envoi d’un message SMS. Les services de domaine sont souvent utilisés pour modéliser un comportement valable sur plusieurs entités. 

> [!NOTE]
> Le terme *service* présente plusieurs significations en développement logiciel. Ici, la définition n’est pas directement liée aux microservices.

**Événements de domaine**. Les événements de domaine peuvent servir à communiquer des signalements à d’autres composantes du système. Comme le nom le suggère, les événements de domaine sont liés au déroulé des opérations au sein du domaine. Par exemple, « un enregistrement a été inséré dans un tableau » n’est pas un événement de domaine. « Une livraison a été annulée » est un événement de domaine. Les événements de domaine sont particulièrement pertinents au sein d’une architecture de microservices. Les microservices étant distribués sans partager de magasins de données, ils tirent parti des événements de domaine pour se coordonner entre eux. Le chapitre [Communication interservice](./interservice-communication.md) évoque plus en détail la messagerie asynchrone.
 
Il existe plusieurs modèles de conception pilotée par domaine non répertoriés ici, comme les fabriques, les référentiels et les modules. Si ces modèles peuvent s’avérer utiles lors de l’implémentation d’un microservice, ils sont moins intéressants au moment de la conception des limites entre les microservices.

## <a name="drone-delivery-applying-the-patterns"></a>Drone Delivery : Application des modèles

Nous allons commencer par les scénarios devant être pris en charge par la limite de contexte Expédition.

- Un client peut demander qu’un drone collecte des marchandises au sein d’une entreprise enregistrée auprès du service de livraison par drone.
- L’expéditeur génère une balise (un code-barres ou une carte RFID) qu’il place sur le colis. 
- Un drone récupère le colis à l’emplacement source et le dépose à sa destination finale.
- Lorsqu’un client planifie une livraison, le système communique un horaire prévu en fonction des données d’itinéraire, des conditions météorologiques et des données d’historique. 
- Lorsque le drone est en vol, un utilisateur peut effectuer le suivi de son parcours et s’informer de l’horaire prévu actualisé. 
- Jusqu’à ce que le drone collecte le colis, le client peut annuler une livraison.
- Le client est informé de la remise du colis.
- L’expéditeur peut demander la confirmation de la livraison au client, sous la forme d’une signature par écrit ou par empreinte digitale.
- Les utilisateurs peuvent rechercher l’historique d’une livraison effectuée.

Pour ces scénarios, l’équipe de développement a identifié les **entités** suivantes.

- Livraison
- Package
- Drone
- Compte
- Confirmation
- Notification
- Tag

Les quatre premiers éléments,associés à la livraison, au colis, au drone et au compte sont tous des **agrégats** représentant des limites de cohérence transactionnelles. Les confirmations et les notifications sont des entités enfants des livraisons, tandis que les balises sont des entités enfants des colis. 

Les **objets de valeur** de cette conception sont l’emplacement, l’horaire prévu, le poids du colis et la taille du colis. 

À titre d’illustration, voici un diagramme UML de l’agrégat de livraison. Notez qu’il comporte des références à d’autres agrégats, dont le compte, le colis et le drone.

![](./images/delivery-entity.png)

Il existe deux événements de domaine :

- Lorsqu’un drone est en vol, l’entité Drone transmet l’événement DroneStatus qui décrit l’emplacement du drone et son état (en vol, à terre).

- L’entité de livraison transmet les événements DeliveryTracking à chaque changement d’étape de la livraison. Il peut par exemple s’agir d’une création de livraison, d’un report, d’une échéance proche ou d’une fin de mission (DeliveryCreated, DeliveryRescheduled, DeliveryHeadedToDropoff et DeliveryCompleted, respectivement). 

Notez que ces événements décrivent les éléments importants du modèle de domaine. Ils ont trait à une situation au sein du domaine, et ne sont liés à aucune construction particulière de langage de programmation.

L’équipe en charge du développement a identifié un domaine supplémentaire de fonctionnalités qui ne peut être pris en compte dans aucune des entités décrites jusqu’à présent. Une partie du système doit coordonner l’ensemble des étapes associées à la planification et à la mise à jour d’une livraison. En conséquence, l’équipe de développement a ajouté deux **services de domaine** à la livraison : un *Planificateur* en charge de la coordination des étapes et un *Superviseur* qui surveille l’état de chaque étape, afin de détecter les défaillances ou les échéances. Il s’agit d’une variante du [modèle de superviseur de l’agent planificateur](../patterns/scheduler-agent-supervisor.md).

![](./images/drone-ddd.png)

> [!div class="nextstepaction"]
> [Identification des limites de microservice](./microservice-boundaries.md)
