---
title: 'Framework d’adoption du cloud : Métriques, indicateurs et tolérance au risque de la base de référence des identités'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Métriques, indicateurs et tolérance au risque de la base de référence des identités
author: BrianBlanchard
ms.openlocfilehash: 4722de66308f3d18885ca930925e68e0e756ec03
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58242090"
---
# <a name="identity-baseline-metrics-indicators-and-risk-tolerance"></a>Métriques, indicateurs et tolérance au risque de la base de référence des identités

Cet article vise à vous aider à quantifier la tolérance au risque de l’entreprise en lien avec la base de référence des identités. La définition de métriques et d’indicateurs vous aide à créer une étude d’opportunité pour investir dans l’adoption progressive de la discipline Base de référence des identités.

## <a name="metrics"></a>Mesures

La base de référence des identités concerne l’identification, l’authentification et l’autorisation de personnes, de groupes d’utilisateurs ou des processus automatisés, et leur fournit un accès approprié aux ressources de vos déploiements cloud. Dans le cadre de votre analyse des risques, vous allez collecter des données relatives à vos services d’identité pour déterminer le risque auquel vous êtes exposé et l’importance d’investir dans la gouvernance de la base de référence des identités pour vos déploiements cloud planifiés.

Voici quelques exemples de métriques utiles que vous devez rassembler pour mieux évaluer la tolérance aux risques dans la discipline Base de référence des identités :

- **Taille des systèmes d’identité**. Nombre total d’utilisateurs, de groupes ou d’autres objets gérés via vos systèmes d’identité.
- **Taille globale de l’infrastructure des services d’annuaire**. Nombre de forêts, de domaines et de locataires d’annuaires utilisés par votre organisation.
- **Dépendance des mécanismes d’authentification hérités ou locaux**. Nombre de charges de travail dépendantes des mécanismes d’authentification hérités ou des services d’authentification multifacteur de tiers.
- **Étendue des services d’annuaire déployés dans le cloud**. Nombre de forêts, de domaines et de locataires d’annuaires que vous avez déployés dans le cloud.
- **Annuaires Active Directory déployés dans le cloud**. Nombre de serveurs Active Directory déployés dans le cloud.
- **Unités d’organisation déployées dans le cloud**. Nombre d’unités d’organisation déployées Active Directory dans le cloud.
- **Étendue de la fédération**. Nombre de systèmes de Base de référence des identités fédérés avec les systèmes de votre organisation.  
- **Utilisateurs avec privilèges élevés**. Nombre de comptes d’utilisateur ayant un accès avec privilèges d’accès élevés aux ressources ou aux outils de gestion.
- **Utilisation de RBAC**. Nombre d’abonnements, de groupes de ressources ou de ressources individuelles non gérés via le contrôle d’accès en fonction du rôle (RBAC).
- **Revendications d’authentification**. Nombre de tentatives d’authentification utilisateur ayant réussi ou échoué.
- **Revendications d’autorisation**. Nombre de réussites et d’échecs des tentatives effectuées par les utilisateurs pour accéder aux ressources.
- **Comptes compromis**. Nombre de comptes d’utilisateur qui ont été compromis.

## <a name="risk-tolerance-indicators"></a>Indicateurs de tolérance aux risques

Les risques liés à la Base de référence des identités sont principalement liés à la complexité de l’infrastructure d’identité de votre organisation. Si tous vos utilisateurs et groupes sont gérés en utilisant un seul annuaire ou un fournisseur d’identité cloud natif avec une intégration minimale à d’autres services, votre niveau de risque est probablement faible. Cependant, comme les besoins de votre entreprise doivent normalement s’accroître, vos systèmes de Base de référence des identités devront probablement prendre en charge des scénarios plus complexes, comme plusieurs annuaires prenant en charge votre organisation interne ou la fédération avec des fournisseurs d’identité externes. À mesure que ces systèmes deviennent plus complexes, le risque augmente.

Au début de l’adoption du cloud, collaborez avec votre équipe Sécurité informatique et les parties prenantes de l’entreprise pour identifier les [risques métier](business-risks.md) liés à l’identité, puis déterminez une base de référence acceptable pour la tolérance au risque pour les identités. Cette section des directives du Framework d’adoption du cloud fournit des exemples, mais les risques et les bases de référence détaillés pour votre entreprise ou vos déploiements peuvent être différents.

Une fois que vous avez une base de référence, établissez des points de référence minimaux représentant une augmentation inacceptable de vos risques identifiés. Ces points de référence font office de déclencheurs indiquant quand vous devez prendre des mesures pour atténuer ces risques. Voici quelques exemples montrant comment des métriques liées à l’identité, comme celles décrites ci-dessus, peuvent justifier un investissement accru dans la discipline Base de référence des identités.

- **Déclencheur de numéro de compte utilisateur**. Une entreprise avec un nombre d’utilisateurs, de groupes ou d’autres objets gérés supérieur à X dans vos systèmes d’identité peut tirer un bénéfice d’un investissement dans la discipline Base de référence des identités pour garantir une gouvernance efficace sur un grand nombre de comptes.
- **Déclencheur de dépendance d’identité locale**. Une entreprise prévoyant de migrer des charges de travail vers le cloud, qui nécessitent des fonctionnalités d’authentification héritées ou d’authentification multifacteur de tiers doit investir dans la discipline Base de référence des identités, afin de réduire les risques liés à la refactorisation ou au déploiement d’une infrastructure cloud supplémentaire.
- **Déclencheur de complexité des services d’annuaire**. Une entreprise gérant plus de X forêts individuelles, domaines ou locataires d’annuaire doit investir dans la discipline Base de référence des identités, afin de réduire les risques liés à la gestion des comptes et les problèmes d’efficacité liés à la multiplicité des informations d’identification utilisateur sur plusieurs systèmes.
- **Déclencheur de services d’annuaire hébergés dans le cloud**. Une entreprise hébergeant un nombre X de machines virtuelles qui sont des serveurs Active Directory hébergés dans le cloud ou qui ont un nombre X d’unités d’organisation gérées sur ces serveurs cloud, peut tirer un bénéfice d’un investissement dans la discipline Base de référence des identités pour optimiser l’intégration à des services d’identité locaux ou externes, quels qu’ils soient.
- **Déclencheur de fédération**. Une entreprise implémentant la fédération d’identités avec un nombre X de systèmes de Base de référence des identités peut tirer un bénéfice d’un investissement dans la discipline Base de référence des identités pour garantir une stratégie d’organisation cohérente entre les membres de la fédération.
- **Déclencheur d’accès avec privilèges élevés**. Une entreprise avec plus de X % des utilisateurs ayant des autorisations élevées pour des outils de gestion et des ressources peut envisager d’investir dans la discipline Base de référence des identités pour réduire le risque d’un provisionnement excessif par inadvertance de l’accès accordé aux utilisateurs.
- **Déclencheur RBAC**. Une entreprise avec moins de X % de ressources utilisant le contrôle d’accès en fonction du rôle (RBAC) peut envisager d’investir dans la discipline Base de référence des identités pour identifier des moyens optimisés d’affecter aux utilisateurs l’accès aux ressources.
- **Déclencheur d’échec d’authentification**. Une entreprise où les échecs d’authentification représentent plus de X % des tentatives peuvent investir dans la discipline Base de référence des identités pour garantir que les méthodes d’authentification ne sont pas soumises à des attaques externes et que les utilisateurs sont en mesure d’utiliser correctement les méthodes d’authentification.
- **Déclencheur d’échec d’autorisation**. Une entreprise où les tentatives d’accès sont rejetées dans une proportion supérieure à X % peut investir dans la discipline Base de référence des identités pour améliorer l’application et la mise à jour des contrôles d’accès, et pour identifier les tentatives d’accès potentiellement malveillantes.
- **Déclencheur de compte compromis**. Une entreprise avec plus de X comptes compromis peut investir dans la discipline Base de référence des identités pour améliorer la robustesse et la sécurité des mécanismes d’authentification, et pour améliorer les mécanismes d’atténuation des risques liés aux comptes compromis.

Les métriques et déclencheurs exacts à utiliser pour évaluer la tolérance au risque, et le niveau d’investissement dans la discipline Base de référence des identités sont spécifiques à votre organisation, mais les exemples ci-dessus devraient constituer une base utile pour la discussion au sein de votre équipe de gouvernance cloud.

## <a name="next-steps"></a>Étapes suivantes

À l’aide du [modèle de gestion cloud ](./template.md), documentez les métriques et les indicateurs de tolérance correspondant au plan d’adoption du cloud actuel.

Sur la base des risques et de la tolérance, établissez un processus de gouvernance et de communication en lien avec l’adhésion à la stratégie Base de référence des identités.

> [!div class="nextstepaction"]
> [Établir des processus d’adhésion à la stratégie](compliance-processes.md)
