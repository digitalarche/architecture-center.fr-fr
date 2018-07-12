---
title: SAP S/4HANA pour machines virtuelles Linux sur Azure
description: Pratiques éprouvées d’exécution de SAP S/4HANA dans un environnement Linux sur Azure avec une haute disponibilité.
author: lbrader
ms.date: 05/11/2018
ms.openlocfilehash: 9635de73ec431e0ac678e4008e0c4835796d47ad
ms.sourcegitcommit: 86d86d71e392550fd65c4f76320d7ecf0b72e1f6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/06/2018
ms.locfileid: "37864502"
---
# <a name="sap-s4hana-for-linux-virtual-machines-on-azure"></a>SAP S/4HANA pour machines virtuelles Linux sur Azure

Cette architecture de référence présente un ensemble de pratiques éprouvées pour l’exécution de S/4HANA dans un environnement à haute disponibilité prenant en charge la reprise après sinistre sur Azure. Cette architecture est déployée avec des tailles de machine virtuelle spécifiques qui peuvent être modifiées en fonction des besoins de votre organisation. 

![](./images/sap-s4hana.png)

*Téléchargez un [fichier Visio][visio-download] de cette architecture.*

> [!NOTE] 
> Le déploiement de cette architecture de référence requiert une licence appropriée des produits SAP et d’autres technologies non Microsoft.

## <a name="architecture"></a>Architecture
 
Cette architecture de référence décrit un système de production de niveau entreprise. En fonction des besoins de votre entreprise, cette configuration peut être réduite à une seule machine virtuelle. En revanche, les composants suivants sont requis :

**Réseau virtuel**. Le service [Réseau virtuel Azure](/azure/virtual-network/virtual-networks-overview) connecte en toute sécurité les ressources Azure entre elles. Dans cette architecture, le réseau virtuel se connecte à un environnement local via une passerelle déployée dans le hub d’une [topologie hub-and-spoke](../hybrid-networking/hub-spoke.md). Le spoke est le réseau virtuel utilisé pour les applications SAP.

**Sous-réseaux**. Le réseau virtuel est subdivisé en [sous-réseaux](/azure/virtual-network/virtual-network-manage-subnet) distincts pour chaque niveau : passerelle, application, base de données et services partagés. 

**Machines virtuelles**. Cette architecture utilise des machines virtuelles exécutant Linux regroupées comme suit pour les couches Application et Base de données :

- **Couche Application**. Inclut le pool Fiori Front-end Server, le pool SAP Web Dispatcher, le pool du serveur d’applications et le cluster des services centraux SAP. Pour la haute disponibilité des services centraux sur les machines virtuelles Azure Linux, un service NFS (Network File Système) à haute disponibilité est requis.
- **Cluster NFS**. Cette architecture utilise un serveur [NFS](/azure/virtual-machines/workloads/sap/high-availability-guide-suse-nfs) exécuté sur un cluster Linux pour stocker les données partagées entre les systèmes SAP. Vous pouvez partager ce cluster centralisé entre plusieurs systèmes SAP. Pour la haute disponibilité du service NFS, vous pouvez utiliser la fonctionnalité High Availability Extension appropriée pour la distribution Linux sélectionnée.
- **SAP HANA**. La couche Base de données utilise au moins deux machines virtuelles Linux dans un cluster pour assurer la haute disponibilité. La réplication de système HANA (HSR) est utilisée pour répliquer du contenu entre les systèmes HANA principal et secondaire. Le clustering Linux est utilisé pour détecter les défaillances du système et faciliter le basculement automatique. Un mécanisme d’isolation basé sur le stockage ou sur le cloud peut être utilisé pour s’assurer que le système défaillant est isolé ou arrêté pour éviter la condition de division (Split-Brain) du cluster.
- **Jumpbox**. Également appelé hôte bastion. Il s’agit d’une machine virtuelle sécurisée sur le réseau, utilisée par les administrateurs pour se connecter aux autres machines virtuelles. Il peut exécuter Windows ou Linux. Le jumpbox Windows facilite la navigation web lors de l’utilisation des outils de gestion HANA Cockpit ou HANA Studio.

**Équilibreurs de charge** : Les équilibreurs de charge SAP intégrés et [Azure Load Balancer](/azure/load-balancer/load-balancer-overview) sont utilisés pour assurer la haute disponibilité. Les instances Azure Load Balancer servent à distribuer le trafic vers les machines virtuelles du sous-réseau de la couche Application.

**Groupes à haute disponibilité**. Les machines virtuelles de tous les pools et clusters (SAP Web Dispatcher, serveurs d’applications SAP, services centraux, NFS et HANA) sont réunies dans des [groupes à haute disponibilité](/azure/virtual-machines/windows/tutorial-availability-sets) distincts, et au moins deux machines virtuelles sont approvisionnées pour chaque rôle. Cela rend les machines virtuelles éligibles pour un [contrat de niveau de service](https://azure.microsoft.com/support/legal/sla/virtual-machines) (SLA) plus élevé. 

**Cartes réseau**. Les [cartes réseau](/azure/virtual-network/virtual-network-network-interface) assurent l’ensemble des communications des machines virtuelles sur un réseau virtuel.

**Groupes de sécurité réseau**. Pour limiter le trafic entrant, sortant et intra-sous-réseau dans le réseau virtuel, vous pouvez utiliser des [groupes de sécurité réseau](/azure/virtual-network/virtual-networks-nsg) (NSG).

**Passerelle**. Une passerelle étend votre réseau local au réseau virtuel Azure. [ExpressRoute](/azure/architecture/reference-architectures/hybrid-networking/expressroute) est le service Azure recommandé pour la création de connexions privées qui ne passent pas par l’Internet public, mais il est également possible d’utiliser une connexion [site à site](/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal). 

**Stockage Azure**. Pour bénéficier d’un stockage permanent du disque dur virtuel d’une machine virtuelle, le service [Stockage Azure](/azure/storage/) est requis. 

## <a name="recommendations"></a>Recommandations

Cette architecture décrit un petit déploiement d’entreprise de niveau production. Votre déploiement sera différent en fonction des besoins de votre entreprise. Utilisez ces recommandations comme point de départ.

### <a name="virtual-machines"></a>Machines virtuelles

Dans les pools et les clusters de serveurs d’applications, ajustez le nombre de machines virtuelles en fonction de vos besoins. Le [guide de planification et implémentation de machines virtuelles Azure](/azure/virtual-machines/workloads/sap/planning-guide) inclut des informations sur l’exécution de SAP NetWeaver sur des machines virtuelles, qui s’appliquent également à SAP S/4HANA.

Pour plus d’informations sur la prise en charge SAP en fonction des types de machine virtuelle Azure et des mesures de débit (SAP), consultez la [Note SAP 1928533](https://launchpad.support.sap.com/#/notes/1928533). 

### <a name="sap-web-dispatcher-pool"></a>Pool SAP Web Dispatcher

Le composant Web Dispatcher sert d’équilibreur de charge pour le trafic SAP entre les serveurs d’applications SAP. Pour assurer la haute disponibilité du composant Web Dispatcher, Azure Load Balancer est utilisé pour implémenter l’installation Web Dispatcher parallèle dans une configuration de tourniquet pour la distribution du trafic HTTP(S) parmi les Web Dispatchers disponibles dans le pool back-end des équilibreurs. 

### <a name="fiori-front-end-server"></a>Fiori Front-end Server

Fiori Front-end Server utilise une [passerelle NetWeaver](https://help.sap.com/doc/saphelp_gateway20sp12/2.0/en-US/76/08828d832e4aa78748e9f82204a864/content.htm?no_cache=true). Pour les déploiements à petite échelle, elle peut être chargée sur le serveur Fiori. Pour les déploiements à grande échelle, vous pouvez déployer un serveur distinct pour la passerelle NetWeaver devant le pool Fiori Front-end Server.

### <a name="application-servers-pool"></a>Pool de serveurs d’applications

Pour gérer les groupes de connexion des serveurs d’applications ABAP, la transaction SMLG est utilisée. Elle s’appuie sur la fonction d’équilibrage de charge au sein du serveur de messages des services centraux pour répartir la charge de travail entre le pool de serveurs d’applications pour le trafic des clients SAP GUI et RFC. La connexion des serveurs d’applications aux services centraux hautement disponibles se fait via le nom du réseau virtuel du cluster. De cette façon, il est inutile de modifier le profil du serveur d’applications pour la connectivité des services centraux après un basculement local. 

### <a name="sap-central-services-cluster"></a>Cluster des services centraux SAP

Vous pouvez déployer les services centraux vers une machine virtuelle unique lorsque la haute disponibilité n’est pas obligatoire. Toutefois, la machine virtuelle unique devient un point de défaillance unique(SPOF) potentiel pour l’environnement SAP. Pour un déploiement de services centraux à haute disponibilité, un cluster NFS à haute disponibilité et un cluster de services centraux à haute disponibilité sont utilisés.

### <a name="nfs-cluster"></a>Cluster NFS

DRBD (Distributed Replicated Block Device) est utilisé pour la réplication entre les nœuds du cluster NFS.

### <a name="availability-sets"></a>Groupes à haute disponibilité

Les groupes à haute disponibilité répartissent les serveurs dans des groupes d’infrastructure physique et de mise à jour différents pour améliorer la disponibilité des services. Placez les machines virtuelles qui présentent le même rôle dans un groupe de disponibilité afin d’éviter les temps d’arrêt causés par la maintenance de l’infrastructure Azure et de respecter les [contrats de niveau de service](https://azure.microsoft.com/support/legal/sla/virtual-machines). Il est recommandé de placer au moins deux machines virtuelles dans un groupe à haute disponibilité.

Toutes les machines virtuelles d’un groupe doivent présenter le même rôle. N’associez pas de serveurs remplissant des rôles différents dans le même groupe à haute disponibilité. Par exemple, ne placez pas un nœud ASCS dans le même groupe de disponibilité que le serveur d’applications.

### <a name="nics"></a>Cartes réseau

Les paysages SAP locaux traditionnels implémentent plusieurs cartes réseau par machine pour séparer le trafic administratif du trafic opérationnel. Sur Azure, le réseau virtuel est un réseau à définition logicielle qui envoie tout le trafic via la même structure réseau. Par conséquent, il est inutile d’utiliser plusieurs cartes réseau. Toutefois, si votre organisation a besoin de séparer le trafic, vous pouvez déployer plusieurs cartes réseau par machine virtuelle, connecter chaque carte réseau à un sous-réseau différent, puis utiliser des NSG pour appliquer des stratégies de contrôle d’accès distinctes.

### <a name="subnets-and-nsgs"></a>Sous-réseaux et NSG

Cette architecture divise l’espace d’adressage du réseau virtuel en sous-réseaux. Chaque sous-réseau peut être associé à un NSG qui définit les stratégies d’accès pour ce sous-réseau. Placez les serveurs d’applications dans un sous-réseau distinct afin de pouvoir les sécuriser plus facilement en gérant les stratégies de sécurité du sous-réseau plutôt que les serveurs individuels.

Quand un NSG est associé à un sous-réseau, il s’applique ensuite à tous les serveurs au sein de ce sous-réseau. Pour plus d’informations sur l’utilisation de NSG pour un contrôle précis des serveurs d’un sous-réseau, consultez [Filtrer le trafic réseau avec les groupes de sécurité réseau](https://azure.microsoft.com/blog/multiple-vm-nics-and-network-virtual-appliances-in-azure/).

Consultez également [Planification et conception de la passerelle VPN](/azure/vpn-gateway/vpn-gateway-plan-design).

### <a name="load-balancers"></a>Équilibreurs de charge

[SAP Web Dispatcher](https://help.sap.com/doc/saphelp_nw73ehp1/7.31.19/en-US/48/8fe37933114e6fe10000000a421937/frameset.htm) gère l’équilibrage de charge du trafic HTTP(S), y compris les applications de style Fiori, à destination d’un pool de serveurs d’applications SAP. 

Pour le trafic en provenance de clients SAP GUI se connectant à un serveur SAP via DIAG ou RFC (Remote Function Call), le serveur de messages des services centraux équilibre la charge par l’intermédiaire des [groupes de connexion](https://wiki.scn.sap.com/wiki/display/SI/ABAP+Logon+Group+based+Load+Balancing) des serveurs d’applications. Par conséquent, aucun équilibreur de charge supplémentaire n’est nécessaire. 

### <a name="azure-storage"></a>Stockage Azure

Nous recommandons l’utilisation du Stockage Premium Azure pour les machines virtuelles du serveur de bases de données. Le stockage Premium offre une latence de lecture/écriture homogène. Pour des informations sur l’utilisation du stockage Premium pour les disques de système d’exploitation et de données d’une machine virtuelle à instance unique, consultez [SLA pour Machines virtuelles](https://azure.microsoft.com/support/legal/sla/virtual-machines/). 

Pour tous les systèmes SAP de production, nous vous recommandons d’utiliser [Azure Managed Disks](/azure/storage/storage-managed-disks-overview) Premium. Des disques managés sont utilisés pour gérer les fichiers VHD pour les disques, pour des questions de fiabilité. Ils assurent également l’isolement des disques des machines virtuelles au sein d’un groupe à haute disponibilité afin d’éviter les points de défaillance uniques.

Pour les serveurs d’applications SAP, y compris les machines virtuelles des services centraux, vous pouvez utiliser le stockage Azure standard afin de réduire les coûts. En effet, l’exécution des applications s’effectue en mémoire et utilise les disques uniquement pour la journalisation. Toutefois, pour le moment le stockage standard est certifié uniquement pour le stockage non managé. Dans la mesure où les serveurs d’applications n’hébergent pas de données, vous pouvez également utiliser les disques P4 et P6 de moins grande capacité pour réduire le coût.

Pour le magasin de données de sauvegarde, nous vous recommandons d’utiliser [le stockage de niveau d’accès froid et/ou le stockage de niveau d’accès archive](/azure/storage/storage-blob-storage-tiers). Ces niveaux de stockage sont des moyens économiques de stocker les données de longue durée moins souvent sollicitées.

## <a name="performance-considerations"></a>Considérations relatives aux performances

Les serveurs d’applications SAP communiquent constamment avec les serveurs de bases de données. Pour les machines virtuelles de base de données HANA, vous pouvez activer [l’accélérateur d’écriture](/azure/virtual-machines/linux/how-to-enable-write-accelerator) pour améliorer la latence d’écriture du journal. Pour optimiser les communications entre les serveurs, utilisez le [réseau accéléré](https://azure.microsoft.com/blog/linux-and-windows-networking-performance-enhancements-accelerated-networking/). Notez que ces accélérateurs sont disponibles uniquement pour certaines séries de machines virtuelles.

Pour obtenir un débit de bande passante de disque et d’ES/S élevé, les pratiques courantes en termes [d’optimisation des performances](/azure/virtual-machines/linux/premium-storage-performance) du volume de stockage s’appliquent à la disposition du stockage Azure. Par exemple, la combinaison de plusieurs disques pour créer un volume de disque agrégé par bandes améliore les performances d’E/S. L’activation du cache de lecture sur un contenu de stockage qui change rarement améliore la vitesse de récupération des données. Pour plus d’informations sur les exigences en termes de performances, consultez [SAP note 1943937 - Hardware Configuration Check Tool](https://launchpad.support.sap.com/#/notes/1943937) (Note SAP 1943937 - Outil de vérification de la configuration matérielle) (compte SAP Service Marketplace requis pour l’accès).

## <a name="scalability-considerations"></a>Considérations relatives à l’extensibilité

Au niveau de la couche de l’application SAP, Azure propose un large éventail de tailles de machines virtuelles pour la montée en puissance ou la montée en charge. Consultez la [Note SAP 1928533](https://launchpad.support.sap.com/#/notes/1928533) - SAP Applications on Azure: Supported Products and Azure VM types (Applications SAP sur Azure : produits et types de machines virtuelles pris en charge (compte SAP Service Marketplace pour l’accès). À mesure que nous certifions des types de machines virtuelles supplémentaires, vous pouvez augmenter ou diminuer la taille des instances avec le même déploiement cloud. 

Dans la couche Base de données, cette architecture exécute HANA sur des machines virtuelles. Si votre charge d travail dépasse la taille maximale de machine virtuelle, Microsoft propose également les [Grandes instances Azure](/azure/virtual-machines/workloads/sap/hana-overview-architecture) pour SAP HANA. Ces serveurs physiques sont colocalisés dans le centre de données certifié Microsoft Azure et à ce jour, fournissent jusqu'à 20 To de capacité de mémoire pour une instance unique. Une configuration à plusieurs nœuds est également possible avec une capacité totale de mémoire pouvant atteindre 60 To.

## <a name="availability-considerations"></a>Considérations relatives à la disponibilité

La redondance des ressources constitue le thème général dans les solutions d’infrastructure à haute disponibilité. Pour les entreprises qui présentent un SLA moins strict, les machines virtuelles Azure à instance unique offrent un SLA garantissant un certain temps de disponibilité. Pour plus d’informations, consultez [Contrats de niveau de service Azure](https://azure.microsoft.com/support/legal/sla/).

Dans cette installation distribuée de l’application SAP, l’installation de base est répliquée pour assurer une haute disponibilité. Pour chaque couche de l’architecture, la conception mise en œuvre à des fins de haute disponibilité varie. 

### <a name="application-tier"></a>Couche Application

- Web Dispatcher. La haute disponibilité est assurée grâce aux instances Web Dispatcher redondantes. Consultez [SAP Web Dispatcher](https://help.sap.com/doc/saphelp_nw70ehp2/7.02.16/en-us/48/8fe37933114e6fe10000000a421937/frameset.htm) dans la documentation SAP.
- Serveurs Fiori. La haute disponibilité est obtenue en équilibrant la charge du trafic au sein d’un pool de serveurs.
- Services centraux. Pour la haute disponibilité des services centraux sur les machines virtuelles Azure Linux, le serveur High-Availability Extension de la distribution Linux sélectionnées est utilisé et le cluster NFS à haute disponibilité héberge le stockage DRBD.
- Serveurs d’applications. La haute disponibilité est obtenue en équilibrant la charge du trafic au sein d’un pool de serveurs d’applications.

### <a name="database-tier"></a>Couche base de données

Cette architecture de référence représente un système de base de données SAP HANA à haute disponibilité composée de deux machines virtuelles Azure. La fonctionnalité de réplication du système natif de la couche Base de données assure un basculement manuel ou automatique entre les nœuds répliqués :

- Pour le basculement manuel, déployez plusieurs instances HAN et utilisez la Réplication de système HANA (HSR).
- Pour le basculement automatique, utilisez les fonctionnalités HSR et Linux HAE (High Availability Extension) pour votre distribution Linux. Linux HAE fournit des services de cluster aux ressources HANA, détectant les événements d’échec et orchestrant le basculement des services errants vers le nœud sain. 

Consultez [Certifications et configurations SAP en cours sur Microsoft Azure](/azure/virtual-machines/workloads/sap/sap-certifications).

### <a name="disaster-recovery-considerations"></a>Considérations relatives à la récupération d’urgence
Chaque couche utilise une stratégie différente pour assurer une protection par récupération d’urgence.

- **Couche Serveurs d’applications**. Les serveurs d’applications SAP ne contiennent pas de données d’entreprise. Sur Azure, une stratégie de récupération d’urgence simple consiste à créer des serveurs d’applications SAP dans la région secondaire, puis à les arrêter. En cas de modification de la configuration ou de mise à jour du noyau sur le serveur d’applications principal, les mêmes modifications doivent être appliquées aux machines virtuelles dans la région secondaire. Par exemple, copiez les exécutables du noyau SAP vers les machines virtuelles de récupération d’urgence. Pour la réplication automatique des serveurs d’applications vers une région secondaire, [Azure Site Recovery](/azure/site-recovery/site-recovery-overview) est la solution recommandée. Au moment d’écrire cet article, ASR ne prend pas encore en charge la réplication des paramètres de configuration du réseau accéléré dans les machines virtuelles Azure.

- **Services centraux**. Ce composant de la pile d’applications SAP ne conserve également aucune donnée d’entreprise. Vous pouvez créer une machine virtuelle dans la région secondaire pour exécuter le rôle Services centraux. Le seul contenu du nœud Services centraux principal à synchroniser est le contenu de partage /sapmnt. En outre, en cas de modification de la configuration ou de mise à jour du noyau sur les serveurs principaux des services centraux, celles-ci doivent être répétées sur la machine virtuelle dans la région secondaire exécutant des services centraux. Pour synchroniser les deux serveurs, vous pouvez utiliser Azure Site Recovery pour répliquer les nœuds de cluster ou simplement utiliser une copie planifiée régulièrement pour copier /sapmnt du côté de la récupération d’urgence. Pour plus d’informations sur le processus de génération, de copie et de basculement du test, téléchargez [SAP NetWeaver: Building a Hyper-V and Microsoft Azure–based Disaster Recovery Solution](http://download.microsoft.com/download/9/5/6/956FEDC3-702D-4EFB-A7D3-2DB7505566B6/SAP%20NetWeaver%20-%20Building%20an%20Azure%20based%20Disaster%20Recovery%20Solution%20V1_5%20.docx) (SAP NetWeaver : créer une solution de récupération d’urgence basée sur Hyper-V et Microsoft Azure) et reportez-vous à la section 4.3 « SAP SPOF layer (ASCS) ». Ce document s’applique à NetWeaver exécuté sur Windows, mais vous pouvez créer la configuration équivalente pour Linux. Pour les services centraux, utilisez [Azure Site Recovery](/en-us/azure/site-recovery/site-recovery-overview) pour répliquer les nœuds de cluster et le stockage. Pour Linux, créez un géo-cluster à trois nœuds à l’aide d’une fonctionnalité High Availability Extension. 

- **Couche Base de données SAP**. Utilisez HSR pour la réplication prise en charge par HANA. Outre une installation locale haute disponibilité à deux nœuds, HSR prend en charge la réplication à plusieurs niveaux, où un troisième nœud d’une région Azure distincte agit comme une entité étrangère, extérieure au cluster, et s’enregistre sur le réplica secondaire de la paire HSR en cluster en tant que cible de réplication. Il en résulte une configuration en chaîne de la réplication. Le basculement vers le nœud de récupération d’urgence est un processus manuel.

Pour utiliser Azure Site Recovery pour générer automatiquement un site de production entièrement répliqué de votre déploiement original, vous devez exécuter [des scripts de déploiement](/azure/site-recovery/site-recovery-runbook-automation) personnalisés. Site Recovery déploie d’abord les machines virtuelles dans des groupes à haute disponibilité, puis exécute des scripts pour ajouter des ressources telles que les équilibreurs de charge. 

## <a name="manageability-considerations"></a>Considérations relatives à la facilité de gestion

SAP HANA comporte une fonctionnalité de sauvegarde qui utilise l’infrastructure Azure sous-jacente. Lors de la sauvegarde de la base de données SAP HANA en cours d’exécution sur des machines virtuelles, les captures instantanées de SAP HANA et du stockage Azure sont utilisées pour garantir la cohérence des fichiers de sauvegarde. Pour plus d’informations, consultez le [guide de sauvegarde pour SAP HANA sur les machines virtuelles Azure](/azure/virtual-machines/workloads/sap/sap-hana-backup-guide) et les [FAQ sur le service de sauvegarde Azure](/azure/backup/backup-azure-backup-faq). Seuls les déploiements HANA à conteneur unique prennent en charge l’instantané de stockage Azure.

### <a name="identity-management"></a>Gestion des identités

Contrôlez l’accès aux ressources en utilisant le système centralisé de gestion des identités à tous les niveaux :

- Fournissez l’accès aux ressources Azure via [le contrôle d’accès en fonction du rôle](/azure/active-directory/role-based-access-control-what-is) (RBAC). 
- Accordez l’accès aux machines virtuelles Azure via LDAP, Azure Active Directory, Kerberos ou un autre système. 
- Prenez en charge l’accès dans les applications elles-mêmes via les services fournis par SAP ou utilisez [OAuth 2.0 et Azure Active Directory](/azure/active-directory/develop/active-directory-protocols-oauth-code). 

### <a name="monitoring"></a>Surveillance

Azure propose plusieurs fonctions de [surveillance et de diagnostic](/azure/architecture/best-practices/monitoring) de l’infrastructure globale. En outre, l’analyse avancée des machines virtuelles Azure (Windows ou Linux) est gérée par Azure Operations Management Suite (OMS). 

Pour assurer une analyse basée sur SAP des ressources et des performances des services de l’infrastructure SAP, l’extension d’[analyse Azure améliorée pour SAP](/azure/virtual-machines/workloads/sap/deployment-guide#d98edcd3-f2a1-49f7-b26a-07448ceb60ca) est utilisée. Cette extension transmet les statistiques d’analyse Azure à l’application SAP pour les fonctions DBA Cockpit et de surveillance du système d’exploitation. L’analyse avancée SAP est une condition préalable obligatoire pour l’exécution de SAP sur Azure. Pour plus d’informations, consultez la [Note SAP 2191498](https://launchpad.support.sap.com/#/notes/2191498) - SAP on Linux with Azure: Enhanced Monitoring (SAP sur Linux avec Azure : analyse avancée).

## <a name="security-considerations"></a>Considérations relatives à la sécurité

SAP possède son propre moteur de gestion des utilisateurs (UME) pour contrôler l’accès et l’autorisation en fonction du rôle dans l’application SAP. Pour plus d’informations, consultez [SAP HANA Security—An Overview](https://archive.sap.com/documents/docs/DOC-62943) (compte SAP Service Marketplace requis pour l’accès.)

Pour une sécurité réseau supplémentaire, envisagez d’implémenter une [zone DMZ réseau](/azure/architecture/reference-architectures/dmz/secure-vnet-hybrid), qui utilise une appliance virtuelle réseau pour créer un pare-feu devant le sous-réseau des pools Web Dispatcher et Fiori Front-End Server.

Les données sont chiffrées en transit et au repos à des fins de sécurité de l’infrastructure. La section « Considérations de sécurité » de [SAP NetWeaver on Azure Virtual Machines (VMs) – Planning and Implementation Guide](/azure/virtual-machines/workloads/sap/planning-guide) (SAP NetWeaver sur les machines virtuelles Azure - Guide de planification et d’implémentation) commence à traiter la sécurité réseau et s’applique à S/4HANA. Le guide spécifie également les ports réseau, que vous devez ouvrir sur les pare-feu pour autoriser la communication de l’application. 

[Azure Disk Encryption](/azure/security/azure-security-disk-encryption) permet de chiffrer des disques de machines virtuelles Linux IaaS. Ce service exploite la fonctionnalité DM-Crypt de Linux pour assurer le chiffrement de volume du système d’exploitation et des disques de données. La solution fonctionne également avec Azure Key Vault, ce qui vous permet de contrôler et de gérer les clés et les secrets de chiffrement de disque dans votre abonnement Key Vault. Les données sur les disques de vos machines virtuelles sont chiffrées au repos dans votre stockage Azure.

Pour le chiffrement des données SAP HANA au repos, nous recommandons l’utilisation de la technologie de chiffrement native SAP HANA. 

> [!NOTE]
> N’utilisez pas le chiffrement des données au repos HANA avec Azure Disk Encryption sur le même serveur. Pour HANA, utilisez uniquement le chiffrement des données HANA.

## <a name="communities"></a>Communautés

Les communautés peuvent répondre aux questions et vous aider à paramétrer un déploiement réussi. Tenez compte des éléments suivants :

- [Blog Running SAP Applications on the Microsoft Platform](https://blogs.msdn.microsoft.com/saponsqlserver/2017/05/04/sap-on-azure-general-update-for-customers-partners-april-2017/) (Exécution d’applications SAP sur la plateforme Microsoft)
- [Support de la communauté Azure](https://azure.microsoft.com/support/community/)
- [Communauté SAP](https://www.sap.com/community.html)
- [Dépassement de capacité de la pile](https://stackoverflow.com/tags/sap/)

[visio-download]: https://archcenter.blob.core.windows.net/cdn/sap-reference-architectures.vsdx
