---
title: Déployer SAP NetWeaver et SAP HANA sur Azure
description: Pratiques éprouvées d’exécution de SAP HANA dans un environnement à haute disponibilité dans Azure.
author: njray
ms.date: 06/29/2017
ms.openlocfilehash: 33171164c59a520a87ef3209c5bb1b208377221c
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/30/2018
---
# <a name="deploy-sap-netweaver-and-sap-hana-on-azure"></a>Déployer SAP NetWeaver et SAP HANA sur Azure

Cette architecture de référence présente un ensemble de pratiques éprouvées pour l’exécution de SAP HANA dans un environnement à haute disponibilité dans Azure. [**Déployez cette solution**.](#deploy-the-solution)

![0][0]

*Téléchargez un [fichier Visio][visio-download] de cette architecture.*

> [!NOTE]
> Le déploiement de cette architecture de référence requiert une licence appropriée des produits SAP et d’autres technologies non Microsoft. Pour plus d’informations sur le partenariat entre Microsoft et SAP, consultez [SAP HANA sur Azure][sap-hana-on-azure].

## <a name="architecture"></a>Architecture

L’architecture est constituée des composants suivants.

- **Réseau virtuel**. Un réseau virtuel est une représentation d’un réseau isolé logiquement dans Azure. Toutes les machines virtuelles de cette architecture de référence sont déployées sur le même réseau virtuel. Le réseau virtuel est en outre subdivisé en sous-réseaux. Créez un sous-réseau distinct pour chaque couche, y compris l’application (SAP NetWeaver), la base de données (SAP HANA), la gestion (le jumpbox) et Active Directory.

- **Machines virtuelles**. Les machines virtuelles de cette architecture sont regroupées en plusieurs couches distinctes.

    - **SAP NetWeaver**. Inclut SAP ASCS, SAP Web Dispatcher et les serveurs d’applications SAP. 
    
    - **SAP HANA**. Cette architecture de référence utilise SAP HANA pour la couche de base de données, en cours d’exécution sur une seule instance [GS5][vm-sizes-mem]. SAP HANA est certifié pour les charges de travail OLAP de production sur GS5 ou [SAP HANA sur des grandes instances Azure][azure-large-instances]. Cette architecture de référence est destinée aux machines virtuelles Azure de série G et M. Pour plus d’informations sur SAP HANA sur les grandes instances Azure de grande taille, consultez [Vue d’ensemble et architecture de SAP HANA (grandes instances) sur Azure][azure-large-instances].
   
    - **Jumpbox**. Également appelé hôte bastion. Il s’agit d’une machine virtuelle sécurisée sur le réseau, utilisée par les administrateurs pour se connecter aux autres machines virtuelles. 
     
    - **Contrôleurs de domaine Windows Server Active Directory (AD).** Les contrôleurs de domaine sont utilisés pour configurer le cluster de basculement Windows Server (voir ci-dessous).
 
- **Groupes à haute disponibilité**. Placez les machines virtuelles pour le SAP Web Dispatcher, le serveur d’applications SAP et les rôles SAP ACSC dans des groupes à haute disponibilité distincts et configurez au moins deux machines virtuelles pour chaque rôle. Cela rend les machines virtuelles éligibles à un contrat de niveau de service (SLA) plus élevé.
    
- **Cartes réseau**. Les machines virtuelles qui s’exécutent sur SAP NetWeaver et SAP HANA nécessitent deux cartes réseau. Chaque carte réseau est affectée à un sous-réseau différent, pour séparer les différents types de trafic. Consultez les [recommandations](#recommendations) ci-dessous pour plus d’informations.

- **Cluster de basculement Windows Server**. Les machines virtuelles qui exécutent SAP ACSC sont configurées comme un cluster de basculement pour une haute disponibilité. Pour prendre en charge le cluster de basculement SIOS DataKeeper Cluster Edition exécute la fonction de volume partagé de cluster (CSV) par la réplication de disques indépendants détenus par les nœuds de cluster. Pour plus de détails, consultez [Running SAP applications on the Microsoft platform][running-sap] (Exécution d’applications SAP sur la plateforme Microsoft).
    
- **Équilibreurs de charge.** Deux instances [Azure Load Balancer] [ azure-lb] sont utilisées. La première, illustrée à gauche du diagramme, répartit le trafic sur les machines virtuelles SAP Web Dispatcher. Cette configuration implémente l’option de répartiteur web parallèle décrite dans [High Availability of the SAP Web Dispatcher][sap-dispatcher-ha] (Haute disponibilité du SAP Web Dispatcher). Le deuxième équilibreur de charge présenté sur la droite permet le basculement au sein du cluster de basculement Windows Server, en dirigeant les connexions entrantes vers le nœud actif/sain.

- **Passerelle VPN.** La passerelle VPN étend votre réseau local sur le réseau virtuel Azure. Vous pouvez également utiliser ExpressRoute, qui utilise une connexion privée dédiée qui ne passe pas par l’Internet public. L’exemple de solution ne déploie pas la passerelle. Pour plus d’informations, consultez la section [Connecter un réseau local à Azure][hybrid-networking].

## <a name="recommendations"></a>Recommandations

Vos exigences peuvent différer de celles de l’architecture décrite ici. Utilisez ces recommandations comme point de départ.

### <a name="load-balancers"></a>Équilibreurs de charge

[SAP Web Dispatcher][sap-dispatcher] traite l’équilibrage du trafic HTTP(S) sur les serveurs à double pile (ABAP et Java). SAP a préconisé des serveurs d’applications à pile unique pendant des années, c’est pourquoi peu d’applications s’exécutent sur un modèle de déploiement à double pile de nos jours. L’équilibreur de charge Azure indiqué dans le diagramme d’architecture implémente le cluster à haute disponibilité pour SAP Web Dispatcher.

L’équilibrage de charge du trafic sur les serveurs d’applications est géré dans SAP. Pour le trafic à partir de clients SAPGUI se connectant à un serveur SAP via DIAG et RFC, le serveur de messagerie SCS équilibre la charge en créant des [groupes de connexion][logon-groups] du serveur d’applications SAP. 

SMLG est une transaction SAP ABAP permettant de gérer la fonction d’équilibrage de la charge de connexion des Services centraux SAP. Le pool principal du groupe de connexion dispose de plusieurs serveurs d’applications. Les clients accédant aux services de cluster ASC se connectent à l’équilibreur de charge Azure via une adresse IP de serveur frontal. Le nom du réseau virtuel du cluster ASC possède également une adresse IP. Si vous le souhaitez, cette adresse peut être associée à une adresse IP supplémentaire sur l’équilibreur de charge Azure, afin que le cluster puisse être géré à distance.  

### <a name="nics"></a>Cartes réseau

Les fonctions de gestion du paysage SAP nécessitent la séparation du trafic du serveur sur différentes cartes réseau. Par exemple, les données d’entreprise doivent être séparées du trafic administratif et de sauvegarde. L’affectation de plusieurs cartes réseau à différents sous-réseaux permet cette séparation des données. Pour plus d’informations, consultez « Réseau » dans le document PDF [Building High Availability for SAP NetWeaver and SAP HANA] [ sap-ha] (Créer la haute disponibilité pour SAP NetWeaver et SAP HANA).

Attribuer la carte réseau d’administration au sous-réseau de gestion et affectez la carte réseau de communication des données à un sous-réseau distinct. Pour obtenir des informations relatives à la configuration, consultez [Créer et gérer une machine virtuelle Windows équipée de plusieurs cartes d’interface réseau][multiple-vm-nics].

### <a name="azure-storage"></a>Stockage Azure

Avec toutes les machines virtuelles du serveur de base de données, nous recommandons l’utilisation du Stockage Premium Azure pour la latence en lecture/écriture. Pour les serveurs d’applications SAP, y compris les machines virtuelles (A)SCS, vous pouvez utiliser le stockage standard Azure, car l’exécution d’applications s’effectue en mémoire et utilise les disques uniquement pour la connexion.

Pour une meilleure fiabilité, nous recommandons l’utilisation d’[Azure Managed Disks][managed-disks]. Les disques gérés assurent que les disques des machines virtuelles au sein d’un groupe à haute disponibilité sont isolés afin d’éviter des points de défaillance uniques.

> [!NOTE]
> Actuellement, le modèle Resource Manager de cette architecture de référence n’utilise pas de disques gérés. Nous avons l’intention de mettre à jour le modèle pour utiliser des managed disks.

Pour obtenir un débit de bande passante de disque et d’ES/S élevé, les pratiques courantes en termes d’optimisation des performances du volume de stockage s’appliquent à la disposition du stockage Azure. Par exemple, l’entrelacement de plusieurs disques pour créer un plus grand volume de disque améliore les performances d’E/S. L’activation du cache de lecture sur un contenu de stockage qui change rarement améliore la vitesse de récupération des données. Pour plus d’informations sur les exigences en termes de performances, consultez [SAP note 1943937 - Hardware Configuration Check Tool][sap-1943937] (Note SAP 1943937 - Outil de vérification de la configuration matérielle).

Pour le magasin de données de sauvegarde, nous recommandons l’utilisation du [niveau de stockage froid] [ cool-blob-storage] du stockage d’objets Blob Azure. Le niveau de stockage froid est un moyen économique de stocker les données à long terme moins souvent sollicitées.

## <a name="scalability-considerations"></a>Considérations relatives à l’extensibilité

Au niveau de la couche de l’application SAP, Azure propose un large éventail de tailles de machines virtuelles pour une augmentation de la taille des instances. Consultez [Note SAP 1928533 - Applications SAP sur Azure : produits et types de machines virtuelles pris en charge pour obtenir une liste exhaustive][sap-1928533]. Augmentez la taille des instances en ajoutant davantage de machines virtuelles au groupe à haute disponibilité.

Pour SAP HANA sur des machines virtuelles avec des applications SAP OLTP et OLAP, la taille de machine virtuelle certifiée par SAP est GS5 avec une seule instance de machine virtuelle. Pour des charges de travail supérieures, Microsoft propose également de [grandes instances Azure] [ azure-large-instances] pour SAP HANA sur les serveurs physiques colocalisés dans un centre de données certifié Microsoft Azure, qui fournit actuellement jusqu’à 4 To de capacité de mémoire pour une seule instance unique. Une configuration à plusieurs nœuds est également possible avec une capacité totale de mémoire pouvant atteindre 32 To.

## <a name="availability-considerations"></a>Considérations relatives à la disponibilité

Dans cette installation distribuée de l’application SAP sur une base de données centralisée, l’installation de base est répliquée pour une haute disponibilité. Pour chaque couche de l’architecture, la conception de la haute disponibilité varie :

- **Répartiteur web** La haute disponibilité est assurée grâce aux instances SAP Web Dispatcher redondantes avec le trafic de l’application SAP. Consultez [SAP Web Dispatcher] [ swd] dans la Documentation SAP.

- **ASC.** Pour la haute disponibilité d’ASC sur les machines virtuelles Windows Azure, le Clustering de basculement Windows Server est utilisé avec SIOS DataKeeper pour implémenter le volume partagé de cluster. Pour obtenir des informations relatives à l’implémentation, consultez [Clustering SAP ASCS on Azure][clustering] (Clustering de SAP ASCS sur Azure).

- **Serveurs d’applications.** La haute disponibilité est obtenue en équilibrant la charge du trafic au sein d’un pool de serveurs d’applications.

- **Couche de la base de données.** Cette architecture de référence déploie une seule instance de base de données SAP HANA. Pour la haute disponibilité, déployez plusieurs instances et utilisez la Réplication de système HANA pour mettre en œuvre le basculement manuel. Pour activer le basculement automatique, une extension à haute disponibilité pour la distribution de Linux spécifique est requise.

### <a name="disaster-recovery-considerations"></a>Considérations relatives à la récupération d’urgence

Chaque couche utilise une stratégie différente pour assurer une protection par récupération d’urgence.

- **Serveurs d’applications.** Les serveurs d’applications SAP ne contiennent pas de données d’entreprise. Sur Azure, une stratégie de récupération d’urgence simple consiste à créer des serveurs d’applications SAP dans une autre région. En cas de modification de la configuration ou de mise à jour du noyau sur le serveur d’applications principal, les mêmes modifications doivent être copiées sur des machines virtuelles dans la zone de récupération d’urgence. Par exemple, les exécutables du noyau copié sur les machines virtuelles de récupération d’urgence.

- **Services centraux SAP.** Ce composant de la pile d’applications SAP ne conserve également aucune donnée d’entreprise. Vous pouvez créer une machine virtuelle dans la zone de récupération d’urgence pour exécuter le rôle SCS. Le seul contenu du nœud SCS principal à synchroniser est le contenu de partage **/sapmnt**. En outre, en cas de modification de la configuration ou de mise à jour du noyau sur les serveurs principaux SCS, celles-ci doivent être répétées sur le SCS de récupération d’urgence. Pour synchroniser les deux serveurs, il suffit d’utiliser une copie planifiée régulièrement pour copier **/sapmnt** du côté de la récupération d’urgence. Pour plus d’informations sur le processus de génération, de copie et de basculement du test, téléchargez [SAP NetWeaver: Building a Hyper-V and Microsoft Azure–based Disaster Recovery Solution][sap-netweaver-dr] (SAP NetWeaver : créer une solution de récupération d’urgence basée sur Hyper-V et Microsoft Azure) et reportez-vous à la section 4.3. Couche SAP SPOF (ASCS).

- **Couche de la base de données.** Utilisez des solutions de réplication prises en charge par HANA, telles que la Réplication de système HANA ou la réplication du stockage. 

## <a name="manageability-considerations"></a>Considérations relatives à la facilité de gestion

SAP HANA comporte une fonctionnalité de sauvegarde qui utilise l’infrastructure Azure sous-jacente. Lors de la sauvegarde de la base de données SAP HANA en cours d’exécution sur des machines virtuelles, les captures instantanées de SAP HANA et du stockage Azure sont utilisées pour garantir la cohérence des fichiers de sauvegarde. Pour plus d’informations, consultez le [guide de sauvegarde pour SAP HANA sur les machines virtuelles Azure][hana-backup] et les [FAQ sur le service de sauvegarde Azure][backup-faq].

Azure propose plusieurs fonctions d’[analyse et de diagnostics] [ monitoring] de l’infrastructure globale. En outre, l’analyse avancée des machines virtuelles Azure (Windows ou Linux) est gérée par Azure Operations Management Suite (OMS).

Pour assurer une analyse basée sur SAP des ressources et des performances des services de l’infrastructure SAP, utilisez l’extension d’analyse avancée SAP Azure. Cette extension transmet les statistiques d’analyse Azure à l’application SAP pour les fonctions DBA Cockpit et de surveillance du système d’exploitation. 

## <a name="security-considerations"></a>Considérations relatives à la sécurité

SAP possède son propre moteur de gestion des utilisateurs (UME) pour contrôler l’accès et l’autorisation en fonction du rôle dans l’application SAP. Pour plus d’informations, consultez [SAP HANA Security - An Overview][sap-security] (Sécurité SAP HANA - Présentation). (l’accès requiert un compte SAP Service Marketplace.)

Les données sont sécurisées en transit et au repos à des fins de sécurité de l’infrastructure. La section « Considérations de sécurité » de [SAP NetWeaver on Azure Virtual Machines (VMs) – Planning and Implementation Guide][netweaver-on-azure] (SAP NetWeaver sur les machines virtuelles Azure - Guide de planification et d’implémentation) commence à traiter la sécurité réseau. Le guide spécifie également les ports réseau, que vous devez ouvrir sur les pare-feu pour autoriser la communication de l’application. 

[Azure Disk Encryption][disk-encryption] permet de chiffrer des disques de machines virtuelles Windows et Linux IaaS. Azure Disk Encryption utilise la fonctionnalité BitLocker Windows et la fonctionnalité DM-Crypt de Linux pour fournir le chiffrement de volume du système d’exploitation et des disques de données. La solution fonctionne également avec Azure Key Vault, ce qui vous permet de contrôler et de gérer les clés et les secrets de chiffrement de disque dans votre abonnement Key Vault. Les données sur les disques de vos machines virtuelles sont chiffrées au repos dans votre stockage Azure.

Pour le chiffrement des données SAP HANA au repos, nous recommandons l’utilisation de la technologie de chiffrement native SAP HANA.

> [!NOTE]
> N’utilisez pas le chiffrement des données au repos HANA avec le chiffrement de disque Azure sur le même serveur.

Envisagez d’utiliser des [groupes de sécurité réseau] [ nsg] (NSG) pour limiter le trafic entre les différents sous-réseaux du réseau virtuel.

## <a name="deploy-the-solution"></a>Déployer la solution 

Les scripts de déploiement pour cette architecture de référence sont disponibles sur [GitHub][github].


### <a name="prerequisites"></a>Prérequis


- Vous devez avoir accès au centre de téléchargement de logiciels SAP pour terminer l’installation.
 
- Installez la dernière version d’[Azure PowerShell][azure-ps]. 

- Ce déploiement nécessite 51 cœurs. Avant de procéder au déploiement, vérifiez que votre abonnement dispose d’un quota suffisant pour les cœurs de machine virtuelle. S’il est insuffisant, utilisez le portail Azure pour soumettre une demande de support afin d’obtenir un quota plus élevé.
 
- Ce déploiement utilise une machine virtuelle de la série GS. Vérifiez la disponibilité de la série GS par région [ici][region-availability].

- Pour estimer le coût de ce déploiement, reportez-vous à [Calculatrice de prix Azure][azure-pricing]. 
 
Cette architecture de référence déploie les machines virtuelles suivantes :

| Nom de la ressource | Taille de la machine virtuelle | Objectif  |
|---------------|---------|----------|
| `ra-sapApps-scs-vm1` ... `ra-sapApps-scs-vmN` | DS11v2 | Services centraux SAP |
| `ra-sapApps-vm1` ... `ra-sapApps-vmN` | DS11v2 | Application SAP NetWeaver |
| `ra-sap-wdp-vm1` ... `ra-sap-wdp-vmN` | DS11v2 | SAP Web Dispatcher |
| `ra-sap-data-vm1` | GS5 | Instance de base de données SAP HANA |
| `ra-sap-jumpbox-vm1` | DS1V2 | Serveur de rebond |

Une seule instance SAP HANA est déployée. Pour les machines virtuelles de l’application, le nombre d’instances à déployer est spécifié dans les paramètres du modèle.

### <a name="deploy-sap-infrastructure"></a>Déployer l’infrastructure SAP

Vous pouvez déployer cette architecture de manière incrémentielle ou en une seule fois. La première fois, nous vous recommandons un déploiement incrémentiel, afin que vous puissiez voir ce que fait chaque étape de déploiement. Spécifiez l’incrément à l’aide d’un des paramètres de *mode* suivants.

| Mode           | Résultat                                                                                                            |
|----------------|-----------------------------------------------------|
| infrastructure | Dépliez l’infrastructure réseau dans Azure.        |
| charge de travail       | Déploie les serveurs SAP sur le réseau.             |
| tout            | Déploie tous les déploiements précédents.              |

Pour déployer la solution, procédez comme suit :

1. Téléchargez ou clonez le [Référentiel GitHub][github] sur votre ordinateur local.

2. Ouvrez une fenêtre PowerShell et accédez au dossier `/sap/sap-hana/`.

3. Exécutez l’applet de commande PowerShell suivant. Pour `<subscription id>`, utilisez l’identifiant de votre abonnement Azure. Pour `<location>`, spécifiez une région Azure, telle que `eastus` ou `westus`. Pour `<mode>`, spécifiez un des modes ci-dessus.

    ```powershell
     .\Deploy-ReferenceArchitecture -SubscriptionId <subscription id> -Location <location> -ResourceGroupName <resource group> <mode>
    ```

4.  Lorsque vous y êtes invité, connectez-vous à votre compte Azure. 

Remplir les scripts de déploiement peut prendre plusieurs heures, selon le mode que vous avez sélectionné.

> [!WARNING]
> Les fichiers de paramètres incluent un mot de passe codé en dur (`AweS0me@PW`) dans divers emplacements. Modifiez ces valeurs avant de déployer.
 
### <a name="configure-sap-applications-and-database"></a>Configurer la base de données et les applications SAP

Après avoir déployé l’infrastructure SAP, installez et configurez vos applications SAP et la base de données HANA sur les machines virtuelles comme suit.

> [!NOTE]
> Pour les instructions d’installation SAP, vous devez disposer d’un nom d’utilisateur pour le portail de support SAP et d’un mot de passe pour télécharger les [guides d’installation SAP][sap-guide].

1. Connectez-vous au jumpbox (`ra-sap-jumpbox-vm1`). Vous allez utiliser le jumpbox pour vous connecter aux autres machines virtuelles. 

2.  Pour chaque machine virtuelle nommée `ra-sap-wdp-vm1` ... `ra-sap-wdp-vmN`, connectez-vous à la machine virtuelle, installez et configurez l’instance SAP Web Dispatcher en suivant la procédure décrite dans le wiki portant sur l’[installation du répartiteur web][sap-dispatcher-install].

3.  Connectez-vous à la machine virtuelle nommée `ra-sap-data-vm1`. Installez et configurez l’instance de base de données SAP Hana à l’aide du [Guide de mise à jour et d’installation du serveur SAP HANA][hana-guide].

4. Pour chaque machine virtuelle nommée `ra-sapApps-scs-vm1` ... `ra-sapApps-scs-vmN`, connectez-vous à la machine virtuelle et installez et configurez les Services centraux SAP (SCS) à l’aide des [guides d’installation SAP][sap-guide].

5.  Pour chaque machine virtuelle nommée `ra-sapApps-vm1` ... `ra-sapApps-vmN`, connectez-vous à la machine virtuelle et installez et configurez l’application SAP NetWeaver à l’aide des [guides d’installation SAP][sap-guide].

**_Les contributeurs de cette architecture de référence_**  &mdash; sont Rick Rainey, Ross Sponholtz et Ben Trinh

[azure-large-instances]: /azure/virtual-machines/workloads/sap/hana-overview-architecture
[azure-lb]: /azure/load-balancer/load-balancer-overview
[azure-pricing]: https://azure.microsoft.com/pricing/calculator/
[azure-ps]: /powershell/azure/overview
[backup-faq]: /azure/backup/backup-azure-backup-faq
[clustering]: https://blogs.msdn.microsoft.com/saponsqlserver/2015/05/20/clustering-sap-ascs-instance-using-windows-server-failover-cluster-on-microsoft-azure-with-sios-datakeeper-and-azure-internal-load-balancer/
[cool-blob-storage]: /azure/storage/storage-blob-storage-tiers
[disk-encryption]: /azure/security/azure-security-disk-encryption
[github]: https://github.com/mspnp/reference-architectures/tree/master/sap/sap-hana
[hana-backup]: /azure/virtual-machines/workloads/sap/sap-hana-backup-guide
[hana-guide]: https://help.sap.com/viewer/2c1988d620e04368aa4103bf26f17727/2.0.01/en-US/7eb0167eb35e4e2885415205b8383584.html
[hybrid-networking]: ../hybrid-networking/index.md
[logon-groups]: https://wiki.scn.sap.com/wiki/display/SI/ABAP+Logon+Group+based+Load+Balancing
[managed-disks]: /azure/storage/storage-managed-disks-overview
[monitoring]: /azure/architecture/best-practices/monitoring
[multiple-vm-nics]: /azure/virtual-machines/windows/multiple-nics
[netweaver-on-azure]: /azure/virtual-machines/workloads/sap/planning-guide
[nsg]: /azure/virtual-network/virtual-networks-nsg
[region-availability]: https://azure.microsoft.com/regions/services/
[running-SAP]: https://blogs.msdn.microsoft.com/saponsqlserver/2016/06/07/sap-on-sql-general-update-for-customers-partners-june-2016/
[sap-1943937]: https://launchpad.support.sap.com/#/notes/1943937
[sap-1928533]: https://launchpad.support.sap.com/#/notes/1928533
[sap-dispatcher]: https://help.sap.com/doc/saphelp_nw73ehp1/7.31.19/en-US/48/8fe37933114e6fe10000000a421937/frameset.htm
[sap-dispatcher-ha]: https://help.sap.com/doc/saphelp_nw73ehp1/7.31.19/en-US/48/9a9a6b48c673e8e10000000a42189b/frameset.htm
[sap-dispatcher-install]: https://wiki.scn.sap.com/wiki/display/SI/Web+Dispatcher+Installation
[sap-guide]: https://service.sap.com/instguides
[sap-ha]: https://support.sap.com/content/dam/SAAP/SAP_Activate/AGS_70.pdf
[sap-hana-on-azure]: https://azure.microsoft.com/services/virtual-machines/sap-hana/
[sap-netweaver-dr]: http://download.microsoft.com/download/9/5/6/956FEDC3-702D-4EFB-A7D3-2DB7505566B6/SAP%20NetWeaver%20-%20Building%20an%20Azure%20based%20Disaster%20Recovery%20Solution%20V1_5%20.docx
[sap-security]: https://archive.sap.com/documents/docs/DOC-62943
[visio-download]: https://archcenter.blob.core.windows.net/cdn/SAP-HANA-architecture.vsdx
[vm-sizes-mem]: /azure/virtual-machines/windows/sizes-memory
[swd]: https://help.sap.com/doc/saphelp_nw70ehp2/7.02.16/en-us/48/8fe37933114e6fe10000000a421937/frameset.htm
[0]: ./images/sap-hana.png "Architecture SAP HANA à l’aide de Microsoft Azure"
