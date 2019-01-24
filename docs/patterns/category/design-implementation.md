---
title: Modèles de conception et d’implémentation
titleSuffix: Cloud Design Patterns
description: Une bonne conception englobe des facteurs tels que la cohérence et la cohérence dans la conception de composants et de déploiements, la facilité de gestion pour simplifier l’administration et le développement, et la réutilisation pour permettre l’utilisation de composants et de sous-systèmes dans d’autres applications et dans d’autres scénarios. Les décisions prises pendant la phase de conception et d’implémentation ont un impact important sur la qualité et le coût total de possession d’applications et de services cloud.
keywords: modèle de conception
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: ff609679bed4ee4aa88daf95ff1cab4ab2d90d48
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54482545"
---
# <a name="design-and-implementation-patterns"></a>Modèles de conception et d’implémentation

Une bonne conception englobe des facteurs tels que la cohérence et la cohérence dans la conception de composants et de déploiements, la facilité de gestion pour simplifier l’administration et le développement, et la réutilisation pour permettre l’utilisation de composants et de sous-systèmes dans d’autres applications et dans d’autres scénarios. Les décisions prises pendant la phase de conception et d’implémentation ont un impact important sur la qualité et le coût total de possession d’applications et de services cloud.

|                                Modèle                                 |                                                                                                      Résumé                                                                                                       |
|------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                     [Ambassadeur](../ambassador.md)                     |                                                         Créez des services d’assistance qui envoient des requêtes réseau pour le compte d’applications ou d’un service consommateur.                                                          |
|          [Couche de lutte contre la corruption](../anti-corruption-layer.md)          |                                                               Implémentez une couche de façade ou d’adaptateur entre une application moderne et un système hérité.                                                                |
|         [Services principaux destinés aux frontaux](../backends-for-frontends.md)         |                                                          Créez différents services principaux destinés à être utilisés par des applications ou interfaces frontales spécifiques.                                                          |
|                           [CQRS](../cqrs.md)                           |                                                         Séparez les opérations qui lisent les données des opérations qui mettent à jour les données en utilisant des interfaces distinctes.                                                         |
| [Consolidation des ressources de calcul](../compute-resource-consolidation.md) |                                                                     Consolidez plusieurs tâches ou opérations en une seule unité de calcul.                                                                      |
|   [Magasin de configurations externes](../external-configuration-store.md)   |                                                        Déplacez les informations de configuration depuis le package de déploiement d’application vers un emplacement centralisé.                                                         |
|            [Agrégation de passerelle](../gateway-aggregation.md)            |                                                                   Utilisez une passerelle pour agréger plusieurs requêtes individuelles dans une requête unique.                                                                   |
|             [Déchargement de passerelle](../gateway-offloading.md)             |                                                                      Déchargez des fonctionnalités de service partagé ou spécialisé sur un proxy de passerelle.                                                                       |
|                [Routage de passerelle](../gateway-routing.md)                |                                                                            Acheminez les requêtes vers plusieurs services à l’aide d’un seul point de terminaison.                                                                            |
|                [Élection du responsable](../leader-election.md)                | Coordonnez les actions effectuées par une collection d’instances de tâche de collaboration dans une application distribuée en élisant l’instance responsable qui sera chargée de gérer les autres instances. |
|              [Canaux et filtres](../pipes-and-filters.md)              |                                                     Divisez une tâche qui exécute un traitement complexe en une série d’éléments séparés qui peuvent être réutilisés.                                                      |
|                        [Sidecar](../sidecar.md)                        |                                                  Déployez les composants d’une application sur un processus ou conteneur distinct pour fournir l’isolation et l’encapsulation.                                                  |
|         [Hébergement de contenu statique](../static-content-hosting.md)         |                                                        Déployez un contenu statique dans un service de stockage cloud qui peut distribuer ce contenu directement au client.                                                        |
|                      [Étranglement](../strangler.md)                      |                                         Faites migrer un système hérité de façon incrémentielle en remplaçant progressivement des parties spécifiques des fonctionnalités par de nouveaux services et applications.                                          |
