---
title: 'Framework d’adoption du cloud : Guide de décision concernant l’identité'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Découvrez comment utiliser l’identité comme un service principal lors des migrations vers Azure.
author: rotycenh
ms.openlocfilehash: addc11928a0bc72917a28b468a04880720a1fdf4
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900905"
---
# <a name="identity-decision-guide"></a>Guide de décision concernant l’identité

Dans tous les environnements (local, hybride ou cloud uniquement), le service informatique doit contrôler les administrateurs, les utilisateurs et les groupes qui ont accès aux ressources. Les services de gestion des identités et des accès (IAM) vous permettent de gérer le contrôle des accès dans le cloud.

![Options d’identité, de la moins complexe à la plus complexe, dans l’ordre des liens ci-dessous](../../_images/discovery-guides/discovery-guide-identity.png)

Passer à : [Déterminer les exigences d’intégration des identités](#determine-identity-integration-requirements) | [Cloud natif](#cloud-baseline) | [Synchronisation d’annuaires](#directory-synchronization) | [Services de domaines hébergés dans le cloud](#cloud-hosted-domain-services) | [Active Directory Federation Services](#active-directory-federation-services) | [Évolution de l’intégration des identités](#evolving-identity-integration) | [En savoir plus](#learn-more)

Plusieurs méthodes permettent de gérer les identités dans un environnement cloud. Leur coût et leur complexité sont variables. Le niveau d’intégration requis avec votre infrastructure d’identité sur site existante est un facteur clé dans la structuration de vos services d’identité cloud.

Les solutions d’identité software as a service (SaaS) basées dans le cloud fournissent un niveau de base pour contrôler les accès et gérer les identités relatives aux ressources cloud. Toutefois, si l’infrastructure Active Directory (AD) de votre organisation présente une structure en forêt complexe ou des unités d’organisation personnalisées, il se peut que vos charges de travail basées sur le cloud nécessitent la duplication d’annuaire vers le cloud pour bénéficier d’un ensemble d’identités, de groupes et de rôles cohérent entre vos environnements en local et cloud. Si la duplication d’annuaire est requise pour une solution globale, cela peut entraîner une plus grande complexité. En outre, pour garantir la prise en charge des applications qui dépendent de mécanismes d’authentification héritée, il se peut que vous deviez déployer des services de domaine dans le cloud.

## <a name="determine-identity-integration-requirements"></a>Déterminer les exigences d’intégration des identités

| Question | Base de référence cloud | Synchronisation de répertoires | Services de domaines hébergés dans le cloud | Services de fédération AD FS |
|------|------|------|------|------|
| À l’heure actuelle, vous manque-t-il un service d’annuaire local ? | OUI | Non  | Non  | Non  |
| Vos charges de travail doivent-elles s’authentifier auprès des services d’identité locaux ? | Non  | OUI | Non  | Non  |
| Vos charges de travail dépendent-elles des mécanismes d’authentification hérités, tels NTLM ou Kerberos ? | Non  | Non  | OUI | Non  |
| L’intégration entre le service d’identité local et celui sur le cloud est-elle impossible ? | Non  | Non  | OUI | Non  |
| Avez-vous besoin de l’authentification unique auprès de plusieurs fournisseurs d’identité ? | Non  | Non  | Non  | OUI |

Dans le cadre de la planification de votre migration vers Azure, vous devez déterminer la meilleure méthode pour intégrer votre gestion des identités et les services d’identité cloud. Les paragraphes suivants illustrent des scénarios d’intégration courants.

### <a name="cloud-baseline"></a>Base de référence cloud

Les plateformes de cloud public proposent un système IAM natif permettant d’accorder l’accès des utilisateurs et des groupes aux fonctionnalités d’administration. Si votre organisation ne dispose pas de solution d’identité locale notable et que vous prévoyez de migrer des charges de travail afin qu’elles soient compatibles avec les mécanismes d’authentification basés sur le cloud, vous devez créer votre infrastructure d’identité à l’aide d’un service d’identité cloud natif.

**Hypothèses relatives à la base de référence cloud**. L’utilisation d’une infrastructure d’identité cloud native (uniquement) implique les hypothèses suivantes :

- Vos ressources basées sur le cloud ne peuvent présenter aucune dépendance avec les services d’annuaire locaux ni les serveurs Active Directory, ou bien les charges de travail doivent être modifiées pour supprimer ces dépendances.
- Les charges de travail d’application ou de service qui sont en cours de migration doivent prendre en charge les mécanismes d’authentification compatibles avec les fournisseurs d’identité cloud ou bien elles peuvent être modifiées facilement pour les prendre en charge. Les fournisseurs d’identité cloud natifs s’appuient sur des mécanismes d’authentification prêts à l’emploi (SAML, OAuth et OpenID Connect, par exemple). Il se peut que les charges de travail existantes qui dépendent de méthodes d’authentification héritées et qui utilisent des protocoles tels que NTLM ou Kerberos doivent être refactorisées avant de migrer vers le cloud.

> [!TIP]
> La plupart des services d’identité cloud natifs ne remplacent pas complètement les répertoires locaux traditionnels. Il se peut que les fonctionnalités de répertoire (gestion de l’ordinateur ou stratégie de groupe, par exemple) ne soient pas disponibles sans passer par des outils ou des services supplémentaires.

En migrant complètement vos services d’identité vers un fournisseur cloud, vous éliminez le besoin de maintenir votre propre infrastructure d’identité et simplifiez considérablement votre gestion informatique.

### <a name="directory-synchronization"></a>Synchronisation de répertoires

Pour les organisations qui disposent déjà d’une infrastructure d’identité, la synchronisation d’annuaires est souvent la meilleure solution pour conserver la gestion des accès et des utilisateurs existante tout en bénéficiant des capacités IAM requises pour gérer les ressources cloud. Ce processus permet de dupliquer les informations d’annuaire entre le cloud et les environnements locaux en continu, ce qui permet l’authentification unique (SSO) des utilisateurs et la mise en œuvre d’un système d’identité, de rôle et d’autorisation cohérent dans toute l’organisation.

Remarque : Les organisations qui ont adopté Office 365 ont peut-être déjà implémenté [la synchronisation d’annuaires](/office365/enterprise/set-up-directory-synchronization) entre leur infrastructure locale Active Directory et Azure Active Directory.

**Hypothèses relatives à la synchronisation d’annuaires**. L’utilisation d’une solution d’identité synchronisée implique les hypothèses suivantes :

- Vous devez maintenir un ensemble commun de comptes utilisateur et de groupes entre le cloud et votre infrastructure informatique locale.
- Vos services d’identité locaux prennent en charge la réplication avec votre fournisseur d’identité cloud.
- Vous avez besoin de mécanismes d’authentification unique SSO pour les utilisateurs qui accèdent à des fournisseurs d’identité sur le cloud et en local.

> [!TIP]
> Toutes les charges de travail basées sur le cloud et dépendantes de mécanismes d’authentification hérités non pris en charge par les services d’identité basés sur le cloud (comme Azure AD) doivent rester connectées aux services de domaine locaux ou aux serveurs virtuels dans l’environnement cloud qui fournit ces services. L’utilisation de services d’identité locaux introduit également des dépendances de connectivité entre les réseaux cloud et locaux.

### <a name="cloud-hosted-domain-services"></a>Services de domaines hébergés dans le cloud

Si vous disposez de charges de travail qui dépendent d’authentification par revendication à l’aide de protocoles hérités (Kerberos ou NTLM, par exemple), et qu’il est impossible de refactoriser ces charges de travail pour accepter des protocoles d’authentification modernes (tels que SAML, OAuth et OpenID Connect), vous devrez peut-être migrer certains de vos services de domaine vers le cloud dans le cadre de votre déploiement cloud.

Ce type de déploiement implique de déployer des machines virtuelles exécutant Active Directory dans vos réseaux virtuels basés sur le cloud pour fournir des services de domaine aux ressources dans le cloud. Toutes les applications et services existants qui migrent vers votre réseau cloud doivent être en mesure d’utiliser ces serveurs d’annuaires hébergés dans le cloud seulement avec quelques modifications mineures.

Il se peut que vos services de domaine et de répertoires existants soient toujours utilisés dans votre environnement local. Dans ce scénario, nous vous recommandons d’utiliser également la synchronisation d’annuaires pour fournir un ensemble d’utilisateurs et de rôles commun dans les environnements cloud et local.

**Hypothèses relatives aux services de domaines hébergés dans le cloud**. L’exécution d’une migration de répertoire implique les hypothèses suivantes :

- Vos charges de travail dépendent de l’authentification par revendication à l’aide de protocoles tels que Kerberos ou NTLM.
- Vos machines virtuelles de charge de travail doivent être jointes au domaine afin d’assurer la gestion ou l’application des objectifs stratégiques de groupe Active Directory.

> [!TIP]
> La migration de répertoire associée à des services de domaine hébergés dans le cloud offre une grande souplesse lors de la migration des charges de travail existantes. Toutefois, l’hébergement de machines virtuelles dans votre réseau virtuel cloud (pour fournir ces services) augmente la complexité de vos tâches d’administration informatique. Au fur et à mesure que votre expérience de migration cloud gagne en maturité, examinez les besoins de maintenance à long terme associés à l’hébergement de ces serveurs. Déterminez si la refactorisation des charges de travail existantes pour les rendre compatibles avec les fournisseurs d’identité cloud (comme Azure Active Directory) peut réduire les besoins de ces serveurs hébergés dans le cloud.

### <a name="active-directory-federation-services"></a>Services de fédération Active Directory (AD FS)

La fédération des identités établit des relations d’approbation entre plusieurs systèmes de gestion des identités afin de permettre l’authentification et des fonctionnalités d’autorisation communes. Vous pouvez ensuite prendre en charge les fonctionnalités d’authentification unique sur plusieurs domaines dans votre organisation ou les systèmes d’identité gérés par vos clients ou partenaires commerciaux.

Azure AD prend en charge la fédération de domaines Active Directory locaux à l’aide d’[Active Directory Federation Services](/azure/active-directory/hybrid/how-to-connect-fed-whatis) (AD FS). Consultez l’architecture de référence [Extension d’AD FS vers Azure](../../../reference-architectures/identity/adfs.md) pour en savoir plus sur l’implémentation dans Azure.

## <a name="evolving-identity-integration"></a>Évolution de l’intégration des identités

L’intégration des identités est un processus itératif. Dans un premier temps, il se peut que vous souhaitiez débuter par une solution cloud native avec un petit ensemble d’utilisateurs et de rôles correspondants pour un déploiement initial. Votre migration gagnant en maturité, vous envisagez d’adopter un modèle fédéré ou d’effectuer une migration de répertoire complète de vos services d’identité locaux vers le cloud. Réétudiez votre stratégie d’identité dans chaque itération du processus de migration.

## <a name="learn-more"></a>En savoir plus

Pour plus d’informations sur les services d’identité de la plateforme Azure, consultez la rubrique suivante.

- [Azure AD](https://azure.microsoft.com/services/active-directory). Azure AD fournit un service d’identité basé sur le cloud. Il vous permet de gérer l’accès à vos ressources Azure et de contrôler la gestion des identités, l’inscription des appareils, l’approvisionnement des utilisateurs, le contrôle de l’accès aux applications et la protection des données.
- [Azure AD Connect](/azure/active-directory/hybrid/whatis-hybrid-identity). L’outil Azure AD Connect vous permet de connecter des instances Azure AD avec vos solutions de gestion des identités existantes, en autorisant la synchronisation de votre répertoire existant avec le cloud.
- [Contrôle d’accès en fonction du rôle](/azure/role-based-access-control/overview) (RBAC). Azure AD fournit le contrôle des accès en fonction du rôle pour gérer efficacement et en toute sécurité l’accès aux ressources dans le plan de gestion. Les travaux et les responsabilités sont classés par rôle, et les utilisateurs sont assignés à ces rôles. Le RBAC vous permet de contrôler les utilisateurs qui ont accès à une ressource, ainsi que les actions qu’ils peuvent effectuer sur cette ressource.
- [Azure AD Privileged Identity Management](/azure/active-directory/privileged-identity-management/pim-configure) (PIM). PIM réduit le temps d’exposition des privilèges d’accès aux ressources et augmente votre visibilité en matière d’utilisation grâce à des rapports et des alertes. PIM autorise les utilisateurs à utiliser ces privilèges uniquement pendant une période précise (juste-à-temps (JIT)), ou en leur attribuant des privilèges pour une plus courte durée après laquelle ces privilèges sont automatiquement révoqués.
- [Intégrer des domaines Active Directory locaux avec Azure Active Directory](../../../reference-architectures/identity/azure-ad.md). Cette architecture de référence fournit un exemple de synchronisation d’annuaires entre des domaines Active Directory locaux et Azure AD.
- [Étendre Active Directory Domain Services (AD DS) à Azure](../../../reference-architectures/identity/adds-extend-domain.md). Cette architecture de référence fournit un exemple de déploiement de serveurs AD DS pour étendre les services de domaine aux ressources basées sur le cloud.
- [Étendre les services de fédération Active Directory (AD FS) à Azure](../../../reference-architectures/identity/adfs.md). Cette architecture de référence configure Active Directory Federation Services (ADFS) afin d’effectuer l’authentification fédérée et l’autorisation auprès de votre répertoire Azure AD.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment implémenter l’application des stratégies dans le cloud.

> [!div class="nextstepaction"]
> [Application de stratégies](../policy-enforcement/overview.md)
