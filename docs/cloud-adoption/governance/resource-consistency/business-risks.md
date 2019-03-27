---
title: 'Framework d’adoption du cloud : Risques métier et motivations associés à la cohérence des ressources'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Risques métier et motivations associés à la cohérence des ressources
author: alexbuckgit
ms.openlocfilehash: 19e0d761e4afa3473099bde2edc960c8b9eadb79
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58242460"
---
# <a name="resource-consistency-motivations-and-business-risks"></a>Risques métier et motivations associés à la cohérence des ressources

Cet article décrit les raisons pour lesquelles les clients adoptent une discipline Cohérence des ressources au sein d’une stratégie de gouvernance cloud. Des exemples sur les risques métier potentiels pouvant conduire à la rédaction d’instructions de stratégie sont également présentés.

<!-- markdownlint-disable MD026 -->

## <a name="is-resource-consistency-relevant"></a>La cohérence des ressources est-elle pertinente ?

En matière de déploiement de ressources et de charges de travail, le cloud offre davantage d’agilité et de flexibilité par rapport à la plupart des centres de données locaux traditionnels. Toutefois, ces avantages potentiels liés au cloud sont également associés à d’éventuels inconvénients en termes de gestion, qui peuvent grandement compromettre la réussite de votre adoption du cloud. Quelles ressources avez-vous déployées ? Quelles équipes possèdent quelles ressources ? Avez-vous suffisamment de ressources pour héberger une charge de travail ? Comment savez-vous que les charges de travail sont intègres ?

La cohérence des ressources est essentielle pour garantir que les ressources sont déployées, mises à jour et configurées de manière homogène et répétée, et s’assurer que les interruptions de service sont réduites et corrigées aussi vite que possible.

La discipline Cohérence des ressources se charge d’identifier et d’atténuer les risques métier associés aux aspects opérationnels de votre déploiement cloud. La cohérence des ressources comprend la surveillance des applications, charges de travail et performances des ressources. Elle inclut également les tâches nécessaires pour répondre aux demandes de mise à l’échelle, corriger les violations de SLA et prévenir de façon proactive les violations de contrat de niveau de service de performances via la correction automatisée.

Dans le cadre des déploiements de test initiaux, vous aurez seulement besoin d’adopter certains standards de marquage et de nommage rapides pour répondre à vos besoins en matière de cohérence des ressources. Dès que votre adoption du cloud gagnera en maturité et que vous déploierez des ressources critiques de plus en plus complexes, le besoin d’investir dans la discipline Cohérence des ressources s’intensifie rapidement.

## <a name="business-risk"></a>Risque métier

La discipline Cohérence des ressources tente de répondre aux principaux risques métier opérationnels. Collaborez avec vos équipes métier et informatiques pour identifier ces risques et surveiller la pertinence de chacun d’entre eux lors de la planification et de l’implémentation de vos déploiements cloud.

Les risques diffèrent en fonction de l’organisation, mais ceux présentés ci-dessous peuvent vous servir à lancer des discussions au sein de votre équipe de gouvernance cloud :

- **Coûts opérationnels inutiles**. Des ressources obsolètes ou inutilisées, ou des ressources surapprovisionnées quand la demande est faible génèrent des coûts opérationnels inutiles.
- **Ressources sous provisionnées**. Les ressources faisant l’objet d’une demande plus élevée que celle anticipée peuvent causer des interruptions de l’activité, car les ressources cloud se retrouvent submergées par cette demande.
- **Gestion inefficace**. L’absence de cohérence dans le marquage et le nommage des métadonnées associées aux ressources peut compliquer le travail du personnel informatique. Celui-ci peut en effet perdre du temps à rechercher les ressources des tâches de gestion ou à identifier les informations relatives à la propriété ou à la comptabilité des ressources. La gestion perd en efficacité, ce qui peut engendrer une augmentation des coûts et ralentir l’équipe informatique dans son travail de traitement des interruptions de service et autres problèmes opérationnels.
- **Interruption de l’activité**. Les interruptions de service entraînant des violations des SLA de votre organisation peuvent conduire à une perte d’activité et avoir d’autres impacts financiers pour votre entreprise.

## <a name="next-steps"></a>Étapes suivantes

À l’aide du [modèle de gestion du cloud](./template.md), documentez les risques métier susceptibles d’être introduits par le plan d’adoption du cloud actuel.

Une fois tous les risques métier réalistes appréhendés, l’étape suivante consiste à documenter la tolérance de l’activité aux risques, ainsi que les indicateurs et métriques clés permettant de surveiller cette tolérance.

> [!div class="nextstepaction"]
> [Comprendre les indicateurs, les métriques et la tolérance au risque](./metrics-tolerance.md)
