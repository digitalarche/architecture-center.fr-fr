---
title: "Explicatif : comment Azure fonctionne-t-il ?"
description: "Explique le fonctionnement interne d’Azure"
author: petertay
ms.openlocfilehash: 847d24b7057d80f3d34aac7900cfb64fec60a640
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/09/2018
---
# <a name="explainer-how-does-azure-work"></a>Explicatif : comment Azure fonctionne-t-il ?

Azure est la plateforme de cloud public de Microsoft. Azure offre une grande collection de services, y compris la plateforme en tant que service (PaaS), l’infrastructure en tant que service (IaaS), la base de données en tant que service (DBaaS) et bien d’autres. Mais ce qu’est -ce qu’Azure exactement et comment fonctionne-t-il ?

Azure, comme les autres plateformes de cloud, s’appuie sur une technologie appelée **virtualisation**. Le matériel informatique peut être, en général, émulé dans le logiciel, car il s’agit simplement d’un ensemble d’instructions encodées de façon permanente ou semi-permanente en silicium. À l’aide d’une couche d’émulation qui mappe les instructions du logiciel pour obtenir des instructions de matériel, le matériel virtualisé peut s’exécuter dans le logiciel comme s’il s’agissait du matériel réel lui-même.

Fondamentalement, le cloud est un ensemble de serveurs physiques dans un ou plusieurs centres de données qui exécutent un matériel virtualisé pour le compte de clients. Ainsi, comment le cloud crée, démarre, arrête et supprime-t-il simultanément des millions d’instances d’un matériel virtualisé pour des millions de clients ?

Pour le comprendre, examinons l’architecture du matériel dans le centre de données.  Dans chaque centre de données se trouve une collection de serveurs dans les racks de serveurs. Chaque rack de serveurs contient de nombreuses **lames** de serveurs ainsi qu’un commutateur de réseau fournissant une connectivité réseau et une unité de distribution de l’alimentation (PDU) fournissant l’électricité. Les racks sont parfois regroupés en unités plus grandes appelées **clusters**. 

Dans chaque rack ou cluster, la plupart des serveurs sont désignés pour exécuter ces instances du matériel virtualisé au nom de l’utilisateur. Toutefois, plusieurs serveurs exécutent le logiciel de gestion du cloud connu sous le nom de contrôleur de structure. Le **contrôleur de structure** est une application distribuée avec beaucoup de responsabilités. Il alloue des services, analyse l’intégrité du serveur et les services qui y sont exécutés et répare des serveurs en cas d’échec.

Chaque instance du contrôleur de structure est connectée à un autre ensemble de serveurs exécutant le logiciel d’orchestration du cloud, généralement appelé **frontal**. Le serveur frontal héberge les services web, les API RESTful et les bases de données Azure internes utilisés pour toutes les fonctions exécutées par le cloud. 

Par exemple, le serveur frontal héberge les services qui gèrent les demandes clients pour allouer des ressources Azure telles que [réseaux virtuels][vnet], [machines virtuelles] [ vms]et des services tels que [CosmosDB]. Tout d’abord, le serveur frontal valide l’utilisateur et vérifie que l’utilisateur est autorisé à allouer les ressources demandées. Dans ce cas, le serveur frontal consulte une base de données pour localiser un rack de serveurs avec une capacité suffisante, puis fait en sorte que le contrôleur de structure sur le rack alloue la ressource.

Par conséquent, tout simplement, Azure est une vaste collection de serveurs et de matériel réseau, ainsi qu’un ensemble complexe d’applications distribuées qui orchestrent la configuration et l’opération du matériel et des logiciels virtualisés sur ces serveurs. Cette orchestration rend alors Azure tellement puissant : les utilisateurs ne sont plus responsables du suivi ni de la mise à niveau du matériel, Azure effectue tout cela en arrière-plan. 

## <a name="next-steps"></a>étapes suivantes

* Maintenant que vous comprenez le fonctionnement interne d’Azure, la première étape pour l’adoption d’Azure consiste à [comprendre l’identité numérique dans Azure](tenant-explainer.md). Vous pouvez alors enfin [créer votre premier utilisateur dans Azure AD][docs-add-users-to-aad].

<!-- Links -->

[cosmosdb]: /azure/cosmos-db/introduction
[docs-add-users-to-aad]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[vms]: /azure/virtual-machines/
[vnet]: /azure/virtual-network/virtual-networks-overview