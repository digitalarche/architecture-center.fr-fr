---
title: Connecter un réseau local à Azure
titleSuffix: Azure Reference Architectures
description: Implémentez une architecture réseau de site à site sécurisée et hautement disponible qui s’étend sur un réseau virtuel Azure et un réseau local connecté à l’aide d’ExpressRoute avec basculement de passerelle VPN.
author: telmosampaio
ms.date: 10/22/2017
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18, networking
ms.openlocfilehash: fba07823d49aa43fdb67652f99a26bd7df3a57c3
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54488288"
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute-with-vpn-failover"></a>Connecter un réseau local à Azure à l’aide d’ExpressRoute avec basculement VPN

Cette architecture de référence montre comment connecter un réseau local à un réseau virtuel Azure (VNet) à l’aide d’ExpressRoute, avec un réseau privé virtuel (VPN) de site à site en tant que connexion de basculement. Le trafic circule entre le réseau local et réseau virtuel Azure via une connexion ExpressRoute. En cas de perte de connectivité au niveau du circuit ExpressRoute, le trafic est acheminé via un tunnel VPN IPSec. [**Déployez cette solution**](#deploy-the-solution).

Notez que si le circuit ExpressRoute n’est pas disponible, l’itinéraire VPN gérera uniquement les connexions d’homologation privées. Les connexions d’homologation publique et Microsoft transiteront via Internet.

![Architecture de référence pour une architecture réseau hybride hautement disponible utilisant ExpressRoute et une passerelle VPN](./images/expressroute-vpn-failover.png)

*Téléchargez un [fichier Visio][visio-download] de cette architecture.*

## <a name="architecture"></a>Architecture

L’architecture est constituée des composants suivants.

- **Réseau local**. Un réseau local privé s’exécutant au sein d’une organisation.

- **Appliance VPN**. Périphérique ou service qui assure la connectivité externe au réseau local. L’appliance VPN peut être un périphérique matériel ou une solution logicielle telle que le service RRAS (Routing and Remote Access Service) dans Windows Server 2012. Pour obtenir la liste des périphériques VPN pris en charge et des informations sur la configuration de certains périphériques VPN pour la connexion à Azure, consultez [À propos des périphériques VPN pour les connexions de la passerelle VPN de site à site][vpn-appliance].

- **Circuit ExpressRoute**. Un circuit de couche 2 ou 3 fourni par le fournisseur de connectivité et qui relie le réseau local à Azure via les routeurs de périphérie. Le circuit utilise l’infrastructure matérielle gérée par le fournisseur de connectivité.

- **Passerelle de réseau virtuel ExpressRoute**. La passerelle de réseau virtuel ExpressRoute permet au réseau virtuel de se connecter au circuit ExpressRoute utilisé pour la connectivité avec votre réseau local.

- **Passerelle de réseau virtuel VPN**. La passerelle de réseau virtuel VPN permet au réseau virtuel de se connecter à l’appliance VPN sur le réseau local. La passerelle de réseau virtuel VPN est configurée pour accepter les requêtes provenant du réseau local uniquement via l’appliance VPN. Pour plus d’informations, consultez [Connecter un réseau local à Microsoft Azure Virtual Network][connect-to-an-Azure-vnet].

- **Connexion VPN**. La connexion a des propriétés qui spécifient le type de connexion (IPSec) et la clé partagée avec l’appliance VPN locale pour chiffrer le trafic.

- **Réseau virtuel Azure**. Chaque réseau virtuel se trouve dans une région Azure unique et peut héberger plusieurs niveaux d’application. Les niveaux d’application peuvent être segmentés à l’aide de sous-réseaux dans chaque réseau virtuel.

- **Sous-réseau de passerelle**. Les passerelles de réseau virtuel sont conservées dans le même sous-réseau.

- **Application cloud**. L’application hébergée dans Azure. Elle peut inclure plusieurs niveaux, avec plusieurs sous-réseaux connectés à l’aide d’équilibreurs de charge Azure. Pour plus d’informations sur l’infrastructure d’application, consultez [Running Windows VM workloads][windows-vm-ra] (Exécution de charges de travail de machine virtuelle Windows) et [Exécution de charges de travail de machine virtuelle Linux][linux-vm-ra].

## <a name="recommendations"></a>Recommandations

Les recommandations suivantes s’appliquent à la plupart des scénarios. Suivez ces recommandations, sauf si vous avez un besoin spécifique qui vous oblige à les ignorer.

### <a name="vnet-and-gatewaysubnet"></a>Réseau virtuel et sous-réseau GatewaySubnet

Créez la passerelle de réseau virtuel ExpressRoute et la passerelle de réseau virtuel VPN sur le même réseau virtuel. Cela signifie qu’ils doivent partager le même sous-réseau nommé *GatewaySubnet*.

Si le réseau virtuel contient déjà un sous-réseau nommé *GatewaySubnet*, vérifiez qu’il possède un espace d’adressage défini sur /27 ou plus. Si le sous-réseau existant est trop petit, utilisez la commande PowerShell suivante pour supprimer le sous-réseau :

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Remove-AzureRmVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet
```

Si le réseau virtuel ne contient aucun sous-réseau nommé **GatewaySubnet**, créez-en un à l’aide de la commande Powershell suivante :

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Add-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet -AddressPrefix "10.200.255.224/27"
$vnet = Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
```

### <a name="vpn-and-expressroute-gateways"></a>Passerelles VPN et ExpressRoute

Vérifiez que votre organisation répond à la [configuration requise d’ExpressRoute] [ expressroute-prereq] pour se connecter à Azure.

Si vous avez déjà une passerelle de réseau virtuel VPN dans votre réseau virtuel Azure, utilisez la commande PowerShell suivante pour la supprimer :

```powershell
Remove-AzureRmVirtualNetworkGateway -Name <yourgatewayname> -ResourceGroupName <yourresourcegroup>
```

Suivez les instructions de l’article [Implémentation d’une architecture réseau hybride avec Azure ExpressRoute][implementing-expressroute] pour établir votre connexion ExpressRoute.

Suivez les instructions de l’article [Implémentation d’une architecture réseau hybride avec Azure et un VPN local][implementing-vpn] pour établir votre connexion de passerelle de réseau virtuel VPN.

Après avoir établi les connexions de passerelle de réseau virtuel, testez l’environnement comme suit :

1. Vérifiez que vous pouvez vous connecter au réseau virtuel Azure à partir de votre réseau local.
2. Contactez votre fournisseur pour arrêter la connectivité ExpressRoute à des fins de test.
3. Vérifiez que vous pouvez toujours vous connecter au réseau virtuel Azure à partir de votre réseau local à l’aide de la connexion de passerelle de réseau virtuel VPN.
4. Contactez votre fournisseur pour rétablir la connectivité ExpressRoute.

## <a name="considerations"></a>Considérations

Pour connaître les considérations relatives à ExpressRoute, consultez l’article [Implémentation d’une architecture réseau hybride avec Azure ExpressRoute][guidance-expressroute].

Pour connaître les considérations relatives au VPN de site à site, consultez l’article [Implémentation d’une architecture réseau hybride avec Azure et un VPN local][guidance-vpn].

Pour connaître les considérations de sécurité générales relatives à Azure, consultez l’article [Services de cloud computing et sécurité réseau Microsoft][best-practices-security].

## <a name="deploy-the-solution"></a>Déployer la solution

**Conditions préalables**. Vous devez disposer d’une infrastructure locale existante déjà configurée avec une appliance réseau appropriée.

Pour déployer la solution, procédez comme suit :

<!-- markdownlint-disable MD033 -->

1. Cliquez sur le bouton ci-dessous :<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>

2. Attendez que le lien s’ouvre dans le portail Azure, puis procédez comme suit :
   - Le nom du **groupe de ressources** est déjà défini dans le fichier de paramètres ; sélectionnez **Créer nouveau** et entrez `ra-hybrid-vpn-er-rg` dans la zone de texte.
   - Sélectionnez la région à partir de la zone déroulante **Emplacement**.
   - Ne modifiez pas les zones de texte **Template Root Uri** (Uri racine de modèle) ou **Parameter Root Uri** (Uri racine de paramètre).
   - Passez en revue les termes et conditions, puis cochez la case **J’accepte les termes et conditions mentionnés ci-dessus**.
   - Cliquez sur le bouton **Acheter**.

3. Attendez la fin du déploiement.

4. Cliquez sur le bouton ci-dessous :<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>

5. Attendez que le lien s’ouvre dans le portail Azure, appuyez sur Entrée, puis procédez comme suit :
   - Sélectionnez **Utiliser l’existant** dans la section **Groupe de ressources** et saisissez `ra-hybrid-vpn-er-rg` dans la zone de texte.
   - Sélectionnez la région à partir de la zone déroulante **Emplacement**.
   - Ne modifiez pas les zones de texte **Template Root Uri** (Uri racine de modèle) ou **Parameter Root Uri** (Uri racine de paramètre).
   - Passez en revue les termes et conditions, puis cochez la case **J’accepte les termes et conditions mentionnés ci-dessus**.
   - Cliquez sur le bouton **Acheter**.

<!-- markdownlint-enable MD033 -->

<!-- links -->

[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[expressroute-prereq]: /azure/expressroute/expressroute-prerequisites
[implementing-expressroute]: ./expressroute.md
[implementing-vpn]: ./vpn.md
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[best-practices-security]: /azure/best-practices-network-security
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
