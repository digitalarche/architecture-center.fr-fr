---
layout: LandingPage
ms.openlocfilehash: 00abbfdeac89a9006517195bd4bbc514d587fe74
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2017
---
# <a name="azure-application-architecture-guide"></a>Guide de l’architecture des applications Azure

Ce guide présente une approche structurée pour concevoir des applications sur Azure qui sont évolutives, robustes et hautement disponibles. Il est basé sur des pratiques éprouvées que nous avons apprises au contact des clients.

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

Ce guide est destiné aux architectes, développeurs et aux équipes d’exploitation des applications. Il ne s’agit pas d’un guide de procédure montrant comment utiliser les services Azure individuels. Après avoir lu ce guide, vous comprendrez les modèles architecturaux et les meilleures pratiques à appliquer lors d’une conception sur la plateforme de Azure Cloud.

## <a name="how-this-guide-is-structured"></a>Structure de ce guide

Le guide d’architecture des applications Azure est organisé comme une série d’étapes, depuis l’architecture et la conception jusqu’à l’implémentation. À chaque étape, des recommandations vous aideront pour la conception de l’architecture de votre application.

**[Styles d’architecture][arch-styles]**. Le premier point de décision est le plus fondamental. Quel type d’architecture concevez-vous ? Cela peut être une architecture de microservices, une application multiniveau plus classique ou une solution Big Data. Nous avons identifié sept styles d’architecture distincts. Voici les avantages et les problèmes de chacun.

> &#10148; La page [Architectures de référence Azure][ref-archs] montre les déploiements recommandés dans Azure, ainsi que des considérations pour l’extensibilité, la disponibilité, la facilité de gestion et la sécurité. La plupart incluent également des modèles Resource Manager à déployer.

**[Choix de technologie][technology-choices]**. Deux choix de technologie doivent être faits dès le départ car ils affectent l’ensemble de l’architecture. Il s’agit des choix des technologies de calcul et de stockage. Le terme *calcul* fait référence au modèle d’hébergement pour les ressources de calcul utilisées par vos applications. Le stockage comprend les bases de données, mais également le stockage pour les files d’attente, les caches, les données IoT, les données de journal non structurées et toutes les autres choses qu’une application peut stocker d’une façon persistante. 

> &#10148; Les [options de calcul][compute-options] et les [options de stockage][storage-options] fournissent des critères de comparaison détaillés pour sélectionner les services de calcul et de stockage.

**[Principes de conception][design-principles]**. Tout au long du processus de conception, gardez à l’esprit ces dix principes de conception de haut niveau. 

> &#10148; Les articles [Meilleures pratiques][best-practices] donnent des recommandations spécifiques sur des zones telles que la mise à l’échelle automatique, la mise en cache, le partitionnement des données, la conception de l’API et d’autres.   

**[Piliers][pillars]**. Une application cloud réussie se concentrera sur ces cinq piliers de la qualité des logiciels : l’extensibilité, la disponibilité, la résilience, la gestion et la sécurité. 

> &#10148; Utilisez nos [checklists de révision de la conception][checklists] pour passer en revue votre conception en fonction de ces piliers de qualité. 

**[Modèles de conception de cloud][patterns]**. Ces modèles de conception sont utiles pour développer des applications fiables, évolutives et sécurisées sur Azure. Chaque modèle décrit un problème, un modèle qui résout le problème et un exemple basé sur Azure.

> &#10148; Consultez le [catalogue complet des modèles de conception de cloud](../patterns/index.md).


[arch-styles]: ./architecture-styles/index.md
[best-practices]: ../best-practices/index.md
[checklists]: ../checklist/index.md
[compute-options]: ./technology-choices/compute-comparison.md
[design-principles]: ./design-principles/index.md
[patterns]: ../patterns/index.md?toc=/azure/architecture/guide/toc.json
[pillars]: ./pillars.md
[ref-archs]: ../reference-architectures/index.md
[storage-options]: ./technology-choices/data-store-comparison.md
[technology-choices]: ./technology-choices/index.md

