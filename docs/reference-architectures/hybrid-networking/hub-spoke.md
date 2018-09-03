---
title: Implémentation d’une topologie de réseau hub-and-spoke dans Azure
description: Comment implémenter une topologie de réseau hub-and-spoke dans Azure.
author: telmosampaio
ms.date: 04/09/2018
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: abe9d6a58f3deeab388c20471c5559d63ef2f245
ms.sourcegitcommit: c4106b58ad08f490e170e461009a4693578294ea
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/10/2018
ms.locfileid: "43016060"
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a>Implémenter une topologie de réseau hub-and-spoke dans Azure

Cette architecture de référence montre comment implémenter une topologie hub-and-spoke dans Azure. Le *hub* est un réseau virtuel (VNet) dans Azure qui centralise la connectivité à votre réseau local. Les *membres spokes* sont des réseaux virtuels appairés avec le hub et qui peuvent être utilisés pour isoler les charges de travail. Le trafic circule entre le centre de données local et le hub via une connexion de passerelle ExpressRoute ou VPN.  [**Déployez cette solution**](#deploy-the-solution).

![[0]][0]

*Téléchargez un [fichier Visio][visio-download] de cette architecture.*


Cette topologie présente les avantages suivants :

* **Réduction les coûts** en centralisant les services qui peuvent être partagés par plusieurs charges de travail, comme les appliances virtuelles réseau et les serveurs DNS.
* **Surmonter les limites des abonnements** en appairant les réseaux virtuels de différents abonnements avec le hub central.
* **Séparation des préoccupations** entre le service informatique central (SecOps, InfraOps) et les charges de travail (DevOps).

Utilisations courantes de cette architecture :

* Charges de travail déployées sur des environnements différents, tels que les environnements de développement, de test et de production, qui requièrent des services partagés tels que DNS, IDS, NTP ou AD DS. Les services partagés sont placés dans le réseau virtuel hub, tandis que chaque environnement est déployé sur un membre spoke pour garantir l’isolation.
* Charges de travail ne nécessitant pas de connectivité entre elles, mais nécessitant un accès aux services partagés.
* Entreprises nécessitant un contrôle centralisé des aspects de la sécurité, tel qu’un pare-feu dans le hub en tant que zone DMZ et une gestion séparée des charges de travail dans chaque membre spoke.

## <a name="architecture"></a>Architecture

L’architecture est constituée des composants suivants.

* **Réseau local**. Un réseau local privé qui s’exécute au sein d’une organisation.

* **Périphérique VPN**. Périphérique ou service qui assure la connectivité externe au réseau local. Le périphérique VPN peut être un périphérique matériel ou une solution logicielle telle que le service RRAS (Routing and Remote Access Service) dans Windows Server 2012. Pour obtenir la liste des périphériques VPN pris en charge et des informations sur la configuration de certains périphériques VPN pour la connexion à Azure, consultez [À propos des périphériques VPN pour les connexions de la passerelle VPN de site à site][vpn-appliance].

* **Passerelle de réseau virtuel VPN ou passerelle ExpressRoute**. La passerelle de réseau virtuel permet au réseau virtuel de se connecter au périphérique VPN, ou circuit ExpressRoute, utilisé pour la connectivité avec votre réseau local. Pour plus d’informations, consultez [Connecter un réseau local à Microsoft Azure Virtual Network][connect-to-an-Azure-vnet].

> [!NOTE]
> Les scripts de déploiement pour cette architecture de référence utilisent une passerelle VPN pour la connectivité et un réseau virtuel dans Azure pour simuler votre réseau local.

* **Réseau virtuel hub**. Réseau virtuel Azure utilisé comme hub dans la topologie hub-and-spoke. Le hub est le point central de la connectivité à votre réseau local et un emplacement pour héberger les services que peuvent utiliser les différentes charges de travail hébergées dans les réseaux virtuels spokes.

* **Sous-réseau de passerelle**. Les passerelles de réseau virtuel sont conservées dans le même sous-réseau.

* **Réseaux virtuels spokes**. Un ou plusieurs réseaux virtuels Azure qui sont utilisés comme membres spokes dans la topologie hub-and-spoke. Les membres spokes peuvent servir à isoler les charges de travail dans leurs propres réseaux virtuels, qui sont alors gérées séparément des autres membres spokes. Chaque charge de travail peut inclure plusieurs niveaux, avec plusieurs sous-réseaux connectés à l’aide d’équilibreurs de charge Azure. Pour plus d’informations sur l’infrastructure d’application, consultez [Running Windows VM workloads][windows-vm-ra] (Exécution de charges de travail de machine virtuelle Windows) et [Exécution de charges de travail de machine virtuelle Linux][linux-vm-ra].

* **Appairage de réseaux virtuels**. Deux réseaux virtuels dans la même région Azure peuvent être connectés à l’aide d’une [connexion d’appairage][vnet-peering]. Les connexions d’appairage sont des connexions non transitives et à faible latence entre des réseaux virtuels. Une fois appairés, les réseaux virtuels échangent le trafic à l’aide de la dorsale principale d’Azure, sans avoir besoin d’un routeur. Dans une topologie de réseau hub-and-spoke, vous utilisez l’appairage de réseaux virtuels pour connecter le hub à chaque membre spoke.

> [!NOTE]
> Cet article couvre uniquement les déploiements [Resource Manager](/azure/azure-resource-manager/resource-group-overview), mais vous pouvez également connecter un réseau virtuel classique à un réseau virtuel Resource Manager dans un même abonnement. De cette façon, vos membres spokes peuvent héberger des déploiements classiques tout en tirant parti des services partagés dans le hub.

## <a name="recommendations"></a>Recommandations

Les recommandations suivantes s’appliquent à la plupart des scénarios. Suivez ces recommandations, sauf si vous avez un besoin spécifique qui vous oblige à les ignorer.

### <a name="resource-groups"></a>Groupes de ressources

Le réseau virtuel hub et chaque réseau virtuel spoke peuvent être implémentés dans différents groupes de ressources, voire dans différents abonnements, tant qu’ils appartiennent au même locataire Azure Active Directory (Azure AD) dans la même région Azure. Cela permet de décentraliser la gestion de chaque charge de travail, tout en partageant les services gérés dans le réseau virtuel hub.

### <a name="vnet-and-gatewaysubnet"></a>Réseau virtuel et sous réseau GatewaySubnet

Créez un sous-réseau nommé *GatewaySubnet*, avec une plage d’adresses de /27. La passerelle de réseau virtuel requiert ce sous-réseau. En allouant 32 adresses à ce sous-réseau, vous devriez éviter les limitations de taille de passerelle.

Pour plus d’informations sur la configuration de la passerelle, consultez les architectures de référence suivantes, en fonction de votre type de connexion :

- [Réseau hybride utilisant ExpressRoute][guidance-expressroute]
- [Réseau hybride utilisant une passerelle VPN][guidance-vpn]

Pour accroître la disponibilité, vous pouvez utiliser ExpressRoute et un VPN pour le basculement. Consultez [Connecter un réseau local à Azure à l’aide d’ExpressRoute avec basculement VPN][hybrid-ha].

Une topologie hub-and-spoke peut également être utilisée sans passerelle, si vous n’avez pas besoin de connectivité avec votre réseau local. 

### <a name="vnet-peering"></a>Homologation de réseaux virtuels

L’appairage de réseaux virtuels est une relation non transitive entre deux réseaux virtuels. Si des membres spokes doivent se connecter les uns aux autres, envisagez d’ajouter une connexion d’appairage distincte entre ces membres spokes.

Toutefois, cette situation vous prive très rapidement de connexions d’appairage en raison du [nombre maximal d’appairages de réseaux virtuels par réseau virtuel][vnet-peering-limit]. Dans ce scénario, envisagez d’utiliser des itinéraires définis par l’utilisateur afin que le trafic destiné à un membre spoke soit envoyé à une appliance virtuelle réseau faisant office de routeur sur le réseau virtuel hub. Ainsi, les membres spokes peuvent se connecter les uns aux autres.

Vous pouvez également configurer les membres spokes afin qu’ils utilisent la passerelle de réseau virtuel hub pour communiquer avec des réseaux distants. Pour que le trafic de la passerelle circule entre le membre spoke et le hub, puis se connecte à des réseaux distants, vous devez effectuer les opérations suivantes :

  - Configurer la connexion d’appairage de réseaux virtuels dans le hub pour **autoriser le transit par passerelle**.
  - Configurer la connexion d’appairage de réseaux virtuels dans chaque membre spoke pour **utiliser des passerelles distantes**.
  - Configurer toutes les connexions d’appairages de réseaux virtuels pour **autoriser le trafic transféré**.

## <a name="considerations"></a>Considérations

### <a name="spoke-connectivity"></a>Connectivité entre membres spokes

Si vous avez besoin de connectivité entre des membres spokes, envisagez d’implémenter une appliance virtuelle réseau pour le routage dans le hub et d’utiliser des itinéraires définis par l’utilisateur dans le membre spoke pour transférer le trafic vers le hub.

![[2]][2]

Dans ce scénario, vous devez configurer les connexions d’appairage pour **autoriser le trafic transféré**.

### <a name="overcoming-vnet-peering-limits"></a>Surmonter les limites d’appairage de réseaux virtuels

Veillez à prendre en compte le [nombre maximal d’appairages de réseaux virtuels par réseau virtuel][vnet-peering-limit] dans Azure. Si vous décidez que vous avez besoin d’un nombre de membres spokes supérieur à celui autorisé par la limite, envisagez de créer une topologie hub-and-spoke/hub-and-spoke, où les membres spokes du premier niveau de membres spokes font également office de hubs. Le diagramme qui suit montre cette topologie.

![[3]][3]

Déterminez également les services qui sont partagés dans le hub, afin que ce dernier puisse prendre en charge un plus grand nombre de membres spokes. Par exemple, si votre hub fournit des services de pare-feu, tenez compte des limites de bande passante de votre solution de pare-feu quand vous ajoutez plusieurs membres spokes. Vous pourriez souhaiter transférer certains de ces services partagés vers un second niveau de hubs.

## <a name="deploy-the-solution"></a>Déployer la solution

Un déploiement pour cette architecture est disponible sur [GitHub][ref-arch-repo]. Il utilise des machines virtuelles dans chaque réseau virtuel pour tester la connectivité. Aucun service réel n’est hébergé dans le sous-réseau **shared-services** du **réseau virtuel hub**.

Le déploiement crée les groupes de ressources suivants dans votre abonnement :

- hub-nva-rg
- hub-vnet-rg
- onprem-jb-rg
- onprem-vnet-rg
- spoke1-vnet-rg
- spoke2-vent-rg

Les fichiers de paramètre modèle font référence à ces noms. Si vous les modifiez, mettez à jour les fichiers de paramètres afin qu’ils correspondent.

### <a name="prerequisites"></a>Prérequis

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-datacenter"></a>Déployer le centre de données local simulé

Pour déployer le centre de données local simulé en tant que réseau virtuel Azure, procédez comme suit :

1. Accédez au dossier `hybrid-networking/hub-spoke` du référentiel des architectures de référence.

2. Ouvrez le fichier `onprem.json` . Remplacez les valeurs de `adminUsername` et `adminPassword`.

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

3. Pour un déploiement Linux, définissez `osType` sur `Linux` (facultatif).

4. Exécutez la commande suivante :

    ```bash
    azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
    ```

5. Attendez que le déploiement se termine. Ce déploiement crée un réseau virtuel, une machine virtuelle et une passerelle VPN. La création de la passerelle VPN peut prendre environ 40 minutes.

### <a name="deploy-the-hub-vnet"></a>Déployer le réseau virtuel du hub

Pour déployer réseau virtuel du hub, procédez comme suit.

1. Ouvrez le fichier `hub-vnet.json` . Remplacez les valeurs de `adminUsername` et `adminPassword`.

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. Pour un déploiement Linux, définissez `osType` sur `Linux` (facultatif).

3. Recherchez les deux instances de `sharedKey`, puis entrez une clé partagée pour la connexion VPN. Les valeurs doivent correspondre.

    ```bash
    "sharedKey": "",
    ```

4. Exécutez la commande suivante :

    ```bash
    azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
    ```

5. Attendez que le déploiement se termine. Ce déploiement crée un réseau virtuel, une machine virtuelle, une passerelle VPN et une connexion à la passerelle.  La création de la passerelle VPN peut prendre environ 40 minutes.

### <a name="test-connectivity-with-the-hub"></a>Tester la connectivité avec le hub

Testez la connectivité entre l’environnement local simulé et le réseau virtuel du hub.

**Déploiement sous Windows**

1. Utilisez le portail Azure pour trouver la machine virtuelle appelée `jb-vm1` dans le groupe de ressources `onprem-jb-rg`.

2. Cliquez sur `Connect` pour ouvrir une session Bureau à distance vers la machine virtuelle. Utilisez le mot de passe spécifié dans le fichier de paramètre `onprem.json`.

3. Ouvrez une console PowerShell dans la machine virtuelle et utilisez la cmdlet `Test-NetConnection` pour vérifier que vous pouvez vous connecter à la machine virtuelle du serveur de rebond dans le réseau virtuel du hub.

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```
Le résultat doit être semblable à ce qui suit :

```powershell
ComputerName     : 10.0.0.68
RemoteAddress    : 10.0.0.68
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.1.000
TcpTestSucceeded : True
```

> [!NOTE]
> Par défaut, les machines virtuelles Windows Server n’autorisent pas les réponses ICMP dans Azure. Si vous souhaitez utiliser `ping` pour tester la connectivité, vous devez activer le trafic ICMP dans le pare-feu Windows avancé et ce, pour chaque machine virtuelle.

**Déploiement sous Linux**

1. Utilisez le portail Azure pour trouver la machine virtuelle appelée `jb-vm1` dans le groupe de ressources `onprem-jb-rg`.

2. Cliquez sur `Connect` et copiez la commande `ssh` qui s’affiche dans le portail. 

3. Depuis un invite Linux, exécutez `ssh` pour vous connecter à l’environnement local simulé. Utilisez le mot de passe spécifié dans le fichier de paramètre `onprem.json`.

4. Utilisez la commande `ping` pour tester la connectivité avec la machine virtuelle du serveur de rebond dans le réseau virtuel du hub :

   ```bash
   ping 10.0.0.68
   ```

### <a name="deploy-the-spoke-vnets"></a>Déployer les réseaux virtuels spokes

Pour déployer les réseaux virtuels spokes, procédez comme suit.

1. Ouvrez le fichier `spoke1.json` . Remplacez les valeurs de `adminUsername` et `adminPassword`.

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. Pour un déploiement Linux, définissez `osType` sur `Linux` (facultatif).

3. Exécutez la commande suivante :

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```
  
4. Répétez les étapes 1-2 pour le fichier `spoke2.json`.

5. Exécutez la commande suivante :

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

6. Exécutez la commande suivante :

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
   ```

### <a name="test-connectivity"></a>Tester la connectivité

Testez la connectivité entre l’environnement local simulé et les réseaux virtuels spokes.

**Déploiement sous Windows**

1. Utilisez le portail Azure pour trouver la machine virtuelle appelée `jb-vm1` dans le groupe de ressources `onprem-jb-rg`.

2. Cliquez sur `Connect` pour ouvrir une session Bureau à distance vers la machine virtuelle. Utilisez le mot de passe spécifié dans le fichier de paramètre `onprem.json`.

3. Ouvrez une console PowerShell dans la machine virtuelle et utilisez l’applet de commande `Test-NetConnection` pour vérifier que vous pouvez vous connecter aux machines virtuelles du serveur de rebond dans les réseaux virtuels spokes.

   ```powershell
   Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
   Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
   ```

**Déploiement sous Linux**

Pour tester la connectivité entre l’environnement local simulé et le réseau virtuel spoke à l’aide de machines virtuelles Linux, procédez comme suit :

1. Utilisez le portail Azure pour trouver la machine virtuelle appelée `jb-vm1` dans le groupe de ressources `onprem-jb-rg`.

2. Cliquez sur `Connect` et copiez la commande `ssh` qui s’affiche dans le portail. 

3. Depuis un invite Linux, exécutez `ssh` pour vous connecter à l’environnement local simulé. Utilisez le mot de passe spécifié dans le fichier de paramètre `onprem.json`.

5. Utilisez la commande `ping` pour tester la connectivité avec les machines virtuelles du serveur de rebond dans chaque spoke :

   ```bash
   ping 10.1.0.68
   ping 10.2.0.68
   ```

### <a name="add-connectivity-between-spokes"></a>Ajouter la connectivité entre les membres spokes

Cette étape est facultative. Si vous souhaitez autoriser les spokes à se connecter les uns aux autres, vous devez utiliser une appliance virtuelle réseau en tant que routeur dans le réseau virtuel du hub, et forcer le trafic entre les spokes et le routeur lors de la tentative de connexion à un autre spoke. Pour déployer une appliance virtuelle réseau de base en tant que machine virtuelle unique, ainsi que des itinéraires définis par l’utilisateur pour permettre la connexion des réseaux virtuels spokes, procédez comme suit :

1. Ouvrez le fichier `hub-nva.json` . Remplacez les valeurs de `adminUsername` et `adminPassword`.

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. Exécutez la commande suivante :

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Topologie hub-and-spoke dans Azure"
[1]: ./images/hub-spoke-gateway-routing.svg "Topologie hub-and-spoke dans Azure avec routage transitif"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Topologie hub-and-spoke dans Azure avec routage transitif utilisant une appliance virtuelle réseau"
[3]: ./images/hub-spokehub-spoke.svg "Topologie hub-and-spoke/hub-and-spoke dans Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
