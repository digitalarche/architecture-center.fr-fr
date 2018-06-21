---
title: Données relationnelles
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: fa2fef27f47acadf00cbfc821c7432c07a3947be
ms.sourcegitcommit: 51f49026ec46af0860de55f6c082490e46792794
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/03/2018
ms.locfileid: "30298819"
---
# <a name="traditional-relational-database-solutions"></a>Solutions de bases de données relationnelles traditionnelles

Les données relationnelles sont modélisées à l’aide du modèle relationnel. Dans ce modèle, les données sont exprimées sous forme de tuples. Un *tuple* est un ensemble de paires attribut/valeur. Par exemple, un tuple peut être (itemid = 5, orderid = 1, item = « Siège », quantité = 200,00). Un jeu de tuples qui partagent tous les mêmes attributs est appelé *relation*. 

Les relations sont naturellement représentées sous forme de tableaux, dans lesquels chaque tuple est exposé comme une ligne dans le tableau. Toutefois, les lignes sont classées de manière explicite, contrairement aux tuples. Le schéma des bases de données définit les colonnes (en-têtes) de chaque tableau. Chaque colonne est définie avec un nom et un type de données pour toutes les valeurs stockées dans cette colonne sur toutes les lignes du tableau.

![Exemple illustrant les données à l’aide d’une base de données relationnelle](../images/example-relational.png)

Une banque de données qui organise les données à l’aide du modèle relationnel est appelée base de données relationnelle. Les clés primaires identifient uniquement des lignes dans un tableau. Des champs de clé étrangère sont utilisés dans un tableau pour faire référence à une ligne dans un autre tableau en faisant référence à la clé primaire de l’autre tableau. Des clés étrangères sont utilisées pour préserver l’intégrité référentielle, faisant en sorte que les lignes référencées ne soient ni modifiées ni supprimées tandis que la ligne de référence dépend d’elles. 

![Exemple illustrant les données à l’aide d’une base de données relationnelle](../images/example-relational2.png)

Les bases de données relationnelles prennent en charge différents types de contraintes qui permettent de garantir l’intégrité des données :

- Les contraintes uniques garantissent que toutes les valeurs d’une colonne sont uniques. 

- Les contraintes de clé étrangère sont associées aux données de deux tableaux. Une clé étrangère fait référence à la clé primaire ou à une autre clé unique à partir d’un autre tableau. Une contrainte de clé étrangère applique l’intégrité référentielle, interdisant les modifications qui génèrent des valeurs de clé étrangère non valides.

- Les contraintes CHECK, également connues sous le nom de contraintes d’intégrité d’entité, limitent les valeurs qui peuvent être stockées dans une colonne unique ou être en relation avec des valeurs d’autres colonnes de la même ligne. 

La plupart des bases de données relationnelles utilisent le langage SQL (Structured Query Langage) qui permet une approche déclarative à l’interrogation. La requête décrit le résultat souhaité, mais pas les étapes pour exécuter la requête. Le moteur décide alors de la meilleure façon d’exécuter la requête. Cela diffère d’une approche procédurale, où le programme de la requête spécifie explicitement les étapes de traitement. Toutefois, les bases de données relationnelles peuvent stocker des routines de codes exécutables sous la forme de procédures et fonctions stockées, ce qui permet une combinaison des approches déclaratives et procédurales.

Pour améliorer les performances des requêtes, les bases de données relationnelles utilisent des *index*. Les index principaux, utilisés par la clé primaire, définissent l’ordre des données sur le disque. Les index secondaires fournissent une autre combinaison de champs, afin que les lignes souhaitées puissent être interrogées efficacement, sans avoir à retrier toutes les données sur le disque.

Étant donné que les bases de données relationnelles appliquent l’intégrité référentielle, il peut devenir difficile de mettre à l’échelle une base de données relationnelle. Ceci puisque toutes les requêtes ou opérations d’insertion peuvent toucher n’importe quel tableau. Vous pouvez mettre à l’échelle une base de données relationnelle en *partitionnant* les données, mais cela requiert une conception minutieuse du schéma. Pour plus d’informations, consultez [Modèle de partitionnement](../../patterns/sharding.md).

Si les données sont non relationnelles ou ont des exigences qui ne sont pas adaptées à une base de données relationnelle, envisagez une banque de données [Non relationnelle ou NoSQL](../big-data/non-relational-data.md).
