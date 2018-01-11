---
title: "Implémentation des services de fédération Active Directory (AD FS) dans Azure"
description: "Comment implémenter une architecture réseau hybride sécurisée avec l’autorisation de service de fédération Active Directory dans Azure.\nconseils, passerelle vpn, expressroute, équilibreur de charge, réseau virtuel, active directory"
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Identity management
pnp.series.prev: adds-forest
cardTitle: Extend AD FS to Azure
ms.openlocfilehash: b8c9ae0621c087c68d449dd13e60046104c01513
ms.sourcegitcommit: 8ab30776e0c4cdc16ca0dcc881960e3108ad3e94
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2017
---
# <a name="extend-active-directory-federation-services-ad-fs-to-azure"></a>Étendre les services de fédération Active Directory (AD FS) dans Azure

Cette architecture de référence implémente un réseau hybride sécurisé qui étend votre réseau local dans Azure et utilise [les services de fédération Active Directory (AD FS)][active-directory-federation-services] pour procéder à une authentification et une autorisation fédérées pour les composants en cours d’exécution dans Azure. [**Déployez cette solution**.](#deploy-the-solution)

[![0]][0]

*Téléchargez un [fichier Visio][visio-download] de cette architecture.*

AD FS peut être hébergé localement, mais si votre application est une application hybride dont certaines parties sont implémentées dans Azure, il peut être plus efficace de répliquer AD FS dans le cloud. 

Le schéma présente les scénarios suivants :

* Le code d’application d’une organisation partenaire accède à une application web hébergée au sein de votre réseau virtuel Azure.
* Un utilisateur externe enregistré dont les informations d’identification sont stockées dans Active Directory Domain Services (DS) accède à une application web hébergée au sein de votre réseau virtuel Azure.
* Un utilisateur connecté à votre réseau virtuel à l’aide d’un appareil autorisé exécute une application web hébergée au sein de votre réseau virtuel Azure.

Utilisations courantes de cette architecture :

* Les applications hybrides où les charges de travail s’exécutent en partie en local et dans Azure.
* Les solutions qui utilisent l’autorisation fédérée pour exposer les applications web aux organisations partenaires.
* Les systèmes accessibles depuis des navigateurs web s’exécutant en dehors du pare-feu de l’organisation.
* Les systèmes qui permettent aux utilisateurs d’accéder aux applications web en se connectant à partir d’appareils externes autorisés tels que des ordinateurs distants, des ordinateurs portables et d’autres appareils mobiles. 

Cette architecture de référence se concentre sur la *fédération passive* dans laquelle les serveurs de fédération décident comment et quand authentifier un utilisateur. L’utilisateur fournit des informations de connexion au démarrage de l’application. Ce mécanisme est le plus couramment utilisé par les navigateurs web et implique un protocole qui redirige le navigateur vers un site sur lequel l’utilisateur s’authentifie. AD FS prend également en charge la *fédération active*, où une application est chargée de fournir des informations d’identification sans intervention supplémentaire de l’utilisateur, mais ce scénario n’est pas couvert par cette architecture.

Pour plus d’informations, consultez [Choisir une solution pour intégrer l’environnement Active Directory local à Azure][considerations]. 

## <a name="architecture"></a>Architecture

Cette architecture étend l’implémentation décrite dans l’article [Extending AD DS to Azure][extending-ad-to-azure] (Étendre AD DS dans Azure). Elle contient les éléments suivants.

* **Sous-réseau AD DS**. Les serveurs AD DS sont contenus dans leur propre sous-réseau avec des règles de groupe de sécurité réseau agissant comme un pare-feu.

* **Serveurs AD DS**. Des contrôleurs de domaine s’exécutant en tant que machines virtuelles dans Azure. Ces serveurs fournissent l’authentification des identités locales au sein du domaine.

* **Sous-réseau AD FS**. Les serveurs AD FS se trouvent dans leur propre sous-réseau avec des règles de groupe de sécurité réseau agissant comme un pare-feu.

* **Serveurs AD FS**. Les serveurs AD FS fournissent l’authentification et l’autorisation fédérées. Dans cette architecture, ils effectuent les tâches suivantes :
  
  * Réception des jetons de sécurité contenant les revendications d’un serveur de fédération partenaire pour le compte d’un utilisateur partenaire. AD FS vérifie que les jetons sont valides avant de transmettre les revendications à l’application web s’exécutant dans Azure pour autoriser les demandes. 
  
    L’application web s’exécutant dans Azure est la *partie de confiance*. Le serveur de fédération partenaire doit émettre les revendications qui sont comprises par l’application web. Les serveurs de fédération partenaires sont appelés *partenaires de compte*, car ils envoient des demandes d’accès pour le compte de comptes authentifiés dans l’organisation partenaire. Les serveurs AD FS sont appelés *partenaires de ressource*, car ils fournissent l’accès aux ressources (l’application web).

  * Authentification et autorisation des requêtes entrantes émises par des utilisateurs externes exécutant un navigateur web ou un appareil qui a besoin d’accéder aux applications web, à l’aide des services AD DS et du [service Active Directory Device Registration][ADDRS].
    
  Les serveurs AD FS sont configurés en tant que batterie de serveurs accessible via un équilibreur de charge Azure. Cette implémentation améliore la disponibilité et l’extensibilité. Les serveurs AD FS ne sont pas exposés directement à Internet. Tout le trafic Internet est filtré à travers les serveurs proxy d’application web AD FS et une zone DMZ (également appelée réseau de périmètre).

  Pour plus d’informations sur le fonctionnement des services AD FS, consultez l’article [Vue d’ensemble des services AD FS][active-directory-federation-services-overview]. De plus, l’article [Déploiement des services AD FS dans Azure][adfs-intro] contient une présentation détaillée de la procédure d’implémentation.

* **Sous-réseau de proxy AD FS**. Les serveurs proxy AD FS peuvent être contenus dans leur propre sous-réseau, avec des règles de groupe de sécurité réseau assurant leur protection. Les serveurs localisés dans ce sous-réseau sont exposés à Internet via un ensemble d’appliances réseau virtuelles qui fournissent un pare-feu entre votre réseau virtuel Azure et Internet.

* **Serveurs proxy d’application web (WAP) AD FS**. Ces machines virtuelles agissent comme des serveurs AD FS pour les demandes entrantes émises par des appareils externes et des organisations partenaires. Les serveurs proxy d’application web agissent comme un filtre pour protéger les serveurs AD FS d’un accès direct à partir d’Internet. Comme avec les serveurs AD FS, le fait de déployer les serveurs proxy d’application web dans une batterie de serveurs avec équilibrage de charge vous offre une meilleure disponibilité et une plus grande extensibilité qu’en déployant une collection de serveurs autonomes.
  
  > [!NOTE]
  > Pour plus d’informations sur l’installation des serveurs proxy d’application web, consultez l’article [Installer et configurer le serveur proxy d’application web][install_and_configure_the_web_application_proxy_server]
  > 
  > 

* **Organisation partenaire**. Une organisation partenaire exécutant une application web qui demande l’accès à une application web s’exécutant dans Azure. Le serveur de fédération au niveau de l’organisation partenaire authentifie les requêtes localement et envoie les jetons de sécurité contenant les revendications aux services AD FS s’exécutant dans Azure. Dans Azure, AD FS valide les jetons de sécurité et, le cas échéant, transmet les revendications à l’application web s’exécutant dans Azure pour autoriser les jetons valides.
  
  > [!NOTE]
  > Vous pouvez également configurer un tunnel VPN à l’aide de la passerelle Azure pour fournir un accès direct à AD FS pour les partenaires de confiance. Les demandes émises par ces partenaires ne passent pas par les serveurs proxy d’application web.
  > 
  > 

Pour plus d’informations sur les parties de l’architecture qui ne sont pas liées à AD FS, consultez les rubriques suivantes :
- [Implémentation d’une architecture réseau hybride sécurisée dans Azure][implementing-a-secure-hybrid-network-architecture]
- [Implémentation d’une architecture réseau hybride sécurisée avec accès Internet dans Azure][implementing-a-secure-hybrid-network-architecture-with-internet-access]
- [Implémentation d’une architecture réseau hybride sécurisée avec identités Active Directory dans Azure][extending-ad-to-azure].


## <a name="recommendations"></a>Recommandations

Les recommandations suivantes s’appliquent à la plupart des scénarios. Suivez ces recommandations, sauf si vous avez un besoin spécifique qui vous oblige à les ignorer. 

### <a name="vm-recommendations"></a>Recommandations pour les machines virtuelles

Créez des machines virtuelles avec suffisamment de ressources pour gérer le volume de trafic attendu. Prenez comme point de départ la taille des machines existantes qui hébergent AD FS localement. Surveillez l’utilisation des ressources. Vous pouvez redimensionner les machines virtuelles et diminuer leur taille si nécessaire.

Suivez les recommandations de l’article [Exécution d’une machine virtuelle Windows sur Azure][vm-recommendations].

### <a name="networking-recommendations"></a>Recommandations pour la mise en réseau

Configurez l’interface réseau pour chacune des machines virtuelles hébergeant des serveurs AD FS et WAP avec des adresses IP privées statiques.

N’attribuez pas d’adresses IP publiques aux machines virtuelles AD FS. Pour plus d’informations, consultez la section Considérations relatives à la sécurité.

Définissez l’adresse IP des serveurs de nom de domaine (DNS) par défaut et secondaires pour les interfaces réseau de chaque machine virtuelle AD FS et WAP de manière à référencer les machines virtuelles Active Directory DS. Les machines virtuelles Active Directory DS doivent exécuter un serveur DNS. Cette étape est nécessaire pour que chaque machine virtuelle puisse rejoindre le domaine.

### <a name="ad-fs-availability"></a>Disponibilité des services AD FS 

Créez une batterie AD FS comprenant au moins deux serveurs pour augmenter la disponibilité du service. Utilisez des comptes de stockage différents pour chaque machine virtuelle AD FS dans la batterie de serveurs. Cette approche permet de s’assurer que la batterie de serveurs restera en partie accessible même en cas de panne au niveau d’un compte de stockage spécifique.

> [!IMPORTANT]
> Nous vous recommandons d’utiliser des [disques managés](/azure/storage/storage-managed-disks-overview). Les disques managés ne nécessitent pas de compte de stockage. Il vous suffit de spécifier leur taille et leur type, puis de les déployer avec une haute disponibilité. Notre [architecture de référence](/azure/architecture/reference-architectures/) ne permet pas encore de déployer des disques managés, mais les [blocs de construction de modèle](https://github.com/mspnp/template-building-blocks/wiki) seront mis à jour pour prendre en charge le déploiement de disques managés dans la version 2.

Créez des groupes à haute disponibilité Azure distincts pour les machines virtuelles AD FS et WAP. Chaque groupe doit contenir au moins deux machines virtuelles. Chaque groupe à haute disponibilité doit avoir au moins deux domaines de mise à jour et deux domaines d’erreur.

Configurez les équilibreurs de charge pour les machines virtuelles AD FS et WAP tel que suit :

* Utilisez un équilibreur de charge Azure pour fournir un accès externe aux machines virtuelles WAP et un équilibreur de charge interne pour répartir la charge sur les serveurs AD FS au sein de la batterie de serveurs.
* Transmettez uniquement le trafic apparaissant sur le port 443 (HTTPS) aux serveurs AD FS/WAP.
* Attribuez une adresse IP statique à l’équilibreur de charge.
* Créez une sonde d’intégrité à l’aide du protocole TCP plutôt que HTTPS. Vous pouvez effectuer un test ping sur le port 443 pour vérifier le bon fonctionnement d’un serveur AD FS.
  
  > [!NOTE]
  > Les serveurs AD FS utilisent le protocole d’indication du nom de serveur (SNI). Par conséquent, tout test d’intégrité effectué à l’aide d’un point de terminaison HTTPS à partir de l’équilibreur de charge échouera.
  > 
  > 
* Ajoutez un enregistrement DNS *A* au domaine pour l’équilibreur de charge AD FS. Spécifiez l’adresse IP de l’équilibreur de charge et donnez-lui un nom dans le domaine (par exemple, adfs.contoso.com). Il s’agit du nom que les clients et les serveurs WAP utiliseront pour accéder à la batterie de serveurs AD FS.

### <a name="ad-fs-security"></a>Sécurité des services AD FS 

Empêchez l’exposition directe des serveurs AD FS à Internet. Les serveurs AD FS sont des ordinateurs joints à un domaine qui sont autorisés à octroyer des jetons de sécurité. Si un serveur est compromis, un utilisateur malveillant peut émettre des jetons d’accès complet à toutes les applications web et à tous les serveurs de fédération qui sont protégés par AD FS. Si votre système doit gérer des demandes d’utilisateurs externes ne se connectant pas à partir de sites de partenaires de confiance, utilisez des serveurs WAP pour gérer ces demandes. Pour plus d’informations, consultez l’article [Where to Place a Federation Server Proxy][where-to-place-an-fs-proxy] (Où placer un serveur proxy de fédération).

Placez les serveurs AD FS et les serveurs WAP dans des sous-réseaux distincts avec leur propre pare-feu. Vous pouvez utiliser des règles de groupe de sécurité réseau pour définir les règles du pare-feu. Si vous avez besoin d’une protection plus complète, vous pouvez implémenter un périmètre de sécurité supplémentaire autour des serveurs à l’aide d’une paire de sous-réseaux et d’appliances réseau virtuelles, comme décrit dans le document [Implémentation d’une architecture réseau hybride sécurisée avec accès Internet dans Azure][implementing-a-secure-hybrid-network-architecture-with-internet-access]. Tous les pare-feu doivent autoriser le trafic sur le port 443 (HTTPS).

Désactivez l’accès en connexion directe aux serveurs AD FS et WAP. Seul le personnel DevOps doit être en mesure de s’y connecter.

Ne joignez pas les serveurs WAP au domaine.

### <a name="ad-fs-installation"></a>Installation des services AD FS 

L’article [Deploying a Federation Server Farm][Deploying_a_federation_server_farm] (Déploiement d’une batterie de serveurs de fédération) fournit des instructions détaillées pour l’installation et la configuration des services AD FS. Avant de configurer le premier serveur AD FS dans la batterie de serveurs, effectuez les tâches suivantes :

1. Obtenez un certificat approuvé publiquement pour l’authentification du serveur. Le *nom de l’objet* doit contenir le nom utilisé par les clients pour accéder au service de fédération. Cela peut être le nom DNS enregistré pour l’équilibreur de charge, par exemple, *adfs.contoso.com* (pour des raisons de sécurité, évitez d’utiliser des noms comprenant des caractères génériques tels que **.contoso.com*). Utilisez le même certificat sur toutes les machines virtuelles du serveur AD FS. Vous pouvez acheter un certificat auprès d’une autorité de certification de confiance, mais si votre organisation utilise les services de certificats Active Directory, vous pouvez également créer votre propre certificat. 
   
    *L’autre nom de l’objet* est utilisé par le service DRS (Device Registration Service) pour activer l’accès à partir d’appareils externes. Il doit être au format *enterpriseregistration.contoso.com*.
   
    Pour plus d’informations, consultez l’article [Obtain and Configure a Secure Sockets Layer (SSL) Certificate for AD FS][adfs_certificates] (Obtenir et configurer un certificat Secure Sockets Layer (SSL) pour AD FS).

2. Sur le contrôleur de domaine, générez une nouvelle clé racine pour le service de distribution de clés. Définissez l’heure effective sur l’heure actuelle à laquelle vous aurez soustrait 10 heures (cette configuration réduit le délai pouvant intervenir lors de la distribution et de la synchronisation des clés dans le domaine). Cette étape est nécessaire pour prendre en charge la création du compte de service de groupe qui est utilisé pour exécuter le service AD FS. La commande PowerShell suivante montre un exemple de la procédure à suivre :
   
    ```powershell
    Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
    ```

3. Ajoutez chaque machine virtuelle du serveur AD FS au domaine.

> [!NOTE]
> Pour installer AD FS, le contrôleur de domaine jouant le rôle de FSMO (Flexible Single Master Operation) de l’émulateur de contrôleur de domaine principal pour le domaine doit être en cours d’exécution et accessible depuis les machines virtuelles AD FS. <<RBC: Is there a way to make this less repetitive?>>
> 
> 

### <a name="ad-fs-trust"></a>Approbation AD FS 

Établissez une approbation de fédération entre votre installation AD FS et les serveurs de fédération des organisations partenaires. Configurez les paramètres de mappage et de filtrage de revendications tel que requis. 

* Le personnel DevOps de chaque organisation partenaire doit ajouter une approbation de partie de confiance pour les applications web accessibles via vos serveurs AD FS.
* Le personnel DevOps de votre organisation doit configurer une approbation de fournisseur de revendications pour que vos serveurs AD FS puissent approuver les revendications émanant des organisations partenaires.
* Le personnel DevOps de votre organisation doit également configurer AD FS pour transmettre les revendications aux applications web de votre organisation.
  
Pour plus d’informations, consultez l’article [Establishing Federation Trust][establishing-federation-trust] (Établir une approbation de fédération).

Publiez les applications web de votre organisation et rendez-les accessibles aux partenaires externes en activant la préauthentification via les serveurs proxy d’application web (WAP). Pour plus d’informations, consultez l’article [Publish Applications using AD FS Preauthentication][publish_applications_using_AD_FS_preauthentication] (Publier des applications avec la préauthentification AD FS)

AD FS prend en charge l’augmentation et la transformation des jetons. Azure Active Directory ne fournit pas cette fonctionnalité. Avec AD FS, lorsque vous configurez des relations d’approbation, vous pouvez :

* Configurer des transformations de revendication pour les règles d’autorisation. Par exemple, vous pouvez mapper une sécurité de groupe à partir d’une représentation utilisée par une organisation partenaire non-Microsoft sur un élément que le service Active Directory DS peut autoriser dans votre organisation.
* Convertir des revendications d’un format vers un autre. Par exemple, vous pouvez effectuer un mappage de SAML 2.0 vers SAML 1.1 si votre application prend uniquement en charge les revendications au format SAML 1.1.

### <a name="ad-fs-monitoring"></a>Analyse des services AD FS 

Le [Pack d’administration Microsoft System Center pour les services de fédération Active Directory 2012 R2][oms-adfs-pack] fournit une analyse proactive et réactive de votre déploiement AD FS pour le serveur de fédération. Ce pack d’administration surveille :

* Les événements que le service AD FS consigne dans ses journaux d’événements.
* Les données de performance collectées par les compteurs de performances AD FS. 
* L’intégrité globale du système AD FS et des applications web (parties de confiance). Enfin, le pack fournit des alertes pour les problèmes critiques et les avertissements. 

## <a name="scalability-considerations"></a>Considérations relatives à l’extensibilité

Les considérations suivantes, dont tous les détails sont présentés dans l’article [Planifier votre déploiement AD FS][plan-your-adfs-deployment], vous donnent des conseils pour redimensionner les batteries de serveurs AD FS :

* Si vous avez moins de 1 000 utilisateurs, ne créez pas de serveurs dédiés, mais installez plutôt AD FS sur chacun des serveurs Active Directory DS dans le cloud. Assurez-vous de disposer d’au moins deux serveurs Active Directory DS pour garantir la disponibilité. Créez un seul serveur WAP.
* Si vous avez entre 1 000 et 15 000 utilisateurs, créez deux serveurs AD FS dédiés et deux serveurs WAP dédiés.
* Si vous avez entre 15 000 et 60 000 utilisateurs, créez entre trois et cinq serveurs AD FS dédiés et au moins deux serveurs WAP dédiés.

Ces considérations supposent que vous utilisez des machines virtuelles doubles à quatre cœurs (Standard D4_v2 ou taille supérieure) dans Azure.

Si vous utilisez la base de données interne Windows pour stocker les données de configuration AD FS, vous ne pouvez ajouter que huit serveurs AD FS dans la batterie de serveurs. Si vous pensez que vous aurez besoin de davantage de serveurs à l’avenir, utilisez SQL Server. Pour plus d’informations, consultez l’article [The Role of the AD FS Configuration Database][adfs-configuration-database] (Rôle de la base de données de configuration AD FS).

## <a name="availability-considerations"></a>Considérations relatives à la disponibilité

Vous pouvez utiliser SQL Server ou la base de données interne Windows pour conserver les informations de configuration AD FS. La base de données interne Windows fournit une redondance de base. Les modifications sont écrites directement dans une seule des bases de données AD FS du cluster AD FS, tandis que les autres serveurs utilisent la réplication par réception pour mettre à jour leurs bases de données. SQL Server peut fournir une redondance de base de données totale et une haute disponibilité grâce à la mise en miroir ou au clustering de basculement.

## <a name="manageability-considerations"></a>Considérations relatives à la facilité de gestion

Le personnel DevOps doit savoir effectuer les tâches suivantes :

* Gérer les serveurs de fédération, y compris la batterie de serveurs AD FS, la stratégie d’approbation au niveau des serveurs de fédération et les certificats utilisés par les services de fédération.
* Gérer les serveurs WAP, y compris la batterie de serveurs WAP et les certificats.
* Gérer les applications web, notamment configurer les parties de confiance, les méthodes d’authentification et les mappages de revendications.
* Sauvegarder les composants des services AD FS.

## <a name="security-considerations"></a>Considérations relatives à la sécurité

AD FS utilise le protocole HTTPS. Par conséquent, assurez-vous que les règles du groupe de sécurité réseau pour le sous-réseau contenant les machines virtuelles de niveau web autorisent les requêtes HTTPS. Ces requêtes peuvent provenir du réseau local, des sous-réseaux contenant la couche web, la couche métier, la couche Données, la zone DMZ privée, la zone DMZ publique et le sous-réseau contenant les serveurs AD FS.

Envisagez d’utiliser un ensemble d’appliances réseau virtuelles qui enregistrent tous les détails du trafic transitant par votre réseau virtuel à des fins d’audit.

## <a name="deploy-the-solution"></a>Déployer la solution

Vous disposez d’une solution sur [GitHub][github] pour déployer cette architecture de référence. Vous avez besoin de la dernière version de [l’interface de ligne de commande Azure][azure-cli] pour exécuter le script PowerShell qui déploie la solution. Pour déployer l’architecture de référence, effectuez les étapes suivantes :

1. Téléchargez ou clonez le dossier de solution à partir de [GitHub][github] sur votre ordinateur local.

2. Ouvrez l’interface de ligne de commande Azure et accédez au dossier de solution local.

3. Exécutez la commande suivante :
   
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```
   
    Remplacez `<subscription id>` par l’identifiant de votre abonnement Azure.
   
    Pour `<location>`, spécifiez une région Azure, telle que `eastus` ou `westus`.
   
    Le paramètre `<mode>` contrôle la granularité du déploiement, et peut prendre l’une des valeurs suivantes :
   
   * `Onpremise` : déploie un environnement local simulé. Vous pouvez utiliser ce déploiement à des fins de test et d’expérience si vous ne disposez pas d’un réseau local, ou si vous souhaitez tester cette architecture de référence sans modifier la configuration de votre réseau local existant.
   * `Infrastructure` : déploie l’infrastructure du réseau virtuel et le serveur de rebond.
   * `CreateVpn` : déploie une passerelle réseau virtuelle Azure et la connecte au réseau local simulé.
   * `AzureADDS` : déploie les machines virtuelles faisant office de serveurs Active Directory DS, déploie Active Directory sur ces machines virtuelles et créé le domaine dans Azure.
   * `AdfsVm` : déploie les machines virtuelles AD FS et les joint au domaine dans Azure.
   * `PublicDMZ` : déploie la zone DMZ publique dans Azure.
   * `ProxyVm` : déploie les machines virtuelles proxy AD FS et les joint au domaine dans Azure.
   * `Prepare` : déploie tous les déploiements précédents. **Il s’agit de l’option recommandée si vous créez un nouveau déploiement et que vous ne disposez pas encore d’une infrastructure locale.** 
   * `Workload` : déploie de manière facultative les machines virtuelles des couches web, métier et données, ainsi que le réseau sous-jacent. Non inclus dans le mode de déploiement `Prepare`.
   * `PrivateDMZ` : déploie de manière facultative la zone DMZ privée dans Azure en amont des machines virtuelles `Workload` déployées ci-dessus. Non inclus dans le mode de déploiement `Prepare`.

4. Attendez la fin du déploiement. Si vous avez choisi l’option `Prepare`, le déploiement prend plusieurs heures et se termine avec le message `Preparation is completed. Please install certificate to all AD FS and proxy VMs.`

5. Redémarrez le serveur de rebond (*ra-adfs-mgmt-vm1* dans le groupe *ra-adfs-security-rg*) pour activer ses paramètres DNS.

6. [Récupérez un certificat SSL pour AD FS][adfs_certificates] et installez ce certificat sur les machines virtuelles AD FS. Notez que vous pouvez vous y connecter via le serveur de rebond. Les adresses IP sont *10.0.5.4* et *10.0.5.5*. Le nom d’utilisateur par défaut est *contoso\testuser*, et le mot de passe est *AweSome@PW*.
   
   > [!NOTE]
   > À ce stade, les commentaires dans le script Deploy-ReferenceArchitecture.ps1 fournissent des instructions détaillées pour créer une autorité et un certificat de test auto-signé à l’aide de la commande `makecert`. Toutefois, effectuez uniquement ces étapes dans le cadre d’un **test** et n’utilisez pas les certificats générés par MakeCert dans un environnement de production.
   > 
   > 

7. Exécutez la commande PowerShell suivante pour déployer la batterie de serveurs AD FS :
   
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> Adfs
    ``` 

8. Sur le serveur de rebond, accédez à `https://adfs.contoso.com/adfs/ls/idpinitiatedsignon.htm` pour tester l’installation AD FS (vous pouvez recevoir un avertissement de certificat que vous pouvez ignorer dans le cadre de ce test). Vérifiez que la page de connexion Contoso Corporation s’affiche. Connectez-vous avec le nom d’utilisateur *contoso\testuser* et le mot de passe *AweS0me@PW*.

9. Installez le certificat SSL sur les machines virtuelles proxy AD FS. Les adresses IP sont *10.0.6.4* et *10.0.6.5*.

10. Exécutez la commande PowerShell suivante pour déployer le premier serveur proxy AD FS :
   
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> Proxy1
    ```

11. Suivez les instructions qui s’affichent dans le script pour tester l’installation du premier serveur proxy.

12. Exécutez la commande PowerShell suivante pour déployer le second serveur proxy :
    
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> Proxy2
    ```

13. Suivez les instructions qui s’affichent dans le script pour tester la configuration complète des serveurs proxy.

## <a name="next-steps"></a>étapes suivantes

* En savoir plus sur [Azure Active Directory][aad].
* En savoir plus sur [Azure Active Directory B2C][aadb2c].

<!-- links -->
[extending-ad-to-azure]: adds-extend-domain.md

[vm-recommendations]: ../virtual-machines-windows/single-vm.md
[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md
[hybrid-azure-on-prem-vpn]: ../hybrid-networking/vpn.md

[azure-cli]: /azure/azure-resource-manager/xplat-cli-azure-resource-manager
[DRS]: https://technet.microsoft.com/library/dn280945.aspx
[where-to-place-an-fs-proxy]: https://technet.microsoft.com/library/dd807048.aspx
[ADDRS]: https://technet.microsoft.com/library/dn486831.aspx
[plan-your-adfs-deployment]: https://msdn.microsoft.com/library/azure/dn151324.aspx
[ad_network_recommendations]: #network_configuration_recommendations_for_AD_DS_VMs
[adfs_certificates]: https://technet.microsoft.com/library/dn781428(v=ws.11).aspx
[create_service_account_for_adfs_farm]: https://technet.microsoft.com/library/dd807078.aspx
[adfs-configuration-database]: https://technet.microsoft.com/library/ee913581(v=ws.11).aspx
[active-directory-federation-services]: https://technet.microsoft.com/windowsserver/dd448613.aspx
[security-considerations]: #security-considerations
[recommendations]: #recommendations
[active-directory-federation-services-overview]: https://technet.microsoft.com/library/hh831502(v=ws.11).aspx
[establishing-federation-trust]: https://blogs.msdn.microsoft.com/alextch/2011/06/27/establishing-federation-trust/
[Deploying_a_federation_server_farm]:  https://azure.microsoft.com/documentation/articles/active-directory-aadconnect-azure-adfs/
[install_and_configure_the_web_application_proxy_server]: https://technet.microsoft.com/library/dn383662.aspx
[publish_applications_using_AD_FS_preauthentication]: https://technet.microsoft.com/library/dn383640.aspx
[managing-adfs-components]: https://technet.microsoft.com/library/cc759026.aspx
[oms-adfs-pack]: https://www.microsoft.com/download/details.aspx?id=41184
[azure-powershell-download]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[aad]: https://azure.microsoft.com/documentation/services/active-directory/
[aadb2c]: https://azure.microsoft.com/documentation/services/active-directory-b2c/
[adfs-intro]: /azure/active-directory/active-directory-aadconnect-azure-adfs
[github]: https://github.com/mspnp/reference-architectures/tree/master/identity/adfs
[adfs_certificates]: https://technet.microsoft.com/library/dn781428(v=ws.11).aspx
[considerations]: ./considerations.md
[visio-download]: https://archcenter.azureedge.net/cdn/identity-architectures.vsdx
[0]: ./images/adfs.png "Architecture réseau hybride sécurisée avec Active Directory"
