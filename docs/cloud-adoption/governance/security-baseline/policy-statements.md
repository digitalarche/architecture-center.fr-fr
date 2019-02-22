---
title: 'Framework d’adoption du cloud : Exemples d’instructions de stratégie de base de référence de la sécurité'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Exemples d’instructions de stratégie de base de référence de la sécurité
author: BrianBlanchard
ms.openlocfilehash: ac40e022f8dff0c3021c04cd9d6ae42d28afaf1f
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900799"
---
# <a name="security-baseline-sample-policy-statements"></a>Exemples d’instructions de stratégie de base de référence de la sécurité

Les instructions de stratégie cloud individuelles sont des recommandations pour la gestion de risques spécifiques identifiés lors de votre processus d’évaluation des risques. Ces instructions doivent fournir un bref récapitulatif des risques ainsi que des plans pour les gérer. Chaque définition d’instruction doit inclure les informations suivantes :

- Risque technique : un récapitulatif du risque que cette stratégie gérera.
- Instruction de stratégie : une explication succincte et claire des exigences de la stratégie.
- Options techniques : recommandations exploitables, spécifications ou autres conseils que des équipes informatiques et des développeurs peuvent utiliser lors de l’implémentation de la stratégie.

Les exemples d’instructions de stratégie suivants gèrent un certain nombre des risques métier courants liés à la sécurité. Ils sont fournis à titre d’exemples que vous pouvez consulter lors de l’élaboration d’instructions de stratégie réelles répondant aux besoins de votre propre organisation. Ces exemples ne sont pas destinés à être normatifs, et il existe potentiellement plusieurs options de stratégie pour la gestion de tout risque identifié unique. Travaillez en étroite collaboration avec les équipes commerciales, de sécurité et informatiques pour identifier les meilleures solutions de stratégie pour vos risques de sécurité particuliers.  

## <a name="asset-classification"></a>Classification des ressources

**Risque technique** : Les ressources qui ne sont pas correctement identifiées comme critiques ou qui impliquent des données sensibles peuvent ne pas se voir accorder une protection suffisante, ce qui conduit à des fuites de données ou des interruptions potentielles de l’activité.

**Instruction de stratégie** : Toutes les ressources déployées doivent être classées par criticité et par classification des données. Les classifications doivent être examinées par l’équipe de gouvernance cloud et le propriétaire de l’application avant le déploiement dans le cloud.

**Option de conception potentielle** : Établissez des [normes de balisage des ressources](../../decision-guides/resource-tagging/overview.md) et vérifiez que le service informatique les applique systématiquement à toutes les ressources déployées à l’aide de [balises de ressources Azure](/azure/azure-resource-manager/resource-group-using-tags).

## <a name="data-encryption"></a>Chiffrement des données

**Risque technique** : Il existe un risque d’exposition de données protégées pendant le stockage.

**Instruction de stratégie** : Toutes les données protégées doivent être chiffrées au repos.

**Option de conception potentielle** : Consultez l’article [Vue d’ensemble du chiffrement Azure](/azure/security/security-azure-encryption-overview) pour voir comment le chiffrement des données au repos est effectué sur la plateforme Azure.  

## <a name="network-isolation"></a>Isolement réseau

**Risque technique** : La connectivité entre des réseaux et des sous-réseaux de réseaux introduit des vulnérabilités potentielles qui peuvent entraîner des fuites de données ou une interruption de services critiques.

**Instruction de stratégie** : Les sous-réseaux de réseau qui contiennent des données protégées doivent être isolés de tous les autres sous-réseaux. Le trafic réseau entre des sous-réseaux de données protégées doit être audité régulièrement.

**Option de conception potentielle** : Dans Azure, l’isolement réseau et de sous-réseau est géré par le biais d’[Azure Virtual Networks](/azure/virtual-network/virtual-networks-overview).

## <a name="secure-external-access"></a>Sécuriser l’accès externe

**Risque technique** : Autoriser l’accès à des charges de travail à partir de l’Internet public introduit un risque d’intrusion entraînant une exposition non autorisée des données ou une interruption de l’activité.

**Instruction de stratégie** : Aucun sous-réseau contenant des données protégées ne doit pas être accessible directement par l’Internet public ou dans les centres de données. L’accès à ces sous-réseaux routé via des sous-réseaux intermédiaires fonctionne. Tous les accès à ces sous-réseaux doivent transiter par une solution de pare-feu capable d’effectuer des analyses des paquets et de bloquer des fonctions.

**Option de conception potentielle** : Dans Azure, sécurisez les points de terminaison publics en déployant une [zone DMZ entre l’Internet public et votre réseau basé sur le cloud](/azure/architecture/reference-architectures/dmz/secure-vnet-dmz).

## <a name="ddos-protection"></a>Protection DDoS

**Risque technique** : Les attaques par déni de service distribué (DDoS) peuvent entraîner une interruption de l’activité.

**Instruction de stratégie** : Déployez des mécanismes d’atténuation DDoS automatisés pour tous les points de terminaison réseau accessibles publiquement.

**Option de conception potentielle** : Utilisez [Azure DDoS Protection](/azure/virtual-network/ddos-protection-overview) pour limiter les perturbations provoquées par les attaques DDoS.

## <a name="secure-on-premises-connectivity"></a>Sécuriser la connectivité locale

**Risque technique** : Le trafic non chiffré entre votre réseau cloud et local sur l’Internet public est vulnérable aux interceptions, ce qui introduit un risque d’exposition des données.

**Instruction de stratégie** : Toutes les connexions entre les réseaux locaux et cloud doivent s’effectuer via une connexion VPN chiffrée sécurisée ou une liaison WAN privée dédiée.

**Option de conception potentielle** : Dans Azure, utilisez un réseau VPN ExpressRoute ou Azure pour établir des connexions privées entre vos réseaux locaux et cloud.

## <a name="network-monitoring-and-enforcement"></a>Supervision et mise en œuvre du réseau

**Risque technique** : Les changements apportés à la configuration du réseau peuvent entraîner de nouvelles vulnérabilités et de nouveaux risques d’exposition des données.

**Instruction de stratégie** : Les outils de gouvernance doivent auditer et appliquer les exigences de configuration réseau définies par l’équipe de base de référence de la sécurité.

**Option de conception potentielle** : Dans Azure, l’activité réseau peut être supervisée à l’aide d’[Azure Network Watcher](/azure/network-watcher/network-watcher-monitoring-overview), tandis qu’[Azure Security Center](/azure/security-center/security-center-network-recommendations) peut permettre d’identifier les failles de sécurité. Azure Policy vous permet de restreindre les ressources réseau et la stratégie de configuration des ressources en fonction des limites définies par l’équipe de sécurité.

## <a name="security-review"></a>Révision de sécurité

**Risque technique** : Au fil du temps, de nouvelles menaces de sécurité et de nouveaux types d’attaques émergent, augmentant le risque d’exposition ou d’interruption de vos ressources cloud.

**Instruction de stratégie** : Les tendances et les attaques potentielles susceptibles d’affecter les déploiements cloud doivent être régulièrement examinées par l’équipe de sécurité pour fournir des mises à jour aux outils de base de référence de sécurité utilisés dans le cloud.

**Option de conception potentielle** : Établissez une réunion de révision de sécurité régulière qui inclut les membres de l’équipe informatique et de gouvernance concernés. Passez en revue les données et métriques de sécurité existantes pour déterminer les lacunes des outils de stratégie et de base de référence de sécurité actuels, ainsi que pour mettre à jour la stratégie afin d’atténuer tous les nouveaux risques.

## <a name="next-steps"></a>Étapes suivantes

Utilisez les exemples mentionnés dans cet article comme point de départ pour développer des stratégies qui gèrent des risques de sécurité spécifiques, dans la continuité de vos plans d’adoption du cloud.

Pour commencer à développer vos propres instructions de stratégie personnalisées liées à la base de référence de la sécurité, téléchargez le [modèle de base de référence de la sécurité](template.md).

Pour accélérer l’adoption de cette discipline, choisissez le [parcours de gouvernance exploitable](../journeys/overview.md) correspondant le mieux à votre environnement. Modifiez ensuite la conception pour intégrer vos décisions spécifiques en matière de stratégie d’entreprise.

> [!div class="nextstepaction"]
> [Parcours de gouvernance exploitables](../journeys/overview.md)
