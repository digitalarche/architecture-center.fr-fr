---
title: 'Framework d’adoption du cloud : Réseaux à définition logicielle'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Discussion sur les réseaux définis par logiciel en tant que service de base dans les migrations Azure
author: rotycenh
ms.openlocfilehash: d164ba488552715dc97719329ae9de3fcf5d83ed
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58243340"
---
# <a name="caf-software-defined-network-decision-guide"></a>Framework d’adoption du cloud : Guide de décision concernant le SDN (Software Defined Network)

La mise en réseau définie par logiciel (ou SDN, pour Software Defined Networking) est une architecture réseau conçue pour permettre une fonctionnalité réseau virtualisée qui peut être gérée, configurée et modifiée de manière centralisée par logiciel. Le SDN fournit une couche d’abstraction sur l’infrastructure de réseau physique et permet l’équivalent virtualisé des routeurs physiques, pare-feu et autres matériels réseau que l’on trouve dans un réseau sur site.

SDN permet au personnel informatique de configurer et de déployer des structures et des capacités réseau qui prennent en charge les besoins de charge de travail à l’aide de ressources virtualisées. La souplesse de la gestion des déploiements par logiciel permet de modifier rapidement les ressources réseau et de prendre en charge les modèles de déploiement à la fois agiles et traditionnels. Les réseaux virtualisés créés avec la technologie SDN sont essentiels à la création de réseaux sécurisés sur une plate-forme de cloud public.

## <a name="networking-decision-guide"></a>Guide de décision concernant la mise en réseau

![Options de mise en réseau, de la moins complexe à la plus complexe, dans l’ordre des liens ci-dessous](../../_images/discovery-guides/discovery-guide-sdn.png)

Passer à : [PaaS uniquement](paas-only.md) | [Natif cloud](cloud-native.md) | | [Zone DMZ cloud](cloud-dmz.md) [Hybride](hybrid.md) | [Modèle hub-and-spoke](hub-spoke.md) | [En savoir plus](#learn-more)

Le SDN fournit plusieurs options avec différents degrés de complexité et de tarification. Le guide de découverte ci-dessus fournit une référence pour personnaliser rapidement ces options afin de s’aligner au mieux sur des stratégies commerciales et technologiques spécifiques.

Le point d’inflexion de ce guide dépend de plusieurs décisions clés que votre équipe Cloud Strategy a prises avant de prendre des décisions sur l’architecture réseau. Les plus importantes sont les décisions concernant votre [Définition de la succession numérique](../../digital-estate/overview.md) et [Conception de l’abonnement](../subscriptions/overview.md). (ce qui peut également nécessiter des données provenant de décisions prises dans le cadre de votre comptabilité dans le cloud et de vos stratégies sur les marchés mondiaux).

Les petits déploiements dans une seule région de moins de 1 000 VM sont moins susceptibles d’être affectés de façon significative par ce point d’inflexion. Inversement, des efforts d’adoption importants avec plus de 1 000 machines virtuelles, de multiples unités commerciales ou de multiples marchés géostratégies pourraient être considérablement affectés par votre décision SDN et ce point d’inflexion clé.

## <a name="choosing-the-right-virtual-networking-architectures"></a>Choisir les bonnes architectures de réseaux virtuels

Cette section développe le guide de décision pour vous aider à choisir les bonnes architectures de réseau virtuel.

Il existe de nombreuses façons de mettre en œuvre les technologies SDN pour créer des réseaux virtuels cloud. La façon dont vous structurez les réseaux virtuels utilisés dans votre migration et la façon dont ces réseaux interagissent avec votre infrastructure informatique existante dépendra d’une combinaison des exigences en matière de charge de travail et de gouvernance.

Lors de la planification de l’architecture de réseau virtuel ou de la combinaison d’architectures à prendre en compte lors de la planification de votre migration vers le Cloud, prenez en compte les questions suivantes pour déterminer ce qui convient à votre entreprise :

| Question | PaaS uniquement | Natif cloud | Zone DMZ cloud | Hybride | Hub-and-Spoke |
|-----|-----|-----|-----|-----|-----|
| Votre charge de travail utilisera-t-elle uniquement les services PaaS ? Ses besoins en capacités réseau seront-ils supérieurs à celles fournies par les services eux-mêmes ? | OUI | Non  | Non  | Non  | Non  |
| Votre charge de travail nécessite-t-elle une intégration avec des applications sur site ? | Non  | Non  | OUI | OUI | OUI |
| Avez-vous établi des stratégies de sécurité avancées et une connectivité sécurisée entre vos locaux et vos réseaux cloud ? | Non  | Non  | Non  | OUI | OUI |
| Votre charge de travail nécessite-t-elle des services d’authentification non pris en charge par les services d’identité cloud, ou avez-vous besoin d’un accès direct aux contrôleurs de domaine locaux ? | Non  | Non  | Non  | OUI | OUI |
| Devrez-vous déployer et gérer un grand nombre de machines virtuelles et de charges de travail ? | Non  | Non  | Non  | Non  | OUI |
| Devrez-vous fournir une gestion centralisée et une connectivité locale tout en déléguant le contrôle des ressources à des équipes de travail individuelles ? | Non  | Non  | Non  | Non  | OUI |

## <a name="virtual-networking-architectures"></a>Architectures de réseaux virtuels

En savoir plus sur les principales architectures réseau définies par le logiciel :

- [**PaaS uniquement**](paas-only.md) : Les produits PaaS (Platform as a Service) prennent en charge un ensemble limité de fonctions réseau intégrées et peuvent ne pas nécessiter un réseau défini explicitement par un logiciel pour répondre aux exigences de charge de travail.
- [**Natif cloud**](cloud-native.md) : Un réseau virtuel natif du cloud est l’architecture réseau par défaut définie par le logiciel lors du déploiement de ressources sur une plate-forme cloud.
- [**Zone DMZ cloud**](cloud-dmz.md) : Fournit une connectivité limitée entre vos locaux et le réseau cloud qui est sécurisé par la mise en œuvre d’une zone démilitarisée sur l’environnement cloud.
- [**Hybride**](hybrid.md) : L’architecture de réseau cloud hybride permet aux réseaux virtuels d’accéder à vos ressources sur site et vice versa.
- [**Hub-and-spoke**](hub-spoke.md) : L’architecture hub-and-spoke vous permet de gérer de manière centralisée la connectivité externe et les services partagés, d’isoler les charges de travail individuelles et de surmonter les limites d’abonnement potentielles.

## <a name="learn-more"></a>En savoir plus

Pour plus d’informations sur la mise en réseau définie par logiciel (ou SDN, pour Software-Defined Networking) dans la plateforme Azure, consultez la rubrique suivante.

- [Réseau virtuel Azure](/azure/virtual-network/virtual-networks-overview) : sur Azure, la capacité fondamentale du SDN est fournie par un réseau virtuel Azure, qui agit comme un cloud analogue aux réseaux physiques locaux. Les réseaux virtuels servent également de limite d’isolement par défaut entre les ressources de la plate-forme.
- [Bonnes pratiques en matière de sécurité des réseaux Azure](/azure/security/azure-security-network-security-best-practices) : suggestions de l’équipe Azure Security sur la façon de configurer vos réseaux virtuels pour minimiser les vulnérabilités de sécurité.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment les équipes opérationnelles utilisent les journaux, la surveillance et la création de rapports pour gérer la conformité des charges de travail dans le cloud en matière d’intégrité et de stratégie.

> [!div class="nextstepaction"]
> [Journaux et rapports](../log-and-report/overview.md)
