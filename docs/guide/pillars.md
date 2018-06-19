---
title: Piliers de la qualité logicielle
description: 'Décrit les cinq piliers de la qualité des logiciels : l’extensibilité, la disponibilité, la résilience, la gestion et la sécurité.'
author: MikeWasson
ms.openlocfilehash: 117706046ca1a9b7f3203a99737347809d0c323f
ms.sourcegitcommit: 85334ab0ccb072dac80de78aa82bcfa0f0044d3f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/11/2018
ms.locfileid: "35252785"
---
# <a name="pillars-of-software-quality"></a>Piliers de la qualité logicielle 

Une application cloud réussie se concentrera sur ces cinq piliers de la qualité des logiciels : l’extensibilité, la disponibilité, la résilience, la gestion et la sécurité.

| Pilier | Description |
|--------|-------------|
| Extensibilité | Capacité d’un système à traiter une charge accrue. |
| Disponibilité | Durée pendant laquelle le système est fonctionnel et opérationnel. |
| Résilience | Capacité d’un système à récupérer après des défaillances et à continuer de fonctionner. |
| gestion | Processus d’opérations assurant l’exécution d’un système en production. |
| Sécurité | Protection des applications et des données contre les menaces. |

## <a name="scalability"></a>Extensibilité

Capacité d’un système à traiter une charge accrue. La mise à l’échelle d’une application peut s’effectuer de deux manières : La mise à l’échelle verticale (augmentation de la taille des *instances*) signifie augmenter la capacité d’une ressource, par exemple à l’aide d’une machine virtuelle de plus grande taille. La mise à l’échelle horizontale (montée en *charge*) consiste à ajouter de nouvelles instances d’une ressource, telles que des machines virtuelles ou des réplicas de base de données. 

La mise à l’échelle horizontale présente des avantages significatifs par rapport à la mise à l’échelle verticale :

- Véritable mise à l’échelle du cloud. Les applications peuvent être conçues pour s’exécuter sur des centaines, voire des milliers de nœuds, pour atteindre des échelles impossibles sur un seul nœud.
- L’échelle horizontale est élastique. Vous pouvez ajouter plus d’instances si la charge augmente, ou en supprimer pendant les périodes plus calme.
- La montée en charge peut être déclenchée automatiquement de manière planifiée ou en réponse aux modifications de la charge. 
- Une montée en charge peut être moins onéreuse qu’une augmentation de la taille des instances. L’exécution de plusieurs machines virtuelles de petite taille peut être moins coûteuse qu’une seule grande machine virtuelle. 
- La mise à l’échelle horizontale peut également améliorer la résilience par l’ajout de redondance. Si une instance tombe en panne, l’application continue de s’exécuter.

Un des avantages de la mise à l’échelle verticale consiste à pouvoir le faire sans modifier l’application. Mais à un moment donné, vous aurez atteint la limite et ne pourrez plus augmenter la taille des instances. À ce stade, vous devrez procéder à une mise à niveau horizontale. 

Celle-ci doit être conçue dans le système. Par exemple, vous pouvez monter en charge les machines virtuelles en les plaçant derrière un équilibreur de charge. Toutefois, chaque machine virtuelle du pool doit être en mesure de gérer n’importe quelle requête client. Par conséquent, l’application doit être sans état ou stocker l’état à l’extérieur (par exemple, dans un cache distribué). Les services Paas managés intègrent souvent les mises à l’échelle horizontale et automatique. La facilité de mise à l’échelle de ces services est un atout majeur lié à l’utilisation des services PaaS.

Cependant, le simple ajout d’instances ne se traduit pas par la mise à l’échelle d’une application. Il se peut que cela déplace le goulet d’étranglement vers un autre emplacement. Par exemple, la mise à l’échelle d’un serveur web frontal pour gérer plus de demandes client, peut déclencher des conflits de verrouillage dans la base de données. Vous devrez alors envisager des mesures supplémentaires, telles que l’accès concurrentiel optimiste ou le partitionnement des données pour augmenter le débit de la base de données.

Effectuez toujours les tests de charge et de performances pour rechercher ces goulets d’étranglement potentiels. Les parties avec état d’un système, telles que les bases de données, sont les causes les plus courantes des goulets d’étranglement et requièrent une conception attentive pour une mise à l’échelle horizontale. La résolution d’un goulet d’étranglement peut en révéler d’autres ailleurs.

Utilisez la [liste de vérification de l’évolutivité][scalability-checklist] pour revoir votre conception du point de vue de l’évolutivité.

### <a name="scalability-guidance"></a>Guide de l’évolutivité

- [Modèles de conception pour l’évolutivité et les performances][scalability-patterns]
- Meilleures pratiques : [Mise à l’échelle automatique][autoscale], [Travaux en arrière-plan][background-jobs], [Mise en cache][caching], [CDN][cdn], [Partitionnement des données][data-partitioning]

## <a name="availability"></a>Disponibilité

La disponibilité est la durée pendant laquelle le système est fonctionnel et opérationnel. Elle est généralement mesurée en pourcentage de la durée de fonctionnement. Les erreurs d’application, les problèmes d’infrastructure et la charge système peuvent réduire la disponibilité. 

Une application de cloud doit avoir un objectif de niveau de service (SLO) qui définit clairement la disponibilité attendue, et la manière dont elle est mesurée. Lors de la définition de la disponibilité, examinez le chemin critique. Le serveur web frontal peut être en mesure de traiter les demandes client, mais si chaque transaction échoue en raison d’une connexion impossible à la base de données, l’application ne sera pas disponible pour les utilisateurs. 

La disponibilité est souvent décrite en termes de « 9 » &mdash;, par exemple, « quatre 9 » signifie une durée de fonctionnement de 99,99 %. Le tableau suivant présente les temps d’arrêt cumulés potentiels à différents niveaux de disponibilité.

| Durée de fonctionnement en % | Temps d’arrêt par semaine | Temps d’arrêt par mois | Temps d’arrêt par an |
|----------|-------------------|--------------------|-------------------|
| 99 % | 1,68 heure | 7,2 heures | 3,65 jours |
| 99,9 % | 10 minutes | 43,2 minutes | 8,76 heures |
| 99,95 % | 5 minutes | 21,6 minutes | 4,38 heures |
| 99,99 % | 1 minute | 4,32 minutes | 52,56 minutes |
| 99, 999 % | 6 secondes | 26 secondes | 5,26 minutes |

Notez qu’une durée de fonctionnement de 99 % peut signifier quasiment une panne de service de 2 heures par semaine. Pour de nombreuses applications, notamment les applications accessibles aux consommateurs, cet objet de niveau de service n’est pas acceptable. En revanche, cinq 9 (99,999 %) signifie pas plus de 5 minutes de temps d’arrêt par *an*. Il est suffisamment difficile de détecter une panne, sans parler de résoudre le problème rapidement. Pour obtenir une disponibilité très élevée (99,99 % ou plus encore), vous ne pouvez pas compter sur une intervention manuelle pour récupérer après des défaillances. L’application doit être dotée de capacités d’auto-diagnostic et de réparation spontanée. Et c’est à ce moment que la résilience devient cruciale.

Dans Azure, les contrats de niveau de service (SLA) décrivent les engagements de Microsoft en matière de durée de fonctionnement et de connectivité. Si le contrat SLA pour un service particulier est de 99,95 %, cela signifie que le service doit être disponible 99,95 % du temps.

Les applications dépendent souvent de plusieurs services. En règle générale, la probabilité d’un temps d’arrêt de chacun de ces services est indépendante. Par exemple, supposons que votre application dépende de deux services, chacun ayant un contrat de niveau de service de 99,9 %. Le contrat SLA composite pour les deux services est de 99,9 % &times; 99,9 % &asymp; 99,8 % ou légèrement inférieur à chaque service en lui-même. 

Utilisez la [liste de vérification de disponibilité][availability-checklist] pour revoir votre conception du point de vue de la disponibilité.

### <a name="availability-guidance"></a>Guide de disponibilité

- [Modèles de conception pour la disponibilité][availability-patterns]
- Meilleures pratiques : [Mise à l’échelle automatique][autoscale], [Travaux en arrière-plan][background-jobs]

## <a name="resiliency"></a>Résilience

La résilience est la capacité d’un système à récupérer après des défaillances et à continuer de fonctionner. L’objectif de la résilience est que l’application retrouve un état entièrement fonctionnel suite à une défaillance. La résilience est étroitement liée à la disponibilité.

Les développeurs d’applications traditionnelles se sont concentrés sur la réduction du temps moyen entre les défaillances (MTBF). Ils se sont efforcés d’empêcher les défaillances du système. Le cloud computing ne nécessite pas le même état d’esprit, en raison de plusieurs facteurs :

- Les systèmes distribués sont complexes et une défaillance peut potentiellement se répercuter dans tout le système.
- Les coûts pour les environnements de cloud restent faibles via l’utilisation d’un matériel standard. Par conséquent, des défaillances matérielles occasionnelles sont possibles. 
- Les applications dépendent souvent de services externes pouvant être temporairement indisponibles ou limitent les utilisateurs intensifs. 
- Les utilisateurs actuels s’attendent à ce qu’une application soit disponible 24 heures sur 24 et 7 jours sur 7 sans jamais se déconnecter.

Tous ces facteurs impliquent que les applications de cloud soient conçues pour subir des défaillances occasionnelles et les résoudre. Azure propose de nombreuses fonctionnalités de résilience déjà intégrées à la plate-forme. Par exemple, 

- Stockage Microsoft Azure, Microsoft Azure SQL Database et Azure Cosmos DB intègrent tous la réplication des données, à la fois dans une région et entre les régions.
- Les disques managés Azure sont automatiquement placés dans diverses unités d’échelle de stockage pour limiter les effets liés aux défaillances matérielles.
- Les machines virtuelles d’un groupe à haute disponibilité sont réparties sur plusieurs domaines d’erreur. Un domaine d’erreur est un groupe de machines virtuelles qui partagent une source d’alimentation et un commutateur réseau communs. Cette répartition limite l’impact des défaillances du matériel physique, des pannes de réseau ou des coupures de courant.

Ceci dit, vous devez toujours faire en sorte que votre application soit résiliente. Les stratégies de résilience peuvent être appliquées à tous les niveaux de l’architecture. Certaines solutions d’atténuation sont plus tactiques par nature &mdash;, par exemple, une nouvelle tentative d’appel à distance après une défaillance réseau temporaire. Les autres solutions d’atténuation sont plus stratégiques, comme le basculement de l’ensemble de l’application vers une région secondaire. Les solutions d’atténuation tactiques peuvent faire une grande différence. S’il est rare pour une région entière de subir une interruption, des problèmes temporaires, tels que la congestion du réseau sont plus courantes &mdash;, c’est pourquoi il convient de les cibler en premier. Le fait de disposer de l’analyse et des diagnostics adaptés est également important à la fois pour détecter les défaillances lorsqu’elles surviennent et pour rechercher les causes racines.

Lorsque vous concevez une application qui doit être résiliente, vous devez comprendre vos besoins en matière de disponibilité. Quel temps d’arrêt maximal est acceptable ? Cela dépend en partie du coût. Combien les temps d’arrêt vont-ils coûter à votre entreprise ? Combien devriez-vous investir pour rendre l’application hautement disponible ?

Utilisez la [liste de vérification de la résilience][resiliency-checklist] pour revoir votre conception du point de vue de la résilience.

### <a name="resiliency-guidance"></a>Aide relative à la résilience

- [Conception d’applications résilientes pour Azure][resiliency]
- [Modèles de conception pour la résilience][resiliency-patterns]
- Meilleures pratiques : [Gestion des erreurs transitoires][transient-fault-handling], [Guide relatif aux nouvelles tentatives pour des services spécifiques][retry-service-specific]

## <a name="management-and-devops"></a>Gestion et DevOps

Ce pilier couvre les processus des opérations qui maintiennent l’exécution d’une application en production.

Les déploiements doivent être fiables et prévisibles. Ils doivent être automatisés pour réduire le risque d’erreur humaine. Il doit s’agir d’un processus rapide et routinier pour ne pas ralentir la publication de nouvelles fonctionnalités ou de résolution de bogues. Il est tout aussi important d’être en mesure d’effectuer une restauration ou une restauration par progression rapidement en cas de problème de mise à jour.

La surveillance et les diagnostics sont cruciaux. Les applications cloud s’exécutent dans un centre de données distant où vous ne possédez pas de contrôle total sur l’infrastructure ou, dans certains cas, sur le système d’exploitation. Dans une grande application, il n’est pas pratique de se connecter à des machines virtuelles pour résoudre un problème ou passer en revue les fichiers journaux. Avec les services PaaS, il se peut même qu’il n’y ait pas de machine virtuelle dédiée à laquelle se connecter. L’analyse et les diagnostics donnent des informations sur le système, qui vous permettent de déterminer quand et où les défaillances surviennent. Tous les systèmes doivent pouvoir être observés. Utilisez un schéma de journalisation commun et cohérent qui vous permet de mettre en corrélation les événements entre les systèmes.

Le processus d’analyse et de diagnostic comporte plusieurs phases distinctes :

- Instrumentation. Générer des données brutes, à partir de journaux des applications, de journaux de serveurs web, de diagnostics intégrés à la plateforme Azure et d’autres sources.
- Collecte et stockage. Consolider les données de manière centralisée.
- Analyse et diagnostic. Résoudre les problèmes et examiner l’intégrité globale.
- Visualisation et alertes. Utilisation de la télémétrie pour détecter les tendances ou alerter l’équipe chargée des opérations.

Utilisez la [liste de vérification DevOps] [ devops-checklist] pour revoir votre conception du point de vue de la gestion et de DevOps.

### <a name="management-and-devops-guidance"></a>Guide relatif à la gestion et à DevOps

- [Modèles de conception pour la gestion et la surveillance][management-patterns]
- Meilleures pratiques : [Analyse et diagnostics][monitoring]

## <a name="security"></a>Sécurité

Vous devez penser à la sécurité tout au long du cycle de vie d’une application, de la conception et de l’implémentation au déploiement et aux opérations. La plateforme Azure protège contre diverses menaces, telles que les intrusions sur le réseau et les attaques DDoS. Vous devez tout de même intégrer la sécurité à votre application et vos processus DevOps.

Voici certaines zones de sécurité générales à prendre en compte. 

### <a name="identity-management"></a>Gestion des identités

Envisagez d’utiliser Azure Active Directory (Azure AD) pour authentifier et autoriser les utilisateurs. Azure AD est un service de gestion des identités et des accès entièrement géré. Vous pouvez l’utiliser pour créer des domaines qui existent seulement dans Azure, ou pour l’intégrer à vos identités Active Directory locales. Azure AD s’intègre également à Office 365, Dynamics CRM Online et à de nombreuses applications SaaS tierces. Pour les applications accessibles aux consommateurs, Azure Active Directory B2C permet aux utilisateurs de s’authentifier à l’aide de leurs comptes de réseaux sociaux existants (tel que Facebook, Google ou LinkedIn), ou de créer un nouveau compte d’utilisateur géré par Azure AD.

Si vous souhaitez intégrer un environnement Active Directory local à un réseau Azure, plusieurs approches sont possibles, selon vos besoins. Pour plus d’informations, consultez nos architectures de référence relatives à la [Gestion des identités][identity-ref-arch].

### <a name="protecting-your-infrastructure"></a>Protection de votre infrastructure 

Contrôlez l’accès aux ressources Azure que vous déployez. Chaque abonnement Azure dispose d’une [relation d’approbation][ad-subscriptions] avec un locataire Azure AD. Utilisez le [contrôle d’accès en fonction du rôle][rbac] (RBAC) pour accorder des autorisations appropriées aux ressources Azure aux utilisateurs de votre organisation. Accordez l’accès en assignant un rôle RBAC aux utilisateurs ou aux groupes selon une certaine étendue. L’étendue peut être appliquée à un abonnement, à un groupe de ressources ou à une ressource unique. [Auditez] [ resource-manager-auditing] toutes les modifications apportées à l’infrastructure. 

### <a name="application-security"></a>Sécurité des applications

En général, les meilleures pratiques de sécurité pour le développement d’applications s’appliquent toujours dans le cloud. Celles-ci incluent des éléments tels que l’utilisation de SSL partout, la protection contre les attaques CSRF et XSS, la prévention des attaques par injection de code SQL et ainsi de suite. 

Les applications cloud utilisent souvent des services gérés ayant des clés d’accès. Ne recherchez jamais ces éléments dans le contrôle de code source. Envisagez de stocker les secrets d’application dans Azure Key Vault.

### <a name="data-sovereignty-and-encryption"></a>Chiffrement et souveraineté des données

Assurez-vous que vos données restent dans la zone géopolitique appropriée lors de l’utilisation de la haute disponibilité d’Azure. Le stockage géo-répliqué d’Azure utilise le concept de [région associée] [ paired-region] dans la même région géopolitique. 

Utilisez le coffre de clés pour protéger les clés et les secrets de chiffrement. Le coffre de clés vous permet de chiffrer les clés et les secrets à l’aide de clés protégées par des modules de sécurité matériels (HSM). De nombreux services de stockage Azure et de base de données prennent en charge le chiffrement des données au repos, y compris [Stockage Microsoft Azure][storage-encryption], [Microsoft Azure SQL Database][sql-db-encryption], [Microsoft Azure SQL Data Warehouse][data-warehouse-encryption], et [Azure Cosmos DB][cosmosdb-encryption].

### <a name="security-resources"></a>Ressources de sécurité

- [Azure Security Center][security-center] fournit des fonctions intégrées de surveillance de la sécurité et de gestion des stratégies sur vos abonnements Azure. 
- [Documentation sur la sécurité Azure][security-documentation]
- [Centre de gestion de la confidentialité Microsoft][trust-center]



<!-- links -->

[dr-guidance]: ../resiliency/disaster-recovery-azure-applications.md
[identity-ref-arch]: ../reference-architectures/identity/index.md
[resiliency]: ../resiliency/index.md

[ad-subscriptions]: /azure/active-directory/active-directory-how-subscriptions-associated-directory
[data-warehouse-encryption]: /azure/data-lake-store/data-lake-store-security-overview#data-protection
[cosmosdb-encryption]: /azure/cosmos-db/database-security
[rbac]: /azure/active-directory/role-based-access-control-what-is
[paired-region]: /azure/best-practices-availability-paired-regions
[resource-manager-auditing]: /azure/azure-resource-manager/resource-group-audit
[security-blog]: https://azure.microsoft.com/blog/tag/security/
[security-center]: https://azure.microsoft.com/services/security-center/
[security-documentation]: /azure/security/
[sql-db-encryption]: /azure/sql-database/sql-database-always-encrypted-azure-key-vault
[storage-encryption]: /azure/storage/storage-service-encryption
[trust-center]: https://azure.microsoft.com/support/trust-center/
 

<!-- patterns -->
[availability-patterns]: ../patterns/category/availability.md
[management-patterns]: ../patterns/category/management-monitoring.md
[resiliency-patterns]: ../patterns/category/resiliency.md
[scalability-patterns]: ../patterns/category/performance-scalability.md


<!-- practices -->
[autoscale]: ../best-practices/auto-scaling.md
[background-jobs]: ../best-practices/background-jobs.md
[caching]: ../best-practices/caching.md
[cdn]: ../best-practices/cdn.md
[data-partitioning]: ../best-practices/data-partitioning.md
[monitoring]: ../best-practices/monitoring.md
[retry-service-specific]: ../best-practices/retry-service-specific.md
[transient-fault-handling]: ../best-practices/transient-faults.md


<!-- checklist -->
[availability-checklist]: ../checklist/availability.md
[devops-checklist]: ../checklist/dev-ops.md
[resiliency-checklist]: ../checklist/resiliency.md
[scalability-checklist]: ../checklist/scalability.md
