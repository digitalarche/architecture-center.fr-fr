---
title: Modèles de sécurité
description: La sécurité est la capacité d’un système à empêcher des actions accidentelles ou malveillantes en dehors de l’utilisation prévue et à empêcher la divulgation ou la perte d’informations. Les applications cloud sont exposées sur Internet en dehors des limites de confiance locales, elles sont souvent publiquement accessibles et peuvent être utilisées par des utilisateurs non approuvés. Les applications doivent être conçues et déployées d’une manière qui les protège des attaques malveillantes, restreint l’accès aux seuls les utilisateurs approuvés et protège les données sensibles.
keywords: modèle de conception
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 8437b8dfef751226580437a1b5678ca0e0e71f18
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/06/2018
ms.locfileid: "30846996"
---
# <a name="security-patterns"></a>Modèles de sécurité

[!INCLUDE [header](../../_includes/header.md)]

La sécurité est la capacité d’un système à empêcher des actions accidentelles ou malveillantes en dehors de l’utilisation prévue et à empêcher la divulgation ou la perte d’informations. Les applications cloud sont exposées sur Internet en dehors des limites de confiance locales, elles sont souvent publiquement accessibles et peuvent être utilisées par des utilisateurs non approuvés. Les applications doivent être conçues et déployées d’une manière qui les protège des attaques malveillantes, restreint l’accès aux seuls les utilisateurs approuvés et protège les données sensibles.


|                    Modèle                     |                                                                                                         Résumé                                                                                                         |
|------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Identité fédérée](../federated-identity.md) |                                                                                Déléguez l’authentification à un fournisseur d’identité externe.                                                                                |
|         [Opérateur de contrôle](../gatekeeper.md)         | Protégez les applications et services à l’aide d’une instance d’hôte dédiée qui agit comme un intermédiaire entre les clients et l’application ou le service, valide et assainit les requêtes, et transmet les requêtes et les données entre eux. |
|          [Clé Valet](../valet-key.md)          |                                                        Utilisez un jeton ou une clé qui fournissent aux clients un accès direct limité à une ressource ou un service spécifique.                                                        |

