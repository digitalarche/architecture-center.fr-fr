---
title: 'Framework d’adoption du cloud : Grandes entreprises – Évolution de la base de référence des identités'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Grandes entreprises – Évolution de la base de référence des identités
author: BrianBlanchard
ms.openlocfilehash: efda14819bfbc70632dc9bb8b4c6c5aca96004c0
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900878"
---
# <a name="large-enterprise-identity-baseline-evolution"></a>Grandes entreprises : Évolution de la base de référence des identités

Cet article fait évoluer le scénario en ajoutant des contrôles de la base de référence des identités au produit minimum viable de la gouvernance.

## <a name="evolution-of-the-narrative"></a>Évolution du scénario

La justification commerciale pour migrer les deux centres de données vers le cloud a été approuvée par le directeur financier. Lors de l’étude de faisabilité technique, plusieurs obstacles ont été identifiés :

- Les données protégées et les applications stratégiques représentent 25 % des charges de travail des deux centres de données. Aucune des deux ne peut être éliminée tant que les stratégies de gouvernance en vigueur concernant les informations d’identification personnelle et les applications stratégiques n’ont pas été modernisées.
- Dans ces centres de données, 7 % des ressources ne sont pas compatibles avec le cloud. Elles seront déplacées vers un autre centre de données avant l’arrêt du contrat régissant le centre de données.
- 15 % des ressources du centre de données (750 machines virtuelles) présentent une dépendance à l’authentification héritée ou à l’authentification multifacteur proposée par des fournisseurs tiers.
- La connexion VPN entre les centres de données existants et Azure offre des vitesses de transmission des données et une latence insuffisantes pour migrer le volume de ressources dans un délai de deux ans avant de mettre le centre de données hors service.

Les deux premiers obstacles seront atténués en parallèle. Cet article porte sur la résolution des troisième et quatrième obstacles.

### <a name="evolution-of-the-cloud-governance-team"></a>Évolution de l’équipe de gouvernance cloud

L’équipe de gouvernance cloud se développe. Étant donné l’accroissement du besoin de prise en charge en matière de gestion des identités, un administrateur des systèmes issu de l’équipe de base de référence des identités participe désormais à une réunion hebdomadaire afin d’informer les membres actuels de l’équipe sur les nouveautés.

### <a name="evolution-of-the-current-state"></a>Évolution de l’état actuel

L’équipe informatique est autorisée à mettre en œuvre les plans du directeur informatique et du directeur financier qui visent à mettre les deux centres de données hors service. Toutefois, le service informatique est préoccupé, car 750 ressources (15 % du total) de ces centres de données devront être déplacées ailleurs que dans le cloud.

### <a name="evolution-of-the-future-state"></a>Évolution de l’état futur

Les nouveaux plans relatifs à l’état futur requièrent une solution de base de référence des identités plus robuste afin de migrer les 750 machines virtuelles tout en conservant les exigences d’authentification héritées. Hormis pour ces deux centres de données, il est probable qu’un pourcentage semblable de ressources d’autres centres de données soient affectées par ce même défi.
Désormais, l’état futur doit également bénéficier d’une connexion entre le fournisseur de cloud et la solution de l’entreprise (MPLS ou location de ligne).

Les modifications apportées aux états actuel et futur exposent à de nouveaux risques qui nécessiteront de nouvelles instructions de stratégie.

## <a name="evolution-of-tangible-risks"></a>Évolution des risques tangibles

**Interruption des activités pendant la migration**. La migration vers le cloud crée un risque contrôlé et limité dans le temps qu’il est possible de gérer. Le déplacement de matériel vieillissant à l’autre bout du monde fait courir un risque bien plus élevé. Une stratégie d’atténuation des risques est nécessaire pour éviter les interruptions des activités.

**Dépendances d’identité existantes**. Les dépendances entre des services d’authentification et d’identité existants sont susceptibles de retarder ou d’empêcher la migration de certaines charges de travail vers le cloud. Si les deux centres de données ne peuvent pas être retournés à temps, les frais de location des centres de données s’élèveront à des millions de dollars.

Ce risque métier peut être lié à quelques risques techniques :

- Il se peut que l’authentification héritée ne soit pas disponible dans le cloud, limitant ainsi le déploiement de certaines applications.
- Il se peut que la solution d’authentification multifacteur tierce actuelle ne soit pas disponible dans le cloud, limitant ainsi le déploiement de certaines applications.
- Le réoutillage ou le déplacement de cloud peut entraîner des interruptions et générer des coûts supplémentaires.
- La vitesse et la stabilité de la connexion VPN risquent de compromettre la migration.
- Le trafic entrant dans le cloud peut entraîner des problèmes de sécurité dans d’autres parties du réseau global.

## <a name="evolution-of-the-policy-statements"></a>Évolution des instructions de stratégie

Les modifications suivantes apportées à la stratégie permettront de réduire les nouveaux risques et de guider l’implémentation.

1. Le fournisseur de cloud choisi doit proposer un moyen d’authentification via des méthodes héritées.
2. Le fournisseur de cloud choisi doit proposer un moyen d’authentification via la solution d’authentification multifacteur tierce actuelle.
3. Une connexion privée à haute vitesse doit être établie entre le fournisseur de cloud et le fournisseur de télécommunications de l’entreprise. Cette connexion relie le fournisseur de cloud au réseau international de centres de données.
4. Tant que des exigences de sécurité suffisantes ne sont pas établies, le trafic public entrant ne peut pas accéder aux ressources de l’entreprise qui sont hébergées dans le cloud. Tous les ports sont bloqués depuis n’importe quelle source en hors du réseau étendu global.

## <a name="evolution-of-the-best-practices"></a>Évolution des meilleures pratiques

Cette conception du produit minimum viable de la gouvernance va évoluer de façon à inclure de nouvelles stratégies Azure ainsi qu’une implémentation d’Active Directory sur une machine virtuelle. Ensemble, ces deux modifications de la conception permettent de répondre aux nouvelles instructions de la stratégie d’entreprise.

Voici les nouvelles meilleures pratiques :

1. Blueprint de la zone démilitarisée : Le côté local de la zone démilitarisée doit être configuré de manière à autoriser la communication entre la solution suivante et les serveurs Active Directory locaux. Pour respecter cette meilleure pratique, la zone démilitarisée doit activer Active Directory Domain Services au-delà des limites du réseau.
2. via les modèles Azure Resource Manager :
    1. Définissez un groupe de sécurité réseau pour bloquer le trafic externe et mettre le trafic interne sur liste verte.
    1. Déployez deux machines virtuelles AD dans une paire à charge équilibrée basée sur une image finale (golden). Au premier démarrage, cette image exécute un script PowerShell pour joindre le domaine et s’inscrire auprès des services du domaine. Pour plus d’informations, consultez la page [Extension d’Active Directory Domain Services (AD DS) à Azure](../../../../reference-architectures/identity/adds-extend-domain.md).
3. Azure Policy : Appliquez le groupe de sécurité réseau à toutes les ressources.
4. Blueprint Azure
    1. Créez un blueprint nommé `active-directory-virtual-machines`.
    1. Ajoutez chaque modèle et stratégies AD au blueprint.
    1. Publiez le blueprint sur un groupe d’administration valable quelconque.
    1. Appliquez le blueprint à un abonnement quelconque exigeant l’authentification multifacteur tierce ou héritée.
    1. L’instance d’AD qui est en cours d’exécution dans Azure est désormais utilisable en tant qu’extension de la solution AD locale. Elle peut ainsi s’intégrer avec l’outil d’authentification multifacteur existant et proposer l’authentification par revendication par le biais de fonctionnalités Active Directory existantes.

## <a name="conclusion"></a>Conclusion

L’ajout de ces modifications au produit minimum viable de la gouvernance permet d’atténuer nombre des risques évoqués dans cet article. Chaque équipe d’adoption du cloud peut alors rapidement dépasser ces obstacles.

## <a name="next-steps"></a>Étapes suivantes

À mesure que l’adoption du cloud évolue et offre davantage de valeur aux activités métier, les risques et besoins en matière de gouvernance du cloud doivent aussi évoluer. Voici quelques évolutions susceptibles de se produire. Pour l’entreprise fictive que nous suivons dans ce parcours, le prochain déclencheur consiste à inclure des données protégées dans le plan d’adoption du cloud. Cette modification exigera de nouveaux contrôles de sécurité.

> [!div class="nextstepaction"]
> [Évolution de la base de référence de la sécurité](./security-baseline-evolution.md)
