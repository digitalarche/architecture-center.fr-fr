---
title: 'Framework d’adoption du cloud : Réseau à définition logicielle – Natif dans le cloud'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Présentation des services de réseau virtuel natifs dans le cloud
author: rotycenh
ms.openlocfilehash: e0ad6803f2ddc982ea0c42c59fdf2486e1710433
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900870"
---
# <a name="software-defined-networks-hub-and-spoke"></a>Réseaux à définition logicielle : Hub-and-Spoke

Avec le modèle de mise en réseau hub-and-spoke, vous pouvez organiser votre infrastructure de réseau cloud basée sur Azure en plusieurs réseaux virtuels connectés. Ce modèle vous permet de gérer plus efficacement les exigences courantes en matière de communication ou de sécurité et de faire face aux potentielles limitations des abonnements.

Dans le modèle hub-and-spoke, le *hub* est un réseau virtuel qui joue le rôle d’emplacement central pour gérer la connectivité externe et les services d’hébergement qui sont utilisés par plusieurs charges de travail. Le terme *spokes* (rayons) désigne des réseaux virtuels qui hébergent des charges de travail et se connectent au hub central à l’aide du [peering de réseaux virtuels](/virtual-network/virtual-network-peering-overview).

L’intégralité du trafic qui entre ou sort des réseaux spoke pour la charge de travail est acheminée via le réseau hub. Le trafic peut alors être routé, examiné ou géré par les règles ou processus informatiques centralisés.

Ce modèle vise à résoudre les problèmes suivants :

- Réduction des coûts et amélioration de la gestion : En centralisant les services qui peuvent être partagés par plusieurs charges de travail (comme les appliances virtuelles réseau et les serveurs DNS) en un seul endroit, le service informatique est capable de réduire la redondance des ressources et les efforts de gestion sur plusieurs charges de travail.
- Dépassement des limites des abonnements : Pour exécuter les charges de travail volumineuses dans le cloud, il se peut que vous ayez besoin d’utiliser davantage de ressources que le quota autorisé par un abonnement Azure unique (consultez les [limites d’abonnement](/azure/azure-subscription-service-limits)). Vous avez la possibilité de dépasser ces limites en appairant des réseaux virtuels de charge de travail issus de différents abonnements à un hub central.
- Séparation des problèmes : Vous avez la possibilité de déployer des charges de travail individuelles entre les équipes informatiques centrales et celles dédiées aux charges de travail.

Le schéma suivant présente un exemple d’architecture hub-and-spoke comprenant une connectivité hybride centralisée.

![Architecture de réseau hub-and-spoke](../../../reference-architectures/hybrid-networking/images/hub-spoke.png)

L’architecture hub-and-spoke est souvent utilisée parallèlement à l’architecture de mise en réseau hybride, en fournissant une connexion centralisée à votre environnement local partagé entre plusieurs charges de travail. Dans ce scénario, l’intégralité du trafic qui transite entre les charges de travail et le site local passe par le hub où il peut être géré et sécurisé.

## <a name="hub-and-spoke-assumptions"></a>Hypothèses relatives à hub-and-spoke

L’implémentation d’une architecture de mise en réseau virtuel hub-and-spoke suppose les hypothèses suivantes :

- Vos déploiements cloud impliquent des charges de travail hébergées dans des environnements de travail distincts (développement, test et production, par exemple) qui s’appuient sur un ensemble de services communs tels qu’un DNS ou des services d’annuaire.
- Vos charges de travail n’ont pas besoin de communiquer entre elles, mais elles disposent de communications externes communes et d’exigences de services partagées.
- Vos charges de travail nécessitent davantage de ressources que celles disponibles dans un seul abonnement Azure.
- Vous devez accorder des droits d’administration délégués aux équipes dédiées aux charges de travail pour leurs propres ressources, tout en conservant le contrôle central de sécurité sur la connectivité externe.

## <a name="global-hub-and-spoke"></a>Modèle hub-and-spoke mondial

Les architectures hub-and-spoke sont généralement implémentées avec des réseaux virtuels déployés dans la même région Azure afin de réduire la latence entre les réseaux. Toutefois, il se peut que les grandes organisations d’envergure mondiale doivent déployer des charges de travail dans plusieurs régions à des fins de disponibilité, de récupération d’urgence ou de respect des exigences réglementaires. Grâce à l’utilisation du [peering de réseau virtuel mondial](/azure/virtual-network/virtual-network-peering-overview), le modèle hub-and-spoke peut étendre la gestion centralisée et le partage de services entre plusieurs régions pour répondre aux charges de travail réparties dans le monde entier.

## <a name="learn-more"></a>En savoir plus

Pour obtenir des exemples illustrant l’implémentation des réseaux hub-and-spoke dans Azure, consultez les rubriques suivantes sur le site des architectures de référence Azure :

- [Implémenter une topologie de réseau hub-and-spoke dans Azure](../../../reference-architectures/hybrid-networking/hub-spoke.md)
- [Implémenter une topologie de réseau hub-and-spoke avec des services partagés dans Azure](../../../reference-architectures/hybrid-networking/shared-services.md)
