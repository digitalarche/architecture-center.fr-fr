---
title: Partitionner pour contourner les limites
description: "Utilisez le partitionnement pour contourner les limites liées à la base de données, au réseau et au calcul"
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 4371490385b24230551bf17db0075052f320b574
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2017
---
# <a name="partition-around-limits"></a>Partitionner pour contourner les limites

## <a name="use-partitioning-to-work-around-database-network-and-compute-limits"></a>Utilisez le partitionnement pour contourner les limites liées à la base de données, au réseau et au calcul

Dans le cloud, la capacité de montée en puissance de la totalité des services présente certaines limites. Les limites des services Azure sont documentées dans l’article [Abonnement Azure et limites, quotas et contraintes de service][azure-limits]. Les limites portent sur le nombre de cœurs, la taille de base de données, le débit des requêtes et le débit du réseau. Si votre système s’étend suffisamment, vous risquez d’atteindre une ou plusieurs de ces limites. Utilisez le partitionnement pour contourner ces limites.

Vous disposez de nombreuses méthodes pour partitionner un système, notamment :

- Partitionnez une base de données pour éviter les limites de taille de base de données, d’E/S de données ou de nombre de sessions simultanées.

- Partitionnez une file d’attente ou un bus de messages afin d’éviter les limites relatives au nombre de requêtes ou de connexions simultanées.

- Partitionnez une application web App Service pour contourner les limites du nombre d’instances par plan App Service. 

Une base de données peut être partitionnée *horizontalement*, *verticalement* ou *fonctionnellement*.

- Dans le cadre du partitionnement horizontal, également appelé partitionnement de base de données, chaque partition stocke un sous-ensemble de données du jeu de données total. Les partitions partagent le même schéma de données. Par exemple, les clients dont le nom commence par A&ndash;M sont stockés dans une partition, et ceux dont le nom commence par N&ndash;Z sont placés dans une autre partition.

- Dans le cadre d’un partitionnement vertical, chaque partition comporte un sous-ensemble des champs relatifs aux éléments présents dans le magasin de données. Par exemple, placez les champs fréquemment utilisés dans une partition, et les champs plus rarement utilisés dans une autre.

- Lors d’un partitionnement fonctionnel, les données sont partitionnées en fonction de leur mode d’utilisation par chaque contexte lié dans le système. Par exemple, stockez les données de facturation dans une partition et les données d’inventaire de produits dans une autre. Les schémas sont indépendants.

Pour plus d’informations, consultez l’article [Partitionnement des données][data-partitioning-guidance].

## <a name="recommendations"></a>Recommandations

**Partitionnez les différentes parties de l’application**. Si les bases de données constituent l’un des choix les plus évidents pour le partitionnement, envisagez également de partitionner le stockage, le cache, les files d’attente et les instances de calcul.

**Concevez la clé de partition pour éviter les zones réactives**. Si vous partitionnez une base de données, mais qu’une seule partition reçoit la majorité des requêtes, vous n’avez pas résolu votre problème. Dans l’idéal, la charge est répartie uniformément entre toutes les partitions. Par exemple, hachez les données par ID client plutôt qu’en fonction de la première lettre du nom du client, car certaines lettres sont plus fréquentes. Le même principe s’applique lors du partitionnement d’une file d’attente de messages. Sélectionnez une clé de partition entraînant une répartition uniforme des messages dans l’ensemble des files d’attente. Pour plus d’informations, consultez l’article [Partitionnement][sharding].

**Partitionnez pour contourner les limites de service et d’abonnement Azure**. Les composants et les services ne sont pas les seuls à présenter des limites ; c’est également le cas des abonnements et des groupes de ressources. Dans le cas des applications très volumineuses, vous pouvez avoir besoin de créer des partitions pour contourner ces limites.  

**Partitionnez à différents niveaux**. Considérez l’exemple d’un serveur de base de données déployé sur une machine virtuelle. Cette machine virtuelle est équipée d’un disque dur virtuel qui est sauvegardé par le service Stockage Azure. Le compte de stockage appartient à un abonnement Azure. Notez que chaque niveau de la hiérarchie comporte des limites. Le serveur de base de données peut présenter une limite de pool de connexions. Les machines virtuelles comportent des limites d’UC et de réseau. Le stockage présente des limites d’IOPS. L’abonnement fait l’objet de limites concernant le nombre de cœurs de machine virtuelle. Il est généralement plus facile de créer des partitions à un niveau inférieur de la hiérarchie. Seules les applications volumineuses nécessitent un partitionnement au niveau abonnement. 

<!-- links -->

[azure-limits]: /azure/azure-subscription-service-limits
[data-partitioning-guidance]: ../../best-practices/data-partitioning.md
[sharding]: ../../patterns/sharding.md

 