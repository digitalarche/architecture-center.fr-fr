---
title: Guide de surveillance et de diagnostic
titleSuffix: Best practices for cloud applications
description: Meilleures pratiques pour la surveillance des applications distribuées dans le cloud.
author: dragon119
ms.date: 07/13/2016
ms.custom: seodec18
ms.openlocfilehash: 82010fc57108d167d334d4a9f1ee669d2841cdcd
ms.sourcegitcommit: 4ba3304eebaa8c493c3e5307bdd9d723cd90b655
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2018
ms.locfileid: "53307416"
---
# <a name="monitoring-and-diagnostics"></a>Surveillance et diagnostics

Les applications et services distribués, exécutés dans le cloud sont, par leur nature, les parties complexes des logiciels comprenant de nombreux éléments mobiles. Dans un environnement de production, il est important de pouvoir contrôler la méthode avec laquelle les utilisateurs utilisent votre système, de suivre l’utilisation des ressources et généralement de surveiller l’intégrité et les performances de votre système. Vous pouvez utiliser ces informations comme aide au diagnostic pour détecter et corriger les problèmes, et également aider à identifier des problèmes potentiels et les empêcher de se produire.

## <a name="monitoring-and-diagnostics-scenarios"></a>Scénarios de surveillance et de diagnostic

Vous pouvez utiliser la surveillance pour mieux comprendre le fonctionnement d’un système. La surveillance est un élément essentiel du maintien des objectifs de qualité de service. Les scénarios courants pour la collecte des données de surveillance sont les suivants :

- S’assurer que le système reste sain.
- Suivre la disponibilité du système et de ses composants.
- Maintenir les performances pour s’assurer que le débit du système ne se dégrade pas inopinément à mesure que le volume de travail augmente.
- Garantir que le système satisfait aux contrats de niveau de service (SLA) établis avec les clients.
- Protéger la confidentialité et la sécurité du système, des utilisateurs et de leurs données.
- Suivre les opérations qui sont effectuées à des fins d’audit ou de réglementation.
- Surveiller l’utilisation quotidienne du système et identifier les tendances qui peuvent entraîner des problèmes si elles ne sont pas traitées.
- Suivre les problèmes qui surviennent, à partir d’un rapport initial via l’analyse des causes possibles, de la rectification, des mises à jour logicielles consécutives et du déploiement.
- Suivre les opérations et les versions de débogage.

> [!NOTE]
> Cette liste n’est pas exhaustive. Ce document porte essentiellement sur les scénarios correspondant aux situations les plus courantes de surveillance. Il peut en exister d’autres moins courantes ou propres à votre environnement.

Les sections suivantes décrivent ces scénarios plus en détail. Les informations pour chaque scénario sont décrites selon le format suivant :

1. Une brève vue d’ensemble du scénario.
2. Les conditions requises typiques de ce scénario.
3. Les données d’instrumentation brutes requises pour favoriser le scénario et les sources possibles de ces informations.
4. La méthode d’analyse et d’association de ces données brutes pour générer des informations de diagnostic dignes d’intérêt.

## <a name="health-monitoring"></a>Surveillance de l’intégrité

Un système est sain s’il est en cours d’exécution et capable de traiter les demandes. L’objectif de la surveillance de l’intégrité consiste à générer un instantané de l’intégrité actuelle du système pour vous permettre de vérifier que tous les composants du système fonctionnent comme prévu.

### <a name="requirements-for-health-monitoring"></a>Conditions requises pour la surveillance de l’intégrité

Un opérateur doit être averti rapidement (en quelques secondes) si une partie du système est considérée comme défectueuse. L’opérateur doit être en mesure de déterminer les parties du système qui fonctionnent normalement et celles qui rencontrent des problèmes. L’intégrité du système est mise en évidence via un système de feux de signalisation :

- Un feu rouge indique un état défectueux (le système est arrêté).
- Un feu jaune indique un état d’intégrité partiellement normal (le système s’exécute avec des fonctionnalités réduites).
- Un feu vert indique un état d’intégrité tout à fait normal.

Un système complet de surveillance de l’intégrité permet à un opérateur d’explorer le système pour afficher l’état d’intégrité des sous-systèmes et des composants. Par exemple, si l’ensemble du système est représenté comme partiellement sain, l’opérateur doit être en mesure de zoomer et de déterminer la fonctionnalité qui est actuellement indisponible.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Sources de données, instrumentation et conditions de collecte de données

Les données brutes requises pour la surveillance de l’intégrité peuvent être générées après les opérations suivantes :

- Le suivi de l’exécution des demandes utilisateur. Ces informations peuvent être utilisées pour déterminer les demandes qui ont réussi, celles qui ont échoué et le temps de réponse pour chaque demande.
- La surveillance des utilisateurs synthétiques. Ce processus simule les étapes effectuées par un utilisateur et suit une série d’étapes prédéfinie. Les résultats de chaque étape doivent être capturés.
- L’enregistrement des exceptions, erreurs et avertissements. Ces informations peuvent être capturées en résultat du suivi des instructions intégrées dans le code d’application, et de la récupération des informations des journaux des événements des services référencés par le système.
- La surveillance de l’intégrité de tous les services tiers utilisés par le système. Cette surveillance peut nécessiter la récupération et l’analyse des données d’intégrité fournies par ces services. Ces informations peuvent prendre différents formats.
- La surveillance de point de terminaison. Ce mécanisme est décrit plus en détail dans la section « Surveillance de la disponibilité ».
- La collecte des informations de performances ambiantes, telles que l’utilisation du processeur d’arrière-plan ou l’activité E/S (y compris le réseau).

### <a name="analyzing-health-data"></a>Analyse des données d’intégrité

Le principal objectif de la surveillance d’intégrité est d’indiquer rapidement si le système est en cours d’exécution. L’analyse à chaud des données immédiates peut déclencher une alerte si un composant critique est détecté comme défectueux, par exemple s’il ne parvient pas à répondre à une série consécutive de tests Ping.) L’opérateur peut prendre ensuite l’action corrective appropriée.

Un système plus avancé peut inclure un élément prédictif qui effectue une analyse à froid des charges de travail récentes et en cours. Une telle analyse permet d’identifier des tendances et de déterminer si le système est susceptible de rester dans un état d’intégrité normal ou s’il nécessite des ressources supplémentaires. Cet élément prédictif doit s’appuyer sur des métriques de performance critiques, par exemple :

- La fréquence des requêtes dirigées vers chaque service ou sous-système.
- Les temps de réponse de ces requêtes.
- Le volume des données échangées avec chaque service.

Si la valeur d’une métrique dépasse un seuil défini, le système peut déclencher une alerte pour permettre à un opérateur ou à la mise à l’échelle automatique (si disponible) de prendre les mesures préventives nécessaires pour maintenir l’intégrité du système. Ces actions peuvent impliquer l’ajout de ressources, le redémarrage d’un ou de plusieurs services défectueux, ou l’application d’une limitation aux demandes de faible priorité.

## <a name="availability-monitoring"></a>Surveillance de la disponibilité

Un système véritablement sain nécessite la disponibilité des composants et des sous-systèmes qui constituent le système. La surveillance de la disponibilité est étroitement liée à la surveillance de l’intégrité. Néanmoins, alors que la surveillance de l’intégrité fournit une vue immédiate de l’intégrité actuelle du système, la surveillance de la disponibilité concerne la disponibilité du système et de ses composants afin de générer des statistiques relatives au temps d’activité du système.

Dans de nombreux systèmes, certains composants (par exemple, une base de données) sont configurés avec une redondance intégrée pour permettre un basculement rapide en cas d’erreur grave ou d’une perte de connectivité. Dans l’idéal, les utilisateurs ne doivent pas être conscients qu’une telle défaillance s’est produite, mais du point de vue de la surveillance de la disponibilité, il est nécessaire de recueillir autant d’informations que possible sur ces défaillances pour en déterminer la cause et prendre les mesures correctives adéquates pour empêcher qu’elles se reproduisent.

Les données requises pour le suivi de la disponibilité peuvent dépendre de plusieurs facteurs de niveau inférieur. Nombre de ces facteurs peuvent être propres à l’application, au système et à l’environnement. Un système de surveillance efficace capture les données de disponibilité qui correspondent à ces facteurs de niveau inférieur, puis les agrège afin de donner une vue d’ensemble du système. Par exemple, dans un système de commerce électronique, la fonctionnalité d’entreprise qui permet à un client de passer des commandes peut dépendre du référentiel dans lequel les détails de la commande sont stockés et du système de paiement qui gère les transactions monétaires pour le paiement de ces commandes. La disponibilité de la partie du système liée à la passation des commandes est donc fonction de la disponibilité du référentiel et du sous-système de paiement.

### <a name="requirements-for-availability-monitoring"></a>Conditions requises pour la surveillance de la disponibilité

Un opérateur doit également être en mesure d’afficher la disponibilité historique de chaque système et sous-système et d’utiliser ces informations pour identifier les tendances susceptibles de provoquer des défaillances régulières d’un ou de plusieurs sous-systèmes. (Les défaillances des services commencent-elles à un moment précis de la journée qui correspond aux heures de traitement de pointe ?)

Une solution de surveillance doit fournir une vue immédiate et historique de la disponibilité ou de l’indisponibilité de chaque sous-système. Elle doit également être en mesure d’alerter rapidement un opérateur en cas d’échec d’un ou de plusieurs services ou si les utilisateurs ne peuvent pas se connecter aux services. Il ne s’agit pas uniquement de surveiller chaque service, mais également d’examiner les actions effectuées par chaque utilisateur en cas d’échec de ces actions lorsqu’il tente de communiquer avec un service. Dans une certaine mesure, un degré de problème de connectivité est normal et peut être dû à des erreurs temporaires. Cependant il peut être utile d’autoriser le système à déclencher une alerte pour un nombre d’échecs de connectivité à un sous-système spécifié se produisant au cours d’une période spécifique.

<!-- markdownlint-disable MD024 MD033 -->

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Sources de données, instrumentation et conditions de collecte de données

Comme pour la surveillance de l’intégrité, les données brutes requises pour prendre en charge la surveillance de la disponibilité peuvent être générées suite à la surveillance d’utilisateurs synthétiques et à l’enregistrement de l’ensemble des exceptions, erreurs et avertissements pouvant se produire. En outre, les données de disponibilité peuvent être obtenues à partir de la surveillance de point de terminaison. L’application peut exposer un ou plusieurs points de terminaison d’intégrité, chacun d’entre eux testant l’accès à une zone fonctionnelle du système. Le système de surveillance peut envoyer une commande ping sur chaque point de terminaison en suivant un calendrier défini, et collecter les résultats (réussite ou échec).

Tous les délais d’attente, les échecs de connectivité au réseau et les nouvelles tentatives de connexion doivent être enregistrés. Toutes les données doivent être horodatées.

### <a name="analyzing-availability-data"></a>Analyse des données de disponibilité

Les données d’instrumentation doivent être agrégées et corrélées pour prendre en charge les types d’analyse suivants :

- La disponibilité immédiate du système et des sous-systèmes.
- Les taux d’échec de la disponibilité du système et des sous-systèmes. Dans l’idéal, un opérateur doit être en mesure de mettre en corrélation les défaillances avec des activités spécifiques : que s’est-il passé lors de l’échec du système ?
- Une vue historique des taux d’échecs du système ou des sous-systèmes sur une période spécifiée et la charge sur le système (nombre de demandes utilisateur par exemple) lorsqu’une erreur s’est produite.
- Les raisons de l’indisponibilité du système ou des sous-systèmes. Les raisons peuvent être par exemple les suivantes : service n’étant pas en cours d’exécution, connectivité perdue, connecté mais expiration du délai d’attente et connecté mais renvoyant des erreurs.

Vous pouvez calculer le pourcentage de disponibilité d’un service sur une période de temps à l’aide de la formule suivante :

```console
%Availability =  ((Total Time – Total Downtime) / Total Time ) * 100
```

Cette formule est utile dans le cadre des contrats SLA. (La [surveillance des contrats SLA](#sla-monitoring) est décrite en détail plus loin dans ce guide.) La définition du *temps d’arrêt* dépend du service. Par exemple, le service de build de Visual Studio Team Services définit les temps d’arrêt comme la période (total des minutes cumulées) pendant laquelle le service de Build n’est pas disponible. Une minute est considérée comme indisponible si l’ensemble des requêtes HTTP continues auprès du service de build pour exécuter des opérations demandées par le client dans la minute entraînent un code d’erreur ou ne renvoient aucune réponse.

## <a name="performance-monitoring"></a>Analyse des performances

Comme le système est placé sous contrainte croissante (en augmentant le volume d’utilisateurs), la taille des jeux de données auxquels ces utilisateurs accèdent augmente, et l’échec d’un ou de plusieurs composants devient de plus en plus probable. Fréquemment, la défaillance d’un composant est précédée par une baisse des performances. Si vous êtes en mesure de détecter cette baisse, vous pouvez prendre des mesures proactives pour y remédier.

Les performances du système dépendent de plusieurs facteurs. Chaque facteur est généralement mesuré via des indicateurs de performance clé (KPI) tels que le nombre de transactions de base de données par seconde ou le volume de requêtes réseau traitées avec succès dans une période spécifiée. Certains de ces indicateurs peuvent être disponibles en tant que mesures de performances spécifiques, tandis que d’autres peuvent être dérivés d’une combinaison de métriques.

> [!NOTE]
> La détermination des bonnes ou mauvaises performances nécessite que vous compreniez le niveau de performance auquel le système doit être capable de fonctionner. Ceci implique d’observer le système pendant qu’il fonctionne sous une charge normale et de capturer les données de chaque indicateur KPI sur une période donnée. Cela peut impliquer de faire fonctionner le système sous une charge simulée dans un environnement de test et de collecter les données appropriées avant de déployer le système dans un environnement de production.
>
> Vous devez également vous assurer que la surveillance de la performance ne devient pas une charge sur le système. Vous pouvez ajuster dynamiquement le niveau de détail des données que le processus de surveillance des performances rassemble.

### <a name="requirements-for-performance-monitoring"></a>Conditions requises pour la surveillance des performances

Pour examiner les performances du système, un opérateur doit généralement consulter les informations suivantes :

- Les taux de réponse pour les demandes utilisateur.
- Le nombre de demandes utilisateur simultanées.
- Le volume du trafic réseau.
- Les vitesses auxquelles les transactions sont terminées.
- Le temps de traitement moyen des demandes.

Il peut également être utile de fournir des outils qui permettent à un opérateur de repérer des corrélations, par exemple :

- Le nombre d’utilisateurs simultanés par rapport au temps de latence des requêtes (temps nécessaire au démarrage du traitement d’une requête une fois que l’utilisateur l’a envoyée).
- Le nombre d’utilisateurs simultanés par rapport au temps de réponse moyen (temps nécessaire pour effectuer une requête après le début du traitement).
- Le volume de demandes par rapport au nombre d’erreurs de traitement.

Avec ces informations fonctionnelles de haut niveau, un opérateur doit être capable d’obtenir une vue détaillée des performances de chaque composant du système. Ces données sont généralement fournies via des compteurs de performances de niveau inférieur qui suivent les informations indiquées ci-dessous :

- L’utilisation de la mémoire.
- Le nombre de threads.
- Le temps de traitement du processeur.
- La longueur de la file d’attente de la requête.
- Les taux et erreurs d’E/S disque ou réseau.
- Le nombre d’octets écrits ou lus.
- Les indicateurs d’intergiciel, tels que la longueur de la file d’attente.

Toutes les visualisations doivent permettre à un opérateur de spécifier une période de temps. Les données affichées peuvent être un instantané de la situation actuelle et/ou une vue historique des performances.

Un opérateur doit être en mesure de déclencher une alerte basée sur une mesure des performances pour toute valeur spécifiée durant n’importe quel intervalle de temps spécifié.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Sources de données, instrumentation et conditions de collecte de données

Vous pouvez collecter les données de performances de haut niveau (débit, nombre d’utilisateurs simultanés, nombre de transactions commerciales, taux d’erreur, etc.) en surveillant la progression des requêtes des utilisateurs lorsqu’elles arrivent et passent par le système. Cela implique d’incorporer des instructions de suivi à des points clés dans le code d’application ainsi que des informations de minutage. L’ensemble des erreurs, exceptions et avertissements doit être capturé avec des données suffisantes pour leur mise en corrélation avec les requêtes qui les ont générés. Le journal des services IIS (Internet Information Services) est une autre source utile.

Vous devez également capturer, si possible, les données de performances pour les systèmes externes que l’application utilise. Ces systèmes externes peuvent fournir leurs propres compteurs de performances ou d’autres fonctionnalités pour demander des données de performances. Si ce n’est pas possible, enregistrez des informations telles que l’heure de début et de fin de chaque requête adressée à un système externe, ainsi que l’état (réussite, échec ou avertissement) de l’opération. Par exemple, vous pouvez utiliser une approche de type chronomètre pour chronométrer les requêtes : démarrer un minuteur au démarrage de la requête, puis l’arrêter lorsque la requête est terminée.

Les données de performances de niveau inférieur pour les composants individuels d’un système peuvent être disponibles via des fonctionnalités et des services tels que les compteurs de performances Windows et les diagnostics Microsoft Azure.

### <a name="analyzing-performance-data"></a>Analyse des données de performances

L’essentiel du travail d’analyse consiste à agréger les données de performances par type de requête utilisateur et/ou en fonction du sous-système ou du service auquel chaque requête est envoyée. L’ajout d’un article à un panier ou l’exécution du processus de validation dans un système de commerce électronique constituent des exemples de requête utilisateur.

Une autre exigence commune est de synthétiser les données de performances dans les centiles sélectionnés. Par exemple, un opérateur peut déterminer les temps de réponse pour 99 %, 95 % et 70 % des requêtes. Il peut exister des cibles de contrats SLA ou d’autres objectifs définis pour chaque centile. Les résultats en cours doivent être indiqués en temps quasi réel afin de détecter les problèmes immédiats. Ils doivent également être agrégés sur une plus longue période à des fins statistiques.

Dans le cas de problèmes de latence ayant un impact sur les performances, un opérateur doit être rapidement en mesure d’identifier la cause du goulot d’étranglement en examinant la latence de chaque étape effectuée pour chaque requête. Les données de performances doivent donc fournir un moyen de mettre en corrélation des mesures de performances pour chaque étape afin de les associer à une demande spécifique.

Selon les spécifications de la visualisation, il peut être utile de générer et de stocker un cube de données contenant les vues des données brutes. Ce cube de données peut autoriser des requêtes ad hoc complexes et l’analyse des informations de performances.

## <a name="security-monitoring"></a>Surveillance de la sécurité

Tous les systèmes commerciaux qui contiennent des données sensibles doivent implémenter une structure de sécurité. La complexité du mécanisme de sécurité est généralement fonction de la sensibilité des données. Dans un système qui requiert l’authentification des utilisateurs, vous devez enregistrer les éléments suivants :

- Toutes les tentatives de connexion, qu’elles échouent ou réussissent.
- Toutes les opérations effectuées par un utilisateur authentifié et les détails de toutes les ressources auxquelles il a accédé.
- Lorsqu’un utilisateur termine une session et se déconnecte.

La surveillance peut aider à détecter les attaques visant le système. Par exemple, un grand nombre de tentatives de connexion en échec peut indiquer une attaque par force brute. Un bond inattendu du nombre de requêtes peut être le résultat d’une attaque de déni de service distribué (DDoS). Vous devez être prêt à surveiller toutes les requêtes effectuées vers toutes les ressources indépendamment de la source de ces requêtes. Un système présentant un problème de connexion peut exposer accidentellement des ressources au monde extérieur sans nécessiter la connexion réelle d’un utilisateur.

### <a name="requirements-for-security-monitoring"></a>Conditions requises pour la surveillance de la sécurité

Les aspects les plus critiques de surveillance de la sécurité doivent rapidement permettre à un opérateur d’effectuer les actions suivantes :

- Détecter les tentatives d’intrusion par une entité non authentifiée.
- Identifier les tentatives des entités d’effectuer des opérations sur les données auxquelles aucun accès ne leur a été accordé.
- Déterminer si tout ou partie du système subit une attaque externe ou interne. (Par exemple, un utilisateur authentifié malveillant peut tenter d’arrêter le système.)

Pour prendre en charge ces conditions, un opérateur doit être informé des éléments suivants :

- Si un compte effectue des tentatives de connexion en échec répétées pendant une période spécifiée.
- Si un compte authentifié tente d’accéder plusieurs fois à une ressource interdite pendant une période spécifiée.
- Si un grand nombre de requêtes non authentifiées ou non autorisées ont lieu pendant une période spécifiée.

Les informations fournies à un opérateur doivent inclure l’adresse hôte de la source correspondant à chaque requête. Si des violations de sécurité se produisent régulièrement à partir d’une plage spécifique d’adresses, ces hôtes risquent alors d’être bloqués.

Une partie essentielle du maintien de la sécurité d’un système est de pouvoir détecter rapidement les actions qui s’écartent du modèle habituel. Des informations comme le nombre de demandes de connexion en échec et/ou réussies peuvent être affichées visuellement pour aider à détecter la présence d’un pic d’activité à un moment inhabituel. (Des utilisateurs se connectant à 3 h 00 et effectuant un grand nombre d’opérations alors que leur journée de travail commence à 9 h 00 en constituent un bon exemple). Ces informations permettent également de configurer une mise à l’échelle automatique à durée définie. Par exemple, si un opérateur observe que de nombreux utilisateurs se connectent régulièrement à un moment précis de la journée, il peut mettre en place des services d’authentification supplémentaires pour gérer le volume de travail, puis les arrêter à la fin du pic.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Sources de données, instrumentation et conditions de collecte de données

La sécurité est un aspect global de la plupart des systèmes distribués. Les données pertinentes sont susceptibles d’être générées en plusieurs points d’un système. Vous devez envisager d’adopter une approche SIEM pour rassembler les informations relatives à la sécurité résultant des événements déclenchés par l’application, l’équipement réseau, les serveurs, les pare-feu, les logiciels antivirus et d’autres éléments de prévention des intrusions.

La surveillance de la sécurité peut intégrer des données d’outils qui ne font pas partie de votre application. Ces outils peuvent inclure des utilitaires qui identifient les activités d’analyse de port par des agences externes ou des filtres réseau qui détectent les tentatives d’accès non authentifié à votre application et à vos données.

Dans tous les cas, les données collectées doivent permettre à un administrateur de déterminer la nature d’une attaque et de prendre les contre-mesures appropriées.

### <a name="analyzing-security-data"></a>Analyse des données de sécurité

Une fonctionnalité de la surveillance de la sécurité concerne la variété des sources à partir desquelles les données émanent. Les différents formats et le niveau de détail nécessitent souvent une analyse complexe des données capturées pour les relier à un thread d’informations cohérent. Hormis les cas les plus simples (comme la détection d’un grand nombre de connexions en échec ou des tentatives répétées d’accès non autorisé à des ressources critiques), il peut s’avérer impossible d’effectuer un traitement automatisé complexe des données de sécurité. Au lieu de cela, il peut être préférable d’écrire ces données horodatées, mais autrement que dans leur forme d’origine, dans un référentiel sécurisé pour permettre une analyse manuelle experte.

## <a name="sla-monitoring"></a>Surveillance des contrats SLA

De nombreux systèmes commerciaux qui prennent en charge les clients payants garantissent les performances du système sous la forme de contrats SLA. Pour l’essentiel, les contrats SLA stipulent que le système peut gérer un volume de travail défini au cours d’une période convenue, sans perdre les informations critiques. La surveillance des contrats SLA vise à garantir que le système peut respecter les contrats SLA mesurables.

> [!NOTE]
> La surveillance des contrats SLA est étroitement liée à la surveillance des performances. Cependant, alors que la surveillance des performances s’attache à veiller à ce que le système fonctionne de manière *optimale*, la surveillance des contrats SLA est régie par une obligation contractuelle qui définit ce que le terme *optimal* signifie réellement.

Les contrats SLA sont souvent définis en termes de :

- Disponibilité générale du système. Par exemple, une organisation peut garantir que le système sera disponible pendant 99,9 % du temps. Cela revient à un temps d’arrêt inférieur ou égal à 9 heures par an, soit environ 10 minutes par semaine.
- Débit opérationnel. Cet aspect est souvent exprimé en une ou plusieurs bornes hautes telles que la garantie que le système peut prendre en charge jusqu’à 100 000 requêtes utilisateur simultanées ou gérer 10 000 transactions commerciales simultanées.
- Temps de réponse opérationnel. Le système peut également donner des garanties pour la vitesse de traitement des requêtes. En voici un exemple : 99 % de toutes les transactions commerciales seront terminées en 2 secondes, et aucune transaction ne prendra plus de 10 secondes.

> [!NOTE]
> Certains contrats correspondant à des systèmes commerciaux peuvent également inclure des contrats SLA pour le service clientèle. En voici un exemple : toutes les demandes d’assistance entraînent une réponse dans les 5 minutes qui suivent, et 99 % de tous les problèmes sont traités intégralement en un jour ouvré. Un [suivi des problèmes](#issue-tracking) efficace (décrit plus loin de cette section) est essentiel au respect des contrats SLA de ce type.

### <a name="requirements-for-sla-monitoring"></a>Conditions requises pour la surveillance des contrats SLA

Au niveau le plus élevé, un opérateur doit être en mesure de déterminer d’un coup d’œil si le système respecte ou non les contrats SLA adoptés. Dans le cas contraire, l’opérateur doit être à même d’explorer et d’examiner les facteurs sous-jacents afin de déterminer les raisons conduisant à des performances inférieures aux normes.

Les indicateurs de niveau supérieur classiques pouvant être représentés visuellement sont les suivants :

- Le pourcentage de disponibilité du service.
- Le débit de l’application (mesuré en matière de transactions réussies et/ou opérations par seconde).
- Le nombre de demandes d’application réussies/échouées.
- Le nombre d’erreurs, d’exceptions et d’avertissements de l’application et du système.

Tous ces indicateurs doivent pouvoir être filtrés par une période de temps spécifiée.

Une application cloud sera probablement constituée de plusieurs sous-systèmes et composants. Un opérateur doit être en mesure de sélectionner un indicateur de niveau supérieur et de connaître sa composition à partir de l’intégrité des éléments sous-jacents. Par exemple, si le temps d’activité de l’ensemble du système tombe en dessous d’une valeur acceptable, un opérateur doit être en mesure de faire un zoom avant et d’identifier les éléments qui contribuent à cette défaillance.

> [!NOTE]
> La disponibilité du système doit être définie avec précaution. Dans un système qui utilise la redondance pour garantir une disponibilité maximale, des instances individuelles d’éléments peuvent échouer, mais le système peut rester fonctionnel. La disponibilité du système, telle que présentée par la surveillance de l’intégrité, doit indiquer la disponibilité agrégée de chaque élément et pas nécessairement si le système s’est en réalité arrêté. En outre, les défaillances peuvent être isolées. Par conséquent, même si un système spécifique n’est pas disponible, le reste du système peut le rester, mais avec des fonctionnalités réduites. (Dans un système de commerce électronique, une défaillance dans le système peut empêcher un client de passer des commandes, mais ce dernier peut toujours être en mesure de parcourir le catalogue de produits.)

À des fins d’alerte, le système doit être en mesure de déclencher un événement si l’un des indicateurs de niveau supérieur dépasse un seuil spécifié. Les détails de niveau inférieur des différents facteurs qui composent l’indicateur de niveau supérieur doivent être disponibles en tant que données contextuelles pour le système d’alerte.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Sources de données, instrumentation et conditions de collecte de données

Les données brutes requises pour prendre en charge la surveillance des contrats SLA sont semblables à celles qui sont requises pour la surveillance des performances, ainsi qu’à certains aspects de la surveillance de l’intégrité et de la disponibilité. (Voir ces sections pour plus de détails.) Vous pouvez capturer ces données en effectuant :

- La surveillance de point de terminaison.
- L’enregistrement des exceptions, erreurs et avertissements.
- Le suivi de l’exécution des requêtes utilisateur.
- La surveillance de la disponibilité des services tiers utilisés par le système.
- L’utilisation des métriques et des compteurs de performances.

Toutes les données doivent être chronométrées et horodatées.

### <a name="analyzing-sla-data"></a>Analyse des données des contrats SLA

Les données d’instrumentation doivent être agrégées pour générer une image des performances globales du système. Les données agrégées doivent également prendre en charge le zoom avant pour permettre l’examen des performances des sous-systèmes sous-jacents. Par exemple, vous devez pouvoir :

- Calculer le nombre total de requêtes utilisateur pendant une période spécifiée, et déterminer le taux de réussite et d’échec de ces requêtes.
- Combiner les temps de réponse des demandes utilisateur pour générer une vue d’ensemble des temps de réponse.
- Analyser la progression des requêtes utilisateur pour décomposer le temps de réponse global d’une requête en temps de réponse des éléments de travail individuels de cette requête.
- Déterminer la disponibilité générale du système comme pourcentage de temps d’activité pour une période spécifique.
- Analyser le pourcentage de temps de disponibilité des composants individuels et des services du système. Cela peut impliquer l’analyse des journaux générés par des services tiers.

De nombreux systèmes commerciaux doivent rapporter les chiffres réels de performances par rapport à ceux convenus dans les contrats SLA pour une période spécifiée, en général un mois. Ces informations peuvent être utilisées pour calculer les crédits ou autres formes de remboursement si les contrats SLA ne sont pas respectés pendant cette période. Vous pouvez calculer la disponibilité d’un service à l’aide de la technique décrite dans la section [Analyse des données de disponibilité](#analyzing-availability-data).

À des fins internes, une organisation peut également suivre le nombre et la nature des incidents ayant provoqué la défaillance des services. Apprendre à résoudre ces problèmes rapidement ou à les éliminer complètement contribue à réduire les temps d’arrêt et à respecter les stipulations des contrats SLA.

## <a name="auditing"></a>Audit

Selon la nature de l’application, il peut exister des réglementations statutaires ou d’autres réglementations légales qui spécifient les conditions requises pour l’audit des opérations des utilisateurs et l’enregistrement de toutes les données d’accès. L’audit peut prouver l’existence de liens entre des clients et des requêtes spécifiques. La non-répudiation est un facteur important dans de nombreux systèmes de commerce électronique pour maintenir la confiance entre le client et l’organisation responsable de l’application ou du service.

### <a name="requirements-for-auditing"></a>Conditions requises pour l’audit

Un analyste doit pouvoir suivre la séquence des opérations commerciales réalisées par les utilisateurs afin que vous puissiez reconstruire leurs actions. Cela peut être nécessaire simplement à des fins d’enregistrement ou dans le cadre d’une investigation.

Les informations d’audit sont très sensibles. Elles incluent probablement des données qui identifient les utilisateurs du système, ainsi que les tâches qu’ils effectuent. Pour cette raison, les informations d’audit sont, de préférence, affichées sous la forme de rapports qui sont disponibles uniquement pour les analystes approuvés plutôt que sous la forme d’un système interactif prenant en charge l’exploration des opérations graphiques. Un analyste doit pouvoir générer une série de rapports. Par exemple, les rapports peuvent répertorier toutes les activités des utilisateurs se produisant pendant une période spécifiée, détailler la chronologie de l’activité d’un seul utilisateur ou répertorier la séquence des opérations exécutées sur une ou plusieurs ressources.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Sources de données, instrumentation et conditions de collecte de données

Les principales sources d’informations d’audit peuvent inclure :

- Le système de sécurité qui gère l’authentification des utilisateurs.
- Les journaux de suivi qui enregistrent l’activité utilisateur.
- Les journaux de sécurité qui suivent toutes les requêtes réseau identifiables et non identifiables.

Le format des données d’audit et la manière dont elles sont stockées peuvent être dictés par des exigences réglementaires. Par exemple, il peut s’avérer impossible de nettoyer les données d’une quelconque façon. (Elles doivent être enregistrées dans leur format d’origine.) L’accès au référentiel dans lequel elles sont conservées doit être protégé pour empêcher toute falsification.

### <a name="analyzing-audit-data"></a>Analyse des données d’audit

Un analyste doit être en mesure d’accéder aux données brutes dans leur intégralité et dans leur forme d’origine. En plus de la nécessité de générer des rapports d’audit courants, les outils permettant d’analyser ces données sont susceptibles d’être spécialisés et maintenus à l’extérieur du système.

## <a name="usage-monitoring"></a>Surveillance de l’utilisation

La surveillance de l’utilisation suit la manière dont les fonctionnalités et les composants d’une application sont utilisés. Un opérateur peut utiliser les données collectées pour :

- Déterminer les fonctionnalités qui sont massivement utilisées et déterminer les zones réactives potentielles dans le système. Les éléments à trafic élevé peuvent tirer parti du partitionnement fonctionnel, voire de la réplication pour répartir la charge de manière plus uniforme. Un opérateur peut également utiliser ces informations pour déterminer les fonctionnalités rarement utilisées qui sont des candidats possibles à la suppression ou au remplacement dans une future version du système.

- Obtenir des informations sur les événements opérationnels du système dans le cadre d’un fonctionnement normal. Par exemple, dans un site de commerce électronique, vous pouvez enregistrer les informations statistiques sur le nombre de transactions et le volume des clients qui en sont responsables. Ces informations peuvent être utilisées pour la planification de la capacité lorsque le nombre de clients augmente.

- Détecter (peut-être indirectement) la satisfaction des utilisateurs grâce aux performances ou à la fonctionnalité du système. Par exemple, dans un système de commerce électronique, l’abandon régulier du panier par un grand nombre d’utilisateurs peut être dû à un problème lié à la fonctionnalité de validation.

- Générer des informations de facturation. Une application commerciale ou un service mutualisé peut facturer les clients pour les ressources qu’ils utilisent.

- Appliquer des quotas. Si un utilisateur dans un système mutualisé dépasse son quota payant de temps de traitement ou d’utilisation de ressources pendant une période spécifiée, leur accès ou le traitement peuvent être limités.

### <a name="requirements-for-usage-monitoring"></a>Conditions requises pour la surveillance de l’utilisation

Pour examiner l’utilisation du système, un opérateur doit généralement consulter les informations suivantes :

- Le nombre de requêtes traitées par chaque sous-système et dirigées vers chaque ressource.
- Le travail effectué par chaque utilisateur.
- Le volume de stockage de données occupé par chaque utilisateur.
- Les ressources auxquelles accède chaque utilisateur.

Un opérateur doit également être en mesure de générer des graphiques. Par exemple, un graphique peut indiquer les utilisateurs les plus gourmands en ressources ou bien les ressources ou fonctionnalités système les plus fréquemment sollicitées.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Sources de données, instrumentation et conditions de collecte de données

Le suivi de l’utilisation peut être effectué à un niveau relativement élevé. Il peut indiquer l’heure de début et de fin de chaque requête, ainsi que la nature de la requête (lecture, écriture, etc., en fonction de la ressource en question). Vous pouvez obtenir ces informations en effectuant :

- Le suivi des activités des utilisateurs.
- La capture des compteurs de performances qui mesurent l’utilisation de chaque ressource.
- La surveillance de la consommation des ressources par chaque utilisateur.

Pour des raisons de mesure, vous devez également être à même de mettre en corrélation les utilisateurs et les opérations qu’ils réalisent, et d’identifier les ressources utilisées par ces opérations. Les informations collectées doivent être suffisamment détaillées pour permettre une facturation précise.

## <a name="issue-tracking"></a>suivi des problèmes

Les clients et les autres utilisateurs peuvent signaler les problèmes si des événements ou un comportement inattendus se produisent dans le système. Le suivi des problèmes s’attache à gérer ces problèmes, à les associer à des efforts visant à résoudre tous les problèmes sous-jacents dans le système et à informer les clients des résolutions possibles.

### <a name="requirements-for-issue-tracking"></a>Conditions requises pour le suivi des problèmes

Les opérateurs effectuent souvent le suivi des problèmes à l’aide d’un système distinct qui leur permet d’enregistrer et de rapporter les détails des problèmes signalés par les utilisateurs. Ces détails peuvent inclure les tâches que l’utilisateur tentait d’effectuer, les symptômes du problème, la séquence des événements et tout message d’erreur ou d’avertissement ayant été émis.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Sources de données, instrumentation et conditions de collecte de données

L’utilisateur ayant signalé le problème en premier lieu est la source de données initiale pour les données de suivi des problèmes. Il peut être en mesure de fournir les données supplémentaires suivantes :

- Un vidage sur incident (si l’application inclut un composant qui s’exécute sur le Bureau de l’utilisateur).
- Un instantané d’écran.
- La date et l’heure auxquelles l’erreur s’est produite, ainsi que toute autre information environnementale comme l’emplacement de l’utilisateur.

Ces informations peuvent contribuer aux efforts de débogage et permettre d’établir un backlog pour les versions futures du logiciel.

### <a name="analyzing-issue-tracking-data"></a>Analyse des données de suivi des problèmes

Différents utilisateurs peuvent signaler un même problème. Le système de suivi des problèmes doit associer des rapports communs.

La progression de l’effort de débogage doit être enregistrée dans chaque rapport de problème. Lorsque le problème est résolu, le client peut être informé de la solution.

Si un utilisateur signale un problème dont la solution est connue dans le système de suivi des problèmes, l’opérateur doit pouvoir indiquer immédiatement la solution à l’utilisateur.

## <a name="tracing-operations-and-debugging-software-releases"></a>Suivi des opérations et versions logicielles de débogage

Lorsqu’un utilisateur signale un problème, il ne connaît bien souvent que son impact immédiat sur ses opérations. Il peut uniquement indiquer les résultats de sa propre expérience à un opérateur chargé de la maintenance du système. Ces expériences sont généralement un symptôme visible d’un ou de plusieurs problèmes fondamentaux. Dans de nombreux cas, un analyste devra examiner en détail la chronologie des opérations sous-jacentes afin d’établir la cause première du problème. Ce processus est appelé *analyse de la cause première*.

> [!NOTE]
> L’analyse de la cause première peut révéler des inefficacités dans la conception d’une application. Dans ces situations, il peut être possible de modifier les éléments concernés et de les déployer dans le cadre d’une version ultérieure. Ce processus nécessite un contrôle prudent et les composants mis à jour doivent être étroitement surveillés.

### <a name="requirements-for-tracing-and-debugging"></a>Conditions requises pour le suivi et le débogage

Pour le suivi d’événements inattendus et d’autres problèmes, il est essentiel que les données de surveillance fournissent suffisamment d’informations pour permettre à un analyste de retrouver l’origine de ces problèmes et de reconstruire la séquence des événements qui se sont produits. Ces informations doivent être suffisantes pour permettre à un analyste de diagnostiquer la cause première des problèmes. Un développeur peut alors apporter les modifications nécessaires pour les empêcher de se reproduire.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Sources de données, instrumentation et conditions de collecte de données

La résolution des problèmes peut impliquer le suivi de toutes les méthodes (et de leurs paramètres) appelées dans le cadre d’une opération de création d’une arborescence illustrant le flux logique à travers le système quand un client effectue une requête spécifique. Les exceptions et les avertissements générés par le système en raison de ce flux doivent être capturés et consignés.

Pour prendre en charge le débogage, le système peut fournir des hooks qui permettent à un opérateur de capturer les informations d’état à des moments cruciaux dans le système. Le système peut également fournir des informations étape par étape détaillées à mesure que les opérations sélectionnées progressent. La capture des données à ce niveau de détail peut imposer une charge supplémentaire sur le système et doit être un processus temporaire. Un opérateur utilise ce processus principalement lorsqu’une série d’événements très rares se produit et s’avère difficile à répliquer, ou lorsqu’une publication d’un ou de plusieurs éléments dans un système requiert une surveillance minutieuse pour s’assurer que les éléments fonctionnent comme prévu.

## <a name="the-monitoring-and-diagnostics-pipeline"></a>Pipeline de surveillance et de diagnostic

La surveillance d’un système distribué à grande échelle constitue une difficulté importante. Il convient de ne pas considérer nécessairement de manière isolée chacun des scénarios décrits dans la section précédente. Il existera vraisemblablement un chevauchement significatif des données de surveillance et de diagnostic requises pour chaque situation, bien que ces données puissent devoir être traitées et présentées de différentes manières. Pour ces raisons, vous devez adopter une approche holistique de la surveillance et du diagnostic.

Vous pouvez envisager l’ensemble du processus de surveillance et de diagnostic comme un pipeline qui comprend les étapes présentées à la figure 1.

![Étapes du pipeline de surveillance et de diagnostic](./images/monitoring/Pipeline.png)

*Figure 1 : Les étapes du pipeline de supervision et de diagnostic.*

La figure 1 montre comment les données de surveillance et de diagnostic peuvent provenir de diverses sources de données. Les étapes d’instrumentation et de collecte portent sur l’identification des sources à partir desquelles les données doivent être capturées, la détermination des données à capturer, et la capture et la mise en forme de ces données afin de pouvoir les examiner facilement. L’étape d’analyse et de diagnostic prend les données brutes et les utilise pour générer des informations pertinentes pouvant être utilisées par un opérateur afin de déterminer l’état du système. L’opérateur peut utiliser ces informations pour prendre des décisions concernant les actions possibles à effectuer, puis réinjecter les résultats dans les étapes d’instrumentation et de collecte. L’étape de visualisation et d’alerte présente une vue utilisable de l’état du système. Elle permet d’afficher des informations en temps quasi réel à l’aide d’une série de tableaux de bord. Elle permet également de générer des rapports, des graphiques et des tableaux pour fournir une vue historique des données qui peut contribuer à l’identification des tendances à long terme. Si les informations montrent qu’un indicateur KPI est susceptible de dépasser les limites acceptables, cette étape peut également déclencher une alerte destinée à un opérateur. Dans certains cas, une alerte peut également servir à déclencher un processus automatisé qui tente de prendre des mesures correctives, telles que la mise à l’échelle automatique.

Notez que ces étapes constituent un processus de flux continu dans lequel les étapes s’effectuent en parallèle. Dans l’idéal, toutes les étapes doivent être configurables de manière dynamique. À certains moments, notamment lorsqu’un système a été récemment déployé ou rencontre des problèmes, il peut être nécessaire de rassembler des données étendues plus fréquemment. À d’autres moments, il doit être possible de revenir à la capture d’un niveau de base d’informations essentielles pour vérifier que le système fonctionne correctement.

En outre, l’ensemble du processus de surveillance doit être considéré comme une solution dynamique et continue soumise à des ajustements et à des améliorations à la suite de commentaires. Par exemple, vous pouvez commencer par mesurer de nombreux facteurs pour déterminer l’intégrité du système. Au fil du temps, l’analyse peut entraîner un perfectionnement alors que vous ignorez les mesures qui ne sont pas pertinentes, ce qui vous permet de vous consacrer plus précisément aux données dont vous avez besoin tout en réduisant les bruits de fond.

## <a name="sources-of-monitoring-and-diagnostic-data"></a>Sources des données de surveillance et de diagnostic

Les informations utilisées par le processus de surveillance peuvent provenir de plusieurs sources, comme illustré dans la figure 1. Au niveau de l’application, les informations proviennent des journaux de suivi incorporés au code du système. Les développeurs doivent suivre une approche standard pour effectuer le suivi du flux de contrôle via leur code. Par exemple, une entrée dans une méthode peut émettre un message de suivi qui spécifie le nom de la méthode, l’heure actuelle, la valeur de chaque paramètre et toute autre information pertinente. Enregistrer l’heure d’entrée et de sortie peut également s’avérer utile.

Vous devez consigner l’ensemble des exceptions et avertissements, et veiller à conserver une trace complète des exceptions et avertissements imbriqués. Dans l’idéal, vous devez également capturer les informations permettant d’identifier l’utilisateur qui exécute le code, ainsi que les informations de corrélation d’activité (pour suivre les requêtes lorsqu’elles passent par le système). Vous devez également consigner les tentatives d’accès à toutes les ressources telles que les files d’attente de messages, les bases de données, les fichiers et les autres services dépendants. Ces informations peuvent être utilisées à des fins d’audit et de contrôle.

De nombreuses applications utilisent des bibliothèques et des infrastructures pour effectuer des tâches courantes telles que l’accès à un magasin de données ou la communication sur un réseau. Ces infrastructures peuvent être configurables pour renvoyer leurs propres messages de suivi et les informations de diagnostic brutes, comme les taux de transactions ainsi que les réussites et échecs de transmission de données.

> [!NOTE]
> Nombre d’infrastructures modernes publient automatiquement les événements de performances et de suivi. La capture de ces informations dépend simplement de la mise à disposition d’un moyen de les récupérer et de les stocker où elles pourront ensuite être traitées et analysées.

Le système d’exploitation sur lequel s’exécute l’application peut être une source d’informations de niveau inférieur à l’échelle du système, telle que les compteurs de performances qui indiquent les vitesses d’E/S, ainsi que l’utilisation de la mémoire et du processeur. Les erreurs du système d’exploitation (par exemple, l’impossibilité d’ouvrir un fichier correctement) peuvent également être signalées.

Vous devez également tenir compte de l’infrastructure et des composants sous-jacents sur lesquels s’exécute le système. Les machines virtuelles, les réseaux virtuels et les services de stockage peuvent tous être des sources importantes de données de compteurs de performances au niveau de l’infrastructure et d’autres données de diagnostic.

Si votre application utilise d’autres services externes, tels qu’un serveur web ou un système de gestion de base de données, ces services peuvent publier leurs propres informations de suivi, journaux et compteurs de performance. Les exemples incluent des vues de gestion dynamique SQL Server pour le suivi des opérations effectuées sur une base de données SQL Server et les journaux de suivi IIS pour l’enregistrement des requêtes adressées à un serveur web.

Comme les composants d’un système sont modifiés et que de nouvelles versions sont déployées, il est important de pouvoir attribuer des problèmes, des événements et des métriques à chaque version. Ces informations doivent être liées au pipeline de version afin que les problèmes avec une version spécifique d’un composant puissent être suivis rapidement et corrigés.

Des problèmes de sécurité peuvent se produire à tout moment dans le système. Par exemple, un utilisateur peut tenter de se connecter avec un identifiant utilisateur ou un mot de passe non valides. Un utilisateur authentifié peut essayer d’obtenir un accès non autorisé à une ressource ou un utilisateur peut fournir une clé non valide ou obsolète pour accéder à des informations chiffrées. Des informations relatives à la sécurité pour les demandes réussies et échouées doivent toujours être consignées.

La section [Instrumentation d’une application](#instrumenting-an-application) contient davantage de conseils sur les informations que vous devez capturer, mais vous pouvez utiliser de nombreuses stratégies pour rassembler ces informations :

- **Surveillance de l’application/du système**. Cette stratégie utilise des sources internes au sein de l’application, l’infrastructure de l’application, le système d’exploitation et l’infrastructure. Le code de l’application peut générer ses propres données de surveillance à des points notables pendant le cycle de vie d’une requête client. L’application peut inclure des instructions de suivi qui peuvent être activées ou désactivées de manière sélective selon les cas. Il peut être également possible d’injecter des diagnostics dynamiquement à l’aide d’une infrastructure de diagnostics. Ces infrastructures fournissent généralement des plug-in qui peuvent se fixer à divers points d’instrumentation de votre code et capturer les données de suivi à ces points.
  
    En outre, votre code et/ou l’infrastructure sous-jacente peuvent déclencher des événements à des points critiques. Les agents de surveillance configurés pour écouter ces événements peuvent enregistrer les informations sur ces événements.

- **Surveillance des utilisateurs réels**. Cette approche enregistre les interactions entre un utilisateur et l’application, et observe le flux de chaque demande et réponse. Ces informations peuvent avoir un objectif double : elles peuvent servir pour l’utilisation du contrôle par chaque utilisateur et pour déterminer si les utilisateurs reçoivent une qualité de service satisfaisante (par exemple, temps de réponse rapides, faible latence et erreurs minimales). Vous pouvez utiliser les données capturées pour identifier les zones préoccupantes dans lesquelles les échecs se produisent le plus souvent. Vous pouvez également les utiliser pour identifier les éléments dans lesquels le système ralentit, probablement en raison de points d’accès dans l’application ou d’une autre forme de goulot d’étranglement. Si vous implémentez cette approche avec soin, il peut être possible de reconstruire les flux des utilisateurs via l’application à des fins de débogage et de test.
  
  > [!IMPORTANT]
  > Vous devez savoir que les données capturées en surveillant les utilisateurs réels peuvent être très sensibles, car elles peuvent contenir des éléments confidentiels. Si vous enregistrez des données capturées, stockez-les en toute sécurité. Si vous souhaitez utiliser les données à des fins de surveillance des performances ou de débogage, commencez par supprimer toutes les informations d’identification personnelle.

- **Surveillance des utilisateurs synthétiques**. Avec cette approche, vous écrivez votre propre client de test qui simule un utilisateur et exécute une série configurable, mais classique, d’opérations. Vous pouvez suivre les performances du client de test afin de déterminer l’état du système. Vous pouvez également utiliser plusieurs instances du client de test dans le cadre d’une opération de test de charge pour définir la réponse du système en situation de contrainte et le type de sortie de surveillance généré dans ces conditions.
  
  > [!NOTE]
  > Vous pouvez implémenter une surveillance des utilisateurs réels et synthétiques en incluant du code qui suit et chronomètre l’exécution des appels de méthode et des autres parties critiques d’une application.

- **Profilage**. Cette approche est principalement destinée à la surveillance et l’amélioration des performances de l’application. Au lieu d’opérer au niveau fonctionnel de la surveillance des utilisateurs réels et synthétiques, elle capture des informations de niveau inférieur pendant l’exécution de l’application. Vous pouvez implémenter le profilage en utilisant un échantillonnage périodique de l’état d’exécution d’une application (en déterminant la partie du code exécutée par l’application à un moment donné dans le temps). Vous pouvez également utiliser l’instrumentation qui insère des sondes dans le code aux moments importants (par exemple, au début et à la fin d’un appel de méthode) et enregistre les méthodes qui ont été appelées, l’heure des appels ainsi que leur durée. Vous pouvez ensuite analyser ces données pour déterminer les parties de l’application qui peuvent causer des problèmes de performances.

- **Surveillance des points de terminaison**. Cette technique utilise un ou plusieurs points de terminaison de diagnostics exposés par l’application spécifiquement pour permettre la surveillance. Un point de terminaison fournit un chemin d’accès au code de l’application et peut renvoyer des informations sur l’intégrité du système. Des points de terminaison différents peuvent se concentrer sur divers aspects de la fonctionnalité. Vous pouvez écrire votre propre client de diagnostic qui envoie des demandes périodiques à ces points de terminaison et assimiler les réponses. Pour plus d’informations, consultez [Modèle Supervision de point de terminaison d’intégrité](../patterns/health-endpoint-monitoring.md).

Pour une couverture maximale, vous devez utiliser une combinaison de ces techniques.

## <a name="instrumenting-an-application"></a>Instrumentation d’une application

L’instrumentation est un élément essentiel du processus de surveillance. Vous pouvez prendre des décisions judicieuses sur les performances et l’intégrité d’un système uniquement si vous capturez tout d’abord les données qui vous permettent de prendre ces décisions. Les informations que vous collectez à l’aide de l’instrumentation doivent être suffisantes pour vous permettre d’évaluer les performances, de diagnostiquer les problèmes et de prendre des décisions sans avoir besoin de vous connecter à un serveur de production distant pour effectuer manuellement le suivi (et le débogage). Les données d’instrumentation comprennent généralement des métriques et des informations écrites dans les journaux de suivi.

Le contenu d’un journal de suivi peut être le résultat des données textuelles écrites par l’application ou des données binaires qui sont créées à la suite d’un événement de suivi (si l’application utilise Suivi d’événements pour Windows [ETW]). Il peut également être généré à partir des journaux système qui enregistrent les événements provenant des parties de l’infrastructure, un serveur web par exemple. Les messages textuels du journal sont souvent conçus pour être lisibles, mais ils doivent également être écrits dans un format facilement analysable par un système automatisé.

Vous devez également classer les journaux. N’écrivez pas toutes les données de suivi dans un seul journal, mais utilisez des journaux distincts pour enregistrer la sortie de suivi des différents aspects opérationnels du système. Vous pouvez alors filtrer rapidement les messages du journal par lecture du journal approprié au lieu de traiter un seul fichier long. Ne jamais consigner des informations ayant des exigences de sécurité différentes (telles que les informations d’audit et les données de débogage) dans le même journal.

> [!NOTE]
> Un journal peut être implémenté en tant que fichier dans le système de fichiers. Il peut également être conservé dans un autre format comme un objet blob dans le stockage d’objets blob. Les informations du journal peuvent également être conservées dans un stockage structuré, comme des lignes dans un tableau.

Généralement, les métriques sont une mesure ou un nombre d’aspects ou de ressources dans le système à un moment donné avec une ou plusieurs balises ou dimensions associées (parfois appelées *échantillon*). Une seule instance d’une métrique n’est généralement pas utile isolément. Les métriques doivent plutôt être capturées au fil du temps. La question clé à examiner concerne le choix des métriques à enregistrer et la fréquence d’enregistrement. Trop souvent, la génération des données pour les métriques impose une charge supplémentaire importante sur le système, alors que la capture des métriques vous fera rarement manquer les circonstances qui entraînent un événement important. Cet examen varie d’une métrique à l’autre. Par exemple, l’utilisation du processeur sur un serveur peut varier considérablement d’une seconde à l’autre, mais une utilisation élevée ne devient un problème que lorsqu’elle se prolonge sur plusieurs minutes.

### <a name="information-for-correlating-data"></a>Informations pour la corrélation des données

Vous pouvez facilement surveiller les compteurs de performances individuels au niveau du système, capturer les métriques correspondant aux ressources et obtenir des informations de traçage d’application à partir de différents fichiers journaux. Néanmoins, certaines formes de surveillance nécessitent la phase d’analyse et de diagnostic dans le pipeline de surveillance pour mettre en corrélation les données récupérées à partir de plusieurs sources. Ces données peuvent prendre plusieurs formes dans les données brutes, et le processus d’analyse doit être fourni avec des données d’instrumentation suffisantes pour pouvoir mapper ces différentes formes. Par exemple, au niveau de l’infrastructure d’application, une tâche peut être identifiée par un ID de thread. Dans une application, la même tâche peut être associée à l’identifiant de l’utilisateur qui l’effectue.

D’autre part, un mappage 1:1 entre les threads et les requêtes utilisateur est peu probable, car les opérations asynchrones peuvent réutiliser les mêmes threads pour effectuer des opérations pour le compte de plusieurs utilisateurs. Pour compliquer davantage les choses, une seule requête peut être gérée par plusieurs threads durant le déroulement de l’exécution dans le système. Si possible, associez chaque requête à un ID d’activité unique qui est propagé dans le système dans le contexte de la requête. (La technique utilisée pour générer et inclure des ID d’activité dans les informations de traçage dépend de la technologie utilisée pour la capture des données de suivi).

Toutes les données de surveillance doivent être horodatées de la même façon. Pour des raisons de cohérence, il est nécessaire d’enregistrer toutes les dates et heures en utilisant le temps universel coordonné. Cela vous permettra de suivre plus facilement des séquences d’événements.

> [!NOTE]
> Il se peut que les ordinateurs qui fonctionnent dans différents fuseaux horaires et réseaux ne soient pas synchronisés. Ne dépendez pas de l’utilisation des horodatages seuls pour la mise en corrélation des données d’instrumentation qui s’étendent sur plusieurs ordinateurs.

### <a name="information-to-include-in-the-instrumentation-data"></a>Informations à inclure dans les données d’instrumentation

Lorsque vous décidez des données d’instrumentation à collecter, tenez compte des points suivants :

- Vérifiez que les informations capturées par les événements de suivi sont lisibles par l’homme et la machine. Adoptez des schémas clairement définis pour ces informations afin de faciliter le traitement automatisé des données des journaux entre les systèmes, et d’assurer la cohérence pour le personnel des opérations et technique qui lit les journaux. Inclure des informations environnementales, telles que l’environnement de déploiement, l’ordinateur sur lequel le processus s’exécute, les détails du processus et la pile des appels.

- Activez le profilage uniquement en cas de nécessité, car il peut imposer une surcharge significative sur le système. Le profilage effectué à l’aide de l’instrumentation enregistre un événement (tel qu’un appel de méthode) chaque fois qu’il se produit, tandis que l’échantillonnage n’enregistre que les événements sélectionnés. La sélection peut être basée sur le temps (une fois toutes les *n* secondes) ou sur la fréquence (une fois toutes les *n* requêtes). Si des événements se produisent très fréquemment, le profilage par instrumentation peut occasionner une charge trop lourde et affecter les performances globales. Dans ce cas, l’approche par échantillonnage peut être préférable. Toutefois, si les événements se produisent peu fréquemment, l’échantillonnage risque de les manquer. Le cas échéant, l’instrumentation peut constituer la meilleure approche.

- Fournir un contexte suffisant pour permettre à un développeur ou un administrateur de déterminer la source de chaque demande. Cela peut inclure une forme d’ID d’activité qui identifie une instance spécifique d’une requête. Cela peut également contenir des informations qui permettent de mettre en corrélation cette activité avec le travail de calcul effectué et les ressources utilisées. Notez que ce travail peut franchir les limites de processus et d’ordinateur. Pour le contrôle, le contexte doit également inclure (directement ou indirectement via d’autres informations mises en corrélation) une référence au client qui a provoqué la formulation de la requête. Ce contexte fournit de précieuses informations sur l’état de l’application au moment de la capture des données de surveillance.

- Enregistrer toutes les demandes, ainsi que les emplacements ou les régions à partir desquels ces demandes sont effectuées. Ces informations peuvent aider à déterminer s’il existe des points d’accès propres à l’emplacement. Elles peuvent également être utiles pour déterminer s’il convient de repartitionner une application ou les données qu’elle utilise.

- Enregistrer et capturer les détails des exceptions avec soin. Les informations critiques de débogage sont souvent perdues à la suite d’une mauvaise gestion des exceptions. Capturez les détails complets des exceptions levées par l’application, notamment les exceptions internes et autres informations de contexte. Incluez si possible la pile des appels.

- Soyez cohérent dans les données que les différents éléments de votre application capturent, car cela peut vous permettre d’analyser les événements et de les mettre en corrélation avec les requêtes utilisateur. Préférez recourir à un package de journalisation complet et configurable pour collecter des informations plutôt que de dépendre de développeurs adoptant la même approche lorsqu’ils implémentent différentes parties du système. Collecter les données à partir des compteurs de performance clés, tels que le volume d’E/S effectué, l’utilisation du réseau, le nombre de demandes, l’utilisation de la mémoire et du processeur. Certains services d’infrastructure peuvent fournir leurs propres compteurs de performances spécifiques, comme le nombre de connexions à une base de données, la vitesse à laquelle les transactions sont effectuées et le nombre de transactions qui réussissent ou échouent. Les applications peuvent également définir leurs propres compteurs de performance spécifiques.

- Consignez tous les appels effectués vers des services externes, comme les systèmes de base de données, les services web ou d’autres services au niveau du système qui font partie de l’infrastructure. Enregistrer les informations sur le temps nécessaire pour réaliser chaque appel et la réussite ou l’échec de l’appel. Si possible, capturer les informations sur toutes les nouvelles tentatives et les échecs pour toutes les erreurs temporaires se produisant.

### <a name="ensuring-compatibility-with-telemetry-systems"></a>Garantir la compatibilité avec les systèmes de télémétrie

Dans de nombreux cas, les informations générées par l’instrumentation le sont sous la forme d’une série d’événements, puis transmises à un système de télémétrie distinct pour le traitement et l’analyse. Un système de télémétrie est généralement indépendant de toute application ou technologie spécifique, mais s’attend à ce que les informations respectent un format spécifique, généralement défini par un schéma. Le schéma spécifie en fait un contrat qui définit les champs et les types de données que peut recevoir le système de télémétrie. Le schéma doit être généralisé pour autoriser les données reçues à partir de diverses plates-formes et périphériques.

Un schéma commun doit inclure les champs qui sont communs à tous les événements d’instrumentation, tels que le nom de l’événement, l’heure de l’événement, l’adresse IP de l’expéditeur et les détails nécessaires pour la mise en corrélation avec d’autres événements (par exemple un identifiant utilisateur, un ID de périphérique et un ID de l’application). N’oubliez pas qu’un nombre quelconque de périphériques peut déclencher des événements ; le schéma ne doit donc dépendre du type de périphérique. En outre, différents périphériques peuvent déclencher des événements pour une même application. L’application peut prendre en charge l’itinérance ou toute autre forme de distribution inter-périphériques.

Le schéma peut également inclure des champs de domaine qui sont pertinents pour un scénario particulier commun entre différentes applications. Il peut s’agir d’informations sur les exceptions, sur les événements de début et de fin d’application et sur la réussite et/ou l’échec des appels d’API de service web. Toutes les applications qui utilisent le même ensemble de champs de domaine doivent émettre le même jeu d’événements, permettant de créer un ensemble de rapports et d’analyses communs.

Enfin, un schéma peut contenir des champs personnalisés pour capturer les détails des événements propres à l’application.

### <a name="best-practices-for-instrumenting-applications"></a>Meilleures pratiques pour l’instrumentation d’applications

La liste suivante récapitule les meilleures pratiques pour l’instrumentation d’une application distribuée fonctionnelle dans le cloud.

- Faciliter la lecture et l’analyse des journaux. Utiliser une journalisation structurée dès que possible. Être concis et descriptif dans les messages de journal.

- Dans tous les journaux, identifier la source et fournir le contexte et les informations de minutage à mesure que chaque enregistrement est écrit.

- Utiliser les mêmes fuseau horaire et format pour tous les horodatages. Cela vous aidera à mettre en corrélation des événements pour les opérations qui s’étendent sur le matériel et les services fonctionnant dans différentes régions géographiques.

- Classer les journaux et écrire des messages dans le fichier journal approprié.

- Ne pas divulguer d’informations sensibles sur le système ou les informations personnelles des utilisateurs. Nettoyer ces informations avant qu’elles ne soient consignées, mais s’assurer que les détails pertinents sont conservés. Par exemple, supprimer l’ID et le mot de passe des chaînes de connexion des base de données, mais écrire les informations restantes dans le journal afin qu’un analyste puisse déterminer que le système accède correctement à la base de données. Enregistrer toutes les exceptions critiques, mais permettre à l’administrateur d’activer et de désactiver la journalisation pour les niveaux inférieurs d’exceptions et d’avertissements. En outre, capturer et enregistrer toutes les informations de logique de nouvelle tentative. Ces données peuvent être utiles pour la surveillance de l’intégrité temporaire du système.

- Suivre les appels hors processus, comme les requêtes vers des services web externes ou des bases de données.

- Ne pas associer des messages de journal ayant des exigences de sécurité différentes dans un même fichier journal. Par exemple, ne pas écrire d’informations de débogage et d’audit dans le même journal.

- À l’exception des événements d’audit, s’assurer que tous les appels de journalisation sont des opérations de type fire-and-forget qui ne bloquent pas la progression des opérations de l’entreprise. Les événements d’audit sont exceptionnels, car ils sont essentiels pour l’entreprise et peuvent être classés comme un élément fondamental des opérations de l’entreprise.

- Vérifier que la journalisation est extensible et qu’elle n’a aucune dépendance directe vis-à-vis d’une cible concrète. Par exemple, plutôt que d’écrire des informations à l’aide de *System.Diagnostics.Trace*, définir une interface abstraite (telle que *ILogger*) qui expose des méthodes de journalisation et qui peut être implémentée par tout moyen approprié.

- S’assurer que la journalisation est valide et ne déclenche pas d’erreurs en cascade. La journalisation ne doit lancer aucune exception.

- Traiter l’instrumentation comme un processus itératif continu et examiner régulièrement les journaux, pas uniquement en cas de problème.

## <a name="collecting-and-storing-data"></a>Collecte et stockage des données

La phase de collecte du processus de surveillance vise à récupérer les informations générées par l’instrumentation, à formater ces données afin de les rendre plus faciles à utiliser lors de la phase d’analyse/de diagnostic et à enregistrer les données transformées dans un stockage fiable. Les données d’instrumentation que vous collectez à partir des différentes parties d’un système distribué peuvent être conservées dans divers emplacements et avec des formats différents. Par exemple, votre code d’application peut générer des fichiers journaux de suivi et générer des données de journal des événements d’application, tandis que les compteurs de performances qui surveillent les principaux aspects de l’infrastructure utilisés par votre application peuvent être capturés via d’autres technologies. Les composants et services tiers utilisés par votre application peuvent fournir des informations d’instrumentation dans différents formats, à l’aide de fichiers de trace distincts, de stockage d’objets blob, voire d’un magasin de données personnalisé.

La collecte de données est souvent effectuée via un service de collecte qui peut s’exécuter de manière autonome à partir de l’application qui génère les données d’instrumentation. La figure 2 illustre un exemple de cette architecture en mettant en évidence le sous-système de collecte des données d’instrumentation.

![Exemple de collecte des données d’instrumentation](./images/monitoring/TelemetryService.png)

*Figure 2 : Collecte des données d’instrumentation.*

Notez qu’il s’agit d’une vue simplifiée. Le service de collecte n’est pas nécessairement un processus unique et peut comporter plusieurs parties constituantes s’exécutant sur différents ordinateurs, comme le décrivent les sections suivantes. En outre, si l’analyse de certaines données de télémétrie doit être effectuée rapidement (analyse à chaud, comme décrit dans la section [Prise en charge de l’analyse à chaud, modérée et à froid](#supporting-hot-warm-and-cold-analysis) plus loin dans ce document), des composants locaux fonctionnant en dehors du service de collecte peuvent effectuer les tâches d’analyse immédiatement. La figure 2 illustre cette situation pour les événements sélectionnés. Les résultats du traitement analytique peuvent être envoyés directement au sous-système de visualisation et d’alerte. Les données soumises aux analyses modérée ou à froid sont stockées pendant qu’elles attendent le traitement.

Pour les applications et services Azure, les diagnostics Azure fournissent une solution possible de capture des données. Les diagnostics Azure collectent des données à partir des sources suivantes pour chaque nœud de calcul, les agrègent, puis les chargent vers Azure Storage :

- Journaux IIS
- Journaux d’échecs de demande IIS
- Journaux des événements Windows
- Compteurs de performances
- Vidages sur incident
- Journaux d’infrastructure de diagnostics Azure  
- Journaux d'erreurs personnalisés
- .NET EventSource
- Suivi d’événements pour Windows basé sur les manifestes

Pour plus d’informations, consultez l’article [Azure : Telemetry Basics and Troubleshooting (Azure : Concepts de base de la télémétrie et résolution des problèmes)](https://social.technet.microsoft.com/wiki/contents/articles/18146.windows-azure-telemetry-basics-and-troubleshooting.aspx).

### <a name="strategies-for-collecting-instrumentation-data"></a>Stratégies de collecte des données d’instrumentation

Considérant la nature évolutive du cloud et pour éviter la nécessité de récupérer manuellement les données de télémétrie de chaque nœud du système, vous devez faire en sorte que les données soient transférées vers un emplacement central et consolidé. Dans un système qui s’étend sur plusieurs centres de données, il peut être utile de commencer par collecter, consolider et stocker des données sur une base régionale, puis d’agréger les données régionales dans un système central unique.

Pour optimiser l’utilisation de la bande passante, vous pouvez choisir de transférer les données moins urgentes en blocs, en tant que lots. Toutefois, les données de ne doivent pas être retardées indéfiniment, surtout si elles contiennent des informations soumises à une contrainte de temps.

#### <a name="pulling-and-pushing-instrumentation-data"></a>*Extraction et transmission des données d’instrumentation*

Le sous-système de collecte des données d’instrumentation peut récupérer activement les données d’instrumentation à partir de divers journaux et d’autres sources pour chaque instance de l’application ( *modèle d’extraction*). Il peut également servir de récepteur passif qui attend que les données soient envoyées à partir des composants qui constituent chaque instance de l’application ( *modèle d’émission*).

Une approche de l’implémentation du modèle d’extraction consiste à utiliser des agents de surveillance s’exécutant localement avec chaque instance de l’application. Un agent de surveillance est un processus distinct qui récupère (extrait) régulièrement les données de télémétrie collectées au niveau du nœud local et écrit ces informations directement dans le stockage centralisé qui est partagé par toutes les instances de l’application. Il s’agit du mécanisme implémenté par les diagnostics Azure. Chaque instance d’un rôle web ou d’un rôle de travail Azure peut être configurée pour capturer le diagnostic et d’autres informations de traçage stockées localement. L’agent de surveillance qui s’exécute avec chaque instance copie les données spécifiées dans Azure Storage. L’article [Activation des diagnostics Azure dans Azure Cloud Services](/azure/cloud-services/cloud-services-dotnet-diagnostics) fournit plus d’informations sur ce processus. Certains éléments comme les journaux IIS, les vidages sur incident et les journaux d’erreurs personnalisés sont écrits dans le stockage d’objets blob. Les données du journal des événements Windows, des événements ETW et des compteurs de performances sont enregistrées dans un stockage de table. La figure 3 illustre ce mécanisme.

![Illustration de l’utilisation d’un agent de surveillance pour extraire des informations et les écrire dans un stockage partagé](./images/monitoring/PullModel.png)

*Figure 3 : Utilisation d’un agent de supervision pour extraire des informations et les écrire dans un stockage partagé.*

> [!NOTE]
> L’utilisation d’un agent de surveillance convient idéalement à la capture des données d’instrumentation qui sont naturellement extraites d’une source de données. Les informations des vues de gestion dynamique SQL Server ou la longueur d’une file d’attente Azure Service Bus en sont un bon exemple.

Il est possible d’utiliser l’approche qui vient d’être décrite pour stocker les données de télémétrie pour une application à petite échelle qui s’exécute sur un nombre limité de nœuds dans un emplacement unique. Toutefois, une application cloud globale, hautement évolutive et complexe peut générer d’énormes volumes de données provenant de centaines de rôles web et de rôles de travail, de partitionnements de base de données et d’autres services. Ce flux de données peut facilement surcharger la bande passante d’E/S disponible avec un emplacement central unique. Par conséquent, votre solution de télémétrie doit être évolutive afin de l’empêcher de se comporter comme un goulot d’étranglement lorsque le système se développe. Dans l’idéal, votre solution doit incorporer un degré de redondance pour réduire les risques de perdre des informations de surveillance importantes (telles que les données d’audit ou de facturation) si une partie du système tombe en panne.

Pour résoudre ces problèmes, vous pouvez implémenter un mécanisme de mise en file d’attente, comme indiqué figure 4. Dans cette architecture, l’agent de surveillance local (s’il peut être configuré de manière appropriée) ou un service de collecte des données personnalisé (sinon) envoie les données vers une file d'attente. Un processus distinct s’exécutant de façon asynchrone (service d’écriture dans le stockage de la figure 4) prend les données de cette file d’attente et les écrit dans le stockage partagé. Une file d’attente de message est adaptée à ce scénario, car elle fournit la sémantique « au moins une fois » assurant qu’une fois publiées, les données mises en file d’attente ne sont pas perdues. Vous pouvez implémenter le service d’écriture dans le stockage en utilisant un rôle de travail distinct.

![Illustration de l’utilisation d’une file d’attente pour mettre en mémoire tampon les données d’instrumentation](./images/monitoring/BufferedQueue.png)

*Figure 4 : Utilisation d’une file d’attente pour mettre en mémoire tampon les données d’instrumentation.*

Le service de collecte des données local peut ajouter des données à une file d’attente dès leur réception. La file d’attente agit comme une mémoire tampon et le service d’écriture dans le stockage peut récupérer et écrire les données à son propre rythme. Par défaut, une file d’attente fonctionne sur la base du premier entré, premier sorti. Cependant, vous pouvez hiérarchiser les messages pour accélérer leur avancée dans la file d’attente s’ils contiennent des données qui doivent être traitées plus rapidement. Pour plus d’informations, consultez [Modèle de file d’attente avec un ordre de priorité](../patterns/priority-queue.md). Vous pouvez également utiliser différents canaux (par exemple, les rubriques Service Bus) pour diriger les données vers différentes destinations en fonction de la forme de traitement analytique nécessaire.

À des fins d’évolutivité, vous pouvez exécuter plusieurs instances du service d’écriture dans le stockage. Si le volume d’événements est élevé, vous pouvez utiliser un concentrateur d’événements pour distribuer les données vers des ressources de calcul différentes pour le traitement et le stockage.

#### <a name="consolidating-instrumentation-data"></a>*Consolidation des données d’instrumentation*

Les données d’instrumentation récupérées par le service de collecte des données à partir d’une seule instance d’une application fournissent une vue localisée de l’intégrité et des performances de cette instance. Pour évaluer l’intégrité globale du système, il est nécessaire de consolider certains aspects des données dans les vues locales. Vous pouvez effectuer cette consolidation une fois que les données ont été stockées, mais dans certains cas, vous pouvez également la réaliser lors de la collecte des données. Au lieu d’être écrites directement dans le stockage partagé, les données d’instrumentation peuvent passer par un service de consolidation de données distinct qui combine les données et agit comme un processus de filtrage et de nettoyage. Par exemple, les données d’instrumentation qui incluent les mêmes informations de corrélation, par exemple un ID d’activité, peuvent être fusionnées. (Il est possible qu’un utilisateur démarre une opération sur un nœud, puis soit transféré vers un autre nœud en cas de défaillance du premier ou selon la configuration de l’équilibrage de charge). Ce processus peut également détecter et supprimer les données dupliquées (toujours une possibilité si le service de télémétrie utilise des files d’attente de messages pour envoyer des données d’instrumentation vers le stockage). La figure 5 illustre un exemple de cette structure.

![Exemple d’utilisation d’un service pour consolider les données d’instrumentation](./images/monitoring/Consolidation.png)

*Figure 5 : Utilisation d’un service distinct pour consolider et nettoyer les données d’instrumentation.*

### <a name="storing-instrumentation-data"></a>Stockage des données d’instrumentation

Les discussions précédentes ont représenté une vue assez simple du mode de stockage des données d’instrumentation. En réalité, il peut être judicieux de stocker les différents types d’informations à l’aide des technologies les plus appropriées au mode d’utilisation de chacun des types.

Par exemple, le stockage de table et le stockage d’objets blob Azure présentent certaines similitudes dans leur mode d’accès. Néanmoins, elles connaissent des restrictions dans les opérations que vous pouvez effectuer en les utilisant, et la granularité des données qu’elles conservent est assez différente. Si vous devez effectuer plusieurs opérations analytiques ou si vous avez besoin de fonctionnalités de recherche en texte intégral sur les données, il peut être plus pertinent d’utiliser le stockage de données qui fournit des fonctionnalités optimisées pour des types spécifiques de requêtes et d’accès aux données. Par exemple : 

- Les données des compteurs de performances peuvent être stockées dans une base de données SQL pour activer l’analyse ad hoc.
- Azure Cosmos DB peut se révéler mieux adapté au stockage des journaux de suivi.
- Les informations sur la sécurité peuvent être écrites dans HDFS.
- Les informations nécessitant une recherche en texte intégral peuvent être stockées via Elasticsearch (qui peut également accélérer les recherches en utilisant une indexation enrichie).

Vous pouvez implémenter un service supplémentaire qui extrait régulièrement des données à partir du stockage partagé, partitionne et filtre les données en fonction de leur objectif, puis les écrit dans un ensemble approprié de magasins de données comme illustré à la figure 6. Une autre approche consiste à inclure cette fonctionnalité dans le processus de consolidation et de nettoyage, et à écrire les données directement dans ces magasins à mesure qu’elles sont récupérées plutôt que de les enregistrer dans une zone de stockage partagée intermédiaire. Chaque approche a des avantages et des inconvénients. L’implémentation d’un service distinct de partitionnement réduit la charge sur le service de consolidation et de nettoyage et permet au moins que certaines données partitionnées soient régénérées si nécessaire (en fonction de la quantité de données conservées dans le stockage partagé). Toutefois, ce service consomme des ressources supplémentaires. En outre, il peut y avoir un délai entre la réception des données d’instrumentation à partir de chaque instance d’application et la conversion de ces données en informations utilisables.

![Partitionnement et stockage des données](./images/monitoring/DataStorage.png)

*Figure 6 : Partitionnement des données en fonction des exigences d’analyse et de stockage.*

Les mêmes données d’instrumentation peuvent servir à plus d’une fin. Par exemple, les compteurs de performances permettent de fournir une vue historique des performances du système au fil du temps. Ces informations peuvent être combinées avec d’autres données d’utilisation pour générer les informations de facturation client. Dans ces situations, les mêmes données peuvent être envoyées à plusieurs destinations, par exemple une base de données de documents qui peut agir comme un magasin à long terme pour la conservation des informations de facturation et un magasin multidimensionnel pour la gestion des analyses des performances complexes.

Vous devez également tenir compte du degré d’urgence de ces données. Les données qui fournissent des informations pour les alertes doivent être accessibles rapidement et doivent donc être conservées dans le stockage de données rapide, et indexées ou structurées pour optimiser les requêtes exécutées par le système d’alerte. Dans certains cas, il peut être nécessaire que le service de télémétrie qui collecte les données sur chaque nœud formate et enregistre les données localement afin qu’une instance locale du système d’alerte puisse rapidement vous informer de tout problème. Les mêmes données peuvent être transmises au service d’écriture dans le stockage indiqué dans les diagrammes précédents, et stockées de façon centralisée si elles doivent également servir à d’autres fins.

Les informations utilisées pour une analyse plus réfléchie, le reporting et l’identification des tendances historiques sont moins urgentes et peuvent être stockées d’une manière qui prend en charge l’exploration de données et les requêtes ad hoc. Pour plus d’informations, consultez la section [Prise en charge de l’analyse à chaud, modérée et à froid](#supporting-hot-warm-and-cold-analysis) plus loin dans ce document.

#### <a name="log-rotation-and-data-retention"></a>*Rotation des journaux et conservation des données*

L’instrumentation peut générer des volumes considérables de données. Ces données peuvent être conservées dans des emplacements différents, des fichiers journaux bruts, fichiers de suivi et autres informations capturées sur chaque nœud à la vue consolidée, nettoyée et partitionnée de ces données conservées dans le stockage partagé. Dans certains cas, une fois que les données ont été traitées et transférées, les données sources brutes d’origine peuvent être supprimées de chaque nœud. Dans d’autres cas, il peut être nécessaire ou simplement utile d’enregistrer les informations brutes. Par exemple, les données générées à des fins de débogage peuvent être laissées disponibles dans leur forme brute, mais peuvent être ensuite ignorées rapidement une fois que les bogues ont été corrigés.

Les données de performances ont souvent une durée de vie plus longue afin de pouvoir être utilisées à des fins d’identification des tendances de performances et de planification de la capacité. La vue consolidée de ces données est en général conservée en ligne pour une période limitée afin de permettre un accès rapide. Après quoi, elle peut être archivée ou ignorée. Les données collectées pour le contrôle et la facturation des clients peuvent devoir être enregistrées indéfiniment. En outre, des exigences réglementaires peuvent imposer l’archivage et l’enregistrement des informations collectées à des fins d’audit et de sécurité. Ces données sont également sensibles et peuvent devoir être chiffrées ou protégées d’une quelconque façon pour empêcher toute falsification. Vous ne devez jamais enregistrer les mots de passe des utilisateurs ou d’autres informations qui peuvent être utilisées pour valider l’usurpation d’identité. Ces informations doivent être supprimées des données avant leur stockage.

#### <a name="down-sampling"></a>*Sous-échantillonnage*

Il est souvent utile de stocker les données historiques afin de pouvoir identifier les tendances à long terme. Plutôt que d’enregistrer les données anciennes dans leur intégralité, il peut être possible de sous-échantillonner les données pour réduire leur résolution et les coûts de stockage. Par exemple, plutôt que d’enregistrer des indicateurs de performance minute par minute, vous pouvez consolider les données antérieures à plus d’un mois pour former une vue heure par heure.

### <a name="best-practices-for-collecting-and-storing-logging-information"></a>Meilleures pratiques pour collecter et stocker des informations de journalisation

La liste suivante récapitule les meilleures pratiques pour la capture et le stockage des informations de journalisation :

- L’agent de surveillance ou le service de collecte des données doit s’exécuter comme un service hors processus et doit être simple à déployer.

- Toutes les sorties de l’agent de surveillance ou du service de collecte des données doivent avoir un format non spécifique indépendant de l’ordinateur, du système d’exploitation ou du protocole réseau. Par exemple, émettre les informations dans un format autodescriptif tels que JSON, MessagePack ou Protobuf plutôt que ETL/ETW. Utiliser un format standard permet au système de construire des pipelines de traitement ; les composants qui lisent, transforment et envoient des données au format convenu peuvent être facilement intégrés.

- Le processus de collecte de données et de surveillance doit être effectué en mode de prévention de défaillance et ne doit pas déclencher de conditions d’erreur en cascade.

- En cas d’échec passager de l’envoi des informations à un récepteur de données, l’agent de surveillance ou le service de collecte des données doit être préparé pour réorganiser les données de télémétrie afin que les informations les plus récentes soient envoyées en premier. (Le service de collecte des données ou l’agent de surveillance peut choisir de supprimer les anciennes données ou de les enregistrer localement et de les transmettre plus tard pour les mettre à jour à son entière discrétion.)

## <a name="analyzing-data-and-diagnosing-issues"></a>Analyse des données et diagnostic des problèmes

Une partie importante du processus de surveillance et de diagnostic consiste à analyser les données collectées pour obtenir une image de l’état général du système. Vous devez avoir défini vos propres indicateurs KPI et les métriques de performance, et il est important de comprendre comment vous pouvez structurer les données qui ont été collectées pour répondre à vos besoins d’analyse. Il est également important de comprendre comment les données capturées dans les différents fichiers de métriques et fichiers journaux sont mises en corrélation, car ces informations peuvent être importantes pour suivre une séquence d’événements et diagnostiquer les problèmes qui surviennent.

Comme décrit dans la section [Consolidation des données d’instrumentation](#consolidating-instrumentation-data), les données correspondant à chaque partie du système sont généralement capturées localement, mais en général, elles doivent être combinées avec des données générées sur d’autres sites participant au système. Ces informations nécessitent d’être corrélées avec attention pour s’assurer que les données sont combinées avec précision. Par exemple, les données d’utilisation pour une opération peuvent s’étendre sur un nœud qui héberge un site web auquel se connecte un utilisateur, un nœud qui exécute un service distinct accessible dans le cadre de cette opération et un stockage de données conservé sur un autre nœud. Ces informations doivent être liées entre elles pour fournir une vue d’ensemble de l’utilisation des ressources et du traitement pour l’opération. Un prétraitement et un filtrage des données peuvent se produire sur le nœud sur lequel les données sont capturées, tandis que l’agrégation et le formatage sont davantage susceptibles de se produire sur un nœud central.

### <a name="supporting-hot-warm-and-cold-analysis"></a>Prise en charge de l’analyse à chaud, modérée et à froid

L’analyse et le reformatage des données à des fins de visualisation, de reporting et d’alerte peuvent être un processus complexe qui utilise son propre ensemble de ressources. Certaines formes de surveillance sont prioritaires et nécessitent que l’analyse immédiate des données soit efficace. C’est ce qu’on appelle *l’analyse à chaud*. Les exemples incluent les analyses nécessaires pour les alertes et certains aspects de la surveillance de la sécurité (comme la détection d’une attaque sur le système). Les données requises pour ces besoins doivent être rapidement disponibles et structurées en vue d’un traitement efficace. Dans certains cas, il peut être nécessaire de déplacer le traitement de l’analyse vers les nœuds individuels où les données sont conservées.

D’autres formes d’analyse sont moins prioritaires et peuvent nécessiter des opérations de calcul et d’agrégation une fois les données brutes reçues. Il s’agit *d’analyse modérée*. L’analyse des performances fait souvent partie de cette catégorie. Dans ce cas, un événement de performances unique et isolé a peu de chances d’être statistiquement significatif. (Il peut être provoqué par une erreur ou un pic soudains.) Les données d’une série d’événements doivent fournir une image plus fiable des performances du système.

L’analyse modérée permet également de diagnostiquer les problèmes d’intégrité. Un événement d’intégrité est généralement traité via une analyse à chaud et peut déclencher une alerte immédiatement. Un opérateur doit être en mesure d’explorer les raisons de l’événement d’intégrité en examinant les données du chemin d’accès encore chaud. Ces données doivent contenir des informations sur les événements conduisant au problème qui a provoqué l’événement d’intégrité.

Certains types de surveillance génèrent des données à plus long terme. Cette analyse peut être effectuée à une date ultérieure, éventuellement selon une planification prédéfinie. Dans certains cas, l’analyse devra peut-être effectuer un filtrage complexe d’importants volumes de données capturées sur une période donnée. Il s’agit d’ *analyse à froid*. La condition principale est de stocker en toute sécurité les données une fois qu’elles ont été capturées. Par exemple, la surveillance et l’audit de l’utilisation nécessitent une image précise de l’état du système à des moments réguliers dans le temps, mais ces informations d’état n’ont pas besoin d’être disponibles immédiatement après leur collecte pour être soumises au traitement.

Un opérateur peut également utiliser l’analyse à froid pour fournir les données pour l’analyse prédictive de l’intégrité. Il peut collecter les informations historiques sur une période spécifiée et les utiliser conjointement avec les données d’intégrité actuelles (récupérées à partir du chemin réactif) pour identifier les tendances qui peuvent provoquer rapidement des problèmes d’intégrité. Dans ces cas, il peut être nécessaire de déclencher une alerte afin que des mesures correctives puissent être prises.

### <a name="correlating-data"></a>Corrélation des données

Les données capturées par l’instrumentation peuvent fournir un instantané de l’état du système, mais l’objectif de l’analyse est de rendre ces données exploitables. Par exemple : 

- Quelle est la cause d’un chargement d’E/S intense au niveau du système à un moment spécifique ?
- Est-il le résultat d’un grand nombre d’opérations de base de données ?
- A-t-il une influence sur les temps de réponse de la base de données, le nombre de transactions par seconde et les temps de réponse de l’application au même point de jonction ?

Dans ce cas, une mesure corrective pouvant réduire la charge peut consister à partitionner les données sur plusieurs serveurs. Par ailleurs, des exceptions peuvent se produire suite à une erreur à un niveau quelconque du système. Une exception se produisant à un niveau déclenche souvent une autre erreur au niveau supérieur.

Pour ces raisons, vous devez être en mesure d’établir une corrélation entre les différents types de données de surveillance à chaque niveau afin de produire une vue d’ensemble de l’état du système et des applications qui s’exécutent sur celui-ci. Vous pouvez ensuite utiliser ces informations pour décider si le système fonctionne de manière satisfaisante ou non, et déterminer ce qui peut être fait pour améliorer la qualité du système.

Comme décrit dans la section [Informations pour la corrélation des données](#information-for-correlating-data), vous devez vous assurer que les données d’instrumentation brutes incluent un contexte et des informations de l’ID d’activité suffisants pour prendre en charge les agrégations requises pour la corrélation des événements. En outre, ces données peuvent être conservées dans différents formats, et il peut être nécessaire d’analyser ces informations pour les convertir dans un format normalisé à des fins d’analyse.

### <a name="troubleshooting-and-diagnosing-issues"></a>Résolution et diagnostic des problèmes

Le diagnostic requiert la possibilité de déterminer la cause des erreurs ou du comportement inattendu, notamment la possibilité de réaliser une analyse de la cause première. Les informations requises sont généralement les suivantes :

- Des informations détaillées provenant des journaux d’événements et des traces pour l’ensemble du système ou pour un sous-système spécifié pendant un laps de temps spécifié.
- Des arborescences des appels de procédure résultant des exceptions et des erreurs de tout niveau spécifié qui se produisent au sein du système ou d’un sous-système spécifié pendant une période spécifiée.
- Des vidages sur incident pour tous les processus ayant échoué n’importe où dans le système ou pour un sous-système spécifié pendant un laps de temps spécifié.
- Des journaux d’activité enregistrant les opérations effectuées par tous les utilisateurs ou par les utilisateurs sélectionnés pendant une période spécifiée.

L’analyse des données pour résoudre les problèmes requiert souvent une connaissance technique approfondie de l’architecture du système et des différents composants qui constituent la solution. Par conséquent, une large part d’intervention manuelle est souvent nécessaire pour interpréter les données, déterminer la cause des problèmes et recommander une stratégie appropriée pour les corriger. Il peut être judicieux de simplement stocker une copie de ces informations dans leur format d’origine et de les rendre disponibles en vue d’une analyse à froid par un expert.

## <a name="visualizing-data-and-raising-alerts"></a>Visualisation des données et déclenchements des alertes

Un aspect important de tout système de surveillance est la possibilité de présenter les données de manière à ce qu’un opérateur puisse repérer rapidement les tendances ou les problèmes. Il est également important de pouvoir informer rapidement un opérateur si un événement significatif nécessitant votre attention s’est produit.

La présentation des données peut prendre plusieurs formes, y compris la visualisation à l’aide de tableaux de bord, d’alertes et de rapports.

### <a name="visualization-by-using-dashboards"></a>Visualisation à l’aide de tableaux de bord

La façon la plus courante de visualiser des données consiste à utiliser des tableaux de bord qui affichent des informations sous forme de tableaux, de graphiques ou d’autres illustrations. Ces éléments peuvent être paramétrés, et un analyste doit être en mesure de sélectionner les paramètres importants (par exemple, la période) correspondant à toute situation spécifique.

Les tableaux de bord peuvent être organisés de façon hiérarchique. Les tableaux de bord de niveau supérieur donnent une vue d’ensemble de chaque aspect du système tout en permettant à un opérateur d’explorer les détails. Par exemple, un tableau de bord qui illustre les E/S de disque du système doit permettre à un analyste d’afficher les taux d’E/S de chaque disque individuel afin de déterminer si un ou plusieurs périphériques spécifiques sont responsables d’un volume de trafic disproportionné. Dans l’idéal, le tableau de bord doit également afficher des informations connexes, telles que la source de chaque requête (utilisateur ou activité) qui génère cette E/S. Ces informations peuvent alors servir à déterminer si (et comment) la charge doit être équitablement répartie entre les périphériques et si le système serait plus performant si plusieurs périphériques étaient ajoutés.

Un tableau de bord peut également utiliser un codage en couleurs ou d’autres repères visuels pour indiquer les valeurs qui apparaissent anormales ou qui sont en dehors d’une plage attendue. Utilisation de l’exemple précédent :

- Un disque indiquant un taux d’E/S proche de sa capacité maximale sur une période prolongée (un disque chaud) peut être mis en surbrillance en rouge.
- Un disque indiquant un taux d’E/S qui s’exécute périodiquement à sa limite maximale sur de brèves périodes (un disque tiède) peut être mis en surbrillance en jaune.
- Un disque présentant une utilisation normale peut s’afficher en vert.

Notez que pour être efficace, un système de tableau de bord doit incorporer les données brutes avec lesquelles travailler. Si vous créez votre propre système de tableau de bord ou utilisez un tableau de bord développé par une autre organisation, vous devez comprendre quelles données d’instrumentation collecter, à quel niveau de granularité et comment elles doivent être formatées pour être utilisées par le tableau de bord.

Un bon tableau de bord n’affiche pas uniquement les informations, il permet également à un analyste de poser des questions ad hoc sur ces informations. Certains systèmes fournissent des outils de gestion pouvant être utilisés par un opérateur pour effectuer ces tâches et explorer les données sous-jacentes. Vous pouvez également, en fonction du référentiel utilisé pour conserver ces informations, interroger ces données directement ou les importer dans des outils tels que Microsoft Excel pour une analyse complémentaire et le reporting.

> [!NOTE]
> Vous devez restreindre l’accès aux tableaux de bord au personnel autorisé, car ces informations peuvent être sensibles. Vous devez également protéger les données sous-jacentes des tableaux de bord pour empêcher les utilisateurs de les modifier.

### <a name="raising-alerts"></a>Déclenchement d’alertes

La génération d’alerte consiste à analyser les données de surveillance et d’instrumentation, et à générer une notification si un événement significatif est détecté.

Les alertes permettent de s’assurer que le système reste sain, réactif et sécurisé. C’est une partie importante de tout système qui donne des garanties de performances, de disponibilité et de confidentialité aux utilisateurs, et dans lequel les données peuvent devoir être traitées immédiatement. Un opérateur peut devoir être notifié de l’événement qui a déclenché l’alerte. La génération d’alerte peut également servir à appeler des fonctions système telles que la mise à l’échelle automatique.

La génération d’alerte dépend généralement des données d’instrumentation suivantes :

- **Événements de sécurité**. Si les journaux d’événements indiquent que des échecs d’authentification et/ou d’autorisation répétés se produisent, le système peut être attaqué, et un opérateur doit en être informé.
- **Mesures de performances**. Le système doit répondre rapidement si une mesure de performance spécifique dépasse un seuil spécifié.
- **Informations de disponibilité**. Si une erreur est détectée, il peut être nécessaire de redémarrer rapidement un ou plusieurs sous-systèmes ou de basculer vers une ressource de sauvegarde. Des erreurs répétées dans un sous-système peuvent indiquer des problèmes plus sérieux.

Les opérateurs peuvent recevoir les informations d’alerte à l’aide de plusieurs canaux de remise comme un courrier électronique, un périphérique de radiomessagerie ou un SMS. Une alerte peut également inclure une indication du niveau de criticité d’une situation. De nombreux systèmes d’alerte prennent en charge les groupes d’abonnés, et un même ensemble d’alertes peut être envoyé à tous les opérateurs membres d’un même groupe.

Un système d’alerte doit être personnalisable, et les valeurs appropriées peuvent être fournies en tant que paramètres à partir des données d’instrumentation sous-jacentes. Cette approche permet à un opérateur de filtrer les données et de se concentrer sur ces seuils ou combinaisons de valeurs qui présentent un intérêt. Notez que dans certains cas, les données d’instrumentation brutes peuvent être fournies au système d’alerte. Dans d’autres situations, il peut être mieux approprié de fournir des données agrégées. (Par exemple, une alerte peut être déclenchée si l’utilisation du processeur pour un nœud a dépassé 90 % au cours des 10 dernières minutes.) Les détails fournis au système d’alerte doivent également inclure toutes les informations de résumé et de contexte appropriées. Ces données peuvent aider à réduire la possibilité que des événements de type faux positif déclenchent une alerte.

### <a name="reporting"></a>Reporting

Le reporting permet de générer une vue d’ensemble du système. Il peut intégrer des données historiques en plus des informations actuelles. Les exigences de reporting elles-mêmes se répartissent en deux grandes catégories : reporting opérationnel et reporting de sécurité.

Le reporting opérationnel inclut généralement les aspects suivants :

- Agrégation des statistiques qui vous permettent de comprendre l’utilisation des ressources du système global ou des sous-systèmes spécifiés pendant un laps de temps spécifié.
- Identification des tendances de l’utilisation des ressources pour le système global ou des sous-systèmes spécifiés pendant une période spécifiée.
- Supervision des exceptions qui se sont produites dans le système ou dans les sous-systèmes spécifiés pendant une période spécifiée.
- Détermination de l’efficacité de l’application quant aux ressources déployées et pour savoir si le volume des ressources (et leur coût associé) peut être réduit sans avoir d’impact inutile sur les performances.

Le reporting de sécurité vise à suivre l’utilisation du système par le client. Il peut inclure les éléments suivants :

- **Audit des opérations de l’utilisateur**. Cela nécessite d’enregistrer les requêtes individuelles effectuées par chaque utilisateur ainsi que les dates et heures. Les données doivent être structurées pour permettre à un administrateur de reconstruire rapidement la séquence des opérations effectuées par un utilisateur sur une période spécifiée.
- **Suivi de l’utilisation des ressources par l’utilisateur**. Cela nécessite d’enregistrer la manière dont chaque requête accède aux diverses ressources qui constituent le système, ainsi que la durée. Un administrateur doit pouvoir utiliser ces données pour générer un rapport d’utilisation par utilisateur sur une période spécifiée, éventuellement à des fins de facturation.

Dans de nombreux cas, les traitements par lots peuvent générer des rapports en fonction d’une planification définie. (La latence ne constitue pas normalement un problème.) Cependant ils doivent également être disponibles pour une génération sur une base ad hoc si nécessaire. Par exemple, si vous stockez des données dans une base de données relationnelle telle qu’une base de données SQL Azure, vous pouvez utiliser un outil tel que SQL Server Reporting Services pour extraire et formater les données et les présenter sous la forme d’un ensemble de rapports.

## <a name="related-patterns-and-guidance"></a>Conseils et modèles connexes

- [Recommandations en matière de mise à l’échelle automatique](../best-practices/auto-scaling.md) décrit comment alléger les contraintes de gestion en réduisant le recours à un opérateur dédié à la surveillance continue des performances d’un système, ainsi qu’à l’ajout et au retrait des ressources.
- [Modèle Supervision de point de terminaison](../patterns/health-endpoint-monitoring.md) décrit l’implémentation des contrôles fonctionnels au sein d’une application à laquelle accèdent des outils externes par le biais de points de terminaison exposés à intervalles réguliers.
- [Modèle de file d’attente avec un ordre de priorité](../patterns/priority-queue.md) indique comment hiérarchiser les messages en file d’attente afin que les requêtes urgentes soient reçues et puissent être traitées avant les messages moins urgents.

## <a name="more-information"></a>Plus d’informations

- [Analyser, diagnostiquer et dépanner Microsoft Azure Storage](/azure/storage/storage-monitoring-diagnosing-troubleshooting)
- [Azure : Telemetry Basics and Troubleshooting (Azure : Concepts de base de la télémétrie et résolution des problèmes)](https://social.technet.microsoft.com/wiki/contents/articles/18146.windows-azure-telemetry-basics-and-troubleshooting.aspx)
- [Activation de Diagnostics dans les services cloud et les machines virtuelles Azure](/azure/cloud-services/cloud-services-dotnet-diagnostics)
- [Cache Redis Azure](https://azure.microsoft.com/services/cache/), [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) et [HDInsight](https://azure.microsoft.com/services/hdinsight/)
- [Utilisation des files d’attente Service Bus](/azure/service-bus-messaging/service-bus-dotnet-get-started-with-queues)
- [Business Intelligence de SQL Server dans Azure Virtual Machines](/azure/virtual-machines/windows/sqlclassic/virtual-machines-windows-classic-ps-sql-bi)
- [Réception de notifications d’alerte](/azure/monitoring-and-diagnostics/insights-receive-alert-notifications) et [Suivi de l’intégrité du service](/azure/monitoring-and-diagnostics/insights-service-health)
- [Application Insights](/azure/application-insights/app-insights-overview)
