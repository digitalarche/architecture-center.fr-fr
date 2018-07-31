---
title: 'Adoption d’Azure : phase initiale'
description: Décrit le niveau de base de connaissances que doit posséder une entreprise pour adopter Azure
author: petertay
ms.openlocfilehash: b5a0a4a2c4ed1d06c97774b0eca643a89a5a2110
ms.sourcegitcommit: c704d5d51c8f9bbab26465941ddcf267040a8459
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/24/2018
ms.locfileid: "39229165"
---
# <a name="adopting-microsoft-azure-foundational"></a>Adoption de Microsoft Azure : phase initiale

Pour une organisation qui débute avec les technologies de cloud, il peut être difficile de choisir le bon endroit pour commencer leur processus d’adoption. L’objectif de la phase d’adoption initiale consiste à fournir un point de départ. Une fois que les personnes au sein de l’organisation ont franchi cette étape, ils auront toutes les connaissances et compétences nécessaires pour déployer les ressources de calcul pour une charge de travail simple dans Azure. 

> [!NOTE]
> Ce guide ne couvre pas le développement d’applications. Pour plus d’informations sur le développement d’applications sur Azure, consultez le [Guide d’Architecture des Applications Azure](/azure/architecture/guide/).

Cette étape du guide s’adresse aux membres suivants de votre organisation :

- *Finance :* propriétaire des ressources financières engagées dans Azure, il est responsable du développement des stratégies et procédures dédiées au suivi des coûts de consommation des ressources, y compris la facturation et la refacturation.
- *Informatique centrale :* chargée d’administrer les ressources cloud de votre organisation, y compris la gestion des ressources et l’accès aux ressources, ainsi que l’analyse et le contrôle d’intégrité des charges de travail.
- *Propriétaires de la charge de travail :* tous les rôles de développement qui sont impliqués dans le déploiement des charges de travail dans Azure, notamment les développeurs, testeurs et les ingénieurs de conception.

## <a name="section-1-azure-basics"></a>Section 1 : Principes fondamentaux d’Azure

Cette section d’introduction est destinée aux personnages de la *finance* et de l’*informatique centrale*. L’objectif de cette section est d’acquérir une compréhension élémentaire de [comment fonctionne Azure](azure-explainer.md) en préparation de la découverte du [concept de la gouvernance de cloud](governance-explainer.md). Il peut également être utile pour les *propriétaires de la charge de travail* dans votre organisation pour passer en revue ce contenu et les aider à comprendre comment l’accès aux ressources est géré.

## <a name="section-2-governance-design-guide"></a>Section 2 : Guide de conception de gouvernance

Maintenant que vous comprenez comment fonctionne Azure et les principes de base de la gouvernance de cloud, votre première étape dans l’adoption d’Azure nécessite l’apprentissage de la [gestion des accès aux ressources](azure-resource-access.md) dans Azure. Cet article décrit les services d’Azure pour effectuer des demandes d’accès aux ressources et les contrôles utilisés pour valider ces demandes.

L’étape suivante consiste à apprendre comment [concevoir un modèle de gouvernance](governance-how-to.md) pour une équipe unique. Cet article décrit comment configurer les services de gestion des accès aux ressources et les contrôles que vous avez appris précédemment.

## <a name="section-3-implementing-a-basic-resource-access-management-model"></a>Section 3 : implémentation d’un modèle de gestion des ressources

La dernière étape de votre processus d’adoption consiste à apprendre à implémenter le modèle de gouvernance conçu précédemment. 

Pour commencer, votre organisation nécessite un compte Azure. Si votre organisation dispose déjà d’un [contrat entreprise Microsoft](https://www.microsoft.com/licensing/licensing-programs/enterprise.aspx) qui n’inclut pas Azure, Azure peut être ajouté en effectuant un engagement financier initial. Consultez les [licences Azure pour l’entreprise](https://azure.microsoft.com/pricing/enterprise-agreement/) pour plus d’informations. 

Lors de la création de votre compte Azure, vous devez spécifier une personne de votre organisation comme étant le **propriétaire du compte** Azure. Un client Azure Active Directory (Azure AD) est ensuite créé par défaut. Le **propriétaire du compte** Azure doit [créer le compte d’utilisateur](/azure/active-directory/add-users-azure-active-directory) de la personne **propriétaire de la charge de travail** de votre organisation. 

Ensuite, le **propriétaire du compte** Azure doit [créer un abonnement](https://docs.microsoft.com/partner-center/create-a-new-subscription) et [associer le client Azure AD](/azure/active-directory/fundamentals/active-directory-how-subscriptions-associated-directory) à cet abonnement.

Enfin, maintenant que l’abonnement est créé que le client Azure AD lui est associé, vous pouvez [ajouter le **propriétaire de la charge de travail** à l’abonnement intégré au **rôle de** propriétaire](/azure/billing/billing-add-change-azure-subscription-administrator#add-an-rbac-owner-for-a-subscription-in-azure-portal).

## <a name="section-4-deploy-a-basic-workload-architecture-to-azure"></a>Section 4 : Déployer une architecture de base de la charge de travail dans Azure

Cette section est à destination de la personne *propriétaire de la charge de travail*. Les *propriétaires de la charge de travail* définissent le calcul et le réseau requis pour leurs charges de travail, sélectionnent les ressources adéquates pour répondre à ces exigences et les déployer dans Azure. 

Pour la phase préparatoire à l’adoption, le propriétaire de la charge de travail peut sélectionner une application web de base ou une machine virtuelle (VM) et un réseau virtuel (VNet). Pour plus d’informations sur ces différents types d’options de calcul dans Azure, passez en revue la [vue d’ensemble des options de calcul d’Azure](/azure/architecture/guide/technology-choices/compute-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json).

Quelle que soit l’option de calcul sélectionnée, chacun de ces déploiements requiert un **groupe de ressources**. Le **propriétaire de la charge de travail** doit [créer le groupe de ressources](/azure/azure-resource-manager/vs-azure-tools-resource-groups-deployment-projects-create-deploy). Dans le cadre du déploiement, le **propriétaire de la charge de travail** spécifie un nom pour le groupe de ressources. Ce nom est utilisé dans les sections suivantes.

### <a name="basic-web-application-paas"></a>Application web de base (PaaS)

Pour une application web de base, sélectionnez un des Démarrages rapides de 5 minutes à partir de la [documentation web apps](/azure/app-service?toc=/azure/architecture/cloud-adoption-guide/toc.json) et suivez les étapes. 

> [!NOTE]
> Certaines des Démarrages rapides déploieront un groupe de ressources par défaut. Dans ce cas, le **propriétaire de la charge de travail** n’a pas besoin de créer un groupe de ressources explicitement. Vous pouvez également déployer l’application web pour le groupe de ressources créé ci-dessus.

Une fois que votre charge de travail simple a été déployée, vous pouvez trouver plus d’informations sur les pratiques éprouvées pour le déploiement d’une [application web de base](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json) dans Azure.

### <a name="single-windows-or-linux-vm-iaas"></a>Machine virtuelle Windows ou Linux simple (IaaS)

Pour une charge de travail simple exécutable sur une machine virtuelle, la première étape consiste à déployer un réseau virtuel. Toutes les ressources IaaS dans Azure telles que les machines virtuelles, les équilibreurs de charge et les passerelles nécessitent un réseau virtuel. Une fois que vous en saurez plus sur [les réseaux virtuels Azure](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json), suivez les étapes pour [déployer un réseau virtuel dans Azure à l’aide du portail](/azure/virtual-network/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). Lorsque vous spécifiez les paramètres pour le réseau virtuel dans le portail Azure, spécifiez le nom du groupe de ressources créé ci-dessus.

L’étape suivante consiste à décider s’il faut déployer une machine virtuelle simple Windows ou Linux. [Pour une machine virtuelle Windows,déployer une machine virtuelle Windows sur Azure à l’aide du portail](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). Là encore, lorsque vous spécifiez les paramètres de la machine virtuelle dans le portail Azure, spécifiez le nom du groupe de ressources créé ci-dessus.

Une fois que vous avez suivi les étapes et déployé la machine virtuelle, vous pouvez en savoir plus sur les [pratiques éprouvées pour l’exécution d’une machine virtuelle Windows dans Azure](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json). [Pour une machine virtuelle Linux, déployer une machine virtuelle Linux dans Azure à l’aide du portail](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). Vous pouvez également en apprendre davantage sur les [pratiques éprouvées pour l’exécution d’une machine virtuelle Linux dans Azure](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json).

## <a name="next-steps"></a>Étapes suivantes

L’étape suivante de préparation du cloud est une [**étape intermédiaire**](../intermediate-stage/overview.md). Dans cette étape intermédiaire, vous découvrirez comment étendre votre réseau local en exécutant plusieurs charges de travail et gouvernances des modèles pour plusieurs équipes.