---
title: Modèle des consommateurs concurrents
titleSuffix: Cloud Design Patterns
description: Ce modèle vise à permettre à plusieurs consommateurs concurrents de traiter les messages reçus sur un même canal de messagerie.
keywords: modèle de conception
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: ea3f48971a78f59ad6575b055278aab449fa26a1
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58244470"
---
# <a name="competing-consumers-pattern"></a>Modèle des consommateurs concurrents

[!INCLUDE [header](../_includes/header.md)]

Ce modèle vise à permettre à plusieurs consommateurs concurrents de traiter les messages reçus sur un même canal de messagerie. Un système peut ainsi traiter simultanément plusieurs messages de façon à optimiser le débit, à améliorer la scalabilité et la disponibilité et à équilibrer la charge de travail.

## <a name="context-and-problem"></a>Contexte et problème

Une application s’exécutant dans le cloud a de nombreuses demandes à gérer. Plutôt que de traiter chaque demande de façon synchrone, une technique courante consiste pour l’application à les passer à un autre service (service consommateur) via un système de messagerie pour être traitées de façon asynchrone. Cette stratégie permet de s’assurer que la logique métier de l’application n’est pas bloquée pendant le traitement des demandes.

Le nombre de demandes peut varier considérablement au fil du temps, et ce pour diverses raisons. Une augmentation soudaine de l’activité utilisateur ou des demandes agrégées en provenance de plusieurs locataires clients peut occasionner une charge de travail imprévisible. Aux heures de pointe, un système peut être amené à traiter plusieurs centaines de demandes à la seconde, alors qu’à d’autres moments, leur nombre peut être très faible. Par ailleurs, la nature du travail effectué pour traiter ces demandes peut varier considérablement. Si une instance unique du service consommateur est utilisée, elle peut se trouver submergée par les demandes ou le système de messagerie peut-être surchargé par un afflux de messages en provenance de l’application. Pour gérer cette charge de travail fluctuante, le système peut exécuter plusieurs instances du service consommateur. Cependant, ces consommateurs doivent être coordonnés pour faire en sorte que chaque message ne soit qu’à un seul consommateur. La charge de travail doit aussi faire l’objet d’un équilibrage de charge entre les consommateurs pour empêcher une instance de devenir un goulot d’étranglement.

## <a name="solution"></a>Solution

Utilisez une file d’attente de messages pour implémenter le canal de communication entre l’application et les instances du service consommateur. L’application poste les demandes sous forme de messages dans la file d’attente, et les instances du service consommateur reçoivent les messages de la file d’attente pour ensuite les traiter. Cette approche permet à un même pool d’instances du service consommateur de traiter les messages de n’importe quelle instance de l’application. La figure illustre l’utilisation d’une file d’attente de messages pour distribuer le travail aux instances d’un service.

![Utilisation d’une file d’attente de messages pour distribuer le travail aux instances d’un service](./_images/competing-consumers-diagram.png)

Cette solution offre les avantages suivants :

- Elle fournit un système à charge nivelée qui peut gérer les écarts importants dans le volume de demandes envoyées par les instances de l’application. La file d’attente joue le rôle de tampon entre les instances de l’application et les instances du service consommateur. L’impact sur la disponibilité et la réactivité de l’application et des instances du service peut ainsi s’en trouver limité, comme décrit dans [Modèle de nivellement de charge basé sur une file d’attente](./queue-based-load-leveling.md). La gestion d’un message qui demande un traitement de longue durée n’empêche pas la gestion simultanée d’autres messages par d’autres instances du service consommateur.

- Elle améliore la fiabilité. Si un producteur communique directement avec un consommateur au lieu d’utiliser ce modèle, mais qu’il ne surveille pas le consommateur, il est fort probable que les messages seront perdus ou ne pourront pas être traités en cas de défaillance du consommateur. Dans ce modèle, les messages ne sont pas envoyés à une instance de service spécifique. Une instance de service défaillante ne bloquera pas un producteur, et les messages pourront être traités par n’importe quelle instance de service active.

- Elle ne nécessite pas de coordination complexe entre les consommateurs ou entre le producteur et les instances de consommateur. La file d’attente de messages offre la garantie que chaque message est remis au moins une fois.

- Elle est scalable. Le système peut augmenter ou diminuer dynamiquement le nombre d’instances du service consommateur à mesure que le volume de messages évolue.

- Elle peut améliorer la résilience si la file d’attente de messages fournit des opérations de lecture transactionnelles. Si une instance du service consommateur lit et traite le message dans le cadre d’une opération transactionnelle et que l’instance du service consommateur subit une défaillance, ce modèle peut faire en sorte que le message soit retourné à la file d’attente pour être récupéré et traité par une autre instance du service consommateur.

## <a name="issues-and-considerations"></a>Problèmes et considérations

Prenez en compte les points suivants lorsque vous choisissez comment implémenter ce modèle :

- **Classement des messages** : l’ordre dans lequel les instances du service consommateur reçoivent les messages n’est pas garanti et ne reflète pas nécessairement l’ordre dans lequel les messages ont été créés. Concevez le système de sorte que le traitement des messages soit idempotent. Vous éliminerez ainsi toute dépendance vis-à-vis de l’ordre de traitement des messages. Pour plus d’informations, consultez [Idempotency Patterns](https://blog.jonathanoliver.com/idempotency-patterns/) sur le blog de Jonathan Oliver.

    > Les files d’attente Microsoft Azure Service Bus peuvent implémenter un classement premier entrée premier sorti en utilisant des sessions de messagerie. Pour plus d’informations, consultez [Modèles de messagerie utilisant des sessions](https://msdn.microsoft.com/magazine/jj863132.aspx).

- **Conception de services à des fins de résilience** : si le système est conçu pour détecter et redémarrer les instances de service en échec, il peut être nécessaire d’implémenter le traitement effectué par les instances de service en tant qu’opérations idempotent pour limiter les effets d’un message unique qui serait récupéré et traité plusieurs fois.

- **Détection des messages incohérents** : un message au format incorrect ou une tâche qui nécessite un accès à des ressources non disponibles peut entraîner l’échec d’une instance de service. Le système doit éviter que ces messages soient retournés à la file d’attente et au lieu de cela capturer et stocker les détails de ces messages à un autre emplacement en vue de leur analyse, si nécessaire.

- **Gestion des résultats** : l’instance de service traitant un message est entièrement dissociée de la logique d’application qui génère le message, et il se peut qu’elles ne puissent pas communiquer directement. Si l’instance de service génère des résultats qui doivent être retransmis à la logique d’application, ces informations doivent être stockées à un emplacement accessible aux deux. Pour éviter que la logique d’application récupère des données incomplètes, le système doit indiquer à quel moment le traitement est terminé.

     > Si vous utilisez Azure, un processus de travail peut retransmettre les résultats à la logique d’application en utilisant une file d’attente de réponses de messages dédiée. La logique d’application doit pouvoir mettre en corrélation ces résultats avec le message d’origine. Ce scénario est décrit plus en détail dans [Notions élémentaires sur la messagerie asynchrone](https://msdn.microsoft.com/library/dn589781.aspx).

- **Mise à l’échelle du système de messagerie** : dans une solution à grande échelle, une file d’attente de messages unique peut être submergée par le nombre de messages et devenir un goulot d’étranglement dans le système. Dans ce cas, envisagez de partitionner le système de messagerie pour envoyer les messages de producteurs spécifiques à une file d’attente particulière, ou utilisez l’équilibrage de charge pour répartir les messages entre plusieurs files d’attente de messages.

- **Assurer la fiabilité du système de messagerie** : disposer d’un système de messagerie fiable est une nécessité pour garantir qu’un message ne se perd pas après avoir été mis en file d’attente par l’application. Il s’agit d’une condition essentielle pour garantir que tous les messages sont remis au moins une fois.

## <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle

Utilisez ce modèle dans les situations suivantes :

- La charge de travail d’une application est divisée en tâches qui peuvent s’exécuter de façon asynchrone.
- Les tâches sont indépendantes et peuvent s’exécuter en parallèle.
- le volume de travail est très variable, ce qui nécessite une solution scalable.
- La solution doit offrir une haute disponibilité et doit être résiliente en cas d’échec du traitement d’une tâche.

Ce modèle peut ne pas avoir d’utilité dans les cas suivants :

- Il n’est pas facile de séparer la charge de travail d’application en tâches discrètes ou il existe un niveau de dépendance élevé entre les tâches.
- Les tâches doivent être effectuées de façon synchrone et la logique d’application doit attendre qu’une tâche se termine avant de continuer.
- Les tâches doivent être effectuées dans un ordre précis.

> Certains systèmes de messagerie prennent en charge les sessions qui permettent à un producteur de regrouper les messages et de veiller à ce qu’ils soient tous gérés par le même consommateur. Ce mécanisme peut être utilisé avec des messages classés par ordre de priorité (s’ils sont pris en charge) pour implémenter une forme de classement de messages qui remet les messages dans l’ordre d’un producteur à un consommateur unique.

## <a name="example"></a>Exemples

Azure propose des files d’attente de stockage et des files d’attente Service Bus qui peuvent servir de mécanisme pour l’implémenter ce modèle. La logique d’application peut poster des messages dans une file d’attente, et les consommateurs implémentés comme tâches dans un ou plusieurs rôles peuvent récupérer les messages de cette file d’attente et les traiter. Pour des besoins de résilience, une file d’attente Service Bus permet à un consommateur d’utiliser le mode `PeekLock` quand il s’agit de récupérer un message dans la file d’attente. Ce mode ne supprime pas réellement le message ; il ne fait que le masquer aux autres consommateurs. Le consommateur d’origine peut supprimer le message à l’issue de son traitement. En cas de défaillance du consommateur, le mode PeekLock expire et le message redevient visible, ce qui permet à un autre consommateur de le récupérer.

Pour obtenir des informations détaillées sur l’utilisation des files d’attente Azure Service Bus, consultez [Files d’attente, rubriques et abonnements Service Bus](https://msdn.microsoft.com/library/windowsazure/hh367516.aspx).

Pour plus d’informations sur l’utilisation des files d’attente de stockage Azure, consultez [Prise en main du stockage de files d’attente Azure à l’aide de .NET](/azure/storage/queues/storage-dotnet-how-to-use-queues).

Le code suivant de la classe `QueueManager` de la solution CompetingConsumers disponible sur [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/competing-consumers) montre comment créer une file d’attente en utilisant une instance `QueueClient` dans le gestionnaire d’événements `Start` d’un rôle web ou de travail.

```csharp
private string queueName = ...;
private string connectionString = ...;
...

public async Task Start()
{
  // Check if the queue already exists.
  var manager = NamespaceManager.CreateFromConnectionString(this.connectionString);
  if (!manager.QueueExists(this.queueName))
  {
    var queueDescription = new QueueDescription(this.queueName);

    // Set the maximum delivery count for messages in the queue. A message
    // is automatically dead-lettered after this number of deliveries. The
    // default value for dead letter count is 10.
    queueDescription.MaxDeliveryCount = 3;

    await manager.CreateQueueAsync(queueDescription);
  }
  ...

  // Create the queue client. By default the PeekLock method is used.
  this.client = QueueClient.CreateFromConnectionString(
    this.connectionString, this.queueName);
}
```

L’extrait de code suivant montre comment une application peut créer et envoyer un lot de messages à la file d’attente.

```csharp
public async Task SendMessagesAsync()
{
  // Simulate sending a batch of messages to the queue.
  var messages = new List<BrokeredMessage>();

  for (int i = 0; i < 10; i++)
  {
    var message = new BrokeredMessage() { MessageId = Guid.NewGuid().ToString() };
    messages.Add(message);
  }
  await this.client.SendBatchAsync(messages);
}
```

Le code suivant montre comment une instance de service consommateur peut recevoir des messages de la file d’attente en suivant une approche basée sur les événements. Le paramètre `processMessageTask` de la méthode `ReceiveMessages` est un délégué qui fait référence au code à exécuter à la réception d’un message. Ce code s’exécute de façon asynchrone.

```csharp
private ManualResetEvent pauseProcessingEvent;
...

public void ReceiveMessages(Func<BrokeredMessage, Task> processMessageTask)
{
  // Set up the options for the message pump.
  var options = new OnMessageOptions();

  // When AutoComplete is disabled it's necessary to manually
  // complete or abandon the messages and handle any errors.
  options.AutoComplete = false;
  options.MaxConcurrentCalls = 10;
  options.ExceptionReceived += this.OptionsOnExceptionReceived;

  // Use of the Service Bus OnMessage message pump.
  // The OnMessage method must be called once, otherwise an exception will occur.
  this.client.OnMessageAsync(
    async (msg) =>
    {
      // Will block the current thread if Stop is called.
      this.pauseProcessingEvent.WaitOne();

      // Execute processing task here.
      await processMessageTask(msg);
    },
    options);
}
...

private void OptionsOnExceptionReceived(object sender,
  ExceptionReceivedEventArgs exceptionReceivedEventArgs)
{
  ...
}
```

Notez que les fonctionnalités de mise à l’échelle automatique, telles que celles disponibles dans Azure, peuvent être utilisées pour démarrer et arrêter des instances de rôle à mesure que la longueur de la file d’attente évolue. Pour plus d’informations, consultez [Recommandations en matière de mise à l’échelle automatique](https://msdn.microsoft.com/library/dn589774.aspx). De même, il n’est pas nécessaire de conserver une correspondance un-à-un entre les instances de rôle et les processus de travail (une même instance de rôle peut implémenter plusieurs processus de travail). Pour plus d’informations, consultez [Modèle de consolidation des ressources de calcul](./compute-resource-consolidation.md).

## <a name="related-patterns-and-guidance"></a>Conseils et modèles connexes

Les modèles et les conseils suivants peuvent présenter un intérêt quand il s’agit d’implémenter ce modèle :

- [Primer de messagerie asynchrone](https://msdn.microsoft.com/library/dn589781.aspx). les files d’attente de messages sont un mécanisme de communication asynchrone. Si un service consommateur doit envoyer une réponse à une application, il peut être nécessaire d’implémenter une certaine forme de messagerie de réponse. Le document Notions élémentaires sur la messagerie asynchrone fournit des informations sur la façon d’implémenter une messagerie de demande/réponse en utilisant des files d’attente de messages.

- [Mise à l’échelle automatique](https://msdn.microsoft.com/library/dn589774.aspx). il est possible de démarrer et d’arrêter des instances d’un service consommateur, car la longueur de la file d’attente dans laquelle les applications postent les messages est variable. La mise à l’échelle automatique peut contribuer à maintenir le débit pendant les périodes d’intense traitement.

- [Modèle de consolidation des ressources de calcul](./compute-resource-consolidation.md). il est possible de consolider plusieurs instances d’un service consommateur dans un processus unique pour réduire les coûts et les surcharges de gestion. Le modèle de consolidation des ressources de calcul décrit les avantages et les inconvénients de cette approche.

- [Queue-based Load Leveling pattern](./queue-based-load-leveling.md) (Modèle de nivellement de charge basé sur une file d’attente). l’introduction d’une file d’attente de messages peut doter le système de capacités de résilience, permettant ainsi aux instances du service de gérer des volumes très variables de demandes en provenance d’instances d’application. La file d’attente de messages fait office de tampon, ce qui a pour effet de niveler la charge. Le modèle de nivellement de la charge basé sur une file d’attente décrit plus en détail ce scénario.

- Ce modèle s’accompagne d’un [exemple d’application](https://github.com/mspnp/cloud-design-patterns/tree/master/competing-consumers).
