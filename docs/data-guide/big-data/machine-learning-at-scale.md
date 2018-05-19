---
title: Machine Learning à l’échelle
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 4e584da18893ac7405fa00863fe034e45b2e3903
ms.sourcegitcommit: 7ced70ebc11aa0df0dc0104092d3cc6ad5c28bd6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/11/2018
---
# <a name="machine-learning-at-scale"></a>Machine Learning à l’échelle

Machine Learning (ML) est une technique utilisée pour l’apprentissage des modèles prédictifs basé sur des algorithmes mathématiques. Machine Learning analyse les relations entre les champs de données pour prédire des valeurs inconnues.

La création et le déploiement d’un modèle Machine Learning constituent un processus itératif :

* Les scientifiques de données explorent les données sources pour déterminer les relations entre les *fonctionnalités* et les *étiquettes* prédites.
* Les scientifiques de données forment et valident des modèles à l’aide d’algorithmes appropriés afin de rechercher le modèle optimal pour la prédiction.
* Le modèle optimal est déployé en production, en tant que service web ou autre fonction encapsulée.
* Lorsque de nouvelles données sont collectées, le modèle est régulièrement reformé pour améliorer son efficacité.

Machine Learning à l’échelle traite deux problèmes d’évolutivité différents. Le premier est l’apprentissage d’un modèle sur des jeux de données volumineux qui nécessitent les fonctionnalités de montée en puissance (scale-out) d’un cluster à former. Le deuxième est l’opérationnisme du modèle formé d’une façon qui peut évoluer pour répondre aux demandes des applications qui le consomme. En général, cela est accompli en déployant les fonctionnalités prédictives en tant que service web qui peut ensuite être monté en puissance.

Machine Learning à l’échelle présente l’avantage qu’il peut produire des fonctionnalités prédictives, puissantes, car les modèles plus efficaces résultent généralement de davantage de données. Une fois qu’un modèle est formé, il peut être déployé en tant que service web sans état, très performant, capable de monter en puissance. 

## <a name="model-preparation-and-training"></a>Préparation et formation du modèle

Pendant la phase de préparation et de formation du modèle, les scientifiques de données explorent les données de manière interactive à l’aide de langages tels que Python et R pour :

* Extraire des exemples à partir d’entrepôts de données volumineux.
* Rechercher et traiter les valeurs hors norme, les doublons et les valeurs manquantes pour nettoyer les données.
* Déterminer les corrélations et les relations dans les données via l’analyse statistique et la visualisation.
* Générer de nouvelles fonctionnalités calculées qui améliorent la prévisibilité des relations statistiques.
* Former des modèles ML à l’aide d’algorithmes prédictifs.
* Valider les modèles formés à l’aide de données retenues pendant la formation.

Pour prendre en charge cette phase d’analyse interactive et de modélisation, la plateforme de données doit permettre aux scientifiques de données d’explorer les données à l’aide de divers outils. En outre, l’apprentissage d’un modèle Machine Learning complexe peut nécessiter le traitement intensif de grands volumes de données, donc il est essentiel de disposer de ressources suffisantes pour la montée en puissance de l’apprentissage du modèle.

## <a name="model-deployment-and-consumption"></a>Déploiement et consommation du modèle

Lorsqu’un modèle est prêt à être déployé, il peut être encapsulé en tant que service web et déployé sur le cloud, dans un appareil de périphérie ou au sein d’un environnement d’exécution ML d’entreprise. Ce processus de déploiement est appelé opérationnalisme.

## <a name="challenges"></a>Défis

Machine Learning à l’échelle présente quelques défis :

- Vous avez généralement besoin d’une grande quantité de données pour former un modèle, en particulier pour les modèles d’apprentissage profond.
- Vous devez préparer ces jeux de données volumineux avant même de pouvoir commencer à former votre modèle.
- La phase de formation du modèle doit accéder aux magasins de données volumineux. Il est courant d’effectuer l’apprentissage du modèle à l’aide du même cluster de données volumineux, comme Spark, que celui utilisé pour la préparation des données. 
- Pour les scénarios comme l’apprentissage profond, vous aurez non seulement besoin d’un cluster qui peut fournir la montée en puissance sur des unités centrales, mais votre cluster devra aussi se composer de nœuds GPU.

## <a name="machine-learning-at-scale-in-azure"></a>Machine Learning à l’échelle dans Azure

Avant de choisir les services ML à utiliser pour l’apprentissage et l’opérationnalisme, vous devez vous demander si vous avez besoin de former un modèle ou si un modèle prédéfini peut répondre à vos besoins. Dans de nombreux cas, l’utilisation d’un modèle prédéfini consiste simplement à appeler un service web ou à utiliser une bibliothèque ML pour charger un modèle existant. Certaines options incluent : 

- Utiliser les services web fournis par Microsoft Cognitive Services.
- Utiliser les modèles de réseau neuronal préformés fournis par Cognitive Toolkit.
- Incorporer les modèles sérialisés fournis par Core ML pour les applications iOS. 

Si un modèle prédéfini ne correspond pas à vos données ou à votre scénario, les options Azure incluent Azure Machine Learning, HDInsight avec Spark MLlib et MMLSpark, Cognitive Toolkit et SQL Machine Learning Services. Si vous décidez d’utiliser un modèle personnalisé, vous devez créer un pipeline qui inclut l’apprentissage et l’opérationnalisme du modèle. 

![Options de modèle dans Azure](./images/machine-learning-model-training-and-deployment.png)

Pour obtenir la liste des choix technologiques pour ML dans Azure, consultez les rubriques suivantes :

- [Sélectionner une technologie Cognitive Services](../technology-choices/cognitive-services.md)
- [Sélectionner une technologie Machine Learning](../technology-choices/data-science-and-machine-learning.md)
- [Choisir une technologie de traitement du langage naturel](../technology-choices/natural-language-processing.md)
