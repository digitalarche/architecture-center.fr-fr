---
title: "Analytique avancée"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 31ba357fe37b1de35a6eea324d2d1d6766e172e5
ms.sourcegitcommit: 29fbcb1eec44802d2c01b6d3bcf7d7bd0bae65fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/27/2018
---
# <a name="advanced-analytics"></a>Analytique avancée

L’analytique avancée va au-delà des rapports historiques et de l’agrégation des données des capacités d’analyse décisionnelles (BI) traditionnelles et utilise des techniques de modélisation mathématiques, probabilistes et statistiques pour permettre le traitement prédictif et la prise de décision automatisée.

Les solutions d’analytique avancée impliquent en général les charges de travail suivantes :

* Exploration et visualisation des données interactives
* Formation au modèle d’apprentissage automatique
* Traitement prédictif par lots ou en temps réel

La plupart des architectures d’analytique avancée incluent tout ou partie des composants suivants :

* **Stockage des données**. Les solutions d’analytique avancée nécessitent des données pour la formation des modèles d’apprentissage automatique. En général, les scientifiques de données doivent explorer les données pour identifier leurs fonctionnalités prédictives et les relations statistiques entre elles ainsi que les valeurs qu’ils prédisent (ou étiquette). L’étiquette prédite peut être une valeur quantitative, par exemple la valeur financière d’un projet ou le retard d’un vol en minutes. Ou elle peut représenter une classe par catégorie, comme « true » ou « false », « flight delay » ou « no flight delay », ou des catégories comme « low risk », « medium risk » ou « high risk ».

* **Traitement par lots**. Pour former un modèle d’apprentissage automatique, vous devez généralement traiter un gros volume de données de formation. La formation du modèle peut prendre un certain temps (de quelques minutes à plusieurs heures). Cette formation peut être effectuée à l’aide de scripts écrits dans des langages comme Python ou R, et peut être montée en charge afin de réduire le temps de formation à l’aide de plateformes de traitement distribué comme Apache Spark hébergées sur HDInsight ou un conteneur Docker.

* **Ingestion de messages en temps réel**. En production, beaucoup d’analytiques avancées transfèrent des flux de flux de données en temps réel vers un modèle prédictif qui a été publié comme un service web. Le flux de données entrant est généralement capturé sous la forme d’une file d’attente et un moteur de traitement de flux de données extrait les données de cette file d’attente puis applique la prédiction aux données d’entrée quasiment en temps réel.  

* **Traitement de flux**. Une fois que vous disposez d’un modèle formé, la prédiction (ou évaluation) est généralement une opération très rapide (de l’ordre des millisecondes) pour un ensemble donné de fonctionnalités. Après la capture des messages en temps réel, les valeurs des fonctionnalités en question peuvent être transférées au service afin de générer une étiquette prédite.

* **Magasin de données analytique**. Dans certains cas, les valeurs d’étiquette prédite sont consignées dans le magasin de données analytiques pour la création de rapports et une analyse future.

* **Analyse et rapports**. Comme son nom l’indique, les solutions d’analytique avancée produisent généralement une sorte de flux de rapport ou analytique qui inclut les valeurs prédites. Souvent, les valeurs d’étiquette prédite servent à remplir des tableaux de bord en temps réel.

* **Orchestration**. Bien que l’exploration de données initiale et la modélisation soient réalisées de façon interactive par des scientifiques de données, plusieurs solutions d’analytique avancée réapprennent régulièrement les modèles avec de nouvelles données &mdash; afin d’améliorer sans cesse la précision des modèles. Ce nouvel apprentissage peut être automatisé à l’aide d’un flux de travail orchestré.

## <a name="machine-learning"></a>Apprentissage automatique
L’apprentissage automatique est une technique de modélisation mathématique permettant de former un modèle prédictif. Le principe général consiste à appliquer un algorithme statistique à un large jeu de données historiques pour identifier les relations entre les champs qu’il contient.

La modélisation d’apprentissage automatique est généralement effectuée par des scientifiques de données qui ont besoin d’explorer et de préparer minutieusement les données avant l’apprentissage d’un modèle. Cette exploration et cette préparation impliquent généralement une phase intense d’analyse et de visualisation des données interactives &mdash;, traditionnellement à l’aide de langages comme Python et R dans des outils interactifs et des environnements spécifiquement conçus pour cette tâche.

Dans certains cas, vous pouvez utiliser des [modèles préformés](/machine-learning-server/install/microsoftml-install-pretrained-models) contenant des données de formation fournies et développées par Microsoft. Ces modèles préformés vous permettent d’évaluer et de classer immédiatement le nouveau contenu, même si vous ne disposez pas des données de formation nécessaires ni des ressources pour gérer des jeux de données volumineux ou former des modèles complexes.

Il existe deux grandes catégories d’apprentissage automatique :

* **Apprentissage supervisé**. L’apprentissage supervisé est l’approche la plus couramment adoptée par l’apprentissage automatique. Dans un modèle d’apprentissage supervisé, la source des données se compose d’un ensemble de champs de données de type *fonctionnalité* qui ont une relation mathématique avec un ou plusieurs champs de données de type *étiquette*. Pendant la phase de formation du processus d’apprentissage automatique, le jeu de données inclut à la fois des fonctionnalités et des étiquettes connues, et un algorithme est appliqué pour inclure une fonction appliquée aux fonctionnalités afin de calculer les prédictions d’étiquette correspondantes. En règle générale, un sous-ensemble du jeu de données d’apprentissage est conservé et utilisé pour valider les performances du modèle formé. Une fois le modèle formé, il peut être déployé en production et utilisé pour prédire des valeurs inconnues. 

* **Apprentissage non supervisé**. Dans un modèle d’apprentissage non supervisé, les données d’apprentissage n’incluent pas les valeurs d’étiquette connues. Au lieu de cela, l’algorithme effectue ses prévisions en fonction de son premier contact avec les données. La forme la plus courante d’apprentissage non supervisé est le *clustering*, où l’algorithme détermine la meilleure façon de fractionner les données en un nombre spécifié de clusters selon des similitudes statistiques dans les fonctionnalités. Dans le clustering, le résultat prédit est le numéro du cluster auquel appartiennent les fonctionnalités d’entrée. Même si elles peuvent parfois être utilisées directement pour générer des prédictions utiles, notamment l’utilisation du clustering pour identifier des groupes d’utilisateurs dans une base de données de clients, les approches d’apprentissage non supervisé servent le plus souvent à identifier les données les plus utiles pour générer un algorithme d’apprentissage supervisé dans l’apprentissage d’un modèle.

Services Azure appropriés :

- [Azure Machine Learning](/azure/machine-learning/)
- [Machine Learning Server (R Server) sur HDInsight](/azure/hdinsight/r-server/r-server-overview)

## <a name="deep-learning"></a>Apprentissage approfondi

Les modèles d’apprentissage automatique basés sur des techniques telles que la régression linéaire ou logistique existent depuis un certain temps. Depuis peu, l’utilisation de techniques d*’apprentissage approfondi* basées sur des réseaux neuronaux ne cesse d’augmenter. Cela s’explique en partie par la disponibilité de systèmes de traitement hautement évolutifs qui réduisent le temps nécessaire à l’apprentissage de modèles complexes. En outre, l’importance croissante du Big Data facilite l’apprentissage des modèles d’apprentissage approfondi dans différents domaines.

Lorsque vous concevez une architecture cloud d’analytique avancée, vous devez envisager la possibilité d’inclure un traitement de modèles d’apprentissage approfondi à grande échelle. Ces modèles peuvent être fournis via des plateformes de traitement distribué comme Apache Spark et la dernière génération de machines virtuelles permettant d’accéder au matériel GPU.

Services Azure appropriés :

- [Machine virtuelle Apprentissage profond](/azure/machine-learning/data-science-virtual-machine/deep-learning-dsvm-overview)
- [Apache Spark sur HDInsight](/azure/hdinsight/spark/apache-spark-overview)

## <a name="artificial-intelligence"></a>Intelligence artificielle

L’intelligence artificielle (IA) désigne les scénarios où un ordinateur imite les fonctions cognitives associées à un esprit humain, par exemple l’apprentissage et la résolution de problèmes. L’intelligence artificielle est un terme générique car elle s’appuie sur des algorithmes d’apprentissage automatique. La plupart des solutions IA reposent sur une combinaison de services prédictifs, souvent implémentés en tant que services web, et des interfaces de langage naturel, par exemple des chatbots qui interagissent par SMS ou reconnaissance vocale, présentés par des applications IA exécutées sur des appareils mobiles ou d’autres clients. Dans certains cas, le modèle d’apprentissage automatique est incorporé à l’application IA. 

## <a name="model-deployment"></a>Déploiement de modèle

Les services prédictifs qui prennent en charge les applications IA peuvent tirer parti de modèles d’apprentissage automatique personnalisés ou de services cognitifs prêts à l’emploi, permettant d’accéder à des modèles préformés. Le processus de déploiement de modèles personnalisés en production est appelé opérationnalisation, où les mêmes modèles IA formés et testés dans l’environnement de traitement sont sérialisés et mis à disposition d’applications externes et de services pour le traitement par lots ou des prédictions en libre-service. Pour utiliser la fonctionnalité prédictive du modèle, ce dernier est désérialisé et chargé à l’aide de la même bibliothèque d’apprentissage automatique contenant l’algorithme utilisé pour l’apprentissage du modèle en premier lieu. Cette bibliothèque fournit des fonctions prédictives (souvent appelées score ou prédiction) qui utilisent le modèle et les fonctionnalités comme entrées et retournent la prédiction. Cette logique est ensuite encapsulée dans une fonction qu’une application peut appeler directement, ou qui peut être exposée comme un service web. 

Services Azure appropriés :

- [Azure Machine Learning](/azure/machine-learning/)
- [Machine Learning Server (R Server) sur HDInsight](/azure/hdinsight/r-server/r-server-overview)


## <a name="see-also"></a>Voir aussi

- [Sélectionner une technologie Cognitive Services](../technology-choices/cognitive-services.md)
- [Sélectionner une technologie Machine Learning](../technology-choices/data-science-and-machine-learning.md)
