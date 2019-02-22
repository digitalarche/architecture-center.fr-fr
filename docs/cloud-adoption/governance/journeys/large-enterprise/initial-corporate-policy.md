---
title: 'Framework d’adoption du cloud : Grandes entreprises - Stratégie d’entreprise initiale sous-tendant la stratégie de gouvernance'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 2/11/2019
description: Grandes entreprises - Stratégie d’entreprise initiale sous-tendant la stratégie de gouvernance.
author: BrianBlanchard
ms.openlocfilehash: d3bc31f95229a82cbc2f6ac6e684a87107783c07
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901089"
---
# <a name="large-enterprise-initial-corporate-policy-behind-the-governance-strategy"></a>Grandes entreprises : Stratégie d’entreprise initiale sous-tendant la stratégie de gouvernance

La stratégie d’entreprise suivante définit la position de gouvernance initiale qui constitue le point de départ de ce parcours. Cet article définit les risques à un stade précoce, les déclarations de stratégie initiales et les processus à mettre en œuvre dès le début pour appliquer des déclarations de stratégie.

> [!NOTE]
>La stratégie d’entreprise n’est pas un document technique, mais oriente de nombreuses décisions techniques. Le MVP de gouvernance décrit dans cette [vue d’ensemble](./overview.md) découle de cette stratégie. Avant d’implémenter un MVP de gouvernance, votre organisation doit développer une stratégie d’entreprise basée sur ses propres objectifs et risques.

## <a name="cloud-governance-team"></a>Équipe de gouvernance cloud

La directrice informatique a récemment rencontré l'équipe de gouvernance informatique pour comprendre l'évolution des stratégies liées aux informations d'identification personnelle notamment, et examiner l'impact d'une modification de ces stratégies. Elle a également évoqué le potentiel global du cloud pour le service informatique et l'entreprise.

Après la réunion, deux membres de l'équipe de gouvernance informatique ont demandé l'autorisation de participer aux efforts de planification sur le cloud. Conscient du besoin de gouvernance et voyant là l'occasion de limiter l'informatique fantôme, le directeur de la gouvernance informatique a appuyé cette idée. C'est ainsi qu'a vu le jour l'équipe de gouvernance cloud. Au cours des prochains mois, ses membres corrigeront les nombreuses erreurs commises lors de l'exploration du cloud en termes de gouvernance. Cela leur vaudra le moniker des gardiens du cloud. Au cours des évolutions à venir, ce parcours démontrera la manière dont les rôles changent au fil du temps.

[!INCLUDE [business-risk](../../../../../includes/cloud-adoption/governance/business-risks.md)]

## <a name="tolerance-indicators"></a>Indicateurs de tolérance

La tolérance au risque actuelle est élevée et l’envie d’investir dans la gouvernance de cloud est faible. Par conséquent, les indicateurs de tolérance agissent comme un système d’avertissement précoce pour déclencher les investissements de temps et d’énergie. Si les indicateurs suivants sont observés, il apparaît judicieux de faire évoluer la stratégie de gouvernance.

- Gestion des coûts : La mise à l’échelle du déploiement dépasse 1 000 ressources dans le cloud, ou les dépenses dépassent 10 000 USD par mois.
- Base de référence des identités : Inclusion d’applications avec exigences d’authentification multifacteur tierces ou héritées.
- Base de référence de la sécurité : Inclusion de données protégées dans des plans définis d’adoption du cloud.
- Cohérence des ressources : Inclusion de toutes les applications stratégiques dans des plans définis d’adoption du cloud.

[!INCLUDE [policy-statements](../../../../../includes/cloud-adoption/governance/policy-statements.md)]

## <a name="next-steps"></a>Étapes suivantes

Cette stratégie d’entreprise prépare l’équipe de gouvernance du cloud à implémenter le MVP de gouvernance, qui constituera la base de l’adoption. L’étape suivante consiste à implémenter ce MVP.

> [!div class="nextstepaction"]
> [Bonne pratique, explication](./best-practice-explained.md)