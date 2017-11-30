---
title: "Implémentation d’une zone DMZ entre Azure et Internet"
description: "Comment implémenter une architecture réseau hybride sécurisée avec accès Internet dans Azure."
author: telmosampaio
ms.date: 11/23/2016
pnp.series.title: Network DMZ
pnp.series.next: nva-ha
pnp.series.prev: secure-vnet-hybrid
cardTitle: DMZ between Azure and the Internet
ms.openlocfilehash: 372d5bb0fc0e3c272843e062210dec5c15b2b78a
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2017
---
# <a name="dmz-between-azure-and-the-internet"></a>Zone DMZ entre Azure et Internet

Cette architecture de référence montre un réseau sécurisé hybride qui étend un réseau local sur Azure et accepte également le trafic Internet. 

[![0]][0] 

*Téléchargez un [fichier Visio][visio-download] de cette architecture.*

Cette architecture de référence étend l’architecture décrite dans [Implementing a DMZ between Azure and your on-premises datacenter][implementing-a-secure-hybrid-network-architecture] (Implémenter une zone DMZ entre Azure et votre centre de données local). Elle ajoute une zone DMZ qui gère le trafic Internet, en plus de la zone DMZ privée qui gère le trafic issu du réseau local 

Utilisations courantes de cette architecture :

* Les applications hybrides où les charges de travail s’exécutent en partie en local et dans Azure.
* Infrastructure Azure qui achemine le trafic entrant en provenance du réseau local et d’Internet.

## <a name="architecture"></a>Architecture

L’architecture est constituée des composants suivants :

* **Adresse IP publique**. Adresse IP publique du point de terminaison public. Les utilisateurs externes connectés à Internet peuvent accéder au système via cette adresse.
* **Appliance virtuelle réseau**. Cette architecture inclut un pool distinct d’appliances virtuelles réseau pour le trafic provenant d’Internet.
* **Équilibreur de charge Azure**. Toutes les demandes entrantes provenant d’Internet passent par l’équilibreur de charge et sont distribuées aux appliances virtuelles réseau dans la zone DMZ publique.
* **Sous-réseau entrant de zone DMZ publique**. Ce sous-réseau accepte les requêtes issues de l’équilibreur de charge Azure. Les requêtes entrantes sont transmises à une des appliances virtuelles réseau de la zone DMZ publique.
* **Sous-réseau sortant de zone DMZ publique**. Les requêtes approuvées par l’appliance virtuelle réseau passent à travers ce sous-réseau vers l’équilibreur de charge interne pour la couche web.

## <a name="recommendations"></a>Recommandations

Les recommandations suivantes s’appliquent à la plupart des scénarios. Suivez ces recommandations, sauf si vous avez un besoin spécifique qui vous oblige à les ignorer. 

### <a name="nva-recommendations"></a>Recommandations relatives aux appliances virtuelles réseau

Utilisez un ensemble d’appliances virtuelles réseau pour le trafic provenant d’Internet, et un autre ensemble pour le trafic émanant du niveau local. L’utilisation d’un seul ensemble d’appliances virtuelles réseau pour ces deux types de trafics réseau constitue un risque pour la sécurité, car elle n’offre aucun périmètre de sécurité entre les deux. L’utilisation d’appliances virtuelles réseau distinctes simplifie la vérification des règles de sécurité et permet d’identifier clairement les règles auxquelles correspondent les différentes requêtes réseau entrantes. Un ensemble d’appliances virtuelles réseau implémente les règles pour le trafic Internet uniquement, et l’autre celles du trafic local uniquement.

Incluez une appliance virtuelle réseau de couche 7 pour mettre fin aux connexions d’application au niveau de l’appliance virtuelle réseau et assurer la compatibilité avec les couches principales. Cela garantit une connectivité symétrique où le trafic de réponse issu des couches principales est renvoyé via l’appliance virtuelle réseau.  

### <a name="public-load-balancer-recommendations"></a>Recommandations relatives à l’équilibreur de charge public

À des fins d’évolutivité et de disponibilité, déployez les appliances virtuelles réseau de la zone DMZ publique dans un [groupe à haute disponibilité][availability-set] et utilisez un [équilibreur de charge accessible sur Internet][load-balancer]pour répartir les requêtes Internet sur les appliances virtuelles réseau du groupe à haute disponibilité.  

Configurer l’équilibreur de charge pour accepter les requêtes uniquement sur les ports nécessaires au trafic Internet. Par exemple, limitez les requêtes HTTP entrantes vers le port 80 et les requêtes HTTPS entrantes vers le port 443.

## <a name="scalability-considerations"></a>Considérations relatives à l’extensibilité

Même si votre architecture requiert initialement une appliance virtuelle réseau unique dans la zone DMZ publique, nous recommandons de placer un équilibreur de charge devant la zone DMZ publique dès le début. Cela facilite l’évolution ultérieure vers plusieurs appliances virtuelles réseau, si nécessaire.

## <a name="availability-considerations"></a>Considérations relatives à la disponibilité

L’équilibreur de charge accessible sur Internet requiert que chaque appliance virtuelle réseau de la zone DMZ du sous-réseau entrant de la zone publique DMZ implémente une [sonde d’intégrité][lb-probe]. Une sonde d’intégrité qui ne répond pas sur ce point de terminaison est considérée comme étant indisponible. Par conséquent, l’équilibreur de charge doit diriger les requêtes vers d’autres appliances virtuelles réseau du même groupe à haute disponibilité. Notez que si toutes les appliances virtuelles réseau ne répondent pas, votre application échoue. Il est donc important de configurer l’analyse afin d’alerter DevOps lorsque le nombre d’instances d’appliances virtuelles réseau saines tombe en dessous d’un seuil défini.

## <a name="manageability-considerations"></a>Considérations relatives à la facilité de gestion

L’ensemble de l’analyse et de la gestion des appliances virtuelles réseau de la zone publique DMZ doit être effectuée par le jumpbox dans le sous-réseau de gestion. Comme indiqué dans [Implementing a DMZ between Azure and your on-premises datacenter][implementing-a-secure-hybrid-network-architecture] (Implémenter une zone DMZ entre Azure et votre centre de données local), définissez un itinéraire de réseau unique à partir du réseau local via la passerelle menant au jumpbox afin de restreindre accès.

Si la connectivité de la passerelle de votre réseau local vers Azure est interrompue, vous pouvez toujours atteindre le jumpbox en déployant une adresse IP publique, en l’ajoutant au jumpbox et en vous connectant à partir d’Internet.

## <a name="security-considerations"></a>Considérations relatives à la sécurité

Cette architecture de référence implémente plusieurs niveaux de sécurité :

* L’équilibreur de charge accessible sur Internet dirige les requêtes vers les appliances virtuelles réseau au sein du sous-réseau DMZ public et uniquement sur les ports nécessaires à l’application.
* Les règles du groupe de sécurité réseau pour les sous-réseaux de la zone DMZ publique entrants et sortants empêchent la compromission des appliances virtuelles réseau par le blocage des requêtes se trouvant en dehors des règles du groupe de sécurité réseau.
* La configuration du routage NAT pour les appliances virtuelles réseau dirige les requêtes entrantes sur les ports 80 et 443 vers l’équilibreur de charge de la couche web, mais ignore les requêtes sur tous les autres ports.

Vous devez journaliser toutes les requêtes entrantes sur tous les ports. Effectuez un audit régulier des journaux en accordant une attention particulière aux requêtes qui se trouvent en dehors des paramètres attendus, comme cela peut indiquer des tentatives d’intrusion.

## <a name="solution-deployment"></a>Déploiement de la solution

Un déploiement pour une architecture de référence implémentant ces recommandations est disponible sur [GitHub][github-folder]. L’architecture de référence peut être déployée avec Windows ou les machines virtuelles Linux en procédant comme suit :

1. Cliquez sur le bouton ci-dessous :<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2FvirtualNetwork.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Une fois le lien ouvert dans le portail Azure, vous devez entrer des valeurs pour certains paramètres :
   * Le nom du **groupe de ressources** est déjà défini dans le fichier de paramètres ; sélectionnez **Créer nouveau** et entrez `ra-public-dmz-network-rg` dans la zone de texte.
   * Sélectionnez la région à partir de la zone déroulante **Emplacement**.
   * Ne modifiez pas les zones de texte **Template Root Uri** (Uri racine de modèle) ou **Parameter Root Uri** (Uri racine de paramètre).
   * Sélectionnez le **type de système d’exploitation** à partir de la zone déroulante, **windows** ou **linux**.
   * Passez en revue les termes et conditions, puis cochez la case **J’accepte les termes et conditions mentionnés ci-dessus**.
   * Cliquez sur le bouton **Acheter**.
3. Attendez la fin du déploiement.
4. Cliquez sur le bouton ci-dessous :<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2Fworkload.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. Une fois le lien ouvert dans le portail Azure, vous devez entrer des valeurs pour certains paramètres :
   * Le nom du **groupe de ressources** est déjà défini dans le fichier de paramètres ; sélectionnez **Créer nouveau** et entrez `ra-public-dmz-wl-rg` dans la zone de texte.
   * Sélectionnez la région à partir de la zone déroulante **Emplacement**.
   * Ne modifiez pas les zones de texte **Template Root Uri** (Uri racine de modèle) ou **Parameter Root Uri** (Uri racine de paramètre).
   * Passez en revue les termes et conditions, puis cochez la case **J’accepte les termes et conditions mentionnés ci-dessus**.
   * Cliquez sur le bouton **Acheter**.
6. Attendez la fin du déploiement.
7. Cliquez sur le bouton ci-dessous :<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2Fsecurity.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
8. Une fois le lien ouvert dans le portail Azure, vous devez entrer des valeurs pour certains paramètres :
   * Le nom du **Groupe de ressources** est déjà défini dans le fichier de paramètres. Sélectionnez **Existant** et entrez `ra-public-dmz-network-rg` dans la zone de texte.
   * Sélectionnez la région à partir de la zone déroulante **Emplacement**.
   * Ne modifiez pas les zones de texte **Template Root Uri** (Uri racine de modèle) ou **Parameter Root Uri** (Uri racine de paramètre).
   * Passez en revue les termes et conditions, puis cochez la case **J’accepte les termes et conditions mentionnés ci-dessus**.
   * Cliquez sur le bouton **Acheter**.
9. Attendez la fin du déploiement.
10. Les fichiers de paramètres incluent un nom d’utilisateur administrateur et un mot de passe codés en dur pour toutes les machines virtuelles, et il est vivement recommandé de modifier immédiatement ces deux éléments. Pour chaque machine virtuelle du déploiement, sélectionnez-la dans le portail Azure, puis cliquez sur **Réinitialiser le mot de passe** dans le panneau **Support + dépannage**. Sélectionnez **Réinitialiser le mot de passe** dans la zone déroulante **Mode**, puis sélectionnez de nouveaux **Nom d’utilisateur** et **Mot de passe**. Cliquez sur le bouton **Mise à jour** pour enregistrer.


[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-dmz

[implementing-a-secure-hybrid-network-architecture]: ./secure-vnet-hybrid.md
[iptables]: https://help.ubuntu.com/community/IptablesHowTo
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview
[load-balancer]: /azure/load-balancer/load-balancer-Internet-overview
[network-security-group]: /azure/virtual-network/virtual-networks-nsg

[visio-download]: https://archcenter.azureedge.net/cdn/dmz-reference-architectures.vsdx


[0]: ./images/dmz-public.png "Architecture réseau hybride sécurisée"