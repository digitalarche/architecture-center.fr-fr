---
title: Sites web WordPress hautement évolutifs et sécurisés
titleSuffix: Azure Example Scenarios
description: Créez un site web WordPress hautement évolutif et sécurisé pour les événements multimédias.
author: david-stanford
ms.date: 09/18/2018
ms.openlocfilehash: c0dad12e1da1f17b75d0661195123da4a8267152
ms.sourcegitcommit: bb7fcffbb41e2c26a26f8781df32825eb60df70c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/20/2018
ms.locfileid: "53644039"
---
# <a name="highly-scalable-and-secure-wordpress-website"></a>Sites web WordPress hautement évolutifs et sécurisés

Cet exemple de scénario s’applique aux entreprises qui ont besoin d’une installation hautement évolutive et sécurisée de WordPress. Ce scénario repose sur un déploiement qui a été utilisé pour une convention importante et qui a été en mesure d’évoluer pour répondre au pic du trafic que les sessions entraînent sur le site.

## <a name="relevant-use-cases"></a>Cas d’usage appropriés

Les autres cas d’usage appropriés sont les suivants :

- événements multimédias qui entraînent des pics de trafic ;
- blogs qui utilisent WordPress comme système de gestion de contenu ;
- sites web d’entreprise ou de e-commerce qui utilisent WordPress ;
- sites web créés à l’aide d’autres systèmes de gestion de contenu.

## <a name="architecture"></a>Architecture

[![Présentation de l’architecture des composants Azure impliqués dans un déploiement de WordPress évolutif et sécurisé](media/secure-scalable-wordpress.png)](media/secure-scalable-wordpress.png#lightbox)

Ce scénario illustre une installation sécurisée et évolutive de WordPress qui utilise des serveurs web Ubuntu et MariaDB. Il existe deux flux de données distincts dans ce scénario, le premier correspondant à l’accès des utilisateurs au site web :

1. Les utilisateurs accèdent au site web frontal via un réseau CDN.
2. Le réseau CDN utilise un équilibreur de charge Azure comme origine et en extrait toutes les données qui ne sont pas mises en cache.
3. L’équilibreur de charge Azure distribue des requêtes aux groupes de machines virtuelles identiques de serveurs web.
4. L’application WordPress extrait toutes les informations dynamiques des clusters Maria DB ; tout le contenu statique est hébergé dans Azure Files.
5. Les clés SSL sont stockées dans Azure Key Vault.

Le second flux de travail correspond au mode de contribution des auteurs pour le nouveau contenu :

1. Les auteurs se connectent en toute sécurité à la passerelle VPN publique.
2. Les informations d’authentification VPN sont stockées dans Azure Active Directory.
3. Une connexion est alors établie avec la zone d’accès d’administrateur.
4. À partir de la zone d’accès d’administrateur, l’auteur peut alors se connecter à l’équilibreur de charge Azure pour la création du cluster.
5. L’équilibreur de charge Azure distribue le trafic vers les groupes de machines virtuelles identiques de serveurs web qui disposent d’un accès en écriture au cluster MariaDB.
6. Le nouveau contenu statique est chargé vers Azure Files tandis que le contenu dynamique est écrit dans le cluster MariaDB.
7. Ces modifications sont ensuite répliquées vers l’autre région par le biais de RSYNC ou d’une réplication maître/subordonné.

### <a name="components"></a>Composants

- [Azure Content Delivery Network (CDN)](/azure/cdn/cdn-overview) est un réseau distribué de serveurs capables de fournir efficacement du contenu web aux utilisateurs. Les réseaux CDN réduisent la latence en stockant le contenu en cache sur des serveurs Edge dans des points de présence (POP) proches des utilisateurs finaux.
- Les [réseaux virtuels](/azure/virtual-network/virtual-networks-overview) permettent à de nombreuses ressources, telles que les machines virtuelles de communiquer en toute sécurité entre elles, avec Internet et avec les réseaux locaux. Les réseaux virtuels fournissent un isolement et une segmentation, ils filtrent et acheminent le trafic et ils autorisent une connexion entre des emplacements. Les deux réseaux sont connectés via VNET Peering.
- Les [groupes de sécurité réseau](/azure/virtual-network/security-overview) contiennent une liste de règles de sécurité qui autorisent ou refusent le trafic réseau entrant ou sortant en fonction du protocole, du port et de l’adresse IP source ou de destination. Les réseaux virtuels dans ce scénario sont sécurisés avec des règles de groupe de sécurité réseau qui limitent le flux du trafic entre les composants d’application.
- Les [équilibreurs de charge](/azure/load-balancer/load-balancer-overview) distribuent le trafic entrant en fonction des règles et des sondes d’intégrité. Un équilibreur de charge offre une latence faible et un débit élevé, et peut augmenter l’échelle jusqu’à des millions de flux pour toutes les applications TCP et UDP. Dans ce scénario, un équilibreur de charge permet de distribuer le trafic à partir du réseau de distribution de contenu vers les serveurs web frontaux.
- Les [groupes de machines virtuelles identiques][docs-vmss] vous permettent de créer et de gérer un groupe de machines virtuelles identiques à charge équilibrée. Le nombre d’instances de machine virtuelle peut augmenter ou diminuer automatiquement en fonction d’une demande ou d’un calendrier défini. Deux groupes de machines virtuelles identiques distincts sont utilisés dans ce scénario : un pour les serveurs web frontaux proposant du contenu, et l’autre pour les serveurs web frontaux utilisés pour créer du contenu.
- Comme [Azure Files](/azure/storage/files/storage-files-introduction) fournit un partage de fichiers totalement managé dans le cloud qui héberge l’ensemble du contenu de WordPress dans ce scénario, toutes les machines virtuelles ont accès aux données.
- La solution [Azure Key Vault](/azure/key-vault/key-vault-overview) est utilisée pour stocker et contrôler étroitement l’accès aux mots de passe, certificats et clés.
- [Azure Active Directory (Azure AD)](/azure/active-directory/fundamentals/active-directory-whatis) est un service cloud et mutualisé de gestion des répertoires et des identités. Dans ce scénario, Azure AD fournit des services d’authentification pour le site web et les tunnels VPN.

### <a name="alternatives"></a>Autres solutions

- [SQL Server pour Linux](/azure/virtual-machines/linux/sql/sql-server-linux-virtual-machines-overview) peut remplacer le magasin de données MariaDB.
- [Azure Database pour MySQL](/azure/mysql/overview) peut remplacer le magasin de données MariaDB si vous préférez une solution entièrement managée.

## <a name="considerations"></a>Considérations

### <a name="availability"></a>Disponibilité

Dans ce scénario, les instances de machine virtuelle sont déployées dans plusieurs régions, avec les données répliquées entre les deux via RSYNC pour le contenu de WordPress et une réplication maître/subordonné pour les clusters MariaDB.

Pour consulter d’autres rubriques relatives à la disponibilité, consultez la [liste de contrôle de la disponibilité][availability] dans le Centre des architectures Azure.

### <a name="scalability"></a>Extensibilité

Ce scénario utilise des groupes de machines virtuelles identiques pour les deux clusters de serveurs web frontaux dans chaque région. Avec des groupes identiques, le nombre d’instances de machine virtuelle qui s’exécutent au niveau de la couche Application frontale peut être automatiquement mis à l’échelle selon la demande du client ou une planification définie. Pour plus d’informations, voir [Vue d’ensemble de la mise à l’échelle automatique avec des groupes de machines virtuelles identiques][docs-vmss-autoscale].

Le serveur principal est un cluster MariaDB dans un groupe à haute disponibilité. Pour plus d’informations, consultez le [didacticiel portant sur le cluster MariaDB][mariadb-tutorial].

Pour consulter d’autres rubriques relatives à l’extensibilité, consultez la [liste de contrôle de l’extensibilité][scalability] dans le Centre des architectures Azure.

### <a name="security"></a>Sécurité

Tout le trafic du réseau virtuel passe dans la couche Application frontale et est protégé par des groupes de sécurité réseau. Des règles limitent le flux du trafic afin que seules les instances de machine virtuelle au niveau de la couche Application frontale puissent accéder à la couche Données principale. Aucun trafic Internet sortant n’est autorisé à partir de la couche Données. Pour réduire l’encombrement de l’attaque, aucun port de gestion à distance directe n’est ouvert. Pour plus d'informations, consultez [Groupes de sécurité réseau Azure][docs-nsg].

Pour obtenir des conseils d’ordre général sur la conception de scénarios sécurisés, consultez la [documentation sur la sécurité Azure][security].

### <a name="resiliency"></a>Résilience

Conjointement avec l’utilisation de plusieurs régions, la réplication des données et les groupes de machines virtuelles identiques, ce scénario utilise des équilibreurs de charge Azure. Ces composants réseau distribuent le trafic vers les instances de machine virtuelle connectées et incluent des sondes d’intégrité qui garantissent que le trafic est uniquement distribué vers des machines virtuelles saines. Tous ces composants réseau sont exposés via un réseau CDN. De cette façon, les ressources réseau et l’application sont résilientes aux problèmes qui perturberaient autrement le trafic et affecteraient l’accès de l’utilisateur final.

Pour obtenir des conseils d’ordre général sur la conception de scénarios résilients, consultez l’article [Conception d’applications résilientes pour Azure][resiliency].

## <a name="pricing"></a>Tarifs

Pour explorer le coût d’exécution de ce scénario, tous les services sont préconfigurés dans le calculateur de coûts. Pour pouvoir observer l’évolution de la tarification pour votre cas d’usage particulier, modifiez les variables appropriées en fonction du trafic que vous escomptez.

Nous avons fourni un [profil de coût][pricing] préconfiguré basé sur le diagramme d’architecture affiché ci-dessus. Pour configurer la calculatrice de prix pour votre cas d’usage, vous devez prendre en compte quelques éléments principaux :

- Quel est le trafic escompté en termes de Go/mois ? La quantité du trafic a le plus gros impact sur vos coûts, car elle affecte le nombre de machines virtuelles qui sont nécessaires pour exposer les données dans le groupe de machines virtuelles identiques. En outre, elle est directement mise en corrélation avec la quantité de données exposées via le réseau CDN.
- Combien de nouvelles données allez-vous écrire sur votre site web ? Les nouvelles données écrites sur votre site web sont mises en corrélation avec la quantité de données mises en miroir dans les régions.
- Quelle est la proportion du contenu dynamique ? Quelle est la proportion statique ? L’écart entre le contenu dynamique et le contenu statique a un impact sur la quantité de données à récupérer à partir de la couche Données par rapport à la quantité de données mises en cache dans le réseau CDN.

<!-- links -->
[architecture]: ./media/architecture-secure-scalable-wordpress.png
[mariadb-tutorial]: /azure/virtual-machines/linux/classic/mariadb-mysql-cluster
[docs-vmss]: /azure/virtual-machine-scale-sets/overview
[docs-vmss-autoscale]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview
[docs-nsg]: /azure/virtual-network/security-overview
[security]: /azure/security/
[availability]: ../../checklist/availability.md
[resiliency]: /azure/architecture/resiliency/
[scalability]: /azure/architecture/checklist/scalability
[pricing]: https://azure.com/e/a8c4809dab444c1ca4870c489fbb196b
