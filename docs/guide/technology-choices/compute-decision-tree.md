---
title: Arbre de décision des services de calcul Azure
titleSuffix: Azure Application Architecture Guide
description: Organigramme permettant de sélectionner un service de calcul.
author: MikeWasson
ms.date: 11/03/2018
ms.custom: seojan19
ms.openlocfilehash: 905b9956c9dcddddb21a87ea588af0ad5160ae2a
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/08/2019
ms.locfileid: "54114077"
---
# <a name="decision-tree-for-azure-compute-services"></a>Arbre de décision des services de calcul Azure

Azure offre de nombreuses manières d’héberger votre code d’application. Le terme *calcul* fait référence au modèle d’hébergement des ressources de calcul utilisées par votre application. L’organigramme suivant vous aidera à choisir un service de calcul pour votre application. Il vous guide au travers d’un ensemble de critères de décisions clé pour trouver une recommandation.

**Traitez cet organigramme comme un point de départ.** Chaque application dispose de ces exigences propres, utilisez donc la recommandation pour commencer. Réalisez ensuite une évaluation plus détaillée, en considérant les aspects suivants :

- Ensemble des fonctionnalités
- [Limites du service](/azure/azure-subscription-service-limits)
- [Coût](https://azure.microsoft.com/pricing/)
- [CONTRAT SLA](https://azure.microsoft.com/support/legal/sla/)
- [Disponibilité régionale](https://azure.microsoft.com/global-infrastructure/services/)
- Écosystème de développement et compétences en équipe
- [Table de comparaison des calculs](./compute-comparison.md)

Si votre application comprend plusieurs charges de travail, évaluez-les séparément. Une solution complète peut incorporer deux services de calcul ou plus.

Pour plus d’informations sur les options d’hébergement des conteneurs dans Azure, consultez [Conteneurs Azure](https://azure.microsoft.com/overview/containers/).

## <a name="flowchart"></a>Organigramme

![Arbre de décision des services de calcul Azure](../images/compute-decision-tree.svg)

## <a name="definitions"></a>Définitions

- L’**opération « lift-and-shift »** est une stratégie visant à migrer une charge de travail vers le cloud sans reconcevoir l’application ou modifier le code. Elle est également appelée *ré-hébergement*. Pour plus d’informations, consultez [Centre de migration Azure](https://azure.microsoft.com/migration/).

- **Optimisé pour le cloud** est une stratégie visant à migrer vers le cloud par la refactorisation d’une application, pour tirer parti des fonctionnalités cloud natives.

## <a name="next-steps"></a>Étapes suivantes

Pour connaître les autres critères à prendre en compte, consultez [Critères de sélection d’un service de calcul Azure](./compute-comparison.md).
