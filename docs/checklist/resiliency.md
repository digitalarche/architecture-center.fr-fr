---
title: "Liste de vérification de résilience"
description: "Liste de vérification fournissant des indications relatives aux problématiques de résilience durant la conception."
author: petertaylor9999
ms.date: 01/10/2018
ms.custom: resiliency, checklist
ms.openlocfilehash: ca4bf77c9348f6c656348d9cd61d3a1241d69ba8
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/08/2018
---
# <a name="resiliency-checklist"></a>Liste de vérification de résilience

La résilience est la capacité d’un système à récupérer après des défaillances et à continuer de fonctionner ; elle est l’un des [piliers de la qualité logicielle](../guide/pillars.md). La conception de votre application pour la résilience nécessite de planifier et de prévenir les différents modes d’échec susceptibles de se produire. Utilisez cette liste de vérification pour passer en revue l’architecture de votre application en termes de résilience. Consultez également la [liste de vérification de la résilience pour des services Azure spécifiques](./resiliency-per-service.md).

## <a name="requirements"></a>Configuration requise

**Définissez les exigences de disponibilité de votre client.** Votre client aura des exigences de disponibilité spécifiques pour les composants de votre application, ce qui aura une incidence sur sa conception. Obtenez l’accord de votre client concernant les objectifs de disponibilité de chaque élément de votre application. Sinon, votre conception risque de ne pas répondre à ses attentes. Pour plus d’informations, consultez [Définissez vos exigences en matière de résilience](../resiliency/index.md#defining-your-resiliency-requirements).

## <a name="application-design"></a>Conception des applications

**Effectuez une analyse du mode d’échec pour votre application.** L’analyse du mode d’échec est un processus qui permet d’intégrer la résilience dans une application au début de la phase de conception. Pour plus d’informations, consultez [Analyse du mode d’échec][fma]. Les objectifs d’une analyse du mode d’échec incluent :  

* Identifier les types d’échecs qu’une application peut rencontrer.
* Déterminer les effets et l’impact potentiels de chaque type d’échec sur l’application.
* Identifier les stratégies de récupération.
  

**Déployez plusieurs instances des services.** Si votre application repose sur une seule instance unique d’un service, cela crée un point de défaillance unique. L’approvisionnement de plusieurs instances améliore à la fois la résilience et l’extensibilité. Pour [Azure App Service](/azure/app-service/app-service-value-prop-what-is/), sélectionnez un [plan App Service](/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview/) qui offre plusieurs instances. Pour les Azure Cloud Services, configurez chacun de vos rôles de manière à utiliser [plusieurs instances](/azure/cloud-services/cloud-services-choose-me/#scaling-and-management). Pour [Machines virtuelles Azure](/azure/virtual-machines/virtual-machines-windows-about/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json), assurez-vous que votre architecture de machine virtuelle inclut plusieurs machines virtuelles et que chaque machine virtuelle est incluse dans un [groupe à haute disponibilité][availability-sets].   

**Utilisez la mise à l’échelle automatique pour répondre aux augmentations de charge.** Si votre application n’est pas configurée pour augmenter automatiquement la taille des instances quand la charge augmente, les services de votre application risquent d’échouer s’ils sont saturés de requêtes utilisateur. Pour plus d’informations, consultez les articles suivants :

* Informations générales : [Liste de contrôle de l’extensibilité](./scalability.md)
* Azure App Service : [Mise à l’échelle manuelle ou automatique du nombre d’instances][app-service-autoscale]
* Services cloud : [Mise à l’échelle automatique d’un service cloud][cloud-service-autoscale]
* Machines virtuelles : [Mise à l’échelle automatique et groupes de machines virtuelles identiques][vmss-autoscale]

**Utilisez l’équilibrage de charge pour distribuer les requêtes.** L’équilibrage de charge distribue les requêtes de votre application à des instances de service intègres en supprimant les instances non intègres de la rotation. Si votre service utilise Azure App Service ou Azure Cloud Services, sa charge est déjà équilibrée pour vous. Cependant, si votre application utilise Machines virtuelles Azure, vous devez approvisionner un équilibreur de charge. Pour plus d’informations, consultez la [vue d’ensemble d’Azure Load Balancer](/azure/load-balancer/load-balancer-overview/).

**Configurez les passerelles Azure Application Gateway de manière à utiliser plusieurs instances.** Selon les exigences de votre application, une passerelle [Azure Application Gateway](/azure/application-gateway/application-gateway-introduction/) peut être mieux adaptée pour distribuer les requêtes aux services de votre application. Cependant, les instances uniques du service Application Gateway ne sont pas garanties par un contrat de niveau de service (SLA). Par conséquent, il se peut que votre application échoue en cas d’échec de l’instance Application Gateway. Approvisionnez plusieurs instances Application Gateway moyennes ou grandes pour garantir la disponibilité du service selon les conditions du [SLA](https://azure.microsoft.com/support/legal/sla/application-gateway/v1_0/).

**Utilisez des groupes à haute disponibilité pour chaque couche Application.** Le placement des instances dans un [groupe à haute disponibilité][availability-sets] permet de bénéficier d’un [SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines/) supérieur. 

**Envisagez de déployer votre application dans plusieurs régions.** Si votre application est déployée dans une seule région, si la région entière devient indisponible (éventualité hautement improbable), votre application sera elle aussi indisponible. Il se peut que cette situation soit inacceptable au regard des conditions du SLA de votre application. Si c’est le cas, envisagez de déployer votre application et ses services dans plusieurs régions. Un déploiement dans plusieurs régions peut s’appuyer sur un modèle actif (répartissant les requêtes entre plusieurs instances actives) ou sur un modèle actif/passif (conservant une instance active en réserve en cas d’échec de l’instance principale). Nous vous recommandons de déployer plusieurs instances des services de votre application sur des paires régionales. Pour plus d’informations, consultez [Continuité des activités et récupération d’urgence (BCDR) : régions jumelées d’Azure](/azure/best-practices-availability-paired-regions).

**Utilisez Azure Traffic Manager pour acheminer le trafic de votre application vers des régions différentes.**  [Azure Traffic Manager][ traffic-manager] effectue un équilibrage de charge au niveau du DNS et achemine le trafic vers des régions différentes en fonction de la méthode de [routage du trafic][ traffic-manager-routing] que vous spécifiez et de l’intégrité des points de terminaison de votre application. Sans Traffic Manager, vous êtes limité à une seule région pour votre déploiement, ce qui limite la mise à l’échelle, augmente la latence pour certains utilisateurs et entraîne l’arrêt de l’application en cas d’interruption de service à l’échelle de la région.

**Configurez et testez les sondes d’intégrité pour vos équilibreurs de charge et vos gestionnaires de trafic.** Assurez-vous que votre logique d’intégrité vérifie les composants critiques du système et répond correctement aux sondes d’intégrité.

* Les sondes d’intégrité pour [Azure Traffic Manager][traffic-manager] et [Azure Load Balancer][load-balancer] remplissent une fonction spécifique. Pour Traffic Manager, la sonde d’intégrité détermine si un basculement vers une autre région est nécessaire. Pour un équilibreur de charge, elle détermine s’il faut supprimer une machine virtuelle de la rotation.      
* Pour une sonde de Traffic Manager, votre point de terminaison d’intégrité doit vérifier les dépendances critiques qui sont déployées dans la même région et dont l’échec devrait déclencher un basculement vers une autre région.  
* Pour un équilibreur de charge, le point de terminaison d’intégrité doit établir un rapport de l’intégrité de la machine virtuelle. N’incluez pas d’autres niveaux ou de services externes. Autrement, si un échec se produit en dehors de la machine virtuelle, l’équilibreur de charge supprimera la machine virtuelle de la rotation.
* Pour savoir comment implémenter la surveillance de l’intégrité dans votre application, consultez [Modèle Surveillance de point de terminaison](https://msdn.microsoft.com/library/dn589789.aspx).

**Surveillez les services tiers.** Si votre application présente des dépendances vis-à-vis de services tiers, identifiez où et comment ces services tiers peuvent échouer et les effets que ces échecs auront sur votre application. Un service tiers peut ne pas inclure de surveillance et de diagnostics. Par conséquent, il est important que vous journalisiez vos appels de ces services et que vous les mettiez en corrélation avec la journalisation de l’intégrité et des diagnostics de votre application à l’aide d’un identificateur unique. Pour plus d’informations sur les pratiques éprouvées en matière de surveillance et de diagnostics, consultez [Guide de surveillance et de diagnostics][monitoring-and-diagnostics-guidance].

**Assurez-vous que les services tiers que vous consommez incluent un contrat de niveau de service (SLA).** Si votre application dépend d’un service tiers, mais que ce service tiers n’offre aucune garantie de disponibilité sous la forme d’un contrat SLA, la disponibilité de votre application ne peut pas non plus être garantie. Le niveau de votre contrat SLA repose sur le composant le moins disponible de votre application.

**Implémentez des modèles de résilience pour les opérations distantes si nécessaire.** Si votre application dépend de la communication entre des services distants, suivez les modèles de conception pour traiter les échecs temporaires, par exemple le [modèle Nouvelle tentative][retry-pattern] et le [modèle Disjoncteur][circuit-breaker]. Pour plus d’informations, consultez [Stratégies de résilience](../resiliency/index.md#resiliency-strategies).

**Implémentez des opérations asynchrones dès que possible.** Les opérations synchrones peuvent monopoliser les ressources et de bloquer les autres opérations pendant que l’appelant attend la fin du processus. Concevez chaque partie de votre application de manière à permettre les opérations asynchrones dès que possible. Pour plus d’informations sur l’implémentation de la programmation asynchrone en C#, consultez [Programmation asynchrone avec async et await][asynchronous-c-sharp].

## <a name="data-management"></a>Gestion des données

**Étudiez les méthodes de réplication pour les sources de données de votre application.** Vos données d’application seront stockées dans des sources de données différentes et présenteront des exigences de disponibilité distinctes. Évaluer les méthodes de réplication pour chaque type de stockage de données dans Azure, y compris la [réplication Stockage Azure](/azure/storage/storage-redundancy/) et [géoréplication active de SQL Database](/azure/sql-database/sql-database-geo-replication-overview/), pour vous assurer que les exigences de votre application en matière de données sont satisfaites.

**Assurez-vous qu’aucun compte d’utilisateur n’a accès à la fois aux données de production et aux données de sauvegarde.** Si un même compte d’utilisateur est autorisé à écrire dans les sources de production et de sauvegarde, vos sauvegardes de données risquent d’être compromises. En effet, un utilisateur malveillant pourrait supprimer volontairement toutes vos données, tandis qu’un utilisateur normal pourrait les supprimer accidentellement. Concevez votre application de manière à limiter les autorisations de chaque compte d’utilisateur afin que seuls les utilisateurs qui ont besoin d’un accès en écriture disposent d’un tel accès, et que celui-ci ne s’applique qu’aux données de production ou de sauvegarde.

**Documentez le processus de basculement et de restauration automatique de votre source de données et testez-le.** Si votre source de données connaît une erreur irrécupérable, un opérateur humain devra suivre un ensemble d’instructions documentées pour effectuer un basculement vers une nouvelle source de données. Si les étapes documentées comportent des erreurs, l’opérateur ne sera pas en mesure de les suivre et de basculer la ressource correctement. Testez régulièrement la procédure pour vérifier qu’un opérateur qui la suit est en mesure de basculer et de restaurer la source de données avec succès.

**Validez vos sauvegardes de données.** Vérifiez régulièrement que vos données de sauvegarde sont celles attendues en exécutant un script pour valider l’intégrité des données, le schéma et les requêtes. Cela ne sert à rien de disposer d’une sauvegarde si celle-ci ne permet pas de restaurer les sources de données. Consignez et signalez toute incohérence pour que le service de sauvegarde puisse être réparé.

**Envisagez d’utiliser un type de compte de stockage géoredondant.** Les données stockées dans un compte Stockage Azure sont toujours répliquées localement. Cependant, plusieurs stratégies de réplication sont proposées lors du provisionnement d’un compte de stockage. Sélectionnez [Stockage géo-redondant avec accès en lecture (RA-GRS)](/azure/storage/storage-redundancy/#read-access-geo-redundant-storage) pour que vos données d’application soient protégées dans l’éventualité peu probable où une région entière deviendrait indisponible.

> [!NOTE]
> Pour les machines virtuelles, ne vous appuyez pas sur la réplication RA-GRS pour restaurer les disques de machine virtuelle (fichiers VHD). Utilisez le service [Sauvegarde Azure][azure-backup] à la place.   
>
>

## <a name="security"></a>Sécurité

**Implémentez la protection de niveau application contre les attaques par déni de service distribué (DDoS).** Les services Azure sont protégés contre les attaques DDoS au niveau de la couche réseau. Cependant, Azure ne peut pas assurer une protection contre les attaques de la couche Application, car il est difficile de distinguer les requêtes utilisateur réelles des requêtes utilisateur malveillantes. Pour plus d’informations sur la protection contre les attaques DDoS ciblant la couche Application, consultez la section Protecting against DDoS (Protection contre les attaques DDoS) du document [Microsoft Azure Network Security](http://download.microsoft.com/download/C/A/3/CA3FC5C0-ECE0-4F87-BF4B-D74064A00846/AzureNetworkSecurity_v3_Feb2015.pdf) (Sécurité réseau de Microsoft Azure) (téléchargement au format PDF).

**Implémentez le principe des privilèges minimum pour l’accès aux ressources de l’application.** L’accès par défaut aux ressources de l’application doit être le plus restrictif possible. Accordez des autorisations de niveau supérieur sur approbation. Si vous octroyez par défaut un accès trop permissif aux ressources de votre application, quelqu’un pourrait supprimer des ressources volontairement ou accidentellement. Azure fournit un [contrôle d’accès en fonction du rôle](/azure/active-directory/role-based-access-built-in-roles/) pour gérer les privilèges utilisateur, mais il est important de vérifier les autorisations avec des privilèges minimum pour les autres ressources qui présentent leur propre système d’autorisations, telles que SQL Server.

## <a name="testing"></a>Test

**Effectuez des tests de basculement et de restauration automatique pour votre application.** Si vous n’avez pas entièrement testé le basculement et la restauration automatique, vous n’avez aucune garantie que les services dépendants de votre application redeviendront actifs de manière synchronisée au cours de la récupération d’urgence. Assurez-vous que les services dépendants de votre application sont basculés et restaurés dans l’ordre correct.

**Effectuez des tests d’injection d’erreurs pour votre application.** Votre application peut échouer pour de nombreuses raisons : expiration de certificat, épuisement des ressources système d’une machine virtuelle, défaillances de stockage, etc. Testez votre application dans un environnement aussi proche que possible de l’environnement de production, en simulant ou en déclenchant des échecs réels. Par exemple, supprimez des certificats, consommez artificiellement les ressources système ou supprimez une source de stockage. Vérifiez la capacité de votre application à récupérer des erreurs de tout type, qu’elles surviennent de façon isolée ou simultanée. Assurez-vous que les erreurs ne se propagent pas ou ne se produisent pas de manière successive dans votre système.

**Exécutez des tests en production en utilisant à la fois des données utilisateur de synthèse et des données utilisateur réelles.** Étant donné que les environnements de test et de production sont rarement identiques, il est important d’utiliser un déploiement Blue/Green ou de contrôle de validité et de tester votre application en production. Vous pouvez ainsi tester votre application en production avec une charge réelle et vous assurer qu’elle fonctionnera comme prévu quand elle sera entièrement déployée.

## <a name="deployment"></a>Déploiement

**Documentez le processus de mise en production de votre application.** Sans documentation détaillée du processus de mise en production, un opérateur risque de déployer une mise à jour incorrecte ou de mal configurer les paramètres de votre application. Définissez et documentez clairement votre processus de mise en production et assurez-vous que toute l’équipe chargée des opérations a accès à cette documentation. 

**Automatisez le processus de déploiement de votre application.** Si votre personnel chargé des opérations est obligé de déployer manuellement votre application, des erreurs humaines risquent de faire échouer le déploiement. 

**Concevez votre processus de mise en production de manière à optimiser la disponibilité de l’application.** Si votre processus de mise en production nécessite le passage hors connexion des services pendant le déploiement, votre application sera indisponible jusqu’à ce qu’ils soient à nouveau en ligne. Utilisez la technique de déploiement [Blue/Green](http://martinfowler.com/bliki/BlueGreenDeployment.html) ou [de contrôle de validité](http://martinfowler.com/bliki/CanaryRelease.html) pour déployer votre application en production. Ces deux techniques impliquent le déploiement de votre code de mise en production parallèlement au code de production pour que les utilisateurs du code de mise en production puissent être redirigés vers le code de production en cas de défaillance.

**Consignez et évaluez les déploiements de votre application.** Si vous utilisez des techniques de déploiement intermédiaire telles qu’un déploiement Blue/Green ou de contrôle de validité, plusieurs versions de votre application seront exécutées en production. Si un problème survient, il est essentiel de déterminer quelle version de votre application en est à l’origine. Implémentez une stratégie de journalisation fiable pour capturer le plus d’informations possible concernant la version.

**Mettez en place un plan de restauration pour le déploiement.** Le déploiement de votre application peut échouer et provoquer l’indisponibilité de votre application. Concevez un processus de restauration pour revenir à la dernière version valide connue et réduire le temps d’arrêt. 

## <a name="operations"></a>Opérations

**Implémentez les meilleures pratiques en matière de surveillance et d’alerte dans votre application.** En l’absence de systèmes de surveillance, de diagnostic et d’alerte appropriés, vous n’aurez aucun moyen de détecter les défaillances dans votre application et de les signaler à un opérateur à des fins de résolution. Pour plus d’informations, consultez [Guide de surveillance et de diagnostic][monitoring-and-diagnostics-guidance].

**Mesurez les statistiques d’appels distants et mettez ces informations à disposition de l’équipe responsable de l’application.**  Si vous n’effectuez pas de suivi et de rapport des statistiques d’appels distants en temps réel, et que vous n’offrez pas un moyen simple de consulter ces informations, l’équipe ne bénéficiera pas d’une vue instantanée de l’intégrité de votre application. Par ailleurs, si vous mesurez seulement la durée moyenne des appels distants, vous ne disposerez pas d’informations suffisantes pour déceler les problèmes affectant les services. Résumez les mesures liées aux appels distants, telles que la latence, le débit et les erreurs, dans les centiles 99 et 95. Effectuez une analyse statistique sur les mesures pour détecter les erreurs qui se produisent dans chaque centile.

**Suivez le nombre d’exceptions temporaires et de nouvelles tentatives sur une plage de temps appropriée.** Si vous ne suivez et ne surveillez pas les exceptions temporaires et les nouvelles tentatives au fil du temps, il se peut qu’une défaillance ou un problème soient dissimulés par la logique de nouvelle tentative de votre application. Autrement dit, si votre surveillance et votre journalisation montrent uniquement la réussite ou l’échec d’une opération, le fait que l’opération a dû être retentée plusieurs fois en raison d’exceptions sera dissimulé. Une augmentation des exceptions au fil du temps indique que le service rencontre un problème et risque d’échouer. Pour plus d’informations, consultez [Guide spécifique relatif au service de nouvelle tentative][retry-service-guidance].

**Implémentez un système d’avertissement anticipé qui alerte un opérateur.** Identifiez les indicateurs de performance clés de l’intégrité de votre application, tels que les exceptions temporaires et la latence des appels distants, et définissez des valeurs de seuil appropriées pour chacun d’eux. Envoyez une alerte à l’équipe chargée des opérations quand une valeur de seuil est atteinte. Définissez ces seuils à des niveaux qui identifient les problèmes avant qu’ils ne deviennent critiques et n’exigent la mise en œuvre d’une récupération.

**Assurez-vous que plusieurs personnes de l’équipe sont formées pour surveiller l’application et effectuer les étapes de récupération manuelle.** Si vous avez un seul opérateur dans l’équipe qui peut surveiller l’application et déclencher la procédure de récupération, celui-ci constitue un point de défaillance unique. Formez plusieurs personnes sur la détection et la récupération et assurez-vous qu’au moins une d’entre elles est disponible à tout moment.

**Assurez-vous que votre application ne se heurte pas aux [limites d’abonnement Azure](/azure/azure-subscription-service-limits/).** Les abonnements Azure présentent des limites pour certains types de ressources, par exemple le nombre de groupes de ressources, le nombre de cœurs et le nombre de comptes de stockage.  Si les exigences de votre application dépassent les limites d’abonnement Azure, créez un autre abonnement Azure et approvisionnez des ressources suffisantes pour celui-ci.

**Assurez-vous que votre application ne se heurte pas aux [limites par service](/azure/azure-subscription-service-limits/).** Les services Azure individuels présentent des limites en termes de consommation, telles que des limites sur le stockage, le débit, le nombre de connexions, les requêtes par seconde et d’autres mesures. Si votre application tente d’utiliser des ressources au-delà de ces limites, elle échouera. Cela donnera lieu à une limitation du service et potentiellement à un temps d’arrêt pour les utilisateurs touchés. Selon le service spécifique et les exigences de votre application, vous pouvez souvent éviter ces limites en montant en puissance (par exemple, en choisissant un autre niveau tarifaire) ou en montant en charge (en ajoutant de nouvelles instances).  

**Concevez votre application de telle sorte que ses exigences de stockage soient inférieures ou égales aux objectifs d’extensibilité et de performance du stockage Azure.** Le stockage Azure étant conçu pour fonctionner selon des objectifs d’extensibilité et de performance prédéfinis, vous devez concevoir votre application de manière à ce que son utilisation du stockage soit inférieure ou égale à ces objectifs. Si vous dépassez ces objectifs, le stockage de votre application sera limité. Pour résoudre ce problème, approvisionnez des comptes de stockage supplémentaires. Si vous vous heurtez à la limite des comptes de stockage, approvisionnez des abonnements Azure supplémentaires, puis approvisionnez-y des comptes de stockage supplémentaires. Pour plus d'informations, consultez [Objectifs d'extensibilité et de performances d'Azure Storage](/azure/storage/storage-scalability-targets/).

**Sélectionnez la taille de machine virtuelle appropriée pour votre application.** Mesurez l’utilisation réelle de l’UC, de la mémoire, du disque et des E/S de vos machines virtuelles en production et vérifiez que la taille de machine virtuelle que vous avez choisie est suffisante. Si ce n’est pas le cas, il se peut que votre application rencontre des problèmes de capacité quand les machines virtuelles sont proches de leurs limites. Les tailles de machine virtuelle sont décrites en détail dans [Tailles des machines virtuelles dans Azure](/azure/virtual-machines/virtual-machines-windows-sizes/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

**Déterminez si la charge de travail de votre application est stable ou fluctue au fil du temps.** Si votre charge de travail fluctue au fil du temps, utilisez les groupes de machines virtuelles identiques Azure pour mettre automatiquement à l’échelle le nombre d’instances de machines virtuelles. Autrement, vous devrez augmenter ou diminuer manuellement le nombre de machines virtuelles. Pour plus d’informations, consultez [Présentation des groupes de machines virtuelles identiques Azure](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview/).

**Sélectionnez le niveau de service approprié pour Azure SQL Database.** Si votre application utilise Azure SQL Database, assurez-vous que vous avez choisi le niveau de service approprié. Si vous sélectionnez un niveau qui ne permet pas de gérer les exigences de votre application en matière d’unités de transaction de base de données (DTU), votre utilisation de données sera limitée. Pour plus d’informations sur la sélection du plan de service approprié, consultez [Options et performances de SQL Database : comprendre ce qui est disponible dans chaque niveau de service](/azure/sql-database/sql-database-service-tiers/).

**Créez un processus pour l’interaction avec le support Azure.** Si le processus de contact du [support Azure](https://azure.microsoft.com/support/plans/) n’est pas défini avant qu’il soit nécessaire de contacter ce dernier, le temps d’arrêt sera prolongé du fait de l’accès au processus de support pour la première fois. Incluez le processus pour contacter le support et faire remonter les problèmes dans le cadre de la résilience de votre application dès le début.

**Assurez-vous que votre application n’utilise pas plus que le nombre maximal de comptes de stockage par abonnement.** Azure autorise un maximum de 200 comptes de stockage par abonnement. Si votre application requiert plus de comptes de stockage qu’il n’y en a actuellement dans votre abonnement, vous devez créer un nouvel abonnement et y créer des comptes de stockage supplémentaires. Pour plus d’informations, consultez [Abonnement Azure et limites, quotas et contraintes de service](/azure/azure-subscription-service-limits/#storage-limits).

**Assurez-vous que votre application ne dépasse pas les objectifs d’extensibilité pour les disques de machine virtuelle.** Une machine virtuelle IaaS Azure prend en charge l’attachement d’un nombre de disques de données qui dépend de plusieurs facteurs, notamment la taille de machine virtuelle et le type de compte de stockage. Si votre application dépasse les objectifs d’extensibilité pour les disques de machine virtuelle, approvisionnez des comptes de stockage supplémentaires et créez-y les disques de machine virtuelle. Pour plus d’informations, consultez [Objectifs de performance et d’extensibilité de Stockage Azure](/azure/storage/storage-scalability-targets/#scalability-targets-for-virtual-machine-disks).

## <a name="telemetry"></a>Télémétrie

**Consignez des données de télémétrie pendant l’exécution de l’application dans l’environnement de production.** Enregistrez des informations de télémétrie fiables pendant l’exécution de l’application dans l’environnement de production ou vous ne disposerez pas de suffisamment d’informations pour diagnostiquer la cause des problèmes pendant son utilisation. Pour plus d’informations, consultez [Surveillance et diagnostics][monitoring-and-diagnostics-guidance].

**Implémentez la journalisation à l’aide d’un modèle asynchrone.** Si les opérations de journalisation sont synchrones, elles risquent de bloquer le code de votre application. Assurez-vous que vos opérations de journalisation sont implémentées en tant qu’opérations asynchrones.

**Mettez en corrélation les données de journalisation au-delà des limites des services.** Dans une application multiniveau classique, une requête utilisateur peut franchir les limites de plusieurs services. Par exemple, en général, une requête utilisateur provient de la couche Web, est transmise à la couche Métier et enfin conservée dans la couche Données. Dans des scénarios plus complexes, une requête utilisateur peut être distribuée à de nombreux services et magasins de données différents. Assurez-vous que votre système de journalisation met en corrélation les appels au-delà des limites des services pour que vous puissiez suivre la requête dans l’ensemble de votre application.

## <a name="azure-resources"></a>Ressources Azure

**Utilisez les modèles Azure Resource Manager pour approvisionner des ressources.** Les modèles Resource Manager permettent d’automatiser plus facilement les déploiements via PowerShell ou l’interface de ligne de commande Azure, ce qui donne lieu à un processus de déploiement plus fiable. Pour plus d’informations, consultez [Présentation d’Azure Resource Manager][resource-manager].

**Nommez les ressources de façon explicite.** Vous pourrez ainsi plus facilement localiser une ressource spécifique et comprendre son rôle. Pour plus d’informations, consultez [Conventions d’affectation de noms pour les ressources Azure](../best-practices/naming-conventions.md).

**Utilisez le contrôle d’accès en fonction du rôle (RBAC).** Utilisez le contrôle RBAC pour contrôler l’accès aux ressources Azure que vous déployez. Le contrôle RBAC vous permet d’assigner des rôles d’autorisation aux membres de votre équipe DevOps afin d’empêcher les suppressions ou les modifications accidentelles des ressources déployées. Pour plus d’informations, consultez [Prise en main de la gestion des accès dans le portail Azure](/azure/active-directory/role-based-access-control-what-is/).

**Utilisez les verrous de ressources pour les ressources critiques, telles que les machines virtuelles.** Les verrous de ressources empêchent un opérateur de supprimer accidentellement une ressource. Pour plus d’informations, consultez [Verrouiller des ressources avec Azure Resource Manager](/azure/azure-resource-manager/resource-group-lock-resources/).

**Choisissez des paires régionales.** En cas de déploiement dans deux régions, choisissez des régions de la même paire régionale. En cas de panne étendue, la récupération d’une région est hiérarchisée pour chaque paire. Certains services de stockage géoredondant fournissent une réplication automatique vers la région jumelée. Pour plus d’informations, consultez [Continuité des activités et récupération d’urgence (BCDR) : régions jumelées d’Azure](/azure/best-practices-availability-paired-regions).

**Organisez les groupes de ressources par fonction et cycle de vie.**  En règle générale, un groupe de ressources doit contenir des ressources qui partagent le même cycle de vie. Cela simplifie la gestion des déploiements, la suppression des déploiements de test et l’attribution des droits d’accès, réduisant ainsi le risque de suppression ou de modification accidentelle d’un déploiement de production. Créez des groupes de ressources distincts pour les environnements de production, de développement et de test. Dans un déploiement multirégion, placez les ressources pour chaque région dans des groupes de ressources distincts. Vous pourrez ainsi plus facilement redéployer une région sans affecter les autres régions.

## <a name="next-steps"></a>étapes suivantes

- [Liste de vérification de la résilience pour des services Azure spécifiques](./resiliency-per-service.md)
- [Analyse du mode d’échec](../resiliency/failure-mode-analysis.md)


<!-- links -->
[app-service-autoscale]: /azure/monitoring-and-diagnostics/insights-how-to-scale/
[asynchronous-c-sharp]: /dotnet/articles/csharp/async
[availability-sets]:/azure/virtual-machines/virtual-machines-windows-manage-availability/
[azure-backup]: https://azure.microsoft.com/documentation/services/backup/
[circuit-breaker]: ../patterns/circuit-breaker.md
[cloud-service-autoscale]: /azure/cloud-services/cloud-services-how-to-scale/
[fma]: ../resiliency/failure-mode-analysis.md
[load-balancer]: /azure/load-balancer/load-balancer-overview/
[monitoring-and-diagnostics-guidance]: ../best-practices/monitoring.md
[resource-manager]: /azure/azure-resource-manager/resource-group-overview/
[retry-pattern]: ../patterns/retry.md
[retry-service-guidance]: ../best-practices/retry-service-specific.md
[traffic-manager]: /azure/traffic-manager/traffic-manager-overview/
[traffic-manager-routing]: /azure/traffic-manager/traffic-manager-routing-methods/
[vmss-autoscale]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview/
