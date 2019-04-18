---
title: Échec et récupération d’urgence pour les applications Azure
description: Approches de la vue d’ensemble de la récupération d’urgence dans Azure
author: MikeWasson
ms.date: 04/10/2019
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.openlocfilehash: f00341b06b8092c798d6260317226190cbace66b
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59646557"
---
# <a name="failure-and-disaster-recovery-for-azure-applications"></a>Échec et récupération d’urgence pour les applications Azure

*Récupération d’urgence* est le processus de restauration des fonctionnalités de l’application à la suite d’une perte catastrophique.

Votre tolérance des fonctionnalités réduites lors d’un sinistre est une décision qui varie d’une application à l’autre. Il peut être acceptable pour certaines applications doit être totalement indisponible ou partiellement disponibles avec des fonctionnalités réduites ou retardée de traitement pour une période donnée. Pour d’autres applications, des fonctionnalités réduites sont inacceptable.

## <a name="disaster-recovery-plan"></a>Plan de récupération d’urgence

Commencez par créer un plan de récupération. Le plan est considérée comme terminé une fois qu’il a été entièrement testée. Inclure les personnes, les processus et les applications nécessaires pour rétablir les fonctionnalités au sein de l’accord de niveau de service (SLA) que vous avez défini pour vos clients.

Prenez en compte les suggestions suivantes lors de la création et le test de votre plan de récupération d’urgence :

- Dans votre plan, incluez le processus pour contacter le support et pour faire remonter les problèmes. Ces informations aideront à éviter les temps d’arrêt prolongé lorsque vous travaillez au cours du processus de récupération pour la première fois.
- Évaluez l’impact commercial des défaillances d’application.
- Choisissez une architecture de récupération de plusieurs régions pour les applications stratégiques.
- Identifiez un propriétaire spécifique du plan de récupération d’urgence, incluant des phases d’automatisation et de test.
- Documentez le processus, en particulier toutes les étapes manuelles.
- Automatiser le processus autant que possible.
- Établir une stratégie de sauvegarde pour tous les référence et les données transactionnelles et test de restauration sauvegarde régulièrement.
- Configurer des alertes pour la pile des services Azure consommés par votre application.
- Former le personnel chargé des opérations d’exécution du plan.
- Effectuer des simulations d’urgence régulières pour valider et améliorer le plan.

Si vous utilisez [Azure Site Recovery](/azure/site-recovery/) pour répliquer des machines virtuelles (VM), créez un plan de récupération entièrement automatisés pour effectuer le basculement de l’application entière.

## <a name="manual-responses"></a>Réponses manuelles

Automation est idéale, certaines stratégies de récupération d’urgence exigent des réponses manuelles.

### <a name="alerts"></a>Alertes

Surveillez votre application pour trouver des signes précurseurs pouvant nécessiter une intervention proactive. Par exemple, si la base de données SQL Azure ou Azure Cosmos DB limite régulièrement votre application, vous devrez peut-être augmenter la capacité de votre base de données ou optimiser vos requêtes. Même si l’application peut gérer les erreurs de limitation en toute transparence, vos données de télémétrie doivent toujours déclencher une alerte afin que vous pouvez suivre.

Pour les limites de service et les seuils de quota, nous vous recommandons de configurer des alertes sur les journaux de métriques et diagnostics de ressources Azure. Dans la mesure du possible, configurez des alertes sur les mesures, qui sont une latence plus faible que les journaux de diagnostics.

Via [Resource Health](/azure/service-health/resource-health-checks-resource-types), Azure fournit certaines d’intégrité intégrés vérifications d’état qui peuvent vous aider à diagnostiquer les problèmes de limitation de service Azure.

### <a name="failover"></a>Basculement

Configurer une stratégie de récupération d’urgence pour chaque application Azure et de ses services Azure. Stratégies de déploiement acceptable pour prendre en charge la récupération d’urgence peuvent varier selon les contrats SLA requis pour tous les composants de chaque application.  

Azure fournit des fonctionnalités différentes dans de nombreux services Azure pour permettre un basculement manuel, tel que [redis cache géo-réplicas](/azure/azure-cache-for-redis/cache-how-to-geo-replication), ou pour automatisé de basculement, tel que [groupes de basculement automatique SQL](/azure/sql-database/sql-database-auto-failover-group). Par exemple : 

- Pour une application qui utilise principalement des machines virtuelles, vous pouvez utiliser Azure Site Recovery pour les niveaux web et logique. Pour plus d’informations, consultez [architecture de récupération d’urgence Azure vers Azure](/azure/site-recovery/azure-to-azure-architecture). Pour SQL Server sur des machines virtuelles, utilisez [groupes de disponibilité SQL Server Always On](/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-availability-group-dr).
- Pour une application qui utilise le Service d’application et de la base de données SQL Azure, vous pouvez utiliser un plan de Service d’application de niveau plus petits configuré dans la région secondaire, qui opère lorsqu’un basculement se produit. Utilisez les groupes de basculement pour le niveau de base de données.

Dans les deux cas, un [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) profil offre pour le basculement automatique entre les régions. [Équilibreurs de charge](/azure/load-balancer/load-balancer-overview) ou [passerelles d’application](/azure/application-gateway/overview) doivent être configurés dans la région secondaire pour prendre en charge une disponibilité plus rapide lors du basculement.

### <a name="operational-readiness-testing"></a>Test de disponibilité opérationnelle

Effectuer un test de disponibilité opérationnelle pour le basculement vers la région secondaire et pour la restauration automatique vers la région primaire. De nombreux services Azure prennent en charge les basculements manuels ou les tests de basculement dans le cadre de simulations de reprise d’activité. Vous pouvez également simuler une panne en cours d’arrêt ou en supprimant des services Azure.

## <a name="application-failure"></a>Échec de l’application

Échecs d’application sont récupérables ou non récupérable. Vous pouvez atténuer une erreur récupérable, mais des erreurs non récupérable provoqueront l’arrêt de l’application.

- Certaines défaillances peuvent être adressés en toute transparence en gérant automatiquement les erreurs et en prenant d’autres mesures. Par exemple, Traffic Manager gère automatiquement les défaillances résultant du logiciel de matériel ou système d’exploitation sous-jacent dans la machine virtuelle hôte.
- Avec des erreurs, l’application peut continuer le traitement des demandes des utilisateurs avec des fonctionnalités réduites.
- Les interruptions de service plus graves peuvent rendre l’application non disponible.

Un système bien conçu sépare les responsabilités au niveau du service &mdash; au moment du design et de l’exécution. Cette séparation empêche une interruption de service dépendant d’arrêter l’application entière. Par exemple, considérez une application de commerce électronique avec les modules suivants :

![AJOUTER UN LIEN À UNE IMAGE CLIPART](./_images/disaster-recovery.png)

Si la base de données pour l’hébergement des commandes tombe en panne, le service de traitement des commandes ne peut pas traiter les transactions commerciales. Selon l’architecture, il peut être impossible pour les services de finalisation des commandes et de traitement des commandes continuer. Toutefois, si les données de produit sont stockées dans un autre emplacement, le catalogue de produits est toujours disponible, même si les autres parties de l’application peuvent être indisponibles.

C’est à vous pour déterminer comment l’application va informer les utilisateurs des problèmes temporaires et pour générer les notifications appropriées dans le système. Dans l’exemple précédent, l’application peut autoriser pour afficher les produits et pour les ajouter à un panier d’achat. Toutefois, lorsque le client tente d’effectuer un achat, l’application doit notifier ces derniers que la fonctionnalité de commande est temporairement indisponible. Bien que pas idéal pour le client, cette approche empêche une interruption du service de l’application.

## <a name="data-corruption-and-restoration"></a>Une restauration et une altération des données

Si un magasin de données échoue, il peut-être des incohérences de données lorsqu’il redevient disponible, en particulier si les données ont été répliquées. Comprendre l’objectif de temps de récupération (RTO) et d’un point de récupération (objectif) des magasins de données répliquées peut vous aider à prédire la quantité de perte de données.

Pour comprendre si le basculement entre les régions est démarré manuellement ou par Microsoft, consultez les contrats SLA de service Azure. Pour les services avec aucun SLA pour le basculement entre régions, Microsoft généralement décide quand basculer et généralement hiérarchise la récupération de données dans la région primaire. Si les données dans la région primaire sont considéré comme irrécupérables, Microsoft bascule vers la région secondaire.

### <a name="restoring-data-from-backups"></a>Restauration des données à partir de sauvegardes

Sauvegardes vous permet de protéger contre la perte d’un composant de l’application en raison d’une suppression accidentelle ou une altération des données. Ils conservent une version fonctionnelle du composant à partir d’un moment antérieur, vous pouvez utiliser pour la restaurer.

Stratégies de récupération d’urgence ne sont pas un substitut pour les sauvegardes, mais les sauvegardes régulières des données d’application prend en charge certains scénarios de récupération d’urgence. Vos choix de stockage de sauvegarde doit être basée sur votre stratégie de récupération d’urgence.

La fréquence d’exécution du processus de sauvegarde détermine votre RPO. Par exemple, si vous effectuez des sauvegardes toutes les heures et un incident survient deux minutes avant la sauvegarde, vous perdrez ainsi 58 minutes de données. Votre plan de récupération d’urgence doit inclure des méthodes de résolution de perte de données.

Il est courant pour les données dans une banque de données aux données de référence dans un autre magasin. Par exemple, considérez une base de données SQL avec une colonne qui accède à un objet blob dans stockage Azure. Si les sauvegardes ne sont pas effectuées simultanément, la base de données peut avoir un pointeur vers un objet blob qui n’a pas été sauvegardé avant la défaillance. L’application ou le plan de récupération d’urgence doit implémenter des processus pour gérer cette incohérence après une récupération.

> [!NOTE]
> Dans certains scénarios, tels que celui de machines virtuelles sauvegardées à l’aide [sauvegarde Azure](/azure/backup/backup-azure-vms-first-look-arm), vous pouvez restaurer uniquement à partir d’une sauvegarde dans la même région. D’autres services Azure, tel que [Cache Azure pour Redis](/azure/azure-cache-for-redis/cache-how-to-geo-replication), fournir des sauvegardes géo-répliquée, que vous pouvez utiliser pour restaurer les services entre les régions.

### <a name="azure-storage-and-azure-sql-database"></a>Stockage Azure et Azure SQL Database

Azure stocke automatiquement les données de stockage Azure et base de données SQL trois fois dans différents domaines d’erreur dans la même région. Si vous utilisez la géoréplication, les données sont stockées dans trois domaines supplémentaires dans une autre région. Toutefois, si les données sont endommagées ou supprimées dans la copie principale (par exemple, en raison d’une erreur de l’utilisateur), la réplication des modifications dans les autres copies.

Vous avez deux options pour la gestion de la corruption potentielle des données ou la suppression :

- **Créer une stratégie de sauvegarde personnalisée.** Vous pouvez stocker vos sauvegardes dans Azure ou en local, en fonction de vos besoins et les règles de gouvernance.
- **Utilisez l’option de restauration de point-à-temps** pour récupérer une base de données SQL.

#### <a name="azure-storage-recovery"></a>Récupération de stockage Azure

Vous pouvez développer un processus de sauvegarde personnalisé pour le stockage Azure ou utiliser un des nombreux outils de sauvegarde tiers.

Stockage Azure fournit la résilience des données via des réplicas automatisés, mais il n’empêche pas les utilisateurs ou code d’application à partir de la corruption des données. La conservation de données fidélité après une erreur d’application ou un utilisateur nécessite des techniques plus avancées, telles que la copie des données vers un emplacement de stockage secondaire avec un journal d’audit. Vous disposez de plusieurs options :

- [**Objets BLOB de blocs.**](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs) Créez un instantané à un point dans le temps de chaque objet blob de blocs. Pour chaque instantané, vous êtes facturé uniquement pour le stockage requis pour stocker les différences dans l’objet blob depuis l’état de capture instantanée précédente. Les instantanés dépendent de l’objet blob d’origine, nous vous recommandons de copier vers un autre objet blob ou même vers un autre compte de stockage. Cette approche garantit que les données de sauvegarde sont protégées contre une suppression accidentelle. Utilisez [AzCopy](/azure/storage/common/storage-use-azcopy) ou [Azure PowerShell](/azure/storage/common/storage-powershell-guide-full) pour copier les objets BLOB vers un autre compte de stockage.

    Pour plus d’informations, consultez [Création d’un instantané d’objet blob](/rest/api/storageservices/creating-a-snapshot-of-a-blob).

- [**Azure Files.**](/azure/storage/files/storage-files-introduction) Utilisez [des instantanés de partage](/azure/storage/files/storage-snapshots-files), AzCopy ou PowerShell pour copier vos fichiers vers un autre compte de stockage.
- [**Stockage Table Azure.**](/azure/storage/tables/table-storage-overview) Utilisez AzCopy pour exporter les données de table vers un autre compte de stockage dans une autre région.

#### <a name="sql-database-recovery"></a>Récupération de la base de données SQL

Pour protéger votre entreprise contre la perte de données, base de données SQL automatiquement effectue une combinaison de sauvegardes de base de données complète une fois par semaine, sauvegardes de base de données différentielle toutes les heures et journaux de transactions sauvegardes chaque 5 à 10 minutes. Pour les niveaux de base, Standard et Premium de SQL Database, utilisez point-à-temps pour restaurer une base de données à une heure antérieure. Pour plus d’informations, consultez les articles suivants :

- [Récupérer une base de données SQL Azure à l’aide des sauvegardes automatisées d’une base de données](/azure/sql-database/sql-database-recovery-using-backups)
- [Vue d’ensemble de la continuité de l’activité avec la base de données Azure SQL](/azure/sql-database/sql-database-business-continuity)

Une autre option consiste à utiliser la géo-réplication active pour base de données SQL, qui réplique automatiquement les modifications de base de données dans des bases de données secondaires dans la région Azure identiques ou différente. Pour plus d’informations, consultez [création et à l’aide de la géo-réplication active](/azure/sql-database/sql-database-active-geo-replication).

Vous pouvez également utiliser une approche plus manuelle pour la sauvegarde et de restauration :

- Utilisez le **copie de base de données** commande pour créer une copie de sauvegarde de la base de données avec une cohérence transactionnelle.
- Utiliser le Service Azure SQL Database Import/Export, qui prend en charge des bases de données exportation vers des fichiers BACPAC (fichiers compressés contenant votre schéma de base de données et les données associées) qui sont stockés dans le stockage Blob Azure. Pour vous protéger contre une interruption de service à l’échelle de la région, copiez les fichiers BACPAC dans une autre région.

### <a name="sql-server-on-vms"></a>SQL Server sur une machine virtuelle

Pour SQL Server s’exécutant sur des machines virtuelles, vous avez deux options : sauvegardes traditionnelles et envoi de journaux.

- Avec les sauvegardes traditionnelles, vous pouvez restaurer à un point spécifique dans le temps, mais le processus de récupération est lent. Restauration de sauvegardes traditionnelles nécessite que vous démarrez avec une sauvegarde complète initiale et puis appliquez toutes les sauvegardes incrémentielles.
- Vous pouvez configurer une session de copie de journal pour retarder la restauration de sauvegardes de journaux. Cela fournit une fenêtre permettant de récupérer des erreurs identifiées sur le réplica principal.

### <a name="azure-database-for-mysql-and-azure-database-for-postgresql"></a>Azure Database pour MySQL et base de données Azure pour PostgreSQL

Dans la base de données Azure pour MySQL et base de données Azure pour PostgreSQL, le service de base de données crée automatiquement une sauvegarde toutes les cinq minutes. Vous pouvez utiliser ces sauvegardes automatisées pour restaurer le serveur et ses bases de données à partir d’un point antérieur dans le temps vers un nouveau serveur. Pour plus d'informations, consultez les pages suivantes :

- [Sauvegarde et restauration d’un serveur Azure Database pour MySQL à l’aide du portail Azure](/azure/mysql/howto-restore-server-portal)
- [Comment sauvegarder et restaurer un serveur de base de données Azure pour PostgreSQL à l’aide du portail Azure](/azure/postgresql/howto-restore-server-portal)

### <a name="azure-cosmos-db"></a>Azure Cosmos DB

COSMOS DB crée automatiquement une sauvegarde à intervalles réguliers. Les sauvegardes sont stockées séparément dans un autre service de stockage et sont répliquées dans le monde entier pour vous protéger contre les sinistres régionaux. En cas de suppression accidentelle de votre base de données ou collection, vous pouvez émettre un ticket de support ou appeler le support technique Azure pour restaurer les données à partir de la dernière sauvegarde automatique. Pour plus d’informations, consultez [sauvegarde en ligne et restauration de la demande dans Azure Cosmos DB](/azure/cosmos-db/online-backup-and-restore).

### <a name="azure-virtual-machines"></a>Machines virtuelles Azure

Pour protéger les Machines virtuelles à partir d’erreurs d’application ou une suppression accidentelle, utilisez [sauvegarde Azure](/azure/backup/). Les sauvegardes créées sont cohérentes sur plusieurs disques de machine virtuelle. En outre, le coffre de sauvegarde Azure peut être répliqué entre les régions pour prendre en charge la récupération d’une perte régionale.

## <a name="network-outage"></a>Panne du réseau

Lorsque certaines parties du réseau Azure sont inaccessibles, vous n’êtes peut-être pas en mesure d’accéder aux données ou votre application. Dans ce cas, nous vous recommandons de conception de la stratégie de récupération d’urgence pour exécuter la plupart des applications avec des fonctionnalités réduites.

Si la réduction des fonctionnalités n’est pas une option, les options restantes sont des interruptions de service ou le basculement vers une autre région.

Dans un scénario de fonctionnalité réduite :

- Si votre application ne peut pas accéder à ses données en raison d’une panne du réseau Azure, vous pourrez peut-être exécuter localement avec fonctionnalités réduites des applications à l’aide des données mises en cache.
- Vous serez peut-être en mesure de stocker des données dans un autre emplacement jusqu'à ce que la connectivité est rétablie.

## <a name="dependent-service-failure"></a>Échec des services dépendants

Pour chaque service dépendant, vous devez comprendre les implications d’une interruption de service et la façon dont l’application doit répondre. De nombreux services incluent des fonctionnalités qui prennent en charge la résilience et la disponibilité, par conséquent, l’évaluation de chaque service indépendamment est susceptible d’améliorer votre plan de récupération d’urgence. Par exemple, Azure Event Hubs prend en charge [basculement](/azure/event-hubs/event-hubs-geo-dr#setup-and-failover-flow) à l’espace de noms secondaire.

## <a name="region-wide-service-disruptions"></a>Interruptions de service à l’échelle de la région

Nombre d’erreurs est faciles à gérer au sein de la même région Azure. Toutefois, dans l’éventualité d’une interruption de service à l’échelle de la région, les copies localement redondantes de vos données ne sont pas disponibles. Si vous avez activé la géo-réplication, il existe trois copies supplémentaires de vos objets BLOB et les tables dans une autre région. Si Microsoft déclare la région perdue, Azure remappe toutes les entrées DNS vers la région secondaire.

> [!NOTE]
> Ce processus se produit uniquement pour les interruptions de service à l’échelle de la région et n’est pas au sein de votre contrôle. Envisagez d’utiliser [Azure Site Recovery](/azure/site-recovery/) pour obtenir de meilleurs RPO et RTO. À l’aide de la récupération de Site, vous décidez qu’une indisponibilité est acceptable et quand basculer vers les machines virtuelles répliquées.

Votre réponse à une interruption de service à l’échelle de la région dépend de votre déploiement et de votre plan de récupération d’urgence.

- En tant qu’une stratégie de contrôle des coûts pour les applications non critiques qui ne nécessitent pas un temps de récupération garanti, il peut être utile pour redéployer vers une autre région.
- Pour les applications qui sont hébergées dans une autre région avec les rôles déployés, mais ne pas distribuer le trafic entre les régions (*déploiement actif/passif*), basculez vers le service hébergé secondaire dans l’autre région.
- Pour les applications qui ont un déploiement à grande échelle secondaire dans une autre région (*déploiement actif/actif*), acheminer le trafic vers cette région.

Pour en savoir plus sur la récupération d’une interruption de service à l’échelle de la région, consultez [récupérer à partir d’une interruption de service à l’échelle de la région](../resiliency/recovery-loss-azure-region.md).

### <a name="vm-recovery"></a>Récupération de machines virtuelles

Pour les applications critiques, planifiez la récupération de machines virtuelles en cas d’une interruption de service à l’échelle de la région.

- Utilisation d’Azure Backup ou une autre méthode de sauvegarde à créer entre les régions des sauvegardes cohérentes des applications. (La réplication du coffre de sauvegarde doit être configurée au moment de la création.)
- Utilisez Site Recovery pour répliquer les régions pour le basculement d’application d’un seul clic et un test de basculement.
- Traffic Manager permet d’automatiser le basculement du trafic utilisateur vers une autre région.

Pour plus d’informations, consultez [récupérer à partir d’une interruption de service à l’échelle de la région, les machines virtuelles](../resiliency/recovery-loss-azure-region.md#virtual-machines).

### <a name="storage-recovery"></a>Récupération du stockage

Pour protéger votre stockage en cas d’une interruption de service à l’échelle de la région :

- Utilisation du stockage géo-redondant.
- Savoir où votre espace de stockage géo-répliquée. Cela affecte où vous déployez d’autres instances de vos données qui nécessitent une affinité régionale avec votre stockage.
- Vérifier la cohérence des données après le basculement et, si nécessaire, restaurez à partir d’une sauvegarde.

Pour plus d’informations, consultez [conception d’applications hautement disponibles à l’aide de RA-GRS](/azure/storage/common/storage-designing-ha-apps-with-ragrs).

### <a name="sql-database-and-sql-server"></a>Base de données SQL et SQL Server

Azure SQL Database fournit deux types de récupération :

- Utilisez la géo-restauration pour restaurer une base de données à partir d’une copie de sauvegarde dans une autre région. Pour plus d’informations, consultez [récupérer une base de données SQL Azure à l’aide de sauvegardes de bases de données automatisées](/azure/sql-database/sql-database-recovery-using-backups).
- Géo-réplication active permet de basculer vers une base de données secondaire. Pour plus d’informations, consultez [création et à l’aide de la géo-réplication active](/azure/sql-database/sql-database-active-geo-replication).

Pour SQL Server s’exécutant sur des machines virtuelles, consultez [haute disponibilité et récupération d’urgence pour SQL Server dans les Machines virtuelles](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/).

## <a name="service-specific-guidance"></a>Guide spécifique relatif au service

Les articles suivants décrivent la récupération d’urgence pour certains services Azure :

| de diffusion en continu                       | Article                                                                                                                                                                                       |
|-------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Azure Database pour MySQL      | [Vue d’ensemble de la continuité d’activité avec Azure Database pour MySQL](/azure/mysql/concepts-business-continuity)                                                  |
| Azure Database pour PostgreSQL | [Vue d’ensemble de la continuité d’activité avec Azure Database pour PostgreSQL](/azure/postgresql/concepts-business-continuity)                                        |
| Services cloud Azure          | [Que faire si une interruption de service Azure affecte Azure Cloud Services](/azure/cloud-services/cloud-services-disaster-recovery-guidance) |
| Cosmos DB                     | [Haute disponibilité avec Azure Cosmos DB](/azure/cosmos-db/high-availability)                                                                                |
| Azure Key Vault               | [Disponibilité et redondance d’Azure Key Vault](/azure/key-vault/key-vault-disaster-recovery-guidance)                                                        |
| Stockage Azure                 | [Récupération d'urgence et basculement de compte de stockage (préversion) dans Stockage Azure](/azure/storage/common/storage-disaster-recovery-guidance)                       |
| Base de données SQL                  | [Restaurer une base de données SQL Azure ou basculer vers une région secondaire](/azure/sql-database/sql-database-disaster-recovery)                                       |
| Virtual Machines              | [Ce qu’il faut en cas d’Azure interruption de service affecte Azure Cloud](/azure/cloud-services/cloud-services-disaster-recovery-guidance)               |
| Réseau virtuel Azure         | [Réseau virtuel – Continuité des activités](/azure/virtual-network/virtual-network-disaster-recovery-guidance)                                                  |

## <a name="next-steps"></a>Étapes suivantes

- [Récupérer suite à une altération des données ou une suppression accidentelle](../resiliency/recovery-data-corruption.md)
- [Récupérer à partir d’une interruption de service à l’échelle de la région](../resiliency/recovery-loss-azure-region.md)