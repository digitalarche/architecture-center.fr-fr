---
title: Modèles de conception de cloud
description: Modèles de conception de cloud pour Microsoft Azure
keywords: Azure
ms.openlocfilehash: 4747c896fc6fc5866be782d76c5290d6b49ad451
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/06/2018
---
# <a name="cloud-design-patterns"></a>Modèles de conception de cloud

[!INCLUDE [header](../../_includes/header.md)]

Ces modèles de conception sont utiles pour développer des applications fiables, évolutives et sécurisées dans le cloud.

Chaque modèle décrit le problème traité par le modèle, les aspects à prendre en compte pour l’application du modèle et un exemple basé sur Microsoft Azure. La plupart des modèles incluent des exemples ou des extraits de code qui montrent comment implémenter le modèle sur Azure. La plupart des modèles sont appropriés à n’importe quel système distribué, qu’il soit hébergé sur Azure ou sur d’autres plateformes cloud.

## <a name="problem-areas-in-the-cloud"></a>Zones à problème du cloud

<ul id="categories" class="panel">
{%- for category in categories %}
    <li>
    {% include ’pattern-category-card’ %}
    </li>
{%- endfor %}
</ul>

## <a name="catalog-of-patterns"></a>Catalogue de modèles

| Modèle | Résumé |
|---------|---------|
|         |         |

{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}
