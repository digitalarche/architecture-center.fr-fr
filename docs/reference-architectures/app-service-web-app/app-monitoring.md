---
title: Supervision des applications web sur Azure
description: Surveillez une application web hébergée dans Azure App Service.
author: adamboeglin
ms.date: 12/12/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: azcat
ms.openlocfilehash: 34c21f4b5356dc0acbd5c2c85124300a6ed13c99
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640496"
---
# <a name="web-application-monitoring-on-azure"></a>Supervision des applications web sur Azure

Les offres de plateforme Azure en tant que service (PaaS) gèrent pour vous les ressources de calcul et impactent la manière dont vous surveillez les déploiements. Azure inclut plusieurs services de supervision, qui ont chacun un rôle spécifique. Ensemble, ces services fournissent une solution complète pour la collecte, l’analyse et l’action sur les données de télémétrie de vos applications et des ressources Azure qu’elles consomment.

Ce scénario traite des services de supervision que vous pouvez utiliser et décrit un modèle de flux de données à utiliser avec plusieurs sources de données. En ce qui concerne la supervision, des nombreux outils et services peuvent être utilisés avec les déploiements Azure. Dans ce scénario, nous choisissons des services justement disponibles parce qu’ils sont faciles à utiliser. D’autres options de supervision sont abordées plus loin dans cet article.

## <a name="relevant-use-cases"></a>Cas d’usage appropriés

Les autres cas d’usage appropriés sont les suivants :

- Instrumentation d’une application web pour la supervision des données de télémétrie.
- Collecte de données de télémétrie frontales et principales pour une application déployée sur Azure.
- Supervision de métriques et quotas associés aux services sur Azure.

## <a name="architecture"></a>Architecture

![« Diagramme d’architecture »](./images/architecture-diagram-app-monitoring.svg)

Ce scénario utilise un environnement Azure managé pour héberger une application et une couche Données. Les données circulent dans le scénario comme suit :

1. Un utilisateur interagit avec l’application.
2. Le navigateur et le service d’application génèrent des données de télémétrie.
3. Application Insights collecte et analyse les données relatives à l’intégrité, aux performances et à l’utilisation de l’application.
4. Les développeurs et administrateurs peuvent passer en revue les informations relatives à l’intégrité, aux performances et à l’utilisation.
5. Azure SQL Database génère des données de télémétrie.
6. Azure Monitor collecte et analyse des métriques et quotas et d’infrastructure.
7. Log Analytics collecte et analyse des journaux d’activité et métriques.
8. Les développeurs et administrateurs peuvent passer en revue les informations relatives à l’intégrité, aux performances et à l’utilisation.

### <a name="components"></a>Composants

- [Azure App Service](/azure/app-service/) est un service PaaS de création et d’hébergement d’applications dans des machines virtuelles managées. L’infrastructure de calcul sous-jacente sur laquelle vos applications sont exécutées est gérée pour vous. App Service permet de surveiller les quotas d’utilisation des ressources et les métriques d’application, les informations de journalisation des diagnostics et les alertes basées sur les métriques. Mieux encore, vous pouvez utiliser Application Insights pour créer des [tests de disponibilité][availability-tests] afin de tester votre application à partir de différentes régions.
- [Application Insights][application-insights] est un service extensible de gestion des performances des applications (APM) destiné aux développeurs et prend en charge plusieurs plateformes. Il surveille l’application, détecte les anomalies de l’application comme les faibles performances et les échecs, et envoie les données de télémétrie au portail Azure. Application Insights peut également être utilisé pour journalisation, le suivi distribué et les métriques d’application personnalisées.
- [Azure Monitor][azure-monitor] fournit des [métriques et des journaux d’activité][metrics] de niveau de base d’infrastructure pour la plupart des services Azure. Vous pouvez interagir de différentes manières avec les métriques, y compris en créant des graphiques dans le portail Azure, en y accédant via l’API REST ou en envoyant des requêtes avec PowerShell ou l’interface CLI. Azure Monitor fournit également ses données directement dans [Log Analytics et autres services], où vous pouvez les interroger et les associer à des données provenant d’autres sources locales ou dans le cloud.
- [Log Analytics][log-analytics] permet de mettre en corrélation les données de performances et d’utilisation collectées par Application Insights avec les données de performances et de configuration des ressources Azure qui prennent en charge l’application. Ce scénario utilise l’[agent Azure Log Analytics][Azure Log Analytics agent] pour envoyer des journaux d’audit SQL Server à Log Analytics. Vous pouvez écrire des requêtes et afficher les données dans le panneau Log Analytics du portail Azure.

## <a name="considerations"></a>Considérations

Une pratique recommandée consiste à ajouter Application Insights à votre code au cours du développement à l’aide des [Kits de développement logiciel (SDK) Application Insights][Application Insights SDKs] et à personnaliser par application. Ces Kits de développement logiciel (SDK) open source sont disponibles pour la plupart des infrastructures d’application. Pour enrichir et contrôler les données que vous collectez, intégrez l’utilisation des Kits de développement logiciel (SDK) des déploiements de test et de production dans votre processus de développement. La principale exigence implique que l’application doit disposer d’une visibilité directe ou indirecte sur le point de terminaison d’ingestion Applications Insights hébergé avec une adresse Internet. Vous pouvez ensuite ajouter des données de télémétrie ou enrichir une collection de données de télémétrie existante.

La supervision du runtime est une autre méthode de prise en main simple. Les données de télémétrie collectées doivent être contrôlées par le biais de fichiers de configuration. Par exemple, vous pouvez inclure des méthodes de runtime qui permettent à des outils tels qu’[Application Insights Status Monitor][Application Insights Status Monitor] de déployer les Kits de développement logiciel (SDK) dans le dossier approprié et d’ajouter les configurations qui conviennent pour commencer la supervision.

Comme Application Insights, Log Analytics fournit des outils d’[analyse des données entre des sources][analyzing data across sources], de création de requêtes complexes, et d’[envoi d’alertes proactives][sending proactive alerts] dans des conditions spécifiées. Vous pouvez également afficher les données de télémétrie dans le [portail Azure][the Azure portal]. Log Analytics enrichit les services de supervision existants tels qu’[Azure Monitor][azure-monitor] et peut également superviser les environnements locaux.

Application Insights et Log Analytics utilisent le [langage de requête Azure Log Analytics][Azure Log Analytics Query Language]. Vous pouvez également utiliser des [requêtes entre ressources](https://azure.microsoft.com/blog/query-across-resources) pour analyser les données de télémétrie collectées par Application Insights et Log Analytics dans une requête unique.

Azure Monitor, Application Insights et Log Analytics envoient tous les trois des [alertes](/azure/monitoring-and-diagnostics/monitoring-overview-alerts). Par exemple, Azure Monitor alerte sur des métriques au niveau de la plateforme, telles que l’utilisation du processeur, alors qu’Application Insights alerte sur des métriques au niveau de l’application, telles que le temps de réponse du serveur. Azure Monitor signale les nouveaux événements dans le journal d’activité Azure, alors que Log Analytics peut générer des alertes sur les données de métriques ou d’événements pour les services configurés pour les utiliser. Les [alertes unifiées dans Azure Monitor](/azure/monitoring-and-diagnostics/monitoring-overview-unified-alerts) constituent une nouvelle expérience d’alertes unifiée dans Azure qui utilise une autre taxonomie.

### <a name="alternatives"></a>Autres solutions

Cet article décrit les options de supervision facilement disponibles avec les fonctionnalités populaires, mais de nombreuses options s’offrent à vous, notamment la possibilité de créer vos propres mécanismes de journalisation. Une pratique recommandée consiste à ajouter des services de supervision lorsque vous générez des niveaux dans une solution. Voici certaines extensions et alternatives possibles :

- Consolidez les métriques Azure Monitor et Application Insights dans Grafana à l’aide d’[Azure Monitor Data Source For Grafana][Azure Monitor Data Source For Grafana].
- [Data Dog][data-dog] comprend un connecteur pour Azure Monitor
- Automatisez les fonctions de supervision à l’aide d’[Azure Automation][Azure Automation].
- Ajoutez une communication avec les [solutions ITSM][ITSM solutions].
- Étendez Log Analytics avec une [solution de gestion][management solution].

### <a name="scalability-and-availability"></a>Extensibilité et disponibilité

Ce scénario se concentre sur des solutions PaaS pour la supervision, principalement du fait qu’elles gèrent pour vous de manière pratique la disponibilité et l’extensibilité et qu’elles s’appuient sur des contrats de niveau de service (SLA). Par exemple, App Services fournit un [SLA][SLA] garanti pour sa disponibilité.

Application Insights [limite][app-insights-limits] le nombre de requêtes pouvant être traitées par seconde. Si vous dépassez la limite de requêtes, vous pouvez rencontrer une limitation des messages. Pour éviter cette limitation, implémentez un [filtrage][message-filtering] ou un [échantillonnage][message-sampling] afin de réduire le débit de données

Toutefois, les considérations relatives à la haute disponibilité de l’application que vous exécutez sont de la responsabilité du développeur. Pour plus d’informations sur la mise à l’échelle, par exemple, consultez la section des [considérations relatives à l’extensibilité](./basic-web-app.md#scalability-considerations) dans l’architecture de référence d’application web de base. Lorsqu’une application est déployée, vous pouvez configurer des tests pour [surveiller sa disponibilité][monitor its availability] à l’aide d’Application Insights.

### <a name="security"></a>Sécurité

Les exigences relatives aux informations personnelles et à la conformité affectent la collecte, la rétention et le stockage des données. En savoir plus sur la façon dont [Application Insights][application-insights] et [Log Analytics][log-analytics] gèrent les données de télémétrie.

Les considérations suivantes relatives à la sécurité peuvent également s’appliquer :

- Développez un plan pour gérer les informations personnelles si les développeurs sont autorisés à collecter leurs propres données ou à enrichir les données de télémétrie existantes.
- Envisagez la rétention des données. Par exemple, Application Insights conserve les données de télémétrie pendant 90 jours. Archivez les données auxquelles vous souhaitez accéder pendant une période plus longue à l’aide de Microsoft Power BI, l’exportation continue ou l’API REST. Des tarifs de stockage s’appliquent.
- Limitez l’accès aux ressources Azure afin de contrôler l’accès aux données et qui peut afficher des données de télémétrie à partir d’une application spécifique. Pour mieux limiter l’accès aux données de télémétrie de supervision, consultez [Contrôle d’accès, rôles et ressources dans Application Insights][Resources, roles, and access control in Application Insights].
- Pensez à contrôler l’accès en lecture/écriture au code d’application pour empêcher les utilisateurs d’ajouter des marqueurs de version ou de balise qui limitent l’ingestion de données à partir de l’application. Avec Application Insights, il n’existe aucun contrôle des éléments de données une fois qu’ils sont envoyés à une ressource. Ainsi, si un utilisateur a accès à toutes les données, il a également accès à toutes les données d’une ressource.
- Si nécessaire, ajoutez des mécanismes de [gouvernance](/azure/security/governance-in-azure) pour appliquer des contrôles de stratégie ou de coût sur des ressources Azure. Par exemple, utilisez Log Analytics pour la supervision liée à la sécurité comme les stratégies et le contrôle d’accès en fonction du rôle, ou utilisez [Azure Policy](/azure/azure-policy/azure-policy-introduction) pour créer, affecter et gérer des définitions de stratégie.
- Pour surveiller les éventuels problèmes de sécurité et obtenir une vue centralisée de l’état de sécurité de vos ressources Azure, envisagez d’utiliser [Azure Security Center](/azure/security-center/security-center-intro).

## <a name="pricing"></a>Tarifs

Les frais de supervision peuvent augmenter rapidement. Envisagez donc la tarification en amont, de comprendre ce que vous surveillez et de vérifier les frais associés de chaque service. Azure Monitor fournit sans frais des [métriques de base][basic metrics], alors que les coûts de supervision d’[Application Insights][application-insights-pricing] et de [Log Analytics][log-analytics] sont basés sur la quantité de données ingérées et le nombre de tests que vous exécutez.

Pour vous aider à commencer, utilisez la [calculatrice de prix][pricing] pour estimer les coûts. Pour pouvoir observer l’évolution de la tarification pour votre cas d’usage particulier, modifiez les diverses options en fonction du déploiement que vous escomptez.

Les données d’Application Insights sont envoyées au portail Azure pendant le débogage et une fois que vous avez publié votre application. À des fins de test et pour éviter des frais, un volume limité de données de télémétrie est instrumenté. Pour ajouter d’autres indicateurs, vous pouvez augmenter la limite de données de télémétrie. Pour un contrôle plus précis, consultez l’article [Échantillonnage dans Application Insights][Sampling in Application Insights].

Après le déploiement, vous pouvez observer un [Flux de métriques temps réel][Live Metrics Stream] des indicateurs de performance. Ces données ne sont pas stockées &mdash; vous affichez des métriques en temps réel &mdash;, mais les données de télémétrie peuvent être collectées et analysées ultérieurement. Aucuns frais ne sont liés aux données Flux de métriques temps réel.

Log Analytics est facturé par Go de données ingéré dans le service. Les 5 premiers Go de données ingérés dans le service Azure Log Analytics chaque mois sont gratuits, et les données sont conservées sans frais pendant les 31 premiers jours dans votre espace de travail Log Analytics.

## <a name="next-steps"></a>Étapes suivantes

Découvrez ces ressources conçues pour vous aider à bien démarrer avec votre propre solution de supervision :

[Architecture de référence d’application web de base][Basic web application reference architecture]

[Démarrer la supervision de votre application web ASP.NET][Start monitoring your ASP.NET Web Application]

[Collecter des données sur les machines virtuelles Azure][Collect data about Azure Virtual Machines]

## <a name="related-resources"></a>Ressources associées

[Supervision des applications et des ressources Azure][Monitoring Azure applications and resources]

[Rechercher et diagnostiquer des exceptions runtime avec Azure Application Insights][Find and diagnose run-time exceptions with Azure Application Insights]

<!-- links -->
[architecture]: ./images/architecture-diagram-app-monitoring.svg
[availability-tests]: /azure/application-insights/app-insights-monitor-web-app-availability
[application-insights]: /azure/application-insights/app-insights-overview
[azure-monitor]: /azure/monitoring-and-diagnostics/monitoring-overview-azure-monitor
[metrics]: /azure/monitoring-and-diagnostics/monitoring-supported-metrics
[Log Analytics et autres services]: /azure/log-analytics/log-analytics-azure-storage
[log-analytics]: /azure/log-analytics/log-analytics-overview
[Azure Log Analytics agent]: https://blogs.msdn.microsoft.com/sqlsecurity/2017/12/28/azure-log-analytics-oms-agent-now-collects-sql-server-audit-logs/
[application-insights-pricing]: https://azure.microsoft.com/pricing/details/application-insights/
[Application Insights SDKs]: /azure/application-insights/app-insights-asp-net
[Application Insights Status Monitor]: https://azure.microsoft.com/updates/application-insights-status-monitor-and-sdk-updated/
[analyzing data across sources]: /azure/log-analytics/log-analytics-dashboards
[sending proactive alerts]: /azure/log-analytics/log-analytics-alerts
[the Azure portal]: /azure/log-analytics/log-analytics-tutorial-dashboards
[Azure Log Analytics Query Language]: https://docs.loganalytics.io/docs/Learn
[cross-resource queries]: https://azure.microsoft.com/blog/query-across-resources/
[alerts]: /azure/monitoring-and-diagnostics/monitoring-overview-alerts
[Alerts (Preview)]: /azure/monitoring-and-diagnostics/monitoring-overview-unified-alerts
[Azure Monitor Data Source For Grafana]: https://grafana.com/plugins/grafana-azure-monitor-datasource
[Azure Automation]: /azure/automation/automation-intro
[ITSM solutions]: https://azure.microsoft.com/blog/itsm-connector-for-azure-is-now-generally-available/
[management solution]: /azure/monitoring/monitoring-solutions
[SLA]: https://azure.microsoft.com/support/legal/sla/app-service/v1_4/
[monitor its availability]: /azure/application-insights/app-insights-monitor-web-app-availability
[Resources, roles, and access control in Application Insights]: /azure/application-insights/app-insights-resources-roles-access-control
[basic metrics]: /azure/monitoring-and-diagnostics/monitoring-supported-metrics
[pricing]: https://azure.microsoft.com/pricing/calculator/#log-analyticsc126d8c1-ec9c-4e5b-9b51-4db95d06a9b1
[Sampling in Application Insights]: /azure/application-insights/app-insights-sampling
[Live Metrics Stream]: /azure/application-insights/app-insights-live-stream
[Basic web application reference architecture]: /azure/architecture/reference-architectures/app-service-web-app/basic-web-app#scalability-considerations
[Start monitoring your ASP.NET Web Application]: /azure/application-insights/quick-monitor-portal
[Collect data about Azure Virtual Machines]: /azure/log-analytics/log-analytics-quick-collect-azurevm
[Monitoring Azure applications and resources]: /azure/monitoring-and-diagnostics/monitoring-overview
[Find and diagnose run-time exceptions with Azure Application Insights]: /azure/application-insights/app-insights-tutorial-runtime-exceptions
[data-dog]: https://www.datadoghq.com/blog/azure-monitoring-enhancements/
[app-insights-limits]: /azure/azure-subscription-service-limits#application-insights-limits
[message-filtering]: /azure/application-insights/app-insights-api-filtering-sampling
[message-sampling]: /azure/application-insights/app-insights-sampling
