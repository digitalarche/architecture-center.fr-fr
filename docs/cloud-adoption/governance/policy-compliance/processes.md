---
title: 'Framework d’adoption du cloud : Surveiller et appliquer l’adhésion aux stratégies'
description: Comment veillez-vous au respect des stratégies établies ?
author: BrianBlanchard
ms.date: 1/4/2019
ms.openlocfilehash: 9066f33c8baa183476a9632e82d6eb960d03752c
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900757"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-processes-can-help-ensure-policy-adherence"></a>Quels processus peuvent aider à assurer le respect des stratégies ?

<!---
I've defined policies, I've provided an architecture guide. Now how do I monitor adherence to policy? If there is a violation, how do I enforce the policy?
--->

Après avoir établi vos déclarations de stratégie cloud et rédigé un guide de conception, vous devez créer des consignes pour être sûr que votre déploiement cloud reste conforme aux exigences de votre stratégie. Ces consignes devront englober les processus continus d’examen et de communication de votre équipe de gouvernance cloud, établir des critères pour déterminer à quel moment les violations des stratégies doivent être sanctionnées et définir les exigences relatives aux systèmes automatisés de surveillance et de conformité qui détecteront les violations et déclencheront des mesures correctives.

Consultez les sections sur les stratégies d’entreprise des [Parcours de gouvernance actionnables](../journeys/overview.md) pour découvrir des exemples de méthodes d’intégration du processus de conformité à la stratégie dans un plan de gouvernance cloud.

## <a name="prioritize-policy-adherence-processes"></a>Classer par ordre de priorité des processus d’adhérence à la stratégie

Combien d’investissements dans l’élaboration de processus sont nécessaires pour appuyer vos objectifs stratégiques ? En fonction de la taille et de la maturité de votre déploiement cloud, l’effort requis pour établir des processus qui prennent en charge la conformité et les coûts associés à cet effort peuvent varier considérablement.

Pour les petits déploiements comprenant des ressources de développement et de test, les exigences de la stratégie peuvent être simples et nécessiter peu de ressources dédiées pour y répondre. En revanche, un déploiement cloud mature et essentiel à votre mission, dont les besoins sont prioritaires en matière de sécurité et de performances, peut nécessiter une équipe d’employés, des processus internes étendus et des outils de surveillance personnalisés pour soutenir vos objectifs stratégiques.

Commencez par évaluer comment les processus décrits ci-dessous peuvent répondre aux exigences de votre stratégie. Déterminez la quantité d’efforts nécessaire pour investir dans ces processus, puis utilisez cette information pour établir un budget et des plans de dotation réalistes pour répondre à ces besoins.

## <a name="establish-cloud-governance-team-processes"></a>Établir des processus d’équipe pour la gouvernance cloud

Avant de définir les déclencheurs de la correction de la conformité à la votre stratégie, vous devez établir les processus globaux que votre équipe utilisera, et la façon dont l’information sera partagée et transmise entre le personnel informatique et l’équipe de gouvernance cloud.

### <a name="assign-cloud-governance-team-members"></a>Affectation des membres de l’équipe de gouvernance cloud

Qui fournira des conseils continus sur la conformité à la stratégie et s’occupera des questions liées à la gestion de la stratégie qui émergent lors du déploiement et de l’exploitation de vos actifs cloud ? La taille et la composition de votre équipe de gouvernance cloud dépendront de la complexité de vos exigences en matière de stratégie, ainsi que des priorités budgétaires et de dotation en personnel que vous attachez au respect des stratégies.

Choisissez des membres de l’équipe qui ont de l’expertise dans les domaines relatifs aux instructions de la stratégie que vous avez définie. Vous pouvez vous en tenir à quelques administrateurs système lors des premiers tests de déploiement. Ils établiront les bases de la gouvernance. Votre équipe de gouvernance cloud devra évoluer pour prendre en charge l’avancement de vos déploiements et la complexification des exigences de vos stratégies, tandis que ces dernières s’intègrent aux exigences générales de votre stratégie d’entreprise.

Au fur et à mesure que vos processus de gouvernance évoluent, vous devrez examiner régulièrement les membres de l’équipe chargée des conseils cloud, pour vérifier qu’ils sont capables de répondre correctement aux dernières exigences de la stratégie. Identifiez les membres de votre personnel informatique ayant une expérience pertinente ou un intérêt dans des domaines spécifiques de gouvernance et intégrez-les dans vos équipes de façon permanente ou ponctuelle en fonction des besoins.

### <a name="reviews-and-policy-iteration"></a>Révisions et itération de la stratégie

Au fur et à mesure que des ressources supplémentaires seront déployées, l’équipe de gouvernance du cloud devra s’assurer que les nouvelles charges de travail ou les nouvelles ressources sont conformes aux exigences de la stratégie. Planifiez des réunions avec les équipes responsables du déploiement de chaque nouvelle ressource, pour discuter son harmonisation avec vos guides de conception.

Au fur et à mesure que votre déploiement global prend de l’ampleur, évaluez régulièrement les nouveaux risques potentiels. Le cas échéant, mettez à jour les instructions de stratégie et les guides de conception. Planifiez des cycles d’examen réguliers pour chacune des cinq disciplines de gouvernance. Vous pourrez ainsi vous assurer que la stratégie est à jour et respectée.

### <a name="education"></a>Formation

Pour être conforme à votre stratégie, le personnel et les développeurs de votre service informatique doivent comprendre ses exigences vis-à-vis de leurs domaines de responsabilité. Prévoyez de consacrer des ressources à la documentation des décisions et des exigences, et formez toutes les équipes concernées à l’aide de vos guides de conception pour la prise en charge des exigences de votre stratégie.

Au fur et à mesure que votre stratégie évolue, mettez régulièrement à jour la documentation et le matériel de formation, et veillez à ce que toutes les exigences et les directives mises à jour soient communiquées au personnel informatique concerné lors des formations.  

### <a name="establish-escalation-paths"></a>Organiser la remontée des informations

Si une ressource n’est pas conforme, qui en est avisé ? Si le personnel informatique détecte un problème de conformité à la stratégie, à qui s’adresse-t-il ? Assurez-vous que la remontée des informations vers l’équipe de gouvernance Cloud est clairement définie. Veillez à ce que les canaux de communication soient tenus à jour afin de refléter les modifications au sein de votre personnel et de votre organisation.

## <a name="violation-triggers-and-actions"></a>Déclencheurs de violations et actions

Après avoir défini votre équipe de gouvernance cloud et ses processus, vous devez définir explicitement les déclencheurs, c’est-à-dire les événements constituant une violation de conformité susceptible d’entraîner des actions. Puis, vous devrez aussi définir quelles devraient être ces actions.

### <a name="define-triggers"></a>Définir des déclencheurs

Passez en revue les exigences de chacun de vos énoncés de votre stratégie, afin de déterminer ce qui en constitue une violation. Générez vos déclencheurs en utilisant les informations que vous avez déjà établies dans le cadre du processus de définition de la stratégie.

* Tolérance au risque : créez des déclencheurs de violation basés sur les mesures et les indicateurs de risque que vous avez établis dans le cadre de votre [analyse de tolérance au risque](risk-tolerance.md).
* Exigences de stratégies définies : les énoncés de stratégie peuvent fournir des contrats de niveau de service (SLA), des exigences de continuité d’activité et reprise d’activité (BRCD) ou des exigences de performances qui doivent servir de base aux déclencheurs de conformité.

### <a name="define-actions"></a>Définir des actions

Chaque déclencheur de violation doit entraîner une action. Les actions déclenchées doivent toujours déclencher l’envoi d’une notification à un membre approprié du personnel informatique ou de l’équipe de gouvernance du cloud lorsqu’une violation se produit. Cette notification peut entraîner un examen manuel du problème de conformité ou déclencher un processus de correction préétabli en fonction du type et de la gravité de la violation détectée.

Exemples de déclencheurs de violations et d’actions :

| Discipline de gouvernance cloud | Exemple de déclencheur | Exemple d’action |
|-----------------------------|----------------|---------------|
| Gestion des coûts | Les dépenses cloud mensuelles sont 20 % plus élevées que prévu. | Notifiez le responsable de l’unité de facturation qui examinera l’utilisation des ressources. |
| Base de référence de la sécurité | Détectez toute activité de connexion d’utilisateur suspecte. | Prévenez l’équipe de sécurité informatique et désactivez le compte de l’utilisateur suspect. |
| Cohérence des ressources | Une charge de travail utilise plus de 90 % de l’UC. | Prévenez l’équipe des opérations informatiques et montez en charge des ressources supplémentaires pour gérer la charge. |

## <a name="monitoring-and-compliance-automation"></a>Automatisation de la surveillance et de la conformité

Lorsque vous avez défini vos déclencheurs et actions de violation de conformité, vous pouvez commencer à planifier la meilleure façon d’utiliser les outils de journalisation et de création de rapport, ainsi que les autres fonctionnalités de votre plate-forme cloud pour automatiser la surveillance et la mise en conformité vis-à-vis de votre stratégie.

Consultez la rubrique [guide de décision pour l’enregistrement et la déclaration](../../decision-guides/log-and-report/overview.md) du Framework d’adoption du cloud pour obtenir des conseils dans le choix du meilleur modèle de surveillance pour votre déploiement.
