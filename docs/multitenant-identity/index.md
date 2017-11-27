---
title: "Gestion des identités pour les applications multi-locataire"
description: "Meilleures pratiques pour la gestion de l’authentification, de l’autorisation et de l’identité dans les applications multi-locataires."
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.next: tailspin
ms.openlocfilehash: c363ac01e798b522fa95f39586e28fe3af5fae4a
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2017
---
# <a name="manage-identity-in-multitenant-applications"></a>Gérer l’identité dans les applications multi-locataire

Cette série d’articles décrit les meilleures pratiques pour les applications multi-locataires, lors de l’utilisation d’Azure AD pour l’authentification et la gestion des identités.

[![GitHub](../_images/github.png) Exemple de code][sample application]

Lorsque vous générez une les applications multi-locataire, l’une des premières difficultés est la gestion des identités utilisateur, car désormais chaque utilisateur appartient à un client. Par exemple :

* Les utilisateurs se connectent avec les informations d’identification de leur organisation.
* Les utilisateurs doivent avoir accès aux données de leur organisation, mais pas aux données appartenant à d’autres clients.
* Une organisation peut s’inscrire auprès de l’application, puis affecter des rôles d’application à ses membres.

Azure Active Directory (Azure AD) possède des fonctionnalités qui prennent en charge tous ces scénarios.

Pour accompagner cette série d’articles, nous avons créé une [implémentation de bout en bout] [ sample application] d’une application multi-locataire. Les articles reflètent ce que nous avons appris au cours de la création de l’application. Pour vous familiariser avec l’application, consultez [Exécution de l’application Surveys][running-the-app].

## <a name="introduction"></a>Introduction

Supposons que vous écriviez une application SaaS d’entreprise pour qu’elle soit hébergée dans le cloud. Bien sûr, cette application aura des utilisateurs :

![Utilisateurs](./images/users.png)

Mais ces utilisateurs appartiennent à des organisations :

![Utilisateurs d’organisation](./images/org-users.png)

Exemple : Tailspin vend des abonnements à son application SaaS. Contoso et Fabrikam s’inscrivent à l’application. Quand Alice (`alice@contoso`) s’inscrit, l’application doit savoir qu’Alice fait partie de Contoso.

* Alice *doit* avoir accès aux données Contoso.
* Alice *ne doit pas* avoir accès aux données Fabrikam.

Ce guide vous explique comment gérer des identités d’utilisateurs dans une application multi-locataire, en utilisant [Azure Active Directory][AzureAD] (Azure AD) pour gérer la connexion et l’authentification.

## <a name="what-is-multitenancy"></a>Qu’est-ce que l’architecture mutualisée ?
Un *locataire* est un groupe d’utilisateurs. Dans une application SaaS, le locataire est un abonné ou un client de l’application. Une *architecture mutualisée* est une architecture où plusieurs locataires partagent la même instance physique de l’application. Bien que les locataires partagent des ressources physiques (par exemple, les machines virtuelles ou le stockage), chaque locataire obtient sa propre instance logique de l’application.

Normalement, les données d’application sont partagées entre les utilisateurs d’un locataire, mais pas avec d’autres locataires.

![Multi-locataire](./images/multitenant.png)

Comparez cette architecture avec une architecture à locataire unique, où chaque locataire a une instance physique dédiée. Dans une architecture à locataire unique, vous ajoutez des locataires mettant en place de nouvelles instances de l’application.

![Locataire unique](./images/single-tenant.png)

### <a name="multitenancy-and-horizontal-scaling"></a>Architecture mutualisée et mise à l’échelle horizontale
Pour atteindre une mise à l’échelle dans le cloud, il est courant d’ajouter des instances physiques supplémentaires. Il s’agit de la *mise à l’échelle horizontale* ou de l’*augmentation de la taille des instances*. Prenons l’exemple d’une application web. Pour gérer davantage de trafic, vous pouvez ajouter des machines virtuelles serveurs supplémentaires et les placer derrière un équilibreur de charge. Chaque machine virtuelle exécute une instance physique distincte de l’application web.

![Équilibrage de la charge d’un site web](./images/load-balancing.png)

Toute requête peut être acheminée vers n’importe quelle instance. Globalement, le système fonctionne comme une instance logique unique. Vous pouvez supprimer une machine virtuelle ou en ajouter une nouvelle, sans affecter les utilisateurs. Dans cette architecture, chaque instance physique est mutualisée, et vous effectuez la mise à l’échelle en ajoutant des instances supplémentaires. Si une instance subit une défaillance, cela ne doit affecter aucun locataire.

## <a name="identity-in-a-multitenant-app"></a>Identité dans une application multi-locataire
Dans une application multi-locataire, vous devez considérer les utilisateurs dans le contexte des locataires.

**Authentification**

* Les utilisateurs se connectent à l’application avec les informations d’identification de leur organisation. Ils n’ont pas à créer de nouveaux profils utilisateur pour l’application.
* Les utilisateurs appartenant à la même organisation font partie du même locataire.
* Quand un utilisateur se connecte, l’application sait à quel locataire l’utilisateur appartient.

**Autorisation**

* Lors de l’autorisation d’actions d’un utilisateur (par exemple, l’affichage d’une ressource), l’application doit prendre en compte le locataire de l’utilisateur.
* Des rôles peuvent être affectés aux utilisateurs de dans l’application, comme « Admin » ou « Utilisateur Standard ». Les attributions de rôles doivent être gérées par le client, et non par le fournisseur SaaS.

**Exemple.** Alice, une employé de Contoso, navigue vers l’application dans son navigateur et clique sur le bouton « Se connecter ». Elle est redirigée vers un écran de connexion où elle entre ses informations d’identification d’entreprise (nom d’utilisateur et mot de passe). À ce stade, elle est connectée à l’application en tant que `alice@contoso.com`. L’application sait également qu’Alice est une utilisatrice administratrice de cette application. Son statut d’administratrice lui permet de voir une liste de toutes les ressources appartenant à Contoso. Toutefois, elle ne peut pas afficher les ressources de Fabrikam, car elle est administratrice uniquement au sein de son client.

Dans ce guide, nous examinerons plus particulièrement l’utilisation d’Azure AD pour la gestion des identités.

* Nous supposons que le client stocke ses profils utilisateur dans Azure AD (notamment les locataires Office 365 et Dynamics CRM).
* Les clients disposant d’Active Directory (AD) sur site peuvent utiliser [Azure AD Connect][ADConnect] pour synchroniser leur AD sur site auprès d’Azure AD.

Si un client disposant d’AD local ne peut pas utiliser Azure AD Connect (en raison d’une stratégie informatique d’entreprise ou pour d’autres raisons), le fournisseur SaaS peut fédérer avec l’AD du client via les services ADFS (Active Directory Federation Services). Cette option est décrite dans [Fédération avec les services ADFS d’un client].

Ce guide ne prend pas en considération les autres aspects d’une architecture mutualisée, comme le partitionnement des données, la configuration par locataire, etc.

[**Suivant**][tailpin]



<!-- Links -->
[ADConnect]: /azure/active-directory/active-directory-aadconnect
[AzureAD]: /azure/active-directory

[Fédération avec les services ADFS d’un client]: adfs.md
[tailpin]: tailspin.md

[running-the-app]: ./run-the-app.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
