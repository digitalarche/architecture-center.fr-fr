---
title: Classification d’images pour les déclarations de sinistre
titleSuffix: Azure Example Scenarios
description: Générer le traitement d’images dans vos applications Azure.
author: david-stanford
ms.date: 07/05/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
social_image_url: /azure/architecture/example-scenario/ai/media/architecture-intelligent-apps-image-processing.png
ms.openlocfilehash: 03ef9d15ec9bf64dc743657e9d1e7a7e275c5a8e
ms.sourcegitcommit: 3b15d65e7c35a19506e562c444343f8467b6a073
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/25/2019
ms.locfileid: "54908322"
---
# <a name="image-classification-for-insurance-claims-on-azure"></a>Classification d’images pour les déclarations de sinistre sur Azure

Ce scénario s’applique aux entreprises qui doivent traiter des images.

Applications potentielles : classification d’images pour un site web de mode, analyse de texte et d’images pour les déclarations de sinistre ou compréhension des données de télémétrie issues des captures d’écran de jeux. Traditionnellement, les entreprises devaient développer une expertise en matière de modèles Machine Learning, effectuer l’apprentissage des modèles et enfin exécuter les images via leur processus personnalisé pour obtenir les données des images.

En utilisant des services Azure tels que l’API Vision par ordinateur et Azure Functions, les sociétés peuvent éliminer la nécessité de gérer des serveurs individuels, tout en réduisant les coûts et en tirant parti de l’expertise que Microsoft a déjà développée en matière de traitement d’images avec Cognitives Services. Cet exemple de scénario concerne plus particulièrement un cas d’usage de traitement d’images. Si vos besoins en termes d’intelligence artificielle sont variés, pensez à la suite complète de [Cognitive Services](/azure/#pivot=products&panel=ai).

## <a name="relevant-use-cases"></a>Cas d’usage appropriés

Les autres cas d’usage appropriés sont les suivants :

- Classification des images sur un site web de mode.
- Classification des données de télémétrie provenant de captures d’écran de jeux.

## <a name="architecture"></a>Architecture

![Architecture pour la classification d’images][architecture]

Ce scénario couvre les composants principaux d’une application web ou mobile. Les données circulent dans le scénario comme suit :

1. La couche d’API est générée à l’aide d’Azure Functions. Ces API permettent à l’application de charger des images et de récupérer des données à partir de Cosmos DB.
2. Quand une image est chargée via un appel d’API, elle est stockée dans le stockage Blob.
3. L’ajout de nouveaux fichiers dans le stockage Blob déclenche l’envoi d’une notification Event Grid à une instance Azure Functions.
4. Azure Functions envoie un lien vers le fichier qui vient d’être chargé à l’API Vision par ordinateur à des fins d’analyse.
5. Une fois que les données ont été retournées à partir de l’API Vision par ordinateur, Azure Functions crée une entrée dans Cosmos DB pour conserver les résultats de l’analyse avec les métadonnées d’images.

### <a name="components"></a>Composants

- L’[API Vision par ordinateur](/azure/cognitive-services/computer-vision/home) fait partie de la suite Cognitive Services et est utilisée pour récupérer des informations sur chaque image.
- [Azure Functions](/azure/azure-functions/functions-overview) fournit l’API principale pour l’application web et assure le traitement des événements correspondant aux images chargées.
- [Event Grid](/azure/event-grid/overview) déclenche un événement lorsqu’une nouvelle image est chargée dans le stockage Blob. L’image est ensuite traitée avec Azure Functions.
- [Stockage Blob](/azure/storage/blobs/storage-blobs-introduction) stocke tous les fichiers image qui sont chargés dans l’application web, ainsi que les fichiers statiques qui sont consommés par l’application web.
- [Cosmos DB](/azure/cosmos-db/introduction) stocke des métadonnées sur chaque image chargée, notamment les résultats du traitement de l’API Vision par ordinateur.

## <a name="alternatives"></a>Autres solutions

- [Service Vision personnalisée](/azure/cognitive-services/custom-vision-service/home). L’API Vision par ordinateur retourne un ensemble de [catégories basées sur la taxonomie][cv-categories]. Si vous devez traiter des informations qui ne sont pas retournées par l’API Vision par ordinateur, pensez au service Vision personnalisée, qui vous permet de créer des classifieurs d’images personnalisés.
- [Recherche Azure](/azure/search/search-what-is-azure-search). Si votre cas d’usage implique l’interrogation de métadonnées pour rechercher des images qui répondent à des critères spécifiques, pensez à utiliser Recherche Azure. Actuellement en préversion, la [recherche cognitive](/azure/search/cognitive-search-concept-intro) intègre parfaitement ce flux de travail.

## <a name="considerations"></a>Considérations

### <a name="scalability"></a>Extensibilité

La plupart des composants utilisés dans ce scénario sont des services gérés avec mise à l’échelle automatique. Deux exceptions notables : Azure Functions est limité à un maximum de 200 instances. Si vous avez besoin de plus d’instances, vous pouvez utiliser plusieurs régions ou plans d’application.

Cosmos DB n’effectue pas de mise à l’échelle automatique en termes d’unités de requête approvisionnées (RU). Pour obtenir des conseils sur l’estimation de vos besoins, consultez la section relative aux [unités de requête](/azure/cosmos-db/request-units) dans notre documentation. Pour tirer pleinement parti de la mise à l’échelle dans Cosmos DB, découvrez le fonctionnement des [clés de partition](/azure/cosmos-db/partition-data) dans Cosmos DB.

Les bases de données NoSQL sacrifient souvent la cohérence (au sens du théorème CAP) au profit de la disponibilité, de l’extensibilité et du partitionnement. Cet exemple de scénario utilisant un modèle de données clé-valeur, la cohérence des transactions est rarement nécessaire, car la plupart des opérations sont par définition atomiques. Vous trouverez de l’aide pour [Choisir un magasin de données adapté](../../guide/technology-choices/data-store-overview.md) dans le Centre des architectures Azure. Si votre implémentation nécessite une cohérence élevée, vous pouvez [choisir votre niveau de cohérence](/azure/cosmos-db/consistency-levels) dans Cosmos DB.

Pour obtenir des conseils d’ordre général sur la conception de solutions évolutives, consultez la [liste de contrôle de l’extensibilité][scalability] dans le Centre des architectures Azure.

### <a name="security"></a>Sécurité

Des [identités managées pour ressources Azure][msi] permettent à d’autres ressources internes d’accéder à votre compte et de les affecter à vos instances Azure Functions. Autorisez uniquement l’accès aux ressources requises dans ces identités pour vous assurer qu’aucun élément supplémentaire n’est exposé à vos fonctions (et potentiellement à vos clients).

Pour obtenir des conseils d’ordre général sur la conception de solutions sécurisées, consultez la [documentation sur la sécurité Azure][security].

### <a name="resiliency"></a>Résilience

Comme tous les composants de ce scénario sont gérés, à un niveau régional, ils sont résilients automatiquement.

Pour obtenir des conseils d’ordre général sur la conception de solutions résilientes, consultez l’article [Conception d’applications résilientes pour Azure][resiliency].

## <a name="pricing"></a>Tarifs

Pour explorer le coût d’exécution de ce scénario, tous les services sont préconfigurés dans le calculateur de coûts. Pour pouvoir observer l’évolution de la tarification pour votre cas d’usage particulier, modifiez les variables appropriées en fonction du trafic que vous escomptez.

Nous proposons trois exemples de profils de coût selon la quantité de trafic (nous supposons que toutes les images font 100 Ko) :

- [Petit][small-pricing] : &lt; 5 000 images traitées par mois.
- [Moyen][medium-pricing] : 500 000 images traitées par mois.
- [Grand][large-pricing] : 50 000 000 images traitées par mois.

## <a name="related-resources"></a>Ressources associées

Pour un parcours d’apprentissage interactif, consultez l’article [Créer une application web serverless dans Azure][serverless].

Avant de déployer cet exemple de scénario dans un environnement de production, passez en revue les pratiques recommandées pour [l’optimisation des performances et de la fiabilité d’Azure Functions][functions-best-practices].

<!-- links -->
[architecture]: ./media/architecture-intelligent-apps-image-processing.png
[small-pricing]: https://azure.com/e/f9b59d238b43423683db73f4a31dc380
[medium-pricing]: https://azure.com/e/7c7fc474db344b87aae93bc29ae27108
[large-pricing]: https://azure.com/e/cbadbca30f8640d6a061f8457a74ba7d
[cognitive-search]: /azure/search/cognitive-search-concept-intro
[serverless]: /azure/functions/tutorial-static-website-serverless-api-with-database
[cv-categories]: /azure/cognitive-services/computer-vision/home#the-86-category-concept
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[functions-best-practices]: /azure/azure-functions/functions-best-practices
[msi]: /azure/app-service/app-service-managed-service-identity
