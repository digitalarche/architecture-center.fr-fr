---
title: Intégration d’entreprise avec des files d’attente de messages et des événements
titleSuffix: Azure Reference Architectures
description: Architecture recommandée pour implémenter un modèle d’intégration d’entreprise avec Azure Logic Apps, Gestion des API Azure, Azure Service Bus et Azure Event Grid.
author: mattfarm
ms.reviewer: jonfan, estfan, LADocs
ms.topic: reference-architecture
ms.date: 12/03/2018
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: integration-services
ms.openlocfilehash: 4c9d2e201bcfc077990d746a1decd55ede2f220a
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58245510"
---
# <a name="enterprise-integration-on-azure-using-message-queues-and-events"></a>Intégration d’entreprise sur Azure avec des files d’attente de messages et des événements

Cette architecture de référence intègre des systèmes back-end d’entreprise, avec des files d’attente de messages et des événements, pour découpler les services et obtenir ainsi une scalabilité et une fiabilité supérieures. Les systèmes principaux peuvent inclure des systèmes SaaS (Logiciel en tant que service), des services Azure et des services web existants dans votre entreprise.

![Architecture de référence pour l’intégration d’entreprise avec des files d’attente et des événements](./_images/enterprise-integration-queues-events.png)

## <a name="architecture"></a>Architecture

L’architecture illustrée ici s’appuie sur une architecture plus simple, qui est illustrée dans [Intégration d’entreprise de base][basic-enterprise-integration]. Cette architecture utilise [Logic Apps][logic-apps] pour orchestrer les workflows, et [Gestion des API][apim] pour créer des catalogues d’API.

Cette version de l’architecture ajoute deux composants qui permettent de rendre le système plus fiable et plus scalable :

- **[Azure Service Bus][service-bus]**. Service Bus est un courtier de messages sécurisé et fiable.

- **[Azure Event Grid][event-grid]**. Event Grid est un service de routage d’événements. Il utilise un modèle de gestion des événements [publication/abonnement](../../patterns/publisher-subscriber.md) (pub/sub).

La communication asynchrone avec un courtier de messages offre plusieurs avantages par rapport aux appels synchrones, directs à des services back-end :

- Fournit un nivellement de la charge pour gérer les pics des charges de travail avec le [Modèle de nivellement de charge basé sur une file d’attente](../../patterns/queue-based-load-leveling.md).
- Effectue un suivi fiable de la progression des workflows à exécution longue qui impliquent plusieurs étapes ou plusieurs applications.
- Aide à découpler les applications.
- S’intègre à des systèmes existants basés sur les messages.
- Permet la mise en file d’attente des travaux quand un système back-end n’est pas disponible.

Event Grid permet aux différents composants du système de réagir aux événements au fur et à mesure qu’ils se produisent, au lieu de s’appuyer sur l’interrogation ou sur des tâches planifiées. Comme avec une file d’attente de messages, il permet de découpler les applications et les services. Une application ou un service peut publier des événements : les abonnés intéressés en sont alors avertis. De nouveaux abonnés peuvent être ajoutés sans mettre à jour l’expéditeur.

De nombreux services Azure prennent en charge l’envoi d’événements à Event Grid. Par exemple, une application logique peut être à l’écoute d’un événement, par exemple quand de nouveaux fichiers sont ajoutés à un magasin d’objets blob. Ce modèle permet des workflows réactifs, où le chargement d’un fichier ou le placement d’un message dans une file d’attente lance une série de processus. Les processus peuvent être exécutés en parallèle ou dans une séquence spécifique.

## <a name="recommendations"></a>Recommandations

Les recommandations décrites dans [Intégration d’entreprise de base][basic-enterprise-integration] s’appliquent à cette architecture. Les recommandations suivantes s’appliquent également :

### <a name="service-bus"></a>Service Bus

Service Bus a deux modes de remise, *extraction (pull)* et *envoi (push)*. Dans le modèle d’extraction, le récepteur interroge de façon continue pour déterminer s’il y a de nouveaux messages. L’interrogation peut être inefficace, surtout si vous avez de nombreuses files d’attente qui reçoivent chacune quelques messages, ou si l’intervalle de temps entre les messages est important. Dans le modèle d’envoi, Service Bus envoie un événement via Event Grid quand il y a des nouveaux messages. Le destinataire s’abonne à l’événement. Quand l’événement est déclenché, le destinataire extrait de Service Bus le lot de messages suivant.

Quand vous créez une application logique pour consommer des messages Service Bus, nous recommandons d’utiliser le modèle d’envoi avec intégration d’Event Grid. Il est souvent plus économique, car l’application logique n’a pas besoin d’interroger Service Bus. Pour plus d’informations, consultez [Vue d’ensemble de l’intégration d’Azure Service Bus à Event Grid](/azure/service-bus-messaging/service-bus-to-event-grid-integration-concept). Actuellement, le [niveau Premium](https://azure.microsoft.com/pricing/details/service-bus/) de Service Bus est nécessaire pour les notifications Event Grid.

Utilisez [PeekLock](/azure/service-bus-messaging/service-bus-messaging-overview#queues) pour accéder à un groupe de messages. Lorsque vous utilisez PeekLock, l’application logique peut suivre des étapes pour valider chaque message avant de terminer ou d’abandonner le message. Cette approche protège contre la perte accidentelle de messages.

### <a name="event-grid"></a>Event Grid

Quand un déclencheur Event Grid est activé, cela signifie *qu’au moins un* événement s’est produit. Par exemple, quand une application logique reçoit un déclencheur Event Grid pour un message Service Bus, elle doit supposer que plusieurs messages peuvent être disponibles pour être traités.

Event Grid utilise un modèle serverless. La facturation est calculée en fonction du nombre d’opérations (exécutions d’événement). Pour plus d’informations, consultez [Prix d’Event Grid](https://azure.microsoft.com/pricing/details/event-grid/). Actuellement, aucune considération relative aux niveaux ne s’applique à Event Grid.

## <a name="scalability-considerations"></a>Considérations relatives à l’extensibilité

Pour bénéficier d’une meilleure extensibilité, le niveau Premium de Service Bus peut augmenter le nombre d’unités de messagerie. Les configurations de niveau Premium peuvent avoir une, deux ou quatre unités de messagerie. Pour plus d’informations sur le dimensionnement de Service Bus, consultez [Bonnes pratiques relatives aux améliorations de performances à l’aide de la messagerie Service Bus](/azure/service-bus-messaging/service-bus-performance-improvements).

## <a name="availability-considerations"></a>Considérations relatives à la disponibilité

Passez en revue le contrat de niveau de service pour chaque service :

- [Contrat de niveau de service de Gestion des API][apim-sla]
- [Contrat SLA d’Event Grid][event-grid-sla]
- [Contrat de niveau de service de Logic Apps][logic-apps-sla]
- [Contrat SLA de Service Bus][sb-sla]

Pour permettre un basculement en cas de panne grave, envisagez d’implémenter la géo-reprise d'activité après sinistre dans Service Bus Premium. Pour plus d’informations, consultez [Géo-reprise d'activité après sinistre avec Azure Service Bus](/azure/service-bus-messaging/service-bus-geo-dr).

## <a name="security-considerations"></a>Considérations relatives à la sécurité

Pour sécuriser Service Bus, utilisez une signature d’accès partagé. Par exemple, vous pouvez accorder à un utilisateur l’accès à des ressources Service Bus avec des droits spécifiques en utilisant l’[authentification SAS](/azure/service-bus-messaging/service-bus-sas). Pour plus d’informations, consultez [Authentification et autorisation Service Bus](/azure/service-bus-messaging/service-bus-authentication-and-authorization).

Si vous devez exposer une file d’attente Service Bus en tant que point de terminaison HTTP (pour la publication de nouveaux messages, par exemple), utilisez Gestion des API pour la sécuriser en exposant le point de terminaison. Vous pouvez alors sécuriser le point de terminaison avec des certificats ou une authentification OAuth si besoin. Le moyen le plus simple de sécuriser un point de terminaison consiste à utiliser une application logique avec un déclencheur de requête/réponse HTTP comme intermédiaire.

Le service Event Grid sécurise la distribution des événements au moyen d’un code de validation. Si vous utilisez Logic Apps pour consommer l’événement, la validation est automatiquement effectuée. Pour en savoir plus, consultez la page [Sécurité et authentification pour Event Grid](/azure/event-grid/security-authentication).

[apim]: /azure/api-management
[apim-sla]: https://azure.microsoft.com/support/legal/sla/api-management/
[event-grid]: /azure/event-grid/
[event-grid-sla]: https://azure.microsoft.com/support/legal/sla/event-grid
[logic-apps]: /azure/logic-apps/logic-apps-overview
[logic-apps-sla]: https://azure.microsoft.com/support/legal/sla/logic-apps
[sb-sla]: https://azure.microsoft.com/support/legal/sla/service-bus/
[service-bus]: /azure/service-bus-messaging/
[basic-enterprise-integration]: ./basic-enterprise-integration.md
