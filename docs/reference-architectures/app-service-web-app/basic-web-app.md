---
title: Application web de base
description: Architecture recommandée pour une application web basique exécutée dans Microsoft Azure.
author: MikeWasson
ms.date: 12/12/2017
cardTitle: Basic web application
ms.openlocfilehash: efd831b1f54fa0662bdfa9874318e7b314172215
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/06/2018
ms.locfileid: "30846401"
---
# <a name="basic-web-application"></a>Application web de base
[!INCLUDE [header](../../_includes/header.md)]

Cette architecture de référence présente un ensemble de pratiques éprouvées pour une application web utilisant [Azure App Service][app-service] et [Azure SQL Database][sql-db]. [**Déployez cette solution.**](#deploy-the-solution)

![[0]][0]

*Téléchargez un [fichier Visio][visio-download] de cette architecture.*

## <a name="architecture"></a>Architecture 

> [!NOTE]
> Cette architecture n’est pas centrée sur le développement d’applications et n’assume pas d’infrastructure d’application particulière. L’objectif est de comprendre comment différents services Azure se complètent.
>
>

Elle comporte les composants suivants :

* **Groupe de ressources**. Un [groupe de ressources](/azure/azure-resource-manager/resource-group-overview) est un conteneur logique pour les ressources Azure.

* **Application App Service**. [Azure App Service][app-service] est une plateforme entièrement gérée pour créer et déployer des applications cloud.     

* **Plan App Service**. Un [plan App Service][app-service-plans] fournit les machines virtuelles gérées qui hébergent votre application. Toutes les applications associées à un plan s’exécutent dans les mêmes instances de machine virtuelle.

* **Emplacements de déploiement**.  Un [emplacement de déploiement][deployment-slots] vous permet de planifier un déploiement et de l’échanger ensuite avec le déploiement de production. De cette façon, vous évitez le déploiement directement dans l’environnement de production. Consultez la section [Facilité de gestion](#manageability-considerations) pour obtenir des recommandations spécifiques.

* **Adresse IP**. L’application App Service a une adresse IP publique et un nom de domaine. Le nom de domaine est un sous-domaine de `azurewebsites.net`, tel que `contoso.azurewebsites.net`.  

* **Azure DNS**. [Azure DNS][azure-dns] est un service d’hébergement pour les domaines DNS qui offre une résolution de noms à l’aide de l’infrastructure Microsoft Azure. En hébergeant vos domaines dans Azure, vous pouvez gérer vos enregistrements DNS avec les mêmes informations d’identification, les mêmes API, les mêmes outils et la même facturation que vos autres services Azure. Pour utiliser un nom de domaine personnalisé, tel que `contoso.com`, créez des enregistrements DNS qui mappent le nom de domaine personnalisé sur l’adresse IP. Pour plus d’informations, consultez [Configurer un nom de domaine personnalisé dans Azure App Service][custom-domain-name].  

* **Base de données SQL Azure**. [SQL Database][sql-db] est une base de données relationnelle sous forme de service dans le cloud. SQL Database partage sa base de code avec le moteur de base de données Microsoft SQL Server. Selon les exigences de votre application, vous pouvez également utiliser [Azure Database pour MySQL](/azure/mysql) ou [Azure Database pour PostgreSQL](/azure/postgresql). Il s’agit de services de base de données entièrement gérés, basés sur des moteurs de base de données managées open source MySQL Server et Postgres, respectivement.

* **Serveur logique**. Dans Azure SQL Database, un serveur logique héberge vos bases de données. Vous pouvez créer plusieurs bases de données dans un serveur logique.

* **Stockage Azure**. Créez un compte de stockage Azure avec un conteneur d’objets blob pour stocker les journaux de diagnostic.

* **Azure Active Directory** (Azure AD). Utilisez Azure AD ou un autre fournisseur d’identité pour l’authentification.

## <a name="recommendations"></a>Recommandations

Vos exigences peuvent différer de celles de l’architecture décrite ici. Utilisez les recommandations de cette section comme point de départ.

### <a name="app-service-plan"></a>Plan App Service
Utilisez les niveaux Standard ou Premium, car ils prennent en charge l’augmentation de la taille des instances, la mise à l’échelle automatique et le protocole SSL. Chaque niveau prend en charge plusieurs *tailles d’instance* qui diffèrent par leur nombre de cœurs et la quantité de mémoire. Après avoir créé un plan, vous pouvez modifier le niveau ou la taille de l’instance. Pour plus d’informations sur les plans App Service, consultez [Tarification de App Service][app-service-plans-tiers].

Vous êtes facturé pour les instances dans le plan App Service, même si l’application est arrêtée. Veillez à supprimer les plans que vous n’utilisez pas (par exemple, les déploiements de test).

### <a name="sql-database"></a>Base de données SQL
Utilisez la [version V12] [ sql-db-v12] de SQL Database. SQL Database prend en charge les [niveaux de service][sql-db-service-tiers] Basic, Standard et Premium, avec plusieurs niveaux de performances au sein de chaque niveau, mesurés en [unités de transaction de base de données (DTU)][sql-dtu]. Planifiez la capacité et choisissez des niveaux de service et de performances correspondant à vos besoins.

### <a name="region"></a>Région
Configurez le plan App Service et SQL Database dans la même région pour réduire la latence du réseau. Généralement, sélectionnez la région la plus proche de vos utilisateurs.

Le groupe de ressources est également doté d’une région, indiquant où les métadonnées de déploiement sont stockées. Placez le groupe de ressources et ses ressources dans la même région. Cela peut améliorer la disponibilité lors du déploiement. 

## <a name="scalability-considerations"></a>Considérations relatives à l’extensibilité

Le principal avantage de Azure App Service est la possibilité de mettre à l’échelle votre application en fonction de la charge. Voici quelques considérations à prendre en compte pendant la planification de la mise à l’échelle de votre application.

### <a name="scaling-the-app-service-app"></a>Mise à l’échelle de l’application App Service

Il existe deux façons de mettre à l’échelle une application App Service :

* *Monter en puissance*, signifiant la modification de la taille de l’instance. La taille de l’instance détermine la quantité de mémoire et de stockage et le nombre de cœurs pour chaque instance de machine virtuelle. Vous pouvez monter en puissance manuellement en modifiant la taille d’instance ou le niveau du plan.  

* *Monter en charge*, signifiant l’ajout d’instances pour traiter une augmentation de charge. Chaque niveau de tarification possède un nombre d’instances maximal. 

  Vous pouvez monter en charge manuellement en modifiant le nombre d’instances, ou en utilisant la [mise à l’échelle automatique ][web-app-autoscale] pour que Azure ajoute ou supprime automatiquement des instances en fonctions des mesures de performances et/ou de planification. Chaque opération de mise à l’échelle se produit rapidement&mdash;généralement en quelques secondes. 

  Pour activer la mise à l’échelle automatique, créez un *profil* de mise à l’échelle qui définit le nombre minimal et maximal d’instances. Les profils peuvent être planifiés. Par exemple, vous pouvez créer des profils différents pour les jours de la semaine et le week-end. Un profil peut contenir des règles pour l’ajout ou la suppression des instances. (Exemple : ajouter deux instances si l’utilisation du processeur est supérieure à 70 % pendant 5 minutes.)
  
Recommandations pour la mise à l’échelle d’une application web :

* Évitez la mise à l’échelle autant que possible car elle peut déclencher le redémarrage de l’application. À la place, sélectionnez un niveau et une taille répondant à vos besoins de performances sous une charge normale, puis augmentez la taille des instances pour gérer les changements dans le volume de trafic.    
* Activez la mise à l’échelle automatique. Si votre application a une charge de travail régulière et prévisible, créez des profils pour planifier à l’avance le nombre d’instances. Si la charge de travail n’est pas prévisible, faites une mise à l’échelle basée sur des règles pour réagir aux variations de charge. Vous pouvez combiner ces deux approches.
* L’utilisation de l’UC est généralement une bonne métrique pour les règles de mise à l’échelle. Toutefois, vous devriez tester la charge de votre application, identifier les goulots d’étranglement potentiels et baser vos règles de mise à l’échelle sur ces données.  
* Les règles de mise à l’échelle incluent une période de *refroidissement*, correspondant à l’intervalle entre la fin d’une action de mise à l’échelle et le démarrage d’une nouvelle. La période de refroidissement permet de stabiliser le système avant une nouvelle mise à l’échelle. Définissez une période de refroidissement plus courte pour l’ajout d’instances et une période de refroidissement plus longue pour la suppression d’instances. Par exemple, choisissez une période de 5 minutes pour ajouter une instance, mais une période de 60 minutes pour supprimer une instance. Il est préférable d’ajouter de nouvelles instances rapidement en cas de charge importante afin de pouvoir gérer le trafic supplémentaire, puis de réduire progressivement le nombre d’instances.

### <a name="scaling-sql-database"></a>Mise à l’échelle de SQL Database
Si vous avez besoin d’un niveau de service ou de performances supérieur pour SQL Database, vous pouvez monter en puissance les bases de données individuelles sans interruption de l’application. Pour plus d’informations, consultez [Options et performances de SQL Database : comprendre ce qui est disponible dans chaque niveau de service][sql-db-scale].

## <a name="availability-considerations"></a>Considérations relatives à la disponibilité
Au moment de l’écriture, le contrat de niveau de service (SLA) pour App Service est de 99,95 % et le contrat SLA pour SQL Database est de 99,99 % pour les niveaux Basic, Standard et Premium. 

> [!NOTE]
> Le contrat SLA App Service s’applique aux instances uniques et multiples.  
>
>

### <a name="backups"></a>Sauvegardes
En cas de perte de données, SQL Database fournit la restauration dans le temps et la restauration géographique. Ces fonctionnalités sont disponibles dans tous les niveaux et sont automatiquement activées. Vous n’avez pas besoin de planifier ou de gérer les sauvegardes. 

- Utilisez la restauration dans le temps pour [récupérer d’une erreur humaine][sql-human-error] en restaurant la base de données à une date/heure antérieure. 
- Utilisez la restauration géographique pour [récupérer d’une interruption de service][sql-outage-recovery] en restaurant une base de données à partir d’une sauvegarde géo-redondante. 

Pour en savoir plus, consultez l’article [Continuité des activités cloud et récupération d’urgence d’une base de données avec SQL Database][sql-backup].

App Service possède une fonctionnalité [Sauvegarde et restauration][web-app-backup] pour les fichiers de votre application. Toutefois, sachez que les fichiers sauvegardés incluent les paramètres de l’application en texte brut et qu’ils peuvent contenir des secrets, comme les chaînes de connexion. Évitez d’utiliser la fonctionnalité de sauvegarde de App Service pour sauvegarder vos bases de données SQL, car elle exporte la base de données dans un fichier .bacpac SQL, en consommant des [DTU][sql-dtu]. Utilisez plutôt la restauration dans le temps de SQL Database, décrite ci-dessus.

## <a name="manageability-considerations"></a>Considérations relatives à la facilité de gestion
Créez des groupes de ressources distincts pour les environnements de production, de développement et de test. Cela simplifie la gestion des déploiements, la suppression des déploiements de test et l’attribution des droits d’accès.

Lorsque vous attribuez des ressources aux groupes de ressources, considérez les points suivants :

* Cycle de vie. D’une façon générale, placez les ressources avec le même cycle de vie dans un même groupe de ressources.
* Accès. Vous pouvez utiliser le [contrôle d’accès en fonction du rôle][rbac] (RBAC) pour appliquer des stratégies d’accès aux ressources dans un groupe.
* Facturation. Vous pouvez afficher les coûts cumulés pour le groupe de ressources.  

Pour plus d’informations, consultez [Présentation d’Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview).

### <a name="deployment"></a>Déploiement
Le déploiement implique deux étapes :

1. Attribution des ressources Azure. Nous vous recommandons d’utiliser les [modèles de Azure Resource Manager][arm-template] pour cette étape. Les modèles facilitent l’automatisation des déploiements via PowerShell ou l’interface de ligne de commande (CLI) de Azure.
2. Déploiement de l’application (code, binaires et fichiers de contenu). Vous disposez de plusieurs options, notamment le déploiement depuis un référentiel Git local, à l’aide de Visual Studio ou bien un déploiement continu à partir du contrôle de code source dans le cloud. Voir [Déploiement de votre application dans Azure App Service][deploy].  

Une application App Service dispose toujours d’un emplacement de déploiement nommé `production`, représentant le site de production en direct. Nous vous recommandons de créer un emplacement de préproduction pour le déploiement des mises à jour. Voici plusieurs avantages de l’utilisation d’un emplacement de préproduction :

* Vous pouvez vérifier la réussite du déploiement, avant de le permuter en production.
* Le déploiement vers un emplacement de préproduction garantit la préparation de toutes les instances avant de le permuter en production. Un grand nombre d’application possèdent un temps de préparation et de démarrage à froid important.

Nous vous recommandons également la création d’un troisième emplacement pour stocker le dernier déploiement correct. Après avoir échangé les emplacements de préproduction et de production, déplacez le déploiement de production précédent (maintenant à l’emplacement de préproduction) vers l’emplacement du dernier déploiement correct. Ainsi, vous pouvez rapidement revenir à la dernière version correcte si vous détectez un problème plus tard.

![[1]][1]

Si vous restaurez une version précédente, assurez-vous que toutes les modifications de schéma de la base de données sont rétrocompatibles.

N’utilisez pas d’emplacements de votre déploiement de production pour des tests, car toutes les applications appartenant au même plan App Service partagent les mêmes instances de machine virtuelle. Par exemple, des tests de charge peuvent dégrader le site de production réelle. Utilisez plutôt des plans App Service distincts pour la production et le test. En plaçant les déploiements de test dans un plan séparé, vous les isolez de la version de production.

### <a name="configuration"></a>Configuration
Stockez les paramètres de configuration en tant que [paramètres de l’application][app-settings]. Définissez les paramètres d’application dans vos modèles Resource Manager ou à l’aide de PowerShell. Lors du runtime, les paramètres d’application sont disponibles pour l’application en tant que variables d’environnement.

Ne recherchez jamais des mots de passe, clés d’accès ou chaînes de connexion dans un contrôle de code source. Au lieu de cela, passez-les en tant que paramètres dans un script de déploiement qui stocke ces valeurs comme paramètres de l’application.

Lorsque vous échangez un emplacement de déploiement, les paramètres d’application sont échangés par défaut. Si vous avez besoin de paramètres différents pour la production et la préproduction, vous pouvez créer des paramètres d’application spécifiques à un emplacement et qui ne seront pas transférés.

### <a name="diagnostics-and-monitoring"></a>Diagnostics et surveillance
Activez la [journalisation des diagnostics][diagnostic-logs], y compris le journal des applications et le journal du serveur web. Configurez la journalisation pour utiliser le stockage d’objets Blob. Pour des raisons de performances, créez un compte de stockage distinct pour les journaux de diagnostic. N’utilisez pas le même compte de stockage pour les journaux et les données d’application. Pour plus d’informations sur la journalisation, consultez le [Guide de surveillance et de diagnostic][monitoring-guidance].

Utiliser un service tel que [New Relic][new-relic] ou [Application Insights][app-insights] pour surveiller les performances et le comportement de l’application sous charge. Soyez conscient des [limites du débit de données][app-insights-data-rate] de Application Insights.

Effectuez un test de charge, à l’aide d’un outil tel que [Visual Studio Team Services][vsts]. Pour obtenir une vue d’ensemble de l’analyse des performances dans les applications cloud, consultez le [Manuel d’analyse de performances][perf-analysis].

Conseils de dépannage de votre application :

* Utilisez le [panneau de dépannage][troubleshoot-blade] dans le portail Azure pour trouver des solutions aux problèmes courants.
* Activez la [diffusion de journaux][web-app-log-stream] pour consulter les informations de journalisation en temps quasi-réel.
* Le [tableau de bord Kudu][kudu] dispose de plusieurs outils pour surveiller et déboguer votre application. Pour plus d'informations, consultez [Outils en ligne de Sites Web Azure que vous devez connaître][kudu] (billet de blog). Vous pouvez atteindre le tableau de bord Kudu à partir du portail Azure. Ouvrez le panneau pour votre application et cliquez sur <strong>Outils</strong>, puis cliquez sur <strong>Kudu</strong>.
* Si vous utilisez Visual Studio, consultez l’article [Dépanner une application web dans Azure App Service à l’aide de Visual Studio][troubleshoot-web-app] pour obtenir des conseils de débogage et de dépannage.

## <a name="security-considerations"></a>Considérations relatives à la sécurité
Cette section répertorie les considérations relatives à la sécurité qui sont spécifiques aux services Azure décrits dans cet article. Il ne s’agit pas d’une liste complète de bonnes pratiques relatives à la sécurité. Pour certaines considérations supplémentaires relatives à la sécurité, consultez [Sécuriser une application dans Azure App Service][app-service-security].

### <a name="sql-database-auditing"></a>Audit de base de données SQL
L’audit peut vous aider à respecter une conformité réglementaire et à découvrir des discordances et irrégularités susceptibles d’indiquer des problèmes pour l’entreprise ou des violations de la sécurité. Voir [Prise en main de l’audit de base de données SQL][sql-audit].

### <a name="deployment-slots"></a>Emplacements de déploiement
Chaque emplacement de déploiement possède une adresse IP publique. Sécuriser les emplacements de production à l’aide de la [connexion de Azure Active Directory][aad-auth] afin que seuls les membres de vos équipes de DevOps et de développement puissent atteindre ces points de terminaison.

### <a name="logging"></a>Journalisation
Les journaux ne devrez jamais enregistrer les mots de passe des utilisateurs ou d’autres informations qui peuvent être utilisées pour valider l’usurpation d’identité. Retirez ces détails des données avant de les stocker.   

### <a name="ssl"></a>SSL
Une application App Service comprend un point de terminaison SSL sur un sous-domaine de `azurewebsites.net` sans coût supplémentaire. Le point de terminaison SSL comprend un certificat générique pour le domaine `*.azurewebsites.net`. Si vous utilisez un nom de domaine personnalisé, vous devez fournir un certificat correspondant au domaine personnalisé. L’approche la plus simple est d’acheter un certificat directement via le portail Azure. Vous pouvez également importer des certificats à partir d’autres autorités de certification. Pour plus d’informations, consultez [Acheter et configurer un certificat SSL pour votre service Azure App Service][ssl-cert].

Comme meilleure pratique de sécurité, votre application doit appliquer le protocole HTTPS en redirigeant les requêtes HTTP. Vous pouvez implémenter cela à l’intérieur de votre application ou utiliser une règle de réécriture d’URL, comme décrit dans [Activer le protocole HTTPS pour une application dans Azure App Service][ssl-redirect].

### <a name="authentication"></a>Authentification
Nous vous recommandons l’authentification via un fournisseur d’identité (IDP), tels que Azure AD, Facebook, Google, ou Twitter. Utilisez OAuth 2 ou OpenID Connect (OIDC) pour le flux d’authentification. Azure AD fournit des fonctionnalités permettant de gérer les utilisateurs et les groupes, créer des rôles d’application, intégrer vos identités locales et consommer des services principaux tels que Office 365 et Skype pour entreprises.

Évitez que l’application gère directement les informations d’identification et des connexions, car cela crée une surface d’attaque potentielle.  Au minimum, vous avez besoin d’une confirmation par e-mail, de la récupération de mot de passe et de l’authentification multifacteur ; la validation de la force du mot de passe ; et le stockage en toute sécurité des hachages de mot de passe. Les grands fournisseurs d’identité gèrent tous ces éléments pour vous, et ils surveillent et améliorent constamment leurs pratiques de sécurité.

Envisagez l’utilisation de [l’authentification App Service][app-service-auth] pour implémenter le flux d’authentification OAuth/OIDC. Avantages de l’authentification App Service :

* Simple à configurer.
* Aucun code n’est requis pour les scénarios d’authentification simples.
* Prend en charge la délégation d’autorisation à l’aide des jetons d’accès OAuth afin de consommer des ressources pour le compte de l’utilisateur.
* Fournit un cache de jeton intégré.

Certaines limitations de l’authentification App Service :  

* Options de personnalisation limitées.
* L’autorisation déléguée est limitée à une ressource principale par ouverture de session.
* Si vous utilisez plusieurs fournisseurs d’identité, il n’existe aucun mécanisme intégré pour la découverte du domaine d’accueil.
* Pour les scénarios d’architecture mutualisée, l’application doit implémenter la logique pour valider l’émetteur du jeton.

## <a name="deploy-the-solution"></a>Déployer la solution
Un exemple de modèle Resource Manager pour cette architecture est [disponible sur GitHub][paas-basic-arm-template].

Pour déployer le modèle à l’aide de PowerShell, exécutez les commandes suivantes :

```
New-AzureRmResourceGroup -Name <resource-group-name> -Location "West US"

$parameters = @{"appName"="<app-name>";"environment"="dev";"locationShort"="uw";"databaseName"="app-db";"administratorLogin"="<admin>";"administratorLoginPassword"="<password>"}

New-AzureRmResourceGroupDeployment -Name <deployment-name> -ResourceGroupName <resource-group-name> -TemplateFile .\PaaS-Basic.json -TemplateParameterObject  $parameters
```

Pour plus d’informations, consultez [Déployer des ressources à l’aide de modèles Azure Resource Manager][deploy-arm-template].

<!-- links -->

[aad-auth]: /azure/app-service-mobile/app-service-mobile-how-to-configure-active-directory-authentication
[app-insights]: /azure/application-insights/app-insights-overview
[app-insights-data-rate]: /azure/application-insights/app-insights-pricing
[app-service]: https://azure.microsoft.com/documentation/services/app-service/
[app-service-auth]: /azure/app-service-api/app-service-api-authentication
[app-service-plans]: /azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview
[app-service-plans-tiers]: https://azure.microsoft.com/pricing/details/app-service/
[app-service-security]: /azure/app-service-web/web-sites-security
[app-settings]: /azure/app-service-web/web-sites-configure
[arm-template]: /azure/azure-resource-manager/resource-group-overview#resource-groups
[azure-dns]: /azure/dns/dns-overview
[custom-domain-name]: /azure/app-service-web/web-sites-custom-domain-name
[deploy]: /azure/app-service-web/web-sites-deploy
[deploy-arm-template]: /azure/resource-group-template-deploy
[deployment-slots]: /azure/app-service-web/web-sites-staged-publishing
[diagnostic-logs]: /azure/app-service-web/web-sites-enable-diagnostic-log
[kudu]: https://azure.microsoft.com/blog/windows-azure-websites-online-tools-you-should-know-about/
[monitoring-guidance]: ../../best-practices/monitoring.md
[new-relic]: http://newrelic.com/
[paas-basic-arm-template]: https://github.com/mspnp/reference-architectures/tree/master/managed-web-app/basic-web-app/Paas-Basic/Templates
[perf-analysis]: https://github.com/mspnp/performance-optimization/blob/master/Performance-Analysis-Primer.md
[rbac]: /azure/active-directory/role-based-access-control-what-is
[resource-group]: /azure/azure-resource-manager/resource-group-overview
[sla]: https://azure.microsoft.com/support/legal/sla/
[sql-audit]: /azure/sql-database/sql-database-auditing-get-started
[sql-backup]: /azure/sql-database/sql-database-business-continuity
[sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[sql-db-overview]: /azure/sql-database/sql-database-technical-overview
[sql-db-scale]: /azure/sql-database/sql-database-service-tiers#scaling-up-or-scaling-down-a-single-database
[sql-db-service-tiers]: /azure/sql-database/sql-database-service-tiers
[sql-db-v12]: /azure/sql-database/sql-database-features
[sql-dtu]: /azure/sql-database/sql-database-service-tiers
[sql-human-error]: /azure/sql-database/sql-database-business-continuity#recover-a-database-after-a-user-or-application-error
[sql-outage-recovery]: /azure/sql-database/sql-database-business-continuity#recover-a-database-to-another-region-from-an-azure-regional-data-center-outage
[ssl-redirect]: /azure/app-service-web/web-sites-configure-ssl-certificate#bkmk_enforce
[sql-resource-limits]: /azure/sql-database/sql-database-resource-limits
[ssl-cert]: /azure/app-service-web/web-sites-purchase-ssl-web-site
[troubleshoot-blade]: https://azure.microsoft.com/updates/self-service-troubleshooting-for-app-service-web-apps-customers/
[troubleshoot-web-app]: /azure/app-service-web/web-sites-dotnet-troubleshoot-visual-studio
[visio-download]: https://archcenter.blob.core.windows.net/cdn/app-service-reference-architectures.vsdx
[vsts]: https://www.visualstudio.com/features/vso-cloud-load-testing-vs.aspx
[web-app-autoscale]: /azure/app-service-web/web-sites-scale
[web-app-backup]: /azure/app-service-web/web-sites-backup
[web-app-log-stream]: /azure/app-service-web/web-sites-enable-diagnostic-log#streamlogs
[0]: ./images/basic-web-app.png "Architecture d’une application web Azure de base"
[1]: ./images/paas-basic-web-app-staging-slots.png "Permutation des emplacements de déploiements de production et de préproduction"
