---
title: "Choisir une technologie d’ingestion de messages en temps réel"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 2e6578b779950b5ef11bda7b8ba1fb2e45e09f4e
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/23/2018
---
# <a name="choosing-a-real-time-message-ingestion-technology-in-azure"></a>Choisir une technologie d’ingestion de messages en temps réel dans Azure

Le traitement en temps réel porte sur les flux de données qui sont capturés en temps réel et traités avec une latence minimale. De nombreuses solutions de traitement en temps réel ont besoin d’un magasin d’ingestion des messages qui agit comme une mémoire tampon pour les messages et qui prend en charge un traitement de montée en puissance, une remise fiable et d’autres sémantiques de files d’attente de message. 

## <a name="what-are-your-options-for-real-time-message-ingestion"></a>Quelles sont vos options pour l’ingestion de messages en temps réel ?

- [Azure Event Hubs](/azure/event-hubs/)
- [Azure IoT Hub](/azure/iot-hub/)
- [Kafka sur HDInsight](/azure/hdinsight/kafka/apache-kafka-get-started)

## <a name="azure-event-hubs"></a>Hubs d'événements Azure

[Azure Event Hubs](/azure/event-hubs/) est une plateforme de diffusion de données hautement évolutive et un service d’ingestion d’événements, capable de recevoir et de traiter des millions d’événements par seconde. Les concentrateurs d’événements peuvent traiter et stocker des événements, des données ou la télémétrie produits par des logiciels et appareils distribués. Les données envoyées à un concentrateur d’événements peuvent être transformées et stockées à l’aide d’adaptateurs de traitement par lot/stockage ou d’un fournisseur d’analyse en temps réel. Event Hubs fournit des fonctionnalités de publication-abonnement avec une faible latence à très grande échelle, ce qui le rend approprié pour les scénarios Big Data.

## <a name="azure-iot-hub"></a>Azure IoT Hub

[Azure IoT Hub](/azure/iot-hub/) est un service managé qui permet des communications bidirectionnelles fiables et sécurisées entre des millions d’appareils IoT et un serveur principal basé sur cloud.

Les fonctionnalités d’IoT Hub incluent :

* Plusieurs options de communication appareil-à-cloud et cloud-à-appareil. Ces options comprennent la messagerie unidirectionnelle, le transfert de fichiers et les méthodes de demande-réponse.
* Routage de messages vers d’autres services Azure.
* Stockage utilisable dans une requête pour les métadonnées d’appareil et les informations d’état synchronisées.
* Sécurité des communications et le contrôle d’accès grâce aux clés de sécurité par appareil ou aux certificats X.509.
* Monitoring de la connectivité des appareils et des événements de gestion de l’identité des appareils.

En termes d’ingestion de messages, IoT Hub est similaire à Event Hubs. Toutefois, il a été spécialement conçu pour gérer la connectivité des appareils IoT, pas seulement l’ingestion de messages. Pour plus d’informations, consultez [Comparaison entre Azure IoT Hub et Azure Event Hubs](/azure/iot-hub/iot-hub-compare-event-hubs). 

## <a name="kafka-on-hdinsight"></a>Kafka sur HDInsight

[Apache Kafka](https://kafka.apache.org/) est une plateforme de diffusion en continu distribuée open source qui permet de générer des pipelines de données et des applications de diffusion en continu en temps réel. Kafka fournit également des fonctionnalités de courtier de messages semblables à une file d’attente, où vous pouvez publier et vous abonner aux flux de données nommés. Il est extrêmement rapide, tolérant aux pannes et horizontalement évolutif. [Kafka sur HDInsight](/azure/hdinsight/kafka/apache-kafka-get-started) offre un Kafka en tant que service managé, hautement évolutif et hautement disponible dans Azure. 

Exemples de scénarios courants d’utilisation de Kafka :

* **Messagerie**. Dans la mesure où Kafka prend en charge le modèle de messagerie publication-abonnement, il est souvent utilisé comme courtier de messages.
* **Suivi des activités**. Étant donné que Kafka fournit la journalisation dans l’ordre des enregistrements, il peut être utilisé pour effectuer le suivi et recréer des activités, comme les actions d’un utilisateur sur un site web.
* **Agrégation**. Avec le traitement de flux de données, vous pouvez agréger des informations à partir de différents flux afin de combiner et de centraliser les informations dans des données opérationnelles.
* **Transformation**. Avec le traitement de flux de données, vous pouvez combiner et enrichir les données à partir de plusieurs rubriques d’entrée dans une ou plusieurs rubriques de sortie.

## <a name="key-selection-criteria"></a>Critères de sélection principaux

Pour restreindre les choix, commencez par répondre à ces questions :

- Avez-vous besoin d’une communication bidirectionnelle entre vos appareils IoT et Azure ? Si c’est le cas, choisissez IoT Hub.

- Avez-vous besoin gérer l’accès pour les appareils individuels et être en mesure de révoquer l’accès à un appareil spécifique ? Si la réponse est Oui, choisissez IoT Hub.

## <a name="capability-matrix"></a>Matrice des fonctionnalités

Les tableaux suivants résument les principales différences entre les fonctionnalités. 

| | IoT Hub | Event Hubs | Kafka sur HDInsight |
| --- | --- | --- | --- |
| Communications cloud-à-appareil | OUI | Non  | Non  |
| Chargement de fichiers initié par l’appareil | OUI | Non  | Non  |
| Informations d’état de l’appareil | [Représentations d’appareil physique](/azure/iot-hub/iot-hub-devguide-device-twins) | Non  | Non  |
| Prise en charge du protocole | MQTT, AMQP, HTTPS <sup>1</sup> | AMQP, HTTPS | [Kafka Protocol](https://cwiki.apache.org/confluence/display/KAFKA/A+Guide+To+The+Kafka+Protocol) |
| Sécurité | Identité par appareil ; contrôle d’accès révocable. | Stratégies d’accès partagé ; révocation limitée par le biais des stratégies de l’éditeur. | Authentification via SASL ; autorisation enfichable ; intégration avec des services d’authentification externes prise en charge. |

[1] Vous pouvez aussi utiliser la [passerelle de protocole Azure IoT](/azure/iot-hub/iot-hub-protocol-gateway) comme passerelle personnalisée pour permettre l’adaptation de protocole pour IoT Hub.

Pour plus d’informations, consultez [Comparaison entre Azure IoT Hub et Azure Event Hubs](/azure/iot-hub/iot-hub-compare-event-hubs).
