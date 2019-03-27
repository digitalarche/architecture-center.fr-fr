---
title: "Framework d'adoption du cloud : Outils d'accélération du déploiement dans Azure"
description: Outils d'accélération du déploiement dans Azure
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
author: BrianBlanchard
ms.openlocfilehash: cd00f2fa132f5f177ccc12f61be8b5342b71197d
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58247420"
---
# <a name="deployment-acceleration-tools-in-azure"></a>Outils d'accélération du déploiement dans Azure

L'[Accélération du déploiement](overview.md) est l'une des [cinq disciplines de la gouvernance du cloud](../governance-disciplines.md). Cette discipline met l'accent sur les moyens d'établir des stratégies pour régir la configuration ou le déploiement des ressources. Parmi les cinq disciplines de la gouvernance du cloud, la gouvernance de la configuration comprend des stratégies de déploiement, d'alignement de la configuration et de haute disponibilité/récupération d'urgence. Des activités manuelles ou des activités DevOps entièrement automatisées peuvent être utilisées. Dans les deux cas, les stratégies sont en grande partie similaires.

Les opérateurs, gardiens et architectes du cloud qui s'intéressent à la gouvernance sont tous susceptibles d'investir beaucoup de temps dans la discipline Accélération du déploiement, qui codifie les stratégies et les exigences dans le cadre de multiples efforts d'adoption du cloud. Les outils de cette chaîne d'outils sont importants pour l'équipe de gouvernance du cloud et ils doivent constituer une priorité absolue dans son parcours d'apprentissage.

La liste suivante énumère les outils natifs qui peuvent contribuer à faire mûrir les stratégies et les processus qui soutiennent cette discipline de gouvernance.

|  | Azure Policy | Groupes d'administration Azure | Modèles Azure Resource Manager | Azure Blueprints | Azure Resource Graph | Gestion des coûts Azure |
|---------|---------|---------|---------|---------|---------|---------|
|Implémentation de stratégies d'entreprise     |OUI |Non   |Non   |Non  | Non  |Non  |
|Application de stratégies dans les abonnements     |Obligatoire |OUI  |Non   |Non  | Non  |Non  |
|Déploiement des ressources définies     |Non  |Non   |OUI  |Non  | Non  |Non  |
|Création d'environnements entièrement conformes      |Obligatoire |Obligatoire  |Obligatoire  |OUI | Non  |Non  |
|Audit des stratégies      |OUI |Non   |Non   |Non  | Non  |Non  |
|Interrogation des ressources Azure      |Non  |Non   |Non   |Non  |OUI |Non  |
|Rapport sur le coût des ressources      |Non  |Non   |Non   |Non  |Non  |OUI |

Les outils supplémentaires suivants peuvent s'avérer nécessaires pour atteindre des objectifs spécifiques en matière d'accélération du déploiement. Ces outils sont souvent utilisés en dehors de l'équipe de gouvernance, mais restent considérés comme un aspect de la discipline Accélération du déploiement.

|  |Portail Azure  |Modèles Azure Resource Manager  |Azure Policy  | Azure DevOps | Sauvegarde Azure | Azure Site Recovery |
|---------|---------|---------|---------|---------|---------|---------|
|Déploiement manuel (ressource unique)     | OUI | OUI  | Non   | Pas efficace | Non  | OUI |
|Déploiement manuel (environnement complet)     | Pas efficace | OUI | Non   | Pas efficace | Non  | OUI |
|Déploiement automatisé (environnement complet)     | Non   | OUI  | Non   | OUI  | Non  | OUI |
|Mise à jour de la configuration d'une seule ressource     | OUI | OUI | Pas efficace | Pas efficace | Non  | Oui - Pendant la réplication |
|Mise à jour de la configuration d'un environnement complet     | Pas efficace | OUI | OUI | OUI  | Non  | Oui - Pendant la réplication |
|Gestion des dérives de configuration     | Pas efficace | Pas efficace | OUI  | OUI  | Non  | Oui - Pendant la réplication |
|Création d'un pipeline automatisé pour déployer du code et configurer des ressources (DevOps)     | Non  | Non  | Non  | OUI | Non  | Non  |
|Récupération des données lors d'une panne ou d'une violation du contrat de niveau de service     | Non  | Non  | Non  | OUI | OUI | OUI |
|Récupération des applications et des données lors d'une panne ou d'une violation du contrat de niveau de service     | Non  | Non  | Non  | OUI | Non  | OUI |

Outre les outils Azure natifs mentionnés ci-dessus, les clients utilisent souvent des outils tiers pour faciliter l'accélération du déploiement et les déploiements DevOps.
