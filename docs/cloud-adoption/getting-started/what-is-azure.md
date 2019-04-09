---
title: 'Framework d’adoption du cloud : Fonctionnement d’Azure'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
description: Explication sur le fonctionnement interne d’Azure
author: petertaylor9999
ms.date: 02/11/2019
ms.openlocfilehash: 215e4e4954a2f3e722e3ac865fea19508f6edadd
ms.sourcegitcommit: 0a8a60d782facc294f7f78ec0e9033e3ee16bf4a
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/08/2019
ms.locfileid: "59068818"
---
<!-- markdownlint-disable MD026 -->

# <a name="how-does-azure-work"></a>Fonctionnement d’Azure

Azure est la plateforme de cloud public de Microsoft. Azure offre une grande collection de services, y compris la plateforme en tant que service (PaaS), infrastructure en tant que service (IaaS), la base de données comme une fonctionnalités du service (DBaaS). Mais ce qu’est -ce qu’Azure exactement et comment fonctionne-t-il ?

<!-- markdownlint-disable MD034 -->

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2ixGo]

<!-- markdownlint-enable MD034 -->

Azure, comme les autres plateformes de cloud, s’appuie sur une technologie appelée **virtualisation**. Le matériel informatique peut être, en général, émulé dans le logiciel, car il s’agit simplement d’un ensemble d’instructions encodées de façon permanente ou semi-permanente en silicium. À l’aide d’une couche d’émulation qui mappe les instructions du logiciel pour obtenir des instructions de matériel, le matériel virtualisé peut s’exécuter dans le logiciel comme s’il s’agissait du matériel réel lui-même.

Fondamentalement, le cloud est un ensemble de serveurs physiques dans un ou plusieurs centres de données qui exécutent un matériel virtualisé pour le compte de clients. Ainsi, comment le cloud crée, démarre, arrête et supprime-t-il simultanément des millions d’instances d’un matériel virtualisé pour des millions de clients ?

Pour le comprendre, examinons l’architecture du matériel dans le centre de données.  À l’intérieur de chaque centre de données est une collection de serveurs dans les racks de serveurs. Chaque rack de serveurs contient de nombreuses **lames** de serveurs ainsi qu’un commutateur de réseau fournissant une connectivité réseau et une unité de distribution de l’alimentation (PDU) fournissant l’électricité. Les racks sont parfois regroupés en unités plus grandes appelées **clusters**.

Dans chaque rack ou cluster, la plupart des serveurs sont désignés pour exécuter ces instances du matériel virtualisé au nom de l’utilisateur. Toutefois, certains serveurs exécutent des logiciels de gestion du cloud connu comme un contrôleur de structure. Le **contrôleur de structure** est une application distribuée avec beaucoup de responsabilités. Il alloue des services, analyse l’intégrité du serveur et les services qui y sont exécutés et répare des serveurs en cas d’échec.

Chaque instance du contrôleur de structure est connectée à un autre ensemble de serveurs exécutant le logiciel d’orchestration du cloud, généralement appelé **frontal**. Le serveur frontal héberge les services web, les API RESTful et les bases de données Azure internes utilisés pour toutes les fonctions exécutées par le cloud.

Par exemple, le serveur frontal héberge les services qui gèrent les demandes de clients pour allouer des ressources Azure telles que [réseaux virtuels](/azure/virtual-network/virtual-networks-overview), [machines virtuelles](/azure/virtual-machines)et des services, comme [Cosmos DB](/azure/cosmos-db/introduction). Tout d’abord, le serveur frontal valide l’utilisateur et vérifie que l’utilisateur est autorisé à allouer les ressources demandées. Dans ce cas, le serveur frontal vérifie une base de données pour localiser un rack de serveurs avec une capacité suffisante et indique ensuite le contrôleur de structure sur ce rack alloue la ressource.

Par conséquent, fondamentalement, Azure est une vaste collection de serveurs et de matériel réseau en cours d’exécution un jeu complexe d’applications distribuées pour orchestrer la configuration et le fonctionnement du matériel virtualisé et de logiciels sur ces serveurs. Cette orchestration qui font d’Azure si puissant &mdash; les utilisateurs ne sont plus responsables de la gestion et la mise à niveau de matériel, car Azure fait tout cela en arrière-plan.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous comprenez les éléments internes d’Azure, découvrez la gouvernance des ressources de cloud.

> [!div class="nextstepaction"]
> [En savoir plus sur la gouvernance des ressources](what-is-governance.md)

<!-- Links -->

[docs-add-users-to-aad]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
