---
title: 'Framework d’adoption du cloud : outils Base de référence des identités dans Azure'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Base de référence des identités dans Azure
author: BrianBlanchard
ms.openlocfilehash: 81b0fa9cfee597da98d8b983fb155eac82d97bf8
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58243670"
---
# <a name="identity-baseline-tools-in-azure"></a>Base de référence des identités dans Azure

[Base de référence des identités](overview.md) est l’une des [cinq disciplines de la gouvernance cloud](../governance-disciplines.md). Cette discipline se concentre sur les manières d’établir des stratégies qui garantissent la cohérence et la continuité des identités utilisateurs, quel que soit le fournisseur de cloud hébergeant l’application ou la charge de travail.

Les outils suivants sont inclus dans le guide de découverte sur Identité hybride.

**Active Directory (local) :** Active Directory est le fournisseur d’identité le plus fréquemment utilisé dans l’entreprise pour stocker et valider les informations d’identification des utilisateurs.

**Azure Active Directory :** Logiciel en tant que service (SaaS) équivalent à Active Directory, capable de se fédérer avec un annuaire Active Directory local.

**Active Directory (IaaS) :** Instance de l’application Active Directory s’exécutant dans une machine virtuelle dans Azure.

L’identité constitue le plan de contrôle de la sécurité informatique. L’authentification protège par conséquent une organisation qui accède au cloud. Les organisations ont besoin d’un plan de contrôle d’identité qui renforce leur sécurité et protège leurs applications cloud contre les intrus.

## <a name="cloud-authentication"></a>Authentification cloud

La première étape pour les organisations souhaitant migrer leurs applications vers le cloud est de choisir une méthode d’authentification appropriée.

Quand vous choisissez cette méthode, Azure AD gère le processus de connexion des utilisateurs. Si vous l’associez à une authentification unique (SSO) fluide, les utilisateurs peuvent se connectent aux applications cloud sans avoir à retaper leurs informations d’identification. L’authentification cloud propose deux options :

**Synchronisation de hachage de mot de passe Azure AD** : Il s’agit du moyen le plus simple d’activer l’authentification pour les objets d’annuaire locaux dans Azure AD. Cette méthode peut également être utilisée avec n’importe quelle méthode comme méthode d’authentification de basculement en cas de panne de votre serveur local.

**Authentification directe Azure AD** : Fournit une validation de mot de passe persistante pour les services d’authentification Azure AD à l’aide d’un agent logiciel qui s’exécute sur un ou plusieurs serveurs locaux.

> [!NOTE]
> La méthode d’authentification directe convient pour les entreprises qui, pour des raisons de sécurité, requièrent l’application immédiate d’heures d’ouverture de session, de stratégies de mot de passe et d’états des comptes d’utilisateur locaux.

**Authentification fédérée :**

Quand vous choisissez cette méthode, Azure AD transfère le processus d’authentification à un système d’authentification approuvée distinct, par exemple les services de fédération Active Directory (AD FS) ou un service de fédération tiers sécurisé, pour valider le mot de passe de l’utilisateur.

L’article [Choisir la méthode d’authentification adaptée à Azure Active Directory](/azure/security/azure-ad-choose-authn) propose un arbre de décision pour vous aider à choisir la meilleure solution pour votre organisation.

Le tableau suivant énumère les outils natifs qui peuvent aider à faire mûrir les stratégies et les processus qui supportent cette discipline de gouvernance.

<!-- markdownlint-disable MD033 -->

|Considération|Synchronisation de hachage du mot de passe + authentification unique transparente|Authentification directe + authentification unique transparente|Fédération avec AD FS|
|:-----|:-----|:-----|:-----|
|Où l’authentification se produit-elle ?|Dans le cloud|Dans le cloud, après un échange de vérification de mot de passe sécurisé avec l’agent d’authentification local|Local|
|Quelles sont les exigences pour le serveur local au-delà du système de provisionnement : Azure AD Connect ?|Aucun|Un serveur par agent d’authentification supplémentaire|Deux ou plusieurs serveurs AD FS<br><br>Deux ou plusieurs serveurs WAP dans le réseau de périmètre/DMZ|
|Quelles sont les exigences Internet et réseau locales au-delà du système de provisionnement ?|Aucun|[Accès Internet sortant](/azure/active-directory/hybrid/how-to-connect-pta-quick-start) à partir des serveurs exécutant des agents d’authentification|[Accès Internet entrant](/windows-server/identity/ad-fs/overview/ad-fs-requirements) pour les serveurs WAP dans le périmètre<br><br>Accès réseau entrant aux serveurs AD FS à partir des serveurs WAP dans le périmètre<br><br>Équilibrage de charge réseau|
|Y a-t-il une exigence de certificat SSL ?|Non |Non |OUI|
|Existe-t-il une solution de surveillance de l’intégrité ?|Non requis|État de l’agent fourni par le [Centre d’administration Azure Active Directory](/azure/active-directory/hybrid/tshoot-connect-pass-through-authentication)|[Azure AD Connect Health](/azure/active-directory/hybrid/how-to-connect-health-adfs)|
|Les utilisateurs obtiennent-ils une authentification unique auprès des ressources cloud à partir d’appareils joints au domaine au sein du réseau d’entreprise ?|Oui avec [authentification unique fluide](/azure/active-directory/hybrid/how-to-connect-sso)|Oui avec [authentification unique fluide](/azure/active-directory/hybrid/how-to-connect-sso)|OUI|
|Quels types de connexion sont pris en charge ?|UserPrincipalName + mot de passe<br><br>Authentification Windows intégrée avec [authentification unique transparente](/azure/active-directory/hybrid/how-to-connect-sso)<br><br>[ID de connexion de substitution](/azure/active-directory/hybrid/how-to-connect-install-custom)|UserPrincipalName + mot de passe<br><br>Authentification Windows intégrée avec [authentification unique transparente](/azure/active-directory/hybrid/how-to-connect-sso)<br><br>[ID de connexion de substitution](/azure/active-directory/hybrid/how-to-connect-pta-faq)|UserPrincipalName + mot de passe<br><br>sAMAccountName + mot de passe<br><br>Authentification Windows intégrée<br><br>[Authentification par certificat et carte à puce](/windows-server/identity/ad-fs/operations/configure-user-certificate-authentication)<br><br>[ID de connexion de substitution](/windows-server/identity/ad-fs/operations/configuring-alternate-login-id)|
|Windows Hello Entreprise est-il pris en charge ?|[Modèle de confiance de clé](/windows/security/identity-protection/hello-for-business/hello-identity-verification)<br><br>[Modèle de certificat de confiance avec Intune](https://blogs.technet.microsoft.com/microscott/setting-up-windows-hello-for-business-with-intune/)|[Modèle de confiance de clé](/windows/security/identity-protection/hello-for-business/hello-identity-verification)<br><br>[Modèle de certificat de confiance avec Intune](https://blogs.technet.microsoft.com/microscott/setting-up-windows-hello-for-business-with-intune/)|[Modèle de confiance de clé](/windows/security/identity-protection/hello-for-business/hello-identity-verification)<br><br>[Modèle de certificat de confiance ](/windows/security/identity-protection/hello-for-business/hello-key-trust-adfs)|
|Quelles sont les options d’authentification multifacteur ?|[Azure MFA](/azure/multi-factor-authentication/)<br><br>[Contrôles personnalisés avec accès conditionnel*](/azure/active-directory/conditional-access/controls#custom-controls-1)|[Azure MFA](/azure/multi-factor-authentication/)<br><br>[Contrôles personnalisés avec accès conditionnel*](/azure/active-directory/conditional-access/controls#custom-controls-1)|[Azure MFA](/azure/multi-factor-authentication/)<br><br>[Serveur Azure MFA](/azure/active-directory/authentication/howto-mfaserver-deploy)<br><br>[MFA tiers](/windows-server/identity/ad-fs/operations/configure-additional-authentication-methods-for-ad-fs)<br><br>[Contrôles personnalisés avec accès conditionnel*](/azure/active-directory/conditional-access/controls#custom-controls-1)|
|Quels sont les états de compte d’utilisateur pris en charge ?|Comptes désactivés<br>(délai pouvant atteindre 30 minutes)|Comptes désactivés<br><br>Compte verrouillé<br><br>Compte expiré<br><br>Mot de passe expiré<br><br>Heures de connexion|Comptes désactivés<br><br>Compte verrouillé<br><br>Compte expiré<br><br>Mot de passe expiré<br><br>Heures de connexion|
|Quelles sont les options d’accès conditionnel ?|[Accès conditionnel Azure AD](/azure/active-directory/active-directory-conditional-access-azure-portal)|[Accès conditionnel Azure AD](/azure/active-directory/active-directory-conditional-access-azure-portal)|[Accès conditionnel Azure AD](/azure/active-directory/active-directory-conditional-access-azure-portal)<br><br>[Règles de revendication AD FS](https://adfshelp.microsoft.com/AadTrustClaims/ClaimsGenerator)|
|Le blocage des protocoles hérités est-il pris en charge ?|[Oui](/azure/active-directory/active-directory-conditional-access-conditions#legacy-authentication)|[Oui](/azure/active-directory/active-directory-conditional-access-conditions#legacy-authentication)|[Oui](/windows-server/identity/ad-fs/operations/access-control-policies-w2k12)|
|Pouvez-vous personnaliser le logo, l’image et la description sur les pages de connexion ?|[Oui, avec Azure AD Premium](/azure/active-directory/customize-branding)|[Oui, avec Azure AD Premium](/azure/active-directory/customize-branding)|[Oui](/azure/active-directory/connect/active-directory-aadconnect-federation-management#customlogo)|
|Quels sont les scénarios avancés pris en charge ?|[Verrouillage de mot de passe intelligent](/azure/active-directory/active-directory-secure-passwords)<br><br>[Rapports sur les informations d’identification divulguées](/azure/active-directory/active-directory-reporting-risk-events)|[Verrouillage de mot de passe intelligent](/azure/active-directory/connect/active-directory-aadconnect-pass-through-authentication-smart-lockout)|Système d’authentification multisite à faible latence<br><br>[Verrouillage extranet AD FS](/windows-server/identity/ad-fs/operations/configure-ad-fs-extranet-soft-lockout-protection)<br><br>[Intégration aux systèmes d’identité tiers](/azure/active-directory/connect/active-directory-aadconnect-federation-compatibility)|

<!-- markdownlint-enable MD033 -->

> [!NOTE]
> Contrôles personnalisés dans Azure AD. L’accès conditionnel ne prend actuellement pas en charge l’inscription d’appareil.

## <a name="next-steps"></a>Étapes suivantes

Le livre blanc [Hybrid Identity Digital Transformation Framework](https://resources.office.com/ww-landing-M365E-EMS-IDAM-Hybrid-Identity-WhitePaper.html?LCID=EN-US) décrit un certain nombre de combinaisons et de solutions pour le choix et l’intégration de chacun de ces éléments.

[L’outil Azure AD Connect](https://aka.ms/aadconnectwiz) vous aide à intégrer vos répertoires locaux à Azure AD.
