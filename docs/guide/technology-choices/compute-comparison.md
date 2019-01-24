---
title: Critères de sélection d’un service de calcul Azure
titleSuffix: Azure Application Architecture Guide
description: Comparaison multidimensionnelle des services de calcul Azure.
author: MikeWasson
ms.date: 08/08/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: 2b6b9b941bf7a3c0136b71ecb65bfe4b4a59e07b
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54484854"
---
# <a name="criteria-for-choosing-an-azure-compute-service"></a>Critères de sélection d’un service de calcul Azure

Le terme *calcul* fait référence au modèle d’hébergement pour les ressources de calcul utilisées par vos applications. Les tableaux suivants effectuent une comparaison multidimensionnelle des différents services de calcul Azure disponibles. Reportez-vous à ces tableaux au moment de choisir une option de calcul pour votre application.

## <a name="hosting-model"></a>Modèle d’hébergement

<!-- markdownlint-disable MD033 -->

| Critères | Virtual Machines | App Service | Service Fabric | Azure Functions | Azure Kubernetes Service | Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Composition de l’application | Sans dépendance | Applications, conteneurs | Services, exécutables invités, conteneurs | Fonctions | Containers | Containers | Scheduled jobs  |
| Densité | Sans dépendance | Plusieurs applications par instance via des plans App Service | Plusieurs services par machine virtuelle | Serverless <a href="#note1"><sup>1</sup></a> | Plusieurs conteneurs par nœud |Aucune instance dédiée | Plusieurs applications par machine virtuelle |
| Nombre minimal de nœuds | 1 <a href="#note2"><sup>2</sup></a>  | 1 | 5 <a href="#note3"><sup>3</sup></a> | Serverless <a href="#note1"><sup>1</sup></a> | 3 <a href="#note3"><sup>3</sup></a> | Aucun nœud dédié | 1 <a href="#note4"><sup>4</sup></a> |
| Gestion de l'état | Sans état ou avec état | Sans état | Sans état ou avec état | Sans état | Sans état ou avec état | Sans état | Sans état |
| Hébergement web | Sans dépendance | Intégré | Sans dépendance | Non applicable | Sans dépendance | Sans dépendance | Non  |
| Peut être déployé vers le réseau virtuel dédié ? | Pris en charge | Pris en charge<a href="#note5"><sup>5</sup></a> | Pris en charge | Pris en charge <a href="#note5"><sup>5</sup></a> | [Pris en charge](/azure/aks/networking-overview) | Non pris en charge | Pris en charge |
| Connectivité hybride | Pris en charge | Pris en charge <a href="#note6"><sup>6</sup></a>  | Pris en charge | Pris en charge <a href="#node7"><sup>7</sup></a> | Pris en charge | Non pris en charge | Pris en charge |

Notes

1. <span id="note1">Avec utilisation d’un plan de consommation. Si vous utilisez un plan App Service, les fonctions sont exécutées sur les machines virtuelles allouées dans le cadre de votre plan App Service. Voir [Choisir le plan de service approprié pour Azure Functions][function-plans].</span>
2. <span id="note2">SLA supérieur avec deux instances ou plus.</span>
3. <span id="note3">Recommandé pour les environnements de production.</span>
4. <span id="note4">Peut revenir au point d’origine une fois la tâche terminée.</span>
5. <span id="note5">Nécessite un environnement App Service Environment (ASE).</span>
6. <span id="note6">Utiliser [les connexions hybrides d’Azure App Service][app-service-hybrid].</span>
7. <span id="note7">Nécessite un plan App Service.</span>

## <a name="devops"></a>DevOps

| Critères | Virtual Machines | App Service | Service Fabric | Azure Functions | Azure Kubernetes Service | Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Débogage local | Sans dépendance | IIS Express, autres <a href="#note1b"><sup>1</sup></a> | Cluster de nœuds local | CLI Visual Studio ou Azure Functions | Minikube, autres | Runtime de conteneurs local | Non pris en charge |
| Modèle de programmation | Sans dépendance | Applications web et API, WebJobs pour les tâches en arrière-plan | Invité exécutable, modèle de service, modèle d’acteur, conteneurs | Fonctions avec déclencheurs | Sans dépendance | Sans dépendance | Application de ligne de commande |
| Mise à jour d’application | Aucune prise en charge intégrée | Emplacements de déploiement | Mise à niveau propagée (par service) | Emplacements de déploiement | Mise à jour propagée | Non applicable |

Notes

1. <span id="note1b">Les options incluent IIS Express pour ASP.NET ou node.js (iisnode) ; le serveur web PHP ; le kit de ressources Azure pour IntelliJ et le kit de ressources Azure pour Eclipse. App Service prend également en charge le débogage à distance de l’application web déployée.</span>
2. <span id="note2b">Voir [Fournisseurs, régions, schémas et versions d’API du Gestionnaire des ressources][resource-manager-supported-services].</span>

## <a name="scalability"></a>Extensibilité

| Critères | Virtual Machines | App Service | Service Fabric | Azure Functions | Azure Kubernetes Service | Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Mise à l’échelle automatique | Groupes de machines virtuelles identiques (VMSS) | Service intégré | Groupes de machines virtuelles identiques (VMSS) | Service intégré | Non pris en charge | Non pris en charge | N/A |
| Équilibrage de charge | Azure Load Balancer | Intégré | Azure Load Balancer | Intégré | Intégré |  Aucune prise en charge intégrée | Azure Load Balancer |
| Limite de mise à l’échelle<a href="#note1c"><sup>1</sup></a> | Image de plateforme : 1 000 nœuds par groupe de machines virtuelles identiques, image personnalisée : 100 nœuds par VMSS | 20 instances, 100 avec App Service Environment | 100 nœuds par VMSS | 200 instances par application de fonction | 100 nœuds par cluster (limite par défaut) |20 groupes de conteneurs par abonnement (limite par défaut). | Limite de 20 cœurs (limite par défaut). |

Notes

1. <span id="note1c">Consultez l’article [Abonnement Azure et limites, quotas et contraintes de service](/azure/azure-subscription-service-limits)</span>.

## <a name="availability"></a>Disponibilité

| Critères | Virtual Machines | App Service | Service Fabric | Azure Functions | Azure Kubernetes Service | Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Contrat SLA | [Contrat SLA pour les machines virtuelles][sla-vm] | [Contrat SLA pour App Service][sla-app-service] | [Contrat SLA pour Service Fabric][sla-sf] | [Contrat SLA pour Azure Functions][sla-functions] | [SLA pour AKS][sla-acs] | [Contrat SLA pour Container Instances](https://azure.microsoft.com/support/legal/sla/container-instances/) | [Contrat SLA pour Azure Batch][sla-batch] |
| Basculement multirégion | Traffic Manager | Traffic Manager | Traffic Manager, cluster multirégion | Non pris en charge | Traffic Manager | Non pris en charge | Non pris en charge |

## <a name="other"></a>Autres

| Critères | Virtual Machines | App Service | Service Fabric | Azure Functions | Azure Kubernetes Service | Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SSL | Configuré au niveau de la machine virtuelle | Pris en charge | Pris en charge  | Pris en charge | [Contrôleur d’entrée](/azure/aks/ingress) | Utilisez le conteneur [side-car](../../patterns/sidecar.md) | Pris en charge |
| Coût | [Windows][cost-windows-vm], [Linux][cost-linux-vm] | [Tarification d’App Service][cost-app-service] | [Tarification de Service Fabric][cost-service-fabric] | [Tarification d’Azure Functions][cost-functions] | [Tarification AKS][cost-acs] | [Tarification Container Instances](https://azure.microsoft.com/pricing/details/container-instances/) | [Tarification d’Azure Batch][cost-batch]
| Styles d’architecture compatibles | [Multiniveau][n-tier], [Big Compute][big-compute] (HPC) | [Web-File d’attente-Worker][w-q-w], [Multiniveau][n-tier] | [Microservices][microservices], [architecture basée sur les événements][event-driven] | [Microservices][microservices], [architecture basée sur les événements][event-driven] | [Microservices][microservices], [architecture basée sur les événements][event-driven] | [Microservices][microservices], automatisation des tâches, programmes de traitement par lots  | [Big Compute][big-compute] (HPC) |

<!-- markdownlint-enable MD033 -->

[cost-linux-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/linux/
[cost-windows-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/windows/
[cost-app-service]: https://azure.microsoft.com/pricing/details/app-service/
[cost-service-fabric]: https://azure.microsoft.com/pricing/details/service-fabric/
[cost-functions]: https://azure.microsoft.com/pricing/details/functions/
[cost-acs]: https://azure.microsoft.com/pricing/details/kubernetes-service/
[cost-batch]: https://azure.microsoft.com/pricing/details/batch/

[function-plans]: /azure/azure-functions/functions-scale
[sla-acs]: https://azure.microsoft.com/support/legal/sla/kubernetes-service
[sla-app-service]: https://azure.microsoft.com/support/legal/sla/app-service/
[sla-batch]: https://azure.microsoft.com/support/legal/sla/batch/
[sla-functions]: https://azure.microsoft.com/support/legal/sla/functions/
[sla-sf]: https://azure.microsoft.com/support/legal/sla/service-fabric/
[sla-vm]: https://azure.microsoft.com/support/legal/sla/virtual-machines/

[resource-manager-supported-services]: /azure/azure-resource-manager/resource-manager-supported-services
[scale-acs]: /azure/container-service/kubernetes/container-service-scale#scaling-considerations

[n-tier]: ../architecture-styles/n-tier.md
[w-q-w]: ../architecture-styles/web-queue-worker.md
[microservices]: ../architecture-styles/microservices.md
[event-driven]: ../architecture-styles/event-driven.md
[big-date]: ../architecture-styles/big-data.md
[big-compute]: ../architecture-styles/big-compute.md

[app-service-hybrid]: /azure/app-service/app-service-hybrid-connections
