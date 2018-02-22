---
title: "Choisir un magasin de données de recherche"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 7fe5952c880921984beb30c71458fd1ef72ef239
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-search-data-store-in-azure"></a>Choisir un magasin de données de recherche dans Azure

Cet article compare les choix technologiques pour les magasins de données de recherche dans Azure. Un magasin de données de recherche est utilisé pour créer et stocker des index spécialisés afin d’effectuer des recherches sur du texte de forme libre. Le texte qui est indexé peut résider dans un magasin de données distinct, comme le stockage blob. Une application soumet une requête au magasin de données de recherche, et le résultat est une liste de documents correspondants. Pour plus d’informations sur ce scénario, consultez [Traitement de texte de forme libre pour la recherche](../scenarios/search.md). 

## <a name="what-are-your-options-when-choosing-a-search-data-store"></a>Quelles sont vos options lors du choix d’un magasin de données de recherche ?
Dans Azure, tous les magasins de données suivants respectent les exigences principales pour la recherche sur des données de texte de forme libre en fournissant un index de recherche :
- [Recherche Azure](/azure/search/search-what-is-azure-search)
- [Elasticsearch](https://azuremarketplace.microsoft.com/marketplace/apps/elastic.elasticsearch?tab=Overview)
- [HDInsight avec Solr](/azure/hdinsight/hdinsight-hadoop-solr-install-linux)
- [Azure SQL Database avec recherche en texte intégral](/sql/relational-databases/search/full-text-search)


## <a name="key-selection-criteria"></a>Critères de sélection principaux

Pour les scénarios de recherche, commencez par choisir le magasin de données de recherche approprié pour vos besoins en répondant à ces questions :

- Voulez-vous un service managé plutôt que de gérer vos propres serveurs ?

- Pouvez-vous spécifier votre schéma d’index au moment de la conception ? Dans le cas contraire, choisissez une option qui prend en charge les schémas pouvant être mis à jour.

- Avez-vous besoin d’un index uniquement pour la recherche en texte intégral ou avez-vous également besoin de l’agrégation rapide de données numériques et autres outils analytiques ? Si vous avez besoin de fonctionnalités au-delà de la recherche en texte intégral, envisagez les options qui prennent en charge une analytique supplémentaire.

- Avez-vous besoin d’un index de recherche pour l’analytique des journaux, avec prise en charge de la collecte de journaux, de l’agrégation et des visualisations sur les données indexées ? Dans ce cas, envisagez d’utiliser Elasticsearch, qui fait partie d’une pile d’analytique des journaux.

- Avez-vous besoin d’indexer les données dans des formats de documents courants tels que PDF, Word, PowerPoint et Excel ? Si la réponse est Oui, choisissez une option qui fournit des indexeurs de documents.

- Votre base de données a-t-elle des besoins de sécurité spécifiques ? Si la réponse est Oui, consultez les fonctionnalités de sécurité ci-dessous.

## <a name="capability-matrix"></a>Matrice des fonctionnalités

Les tableaux suivants résument les principales différences entre les fonctionnalités.

### <a name="general-capabilities"></a>Fonctionnalités générales
| | Recherche Azure | Elasticsearch | HDInsight avec Solr | Base de données SQL | 
| --- | --- | --- | --- | --- | 
| Un service managé | OUI | Non  | OUI | OUI |  
| de l’API REST | OUI | OUI | OUI | Non  |
| Programmabilité | .NET | Java | Java | T-SQL | 
| Indexeurs de documents pour les types de fichiers courants (PDF, DOCX, TXT, etc.) | OUI | Non  | OUI | Non  |

### <a name="manageability-capabilities"></a>Fonctionnalités de facilité de gestion
| | Recherche Azure | Elasticsearch | HDInsight avec Solr | Base de données SQL | 
| --- | --- | --- | --- | --- |
| Schéma pouvant être mis à jour | Non  | OUI | OUI | OUI |
| Prend en charge la montée en puissance  | OUI | OUI | OUI | Non  |

### <a name="analytic-workload-capabilities"></a>Fonctionnalités de la charge de travail analytique
| | Recherche Azure | Elasticsearch | HDInsight avec Solr | SQL Databash | 
| --- | --- | --- | --- | --- | 
| Prend en charge l’analytique au-delà de la recherche en texte intégral | Non  | OUI | OUI | OUI |
| Fait partie d’une pile d’analytique des journaux | Non  | Oui (ELK) |  Non  | Non  |
| Prend en charge la recherche sémantique | Oui (rechercher uniquement les documents similaires) | OUI | OUI | OUI | 

### <a name="security-capabilities"></a>Fonctionnalités de sécurité
| | Recherche Azure | Elasticsearch | HDInsight avec Solr | SQL Databash | 
| --- | --- | --- | --- | --- | 
| Sécurité au niveau des lignes | Partielle (requête d’application requise pour filtrer par ID de groupe) | Partielle (requête d’application requise pour filtrer par ID de groupe) | OUI | OUI | 
| Chiffrement transparent des données | Non  | Non  | Non  | OUI |  
| Restreindre l’accès à des adresses IP spécifiques | Non  | OUI | OUI | OUI |   
| Restreindre l’accès pour autoriser l’accès au réseau virtuel uniquement | Non  | OUI | OUI | OUI |  
| Authentification Active Directory (authentification intégrée) | Non  | Non  | Non  | OUI | 

## <a name="see-also"></a>Voir aussi

[Traitement de texte de forme libre pour la recherche](../scenarios/search.md)