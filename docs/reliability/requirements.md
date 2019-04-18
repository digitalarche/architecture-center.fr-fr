---
title: Définition des exigences pour les applications Azure résilientes
description: Vue d’ensemble de la collecte des exigences de disponibilité et la résilience pour les applications Azure
author: MikeWasson
ms.date: 04/10/2019
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.openlocfilehash: 2af2ce4200c285abfda1395862d21efc3c17e577
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59646548"
---
# <a name="developing-requirements-for-resilient-azure-applications"></a>Développement de spécifications pour les applications Azure résilientes

Création *résilience* (récupération après des échecs) et *disponibilité* (en cours d’exécution dans un état sain, sans temps d’arrêt significatif) dans vos applications commence par collecter les impératifs. Par exemple, le temps d’inactivité est acceptable ? Combien les interruptions potentielles coûte votre entreprise ? Quelles sont les exigences de disponibilité de votre client ? Combien investir dans rendez votre application hautement disponible ? Quel est le risque et le coût ?

## <a name="identify-distinct-workloads"></a>Identifier les charges de travail distinctes

Solutions de cloud sont généralement consistent de plusieurs applications *charges de travail*. Une charge de travail est une fonctionnalité distincte ou une tâche qui est logiquement séparé des autres tâches en termes de besoins de stockage logique et les données. Par exemple, une application de commerce électronique peut avoir des charges de travail suivantes :

- Parcourir et rechercher un catalogue de produits.
- Créer et suivre des commandes.
- Afficher les recommandations.

Chaque charge de travail a des exigences différentes pour la disponibilité, évolutivité, la cohérence des données et récupération d’urgence. Votre prendre des décisions en équilibrant les coûts par rapport à des risques pour chaque charge de travail.

Également décomposez les charges de travail par objectif de niveau de service. Si un service est composé de charges de travail critiques et moins critiques, gérez-les de manière distincte et spécifiez les fonctionnalités du service et le nombre d’instances nécessaires pour répondre à leurs besoins de disponibilité.

## <a name="plan-for-usage-patterns"></a>Plan pour les modèles d’utilisation

Identifier les différences dans les exigences pendant les périodes critiques et non critiques. Y a-t-il certaines périodes critiques durant lesquelles le système doit être disponible ? Par exemple, une application de la déclaration d’impôts ne peut pas échouer pendant une date limite de dépôt et une service de diffusion en continu de vidéo ne doit pas rester pendant un événement en direct. Dans ces situations, évaluez le coût par rapport au risque.

- Pour garantir des temps d’activité et répondre aux contrats de niveau de service (SLA) dans des périodes critiques, planifier une redondance entre plusieurs régions en cas d’un échec, même si elle est plus onéreux.
- À l’inverse, au cours des périodes non critiques, exécutez votre application dans une seule région pour réduire les coûts.
- Dans certains cas, vous pouvez réduire les dépenses supplémentaires à l’aide des techniques sans serveur modernes qui ont de facturation basé sur la consommation.

## <a name="establish-availability-and-recovery-metrics"></a>Établir des mesures de disponibilité et récupération

Créer des numéros de ligne de base pour les deux ensembles de mesures dans le cadre du processus de configuration requise. Le premier jeu vous permet de déterminer où ajouter la redondance des services cloud et les contrats SLA pour fournir aux clients. Le deuxième ensemble de vous aider à que planifier votre récupération d’urgence.

### <a name="availability-metrics"></a>Métriques de disponibilité

Utiliser ces mesures pour planifier la redondance et de déterminer les contrats SLA du client.

- **Temps moyen de récupération (MTTR)** est la durée moyenne nécessaire à la restauration d’un composant après une défaillance.
- **Temps moyen entre défaillances (MTBF)** est la durée pendant laquelle un composant peut raisonnablement s’attendre à dernière entre pannes.

### <a name="recovery-metrics"></a>Mesures de récupération

Dériver de ces valeurs en effectuant une évaluation des risques et assurez-vous que vous comprenez les coûts et les risques de temps d’arrêt et pertes de données. Ces exigences non fonctionnelles d’un système et doivent être définis par les besoins de l’entreprise.

- **L’objectif de temps de récupération (RTO)** est la durée maximale acceptable pendant une application n’est pas disponible après un incident.
- **L’objectif de point de récupération (RPO)** est la durée maximale de perte de données qui est acceptable pendant une panne.

Si la valeur MTTR de *n’importe quel* composant critique dans une configuration à haute disponibilité dépasse le RTO de système, une défaillance dans le système peut entraîner une interruption inacceptable des opérations. Autrement dit, vous ne pouvez pas restaurer le système au sein de l’objectif RTO défini.

## <a name="determine-workload-availability-targets"></a>Déterminer les objectifs de disponibilité de la charge de travail

Définir vos propres contrats SLA cibles pour chaque charge de travail dans votre solution pour vous pouvez de déterminer si l’architecture répond aux exigences de l’entreprise.

### <a name="consider-cost-and-complexity"></a>Prendre en compte les coûts et la complexité

Tout le reste étant égal, une disponibilité plus élevée est préférable. Mais si vous cherchez plus de neuf, le coût et la complexité de croître. Une disponibilité de 99,99 % se traduit par environ cinq minutes de temps d’arrêt total par mois. Il est intéressant de la complexité supplémentaire et de coût pour atteindre cinq neuf ? La réponse dépend des exigences de l’entreprise.

Voici quelques autres considérations à prendre en compte lors de la définition d’un contrat SLA :

- Pour atteindre quatre neuf (99,99 %), vous ne pouvez pas compter sur une intervention manuelle pour récupérer après des défaillances. L’application doit pouvoir effectuer des diagnostics et se réparer elle-même.
- Au-delà de quatre neuf, il est difficile de détecter les pannes assez rapidement pour respecter le contrat SLA.
- Pensez à la fenêtre de temps contre laquelle est mesuré votre contrat SLA. Plus la fenêtre est petite, plus les tolérances seront strictes. Il est inutile de définir votre contrat SLA en termes de disponibilité de toutes les heures ou tous les jour.
- Utilisez les mesures MTBF et MTTR. Plus votre contrat SLA, moins fréquemment le service peut aller vers le bas et le plus rapidement le service doit récupérer.
- Obtenez l’accord de vos clients pour les objectifs de disponibilité de chaque élément de votre application et il de document. Sinon, votre conception ne répond pas aux attentes des clients.

### <a name="identify-dependencies"></a>Identifier les dépendances

Effectuer des exercices de mappage de dépendances pour identifier les dépendances internes et externes. Exemples incluent des dépendances relatives à la sécurité ou d’identité, telles que Active Directory, ou des services tiers tel qu’un fournisseur de paiement ou par courrier électronique de service de messagerie.

Une attention particulière aux dépendances externes qui peuvent être un point de défaillance unique ou provoquer des goulots d’étranglement. Si une charge de travail nécessite de 99,99 %, mais dépend d’un service avec un contrat SLA de 99,9 %, ce service ne peut pas être un point de défaillance unique dans le système. Une solution consiste à avoir un chemin d’accès de secours en cas d’échec du service. Vous pouvez également prendre d’autres mesures pour récupérer à partir d’un échec dans ce service.

Le tableau suivant présente les temps d’arrêt cumulatifs potentiels pour différents niveaux de contrat SLA.

| **CONTRAT SLA** | **Temps d’arrêt par semaine** | **Temps d’arrêt par mois** | **Temps d’arrêt par an** |
|---------|-----------------------|------------------------|-----------------------|
| 99 %     | 1,68 heure            | 7,2 heures              | 3,65 jours             |
| 99,9 %   | 10,1 minutes          | 43,2 minutes           | 8,76 heures            |
| 99,95 %  | 5 minutes             | 21,6 minutes           | 4,38 heures            |
| 99,99 %  | 1,01 minute          | 4,32 minutes           | 52,56 minutes         |
| 99, 999 % | 6 secondes             | 25,9 secondes           | 5,26 minutes          |

## <a name="understand-service-level-agreements"></a>Comprendre les contrats de niveau de service

Dans Azure, le [contrat de niveau de Service](https://azure.microsoft.com/en-us/support/legal/sla/) décrivent les engagements de Microsoft de fonctionnement et de connectivité. Si le contrat SLA pour un service particulier est de 99,9 %, vous devriez le service doit être disponible 99,9 % du temps. Différents services ont différents contrats SLA.

Le contrat SLA Azure inclut également des dispositions permettant d’obtenir un crédit de service si le contrat SLA n’est pas rempli, ainsi que les définitions spécifiques de *disponibilité* pour chaque service. Cet aspect du contrat SLA agit comme une stratégie d’application.

### <a name="composite-slas"></a>Contrats SLA composites

*Contrats SLA composites* impliquer plusieurs services prenant en charge une application, chacune avec différents niveaux de disponibilité. Par exemple, considérez une application web app Service qui écrit dans la base de données SQL Azure. Au moment de la rédaction, ces services Azure possèdent les contrats SLA suivants :

- Applications web App Service = 99,95 %
- SQL Database = 99,99%

Quel est le temps d’arrêt maximal attendu pour cette application ? En cas d’échec d’un service, l’application entière échoue. La probabilité de chaque échec du service est indépendante, et donc le contrat SLA composite pour cette application est de 99,95 % × 99,99 % = 99,94 %. Qui est inférieur aux contrats SLA individuels, ce qui n’est pas surprenant, car une application qui s’appuie sur plusieurs services possède plusieurs points de défaillance potentiels.

Vous pouvez améliorer le contrat SLA composite en créant des chemins de secours indépendants. Par exemple, si la base de données SQL n’est pas disponible, placez des transactions dans une file d’attente pour un traitement ultérieur.

![Contrat SLA composite](_images/composite-sla.png)

Grâce à cette conception, l’application est toujours disponible, même si elle ne peut pas se connecter à la base de données. Toutefois, cela échoue si la base de données et la file d’attente échouent en même temps. Le pourcentage de temps pour une défaillance simultanée attendu est 0,0001 × 0,001, donc le contrat SLA composite pour ce chemin d’accès combiné est :

- Base de données *ou* file d’attente = 1,0 − (0,0001 × 0,001) = 99,99999 %

Le contrat SLA composite total est de :

- Application Web *et* (base de données *ou* file d’attente) = 99,95 % × 99,99999 % = \~99,95 %

Il existe des compromis à cette approche. La logique d’application est plus complexe, vous payez pour la file d’attente et vous devez tenir compte des problèmes de cohérence des données.

### <a name="slas-for-multiregion-deployments"></a>Contrats SLA pour les déploiements multirégion

*Contrats SLA pour les déploiements multirégion* impliquent une technique de haute disponibilité pour déployer l’application dans plusieurs régions et utilisez Azure Traffic Manager pour basculer en cas d’échec de l’application dans une région.

Le contrat SLA composite pour un déploiement multirégion est calculé comme suit :

- *N* est le contrat SLA composite pour l’application déployée dans une région.
- *R* est le nombre de régions où l’application est déployée.

Les risques que l’application échoue dans toutes les régions en même temps soient ((1 − N) \^ R). Par exemple, si le contrat SLA de région unique est de 99,95 % :

- Le contrat SLA combiné pour les deux régions = (1 − (1 − 0.9995) \^ 2) = 99.999975 %
- Le contrat SLA combiné pour les quatre régions = (1 − (1 − 0.9995) \^ 4) = 99.999999 %

Le [contrat SLA pour Traffic Manager](https://azure.microsoft.com/en-us/support/legal/sla/traffic-manager/v1_0/) est également un facteur. Le basculement n’est pas instantané dans les configurations actif / passif, ce qui peuvent entraîner un temps d’arrêt lors d’un basculement. Consultez [surveillance des points de terminaison de Traffic Manager et le basculement](/azure/traffic-manager/traffic-manager-monitoring).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Architecte pour la résilience et disponibilité](./architect.md)
