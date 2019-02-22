---
title: 'Framework d’adoption du cloud : Outils de base de référence de la sécurité dans Azure'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Explication des outils capables d’améliorer la base de référence de la sécurité dans Azure
author: BrianBlanchard
ms.openlocfilehash: b316626c8ad717514f7f592abefa0f33a92afdca
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900885"
---
# <a name="security-baseline-tools-in-azure"></a>Outils de base de référence de la sécurité dans Azure

La discipline [Base de référence de la sécurité](overview.md) est l’une des [cinq disciplines de la gouvernance cloud](../governance-disciplines.md). Cette discipline se concentre sur l’établissement de stratégies qui protègent le réseau, les ressources et, plus important encore, les données qui résideront sur la solution d’un fournisseur de services cloud. Parmi les cinq disciplines de la gouvernance cloud, la discipline Base de référence de la sécurité inclut la classification du parc numérique et des données. Elle inclut également la documentation relative aux risques, à la tolérance métier et aux stratégies d’atténuation associées à la sécurité des données, des ressources et du réseau. D’un point de vue technique, cela englobe également la participation aux décisions concernant le [chiffrement](../../decision-guides/encryption/overview.md), la [configuration réseau requise](../../decision-guides/software-defined-network/overview.md), les [stratégies d’identités hybrides](../../decision-guides/identity/overview.md) et les outils d’[automatisation de l’application](../../decision-guides/policy-enforcement/overview.md) des stratégies de sécurité dans les [groupes de ressources](../../decision-guides/resource-consistency/overview.md).

La liste suivante énumère les outils Azure qui peuvent contribuer au développement des stratégies et des processus soutenant cette discipline de base de référence de la sécurité.

|                                                            | [Portail Azure](https://azure.microsoft.com/features/azure-portal/) / [Gestionnaire des ressources](/azure/azure-resource-manager/resource-group-overview)  | [Azure Key Vault](/azure/key-vault)  | [Azure AD](/azure/active-directory/fundamentals/active-directory-whatis) | [Azure Policy](/azure/governance/policy/overview) | [Centre de sécurité Azure](/azure/security-center/security-center-intro) | [Azure Monitor](/azure/azure-monitor/overview) |
|------------------------------------------------------------|---------------------------------|-----------------|----------|--------------|-----------------------|---------------|
| Appliquer des contrôles d’accès aux ressources et à la création de ressources   | OUI                             | Non               | OUI      | Non            | Non                     | Non             |
| Sécuriser les réseaux virtuels                                    | OUI                             | Non               | Non        | OUI          | Non                     | Non             |
| Chiffrer les disques virtuels                                     | Non                               | OUI             | Non        | Non            | Non                     | Non             |
| Chiffrer le stockage et bases de données PaaS                         | Non                               | OUI             | Non        | Non            | Non                     | Non             |
| Gérer les services d’identité hybride                            | Non                               | Non               | OUI      | Non            | Non                     | Non             |
| Restreindre les types de ressources autorisés                         | Non                               | Non               | Non        | OUI          | Non                     | Non             |
| Appliquer des restrictions géo-régionales                          | Non                               | Non               | Non        | OUI          | Non                     | Non             |
| Surveiller l’intégrité de la sécurité des réseaux et des ressources          | Non                               | Non               | Non        | Non            | OUI                   | OUI           |
| Détecter les activités malveillantes                                  | Non                               | Non               | Non        | Non            | OUI                   | OUI           |
| Détecter les vulnérabilités de manière préemptive                        | Non                               | Non               | Non        | Non            | OUI                   | Non             |
| Configurer la sauvegarde et la récupération d’urgence                     | OUI                             | Non               | Non        | Non            | Non                     | Non             |

Vous pouvez consulter une liste complète des outils et services de sécurité Azure dans l’article [Services et technologies de sécurité disponibles sur Azure](/azure/security/azure-security-services-technologies).

Il est également courant pour les clients d’utiliser des outils tiers pour faciliter les activités de base de référence de la sécurité. Pour plus d’informations, consultez l’article [Intégrer les solutions de sécurité dans Azure Security Center](/azure/security-center/security-center-partner-integration).

En plus des outils de sécurité, le [Microsoft Trust Center](https://www.microsoft.com/trustcenter/guidance/risk-assessment) contient de nombreux conseils, rapports et documents connexes qui peuvent vous aider à effectuer des évaluations des risques dans le cadre de votre processus de planification de migration.
