---
title: Modèles de performances et d’extensibilité
titleSuffix: Cloud Design Patterns
description: Les performances sont une indication de la réactivité d’un système pour exécuter des actions dans un intervalle de temps donné, tandis que l’extensibilité est la capacité d’un système à gérer les augmentations de charge sans impact sur les performances ou à augmenter les ressources disponibles de manière adéquate. Les applications cloud rencontrent généralement des charges de travail variables et des pics soudains dans l’activité. Prédire ces derniers, en particulier dans un scénario multi-locataire, est presque impossible. Au lieu de cela, les applications doivent être en mesure d’augmenter la taille des instances dans les limites pour répondre aux pics de demande et de diminuer la taille des instances lorsque la demande diminue. L’extensibilité ne concerne pas seulement les instances de calcul, mais également d’autres éléments, tels que le stockage des données, l’infrastructure de messagerie et bien plus encore.
keywords: modèle de conception
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: a57194f70218a8294507bd389ea9526660e3041b
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54480723"
---
# <a name="performance-and-scalability-patterns"></a>Modèles de performances et d’extensibilité

[!INCLUDE [header](../../_includes/header.md)]

Les performances sont une indication de la réactivité d’un système pour exécuter des actions dans un intervalle de temps donné, tandis que l’extensibilité est la capacité d’un système à gérer les augmentations de charge sans impact sur les performances ou à augmenter les ressources disponibles de manière adéquate. Les applications cloud rencontrent généralement des charges de travail variables et des pics soudains dans l’activité. Prédire ces derniers, en particulier dans un scénario multi-locataire, est presque impossible. Au lieu de cela, les applications doivent être en mesure d’augmenter la taille des instances dans les limites pour répondre aux pics de demande et de diminuer la taille des instances lorsque la demande diminue. L’extensibilité ne concerne pas seulement les instances de calcul, mais également d’autres éléments, tels que le stockage des données, l’infrastructure de messagerie et bien plus encore.

|                           Modèle                            |                                                                        Résumé                                                                         |
|--------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
|               [Cache-Aside](../cache-aside.md)               |                                                   Chargez les données à la demande dans un cache à partir d’une banque de données                                                   |
|                      [CQRS](../cqrs.md)                      |                           Séparez les opérations qui lisent les données des opérations qui mettent à jour les données en utilisant des interfaces distinctes.                           |
|            [Approvisionnement en événements](../event-sourcing.md)            |                     Utilisez un magasin d’ajout uniquement pour enregistrer la série complète d’événements qui décrivent les actions exécutées sur les données dans un domaine.                      |
|               [Table d’index](../index-table.md)               |                                Créez des index sur les champs des magasins de données qui sont souvent référencés par les requêtes.                                |
|         [Vue matérialisée](../materialized-view.md)         |       Générez des vues préremplies sur les données d’un ou de plusieurs magasins de données lorsque les données ne sont pas adéquatement formatées pour les opérations de requête requises.        |
|            [File d’attente de priorité](../priority-queue.md)            | Classez par ordre de priorité les requêtes envoyées aux services, de telle sorte que les demandes ayant une priorité plus élevée soient reçues et traitées plus rapidement que celles de moindre priorité. |
| [Nivellement de la charge basé sur une file d’attente](../queue-based-load-leveling.md) |              Utilisez une file d’attente qui agit comme mémoire tampon entre une tâche et un service qu’elle appelle, afin d’atténuer les surcharges intermittentes.               |
|                  [Partitionnement](../sharding.md)                  |                                           Divisez un magasin de données en un ensemble de partitions horizontales ou de partitions de base de données.                                           |
|    [Hébergement de contenu statique](../static-content-hosting.md)    |                          Déployez un contenu statique dans un service de stockage cloud qui peut distribuer ce contenu directement au client.                          |
|                [Limitation](../throttling.md)                |                Contrôlez la consommation des ressources utilisées par une instance d’une application, un locataire ou un service entier.                 |
