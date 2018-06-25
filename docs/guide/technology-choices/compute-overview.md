---
title: Vue d’ensemble des options de calcul Azure
description: Vue d’ensemble des options de calcul Azure
author: MikeWasson
ms.date: 06/13/2018
ms.openlocfilehash: ceb70f8eeff42e6cadb8a63c2f36986f26322201
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206556"
---
# <a name="overview-of-azure-compute-options"></a>Vue d’ensemble des options de calcul Azure

Le terme *calcul* fait référence au modèle d’hébergement des ressources de calcul utilisées par votre application. 

## <a name="overview"></a>Vue d'ensemble

À l’une des extrémités du spectre se trouve la fonctionnalité **infrastructure as a service** (IaaS). IaaS vous permet d’approvisionner les machines virtuelles dont vous avez besoin, ainsi que les composants de stockage et réseau associés. Ensuite, vous déployez les logiciels et les applications dont vous souhaitez disposer sur ces machines virtuelles. Ce modèle est le plus proche d’un environnement local traditionnel, à ceci près que l’infrastructure est gérée par Microsoft. Vous continuez de gérer les différentes machines virtuelles.  

La fonctionnalité **platform as a Service** (PaaS) offre un environnement d’hébergement géré, dans lequel vous pouvez déployer votre application sans avoir à gérer de machines virtuelles ni de ressources réseau. Par exemple, plutôt que de créer des machines virtuelles individuelles, vous spécifiez un nombre d’instances, et le service approvisionne, configure et gère les ressources nécessaires. Azure App Service constitue un exemple de service PaaS.

Il existe toute une gamme de services entre IaaS et le modèle PaaS pur. Par exemple, les machines virtuelles Azure peuvent faire l’objet d’une mise à l’échelle automatique à l’aide de VM Scale Sets. Cette fonctionnalité de mise à l’échelle automatique n’est pas strictement PaaS, mais ce type de fonctionnalité de gestion peut figurer dans un service PaaS.

La fonctionnalité **Functions as a Service** (FaaS) se révèle encore plus complète en vous évitant d’avoir à vous soucier de l’environnement d’hébergement. Au lieu de créer des instances de calcul et de déployer du code pour ces instances, vous vous contentez de déployer votre code, et le service l’exécute alors automatiquement. Vous n’avez pas besoin d’administrer les ressources de calcul. Ces services utilisent une architecture sans serveur et montent ou diminuent en puissance en toute transparence jusqu’au niveau requis pour gérer le trafic. Azure Functions est un exemple de service FaaS.

La fonctionnalité IaaS offre le meilleur niveau de contrôle, de flexibilité et de portabilité. FaaS est un gage de simplicité, de mise à l’échelle élastique et d’économies substantielles, car vous payez uniquement pour le temps pendant lequel votre code s’exécute. La fonctionnalité PaaS se situe quelque part entre les deux. En général, plus un service offre de flexibilité, plus la responsabilité de la configuration et de la gestion des ressources vous revient. Les services FaaS gèrent automatiquement la quasi-totalité des aspects de l’exécution d’une application, alors que les solutions IaaS vous contraignent à approvisionner, configurer et gérer les machines virtuelles et les composants réseau que vous créez.

## <a name="azure-compute-options"></a>Options de calcul Azure

Les principales options de calcul actuellement disponibles dans Azure sont les suivantes :

- [Machines Virtuelles](/azure/virtual-machines/) est un service IaaS qui vous permet de déployer et gérer des machines virtuelles à l’intérieur d’un réseau virtuel (VNet).
- [App Service](/azure/app-service/app-service-value-prop-what-is) est une offre PaaS managée pour l’hébergement d’applications web, de serveurs principaux d’applications mobiles, d’API RESTful ou de processus d’entreprise automatisés.
- [Service Fabric](/azure/service-fabric/service-fabric-overview) est une plateforme de systèmes distribués qui peut s’exécuter dans de nombreux environnements, y compris sur Azure ou localement. Service Fabric est un orchestrateur de microservices sur un cluster de machines. 
- [Azure Container Service](/azure/container-service/container-service-intro) vous permet de créer, configurer et gérer un cluster de machines virtuelles préconfigurées pour l’exécution d’applications en conteneur.
- [Azure Container Instances](/azure/container-instances/container-instances-overview) propose la façon la plus simple et rapide qui soit d’exécuter un conteneur dans Azure, sans avoir à configurer des machines virtuelles ni adopter un service de niveau supérieur.
- [Azure Functions](/azure/azure-functions/functions-overview) est un service FaaS géré.
- [Azure Batch](/azure/batch/batch-technical-overview) est un service géré qui permet d’exécuter des applications de calcul haute performance (HPC) et parallèles à grande échelle.
- [Cloud Services](/azure/cloud-services/cloud-services-choose-me) est un service géré permettant d’exécuter des applications cloud. Il utilise un modèle d’hébergement PaaS. 

Lorsque vous sélectionnez une option de calcul, voici quelques facteurs à prendre en compte :

- Modèle d’hébergement. Comment le service est-il hébergé ? Quelles sont les exigences et limitations imposées par cet environnement d’hébergement ? 
- DevOps. Existe-t-il une prise en charge intégrée des mises à niveau d’application ? Quel est le modèle de déploiement utilisé ?
- Extensibilité. Comment le service gère-t-il l’ajout ou la suppression d’instances ? Peut-il faire l’objet d’une mise à l’échelle automatique basée sur la charge et sur d’autres métriques ? 
- Disponibilité. Quel est le Contrat de niveau de service (SLA) du service ? 
- Coût. Outre le coût du service proprement dit, considérez le coût des opérations de gestion d’une solution reposant sur ce service. Par exemple, les solutions IaaS peuvent augmenter le coût des opérations.
- Quelles sont les limitations globales de chaque service ? 
- Quels sont les types d’architectures d’application les mieux adaptés à ce service ? 

## <a name="next-steps"></a>Étapes suivantes

Pour vous aider à sélectionner un service de calcul pour votre application, utilisez l’[Arbre de décision des services de calcul Azure](./compute-decision-tree.md).

Pour découvrir une comparaison détaillée des options de calcul dans Azure, consultez l’article [Criteria for choosing an Azure compute service (Critères de choix d’un service de calcul Azure)](./compute-comparison.md).
