---
title: "Framework d’adoption du cloud : Guides de décision en matière d'architecture"
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Découvrez les guides de décision en matière d'architecture du framework d'adoption du cloud.
author: rotycenh
ms.openlocfilehash: b43047b5fc5b636bc84b9f28ec24846730e63672
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901062"
---
# <a name="architectural-decision-guides"></a>Guides de décision en matière d'architecture

Les guides de décision en matière d’architecture du framework d'adoption du cloud décrivent les modèles qui permettent de formuler des recommandations de conception de gouvernance cloud. Chaque guide de décision se concentre sur un composant d'infrastructure de base en matière de déploiement cloud et répertorie les modèles conçus pour prendre en charge des scénarios spécifiques de déploiement dans le cloud.

Lorsque vous commencez à établir la gouvernance cloud de votre organisation, des parcours de gouvernance exploitables fournissent une feuille de route de référence. Cela étant, ces parcours reposent sur des hypothèses en termes d'exigences et de priorités, qui ne reflètent pas systématiquement celles de votre organisation.
Ces guides de décision complètent les exemples de parcours de gouvernance en proposant d'autres modèles pour vous permettre d'aligner les choix de conception architecturale faits dans les exemples de recommandations de conception avec vos propres exigences.

## <a name="design-guidance-categories"></a>Catégories de recommandations de conception

Chacune des catégories suivantes représente une technologie de base de tout déploiement dans le cloud. Pour chaque choix des exemples de parcours de conception ne correspondant pas à vos besoins, examinez les options dans la section appropriée ci-dessous afin de sélectionner un modèle mieux adapté vos besoins.

[Abonnements](./subscriptions/overview.md) : Planifiez la conception d'abonnement et la structure de compte de votre déploiement dans le cloud pour répondre aux fonctionnalités de votre organisation en termes de propriété, de facturation et de gestion.

[Identité](./identity/overview.md) : Intégrez des services d’identité basés sur le cloud à vos ressources d’identité existantes pour prendre en charge le contrôle d’accès au sein de votre environnement informatique.

[Application de stratégies](./policy-enforcement/overview.md) : Définissez et appliquez des règles de stratégie organisationnelle pour les ressources et charges de travail que vous déployez dans le cloud.

[Cohérence des ressources](./resource-consistency/overview.md) : Vérifiez que le déploiement et l'organisation de vos ressources cloud s'alignent en termes de gestion des ressources et d'exigences de stratégie.

[Identification des ressources](./resource-tagging/overview.md) : Organisez vos ressources cloud de manière à ce qu'elles prennent en charge les modèles de facturation, les approches en matière de comptabilité cloud, le contrôle d'accès et l'efficacité opérationnelle. L'identification des ressources requiert un schéma d'attribution de noms et de métadonnées cohérent et bien organisé.

[Réseaux à définition logicielle](./software-defined-network/overview.md) : Déployez des charges de travail sécurisées dans le cloud moyennant le déploiement rapide et la modification des fonctionnalités de mise en réseau virtualisées. Les réseaux à définition logicielle peuvent prendre en charge des flux de travail agiles, isoler des ressources et intégrer des systèmes basés sur le cloud à votre infrastructure informatique existante.

[Chiffrement](./encryption/overview.md) : Sécurisez vos données sensibles à l’aide du chiffrement, un point important en matière de sécurité au sein d’un déploiement dans le cloud.

[Journaux et rapports](./log-and-report/overview.md) : Surveillez les données de journal générées par les ressources cloud. L'analyse des données vous renseigne sur l'intégrité des opérations, de la maintenance et le statut de mise en œuvre de la stratégie des charges de travail.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment les abonnements et les comptes servent de base à un déploiement dans le cloud.

> [!div class="nextstepaction"]
> [Conception des abonnements](subscriptions/overview.md)
