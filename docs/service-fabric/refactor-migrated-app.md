---
title: "Refactoriser une application Azure Service Fabric migrée depuis Azure Cloud Services"
description: "Comment refactoriser une application Azure Service Fabric existante migrée depuis Azure Cloud Services"
author: petertay
ms.date: 01/30/2018
ms.openlocfilehash: 4889fae8f157b0f1205e7d8223f125974be59ba9
ms.sourcegitcommit: 2c9a8edf3e44360d7c02e626ea8ac3b03fdfadba
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/03/2018
---
# <a name="refactor-an-azure-service-fabric-application-migrated-from-azure-cloud-services"></a>Refactoriser une application Azure Service Fabric migrée depuis Azure Cloud Services

[![GitHub](../_images/github.png) Exemple de code][sample-code]

Cet article décrit la refactorisation d’une application Azure Service Fabric existante en une architecture plus granulaire. Cet article se concentre sur les considérations de conception, d’empaquetage, de performances et de déploiement de l’application Service Fabric refactorisée.

## <a name="scenario"></a>Scénario

Comme indiqué dans l’article précédent, [Migration d’une application Azure Cloud Services vers Azure Service Fabric][migrate-from-cloud-services], l’équipe Patterns & Practices a écrit un livre en 2012, décrivant le processus de conception et d’implémentation d’une application Cloud Services dans Azure. Ce manuel décrit une société fictive nommée Tailspin souhaitant créer une application Cloud Services nommée **Surveys**. L’application Surveys permet aux utilisateur de créer et de publier des enquêtes auxquelles le public peut répondre. Le diagramme suivant illustre l’architecture de cette version de l’application Surveys :

![](./images/tailspin01.png)

Le rôle web **Tailspin.Web** héberge un site ASP.NET MVC que les clients de Tailspin utilisent pour :
* s’inscrire à l’application Surveys,
* créer ou supprimer une enquête unique,
* voir les résultats d’une enquête unique,
* demander que les résultats d’enquête soient exportés en SQL, et
* afficher l’analyse et les résultats d’enquête agrégés.

Le rôle web **Tailspin.Web.Survey.Public** héberge également un site ASP.NET MVC visité par le public pour remplir les enquêtes. Ces réponses sont placées dans une file d’attente pour être enregistrées.

Le rôle de travail **Tailspin.Workers.Survey** exécute un traitement en arrière-plan en choisissant les requêtes à partir de plusieurs files d’attente.

L’équipe Patterns & Practices a ensuite créé un nouveau projet pour porter cette application sur Azure Service Fabric. L’objectif de ce projet était d’effectuer uniquement les modifications de code nécessaires pour permettre l’exécution de l’application dans un cluster Azure Service Fabric. Par conséquent, les rôles web et de travail originaux n’ont pas été décomposés en une architecture plus granulaire. L’architecture obtenue est très similaire à la version Cloud Service de l’application :

![](./images/tailspin02.png)

Le service **Tailspin.Web** est porté à partir du rôle web *Tailspin.Web* d’origine.

Le service **Tailspin.Web.Survey.Public** est porté à partir du rôle web *Tailspin.Web.Survey.Public* d’origine.

Le service **Tailspin.AnswerAnalysisService** est porté à partir du rôle de travail *Tailspin.AnswerAnalysisService* d’origine.

> [!NOTE] 
> Alors que les modifications apportées au code des rôles web et de travail ont été minimes, **Tailspin.Web** et **Tailspin.Web.Survey.Public** ont été modifiés pour auto-héberger un serveur web [Kestrel]. L’application Surveys antérieure est une application ASP.Net qui était hébergée par Interet Information Services (IIS), mais il est impossible d’exécuter IIS en tant que service dans Service Fabric. Par conséquent, n’importe quel serveur web doit pouvoir être auto-hébergé, comme [Kestrel]. Dans certaines situations, il est possible d’exécuter IIS à l’intérieur d’un conteneur dans Service Fabric. Voir [Scénarios d’utilisation des conteneurs][container-scenarios] pour plus d’informations.  

Maintenant, Tailspin refactorise l’application Surveys en une architecture plus granulaire. La motivation de Tailspin pour la refactorisation est de faciliter le développement, la compilation et le déploiement de l’application Surveys. En décomposant les rôles web et de travail en une architecture plus granulaire, Tailspin souhaite supprimer les dépendances de données et les communications existantes étroitement couplées entre ces rôles.

Tailspin voit d’autres avantages au passage de l’application Surveys en une architecture plus granulaire :
* Chaque service peut être empaqueté dans des projets indépendants ayant une portée suffisamment petite pour être gérés par une petite équipe.
* Chaque service peut être déployé et faire l’objet d’un contrôle de version indépendamment.
* Chaque service peut être implémenté à l’aide de la meilleure technologie possible. Par exemple, un cluster Service Fabric peut inclure des services construits à l’aide de différentes versions de .NET Framework, Java ou d’autres langages tels que C ou C++.
* Chaque service peut être mis à l’échelle indépendamment pour répondre aux augmentations et diminutions de charge.

> [!NOTE] 
> La multilocation est hors de portée pour la refactorisation de cette application. Tailspin dispose de plusieurs options pour prendre en charge la multilocation et peut prendre ces décisions de conception plus tard sans affecter la conception initiale. Par exemple, Tailspin peut créer des instances de services distinctes pour chaque client au sein d’un cluster ou créer un cluster séparé pour chaque client.

## <a name="design-considerations"></a>Remarques relatives à la conception
 
Le diagramme suivant illustre l’architecture de l’application Surveys refactorisée en une architecture plus granulaire :

![](./images/surveys_03.png)

**Tailspin.Web** est un service sans état auto-hébergeant une application ASP.NET MVC visitée par les clients de Tailspin pour créer des enquêtes et afficher des résultats d’enquête. Ce service partage la majeure partie de son code avec le service *Tailspin.Web* de l’application Service Fabric migrée. Comme mentionné précédemment, ce service utilise ASP.NET Core et passe d’une utilisation de Kestrel en tant que serveur web frontal à l’implémentation de WebListener.

**Tailspin.Web.Surveys.Public** est un service sans état qui auto-héberge également un site ASP.NET MVC. Les utilisateurs visitent ce site pour sélectionner des enquêtes depuis une liste et les remplir. Ce service partage la majeure partie de son code avec le service *Tailspin.Web.Survey.Public* de l’application Service Fabric migrée. Ce service utilise également ASP.NET Core et passe également d’une utilisation de Kestrel en tant que serveur web frontal à l’implémentation de WebListener.

**Tailspin.SurveyResponseService** est un service avec état qui stocke les réponses de l’enquête dans le Stockage Blob Azure. Il fusionne également les réponses dans des données d’analyse d’enquête. Le service est implémenté comme un service avec état, car elle utilise un [ReliableConcurrentQueue][reliable-concurrent-queue] pour traiter les réponses d’enquête en lots. Cette fonctionnalité a été initialement implémentée dans le service *Tailspin.Web.Survey.Public* de l’application Service Fabric migrée. Tailspin a refactorisé les fonctionnalités d’origine dans ce service pour lui permettre une mise en l’échelle de manière indépendante.

**Tailspin.SurveyManagementService** est un service sans état qui stocke et récupère les enquêtes et les questions d’enquête. Le service utilise le Stockage Blob Azure. Cette fonctionnalité a également été initialement implémentée dans le service *Tailspin.AnswerAnalysisService* dans l’application Service Fabric migrée. Tailspin a refactorisé les fonctionnalités d’origine dans ce service pour lui permettre également une mise en l’échelle de manière indépendante.

**Tailspin.SurveyAnswerService** est un service sans état qui récupère les réponses d’enquête ainsi qu’une analyse d’enquête. Le service utilise également le Stockage Blob Azure. Cette fonctionnalité a également été initialement implémentée dans le service *Tailspin.AnswerAnalysisService* dans l’application Service Fabric migrée. Tailspin a refactorisé la fonctionnalité d’origine dans ce service car il attend une charge moindre et veut utiliser moins d’instances pour conserver les ressources.

**Tailspin.SurveyAnalysisService** est un service sans état qui conserve les données de résumé des réponses d’enquête dans un cache Redis pour une récupération rapide. Ce service est appelé par le *Tailspin.SurveyResponseService* chaque fois qu’une enquête reçoit une réponse et les nouvelles données de réponse d’enquête sont fusionnées dans les données de synthèse. Ce service inclut les fonctionnalités restantes dans le service *Tailspin.SurveyAnalysisService* service à partir de l’application Service Fabric migrée.

## <a name="stateless-versus-stateful-services"></a>Services sans état par rapport aux services avec état

Azure Service Fabric prend en charge les modèles de programmation suivants :
* Le modèle exécutable invité permet d’empaqueter n’importe quel fichier exécutable comme un service et de le déployer dans un cluster Service Fabric. Service Fabric orchestre et gère l’exécution de l’exécutable invité.
* Le modèle de conteneur permet le déploiement des services dans les images de conteneur. Service Fabric prend en charge la création et la gestion des conteneurs sur les conteneurs de noyau Linux ainsi que les conteneurs Windows Server. 
* Le modèle de programmation de services fiables permet la création de services sans état ou avec état s’intègrant à toutes les fonctionnalités de la plateforme Service Fabric. Les services avec état permettent de stocker un état répliqué dans le cluster Service Fabric. Les services sans état ne le permettent pas.
* Le modèle de programmation des acteurs fiables permet la création de services qui implémentent le modèle d’acteur virtuel.

Tous les services de l’application Surveys sont des services sans état fiables, à l’exception du service *Tailspin.SurveyResponseService*. Ce service implémente un [ReliableConcurrentQueue][reliable-concurrent-queue] pour traiter les réponses d’enquête une fois reçues. Les réponses dans le ReliableConcurrentQueue sont enregistrées dans le Stockage Blob Azure et passées au *Tailspin.SurveyAnalysisService* pour analyse. Tailspin choisit un ReliableConcurrentQueue basé, car les réponses ne nécessitent pas un ordre premier entré, premier sorti (FIFO) strict fourni par une file d’attente comme Azure Service Bus. Un ReliableConcurrentQueue est également conçu pour fournir un débit élevé et une faible latence pour les opérations de mise en file d’attente et de retrait.

Notez que les opérations pour conserver les éléments retirés d’une file d’attente à partir d’un ReliableConcurrentQueue doivent idéalement être idempotente. Si une exception est levée pendant le traitement d’un élément à partir de la file d’attente, le même élément est susceptible d’être traité plusieurs fois. Dans l’application Surveys, l’opération de fusion des réponses d’enquête répond que le *Tailspin.SurveyAnalysisService* n’est pas idempotente car Tailspin a décidé que les données d’analyse d’enquête sont seulement une capture instantanée des données d’analyse et n’ont pas besoin d’être cohérentes. Les réponses d’enquête enregistrées dans le Stockage Blob Azure sont finalement cohérentes, de sorte que l’analyse finale de l’enquête peut toujours être recalculée correctement à partir de ces données.

## <a name="communication-framework"></a>Infrastructure de communication

Chaque service dans l’application Surveys communique à l’aide d’une API web RESTful. Les API RESTful offre les avantages suivants :
* Facilité d’utilisation : chaque service est généré à l’aide de ASP.Net Core MVC, qui prend nativement en charge la création d’API web.
* Sécurité : alors que chaque service ne requiert pas SSL, Tailspin peut demander à chaque service de le demander. 
* Contrôle de version : les clients peuvent être écrits et testés par rapport à une version spécifique d’une API web.

Les services dans l’application Survey peuvent utiliser le [proxy inverse][reverse-proxy] implémenté par Service Fabric. Le proxy inverse est un service qui s’exécute sur chaque nœud dans le cluster Service Fabric et qui permet la résolution des points de terminaison, les nouvelles tentatives automatiques et gère d’autres types d’échec de connexion. Pour utiliser le proxy inverse, chaque appel d’API RESTful vers un service spécifique est effectué à l’aide d’un port de proxy inverse prédéfini.  Par exemple, si le port de proxy inverse a été défini sur **19081**, un appel à *Tailspin.SurveyAnswerService* peut être effectué comme suit :

```csharp
static SurveyAnswerService()
{
    httpClient = new HttpClient
    {
        BaseAddress = new Uri("http://localhost:19081/Tailspin/SurveyAnswerService/")
    };
}
```
Pour activer le proxy inverse, spécifiez un port de proxy inverse lors de la création du cluster Service Fabric. Pour en savoir plus, consultez [Proxy inverse][reverse-proxy] dans Azure Service Fabric.

## <a name="performance-considerations"></a>Considérations relatives aux performances

Tailspin a créé les services ASP.NET Core pour *Tailspin.Web* et *Tailspin.Web.Surveys.Public* à l’aide de modèles Visual Studio. Par défaut, ces modèles incluent la journalisation dans la console. La journalisation dans la console peut être effectuée pendant le développement et le débogage, mais toute la journalisation dans la console doit être supprimée lorsque l’application est déployée en production.

> [!NOTE]
> Pour plus d’informations sur la configuration de la surveillance et du diagnostic des applications Service Fabric exécutées en production, consultez [Surveillance et diagnostics][monitoring-diagnostics] pour Azure Service Fabric.

Par exemple, les lignes suivantes dans *startup.cs* pour chaque service web du serveur frontal doivent être COMMENTÉES :

```csharp
// This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    //loggerFactory.AddConsole(Configuration.GetSection("Logging"));
    //loggerFactory.AddDebug();

    app.UseMvc();
}
```

> [!NOTE]
> Ces lignes peuvent être exclues de manière conditionnelle lorsque Visual Studio est défini sur «release » lors de la publication.

Enfin, lorsque Tailspin déploie l’application Tailspin en production, ils basculent Visual Studio en mode **release**.

## <a name="deployment-considerations"></a>Points à prendre en considération pour le déploiement

L’application Surveys refactorisée est composée de cinq services sans état et d’un service avec état, afin que la planification du cluster soit limitée à la détermination adéquate de la taille de la machine virtuelle et du nombre de nœuds. Dans le fichier *applicationmanifest.xml* décrivant le cluster, Tailspin définit l’attribut *InstanceCount* de la balise *StatelessService* sur -1 pour chaque service. La valeur -1 ordonne à Service Fabric de créer une instance du service sur chaque nœud du cluster.

> [!NOTE]
> Les services avec état nécessitent une étape supplémentaire de planification du nombre correct de partitions et de réplicas pour leurs données.

Tailspin déploie le cluster à l’aide du portail Azure. Le type de ressource du cluster Service Fabric déploie toute l’infrastructure nécessaire, y compris les groupes de machines virtuelles identiques et un équilibreur de charge. Les tailles recommandées des machines virtuelles sont affichées dans le portail Azure pendant le processus d’approvisionnement pour le cluster Service Fabric. Notez qu’étant donné que les machines virtuelles sont déployées dans un groupe de machines virtuelles identiques, elles peuvent être mises à l’échelle en fonction de la charge de l’utilisateur.

> [!NOTE]
> Comme indiqué précédemment, dans la version migrée de l’application Surveys, les deux serveurs web frontaux étaient auto-hébergés à l’aide d’ASP.Net Core et de Kestrel comme un serveur web. Alors que la version migrée de l’application Surveys n’utilise pas un proxy inverse, il est fortement recommandé d’en utiliser un, comme IIS, Nginx ou Apache. Pour plus d’informations, consultez [Introduction à l’implémentation du serveur web Kestrel dans ASP.NET Core][kestrel-intro].
> Dans l’application Surveys refactorisée, les deux serveurs web frontaux sont auto-hébergés à l’aide d’ASP.Net Core avec [WebListener][weblistener] comme un serveur web de sorte que le proxy inverse n’est pas nécessaire.

## <a name="next-steps"></a>étapes suivantes

Le code de l’application Surveys est disponible sur [GitHub][sample-code].

Si vous venez de démarrer avec [Azure Service Fabric][service-fabric], configurez d’abord votre environnement de développement, puis téléchargez la dernière version du [SDK Azure ][azure-sdk] et du [SDK Azure Service Fabric ][service-fabric-sdk]. Le SDK inclut le gestionnaire de clusters OneBox afin que vous puissiez déployer et tester l’application Surveys localement avec le débogage F5 complet.

<!-- links -->
[azure-sdk]: https://azure.microsoft.com/en-us/downloads/archive-net-downloads/
[container-scenarios]: /azure/service-fabric/service-fabric-containers-overview
[kestrel]: https://docs.microsoft.com/aspnet/core/fundamentals/servers/kestrel?tabs=aspnetcore2x
[kestrel-intro]: https://docs.microsoft.com/aspnet/core/fundamentals/servers/kestrel?tabs=aspnetcore1x
[migrate-from-cloud-services]: migrate-from-cloud-services.md
[monitoring-diagnostics]: /azure/service-fabric/service-fabric-diagnostics-overview
[reliable-concurrent-queue]: /azure/service-fabric/service-fabric-reliable-services-reliable-concurrent-queue
[reverse-proxy]: /azure/service-fabric/service-fabric-reverseproxy
[sample-code]: https://github.com/mspnp/cloud-services-to-service-fabric/tree/master/servicefabric-phase-2
[service-fabric]: /azure/service-fabric/service-fabric-get-started
[service-fabric-sdk]: /azure/service-fabric/service-fabric-get-started
[weblistener]: https://docs.microsoft.com/aspnet/core/fundamentals/servers/weblistener