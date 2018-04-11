---
title: 'Explicatif : qu’est-ce qu’un groupe de ressources Azure ?'
description: Explique la fonction Azure interne d’un groupe de ressources
author: petertay
ms.openlocfilehash: e7c7334bd88c28f57498486bd2bed3c349565222
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/09/2018
---
# <a name="what-is-an-azure-resource-group"></a>Qu’est-ce qu’un groupe de ressources Azure ?

Dans l’article explicatif [Qu’est-ce qu’Azure Resource Manager ?](resource-manager-explainer.md), vous avez appris qu’Azure Resource Manager nécessite un **identificateur du groupe de ressources** lorsqu’un appel est effectué pour créer, lire, mettre à jour ou supprimer une ressource. Cet ID de groupe de ressources fait référence à un **groupe de ressources**. Un groupe de ressources est simplement un identificateur qu’Azure Resource Manager applique aux ressources pour les regrouper. Cet ID de groupe de ressources permet à Azure Resource Manager d’effectuer des opérations sur un groupe de ressources qui partagent cet ID.

Par exemple, un utilisateur peut effectuer un appel **supprimer** pour une API RESTful Azure Resource Manager en spécifiant l’ID de groupe de ressources sans inclure d’ID de ressource spécifique. Azure Resource Manager interroge une base de données Azure interne pour toutes les ressources avec l’ID de groupe de ressources spécifiée et appelle l’API RESTful pour supprimer chaque ressource.

Un groupe de ressources ne peut pas inclure des ressources à partir de différents abonnements. Ceci car il existe une relation un-à-plusieurs entre les ID de locataire et d’abonnement &mdash; plusieurs abonnements peuvent faire confiance au même client pour fournir l’authentification et autorisation, mais chaque abonnement ne peut faire confiance qu’à un seul client. Il existe également une relation un-à-plusieurs entre les ID d’abonnement et de groupe de ressources &mdash; plusieurs groupes de ressources peuvent appartenir au même abonnement, mais chaque groupe de ressources ne peut appartenir qu’à un seul abonnement. Enfin, il existe une relation un-à-plusieurs entre les ID de groupe de ressources et de ressource &mdash; un seul groupe de ressources peut avoir plusieurs ressources, mais chaque ressource ne peut appartenir qu’à un seul groupe de ressources.

## <a name="next-steps"></a>étapes suivantes

* Maintenant que vous en savez plus sur les groupes de ressources Azure, développez des connaissances de base [en ce qui concerne la restriction de l’accès aux ressources](/azure/active-directory/active-directory-understanding-resource-access?toc=/azure/architecture/cloud-adoption-guide/toc.json). Bien que cette notion ne fasse pas partie des fondamentales dans la phase d’adoption, elle est importante au cours de la phase d’adoption intermédiaire. Vous pourrez alors [créer votre premier groupe de ressources](/azure/azure-resource-manager/resource-group-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json) et consulter le [guide de conception des groupes de ressources Azure](resource-group.md).
