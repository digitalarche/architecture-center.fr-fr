---
title: Centre de données virtuel Azure - Une perspective réseau
description: Découvrez comment créer votre centre de données virtuel dans Azure
author: tracsman
manager: rossort
tags: azure-resource-manager
ms.service: virtual-network
ms.date: 04/3/2018
ms.author: jonor
ms.openlocfilehash: 8df02bbb6d926f0ab5402fab4eb2b25fdc2057cd
ms.sourcegitcommit: 8ec48a0e2c080c9e2e0abbfdbc463622b28de2f2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/18/2018
ms.locfileid: "40211570"
---
# <a name="azure-virtual-datacenter-a-network-perspective"></a>Centre de données virtuel Azure : une perspective réseau

## <a name="overview"></a>Vue d’ensemble
La migration d’applications locales vers Azure, même dépourvue de modifications majeures (approche « lift-and-shift »), permet aux organisations de bénéficier d’une infrastructure sécurisée et rentable. Toutefois, pour tirer pleinement profit de l’agilité inhérente au cloud computing, les entreprises doivent faire évoluer leurs architectures de façon à exploiter au mieux les capacités d’Azure. Microsoft Azure offre des services et une infrastructure hyperscale, des capacités et une fiabilité de qualité professionnelle, ainsi que de nombreuses options de connectivité hybride. Les clients peuvent choisir d’accéder à ces services cloud par le biais d’Internet ou avec Azure ExpressRoute, qui fournit une connectivité réseau privée. La plateforme Microsoft Azure permet aux clients d’étendre leur infrastructure dans le cloud et de générer des architectures à plusieurs niveaux en toute transparence. En outre, les partenaires Microsoft offrent des fonctionnalités améliorées en proposant des services de sécurité et des appliances virtuelles qui sont optimisés pour s’exécuter dans Azure.

Cet article fournit une vue d’ensemble des modèles et des conceptions qui permettent de balayer les inquiétudes de nombreux clients concernant la mise à l’échelle, les performances et la sécurité de l’architecture lorsque ces clients envisagent une migration massive vers le cloud. Cet article décrit également comment adapter les différents rôles informatiques d’une organisation à la gestion et à la gouvernance du système, en mettant l’accent sur les exigences liées à la sécurité et sur l’optimisation des coûts.

## <a name="what-is-a-virtual-data-center"></a>Qu’est-ce qu’un centre de données virtuel ?
À l’origine, les solutions cloud étaient conçues pour héberger des applications uniques et relativement isolées dans le domaine public. Cette approche a donné de bons résultats pendant quelques années. Toutefois, à mesure que les avantages offerts par les solutions cloud sont devenus évidents et que de multiples charges de travail à grande échelle ont été hébergées sur le cloud, il s’est révélé crucial de résoudre les problèmes de sécurité, de fiabilité, de performances et de coûts des déploiements rencontrés dans une ou plusieurs régions sur l’ensemble du cycle de vie du service cloud.

Le diagramme de déploiement cloud ci-après illustre certains exemples de failles de sécurité (encadré rouge) et de possibilités d’optimisation des appliances virtuelles réseau entre les charges de travail (encadré orange).

[![0]][0]

Le centre de données virtuel (VDC) est né de cette nécessité de procéder à une mise à l’échelle pour gérer les charges de travail des entreprises, ainsi que de l’obligation de faire face aux problèmes induits par la prise en charge d’applications à grande échelle dans le cloud public.

Un VDC ne se résume pas simplement aux charges de travail d’application dans le cloud, mais englobe également le réseau, la sécurité, la gestion et l’infrastructure (par exemple, le DNS et les services d’annuaire). En outre, il fournit généralement une connexion privée à un réseau ou un centre de données locaux. À mesure que le volume de charges de travail déplacées vers Azure ne cesse de croître, il est important de réfléchir à l’infrastructure sous-jacente et aux objets dans lesquels ces charges de travail sont placées. Il convient d’envisager avec soin le mode de structuration des ressources afin d’éviter la prolifération de centaines d’« îlots de charge de travail » qui doivent être gérés séparément avec des flux de données, des modèles de sécurité et des problèmes de conformité indépendants.

Un centre de données virtuel constitue essentiellement une série d’entités distinctes mais associées qui présentent une infrastructure, des fonctionnalités et des capacités sous-jacentes communes. La visualisation de vos charges de travail sous la forme d’un VDC intégré vous permet d’alléger les coûts grâce aux économies d’échelle, à l’optimisation de la sécurité par le biais de la centralisation des composants et des flux de données, ainsi qu’à la simplification des opérations, de la gestion et des audits de conformité.

> [!NOTE]
> Il est important de comprendre que le VDC n’est **PAS** un produit Azure distinct, mais la combinaison de différentes fonctionnalités et capacités en vue de répondre précisément à vos exigences. Le VDC est un moyen d’envisager vos charges de travail et l’utilisation d’Azure pour optimiser vos ressources et capacités dans le cloud. Le centre de données virtuel constitue donc une approche modulaire de la procédure de création de services informatiques dans Azure, en respectant les rôles et responsabilités de l’organisation.

Le VDC peut aider les entreprises à déplacer des charges de travail et des applications vers Azure dans le cadre des scénarios suivants :

-   Hébergement de plusieurs charges de travail associées
-   Migration de charges de travail d’un environnement local vers Azure
-   Implémentation d’exigences de gestion partagée ou centralisée de la sécurité et des accès dans l’ensemble des charges de travail
-   Combinaison appropriée de DevOps et d’un service informatique centralisé pour une grande entreprise

La méthode clé pour déverrouiller les avantages d’un VDC consiste à utiliser une topologie centralisée (hub and spokes) avec un mélange des fonctionnalités suivantes : [réseau virtuel Azure][VNet], [groupes de sécurité réseau (NSG)][NSG], [homologation de réseaux virtuels][VNetPeering], [itinéraires définis par l’utilisateur (UDR)][UDR] et Azure Identity avec [contrôle d’accès en fonction du rôle (RBAC)][RBAC].

## <a name="who-needs-a-virtual-data-center"></a>Qui doit utiliser un centre de données virtuel ?
Tout client Azure ayant besoin de déplacer plus de deux charges de travail vers Azure peut tirer profit de l’utilisation de ressources communes. Selon l’ampleur du projet, même de simples applications peuvent bénéficier de l’exploitation des modèles et des composants utilisés pour générer un VDC.

Si votre organisation dispose d’une équipe/un département informatique, réseau, sécurité et/ou conformité centralisés, un VDC peut faciliter l’application des points de stratégie, la répartition des tâches et garantir l’uniformité des composants communs sous-jacents tout en offrant aux équipes d’application le degré de liberté et de contrôle adapté à vos exigences.

Les organisations qui se tournent vers DevOps peuvent utiliser les concepts VDC pour fournir des poches autorisées de ressources Azure et offrir l’assurance qu’elles disposent d’un contrôle total au sein de ce groupe (abonnement ou groupe de ressources dans un abonnement commun), tandis que les limites de réseau et de sécurité restent conformes aux définitions d’une stratégie centralisée dans un groupe de ressources et un réseau virtuel concentrateur.

## <a name="considerations-on-implementing-a-virtual-data-center"></a>Considérations en matière d’implémentation d’un centre de données virtuel
Lorsque vous concevez un VDC, vous devez prendre en compte plusieurs aspects cruciaux :

-   Services d’identité et d’annuaire
-   Infrastructure de sécurité
-   Connectivité avec le cloud
-   Connectivité au sein du cloud

### <a name="identity-and-directory-service"></a>Services d’identité et d’annuaire
Les services d’identité et d’annuaire constituent un aspect clé de tous les centres de données, à la fois au niveau local et dans le cloud. La notion d’identité est liée à tous les aspects des accès et des autorisations vis-à-vis des services au sein du VDC. Pour garantir que seuls les utilisateurs et processus autorisés auront accès à votre compte et à vos ressources Azure, Azure utilise plusieurs types d’informations d’identification pour l’authentification. Ces informations regroupent des mots de passe (pour l’accès au compte Azure), des clés de chiffrement, des signatures numériques et des certificats. [*Azure Multi-Factor Authentication* (MFA)][MFA] est une couche de sécurité supplémentaire pour l’accès aux services Azure. Azure MFA assure une authentification forte par le biais d’une série d’options de vérification simples (appel téléphonique, SMS ou notification d’application mobile) qui permet aux clients de choisir leur méthode préférée.

Toute grande entreprise doit définir un processus de gestion d’identité qui décrit la gestion des différentes identités, leur authentification, ainsi que les autorisations, rôles et privilèges dans le VDC. Ce processus doit avoir pour fonction d’améliorer la sécurité et la productivité tout en minimisant les coûts, les temps d’arrêt et les tâches manuelles répétitives.

Les entreprises/organisations peuvent nécessiter une combinaison de services complexe pour différents cœurs de métier, et les employés assument fréquemment plusieurs rôles lorsqu’ils sont impliqués dans divers projets. Un VDC requiert une coopération satisfaisante entre les différentes équipes, chacune dotée d’un rôle spécifique, afin de garantir le bon fonctionnement des systèmes avec une gouvernance adéquate. La matrice des responsabilités, des accès et des droits peut se révéler d’une grande complexité. La gestion d’identité dans le VDC est implémentée par le biais d’[*Azure Active Directory* (AAD)][AAD] et du contrôle d’accès en fonction du rôle (RBAC).

Un service d’annuaire est une infrastructure d’informations partagée pour la localisation, la gestion, l’administration et l’organisation des éléments et ressources réseau utilisés quotidiennement. Ces ressources peuvent inclure des volumes, des dossiers, des fichiers, des imprimantes, des utilisateurs, des groupes, des appareils et d’autres objets. Chacune des ressources du réseau est considérée comme un objet par le serveur d’annuaire. Les informations concernant une ressource sont stockées sous la forme d’une collection d’attributs associée à cette ressource ou à cet objet.

Tous les services métiers en ligne Microsoft reposent sur Azure Active Directory (AAD) pour la connexion et les autres besoins en matière d’identification. Azure Active Directory est une solution cloud complète et hautement disponible de gestion des identités et des accès qui associe des services d'annuaire principaux, une gouvernance avancée des identités et la gestion de l'accès aux applications. Il est possible d’intégrer AAD au service Active Directory local afin d’autoriser l’authentification unique pour toutes les applications basées sur le cloud et hébergées localement. Les attributs utilisateur du service Active Directory local peuvent être automatiquement synchronisés avec AAD.

Il n’est pas nécessaire qu’un administrateur général unique attribue toutes les autorisations dans un VDC. À la place, chaque département spécifique (ou groupe d’utilisateurs ou de services dans le service d’annuaire) peut disposer des autorisations requises pour gérer ses propres ressources dans un VDC. La structuration des autorisations doit être équilibrée. En effet, un nombre d’autorisations excessif risque de compromettre les performances, tandis que des autorisations insuffisantes ou trop flexibles sont susceptibles d’accroître les risques pour la sécurité. Le contrôle d’accès en fonction du rôle Azure (RBAC) contribue à résoudre ce problème en offrant une gestion affinée des accès aux ressources du VDC.

#### <a name="security-infrastructure"></a>Infrastructure de sécurité
Dans le contexte d’un VDC, l’infrastructure de sécurité est essentiellement liée à la répartition du trafic dans le segment de réseau virtuel spécifique du VDC, ainsi qu’au mode de contrôle des flux d’entrée et de sortie dans l’ensemble du VDC. Azure repose sur une architecture mutualisée qui empêche le trafic non autorisé et accidentel entre les déploiements, par le biais de l’isolement du réseau virtuel (VNet), d’ACL, d’équilibreurs de charge et de filtres IP, ainsi que de stratégies de flux de trafic. La traduction d’adresses réseau (NAT) sépare le trafic réseau interne du trafic externe.

La structure Azure alloue des ressources d’infrastructure aux charges de travail des locataires et gère les communications à destination et en provenance des machines virtuelles. L’hyperviseur Azure applique la séparation de mémoire et de processus entre les machines virtuelles, et route en toute sécurité le trafic réseau vers les locataires du système d’exploitation invité.

#### <a name="connectivity-to-the-cloud"></a>Connectivité avec le cloud
Le VDC nécessite une connectivité avec les réseaux externes pour offrir des services aux clients, aux partenaires et/ou aux utilisateurs internes. Généralement, cela implique une connexion non seulement à Internet, mais également aux réseaux et aux centres de données locaux.

Les clients peuvent élaborer leurs stratégies de sécurité de façon à déterminer quels services spécifiques hébergés du VDC sont accessibles et comment depuis Internet au moyen d’appliances virtuelles réseau (avec un filtrage et une inspection du trafic), ainsi qu’à l’aide de stratégies de routage personnalisées et d’un filtrage réseau (routage défini par l’utilisateur et les groupes de sécurité réseau).

Les entreprises doivent fréquemment connecter les VDC à des centres de données locaux ou à d’autres ressources. La connectivité entre Azure et les réseaux locaux constitue donc un aspect essentiel de la conception d’une architecture efficace. Pour créer une interconnexion entre un VDC et des réseaux locaux dans Azure, les entreprises disposent de deux méthodes : transit par le biais d’Internet et/ou par des connexions directes privées.

Un [**réseau privé virtuel de site à site Azure**][VPN] est un service d’interconnexion sur Internet entre les réseaux locaux et le VDC, établi par l’intermédiaire de connexions chiffrées sécurisées (tunnels IPsec/IKE). La connexion site à site Azure est flexible et facile à créer et ne nécessite aucun approvisionnement supplémentaire, car toutes les connexions se connectent par le biais d’internet.

[**ExpressRoute**][ExR] est un service de connectivité Azure qui vous permet de créer des connexions privées entre le VDC et les réseaux locaux. Les connexions ExpressRoute ne sont pas établies par l’intermédiaire du réseau public Internet et offrent un surcroît de sécurité et de fiabilité et des débits supérieurs (jusqu’à 10 Gbits/s), ainsi qu’une latence constante. Le service ExpressRoute se révèle d’une grande utilité pour les VDC, car les clients ExpressRoute peuvent bénéficier des règles de conformité associées aux connexions privées.

Le déploiement de connexions ExpressRoute implique la souscription d’un engagement auprès d’un fournisseur de services ExpressRoute. Les clients qui doivent être opérationnels rapidement commencent généralement par utiliser un réseau privé virtuel de site à site pour établir la connectivité entre le VDC et les ressources locales, puis effectuent une migration vers une connexion ExpressRoute.

#### <a name="connectivity-within-the-cloud"></a>*Connectivité au sein du cloud*
Les services de [réseaux virtuels][VNet] et d’[homologation de réseaux virtuels][VNetPeering] sont les services de connectivité réseau de base au sein d’un VDC. Un réseau virtuel garantit une limite d’isolement naturelle pour les ressources du VDC, et l’homologation des réseaux virtuels permet l’intercommunication entre différents réseaux virtuels au sein de la même région Azure ou même de plusieurs régions. Le contrôle du trafic à l’intérieur d’un réseau virtuel et entre plusieurs réseaux virtuels doit correspondre à un ensemble de règles de sécurité spécifiées par le biais d’ACL ([Groupe de sécurité réseau][NSG]), d’[appliances virtuelles réseau][NVA] et de tables de routage personnalisées ([itinéraires définis par l’utilisateur][UDR]).

## <a name="virtual-data-center-overview"></a>Vue d’ensemble d’un centre de données virtuel

### <a name="topology"></a>Topologie
_Hub-and-spoke_ (réseau en étoile) est un modèle permettant l’extension d’un centre de données virtuel au sein d’une seule région Azure.

[![1]][1]

Le concentrateur (hub) est la zone centrale qui contrôle et inspecte le trafic d’entrée et/ou de sortie entre différentes zones : Internet, le niveau local et les rayons (spokes). La topologie hub-and-spoke offre au département informatique un moyen efficace d’appliquer des stratégies de sécurité à un emplacement central, tout en réduisant les risques de configuration incorrecte et d’exposition.

Le concentrateur contient les composants de service communs consommés par les rayons. Voici quelques exemples classiques de services centraux communs :

-   Infrastructure Windows Active Directory (avec le service AD FS associé) requise pour l’authentification utilisateur des tiers qui veulent accéder aux charges de travail du rayon à partir de réseaux non approuvés
-   Service DNS pour la résolution de l’attribution de nom concernant la charge de travail dans les rayons, afin d’accéder aux ressources locales et sur Internet
-   Infrastructure à clé publique (PKI) pour implémenter l’authentification unique sur les charges de travail
-   Contrôle de flux (TCP/UDP) entre les rayons et Internet
-   Contrôle de flux entre le rayon et les ressources locales
-   Le cas échéant, contrôle de flux entre deux rayons

Le VDC réduit le coût global au moyen de l’infrastructure de concentrateur partagée entre plusieurs rayons.

Le rôle de chaque rayon peut consister à héberger différents types de charges de travail. Les rayons peuvent également offrir une approche modulaire pour les déploiements renouvelables (par exemple, développement et test, test d’acceptation utilisateur, pré-production et production) des mêmes charges de travail. En outre, les rayons peuvent servir à séparer et autoriser différents groupes au sein de votre organisation (par exemple, les groupes DevOps). À l’intérieur d’un rayon, il est tout aussi possible de déployer une charge de travail de base que des charges de travail multiniveau complexes avec un contrôle du trafic entre les différents niveaux.

#### <a name="subscription-limits-and-multiple-hubs"></a>Limites d’abonnement et concentrateurs multiples
Dans Azure, chacun des composants, quel qu’en soit le type, est déployé dans un abonnement Azure. L’isolement des composants Azure dans différents abonnements Azure peut satisfaire aux exigences de différents cœurs de métier, telles que la configuration de niveaux d’accès et d’autorisation différenciés.

Il est possible pour un VDC de monter en puissance en augmentant le nombre de rayons ; toutefois, à l’instar de chaque système informatique, il existe des limites de plateforme. Le déploiement d’un concentrateur est lié à un abonnement Azure spécifique, qui fait l’objet de restrictions et de limites, telles que le nombre maximal d’homologations de réseaux virtuels (pour plus d’informations, voir l’article [Abonnement Azure et limites, quotas et contraintes de service][Limits]). Dans les cas où ces limites peuvent poser problème, il est possible de procéder à la montée en puissance de l’architecture en faisant évoluer un modèle hub-and-spoke unique vers un cluster constitué de plusieurs concentrateurs et rayons. Plusieurs concentrateurs situés dans une ou plusieurs régions Azure peuvent être interconnectés à l’aide de VNET Peering, d’un réseau privé virtuel ExpressRoute ou de site à site.

[![2]][2]

L’introduction de plusieurs concentrateurs augmente le coût et l’effort de gestion du système et n’est donc justifiée que par une volonté d’extensibilité (par exemple, pour contourner les limites du système ou à des fins de redondance) et de réplication régionale (exemples : performances de l’utilisateur final ou récupération d’urgence). Dans les scénarios qui nécessitent plusieurs concentrateurs, ces derniers doivent tous, dans la mesure du possible, offrir le même ensemble de services afin d’en faciliter l’exploitation.

#### <a name="interconnection-between-spokes"></a>Interconnexion entre les rayons
Il est possible d’implémenter des charges de travail multiniveau complexes à l’intérieur d’un même rayon. Les configurations multiniveau peuvent être implémentées à l’aide de sous-réseaux (un pour chaque niveau) dans le même réseau virtuel, ainsi qu’au moyen d’un filtrage des flux par l’intermédiaire de NSG.

En outre, un architecte peut vouloir déployer une charge de travail multiniveau dans plusieurs réseaux virtuels. L’homologation de réseaux virtuels permet de connecter des rayons à d’autres rayons du même concentrateur ou de différents concentrateurs. Ce scénario s’applique généralement aux cas où les serveurs de traitement applicatif sont situés dans un même rayon (réseau virtuel), tandis que la base de données est déployée dans un autre rayon (réseau virtuel). Dans ce cas, il est facile d’interconnecter les rayons avec l’homologation de réseaux virtuels et d’éviter ainsi le transit par le concentrateur. Toutefois, il convient d’étudier avec soin l’architecture et la sécurité pour s’assurer que le contournement du concentrateur n’élude pas de points de sécurité ou d’audit importants susceptibles d’exister uniquement dans le concentrateur.

[![3]][3]

Il est également possible d’interconnecter des rayons à un rayon jouant le rôle de concentrateur. Cette approche crée une hiérarchie à deux niveaux : le rayon du niveau supérieur (niveau 0) devient le concentrateur de rayons inférieurs (niveau 1) dans la hiérarchie. Les rayons du VDC doivent transférer le trafic vers le concentrateur central pour qu’il atteigne le réseau local ou Internet. Une architecture à deux niveaux de concentrateur introduit un routage complexe qui supprime les avantages d’une relation hub-and-spoke simple.

Bien qu’Azure autorise des topologies complexes, les deux principes fondamentaux du concept de VDC sont la répétabilité et la simplicité. Afin de minimiser l’effort de gestion, la conception hub-and-spoke (réseau en étoile) simple constitue donc l’architecture de référence recommandée pour un VDC.

### <a name="components"></a>Composants
Un centre de données virtuel comporte quatre types de composants de base : **infrastructure**, **réseaux de périmètre**, **charges de travail** et **surveillance**.

Chaque type de composant intègre plusieurs ressources et fonctionnalités Azure. Votre VDC est constitué d’instances de plusieurs types de composants et de diverses variantes du même type de composant. Par exemple, vous pouvez disposer de nombreuses instances de charge de travail distinctes, séparées de façon logique, qui représentent différentes applications. Utilisez ces différents types de composants et instances afin de générer le VDC.

[![4]][4]

L’architecture de VDC de haut niveau ci-dessus représente différents types de composants utilisés dans diverses zones de la topologie hub-and-spoke (réseau en étoile). Le diagramme illustre les composants d’infrastructure dans les différentes parties de l’architecture.

Une bonne pratique (pour un centre de données local ou un VDC) consiste à baser les droits d’accès et les privilèges sur un groupe. La manipulation de groupes, plutôt que d’utilisateurs individuels, garantit une gestion cohérente des stratégies d’accès entre les équipes et contribue à minimiser les erreurs de configuration. L’attribution et la suppression d’utilisateurs dans les groupes appropriés facilitent l’actualisation continue des privilèges d’un utilisateur spécifique.

Les noms de chaque groupe de rôles doivent comporter un préfixe unique de façon à simplifier l’identification du groupe associé à une charge de travail spécifique. Par exemple, une charge de travail hébergeant un service d’authentification peut être associée à des groupes appelés *AuthServiceNetOps, AuthServiceSecOps, AuthServiceDevOps et AuthServiceInfraOps*. De même, les rôles centralisés ou les rôles non associés à un service spécifique peuvent être précédés de « Corp », *CorpNetOps* par exemple.

De nombreuses organisations utilisent une variante des groupes ci-après pour procéder à une répartition générale des rôles :

-   Le *groupe informatique central (Corp)* dispose des droits de propriété requis pour contrôler les composants d’infrastructure (tels que les composants de mise en réseau et de sécurité), et doit donc être doté du rôle de contributeur dans l’abonnement (et avoir le contrôle du concentrateur) et de droits de contributeur de réseau dans les rayons. Une grande organisation répartit généralement ces responsabilités de gestion entre plusieurs équipes, telles qu’un groupe Opérations de réseau (CorpNetOps), exclusivement axé sur la mise en réseau, et un groupe Opérations de sécurité (CorpSecOps), chargé de la stratégie de pare-feu et de sécurité. Dans ce cas précis, il convient de créer deux groupes distincts pour l’attribution de ces rôles personnalisés.
-   Le *groupe de développement et test (AppDevOps)* est chargé de déployer les charges de travail (applications ou services). Ce groupe joue le rôle de contributeur de machine virtuelle pour les déploiements IaaS et/ou un ou plusieurs rôles de contributeur PaaS (voir l’article [Rôles intégrés pour le contrôle d’accès en fonction du rôle Azure][Roles]). En option, l’équipe de développement et de test peut avoir besoin de disposer d’une visibilité des stratégies de sécurité (NSG) et des stratégies de routage (UDR) dans le concentrateur ou dans un rayon spécifique. Par conséquent, outre les rôles de contributeur pour les charges de travail, ce groupe nécessite également le rôle de lecteur de réseau.
-   Le *groupe des opérations et de la maintenance (CorpInfraOps ou AppInfraOps)* est chargé de gérer les charges de travail en production. Ce groupe doit être un contributeur d’abonnement sur les charges de travail dans tous les abonnement de production. Certaines organisations peuvent également déterminer si elles ont besoin d’un groupe de prise en charge du processus d’escalade avec le rôle de contributeur d’abonnement en production ainsi que dans l’abonnement de concentrateur central, afin de résoudre les éventuels problèmes de configuration dans l’environnement de production.

Un VDC est structuré de façon que les groupes créés pour les groupes informatiques centraux qui gèrent le concentrateur disposent de groupes correspondants au niveau de la charge de travail. Outre la gestion des ressources de concentrateur, seuls les groupes informatiques centraux sont en mesure de contrôler l’accès externe et les autorisations de niveau supérieur sur l’abonnement. Toutefois, les groupes de charge de travail peuvent contrôler les ressources et les autorisations de leur réseau virtuel indépendamment du groupe informatique central.

Le VDC doit être partitionné de façon à héberger de manière sécurisée plusieurs projets appartenant à différents cœurs de métier. Tous les projets nécessitent plusieurs environnements isolés (développement, test d’acceptation utilisateur (UAT), production). Les abonnements Azure distincts pour chacun de ces environnements assurent un isolement naturel.

[![5]][5]

Le diagramme précédent illustre la relation entre les projets, les utilisateurs et les groupes d’une organisation et les environnements dans lesquels les composants Azure sont déployés.

Dans le contexte informatique, un environnement (ou niveau) désigne généralement un système dans lequel plusieurs applications sont déployées et exécutées. Les grandes entreprises utilisent un environnement de développement (dans lequel les modifications sont initialement effectuées et testées) et un environnement de production (destiné aux utilisateurs finaux). Ces environnements sont séparés, le plus souvent par des environnements intermédiaires, afin d’autoriser l’échelonnement des phases de déploiement (lancement), de test et de restauration en cas de problème. Les architectures de déploiement varient sensiblement, mais le processus de base qui consiste à commencer par le niveau développement (DEV) et à terminer par le niveau production (PROD) reste généralement appliqué.

Une architecture communément employée pour ces types d’environnements multiniveau est constituée d’environnements DevOps (développement et test), UAT (intermédiaire) et de production. Les organisations peuvent tirer profit d’un ou de plusieurs locataires Azure AD afin de définir l’accès et les droits pour ces environnements. Le diagramme précédent illustre un cas d’utilisation de deux locataires Azure AD distincts : l’un pour DevOps et UAT, et l’autre exclusivement destiné à la production.

La présence de différents locataires Azure AD met en œuvre la séparation entre les environnements. Un même groupe d’utilisateurs (par exemple, le groupe informatique central) doit s’authentifier à l’aide d’un URI différent pour accéder à un autre locataire AD et modifier les rôles ou autorisations des environnements DevOps ou de production d’un projet. L’utilisation d’une authentification utilisateur distincte pour accéder aux différents environnements réduit les éventuelles interruptions et autres problèmes dus à des erreurs humaines.

#### <a name="component-type-infrastructure"></a>Type de composant : infrastructure
Ce type de composant est l’endroit où réside la majeure partie de l’infrastructure sous-jacente. Il s’agit également du composant auquel vos équipes informatique, sécurité et/ou conformité centralisées consacrent la majorité de leur temps.

[![6]][6]

Les composants d’infrastructure assurent une interconnexion entre les différents composants d’un VDC et sont présents à la fois dans le concentrateur et dans les rayons. La gestion et la maintenance des composants d’infrastructure sont généralement confiées à l’équipe informatique et/ou sécurité centralisées.

L’une des principales tâches de l’équipe d’infrastructure informatique consiste à garantir la cohérence des schémas d’adresse IP dans l’ensemble de l’entreprise. L’espace privé d’adresse IP attribué au VDC doit être cohérent et ne PAS chevaucher les adresses IP privées affectées à vos réseaux locaux.

Bien que la traduction d’adresses réseau (NAT) sur les routeurs de périphérie locaux ou dans les environnements Azure permette d’éviter les conflits d’adresses IP, elle ajoute de la complexité à vos composants d’infrastructure. L’une des finalités clés d’un VDC étant de simplifier la gestion, l’utilisation du processus NAT pour gérer les problèmes d’IP n’est pas recommandée.

Les composants d’infrastructure comprennent les fonctionnalités suivantes :

-   [**Services d’identité et d’annuaire**][AAD]. L’accès à chaque type de ressource dans Azure est contrôlé par une identité stockée dans un service d’annuaire. Le service d’annuaire stocke non seulement la liste des utilisateurs, mais également les droits d’accès aux ressources dans un abonnement Azure spécifique. Ces services peuvent exister uniquement dans le cloud, ou être synchronisés avec l’identité locale stockée dans Active Directory.
-   [**Réseau virtuel**][VPN]. Les réseaux virtuels sont l’un des principaux composants d’un VDC et vous permettent de créer une limite d’isolement du trafic sur la plateforme Azure. Un réseau virtuel est constitué d’un ou de plusieurs segments de réseau virtuel, chacun étant doté d’un préfixe de réseau IP (sous-réseau) spécifique. Le réseau virtuel définit une zone de périmètre interne dans laquelle les machines virtuelles IaaS et les services PaaS peuvent établir des communications privées. Les machines virtuelles (et les services PaaS) figurant dans un réseau virtuel ne peuvent pas communiquer directement avec les machines virtuelles (et les services PaaS) d’un autre réseau virtuel, même si ces deux réseaux virtuels sont créés par le même client, dans le cadre du même abonnement. Cet isolement est une propriété critique qui garantit que les machines virtuelles et les communications du client restent privées dans un réseau virtuel.
-   [**Itinéraire défini par l’utilisateur**][UDR]. Le routage du trafic dans un réseau virtuel repose par défaut sur la table de routage système. Un itinéraire défini par l’utilisateur est une table de routage personnalisée que les administrateurs réseau peuvent associer à un ou plusieurs sous-réseaux afin de remplacer le comportement de la table de routage système et de définir un chemin de communication au sein d’un réseau virtuel. L’existence d’itinéraires définis par l’utilisateur garantit que le trafic de sortie en provenance du rayon transite par des machines virtuelles personnalisées et/ou des appliances virtuelles réseau et des équilibreurs de charge spécifiques présents dans le concentrateur et dans les rayons.
-   [**Groupe de sécurité réseau (NSG)**][NSG]. Un groupe de sécurité réseau est une liste de règles de sécurité qui font office de filtrage du trafic sur les sources IP, la destination IP, les protocoles, les ports de source IP et les ports de destination IP. Le NSG peut être appliqué à un sous-réseau et/ou à une carte réseau virtuelle associée à une machine virtuelle Azure. Les NSG jouent un rôle essentiel dans l’implémentation d’un contrôle de flux approprié dans le concentrateur et dans les rayons. Le niveau de sécurité offert par le NSG dépend des ports que vous ouvrez et du but dans lequel vous le faites. Les clients doivent appliquer des filtres supplémentaires par machine virtuelle avec des pare-feu basés sur l’hôte, tels qu’IPtables ou le Pare-feu Windows.
-   [**DNS**][DNS]. La résolution de noms des ressources dans les réseaux virtuels d’un VDC est assurée par le biais du DNS. Azure fournit des services DNS pour les résolutions de noms [DNS] [public] et [privé][PrivateDNS]. Les zones privées fournissent la résolution de noms au sein d’un réseau virtuel, ainsi qu’entre des réseaux virtuels. Vous pouvez avoir des zones privées entre plusieurs réseaux virtuels au sein de la même région, mais aussi entre plusieurs régions et abonnements. Pour la résolution publique, Azure DNS fournit un service d’hébergement pour les domaines DNS qui offre une résolution de noms à l’aide de l’infrastructure Microsoft Azure. En hébergeant vos domaines dans Azure, vous pouvez gérer vos enregistrements DNS avec les mêmes informations d’identification, les mêmes API, les mêmes outils et la même facturation que vos autres services Azure.
-   [**Abonnement**][SubMgmt] et [**Gestion des groupes de ressources**][RGMgmt]. Un abonnement définit une limite naturelle permettant de créer plusieurs groupes de ressources dans Azure. Les ressources d’un abonnement sont regroupées dans des conteneurs logiques nommés groupes de ressources. Le groupe de ressources représente un groupe logique destiné à organiser les ressources d’un VDC.
-   [**Contrôle d’accès en fonction du rôle (RBAC)**][RBAC]. Le mécanisme RBAC permet de mapper un rôle organisationnel sur des droits d’accès à des ressources Azure spécifiques, ce qui vous donne la possibilité de restreindre les utilisateurs à un sous-ensemble d’actions. Avec RBAC, vous pouvez accorder l’accès en attribuant le rôle approprié à des utilisateurs, groupes et applications dans l’étendue adéquate. L’étendue d’une attribution de rôle peut être un abonnement Azure, un groupe de ressources ou une ressource unique. Le mécanisme RBAC autorise l’héritage des autorisations. Un rôle attribué à une étendue parent accorde également l’accès aux enfants qu’elle contient. RBAC vous permet de séparer les tâches et d’accorder aux utilisateurs uniquement les accès dont ils ont besoin pour accomplir leur travail. Par exemple, utilisez RBAC pour autoriser un employé à gérer les machines virtuelles d’un abonnement, et pour permettre à un autre employé de gérer les bases de données SQL au sein du même abonnement.
-   [**Homologation de réseaux virtuels**][VNetPeering]. La fonctionnalité fondamentale utilisée pour créer l’infrastructure d’un VDC est l’homologation de réseaux virtuels, un mécanisme qui connecte deux réseaux virtuels de la même région par le biais du réseau du centre de données Azure, ou bien en utilisant la dorsale principale mondiale de Azure entre plusieurs régions.

#### <a name="component-type-perimeter-networks"></a>Type de composant : réseaux de périmètre
Les composants de type [réseau de périmètre][DMZ] (également appelé réseau DMZ) vous permettent d’assurer la connectivité réseau des réseaux de vos centres de données locaux ou physiques, ainsi que la connectivité à destination et en provenance d’Internet. Il s’agit également des composants auxquels vos équipes réseau et sécurité consacrent généralement la majorité de leur temps.

Les paquets entrants doivent transiter par les appliances de sécurité dans le concentrateur, notamment le pare-feu et les systèmes de détection et de prévention d’intrusion, avant d’atteindre les serveurs principaux dans les rayons. Avant de quitter le réseau, les paquets Internet des charges de travail doivent également traverser les appliances de sécurité dans le réseau de périmètre à des fins d’application des stratégies, d’inspection et d’audit.

Les composants de type réseau de périmètre fournissent les fonctionnalités suivantes :

-   [Réseaux virtuels][VNet], [itinéraires définis par l’utilisateur][UDR], [groupes de sécurité réseau][NSG]
-   [Appliance virtuelle réseau][NVA]
-   [Équilibreur de charge][ALB]
-   [Application Gateway][AppGW] / [Pare-feu d’applications web (WAF)][WAF]
-   [Adresses IP publiques][PIP]

En règle générale, les équipes informatique et sécurité centralisées sont chargées de la définition des exigences et des opérations des réseaux de périmètre.

[![7]][7]

Le diagramme précédent illustre l’application de deux périmètres avec un accès à Internet et à un réseau local, ces deux périmètres résidant dans le concentrateur. Dans un concentrateur unique, le réseau de périmètre vers Internet peut monter en puissance pour prendre en charge une multitude de cœurs de métier, à l’aide de plusieurs batteries de pare-feu d’applications web (WAF) et/ou de pare-feu.

[**Réseaux virtuels**][VNet] Le concentrateur repose généralement sur un réseau virtuel avec plusieurs sous-réseaux pour héberger les différents types de services qui filtrent et inspectent le trafic à destination ou en provenance d’Internet par le biais d’appliances réseau virtuelles, de WAF et de passerelles Azure Application Gateway.

[**Itinéraires définis par l’utilisateur**][UDR] Ces itinéraires permettent aux clients de déployer des pare-feu, des systèmes de détection et de prévention d’intrusion et d’autres appliances virtuelles, et de router le trafic réseau à travers ces appliances de sécurité à des fins d’application de stratégies de limites de sécurité, d’audit et d’inspection. Les itinéraires définis par l’utilisateur peuvent être créés à la fois dans le concentrateur et dans les rayons pour garantir que le trafic transite par des machines virtuelles personnalisées, des appliances virtuelles réseau et des équilibreurs de charge spécifiques utilisés par le VDC. Pour garantir que le trafic généré à partir de machines virtuelles figurant dans le rayon transite par les appliances virtuelles correctes, il convient de configurer un itinéraire défini par l’utilisateur dans les sous-réseaux du rayon en définissant l’adresse IP frontale de l’équilibreur de charge interne en tant que tronçon suivant. L’équilibreur de charge interne distribue le trafic interne vers les appliances virtuelles (pool principal d’équilibreur de charge).

[![8]][8]

[**Appliances virtuelles réseau**][NVA] Dans le concentrateur, le réseau de périmètre disposant d’un accès à Internet est habituellement géré par le biais d’une batterie de pare-feu et/ou de pare-feu d’applications web (WAF).

Les différents cœurs de métier utilisent généralement de nombreuses applications web qui sont susceptibles de présenter diverses vulnérabilités et d’être exposées à des codes malveillants exploitant une faille de sécurité. Les pare-feu d’applications web sont un type de produit spécial permettant de détecter les attaques contre les applications web (HTTP/HTTPS) de façon plus approfondie qu’un pare-feu générique. Contrairement à la technologie de pare-feu classique, les WAF intègrent un ensemble de fonctionnalités spécifiques pour protéger les serveurs web internes contre les menaces.

Une batterie de pare-feu est un groupe de pare-feu opérant en tandem et relevant d’une entité de gestion commune, avec un ensemble de règles de sécurité pour protéger les charges de travail hébergées dans les rayons, et pour contrôler l’accès aux réseaux locaux. Une batterie de pare-feu est équipée de logiciels moins spécialisés qu’un WAF, mais dispose d’un vaste champ d’application permettant de filtrer et d’inspecter n’importe quel type de trafic en entrée et en sortie. Les batteries de pare-feu sont normalement implémentées dans Azure par le biais d’appliances virtuelles réseau, qui sont disponibles dans la plateforme Place de marché Azure.

Il est recommandé d’utiliser un ensemble d’appliances virtuelles réseau pour le trafic provenant d’Internet, et un autre ensemble pour le trafic émanant du niveau local. L’utilisation d’un seul ensemble d’appliances virtuelles réseau pour ces deux types de trafics réseau constitue un risque pour la sécurité, car elle n’offre aucun périmètre de sécurité entre les deux. L’utilisation d’appliances virtuelles réseau distinctes simplifie la vérification des règles de sécurité et permet d’identifier clairement les règles auxquelles correspondent les différentes demandes réseau entrantes.

La plupart des grandes entreprises gèrent plusieurs domaines. Le service Azure DNS permet d’héberger les enregistrements DNS d’un domaine spécifique. En guise d’exemple, l’adresse IP virtuelle (VIP) de l’équilibreur de charge externe Azure (ou des WAF) peut être enregistrée dans l’enregistrement A d’un enregistrement Azure DNS.

[**Azure Load Balancer**][ALB] L’équilibreur de charge Azure offre un service haute disponibilité de couche 4 (TCP, UDP), qui peut distribuer le trafic entrant entre des instances de service définies dans un jeu d’équilibrage de la charge. Le trafic envoyé à l’équilibreur de charge à partir de points de terminaison frontaux (points de terminaison IP publics ou privés) peut être redistribué avec ou sans traduction d’adresses à un ensemble d’un pool d’adresses IP principal (par exemple, des appliances virtuelles réseau ou des machines virtuelles).

Azure Load Balancer peut également analyser à l’aide de sondes l’intégrité des différentes instances de serveur ; si l’une des sondes ne répond pas, l’équilibreur de charge arrête d’envoyer le trafic à l’instance défaillante correspondante. Un VDC comporte un équilibreur de charge externe dans le concentrateur (par exemple, pour équilibrer le trafic envoyé aux appliances virtuelles réseau) et dans les rayons (pour effectuer des tâches telles que l’équilibrage du trafic entre les différentes machines virtuelles d’une application multiniveau).

[**Application Gateway**][AppGW] Microsoft Azure Application Gateway est une appliance virtuelle dédiée qui intègre Application Delivery Controller (ADC) sous la forme d’un service, offrant ainsi diverses fonctionnalités d’équilibrage de charge de couche 7 pour votre application. Cette appliance vous permet d’optimiser la productivité de la batterie de serveurs Web en déchargeant une terminaison SSL nécessitant de nombreuses ressources du processeur vers la passerelle Application Gateway. Elle fournit également d’autres fonctionnalités de routage de couche 7, notamment la distribution en tourniquet (round robin) du trafic entrant, l’affinité de session basée sur les cookies, le routage basé sur le chemin d’accès de l’URL et la possibilité d’héberger plusieurs sites web derrière une seule passerelle Application Gateway. Un pare-feu d’applications web (WAF) est également intégré à la référence SKU Application Gateway WAF. Cette référence SKU protège les applications web contre les vulnérabilités et les codes malveillants exploitant une faille de sécurité les plus courants sur le web. La passerelle Application Gateway peut être configurée en tant que passerelle internet, passerelle interne uniquement ou une combinaison des deux. 

[**Adresses IP publiques**][PIP] Certaines fonctionnalités Azure vous permettent d’associer des points de terminaison de service à une adresse IP publique pour rendre votre ressource accessible à partir d’Internet. Ce point de terminaison utilise la traduction d’adresses réseau (NAT) pour router le trafic vers l’adresse et le port internes sur le réseau virtuel Azure. Il s’agit du principal chemin d’accès pour que le trafic externe passe dans le réseau virtuel. Il est possible de configurer les adresses IP publiques de manière à déterminer le type de trafic transmis, ainsi que la façon dont la traduction s’opère sur le réseau virtuel et son emplacement précis.

#### <a name="component-type-monitoring"></a>Type de composant : surveillance
Les composants de surveillance offrent une vue d’ensemble de tous les autres types de composants et génèrent des alertes concernant ces derniers. Toutes les équipes doivent avoir accès à la surveillance des composants et services qui leur sont accessibles. Si vous disposez d’un support technique centralisé ou d’équipes des opérations, vous devez leur offrir un accès intégré aux données fournies par ces composants.

Azure propose différents types de services de journalisation et de surveillance pour effectuer le suivi du comportement des ressources hébergées par Azure. La gouvernance et le contrôle des charges de travail dans Azure reposent non seulement sur la collecte des données de journalisation, mais également sur la possibilité de déclencher des actions basées sur des événements signalés spécifiques.

[**Azure Monitor**][Monitor] - Azure comprend plusieurs services qui effectuent individuellement un rôle ou une tâche spécifique dans l’espace d’analyse. Ensemble, ces services fournissent une solution complète pour la collecte, l’analyse et l’action sur les données de télémétrie de votre application et des ressources Azure qui les prennent en charge. Ces services peuvent aussi surveiller les ressources locales critiques afin de fournir un environnement de surveillance hybride. Comprendre les outils et les données disponibles est la première étape du développement d’une stratégie de surveillance complète pour votre application.

Il existe deux principaux types de journaux dans Azure :

-   Les [**journaux d’activité**][ActLog] (également désignés sous le terme de « journaux des opérations ») fournissent des informations sur les opérations qui ont été effectuées sur les ressources de l’abonnement Azure. Ces journaux signalent les événements de plan de contrôle relatifs à vos abonnements. Chaque ressource Azure génère des journaux d’audit.

-   Les [**journaux de diagnostic Azure**][DiagLog] sont les journaux générés par une ressource qui fournissent des informations fréquentes et fournies concernant le fonctionnement de cette ressource. Le contenu de ces journaux varie en fonction du type de ressource.

[![9]][9]

Dans un VDC, il est primordial d’assurer le suivi des journaux des groupes de sécurité réseau, et notamment des informations suivantes :

-   [**Journaux des événements**][NSGLog] : indiquent les règles NSG qui sont appliquées aux machines virtuelles et aux rôles d’instance en fonction de l’adresse MAC.
-   [**Journaux de compteur**][NSGLog] : indiquent le nombre d’exécutions de chaque règle NSG pour refuser ou autoriser le trafic.

Tous les journaux peuvent être stockés dans des comptes de stockage Azure à des fins d’audit, d’analyse statique ou de sauvegarde. Lorsque les journaux sont stockés dans un compte de stockage Azure, les clients peuvent utiliser différents types d’infrastructures pour récupérer, préparer, analyser et visualiser ces données afin de signaler l’état et l’intégrité des ressources cloud.

Les grandes entreprises doivent avoir préalablement acquis une infrastructure standard pour la surveillance des systèmes locaux, et peuvent étendre cette infrastructure afin d’y intégrer les journaux générés par les déploiements cloud. Dans le cas des organisations qui souhaitent conserver l’ensemble de la journalisation dans le cloud, la solution [Log Analytics][../log-analytics/log-analytics-overview .md] constitue un excellent choix. La solution Log Analytics étant implémentée sous la forme d’un service informatique, elle peut être opérationnelle rapidement, avec un investissement minimal en services d’infrastructure. Log Analytics peut également s’intégrer aux composants System Center, tels que System Center Operations Manager, afin d’étendre au cloud vos investissements de gestion existants.

Log Analytics est un service dans Azure conçu pour faciliter la collecte, la mise en corrélation, la recherche et l’exploitation des données de journalisation et de performances générées par les systèmes d’exploitation, les applications et les composants cloud d’infrastructure. Ce composant vous offre des informations opérationnelles en temps réel à l’aide d’une fonction de recherche intégrée et de tableaux de bord personnalisés, qui vous permettent d’analyser tous les enregistrements de l’ensemble de vos charges de travail dans un VDC.

La solution [Network Performance Monitor (NPM)][NPM] à l’intérieur d’OMS peut fournir des informations détaillées sur le réseau, de bout en bout, y compris une vue unique de vos réseaux Azure et des réseaux locaux. Avec une surveillance spéciale les services publics et ExpressRoute.

#### <a name="component-type-workloads"></a>Type de composant : charges de travail
Les composants de type charge de travail désignent l’emplacement où résident vos applications et services proprement dits. Il s’agit également du composant auquel vos équipes de développement d’applications consacrent la majorité de leur temps.

Les possibilités en matière de charge de travail sont réellement illimitées. Voici quelques-uns des innombrables types de charges de travail possibles :

**Applications métiers internes**. Les applications métiers sont des applications informatiques cruciales pour le fonctionnement continu d’une entreprise. Les applications métiers présentent certaines caractéristiques en commun :

-   **Interactives**. Les applications métiers sont interactives par nature : elles reçoivent des données et renvoient des résultats ou rapports.
-   **Pilotées par les données**. Les applications métiers utilisent de nombreuses données et accèdent fréquemment aux bases de données ou à d’autres types de stockages.
-   **Intégrées**. Les applications métiers assurent une intégration à d’autres systèmes au sein ou à l’extérieur de l’organisation.

**Sites web destinés aux clients (accessible sur Internet ou en interne)**. La plupart des applications qui interagissent avec Internet sont des sites web. Azure permet d’exécuter un site web sur une machine virtuelle IaaS ou à partir d’un site [Azure Web Apps][WebApps] (PaaS). Azure Web Apps prend en charge l’intégration à des réseaux virtuels permettant le déploiement des applications web dans le rayon d’un VDC. En examinant les sites web internes, avec l’intégration aux réseaux virtuels, vous n’avez pas besoin d’exposer un point de terminaison Internet pour vos applications, mais vous pouvez utiliser via des adresses routables non-Internet privées des ressources à partir de votre réseau virtuel privé.

**Big Data/analytique** Lorsqu’il devient nécessaire de traiter de très gros volumes de données, la montée en puissance des bases de données peut ne pas s’effectuer correctement. La technologie Hadoop offre un système permettant d’exécuter des requêtes distribuées en parallèle sur un grand nombre de nœuds. Les clients ont la possibilité d’exécuter des charges de travail de données dans des machines virtuelles IaaS ou PaaS ([HDInsight][HDI]). HDInsight prend en charge le déploiement dans un réseau virtuel basé sur l’emplacement et peut être déployé sur un cluster dans un rayon du VDC.

**Événements et messagerie**. [Azure Event Hubs][EventHubs] est un service d’ingestion de données de télémétrie à très grande échelle qui collecte, transforme et stocke des millions d’événements. En tant que plateforme de streaming distribuée, il offre une faible latence et une durée de rétention configurable vous permettant d’ingérer des quantités massives de données de télémétrie dans Azure et de lire les données de plusieurs applications. Le service Event Hubs prend en charge le traitement de pipelines en temps réel et par lots sur le même flux.

Il est possible d’implémenter un service de messagerie cloud haute fiabilité entre les applications et les services par le biais du service [Azure Service Bus][ServiceBus] qui offre une messagerie répartie asynchrone entre un client et un serveur, ainsi qu’une messagerie structurée du type premier entré, premier sorti (FIFO) et des fonctionnalités de publication/abonnement.

[![10]][10]

### <a name="multiple-vdc"></a>VDC multiples
Jusqu’à présent, cet article s’est concentré sur un seul VDC en décrivant l’architecture et les composants de base qui permettent d’obtenir un VDC résilient. Les fonctionnalités Azure telles que l’équilibreur de charge Azure, les appliances virtuelles réseau, les groupes à haute disponibilité, les groupes identiques et divers autres mécanismes contribuent à l’obtention d’un système qui vous permet d’assurer des niveaux de contrat de niveau de service élevés dans vos services de production.

Toutefois, un VDC unique n’est hébergé que dans une seule région et est exposé à des interruptions majeures susceptibles d’affecter la totalité de cette région. Les clients qui souhaitent bénéficier de contrats de niveau de service élevés doivent protéger les services en déployant le même projet dans (au moins) deux VDC, placés dans différentes régions.

Outre la question des contrats de niveau de service, il existe plusieurs scénarios courants dans lesquels le déploiement de plusieurs VDC se révèle judicieux :

-   Présence régionale/mondiale
-   Récupération d’urgence
-   Mécanisme de redirection du trafic vers d’autres centres de données

#### <a name="regionalglobal-presence"></a>Présence régionale/mondiale
Les centres de données Azure sont présents dans de nombreuses régions du monde. Lorsqu’ils sélectionnent plusieurs centres de données Azure, les clients doivent considérer deux facteurs associés : les distances géographiques et la latence. Les clients doivent évaluer la distance géographique entre les VDC, ainsi qu’entre le VDC et les utilisateurs finaux, afin d’optimiser l’expérience utilisateur.

La région Azure où sont hébergés les VDC doit également respecter les exigences réglementaires établies par toute compétence juridique de laquelle relève votre organisation.

#### <a name="disaster-recovery"></a>Récupération d'urgence
L’implémentation d’un plan de récupération d’urgence est étroitement associée au type de charge de travail concernée, ainsi qu’à la possibilité de synchroniser l’état de la charge de travail entre différents VDC. Dans l’idéal, la plupart des clients souhaitent synchroniser les données d’application entre des déploiements s’exécutant dans deux VDC distincts afin d’implémenter un mécanisme de basculement rapide. La plupart des applications sont sensibles à la latence, ce qui peut entraîner des délais d’expiration et des retards dans la synchronisation des données.

La synchronisation ou l’analyse des pulsations des applications dans différents VDC nécessitent une communication entre ces derniers. Deux VDC situés dans des régions distinctes peuvent être connectés par le biais de différentes méthodes :

-   VNET Peering - VNET Peering peut se connecter à des concentrateurs dans différentes régions
-   Homologation privée ExpressRoute lorsque les concentrateurs de VDC sont connectés au même circuit ExpressRoute
-   Plusieurs circuits ExpressRoute connectés au moyen de votre segment principal (backbone) d’entreprise et de votre maille VDC connectée aux circuits ExpressRoute
-   Connexions de réseau privé virtuel de site à site entre les concentrateurs de vos VDC dans chaque région Azure

Les connexions VNET Peering et ExpressRoute constituent généralement le mécanisme privilégié en raison d’une bande passante plus importante et d’une latence constante lors du transit à travers le segment principal (backbone) de Microsoft.

Il n’existe aucune formule magique pour valider une application distribuée entre (au moins) deux VDC différents situés dans des régions distinctes. Les clients doivent exécuter des tests de qualification de réseau pour vérifier la latence et la bande passante des connexions et pour déterminer si la réplication synchrone ou asynchrone des données est appropriée, ainsi que l’objectif de délai de récupération (RTO) optimal possible pour les charges de travail.

#### <a name="mechanism-to-divert-traffic-between-dc"></a>Mécanisme de redirection du trafic vers d’autres centres de données
Une technique efficace pour rediriger le trafic entrant dans un centre de données vers un autre repose sur DNS. [Azure Traffic Manager][TM] utilise le mécanisme DNS (Domain Name System) pour diriger le trafic de l’utilisateur final vers le point de terminaison public le plus approprié dans un VDC spécifique. Au moyen de sondes, Traffic Manager vérifie périodiquement l’intégrité du service des points de terminaison publics dans différents VDC et, en cas de défaillance de ces points de terminaison, redirige automatiquement le trafic vers le VDC secondaire.

Traffic Manager fonctionne sur les points de terminaison publics Azure et permet, par exemple, de contrôler/rediriger le trafic vers les machines virtuelles Azure et Web Apps dans le VDC approprié. Traffic Manager est résilient même en cas de défaillance de la totalité d’une région Azure et peut contrôler la distribution du trafic utilisateur pour les points de terminaison de service dans différents VDC en fonction de plusieurs critères (par exemple, l’échec d’un service dans un VDC spécifique, ou la sélection du VDC présentant la latence réseau la plus faible pour le client).

### <a name="conclusion"></a>Conclusion
Le centre de données virtuel est une approche de migration de centre de données vers le cloud combinant plusieurs fonctionnalités et capacités afin de créer une architecture évolutive dans Azure qui optimise l’utilisation des ressources du cloud en réduisant les coûts et en simplifiant la gouvernance du système. Le concept de VDC repose sur une topologie hub-and-spoke (réseau en étoile), en fournissant les services partagés communs dans le concentrateur et en autorisant des applications/charges de travail spécifiques dans les rayons. Un VDC respecte la structure des rôles d’une entreprise, où collaborent différents départements (équipes informatique centrale, DevOps, opérations et maintenance) dotés chacun d’une liste spécifique de rôles et de tâches. Un VDC répond aux exigences d’une migration « lift-and-shift », mais offre également de nombreux avantages pour les déploiements de cloud natifs.

## <a name="references"></a>Références
Ce document a abordé les fonctionnalités ci-après. Pour plus d’informations sur ces dernières, cliquez sur les liens correspondants.

| | | |
|-|-|-|
|Fonctionnalités réseau|Équilibrage de la charge.|Connectivité|
|[Réseaux virtuels Azure][VNet]</br>[Groupes de sécurité réseau][NSG]</br>[Journaux de groupe de sécurité réseau][NSGLog]</br>[Itinéraire défini par l’utilisateur][UDR]</br>[Appliances virtuelles réseau][NVA]</br>[Adresses IP publiques][PIP]</br>[DNS]|[Azure Load Balancer (L3) ][ALB]</br>[Application Gateway (L7) ][AppGW]</br>[Pare-feu d’applications web][WAF]</br>[Azure Traffic Manager][TM] |[Homologation de réseaux virtuels][VNetPeering]</br>[Réseau privé virtuel][VPN]</br>[ExpressRoute][ExR]
|Identité</br>|Surveillance</br>|Meilleures pratiques</br>|
|[Azure Active Directory][AAD]</br>[Multi-Factor Authentication][MFA]</br>[Contrôle d’accès en fonction du rôle][RBAC]</br>[Rôles Azure Active Directory par défaut][Roles] |[Azure Monitor][Monitor]</br>[Journaux d’activité][ActLog]</br>[Journaux de diagnostic][DiagLog]</br>[Microsoft Operations Management Suite][OMS]</br>[Network Performance Monitor][NPM]|[Meilleures pratiques en matières de réseaux de périmètre][DMZ]</br>[Gestion des abonnements][SubMgmt]</br>[Gestion des groupes de ressources][RGMgmt]</br>[Limites d’abonnement Azure][Limits] |
|Autres services Azure|
|[Azure Web Apps][WebApps]</br>[HDInsights (Hadoop) ][HDI]</br>[Event Hubs][EventHubs]</br>[Service Bus][ServiceBus]|



## <a name="next-steps"></a>Étapes suivantes
 - Explorez l’[homologation de réseaux virtuels][VNetPeering], la technologie sous-jacente des conceptions hub-and-spoke (réseau en étoile) de VDC
 - Implémentez [AAD][AAD] pour prendre en main l’exploration de [RBAC][RBAC].
 - Développez un modèle de gestion des abonnements et des ressources, ainsi qu’un modèle RBAC, pour vous conformer à la structure, aux exigences et aux stratégies de votre organisation. L’activité la plus importante est la planification. Dans la mesure du possible, planifiez les réorganisations, les fusions, les nouvelles gammes de produits, etc.

<!--Image References-->
[0]: ./images/networking-redundant-equipment.png "Exemples de chevauchement de composants" 
[1]: ./images/networking-vdc-high-level.png "Exemple de haut niveau de VDC hub-and-spoke" (réseau en étoile)
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
[NVA]: /azure/architecture/reference-architectures/dmz/nva-ha
[SubMgmt]: /azure/azure-resource-manager/resource-manager-subscription-governance 
[RGMgmt]: /azure/azure-resource-manager/resource-group-overview
[DMZ]: /azure/best-practices-network-security
[ALB]: /azure/load-balancer/load-balancer-overview
[PIP]: /azure/virtual-network/resource-groups-networking#public-ip-address
[AppGW]: /azure/application-gateway/application-gateway-introduction
[WAF]: /azure/application-gateway/application-gateway-web-application-firewall-overview
[Monitor]: /azure/monitoring-and-diagnostics/
[ActLog]: /azure/monitoring-and-diagnostics/monitoring-overview-activity-logs 
[DiagLog]: /azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs
[NSGLog]: /azure/virtual-network/virtual-network-nsg-manage-log
[OMS]: /azure/operations-management-suite/operations-management-suite-overview
[NPM]: /azure/log-analytics/log-analytics-network-performance-monitor
[WebApps]: /azure/app-service/
[HDI]: /azure/hdinsight/hdinsight-hadoop-introduction
[EventHubs]: /azure/event-hubs/event-hubs-what-is-event-hubs 
[ServiceBus]: /azure/service-bus-messaging/service-bus-messaging-overview
[TM]: /azure/traffic-manager/traffic-manager-overview
