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
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54482083"
---
# <a name="security-patterns"></a><span data-ttu-id="6e1a8-106">Modèles de sécurité</span><span class="sxs-lookup"><span data-stu-id="6e1a8-106">Security patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="6e1a8-107">La sécurité est la capacité d’un système à empêcher des actions accidentelles ou malveillantes en dehors de l’utilisation prévue et à empêcher la divulgation ou la perte d’informations.</span><span class="sxs-lookup"><span data-stu-id="6e1a8-107">Security is the capability of a system to prevent malicious or accidental actions outside of the designed usage, and to prevent disclosure or loss of information.</span></span> <span data-ttu-id="6e1a8-108">Les applications cloud sont exposées sur Internet en dehors des limites de confiance locales, elles sont souvent publiquement accessibles et peuvent être utilisées par des utilisateurs non approuvés.</span><span class="sxs-lookup"><span data-stu-id="6e1a8-108">Cloud applications are exposed on the Internet outside trusted on-premises boundaries, are often open to the public, and may serve untrusted users.</span></span> <span data-ttu-id="6e1a8-109">Les applications doivent être conçues et déployées d’une manière qui les protège des attaques malveillantes, restreint l’accès aux seuls les utilisateurs approuvés et protège les données sensibles.</span><span class="sxs-lookup"><span data-stu-id="6e1a8-109">Applications must be designed and deployed in a way that protects them from malicious attacks, restricts access to only approved users, and protects sensitive data.</span></span>

|                    <span data-ttu-id="6e1a8-110">Modèle</span><span class="sxs-lookup"><span data-stu-id="6e1a8-110">Pattern</span></span>                     |                                                                                                         <span data-ttu-id="6e1a8-111">Résumé</span><span class="sxs-lookup"><span data-stu-id="6e1a8-111">Summary</span></span>                                                                                                         |
|------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [<span data-ttu-id="6e1a8-112">Identité fédérée</span><span class="sxs-lookup"><span data-stu-id="6e1a8-112">Federated Identity</span></span>](../federated-identity.md) |                                                                                <span data-ttu-id="6e1a8-113">Déléguez l’authentification à un fournisseur d’identité externe.</span><span class="sxs-lookup"><span data-stu-id="6e1a8-113">Delegate authentication to an external identity provider.</span></span>                                                                                |
|         [<span data-ttu-id="6e1a8-114">Opérateur de contrôle</span><span class="sxs-lookup"><span data-stu-id="6e1a8-114">Gatekeeper</span></span>](../gatekeeper.md)         | <span data-ttu-id="6e1a8-115">Protégez les applications et services à l’aide d’une instance d’hôte dédiée qui agit comme un intermédiaire entre les clients et l’application ou le service, valide et assainit les requêtes, et transmet les requêtes et les données entre eux.</span><span class="sxs-lookup"><span data-stu-id="6e1a8-115">Protect applications and services by using a dedicated host instance that acts as a broker between clients and the application or service, validates and sanitizes requests, and passes requests and data between them.</span></span> |
|          [<span data-ttu-id="6e1a8-116">Clé Valet</span><span class="sxs-lookup"><span data-stu-id="6e1a8-116">Valet Key</span></span>](../valet-key.md)          |                                                        <span data-ttu-id="6e1a8-117">Utilisez un jeton ou une clé qui fournissent aux clients un accès direct limité à une ressource ou un service spécifique.</span><span class="sxs-lookup"><span data-stu-id="6e1a8-117">Use a token or key that provides clients with restricted direct access to a specific resource or service.</span></span>                                                        |
