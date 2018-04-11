---
title: Critères de sélection d’une option de calcul Azure
description: Comparaison multidimensionnelle des services de calcul Azure.
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 36b57d1fb674b5a1452a0e8208de836963b2b01b
ms.sourcegitcommit: c53adf50d3a787956fc4ebc951b163a10eeb5d20
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2017
---
# <a name="criteria-for-choosing-an-azure-compute-option"></a>Critères de sélection d’une option de calcul Azure

Le terme *calcul* fait référence au modèle d’hébergement pour les ressources de calcul utilisées par vos applications. Les tableaux suivants effectuent une comparaison multidimensionnelle des différents services de calcul Azure disponibles. Reportez-vous à ces tableaux au moment de choisir une option de calcul pour votre application.

## <a name="hosting-model"></a>Modèle d’hébergement

| Critères | Machines virtuelles | App Service | Service Fabric | Azure Functions | Azure Container Service | Services cloud | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Composition de l’application | Sans dépendance | Applications | Services, exécutables invités, conteneurs | Fonctions | Conteneurs | contrôleur | Scheduled jobs  |
| Densité | Sans dépendance | Plusieurs applications par instance via des plans d’application | Plusieurs services par machine virtuelle | Aucune instance dédiée <a href="#note1"><sup>1</sup></a> | Plusieurs conteneurs par machine virtuelle | Une instance de rôle par machine virtuelle | Plusieurs applications par machine virtuelle |
| Nombre minimal de nœuds | 1 <a href="#note2"><sup>2</sup></a>  | 1 | 5 <a href="#note3"><sup>3</sup></a> | Aucun nœud dédié <a href="#note1"><sup>1</sup></a> | 3 | 2 | 1 <a href="#note4"><sup>4</sup></a> |
| Gestion de l'état | Sans état ou avec état | Sans état | Sans état ou avec état | Sans état | Sans état ou avec état | Sans état | Sans état |
| Hébergement web | Sans dépendance | Intégré | Sans dépendance | Non applicable | Sans dépendance | Intégré (Internet Information Services) | Non |
| SE | Windows, Linux | Windows, Linux  | Windows, Linux | Non applicable | Windows (version préliminaire), Linux | Windows | Windows, Linux |
| Peut être déployé vers le réseau virtuel dédié ? | Pris en charge | Pris en charge <a href="#note5"><sup>5</sup></a> | Pris en charge | Non pris en charge | Pris en charge | Pris en charge <a href="#note6"><sup>6</sup></a> | Pris en charge |
| Connectivité hybride | Pris en charge | Pris en charge <a href="#note1"><sup>7</sup></a>  | Pris en charge | Non pris en charge | Pris en charge | Pris en charge <a href="#note8"><sup>8</sup></a> | Pris en charge |

Remarques

1. <span id="note1">Avec utilisation d’un plan de consommation. Si vous utilisez un plan App Service, les fonctions sont exécutées sur les machines virtuelles allouées dans le cadre de votre plan App Service. Voir [Choisir le plan de service approprié pour Azure Functions][function-plans].</a>
2. <span id="note2">SLA supérieur avec deux instances ou plus.</a>
3. <span id="note3">Pour les environnements de production.</a>
4. <span id="note4">Peut revenir au point d’origine une fois la tâche terminée.</a>
5. <span id="note5">Nécessite un environnement App Service Environment (ASE).</a>
6. <span id="note6">Réseau virtuel classique uniquement.</a>
7. <span id="note7">Nécessite des connexions hybrides ASE ou BizTalk</a>
8. <span id="note8">Réseau virtuel classique ou réseau virtuel du Gestionnaire de ressources via l’homologation de réseaux virtuels</a>

## <a name="devops"></a>DevOps

| Critères | Machines virtuelles | App Service | Service Fabric | Azure Functions | Azure Container Service | Services cloud | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Débogage local | Sans dépendance | IIS Express, autres <a href="#note1b"><sup>1</sup></a> | Cluster de nœuds local | Interface de ligne de commande Azure Functions | Runtime de conteneurs local | Émulateur local | Non pris en charge |
| Modèle de programmation | Sans dépendance | Application web, WebJobs pour les tâches en arrière-plan | Invité exécutable, modèle de service, modèle d’acteur, conteneurs | Fonctions avec déclencheurs | Sans dépendance | Rôle web, rôle de travail | Application de ligne de commande |
| Gestionnaire de ressources | Pris en charge | Pris en charge | Pris en charge | Pris en charge | Pris en charge | Limité <a href="#note2b"><sup>2</sup></a> | Pris en charge |  
| Mise à jour d’application | Aucune prise en charge intégrée | Emplacements de déploiement | Mise à niveau propagée (par service) | Aucune prise en charge intégrée | Dépend de l’orchestrateur. Prise en charge des mises à niveau propagées dans la plupart des cas | Échange d’adresses IP virtuelles ou mise à jour propagée | Non applicable |

Remarques

1. <span id="note1b">Les options incluent IIS Express pour ASP.NET ou node.js (iisnode) ; le serveur web PHP ; le kit de ressources Azure pour IntelliJ et le kit de ressources Azure pour Eclipse. App Service prend également en charge le débogage à distance de l’application web déployée.</a>
2. <span id="note2b">Voir [Fournisseurs, régions, schémas et versions d’API du Gestionnaire des ressources][resource-manager-supported-services]. 


## <a name="scalability"></a>Extensibilité

| Critères | Machines virtuelles | App Service | Service Fabric | Azure Functions | Azure Container Service | Services cloud | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Mise à l’échelle automatique | Groupes de machines virtuelles identiques (VMSS) | Service intégré | Groupes de machines virtuelles identiques (VMSS) | Service intégré | Non pris en charge | Service intégré | N/A |
| Équilibrage de charge | Azure Load Balancer | Intégré | Azure Load Balancer | Intégré | Azure Load Balancer | Intégré | Azure Load Balancer |
| Limite de la mise à l’échelle | Image de plateforme : 1 000 nœuds par VMSS, image personnalisée : 100 nœuds par VMSS | 20 instances, 50 avec App Service Environment | 100 nœuds par VMSS | Infini <a href="#note1c"><sup>1</sup></a> | 100 | Aucune limite définie, 200 nœuds max. (recommandé) | Limite de 20 cœurs par défaut. Contactez le service client pour augmenter la limite. |

Remarques

1. <span id="note1c">Avec utilisation d’un plan de consommation. Si vous utilisez le plan App Service, les limites de mise à l’échelle d’AppService s’appliquent. Voir [Choisir le plan de service approprié pour Azure Functions][function-plans].</a>

## <a name="availability"></a>Availability

| Critères | Machines virtuelles | App Service | Service Fabric | Azure Functions | Azure Container Service | Services cloud | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Contrat SLA | [Contrat SLA pour les machines virtuelles][sla-vm] | [Contrat SLA pour App Service][sla-app-service] | [Contrat SLA pour Service Fabric][sla-sf] | [Contrat SLA pour Azure Functions][sla-functions] | [Contrat SLA pour Azure Container Service][sla-acs] | [Contrat SLA pour Azure Cloud Services][sla-cloud-service] | [Contrat SLA pour Azure Batch][sla-batch] |
| Basculement multirégion | Traffic Manager | Traffic Manager | Traffic Manager, cluster multirégion | Non pris en charge  | Traffic Manager | Traffic Manager | Non pris en charge |

## <a name="security"></a>Sécurité

| Critères | Machines virtuelles | App Service | Service Fabric | Azure Functions | Azure Container Service | Services cloud | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SSL | Configuré au niveau de la machine virtuelle | Pris en charge | Pris en charge  | Pris en charge | Configuré au niveau de la machine virtuelle | Pris en charge | Pris en charge |
| RBAC | Pris en charge | Pris en charge | Pris en charge | Pris en charge | Pris en charge | Non pris en charge | Pris en charge |

## <a name="other"></a>Autres

| Critères | Machines virtuelles | App Service | Service Fabric | Azure Functions | Azure Container Service | Services cloud | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Coût | [Windows][cost-windows-vm], [Linux][cost-linux-vm] | [Tarification d’App Service][cost-app-service] | [Tarification de Service Fabric][cost-service-fabric] | [Tarification d’Azure Functions][cost-functions] | [Tarification d’Azure Container Service][cost-acs] | [Tarification d’Azure Cloud Services][cost-cloud-services] | [Tarification d’Azure Batch][cost-batch]
| Styles d’architecture compatibles | Multiniveau, Big Compute (HPC) | Web-File d’attente-Worker | Microservices, architecture basée sur les événements (EDA) | Microservices, EDA | Microservices, EDA | Web-File d’attente-Worker | Big Compute |

[cost-linux-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/linux/
[cost-windows-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/windows/
[cost-app-service]: https://azure.microsoft.com/pricing/details/app-service/
[cost-service-fabric]: https://azure.microsoft.com/pricing/details/service-fabric/
[cost-functions]: https://azure.microsoft.com/pricing/details/functions/
[cost-acs]: https://azure.microsoft.com/pricing/details/container-service/
[cost-cloud-services]: https://azure.microsoft.com/pricing/details/cloud-services/
[cost-batch]: https://azure.microsoft.com/pricing/details/batch/

[function-plans]: /azure/azure-functions/functions-scale
[sla-acs]: https://azure.microsoft.com/support/legal/sla/container-service/
[sla-app-service]: https://azure.microsoft.com/support/legal/sla/app-service/
[sla-batch]: https://azure.microsoft.com/support/legal/sla/batch/
[sla-cloud-service]: https://azure.microsoft.com/support/legal/sla/cloud-services/
[sla-functions]: https://azure.microsoft.com/support/legal/sla/functions/
[sla-sf]: https://azure.microsoft.com/support/legal/sla/service-fabric/
[sla-vm]: https://azure.microsoft.com/support/legal/sla/virtual-machines/

[resource-manager-supported-services]: /azure/azure-resource-manager/resource-manager-supported-services