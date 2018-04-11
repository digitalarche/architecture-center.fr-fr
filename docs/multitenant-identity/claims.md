---
title: Utilisation d’identités basées sur les revendications dans les applications multi-locataires
description: Comment utiliser des revendications pour la validation de l’émetteur et l’autorisation
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: authenticate
pnp.series.next: signup
ms.openlocfilehash: 61788d9759715b21ef1bdda59c5b54d923fd8f62
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2017
---
# <a name="work-with-claims-based-identities"></a>Utilisation d’identités basées sur les revendications

[![GitHub](../_images/github.png) Exemple de code][sample application]

## <a name="claims-in-azure-ad"></a>Revendications dans Azure AD
Quand un utilisateur se connecte, Azure AD envoie un jeton d’ID qui contient un ensemble de revendications concernant l’utilisateur. Une revendication est simplement une information, exprimée sous la forme d’une paire clé/valeur. Par exemple : `email`=`bob@contoso.com`.  Les revendications ont un émetteur &mdash; dans ce cas, Azure AD &mdash; qui est l’entité qui authentifie l’utilisateur et crée les revendications. Vous approuvez les revendications, car vous approuvez l’émetteur. (À l’inverse, si vous n’approuvez l’émetteur, n’approuvez pas les revendications !)

De façon générale :

1. L’utilisateur s’authentifie.
2. Le fournisseur d’identité envoie un ensemble de revendications.
3. L’application normalise ou renforce les revendications (facultatif).
4. L’application utilise les revendications pour prendre des décisions d’autorisation.

Dans OpenID Connect, l’ensemble de revendications que vous obtenez est contrôlé par le [paramètre d’étendue] de la demande d’authentification. Toutefois, Azure AD émet un ensemble limité de revendications via OpenID Connect ; voir [Types de jeton et de revendication pris en charge]. Pour obtenir davantage d’informations sur l’utilisateur, utilisez l’API Graph Azure AD.

Voici quelques-unes des revendications d’AAD auxquelles une application peut généralement s’intéresser :

| Type de revendication dans le jeton d’ID | Description |
| --- | --- |
| aud |Pour qui le jeton a été émis. Il s’agit de l’ID client de l’application. En général, vous n’avez pas à vous soucier de cette revendication, car le middleware la valide automatiquement. Exemple : `"91464657-d17a-4327-91f3-2ed99386406f"` |
| groups |Liste de groupes AAD dont l’utilisateur est membre. Exemple : `["93e8f556-8661-4955-87b6-890bc043c30f", "fc781505-18ef-4a31-a7d5-7d931d7b857e"]` |
| iss |[Émetteur] du jeton OIDC. Exemple : `https://sts.windows.net/b9bd2162-77ac-4fb2-8254-5c36e9c0a9c4/` |
| name |Nom d’affichage de l’utilisateur. Exemple : `"Alice A."` |
| oid |Identificateur d’objet pour l’utilisateur dans AAD. Cette valeur est l’identificateur non modifiable et non réutilisable de l’utilisateur. Utilisez cette valeur, et non pas l’adresse de messagerie, comme identificateur unique pour les utilisateurs ; en effet, les adresses de messagerie peuvent changer. Si vous utilisez l’API Azure AD Graph dans votre application, l’ID objet est cette valeur utilisée pour demander des informations de profil. Exemple : `"59f9d2dc-995a-4ddf-915e-b3bb314a7fa4"` |
| roles |Liste des rôles d’application pour l’utilisateur.    Exemple : `["SurveyCreator"]` |
| tid |ID de locataire. Cette valeur est un identificateur unique pour le client dans Azure AD. Exemple : `"b9bd2162-77ac-4fb2-8254-5c36e9c0a9c4"` |
| unique_name |Nom d’affichage explicite de l’utilisateur. Exemple : `"alice@contoso.com"` |
| upn |Nom d’utilisateur principal. Exemple : `"alice@contoso.com"` |

Ce tableau répertorie les types de revendications tels qu’ils apparaissent dans le jeton d’ID. Dans ASP.NET Core, le middleware OpenID Connect convertit certains des types de revendications quand il remplit la collection de revendications pour le nom principal de l’utilisateur :

* oid > `http://schemas.microsoft.com/identity/claims/objectidentifier`
* tid > `http://schemas.microsoft.com/identity/claims/tenantid`
* unique_name > `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`
* upn > `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn`

## <a name="claims-transformations"></a>Transformations de revendication
Pendant le flux d’authentification, vous pouvez modifier les revendications que vous obtenez du fournisseur d’identité. Dans ASP.NET Core, vous pouvez effectuer une transformation des revendications à l’intérieur de l’événement **AuthenticationValidated** à partir du middleware OpenID Connect. (Consultez la page [Authentication events].)

Toutes les revendications que vous ajoutez pendant **AuthenticationValidated** sont stockées dans le cookie d’authentification de session. Elles ne font pas l’objet d’une transmission de type push à Azure AD.

Voici quelques exemples de transformation de revendications :

* **Normalisation des revendications**, ou rendre les réclamations cohérentes entre les utilisateurs. Cela est particulièrement approprié si vous obtenez des revendications de plusieurs fournisseurs d’identité, qui peuvent utiliser des types de revendications différents pour des informations similaires.
  Par exemple, Azure AD envoie une revendication « upn » qui contient l’adresse de messagerie de l’utilisateur. D’autres fournisseurs d’identité peuvent envoyer une revendication « email ». Le code suivant convertit la revendication « upn » en revendication « email » :
  
  ```csharp
  var email = principal.FindFirst(ClaimTypes.Upn)?.Value;
  if (!string.IsNullOrWhiteSpace(email))
  {
      identity.AddClaim(new Claim(ClaimTypes.Email, email));
  }
  ```
* Ajoutez **les valeurs de revendication par défaut** pour les revendications qui ne sont pas présentes &mdash; par exemple, affectation d’un utilisateur à un rôle par défaut. Dans certains cas, cela peut simplifier la logique d’autorisation.
* Ajoutez **les types de revendication personnalisée** avec les informations spécifiques à l’application sur l’utilisateur. Par exemple, vous pouvez stocker des informations sur l’utilisateur dans une base de données. Vous pouvez ajouter une revendication personnalisée avec ces informations sur le ticket d’authentification. La revendication est stockée dans un cookie. Ainsi, vous n’aurez besoin de la récupérer dans la base de données qu’une seule fois par session de connexion. En revanche, vous souhaitez également éviter de créer des cookies trop volumineux. Par conséquent, réfléchissez à un compromis entre la taille des cookies et les recherches dans la base de données.   

Une fois le flux d’authentification terminé, les revendications sont disponibles dans `HttpContext.User`. À ce stade, vous devez les traiter comme une collection en lecture seule &mdash; vous pouvez par exemple les utiliser pour prendre des décisions d’autorisation.

## <a name="issuer-validation"></a>Validation de l’émetteur
Dans OpenID Connect, la revendication de l’émetteur (« iss ») identifie le fournisseur d’identité qui a émis le jeton d’ID. Une partie du flux d’authentification OIDC consiste à vérifier que la revendication de l’émetteur correspond à l’émetteur réel. Le middleware OIDC gère cela automatiquement.

Dans Azure AD, la valeur d’émetteur est unique par locataire AD (`https://sts.windows.net/<tenantID>`). Par conséquent, une application doit effectuer une vérification supplémentaire pour garantir que l’émetteur représente un locataire qui est autorisé à se connecter à l’application.

Pour une application à locataire unique, vous pouvez simplement vérifier que l’émetteur est votre propre locataire. En fait, le middleware OIDC effectue cette opération automatiquement par défaut. Dans une application multi-locataire, vous devez autoriser plusieurs émetteurs, correspondant aux différents locataires. Voici une approche générale à utiliser :

* Dans les options de middleware OIDC, affectez à **ValidateIssuer** sur false. Cela désactive la vérification automatique.
* Quand un locataire s’inscrit, stockez le locataire et l’émetteur dans votre base de données utilisateur.
* Chaque fois qu’un utilisateur se connecte, recherchez l’émetteur dans la base de données. Si l’émetteur est introuvable, cela signifie que le locataire ne s’est pas inscrit. Vous pouvez le rediriger vers une page d’inscription.
* Vous pouvez également bloquer certains locataires ; par exemple, pour les clients qui n’ont pas payé leur abonnement.

Pour obtenir plus de détails, consultez [Inscription et intégration de locataire dans une application multi-locataire][signup].

## <a name="using-claims-for-authorization"></a>Utilisation de revendications pour l’autorisation
Avec les revendications, l’identité d’un utilisateur n’est plus une entité monolithique. Par exemple, un utilisateur peut avoir une adresse e-mail, un numéro de téléphone, une date de naissance, un genre, etc. Il est possible que le point de distribution d’émission (IDP) de l’utilisateur stocke toutes ces informations. Mais lorsque vous authentifiez l’utilisateur, vous obtiendrez généralement un sous-ensemble de ces informations sous la forme de revendications. Dans ce modèle, l’identité de l’utilisateur est simplement un ensemble de revendications. Au moment de prendre des décisions d’autorisation à propos d’un utilisateur, vous rechercherez des ensembles de revendications spécifiques. En d’autres termes, la question « L’utilisateur X peut-il effectuer l’action Y ? » devient finalement « L’utilisateur X dispose-t-il de la revendication Z ? ».

Voici quelques modèles de base pour la vérification des revendications.

* Pour vérifier que l’utilisateur a une revendication particulière avec une valeur particulière :
  
   ```csharp
   if (User.HasClaim(ClaimTypes.Role, "Admin")) { ... }
   ```
   Ce code vérifie si l’utilisateur a une revendication de rôle avec la valeur « Admin ». Il gère correctement le cas où l’utilisateur n’a aucune revendication de rôle ou a plusieurs revendications de rôle.
  
   La classe **ClaimTypes** définit des constantes pour les types de revendications couramment utilisés. Toutefois, vous pouvez utiliser n’importe quelle valeur de chaîne pour le type de revendication.
* Pour obtenir une valeur unique pour un type de revendication, quand vous prévoyez qu’il y aura au plus une valeur :
  
  ```csharp
  string email = User.FindFirst(ClaimTypes.Email)?.Value;
  ```
* Pour obtenir toutes les valeurs pour un type de revendication :
  
  ```csharp
  IEnumerable<Claim> groups = User.FindAll("groups");
  ```

Pour en savoir plus, consultez [Autorisation basée sur les ressources et les rôles dans les applications multi-locataires][authorization].

[**Suivant**][signup]


<!-- Links -->

[paramètre d’étendue]: http://nat.sakimura.org/2012/01/26/scopes-and-claims-in-openid-connect/
[Types de jeton et de revendication pris en charge]: /azure/active-directory/active-directory-token-and-claims/
[Émetteur]: http://openid.net/specs/openid-connect-core-1_0.html#IDToken
[Authentication events]: authenticate.md#authentication-events
[signup]: signup.md
[Claims-Based Authorization]: /aspnet/core/security/authorization/claims
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[authorization]: authorize.md
