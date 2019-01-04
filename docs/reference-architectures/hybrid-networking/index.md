---
title: Connecter un réseau local à Azure
titleSuffix: Azure Reference Architectures
description: Comparatif des architectures de référence pour connecter un réseau local à Azure.
author: telmosampaio
ms.date: 07/02/2018
ms.openlocfilehash: f13249f225ad7ab5072de2b2175cdc2ffb6d0074
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011053"
---
# <a name="choose-a-solution-for-connecting-an-on-premises-network-to-azure"></a>Choisir une solution pour connecter un réseau local à Azure

Cet article compare les options disponibles pour connecter un réseau local à un réseau virtuel (VNet) Azure. Pour chaque option, une architecture de référence plus détaillée est disponible.

## <a name="vpn-connection"></a>Connexion VPN

Une [passerelle VPN](/azure/vpn-gateway/vpn-gateway-about-vpngateways) est un type de passerelle de réseau virtuel qui envoie le trafic chiffré entre un réseau virtuel Azure et un emplacement local. Le trafic chiffré est envoyé via le réseau Internet public.

Cette architecture est idéale pour les applications hybrides avec un trafic peu volumineux entre le matériel sur site et le cloud, ou dans le cas où vous êtes disposé à accepter une latence un peu plus élevée pour bénéficier de la flexibilité et de la puissance de la technologie cloud.

### <a name="benefits"></a>Avantages

- Cette méthode est facile à configurer.

### <a name="challenges"></a>Défis

- Vous avez besoin d’un appareil VPN local.
- Même si Microsoft garantit une disponibilité de 99,9 % pour chaque passerelle VPN, ce contrat [SLA](https://azure.microsoft.com/support/legal/sla/vpn-gateway/) couvre uniquement la passerelle VPN, et non votre connexion réseau à la passerelle.
- Une connexion VPN via la passerelle VPN Azure prend actuellement en charge une bande passante maximale de 1,25 Mbits/s. Vous devrez peut-être partitionner votre réseau virtuel Azure sur plusieurs connexions VPN si vous pensez dépasser ce débit.

### <a name="reference-architecture"></a>Architecture de référence

- [Réseau hybride avec une passerelle VPN](./vpn.md)

<!-- markdownlint-disable MD024 -->

## <a name="azure-expressroute-connection"></a>Connexion Azure ExpressRoute

Les connexions [ExpressRoute](/azure/expressroute/) utilisent une connexion privée et dédiée via un fournisseur de connectivité tiers. La connexion privée étend votre réseau local à Azure.

Cette architecture est idéale pour les applications hybrides exécutant des charges de travail critiques et volumineuses qui nécessitent un niveau élevé d’évolutivité.

### <a name="benefits"></a>Avantages

- La bande passante disponible est plus élevée, avec jusqu’à 10 Gbits/s selon le fournisseur de connectivité choisi.
- Cette méthode prend en charge la mise à l’échelle dynamique de la bande passante afin de réduire les coûts pendant les périodes de faible demande. Toutefois, tous les fournisseurs de connectivité ne proposent pas cette option.
- En fonction du fournisseur de connectivité choisi, votre organisation peut bénéficier d’un accès direct aux clouds nationaux.
- Vous bénéficiez d’un contrat SLA garantissant une disponibilité de 99,9 % sur l’ensemble de la connexion.

### <a name="challenges"></a>Défis

- Cette méthode peut être difficile à configurer. La création d’une connexion ExpressRoute nécessite l’intervention d’un fournisseur de connectivité tiers. Le fournisseur est responsable de l’approvisionnement de la connexion réseau.
- Cette méthode requiert des routeurs locaux offrant une large bande passante.

### <a name="reference-architecture"></a>Architecture de référence

- [Réseau hybride avec ExpressRoute](./expressroute.md)

## <a name="expressroute-with-vpn-failover"></a>ExpressRoute avec basculement VPN

Cette option combine les deux méthodes précédentes : elle utilise ExpressRoute dans des conditions normales, mais bascule vers une connexion VPN en cas de perte de connectivité au niveau du circuit ExpressRoute.

Cette architecture est idéale pour les applications hybrides qui nécessitent une bande passante ExpressRoute plus élevée, ainsi qu’une connectivité réseau hautement disponible.

### <a name="benefits"></a>Avantages

- Haute disponibilité si le circuit ExpressRoute échoue, même lorsque la connexion de secours intervient sur un réseau de bande passante inférieure.

### <a name="challenges"></a>Défis

- Cette méthode est difficile à configurer. Vous devez configurer à la fois une connexion VPN et un circuit ExpressRoute.
- Cette méthode requiert un matériel redondant (appliances VPN) et une connexion de passerelle VPN Azure redondante pour laquelle vous devrez payer des frais supplémentaires.

### <a name="reference-architecture"></a>Architecture de référence

- [Réseau hybride avec ExpressRoute et basculement du VPN](./expressroute-vpn-failover.md)

<!-- markdownlint-disable MD024 -->

## <a name="hub-spoke-network-topology"></a>Topologie de réseau hub-and-spoke

Une topologie de réseau hub-and-spoke consiste à isoler des charges de travail tout en partageant des services tels que l’identité et la sécurité. Le hub est un réseau virtuel (VNet) dans Azure qui centralise la connectivité à votre réseau local. Les rayons sont des réseaux virtuels qui s’associent au hub. Les services partagés sont déployés dans le hub, tandis que les charges de travail individuelles sont déployées en tant que rayons.

### <a name="reference-architectures"></a>Architectures de référence

- [Topologie hub-and-spoke](./hub-spoke.md)
- [Topologie hub-and-spoke avec des services partagés](./shared-services.md)
