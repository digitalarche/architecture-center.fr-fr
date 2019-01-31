---
title: Application web multiniveau conçue pour la haute disponibilité/reprise d’activité
titleSuffix: Azure Example Scenarios
description: Créez une application web multiniveau développée pour la haute disponibilité et la reprise d’activité après sinistre sur Azure à l’aide de machines virtuelles Azure, de groupes à haute disponibilité, de zones de disponibilité, d’Azure Site Recovery et d’Azure Traffic Manager.
author: sujayt
ms.date: 11/16/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: product-team
social_image_url: /azure/architecture/example-scenario/infrastructure/media/arhitecture-disaster-recovery-multi-tier-app.png
ms.openlocfilehash: c60a2a07db578c447eb0682270c105b79e80e12b
ms.sourcegitcommit: 3b15d65e7c35a19506e562c444343f8467b6a073
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/25/2019
ms.locfileid: "54908509"
---
# <a name="multitier-web-application-built-for-high-availability-and-disaster-recovery-on-azure"></a>Application web multiniveau développée pour la haute disponibilité et la reprise d’activité sur Azure

Cet exemple de scénario s’applique à tout secteur d’activité ayant besoin de déployer des applications multiniveaux résilientes conçues pour la haute disponibilité et la reprise d’activité. Dans ce scénario, l’application se compose de trois couches.

- Couche web : couche supérieure comprenant l’interface utilisateur. Cette couche analyse les interactions utilisateur et transmet les actions à la couche suivante pour le traitement.
- Couche métier : traite les interactions utilisateur et prend des décisions logiques concernant les étapes suivantes. Cette couche relie la couche web et la couche données.
- Couche données : stocke les données d’application. Vous utilisez généralement une base de données, un stockage d’objets ou un stockage de fichiers.

Les applications stratégiques s’exécutant sur Windows ou Linux font partie des scénarios d’application courants. Il peut s’agir d’une application du commerce comme SAP et SharePoint, ou une application métier personnalisée.

## <a name="relevant-use-cases"></a>Cas d’usage appropriés

Les autres cas d’usage appropriés sont les suivants :

- Déploiement d’applications hautement résilientes comme SAP et SharePoint
- Conception d’un plan de continuité d’activité et de reprise d’activité pour les applications métier
- Configurer la reprise d’activité et effectuer les exercices associés à des fins de conformité

## <a name="architecture"></a>Architecture

Ce scénario présente une application multiniveau qui utilise ASP.NET et Microsoft SQL Server. Dans les [régions Azure qui prennent en charge les zones de disponibilité](/azure/availability-zones/az-overview#regions-that-support-availability-zones), vous pouvez déployer vos machines virtuelles dans une région source sur diverses zones de disponibilité et répliquer les machines virtuelles sur la région cible utilisée pour la reprise d’activité. Dans les régions Azure qui ne prennent pas en charge les zones de disponibilité, vous pouvez déployer vos machines virtuelles au sein d’un groupe à haute disponibilité et les répliquer sur la région cible.

![Vue d’ensemble de l’architecture d’une application web multiniveau hautement résiliente][architecture]

- Distribuez les machines virtuelles dans chaque niveau sur deux zones de disponibilité dans les régions qui prennent en charge les zones. Dans les autres régions, déployez les machines virtuelles dans chaque niveau au sein d’un groupe à haute disponibilité.
- La couche Données peut être configurée pour utiliser des groupes de disponibilité Always On. Avec cette configuration de SQL Server, une base de données primaire au sein d’un cluster est configurée avec un maximum de huit bases de données secondaires. En cas de problème avec la base de données primaire, le cluster bascule sur l’une des bases de données secondaires, ce qui permet à l’application de rester disponible. Pour plus d’informations, consultez [Vue d’ensemble des groupes de disponibilité AlwaysOn pour SQL Server][docs-sql-always-on].
- Pour les scénarios de reprise d’activité, vous pouvez configurer la réplication asynchrone native SQL Always On sur la région cible utilisée pour la reprise d’activité. Vous pouvez aussi configurer la réplication Azure Site Recovery sur la région cible si le taux de changement des données est dans les limites prises en charge par Azure Site Recovery.
- Les utilisateurs accèdent à la couche web ASP.NET front-end via le point de terminaison Traffic Manager.
- Traffic Manager redirige le trafic vers le point de terminaison de l’adresse IP publique principale dans la région source principale.
- L’adresse IP publique redirige l’appel vers l’une des instances de machine virtuelle de la couche web par le biais d’un équilibreur de charge public. Toutes les instances de machine virtuelle de la couche web se trouvent dans un seul sous-réseau.
- À partir de la machine virtuelle de la couche web, chaque appel est routé vers une des instances de machine virtuelle de la couche métier à travers un équilibreur de charge interne pour le traitement. Toutes les machines virtuelles de la couche métier sont dans un sous-réseau distinct.
- L’opération est traitée dans la couche métier et l’application ASP.NET se connecte au cluster Microsoft SQL Server dans une couche back-end via un équilibreur de charge interne Azure. Ces instances SQL Server back-end sont dans un sous-réseau distinct.
- Le point de terminaison secondaire de Traffic Manager est configuré comme l’adresse IP publique dans la région cible utilisée pour la reprise d’activité.
- En cas de perturbation de la région principale, vous appelez le basculement d’Azure Site Recovery et l’application devient active dans la région cible.
- Le point de terminaison Traffic Manager redirige automatiquement le trafic client vers l’adresse IP publique de la région cible.

### <a name="components"></a>Composants

- Les [groupes à haute disponibilité][docs-availability-sets] garantissent que les machines virtuelles que vous déployez sur Azure sont distribuées sur plusieurs nœuds matériels isolés dans un cluster. En cas de défaillance matérielle ou logicielle dans Azure, seule une partie de vos machines virtuelles est affectée et votre solution globale reste disponible et opérationnelle.
- Les [zones de disponibilité][docs-availability-zones] protègent les applications et les données contre les défaillances de centre de données. Les zones de disponibilité sont des emplacements physiques individuels au sein d’une région Azure. Chaque zone est composée d’un ou de plusieurs centres de données équipés d’une alimentation, d’un système de refroidissement et d’un réseau indépendants.
- [Azure Site Recovery (ASR)][docs-azure-site-recovery] vous permet de répliquer des machines virtuelles sur une autre région Azure pour assurer la continuité d’activité et la reprise d’activité. Vous pouvez effectuer régulièrement des exercices de reprise d’activité pour garantir que vous répondez aux besoins de conformité. La machine virtuelle est répliquée avec les paramètres spécifiés dans la région sélectionnée afin que vous puissiez récupérer vos applications en cas de panne dans la région source.
- [Azure Traffic Manager][docs-traffic-manager] est un équilibreur de charge du trafic DNS qui distribue le trafic de manière optimale aux services dans toutes les régions Azure du monde, tout en offrant réactivité et haute disponibilité.
- [Azure Load Balancer][docs-load-balancer] distribue le trafic entrant en fonction des règles définies et des sondes d’intégrité. Un équilibreur de charge offre une latence faible et un débit élevé, et peut gérer jusqu’à des millions de flux pour toutes les applications TCP et UDP. Un équilibreur de charge public est utilisé dans ce scénario pour distribuer le trafic client entrant à la couche web. Dans ce scénario, un équilibreur de charge interne est utilisé pour distribuer le trafic de la couche métier au cluster SQL Server back-end.

### <a name="alternatives"></a>Autres solutions

- Vous pouvez remplacer Windows par d’autres systèmes d’exploitation, car rien dans l’infrastructure ne dépend du système d’exploitation.
- [SQL Server pour Linux][docs-sql-server-linux] peut remplacer le magasin de données back-end.
- Vous pouvez remplacer la base de données par n’importe quelle application de base de données standard disponible.

## <a name="other-considerations"></a>Autres points à considérer

### <a name="scalability"></a>Extensibilité

Vous pouvez ajouter ou supprimer des machines virtuelles dans chaque niveau selon vos besoins de mise à l’échelle. Comme ce scénario utilise des équilibreurs de charge, vous pouvez ajouter davantage de machines virtuelles à un niveau sans affecter la durée de fonctionnement des applications.

Pour consulter d’autres rubriques relatives à l’extensibilité, consultez la [liste de contrôle de l’extensibilité][scalability] dans le Centre des architectures Azure.

### <a name="security"></a>Sécurité

Tout le trafic de réseau virtuel dans la couche Application front-end est protégé par des groupes de sécurité réseau. Des règles limitent le flux du trafic afin que seules les instances de machine virtuelle au niveau de la couche Application frontale puissent accéder à la couche Données principale. Aucun trafic Internet sortant n’est autorisé à partir de la couche métier ou de la couche base de données. Pour réduire l’encombrement de l’attaque, aucun port de gestion à distance directe n’est ouvert. Pour plus d'informations, consultez [Groupes de sécurité réseau Azure][docs-nsg].

Pour obtenir des conseils d’ordre général sur la conception de scénarios sécurisés, consultez la [documentation sur la sécurité Azure][security].

## <a name="pricing"></a>Tarifs

La configuration de la reprise d’activité pour les machines virtuelles Azure à l’aide d’Azure Site Recovery entraîne les frais suivants en continu.

- Coût de licence Azure Site Recovery par machine virtuelle.
- Coût de sortie réseau pour répliquer les changements de données des disques de machine virtuelle sources sur une autre région Azure. Azure Site Recovery utilise la compression intégrée pour réduire les besoins de transfert de données d’environ 50 %.
- Coût de stockage sur le site de récupération. Ce coût est généralement le même que celui du stockage de la région source plus tout stockage supplémentaire nécessaire pour conserver les points de récupération sous forme d’instantanés pour la récupération.

Nous avons fourni un [exemple de calculateur de coût][calculator] afin de configurer la reprise d’activité pour une application à trois niveaux à l’aide de six machines virtuelles. Tous les services sont préconfigurés dans le calculateur de coût. Pour voir les tarifs dans votre cas d’usage en particulier, changez les variables appropriées pour estimer le coût.

<!-- links -->
[architecture]: ./media/arhitecture-disaster-recovery-multi-tier-app.png
[autoscaling]: /azure/architecture/best-practices/auto-scaling
[availability]: ../../checklist/availability.md
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[docs-availability-zones]: /azure/availability-zones/az-overview
[docs-load-balancer]: /azure/load-balancer/load-balancer-overview
[docs-nsg]: /azure/virtual-network/security-overview
[docs-vmss]: /azure/virtual-machine-scale-sets/overview
[docs-sql-always-on]: /sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server
[docs-vmss-autoscale]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview
[docs-vnet]: /azure/virtual-network/virtual-networks-overview
[docs-sql-server-linux]: /sql/linux/sql-server-linux-overview?view=sql-server-linux-2017
[docs-traffic-manager]: /azure/traffic-manager/
[docs-azure-site-recovery]: /azure/site-recovery/azure-to-azure-quickstart/
[docs-availability-sets]: /azure/virtual-machines/windows/manage-availability/
[calculator]: https://azure.com/e/6835332265044d6d931d68c917979e6d/
