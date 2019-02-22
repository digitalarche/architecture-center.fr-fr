---
title: 'Framework d’adoption du cloud : Grandes entreprises - Évolution multicloud'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Grandes entreprises - Évolution multicloud
author: BrianBlanchard
ms.openlocfilehash: 5ef29aa523c04ff93b2d4f983482f94654a4a039
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900989"
---
# <a name="large-enterprise-multi-cloud-evolution"></a>Grandes entreprises : Évolution multicloud

## <a name="evolution-of-the-narrative"></a>Évolution du scénario

Microsoft reconnaît que les clients adoptent plusieurs clouds pour des raisons bien spécifiques. C’est également le cas de l’entreprise fictive dont nous allons parler dans cet article. En parallèle du parcours d’adoption d’Azure, la réussite de l’entreprise l’a amenée à acquérir une autre entreprise, petite mais complémentaire. Cette entreprise exécute toutes ses opérations informatiques sur un fournisseur de cloud différent.

Cet article décrit les changements observés suite à l’intégration de la nouvelle organisation. Pour ce scénario, nous partons du principe que cette entreprise a atteint toutes les évolutions de gouvernance décrites dans ce parcours client.

### <a name="evolution-of-the-current-state"></a>Évolution de l’état actuel

Dans la phase précédente de ce scénario, l’entreprise avait commencé à implémenter des contrôles de coûts et des fonctionnalités de surveillance des coûts, étant donné que les dépenses cloud commençaient à faire partie des dépenses opérationnelles régulières de l’entreprise.

Depuis, certains changements ont eu lieu et ceux-ci vont avoir un impact sur la gouvernance :

- L’identité est contrôlée par une instance locale d’Active Directory. L’identité hybride est facilitée grâce à la réplication vers Azure Active Directory.
- Les opérations informatiques ou opérations cloud sont principalement gérées par Azure Monitor et les autres automatisations associées.
- La continuité d’activité et la reprise d’activité sont contrôlées par des instances Azure Vault.
- Azure Security Center est utilisé pour surveiller les attaques et les violations de sécurité.
- Azure Security Center et Azure Monitor sont utilisés pour surveiller la gouvernance du cloud.
- Azure Blueprints, Azure Policy et des groupes d’administration sont utilisés pour automatiser la conformité à la stratégie.

### <a name="evolution-of-the-future-state"></a>Évolution de l’état futur

L’objectif est d’intégrer l’entreprise ayant fait l’objet de l’acquisition aux opérations existantes, dans la mesure du possible.

## <a name="evolution-of-tangible-risks"></a>Évolution des risques tangibles

**Coût d’acquisition de l’entreprise** : d’après les prévisions, l’acquisition de la nouvelle entreprise sera rentabilisée dans cinq ans environ. En raison de la durée de rentabilisation relativement longue, le conseil d’administration souhaite contrôler autant que possible les coûts d’acquisition. Il existe un risque de conflit entre le contrôle des coûts et l’intégration technique.

Ce risque métier peut être associé à quelques risques techniques

- La migration cloud risque de générer des coûts d’acquisition supplémentaires.
- En outre, le nouvel environnement risque de ne pas être correctement gouverné ou de provoquer des violations de stratégie.

## <a name="evolution-of-the-policy-statements"></a>Évolution des instructions de stratégie

Les modifications suivantes apportées à la stratégie permettront de réduire les nouveaux risques et de guider l’implémentation.

1. Toutes les ressources dans un cloud secondaire doivent être surveillées à l’aide des outils existants de surveillance de la sécurité et de gestion opérationnelle.
2. Toutes les unités d’organisation doivent être intégrées dans le fournisseur d’identité existant.
3. Le fournisseur d’identité principal doit régir l’authentification des ressources dans le cloud secondaire.

## <a name="evolution-of-the-best-practices"></a>Évolution des meilleures pratiques

Cette section de l’article va faire évoluer la conception du produit minimum viable pour la gouvernance afin d’inclure de nouvelles stratégies Azure, ainsi qu’une implémentation d’Azure Cost Management. Ensemble, ces deux modifications de la conception permettront de répondre aux nouvelles instructions de la stratégie d’entreprise.

1. Connectez les réseaux. Cette tâche est exécutée par les équipes responsables de la sécurité informatique et de la mise en réseau, et prise en charge par la gouvernance
    1. L’ajout d’une connexion d’un fournisseur MPLS/de ligne allouée au nouveau cloud intégrera les réseaux. L’ajout de configurations de pare-feu et de tables de routage contrôlera l’accès et le trafic entre les environnements.
2. Consolidez les fournisseurs d’identité. En fonction des charges de travail hébergées dans le cloud secondaire, diverses options sont proposées pour consolider le fournisseur d’identité. Voici quelques exemples :
    1. Pour les applications qui utilisent l’authentification OAuth2, les utilisateurs Active Directory dans le cloud secondaire peuvent simplement être répliqués vers le locataire Azure AD existant.
    2. De l’autre côté, la fédération entre les deux fournisseurs d’identité locaux permettrait aux utilisateurs des nouveaux domaines Active Directory d’être répliqués sur Azure.
3. Ajoutez des ressources à Azure Site Recovery
    1. Dès le départ, Azure Site Recovery a été conçu en tant qu’outil hybride/multicloud.
    2. Les machines virtuelles dans le cloud secondaire peuvent être protégées par les mêmes processus Azure Site Recovery utilisés pour protéger les ressources locales.
4. Ajoutez des ressources à Azure Cost Management
    1. Dès le départ, Azure Cost Management a été conçu en tant qu’outil multicloud.
    2. Les machines virtuelles dans le cloud secondaire peuvent être compatibles avec Azure Cost Management pour certains fournisseurs cloud. Des frais supplémentaires peuvent s’appliquer.
5. Ajoutez des ressources à Azure Monitor
    1. Dès le départ, Azure Monitor a été conçu en tant qu’outil cloud hybride.
    2. Les machines virtuelles dans le cloud secondaire peuvent être compatibles avec les agents Azure Monitor, ce qui leur permet d’être incluses dans Azure Monitor pour la surveillance opérationnelle.
6. Outils de mise en œuvre de la gouvernance
    1. La mise en œuvre de la gouvernance est spécifique au cloud.
    2. À l’inverse, les stratégies d’entreprise établies dans le parcours de gouvernance ne le sont pas. Bien que l’implémentation puisse varier d’un cloud à l’autre, les instructions de stratégie peuvent être appliquées au fournisseur secondaire.

Au fur et à mesure du développement de l’adoption du cloud, l’évolution de la conception ci-dessus continuera de mûrir.

## <a name="next-steps"></a>Étapes suivantes

Dans de nombreuses grandes entreprises, les disciplines de la gouvernance cloud peuvent bloquer l’adoption. L’article suivant présente des réflexions sur comment transformer la gouvernance en sport collectif, pour assurer la réussite à long terme dans le cloud.

> [!div class="nextstepaction"]
> [Couches de gouvernance multiples](./multiple-layers-of-governance.md)
