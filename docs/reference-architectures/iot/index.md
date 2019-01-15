---
title: Architecture de référence Azure IoT
description: Architecture recommandée pour les applications IoT sur Azure utilisant des composants PaaS (Platform-as-a-Service)
titleSuffix: Azure Reference Architectures
author: MikeWasson
ms.date: 01/09/2019
ms.openlocfilehash: c0aa1771abc99b1f17f1e553c9626e50a386f34c
ms.sourcegitcommit: d5ea427c25f9f7799cc859b99f328739ca2d8c1c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/15/2019
ms.locfileid: "54307686"
---
# <a name="azure-iot-reference-architecture"></a>Architecture de référence Azure IoT

Cette architecture de référence montre une architecture recommandée pour les applications IoT sur Azure utilisant des composants PaaS (Platform-as-a-Service).

![Diagramme de l’architecture](./_images/iot.png)

Les applications IoT peuvent être décrites comme des **éléments** (appareils) envoyant des données qui génèrent des **insights**. Ces insights génèrent des **actions** permettant d’améliorer une activité ou un processus. Un moteur (l’élément) envoyant des données de température en est un exemple. Ces données sont utilisées pour déterminer si le moteur fonctionne comme prévu (l’insight). L’insight est utilisé pour hiérarchiser de manière proactive la planification de la maintenance du moteur (l’action).

Cette architecture de référence utilise des composants Azure PaaS (Platform-as-a-Service). Les autres options permettant de générer des solutions IoT sur Azure sont les suivantes :

- [Azure IoT Central](/azure/iot-central/). IoT Central est une solution SaaS (Software-as-a-Service) entièrement managée. Elle réduit les choix techniques et vous permet de vous concentrer exclusivement sur votre solution. Cette simplicité présente l’inconvénient que la solution est moins personnalisable qu’une solution basée sur PaaS.
- Utilisation de composants OSS comme la pile SMACK (Spark, Mesos, Akka, Cassandra, Kafka) déployés sur des machines virtuelles Azure. Cette approche offre un très grand contrôle mais est plus complexe.

De façon générale, il existe deux façons de traiter les données de télémétrie : le chemin à chaud et le chemin à froid. La différence réside dans les exigences de latence et d’accès aux données.

- Le **chemin à chaud** analyse les données en quasi temps réel, à mesure qu’elles arrivent. Dans le chemin à chaud, les données de télémétrie doivent être traitées avec une latence très faible. Le chemin à chaud est généralement implémenté à l’aide d’un moteur de traitement de flux. La sortie peut déclencher une alerte, ou être écrite dans un format structuré qui peut être interrogé en utilisant des outils d’analyse.
- Le **chemin à froid** effectue un traitement par lots à des intervalles plus longs (toutes les heures ou chaque jour). Le chemin à froid fonctionne généralement sur des volumes importants de données, mais les résultats n’ont pas besoin d’être aussi rapides que ceux du chemin à chaud. Dans le chemin à froid, les données de télémétrie brutes sont capturées puis envoyées dans un processus de traitement par lots.

## <a name="architecture"></a>Architecture

Cette architecture est constituée des composants suivants. Certaines applications peuvent ne pas avoir besoin de tous les composants listés ici.

**Appareils IoT**. Les appareils peuvent s’inscrire de manière sécurisée au cloud et peuvent se connecter au cloud pour envoyer et recevoir des données. Certains appareils peuvent être des **appareils de périphérie** qui effectuent un traitement de données sur l’appareil lui-même ou dans une passerelle locale. Nous recommandons [Azure IoT Edge](/azure/iot-edge/) pour le traitement en périphérie.

**Passerelle cloud**. Une passerelle cloud fournit un hub cloud pour que les appareils se connectent de manière sécurisée au cloud et envoient des données. Elle s’occupe aussi de la gestion des appareils et des fonctionnalités, dont la commande et le contrôle des appareils. Pour la passerelle cloud, nous recommandons [IoT Hub](/azure/iot-hub/). IoT Hub est un service cloud hébergé qui reçoit les événements des appareils en faisant office de répartiteur de messages entre les appareils et les services back-end. IoT Hub fournit une connectivité sécurisée, l’ingestion des événements, la communication bidirectionnelle et la gestion des appareils.

**Provisionnement des appareils**. Pour l’inscription et la connexion de grands ensembles d’appareils, nous recommandons d’utiliser le [service IoT Hub Device Provisioning](/azure/iot-dps/) (DPS). DPS vous permet d’attribuer des appareils à des points de terminaison Azure IoT Hub spécifiques à grande échelle et de les inscrire auprès de ces derniers.

**Traitement de flux**. Le traitement de flux analyse de grands flux d’enregistrements de données et évalue les règles pour ces flux. Pour le traitement de flux, nous recommandons [Azure Stream Analytics](/azure/stream-analytics/). Stream Analytics peut exécuter une analyse complexe à grande échelle à l’aide de fonctions de fenêtrage temporel, d’agrégations de flux de données et de jonctions de sources de données externes. Apache Spark sur [Azure Databricks](/azure/azure-databricks/) constitue une autre option.

Le machine learning permet l’exécution d’algorithmes prédictifs sur des données de télémétrie historiques, afin de prendre en charge des scénarios tels que la maintenance prédictive. Pour le machine learning, nous recommandons le [service Azure Machine Learning](/azure/machine-learning/service/).

Le **stockage du chemin à chaud** contient les données qui doivent être disponibles immédiatement sur l’appareil pour la création de rapports et la visualisation. Pour le stockage du chemin à chaud, nous recommandons [Cosmos DB](/azure/cosmos-db/introduction). Cosmos DB est une base de données multimodèle distribuée à l’échelle mondiale.

Le **stockage de chemin à froid** contient les données qui sont conservées à plus long terme et qui sont utilisées pour le traitement par lots. Pour le stockage de chemin à froid, nous recommandons le [Stockage Blob Azure](/azure/storage/blobs/storage-blobs-introduction). Les données peuvent être archivées dans le Stockage Blob à moindre coût et pour une durée indéterminée, et sont facilement accessibles pour le traitement par lots.

La **transformation des données** manipule ou agrège le flux de données de télémétrie. Parmi les exemples figurent la transformation de protocole, comme la conversion de données binaires au format JSON, ou la combinaison de points de données. Si les données doivent être transformées avant d’atteindre IoT Hub, nous recommandons d’utiliser une [passerelle de protocole](/azure/iot-hub/iot-hub-protocol-gateway) (non illustrée). Sinon, les données peuvent être transformées après avoir atteint IoT Hub. Dans ce cas, nous recommandons d’utiliser [Azure Functions](/azure/azure-functions/). Functions dispose d’une intégration native à IoT Hub, à Cosmos DB et au Stockage Blob.

L’**intégration des processus métier** effectue des actions basées sur les insights provenant des données de l’appareil. Cela peut inclure le stockage de messages d’information, le déclenchement d’alarmes, l’envoi d’e-mails ou de SMS, ou l’intégration à CRM. Nous recommandons d’utiliser [Azure Logic Apps](/azure/logic-apps/logic-apps-overview) pour l’intégration des processus métier.

La **gestion des utilisateurs** permet de limiter les utilisateurs ou groupes qui peuvent effectuer des actions sur des appareils, comme la mise à niveau de microprogrammes. Elle définit également les fonctionnalités pour les utilisateurs dans les applications. Nous recommandons d’utiliser [Azure Active Directory](/azure/active-directory/) pour authentifier et autoriser les utilisateurs.

## <a name="scalability-considerations"></a>Considérations relatives à l’extensibilité

Une application IoT doit être générée comme des services discrets que vous pouvez mettre à l’échelle de manière indépendante. Considérez les points de scalabilité suivants :

**IoTHub**. Pour IoT Hub, tenez compte des facteurs d’échelle suivants :

- Le [quota quotidien](/azure/iot-hub/iot-hub-devguide-quotas-throttling) maximal de messages dans IoT Hub.
- Le quota d’appareils connectés dans une instance IoT Hub.
- Débit d’ingestion : rapidité à laquelle IoT Hub peut ingérer les messages.
- Débit de traitement : rapidité à laquelle les messages entrants sont traités.

Chaque hub IoT est approvisionné avec un certain nombre d’unités dans un niveau spécifique. Le niveau et le nombre d’unités déterminent le quota quotidien maximal de messages que les appareils peuvent envoyer au hub. Pour plus d’informations, consultez les quotas et la limitation IoT Hub. Vous pouvez effectuer un scale up d’un hub sans interrompre les opérations existantes.

**Stream Analytics**. Les tâches Stream Analytics supportent mieux la mise à l’échelle si elles sont parallèles sur tous les points du pipeline Stream Analytics, depuis l’entrée à interroger jusqu’à la sortie. Une tâche entièrement parallèle permet à Stream Analytics de répartir le travail entre plusieurs nœuds de calcul. Sinon, Stream Analytics doit combiner les données de flux dans un seul et même endroit. Pour plus d’informations, consultez [Profiter de la parallélisation de requête dans Azure Stream Analytics](/azure/stream-analytics/stream-analytics-parallelization).

IoT Hub partitionne automatiquement les messages d’appareil en fonction de l’ID de l’appareil. Tous les messages provenant d’un appareil particulier arrivent toujours sur la même partition, mais une seule et même partition a des messages provenant de plusieurs appareils. Par conséquent, l’unité de parallélisation est l’ID de la partition.

**Fonctions**. Lors de la lecture du point de terminaison Event Hubs, il existe un nombre maximal d’instances de fonction par partition de hub d’événements. La vitesse de traitement maximale est déterminée par la rapidité à laquelle une instance de fonction peut traiter les événements d’une partition unique. La fonction doit traiter les messages par lots.

**Cosmos DB**. Pour effectuer un scale out d’une collection Cosmos DB, créez la collection avec une clé de partition et ajoutez cette clé de partition dans chaque document que vous écrivez. Pour plus d’informations, consultez [Bonnes pratiques lors du choix d’une clé de partition](/azure/cosmos-db/partition-data#best-practices-when-choosing-a-partition-key).

- Si vous stockez et mettez à jour un seul document par appareil, l’ID de l’appareil est une bonne clé de partition. Les écritures sont réparties uniformément entre les différentes clés. La taille de chaque partition est strictement limitée, car il existe un document unique pour chaque valeur de clé.
- Si vous stockez un document distinct pour chaque message de l’appareil, l’utilisation de l’ID de l’appareil comme clé de partition dépassera rapidement la limite de 10 Go par partition. Dans ce cas, l’ID de message est une meilleure clé de partition. En général, vous inclurez quand même l’ID de l’appareil dans le document pour l’indexation et l’interrogation.

## <a name="security-considerations"></a>Considérations relatives à la sécurité

### <a name="trustworthy-and-secure-communication"></a>Communication fiable et sécurisée

Toutes les informations reçues d’un appareil et envoyées vers celui-ci doivent être dignes de confiance. À moins qu’un appareil puisse prendre en charge les fonctionnalités de chiffrement suivantes, il doit être limité aux réseaux locaux et toutes les communications d’interconnexion doivent passer par une passerelle locale :

- Chiffrement des données avec un algorithme de chiffrement à clé symétrique publiquement analysé, largement implémenté et dont la sécurité a été prouvée.
- Signature numérique avec un algorithme de signature à clé symétrique publiquement analysé, largement implémenté et dont la sécurité a été prouvée.
- Prise en charge soit de TLS 1.2 pour TCP ou d’autres chemins de communication basés sur les flux de données, soit de DTLS 1.2 pour les chemins de communication basés sur un datagramme. La prise en charge de la gestion des certificats X.509 est facultative et peut être remplacée par le mode de clé prépartagée plus efficace en ce qui concerne le calcul et la communication pour TLS. Ce mode peut être implémenté avec une prise en charge des algorithmes AES et SHA-2.
- Clés par appareil et magasin de clés pouvant être mis à jour. Chaque appareil doit avoir un élément de clé unique ou des jetons qui l’identifient dans le système. Les appareils doivent stocker la clé de manière sécurisée sur l’appareil (par exemple, à l’aide d’un magasin de clés sécurisé). L’appareil doit être en mesure de mettre à jour les clés ou les jetons régulièrement ou de façon réactive dans des situations d’urgence comme une intrusion dans le système.
- Les microprogrammes et les logiciels d’application sur l’appareil doivent autoriser les mises à jour pour permettre la réparation des failles de sécurité détectées.

Toutefois, de nombreux appareils comportent trop de contraintes pour prendre en charge ces exigences. Dans ce cas, une passerelle locale doit être utilisée. Les appareils se connectent de manière sécurisée à la passerelle locale via un réseau local, et la passerelle permet une communication sécurisée vers le cloud.

### <a name="physical-tamper-proofing"></a>Vérification des falsifications physiques

Il est fortement recommandé que la conception de l’appareil intègre des fonctionnalités qui assurent une protection contre les tentatives de manipulation physique, afin de garantir l’intégrité et la fiabilité de la sécurité du système global.

Par exemple : 

- Choisissez des microcontrôleurs/microprocesseurs ou du matériel auxiliaire qui fournissent un stockage sécurisé et l’utilisation d’un élément de clé de chiffrement, comme l’intégration d’un module de plateforme sécurisée (TPM).
- Chargeur de démarrage sécurisé et chargement de logiciels sécurisé, ancrés dans le module de plateforme sécurisée (TPM).
- Utilisez des capteurs pour détecter les tentatives d’intrusion et les tentatives de manipulation de l’environnement de l’appareil, avec la génération d’alertes et, potentiellement, l’« auto-destruction numérique » de l’appareil.

Pour en savoir plus sur les considérations relatives à la sécurité, consultez [Architecture de sécurité de l’Internet des objets (IoT)](/azure/iot-fundamentals/iot-security-architecture).

### <a name="monitoring-and-logging"></a>Surveillance et journalisation

La journalisation et la supervision des systèmes sont utilisées pour déterminer si la solution fonctionne et pour aider à résoudre les problèmes. La supervision et la journalisation des systèmes permettent de répondre aux questions opérationnelles suivantes :

- Les appareils ou les systèmes sont-ils dans une condition d’erreur ?
- Les appareils ou les systèmes sont-ils correctement configurés ?
- Les appareils ou les systèmes génèrent-ils des données précises ?
- Les systèmes répondent-ils aux attentes des entreprises et des utilisateurs finaux ?

Les outils de journalisation et de supervision comportent généralement les quatre composants suivants :

- Outils de visualisation des performances et de la chronologie du système pour superviser le système et pour le dépannage de base.
- Ingestion de données mises en mémoire tampon, pour mettre en mémoire tampon des données de journal.
- Magasin de persistance pour stocker des données de journal.
- Fonctionnalités de recherche et de requête pour afficher des données de journal afin de les utiliser dans un dépannage détaillé.

Les systèmes de supervision fournissent des insights sur l’intégrité,la sécurité, la stabilité et les performances d’une solution IoT. Ces systèmes peuvent également fournir une vue plus détaillée, en enregistrant les changements de configuration des composants et en fournissant les données de journalisation extraites qui peuvent permettre de détecter des failles de sécurité potentielles, améliorer le processus de gestion des incidents et aider le propriétaire du système à résoudre les problèmes. Les solutions de supervision complètes incluent la possibilité de demander des informations concernant des sous-systèmes spécifiques ou l’agrégation sur plusieurs sous-systèmes.

Le développement de systèmes de supervision doit commencer par définir des critères pour l’intégrité des opérations, la conformité à la réglementation et l’audit. Les métriques collectées peuvent inclure ce qui suit :

- Appareils physiques, appareils de périphérie et composants d’infrastructure signalant les changements de configuration.
- Applications signalant les changements de configuration, les journaux d’audit de sécurité, les taux de demandes, les temps de réponse, les taux d’erreur et les statistiques de nettoyage de la mémoire pour les langages managés.
- Bases de données, magasins de persistance et caches signalant les performances des requêtes et en écriture, les changements de schéma, le journal d’audit de sécurité, les verrous ou blocages, les performances d’index, le processeur, la mémoire et l’utilisation de disque.
- Services managés (IaaS, PaaS, SaaS et FaaS) générant des rapports sur les métriques d’intégrité et les changements de configuration qui affectent les performances et l’intégrité des systèmes dépendants.

La visualisation des métriques de supervision avertit les opérateurs des instabilités du système et facilitent la réponse aux incidents.

### <a name="tracing-telemetry"></a>Suivi des données de télémétrie

Le suivi des données de télémétrie permet à un opérateur de suivre l’évolution d’un élément de télémétrie à partir de sa création par le biais du système. Le suivi est important pour le débogage et la résolution des problèmes. Pour les solutions IoT qui utilisent Azure IoT Hub et les [SDK IoT Hub Device](/azure/iot-hub/iot-hub-devguide-sdks), les datagrammes de suivi peuvent provenir des messages cloud-à-appareil et être inclus dans le flux de données de télémétrie.

### <a name="logging"></a>Journalisation

Les systèmes de journalisation sont déterminants pour comprendre les actions ou les activités qu’une solution a effectuées et les défaillances qui se sont produites. De plus, ils peuvent aider à corriger ces défaillances. Les journaux peuvent être analysés pour comprendre et résoudre des conditions d’erreur, améliorer les caractéristiques de performances et garantir la conformité aux règles et réglementations applicables.

Bien que la journalisation en texte brut ait moins d’impact sur les coûts de développement initiaux, elle est plus difficile à analyser/lire par un ordinateur. Nous vous recommandons d’utiliser une journalisation structurée, car les informations collectées sont à la fois analysables par un ordinateur et lisibles par l’homme. La journalisation structurée ajoute un contexte situationnel et des métadonnées aux informations du journal. Dans la journalisation structurée, les propriétés sont de première classe, sous la forme de paires clé/valeur ou avec un schéma fixe pour améliorer les fonctionnalités de recherche et de requête.

## <a name="next-steps"></a>Étapes suivantes

- Pour obtenir une présentation plus détaillée de l’architecture recommandée et des options d’implémentation, consultez [Architecture de référence Microsoft Azure IoT](http://aka.ms/iotrefarchitecture) (PDF).

- Pour obtenir une documentation détaillée sur les différents services Azure IoT, consultez [Notions de base d’Azure IoT](/azure/iot-fundamentals/).
