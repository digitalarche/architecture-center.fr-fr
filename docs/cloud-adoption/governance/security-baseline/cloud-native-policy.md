---
title: "Framework d'adoption du cloud : Stratégie Base de référence de la sécurité Native cloud"
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Stratégie Base de référence de la sécurité Native cloud
author: BrianBlanchard
ms.openlocfilehash: fc161009f6ce7aa2b775fe6b3b53ff1e1d62a848
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58242430"
---
# <a name="cloud-native-security-baseline-policy"></a>Stratégie Base de référence de la sécurité Native cloud

La discipline [Base de référence de la sécurité](overview.md) est l'une des [cinq disciplines de la gouvernance cloud](../governance-disciplines.md). Elle se concentre sur les sujets généraux liés à la sécurité, comme la protection du réseau, les ressources numériques, les données, etc. Comme indiqué dans le [Guide de révision des stratégies](../policy-compliance/what-is-a-cloud-policy-review.md), pour chacune des cinq disciplines, le Framework d'adoption du cloud comprend trois niveaux d'**exemples de stratégies** : Native cloud, Entreprise et Conformité au principe de conception cloud. Cet article traite de l'exemple de stratégie Native cloud pour la discipline Base de référence de la sécurité.

> [!NOTE]
> Microsoft n'est pas en position d'imposer une stratégie d'entreprise ou une stratégie informatique. Cet article a pour but de vous aider à vous préparer à une révision de stratégie interne. Il est supposé que cet exemple de stratégie sera étendu, validé et testé sur votre stratégie d'entreprise avant toute tentative d'utilisation. Il est déconseillé d'utiliser cet exemple de stratégie en l'état.

## <a name="policy-alignment"></a>Alignement des stratégies

Cet exemple de stratégie repose sur un scénario Native cloud, ce qui signifie que les outils et plateformes fournis par Azure sont suffisants pour atténuer les risques métier liés au déploiement. Dans ce scénario, il est supposé qu'une configuration simple des services Azure par défaut fournit une protection suffisante des ressources.

## <a name="cloud-security-and-compliance"></a>Conformité et sécurité du cloud

La sécurité est intégrée à tous les aspects d'Azure pour offrir des avantages uniques basés sur des renseignements de sécurité disponibles à l'échelle mondiale, des contrôles orientés client sophistiqués et une infrastructure de sécurité renforcée. Cette combinaison puissante permet de protéger vos applications et vos données, prend en charge vos efforts de conformité, et offre une sécurité rentable pour les entreprises de toutes tailles. Quelle que soit la stratégie de sécurité, cette approche permet de démarrer sur des bases solides, mais elle ne doit pas vous empêcher d'adopter des pratiques rigoureuses vis-à-vis des services de sécurité utilisés.

### <a name="built-in-security-controls"></a>Contrôles de sécurité intégrés

La robustesse de l'infrastructure de sécurité est difficile à maintenir lorsque les contrôles de sécurité ne sont pas intuitifs et doivent être configurés séparément. Avec Azure, les contrôles de sécurité intégrés aux services vous aident à protéger rapidement les données et les charges de travail, et à gérer les risques dans les environnements hybrides. Les solutions partenaires intégrées vous permettent également de migrer facilement les protections existantes vers le cloud.

### <a name="cloud-native-identity-policies"></a>Stratégies d'identité de type Native cloud

L'identité joue désormais un rôle majeur pour la sécurité, au niveau du réseau. Les périmètres réseau sont devenus de plus en plus poreux, et cette défense de périmètre ne peut pas être aussi efficace qu'elle l'était avant l'évolution des applications BYOD et cloud. La gestion des identités et le contrôle d'accès Azure permettent un accès transparent et sécurisé à toutes vos applications.

Un exemple de stratégie Native cloud portant sur l'identité dans les annuaires cloud et locaux peut inclure des exigences semblables aux suivantes :

* Accès autorisé aux ressources avec contrôle d'accès en fonction du rôle (RBAC), authentification multifacteur (MFA) et authentification unique (SSO)
* Atténuation rapide des risques liés aux identités utilisateur soupçonnées d'être compromises
* Accès juste-à-temps (JIT) et « just-enough » accordé tâche par tâche pour limiter l'exposition des informations d'identification des administrateurs dotés de privilèges excessifs
* Identité utilisateur étendue et accès aux stratégies de plusieurs environnements via Azure Active Directory

S'il est important d'appréhender la [Base de référence des identités](../identity-baseline/overview.md) dans le contexte de la Base de référence de la sécurité, les [cinq disciplines de la gouvernance cloud](../overview.md) considèrent la [Base de référence des identités](../identity-baseline/overview.md) comme leur propre discipline, indépendamment de la Base de référence de la sécurité.

### <a name="network-access-policies"></a>Stratégies d'accès réseau

Le contrôle réseau comprend la configuration, la gestion et la sécurisation d'éléments réseau tels que le réseau virtuel, l'équilibrage de charge, le DNS et les passerelles. Les contrôles permettent aux services de communiquer et d'interagir. Azure inclut une infrastructure réseau robuste et sécurisée pour prendre en charge les exigences de connectivité de vos applications et services. La connectivité réseau est possible entre les ressources hébergées dans Azure, entre les ressources hébergées sur site et dans Azure, mais aussi vers et à partir d’Internet et d’Azure.

Une stratégie Native cloud destinée aux contrôles réseau peut inclure des exigences semblables aux suivantes :

* Les connexions hybrides aux ressources locales (bien que techniquement possibles dans Azure) peuvent ne pas être autorisées dans le cadre d'une stratégie Native cloud. Si une connexion hybride s'avère nécessaire, l'exemple de stratégie de sécurité Entreprise, plus robuste, constituera une référence plus pertinente.
* Les utilisateurs peuvent établir des connexions sécurisées vers et au sein d'Azure à l'aide de réseaux virtuels et de groupes de sécurité réseau.
* Le Pare-feu Windows Azure natif protège les hôtes du trafic réseau malveillant par un accès limité aux ports. La nécessité de bloquer (ou de ne pas activer) le trafic direct vers une machine virtuelle via RDP, port TCP/UDP 3389, serait un bon exemple de cette stratégie.
* Des services tels que le pare-feu d'applications web et Azure DDoS Protection protègent les applications et garantissent la disponibilité des machines virtuelles exécutées dans Azure. Ces fonctionnalités ne doivent pas être désactivées ou mal utilisées.

### <a name="data-protection"></a>Protection des données

Pour assurer la protection des données dans le cloud, l'un des facteurs clés consiste à tenir compte de l'état des données, mais aussi des contrôles disponibles pour chaque état. Dans le cadre des meilleures pratiques de chiffrement et de sécurité des données Azure, les recommandations portent sur les états suivants des données :

* Des contrôles de chiffrement des données sont intégrés aux services, des machines virtuelles jusqu'au stockage et à SQL Database.
* Lorsque les données se déplacent entre les clouds et entre les clients, leur protection peut être assurée par des protocoles de chiffrement standard.
* Azure Key Vault permet aux utilisateurs d'assurer la sécurité ainsi que le contrôle des clés de chiffrement et des secrets utilisés par les applications et services cloud.
* Azure Information Protection vous aidera à classer, étiqueter et protéger vos données sensibles dans les applications.

Bien que les fonctionnalités mentionnées ci-dessus soient intégrées à Azure, chacune d'entre elles nécessite une configuration qui peut entraîner une hausse des coûts. Il est vivement conseillé d'associer chaque fonctionnalité Native cloud à une [stratégie de classification des données](../policy-compliance/what-is-data-classification.md).

### <a name="security-monitoring"></a>Surveillance de la sécurité

Ici, la surveillance de la sécurité est une stratégie proactive qui audite vos ressources afin d’identifier les systèmes qui ne répondent pas aux normes organisationnelles ou aux meilleures pratiques. Azure Security Center fournit une Base de référence de la sécurité unifiée et une protection avancée contre les menaces à l'ensemble des charges de travail cloud hybrides. Avec Security Center, vous pouvez appliquer des stratégies de sécurité à l'ensemble de vos charges de travail, limiter votre exposition aux menaces, détecter et répondre aux attaques grâce aux fonctionnalités suivantes :

* Vue unifiée de la sécurité sur toutes vos charges de travail cloud et locales avec Azure Security Center
* Supervision et évaluations continues de la sécurité pour assurer la conformité et corriger les vulnérabilités
* Outils interactifs et renseignements sur les menaces contextuels pour simplifier les investigations
* Journalisation étendue et intégration aux informations de sécurité existantes

### <a name="extending-cloud-native-policies"></a>Extension des stratégies de type Native cloud

L'utilisation du cloud peut réduire une partie du fardeau que représente la sécurité. Microsoft assure la sécurité physique des centres de données Azure et contribue à protéger la plateforme cloud des menaces d'infrastructure, comme les attaques DDoS. Sachant que Microsoft emploie des milliers de spécialistes de la cybersécurité qui œuvrent chaque jour pour garantir la sécurité, les ressources engagées pour atténuer les cyberattaques sont considérables. En fait, la plupart des entreprises qui se demandaient autrefois si le cloud était sécurisé savent maintenant que compte tenu du niveau d'investissement que des fournisseurs comme Microsoft engagent en termes de ressources humaines et d'infrastructures, le cloud est en réalité plus sécurisé que la plupart des centres de données locaux.

Malgré cet investissement dans la Base de référence de la sécurité Native cloud, il est conseillé d'étendre les stratégies de type Native cloud par défaut de toute stratégie Base de référence de la sécurité. Voici quelques exemples de stratégies étendues à prendre en compte, même dans un environnement de type Native cloud :

* **Sécuriser des machines virtuelles**. La sécurité doit être la priorité absolue de toutes les organisations, et pour la garantir, vous devez : évaluer votre état de sécurité, vous protéger contre les menaces, puis détecter et réagir rapidement aux menaces qui surviennent.
* **Protéger le contenu des machines virtuelles**. La mise en place de sauvegardes automatiques régulières est essentielle pour vous protéger des erreurs des utilisateurs. Mais ce n'est pas suffisant ; vous devez également vous assurer que vos sauvegardes sont protégées des cyberattaques et disponibles lorsque vous en avez besoin.
* **Superviser les machines virtuelles et les applications**. Ce modèle englobe différentes tâches, notamment l'obtention d'insights sur l'intégrité de vos machines virtuelles, l'identification des interactions entre elles et l'établissement de moyens pour superviser les applications exécutées par ces machines virtuelles. Toutes ces tâches sont essentielles pour permettre à vos applications de fonctionner 24 heures sur 24.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez examiné l'exemple de stratégie Base de référence de la sécurité Native cloud, revenez au [Guide de révision des stratégies](../policy-compliance/what-is-a-cloud-policy-review.md) et utilisez cet exemple afin de créer vos propres stratégies pour l'adoption du cloud.

> [!div class="nextstepaction"]
> [Élaborez vos propres stratégies à l'aide du Guide de révision des stratégies](../policy-compliance/what-is-a-cloud-policy-review.md)
