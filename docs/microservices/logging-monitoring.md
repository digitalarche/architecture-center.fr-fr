---
title: Journalisation et surveillance dans les microservices
description: Journalisation et surveillance dans les microservices
author: MikeWasson
ms.date: 12/08/2017
ms.openlocfilehash: 1da67047daa9ae87cda5dd7dd581d6081183c428
ms.sourcegitcommit: 786bafefc731245414c3c1510fc21027afe303dc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2017
ms.locfileid: "26652992"
---
# <a name="designing-microservices-logging-and-monitoring"></a>Conception de microservices : journalisation et surveillance

Dans toute application complexe, il arrive toujours un moment où quelque chose pose problème. Dans une application de microservices, vous devez suivre le déroulement des opérations dans plusieurs douzaines, voire plusieurs centaines de services. Les fonctions de journalisation et de surveillance jouent un rôle primordial dans la présentation d’une vue holistique du système. 

![](./images/monitoring.png)

Dans une architecture de microservices, l’identification de la cause précise des erreurs ou des goulots d’étranglement des performances peut se révéler particulièrement ardue. Une même opération utilisateur peut s’étendre sur plusieurs services. Il est possible que des services atteignent les limites d’E/S réseau au sein du cluster. Une chaîne d’appels entre les services peut engendrer une régulation de flux dans le système, entraînant ainsi une latence élevée ou des échecs en cascade. En outre, vous ignorez généralement dans quel nœud un conteneur spécifique s’exécutera. Les conteneurs placés sur le même nœud peuvent entrer en concurrence pour bénéficier de ressources processeur ou mémoire limitées. 

Pour faciliter la compréhension du déroulement des opérations, l’application doit émettre des événements de télémétrie. Vous pouvez classer ces événements en mesures et en journaux textuels. 

Les *mesures* sont des valeurs numériques qui peuvent être analysées. Vous pouvez les utiliser pour observer le système en temps réel (ou quasiment en temps réel) ou pour analyser les tendances des performances au fil du temps. Les mesures disponibles sont les suivantes :

- Mesures système de niveau nœud, comprenant l’utilisation de l’UC, de la mémoire, du réseau, du disque et du système de fichiers. Les mesures système vous aident à comprendre l’allocation des ressources pour chacun des nœuds du cluster et à corriger les valeurs hors norme.
 
- Mesures Kubernetes. Étant donné que les services s’exécutent dans des conteneurs, vous devez collecter les mesures au niveau du conteneur, et non simplement au niveau de la machine virtuelle. Dans Kubernetes, cAdvisor (Container Advisor) est l’agent qui collecte les statistiques relatives aux ressources processeur, mémoire, système de fichiers et réseau utilisées par chaque conteneur. Le démon kubelet collecte les statistiques de ressources recueillies par cAdvisor et les expose par le biais d’une API REST.
   
- Mesures d’application. Ces dernières englobent toutes les mesures qui permettent de comprendre le comportement d’un service. Il s’agit par exemple du nombre de requêtes HTTP entrantes en file d’attente, de la latence des requêtes, de la longueur de la file d’attente de messages ou du nombre de transactions traitées par seconde.

- Mesures des services dépendants. Les services au sein du cluster peuvent appeler des services externes situés en dehors du cluster, tels que des services PaaS gérés. Vous pouvez surveiller les services Azure à l’aide de la plateforme [Azure Monitor](/azure/monitoring-and-diagnostics/monitoring-overview). Les services tiers peuvent ou non fournir des mesures. Si ce n’est pas le cas, vous devrez vous appuyer sur vos propres mesures d’application pour effectuer le suivi des statistiques relatives aux taux de latence et d’erreurs.

Les *journaux* sont des enregistrements des événements qui surviennent pendant l’exécution de l’application. Ils comprennent des éléments tels que les journaux d’application (déclarations de trace) ou les journaux de serveur web. Les journaux se révèlent particulièrement utiles pour la forensique et l’analyse de la cause racine. 

## <a name="considerations"></a>Considérations

L’article [Surveillance et diagnostics](../best-practices/monitoring.md) décrit les meilleures pratiques générales en matière de surveillance d’une application. Voici quelques points spécifiques à prendre en compte dans le contexte d’une architecture de microservices.

**Configuration et gestion**. Prévoyez-vous d’utiliser un service géré pour la journalisation et la surveillance, ou envisagez-vous de déployer des composants de journalisation et de surveillance sous forme de conteneurs à l’intérieur du cluster ? Pour plus d’informations sur ces possibilités, consultez la section [Options technologiques](#technology-options) ci-dessous.

**Taux d’ingestion**. À quelle vitesse le système peut-il ingérer les événements de télémétrie ? Que se passe-t-il si ce taux est dépassé ? Par exemple, le système peut limiter les clients (auquel cas les données de télémétrie sont perdues), ou il peut sous-échantillonner les données. Vous pouvez parfois atténuer ce problème en réduisant la quantité de données que vous collectez :

  - Agrégez les mesures en calculant les statistiques, telles que la moyenne et l’écart type, puis envoyez ces données statistiques au système de surveillance.  

  - Sous-échantillonnez les données ; autrement dit, ne traitez qu’un certain pourcentage des événements.

  - Traitez les données par lot afin de réduire le nombre d’appels réseau adressés au service de surveillance.

**Coût**. Le coût d’ingestion et de stockage des données de télémétrie peut être substantiel, en particulier si ces données sont volumineuses. Dans certains cas, ce coût peut même dépasser celui de l’exécution de l’application. Dans cette éventualité, vous devrez peut-être réduire le volume de données de télémétrie en agrégeant ou en sous-échantillonnant les données, ou en traitant les données par lot, comme décrit ci-dessus. 
        
**Fidélité des données**. Quel est le degré de précision des mesures ? Les moyennes peuvent masquer des valeurs hors norme, en particulier à grande échelle. En outre, si le taux d’échantillonnage est trop faible, il risque de lisser les fluctuations des données. Ainsi, vous pouvez avoir l’impression que toutes les requêtes présentent approximativement la même latence de bout en bout, alors qu’en réalité, une partie significative des requêtes exigent beaucoup plus de temps. 

**Latence**. Pour bénéficier de fonctions de surveillance et d’émission d’alertes en temps réel, vous devez pouvoir accéder rapidement aux données de télémétrie. Dans quelle mesure les données présentées dans le tableau de bord de surveillance sont-elles éloignées des données en temps réel ? Le sont-elles de quelques secondes ? De plus d’une minute ?

**Stockage**. Dans le cas des journaux, il peut se révéler plus efficace d’écrire les événements de journal dans le stockage éphémère du cluster, et de configurer un agent pour qu’il expédie les fichiers journaux vers un stockage plus persistant.  Au final, les données doivent être déplacées vers le stockage à long terme afin d’être disponibles pour une analyse rétrospective. Étant donné qu’une architecture de microservices peut générer un gros volume de données de télémétrie, le coût du stockage de ces données constitue un facteur important. Prenez également en compte la façon dont vous interrogerez les données. 

**Tableau de bord et visualisation**. Bénéficiez-vous d’une vue holistique du système englobant la totalité des services au sein du cluster et des services externes ? Si vous écrivez des données de télémétrie et des journaux à différents emplacements, le tableau de bord peut-il présenter et mettre en corrélation l’ensemble de ces informations ? Le tableau de bord de surveillance doit au moins fournir les informations suivantes :

- allocation globale des ressources à des fins de capacité et de croissance, comprenant le nombre de conteneurs, les mesures de système de fichiers, ainsi que l’allocation réseau et de mémoire centrale ;
- mesures de conteneur mises en corrélation au niveau service ;
- mesures système mises en corrélation avec les conteneurs ;
- erreurs de service et valeurs hors norme.
    

## <a name="distributed-tracing"></a>Traçage distribué

Comme indiqué précédemment, l’une des difficultés inhérentes aux microservices réside dans la compréhension du flux d’événements dans l’ensemble des services. Une même opération ou transaction peut impliquer des appels adressés à plusieurs services. Pour permettre de reconstruire l’intégralité de la séquence des étapes, chaque service doit propager un *ID de corrélation* faisant office d’identificateur unique pour cette opération. L’ID de corrélation autorise un [traçage distribué](http://microservices.io/patterns/observability/distributed-tracing.html) dans l’ensemble des services.

Le premier service à recevoir une requête client doit générer l’ID de corrélation. Si le service adresse un appel HTTP à un autre service, il place l’ID de corrélation dans un en-tête de requête. De même, si le service envoie un message asynchrone, il place l’ID de corrélation dans le message. Les services en aval continuent de propager cet ID de corrélation, de sorte que ce dernier transite par la totalité du système. En outre, l’ensemble du code qui écrit des mesures d’application ou des événements de journal doit inclure l’ID de corrélation.

Lors de la mise en corrélation des appels de service, vous pouvez calculer des mesures opérationnelles telles que la latence de bout en bout d’une transaction complète, le nombre de transactions réussies par seconde et le pourcentage de transactions ayant échoué. L’inclusion des ID de corrélation dans les journaux des applications permet d’effectuer l’analyse de la cause racine. Si une opération échoue, vous pouvez trouver les instructions de journal concernant tous les appels de service qui faisaient partie de la même opération. 

Voici quelques considérations à prendre en compte lors de l’implémentation du traçage distribué :

- Pour l’instant, il n’existe aucun en-tête HTTP standard pour les ID de corrélation. Votre équipe doit s’accorder sur une valeur d’en-tête personnalisée. Ce choix peut être conditionné par votre infrastructure de journalisation/surveillance ou par votre décision d’utiliser une maille de services.

- Dans le cas des messages asynchrones, si votre infrastructure de messagerie prend en charge l’ajout de métadonnées aux messages, vous devez inclure l’ID de corrélation sous la forme d’une métadonnée. Dans le cas contraire, intégrez cet ID au schéma de message.

- Au lieu d’un simple identificateur opaque, vous pouvez envoyer un *contexte de corrélation* contenant des informations plus complètes, telles que les relations appelant-appelé. 

- Le Kit de développement logiciel (SDK) Azure Application Insights injecte automatiquement le contexte de corrélation dans les en-têtes HTTP et inclut l’ID de corrélation dans les journaux d’Application Insights. Si vous décidez d’utiliser les fonctionnalités de corrélation intégrées à Application Insights, certains services peuvent malgré tout nécessiter la propagation explicite des en-têtes de corrélation, selon les bibliothèques utilisées. Pour plus d’informations, consultez l’article [Corrélation de télémétrie dans Application Insights](/azure/application-insights/application-insights-correlation).
   
- Si vous utilisez Istio ou linkerd en tant que maille de services, ces technologies génèrent automatiquement des en-têtes de corrélation lorsque les appels HTTP sont acheminés par le biais des proxys de maille de services. Les services doivent transférer les en-têtes appropriés. 

    - Istio : [Distributed Request Tracing (Traçage de requêtes distribué)](https://istio-releases.github.io/v0.1/docs/tasks/zipkin-tracing.html)
    
    - linkerd : [Context Headers (En-têtes de contexte)](https://linkerd.io/config/1.3.0/linkerd/index.html#http-headers)
    
- Déterminez la façon dont vous agrégerez les journaux. Vous pouvez faire en sorte de normaliser le mode d’inclusion des ID de corrélation dans les journaux par les différentes équipes. Utilisez un format structuré ou semi-structuré, tel que JSON, et définissez un champ commun pour le stockage de l’ID de corrélation.

## <a name="technology-options"></a>Options technologiques

**Application Insights** est un service géré dans Azure qui ingère et stocke des données de télémétrie et qui fournit des outils permettant d’analyser et de rechercher les données. Pour utiliser Application Insights, vous installez un package d’instrumentation dans votre application. Ce package surveille l’application et envoie les données de télémétrie au service Application Insights. Il peut également extraire les données de télémétrie de l’environnement hôte. Application Insights intègre des fonctions de mise en corrélation et de suivi des dépendances. Ce service vous permet de suivre l’ensemble des mesures système, des mesures d’application et des mesures de service Azure au même endroit.

Gardez à l’esprit qu’Application Insights applique une limitation si le débit de données dépasse une limite maximale ; pour plus d’informations, consultez la section [Limites d’Application Insights](/azure/azure-subscription-service-limits#application-insights-limits). Une même opération peut générer plusieurs événements de télémétrie ; par conséquent, si l’application est confrontée à un trafic dense, elle fera probablement l’objet d’une limitation. Pour atténuer ce problème, vous pouvez procéder à un échantillonnage afin de réduire le trafic de télémétrie. Cette approche présente l’inconvénient d’engendrer des mesures moins précises. Pour plus d’informations, consultez l’article [Échantillonnage dans Application Insights](/azure/application-insights/app-insights-sampling). Vous pouvez également réduire le volume de données en agrégeant préalablement les mesures, autrement dit en calculant les valeurs statistiques telles que la moyenne et l’écart type, puis en envoyant ces valeurs plutôt que les données de télémétrie brutes. L’approche qui consiste à utiliser Application Insights à grande échelle est décrite dans le billet de blog [Azure Monitoring and Analytics at Scale (Surveillance et analyse Azure à grande échelle)](https://blogs.msdn.microsoft.com/azurecat/2017/05/11/azure-monitoring-and-analytics-at-scale/).

En outre, assurez-vous que vous avez bien compris le modèle de tarification d’Application Insights, car vous serez facturé en fonction du volume de données. Pour plus d’informations, consultez l’article [Gérer la tarification et le volume de données dans Application Insights](/azure/application-insights/app-insights-pricing). Si votre application génère un gros volume de données de télémétrie et que vous ne souhaitez pas effectuer un échantillonnage ou une agrégation des données, Application Insights ne constitue peut-être pas le choix approprié. 

Si Application Insights ne répond pas à vos besoins, voici certaines suggestions d’approches qui utilisent des technologies open source populaires.

Dans le cas des mesures système et de conteneur, envisagez d’exporter ces mesures vers une base de données en séries chronologiques telle que **Prometheus** ou **InfluxDB** qui s’exécute dans le cluster.

- InfluxDB est un système basé sur la transmission de type push. Un agent doit transmettre (push) les mesures. Vous pouvez utiliser le service [Heapster][heapster], qui collecte les mesures à l’échelle du cluster à partir d’un kubelet, agrège les données, puis les transmet à InfluxDB ou à une autre solution de stockage de séries chronologiques. Azure Container Service déploie Heapster dans le cadre de la configuration du cluster. Une autre option consiste à utiliser l’agent [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/), qui collecte et rapporte les mesures. 

- Prometheus est un système basé sur l’extraction (pull). Il récupère régulièrement les mesures à partir d’emplacements configurés. Prometheus peut récupérer les mesures générées par cAdvisor ou par kube-state-metrics. [kube-state-metrics][kube-state-metrics] est un service qui collecte des mesures à partir du serveur d’API Kubernetes et les rend accessibles à Prometheus (ou à un système de récupération compatible avec un point de terminaison client Prometheus). Alors que Heapster agrège les mesures que Kubernetes génère et transfère à un récepteur, kube-state-metrics génère ses propres mesures et les rend récupérables par le biais d’un point de terminaison. Pour les mesures système, utilisez [l’exportateur Node](https://github.com/prometheus/node_exporter), qui est un exportateur Prometheus pour les mesures système. Le système Prometheus prend en charge les données de type virgule flottante, mais non les données de type chaîne ; il se révèle donc adapté aux mesures système, mais pas aux journaux.

- Pour visualiser et surveiller les données, utilisez un outil de tableau de bord tel que **Kibana** ou **Grafana**. Le service de tableau de bord peut également s’exécuter à l’intérieur d’un conteneur dans le cluster.

Pour les journaux d’application, envisagez d’utiliser **Fluentd** et **Elasticsearch**. Fluentd est un collecteur de données open source, tandis qu’Elasticsearch est une base de données de documents qui est optimisée pour jouer le rôle de moteur de recherche. Dans le cadre de cette approche, chaque service envoie des journaux à `stdout` et `stderr`, et Kubernetes écrit ces flux dans le système de fichiers local. Fluentd collecte les journaux, les enrichit éventuellement avec des métadonnées supplémentaires de Kubernetes, puis envoie les journaux à Elasticsearch. Utilisez Kibana, Grafana ou un outil similaire afin de créer un tableau de bord pour Elasticsearch. Fluentd s’exécute en tant que daemonset dans le cluster, ce qui garantit qu’un seul pod Fluentd est attribué à chaque nœud. Vous pouvez configurer Fluentd pour qu’il collecte les journaux de kubelet, ainsi que les journaux de conteneur. Dans le cas de volumes élevés, l’écriture de journaux dans le système de fichiers local peut devenir un goulot d’étranglement des performances, en particulier lorsque plusieurs services sont en cours d’exécution sur le même nœud. Surveillez la latence de disque et l’utilisation du système de fichiers en production.

L’un des avantages liés à l’utilisation de Fluentd avec Elasticsearch pour les journaux réside dans le fait que les services ne requièrent aucune dépendance de bibliothèque supplémentaire. Chaque service écrit simplement dans `stdout` et dans `stderr`, et Fluentd gère l’exportation des journaux dans Elasticsearch. En outre, les équipes qui écrivent les services n’ont pas besoin de comprendre comment configurer l’infrastructure de journalisation. L’une des difficultés consiste à configurer le cluster Elasticsearch pour un déploiement de production afin de l’adapter à la gestion de votre trafic. 

Une autre option consiste à envoyer les fichiers journaux au service Log Analytics d’Operations Management Suite (OMS). Le service [Log Analytics][log-analytics] collecte les données de journal dans un référentiel central, et peut également consolider les données issues d’autres services Azure que votre application utilise. Pour plus d’informations, consultez l’article [Surveiller un cluster Azure Container Service avec Microsoft Operations Management Suite (OMS)][k8s-to-oms].

## <a name="example-logging-with-correlation-ids"></a>Exemple : journalisation avec des ID de corrélation

Pour illustrer certains des points abordés dans cet article, voici un exemple détaillé de la façon dont le service Package implémente la journalisation. Le service Package a été écrit en TypeScript et utilise l’infrastructure web [Koa](http://koajs.com/) pour Node.js. Vous pouvez choisir entre plusieurs bibliothèques de journalisation Node.js. Nous avons choisi [Winston](https://github.com/winstonjs/winston), une bibliothèque de journalisation communément utilisée qui répondait à nos besoins en matière de performances lorsque nous l’avons testée.

Pour encapsuler les détails d’implémentation, nous avons défini une interface `ILogger` abstraite :

```ts
export interface ILogger {
    log(level: string, msg: string, meta?: any)
    debug(msg: string, meta?: any)
    info(msg: string, meta?: any)
    warn(msg: string, meta?: any)
    error(msg: string, meta?: any)
}
```

Voici une implémentation d’`ILogger` qui inclut la bibliothèque Winston dans un wrapper. Elle utilise l’ID de corrélation en tant que paramètre de constructeur et injecte cet ID dans chaque message de journal. 

```ts
class WinstonLogger implements ILogger {
    constructor(private correlationId: string) {}
    log(level: string, msg: string, payload?: any) {
        var meta : any = {};
        if (payload) { meta.payload = payload };
        if (this.correlationId) { meta.correlationId = this.correlationId }
        winston.log(level, msg, meta)
    }
  
    info(msg: string, payload?: any) {
        this.log('info', msg, payload);
    }
    debug(msg: string, payload?: any) {
        this.log('debug', msg, payload);
    }
    warn(msg: string, payload?: any) {
        this.log('warn', msg, payload);
    }
    error(msg: string, payload?: any) {
        this.log('error', msg, payload);
    }
}
```

Le service Package a besoin d’extraire l’ID de corrélation de la requête HTTP. Par exemple, si vous utilisez linkerd, l’ID de corrélation figure dans l’en-tête `l5d-ctx-trace`. Dans Koa, la requête HTTP est stockée dans un objet de contexte transmis par le biais du pipeline de traitement des requêtes. Nous pouvons définir une fonction d’intergiciel (middleware) pour obtenir l’ID de corrélation à partir de l’objet de contexte et initialiser l’enregistreur d’événements. (Une fonction d’intergiciel (middleware) dans Koa correspond simplement à une fonction qui est exécutée pour chaque requête.)

```ts
export type CorrelationIdFn = (ctx: Context) => string;

export function logger(level: string, getCorrelationId: CorrelationIdFn) {
    winston.configure({ 
        level: level,
        transports: [new (winston.transports.Console)()]
        });
    return async function(ctx: any, next: any) {
        ctx.state.logger = new WinstonLogger(getCorrelationId(ctx));
        await next();
    }
}
```

Cet intergiciel (middleware) appelle une fonction définie par l’appelant, `getCorrelationId`, afin d’obtenir l’ID de corrélation. Puis il crée une instance de l’enregistreur d’événements et le remise (stash) dans `ctx.state`, qui constitue un dictionnaire de clés-valeurs utilisé dans Koa pour transmettre des informations par le biais du pipeline. 

L’intergiciel (middleware) d’enregistreur d’événements est ajouté au pipeline au moment du démarrage :

```ts
app.use(logger(Settings.logLevel(), function (ctx) {
    return ctx.headers[Settings.correlationHeader()];  
}));
```

Une fois tous les éléments configurés, il est facile d’ajouter des instructions de journalisation au code. Par exemple, voici la méthode qui recherche un package. Elle effectue deux appels à la méthode `ILogger.info`.

```ts
async getById(ctx: any, next: any) {
  var logger : ILogger = ctx.state.logger;
  var packageId = ctx.params.packageId;
  logger.info('Entering getById, packageId = %s', packageId);

  await next();

  let pkg = await this.repository.findPackage(ctx.params.packageId)

  if (pkg == null) {
    logger.info(`getById: %s not found`, packageId);
    ctx.response.status= 404;
    return;
  }

  ctx.response.status = 200;
  ctx.response.body = this.mapPackageDbToApi(pkg);
}
```

Nous n’avons pas besoin d’inclure l’ID de corrélation dans les instructions de journalisation, car cette opération est automatiquement effectuée par la fonction d’intergiciel (middleware). Ceci simplifie le code de journalisation et réduit le risque qu’un développeur oublie d’inclure l’ID de corrélation. En outre, étant donné que toutes les instructions de journalisation utilisent l’interface `ILogger` abstraite, l’implémentation de l’enregistreur d’événements sera facilement remplaçable ultérieurement.

> [!div class="nextstepaction"]
> [Intégration et livraison continues](./ci-cd.md)

<!-- links -->

[app-insights]: /azure/application-insights/app-insights-overview
[heapster]: https://github.com/kubernetes/heapster
[kube-state-metrics]: https://github.com/kubernetes/kube-state-metrics
[k8s-to-oms]: /azure/container-service/kubernetes/container-service-kubernetes-oms
[log-analytics]: /azure/log-analytics/