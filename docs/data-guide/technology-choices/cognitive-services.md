---
title: Sélectionner une technologie Cognitive Services
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: AI
ms.openlocfilehash: e8bbdf4874d5fc669e1c08add8cf212217d176a4
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58246210"
---
# <a name="choosing-a-microsoft-cognitive-services-technology"></a>Sélectionner une technologie Cognitive Services Microsoft

Microsoft Cognitive services constitue des API basées sur le cloud que vous pouvez utiliser dans les applications d’intelligence artificielle (AI) et les flux de données. Ils vous fournissent des modèles préformés prêts à être utilisés dans votre application, ne nécessitant aucune donnée et aucun apprentissage du modèle de votre part. Les services cognitifs sont développés par l’équipe de recherche et d’AI de Microsoft et exploitent les derniers algorithmes d’apprentissage approfondi. Ils sont utilisés sur des interfaces REST HTTP. En outre, les kits de développement logiciel sont disponibles pour plusieurs infrastructures de développement d’applications courantes.

Cognitive Services inclut :

- Analyse de texte
- Vision par ordinateur
- Analyse des vidéos
- Reconnaissance et génération vocale
- Compréhension du langage naturel
- Recherche intelligente

Principaux avantages :

- Effort de développement minimal pour les services d’AI de pointe.
- Intégration facile à des applications via les interfaces REST HTTP.
- Prise en charge intégrée des Cognitif Services de consommation dans Azure Data Lake Analytics.

Considérations :

- Disponible uniquement sur le Web. La connectivité Internet est généralement requise. Une exception est le Service Vision personnalisée, dont vous pouvez exporter le modèle formé pour la prédiction sur les appareils et à la périphérie IoT.

- Bien que la personnalisation considérable soit prise en charge, les services disponibles ne conviennent pas à toutes les exigences d’analytique prédictive.

<!-- markdownlint-disable MD026 -->

## <a name="what-are-your-options-when-choosing-amongst-the-cognitive-services"></a>De quelles options disposez-vous lors du choix de Cognitive Services ?

<!-- markdownlint-disable MD026 -->

Dans Azure, il existe des douzaines de Cognitive Services disponibles. La liste actuelle de ces services est disponible dans un répertoire classé selon zone fonctionnelle prise en charge :

- [Vision](https://azure.microsoft.com/services/cognitive-services/directory/vision/)
- [Speech](https://azure.microsoft.com/services/cognitive-services/directory/speech/)
- [Connaissance](https://azure.microsoft.com/services/cognitive-services/directory/know/)
- [action](https://azure.microsoft.com/services/cognitive-services/directory/search/)
- [Langage](https://azure.microsoft.com/services/cognitive-services/directory/lang/)

## <a name="key-selection-criteria"></a>Critères de sélection principaux

Pour restreindre les choix, commencez par répondre aux questions suivantes :

- Quel type de données traitez-vous ? Limiter vos options en fonction du type de données d’entrée que vous utilisez. Par exemple, si votre entrée est un texte, sélectionnez les services qui ont un type d’entrée de texte.

- Disposez-vous des données pour former un modèle ? Si oui, envisagez des services personnalisés qui vous permettent d’effectuer l’apprentissage de leurs modèles sous-jacents avec les données que vous fournissez pour des performances et une précision accrues.

## <a name="capability-matrix"></a>Matrice des fonctionnalités

Les tableaux suivants résument les principales différences entre les fonctionnalités.

### <a name="uses-prebuilt-models"></a>Utilise des modèles prédéfinis

|                                                   |             Type d’entrée              |                                                                                Avantage clé                                                                                |
|---------------------------------------------------|-------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                API Analyse de texte                 |                Texte                 |                                                       Évaluez les opinions et les rubriques pour comprendre ce que les clients recherchent.                                                        |
|                API de liaison d’entités                 |                Texte                 |                                               Renforcez les liaisons de données de votre application par une reconnaissance et une levée d’ambiguïté des entités nommées.                                               |
| LUIS (Language Understanding Intelligent Service) |                Texte                 |                                                          Apprenez à vos applications à comprendre les commandes de vos utilisateurs.                                                          |
|                 Service d’établissement QnA                 |                Texte                 |                                             Distillez des informations sous forme de FAQ dans des réponses de style conversationnel dans lesquelles il est facile de naviguer.                                              |
|              API Analyse linguistique              |                Texte                 |                                                            Simplifiez les concepts du langage complexe et d’analyse de texte.                                                             |
|           Service d’exploration de la base de connaissances           |                Texte                 |                                          Activez des expériences de recherche interactive sur des données structurées via des entrées en langage naturel.                                          |
|              API Modèle de langage web               |                Texte                 |                                                         Utilisez des modèles de langage prédictif formés sur les données à l’échelle du Web.                                                         |
|              API Connaissances universitaires               |                Texte                 |                                        Exploitez le contenu pédagogique riche de Microsoft Academic Graph alimenté par Bing.                                         |
|               API Suggestion automatique Bing                |                Texte                 |                                                        Intégrez des options intelligentes de suggestion automatique aux recherches.                                                        |
|               API Vérification orthographique Bing                |                Texte                 |                                                             Détectez et corrigez les fautes d’orthographe dans votre application.                                                             |
|                API de traduction de texte Translator Text                |                Texte                 |                                                                           Traduction automatique.                                                                            |
|                API Recommandations                |                Texte                 |                                                             Prédisez et recommandez les articles recherchés par vos clients.                                                              |
|              API Recherche d’entités Bing               |       Texte (requête de recherche sur le Web)       |                                                           Identifier et augmenter les informations de l’entité à partir du Web.                                                           |
|               API Recherche d’images Bing               |       Texte (requête de recherche sur le Web)       |                                                                            Recherchez des images.                                                                             |
|               API Recherche d’actualités Bing                |       Texte (requête de recherche sur le Web)       |                                                                             Recherchez des actualités.                                                                              |
|               API Recherche de vidéos Bing               |       Texte (requête de recherche sur le Web)       |                                                                            Rechercher des vidéos.                                                                             |
|                API Recherche Web Bing                |       Texte (requête de recherche sur le Web)       |                                                        Obtenez des informations de recherche améliorées à partir de milliards de documents Web.                                                        |
|                  API Reconnaissance vocale Bing                  |           Texte ou Speech            |                                                                  Convertissez de la voix en texte et inversement.                                                                   |
|              API Reconnaissance de l’orateur              |               Speech                |                                                       Utilisez Speech pour identifier et authentifier les orateurs individuels.                                                        |
|               API de traduction de conversation Translator Speech               |               Speech                |                                                                   Effectuez des traductions vocales en temps réel.                                                                   |
|                API Vision par ordinateur                |    Images (ou trames de vidéo)    | Distillez des informations utilisables à partir d’images, créez automatiquement la description de photos, dérivez des balises, reconnaissez des célébrités, extrayez du texte et créez des miniatures exactes. |
|                 Content Moderator                 |        Texte, images ou vidéo        |                                                               Modération automatisée des images, textes et vidéos.                                                                |
|                    API Émotion                    | Images (photos avec sujets humains) |                                                              Identifiez la plage d’émotions des sujets humains.                                                               |
|                     API Visage                      | Images (photos avec sujets humains) |                                                       Détectez, identifiez, analysez, organisez et balisez des visages sur des photos.                                                       |
|                   Video Indexer                   |                Vidéo                |                        Enregistrez en vidéo des informations telles que des sentiments, transcrivez des messages, reconnaissez des visages et des émotions et extrayez des mots clés.                         |

### <a name="trained-with-custom-data-you-provide"></a>Formé avec les données personnalisées fournies

| | Type d’entrée | Avantage clé |
| --- | --- | --- |
| Service Vision personnalisée | Images (ou trames de vidéo) | Personnalisez vos propres modèles de vision par ordinateur. |
| Custom Speech Service | Speech | Surmontez les obstacles de la reconnaissance vocale, tels que le style d’élocution, le fond sonore et le vocabulaire. |
| Service Décision personnalisée | Contenu Web (par exemple, les flux RSS) | Utiliser le Machine Learning pour sélectionner automatiquement le contenu approprié pour votre page d’accueil |
| API Recherche personnalisée Bing | Texte (requête de recherche sur le Web) | Outil de recherche de la qualité commerciale. |
