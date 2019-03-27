---
title: Récupérer de la perte d’une région Azure
description: Présentation et conception d’applications à tolérance de panne résilientes et hautement disponibles, et planification de la reprise d’activité après sinistre.
author: adamglick
ms.date: 08/18/2016
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.custom: resiliency
ms.openlocfilehash: 7f207bbc0bb0128126f9b828dc100d43553cb100
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58242710"
---
[!INCLUDE [header](../_includes/header.md)]

# <a name="azure-resiliency-technical-guidance-recovery-from-a-region-wide-service-disruption"></a>Guide technique de la résilience Azure - Récupération d’une interruption de service à l’échelle de la région

Azure est divisé physiquement et logiquement en unités appelées régions. Une région se compose d’un ou plusieurs centres de données très proches les uns des autres.

Exceptionnellement, il est possible que les sites de toute une région deviennent inaccessibles, par exemple en raison de défaillances au niveau du réseau ou en raison d’une catastrophe naturelle. Cette section présente les fonctionnalités d’Azure permettant de créer des applications distribuées entre les régions. Cette distribution permet de minimiser le risque qu’une défaillance dans une région affecte d’autres régions.

## <a name="cloud-services"></a>Services cloud

### <a name="resource-management"></a>Gestion des ressources

Vous pouvez distribuer des instances de calcul entre les régions en créant un service cloud distinct dans chaque région cible et en publiant le package de déploiement pour chaque service cloud. Toutefois, notez que la répartition du trafic entre les services cloud dans différentes régions doit être effectuée par le développeur de l’application ou par un service de gestion du trafic.

Lors de la planification de la capacité, il est important de déterminer à l’avance le nombre d’instances de rôle de réserve à déployer pour la récupération d’urgence. Un déploiement secondaire à pleine échelle permet de garantir que la capacité est déjà disponible au moment voulu ; toutefois, le coût est alors doublé. Un modèle courant consiste à disposer d’un petit déploiement secondaire suffisamment grand pour exécuter les services critiques. Ce petit déploiement secondaire vous permettra de réserver de la capacité et de tester la configuration de l’environnement secondaire.

> [!NOTE]
> Le quota d’abonnements n’est pas une garantie de capacité. Il constitue simplement une limite de crédit. Pour garantir la capacité, le nombre requis de rôles doit être défini dans le modèle de service et les rôles doivent être déployés.

### <a name="load-balancing"></a>Équilibrage de la charge.

Pour équilibrer la charge de trafic entre les régions, il est nécessaire d’utiliser une solution de gestion du trafic. Azure fournit [Azure Traffic Manager](https://azure.microsoft.com/services/traffic-manager/). Vous pouvez également utiliser des services tiers qui offrent des fonctionnalités de gestion de trafic similaires.

### <a name="strategies"></a>Stratégies

De nombreuses autres stratégies sont disponibles pour implémenter un système de calcul distribué entre les régions. Celles-ci doivent être adaptées aux exigences professionnelles et aux circonstances de la demande. À un niveau élevé, les approches peuvent être divisées selon les catégories suivantes :

- **Redéploiement en cas de sinistre** : Dans cette approche, l’application est redéployée de bout en bout au moment du sinistre. Cette stratégie est appropriée pour les applications non critiques qui ne nécessitent pas une garantie de délai de récupération.

- **Rechange semi-automatique (actif/passif)**  : un service hébergé secondaire est créé dans une autre région et les rôles sont déployés pour garantir une fonctionnalité minimale ; toutefois, les rôles ne reçoivent pas de trafic de production. Cette approche est utile pour les applications qui n’ont pas été conçues pour distribuer le trafic entre les régions.

- **Échange à chaud (actif/actif)**  : L’application est conçue pour recevoir la charge de production dans plusieurs régions. Les services cloud de chaque région peuvent être configurés pour une capacité plus élevée que celle nécessaire à la récupération d’urgence. La taille des services cloud peut également augmenter selon les besoins au moment de l’incident et du basculement. Cette approche nécessite un investissement important en matière de conception des applications mais elle présente des avantages significatifs, comme par exemple un délai de récupération rapide et garanti, un test continu de tous les emplacements de récupération et une utilisation efficace de la capacité.

Ce document ne fournit pas de description complète de la conception distribuée. Pour plus d’informations, consultez [Récupération d’urgence et haute disponibilité des applications développées sur Microsoft Azure](https://aka.ms/drtechguide).

## <a name="virtual-machines"></a>Machines virtuelles

La récupération des machines virtuelles Infrastructure as a Service (IaaS) est similaire à la récupération de calcul Platform as a Service (PaaS) à bien des égards. Il existe toutefois des différences importantes en raison du fait qu’une machine virtuelle IaaS est composée de la machine virtuelle et du disque de machine virtuelle.

- **Utilisez Azure Backup pour créer des sauvegardes cohérentes des applications dans différentes régions**.
  [Azure Backup](https://azure.microsoft.com/services/backup/) permet aux clients de créer des sauvegardes cohérentes des applications sur plusieurs disques de machine virtuelle et de prendre en charge la réplication des sauvegardes entre différentes régions. Vous pouvez effectuer cette opération en choisissant de géorépliquer le coffre de sauvegarde au moment de la création. Veuillez noter que la réplication du coffre de sauvegarde doit être configurée au moment de la création. Elle ne peut pas être définie ultérieurement. Si une région est perdue, Microsoft met les sauvegardes à disposition des clients. Les clients seront en mesure d’effectuer une restauration à partir des points de restauration configurés.

- **Séparez le disque de données du disque du système d’exploitation**. Pour les machines virtuelles IaaS, il est important de garder en mémoire que vous ne pouvez pas modifier le disque du système d’exploitation sans recréer la machine virtuelle. Ce n’est pas un problème si votre stratégie de récupération après un incident est le redéploiement. En revanche, cette méthode peut poser problème si vous utilisez l’approche de rechange semi-automatique pour réserver de la capacité. Pour implémenter correctement cette stratégie, vous devez disposer du disque du système d’exploitation déployé aux emplacements principaux et secondaires et les données d’application doivent être stockées sur un lecteur distinct. Si possible, utilisez une configuration de système d’exploitation standard qui peut être fournie aux deux emplacements. Après un basculement, vous devez associer le lecteur de données à vos machines virtuelles IaaS existantes dans le contrôleur de domaine secondaire. Utilisez AzCopy pour copier des instantanés du ou des disques de données sur un site distant.

- **Ayez conscience des problèmes potentiels de cohérence après un basculement géographique de plusieurs disques de machines virtuelles**. Les disques de machine virtuelle sont implémentés en tant qu’objets blob Azure Storage et ont les mêmes caractéristiques de géoréplication. Si [Azure Backup](https://azure.microsoft.com/services/backup/) n’est pas utilisé, il n’existe aucune garantie de cohérence entre les disques, étant donné que la géoréplication est asynchrone et que la réplication se fait de manière indépendante. Chaque disque de machine virtuelle se trouve dans un état de cohérence en cas d’incident après un basculement géographique, mais cette cohérence n’est pas assurée entre les disques. Dans certains cas, cela peut entraîner des problèmes (par exemple, en cas d’entrelacement de disques).

## <a name="storage"></a>Stockage

### <a name="recovery-by-using-geo-redundant-storage-of-blob-table-queue-and-vm-disk-storage"></a>Récupération à l’aide du stockage géoredondant des objets blob, des tables, des files d’attente et du stockage sur disque de machine virtuelle

Dans Azure, les objets blob, tables, files d’attente et disques de machine virtuelle sont tous géorépliqués par défaut. Cela s’appelle le stockage géoredondant (GRS). GRS réplique les données de stockage dans un centre de données couplé, situé à des centaines de kilomètres, dans une région géographique donnée. GRS est conçu pour fournir une durabilité supplémentaire en cas d’incident grave dans le centre de données. Microsoft contrôle quand un basculement a lieu et le basculement est réservé aux incidents majeurs pour lesquels on considère qu’il est impossible de restaurer l’emplacement principal d’origine dans un délai raisonnable. Dans certains cas, cela peut prendre plusieurs jours. Les données sont généralement répliquées en quelques minutes, même si l’intervalle de synchronisation n’est pas encore couvert par un contrat de niveau de service.

En cas de basculement géographique, il n’y aura aucune modification de l’accès du compte (l’URL et la clé du compte ne sont pas modifiées). Toutefois, le compte de stockage sera dans une région différente après le basculement. Cela peut affecter les applications qui nécessitent une affinité régionale avec leur compte de stockage. Même pour les services et applications qui ne nécessitent pas un compte de stockage dans le même centre de données, les frais de latence et de bande passante entre centres de données peuvent être l’une des bonnes raisons pour lesquelles déplacer temporairement le trafic vers la région de basculement. Cela peut jouer un rôle important dans la stratégie globale de récupération d’urgence.

Outre le basculement automatique offert par GRS, Azure propose un service qui vous donne un accès en lecture à la copie de vos données à l’emplacement de stockage secondaire. Il s’agit du stockage géoredondant avec accès en lecture (RA-GRS).

Pour plus d’informations sur le stockage GRS et RA-GRS, consultez [Réplication Azure Storage](/azure/storage/storage-redundancy/).

### <a name="geo-replication-region-mappings"></a>Mappages de régions de géoréplication

Il est important de savoir où vos données sont géorépliquées afin de savoir où déployer les autres instances de vos données qui nécessitent une affinité régionale avec votre stockage. Pour plus d’informations, consultez [Régions jumelées Azure](/azure/best-practices-availability-paired-regions).

### <a name="geo-replication-pricing"></a>Tarification de la géoréplication

La géoréplication est incluse dans le tarif actuel d’Azure Storage. Il s’agit du stockage géoredondant (GRS, Geo-Redundant Storage). Si vous ne souhaitez pas que vos données soient géorépliquées, vous pouvez désactiver la géoréplication pour votre compte. Il s’agit du stockage localement redondant, qui est facturé à un prix réduit par rapport au GRS.

### <a name="determining-if-a-geo-failover-has-occurred"></a>Déterminer si un géo-basculement s’est produit

Si un basculement géographique se produit, il est publié dans le [tableau de bord d’état du service Azure](https://azure.microsoft.com/status/). Toutefois, les applications peuvent implémenter un moyen automatisé pour détecter ces basculements géographiques en analysant la région géographique de leur compte de stockage. Cela peut servir à déclencher d’autres opérations de récupération telles que l’activation des ressources de calcul dans la région géographique vers laquelle leur stockage a été déplacé. Vous pouvez effectuer ce type de requête à partir de l’API Gestion des services à l’aide de l’option [d’obtention des propriétés du compte de stockage](https://msdn.microsoft.com/library/ee460802.aspx). Les propriétés pertinentes sont :

    <GeoPrimaryRegion>primary-region</GeoPrimaryRegion>
    <StatusOfPrimary>[Available|Unavailable]</StatusOfPrimary>
    <LastGeoFailoverTime>DateTime</LastGeoFailoverTime>
    <GeoSecondaryRegion>secondary-region</GeoSecondaryRegion>
    <StatusOfSecondary>[Available|Unavailable]</StatusOfSecondary>

### <a name="vm-disks-and-geo-failover"></a>Disques de machine virtuelle et basculement géographique

Comme expliqué dans la section sur les disques de machine virtuelle, il n’existe aucune garantie de la cohérence des données entre les disques de machine virtuelle après un basculement. Pour assurer l’exactitude des sauvegardes, il est nécessaire d’utiliser un logiciel de sauvegarde tel que Data Protection Manager pour sauvegarder et restaurer les données d’application.

## <a name="database"></a>Base de données

### <a name="sql-database"></a>Base de données SQL

Azure SQL Database fournit deux types de récupération : la géorestauration et la géoréplication active.

#### <a name="geo-restore"></a>Restauration géographique

[Restauration géographique](/azure/sql-database/sql-database-recovery-using-backups/#geo-restore) est également disponible avec les bases de données De base, Standard et Premium. Elle fournit l’option de récupération par défaut lorsque la base de données est indisponible en raison d’un incident dans la région où la base de données est hébergée. À l’image de la limite de restauration dans le temps, la géo-restauration s’appuie sur les sauvegardes de base de données dans le stockage Azure géoredondant. Elle restaure à partir de la copie de sauvegarde géorépliquée et est par conséquent résistante aux pannes de stockage dans la région primaire. Pour en savoir plus, consultez la rubrique [Restaurer une base de données SQL Azure ou basculer vers une base de données secondaire](/azure/sql-database/sql-database-disaster-recovery/).

#### <a name="active-geo-replication"></a>Géoréplication active

[Géoréplication active](/azure/sql-database/sql-database-geo-replication-overview/) est disponible avec tous les niveaux de bases de données. Elle est conçue pour les applications qui ont des exigences de récupération plus agressives que celles proposées par la géo-restauration. À l'aide de la géoréplication active, vous pouvez créer jusqu'à quatre répliques secondaires sur des serveurs dans différentes régions. Vous pouvez lancer le basculement sur n’importe quelle base de données secondaire. En outre, la géoréplication active peut être utilisée pour prendre en charge les scénarios de mise à niveau ou de déplacement d'application, ainsi que l'équilibrage de charge pour les charges de travail en lecture seule. Pour plus d’informations, consultez [Configurer la géoréplication](/azure/sql-database/sql-database-geo-replication-portal/) et [Basculement vers la base de données secondaire](/azure/sql-database/sql-database-geo-replication-failover-portal/). Pour plus d’informations sur la conception, l’implémentation et la mise à niveau d’applications sans interruption, consultez [Concevoir une application pour la récupération d’urgence cloud à l’aide de la géoréplication active dans SQL Database](/azure/sql-database/sql-database-designing-cloud-solutions-for-disaster-recovery/) et [Gestion des mises à niveau propagées des applications cloud à l’aide de la géoréplication active de SQL Database](/azure/sql-database/sql-database-manage-application-rolling-upgrade/).

### <a name="sql-server-on-virtual-machines"></a>SQL Server sur Virtual Machines

Plusieurs options sont disponibles pour la récupération et la haute disponibilité de SQL Server 2012 (et version ultérieure) sur Azure Virtual Machines. Pour plus d’informations, consultez [Haute disponibilité et récupération d’urgence pour SQL Server dans Azure Virtual Machines](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/).

## <a name="other-azure-platform-services"></a>Autres services de plateforme Azure

Lorsque vous tentez d’exécuter votre service cloud dans plusieurs régions Azure, vous devez prendre en compte les implications pour chacune de vos dépendances. Dans les sections suivantes, les instructions spécifiques au service partent du principe que vous devez utiliser le même service Azure dans un autre centre de données Azure. Cela nécessite d’effectuer des tâches de configuration et de réplication des données.

> [!NOTE]
> Dans certains cas, ces étapes peuvent aider à limiter l’interruption à un service spécifique plutôt qu’à l’ensemble d’un centre de données. Du point de vue de l’application, l’interruption d’un service donné peut être tout aussi restrictive et nécessiterait la migration temporaire du service dans une autre région Azure.

### <a name="service-bus"></a>Service Bus

Azure Service Bus utilise un espace de noms unique qui ne couvre pas les régions Azure. La première exigence consiste donc à configurer les espaces de noms du Service Bus nécessaires dans l’autre région. Toutefois, il faut également réfléchir à la durabilité des messages mis en file d’attente. Il existe plusieurs stratégies de réplication des messages entre les régions Azure. Pour plus d’informations sur ces stratégies de réplication et d’autres stratégies de récupération d’urgence, consultez [Bonnes pratiques pour protéger les applications contre les pannes de Service Bus et les sinistres](/azure/service-bus-messaging/service-bus-outages-disasters/). Pour d’autres considérations relatives à la disponibilité, consultez la section [Service Bus (Disponibilité)](recovery-local-failures.md#other-azure-platform-services).

### <a name="app-service"></a>App Service

Pour migrer une application Azure App Service, comme Web Apps ou Mobile Apps, vers une région Azure secondaire, vous devez disposer d’une sauvegarde du site web disponible pour la publication. Si la panne ne concerne pas l’ensemble du centre de données Azure, vous pouvez utiliser un FTP pour télécharger une sauvegarde récente du contenu du site. Créez ensuite une nouvelle application dans l’autre région, sauf si vous l’avez fait précédemment pour réserver de la capacité. Publiez le site dans la nouvelle région et apportez les modifications nécessaires à la configuration. Ces modifications peuvent inclure des chaînes de connexion de base de données ou d’autres paramètres régionaux. Si nécessaire, ajoutez le certificat SSL du site et modifiez l’enregistrement DNS CNAME afin que le nom de domaine personnalisé pointe vers l’URL de l’application web Azure redéployée.

### <a name="hdinsight"></a>HDInsight

Par défaut, les données associées à HDInsight sont stockées dans Azure Blob Storage. HDInsight nécessite qu’un cluster Hadoop traitant des tâches MapReduce se trouve dans la même région que le compte de stockage qui contient les données en cours d’analyse. Si vous utilisez la fonctionnalité de géoréplication disponible dans Azure Storage, vous pouvez accéder à vos données dans la région secondaire dans laquelle les données ont été répliquées si pour une raison quelconque la région primaire n’est plus disponible. Vous pouvez créer un nouveau cluster Hadoop dans la région où les données ont été répliquées et poursuivre leur traitement. Pour d’autres considérations relatives à la disponibilité, consultez la section [HDInsight (Disponibilité)](recovery-local-failures.md#other-azure-platform-services).

### <a name="sql-reporting"></a>SQL Reporting

À ce stade, la récupération après la perte d’une région Azure requiert plusieurs instances SQL Reporting dans différentes régions Azure. Ces instances SQL Reporting doivent accéder aux mêmes données et ces dernières doivent avoir leur propre plan de récupération en cas d’incident. Vous pouvez également conserver des copies de sauvegarde externes du fichier RDL pour chaque rapport.

### <a name="media-services"></a>Media Services

Azure Media Services adopte une approche de récupération différente pour l’encodage et le streaming. En règle générale, le streaming est plus critique pendant une panne régionale. Pour vous préparer à cette éventualité, vous devez disposer d’un compte Media Services dans deux régions Azure différentes. Le contenu encodé doit se trouver dans les deux régions. En cas de panne, vous pouvez rediriger le trafic du streaming vers l’autre région. L’encodage peut être effectué dans n’importe quelle région Azure. Si l’encodage est urgent, par exemple lors du traitement d’événements en direct, vous devez être prêt à envoyer des tâches à un autre centre de données en cas de panne.

### <a name="virtual-network"></a>Réseau virtuel

Les fichiers de configuration constituent le moyen le plus rapide de configurer un réseau virtuel dans une autre région Azure. Après avoir configuré le réseau virtuel dans la région Azure principale, [exportez les paramètres de réseau virtuel](/azure/virtual-network/virtual-networks-create-vnet-classic-portal/) pour le réseau actuel dans un fichier de configuration réseau. En cas de panne dans la région primaire, [restaurez le réseau virtuel](/azure/virtual-network/virtual-networks-create-vnet-classic-portal/) à partir du fichier de configuration stocké. Configurez ensuite les autres services cloud, les machines virtuelles ou les paramètres entre sites locaux pour utiliser le nouveau réseau virtuel.

## <a name="checklists-for-disaster-recovery"></a>Listes de contrôle pour la récupération d’urgence

## <a name="cloud-services-checklist"></a>Liste de contrôle de Cloud Services

1. Consultez la section Services cloud de ce document.
2. Créez une stratégie de récupération d’urgence entre les régions.
3. Maîtrisez les inconvénients de la réservation de capacité dans d’autres régions.
4. Utilisez les outils de routage du trafic, notamment Azure Traffic Manager.

## <a name="virtual-machines-checklist"></a>Liste de contrôle de Virtual Machines

1. Consulter la section Machines virtuelles de ce document.
2. Utilisez [Azure Backup](https://azure.microsoft.com/services/backup/) pour créer des sauvegardes cohérentes des applications dans différentes régions.

## <a name="storage-checklist"></a>Liste de contrôle du stockage

1. Consultez la section Stockage de ce document.
2. Ne désactivez pas la géoréplication des ressources de stockage.
3. Maîtrisez l’autre région pour la géoréplication en cas de basculement.
4. Créez des stratégies de sauvegarde personnalisées pour les stratégies de basculement contrôlées par l’utilisateur.

## <a name="sql-database-checklist"></a>Liste de contrôle de la Base de données SQL

1. Consultez la section Base de données SQL de ce document.
2. Utilisez la [géo-restauration](/azure/sql-database/sql-database-recovery-using-backups/#geo-restore) ou la [géoréplication](/azure/sql-database/sql-database-geo-replication-overview/) en fonction des besoins.

## <a name="sql-server-on-virtual-machines-checklist"></a>Liste de contrôle de SQL Server sur les machines virtuelles

1. Consultez la section SQL Server sur Virtual Machines de ce document.
2. Utilisez des groupes de disponibilité AlwaysOn entre les régions ou la mise en miroir de bases de données.
3. Ou utilisez la sauvegarde et la restauration dans le stockage d’objets blob.

## <a name="service-bus-checklist"></a>Liste de contrôle de Service Bus

1. Consultez la section Service Bus de ce document.
2. Configurez un espace de noms Service Bus dans une autre région.
3. Prenez en compte les stratégies de réplication personnalisées pour les messages entre les régions.

## <a name="app-service-checklist"></a>Liste de vérification App Service

1. Consultez la section App Service de ce document.
2. Gérez les sauvegardes de sites en dehors de la région principale.
3. Si une panne est partielle, tentez de récupérer le site actuel via FTP.
4. Envisagez de déployer le site sur un nouveau site ou un site existant dans une autre région.
5. Planifiez les modifications de configuration pour les applications et les enregistrements DNS CNAME.

## <a name="hdinsight-checklist"></a>Liste de contrôle de HDInsight

1. Consultez la section HDInsight de ce document.
2. Créez un cluster Hadoop dans la région contenant les données répliquées.

## <a name="sql-reporting-checklist"></a>Liste de contrôle de SQL Reporting

1. Consultez la section SQL Reporting de ce document.
2. Conservez une autre instance SQL Reporting dans une région différente.
3. Conservez un plan distinct pour répliquer la cible dans cette région.

## <a name="media-services-checklist"></a>Liste de contrôle de Media Services

1. Consultez la section Media Services de ce document.
2. Créez un compte Media Services dans une autre région.
3. Encodez le même contenu dans les deux régions pour prendre en charge le basculement du streaming.
4. Envoyez des travaux d’encodage dans une autre région en cas d’interruption de service.

## <a name="virtual-network-checklist"></a>Liste de contrôle du réseau virtuel

1. Consultez la section Réseau virtuel de ce document.
2. Utilisez les paramètres du réseau virtuel qui ont été exportés pour le recréer dans une autre région.
