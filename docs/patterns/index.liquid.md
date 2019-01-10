---
title: Modèles de conception de cloud
titleSuffix: Azure Architecture Center
description: Modèles de conception de cloud pour Microsoft Azure
keywords: Azure
author: dragon119
ms.date: 12/10/2018
ms.custom: seodec18
ms.openlocfilehash: 873d4cf02690a2cc3ffe4f35b044dedf70700fb5
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011019"
---
# <a name="cloud-design-patterns"></a><span data-ttu-id="48b88-104">Modèles de conception de cloud</span><span class="sxs-lookup"><span data-stu-id="48b88-104">Cloud Design Patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="48b88-105">Ces modèles de conception sont utiles pour développer des applications fiables, évolutives et sécurisées dans le cloud.</span><span class="sxs-lookup"><span data-stu-id="48b88-105">These design patterns are useful for building reliable, scalable, secure applications in the cloud.</span></span>

<span data-ttu-id="48b88-106">Chaque modèle décrit le problème traité par le modèle, les aspects à prendre en compte pour l’application du modèle et un exemple basé sur Microsoft Azure.</span><span class="sxs-lookup"><span data-stu-id="48b88-106">Each pattern describes the problem that the pattern addresses, considerations for applying the pattern, and an example based on Microsoft Azure.</span></span> <span data-ttu-id="48b88-107">La plupart des modèles incluent des exemples ou des extraits de code qui montrent comment implémenter le modèle sur Azure.</span><span class="sxs-lookup"><span data-stu-id="48b88-107">Most of the patterns include code samples or snippets that show how to implement the pattern on Azure.</span></span> <span data-ttu-id="48b88-108">La plupart des modèles sont appropriés à n’importe quel système distribué, qu’il soit hébergé sur Azure ou sur d’autres plateformes cloud.</span><span class="sxs-lookup"><span data-stu-id="48b88-108">However, most of the patterns are relevant to any distributed system, whether hosted on Azure or on other cloud platforms.</span></span>

## <a name="problem-areas-in-the-cloud"></a><span data-ttu-id="48b88-109">Zones à problème du cloud</span><span class="sxs-lookup"><span data-stu-id="48b88-109">Problem areas in the cloud</span></span>

<!-- markdownlint-disable MD033 -->

<ul id="categories" class="panel">
<span data-ttu-id="48b88-110">{%- for category in categories %}</span><span class="sxs-lookup"><span data-stu-id="48b88-110">{%- for category in categories %}</span></span>
    <li>
    <span data-ttu-id="48b88-111">{% include ’pattern-category-card’ %}</span><span class="sxs-lookup"><span data-stu-id="48b88-111">{% include 'pattern-category-card' %}</span></span>
    </li>
<span data-ttu-id="48b88-112">{%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="48b88-112">{%- endfor %}</span></span>
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="catalog-of-patterns"></a><span data-ttu-id="48b88-113">Catalogue de modèles</span><span class="sxs-lookup"><span data-stu-id="48b88-113">Catalog of patterns</span></span>

| <span data-ttu-id="48b88-114">Modèle</span><span class="sxs-lookup"><span data-stu-id="48b88-114">Pattern</span></span> | <span data-ttu-id="48b88-115">Résumé</span><span class="sxs-lookup"><span data-stu-id="48b88-115">Summary</span></span> |
|---------|---------|
|         |         |

<span data-ttu-id="48b88-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="48b88-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span></span>
