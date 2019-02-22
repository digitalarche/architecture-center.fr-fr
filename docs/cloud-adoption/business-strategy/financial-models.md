---
title: 'Framework d’adoption du cloud : Créer un modèle financier pour la transformation cloud'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
description: Comment créer un modèle financier pour la transformation cloud.
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.openlocfilehash: 4fe9b178962bf2cd6a79233278c73085237772f0
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55898202"
---
# <a name="create-a-financial-model-for-cloud-transformation"></a>Créer un modèle financier pour la transformation cloud

La création d’un modèle financier qui fait un état précis de tous les bénéfices d’une transformation cloud pour l’entreprise peut être compliquée. Les modèles financiers et les justifications varient souvent d’une organisation à l’autre. Cet article présente quelques formules de calcul et met en évidence un certain nombre d’éléments qui sont communément négligés au moment de la création d’un modèle financier.

## <a name="return-on-investment-roi"></a>Retour sur investissement (ROI)

Le retour sur investissement (ROI) est souvent un critère important pour la direction ou les responsables d’une entreprise. Il est utilisé pour comparer différents moyens d’investir une partie des ressources du capital. La formule du retour sur investissement est relativement simple, mais les valeurs de données à entrer dans cette formule peuvent être plus difficiles à déterminer. Globalement, le retour sur investissement est ce que vous fait gagner un investissement initial. Il est généralement représenté sous forme de pourcentage :

![Le retour sur investissement est égal à : (Gain de l’investissement – Coût de l’investissement) / Coût de l’investissement](../_images/formula-roi.png)

<!-- markdownlint-disable MD036 -->
*Retour sur investissement = (Gain de l’investissement &minus; Investissement initial) / Investissement initial*
<!-- markdownlint-enable MD036 -->

Dans les sections suivantes, nous allons voir quelles données sont à inclure dans le calcul de l’investissement initial et du gain de cet investissement (revenus).

## <a name="calculating-initial-investment"></a>Calcul de l’investissement initial

L’investissement initial correspond aux dépenses d’investissement (CAPEX) et aux dépenses d’exploitation (OPEX) à engager pour effectuer une transformation. La classification des coûts dépend du modèle comptable utilisé et des préférences du directeur financier. Toutefois, cette catégorie comprend des éléments comme les services professionnels à transformer, les licences logicielles utilisées uniquement pour les besoins de la transformation, le coût des services cloud durant la transformation et, éventuellement, le coût des employés salariés pendant la phase de transformation.

Faites le total de ces coûts pour obtenir une estimation de l’investissement initial.

## <a name="calculating-the-gain-from-investment"></a>Calcul du gain de l’investissement

Le gain de l’investissement est souvent calculé avec une deuxième formule, qui s’applique plus spécifiquement aux résultats de l’entreprise et aux changements techniques associés. Les revenus ne sont pas aussi simples à calculer que la réduction des coûts.

Pour calculer les revenus, vous devez utiliser deux variables :

![Le gain de l’investissement est égal à : Écarts de revenus + Écarts de coûts](../_images/formula-gain-from-investment.png)

<!-- markdownlint-disable MD036 -->
*Gain de l’investissement = Écarts de revenus + Écarts de coûts*
<!-- markdownlint-enable MD036 -->

Chaque variable est décrite ci-après.

## <a name="revenue-delta"></a>Écart de revenus

L’écart de revenus doit être estimé en amont avec l’entreprise. Une fois que les parties prenantes de l’entreprise ont validé l’impact sur les revenus, cette donnée peut être utilisée pour améliorer les résultats.

## <a name="cost-deltas"></a>Écarts de coûts

Les écarts de coûts correspondent à la variation des coûts, à la hausse ou à la baisse, résultant de la transformation. Plusieurs variables peuvent impacter les écarts de coûts indépendamment les unes des autres. Les revenus dépendent en grande partie de coûts de base comme les réductions de dépenses d’investissement, l’évitement de coûts, les réductions de coûts d’exploitation et les amortissements dégressifs. Les sections suivantes présentent des exemples d’écarts de coûts à prendre en compte.

### <a name="depreciation-reductions-or-acceleration"></a>Amortissement dégressif ou accéléré

Vous pouvez obtenir des conseils sur l’amortissement auprès du service financier de l’entreprise. Le paragraphe suivant donne des informations d’ordre général au sujet de l’amortissement.

Quand du capital est investi dans l’acquisition d’une ressource, cet investissement peut être utilisé à des fins financières ou fiscales pour générer des revenus tout au long de la durée de vie attendue de la ressource. Certaines entreprises voient l’amortissement comme un avantage fiscal positif. D’autres le considèrent comme une dépense permanente engagée, similaire à d’autres dépenses récurrentes attribuées au budget informatique annuel.

Discutez avec le service financier pour voir si l’élimination de l’amortissement serait envisageable et si cela aurait un impact positif sur les écarts de coûts.

### <a name="physical-asset-recovery"></a>Récupération de ressources physiques

Dans certains cas, les ressources mises hors service peuvent être vendues, générant ainsi des revenus supplémentaires. Ces revenus sont souvent amalgamés dans la réduction des coûts par facilité, alors qu’ils correspondent à une augmentation de revenus et qu’ils sont imposables en tant que tels. Discutez avec le service financier pour comprendre la viabilité de cette option et savoir comment prendre en compte les revenus qui en résultent.

### <a name="operational-cost-reductions"></a>Réductions des coûts d’exploitation

Les dépenses récurrentes nécessaires au fonctionnement de l’entreprise sont souvent appelées dépenses d’exploitation (OPEX). La catégorie des dépenses d’exploitation est très large. Dans la plupart des modèles comptables, elle inclut les éléments suivants : les licences logicielles, les frais d’hébergement, les factures d’électricité, les loyers immobiliers, les dépenses de climatisation, le personnel temporaire embauché, les locations d’équipements, les pièces de rechange, les contrats de maintenance, les services de réparation, les services de continuité d’activité/reprise d’activité après sinistre, et plusieurs autres dépenses non soumises à l’obligation d’approbation des dépenses d’investissement.

Cette catégorie est l’une des catégories recouvrant le plus de types de dépenses différentes pendant une transformation opérationnelle. Le temps consacré à l’établissement d’une liste exhaustive des dépenses est rarement du temps perdu. Interrogez le directeur informatique et le service financier pour vous assurer que vous avez bien pris en compte tous les coûts d’exploitation.

### <a name="cost-avoidance"></a>Évitement des coûts

Quand une dépense d’exploitation (OPEX) est prévue, mais pas encore intégrée à un budget approuvé, il n’est pas approprié de la mettre dans une catégorie de réductions des coûts. Par exemple, si les licences VMWare et Microsoft doivent être renégociées et achetées l’année prochaine, elles ne constituent pas encore des coûts engagés. Les réductions dans ces coûts prévus sont plutôt à traiter comme des coûts d’exploitation dans les calculs d’écarts de coûts. Ces dépenses doivent toutefois rester de manière informelle dans la catégorie « Évitement des coûts » jusqu’à ce que la négociation et l’approbation du budget soient terminées.

### <a name="soft-cost-reductions"></a>Réductions des coûts accessoires

Certaines entreprises incluent également les coûts accessoires, tels que les réductions de la complexité opérationnelle ou la réduction du personnel à plein temps en charge d’un centre de données. Toutefois, l’intégration des coûts accessoires est parfois déconseillée. En effet, elle introduit une hypothèse non documentée selon laquelle la réduction des coûts équivaudra de manière tangible aux économies réalisées. Les projets technologiques donnent rarement lieu à une récupération des coûts accessoires.

### <a name="headcount-reductions"></a>Réductions d’effectifs

Les gains de temps pour le personnel sont souvent inclus dans la réduction des coûts accessoires. Quand ces gains de temps correspondent à une réduction réelle des effectifs ou de la masse salariale du service informatique, ils peuvent être calculés séparément comme une réduction d’effectifs.

Ceci dit, les compétences requises en local correspondent généralement à des compétences similaires (voir supérieures) nécessaires dans le cloud. Cela signifie que les employés ne sont en général pas licenciés après une migration cloud.

Il y a une exception, quand la capacité opérationnelle est fournie par un tiers ou par un fournisseur de services managés (MSP). Si les systèmes informatiques sont gérés par un tiers, les coûts de leur exploitation peuvent être éliminés par une solution cloud native ou un MSP de cloud natif. Un MSP de cloud natif travaille souvent plus efficacement et potentiellement à moindre coût. Si c’est le cas, les réductions de coûts d’exploitation entrent dans les calculs des coûts de base.

### <a name="capital-expense-reductions-or-avoidance"></a>Réductions ou évitement des dépenses d’investissement

Les dépenses d’investissement (CAPEX) sont légèrement différentes des dépenses d’exploitation. En règle générale, cette catégorie s’applique aux cycles de renouvellement ou à l’extension des centres de données. Par exemple, une extension de centre de données serait l’ajout d’un nouveau cluster à hautes performances pour héberger une solution de Big Data ou un entrepôt de données. Cette extension serait généralement à inclure dans la catégorie des dépenses d’investissement. Les cycles standards de renouvellement sont les cas les plus courants. Certaines entreprises ont mis en place des cycles fixes de renouvellement de leur matériel, c’est-à-dire qu’elles mettent hors service et remplacent leurs ressources selon un cycle régulier (habituellement tous les trois, cinq ou huit ans). Souvent, ces cycles coïncident avec les cycles de location ou la durée de vie prévue des équipements. À chaque nouveau cycle de renouvellement, le service informatique investit pour acquérir du nouveau matériel.

Si un cycle de renouvellement est approuvé et budgétisé, la transformation cloud peut vous aider à éliminer ce coût. Si un cycle de renouvellement est planifié, mais pas encore approuvé, la transformation cloud peut engendrer un évitement de coûts d’exploitation. Les deux scénarios sont à inclure dans les écarts de coûts.

## <a name="next-steps"></a>Étapes suivantes

Consultez quelques exemples de résultats comptables dans le contexte d’une transformation cloud.

> [!div class="nextstepaction"]
> [Exemples de résultats comptables](./business-outcomes/fiscal-outcomes.md)