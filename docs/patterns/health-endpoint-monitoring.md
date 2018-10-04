---
title: Surveillance de point de terminaison d’intégrité
description: Implémentez des contrôles fonctionnels dans une application à laquelle des outils externes peuvent accéder par le biais de points de terminaison exposés à intervalles réguliers.
keywords: modèle de conception
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- availability
- management-monitoring
- resiliency
ms.openlocfilehash: 22a4e47c4dd8dd3dd11a4238e859acbea49f9d1b
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428973"
---
# <a name="health-endpoint-monitoring-pattern"></a>Modèle Surveillance de point de terminaison

[!INCLUDE [header](../_includes/header.md)]

Implémentez des contrôles fonctionnels dans une application à laquelle des outils externes peuvent accéder par le biais de points de terminaison exposés à intervalles réguliers. Cela peut aider à vérifier que les applications et les services fonctionnent correctement.

## <a name="context-and-problem"></a>Contexte et problème

Nous vous recommandons d’analyser les applications web et services principaux pour être sûr qu’ils sont disponibles et fonctionnent correctement (d’ailleurs, il s’agit souvent d’une exigence professionnelle). Toutefois, il est plus difficile de surveiller les services exécutés dans le cloud que les services locaux. Par exemple, vous ne contrôlez pas totalement l’environnement d’hébergement, et les services dépendent généralement d’autres services fournis par des fournisseurs de plateformes ou autres.

De nombreux facteurs affectent les applications hébergées dans le cloud, tels que la latence du réseau, les performances et la disponibilité des systèmes de calcul et de stockage sous-jacents, et la bande passante réseau entre eux. Le service peut échouer entièrement ou partiellement en raison de l’un de ces facteurs. Ainsi, vous devez vérifier régulièrement que le service fonctionne correctement pour garantir le niveau de disponibilité exigé, qui peut faire partie de votre contrat de niveau de service (SLA).

## <a name="solution"></a>Solution

Implémentez la surveillance de l’intégrité en envoyant des demandes à un point de terminaison sur l’application. L’application doit effectuer les vérifications nécessaires et retourner une indication de son état.

Un contrôle de surveillance de l’intégrité combine en général deux facteurs :

- Les contrôles (le cas échéant) effectués par l’application ou le service en réponse à la demande envoyée au point de terminaison de vérification de l’intégrité.
- L’analyse des résultats par l’outil ou le framework qui effectue le contrôle de vérification de l’intégrité.

Le code de réponse indique l’état de l’application et éventuellement les composants ou services qu’elle utilise. Le contrôle de latence ou de temps de réponse est effectué par l’outil ou le framework de surveillance. La figure présente une vue d’ensemble du modèle.

![Vue d’ensemble du modèle](./_images/health-endpoint-monitoring-pattern.png)

Le code de surveillance de l’intégrité peut effectuer d’autres contrôles dans l’application :
- Vérification de la disponibilité ou du temps de réponse du stockage cloud ou d’une base de données.
- Vérification d’autres ressources ou services situés dans l’application, ou situés ailleurs mais utilisés par l’application.

Il existe des outils et services qui surveillent des applications web en soumettant une demande à un ensemble de points de terminaison configurable et en évaluant les résultats par rapport à un ensemble de règles configurables. Il est relativement facile de créer un point de terminaison de service dont la seule fonction est d’effectuer certains tests fonctionnels sur le système.

Voici quelques-uns des contrôles que les outils de surveillance peuvent effectuer :

- Validation du code de réponse. Par exemple, une réponse HTTP 200 (OK) indique que l’application a répondu sans erreur. Le système de surveillance peut également contrôler d’autres codes de réponse pour fournir des résultats plus complets.
- Vérification du contenu de la réponse pour détecter les erreurs, même quand le code d’état 200 (OK) est retourné. Cela permet de détecter les erreurs qui affectent uniquement une section de la page web retournée ou de la réponse du service. Vous pouvez par exemple vérifier le titre d’une page ou rechercher une expression spécifique qui indique que la page correcte a été retournée.
- Mesure du temps de réponse, qui indique une combinaison de la latence du réseau et de la durée nécessaire à l’application pour exécuter la demande. Une valeur croissante peut indiquer un problème au niveau de l’application ou du réseau.
- Vérification de ressources ou services situés en dehors de l’application, comme un réseau de distribution de contenu utilisé par l’application pour remettre du contenu à partir de caches globaux.
- Vérification de l’expiration des certificats SSL.
- Mesure du temps de réponse d’une recherche DNS pour l’URL de l’application, afin de mesurer la latence DNS et les échecs DNS.
- Validation de l’URL retournée par la recherche DNS, pour vérifier que les entrées sont correctes. Cela peut aider à éviter la redirection des demandes malveillantes suite à une attaque du serveur DNS.

Il est également utile, dans la mesure du possible, d’exécuter ces contrôles à partir de différents emplacements locaux ou hébergés pour mesurer et comparer les temps de réponse. Dans l’idéal, vous devez surveiller les applications à partir d’emplacements qui sont proches des clients, pour obtenir une vue précise des performances de chaque emplacement. En plus de fournir un mécanisme de vérification plus robuste, les résultats peuvent vous aider à choisir l’emplacement de déploiement de l’application&mdash;et s’il faut la déployer dans plusieurs centres de données.

Vous devez aussi exécuter des tests sur toutes les instances de service utilisées par les clients, pour vérifier que l’application fonctionne correctement pour tous les clients. Par exemple, si le stockage du client est réparti sur plusieurs comptes de stockage, le processus de surveillance doit tous les vérifier.

## <a name="issues-and-considerations"></a>Problèmes et considérations

Prenez en compte les points suivants lorsque vous choisissez comment implémenter ce modèle :

Comment valider la réponse. Par exemple, un seul code d’état 200 (OK) suffit-il pour vérifier que l’application fonctionne correctement ? Même si cela fournit la mesure la plus simple de la disponibilité des applications, et s’il s’agit de l’implémentation minimale de ce modèle, cela ne fournit que peu d’informations sur les opérations, les tendances et les éventuels problèmes à venir dans l’application.

   >  Vérifiez que l’application retourne correctement un code d’état 200 (OK) uniquement quand la ressource cible est détectée et traitée. Dans certains scénarios, par exemple quand vous utilisez une page maître pour héberger la page web cible, le serveur retourne un code d’état 200 (OK) au lieu d’un code 404 (Introuvable), même quand la page de contenu cible est introuvable.

Le nombre de points de terminaison à exposer pour une application. Une approche consiste à exposer au moins un point de terminaison pour les services de base utilisés par l’application, et un autre pour les services de priorité inférieure. Ceci permet d’affecter différents niveaux d’importance à chaque résultat de surveillance. Pour bénéficier d’une granularité de surveillance supplémentaire, vous pouvez exposer plusieurs points de terminaison, par exemple un pour chaque service de base. Ainsi, un contrôle de vérification de l’intégrité peut vérifier la base de données, le stockage et un service de géocodage externe utilisé par une application, chacun nécessitant un niveau de disponibilité et un temps de réponse différent. L’application peut toujours être saine si le service de géocodage ou une autre tâche en arrière-plan n’est pas disponible pendant quelques minutes.

S’il vaut mieux utiliser le même point de terminaison pour la surveillance et pour l’accès général, mais vers un chemin spécifique conçu pour les contrôles de vérification de l’intégrité, par exemple /HealthCheck/{GUID}/ sur le point de terminaison d’accès général. Ainsi, certains tests fonctionnels de l’application peuvent être exécutés par les outils de surveillance (tels que l’ajout de l’inscription d’un utilisateur, la connexion et la prise d’une commande test) tout en vérifiant que le point de terminaison d’accès général est disponible.

Le type d’informations à collecter dans le service en réponse aux demandes de surveillance, et comment retourner ces informations. La plupart des outils et frameworks existants examinent uniquement le code d’état HTTP retourné par le point de terminaison. Pour retourner et valider des informations supplémentaires, vous devrez peut-être créer un service ou un utilitaire de surveillance personnalisé.

La quantité d’informations à collecter. Un traitement excessif lors de la vérification peut surcharger l’application et avoir un impact sur d’autres utilisateurs. La durée nécessaire peut dépasser le délai d’expiration du système de surveillance, et l’application risque d’être marquée comme étant indisponible. La plupart des applications incluent une instrumentation telle que des gestionnaires d’erreurs et des compteurs de performances qui enregistrent les performances et des informations d’erreurs détaillées dans des journaux. Ces informations peuvent suffire et éviter d’avoir à retourner des informations supplémentaires à partir d’un contrôle de vérification de l’intégrité.

Mise en cache de l’état du point de terminaison. L’exécution trop fréquente d’un contrôle d’intégrité peut être couteuse. Si l’état d’intégrité est indiqué sur un tableau de bord (par exemple), vous ne souhaitez pas que chaque requête faite depuis le tableau de bord déclenche un contrôle d’intégrité. Vérifiez plutôt l’intégrité du système périodiquement et mettez l’état en cache. Exposez un point de terminaison qui retourne l’état mis en cache.

Comment configurer la sécurité des points de terminaison de surveillance pour les protéger contre l’accès public, qui risque d’exposer l’application à des attaques malveillantes, d’exposer des informations sensibles ou de favoriser les attaques par déni de service (DoS). En général, cette configuration de la sécurité doit être effectuée dans l’application, pour pouvoir être mise à jour facilement sans avoir à redémarrer l’application. Vous pouvez appliquer une ou plusieurs des techniques suivantes :

- Sécurisez le point de terminaison en exigeant l’authentification. Pour cela, utilisez une clé de sécurité d’authentification dans l’en-tête de demande ou passez les informations d’identification avec la demande, sous réserve que l’outil ou le service de surveillance prenne en charge l’authentification.

  - Utilisez un point de terminaison obscur ou masqué. Par exemple, exposez le point de terminaison sur une adresse IP différente de celle utilisée par l’URL d’application par défaut, configurez le point de terminaison sur un port HTTP non standard et/ou utilisez un chemin complexe à la page de test. Vous pouvez généralement spécifier des ports et des adresses de point de terminaison supplémentaires dans la configuration de l’application, et ajouter des entrées pour ces points de terminaison au serveur DNS si nécessaire pour éviter d’avoir à spécifier l’adresse IP directement.

  - Exposez une méthode sur un point de terminaison qui accepte un paramètre tel qu’une valeur de clé ou une valeur de mode d’opération. En fonction de la valeur fournie pour ce paramètre, quand une demande est reçue, le code peut effectuer un test ou un ensemble de tests spécifique, ou retourner un code d’erreur 404 (Introuvable) si la valeur du paramètre n’est pas reconnue. Vous pouvez définir les valeurs de paramètre reconnues dans la configuration de l’application.

     >  Les attaques DoS auront probablement un impact moindre sur un point de terminaison distinct qui effectue des tests fonctionnels de base sans compromettre le fonctionnement de l’application. Dans l’idéal, évitez d’utiliser un test susceptible d’exposer des informations sensibles. Si vous devez retourner des informations qui peuvent être utiles à un agresseur, réfléchissez à la façon dont vous protégerez le point de terminaison et les données contre tout accès non autorisé. Dans ce cas, s’appuyer uniquement sur l’obscurité n’est pas suffisant. Vous devez également envisager l’utilisation d’une connexion HTTPS et le chiffrement des données sensibles, mais cela augmentera la charge sur le serveur.

- Comment accéder à un point de terminaison sécurisé à l’aide de l’authentification. Les outils et frameworks ne peuvent pas tous être configurés pour inclure des informations d’identification avec la demande de vérification de l’intégrité. Par exemple, les fonctionnalités de vérification de l’intégrité intégrées à Microsoft Azure ne peuvent pas fournir d’informations d’identification pour l’authentification. [Pingdom](https://www.pingdom.com/), [Panopta](https://www.panopta.com/), [NewRelic](https://newrelic.com/) et [Statuscake](https://www.statuscake.com/) constituent des solutions alternatives.

- Comment vérifier que les performances de l’agent de surveillance sont correctes. Une approche consiste à exposer un point de terminaison qui retourne simplement une valeur à partir de la configuration de l’application ou une valeur aléatoire qui peut être utilisée pour tester l’agent.

   >  Vérifiez également que le système de surveillance exécute des contrôles sur lui-même, par exemple un test intégré et un test automatique, pour éviter qu’il n’émette de faux résultats positifs.

## <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle

Ce modèle est utile dans les situations suivantes :
- Surveillance d’applications web et de sites web pour vérifier la disponibilité.
- Surveillance d’applications web et de sites web pour vérifier le bon fonctionnement.
- Surveillance de services de couche intermédiaire ou de services partagés pour détecter et isoler une défaillance susceptible de perturber d’autres applications.
- En complément de l’instrumentation existante dans l’application, telle que les compteurs de performances et les gestionnaires d’erreurs. La vérification de l’intégrité ne se substitue pas à l’obligation de journalisation et d’audit dans l’application. L’instrumentation peut fournir des informations précieuses pour un framework existant qui surveille des compteurs et des journaux d’erreurs pour détecter les défaillances ou autres problèmes. Toutefois, elle ne peut pas fournir d’informations si l’application n’est pas disponible.

## <a name="example"></a>Exemples

Les exemples de code suivants, tirés de la classe `HealthCheckController` (un exemple qui illustre ce modèle est disponible sur [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/health-endpoint-monitoring)), montrent comment exposer un point de terminaison pour exécuter différents contrôles d’intégrité.

La méthode `CoreServices`, illustrée ci-dessous en langage C#, effectue une série de contrôles sur des services utilisés dans l’application. Si tous les tests s’exécutent sans erreur, la méthode retourne un code d’état 200 (OK). Si l’un des tests lève une exception, la méthode retourne un code d’état 500 (Erreur interne). La méthode peut éventuellement retourner des informations supplémentaires quand une erreur se produit, si le framework ou l’outil de surveillance peut les exploiter.

```csharp
public ActionResult CoreServices()
{
  try
  {
    // Run a simple check to ensure the database is available.
    DataStore.Instance.CoreHealthCheck();

    // Run a simple check on our external service.
    MyExternalService.Instance.CoreHealthCheck();
  }
  catch (Exception ex)
  {
    Trace.TraceError("Exception in basic health check: {0}", ex.Message);

    // This can optionally return different status codes based on the exception.
    // Optionally it could return more details about the exception.
    // The additional information could be used by administrators who access the
    // endpoint with a browser, or using a ping utility that can display the
    // additional information.
    return new HttpStatusCodeResult((int)HttpStatusCode.InternalServerError);
  }
  return new HttpStatusCodeResult((int)HttpStatusCode.OK);
}
```
La méthode `ObscurePath` montre comment lire un chemin à partir de la configuration d’application et l’utiliser comme point de terminaison pour les tests. Cet exemple, en C#, montre également comment accepter un ID comme paramètre et l’utiliser pour vérifier si les requêtes sont valides.

```csharp
public ActionResult ObscurePath(string id)
{
  // The id could be used as a simple way to obscure or hide the endpoint.
  // The id to match could be retrieved from configuration and, if matched,
  // perform a specific set of tests and return the result. If not matched it
  // could return a 404 (Not Found) status.

  // The obscure path can be set through configuration to hide the endpoint.
  var hiddenPathKey = CloudConfigurationManager.GetSetting("Test.ObscurePath");

  // If the value passed does not match that in configuration, return 404 (Not Found).
  if (!string.Equals(id, hiddenPathKey))
  {
    return new HttpStatusCodeResult((int)HttpStatusCode.NotFound);
  }

  // Else continue and run the tests...
  // Return results from the core services test.
  return this.CoreServices();
}
```

La méthode `TestResponseFromConfig` montre comment exposer un point de terminaison qui effectue un contrôle pour une valeur de paramètre de configuration spécifiée.

```csharp
public ActionResult TestResponseFromConfig()
{
  // Health check that returns a response code set in configuration for testing.
  var returnStatusCodeSetting = CloudConfigurationManager.GetSetting(
                                                          "Test.ReturnStatusCode");

  int returnStatusCode;

  if (!int.TryParse(returnStatusCodeSetting, out returnStatusCode))
  {
    returnStatusCode = (int)HttpStatusCode.OK;
  }

  return new HttpStatusCodeResult(returnStatusCode);
}
```
## <a name="monitoring-endpoints-in-azure-hosted-applications"></a>Surveillance des points de terminaison dans les applications hébergées Azure

Voici quelques options pour la surveillance des points de terminaison dans les applications Azure :

- Utiliser les fonctionnalités de surveillance intégrées d’Azure

- Utiliser un service ou un framework tiers tel que Microsoft System Center Operations Manager

- Créer un utilitaire ou un service personnalisé qui s’exécute sur votre propre serveur ou sur un serveur hébergé

   >  Bien qu’Azure fournisse un ensemble relativement complet d’options de surveillance, vous pouvez utiliser des services et des outils supplémentaires pour obtenir d’autres informations. Azure Management Services fournit un mécanisme de surveillance intégré pour les règles d’alerte. La section Alertes de la page des services de gestion dans le portail Azure vous permet de configurer jusqu’à dix règles d’alerte par abonnement pour vos services. Ces règles spécifient une condition et une valeur de seuil pour un service, telle que la charge de l’UC ou le nombre de demandes ou d’erreurs par seconde, et le service peut envoyer automatiquement des notifications par e-mail aux adresses que vous définissez dans chaque règle.

Les conditions que vous pouvez surveiller varient en fonction du mécanisme d’hébergement que vous choisissez pour votre application (par exemple, sites web, services cloud, machines virtuelles ou services mobiles), mais tous ces mécanismes incluent la possibilité de créer une règle d’alerte qui utilise un point de terminaison web que vous spécifiez dans les paramètres de votre service. Ce point de terminaison doit répondre en temps opportun pour que le système d’alerte puisse détecter si l’application fonctionne correctement.

>  En savoir plus sur la [création de notifications d’alerte][portal-alerts].

Si vous hébergez votre application dans des machines virtuelles ou des rôles de travail et web de Services cloud Azure, vous pouvez tirer parti de l’un des services intégrés à Azure appelé Traffic Manager. Traffic Manager est un service de routage et d’équilibrage de charge qui peut distribuer les demandes à des instances spécifiques de votre application hébergée par les services cloud en fonction d’une gamme de règles et de paramètres.

En plus de router les demandes, Traffic Manager exécute une requête ping sur une URL, un port et un chemin relatif que vous spécifiez régulièrement pour déterminer, parmi les instances de l’application définies dans ses règles, lesquelles sont actives et répondent aux requêtes. S’il détecte un code d’état 200 (OK), Traffic Manager marque l’application comme étant disponible. S’il détecte un autre code d’état, il marque l’application comme étant hors connexion. Vous pouvez afficher l’état dans la console de Traffic Manager et configurer la règle pour rediriger les demandes vers d’autres instances de l’application qui répondent.

Toutefois, Traffic Manager n’attend que dix secondes pour recevoir une réponse à partir de l’URL de surveillance. Ainsi, vous devez vérifier que votre code de vérification de l’intégrité s’exécute pendant ce laps de temps, en prenant en compte la latence du réseau pour l’aller-retour entre Traffic Manager et votre application.

>  En savoir plus sur l’utilisation de [Traffic Manager pour surveiller vos applications](https://azure.microsoft.com/documentation/services/traffic-manager/). Traffic Manager est également abordé dans [Multiple Datacenter Deployment Guidance (Recommandations pour le déploiement dans plusieurs centres de données)](https://msdn.microsoft.com/library/dn589779.aspx).

## <a name="related-guidance"></a>Aide connexe

Les conseils suivants peuvent être utiles lors de l’implémentation de ce modèle :
- [Instrumentation and Telemetry Guidance (Recommandations relatives à l’instrumentation et la télémétrie)](https://msdn.microsoft.com/library/dn589775.aspx). La vérification de l’intégrité des services et des composants s’effectue généralement par l’interrogation, mais il est également utile de disposer d’informations pour surveiller les performances des applications et détecter les événements qui se produisent au moment de l’exécution. Ces données peuvent être retransmises aux outils de surveillance en guise d’informations supplémentaires pour la surveillance de l’intégrité. L’article Instrumentation and Telemetry Guidance explore la collecte des informations de diagnostics à distance par l’instrumentation dans les applications.
- [Recevoir les notifications d’alerte][portal-alerts].
- Ce modèle comprend un [exemple d’application](https://github.com/mspnp/cloud-design-patterns/tree/master/health-endpoint-monitoring) téléchargeable.

[portal-alerts]: https://azure.microsoft.com/documentation/articles/insights-receive-alert-notifications/
