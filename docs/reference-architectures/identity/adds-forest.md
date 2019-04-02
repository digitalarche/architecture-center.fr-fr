---
title: Créer une forêt de ressources AD DS dans Azure
titleSuffix: Azure Reference Architectures
description: >-
  Comment créer un domaine Active Directory approuvé dans Azure.

  conseils, passerelle vpn, expressroute, équilibreur de charge, réseau virtuel, active directory
author: telmosampaio
ms.date: 05/02/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18, identity
ms.openlocfilehash: 5fe966f657782b41ec1926d0fd4bb83eb7a3c0fb
ms.sourcegitcommit: 548374a0133f3caed3934fda6a380c76e6eaecea
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/25/2019
ms.locfileid: "58420037"
---
# <a name="create-an-active-directory-domain-services-ad-ds-resource-forest-in-azure"></a>Créer une forêt de ressources Active Directory Domain Services (AD DS) dans Azure

Cette architecture de référence montre comment créer, dans Azure, un domaine Active Directory distinct approuvé par les domaines de votre forêt AD locale. [**Déployez cette solution**](#deploy-the-solution).

![Architecture réseau hybride sécurisée avec des domaines Active Directory distincts](./images/adds-forest.png)

*Téléchargez un [fichier Visio][visio-download] de cette architecture.*

Active Directory Domain Services (AD DS) stocke des informations d’identité dans une structure hiérarchique. Le nœud supérieur de la structure hiérarchique est appelé « forêt ». Une forêt contient des domaines, qui à leur tour contiennent d’autres types d’objets. Cette architecture de référence crée une forêt AD DS dans Azure avec une relation d’approbation à sens unique sortante avec un domaine local. La forêt dans Azure contient un domaine qui n’existe pas localement. En raison de la relation d’approbation, les ouvertures de session effectuées dans les domaines locaux peuvent être approuvées pour l’accès aux ressources dans le domaine Azure distinct.

Parmi les utilisations courantes de cette architecture figurent la conservation d’une séparation de sécurité pour les objets et les identités contenus dans le cloud et la migration de domaines individuels depuis l’environnement local vers le cloud.

Pour plus d’informations, consultez [Choisir une solution pour intégrer l’environnement Active Directory local à Azure][considerations].

## <a name="architecture"></a>Architecture

L’architecture possède les composants suivants :

- **Réseau local**. Le réseau local contient ses propres forêt et domaines Active Directory.
- **Serveurs Active Directory**. Il s’agit de contrôleurs de domaine qui implémentent des services de domaine s’exécutant en tant que machines virtuelles dans le cloud. Ces serveurs hébergent une forêt qui contient un ou plusieurs domaines, distincts de ceux situés dans l’environnement local.
- **Relation d’approbation à sens unique**. L’exemple dans le diagramme montre une approbation à sens unique entre le domaine dans Azure et le domaine local. Cette relation permet aux utilisateurs locaux d’accéder aux ressources du domaine dans Azure, mais pas l’inverse. Il est possible de créer une approbation bidirectionnelle, si les utilisateurs du cloud requièrent également l’accès à des ressources locales.
- **Sous-réseau Active Directory**. Les serveurs AD DS sont hébergés dans un sous-réseau distinct. Des règles de groupe de sécurité réseau (NSG) protègent les serveurs AD DS et fournissent un pare-feu contre le trafic en provenance de sources inconnues.
- **Passerelle Azure**. La passerelle Azure fournit une connexion entre le réseau local et le réseau virtuel Azure. Il peut s’agir d’une [connexion VPN][azure-vpn-gateway] ou [d’Azure ExpressRoute][azure-expressroute]. Pour plus d’informations, consultez [Implémentation d’une architecture réseau hybride sécurisée dans Azure][implementing-a-secure-hybrid-network-architecture].

## <a name="recommendations"></a>Recommandations

Pour obtenir des recommandations spécifiques sur l’implémentation d’Active Directory dans Azure, consultez [extension Active Directory Services de domaine (AD DS) pour Azure][adds-extend-domain].

### <a name="trust"></a>Trust

Les domaines locaux sont contenus dans une forêt différente des domaines dans le cloud. Pour activer l’authentification des utilisateurs locaux dans le cloud, les domaines dans Azure doivent faire confiance au domaine d’ouverture de session dans la forêt locale. De même, si le cloud fournit un domaine d’ouverture de session pour les utilisateurs externes, il peut être nécessaire que la forêt locale fasse confiance au domaine du cloud.

Vous pouvez établir des approbations au niveau forêt, en [créant des approbations de forêt][creating-forest-trusts], ou au niveau domaine, en [créant des approbations externes][creating-external-trusts]. Une approbation de niveau forêt crée une relation entre tous les domaines dans deux forêts. Une approbation de niveau domaine externe crée uniquement une relation entre deux domaines spécifiés. Vous devez uniquement créer des approbations de niveau domaine externe entre des domaines dans des forêts différentes.

Les approbations peuvent être unidirectionnelles (à sens unique) ou bidirectionnelles :

- Une approbation à sens unique permet aux utilisateurs d’une forêt ou domaine (forêt ou domaine *entrant*) d’accéder aux ressources contenues dans une autre forêt ou domaine (forêt ou domaine *sortant*).
- Une approbation bidirectionnelle permet aux utilisateurs de l’un des domaines ou forêts d’accéder aux ressources contenues dans l’autre domaine ou forêt.

Le tableau suivant récapitule les configurations d’approbation pour certains scénarios simples :

| Scénario | Approbation locale | Approbation au niveau du cloud |
| --- | --- | --- |
| Les utilisateurs locaux requièrent l’accès aux ressources dans le cloud, mais l’inverse n’est pas vrai. |À sens unique, entrante |À sens unique, sortante |
| Les utilisateurs dans le cloud requièrent l’accès aux ressources locales, mais l’inverse n’est pas vrai. |À sens unique, sortante |À sens unique, entrante |
| Les utilisateurs dans le cloud et locaux requièrent l’accès aux ressources contenues dans le cloud et locales. |Bidirectionnelle, entrante et sortante |Bidirectionnelle, entrante et sortante |

## <a name="scalability-considerations"></a>Considérations relatives à l’extensibilité

Active Directory peut évoluer automatiquement pour les contrôleurs de domaine qui font partie du même domaine. Les demandes sont distribuées sur tous les contrôleurs d’un domaine. Vous pouvez ajouter un contrôleur de domaine, qui se synchronise alors automatiquement avec le domaine. Ne configurez pas d’équilibreur de charge distinct pour diriger le trafic vers des contrôleurs dans le domaine. Vérifiez que tous les contrôleurs de domaine disposent de suffisamment de ressources mémoire et de stockage pour gérer la base de données du domaine. Attribuez la même taille aux machines virtuelles de contrôleur de domaine.

## <a name="availability-considerations"></a>Considérations relatives à la disponibilité

Provisionnez au moins deux contrôleurs de domaine par domaine. Cela permet une réplication automatique entre les serveurs. Créez un groupe à haute disponibilité pour les machines virtuelles faisant office de serveurs Active Directory chargés de gérer chaque domaine. Placez au moins deux serveurs dans ce groupe à haute disponibilité.

Envisagez également de désigner un ou plusieurs serveurs dans chaque domaine en tant que [maîtres d’opérations en attente][standby-operations-masters] en cas d’échec de la connectivité à un serveur agissant en tant que rôle de FSMO (Flexible Single Master Operation).

## <a name="manageability-considerations"></a>Considérations relatives à la facilité de gestion

Pour plus d’informations sur les considérations relatives à la gestion et à la surveillance, consultez [Extension d’Active Directory à Azure][adds-extend-domain].

Pour plus d’informations, consultez [Surveillance d’Active Directory][monitoring_ad]. Vous pouvez installer des outils tels que [Microsoft Systems Center][microsoft_systems_center] sur un serveur de surveillance dans le sous-réseau de gestion pour effectuer ces tâches.

## <a name="security-considerations"></a>Considérations relatives à la sécurité

Les approbations de niveau forêt sont transitives. Si vous établissez une approbation de niveau forêt entre une forêt locale et une forêt dans le cloud, cette approbation est étendue aux autres domaines créés dans l’une ou l’autre des forêts. Si vous utilisez des domaines pour fournir une séparation à des fins de sécurité, envisagez de créer des approbations au niveau domaine uniquement. Les approbations de niveau domaine sont non transitives.

Pour connaître les considérations relatives à la sécurité propres à Active Directory, consultez la section sur les considérations relatives à la sécurité dans [Extension d’Active Directory à Azure][adds-extend-domain].

## <a name="deploy-the-solution"></a>Déployer la solution

Un déploiement pour cette architecture est disponible sur [GitHub][github]. Remarque : le déploiement entier peut prendre jusqu’à deux heures, en incluant la création de la passerelle VPN et l’exécution des scripts qui configurent AD DS.

### <a name="prerequisites"></a>Conditions préalables

1. Clonez, dupliquez ou téléchargez le fichier zip pour le [dépôt GitHub](https://github.com/mspnp/identity-reference-architectures).

2. Installez [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).

3. Installez le package npm des [modules Azure](https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks).

   ```bash
   npm install -g @mspnp/azure-building-blocks
   ```

4. À partir d’une invite de commandes, d’une invite bash ou de l’invite de commandes PowerShell, connectez-vous à votre compte Azure, comme suit :

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a>Déployer le centre de données local simulé

1. Accédez au dossier `identity/adds-forest` du dépôt GitHub.

2. Ouvrez le fichier `onprem.json` . Cherchez les instances de `adminPassword` et `Password` et ajoutez les valeurs pour les mots de passe.

3. Exécutez la commande suivante et attendez que le déploiement se termine :

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p onprem.json --deploy
    ```

### <a name="deploy-the-azure-vnet"></a>Déployer le réseau virtuel Azure

1. Ouvrez le fichier `azure.json` . Cherchez les instances de `adminPassword` et `Password` et ajoutez les valeurs pour les mots de passe.

2. Dans le même fichier, recherchez les instances de `sharedKey` et entrez les clés partagées pour la connexion VPN.

    ```json
    "sharedKey": "",
    ```

3. Exécutez la commande suivante et attendez que le déploiement se termine.

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p azure.json --deploy
    ```

   Déployez dans le même groupe de ressource que le réseau virtuel local.

### <a name="test-the-ad-trust-relation"></a>Tester la relation d’approbation AD

1. Utilisez le portail Azure, accédez au groupe de ressources que vous avez créé.

2. Utilisez le portail Azure pour trouver la machine virtuelle appelée `ra-adt-mgmt-vm1`.

3. Cliquez sur `Connect` pour ouvrir une session Bureau à distance vers la machine virtuelle. Le nom d’utilisateur est `contoso\testuser`, et le mot de passe est celui que vous avez spécifié dans le fichier de paramètre `onprem.json`.

4. Une fois dans votre session Bureau à distance, ouvrez une autre session Bureau à distance vers 192.168.0.4, qui correspond à l’adresse IP de la machine virtuelle appelée `ra-adtrust-onpremise-ad-vm1`. Le nom d’utilisateur est `contoso\testuser`, et le mot de passe est celui que vous avez spécifié dans le fichier de paramètre `azure.json`.

5. Une fois dans la session Bureau à distance pour `ra-adtrust-onpremise-ad-vm1`, allez dans **Gestionnaire de serveur** et cliquez sur **Outils** > **Domaines Active Directory et approbations**.

6. Dans le volet gauche, faites un clic droit sur contoso.com, puis sélectionnez **Propriétés**.

7. Cliquez sur l’onglet **Approbation**. Vous devez voir treyresearch.net répertorié comme une approbation entrante.

![Capture d’écran de la boîte de dialogue d’approbation Forêt Active Directory](./images/ad-forest-trust.png)

## <a name="next-steps"></a>Étapes suivantes

- Découvrez les bonnes pratiques pour [étendre votre domaine AD DS local à Azure][adds-extend-domain].
- Découvrez les bonnes pratiques pour [créer une infrastructure AD FS][adfs] dans Azure.

<!-- links -->
[adds-extend-domain]: adds-extend-domain.md
[adfs]: adfs.md
[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks

[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md

[running-VMs-for-an-N-tier-architecture-on-Azure]: ../virtual-machines-windows/n-tier.md

[ad-azure-guidelines]: https://msdn.microsoft.com/library/azure/jj156090.aspx
[azure-expressroute]: /azure/expressroute/expressroute-introduction
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[considerations]: ./considerations.md
[creating-external-trusts]: https://technet.microsoft.com/library/cc816837(v=ws.10).aspx
[creating-forest-trusts]: https://technet.microsoft.com/library/cc816810(v=ws.10).aspx
[github]: https://github.com/mspnp/identity-reference-architectures/tree/master/adds-forest
[incoming-trust]: https://raw.githubusercontent.com/mspnp/identity-reference-architectures/master/adds-forest/extensions/incoming-trust.ps1
[microsoft_systems_center]: https://microsoft.com/cloud-platform/system-center
[monitoring_ad]: https://msdn.microsoft.com/library/bb727046.aspx
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[solution-script]: https://raw.githubusercontent.com/mspnp/identity-reference-architectures/master/adds-forest/Deploy-ReferenceArchitecture.ps1
[standby-operations-masters]: https://technet.microsoft.com/library/cc794737(v=ws.10).aspx
[outgoing-trust]: https://raw.githubusercontent.com/mspnp/identity-reference-architectures/master/adds-forest/extensions/outgoing-trust.ps1
[verify-a-trust]: https://technet.microsoft.com/library/cc753821.aspx
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx
