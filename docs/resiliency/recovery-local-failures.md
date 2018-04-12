---
title: Guide technique - Récupération suite à des défaillances locales dans Azure
description: Article dédié à la compréhension et à la conception d’applications robustes, hautement disponibles et tolérantes aux pannes, ainsi qu’à la planification de la récupération d’urgence axée sur les défaillances locales dans Azure.
author: adamglick
ms.date: 08/18/2016
ms.openlocfilehash: 5fc929bd1affe3dd6616f908bae0e7d2fefb89d5
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/06/2018
---
[!INCLUDE [header](../_includes/header.md)]

# <a name="azure-resiliency-technical-guidance-recovery-from-local-failures-in-azure"></a>Guide technique de la résilience Azure - Récupération suite à des défaillances locales dans Azure

Il existe deux menaces principales à la disponibilité des applications :

* La défaillance des appareils, tels que les lecteurs et les serveurs.
* L’épuisement des ressources critiques, telles que le calcul dans des conditions de charge maximale.

Azure offre une combinaison de gestion des ressources, d’élasticité, d’équilibrage de charge et de partitionnement pour permettre d’obtenir une haute disponibilité dans ces circonstances. Certaines de ces fonctionnalités sont exécutées automatiquement pour tous les services Azure. Cependant, dans certains cas, le développeur d’application doit effectuer un travail supplémentaire pour tirer parti de ces fonctionnalités.

## <a name="cloud-services"></a>Cloud Services
Azure Cloud Services se compose de collections d’un ou de plusieurs rôles Web ou de travail. Une ou plusieurs instances d’un rôle peuvent s’exécuter simultanément. Le nombre d’instances est déterminé par la configuration. Les instances de rôle sont surveillées et gérées à l’aide d’un composant appelé le contrôleur de structure. Le contrôleur de structure détecte automatiquement les défaillances logicielles et matérielles, et y répond.

Chaque instance de rôle s’exécute dans sa propre machine virtuelle et communique avec son contrôleur de structure par le biais d’un agent invité. L’agent invité collecte les métriques de ressource et de nœud, y compris l’utilisation des machines virtuelles, les états, les journaux, l’utilisation des ressources, les exceptions et les conditions de défaillance. Le contrôleur de structure interroge l’agent invité à des intervalles configurables, et redémarre la machine virtuelle si l’agent invité ne répond pas. En cas de défaillance matérielle, le contrôleur de structure associé déplace toutes les instances de rôle affectées vers un nouveau nœud de matériel et reconfigure le réseau pour y acheminer le trafic.

Pour tirer parti de ces fonctionnalités, les développeurs doivent s’assurer que tous les rôles de service évitent de stocker l’état sur les instances de rôle. Au lieu de cela, toutes les données persistantes doivent être accessibles à partir d’un stockage durable, tels que Stockage Azure ou Azure SQL Database. Ainsi, tous les rôles sont en mesure de gérer les demandes. Cela signifie également que les instances de rôle peuvent tomber en panne à tout moment sans créer d’incohérences dans l’état temporaire ou permanent du service.

La nécessité de stocker l’état en dehors des rôles présente plusieurs implications. Par exemple, cela implique que toutes les modifications associées à une table Azure Storage doivent être effectuées dans une seule transaction de groupe d’entités, si possible. Bien entendu, il n’est pas toujours possible d’apporter toutes les modifications dans une transaction unique. Vous devez tout particulièrement veiller à garantir que les défaillances d’instances de rôle ne provoquent aucun problème lorsqu’elles interrompent des opérations de longue durée qui englobent un minimum de deux mises à jour de l’état permanent du service. Si un autre rôle tente à nouveau une telle opération, il doit anticiper les situations pour lesquelles le travail a été partiellement effectué, et les gérer efficacement.

Par exemple, imaginons un service qui partitionne les données dans plusieurs magasins. Si un rôle de travail tombe en panne pendant le déplacement d’une partition, le déplacement de la partition peut ne pas aboutir. Ou bien, le déplacement peut être répété depuis le début par un autre rôle de travail, ce qui peut éventuellement générer des données orphelines ou corrompre des données. Pour éviter ces problèmes, les opérations de longue durée doivent être du même type que l’une ou l’ensemble des opérations suivantes :

* *Idempotente*: répétable sans effet secondaire. Pour être idempotente, une opération de longue durée doit présenter un effet identique dans toutes ses itérations, même lorsqu’elle est interrompue pendant l’exécution.
* *Redémarrable de façon incrémentielle*: en mesure de continuer à partir du point de défaillance le plus récent. Pour être redémarrable de façon incrémentielle, une opération de longue durée doit se composer d’une séquence d’opérations atomiques plus petites. Elle doit également enregistrer sa progression dans le stockage durable, de façon à ce que chacun des appels ultérieurs reprenne là où son prédécesseur s’est arrêté.

Enfin, toutes les opérations de longue durée doivent être appelées à plusieurs reprises jusqu’à leur réussite. Par exemple, une opération d’approvisionnement peut être placée dans une file d’attente Azure, puis supprimée de la file d’attente par un rôle de travail uniquement lorsqu’elle réussit. Un nettoyage de la mémoire peut être nécessaire afin de nettoyer les données créées par des opérations interrompues.

### <a name="elasticity"></a>Élasticité
Le nombre initial d’instances en cours d’exécution pour chaque rôle est déterminé dans la configuration de chaque rôle. Les administrateurs doivent initialement configurer chaque rôle pour l’exécution avec deux ou plusieurs instances, en fonction de la charge attendue. Cependant, vous pouvez facilement monter en puissance ou descendre en puissance les instances de rôle en fonction des variations des modèles d’utilisation. Vous pouvez effectuer cette opération manuellement dans le portail Azure ou automatiser le processus à l’aide de Windows PowerShell, de l’API Gestion des services ou d’outils tiers. Pour plus d’informations, consultez [Mise à l’échelle automatique d’un service cloud](/azure/cloud-services/cloud-services-how-to-scale/).

### <a name="partitioning"></a>Partitionnement
Le contrôleur de structure Azure utilise deux types de partitions :

* Un *domaine de mise à jour* est utilisé pour mettre à niveau les instances d’un rôle de service dans les groupes. Azure déploie les instances de service dans plusieurs domaines de mise à jour. Dans le cadre d’une mise à jour sur place, le contrôleur de structure interrompt toutes les instances d’un domaine de mise à jour, les met à jour et les redémarre avant de passer au domaine de mise à jour suivant. Cette approche prévient toute indisponibilité de l’intégralité du service durant le processus de mise à jour.
* Un *domaine d’erreur* définit les points potentiels de défaillance matérielle ou de réseau. Le contrôleur de structure garantit que les instances d’un rôle en présentant plusieurs sont distribuées entre plusieurs domaines d’erreur pour éviter toute interruption du service par des défaillances matérielles isolées. L’exposition aux défaillances de serveur et de cluster est régie par les domaines d’erreur.

Le [contrat de niveau de service Azure](https://azure.microsoft.com/support/legal/sla/) garantit que lorsqu’au minimum deux instances de rôles Web sont déployées sur des domaines d’erreur et de mise à niveau différents, elles présentent une connectivité externe au moins 99,95 % du temps. Contrairement aux domaines de mise à jour, il n’existe aucun moyen de contrôler le nombre de domaines d’erreur. Azure alloue automatiquement les domaines d’erreur et distribue les instances de rôle entre elles. Les deux premières instances au minimum de chaque rôle sont placées dans des domaines d’erreur et de mise à niveau différents pour garantir que les rôles présentant au moins deux instances satisferont les contrats de niveau de service. Cela est représenté dans le diagramme suivant.

![Vue simplifiée de l’isolement de domaines d’erreur](./images/technical-guidance-recovery-local-failures/partitioning-1.png)

### <a name="load-balancing"></a>Équilibrage de la charge
Tout le trafic entrant d’un rôle web passe par un équilibreur de charge sans état, qui distribue les demandes des clients entre les instances de rôle. Des instances de rôle isolées n’ont pas d’adresses IP publiques et ne sont pas directement adressables à partir d’Internet. Les rôles Web sont sans état, ce qui permet de diriger les requêtes des clients vers toute instance de rôle. Un événement [StatusCheck](https://msdn.microsoft.com/library/microsoft.windowsazure.serviceruntime.roleenvironment.statuscheck.aspx) est déclenché toutes les 15 secondes. Vous pouvez utiliser cet événement pour indiquer si le rôle est prêt à recevoir le trafic, ou s’il est occupé et doit être retiré de la rotation d’équilibrage de charge.

## <a name="virtual-machines"></a>Virtual Machines
Azure Virtual Machines diffère des rôles de calcul Platform as a Service (PaaS) par plusieurs aspects relatifs à la haute disponibilité. Dans certains cas, vous devez effectuer des opérations supplémentaires pour garantir une haute disponibilité.

### <a name="disk-durability"></a>Durabilité des disques
Contrairement aux instances de rôle PaaS, les données stockées sur les disques de machines virtuelles demeurent persistantes, même en cas de déplacement des machines virtuelles. Les machines virtuelles Azure utilisent les disques de machines virtuelles existant en tant qu’objets blob dans Azure Storage. En raison des caractéristiques de disponibilité d’Azure Storage, les données stockées sur des lecteurs d’une machine virtuelle sont également hautement disponibles.

Notez que le lecteur D (sur les machines virtuelles Windows) fait exception à cette règle. Le lecteur D correspond au stockage physique sur le serveur monté en rack qui héberge la machine virtuelle ; ses données seront perdues en cas de recyclage de la machine virtuelle. Le lecteur D est dédié au stockage temporaire uniquement. Sous Linux, Azure expose « généralement » (mais pas systématiquement) le disque temporaire local en tant qu’appareil en mode bloc /dev/sdb. Il est souvent monté par l’Agent Linux Azure sous forme de point de montage /mnt/resource ou /mnt (configurable via/etc/waagent.conf).

### <a name="partitioning"></a>Partitionnement
Azure comprend de manière native les niveaux d’une application PaaS (rôle Web et rôle de travail), et peut par conséquent les distribuer efficacement entre les domaines d’erreur et de mise à jour. En revanche, les niveaux dans une application Infrastructure as a Service (IaaS) doivent être définis manuellement via les groupes à haute disponibilité. Des groupes à haute disponibilité sont requis pour les contrats de niveau de service établis sous IaaS.

![Groupes à haute disponibilité pour les machines virtuelles Azure](./images/technical-guidance-recovery-local-failures/partitioning-2.png)

Dans le diagramme précédent, le niveau Internet Information Services (IIS, qui fonctionne comme un niveau d’application Web) et le niveau SQL (qui fonctionne comme un niveau de données) sont affectés à différents groupes à haute disponibilité. Cela garantit que toutes les instances de chaque niveau disposent d’une redondance matérielle, en distribuant les machines virtuelles dans les domaines d’erreur, et que les niveaux dans leur intégralité ne sont pas arrêtés pendant une mise à jour.

### <a name="load-balancing"></a>Équilibrage de la charge
Si le trafic est distribué entre les machines virtuelles, vous devez les regrouper dans une application et équilibrer la charge sur un point de terminaison TCP ou UDP spécifique. Pour plus d’informations, consultez [Équilibrage de charge des machines virtuelles](/azure/virtual-machines/virtual-machines-linux-load-balance/?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). Si les machines virtuelles sont approvisionnées par une autre source (par exemple, un mécanisme de file d’attente), aucun équilibreur de charge n’est requis. L’équilibreur de charge utilise un contrôle d’intégrité de base pour déterminer si le trafic doit être envoyé au nœud. Il est également possible de créer vos propres sondes pour implémenter des mesures spécifiques de contrôle d’intégrité de l’application. Le cas échéant, elles déterminent si la machine virtuelle doit recevoir le trafic.

## <a name="storage"></a>Stockage
Azure Storage est le service de données durables de ligne de base pour Azure. Il fournit des objets blob, des tables, des files d’attente et du stockage sur disque de machine virtuelle. Il utilise une combinaison de réplication et de gestion des ressources pour assurer la haute disponibilité dans un centre de données unique. Le contrat de niveau de service relatif à la disponibilité d’Azure Storage garantit qu’au moins 99,9 % du temps :

* Des demandes correctement mises en forme pour ajouter, mettre à jour, lire et supprimer des données, sont traitées comme nécessaire.
* Les comptes de stockage pourront se connecter à la passerelle Internet.

### <a name="replication"></a>Réplication
Azure Storage simplifie la durabilité des données en conservant plusieurs copies de toutes les données sur des lecteurs différents situés dans des sous-systèmes de stockage physique entièrement indépendants au sein de la région. Les données sont répliquées de façon synchrone et toutes les copies sont validées avant la reconnaissance de l’écriture. Azure Storage est fortement cohérent, ce qui signifie que les lectures correspondent forcément aux écritures les plus récentes. En outre, les copies de données sont analysées en continu pour détecter et corriger une altération binaire, une menace souvent négligée en matière d’intégrité des données stockées.

Les services bénéficient de la réplication, simplement par l’utilisation d’Azure Storage. Aucune tâche supplémentaire n’est requise par le développeur de service pour la récupération suite à une défaillance locale.

### <a name="resource-management"></a>Gestion des ressources
La taille des comptes de stockage créés après le mai 2014 peut être augmentée jusqu’à 500 To (contre 200 To auparavant). Si de l’espace supplémentaire est nécessaire, les applications doivent être conçues pour utiliser plusieurs comptes de stockage.

### <a name="virtual-machine-disks"></a>Disques de machines virtuelles
Un disque de machine virtuelle est stocké comme un objet blob de pages dans Azure Storage. Par conséquent, il présente les mêmes propriétés de durabilité et d’extensibilité que le stockage d’objets blob. Cette conception rend les données d’un disque de machine virtuelle persistantes, même si le serveur exécutant la machine virtuelle tombe en panne et que la machine virtuelle doit être redémarrée sur un autre serveur.

## <a name="database"></a>Base de données
### <a name="sql-database"></a>Base de données SQL
La Base de données SQL Azure fournit une base de données en tant que service. Elle permet aux applications d’effectuer rapidement des opérations d’approvisionnement, d’insérer des données et d’interroger des bases de données relationnelles. Elle offre de nombreuses fonctionnalités communes de SQL Server, tout en vous épargnant la complexité associée au matériel, à la configuration, à la mise à jour corrective et à la résilience.

> [!NOTE]
> Azure SQL Database ne fournit pas un ensemble de fonctionnalités identique à celui de SQL Server. Il est conçu pour répondre à une série d’exigences spécifiques aux applications cloud (mise à l’échelle élastique, base de données en tant que service pour réduire les coûts de maintenance et ainsi de suite). Pour plus d’informations, consultez [Choisir une option SQL Server cloud : Base de données SQL Azure (PaaS) ou SQL Server sur des machines virtuelles Azure (IaaS)](/azure/sql-database/sql-database-paas-vs-sql-server-iaas/).
> 
> 

#### <a name="replication"></a>Réplication
Azure SQL Database fournit la résilience intégrée dédiée aux défaillances au niveau du nœud. Toutes les écritures dans une base de données sont automatiquement répliquées sur au minimum deux nœuds d’arrière-plan via une technique de validation de quorum. (Le nœud principal et au moins un des nœuds secondaires doivent confirmer que l’activité est écrite dans le journal des transactions avant que la transaction soit considérée comme réussie et qu’elle soit renvoyée.) En cas de défaillance du nœud, la base de données bascule automatiquement vers l’un des réplicas secondaires. Cela entraîne une interruption de connexion temporaire pour les applications clientes. Pour cette raison, tous les clients Azure SQL Database doivent implémenter une certaine forme de gestion des connexions temporaires. Pour plus d’informations, consultez [Guide spécifique relatif au service de nouvelle tentative](/azure/best-practices-retry-service-specific/).

#### <a name="resource-management"></a>Gestion des ressources
Lors de la création, chaque base de données est configurée avec une limite de taille supérieure. La taille maximale actuellement disponible est de 1 To (les limites de taille varient en fonction de votre niveau de service, consultez la section relative aux [niveaux de service et de performances des bases de données Azure SQL](/azure/sql-database/sql-database-resource-limits/#service-tiers-and-performance-levels). Lorsqu’une base de données atteint sa limite de taille maximale, elle refuse les commandes INSERT ou UPDATE supplémentaires. (L’interrogation et la suppression des données sont toujours possibles.)

Dans une base de données, Azure SQL Database utilise une structure pour gérer les ressources. Toutefois, au lieu d’un contrôleur de structure, elle utilise une topologie en anneau pour détecter les erreurs. Dans un cluster, chaque réplica a deux voisins et il est chargé de détecter leur mise hors service. Lorsqu’un réplica est mis hors service, ses voisins déclenchent un Reconfiguration Agent pour le recréer sur une autre machine. La limitation du moteur fournie vous assure qu’un serveur logique n’utilise pas trop de ressources sur une machine ou ne dépasse les limites physiques de la machine.

### <a name="elasticity"></a>Élasticité
Si l’application nécessite davantage que la limite de base de données de 1 To, elle doit implémenter une approche d’augmentation de la taille des instances. Vous augmentez la taille des instances avec Azure SQL Database via un partitionnement manuel des données entre plusieurs bases de données SQL. Cette approche d’augmentation de la taille des instances permet d’obtenir une augmentation quasi linéaire du prix par rapport à la mise à l’échelle. La croissance élastique ou la capacité à la demande peuvent augmenter avec une tarification incrémentielle en fonction des besoins, car les bases de données sont facturées en fonction de la taille moyenne réelle utilisée chaque jour, pas en fonction de la taille maximale possible.

## <a name="sql-server-on-virtual-machines"></a>SQL Server sur Virtual Machines
En installant SQL Server (version 2014 ou ultérieure) sur Azure Virtual Machines, vous pouvez tirer profit des fonctionnalités traditionnelles de disponibilité de SQL Server. Ces fonctionnalités incluent les groupes de disponibilité AlwaysOn et la mise en miroir de bases de données. Veuillez noter que les machines virtuelles Azure, le stockage et le réseau ont des caractéristiques opérationnelles différentes de celles d’une infrastructure informatique non virtualisée locale. Pour une implémentation réussie d’une solution de haute disponibilité/de récupération d’urgence SQL Server dans Azure, vous devez comprendre ces différences et concevoir votre solution de façon à les gérer.

### <a name="high-availability-nodes-in-an-availability-set"></a>Nœuds haute disponibilité d’un groupe à haute disponibilité
Lorsque vous implémentez une solution de haute disponibilité dans Azure, vous pouvez utiliser le groupe à haute disponibilité dans Azure pour placer les nœuds haute disponibilité dans des domaines d’erreur et des domaines de mise à niveau distincts. Pour être clair, le groupe à haute disponibilité est un concept Azure. Nous vous recommandons de suivre cette meilleure pratique pour vous assurer de la haute disponibilité de vos bases de données, que vous utilisiez des groupes de disponibilité AlwaysOn, la mise en miroir de bases de données ou une autre solution. Si vous ne suivez pas cette meilleure pratique, vous pourriez croire, à tort, que votre système est hautement disponible. Mais en réalité, vos nœuds peuvent tous tomber en panne simultanément, car ils se trouvent dans le même domaine d’erreur de la région Azure.

Cette recommandation n’est pas applicable avec la copie des journaux de transaction. Puisque la copie des journaux de transaction est une fonctionnalité de récupération d’urgence, vous devez vous assurer que les serveurs s’exécutent dans des régions Azure distinctes. Par définition, ces régions sont des domaines d’erreur distincts.

Pour que les machines virtuelles Azure Cloud Services déployées via le portail classique se trouvent dans le même groupe à haute disponibilité, vous devez les déployer dans le même service cloud. Les machines virtuelles déployées via Azure Resource Manager (portail actuel) ne présentent pas cette limite. Pour les machines virtuelles déployées sur le portail classique dans Azure Cloud Service, seuls les nœuds du même service cloud peuvent participer au même groupe à haute disponibilité. En outre, les machines virtuelles Cloud Services doivent se trouver dans le même réseau virtuel pour s’assurer qu’elles conservent leur adresse IP même après la réparation du service. Cela vous permet d’éviter les temps d’interruption de mise à jour DNS.

### <a name="azure-only-high-availability-solutions"></a>Azure uniquement : solutions de haute disponibilité
Vous pouvez disposer d’une solution de haute disponibilité pour vos bases de données SQL Server dans Azure à l’aide des groupes de disponibilité AlwaysOn ou de la mise en miroir de bases de données.

Le diagramme suivant illustre l’architecture des groupes de disponibilité AlwaysOn exécutés dans Azure Virtual Machines. Ce diagramme a été extrait de l’article détaillé sur ce sujet, [Haute disponibilité et récupération d’urgence pour SQL Server dans Azure Virtual Machines](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/).

![Groupes de disponibilité AlwaysOn dans Microsoft Azure](./images/technical-guidance-recovery-local-failures/high_availability_solutions-1.png)

Vous pouvez également approvisionner automatiquement un déploiement de groupe de disponibilité AlwaysOn de bout en bout sur les machines virtuelles Azure, en utilisant le modèle AlwaysOn dans le portail Azure. Pour plus d’informations, consultez [Offre AlwaysOn SQL Server dans la galerie du portail Microsoft Azure](https://blogs.technet.microsoft.com/dataplatforminsider/2014/08/25/sql-server-alwayson-offering-in-microsoft-azure-portal-gallery/).

Le diagramme suivant illustre l’utilisation de la mise en miroir de bases de données sur Azure Virtual Machines. Il a également été extrait de la rubrique détaillée [Haute disponibilité et récupération d’urgence pour SQL Server sur des machines virtuelles Azure](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/).

![Mise en miroir de bases de données dans Microsoft Azure](./images/technical-guidance-recovery-local-failures/high_availability_solutions-2.png)

> [!NOTE]
> Dans les deux architectures, un contrôleur de domaine est requis. Toutefois, avec la mise en miroir de bases de données, il est possible d’utiliser des certificats de serveur pour éliminer le recours à un contrôleur de domaine.
> 
> 

## <a name="other-azure-platform-services"></a>Autres services de plateforme Azure
Les applications qui reposent sur Azure bénéficient des fonctionnalités de plateforme pour la récupération suite à des défaillances locales. Dans certains cas, vous pouvez entreprendre des actions spécifiques pour augmenter la disponibilité de votre scénario particulier.

### <a name="service-bus"></a>Service Bus
Pour éviter une panne temporaire d’Azure Service Bus, envisagez de créer une file d’attente durable côté client. Cette fonctionnalité utilise temporairement un autre mécanisme stockage local pour stocker les messages qui ne peuvent pas être ajoutés à la file d’attente Service Bus. L’application peut décider comment traiter les messages temporairement stockés après la restauration du service. Pour plus d’informations, consultez [Bonnes pratiques relatives aux améliorations des performances dans l’utilisation de la messagerie répartie Service Bus](/azure/service-bus-messaging/service-bus-performance-improvements/) et [Service Bus (récupération d’urgence)](recovery-loss-azure-region.md#other-azure-platform-services).

### <a name="hdinsight"></a>HDInsight
Par défaut, les données associées à Azure HDInsight sont stockées dans Stockage Blob Azure. Azure Storage spécifie les propriétés de haute disponibilité et de durabilité pour le stockage d’objets blob. Le traitement à plusieurs nœuds associé à des travaux Hadoop MapReduce est effectué sur un système de fichiers HDFS (Hadoop Distributed File System) temporaire, qui est approvisionné quand HDInsight le requiert. Les résultats d’un travail MapReduce sont également stockés par défaut dans le stockage d’objets blob Azure, afin que les données traitées soient durables et demeurent hautement disponibles une fois le cluster Hadoop déprovisionné. Pour plus d’informations, consultez la section [HDInsight (récupération d’urgence)](recovery-loss-azure-region.md#other-azure-platform-services).

## <a name="checklists-for-local-failures"></a>Listes de contrôle pour les défaillances locales
### <a name="cloud-services"></a>Cloud Services
1. Consultez la section Services cloud de ce document.
2. Configurez au moins deux instances pour chaque rôle.
3. Rendez l’état persistant dans le stockage durable, mais pas sur les instances de rôle.
4. Gérez correctement l’événement StatusCheck.
5. Encapsulez les modifications associées dans les transactions lorsque cela est possible.
6. Vérifiez que les tâches du rôle de travail sont idempotentes et redémarrables.
7. Continuez à appeler les opérations jusqu’à leur réussite.
8. Envisagez de mettre en place des stratégies de mise à l’échelle automatique.

### <a name="virtual-machines"></a>Virtual Machines
1. Consulter la section Machines virtuelles de ce document.
2. N’utilisez pas le lecteur D pour le stockage persistant.
3. Groupez les machines dans un niveau de service d’un groupe à haute disponibilité.
4. Configurez l’équilibrage de charge et les sondes facultatives.

### <a name="storage"></a>Stockage
1. Consultez la section Stockage de ce document.
2. Utilisez plusieurs comptes de stockage lorsque les données ou la bande passante dépassent les quotas.

### <a name="sql-database"></a>Base de données SQL
1. Consultez la section Base de données SQL de ce document.
2. Implémentez une stratégie de nouvelle tentative pour gérer les erreurs temporaires.
3. Utilisez le partitionnement en tant que stratégie d’augmentation de la taille des instances.

### <a name="sql-server-on-virtual-machines"></a>SQL Server sur Virtual Machines
1. Consultez la section SQL Server sur Virtual Machines de ce document.
2. Suivez les recommandations précédentes relatives à Virtual Machines.
3. Utilisez les fonctionnalités de haute disponibilité de SQL Server, telles que AlwaysOn.

### <a name="service-bus"></a>Service Bus
1. Consultez la section Service Bus de ce document.
2. Envisagez de créer une file d’attente durable côté client en tant que sauvegarde.

### <a name="hdinsight"></a>HDInsight
1. Consultez la section HDInsight de ce document.
2. Aucune tâche supplémentaire de disponibilité requise pour les défaillances locales.

