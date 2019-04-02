---
title: Ingestion et analyse en masse de flux d’actualités sur Azure
description: Créez un pipeline pour ingérer et analyser du texte, des images, des sentiments et autres données de flux d’actualités RSS en utilisant seulement des services Azure, notamment Azure Cosmos DB et Azure Cognitive Services.
author: njray
ms.date: 2/1/2019
ms.custom: azcat-ai, AI
social_image_url: /azure/architecture/example-scenario/ai/media/mass-ingestion-newsfeeds-architecture.png
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.openlocfilehash: e1bc636103753d474b545a6e3e9118b2ef302b34
ms.sourcegitcommit: ea97ac004c38c6b456794c1a8eef29f8d2b77d50
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/26/2019
ms.locfileid: "58489256"
---
# <a name="mass-ingestion-and-analysis-of-news-feeds-on-azure"></a>Ingestion et analyse en masse de flux d’actualités sur Azure

Cet exemple de scénario décrit un pipeline pour l’ingestion massive et près de l’analyse en temps réel de documents à l’aide de canaux RSS publiques.  Elle utilise Azure Cognitive Services pour fournissent des informations utiles, y compris la traduction de texte, la reconnaissance des visages et la détection de sentiments.

Ce scénario contient des exemples pour [anglais][english], [russe][russian], et [allemand] [ german] news flux, mais vous pouvez l’étendre facilement aux autres flux RSS. Pour faciliter le déploiement, la collecte de données, le traitement et l’analyse reposent entièrement sur les services Azure.

## <a name="relevant-use-cases"></a>Cas d’usage appropriés

Tandis que ce scénario est basé sur le traitement de flux RSS, elle s’applique à n’importe quel document, un site Web ou un article dans lequel vous devez :

* Traduire un texte pour le langage de choix.
* Rechercher des expressions clés, les entités et les sentiments de l’utilisateur dans le contenu numérique.
* Détecter des objets, le texte et points de repère dans des images associés à un article numérique.
* Détecter des personnes par leur type et un âge dans n’importe quelle image associée à contenu numérique.

## <a name="architecture"></a>Architecture

![][architecture]

Les données circulent dans la solution comme suit :

1. Un flux d’actualités RSS agit comme le générateur qui obtient des données à partir d’un document ou un article. Par exemple, avec un article, les données comprend généralement un titre, un résumé de l’organisme d’origine de l’article et parfois d’images.

2. Un processus de générateur ou ingestion insère l’article et les images associées dans Azure Cosmos DB [Collection][collection].

3. Une notification déclenche une fonction de réception dans Azure Functions qui enregistre le texte de l’article dans CosmosDB et les images de l’article (le cas échéant) dans le stockage Blob Azure.  L’article est ensuite transmis à la file d’attente suivante.

4. Une fonction de traduction est déclenchée par l’événement de la file d’attente. Il utilise le [traduire les API de texte] [ translate-text] d’Azure Cognitive Services pour détecter la langue, traduire, si nécessaire et y recueillir les sentiments, les expressions clés et les entités le corps et le titre. Il transmet ensuite l’article à la file d’attente suivante.

5. Une fonction de détection est déclenchée à partir de l’article en file d’attente. Il utilise le [vision par ordinateur] [ vision] service pour détecter des objets, les points de repère et les mots écrits dans l’image associée, puis transmet l’article à la file d’attente suivante.

6. Une fonction face est déclenchée est déclenchée à partir de l’article en file d’attente. Il utilise le [API visage de Azure] [ face] service pour détecter les faces pour sexe et l’âge dans l’image associée, puis transmet l’article à la file d’attente suivante.

7. Lorsque toutes les fonctions sont terminées, la fonction de notification est déclenchée. Il charge les enregistrements traités pour l’article et les analyse pour les résultats souhaités. Si trouvée, le contenu est marqué et une notification est envoyée au système de votre choix.

À chaque étape du processus, la fonction écrit les résultats dans Azure Cosmos DB. Au final, les données peuvent être utilisées comme vous le souhaitez. Par exemple, vous pouvez l’utiliser pour améliorer les processus d’entreprise, recherchez les nouveaux clients ou identifier les problèmes de satisfaction client.

### <a name="components"></a>Composants

La liste suivante des composants Azure est utilisée dans cet exemple.

* [Stockage Azure] [ storage] est utilisé pour contenir des images bruts et des fichiers vidéo associés à un article. Un compte de stockage secondaire est créé avec Azure App Service et est utilisé pour héberger le code de fonction Azure et les journaux.

* [Azure Cosmos DB] [ cosmos-db] contient l’article texte, image et vidéo des informations de suivi. Les résultats des étapes de Cognitive Services sont également stockés ici.

* [Azure Functions] [ functions] exécute le code de fonction utilisé pour répondre aux messages de file d’attente et de transformer le contenu entrant. [Azure App Service] [ aas] héberge le code de fonction et traite les enregistrements en série. Ce scénario comprend cinq fonctions : Ingérer, transformer, détecter l’objet, visage et en informer.

* [Azure Service Bus] [ service-bus] héberge les files d’attente Azure Service Bus utilisés par les fonctions.

* [Azure Cognitive Services] [ acs] fournit l’intelligence artificielle pour le pipeline selon les implémentations de la [vision par ordinateur] [ vision] service, [Face API][face], et [traduire le texte] [ translate-text] service de traduction automatique.

* [Azure Application Insights] [ aai] fournit analytique pour vous aider à diagnostiquer les problèmes et à comprendre les fonctionnalités de votre application.

### <a name="alternatives"></a>Autres solutions

* Au lieu d’utiliser un modèle basé sur la notification de la file d’attente et Azure Functions, utilisez un autre modèle pour ce flux de données. Par exemple, [rubriques Azure Service Bus] [ topics] peut être utilisé pour les processus et non les différentes parties de l’article en parallèle en tant que le traitement en série effectué dans cet exemple. Pour plus d’informations, comparer [files d’attente et rubriques][queues-topics].

* Utilisez [application logique Azure] [ logic-app] pour implémenter le code de fonction et d’implémenter le verrouillage des enregistrements comme [Redlock] [ redlock] (nécessaire en parallèle traitement jusqu'à ce qu’Azure Cosmos DB prend en charge [mises à jour partielles de documents][partial]). Pour plus d’informations, [comparer Functions et Logic Apps][compare].

* Implémentation de cette architecture à l’aide de composants d’intelligence artificielle personnalisés plutôt que les services Azure existants. Par exemple, étendre le pipeline à l’aide d’un modèle personnalisé qui détecte certaines personnes dans une image plutôt que le nombre de personnes générique, sexe et âge des données collectées dans cet exemple. Pour utiliser l’apprentissage automatique personnalisés ou des modèles d’intelligence artificielle avec cette architecture, créer les modèles en tant que points de terminaison RESTful peuvent donc être appelés à partir d’Azure Functions.

* Utilisez un autre mécanisme d’entrée au lieu de flux RSS. Permet de plusieurs générateurs ou processus d’ingestion de flux Azure Cosmos DB et le stockage Azure.

## <a name="considerations"></a>Considérations

Par souci de simplicité, cet exemple de scénario utilise uniquement quelques exemples des API disponibles et des services d’Azure Cognitive Services. Par exemple, le texte dans les images peut être analysé à l’aide de la [API d’Analytique de texte][text-analytics]. La langue cible dans ce scénario est supposée pour être l’anglais, mais vous pouvez modifier l’entrée à tout [langue prise en charge] [ language] de votre choix.

### <a name="scalability"></a>Extensibilité

Les fonctions Azure mise à l’échelle varie selon le [plan d’hébergement] [ plan] vous utilisez. Cette solution suppose un [plan de consommation][plan-c], dans le calcul power est alloué automatiquement aux fonctions si nécessaire. Vous payez uniquement lorsque vos fonctions sont en cours d’exécution. Une autre option consiste à utiliser un [Azure App Service] [ plan-aas] plan, ce qui vous permet de mettre à l’échelle les niveaux pour allouer une quantité différente de ressources.

Avec Azure Cosmos DB, la clé est de distribuer votre charge de travail à peu près uniformément entre un nombre suffisant de [clés de partition][keys]. Il n’existe aucune limite à la quantité totale de données qu’un conteneur peut stocker ou à la quantité totale de [débit] [ throughput] un conteneur pouvant prendre en charge.

### <a name="management-and-logging"></a>Gestion et la journalisation

Cette solution utilise [Application Insights] [ aai] pour collecter des informations de journalisation et de performances. Une instance d’Application Insights est créée avec le déploiement dans le même groupe de ressources que les autres services nécessaires pour ce déploiement.

Pour afficher les journaux générés par la solution :

1. Accédez à [Azure portal] [ portal] et accédez au groupe de ressources créé pour le déploiement.

2. Cliquez sur le **Application Insights** instance.

3. À partir de la **Application Insights** section, accédez à **examiner\\recherche** et rechercher les données.

### <a name="security"></a>Sécurité

Azure Cosmos DB utilise une connexion sécurisée et la signature d’accès partagé via le C\# SDK fourni par Microsoft. Il n’existe aucun autres surfaces d’exposition exposés en externe. En savoir plus sur la sécurité [meilleures pratiques] [ db-practices] pour Azure Cosmos DB.

## <a name="pricing"></a>Tarifs

Le coût quotidien estimé pour conserver ce déploiement disponible est d’environ \$20 US sans données circulant dans le système.

Azure Cosmos DB est puissante, mais entraîne le plus grand [coût] [ db-cost] dans ce déploiement. Vous pouvez utiliser une autre solution de stockage par la refactorisation du code Azure Functions fourni.

La tarification pour Azure Functions peut varier selon le [plan] [ function-plan] elle s’exécute dans.

## <a name="deploy-the-scenario"></a>Déployez le scénario

> [!NOTE]
> Vous devez disposer d’un compte Azure existant. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit][free] avant de commencer.

Tout le code pour ce scénario est disponible dans le [GitHub] [ github] référentiel. Ce référentiel contient le code source utilisé pour générer l’application de générateur qui alimente le pipeline pour cette démonstration.

[architecture]: ./media/mass-ingestion-newsfeeds-architecture.png
[aai]: /azure/azure-monitor/app/app-insights-overview
[aas]: https://azure.microsoft.com/try/app-service/
[acs]: https://azure.microsoft.com/services/cognitive-services/directory/
[collection]: /rest/api/cosmos-db/collections
[compare]: /azure/azure-functions/functions-compare-logic-apps-ms-flow-webjobs#compare-azure-functions-and-azure-logic-apps
[cosmos-db]: /azure/cosmos-db/introduction
[db-cost]: https://azure.microsoft.com/pricing/details/cosmos-db/
[db-practices]: /azure/cosmos-db/database-security
[db-collection]: /azure/cosmos-db/databases-containers-items
[english]: https://www.nasa.gov/rss/dyn/breaking_news.rss
[face]: /azure/cognitive-services/face/overview
[free]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[functions]: /azure/azure-functions/functions-overview
[function-plan]: /azure/azure-functions/functions-scale
[german]: http://www.bamf.de/SiteGlobals/Functions/RSS/DE/Feed/RSSNewsfeed_Meldungen
[github]: https://github.com/Azure/cognitive-services
[keys]: /azure/cosmos-db/partition-data
[language]: /azure/cognitive-services/translator/reference/v3-0-languages
[logic-app]: /azure/logic-apps/logic-apps-overview
[queues-topics]: /azure/service-bus-messaging/service-bus-queues-topics-subscriptions
[partial]: https://feedback.azure.com/forums/263030-azure-cosmos-db/suggestions/6693091-be-able-to-do-partial-updates-on-document
[plan]: /azure/azure-functions/functions-scale
[plan-aas]: /azure/azure-functions/functions-scale#app-service-plan
[plan-c]: /azure/azure-functions/functions-scale#consumption-plan
[portal]: http://portal.azure.com
[redlock]: https://redis.io/topics/distlock
[russian]: http://government.ru/all/rss/
[service-bus]: /azure/service-bus-messaging/
[storage]: /azure/storage/common/storage-account-overview 
[throughput]: /azure/cosmos-db/scaling-throughput
[topics]: /azure/service-bus-messaging/service-bus-dotnet-how-to-use-topics-subscriptions
[text-analytics]: /azure/cognitive-services/text-analytics/
[translate-text]: /azure/cognitive-services/translator/translator-info-overview
[vision]: /azure/cognitive-services/computer-vision/home
