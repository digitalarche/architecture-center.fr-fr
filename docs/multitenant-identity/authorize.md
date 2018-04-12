---
title: Autorisation dans les applications multi-locataires
description: Comment déclarer des autorisations dans une application multi-locataire
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: app-roles
pnp.series.next: web-api
ms.openlocfilehash: 03c4d5fa10c75437a7b066534619ba9a123c350c
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/06/2018
---
# <a name="role-based-and-resource-based-authorization"></a>Autorisation basée sur les ressources et les rôles

[![GitHub](../_images/github.png) Exemple de code][sample application]

Notre [implémentation de référence] est une application ASP.NET Core. Dans cet article, nous allons examiner deux approches générales d’autorisation s’appuyant sur les API d’autorisation proposées dans ASP.NET Core.

* **Autorisation basée sur les rôles**. Autorisation d’une action en fonction des rôles affectés à un utilisateur. Par exemple, certaines actions nécessitent un rôle d’administrateur.
* **Autorisation basée sur les ressources**. Autorisation d’une action en fonction d’une ressource particulière. Par exemple, chaque ressource a un propriétaire. Le propriétaire peut supprimer la ressource ; les autres ne le peuvent pas.

Une application classique utilise une combinaison des deux. Par exemple, pour supprimer une ressource, l’utilisateur doit être le propriétaire de la ressource *ou* un administrateur.

## <a name="role-based-authorization"></a>Autorisation basée sur les rôles
L’application [Tailspin Surveys][Tailspin] définit les rôles suivants :

* Administrateur. Peut effectuer toutes les opérations CRUD sur toute enquête appartenant à ce locataire.
* Créateur. Peut créer de nouvelles enquêtes.
* Lecteur. Peut lire toutes les enquêtes qui appartiennent à ce locataire.

Les rôles s’appliquent aux *utilisateurs* de l’application. Dans l’application Surveys, un utilisateur est un administrateur, un créateur ou un lecteur.

Pour plus d’informations sur la façon de définir et de gérer les rôles, consultez [Rôles d’application].

Quelle que soit la façon dont vous gérez les rôles, votre code d’autorisation se présente de la même façon. ASP.NET Core possède une abstraction appelée [stratégies d’autorisation][policies]. Avec cette fonctionnalité, vous définissez des stratégies d’autorisation dans le code, puis vous appliquez ces stratégies à des actions de contrôleur. La stratégie est dissociée du contrôleur.

### <a name="create-policies"></a>Création des stratégies
Pour définir une stratégie, commencez par créer une classe qui implémente `IAuthorizationRequirement`. Il est plus facile de dériver de `AuthorizationHandler`. Dans la méthode `Handle` , examinez la ou les revendications pertinentes.

Voici un exemple de l’application Surveys de Tailspin :

```csharp
public class SurveyCreatorRequirement : AuthorizationHandler<SurveyCreatorRequirement>, IAuthorizationRequirement
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, SurveyCreatorRequirement requirement)
    {
        if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyAdmin) || 
            context.User.HasClaim(ClaimTypes.Role, Roles.SurveyCreator))
        {
            context.Succeed(requirement);
        }
        return Task.FromResult(0);
    }
}
```

Cette classe définit la configuration requise pour qu’un utilisateur crée une enquête. L’utilisateur doit avoir le rôle SurveyAdmin ou SurveyCreator.

Dans votre classe de démarrage, définissez une stratégie nommée qui inclut une ou plusieurs conditions. S’il existe plusieurs conditions, l’utilisateur doit satisfaire *chaque* condition pour être autorisé. Le code suivant définit deux stratégies :

```csharp
services.AddAuthorization(options =>
{
    options.AddPolicy(PolicyNames.RequireSurveyCreator,
        policy =>
        {
            policy.AddRequirements(new SurveyCreatorRequirement());
            policy.RequireAuthenticatedUser(); // Adds DenyAnonymousAuthorizationRequirement 
            // By adding the CookieAuthenticationDefaults.AuthenticationScheme, if an authenticated
            // user is not in the appropriate role, they will be redirected to a "forbidden" page.
            policy.AddAuthenticationSchemes(CookieAuthenticationDefaults.AuthenticationScheme);
        });

    options.AddPolicy(PolicyNames.RequireSurveyAdmin,
        policy =>
        {
            policy.AddRequirements(new SurveyAdminRequirement());
            policy.RequireAuthenticatedUser();  
            policy.AddAuthenticationSchemes(CookieAuthenticationDefaults.AuthenticationScheme);
        });
});
```

Ce code définit aussi le schéma d’authentification, qui indique à ASP.NET quel intergiciel (middleware) d’authentification exécuter en cas d’échec de l’autorisation. Dans ce cas, nous spécifions le middleware d’authentification de cookie, car il peut rediriger l’utilisateur vers une page « interdite ». L’emplacement de la page interdite est défini dans l’option `AccessDeniedPath` du middleware de cookie ; consultez [Configuration du middleware d’authentification].

### <a name="authorize-controller-actions"></a>Autorisation des actions de contrôleur
Enfin, pour autoriser une action dans un contrôleur MVC, définissez la stratégie dans l’attribut `Authorize` :

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
public IActionResult Create()
{
    var survey = new SurveyDTO();
    return View(survey);
}
```

Dans les versions antérieures d’ASP.NET, vous devez définir la propriété **Roles** sur l’attribut :

```csharp
// old way
[Authorize(Roles = "SurveyCreator")]
```

Elle est toujours prise en charge dans ASP.NET Core, mais elle présente certains inconvénients par rapport aux stratégies d’autorisation :

* Elle suppose un type de revendication particulier. Les stratégies peuvent identifier n’importe quel type de revendication. Les rôles sont simplement un type de revendication.
* Le nom de rôle est codé en dur dans l’attribut. Avec les stratégies, la logique d’autorisation se trouve à un seul et même endroit, ce qui facilite la mise à jour ou même le chargement à partir des paramètres de configuration.
* Les stratégies permettent des décisions d’autorisation plus complexes (par exemple, un âge >= 21) qui ne peuvent pas être exprimées par la simple appartenance à un rôle.

## <a name="resource-based-authorization"></a>Autorisation basée sur les ressources
L’*autorisation basée sur les ressources* intervient chaque fois que l’autorisation dépend d’une ressource spécifique qui sera affectée par une opération. Dans l’application Surveys de Tailspin, chaque enquête a un propriétaire et des collaborateurs zéro-à-plusieurs.

* Le propriétaire peut lire, mettre à jour, supprimer et publier l’enquête, ainsi qu’annuler sa publication.
* Le propriétaire peut affecter des collaborateurs à l’enquête.
* Les collaborateurs peuvent lire et mettre à jour l’enquête.

Notez que « propriétaire » et « collaborateur » ne sont pas des rôles d’application : ils sont stockés par enquête, dans la base de données de l’application. Pour vérifier si un utilisateur peut supprimer une enquête, par exemple, l’application vérifie si l’utilisateur est le propriétaire de cette enquête.

Dans ASP.NET Core, implémentez l’autorisation basée sur les ressources en dérivant d’**AuthorizationHandler** et en remplaçant la méthode **Handle**.

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override void HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement operation, Survey resource)
    {
    }
}
```

Notez que cette classe est fortement typée pour les objets de Surveys.  Inscrivez la classe pour l’injection de dépendances au démarrage :

```csharp
services.AddSingleton<IAuthorizationHandler>(factory =>
{
    return new SurveyAuthorizationHandler();
});
```

Pour effectuer des vérifications d’autorisation, utilisez l’interface **IAuthorizationService** , que vous pouvez injecter dans vos contrôleurs. Le code suivant vérifie si un utilisateur peut lire une enquête :

```csharp
if (await _authorizationService.AuthorizeAsync(User, survey, Operations.Read) == false)
{
    return StatusCode(403);
}
```

Sachant que nous passons un objet `Survey`, cet appel appelle `SurveyAuthorizationHandler`.

Dans votre code d’autorisation, une bonne approche consiste à agréger toutes les autorisations de l’utilisateur basées sur les rôles et basées sur les ressources, puis vérifiez l’agrégat défini par rapport à l’opération souhaitée.
Voici un exemple de l’application Surveys. L’application définit plusieurs types d’autorisations :

* Admin
* Contributeur
* Créateur
* Propriétaire
* Lecteur

L’application définit également un ensemble d’opérations possibles sur les enquêtes :

* Créer
* Lire
* Mettre à jour
* Supprimer
* Publish
* Annuler la publication

Le code suivant crée une liste d’autorisations pour un utilisateur et une enquête particuliers. Notez que ce code examine aussi bien les rôles d’application de l’utilisateur que les champs du propriétaire/collaborateur dans l’enquête.

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement requirement, Survey resource)
    {
        var permissions = new List<UserPermissionType>();
        int surveyTenantId = context.User.GetSurveyTenantIdValue();
        int userId = context.User.GetSurveyUserIdValue();
        string user = context.User.GetUserName();

        if (resource.TenantId == surveyTenantId)
        {
            // Admin can do anything, as long as the resource belongs to the admin's tenant.
            if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyAdmin))
            {
                context.Succeed(requirement);
                return Task.FromResult(0);
            }

            if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyCreator))
            {
                permissions.Add(UserPermissionType.Creator);
            }
            else
            {
                permissions.Add(UserPermissionType.Reader);
            }

            if (resource.OwnerId == userId)
            {
                permissions.Add(UserPermissionType.Owner);
            }
        }
        if (resource.Contributors != null && resource.Contributors.Any(x => x.UserId == userId))
        {
            permissions.Add(UserPermissionType.Contributor);
        }

        if (ValidateUserPermissions[requirement](permissions))
        {
            context.Succeed(requirement);
        }
        return Task.FromResult(0);
    }
}
```

Dans une application multilocataire, vous devez vérifier que les autorisations ne sont pas « divulguées » dans les données d’un autre locataire. Dans l’application Surveys, l’autorisation Contributor est autorisée entre les clients (vous pouvez assigner le rôle de contributeur à un utilisateur d’un autre locataire). Les autres types d’autorisation sont limités aux ressources qui appartiennent au locataire de cet utilisateur. Pour appliquer cette condition, le code vérifie l’ID du locataire avant d’accorder l’autorisation. Le code vérifie donc l’ID du locataire avant d’accorder ces types d’autorisations (le champ `TenantId` tel qu’affecté au moment de la création de l’enquête.)

L’étape suivante consiste à vérifier l’opération (lecture, mise à jour, suppression, etc.) par rapport aux autorisations. L’application Surveys implémente cette étape à l’aide d’une table de choix de fonctions :

```csharp
static readonly Dictionary<OperationAuthorizationRequirement, Func<List<UserPermissionType>, bool>> ValidateUserPermissions
    = new Dictionary<OperationAuthorizationRequirement, Func<List<UserPermissionType>, bool>>

    {
        { Operations.Create, x => x.Contains(UserPermissionType.Creator) },

        { Operations.Read, x => x.Contains(UserPermissionType.Creator) ||
                                x.Contains(UserPermissionType.Reader) ||
                                x.Contains(UserPermissionType.Contributor) ||
                                x.Contains(UserPermissionType.Owner) },

        { Operations.Update, x => x.Contains(UserPermissionType.Contributor) ||
                                x.Contains(UserPermissionType.Owner) },

        { Operations.Delete, x => x.Contains(UserPermissionType.Owner) },

        { Operations.Publish, x => x.Contains(UserPermissionType.Owner) },

        { Operations.UnPublish, x => x.Contains(UserPermissionType.Owner) }
    };
```

[**Suivant**][web-api]

<!-- Links -->
[Tailspin]: tailspin.md

[Rôles d’application]: app-roles.md
[policies]: /aspnet/core/security/authorization/policies
[implémentation de référence]: tailspin.md
[Configuration du middleware d’authentification]: authenticate.md#configure-the-auth-middleware
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[web-api]: web-api.md
