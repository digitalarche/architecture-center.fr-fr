---
title: Modèle d’identité fédérée
titleSuffix: Cloud Design Patterns
description: Déléguez l’authentification à un fournisseur d’identité externe.
keywords: modèle de conception
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: f4e8d2dff1c653cf2b8ef0109f1cc3027e5428b9
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58242320"
---
# <a name="federated-identity-pattern"></a>Modèle d’identité fédérée

[!INCLUDE [header](../_includes/header.md)]

Déléguez l’authentification à un fournisseur d’identité externe. Cette approche permet de simplifier le développement, de réduire au minimum la nécessité d’administrer les utilisateurs et d’améliorer l’expérience utilisateur de l’application.

## <a name="context-and-problem"></a>Contexte et problème

En règle générale, les utilisateurs ont besoin de recourir à plusieurs applications fournies et hébergées par différentes organisations avec lesquelles ils ont une relation professionnelle. Ces utilisateurs peuvent être amenés à utiliser des informations d’identification spécifiques (et différentes) pour chacune d’elles. Les conséquences peuvent être les suivantes :

- **Dissociation de l’expérience utilisateur**. Les utilisateurs oublient souvent leurs informations d’identification de connexion si elles diffèrent d’une application à l’autre.

- **Exposition des failles de sécurité**. Quand un utilisateur quitte la société, le compte doit immédiatement être mis hors service. Il est facile d’oublier cela dans les grandes organisations.

- **Complication de la gestion des utilisateurs**. Les administrateurs doivent gérer les informations d’identification pour tous les utilisateurs et effectuer d’autres tâches telles que fournir des rappels de mot de passe.

Les utilisateurs préfèrent généralement utiliser les mêmes informations d’identification pour toutes ces applications.

## <a name="solution"></a>Solution

Implémentez un mécanisme d’authentification pouvant utiliser l’identité fédérée. Séparez l’authentification des utilisateurs du code d’application et déléguez l’authentification à un fournisseur d’identité approuvé. Vous pouvez ainsi simplifier le développement et permettre aux utilisateurs de s’authentifier à l’aide d’un large éventail de fournisseurs d’identité (IdP) tout en réduisant les coûts d’administration. En outre, vous pouvez séparer clairement l’authentification de l’autorisation.

Les fournisseurs d’identité approuvés incluent les annuaires d’entreprise, les services de fédération locaux et autres services de jeton de sécurité (STS) fournis par des partenaires commerciaux ou des fournisseurs d’identité sociaux qui peuvent authentifier les utilisateurs disposant d’un compte Microsoft, Google, Yahoo! ou Facebook, par exemple.

La figure illustre le modèle d’identité fédérée quand une application cliente doit accéder à un service qui requiert une authentification. L’authentification est effectuée par un IdP qui fonctionne conjointement avec un STS. L’IdP émet des jetons de sécurité qui fournissent des informations sur l’utilisateur authentifié. Ces informations, appelées revendications, incluent l’identité de l’utilisateur et, éventuellement, d’autres informations telles que l’appartenance au rôle et des droits d’accès plus précis.

![Vue d’ensemble de l’authentification fédérée](./_images/federated-identity-overview.png)

Ce modèle est souvent appelé contrôle d’accès basé sur les revendications. Les applications et services autorisent l’accès aux fonctions et fonctionnalités d’après les revendications contenues dans le jeton. Le service qui requiert une authentification doit approuver l’IdP. L’application cliente contacte l’IdP qui effectue l’authentification. Si l’authentification réussit, l’IdP retourne un jeton contenant les revendications qui identifient l’utilisateur au STS (notez que l’IdP et le STS peuvent être le même service). Le STS peut transformer et augmenter les revendications dans le jeton en fonction de règles prédéfinies, avant de le retourner au client. L’application cliente peut ensuite passer ce jeton au service en tant que preuve de son identité.

> La chaîne d’approbation peut comporter des STS supplémentaires. Par exemple, dans le scénario décrit plus loin, un STS local approuve un autre STS qui est responsable de l’accès à un fournisseur d’identité pour authentifier l’utilisateur. Cette approche est courante dans les scénarios d’entreprise où il existe un annuaire et un STS locaux.

L’authentification fédérée fournit une solution basée sur des normes au problème d’approbation d’identités entre différents domaines et peut prendre en charge l’authentification unique. Elle se développe parmi tous les types d’applications, en particulier les applications hébergées dans le cloud, car elle prend en charge l’authentification unique sans nécessiter une connexion réseau directe aux fournisseurs d’identité. L’utilisateur n’a pas besoin d’entrer d’informations d’identification pour chaque application. Cela accroît la sécurité, en empêchant la création d’informations d’identification nécessaires pour accéder à de nombreuses applications différentes et masque les informations d’identification de l’utilisateur à la vue de tous, sauf du fournisseur d’identité d’origine. Les applications ne voient que les informations d’identité authentifiées contenues dans le jeton.

En outre, l’identité fédérée a l’avantage majeur que la gestion de l’identité et des informations d’identification revient au fournisseur d’identité. L’application ou le service n’a pas besoin de fournir de fonctionnalités de gestion d’identité. En outre, dans les scénarios d’entreprise, l’annuaire d’entreprise n’a pas besoin de connaître l’utilisateur s’il approuve le fournisseur d’identité. Ainsi, l’annuaire se voit déchargé de toutes les tâches administratives liées à la gestion de l’identité de l’utilisateur.

## <a name="issues-and-considerations"></a>Problèmes et considérations

Considérez les éléments suivants quand vous concevez des applications qui implémentent l’authentification fédérée :

- L’authentification peut être un point de défaillance unique. Si vous déployez votre application sur plusieurs centres de données, envisagez de déployer votre mécanisme de gestion des identités sur les mêmes centres de données pour assurer la disponibilité et la fiabilité des applications.

- Les outils d’authentification permettent de configurer le contrôle d’accès en fonction des revendications de rôle contenues dans le jeton d’authentification. Cette approche est souvent appelée contrôle d’accès en fonction du rôle (RBAC) et peut permettre un niveau de contrôle plus précis de l’accès aux fonctionnalités et aux ressources.

- Contrairement à un annuaire d’entreprise, l’authentification basée sur les revendications à l’aide de fournisseurs d’identité sociaux ne fournit généralement pas d’informations sur l’utilisateur authentifié, à part une adresse e-mail et, éventuellement, un nom. Certains fournisseurs d’identité sociaux, tels qu’un compte Microsoft, ne fournissent qu’un identificateur unique. Généralement, l’application doit conserver certaines informations sur les utilisateurs inscrits et être capable de faire correspondre ces informations à l’identificateur contenu dans les revendications au sein du jeton. En général, cette opération est effectuée par le biais de l’inscription quand l’utilisateur accède pour la première fois à l’application, puis les informations sont injectées dans le jeton en tant que revendications supplémentaires après chaque authentification.

- Si plusieurs fournisseurs d’identité sont configurés pour le STS, il doit détecter le fournisseur d’identité vers lequel l’utilisateur doit être redirigé pour l’authentification. Ce processus est appelé découverte de domaine d’accueil. Le STS peut être en mesure d’effectuer cette opération automatiquement à partir d’une adresse e-mail ou d’un nom d’utilisateur fourni par l’utilisateur, d’un sous-domaine de l’application à laquelle accède l’utilisateur, de l’étendue des adresses IP de l’utilisateur ou du contenu d’un cookie stocké dans le navigateur de l’utilisateur. Par exemple, si l’utilisateur a entré une adresse e-mail dans le domaine Microsoft, comme user@live.com, le STS redirige l’utilisateur vers la page de connexion au compte Microsoft. À l’occasion des visites ultérieures, le STS peut utiliser un cookie pour indiquer que la dernière connexion s’est effectuée avec un compte Microsoft. Si la découverte automatique ne peut pas déterminer le domaine d’accueil, le STS affiche une page de découverte de domaine d’accueil qui répertorie les fournisseurs d’identité approuvés, et l’utilisateur doit sélectionner celui qu’il souhaite utiliser.

## <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle

Ce modèle est utile pour les scénarios tels que les suivants :

- **Authentification unique dans l’entreprise**. Dans ce scénario, vous devez authentifier les employés pour les applications d’entreprise hébergées dans le cloud hors de la limite de sécurité de l’entreprise, sans qu’ils aient besoin de se connecter chaque fois qu’ils accèdent à une application. L’expérience utilisateur est la même que durant l’utilisation d’applications locales, configuration dans laquelle ils sont authentifiés quand ils se connectent à un réseau d’entreprise, puis ont accès à toutes les applications appropriées sans avoir à se reconnecter.

- **Identité fédérée avec plusieurs partenaires**. Dans ce scénario, vous devez authentifier les employés de l’entreprise et les partenaires commerciaux qui ne disposent pas de comptes dans l’annuaire d’entreprise. Ce scénario est courant dans les applications B2B, les applications qui s’intègrent à des services tiers et dans les situations où des entreprises dotées de différents systèmes informatiques ont des ressources partagées ou fusionnées.

- **Identité fédérée dans les applications SaaS**. Dans ce scénario, les éditeurs de logiciels indépendants proposent des services prêts à l’emploi pour plusieurs clients ou locataires. Chaque locataire s’authentifie à l’aide d’un fournisseur d’identité approprié. Par exemple, les utilisateurs professionnels utilisent leurs informations d’identification d’entreprise, tandis que les consommateurs et les clients du locataire utilisent leurs informations d’identification d’identité sociale.

Ce modèle peut s’avérer inutile dans les situations suivantes :

- Tous les utilisateurs de l’application peuvent être authentifiés par le même fournisseur d’identité. Cette situation est courante dans les applications métier qui ont recours à un annuaire d’entreprise (accessible dans l’application) pour l’authentification, en utilisant un VPN ou, dans un scénario hébergé sur le cloud, en utilisant une connexion de réseau virtuel entre l’annuaire local et l’application.

- L’application a initialement été créée à l’aide d’un autre mécanisme d’authentification, peut-être avec des magasins d’utilisateurs personnalisés, ou n’a pas la capacité de gérer les standards de négociation utilisés par les technologies basées sur les revendications. L’introduction de l’authentification basée sur les revendications et du contrôle d’accès dans des applications existantes peut être complexe et probablement non rentable.

## <a name="example"></a>Exemples

Une organisation héberge une application SaaS (Software as a Service) multi-locataire dans Microsoft Azure. L’application comprend un site web que les locataires peuvent utiliser pour gérer celle-ci pour leurs propres utilisateurs. L’application permet aux locataires d’accéder au site web à l’aide d’une identité fédérée qui est générée par Active Directory Federation Services (ADFS) quand un utilisateur est authentifié par l’annuaire Active Directory de cette organisation.

![Comment les utilisateurs au sein d’un grand abonné d’entreprise accèdent à l’application](./_images/federated-identity-multitenat.png)

L’illustration montre comment les locataires s’authentifient auprès de leur propre fournisseur d’identité (étape 1), en l’occurrence ADFS. Après avoir authentifié un locataire, ADFS émet un jeton. Le navigateur client transmet ce jeton au fournisseur de fédération de l’application SaaS, qui approuve les jetons émis par le service ADFS du locataire, afin de récupérer un jeton qui est valide pour le fournisseur de fédération SaaS (étape 2). Si nécessaire, le fournisseur de fédération SaaS transforme les revendications dans le jeton en revendications que l’application reconnaît (étape 3) avant de retourner le nouveau jeton au navigateur client. L’application approuve les jetons émis par le fournisseur de fédération SaaS et utilise les revendications dans le jeton pour appliquer des règles d’autorisation (étape 4).

Les locataires n’ont pas besoin de mémoriser des informations d’identification distinctes pour accéder à l’application, et un administrateur au sein de l’entreprise du locataire peut configurer dans son propre service ADFS la liste des utilisateurs qui peuvent accéder à l’application.

## <a name="related-guidance"></a>Aide connexe

- [Microsoft Azure Active Directory](https://azure.microsoft.com/services/active-directory/)
- [Services de domaine Active Directory](https://msdn.microsoft.com/library/bb897402.aspx)
- [Active Directory Federation Services](https://msdn.microsoft.com/library/bb897402.aspx)
- [Gestion des identités pour les applications multi-locataires dans Microsoft Azure](/azure/architecture/multitenant-identity)
- [Applications multi-locataires dans Azure](/azure/dotnet-develop-multitenant-applications)
