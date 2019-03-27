---
title: Créer pour les besoins de l’entreprise
titleSuffix: Azure Application Architecture Guide
description: Chaque décision de conception doit être justifiée par un besoin de l’entreprise.
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: 9529d1e11dc67144d2dcb641ea666de7c2205f20
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58245720"
---
# <a name="build-for-the-needs-of-the-business"></a>Créer pour les besoins de l’entreprise

## <a name="every-design-decision-must-be-justified-by-a-business-requirement"></a>Chaque décision de conception doit être justifiée par une exigence métier

Ce principe de conception peut vous sembler évident, mais vous devez impérativement le garder à l’esprit lorsque vous concevez une solution. Prévoyez-vous des millions d’utilisateurs pour votre solution ou seulement quelques milliers ? Une interruption de votre application pendant une heure est-elle acceptable ? Envisagez-vous d’importants pics de trafic ou une charge de travail très prévisible ? En fin de compte, chaque décision de conception doit être justifiée par une exigence métier.

## <a name="recommendations"></a>Recommandations

**Définissez les objectifs stratégiques**, notamment l’objectif de délai de récupération (RTO), l’objectif de point de récupération (RPO) et l’interruption acceptable maximale (MTO). Ces valeurs doivent refléter les décisions prises concernant l’architecture. Par exemple, pour obtenir une faible valeur RTO, vous pouvez implémenter un basculement automatisé vers une région secondaire. Mais si votre solution peut tolérer une valeur RTO plus élevée, ce degré de redondance peut se révéler inutile.

**Documentez les Contrats de niveau de service (SLA) et les objectifs de niveau de service (SLO)**, y compris les métriques de disponibilité et de performances. Vous pouvez créer une solution offrant une disponibilité de 99,95 %. Est-ce suffisant ? La réponse est une décision économique.

**Modélisez l’application autour du domaine de l’entreprise**. Commencez par analyser les exigences métiers. Utilisez ces exigences pour modéliser l’application. Envisagez d’adopter une approche de conception pilotée par le domaine pour créer des [modèles de domaine][domain-model] qui reflètent les processus d’entreprise et les cas d’usage.

**Capturez les exigences fonctionnelles et non fonctionnelles**. Les exigences fonctionnelles vous permettent de déterminer si l’application exécute les actions adéquates. Les exigences non fonctionnelles vous aident à évaluer si l’application exécute ces actions *de façon efficace*. En particulier, vérifiez que vous connaissez bien vos exigences en matière d’extensibilité, de disponibilité et de latence. Ces exigences influenceront les décisions de conception et le choix de la technologie.

**Décomposez par charge de travail**. Dans ce contexte, le terme « charge de travail » renvoie à une fonctionnalité discrète ou à une tâche informatique, qui peuvent être séparées logiquement des autres tâches. Les différentes charges de travail peuvent présenter des exigences distinctes en matière de disponibilité, d’extensibilité, de cohérence des données et de récupération d’urgence.

**Planifiez dans une perspective de croissance**. Une solution peut répondre à vos besoins actuels en termes de nombre d’utilisateurs, de volume de transactions, de stockage des données, etc. Toutefois, une application robuste est en mesure de prendre en charge une croissance sans impliquer de modifications architecturales majeures. Consultez les articles [Penser la conception des applications pour augmenter la taille des instances](scale-out.md) et [Partitionner pour contourner les limites](partition.md). Tenez également compte du fait que votre modèle d’entreprise et vos exigences métiers sont susceptibles d’évoluer avec le temps. Si le modèle de service et les modèles de données d’une application sont trop rigides, il sera difficile de faire évoluer l’application pour de nouveaux scénarios et cas d’usage. Consultez l’article [Penser la conception des applications en vue de leur évolution](design-for-evolution.md).

**Gérez les coûts**. Dans une application locale traditionnelle, vous payez d’avance pour le matériel (dépenses d’investissement). Dans une application cloud, vous payez pour les ressources que vous consommez. Assurez-vous que vous comprenez le modèle de tarification relatif aux services que vous consommez. Le coût total inclut l’utilisation de la bande passante réseau, le stockage, les adresses IP, la consommation des services, ainsi que d’autres facteurs. Pour plus d’informations, consultez la page [Tarification Azure][pricing]. Prenez également en compte vos coûts d’exploitation. Dans le cloud, vous n’avez pas à gérer le matériel ni toute autre infrastructure, mais vous devez continuer à gérer vos applications, y compris les opérations DevOps, la réponse aux incidents, la récupération d’urgence, et ainsi de suite.

[domain-model]: https://martinfowler.com/eaaCatalog/domainModel.html
[pricing]: https://azure.microsoft.com/pricing/
