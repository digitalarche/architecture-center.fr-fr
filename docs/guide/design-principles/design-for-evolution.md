---
title: Penser la conception des applications en vue de leur évolution
description: Une conception évolutive est essentielle pour une innovation continue.
author: MikeWasson
ms.date: 08/30/2018
ms.openlocfilehash: df5a2d0756295a9632b3ea336527b2fbfb35318c
ms.sourcegitcommit: f6be2825bf2d37dfe25cfab92b9e3973a6b51e16
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/08/2018
ms.locfileid: "48858144"
---
# <a name="design-for-evolution"></a>Penser la conception des applications en vue de leur évolution

## <a name="an-evolutionary-design-is-key-for-continuous-innovation"></a>Une conception évolutive est essentielle pour une innovation continue

Toutes les applications réussies évoluent avec le temps, que ce soit pour résoudre les bogues, ajouter de nouvelles fonctionnalités, introduire de nouvelles technologies ou améliorer l’évolutivité et la résilience des systèmes existants. Toutefois, si toutes les parties d’une application sont étroitement liées, l’introduction de modifications dans le système se révèle particulièrement ardue. Un changement dans une partie de l’application risque d’entraîner l’interruption d’une autre portion du système ou de se répercuter sur la totalité de la base de code.

Ce problème ne concerne pas uniquement les applications monolithiques. Une application peut être décomposée en services et présenter encore ce type de couplage étroit qui rigidifie et fragilise le système. Toutefois, lorsque les services ont été conçus pour évoluer, les équipes peuvent innover et offrir en permanence de nouvelles fonctionnalités. 

Les microservices sont de plus en plus utilisés pour garantir une conception évolutive, car ils tiennent compte de la plupart des considérations répertoriées dans cet article.

## <a name="recommendations"></a>Recommandations

**Appliquez une forte cohésion et un couplage faible**. Un service est *cohésif* s’il fournit des fonctionnalités qui sont rattachées logiquement. Les services sont *faiblement couplés* si vous pouvez changer un service sans modifier les autres. En règle générale, une forte cohésion signifie que les modifications d’une fonction nécessiteront des changements dans les autres fonctions associées. Si vous constatez que la mise à jour d’un service requiert des mises à jour coordonnées d’autres services, ceci peut indiquer que vos services ne sont pas cohésifs. L’un des objectifs de la conception pilotée par domaine consiste à identifier ces limites.

**Encapsulez les connaissances du domaine**. Lorsqu’un client consomme un service, la responsabilité de l’application des règles d’entreprise du domaine ne doit pas lui incomber. À la place, le service doit encapsuler toutes les connaissances du domaine qui relèvent de sa responsabilité. Dans le cas contraire, chaque client devra appliquer les règles d’entreprise, et les connaissances du domaine seront alors éparpillées entre les différentes parties de l’application. 

**Utilisez une messagerie asynchrone**. Une messagerie asynchrone est un moyen de découpler le producteur de messages du consommateur de messages. Le producteur ne dépend pas de la réponse au message par le consommateur ni de l’exécution d’une action spécifique de la part de ce dernier. Avec une architecture de type publication/abonnement, le producteur peut même ignorer qui consomme le message. Les nouveaux services peuvent facilement consommer les messages sans impliquer aucune modification pour le producteur.

**N’intégrez pas de connaissances du domaine dans une passerelle**. Les passerelles peuvent se révéler utiles dans une architecture de microservices pour des aspects tels que le routage des requêtes, la traduction de protocole, l’équilibrage de charge ou l’authentification. Toutefois, la passerelle doit être restreinte à ce type de fonctionnalités d’infrastructure. Elle ne doit pas implémenter de connaissances du domaine afin d’éviter de devenir lourdement tributaire.

**Exposez des interfaces ouvertes**. Évitez de créer des couches de traduction personnalisées situées entre les services. À la place, un service doit exposer une API avec un contrat d’API bien défini. L’API doit faire l’objet d’un contrôle de version pour que vous puissiez la faire évoluer tout en préservant la compatibilité descendante. De cette façon, vous pouvez mettre à jour un service sans coordonner les mises à jour de tous les services en amont qui dépendent de ce service. Les services destinés au public doivent exposer une API RESTful sur HTTP. Les services principaux peuvent utiliser un protocole de messagerie de type RPC pour des raisons de performances. 

**Concevez et testez vos applications par rapport aux contrats de service**. Lorsque les services exposent des API bien définies, vous pouvez procéder aux phases de développement et de test par rapport à ces API. De cette façon, vous pouvez développer et tester un service spécifique sans passer en revue tous les services qui en dépendent. (Bien entendu, vous êtes toujours tenu d’effectuer des tests d’intégration et de charge par rapport aux services proprement dits.)

**Extrayez l’infrastructure de la logique du domaine**. Dissociez la logique du domaine des fonctionnalités associées à l’infrastructure, telles que la messagerie ou la persistance. Dans le cas contraire, l’apport de modifications à la logique du domaine nécessitera une mise à jour des couches d’infrastructure, et vice versa. 

**Déchargez les préoccupations transversales vers un service distinct**. Par exemple, si plusieurs services doivent authentifier des requêtes, vous pouvez déplacer cette fonctionnalité vers un service dédié. Vous serez ensuite en mesure de faire évoluer le service d’authentification &mdash; par exemple, en ajoutant un nouveau flux d’authentification &mdash; sans modifier aucun des services qui l’utilisent.

**Déployez les services de manière indépendante**. Lorsque l’équipe DevOps a la possibilité de déployer un service spécifique indépendamment des autres services de l’application, les mises à jour peuvent s’effectuer rapidement et en toute sécurité. Les résolutions de bogues et les nouvelles fonctionnalités sont ainsi déployables à un rythme plus régulier. Concevez l’application et son processus de mise en production en faisant en sorte qu’ils prennent en charge des mises à jour indépendantes.
