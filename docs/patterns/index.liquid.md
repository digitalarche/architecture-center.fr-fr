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
# <a name="cloud-design-patterns"></a>Modèles de conception de cloud

[!INCLUDE [header](../../_includes/header.md)]

Ces modèles de conception sont utiles pour développer des applications fiables, évolutives et sécurisées dans le cloud.

Chaque modèle décrit le problème traité par le modèle, les aspects à prendre en compte pour l’application du modèle et un exemple basé sur Microsoft Azure. La plupart des modèles incluent des exemples ou des extraits de code qui montrent comment implémenter le modèle sur Azure. La plupart des modèles sont appropriés à n’importe quel système distribué, qu’il soit hébergé sur Azure ou sur d’autres plateformes cloud.

## <a name="problem-areas-in-the-cloud"></a>Zones à problème du cloud

<!-- markdownlint-disable MD033 -->

<ul id="categories" class="panel">
{%- for category in categories %}
    <li>
    {% include ’pattern-category-card’ %}
    </li>
{%- endfor %}
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="catalog-of-patterns"></a>Catalogue de modèles

| Modèle | Résumé |
|---------|---------|
|         |         |

{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}
