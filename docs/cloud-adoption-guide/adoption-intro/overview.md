---
title: "Adoption d’Azure : phase initiale"
description: "Décrit le niveau de base de connaissances que doit posséder une entreprise pour adopter Azure"
author: petertay
ms.openlocfilehash: e9421b610e4eb07a3ed37bca56e513b0689484ef
ms.sourcegitcommit: 9ba82cf84cee06ccba398ec04c51dab0e1ca8974
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/13/2018
---
# <a name="adopting-azure-foundational"></a>Adoption d’Azure : phase initiale

L’adoption d’Azure est la première phase de maturité organisationnelle d’une entreprise. À la fin de cette phase, les membres de votre organisation peuvent déployer des charges de travail simples sur Azure.

La liste ci-dessous répertorie les tâches à accomplir durant la phase d’adoption initiale. La liste étant progressive, chaque tâche doit être effectuée dans l’ordre indiqué. Si vous avez déjà effectué une tâche particulière, passez à la tâche suivante de la liste. 

1. Présentation des éléments internes d’Azure :
    - **Explicatif :** [comment fonctionne Azure ?](azure-explainer.md)
2. Présentation de l’identité numérique d’entreprise dans Azure :
    - **Explicatif :** [qu’est-ce qu’un locataire Azure Active Directory ?](tenant-explainer.md)
    - **Guide pratique pour** [obtenir un locataire Azure Active Directory](/azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **Guide :** [conception du locataire Azure AD](tenant.md)
    - **Guide pratique pour** [ajouter de nouveaux utilisateurs à Azure Active Directory](/azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json)    
3. Présentation des abonnements dans Azure :
    - **Explicatif :** [qu’est-ce qu’un abonnement Azure ?](subscription-explainer.md)
    - **Guide :** [conception d’un abonnement Azure](subscription.md)
4. Présentation de la gestion des ressources dans Azure : 
    - **Explicatif :** [qu’est-ce qu’Azure Resource Manager ?](resource-manager-explainer.md)
    - **Explicatif :** [qu’est-ce qu’un groupe de ressources Azure ?](resource-group-explainer.md)
    - **Explicatif :** [présentation de l’accès aux ressources dans Azure](/azure/active-directory/active-directory-understanding-resource-access?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **Guide pratique pour** [créer un groupe de ressources Azure à l’aide du portail Azure](/azure/azure-resource-manager/resource-group-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **Guide :** [conception d’un groupe de ressources Azure](resource-group.md)
    - **Guide :** [conventions d’affectation de noms pour les ressources Azure](/azure/architecture/best-practices/naming-conventions?toc=/azure/architecture/cloud-adoption-guide/toc.json)
5. Déploiement d’une architecture Azure de base :
    - Découvrez-en plus sur les différents types d’options de calcul Azure, tels que IaaS (Infrastructure as a Service) et PaaS (Platform as a Service) dans la [vue d’ensemble des options de calcul Azure](/azure/architecture/guide/technology-choices/compute-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json).
    - Maintenant que vous connaissez les différents types d’options de calcul Azure, choisissez une application web PaaS ou une machine virtuelle IaaS comme première ressource dans Azure :
    - PaaS : introduction à Platform as a Service :
        - **Guide pratique pour** [déployer une application web de base sur Azure](/azure/app-service/app-service-web-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Guide :** pratiques éprouvées pour le déploiement d’une [application web de base](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json) sur Azure
    - IaaS : introduction au réseau virtuel :
        - **Explicatif :** [réseau virtuel Azure](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Guide pratique pour** [déployer un réseau virtuel sur Azure à l’aide du portail](/azure/virtual-network/virtual-networks-create-vnet-arm-pportal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - IaaS : déploiement d’une charge de travail de machine virtuelle unique (Windows et Linux) :
        - **Guide pratique pour** [déployer une machine virtuelle Windows sur Azure à l’aide du portail](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Guide :** [pratiques éprouvées pour l’exécution d’une machine virtuelle Windows sur Azure](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Guide pratique pour** [déployer une machine virtuelle Linux sur Azure à l’aide du portail](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Guide :** [pratiques éprouvées pour l’exécution d’une machine virtuelle Linux sur Azure](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)
