---
title: Déployer des appliances virtuelles réseau hautement disponibles
titleSuffix: Azure Reference Architectures
description: Déployez des appliances virtuelles réseau dans un environnement à haute disponibilité.
author: telmosampaio
ms.date: 12/08/2018
ms.custom: seodec18
ms.openlocfilehash: d3f9017db1bbf9741b10db16eb5a3dbab78f1160
ms.sourcegitcommit: 7d21aec9d9de0004ac777c1d1e364f53aac2350d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2018
ms.locfileid: "53120750"
---
# <a name="deploy-highly-available-network-virtual-appliances"></a>Déployer des appliances virtuelles réseau hautement disponibles

Cet article explique comment déployer des appliances virtuelles réseau pour la haute disponibilité dans Azure. Une appliance virtuelle réseau est généralement utilisée pour contrôler le flux de trafic réseau à partir d’un réseau de périmètre, également appelé DMZ, vers d’autres réseaux ou sous-réseaux. Pour en savoir plus sur l’implémentation d’une zone DMZ dans Azure, consultez [Microsoft cloud services and network security][cloud-security] (Services de cloud computing Microsoft et sécurité réseau). L’article inclut des exemples d’architecture pour l’entrée ou la sortie uniquement, et pour les deux à la fois.

**Configuration requise :** Cet article nécessite une connaissance élémentaire des réseaux Azure, des [équilibreurs de charge Azure][lb-overview] et des [routes définies par l’utilisateur][udr-overview].

## <a name="architecture-diagrams"></a>Diagrammes architecturaux

Une appliance virtuelle réseau peut être déployée sur une zone DMZ dans bon nombre d’architectures différentes. Par exemple, l’exemple suivant illustre l’utilisation d’une [appliance virtuelle réseau] [ nva-scenario] pour l’entrée.

![[0]][0]

Dans cette architecture, l’appliance virtuelle réseau assure une limite réseau sécurisée par la vérification de l’ensemble du trafic réseau entrant et sortant et la transmission uniquement du trafic répondant aux règles de sécurité du réseau. Toutefois, le fait que tout le trafic réseau doive traverser l’appliance virtuelle réseau signifie que cette dernière est un point unique de défaillance au sein du réseau. Si celle-ci échoue, il n’existe aucun autre chemin d’accès pour le trafic réseau et tous les sous-réseaux principaux sont indisponibles.

Pour rendre une appliance virtuelle réseau hautement disponible, déployez plusieurs appliances virtuelles réseau dans un groupe à haute disponibilité.

Les architectures suivantes décrivent les ressources et la configuration nécessaire pour des appliances virtuelles réseau hautement disponibles :

| Solution | Avantages | Considérations |
| --- | --- | --- |
| [Entrée avec appliances virtuelles réseau de couche 7][ingress-with-layer-7] |Tous les nœuds d’appliance virtuelle réseau sont actifs. |Nécessite une appliance virtuelle réseau pouvant arrêter les connexions et utiliser SNAT<br/> Requiert un ensemble distinct d’appliances virtuelles réseau pour le trafic provenant d’Internet et d’Azure <br/> Peut uniquement être utilisé pour le trafic provenant de l’extérieur d’Azure |
| [Sortie avec appliances virtuelles réseau de couche 7][egress-with-layer-7] |Tous les nœuds d’appliance virtuelle réseau sont actifs. | Nécessite une appliance virtuelle réseau pouvant arrêter les connexions et implémente la traduction des adresses réseau source (SNAT)
| [Entrée-sortie avec appliances virtuelles réseau de couche 7][ingress-egress-with-layer-7] |Tous les nœuds sont actifs.<br/>Capable de gérer le trafic provenant d’Azure. |Nécessite une appliance virtuelle réseau pouvant arrêter les connexions et utiliser SNAT<br/>Requiert un ensemble distinct d’appliances virtuelles réseau pour le trafic provenant d’Internet et d’Azure |
| [Commutateur PIP-UDR][pip-udr-switch] |Ensemble unique d’appliances virtuelles réseau pour tout le trafic<br/>Peut gérer tout le trafic (aucune limite sur les règles de port) |Actif/Passif<br/>Requiert un processus de basculement |
| [PIP-UDR sans SNAT](#pip-udr-nvas-without-snat) | Ensemble unique d’appliances virtuelles réseau pour tout le trafic<br/>Peut gérer tout le trafic (aucune limite sur les règles de port)<br/>Ne nécessite pas de configuration SNAT pour les requêtes entrantes |Actif/Passif<br/>Requiert un processus de basculement<br/>Le sondage et la logique de basculement sont exécutés en dehors du réseau virtuel |

## <a name="ingress-with-layer-7-nvas"></a>Entrée avec appliances virtuelles réseau de couche 7

La figure suivante montre une architecture à haute disponibilité qui implémente une zone DMZ d’entrée derrière un équilibreur de charge accessible sur Internet. Cette architecture est conçue pour fournir une connectivité HTTP ou HTTPS aux charges de travail Azure pour le trafic de couche 7 :

![[1]][1]

L’avantage de cette architecture réside dans le fait que toutes les appliances virtuelles réseau sont actives et qu’en cas d’échec de l’une d’entre elles l’équilibreur de charge dirige le trafic réseau vers l’autre appliance virtuelle. Les deux appliances virtuelles réseau acheminent le trafic vers l’équilibreur de charge interne. Ainsi, le trafic est maintenu tant que l’une d’entre elles est active. Les appliances virtuelles réseau doivent arrêter le trafic SSL destiné aux machines virtuelles de la couche web. Ces appliances virtuelles réseau ne peuvent pas être étendues pour gérer le trafic local, car celui-ci nécessite un autre ensemble dédié d’appliances virtuelles réseau possédant leurs propres itinéraires réseau.

> [!NOTE]
> Cette architecture est utilisée dans l’architecture de référence de la [zone DMZ située entre Azure et votre centre de données local][dmz-on-prem] et celle de la [zone DMZ située entre Azure et Internet][dmz-internet]. Chacune de ces architectures de référence inclue une solution de déploiement utilisable. Suivez les liens ci-après pour plus d’informations.

## <a name="egress-with-layer-7-nvas"></a>Sortie avec appliances virtuelles réseau de couche 7

L’architecture précédente peut être développée de manière à fournir une zone DMZ de sortie pour les requêtes provenant de la charge de travail Azure. L’architecture suivante est conçue pour assurer une haute disponibilité des appliances virtuelles réseau de la zone DMZ pour le trafic de couche 7, tel que HTTP ou HTTPS :

![[2]][2]

Dans cette architecture, tout le trafic provenant d’Azure est acheminé vers un équilibreur de charge interne. Ce dernier répartit les requêtes sortantes entre un ensemble d’appliances virtuelles réseau. Celles-ci dirigent le trafic vers Internet à l’aide de leurs adresses IP publiques individuelles.

> [!NOTE]
> Cette architecture est utilisée dans l’architecture de référence de la [zone DMZ située entre Azure et votre centre de données local][dmz-on-prem] et celle de la [zone DMZ située entre Azure et Internet][dmz-internet]. Chacune de ces architectures de référence inclue une solution de déploiement utilisable. Suivez les liens ci-après pour plus d’informations.

## <a name="ingress-egress-with-layer-7-nvas"></a>Entrée-sortie avec appliances virtuelles réseau de couche 7

Les deux architectures précédentes comportaient une zone DMZ distincte pour l’entrée et la sortie. L’architecture suivante montre comment créer une zone DMZ pouvant être utilisée pour l’entrée et la sortie du trafic de couche 7, tel que HTTP ou HTTPS :

![[4]][4]

Dans cette architecture, les appliances virtuelles réseau traitent les requêtes entrantes à partir de la passerelle d’application. Les appliances virtuelles réseau traitent également des requêtes sortantes issues des machines virtuelles de la charge de travail du pool principal de l’équilibreur de charge. Étant donné que le trafic entrant est routé avec une passerelle d’application et que le trafic sortant est routé avec un équilibreur de charge, les appliances virtuelles réseau sont responsables du maintien de l’affinité de session. Autrement dit, la passerelle d’application gère un mappage de requêtes entrantes et sortantes afin de pouvoir envoyer la réponse correcte au demandeur d’origine. Toutefois, l’équilibreur de charge interne n’a pas accès aux mappages de la passerelle d’application et utilise sa propre logique pour envoyer des réponses aux appliances virtuelles réseau. Il est possible que l’équilibreur de charge puisse envoyer une réponse à une appliance virtuelle réseau qui n’a pas initialement reçu la requête issue de la passerelle d’application. Dans ce cas, les appliances virtuelles réseau doivent communiquer et se transférer la réponse entre elles afin que l’appliance virtuelle réseau appropriée puisse transférer la réponse vers la passerelle d’application.

> [!NOTE]
> Vous pouvez également résoudre le problème de routage asymétrique en s’assurant que les appliances virtuelles réseau effectuent la traduction des adresses réseau sources entrantes (SNAT). Cela remplacerait l’IP source d’origine du demandeur par une des adresses IP de l’appliance virtuelle réseau utilisée sur le flux entrant. Cela garantit de pouvoir utiliser plusieurs appliances virtuelles réseau à la fois, tout en conservant la symétrie de l’itinéraire.

## <a name="pip-udr-switch-with-layer-4-nvas"></a>Commutateur PIP-UDR avec appliances virtuelles réseau de couche 4

L’architecture suivante illustre une architecture comportant une appliance virtuelle réseau active et une autre appliance passive. Cette architecture gère à la fois l’entrée et la sortie pour le trafic de couche 4 :

![[3]][3]

> [!TIP]
> Pour cette architecture, une solution complète est disponible sur [GitHub][pnp-ha-nva].

Cette architecture est similaire à la première architecture abordée dans cet article. Cette architecture incluait une seule appliance virtuelle réseau acceptant et filtrant les requêtes entrantes de couche 4. Cette architecture ajoute une deuxième appliance virtuelle réseau pour garantir une haute disponibilité. Si l’appliance virtuelle réseau échoue, l’appliance virtuelle réseau passive est activée et les UDR et PIP sont modifiés de manière à pointer vers les cartes d’interface réseau de l’appliance virtuelle réseau active. Ces modifications apportées à l’UDR et au PIP peuvent être effectuées manuellement ou à l’aide d’un processus automatisé. Le processus automatisé est généralement un démon ou tout autre service d’analyse s’exécutant dans Azure. Il interroge une sonde d’intégrité située sur l’appliance virtuelle réseau active et déclenche le commutateur UDR et PIP lorsqu’il détecte un échec de cette dernière.

L’illustration précédente montre un exemple de cluster [ZooKeeper][zookeeper] proposant un démon à haute disponibilité. Au sein du cluster ZooKeeper, un quorum de nœuds choisit une amorce de début. Si l’amorce de début échoue, les nœuds restants en élisent une nouvelle. Pour cette architecture, le nœud de l’amorce de début exécute le démon qui interroge le point de terminaison d’intégrité situé sur l’appliance virtuelle réseau. Si l’appliance virtuelle réseau ne répond pas à la sonde d’intégrité, le démon active l’appliance virtuelle réseau. Le démon appelle ensuite l’API REST Azure pour retirer le PIP de l’appliance virtuelle réseau défaillante et l’associe à l’appliance virtuelle réseau qui vient d’être activée. Le démon modifie ensuite l’UDR de manière à pointer vers l’adresse IP interne de l’appliance virtuelle réseau venant d’être activée.

N’incluez pas les nœuds Zookeeper dans un sous-réseau uniquement accessible à l’aide d’un itinéraire incluant l’appliance virtuelle réseau. Dans le cas contraire, les nœuds Zookeeper sont inaccessibles en cas d’échec de l’appliance virtuelle réseau. Si le démon échoue pour une raison quelconque, vous ne pourrez accéder à aucun des nœuds ZooKeeper pour diagnostiquer le problème.

Pour voir la solution complète, y compris un exemple de code, consultez les fichiers du [dépôt GitHub][pnp-ha-nva].

## <a name="pip-udr-nvas-without-snat"></a>Appliances virtuelles réseau PIP-UDR sans SNAT

Cette architecture utilise deux machines virtuelles Azure pour héberger le pare-feu d’appliance virtuelle réseau dans une configuration de type actif-passif qui prend en charge le basculement automatisé, mais qui ne nécessite pas de SNAT (Source Network Address Translation).

![Appliances virtuelles réseau PIP-UDR sans architecture SNAT](./images/nva-ha/pip-udr-without-snat.png)

> [!TIP]
> Pour cette architecture, une solution complète est disponible sur [GitHub][ha-nva-fo].

Cette solution est conçue pour les clients Azure qui ne peuvent pas configurer le SNAT pour les requêtes dans leur pare-feu d’appliance virtuelle réseau. Le SNAT masque l’adresse IP client source d’origine. Si vous devez journaliser les adresses IP d’origine ou les utiliser dans d’autres composants de sécurité multiniveau situés derrière vos appliances virtuelles réseau, cette solution offre une approche de base.

Le basculement des entrées de la table de routes définies par l’utilisateur est automatisé par une adresse de tronçon suivant qui est définie sur l’adresse IP d’une interface, sur la machine virtuelle où le pare-feu d’appliance virtuelle réseau est actif. La logique du basculement automatisé est hébergée dans une application de fonction que vous créez à l’aide d’[Azure Functions](/azure/azure-functions/). Le code de basculement s’exécute comme une fonction serverless dans Azure Functions. Le déploiement est pratique, économique ainsi que facile à gérer et à personnaliser. En outre, l’application de fonction est hébergée dans Azure Functions.Elle n’a donc aucune dépendance au réseau virtuel. Si les modifications apportées au réseau virtuel impactent les pare-feu d’appliance virtuelle réseau, l’application de fonction continue de s’exécuter indépendamment. Les tests sont également plus précis, car ils ont lieu en dehors du réseau virtuel et utilisent la même route que les requêtes client entrantes.

Pour vérifier la disponibilité du pare-feu d’appliance virtuelle réseau, le code de l’application de fonction le sonde de deux manières :

- En supervisant l’état des machines virtuelles Azure qui hébergent le pare-feu d’appliance virtuelle réseau

- En vérifiant si un port est ouvert entre le pare-feu et le serveur web back-end Avec cette option, l’appliance virtuelle réseau doit exposer un socket via PIP pour le code de l’application de fonction à tester.

Vous choisissez le type de sonde que vous souhaitez utiliser au moment de configurer l’application de fonction. Pour voir la solution complète, y compris un exemple de code, consultez les fichiers du [dépôt GitHub][ha-nva-fo].

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [implémenter une zone DMZ située entre Azure et votre centre de données local] [ dmz-on-prem] à l’aide des appliances virtuelles réseau de couche 7.
- Découvrez comment [implémenter une zone DMZ située entre Azure et Internet][dmz-internet] à l’aide des appliances virtuelles réseau de couche 7.
- [Problèmes d’appliance virtuelle réseau dans Azure](/azure/virtual-network/virtual-network-troubleshoot-nva)

<!-- links -->

[cloud-security]: /azure/best-practices-network-security
[dmz-on-prem]: ./secure-vnet-hybrid.md
[dmz-internet]: ./secure-vnet-dmz.md
[egress-with-layer-7]: #egress-with-layer-7-nvas
[ingress-with-layer-7]: #ingress-with-layer-7-nvas
[ingress-egress-with-layer-7]: #ingress-egress-with-layer-7-nvas
[lb-overview]: /azure/load-balancer/load-balancer-overview/
[nva-scenario]: /azure/virtual-network/virtual-network-scenario-udr-gw-nva/
[pip-udr-switch]: #pip-udr-switch-with-layer-4-nvas
[udr-overview]: /azure/virtual-network/virtual-networks-udr-overview/
[zookeeper]: https://zookeeper.apache.org/
[pnp-ha-nva]: https://github.com/mspnp/ha-nva
[ha-nva-fo]: https://aka.ms/ha-nva-fo

<!-- images -->

[0]: ./images/nva-ha/single-nva.png "Architecture d’appliances virtuelle réseau unique"
[1]: ./images/nva-ha/l7-ingress.png "Entrée de couche 7"
[2]: ./images/nva-ha/l7-ingress-egress.png "Sortie de couche 7"
[3]: ./images/nva-ha/active-passive.png "Cluster actif-passif"
[4]: ./images/nva-ha/l7-ingress-egress-ag.png
