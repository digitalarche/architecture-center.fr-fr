---
title: "Explicatif : qu’est-ce qu’Azure Resource Manager ?"
description: "Explique le fonctionnement interne d’Azure Resource Manager"
author: petertay
ms.openlocfilehash: 60f09901bdc4b292abd73335b78c7d56a76f27a6
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/09/2018
---
# <a name="explainer-what-is-azure-resource-manager"></a>Explicatif : qu’est-ce qu’Azure Resource Manager ?

Dans l’article explicatif [Comment fonctionne Azure ?](azure-explainer.md), vous avez découvert l’architecture interne d’Azure. Cette architecture comprend un serveur frontal qui héberge les applications distribuées qui gèrent les services Azure internes.

Le serveur frontal Azure inclut un service appelé Azure Resource Manager. Ce service est responsable du cycle de vie des ressources hébergées dans Azure, de leur création à leur suppression. Il existe plusieurs moyens d’interagir avec Azure Resource Manager (à l’aide de Powershell, de l’interface de ligne de commande Azure et des kits SDK), mais chacun de ces outils est simplement un wrapper sur une API RESTful hébergée par Azure Resource Manager.

L’API RESTful fournie par Azure Resource Manager est une interface cohérente à travers un ensemble de **fournisseurs de ressources**. Les fournisseurs de ressources sont simplement des services Azure qui créent, lisent, suppriment et mettent à jour des ressources dans Azure. De fait, l’API RESTful inclut des méthodes pour chacune de ces fonctions. 

L’API RESTful nécessite un jeton d’accès pour l’utilisateur, un **ID d’abonnement** et désormais un nouvel élément, à savoir un **ID de groupe de ressources**. Nous aborderons les groupes de ressources dans l’article explicatif sur les [groupes de ressources](resource-group-explainer.md). Azure Resource Manager nécessite également l’**ID de locataire**, qui est codé en tant qu’élément du jeton d’accès. 

Après la réception d’un appel d’API valide pour créer une ressource, Azure Resource Manager trouve la capacité dans la région spécifiée et copie tous les fichiers nécessaires dans un emplacement intermédiaire. La demande est ensuite envoyée au contrôleur de structure dans le rack, qui se charge alors d’allouer les ressources. Le contrôleur de structure répond à la demande en émettant une notification d’échec ou de réussite, et un **ID de ressource** pour la nouvelle ressource créée. Ces quatre ID sont stockés en interne dans Azure et forment ensemble un identificateur unique pour une ressource déployée.

## <a name="next-steps"></a>étapes suivantes

* Maintenant que vous en savez davantage sur le fonctionnement interne d’Azure Resource Manager, découvrez comment les [groupes de ressources](resource-group-explainer.md) peuvent vous aider à créer votre premier groupe de ressources.
