---
title: Modèles de sécurité
titleSuffix: Cloud Design Patterns
description: La sécurité est la capacité d’un système à empêcher des actions accidentelles ou malveillantes en dehors de l’utilisation prévue et à empêcher la divulgation ou la perte d’informations. Les applications cloud sont exposées sur Internet en dehors des limites de confiance locales, elles sont souvent publiquement accessibles et peuvent être utilisées par des utilisateurs non approuvés. Les applications doivent être conçues et déployées d’une manière qui les protège des attaques malveillantes, restreint l’accès aux seuls les utilisateurs approuvés et protège les données sensibles.
keywords: modèle de conception
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: c18255dccdbd8659705884a4141f5ecd099d9ece
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58248340"
---
# <a name="security-patterns"></a>Modèles de sécurité

[!INCLUDE [header](../../_includes/header.md)]

La sécurité est la capacité d’un système à empêcher des actions accidentelles ou malveillantes en dehors de l’utilisation prévue et à empêcher la divulgation ou la perte d’informations. Les applications cloud sont exposées sur Internet en dehors des limites de confiance locales, elles sont souvent publiquement accessibles et peuvent être utilisées par des utilisateurs non approuvés. Les applications doivent être conçues et déployées d’une manière qui les protège des attaques malveillantes, restreint l’accès aux seuls les utilisateurs approuvés et protège les données sensibles.

|                    Modèle                     |                                                                                                         Résumé                                                                                                         |
|------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Identité fédérée](../federated-identity.md) |                                                                                Déléguez l’authentification à un fournisseur d’identité externe.                                                                                |
|         [Opérateur de contrôle](../gatekeeper.md)         | Protégez les applications et services à l’aide d’une instance d’hôte dédiée qui agit comme un intermédiaire entre les clients et l’application ou le service, valide et assainit les requêtes, et transmet les requêtes et les données entre eux. |
|          [Clé Valet](../valet-key.md)          |                                                        Utilisez un jeton ou une clé qui fournissent aux clients un accès direct limité à une ressource ou un service spécifique.                                                        |
