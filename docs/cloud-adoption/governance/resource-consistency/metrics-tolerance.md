---
title: 'Framework d’adoption du cloud : Métriques de cohérence des ressources, indicateurs et tolérance au risque'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Métriques de cohérence des ressources, indicateurs et tolérance au risque
author: BrianBlanchard
ms.openlocfilehash: 3a2561a6d1d81a6395bb256e921a7a26898b45a0
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901033"
---
# <a name="resource-consistency-metrics-indicators-and-risk-tolerance"></a>Métriques de cohérence des ressources, indicateurs et tolérance au risque

Cet article vise à vous aider à quantifier la tolérance au risque de l’activité en lien avec la cohérence des ressources. La définition de métriques et d’indicateurs vous aide à créer une étude de rentabilité pour investir dans la maturation de la discipline Cohérence des ressources.

## <a name="metrics"></a>Mesures

La discipline Cohérence des ressources se concentre sur la réponse aux risques liés à la gestion opérationnelle de vos déploiements cloud. Dans le cadre de votre analyse des risques, vous aurez besoin de collecter des données relatives à vos opérations informatiques pour déterminer le risque auquel vous êtes exposé et l’importance d’investir dans la gouvernance Cohérence des ressources pour vos déploiements cloud planifiés.

Chaque organisation a ses propres scénarios opérationnels, mais les éléments suivants représentent des exemples utiles des métriques à collecter lors de l’évaluation de la tolérance au risque dans la discipline Cohérence des ressources :

- **Ressources cloud**. Nombre total de ressources cloud déployées.
- **Ressources sans balise**. Nombre de ressources sans comptabilité requise, impact sur l’activité ou balises d’organisation.
- **Ressources sous-utilisées**. Nombre de ressources où les capacités de mémoire, d’UC et de réseau sont toutes constamment sous-utilisées.
- **Épuisement des ressources**. Nombre de ressources où les capacités de mémoire, d’UC et de réseau sont épuisées par la charge.
- **Âge de la ressource**. Temps écoulé depuis que la ressource a été déployée ou modifiée pour la dernière fois.
- **Disponibilité du service**. Pourcentage de temps d’activité réel des charges de travail hébergées dans le cloud par rapport au temps d’activité prévu.
- **Machines virtuelles en état critique**. Nombre de machines virtuelles déployées où un ou plusieurs problèmes critiques ont été détectés, qui doivent être corrigés afin de restaurer la fonctionnalité normale attendue.
- **Alertes par gravité**. Nombre total d’alertes sur une ressource déployée, décomposées selon la gravité.
- **Liens de sous-réseau défectueux**. Nombre de ressources confrontées à des problèmes de connectivité réseau.
- **Points de terminaison de service défectueux**. Nombre de problèmes avec des points de terminaison réseau externes.
- **Incidents d’intégrité de service du fournisseur de cloud**. Nombre d’interruptions ou d’incidents liés aux performances causés par le fournisseur de cloud.
- **Intégrité de la sauvegarde**. Nombre de sauvegardes activement synchronisées.
- **Intégrité de la récupération**. Nombre d’opérations de récupération effectuées avec succès.

## <a name="risk-tolerance-indicators"></a>Indicateurs de tolérance au risque

Les plateformes cloud offrent un ensemble de fonctionnalités de base permettant aux équipes de déploiement de gérer efficacement de petits déploiements sans planification ou processus supplémentaires importants. Par conséquent, les premières charges de travail de développement/test ou d’expérimentation incluant une quantité relativement faible de ressources cloud présentent un faible niveau de risque, et n’auront probablement pas besoin de beaucoup pour la mise en place d’une stratégie formelle Cohérence des ressources.

Cependant, à mesure que la taille de vos ressources cloud augmente, leur gestion devient de plus en plus difficile. Avec davantage de ressources dans le cloud, il est essentiel de pouvoir identifier la propriété de celles-ci et de contrôler celles qui sont utiles afin de minimiser les risques. À mesure que davantage de charges de travail stratégiques sont déployées dans le cloud, le temps d’activité du service devient plus critique et la tolérance en lien avec le coût potentiel de perturbations du service diminue rapidement.

Au début de l’adoption du cloud, collaborez avec votre équipe des opérations informatiques et les parties prenantes de l’entreprise pour identifier les [risques commerciaux](business-risks.md) liés à la cohérence des ressources, puis déterminez une ligne de base acceptable pour la tolérance au risque. Cette section du Framework d’adoption du cloud fournit des exemples, mais les risques et les lignes de base détaillés pour votre entreprise ou vos déploiements peuvent être différents.

Une fois que vous avez une ligne de base, établissez des seuils minimaux représentant une augmentation inacceptable de vos risques identifiés. Ces seuils font office de déclencheurs lorsque vous devez prendre des mesures pour atténuer ces risques. Vous trouverez ci-dessous quelques exemples montrant comment des métriques d’exploitation telles que celles décrites ci-dessus peuvent justifier un investissement accru dans la discipline Cohérence des ressources.

- **Déclencheur de marquage et de nommage**. Une entreprise disposant de plus de X ressources dépourvues des informations de marquage requises ou ne respectant pas les standards de nommage doit envisager d’investir dans la discipline Cohérence des ressources afin d’affiner ces standards et de garantir leur application cohérente aux ressources déployées dans le cloud.
- **Déclencheur de surapprovisionnement de ressources**. Si une entreprise a plus de X % de ses ressources utilisant régulièrement une très faible proportion de leurs capacités de mémoire, d’UC ou de réseau disponibles, il est conseillé d’investir dans la discipline Cohérence des ressources afin d’optimiser l’utilisation des ressources pour ces éléments.
- **Déclencheur de sous-approvisionnement de ressources**. Si une entreprise a plus de X % de ses ressources épuisant régulièrement la plupart de leurs capacités de mémoire, d’UC ou de réseau, il est conseillé d’investir dans la discipline Cohérence des ressources afin de s’assurer de disposer des ressources nécessaires pour éviter des interruptions de service.
- **Déclencheur d’âge de ressource**. Une entreprise disposant de plus de X ressources qui n’ont pas été mises à jour depuis plus de X mois pourrait tirer profit d’un investissement dans la discipline Cohérence des ressources visant à garantir que les ressources actives sont dûment corrigées et saines, tout en éliminant celles qui sont obsolètes ou inutilisées.  
- **Déclencheur de disponibilité du service**. Une entreprise ayant connu un temps disponibilité inférieur à X % de services stratégiques doit investir dans la discipline Cohérence des ressources afin d’améliorer la fiabilité de ses services.
- **Déclencheur d’intégrité de machine virtuelle**. Une entreprise dont plus de X % des machines virtuelles rencontrent un problème d’intégrité critique doit investir dans la discipline Cohérence des ressources pour identifier les problèmes et améliorer la stabilité des machines virtuelles.
- **Déclencheur d’intégrité du réseau**. Une entreprise dont plus de X % des sous-réseaux ou des points de terminaison de réseau rencontrent des problèmes de connectivité doit investir dans la discipline Cohérence des ressources afin d’identifier et de résoudre les problèmes de réseau.
- **Déclencheur de couverture de sauvegarde**. Une entreprise dont X % des ressources stratégiques ne disposent pas de sauvegarde à jour bénéficierait d’un investissement accru dans la discipline Cohérence des ressources afin de garantir une stratégie de sauvegarde cohérente.
- **Déclencheur d’intégrité de sauvegarde**. Une entreprise confrontée à plus de X % d’échecs des opérations de restauration doit investir dans la discipline Cohérence des ressources afin d’identifier les problèmes de sauvegarde et de garantir la protection des ressources importantes.

Les métriques et déclencheurs exacts à utiliser pour évaluer la tolérance au risque, et le niveau d’investissement dans la discipline Cohérence des ressources sont spécifiques à votre organisation, mais les exemples ci-dessus devraient constituer une base utile de discussion au sein de votre équipe de gouvernance cloud.  

## <a name="next-steps"></a>Étapes suivantes

À l’aide du [modèle de gestion cloud ](./template.md), documenter les métriques et les indicateurs de tolérance correspondant au plan d’adoption du cloud actuel.

Sur la base des risques et de la tolérance, établir un processus de gouvernance et de communication en lien avec l’adhésion à la stratégie Cohérence des ressources.

> [!div class="nextstepaction"]
> [Établir des processus de conformité à la stratégie](compliance-processes.md)
