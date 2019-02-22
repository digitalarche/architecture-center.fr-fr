---
title: 'Framework d’adoption du cloud : Fonctionnement d’Azure'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
description: Explication sur le fonctionnement interne d’Azure
author: petertaylor9999
ms.date: 2/11/2019
ms.openlocfilehash: 5ce9f0535584cbc45d757be5aa6f2fd64c7cc39f
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55898066"
---
<!-- markdownlint-disable MD026 -->

# <a name="how-does-azure-work"></a>Fonctionnement d’Azure

Azure est la plateforme de cloud public de Microsoft. Azure offre une grande collection de services, y compris la plateforme en tant que service (PaaS), l’infrastructure en tant que service (IaaS), la base de données en tant que service (DBaaS) et bien d’autres. Mais ce qu’est -ce qu’Azure exactement et comment fonctionne-t-il ?

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2ixGo]

Azure, comme les autres plateformes de cloud, s’appuie sur une technologie appelée **virtualisation**. Le matériel informatique peut être, en général, émulé dans le logiciel, car il s’agit simplement d’un ensemble d’instructions encodées de façon permanente ou semi-permanente en silicium. À l’aide d’une couche d’émulation qui mappe les instructions du logiciel pour obtenir des instructions de matériel, le matériel virtualisé peut s’exécuter dans le logiciel comme s’il s’agissait du matériel réel lui-même.

Fondamentalement, le cloud est un ensemble de serveurs physiques dans un ou plusieurs centres de données qui exécutent un matériel virtualisé pour le compte de clients. Ainsi, comment le cloud crée, démarre, arrête et supprime-t-il simultanément des millions d’instances d’un matériel virtualisé pour des millions de clients ?

Pour le comprendre, examinons l’architecture du matériel dans le centre de données.  Dans chaque centre de données se trouve une collection de serveurs dans les racks de serveurs. Chaque rack de serveurs contient de nombreuses **lames** de serveurs ainsi qu’un commutateur de réseau fournissant une connectivité réseau et une unité de distribution de l’alimentation (PDU) fournissant l’électricité. Les racks sont parfois regroupés en unités plus grandes appelées **clusters**.

Dans chaque rack ou cluster, la plupart des serveurs sont désignés pour exécuter ces instances du matériel virtualisé au nom de l’utilisateur. Toutefois, plusieurs serveurs exécutent le logiciel de gestion du cloud connu sous le nom de contrôleur de structure. Le **contrôleur de structure** est une application distribuée avec beaucoup de responsabilités. Il alloue des services, analyse l’intégrité du serveur et les services qui y sont exécutés et répare des serveurs en cas d’échec.

Chaque instance du contrôleur de structure est connectée à un autre ensemble de serveurs exécutant le logiciel d’orchestration du cloud, généralement appelé **frontal**. Le serveur frontal héberge les services web, les API RESTful et les bases de données Azure internes utilisés pour toutes les fonctions exécutées par le cloud.

Par exemple, le serveur frontal héberge les services qui gèrent les demandes clients pour allouer des ressources Azure telles que des [réseaux virtuels][vnet], [machines virtuelles][vms]et des services tels que [CosmosDB][cosmosdb]. Tout d’abord, le serveur frontal valide l’utilisateur et vérifie que l’utilisateur est autorisé à allouer les ressources demandées. Dans ce cas, le serveur frontal consulte une base de données pour localiser un rack de serveurs avec une capacité suffisante, puis fait en sorte que le contrôleur de structure sur le rack alloue la ressource.

Par conséquent, tout simplement, Azure est une vaste collection de serveurs et de matériel réseau, ainsi qu’un ensemble complexe d’applications distribuées qui orchestrent la configuration et l’opération du matériel et des logiciels virtualisés sur ces serveurs. Cette orchestration rend alors Azure tellement puissant : les utilisateurs ne sont plus responsables du suivi ni de la mise à niveau du matériel, Azure effectue tout cela en arrière-plan.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous comprenez le fonctionnement interne d’Azure, poursuivez votre apprentissage avec la gouvernance des ressources cloud.

> [!div class="nextstepaction"]
> [En savoir plus sur la gouvernance des ressources](what-is-governance.md)

<!-- Links -->

[cosmosdb]: /azure/cosmos-db/introduction
[docs-add-users-to-aad]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[vms]: /azure/virtual-machines/
[vnet]: /azure/virtual-network/virtual-networks-overview
