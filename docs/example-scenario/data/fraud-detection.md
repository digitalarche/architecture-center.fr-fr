---
title: 'Analyse avancée : Détection des fraudes en temps réel'
description: Solution éprouvée pour la détection d’activités frauduleuses en temps réel à l’aide d’Azure Event Hubs et de Stream Analytics.
author: alexbuckgit
ms.date: 07/05/2018
ms.openlocfilehash: cf375445b38b0ff7d6fbc400902d5e97b34b4fed
ms.sourcegitcommit: 5d99b195388b7cabba383c49a81390ac48f86e8a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/06/2018
ms.locfileid: "37891286"
---
# <a name="real-time-fraud-detection-on-azure"></a>Détection des fraudes en temps réel dans Azure

Cet exemple de scénario convient aux organisations qui doivent analyser les données en temps réel pour détecter les transactions frauduleuses ou d’autres activités anormales.

Applications potentielles : identification d’une activité de carte de crédit frauduleuse ou d’appels douteux via un téléphone mobile. Les systèmes d’analyse en ligne traditionnels peuvent nécessiter des heures pour transformer et analyser les données afin d’identifier toute activité anormale.

En utilisant des services Azure entièrement gérés tels qu’Event Hubs et Stream Analytics, les entreprises peuvent éliminer la nécessité de gérer des serveurs individuels, tout en réduisant les coûts et en tirant parti de l’expertise de Microsoft en matière d’ingestion des données à l’échelle du cloud et d’analyse en temps réel. Ce scénario concerne plus spécialement la détection d’activités frauduleuses. Si vous avez d’autres besoins relatifs à l’analyse des données, vous devez consulter la liste des [services Azure Analytics][product-category] disponibles.

Cet exemple représente une partie d’une stratégie et d’une architecture de traitement des données plus larges. D’autres options relevant de cet aspect d’une architecture globale sont décrites plus loin dans cet article.
 
## <a name="potential-use-cases"></a>Cas d’usage potentiels

Pensez à cette solution pour les cas d’usage suivants :

* Détection d’appels douteux via un téléphone mobile dans les scénarios de télécommunications.
* Identification de transactions frauduleuses par carte de crédit pour les établissements bancaires.
* Identification d’achats frauduleux dans les scénarios de vente au détail ou de e-commerce.

## <a name="architecture"></a>Architecture

![Présentation de l’architecture des composants Azure d’une solution de détection des fraudes en temps réel][architecture-diagram]

Cette solution couvre les composants principaux d’un pipeline d’analyse en temps réel. Les données circulent dans la solution comme suit :

1. Les métadonnées d’appel par téléphone mobile sont envoyées à partir du système source vers une instance Azure Event Hubs. 
2. Un travail Stream Analytics est démarré, et reçoit des données via la source Event Hub.
3. Le travail Stream Analytics exécute une requête prédéfinie pour transformer le flux d’entrée et l’analyser en fonction d’un algorithme de transactions frauduleuses. Cette requête utilise une fenêtre bascule pour segmenter le flux en unités temporelles distinctes.
4. Le travail Stream Analytics écrit le flux transformé représentant les appels frauduleux détectés dans un récepteur de sortie dans le stockage Blob Azure.

### <a name="components"></a>Composants

* [Azure Event Hubs][docs-event-hubs] est un service d’ingestion d’événements et de plateforme de diffusion en temps réel, capable de recevoir et de traiter des millions d’événements par seconde. Les concentrateurs d’événements peuvent traiter et stocker des événements, des données ou la télémétrie produits par des logiciels et appareils distribués. Dans cette solution, Event Hubs reçoit toutes les métadonnées d’appels téléphoniques à analyser pour détecter toute activité frauduleuse.
* [Azure Stream Analytics][docs-stream-analytics] est un moteur de traitement des événements qui peut analyser de grands volumes de données de streaming à partir d’appareils et d’autres sources de données. Il prend également en charge l’extraction d’informations à partir de flux de données pour identifier des modèles et des relations. Ces modèles peuvent déclencher d’autres actions en aval. Dans cette solution, Stream Analytics transforme le flux d’entrée d’Event Hubs pour identifier les appels frauduleux.
* [Stockage Blob][docs-blob-storage] est utilisé dans cette solution pour stocker les résultats du travail Stream Analytics.

## <a name="considerations"></a>Considérations

### <a name="alternatives"></a>Autres solutions

De nombreux choix technologiques sont disponibles pour l’ingestion des messages en temps réel, le stockage de données, le traitement de flux, le stockage des données d’analyse et l’analyse et la création de rapports. Pour une vue d’ensemble de ces options, de leurs fonctionnalités et des principaux critères de sélection, consultez l’article [Big data architectures: Real-time processing](/azure/architecture/data-guide/technology-choices/real-time-ingestion) (Architectures de Big Data : Traitement en temps réel) dans le Guide d’architecture des données Azure.

En outre, des algorithmes plus complexes pour la détection des fraudes peuvent être produits par différents services Machine Learning dans Azure. Pour une vue d’ensemble de ces options, consultez l’article [Technology choices for machine learning](/azure/architecture/data-guide/technology-choices/data-science-and-machine-learning) (Choix technologiques en matière de Machine Learning) dans le [Guide d’architecture des données Azure](../../data-guide/index.md).

### <a name="availability"></a>Disponibilité

Azure Monitor fournit des interfaces utilisateur unifiées pour la surveillance entre divers services Azure. Pour plus d’informations, consultez l’article [Monitoring in Microsoft Azure](/azure/monitoring-and-diagnostics/monitoring-overview) (Surveillance dans Microsoft Azure). Event Hubs et Stream Analytics sont intégrés à Azure Monitor. 

Pour voir d’autres considérations relatives à la disponibilité, consultez la [liste de contrôle de la disponibilité][availability] dans le Centre des architectures Azure.

### <a name="scalability"></a>Extensibilité

Les composants de cette solution sont conçus pour une ingestion à très grande échelle et une analyse en temps réel parallèle. Azure Event Hubs est un service très évolutif, capable de recevoir et de traiter des millions d’événements par seconde avec une faible latence.  Event Hubs peut effectuer [automatiquement une montée en puissance](/azure/event-hubs/event-hubs-auto-inflate) en augmentant le nombre d’unités de débit pour répondre aux besoins d’utilisation. Azure Stream Analytics est capable d’analyser des volumes élevés de données de diffusion en continu provenant de nombreuses sources. Vous pouvez monter en puissance Stream Analytics en augmentant le nombre [d’unités de streaming](/azure/stream-analytics/stream-analytics-streaming-unit-consumption) allouées pour exécuter votre travail de diffusion en continu.

Pour obtenir des conseils d’ordre général sur la conception de solutions évolutives, consultez la [liste de contrôle de l’extensibilité][scalability] dans le Centre des architectures Azure.

### <a name="security"></a>Sécurité

Azure Event Hubs sécurise les données via un [modèle de sécurité et d’authentification][docs-event-hubs-security-model] basé sur une combinaison de jetons de signature d’accès partagé (SAS) et d’éditeurs d’événements. Un éditeur d’événements définit un point de terminaison virtuel pour un hub d’événements. L’éditeur ne peut être utilisé que pour envoyer des messages à un hub d’événements. Il n'est pas possible de recevoir des messages à partir de l'éditeur.

Pour obtenir des conseils d’ordre général sur la conception de solutions sécurisées, consultez la [documentation sur la sécurité Azure][security].

### <a name="resiliency"></a>Résilience

Pour obtenir des conseils d’ordre général sur la conception de solutions résilientes, consultez l’article [Conception d’applications résilientes pour Azure][resiliency].

## <a name="deploy-the-solution"></a>Déployer la solution

Pour déployer cette solution, vous pouvez suivre ce [didacticiel pas à pas][tutorial] montrant comment déployer manuellement chaque composant de la solution. Ce didacticiel fournit également une application cliente .NET pour générer des exemples de métadonnées d’appels téléphoniques et envoyer ces données vers une instance Event Hub. 

## <a name="pricing"></a>Tarifs

Pour explorer le coût d’exécution de cette solution, tous les services sont préconfigurés dans le calculateur de coûts. Pour pouvoir observer l’évolution de la tarification pour votre cas d’usage particulier, modifiez les variables appropriées en fonction du volume de données que vous escomptez.

Nous proposons trois exemples de profils de coût basés sur la quantité de trafic que vous escomptez :

* [Petit][small-pricing] : traite un million d’événements via une unité de streaming standard par mois.
* [Moyen][medium-pricing] : traite 100 millions d’événements via cinq unités de streaming standard par mois.
* [Grand][large-pricing] : traite 999 millions d’événements via 20 unités de streaming standard par mois.

## <a name="related-resources"></a>Ressources associées

Les scénarios de détection des fraudes plus complexes peuvent bénéficier d’un modèle Machine Learning. Pour les solutions créées à l’aide de Machine Learning Server, consultez l’article [Fraud detection using Machine Learning Server][r-server-fraud-detection] (Détection des fraudes à l’aide de Machine Learning Server). Pour les autres modèles de solution utilisant Machine Learning Server, consultez l’article [Scénarios de science des données et modèles de solution][docs-r-server-sample-solutions]. Pour un exemple de solution utilisant Azure Data Lake Analytics, consultez l’article [Using Azure Data Lake and R for Fraud Detection][technet-fraud-detection] (Utilisation d’Azure Data Lake et de R pour la détection des fraudes).  

<!-- links -->
[product-category]: https://azure.microsoft.com/product-categories/analytics/
[tutorial]: /azure/stream-analytics/stream-analytics-real-time-fraud-detection
[small-pricing]: https://azure.com/e/74149ec312c049ccba79bfb3cfa67606
[medium-pricing]: https://azure.com/e/4fc94f7376de484d8ae67a6958cae60a
[large-pricing]: https://azure.com/e/7da8804396f9428a984578700003ba42
[architecture-diagram]: ./images/architecture-diagram-fraud-detection.png
[docs-event-hubs]: /azure/event-hubs/event-hubs-what-is-event-hubs
[docs-event-hubs-security-model]: /azure/event-hubs/event-hubs-authentication-and-security-model-overview
[docs-stream-analytics]: /azure/stream-analytics/stream-analytics-introduction
[docs-blob-storage]: /azure/storage/blobs/storage-blobs-introduction
[docs-r-server-sample-solutions]: /machine-learning-server/r/sample-solutions
[r-server-fraud-detection]: https://microsoft.github.io/r-server-fraud-detection/
[technet-fraud-detection]: https://blogs.technet.microsoft.com/machinelearning/2017/06/28/using-azure-data-lake-and-r-for-fraud-detection/
[availability]: /azure/architecture/checklist/availability
[scalability]: /azure/architecture/checklist/scalability
[resiliency]: ../../resiliency/index.md
[security]: /azure/security/

