---
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 80cb7fde0694257a5c413b702505e27f18aed8d3
ms.sourcegitcommit: d702b4d27e96e7a5a248dc4f2f0e25cf6e82c134
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/23/2018
ms.locfileid: "31771684"
---
# <a name="azure-application-architecture-guide"></a>Guide de l’architecture des applications Azure

Ce guide présente une approche structurée pour concevoir des applications sur Azure qui sont évolutives, robustes et hautement disponibles. Il est basé sur des pratiques éprouvées que nous avons apprises au contact des clients.

<br/>

<img src="./images/guide-steps.svg" style="max-width:800px;"/>

## <a name="introduction"></a>Introduction

Le cloud change la façon dont les applications sont conçues. À la place de monolithes, les applications sont décomposées en services plus petits et décentralisés. Ces services communiquent via des API ou à l’aide d’une messagerie ou d’une gestion des évènements asynchrone. Les applications sont mises à l’échelle horizontalement, en ajoutant de nouvelles instances lorsque c’est demandé. 

Ces tendances amènent de nouveaux défis. L’état de l’application est distribué. Les opérations sont effectuées en parallèle et de façon asynchrone. Le système dans son ensemble doit être résilient lorsque des échecs se produisent. Les déploiements doivent être automatisée et prévisibles. La surveillance et les données de télémétrie sont critiques pour obtenir des informations du système. Le guide de l’architecture des applications Azure est conçu pour vous aider à naviguer au travers de ces modifications. 

<table>
<thead>
    <tr><th>Traditionnel sur site</th><th>Cloud moderne</th></tr>
</thead>
<tbody>
<tr><td>Monolithique, centralisé<br/>
Conçu pour une extensibilité prévisible<br/>
Base de données relationnelle<br/>
Cohérence forte<br/>
Traitement de série et synchronisé<br/>
Conception pour éviter les échecs (MTBF)<br/>
Grandes mises à jour occasionnelles<br/>
Gestion manuelle<br/>
Serveurs en flocon</td>
<td>
Décomposé, décentralisé<br/>
Conception pour une mise à l’échelle élastique<br/>
Persistance polyglotte (combinaison de technologies de stockage)<br/>
Cohérence éventuelle<br/>
Traitement parallèle et asynchrone<br/>
Conception pour l’échec (MTTR)<br/>
Petites mises à jour fréquentes<br/>
Autogestion automatisée<br/>
Infrastructure immuable<br/>
</td>
</tbody>
</table>

Ce guide est destiné aux architectes, développeurs et aux équipes d’exploitation des applications. Il ne s’agit pas d’un guide de procédure montrant comment utiliser les services Azure individuels. Après avoir lu ce guide, vous comprendrez les modèles architecturaux et les meilleures pratiques à appliquer lors d’une conception sur la plateforme de Azure Cloud. Vous pouvez également télécharger une [version du livre électronique du guide][ebook].

## <a name="how-this-guide-is-structured"></a>Structure de ce guide

Le guide d’architecture des applications Azure est organisé comme une série d’étapes, depuis l’architecture et la conception jusqu’à l’implémentation. À chaque étape, des recommandations vous aideront pour la conception de l’architecture de votre application.

### <a name="architecture-styles"></a>Styles d’architecture

Le premier point de décision est le plus fondamental. Quel type d’architecture concevez-vous ? Cela peut être une architecture de microservices, une application multiniveau plus classique ou une solution Big Data. Nous avons identifié plusieurs styles d’architecture distincts. Voici les avantages et les problèmes de chacun.

En savoir plus :

- [Styles d’architecture][arch-styles]
- [Architectures de référence Azure][ref-archs]

### <a name="technology-choices"></a>Choix de technologie

Deux choix de technologie doivent être faits dès le départ car ils affectent l’ensemble de l’architecture. Il s’agit des choix des technologies de calcul et magasins de données. *Calcul* fait référence au modèle d’hébergement pour les ressources de calcul utilisées par vos applications. Les *magasins de données* comprennent les bases de données, mais également le stockage pour les files d’attente, les caches, les journaux et toutes les autres choses qu’une application peut stocker d’une façon persistante. 

En savoir plus :

- [Choisir un service de calcul](./technology-choices/compute-overview.md)
- [Choisir un magasin de données](./technology-choices/data-store-overview.md)

### <a name="design-principles"></a>Principes de conception

Nous avons identifié dix principes de conception de haut niveau qui rendront votre application plus évolutive, résiliente et gérable. Ces principes de conception s’appliquent à tous les styles architecture. Tout au long du processus de conception, gardez à l’esprit ces dix principes de conception de haut niveau. Envisagez l’ensemble de meilleures pratiques pour les aspects spécifiques de l’architecture, tels que la mise à l’échelle automatique, la mise en cache, le partitionnement des données, la conception d’API et d’autres encore.

En savoir plus :

- [Principes de conception pour les applications Azure][design-principles]
- [Meilleures pratiques lors de la génération sur le cloud][best-practices]

### <a name="quality-pillars"></a>Piliers de qualité

Une application cloud réussie se concentrera sur cinq piliers de qualité logicielle : l’extensibilité, la disponibilité, la résilience, la gestion et la sécurité. Utilisez nos checklists de révision de la conception pour passer en revue votre architecture en fonction de ces piliers de qualité.

En savoir plus :

- [Piliers de la qualité logicielle][pillars]
- [Checklists de révision de la conception][checklists] 

### <a name="cloud-design-patterns"></a>Modèles de conception de cloud

Les modèles de conception sont des solutions générales à des problèmes de conception logicielle courants. Nous avons identifié un ensemble de modèles de conception qui sont particulièrement utiles lors de la conception d’applications distribuées pour le cloud.

En savoir plus :

- [Catalogue de modèles de conception de cloud](../patterns/index.md)


[arch-styles]: ./architecture-styles/index.md
[best-practices]: ../best-practices/index.md
[checklists]: ../checklist/index.md
[compute-options]: ./technology-choices/compute-comparison.md
[design-principles]: ./design-principles/index.md
[ebook]: https://azure.microsoft.com/campaigns/cloud-application-architecture-guide/
[patterns]: ../patterns/index.md?toc=/azure/architecture/guide/toc.json
[pillars]: ./pillars.md
[ref-archs]: ../reference-architectures/index.md
[storage-options]: ./technology-choices/data-store-comparison.md
[technology-choices]: ./technology-choices/index.md

