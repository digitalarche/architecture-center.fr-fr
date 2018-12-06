---
title: Utiliser une assertion du client pour obtenir des jetons d’accès d’Azure AD
description: Utilisation d’une assertion du client pour obtenir des jetons d’accès d’Azure AD.
author: MikeWasson
ms.date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: adfs
pnp.series.next: key-vault
ms.openlocfilehash: 58eed82c982fe1c6cba0f04b237d92d117a26fd4
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902261"
---
# <a name="use-client-assertion-to-get-access-tokens-from-azure-ad"></a>Utiliser une assertion du client pour obtenir des jetons d’accès d’Azure AD

[![GitHub](../_images/github.png) Exemple de code][sample application]

## <a name="background"></a>Arrière-plan
Lors de l’utilisation d’un flux de code d’autorisation ou d’un flux hybride dans OpenID Connect, le client reçoit un jeton d’accès en échange d’un code d’autorisation. Au cours de cette étape, le client doit s’authentifier auprès du serveur.

![Clé secrète client](./images/client-secret.png)

L’un des moyens d’authentifier le client consiste à utiliser une clé secrète client. Voici comment l’application [Tailspin Surveys][Surveys] est configurée par défaut.

Voici un exemple de requête du client au fournisseur d’identité, demandant un jeton d’accès. Notez le paramètre `client_secret` .

```
POST https://login.microsoftonline.com/b9bd2162xxx/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

resource=https://tailspin.onmicrosoft.com/surveys.webapi
  &client_id=87df91dc-63de-4765-8701-b59cc8bd9e11
  &client_secret=i3Bf12Dn...
  &grant_type=authorization_code
  &code=PG8wJG6Y...
```

La clé secrète n’est qu’une simple chaîne. Veillez donc à ne pas communiquer sa valeur. La meilleure pratique consiste à conserver la clé secrète client en dehors du contrôle de code source. Lorsque vous déployez un élément dans Azure, stockez la clé secrète dans un [paramètre d’application][configure-web-app].

Toutefois, toute personne ayant accès à l’abonnement Azure peut afficher les paramètres d’application. De plus, il est toujours tentant de vérifier les clés secrètes dans le contrôle de code source (par exemple, dans les scripts de déploiement), de les partager par courrier électronique, etc.

Pour renforcer la sécurité, vous pouvez utiliser l’ [assertion du client] au lieu d’une clé secrète client. Avec l’assertion du client, le client utilise un certificat X.509 pour prouver que la demande de jeton provient de lui. Le certificat client est installé sur le serveur web. En général, il est plus facile de restreindre l’accès au certificat que de garantir que personne ne révèle par inadvertance une clé secrète client. Pour en savoir plus sur la configuration de certificats dans une application web, consultez [Using Certificates in Azure Websites Applications][using-certs-in-websites].

Voici une requête de jeton utilisant l’assertion du client :

```
POST https://login.microsoftonline.com/b9bd2162xxx/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

resource=https://tailspin.onmicrosoft.com/surveys.webapi
  &client_id=87df91dc-63de-4765-8701-b59cc8bd9e11
  &client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer
  &client_assertion=eyJhbGci...
  &grant_type=authorization_code
  &code= PG8wJG6Y...
```

Notez que le paramètre `client_secret` n’est plus utilisé. À la place, le paramètre `client_assertion` contient un jeton JWT qui a été signé à l’aide du certificat client. Le paramètre `client_assertion_type` spécifie le type d’assertion &mdash; dans ce cas, jeton JWT. Le serveur valide le jeton JWT. Si le jeton JWT n’est pas valide, la requête de jeton renvoie une erreur.

> [!NOTE]
> Les certificats X.509 ne représentent pas la seule forme d’assertion du client ; nous en parlons ici car ils sont pris en charge par Azure AD.
> 
> 

Au moment de l’exécution, l’application web lit le certificat à partir du magasin de certificats. Le certificat doit être installé sur la même machine que l’application web.

L’application Surveys inclut une classe d’assistance qui crée un certificat [ClientAssertionCertificate](/dotnet/api/microsoft.identitymodel.clients.activedirectory.clientassertioncertificate) que vous pouvez transmettre à la méthode [AuthenticationContext.AcquireTokenSilentAsync](/dotnet/api/microsoft.identitymodel.clients.activedirectory.authenticationcontext.acquiretokensilentasync) pour obtenir un jeton Azure AD.

```csharp
public class CertificateCredentialService : ICredentialService
{
    private Lazy<Task<AdalCredential>> _credential;

    public CertificateCredentialService(IOptions<ConfigurationOptions> options)
    {
        var aadOptions = options.Value?.AzureAd;
        _credential = new Lazy<Task<AdalCredential>>(() =>
        {
            X509Certificate2 cert = CertificateUtility.FindCertificateByThumbprint(
                aadOptions.Asymmetric.StoreName,
                aadOptions.Asymmetric.StoreLocation,
                aadOptions.Asymmetric.CertificateThumbprint,
                aadOptions.Asymmetric.ValidationRequired);
            string password = null;
            var certBytes = CertificateUtility.ExportCertificateWithPrivateKey(cert, out password);
            return Task.FromResult(new AdalCredential(new ClientAssertionCertificate(aadOptions.ClientId, new X509Certificate2(certBytes, password))));
        });
    }

    public async Task<AdalCredential> GetCredentialsAsync()
    {
        return await _credential.Value;
    }
}
```

Pour plus d’informations sur la configuration d’une assertion du client dans l’application Surveys, consultez l’article [Utiliser Azure Key Vault pour protéger les secrets d’application][key vault].

[**Suivant**][key vault]

<!-- Links -->
[configure-web-app]: /azure/app-service-web/web-sites-configure/
[azure-management-portal]: https://portal.azure.com
[assertion du client]: https://tools.ietf.org/html/rfc7521
[key vault]: key-vault.md
[Setup-KeyVault]: https://github.com/mspnp/multitenant-saas-guidance/blob/master/scripts/Setup-KeyVault.ps1
[Surveys]: tailspin.md
[using-certs-in-websites]: https://azure.microsoft.com/blog/using-certificates-in-azure-websites-applications/

[sample application]: https://github.com/mspnp/multitenant-saas-guidance
