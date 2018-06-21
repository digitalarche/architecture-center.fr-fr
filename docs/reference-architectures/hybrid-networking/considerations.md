---
title: Choisir une solution pour connecter un réseau local à Azure
description: Comparatif des architectures de référence pour connecter un réseau local à Azure.
author: telmosampaio
ms.date: 04/06/2017
ms.openlocfilehash: 274b9df1817632a7f3eaafa8bf02e965fdc3feea
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2017
ms.locfileid: "24541047"
---
# <a name="choose-a-solution-for-connecting-an-on-premises-network-to-azure"></a>Choisir une solution pour connecter un réseau local à Azure

Cet article compare les options disponibles pour connecter un réseau local à un réseau virtuel (VNet) Azure. Pour chaque option, nous fournissons une architecture de référence et une solution pouvant être déployée.

## <a name="vpn-connection"></a>Connexion VPN

Utilisez un réseau privé virtuel (VPN) pour connecter votre réseau local à un réseau virtuel Azure via un tunnel VPN IPSec.

Cette architecture est idéale pour les applications hybrides avec un trafic peu volumineux entre le matériel sur site et le cloud, ou dans le cas où vous êtes disposé à accepter une latence un peu plus élevée pour bénéficier de la flexibilité et de la puissance de la technologie cloud.

**Avantages**

- Cette méthode est facile à configurer.

**Défis**

- Vous avez besoin d’un appareil VPN local.
- Bien que Microsoft garantit une disponibilité de 99,9 % pour chaque passerelle VPN, ce contrat SLA couvre uniquement la passerelle VPN, et pas votre connexion réseau à la passerelle.
- Une connexion VPN via la passerelle VPN Azure prend actuellement en charge une bande passante maximale de 200 Mbits/s. Vous devrez peut-être partitionner votre réseau virtuel Azure sur plusieurs connexions VPN si vous pensez dépasser ce débit.

**[En savoir plus...][vpn]**

## <a name="azure-expressroute-connection"></a>Connexion Azure ExpressRoute

Les connexions ExpressRoute utilisent une connexion privée et dédiée via un fournisseur de connectivité tiers. La connexion privée étend votre réseau local à Azure. 

Cette architecture est idéale pour les applications hybrides exécutant des charges de travail critiques et volumineuses qui nécessitent un niveau élevé d’évolutivité. 

**Avantages**

- La bande passante disponible est plus élevée, avec jusqu’à 10 Gbits/s selon le fournisseur de connectivité choisi.
- Cette méthode prend en charge la mise à l’échelle dynamique de la bande passante afin de réduire les coûts pendant les périodes de faible demande. Toutefois, tous les fournisseurs de connectivité ne proposent pas cette option.
- En fonction du fournisseur de connectivité choisi, votre organisation peut bénéficier d’un accès direct aux clouds nationaux.
- Vous bénéficiez d’un contrat SLA garantissant une disponibilité de 99,9 % sur l’ensemble de la connexion.

**Défis**

- Cette méthode peut être difficile à configurer. La création d’une connexion ExpressRoute nécessite l’intervention d’un fournisseur de connectivité tiers. Le fournisseur est responsable de l’approvisionnement de la connexion réseau.
- Cette méthode requiert des routeurs locaux offrant une large bande passante.

**[En savoir plus...][expressroute]**

## <a name="expressroute-with-vpn-failover"></a>ExpressRoute avec basculement VPN

Cette option combine les deux méthodes précédentes : elle utilise ExpressRoute dans des conditions normales, mais bascule vers une connexion VPN en cas de perte de connectivité au niveau du circuit ExpressRoute.

Cette architecture est idéale pour les applications hybrides qui nécessitent une bande passante ExpressRoute plus élevée, ainsi qu’une connectivité réseau hautement disponible. 

**Avantages**

- Haute disponibilité si le circuit ExpressRoute échoue, même lorsque la connexion de secours intervient sur un réseau de bande passante inférieure.

**Défis**

- Cette méthode est difficile à configurer. Vous devez configurer à la fois une connexion VPN et un circuit ExpressRoute.
- Cette méthode requiert un matériel redondant (appliances VPN) et une connexion de passerelle VPN Azure redondante pour laquelle vous devrez payer des frais supplémentaires.

**[En savoir plus...][expressroute-vpn-failover]**

<!-- links -->
[expressroute]: ./expressroute.md
[expressroute-vpn-failover]: ./expressroute-vpn-failover.md
[vpn]: ./vpn.md