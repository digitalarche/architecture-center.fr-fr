---
title: "Modèle ambassadeur"
description: "Créez des services d’assistance qui envoient des requêtes réseau pour le compte d’applications ou d’un service consommateur."
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: 6c545619aab6a5817e55854350e3769834df27cd
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2017
---
# <a name="ambassador-pattern"></a>Modèle ambassadeur

Créez des services d’assistance qui envoient des requêtes réseau pour le compte d’applications ou d’un service consommateur. Un service Ambassadeur peut être considéré comme un proxy hors processus colocalisé avec le client.

Ce modèle peut être utile pour décharger des tâches de connectivité client courantes telles que la surveillance, la journalisation, le routage, la sécurité (par exemple, TLS), mais aussi des [modèles de résilience][resiliency-patterns] sans dépendance à un quelconque langage. Il est souvent utilisé avec des applications héritées ou d’autres applications qui sont difficiles à modifier afin d’étendre leurs capacités de mise en réseau. Il peut également être utilisé par une équipe spécialisée pour implémenter ces fonctionnalités.

## <a name="context-and-problem"></a>Contexte et problème

Les applications cloud résilientes nécessitent des fonctionnalités telles que [la rupture de circuit][circuit-breaker], le routage, le contrôle et la surveillance, ainsi que la possibilité d’effectuer des mises à jour de configuration au niveau du réseau. Il peut être difficile, voire impossible, de mettre à jour les applications héritées ou les bibliothèques de code existantes pour ajouter ces fonctionnalités, car le code n’est pas conservé ou qu’il est difficilement modifiable par l’équipe de développement.

Les appels réseau peuvent également nécessiter une configuration importante en matière de connexion, d’authentification et d’autorisation. Si ces appels sont utilisés dans plusieurs applications et générés à l’aide de plusieurs langages et infrastructures, ils doivent être configurés pour chacune de ces instances. En outre, il est possible que les fonctionnalités réseau et de sécurité doivent être gérées par une équipe centrale au sein de votre organisation. En présence d’une base de code volumineuse, il peut être risqué pour cette équipe de mettre à jour un code d’application qu’elle ne connait pas bien.

## <a name="solution"></a>Solution

Placez les bibliothèques et les infrastructures client dans un processus externe qui agit comme un proxy entre votre application et les services externes. Déployez le proxy dans le même environnement d’hôte que votre application pour pouvoir contrôler les fonctionnalités de routage, de résilience et de sécurité, et éviter toute restriction d’accès liée à l’hôte. Vous pouvez également utiliser le modèle ambassadeur pour normaliser et étendre l’instrumentation. Le proxy peut analyser les indicateurs de performance tels que la latence ou l’utilisation des ressources, et ce dans le même environnement que celui de l’application.

![](./_images/ambassador.png)

Les fonctionnalités qui sont déchargées vers l’ambassadeur peuvent être gérées indépendamment de l’application. Vous pouvez mettre à jour et modifier l’ambassadeur sans que cela n’ait d’incidence sur les fonctionnalités héritées de l’application. Les différentes équipes spécialisées peuvent également implémenter et gérer les fonctionnalités de sécurité, de mise en réseau ou d’authentification qui ont été déplacées vers l’ambassadeur.

Les services Ambassadeur peuvent être déployés en tant que [side-car][sidecar] pour accompagner le cycle de vie d’une application ou d’un service consommateur. Si un ambassadeur est partagé par plusieurs processus distincts sur un ordinateur hôte commun, vous pouvez aussi le déployer en tant que démon ou service Windows. Si le service consommateur est exécuté en conteneur, l’ambassadeur doit être créé en tant que conteneur distinct sur le même ordinateur hôte et les liaisons appropriées doivent être configurées pour établir la communication entre les deux instances.

## <a name="issues-and-considerations"></a>Problèmes et considérations

- Le proxy provoque une augmentation de la latence. Demandez-vous si une bibliothèque cliente, appelée directement par l’application, pourrait être une meilleure approche.
- Demandez-vous quel impact pourrait avoir l’intégration de fonctionnalités généralisées au proxy. L’ambassadeur pourrait par exemple gérer de nouvelles tentatives, mais cela pourrait s’avérer risqué, sauf si toutes les opérations sont idempotentes.
- Envisagez de déployer un mécanisme qui permettrait au client de transférer du contexte au proxy, et inversement. Par exemple, ajoutez des en-têtes de requête HTTP pour refuser de nouvelles tentatives ou spécifiez le nombre maximal de tentatives autorisées.
- Déterminez de quelle manière vous allez empaqueter et déployer le proxy.
- Choisissez d’utiliser une instance partagée unique pour tous les clients ou une instance pour chaque client.

## <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle

Utilisez ce modèle dans les situations suivantes :

- Vous devez générer un ensemble commun de fonctionnalités de connectivité client pour plusieurs langages ou infrastructures.
- Vous devez décharger des problèmes de connectivité client transversaux vers les développeurs d’infrastructure ou d’autres équipes plus spécialisées.
- Vous devez répondre à des exigences de connectivité cloud ou de cluster dans une application héritée ou une application qui est difficile à modifier.

Ce modèle peut ne pas convenir :

- Lorsque la latence des requêtes réseau est critique. Un proxy introduira un surcroît de latence qui, bien que minimal, pourrait dans certains cas affecter l’application.
- Lorsque les fonctionnalités de connectivité client sont consommées par un seul langage. Dans ce cas, il peut être plus judicieux d’utiliser une bibliothèque cliente qui est distribuée aux équipes de développement sous la forme d’un package.
- Lorsque les fonctionnalités de connectivité ne peuvent pas être généralisées et nécessitent une intégration plus étroite avec l’application cliente.

## <a name="example"></a>Exemple

Le diagramme suivant illustre une application envoyant une requête à un service distant via un proxy Ambassadeur. L’ambassadeur assure le routage, la rupture de circuit et la journalisation. Il appelle le service distant, puis renvoie la réponse à l’application cliente :

![](./_images/ambassador-example.png) 

## <a name="related-guidance"></a>Aide connexe

- [Modèle side-car](./sidecar.md)

<!-- links -->

[circuit-breaker]: ./circuit-breaker.md
[resiliency-patterns]: ./category/resiliency.md
[sidecar]: ./sidecar.md
