---
title: Guide d’architecture des données Azure
description: null
author: zoinerTejada
ms.date: 02/12/2018
layout: LandingPage
ms.topic: landing-page
ms.service: architecture-center
ms.subservice: cloud-fundamentals
---

# <a name="azure-data-architecture-guide"></a>Guide d’architecture des données Azure

Ce guide présente une approche structurée pour concevoir des solutions orientées données sur Microsoft Azure. Il est basé sur des pratiques éprouvées que nous avons apprises au contact des clients.

## <a name="introduction"></a>Introduction

Le cloud change la façon dont les applications sont conçues, y compris la façon dont les données sont traitées et stockées. Au lieu d’une seule base de données à usage général qui gère toutes les données d’une solution, les solutions de _persistance polyglotte_ utilisent plusieurs banques de données spécialisées, chacune optimisée pour fournir des fonctions spécifiques. La perspective sur les données de la solution est par conséquent modifiée. Il ne s’agit plus de plusieurs couches de logique métier qui lisent et écrivent sur une couche de données unique. Les solutions sont plutôt conçues autour d’un *pipeline de données* qui décrit le flux des données via une solution, l’endroit où elles sont traitées, stockées et comment elles sont consommées par le composant suivant dans le pipeline.

## <a name="how-this-guide-is-structured"></a>Structure de ce guide

Ce guide se structure autour de deux catégories de solution de données, les *charges de travail SGBDR traditionnelles* et les *solutions Big Data*.

**[Charges de travail SGBDR traditionnelles](./relational-data/index.md)**. Ces solutions incluent le traitement transactionnel en ligne (OLTP) et le traitement analytique en ligne (OLAP). Les données dans les systèmes OLTP sont en général des données relationnelles suivant un schéma prédéfini et un ensemble de contraintes afin de maintenir l’intégrité référentielle. Souvent, les données provenant de plusieurs sources dans l’organisation peuvent être consolidées dans un entrepôt de données, à l’aide d’un processus ETL pour déplacer et transformer les données source.

![Charges de travail SGBDR traditionnelles](./images/guide-rdbms.svg)

**[Solutions Big Data](./big-data/index.md)**. Une architecture Big Data est conçue pour gérer l’ingestion, le traitement et l’analyse de données trop volumineuses ou complexes pour les systèmes de base de données traditionnels. Les données peuvent être traitées par lot ou en temps réel. Les solutions Big data impliquent généralement une grande quantité de données non relationnelles, telles que les données clé-valeur, les documents JSON ou les données Time Series. Les systèmes SGBDR traditionnels sont souvent peu adaptés pour stocker ce type de données. Le terme *NoSQL* fait référence à une famille de bases de données conçues pour contenir des données non relationnelles. (Le terme n’est pas vraiment exact, car plusieurs banques de données non relationnelles prennent en charge des requêtes SQL compatibles.)

![Solutions Big Data](./images/guide-big-data.svg)

Ces deux catégories ne sont pas mutuellement exclusives et ne se chevauchent pas entre elles, mais nous pensons qu’elles constituent un cadre utile pour cette discussion. Pour chaque catégorie, le guide aborde des **scénarios courants**, et inclue les services Azure concernés et l’architecture appropriée pour le scénario. En outre, le guide compare les **choix technologiques** pour les solutions de données dans Azure, y compris les options open source. Dans chaque catégorie, les critères de sélection clé et une matrice de capacité sont décrits pour vous aider à choisir la technologie adaptée à votre scénario.

Ce guide ne vise pas à vous enseigner les principes de la science des données ou des bases de données &mdash;, vous trouverez des livres entiers sur ces sujets. Au lieu de cela, l’objectif est de vous aider à sélectionner l’architecture de données ou le pipeline de données le mieux adapté à votre scénario, puis les services et les technologies Azure qui correspondent le mieux à vos besoins. Si vous avez déjà une architecture à l’esprit, vous pouvez passer directement aux choix de la technologie.
