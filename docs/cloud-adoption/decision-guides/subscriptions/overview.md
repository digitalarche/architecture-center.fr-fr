---
title: 'Framework d’adoption du cloud : Guide de décision concernant les abonnements'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Découvrez comment utiliser un abonnement de plateforme cloud comme un service principal lors des migrations Azure.
author: rotycenh
ms.openlocfilehash: c0781f6af25150d359395b1b80506dd0cfee8e3c
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900805"
---
# <a name="subscription-decision-guide"></a>Guide de décision concernant les abonnements

Toutes les plateformes cloud sont basées sur un modèle de propriété de base qui fournit de nombreuses options de facturation et de gestion des ressources. La structure qu’utilise Azure est différente de celle des autres fournisseurs cloud, car elle comprend diverses options de prise en charge concernant la hiérarchie organisationnelle et la propriété groupée des abonnements. En général, il y a une personne qui est responsable de la facturation et une autre qui est désignée comme propriétaire principal pour la gestion des ressources.

![Options d’abonnement, de la moins complexe à la plus complexe, dans l’ordre des liens ci-dessous](../../_images/discovery-guides/discovery-guide-subscriptions.png)

Passer à : [Conception d’abonnements et contrats Entreprise Azure](#subscriptions-design-and-azure-enterprise-agreements) | [Modèles de conception d’abonnements](#subscription-design-patterns) | [Groupes d’administration](#management-groups) | [Organisation au niveau de l’abonnement](#organization-at-the-subscription-level)

La conception d’abonnements est l’une des stratégies les plus couramment utilisées par les entreprises pour établir une structure ou organiser les ressources lors de l’adoption du cloud.

**Hiérarchie d’abonnement** : Un *abonnement* est une collection logique de services Azure (comme des machines virtuelles, des bases de données SQL, des services d’applications ou des conteneurs). Dans Azure, chaque ressource est déployée dans un seul abonnement, et cet abonnement n’appartient qu’à un seul *compte*. Ce compte est un compte d’utilisateur (ou, de préférence, un compte de service) qui fournit l’accès aux fonctionnalités de facturation et d’administration dans l’abonnement. Pour les clients qui se sont engagés à utiliser une certaine quantité de stockage Azure par le biais d’un contrat Entreprise (EA), un autre niveau de contrôle appelé *département* est ajouté. Dans le portail EA, vous pouvez utiliser les abonnements, les comptes et les services pour créer une hiérarchie à des fins de facturation et de gestion.

La complexité des abonnements varie selon leur conception. Les décisions relatives à la stratégie de conception ont des points d’inflexion uniques, puisqu’elles prennent en compte généralement les contraintes liées à l’entreprise et à l’informatique. Avant de prendre des décisions d’ordre technique, les architectes informatiques et les preneurs de décisions collaborent avec les parties prenantes et l’équipe chargée des stratégies cloud pour connaître l’approche de gestion des comptes cloud qui est souhaitée, les pratiques de contrôle de gestion de vos unités commerciales ainsi que les besoins de votre organisation concernant le marché international.

**Point d’inflexion** : Dans l’image ci-dessus, la ligne en pointillés représente le point d’inflexion qui existe entre les modèles simples et complexes de conception des abonnements. D’autres décisions techniques peuvent avoir un impact significatif sur la conception des abonnements, comme les décisions basées sur la taille du patrimoine numérique par rapport aux limites d’abonnement Azure, aux stratégies d’isolement et de ségrégation des données, ainsi que par rapport aux divisions opérationnelles informatiques.

**Autres points à considérer** : Une chose importante à noter lorsque vous sélectionnez une conception d’abonnement est que les abonnements ne constituent pas le seul moyen de regrouper des ressources ou des déploiements. Les abonnements ont été créés dans les tout débuts d’Azure, et sont donc limités par certaines solutions Azure plus anciennes comme Azure Service Manager.

La structure des déploiements, l’automatisation et les nouvelles approches relatives au regroupement des ressources peuvent affecter la conception des abonnements de votre structure. Avant de finaliser une conception d’abonnement, réfléchissez à la façon dont la [cohérence des ressources](../resource-consistency/overview.md) peut influencer votre choix. Par exemple, une grande entreprise multinationale peut d’abord envisager un modèle complexe pour sa gestion des abonnements. Toutefois, cette même entreprise pourra obtenir de plus grands bénéfices avec un modèle d’unité commerciale plus simple et une hiérarchie de groupes d’administration.

## <a name="subscriptions-design-and-azure-enterprise-agreements"></a>Création d’abonnements et contrats Entreprise Azure

Tous les abonnements Azure sont associés à un compte, qui est lui-même connecté à la facturation et au contrôle d’accès de niveau supérieur de chaque abonnement. Un même compte peut comprendre plusieurs abonnements et peut fournir des fonctionnalités de base pour l’organisation des abonnements.

Pour les déploiements Azure de petite échelle, votre domaine cloud peut se composer d’un seul abonnement ou d’une petite collection d’abonnements. Toutefois, les déploiements à plus grande échelle Azure peuvent nécessiter de s’étendre sur plusieurs abonnements afin de prendre en charge la structure de votre organisation et de contourner les [limites et quotas relatifs aux abonnements](/azure/azure-subscription-service-limits).

Chaque contrat Entreprise Azure fournit des fonctionnalités supplémentaires permettant d’organiser les abonnements, et est inclus dans les hiérarchies qui reflètent les priorités de votre organisation. Votre Accord de Mise en Œuvre Entreprise définit la forme et l’utilisation des services Azure au sein de votre entreprise d’un point de vue contractuel. Dans chaque contrat Entreprise, vous pouvez subdiviser l’environnement en services, en comptes et en abonnements, pour qu’il corresponde à la structure de votre organisation.

![hiérarchie](../../_images/infra-subscriptions/agreement.png)

## <a name="subscription-design-patterns"></a>Modèles de conception des abonnements

Chaque entreprise est différente. Par conséquent, la hiérarchie de comptes, services et abonnements qui est activée dans l’ensemble du contrat Entreprise Azure permet une grande flexibilité dans la façon dont Azure est organisé. La modélisation de votre hiérarchie selon les besoins de votre entreprise au niveau de la facturation, de la gestion des ressources et de l’accès aux ressources constitue la première et la plus importante des décisions à prendre avant de se lancer dans le cloud public.

Les modèles d’abonnement suivants reflètent une augmentation globale de la complexité de la conception des abonnements, permettant de prendre en charge les priorités potentielles de l’organisation :

### <a name="single-subscription"></a>Abonnement unique

Un seul abonnement par compte peut s’avérer suffisant pour les organisations qui doivent déployer un petit nombre de ressources hébergées dans le cloud. C’est souvent le premier modèle d’abonnement que vous implémentez au début de votre adoption du cloud, car il vous permet d’effectuer des déploiements de petite échelle à des fins d’expérimentation ou de preuve de concept, afin d’explorer les fonctionnalités d’une plateforme cloud.

Toutefois, il peut y avoir des limitations techniques quant au nombre de ressources qu’un même abonnement peut prendre en charge. À mesure que grandit votre domaine cloud, vous pouvez également vouloir prendre en charge l’organisation de vos ressources afin de mieux organiser les stratégies et le contrôle d’accès, par rapport à ce que permet l’utilisation d’un seul abonnement.

### <a name="application-category-pattern"></a>Modèle de catégorie d’application

La nécessité de plusieurs abonnements devient de plus en plus probable à mesure que grandit l’empreinte cloud de votre organisation. Dans ce scénario, les abonnements sont généralement créés pour prendre en charge des applications qui comportent des différences fondamentales concernant la criticité métier, les exigences de conformité, les contrôles d’accès ou les besoins en protection des données. Les abonnements et les comptes qui prennent en charge ces catégories d’application sont tous organisés dans un même département qui est détenu et administré par le personnel informatique.

Chaque organisation classe ses applications différemment, souvent en séparant les abonnements en fonction des applications ou des services, ou en suivant des archétypes d’application. Les charges de travail qui peuvent justifier l’utilisation d’un abonnement séparé dans ce modèle sont les suivantes :

- Applications expérimentales ou à faible risque
- Applications comportant des données protégées
- Charges de travail stratégiques
- Applications soumises à des exigences réglementaires (HIPAA ou FedRAMP)
- Charges de travail par lots
- Charges de travail de Big data comme Hadoop
- Charges de travail conteneurisées utilisant des orchestrateurs de déploiement comme Kubernetes
- Charges de travail d’analytique

Ce modèle prend en charge plusieurs propriétaires de comptes responsables de charges de travail spécifiques. Étant donné qu’il ne dispose pas d’une structure très complexe au niveau Département de la hiérarchie du contrat Entreprise, ce modèle ne nécessite pas l’implémentation d’un contrat Entreprise Azure.

![Modèle de catégorie d’application](../../_images/infra-subscriptions/application.png)

### <a name="functional-pattern"></a>Modèle fonctionnel

Ce modèle organise les abonnements et les comptes selon les fonctions, telles que la finance, les ventes ou le support informatique, en utilisant la hiérarchie Entreprise/Service/Compte/Abonnement fournie aux clients du contrat Entreprise Azure.

![Modèle fonctionnel](../../_images/infra-subscriptions/functional.png)

### <a name="business-unit-pattern"></a>Modèle d’unité commerciale

Ce modèle regroupe les abonnements et les comptes en fonction de la catégorie Pertes et profits, de l’unité commerciale, de la division, du centre de profit ou d’une structure métier similaire, à l’aide de la hiérarchie du contrat Entreprise Azure.

![Modèle d’unité commerciale](../../_images/infra-subscriptions/business.png)

### <a name="geographic-pattern"></a>Modèle géographique

Pour les organisations internationales, ce modèle regroupe les abonnements et les comptes en fonction des zones géographiques en utilisant la hiérarchie du contrat Entreprise Azure.

![Modèle géographique](../../_images/infra-subscriptions/geographic.png)

### <a name="mixed-patterns"></a>Modèles mixtes

Hiérarchie Entreprise/Service/Compte/Abonnement. Toutefois, vous pouvez combiner des modèles, comme le modèle géographique et le modèle d’unité commerciale, pour répondre aux structures plus complexes de facturation et d’organisation de votre entreprise. De plus, votre [conception de la cohérence des ressources](../resource-consistency/overview.md) peut étendre davantage la gouvernance et la structure organisationnelle de votre conception d’abonnement.

Les groupes d’administration, comme nous allons le voir dans la section suivante, peuvent aider à la prise en charge de structures organisationnelles plus complexes.

Les groupes d’administration, qui sont abordés dans la section suivante, peuvent aider à la prise en charge de structures organisationnelles plus complexes.

## <a name="management-groups"></a>Groupes d’administration

En plus de la structure du département et de l’organisation qui est fournie via les contrats Entreprise, les [groupes d’administration Azure](/azure/governance/management-groups/index) offrent davantage de flexibilité pour organiser les stratégies, le contrôle d’accès et la conformité dans plusieurs abonnements. Les groupes d’administration prennent en charge jusqu’à six niveaux d’imbrication, ce qui vous permet de créer une hiérarchie séparée de celle de la facturation. Cela peut être utilisé uniquement pour une gestion efficace des ressources.

Les groupes d’administration permettent de reproduire une hiérarchie de facturation et c’est ce par quoi les entreprises commencent souvent. Cependant, la puissance des groupes d’administration se manifeste quand ils sont utilisés pour modéliser l’organisation dans laquelle des abonnements associés, qu’ils se trouvent ou non dans la hiérarchie de facturation, sont regroupés avec la nécessité de leur attribuer des rôles communs ainsi que des stratégies et des initiatives.

Voici quelques exemples :

- Production/Hors production : Certaines entreprises créent des groupes d’administration pour identifier leurs abonnements production et hors production. Les groupes d’administration permettent à ces clients de gérer plus facilement les rôles et les stratégies. Par exemple, si un abonnement hors production peut octroyer aux développeurs un accès « contributeur », en production, ils ne bénéficie que d’un accès « lecteur ».
- Services internes/Services externes : À l’instar des abonnements de production/hors production, les entreprises ont souvent des exigences, des stratégies et des rôles différents pour les services internes et les services externes (destinés aux clients).

## <a name="organization-at-the-subscription-level"></a>Organisation au niveau de l’abonnement

Quand vous êtes amené à prendre des décisions concernant vos départements et vos comptes (ou groupes d’administration), votre objectif premier est de diviser votre environnement Azure à l’image de votre organisation. Or, vos décisions les plus importantes sont celles qui ont trait aux abonnements, car elles ont un impact sur la sécurité, la scalabilité et la facturation.

Aidez-vous des modèles suivants :

- **Application/Service** : Les abonnements représentent une application ou un service (portefeuille d’applications).

- **Cycle de vie** : Les abonnements représentent le cycle de vie d’un service, par exemple Production ou Développement.

- **Service** : Les abonnements représentent les services de l’organisation.

Les deux premiers modèles sont les plus couramment utilisés et sont fortement recommandés. L’approche Cycle de vie convient pour la plupart des organisations. Dans ce cas, la recommandation générale consiste à utiliser les deux abonnements de base (Production et Hors production), puis à utiliser les groupes de ressources permettant de fragmenter encore davantage les environnements.

Pour obtenir une description générale de la façon dont les abonnements et les ressources Azure sont utilisés pour regrouper et gérer les ressources, consultez [Gestion des accès aux ressources dans Azure](../../getting-started/azure-resource-access.md).

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment les services d’identité sont utilisés pour le contrôle d’accès et la gestion dans le cloud.

> [!div class="nextstepaction"]
> [Identité](../identity/overview.md)
