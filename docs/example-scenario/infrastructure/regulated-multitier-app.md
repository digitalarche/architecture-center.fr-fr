---
title: Création d’applications web sécurisées avec des machines virtuelles Windows
titleSuffix: Azure Example Scenarios
description: Créez une application web sécurisée, à plusieurs niveaux avec Windows Server sur Azure à l’aide de groupes identiques, d’Application Gateway et d’équilibreurs de charge.
author: iainfoulds
ms.date: 12/06/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: seodec18, Windows
social_image_url: /azure/architecture/example-scenario/infrastructure/media/architecture-regulated-multitier-app.png
ms.openlocfilehash: 440d208b423703fe791dcbe2cad0609fef0e6508
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58246220"
---
# <a name="building-secure-web-applications-with-windows-virtual-machines-on-azure"></a>Création d’applications web sécurisées avec des machines virtuelles Windows Azure

Ce scénario fournit des conseils de conception et d’architecture pour l’exécution d’applications web sécurisées multiniveaux sur Microsoft Azure. Dans cet exemple, une application ASP.NET se connecte de manière sécurisée à un cluster back-end protégé de Microsoft SQL Server à l’aide de machines virtuelles.

En règle générale, les organisations devaient maintenir les applications et les services hérités localement afin de fournir une infrastructure sécurisée. En déployant ces applications Windows Server dans Azure de manière sécurisée, les organisations peuvent moderniser leurs déploiements ainsi que réduire leurs coûts d’exploitation locaux et leurs frais de gestion.

## <a name="relevant-use-cases"></a>Cas d’usage appropriés

Voici quelques exemples où ce scénario peut être appliqué :

- Modernisation des déploiements d’application dans un environnement cloud sécurisé.
- Réduction des frais liés à la gestion des applications et des services hérités locaux.
- Amélioration des soins et de l’expérience du patient avec de nouvelles plateformes d’application.

## <a name="architecture"></a>Architecture

![Présentation de l’architecture des composants Azure impliqués dans les applications de Windows Server à plusieurs niveaux pour les secteurs industriels réglementés][architecture]

Ce scénario montre une application web front-end qui se connecte à une base de données back-end, les deux s’exécutant sur Windows Server 2016. Les données circulent dans le scénario comme suit :

1. Les utilisateurs accèdent à l’application ASP.NET front-end via Azure Application Gateway.
2. La passerelle d’application distribue le trafic vers les instances de machine virtuelle dans un groupe de machines virtuelles identiques Azure.
3. L’application se connecte au cluster Microsoft SQL Server dans une couche back-end via un équilibreur de charge Azure. Ces instances SQL Server principales sont situées dans un réseau virtuel Azure distinct, sécurisé par des règles de groupe de sécurité réseau qui limitent le flux de trafic.
4. L’équilibreur de charge répartit le trafic de SQL Server vers les instances de machine virtuelle dans un autre groupe de machines virtuelles identiques.
5. Le Stockage Blob Azure fait office de [témoin cloud][cloud-witness] pour le cluster SQL Server dans le niveau back-end. La connexion à partir du réseau virtuel est activée avec un point de terminaison de service de réseau virtuel pour le stockage Azure.

### <a name="components"></a>Composants

- [Azure Application Gateway][appgateway-docs] est un équilibreur de charge du trafic web de couche 7, prenant en charge les applications et pouvant distribuer le trafic en fonction de règles de routage spécifiques. App Gateway peut également gérer le déchargement SSL afin d’améliorer les performances du serveur web.
- [Le réseau virtuel Azure][vnet-docs] permet à de nombreuses ressources, telles que les machines virtuelles de communiquer en toute sécurité entre elles, avec Internet et avec les réseaux locaux. Les réseaux virtuels fournissent un isolement et une segmentation, ils filtrent et acheminent le trafic et ils autorisent une connexion entre des emplacements. Deux réseaux virtuels combinés avec les groupes de sécurité réseau appropriés sont utilisés dans ce scénario pour fournir une [zone démilitarisée][dmz] (DMZ) et une isolation de composants de l’application. L’appairage de réseau virtuel connecte les deux réseaux ensemble.
- Les [groupes identiques de machines virtuelles Azure][scaleset-docs] permettent de créer et de gérer un groupe de machines virtuelles identiques à charge équilibrée. Le nombre d’instances de machine virtuelle peut augmenter ou diminuer automatiquement en fonction d’une demande ou d’un calendrier défini. Deux groupes distincts de machines virtuelles identiques sont utilisés dans ce scénario, un pour les instances d’applications ASP.NET frontales et un autre pour les instances de machine virtuelle du cluster SQL Server principal. La configuration de l’état souhaité (DSC) de PowerShell ou l’extension de script personnalisé Azure peut servir à configurer les instances de machine virtuelle avec les logiciels requis et les paramètres de configuration.
- Les [groupes de sécurité réseau Azure][nsg-docs] contiennent une liste de règles de sécurité qui autorisent ou refusent le trafic réseau entrant ou sortant en fonction de l’adresse IP source ou de destination, du port et du protocole. Les réseaux virtuels dans ce scénario sont sécurisés avec des règles de groupe de sécurité réseau qui limitent le flux du trafic entre les composants d’application.
- [Azure Load Balancer][loadbalancer-docs] distribue le trafic entrant en fonction des règles et des sondes d’intégrité. Un équilibreur de charge offre une latence faible et un débit élevé, et peut augmenter l’échelle jusqu’à des millions de flux pour toutes les applications TCP et UDP. Dans ce scénario, un équilibreur de charge interne permet de distribuer le trafic de la couche Application frontale vers le cluster SQL Server principal.
- [Stockage Blob Azure][cloudwitness-docs] agit comme un emplacement de témoin Cloud pour le cluster SQL Server. Ce témoin est utilisé pour les opérations de cluster et les décisions nécessitant un vote supplémentaire pour décider du quorum. L’utilisation d’un témoin Cloud supprime la nécessité d’une machine virtuelle supplémentaire d’agir comme un témoin de partage de fichiers traditionnel.

### <a name="alternatives"></a>Autres solutions

- Vous pouvez utiliser indifféremment Linux ou Windows dans la mesure où l’infrastructure n’est pas dépendante du système d’exploitation.

- [SQL Server pour Linux][sql-linux] peut remplacer le magasin de données back-end.

- [Cosmos DB](/azure/cosmos-db/introduction) est une autre solution de magasin de données.

## <a name="considerations"></a>Considérations

### <a name="availability"></a>Disponibilité

Dans ce scénario, les instances de machine virtuelle sont déployées sur des [zones de disponibilité](/azure/availability-zones/az-overview). Chaque zone de disponibilité est composée d’un ou de plusieurs centres de données équipés d’une alimentation, d’un système de refroidissement et d’un réseau indépendants. Chaque région activée a un minimum de trois zones de disponibilité. Cette distribution d’instances de machine virtuelle entre des zones offre une haute disponibilité pour les couches Application.

La couche Données peut être configurée pour utiliser des groupes de disponibilité Always On. Avec cette configuration de SQL Server, une base de données primaire au sein d’un cluster est configurée avec un maximum de huit bases de données secondaires. En cas de problème avec la base de données primaire, le cluster bascule sur l’une des bases de données secondaires, ce qui permet à l’application de rester disponible. Pour plus d’informations, consultez [Vue d’ensemble des groupes de disponibilité AlwaysOn pour SQL Server][sqlalwayson-docs].

Pour obtenir d’autres conseils sur la disponibilité, consultez la [liste de contrôle de la disponibilité][availability] dans le Centre des architectures Azure.

### <a name="scalability"></a>Extensibilité

Ce scénario utilise des groupes de machines virtuelles identiques pour les composants frontaux et principaux. Avec des groupes identiques, le nombre d’instances de machine virtuelle qui s’exécutent au niveau de la couche Application frontale peut être automatiquement mis à l’échelle selon la demande du client ou une planification définie. Pour plus d’informations, voir [Vue d’ensemble de la mise à l’échelle automatique avec des groupes de machines virtuelles identiques][vmssautoscale-docs].

Pour consulter d’autres rubriques relatives à l’extensibilité, consultez la [liste de contrôle de l’extensibilité][scalability] dans le Centre des architectures Azure.

### <a name="security"></a>Sécurité

Tout le trafic du réseau virtuel passe dans la couche Application frontale et est protégé par des groupes de sécurité réseau. Des règles limitent le flux du trafic afin que seules les instances de machine virtuelle au niveau de la couche Application frontale puissent accéder à la couche Données principale. Aucun trafic Internet sortant n’est autorisé à partir de la couche Données. Pour réduire l’encombrement de l’attaque, aucun port de gestion à distance directe n’est ouvert. Pour plus d'informations, consultez [Groupes de sécurité réseau Azure][nsg-docs].

Pour afficher des conseils sur le déploiement d’une [infrastructure conforme][pci-dss] à la norme de sécurité de l’industrie des cartes de paiement (PCI DSS 3.2). Pour obtenir des conseils d’ordre général sur la conception de scénarios sécurisés, consultez la [documentation sur la sécurité Azure][security].

### <a name="resiliency"></a>Résilience

En conjonction avec l’utilisation de zones de disponibilité et de groupe de machines virtuelles identiques, ce scénario utilise Azure Application Gateway et un équilibreur de charge. Ces deux composants de mise en réseau distribuent le trafic vers les instances de machine virtuelle connectées et incluent des sondes d’intégrité qui garantissent que le trafic est uniquement distribué vers des machines virtuelles saines. Deux instances Application Gateway sont configurées dans une configuration actif/passif, et un équilibreur de charge redondant interzone est utilisé. Cette configuration rend les ressources de mise en réseau et l’application résilientes aux problèmes qui perturberaient autrement le trafic et affecteraient l’accès de l’utilisateur final.

Pour obtenir des conseils d’ordre général sur la conception de scénarios résilients, consultez l’article [Conception d’applications résilientes pour Azure][resiliency].

## <a name="deploy-the-scenario"></a>Déployez le scénario

### <a name="prerequisites"></a>Prérequis

- Vous devez disposer d’un compte Azure existant. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

- Un domaine dans Azure Active Directory (AD) Domain Services est nécessaire pour déployer un cluster SQL Server dans le groupe identique principal.

### <a name="deploy-the-components"></a>Déployer les composants

Pour déployer l’infrastructure principale pour ce scénario avec un modèle Azure Resource Manager, procédez comme suit.

<!-- markdownlint-disable MD033 -->

1. Sélectionnez le bouton **Déployer sur Azure** :<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Finfrastructure%2Fregulated-multitier-app.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Attendez l’ouverture de la solution Template deployment dans le portail Azure, puis procédez comme suit :
   - Sélectionnez **Créer un nouveau** groupe de ressources, puis indiquez un nom, par exemple *myWindowsscenario* dans la zone de texte.
   - Dans la zone de liste déroulante **Emplacement**, sélectionnez une région.
   - Indiquez un nom d’utilisateur et un mot de passe sécurisé pour les instances de groupe de machines virtuelles identiques.
   - Passez en revue les termes et conditions, puis cochez la case **J’accepte les termes et conditions mentionnés ci-dessus**.
   - Sélectionnez le bouton **Acheter**.

<!-- markdownlint-enable MD033 -->

Le déploiement prend 15 à 20 minutes.

## <a name="pricing"></a>Tarifs

Pour explorer le coût d’exécution de ce scénario, tous les services sont préconfigurés dans le calculateur de coûts. Pour pouvoir observer l’évolution de la tarification pour votre cas d’usage particulier, modifiez les variables appropriées en fonction du trafic que vous escomptez.

Nous proposons trois exemples de profils de coût basés sur le nombre d’instances de groupe identique de machines virtuelles exécutant vos applications.

- [Petit][small-pricing] : 2 instances de machine virtuelle frontales et 2 instances principales.
- [Moyen][medium-pricing] : 20 instances de machine virtuelle frontales et 5 instances principales.
- [Grand][large-pricing] : 100 instances de machine virtuelle frontales et 10 instances principales.

## <a name="related-resources"></a>Ressources associées

Ce scénario a utilisé un groupe de machines virtuelles identiques principales qui exécutent un cluster Microsoft SQL Server. Cosmos DB peut également servir de couche Données évolutive et sécurisée pour les données d’application. Un [point de terminaison de service du réseau virtuel Azure][vnetendpoint-docs] permet de sécuriser vos ressources critiques de service Azure sur vos réseaux virtuels uniquement. Dans ce scénario, les points de terminaison de réseau virtuel permettent de sécuriser le trafic entre la couche Application frontale et Cosmos DB. Pour plus d’informations, consultez la [présentation d’Azure Cosmos DB](/azure/cosmos-db/introduction).

Pour obtenir un guide d’implémentation plus détaillé, consultez l’[architecture de référence pour les applications multiniveaux qui utilisent SQL Server][ntiersql-ra].

<!-- links -->
[appgateway-docs]: /azure/application-gateway/overview
[architecture]: ./media/architecture-regulated-multitier-app.png
[autoscaling]: /azure/architecture/best-practices/auto-scaling
[availability]: ../../checklist/availability.md
[cloudwitness-docs]: /windows-server/failover-clustering/deploy-cloud-witness
[loadbalancer-docs]: /azure/load-balancer/load-balancer-overview
[nsg-docs]: /azure/virtual-network/security-overview
[ntiersql-ra]: /azure/architecture/reference-architectures/n-tier/n-tier-sql-server
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[scaleset-docs]: /azure/virtual-machine-scale-sets/overview
[sqlalwayson-docs]: /sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server
[vmssautoscale-docs]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview
[vnet-docs]: /azure/virtual-network/virtual-networks-overview
[vnetendpoint-docs]: /azure/virtual-network/virtual-network-service-endpoints-overview
[pci-dss]: /azure/security/blueprints/pcidss-iaaswa-overview
[dmz]: /azure/virtual-network/virtual-networks-dmz-nsg
[sql-linux]: /sql/linux/sql-server-linux-overview?view=sql-server-linux-2017
[cloud-witness]: /windows-server/failover-clustering/deploy-cloud-witness
[small-pricing]: https://azure.com/e/711bbfcbbc884ef8aa91cdf0f2caff72
[medium-pricing]: https://azure.com/e/b622d82d79b34b8398c4bce35477856f
[large-pricing]: https://azure.com/e/1d99d8b92f90496787abecffa1473a93
