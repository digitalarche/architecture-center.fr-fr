---
title: Implémentation d’une architecture réseau hybride sécurisée dans Azure
description: Comment implémenter une architecture réseau hybride sécurisée dans Azure.
author: telmosampaio
ms.date: 11/23/2016
pnp.series.title: Network DMZ
pnp.series.prev: ./index
pnp.series.next: secure-vnet-dmz
cardTitle: DMZ between Azure and on-premises
ms.openlocfilehash: 81dea2e4439d5a01ebb88ab86dc0a59609bb7bc3
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/06/2018
ms.locfileid: "30849652"
---
# <a name="dmz-between-azure-and-your-on-premises-datacenter"></a>DMZ entre Azure et votre centre de données local

Cette architecture de référence montre un réseau sécurisé hybride qui étend un réseau local sur Azure. L’architecture implémente une DMZ, également appelée *réseau de périmètre*, entre le réseau local et un réseau virtuel Azure (VNet). La DMZ comprend des appliances virtuelles réseau (NVA) qui implémentent des fonctionnalités de sécurité telles que des pare-feu et l’inspection des paquets. Tout le trafic sortant du réseau virtuel est acheminé de force vers Internet via le réseau local afin qu’il puisse être audité.

[![0]][0] 

*Téléchargez un [fichier Visio][visio-download] de cette architecture.*

Cette architecture requiert une connexion à votre centre de données local, à l’aide d’une [passerelle VPN][ra-vpn] ou d’une connexion [ExpressRoute][ra-expressroute]. Utilisations courantes de cette architecture :

* Les applications hybrides où les charges de travail s’exécutent en partie en local et dans Azure.
* Une infrastructure qui nécessite un contrôle granulaire sur le trafic entrant d’un réseau virtuel Azure à partir d’un centre de données local.
* Les applications qui doivent auditer le trafic sortant. Il s’agit souvent d’une exigence réglementaire de nombreux systèmes commerciaux, qui peut permettre d’éviter la divulgation publique d’informations privées.

## <a name="architecture"></a>Architecture

L’architecture est constituée des composants suivants.

* **Réseau local**. Un réseau local privé implémenté dans une organisation.
* **Réseau virtuel Azure (VNet)**. Le réseau virtuel héberge l’application et les autres ressources s’exécutant dans Azure.
* **Passerelle**. La passerelle assure la connectivité entre les routeurs du réseau local et ceux du réseau virtuel.
* **Appliance virtuelle réseau (NVA)**. Appliance virtuelle réseau est un terme générique qui décrit une machine virtuelle effectuant des tâches comme autoriser ou refuser l’accès en tant que pare-feu, optimiser des opérations de réseau étendu (WAN) (y compris la compression réseau), personnaliser le routage ou d’autres fonctionnalités réseau.
* **Sous-réseaux de couche Données, de couche Business et de couche Web**. Sous-réseaux hébergeant les machines virtuelles et les services qui implémentent un exemple d’application à 3 couches s’exécutant dans le cloud. Consultez [Exécuter des machines virtuelles Windows pour une architecture multiniveau][ra-n-tier] pour plus d’informations.
* **Itinéraires définis par l’utilisateur (UDR)**. Les [itinéraires définis par l’utilisateur][udr-overview] définissent le flux du trafic IP au sein des réseaux virtuels Azure.

    > [!NOTE]
    > Selon les exigences de votre connexion VPN, vous pouvez configurer des itinéraires de protocole de passerelle frontière (BGP) au lieu d’utiliser des itinéraires définis par l’utilisateur pour implémenter les règles de transfert qui orientent le trafic par le biais du réseau local.
    > 
    > 

* **Sous-réseau de gestion.** Ce sous-réseau contient des machines virtuelles qui implémentent des fonctionnalités de gestion et de surveillance des composants s’exécutant dans le réseau virtuel.

## <a name="recommendations"></a>Recommandations

Les recommandations suivantes s’appliquent à la plupart des scénarios. Suivez ces recommandations, sauf si vous avez un besoin spécifique qui vous oblige à les ignorer. 

### <a name="access-control-recommendations"></a>Recommandations en matière de contrôle d’accès

Utilisez le [contrôle d’accès en fonction du rôle][rbac] pour gérer les ressources dans votre application. Envisagez de créer les [rôles personnalisés][rbac-custom-roles] suivants :

- Un rôle DevOps avec des autorisations pour gérer l’infrastructure de l’application, déployer les composants d’application et surveiller et redémarrer les machines virtuelles.  

- Un rôle d’administrateur informatique centralisé pour gérer et surveiller les ressources réseau.

- Un rôle d’administrateur informatique de sécurité pour gérer les ressources réseau sécurisées telles que les appliances virtuelles réseau. 

Les rôles d’administrateur informatique et DevOps ne doivent pas avoir accès aux ressources d’appliances virtuelles réseau. Cela doit être limité au rôle d’administrateur informatique de sécurité.

### <a name="resource-group-recommendations"></a>Recommandations en matière de groupes de ressources

Les ressources Azure, telles que les machines virtuelles, les réseaux virtuels et les équilibreurs de charge peuvent être facilement gérées en les regroupant dans des groupes de ressources. Assignez des rôles de contrôle d’accès en fonction du rôle à chaque groupe de ressources pour limiter l’accès.

Nous vous recommandons de créer les groupes de ressources suivants :

* Un groupe de ressources contenant le réseau virtuel (sauf les machines virtuelles), les groupes de sécurité réseau et les ressources de passerelle pour la connexion au réseau local. Assignez le rôle d’administrateur informatique centralisé à ce groupe de ressources.
* Un groupe de ressources contenant les machines virtuelles pour les appliances virtuelles réseau (y compris l’équilibreur de charge), le serveur de rebond et les autres machines virtuelles de gestion, ainsi que les itinéraires définis par l’utilisateur pour le sous-réseau de passerelle qui force tout le trafic via les appliances virtuelles réseau. Assignez le rôle d’administrateur informatique de sécurité à ce groupe de ressources.
* Des groupes de ressources distincts pour chaque couche Application qui contiennent l’équilibreur de charge et les machines virtuelles. Notez que ce groupe de ressources ne doit pas inclure les sous-réseaux de chaque couche. Assignez le rôle DevOps à ce groupe de ressources.

### <a name="virtual-network-gateway-recommendations"></a>Recommandations en matière de passerelle de réseau virtuel

Le trafic local est transmis au réseau virtuel via une passerelle de réseau virtuel. Nous vous recommandons une [passerelle VPN Azure][guidance-vpn-gateway] ou une [passerelle Azure ExpressRoute][guidance-expressroute].

### <a name="nva-recommendations"></a>Recommandations en matière d’appliances virtuelles réseau

Les appliances virtuelles réseau fournissent différents services pour gérer et surveiller le trafic réseau. La [Place de marché Microsoft Azure][azure-marketplace-nva] propose plusieurs appliances virtuelles réseau de fournisseurs tiers que vous pouvez utiliser. Si aucune de ces appliances virtuelles réseau ne répond à vos besoins, vous pouvez créer une appliance virtuelle réseau personnalisée à l’aide de machines virtuelles. 

Par exemple, le déploiement de solution pour cette architecture de référence implémente une appliance virtuelle réseau avec les fonctionnalités suivantes sur une machine virtuelle :

* Le trafic est acheminé à l’aide du [transfert IP][ip-forwarding] sur les interfaces réseau (cartes réseau) d’appliance virtuelle réseau.
* Le trafic a l’autorisation de passer par l’appliance virtuelle réseau uniquement si cela est nécessaire. Chaque machine virtuelle d’appliance virtuelle réseau dans l’architecture de référence est un routeur Linux simple. Le trafic entrant arrive sur l’interface réseau *eth0*, et le trafic sortant correspond aux règles définies par des scripts personnalisés distribués via l’interface réseau *eth1*.
* Les appliances virtuelles réseau peuvent uniquement être configurées à partir du sous-réseau de gestion. 
* Le trafic acheminé vers le sous-réseau de gestion ne passe pas par les appliances virtuelles réseau. Sinon, dans le cas où les appliances virtuelles réseau échouent, il n’y aura aucun itinéraire vers le sous-réseau de gestion pour les résoudre.  
* Les machines virtuelles pour l’appliance virtuelle réseau sont placées dans un [groupe à haute disponibilité][availability-set] derrière un équilibreur de charge. L’itinéraire défini par l’utilisateur dans le sous-réseau de passerelle dirige les requêtes d’appliance virtuelle réseau vers l’équilibreur de charge.

Incluez une appliance virtuelle réseau de couche 7 pour mettre fin aux connexions d’application au niveau de l’appliance virtuelle réseau et assurer la compatibilité les couches principales. Cela garantit une connectivité symétrique dans laquelle le trafic de réponse issu des couches principales est renvoyé via l’appliance virtuelle réseau.

Une autre option à prendre en compte consiste à connecter à plusieurs appliances virtuelles réseau à la suite, avec chaque appliance virtuelle réseau exécutant une tâche de sécurité spécialisée. Cela permet de gérer chaque fonction de sécurité par appliance virtuelle réseau. Par exemple, une appliance virtuelle réseau implémentant un pare-feu peut être placée en série avec une appliance virtuelle réseau exécutant des services d’identité. Néanmoins, cette facilité de gestion implique l’ajout de sauts de réseau supplémentaires qui peuvent augmenter la latence. Vous devez donc vous assurer que cela n’affecte pas les performances de votre application.


### <a name="nsg-recommendations"></a>Recommandations en matière de groupes de sécurité réseau

La passerelle VPN expose une adresse IP publique pour la connexion au réseau local. Nous vous recommandons de créer un groupe de sécurité réseau (NSG) pour le sous-réseau d’appliance virtuelle réseau entrant, avec des règles pour bloquer l’ensemble du trafic ne provenant pas du réseau local.

Nous vous recommandons également des groupes de sécurité réseau pour chaque sous-réseau afin de fournir un deuxième niveau de protection contre le trafic qui entre en contournant une appliance virtuelle réseau mal configurée ou désactivée. Par exemple, le sous-réseau de couche Web dans l’architecture de référence implémente un groupe de sécurité réseau avec une règle pour ignorer toutes les requêtes autres que celles reçues du réseau local (192.168.0.0/16) ou du réseau virtuel et une autre règle qui ignore toutes les requêtes non effectuées sur le port 80.

### <a name="internet-access-recommendations"></a>Recommandations en matière d’accès Internet

[Acheminez de force][azure-forced-tunneling] tout le trafic Internet sortant via votre réseau local à l’aide du tunnel VPN de site à site et acheminez-le vers Internet à l’aide de la traduction d’adresses réseau (NAT). Cela empêche la divulgation accidentelle d’informations confidentielles stockées dans votre couche Données et permet d’inspecter et d’auditer tout le trafic sortant.

> [!NOTE]
> Ne bloquez pas complètement le trafic Internet des couches Application, car cela empêchera ces couches d’utiliser les services PaaS Azure qui reposent sur les adresses IP publiques, tels que la journalisation des diagnostics, le téléchargement des extensions de machines virtuelles et d’autres fonctionnalités. Les diagnostics Azure exigent également que les composants puissent lire et écrire dans un compte de stockage Azure.
> 
> 

Vérifiez que le trafic Internet sortant est correctement acheminé de force. Si vous utilisez une connexion VPN avec le [service d’accès distant et de routage][routing-and-remote-access-service] sur un serveur local, utilisez un outil tel que [WireShark][wireshark]ou [Microsoft Message Analyzer](https://www.microsoft.com/download/details.aspx?id=44226).

### <a name="management-subnet-recommendations"></a>Recommandations en matière de sous-réseau de gestion

Le sous-réseau de gestion contient un serveur de rebond qui exécute les fonctionnalités de gestion et de surveillance. Limitez l’exécution de toutes les tâches de gestion sécurisées au serveur de rebond.
 
Ne créez pas d’adresse IP publique pour le serveur de rebond. Au lieu de cela, créez un itinéraire pour accéder au serveur de rebond via la passerelle entrante. Créez des règles de groupe de sécurité réseau de façon que le sous-réseau de gestion réponde uniquement aux requêtes de l’itinéraire autorisé.

## <a name="scalability-considerations"></a>Considérations relatives à l’extensibilité

L’architecture de référence utilise un équilibreur de charge pour diriger le trafic réseau local vers un pool d’appareils d’appliance virtuelle réseau qui acheminent le trafic. Les appliances virtuelles réseau sont placées dans un [groupe à haute disponibilité][availability-set]. Cette conception vous permet de surveiller le débit des appliances virtuelles réseau dans le temps et d’ajouter des appareils d’appliance virtuelle réseau pour répondre aux augmentations de charge.

La passerelle VPN de référence Standard prend en charge un débit de 100 Mbits/s maximum. La référence Hautes performances fournit jusqu’à 200 Mbits/s. Pour des bandes passantes plus élevées, envisagez d’effectuer une mise à niveau vers une passerelle ExpressRoute. ExpressRoute offre jusqu’à 10 Gbits/s de bande passante avec une latence inférieure à une celle d’une connexion VPN.

Pour plus d’informations sur l’extensibilité des passerelles Azure, consultez la section sur les considérations en matière d’extensibilité de [Implementing a hybrid network architecture with Azure and on-premises VPN][guidance-vpn-gateway-scalability] (Implémentation d’une architecture réseau hybride avec Azure et un VPN local) et [Implementing a hybrid network architecture with Azure ExpressRoute][guidance-expressroute-scalability] (Implémentation d’une architecture réseau hybride avec Azure ExpressRoute).

## <a name="availability-considerations"></a>Considérations relatives à la disponibilité

Comme mentionné, l’architecture de référence utilise un pool d’appareils d’appliance virtuelle réseau derrière un équilibreur de charge. L’équilibreur de charge utilise une sonde d’intégrité pour surveiller chaque appliance virtuelle réseau et supprime toutes les appliances virtuelles réseau inactives du pool.

Si vous utilisez Azure ExpressRoute pour fournir la connectivité entre le réseau local et le réseau virtuel, [configurez une passerelle VPN pour proposer un basculement][ra-vpn-failover] si la connexion ExpressRoute n’est plus disponible.

Pour des informations spécifiques sur le maintien de la disponibilité des connexions ExpressRoute et VPN, consultez les considérations en matière de disponibilité de [Implementing a hybrid network architecture with Azure and on-premises VPN][guidance-vpn-gateway-availability] (Implémentation d’une architecture réseau hybride avec Azure et un VPN local) et [Implementing a hybrid network architecture with Azure ExpressRoute][guidance-expressroute-availability] (Implémentation d’une architecture réseau hybride avec Azure ExpressRoute). 

## <a name="manageability-considerations"></a>Considérations relatives à la facilité de gestion

L’ensemble de la surveillance des ressources et des applications doit être effectuée par le serveur de rebond dans le sous-réseau de gestion. Selon les exigences de votre application, vous aurez peut-être besoin de ressources de surveillance supplémentaires dans le sous-réseau de gestion. Dans ce cas, ces ressources doivent être accessibles via le serveur de rebond.

Si la connectivité de la passerelle de votre réseau local vers Azure est interrompue, vous pouvez toujours atteindre le serveur de rebond en déployant une adresse IP publique, en l’ajoutant au serveur de rebond et en communiquant à distance à partir d’Internet.

Chaque sous-réseau de couche dans l’architecture de référence est protégé par les règles du groupe de sécurité réseau. Vous devrez peut-être créer une règle pour ouvrir le port 3389 pour l’accès au protocole RDP sur les machines virtuelles Windows ou le port 22 pour l’accès SSH sur les machines virtuelles Linux. D’autres outils de gestion et de surveillance peuvent nécessiter des règles pour ouvrir des ports supplémentaires.

Si vous utilisez ExpressRoute pour fournir la connectivité entre votre centre de données local et Azure, utilisez [Azure Connectivity Toolkit (AzureCT)][azurect] pour surveiller et résoudre les problèmes de connexion.

Vous pouvez trouver des informations supplémentaires spécifiques sur la surveillance et la gestion des connexions ExpressRoute et VPN dans les articles [Implementing a hybrid network architecture with Azure and on-premises VPN][guidance-vpn-gateway-manageability] (Implémentation d’une architecture réseau hybride avec Azure et un VPN local) et [Implementing a hybrid network architecture with Azure ExpressRoute][guidance-expressroute-manageability] (Implémentation d’une architecture réseau hybride avec Azure ExpressRoute).

## <a name="security-considerations"></a>Considérations relatives à la sécurité

Cette architecture de référence implémente plusieurs niveaux de sécurité.

### <a name="routing-all-on-premises-user-requests-through-the-nva"></a>Routage de toutes les requêtes des utilisateurs locales via l’appliance virtuelle réseau
L’itinéraire défini par l’utilisateur dans le sous-réseau de passerelle bloque toutes les requêtes des utilisateurs autres que celles reçues du site local. L’itinéraire défini par l’utilisateur transmet les requêtes autorisées aux appliances virtuelles réseau dans le sous-réseau de DMZ privé, et ces requêtes sont transmises à l’application si elles sont autorisées par les règles d’appliance virtuelle réseau. Vous pouvez ajouter d’autres itinéraires à l’itinéraire défini par l’utilisateur, mais veillez à ce qu’ils ne contournent pas par inadvertance les appliances virtuelles réseau ou bloquent le trafic destiné au sous-réseau de gestion.

L’équilibreur de charge devant les appliances virtuelles réseau agit également en tant qu’appareil de sécurité en ignorant le trafic sur les ports qui ne sont pas ouverts dans les règles d’équilibrage de charge. Les équilibreurs de charge dans l’architecture de référence écoutent uniquement les requêtes HTTP sur le port 80 et les requêtes HTTPS sur le port 443. Documentez les règles supplémentaires que vous ajoutez aux équilibreurs de charge et surveillez le trafic pour vous assurer qu’il n’y a aucun problème de sécurité.

### <a name="using-nsgs-to-blockpass-traffic-between-application-tiers"></a>Utilisation des groupes de sécurité réseau pour bloquer/transmettre le trafic entre les couches Application
Le trafic entre les couches est limité à l’aide des groupes de sécurité réseau. La couche Business bloque tout le trafic qui ne provient pas de la couche Web, et la couche Données bloque tout le trafic qui ne provient pas de la couche Business. Si vous avez besoin d’une exigence pour développer les règles du groupe de sécurité réseau afin d’autoriser un accès plus large à ces couches, évaluez ces exigences par rapport aux risques de sécurité. Chaque nouveau chemin d’accès entrant représente un risque d’altération d’application ou de divulgation de données volontaire ou accidentelle.

### <a name="devops-access"></a>Accès DevOps
Utilisez le [contrôle d’accès en fonction du rôle][rbac] pour restreindre les opérations que le rôle DevOps peut effectuer sur chaque couche. Lorsque vous accordez des autorisations, utilisez le [principe des privilèges minimum][security-principle-of-least-privilege]. Journalisez toutes les opérations d’administration et réalisez des audits réguliers pour vérifier qu’aucune modification de configuration n’est prévue.

## <a name="solution-deployment"></a>Déploiement de la solution

Un déploiement pour une architecture de référence implémentant ces recommandations est disponible sur [GitHub][github-folder]. L’architecture de référence peut être déployée en procédant comme suit :

1. Cliquez sur le bouton ci-dessous :<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-hybrid%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Une fois le lien ouvert dans le portail Azure, vous devez entrer des valeurs pour certains paramètres :   
   * Le nom du **groupe de ressources** est déjà défini dans le fichier de paramètres ; sélectionnez **Créer nouveau** et entrez `ra-private-dmz-rg` dans la zone de texte.
   * Sélectionnez la région à partir de la zone déroulante **Emplacement**.
   * Ne modifiez pas les zones de texte **Template Root Uri** (Uri racine de modèle) ou **Parameter Root Uri** (Uri racine de paramètre).
   * Passez en revue les termes et conditions, puis cochez la case **J’accepte les termes et conditions mentionnés ci-dessus**.
   * Cliquez sur le bouton **Acheter**.
3. Attendez la fin du déploiement.
4. Les fichiers de paramètres incluent un nom d’utilisateur administrateur et un mot de passe codés en dur pour toutes les machines virtuelles, et il est vivement recommandé de modifier immédiatement ces deux éléments. Pour chaque machine virtuelle du déploiement, sélectionnez-la dans le portail Azure, puis cliquez sur **Réinitialiser le mot de passe** dans le panneau **Support + dépannage**. Sélectionnez **Réinitialiser le mot de passe** dans la zone déroulante **Mode**, puis sélectionnez de nouveaux **Nom d’utilisateur** et **Mot de passe**. Cliquez sur le bouton **Mise à jour** pour enregistrer.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment implémenter une [DMZ située entre Azure et Internet](secure-vnet-dmz.md).
* Découvrez comment implémenter [une architecture réseau hybride hautement disponible][ra-vpn-failover].
* Pour plus d’informations sur la gestion de la sécurité réseau avec Azure, consultez [Services de cloud computing et sécurité réseau Microsoft][cloud-services-network-security].
* Pour plus d’informations sur la protection des ressources dans Azure, consultez [Prise en main de la sécurité de Microsoft Azure][getting-started-with-azure-security]. 
* Pour plus d’informations sur la gestion des problèmes de sécurité sur une connexion de passerelle Azure, consultez [Implementing a hybrid network architecture with Azure and on-premises VPN][guidance-vpn-gateway-security] (Implémentation d’une architecture réseau hybride avec Azure et un VPN local) et [Implementing a hybrid network architecture with Azure ExpressRoute][guidance-expressroute-security] (Implémentation d’une architecture réseau hybride avec Azure ExpressRoute).
  > 

<!-- links -->

[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azurect]: https://github.com/Azure/NetworkMonitoring/tree/master/AzureCT
[azure-forced-tunneling]: https://azure.microsoft.com/en-gb/documentation/articles/vpn-gateway-forced-tunneling-rm/
[azure-marketplace-nva]: https://azuremarketplace.microsoft.com/marketplace/apps/category/networking
[cloud-services-network-security]: https://azure.microsoft.com/documentation/articles/best-practices-network-security/
[getting-started-with-azure-security]: /azure/security/azure-security-getting-started
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-hybrid
[guidance-expressroute]: ../hybrid-networking/expressroute.md
[guidance-expressroute-availability]: ../hybrid-networking/expressroute.md#availability-considerations
[guidance-expressroute-manageability]: ../hybrid-networking/expressroute.md#manageability-considerations
[guidance-expressroute-security]: ../hybrid-networking/expressroute.md#security-considerations
[guidance-expressroute-scalability]: ../hybrid-networking/expressroute.md#scalability-considerations
[guidance-vpn-gateway]: ../hybrid-networking/vpn.md
[guidance-vpn-gateway-availability]: ../hybrid-networking/vpn.md#availability-considerations
[guidance-vpn-gateway-manageability]: ../hybrid-networking/vpn.md#manageability-considerations
[guidance-vpn-gateway-scalability]: ../hybrid-networking/vpn.md#scalability-considerations
[guidance-vpn-gateway-security]: ../hybrid-networking/vpn.md#security-considerations
[ip-forwarding]: /azure/virtual-network/virtual-networks-udr-overview#ip-forwarding
[ra-expressroute]: ../hybrid-networking/expressroute.md
[ra-n-tier]: ../virtual-machines-windows/n-tier.md
[ra-vpn]: ../hybrid-networking/vpn.md
[ra-vpn-failover]: ../hybrid-networking/expressroute-vpn-failover.md
[rbac]: /azure/active-directory/role-based-access-control-configure
[rbac-custom-roles]: /azure/active-directory/role-based-access-control-custom-roles
[routing-and-remote-access-service]: https://technet.microsoft.com/library/dd469790(v=ws.11).aspx
[security-principle-of-least-privilege]: https://msdn.microsoft.com/library/hdb58b2f(v=vs.110).aspx#Anchor_1
[udr-overview]: /azure/virtual-network/virtual-networks-udr-overview
[visio-download]: https://archcenter.blob.core.windows.net/cdn/dmz-reference-architectures.vsdx
[wireshark]: https://www.wireshark.org/
[0]: ./images/dmz-private.png "Architecture réseau hybride sécurisée"
