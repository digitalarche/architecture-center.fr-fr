---
title: Vue matérialisée
description: Générez des vues préremplies sur les données d’un ou de plusieurs magasins de données lorsque les données ne sont pas adéquatement formatées pour les opérations de requête requises.
keywords: modèle de conception
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- performance-scalability
ms.openlocfilehash: 992abcb57204c65a7ca9e9e2525d3ea7339c4a2c
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2017
ms.locfileid: "24540231"
---
# <a name="materialized-view-pattern"></a>Modèle de vue matérialisée

[!INCLUDE [header](../_includes/header.md)]

Générez des vues préremplies sur les données d’un ou de plusieurs magasins de données lorsque les données ne sont pas adéquatement formatées pour les opérations de requête requises. Cela peut aider à la prise en charge efficace des requêtes et de l’extraction de données, et améliorer les performances de l’application.

## <a name="context-and-problem"></a>Contexte et problème

Lorsque vous stockez des données, la priorité pour les développeurs et les administrateurs de données porte souvent sur la méthode de stockage des données, et non sur leur lecture. En général, le format de stockage choisi est étroitement lié au format des données, aux exigences de gestion de la taille des données et de l’intégrité des données, ainsi qu’au type de magasin utilisé. Par exemple, lorsque vous utilisez le stockage de documents NoSQL, les données sont souvent représentées comme une série d’agrégats, contenant chacun toutes les informations de cette entité.

Toutefois, cela peut avoir un effet négatif sur les requêtes. Lorsqu’une requête nécessite uniquement un sous-ensemble des données de certaines entités (par exemple, un résumé des commandes de plusieurs clients sans tous les détails des commandes), il faut extraire toutes les données des entités pertinentes afin d’obtenir les informations requises.

## <a name="solution"></a>Solution

Pour une prise en charge efficace des requêtes, une solution courante consiste à générer, à l’avance, une vue qui matérialise les données dans un format adapté à l’ensemble de résultats requis. Le modèle de vue matérialisée décrit la génération de vues préremplies de données dans les environnements où la source de données n’est pas dans un format approprié pour les requêtes, où il est difficile de générer une requête appropriée et où les performances des requêtes sont faibles en raison de la nature de la les données ou du magasin de données.

Ces vues matérialisées, qui contiennent uniquement les données requises par une requête, permettent aux applications d’obtenir rapidement les informations que dont elles ont besoin. En plus des jointures de tables ou de la combinaison des entités de données, les vues matérialisées peuvent inclure les valeurs actuelles des colonnes calculées ou des éléments de données, les résultats de la combinaison de valeurs ou de l’exécution de transformations sur les éléments de données, et les valeurs spécifiées dans le cadre de la requête. Une vue matérialisée peut même être optimisée pour une seule requête.

Important : une vue matérialisée (avec les données qu’elle contient) peut être complètement supprimée, car elle peut être reconstruite entièrement à partir des magasins de données source. Une vue matérialisée n’est jamais mise à jour directement par une application, et il s’agit donc d’un cache spécialisé.

En cas de modification des données source de la vue, celle-ci doit être mise à jour pour inclure les nouvelles informations. Vous pouvez planifier l’exécution de cette action automatiquement ou lorsque le système détecte une modification des données d’origine. Dans certains cas, il peut être nécessaire de régénérer la vue manuellement. L’illustration montre un exemple d’utilisation du modèle de vue matérialisée.

![L’illustration 1 montre un exemple d’utilisation du modèle de vue matérialisée.](./_images/materialized-view-pattern-diagram.png)


## <a name="issues-and-considerations"></a>Problèmes et considérations

Prenez en compte les points suivants lorsque vous choisissez comment implémenter ce modèle :

Comment et quand la vue sera mise à jour. Dans l’idéal, il se régénère en réponse à un événement une modification des données source, bien que cela puisse entraîner une charge excessive si les données source changent rapidement. Vous pouvez également envisager d’utiliser une tâche planifiée, un déclencheur externe ou une action manuelle pour régénérer la vue.

Dans certains systèmes (par exemple, lorsque vous utilisez le modèle d’approvisionnement en événements pour tenir à jour un magasin contenant uniquement les événements qui ont modifié les données), les vues matérialisées sont nécessaires. Le préremplissage des vues en examinant tous les événements pour déterminer l’état actuel peut être la seule façon d’obtenir des informations à partir du magasin d’événements. Si vous n’utilisez pas l’approvisionnement en événements, vous devez déterminer si une vue matérialisée est utile ou non. Les vues matérialisées sont généralement adaptées à une requête ou à un petit nombre de requêtes. Si de nombreuses requêtes sont utilisées, les vues matérialisées peuvent entraîner des besoins en capacité de stockage et des coûts de stockage inacceptables.

Tenez compte de l’impact sur la cohérence des données lors de la génération de la vue, ainsi que du moment de mise à jour de la vue si cela se produit selon une planification. Si la source de données change au point où la vue est générée, la copie des données dans la vue ne sera pas entièrement cohérente avec les données d’origine.

Déterminez où vous allez stocker la vue. La vue ne doit pas forcément se trouver dans le même magasin ou la même partition que les données d’origine. Il peut s’agir d’un sous-ensemble de quelques partitions différentes combinées.

Une vue perdue peut être reconstruite. Par conséquent, si la vue est temporaire et utilisée uniquement pour améliorer les performances des requêtes en reflétant l’état actuel des données, ou améliorer l’évolutivité, elle peut être stockée dans un cache ou dans un emplacement moins fiable.

Lorsque vous définissez une vue matérialisée, optimisez sa valeur en ajoutant des éléments de données ou des colonnes qui reposent sur un calcul ou une transformation des éléments de données existants, sur les valeurs transmises dans la requête ou sur des combinaisons de ces valeurs, le cas échéant.

Si le mécanisme de stockage le prend en charge, pensez à indexer la vue matérialisée pour accroître davantage les performances. La plupart des bases de données relationnelles prennent en charge l’indexation pour les vues, tout comme les solutions Big Data basées sur Apache Hadoop.

## <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle

Ce modèle est utile dans les situations suivantes :
- Création de vues matérialisées sur des données qui sont difficiles à interroger directement, ou lorsque les requêtes doivent être très complexes pour extraire des données stockées d’une manière normalisée, non structurée ou semi-structurée.
- Création de vues temporaires qui peuvent améliorer considérablement les performances des requêtes ou servir directement de vues source ou d’objets de transfert de données pour l’interface utilisateur, pour la création de rapports ou pour l’affichage.
- Prise en charge occasionnelle de scénarios connectés ou déconnectés où la connexion au magasin de données n’est pas toujours disponible. La vue peut être mise en cache localement dans ce cas.
- Simplification des requêtes et exposition de données pour une expérimentation d’une manière qui ne nécessite pas la connaissance du format des données source. Par exemple, jointure de tables différentes dans une ou plusieurs bases de données, ou un ou plusieurs domaines dans des banques NoSQL, puis formatage des données en fonction de leur éventuelle utilisation.
- Fourniture d’accès à des sous-ensembles spécifiques des données source qui, pour des raisons de confidentialité ou de sécurité, ne doivent pas être accessibles par tous, modifiables ou entièrement exposées aux utilisateurs.
- Liaison de différents magasins de données pour tirer parti de leurs fonctionnalités individuelles. Par exemple, utilisation d’un magasin cloud efficace pour l’écriture en tant que magasin de données de référence, et d’une base de données relationnelle qui offre des performances acceptables de lecture et d’interrogation pour contenir les vues matérialisées.

Ce modèle est inutile dans les situations suivantes :
- La source de données est simple et facile à interroger.
- La source de données change très rapidement ou est accessible sans utilisation d’une vue. Dans ces cas de figure, vous devez éviter la charge de traitement inhérente à la création de vues.
- La cohérence est une priorité élevée. Parfois, les vues ne sont pas entièrement cohérentes avec les données d’origine.

## <a name="example"></a>Exemple

L’illustration suivante montre un exemple d’utilisation du modèle de vue matérialisée pour générer une synthèse des ventes. Les données des tables Order, OrderItem et Customer situées dans des partitions distinctes d’un compte de stockage Azure sont combinées pour générer une vue contenant la valeur totale des ventes pour chaque produit de la catégorie Electronics, ainsi que le nombre de clients ayant acheté chaque article.

![Figure 2 : Utilisation du modèle de vue matérialisée pour générer une synthèse des ventes](./_images/materialized-view-summary-diagram.png)


La création de cette vue matérialisée nécessite des requêtes complexes. Toutefois, en exposant les résultats de la requête comme une vue matérialisée, les utilisateurs peuvent facilement obtenir les résultats et les utiliser directement ou les incorporer dans une autre requête. La vue est susceptible d’être utilisée dans un système de création de rapports ou un tableau de bord. Elle peut être mise à jour de manière planifiée, par exemple chaque semaine.

>  Bien que cet exemple utilise le stockage de table Azure, de nombreux systèmes de gestion de base de données relationnelle fournissent également une prise en charge native pour les vues matérialisées.

## <a name="related-patterns-and-guidance"></a>Conseils et modèles connexes

Les modèles et les conseils suivants peuvent aussi présenter un intérêt quand il s’agit d’implémenter ce modèle :
- [Primer de cohérence des données](https://msdn.microsoft.com/library/dn589800.aspx). Les informations récapitulatives d’une vue matérialisée doivent être tenues à jour afin de refléter les valeurs de données sous-jacentes. Lorsque les valeurs de données changent, il n’est pas pratique de mettre à jour les données de synthèse en temps réel. À la place, vous devez adopter une approche cohérente. Résume les problèmes se rapportant au maintient de la cohérence des données distribuées, et décrit les avantages et les compromis des différents modèles de cohérence.
- [Modèle de séparation des responsabilités en matière de commande et de requête (CQRS)](cqrs.md). Utilisez ce modèle pour mettre à jour les informations contenues dans une vue matérialisée en réponse aux événements qui se produisent lorsque les valeurs de données sous-jacentes changent.
- [Modèle d'approvisionnement en événements](event-sourcing.md). Utilisez-le conjointement au modèle CQRS pour gérer les informations dans une vue matérialisée. Lorsque les valeurs de données sur lesquelles repose une vue matérialisée changent, le système peut déclencher des événements qui décrivent ces modifications et les enregistrent dans un magasin d’événements.
- [Modèle de table d’index](index-table.md). Les données d’une vue matérialisée sont généralement organisées par clé primaire, mais les requêtes doivent parfois récupérer les informations de cette vue en examinant les données d’autres champs. Utilisez ce modèle pour créer des index secondaires sur des jeux de données des magasins de données qui ne prennent pas en charge les index secondaires natifs.
