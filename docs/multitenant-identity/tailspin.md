---
title: À propos de l’application Tailspin Surveys
description: Vue d’ensemble de l’application Tailspin Surveys.
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.openlocfilehash: de2830b5c492e027c189a79e45ccc6634cab436b
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54483222"
---
# <a name="the-tailspin-scenario"></a>Le scénario Tailspin

[![GitHub](../_images/github.png) Exemple de code][sample application]

Tailspin est une société fictive qui développe une application SaaS nommée Surveys. Cette application permet aux organisations de créer et de publier des enquêtes en ligne.

* Une organisation peut s’inscrire auprès de l’application.
* Une fois que l’organisation est inscrite, les utilisateurs peuvent se connecter à l’application avec leurs informations d’identification professionnelles.
* Les utilisateurs peuvent créer, modifier et publier des enquêtes.

> [!NOTE]
> Pour vous familiariser avec l’application, consultez [Exécution de l’application Surveys].

## <a name="users-can-create-edit-and-view-surveys"></a>Les utilisateurs peuvent créer, modifier et consulter des enquêtes.

Un utilisateur authentifié peut consulter toutes les enquêtes qu’il a créées ou pour lesquelles il détient des droits de contributeur. Il peut également créer des enquêtes. Notez que l’utilisateur est connecté à l’aide de son identité professionnelle, `bob@contoso.com`.

![Application Surveys](./images/surveys-screenshot.png)

Cette capture d’écran illustre la page Edit Survey :

![Modifier l’enquête](./images/edit-survey.png)

Les utilisateurs peuvent aussi consulter les enquêtes créées par d’autres utilisateurs au sein du même locataire.

![Enquêtes client](./images/tenant-surveys.png)

## <a name="survey-owners-can-invite-contributors"></a>Les propriétaires d’enquêtes peuvent inviter des contributeurs.

Lorsqu’un utilisateur crée une enquête, il peut inviter d’autres personnes à devenir des collaborateurs sur l’enquête. Les collaborateurs peuvent modifier l’enquête, mais ils ne peuvent pas la supprimer, ni la publier.

![Ajouter un collaborateur](./images/add-contributor.png)

Un utilisateur peut ajouter des collaborateurs à partir d’autres clients, ce qui permet le partage des ressources entre locataires. Dans cette capture d’écran, Bob (`bob@contoso.com`) ajoute Alice (`alice@fabrikam.com`) en tant que contributeur à une enquête que Bob a créée.

Lorsqu’Alice se connecte, elle voit l’enquête répertoriée sous « Surveys I can contribute to » (Enquêtes auxquelles je peux contribuer).

![Collaborateur de l’enquête](./images/contributor.png)

Notez qu’Alice se connecte à son propre locataire et non en tant qu’invité du locataire Contoso. Alice a des autorisations de contributeur uniquement pour cette enquête &mdash; elle ne peut pas consulter les autres enquêtes du locataire Contoso.

## <a name="architecture"></a>Architecture

L’application Surveys se compose d’un serveur web frontal et d’un serveur principal d’API web. Les deux sont implémentés à l’aide d’[ASP.NET Core].

L’application web utilise Azure Active Directory (Azure AD) pour authentifier les utilisateurs. L’application web appelle également Azure AD pour obtenir des jetons d’accès OAuth 2 pour l’API web. Les jetons d’accès sont mis en cache dans le Cache Redis Azure. Le cache permet à plusieurs instances de partager le même cache de jeton (par exemple, dans une batterie de serveurs).

![Architecture](./images/architecture.png)

[**Suivant**][authentication]

<!-- links -->

[authentication]: authenticate.md

[Exécution de l’application Surveys]: ./run-the-app.md
[ASP.NET Core]: /aspnet/core
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
