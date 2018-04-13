---
title: Modèle de couche de lutte contre la corruption
description: Implémentez une couche de façade ou d’adaptateur entre une application moderne et un système hérité.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: efb1f90be33c2621c7a24c42730da9fffe70dfad
ms.sourcegitcommit: ea7108f71dab09175ff69322874d1bcba800a37a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/17/2018
---
# <a name="anti-corruption-layer-pattern"></a>Modèle de couche de lutte contre la corruption

Implémentez une couche de façade ou d’adaptateur entre différents sous-systèmes qui ne partagent pas la même sémantique. Cette couche traduit les requêtes qu’un sous-système envoie à l’autre sous-système. Utilisez ce modèle pour vous assurer que la conception d’une application n’est pas limitée par les dépendances aux sous-systèmes externes. Ce modèle a d’abord été décrit par Eric Evans dans *Domain-Driven Design* (Conception orientée domaine).

## <a name="context-and-problem"></a>Contexte et problème

La plupart des applications s’appuient sur d’autres systèmes pour certaines données ou fonctionnalités. Par exemple, lorsqu’une application héritée est migrée vers un système moderne, elle peut toujours avoir besoin de ressources héritées existantes. De nouvelles fonctionnalités doivent être en mesure d’appeler le système hérité. Cela est particulièrement vrai lorsqu’il s’agit de migrations graduelles pour lesquelles différentes fonctionnalités d’une application plus importante sont déplacées vers un système moderne au fil du temps.

Souvent, ces systèmes hérités connaissent des problèmes de qualité comme des schémas de données complexes ou des API obsolètes. Les fonctionnalités et technologies utilisées dans les systèmes hérités peuvent varier considérablement par rapport à des systèmes plus modernes. Pour interagir avec le système hérité, la nouvelle application doit éventuellement prendre en charge une infrastructure, des protocoles, des modèles de données, des API obsolètes ou d’autres fonctionnalités que vous n’implémenteriez pas dans une application moderne.

Maintenir l’accès entre les systèmes nouveaux et hérités peut forcer le nouveau système à respecter au moins certaines API ou d’autres sémantiques du système hérité. Lorsque ces fonctionnalités héritées ont des problèmes de qualité, le fait de les prendre en charge « corrompt » ce qui pourrait être une application moderne correctement conçue. 

Des problèmes similaires peuvent se produire avec n’importe quel système externe que votre équipe de développement ne contrôle pas, pas uniquement avec les systèmes hérités. 

## <a name="solution"></a>Solution

Isolez les différents sous-systèmes en plaçant une couche de lutte contre la corruption entre eux. Cette couche traduit les communications entre les deux systèmes, ce qui permet à un système de rester identique alors que l’autre système peut éviter de compromettre sa conception et son approche technologique.

![](./_images/anti-corruption-layer.png) 

Le diagramme ci-dessus illustre une application comprenant deux sous-systèmes. Le sous-système A appelle le sous-système B à travers une couche de lutte contre la corruption. La communication entre le sous-système A et la couche de lutte contre la corruption utilise toujours le modèle de données et l’architecture du sous-système A. Les appels à partir de la couche de lutte contre la corruption vers le sous-système B sont conformes au modèle de données ou aux méthodes de ce sous-système. La couche de lutte contre la corruption contient toute la logique nécessaire pour effectuer la traduction entre les deux systèmes. La couche peut être implémentée en tant que composant dans l’application ou en tant que service indépendant.

## <a name="issues-and-considerations"></a>Problèmes et considérations

- La couche de lutte contre la corruption peut ajouter de la latence aux appels effectués entre les deux systèmes.
- La couche de lutte contre la corruption ajoute un service supplémentaire qui doit être géré et maintenu.
- Réfléchissez à la façon dont votre couche de lutte contre la corruption va être mise à l’échelle.
- Évaluez si vous avez besoin de plus d’une couche de lutte contre la corruption. Vous pouvez décomposer les fonctionnalités en plusieurs services à l’aide de différents langages ou technologies ; vous pouvez également choisir de partitionner la couche de lutte contre la corruption pour d’autres raisons.
- Réfléchissez à la façon dont la couche de lutte contre la corruption sera gérée par rapport à vos autres applications ou services. Comment sera-t-elle intégrée à vos processus de surveillance, de mise en production et de configuration ?
- Assurez-vous que la cohérence des transactions et des données est conservée et qu’elle peut être surveillée.
- Déterminez si la couche de lutte contre la corruption doit gérer toutes les communications entre différents sous-systèmes ou uniquement un sous-ensemble de fonctionnalités. 
- Si la couche de lutte contre la corruption fait partie d’une stratégie de migration d’application, déterminez si elle sera permanente, ou si elle sera mise hors service une fois toutes les fonctionnalités hérités migrées.

## <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle

Utilisez ce modèle dans les situations suivantes :

- Une migration est planifiée pour avoir lieu en plusieurs étapes, mais l’intégration entre les systèmes nouveaux et hérités doit être maintenue.
- Deux ou plusieurs sous-systèmes ont des sémantiques différentes, mais doivent tout de même communiquer. 

Ce modèle peut ne pas convenir s’il n’y a aucune différence de sémantique significative entre les systèmes nouveaux et hérités. 

## <a name="related-guidance"></a>Aide connexe

- [Modèle d’étranglement](./strangler.md)
