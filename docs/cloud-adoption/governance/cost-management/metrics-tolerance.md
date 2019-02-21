---
title: 'Framework d’adoption du cloud : Mesures, indicateurs et tolérance au risque de Cost Management'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Raison de Cost Management dans le cadre de la gouvernance du cloud
author: BrianBlanchard
ms.openlocfilehash: 76e6b1b32dd862322f6cafd9aa63c6c4f79f4f5d
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901097"
---
# <a name="cost-management-metrics-indicators-and-risk-tolerance"></a>Mesures, indicateurs et tolérance au risque de Cost Management

Cet article a pour objectif de vous aider à quantifier la tolérance au risque de l’entreprise en lien avec Cost Management. La définition de mesures et d’indicateurs vous aide à créer une étude de rentabilité pour investir dans la maturité de la discipline Cost Management.

## <a name="metrics"></a>Mesures

Cost Management met généralement l’accent sur les mesures liées aux coûts. Dans le cadre de votre analyse des risques, vous aurez besoin de collecter des données relatives à vos dépenses actuelles et planifiées liées aux charges de travail cloud afin de déterminer le niveau de risque auquel vous êtes exposé et l’importance des investissements dans la gouvernance des coûts pour votre stratégie d’adoption du cloud.

Voici quelques exemples de mesures utiles que vous devez rassembler pour mieux évaluer la tolérance aux risques dans la discipline Base de référence de la sécurité :

- Dépenses annuelles : Coût annuel total des services fournis par un fournisseur de services cloud
- Dépenses mensuelles : Coût mensuel total des services fournis par un fournisseur de services cloud
- Ratio du prévisionnel par rapport au réel : Ratio de comparaison des dépenses prévues et des dépenses réelles (mensuelles ou annuelles)
- Ratio du rythme d’adoption (MOM) : Pourcentage du delta dans les coûts du cloud d’un mois à l’autre
- Coût cumulé : Total de dépenses quotidiennes cumulées, à partir du début du mois
- Tendances des dépenses : Tendance des dépenses par rapport au budget

## <a name="risk-tolerance-indicators"></a>Indicateurs de tolérance au risque

Au cours des déploiements précoces, tels que le développement/les tests ou les premières charges de travail expérimentales, Cost Management est susceptible de présenter un risque relativement faible. Au fur et à mesure que les ressources sont déployées, le risque augmente, et la tolérance au risque de l’entreprise est susceptible de diminuer. De plus, à mesure que les équipes d’adoption du cloud ont la possibilité de configurer ou de déployer des ressources dans le cloud, le risque augmente et la tolérance diminue. À l’inverse, le développement d’une discipline Cost Management nécessite des collaborateurs de la phase d’adoption du cloud pour le déploiement de nouvelles technologies plus innovantes.

Lors des premières étapes de l’adoption du cloud, vous collaborez avec votre entreprise pour déterminer une base de référence de tolérance au risque. Une fois que vous disposez d’une base de référence, vous devez déterminer les critères qui déclenchent un investissement dans la discipline Cost Management. Ces critères seront probablement différents pour chaque organisation.

Après avoir identifié les [risques métier](./business-risks.md), vous collaborez avec votre entreprise pour définir les bancs d’essai permettant d’identifier les déclencheurs d’augmentation potentielle de ces risques. Voici quelques exemples de mesures, telles que celles mentionnées ci-dessus, par rapport à votre tolérance aux risques de référence pour inciter votre entreprise à investir davantage dans Cost Management.

- Engagement justifié (cas le plus courant) : Entreprise qui s’engage à dépenser n 000 000 € cette année dans des services de fournisseur cloud. Elle a besoin d’une discipline Cost Management pour veiller à ne pas dépasser ses objectifs de dépenses de plus de 20 % et à utiliser au moins 90 % de ce budget.
- Déclencheur de pourcentage : Entreprise dont les dépenses cloud sont stables pour ses systèmes de production. En cas de hausse de plus X %, une discipline Cost Management s’avère être un investissement judicieux.
- Déclencheur de surapprovisionnement : Entreprise qui considère ses solutions déployées sont suraprovisionnées. Cost Management est un investissement prioritaire tant que l’entreprise ne peut pas démontrer que l’approvisionnement des ressources est parfaitement adapté à l’utilisation de celles-ci.
- Déclencheur de dépense mensuelle : Des dépenses mensuelles de plus de n 000 € constituent un coût important pour une entreprise. Si les dépenses dépassent ce montant au cours d’un mois donné, l’entreprise doit investir dans Cost Management.
- Déclencheur de dépense annuelle : Entreprise disposant d’un budget informatique de R&D qui permet de dépenser n 000 € par an pour l’expérimentation cloud. L’entreprise peut exécuter des charges de travail de production dans le cloud, mais le déclencheur est toujours considéré comme une solution expérimentale si le budget ne dépasse pas ce montant. En cas de dépassement du budget, celui-ci doit être traité comme un investissement de production, ce qui oblige à gérer étroitement les dépenses.
- Opposition aux dépenses d’exploitation (rare) : Une entreprise refuse de subir des dépenses d’exploitation et a besoin de disposer de contrôles Cost Management avant de déployer une charge de travail de développement/de test.

## <a name="next-steps"></a>Étapes suivantes

À l’aide du [modèle de gestion cloud ](./template.md), documenter les métriques et les indicateurs de tolérance correspondant au plan d’adoption du cloud actuel.

Sur la base des risques et de la tolérance, établir un processus de gouvernance et de communication en lien avec l’adhésion à la stratégie Cost Management.

> [!div class="nextstepaction"]
> [Établir des processus de conformité à la stratégie](compliance-processes.md)
