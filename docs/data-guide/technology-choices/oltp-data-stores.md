---
title: Choisir un magasin de données OLTP
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 1c27d7d5f3b78f40822de6b77664dbf49b1367f6
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/31/2018
---
# <a name="choosing-an-oltp-data-store-in-azure"></a>Choisir un magasin de données OLTP dans Azure

Le traitement transactionnel en ligne (OLTP) est la gestion des données transactionnelles et du traitement transactionnel. Cette rubrique compare des options des solutions OLTP dans Azure.

> [!NOTE]
> Pour plus d’informations sur l’utilisation d’un magasin de données OLTP, consultez [Traitement transactionnel en ligne](../scenarios/online-analytical-processing.md).

## <a name="what-are-your-options-when-choosing-an-oltp-data-store"></a>Quelles sont vos options lors du choix d’un magasin de données OLTP ?

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
| | Base de données SQL Azure | SQL Server sur une machine virtuelle Azure | Base de données Azure pour MySQL | Base de données Azure pour PostgreSQL |
| --- | --- | --- | --- | --- | --- |
| Est un service géré | OUI | Non  | OUI | OUI |
| S’exécute sur la plateforme | N/A | Windows, Linux, Docker | N/A | N/A |
| Programmabilité <sup>1</sup> | T-SQL, .NET, R | T-SQL, .NET, R, Python | T-SQL, .NET, R, Python | SQL | SQL |

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
| | Base de données SQL Azure | SQL Server sur une machine virtuelle Azure| Base de données Azure pour MySQL | Base de données Azure pour PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| Sécurité au niveau des lignes | OUI | OUI | OUI | OUI |
| Masquage de données | OUI | OUI | Non  | Non  |
| Chiffrement transparent des données | OUI | OUI | OUI | OUI |
| Restreindre l’accès à des adresses IP spécifiques | OUI | OUI | OUI | OUI |
| Restreindre l’accès pour autoriser l’accès au seul réseau virtuel | OUI | OUI | Non  | Non  |
| Authentification Azure Active Directory | OUI | OUI | Non  | Non  |
| Authentification Active Directory | Non  | OUI | Non  | Non  |
| Authentification multifacteur | OUI | OUI | Non  | Non  |
| Prend en charge [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine) | OUI | OUI | OUI | Non  | Non  |
| IP privée | Non  | OUI | OUI | Non  | Non  |

