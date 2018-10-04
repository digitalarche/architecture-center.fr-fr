---
title: Recommandations en matière de partitionnement des données
description: Recommandations concernant le fractionnement de partitions pour une gestion et un accès distincts.
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: 579f78004a662fe412cb8dcc25b9ad1db9782125
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/28/2018
ms.locfileid: "47429500"
---
# <a name="data-partitioning"></a>Partitionnement des données

Au sein de nombreuses solutions à grande échelle, les données sont divisées en partitions distinctes qui peuvent être gérées et accessibles séparément. La stratégie de partitionnement doit être choisie avec soin afin d’optimiser les avantages tout en réduisant les effets négatifs. Le partitionnement peut aider à améliorer l’extensibilité, à réduire la contention et à optimiser les performances. Autre avantage du partitionnement, il peut fournir un mécanisme permettant de diviser les données selon le modèle d’utilisation. Par exemple, vous pouvez archiver les données (froides) anciennes et moins actives dans le stockage de données le plus économique.

## <a name="why-partition-data"></a>Pourquoi partitionner les données ?
La plupart des services et applications cloud stockent et récupèrent des données dans le cadre de leurs opérations. La conception des magasins de données utilisés par une application peut avoir une incidence considérable sur les performances, le débit et l’évolutivité d’un système. Une technique couramment appliquée dans les systèmes à grande échelle consiste à diviser les données en partitions distinctes.

> Dans cet article, le terme *partitionnement* désigne le processus de division physique des données sous forme de magasins de données distincts. Il convient de le distinguer du partitionnement de tables SQL Server, qui est un concept différent.

Le partitionnement des données présente un certain nombre d’avantages. Par exemple, il peut être utilisé pour :

* **Améliorer l’évolutivité**. Quand vous procédez à la montée en puissance d’un système de base de données unique, celui-ci finit par atteindre une limite liée au matériel physique. Si vous divisez les données en plusieurs partitions, chacune étant hébergée sur un serveur distinct, vous pouvez faire évoluer le système presque indéfiniment.
* **Améliorer les performances**. Les opérations d’accès aux données présentes au sein de chaque partition interviennent sur un plus petit volume de données. Sous réserve que les données sont partitionnées convenablement, le partitionnement peut améliorer l’efficacité de votre système. Les opérations qui affectent plusieurs partitions peuvent s’exécuter en parallèle. Chaque partition peut être située près de l’application qui l’utilise afin de réduire la latence du réseau.
* **Améliorer la disponibilité**. Diviser les données au sein de plusieurs serveurs permet d’éviter un point de défaillance unique. Si un serveur tombe en panne ou est en cours de maintenance planifiée, seules les données présentes au sein de cette partition sont indisponibles. Les opérations intervenant sur les autres partitions peuvent se poursuivre. L’augmentation du nombre de partitions réduit l’impact relatif d’une défaillance sur un serveur unique en réduisant le pourcentage de données qui ne seront pas disponibles. La réplication de chaque partition permet de réduire le risque d’une défaillance de partition unique qui affecterait les opérations. Cela permet aussi de séparer les données critiques qui doivent être hautement disponibles en permanence des données peu importantes, dont les exigences de disponibilité sont moindres (fichiers journaux, par exemple).
* **Améliorer la sécurité**. Selon la nature des données et la manière dont elles sont partitionnées,les données sensibles et non sensibles peuvent être séparées dans différentes partitions et donc dans différents serveurs ou magasins de données. Il est ensuite possible d’optimiser le sécurité de manière spécifique concernant les données sensibles.
* **Procurer une flexibilité opérationnelle**. Le partitionnement offre de nombreuses possibilités de réglage des opérations, d’optimisation de l’efficacité de l’administration et de réduction des coûts. Par exemple, vous pouvez définir différentes stratégies de gestion, surveillance, sauvegarde et restauration et autres tâches d’administration en fonction de l’importance des données présentes dans chaque partition.
* **Faire correspondre le magasin de données au modèle d’utilisation**. Le partitionnement permet le déploiement de chaque partition sur un type de magasin de données différent, en fonction du coût et des fonctionnalités intégrées proposées par le magasin de données. Par exemple, il est possible de stocker les données binaires volumineuses dans un magasin de données blob et de stocker les données plus structurées dans une base de données de documents. Pour plus d’informations, consultez [Building a Polyglot Solution] dans le guide des modèles et pratiques, et [Data Access for Highly-Scalable Solutions: Using SQL, NoSQL, and Polyglot Persistence] sur le site web Microsoft.

Certains systèmes n’implémentent pas le partitionnement, car cette technique est considérée comme un coût plutôt qu’un avantage. Les arguments courants motivant ce point de vue sont les suivants :

* De nombreux systèmes de stockage de données ne prennent pas en charge les jointures entre les partitions, et il peut s’avérer difficile de maintenir l’intégrité référentielle au sein d’un système partitionné. Il est souvent nécessaire de mettre en œuvre des jointures et vérifications de l’intégrité dans le code applicatif (au niveau de la couche de partitionnement), ce qui peut conduire à une augmentation des E/S et une complexité applicative supplémentaire.
* La maintenance des partitions ne se révèle pas toujours aisée. Dans un système où les données sont volatiles, vous serez peut-être amené à rééquiliber régulièrement les partitions pour réduire la contention et les zones sensibles.
* Certains outils courants ne fonctionnent pas naturellement avec des données partitionnées.

## <a name="designing-partitions"></a>Concevoir des partitions
Il est possible de partitionner les données de différentes manières : horizontale, verticale ou fonctionnelle. La stratégie que vous choisissez dépend des motifs du partitionnement des données et des besoins des applications et services qui utilisent ces données.

> [!NOTE]
> Les schémas de partitionnement décrits dans ces recommandations sont expliqués indépendamment de la technologie de stockage de données sous-jacente. Ces schémas peuvent être appliqués à de nombreux types de magasins de données, notamment les bases de données relationnelles et NoSQL.
>
>

### <a name="partitioning-strategies"></a>Stratégies de partitionnement
Les trois stratégies de partitionnement des données habituelles sont les suivantes :

* **Partitionnement horizontal** (souvent appelé *sharding*). Dans cette stratégie, chaque partition est un magasin de données à part entière, mais toutes les partitions ont le même schéma. Chaque partition est appelée *shard* et comporte une partie spécifique des données, par exemple, toutes les commandes d’un groupe spécifique de clients dans une application de commerce électronique.
* **Partitionnement vertical**. Dans cette stratégie, chaque partition comporte une partie des champs des éléments présents dans le magasin de données. Les champs sont divisés selon leur modèle d’utilisation. Par exemple, les champs fréquemment utilisés peuvent être placés dans une partition verticale et les champs moins fréquemment utilisés dans une autre.
* **Partitionnement fonctionnel**. Dans cette stratégie, les données sont agrégées en fonction de leur utilisation par chaque contexte limité au sein du système. Par exemple, un système de commerce électronique qui implémente des fonctions d’entreprise distinctes pour la facturation et la gestion de l’inventaire des produits peut stocker les données de facturation dans une partition et les données d’inventaire des produits dans une autre.

Il est important de noter qu’il est possible d’associer les trois stratégies décrites. Elles ne s’excluent pas mutuellement et nous vous recommandons de les considérer dans leur ensemble au moment de concevoir un schéma de partitionnement. Vous pouvez, par exemple, diviser les données en partitions, puis utiliser le partitionnement vertical pour ensuite sous-diviser les données au sein de chaque partition. De même, les données contenues dans une partition fonctionnelle peuvent être divisées en partitions (qui peuvent aussi faire l’objet d’un partitionnement vertical).

Cependant, les différentes exigences de chaque stratégie peuvent présenter un certain nombre d’enjeux conflictuels. Vous devez évaluer et mettre en balance tous ces paramètres pour concevoir un schéma de partitionnement qui respecte les objectifs généraux de performance de traitement des données de votre système. Les sections qui suivent décrivent chacune des stratégies plus en détail.

### <a name="horizontal-partitioning-sharding"></a>Partitionnement horizontal (sharding)
La figure 1 présente une vue d’ensemble du partitionnement horizontal ou sharding. Dans cet exemple, les données relatives à l’inventaire des produits sont divisées en partitions en fonction de la clé du produit. Chaque partition comporte les données relatives à une plage contiguë de clés de partition (A à G et H à Z), classées par ordre alphabétique.

![Partitionnement horizontal des données (sharding) en fonction d’une clé de partition](./images/data-partitioning/DataPartitioning01.png)

*Figure 1 : Partitionnement horizontal des données (sharding) en fonction d’une clé de partition*

Le partitionnement vous permet de répartir la charge sur davantage d’ordinateurs, ce qui réduit la contention et améliore les performances. Vous pouvez faire évoluer le système en ajoutant des partitions qui s’exécutent sur des serveurs supplémentaires.

Le facteur le plus important lors de la mise en œuvre de cette stratégie de partitionnement correspond au choix de la clé de partitionnement. Il peut s’avérer difficile de modifier la clé une fois le système est en fonctionnement. La clé doit garantir un partitionnement des données permettant de disposer d’une charge de travail aussi homogène que possible au sein des partitions.

Notez que les différentes partitions ne doivent pas nécessairement contenir des volumes de données similaires. En fait, le plus important est d’équilibrer le nombre de demandes. Si certaines partitions peuvent être très volumineuses, le nombre d’accès dont fait l’objet chaque élément est très faible. En revanche, dans d’autres partitions plus petites, chaque élément fait l’objet d’accès beaucoup plus fréquents. Il est aussi important de s’assurer qu’une partition ne dépasse pas les limites d’échelle (en termes de capacité et de ressources de traitement) du magasin de données servant à héberger cette partition.

Si vous utilisez un schéma de partitionnement, évitez de créer des zones sensibles (ou partitions actives) qui pourraient avoir des répercussions sur les performances et la disponibilité. Par exemple, si vous hachez l’identificateur d’un client plutôt que la première lettre de son nom, vous empêchez la répartition non équilibrée qui résulte de l’utilisation d’initiales plus ou moins courantes. Il s’agit d’une technique classique qui permet de répartir les données de façon plus homogène entre les partitions.

Choisissez une clé de partitionnement qui limite toute exigence future de fractionnement des partitions volumineuses en plus petits éléments, de regroupement de petites partitions en partitions plus volumineuses ou de modification du schéma qui décrit les données stockées dans un jeu de partitions. Ces opérations peuvent prendre beaucoup de temps et nécessiter la mise hors connexion d’une ou plusieurs partitions pendant leur exécution.

En cas de réplication de partitions, certains réplicas peuvent parfois être maintenus en ligne pendant que d’autres sont fractionnés, fusionnés ou reconfigurés. Cependant, le système peut être contraint de limiter les opérations qui peuvent être exécutées sur les données de ces partitions pendant que la reconfiguration se produit. Par exemple, les données des réplicas peuvent être marquées en lecture seule pour limiter la portée des incohérences qui pourraient survenir pendant la restructuration des partitions.

> Pour obtenir des informations et des conseils plus détaillés sur la plupart de ces considérations, ainsi que des techniques de bonnes pratiques pour concevoir des magasins de données qui implémentent le partitionnement horizontal, consultez [Modèle de partitionnement].
>
>

### <a name="vertical-partitioning"></a>Partitionnement vertical
L’utilisation la plus courante du partitionnement vertical vise à réduire les E/S et les coûts de performance associés à la recherche des éléments les plus fréquemment utilisés. La figure 2 montre un exemple de partitionnement vertical. Dans cet exemple, les différentes propriétés de chaque élément de données se trouvent dans des partitions différentes. L’un d’elles contient des données qui font l’objet d’accès plus fréquents, comme le nom, la description et les informations tarifaires des produits. Une autre contient le volume en stock et la date de la dernière commande.

![Partitionnement vertical des données en fonction du modèle d’utilisation](./images/data-partitioning/DataPartitioning02.png)

*Figure 2 : Partitionnement vertical des données en fonction du modèle d’utilisation*

Dans cet exemple, l’application émet des requêtes à intervalles réguliers concernant le nom, la description et le prix du produit quand il s’agit d’afficher les détails du produit aux clients. Le niveau des stocks et la date de dernière commande du produit auprès du fabricant sont stockés au sein d’une partition distincte, ces deux éléments étant généralement utilisés conjointement.

Ce schéma de partitionnement présente l’avantage de séparer les données relativement lentes (nom, description et prix du produit) des données plus dynamiques (niveau de stock et dernière date de commande). Une application peut tirer parti de la mise en mémoire cache des données lentes si elles font l’objet d’accès fréquents.

Un autre scénario habituel concernant cette stratégie de partitionnement consiste à optimiser la sécurité des données sensibles. Pour ce faire, vous pouvez par exemple stocker les numéros de carte de crédit et les numéros de vérification correspondants dans des partitions distinctes.

Le partitionnement vertical peut aussi limiter le nombre d’accès simultanés aux données nécessaires.

> Le partitionnement vertical fonctionne au niveau de l’entité au sein d’un magasin de données, en normalisant partiellement une entité pour organiser un *large* élément en un jeu d’éléments *restreints*. Il est parfaitement adapté aux magasins de données organisés en colonnes, tels que HBase et Cassandra. Si les données présentes au sein d’une collection de colonnes sont peu susceptibles d’être modifiées, vous pouvez également envisager d’utiliser des magasins organisés en colonnes dans SQL Server.
>
>

### <a name="functional-partitioning"></a>Partitionnement fonctionnel
Concernant les systèmes au sein desquels il est possible d’identifier un contexte limité pour chaque secteur d’activité ou service au sein de l’application, le partitionnement fonctionnel constitue une technique permettant d’améliorer les performances en matière d’isolement et d’accès aux données. Une autre utilisation courante du partitionnement fonctionnel consiste à séparer les données en lecture-écriture des données en lecture seule utilisées pour générer des rapports. La figure 3 présente une vue d’ensemble du partitionnement fonctionnel au sein duquel les données d’inventaire sont séparées des données relatives aux clients.

![Partitionnement fonctionnel des données en fonction du contexte limité ou du sous-domaine](./images/data-partitioning/DataPartitioning03.png)

*Figure 3 : Partitionnement fonctionnel des données en fonction du contexte limité ou du sous-domaine*

Cette stratégie de partitionnement peut contribuer à réduire la contention d’accès aux données entre les différentes parties d’un système.

## <a name="designing-partitions-for-scalability"></a>Concevoir des partitions évolutives
Il est essentiel de tenir compte de la taille et de la charge de travail de chaque partition et de les équilibrer de sorte que les données soient réparties de manière à assurer une extensibilité maximale. Cependant, vous devez également partitionner les données de sorte qu’elles ne dépassent pas les limites d’échelle d’un magasin de partitions.

Pour concevoir des partitions extensibles, procédez comme suit :

1. Analysez l’application pour comprendre les modèles d’accès aux données, telles que la taille du jeu de résultats retourné par chaque requête, la fréquence d’accès, la latence inhérente et les exigences de traitement de calcul côté serveur. Dans de nombreux cas, seules quelques entités principales nécessitent la majorité des ressources de traitement.
2. Servez-vous de cette analyse pour déterminer les objectifs d’extensibilité actuels et futurs, tels que la taille des données et la charge de travail. Répartissez ensuite les données entre les partitions pour satisfaire aux objectifs d’extensibilité. Concernant la stratégie de partitionnement horizontal, le choix de la clé de partitionnement appropriée est important pour s’assurer de l’homogénéité de la répartition. Pour plus d’informations, consultez [Modèle de partitionnement].
3. Assurez-vous que les ressources disponibles pour chaque partition sont suffisantes pour traiter les exigences d’évolutivité en termes de débit et de taille des données. Par exemple, le nœud qui héberge une partition peut imposer une limite stricte concernant la quantité d’espace de stockage, la puissance de traitement ou la bande passante réseau qu’il fournit. Si les exigences de stockage et de traitement des données sont susceptibles de dépasser ces limites, il peut s’avérer nécessaire d’affiner votre stratégie de partitionnement ou de fractionner davantage les données. Par exemple, une approche d’extensibilité peut consister à séparer les données de journalisation des principales fonctionnalités de l’application. Pour cela, vous pouvez utiliser différents magasins de données pour éviter que le total des besoins en stockage de données dépasse la limite de mise à l’échelle du nœud. Si le nombre total de magasins de données dépasse la limite du nœud, il peut être nécessaire d’utiliser des nœuds de stockage distincts.
4. Surveillez le système en cours d’utilisation pour vérifier que les données sont réparties comme prévu et que les partitions peuvent gérer la charge qui leur est affectée. Il est possible que l’utilisation ne corresponde pas à celle prévue par l’analyse. Dans ce cas, il est parfois possible de rééquilibrer les partitions. En cas d’échec, il peut s’avérer nécessaire de redéfinir certains composants du système pour obtenir l’équilibrage nécessaire.

Notez que certains environnements cloud allouent les ressources en fonction des limites de l’infrastructure. Vérifiez que les limites sélectionnées offrent assez d’espace pour permettre toute croissance anticipée en termes de volume de données, de stockage des données, de puissance de traitement et de bande passante.

Par exemple, si vous utilisez le stockage de table Azure, une partition occupée peut réclamer plus de ressources qu’il n’y en a à disposition dans une partition unique pour gérer les demandes. (Le volume de demandes qu’une même partition peut gérer sur une période donnée est limité. Pour plus d’informations, consultez la page [Azure Storage Scalability and Performance Targets] sur le site web Microsoft.)

 Si c’est le cas, vous devrez peut-être repartitionner la partition pour répartir la charge. Si la taille totale ou le débit de ces tables dépasse la capacité d’un compte de stockage, il peut être nécessaire de créer des comptes de stockage supplémentaires et de répartir les tables sur ces comptes. Si le nombre de comptes de stockage dépasse le nombre de comptes disponibles pour un abonnement, il peut être nécessaire d’utiliser plusieurs abonnements.

## <a name="designing-partitions-for-query-performance"></a>Concevoir des partitions pour garantir la performance des requêtes
La performance des requêtes peut souvent être améliorée en utilisant des jeux de données plus petits et en exécutant des requêtes parallèles. Chaque partition doit contenir une petite proportion du jeu de données entier. Cette réduction de volume peut améliorer la performance des requêtes. Cependant, le partitionnement n’est pas une alternative permettant de concevoir et de configurer une base de données de manière appropriée. Assurez-vous, par exemple, que vous disposez des index nécessaires si vous utilisez une base de données relationnelle.

Pour concevoir des partitions qui favorisent la performance des requêtes, procédez comme suit :

1. Examinez les exigences et les performances applicatives :
   * En fonction des exigences opérationnelles, identifiez les requêtes importantes qui doivent toujours être exécutées avec rapidité.
   * Surveillez le système afin d’identifier les requêtes s’exécutant lentement.
   * Identifiez les requêtes les plus fréquemment exécutées. Une seule instance de chaque requête peut correspondre à un coût minime, mais la consommation cumulée de ressources peut être considérable. Il peut être avantageux de séparer les données récupérées par ces requêtes dans une partition distincte, voire dans un cache.
2. Partitionnez les données à l’origine du ralentissement des performances :
   * Limitez la taille de chaque partition afin que le temps de réponse aux requêtes corresponde à l’objectif.
   * Concevez la clé de partitionnement de sorte que l’application puisse aisément trouver la partition si vous implémentez un partitionnement horizontal. La requête n’est alors pas contrainte de parcourir chaque partition.
   * Tenez compte de l’emplacement d’une partition. Si possible, essayez de conserver les données au sein de partitions physiquement proches des applications et utilisateurs qui y accèdent.
3. Si une entité présente des exigences en matière de débit et de performance des requêtes, utilisez le partitionnement fonctionnel en vous basant sur cette entité. Si cela ne permet toujours pas de satisfaire aux exigences, appliquez aussi un partitionnement horizontal. Dans la plupart des cas, une seule stratégie de partitionnement suffit, mais dans certains cas, il s’avère plus efficace d’associer les deux stratégies.
4. Envisagez d’utiliser des requêtes asynchrones qui s’exécutent en parallèle sur plusieurs partitions pour améliorer les performances.

## <a name="designing-partitions-for-availability"></a>Concevoir des partitions évolutives
Le fait de partitionner des données peut améliorer la disponibilité des applications en veillant à ce que l’ensemble du jeu de données ne constitue pas un point de défaillance unique et que les sous-ensembles individuels du jeu de données puissent être gérés indépendamment. Le fait de répliquer des partitions qui contiennent des données critiques peut aussi contribuer à améliorer la disponibilité.

Lors de la conception et de la mise en œuvre des partitions, tenez compte des facteurs suivants qui affectent la disponibilité :

* **Importance des données concernant les opérations d’exploitation**. Certaines données peuvent comporter des informations d’entreprise importantes, telles que des détails de facture ou des transactions bancaires. D’autres données peuvent inclure des données opérationnelles moins importantes, comme des fichiers journaux, des suivis de performances, etc. Après avoir identifié le type des données, considérez les points suivants :
  * Stockage des données critiques au sein de partitions hautement disponibles avec un plan de sauvegarde approprié.
  * Établissement de mécanismes ou procédures de gestion et de surveillance distincts selon l’importance de chaque jeu de données. Placez les données présentant le même niveau d’importance dans la même partition pour qu’elles puissent être sauvegardées ensemble selon une fréquence appropriée. Par exemple, les partitions qui comportent des données de transactions bancaires doivent être sauvegardées plus fréquemment que les partitions qui comportent des informations de journalisation ou de suivi.
* **Gestion des partitions individuelles**. Le fait de créer des partitions venant favoriser une gestion et une maintenance indépendantes présente plusieurs avantages. Par exemple : 
  * En cas de défaillance d’une partition, elle peut être récupérée de manière indépendante, sans affecter les instances des applications accédant aux données présentes au sein des autres partitions.
  * Le partitionnement des données par zone géographique permet l’exécution des tâches de maintenance planifiée à des heures creuses pour chaque emplacement. Assurez-vous que les partitions ne sont pas trop volumineuses pour empêcher l’exécution de toute maintenance planifiée au cours de cette période.
* **Réplication des données importantes dans plusieurs partitions**. Cette stratégie peut améliorer la disponibilité et les performances, bien qu’elle puisse également présenter certains problèmes de cohérence. Synchroniser les modifications apportées aux données contenues dans une partition avec chaque réplica prend du temps. Au cours de cette période, les différentes partitions contiennent des valeurs de données différentes.

## <a name="understanding-how-partitioning-affects-design-and-development"></a>Impact du partitionnement sur la conception et le développement
L’utilisation du partitionnement complique la conception et le développement du système. Envisagez le partitionnement comme une composante fondamentale de la conception du système, même si celui-ci ne contient au départ qu’une seule partition. Si vous décidez d’utiliser le partitionnement après coup, une fois que le système commence à présenter des problèmes de performance et d’extensibilité, cela ne fera qu’accroître la complexité, dans la mesure où vous aurez déjà un système actif à gérer.

Si vous mettez à jour le système de façon à intégrer le partitionnement dans cet environnement, vous devrez modifier la logique d’accès aux données. Vous pourrez aussi être amené à migrer de grandes quantités de données existantes de façon à les répartir entre les partitions, souvent à des moments où les utilisateurs auront besoin d’utiliser le système.

Dans certains cas, le partitionnement n’est pas considéré comme un élément important, le jeu de données initial étant peu volumineux et pouvant être facilement traité par un seul serveur. Cela peut être le cas dans un système qui n’est pas censé évoluer au-delà de sa taille initiale, mais de nombreux systèmes commerciaux doivent évoluer à mesure que le nombre d’utilisateurs augmente. Cette évolution est généralement accompagnée d’une croissance du volume de données.

De même, il est important de comprendre que le partitionnement n’est pas toujours réservé aux magasins de données volumineux. Par exemple, un magasin de données de petite taille peut faire l’objet d’accès de la part de centaines de clients simultanés. Le fait de partitionner les données dans une telle situation peut aider à réduire la contention et améliorer le débit.

Au moment de concevoir un schéma de partitionnement de données, tenez compte des points suivants :

* **Si possible, conservez les données relatives aux opérations de base de données les plus courantes dans chaque partition afin de réduire les opérations d’accès aux données entre partitions**. Le fait d’interroger plusieurs partitions peut se révéler plus long que d’interroger une seule partition, mais l’optimisation des partitions d’un jeu de requêtes peut, au contraire, affecter les autres jeux de requêtes. Quand vous ne pouvez pas éviter d’interroger plusieurs partitions, limitez la durée des requêtes en exécutant des requêtes parallèles et en agrégeant les résultats dans l’application. Il se peut néanmoins que cette approche ne soit pas possible dans certains cas, par exemple quand le résultat d’une requête doit être obtenu et utilisé dans la requête suivante.
* **Si les requêtes utilisent des données de référence relativement statiques, telles que des tables de codes postaux ou des listes de produits, envisagez de répliquer ces données dans toutes les partitions de façon à réduire le recours à des opérations de recherche distinctes dans différentes partitions**. Cette approche peut aussi limiter les chances de voir les données de référence devenir un jeu de données « sensible » soumis à un trafic dense à l’échelle du système entier. Cependant, la synchronisation des modifications pouvant intervenir sur ces données de référence s’accompagnent de coûts supplémentaires.
* **Dans la mesure du possible, réduisez les exigences en matière d’intégrité référentielle dans les partitions verticales et fonctionnelles**. Dans ces schémas, l’application elle-même est chargée de maintenir l’intégrité référentielle au sein des partitions lorsque les données sont mises à jour et utilisées. Les requêtes qui doivent joindre les données de plusieurs partitions s’exécutent plus lentement que celles qui joignent les données d’une même partition, car l’application doit généralement exécuter des requêtes consécutives basées sur une clé, puis sur une clé étrangère. Envisagez plutôt de répliquer ou de dénormaliser les données pertinentes. Pour réduire le durée des requêtes consultant plusieurs partitions, exécutez des requêtes parallèles sur les partitions et joignez les données dans l’application.
* **Considérez l’effet possible du schéma de partitionnement sur la cohérence des données au sein des partitions.** Déterminez si une forte cohérence est un élément indispensable. Une approche courante alternative dans le cloud consiste à mettre en œuvre une cohérence finale. Les données de chaque partition sont mises à jour séparément, et la logique d’application vérifie que toutes les mises à jour aboutissent. De même, elle gère les incohérences qui peuvent découler de l’interrogation de données pendant l’exécution d’une opération finalement cohérente. Pour plus d’informations sur l’implémentation de la cohérence finale, consultez [Data Consistency Primer](Manuel d’introduction à la cohérence des données).
* **Tenez compte de la façon dont les requêtes localisent la partition appropriée**. Si une requête doit parcourir toutes les partitions pour localiser les données souhaitées, cela affecte considérablement les performances, même si plusieurs requêtes en parallèle sont exécutées. Les requêtes utilisées avec des stratégies de partitionnement vertical et fonctionnel peuvent spécifier naturellement les partitions. Cependant, le partitionnement horizontal (sharding) peut contrarier la localisation d’un élément, car chaque partition a le même schéma. Une solution de partitionnement classique consiste à actualiser une carte pouvant être utilisée pour rechercher l’emplacement de la partition afin de consulter des éléments de données spécifiques. Cette carte peut être implémentée dans la logique de partitionnement de l’application ou tenue à jour par le magasin de données s’il prend en charge le partitionnement transparent.
* **Quand vous utilisez une stratégie de partitionnement horizontal, pensez à rééquilibrer régulièrement les partitions**. Cela permet de répartir les données de manière homogène en fonction de leur taille et de la charge de travail et ainsi de réduire les zones sensibles, d’optimiser la performance des requêtes et de contourner les limites de stockage physique. Il s’agit cependant d’une tâche complexe qui nécessite souvent l’utilisation d’un processus ou d’un outil personnalisé.
* **Si vous répliquez chaque partition, elle offre une protection supplémentaire contre les défaillances**. Si une réplique connaît une défaillance, les requêtes peuvent être dirigées vers une copie de travail.
* **Si vous atteignez les limites physiques d’une stratégie de partitionnement, vous devrez peut-être développer l’extensibilité à un autre niveau**. Par exemple, si le partitionnement se situe au niveau de la base de données, vous devrez peut-être localiser ou répliquer les partitions dans plusieurs bases de données. Si le partitionnement se situe déjà au niveau de la base de données et que les limitations physiques posent problème, vous serez peut-être amené à localiser ou à répliquer les partitions dans plusieurs comptes d’hébergement.
* **Évitez les transactions qui accèdent à des données présentes dans plusieurs partitions**. Certains magasins de données implémentent l’intégrité et la cohérence transactionnelles pour les opérations qui modifient les données, mais seulement quand ces données se trouvent dans une même partition. Si vous devez disposer d’une prise en charge au sein de plusieurs partitions, il vous faut probablement mettre en œuvre ce mécanisme en l’intégrant à votre logique applicative, la plupart des systèmes de partitionnement ne proposant pas de prise en charge native.

Tous les magasins de données nécessitent une certaine gestion opérationnelle et une certaine surveillance. Les tâches associées peuvent aller du chargement, de sauvegarde, de la restauration et de la réorganisation des données à la vérification du fonctionnement correct et efficace du système.

Tenez compte des facteurs suivants qui affectent la gestion des opérations :

* **Implémentation des tâches opérationnelles et de gestion appropriées quand les données sont partitionnées**. Ces tâches peuvent inclure la sauvegarde et la restauration, l’archivage de données, la surveillance du système et d’autres tâches d’administration. Par exemple, la maintenance de la cohérence logique au cours des opérations de sauvegarde et de restauration peut se révéler compliquée.
* **Chargement des données dans plusieurs partitions et ajout de nouvelles données en provenance d’autres sources**. Il peut arriver que certains outils et utilitaires ne prennent pas en charge certaines opérations de données partitionnées, telles que le chargement des données dans la partition appropriée. Cela signifie que vous serez peut-être amené à créer ou à vous procurer de nouveaux outils et utilitaires.
* **Archivage et suppression des données de façon régulière**. Pour éviter une croissance excessive des partitions, vous devez archiver et supprimer les données de façon régulière (parfois tous les mois). Il peut être nécessaire de transformer les données pour les adapter à un schéma d’archivage différent.
* **Identification des problèmes d’intégrité des données**. Envisagez d’exécuter un processus périodique capable d’identifier tout problème d’intégrité de données, par exemple, des données au sein d’une seule partition qui font référence à des informations manquantes dans une autre. Le processus peut soit tenter de résoudre ces problèmes automatiquement, soit déclencher une alerte auprès d’un opérateur pour qu’il les corrige manuellement. Par exemple, dans une application de commerce électronique, les informations relatives aux commandes peuvent être stockées dans une partition, mais les éléments de ligne constituant chaque commande peuvent être stockés dans une autre partition. Le processus pour passer une commande doit ajouter des données à d’autres partitions. Si ce processus échoue, il se peut que certains éléments de ligne stockés ne correspondent à aucune commande.

Les différentes technologies de stockage de données fournissent généralement leurs propres fonctionnalités pour prendre en charge le partitionnement. Les sections suivantes récapitulent les options qui sont implémentées par les magasins de données couramment utilisés par les applications Azure. Elles décrivent également comment concevoir des applications pour tirer le meilleur parti de ces fonctionnalités.

## <a name="partitioning-strategies-for-azure-sql-database"></a>Stratégies de partitionnement pour Azure SQL Database
Azure SQL Database est une base de données relationnelle sour forme de service qui s’exécute dans le cloud. Ce service est basé sur Microsoft SQL Server. Une base de données relationnelle divise les informations en différentes tables, chacune de ces tables comprenant des informations relatives aux entités sous forme de séries de lignes. Chaque ligne comporte des colonnes qui contiennent les données relatives aux champs individuels d’une entité. La page [Qu’est-ce qu’Azure SQL Database ?] sur le site web de Microsoft présente des informations détaillées concernant la création et l’utilisation de bases de données SQL.

## <a name="horizontal-partitioning-with-elastic-database"></a>Partitionnement horizontal avec la fonction Base de données élastique
Une base de données SQL est limitée quant au volume de données qu’elle peut contenir. Le débit est limité par des facteurs architecturaux et par le nombre de connexions simultanées prises en charge. La fonctionnalité de base de données élastique de SQL Database prend en charge la mise à l’échelle horizontale d’une base de données SQL. Grâce à elle, vous pouvez partitionner vos données sur des partitions réparties sur plusieurs bases de données SQL. Vous pouvez également ajouter ou supprimer des partitions à mesure que le volume de données que vous devez gérer augmente ou diminue. La fonction Base de données élastique permet également de réduire la contention en répartissant la charge parmi les bases de données.

> [!NOTE]
> La fonctionnalité de base de données élastique remplace la fonctionnalité de fédérations d’Azure SQL Database. Les installations existantes de Fédération pour SQL Database peuvent être migrées vers une base de données élastique à l’aide de l’utilitaire de migration de fédérations. Vous pouvez également implémenter votre propre mécanisme de partitionnement si votre système ne se prête pas naturellement aux fonctionnalités offertes par les bases de données élastiques.
>
>

Chaque partition est mise en œuvre sous forme de base de données SQL. Une partition peut contenir plusieurs jeux de données (appelés *shardlets*). Chaque base de données tient à jour des métadonnées qui décrivent les shardlets qu’elle contient. Un shardlet peut correspondre à un seul élément de données ou à un groupe d’éléments partageant la même clé de shardlet. Par exemple, si vous procédez au partitionnement de données au sein d’une application mutualisée, la clé de shardlet peut correspondre à l’ID client, et toutes les données relatives à un client seront stockées au sein du même shardlet. Les données relatives aux autres clients seront stockées au sein de différents shardlets.

Il incombe au programmeur d’associer un groupe de données à une clé de shardlet. Une base de données SQL distincte fonctionne comme un gestionnaire global des cartes des partitions. Cette base de données contient une liste de l’ensemble des partitions et shardlets dans le système. Une application cliente qui accède aux données se connecte d’abord à la base de données du gestionnaire global des cartes des partitions pour obtenir une copie de la carte (répertoriant les partitions et les shardlets), qu’elle place ensuite en mémoire cache locale.

L’application utilise ensuite ces informations pour acheminer les demandes de données vers la partition appropriée. Cette fonctionnalité est masquée derrière une série d’API présentes dans la Bibliothèque cliente Base de données élastique Azure SQL Database, qui est disponible sous forme de package NuGet. La page [Vue d’ensemble des fonctionnalités de bases de données élastiques] sur le site Web de Microsoft dresse une présentation plus complète de la fonction Base de données élastique.

> [!NOTE]
> Vous pouvez répliquer la base de données du gestionnaire global des cartes des partitions pour réduire la latence et améliorer la disponibilité. Si vous implémentez la base de données à l’aide de l’un des niveaux tarifaires Premium, vous pouvez configurer la géoréplication active pour copier des données en continu dans des bases de données présentes dans différentes régions. Créez une copie de la base de données dans chaque région où résident des utilisateurs. Ensuite, configurez votre application pour qu’elle se connecte à cette copie pour obtenir la carte des partitions.
>
> Une autre approche consiste à utiliser la Synchronisation des données SQL Azure ou un pipeline Azure Data Factory pour répliquer la base de données du gestionnaire des cartes des partitions dans les différentes régions. Cette forme de réplication s’exécute périodiquement et est plus appropriée si la carte des partitions change peu fréquemment. En outre, vous n’êtes pas obligé de créer la base de données du gestionnaire des cartes des partitions à l’aide d’un niveau tarifaire Premium.
>
>

La fonction Base de données élastique présente deux schémas de mappage des données vers les shardlets et de stockage dans ces derniers :

* Une **carte de partitions de liste** décrit une association entre une clé unique et un shardlet. Par exemple, dans un système mutualisé, les données relatives à chaque client peuvent être associées à une clé unique et stockées dans leur propre shardlet. Pour garantir la confidentialité et l’isolement (autrement dit, pour empêcher un client d’épuiser les ressources de stockage de données disponibles pour d’autres), chaque shardlet peut être stocké dans sa propre partition.

![Utilisation d’une carte de partitions de liste pour stocker les données d’un client dans des partitions distinctes](./images/data-partitioning/PointShardlet.png)

*Figure 4 : Utilisation d’une carte de partitions de liste pour stocker les données d’un client dans des partitions distinctes*

* Une **carte de partitions de plage** décrit une association entre un jeu de valeurs de clé contiguës et un shardlet. Dans l’exemple d’architecture mutualisée décrit précédemment, vous pouvez, comme alternative à la mise en œuvre de shardlets dédiés, regrouper les données d’un jeu de clients (chacun présentant sa propre clé) au sein du même shardlet. Ce schéma est moins onéreux que le premier (car les clients partagent les ressources de stockage de données), mais il présente un risque de baisse de la confidentialité et de l’isolement des données.

![Utilisation d’une carte de partitions de plage pour stocker des données relatives à une plage de clients dans une partition](./images/data-partitioning/RangeShardlet.png)

*Figure 5 : Utilisation d’une carte de partitions de plage pour stocker des données relatives à une plage de clients dans une partition*

Remarquez qu’une même partition peut comporter des données relatives à plusieurs shardlets. Par exemple, vous pouvez utiliser des  shardlets de liste pour stocker des données relatives à différents clients non contigus dans la même partition. Vous pouvez aussi associer des shardlets de plage et des shardlets de liste dans une même partition, bien qu’ils soient interrogés par le biais de différentes cartes au sein de la base de données du gestionnaire global des cartes des partitions. (La base de données du gestionnaire global des cartes des partitions peut contenir plusieurs cartes des partitions.) La figure 6 illustre cette approche.

![Implémentation de plusieurs cartes des partitions](./images/data-partitioning/MultipleShardMaps.png)

*Figure 6 : Implémentation de plusieurs cartes des partitions*

Le schéma de partitionnement que vous implémentez peut avoir une incidence significative sur les performances de votre système. Il peut également affecter la vitesse à laquelle les partitions doivent être ajoutées ou supprimées, ou la vitesse à laquelle les données doivent être repartitionnées entre les partitions. Considérez les points suivants quand vous utilisez la fonction Base de données élastique pour partitionner des données :

* Regroupez les données utilisées conjointement dans une même partition et évitez les opérations nécessitant un accès à des données présentes dans plusieurs partitions. N’oubliez pas qu’avec la fonctionnalité de base de données élastique, une partition constitue une base de données SQL à part entière et qu’Azure SQL Database ne prend pas en charge les jointures entre bases de données (qui doivent être effectuées côté client). Souvenez-vous également qu’avec Azure SQL Database, les contraintes d’intégrité référentielle, les déclencheurs et les procédures stockées dans une base de données ne peuvent pas faire référence à des objets présents dans une autre. Aussi, ne concevez pas un système présentant des dépendances entre les partitions. Une base de données SQL peut cependant contenir des tables qui contiennent des copies des données de référence fréquemment utilisées par des requêtes et d’autres opérations. Il n’est pas obligatoire que ces tables appartiennent à un shardlet spécifique. Le fait de répliquer ces données sur des partitions peut permettre de s’affranchir de la nécessité de consulter des données présentes dans différentes bases de données. Dans l’idéal, ces données doivent être statiques ou lentes afin de réduire les efforts de réplication et de diminuer le risque qu’elles deviennent obsolètes.

  > [!NOTE]
  > Bien que SQL Database ne prenne pas en charge les jointures entre bases de données, vous pouvez exécuter des requêtes sur plusieurs partitions avec l’API de base de données élastique. Ces requêtes peuvent itérer en toute transparence les données contenues dans tous les shardlets qui sont référencés par une carte des partitions. L’API Base de données élastique répartit les requêtes sur plusieurs partitions en une série de requêtes individuelles (une pour chaque base de données), puis fusionne les résultats. Pour plus d’informations, consultez la page [Requête sur plusieurs partitions] sur le site web Microsoft.
  >
  >
* Les données stockées au sein des shardlets appartenant à la même carte des partitions doivent présenter le même schéma. Par exemple, ne créez pas une carte de partitions de liste pointant vers des shardlets comportant des données relatives aux clients et d’autres shardlets comportant des informations relatives aux produits. Cette règle n’est pas indispensable pour la fonction Base de données élastique, mais la gestion et l’interrogation des données deviennent très complexes si chaque shardlet présente un schéma différent. Dans l’exemple que nous venons de citer, une bonne solution consiste à créer deux cartes de partitions de liste : une qui référence les données relatives aux clients, et une autre qui pointe vers les informations sur les produits. N’oubliez pas que les données appartenant à différentes shardlets peuvent être stockées au sein de la même partition.

  > [!NOTE]
  > La fonctionnalité de requête entre partitions de l'API Base de données élastique dépend de chaque shardlet dans la carte de partitions contenant le même schéma.
  >
  >
* Les opérations transactionnelles sont prises en charge uniquement pour les données stockées dans la même partition, et non dans différentes partitions. Les transactions peuvent être stockées au sein de différents shardlets tant qu’elles font partie de la même partition. Par conséquent, si votre logique professionnelle nécessite la réalisation de transactions, stockez les données concernées au sein de la même partition ou mettez en œuvre la cohérence finale. Pour plus d’informations, consultez les [Data Consistency Primer].
* Disposez les partitions à proximité des utilisateurs qui accèdent aux données stockées dans ces partitions (autrement dit, géolocalisez les partitions). Cette stratégie aide à réduire la latence.
* Évitez de disposer de diverses partitions très actives (zones sensibles) et de partitions relativement inactives. Essayez de répartir la charge de manière homogène parmi les partitions. Cela peut nécessiter un hachage des clés de shardlet.
* Si vous géolocalisez des partitions, assurez-vous que la carte des clés hachées renvoie vers les shardlets stockés au sein des partitions stockées à proximité des utilisateurs qui accèdent à ces données.
* Actuellement, seul un jeu limité de types de données SQL est pris en charge en tant que clés de shardlet : *int, bigint, varbinary* et *uniqueidentifier*. Les types SQL *int* et *bigint* correspondent aux types de données *int* et *long* en C# ; ils présentent les mêmes plages. Le type SQL *varbinary* peut être géré à l’aide d’un tableau *Byte* en C# et le type SQL *uniqueidentier* correspond à la classe *Guid* de .NET Framework.

Comme son nom l’indique, la fonctionnalité de base de données élastique permet à un système d’ajouter et de supprimer des partitions à mesure que le volume de données augmente et diminue. Les API de la Bibliothèque cliente Base de données élastique Azure SQL Database permettent à une application de créer et de supprimer des partitions de manière dynamique (et de mettre à jour le gestionnaire des cartes des partitions de manière transparente). Toutefois, la suppression d’une partition est une opération destructive qui nécessite également la suppression de toutes les données présentes dans cette partition.

Si une application doit fractionner une partition en deux partitions distinctes ou combiner des partitions, la fonction Base de données élastique propose un service distinct de fractionnement et de fusion. Ce service s’exécute dans un service hébergé dans le cloud (qui doit être créé par le développeur) et migre les données en toute sécurité entre les partitions. Pour plus d'informations, consultez la rubrique [Mise à l'échelle à l’aide de l'outil de fractionnement et de fusion de bases de données élastiques] sur le site Web de Microsoft.

## <a name="partitioning-strategies-for-azure-storage"></a>Stratégies de partitionnement pour Azure Storage
Le service Stockage Azure fournit quatre abstractions pour la gestion des données :

* Blob Storage stocke les données d’objets non structurées. Un objet blob peut correspondre à n'importe quel type de données texte ou binaires, par exemple, un document, un fichier multimédia ou un programme d'installation d'application. Blob Storage est parfois appelé stockage d’objets.
* Table Storage stocke des jeux de données structurés. Le stockage de tables est un magasin de données de clés-attributs NoSQL qui permet le développement rapide et l'accès rapide à de grosses quantités de données.
* Queue Storage fournit une messagerie fiable pour le traitement des flux de travail et pour la communication entre les composants des services cloud.
* File Storage propose un stockage partagé pour les applications héritées utilisant le protocole SMB standard. Les machines virtuelles et les services cloud Microsoft Azure peuvent partager des données de fichiers entre plusieurs composants d’application grâce à des partages montés. Les applications locales peuvent accéder aux données de fichiers d’un partage via l’API REST du service de stockage de fichiers.

Le stockage de tables et le stockage d’objets blob correspondent essentiellement à des magasins de valeurs de clés optimisés pour le stockage respectif de données structurées et non structurées. Les files d’attente de stockage sont un mécanisme permettant de créer des applications extensibles faiblement couplées. Le stockage de tables, le stockage de fichiers, le stockage blob et les files d’attente de stockage sont créés dans le contexte d’un compte de stockage Azure. Les comptes de stockage prennent en charge trois formes de redondance :

* Le **stockage localement redondant**, qui tient à jour trois copies des données dans un centre de données unique. Cette forme de redondance fournit une protection contre les défaillances matérielles, mais pas contre un incident affectant la totalité du centre de données.
* Le **stockage de zone redondant**, qui tient à jour trois copies des données réparties parmi différents centres de données dans la même région (ou deux régions proches géographiquement). Cette forme de redondance permet une protection contre les incidents se produisant au sein d’un centre de données unique, mais pas contre des déconnexions à grande échelle du réseau qui affectent une région tout entière. Notez que pour l’instant, le stockage redondant dans une zone est uniquement disponible pour les objets blob de blocs.
* Le **stockage géoredondant**, qui tient à jour six copies des données : trois copies dans une région (votre région locale) et trois autres copies dans une région distante. Cette forme de redondance fournit le plus haut niveau de protection contre les incidents.

Microsoft a publié des objectifs d’extensibilité pour Azure Storage. Pour plus d’informations, consultez la page [Azure Storage Scalability and Performance Targets] sur le site web Microsoft. Actuellement, la capacité totale du compte de stockage ne peut pas dépasser 500 To. (Cela inclut la taille des données conservées dans le stockage de tables, le stockage de fichiers et le stockage blob, ainsi que les messages en attente dans la file d’attente de stockage).

Le taux de requêtes maximal d’un compte de stockage (pour une entité, un blob ou un message d’une taille de 1 Ko) est de 20 000 requêtes par seconde. Un compte de stockage présente un maximum de 1 000 IOPS (d’une taille de 8 Ko) par partage de fichiers. Si votre système est susceptible de dépasser ces limites, partitionnez la charge sur plusieurs comptes de stockage. Un même abonnement Azure peut créer jusqu’à 200 comptes de stockage. Notez toutefois que ces limites peuvent évoluer au fil du temps.

## <a name="partitioning-azure-table-storage"></a>Partitionner le stockage de tables Azure
Le stockage de tables Azure est un magasin de clés-valeurs conçue pour le partitionnement. Toutes les entités sont stockées au sein d’une partition et les partitions sont gérées en interne par le stockage de tables Azure. Chaque entité stockée dans une table doit présenter une clé en deux parties comprenant :

* **La clé de partition**. Il s’agit d’une valeur de chaîne qui détermine la partition dans laquelle le stockage de tables Azure placera l’entité. Toutes les entités présentant la même clé de partition sont stockées au sein de la même partition.
* **La clé de ligne**. Il s’agit d’une autre valeur de chaîne qui identifie l’entité au sein de la partition. Toutes les entités présentes au sein d’une partition sont triées par ordre croissant lexical selon cette clé. La combinaison de la clé de partition et de la clé de ligne doit être unique pour chaque entité, et sa longueur ne peut pas dépasser 1 Ko.

Le reste des données relatives à une entité est composé de champs définis par l’application. Aucun schéma particulier n’est appliqué et chaque ligne peut contenir un jeu différent de champs définis par l’application. La seule limite concerne la taille maximale d’une entité (notamment les clés de partition et de ligne), qui est actuellement d’1 Mo. La taille maximale d’une table est de 200 To, bien que ce chiffre puisse évoluer ultérieurement. (Pour obtenir les dernières informations sur ces limites, consultez la page [Azure Storage Scalability and Performance Targets] sur le site web Microsoft.)

Si vous tentez de stocker des entités qui dépassent cette capacité, fractionnez-les en plusieurs tables. Utilisez le partitionnement vertical pour diviser les champs dans les groupes qui sont le plus susceptibles d’être sollicités ensemble.

La figure 7 illustre la structure logique d’un exemple de compte de stockage (données Contoso) pour une application de commerce électronique fictive. Le compte de stockage comprend trois tables : informations relatives aux clients, informations relatives aux produits et informations relatives aux commandes. Chaque table a plusieurs partitions.

Dans la table des informations relatives aux clients, les données sont partitionnées en fonction de la ville dans laquelle se trouve le client et la clé de ligne contient l’ID du client. Dans la table des informations relatives aux produits, les produits sont partitionnés par catégorie et la clé de ligne contient le numéro du produit. Dans la table des informations relatives aux commandes, les commandes sont partitionnées en fonction de la date et la clé de ligne indique l’horaire de réception de la commande. Remarquez que toutes les données sont classées en fonction de la clé de ligne au sein de chaque partition.

![Tables et partitions dans un exemple de compte de stockage](./images/data-partitioning/TableStorage.png)

*Figure 7 : Tables et partitions dans un exemple de compte de stockage*

> [!NOTE]
> Le stockage de tables Azure ajoute également un champ d’horodatage à chaque entité. Le champ d’horodatage est géré par le stockage de tables et actualisé à chaque modification et réinscription de l’entité dans une partition. Le service de stockage de table utilise ce champ pour implémenter l’accès concurrentiel optimiste. Chaque fois qu’une application écrit une entité dans le stockage de table, le service de stockage de table compare la valeur de l’horodatage de l’entité qui est écrite à la valeur contenue dans le stockage de table. Si les valeurs sont différentes, cela signifie qu’une autre application a dû modifier l’entité depuis sa dernière récupération, et l’opération d’écriture échoue. Ne modifiez pas ce champ dans votre propre code et ne spécifiez pas de valeur pour ce champ quand vous créez une entité.
>
>

Le stockage de tables Azure utilise la clé de partition pour déterminer la méthode de stockage des données. Si une entité est ajoutée à une table avec une clé de partition précédemment inutilisée, le stockage de tables Azure crée une partition pour cette entité. Les autres entités présentant la même clé de partition sont stockées au sein de la même partition.

Ce mécanisme met efficacement en œuvre une stratégie de montée en puissance automatique. Chaque partition est stockée sur un serveur unique dans un centre de données Azure pour s’assurer que les requêtes qui récupèrent des données à partir d’une partition unique s’exécutent rapidement. Toutefois, il est possible de répartir différentes partitions sur plusieurs serveurs. En outre, un serveur unique peut héberger plusieurs partitions si ces dernières sont limitées en taille.

Considérez les points suivants quand vous concevez vos entités pour le stockage de tables Azure :

* La sélection des valeurs des clés de partition et des clés de ligne doit être motivée en fonction de la méthode d’accès aux données. Choisissez une combinaison de clé de partition et de clé de ligne qui prend en charge la plupart de vos requêtes. Les requêtes les plus efficaces récupèrent les données en spécifiant la clé de partition et la clé de ligne. Les requêtes qui spécifient une clé de partition et une plage de clés de ligne peuvent être effectuées en analysant une seule partition. Cette opération est relativement rapide, car les données sont conservées par ordre de clé de ligne. Si les requêtes ne spécifient pas la partition à analyser, la clé de partition peut obliger le stockage de tables Azure à analyser chaque partition pour rechercher vos données.

  > [!TIP]
  > Si une entité présente une clé naturelle, utilisez-la alors en tant que clé de partition et spécifiez une chaîne vide en tant que clé de ligne. Si une entité présente une clé composite comprenant deux propriétés, sélectionnez la propriété variable la plus lente en tant que clé de partition et l’autre en tant que clé de ligne. Si une entité présente plus de deux propriétés de clé, utilisez une concaténation des propriétés pour spécifier les clés de partition et de ligne.
  >
  >
* Si vous effectuez régulièrement des requêtes pour rechercher des données à l’aide de champs autres que les clés de partition et de ligne, implémentez le [Index Table Pattern].
* Si vous générez des clés de partition à l’aide d’une séquence incrémentielle croissante ou décroissante unitone (par exemple, « 0001 », « 0002 », « 0003 », et ainsi de suite) et que chaque partition contient uniquement une quantité limitée de données, le stockage de tables Azure peut regrouper physiquement ces partitions sur le même serveur. Ce mécanisme suppose que l’application est plus susceptible d’effectuer des requêtes sur une plage contiguë de partitions (requêtes de plage) et est optimisée pour cet usage. Cependant, cette approche peut conduire à la présence de zones sensibles axées sur un seul serveur, car toutes les insertions de nouvelles entités seront probablement concentrées sur l’une ou l’autre des extrémités des plages contiguës. Elle peut également réduire l’évolutivité. Pour répartir la charge de manière plus homogène entre les serveurs, envisagez de hacher la clé de partition pour rendre la séquence plus aléatoire.
* Le stockage de tables Azure prend en charge les opérations transactionnelles pour les entités appartenant à la même partition. Cela signifie qu’une application peut effectuer plusieurs opérations d’insertion, de mise à jour, de suppression, de remplacement ou de fusion sous forme d’unité atomique (tant que la transaction ne comporte pas plus de 100 entités et que la charge utile de la demande ne dépasse pas 4 Mo). Les opérations qui chevauchent plusieurs partitions ne sont pas transactionnelles et peuvent vous obliger à implémenter la cohérence finale, comme décrit dans les [Data Consistency Primer]. Pour plus d’informations sur le stockage de tables et les transactions, consultez la page [Performing Entity Group Transactions] sur le site web Microsoft.
* Accordez une attention particulière à la précision de la clé de partition, pour les raisons suivantes :
  * L’utilisation de la même clé de partition pour chaque entité fait en sorte que le service de stockage de table crée une seule grande partition sur un seul serveur. Cela empêche la montée en charge et restreint la charge à un seul serveur. Par conséquent, cette approche convient uniquement pour les systèmes qui gèrent un nombre restreint d’entités. Cependant, cette approche garantit que toutes les entités peuvent participer aux transactions relatives à des groupes d’entités.
  * Le fait d’utiliser une clé de partition unique pour chaque entité conduit le service de stockage de tables à créer une partition distincte pour chaque entité, ce qui peut mener à la création d’un grand nombre de petites partitions (en fonction de la taille des entités). Cette approche est plus extensible que l’utilisation d’une clé de partition unique, mais les transactions de groupe d’entités ne sont pas possibles. De plus, les requêtes qui extraient plusieurs entités peuvent impliquer la lecture à partir de plusieurs serveurs. Cependant, si l’application effectue des requêtes de plage, l’utilisation d’une séquence unitone pour générer les clés de partition peut permettre d’optimiser ces requêtes.
  * Le fait de partager la clé de partition au sein d’un sous-ensemble d’entités vous permet de regrouper les entités associées au sein de la même partition. Les opérations qui impliquent des entités associées peuvent être effectuées à l’aide de transactions relatives à des groupes d’entités et les requêtes intervenant sur un jeu d’entités associées peuvent être traitées en accédant à un seul serveur.

Pour plus d’informations sur le partitionnement des données dans le stockage de tables Azure, consultez l’article [Azure Storage Table Design Guide] sur le site web Microsoft.

## <a name="partitioning-azure-blob-storage"></a>Partitionner le stockage d’objets blob Azure 
Grâce au service Stockage Blob Azure, il est possible de stocker des objets binaires volumineux dont la taille peut atteindre jusqu’à 5 To pour les objets blob de blocs ou 1 To pour les objets blob de pages. (Pour obtenir les informations les plus récentes, consultez la page [Azure Storage Scalability and Performance Targets] sur le site web Microsoft.) Utilisez des objets blob de blocs dans des situations telles que la diffusion en continu, lorsque vous devez téléverser ou télécharger rapidement d’importants volumes de données. Utilisez des objets blob de pages pour les applications nécessitant un accès aléatoire plutôt qu’en série à certaines parties des données.

Chaque objet blob (de blocs ou de pages) est stocké au sein d’un conteneur dans un compte de stockage Azure. Vous pouvez utiliser des conteneurs pour regrouper des blobs associés présentant les mêmes exigences de sécurité. Ce regroupement est logique, et non physique. Dans un conteneur, chaque objet blob a un nom unique.

La clé de partition d’un blob est le nom du compte + le nom du conteneur + le nom du blob. Cela signifie que chaque blob peut avoir sa propre partition si la charge sur le blob l’exige. Les blobs peuvent être répartis sur plusieurs serveurs afin d’offrir un accès horizontal. Toutefois, un blob donné ne peut être servi que par un seul serveur. 

Les actions d’écriture d’un seul bloc (objet blob de blocs) ou page (objet blob de pages) sont atomiques, mais pas les opérations intervenant sur des blocs, des pages ou des objets blob. S’il vous faut garantir la cohérence quand vous réalisez des opérations d’écriture entre des blocs, des pages et des objets blob, désactivez un verrou d’écriture à l’aide d’un bail d’objet blob.

Le stockage blob Azure cible des taux de transfert atteignant jusqu’à 60 Mo par seconde ou 500 requêtes par seconde pour chaque blob. Si vous prévoyez de dépasser ces limites et que les données d’objets blob sont relativement statiques, répliquez les objets blob à l’aide du réseau de distribution de contenu Azure. Pour plus d’informations, consultez l’article [Azure Content Delivery Network] du site web de Microsoft. Pour obtenir des instructions et des considérations supplémentaires, consultez l’article [Utilisation d’Azure CDN].

## <a name="partitioning-azure-storage-queues"></a>Partitionner les files d’attente de stockage
Les files d’attente de stockage Azure vous permettent de mettre en œuvre une messagerie asynchrone entre les processus. Un compte de stockage Azure peut contenir un nombre illimité de files d’attente, et chaque file d’attente peut contenir un nombre illimité de messages. La seule limite concerne l’espace disponible dans le compte de stockage. La taille maximale d’un message est de 64 Ko. Si vous devez utiliser des messages dont la taille est supérieure, envisagez plutôt d’utiliser des files d’attente Service Bus Azure.

Chaque file d’attente de stockage a un nom unique au sein du compte de stockage qui la contient. Azure partitionne les files d’attente en fonction du nom. Tous les messages de la même file d’attente sont stockés dans la même partition, qui est contrôlée par un serveur unique. Différentes files d’attente peuvent être gérées par différents serveurs afin d’équilibrer la charge. La répartition des files d’attente entre les serveurs est transparente pour les applications et les utilisateurs.

 Dans une application à grande échelle, n’utilisez pas la même file d’attente de stockage pour toutes les instances de l’application, car cette approche peut transformer le serveur qui héberge la file d’attente en zone sensible. Utilisez plutôt différentes files d’attente pour les différentes zones fonctionnelles de l’application. Les files d’attente de stockage Azure ne prennent pas en charge les transactions. Ainsi, le fait de diriger les messages vers différentes files d’attente ne présente que des répercussions limitées sur la cohérence de la messagerie.

Une file d’attente de stockage Azure peut gérer jusqu’à 2 000 messages par seconde.  Si vous devez traiter les messages à une vitesse supérieure, créez plusieurs files d’attente. Par exemple, dans une application globale, créez des files d’attente de stockage distinctes dans des comptes de stockage distincts pour gérer les instances de l’application en cours d’exécution dans chaque région.

## <a name="partitioning-strategies-for-azure-service-bus"></a>Stratégies de partitionnement pour Azure Service Bus
Azure Service Bus utilise un courtier de messages pour gérer les messages envoyés à une file d’attente ou une rubrique Service Bus. Par défaut, tous les messages envoyés à une file d’attente ou une rubrique sont gérés par le même processus de courtier de messages. Cette architecture peut imposer une limite concernant le débit global de la file d’attente des messages. Toutefois, vous pouvez également partitionner une file d’attente ou une rubrique quand elle est créée. Pour cela, affectez la valeur *true* à la propriété *EnablePartitioning* de la description de file d’attente ou de rubrique.

Une file d’attente ou une rubrique partitionnée est divisée en plusieurs fragments, chacun d’eux étant secondé par une banque de messages et un courtier de messages distincts. Service Bus prend en charge la création et la gestion de ces fragments. Lorsqu’une application publie un message à destination d’une file d’attente ou d’une rubrique partitionnée, Service Bus attribue ce message à un fragment de cette file d’attente ou rubrique. Lorsqu’une application reçoit un message à partir d’une file d’attente ou d’un abonnement, Service Bus vérifie chaque fragment pour identifier le message suivant disponible et le transmet ensuite à l’application pour le traiter.

Cette structure permet de répartir la charge parmi les courtiers de messages et les banques de messages, ce qui améliore l’extensibilité et la disponibilité. Si la banque de messages ou le courtier de messages pour un fragment est temporairement indisponible, Service Bus peut récupérer des messages à partir de l’un des fragments restants disponibles.

Service Bus attribue un message à un fragment comme suit :

* Si le message appartient à une session, tous les messages présentant la même valeur pour la propriété *SessionId* sont envoyés au même fragment.
* Si le message n’appartient pas à une session, mais que l’expéditeur a spécifié une valeur pour la propriété *PartitionKey*, tous les messages ayant la même valeur *PartitionKey* sont envoyés au même fragment.

  > [!NOTE]
  > Si les propriétés *SessionId* et *PartitionKey* sont toutes deux spécifiées, elles doivent être définies sur la même valeur, sinon le message est rejeté.
  >
  >
* Si les propriétés *SessionId* et *PartitionKey* d’un message ne sont pas spécifiées, mais que la détection des doublons est activée, la propriété *MessageId* est utilisée. Tous les messages présentant la même propriété *MessageId* sont dirigés vers le même fragment.
* Si des messages ne présentent aucune propriété *SessionId, PartitionKey* ou *MessageId*, Service Bus les attribue aux fragments de manière séquentielle. Si un fragment n’est pas disponible, Service Bus passe au suivant. Ainsi, une défaillance temporaire de l’infrastructure de messagerie n’entraîne pas de défaillance de l’opération d’envoi des messages.

Considérez les points suivants quand vous décidez ou non de partitionner une rubrique ou une file d’attente de messages Service Bus, et à l’aide de quelle méthode :

* Les rubriques et files d’attente Service Bus sont créées dans l’étendue d’un espace de noms Service Bus. Service Bus permet actuellement de disposer jusqu’à 100 rubriques ou files d’attente par espace de noms.
* Chaque espace de noms Service Bus impose des quotas sur les ressources disponibles, telles que le nombre d’abonnements par rubrique, le nombre d’envois et de réceptions simultanés de demandes par seconde, et le nombre maximal de connexions simultanées pouvant être établies. Ces quotas sont documentés sur la page [Quotas Service Bus]sur le site Web de Microsoft. Si vous pensez dépasser ces valeurs, créez des espaces de nom supplémentaires disposant de leurs propres rubriques et files d’attente, puis répartissez le travail entre ces espaces de nom. Par exemple, au sein d’une application globale, créez des espaces de noms distincts dans chaque région et configurez les instances applicatives pour utiliser les rubriques et les files d’attente présentes au sein de l’espace de nom le plus proche.
* Les messages envoyés dans le cadre d’une transaction doivent spécifier une clé de partition. Il peut s’agir d’une propriété *SessionId*, *PartitionKey* ou *MessageId*. Tous les messages envoyés dans le cadre de la même transaction doivent spécifier la même clé de partition, car ils doivent être gérés par le même processus Broker de messages. Vous ne pouvez pas envoyer des messages à différentes files d’attente ou rubriques au sein de la même transaction.
* Vous ne pouvez pas configurer les rubriques et les files d’attente partitionnées pour qu’elles soient supprimées automatiquement quand elles deviennent inactives.
* Actuellement, vous ne pouvez pas utiliser des files d’attente et des rubriques partitionnées avec le protocole AMQP (Advanced Message Queueing Protocol) si vous créez des solutions interplateformes ou hybrides.

## <a name="partitioning-strategies-for-cosmos-db"></a>Stratégies de partitionnement pour Cosmos DB

Azure Cosmos DB est une base de données NoSQL pouvant stocker des documents JSON à l’aide de [l’API SQL Azure Cosmos DB][cosmosdb-sql-api]. Un document de base de données Cosmos DB correspond à une représentation sérialisée JSON d’un objet ou autre élément de données. Aucun schéma fixe n’est appliqué, mais chaque document doit contenir un ID unique.

Les documents sont organisés en collections. Vous pouvez regrouper des documents connexes dans une collection. Par exemple, dans un système gérant des publications de blog, vous pouvez stocker le contenu de chaque publication de blog sous forme de document dans une collection. Vous pouvez également créer des collections pour chaque type d’objet. Sinon, dans une application mutualisée (telle qu’un système dans lequel différents auteurs contrôlent et gèrent leurs propres publications de blog), vous pouvez partitionner les blogs par auteur et créer des collections distinctes pour chaque auteur. L’espace de stockage alloué aux collections est flexible et peut évoluer à la hausse ou à la baisse en fonction des besoins.

Cosmos DB prend en charge le partitionnement automatique des données d’après une clé de partition définie au niveau de l’application. Une *partition logique* est une partition qui stocke toutes les données associées à une valeur de clé de partition unique. Tous les documents qui partagent la même valeur de clé de partition sont placés dans la même partition logique. Cosmos DB distribue les valeurs en fonction du hachage de la clé de partition. Une partition logique présente une taille maximale de 10 Go. Pour cette raison, le choix de la clé de partition est une décision importante que vous devrez prendre au moment de la conception. Choisissez une propriété avec une large gamme de valeurs, voire des modèles d’accès. Pour en savoir plus, voir [Partitionner et mettre à l’échelle dans Azure Cosmos DB](/azure/cosmos-db/partition-data).

> [!NOTE]
> Chaque base de données Cosmos DB présente un *niveau de performances* qui détermine la quantité de ressources obtenue. À chaque niveau de performance est associé un taux limite d’*unités de demande*. Le taux limite d’unités de demande spécifie le volume de ressources réservé et disponible exclusivement pour cette collection. Le coût d’une collection dépend du niveau de performances sélectionné pour cette collection. Plus le niveau de performances (et le taux limite d’unités de demande) est élevé, plus la charge est élevée. Vous pouvez régler le niveau de performance d’une collection à l’aide du portail Azure. Pour en savoir plus, voir [Unités de requête dans Azure Cosmos DB][cosmos-db-ru].
>
>

Si le mécanisme de partitionnement Cosmos DB offre n’est pas suffisant, vous devrez peut-être partitionner les données au niveau de l’application. Les collections de documents fournissent un mécanisme naturel permettant de partitionner des données dans une base de données unique. La méthode la plus simple permettant de mettre en œuvre le partitionnement consiste à créer une collection pour chaque partition. Les conteneurs sont des ressources logiques. Ils peuvent s’étendre sur un ou plusieurs serveurs. Les conteneurs de taille fixe ont une limite maximale de 10 Go et de 10 000 RU/s de débit. Un nombre illimité de conteneurs ne présente aucune taille de stockage maximale, mais doit spécifier une clé de partition. Il incombe à l’application cliente d’envoyer les requêtes vers la partition appropriée, généralement en implémentant son propre mécanisme de mappage en fonction de certains attributs des données qui définissent la clé de partitionnement. 

Toutes les bases de données sont créées dans le contexte d’un compte de base de données Cosmos DB. Un même compte peut contenir plusieurs bases de données et indique les régions dans lesquelles les bases de données sont créées. Chaque compte applique également son propre contrôle d’accès. Vous pouvez utiliser des comptes Cosmos DB pour géolocaliser les partitions (collections au sein des bases de données) situées à proximité des utilisateurs ayant besoin d’y accéder, et appliquer des restrictions afin que seuls ces utilisateurs puissent s’y connecter.

Lorsque vous décidez de la procédure à suivre pour partitionner des données à l’aide de l’API SQL Cosmos DB, tenez compte des points suivants :

* **Les ressources disponibles pour une base de données Cosmos DB sont soumises aux limites de quota du compte**. Chaque base de données peut contenir un certain nombre de collections, et chaque collection est associée à un niveau de performances qui régit le taux limite d’unités de demande (débit réservé) pour cette collection. Pour plus d’informations, consultez [Abonnement Azure et limites, quotas et contraintes de service][azure-limits].
* **Chaque document doit avoir un attribut pouvant être utilisé pour identifier de manière unique ce document dans la collection dans laquelle il est stocké**. Cet attribut est différent de la clé de partition, qui définit la collection dans laquelle le document est stocké. Une collection peut contenir un grand nombre de documents. En théorie, ce nombre est limité uniquement par la longueur maximale de l’ID de document. L’ID du document peut comprendre jusqu’à 255 caractères.
* **Toutes les opérations sur un document sont exécutées dans le contexte d’une transaction. Les transactions se limitent à la collection dans laquelle se trouve le document.** Si une opération échoue, la tâche réalisée est annulée. Quand un document est sujet à une opération, toutes les modifications apportées sont soumises à un isolement de niveau capture instantanée. Ce mécanisme garantit que si, par exemple, une requête visant à créer un document échoue, un autre utilisateur qui interroge la base de données simultanément ne verra pas un document partiel supprimé par la suite.
* **Les requêtes de base de données se limitent également au niveau de la collection**. Une seule requête peut récupérer les données issues d’une seule collection. Si vous devez récupérer des données issues de plusieurs collections, vous devez interroger chaque collection individuellement et fusionner les résultats dans votre code d’application.
* **Cosmos DB prend en charge les éléments programmables qui peuvent tous être stockés dans une collection avec des documents**. Il s’agit notamment des procédures stockées, des fonctions définies par l’utilisateur et des déclencheurs (écrits en JavaScript). Ces éléments peuvent accéder à n’importe quel document au sein de la même collection. En outre, ces éléments s’exécutent soit dans le cadre de la portée de la transaction actuelle (dans le cas d’un déclencheur intervenant suite à une opération de création, de suppression ou de remplacement effectuée sur un document), soit en démarrant une nouvelle transaction (dans le cas d’une procédure stockée exécutée suite à une demande client explicite). Si le code d’un élément programmable lève une exception, la transaction est annulée. Vous pouvez utiliser des procédures stockées et des déclencheurs pour maintenir l’intégrité et la cohérence entre des documents, mais ces documents doivent tous faire partie de la même collection.
* **Les collections que vous souhaitez conserver dans les bases de données doivent être peu susceptibles de dépasser les limites de débit définies par les niveaux de performances des collections**. Pour en savoir plus, voir [Unités de requête dans Azure Cosmos DB][cosmos-db-ru]. Si vous prévoyez d’atteindre ces limites, envisagez de fractionner les collections des bases de données entre différents comptes afin de réduire la charge par collection.

## <a name="partitioning-strategies-for-azure-search"></a>Stratégies de partitionnement pour Azure Search
La capacité à rechercher des données est souvent la méthode principale de navigation et d’exploration fournie par de nombreuses applications web. Elle permet aux utilisateurs de trouver des ressources rapidement (par exemple, des produits dans une application de commerce électronique) en fonction de combinaisons de critères de recherche. Le service de recherche Azure Search propose des fonctionnalités de recherche en texte intégral dans le contenu Web, ainsi que des fonctionnalités telles que les requêtes prédictives, les requêtes suggérées en fonction des correspondances suivantes et la navigation à facettes. Une description complète de ces fonctionnalités est disponible dans la page [Présentation d’Azure Storage] sur le site web de Microsoft.

Azure Search stocke le contenu pouvant faire l’objet d’une recherche sous forme de documents JSON dans une base de données. Vous définissez des index qui spécifient les champs de recherche dans ces documents et fournissez ces définitions à Azure Search. Quand un utilisateur soumet une demande de recherche, Azure Search utilise les index appropriés pour trouver les éléments correspondants.

Pour réduire la contention, le stockage utilisé par Azure Search peut être divisé en 1, 2, 3, 4, 6, ou 12 partitions, et chaque partition peut être répliquée jusqu’à 6 fois. Le produit du nombre de partitions multiplié par le nombre de réplicas est appelé *unité de recherche*. Une même instance d’Azure Search peut contenir au maximum 36 unités de recherche (une base de données avec 12 partitions prend en charge 3 réplicas au maximum).

La facturation se fait en fonction de chaque unité de recherche allouée à votre service. À mesure que le volume de contenu pouvant faire l’objet d’une recherche ou que le taux des demandes de recherche augmente, vous pouvez ajouter des unités de recherche à une instance existante d’Azure Search pour gérer la charge supplémentaire. Azure Search distribue uniformément les documents parmi les partitions. Aucune stratégie de partitionnement manuel n’est actuellement prise en charge.

Chaque partition peut contenir un maximum de 15 millions de documents ou occuper 300 Go d’espace de stockage (la valeur la plus petite étant retenue). Vous pouvez créer jusqu’à 50 index. Les performances du service varient et dépendent de la complexité des documents, des index disponibles et de la latence du réseau. En moyenne, un réplica (1 unité de recherche) doit pouvoir gérer 15 requêtes par seconde, même s’il convient de réaliser une mesure à l’aide de vos propres données pour obtenir un résultat plus précis du débit. Pour plus d’informations, consultez la page [Limites de service d’Azure Search] sur le site web de Microsoft.

> [!NOTE]
> Vous pouvez stocker un jeu de types de données limité dans les documents pouvant faire l’objet d’une recherche, notamment des chaînes, des valeurs booléennes, des données numériques, des données d’horodatage et certaines données géographiques. Pour plus d’informations, consultez la page [Types de données pris en charge (Azure Search)] sur le site web de Microsoft.
>
>

Vous disposez d’un contrôle limité sur la manière dont Azure Search partitionne les données pour chaque instance du service. Cependant, dans un environnement global, vous pouvez améliorer les performances et réduire la latence et la contention en partitionnant le service proprement dit en appliquant l’une des stratégies suivantes :

* Créez une instance d’Azure Search dans chaque région géographique et vérifiez que les applications clientes sont dirigées vers l’instance disponible la plus proche. Cette stratégie exige la réplication de chaque mise à jour du contenu pouvant faire l’objet d’une recherche en temps voulu au sein de toutes les instances du service.
* Créez deux niveaux d’Azure Search :

  * Un service local dans chaque région qui contient les données les plus fréquemment sollicitées par les utilisateurs dans cette région. Les utilisateurs peuvent diriger les demandes ici pour obtenir des résultats rapides, mais limités.
  * Un service global qui englobe toutes les données. Les utilisateurs peuvent diriger les demandes ici pour obtenir des résultats plus lents, mais plus complets.

Cette approche est plus appropriée quand les données faisant l’objet d’une recherche présentent des variations régionales significatives.

## <a name="partitioning-strategies-for-azure-redis-cache"></a>Stratégies de partitionnement pour le cache Redis Azure
Le cache Redis Azure propose un service de mise en cache partagé dans le cloud basé sur le magasin de données de clés et de valeurs Redis. Comme son nom l’indique, le cache Redis Azure est conçu comme une solution de mise en cache. Utilisez-le uniquement pour contenir des données temporaires, et non comme un magasin de données permanent. Les applications qui utilisent le cache Redis Azure doivent pouvoir continuer à fonctionner si le cache n’est pas disponible. Le cache Redis Azure prend en charge la réplication principale/secondaire pour fournir une haute disponibilité, mais le service limite actuellement la taille maximale du cache à 53 Go. Si vous devez disposer de davantage d’espace, vous devez créer des caches supplémentaires. Pour plus d’informations, consultez la page [Cache Redis Azure] sur le site web Microsoft.

Le partitionnement d’un magasin de données Redis implique la répartition des données entre des instances du service Redis. Chaque instance constitue une partition unique. Le cache Redis Azure fait abstraction des services Redis derrière une façade et ne les expose pas directement. La méthode la plus simple pour implémenter le partitionnement consiste à créer plusieurs instances de cache Redis Azure et à répartir les données entre elles.

Vous pouvez associer chaque élément de données à un identificateur (une clé de partition) qui indique dans quel cache l’élément est stocké. La logique de l’application cliente peut ensuite utiliser cet identificateur pour acheminer les demandes vers la partition appropriée. Ce schéma est très simple mais, si le schéma de partitionnement change (par exemple si d’autres instances de cache Redis Azure sont créées), vous devrez peut-être reconfigurer les applications clientes.

Le cache Redis natif (pas le cache Redis Azure) prend en charge le partitionnement côté serveur basé sur le clustering Redis. Avec cette approche, vous pouvez répartir les données de manière égale parmi les serveurs à l’aide d’un mécanisme de hachage. Chaque serveur Redis stocke les métadonnées décrivant la plage de clés de hachage stockées dans la partition et contient également les informations relatives aux clés de hachage présentes au sein de partitions présentes sur d’autres serveurs.

Les applications clientes envoient simplement les demandes à l’un des serveurs Redis participants (probablement le plus proche). Le serveur Redis examine la demande du client. Si elle peut être résolue localement, il effectue l’opération demandée. Dans le cas contraire, il transfère la demande au serveur approprié.

Ce modèle est mis en œuvre à l’aide du clustering Redis et est décrit plus en détail sur la page [Redis cluster tutorial] sur le site web de Redis. Le clustering Redis est transparent pour les applications clientes. Vous pouvez ajouter des serveurs Redis supplémentaires au cluster (et repartitionner les données) sans avoir à reconfigurer les clients.

> [!IMPORTANT]
> Le cache Redis Azure ne prend pas en charge le clustering Redis. Si vous souhaitez implémenter cette approche avec Azure, vous devez implémenter vos propres serveurs Redis en installant Redis sur un ensemble de machines virtuelles Azure et en les configurant manuellement. Le billet de blog [Running Redis on a CentOS Linux VM in Windows Azure] sur le site web de Microsoft présente un exemple décrivant comment créer et configurer un nœud Redis exécuté en tant que machine virtuelle Azure.
>
>

La page [Partitioning: how to split data among multiple Redis instances] sur le site web de Redis fournit des informations supplémentaires sur l’implémentation du partitionnement avec Redis. Le reste de cette section part du principe que vous mettez en œuvre le partitionnement côté client ou assisté par proxy.

Considérez les points suivants quand vous décidez de partitionner des données avec le cache Redis Azure :

* Le cache Redis Azure n’est pas destiné à fonctionner en tant que magasin de données permanent. Ainsi, quel que soit le schéma de partitionnement que vous implémentez, votre code d’application doit pouvoir récupérer les données à partir d’un emplacement autre que le cache.
* Les données fréquemment sollicitées doivent être conservées dans la même partition. Redis est un magasin de clés/valeurs puissant qui offre plusieurs mécanismes hautement optimisés pour structurer les données. Il peut s’agir des mécanismes suivants :

  * Chaînes simples (données binaires d’une longueur allant jusqu’à 512 Mo)
  * Types d’agrégation comme des listes (qui peuvent servir de files d’attente et de piles)
  * Ensembles (triés et non triés)
  * Hachages (qui peuvent regrouper des champs associés, tels que des éléments qui représentent les champs d’un objet)
* Les types d’agrégation permettent d’associer de nombreuses valeurs connexes avec la même clé. Une clé Redis identifie une liste, un ensemble ou un hachage plutôt que les éléments de données qu’elle contient. Ces types sont tous disponibles avec le cache Redis Azure et sont décrits dans la page [Data types] sur le site web de Redis. Par exemple, dans le cadre d’un système de commerce électronique qui assure le suivi des commandes passées par les clients, les détails de chaque client peuvent être stockés dans un hachage Redis indexé à l’aide de l’ID du client. Chaque hachage peut contenir une collection d’ID de commande relatifs au client. Un jeu Redis distinct peut contenir les commandes, à nouveau structurées sous forme de hachages, indexées à l’aide de l’ID de la commande. La figure 8 illustre cette structure. Remarquez que Redis ne met pas en œuvre toutes les formes d’intégrité référentielle ; il incombe au développeur de maintenir les relations entre les clients et commandes.

![Structure suggérée au sein du stockage Redis pour enregistrer les commandes des clients et les détails associés](./images/data-partitioning/RedisCustomersandOrders.png)

*Figure 8 : Structure suggérée au sein du stockage Redis pour enregistrer les commandes des clients et les détails associés*

> [!NOTE]
> Dans Redis, toutes les clés sont des valeurs de données binaires (telles que des chaînes Redis) pouvant contenir jusqu’à 512 Mo de données. En théorie, une clé peut contenir pratiquement n’importe quelle information. Cependant, nous vous recommandons d’adopter une convention d’affectation de noms cohérente concernant les clés et permettant de décrire le type de données et d’identifier l’entité, sans pour autant que le nom soit trop long. Une approche courante consiste à utiliser des clés au format « type_entité:ID ». Par exemple, vous pouvez utiliser « client:99 » pour indiquer la clé pour un client avec l’ID 99.
>
>

* Vous pouvez mettre en œuvre le partitionnement vertical en stockant les informations connexes dans des agrégations différentes au sein de la même base de données. Par exemple, dans une application de commerce électronique, vous pouvez stocker les informations relatives aux produits fréquemment sollicitées dans une table de hachage Redis et les informations les moins fréquemment sollicitées dans une autre.
  Les deux hachages peuvent utiliser le même ID de produit dans le cadre de la clé. Par exemple, vous pouvez utiliser « product: *nn* » (où *nn* correspond à l’ID du produit) pour les informations sur le produit et « product_details:*nn* » pour les informations détaillées. Cette stratégie favorise la réduction du volume de données que la plupart des requêtes sont susceptibles de récupérer.
* Vous pouvez repartitionner un magasin de données Redis, mais sachez qu’il s’agit d’une tâche longue et complexe. Le clustering Redis peut repartitionner automatiquement les données, mais cette capacité n’est pas disponible avec le cache Redis Azure. Ainsi, quand vous concevez votre schéma de partitionnement, essayez de laisser suffisamment d’espace libre dans chaque partition pour permettre l’augmentation attendue du volume de données au fil du temps. Cependant, n’oubliez pas que le cache Redis Azure est conçu pour mettre temporairement des données en cache et que les données stockées dans le cache peuvent présenter une durée de vie limitée, spécifiée en tant que valeur de durée de vie. Concernant les données relativement volatiles, la durée de vie peut être courte. En revanche, concernant les données statiques, la durée de vie peut être beaucoup plus longue. Évitez de stocker de grandes quantités de données durables dans le cache si le volume de ces données est susceptible de remplir le cache. Vous pouvez spécifier une stratégie d’éviction permettant au cache Redis Azure de supprimer les données si l’espace est limité.

  > [!NOTE]
  > Le cache Redis Azure vous permet de spécifier la taille maximale du cache (de 250 Mo à 53 Go) en sélectionnant le niveau tarifaire approprié. Cependant, après avoir créé un cache Redis Azure, vous ne pouvez plus en augmenter ni en réduire la taille.
  >
  >
* Les lots et transactions Redis ne peuvent pas couvrir plusieurs connexions. Ainsi, toutes les données affectées par un lot ou une transaction doivent être stockées dans la même base de données (partition).

  > [!NOTE]
  > Une séquence d’opérations relative à une transaction Redis n’est pas nécessairement atomique. Les commandes qui composent une transaction sont vérifiées et mises en file d’attente avant d’être exécutées. Si une erreur se produit pendant cette phase, la file d’attente entière est ignorée. Cependant, une fois la transaction envoyée avec succès, les commandes placées en file d’attente sont exécutées en séquence. Si une commande échoue, seule cette commande cesse de s’exécuter. Toutes les commandes précédentes et suivantes dans la file d’attente sont exécutées. Pour plus d’informations, consultez la page [Transactions] sur le site web de Redis.
  >
  >
* Redis prend en charge un nombre limité d’opérations atomiques. Les seules opérations de ce type qui prennent en charge plusieurs clés et valeurs sont les opérations MGET et MSET. Les opérations MGET retournent une collection de valeurs pour une liste de clés spécifiée, et les opérations MSET stockent une collection de valeurs pour une liste de clés spécifiée. Si vous devez utiliser ces opérations, les paires clés/valeurs référencées par les commandes MSET et MGET doivent être stockées dans la même base de données.

## <a name="partitioning-strategies-for-azure-service-fabric"></a>Stratégies de partitionnement pour Azure Service Fabric
Azure Service Fabric est une plateforme de microservices qui fournit un runtime pour les applications distribuées dans le cloud. Service Fabric prend en charge les exécutables invités .Net, les services avec état et sans état, ainsi que les conteneurs. Les services avec état fournissent une [collection fiable][service-fabric-reliable-collections] pour stocker les données de manière permanente dans une collection clé-valeur au sein du cluster Service Fabric. Pour plus d’informations sur les stratégies relatives aux clés de partitionnement dans une collection fiable, consultez l’article [Instructions et recommandations pour les collections fiables dans Azure Service Fabric].

### <a name="more-information"></a>Plus d’informations
* L’article [Vue d’ensemble d’Azure Service Fabric] constitue une présentation d’Azure Service Fabric.
* L’article [Partitionnement des services fiables Service Fabric] fournit des informations supplémentaires sur les services fiables dans Azure Service Fabric.

## <a name="partitioning-strategies-for-azure-event-hubs"></a>Stratégies de partitionnement pour Azure Event Hubs

[Azure Event Hubs][event-hubs] est conçu pour la diffusion de données en continu à grande échelle, et le partitionnement est intégré au service pour permettre une mise à l’échelle horizontale. Chaque consommateur lit uniquement une partition spécifique du flux de messages. 

L’éditeur d’événements est uniquement informé de sa clé de partition, et non de la partition sur laquelle les événements sont publiés. Grâce à cette dissociation de la clé et de la partition, l’expéditeur n’a pas besoin de connaître trop d’informations sur le traitement en aval. (Il est également possible d’envoyer des événements directement à une partition spécifique, mais cette opération est généralement déconseillée.)  

Lorsque vous sélectionnez le nombre de partitions, tenez compte de la mise à l’échelle sur le long terme. Après la création d’un Event Hub, vous ne pourrez plus modifier le nombre de partitions. 

Pour plus d’informations sur l’utilisation de partitions dans Event Hubs, consultez l’article [Qu’est-ce qu’Event Hubs ?].

Pour découvrir des considérations relatives aux compromis entre la disponibilité et la cohérence, consultez l’article [Disponibilité et cohérence dans Event Hubs].

## <a name="rebalancing-partitions"></a>Rééquilibrer les partitions
À mesure que le système évolue et que vous comprenez mieux les modèles d’utilisation, vous devrez peut-être ajuster le schéma de partitionnement. Par exemple, des partitions spécifiques peuvent commencer à concentrer un volume de trafic disproportionné et devenir sensibles, provoquant une contention excessive. De plus, il se peut que vous ayez sous-évalué le volume des données stockées dans certaines partitions et que celles-ci atteignent leurs limites de capacité de stockage. Quelle que soit la cause, il est parfois nécessaire de rééquilibrer des partitions pour répartir la charge de manière plus homogène.

Dans certains cas, les systèmes de stockage de données qui n’indiquent pas explicitement comment les données sont allouées aux serveurs peuvent rééquilibrer automatiquement les partitions dans les limites des ressources disponibles. Dans d’autres cas, le rééquilibrage correspond à une tâche administrative qui se compose de deux étapes :

1. Détermination de la nouvelle stratégie de partitionnement pour identifier :
   * Les partitions qu’il faudrait fractionner (ou éventuellement combiner)
   * Comment allouer les données à ces nouvelles partitions en créant de nouvelles clés de partition.
2. Migrer les données concernées à partir de l’ancien schéma de partitionnement vers le nouveau jeu de partitions.

> [!NOTE]
> Le mappage des collections de bases de données sur les serveurs est transparent, mais il se peut que vous atteigniez tout de même les limites de capacité de stockage et de débit d’un compte Cosmos DB. Dans ce cas, vous devrez peut-être revoir votre schéma de partitionnement et migrer les données.
>
>

En fonction de la technologie de stockage de données et de la conception de votre système de stockage, vous pouvez éventuellement migrer des données entre des partitions alors qu’elles sont en cours d’utilisation (migration en ligne). Si ce n’est pas possible, vous devrez peut-être rendre les partitions concernées temporairement indisponibles pendant la migration des données (migration hors connexion).

## <a name="offline-migration"></a>Migration hors connexion
La migration hors connexion est sans doute l’approche la plus simple, car elle réduit les risques de contention. N’apportez aucune modification aux données pendant leur déplacement et leur restructuration.

Sur le plan conceptuel, ce processus comprend les étapes suivantes :

1. Mettre la partition hors connexion
2. Fractionner/fusionner et déplacer les données vers les nouvelles partitions
3. Vérifier les données
4. Mettre les nouvelles partitions en ligne
5. Supprimer l’ancienne partition

Pour conserver une certaine disponibilité, vous pouvez mettre la partition d’origine en lecture seule à l’étape 1 au lieu de la rendre indisponible. Cela permet aux applications de lire les données pendant leur déplacement, sans pouvoir les modifier.

## <a name="online-migration"></a>Migration en ligne
La migration en ligne est plus complexe à réaliser, mais entraîne moins de perturbations pour les utilisateurs, car les données restent disponibles pendant toute la procédure. Le processus est similaire à celui utilisé pour la migration hors connexion, à ceci près que la partition d’origine n’est pas mise hors connexion (étape 1). En fonction de la précision du processus de migration (par exemple, élément par élément ou partition par partition), le code d’accès aux données présent dans les applications clientes devra peut-être gérer la lecture et l’écriture des données stockées dans deux emplacements (la partition d’origine et la nouvelle partition).

Pour obtenir un exemple de solution prenant en charge la migration en ligne, consultez la page [Mise à l'échelle à l’aide de l'outil de fractionnement et de fusion de bases de données élastiques] sur le site web de Microsoft.

## <a name="related-patterns-and-guidance"></a>Conseils et modèles connexes
Quand vous étudiez des stratégies d’implémentation de la cohérence des données, les modèles suivants peuvent également s’appliquer à votre cas :

* La page [Data Consistency Primer] sur le site web Microsoft décrit les stratégies de maintien de la cohérence dans un environnement distribué tel que le cloud.
* La page [Data Partitioning Guidance] sur le site web Microsoft présente une vue d’ensemble de la conception de partitions visant à satisfaire divers critères dans une solution distribuée.
* Le [Modèle de partitionnement] tel que décrit sur le site web Microsoft récapitule certaines stratégies courantes en matière de partitionnement de données.
* Le [Index Table Pattern] tel que décrit sur le site web Microsoft illustre comment créer des index secondaires pour les données. Cette approche permet à une application de récupérer rapidement des données à l’aide de requêtes ne faisant pas référence à la clé primaire d’une collection.
* Le [Materialized View Pattern] tel que décrit sur le site web Microsoft illustre comment générer des affichages prédéfinis qui synthétisent les données pour prendre en charge des opérations de requêtes rapides. Cette approche peut s’avérer utile au sein d’un magasin de données partitionnées si les partitions contenant les données synthétisées sont réparties sur plusieurs sites.
* L’article [Utilisation d’Azure CDN] sur le site web Microsoft fournit des conseils supplémentaires sur la configuration et l’utilisation du réseau de distribution de contenu avec Azure.

## <a name="more-information"></a>Plus d’informations
* La page [Qu’est-ce qu’Azure SQL Database ?] sur le site web Microsoft présente des informations détaillées sur la création et l’utilisation des bases de données SQL.
* La page [Vue d’ensemble des fonctionnalités de bases de données élastiques] sur le site web Microsoft dresse une présentation complète de la fonction Base de données élastique.
* La page [Mise à l'échelle à l’aide de l'outil de fractionnement et de fusion de bases de données élastiques] sur le site web Microsoft présente des informations sur l’utilisation du service de fractionnement et de fusion pour gérer les partitions de bases de données élastiques.
* La page [Objectifs de performance et évolutivité d’Azure Storage](https://msdn.microsoft.com/library/azure/dn249410.aspx) sur le site web Microsoft présente les limites actuelles en matière de dimensionnement et de débit d’Azure Storage.
* La page [Performing entity group transactions] sur le site web Microsoft fournit des informations détaillées sur l’implémentation des opérations transactionnelles relatives à des entités stockées dans le service de stockage de tables Azure.
* L’article [Azure Storage Table Design Guide] sur le site web Microsoft fournit des informations détaillées sur le partitionnement des données dans le service de stockage de tables Azure.
* La page [Utilisation d’Azure CDN] sur le site web Microsoft décrit comment répliquer des données contenues dans le service de stockage d’objets blob Azure à l’aide du réseau de distribution de contenu Azure.
* La page [Présentation d’Azure Storage] sur le site web Microsoft présente une description complète des fonctionnalités disponibles dans Azure Search.
* La page [Limites de service d’Azure Search] sur le site web Microsoft fournit des informations sur la capacité de chaque instance d’Azure Search.
* La page [Types de données pris en charge (Azure Search)] sur le site web Microsoft présente une synthèse des types de données que vous pouvez utiliser dans les documents et les index pouvant faire l’objet d’une recherche.
* La page [Cache Redis Azure] sur le site web Microsoft présente le cache Redis Azure.
* La page [Partitioning: how to split data among multiple Redis instances] sur le site web de Redis fournit des informations sur la façon d’implémenter le partitionnement avec Redis.
* Le billet de blog [Running Redis on a CentOS Linux VM in Windows Azure] sur le site web de Microsoft présente un exemple décrivant comment créer et configurer un nœud Redis exécuté en tant que machine virtuelle Azure.
* La page [Data Types] sur le site web de Redis présente les types de données disponibles avec Redis et le cache Redis Azure.

[Disponibilité et cohérence dans Event Hubs]: /azure/event-hubs/event-hubs-availability-and-consistency
[azure-limits]: /azure/azure-subscription-service-limits
[Azure Content Delivery Network]: /azure/cdn/cdn-overview
[Cache Redis Azure]: https://azure.microsoft.com/services/cache/
[Azure Storage Scalability and Performance Targets]: /azure/storage/storage-scalability-targets
[Azure Storage Table Design Guide]: /azure/storage/storage-table-design-guide
[Building a Polyglot Solution]: https://msdn.microsoft.com/library/dn313279.aspx (Création d’une solution polyglotte)
[cosmos-db-ru]: /azure/cosmos-db/request-units
[Data Access for Highly-Scalable Solutions: Using SQL, NoSQL, and Polyglot Persistence]: https://msdn.microsoft.com/library/dn271399.aspx (Accès aux données pour les solutions hautement évolutives : Utilisation de la persistance SQL, NoSQL et polyglotte)
[Data Consistency Primer]: https://aka.ms/Data-Consistency-Primer
[Data Partitioning Guidance]: https://msdn.microsoft.com/library/dn589795.aspx
[Data Types]: https://redis.io/topics/data-types
[cosmosdb-sql-api]: /azure/cosmos-db/sql-api-introduction
[Vue d’ensemble des fonctionnalités de bases de données élastiques]: /azure/sql-database/sql-database-elastic-scale-introduction
[event-hubs]: /azure/event-hubs
[Federations Migration Utility]: https://code.msdn.microsoft.com/vstudio/Federations-Migration-ce61e9c1
[Instructions et recommandations pour les collections fiables dans Azure Service Fabric]: /azure/service-fabric/service-fabric-reliable-services-reliable-collections-guidelines
[Index Table Pattern]: ../patterns/index-table.md
[Materialized View Pattern]: ../patterns/materialized-view.md
[Requête sur plusieurs partitions]: /azure/sql-database/sql-database-elastic-scale-multishard-querying
[Vue d’ensemble d’Azure Service Fabric]: /azure/service-fabric/service-fabric-overview
[Partitionnement des services fiables Service Fabric]: /azure/service-fabric/service-fabric-concepts-partitioning
[Partitioning: how to split data among multiple Redis instances]: https://redis.io/topics/partitioning
[Performing Entity Group Transactions]: https://msdn.microsoft.com/library/azure/dd894038.aspx
[Redis cluster tutorial]: https://redis.io/topics/cluster-tutorial
[Running Redis on a CentOS Linux VM in Windows Azure]: https://blogs.msdn.microsoft.com/tconte/2012/06/08/running-redis-on-a-centos-linux-vm-in-windows-azure/
[Mise à l'échelle à l’aide de l'outil de fractionnement et de fusion de bases de données élastiques]: /azure/sql-database/sql-database-elastic-scale-overview-split-and-merge
[Utilisation d’Azure CDN]: /azure/cdn/cdn-create-new-endpoint
[Quotas Service Bus]: /azure/service-bus-messaging/service-bus-quotas
[service-fabric-reliable-collections]: /azure/service-fabric/service-fabric-reliable-services-reliable-collections
[Limites de service d’Azure Search]:  /azure/search/search-limits-quotas-capacity
[Modèle de partitionnement]: ../patterns/sharding.md
[Types de données pris en charge (Azure Search)]:  https://msdn.microsoft.com/library/azure/dn798938.aspx
[Transactions]: https://redis.io/topics/transactions
[Qu’est-ce qu’Event Hubs ?]: /azure/event-hubs/event-hubs-what-is-event-hubs
[Présentation d’Azure Storage]: /azure/search/search-what-is-azure-search
[Qu’est-ce qu’Azure SQL Database ?]: /azure/sql-database/sql-database-technical-overview
