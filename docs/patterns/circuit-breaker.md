---
title: Modèle Disjoncteur
titleSuffix: Cloud Design Patterns
description: Gérer les erreurs dont la résolution peut prendre un certain temps lors de la connexion à une ressource ou à un service distant.
keywords: modèle de conception
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 7cc84b3c14ea277aa82643f3141f0693ec702a49
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58249314"
---
# <a name="circuit-breaker-pattern"></a>Modèle Disjoncteur

Gérer les erreurs pour lesquelles la récupération peut prendre un certain temps lors de la connexion à une ressource ou à un service distant. Cela peut améliorer la stabilité et la résilience d’une application.

## <a name="context-and-problem"></a>Contexte et problème

Dans un environnement distribué, les appels à des ressources et services distants peuvent échouer en raison d’erreurs temporaires telles que des connexions réseau lentes, des expirations de délais d’attente, ou encore une indisponibilité temporaire ou une sollicitation trop importante des ressources. Ces erreurs disparaissent en général automatiquement après un court laps de temps, et une application cloud fiable doit être prête à les gérer à l’aide d’une stratégie telle que le [modèle Nouvelle tentative](./retry.md).

Toutefois, dans certaines situations les erreurs sont dues à des événements imprévus et peuvent durer beaucoup plus longtemps. Ces erreurs peuvent aller d’une perte partielle de connectivité à la défaillance complète d’un service. Dans ces cas-là, il ne sert à rien qu’une application effectue de nouvelles tentatives qui sont vouées à l’échec. Au lieu de cela, elle doit rapidement reconnaître que l’opération a échoué et traiter cet échec en conséquence.

De plus, si un service est très occupé, un échec dans une partie du système peut entraîner des défaillances en cascade. Par exemple, une opération qui appelle un service peut être configurée pour implémenter un délai d’attente et répondre avec un message d’échec si le service ne répond pas pendant cette période. Toutefois, cette stratégie peut entraîner le blocage de plusieurs demandes simultanées à la même opération jusqu’à ce que le délai d’attente ait expiré. Ces demandes bloquées peuvent contenir des ressources système critiques telles que de la mémoire, des threads, des connexions de base de données, et ainsi de suite. Ces ressources peuvent ainsi être épuisées, entraînant l’échec d’autres parties du système qui doivent utiliser les mêmes ressources. Dans ces situations, il serait préférable que l’opération échoue immédiatement et que l’appel du service soit tenté uniquement s’il est susceptible de réussir. Notez que la définition d’un délai d’attente plus court peut aider à résoudre ce problème, mais il ne faut pas qu’il soit si court que l’opération échoue la plupart du temps, même si la demande au service aboutirait finalement.

## <a name="solution"></a>Solution

Le modèle Disjoncteur, popularisé par Michael Nygard dans son livre, [Release it!](https://pragprog.com/book/mnee/release-it) (Libérez-le !), peut empêcher qu’une application ne tente d’exécuter à répétition une opération qui échouera probablement. Cela lui permet de continuer sans attendre que l’erreur soit corrigée ou sans gaspiller des cycles de processeur pendant qu’elle détermine qu’il s’agit d’une erreur de longue durée. Le modèle Disjoncteur permet également à une application de détecter si l’erreur a été corrigée. Si le problème semble avoir été résolu, l’application peut essayer d’appeler l’opération.

> L’objectif du modèle Disjoncteur est différent de celui du modèle Nouvelle tentative. Le modèle Nouvelle tentative permet à une application de retenter une opération en partant du principe qu’elle finira par réussir. Le modèle Disjoncteur empêche une application d’effectuer une opération qui échouera probablement. Une application peut combiner ces deux modèles en utilisant le modèle Nouvelle tentative pour appeler une opération par le biais d’un disjoncteur. Toutefois, la logique de nouvelle tentative doit être sensible aux exceptions retournées par le disjoncteur, et abandonner les nouvelles tentatives si le disjoncteur indique qu’une erreur n’est pas temporaire.

Un disjoncteur agit comme un proxy pour les opérations qui risquent d’échouer. Le proxy doit contrôler le nombre d’échecs qui se sont produits récemment, et utiliser ces informations pour décider s’il faut autoriser la poursuite de l’opération ou simplement retourner une exception immédiatement.

Le proxy peut être implémenté en tant que machine d’état avec les états suivants qui simulent la fonctionnalité d’un disjoncteur électrique :

- **Fermé** : La requête de l’application est routée vers l’opération. Le proxy tient à jour un décompte du nombre d’échecs récents, et si l’appel à l’opération n’aboutit pas, le proxy incrémente ce nombre. Si le nombre d’échecs récents dépasse un seuil spécifié pendant une période donnée, le proxy est placé à l’état **Ouvert**. À ce stade, le proxy démarre un minuteur de délai d’attente, et quand ce minuteur expire le proxy est placé à l’état **Demi-ouvert**.

    > Le minuteur de délai d’attente a pour but de donner au système suffisamment de temps pour résoudre le problème qui a provoqué l’échec, avant d’autoriser l’application à tenter une nouvelle fois d’effectuer l’opération.

- **Ouverte** : La requête de l’application est immédiatement un échec. Une exception est retournée à l’application.

- **Demi-ouvert** : Un nombre limité de requêtes provenant de l’application est autorisé à passer et à appeler l’opération. Si ces demandes réussissent, on considère que l’erreur ayant provoqué l’échec a été corrigée et le disjoncteur passe à l’état **Fermé** (le compteur d’échecs est réinitialisé). Si une demande échoue, le disjoncteur considère que l’erreur est toujours présente. Il repasse donc à l’état **Ouvert** et redémarre le minuteur de délai d’attente pour accorder au système un délai supplémentaire pour récupérer suite à la défaillance.

    > L’état **Demi-ouvert** est utile pour empêcher qu’un service en train de récupérer ne soit submergé soudainement de demandes. Quand un service est en cours de récupération, il peut être capable de prendre en charge un volume limité de demandes jusqu’à ce que la récupération soit terminée, mais pendant que la récupération est en cours un flux de travail peut provoquer un nouvel échec ou l’expiration du délai de service.

![États du disjoncteur](./_images/circuit-breaker-diagram.png)

Dans la figure, le compteur d’échecs utilisé par l’état **Fermé** est temporel. Il est réinitialisé automatiquement à intervalles réguliers. Cela empêche que le disjoncteur ne passe à l’état **Ouvert** s’il rencontre des défaillances occasionnelles. Le seuil d’échec qui fait basculer le disjoncteur à l’état **Ouvert** est atteint uniquement quand un nombre spécifié d’échecs se sont produits pendant un intervalle spécifié. Le compteur utilisé par l’état **Demi-ouvert** enregistre le nombre de tentatives d’appel de l’opération qui ont réussi. Le disjoncteur repasse à l’état **Fermé** une fois qu’un nombre spécifié d’appels d’opérations consécutifs ont abouti. Si un appel échoue, le disjoncteur bascule immédiatement à l’état **Ouvert** et le compteur de réussites est réinitialisé la prochaine fois qu’il bascule à l’état **Demi-ouvert**.

> Le mode de récupération du système est géré en externe, éventuellement par la restauration ou le redémarrage d’un composant défectueux ou la réparation d’une connexion réseau.

Le modèle Disjoncteur assure la stabilité pendant que le système récupère après un échec, et il réduit l’impact sur les performances. Il peut aider à maintenir le temps de réponse du système en rejetant rapidement une demande pour une opération qui échouera probablement, plutôt que d’attendre que l’opération expire ou ne retourne jamais. Si le disjoncteur déclenche un événement chaque fois qu’il change d’état, cette information peut être utilisée pour surveiller l’intégrité de la partie du système protégée par le disjoncteur, ou pour alerter l’administrateur quand un disjoncteur bascule à l’état **Ouvert**.

Le modèle est personnalisable et peut être adapté en fonction du type de défaillance possible. Par exemple, vous pouvez appliquer un minuteur de délai d’attente croissant à un disjoncteur. Vous pouvez placer initialement le disjoncteur à l’état **Ouvert** pendant quelques secondes et, si l’échec n’a pas été résolu, augmenter le délai de quelques minutes, et ainsi de suite. Dans certains cas, plutôt que de faire en sorte que l’état **Ouvert** retourne un échec et déclenche une exception, il peut être utile de retourner une valeur par défaut qui est significative pour l’application.

## <a name="issues-and-considerations"></a>Problèmes et considérations

Prenez en compte les points suivants quand vous choisissez comment implémenter ce modèle :

**Gestion des exceptions**. Une application appelant une opération par le biais d’un disjoncteur doit pouvoir gérer les exceptions déclenchées si l’opération n’est pas disponible. Le traitement des exceptions est propre à l’application. Par exemple, une application peut temporairement dégrader ses fonctionnalités, appeler une opération de remplacement pour tenter d’effectuer la même tâche ou obtenir les mêmes données, ou signaler l’exception à l’utilisateur et lui demander de réessayer ultérieurement.

**Types d’exceptions**. Une demande peut échouer pour de nombreuses raisons, certaines d’entre elles pouvant indiquer un type de défaillance plus grave que d’autres. Par exemple, une demande peut échouer car un service distant s’est bloqué et plusieurs minutes seront nécessaires à sa récupération, ou à cause d’une expiration de délai d’attente due à une surcharge temporaire du service. Un disjoncteur peut être en mesure d’examiner les types d’exceptions qui se produisent et d’ajuster sa stratégie d’après la nature de ces exceptions. Par exemple, vous pouvez faire en sorte qu’un plus grand nombre d’exceptions de délai d’attente soit nécessaires pour faire basculer le disjoncteur à l’état **Ouvert**, par rapport au nombre de défaillances dues à une indisponibilité totale du service.

**Journalisation**. Un disjoncteur doit consigner toutes les demandes ayant échoué (et éventuellement celles ayant réussi) pour permettre à un administrateur de surveiller l’intégrité de l’opération.

**Capacité de restauration**. Vous devez configurer le disjoncteur pour qu’il corresponde au modèle de récupération probable de l’opération qu’il protège. Par exemple, si le disjoncteur reste à l’état **Ouvert** pendant une longue période, il peut lever des exceptions même si la cause de l’échec a disparu. De même, un disjoncteur peut fluctuer et réduire le temps de réponse des applications s’il bascule trop rapidement de l’état **Ouvert** à l’état **Demi-ouvert**.

**Test des opérations ayant échoué**. À l’état **Ouvert**, plutôt que d’utiliser un minuteur pour déterminer quand basculer à l’état **Demi-ouvert**, un disjoncteur peut exécuter régulièrement un test ping sur la ressource ou le service distant pour déterminer s’il ou elle est de nouveau disponible. Cette commande ping peut prendre la forme d’une tentative d’appel d’une opération qui a précédemment échoué. Vous pourriez aussi utiliser une opération spéciale fournie par le service distant spécifiquement pour tester l’intégrité du service, comme indiqué par le [Modèle de surveillance de point de terminaison d’intégrité](./health-endpoint-monitoring.md).

**Remplacement manuel**. Dans un système où le temps de récupération pour une opération ayant échoué est très variable, il est préférable de fournir une option de réinitialisation manuelle qui permet à un administrateur de fermer un disjoncteur (et de réinitialiser le compteur d’échecs). De même, un administrateur peut forcer un disjoncteur à l’état **Ouvert** (et redémarrer le minuteur de délai d’attente) si l’opération protégée par le disjoncteur est temporairement indisponible.

**Accès concurrentiel**. Le même disjoncteur peut être sollicité par un grand nombre d’instances simultanées d’une application. L’implémentation ne doit pas bloquer les demandes simultanées, ni ajouter une surcharge excessive à chaque appel à une opération.

**Différenciation des ressources**. Soyez prudent lors de l’utilisation d’un disjoncteur unique pour un type de ressource s’il peut y avoir plusieurs fournisseurs indépendants sous-jacents. Par exemple, dans un magasin de données qui contient plusieurs partitions, l’une d’entre elles peut être entièrement accessible alors qu’une autre rencontre un problème temporaire. Si les réponses aux erreurs dans ces scénarios sont fusionnées, une application risque de tenter d’accéder à certaines partitions même quand l’échec est très probable, tandis que l’accès à d’autres partitions risque d’être bloqué alors qu’il a toutes les chances de réussir.

**Disjonctage accéléré**. Parfois, une réponse d’échec peut contenir suffisamment d’informations pour que le disjoncteur bascule immédiatement et le demeure pendant un laps de temps minimum. Par exemple, la réponse d’erreur d’une ressource partagée qui est surchargée peut indiquer qu’une nouvelle tentative immédiate n’est pas recommandée, et que l’application doit plutôt réessayer dans quelques minutes.

> [!NOTE]
> Un service peut retourner l’erreur HTTP 429 (Demandes trop nombreuses) s’il limite le client, ou l’erreur HTTP 503 (Service indisponible) si le service n’est pas disponible actuellement. La réponse peut inclure des informations supplémentaires, telles que la durée prévue du retard.

**Relecture des demandes ayant échoué**. À l’état **Ouvert**, au lieu de simplement échouer rapidement, un disjoncteur peut aussi enregistrer les détails de chaque demande dans un journal, et faire en sorte que ces demandes soient relues quand la ressource ou le service distant redevient disponible.

**Délais d’attente inappropriés sur les services externes**. Il est possible qu’un disjoncteur ne puisse pas protéger entièrement les applications contre les opérations qui échouent dans des services externes configurés avec un long délai d’attente. Si le délai d’attente est trop long, un thread exécutant un disjoncteur peut être bloqué pendant une période prolongée avant que le disjoncteur n’indique que l’opération a échoué. Pendant ce temps, de nombreuses autres instances peuvent également tenter d’appeler le service par le biais du disjoncteur, et occuper un nombre important de threads avant que toutes n’échouent.

## <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle

Utilisez ce modèle :

- Pour empêcher qu’une application n’essaie d’appeler un service distant ou d’accéder à une ressource partagée, si cette opération a de grandes chances d’échouer.

Ce modèle n’est pas recommandé :

- Pour la gestion de l’accès aux ressources privées locales dans une application, comme la structure de données en mémoire. Dans cet environnement, l’utilisation d’un disjoncteur ajouterait une surcharge à votre système.
- Comme substitut pour la gestion des exceptions dans la logique métier de vos applications.

## <a name="example"></a>Exemples

Dans une application web, plusieurs pages sont remplies avec des données extraites à partir d’un service externe. Si le système implémente une mise en cache minimale, la plupart des accès à ces pages provoquent un aller-retour vers le service. Les connexions de l’application web au service peuvent être configurées avec un délai d’expiration (en général 60 secondes) et, si le service ne répond pas dans le délai imparti, la logique dans chaque page web considère que le service est indisponible et lève une exception.

Toutefois, si le service échoue et que le système est très occupé, les utilisateurs peuvent être contraints d’attendre jusqu’à 60 secondes avant qu’une exception ne se produise. Finalement, des ressources telles que de la mémoire, des connexions et des threads risquent d’être épuisées, empêchant d’autres utilisateurs de se connecter au système même s’ils n’accèdent pas à des pages qui extraient des données à partir du service.

La montée en charge du système par l’ajout de serveurs web et l’implémentation de l’équilibrage de charge peut retarder l’épuisement des ressources, mais elle ne résout pas le problème car les demandes des utilisateurs n’auront toujours pas de réponse et tous les serveurs web risquent quand même d’être à cours de ressources.

Encapsuler la logique qui se connecte au service et extrait les données dans un disjoncteur pourrait aider à résoudre ce problème et à gérer l’échec du service de façon plus élégante. Les demandes des utilisateurs échoueront quand même, mais plus rapidement, et les ressources ne seront pas bloquées.

La classe `CircuitBreaker` tient à jour les informations d’état concernant un disjoncteur dans un objet qui implémente l’interface `ICircuitBreakerStateStore` illustrée dans le code suivant.

```csharp
interface ICircuitBreakerStateStore
{
  CircuitBreakerStateEnum State { get; }

  Exception LastException { get; }

  DateTime LastStateChangedDateUtc { get; }

  void Trip(Exception ex);

  void Reset();

  void HalfOpen();

  bool IsClosed { get; }
}
```

La propriété `State` indique l’état actuel du disjoncteur et a la valeur **Open**, **HalfOpen** ou **Closed** telle que définie par l’énumération `CircuitBreakerStateEnum`. La propriété `IsClosed` doit être true si le disjoncteur est fermé, mais false s’il est ouvert ou à demi ouvert. La méthode `Trip` fait basculer le disjoncteur à l’état ouvert et enregistre l’exception qui a provoqué le changement d’état, ainsi que la date et l’heure de l’exception. Les propriétés `LastException` et `LastStateChangedDateUtc` retournent ces informations. La méthode `Reset` ferme le disjoncteur et la méthode `HalfOpen` le fait passer à l’état demi-ouvert.

La classe `InMemoryCircuitBreakerStateStore` dans l’exemple contient une implémentation de l’interface `ICircuitBreakerStateStore`. La classe `CircuitBreaker` crée une instance de cette classe pour contenir l’état du disjoncteur.

La méthode `ExecuteAction` dans la classe `CircuitBreaker` encapsule une opération, spécifiée comme délégué `Action`. Si le disjoncteur est fermé, `ExecuteAction` appelle le délégué `Action`. Si l’opération échoue, un gestionnaire d’exceptions appelle `TrackException`, qui fait passer le disjoncteur à l’état ouvert. L’exemple de code suivant met en évidence ce flux.

```csharp
public class CircuitBreaker
{
  private readonly ICircuitBreakerStateStore stateStore =
    CircuitBreakerStateStoreFactory.GetCircuitBreakerStateStore();

  private readonly object halfOpenSyncObject = new object ();
  ...
  public bool IsClosed { get { return stateStore.IsClosed; } }

  public bool IsOpen { get { return !IsClosed; } }

  public void ExecuteAction(Action action)
  {
    ...
    if (IsOpen)
    {
      // The circuit breaker is Open.
      ... (see code sample below for details)
    }

    // The circuit breaker is Closed, execute the action.
    try
    {
      action();
    }
    catch (Exception ex)
    {
      // If an exception still occurs here, simply
      // retrip the breaker immediately.
      this.TrackException(ex);

      // Throw the exception so that the caller can tell
      // the type of exception that was thrown.
      throw;
    }
  }

  private void TrackException(Exception ex)
  {
    // For simplicity in this example, open the circuit breaker on the first exception.
    // In reality this would be more complex. A certain type of exception, such as one
    // that indicates a service is offline, might trip the circuit breaker immediately.
    // Alternatively it might count exceptions locally or across multiple instances and
    // use this value over time, or the exception/success ratio based on the exception
    // types, to open the circuit breaker.
    this.stateStore.Trip(ex);
  }
}
```

L’exemple suivant montre le code (omis dans l’exemple précédent) qui est exécuté si le disjoncteur n’est pas fermé. Il vérifie d’abord si le disjoncteur est ouvert depuis plus longtemps que la durée spécifiée par le champ `OpenToHalfOpenWaitTime` local dans la classe `CircuitBreaker`. Si c’est le cas, la méthode `ExecuteAction` fait passer le disjoncteur à l’état demi-ouvert, puis tente d’effectuer l’opération spécifiée par le délégué `Action`.

Si l’opération réussit, le disjoncteur est réinitialisé à l’état fermé. Si l’opération échoue, il est rebasculé à l’état ouvert et l’heure de l’exception est mise à jour pour que le disjoncteur attende pendant un laps de temps supplémentaire avant de réessayer d’exécuter l’opération.

Si le disjoncteur est ouvert depuis un laps de temps inférieur à la valeur `OpenToHalfOpenWaitTime`, la méthode `ExecuteAction` lève simplement une exception `CircuitBreakerOpenException` et retourne l’erreur qui a fait basculer le disjoncteur à l’état ouvert.

Cet exemple utilise aussi un verrou pour empêcher que le disjoncteur n’essaie d’effectuer des appels simultanés à l’opération pendant qu’il est à demi ouvert. Une tentative simultanée d’appel de l’opération est traitée comme si le disjoncteur était ouvert, et échoue avec une exception comme décrit plus loin.

```csharp
    ...
    if (IsOpen)
    {
      // The circuit breaker is Open. Check if the Open timeout has expired.
      // If it has, set the state to HalfOpen. Another approach might be to
      // check for the HalfOpen state that had be set by some other operation.
      if (stateStore.LastStateChangedDateUtc + OpenToHalfOpenWaitTime < DateTime.UtcNow)
      {
        // The Open timeout has expired. Allow one operation to execute. Note that, in
        // this example, the circuit breaker is set to HalfOpen after being
        // in the Open state for some period of time. An alternative would be to set
        // this using some other approach such as a timer, test method, manually, and
        // so on, and check the state here to determine how to handle execution
        // of the action.
        // Limit the number of threads to be executed when the breaker is HalfOpen.
        // An alternative would be to use a more complex approach to determine which
        // threads or how many are allowed to execute, or to execute a simple test
        // method instead.
        bool lockTaken = false;
        try
        {
          Monitor.TryEnter(halfOpenSyncObject, ref lockTaken);
          if (lockTaken)
          {
            // Set the circuit breaker state to HalfOpen.
            stateStore.HalfOpen();

            // Attempt the operation.
            action();

            // If this action succeeds, reset the state and allow other operations.
            // In reality, instead of immediately returning to the Closed state, a counter
            // here would record the number of successful operations and return the
            // circuit breaker to the Closed state only after a specified number succeed.
            this.stateStore.Reset();
            return;
          }
        }
        catch (Exception ex)
        {
          // If there's still an exception, trip the breaker again immediately.
          this.stateStore.Trip(ex);

          // Throw the exception so that the caller knows which exception occurred.
          throw;
        }
        finally
        {
          if (lockTaken)
          {
            Monitor.Exit(halfOpenSyncObject);
          }
        }
      }
      // The Open timeout hasn't yet expired. Throw a CircuitBreakerOpen exception to
      // inform the caller that the call was not actually attempted,
      // and return the most recent exception received.
      throw new CircuitBreakerOpenException(stateStore.LastException);
    }
    ...
```

Pour utiliser un objet `CircuitBreaker` pour protéger une opération, une application crée une instance de la classe `CircuitBreaker` et appelle la méthode `ExecuteAction`, en spécifiant l’opération à effectuer en tant que paramètre. L’application doit être prête à intercepter l’exception `CircuitBreakerOpenException` si l’opération échoue car le disjoncteur est ouvert. Le code suivant montre un exemple :

```csharp
var breaker = new CircuitBreaker();

try
{
  breaker.ExecuteAction(() =>
  {
    // Operation protected by the circuit breaker.
    ...
  });
}
catch (CircuitBreakerOpenException ex)
{
  // Perform some different action when the breaker is open.
  // Last exception details are in the inner exception.
  ...
}
catch (Exception ex)
{
  ...
}
```

## <a name="related-patterns-and-guidance"></a>Conseils et modèles connexes

Les modèles suivants peuvent également être utiles lors de l’implémentation de ce modèle :

- [Modèle Nouvelle tentative](./retry.md). Décrit comment une application peut gérer les défaillances temporaires anticipées quand elle tente de se connecter à un service ou à une ressource réseau en réessayant d’exécuter en toute transparence une opération qui a échoué précédemment.

- [Modèle Supervision de point de terminaison d’intégrité](./health-endpoint-monitoring.md). Un disjoncteur peut tester l’intégrité d’un service en envoyant une demande à un point de terminaison exposé par le service. Le service doit retourner des informations indiquant son état.
