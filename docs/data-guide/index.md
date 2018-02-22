---
title: "Guide d’architecture des données Azure"
description: 
author: zoinerTejada
ms:date: 02/12/2018
layout: LandingPage
ms.openlocfilehash: 848601f27faf56ea069852d8983e4d10fbad9d77
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/14/2018
---
# <a name="azure-data-architecture-guide"></a>Guide d’architecture des données Azure

Ce guide présente une approche structurée pour concevoir des solutions orientées données sur Microsoft Azure. Il est basé sur des pratiques éprouvées que nous avons apprises au contact des clients.

## <a name="introduction"></a>Introduction

Le cloud change la façon dont les applications sont conçues, y compris la façon dont les données sont traitées et stockées. Au lieu d’une seule base de données à usage général qui gère toutes les données d’une solution, les solutions de _persistance polyglotte_ utilisent plusieurs banques de données spécialisées, chacune optimisée pour fournir des fonctions spécifiques. La perspective sur les données de la solution est par conséquent modifiée. Il ne s’agit plus de plusieurs couches de logique métier qui lisent et écrivent sur une couche de données unique. Les solutions sont plutôt conçues autour d’un *pipeline de données* qui décrit le flux des données via une solution, l’endroit où elles sont traitées, stockées et comment elles sont consommées par le composant suivant dans le pipeline. 

## <a name="how-this-guide-is-structured"></a>Structure de ce guide

Ce guide est structuré autour d’un pivot de base : la distinction entre données *relationnelles* et données *non relationnelles*. 

![](./images/guide-steps.svg)

Les données relationnelles sont généralement stockées dans un système SGBDR traditionnel ou un entrepôt de données. Elles disposent d’un schéma prédéfini (« schéma en écriture ») avec un ensemble de contraintes pour maintenir l’intégrité référentielle. La plupart des bases de données relationnelles utilisent le langage SQL (Structured Query Langage) pour l’interrogation. Les solutions qui utilisent des bases de données relationnelles incluent le traitement transactionnel en ligne (OLTP) et le traitement analytique en ligne (OLAP).

Les données non relationnelles sont toutes les données qui n’utilisent pas le [modèle relationnel](https://en.wikipedia.org/wiki/Relational_model) trouvé dans les systèmes SGBDR traditionnels. Cela peut inclure des données de la valeur de clé, des données JSON, un graphique de données, des données de la série chronologique et d’autres types de données. Le terme *NoSQL* fait référence à des bases de données conçues pour contenir différents types de données non relationnelles. Toutefois, le terme n’est pas vraiment précis, car plusieurs banques de données non relationnelles prennent en charge des requêtes SQL compatibles. Les données non relationnelles et les bases de données NoSQL utilisent souvent des discussions de solutions de *Big Data*. Une architecture Big Data est conçue pour gérer l’ingestion, le traitement et l’analyse de données trop volumineuses ou complexes pour les systèmes de base de données traditionnels. 

Dans chacune de ces deux catégories principales, le Guide d’architecture de données contient les sections suivantes :

- **Concepts.** Propose une vue d’ensemble des articles qui présentent les concepts principaux que vous devez comprendre lorsque vous travaillez avec ce type de données.
- **Scénarios.** Un jeu représentatif des scénarios de données, y compris une discussion sur les services Azure concernés et l’architecture appropriée pour le scénario.
- **Choix de technologie.** Comparaisons détaillées des différentes technologies de données disponibles sur Azure, y compris les options Open Source. Dans chaque catégorie, les critères de sélection clé et une matrice de capacité sont décrits pour vous aider à choisir la technologie adaptée à votre scénario.

Ce guide ne vise pas à vous enseigner les principes de la science des données ou des bases de données &mdash;, vous trouverez des livres entiers sur ces sujets. Au lieu de cela, l’objectif est de vous aider à sélectionner l’architecture de données ou le pipeline de données le mieux adapté à votre scénario, puis les services et les technologies Azure qui correspondent le mieux à vos besoins. Si vous avez déjà une architecture à l’esprit, vous pouvez passer directement aux choix de la technologie.

## <a name="traditional-rdbms"></a>SGBDR traditionnel

### <a name="concepts"></a>Concepts

- [Données relationnelles](./concepts/relational-data.md) 
- [Données transactionnelles](./concepts/transactional-data.md) 
- [Modélisation sémantique](./concepts/semantic-modeling.md) 

### <a name="scenarios"></a>Scénarios

- [Traitement analytique en ligne (OLAP)](./scenarios/online-analytical-processing.md)
- [Traitement transactionnel en ligne (OLTP)](./scenarios/online-transaction-processing.md) 
- [Entrepôt de données et mini-Data Warehouse](./scenarios/data-warehousing.md)
- [ETL](./scenarios/etl.md) 

## <a name="big-data-and-nosql"></a>Big Data et NoSQL

### <a name="concepts"></a>Concepts

- [Banques de données non relationnelles](./concepts/non-relational-data.md)
- [Utilisation des fichiers CSV et JSON](./concepts/csv-and-json.md)
- [Architectures de Big Data](./concepts/big-data.md)
- [Analytique avancée](./concepts/advanced-analytics.md) 
- [Machine Learning à l’échelle](./concepts/machine-learning-at-scale.md)

### <a name="scenarios"></a>Scénarios

- [Traitement par lots](./scenarios/batch-processing.md)
- [Traitement en temps réel](./scenarios/real-time-processing.md)
- [Recherche de texte de forme libre](./scenarios/search.md)
- [Exploration interactive des données](./scenarios/interactive-data-exploration.md)
- [Traitement en langage naturel](./scenarios/natural-language-processing.md)
- [Solutions de la série chronologique](./scenarios/time-series.md)

## <a name="cross-cutting-concerns"></a>Problèmes transversaux

- [Transfert de données](./scenarios/data-transfer.md) 
- [Extension des solutions de données locales vers le cloud](./scenarios/hybrid-on-premises-and-cloud.md) 
- [Sécurisation des solutions de données](./scenarios/securing-data-solutions.md) 
