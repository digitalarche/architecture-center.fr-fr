---
title: Arbre de décision des services de calcul Azure
description: Un organigramme pour sélectionner un service de calcul
author: MikeWasson
ms.date: 04/21/2018
ms.openlocfilehash: 3dcfbd156d4fced863a56bcc8bb74483aa665f9f
ms.sourcegitcommit: 7ced70ebc11aa0df0dc0104092d3cc6ad5c28bd6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/11/2018
---
# <a name="decision-tree-for-azure-compute-services"></a>Arbre de décision des services de calcul Azure

Azure offre de nombreuses manières d’héberger votre code d’application. Le terme *calcul* fait référence au modèle d’hébergement des ressources de calcul utilisées par votre application. L’organigramme suivant vous aidera à choisir un service de calcul pour votre application. Il vous guide au travers d’un ensemble de critères de décisions clé pour trouver une recommandation. 

**Traitez cet organigramme comme un point de départ.** Chaque application dispose de ces exigences propres, utilisez donc la recommandation pour commencer. Réalisez ensuite une évaluation plus détaillée, en considérant les aspects suivants :
 
- Ensemble des fonctionnalités
- [Limites du service](/azure/azure-subscription-service-limits)
- [Coût](https://azure.microsoft.com/pricing/)
- [Contrat SLA](https://azure.microsoft.com/support/legal/sla/)
- [Disponibilité régionale](https://azure.microsoft.com/global-infrastructure/services/)
- Écosystème de développement et compétences en équipe
- [Table de comparaison des calculs](./compute-comparison.md)

Si votre application comprend plusieurs charges de travail, évaluez-les séparément. Une solution complète peut incorporer deux services de calcul ou plus.

![](../images/compute-decision-tree.svg)

