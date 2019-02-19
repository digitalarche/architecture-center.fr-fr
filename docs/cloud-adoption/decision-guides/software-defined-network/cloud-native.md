---
title: 'Framework d’adoption du cloud : Réseau à définition logicielle – Natif dans le cloud'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Présentation des services de réseau virtuel natifs dans le cloud
author: rotycenh
ms.openlocfilehash: c6200491bc9ba35a9f00e0003e51716b58628980
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900704"
---
# <a name="software-defined-networks-cloud-native"></a>Réseaux à définition logicielle : Cloud natif

Un réseau virtuel natif dans nuage est indispensable lors du déploiement de ressources IaaS telles que des machines virtuelles sur une plateforme cloud. L’accès à des réseaux virtuels à partir de sources externes, similaires au web, nécessite un approvisionnement explicite. Ces types de réseaux virtuels prennent en charge la création de sous-réseaux, de règles d’acheminement, de pare-feu virtuels et de dispositifs de gestion du trafic.

Un réseau virtuel natif dans le cloud ne dépend pas de ressources locales ou d’autres ressources non-cloud de votre organisation pour prendre en charge les charges de travail hébergées dans le cloud. Toutes les ressources requises sont approvisionnées soit dans le réseau virtuel, soit à l’aide d’offres PaaS gérées.

## <a name="cloud-native-assumptions"></a>Postulats concernant le caractère natif dans le cloud

Le déploiement d’un réseau virtuel natif dans le cloud suppose ce qui suit :

- Les charges de travail que vous déployez sur le réseau virtuel ne dépendent pas d’applications ou de services qui ne sont accessibles que depuis votre réseau local. À moins de fournir des points de terminaison accessibles sur l’Internet public, les applications et services locaux hébergés en interne ne sont pas utilisables par des ressources hébergées sur une plateforme cloud.
- La gestion des identités et le contrôle d’accès de votre charge de travail dépendent des services d’identité de la plateforme cloud ou de serveurs IaaS hébergés dans votre environnement cloud. Vous n’aurez pas besoin de vous connecter directement à des services d’identité hébergés localement ou dans d’autres emplacements externes.
- Il n’est pas nécessaire que vos services d’identité prennent en charge la connexion unique (SSO) avec des annuaires locaux.

Les réseaux virtuels natifs dans le cloud n’ont pas de dépendances externes. Ainsi, leur déploiement et leur configuration sont simples, de sorte que cette architecture constitue souvent un choix optimal à des fins expérimentales, ou pour d’autres déploiements autonomes plus petits ou à itérations rapides.

Les autres problèmes que votre équipe chargée de l’adoption du cloud doit prendre en compte lors de la discussion d’une architecture réseau virtuel native dans le cloud sont les suivants :

- Des charges de travail existantes conçues pour s’exécuter dans un centre de données local peuvent nécessiter des modifications importantes pour tirer parti de fonctionnalités cloud telles que des services de stockage ou d’authentification.
- Les réseaux natifs dans le cloud sont gérés uniquement à l’aide des outils de gestion de plateforme cloud. Par conséquent, la gestion et la stratégie peuvent diverger au fil du temps par rapport aux normes informatiques existantes.

## <a name="learn-more"></a>En savoir plus

Pour plus d’informations sur les réseaux virtuels natifs dans le cloud sur la plateforme Azure, consultez la rubrique suivante.

- [Réseau virtuel Azure : guides pratiques](/azure/virtual-network/virtual-network-vnet-plan-design-arm). Les réseaux virtuels Azure nouvellement créés sont natifs du cloud par défaut. Aidez-vous de ces guides pour planifier la conception et le déploiement de vos réseaux virtuels.
- [Limites d’abonnement : Mise en réseau](/azure/azure-subscription-service-limits?toc=%2fazure%2fvirtual-network%2ftoc.json#networking-limits). Un réseau virtuel et des ressources connectées ne peuvent exister que dans un abonnement unique, et sont liés par des limites d’abonnement.
