---
title: Implémentation d’une topologie de réseau hub-and-spoke dans Azure
description: Comment implémenter une topologie de réseau hub-and-spoke dans Azure.
author: telmosampaio
ms.date: 02/23/2018
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: 243ad026c7c9703d9659cbef6815131fcdaa8a11
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/06/2018
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

Un déploiement pour cette architecture est disponible sur [GitHub][ref-arch-repo]. Il utilise des machines virtuelles Ubuntu dans chaque réseau virtuel pour tester la connectivité. Aucun service réel n’est hébergé dans le sous-réseau **shared-services** du **réseau virtuel hub**.

### <a name="prerequisites"></a>Prérequis


Avant de pouvoir déployer l’architecture de référence sur votre propre abonnement, vous devez effectuer les étapes suivantes.

1. Clonez, dupliquez ou téléchargez le fichier zip pour le référentiel GitHub des [architectures de référence][ref-arch-repo].

2. Vérifiez qu’Azure CLI 2.0 est installé sur votre ordinateur. Pour des instructions d’installation de l’interface de ligne de commande, consultez [Installer Azure CLI 2.0][azure-cli-2].

3. Installez le package npm des [blocs de construction Azure][azbb].

4. À partir d’une invite de commandes, d’une invite bash ou de l’invite de commandes PowerShell, connectez-vous à votre compte Azure à l’aide de la commande ci-dessous et suivez les invites.

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a>Déployer le centre de données local simulé à l’aide d’azbb

Pour déployer le centre de données local simulé en tant que réseau virtuel Azure, procédez comme suit :

1. Accédez au dossier `hybrid-networking\hub-spoke\` pour le dépôt que vous avez téléchargé à l’étape des prérequis ci-dessus.

2. Ouvrez le fichier `onprem.json` et entrez un nom d’utilisateur et un mot de passe entre les guillemets aux lignes 36 et 37, comme illustré ci-dessous, puis enregistrez le fichier.

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

3. À la ligne 38, pour `osType`, saisissez `Windows` ou `Linux` pour installer le système d’exploitation Windows Server 2016 Datacenter ou Ubuntu 16.04 sur le serveur de rebond.

4. Exécutez `azbb` pour déployer l’environnement local simulé, comme indiqué ci-dessous.

   ```bash
   azbb -s <subscription_id> -g onprem-vnet-rg - l <location> -p onoprem.json --deploy
   ```
   > [!NOTE]
   > Si vous décidez d’utiliser un nom de groupe de ressources différent (autre que `onprem-vnet-rg`), veillez à rechercher tous les fichiers de paramètre qui utilisent ce nom et à les modifier afin qu’ils utilisent votre propre nom de groupe de ressources.

5. Attendez que le déploiement se termine. Ce déploiement crée un réseau virtuel, une machine virtuelle et une passerelle VPN. La création de la passerelle VPN peut prendre plus de 40 minutes.

### <a name="azure-hub-vnet"></a>Réseau virtuel hub Azure

Pour déployer le réseau virtuel hub et vous connecter au réseau virtuel local simulé créé ci-dessus, effectuez les étapes suivantes.

1. Ouvrez le fichier `hub-vnet.json` et entrez un nom d’utilisateur et un mot de passe entre les guillemets aux lignes 39 et 40, comme illustré ci-dessous.

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

2. À la ligne 41, pour `osType`, saisissez `Windows` ou `Linux` pour installer le système d’exploitation Windows Server 2016 Datacenter ou Ubuntu 16.04 sur le serveur de rebond.

3. Saisissez une clé partagée entre les guillemets à la ligne 72, comme illustré ci-dessous, puis enregistrez le fichier.

   ```bash
   "sharedKey": "",
   ```

4. Exécutez `azbb` pour déployer l’environnement local simulé, comme indiqué ci-dessous.

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet.json --deploy
   ```
   > [!NOTE]
   > Si vous décidez d’utiliser un nom de groupe de ressources différent (autre que `hub-vnet-rg`), veillez à rechercher tous les fichiers de paramètre qui utilisent ce nom et à les modifier afin qu’ils utilisent votre propre nom de groupe de ressources.

5. Attendez que le déploiement se termine. Ce déploiement crée un réseau virtuel, une machine virtuelle, une passerelle VPN et une connexion à la passerelle créée à la section précédente. La création de la passerelle VPN peut prendre plus de 40 minutes.

### <a name="optional-test-connectivity-from-onprem-to-hub"></a>(Facultatif) Tester la connectivité entre l’instance locale et le hub

Pour tester la connectivité entre l’environnement local simulé et le réseau virtuel du hub à l’aide de machines virtuelles Windows, procédez comme suit.

1. À partir du portail Azure, accédez au groupe de ressources `onprem-jb-rg`, puis cliquez sur la ressource de machine virtuelle `jb-vm1`.

2. Dans l’angle supérieur gauche du volet de la machine virtuelle, dans le portail, cliquez sur `Connect`, puis suivez les invites afin d’utiliser le Bureau à distance pour vous connecter à la machine virtuelle. Assurez-vous d’utiliser le nom d’utilisateur et le mot de passe spécifiés aux lignes 36 et 37 du fichier `onprem.json`.

3. Ouvrez une console PowerShell dans la machine virtuelle et utilisez la cmdlet `Test-NetConnection` pour vérifier que vous pouvez vous connecter à la machine virtuelle du serveur de rebond dans le hub, comme indiqué ci-dessous.

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```
   > [!NOTE]
   > Par défaut, les machines virtuelles Windows Server n’autorisent pas les réponses ICMP dans Azure. Si vous souhaitez utiliser `ping` pour tester la connectivité, vous devez activer le trafic ICMP dans le pare-feu Windows avancé et ce, pour chaque machine virtuelle.

Pour tester la connectivité entre l’environnement local simulé et le réseau virtuel du hub à l’aide de machines virtuelles Linux, procédez comme suit :

1. À partir du portail Azure, accédez au groupe de ressources `onprem-jb-rg`, puis cliquez sur la ressource de machine virtuelle `jb-vm1`.

2. Dans l’angle supérieur gauche du panneau de votre machine virtuelle, dans le portail, cliquez sur `Connect`, puis copiez la commande `ssh` affichée sur le portail. 

3. À partir d’une invite Linux, exécutez `ssh` pour vous connecter au serveur de rebond de l’environnement local simulé en utilisant les informations copiées à l’étape 2 (ci-dessus), comme indiqué ici.

   ```bash
   ssh <your_user>@<public_ip_address>
   ```

4. Utilisez le mot de passe que vous avez spécifié à la ligne 37 dans le fichier `onprem.json` pour vous connecter à la machine virtuelle.

5. La commande `ping` vous permet de tester la connectivité au serveur de rebond du hub, comme indiqué ci-dessous.

   ```bash
   ping 10.0.0.68
   ```

### <a name="azure-spoke-vnets"></a>Réseaux virtuels spokes Azure

Pour déployer les réseaux virtuels spokes, procédez comme suit.

1. Ouvrez le fichier `spoke1.json` et entrez un nom d’utilisateur et un mot de passe entre les guillemets aux lignes 47 et 48, comme illustré ci-dessous, puis enregistrez le fichier.

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

2. À la ligne 49, pour `osType`, saisissez `Windows` ou `Linux` pour installer le système d’exploitation Windows Server 2016 Datacenter ou Ubuntu 16.04 sur le serveur de rebond.

3. Exécutez `azbb` pour déployer le premier environnement de réseau virtuel spoke, comme indiqué ci-dessous.

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg - l <location> -p spoke1.json --deploy
   ```
  
   > [!NOTE]
   > Si vous décidez d’utiliser un nom de groupe de ressources différent (autre que `spoke1-vnet-rg`), veillez à rechercher tous les fichiers de paramètre qui utilisent ce nom et à les modifier afin qu’ils utilisent votre propre nom de groupe de ressources.

4. Répétez l’étape 1 ci-dessus pour le fichier `spoke2.json`.

5. Exécutez `azbb` pour déployer le deuxième environnement de réseau virtuel spoke, comme indiqué ci-dessous.

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg - l <location> -p spoke2.json --deploy
   ```
   > [!NOTE]
   > Si vous décidez d’utiliser un nom de groupe de ressources différent (autre que `spoke2-vnet-rg`), veillez à rechercher tous les fichiers de paramètre qui utilisent ce nom et à les modifier afin qu’ils utilisent votre propre nom de groupe de ressources.

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a>Appairage de réseaux virtuels hub Azure avec des réseaux virtuels spokes

Procédez comme suit pour créer une connexion d’homologation entre le réseau virtuel du hub et les réseaux virtuels spokes.

1. Ouvrez le fichier `hub-vnet-peering.json` et vérifiez l’exactitude du nom du groupe de ressources et de celui du réseau virtuel de chaque homologation de réseau virtuel, à compter de la ligne 29.

2. Exécutez `azbb` pour déployer le premier environnement de réseau virtuel spoke, comme indiqué ci-dessous.

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet-peering.json --deploy
   ```

   > [!NOTE]
   > Si vous décidez d’utiliser un nom de groupe de ressources différent (autre que `hub-vnet-rg`), veillez à rechercher tous les fichiers de paramètre qui utilisent ce nom et à les modifier afin qu’ils utilisent votre propre nom de groupe de ressources.

### <a name="test-connectivity"></a>Tester la connectivité

Pour tester la connectivité entre l’environnement local simulé et le réseau virtuel spoke à l’aide de machines virtuelles Windows, procédez comme suit.

1. À partir du portail Azure, accédez au groupe de ressources `onprem-jb-rg`, puis cliquez sur la ressource de machine virtuelle `jb-vm1`.

2. Dans l’angle supérieur gauche du volet de la machine virtuelle, dans le portail, cliquez sur `Connect`, puis suivez les invites afin d’utiliser le Bureau à distance pour vous connecter à la machine virtuelle. Assurez-vous d’utiliser le nom d’utilisateur et le mot de passe spécifiés aux lignes 36 et 37 du fichier `onprem.json`.

3. Ouvrez une console PowerShell dans la machine virtuelle et utilisez la cmdlet `Test-NetConnection` pour vérifier que vous pouvez vous connecter à la machine virtuelle du serveur de rebond dans le hub, comme indiqué ci-dessous.

   ```powershell
   Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
   Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
   ```

Pour tester la connectivité entre l’environnement local simulé et le réseau virtuel spoke à l’aide de machines virtuelles Linux, procédez comme suit :

1. À partir du portail Azure, accédez au groupe de ressources `onprem-jb-rg`, puis cliquez sur la ressource de machine virtuelle `jb-vm1`.

2. Dans l’angle supérieur gauche du panneau de votre machine virtuelle, dans le portail, cliquez sur `Connect`, puis copiez la commande `ssh` affichée sur le portail. 

3. À partir d’une invite Linux, exécutez `ssh` pour vous connecter au serveur de rebond de l’environnement local simulé en utilisant les informations copiées à l’étape 2 (ci-dessus), comme indiqué ici.

   ```bash
   ssh <your_user>@<public_ip_address>
   ```

4. Utilisez le mot de passe que vous avez spécifié à la ligne 37 dans le fichier `onprem.json` pour vous connecter à la machine virtuelle.

5. La commande `ping` vous permet de tester la connectivité avec les machines virtuelles du serveur de rebond dans chaque spoke, comme indiqué ci-dessous.

   ```bash
   ping 10.1.0.68
   ping 10.2.0.68
   ```

### <a name="add-connectivity-between-spokes"></a>Ajouter la connectivité entre les membres spokes

Si vous souhaitez autoriser les spokes à se connecter les uns aux autres, vous devez utiliser une appliance virtuelle réseau en tant que routeur dans le réseau virtuel du hub, et forcer le trafic entre les spokes et le routeur lors de la tentative de connexion à un autre spoke. Pour déployer une appliance virtuelle réseau de base en tant que machine virtuelle unique, ainsi que les itinéraires définis par l’utilisateur qui sont requis pour permettre la connexion des réseaux virtuels spokes, procédez comme suit :

1. Ouvrez le fichier `hub-nva.json` et entrez un nom d’utilisateur et un mot de passe entre les guillemets aux lignes 13 et 14, comme illustré ci-dessous, puis enregistrez le fichier.

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```
2. Exécutez `azbb` pour déployer la VM de l’appliance virtuelle réseau et les itinéraires définis par l’utilisateur.

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg - l <location> -p hub-nva.json --deploy
   ```
   > [!NOTE]
   > Si vous décidez d’utiliser un nom de groupe de ressources différent (autre que `hub-nva-rg`), veillez à rechercher tous les fichiers de paramètre qui utilisent ce nom et à les modifier afin qu’ils utilisent votre propre nom de groupe de ressources.

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
