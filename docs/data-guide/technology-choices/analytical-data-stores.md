---
title: Sélectionner un magasin de données analytique
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: 166361c73a3a9c812e07445f6b039e843e5e32f8
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902329"
---
# <a name="choosing-an-analytical-data-store-in-azure"></a>Sélectionner un magasin de données analytique dans Azure

Une architecture [Big Data](../big-data/index.md) nécessite souvent un magasin de données analytique qui fournit les données traitées dans un format structuré qui peut être interrogé à l’aide des outils d’analyse. Les magasins de données analytiques prenant en charge l’interrogation des données provenant d’un chemin relatif et d’un chemin à froid sont collectivement appelés couche service ou stockage de service des données.

La couche service gère les données traitées à partir du chemin réactif et du chemin à froid. Dans l[’architecture lambda](../big-data/index.md#lambda-architecture), la couche service est divisée en une couche de _service vitesse_, qui stocke les données qui ont été traitées de façon incrémentielle, et en une couche de _service par lots_, contenant la sortie traitée par lots. La couche service requiert une prise en charge des lectures aléatoires à faible latence. Le stockage des données de la couche vitesse doit également prendre en charge les écritures aléatoires, car le chargement par lots des données dans ce magasin entraînerait des délais indésirables. En revanche, le stockage des données de la couche de traitement par lots n’a pas besoin de prendre en charge les écritures aléatoires, mais plutôt les écritures par lots.

Il n’existe pas de choix optimal pour la gestion des données de toutes les tâches de stockage de données. Chaque solution de gestion des données est optimisée pour différentes tâches. La plupart des applications cloud du monde réel et des processus Big Data dépendent de diverses exigences en matière de stockage de données, et utilisent souvent une combinaison de solutions de stockage de données.

## <a name="what-are-your-options-when-choosing-an-analytical-data-store"></a>Quelles sont vos options lors du choix d’un magasin de données analytique ?

Il existe plusieurs options pour le stockage de service des données dans Azure, en fonction de vos besoins :

- [SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [Base de données SQL Azure](/azure/sql-database/)
- [SQL Server dans les machines virtuelles Azure](/sql/sql-server/sql-server-technical-documentation)
- [HBase/Phoenix sur HDInsight](/azure/hdinsight/hbase/apache-hbase-overview)
- [Hive LLAP sur HDInsight](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)
- [Azure Analysis Services](/azure/analysis-services/analysis-services-overview)
- [Azure Cosmos DB](/azure/cosmos-db/)

Ces options offrent divers modèles de base de données optimisés pour différents types de tâches :

- Les bases de données [Clé/valeur](https://msdn.microsoft.com/library/dn313285.aspx#sec7) contiennent un objet sérialisé unique pour chaque valeur de clé. Elles sont parfaites pour le stockage de grands volumes de données lorsque vous souhaitez obtenir un élément pour une valeur de clé donnée et que vous n’avez pas besoin d’interroger les données en fonction d’autres propriétés de l’élément.
- Les bases de données [Document](https://msdn.microsoft.com/library/dn313285.aspx#sec8) sont des bases de données Clé/valeur dans lesquelles les valeurs sont des *documents*. Dans ce contexte, un « document » est une collection de champs et de valeurs nommés. La base de données stocke généralement les données dans un format comme XML, YAML, JSON ou BSON, mais également au format texte brut. Les bases de données Document peuvent lancer des requêtes sur des champs non-clés et définir des index secondaires pour améliorer l’efficacité des requêtes. Une base de données Document est ainsi plus adaptée aux applications qui doivent récupérer des données en fonction de critères plus complexes que la valeur de la clé du document. Par exemple, vous pouvez lancer une requête sur des champs tels que l’ID de produit, l’ID de client ou le nom du client.
- Les bases de données [Famille de colonnes](https://msdn.microsoft.com/library/dn313285.aspx#sec9) sont des magasins de données Clé/valeur qui structurent le stockage des données dans des collections de colonnes liées appelées familles de colonnes. Par exemple, une base de données de recensement peut contenir un groupe de colonnes pour le nom de la personne (prénom, deuxième prénom et nom), un groupe pour l’adresse de la personne, et un groupe pour les informations de profil de la personne (date de naissance, sexe). La base de données peut stocker chaque famille de colonnes dans une partition distincte, tout en associant l’ensemble des données d’une personne à la même clé. Une application peut lire une famille de colonnes unique sans parcourir toutes les données d’une entité.
- Les bases de données de [graphiques](https://msdn.microsoft.com/library/dn313285.aspx#sec10) stockent des informations comme une collection d’objets et des relations. Une base de données de graphiques peut lancer efficacement des requêtes qui parcourent le réseau d’objets et les relations entre eux. Par exemple, les objets peuvent représenter les employés d’une base de données de ressources humaines, et vous pouvez définir des requêtes comme « rechercher tous les employés travaillant directement ou indirectement pour Scott. »

## <a name="key-selection-criteria"></a>Critères de sélection principaux

Pour restreindre les choix, commencez par répondre à ces questions :

- Avez-vous besoin d’un stockage de service pouvant servir de chemin réactif pour vos données ? Si oui, limitez vos options à celles qui sont optimisées pour une couche de service vitesse.

- Avez-vous besoin de la prise en charge de l’architecture Massively Parallel Processing (MPP), où les requêtes sont automatiquement réparties entre plusieurs processus ou nœuds ? Si oui, sélectionnez une option prenant en charge l’augmentation de la taille des instances de la requête.

- Vous préférez utiliser un magasin de données relationnelles ? Dans ce cas, limitez vos options à celles proposant un modèle de base de données relationnelle. Notez cependant que certains magasins de données non relationnelles prennent en charge la syntaxe SQL pour l’interrogation, et des outils comme PolyBase peuvent servir à interroger des magasins de données non relationnelles.

## <a name="capability-matrix"></a>Matrice des fonctionnalités

Les tableaux suivants résument les principales différences entre les fonctionnalités.

### <a name="general-capabilities"></a>Fonctionnalités générales

| | Base de données SQL | SQL Data Warehouse | HBase/Phoenix sur HDInsight | Hive LLAP sur HDInsight | Azure Analysis Services | Cosmos DB |
| --- | --- | --- | --- | --- | --- | --- |
| Est un service géré | Oui | Oui | Oui <sup>1</sup> | Oui <sup>1</sup> | Oui | Oui |
| Modèle de base de données primaire | Relationnel (format en colonnes lors de l’utilisation des index columnstore) | Tables relationnelles avec stockage en colonnes | Stockage de colonnes larges | Hive/In-Memory | Modèles sémantiques MOLAP/tabulaires | Stockage de documents, graphiques, stockage de clé-valeur, stockage de colonnes larges |
| Prise en charge du langage SQL | Oui | Oui | Oui (à l’aide du pilote JDBC [Phoenix](https://phoenix.apache.org/)) | Oui | Non  | Oui |
| Optimisé pour la couche de service vitesse | Oui <sup>2</sup> | Non  | OUI | Oui | Non  | Oui |

[1] Avec mise à l’échelle et configuration manuelles.

[2] Avec un hachage et des tables à mémoire optimisée ou des index non cluster.
 
### <a name="scalability-capabilities"></a>Fonctionnalités d’évolutivité

|                                                  | Base de données SQL | SQL Data Warehouse | HBase/Phoenix sur HDInsight | Hive LLAP sur HDInsight | Azure Analysis Services | Cosmos DB |
|--------------------------------------------------|--------------|--------------------|----------------------------|------------------------|-------------------------|-----------|
| Serveurs régionaux redondants pour assurer une haute disponibilité |     Oui      |        OUI         |            Oui             |           Non            |           Non             |    Oui    |
|             Prend en charge l’augmentation de la taille des instances de la requête             |      Non       |        OUI         |            OUI             |          OUI           |           OUI           |    Oui    |
|          Évolutivité dynamique (montée en puissance)          |     Oui      |        Oui         |             Non              |           Non            |           OUI           |    Oui    |
|        Prend en charge la mise en cache en mémoire des données        |     Oui      |        Oui         |             Non              |          OUI           |           Oui           |    Non      |

### <a name="security-capabilities"></a>Fonctionnalités de sécurité

| | Base de données SQL | SQL Data Warehouse | HBase/Phoenix sur HDInsight | Hive LLAP sur HDInsight | Azure Analysis Services | Cosmos DB |
| --- | --- | --- | --- | --- | --- | --- |
| Authentification  | SQL / Azure Active Directory (Azure AD) | SQL / Azure AD | local / Azure AD<sup>1</sup> | local / Azure AD <sup>1</sup> | Azure AD | utilisateurs de base de données / Azure AD via un contrôle d’accès (IAM) |
| Chiffrement des données au repos | Oui <sup>2</sup> | Oui <sup>2</sup> | Oui <sup>1</sup> | Oui <sup>1</sup> | Oui | Oui |
| Sécurité au niveau des lignes | Oui | Non  | Oui <sup>1</sup> | Oui <sup>1</sup> | Oui (via la sécurité au niveau de l’objet dans le modèle) | Non  |
| Prend en charge les pare-feu | Oui | Oui | Oui <sup>3</sup> | Oui <sup>3</sup> | Oui | Oui |
| Masquage des données dynamiques | Oui | Non  | Oui <sup>1</sup> | Oui * | Non  | Non  |

[1] Suppose d’utiliser un [cluster HDInsight joint à un domaine](/azure/hdinsight/domain-joined/apache-domain-joined-introduction).

[2] Nécessite l’utilisation du chiffrement transparent des données (TDE) pour chiffrer et déchiffrer vos données au repos.

[3] Si utilisé au sein d’un réseau virtuel Azure. Voir [Étendre HDInsight à l’aide d’un réseau virtuel Azure](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network).
