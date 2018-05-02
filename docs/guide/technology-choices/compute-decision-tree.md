---
title: Arbre de décision des services de calcul Azure
description: Un organigramme pour sélectionner un service de calcul
author: MikeWasson
ms.date: 04/21/2018
ms.openlocfilehash: f5703f4906ca2ea6f825b383710eb4bd335f5043
ms.sourcegitcommit: d702b4d27e96e7a5a248dc4f2f0e25cf6e82c134
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/23/2018
---
# <a name="decision-tree-for-azure-compute-services"></a>Arbre de décision des services de calcul Azure

Azure offre de nombreuses manières d’héberger votre code d’application. Le terme *calcul* fait référence au modèle d’hébergement des ressources de calcul utilisées par votre application. L’organigramme suivant vous aidera à choisir un service de calcul pour votre application.
 
![](../images/compute-decision-tree.svg)

Il vous guide au travers d’un ensemble de critères de décisions clé pour trouver une recommandation. Chaque application dispose de ces exigences propres. Vous devez donc vous baser sur la recommandation pour commencer. Réalisez ensuite une analyse plus détaillée, en considérant les aspects suivants :
 
- Ensemble des fonctionnalités
- [Limites du service](/azure/azure-subscription-service-limits)
- [Coût](https://azure.microsoft.com/pricing/)
- [Contrat SLA](https://azure.microsoft.com/support/legal/sla/)
- [Disponibilité régionale](https://azure.microsoft.com/global-infrastructure/services/)
- Écosystème de développement et compétences en équipe
- [Table de comparaison des calculs](./compute-comparison.md)

Si votre application comprend plusieurs charges de travail, évaluez-les séparément. Une solution complète peut incorporer deux services de calcul ou plus.

