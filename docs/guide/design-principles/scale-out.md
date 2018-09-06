---
title: Penser la conception des applications pour augmenter la taille des instances
description: Les applications cloud doivent être conçues pour une mise à l’échelle horizontale.
author: MikeWasson
ms.date: 08/30/2018
ms.openlocfilehash: 9b57f4e6a17eece4f5283436e104c286602bb54f
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/31/2018
ms.locfileid: "43325652"
---
# <a name="design-to-scale-out"></a>Penser la conception des applications pour augmenter la taille des instances

## <a name="design-your-application-so-that-it-can-scale-horizontally"></a>Concevoir votre application de manière à permettre sa mise à l’échelle horizontale

Le principal avantage du cloud est la mise à l’échelle élastique &mdash; la possibilité d’utiliser exactement la capacité dont vous avez besoin, en effectuant une montée en charge à mesure que la charge augmente et une mise à l’échelle lorsque vous n’avez plus besoin de cette capacité supplémentaire. Concevez votre application afin qu’elle puisse évoluer horizontalement, par l’ajout ou la suppression de nouvelles instances en fonction de la demande.

## <a name="recommendations"></a>Recommandations

**Évitez l’adhérence des instances**. L’adhérence, ou *affinité de session*, intervient lorsque les requêtes émanant du même client sont toujours routées vers le même serveur. L’adhérence limite les possibilités de montée en charge de l’application. Par exemple, le trafic provenant d’un utilisateur générant beaucoup de volume ne sera pas distribué entre les différentes instances. L’adhérence peut être due au stockage de l’état de session en mémoire et à l’utilisation de clés spécifiques à l’ordinateur pour le chiffrement. Assurez-vous que n’importe quelle instance peut gérer n’importe quelle requête. 

**Identifiez les goulots d’étranglement**. La montée en puissance ne règle pas tous les problèmes de performance. Par exemple, si votre base de données principale est le goulot d’étranglement, ajouter des serveurs web ne servira à rien. Identifiez et éliminez les goulots d’étranglement au niveau du système avant de multiplier les instances. Les parties avec état du système sont les causes les plus courantes des goulots d’étranglement. 

**Décomposez les charges de travail par exigences d’évolutivité.**  Les applications se composent souvent de plusieurs charges de travail, avec des exigences différentes en matière d’évolutivité. Par exemple, une application peut avoir un site web public et un site d’administration distinct. Le site public peut rencontrer des pics de trafic soudains, alors que le site d’administration a une charge moins élevée et plus prévisible. 

**Déchargez les tâches gourmandes en ressources.** Le cas échéant, les tâches qui nécessitent beaucoup de ressources de processeur ou d’E/S doivent être déplacées vers des [tâches en arrière-plan][background-jobs] afin de réduire la charge au niveau du serveur frontal qui traite les requêtes d’utilisateur.

**Utilisez les fonctionnalités de mise à l’échelle automatique intégrées**. De nombreux services de calcul Azure offrent une prise en charge intégrée de la mise à l’échelle automatique. Si l’application a une charge de travail régulière et prévisible, nous vous recommandons de planifier la montée en charge, par exemple pendant les heures ouvrables. Sinon, lorsque la charge de travail n’est pas prévisible, utilisez les indicateurs de performance tels que la longueur de file d’attente du processeur ou des requêtes pour déclencher la mise à l’échelle automatique. Pour connaître les bonnes pratiques en matière de mise à l’échelle automatique, consultez [Mise à l’échelle automatique][autoscaling].

**Envisagez une mise à l’échelle automatique agressive pour les charges de travail critiques**. Pour les charges de travail critiques, il est essentiel d’anticiper la demande. Il est préférable d’ajouter de nouvelles instances rapidement en cas de charge importante afin de pouvoir gérer le trafic supplémentaire, puis de réduire progressivement le nombre d’instances.

**Pensez la conception des applications pour réduire la taille des instances**.  N’oubliez pas que dans le cadre d’une évolutivité élastique, l’application connaîtra des périodes de mise à l’échelle, lorsque des instances sont supprimées. L’application doit gérer de manière appropriée les instances en cours de suppression. Voici quelques méthodes de gestion de la mise à l’échelle :

- Détectez les événements d’arrêt (le cas échéant) et arrêtez correctement les instances. 
- Les clients/consommateurs d’un service doivent prendre en charge les nouvelles tentatives et la gestion des erreurs temporaires. 
- Pour les tâches longues, envisagez de fragmenter le travail à l’aide de points de contrôle ou du modèle [Canaux et filtres][pipes-filters-pattern]. 
- Placez les éléments de travail dans une file d’attente afin qu’une autre instance puisse s’en charger dans le cas où une instance est supprimée au cours du processus de traitement. 


<!-- links -->

[autoscaling]: ../../best-practices/auto-scaling.md
[background-jobs]: ../../best-practices/background-jobs.md
[pipes-filters-pattern]: ../../patterns/pipes-and-filters.md
