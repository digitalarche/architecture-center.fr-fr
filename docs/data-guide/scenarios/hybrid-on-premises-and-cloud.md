---
title: "Extension des solutions de données locales vers le cloud"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 66fc225dc123202ba587d82f15ea0883e1bbf3b5
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/14/2018
---
# <a name="extending-on-premises-data-solutions-to-the-cloud"></a>Extension des solutions de données locales vers le cloud

Lorsque les organisations déplacent des charges de travail et des données vers le cloud, leurs centres de données locaux continuent souvent à jouer un rôle important. Le terme *cloud hybride* fait référence à une combinaison de cloud public et de centres de données locaux pour créer un environnement informatique intégré qui s’étend sur les deux. Certaines organisations utilisent le cloud hybride comme chemin d’accès pour migrer tout leur centre de données vers le cloud au fur et à mesure. D’autres organisations utilisent des services cloud pour étendre leur infrastructure locale. 

Cet article décrit certaines considérations et les meilleures pratiques pour gérer des données dans une solution de cloud hybride,

## <a name="when-to-use-a-hybrid-solution"></a>Quand utiliser une solution hybride

Envisagez d’utiliser une solution hybride dans les scénarios suivants :

* En tant que stratégie de transition lors d’une migration à plus long terme vers une solution cloud entièrement native.
* Lorsque les réglementations ou les stratégies n’autorisent pas le déplacement de données ou de charges de travail spécifiques vers le cloud.
* Pour la récupération d’urgence et la tolérance de panne, en répliquant des données et des services entre des environnements locaux et cloud.
* Pour réduire la latence entre votre centre de données sur site et les emplacements distants, en hébergeant la partie de votre architecture dans Azure.

## <a name="challenges"></a>Défis

* Création d’un environnement cohérent en termes de sécurité, de gestion et de développement et évitement de la duplication du travail.

* Création d’une connexion des données fiable, à faible latence et sécurisée entre vos environnements locaux et cloud.

* Réplication de vos données ainsi que modification d’applications et d’outils pour utiliser les bons magasins de données au sein de chaque environnement.

* Sécurisation et chiffrement des données hébergées dans le cloud, mais consultés localement, ou vice versa.

## <a name="on-premises-data-stores"></a>Magasins de données locales

Les magasins de données incluent des bases de données et des fichiers. Il peut y avoir plusieurs raisons de les conserver localement. Il peut y avoir des réglementations ou des stratégies qui n’autorisent pas le déplacement de données ou de charges de travail spécifiques vers le cloud. Des préoccupations quant à la souveraineté des données, à la confidentialité ou aux problèmes de sécurité peuvent favoriser le positionnement local. Pendant une migration, vous pouvez souhaiter conserver localement des données d’une application qui n’a pas encore été migrée.

Les considérations relatives au placement des données d’application dans un cloud public sont notamment les suivantes :

* **Coût** : Le coût du stockage dans Azure peut être nettement plus faible que le coût de maintenance du stockage avec des caractéristiques semblables dans un centre de données local. Comme de nombreuses sociétés ont investi dans des réseaux SAN haut de gamme, elles ne bénéficieront peut-être pas de tous les avantages en matière de coût avant que le matériel existant ne soit obsolète.
* **Mise à l’échelle élastique**. La planification et la gestion de la croissance de la capacité des données dans un environnement local peut constituer un défi, en particulier lorsque la croissance des données est difficilement prédictible. Ces applications peuvent tirer parti de la capacité à la demande et du stockage pratiquement illimité disponibles dans le cloud. Cette considération est moins pertinente pour les applications qui se composent de jeux de données de taille relativement statique.
* **Récupération d’urgence**. Les données stockées dans Azure peuvent être répliquées automatiquement dans une région Azure et entre des régions géographiques. Dans les environnements hybrides, ces mêmes technologies peuvent être utilisées pour effectuer une réplication entre des magasins de données locaux et basés sur le cloud.

## <a name="extending-data-stores-to-the-cloud"></a>Extension des magasins de données dans le cloud

Plusieurs options permettent d’étendre les magasins de données locaux au cloud. Une option consiste à avoir des réplicas locaux et cloud. Cela peut aider à atteindre un niveau élevé de tolérance de pannes, mais peut nécessiter d’apporter des modifications à des applications pour se connecter au magasin de données approprié en cas de basculement.

Une autre option consiste à déplacer une partie des données vers le stockage cloud, tout en conservant localement les plus récentes ou les plus souvent consultées. Cette méthode peut fournir une option plus rentable pour le stockage à long terme, ainsi améliorer les temps de réponse d’accès aux données en réduisant votre jeu de données opérationnelles.

Une troisième option consiste à conserver localement toutes les données, mais à utiliser le cloud computing pour héberger des applications. Pour ce faire, vous hébergez votre application dans le cloud et vous la connectez à votre magasin de données local via une connexion sécurisée. 

## <a name="azure-stack"></a>Azure Stack

Pour une solution de cloud hybride complète, envisagez d’utiliser [Microsoft Azure Stack](/azure/azure-stack/). Azure Stack est une plateforme cloud hybride qui vous permet de fournir des services Azure depuis votre centre de données. Cela permet de maintenir la cohérence entre le magasin de données local et Azure, en utilisant des outils identiques sans qu’aucune modification de code ne soit nécessaire. 

Voici quelques cas d’usage pour Azure et Azure Stack :

* **Solutions edge et déconnectées**. Répondez aux besoins de latence et de connectivité en traitant les données localement dans Azure Stack, puis en les agrégeant dans Azure pour une analytique plus poussée, avec une logique d’application commune. 
* **Applications cloud conformes à diverses réglementations**. Développez et déployez des applications dans Azure, en profitant de la flexibilité de déployer les mêmes applications localement sur Azure Stack pour répondre aux exigences des réglementations et des stratégies.
* **Modèle d’application cloud en local**. Utilisez Azure pour mettre à jour et étendre des applications existantes ou en créer. Utilisez un processus DevOps cohérent sur Azure dans le cloud et sur Azure Stack en local.

## <a name="sql-server-data-stores"></a>Magasins de données SQL Server

Si vous exécutez SQL Server en local, vous pouvez utiliser le service de stockage d’objets Microsoft Azure Blob pour la sauvegarde et la restauration. Pour plus d’informations, voir [SQL Server Backup and Restore with Microsoft Azure Blob Storage Service](/sql/relational-databases/backup-restore/sql-server-backup-and-restore-with-microsoft-azure-blob-storage-service)(en anglais). Cette fonctionnalité vous offre un stockage hors site illimité ainsi que la possibilité de partager les mêmes sauvegardes entre SQL Server s’exécutant localement et SQL Server s’exécutant sur une machine virtuelle dans Azure. 

[Azure SQL Database](/azure/sql-database/) est une base de données relationnelle en tant que service. Comme Azure SQL Database utilise le moteur Microsoft SQL Server, les applications peuvent accéder aux données de la même façon avec les deux technologies. Azure SQL Database peut également être combinée avec SQL Server de plusieurs manières utiles. Par exemple, la fonctionnalité [SQL Server Stretch Database](/sql/sql-server/stretch-database/stretch-database) permet l’accès à une application qui ressemble à une seule table dans une base de données SQL Server alors que certaines lignes ou toutes les lignes de cette table peuvent être stockées dans la base de données Azure SQL. Cette technologie déplace automatiquement dans le cloud les données qui n’ont pas été consultées pendant une période définie. Les applications qui lisent ces données ne se rendent pas compte que des données ont été déplacées vers le cloud.

Conserver des magasins de données localement et dans le cloud peut être difficile lorsque vous souhaitez que les données restent synchronisées. Vous pouvez y remédier avec [SQL Data Sync](/azure/sql-database/sql-database-sync-data), un service conçu sur la base de données Azure SQL qui vous permet de synchroniser les données choisies de manière bidirectionnelle sur plusieurs bases de données Azure SQL et instances SQL Server. Même si la synchronisation des données facilite le maintien à jour de vos données dans ces divers magasins de données, elle ne doit pas être utilisée pour la récupération d’urgence, ni pour la migration de la base de données SQL Server locale vers la base de données Azure SQL.

Pour la récupération d’urgence et la continuité de votre activité, vous pouvez utiliser des [groupes de disponibilité AlwaysOn](/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server) afin de répliquer des données sur deux ou plusieurs instances de SQL Server, dont certaines peuvent être en cours d’exécution sur des machines virtuelles Azure situées dans une autre région géographique.

## <a name="network-shares-and-file-based-data-stores"></a>Partages réseau et magasins de données basés sur des fichiers

Dans une architecture de cloud hybride, il est courant qu’une organisation conserve les fichiers plus récents localement et archive les fichiers plus anciens vers le cloud. Cette procédure est parfois appelée hiérarchisation des fichiers, l’accès transparent aux deux groupes de fichiers étant assuré, qu’ils soient locaux ou hébergés dans le cloud. Cette approche permet de minimiser l’utilisation de la bande passante réseau et les temps d’accès aux fichiers les plus récents, qui sont susceptibles d’être consultés le plus fréquemment. En même temps, vous bénéficiez des avantages du stockage sur le cloud des données archivées. 

Les organisations peuvent également souhaiter déplacer entièrement leurs partages réseau dans le cloud. Il serait, par exemple, souhaitable que les applications qui y ont accès se trouvent également dans le cloud. Cette procédure peut être effectuée à l’aide d’outils d’[orchestration des données](../technology-choices/pipeline-orchestration-data-movement.md).


[Azure StorSimple](/azure/storsimple/) propose la solution de stockage intégrée la plus complète pour gérer les tâches de stockage entre vos appareils locaux et le stockage cloud Azure. StorSimple est une solution SAN efficace, rentable et facile à gérer qui élimine la plupart des problèmes et des frais liés à la protection des données et du stockage d’entreprise. Elle utilise l’appareil propriétaire de la série StorSimple 8000, s’intègre à des services cloud et fournit un ensemble d’outils de gestion intégrés.

[Les fichiers Azure](/azure/storage/files/storage-files-introduction) sont une autre méthode d’utilisation des partages réseau locaux en même temps que le stockage de fichiers basés sur le cloud. Les fichiers Azure proposent des partages de fichiers entièrement gérés auxquels vous pouvez accéder avec le protocole [Server Message Block](https://msdn.microsoft.com/library/windows/desktop/aa365233.aspx?f=255&MSPPError=-2147217396) (SMB) (parfois appelé CIFS). Vous pouvez monter des fichiers Azure en tant que partage de fichiers sur votre ordinateur local, ou les utiliser avec les applications existantes qui ont accès à des fichiers locaux ou de partage réseau.

Pour synchroniser les partages de fichiers dans des fichiers Azure avec vos serveurs Windows locaux, utilisez [Azure File Sync](/azure/storage/files/storage-sync-files-planning). Un des avantages majeurs d’Azure File Sync est sa capacité à hiérarchiser des fichiers entre votre serveur de fichiers local et les fichiers Azure. Cela vous permet de ne garder localement que les fichiers les plus récents ou les plus récemment consultés. 

Pour plus d’informations, consultez [Décision d’utiliser le stockage Blob Azure, les fichiers Azure ou les disques Azure,](/azure/storage/common/storage-decide-blobs-files-disks).

## <a name="hybrid-networking"></a>Mise en réseau hybride

Cet article est axé sur les solutions de données hybrides, mais prend également en considération la manière d’étendre votre réseau local à Azure. Pour plus d’informations sur cet aspect des solutions hybrides, consultez les rubriques suivantes :

- [Choisir une solution pour connecter un réseau local à Azure](../../reference-architectures/hybrid-networking/considerations.md)
- [Hybrid network reference architectures](../../reference-architectures/hybrid-networking/index.md)(en anglais)

