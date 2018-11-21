---
title: Bureaux virtuels Linux avec Citrix
description: Générez un environnement VDI pour les bureaux Linux à l’aide de Citrix sur Azure.
author: miguelangelopereira
ms.date: 09/12/2018
ms.openlocfilehash: 383642b05926c5a09abf0b2f95fef10539d95aec
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610649"
---
# <a name="linux-virtual-desktops-with-citrix"></a>Bureaux virtuels Linux avec Citrix

Cet exemple de scénario s’applique à tous les secteurs qui ont besoin d’une infrastructure de bureaux virtuels (VDI) pour les postes de travail Linux. Le sigle VDI désigne le processus d’exécution d’un bureau utilisateur dans une machine virtuelle hébergée sur un serveur dans le centre de données. Dans ce scénario, le client a choisi d’utiliser une solution Citrix pour satisfaire ses besoins en matière de VDI.

Les entreprises disposent souvent d’environnements hétérogènes avec des collaborateurs qui utilisent différents appareils et systèmes d’exploitation. Il peut être difficile de fournir un accès uniforme aux applications tout en assurant la sécurité de l’environnement. Une solution VDI pour les postes de travail Linux vous permet de fournir un accès aux utilisateurs, quel que soit leur appareil ou leur système d’exploitation.

Parmi les avantages de cet exemple de solution, mentionnons les suivants :
* Le retour sur investissement est plus élevé avec des bureaux virtuels Linux partagés en permettant à un plus grand nombre d’utilisateurs d’accéder à la même infrastructure. En regroupant les ressources sur un environnement VDI centralisé, les appareils des utilisateurs n’ont pas besoin d’être aussi puissants.
* Les performances sont les mêmes, quel que soit l’appareil de l’utilisateur.
* Les utilisateurs peuvent accéder aux applications Linux depuis n’importe quel appareil (y compris les appareils qui ne fonctionnent pas sous Linux).
* Les données sensibles de tous les collaborateurs répartis peuvent être sécurisées dans le centre de données Azure.

## <a name="relevant-use-cases"></a>Cas d’utilisation appropriés

Pensez à ce scénario pour les cas d’usage suivants :

* Accès sécurisé à des postes de travail VDI Linux spécialisés et stratégiques à partir d’appareils fonctionnant ou non sous Linux

## <a name="architecture"></a>Architecture

[![](./media/azure-citrix-sample-diagram.png "Diagramme d’architecture")](./media/azure-citrix-sample-diagram.png#lightbox)

Cet exemple de scénario illustre comment permettre au réseau d’entreprise d’accéder aux bureaux virtuels Linux :

* Une connexion ExpressRoute est établie entre l’environnement local et Azure afin d’assurer la rapidité et la fiabilité de la connectivité avec le cloud.
* Solution Citrix XenDeskop déployée pour une infrastructure VDI.
* CitrixVDA fonctionne sur Ubuntu (ou une autre distribution compatible).
* Les groupes de sécurité réseau Azure appliquent les ACL réseau appropriées.
* Citrix ADC (NetScaler) publie et équilibre le chargement de tous les services Citrix.
* Les services de domaine Active Directory sont utilisés pour accéder aux serveurs Citrix. Les serveurs VDA ne sont pas rattachés au domaine.
* Azure Hybrid File Sync permet de partager le stockage sur l’ensemble de la solution. Par exemple, il peut être utilisé dans des solutions distantes/à domicile.

Dans ce scénario, les références SKU suivantes sont utilisées :

- Citrix ADC (NetScaler) : 2 x D4sv3 avec [image Paiement à l’utilisation de NetScaler 12.0 VPX Standard Edition 200 MBPS ](https://azuremarketplace.microsoft.com/pt-br/marketplace/apps/citrix.netscalervpx-120?tab=PlansAndPrice)
- Citrix License Server : 1 x D2s v3
- Citrix VDA : 4 x D8s v3
- Citrix StoreFront : 2 x D2s v3
- Citrix Delivery Controller : 2 x D2s v3
- Contrôleurs de domaine : 2 x D2sv3
- Serveurs de fichiers Azure : 2 x D2sv3

> [!NOTE]
> Toutes les licences (autres que NetScaler) sont sous licence BYOL (apportez votre propre licence)

### <a name="components"></a>Composants

- Le [réseau virtuel Azure](/azure/virtual-network/virtual-networks-overview) permet à de nombreuses ressources, telles que les machines virtuelles de communiquer en toute sécurité entre elles, avec Internet et avec les réseaux locaux. Les réseaux virtuels fournissent un isolement et une segmentation, ils filtrent et acheminent le trafic et ils autorisent une connexion entre des emplacements. Dans ce scénario, un réseau virtuel est utilisé pour toutes les ressources.
- Les [groupes de sécurité réseau Azure](/azure/virtual-network/security-overview) contiennent une liste de règles de sécurité qui autorisent ou refusent le trafic réseau entrant ou sortant en fonction de l’adresse IP source ou de destination, du port et du protocole. Les réseaux virtuels dans ce scénario sont sécurisés avec des règles de groupe de sécurité réseau qui limitent le flux du trafic entre les composants d’application.
- [Azure Load Balancer](/azure/application-gateway/overview) distribue le trafic entrant en fonction des règles et des sondes d’intégrité. Un équilibreur de charge offre une latence faible et un débit élevé, et peut augmenter l’échelle jusqu’à des millions de flux pour toutes les applications TCP et UDP. Dans ce scénario, un équilibreur de charge interne permet de répartir le trafic sur Citrix NetScaler.
- [Azure Hybrid File Sync](https://github.com/MicrosoftDocs/azure-docs/edit/master/articles/storage/files/storage-sync-files-planning.md) est utilisé pour tout le stockage partagé. Le stockage est répliqué sur deux serveurs de fichiers à l’aide d’Hybrid File Sync.
- [Azure SQL Database](/azure/sql-database/sql-database-technical-overview) est une base de données relationnelle en tant que service reposant sur la dernière version stable du moteur de base de données Microsoft SQL Server. Elle sert à l’hébergement des bases de données Citrix.
- [ExpressRoute](/azure/expressroute/expressroute-introduction) vous permet d’étendre vos réseaux locaux au cloud de Microsoft via une connexion privée assurée par un fournisseur de connectivité. 
- [Les services de domaine Active Directory sont utilisés pour les services d’annuaire et l’authentification des utilisateurs
- Les [groupes à haute disponibilité Azure](/azure/virtual-machines/windows/tutorial-availability-sets) veillent à ce que les machines virtuelles que vous déployez sur Azure soient distribuées sur plusieurs nœuds matériels isolés d’un cluster. Leur utilisation garantit qu’en cas de défaillance matérielle ou logicielle dans Azure, seul un sous-ensemble de vos machines virtuelles est affecté et que votre solution globale reste disponible et opérationnelle. 
- [Citrix ADC (NetScaler)](https://www.citrix.com/products/citrix-adc) est un contrôleur de livraison d’applications qui effectue des analyses de trafic spécifiques sur les applications afin de distribuer, d’optimiser et de protéger de façon intelligente le trafic réseau couche 4-couche 7 (L4-L7) pour les applications Web. 
- [Citrix StoreFront](https://www.citrix.com/products/citrix-virtual-apps-and-desktops/citrix-storefront.html) est un magasin d’applications professionnelles qui améliore la sécurité et simplifie les déploiements pour une expérience utilisateur moderne et presque native inégalée sur toutes les plateformes Citrix Receiver. StoreFront facilite la gestion des environnements d’applications et de bureaux virtuels Citrix sur plusieurs sites et en plusieurs versions. 
- [Citrix License Server](https://www.citrix.com/buy/licensing/overview.html) gère les licences des produits Citrix.
- [Citrix XenDesktops VDA](https://docs.citrix.com/en-us/citrix-virtual-apps-desktops-service) permet de se connecter aux applications et aux bureaux. Le VDA est installé sur la machine qui exécute les applications ou les bureaux virtuels de l’utilisateur. Il permet aux machines de s’enregistrer auprès des contrôleurs de livraison et de gérer la connexion HDX (High Definition eXperience) à un appareil utilisateur.
- [Citrix Delivery Controller](https://docs.citrix.com/en-us/xenapp-and-xendesktop/7-15-ltsr/manage-deployment/delivery-controllers) est le composant côté serveur chargé de gérer l’accès des utilisateurs, ainsi que la distribution et l’optimisation des connexions. Les contrôleurs assurent également les services de création de machines qui effectuent les images des bureaux et des serveurs.

### <a name="alternatives"></a>Autres solutions

- Il existe plusieurs partenaires ayant des solutions VDI prises en charge par Azure, comme VMware, Workspot, etc. Cet exemple précis d’architecture repose sur un projet de déploiement ayant utilisé Citrix.
- Citrix fournit un service cloud qui résume une partie de cette architecture. Il peut s’agir d’une autre option pour cette solution. Pour plus d’informations, consultez la page [Cloud Citrix](https://www.citrix.com/products/citrix-cloud).

## <a name="considerations"></a>Considérations

- Consultez les [conditions requises Linux pour Citrix](https://docs.citrix.com/en-us/linux-virtual-delivery-agent/current-release/system-requirements).
- La latence peut avoir une incidence sur l’ensemble de la solution. Pour un environnement de production, effectuez des essais en conséquence.
- Selon le scénario, la solution peut avoir besoin de machines virtuelles avec GPU pour VDA. Pour cette solution, nous partons du principe que le GPU n’est pas nécessaire.

### <a name="availability-scalability-and-security"></a>Disponibilité, extensibilité et sécurité

- Cet exemple de solution est conçu pour assurer une haute disponibilité de tous les rôles autres que celui du serveur de licences. Comme l’environnement continue de fonctionner pendant un délai supplémentaire de 30 jours si le serveur de licences se trouve hors ligne, aucune redondance supplémentaire n’est requise sur ce serveur.
- Tous les serveurs assurant des rôles similaires doivent être déployés dans des [groupes à haute disponibilité](/azure/virtual-machines/windows/manage-availability#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy).
- Cet exemple de solution ne comprend pas les fonctionnalités de reprise d’activité après sinistre. [Azure Site Recovery](/azure/site-recovery/site-recovery-overview) peut compléter idéalement cette conception.
- Pour une solution de gestion des déploiements de production, il est nécessaire d’implémenter des solutions de type [sauvegarde](/azure/backup/backup-introduction-to-azure-backup), [surveillance](/azure/monitoring-and-diagnostics/monitoring-overview) et [mise à jour](/azure/automation/automation-update-management).
- Cet exemple de solution doit fonctionner pour environ 250 utilisateurs simultanés (environ 50-60 par serveur VDA) avec un usage mixte. Mais la capacité dépend beaucoup du type d’applications utilisées. Pour une utilisation en production, des tests de charge rigoureux doivent être effectués.

## <a name="deploy-this-scenario"></a>Déployer ce scénario

Pour en savoir plus sur le déploiement, consultez la [documentation officielle Citrix](https://docs.citrix.com/en-us/citrix-virtual-apps-desktops/install-configure.html).

## <a name="pricing"></a>Tarifs

- Les licences Citrix XenDesktop ne sont pas incluses dans les frais de service Azure.
- La licence Citrix NetScaler est incluse dans un modèle de paiement à l’utilisation.
- L’utilisation d’instances réservées réduit considérablement les coûts de calcul de la solution.
- Les coûts liés à ExpressRoute ne sont pas inclus.

## <a name="next-steps"></a>Étapes suivantes

- Consultez [ici](https://docs.citrix.com/en-us/citrix-virtual-apps-desktops/install-configure) la documentation Citrix sur la planification et le déploiement.
- Pour déployer Citrix ADC (NetScaler) dans Azure, consultez [ici](https://github.com/citrix/netscaler-azure-templates) les modèles de Gestionnaire des ressources fournis par Citrix.
