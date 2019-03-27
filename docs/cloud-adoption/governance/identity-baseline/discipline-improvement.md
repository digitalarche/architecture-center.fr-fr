---
title: 'Framework d’adoption du cloud : Amélioration de la discipline Base de référence des identités'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Amélioration de la discipline Base de référence des identités
author: BrianBlanchard
ms.openlocfilehash: c96a638af549782fec22b2068c9b4943df4b943a
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58243250"
---
# <a name="identity-baseline-discipline-improvement"></a>Amélioration de la discipline Base de référence des identités

La discipline Base de référence des identités se concentre sur les manières d’établir des stratégies qui garantissent la cohérence et la continuité des identités utilisateurs, quel que soit le fournisseur de cloud hébergeant l’application ou la charge de travail. Dans les cinq disciplines de la gouvernance cloud, la Base de référence des identités inclut des décisions concernant la [stratégie d’identité hybride](../../decision-guides/identity/overview.md), l’évaluation et l’extension de référentiels d’identités, l’implémentation de l’authentification unique (même authentification), l’audit et la surveillance des utilisations non autorisées ou des acteurs malveillants. Dans certains cas, elle peut également impliquer des décisions pour moderniser, consolider ou intégrer plusieurs fournisseurs d’identité.

Cet article décrit certaines tâches potentielles que votre entreprise peut entreprendre pour mieux développer et faire mûrir la discipline de Base de référence des identités. Ces tâches peuvent être décomposées en phases de planification, de construction, d’adoption et d’exploitation de l’implémentation d’une solution cloud, qui sont ensuite répétées pour permettre le développement d’une [approche incrémentielle de la gouvernance cloud](../journeys/overview.md#an-incremental-approach-to-cloud-governance).

![Quatre phases d’adoption](../../_images/adoption-phases.png)

*Figure 1 : Phases d’adoption de l’approche incrémentielle de la gouvernance cloud.*

Il est impossible pour un même document de prendre en compte les exigences de toutes les organisations. Par conséquent, cet article présente des exemples d’activités minimales et potentielles suggérés pour chaque phase du processus de maturation de la gouvernance. L’objectif initial de ces activités est de vous aider à créer un [produit minimum viable de stratégie](../journeys/overview.md#an-incremental-approach-to-cloud-governance) et à établir une infrastructure pour une évolution de stratégie incrémentielle. Votre équipe de gouvernance cloud devra décider combien investir dans ces activités pour améliorer vos fonctionnalités de gouvernance de base de référence des identités.

> [!CAUTION]
> Les activités minimales ou potentielles décrites dans cet article ne sont pas alignées sur des stratégies d’entreprise spécifiques ou des exigences de conformité de tiers. Ces conseils visent à favoriser les échanges qui conduiront à un alignement des deux exigences avec un modèle de gouvernance cloud.

## <a name="planning-and-readiness"></a>Planification et préparation

Cette phase de maturité de la gouvernance comble le fossé entre les résultats opérationnels et les stratégies concrètes. Au cours de ce processus, l’équipe de direction définit des métriques spécifiques, les mappe au patrimoine numérique, et commence à planifier l’effort global de migration.

**Activités minimales suggérées :**

* Évaluez vos options de [chaîne d’outils d’identité](toolchain.md) et implémentez une stratégie hybride appropriée pour votre organisation.
* Développez un brouillon de document d’instructions relatives à l’architecture et distribuez-le aux principales parties prenantes.
* Formez et impliquez les personnes et les équipes affectées par le développement des instructions relatives à l’architecture.

**Activités potentielles :**

* Définissez les rôles et attributions qui régissent la gestion des identités et des accès dans le cloud.
* Définissez vos groupes locaux et mappez-les aux rôles cloud correspondants.
* Fournisseurs d’identité d’inventaire (y compris les identités pilotées par base de données utilisées par des applications personnalisées).
* Examinez les options pour la consolidation ou l’intégration de fournisseurs d’identité où une duplication existe pour simplifier la solution d’identité globale.
* Évaluez la compatibilité hybride de fournisseurs d’identité existants.
* Pour les fournisseurs d’identité qui ne sont pas hybride compatibles, évaluez les options de consolidation ou de remplacement.

## <a name="build-and-pre-deployment"></a>Génération et prédéploiement

Il convient de répondre à un certain nombre de prérequis techniques et non techniques pour réussir la migration d’un environnement. Ce processus porte essentiellement sur les décisions, la préparation et l’infrastructure qui sert de base à la migration.

**Activités minimales suggérées :**

* Envisagez un test pilote avant d’implémenter votre [chaîne d’outils d’identité](toolchain.md), en vous assurant que cela simplifie autant que possible l’expérience utilisateur.
* Appliquez les commentaires résultants des tests pilotes dans le prédéploiement. Répétez l’opération jusqu’à ce que les résultats soient acceptables.
* Mettez à jour le document d’instructions relatives à l’architecture pour inclure des plans de déploiement et d’adoption par les utilisateurs, puis distribuez-les aux principales parties prenantes.
* Envisagez d’établir d’un programme à l’adresse des utilisateurs précoces et de déployer vers un nombre limité d’utilisateurs.
* Continuez de former les personnes et les équipes les plus concernées par les instructions relatives à l’architecture.

**Activités potentielles :**

* Évaluez votre architecture logique et physique, puis élaborez une [stratégie d’identité hybride](../../decision-guides/identity/overview.md).
* Mappez les stratégies de gestion des identités et des accès, telles que les affectations d’ID de connexion, puis choisissez la méthode d’authentification appropriée pour Azure AD.
  * En cas de fédération, activez les restrictions de locataire pour les comptes administratifs.
* Intégrez vos annuaires locaux et cloud.
* Envisagez d’utiliser les modèles d’accès suivants :
  * Modèle [Accès le moins privilégié](/windows-server/identity/ad-ds/plan/security-best-practices/implementing-least-privilege-administrative-models)
  * Modèle d’accès [Base de référence des identités privilégiées](/azure/active-directory/privileged-identity-management/pim-configure)
* Finalisez tous les détails de la pré-intégration et réexaminez les [meilleures pratiques en matière d’identités](/azure/security/azure-security-identity-management-best-practices).
  * Activez l’identité unique, l’authentification unique (SSO) ou l’authentification unique transparente.
  * Configurez l’authentification multifacteur (MFA) pour les administrateurs.
  * Consolidez ou intégrez les fournisseurs d’identité si nécessaire.
  * Implémentez les outils nécessaires pour centraliser la gestion des identités.
  * Activez l’accès juste-à-temps (JIT) et les alertes de modification de rôle.
  * Effectuez une analyse des risques des activités d’administration clés pour l’affectation à des rôles intégrés.
  * Envisagez d’opérer un déploiement mis à jour d’une authentification renforcée pour tous les utilisateurs.
  * Activez le modèle Base de référence des identités privilégiées (PIM) pour l’accès (activation limitée dans le temps) pour des rôles d’administration supplémentaires.
  * Séparez les comptes d’utilisateurs des comptes d’administrateur général (pour vous assurer que les administrateurs n’ouvrent pas des e-mails par inadvertance ou n’exécutent pas de programmes associés à leurs comptes d’administrateur général).

## <a name="adopt-and-migrate"></a>Adoption et migration

La migration est un processus incrémentiel qui porte essentiellement sur le déplacement, le test et l’adoption d’applications ou de charges de travail dans un patrimoine numérique.

**Activités minimales suggérées :**

* Migrez votre [chaîne d’outils d’identité](toolchain.md) du développement vers la production.
* Mettez à jour le document d’instructions relatives à l’architecture et distribuez-le aux principales parties prenantes.
* Élaborer une documentation et des ressources pédagogiques, des messages de sensibilisation, des incitations et d’autres programmes afin de favoriser l’adoption par les utilisateurs.

**Activités potentielles :**

* Validez le fait que les meilleures pratiques définies pendant les phases de génération/prédéploiement sont correctement mises en œuvre.
* Validez ou affinez votre [stratégie d’identité hybride](../../decision-guides/identity/overview.md).
* Assurez-vous que chaque application ou charge de travail reste alignée avec la stratégie d’identité avant la mise en production.
* Validez le fait que l’authentification unique (SSO) et l’authentification unique transparente fonctionnent comme prévu pour vos applications.
* Supprimez des magasins d’identités alternatifs pour en réduire le nombre lorsque cela est possible.
* Examinez la nécessité de disposer de magasins d’identités dans l’application ou dans la base de données. Des identités ne provenant pas d’un fournisseur d’identité approprié (interne ou tiers) peuvent constituer une risque pour l’application et les utilisateurs.
* Activez l’accès conditionnel pour [applications fédérées locales](/azure/active-directory/active-directory-device-registration-on-premises-setup).
* Distribuez l’identité dans les régions globales de plusieurs hubs avec une synchronisation entre régions.
* Établissez une fédération de contrôle d’accès en fonction du rôle (RBAC) centrale.

## <a name="operate-and-post-implementation"></a>Exploitation et post-implémentation

Une fois la transformation terminée, la gouvernance et les opérations doivent perdurer tout au long du cycle de vie naturel d’une application ou d’une charge de travail. Cette phase de maturité de la gouvernance est centrée sur les activités qui interviennent généralement après l’implémentation de la solution et le début de stabilisation du cycle de transformation.

**Activités minimales suggérées :**

* Personnalisez votre [chaîne d’outils de Base de référence des identités](toolchain.md) en fonction des changements résultant de l’évolution des besoins identité de votre organisation.
* Automatisez les notifications et les rapports pour vous avertir des menaces potentielles.
* Surveillez l’utilisation du système et la progression de l’adoption par les utilisateurs, et générez des rapports ad hoc.
* Générez des rapports sur les métriques post-déploiement et distribuez-les aux parties prenantes.
* Affinez les instructions relatives à l’architecture pour guider les futurs processus d’adoption.
* Communiquez avec les équipes concernées et formez-les régulièrement pour vous assurer de leur adhésion continue aux instructions relatives à l’architecture.

**Activités potentielles :**

* Effectuez des audits périodiques des stratégies d’identité et des pratiques d’adhésion.
* Recherchez régulièrement des acteurs malveillants et des violations de données, en particulier en lien à des fraudes en matière d’identité, telles que des prises de contrôle éventuelles de comptes d’administrateur.
* Configurez un outil de surveillance et de génération de rapports.
* Envisagez une intégration plus étroite avec les systèmes de sécurité et de prévention des fraudes.
* Passez régulièrement en revue les droits d’accès pour les utilisateurs ou les rôles avec élévation de privilèges.
  * Identifiez chaque utilisateur éligible pour activer le privilège d’administration.
* Passez en revue les processus d’intégration, de cessation de contrat et de mise à jour des informations d’identification.
* Examinez les niveaux croissants d’automation et de communication entre les modules de gestion des identités et des accès (IAM).
* Envisagez d’implémenter une approche des opérations de sécurité de développement (DevSecOps).
* Effectuez une analyse d’impact pour évaluer les résultats en matière de coûts, de sécurité et d’adoption par les utilisateurs.
* Générez périodiquement un rapport d’impact indiquant les modifications de métriques créées par le système, et estimez les impacts sur l’activité de la [stratégie d’identité hybride](../../decision-guides/identity/overview.md).
* Mettez en place la surveillance intégrée recommandée par [Azure Security Center](/azure/security-center/security-center-intro).

## <a name="next-steps"></a>Étapes suivantes

À présent que vous avez compris le concept de gouvernance des identités cloud, examinez la [chaîne d’outils de base de référence des identités](toolchain.md) pour identifier les outils et fonctionnalités Azure dont vous aurez besoin lors du développement de la discipline de gouvernance Base de référence des identités sur la plateforme Azure.

> [!div class="nextstepaction"]
> [Chaîne d’outils de base de référence des identités pour Azure](toolchain.md)