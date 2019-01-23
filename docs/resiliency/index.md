---
title: Conception d’applications résilientes pour Azure
description: Comment créer des applications résilientes dans Azure, pour une haute disponibilité et une récupération d’urgence.
author: MikeWasson
ms.date: 12/18/2018
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.custom: resiliency
ms.openlocfilehash: ba3637ce90b793425b1238ee0d99d36a936da6ca
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54488594"
---
# <a name="designing-resilient-applications-for-azure"></a>Conception d’applications résilientes pour Azure

Dans un système distribué, des défaillances peuvent se produire. Le matériel peut être défaillant. Le réseau peut subir des échecs temporaires. Exceptionnellement, un service ou une région entiers peuvent rencontrer des interruptions, mais même ces dernières doivent être planifiées.

La création d’une application fiable dans le cloud est différente de la création d’une application fiable dans un environnement d’entreprise. Si par le passé vous avez acheté du matériel haut de gamme pour gagner en puissance, dans un environnement de cloud vous devez augmenter vos performances de manière horizontale, et pas verticale. Les coûts pour les environnements de cloud restent faibles via l’utilisation d’un matériel standard. Au lieu d’essayer d’éviter tous les échecs, l’objectif est de réduire les effets d’un échec au sein du système.

Cet article fournit une vue d’ensemble de la création d’applications résilientes dans Microsoft Azure. Il commence par une définition du terme *résilience* et de concepts associés. Il décrit ensuite un processus pour atteindre une résilience, à l’aide d’une approche structurée pendant la durée de vie d’une application, depuis la conception et l’implémentation jusqu’au déploiement et aux opérations.

<!-- markdownlint-disable MD026 -->

## <a name="what-is-resiliency"></a>Qu’est-ce que la résilience ?

<!-- markdownlint-enable MD026 -->

**La résilience** est la capacité d’un système à récupérer après des défaillances et à continuer de fonctionner. Il ne s’agit pas d’*éviter* les défaillances, mais d’y *répondre* en évitant les temps d’arrêt ou la perte de données. L’objectif de la résilience est que l’application retrouve un état entièrement fonctionnel suite à une défaillance.

Deux aspects importants de la résilience sont la haute disponibilité et la récupération d’urgence.

- **La haute disponibilité** (HA) est la capacité de l’application à continuer de s’exécuter dans un état sain, sans temps d’arrêt significatif. « Sain » signifie que l’application est réactive et que les utilisateurs peuvent s’y connecter et interagir avec elle.  
- **La récupération d’urgence** est la capacité à récupérer après des incidents rares mais majeurs : des échecs non temporaires et à grande échelle, telles que l’interruption de service qui peut affecter une région entière. La récupération d’urgence inclut la sauvegarde et l’archivage des données. Elle peut également inclure une intervention manuelle, comme la restauration d’une base de données à partir de la sauvegarde.

Une façon d’associer la haute disponibilité et la récupération d’urgence est que la récupération d’urgence démarre lorsque l’impact d’une erreur dépasse la capacité de la conception de haute disponibilité pour la manipuler.  

Lorsque vous concevez la résilience, vous devez comprendre vos exigences en matière de disponibilité. Quel temps d’arrêt maximal est acceptable ? Cela dépend en partie du coût. Combien les temps d’arrêt vont-ils coûter à votre entreprise ? Combien devriez-vous investir pour rendre l’application hautement disponible ? Vous devez également définir ce que cela signifie que l’application soit disponible. Par exemple, l’application est-elle « inactive » si un client peut envoyer une commande mais que le système ne peut pas la traiter dans les délais d’exécution habituels ? Envisagez également la probabilité qu’un type particulier de défaillance se produise, et si une stratégie d’atténuation est économique.

Un autre terme courant est la **continuité de l’activité**. Il s’agit de la capacité d’effectuer des fonctions opérationnelles essentielles pendant et après des conditions défavorables, comme une catastrophe naturelle ou la panne d’un service. La continuité de l’activité couvre l’intégralité des opérations de l’entreprise, y compris les installations physiques, les personnes, les communications, le transport, et l’informatique. Cet article se concentre sur les applications cloud, mais la planification de résilience doit être effectuée dans le contexte des exigences globales de continuité de l’activité.

La **sauvegarde des données** est une composante essentielle de la récupération d’urgence. Si les composants sans état d’une application subissent une défaillance, vous avez toujours la possibilité de les redéployer. Cependant, si des données sont perdues, le système ne peut retrouver sa stabilité. Les données doivent être sauvegardées, idéalement dans une région différente, ceci pour éviter tout risque en cas d’incident se produisant à l’échelle régionale.

La sauvegarde et la *réplication des données* sont deux processus distincts. La réplication des données consiste en leur copie quasiment en temps réel, de manière à ce que le système puisse basculer rapidement vers un réplica. De nombreux systèmes de bases de données prennent en charge la réplication. Par exemple, SQL Server prend en charge les groupes de disponibilité SQL Server AlwaysOn. La réplication des données peut réduire les intervalles de récupération après une panne, en garantissant la conservation permanente d’un réplica des données. Toutefois, la réplication des données ne vous protège pas contre les erreurs humaines. Les données corrompues en raison d’une erreur humaine sont elles aussi copiées sur les réplicas. Par conséquent, il vous faut intégrer une stratégie de sauvegarde à long terme dans votre stratégie de récupération d’urgence.

## <a name="process-to-achieve-resiliency"></a>Processus pour obtenir la résilience

La résilience n’est pas un module complémentaire. Elle doit être conçue dans le système et mise en pratique opérationnelle. Voici un modèle général à suivre :

1. **Définissez** vos exigences en matière de disponibilité, en fonction des besoins de l’entreprise.
2. **Concevez** l’application pour assurer la résilience. Démarrez avec une architecture qui suit des pratiques éprouvées, puis identifiez les points de défaillance possibles de cette architecture.
3. **Implémentez** des stratégies pour détecter les défaillances et récupérer en cas de panne.
4. **Testez** l’implémentation en simulant des erreurs et en déclenchant des basculements forcés.
5. **Déployez** l’application en production à l’aide d’un processus fiable et renouvelable.
6. **Surveillez** l’application pour détecter les défaillances. En surveillant le système, vous pouvez évaluer la santé de l’application et répondre aux incidents si nécessaire.
7. **Répondez** aux défaillances qui nécessitent des interventions manuelles.

Dans le reste de l’article, nous expliquons plus en détail chacune de ces étapes.

## <a name="define-your-availability-requirements"></a>Définir vos exigences de disponibilité

La planification de la résilience démarre avec les besoins de l’entreprise. Voici certaines approches pour aborder la résilience dans ces termes.

### <a name="decompose-by-workload"></a>Décomposez par charge de travail

De nombreuses solutions cloud sont constituées de plusieurs charges de travail d’application. Le terme « charge de travail » dans ce contexte renvoie à une fonctionnalité discrète ou une tâche informatique, qui peut être séparée logiquement des autres tâches, en termes de logique métier et de besoins de stockage des données. Par exemple, une application e-commerce peut inclure les charges de travail suivantes :

- Parcourir et rechercher un catalogue de produits.
- Créer et suivre des commandes.
- Afficher les recommandations.

Ces charges de travail peuvent avoir des exigences différentes concernant la disponibilité, la scalabilité, la cohérence des données et la reprise d’activité. L’entreprise doit choisir si elle préfère privilégier les coûts par rapport aux risques ou inversement.

Envisagez également des modèles d’utilisation. Y a-t-il certaines périodes critiques durant lesquelles le système doit être disponible ? Par exemple, un service de déclaration d’impôts ne peut pas s’arrêter juste avant la date limite de dépôt, un service de diffusion de vidéo en continu doit continuer de fonctionner pendant un événement sportif important, etc. Pendant les périodes critiques, vous pouvez avoir des déploiements redondants dans plusieurs régions. L’application pourrait donc rencontrer un échec en cas de défaillance d’une région. Toutefois, comme un déploiement multirégion est plus coûteux, durant les périodes moins critiques, vous pouvez exécuter l’application dans une seule région. Dans certains cas, les dépenses supplémentaires peuvent être atténuées avec des techniques serverless modernes, qui utilisent une facturation basée sur la consommation pour ne pas vous facturer les ressources de calcul sous-utilisées.

### <a name="rto-and-rpo"></a>RTO et RPO

Deux métriques importantes sont à prendre en considération : l’objectif de délai de récupération et l’objectif de point de récupération, puisqu’ils entrent dans le cadre de la reprise d’activité.

- L’**objectif de délai de récupération** (RTO) est la durée maximale acceptable pendant laquelle une application peut être indisponible après un incident. Si votre objectif RTO est de 90 minutes, vous devez être en mesure de remettre l’application en état de fonctionnement dans les 90 minutes à partir du début d’une panne. Si vous avez un objectif RTO très bas, vous pouvez conserver un deuxième déploiement régional exécuté sur une configuration active/passive en veille, pour vous protéger d’une panne régionale. Dans certains cas, vous pouvez déployer une configuration active/active pour obtenir un RTO encore plus bas.

- L’**objectif de point de récupération** (RPO) est la durée maximale de perte de données acceptable pendant une panne. Par exemple, si vous stockez des données dans une base de données unique, sans aucune réplication vers d’autres bases de données, et effectuez des sauvegardes toutes les heures, vous risquez de perdre jusqu’à une heure de données.

Les objectifs RTO et RPO sont des exigences non opérationnelles d’un système et doivent être définis selon les besoins de l’entreprise. Pour dériver ces valeurs, évaluez les risques ainsi que le coût d’une perte de données et d’un temps d’arrêt.

### <a name="mttr-and-mtbf"></a>MTTR et MTBF

Deux autres mesures courantes de la disponibilité sont le temps moyen de récupération (MTTR) et le temps moyen entre échecs (MTBF). Ces mesures sont généralement utilisées en interne par les fournisseurs de services pour déterminer les services cloud pour lesquels ajouter la redondance et les contrats SLA à fournir aux clients.

Le **temps moyen de récupération** (MTTR) est la durée moyenne nécessaire à la restauration d’un composant après un échec. Le MTTR est un fait empirique concernant un système. À partir du MTTR de chaque composant, vous pouvez estimer le MTTR d’une application entière. Une application créée à partir de plusieurs composants avec des valeurs MTTR basses est une application avec un MTTR général bas, c’est-à-dire qu’elle récupère rapidement après un échec.

Le **temps moyen entre échecs** (MTBF) est la durée d’exécution raisonnable d’un composant entre les pannes. Cette métrique peut vous aider à calculer la fréquence à laquelle un service devient indisponible. Un composant non fiable a un MTBF bas, ce qui entraîne un chiffre de contrat SLA bas pour ce composant. Toutefois, un MTBF bas peut être atténué en déployant plusieurs instances du composant qui peuvent être basculées l’une sur l’autre.

> [!NOTE]
> Si UNE valeur MTTR des composants d’une configuration à haute disponibilité dépasse le RTO du système, un échec dans le système entraîne une interruption inacceptable de l’activité. Le système ne peut alors pas être restauré dans l’objectif RTO défini.

### <a name="slas"></a>Contrats SLA

Dans Azure, les [contrats de niveau de service][sla] (SLA) décrivent les engagements de Microsoft en matière de temps de fonctionnement et de connectivité. Si le contrat SLA pour un service particulier est de 99,9 %, cela signifie que le service doit être disponible 99,9 % du temps.

> [!NOTE]
> Le contrat SLA Azure comprend également des dispositions permettant d’obtenir un crédit de service si le contrat SLA n’est pas rempli, ainsi que des définitions spécifiques de « disponibilité » pour chaque service. Cet aspect du contrat SLA agit comme une stratégie d’application.

Vous devez définir vos propres contrats SLA cibles pour chaque charge de travail dans votre solution. Un contrat SLA permet d’évaluer si l’architecture répond aux exigences de l’entreprise. Par exemple, si une charge de travail requiert une disponibilité de 99,99 %, mais dépend d’un service avec un contrat SLA de 99,9 %, ce service ne peut pas être un point de défaillance unique dans le système. Une solution consiste à posséder un chemin d’accès de secours en cas d’échec du service, ou à prendre d’autres mesures pour récupérer suite à un échec dans ce service.

Le tableau suivant présente les temps d’arrêt cumulatifs potentiels pour différents niveaux de contrat SLA.

| Contrat SLA | Temps d’arrêt par semaine | Temps d’arrêt par mois | Temps d’arrêt par an |
| --- | --- | --- | --- |
| 99 % |1,68 heure |7,2 heures |3,65 jours |
| 99,9 % |10,1 minutes |43,2 minutes |8,76 heures |
| 99,95 % |5 minutes |21,6 minutes |4,38 heures |
| 99,99 % |1,01 minute |4,32 minutes |52,56 minutes |
| 99, 999 % |6 secondes |25,9 secondes |5,26 minutes |

Bien entendu, une plus grande disponibilité est préférable, tout le reste étant égal. Mais si vous cherchez à obtenir davantage de 9, le coût et la complexité nécessaires pour atteindre ce niveau augmenteront. Une disponibilité de 99,99 % se traduit par environ 5 minutes de temps d’arrêt total par mois. L’augmentation du coût et de la complexité en vaut-elle la peine pour atteindre cinq 9 (99,999 %) de disponibilité ? La réponse dépend des exigences de l’entreprise.

Voici quelques autres considérations à prendre en compte lors de la définition d’un contrat SLA :

- Pour obtenir les quatre 9 (99,99 %), vous ne pouvez probablement pas avoir recours à une intervention manuelle pour récupérer en cas de défaillances. L’application doit pouvoir effectuer des diagnostics et se réparer elle-même.
- Au-delà de quatre 9, il devient difficile de détecter les pannes assez rapidement pour respecter les engagements du contrat SLA.
- Pensez à la fenêtre de temps contre laquelle est mesuré votre contrat SLA. Plus la fenêtre est petite, plus les tolérances seront strictes. Cela n’a pas de sens de définir votre contrat SLA en termes de temps de fonctionnement par heure ou par jour.
- Utilisez les mesures MTBF et MTTR. Plus votre SLA est bas, moins le service sera indisponible et plus vite il récupèrera.

### <a name="composite-slas"></a>Contrats SLA composites

Envisagez une application web App Service qui écrit à Azure SQL Database. Au moment de la rédaction, ces services Azure possèdent les contrats SLA suivants :

- Applications web App Service = 99,95%
- SQL Database = 99,99%

![Contrat SLA composite](./images/sla1.png)

Quel est le temps d’arrêt maximal attendu pour cette application ? En cas d’échec d’un service, l’application entière échoue. En règle générale, la probabilité d’échec de chaque service est indépendante, par conséquent le contrat SLA composite pour cette application est de 99,95 % &times; 99,99 % = 99,94 %. Ce taux est inférieur aux contrats SLA individuels, ce qui n’est pas surprenant, car une application qui repose sur plusieurs services possède davantage de points de défaillance potentiels.

En revanche, vous pouvez améliorer le contrat SLA composite en créant des chemins d’accès de secours indépendants. Par exemple, si SQL Database n’est pas disponible, placez des transactions dans une file d’attente pour un traitement ultérieur.

![Contrat SLA composite](./images/sla2.png)

Grâce à cette conception, l’application est toujours disponible, même si elle ne peut pas se connecter à la base de données. Toutefois, cela échoue si la base de données et la file d’attente échouent en même temps. Le pourcentage de temps attendu pour une défaillance simultanée est 0,0001 &times; 0,001, donc le contrat SLA composite pour ce chemin d’accès combiné est :  

- Base de données OU file d’attente = 1,0 &minus; (0,0001 &times; 0,001) = 99,99999 %

Le contrat SLA composite total est de :

- Application web ET (base de données OU file d’attente) = 99,95 % &times; 99,99999 % = ~99,95%

Mais il existe des compromis à cette approche. La logique d’application est plus complexe, vous payez pour la file d’attente, et il peut y avoir des problèmes de cohérence des données à prendre en compte.

**Contrat SLA pour déploiement multi-région**. Une autre technique de haute disponibilité consiste à déployer l’application dans plusieurs régions, et à utiliser Azure Traffic Manager pour basculer au cas où l’application échoue dans une région. Pour un déploiement multirégion, le contrat SLA composite est calculé comme suit.

*N* est le contrat SLA composite de l’application déployée dans une région, et *R* est le nombre de régions où l’application est déployée. Le risque que l’application échoue dans toutes les régions en même temps est ((1 &minus; N) ^ R).

Par exemple, si le contrat SLA d’une seule région est 99,95 %,

- le contrat SLA combiné des deux régions = (1 &minus; (0,9995 ^ 2)) = 99,999975 %
- le contrat SLA combiné pour quatre régions = (1 &minus; (0,9995 ^ 4)) = 99,999999 %

Vous devez aussi prendre en compte le [contrat SLA pour Traffic Manager][tm-sla]. Au moment de la rédaction, le contrat SLA pour Traffic Manager est de 99,99 %.

Par ailleurs, le basculement n’est pas instantané dans les configurations actives-passives, ce qui peut entraîner un temps d’arrêt pendant le basculement. Consultez [Surveillance et basculement des points de terminaison Traffic Manager][tm-failover].

Le numéro de contrat SLA calculé est une base utile, mais il ne dit pas tout sur la disponibilité. Souvent, il arrive qu’une application se dégrade normalement lorsqu’un chemin d’accès non critique échoue. Imaginez une application qui propose un catalogue de livres. Si l’application ne peut pas récupérer l’image miniature de la couverture, elle affichera une image de substitution. Dans ce cas, le fait que l’application n’ait pas réussi à obtenir l’image ne réduit pas son temps de fonctionnement, mais cela affecte l’expérience utilisateur.  

## <a name="design-for-resiliency"></a>Conception pour la résilience

Pendant la phase de conception, vous devez effectuer une analyse du mode d’échec (FMA). Une analyse FMA vise à identifier les points de défaillance possibles et à définir la manière dont l’application y répondra.

- Comment l’application détectera-t-elle ce type d’échec ?
- Comment l’application répondra-t-elle à ce type d’échec ?
- Comment enregistrerez-vous et surveillerez-vous ce type d’échec ?

Pour plus d’informations sur le processus FMA, avec des recommandations spécifiques pour Azure, consultez [Guide de résilience Azure : Analyse du mode d’échec][fma].

### <a name="example-of-identifying-failure-modes-and-detection-strategy"></a>Exemple d’identification des modes d’échec et de stratégie de détection

**Point d’échec :** Appel à un service web externe / une API.

| Mode d’échec | Stratégie de détection |
| --- | --- |
| Le service est indisponible |HTTP 5xx |
| Limitation |HTTP 429 (Trop de demandes) |
| Authentification |HTTP 401 (Non autorisé) |
| Temps de réponse lent |Délai d’expiration de la requête |

### <a name="redundancy-and-designing-for-failure"></a>Redondance et conception pour les défaillances

Les défaillances n’ont pas toutes la même incidence. Certaines défaillances matérielles, comme une panne de disque, peuvent affecter un seul ordinateur hôte. Un commutateur réseau défaillant peut impacter un rack entier de serveurs. Vous déplorerez moins fréquemment des défaillances perturbant un centre de données dans son ensemble, comme une panne d’alimentation. Exceptionnellement, une région entière peut être indisponible.

La redondance est l’un des moyens de rendre une application résiliente. Toutefois, vous devez planifier en fonction de cette redondance lorsque vous concevez l’application. Par ailleurs, le niveau de redondance dont vous avez besoin dépend des exigences de votre entreprise. Toutes les applications ne nécessitent pas une redondance entre les régions à titre de prévention contre les pannes régionales. En général, il existe un compromis entre redondance et fiabilité supérieures d’un côté contre complexité et coûts plus élevés de l’autre.  

Azure offre un certain nombre de fonctionnalités servant à rendre une application redondante à tous les nivaux de défaillances, d’une machine virtuelle unique à une région entière.

![Fonctionnalités de résilience Azure](./images/redundancy.svg)

**Machine virtuelle unique**. Azure fournit un [contrat SLA de durée de fonctionnement](https://azure.microsoft.com/support/legal/sla/virtual-machines) pour les machines virtuelles individuelles. (La machine virtuelle doit utiliser le stockage Premium pour tous les disques de système d’exploitation et disques de données). Bien que vous puissiez obtenir un contrat supérieur en exécutant deux machines virtuelles ou plus, une machine virtuelle unique peut présenter une fiabilité suffisante pour certaines charges de travail. Toutefois, pour les charges de travail de production, nous vous recommandons d’utiliser au moins deux machines virtuelles pour la redondance.

**Groupes à haute disponibilité**. Pour vous protéger contre les défaillances matérielles localisées, comme une panne de disque ou de commutateur réseau, déployez au moins deux machines virtuelles dans un groupe à haute disponibilité. Un groupe à haute disponibilité se compose d’au moins deux *domaines d’erreur* qui partagent une source d’alimentation et un commutateur réseau. Les machines virtuelles d’un groupe à haute disponibilité sont distribuées entre les domaines d’erreur. Ainsi, si une défaillance matérielle affecte un domaine d’erreur, le trafic réseau peut toujours être acheminé vers les machines virtuelles des autres domaines d’erreur. Pour plus d’informations sur les groupes à haute disponibilité, consultez la section [Gestion de la disponibilité des machines virtuelles Windows dans Azure](/azure/virtual-machines/windows/manage-availability).

**Zones de disponibilité**.  Une zone de disponibilité est une zone physiquement séparée au sein d’une région Azure. Chaque zone de disponibilité possède une source d’alimentation, un réseau et un système de refroidissement propres. Le déploiement des machines virtuelles entre les zones de disponibilité aide à protéger une application contre les défaillances à l’échelle du centre de données. Les régions ne prennent pas toutes en charge les zones de disponibilité. Pour obtenir la liste des régions et services pris en charge, consultez [Que sont les zones de disponibilité dans Azure ?](/azure/availability-zones/az-overview).

Si vous envisagez d’utiliser des zones de disponibilité dans votre déploiement, commencez par vérifier que votre architecture d’application et votre base de code peuvent prendre en charge cette configuration. Si vous déployez des logiciels du commerce, consultez le fournisseur des logiciels et effectuez des tests avant de les déployer en production. Une application doit être en mesure de maintenir son état et d’éviter la perte de données en cas de panne dans la zone configurée. L’application doit prendre en charge l’exécution dans une infrastructure distribuée élastique sans composant d’infrastructure codé en dur spécifié dans la base de code.

**Azure Site Recovery**.  Répliquez des machines virtuelles Azure dans une autre région Azure pour couvrir les besoins en termes de continuité d’activité et de reprise d’activité. Vous pouvez effectuer régulièrement des exercices de reprise d’activité pour garantir que vous répondez aux besoins de conformité. La machine virtuelle est répliquée avec les paramètres spécifiés dans la région sélectionnée afin que vous puissiez récupérer vos applications en cas de panne dans la région source. Pour plus d’informations, consultez [Replicate Azure VMs using ASR][site-recovery] (Répliquer des machines virtuelles Azure à l’aide d’ASR). Prenez en compte les chiffres de RTO et RPO pour votre solution ici et vérifiez pendant les tests que le point de récupération et le temps de récupération sont adaptés à vos besoins.

**Régions jumelées**. Pour protéger une application contre une panne régionale, vous pouvez la déployer dans plusieurs régions, en vous appuyant sur Azure Traffic Manager pour distribuer le trafic Internet entre les différentes régions. Chaque région Azure est jumelée à une autre région. Ensemble, elles forment une [paire régionale](/azure/best-practices-availability-paired-regions). Une région se trouve dans la même zone géographique que la région avec laquelle elle est jumelée (à l’exception de Brésil Sud) pour répondre aux exigences de la résidence de données en termes d’impôts et d’application de la loi.

Lorsque vous concevez une application multirégion, tenez compte du fait que la latence du réseau entre les régions et plus importante qu’au sein d’une région unique. Par exemple, si vous répliquez une base de données pour prendre en charge un basculement, utilisez la réplication des données synchrone au sein d’une région unique, mais la réplication des données asynchrones entre plusieurs régions.

| &nbsp; | Groupe à haute disponibilité | Zone de disponibilité | Azure Site Recovery/Région jumelée |
|--------|------------------|-------------------|---------------|
| Étendue de la défaillance | Rack | Centre de données | Région |
| Routage des requêtes | Load Balancer | Équilibreur de charge entre les zones | Traffic Manager |
| Latence du réseau | Très faible | Faible | Moyenne à élevée |
| Réseau virtuel  | Réseau virtuel | Réseau virtuel | Homologation de réseaux virtuels entre régions |

## <a name="implement-resiliency-strategies"></a>Implémenter des stratégies de résilience

Cette section contient une enquête concernant quelques stratégies de résilience courantes. La plupart d’entre elles ne sont pas limitées à une technologie particulière. Les descriptions de cette section résument l’idée générale de chaque technique, avec des liens pour obtenir des informations supplémentaires.

**Relancez des échecs temporaires**. Les échecs temporaires peuvent résulter d’une perte momentanée de la connectivité réseau, d’une connexion à une base de données interrompue ou d’un délai d’attente lorsqu’un service est occupé. Parfois, un échec temporaire peut être résolu simplement en relançant la requête. Pour de nombreux services Azure, le Kit de développement logiciel client implémente des relances automatiques, d’une façon transparente pour l’appelant ; consultez [Guide spécifique relatif au service de relance][retry-service-specific guidance].

Chaque tentative de relance augmente la latence totale. En outre, un trop grand nombre de demandes en échec peut provoquer un goulot d’étranglement, étant donné que les demandes s’accumulent dans la file d’attente. Ces demandes bloquées peuvent contenir des ressources système critiques telles que la mémoire, des threads, les connexions de la base de données, etc. Cela peut provoquer une succession d’échecs. Pour éviter ce problème, augmentez le délai entre chaque relance et limitez le nombre total de demandes en échec.

![Diagramme des nouvelles tentatives](./images/retry.png)

**Équilibrez la charge sur les instances**. Pour l’extensibilité, une application cloud doit être en mesure de monter en charge en ajoutant d’autres instances. Cette approche améliore également la résilience, car des instances défectueuses peuvent être supprimées de la rotation. Par exemple : 

- Placez au moins deux machines virtuelles derrière un équilibreur de charge. L’équilibreur de charge répartit le trafic vers toutes les machines virtuelles. Consultez [Exécuter des machines virtuelles à charge équilibrée à des fins d’extensibilité et de disponibilité][ra-multi-vm].
- Montez en charge une application Azure App Service à plusieurs instances. App Service équilibre automatiquement la charge entre les instances. Consultez [Application web de base][ra-basic-web].
- Utilisez [Azure Traffic Manager] [ tm] pour répartir le trafic sur un ensemble de points de terminaison.

**Répliquez des données**. La réplication de données est une stratégie générale pour gérer les échecs non temporaires dans un magasin de données. De nombreuses technologies de stockage fournissent une réplication intégrée, notamment Stockage Azure, Azure SQL Database, Cosmos DB et Apache Cassandra. Il est important de tenir compte des chemins d’accès de lecture et d’écriture. Selon la technologie de stockage, vous pouvez trouver plusieurs réplicas accessibles en écriture, ou un seul réplica accessible en écriture et plusieurs réplicas en lecture seule.

Pour optimiser la disponibilité, les réplicas peuvent être placés dans plusieurs régions. Toutefois, cela augmente la latence lors de la réplication des données. En règle générale, la réplication entre les régions est effectuée de manière asynchrone, ce qui implique un modèle de cohérence éventuel et une perte de données potentielle si un réplica échoue.

Vous pouvez utiliser [Azure Site Recovery][site-recovery] pour répliquer des machines virtuelles Azure d’une région vers une autre. Site Recovery réplique les données en continu dans la région cible. En cas de panne au niveau de votre site principal, vous basculez vers l’emplacement secondaire.

**Appliquer une dégradation normale**. Si un service échoue et qu’il n’existe aucun chemin d’accès de basculement, l’application devrait se dégrader normalement, tout en offrant une expérience utilisateur acceptable. Par exemple : 

- Placer un élément de travail dans une file d’attente, pour qu’il soit traité ultérieurement.
- Retourner une valeur estimée.
- Utiliser des données mises en cache localement.
- Afficher un message d’erreur pour l’utilisateur. (Cette option est préférable plutôt que l’application cesse de répondre aux demandes.)

**Limitez les utilisateurs à volume important**. Parfois, un petit nombre d’utilisateurs peut créer une charge excessive. Cela peut avoir un impact sur les autres utilisateurs, réduisant ainsi la disponibilité globale de votre application.

Si un seul client soumet un nombre excessif de demandes, l’application peut limiter le client pendant un certain temps. Pendant la période de limitation, l’application refuse certaines ou toutes les demandes de ce client (en fonction de la stratégie de limitation exacte). Le seuil de limitation peut dépendre du niveau de service du client.

La limitation n’implique pas que le client agissait à des fins malveillantes, uniquement qu’il a dépassé son quota de service. Dans certains cas, un consommateur peut constamment dépasser son quota, ou bien mal se comporter. Dans ce cas, vous pouvez aller plus loin et bloquer l’utilisateur. En règle générale, cela se fait en bloquant une clé API ou une plage d’adresses IP. Pour en savoir plus, consultez le [Modèle de limitation][throttling-pattern].

**Utilisez un disjoncteur**. Le modèle [Disjoncteur][circuit-breaker-pattern] peut empêcher qu’une application ne tente d’exécuter à plusieurs reprises une opération qui risque d’échouer. Le disjoncteur inclut dans un wrapper des appels vers un service et suit le nombre d’échecs récents. Si le nombre d’échec dépasse un seuil, le disjoncteur renvoie un code d’erreur sans avoir appelé le service. Le service a ainsi le temps de récupérer.

**Utilisez le nivellement de charge pour lisser les pics de trafic**.
Les applications peuvent rencontrer des pics soudains dans le trafic, ce qui peut surcharger les services sur le serveur principal. Si un serveur principal ne parvient pas à répondre aux demandes assez rapidement, cela peut provoquer un envoi des demandes en file d’attente (sauvegarde) ou une limitation de l’application par le service. Pour éviter ce problème, vous pouvez utiliser une file d’attente en tant que mémoire tampon. Lorsqu’il y a un nouvel élément de travail, au lieu d’appeler le serveur principal immédiatement, l’application met un élément de travail en file d’attente pour qu’il soit exécuté de façon asynchrone. La file d’attente agit comme une mémoire tampon qui lisse des pics de charge. Pour en savoir plus, consultez [Modèle de nivellement de la charge basé sur une file d’attente][load-leveling-pattern].

**Isolez les ressources critiques**. Des successions d’échecs peuvent parfois se produire dans un sous-système, provoquant des défaillances dans d’autres parties de l’application. Cela peut se produire si une défaillance empêche que certaines ressources, telles que des threads ou des sockets, ne soient libérées en temps voulu, menant à un épuisement des ressources.

Pour éviter ce problème, vous pouvez partitionner un système en groupes isolés, de manière à ce qu’une défaillance présente dans une partition ne détériore pas l’ensemble du système. Cette technique est parfois appelée le modèle de cloisonnement.

Exemples :

- Partitionnez une base de données (par exemple, par le client) et affectez un pool distinct d’instances de serveur web pour chaque partition.  
- Utilisez des pools de threads distincts pour isoler différents services. Cela permet d’éviter les successions d’échecs en cas de défaillance d’un des services. Pour obtenir un exemple, consultez la bibliothèque [Netflix Hystrix][hystrix].
- Utilisez des [conteneurs][containers] pour limiter les ressources disponibles pour un sous-système spécifique.

![Diagramme du modèle de cloisonnement](./images/bulkhead.png)

**Appliquez des transactions de compensation**. Une [transaction de compensation][compensating-transaction-pattern] est une transaction qui annule les effets d’une autre transaction terminée. Dans un système distribué, il est parfois difficile d’obtenir une cohérence transactionnelle élevée. Les transactions de compensation sont un moyen de garantir la cohérence à l’aide d’une série de transactions individuelles plus petites, qui peuvent être annulées à chaque étape.

Par exemple, pour réserver un voyage, un client peut réserver une voiture, un hôtel et un vol. Si une de ces étapes échoue, l’opération entière échoue. Au lieu de tenter d’utiliser une unique transaction distribuée pour l’intégralité de l’opération, vous pouvez définir une transaction de compensation pour chaque étape. Par exemple, pour annuler la réservation d’une voiture, vous annulez la réservation. Pour exécuter l’ensemble de l’opération, un coordinateur exécute chaque étape. Si une étape échoue, le coordinateur applique des transactions de compensation pour annuler toutes les étapes déjà effectuées.

## <a name="test-for-resiliency"></a>Tester la résilience

En règle générale, vous ne pouvez pas tester la résilience de la même façon que vous testez les fonctionnalités de l’application (en exécutant des tests unitaires, etc). Au lieu de cela, vous devez tester le fonctionnement de la charge de travail de bout en bout dans les conditions d’échec qui ne se produisent que par intermittence.

Les tests sont un processus itératif. Testez l’application, mesurez le résultat, analysez et résolvez tous les échecs résultants, puis répétez le processus.

**Test d’injection d’erreurs**. Testez la résilience du système lors de pannes, en déclenchant des échecs réels ou simulés. Voici quelques scénarios courants de défaillance à tester :

- Arrêt des instances de machine virtuelle.
- Blocage des processus.
- Expiration des certificats.
- Changement des clés d’accès.
- Arrêt du service DNS sur les contrôleurs de domaine.
- Limitation des ressources système disponibles, comme la RAM ou le nombre de threads.
- Démontage des disques.
- Redéploiement d’une machine virtuelle.

Mesurez les temps de récupération et vérifiez qu’ils répondent aux exigences de votre entreprise. Testez également des combinaisons de mode d’échec. Assurez-vous que les échecs ne produisent de manière successive et soient gérés de manière isolée.

C’est une autre raison qui démontre pourquoi il est important d’analyser les points de défaillance possibles pendant la phase de conception. Les résultats de cette analyse doivent être entrés dans votre plan de test.

**Test de charge**. Le test de charge est essentiel pour identifier les défaillances qui se produisent uniquement lors du chargement, telles que la saturation de la base de données principale ou la limitation du service. Testez la charge maximale à l’aide des données de production ou de données synthétiques qui soient aussi proches que possible des données de production. L’objectif est de voir comment se comporte l’application dans des conditions réelles.

**Exercices de reprise d’activité après sinistre**. La mise en place d’un plan de reprise d’activité après sinistre, aussi bon soit-il, ne suffit pas. Vous devez le tester régulièrement pour vérifier son bon fonctionnement dans une situation critique. Pour les machines virtuelles Azure, vous pouvez utiliser [Azure Site Recovery] [site-recovery] pour répliquer et [effectuer des exercices de reprise d’activité après sinistre] [site-recovery-test-failover] sans impacter les applications de production ni la réplication continue.

## <a name="deploy-using-reliable-processes"></a>Déployer à l’aide de processus fiables

Une fois qu’une application est déployée en production, les mises à jour deviennent une source possible d’erreurs. Dans le pire des cas, une mise à jour incorrecte peut entraîner un temps d’arrêt. Pour éviter cela, le processus de déploiement doit être prévisible et renouvelable. Le déploiement inclut l’approvisionnement des ressources Azure, le déploiement du code de l’application, et l’application des paramètres de configuration. Une mise à jour peut impliquer les trois, ou un sous-ensemble.

Le point essentiel est que les déploiements manuels sont sujets à l’erreur. Par conséquent, il est recommandé d’avoir un processus idempotent automatisé, que vous pouvez exécuter à la demande et exécutez de nouveau si quelque chose échoue.

- Utilisez les modèles Azure Resource Manager afin d’automatiser l’approvisionnement des ressources Azure.
- Utilisez la [Configuration de l’état souhaité Azure Automation][dsc] (DSC) pour configurer les machines virtuelles.
- Utilisez un processus de déploiement automatisé pour le code d’application.

Voici deux concepts liés au déploiement résilient : *l’infrastructure en tant que code* et *l’infrastructure immuable*.

- **L’infrastructure en tant que code** consiste à utiliser un code pour gérer et configurer l’infrastructure. L’infrastructure en tant que code peut utiliser une approche déclarative ou une approche impérative (ou une combinaison des deux). Les modèles Resource Manager sont un exemple d’approche déclarative. Les scripts PowerShell sont un exemple d’approche impérative.
- **L’infrastructure immuable** est le principe selon lequel vous ne devez pas modifier l’infrastructure après son déploiement en production. Dans le cas contraire, vous pouvez parvenir à un état dans lequel les modifications ad hoc ont été appliquées. Il est alors difficile de savoir avec exactitude ce qui a changé et de raisonner sur le système.

Une autre question qui se pose est comment déployer une mise à jour de l’application. Nous vous recommandons des techniques telles que le déploiement bleu-vert ou les versions de contrôle de validité, qui envoient les mises à jour de manière hautement contrôlée, afin de limiter les conséquences possibles d’un déploiement incorrect.

- [le déploiement bleu-vert][blue-green] est une technique dans laquelle une mise à jour est déployée dans un environnement de production distinct de l’application en temps réel. Après avoir validé le déploiement, passez le routage du trafic vers la version mise à jour. Cela est permis, par exemple, par les applications web Azure App Service avec des emplacements de préproduction.
- [Les versions de contrôle de validité][canary-release] sont similaires aux déploiements bleu-vert. Au lieu de passer tout le trafic vers la version mise à jour, vous déployez la mise à jour pour un petit pourcentage d’utilisateurs, en routant une partie du trafic vers le nouveau déploiement. Si un problème se produit, arrêtez et revenez à l’ancien déploiement. Dans le cas contraire, routez davantage du trafic vers la nouvelle version, jusqu’à ce qu’elle obtienne 100 % du trafic.

Peu importe l’approche que vous choisissez, assurez-vous que vous pouvez revenir au dernier déploiement correct, au cas où la nouvelle version ne fonctionne pas. En outre, le cas échéant, les journaux d’application doivent indiquer quelle version a provoqué l’erreur.

## <a name="monitor-to-detect-failures"></a>Surveiller et détecter des défaillances

La surveillance et les diagnostics sont cruciaux pour assurer la résilience. En cas de problème, vous devez savoir qu’il y a un échec, et vous devez en comprendre la cause.

La surveillance d’un système distribué à grande échelle constitue une difficulté importante. Pensez à une application qui s’exécute sur quelques dizaines de machines virtuelles &mdash; il n’est pas pratique de se connecter à chaque machine virtuelle une à une, et d’examiner les fichiers journaux, en essayant de résoudre un problème. De plus, le nombre d’instances de machine virtuelle n’est probablement pas statique. Les machines virtuelles sont ajoutées et supprimées tandis que l’application augmente et diminue la taille des instances. Parfois une instance peut échouer et doit être réapprovisionnée. En outre, une application cloud classique peut utiliser plusieurs banques de données (stockage Azure, SQL Database, Cosmos DB, cache Redis), et une seule action utilisateur peut couvrir plusieurs sous-systèmes.

Vous pouvez considérer le processus de surveillance et de diagnostics comme un pipeline avec plusieurs étapes distinctes :

![Contrat SLA composite](./images/monitoring.png)

- **Instrumentation**. Les données brutes pour la surveillance et les diagnostics proviennent de diverses sources, comprenant les journaux d’application, les journaux de serveur web, les compteurs de performance du système d’exploitation, les journaux de base de données et les diagnostics intégrés dans la plateforme Azure. La plupart des services Azure possèdent une fonctionnalité de diagnostics que vous pouvez utiliser pour déterminer la cause des problèmes.
- **Collecte et stockage**. Les données d’instrumentation brutes peuvent être contenues dans divers emplacements et sous des formats variés (par exemple, journaux des traces au niveau de l’application, les journaux IIS, les compteurs de performances). Ces sources distinctes sont collectées, consolidées et placées dans un stockage fiable.
- **Analyse et diagnostic**. Une fois que les données sont consolidées, elles peuvent être analysées pour résoudre les problèmes et fournir une vue d’ensemble de l’intégrité de l’application.
- **Visualisation et alertes**. Dans cette étape, les données de télémétrie sont présentées de manière à ce qu’un opérateur puisse repérer rapidement les problèmes ou les tendances. Les exemples comprennent des tableaux de bord ou des alertes par courrier électronique.  

La surveillance est différente de la détection de défaillance. Par exemple, votre application peut détecter un échec temporaire et réessayer, n’entraînant aucun temps d’arrêt. Mais il doit également connecter l’opération de relance pour que vous puissiez surveiller le taux d’échec, afin d’obtenir une vue d’ensemble de l’intégrité de l’application.

Les journaux d’application sont une source importante de données de diagnostic. Les meilleures pratiques pour la journalisation de l’application sont :

- Journalisation en production. Autrement, vous perdez de la vision où vous en avez le plus besoin.
- Journalisation des événements au niveau des limites de service. Vous devez inclure un ID de corrélation qui franchit les limites de service. Si une transaction passe par plusieurs services et que l’un d’entre eux échoue, l’ID de corrélation vous aidera à déterminer la raison pour laquelle la transaction a échoué.
- Utilisez la journalisation sémantique, également appelée journalisation structurée. Les journaux non structurés rendent difficile l’automatisation de la consommation et de l’analyse des données du journal, qui est nécessaire à l’échelle du cloud.
- Utilisez la journalisation asynchrone. Dans le cas contraire, le système de journalisation peut provoquer l’échec de l’application en provoquant la sauvegarde des demandes, comme elles sont bloquées en attendant d’écrire un événement de Journalisation.
- La journalisation de l’application est différente de l’audit. L’audit peut être effectué pour des raisons réglementaires ou de conformité. Par conséquent, les enregistrements d’audit doivent être complets, et il n’est pas acceptable d’en supprimer lors du traitement des transactions. Si une application nécessite un audit, il doit être maintenu séparé de la journalisation des diagnostics.

Pour plus d’informations sur la surveillance et les diagnostics, consultez [Guide de surveillance et de diagnostics][monitoring-guidance].

## <a name="respond-to-failures"></a>Répondre aux défaillances

Les sections précédentes ont porté sur les stratégies de récupération automatique, qui sont critiques pour la haute disponibilité. Toutefois, une intervention manuelle est parfois nécessaire.

- **Alertes**. Surveillez votre application pour trouver des signes précurseurs pouvant nécessiter une intervention proactive. Par exemple, si vous voyez que SQL Database ou Cosmos DB limitent régulièrement votre application, vous devrez peut-être améliorer la capacité de votre base de données ou optimiser vos requêtes. Dans cet exemple, même si l’application peut gérer les erreurs de limitation en toute transparence, votre télémétrie doit toujours déclencher une alerte afin que vous puissiez suivre l’activité.  
- **Basculement manuel**. Certains systèmes ne peuvent pas basculer automatiquement et nécessitent un basculement manuel. Pour les machines virtuelles Azure configurées avec [Azure Site Recovery][site-recovery], vous pouvez [effectuer un basculement][site-recovery-failover] et récupérer vos machines virtuelles dans une autre région en quelques minutes.
- **Test de disponibilité opérationnelle**. Si votre application bascule dans une région secondaire, vous devez effectuer un test de disponibilité opérationnelle avant de basculer vers la région principale. Le test permet de vérifier que la région principale est intègre et de nouveau prête à recevoir du trafic.
- **Vérification de la cohérence des données**. Si une défaillance se produit dans un magasin de données, des incohérences de données pourront être trouvées lorsque le magasin sera de nouveau disponible, en particulier si les données ont été répliquées.
- **Restauration à partir de la sauvegarde**. Par exemple, si SQL Database subit une panne régionale, vous pouvez effectuer une géo-restauration de la base de données à partir de la dernière sauvegarde.

Documentez et testez votre plan de récupération d’urgence. Évaluez l’impact commercial des défaillances d’application. Automatisez le processus autant que possible et documentez toutes les étapes manuelles, telles que le basculement manuel ou la restauration des données à partir des sauvegardes. Testez régulièrement votre processus de récupération d’urgence pour valider et améliorer le plan.

## <a name="summary"></a>Résumé

Cet article décrit la résilience d’un point de vue global, en mettant en évidence certains des défis uniques du cloud. Ceux-ci incluent la nature distribuée du cloud computing, l’utilisation de matériel standard, et la présence des défaillances réseau temporaires.

Voici les principaux points de l’article à retenir :

- La résilience entraîne une plus grande disponibilité et une réduction du délai moyen pour récupérer en cas de panne.
- L’obtention de la résilience dans le cloud requiert un ensemble de techniques diverses à partir de solutions traditionnelles locales.
- La résilience ne se produit pas par accident. Elle doit être conçue et intégrée dès le début.
- La résilience touche chaque partie du cycle de vie de l’application, depuis la planification et le codage jusqu’aux opérations.
- Testez et surveillez !

<!-- links -->

[blue-green]: https://martinfowler.com/bliki/BlueGreenDeployment.html
[canary-release]: https://martinfowler.com/bliki/CanaryRelease.html
[circuit-breaker-pattern]: https://msdn.microsoft.com/library/dn589784.aspx
[compensating-transaction-pattern]: https://msdn.microsoft.com/library/dn589804.aspx
[containers]: https://en.wikipedia.org/wiki/Operating-system-level_virtualization
[dsc]: /azure/automation/automation-dsc-overview
[contingency-planning-guide]: https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-34r1.pdf
[fma]: failure-mode-analysis.md
[hystrix]: https://medium.com/netflix-techblog/introducing-hystrix-for-resilience-engineering-13531c1ab362
[jmeter]: https://jmeter.apache.org/
[load-leveling-pattern]: ../patterns/queue-based-load-leveling.md
[monitoring-guidance]: ../best-practices/monitoring.md
[ra-basic-web]: ../reference-architectures/app-service-web-app/basic-web-app.md
[ra-multi-vm]: ../reference-architectures/virtual-machines-windows/multi-vm.md
[checklist]: ../checklist/resiliency.md
[retry-pattern]: ../patterns/retry.md
[retry-service-specific guidance]: ../best-practices/retry-service-specific.md
[sla]: https://azure.microsoft.com/support/legal/sla/
[throttling-pattern]: ../patterns/throttling.md
[tm]: https://azure.microsoft.com/services/traffic-manager/
[tm-failover]: /azure/traffic-manager/traffic-manager-monitoring
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager
[site-recovery]:/azure/site-recovery/azure-to-azure-quickstart/
[site-recovery-test-failover]:/azure/site-recovery/azure-to-azure-tutorial-dr-drill/
[site-recovery-failover]:/azure/site-recovery/azure-to-azure-tutorial-failover-failback/
