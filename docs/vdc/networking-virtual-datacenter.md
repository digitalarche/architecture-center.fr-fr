---
title: 'Centre de données virtuel Azure : perspective réseau'
description: Découvrez comment créer votre centre de données virtuel dans Azure
author: tracsman
manager: rossort
tags: azure-resource-manager
ms.service: virtual-network
ms.date: 09/24/2018
ms.author: jonor
ms.openlocfilehash: b10930ac6d6458872d8b626825d21bd0bed2b807
ms.sourcegitcommit: 16bc6a91b6b9565ca3bcc72d6eb27c2c4ae935e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/28/2018
ms.locfileid: "52550459"
---
# <a name="azure-virtual-datacenter-a-network-perspective"></a>Centre de données virtuel Azure : perspective réseau

## <a name="overview"></a>Vue d’ensemble
La migration d’applications locales vers Azure, même en l’absence de changements majeurs, permet aux organisations de bénéficier d’une infrastructure sécurisée et rentable. Il s’agit de l’approche **lift-and-shift**. Pour tirer le meilleur parti de l’agilité du cloud computing, les entreprises doivent faire évoluer leurs architectures afin d’exploiter au mieux les fonctionnalités Azure. Microsoft Azure offre des services et une infrastructure hyperscale, des capacités et une fiabilité de niveau entreprise ainsi que de nombreuses options de connectivité hybride. 

Les clients peuvent choisir d’accéder à ces services cloud via Internet ou avec Azure ExpressRoute, qui fournit une connectivité réseau privée. La plateforme Microsoft Azure permet aux clients d’étendre leur infrastructure dans le cloud et de générer des architectures multiniveaux sans interruption. Les partenaires Microsoft offrent également des fonctionnalités améliorées en proposant des services de sécurité et des appliances virtuelles optimisées pour s’exécuter dans Azure.

Cet article fournit une vue d’ensemble des modèles et des conceptions qui permettent de répondre aux préoccupations de nombreux clients concernant la mise à l’échelle, les performances et la sécurité de l’architecture quand ces clients envisagent une migration massive vers le cloud. Cet article décrit également comment adapter les différents rôles informatiques d’une organisation à la gestion et à la gouvernance du système. L’accent est mis sur les exigences de sécurité et l’optimisation des coûts.

## <a name="what-is-a-virtual-datacenter"></a>Qu’est-ce qu’un centre de données virtuel ?
Les solutions cloud ont d’abord été conçues pour héberger des applications uniques et relativement isolées dans le domaine public. Cette approche a donné de bons résultats pendant quelques années. Les avantages des solutions cloud sont ensuite devenus évidents, et plusieurs charges de travail à grande échelle ont été hébergées dans le cloud. Il s’est révélé crucial de résoudre les problèmes de sécurité, de fiabilité, de performances et de coûts des déploiements dans une ou plusieurs régions sur l’ensemble du cycle de vie du service cloud.

Le diagramme suivant de déploiement cloud montre un exemple de faille de sécurité dans la **zone rouge**. La **zone jaune** indique qu’il est possible d’optimiser les appliances virtuelles réseau parmi les charges de travail.

[![0]][0]

Le VDC (centre de données virtuel) est né d’un besoin de scalabilité des charges de travail de l’entreprise. Il répond également aux problèmes rencontrés durant la prise en charge d’applications à grande échelle dans le cloud public.

Le VDC ne se limite pas aux charges de travail des applications dans le cloud. Il est lié également au réseau, à la sécurité, à la gestion et à l’infrastructure. Les DNS et les services d’annuaire en sont de bons exemples. Il fournit généralement une connexion privée à un réseau local ou un centre de données. Plus le volume des charges de travail déplacées vers Azure augmente, plus il est important de réfléchir à l’infrastructure sous-jacente et aux objets dans lesquels ces charges de travail sont placées. Réfléchissez à la manière dont les ressources sont structurées pour éviter la prolifération de centaines d’**îlots de charge de travail** qui doivent être gérés séparément avec des flux de données, des modèles de sécurité et des problèmes de conformité indépendants.

Le centre de données virtuel est une collection d’entités distinctes mais liées, qui ont des fonctions, des fonctionnalités et une infrastructure communes. En visualisant vos charges de travail sous la forme d’un VDC intégré, vous pouvez réduire les coûts grâce aux économies d’échelle et à l’optimisation de la sécurité via la centralisation des composants et des flux de données. Vous bénéficiez également d’une simplification des opérations, de la gestion et des audits de conformité.

> [!NOTE]  
> Il est important de comprendre que le VDC n’est **pas** un produit Azure distinct. Il s’agit de la combinaison de diverses fonctionnalités et capacités qui doivent répondre de manière précise à vos exigences. Le VDC est un moyen d’utiliser vos charges de travail et Azure pour optimiser vos ressources et capacités dans le cloud. Le centre de données virtuel constitue donc une approche modulaire permettant de créer des services informatiques dans Azure, tout en respectant les rôles et les responsabilités au sein de l’organisation.

Le VDC peut aider les entreprises à déplacer des charges de travail et des applications vers Azure dans le cadre des scénarios suivants :

-   Héberger plusieurs charges de travail liées
-   Migrer les charges de travail d’un environnement local vers Azure
-   Implémenter les exigences de gestion partagée ou centralisée de la sécurité et des accès dans l’ensemble des charges de travail
-   Combiner de manière appropriée Azure DevOps et la centralisation des services informatiques pour une grande entreprise

La clé permettant d’accéder aux avantages du VDC repose sur l’utilisation d’une topologie centralisée, de type hub-and-spoke, avec un mélange de fonctionnalités Azure : 

- [Réseau virtuel Azure][VNet] 
- [Groupes de sécurité réseau (NSG)][NSG]
- [Appairage de réseaux virtuels][VNetPeering] 
- [UDR (routes définies par l’utilisateur)][UDR]
- Services d’identité Azure et [contrôle d’accès en fonction du rôle (RBAC)][RBAC] 
- Éventuellement [Pare-feu Azure][AzFW], [Azure DNS][DNS], [Azure Front Door Service][AFD] et [Azure Virtual WAN][vWAN]

## <a name="who-should-implement-a-virtual-datacenter"></a>Qui doit implémenter un centre de données virtuel ?
Tout client Azure ayant besoin de migrer plusieurs charges de travail vers Azure peut tirer parti de l’utilisation de ressources communes. En fonction de l’ampleur du projet, même les applications individuelles peuvent tirer parti des modèles et des composants utilisés pour créer le VDC.

Si votre organisation dispose d’une infrastructure informatique, d’un réseau, d’une sécurité, d’une équipe de conformité ou d’un service reposant sur un modèle centralisé, le VDC peut vous aider à appliquer des points de stratégie et à répartir les tâches. Il garantit également l’uniformité des composants communs sous-jacents tout en offrant aux équipes liées aux applications la liberté et le contrôle nécessaires pour répondre à vos besoins.

Les organisations qui se tournent vers Azure DevOps peuvent utiliser les concepts du VDC pour fournir des groupes de ressources Azure autorisées, et garantir un contrôle total au sein de ces groupes. Les groupes sont des abonnements ou des groupes de ressources d’un abonnement commun. Toutefois, les limites relatives au réseau et à la sécurité restent conformes à la définition d’une stratégie centralisée dans un réseau virtuel de hubs et un groupe de ressources.

## <a name="considerations-when-you-implement-a-virtual-datacenter"></a>Considérations liées à l’implémentation d’un centre de données virtuel
Quand vous concevez votre VDC, vous devez prendre en compte plusieurs problèmes essentiels :

-   Services d’identité et d’annuaire
-   Infrastructure de sécurité
-   Connectivité avec le cloud
-   Connectivité au sein du cloud

### <a name="identity-and-directory-services"></a>Services d’identité et d’annuaire
Les services d’identité et d’annuaire constituent un aspect clé de tous les centres de données, à la fois en local et dans le cloud. La notion d’identité est liée à tous les aspects des accès et des autorisations vis-à-vis des services au sein du VDC. Pour garantir que seuls les utilisateurs et processus autorisés ont accès à votre compte et à vos ressources Azure, Azure utilise plusieurs types d’informations d’identification pour l’authentification. Ces informations regroupent les mots de passe permettant d’accéder au compte Azure, les clés de chiffrement, les signatures numériques et les certificats. 

[Azure Multi-Factor Authentication][MFA] est une couche de sécurité supplémentaire permettant d’accéder aux services Azure. Elle fournit une authentification renforcée avec tout un éventail d’options de vérification simples. Les clients peuvent choisir la méthode qu’ils préfèrent. Les options incluent l’appel téléphonique, l’envoi d’un SMS ou la notification d’application mobile. 

Toute grande entreprise doit définir un processus de gestion des identités qui décrit la gestion des différentes identités, leur authentification ainsi que les autorisations, rôles et privilèges dans le VDC. Ce processus a pour but d’améliorer la sécurité et la productivité tout en réduisant les coûts, les temps d’arrêt et les tâches manuelles répétitives.

Les entreprises et les organisations peuvent avoir besoin d’une combinaison de services exigeante pour différents secteurs d’activité. De plus, les employés ont souvent des rôles distincts selon les projets auxquels ils participent. Le VDC nécessite une coopération satisfaisante entre les différentes équipes, chacune ayant un rôle spécifique, afin de garantir le bon fonctionnement des systèmes avec une gouvernance adéquate. La matrice des responsabilités, des accès et des droits peut se révéler complexe. La gestion des identités dans le VDC est implémentée via [Azure AD (Azure Active Directory)][AAD] et le contrôle d’accès en fonction du rôle (RBAC).

Un service d’annuaire est une infrastructure d’informations partagée qui permet de localiser, de gérer, d’administrer et d’organiser les éléments et ressources réseau utilisés quotidiennement. Ces ressources peuvent inclure des volumes, des dossiers, des fichiers, des imprimantes, des utilisateurs, des groupes, des appareils et d’autres objets. Chacune des ressources du réseau est considérée comme un objet par le serveur d’annuaire. Les informations concernant une ressource sont stockées sous la forme d’une collection d’attributs associée à cette ressource ou à cet objet.

Tous les services professionnels Microsoft Online utilisent Azure Active Directory (Azure AD) pour l’authentification et les autres besoins d’identification. Azure Active Directory est une solution cloud complète et hautement disponible de gestion des identités et des accès qui associe des services d'annuaire principaux, une gouvernance avancée des identités et la gestion de l'accès aux applications. Il est possible d’intégrer Azure AD au service Active Directory local afin d’autoriser l’authentification unique pour toutes les applications cloud et hébergées localement. Les attributs utilisateur du service Active Directory local peuvent être automatiquement synchronisés avec Azure AD.

Il n’est pas nécessaire qu’un seul administrateur général affecte toutes les autorisations dans une implémentation de VDC. À la place, chaque département spécifique, ou groupe d’utilisateurs ou de services du service d’annuaire, peut disposer des autorisations nécessaires pour gérer ses propres ressources dans le VDC. La structuration des autorisations doit être équilibrée. Trop d’autorisations entravent l’efficacité des performances. Trop peu d’autorisations ou des autorisations insuffisantes augmentent les risques de sécurité. Le contrôle d’accès en fonction du rôle (RBAC) Azure contribue à résoudre ce problème en offrant une gestion affinée des accès aux ressources du VDC.

#### <a name="security-infrastructure"></a>Infrastructure de sécurité
Cette infrastructure de sécurité fait référence à la répartition du trafic dans le segment réseau virtuel spécifique du VDC ainsi qu’au mode de contrôle des flux d’entrée et de sortie dans l’ensemble du VDC. Azure est basé sur une architecture multilocataire qui bloque le trafic non autorisé et non intentionnel entre les déploiements. Il utilise l’isolement de réseau virtuel, les ACL (listes de contrôle d’accès), les équilibreurs de charge, les filtres IP et les stratégies de flux de trafic. La traduction d’adresses réseau (NAT) sépare le trafic réseau interne du trafic externe.

La structure Azure alloue des ressources d’infrastructure aux charges de travail des locataires et gère les communications à destination et en provenance des machines virtuelles. L’hyperviseur Azure applique la séparation de mémoire et de processus entre les machines virtuelles, et route en toute sécurité le trafic réseau vers les locataires du système d’exploitation invité.

#### <a name="connectivity-to-the-cloud"></a>Connectivité avec le cloud
L’implémentation d’un VDC nécessite une connectivité avec les réseaux externes pour offrir des services aux clients, aux partenaires et aux utilisateurs internes. Cela implique généralement une connectivité non seulement à Internet, mais également aux réseaux et centres de données locaux.

Les clients contrôlent les services ayant accès à l’Internet public et ceux accessibles à partir de l’Internet public via le [Pare-feu Azure][AzFW] ou d’autres types de NVA (appliance virtuelle réseau). Ils contrôlent également les stratégies de routage personnalisées à l’aide de [routes définies par l’utilisateur][UDR] ainsi que le filtrage réseau à l’aide de [groupes de sécurité réseau][NSG]. Nous recommandons de protéger également les ressources Internet via le service [Azure DDoS Protection Standard][DDOS].

Les entreprises doivent souvent connecter leurs implémentations du VDC à des centres de données locaux ou d’autres ressources. La connectivité entre Azure et les réseaux locaux est donc un aspect essentiel de la conception d’une architecture efficace. Pour créer des interconnexions entre des implémentations de VDC et des réseaux locaux dans Azure, les entreprises disposent de deux méthodes : transit via Internet ou connexions directes privées.

Un [VPN site à site Azure][VPN] connecte le réseau local d’une entreprise à son implémentation du VDC via Internet. Le trafic via le réseau VPN est chiffré à l’aide du tunneling IPSec et IKE. Une connexion de site à site Azure est flexible et rapide à créer. Ce VPN ne nécessite aucun approvisionnement supplémentaire, car toutes les connexions se font via Internet.

En cas de très nombreuses connexions VPN, le service réseau [Azure Virtual WAN][vWAN] offre une connectivité de branche à branche optimisée et automatisée via Azure. Avec Virtual WAN, vous pouvez connecter et configurer des appareils de succursale pour communiquer avec Azure. Vous pouvez effectuer cette connexion manuellement ou à l’aide d’appareils de fournisseurs favoris via un partenaire Virtual WAN. Les appareils de fournisseurs favoris présentent plusieurs avantages, notamment la simplification de l’utilisation et de la connectivité ainsi que la gestion de la configuration. Le tableau de bord intégré Azure WAN fournit des insights de résolution de problème instantanés qui peuvent vous faire gagner du temps. De plus, il vous permet de voir facilement la connectivité de site à site à grande échelle.

[ExpressRoute][ExR], un service de connectivité Azure, permet d’établir des connexions privées entre le réseau local de l’entreprise et son implémentation du VDC. Il offre une sécurité, une fiabilité et des vitesses supérieures, allant jusqu’à 10 Gbits/s, avec une latence constante. ExpressRoute est utile pour les implémentations de VDC, car les clients ExpressRoute bénéficient des règles de conformité associées aux connexions privées. [ExpressRoute Direct][ExRD] permet de se connecter directement aux routeurs Microsoft à 100 Gbits/s pour les clients ayant des besoins en bande passante plus importants.

Le déploiement de connexions ExpressRoute implique généralement la souscription d’un engagement auprès d’un fournisseur de services ExpressRoute. Les clients qui doivent être opérationnels rapidement commencent généralement par utiliser un VPN site à site pour établir la connectivité entre le VDC et les ressources locales. Ils migrent ensuite vers une connexion ExpressRoute quand l’interconnexion physique avec le fournisseur de services est achevée.

#### <a name="connectivity-within-the-cloud"></a>Connectivité au sein du cloud
Les services de [réseaux virtuels][VNet] et d’[appairage de réseaux virtuels][VNetPeering] sont les services de connectivité réseau de base au sein du VDC. Un réseau virtuel garantit une limite naturelle pour l’isolement des ressources de VDC. De plus, l’appairage de réseaux virtuels permet l’intercommunication entre différents réseaux virtuels de la même région Azure, voire d’une région à une autre. Le contrôle du trafic à l’intérieur d’un réseau virtuel et entre plusieurs réseaux virtuels doit correspondre à un ensemble de règles de sécurité spécifiées via les listes de contrôle d’accès. Consultez les informations sur les [groupes de sécurité réseau][NSG], les [NVA (appliances virtuelles réseau)][NVA] et les [tables de routage personnalisées pour les routes définies par l’utilisateur][UDR].

## <a name="virtual-datacenter-overview"></a>Vue d’ensemble du centre de données virtuel

### <a name="topology"></a>Topologie
La topologie **hub-and-spoke** est un modèle qui permet d’étendre un centre de données virtuel au sein d’une seule région Azure.

[![1]][1]

Un hub est la zone réseau centrale qui contrôle et inspecte le trafic d’entrée ou de sortie entre différentes zones : Internet, le réseau local et les spokes (rayons). La topologie hub-and-spoke offre au service informatique un moyen efficace d’appliquer des stratégies de sécurité à un emplacement central. Elle réduit également les risques de configuration incorrecte et d’exposition.

Le concentrateur contient les composants de service communs consommés par les rayons. Voici des exemples de services centraux usuels :

-   Infrastructure Windows Active Directory, nécessaire à l’authentification utilisateur des tiers qui souhaitent accéder aux charges de travail du rayon à partir de réseaux non approuvés. Elle inclut les services de fédération Active Directory (AD FS) associés.
-   Service DNS destiné à résoudre le nommage pour la charge de travail dans les rayons, afin de permettre l’accès aux ressources locales et sur Internet si le service [Azure DNS][DNS] n’est pas utilisé.
-   Infrastructure à clé publique (PKI) permettant d’implémenter l’authentification unique sur les charges de travail.
-   Contrôle de flux du trafic TCP et UDP entre les zones du réseau à rayons et Internet.
-   Contrôle de flux entre les rayons et les ressources locales.
-   Si nécessaire, contrôle de flux entre un rayon et un autre.

Le VDC réduit le coût global au moyen de l’infrastructure de concentrateur partagée entre plusieurs rayons.

Le rôle de chaque rayon peut consister à héberger différents types de charges de travail. Les rayons offrent également une approche modulaire pour les déploiements renouvelables des mêmes charges de travail. C’est le cas, par exemple, des phases de développement et de test, de tests d’acceptation utilisateur, de préproduction et de production. Les rayons permettent également de séparer et d’activer différents groupes au sein de votre organisation. Les groupes Azure DevOps en sont un exemple. À l’intérieur d’un rayon, il est tout aussi possible de déployer une charge de travail de base que des charges de travail multiniveaux complexes avec un contrôle du trafic entre les différents niveaux.

#### <a name="subscription-limits-and-multiple-hubs"></a>Limites d’abonnement et concentrateurs multiples
Dans Azure, chacun des composants, quel qu’en soit le type, est déployé dans un abonnement Azure. L’isolement des composants Azure dans différents abonnements Azure peut répondre aux exigences de différents métiers. C’est le cas, par exemple, de la mise en place de niveaux d’accès et d’autorisation différenciés.

Vous pouvez effectuer un scale-up d’une seule implémentation de VDC vers un grand nombre de rayons. Mais comme pour tout système informatique, les plateformes ont des limites. Le déploiement du hub est lié à un abonnement Azure spécifique, qui comporte des restrictions et des limites. Le nombre maximal d’appairages de réseaux virtuels est un bon exemple. Pour plus d’informations, consultez [Abonnement Azure et limites, quotas et contraintes de service][Limits]. Au cas où les limites pourraient être un problème, l’architecture peut encore faire l’objet d’un scale-up. Étendez le modèle à partir d’une seule topologie hub-and-spoke vers un cluster de topologies hub-and-spoke. Interconnectez plusieurs hubs dans une ou plusieurs régions Azure à l’aide de l’appairage de réseaux virtuels, d’ExpressRoute, de Virtual WAN ou d’un VPN site à site.

[![2]][2]

L’introduction de plusieurs hubs augmente les efforts liés au coût et à la gestion du système. Cela se justifie donc uniquement pour des raisons de scalabilité, par exemple pour étendre les limites du système ou à des fins de redondance ainsi que dans le cadre d’une réplication régionale, par exemple pour fournir un niveau de performance à l’utilisateur final ou une reprise d’activité. Dans les scénarios qui nécessitent plusieurs concentrateurs, ces derniers doivent tous, dans la mesure du possible, offrir le même ensemble de services afin d’en faciliter l’exploitation.

#### <a name="interconnection-between-spokes"></a>Interconnexion entre les rayons
Il est possible d’implémenter des charges de travail multiniveaux complexes dans un même rayon. Vous pouvez implémenter des configurations multiniveaux en utilisant un seul sous-réseau pour chaque niveau du même réseau virtuel. Filtrez les flux à l’aide de NSG (groupes de sécurité réseau).

Un architecte peut souhaiter déployer une charge de travail multiniveau sur plusieurs réseaux virtuels. Avec l’appairage de réseaux virtuels, vous pouvez connecter des rayons à d’autres rayons du même hub ou de hubs distincts. Ce scénario s’applique généralement aux cas où les serveurs de traitement d’application sont situés dans un même rayon ou réseau virtuel. La base de données est déployée sur un autre rayon ou réseau virtuel. Dans ce cas, il est facile d’interconnecter les rayons avec l’appairage de réseaux virtuels et d’éviter ainsi le transit par le hub. Toutefois, il convient d’étudier avec soin l’architecture et la sécurité pour vérifier que le contournement du hub n’élude pas de points de sécurité ou d’audit importants susceptibles d’exister uniquement dans le hub.

[![3]][3]

Il est également possible d’interconnecter des rayons à un rayon jouant le rôle de concentrateur. Cette approche crée une hiérarchie à deux niveaux : les rayons du niveau supérieur (niveau 0) deviennent le hub de rayons inférieurs (niveau 1) dans la hiérarchie. Les rayons d’une implémentation de VDC doivent transférer le trafic vers le hub central pour qu’il atteigne le réseau local ou Internet. Une architecture à deux niveaux de concentrateur introduit un routage complexe qui supprime les avantages d’une relation hub-and-spoke simple.

Bien qu’Azure autorise des topologies complexes, les deux principes fondamentaux du concept de VDC sont la répétabilité et la simplicité. Pour réduire l’effort de gestion, la conception d’une topologie hub-and-spoke simple est l’architecture de référence que nous recommandons pour un VDC.

### <a name="components"></a>Composants
L’implémentation d’un centre de données virtuel comporte quatre types de composant de base : **infrastructure**, **réseaux de périmètre**, **charges de travail** et **supervision**.

Chaque type de composant intègre plusieurs ressources et fonctionnalités Azure. L’implémentation du VDC d’une entreprise comprend des instances de plusieurs types de composant et de plusieurs variantes du même type de composant. Par exemple, vous pouvez disposer de nombreuses instances de charge de travail distinctes, séparées de façon logique, qui représentent différentes applications. Utilisez ces différents types de composants et instances afin de générer le VDC.

[![4]][4]

L’architecture générale ci-dessus d’une implémentation de VDC montre différents types de composant utilisés dans différentes zones de la topologie hub-and-spoke. Le diagramme illustre les composants d’infrastructure dans les différentes parties de l’architecture.

Une bonne pratique pour un centre de données local ou une implémentation de VDC consiste à baser les droits d’accès et les privilèges sur des groupes. Utilisez des groupes, plutôt que des utilisateurs individuels, pour maintenir une gestion cohérente des stratégies d’accès entre les équipes et réduire les erreurs de configuration. Attribuez et supprimez des utilisateurs dans les groupes appropriés pour permettre l’actualisation continue des privilèges d’un utilisateur spécifique.

Chaque groupe de rôles doit avoir un préfixe unique pour ses noms. Ce préfixe permet d’identifier facilement le groupe associé à une charge de travail spécifique. Par exemple, une charge de travail hébergeant un service d’authentification peut être associée à des groupes appelés **AuthServiceNetOps**, **AuthServiceSecOps**, **AuthServiceDevOps** et **AuthServiceInfraOps**. Les rôles centralisés ou les rôles non associés à un service spécifique peuvent être précédés de **Corp**. Exemple : **CorpNetOps**.

De nombreuses organisations utilisent une variante des groupes ci-après pour procéder à une répartition générale des rôles :

-   Le groupe du service informatique central, **Corp,**, détient les droits de propriété permettant de contrôler les composants d’infrastructure. Exemples : réseau et sécurité. Le groupe doit avoir le rôle de contributeur pour l’abonnement, le contrôle du hub et les droits de contributeur réseau sur les rayons. Les grandes organisations divisent souvent ces responsabilités de gestion entre plusieurs équipes. C’est le cas, par exemple, pour un groupe d’opérations réseau **CorpNetOps**, exclusivement axé sur le réseau, et pour un groupe d’opérations de sécurité **CorpSecOps**, responsable de la stratégie de pare-feu et de sécurité. Dans ce cas précis, il convient de créer deux groupes distincts pour l’attribution de ces rôles personnalisés.
-   Le groupe de développement/test, **AppDevOps,**, est responsable du déploiement des charges de travail d’application ou de service. Ce groupe joue le rôle de contributeur de machine virtuelle pour les déploiements IaaS. Il peut également jouer un ou plusieurs rôles de contributeur PaaS. Consultez [Rôles intégrés pour les ressources Azure][Roles]. Éventuellement, l’équipe de développement/test peut avoir besoin de visibilité sur les stratégies de sécurité (NSG) et les stratégies de routage (UDR) dans le hub ou un rayon spécifique. En plus du rôle de contributeur pour les charges de travail, ce groupe nécessite également le rôle de lecteur de réseau.
-   Le groupe des opérations et de la maintenance, **CorpInfraOps** ou **AppInfraOps**, est chargé de gérer les charges de travail en production. Ce groupe doit être un contributeur d’abonnement sur les charges de travail dans tous les abonnement de production. Certaines organisations peuvent également déterminer si elles ont besoin d’un groupe supplémentaire de type équipe du support technique pour la prise en charge du processus d’escalade avec le rôle de contributeur d’abonnement en production ainsi que l’abonnement au hub central. Le groupe supplémentaire corrige les problèmes de configuration potentiels dans l’environnement de production.

Le VDC est structuré de telle sorte que les groupes créés pour les groupes du service informatique central gérant le hub aient des groupes correspondants au niveau de la charge de travail. En plus de la gestion des ressources de hub, seuls les groupes du service informatique central peuvent contrôler l’accès externe et les autorisations de niveau supérieur sur l’abonnement. Toutefois, les groupes de charge de travail contrôlent les ressources et les autorisations de leur réseau virtuel indépendamment du groupe du service informatique central.

Partitionnez le VDC pour héberger de manière sécurisée plusieurs projets appartenant à différents métiers. Tous les projets nécessitent des environnements isolés distincts, par exemple le développement, l’UAT (test d’acceptation utilisateur) et la production. Les abonnements Azure distincts pour chacun de ces environnements assurent un isolement naturel.

[![5]][5]

Le diagramme précédent illustre la relation entre les projets, les utilisateurs et les groupes d’une organisation, et les environnements dans lesquels les composants Azure sont déployés.

En informatique, un environnement ou un niveau désigne généralement un système dans lequel plusieurs applications sont déployées et exécutées. Les grandes entreprises utilisent un environnement de développement pour effectuer et tester des changements ainsi qu’un environnement de production destiné aux utilisateurs finaux. Ces environnements sont séparés, le plus souvent par des environnements de préproduction, pour permettre l’échelonnement des phases de déploiement (lancement), de test et de restauration en cas de problème. Les architectures de déploiement varient sensiblement, mais suivent généralement le processus de base qui consiste à commencer par le niveau développement **DEV** et à finir par le niveau production **PROD**.

Une architecture communément employée pour ces types d’environnement multiniveaux comprend les environnements Azure DevOps pour le développement et les tests, UAT pour la préproduction, et les environnements de production. Les organisations peuvent tirer profit d’un ou de plusieurs locataires Azure AD afin de définir l’accès et les droits pour ces environnements. Le diagramme précédent illustre un cas d’utilisation de deux locataires Azure AD distincts : l’un pour Azure DevOps et UAT, et l’autre exclusivement destiné à la production.

La présence de différents locataires Azure AD met en œuvre la séparation entre les environnements. Le même groupe d’utilisateurs, par exemple le groupe du service informatique central, doit s’authentifier à l’aide d’un URI distinct pour accéder à un autre locataire Azure AD et modifier les rôles ou autorisations des environnements Azure DevOps ou de production d’un projet. L’utilisation d’authentifications utilisateur distinctes pour accéder à différents environnements réduit les éventuelles interruptions et autres problèmes dus à des erreurs humaines.

#### <a name="component-type-infrastructure"></a>Type de composant : infrastructure
Ce type de composant est l’endroit où réside la majeure partie de l’infrastructure sous-jacente. Il s’agit également du composant auquel vos équipes d’informatique centralisée, de sécurité et de conformité consacrent la majorité de leur temps.

[![6]][6]

Les composants d’infrastructure fournissent une interconnexion entre les différents composants du VDC. Ils sont présents dans le hub et les rayons. La responsabilité de la gestion et de la maintenance des composants d’infrastructure est généralement attribuée à l’équipe du service informatique central ou de la sécurité.

L’une des principales tâches de l’équipe d’infrastructure informatique consiste à garantir la cohérence des schémas d’adresse IP dans l’ensemble de l’entreprise. L’espace d’adressage IP privé affecté au VDC doit être cohérent. Il ne peut pas chevaucher les adresses IP privées affectées sur vos réseaux locaux.

NAT sur les routeurs de périphérie locaux ou dans les environnements Azure permet d’éviter les conflits d’adresses IP. Mais cela entraîne des complications pour vos composants d’infrastructure. La simplicité de gestion est l’un des objectifs clés du VDC. L’utilisation de NAT pour résoudre les problèmes liés aux adresses IP n’est pas une solution que nous recommandons.

Les composants d’infrastructure ont les fonctionnalités suivantes :

-   [**Services d’identité et d’annuaire**][AAD]. L’accès à chaque type de ressource dans Azure est contrôlé par une identité stockée dans un service d’annuaire. Le service d’annuaire stocke la liste des utilisateurs et les droits d’accès aux ressources dans un abonnement Azure spécifique. Ces services peuvent exister uniquement dans le cloud. Ils peuvent également être synchronisés avec une identité locale stockée dans Azure Active Directory.
-   [**Réseau virtuel**][VPN]. Les réseaux virtuels sont l’un des composants principaux du VDC. À l’aide des réseaux virtuels, vous pouvez créer une limite d’isolement du trafic sur la plateforme Azure. Un réseau virtuel est composé d’un ou de plusieurs segments de réseau virtuel. Chacun a un préfixe de réseau IP spécifique, appelé sous-réseau. Le réseau virtuel définit une zone de périmètre interne dans laquelle les machines virtuelles IaaS et les services PaaS peuvent établir des communications privées. Les machines virtuelles et les services PaaS d’un réseau virtuel ne peuvent pas communiquer directement avec les machines virtuelles et les services PaaS d’un autre réseau virtuel. Cet isolement se produit même si le client crée les deux réseaux virtuels sous le même abonnement. L’isolement est une propriété critique qui garantit que les machines virtuelles et les communications du client restent privées dans un réseau virtuel.
-   [**Itinéraire défini par l’utilisateur**][UDR]. Le routage du trafic dans un réseau virtuel repose par défaut sur la table de routage système. Une route définie par l’utilisateur est une table de routage personnalisée que les administrateurs réseau peuvent associer à un ou plusieurs sous-réseaux. Cette association remplace le comportement de la table de routage système et définit un chemin de communication dans un réseau virtuel. La présence d’UDR garantit que le trafic de sortie en provenance du rayon transite par des machines virtuelles personnalisées ou des appliances virtuelles réseau et des équilibreurs de charge spécifiques présents dans le hub et les rayons.
-   [**Groupe de sécurité réseau (NSG)**][NSG]. Un groupe de sécurité réseau est une liste de règles de sécurité qui fait office de filtrage du trafic sur les sources IP, les destinations IP, les protocoles, les ports sources IP et les ports de destination IP. Le NSG peut être appliqué à un sous-réseau, à une carte d’interface réseau virtuelle associée à une machine virtuelle Azure, ou aux deux. Les NSG jouent un rôle essentiel dans l’implémentation d’un contrôle de flux approprié dans le hub et les rayons. Le niveau de sécurité offert par le NSG dépend des ports que vous ouvrez et du but dans lequel vous le faites. Les clients doivent appliquer des filtres supplémentaires par machine virtuelle avec des pare-feu basés sur l’hôte, par exemple IPtables ou le Pare-feu Windows.
-   [**DNS**][DNS]. La résolution de noms des ressources dans les réseaux virtuels du VDC est fournie via le DNS. Azure fournit des services DNS pour la résolution de noms [publique][DNS] et [privée][PrivateDNS]. Les zones privées fournissent la résolution de noms au sein d’un réseau virtuel, ainsi qu’entre des réseaux virtuels. Vous pouvez avoir des zones privées qui couvrent plusieurs réseaux virtuels au sein de la même région, et qui s’étendent également à plusieurs régions et abonnements. Pour la résolution publique, Azure DNS fournit un service d’hébergement des domaines DNS. Il offre une résolution de noms à l’aide de l’infrastructure Microsoft Azure. En hébergeant vos domaines dans Azure, vous pouvez gérer vos enregistrements DNS à l’aide des mêmes informations d’identification, les mêmes API, les mêmes outils et la même facturation que vos autres services Azure.
-   [**Abonnement**][SubMgmt] et [**Gestion des groupes de ressources**][RGMgmt]. Un abonnement définit une limite naturelle permettant de créer plusieurs groupes de ressources dans Azure. Les ressources d’un abonnement sont regroupées dans des conteneurs logiques nommés **groupes de ressources**. Le groupe de ressources représente un groupe logique permettant d’organiser les ressources du VDC.
-   [**Contrôle d’accès en fonction du rôle (RBAC)**][RBAC]. RBAC permet de mapper des rôles organisationnels à des droits d’accès à des ressources Azure spécifiques, ce qui vous permet de restreindre les utilisateurs à un sous-ensemble d’actions. Avec RBAC, vous pouvez accorder l’accès en attribuant le rôle approprié à des utilisateurs, groupes et applications dans l’étendue adéquate. L’étendue d’une attribution de rôle peut être un abonnement Azure, un groupe de ressources ou une ressource unique. Le mécanisme RBAC autorise l’héritage des autorisations. Un rôle attribué à une étendue parent accorde également l’accès aux enfants qu’elle contient. Avec RBAC, vous pouvez répartir les tâches et accorder aux utilisateurs uniquement les accès dont ils ont besoin pour faire leur travail. Par exemple, utilisez RBAC pour permettre à un employé de gérer les machines virtuelles d’un abonnement. Un autre employé peut gérer les bases de données SQL du même abonnement.
-   [**Appairage de réseaux virtuels**][VNetPeering]. La fonctionnalité fondamentale qui permet de créer l’infrastructure du VDC est l’appairage de réseaux virtuels. Il s’agit d’un mécanisme qui connecte deux réseaux virtuels de la même région via le réseau du centre de données Azure ou le cœur de réseau (backbone) mondial Azure couvrant plusieurs régions.

#### <a name="component-type-perimeter-networks"></a>Type de composant : réseaux de périmètre
Avec les composants de type [réseau de périmètre][DMZ] (ou réseau DMZ), vous pouvez fournir une connectivité réseau aux réseaux de vos centres de données locaux ou physiques ainsi qu’une connectivité à destination et en provenance d’Internet. Il s’agit également des composants auxquels vos équipes réseau et sécurité consacrent généralement la majorité de leur temps.

Les paquets entrants doivent transiter par les appliances de sécurité du hub avant d’atteindre les serveurs back-end dans les rayons. Exemples : pare-feu, systèmes de détection et de prévention d’intrusion. Avant de quitter le réseau, les paquets Internet des charges de travail doivent également transiter par les appliances de sécurité du réseau de périmètre. Les finalités de ce flux sont la mise en œuvre, l’inspection et l’audit des stratégies.

Les composants de type réseau de périmètre fournissent les fonctionnalités suivantes :

-   [Réseaux virtuels][VNet], [UDR][UDR] et [NSG][NSG]
-   [Appliances virtuelles réseau][NVA]
-   [Azure Load Balancer][ALB]
-   [Azure Application Gateway][AppGW] et [WAF (pare-feu d’applications web)][WAF]
-   [Adresses IP publiques][PIP]
-   [Azure Front Door][AFD]
-   [Pare-feu Azure][AzFW]

En règle générale, les équipes du service informatique central et de la sécurité sont chargées de la définition des exigences et du fonctionnement des réseaux de périmètre.

[![7]][7]

Le diagramme précédent illustre l’application de deux périmètres avec un accès à Internet et à un réseau local. Ces deux périmètres résident dans les hubs DMZ et vWAN. Dans le hub DMZ, vous pouvez effectuer un scale-up du réseau de périmètre vers Internet pour permettre la prise en charge d’un grand nombre de métiers à l’aide de plusieurs batteries d’instances WAF (pare-feu d’applications web) ou de Pare-feu Azure. Dans le hub vWAN, la connectivité hautement scalable, de branche à branche et de branche à Azure, est obtenue via un VPN ou ExpressRoute selon les besoins.

[**Réseaux virtuels**][VNet]. Le hub repose généralement sur un réseau virtuel comprenant plusieurs sous-réseaux pour héberger les différents types de service qui filtrent et inspectent le trafic à destination ou en provenance d’Internet via les instances NVA, WAF et Azure Application Gateway.

[**UDR**][UDR]. Avec les UDR, les clients peuvent déployer des pare-feu, des systèmes de détection ou de prévention d’intrusion ainsi que d’autres appliances virtuelles. Ils peuvent router le trafic réseau via ces appliances de sécurité pour garantir la mise en œuvre, l’audit et l’inspection des stratégies relatives aux limites de sécurité. Vous pouvez créer des UDR dans le hub et les rayons. Ils garantissent que le trafic transite par les machines virtuelles personnalisés, les appliances virtuelles réseau et les équilibreurs de charge spécifiques utilisés par le VDC. Pour vérifier que le trafic généré par les machines virtuelles résidant dans le rayon transite vers les appliances virtuelles appropriées, définissez un UDR dans les sous-réseaux du rayon. Définissez l’adresse IP front-end de l’équilibreur de charge interne en tant que prochain tronçon. L’équilibreur de charge interne distribue le trafic interne vers les appliances virtuelles ou le pool back-end de l’équilibreur de charge.

[**Pare-feu Azure**][AzFW] est un service de sécurité réseau informatique managé qui protège vos ressources Réseau virtuel Azure. Il s’agit d’un service de pare-feu avec état intégral, doté d’une haute disponibilité intégrée et d’une scalabilité illimitée dans le cloud. Vous pouvez créer, appliquer et consigner des stratégies de connectivité réseau et d’application de façon centralisée entre les abonnements et les réseaux virtuels. Le service Pare-feu Azure utilise une adresse IP publique statique pour vos ressources de réseau virtuel. Il permet aux pare-feu externes d’identifier le trafic provenant de votre réseau virtuel. Le service est totalement intégré à Azure Monitor pour la journalisation et les analyses.

[**Appliances virtuelles réseau**][NVA]. Dans le hub, le réseau de périmètre disposant d’un accès à Internet est normalement géré via une instance de Pare-feu Azure, une batterie de pare-feu ou WAF (pare-feu d’applications web).

Les différents métiers utilisent habituellement de nombreuses applications web. Ces applications ont tendance à comporter diverses vulnérabilités face aux attaques potentielles. Les pare-feu d’applications web sont un type de produit particulier qui permet de détecter les attaques contre les applications web (HTTP/HTTPS) de façon plus approfondie qu’un pare-feu générique. Contrairement à la technologie de pare-feu classique, les WAF intègrent un ensemble de fonctionnalités spécifiques pour protéger les serveurs web internes contre les menaces.

Toutefois, une seule implémentation de VDC doit être hébergée dans une seule région, car les conditions d’abonnement ne permettent pas de couvrir plusieurs régions. Cette exigence rend l’implémentation d’un seul VDC vulnérable aux pannes régionales.

Une instance de Pare-feu Azure ou des batteries de pare-feu NVA utilisent un plan d’administration commun. Un ensemble de règles de sécurité protège les charges de travail hébergées dans les rayons et contrôle l’accès aux réseaux locaux. Le Pare-feu Azure dispose d’une scalabilité intégrée, alors que les pare-feu NVA peuvent être manuellement mis à l’échelle derrière un équilibreur de charge. En règle générale, une batterie de pare-feu a moins de logiciels spécialisés que WAF. Mais elle dispose d’une étendue d’application plus large pour filtrer et inspecter tout type de trafic en sortie et en entrée. Si une approche NVA est appliquée, vous pouvez trouver et déployer les pare-feu à partir de la Place de Marché Azure.

Nous vous recommandons d’utiliser un ensemble d’instances de Pare-feu Azure, ou NVA, pour le trafic provenant d’Internet. Utilisez-en un autre pour le trafic local. L’utilisation d’un seul ensemble de pare-feu pour ces deux types de trafic réseau constitue un risque pour la sécurité, car elle n’offre aucun périmètre de sécurité entre les deux. L’utilisation de couches de pare-feu distinctes simplifie la vérification des règles de sécurité et permet d’identifier clairement les règles auxquelles correspondent les différentes requêtes réseau entrantes.

La plupart des grandes entreprises gèrent plusieurs domaines. Le service [Azure DNS][DNS] permet d’héberger les enregistrements DNS d’un domaine spécifique. Par exemple, vous pouvez inscrire la VIP (adresse IP virtuelle) de l’équilibreur de charge externe Azure, ou WAF, dans l’enregistrement **A** d’un enregistrement Azure DNS. Les [**DNS privés**][PrivateDNS] permettent également de gérer les espaces d’adressage privé au sein des réseaux virtuels.

[**Azure Load Balancer**][ALB] offre un service à haute disponibilité, de couche 4, TCP ou UDP. Il peut répartir le trafic entrant entre les instances de service définies dans un ensemble à charge équilibrée. Le trafic envoyé à l’équilibreur de charge à partir de points de terminaison front-end (adresses IP publiques ou privées) peut être redistribué avec ou sans traduction d’adresses à un ensemble d’un pool d’adresses IP back-end. Exemples : appliances virtuelles réseau ou machines virtuelles.

Azure Load Balancer peut également sonder l’intégrité des différentes instances de serveur. Quand une sonde ne répond pas, l’équilibreur de charge cesse d’envoyer du trafic à l’instance non saine. Dans le VDC, un équilibreur de charge externe du hub équilibre le trafic vers les NVA (appliances virtuelles réseau), par exemple. Dans les rayons, il effectue des tâches telles que l’équilibrage du trafic entre les différentes machines virtuelles d’une application multiniveau.

[**Azure Front Door**][AFD] est une plateforme d’accélération d’applications web, scalable et à haute disponibilité, intégrant un équilibreur de charge HTTP global, la protection d’application et le réseau de distribution de contenu. Front Door s’exécute dans plus de 100 sites en périphérie du réseau mondial de Microsoft. À l’aide de Front Door, vous pouvez créer et utiliser vos applications web dynamiques et votre contenu statique. De plus, vous pouvez effectuer un scale-out de ces derniers. Front Door offre à votre application les avantages suivants : 
* Niveau de performance de classe mondiale pour les utilisateurs finaux.
* Automation unifiée de la maintenance régionale ou par tampons.
* Automation BCDR.
* Informations client ou utilisateur unifiées. 
* Mise en cache. 
* Insights de services. La plateforme offre des SLA relatifs aux performances, à la fiabilité et au support, des certifications de conformité et des pratiques de sécurité vérifiables, dont le développement, la mise en œuvre et la prise en charge sont effectués de manière native par Azure.

[**Application Gateway**][AppGW] est une appliance virtuelle dédiée qui intègre le service ADC (Application Delivery Controller), offrant ainsi diverses fonctionnalités d’équilibrage de charge de couche 7 à votre application. Vous pouvez optimiser la productivité de la batterie de serveurs web en déchargeant l’arrêt SSL nécessitant de nombreuses ressources d’UC vers l’instance Application Gateway. Elle fournit également d’autres fonctionnalités de routage de couche 7, notamment : 
* Répartition en tourniquet (round robin) du trafic entrant. 
* L’affinité de session basée sur les cookies. 
* Routage basé sur le chemin de l’URL. 
* Possibilité d’héberger plusieurs sites web derrière une seule instance Application Gateway. Un WAF (pare-feu d’applications web) est également fourni dans le cadre de la référence WAF Application Gateway. Cette référence SKU protège les applications web contre les vulnérabilités et les codes malveillants exploitant une faille de sécurité les plus courants sur le web. Vous pouvez configurer Application Gateway en tant que passerelle accessible sur Internet, passerelle interne uniquement ou une combinaison des deux. 

[**Adresses IP publiques**][PIP]. Avec certaines fonctionnalités Azure, vous pouvez associer des points de terminaison de service à une adresse IP publique, pour que votre ressource soit accessible à partir d’Internet. Ce point de terminaison utilise NAT (traduction d’adresses réseau) pour router le trafic vers l’adresse et le port internes sur le réseau virtuel Azure. Il s’agit du principal chemin d’accès pour que le trafic externe passe dans le réseau virtuel. Vous pouvez configurer des adresses IP publiques pour déterminer le type de trafic passé ainsi que la façon dont la traduction s’opère sur le réseau virtuel et son emplacement précis.

Le service [**Azure DDoS Protection standard**][DDOS] fournit des fonctionnalités d’atténuation supplémentaires par rapport au niveau de [service De base][DDOS] destinées spécifiquement aux ressources de réseau virtuel Azure. Le service DDoS Protection Standard est facile à activer et ne nécessite aucun changement de l’application. Les stratégies de protection sont paramétrées par le biais d’algorithmes de surveillance du trafic et d’apprentissage automatique dédiés. Ces stratégies sont appliquées aux adresses IP publiques associées aux ressources déployées sur des réseaux virtuels. Exemples : instances Azure Load Balancer, Azure Application Gateway et Azure Service Fabric. Les données de télémétrie en temps réel sont disponibles par le biais d’affichages Azure Monitor pendant une attaque et à des fins d’historique. Vous pouvez ajouter une protection de la couche Application via le pare-feu d’applications web Azure Application Gateway. La protection est assurée pour les adresses IP publiques IPv4 Azure.

#### <a name="component-type-monitoring"></a>Type de composant : surveillance
Les composants de surveillance offrent une vue d’ensemble de tous les autres types de composants et génèrent des alertes concernant ces derniers. Toutes les équipes doivent avoir accès à la surveillance des composants et services qui leur sont accessibles. Si vous disposez d’équipes des opérations ou d’un support technique centralisé, ils doivent disposer d’un accès intégré aux données fournies par ces composants.

Azure propose différents types de service de journalisation et de supervision pour effectuer le suivi du comportement des ressources hébergées par Azure. La gouvernance et le contrôle des charges de travail dans Azure reposent non seulement sur la collecte des données de journalisation, mais également sur le déclenchement d’actions en fonction d’événements signalés spécifiques.

[**Azure Monitor**][Monitor]. Azure comprend plusieurs services qui effectuent individuellement un rôle ou une tâche spécifique dans l’espace d’analyse. Ensemble, ces services fournissent une solution complète pour la collecte, l’analyse et l’action sur les données de télémétrie de votre application et des ressources Azure qui les prennent en charge. Ces services peuvent aussi surveiller les ressources locales critiques afin de fournir un environnement de surveillance hybride. Comprendre les outils et les données disponibles est la première étape du développement d’une stratégie de surveillance complète pour votre application.

Il existe deux principaux types de journaux dans Azure :

-   Le [Journal d’activité Azure][ActLog], dont l’ancienne appellation correspondait à **journaux des opérations**, fournit un insight des opérations effectuées sur les ressources de l’abonnement Azure. Ces journaux signalent les événements de plan de contrôle relatifs à vos abonnements. Chaque ressource Azure génère des journaux d’audit.

-   Les [journaux de diagnostic Azure Monitor][DiagLog] sont des journaux générés par une ressource qui fournit des données fréquentes et détaillées sur le fonctionnement de cette ressource. Le contenu de ces journaux varie en fonction du type de ressource.

[![9]][9]

Dans le VDC, il est important de suivre les journaux NSG, en particulier les informations suivantes :

-   Les [journaux des événements][NSGLog] fournissent des informations sur les règles NSG appliquées aux machines virtuelles et aux rôles d’instance en fonction de l’adresse MAC.
-   Les [journaux de compteur][NSGLog] indiquent le nombre d’exécutions de chaque règle NSG pour refuser ou autoriser le trafic.

Tous les journaux peuvent être stockés dans des comptes de stockage Azure à des fins d’audit, d’analyse statique ou de sauvegarde. Quand vous stockez les journaux dans un compte de stockage Azure, les clients peuvent utiliser différents types de framework pour récupérer, préparer, analyser et visualiser ces données afin de signaler l’état et l’intégrité des ressources cloud. 

Les grandes entreprises doivent avoir acquis au préalable un framework standard pour la supervision des systèmes locaux. Ils peuvent étendre ce framework pour intégrer les journaux générés par les déploiements cloud. À l’aide d’[Azure Log Analytics][https://docs.microsoft.com/en-us/azure/log-analytics/log-analytics-queries], les organisations peuvent conserver l’ensemble de la journalisation dans le cloud. Log Analytics est implémenté en tant que service cloud. Il peut donc être opérationnel rapidement, avec un investissement minimal dans les services d’infrastructure. Log Analytics s’intègre également aux composants System Center, tels que System Center Operations Manager, pour étendre au cloud vos investissements de gestion existants. 

Log Analytics est un service dans Azure conçu pour faciliter la collecte, la mise en corrélation, la recherche et l’exploitation des données de journalisation et de performances générées par les systèmes d’exploitation, les applications et les composants cloud d’infrastructure. Il fournit aux clients des insights opérationnels en temps réel à l’aide de la recherche intégrée et de tableaux de bord personnalisés pour analyser tous les enregistrements de l’ensemble de vos charges de travail dans le VDC.

[Azure Network Watcher][NetWatch] fournit des outils permettant d’effectuer une supervision et des diagnostics, d’afficher les métriques et d’activer ou de désactiver les journaux des ressources d’un réseau virtuel Azure. Il s’agit d’un service multifacette qui permet d’accéder aux fonctionnalités suivantes et à bien d’autres encore :
-    Superviser la communication entre une machine virtuelle et un point de terminaison
-    Afficher les ressources se trouvant sur un réseau virtuel et leurs relations
-    Diagnostiquer les problèmes de filtrage du trafic réseau à destination ou en provenance d’une machine virtuelle
-    Diagnostiquer les problèmes de routage réseau d’une machine virtuelle
-    Diagnostiquer les connexions sortantes d’une machine virtuelle
-    Capturer les paquets à destination et en provenance d’une machine virtuelle
-    Diagnostiquer les problèmes liés à une passerelle de réseau virtuel Azure et aux connexions
-    Déterminer les latences relatives entre les régions Azure et les fournisseurs de services Internet
-    Afficher les règles de sécurité d’une interface réseau
-    Afficher les métriques réseau
-    Analyser le trafic à destination ou en provenance d’un groupe de sécurité réseau
-    Afficher les journaux de diagnostic des ressources réseau

La solution [Network Performance Monitor][NPM] dans Operations Management Suite peut fournir des informations détaillées sur le réseau, de bout en bout. Ces informations incluent une seule vue de vos réseaux Azure et de vos réseaux locaux. La solution dispose de fonctionnalités de supervision spécifiques pour ExpressRoute et les services publics.

#### <a name="component-type-workloads"></a>Type de composant : charges de travail
Les composants de type charge de travail désignent l’emplacement où résident vos applications et services proprement dits. Il s’agit également des composants auxquels vos équipes de développement d’applications consacrent la majorité de leur temps.

Les possibilités de charge de travail sont illimitées. Voici quelques-uns des innombrables types de charges de travail possibles :

Les **applications métier internes** sont des applications informatiques essentielles au fonctionnement continu d’une entreprise. Les applications métier ont certaines caractéristiques communes :

-   Elles sont **interactives** par nature. Une fois que vous avez entré des données, des résultats ou des rapports sont retournés.
-   Elles sont **pilotées par les données**&mdash;Elles utilisent les données de manière intensive et accèdent fréquemment aux bases de données ou à d’autres types de stockage.
-   Elles sont **intégrées**&mdash;Elles assurent une intégration avec d’autres systèmes au sein ou en dehors de l’organisation.

**Sites web destinés aux clients (accessibles par Internet ou en interne)**. La plupart des applications qui interagissent avec Internet sont des sites web. Azure permet d’exécuter un site web sur une machine virtuelle IaaS ou à partir d’un site [Web Apps avec Microsoft Azure App Service][WebApps] (PaaS). Web Apps prend en charge l’intégration avec les réseaux virtuels qui permettent le déploiement de Web Apps dans le rayon du VDC. Quand vous consultez des sites web internes, avec l’intégration de réseau virtuel, vous n’avez pas besoin d’exposer un point de terminaison Internet pour vos applications. Toutefois, à la place, vous pouvez utiliser les ressources via des adresses routables non Internet privées à partir de votre réseau virtuel privé.

**Big Data et analytique**. En cas de scale-up lié à un volume de données très important, il arrive que le scale-up des bases de données ne s’effectue pas correctement. La technologie Apache Hadoop offre un système permettant d’exécuter des requêtes distribuées en parallèle sur un grand nombre de nœuds. Les clients peuvent exécuter des charges de travail de données dans des machines virtuelles IaaS ou PaaS, [Azure HDInsight][HDI]. HDInsight prend en charge le déploiement sur un réseau virtuel en fonction de l’emplacement. Il peut être déployé sur un cluster d’un rayon du VDC.

**Événements et messagerie**. [Azure Event Hubs][EventHubs] est un service d’ingestion de télémétrie hyperscale qui collecte, transforme et stocke des millions d’événements. En tant que plateforme de streaming distribuée, il offre une faible latence et un délai de conservation configurable. Vous pouvez donc ingérer des quantités massives de télémétrie dans Azure et lire ces données à partir de plusieurs applications. Le service Event Hubs prend en charge le traitement de pipelines en temps réel et par lots sur le même flux.

Vous pouvez implémenter un service de messagerie cloud à haut niveau de fiabilité entre applications et services via [Azure Service Bus][ServiceBus]. Il offre une messagerie répartie asynchrone entre le client et le serveur, une messagerie FIFO (premier entré, premier sorti) structurée ainsi que des fonctionnalités de publication et d’abonnement.

[![10]][10]

### <a name="multiple-vdc-implementations"></a>Implémentations de VDC multiples


Un seul VDC est hébergé dans une seule région. Il est vulnérable face à une panne majeure susceptible d’affecter toute la région. Les clients qui souhaitent bénéficier de contrats SLA de haut niveau doivent protéger leurs services en déployant le même projet dans au moins deux implémentations de VDC, placées dans des régions distinctes.

En plus de la question des contrats SLA, il existe divers scénarios courants dans lesquels il est judicieux de déployer plusieurs implémentations de VDC :

-   Présence régionale ou mondiale
-   Récupération d’urgence.
-   Mécanisme de redirection du trafic entre des centres de données

#### <a name="regional-and-global-presence"></a>Présence régionale et mondiale
Les centres de données Azure sont présents dans de nombreuses régions du monde. Quand les clients sélectionnent plusieurs centres de données Azure, ils doivent prendre en compte deux facteurs associés : les distances géographiques et la latence. Pour offrir la meilleure expérience utilisateur possible, les clients doivent évaluer la distance géographique entre les implémentations de VDC et la distance entre les implémentations de VDC et les utilisateurs finaux.

La région Azure où sont hébergées les implémentations de VDC doit également respecter les exigences réglementaires établies par la juridiction dont relève votre organisation.

#### <a name="disaster-recovery"></a>Récupération d'urgence
L’implémentation d’un plan de reprise d’activité est étroitement liée au type de charge de travail concernée ainsi qu’à la synchronisation de l’état de la charge de travail entre différentes implémentations de VDC. Dans l’idéal, la plupart des clients souhaitent synchroniser les données d’application entre des déploiements qui s’exécutent dans deux implémentations de VDC distinctes pour implémenter un mécanisme de basculement rapide. La plupart des applications sont sensibles à la latence, ce qui peut entraîner des délais d’expiration et des retards dans la synchronisation des données.

La synchronisation ou la supervision des pulsations des applications dans différentes implémentations de VDC nécessite une communication entre ces dernières. Deux implémentations de VDC dans des régions distinctes peuvent être connectées comme suit :

-   Appairage de réseaux virtuels permettant de connecter des hubs dans différentes régions
-   Appairage privé ExpressRoute quand les hubs de VDC sont connectés au même circuit ExpressRoute
-   Circuits ExpressRoute multiples connectés via votre cœur de réseau (backbone) d’entreprise et votre maillage de VDC connecté aux circuits ExpressRoute
-   Connexions VPN site à site entre vos hubs VDC dans chaque région Azure

En règle générale, les connexions par appairage de réseaux virtuels ou les connexions ExpressRoute constituent le mécanisme privilégié, car la bande passante est plus importante et la latence constante durant le transit par le cœur de réseau Microsoft.

Il n’existe aucune formule magique pour valider une application distribuée entre deux ou plusieurs implémentations de VDC différentes situées dans des régions distinctes. Les clients doivent exécuter des tests de qualification du réseau pour vérifier la latence et la bande passante des connexions. Ils doivent ensuite déterminer si la réplication synchrone ou asynchrone des données est appropriée ainsi que l’objectif de délai de récupération (RTO) optimal pour les charges de travail.

#### <a name="mechanism-to-divert-traffic-between-datacenters"></a>Mécanisme de redirection du trafic entre des centres de données
Une technique efficace permettant de rediriger le trafic entrant dans un centre de données vers un autre centre de données repose sur le DNS (Domain Name System). [Azure Traffic Manager][TM] utilise le mécanisme DNS pour diriger le trafic de l’utilisateur final vers le point de terminaison public le plus approprié dans un VDC spécifique. À l’aide de sondes, Traffic Manager vérifie périodiquement l’intégrité du service des points de terminaison publics dans différentes implémentations de VDC. En cas d’échec de ces points de terminaison, il effectue un routage automatique vers le VDC secondaire.

Traffic Manager fonctionne sur les points de terminaison publics Azure. Par exemple, il peut contrôler ou rediriger le trafic vers les machines virtuelles Azure et Web Apps dans le VDC approprié. Traffic Manager est résilient même en cas de défaillance d’une région Azure entière. Il peut contrôler la distribution du trafic utilisateur pour les points de terminaison de service dans différentes implémentations de VDC en fonction de plusieurs critères. Le cas peut se présenter, par exemple, pour répondre à une défaillance de service dans un VDC spécifique ou afin de sélectionner un VDC ayant la latence réseau la plus faible pour le client.

### <a name="conclusion"></a>Conclusion
Le centre de données virtuel est une approche relative à la migration d’un centre de données vers le cloud combinant plusieurs fonctionnalités et capacités pour créer une architecture scalable dans Azure. Il optimise l’utilisation des ressources dans le cloud, réduit les coûts et simplifie la gouvernance du système. Le concept de VDC est basé sur une topologie hub-and-spoke qui fournit des services partagés communs dans le hub. Il permet l’utilisation d’applications ou de charges de travail spécifiques dans les rayons. Le VDC correspond à la structure des rôles d’une entreprise, où différents services sont associés à un travail commun. Exemples : service informatique central, Azure DevOps, opérations et maintenance. Chaque service a une liste spécifique de rôles et de tâches. Le concept de VDC répond aux exigences d’une migration **lift-and-shift**. Toutefois, il offre également de nombreux avantages pour les déploiements cloud natifs.

## <a name="references"></a>Références
Découvrez plus en détail les fonctionnalités suivantes présentées dans cet article :

| | | |
|-|-|-|
|Fonctionnalités réseau|Équilibrage de la charge|Connectivité|
|[Réseau virtuel Azure][VNet]</br>[Groupes de sécurité réseau][NSG]</br>[Journaux NSG][NSGLog]</br>[Routes définies par l’utilisateur][UDR]</br>[Appliances virtuelles réseau][NVA]</br>[Adresses IP publiques][PIP]</br>[Azure DDoS Protection][DDOS]</br>[Pare-feu Azure][AzFW]</br>[Azure DNS][DNS]|[Azure Front Door][AFD]</br>[Azure Load Balancer (L3)][ALB]</br>[Application Gateway (L7)][AppGW]</br>[Pare-feu d’applications web][WAF]</br>[Azure Traffic Manager][TM]</br></br></br></br></br> |[Appairage de réseaux virtuels][VNetPeering]</br>[Réseau privé virtuel][VPN]</br>[Virtual WAN][vWAN]</br>[ExpressRoute][ExR]</br>[ExpressRoute Direct][ExRD]</br></br></br></br></br>
|Identité</br>|Surveillance</br>|Meilleures pratiques</br>|
|[Azure Active Directory][AAD]</br>[Multi-Factor Authentication][MFA]</br>[Contrôle d’accès en fonction du rôle][RBAC]</br>[Rôles Azure AD par défaut][Roles]</br></br></br> |[Network Watcher][NetWatch]</br>[Azure Monitor][Monitor]</br>[Journal d’activité][ActLog]</br>[Superviser les journaux de diagnostics][DiagLog]</br>[Microsoft Operations Management Suite][OMS]</br>[Network Performance Monitor][NPM]|[Bonnes pratiques du réseau de périmètre][DMZ]</br>[Gestion des abonnements][SubMgmt]</br>[Gestion des groupes de ressources][RGMgmt]</br>[Limites d’abonnement Azure][Limits] </br></br></br>|
|Autres services Azure|
|[Web Apps][WebApps]</br>[HDInsight (Hadoop)][HDI]</br>[Event Hubs][EventHubs]</br>[Service Bus][ServiceBus]|

## <a name="next-steps"></a>Étapes suivantes
 - Explorez l’[appairage de réseaux virtuels][VNetPeering], la technologie sous-jacente des topologies hub-and-spoke de VDC.
 - Implémentez [Azure AD][AAD] pour bien démarrer avec l’exploration de [RBAC][RBAC].
 - Développez un modèle de gestion des abonnements et des ressources ainsi qu’un modèle RBAC pour répondre à la structure, aux exigences et aux stratégies de votre organisation. L’activité la plus importante est la planification. Dans la mesure du possible, planifiez les réorganisations, les fusions, les nouvelles gammes de produits et les autres éventualités.

<!--Image References-->
[0]: ./images/networking-redundant-equipment.png "Exemples de chevauchement de composants" 
[1]: ./images/networking-vdc-high-level.png "Exemple de haut niveau de VDC hub-and-spoke (réseau en étoile)"
[2]: ./images/networking-hub-spokes-cluster.png "Cluster de concentrateurs et de rayons"
[3]: ./images/networking-spoke-to-spoke.png "Rayon à rayon"
[4]: ./images/networking-vdc-block-level-diagram.png "Diagramme de niveau bloc du VDC"
[5]: ./images/networking-users-groups-subsciptions.png "Utilisateurs, groupes, abonnements et projets"
[6]: ./images/networking-infrastructure-high-level.png "Diagramme d’infrastructure de haut niveau"
[7]: ./images/networking-highlevel-perimeter-networks.png "Diagramme d’infrastructure de haut niveau"
[8]: ./images/networking-vnet-peering-perimeter-neworks.png "Homologation de réseaux virtuels et réseaux de périmètre"
[9]: ./images/networking-high-level-diagram-monitoring.png "Diagramme de surveillance de haut niveau"
[10]: ./images/networking-high-level-workloads.png "Diagramme de charge de travail de haut niveau"

<!--Link References-->
[Limits]: /azure/azure-subscription-service-limits
[Roles]: /azure/role-based-access-control/built-in-roles
[VNet]: /azure/virtual-network/virtual-networks-overview
[NSG]: /azure/virtual-network/virtual-networks-nsg
[DNS]: /azure/dns/dns-overview
[PrivateDNS]: /azure/dns/private-dns-overview
[VNetPeering]: /azure/virtual-network/virtual-network-peering-overview 
[UDR]: /azure/virtual-network/virtual-networks-udr-overview 
[RBAC]: /azure/role-based-access-control/overview
[MFA]: /azure/multi-factor-authentication/multi-factor-authentication
[AAD]: /azure/active-directory/active-directory-whatis
[VPN]: /azure/vpn-gateway/vpn-gateway-about-vpngateways 
[ExR]: /azure/expressroute/expressroute-introduction
[ExRD]: https://docs.microsoft.com/en-us/azure/expressroute/expressroute-erdirect-about
[vWAN]: /azure/virtual-wan/virtual-wan-about
[NVA]: /azure/architecture/reference-architectures/dmz/nva-ha
[AzFW]: /azure/firewall/overview
[SubMgmt]: /azure/architecture/cloud-adoption/appendix/azure-scaffold 
[RGMgmt]: /azure/azure-resource-manager/resource-group-overview
[DMZ]: /azure/best-practices-network-security
[ALB]: /azure/load-balancer/load-balancer-overview
[DDOS]: /azure/virtual-network/ddos-protection-overview
[PIP]: /azure/virtual-network/resource-groups-networking#public-ip-address
[AFD]: https://docs.microsoft.com/en-us/azure/frontdoor/front-door-overview
[AppGW]: /azure/application-gateway/application-gateway-introduction
[WAF]: /azure/application-gateway/application-gateway-web-application-firewall-overview
[Monitor]: /azure/monitoring-and-diagnostics/
[ActLog]: /azure/monitoring-and-diagnostics/monitoring-overview-activity-logs 
[DiagLog]: /azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs
[NSGLog]: /azure/virtual-network/virtual-network-nsg-manage-log
[OMS]: /azure/operations-management-suite/operations-management-suite-overview
[NPM]: /azure/log-analytics/log-analytics-network-performance-monitor
[NetWatch]: /azure/network-watcher/network-watcher-monitoring-overview
[WebApps]: /azure/app-service/
[HDI]: /azure/hdinsight/hdinsight-hadoop-introduction
[EventHubs]: /azure/event-hubs/event-hubs-what-is-event-hubs 
[ServiceBus]: /azure/service-bus-messaging/service-bus-messaging-overview
[TM]: /azure/traffic-manager/traffic-manager-overview

