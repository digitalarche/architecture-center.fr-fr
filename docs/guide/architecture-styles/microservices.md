---
title: Style d’architecture de microservices
description: Décrit les avantages, les inconvénients et les bonnes pratiques pour les architectures de microservices sur Azure
author: MikeWasson
ms.date: 11/13/2018
ms.openlocfilehash: 4e5d50f829323829c953977257e690354566ebf6
ms.sourcegitcommit: 19a517a2fb70768b3edb9a7c3c37197baa61d9b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2018
ms.locfileid: "52295529"
---
# <a name="microservices-architecture-style"></a>Style d’architecture de microservices

Une architecture de microservices se compose d’un ensemble de petits services autonomes. Chaque service est autonome et doit implémenter une fonctionnalité unique. 

![](./images/microservices-logical.svg)
 
À certains égards, les microservices sont l’évolution naturelle des architectures orientées services (SOA), mais il existe des différences. Voici certaines caractéristiques qui définissent un microservice :

- Dans une architecture de microservices, les services sont de petite taille, indépendants et faiblement liés.

- Chaque service est un code base distinct, qui peut être géré par une petite équipe de développement.

- Les services peuvent être déployés de manière indépendante. Une équipe peut mettre à jour un service existant sans avoir à recréer et redéployer l’application entière.

- Les services sont responsables de la persistance de leurs propres données ou de leur propre état externe. Ils se distinguent en cela du modèle traditionnel dans lequel une couche de données distincte gère la persistance des données.

- Les services communiquent entre eux à l’aide d’API bien définies. Les détails de l’implémentation interne de chaque service sont cachés aux autres services.

- Les services n’ont pas besoin de partager une pile de technologies, des bibliothèques ou des frameworks identiques.

Outre les services proprement dits, une architecture de microservices type est constituée d’autres composants :

**Gestion** : le composant de gestion est chargé de placer les services sur les nœuds, d’identifier les défaillances, de rééquilibrer les services entre les nœuds, et ainsi de suite.  

**Découverte des services** :  tient à jour une liste des services et des nœuds sur lesquels ils se trouvent. Permet de rechercher le point de terminaison d’un service. 

**Passerelle d’API** : la passerelle d’API est le point d’entrée des clients. Les clients n’appellent pas les services directement. Au lieu de cela, ils appellent la passerelle d’API, qui transfère l’appel aux services appropriés sur le backend. La passerelle d’API peut agréger les réponses de plusieurs services et retourner la réponse agrégée. 

L’utilisation de la passerelle d’API offre les avantages suivants :

- Il dissocie les clients des services. Les services peuvent être versionnés ou refactorisés sans avoir à mettre à jour tous les clients.

-  Les services peuvent utiliser des protocoles de messagerie qui ne sont pas compatibles avec le web, comme AMQP.

- La passerelle d’API peut exécuter d’autres fonctions transversales, telles que l’authentification, la journalisation, la terminaison SSL et l’équilibrage de charge.

## <a name="when-to-use-this-architecture"></a>Quand utiliser cette architecture

Envisagez ce style d’architecture pour :

- Les applications volumineuses qui exigent une grande rapidité de diffusion.

- Les applications complexes qui doivent offrir une grande scalabilité.

- Les applications dotées de domaines étoffés ou d’un grand nombre de sous-domaines.

- Une organisation constituée de petites équipes de développement.


## <a name="benefits"></a>Avantages 

- **Déploiements indépendants** : vous pouvez mettre à jour un service sans avoir à redéployer l’application entière et effectuer une restauration ou une restauration par progression si une mise à jour est une source de problèmes. Les résolutions de bogues et les diffusions de fonctionnalités sont plus faciles à gérer et moins risquées.

- **Développement indépendant** : une même équipe de développement peut créer, tester et déployer un service. Résultat : l’innovation est continue et la cadence des diffusions est supérieure. 

- **Petites équipes spécialisées** : les équipes peuvent se concentrer sur un seul service. la portée de chaque service étant plus limitée, le code base est plus facile à comprendre et les nouveaux membres d’équipes s’intègrent plus facilement.

- **Isolation des erreurs** : si un service tombe en panne, il n’impacte pas l’ensemble de l’application. Cependant, cela ne signifie pas pour autant que vous bénéficiez d’une résilience ex nihilo. Vous devez toujours suivre les bonnes pratiques de résilience et les modèles de conception. Consultez [Conception d’applications résilientes pour Azure][resiliency-overview].

- **Piles de technologies mixtes** : les équipes peuvent opter pour les technologies qui se prêtent le mieux à leur service. 

- **Mise à l’échelle précise** : les services peuvent mis à l’échelle de manière indépendante. Dans le même temps, la plus grande densité des services par machine virtuelle signifie que les ressources des machines virtuelles sont entièrement exploitées. En utilisant des contraintes de placement, un service peut être assorti à un profil de machine virtuelle (processeur rapide, mémoire volumineuse, etc.).

## <a name="challenges"></a>Défis

- **Complexité** : une application de microservices possède plus d’éléments mobiles qu’une application monolithique équivalente. Si chaque service est plus simple, le système dans son ensemble est plus complexe.

- **Développement et tests** : développer avec des dépendances de services demande une approche différente. Les outils existants ne sont pas nécessairement conçus pour fonctionner avec des dépendances de services. Refactoriser au-delà des limites des services peut être une tâche ardue. Il est aussi difficile de tester les dépendances de services, en particulier quand l’application évolue rapidement.

- **Manque de gouvernance** : l’approche décentralisée de création de microservices présente des avantages, mais elle peut aussi être une source de problèmes. Vous pouvez en effet vous trouver face à une diversité de langages et de frameworks telle que l’application devient difficile à gérer. Il peut être utile de mettre en place certains standards à l’échelle du projet, sans trop amoindrir la flexibilité des équipes. Cela vaut en particulier pour les fonctionnalités transversales, telles que la journalisation.

- **Surcharge du réseau et latence** : l’utilisation de nombreux services granulaires de petite taille peut se traduire par une intensification des communications entre les services. De même, si la chaîne des dépendances de services devient trop longue (le service A appelle le service B, qui appelle le service C, etc.), la latence supplémentaire qui s’ensuit peut devenir un problème. Vous devrez faire preuve de circonspection au moment de concevoir des API. Évitez les API trop bavardes, envisagez des formats de sérialisation et cherchez à utiliser des modèles de communication asynchrone.

- **Intégrité des données** : chaque microservice étant responsable de la persistance de ses propres données, la cohérence des données peut être une gageure. Adoptez la cohérence éventuelle dans la mesure du possible.

- **Gestion** : une expérience réussie avec les microservices exige une culture DevOps mature. La mise en place d’une journalisation corrélée entre les services peut poser des problèmes. En règle générale, la journalisation doit mettre en corrélation plusieurs appels de services pour une même opération utilisateur.

- **Gestion des versions** : les mises à jour apportées à un service ne doivent pas perturber les services qui en dépendent. Plusieurs services pouvant être mis à jour à tout moment, sans une conception minutieuse, vous risquez de rencontrer des problèmes de compatibilité descendante ou ascendante.

- **Compétences** : les microservices sont des systèmes hautement distribués. Évaluez avec soin les chances de réussite en tenant compte des compétences et de l’expérience de l’équipe.

## <a name="best-practices"></a>Meilleures pratiques

- Modélisez les services autour du domaine de l’entreprise. 

- Décentralisez tout. Les équipes individuelles sont chargées de concevoir et de créer des services. Évitez de partager du code ou des schémas de données. 

- Le stockage de données doit être privé par rapport au service qui détient les données. Utilisez le stockage le mieux adapté à chaque service et type de données. 

- Les services communiquent via des API bien conçues. Évitez de divulguer les détails d’une implémentation. Les API doivent modéliser le domaine, et non l’implémentation interne du service.

- Évitez le couplage entre les services. Le couplage trouve souvent son origine dans les schémas de base de données partagés et les protocoles de communication rigides.

- Déchargez les problèmes transversaux, tels que l’authentification et la terminaison SSL, sur la passerelle.

- Maintenez la connaissance du domaine à l’extérieur de la passerelle. La passerelle doit traiter et acheminer les demandes des clients sans connaître les règles d’entreprise ou la logique du domaine. Sinon, la passerelle devient une dépendance et peut provoquer un couplage entre les services.

- Le couplage entre les services doit être faible et leur cohésion fonctionnelle élevée. Les fonctions susceptibles de changer en même temps doivent être empaquetées et déployées ensemble. Si elles résident dans des services distincts, ces services finissent par être fortement couplés, car une modification dans un service nécessite la mise à jour de l’autre service. Une communication proéminente entre deux services peut être le signe d’un couplage étroit et d’une faible cohésion. 

- Isolez les défaillances. Utilisez des stratégies de résilience pour empêcher que les défaillances au sein d’un service se répercutent en cascade. Consultez [Modèles de résilience][resiliency-patterns] et [Conception d’applications résilientes][resiliency-overview].

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir des instructions détaillées sur la création d’une architecture de microservices sur Azure, consultez [Conception, génération et exploitation de microservices sur Azure](../../microservices/index.md).


<!-- links -->

[resiliency-overview]: ../../resiliency/index.md
[resiliency-patterns]: ../../patterns/category/resiliency.md



