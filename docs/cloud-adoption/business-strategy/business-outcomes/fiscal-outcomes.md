---
title: 'Framework d’adoption du cloud : Exemples de résultats comptables'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
description: Exemples de résultats comptables
author: BrianBlanchard
ms.date: 10/10/2018
ms.topic: guide
ms.openlocfilehash: 5b28e22fca294bd1311d29ee5f70ca45e70fae3e
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640224"
---
# <a name="examples-of-fiscal-outcomes"></a>Exemples de résultats comptables

Au niveau le plus haut, les discussions en matière de comptabilité portent sur trois concepts de base :

* Chiffre d’affaires : L’entreprise gagnera-t-elle plus d’argent suite à la vente de biens ou de services ?
* Coût : Est-ce que moins d’argent sera consacré à la création, au marketing, à la vente ou à la livraison de biens ou de services ?
* Bénéfice : Bien que rares, certaines transformations peuvent à la fois augmenter le chiffre d’affaires et réduire les coûts. Il s’agit d’un résultat bénéficiaire.

Le reste de cet article explique ces résultats comptables dans le contexte d’une transformation cloud.

> [!NOTE]
> Les exemples suivants sont basés sur des hypothèses et ne doivent pas être considérés comme une garantie de retour lors de l’adoption d’une stratégie cloud.

## <a name="revenue-outcomes"></a>Résultats en termes de chiffre d’affaires

### <a name="new-revenue-streams"></a>Nouveaux flux de revenus

Le cloud offre des opportunités de fournir de nouveaux produits à des clients ou de fournir des produits existants d’une nouvelle façon. Les nouveaux flux de revenus sont une source d’innovation, d’entrepreneuriat et de motivation pour de nombreuses personnes dans le monde des affaires. Toutefois, ils sont aussi sources d’échec et sont perçus par un grand nombre d’entreprises comme un grand risque. Quand le département informatique en propose, il y a de fortes chances pour qu’ils soient refusés. Pour renforcer la crédibilité de ces revenus, établissez un partenariat avec un chef d’entreprise reconnu pour ses innovations. Une validation du flux de revenus tôt dans le processus permettra à l’entreprise d’éviter les obstacles.

* Exemple : Une société vend des livres depuis plus de cent ans. Un employé de la société réalise que le contenu des livres peut être distribué par voie électronique et crée un appareil qui peut être vendu dans la librairie et qui permet de les télécharger directement, et ainsi de générer X euros avec les nouvelles ventes.

### <a name="revenue-increases"></a>Augmentation du chiffre d’affaires

D’une portée mondiale et numérique, le cloud permet aux entreprises d’augmenter le chiffre d’affaires des flux de revenus existants. Il arrive souvent que ce type de résultat provienne d’un alignement avec la direction des ventes ou du marketing.

* Exemple : Une entreprise qui vend des appareils pourrait en vendre plus si les vendeurs avaient la possibilité d’accéder de façon sécurisée au catalogue numérique et aux niveaux des stocks de l’entreprise. Malheureusement, ces données se trouvent seulement dans le système ERP de l’entreprise, qui est accessible seulement via un appareil connecté au réseau. La création d’un service frontal pour s’interfacer avec l’ERP en exposant la liste du catalogue et les niveaux des stocks non sensibles à une application dans le cloud permettrait aux vendeurs d’accéder aux données dont ils ont besoin alors qu’ils se trouvent sur site chez un client. L’extension d’AD avec Azure AD et l’intégration des accès en fonction du rôle dans l’application permettraient à l’entreprise de garantir la sécurisation des données. Ce projet simple pourrait avoir un impact de X % sur le chiffre d’affaires d’une ligne de produits existante.

### <a name="profit-increases"></a>Augmentation des bénéfices

Il est rare qu’un même effort augmente les revenus et réduit les coûts simultanément. Néanmoins, quand cela se produit, alignez les résultats d’un ou plusieurs des revenus générés sur l’un ou plusieurs des coûts générés pour communiquer le résultat souhaité.

## <a name="cost-outcomes"></a>Résultats en termes de coûts

### <a name="cost-reduction"></a>Réduction des coûts

Cloud computing peut réduire les dépenses de capital (dépenses d’investissement) liés à l’achat de matériel et logiciel, configuration des centres de données, l’exécution des centres de données sur site, etc.... Les racks de serveurs, l’alimentation électrique permanente pour le fonctionnement et le refroidissement, et les experts en informatique pour la gestion de l’infrastructure s’additionnent rapidement. Arrêt d’un centre de données peut réduire les engagements en matière de dépenses d’investissement. Cela est parfois appelé « tirer parti de l’entreprise de centre de données ». Réduction des coûts est généralement mesurée en dollars dans le budget actuel, ce qui peut s’étendre de 1 à 5 ans, selon la façon dont le directeur financier gère des finances.

* Exemple 1 : Comptes de centre de données d’une société d’un pourcentage important du budget informatique annuel. IT choisit d’exécuter une Transformation opérationnelle une migre les ressources dans ce centre de données à l’infrastructure en tant qu’une solution de service (IaaS), création d’une réduction des coûts de trois ans.
* Exemple 2 : Une société de portefeuille a récemment acquis une nouvelle entreprise. Dans l’acquisition, les termes du contrat indiquent que la nouvelle entité retiré les centres de données en cours dans les six mois. Si elle ne le fait pas, elle entraîne une pénalité de 1 million d’euros/mois au profit de la société de portefeuille. Le déplacement des ressources numériques vers le cloud dans le cadre d’une transformation opérationnelle pourrait permettre une mise hors service rapide des anciennes ressources.
* Exemple 3 : Une société qui s’occupe de la gestion des impôts sur le revenu de particuliers réalise 70 % de son chiffre d’affaires annuel au cours des trois premiers mois de l’année. Le reste de l’année, leur important investissement dans l’informatique est relativement peu utilisé. Une transformation opérationnelle permettrait au département informatique de déployer la capacité de traitement/hébergement nécessaire pour ces trois mois. Pendant les 9 mois restants, les coûts IaaS pourraient être considérablement réduits en diminuant la surface du système informatique.

### <a name="coverdell"></a>Coverdell

Coverdell modernise son infrastructure afin de réaliser des économies de coûts avec Azure. La décision de Coverdell d’investir dans Azure et d’unifier son réseau de sites web, ses applications, ses données et son infrastructure au sein de cet environnement a permis de réaliser davantage d’économies que ce que la société attendait. La migration vers un environnement Azure uniquement éliminé 54 000 USD pour les coûts mensuels pour les services de colocalisation. Avec l’infrastructure la société nouvelle, États-Unis uniquement, Coverdell s’attend à enregistrer une estimation $1 M USD au cours des deux ou trois années.
« Ayant un accès à la pile de technologies Azure s’ouvre la porte pour certains évolutive, facile à implémenter et des solutions hautement disponibles qui sont rentables. Ainsi, nos architectes peuvent être bien plus créatif avec les solutions qu’ils fournissent. »
Ryan Sorensen, directeur du développement et applications Enterprise Architecture Coverdell

### <a name="cost-avoidance"></a>Évitement des coûts

Arrêt des centres de données peut également entraîner une baisse des coûts en empêchant les cycles d’actualisation ultérieure. Un cycle de renouvellement est un processus consistant à acheter de nouveaux matériels et logiciels pour remplacer des systèmes locaux vieillissants. Dans Azure, le matériel et le système d’exploitation sont maintenus, corrigés et actualisés régulièrement sans coût supplémentaire pour les clients. Ceci permet à un directeur financier d’éliminer certaines dépenses planifiées futures dans les prévisions financières à long terme. La suppression des coûts se mesure en euros. Elle diffère de la réduction des coûts en cela qu’elle est centrée sur un budget futur qui n’a pas encore été entièrement approuvé.

* Exemple : Centre de données d’une société est prêt à être un renouvellement de bail dans six mois. Ce centre de données a été en service pendant huit ans. Quatre ans, tous les serveurs ont été actualisées et virtualisé évaluation des coûts de la société $ millions. L’année prochaine, l’entreprise prévoit d’actualiser à nouveau le matériel et les logiciels. Migrer les ressources dans ce centre de données, dans le cadre d’une Transformation opérationnelle, créerait une baisse des coûts, en supprimant l’actualisation planifiée de budget prévues de l’année suivante. Elle pourrait également produire une réduction des coûts en réduisant ou en éliminant les coûts liés à la location des locaux.

### <a name="capex-versus-opex"></a>Dépenses d’investissement (CapEx) et dépenses d’exploitation (OpEx)

Avant d’aborder les résultats en termes de coûts, il est important de comprendre les deux options principales pour les coûts : les dépenses d’investissement (CAPEX, Capital Expenses) et les dépenses d’exploitation (OPEX, Operational Expenses).

Les termes suivants sont destinés à permettre de bien comprendre les différences entre CAPEX et OPEX dans les conversations avec l’entreprise à propos de votre parcours de transformation.

* Un **investissement** est de l’argent ou des actifs appartenant à une entreprise, destiné à contribuer à un objectif particulier, comme augmenter la capacité d’un serveur ou créer une application.
* **MAJUSCULE dépenses (d’exploitation CapEx)** est une dépense qui génère des avantages sur une longue période. Une telle dépense est généralement non récurrente et aboutit à l’acquisition de ressources permanentes. La création d’une application peut être qualifiée de dépense d’investissement.
* Une **dépense d’exploitation (OPEX)** est une dépense qui est un coût continu lié à l’exercice de l’activité par l’entreprise. La consommation de services cloud dans un modèle de paiement à l’utilisation peut être qualifiée de dépense d’exploitation.
* Un actif est une ressource économique qui peut être détenue ou contrôlée pour produire de la valeur. Des serveurs, des lacs de données et des applications peuvent tous être considérés comme des ressources.
* La **dépréciation** est la façon dont la valeur d’un actif diminue au fil du temps. Plus pertinente pour les conversations autour de CAPEX/OPEX est la façon dont les coûts d’un actif sont alloués entre les périodes auxquelles il est utilisé. Par exemple, si vous générez une application de cette année, mais il est supposé avoir une durée moyenne de cinq ans (par exemple, la plupart des applications), puis le coût de l’équipe de développement et les outils nécessaires requises pour créer et déployer la base de code est amorti uniformément sur cinq ans.
* La **valorisation** est le processus d’estimation de la valeur d’une entreprise. Dans la plupart des secteurs d’activité, la valorisation est basée sur la capacité de l’entreprise à générer du chiffre d’affaires et du bénéfice, tout en respectant les coûts d’exploitation nécessaires pour créer les biens qui sont à l’origine de ce chiffre d’affaires. Dans certains secteurs comme la distribution ou dans certains types de transactions comme le capital investissement, les actifs et la dépréciation peuvent jouer un rôle important dans la valorisation de la société.

Il y a fort à parier que différents dirigeants, y compris le DSI, discutent de la meilleure utilisation du capital pour développer la société dans la direction voulue. Donner au DSI le moyen de convertir des discussions où il est en concurrence avec d’autres dirigeants sur le CAPEX en objectifs comptables OPEX clairs peut être un résultat intéressant par lui-même. Dans de nombreux secteurs, les directeurs financiers recherchent activement des moyens pour mieux associer des chiffrages comptables au coût des biens vendus.

Cependant, avant d’associer des parcours de transformation à ce type de conversion des CAPEX en OPEX, il est judicieux de rencontrer des membres des équipes de la direction financière ou informatique, pour déterminer si l’entreprise préfère les structures de coûts de type CAPEX ou OPEX. Dans certaines organisations, aboutir à réduire les CAPEX en faveur des OPEX est un résultat réellement non souhaitable. Comme mentionné plus haut, cela se produit parfois dans les sociétés de distribution, de portefeuille ou de capital investissement, qui accordent une valeur plus importante aux modèles traditionnels de comptabilisation des actifs, lesquels accordent une valeur minime à l’informatique. Vous pouvez aussi le voir dans les organisations qui ont eu dans le passé des expériences négatives lors de l’externalisation de leur service informatique ou d’autres fonctions.

Si des OPEX sont souhaitables, l’exemple suivant peut être un résultat viable pour l’entreprise :

* Exemple : Centre de données de la société est actuellement Amortir à $X par an pour les trois prochaines années. Il est prévu que Y euros seront nécessaires pour actualiser le matériel au cours des prochaines années. Nous pouvons convertir toutes ces CAPEX en un modèle OPEX à un coût uniforme de Z euros/mois, ce qui permet une meilleure gestion et un meilleur contrôle des coûts d’exploitation de la technologie.
