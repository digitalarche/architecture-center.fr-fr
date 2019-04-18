---
title: Inscription et intégration de locataire dans des applications multi-locataires
description: Comment intégrer des locataires à une application multilocataire.
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: claims
pnp.series.next: app-roles
ms.openlocfilehash: eb4e65b20ec3339b633b65d2adad768e98d1bdbb
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640598"
---
# <a name="tenant-sign-up-and-onboarding"></a>Inscription et intégration de locataire

[![GitHub](../_images/github.png) Exemple de code][sample application]

Cet article décrit comment implémenter un processus d’*inscription* dans une application multi-locataire, ce qui permet à un client d’inscrire son organisation auprès de votre application.
Il existe plusieurs raisons d’implémenter un processus d’inscription :

* Autoriser un administrateur Active Directory à consentir que l’ensemble de l’organisation du client utilise l’application.
* Collecter des informations relatives au paiement par carte de crédit ou d’autres informations du client.
* Effectuer toute configuration par locataire en une seule fois nécessaire à votre application.

## <a name="admin-consent-and-azure-ad-permissions"></a>Consentement administrateur et autorisations Azure AD

Pour s’authentifier auprès d’Azure AD, une application doit accéder au répertoire de l’utilisateur. L’application doit avoir au minimum une autorisation de lecteur du profil utilisateur. La première fois qu’un utilisateur se connecte, Azure AD affiche une page de consentement qui répertorie les autorisations demandées. En cliquant sur **Accepter**, l’utilisateur accorde l’autorisation à l’application.

Par défaut, le consentement est accordé par utilisateur. Chaque utilisateur qui se connecte voit la page de consentement. Toutefois, Azure AD prend également en charge le *consentement administrateur*, qui permet à un administrateur Active Directory de donner son consentement pour l’intégralité d’une organisation.

Quand le flux de consentement administrateur est utilisé, la page de consentement indique que l’administrateur Active Directory accorde l’autorisation pour le compte de l’intégralité du locataire :

![Invite de consentement administrateur](./images/admin-consent.png)

Une fois que l’administrateur a cliqué sur **Accepter**, les autres utilisateurs appartenant au même locataire peuvent se connecter, et Azure AD omet alors l’écran de consentement.

Seul un administrateur Active Directory peut donner un consentement administrateur, car il accorde l’autorisation pour le compte de l’ensemble de l’organisation. Si un utilisateur non-administrateur tente de s’authentifier avec le flux de consentement administrateur, Azure AD affiche une erreur :

![Erreur de consentement](./images/consent-error.png)

Si l’application nécessite des autorisations supplémentaires ultérieurement, le client doit s’inscrire à nouveau et donner son consentement pour les autorisations mises à jour.

## <a name="implementing-tenant-sign-up"></a>Implémentation de l’inscription du locataire

Pour l’application [Tailspin Surveys][Tailspin], nous avons défini plusieurs conditions requises pour le processus d’inscription :

* Un locataire doit s’inscrire avant que des utilisateurs puissent se connecter.
* L’inscription utilise le flux de consentement administrateur.
* L’inscription ajoute le locataire de l’utilisateur à la base de données de l’application.
* Une fois un locataire inscrit, l’application affiche une page d’intégration.

Dans cette section, nous allons passer en revue notre implémentation de la procédure d’inscription.
Il est important de comprendre que l’opposition de l’« inscription » et de la « connexion » est un concept d’application. Pendant le flux d’authentification, Azure AD ne sais pas, de façon intrinsèque, si l’utilisateur est en cours d’inscription. C’est à l’application d’effectuer le suivi du contexte.

Quand un utilisateur anonyme visite l’application Surveys, deux boutons s’affichent à lui : un pour se connecter et un pour « abonner votre société » (inscription).

![Page d’inscription de l’application](./images/sign-up-page.png)

Ces boutons appellent des actions dans la classe `AccountController`.

Le `SignIn` action retourne un **ChallengeResult**, ce qui entraîne le middleware OpenID Connect rediriger vers le point de terminaison d’authentification. Il s’agit de la méthode par défaut pour déclencher l’authentification dans ASP.NET Core.

```csharp
[AllowAnonymous]
public IActionResult SignIn()
{
    return new ChallengeResult(
        OpenIdConnectDefaults.AuthenticationScheme,
        new AuthenticationProperties
        {
            IsPersistent = true,
            RedirectUri = Url.Action("SignInCallback", "Account")
        });
}
```

Comparez maintenant l’action `SignUp` :

```csharp
[AllowAnonymous]
public IActionResult SignUp()
{
    var state = new Dictionary<string, string> { { "signup", "true" }};
    return new ChallengeResult(
        OpenIdConnectDefaults.AuthenticationScheme,
        new AuthenticationProperties(state)
        {
            RedirectUri = Url.Action(nameof(SignUpCallback), "Account")
        });
}
```

Comme `SignIn`, l’action `SignUp` renvoie également `ChallengeResult`. Mais cette fois, nous ajoutons une information d’état pour `AuthenticationProperties` dans `ChallengeResult` :

* signup : indicateur booléen signalant que l’utilisateur a commencé le processus d’inscription.

Les informations d’état dans `AuthenticationProperties` sont ajoutées au paramètre [state] OpenID Connect, qui effectue des allers-retours pendant le flux d’authentification.

![Paramètre d’état](./images/state-parameter.png)

Une fois que l’utilisateur s’authentifie dans Azure AD et qu’il est redirigé vers l’application, le ticket d’authentification contient l’état. Nous utilisons cela pour garantir que la valeur « signup » est conservée dans l’intégralité du flux d’authentification.

## <a name="adding-the-admin-consent-prompt"></a>Ajout de l’invite de consentement administrateur

Dans Azure AD, le flux de consentement administrateur est déclenché en ajoutant un paramètre « prompt » à la chaîne de requête dans la demande d’authentification :

<!-- markdownlint-disable MD040 -->

```
/authorize?prompt=admin_consent&...
```

<!-- markdownlint-enable MD040 -->

L’application Surveys ajoute l’invite pendant l’événement `RedirectToAuthenticationEndpoint` . Cet événement est appelé juste avant que le middleware ne redirige vers le point de terminaison de l’authentification.

```csharp
public override Task RedirectToAuthenticationEndpoint(RedirectContext context)
{
    if (context.IsSigningUp())
    {
        context.ProtocolMessage.Prompt = "admin_consent";
    }

    _logger.RedirectToIdentityProvider();
    return Task.FromResult(0);
}
```

Le paramètre `ProtocolMessage.Prompt` indique au middleware d’ajouter le paramètre « prompt » à la demande d’authentification.

Notez que l’invite est nécessaire uniquement pendant l’inscription. Une connexion normale ne doit pas l’inclure. Pour faire la distinction entre les deux, nous vérifions la valeur `signup` dans l’état d’authentification. La méthode d’extension suivante vérifie cette condition :

```csharp
internal static bool IsSigningUp(this BaseControlContext context)
{
    Guard.ArgumentNotNull(context, nameof(context));

    string signupValue;
    // Check the HTTP context and convert to string
    if ((context.Ticket == null) ||
        (!context.Ticket.Properties.Items.TryGetValue("signup", out signupValue)))
    {
        return false;
    }

    // We have found the value, so see if it's valid
    bool isSigningUp;
    if (!bool.TryParse(signupValue, out isSigningUp))
    {
        // The value for signup is not a valid boolean, throw

        throw new InvalidOperationException($"'{signupValue}' is an invalid boolean value");
    }

    return isSigningUp;
}
```

## <a name="registering-a-tenant"></a>Inscription d’un locataire

L’application Surveys stocke certaines informations sur chaque locataire et utilisateur dans la base de données de l’application.

![Table Tenant](./images/tenant-table.png)

Dans la table Tenant, IssuerValue est la valeur de la revendication de l’émetteur pour le locataire. Pour Azure AD, cette valeur est `https://sts.windows.net/<tentantID>` et elle donne une valeur unique par client.

Lorsqu’un locataire se connecte, l’application Surveys écrit un enregistrement de locataire dans la base de données. Cela se produit au sein de l’événement `AuthenticationValidated`. (Ne le faites pas avant cet événement, car le jeton d’ID ne sera pas encore validé, et vous ne pourrez donc pas approuver les valeurs de revendication.) Voir [Authentification].

Voici le code approprié de l’application Surveys :

```csharp
public override async Task TokenValidated(TokenValidatedContext context)
{
    var principal = context.AuthenticationTicket.Principal;
    var userId = principal.GetObjectIdentifierValue();
    var tenantManager = context.HttpContext.RequestServices.GetService<TenantManager>();
    var userManager = context.HttpContext.RequestServices.GetService<UserManager>();
    var issuerValue = principal.GetIssuerValue();
    _logger.AuthenticationValidated(userId, issuerValue);

    // Normalize the claims first.
    NormalizeClaims(principal);
    var tenant = await tenantManager.FindByIssuerValueAsync(issuerValue)
        .ConfigureAwait(false);

    if (context.IsSigningUp())
    {
        if (tenant == null)
        {
            tenant = await SignUpTenantAsync(context, tenantManager)
                .ConfigureAwait(false);
        }

        // In this case, we need to go ahead and set up the user signing us up.
        await CreateOrUpdateUserAsync(context.Ticket, userManager, tenant)
            .ConfigureAwait(false);
    }
    else
    {
        if (tenant == null)
        {
            _logger.UnregisteredUserSignInAttempted(userId, issuerValue);
            throw new SecurityTokenValidationException($"Tenant {issuerValue} is not registered");
        }

        await CreateOrUpdateUserAsync(context.Ticket, userManager, tenant)
            .ConfigureAwait(false);
    }
}
```

Ce code effectue les actions suivantes :

1. Vérifier si la valeur d’émetteur du locataire se trouve déjà dans la base de données. Si le locataire ne s’est pas inscrit, `FindByIssuerValueAsync` renvoie la valeur null.
2. Si l’utilisateur s’inscrit :
   1. Ajouter le locataire à la base de données (`SignUpTenantAsync`).
   2. Ajouter l’utilisateur authentifié à la base de données (`CreateOrUpdateUserAsync`).
3. Sinon, exécuter le flux de connexion normal :
   1. Si l’émetteur du locataire est introuvable dans la base de données, cela signifie que le locataire n’est pas inscrit, et le client doit s’inscrire. Dans ce cas, lever une exception pour provoquer l’échec de l’authentification.
   2. Sinon, créer un enregistrement de base de données pour cet utilisateur, s’il n’existe pas déjà (`CreateOrUpdateUserAsync`).

Voici la méthode `SignUpTenantAsync` qui ajoute le locataire à la base de données.

```csharp
private async Task<Tenant> SignUpTenantAsync(BaseControlContext context, TenantManager tenantManager)
{
    Guard.ArgumentNotNull(context, nameof(context));
    Guard.ArgumentNotNull(tenantManager, nameof(tenantManager));

    var principal = context.Ticket.Principal;
    var issuerValue = principal.GetIssuerValue();
    var tenant = new Tenant
    {
        IssuerValue = issuerValue,
        Created = DateTimeOffset.UtcNow
    };

    try
    {
        await tenantManager.CreateAsync(tenant)
            .ConfigureAwait(false);
    }
    catch(Exception ex)
    {
        _logger.SignUpTenantFailed(principal.GetObjectIdentifierValue(), issuerValue, ex);
        throw;
    }

    return tenant;
}
```

Voici un résumé de l’intégralité du flux d’inscription dans l’application Surveys :

1. L’utilisateur clique sur le bouton **S’inscrire** .
2. Le `AccountController.SignUp` action retourne un résultat de test.  L’état d’authentification inclut la valeur de « signup ».
3. Dans l’événement `RedirectToAuthenticationEndpoint`, ajoutez l’invite `admin_consent`.
4. Le middleware OpenID Connect redirige vers Azure AD et l’utilisateur s’authentifie.
5. Dans l’événement `AuthenticationValidated` , recherchez l’état « signup ».
6. Ajoutez le locataire à la base de données.

[**Suivant**][app roles]

<!-- links -->

[app roles]: app-roles.md
[Tailspin]: tailspin.md

[state]: https://openid.net/specs/openid-connect-core-1_0.html#AuthRequest
[Authentification]: authenticate.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
