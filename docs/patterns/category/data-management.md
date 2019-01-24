---
title: Modèles de gestion des données
titleSuffix: Cloud Design Patterns
description: La gestion des données constitue l’élément clé des applications cloud et a une incidence sur la plupart des attributs de qualité. Les données sont généralement hébergées dans des emplacements distincts et sur plusieurs serveurs pour des raisons de performances, d’extensibilité ou de disponibilité, ce qui peut soulever une multitude de problèmes. Par exemple, il est impératif de préserver la cohérence des données et de synchroniser généralement les données entre les différents emplacements.
keywords: modèle de conception
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 25571a431836656856ed3f299455dfdb94ae3477
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54486979"
---
# <a name="data-management-patterns"></a>Modèles de gestion des données

[!INCLUDE [header](../../_includes/header.md)]

La gestion des données constitue l’élément clé des applications cloud et a une incidence sur la plupart des attributs de qualité. Les données sont généralement hébergées dans des emplacements distincts et sur plusieurs serveurs pour des raisons de performances, d’extensibilité ou de disponibilité, ce qui peut soulever une multitude de problèmes. Par exemple, il est impératif de préserver la cohérence des données et de synchroniser généralement les données entre les différents emplacements.

|                        Modèle                         |                                                                  Résumé                                                                  |
|--------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------|
|            [Cache-Aside](../cache-aside.md)            |                                            Chargez les données à la demande dans un cache à partir d’une banque de données                                             |
|                   [CQRS](../cqrs.md)                   |                    Séparez les opérations qui lisent les données des opérations qui mettent à jour les données en utilisant des interfaces distinctes.                     |
|         [Approvisionnement en événements](../event-sourcing.md)         |               Utilisez un magasin d’ajout uniquement pour enregistrer la série complète d’événements qui décrivent les actions exécutées sur les données dans un domaine.               |
|            [Table d’index](../index-table.md)            |                         Créez des index sur les champs des magasins de données qui sont souvent référencés par les requêtes.                          |
|      [Vue matérialisée](../materialized-view.md)      | Générez des vues préremplies des données d’un ou de plusieurs magasins de données lorsque les données ne sont pas formatées de manière adéquate pour les opérations de requête requises. |
|               [Partitionnement](../sharding.md)               |                                    Divisez un magasin de données en un ensemble de partitions horizontales ou de partitions de base de données.                                     |
| [Hébergement de contenu statique](../static-content-hosting.md) |                   Déployez un contenu statique dans un service de stockage cloud qui peut distribuer ce contenu directement au client.                    |
|              [Clé de sûreté](../valet-key.md)              |                 Utilisez un jeton ou une clé qui fournissent aux clients un accès direct limité à une ressource ou un service spécifique.                 |
