---
title: "Framework d'adoption du cloud : Exemples de déclarations de stratégie Base de référence des identités"
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Exemples de déclarations de stratégie Base de référence des identités
author: BrianBlanchard
ms.openlocfilehash: 5fad9265b9c048ee502c7e084ddd03faa0ad3e23
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901101"
---
# <a name="identity-baseline-sample-policy-statements"></a>Exemples de déclarations de stratégie Base de référence des identités

Les déclarations de stratégie cloud individuelles sont des recommandations destinées à traiter des risques spécifiques identifiés lors de votre processus d'évaluation des risques. Ces déclarations doivent fournir un bref récapitulatif des risques, ainsi que des plans établis pour les gérer. Chaque définition de déclaration doit inclure les informations suivantes :

- Risque technique : récapitulatif du risque à gérer par cette stratégie.
- Déclaration de stratégie : explication succincte et claire des exigences de la stratégie.
- Options de conception : recommandations exploitables, spécifications ou autres conseils que des équipes informatiques et des développeurs peuvent utiliser lors de l'implémentation de la stratégie.

Les exemples de déclarations de stratégie suivants gèrent un certain nombre des risques métier courants liés à l'identité. Ils sont fournis à des fins de référence lors de l'élaboration de déclarations de stratégie répondant aux besoins de votre organisation. Ces exemples ne sont pas destinés à être normatifs, et il existe potentiellement plusieurs options de stratégie pour la gestion de tout risque spécifique. Travaillez en étroite collaboration avec les équipes métier et informatiques pour identifier les meilleures solutions de stratégie pour votre ensemble de risques unique.

## <a name="lack-of-access-controls"></a>Absence de contrôles d'accès

**Risque technique** : des paramètres de contrôle d'accès insuffisants ou ad hoc peuvent introduire un risque d'accès non autorisé à des ressources sensibles ou stratégiques.

**Déclaration de stratégie** : toutes les ressources déployées dans le cloud doivent être contrôlées à l'aide d'identités et de rôles approuvés par les stratégies de gouvernance en vigueur.

**Options de conception potentielles** : l'[Accès conditionnel Azure Active Directory](/azure/active-directory/conditional-access/overview) est le mécanisme de contrôle d'accès par défaut dans Azure.

## <a name="overprovisioned-access"></a>Accès surapprovisionné

**Risque technique** : les utilisateurs et les groupes qui contrôlent des ressources situées au-delà de leur domaine de responsabilité peuvent provoquer des modifications non autorisées conduisant à des pannes ou à des vulnérabilités en matière de sécurité.

**Déclaration de stratégie** : les stratégies suivantes seront implémentées.

- Un modèle d'accès à privilège minimum sera appliqué à toutes les ressources impliquées dans des applications stratégiques ou des données protégées.
- Les autorisations élevées doivent constituer une exception, et toute exception de ce type doit être communiquée à l'équipe de gouvernance cloud. Les exceptions seront régulièrement vérifiées.

**Options de conception potentielles** : consultez les [Meilleures pratiques d'Azure Identity Management](/azure/security/azure-security-identity-management-best-practices) pour implémenter une stratégie de contrôle d'accès en fonction du rôle (RBAC) qui restreint l'accès selon les principes [à savoir](https://wikipedia.org/wiki/Need_to_know) et [sécurité à privilège minimum](https://wikipedia.org/wiki/Principle_of_least_privilege).

## <a name="lack-of-shared-management-accounts-between-on-premises-and-the-cloud"></a>Absence de comptes de gestion partagés entre l'infrastructure locale et le cloud

**Risque technique** : les membres du personnel de gestion ou d'administration informatique qui possèdent un compte au sein de votre infrastructure Active Directory locale peuvent ne pas disposer d'un accès suffisant aux ressources du cloud et ne pas être en mesure de résoudre efficacement les problèmes opérationnels ou de sécurité.

**Déclaration de stratégie** : tous les groupes de l'infrastructure Active Directory locale qui disposent de privilèges élevés doivent être mappés vers un rôle RBAC approuvé.

**Options de conception potentielles** : implémentez une solution d'identité hybride entre votre infrastructure Azure Active Directory basée sur le cloud et votre infrastructure Active Directory locale, et ajoutez les groupes locaux requis aux rôles RBAC nécessaires pour leur permettre d'accomplir leur travail.

## <a name="weak-authentication-mechanisms"></a>Mécanismes d'authentification faibles

**Risque technique** : lorsque les systèmes de gestion des identités utilisent des méthodes d'authentification des utilisateurs qui ne sont pas suffisamment sécurisées (combinaisons utilisateur/mot de passe de base, par exemple), les mots de passe peuvent être compromis ou piratés, ce qui représente un risque majeur d'accès non autorisé aux systèmes cloud sécurisés.

**Déclaration de stratégie** : tous les comptes sont tenus de se connecter aux ressources sécurisées à l'aide d'une méthode d'authentification multifacteur (MFA).

**Options de conception potentielles** : pour Azure Active Directory, implémentez [Azure Multi-Factor Authentication](/azure/active-directory/authentication/concept-mfa-howitworks) dans le cadre de votre processus d'autorisation utilisateur.

## <a name="isolated-identity-providers"></a>Fournisseurs d'identité isolés

**Risque technique** : des fournisseurs d'identité incompatibles peuvent empêcher le partage de ressources ou de services avec des clients ou d'autres partenaires commerciaux.

**Déclaration de stratégie** : le déploiement des applications nécessitant une authentification des clients doit utiliser un fournisseur d'identité approuvé compatible avec le fournisseur d'identité principal pour les utilisateurs internes.

**Options de conception potentielles** : implémentez la [Fédération avec Azure Active Directory](/azure/active-directory/hybrid/whatis-fed) entre vos fournisseurs d'identité interne et client.

## <a name="identity-reviews"></a>Révisions des identités

**Risque technique** : au fil du temps, l'ajout de nouveaux déploiements dans le cloud ou l'apparition d'autres problèmes de sécurité peuvent accroître les risques d'accès non autorisé à des ressources sécurisées.

**Déclaration de stratégie** : les processus de gouvernance cloud doivent inclure des révisions trimestrielles avec les équipes de gestion des indentés. L'objectif est d'identifier les modèles d'utilisation ou les acteurs malveillants devant être bloqués par la configuration des ressources cloud.

**Options de conception potentielles** : tous les trimestres, organisez une réunion destinée à la révision de la sécurité avec les membres de l'équipe de gouvernance et le personnel informatique chargé de la gestion des services d'identité. Passez en revue les données et métriques de sécurité existantes pour déterminer les lacunes de la stratégie et des outils de gestion des identités actuels, ainsi que pour mettre à jour la stratégie afin d'atténuer les nouveaux risques éventuels.

## <a name="next-steps"></a>Étapes suivantes

Utilisez les exemples mentionnés dans cet article comme point de départ pour développer des stratégies qui répondent à des risques métier spécifiques, dans la continuité de vos plans d'adoption du cloud.

Pour commencer à développer vos déclarations de stratégie personnalisées pour la Base de référence des identités, téléchargez le [modèle Base de référence des identités](template.md).

Pour accélérer l'adoption de cette discipline, choisissez le [parcours de gouvernance exploitable](../journeys/overview.md) correspondant le mieux à votre environnement. Modifiez ensuite la conception pour intégrer vos décisions spécifiques en matière de stratégie d'entreprise.

> [!div class="nextstepaction"]
> [Parcours de gouvernance exploitables](../journeys/overview.md)