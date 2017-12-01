---
title: "Exécuter des machines virtuelles Windows pour une architecture multiniveau"
description: "Découvrez comment implémenter une architecture multiniveau dans Azure, en privilégiant la disponibilité, la sécurité, la scalabilité et la facilité de gestion."
author: MikeWasson
ms.date: 11/22/2016
pnp.series.title: Windows VM workloads
pnp.series.next: multi-region-application
pnp.series.prev: multi-vm
ms.openlocfilehash: e25d10d661ac4759f209bd27384303dee2ee454e
ms.sourcegitcommit: 583e54a1047daa708a9b812caafb646af4d7607b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/28/2017
---
# <a name="run-windows-vms-for-an-n-tier-application"></a>Exécuter des machines virtuelles Windows pour une architecture multiniveau

Cette architecture de référence présente un ensemble de pratiques éprouvées pour l’exécution de machines virtuelles Windows pour une application multiniveau. [**Déployez cette solution**.](#deploy-the-solution) 

![[0]][0]

*Téléchargez un [fichier Visio][visio-download] de cette architecture.*

## <a name="architecture"></a>Architecture 

Il existe de nombreuses façons d’implémenter une architecture multiniveau. Le diagramme illustre une application web à 3 niveaux standard. Cette architecture repose sur celle décrite dans [Exécuter des machines virtuelles à charge équilibrée pour la scalabilité et la disponibilité][multi-vm]. Les niveaux Web et Business utilisent des machines virtuelles à charge équilibrée.

* **Groupes à haute disponibilité.** Créez un [groupe à haute disponibilité][azure-availability-sets] pour chaque niveau et configurez au moins deux machines virtuelles dans chaque niveau. Cela rend les machines virtuelles éligibles pour un [niveau contrat de service (SLA)][vm-sla] plus élevé. Vous pouvez déployer une seule machine virtuelle dans un groupe à haute disponibilité, mais cette machine virtuelle unique ne sera pas éligible à la garantie de contrat de niveau de service, à moins qu’elle n’utilise le stockage Premium Azure pour tous les systèmes d’exploitation et disques de données.  
* **Sous-réseaux.** Créez un sous-réseau distinct pour chaque niveau. Spécifiez la plage d’adresses et le masque de sous-réseau d’après la notation [CIDR]. 
* **Équilibreurs de charge.** Utilisez un [équilibreur de charge accessible sur Internet][load-balancer-external] pour distribuer le trafic Internet entrant vers le niveau Web et un [équilibreur de charge interne][load-balancer-internal] pour distribuer le trafic réseau du niveau Web vers le niveau Business.
* **Serveur de rebond (jumpbox).** Également appelée [hôte bastion]. Machine virtuelle sécurisée sur le réseau, utilisée par les administrateurs pour se connecter aux autres machines virtuelles. Le serveur de rebond a un groupe de sécurité réseau qui autorise le trafic distant provenant uniquement d’adresses IP publiques figurant sur une liste verte. Le groupe de sécurité réseau doit autoriser le trafic RDP (Bureau à distance).
* **Surveillance.** Des logiciels de surveillance tels que [Nagios], [Zabbix] ou [Icinga] peuvent vous donner une idée du temps de réponse, du temps de fonctionnement de machine virtuelle et de l’intégrité globale de votre système. Installez le logiciel de surveillance sur une machine virtuelle placée sur un sous-réseau de gestion distinct.
* **Groupes de sécurité réseau.** Utilisez des [groupes de sécurité réseau][nsg] pour limiter le trafic réseau au sein du réseau virtuel. Par exemple, dans l’architecture à 3 niveaux illustrée ici, le niveau base de données n’accepte pas le trafic en provenance du frontend web, mais uniquement du niveau Business et du sous-réseau de gestion.
* **Groupe de disponibilité SQL Server AlwaysOn.** Fournit une haute disponibilité du niveau Données, en activant la réplication et le basculement.
* **Serveurs AD DS (Active Directory Domain Services)**. Avec les versions antérieures à Windows Server 2016, les groupes de disponibilité SQL Server AlwaysOn doivent être joints à un domaine. Ceci est dû au fait que les groupes de disponibilité reposent sur la technologie WSFC (Cluster de basculement Windows Server). Windows Server 2016 introduit la possibilité de créer un cluster de basculement sans Active Directory, auquel cas les serveurs de domaine Active Directory ne sont pas nécessaires pour cette architecture. Pour plus d’informations, consultez [Nouveautés du clustering de basculement dans Windows Server 2016][wsfc-whats-new].

## <a name="recommendations"></a>Recommandations

Vos exigences peuvent différer de celles de l’architecture décrite ici. Utilisez ces recommandations comme point de départ. 

### <a name="vnet--subnets"></a>Réseau virtuel / sous-réseaux

Quand vous créez le réseau virtuel, déterminez le nombre d’adresses IP dont vos ressources sur chaque sous-réseau ont besoin. Spécifiez un masque de sous-réseau et une plage d’adresses de réseau virtuel suffisamment large pour les adresse IP requises, à l’aide de la notation [CIDR]. Utilisez un espace d’adressage qui se trouve dans les [blocs d’adresses IP privées][private-ip-space] standard, à savoir 10.0.0.0/8, 172.16.0.0/12 et 192.168.0.0/16.

Choisissez une plage d’adresses qui ne chevauche pas votre réseau local, au cas où vous devriez configurer ultérieurement une passerelle entre le réseau virtuel et votre réseau local. Une fois le réseau virtuel créé, vous ne pouvez plus modifier la plage d’adresses.

Concevez les sous-réseaux en tenant compte des exigences en matière de sécurité et de fonctionnalités. Toutes les machines virtuelles du même niveau ou rôle doivent être placées sur le même sous-réseau, qui peut constituer une limite de sécurité. Pour plus d’informations sur la conception des réseaux virtuels et des sous-réseaux, consultez [Planifier et concevoir des réseaux virtuels Azure][plan-network].

Pour chaque sous-réseau, spécifiez l’espace d’adressage du sous-réseau d’après la notation CIDR. Par exemple, « 10.0.0.0/24 » crée une plage de 256 adresses IP. Les machines virtuelles peuvent en utiliser 251, tandis que cinq sont réservées. Vérifiez que les plages d’adresses ne chevauchent pas plusieurs sous-réseaux. Voir le [FAQ sur les réseaux virtuels][vnet faq].

### <a name="network-security-groups"></a>groupes de sécurité réseau ;

Utilisez des règles de groupe de sécurité réseau pour limiter le trafic entre les niveaux. Par exemple, dans l’architecture à 3 niveaux ci-dessus, le niveau Web ne communique pas directement avec le niveau Base de données. Pour appliquer cette recommandation, le niveau Base de données doit bloquer le trafic entrant provenant du sous-réseau du niveau Web.  

1. Créez un groupe de sécurité réseau et associez-le au sous-réseau du niveau Base de données.
2. Ajoutez une règle qui interdit tout le trafic entrant provenant du réseau virtuel. (Utilisez la balise `VIRTUAL_NETWORK` dans la règle.) 
3. Ajoutez une règle avec une priorité plus élevée qui autorise le trafic entrant provenant du sous-réseau du niveau Business. Cette règle remplace la règle précédente, et permet au niveau Business de communiquer avec le niveau Base de données.
4. Ajoutez une règle qui autorise le trafic entrant provenant du sous-réseau du niveau Base de données. Cette règle autorise la communication entre les machines virtuelles du niveau Base de données, qui est nécessaire pour la réplication de base de données et le basculement.
5. Ajoutez une règle qui autorise le trafic RDP à partir du sous-réseau du serveur de rebond. Cette règle permet aux administrateurs de se connecter au niveau Base de données à partir du serveur de rebond.
   
   > [!NOTE]
   > Un groupe de sécurité réseau a des règles par défaut qui autorisent tout le trafic entrant provenant du réseau virtuel. Vous ne pouvez pas supprimer ces règles, mais vous pouvez les remplacer en créant des règles de priorité plus élevée.
   > 
   > 

### <a name="load-balancers"></a>Équilibreurs de charge

L’équilibreur de charge externe distribue le trafic Internet vers le niveau Web. Créez une adresse IP publique pour cet équilibreur de charge. Voir [Création d’un équilibreur de charge accessible sur Internet][lb-external-create].

L’équilibreur de charge interne distribue le trafic réseau du niveau Web vers le niveau Business. Pour attribuer une adresse IP privée à cet équilibreur de charge, créez une configuration IP frontend et associez-la au sous-réseau du niveau Business. Voir [Bien démarrer avec la création d’un équilibreur de charge interne][lb-internal-create].

### <a name="sql-server-always-on-availability-groups"></a>Configurer les groupes de disponibilité SQL Server AlwaysOn

Nous vous recommandons d’utiliser des [groupes de disponibilité AlwaysOn][sql-alwayson] pour la haute disponibilité de SQL Server. Avec les versions antérieures à Windows Server 2016, les groupes de disponibilité AlwaysOn nécessitent un contrôleur de domaine et tous les nœuds du groupe de disponibilité doivent être dans le même domaine Active Directory.

Les autres niveaux se connectent à la base de données par le biais d’un [écouteur de groupe de disponibilité][sql-alwayson-listeners]. L’écouteur permet à un client SQL de se connecter sans connaître le nom de l’instance physique de SQL Server. Les machines virtuelles qui accèdent à la base de données doivent être jointes au domaine. Le client (dans ce cas, un autre niveau) utilise le système DNS pour résoudre le nom du réseau virtuel de l’écouteur en adresses IP.

Configurez le groupe de disponibilité SQL Server AlwaysOn comme suit :

1. Créez un cluster WSFC (clustering de basculement Windows Server), un groupe de disponibilité SQL Server AlwaysOn et un réplica principal. Pour plus d’informations, consultez [Bien démarrer avec les groupes de disponibilité AlwaysOn][sql-alwayson-getting-started]. 
2. Créez un équilibreur de charge interne avec une adresse IP privée statique.
3. Créez un écouteur de groupe de disponibilité et mappez le nom DNS de l’écouteur à l’adresse IP d’un équilibreur de charge interne. 
4. Créez une règle d’équilibreur de charge pour le port d’écoute SQL Server (port TCP 1433 par défaut). La règle d’équilibreur de charge doit activer *l’adresse IP flottante*, également appelé Retour direct du serveur. Cela force la machine virtuelle à répondre directement au client, ce qui permet de bénéficier d’une connexion directe au réplica principal.
  
  > [!NOTE]
  > Quand l’adresse IP flottante est activée, le numéro de port frontend doit être identique au numéro de port backend dans la règle d’équilibreur de charge.
  > 
  > 

Quand un client SQL tente de se connecter, l’équilibreur de charge achemine la demande de connexion au réplica principal. En cas de basculement vers un autre réplica, l’équilibreur de charge achemine automatiquement les demandes suivantes à un nouveau réplica principal. Pour plus d’informations, consultez [Configurer un écouteur à équilibrage de charge interne pour des groupes de disponibilité SQL Server AlwaysOn][sql-alwayson-ilb].

Lors d’un basculement, les connexions clientes existantes sont fermées. Une fois le basculement terminé, les nouvelles connexions sont acheminées vers le nouveau réplica principal.

Si votre application effectue beaucoup plus de lectures que d’écritures, vous pouvez décharger certaines des requêtes en lecture seule vers un réplica secondaire. Voir [Utilisation d’un écouteur pour se connecter à un réplica secondaire en lecture seule (routage en lecture seule)][sql-alwayson-read-only-routing].

Testez votre déploiement en [forçant un basculement manuel][sql-alwayson-force-failover] du groupe de disponibilité.

### <a name="jumpbox"></a>Serveur de rebond

Le serveur de rebond ayant des exigences de performances minimales, sélectionnez une petite taille de machine virtuelle telle que A1 Standard. 

Créez-lui une [adresse IP publique]. Placez-le sur le même réseau virtuel que les autres machines virtuelles, mais sur un sous-réseau de gestion distinct.

N’autorisez pas l’accès RDP à partir de l’Internet public aux machines virtuelles qui exécutent la charge de travail d’application. Au lieu de cela, tout l’accès RDP à ces machines virtuelles doit transiter par le serveur de rebond. Un administrateur se connecte au serveur de rebond, puis se connecte à l’autre machine virtuelle à partir du serveur de rebond. Le serveur de rebond autorise le trafic SSH à partir d’Internet, mais uniquement à partir d’adresses IP connues et sûres.

Pour sécuriser le serveur de rebond, créez un groupe de sécurité réseau et appliquez-le au sous-réseau du serveur de rebond. Ajoutez une règle de groupe de sécurité réseau qui autorise les connexions RDP provenant uniquement d’un ensemble sûr d’adresses IP publiques. Le groupe de sécurité réseau peut être attaché au sous-réseau ou à la carte réseau du serveur de rebond. Dans le cas présent, nous vous recommandons de l’attacher à la carte réseau pour que le trafic RDP soit autorisé uniquement sur le serveur de rebond, même si vous ajoutez d’autres machines virtuelles au même sous-réseau.

Configurez les groupes de sécurité réseau pour les autres sous-réseaux de façon à autoriser le trafic RDP provenant du sous-réseau de gestion.

## <a name="availability-considerations"></a>Considérations relatives à la disponibilité

Au niveau Base de données, le fait d’avoir plusieurs machines virtuelles ne se traduit pas automatiquement par une base de données hautement disponible. Pour une base de données relationnelle, vous devez généralement utiliser la réplication et le basculement pour obtenir une haute disponibilité. Pour SQL Server, nous vous recommandons d’utiliser des [groupes de disponibilité AlwaysOn][sql-alwayson]. 

Si vous avez besoin d’une disponibilité plus élevée que celle fournie par le [contrat SLA Azure pour les machines virtuelles][vm-sla], répliquez l’application entre deux régions et utilisez Azure Traffic Manager pour le basculement. Pour plus d’informations, consultez [Exécuter des machines virtuelles Windows dans plusieurs régions à des fins de haute disponibilité][multi-dc].   

## <a name="security-considerations"></a>Considérations relatives à la sécurité

Chiffrez les données sensibles au repos et utilisez [Azure Key Vault][azure-key-vault] pour gérer les clés de chiffrement de base de données. Key Vault peut stocker des clés de chiffrement dans des modules de sécurité matériel (HSM). Pour plus d’informations, consultez [Configurer l’intégration d’Azure Key Vault pour SQL Server sur les machines virtuelles Azure][sql-keyvault]. Nous vous recommandons également de stocker les secrets de l’application, tels que les chaînes de connexion de base de données, dans Key Vault.

Ajoutez une appliance virtuelle réseau (NVA) pour créer un réseau de périmètre (DMZ) entre Internet et le réseau virtuel Azure. NVA est un terme générique décrivant une appliance virtuelle qui peut effectuer des tâches liées au réseau, telles que pare-feu, inspection des paquets, audit et routage personnalisé. Pour plus d’informations, consultez [Implémentation d’une zone DMZ entre Azure et Internet][dmz].

## <a name="scalability-considerations"></a>Considérations relatives à l’extensibilité

Les équilibreurs de charge distribuent le trafic réseau vers les niveaux Web et Business. Mettez à l’échelle votre solution horizontalement en ajoutant de nouvelles instances de machine virtuelle. Notez que vous pouvez mettre à l’échelle les niveaux Web et Business indépendamment, en fonction de la charge. Pour réduire les complications possibles dues à la nécessité de conserver l’affinité du client, les machines virtuelles du niveau Web doivent être sans état. Les machines virtuelles qui hébergent la logique métier doivent également être sans état.

## <a name="manageability-considerations"></a>Considérations relatives à la facilité de gestion

Simplifiez la gestion de l’ensemble du système en utilisant des outils d’administration centralisée tels que [Azure Automation][azure-administration], [Microsoft Operations Management Suite][operations-management-suite], [Chef][chef] ou [Puppet][puppet]. Ces outils peuvent consolider des informations de diagnostic et de contrôle d’intégrité capturées à partir de plusieurs machines virtuelles, afin de fournir une vue d’ensemble du système.

## <a name="deploy-the-solution"></a>Déployer la solution

Un déploiement pour cette architecture de référence est disponible sur [GitHub][github-folder]. 

### <a name="prerequisites"></a>Composants requis

Avant de pouvoir déployer l’architecture de référence sur votre propre abonnement, vous devez effectuer les étapes suivantes.

1. Clonez, dupliquez ou téléchargez le fichier zip pour le dépôt GitHub des [architectures de référence AzureCAT][ref-arch-repo].

2. Vérifiez qu’Azure CLI 2.0 est installé sur votre ordinateur. Pour installer l’interface CLI, suivez les instructions fournies dans [Installer Azure CLI 2.0][azure-cli-2].

3. Installez le package npm des [modules Azure][azbb].

  ```bash
  npm install -g @mspnp/azure-building-blocks
  ```

4. À partir d’une invite de commandes, d’une invite bash ou de l’invite de commandes PowerShell, connectez-vous à votre compte Azure à l’aide de l’une des commandes ci-dessous et suivez les invites.

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a>Déployer la solution à l’aide d’azbb

Pour déployer les machines virtuelles Windows pour une architecture de référence d’application multiniveau, procédez comme suit :

1. Accédez au dossier `virtual-machines\n-tier-windows` pour rechercher le référentiel que vous avez cloné à l’étape 1 des conditions préalables ci-dessus.

2. Le fichier de paramètres spécifie un nom d’utilisateur et un mot de passe administrateur par défaut pour chaque machine virtuelle dans le déploiement. Vous devez les modifier avant de déployer l’architecture de référence. Ouvrez le fichier `n-tier-windows.json` et remplacez chaque champ **adminUsername** et **adminPassword** par vos nouveaux paramètres.
  
  > [!NOTE]
  > Plusieurs scripts s’exécutent pendant ce déploiement, à la fois dans les objets **VirtualMachineExtension** et dans les paramètres **extensions** pour certains des objets **VirtualMachine**. Certains de ces scripts nécessitent le nom d’utilisateur et le mot de passe administrateur que vous venez de modifier. Il est recommandé que vous passiez en revue ces scripts pour vous assurer que vous avez spécifié les bonnes informations d’identification. Le déploiement risque d’échouer si vous n’avez pas spécifié les bonnes informations d’identification.
  > 
  > 

Enregistrez le fichier .

3. Déployez l’architecture de référence à l’aide de l’outil de ligne de commande **azbb**, comme indiqué ci-dessous.

  ```bash
  azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-windows.json --deploy
  ```

Pour plus d’informations sur le déploiement de cet exemple d’architecture de référence à l’aide des blocs de construction Azure, visitez notre [référentiel GitHub][git].


<!-- links -->
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-dc]: multi-region-application.md
[multi-vm]: multi-vm.md
[n-tier]: n-tier.md
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-windows-manage-availability#configure-each-application-tier-into-separate-availability-sets
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-cli-2]: https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-key-vault]: https://azure.microsoft.com/services/key-vault
[hôte bastion]: https://en.wikipedia.org/wiki/Bastion_host
[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-windows
[lb-external-create]: /azure/load-balancer/load-balancer-get-started-internet-portal
[lb-internal-create]: /azure/load-balancer/load-balancer-get-started-ilb-arm-portal
[load-balancer-external]: /azure/load-balancer/load-balancer-internet-overview
[load-balancer-internal]: /azure/load-balancer/load-balancer-internal-overview
[nsg]: /azure/virtual-network/virtual-networks-nsg
[operations-management-suite]: https://www.microsoft.com/server-cloud/operations-management-suite/overview.aspx
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[adresse IP publique]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[puppet]: https://puppetlabs.com/blog/managing-azure-virtual-machines-puppet
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[sql-alwayson]: https://msdn.microsoft.com/library/hh510230.aspx
[sql-alwayson-force-failover]: https://msdn.microsoft.com/library/ff877957.aspx
[sql-alwayson-getting-started]: https://msdn.microsoft.com/library/gg509118.aspx
[sql-alwayson-ilb]: /azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-alwayson-int-listener
[sql-alwayson-listeners]: https://msdn.microsoft.com/library/hh213417.aspx
[sql-alwayson-read-only-routing]: https://technet.microsoft.com/library/hh213417.aspx#ConnectToSecondary
[sql-keyvault]: /azure/virtual-machines/virtual-machines-windows-ps-sql-keyvault
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vnet faq]: /azure/virtual-network/virtual-networks-faq
[wsfc-whats-new]: https://technet.microsoft.com/windows-server-docs/failover-clustering/whats-new-in-failover-clustering
[Nagios]: https://www.nagios.org/
[Zabbix]: http://www.zabbix.com/
[Icinga]: http://www.icinga.org/
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[0]: ./images/n-tier-diagram.png "Architecture multiniveau à l’aide de Microsoft Azure"