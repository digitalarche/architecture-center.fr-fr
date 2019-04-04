---
title: Modèles de conception de cloud
titleSuffix: Azure Architecture Center
description: Modèles de conception pour créer des applications fiables, scalables et sécurisées dans le cloud.
keywords: Azure
author: dragon119
ms.date: 03/01/2018
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: c8fe971e031825632c2bb157bfd23e15f56520a3
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58343473"
---
# <a name="cloud-design-patterns"></a>Modèles de conception de cloud

Ces modèles de conception sont utiles pour développer des applications fiables, évolutives et sécurisées dans le cloud.

Chaque modèle décrit le problème traité par le modèle, les aspects à prendre en compte pour l’application du modèle et un exemple basé sur Microsoft Azure. La plupart des modèles incluent des exemples ou des extraits de code qui montrent comment implémenter le modèle sur Azure. La plupart des modèles sont appropriés à n’importe quel système distribué, qu’il soit hébergé sur Azure ou sur d’autres plateformes cloud.

## <a name="challenges-in-cloud-development"></a>Défis liés au développement cloud

<!-- markdownlint-disable MD033 -->
<table>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/availability.md"><img src="_images/category/availability.svg" alt="Availability" /></a></td>
    <td>
        <h3><a href="./category/availability.md">Disponibilité</a></h3>
        <p>La disponibilité est la proportion de temps durant laquelle le système est opérationnel et fonctionne, généralement exprimée en pourcentage de temps d’activité. Elle peut être affectée par des erreurs système, des problèmes d’infrastructure, des attaques malveillantes ou la charge du système.  En général, les applications cloud fournissent aux utilisateurs un contrat de niveau de service (SLA), les applications doivent donc être conçues pour optimiser la disponibilité.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/data-management.md"><img src="_images/category/data-management.svg" alt="Data Management" /></a></td>
    <td>
        <h3><a href="./category/data-management.md">Gestion des données</a></h3>
        <p>La gestion des données est l’élément clé des applications cloud et a une incidence sur la plupart des attributs de qualité. Les données sont généralement hébergées dans des emplacements distincts et sur plusieurs serveurs pour des raisons de performances, d’extensibilité ou de disponibilité, ce qui peut soulever une multitude de problèmes. Par exemple, il est impératif de préserver la cohérence des données et de synchroniser généralement les données entre les différents emplacements.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/design-implementation.md"><img src="_images/category/design-implementation.svg" alt="Design and Implementation" /></a></td>
    <td>
        <h3><a href="./category/design-implementation.md">Conception et implémentation</a></h3>
        <p>Une bonne conception englobe des facteurs tels que la cohérence et la cohérence dans la conception de composants et de déploiements, la facilité de gestion pour simplifier l’administration et le développement, et la réutilisation pour permettre l’utilisation de composants et de sous-systèmes dans d’autres applications et dans d’autres scénarios. Les décisions prises pendant la phase de conception et d’implémentation ont un impact important sur la qualité et le coût total de possession d’applications et de services cloud.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/messaging.md"><img src="_images/category/messaging.svg" alt="Messaging" /></a></td>
    <td>
        <h3><a href="./category/messaging.md">Messagerie</a></h3>
        <p>La nature distribuée des applications cloud nécessite une infrastructure de messagerie qui connecte les composants et les services, dans l’idéal, d’une manière faiblement couplée afin d’optimiser l’extensibilité. La messagerie asynchrone est largement utilisée et offre de nombreux avantages, mais elle apporte également des défis, tels que le classement des messages, la gestion des messages incohérents, l’idempotence et bien plus encore.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/management-monitoring.md"><img src="_images/category/management-monitoring.svg" alt="Management and Monitoring" /></a></td>
    <td>
        <h3><a href="./category/management-monitoring.md">Gestion et surveillance</a></h3>
        <p>Les applications cloud s’exécutent dans un centre de données distant où vous ne possédez pas de contrôle total sur l’infrastructure ou, dans certains cas, sur le système d’exploitation. Cela peut rendre la gestion et la surveillance plus difficile que pour un déploiement sur site. Les applications doivent exposer des informations de runtime que les administrateurs et opérateurs peuvent utiliser pour gérer et surveiller le système, ainsi que pour modifier les personnalisations et les exigences de l’entreprise, sans que l’application ne doive être arrêtée ou redéployée.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/performance-scalability.md"><img src="_images/category/performance-scalability.svg" alt="Performance and Scalability" /></a></td>
    <td>
        <h3><a href="./category/performance-scalability.md">Performances et évolutivité</a></h3>
        <p>Les performances sont une indication de la réactivité d’un système pour exécuter des actions dans un intervalle de temps donné, tandis que l’extensibilité est la capacité d’un système à gérer les augmentations de charge sans impact sur les performances ou à augmenter les ressources disponibles de manière adéquate. Les applications cloud rencontrent généralement des charges de travail variables et des pics soudains dans l’activité. Prédire ces derniers, en particulier dans un scénario multi-locataire, est presque impossible. Au lieu de cela, les applications doivent être en mesure d’augmenter la taille des instances dans les limites pour répondre aux pics de demande et de diminuer la taille des instances lorsque la demande diminue. L’extensibilité ne concerne pas seulement les instances de calcul, mais également d’autres éléments, tels que le stockage des données, l’infrastructure de messagerie et bien plus encore.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/resiliency.md"><img src="_images/category/resiliency.svg" alt="Resiliency" /></a></td>
    <td>
        <h3><a href="./category/resiliency.md">Résilience</a></h3>
        <p>La résilience est la capacité d’un système à traiter de manière appropriée les défaillances et à récupérer après ces dernières. La nature de l’hébergement cloud, où les applications sont souvent multi-locataires, utilisent les services de plateforme partagée, sont en concurrence pour la bande passante et les ressources, communiquent via Internet et s’exécutent sur du matériel de base, signifie qu’il existe une probabilité accrue que des erreurs temporaires et des erreurs plus permanentes se produisent. Pouvoir détecter les pannes et récupérer rapidement et efficacement est nécessaire afin de maintenir la résilience.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/security.md"><img src="_images/category/security.svg" alt="Security" /></a></td>
    <td>
        <h3><a href="./category/security.md">Sécurité</a></h3>
        <p>La sécurité est la capacité d’un système à empêcher des actions accidentelles ou malveillantes en dehors de l’utilisation prévue et à empêcher la divulgation ou la perte d’informations. Les applications cloud sont exposées sur Internet en dehors des limites de confiance locales, elles sont souvent publiquement accessibles et peuvent être utilisées par des utilisateurs non approuvés. Les applications doivent être conçues et déployées d’une manière qui les protège des attaques malveillantes, restreint l’accès aux seuls les utilisateurs approuvés et protège les données sensibles.</p>
    </td>
</tr>
</table>
<!-- markdownlint-disable MD033 -->

## <a name="catalog-of-patterns"></a>Catalogue de modèles

|                                Modèle                                |                                                                                                         Résumé                                                                                                         |
|-----------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                     [Ambassadeur](./ambassador.md)                     |                                                            Créez des services d’assistance qui envoient des requêtes réseau pour le compte d’applications ou d’un service consommateur.                                                            |
|          [Couche de lutte contre la corruption](./anti-corruption-layer.md)          |                                                                  Implémentez une couche de façade ou d’adaptateur entre une application moderne et un système hérité.                                                                  |
|         [Services principaux destinés aux frontaux](./backends-for-frontends.md)         |                                                            Créez différents services principaux destinés à être utilisés par des applications ou interfaces frontales spécifiques.                                                             |
|                       [Cloisonnement](./bulkhead.md)                       |                                                        Isolez les éléments d’une application sous forme de pools afin qu’en cas de défaillance de l’un d’eux, les autres continuent à fonctionner.                                                        |
|                    [Cache-Aside](./cache-aside.md)                    |                                                                                   Chargez les données à la demande dans un cache à partir d’un magasin de données.                                                                                    |
|                [Disjoncteur](./circuit-breaker.md)                |                                                     Gérer les erreurs dont la résolution peut prendre un certain temps lors de la connexion à une ressource ou à un service distant.                                                     |
| [Vérification des revendications](./claim-check.md) | Diviser un message volumineux en une vérification des revendications et une charge utile pour éviter de surcharger un bus de messages. |
|       [Transaction de compensation](./compensating-transaction.md)       |                                                         Annulez le travail effectué par une série d’étapes qui définissent ensemble une opération cohérente.                                                         |
|            [Consommateurs concurrents](./competing-consumers.md)            |                                                            Ce modèle vise à permettre à plusieurs consommateurs concurrents de traiter les messages reçus sur un même canal de messagerie.                                                             |
| [Consolidation des ressources de calcul](./compute-resource-consolidation.md) |                                                                        Consolidez plusieurs tâches ou opérations en une seule unité de calcul.                                                                        |
|                           [CQRS](./cqrs.md)                           |                                                           Séparez les opérations qui lisent les données des opérations qui mettent à jour les données en utilisant des interfaces distinctes.                                                            |
|                 [Approvisionnement en événements](./event-sourcing.md)                 |                                                      Utilisez un magasin d’ajout uniquement pour enregistrer la série complète d’événements qui décrivent les actions exécutées sur les données dans un domaine.                                                      |
|   [Magasin de configurations externes](./external-configuration-store.md)   |                                                           Déplacez les informations de configuration depuis le package de déploiement d’application vers un emplacement centralisé.                                                           |
|             [Identité fédérée](./federated-identity.md)             |                                                                                Déléguez l’authentification à un fournisseur d’identité externe.                                                                                |
|                     [Opérateur de contrôle](./gatekeeper.md)                     | Protégez les applications et services à l’aide d’une instance d’hôte dédiée qui agit comme un intermédiaire entre les clients et l’application ou le service, valide et assainit les demandes et transmet les demandes et les données entre eux. |
|            [Agrégation de passerelle](./gateway-aggregation.md)            |                                                                     Utilisez une passerelle pour agréger plusieurs requêtes individuelles dans une requête unique.                                                                      |
|             [Déchargement de passerelle](./gateway-offloading.md)             |                                                                         Déchargez des fonctionnalités de service partagé ou spécialisé sur un proxy de passerelle.                                                                         |
|                [Routage de passerelle](./gateway-routing.md)                |                                                                              Acheminez les requêtes vers plusieurs services à l’aide d’un seul point de terminaison.                                                                               |
|     [Surveillance de point de terminaison d’intégrité](./health-endpoint-monitoring.md)     |                                              Implémentez des contrôles fonctionnels dans une application à laquelle des outils externes peuvent accéder par le biais de points de terminaison exposés à intervalles réguliers.                                               |
|                    [Table d’index](./index-table.md)                    |                                                                Créez des index sur les champs des magasins de données qui sont souvent référencés par les requêtes.                                                                 |
|                [Élection du responsable](./leader-election.md)                |   Coordonnez les actions effectuées par un ensemble d’instances de tâche de collaboration dans une application distribuée en élisant l’instance responsable qui sera chargée de gérer les autres instances.    |
|              [Vue matérialisée](./materialized-view.md)              |                                        Générez des vues préremplies sur les données d’un ou de plusieurs magasins de données lorsque les données ne sont pas adéquatement formatées pour les opérations de requête requises.                                        |
|              [Canaux et filtres](./pipes-and-filters.md)              |                                                        Divisez une tâche qui exécute un traitement complexe en une série d’éléments séparés qui peuvent être réutilisés.                                                        |
|                 [File d’attente de priorité](./priority-queue.md)                 |                                 Classez par ordre de priorité les requêtes envoyées aux services, de telle sorte que les demandes ayant une priorité plus élevée soient reçues et traitées plus rapidement que celles de moindre priorité.                                  |
| [Serveur de publication/abonné](./publisher-subscriber.md) | Activez une application pour annoncer des événements à plusieurs consommateurs intéressés de manière asynchrone, sans coupler les expéditeurs aux destinataires. |
|      [Nivellement de la charge basé sur une file d’attente](./queue-based-load-leveling.md)      |                                               Utilisez une file d’attente qui agit comme mémoire tampon entre une tâche et un service qu’elle appelle, afin d’atténuer les surcharges intermittentes.                                               |
|                          [Nouvelle tentative](./retry.md)                          |               Permettez à une application de gérer les défaillances temporaires anticipées quand elle tente de se connecter à un service ou à une ressource réseau en réessayant d’exécuter en toute transparence une opération qui a échoué précédemment.                |
|     [Superviseur de l’agent du planificateur](./scheduler-agent-supervisor.md)     |                                                              Coordonnez un ensemble d’actions sur un ensemble distribué de services et d’autres ressources à distance.                                                               |
|                       [Partitionnement](./sharding.md)                       |                                                                           Divisez un magasin de données en un ensemble de partitions horizontales ou de partitions de base de données.                                                                            |
|                        [Sidecar](./sidecar.md)                        |                                                    Déployez les composants d’une application sur un processus ou conteneur distinct pour fournir l’isolation et l’encapsulation.                                                     |
|         [Hébergement de contenu statique](./static-content-hosting.md)         |                                                          Déployez un contenu statique dans un service de stockage cloud qui peut distribuer ce contenu directement au client.                                                           |
|                      [Étranglement](./strangler.md)                      |                                            Faites migrer un système hérité de façon incrémentielle en remplaçant progressivement des parties spécifiques des fonctionnalités par de nouveaux services et applications.                                            |
|                     [Limitation](./throttling.md)                     |                                                 Contrôlez la consommation des ressources utilisées par une instance d’une application, un locataire ou un service entier.                                                 |
|                      [Clé Valet](./valet-key.md)                      |                                                        Utilisez un jeton ou une clé qui fournissent aux clients un accès direct limité à une ressource ou un service spécifique.                                                        |
