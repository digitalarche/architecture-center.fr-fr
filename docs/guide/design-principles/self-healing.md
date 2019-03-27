---
title: Penser la conception des applications pour la réparation spontanée
titleSuffix: Azure Application Architecture Guide
description: Les applications résilientes peuvent opérer une récupération après un échec sans intervention manuelle.
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: 5e5af0be41fa892e490d556ef4286d5367144fd9
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58249504"
---
# <a name="design-for-self-healing"></a>Penser la conception des applications pour la réparation spontanée

## <a name="design-your-application-to-be-self-healing-when-failures-occur"></a>Concevez votre application pour qu’elle se répare spontanément en cas de défaillance

Dans un système distribué, des défaillances peuvent se produire. Le matériel peut être défaillant. Le réseau peut subir des échecs temporaires. Exceptionnellement, un service ou une région entiers peuvent rencontrer des interruptions, mais même ces dernières doivent être planifiées.

Il est donc essentiel de concevoir votre application pour qu’elle se répare spontanément en cas de défaillance. Cela nécessite une approche en trois étapes :

- Détecter les défaillances.
- Répondre aux défaillances de manière appropriée.
- Enregistrer et analyser les défaillances afin de bénéficier d’informations exploitables.

La manière dont vous répondez à un type particulier de défaillance peut dépendre des exigences de disponibilité de votre application. Par exemple, si vous avez besoin d’une très haute disponibilité, vous pouvez effectuer un basculement automatique dans une région secondaire en cas de panne régionale. Toutefois, cela implique un coût plus élevé qu’un déploiement dans une seule région.

En outre, ne pensez pas uniquement aux événements importants tels que les pannes de courant régionales qui sont généralement rares. Vous devez vous focaliser tout autant, voire davantage, sur la gestion des défaillances locales et de courte durée, telles que les défaillances de connectivité réseau ou les échecs de connexion aux bases de données.

## <a name="recommendations"></a>Recommandations

**Relance des opérations ayant échoué**. Les échecs temporaires peuvent se produire en raison d’une perte momentanée de la connectivité réseau, d’une connexion à une base de données interrompue ou d’un délai d’attente lorsqu’un service est occupé. Intégrez une logique de relance à votre application pour gérer les défaillances temporaires. Pour de nombreux services Azure, le Kit de développement logiciel client implémente de nouvelles tentatives automatiques. Pour plus d’informations, consultez [Gestion des erreurs temporaires][transient-fault-handling] et le [Modèle Nouvelle tentative][retry].

**Protéger les services distants ayant échoué (Disjoncteur)**. Il est judicieux de lancer de nouvelles tentatives après une défaillance temporaire, mais si le problème persiste, vous pouvez vous retrouver avec un trop grand nombre d’appels vers le service défectueux. Cela peut entraîner des échecs en cascade à mesure que les requêtes sont sauvegardées. Utilisez le [modèle Disjoncteur][circuit-breaker] pour effectuer un Fail-fast (sans effectuer l’appel distant) quand l’échec d’une opération est susceptible de se produire.

**Isoler les ressources critiques (Cloisonnement)**. Un sous-système peut parfois être victime de défaillances en cascade. Cela peut se produire si une défaillance empêche que certaines ressources, telles que des threads ou des sockets, ne soient libérées en temps voulu, menant à un épuisement des ressources. Pour éviter ce problème, partitionnez un système en groupes isolés, de manière à ce qu’une défaillance présente dans une partition ne détériore pas l’ensemble du système.

**Effectuer un nivellement de la charge**. Les applications peuvent rencontrer des pics soudains dans le trafic qui peuvent surcharger les services sur le serveur principal. Pour éviter cela, utilisez le [Modèle de nivellement de charge basé sur une file d’attente][load-level] afin de mettre en file d’attente les éléments de travail pour qu’ils s’exécutent de manière asynchrone. La file d’attente agit comme une mémoire tampon qui lisse des pics de charge.

**Effectuer un basculement**. Si une instance ne peut pas être atteinte, effectuez un basculement vers une autre instance. Pour les éléments sans état, tels qu’un serveur web, placez plusieurs instances derrière un équilibreur de charge ou un gestionnaire de trafic. Pour les éléments avec état, tels qu’une base de données, utilisez des réplicas et le basculement. Selon la banque de données et la manière dont elle est répliquée, l’application peut avoir besoin de mettre en place la cohérence éventuelle.

**Compenser les transactions qui ont échoué**. En règle générale, évitez les transactions distribuées, car elles requièrent une coordination entre les services et les ressources. Au lieu de cela, concevez une opération à partir de transactions individuelles de plus petite taille. Si l’opération échoue en cours de traitement, utilisez des [transactions de compensation][compensating-transactions] pour annuler les étapes qui ont déjà été effectuées.

**Créer des points de contrôle pour les transactions de longue durée**. Les points de contrôle peuvent fournir la résilience en cas d’échec d’une opération longue. Lors du redémarrage de l’opération (par exemple si elle est récupérée par une autre machine virtuelle), elle peut être reprise à partir du dernier point de contrôle.

**Appliquer une dégradation normale**. Parfois, vous ne pouvez pas résoudre le problème, mais vous pouvez fournir des fonctionnalités réduites qui s’avèrent toujours utiles. Imaginez une application qui propose un catalogue de livres. Si l’application ne peut pas récupérer l’image miniature de la couverture, elle affichera une image de substitution. Des sous-systèmes entiers peuvent s’avérer non critiques pour l’application. Par exemple, dans le cas d’un site de commerce en ligne, l’affichage des recommandations de produits est probablement moins important que le traitement des commandes.

**Limiter les clients**. Parfois, seuls quelques utilisateurs créent une charge excessive, ce qui peut réduire la disponibilité de votre application pour les autres utilisateurs. Dans ce cas, limitez le client pendant un certain temps. Consultez le [Modèle de limitation][throttle].

**Bloquer les acteurs malveillants**. Le fait de limiter un client ne signifie pas que celui-ci agissait de façon malveillante. Cela signifie simplement que le client a dépassé son quota de service. Mais si un client dépasse constamment son quota ou se comporte de manière incorrecte, vous pouvez le bloquer. Définissez un processus hors-bande pour que l’utilisateur puisse demander à être débloqué.

**Utiliser la fonction d’élection du responsable**. Lorsque vous avez besoin de coordonner une tâche, utilisez la fonction [Élection du responsable][leader-election] pour sélectionner un coordinateur. De cette façon, le coordinateur ne constitue pas un point de défaillance unique. Si le coordinateur échoue, un autre est sélectionné. Au lieu d’implémenter vous-même un algorithme d’élection du responsable, envisagez d’adopter une solution prête à l’emploi telle que Zookeeper.

**Procéder à des tests avec injection d’erreurs**. Trop souvent, on teste le chemin d’accès en cas de réussite mais pas celui en cas d’échec. Un système peut s’exécuter en mode de production pendant longtemps avant qu’un chemin d’échec ne soit utilisé. Utilisez l’injection d’erreurs pour tester la résilience du système lors de pannes, en déclenchant des échecs réels ou simulés.

**Adopter l’ingénierie du chaos**. L’ingénierie du chaos étend la notion d’injection d’erreurs en injectant de manière aléatoire des défaillances ou des conditions anormales dans les instances de production.

Pour mettre en place une approche structurée permettant la réparation spontanée de vos applications, consultez l’article [Conception d’applications résilientes pour Azure][resiliency-overview].

<!-- links -->

[circuit-breaker]: ../../patterns/circuit-breaker.md
[compensating-transactions]: ../../patterns/compensating-transaction.md
[leader-election]: ../../patterns/leader-election.md
[load-level]: ../../patterns/queue-based-load-leveling.md
[resiliency-overview]: ../../resiliency/index.md
[retry]: ../../patterns/retry.md
[throttle]: ../../patterns/throttling.md
[transient-fault-handling]: ../../best-practices/transient-faults.md
