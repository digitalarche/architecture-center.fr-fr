---
title: "Utiliser la meilleure banque de données pour le travail"
description: "Choisissez la technologie de stockage la mieux adaptée à vos données et son mode d’utilisation"
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: ef9439f7a3766d13b498eac915e0f5afd23de4e2
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2017
---
# <a name="use-the-best-data-store-for-the-job"></a>Utiliser la meilleure banque de données pour le travail

## <a name="pick-the-storage-technology-that-is-the-best-fit-for-your-data-and-how-it-will-be-used"></a>Choisissez la technologie de stockage la mieux adaptée à vos données et son mode d’utilisation

Le temps où vous stockiez toutes vos données dans une importante base de données relationnelle SQL est révolu. Les bases de données relationnelles remplissent parfaitement leur rôle &mdash; offrir des garanties ACID pour les transactions reposant sur des données relationnelles. Mais ce n’est pas sans inconvénients :

- Les requêtes peuvent nécessiter des jointures coûteuses.
- Les données doivent être normalisées et conformes à un schéma prédéfini (schéma en écriture).
- La contention de verrouillage peut affecter les performances.

Dans toute solution de grande envergure, il est peu probable qu’une technologie de banque de données unique réponde à tous vos besoins. Les alternatives aux bases de données relationnelles incluent des banques de clés/valeurs, des bases de données de documents, des bases de données de moteurs de recherche, des bases de données de séries chronologiques, des bases de données de familles de colonnes et des bases de données graphiques. Chacune de ces alternatives a ses avantages et ses inconvénients, et sera plus ou moins adaptée en fonction du type de données. 

Par exemple, vous pouvez stocker un catalogue de produits dans une base de données de documents, telle que Cosmos DB, qui offre un schéma flexible. Dans ce cas, chaque description de produit constitue un document autonome. Pour lancer des requêtes sur la totalité du catalogue, vous devrez peut-être indexer le catalogue et stocker l’index dans Azure Search. L’inventaire de produits peut être stocké dans une base de données SQL, car ces données requièrent des garanties ACID.

N’oubliez pas que les données représentent bien plus que les données d’application persistantes. Elles incluent également les journaux des applications, les événements, les messages et les caches.

## <a name="recommendations"></a>Recommandations

**N’utilisez pas une base de données relationnelle pour tous les types de données**. Envisagez d’adopter d’autres banques de données le cas échéant. Voir [Choisir le magasin de données correct][data-store-overview].

**Adopter la persistance polyglotte**. Dans toute solution de grande envergure, il est peu probable qu’une technologie de banque de données unique réponde à tous vos besoins. 

**Tenez compte du type de données**. Par exemple, placez les données transactionnelles dans SQL, les documents JSON dans une base de données de documents, les données de télémétrie dans une base de données de séries chronologiques, les journaux des applications dans Elasticsearch et les objets blob dans le Stockage Blob Azure.

**Préférez la disponibilité à la cohérence (forte)**. Le théorème CAP implique qu’un système distribué doit trouver un compromis entre la disponibilité et la cohérence. (Les partitions réseau, l’autre élément principal du théorème CAP, ne peuvent jamais être totalement ignorées.) Souvent, vous pouvez obtenir une meilleure disponibilité en adoptant un modèle de *cohérence éventuelle*. 

**Prenez en compte les compétences de l’équipe de développement**. Si la persistance polyglotte offre de nombreux avantages, elle peut également s’avérer très complexe. L’adoption d’une nouvelle technologie de stockage de données requiert un nouvel ensemble de compétences. L’équipe de développement doit comprendre comment tirer le meilleur parti de la technologie. Ils doivent découvrir les modèles d’utilisation appropriés, savoir comment optimiser les requêtes, ajuster les performances, etc. Prenez en compte ces facteurs lors de l’adoption de technologies de stockage. 

**Utilisez des transactions de compensation**. Avec la persistance polyglotte, il est possible qu’une transaction unique écrive des données dans plusieurs banques de données. En cas de problème, utilisez des transactions de compensation pour annuler toutes les étapes qui ont déjà été effectuées.

**Examinez les limites de contexte**. Le terme *Limite de contexte* est associé à la conception axée sur le domaine. Une limite de contexte est une limite explicite autour d’un modèle de domaine qui définit les parties du domaine auxquelles s’applique le modèle. Dans l’idéal, une limite de contexte correspond à un sous-domaine du domaine d’entreprise. Les limites de contexte dans votre système représentent un facteur important dans l’adoption de la persistance polyglotte. Par exemple, les « produits » peuvent apparaître dans le sous-domaine Catalogue de produits et le sous-domaine Inventaire de produits, mais il est très probable que ces deux sous-domaines présentent des exigences différentes en matière de stockage, de mise à jour et de recherche de produits.

[data-store-overview]: ../technology-choices/data-store-overview.md