---
title: Connecter un réseau local à Azure à l’aide d’ExpressRoute
description: Comment implémenter une architecture réseau de site à site sécurisée qui s’étend sur un réseau virtuel Azure et un réseau local connecté à l’aide d’Azure ExpressRoute.
author: telmosampaio
ms.date: 10/22/2017
pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute-vpn-failover
pnp.series.prev: vpn
cardTitle: ExpressRoute
ms.openlocfilehash: 16711acb179c05152fc5ef8c7bf3eeb8d067a382
ms.sourcegitcommit: dbbf914757b03cdee7a274204f9579fa63d7eed2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/02/2018
ms.locfileid: "50916615"
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute"></a>Connecter un réseau local à Azure à l’aide d’ExpressRoute

Cette architecture de référence montre comment connecter un réseau local à des réseaux virtuels dans Azure, avec [Azure ExpressRoute][expressroute-introduction]. Les connexions ExpressRoute utilisent une connexion privée et dédiée via un fournisseur de connectivité tiers. La connexion privée étend votre réseau local à Azure. [**Déployez cette solution**.](#deploy-the-solution)

![[0]][0]

*Téléchargez un [fichier Visio][visio-download] de cette architecture.*

## <a name="architecture"></a>Architecture

L’architecture est constituée des composants suivants.

* **Réseau d’entreprise local**. Un réseau local privé qui s’exécute au sein d’une organisation.

* **Circuit ExpressRoute**. Un circuit de couche 2 ou 3 fourni par le fournisseur de connectivité et qui relie le réseau local à Azure via les routeurs de périphérie. Le circuit utilise l’infrastructure matérielle gérée par le fournisseur de connectivité.

* **Routeurs de périphérie locaux**. Les routeurs qui connectent le réseau local au circuit géré par le fournisseur. Selon l’approvisionnement de votre connexion, vous devrez peut-être fournir les adresses IP publiques utilisées par les routeurs.
* **Routeurs Microsoft Edge**. Avec deux routeurs, vous disposez d’une configuration hautement disponible en mode actif-actif. Ces routeurs permettent à un fournisseur de connectivité de connecter ses circuits directement à son centre de données. Selon l’approvisionnement de votre connexion, vous devrez peut-être fournir les adresses IP publiques utilisées par les routeurs.

* **Réseaux virtuels (VNet) Azure** Chaque réseau virtuel se trouve dans une région Azure unique et peut héberger plusieurs niveaux d’application. Les niveaux d’application peuvent être segmentés à l’aide de sous-réseaux dans chaque réseau virtuel.

* **Services publics Azure**. Les services Azure qui peuvent être utilisés dans une application hybride. Ces services sont également disponibles par Internet, mais en y accédant via un circuit ExpressRoute, vous profitez d’une latence faible et de performances plus prévisibles, étant donné que le trafic ne passe pas par Internet. Les connexions sont effectuées à l’aide de [l’homologation publique][expressroute-peering], avec des adresses qui appartiennent soit à votre organisation, soit à votre fournisseur de connectivité.

* **Services Office 365**. Les applications et services Office 365 publics fournis par Microsoft. Les connexions sont effectuées à l’aide de [l’homologation Microsoft][expressroute-peering], avec des adresses qui appartiennent soit à votre organisation, soit à votre fournisseur de connectivité. Vous pouvez également vous connecter directement à Microsoft CRM Online via l’homologation Microsoft.

* **Fournisseurs de connectivité** (non affiché). Les sociétés qui fournissent une connexion à l’aide d’une connectivité de couche 2 ou 3 entre votre centre de données et un centre de données Azure.

## <a name="recommendations"></a>Recommandations

Les recommandations suivantes s’appliquent à la plupart des scénarios. Suivez ces recommandations, sauf si vous avez un besoin spécifique qui vous oblige à les ignorer.

### <a name="connectivity-providers"></a>Fournisseurs de connectivité

Sélectionnez un fournisseur de connectivité ExpressRoute adapté selon votre localisation. Pour obtenir une liste des fournisseurs de connectivité disponibles pour votre localisation, utilisez la commande Azure PowerShell suivante :

```powershell
Get-AzureRmExpressRouteServiceProvider
```

Les fournisseurs de connectivité ExpressRoute connectent votre centre de données à Microsoft, comme suit :

* **Colocalisation avec un échange de cloud**. Si vous êtes colocalisé avec une installation dans le cadre d’un échange de cloud, vous pouvez commander des interconnexions virtuelles vers Azure via l’échange Ethernet du fournisseur de colocalisation. Les fournisseurs de colocalisation peuvent offrir des interconnexions de couche 2 ou des interconnexions de couche 3 gérées entre votre infrastructure dans l’installation de colocalisation et Azure.
* **Connexions Ethernet point à point**. Vous pouvez connecter vos centres de données/bureaux locaux à Azure via des liaisons Ethernet point à point. Les fournisseurs Ethernet point à point peuvent offrir des connexions de couche 2 ou des connexions de couche 3 gérées entre votre site et Azure.
* **Réseaux universels (IPVPN)**. Vous pouvez intégrer votre réseau étendu (WAN) avec Azure. Les fournisseurs d’IPVPN (en général un VPN MPLS) offrent une connectivité universelle entre vos succursales et vos centres de données. Azure peut être interconnecté à votre réseau étendu afin qu’il apparaisse comme n’importe quelle autre succursale. Les fournisseurs de réseaux étendus offrent généralement une connectivité de couche 3 gérée.

Pour plus d’informations sur les fournisseurs de connectivité, consultez l’article [Présentation d’ExpressRoute][expressroute-introduction].

### <a name="expressroute-circuit"></a>Circuit ExpressRoute

Vérifiez que votre organisation répond à la [configuration requise d’ExpressRoute][expressroute-prereqs] pour se connecter à Azure.

Si ce n’est pas déjà fait, ajoutez un sous-réseau nommé `GatewaySubnet` à votre réseau virtuel Azure et créez une passerelle de réseau virtuel ExpressRoute à l’aide du service de passerelle VPN Azure. Pour plus d’informations sur ce processus, consultez l’article [Workflows ExpressRoute d’approvisionnement du circuit et états du circuit][ExpressRoute-provisioning].

Créez un circuit ExpressRoute comme suit :

1. Exécutez la commande PowerShell suivante :
   
    ```powershell
    New-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>> -Location <<location>> -SkuTier <<sku-tier>> -SkuFamily <<sku-family>> -ServiceProviderName <<service-provider-name>> -PeeringLocation <<peering-location>> -BandwidthInMbps <<bandwidth-in-mbps>>
    ```
2. Envoyez le `ServiceKey` du nouveau circuit au fournisseur de services.

3. Attendez que le fournisseur approvisionne le circuit. Pour vérifier l’état d’approvisionnement d’un circuit, exécutez la commande PowerShell suivante :
   
    ```powershell
    Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    ```

    Le champ `Provisioning state` de la section `Service Provider` du résultat passera de `NotProvisioned` à `Provisioned` une fois le circuit prêt.

    > [!NOTE]
    > Si vous utilisez une connexion de couche 3, le fournisseur doit configurer et gérer le routage à votre place. Vous fournissez les informations nécessaires pour permettre au fournisseur d’implémenter les itinéraires appropriés.
    > 
    > 

4. Si vous utilisez une connexion de couche 2 :

    1. Réservez deux sous-réseaux /30 constitués d’adresses IP publiques valides pour chaque type d’homologation que vous souhaitez implémenter. Ces sous-réseaux /30 seront utilisés pour fournir des adresses IP aux routeurs utilisés pour le circuit. Si vous implémentez une homologation privée, publique ou Microsoft, vous aurez besoin de 6 sous-réseaux /30 avec des adresses IP publiques valides.     

    2. Configurez l’acheminement pour le circuit ExpressRoute. Exécutez les commandes PowerShell suivantes pour chaque type d’homologation à configurer (privée, publique et Microsoft). Pour plus d’informations, consultez l’article [Créer et modifier l’acheminement d’un circuit ExpressRoute à l’aide de PowerShell][configure-expressroute-routing].
   
        ```powershell
        Set-AzureRmExpressRouteCircuitPeeringConfig -Name <<peering-name>> -Circuit <<circuit-name>> -PeeringType <<peering-type>> -PeerASN <<peer-asn>> -PrimaryPeerAddressPrefix <<primary-peer-address-prefix>> -SecondaryPeerAddressPrefix <<secondary-peer-address-prefix>> -VlanId <<vlan-id>>

        Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit <<circuit-name>>
        ```

    3. Réservez un autre pool d’adresses IP publiques valides à utiliser pour la traduction d’adresse réseau (NAT) pour l’homologation publique et Microsoft. Il est recommandé d’avoir un pool différent pour chaque homologation. Indiquez le pool à votre fournisseur de connectivité, afin qu’il puisse configurer des annonces (BGP) pour ces plages.

5. Exécutez les commandes PowerShell suivantes pour lier votre ou vos réseaux virtuels privés au circuit ExpressRoute. Pour plus d’informations, consultez l’article [Liaison d’un réseau virtuel à un circuit ExpressRoute][link-vnet-to-expressroute].

    ```powershell
    $circuit = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $gw = Get-AzureRmVirtualNetworkGateway -Name <<gateway-name>> -ResourceGroupName <<resource-group>>
    New-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> -ResourceGroupName <<resource-group>> -Location <<location> -VirtualNetworkGateway1 $gw -PeerId $circuit.Id -ConnectionType ExpressRoute
    ```

Vous pouvez connecter plusieurs réseaux virtuels situés dans des régions différentes à un même circuit ExpressRoute, tant que tous les réseaux virtuels et le circuit ExpressRoute sont situés dans la même région géopolitique.

### <a name="troubleshooting"></a>Résolution de problèmes 

Si un circuit ExpressRoute qui fonctionnait ne parvient plus à se connecter, en l’absence de toute modification de configuration en local ou sur votre réseau privé virtuel, vous devrez peut-être contacter le fournisseur de connectivité et travailler avec lui pour résoudre le problème. Pour vérifier que le circuit ExpressRoute a été approvisionné, utilisez les commandes Powershell suivantes :

```powershell
Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

Le résultat de cette commande montre plusieurs propriétés de votre circuit, y compris `ProvisioningState`, `CircuitProvisioningState` et `ServiceProviderProvisioningState` comme vous pouvez le voir ci-dessous.

```
ProvisioningState                : Succeeded
Sku                              : {
                                     "Name": "Standard_MeteredData",
                                     "Tier": "Standard",
                                     "Family": "MeteredData"
                                   }
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : NotProvisioned
```

Si `ProvisioningState` n’est pas défini sur `Succeeded` une fois que vous avez tenté de créer un nouveau circuit, supprimez le circuit en utilisant la commande ci-dessous et essayez de le créer à nouveau.

```powershell
Remove-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

Si votre fournisseur avait déjà approvisionné le circuit et que `ProvisioningState` est défini sur `Failed`, ou que `CircuitProvisioningState` n’est pas `Enabled`, contactez votre fournisseur pour obtenir de l’aide.

## <a name="scalability-considerations"></a>Considérations relatives à l’extensibilité

Les circuits ExpressRoute fournissent un chemin à bande passante élevée entre les réseaux. En règle générale, plus la bande passante est élevée, plus c’est cher. 

ExpressRoute propose deux [plans de tarification][expressroute-pricing] à ses clients, un plan de données limité et plan de données illimité. Les frais varient en fonction de la bande passante du circuit. La bande passante disponible varie d’un fournisseur à l’autre. Utilisez l’applet de commande `Get-AzureRmExpressRouteServiceProvider` pour voir les fournisseurs disponibles dans votre région et les bandes passantes qu’ils proposent.
 
Un même circuit ExpressRoute peut prendre en charge un certain nombre d’homologations et de liens de réseau virtuel. Pour plus d’informations, consultez l’article sur les [limites d’ExpressRoute](/azure/azure-subscription-service-limits).

Le module complémentaire ExpressRoute Premium propose certaines fonctionnalités payantes supplémentaires :

* Augmentation des limites d'itinéraire pour l'homologation publique et privée. 
* Augmentation du nombre de liens de réseau virtuel par circuit ExpressRoute. 
* Connectivité globale des services.

Pour plus de détails, consultez la page sur la [tarification d’ExpressRoute][expressroute-pricing]. 

La conception des circuits ExpressRoute vous permet d’augmenter temporairement jusqu’à deux fois la limite de la bande passante obtenue sur le réseau sans frais supplémentaires, grâce à des liens redondants. Toutefois, tous les fournisseurs de connectivité ne prennent pas en charge cette fonction. Avant toute chose, vérifiez donc que votre fournisseur de connectivité propose cette fonctionnalité.

Bien que certains fournisseurs vous permettent de modifier votre bande passante, assurez-vous de choisir une bande passante initiale qui dépasse vos besoins et vous donne la possibilité d’évoluer. Si vous avez besoin d’augmenter la bande passante à l’avenir, vous avez deux choix :

- Augmentez la bande passante. Si possible, évitez cette option, car tous les fournisseurs ne vous permettent pas d’augmenter la bande passante de façon dynamique. Si vous avez vraiment besoin d’augmenter votre bande passante, contactez votre fournisseur pour vérifier qu’il prend en charge la modification des propriétés de bande passante ExpressRoute via les commandes Powershell. Si c’est le cas, exécutez les commandes ci-dessous.

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $ckt.ServiceProviderProperties.BandwidthInMbps = <<bandwidth-in-mbps>>
    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    Vous pouvez augmenter votre bande passante sans perte de connectivité. La rétrogradation de la bande passante entraîne une interruption de connectivité, car vous devez supprimer le circuit et le recréer avec la nouvelle configuration.

- Changez de plan de tarification et/ou passez à la version Premium. Pour ce faire, exécutez les commandes suivantes : La propriété `Sku.Tier` peut avoir la valeur `Standard` ou `Premium` ; la propriété `Sku.Name` peut avoir la valeur `MeteredData` ou `UnlimitedData`.

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>

    $ckt.Sku.Tier = "Premium"
    $ckt.Sku.Family = "MeteredData"
    $ckt.Sku.Name = "Premium_MeteredData"

    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    > [!IMPORTANT]
    > Assurez-vous que la propriété `Sku.Name` correspond aux propriétés `Sku.Tier` et `Sku.Family`. Si vous modifiez la famille et la couche, mais pas le nom, votre connexion sera désactivée.
    > 
    > 

    Vous pouvez mettre à niveau la référence SKU sans interruption, mais vous ne pouvez pas passer du plan de tarification illimité au plan limité. Lorsque vous rétrogradez la référence SKU, votre consommation de bande passante doit rester dans la limite par défaut de la référence SKU standard.

## <a name="availability-considerations"></a>Considérations relatives à la disponibilité

ExpressRoute ne prend pas en charge les protocoles de redondance de routeur tels que le protocole HSRP (Hot Standby Router Protocol) et le protocole VRRP (Virtual Router Redundancy Protocol) pour garantir la haute disponibilité. Il utilise une paire de sessions BGP redondantes pour chaque homologation. Pour faciliter les connexions hautement disponibles à votre réseau, Azure vous fournit deux ports redondants sur deux routeurs (compris avec Microsoft Edge) dans une configuration actif-actif.

Par défaut, les sessions BGP utilisent un délai d’inactivité de 60 secondes. Si une session expire trois fois (180 secondes au total), le routeur est marqué comme non disponible et tout le trafic est redirigé vers le routeur restant. Ce délai d’attente de 180 secondes peut être trop long pour certaines applications critiques. Dans ce cas, vous pouvez modifier vos paramètres de délai d’attente BGP sur le routeur local et choisir une valeur inférieure.

Vous pouvez configurer la haute disponibilité pour votre connexion Azure de différentes façons, selon votre type de fournisseur et le nombre de circuits ExpressRoute et de connexions de passerelle de réseau virtuel que vous souhaitez configurer. Voici un résumé des options disponibles :

* Si vous utilisez une connexion de couche 2, déployez des routeurs redondants sur votre réseau local dans une configuration actif-actif. Connectez le circuit principal à un routeur et le circuit secondaire à l’autre. Vous bénéficierez ainsi d’une connexion hautement disponible aux deux extrémités de la connexion. Cela peut s’avérer nécessaire si vous avez besoin du contrat de niveau de service d’ExpressRoute. Pour plus de détails, consultez l’article [SLA pour Azure ExpressRoute][sla-for-expressroute].

    Le diagramme suivant illustre une configuration avec des routeurs redondants locaux connectés au circuit principal et au circuit secondaire. Chaque circuit gère le trafic pour une homologation publique et une homologation privée (chaque homologation est conçue comme une paire d’espaces d’adressage /30, comme décrit dans la section précédente).

    ![[1]][1]

* Si vous utilisez une connexion de couche 3, vérifiez qu’elle fournit des sessions BGP redondantes qui gèrent la disponibilité à votre place.

* Connectez le réseau virtuel à plusieurs circuits ExpressRoute fournis par différents fournisseurs de services. Cette stratégie vous fournit des fonctionnalités de récupération d’urgence et de haute disponibilité supplémentaires.

* Configurez un réseau VPN de site à site comme un chemin d’accès de basculement pour ExpressRoute. Pour en savoir plus sur cette option, consultez l’article [Connecter un réseau local à Azure à l’aide d’ExpressRoute avec basculement VPN][highly-available-network-architecture].
 Cette option ne s’applique qu’à l’homologation privée. Pour les services Azure et Office 365, Internet est le seul chemin de basculement disponible. 

## <a name="manageability-considerations"></a>Considérations relatives à la facilité de gestion

Vous pouvez utiliser le [kit de connectivité Azure (AzureCT)][azurect] pour contrôler la connectivité entre votre centre de données sur site et Azure. 

## <a name="security-considerations"></a>Considérations relatives à la sécurité

Vous pouvez configurer les options de sécurité pour votre connexion Azure de différentes façons selon vos besoins en matière de conformité et vos préoccupations en matière de sécurité. 

ExpressRoute fonctionne en couche 3. Les menaces liées à la couche Application peuvent être évitées à l’aide d’une appliance de sécurité réseau qui limite le trafic aux ressources légitimes. En outre, les connexions ExpressRoute qui utilisent l’homologation publique ne peuvent être initiées que sur site. Ainsi, les services non autorisés ne peuvent pas accéder aux données locales à partir d’Internet, ni les compromettre.

Pour optimiser la sécurité, ajoutez des appliances de sécurité réseau entre le réseau local et les routeurs de périphérie du fournisseur. Ainsi, vous pourrez limiter l’afflux de trafic non autorisé à partir du réseau virtuel :

![[2]][2]

Pour des raisons de conformité ou en cas d’audit, il peut être nécessaire d’interdire l’accès direct à Internet à partir des composants qui s’exécutent dans le réseau virtuel et d’implémenter le [tunneling forcé][forced-tuneling]. Dans ce cas, le trafic Internet doit être redirigé vers un proxy exécuté en local, où il peut être audité. Le proxy peut être configuré pour bloquer les flux de trafic sortant non autorisés et filtrer le trafic entrant potentiellement malveillant.

![[3]][3]

Pour optimiser la sécurité, n’utilisez pas d’adresse IP publique pour vos machines virtuelles. Par contre, vous pouvez utiliser des groupes de sécurité réseau pour vous assurer que ces machines virtuelles ne sont pas accessibles publiquement. Les machines virtuelles doivent être disponibles uniquement via l’adresse IP interne. Vous pouvez rendre ces adresses accessibles via le réseau ExpressRoute et ainsi permettre au personnel DevOps local d’en effectuer la configuration ou la maintenance.

Si vous devez exposer sur un réseau externe des points de terminaison de gestion pour les machines virtuelles, utilisez des groupes de sécurité réseau ou des listes de contrôle d’accès pour restreindre la visibilité de ces ports à une liste verte d’adresses IP ou de réseaux.

> [!NOTE]
> Par défaut, les machines virtuelles Azure déployées via le portail Azure incluent une adresse IP publique qui fournit un accès en connexion.  
> 
> 


## <a name="deploy-the-solution"></a>Déployer la solution

**Configuration requise.** Vous devez disposer d’une infrastructure locale existante déjà configurée avec une appliance réseau appropriée.

Pour déployer la solution, procédez comme suit :

1. Cliquez sur le bouton ci-dessous :<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Attendez que le lien s’ouvre dans le portail Azure, puis procédez comme suit :
   * Le nom du **groupe de ressources** est déjà défini dans le fichier de paramètres ; sélectionnez **Créer nouveau** et entrez `ra-hybrid-er-rg` dans la zone de texte.
   * Sélectionnez la région à partir de la zone déroulante **Emplacement**.
   * Ne modifiez pas les zones de texte **Template Root Uri** (Uri racine de modèle) ou **Parameter Root Uri** (Uri racine de paramètre).
   * Passez en revue les termes et conditions, puis cochez la case **J’accepte les termes et conditions mentionnés ci-dessus**.
   * Cliquez sur le bouton **Acheter**.
3. Attendez la fin du déploiement.
4. Cliquez sur le bouton ci-dessous :<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
5. Attendez que le lien s’ouvre dans le portail Azure, puis procédez comme suit :
   * Sélectionnez **Utiliser l’existant** dans la section **Groupe de ressources** et saisissez `ra-hybrid-er-rg` dans la zone de texte.
   * Sélectionnez la région à partir de la zone déroulante **Emplacement**.
   * Ne modifiez pas les zones de texte **Template Root Uri** (Uri racine de modèle) ou **Parameter Root Uri** (Uri racine de paramètre).
   * Passez en revue les termes et conditions, puis cochez la case **J’accepte les termes et conditions mentionnés ci-dessus**.
   * Cliquez sur le bouton **Acheter**.
6. Attendez la fin du déploiement.


<!-- links -->
[forced-tuneling]: ../dmz/secure-vnet-hybrid.md
[highly-available-network-architecture]: ./expressroute-vpn-failover.md

[expressroute-technical-overview]: /azure/expressroute/expressroute-introduction
[expressroute-prereqs]: /azure/expressroute/expressroute-prerequisites
[configure-expressroute-routing]: /azure/expressroute/expressroute-howto-routing-arm
[sla-for-expressroute]: https://azure.microsoft.com/support/legal/sla/expressroute
[link-vnet-to-expressroute]: /azure/expressroute/expressroute-howto-linkvnet-arm
[ExpressRoute-provisioning]: /azure/expressroute/expressroute-workflows
[expressroute-introduction]: /azure/expressroute/expressroute-introduction
[expressroute-peering]: /azure/expressroute/expressroute-circuit-peerings
[expressroute-pricing]: https://azure.microsoft.com/pricing/details/expressroute/
[expressroute-limits]: /azure/azure-subscription-service-limits#networking-limits
[azurect]: https://github.com/Azure/NetworkMonitoring/tree/master/AzureCT
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[er-circuit-parameters]: https://github.com/mspnp/reference-architectures/tree/master/hybrid-networking/expressroute/parameters/expressRouteCircuit.parameters.json
[azure-powershell-download]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[0]: ./images/expressroute.png "Architecture réseau hybride avec Azure ExpressRoute"
[1]: ../_images/guidance-hybrid-network-expressroute/figure2.png "Utilisation de routeurs redondants avec les circuits ExpressRoute principaux et secondaires"
[2]: ../_images/guidance-hybrid-network-expressroute/figure3.png "Ajout d’appareils de sécurité au réseau local"
[3]: ../_images/guidance-hybrid-network-expressroute/figure4.png "Utilisation du tunneling forcé pour auditer le trafic Internet"
[4]: ../_images/guidance-hybrid-network-expressroute/figure5.png "Localisation du champ ServiceKey d’un circuit ExpressRoute"  