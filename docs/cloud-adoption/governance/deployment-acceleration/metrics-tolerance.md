---
title: 'Framework d’adoption du cloud : Métriques d’accélération du déploiement, indicateurs et tolérance au risque'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Métriques d’accélération du déploiement, indicateurs et tolérance au risque
author: alexbuckgit
ms.openlocfilehash: 87d6f9f67b98d5761aced560392c9d38fa682ee7
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58241210"
---
# <a name="deployment-acceleration-metrics-indicators-and-risk-tolerance"></a>Métriques d’accélération du déploiement, indicateurs et tolérance au risque

Cet article vise à vous aider à quantifier la tolérance au risque de l’activité en lien avec l’accélération du déploiement. La définition de métriques et d’indicateurs vous aide à créer une étude de rentabilité pour investir dans la maturité de la discipline Accélération du déploiement.

## <a name="metrics"></a>Mesures

La discipline Accélération du déploiement se concentre sur le déploiement, la mise à jour et la maintenance des ressources cloud configurées pour que les systèmes fonctionnent correctement. Les informations suivantes sont utiles lors de l’adoption de cette discipline de gouvernance cloud :

- **Objectif de délai de récupération (RTO) :** Il s’agit de la durée maximale acceptable pendant laquelle une application peut être indisponible après un incident.
- **Objectif de point de récupération (RPO) :** Il s’agit de la durée maximale de perte de données acceptable pendant une panne. Par exemple, si vous stockez des données dans une base de données unique, sans aucune réplication vers d’autres bases de données, et effectuez des sauvegardes toutes les heures, vous risquez de perdre jusqu’à une heure de données.
- **Temps moyen de récupération (MTTR) :** Il s’agit de la durée moyenne nécessaire à la restauration d’un composant après une défaillance.
- **Temps moyen entre les défaillances (MTBF) :** Il s’agit de la durée attendue pendant laquelle un composant peut raisonnablement s’exécuter entre les pannes. Cette métrique peut vous aider à calculer la fréquence à laquelle un service devient indisponible.
- **Contrats de niveau de service (SLA) :** Ce contrat peut inclure à la fois les engagements de Microsoft en matière de fonctionnement et de connectivité des services Azure, mais aussi les engagements pris par l’entreprise auprès de ses clients internes et externes.
- **Temps de déploiement :** Il s’agit du temps nécessaire pour déployer des mises à jour sur un système existant.
- **Ressources non conformes :** Il s’agit du nombre ou du pourcentage de ressources non conformes avec les stratégies définies.

## <a name="risk-tolerance-indicators"></a>Indicateurs de tolérance au risque

Les risques liés à l’accélération du déploiement dépendent en grande partie du nombre et de la complexité des systèmes cloud qui sont déployés pour votre entreprise. Plus votre patrimoine cloud s’agrandit, plus le nombre de systèmes déployés et la fréquence de mise à jour de vos ressources cloud augmentent. Avec les dépendances entre les ressources, il est de plus en plus important de veiller à ce que les ressources soient correctement configurées et de concevoir des systèmes capables de résilience, particulièrement lorsqu’une ou plusieurs ressources subissent des temps d’arrêt imprévus.

<!-- "en-us" location is required for the URL below. -->

Envisagez d’adopter une culture organisationnelle de type DevOps ou [DevSecOps](https://www.microsoft.com/en-us/securityengineering/devsecops) au tout début de votre parcours d’adoption du cloud. L’organisation informatique classique des entreprises sépare souvent les équipes chargées des opérations, de la sécurité et du développement. Bien souvent, ces équipes ne collaborent pas ou sont même hostiles les unes aux autres. C’est en reconnaissant ces défis au tout début du processus et en intégrant les parties prenantes principales de chacune des équipes que vous pouvez favoriser l’agilité dans votre adoption du cloud, tout en gardant la main sur la sécurité et la gouvernance.

Collaborez avec votre équipe DevSecOps et les parties prenantes de l’entreprise pour identifier les [risques commerciaux](business-risks.md) liés à la configuration, puis déterminez une ligne de base acceptable pour configurer la tolérance au risque. Cette section des instructions du framework d’adoption du cloud fournit des exemples, mais les risques et les bases de référence détaillés pour votre entreprise ou vos déploiements risquent d’être différents.

Une fois que vous avez une ligne de base, établissez des seuils minimaux représentant une augmentation inacceptable de vos risques identifiés. Ces seuils font office de déclencheurs lorsque vous devez prendre des mesures pour atténuer ces risques. Voici quelques exemples montrant comment des métriques liées à la configuration, comme celles décrites ci-dessus, peuvent justifier un investissement accru dans la discipline Accélération du déploiement.

- **Déclencheur relatif au contrat de niveau de service (SLA) :** Lorsqu’une entreprise n’est pas capable de respecter les contrats de niveau de service de ses clients externes ou partenaires internes, elle doit investir dans la discipline Accélération du déploiement afin de réduire les temps d’arrêt du système.
- **Déclencheur relatif au temps de récupération :** Lorsqu’une entreprise dépasse les seuils de temps de récupération définis après une défaillance du système, elle doit investir dans l’amélioration de sa discipline Accélération du déploiement pour réduire ou éliminer les pannes ou l’impact des temps d’arrêt des composants individuels.
- **Déclencheur relatif aux dérives de configuration :** Lorsque les principaux composants système d’une entreprise subissent des changements de configuration inattendus, des défaillances lors du déploiement ou des mises à jour de système, l’entreprise doit investir dans la discipline Accélération du déploiement pour identifier les causes racine et définir les actions correctives.  
- **Déclencheur relatif à la non conformité :** Si le nombre de ressources non conformes dépasse un seuil défini (soit un nombre total de ressources ou un pourcentage), l’entreprise doit investir dans les améliorations de la discipline Accélération du déploiement pour garantir que la configuration des ressources reste conforme tout au long du cycle de vie des ressources en question.
- **Déclencheur relatif au calendrier du projet :** Si le temps de déploiement des ressources et des applications d’une entreprise dépasse souvent un seuil défini, l’entreprise doit investir dans ses processus Accélération du déploiement pour introduire des déploiements automatisés appliqués à la cohérence et à la prévisibilité, ou bien les améliorer. Les temps de déploiement qui sont mesurés en jours ou en semaines indiquent généralement qu’une stratégie Accélération du déploiement n’est pas optimale.

## <a name="next-steps"></a>Étapes suivantes

À l’aide du [modèle de gestion cloud ](./template.md), documentez les métriques et les indicateurs de tolérance correspondant au plan d’adoption du cloud actuel.

Sur la base des risques et de la tolérance, établissez un processus de gouvernance et de communication en lien avec l’adhésion à la stratégie Accélération du déploiement.

> [!div class="nextstepaction"]
> [Établir des processus d’adhésion à la stratégie](compliance-processes.md)
