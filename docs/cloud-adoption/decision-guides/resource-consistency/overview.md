---
title: 'Framework d’adoption du cloud : Guide de décision pour la cohérence des ressources'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Apprenez-en davantage sur la cohérence des ressources lors de la planification d’une migration Azure.
author: rotycenh
ms.openlocfilehash: 3159e4b7aeddfdd99261c0f68591998d741f3359
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640042"
---
# <a name="caf-resource-consistency-decision-guide"></a>Framework d’adoption du cloud : Guide de décision pour la cohérence des ressources

Azure [conception d’un abonnement](../subscriptions/overview.md) définit la manière dont vous organisez vos ressources cloud par rapport à la structure de votre organisation, des pratiques de gestion des comptes et des besoins de charge de travail. En plus de ce niveau de structure, l’adressage de vos exigences de stratégie de gouvernance d’organisation entre vos biens immobiliers cloud nécessite la possibilité de manière cohérente organiser, déployer et gérer des ressources au sein d’un abonnement.

![Traçage des options d’application de la stratégie de la moins complexe à la plus complexe, dans l’ordre des liens ci-dessous](../../_images/discovery-guides/discovery-guide-resource-consistency.png)

Passer à : [Regroupement de base](#basic-grouping) | [Cohérence de déploiement](#deployment-consistency) | [Cohérence de stratégie](#policy-consistency) | [Cohérence hiérarchique](#hierarchical-consistency) | [Cohérence automatisée](#automated-consistency)

Décisions concernant le niveau d’exigences de cohérence des ressources de votre espace de cloud sont essentiellement motivées par ces facteurs : taille de biens immobiliers numérique après la migration, entreprise ou exigences environnementales qui ne correspondent pas parfaitement dans votre abonnement approches de conception, ni d’assurer la gouvernance au fil du temps une fois que les ressources ont été déployés. 

Ces facteurs gagne en importance, les avantages d’assurer un déploiement cohérent, regroupement et la gestion des ressources de cloud devient importante. Atteindre des niveaux de cohérence des ressources pour répondre aux besoins croissants nécessite plus de temps passé dans automation, outils et l’application de la cohérence, et il en résulte une plus de temps passé sur la gestion des modifications et de suivi.

## <a name="basic-grouping"></a>Regroupement de base

Dans Azure, les [groupes de ressources](/azure/azure-resource-manager/resource-group-overview#resource-groups) sont un mécanisme d’organisation des ressources de base permettant de regrouper logiquement les ressources dans un abonnement.

Les groupes de ressources servent de conteneurs pour les ressources ayant un cycle de vie commun ou des contraintes de gestion partagées telles que les exigences en matière de stratégie ou de contrôle d’accès en fonction du rôle (RBAC). Les groupes de ressources ne peuvent pas être imbriqués et les ressources ne peuvent appartenir qu’à un seul groupe de ressources. Certaines actions peuvent agir sur toutes les ressources contenues dans un groupe de ressources. Par exemple, la suppression d’un groupe de ressources supprime toutes les ressources dans ce groupe. Il existe des modèles communs lors de la création de groupes de ressources, généralement répartis en deux catégories :

- Charges de travail informatiques traditionnelles : Elles sont le plus souvent regroupés par éléments au sein d’un même cycle de vie, comme une application. Le regroupement par application permet de gestion des applications individuelles.
- Charges de travail informatiques de type agile : Elles se concentrent sur les applications cloud pour les clients externes. Ces groupes de ressources reflètent souvent les couches fonctionnelles de déploiement (comme la couche web ou la couche application) et de gestion.

## <a name="deployment-consistency"></a>Cohérence du déploiement

Bâtis sur le mécanisme de regroupement des ressources de base, la plateforme Azure fournit un système à l’aide de modèles pour déployer vos ressources à l’environnement de cloud. Vous pouvez utiliser des modèles pour créer une organisation et des conventions de nommage cohérentes lors du déploiement des charges de travail, en appliquant ces aspects de la conception du déploiement et de la gestion de vos ressources.

Les [modèles Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview#template-deployment) vous permettent de déployer vos ressources de façon répétée et cohérente en utilisant une configuration prédéterminée et une structure de groupes de ressources. Les modèles Resource Manager vous aident à définir un ensemble de standards qui serviront de base à vos déploiements.

Par exemple, vous pouvez avoir un modèle standard pour le déploiement d’une charge de travail du serveur web qui contient les deux machines virtuelles en tant que serveurs web associés à un équilibreur de charge pour répartir le trafic entre les serveurs. Vous pouvez réutiliser ce modèle pour créer un ensemble structurellement identique de machines virtuelles et un équilibreur de charge chaque fois que ce type de charge de travail est nécessaire, modifier uniquement le nom du déploiement et des adresses IP impliquées.

Notez que vous pouvez également déployer ces modèles à l’aide d’un programme et les intégrer à vos systèmes de CI/CD.

## <a name="policy-consistency"></a>Stratégie de cohérence

Pour s’assurer que les stratégies de gouvernance sont appliquées lors de la création des ressources, une partie de la conception du regroupement des ressources implique l’utilisation d’une configuration commune lors du déploiement des ressources.

En combinant des groupes de ressources et des modèles Resource Manager standardisés, vous pouvez appliquer des standards pour les paramètres requis dans un déploiement et les règles [Azure Policy](/azure/governance/policy/overview) appliquées à chaque ressource ou groupe de ressources.

Par exemple, l’une de vos exigences est que toutes les machines virtuelles déployées dans votre abonnement doivent se connecter à un sous-réseau commun géré par votre équipe informatique centrale. Vous pouvez créer un modèle standard pour le déploiement des machines virtuelles de charge de travail qui créerait un groupe de ressources distinct pour la charge de travail et y déploierait les machines virtuelles requises. Ce groupe de ressources disposerait d’une règle de stratégie permettant uniquement de joindre les interfaces réseau du groupe de ressources au sous-réseau partagé.

La section [Application de stratégie](../policy-enforcement/overview.md) fournit une discussion approfondie sur la mise en œuvre de vos décisions stratégiques dans le cadre d’un déploiement cloud.

## <a name="hierarchical-consistency"></a>Cohérence hiérarchique

Groupes de ressources permet de prendre en charge des niveaux supplémentaires de la hiérarchie au sein de votre organisation au sein de l’abonnement, en appliquant des règles de stratégie de Azure et accéder à des contrôles au niveau du groupe de ressources. Toutefois, comme la taille de votre activité principale cloud augmente, vous devrez peut-être prendre en charge plus complexe entre abonnements gouvernance que peut être pris en charge à l’aide de la hiérarchie d’entreprise/service/compte/abonnement de l’accord entreprise Azure. 

[Groupes d’administration Azure](../subscriptions/overview.md#management-groups) vous permet d’abonnements d’organisation dans les structures organisationnelles plus complexes par regroupement des abonnements dans une autre hiérarchie à celle établie par la structure de l’accord de votre entreprise. Cette hiérarchie autre vous permet d’appliquer des mécanismes de mise en œuvre contrôle et la stratégie d’accès entre plusieurs abonnements et les ressources qu’ils contiennent. Les hiérarchies de groupes de gestion peuvent servir à correspondent à des abonnements de votre espace cloud avec les opérations ou les exigences de gouvernance d’entreprise. 

## <a name="automated-consistency"></a>Cohérence automatisée

Pour les grands déploiements cloud, la gouvernance mondiale devient à la fois plus importante et plus complexe. Il est essentiel d’appliquer et de faire respecter automatiquement les exigences de gouvernance lors du déploiement des ressources, ainsi que de répondre aux exigences mises à jour pour les déploiements existants.

[Azure Blueprints](/azure/governance/blueprints/overview) donnent les moyens aux organisations de supporter la gouvernance globale des vastes domaines cloud dans Azure. Les blueprints vont au-delà des fonctionnalités fournies par les modèles standard Azure Resource Manager pour créer des orchestrations de déploiement complètes capables de déployer des ressources et d’appliquer des règles de stratégie. Les blueprints prennent en charge le contrôle de versions et permettent d’appliquer des mises à jour à tous les abonnements dans lesquels le blueprint a été utilisé. Ils permettent également de bloquer les abonnements déployés pour éviter la création et la modification non autorisées des ressources.

Ces packages de déploiement permettent aux équipes informatiques et de développement de déployer rapidement de nouvelles charges de travail et des ressources réseau conformes à l’évolution des exigences des stratégies de l’entreprise. Les blueprints peuvent également être intégrés dans les pipelines de CI/CD afin d’appliquer les normes de gouvernance révisées aux déploiements au fur et à mesure de leur mise à jour.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment le nommage et le balisage des ressources sont utilisés pour mieux organiser et gérer vos ressources cloud.

> [!div class="nextstepaction"]
> [Nommage et marquage des ressources](../resource-tagging/overview.md)
