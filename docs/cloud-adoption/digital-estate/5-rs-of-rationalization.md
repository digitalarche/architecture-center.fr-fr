---
title: 'Framework d’adoption du cloud : Les 5 R de la rationalisation'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
description: Passez en revue les options disponibles pour rationaliser un patrimoine numérique.
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.openlocfilehash: ee196487e6f59b1e1b3c63bab9496cbbf805affd
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55897471"
---
# <a name="the-5-rs-of-rationalization"></a>Les 5 R de la rationalisation

La rationalisation du cloud est le processus qui consiste à évaluer des ressources dans le but de déterminer la meilleure approche à adopter pour la migration ou la modernisation de chaque ressource présente dans le cloud. Pour plus d’informations sur le processus de rationalisation, consultez [Qu’est-ce qu’un patrimoine numérique ?](overview.md)

Les « 5 R de la rationalisation » listés ici décrivent les options de rationalisation les plus courantes.

## <a name="rehost"></a>Réhébergement

Également appelé « lift and shift », le réhébergement déplace une ressource vers le fournisseur de cloud choisi, en apportant des modifications minimales à l’architecture globale.

Les moteurs les plus courants sont les suivants :

* Réduire les dépenses d’investissement
* Libérer de l’espace au sein du centre de données
* Obtenir un retour sur investissement rapide

Facteurs d’analyse quantitative :

* Taille de la machine virtuelle (processeur, mémoire, stockage)
* Dépendances (trafic réseau)
* Compatibilité des ressources

Facteurs d’analyse qualitative :

* Tolérance au changement
* Priorités de l’entreprise
* Événements critiques pour l’entreprise
* Dépendances de processus

## <a name="refactor"></a>Refactorisation

Les options PaaS permettent de réduire les coûts d’exploitation associés à de nombreuses applications. Il paraît plus prudent de refactoriser légèrement une application pour qu’elle corresponde à un modèle PaaS.

La refactorisation s’applique également au processus de développement d’applications, dans lequel du code est refactorisé pour permettre à une application de répondre à de nouvelles opportunités commerciales.

Les moteurs les plus courants sont les suivants :

* Mises à jour plus rapides
* Portabilité du code
* Une plus grande efficacité du cloud (ressources, vitesse, coût)

Facteurs d’analyse quantitative :

* Taille des ressources d’application (processeur, mémoire, stockage)
* Dépendances (trafic réseau)
* Trafic utilisateur (consultations de page, temps passé sur une page, temps de chargement)
* Plateforme de développement (langages, plateforme de données, services de niveau intermédiaire)

Facteurs d’analyse qualitative :

* Investissements continus
* Options de cloud bursting/Chronologies
* Dépendances des processus métier

## <a name="rearchitect"></a>Réarchitecture

Certaines applications vieillissantes ne sont pas compatibles avec les fournisseurs de cloud, en raison des décisions qui ont été prises concernant l’architecture lors de la conception de l’application. Dans ce cas, l’application doit subir une réarchitecture avant d’être transformée.

Dans d’autres cas, les applications qui sont compatibles avec le cloud mais qui ne sont pas natives du cloud peuvent bénéficier d’une meilleure rentabilité et d’une efficacité opérationnelle si vous les réarchitecturez pour en faire des applications natives du cloud.

Les moteurs les plus courants sont les suivants :

* Mise à l’échelle et agilité de l’application
* Adoption facilitée des nouvelles fonctionnalités cloud
* Piles de technologies mixtes

Facteurs d’analyse quantitative :

* Taille des ressources d’application (processeur, mémoire, stockage)
* Dépendances (trafic réseau)
* Trafic utilisateur (consultations de page, temps passé sur une page, temps de chargement)
* Plateforme de développement (langages, plateforme de données, services de niveau intermédiaire)

Facteurs d’analyse qualitative :

* Investissements continus en augmentation
* Coûts d’exploitation
* Boucles de rétroaction et investissements DevOps potentiels

## <a name="rebuild"></a>Reconstruction

Dans certains scénarios, les efforts nécessaires à la réarchitecture d’une application peuvent être trop importants pour justifier tout investissement supplémentaire. C’est particulièrement vrai pour les applications qui répondaient autrefois aux besoins de l’entreprise, mais qui désormais ne sont plus prises en charge ou ne sont plus alignées sur les processus actuels de l’entreprise. Dans ce cas, une nouvelle base de code est créée pour s’aligner sur une approche [cloud native](https://azure.microsoft.com/overview/cloudnative/).

Les moteurs les plus courants sont les suivants :

* Accélérer l’innovation
* Créer des applications plus rapidement
* Réduire les coûts d’exploitation

Facteurs d’analyse quantitative :

* Taille des ressources d’application (processeur, mémoire, stockage)
* Dépendances (trafic réseau)
* Trafic utilisateur (consultations de page, temps passé sur une page, temps de chargement)
* Plateforme de développement (langages, plateforme de données, services de niveau intermédiaire)

Facteurs d’analyse qualitative :

* Baisse de la satisfaction des utilisateurs finaux
* Processus d’entreprise limités dans leur fonctionnalité
* Amélioration potentielle des coûts, de l’expérience ou du chiffre d’affaires

## <a name="replace"></a>Replace

Les solutions sont généralement implémentées à l’aide de la technologie et de l’approche les mieux adaptées sur le moment. Dans certains cas, les applications SaaS peuvent fournir toutes les fonctionnalités nécessaires à l’application hébergée. Dans ces scénarios, une charge de travail peut être programmée pour un remplacement futur, ce qui évite tout effort de transformation.

Les moteurs les plus courants sont les suivants :

* Normaliser les bonnes pratiques du secteur
* Accélérer l’adoption des approches basées sur les processus d’entreprise
* Réallouer les investissements de développement dans des applications qui créent un avantage compétitif ou autre avantage

Facteurs d’analyse quantitative :

* Réductions des coûts d’exploitation généraux
* Taille de la machine virtuelle (processeur, mémoire, stockage)
* Dépendances (trafic réseau)
* Ressources à mettre hors service

Facteurs d’analyse qualitative :

* Analyse coûts-avantages de l’architecture actuelle par rapport à une solution SaaS
* Diagramme des processus métier
* Schémas de données
* Processus personnalisés ou automatisés

## <a name="next-steps"></a>Étapes suivantes

Collectivement, ces 5 R de la rationalisation ne peuvent pas être appliqués à un patrimoine numérique en vue d’effectuer des décisions de rationalisation concernant l’état futur de chaque application.

> [!div class="nextstepaction"]
> [Qu’est-ce qu’un patrimoine numérique ?](overview.md)
