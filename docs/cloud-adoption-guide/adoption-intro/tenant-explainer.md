---
title: 'Fiche explicative : qu’est-ce qu’un client Azure Active Directory ?'
description: Explique le fonctionnement interne d’Azure Active Directory en matière d’identité en tant que service (IDaaS) dans Azure.
author: petertay
ms.openlocfilehash: ce5a33b92047e1f360eee8fcbc7a726bcf8cd19f
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/09/2018
---
# <a name="explainer-what-is-an-azure-active-directory-tenant"></a>Fiche explicative : qu’est-ce qu’un client Azure Active Directory ?

Dans l’article explicatif [Comment fonctionne Azure ?](azure-explainer.md), vous avez appris qu’Azure est une collection de serveurs et d’équipements de mise en réseau sur lesquels fonctionnent du matériel virtualisé et des logiciels pour le compte des utilisateurs. Vous savez également que certains de ces serveurs exécutent une application d’orchestration distribuée pour gérer la création, la lecture, la mise à jour et la suppression de ressources Azure.

Toutefois, Azure n’autorise évidemment pas n’importe qui à effectuer ces opérations sur une ressource. Azure en limite l’accès à l’aide d’un service d’identité numérique approuvé nommé **Azure Active Directory** (Azure AD). Azure AD stocke le nom d’utilisateur, les mots de passe, des données de profil et d’autres informations. Les utilisateurs Azure AD sont segmentés sous forme de **clients**. Un client est une construction logique qui représente une instance d’Azure AD dédiée et sécurisée, généralement associée à une organisation.

Pour créer un client, Azure demande un **compte privilégié**. Ce compte privilégié est associé soit à un compte Azure soit à un contrat entreprise. Les deux sont des constructions de facturation qui ne sont pas stockées dans Azure AD &mdash; ces comptes sont conservés dans une base de données de facturation très sécurisée. 

Une fois le client créé, un **ID de client** est généré et enregistré dans une base de données Azure AD interne et très sécurisé. Le propriétaire d’un compte privilégié peut alors se connecter à un Portail Azure et ajouter des utilisateurs à ce nouveau client Azure AD. 

La plupart des entreprises disposent déjà d’au moins un service de gestion des identités, généralement Active Directory Domain Services (AD DS). Azure AD est capable de synchroniser ou de fédérer l’identité des utilisateurs à partir d’AD DS, ce qui évite aux entreprises d’avoir à gérer les identités séparément dans les deux environnements. Les articles relatifs aux phases d’adoption intermédiaire et avancée décriront plus en détail l’identité numérique.

## <a name="next-steps"></a>étapes suivantes

* Maintenant que vous avez découvert les clients Azure AD, la première étape de la phase préparatoire à l’adoption consiste à savoir [comment obtenir un client Azure Active Directory][how-to-get-aad-tenant]. Ensuite, consultez [l’aide à la conception de clients Azure AD](tenant.md).

<!-- Links -->
[how-to-get-aad-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json