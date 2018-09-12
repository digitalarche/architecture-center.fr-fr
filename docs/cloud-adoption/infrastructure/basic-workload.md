---
title: 'Adoption du cloud d’entreprise : Déployer une charge de travail de base'
description: Décrit comment déployer une charge de travail de base sur Azure
author: petertaylor9999
ms.openlocfilehash: d6dd8e107b4b9e48697c4c7bc4d32018eb79de0b
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/31/2018
ms.locfileid: "43327050"
---
# <a name="enterprise-cloud-adoption-deploy-a-basic-workload"></a>Adoption du cloud d’entreprise : Déployer une charge de travail de base

Le terme **charge de travail** définit généralement une unité arbitraire de fonctionnalité, par exemple une application ou un service. Une charge de travail est envisagée sous l’angle des artefacts de code qui sont déployés sur un serveur, mais également sous l’angle de tous les autres services nécessaires. Cette définition se révèle pertinente dans le cas d’une application ou d’un service local(e), mais dans le cloud, elle mérite d’être développée.

Dans le cloud, une charge de travail englobe non seulement tous les artefacts, mais également les ressources cloud. Les ressources cloud sont ici incluses dans notre définition en raison d’un concept appelé infrastructure en tant que code. Comme vous l’avez appris dans l’article [Fonctionnement d’Azure](../getting-started/what-is-azure.md), les ressources dans Azure sont déployées par un service d’orchestrateur. Le service d’orchestrateur expose cette fonctionnalité via une API web, laquelle peut être appelée à l’aide de plusieurs outils, tels que Powershell, l’interface de ligne de commande (CLI) d’Azure et le portail Azure. Cela signifie que nous pouvons spécifier nos ressources dans un fichier lisible sur ordinateur qui peut être stocké en même temps que les artefacts de code associés à notre application.

Nous pouvons ainsi définir une charge de travail en termes d’artefacts de code et de ressources cloud nécessaires, ce qui nous permet d’isoler nos charges de travail. Les charges de travail peuvent être isolées par la façon dont les ressources sont organisées, par la topologie du réseau ou par d’autres attributs. L’isolement des charges de travail a pour but d’associer les ressources spécifiques d’une charge de travail à une équipe donnée, afin que cette équipe puisse gérer indépendamment tous les aspects de ces ressources. Cela permet à plusieurs équipes de partager des services de gestion de ressources dans Azure tout en évitant une suppression ou une modification involontaire des ressources d’une autre équipe.

Cet isolement permet également de favoriser ce que l’on appelle le DevOps. Le DevOps inclut les pratiques de développement logiciel couvrant le développement logiciel et les opérations informatiques décrits ci-dessus, mais en y ajoutant autant que possible le recours à l’automatisation. Le principe d’intégration continue/livraison continue (CI/CD) est au cœur du DevOps. L’intégration continue fait référence aux processus de génération automatisés qui sont exécutés à chaque fois qu’un développeur valide une modification de code. La livraison continue renvoie à des processus automatisés qui déploient ce code vers différents environnements, par exemple un environnement de développement à des fins de test ou un environnement de production pour le déploiement final.

## <a name="basic-workload"></a>Charge de travail de base

Une **charge de travail de base** est généralement définie comme une application web unique, ou un réseau virtuel avec une machine virtuelle. 

> [!NOTE]
> Ce guide ne couvre pas le développement d’applications. Pour plus d’informations sur le développement d’applications sur Azure, consultez le [Guide d’Architecture des Applications Azure](/azure/architecture/guide/).

Que la charge de travail soit une application web ou une machine virtuelle, chacun de ces déploiements nécessite un **groupe de ressources**. Un utilisateur autorisé à créer un groupe de ressources doit le faire avant de suivre les étapes ci-dessous.

## <a name="basic-web-application-paas"></a>Application web de base (PaaS)

Pour une application web de base, sélectionnez un des Démarrages rapides de 5 minutes à partir de la [documentation web apps](/azure/app-service?toc=/azure/architecture/cloud-adoption-guide/toc.json) et suivez les étapes. 

> [!NOTE]
> Certaines des Démarrages rapides déploieront un groupe de ressources par défaut. Dans ce cas, il n’est pas nécessaire de créer un groupe de ressources de façon explicite. Vous pouvez également déployer l’application web pour le groupe de ressources créé ci-dessus.

Une fois que votre charge de travail simple a été déployée, vous pouvez trouver plus d’informations sur les pratiques éprouvées pour le déploiement d’une [application web de base](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json) dans Azure.

## <a name="single-windows-or-linux-vm-iaas"></a>Machine virtuelle Windows ou Linux simple (IaaS)

Pour une charge de travail simple exécutable sur une machine virtuelle, la première étape consiste à déployer un réseau virtuel. Toutes les ressources IaaS dans Azure telles que les machines virtuelles, les équilibreurs de charge et les passerelles nécessitent un réseau virtuel. Une fois que vous en saurez plus sur [les réseaux virtuels Azure](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json), suivez les étapes pour [déployer un réseau virtuel dans Azure à l’aide du portail](/azure/virtual-network/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). Lorsque vous spécifiez les paramètres pour le réseau virtuel dans le portail Azure, spécifiez le nom du groupe de ressources créé ci-dessus.

L’étape suivante consiste à décider s’il faut déployer une machine virtuelle simple Windows ou Linux. [Pour une machine virtuelle Windows,déployer une machine virtuelle Windows sur Azure à l’aide du portail](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). Là encore, lorsque vous spécifiez les paramètres de la machine virtuelle dans le portail Azure, spécifiez le nom du groupe de ressources créé ci-dessus.

Une fois que vous avez suivi les étapes et déployé la machine virtuelle, vous pouvez en savoir plus sur les [pratiques éprouvées pour l’exécution d’une machine virtuelle Windows dans Azure](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json). [Pour une machine virtuelle Linux, déployer une machine virtuelle Linux dans Azure à l’aide du portail](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). Vous pouvez également en apprendre davantage sur les [pratiques éprouvées pour l’exécution d’une machine virtuelle Linux dans Azure](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json).