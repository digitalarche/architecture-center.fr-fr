---
title: Traitement transactionnel en ligne (OLTP)
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 8650b919fc1a59240343015493a1fe41c8729a72
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/06/2018
---
# <a name="online-transaction-processing-oltp"></a>Traitement transactionnel en ligne (OLTP)

La gestion des données transactionnelles à l’aide de systèmes informatiques est nommée Traitement transactionnel en ligne (OLTP). Les systèmes OLTP enregistrent les interactions de l’entreprise lorsqu’elles se produisent dans les opérations quotidiennes de l’organisation et prennent en charge l’interrogation de ces données pour faire des inférences.

## <a name="transactional-data"></a>Données transactionnelles

Les données transactionnelles sont des informations qui assurent le suivi des interactions relatives aux activités d’une organisation. Ces interactions sont généralement des transactions commerciales, tels que les paiements reçus des clients, les paiements effectués aux fournisseurs, le déplacement de produits dans le cadre de l’inventaire, les commandes prises ou les services fournis. Les événements transactionnels, qui représentent les transactions proprement dites, contiennent une dimension de temps, des valeurs numériques et des références à d’autres données. 

Les transactions doivent généralement être *atomiques* et *cohérentes*. Atomicité signifie qu’une transaction complète réussit ou échoue toujours en tant que seule unité de travail et n’est jamais laissée dans un état à moitié terminé. Si une transaction ne peut pas être effectuée, le système de base de données doit restaurer toutes les étapes qui ont déjà été effectuées dans le cadre de cette transaction. Dans un système de gestion de base de données relationnelle traditionnel, cette restauration a lieu automatiquement si une transaction ne peut pas être effectuée. Cohérence signifie que les transactions laissent toujours les données dans un état valide. (Il s’agit de descriptions très informelles de l’atomicité et de la cohérence. Il existe des définitions plus formelles de ces propriétés, telles qu’ [ACID](https://en.wikipedia.org/wiki/ACID).)

Les bases de données transactionnelles peuvent prendre en charge une cohérence forte pour les transactions grâce à différentes stratégies de verrouillage, tels que le verrouillage pessimiste, pour garantir que toutes les données sont fortement cohérentes dans le contexte de l’entreprise, pour tous les utilisateurs et les processus. 

L’architecture de déploiement la plus courante utilisant des données transactionnelles est le niveau de magasin de données dans une architecture à 3 couches. Une architecture à 3 couches se compose généralement d’une couche de présentation, d’une couche de logique métier et d’une couche de magasin de données. L’architecture [multiniveau](/azure/architecture/guide/architecture-styles/n-tier) est une architecture de déploiement associée pouvant avoir plusieurs couches intermédiaires qui gèrent la logique métier.

## <a name="typical-traits-of-transactional-data"></a>Caractéristiques des données transactionnelles classiques

Les données transactionnelles ont tendance à présenter les caractéristiques suivantes :

| Prérequis | Description |
| --- | --- |
| Normalisation | Très normalisées |
| Schéma | Schéma lors de l’écriture, fortement appliqué|
| Cohérence | Cohérence forte, garanties ACID |
| Intégrité | Intégrité élevée |
| Utilisent des transactions | OUI |
| Stratégie de verrouillage | Pessimiste ou optimiste|
| Peut être mise à jour | OUI |
| Modifiable | OUI |
| Charge de travail | Haut volume d’écriture, volume de lecture modéré |
| Indexation | Index primaires et secondaires |
| Taille de donnée | Petite à moyenne taille |
| Modèle | Relationnel |
| Forme des données | Tabulaire |
| Flexibilité de requête | Très flexible |
| Scale | Petite (Mo) à grande (quelques To) | 

## <a name="when-to-use-this-solution"></a>Quand utiliser cette solution ?

Choisissez OLTP lorsque vous avez besoin de traiter et stocker efficacement les transactions commerciales et les rendre immédiatement disponibles pour les applications clientes de manière cohérente. Utilisez cette architecture lorsque tout retard tangible dans le traitement peut avoir un impact négatif sur les opérations quotidiennes de l’entreprise.

Les systèmes OLTP sont conçus pour traiter et stocker efficacement des transactions, ainsi que les données transactionnelles de requête. L’objectif de traiter et de stocker efficacement des transactions individuelles par un système OLTP est atteint en partie par la normalisation des données &mdash; c’est-à-dire, le fractionnement des données en blocs plus petits qui sont moins redondants. Cela prend en charge l’efficacité en permettant au système OLTP de traiter indépendamment une grande quantité de transactions et en évitant le traitement supplémentaire nécessaire pour maintenir l’intégrité des données en présence de données redondantes.

## <a name="challenges"></a>Défis
La mise en œuvre et l’utilisation d’un système OLTP peuvent créer quelques défis :

- Les systèmes OLTP ne sont pas toujours appropriés pour gérer des agrégats sur de grandes quantités de données, bien qu’il existe des exceptions, notamment une solution SQL Server correctement planifiée. L’analytique des données qui s’appuient sur des calculs d’agrégation pour des millions de transactions individuelles exige, pour un système OLTP, de très nombreuses ressources. Elles peuvent s’exécuter lentement et entraîner un ralentissement en bloquant les autres transactions dans la base de données.
- Lors de la réalisation d’analyses et la création de rapports sur les données très normalisées, les requêtes ont tendance à être complexes, car la plupart doivent dénormaliser les données à l’aide de jonctions. En outre, les conventions d’affectation de noms pour les objets de base de données dans les systèmes OLTP ont tendance à être laconique et concise. La normalisation accrue couplée à des conventions d’affectation de noms laconiques complique l’interrogation des systèmes OLTP par les utilisateurs professionnels, sans l’aide d’un développeur de base de données ou développeur des données.
- Le stockage indéfini de l’historique des transactions et le stockage de trop de données dans une table peuvent entraîner le ralentissement des performances de requêtes, en fonction du nombre de transactions stockées. La solution courante consiste à maintenir une période pertinente (par exemple, l’année fiscale en cours) dans le système OLTP et décharger les données historiques vers d’autres systèmes, par exemple un mini-data Warehouse ou un [entrepôt de données](./data-warehousing.md).

## <a name="oltp-in-azure"></a>OLTP dans Azure

Les applications telles que les sites Web hébergés dans [App Service Web Apps](/azure/app-service/app-service-web-overview), les API REST en cours d’exécution dans l’App Service ou les applications de bureau ou mobiles communiquent avec le système OLTP, en général via une API REST intermédiaire.

Dans la pratique, la plupart des charges de travail ne sont pas purement OLTP. Généralement, elles comportent également un composant analytique. En outre, il existe une demande croissante de rapports en temps réel, telles que l’exécution des rapports à partir du système d’exploitation. Cela est également appelé HTAP (Traitement transactionnel et analytique hybride). Pour plus d’informations, consultez [Traitement analytique en ligne (OLAP)](./online-analytical-processing.md).

Dans Azure, tous les magasins de données suivants répondront aux principales exigences du traitement OLTP et de la gestion des données transactionnelles :

- [Azure SQL Database](/azure/sql-database/)
- [SQL Server sur une machine virtuelle Azure](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-iaas-overview?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json)
- [Azure Database pour MySQL](/azure/mysql/)
- [Base de données Azure pour PostgreSQL](/azure/postgresql/)

## <a name="key-selection-criteria"></a>Critères de sélection principaux

Pour restreindre les choix, commencez par répondre aux questions suivantes :

- Préférez-vous opter pour un service géré plutôt que de gérer vos propres serveurs ?

- Votre solution a-t-elle des dépendances spécifiques pour la compatibilité de Microsoft SQL Server, MySQL ou PostgreSQL ? Votre application peut limiter les magasins de données que vous pouvez choisir en fonction des pilotes pris en charge pour la communication avec le magasin de données, ou du magasin de données supposé être utilisé.

- Vos exigences de débit d’écriture sont-elles particulièrement élevées ? Si oui, choisissez une option qui fournit des tables en mémoire. 

- Votre solution est-elle mutualisée ? Dans ce cas, envisagez des options prenant en charge des pools de capacité, où plusieurs instances de base de données proviennent d’un pool élastique de ressources, et non de ressources fixes par base de données. Ceci vous permet de mieux distribuer la capacité sur toutes les instances de base de données et d’améliorer la rentabilité de votre solution.

- Vos données doivent-elles être lisibles avec une faible latence dans plusieurs régions ? Si oui, choisissez une option prenant en charge des réplicas secondaires lisibles.

- Votre base de données doit-elle être hautement disponible dans plusieurs régions géographiques ? Si oui, choisissez une option prenant en charge la réplication géographique. Envisagez également les options qui prennent en charge le basculement automatique du réplica principal vers un réplica secondaire.

- Votre base de données a-t-elle des besoins de sécurité spécifiques ? Si oui, examinez les options qui fournissent des fonctionnalités telles que la sécurité au niveau des lignes, le masquage de données et le chiffrement transparent des données.

## <a name="capability-matrix"></a>Matrice des fonctionnalités

Les tableaux suivants résument les principales différences entre les fonctionnalités.

### <a name="general-capabilities"></a>Fonctionnalités générales 

|                              | Base de données SQL Azure | SQL Server sur une machine virtuelle Azure | Base de données Azure pour MySQL | Base de données Azure pour PostgreSQL |
|------------------------------|--------------------|----------------------------------------|--------------------------|-------------------------------|
|      Est un service géré      |        OUI         |                   Non                    |           OUI            |              OUI              |
|       S’exécute sur la plateforme       |        N/A         |         Windows, Linux, Docker         |           N/A            |              N/A              |
| Programmabilité <sup>1</sup> |   T-SQL, .NET, R   |         T-SQL, .NET, R, Python         |  T-SQL, .NET, R, Python  |              SQL              |

[1] sans prise en charge des pilotes client, ce qui permet de se connecter à de nombreux langages de programmation et d’utiliser le magasin de données OLTP.

### <a name="scalability-capabilities"></a>Fonctionnalités d’évolutivité

| | Base de données SQL Azure | SQL Server sur une machine virtuelle Azure| Base de données Azure pour MySQL | Base de données Azure pour PostgreSQL|
| --- | --- | --- | --- | --- | --- |
| Taille maximale de l’instance de la base de données | [4 To](/azure/sql-database/sql-database-resource-limits) | 256 To | [1 To](/azure/mysql/concepts-limits) | [1 To](/azure/postgresql/concepts-limits) |
| Prend en charge les pools de capacité  | OUI | OUI | Non  | Non  |
| Prend en charge la mise à l’échelle des clusters  | Non  | OUI | Non  | Non  |
| Évolutivité dynamique (montée en puissance)  | OUI | Non  | OUI | OUI |

### <a name="analytic-workload-capabilities"></a>Fonctionnalités de la charge de travail analytique

| | Base de données SQL Azure | SQL Server sur une machine virtuelle Azure| Base de données Azure pour MySQL | Base de données Azure pour PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| Tables temporelles | OUI | OUI | Non  | Non  |
| Tables en mémoire (optimisées en mémoire) | OUI | OUI | Non  | Non  |
| Prise en charge de ColumnStore | OUI | OUI | Non  | Non  |
| Traitement adaptatif des requêtes | OUI | OUI | Non  | Non  |

### <a name="availability-capabilities"></a>Fonctionnalités de disponibilité

| | Base de données SQL Azure | SQL Server sur une machine virtuelle Azure| Base de données Azure pour MySQL | Base de données Azure pour PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| Bases de données secondaires accessibles en lecture | OUI | OUI | Non  | Non  | 
| Réplication géographique | OUI | OUI | Non  | Non  | 
| Basculement automatique vers la base de données secondaire | OUI | Non  | Non  | Non |
| Limite de restauration dans le temps | OUI | OUI | OUI | OUI |

### <a name="security-capabilities"></a>Fonctionnalités de sécurité

|                                                                                                             | Base de données SQL Azure | SQL Server sur une machine virtuelle Azure | Base de données Azure pour MySQL | Base de données Azure pour PostgreSQL |
|-------------------------------------------------------------------------------------------------------------|--------------------|----------------------------------------|--------------------------|-------------------------------|
|                                             Sécurité au niveau des lignes                                              |        OUI         |                  OUI                   |           OUI            |              OUI              |
|                                                Masquage de données                                                 |        OUI         |                  OUI                   |            Non             |              Non                |
|                                         Chiffrement transparent des données                                         |        OUI         |                  OUI                   |           OUI            |              OUI              |
|                                  Restreindre l’accès à des adresses IP spécifiques                                   |        OUI         |                  OUI                   |           OUI            |              OUI              |
|                                  Restreindre l’accès pour autoriser l’accès au seul réseau virtuel                                  |        OUI         |                  OUI                   |            Non             |              Non                |
|                                    Authentification Azure Active Directory                                    |        OUI         |                  OUI                   |            Non             |              Non                |
|                                       Authentification Active Directory                                       |         Non          |                  OUI                   |            Non             |              Non                |
|                                         Authentification multifacteur                                         |        OUI         |                  OUI                   |            Non             |              Non                |
| Prend en charge [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine) |        OUI         |                  OUI                   |           OUI            |              Non                |
|                                                 IP privée                                                  |         Non          |                  OUI                   |           OUI            |              Non                |

