---
title: Implémentation d’une topologie de réseau hub-and-spoke avec des services partagés dans Azure
description: Procédure d’implémentation d’une topologie de réseau hub-and-spoke avec des services partagés dans Azure.
author: telmosampaio
ms.date: 02/25/2018
pnp.series.title: Implement a hub-spoke network topology with shared services in Azure
pnp.series.prev: hub-spoke
ms.openlocfilehash: 83367a3be2f7a1e33c2ef7018d42f70aae99104d
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/16/2018
---
# <a name="implement-a-hub-spoke-network-topology-with-shared-services-in-azure"></a>Implémentation d’une topologie de réseau hub-and-spoke avec des services partagés dans Azure

Cette architecture de référence s’appuie sur l’architecture de référence [hub-and-spoke][guidance-hub-spoke] de manière à inclure dans le hub des services partagés qui peuvent être utilisés par tous les spokes. Les premiers services que vous devez partager, en tant que première étape de la migration d’un centre de données vers le cloud et la création d’un [centre de données virtuel], sont l’identité et la sécurité. Cette architecture de référence vous montre comment étendre vos services Active Directory à partir de votre centre de données local vers Azure, et comment ajouter une appliance virtuelle réseau qui peut jouer le rôle de pare-feu dans une topologie hub-and-spoke.  [**Déployez cette solution**](#deploy-the-solution).

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

* **Sous-réseau de services partagés**. Sous-réseau dans le réseau virtuel hub utilisé pour héberger les services que peuvent partager tous les membres spokes, tels que DNS ou AD DS.

* **Sous-réseau de DMZ**. Sous-réseau du réseau virtuel du hub utilisé pour héberger les appliances virtuelles réseau qui peuvent jouer le rôle d’appliances de sécurité, comme les pare-feu.

* **Réseaux virtuels spokes**. Un ou plusieurs réseaux virtuels Azure qui sont utilisés comme membres spokes dans la topologie hub-and-spoke. Les membres spokes peuvent servir à isoler les charges de travail dans leurs propres réseaux virtuels, qui sont alors gérées séparément des autres membres spokes. Chaque charge de travail peut inclure plusieurs niveaux, avec plusieurs sous-réseaux connectés à l’aide d’équilibreurs de charge Azure. Pour plus d’informations sur l’infrastructure d’application, consultez [Running Windows VM workloads][windows-vm-ra] (Exécution de charges de travail de machine virtuelle Windows) et [Exécution de charges de travail de machine virtuelle Linux][linux-vm-ra].

* **Appairage de réseaux virtuels**. Deux réseaux virtuels dans la même région Azure peuvent être connectés à l’aide d’une [connexion d’appairage][vnet-peering]. Les connexions d’appairage sont des connexions non transitives et à faible latence entre des réseaux virtuels. Une fois appairés, les réseaux virtuels échangent le trafic à l’aide de la dorsale principale d’Azure, sans avoir besoin d’un routeur. Dans une topologie de réseau hub-and-spoke, vous utilisez l’appairage de réseaux virtuels pour connecter le hub à chaque membre spoke.

> [!NOTE]
> Cet article couvre uniquement les déploiements [Resource Manager](/azure/azure-resource-manager/resource-group-overview), mais vous pouvez également connecter un réseau virtuel classique à un réseau virtuel Resource Manager dans un même abonnement. De cette façon, vos membres spokes peuvent héberger des déploiements classiques tout en tirant parti des services partagés dans le hub.

## <a name="recommendations"></a>Recommandations

Toutes les recommandations de l’architecture de référence [hub-and-spoke][guidance-hub-spoke] concernent également celle des services partagés. 

De plus, les recommandations suivantes s’appliquent à la plupart des scénarios relatifs aux services partagés. Suivez ces recommandations, sauf si vous avez un besoin spécifique qui vous oblige à les ignorer.

### <a name="identity"></a>Identité

La plupart des organisations incluent un environnement Active Directory Directory Services (AD DS) dans leur centre de données local. Pour faciliter la gestion des ressources déplacées vers Azure à partir de votre réseau local et qui dépendent d’AD DS, il est recommandé d’héberger les contrôleurs de domaine AD DS dans Azure.

Si vous devez utiliser des objets de stratégie de groupe, que vous souhaitez contrôler séparément pour Azure et votre environnement local, utilisez un site Active Directory différent pour chaque région Azure. Placez vos contrôleurs de domaine dans un réseau virtuel central (hub) auquel les charges de travail dépendantes peuvent accéder.

### <a name="security"></a>Sécurité

Lorsque vous déplacez des charges de travail depuis votre environnement local vers Azure, certaines d’entre elles doivent être hébergées sur des machines virtuelles. Pour des raisons de conformité, vous devrez peut-être appliquer des restrictions sur le trafic qui traverse ces charges de travail. 

Vous pouvez utiliser les appliances virtuelles réseau dans Azure pour héberger différents types de services de gestion de la sécurité et des performances. Si vous êtes familiarisé avec un ensemble donné d’appliances locales, il est recommandé d’utiliser les mêmes appliances virtualisées dans Azure, le cas échéant.

> [!NOTE]
> Les scripts de déploiement de cette architecture de référence utilisent une machine virtuelle Ubuntu avec transfert d’IP activé pour imiter une appliance virtuelle réseau.

## <a name="considerations"></a>Considérations

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

3. Installez le package npm des [modules Azure][azbb].

4. À partir d’une invite de commandes, d’une invite bash ou de l’invite de commandes PowerShell, connectez-vous à votre compte Azure à l’aide de la commande ci-dessous et suivez les invites.

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a>Déployer le centre de données local simulé à l’aide d’azbb

Pour déployer le centre de données local simulé en tant que réseau virtuel Azure, procédez comme suit :

1. Accédez au dossier `hybrid-networking\shared-services-stack\` pour le dépôt que vous avez téléchargé à l’étape des prérequis ci-dessus.

2. Ouvrez le fichier `onprem.json` et entrez un nom d’utilisateur et un mot de passe entre les guillemets aux lignes 45 et 46, comme illustré ci-dessous, puis enregistrez le fichier.

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

3. Exécutez `azbb` pour déployer l’environnement local simulé, comme indiqué ci-dessous.

   ```bash
   azbb -s <subscription_id> -g onprem-vnet-rg - l <location> -p onoprem.json --deploy
   ```
   > [!NOTE]
   > Si vous décidez d’utiliser un nom de groupe de ressources différent (autre que `onprem-vnet-rg`), veillez à rechercher tous les fichiers de paramètre qui utilisent ce nom et à les modifier afin qu’ils utilisent votre propre nom de groupe de ressources.

4. Attendez que le déploiement se termine. Ce déploiement crée un réseau virtuel, une machine virtuelle exécutant Windows et une passerelle VPN. La création de la passerelle VPN peut prendre plus de 40 minutes.

### <a name="azure-hub-vnet"></a>Réseau virtuel hub Azure

Pour déployer le réseau virtuel hub et vous connecter au réseau virtuel local simulé créé ci-dessus, effectuez les étapes suivantes.

1. Ouvrez le fichier `hub-vnet.json` et entrez un nom d’utilisateur et un mot de passe entre les guillemets aux lignes 50 et 51, comme illustré ci-dessous.

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

2. À la ligne 52, pour `osType`, saisissez `Windows` ou `Linux` pour installer le système d’exploitation Windows Server 2016 Datacenter ou Ubuntu 16.04 sur le serveur de rebond.

3. Saisissez une clé partagée entre les guillemets à la ligne 83, comme illustré ci-dessous, puis enregistrez le fichier.

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

### <a name="adds-in-azure"></a>ADDS dans Azure

Pour déployer des contrôleurs de domaine AD DS dans Azure, procédez comme suit.

1. Ouvrez le fichier `hub-adds.json` et entrez un nom d’utilisateur et un mot de passe entre les guillemets aux lignes 14 et 15, comme illustré ci-dessous, puis enregistrez le fichier.

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

2. Exécutez `azbb` pour déployer les contrôleurs de domaine AD DS, comme indiqué ci-dessous.

   ```bash
   azbb -s <subscription_id> -g hub-adds-rg - l <location> -p hub-adds.json --deploy
   ```
  
   > [!NOTE]
   > Si vous décidez d’utiliser un nom de groupe de ressources différent (autre que `hub-adds-rg`), veillez à rechercher tous les fichiers de paramètre qui utilisent ce nom et à les modifier afin qu’ils utilisent votre propre nom de groupe de ressources.

   > [!NOTE]
   > Cette partie du déploiement peut prendre plusieurs minutes, car elle requiert la jonction de deux machines virtuelles au domaine hébergé dans le centre de données local simulé, puis l’installation d’AD DS sur ces dernières.

### <a name="nva"></a>Appliances virtuelles réseau

Pour déployer une appliance virtuelle réseau dans le sous-réseau `dmz`, effectuez les opérations suivantes :

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

### <a name="azure-spoke-vnets"></a>Réseaux virtuels spokes Azure

Pour déployer les réseaux virtuels spokes, procédez comme suit.

1. Ouvrez le fichier `spoke1.json` et entrez un nom d’utilisateur et un mot de passe entre les guillemets aux lignes 52 et 53, comme illustré ci-dessous, puis enregistrez le fichier.

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

2. À la ligne 54, pour `osType`, saisissez `Windows` ou `Linux` pour installer le système d’exploitation Windows Server 2016 Datacenter ou Ubuntu 16.04 sur le serveur de rebond.

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

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[guidance-hub-spoke]: ./hub-spoke.md
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[centre de données virtuel]: https://aka.ms/vdc
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/shared-services.png "Topologie de service partagé dans Azure"
[3]: ./images/hub-spokehub-spoke.svg "Topologie hub-and-spoke/hub-and-spoke dans Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
