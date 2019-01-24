---
title: Création d’une analyse de rentabilisation d’une migration cloud
titleSuffix: Enterprise Cloud Adoption
description: Points importants à prendre en compte pour élaborer une justification de la migration cloud
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.openlocfilehash: 8c27ce211f500ee2eec4f7775a7f68f214dba433
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54488322"
---
# <a name="enterprise-cloud-adoption-building-a-cloud-migration-business-case"></a>Adoption du cloud d’entreprise : Création d’une analyse de rentabilisation d’une migration cloud

Les migrations cloud peuvent rapidement générer un retour sur investissement (ROI), grâce aux efforts de transformation du cloud. Toutefois, l’élaboration d’une justification qui présente concrètement tous les coûts et bénéfices connexes peut s’avérer complexe. Cet article vise à vous aider à réfléchir aux données dont vous avez besoin pour créer un modèle financier aligné sur les résultats d’une migration cloud. Avant toute chose, nous allons dissiper quelques mythes autour de la migration cloud afin d’éviter à votre organisation de commettre certaines erreurs courantes.

## <a name="dispelling-cloud-migration-myths"></a>Démystification de la migration cloud

**Mythe : Le cloud est toujours moins cher.** Il est courant de penser que l’exploitation d’un centre de données dans le cloud revient moins cher qu’un centre de données local. Cela n’est pas forcément vrai. Dans certains cas, les coûts d’exploitation du cloud peuvent en réalité être plus élevés. Souvent, cela s’explique par une mauvaise gestion des coûts, une inadéquation des architectures système, une duplication des processus, des configurations système inhabituelles ou une hausse des frais de personnel. Heureusement, il est possible d’atténuer la plupart de ces problèmes pour obtenir un retour sur investissement rapide. Les conseils donnés dans la section [Élaboration de la justification](#building-the-business-justification) peuvent vous aider à détecter et éviter ces écarts. Dissiper les autres mythes cités ci-dessous peut également être utile.

**Mythe : Tout doit être placé dans le cloud.** En fait, il existe différents axes stratégiques qui peuvent vous amener à choisir une solution hybride. Avant de finaliser le modèle financier, il est conseillé d’effectuer une première analyse quantitative comme cela est expliqué dans les [articles consacrés au patrimoine numérique](../digital-estate/5-rs-of-rationalization.md). Pour plus d’informations sur les critères quantitatifs impliqués dans la rationalisation, consultez [Les 5 R de la rationalisation](../digital-estate/5-rs-of-rationalization.md). Chaque approche utilise des données d’inventaire faciles à obtenir ainsi qu’une brève analyse quantitative pour identifier les charges de travail ou les applications susceptibles de présenter des coûts plus élevés dans le cloud. Ces approches peuvent également identifier les dépendances ou les modèles de trafic qui nécessitent une solution hybride.

**Mythe : La mise en miroir de mon environnement local va m’aider à faire des économies dans le cloud.** Lors de la planification du patrimoine numérique, il n’est pas rare que les clients détectent une sous-utilisation des ressources de 50 % par rapport à la capacité de l’environnement provisionné. Si les ressources sont provisionnées dans le cloud en fonction du provisionnement actuel, vous aurez du mal à réaliser des économies. Essayez de réduire la taille des ressources déployées pour les aligner avec les modèles d’utilisation plutôt qu’avec les modèles de provisionnement.

**Mythe : Les coûts de serveur impactent les analyses de rentabilisation de la migration cloud.** Cela est parfois vrai. Certaines entreprises jugent qu’il est important de réduire leurs dépenses d’investissement actuelles pour les serveurs. Cela dépend toutefois de plusieurs facteurs. Les entreprises qui renouvellent leur matériel tous les cinq à huit ans ont peu de chance d’obtenir un retour sur investissement rapide de leur migration cloud. Celles qui ont des cycles de renouvellement standards ou fixes peuvent atteindre un seuil de rentabilité rapidement. Dans les deux cas, d’autres types de dépenses peuvent être les déclencheurs financiers qui justifient la migration. Voici quelques exemples des coûts qui sont souvent négligés dans une analyse axée sur les coûts des serveurs uniquement ou sur les coûts des machines virtuelles uniquement :

- Les coûts logiciels pour la virtualisation, les serveurs et les middlewares (intergiciels) peuvent être considérables. Les fournisseurs de cloud éliminent certains de ces coûts. Les programmes [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/#services) et [Réservations](https://azure.microsoft.com/reservations/) sont deux exemples de fournisseur de cloud qui permettent de réduire les coûts de virtualisation.
- Les pertes financières dues à des pannes peuvent rapidement dépasser les coûts matériels ou logiciels. Si le centre de données actuel est instable, travaillez avec l’entreprise pour quantifier l’impact des arrêts sur les coûts d’opportunité ou les coûts d’exploitation actuels.
- Les coûts environnementaux peuvent également avoir un impact à ne pas négliger. Pour une famille américaine moyenne, le logement constitue le plus gros investissement et le coût le plus élevé dans leur budget. Ce constat est souvent le même pour les centres de données. Les coûts immobiliers, d’équipement et d’énergie représentent une bonne part des coûts locaux. À la mise hors service des centres de données, l’entreprise peut reconvertir ces équipements ou éventuellement les sortir entièrement des coûts.

**Mythe : Les dépenses d’exploitation (OPEX) sont préférables aux dépenses d’investissement (CAPEX).** Comme cela est expliqué dans l’article sur les [résultats financiers](business-outcomes/fiscal-outcomes.md), les dépenses d’exploitation peuvent être une bonne chose. Cependant, elles sont parfois perçues de façon négative par les entreprises de certains secteurs d’activité. Par exemple, les points suivants sur les dépenses d’exploitation méritent d’être discutés en détail avec les unités comptables et commerciales de l’entreprise :

- Si l’entreprise perçoit les immobilisations comme un facteur d’évaluation économique, la réduction des dépenses d’investissement peut être considérée comme un résultat négatif. Ce sentiment, même s’il n’est pas universel, se rencontre plus fréquemment dans les entreprises des secteurs de la vente au détail, de l’industrie et de la construction.
- Par ailleurs, une hausse des dépenses d’exploitation peut être considérée comme un résultat négatif dans les entreprises détenues par une société d’investissement privé ou celles ayant besoin d’un apport de capitaux.
- Les entreprises dont la principale priorité est l’amélioration des marges commerciales ou la baisse du coût des marchandises vendues (COGS) peuvent voir les dépenses d’exploitation comme un résultat négatif.

Les dépenses d’exploitation ne sont pas toujours une mauvaise chose. Les entreprises ont tendance à percevoir les dépenses d’exploitation de manière plus favorable que les dépenses d’investissement. Par exemple, cette approche peut convenir aux entreprises qui tentent d’améliorer leur trésorerie, de réduire leurs dépenses d’investissement ou de diminuer leurs actifs.

Avant de fournir une justification financière axée sur la conversion de dépenses d’investissement en dépenses d’exploitation, vous devez bien comprendre ce qui est préférable pour l’entreprise. La comptabilité et les achats peuvent généralement vous aider à mieux aligner votre message avec les objectifs financiers.

**Mythe : Migrer vers le cloud est aussi simple que d’appuyer sur un bouton.** Une migration est une transformation technique qui s’effectue manuellement. Dans votre justification, en particulier pour les facteurs temps, prenez en compte les aspects suivants qui peuvent allonger le processus de migration des ressources :

- Limitations de bande passante : la quantité de bande passante entre le centre de données actuel et le fournisseur de cloud commande les chronologies pendant la migration.
- Chronologies des tests avec l’entreprise : tester les applications avec l’entreprise pour valider leur disponibilité et leurs performances peut prendre du temps. Il est donc primordial de planifier les processus de test en accord avec les utilisateurs décisionnaires.
- Chronologies de la réalisation de la migration : le temps et l’effort humain nécessaires pour réaliser la migration sont des facteurs susceptibles d’accroître les coûts et les délais de réalisation. L’allocation d’employés ou de partenaires contractuels peut également retarder le processus et doit donc être prise en compte dans le plan.

L’adoption du cloud peut être ralentie par des obstacles techniques et culturels. Quand le temps est un facteur important de la justification, la planification est la meilleure mesure d’atténuation. Durant la planification, il y a deux approches possibles qui peuvent vous aider à atténuer les risques liées aux chronologies.

- Tout d’abord, prenez le temps de bien comprendre les contraintes techniques de l’adoption. Même si vous devez effectuer la migration dans des délais courts, il est important d’établir des chronologies d’exécution réalistes.
- En second lieu, prenez en considération les éventuels freins culturels ou réticences du personnel, car ils ont un plus grand impact que les contraintes techniques. L’adoption du cloud engendre des changements, qui sont nécessaires pour parvenir à la transformation souhaitée. Malheureusement, de tels changements font parfois peur, et les personnes réticentes ont alors besoin d’explications supplémentaires pour accepter le plan. Identifiez les principales personnes de l’équipe qui sont réfractaires au changement, et impliquez-les en amont dans le processus.

Pour anticiper et atténuer au maximum les risques liés aux chronologies, préparez les parties prenantes décisionnaires en leur présentant une solide justification avec les bénéfices et les résultats pour l’entreprise. Expliquez-leur quels changements accompagneront cette transformation. Soyez clair et établissez des objectifs réalistes dès le début. Si le processus est freiné par une personne ou une technologie, vous aurez ainsi plus de soutien de la part des responsables.

## <a name="building-the-business-justification"></a>Élaboration de la justification

Le processus suivant définit une approche possible pour élaborer la justification d’une migration cloud. Si, lors de la lecture de ce contenu, vous avez besoin d’explications supplémentaires sur des calculs ou des termes financiers, consultez l’article consacré aux [modèles financiers](financial-models.md).

D’un point de vue général, la formule utilisée pour la justification est simple. Toutefois, dans le détail, les éléments de données requis pour remplir la formule peuvent être difficiles à déterminer. De manière globale, la justification s’attache au retour sur investissement (ROI) associé à la proposition de changement technique. La formule générique pour le retour sur investissement est :

Retour sur investissement = (Gain lié à l’investissement &minus; Investissement initial) / Investissement initial

La décomposition de cette formule crée une vue axée sur la migration de la formule qui place chacune des variables d’entrée à droite de cette équation. Les sections suivantes de cet article présentent différents points à prendre en compte.

![Le retour sur investissement est égal à : (Gain lié à l’investissement – Coût de l’investissement) / Coût de l’investissement](../_images/formula-roi.png)

## <a name="migration-specific-initial-investment"></a>Investissement initial pour la migration

- Les fournisseurs de cloud comme Azure proposent des outils de calcul pour estimer le montant des investissements dans le cloud. La [calculatrice de prix Azure](https://azure.microsoft.com/en-in/pricing/) en est un exemple.
- Certains fournisseurs de cloud prennent également en charge des calculatrices d’écarts de coûts. Un exemple est la [calculatrice du coût total de possession (TCO) Azure](https://azure.com/tco).
- Pour des structures de coûts plus détaillées, effectuez plutôt une [planification du patrimoine numérique](../digital-estate/overview.md).
- Estimez le coût de la migration.
- Estimez le coût des besoins de formation. [Microsoft Learn](https://docs.microsoft.com/learn/) peut être une solution pour atténuer ces coûts.
- Dans certaines entreprises, le temps passé par le personnel existant doit aussi être inclus dans les coûts initiaux. Pour obtenir des conseils, consultez le service financier.
- Discutez des coûts supplémentaires ou des coûts cachés éventuels avec le service financier pour faire valider votre estimation.

## <a name="migration-specific-revenue-deltas"></a>Écarts de revenus liés à la migration

Cet aspect est souvent négligé lors de l’élaboration d’une justification de la migration. Dans certains domaines, le cloud peut réduire les coûts. Toutefois, l’objectif final de toute transformation est d’améliorer les résultats à long terme. Tenez compte des répercussions en aval pour comprendre comment améliorer les revenus dans la durée. Quelles nouvelles technologies seront à la disposition de l’entreprise après cette migration, mais qui ne peuvent pas encore être exploitées aujourd’hui ? Quels projets ou objectifs de l’entreprise sont bloqués en raison de dépendances sur des technologies héritées ? Quels programmes sont en attente à cause des coûts d’exploitation élevés des technologies sous-jacentes ?

Après avoir étudié les nouvelles opportunités offertes par le cloud, travaillez avec l’entreprise pour calculer les hausses de revenus envisageables grâce à ces opportunités.

## <a name="migration-specific-cost-deltas"></a>Écarts de coûts liés à la migration

Calculez les nouveaux coûts estimés après la migration proposée. Pour plus d’informations sur les différents types d’écarts de coûts, consultez [Modèles financiers](financial-models.md). Les fournisseurs de cloud proposent souvent des outils permettant de calculer les écarts de coûts. Un exemple est la [calculatrice du coût total de possession (TCO) Azure](https://azure.com/tco).

Voici d’autres exemples de coûts qu’une migration cloud peut réduire :

- Arrêt ou réduction du centre de données (coûts environnementaux)
- Diminution de la consommation d’énergie (coûts environnementaux)
- Arrêt du rack (récupération de ressources physiques)
- Pas de renouvellement du matériel (évitement de coûts)
- Pas de renouvellement des logiciels (réduction des coûts opérationnels ou évitement de coûts)
- Regroupement des achats (réduction des coûts opérationnels et réduction potentielle des coûts accessoires)

## <a name="when-roi-results-are-surprising"></a>Quand les résultats de retour sur investissement ne sont pas ceux escomptés

Si le retour sur investissement d’une migration cloud n’est pas conforme aux attentes, il peut être utile de revoir la section sur les mythes courants au début de cet article.

Cependant, vous devez aussi savoir qu’une migration cloud n’est pas toujours synonyme d’économies. Certaines applications ont un coût d’exploitation plus élevé dans le cloud qu’en local. Elles peuvent donc largement biaiser les résultats dans une analyse.

Quand le retour sur investissement est inférieur à 20 %, envisagez d’effectuer une [planification du patrimoine numérique](../digital-estate/overview.md), en portant une attention particulière à la [rationalisation](../digital-estate/rationalize.md). Durant l’analyse quantitative, examinez chaque application pour rechercher les charges de travail qui faussent les résultats. Il peut être judicieux de supprimer ces charges de travail du plan. Si des données d’utilisation sont disponibles, essayez de réduire la taille des machines virtuelles en fonction de l’utilisation réelle.

Si le retour sur investissement n’est toujours pas suffisant, travaillez en collaboration avec votre représentant commercial Microsoft ou [demandez de l’aide à un partenaire expérimenté](https://azure.microsoft.com/en-us/migration/partners/).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer un modèle financier pour la transformation cloud](financial-models.md)