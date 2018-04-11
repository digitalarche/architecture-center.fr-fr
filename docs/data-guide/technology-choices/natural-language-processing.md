---
title: Choisir une technologie de traitement du langage naturel
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: dacf7bf9cf3e9efed212f34da93c1470954965cf
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-natural-language-processing-technology-in-azure"></a>Choisir une technologie de traitement du langage naturel dans Azure

Le traitement de texte de forme libre est effectué sur des documents contenant des paragraphes de texte, généralement à des fins de prise en charge de la recherche, mais il est également utilisé pour effectuer d’autres tâches de traitement du langage naturel (NLP) comme l’analyse des sentiments, la détection de la rubrique, la détection de la langue, l’extraction d’expressions clés et le classement des documents. Cet article se concentre sur les choix technologiques prenant en charge les tâches NLP.

## <a name="what-are-your-options-when-choosing-an-nlp-service"></a>Quelles sont vos options lors du choix d’un service NLP ?

Dans Azure, les services suivants fournissent des fonctionnalités de traitement du langage naturel (NLP) :

- [Azure HDInsight avec Spark et Spark MLlib](/azure/hdinsight/spark/apache-spark-overview)
- [Microsoft Cognitive Services](/azure/#pivot=products&panel=cognitive)

## <a name="key-selection-criteria"></a>Critères de sélection principaux

Pour restreindre les choix, commencez par répondre à ces questions :

- Voulez-vous utiliser des modèles prédéfinis ? Si la réponse est Oui, envisagez d’utiliser les API proposées Microsoft Cognitive Services.

- Devez-vous effectuer l’apprentissage de modèles personnalisés avec une grande quantité de données de texte ? Si la réponse est Oui, envisagez d’utiliser Azure HDInsight avec Spark MLlib et Spark NLP.

- Avez-vous besoin de fonctionnalités NLP de bas niveau comme la création de jetons, la recherche de radical, la lemmatisation et TF/IDF (term frequency/inverse document frequency) ? Si la réponse est Oui, envisagez d’utiliser Azure HDInsight avec Spark MLlib et Spark NLP.

- Avez-vous besoin de fonctionnalités NLP simples, de haut niveau comme l’identification d’entité et d’intention, la détection de la rubrique, la vérification orthographique ou l’analyse des sentiments ? Si la réponse est Oui, envisagez d’utiliser les API proposées Microsoft Cognitive Services.

## <a name="capability-matrix"></a>Matrice des fonctionnalités

Les tableaux suivants résument les principales différences entre les fonctionnalités.  

### <a name="general-capabilities"></a>Fonctionnalités générales

| | Azure HDInsight | Microsoft Cognitive Services |
| --- | --- | --- |
| Fournit des modèles préformés en tant que service | Non  | OUI |
| de l’API REST | OUI | OUI |
| Programmabilité | Python, Scala, Java | C#, Java, Node.js, Python, PHP, Ruby |
| Prend en charge le traitement des jeux de données et des documents volumineux | OUI | Non  |

### <a name="low-level-natural-language-processing-capabilities"></a>Capacités de traitement du langage naturel de bas niveau

| | Azure HDInsight | Microsoft Cognitive Services |  
| --- | --- | --- | 
| Générateur de jetons | Oui (Spark NLP) | Oui (API Analyse linguistique) |
| Générateur de formes dérivées | Oui (Spark NLP) | Non  |
| Générateur de lemmatisation | Oui (Spark NLP) | Non  |
| Balisage morphosyntaxique | Oui (Spark NLP) | Oui (API Analyse linguistique) |
| TF/IDF (Term frequency/inverse-document frequency) | Oui (Spark MLlib) | Non  |
| Similarité de chaîne&mdash;modifier le calcul de la distance | Oui (Spark MLlib) | Non  |
| Calcul N-gramme | Oui (Spark MLlib) | Non  |
| Arrêter la suppression de mots | Oui (Spark MLlib) | Non  |

### <a name="high-level-natural-language-processing-capabilities"></a>Capacités de traitement du langage naturel de haut niveau

| | Azure HDInsight | Microsoft Cognitive Services |
| --- | --- | --- | 
| Identification de l’intention/l’entité et extraction | Non  | Oui (API LUIS (Language Understanding Intelligent Service)) |    
| Détection de la rubrique | Oui (Spark NLP) | Oui (API Analyse de texte) |
| Vérification orthographique | Oui (Spark NLP) | Oui (API Vérification orthographique Bing) |
| analyse de sentiments | Oui (Spark NLP) | Oui (API Analyse de texte) |
| Détection de la langue | Non  | Oui (API Analyse de texte) |
| Prend en charge plusieurs langues en plus de l’anglais | Non  | Oui (varie selon l’API) |

## <a name="see-also"></a>Voir aussi

[Traitement de langage naturel](../scenarios/natural-language-processing.md)