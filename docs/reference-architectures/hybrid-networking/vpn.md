---
title: Connecter un réseau local à Azure à l’aide d’un VPN
description: Comment implémenter une architecture réseau de site à site sécurisée qui s’étend sur un réseau virtuel Azure et un réseau local connecté à l’aide d’un VPN.
author: RohitSharma-pnp
ms.date: 10/22/2018
pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute
pnp.series.prev: ./index
cardTitle: VPN
ms.openlocfilehash: 6e705c40663eff421e79067f916a1ebad6e72822
ms.sourcegitcommit: dbbf914757b03cdee7a274204f9579fa63d7eed2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/02/2018
ms.locfileid: "50916547"
---
# <a name="connect-an-on-premises-network-to-azure-using-a-vpn-gateway"></a>Connecter un réseau local à Azure à l’aide d’une passerelle VPN

Cette architecture de référence montre comment étendre un réseau local à Azure à l’aide d’un réseau privé virtuel (VPN) site à site. Le trafic circule entre le réseau local et un réseau virtuel Azure (VNet) via un tunnel VPN IPSec. [**Déployez cette solution**.](#deploy-the-solution)

![[0]][0]

*Téléchargez un [fichier Visio][visio-download] de cette architecture.*

## <a name="architecture"></a>Architecture 

L’architecture est constituée des composants suivants.

* **Réseau local**. Un réseau local privé s’exécutant au sein d’une organisation.

* **Appliance VPN**. Périphérique ou service qui assure la connectivité externe au réseau local. L’appliance VPN peut être un périphérique matériel ou une solution logicielle telle que le service RRAS (Routing and Remote Access Service) dans Windows Server 2012. Pour obtenir la liste des appareils VPN pris en charge et des informations sur la configuration pour les connecter à une passerelle VPN Azure, consultez les instructions pour l’appareil sélectionné dans l’article [À propos des périphériques VPN pour les connexions de la passerelle VPN de site à site][vpn-appliance].

* **Réseau virtuel**. L’application cloud et les composants de la passerelle VPN Azure résident dans le même [réseau virtuel][azure-virtual-network].

* **Passerelle VPN Azure**. Le service de [passerelle VPN][azure-vpn-gateway] vous permet de connecter le réseau virtuel au réseau local via une appliance VPN. Pour plus d’informations, consultez [Connecter un réseau local à Microsoft Azure Virtual Network][connect-to-an-Azure-vnet]. La passerelle VPN inclut les éléments suivants :
  
  * **Passerelle de réseau virtuel**. Ressource qui fournit une appliance VPN virtuelle pour le réseau virtuel. Elle achemine le trafic du réseau local vers le réseau virtuel.
  * **Passerelle de réseau local**. Abstraction de l’appliance VPN locale. Le trafic réseau allant de l’application cloud au réseau local est acheminé via cette passerelle.
  * **Connexion**. La connexion a des propriétés qui spécifient le type de connexion (IPSec) et la clé partagée avec l’appliance VPN locale pour chiffrer le trafic.
  * **Sous-réseau de passerelle**. La passerelle de réseau virtuel est conservée dans son propre sous-réseau, qui est soumis à des exigences diverses, décrites dans la section Recommandations ci-dessous.

* **Application cloud**. L’application hébergée dans Azure. Elle peut inclure plusieurs niveaux, avec plusieurs sous-réseaux connectés à l’aide d’équilibreurs de charge Azure. Pour plus d’informations sur l’infrastructure d’application, consultez [Running Windows VM workloads][windows-vm-ra] (Exécution de charges de travail de machine virtuelle Windows) et [Exécution de charges de travail de machine virtuelle Linux][linux-vm-ra].

* **Équilibreur de charge interne**. Le trafic réseau de la passerelle VPN est acheminé vers l’application cloud via un équilibreur de charge interne. L’équilibreur de charge se trouve dans le sous-réseau frontal de l’application.

## <a name="recommendations"></a>Recommandations

Les recommandations suivantes s’appliquent à la plupart des scénarios. Suivez ces recommandations, sauf si vous avez un besoin spécifique qui vous oblige à les ignorer.

### <a name="vnet-and-gateway-subnet"></a>Réseau virtuel et sous-réseau de passerelle

Créez un réseau virtuel Azure avec un espace d’adressage suffisant pour toutes les ressources requises. Assurez-vous que l’espace d’adressage du réseau virtuel puisse évoluer au cas où vous auriez besoin de machines virtuelles supplémentaires à l’avenir. L’espace d’adressage du réseau virtuel ne doit pas chevaucher le réseau local. Par exemple, le diagramme ci-dessus utilise l’espace d’adressage 10.20.0.0/16 pour le réseau virtuel.

Créez un sous-réseau nommé *GatewaySubnet*, avec une plage d’adresses de /27. La passerelle de réseau virtuel requiert ce sous-réseau. En allouant 32 adresses à ce sous-réseau, vous devriez éviter les limitations de taille de passerelle. En outre, évitez de placer ce sous-réseau au milieu de l’espace d’adressage. La meilleure pratique consiste à définir l’espace d’adressage du sous-réseau de passerelle à l’extrémité supérieure de l’espace d’adressage du réseau virtuel. L’exemple du diagramme utilise 10.20.255.224/27.  Voici une procédure rapide pour calculer le [CIDR] :

1. Définissez les bits variables dans l’espace d’adressage du réseau virtuel sur 1, jusqu'aux bits utilisés par le sous-réseau de passerelle, puis les bits restants sur 0.
2. Convertissez les bits obtenus en nombre décimal et exprimez celui-ci sous la forme d’un espace d’adressage dont la longueur de préfixe correspond à la taille du sous-réseau de passerelle.

Par exemple, pour un réseau virtuel avec une plage d’adresses IP de 10.20.0.0/16, l’étape 1 ci-dessus donne le résultat 10.20.0b11111111.0b11100000.  La conversion en nombre décimal et l’expression sous la forme d’un espace d’adressage génère 10.20.255.224/27. 

> [!WARNING]
> Ne déployez pas de machines virtuelles pour le sous-réseau de passerelle. N’affectez pas non plus de groupe de sécurité réseau à ce sous-réseau, car la passerelle cesserait de fonctionner.
> 
> 

### <a name="virtual-network-gateway"></a>Passerelle de réseau virtuel

Allouez une adresse IP publique à la passerelle de réseau virtuel.

Créez la passerelle de réseau virtuel dans le sous-réseau de passerelle et affectez-lui l’adresse IP publique nouvellement allouée. Utilisez le type de passerelle qui correspond le mieux à vos besoins et qui est activé par votre appliance VPN :

- Créez une [passerelle basée sur des stratégies][policy-based-routing] si vous avez besoin contrôler avec précision la façon dont les requêtes sont acheminées en fonction de critères de stratégie tels que les préfixes d’adresse. Les passerelles basées sur des stratégies utilisent le routage statique et fonctionnent uniquement avec les connexions de site à site.

- Créez une [passerelle basée sur des itinéraires][route-based-routing] si vous vous connectez au réseau local via RRAS, prenez en charge les connexions entre plusieurs régions ou sites, ou implémentez des connexions de réseau virtuel à réseau virtuel (y compris des itinéraires qui traversent plusieurs réseaux virtuels). Les passerelles basées sur des itinéraires utilisent le routage dynamique pour diriger le trafic entre les réseaux. Elles tolèrent mieux les incidents dans le chemin d’accès réseau que les itinéraires statiques, car elles peuvent essayer d’autres itinéraires. Les passerelles basées sur des itinéraires peuvent également réduire la charge de gestion, car les itinéraires ne doivent pas nécessairement être mis à jour manuellement en cas de changement des adresses réseau.

Pour obtenir la liste des appliances VPN prises en charge, voir [À propos des périphériques VPN pour les connexions de la passerelle VPN de site à site][vpn-appliances].

> [!NOTE]
> Si vous souhaitez changer de type de passerelle une fois celle-ci créée, il vous faudra la supprimer et en recréer une autre.
> 
> 

Sélectionnez la référence SKU de passerelle VPN Azure qui correspond le mieux à vos besoins en débit. Pour plus d’informations, consultez [SKU de passerelle][azure-gateway-skus]

> [!NOTE]
> La référence SKU de base n’est pas compatible avec Azure ExpressRoute. Vous pouvez [modifier la référence SKU][changing-SKUs] une fois la passerelle créée.
> 
> 

Le coût facturé est calculé en fonction de la durée pendant laquelle une passerelle est configurée et disponible. Voir la [tarification de la passerelle VPN][azure-gateway-charges].

Créez des règles de routage pour le sous-réseau de passerelle qui dirigent le trafic entrant de l’application de la passerelle vers l’équilibreur de charge interne, au lieu d’autoriser les requêtes à passer directement jusqu’aux machines virtuelles d’application.

### <a name="on-premises-network-connection"></a>Connexion au réseau local

Créer une passerelle de réseau local. Spécifiez l’adresse IP publique de l’appliance VPN locale et l’espace d’adressage du réseau local. Notez que l’appliance VPN locale doit avoir une adresse IP publique accessible par la passerelle de réseau local dans la passerelle VPN Azure. Le périphérique VPN ne peut pas se trouver derrière un périphérique de traduction d’adresses réseau (NAT).

Créez une connexion de site à site pour la passerelle de réseau virtuel et la passerelle de réseau local. Sélectionnez le type de connexion de site à site (IPSec) et spécifiez la clé partagée. Le chiffrement de site à site avec la passerelle VPN Azure est basé sur le protocole IPSec, à l’aide de clés prépartagées pour l’authentification. Vous spécifiez la clé lorsque vous créez la passerelle VPN Azure. Vous devez configurer l’appliance VPN exécutée en local avec la même clé. Les autres mécanismes d’authentification ne sont pas pris en charge actuellement.

Vérifiez que l’infrastructure de routage locale est configurée pour transférer les requêtes destinées aux adresses de réseau virtuel Azure au périphérique VPN.

Ouvrez les ports requis par l’application cloud dans le réseau local.

Testez la connexion pour vérifier les points suivants :

* L’appliance VPN locale achemine correctement le trafic vers l’application cloud via la passerelle VPN Azure.
* Le réseau virtuel réachemine correctement le trafic vers le réseau local.
* Le trafic interdit dans les deux directions est correctement bloqué.

## <a name="scalability-considerations"></a>Considérations relatives à l’extensibilité

Vous pouvez obtenir une évolutivité verticale limitée en passant des références SKU de passerelle VPN basique ou standard à la référence SKU de VPN hautes performances.

Pour les réseaux virtuels qui attendent un gros volume de trafic VPN, envisagez de distribuer les différentes charges de travail dans des réseaux virtuels distincts plus petits et de configurer une passerelle VPN pour chacun d’entre eux.

Vous pouvez partitionner le réseau virtuel horizontalement ou verticalement. Pour un partitionnement horizontal, déplacez certaines instances de machine virtuelle de chaque couche vers des sous-réseaux du nouveau réseau virtuel. Ainsi, chaque réseau virtuel a la même structure et les mêmes fonctionnalités. Pour un partitionnement vertical, réorganisez chaque couche pour diviser les fonctionnalités en différentes zones logiques (par exemple, la gestion des commandes, la facturation, la gestion des comptes client, etc.). Chaque zone fonctionnelle peut alors être placée dans son propre réseau virtuel.

La réplication d’un contrôleur de domaine Active Directory local dans le réseau virtuel et l’implémentation d’un DNS dans le réseau virtuel contribuent à réduire la part du trafic relative à la sécurité et à l’administration qui passe du site local au cloud. Pour plus d’informations, voir [Extension d’Active Directory Domain Services (AD DS) à Azure][adds-extend-domain].

## <a name="availability-considerations"></a>Considérations relatives à la disponibilité

Si vous devez vous assurer que le réseau local reste disponible pour la passerelle VPN Azure, implémentez un cluster de basculement pour la passerelle VPN locale.

Si votre organisation possède plusieurs sites locaux, créez des [connexions multisites][vpn-gateway-multi-site] vers un ou plusieurs réseaux virtuels Azure. Cette approche requiert un routage dynamique (basé sur des itinéraires). Veillez donc à ce que la passerelle VPN locale prenne en charge cette fonctionnalité.

Pour plus d’informations sur les contrats de niveau de service, voir le [contrat de niveau de service pour la passerelle VPN][sla-for-vpn-gateway]. 

## <a name="manageability-considerations"></a>Considérations relatives à la facilité de gestion

Surveillez les informations de diagnostic provenant des appliances VPN locales. Ce processus dépend des fonctionnalités fournies par l’appliance VPN. Par exemple, si vous utilisez le service de routage et d’accès distant sur Windows Server 2012, [journalisation RRAS][rras-logging].

Utilisez les [diagnostics de la passerelle VPN Azure][gateway-diagnostic-logs] pour capturer des informations sur les problèmes de connectivité. Ces journaux contiennent des informations telles que la source et les destinations des requêtes de connexion, le protocole utilisé et la méthode d’établissement de la connexion (ou la raison de l’échec).

Analysez les journaux des opérations de la passerelle VPN Azure à l’aide des journaux d’audit disponibles dans le portail Azure. Des journaux distincts sont disponibles pour la passerelle de réseau local, la passerelle de réseau Azure et la connexion. Ces informations permettent de suivre toutes les modifications apportées à la passerelle et peuvent être utiles si une passerelle cesse de fonctionner pour une raison quelconque.

![[2]][2]

Contrôlez la connectivité et suivez les événements d’échec de connectivité. Vous pouvez utiliser un package de contrôle comme [Nagios][nagios] pour capturer ces informations et les consigner dans des rapports.

## <a name="security-considerations"></a>Considérations relatives à la sécurité

Générez une clé partagée différente pour chaque passerelle VPN. Utilisez une clé partagée forte qui résiste aux attaques en force brute.

> [!NOTE]
> Actuellement, vous ne pouvez pas utiliser Azure Key Vault afin de prépartager des clés pour la passerelle VPN Azure.
> 
> 

Assurez-vous que l’appliance VPN locale utilise une méthode de chiffrement [compatible avec la passerelle VPN Azure][vpn-appliance-ipsec]. Pour le routage basé sur des stratégies, la passerelle VPN Azure prend en charge les algorithmes de chiffrement AES256, AES128 et 3DES. Les passerelles basées sur des itinéraires prennent en charge AES256 et 3DES.

Si votre appliance VPN locale se trouve sur un réseau de périmètre (DMZ) qui a un pare-feu entre le réseau de périmètre et Internet, vous devrez peut-être configurer des [règles de pare-feu supplémentaires][additional-firewall-rules] pour permettre la connexion VPN de site à site.

Si l’application du réseau virtuel envoie des données à Internet, envisagez de [mettre en œuvre le tunneling forcé][forced-tunneling] pour acheminer tout le trafic Internet via le réseau local. Cette approche vous permet d’auditer les requêtes sortantes effectuées par l’application à partir de l’infrastructure locale.

> [!NOTE]
> Le tunneling forcé peut influer sur la connectivité aux services Azure (le service de stockage, par exemple) et le gestionnaire de licences de Windows.
> 
> 


## <a name="troubleshooting"></a>Résolution de problèmes 

Pour obtenir des informations générales sur la résolution des problèmes courants liés au VPN, voir la page correspondante ([Troubleshooting common VPN related errors][troubleshooting-vpn-errors]).

Les recommandations suivantes sont utiles pour déterminer si votre appliance VPN locale fonctionne correctement.

- **Vérifiez les fichiers journaux générés par l’appliance VPN pour les erreurs ou les échecs.**

    Cela vous aidera à déterminer si l’appliance VPN fonctionne correctement. L’emplacement de ces informations varie en fonction de votre application. Par exemple, avec RRAS sur Windows Server 2012, vous pouvez utiliser la commande PowerShell suivante pour afficher des informations sur les événements d’erreur du service RRAS :

    ```PowerShell
    Get-EventLog -LogName System -EntryType Error -Source RemoteAccess | Format-List -Property *
    ```

    La propriété *Message* de chaque entrée fournit une description de l’erreur. Voici quelques exemples courants :

        - Inability to connect, possibly due to an incorrect IP address specified for the Azure VPN gateway in the RRAS VPN network interface configuration.

        ```
        EventID            : 20111
        MachineName        : on-prem-vm
        Data               : {41, 3, 0, 0}
        Index              : 14231
        Category           : (0)
        CategoryNumber     : 0
        EntryType          : Error
        Message            : RoutingDomainID- {00000000-0000-0000-0000-000000000000}: A demand dial connection to the remote
                             interface AzureGateway on port VPN2-4 was successfully initiated but failed to complete
                             successfully because of the  following error: The network connection between your computer and
                             the VPN server could not be established because the remote server is not responding. This could
                             be because one of the network devices (for example, firewalls, NAT, routers, and so on) between your computer
                             and the remote server is not configured to allow VPN connections. Please contact your
                             Administrator or your service provider to determine which device may be causing the problem.
        Source             : RemoteAccess
        ReplacementStrings : {{00000000-0000-0000-0000-000000000000}, AzureGateway, VPN2-4, The network connection between
                             your computer and the VPN server could not be established because the remote server is not
                             responding. This could be because one of the network devices (for example, firewalls, NAT, routers, and so on)
                             between your computer and the remote server is not configured to allow VPN connections. Please
                             contact your Administrator or your service provider to determine which device may be causing the
                             problem.}
        InstanceId         : 20111
        TimeGenerated      : 3/18/2016 1:26:02 PM
        TimeWritten        : 3/18/2016 1:26:02 PM
        UserName           :
        Site               :
        Container          :
        ```

        - The wrong shared key being specified in the RRAS VPN network interface configuration.

        ```
        EventID            : 20111
        MachineName        : on-prem-vm
        Data               : {233, 53, 0, 0}
        Index              : 14245
        Category           : (0)
        CategoryNumber     : 0
        EntryType          : Error
        Message            : RoutingDomainID- {00000000-0000-0000-0000-000000000000}: A demand dial connection to the remote
                             interface AzureGateway on port VPN2-4 was successfully initiated but failed to complete
                             successfully because of the  following error: Internet key exchange (IKE) authentication credentials are unacceptable.

        Source             : RemoteAccess
        ReplacementStrings : {{00000000-0000-0000-0000-000000000000}, AzureGateway, VPN2-4, IKE authentication credentials are
                             unacceptable.
                             }
        InstanceId         : 20111
        TimeGenerated      : 3/18/2016 1:34:22 PM
        TimeWritten        : 3/18/2016 1:34:22 PM
        UserName           :
        Site               :
        Container          :
        ```

    Vous pouvez également obtenir des informations de journal des événements sur les tentatives de connexion via le service RRAS à l’aide de la commande PowerShell suivante : 

    ```
    Get-EventLog -LogName Application -Source RasClient | Format-List -Property *
    ```

    En cas d’échec de connexion, ce fichier journal contient des erreurs semblables à ce qui suit :

    ```
    EventID            : 20227
    MachineName        : on-prem-vm
    Data               : {}
    Index              : 4203
    Category           : (0)
    CategoryNumber     : 0
    EntryType          : Error
    Message            : CoId={B4000371-A67F-452F-AA4C-3125AA9CFC78}: The user SYSTEM dialed a connection named
                         AzureGateway that has failed. The error code returned on failure is 809.
    Source             : RasClient
    ReplacementStrings : {{B4000371-A67F-452F-AA4C-3125AA9CFC78}, SYSTEM, AzureGateway, 809}
    InstanceId         : 20227
    TimeGenerated      : 3/18/2016 1:29:21 PM
    TimeWritten        : 3/18/2016 1:29:21 PM
    UserName           :
    Site               :
    Container          :
    ```

- **Vérifiez la connectivité et le routage sur la passerelle VPN.**

    L’appliance VPN n’achemine peut-être pas correctement le trafic via la passerelle VPN Azure. Utilisez un outil tel que [PsPing][psping] pour vérifier la connectivité et le routage sur la passerelle VPN. Par exemple, pour tester la connectivité d’un ordinateur local à un serveur web situé sur le réseau virtuel, exécutez la commande suivante (en remplaçant `<<web-server-address>>` par l’adresse du serveur web) :

    ```
    PsPing -t <<web-server-address>>:80
    ```

    Si l’ordinateur local peut acheminer le trafic vers le serveur web, vous devriez obtenir une sortie de ce type :

    ```
    D:\PSTools>psping -t 10.20.0.5:80

    PsPing v2.01 - PsPing - ping, latency, bandwidth measurement utility
    Copyright (C) 2012-2014 Mark Russinovich
    Sysinternals - www.sysinternals.com

    TCP connect to 10.20.0.5:80:
    Infinite iterations (warmup 1) connecting test:
    Connecting to 10.20.0.5:80 (warmup): 6.21ms
    Connecting to 10.20.0.5:80: 3.79ms
    Connecting to 10.20.0.5:80: 3.44ms
    Connecting to 10.20.0.5:80: 4.81ms

      Sent = 3, Received = 3, Lost = 0 (0% loss),
      Minimum = 3.44ms, Maximum = 4.81ms, Average = 4.01ms
    ```

    Si l’ordinateur local ne peut pas communiquer avec la destination spécifiée, vous verrez des messages de ce type :

    ```
    D:\PSTools>psping -t 10.20.1.6:80

    PsPing v2.01 - PsPing - ping, latency, bandwidth measurement utility
    Copyright (C) 2012-2014 Mark Russinovich
    Sysinternals - www.sysinternals.com

    TCP connect to 10.20.1.6:80:
    Infinite iterations (warmup 1) connecting test:
    Connecting to 10.20.1.6:80 (warmup): This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80:
      Sent = 3, Received = 0, Lost = 3 (100% loss),
      Minimum = 0.00ms, Maximum = 0.00ms, Average = 0.00ms
    ```

- **Vérifiez que le pare-feu local autorise le passage du trafic VPN et que les ports appropriés sont ouverts.**

- **Vérifiez que l’appliance VPN locale utilise une méthode de chiffrement [compatible avec la passerelle VPN Azure][vpn-appliance].** Pour le routage basé sur des stratégies, la passerelle VPN Azure prend en charge les algorithmes de chiffrement AES256, AES128 et 3DES. Les passerelles basées sur des itinéraires prennent en charge AES256 et 3DES.

Les recommandations suivantes sont utiles pour déterminer s’il existe un problème avec la passerelle VPN Azure :

- **Examinez les [journaux de diagnostic de la passerelle VPN Azure][gateway-diagnostic-logs] à la recherche de problèmes éventuels.**

- **Vérifiez que la passerelle VPN Azure et l’appliance VPN locale sont configurées avec la même clé d’authentification partagée.**

    Vous pouvez afficher la clé partagée stockée par la passerelle VPN Azure à l’aide de la commande CLI Azure suivante :

    ```
    azure network vpn-connection shared-key show <<resource-group>> <<vpn-connection-name>>
    ```

    Utilisez la commande appropriée pour votre appliance VPN locale afin d’afficher la clé partagée configurée pour cette application.

    Vérifiez que le sous-réseau *GatewaySubnet* contenant la passerelle VPN Azure n’est pas associé à un groupe de sécurité réseau.

    Vous pouvez afficher les détails du sous-réseau à l’aide de la commande CLI Azure suivante :

    ```
    azure network vnet subnet show -g <<resource-group>> -e <<vnet-name>> -n GatewaySubnet
    ```

    Vérifiez qu’il n’existe aucun champ de données nommé *Network Security Group id*. L’exemple suivant montre les résultats pour une instance du sous-réseau *GatewaySubnet* à laquelle est affectée un groupe de sécurité réseau (*VPN-Gateway-Group*). Cela peut empêcher la passerelle de fonctionner correctement si les règles sont définies pour ce groupe de sécurité réseau.

    ```
    C:\>azure network vnet subnet show -g profx-prod-rg -e profx-vnet -n GatewaySubnet
        info:    Executing command network vnet subnet show
        + Looking up virtual network "profx-vnet"
        + Looking up the subnet "GatewaySubnet"
        data:    Id                              : /subscriptions/########-####-####-####-############/resourceGroups/profx-prod-rg/providers/Microsoft.Network/virtualNetworks/profx-vnet/subnets/GatewaySubnet
        data:    Name                            : GatewaySubnet
        data:    Provisioning state              : Succeeded
        data:    Address prefix                  : 10.20.3.0/27
        data:    Network Security Group id       : /subscriptions/########-####-####-####-############/resourceGroups/profx-prod-rg/providers/Microsoft.Network/networkSecurityGroups/VPN-Gateway-Group
        info:    network vnet subnet show command OK
    ```

- **Vérifiez que les machines virtuelles du réseau virtuel Azure sont configurés pour autoriser le trafic provenant de l’extérieur du réseau virtuel.**

    Vérifiez toutes les règles du groupe de sécurité réseau associées aux sous-réseaux contenant ces machines virtuelles. Vous pouvez afficher toutes les règles du groupe de sécurité réseau à l’aide de la commande CLI Azure suivante :

    ```
    azure network nsg show -g <<resource-group>> -n <<nsg-name>>
    ```

- **Vérifiez que la passerelle VPN Azure est connectée.**

    Vous pouvez utiliser la commande Azure PowerShell suivante pour vérifier l’état actuel de la connexion VPN Azure. Le paramètre `<<connection-name>>` est le nom de la connexion VPN Azure qui relie la passerelle du réseau virtuel et la passerelle locale.

    ```
    Get-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> - ResourceGroupName <<resource-group>>
    ```

    Les extraits de code suivants montrent la sortie générée si la passerelle est connectée (premier exemple) et déconnectée (second exemple) :

    ```
    PS C:\> Get-AzureRmVirtualNetworkGatewayConnection -Name profx-gateway-connection -ResourceGroupName profx-prod-rg

    AuthorizationKey           :
    VirtualNetworkGateway1     : Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
    VirtualNetworkGateway2     :
    LocalNetworkGateway2       : Microsoft.Azure.Commands.Network.Models.PSLocalNetworkGateway
    Peer                       :
    ConnectionType             : IPsec
    RoutingWeight              : 0
    SharedKey                  : ####################################
    ConnectionStatus           : Connected
    EgressBytesTransferred     : 55254803
    IngressBytesTransferred    : 32227221
    ProvisioningState          : Succeeded
    ...
    ```

    ```
    PS C:\> Get-AzureRmVirtualNetworkGatewayConnection -Name profx-gateway-connection2 -ResourceGroupName profx-prod-rg

    AuthorizationKey           :
    VirtualNetworkGateway1     : Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
    VirtualNetworkGateway2     :
    LocalNetworkGateway2       : Microsoft.Azure.Commands.Network.Models.PSLocalNetworkGateway
    Peer                       :
    ConnectionType             : IPsec
    RoutingWeight              : 0
    SharedKey                  : ####################################
    ConnectionStatus           : NotConnected
    EgressBytesTransferred     : 0
    IngressBytesTransferred    : 0
    ProvisioningState          : Succeeded
    ...
    ```

Les recommandations suivantes sont utiles pour déterminer s’il existe un problème avec la configuration de machine virtuelle hôte, l’utilisation de la bande passante réseau ou les performances des applications :

- **Vérifiez que le pare-feu du système d’exploitation invité s’exécutant sur des machines virtuelles Azure dans le sous-réseau est correctement configuré pour permettre le trafic autorisé provenant des plages d’adresses IP locales.**

- **Vérifiez que le volume de trafic n’est pas proche de la limite de la bande passante disponible pour la passerelle VPN Azure.**

    La façon de vérifier cela dépend de l’appliance VPN exécutée localement. Par exemple, avec RRAS sur Windows Server 2012, vous pouvez utiliser l’Analyseur de performances pour suivre le volume de données reçues et transmises sur la connexion VPN. À l’aide de l’objet *RAS Total*, sélectionnez les compteurs d’*octets reçus par seconde* et d’*octets transmis par seconde* :

    ![[3]][3]

    Vous devez comparer les résultats à la bande passante disponible pour la passerelle VPN (100 Mbits/s pour les références SKU basiques et standard, et 200 Mbits/s pour la référence SKU hautes performances) :

    ![[4]][4]

- **Vérifiez que vous avez déployé le nombre approprié de machines virtuelles, avec la taille correcte, pour votre charge applicative.**

    Déterminez si toutes les machines virtuelles du réseau virtuel Azure s’exécutent lentement. Si c’est le cas, elles peuvent être surchargées, c’est-à-dire pas assez nombreuses pour gérer la charge, ou les équilibreurs de charge ne sont pas configurés correctement. Pour le déterminer, [capturez et analysez les informations de diagnostic][azure-vm-diagnostics]. Vous pouvez examiner les résultats à l’aide du portail Azure, mais de nombreux outils tiers sont également disponibles pour obtenir des informations détaillées sur les données de performances.

- **Vérifiez que l’application utilise efficacement les ressources cloud.**

    Servez-vous du code d’application en cours d’exécution sur chaque machine virtuelle afin de déterminer si les applications font un usage optimal des ressources. Vous pouvez utiliser des outils tels qu’[Application Insights][application-insights].

## <a name="deploy-the-solution"></a>Déployer la solution


**Configuration requise.** Vous devez disposer d’une infrastructure locale existante déjà configurée avec une appliance réseau appropriée.

Pour déployer la solution, procédez comme suit :

1. Cliquez sur le bouton ci-dessous :<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fvpn%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Attendez que le lien s’ouvre dans le portail Azure, puis procédez comme suit : 
   * Le nom du **groupe de ressources** est déjà défini dans le fichier de paramètres ; sélectionnez **Créer nouveau** et entrez `ra-hybrid-vpn-rg` dans la zone de texte.
   * Sélectionnez la région à partir de la zone déroulante **Emplacement**.
   * Ne modifiez pas les zones de texte **Template Root Uri** (Uri racine de modèle) ou **Parameter Root Uri** (Uri racine de paramètre).
   * Passez en revue les termes et conditions, puis cochez la case **J’accepte les termes et conditions mentionnés ci-dessus**.
   * Cliquez sur le bouton **Acheter**.
3. Attendez la fin du déploiement.



<!-- links -->

[adds-extend-domain]: ../identity/adds-extend-domain.md
[expressroute]: ../hybrid-networking/expressroute.md
[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md

[naming conventions]: /azure/guidance/guidance-naming-conventions

[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[arm-templates]: /azure/resource-group-authoring-templates
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-portal]: /azure/azure-portal/resource-group-portal
[azure-powershell]: /azure/powershell-azure-resource-manager
[azure-virtual-network]: /azure/virtual-network/virtual-networks-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: https://azure.microsoft.com/services/vpn-gateway/
[azure-gateway-charges]: https://azure.microsoft.com/pricing/details/vpn-gateway/
[azure-gateway-skus]: /azure/vpn-gateway/vpn-gateway-about-vpngateways#gwsku
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[vpn-gateway-multi-site]: /azure/vpn-gateway/vpn-gateway-multi-site
[policy-based-routing]: https://en.wikipedia.org/wiki/Policy-based_routing
[route-based-routing]: https://en.wikipedia.org/wiki/Static_routing
[network-security-group]: /azure/virtual-network/virtual-networks-nsg
[sla-for-vpn-gateway]: https://azure.microsoft.com/support/legal/sla/vpn-gateway/v1_2/
[additional-firewall-rules]: https://technet.microsoft.com/library/dn786406.aspx#firewall
[nagios]: https://www.nagios.org/
[azure-vpn-gateway-diagnostics]: https://blogs.technet.microsoft.com/keithmayer/2014/12/18/diagnose-azure-virtual-network-vpn-connectivity-issues-with-powershell/
[ping]: https://technet.microsoft.com/library/ff961503.aspx
[tracert]: https://technet.microsoft.com/library/ff961507.aspx
[psping]: https://technet.microsoft.com/sysinternals/jj729731.aspx
[nmap]: https://nmap.org
[changing-SKUs]: https://azure.microsoft.com/blog/azure-virtual-network-gateway-improvements/
[gateway-diagnostic-logs]: https://blogs.technet.microsoft.com/keithmayer/2016/10/12/step-by-step-capturing-azure-resource-manager-arm-vnet-gateway-diagnostic-logs/
[troubleshooting-vpn-errors]: https://blogs.technet.microsoft.com/rrasblog/2009/08/12/troubleshooting-common-vpn-related-errors/
[rras-logging]: https://www.petri.com/enable-diagnostic-logging-in-windows-server-2012-r2-routing-and-remote-access
[create-on-prem-network]: https://technet.microsoft.com/library/dn786406.aspx#routing
[azure-vm-diagnostics]: https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/
[application-insights]: /azure/application-insights/app-insights-overview-usage
[forced-tunneling]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-forced-tunneling/
[vpn-appliances]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[vpn-appliance-ipsec]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices#ipsec-parameters
[virtualNetworkGateway-parameters]: https://github.com/mspnp/hybrid-networking/vpn/parameters/virtualNetworkGateway.parameters.json
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[CIDR]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[0]: ./images/vpn.png "Réseau hybride incluant les infrastructures Azure et locales"
[2]: ../_images/guidance-hybrid-network-vpn/audit-logs.png "Journaux d’audit dans le portail Azure"
[3]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-counters.png "Compteurs de performances pour surveiller le trafic du réseau VPN"
[4]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-graph.png "Exemple de graphique de performances du réseau VPN"
