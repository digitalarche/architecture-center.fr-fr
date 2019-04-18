---
title: 'Framework d’adoption du cloud : Principes opérationnels de base'
description: Aide sur les principes opérationnels de base
author: petertaylor9999
ms.date: 09/20/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.openlocfilehash: 7d7f6bd46fb60a190a7fb27432d5ff4b74b0c597
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640700"
---
# <a name="establishing-an-operational-fitness-review"></a>Mettre en place une évaluation de l’adéquation opérationnelle

Lorsque votre entreprise commence à exploiter des charges de travail dans Azure, il lui faut établir un processus **d’évaluation de l’adéquation opérationnelle** pour lister, implémenter et vérifier de manière itérative les exigences **non fonctionnelles** de ces charges de travail. Ces exigences _non fonctionnelles_ sont liées au comportement opérationnel attendu du service. Il existe cinq catégories essentielles d’exigences non fonctionnelles, nommées [piliers de la qualité logicielle](../../guide/pillars.md) : l’extensibilité, la disponibilité, la résilience (y compris la continuité d’activité et la reprise d’activité), la gestion et la sécurité. L’objectif d’un processus d’évaluation de l’adéquation opérationnelle est de vérifier que les charges de travail critiques répondent aux attentes de votre entreprise en ce qui concerne les piliers de la qualité.

Votre entreprise doit donc procéder à un processus d’évaluation de l’adéquation opérationnelle pour bien comprendre les problèmes liés à l’exécution de la charge de travail dans un environnement de production, trouver comment corriger les problèmes, puis les résoudre. Cet article décrit un processus d’évaluation de l’adéquation opérationnelle de haut niveau qui pourra permettre à votre entreprise d’atteindre cet objectif.

## <a name="operational-fitness-at-microsoft"></a>L’adéquation opérationnelle chez Microsoft

Le développement de la plateforme Azure a dès le départ constitué un projet de développement et d’intégration continus, entrepris par de nombreuses équipes Microsoft. Il serait très difficile de garantir la qualité et la cohérence d’un projet de la taille et de la complexité d’Azure sans un processus robuste permettant de lister et d’implémenter régulièrement les exigences non fonctionnelles fondamentales.

Les processus suivis par Microsoft constituent la base de ceux qui sont décrits dans ce document.

## <a name="understanding-the-problem"></a>L’identification du problème

Comme vous l’avez appris dans [Bien démarrer](../../cloud-adoption/getting-started/overview.md), la première étape de la transformation numérique d’une entreprise consiste à identifier les problèmes métier à résoudre avec l’adoption d’Azure. La suivante vise à trouver une solution de haut niveau au problème, comme la migration d’une charge de travail vers le cloud, ou l’adaptation d’un service local existant aux fonctionnalités cloud. Enfin, la solution est conçue et implémentée.

Pendant ce processus, l’accent est souvent mis sur les _fonctions_ du service. Autrement dit, il existe un ensemble d’exigences _fonctionnelles_ souhaitées pour que le service fonctionne. Prenons par exemple un service de livraison de produits. Plusieurs fonctions sont nécessaires : déterminer les lieux de départ et de destination du produit, assurer son suivi au cours de la livraison, envoyer des notifications aux clients, etc.

En revanche, le _non fonctionnelles_ exigences se rapportent aux propriétés telles que le service [fiabilité](../../reliability/index.md) et [évolutivité](../../checklist/scalability.md). Ces propriétés diffèrent des exigences fonctionnelles, car elles n’affectent pas directement les fonctions finales du service. En revanche, elles touchent aux _performances_ et à la _continuité_ du service.

Certaines exigences non fonctionnelles peuvent être spécifiées par un contrat de niveau de service (SLA). Par exemple, en ce qui concerne la continuité du service, on peut exprimer l’exigence de disponibilité sous forme de pourcentage, comme **disponible 99,99 % du temps**. D’autres exigences non fonctionnelles sont plus difficiles à définir et susceptibles de changer en fonction de l’évolution des besoins de production. Par exemple, un service aux consommateurs risque de se retrouver face à des exigences de débit imprévues après une forte hausse de popularité.

! [REMARQUE] Définition de la configuration requise pour la résilience, y compris les explications de RPO, RTO, contrat SLA et des concepts connexes, sont abordées plus en détail dans [développement de spécifications pour les applications Azure résilientes](../../reliability/requirements.md).

## <a name="operational-fitness-review-process"></a>Le processus d’évaluation de l’adéquation opérationnelle

Pour préserver les performances et assurer la continuité des services de l’entreprise, la clé est d’implémenter un processus _d’évaluation de l’adéquation opérationnelle_.

![Vue d’ensemble du processus d’évaluation de l’adéquation opérationnelle](_images/ofr-flow.png)

À haut niveau, le processus se compose de deux phases. Dans la phase des prérequis sont établies les exigences, qui sont ensuite mises en correspondance avec les services de soutien. Cette phase se produit peu fréquemment, peut-être une fois par an ou à l’introduction de nouvelles opérations. Le résultat de la phase des prérequis est utilisé dans la phase des flux. Cette dernière est plus fréquente ; nous vous recommandons une périodicité mensuelle.

### <a name="prerequisites-phase"></a>La phase des prérequis

Les étapes de cette phase visent à capturer les exigences associées à une évaluation régulière des services importants.

- **Identifier les opérations d’entreprise critiques**. Identifiez les opérations stratégiques de l’entreprise. Les opérations d’entreprise sont indépendantes des fonctions de service de soutien. En d’autres termes, elles représentent les activités réelles que doit effectuer l’entreprise et sont portées par un ensemble de services informatiques. Le terme **stratégique** (ou **critique pour l’entreprise**), reflète un impact très important pour l’entreprise si l’opération est entravée. Prenons l’exemple d’un détaillant en ligne et de deux de ses opérations d’entreprise : « permettre à un client d’ajouter un article au panier » ou « traiter un paiement par carte de crédit ». Si l’une des deux échouait, les clients ne pourraient pas mener à bien les transactions et l’entreprise ne réaliserait pas de ventes.

- **Faire correspondre les opérations aux services**. Faites correspondre ces opérations d’entreprise aux services de soutien associés. Dans l’exemple du panier ci-dessus, plusieurs services peuvent être concernés : un service de gestion des stocks, un service de panier d’achat, etc. Dans l’exemple précédent de paiement par carte de crédit, un service de paiement local est susceptible d’interagir avec un service tiers de traitement des paiements.

- **Analyser les dépendances entre les services**. La plupart des opérations d’entreprise impliquent une orchestration entre plusieurs services de soutien. Il est important de comprendre les dépendances entre les services et le flux de transactions critiques via ces services. Examinez également les dépendances entre les services locaux et les services Azure. Dans l’exemple du panier d’achat, le service de gestion des stocks peut être hébergé en local et ingérer des données d’entrée émises par les employés dans un entrepôt physique, ou bien stocker des données dans un service Azure comme le [Stockage Azure](/azure/storage/common/storage-introduction) ou une base de données comme [Azure Cosmos DB](/azure/cosmos-db/introduction).

À partir de ces activités est produite une série **d’indicateurs de tableau de bord** pour les opérations de service. Ces métriques sont classées en fonction des critères non fonctionnels, comme la disponibilité, l’extensibilité et la récupération d’urgence, que le service doit respecter d’un point de vue opérationnel. Elles sont exprimées à un niveau de granularité adapté à l’opération de service, quel qu’il soit.

Le tableau de bord doit être exprimé en termes simples pour faciliter la discussion entre les responsables des résultats d’entreprise et les ingénieurs. Par exemple, un indicateur d’évolutivité peut apparaître en _vert_ si les critères souhaités sont remplis, en _jaune_ si ce n’est pas le cas mais qu’une correction planifiée est activement en cours d’implémentation, et en _rouge_ en cas d’échec sans plan ni mesures.

Il est important de souligner que ces métriques doivent refléter directement les besoins de l’entreprise.

### <a name="service-review-phase"></a>La phase d’évaluation des services

La phase d’évaluation des services est essentielle au processus d’évaluation de l’adéquation opérationnelle.

- **Mettre en place des métriques de service**. Les indicateurs de tableau de bord permettent d’effectuer le monitoring des services pour vérifier qu’ils répondent aux attentes de l’entreprise. Cette surveillance est donc essentielle. Si vous n’êtes pas en mesure d’assurer le monitoring d’un ensemble de services en regard des exigences non fonctionnelles, les indicateurs de tableau de bord correspondants doivent être considérés comme rouges. La première étape pour y remédier consiste à implémenter le monitoring du service en question. Par exemple, si l’entreprise attend d’un service qu’il fonctionne avec une disponibilité de 99,99 %, mais qu’aucune télémétrie de production n’est en place pour la mesurer, partez du principe que vous ne répondez pas à cette exigence.

- **Prévoir des mesures de correction**. Pour chaque opération de service dont les métriques se situent sous le seuil admissible, calculez ce que coûterait une correction du service permettant d’y remédier. Si ce coût est supérieur au revenu attendu du service, prenez en compte les coûts non tangibles comme l’expérience utilisateur. Par exemple, si les clients ont des difficultés à passer une commande à l’aide du service, ils risquent de choisir un concurrent à la place.

- **Implémenter la correction**. Dès que les responsables des résultats d’entreprise et les ingénieurs se sont mis d’accord sur un plan, celui-ci doit être implémenté. L’état de l’implémentation doit être signalé à chaque fois que les indicateurs de tableau de bord sont examinés.

Ce processus est itératif. Dans l’idéal, l’entreprise doit avoir une équipe responsable dédiée. Des réunions régulières doivent être organisées pour examiner les projets de correction existants, lancer l’évaluation de base des nouvelles charges de travail et effectuer le suivi du tableau de bord global de l’entreprise. L’équipe doit avoir l’autorité nécessaire pour demander des comptes aux équipes de correction qui ne respectent pas les métriques ou affichent des retards.

## <a name="structure-of-the-operational-fitness-review-team"></a>La structure de l’équipe chargée de l’évaluation de l’adéquation opérationnelle

L’équipe d’évaluation de l’adéquation opérationnelle se compose des rôles suivants :

1. **Responsable des résultats d’entreprise (Business Owner)**. Source de connaissances sur l’entreprise, il identifie et hiérarchise les opérations « critiques » pour l’entreprise. Il compare également le coût de prévention à l’impact sur l’entreprise et prend la décision finale concernant les mesures de correction à appliquer.

2. **Conseiller d’entreprise (Business Advocate)**. Il est chargé de décomposer les opérations d’entreprise en différentes parties correspondant à une infrastructure et des services cloud et locaux. Une connaissance approfondie des technologies associées à chaque opération est nécessaire.

3. **Responsable de l’ingénierie (Engineering Owner)**. Ce rôle est chargé d’implémenter les services associés à l’opération de l’entreprise. Ils peuvent participer à la conception, à la mise en œuvre et au déploiement de solutions à des problèmes liés aux exigences non fonctionnelles et révélés par l’équipe d’évaluation de l’adéquation opérationnelle.

4. **Responsable des services (Service Owner)**. Il est responsable du fonctionnement des applications et des services de l’entreprise. Ils collectent des données de journalisation et d’utilisation de ces applications et services, qui servent à identifier les problèmes et à vérifier les correctifs une fois déployés.

## <a name="operational-fitness-review-meeting"></a>Les réunions d’évaluation de l’adéquation opérationnelle

Nous recommandons d’organiser des réunions régulières au sein de l’équipe d’évaluation de l’adéquation opérationnelle. Elle peut par exemple se réunir tous les mois et envoyer à la direction un rapport trimestriel sur l’état et les métriques.

Les détails du processus et des réunions doivent être parfaitement adaptés à vos besoins. Nous recommandons les tâches suivantes pour commencer :

1. Le responsable des résultats d’entreprise et le conseiller d’entreprise listent et déterminent les exigences non fonctionnelles de chaque opération d’entreprise, avec la participation des responsables de l’ingénierie et des services. La priorité des opérations d’entreprise déjà identifiées est examinée et vérifiée. Pour ce qui est des nouvelles opérations, une priorité leur est attribuée dans la liste existante.

2. Les responsables de l’ingénierie et des services font correspondre **l’état actuel** des opérations d’entreprise aux services cloud et locaux associés : la liste des composants de chaque service est orientée sous la forme d’une arborescence des dépendances. Une fois la liste et l’arborescence générées, il convient de déterminer les **chemins critiques** dans l’arborescence.

3. Les responsables de l’ingénierie et des services évaluent l’état actuel de la journalisation et du monitoring opérationnels des services listés à l’étape précédente. Journalisation robuste et de surveillance sont critiques, afin d’identifier les composants de service qui contribuent à ne pas répondre aux exigences non fonctionnelles. Sinon, il faut créer et mettre en œuvre un plan permettant de les mettre en place.

4. Des indicateurs de tableau de bord sont créés pour les nouvelles opérations d’entreprise. Ce tableau de bord se compose de la liste des composants constitutifs de chacun des services identifiés à l’étape 2, alignés sur les exigences non fonctionnelles, et de métriques représentant la manière dont les composants respectent ces exigences.

5. Lorsqu’un composant constitutif ne parvient pas à répondre aux exigences non fonctionnelles, une solution de haut niveau est conçue et un responsable de l’ingénierie désigné. Le responsable des résultats d’entreprise et le conseiller d’entreprise doivent alors établir un budget pour le travail de correction, en fonction du revenu attendu de l’opération d’entreprise.

6. Enfin, le travail de correction en cours est soumis à une évaluation. Chacun des indicateurs de tableau de bord du travail en cours est comparé à la métrique attendue. Dans le cas des composants constitutifs qui satisfont aux indicateurs, le responsable des services présente les données de journalisation et de monitoring pour confirmer le respect de la métrique. Dans les autres cas, chaque responsable de l’ingénierie explique les problèmes en cause et les nouvelles conceptions permettant d’y remédier.

## <a name="recommended-resources"></a>Ressources recommandées

- [Piliers de la qualité logicielle](../../guide/pillars.md).
    Cette section du guide d’architecture des applications Azure décrit les cinq piliers de la qualité logicielle : scalabilité, disponibilité, résilience, gestion et sécurité.
- [Dix principes de conception pour les applications Azure](../../guide/design-principles/index.md).
    Cette section du guide d’architecture des applications Azure présente un ensemble de principes de conception visant à rendre les applications plus évolutives, plus résilientes et plus faciles à gérer.
- [Conception d’applications Azure fiables](../../reliability/index.md).
    Ce guide commence par une définition du terme « résilience » et des concepts associés. Il décrit ensuite un processus pour atteindre une résilience, à l’aide d’une approche structurée pendant la durée de vie d’une application, depuis la conception et l’implémentation jusqu’au déploiement et aux opérations.
- [Modèles de conception cloud](../../patterns/index.md).
    Ces modèles de conception sont utiles aux équipes d’ingénieurs qui souhaitent créer des applications selon les piliers de la qualité logicielle.
