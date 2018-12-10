---
title: Exécuter une batterie de serveurs SharePoint Server 2016 hautement disponible dans Azure
titleSuffix: Azure Reference Architectures
description: Architecture recommandée pour déployer une batterie de serveurs SharePoint Server 2016 à haute disponibilité sur Azure.
author: njray
ms.date: 07/26/2018
ms.custom: seodec18
ms.openlocfilehash: 6cc8255f95cb4944ff3ef138ad5edf2e5bbea4b4
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2018
ms.locfileid: "53120099"
---
# <a name="run-a-highly-available-sharepoint-server-2016-farm-in-azure"></a>Exécuter une batterie de serveurs SharePoint Server 2016 hautement disponible dans Azure

Cette architecture de référence présente des pratiques éprouvées pour déployer une batterie de serveurs SharePoint Server 2016 hautement disponible sur Azure à l’aide de la topologie MinRole et de groupes de disponibilité Always On SQL Server. La batterie de serveurs SharePoint est déployée dans un réseau virtuel sécurisé sans aucun point de terminaison ou présence exposé à Internet. [**Déployez cette solution**](#deploy-the-solution).

![Architecture de référence d’une batterie de serveurs SharePoint Server 2016 hautement disponible dans Azure](./images/sharepoint-ha.png)

*Téléchargez un [fichier Visio][visio-download] de cette architecture.*

## <a name="architecture"></a>Architecture

Cette architecture repose sur celle décrite dans l’article [Exécuter des machines virtuelles Windows pour une application multiniveau][windows-n-tier]. Elle déploie une batterie de serveurs SharePoint Server 2016 à haute disponibilité dans un réseau virtuel Azure (VNet). Cette architecture est adaptée à un environnement de test ou de production, une infrastructure hybride de SharePoint avec Office 365, ou comme base pour un scénario de récupération d’urgence.

L’architecture est constituée des composants suivants :

- **Groupe de ressources**. Un [groupe de ressources][resource-group] est un conteneur qui héberge les ressources Azure associées. Un groupe de ressources est utilisé pour les serveurs SharePoint et un autre groupe de ressources est utilisé pour les composants d’infrastructure qui sont indépendants des machines virtuelles, tels que le réseau virtuel et les équilibreurs de charge.

- **Réseau virtuel**. Les ordinateurs virtuels sont déployés dans un réseau virtuel avec un espace d’adressage intranet unique. Le réseau virtuel est en outre subdivisé en sous-réseaux.

- **Machines virtuelles (VM)**. Les machines virtuelles sont déployées dans le réseau virtuel, les adresses IP statiques privées sont affectées à toutes les machines virtuelles. Les adresses IP statiques sont recommandées pour les machines virtuelles exécutant SQL Server et SharePoint Server 2016, afin d’éviter les problèmes liés à la mise en cache des adresses IP et les changements d’adresses après un redémarrage.

- **Groupes à haute disponibilité**. Placez les machines virtuelles pour chaque rôle SharePoint dans différents [groupes à haute disponibilité][availability-set] et configurez au moins deux machines virtuelles (VM) pour chaque rôle. Cela rend les machines virtuelles éligibles à un contrat de niveau de service (SLA) plus élevé.

- **Équilibreur de charge interne**. L’[équilibrage de charge][load-balancer] distribue le trafic de requête SharePoint du réseau local aux serveurs web frontaux de la batterie de serveurs SharePoint.

- **Groupes de sécurité réseau (NSG)**. Pour chaque sous-réseau contenant des ordinateurs virtuels, un [groupe de sécurité réseau][nsg] est créé. Utilisez les groupes de sécurité réseau pour limiter le trafic au sein du réseau virtuel, afin d’isoler les sous-réseaux.

- **Passerelle**. La passerelle fournit une connexion entre votre réseau local et le réseau virtuel Azure. Votre connexion peut utiliser une passerelle ExpressRoute ou VPN de site à site. Pour plus d’informations, consultez la section [Connecter un réseau local à Azure][hybrid-ra].

- **Contrôleurs de domaine Windows Server Active Directory (AD)**. Cette architecture de référence déploie des contrôleurs de domaine Windows Server AD. Ces contrôleurs de domaine s’exécutent dans le réseau virtuel Azure et ont une relation d’approbation avec la forêt locale Windows Server Active Directory. Les requêtes web des clients pour les ressources de la batterie de serveurs SharePoint sont authentifiées sur le réseau virtuel, au lieu d’envoyer le trafic d’authentification via la connexion de passerelle vers le réseau local. Des enregistrements sont créés dans DNS, intranet A ou CNAME, afin que les utilisateurs de l’intranet puissent résoudre le nom de la batterie de serveurs SharePoint à l’adresse IP privée de l’équilibreur de charge interne.

  SharePoint Server 2016 prend également en charge l’utilisation de [Microsoft Azure Active Directory Domain Services](/azure/active-directory-domain-services/). Microsoft Azure AD Domain Services fournit des services de domaine managé, afin que vous n’ayez pas besoin de déployer et de gérer les contrôleurs de domaine dans Azure.

- **Groupe de disponibilité SQL Server Always On**. Pour obtenir une haute disponibilité de la base de données SQL Server, nous vous recommandons les [groupes de disponibilité SQL Server Always On][sql-always-on]. Deux machines virtuelles sont utilisées pour SQL Server. L’une contient le réplica de base de données primaire et l’autre le réplica secondaire.

- **Machine virtuelle du nœud majoritaire**. Cette machine virtuelle permet au cluster de basculement d’établir le quorum. Pour plus d’informations, consultez l’article [Présentation des configurations de quorum dans un cluster de basculement][sql-quorum].

- **Serveurs SharePoint**. Les serveurs SharePoint effectuent le service web frontal, la mise en cache, l’application et la recherche des rôles.

- **Jumpbox**. Également appelée [hôte bastion][bastion-host]. Il s’agit d’une machine virtuelle sécurisée sur le réseau, utilisée par les administrateurs pour se connecter aux autres machines virtuelles. La jumpbox a un groupe de sécurité réseau qui autorise le trafic distant provenant uniquement d’adresses IP publiques figurant sur une liste verte. Le groupe de sécurité réseau doit autoriser le trafic RDP (Bureau à distance).

## <a name="recommendations"></a>Recommandations

Vos exigences peuvent différer de celles de l’architecture décrite ici. Utilisez ces recommandations comme point de départ.

### <a name="resource-group-recommendations"></a>Recommandations pour les groupes de ressources

Nous vous recommandons de séparer les groupes de ressources en fonction du rôle de serveur et d’avoir un groupe de ressources distinct pour les composants d’infrastructure qui sont des ressources globales. Dans cette architecture, les ressources SharePoint forment un groupe, alors que le serveur SQL Server et les autres composants de l’utilitaire en forment un autre.

### <a name="virtual-network-and-subnet-recommendations"></a>Recommandations pour les réseaux virtuels et les sous-réseaux

Utilisez un sous-réseau pour chaque rôle SharePoint, ainsi qu’un sous-réseau pour la passerelle et un autre pour la jumpbox.

Le sous-réseau de passerelle doit être nommé *GatewaySubnet*. Attribuez l’espace d’adressage du sous-réseau passerelle à partir de la dernière partie de l’espace d’adressage du réseau virtuel. Pour plus d’informations, consultez l’article [Connect an on-premises network to Azure using a VPN gateway][hybrid-vpn-ra] (Connecter un réseau local à Azure à l’aide d’une passerelle VPN).

### <a name="vm-recommendations"></a>Recommandations pour les machines virtuelles

Cette architecture nécessite au moins 44 cœurs :

- 8 serveurs SharePoint sur Standard_DS3_v2 (4 cœurs chacun) = 32 cœurs
- 2 contrôleurs de domaine Active Directory sur Standard_DS1_v2 (1 cœur chacun) = 2 cœurs
- 2 machines virtuelles SQL Server sur Standard_DS3_v2 = 8 cœurs
- 1 nœud majoritaire sur Standard_DS1_v2 = 1 noyau
- 1 serveur d’administration sur Standard_DS1_v2 = 1 noyau

Assurez-vous que votre abonnement Azure dispose d’un quota de cœurs de machines virtuelles suffisamment important pour le déploiement ou le déploiement échouera. Consultez l’article [Abonnement Azure et limites, quotas et contraintes de service][quotas].

Pour tous les rôles SharePoint, à l’exception de l’indexeur de recherche, l’utilisation de la taille de machine virtuelle [Standard_DS3_v2][vm-sizes-general] est recommandée. L’indexeur de recherche doit être de taille [Standard_DS13_v2][vm-sizes-memory] au minimum. À des fins de test, les fichiers de paramètres pour cette architecture de référence spécifient la plus petite taille DS3_v2 pour le rôle de l’indexeur de recherche. Pour un déploiement de production, mettez à jour les fichiers de paramètres pour utiliser la taille DS13 ou une taille supérieure. Pour plus d’informations, consultez l’article [Configuration matérielle et logicielle requise pour une solution SharePoint Server 2016][sharepoint-reqs].

Pour les machines virtuelles SQL Server, nous vous recommandons un minimum de 4 cœurs et 8 Go de RAM. Les fichiers de paramètres pour cette architecture de référence spécifient la taille de DS3_v2. Pour un déploiement de production, vous devrez peut-être spécifier une plus grande taille de machine virtuelle. Pour plus d’informations, consultez l’article [Planification et configuration de la capacité de SQL Server et du stockage (SharePoint Server)](/sharepoint/administration/storage-and-sql-server-capacity-planning-and-configuration#estimate-memory-requirements).

### <a name="nsg-recommendations"></a>Recommandations en matière de groupes de sécurité réseau

Nous vous recommandons d’avoir un groupe de sécurité réseau pour chaque sous-réseau contenant des machines virtuelles, cela permet d’activer l’isolation des sous-réseaux. Si vous souhaitez configurer l’isolation de sous-réseaux, ajoutez des règles de groupe de sécurité réseau qui définissent le trafic entrant ou sortant autorisé ou refusé pour chaque sous-réseau. Pour plus d’informations, consultez l’article [Filtrer le trafic réseau avec les groupes de sécurité réseau][virtual-networks-nsg].

N’affectez pas un groupe de sécurité réseau au sous-réseau de passerelle, ou la passerelle cessera de fonctionner.

### <a name="storage-recommendations"></a>Recommandations de stockage

La configuration du stockage des machines virtuelles dans la batterie de serveurs doit correspondre aux meilleures pratiques appropriées, utilisées pour les déploiements locaux. Les serveurs SharePoint doivent avoir un disque distinct pour les journaux. Les serveurs SharePoint hébergeant les rôles des index de recherche nécessitent un espace disque supplémentaire pour stocker l’index de recherche. Pour SQL Server, la pratique courante consiste à séparer les données et les journaux. Ajoutez davantage de disques pour le stockage de sauvegarde de base de données et utilisez un disque distinct pour [tempdb][tempdb].

Pour une meilleure fiabilité, nous recommandons l’utilisation d’[Azure Managed Disks][managed-disks]. Les disques gérés assurent que les disques des machines virtuelles au sein d’un groupe à haute disponibilité sont isolés afin d’éviter des points de défaillance uniques.

> [!NOTE]
> Actuellement, le modèle Resource Manager de cette architecture de référence n’utilise pas de disques gérés. Nous avons l’intention de mettre à jour le modèle pour utiliser des disques gérés.

Utilisez des disques gérés Premium pour toutes les machines virtuelles SQL Server et SharePoint. Vous pouvez utiliser des disques gérés standard pour le serveur de nœud majoritaire, les contrôleurs de domaine et le serveur d’administration.

### <a name="sharepoint-server-recommendations"></a>Recommandations relatives à SharePoint Server

Avant de configurer la batterie de serveurs SharePoint, assurez-vous que vous avez un compte de service Windows Server Active Directory par service. Pour cette architecture, vous devez au minimum disposer des comptes au niveau du domaine suivants pour isoler les privilèges par rôle :

- Compte de service SQL Server
- Compte de configuration d’utilisateur
- Compte de batterie de serveurs
- Compte Search Service
- Compte d’accès au contenu
- Comptes de Pool d’applications web
- Comptes de Pool d’applications de service
- Compte de super utilisateur de cache
- Compte de super lecteur de cache

Pour satisfaire à l’exigence de prise en charge pour un débit de disque de 200 Mo par seconde au minimum, assurez-vous de planifier l’architecture de recherche. Consultez l’article [Planifier l’architecture de recherche d’entreprise dans SharePoint Server 2013][sharepoint-search]. Suivez également les instructions contenues dans l’article [Best practices for crawling in SharePoint Server 2016][sharepoint-crawling] (Meilleures pratiques pour l’analyse dans SharePoint Server 2016).

De plus, stockez les données de composant de recherche sur un volume de stockage distinct ou sur une partition avec des performances élevées. Pour réduire la charge et améliorer le débit, configurez les comptes d’utilisateur de cache d’objets, qui sont requis dans cette architecture. Fractionnez les fichiers de système d’exploitation Windows Server, les fichiers programme SharePoint Server 2016 et les journaux de diagnostic sur trois volumes de stockage distincts ou sur des partitions avec des performances normales.

Pour plus d’informations sur ces recommandations, consultez l’article [Comptes d’administration et de service de déploiement initial dans SharePoint Server 2016][sharepoint-accounts].

### <a name="hybrid-workloads"></a>Charges de travail hybrides

Cette architecture de référence déploie une batterie de serveurs SharePoint Server 2016 qui peut être utilisée comme un [environnement hybride de SharePoint][sharepoint-hybrid] &mdash;, en étendant SharePoint Server 2016 à Office 365 SharePoint Online. Si vous avez Office Online Server, consultez l’article [Prise en charge de Microsoft Office Web Apps et Office Online Server dans Azure][office-web-apps].

Les applications de service par défaut de ce déploiement sont conçues pour prendre en charge les charges de travail hybrides. Toutes les charges de travail hybrides SharePoint Server 2016 et Office 365 peuvent être déployées sur cette batterie de serveurs sans apporter de modifications à l’infrastructure SharePoint, à une exception près : l’application de service de recherche hybride cloud ne doit pas être déployée sur les serveurs hébergeant une topologie de recherche existante. Par conséquent, une ou plusieurs machines virtuelles en fonction du rôle de recherche doivent être ajoutées à la batterie de serveurs pour prendre en charge ce scénario hybride.

### <a name="sql-server-always-on-availability-groups"></a>Groupes de disponibilité SQL Server Always On

Cette architecture utilise des ordinateurs virtuels SQL Server, car SharePoint Server 2016 ne peut pas utiliser les bases de données SQL Azure. Pour prendre en charge la haute disponibilité dans SQL Server, nous recommandons d’utiliser des groupes de disponibilité Always On, qui spécifient un ensemble de bases de données qui basculent ensemble, ce qui les rend hautement disponibles et récupérables. Dans cette architecture de référence, les bases de données sont créés au cours du déploiement, mais vous devez manuellement activer les groupes de disponibilité Always On et ajouter les bases de données SharePoint à un groupe de disponibilité. Pour plus d’informations, consultez l’article [Créer le groupe de disponibilité et ajouter les bases de données SharePoint][create-availability-group].

Nous recommandons également l’ajout d’une adresse IP d’écouteur au cluster, il s’agit de l’adresse IP privée de l’équilibreur de charge interne pour les machines virtuelles SQL Server.

Pour les tailles de machine virtuelle recommandées et d’autres recommandations relatives aux performances pour SQL Server s’exécutant dans Azure, consultez l’article [Meilleures pratiques relatives aux performances de SQL Server dans les machines virtuelles Azure][sql-performance]. Suivez également les recommandations de l’article [Meilleures pratiques pour SQL Server dans une batterie de serveurs SharePoint Server 2016][sql-sharepoint-best-practices].

Le serveur de nœud majoritaire devrait se trouver sur un ordinateur distinct des partenaires de réplication. Le serveur permet au serveur du partenaire de réplication secondaire dans une session de mode haute sécurité, de déterminer s’il faut initier un basculement automatique. Contrairement aux deux partenaires, le serveur de nœud majoritaire ne sert pas la base de données, mais au lieu de cela, il prend en charge le basculement automatique.

## <a name="scalability-considerations"></a>Considérations relatives à l’extensibilité

Pour mettre à l’échelle les serveurs existants, modifiez simplement la taille de machine virtuelle.

Avec la fonctionnalité [MinRoles][minroles] dans SharePoint Server 2016, vous pouvez mettre à niveau les serveurs basés sur le rôle de serveur et supprimer des serveurs à partir d’un rôle. Lorsque vous ajoutez des serveurs à un rôle, vous pouvez spécifier les rôles uniques ou l’un des rôles combinés. Si vous ajoutez des serveurs au rôle de recherche, toutefois, vous devez également reconfigurer la topologie de recherche à l’aide de PowerShell. Vous pouvez également convertir des rôles à l’aide de MinRoles. Pour plus d’informations, consultez l’article [Gestion d’une batterie de serveurs MinRole dans SharePoint Server 2016][sharepoint-minrole].

Notez que SharePoint Server 2016 ne prend pas en charge l’utilisation de groupes de machines virtuelles identiques pour la mise à l’échelle automatique.

## <a name="availability-considerations"></a>Considérations relatives à la disponibilité

Cette architecture de référence prend en charge la haute disponibilité dans une région Azure, étant donné que chaque rôle dispose d’au moins deux ordinateurs virtuels déployés dans un groupe à haute disponibilité.

Pour vous protéger contre une panne régionale, créez une batterie de serveurs de récupération d’urgence distincte dans une région Azure différente. Vos objectifs de délai de récupération (RTO) et les objectifs de point de récupération (RPO) déterminent la configuration requise. Pour plus d’informations, consultez l’article [Choisir une stratégie de récupération d’urgence pour SharePoint Server 2016][sharepoint-dr]. La région secondaire doit être une *région jumelée* avec la région primaire. En cas de panne étendue, la récupération d’une région est hiérarchisée pour chaque paire. Pour plus d’informations, consultez l’article [Continuité des activités et récupération d’urgence (BCDR) : régions jumelées d’Azure][paired-regions].

## <a name="manageability-considerations"></a>Considérations relatives à la facilité de gestion

Pour exploiter et maintenir les serveurs, les batteries de serveurs et les sites, suivez les pratiques recommandées pour les opérations SharePoint. Pour plus d’informations, consultez l’article [Opérations pour SharePoint Server 2016][sharepoint-ops].

Les tâches à prendre en compte lors de la gestion de SQL Server dans un environnement SharePoint peuvent différer de celles généralement prises en compte pour une application de base de données. Une bonne pratique consiste à sauvegarder toutes les bases de données SQL chaque semaine avec des sauvegardes incrémentielles nocturnes. Sauvegardez les journaux des transactions toutes les 15 minutes. Une autre pratique consiste à implémenter des tâches de maintenance de SQL Server sur les bases de données lors de la désactivation de celles intégrées à SharePoint. Pour plus d’informations, consultez l’article [Planification et configuration de la capacité de SQL Server et du stockage][sql-server-capacity-planning].

## <a name="security-considerations"></a>Considérations relatives à la sécurité

Les comptes de service au niveau du domaine utilisés pour exécuter un serveur SharePoint 2016 nécessitent des contrôleurs de domaine Windows Server Active Directory pour les processus de jonction de domaine et d’authentification. Azure Active Directory Domain Services ne peut pas être utilisé à cet effet. Pour étendre l’infrastructure d’identité Windows Server Active Directory déjà en place dans l’intranet, cette architecture utilise deux contrôleurs de domaine réplica Windows Server Active Directory d’une forêt locale existante Windows Server Active Directory.

En outre, il est toujours conseillé de planifier le renforcement de la sécurité. Les autres recommandations sont les suivantes :

- Ajouter des règles pour les groupes de sécurité réseau pour isoler les sous-réseaux et les rôles.
- N’affectez pas les adresses IP publiques aux machines virtuelles.
- Pour la détection d’intrusion et l’analyse des charges utiles, envisagez d’utiliser une appliance virtuelle réseau devant les serveurs web frontaux à la place d’un équilibreur de charge interne Azure.
- En option, utilisez les stratégies IPsec pour le chiffrement du trafic de texte en clair entre les serveurs. Si vous effectuez également l’isolation des sous-réseaux, mettez à jour vos règles de groupe de sécurité réseau pour autoriser le trafic IPsec.
- Installez des agents de logiciels anti-programmes malveillants pour les machines virtuelles.

## <a name="deploy-the-solution"></a>Déployer la solution

Un déploiement pour cette architecture de référence est disponible sur [GitHub][github]. La totalité du déploiement peut durer plusieurs heures.

Le déploiement crée les groupes de ressources suivants dans votre abonnement :

- ra-onprem-sp2016-rg
- ra-sp2016-network-rg

Les fichiers de paramètre modèle font référence à ces noms. Si vous les modifiez, mettez à jour les fichiers de paramètres afin qu’ils correspondent.

Les fichiers de paramètres incluent un mot de passe codé en dur dans divers emplacements. Modifiez ces valeurs avant de déployer.

### <a name="prerequisites"></a>Prérequis

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deployment-steps"></a>Étapes du déploiement

1. Exécutez la commande suivante pour déployer un réseau simulé local.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p onprem.json --deploy
    ```

2. Exécutez la commande suivante pour déployer le réseau virtuel et la passerelle VPN Azure.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p connections.json --deploy
    ```

3. Exécutez la commande suivante pour déployer la jumpbox, les contrôleurs de domaine AD et les machines virtuelles SQL Serveur.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure1.json --deploy
    ```

4. Exécutez la commande suivante pour créer le cluster de basculement et le groupe de disponibilité.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure2-cluster.json --deploy
    ```

5. Exécutez la commande suivante pour déployer les machines virtuelles restantes.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure3.json --deploy
    ```

À ce stade, vérifiez que vous pouvez établir une connexion TCP du serveur web frontal à l’équilibreur de charge pour le groupes de disponibilité AlwaysOn SQL Server. Pour ce faire, suivez les étapes suivantes :

1. Utilisez le portail Azure pour trouver la machine virtuelle appelée `ra-sp-jb-vm1` dans le groupe de ressources `ra-sp2016-network-rg`. Il s’agit de la machine virtuelle de la jumpbox.

2. Cliquez sur `Connect` pour ouvrir une session Bureau à distance vers la machine virtuelle. Utilisez le mot de passe spécifié dans le fichier de paramètre `azure1.json`.

3. Depuis la session Bureau à distance, connectez-vous à 10.0.5.4. Il s’agit de l’adresse IP de la machine virtuelle nommée `ra-sp-app-vm1`.

4. Ouvrez une console PowerShell dans la machine virtuelle et utilisez la cmdlet `Test-NetConnection` pour vérifier que vous pouvez vous connecter à l’équilibreur de charge.

    ```powershell
    Test-NetConnection 10.0.3.100 -Port 1433
    ```

Le résultat doit être semblable à ce qui suit :

```console
ComputerName     : 10.0.3.100
RemoteAddress    : 10.0.3.100
RemotePort       : 1433
InterfaceAlias   : Ethernet 3
SourceAddress    : 10.0.0.132
TcpTestSucceeded : True
```

S’il échoue, utilisez le portail Azure pour redémarrer la machine virtuelle nommée `ra-sp-sql-vm2`. Après son redémarrage, exécutez de nouveau la commande `Test-NetConnection`. Vous devrez peut-être patienter environ une minute après le redémarrage pour que la connexion soit établie.

Effectuez le déploiement comme suit.

1. Exécutez la commande suivante pour déployer le nœud principal de la batterie de serveurs SharePoint.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure4-sharepoint-server.json --deploy
    ```

2. Exécutez la commande suivante pour déployer le cache, la recherche et le serveur web SharePoint.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure5-sharepoint-farm.json --deploy
    ```

3. Exécutez la commande suivante pour créer les règles du groupe de sécurité réseau.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure6-security.json --deploy
    ```

### <a name="validate-the-deployment"></a>Valider le déploiement

1. Dans le [portail Azure][azure-portal], accédez au groupe de ressources `ra-onprem-sp2016-rg`.

2. Dans la liste des ressources, sélectionnez la ressource de machine virtuelle nommée `ra-onpr-u-vm1`.

3. Connectez-vous à la machine virtuelle, comme décrit dans la section [Connexion à la machine virtuelle][connect-to-vm]. Le nom d’utilisateur est `\onpremuser`.

4. Lorsque la connexion à distance à la machine virtuelle est établie, ouvrez un navigateur sur l’ordinateur virtuel et accédez à `http://portal.contoso.local`.

5. Dans la zone **Sécurité Windows**, connectez-vous au portail SharePoint en utilisant `contoso.local\testuser` pour le nom d’utilisateur.

Cette ouverture de session« tunnèle » du domaine Fabrikam.com utilisé par le réseau local au domaine contoso.local utilisé par le portail SharePoint. Lorsque le site SharePoint s’ouvre, le site de démonstration racine s’affiche.

**_Contributeurs à cette architecture de référence_** &mdash; Joe Davies, Bob Fox, Neil Hodgkinson, Paul Stork

<!-- links -->

[availability-set]: /azure/virtual-machines/windows/manage-availability
[azure-portal]: https://portal.azure.com
[azure-ps]: /powershell/azure/overview
[azure-pricing]: https://azure.microsoft.com/pricing/calculator/
[bastion-host]: https://en.wikipedia.org/wiki/Bastion_host
[create-availability-group]: https://technet.microsoft.com/library/mt793548(v=office.16).aspx
[connect-to-vm]: /azure/virtual-machines/windows/quick-create-portal#connect-to-virtual-machine
[github]: https://github.com/mspnp/reference-architectures
[hybrid-ra]: ../hybrid-networking/index.md
[hybrid-vpn-ra]: ../hybrid-networking/vpn.md
[load-balancer]: /azure/load-balancer/load-balancer-internal-overview
[managed-disks]: /azure/storage/storage-managed-disks-overview
[minroles]: https://technet.microsoft.com/library/mt346114(v=office.16).aspx
[nsg]: /azure/virtual-network/virtual-networks-nsg
[office-web-apps]: https://support.microsoft.com/help/3199955/office-web-apps-and-office-online-server-supportability-in-azure
[paired-regions]: /azure/best-practices-availability-paired-regions
[readme]: https://github.com/mspnp/reference-architectures/tree/master/sharepoint/sharepoint-2016
[resource-group]: /azure/azure-resource-manager/resource-group-overview
[quotas]: /azure/azure-subscription-service-limits
[sharepoint-accounts]: https://technet.microsoft.com/library/ee662513(v=office.16).aspx
[sharepoint-crawling]: https://technet.microsoft.com/library/dn535606(v=office.16).aspx
[sharepoint-dr]: https://technet.microsoft.com/library/ff628971(v=office.16).aspx
[sharepoint-hybrid]: https://aka.ms/sphybrid
[sharepoint-minrole]: https://technet.microsoft.com/library/mt743705(v=office.16).aspx
[sharepoint-ops]: https://technet.microsoft.com/library/cc262289(v=office.16).aspx
[sharepoint-reqs]: https://technet.microsoft.com/library/cc262485(v=office.16).aspx
[sharepoint-search]: https://technet.microsoft.com/library/dn342836.aspx
[sql-always-on]: /sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server
[sql-performance]: /azure/virtual-machines/windows/sql/virtual-machines-windows-sql-performance
[sql-server-capacity-planning]: https://technet.microsoft.com/library/cc298801(v=office.16).aspx
[sql-quorum]: https://technet.microsoft.com/library/cc731739(v=ws.11).aspx
[sql-sharepoint-best-practices]: https://technet.microsoft.com/library/hh292622(v=office.16).aspx
[tempdb]: /sql/relational-databases/databases/tempdb-database
[virtual-networks-nsg]: /azure/virtual-network/virtual-networks-nsg
[visio-download]: https://archcenter.blob.core.windows.net/cdn/Sharepoint-2016.vsdx
[vm-sizes-general]: /azure/virtual-machines/windows/sizes-general
[vm-sizes-memory]: /azure/virtual-machines/windows/sizes-memory
[windows-n-tier]: ../virtual-machines-windows/n-tier.md
