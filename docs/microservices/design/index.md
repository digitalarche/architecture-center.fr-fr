---
title: Implémentation de référence des microservices pour Azure Kubernetes Service
description: Cette implémentation de référence présente les bonnes pratiques pour une architecture de microservices
author: MikeWasson
ms.date: 02/26/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: 17e275e5b5f45233f7467192402cb28fce35c57b
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59068903"
---
# <a name="designing-a-microservices-architecture"></a>Conception d’une architecture de microservices

Les microservices sont devenus un style architectural populaire pour générer des applications cloud résilientes, hautement évolutives, capables d’évoluer rapidement et pouvant être déployées indépendamment. Toutefois, au-delà du concept à la mode, les microservices requièrent une approche différente pour concevoir et générer des applications.

Dans cet ensemble d’articles, nous explorons comment générer et exécuter une architecture de microservices sur Azure. Les sujets abordés sont les suivants :

- [Communication entre les services](./interservice-communication.md)
- [Conception d’API](./api-design.md)
- [Passerelles d’API](./gateway.md)
- [Considérations sur les données](./data-considerations.md)
- [Modèles de conception](./patterns.md)

## <a name="prerequisites"></a>Prérequis

Avant de lire ces articles, vous pouvez commencer par ce qui suit :

- [Introduction aux architectures de microservices](../introduction.md). Comprendre les avantages et les problématiques des microservices, et quand utiliser ce style d’architecture.
- [Utilisation de l’analyse de domaine pour modéliser les microservices](../model/domain-analysis.md). Découvrir une approche orientée domaine pour la modélisation des microservices.

## <a name="reference-implementation"></a>Implémentation de référence

Pour illustrer les bonnes pratiques concernant une architecture de microservices, nous avons créé une implémentation de référence que nous appelons « application de livraison par drone (Drone Delivery) ». Cette implémentation s’exécute sur Kubernetes à l’aide d’Azure Kubernetes Service (AKS). Vous trouverez l’implémentation de référence sur [GitHub][drone-ri].

![Architecture de l’application de livraison par drone (Drone Delivery)](../images/drone-delivery.png)

## <a name="scenario"></a>Scénario

Fabrikam, Inc. veut lancer un service de livraison par drone. La société gère un parc de drones. Les entreprises souscrivent au service, et les utilisateurs peuvent demander à ce qu’un drone vienne récupérer les marchandises à livrer. Lorsqu’un client planifie un enlèvement de colis, un système principal assigne un drone et donne à l’utilisateur un délai de livraison estimé. Une fois la livraison en cours, le client peut suivre la position du drone, avec un ATE mis à jour en permanence.

Ce scénario implique un domaine assez complexe. Certaines entreprises veulent programmer des drones, suivre des colis, gérer les comptes d’utilisateur, ainsi que stocker et analyser les données d’historique. En outre, Fabrikam souhaite commercialiser ce service et effectuer une itération rapidement, en ajoutant des nouvelles fonctionnalités. L’application doit fonctionner à l’échelle du cloud, avec un objectif de niveau de service (SLO) élevé. Fabrikam prévoit également que les diverses parties du système auront des besoins très différents pour le stockage et l’interrogation des données. En s’appuyant sur toutes ces considérations, Fabrikam opte pour une architecture de microservices pour l’application Drone Delivery.

> [!NOTE]
> Pour éclairer votre choix entre une architecture de microservices et d’autres styles d’architecture, consultez le [Guide de l’architecture des applications Azure](../../guide/index.md).

Notre implémentation de référence utilise Kubernetes avec [Azure Kubernetes Service](/azure/aks/) (AKS). Toutefois, la plupart des principaux défis et décisions architecturales s’appliquent à n’importe quel orchestrateur de conteneur, y compris [Azure Service Fabric](/azure/service-fabric/).

<!-- links -->

[drone-ri]: https://github.com/mspnp/microservices-reference-implementation/tree/v0.1.0-orig

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Choisir une option de calcul](./compute-options.md)