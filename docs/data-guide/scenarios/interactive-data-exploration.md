---
title: Exploration interactive des données
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 20740a8fe912a63526c847416b832941f4ac33ec
ms.sourcegitcommit: 51f49026ec46af0860de55f6c082490e46792794
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/03/2018
ms.locfileid: "30297953"
---
# <a name="interactive-data-exploration"></a>Exploration interactive des données

Dans de nombreuses solutions d’informatique décisionnelle (BI) d’entreprise, les rapports et les modèles sémantiques sont créés par les spécialistes de la BI et gérés de façon centralisée. Cependant, les organisations veulent de plus en plus souvent donner les moyens aux utilisateurs de prendre des décisions guidées par les données. Par ailleurs, elles sont de plus en plus nombreuses à engager des *scientifiques des données* ou des *analystes de données*, dont le travail consiste à explorer les données de façon interactive et à appliquer des modèles statistiques et des techniques d’analyse pour trouver des tendances et des modèles dans les données. L’exploration interactive des données requiert des outils et des plateformes qui assurent un traitement à faible latence des visualisations de données et des requêtes ad hoc.

![](./images/data-exploration.png)

## <a name="self-service-bi"></a>Informatique décisionnelle en libre-service

L’informatique décisionnelle en libre-service est le nom donné à une approche moderne de la prise de décision selon laquelle les utilisateurs ont l’autonomie nécessaire pour rechercher, explorer et partager des informations à partir de l’ensemble des données de l’entreprise. Pour cela, la solution de données doit remplir plusieurs conditions :

* Découverte des sources de données d’entreprise au moyen d’un catalogue de données.
* Gestion des données de référence garantissant la cohérence des valeurs et des définitions des entités de données.
* Outils interactifs de modélisation et de visualisation des données pour les utilisateurs professionnels.

Dans une solution d’informatique décisionnelle en libre-service, les utilisateurs professionnels recherchent et exploitent en général les sources de données qui correspondent à leur propre domaine dans l’entreprise, et utilisent des outils intuitifs et des applications de productivité pour définir des rapports et des modèles de données personnels, qu’ils peuvent partager avec leurs collègues.

Services Azure appropriés :

- [Azure Data Catalog](/azure/data-catalog/data-catalog-what-is-data-catalog)
- [Microsoft Power BI](https://powerbi.microsoft.com/)

## <a name="data-science-experimentation"></a>Expérimentation en science des données
Lorsqu’une organisation a besoin de techniques avancées d’analytique et de modélisation prédictive, le travail de préparation initial est généralement effectué par des spécialistes de la science des données. Un scientifique des données explore les données et applique des techniques d’analyse statistique pour trouver des relations entre les *traits* des données et les *étiquettes* prédites souhaitées. L’exploration de données est généralement effectuée à l’aide de langages de programmation comme Python et R, qui prennent en charge nativement la modélisation statistique et la visualisation. Les scripts utilisés sont généralement hébergés dans des environnements spécialisés, par exemple, des Notebooks Jupyter. Ces outils permettent aux scientifiques des données d’explorer les données par programme tout en documentant et en partageant les informations trouvées.

Services Azure appropriés :

- [Azure Notebooks](https://notebooks.azure.com/)
- [Azure Machine Learning Studio](/azure/machine-learning/studio/what-is-ml-studio)
- [Services Azure Machine Learning – Expérimentation](/azure/machine-learning/preview/experimentation-service-configuration)
- [La machine virtuelle de la science des données](/azure/machine-learning/data-science-virtual-machine/overview)

## <a name="challenges"></a>Défis

- **Respect de la confidentialité des données.** Faites attention quand vous mettez des données personnelles à la disposition des utilisateurs à des fins d’analyse et de création de rapports en libre-service. Cela soulève probablement des considérations relatives à la conformité, en raison des directives organisationnelles, ainsi que des problèmes de réglementation. 

- **Volume de données**. S’il peut être utile de donner accès à la totalité de la source de données aux utilisateurs, cela peut occasionner de très longues opérations Excel ou Power BI, ou des requêtes Spark SQL qui utilisent de nombreuses ressources de cluster.

- **Connaissances des utilisateurs**. Les utilisateurs créent leurs propres requêtes et agrégations afin de prendre des décisions d’entreprise éclairées. Avez-vous l’assurance qu’ils ont les compétences nécessaires en analyse et en interrogation pour obtenir des résultats corrects ?

- **Partage des résultats**. Le fait que les utilisateurs puissent créer et partager des rapports ou des visualisations de données risque de poser des questions de sécurité.

## <a name="architecture"></a>Architecture

Bien que l’objectif de ce scénario soit de prendre en charge l’analyse interactive des données, les tâches de nettoyage, d’échantillonnage et de structuration impliquées dans la science des données représentent souvent des processus longs. D’où la pertinence d’une architecture de [traitement par lots](../big-data/batch-processing.md).

## <a name="technology-choices"></a>Choix de technologie

Les technologies suivantes sont recommandées pour l’exploration interactive des données dans Azure.

### <a name="data-storage"></a>Stockage des données

- **Conteneurs Azure Storage Blob** ou **Azure Data Lake Store**. Les scientifiques des données travaillent généralement avec des données sources brutes, de façon à avoir accès à tous les traits, à toutes les valeurs hors norme et à toutes les erreurs dans les données possibles. Dans un scénario de Big Data, ces données prennent généralement la forme de fichiers dans un magasin de données.

Pour plus d’informations, consultez la page [Stockage de données](../technology-choices/data-storage.md).

### <a name="batch-processing"></a>Traitement par lots

- **R Server** ou **Spark**. La plupart des scientifiques des données utilisent des langages de programmation prenant en charge de nombreux packages mathématiques et statistiques, par exemple, R ou Python. En cas de gros volumes de données, il est possible de réduire la latence en utilisant des plateformes qui permettent à ces langages d’utiliser des traitements distribués. R Server peut être utilisé seul ou conjointement avec Spark pour monter en charge les fonctions de traitement de R ; Spark prend nativement en charge Python pour des fonctionnalités de montée en charge similaires dans ce langage.
- **Hive**. Hive est un bon choix pour transformer les données suivant une sémantique de type SQL. Les utilisateurs peuvent créer et charger des tables à l’aide d’instructions HiveQL, similaires à SQL d’un point de vue sémantique.

Pour plus d’informations, consultez la section [Traitement par lots](../technology-choices/batch-processing.md).

### <a name="analytical-data-store"></a>Magasin de données analytiques

- **Spark SQL**. Spark SQL est une API qui repose sur Spark et prend en charge la création de trames de données et de tables interrogeables avec la syntaxe SQL. Que les fichiers de données à analyser soient des fichiers sources bruts ou de nouveaux fichiers nettoyés et préparés par un processus de traitement par lots, les utilisateurs peuvent définir des tables Spark SQL dessus pour aller plus loin dans l’interrogation de l’analyse. 
- **Hive**. En parallèle du traitement des données brutes par lots avec Hive, vous pouvez créer une base de données Hive contenant les tables et les vues Hive en fonction des dossiers dans lesquels sont stockées les données, ce qui permet d’effectuer des requêtes interactives à des fins d’analyse et de création de rapports. HDInsight contient un type de cluster Hive interactif qui utilise la mise en cache en mémoire afin de réduire le temps de réponse aux requêtes Hive. Les utilisateurs qui sont à l’aise avec la syntaxe de type SQL peuvent utiliser Interactive Hive pour explorer les données.

Pour plus d’informations, consultez la section [Magasins de données analytiques](../technology-choices/analytical-data-stores.md).

### <a name="analytics-and-reporting"></a>Analytique et création de rapports

- **Jupyter**. Les Notebooks Jupyter offrent une interface sur navigateur pour exécuter du code dans des langages comme R, Python ou Scala. Lorsque R Server ou Spark est utilisé pour traiter les données par lots, ou Spark SQL pour définir un schéma des tables à des fins d’interrogation, Jupyter peut représenter un bon choix pour interroger les données. Si vous utilisez Spark, vous pouvez vous servir de l’API de trame de données Spark standard ou de l’API Spark SQL ainsi que des instructions SQL incorporées pour interroger les données et générer des visualisations.
- **Clients Interactive Hive**. Si vous utilisez un cluster Interactive Hive pour interroger les données, vous pouvez utiliser la vue Hive dans le tableau de bord du cluster Ambari, l’outil de ligne de commande Beeline ou n’importe quel outil ODBC (avec le pilote ODBC de Hive), par exemple, Microsoft Excel ou Power BI.

Pour plus d'informations, consultez la section [Technologies d’analytique des données et de création de rapports](../technology-choices/analysis-visualizations-reporting.md).