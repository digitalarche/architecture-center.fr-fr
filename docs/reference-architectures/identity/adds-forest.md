---
title: "Créer une forêt de ressources AD DS dans Azure"
description: "Comment créer un domaine Active Directory approuvé dans Azure.\nconseils, passerelle vpn, expressroute, équilibreur de charge, réseau virtuel, active directory"
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Identity management
pnp.series.prev: adds-extend-domain
pnp.series.next: adfs
cardTitle: Create an AD DS forest in Azure
ms.openlocfilehash: b946afa91e8bd303c51f97e18be170c4105cc8c5
ms.sourcegitcommit: 8ab30776e0c4cdc16ca0dcc881960e3108ad3e94
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2017
---
# <a name="create-an-active-directory-domain-services-ad-ds-resource-forest-in-azure"></a>Créer une forêt de ressources Active Directory Domain Services (AD DS) dans Azure

Cette architecture de référence montre comment créer, dans Azure, un domaine Active Directory distinct approuvé par les domaines de votre forêt AD locale. [**Déployez cette solution**.](#deploy-the-solution)

[![0]][0] 

*Téléchargez un [fichier Visio][visio-download] de cette architecture.*

Active Directory Domain Services (AD DS) stocke des informations d’identité dans une structure hiérarchique. Le nœud supérieur de la structure hiérarchique est appelé « forêt ». Une forêt contient des domaines, qui à leur tour contiennent d’autres types d’objets. Cette architecture de référence crée une forêt AD DS dans Azure avec une relation d’approbation à sens unique sortante avec un domaine local. La forêt dans Azure contient un domaine qui n’existe pas localement. En raison de la relation d’approbation, les ouvertures de session effectuées dans les domaines locaux peuvent être approuvées pour l’accès aux ressources dans le domaine Azure distinct. 

Parmi les utilisations courantes de cette architecture figurent la conservation d’une séparation de sécurité pour les objets et les identités contenus dans le cloud et la migration de domaines individuels depuis l’environnement local vers le cloud. 

Pour plus d’informations, consultez [Choisir une solution pour intégrer l’environnement Active Directory local à Azure][considerations]. 

## <a name="architecture"></a>Architecture

L’architecture possède les composants suivants :

* **Réseau local**. Le réseau local contient ses propres forêt et domaines Active Directory.
* **Serveurs Active Directory**. Il s’agit de contrôleurs de domaine qui implémentent des services de domaine s’exécutant en tant que machines virtuelles dans le cloud. Ces serveurs hébergent une forêt qui contient un ou plusieurs domaines, distincts de ceux situés dans l’environnement local.
* **Relation d’approbation à sens unique**. L’exemple dans le diagramme montre une approbation à sens unique entre le domaine dans Azure et le domaine local. Cette relation permet aux utilisateurs locaux d’accéder aux ressources du domaine dans Azure, mais pas l’inverse. Il est possible de créer une approbation bidirectionnelle, si les utilisateurs du cloud requièrent également l’accès à des ressources locales.
* **Sous-réseau Active Directory**. Les serveurs AD DS sont hébergés dans un sous-réseau distinct. Des règles de groupe de sécurité réseau (NSG) protègent les serveurs AD DS et fournissent un pare-feu contre le trafic en provenance de sources inconnues.
* **Passerelle Azure**. La passerelle Azure fournit une connexion entre le réseau local et le réseau virtuel Azure. Il peut s’agir d’une [connexion VPN][azure-vpn-gateway] ou [d’Azure ExpressRoute][azure-expressroute]. Pour plus d’informations, consultez [Implémentation d’une architecture réseau hybride sécurisée dans Azure][implementing-a-secure-hybrid-network-architecture].

## <a name="recommendations"></a>Recommandations

Pour obtenir des recommandations spécifiques sur l’implémentation d’Active Directory dans Azure, consultez les articles suivants :

- [Extension d’Active Directory Domain Services (AD DS) à Azure][adds-extend-domain]. 
- [Recommandations pour déployer Windows Server Active Directory sur des machines virtuelles Azure][ad-azure-guidelines].

### <a name="trust"></a>Trust

Les domaines locaux sont contenus dans une forêt différente des domaines dans le cloud. Pour activer l’authentification des utilisateurs locaux dans le cloud, les domaines dans Azure doivent faire confiance au domaine d’ouverture de session dans la forêt locale. De même, si le cloud fournit un domaine d’ouverture de session pour les utilisateurs externes, il peut être nécessaire que la forêt locale fasse confiance au domaine du cloud.

Vous pouvez établir des approbations au niveau forêt, en [créant des approbations de forêt][creating-forest-trusts], ou au niveau domaine, en [créant des approbations externes][creating-external-trusts]. Une approbation de niveau forêt crée une relation entre tous les domaines dans deux forêts. Une approbation de niveau domaine externe crée uniquement une relation entre deux domaines spécifiés. Vous devez uniquement créer des approbations de niveau domaine externe entre des domaines dans des forêts différentes.

Les approbations peuvent être unidirectionnelles (à sens unique) ou bidirectionnelles :

* Une approbation à sens unique permet aux utilisateurs d’une forêt ou domaine (forêt ou domaine *entrant*) d’accéder aux ressources contenues dans une autre forêt ou domaine (forêt ou domaine *sortant*).
* Une approbation bidirectionnelle permet aux utilisateurs de l’un des domaines ou forêts d’accéder aux ressources contenues dans l’autre domaine ou forêt.

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

Vous disposez d’une solution sur [GitHub][github] pour déployer cette architecture de référence. Vous avez besoin de la dernière version de l’interface de ligne de commande Azure pour exécuter le script PowerShell qui déploie la solution. Pour déployer l’architecture de référence, effectuez les étapes suivantes :

1. Téléchargez ou clonez le dossier de solution à partir de [GitHub][github] sur votre ordinateur local.

2. Ouvrez l’interface de ligne de commande Azure et accédez au dossier de solution local.

3. Exécutez la commande suivante :
   
    ```Powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```
   
    Remplacez `<subscription id>` par l’identifiant de votre abonnement Azure.
   
    Pour `<location>`, spécifiez une région Azure, telle que `eastus` ou `westus`.
   
    Le paramètre `<mode>` contrôle la granularité du déploiement, et peut prendre l’une des valeurs suivantes :
   
   * `Onpremise` : déploie l’environnement local simulé.
   * `Infrastructure` : déploie l’infrastructure du réseau virtuel et le serveur de rebond sur Azure.
   * `CreateVpn` : déploie la passerelle réseau virtuel Azure et la connecte au réseau local simulé.
   * `AzureADDS` : déploie les machines virtuelles faisant office de serveurs Active Directory DS, déploie Active Directory sur ces machines virtuelles et déploie le domaine sur Azure.
   * `WebTier` : déploie l’équilibreur de charge et les machines virtuelles de niveau web.
   * `Prepare` : déploie tous les déploiements précédents. **Il s’agit de l’option recommandée si vous n’avez pas de réseau local, mais que vous souhaitez déployer l’architecture de référence complète décrite ci-dessus à des fins de test ou d’évaluation.** 
   * `Workload` : déploie les équilibreurs de charge et les machines virtuelles de niveau entreprise et données. Notez que ces machines virtuelles ne sont pas incluses dans le déploiement `Prepare`.

4. Attendez la fin du déploiement. Si vous déployez le déploiement `Prepare`, l’opération prend plusieurs heures.
     
5. Si vous utilisez la configuration locale simulée, configurez la relation d’approbation entrante :
   
   1. Connectez-vous au serveur de rebond (*ra-adtrust-mgmt-vm1* dans le groupe de ressources *ra-adtrust-security-rg*). Connectez-vous en tant que *testuser* avec le mot de passe*AweS0me@PW*.
   2. Sur le serveur de rebond, ouvrez une session RDP sur la première machine virtuelle dans le domaine *contoso.com* (le domaine local). Cette machine virtuelle possède l’adresse IP 192.168.0.4. Le nom d’utilisateur est *contoso\testuser*, et le mot de passe est *AweS0me@PW*.
   3. Téléchargez le script [incoming-trust.ps1][incoming-trust] et exécutez-le pour créer l’approbation entrante à partir du domaine *treyresearch.com*.

6. Si vous utilisez votre propre infrastructure locale :
   
   1. Téléchargez le script [incoming-trust.ps1][incoming-trust].
   2. Modifiez le script et remplacez la valeur de la variable `$TrustedDomainName` par le nom de votre domaine.
   3. Exécutez le script.

7. À partir du serveur de rebond, connectez-vous à la première machine virtuelle dans le domaine *treyresearch.com* (le domaine dans le cloud). Cette machine virtuelle possède l’adresse IP 10.0.4.4. Le nom d’utilisateur est *treyresearch\testuser*, et le mot de passe est *AweS0me@PW*.

8. Téléchargez le script [outgoing-trust.ps1][outgoing-trust] et exécutez-le pour créer l’approbation sortante à partir du domaine *treyresearch.com*. Si vous utilisez vos propres machines locales, commencez par modifier le script. Définissez la variable `$TrustedDomainName` sur le nom de votre domaine local, puis spécifiez les adresses IP des serveurs Active Directory DS pour ce domaine dans la variable `$TrustedDomainDnsIpAddresses`.

9. Patientez quelques minutes, puis connectez-vous à une machine virtuelle locale et effectuez les étapes décrites dans l’article [Vérifier une approbation][verify-a-trust] pour déterminer si la relation d’approbation entre les domaines *contoso.com* et *treyresearch.com* est configurée correctement.

## <a name="next-steps"></a>étapes suivantes

* Découvrez les bonnes pratiques pour [étendre votre domaine AD DS local à Azure][adds-extend-domain].
* Découvrez les bonnes pratiques pour [créer une infrastructure AD FS][adfs] dans Azure.

<!-- links -->
[adds-extend-domain]: adds-extend-domain.md
[adfs]: adfs.md

[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md

[running-VMs-for-an-N-tier-architecture-on-Azure]: ../virtual-machines-windows/n-tier.md

[ad-azure-guidelines]: https://msdn.microsoft.com/library/azure/jj156090.aspx
[azure-expressroute]: https://azure.microsoft.com/documentation/articles/expressroute-introduction/
[azure-vpn-gateway]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-vpngateways/
[considerations]: ./considerations.md
[creating-external-trusts]: https://technet.microsoft.com/library/cc816837(v=ws.10).aspx
[creating-forest-trusts]: https://technet.microsoft.com/library/cc816810(v=ws.10).aspx
[github]: https://github.com/mspnp/reference-architectures/tree/master/identity/adds-forest
[incoming-trust]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/extensions/incoming-trust.ps1
[microsoft_systems_center]: https://www.microsoft.com/server-cloud/products/system-center-2016/
[monitoring_ad]: https://msdn.microsoft.com/library/bb727046.aspx
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[solution-script]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/Deploy-ReferenceArchitecture.ps1
[standby-operations-masters]: https://technet.microsoft.com/library/cc794737(v=ws.10).aspx
[outgoing-trust]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/extensions/outgoing-trust.ps1
[verify-a-trust]: https://technet.microsoft.com/library/cc753821.aspx
[visio-download]: https://archcenter.azureedge.net/cdn/identity-architectures.vsdx
[0]: ./images/adds-forest.png "Architecture réseau hybride sécurisée avec des domaines Active Directory distincts"