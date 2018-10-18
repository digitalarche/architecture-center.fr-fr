---
title: Liste de vérification de la résilience pour les services Azure
description: Liste de vérification qui fournit des conseils de résilience pour divers services Azure.
author: petertaylor9999
ms.date: 03/02/2018
ms.custom: resiliency, checklist
ms.openlocfilehash: 50808a837132e905cc89c3c43d40852a04f4885c
ms.sourcegitcommit: b2a4eb132857afa70201e28d662f18458865a48e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2018
ms.locfileid: "48819191"
---
# <a name="resiliency-checklist-for-specific-azure-services"></a>Liste de vérification de la résilience pour des services Azure spécifiques

La résilience est la capacité d’un système à récupérer après des défaillances et à continuer de fonctionner ; elle est l’un des [piliers de la qualité logicielle](../guide/pillars.md). Chaque technologie a ses propres modes d’échec, que vous devez prendre en compte lorsque vous concevez et implémentez votre application. Utilisez cette liste de vérification pour passer en revue les considérations relatives à la résilience de services Azure spécifiques. Examinez également la [liste de vérification de résilience générale](./resiliency.md).

## <a name="app-service"></a>App Service

**Utilisez le niveau Standard ou Premium.** Ces niveaux prennent en charge les emplacements de préproduction et les sauvegardes automatisées. Pour plus d’informations, consultez la rubrique [Présentation détaillée des plans Azure App Service](/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview/)

**Évitez de monter ou de descendre en puissance.** À la place, sélectionnez un niveau et une taille d’instance répondant à vos besoins de performances sous une charge normale, puis [augmentez la taille des instances](/azure/app-service-web/web-sites-scale/) pour gérer les changements dans le volume de trafic. Le fait de monter ou de descendre en puissance peut déclencher un redémarrage de l’application.  

**Stockez la configuration en tant que paramètres de l’application.** Utilisez les paramètres de l’application pour conserver les paramètres de configuration en tant que paramètres de l’application. Définissez les paramètres dans vos modèles Resource Manager ou à l’aide de PowerShell afin de pouvoir les appliquer dans le cadre d’un processus de déploiement/mise à jour automatisé, ce qui est plus fiable. Pour plus d’informations, consultez [Configurer des applications web dans Azure App Service](/azure/app-service-web/web-sites-configure/).

**Créez des plans App Service distincts pour la production et les tests.** N’utilisez pas les emplacements de votre déploiement de production pour les tests.  Toutes les applications qui se trouvent dans un même plan App Service partagent les mêmes instances de machines virtuelles. Si vous placez des déploiements de production et de test dans le même plan, cela peut avoir un impact négatif sur le déploiement de production. Par exemple, des tests de charge peuvent dégrader le site de production réelle. En plaçant les déploiements de test dans un plan séparé, vous les isolez de la version de production.  

**Séparez les applications web des API web.** Si votre solution présente à la fois un serveur web frontal et une API web, envisagez de les décomposer en applications App Service distinctes. Cette conception facilite la décomposition de la solution par charge de travail. Vous pouvez exécuter l’application web et l’API dans des plans App Service distincts et donc les mettre à l’échelle indépendamment l’une de l’autre. Si vous n’avez pas besoin de ce niveau d’extensibilité au départ, vous pouvez déployer les applications dans le même plan et les déplacer ultérieurement dans des plans distincts si nécessaire.

**Évitez d’utiliser la fonctionnalité de sauvegarde d’App Service pour sauvegarder des bases de données SQL Azure.** Utilisez les [sauvegardes automatisées SQL Database][sql-backup] à la place. La fonctionnalité de sauvegarde d’App Service exporte la base de données dans un fichier .bacpac SQL, ce qui consomme des DTU.  

**Tirez parti d’un emplacement de préproduction pour vos déploiements.** Créez un emplacement de déploiement pour la préproduction. Déployez les mises à jour de l’application dans l’emplacement de préproduction et vérifiez le déploiement avant de le basculer en production. Cela réduit le risque de déploiement d’une mise à jour incorrecte en production. De plus, toutes les instances seront ainsi préparées avant d’être basculées en production. Un grand nombre d’applications présentent un temps de préparation et de démarrage à froid important. Pour plus d’informations, consultez [Configurer des environnements intermédiaires pour les applications web dans Azure App Service](/azure/app-service-web/web-sites-staged-publishing/).

**Créez un emplacement de déploiement pour stocker le dernier déploiement valide connu.** Quand vous déployez une mise à jour en production, déplacez le déploiement de production précédent dans l’emplacement réservé au dernier déploiement valide connu. Vous pourrez ainsi plus facilement effectuer une restauration en cas de déploiement incorrect. Si vous détectez un problème plus tard, vous pourrez rapidement revenir à la dernière version correcte connue. Pour plus d’informations, voir la rubrique [Application web de base](../reference-architectures/app-service-web-app/basic-web-app.md).

**Activez la journalisation des diagnostics**, y compris la journalisation d’application la journalisation de serveur web. La journalisation est importante pour la surveillance et les diagnostics. Consultez [Activer la journalisation des diagnostics pour les applications web dans Azure App Service](/azure/app-service-web/web-sites-enable-diagnostic-log/).

**Effectuez la journalisation dans un stockage Blob.** Cela facilite la collecte et l’analyse des données.

**Créez un compte de stockage distinct pour les journaux.** N’utilisez pas le même compte de stockage pour les journaux et les données d’application. Cela permet d’empêcher la journalisation de dégrader les performances de l’application.

**Analysez les performances.** Utilisez un service d’analyse des performances tel que [New Relic](https://newrelic.com/) ou [Application Insights](/azure/application-insights/app-insights-overview/) pour analyser les performances et le comportement de l’application sous charge.  L’analyse des performances vous offre des renseignements en temps réel sur l’application. Elle vous permet de diagnostiquer les problèmes et d’effectuer une analyse des causes premières des échecs.

## <a name="application-gateway"></a>Application Gateway

**Approvisionnez au moins deux instances.** Déployez une passerelle Application Gateway avec au moins deux instances. Une seule instance constitue un point de défaillance unique. Utilisez deux instances ou plus à des fins de redondance et d’extensibilité. Afin d’être éligible au [SLA](https://azure.microsoft.com/support/legal/sla/application-gateway), vous devez approvisionner au moins deux instances moyennes ou grandes.

## <a name="cosmos-db"></a>Cosmos DB

**Répliquez la base de données dans différentes régions.** Cosmos DB vous permet d’associer n’importe quel nombre de régions Azure à un compte de base de données Cosmos DB. Une base de données Cosmos DB peut présenter une région d’écriture et plusieurs régions de lecture. En cas de défaillance dans la région d’écriture, vous pouvez lire les données à partir d’un autre réplica. Le Kit de développement logiciel (SDK) client gère cela automatiquement. Vous pouvez également basculer la région d’écriture vers une autre région. Pour en savoir plus, voir [Comment distribuer des données mondialement avec Azure Cosmos DB](/azure/cosmos-db/distribute-data-globally).

## <a name="event-hubs"></a>Event Hubs

**Utilisez des points de contrôle**.  Un consommateur d’événements doit écrire sa position actuelle sur le stockage persistant selon un intervalle prédéfini. De cette façon, si le consommateur rencontre une erreur (par exemple, blocage du consommateur ou échec de l’hôte), une nouvelle instance peut reprendre la lecture du flux à partir de la dernière position enregistrée. Pour en savoir plus, consultez la section relative aux [consommateurs d’événements](/azure/event-hubs/event-hubs-features#event-consumers).

**Gérer les messages en double.** Si un consommateur d’événements échoue, le traitement des messages reprend à partir du dernier point de contrôle enregistré. Les messages qui ont déjà été traités après le dernier point de contrôle seront traités à nouveau. Par conséquent, votre logique de traitement des messages doit être idempotente, ou l’application doit être en mesure de dédupliquer des messages.

**Traiter les exceptions**. En règle générale, un consommateur d’événements traite un lot de messages dans une boucle. Vous devez gérer les exceptions au sein de cette boucle de traitement, afin d’éviter de perdre un lot de messages entier lorsqu’un seul message provoque une exception.

**Utiliser une file d’attente de lettres mortes.** Si le traitement d’un message entraîne une défaillance non temporaire, placez le message dans une file d’attente de lettres mortes, afin de pouvoir suivre l’état. Selon le scénario, vous pourrez ultérieurement essayer de traiter le message une nouvelle fois, appliquer une transaction de compensation ou exécuter une autre action. Notez qu’un hub Event Hubs n’inclut aucune fonctionnalité de file d’attente de lettres mortes intégrée. Vous pouvez utiliser le Stockage File d’attente Azure ou Service Bus pour implémenter une file d’attente de lettres mortes, ou utiliser Azure Functions ou tout autre mécanisme d’événement.  

**Implémenter la récupération d’urgence via le basculement vers un espace de noms Event Hubs secondaire.** Pour en savoir plus, voir [Géorécupération d’urgence Azure Event Hubs](/azure/event-hubs/event-hubs-geo-dr).

## <a name="redis-cache"></a>Cache Redis

**Configurez la géoréplication**. La géoréplication fournit un mécanisme permettant de lier deux instances Cache Redis Azure de niveau Premium. Les données écrites dans le cache principal sont répliquées dans un cache secondaire en lecture seule. Pour plus d’informations, consultez [Comment configurer la géoréplication pour Cache Redis Azure](/azure/redis-cache/cache-how-to-geo-replication).

**Configurez la persistance des données.** La persistance Redis vous permet de conserver les données stockées dans Redis. Vous pouvez également prendre des instantanés et sauvegarder les données que vous pouvez charger en cas de défaillance matérielle. Pour plus d’informations, consultez [Comment configurer la persistance des données pour un Cache Redis Azure Premium](/azure/redis-cache/cache-how-to-premium-persistence).

Si vous utilisez Cache Redis comme un cache de données temporaire et non comme un magasin persistant, ces recommandations peuvent ne pas s’appliquer. 

## <a name="search"></a>Recherche

**Approvisionnez plusieurs réplicas.** Utilisez au moins deux réplicas pour une haute disponibilité en lecture ou trois pour une haute disponibilité en lecture-écriture.

**Configurez des indexeurs pour les déploiements multirégions.** Si vous disposez d’un déploiement multirégion, prenez en considération vos options pour la continuité de l’indexation.

  * Si la source de données est géorépliquée, vous devez généralement pointer chaque indexeur de chaque service Recherche Azure sur son réplica de source de données local. Cependant, cette approche n’est pas recommandée pour les jeux de données volumineux stockés dans Azure SQL Database. En effet, Recherche Azure peut effectuer une indexation incrémentielle à partir des réplicas SQL Database principaux, mais pas à partir des réplicas secondaires. Pointez tous les indexeurs sur le réplica principal à la place. Après un basculement, pointez les indexeurs Recherche Azure sur le nouveau réplica principal.  
  * Si la source de données n’est pas géorépliquée, pointez plusieurs indexeurs sur la même source de données afin que les services Recherche Azure de plusieurs régions effectuent une indexation continue et indépendante à partir de la source de données. Pour plus d’informations, consultez [Considérations sur les performances et l’optimisation de Recherche Azure][search-optimization].

## <a name="service-bus"></a>Service Bus

**Utilisez le niveau Premium pour les charges de travail de production**. [Service Bus Premium Messaging](/azure/service-bus-messaging/service-bus-premium-messaging) met à disposition des ressources de traitement dédiées et réservées, ainsi qu’une capacité mémoire permettant de prendre en charge des performances et des débits prévisibles. Le niveau Premium Messaging donne également accès à de nouvelles fonctionnalités réservées dans un premier temps aux clients Premium. Vous pouvez décider du nombre d’unités de messagerie en fonction de la charge de travail prévue.

**Gérer les messages en double**. Si un serveur de publication échoue immédiatement après l’envoi d’un message, ou s’il rencontre des problèmes de réseau ou de système, il peut par erreur ne pas enregistrer que le message a été livré et envoyer deux fois le même message au système. La messagerie Service Bus peut gérer ce problème en permettant de détecter les doublons. Pour en savoir plus, consultez [Détection des doublons](/azure/service-bus-messaging/duplicate-detection).

**Gérer les exceptions**. Les API de messagerie génèrent des exceptions lorsqu’une erreur utilisateur, une erreur de configuration ou toute autre erreur se produit. Le code client (expéditeur et destinataire) doit traiter ces exceptions dans son code. Ce point est particulièrement important dans le traitement par lots, où la gestion des exceptions permet d’éviter de perdre la totalité d’un lot de messages. Pour plus d’informations, consultez [Exceptions de la messagerie Service Bus](/azure/service-bus-messaging/service-bus-messaging-exceptions).

**Stratégie de nouvelle tentative**. La messagerie Service Bus permet de choisir la meilleure stratégie de nouvelle tentative pour vos applications. La stratégie par défaut consiste à autoriser un maximum de 9 nouvelles tentatives et à attendre 30 secondes, mais ce délai peut être modifié ultérieurement. Pour en savoir plus, consultez [Stratégie de nouvelle tentative : messagerie Service Bus](/azure/architecture/best-practices/retry-service-specific#service-bus).

**Utiliser une file d’attente de lettres mortes**. Si un message ne peut pas être traité ou transmis à un destinataire après plusieurs tentatives, il est placé dans une file d’attente de lettres mortes. Implémentez un processus permettant de lire les messages de la file d’attente des lettres mortes, de les inspecter et de remédier au problème. Selon le scénario, vous pouvez effectuer une nouvelle tentative de message tel quel, apporter des modifications et effectuer une nouvelle tentative, ou ignorer le message. Pour en savoir plus, consultez [Vue d’ensemble des files d’attente de lettres mortes Service Bus](/azure/service-bus-messaging/service-bus-dead-letter-queues).

**Utiliser la géorécupération d’urgence**. La géorécupération d’urgence permet de s’assurer que le traitement des données continue à fonctionner dans une autre région ou un autre centre de données si toute une région Azure ou tout un centre de données Azure n’est plus accessible suite à une urgence. Pour plus d’informations, consultez [Géorécupération d’urgence Azure Service Bus](/azure/service-bus-messaging/service-bus-geo-dr).


## <a name="storage"></a>Stockage

**Pour les données d’application, utilisez le stockage géographiquement redondant avec accès en lecture (RA-GRS).** Le stockage RA-GRS réplique les données vers une région secondaire et offre un accès en lecture seule à partir de la région secondaire. En cas de panne de stockage dans la région primaire, l’application peut lire les données à partir de la région secondaire. Pour plus d’informations, consultez l’article [Réplication de Stockage Azure](/azure/storage/storage-redundancy/).

**Pour les disques de machine virtuelle, utilisez Managed Disks.** Le service [Managed Disks][managed-disks] augmente la fiabilité des machines virtuelles dans un groupe à haute disponibilité en isolant suffisamment les disques les uns des autres pour éviter les points de défaillance uniques. Par ailleurs, les disques Managed Disks ne sont pas soumis aux limites d’IOPS des disques durs virtuels créés dans un compte de stockage. Pour plus d’informations, consultez [Gérer la disponibilité des machines virtuelles Windows dans Azure][vm-manage-availability].

**Pour le service Stockage File d’attente, créez une file d’attente de sauvegarde dans une autre région.** Pour le service Stockage File d’attente, un réplica en lecture seule présente peu d’utilité, car vous ne pouvez pas mettre des éléments en file d’attente ou les en enlever. Créez une file d’attente de sauvegarde dans une autre région à la place. En cas de panne de stockage, l’application peut utiliser la file d’attente de sauvegarde jusqu’à ce que la région primaire redevienne disponible. De cette façon, l’application peut continuer à traiter les nouvelles requêtes.  

## <a name="sql-database"></a>Base de données SQL

**Utilisez le niveau Standard ou Premium.** Ces niveaux offrent une limite de restauration dans le temps plus importante (35 jours). Pour plus d’informations, consultez les [options et performances de SQL Database](/azure/sql-database/sql-database-service-tiers/).

**Activez l’audit SQL Database.** L’audit peut être utilisé pour identifier les attaques malveillantes et les erreurs humaines. Pour en savoir plus, voir [Prise en main de l’audit de base de données SQL](/azure/sql-database/sql-database-auditing-get-started/).

**Utilisez la géoréplication active.** Utilisez la géoréplication active pour créer une base de données secondaire accessible en lecture dans une autre région.  Si votre base de données primaire échoue ou doit simplement être mise hors connexion, vous pouvez effectuer un basculement manuel vers la base de données secondaire.  La base de données secondaire reste en lecture seule jusqu’à ce que vous effectuiez le basculement vers cette dernière.  Pour plus d’informations, consultez [Géoréplication active de SQL Database](/azure/sql-database/sql-database-geo-replication-overview/).

**Utilisez le partitionnement.** Envisagez d’utiliser le partitionnement pour partitionner la base de données horizontalement. Le partitionnement peut assurer une isolation des erreurs. Pour plus d’informations, consultez [Montée en charge avec Azure SQL Database](/azure/sql-database/sql-database-elastic-scale-introduction/).

**Utilisez la limite de restauration dans le temps pour récupérer des erreurs humaines.**  La limite de restauration dans le temps permet de revenir à un état antérieur de la base de données. Pour plus d’informations, consultez [Récupérer une base de données SQL Azure à l’aide des sauvegardes automatisées d’une base de données][sql-restore].

**Utilisez la géorestauration pour récupérer d’une panne de service.** La géorestauration assure la restauration d’une base de données à partir d’une sauvegarde géoredondante.  Pour plus d’informations, consultez [Récupérer une base de données SQL Azure à l’aide des sauvegardes automatisées d’une base de données][sql-restore].

## <a name="sql-data-warehouse"></a>SQL Data Warehouse

**Ne désactivez pas la géosauvegarde.** Par défaut, SQL Data Warehouse effectue une sauvegarde complète de vos données toutes les 24 heures en cas de récupération d’urgence. Nous vous recommandons de ne pas désactiver cette fonctionnalité. Pour plus d’informations, consultez [Géosauvegardes](/azure/sql-data-warehouse/backup-and-restore#geo-backups).

## <a name="sql-server-running-in-a-vm"></a>SQL Server exécuté dans une machine virtuelle

**Répliquez la base de données.** Utilisez les groupes de disponibilité Always On SQL Server pour répliquer la base de données. Vous bénéficierez ainsi d’une haute disponibilité en cas d’échec d’une instance SQL Server. Pour plus d’informations, consultez [Exécuter des machines virtuelles Windows pour une architecture multiniveau](../reference-architectures/virtual-machines-windows/n-tier.md).

**Sauvegardez la base de données.** Si vous utilisez déjà le service [Sauvegarde Azure](https://azure.microsoft.com/documentation/services/backup/) pour sauvegarder vos machines virtuelles, envisagez d’utiliser [Sauvegarde Azure pour les charges de travail SQL Server à l’aide de DPM](/azure/backup/backup-azure-backup-sql/). Avec cette approche, il y a un rôle d’administrateur de sauvegarde pour l’organisation et une procédure de récupération unifiée pour les machines virtuelles et SQL Server. Autrement, utilisez la [sauvegarde gérée SQL Server dans Microsoft Azure](https://msdn.microsoft.com/library/dn449496.aspx).

## <a name="traffic-manager"></a>Traffic Manager

**Effectuez une restauration manuelle.** Après un basculement Traffic Manager, effectuez une restauration manuelle plutôt que de recourir à la restauration automatique. Avant d’effectuer la restauration, vérifiez que tous les sous-systèmes de l’application sont intègres.  Sinon, l’application risque d’alterner continuellement entre les centres de données. Pour plus d’informations, consultez [Exécuter des machines virtuelles Windows dans plusieurs régions à des fins de haute disponibilité](../reference-architectures/virtual-machines-windows/multi-region-application.md).

**Créez un point de terminaison de sonde d’intégrité.** Créez un point de terminaison personnalisé qui rend compte de l’intégrité globale de l’application. Cela permet à Traffic Manager d’effectuer un basculement en cas d’échec d’un chemin critique, et pas simplement du serveur frontal. Le point de terminaison doit retourner un code d’erreur HTTP si une dépendance critique est défectueuse ou inaccessible. Cependant, ne signalez pas les erreurs liées aux services non critiques. Sinon, la sonde d’intégrité risque de déclencher un basculement inutilement, créant de faux positifs. Pour plus d’informations, consultez [Surveillance et basculement des points de terminaison Traffic Manager](/azure/traffic-manager/traffic-manager-monitoring/).

## <a name="virtual-machines"></a>Virtual Machines

**Évitez d’exécuter une charge de travail de production sur une machine virtuelle unique.** Un déploiement de machine virtuelle unique ne résiste pas aux maintenances planifiées ou non planifiées. Au lieu de cela, placez plusieurs machines virtuelles dans un groupe à haute disponibilité ou un [groupe de machines virtuelles identiques](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview/), avec un équilibreur de charge à l’avant.

**Spécifiez un groupe à haute disponibilité quand vous approvisionnez la machine virtuelle.** Actuellement, il n’existe aucun moyen d’ajouter une machine virtuelle à un groupe à haute disponibilité après l’approvisionnement de la machine virtuelle. Quand vous ajoutez une nouvelle machine virtuelle à un groupe à haute disponibilité, veillez à créer une carte réseau pour la machine virtuelle, puis ajoutez cette carte au pool d’adresses principal sur l’équilibreur de charge. Sinon, l’équilibreur de charge n’acheminera pas le trafic vers cette machine virtuelle.

**Placez chaque couche Application dans un groupe à haute disponibilité distinct.** Dans une application multiniveau, ne placez pas de machines virtuelles de niveaux différents dans le même groupe à haute disponibilité. Les machines virtuelles d’un groupe à haute disponibilité sont réparties dans des domaines d’erreur et des domaines de mise à jour. Toutefois, pour tirer parti de la redondance des domaines d’erreur et des domaines de mise à jour, chaque machine virtuelle du groupe à haute disponibilité doit être en mesure de gérer les mêmes requêtes de clients.

**Choisissez la taille de machine virtuelle appropriée en fonction des exigences de performances.** Quand vous déplacez une charge de travail existante vers Azure, commencez par choisir la taille de machine virtuelle qui correspond le mieux à vos serveurs locaux. Mesurez ensuite les performances de votre charge de travail réelle en termes de processeur, de mémoire et d’IOPS de disque, puis ajustez la taille si nécessaire. Vous aurez ainsi l’assurance que l’application se comportera comme prévu dans un environnement cloud. En outre, si vous avez besoin de plusieurs cartes réseau, tenez compte de la limite de la carte réseau pour chaque taille.

**Utilisez le service Managed Disks pour les disques durs virtuels.** Le service [Managed Disks][managed-disks] augmente la fiabilité des machines virtuelles dans un groupe à haute disponibilité en isolant suffisamment les disques les uns des autres pour éviter les points de défaillance uniques. Par ailleurs, les disques Managed Disks ne sont pas soumis aux limites d’IOPS des disques durs virtuels créés dans un compte de stockage. Pour plus d’informations, consultez [Gérer la disponibilité des machines virtuelles Windows dans Azure][vm-manage-availability].

**Installez les applications sur un disque de données plutôt que sur le disque du système d’exploitation.** Sinon, il se peut que vous atteigniez la limite de taille du disque.

**Utilisez le service Sauvegarde Azure pour sauvegarder les machines virtuelles.** Les sauvegardes assurent une protection contre la perte accidentelle de données. Pour plus d’informations, consultez [Protéger les machines virtuelles Azure avec un coffre Recovery Services](/azure/backup/backup-azure-vms-first-look-arm/).

**Activez les journaux de diagnostic**, y compris les mesures d’intégrité de base, les journaux d’infrastructure et les [diagnostics de démarrage][boot-diagnostics]. Les diagnostics de démarrage peuvent vous aider à identifier le problème de démarrage si votre machine virtuelle refuse de démarrer. Pour plus d’informations, consultez [Présentation des journaux de diagnostic Azure][diagnostics-logs].

**Utilisez l’extension AzureLogCollector** (machines virtuelles Windows uniquement). Cette extension agrège les journaux de la plateforme Azure et les charge dans le stockage Azure, sans que l’opérateur ait à se connecter à distance à la machine virtuelle. Pour plus d’informations, consultez [Extension AzureLogCollector](/azure/virtual-machines/virtual-machines-windows-log-collector-extension/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

## <a name="virtual-network"></a>Réseau virtuel

**Pour mettre sur liste verte ou bloquer des adresses IP publiques, ajoutez un groupe de sécurité réseau (NSG) au sous-réseau.** Bloquez l’accès aux utilisateurs malveillants ou autorisez l’accès uniquement aux utilisateurs qui disposent du privilège requis pour accéder à l’application.  

**Créez une sonde d’intégrité personnalisée.** Les sondes d’intégrité de Load Balancer peuvent tester le protocole TCP ou HTTP. Si une machine virtuelle exécute un serveur HTTP, la sonde HTTP est un meilleur indicateur de l’état d’intégrité qu’une sonde TCP. Pour une sonde HTTP, utilisez un point de terminaison personnalisé qui signale l’intégrité globale de l’application, y compris toutes les dépendances critiques. Pour plus d’informations, consultez [Vue d’ensemble de Azure Load Balancer](/azure/load-balancer/load-balancer-overview/).

**Ne bloquez la sonde d’intégrité.** La sonde d’intégrité de Load Balancer est envoyée à partir d’une adresse IP connue, à savoir 168.63.129.16. Ne bloquez le trafic à destination ou en provenance de cette adresse IP dans aucune stratégie de pare-feu ou règle NSG. Si vous bloquez la sonde d’intégrité, l’équilibreur de charge supprimera la machine virtuelle de la rotation.

**Activez la journalisation de Load Balancer.** Les journaux indiquent combien de machines virtuelles sur le serveur principal ne reçoivent pas de trafic réseau en raison d’une absence de réponse de la part de la sonde. Pour plus d’informations, consultez [Analyse des journaux d’Azure Load Balancer](/azure/load-balancer/load-balancer-monitor-log/).

<!-- links -->
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[diagnostics-logs]: /azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs/
[managed-disks]: /azure/storage/storage-managed-disks-overview
[search-optimization]: /azure/search/search-performance-optimization/
[sql-backup]: /azure/sql-database/sql-database-automated-backups/
[sql-restore]: /azure/sql-database/sql-database-recovery-using-backups/
[vm-manage-availability]: /azure/virtual-machines/windows/manage-availability#use-managed-disks-for-vms-in-an-availability-set
