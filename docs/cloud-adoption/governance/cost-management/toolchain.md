---
title: 'Framework d’adoption du cloud : Outils de gestion des coûts dans Azure'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Outils de gestion des coûts dans Azure
author: BrianBlanchard
ms.openlocfilehash: 58dfa604863f704fd9b9fbb8d0693447cecdaf84
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900829"
---
# <a name="cost-management-tools-in-azure"></a>Outils de gestion des coûts dans Azure

La [gestion des coûts](overview.md) est l’une des [cinq disciplines de la gouvernance cloud](../governance-disciplines.md). Cette discipline vise essentiellement à établir des plans de dépenses cloud, à allouer des budgets cloud, à superviser et à appliquer les budgets cloud, à détecter les anomalies coûteuses et à ajuster le plan de gouvernance cloud en cas d’écarts dans les dépenses réelles.

La liste suivante énumère les outils natifs Azure qui peuvent contribuer à affiner les stratégies et les processus qui soutiennent cette discipline de gouvernance.

|  | [Portail Azure](https://azure.microsoft.com/features/azure-portal/)  | [Azure Cost Management](/azure/cost-management/overview-cost-mgt)  | [Azure EA Content Pack](/power-bi/service-connect-to-azure-enterprise)  | [Azure Policy](/azure/governance/policy/overview) |
|---------|---------|---------|---------|---------|
|Contrat Entreprise obligatoire ?     | Non          | Oui (pas obligatoire avec [Cloudyn](/azure/cost-management/overview))         | OUI         | Non          |
|Contrôle du budget     | Non          | OUI         | Non          | OUI         |
|Suivi des dépenses sur une même ressource    | OUI         | OUI         | OUI         | Non          |
|Suivi des dépenses sur plusieurs ressources    | Non          | OUI        | OUI         | Non          |
|Contrôle des dépenses sur une même ressource     | Oui - dimensionnement manuel         | OUI         | Non          | OUI         |
|Assurer le respect des dépenses sur plusieurs ressources    | Non          | OUI         | Non          | OUI         |
|Surveillance et détection des tendances     | Oui - limité         | OUI        | OUI         | Non          |
|Détection des anomalies de dépenses     | Non          | OUI        | OUI         | Non         |
|Communication des écarts     | Non         | OUI        | OUI        | Non         |
