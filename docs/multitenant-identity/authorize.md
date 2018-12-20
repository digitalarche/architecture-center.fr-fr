---
title: Autorisation dans les applications multi-locataires
description: Comment déclarer des autorisations dans une application multi-locataire
author: MikeWasson
ms.date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: app-roles
pnp.series.next: web-api
ms.openlocfilehash: 8ff2317eb85197ed93e048b6a2d836405436cc17
ms.sourcegitcommit: 4ba3304eebaa8c493c3e5307bdd9d723cd90b655
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2018
ms.locfileid: "53307161"
---
# <a name="role-based-and-resource-based-authorization"></a><span data-ttu-id="a65b2-103">Autorisation basée sur les ressources et les rôles</span><span class="sxs-lookup"><span data-stu-id="a65b2-103">Role-based and resource-based authorization</span></span>

<span data-ttu-id="a65b2-104">[![GitHub](../_images/github.png) Exemple de code][sample application]</span><span class="sxs-lookup"><span data-stu-id="a65b2-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="a65b2-105">Notre [implémentation de référence] est une application ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="a65b2-105">Our [reference implementation] is an ASP.NET Core application.</span></span> <span data-ttu-id="a65b2-106">Dans cet article, nous allons examiner deux approches générales d’autorisation s’appuyant sur les API d’autorisation proposées dans ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="a65b2-106">In this article we'll look at two general approaches to authorization, using the authorization APIs provided in ASP.NET Core.</span></span>

* <span data-ttu-id="a65b2-107">**Autorisation basée sur les rôles**.</span><span class="sxs-lookup"><span data-stu-id="a65b2-107">**Role-based authorization**.</span></span> <span data-ttu-id="a65b2-108">Autorisation d’une action en fonction des rôles affectés à un utilisateur.</span><span class="sxs-lookup"><span data-stu-id="a65b2-108">Authorizing an action based on the roles assigned to a user.</span></span> <span data-ttu-id="a65b2-109">Par exemple, certaines actions nécessitent un rôle d’administrateur.</span><span class="sxs-lookup"><span data-stu-id="a65b2-109">For example, some actions require an administrator role.</span></span>
* <span data-ttu-id="a65b2-110">**Autorisation basée sur les ressources**.</span><span class="sxs-lookup"><span data-stu-id="a65b2-110">**Resource-based authorization**.</span></span> <span data-ttu-id="a65b2-111">Autorisation d’une action en fonction d’une ressource particulière.</span><span class="sxs-lookup"><span data-stu-id="a65b2-111">Authorizing an action based on a particular resource.</span></span> <span data-ttu-id="a65b2-112">Par exemple, chaque ressource a un propriétaire.</span><span class="sxs-lookup"><span data-stu-id="a65b2-112">For example, every resource has an owner.</span></span> <span data-ttu-id="a65b2-113">Le propriétaire peut supprimer la ressource ; les autres ne le peuvent pas.</span><span class="sxs-lookup"><span data-stu-id="a65b2-113">The owner can delete the resource; other users cannot.</span></span>

<span data-ttu-id="a65b2-114">Une application classique utilise une combinaison des deux.</span><span class="sxs-lookup"><span data-stu-id="a65b2-114">A typical app will employ a mix of both.</span></span> <span data-ttu-id="a65b2-115">Par exemple, pour supprimer une ressource, l’utilisateur doit être le propriétaire de la ressource *ou* un administrateur.</span><span class="sxs-lookup"><span data-stu-id="a65b2-115">For example, to delete a resource, the user must be the resource owner *or* an admin.</span></span>

## <a name="role-based-authorization"></a><span data-ttu-id="a65b2-116">Autorisation basée sur les rôles</span><span class="sxs-lookup"><span data-stu-id="a65b2-116">Role-Based Authorization</span></span>
<span data-ttu-id="a65b2-117">L’application [Tailspin Surveys][Tailspin] définit les rôles suivants :</span><span class="sxs-lookup"><span data-stu-id="a65b2-117">The [Tailspin Surveys][Tailspin] application defines the following roles:</span></span>

* <span data-ttu-id="a65b2-118">Administrateur.</span><span class="sxs-lookup"><span data-stu-id="a65b2-118">Administrator.</span></span> <span data-ttu-id="a65b2-119">Peut effectuer toutes les opérations CRUD sur toute enquête appartenant à ce locataire.</span><span class="sxs-lookup"><span data-stu-id="a65b2-119">Can perform all CRUD operations on any survey that belongs to that tenant.</span></span>
* <span data-ttu-id="a65b2-120">Créateur.</span><span class="sxs-lookup"><span data-stu-id="a65b2-120">Creator.</span></span> <span data-ttu-id="a65b2-121">Peut créer de nouvelles enquêtes.</span><span class="sxs-lookup"><span data-stu-id="a65b2-121">Can create new surveys</span></span>
* <span data-ttu-id="a65b2-122">Lecteur.</span><span class="sxs-lookup"><span data-stu-id="a65b2-122">Reader.</span></span> <span data-ttu-id="a65b2-123">Peut lire toutes les enquêtes qui appartiennent à ce locataire.</span><span class="sxs-lookup"><span data-stu-id="a65b2-123">Can read any surveys that belong to that tenant</span></span>

<span data-ttu-id="a65b2-124">Les rôles s’appliquent aux *utilisateurs* de l’application.</span><span class="sxs-lookup"><span data-stu-id="a65b2-124">Roles apply to *users* of the application.</span></span> <span data-ttu-id="a65b2-125">Dans l’application Surveys, un utilisateur est un administrateur, un créateur ou un lecteur.</span><span class="sxs-lookup"><span data-stu-id="a65b2-125">In the Surveys application, a user is either an administrator, creator, or reader.</span></span>

<span data-ttu-id="a65b2-126">Pour plus d’informations sur la façon de définir et de gérer les rôles, consultez [Rôles d’application].</span><span class="sxs-lookup"><span data-stu-id="a65b2-126">For a discussion of how to define and manage roles, see [Application roles].</span></span>

<span data-ttu-id="a65b2-127">Quelle que soit la façon dont vous gérez les rôles, votre code d’autorisation se présente de la même façon.</span><span class="sxs-lookup"><span data-stu-id="a65b2-127">Regardless of how you manage the roles, your authorization code will look similar.</span></span> <span data-ttu-id="a65b2-128">ASP.NET Core possède une abstraction appelée [stratégies d’autorisation][policies].</span><span class="sxs-lookup"><span data-stu-id="a65b2-128">ASP.NET Core has an abstraction called [authorization policies][policies].</span></span> <span data-ttu-id="a65b2-129">Avec cette fonctionnalité, vous définissez des stratégies d’autorisation dans le code, puis vous appliquez ces stratégies à des actions de contrôleur.</span><span class="sxs-lookup"><span data-stu-id="a65b2-129">With this feature, you define authorization policies in code, and then apply those policies to controller actions.</span></span> <span data-ttu-id="a65b2-130">La stratégie est dissociée du contrôleur.</span><span class="sxs-lookup"><span data-stu-id="a65b2-130">The policy is decoupled from the controller.</span></span>

### <a name="create-policies"></a><span data-ttu-id="a65b2-131">Création des stratégies</span><span class="sxs-lookup"><span data-stu-id="a65b2-131">Create policies</span></span>
<span data-ttu-id="a65b2-132">Pour définir une stratégie, commencez par créer une classe qui implémente `IAuthorizationRequirement`.</span><span class="sxs-lookup"><span data-stu-id="a65b2-132">To define a policy, first create a class that implements `IAuthorizationRequirement`.</span></span> <span data-ttu-id="a65b2-133">Il est plus facile de dériver de `AuthorizationHandler`.</span><span class="sxs-lookup"><span data-stu-id="a65b2-133">It's easiest to derive from `AuthorizationHandler`.</span></span> <span data-ttu-id="a65b2-134">Dans la méthode `Handle` , examinez la ou les revendications pertinentes.</span><span class="sxs-lookup"><span data-stu-id="a65b2-134">In the `Handle` method, examine the relevant claim(s).</span></span>

<span data-ttu-id="a65b2-135">Voici un exemple de l’application Surveys de Tailspin :</span><span class="sxs-lookup"><span data-stu-id="a65b2-135">Here is an example from the Tailspin Surveys application:</span></span>

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

<span data-ttu-id="a65b2-136">Cette classe définit la configuration requise pour qu’un utilisateur crée une enquête.</span><span class="sxs-lookup"><span data-stu-id="a65b2-136">This class defines the requirement for a user to create a new survey.</span></span> <span data-ttu-id="a65b2-137">L’utilisateur doit avoir le rôle SurveyAdmin ou SurveyCreator.</span><span class="sxs-lookup"><span data-stu-id="a65b2-137">The user must be in the SurveyAdmin or SurveyCreator role.</span></span>

<span data-ttu-id="a65b2-138">Dans votre classe de démarrage, définissez une stratégie nommée qui inclut une ou plusieurs conditions.</span><span class="sxs-lookup"><span data-stu-id="a65b2-138">In your startup class, define a named policy that includes one or more requirements.</span></span> <span data-ttu-id="a65b2-139">S’il existe plusieurs conditions, l’utilisateur doit satisfaire *chaque* condition pour être autorisé.</span><span class="sxs-lookup"><span data-stu-id="a65b2-139">If there are multiple requirements, the user must meet *every* requirement to be authorized.</span></span> <span data-ttu-id="a65b2-140">Le code suivant définit deux stratégies :</span><span class="sxs-lookup"><span data-stu-id="a65b2-140">The following code defines two policies:</span></span>

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

<span data-ttu-id="a65b2-141">Ce code définit aussi le schéma d’authentification, qui indique à ASP.NET quel intergiciel (middleware) d’authentification exécuter en cas d’échec de l’autorisation.</span><span class="sxs-lookup"><span data-stu-id="a65b2-141">This code also sets the authentication scheme, which tells ASP.NET which authentication middleware should run if authorization fails.</span></span> <span data-ttu-id="a65b2-142">Dans ce cas, nous spécifions le middleware d’authentification de cookie, car il peut rediriger l’utilisateur vers une page « interdite ».</span><span class="sxs-lookup"><span data-stu-id="a65b2-142">In this case, we specify the cookie authentication middleware, because the cookie authentication middleware can redirect the user to a "Forbidden" page.</span></span> <span data-ttu-id="a65b2-143">L’emplacement de la page interdite est défini dans l’option `AccessDeniedPath` du middleware de cookie ; consultez [Configuration du middleware d’authentification].</span><span class="sxs-lookup"><span data-stu-id="a65b2-143">The location of the Forbidden page is set in the `AccessDeniedPath` option for the cookie middleware; see [Configuring the authentication middleware].</span></span>

### <a name="authorize-controller-actions"></a><span data-ttu-id="a65b2-144">Autorisation des actions de contrôleur</span><span class="sxs-lookup"><span data-stu-id="a65b2-144">Authorize controller actions</span></span>
<span data-ttu-id="a65b2-145">Enfin, pour autoriser une action dans un contrôleur MVC, définissez la stratégie dans l’attribut `Authorize` :</span><span class="sxs-lookup"><span data-stu-id="a65b2-145">Finally, to authorize an action in an MVC controller, set the policy in the `Authorize` attribute:</span></span>

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
public IActionResult Create()
{
    var survey = new SurveyDTO();
    return View(survey);
}
```

<span data-ttu-id="a65b2-146">Dans les versions antérieures d’ASP.NET, vous devez définir la propriété **Roles** sur l’attribut :</span><span class="sxs-lookup"><span data-stu-id="a65b2-146">In earlier versions of ASP.NET, you would set the **Roles** property on the attribute:</span></span>

```csharp
// old way
[Authorize(Roles = "SurveyCreator")]
```

<span data-ttu-id="a65b2-147">Elle est toujours prise en charge dans ASP.NET Core, mais elle présente certains inconvénients par rapport aux stratégies d’autorisation :</span><span class="sxs-lookup"><span data-stu-id="a65b2-147">This is still supported in ASP.NET Core, but it has some drawbacks compared with authorization policies:</span></span>

* <span data-ttu-id="a65b2-148">Elle suppose un type de revendication particulier.</span><span class="sxs-lookup"><span data-stu-id="a65b2-148">It assumes a particular claim type.</span></span> <span data-ttu-id="a65b2-149">Les stratégies peuvent identifier n’importe quel type de revendication.</span><span class="sxs-lookup"><span data-stu-id="a65b2-149">Policies can check for any claim type.</span></span> <span data-ttu-id="a65b2-150">Les rôles sont simplement un type de revendication.</span><span class="sxs-lookup"><span data-stu-id="a65b2-150">Roles are just a type of claim.</span></span>
* <span data-ttu-id="a65b2-151">Le nom de rôle est codé en dur dans l’attribut.</span><span class="sxs-lookup"><span data-stu-id="a65b2-151">The role name is hard-coded into the attribute.</span></span> <span data-ttu-id="a65b2-152">Avec les stratégies, la logique d’autorisation se trouve à un seul et même endroit, ce qui facilite la mise à jour ou même le chargement à partir des paramètres de configuration.</span><span class="sxs-lookup"><span data-stu-id="a65b2-152">With policies, the authorization logic is all in one place, making it easier to update or even load from configuration settings.</span></span>
* <span data-ttu-id="a65b2-153">Les stratégies permettent des décisions d’autorisation plus complexes (par exemple, un âge >= 21) qui ne peuvent pas être exprimées par la simple appartenance à un rôle.</span><span class="sxs-lookup"><span data-stu-id="a65b2-153">Policies enable more complex authorization decisions (e.g., age >= 21) that can't be expressed by simple role membership.</span></span>

## <a name="resource-based-authorization"></a><span data-ttu-id="a65b2-154">Autorisation basée sur les ressources</span><span class="sxs-lookup"><span data-stu-id="a65b2-154">Resource based authorization</span></span>
<span data-ttu-id="a65b2-155">*Autorisation basée sur les ressources* se produit chaque fois que l’autorisation dépend d’une ressource spécifique qui sera affectée par une opération.</span><span class="sxs-lookup"><span data-stu-id="a65b2-155">*Resource based authorization* occurs whenever the authorization depends on a specific resource that will be affected by an operation.</span></span> <span data-ttu-id="a65b2-156">Dans l’application Surveys de Tailspin, chaque enquête a un propriétaire et des collaborateurs zéro-à-plusieurs.</span><span class="sxs-lookup"><span data-stu-id="a65b2-156">In the Tailspin Surveys application, every survey has an owner and zero-to-many contributors.</span></span>

* <span data-ttu-id="a65b2-157">Le propriétaire peut lire, mettre à jour, supprimer et publier l’enquête, ainsi qu’annuler sa publication.</span><span class="sxs-lookup"><span data-stu-id="a65b2-157">The owner can read, update, delete, publish, and unpublish the survey.</span></span>
* <span data-ttu-id="a65b2-158">Le propriétaire peut affecter des collaborateurs à l’enquête.</span><span class="sxs-lookup"><span data-stu-id="a65b2-158">The owner can assign contributors to the survey.</span></span>
* <span data-ttu-id="a65b2-159">Les collaborateurs peuvent lire et mettre à jour l’enquête.</span><span class="sxs-lookup"><span data-stu-id="a65b2-159">Contributors can read and update the survey.</span></span>

<span data-ttu-id="a65b2-160">Notez que « propriétaire » et « collaborateur » ne sont pas des rôles d’application : ils sont stockés par enquête, dans la base de données de l’application.</span><span class="sxs-lookup"><span data-stu-id="a65b2-160">Note that "owner" and "contributor" are not application roles; they are stored per survey, in the application database.</span></span> <span data-ttu-id="a65b2-161">Pour vérifier si un utilisateur peut supprimer une enquête, par exemple, l’application vérifie si l’utilisateur est le propriétaire de cette enquête.</span><span class="sxs-lookup"><span data-stu-id="a65b2-161">To check whether a user can delete a survey, for example, the app checks whether the user is the owner for that survey.</span></span>

<span data-ttu-id="a65b2-162">Dans ASP.NET Core, implémentez l’autorisation basée sur les ressources en dérivant d’**AuthorizationHandler** et en remplaçant la méthode **Handle**.</span><span class="sxs-lookup"><span data-stu-id="a65b2-162">In ASP.NET Core, implement resource-based authorization by deriving from **AuthorizationHandler** and overriding the **Handle** method.</span></span>

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override void HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement operation, Survey resource)
    {
    }
}
```

<span data-ttu-id="a65b2-163">Notez que cette classe est fortement typée pour les objets de Surveys.</span><span class="sxs-lookup"><span data-stu-id="a65b2-163">Notice that this class is strongly typed for Survey objects.</span></span>  <span data-ttu-id="a65b2-164">Inscrivez la classe pour l’injection de dépendances au démarrage :</span><span class="sxs-lookup"><span data-stu-id="a65b2-164">Register the class for DI on startup:</span></span>

```csharp
services.AddSingleton<IAuthorizationHandler>(factory =>
{
    return new SurveyAuthorizationHandler();
});
```

<span data-ttu-id="a65b2-165">Pour effectuer des vérifications d’autorisation, utilisez l’interface **IAuthorizationService** , que vous pouvez injecter dans vos contrôleurs.</span><span class="sxs-lookup"><span data-stu-id="a65b2-165">To perform authorization checks, use the **IAuthorizationService** interface, which you can inject into your controllers.</span></span> <span data-ttu-id="a65b2-166">Le code suivant vérifie si un utilisateur peut lire une enquête :</span><span class="sxs-lookup"><span data-stu-id="a65b2-166">The following code checks whether a user can read a survey:</span></span>

```csharp
if (await _authorizationService.AuthorizeAsync(User, survey, Operations.Read) == false)
{
    return StatusCode(403);
}
```

<span data-ttu-id="a65b2-167">Sachant que nous passons un objet `Survey`, cet appel appelle `SurveyAuthorizationHandler`.</span><span class="sxs-lookup"><span data-stu-id="a65b2-167">Because we pass in a `Survey` object, this call will invoke the `SurveyAuthorizationHandler`.</span></span>

<span data-ttu-id="a65b2-168">Dans votre code d’autorisation, une bonne approche consiste à agréger toutes les autorisations de l’utilisateur basées sur les rôles et basées sur les ressources, puis vérifiez l’agrégat défini par rapport à l’opération souhaitée.</span><span class="sxs-lookup"><span data-stu-id="a65b2-168">In your authorization code, a good approach is to aggregate all of the user's role-based and resource-based permissions, then check the aggregate set against the desired operation.</span></span>
<span data-ttu-id="a65b2-169">Voici un exemple de l’application Surveys.</span><span class="sxs-lookup"><span data-stu-id="a65b2-169">Here is an example from the Surveys app.</span></span> <span data-ttu-id="a65b2-170">L’application définit plusieurs types d’autorisations :</span><span class="sxs-lookup"><span data-stu-id="a65b2-170">The application defines several permission types:</span></span>

* <span data-ttu-id="a65b2-171">Admin</span><span class="sxs-lookup"><span data-stu-id="a65b2-171">Admin</span></span>
* <span data-ttu-id="a65b2-172">Contributeur</span><span class="sxs-lookup"><span data-stu-id="a65b2-172">Contributor</span></span>
* <span data-ttu-id="a65b2-173">Créateur</span><span class="sxs-lookup"><span data-stu-id="a65b2-173">Creator</span></span>
* <span data-ttu-id="a65b2-174">Propriétaire</span><span class="sxs-lookup"><span data-stu-id="a65b2-174">Owner</span></span>
* <span data-ttu-id="a65b2-175">Lecteur</span><span class="sxs-lookup"><span data-stu-id="a65b2-175">Reader</span></span>

<span data-ttu-id="a65b2-176">L’application définit également un ensemble d’opérations possibles sur les enquêtes :</span><span class="sxs-lookup"><span data-stu-id="a65b2-176">The application also defines a set of possible operations on surveys:</span></span>

* <span data-ttu-id="a65b2-177">Créer</span><span class="sxs-lookup"><span data-stu-id="a65b2-177">Create</span></span>
* <span data-ttu-id="a65b2-178">Lire</span><span class="sxs-lookup"><span data-stu-id="a65b2-178">Read</span></span>
* <span data-ttu-id="a65b2-179">Mettre à jour</span><span class="sxs-lookup"><span data-stu-id="a65b2-179">Update</span></span>
* <span data-ttu-id="a65b2-180">Supprimer</span><span class="sxs-lookup"><span data-stu-id="a65b2-180">Delete</span></span>
* <span data-ttu-id="a65b2-181">Publish</span><span class="sxs-lookup"><span data-stu-id="a65b2-181">Publish</span></span>
* <span data-ttu-id="a65b2-182">Dépublier</span><span class="sxs-lookup"><span data-stu-id="a65b2-182">Unpublish</span></span>

<span data-ttu-id="a65b2-183">Le code suivant crée une liste d’autorisations pour un utilisateur et une enquête particuliers.</span><span class="sxs-lookup"><span data-stu-id="a65b2-183">The following code creates a list of permissions for a particular user and survey.</span></span> <span data-ttu-id="a65b2-184">Notez que ce code examine aussi bien les rôles d’application de l’utilisateur que les champs du propriétaire/collaborateur dans l’enquête.</span><span class="sxs-lookup"><span data-stu-id="a65b2-184">Notice that this code looks at both the user's app roles, and the owner/contributor fields in the survey.</span></span>

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

<span data-ttu-id="a65b2-185">Dans une application multilocataire, vous devez vérifier que les autorisations ne sont pas « divulguées » dans les données d’un autre locataire.</span><span class="sxs-lookup"><span data-stu-id="a65b2-185">In a multi-tenant application, you must ensure that permissions don't "leak" to another tenant's data.</span></span> <span data-ttu-id="a65b2-186">Dans l’application Surveys, l’autorisation Contributor est autorisée entre les locataires &mdash; vous pouvez assigner le rôle de contributeur à un utilisateur d’un autre locataire.</span><span class="sxs-lookup"><span data-stu-id="a65b2-186">In the Surveys app, the Contributor permission is allowed across tenants &mdash; you can assign someone from another tenant as a contributor.</span></span> <span data-ttu-id="a65b2-187">Les autres types d’autorisation sont limités aux ressources qui appartiennent au locataire de cet utilisateur.</span><span class="sxs-lookup"><span data-stu-id="a65b2-187">The other permission types are restricted to resources that belong to that user's tenant.</span></span> <span data-ttu-id="a65b2-188">Pour appliquer cette condition, le code vérifie l’ID du locataire avant d’accorder l’autorisation.</span><span class="sxs-lookup"><span data-stu-id="a65b2-188">To enforce this requirement, the code checks the tenant ID before granting the permission.</span></span> <span data-ttu-id="a65b2-189">Le code vérifie donc l’ID du locataire avant d’accorder ces types d’autorisations (le champ `TenantId` tel qu’affecté au moment de la création de l’enquête.)</span><span class="sxs-lookup"><span data-stu-id="a65b2-189">(The `TenantId` field as assigned when the survey is created.)</span></span>

<span data-ttu-id="a65b2-190">L’étape suivante consiste à vérifier l’opération (lecture, mise à jour, suppression, etc.) par rapport aux autorisations.</span><span class="sxs-lookup"><span data-stu-id="a65b2-190">The next step is to check the operation (read, update, delete, etc) against the permissions.</span></span> <span data-ttu-id="a65b2-191">L’application Surveys implémente cette étape à l’aide d’une table de choix de fonctions :</span><span class="sxs-lookup"><span data-stu-id="a65b2-191">The Surveys app implements this step by using a lookup table of functions:</span></span>

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

<span data-ttu-id="a65b2-192">[**Suivant**][web-api]</span><span class="sxs-lookup"><span data-stu-id="a65b2-192">[**Next**][web-api]</span></span>

<!-- Links -->
[Tailspin]: tailspin.md

[Rôles d’application]: app-roles.md
[Application roles]: app-roles.md
[policies]: /aspnet/core/security/authorization/policies
[implémentation de référence]: tailspin.md
[reference implementation]: tailspin.md
[Configuration du middleware d’authentification]: authenticate.md#configure-the-auth-middleware
[Configuring the authentication middleware]: authenticate.md#configure-the-auth-middleware
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[web-api]: web-api.md
