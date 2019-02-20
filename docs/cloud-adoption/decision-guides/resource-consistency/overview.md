---
title: 'Framework d’adoption du cloud : Guide de décision pour la cohérence des ressources'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Apprenez-en davantage sur la cohérence des ressources lors de la planification d’une migration Azure.
author: rotycenh
ms.openlocfilehash: 8170bfd09218a451e086a57e0631b7e567eb2b82
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900865"
---
# <a name="caf-resource-consistency-decision-guide"></a>Framework d’adoption du cloud : Guide de décision pour la cohérence des ressources

La [conception d’abonnement](../subscriptions/overview.md) Azure définit la façon dont vous organisez vos ressources cloud par rapport à la structure globale de votre organisation. De plus, l’intégration de vos normes de gestion informatique existantes et de vos stratégies organisationnelles dépend de la façon dont vous déployez et organisez les ressources cloud au sein d’un abonnement.

Les outils disponibles pour mettre en œuvre le déploiement, le regroupement et la gestion de vos ressources varient selon la plateforme cloud utilisée. En général, chaque solution comprend les fonctionnalités suivantes :

- Mécanisme de regroupement logique sous le niveau de l’abonnement ou du compte.
- Possibilité de déployer les ressources par programmation avec des API.
- Modèles permettant de créer des déploiements standardisés.
- Possibilité de déployer des règles de stratégie au niveau de l’abonnement, du compte et du regroupement des ressources.

![Traçage des options d’application de la stratégie de la moins complexe à la plus complexe, dans l’ordre des liens ci-dessous](../../_images/discovery-guides/discovery-guide-resource-consistency.png)

Passer à : [Regroupement de base](#basic-grouping) | [Cohérence de déploiement](#deployment-consistency) | [Cohérence de stratégie](#policy-consistency) | [Cohérence hiérarchique](#hierarchical-consistency) | [Cohérence automatisée](#automated-consistency)

Les décisions de déploiement et de regroupement des ressources sont principalement motivées par les facteurs suivants : la taille du parc numérique après la migration, la complexité métier ou environnementale qui ne s’intègre pas parfaitement dans vos approches de conception d’abonnement existantes, ou la nécessité de renforcer la gouvernance au fil du temps après le déploiement des ressources. Les conceptions de regroupement des ressources plus avancées exigent un effort accru pour assurer un regroupement précis, ce qui se traduit par une augmentation du temps consacré à la gestion et au suivi des changements.

## <a name="basic-grouping"></a>Regroupement de base

Dans Azure, les [groupes de ressources](/azure/azure-resource-manager/resource-group-overview#resource-groups) sont un mécanisme d’organisation des ressources de base permettant de regrouper logiquement les ressources dans un abonnement.

Les groupes de ressources servent de conteneurs pour les ressources ayant un cycle de vie commun ou des contraintes de gestion partagées telles que les exigences en matière de stratégie ou de contrôle d’accès en fonction du rôle (RBAC). Les groupes de ressources ne peuvent pas être imbriqués et les ressources ne peuvent appartenir qu’à un seul groupe de ressources. Certaines actions peuvent agir sur toutes les ressources contenues dans un groupe de ressources. Par exemple, la suppression d’un groupe de ressources supprime toutes les ressources dans ce groupe. Il existe des modèles communs lors de la création de groupes de ressources, généralement répartis en deux catégories :

- Charges de travail informatiques traditionnelles : Elles sont le plus souvent regroupés par éléments au sein d’un même cycle de vie, comme une application. Le regroupement par application permet de gestion des applications individuelles.
- Charges de travail informatiques de type agile : Elles se concentrent sur les applications cloud pour les clients externes. Ces groupes de ressources reflètent souvent les couches fonctionnelles de déploiement (comme la couche web ou la couche application) et de gestion.

## <a name="deployment-consistency"></a>Cohérence du déploiement

S’appuyant sur le mécanisme de regroupement des ressources de base, la plupart des plateformes cloud offrent un système permettant d’utiliser des modèles de déploiement de vos ressources dans l’environnement cloud. Vous pouvez utiliser des modèles pour créer une organisation et des conventions de nommage cohérentes lors du déploiement des charges de travail, en appliquant ces aspects de la conception du déploiement et de la gestion de vos ressources.

Les [modèles Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview#template-deployment) vous permettent de déployer vos ressources de façon répétée et cohérente en utilisant une configuration prédéterminée et une structure de groupes de ressources. Les modèles Resource Manager vous aident à définir un ensemble de standards qui serviront de base à vos déploiements.

Par exemple, vous pouvez disposer d’un modèle standard pour déployer une charge de travail de serveur web qui contient deux machines virtuelles en tant que serveurs web combinés avec un équilibreur de charge gérant le trafic entre les serveurs. Vous pouvez ensuite réutiliser ce modèle pour créer des déploiements structurellement identiques chaque fois qu’une nouvelle charge de travail de serveur web est nécessaire, en ne modifiant que le nom du déploiement et les adresses IP concernées.

Notez que vous pouvez également déployer ces modèles à l’aide d’un programme et les intégrer à vos systèmes de CI/CD.

## <a name="policy-consistency"></a>Stratégie de cohérence

Pour s’assurer que les stratégies de gouvernance sont appliquées lors de la création des ressources, une partie de la conception du regroupement des ressources implique l’utilisation d’une configuration commune lors du déploiement des ressources.

En combinant des groupes de ressources et des modèles Resource Manager standardisés, vous pouvez appliquer des standards pour les paramètres requis dans un déploiement et les règles [Azure Policy](/azure/governance/policy/overview) appliquées à chaque ressource ou groupe de ressources.

Par exemple, l’une de vos exigences est que toutes les machines virtuelles déployées dans votre abonnement doivent se connecter à un sous-réseau commun géré par votre équipe informatique centrale. Vous pouvez créer un modèle standard pour le déploiement des machines virtuelles de charge de travail qui créerait un groupe de ressources distinct pour la charge de travail et y déploierait les machines virtuelles requises. Ce groupe de ressources disposerait d’une règle de stratégie permettant uniquement de joindre les interfaces réseau du groupe de ressources au sous-réseau partagé.

La section [Application de stratégie](../policy-enforcement/overview.md) fournit une discussion approfondie sur la mise en œuvre de vos décisions stratégiques dans le cadre d’un déploiement cloud.

## <a name="hierarchical-consistency"></a>Cohérence hiérarchique

Au fur et à mesure que la taille de votre investissement cloud augmente, vous aurez peut-être des exigences de gouvernance plus complexes que celles qui peuvent être prises en charge par la hiérarchie Entreprise/Service/Compte /Abonnement de l’accord Entreprise Azure. Les groupes de ressources vous permettent de prendre en charge des niveaux hiérarchiques supplémentaires au sein de votre organisation, en appliquant les règles Azure Policy et les contrôles d’accès au niveau d’un groupe de ressources.

Les [groupes d’administration Azure](../subscriptions/overview.md#management-groups) peuvent prendre en charge des structures organisationnelles plus complexes en superposant une hiérarchie alternative à la structure de votre accord entreprise. Cela permet aux abonnements et aux ressources qu’ils contiennent de prendre en charge les mécanismes de contrôle d’accès et d’implémentation des stratégies organisés en fonction des exigences organisationnelles de votre entreprise.

## <a name="automated-consistency"></a>Cohérence automatisée

Pour les grands déploiements cloud, la gouvernance mondiale devient à la fois plus importante et plus complexe. Il est essentiel d’appliquer et de faire respecter automatiquement les exigences de gouvernance lors du déploiement des ressources, ainsi que de répondre aux exigences mises à jour pour les déploiements existants.

[Azure Blueprints](/azure/governance/blueprints/overview) donnent les moyens aux organisations de supporter la gouvernance globale des vastes domaines cloud dans Azure. Les blueprints vont au-delà des fonctionnalités fournies par les modèles standard Azure Resource Manager pour créer des orchestrations de déploiement complètes capables de déployer des ressources et d’appliquer des règles de stratégie. Les blueprints prennent en charge le contrôle de versions et permettent d’appliquer des mises à jour à tous les abonnements dans lesquels le blueprint a été utilisé. Ils permettent également de bloquer les abonnements déployés pour éviter la création et la modification non autorisées des ressources.

Ces packages de déploiement permettent aux équipes informatiques et de développement de déployer rapidement de nouvelles charges de travail et des ressources réseau conformes à l’évolution des exigences des stratégies de l’entreprise. Les blueprints peuvent également être intégrés dans les pipelines de CI/CD afin d’appliquer les normes de gouvernance révisées aux déploiements au fur et à mesure de leur mise à jour.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment le nommage et le balisage des ressources sont utilisés pour mieux organiser et gérer vos ressources cloud.

> [!div class="nextstepaction"]
> [Nommage et marquage des ressources](../resource-tagging/overview.md)
