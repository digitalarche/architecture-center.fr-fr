---
title: Sécurisation des solutions de données
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: 47e3be2afd14d980b98ac9659f7f1e5a4df3403f
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/08/2019
ms.locfileid: "54110873"
---
# <a name="securing-data-solutions"></a>Sécurisation des solutions de données

Pour un grand nombre d’entre nous, rendre les données accessibles dans le cloud, en particulier après l’utilisation exclusive des banques de données locales, peut entraîner certains problèmes relatifs à l’accessibilité accrue aux données et aux nouvelles méthodes de sécurisation.

## <a name="challenges"></a>Défis

- Centralisation de la surveillance et de l’analyse des événements de sécurité stockés dans plusieurs journaux.
- Mise en œuvre de la gestion du chiffrement et des autorisations à l’échelle de vos applications et de vos services.
- Garantie du fait que la gestion des identités fonctionne pour tous les composants de la solution, localement ou dans le cloud.

## <a name="data-protection"></a>Protection des données

La première étape de protection des informations consiste à identifier les éléments à protéger. Développez des recommandations claires, simples et correctement communiquées pour identifier, protéger et surveiller les ressources de données les plus importantes partout où elles résident. Établissez le niveau de protection le plus élevé pour les ressources qui ont un impact disproportionné sur la mission ou la rentabilité de l’organisation. Ces ressources sont nommées ressources de valeur élevée ou HVA. Effectuez une analyse stricte des dépendances de sécurité et de cycle de vie des HVA et établissez des conditions et des contrôles de sécurité appropriés. De même, identifiez et classez les ressources sensibles et définissez les technologies et les processus pour appliquer automatiquement les contrôles de sécurité.

Une fois les données nécessaires à la protection identifiées, envisagez la façon dont vous voulez protéger les données *au repos* et données *en transit*.

- **Données au repos** : données qui existent statiquement sur un média physique, qu’il s’agisse d’un disque magnétique ou optique, localement ou dans le cloud.
- **Données en transit** : données transférées entre des composants, des emplacements ou des programmes, sur le réseau, via une connexion Service Bus (depuis un emplacement local vers le cloud, ou inversement) ou au cours d’un processus d’entrée/sortie.

Pour en savoir plus sur la protection de vos données au repos ou en transit, consultez [Meilleures pratiques de chiffrement et de sécurité des données Azure](/azure/security/azure-security-data-encryption-best-practices).

## <a name="access-control"></a>Contrôle d’accès

Afin de mieux protéger vos données dans le cloud vous pouvez combiner la gestion des identités et le contrôle d’accès. Étant donné la diversité et le type des services de cloud, ainsi que la popularité grandissante du [cloud hybride](../scenarios/hybrid-on-premises-and-cloud.md), il existe plusieurs pratiques clés à suivre lorsqu’il s’agit de contrôler les identités et l’accès :

- centralisation de la gestion des identités
- activation de l’authentification unique (SSO)
- déploiement de la gestion des mots de passe
- application de l’authentification multifacteur (MFA) pour les utilisateurs
- utilisation du contrôle d’accès en fonction du rôle (RBAC).
- Les stratégies d’accès conditionnel peuvent être configurées, améliorant ainsi le concept classique de l’identité de l’utilisateur à l’aide de propriétés supplémentaires liées à l’emplacement de l’utilisateur, au type d’appareil, au niveau de correctif logiciel et ainsi de suite.
- Contrôle des emplacements de création des ressources à l’aide du gestionnaire de ressources.
- Surveillance active des activités suspectes

Pour plus d’informations, consultez [Meilleures pratiques en matière de sécurité du contrôle d’accès et de la gestion des identités Azure](/azure/security/azure-security-identity-management-best-practices).

## <a name="auditing"></a>Audit

Au-delà du contrôle des identités et de l’accès mentionné précédemment, les services et applications que vous utilisez dans le cloud doivent générer des événements liés à la sécurité que vous pouvez surveiller. Le principal défi de surveillance de ces événements est de gérer les quantités de journaux, afin d’éviter d’éventuels problèmes ou de dépanner les problèmes advenus. Les applications sur le cloud ont tendance à contenir de nombreux éléments mobiles, la plupart d'entre eux générant un certain niveau de journalisation et de télémétrie. Utilisez la surveillance et l’analyse centralisées pour pouvoir gérer et mieux comprendre la grande quantité d’informations.

Pour plus d’informations, consultez [Audit et journalisation Azure](/azure/security/azure-log-audit).

## <a name="securing-data-solutions-in-azure"></a>Sécurisation des solutions de données dans Azure

### <a name="encryption"></a>Chiffrement

**Machines virtuelles**. Utilisez [Azure Disk Encryption](/azure/security/azure-security-disk-encryption) pour chiffrer les disques attachés aux machines virtuelles Windows ou Linux. La solution est intégrée à [Azure Key Vault](/azure/key-vault/) pour faciliter le contrôle et la gestion des clés et des secrets de chiffrement de disque.

**Stockage Azure**. Utilisez [Chiffrement du Service de stockage Azure](/azure/storage/common/storage-service-encryption) pour chiffrer automatiquement des données au repos dans le stockage Azure. Le chiffrement, le déchiffrement et la gestion des clés sont totalement transparents pour les utilisateurs. Les données peuvent également être sécurisées en transit à l’aide du chiffrement côté client avec Azure Key Vault. Pour plus d’informations, consultez [Chiffrement côté client et Azure Key Vault pour le stockage Microsoft Azure](/azure/storage/common/storage-client-side-encryption).

**SQL Database** et **Azure SQL Data Warehouse**. Utilisez [Transparent Data Encryption](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql) (TDE) pour exécuter le chiffrement et le déchiffrement en temps réel de vos bases de données, sauvegardes associées et les fichiers journaux des transactions sans que cela nécessite de modifications de vos applications. SQL Database peut également utiliser [Always Encrypted](/azure/sql-database/sql-database-always-encrypted-azure-key-vault) afin de protéger des données sensibles au repos sur le serveur, lors de leur déplacement entre client et serveur, et lors de leur utilisation. Vous pouvez utiliser Azure Key Vault pour stocker vos clés de chiffrement Always Encrypted.

### <a name="rights-management"></a>Gestion des droits

[Azure Rights Management](/information-protection/understand-explore/what-is-azure-rms) est un service cloud qui utilise des stratégies de chiffrement, d’identité et d’autorisation pour sécuriser fichiers et courriers électroniques. Il fonctionne sur plusieurs appareils : téléphones, tablettes et PC. Il est possible de protéger les informations au sein et à l’extérieur de votre organisation, car cette protection reste auprès des données, même quand celles-ci quittent les limites de votre organisation.

### <a name="access-control"></a>Contrôle d’accès

Utilisez le [Contrôle d’accès en fonction du rôle](/azure/active-directory/role-based-access-control-what-is) (RBAC) pour limiter l’accès aux ressources Azure aux rôles d’utilisateur. Si vous utilisez Active Directory local, vous pouvez [synchroniser avec Azure AD](/azure/active-directory/active-directory-hybrid-identity-design-considerations-directory-sync-requirements) pour fournir aux utilisateurs une identité de cloud basée sur leur identité locale.

Utilisez [Accès conditionnel dans Azure Active Directory](/azure/active-directory/active-directory-conditional-access-azure-portal) afin d’appliquer des contrôles sur l’accès aux applications dans votre environnement en fonction de conditions spécifiques. Par exemple, votre déclaration de stratégie peut prendre la forme suivante : _Quand des prestataires essaient d’accéder à nos applications cloud à partir de réseaux non approuvés, bloquer l’accès_.

[Azure AD Privileged Identity Management](/azure/active-directory/active-directory-privileged-identity-management-configure) peut vous aider à gérer, contrôler et surveiller vos utilisateurs et les types de tâches qu’ils exécutent avec leurs privilèges d’administrateur. Il s’agit d’une étape importante pour limiter le nombre de personnes dans votre organisation pouvant effectuer des opérations privilégiées dans les applications Azure AD, Azure, Office 365 ou SaaS, et surveiller leurs activités.

### <a name="network"></a>Réseau

Pour protéger les données en transit, utilisez toujours SSL/TLS lors de l’échange de données entre les différents emplacements. Dans certains cas, il peut être nécessaire d’isoler l’intégralité du canal de communication entre votre infrastructure locale et sur le cloud, via un réseau privé virtuel (VPN) ou [ExpressRoute](/azure/expressroute/). Pour plus d’informations, consultez [Extension des solutions de données locales vers le cloud](../scenarios/hybrid-on-premises-and-cloud.md).

Utilisez [groupes de sécurité réseau](/azure/virtual-network/virtual-networks-nsg) (NSG) afin de réduire le nombre de vecteurs d’attaque potentielle. Un groupe de sécurité réseau contient une liste de règles de sécurité qui autorisent ou refusent le trafic réseau entrant ou sortant en fonction de l’adresse IP source ou de destination, du port et du protocole.

Utilisez [points de terminaison de service Réseau virtuel](/azure/virtual-network/virtual-network-service-endpoints-overview) pour sécuriser les ressources SQL Azure ou de stockage Azure, afin que seul le trafic à partir de votre réseau virtuel puisse accéder à ces ressources.

Les machines virtuelles au sein d’un réseau virtuel Azure (VNet) peuvent communiquer en toute sécurité avec d’autres VNet à l’aide de [l’appairage de réseau virtuel](/azure/virtual-network/virtual-network-peering-overview). Le trafic réseau entre les réseaux virtuels homologués est privé. Le trafic entre les réseaux virtuels reste sur le réseau principal de Microsoft.

Pour plus d’informations, consultez [Sécurité du réseau Azure](/azure/security/azure-network-security)

### <a name="monitoring"></a>Surveillance

[Azure Security Center](/azure/security-center/security-center-intro) collecte, analyse et intègre automatiquement des données de journaux provenant de vos ressources Azure, du réseau et des solutions partenaires connectées, telles que les solutions de pare-feu, pour détecter les menaces réelles et réduire le nombre de faux positifs.

[Log Analytics](/azure/log-analytics/log-analytics-overview) fournit un accès centralisé à vos journaux et vous aide à analyser ces données et à créer des alertes personnalisées.

[Détection des menaces Azure SQL Database](/azure/sql-database/sql-database-threat-detection) permet de détecter les activités anormales indiquant des tentatives inhabituelles ou potentiellement dangereuses visant à accéder ou à exploiter des bases de données. Les responsables de la sécurité ou autres administrateurs désignés peuvent obtenir une notification immédiate concernant les activités suspectes qui interviennent sur la base de données. Chaque notification contient des détails sur l’activité suspecte et fournit des recommandations pour vous aider à étudier et corriger la menace.
