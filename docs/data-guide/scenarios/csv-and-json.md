---
title: Traitement des fichiers CSV et JSON
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 02e684d562cfe555f9e3596ad0a2f1a00d05c7a7
ms.sourcegitcommit: 51f49026ec46af0860de55f6c082490e46792794
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/03/2018
ms.locfileid: "30298607"
---
# <a name="working-with-csv-and-json-files-for-data-solutions"></a>Utilisation des fichiers CSV et JSON pour des solutions de données

CSV et JSON sont probablement les formats les plus fréquemment utilisés pour l’ingestion, l’échange et le stockage des données non structurées ou semi-structurées. 

## <a name="about-csv-format"></a>À propos du format CSV

Les fichiers CSV (valeurs séparées par des virgules) sont fréquemment utilisés pour échanger des données tabulaires entre les systèmes en texte brut. Ils contiennent généralement une ligne d’en-tête qui fournit les noms des colonnes pour les données, mais sont autrement considérés comme étant semi-structurés. Cela est dû au fait que les fichiers CSV ne peuvent pas représenter naturellement des données hiérarchiques ou relationnelles. Les relations entre les données sont généralement gérées avec plusieurs fichiers CSV, où les clés étrangères sont stockées dans des colonnes d’un ou plusieurs fichiers, mais les relations entre ces fichiers ne sont pas exprimées par le format de lui-même. Les fichiers au format CSV peuvent utiliser d’autres délimiteurs autres que des virgules, notamment des tabulations ou des espaces.

En dépit de leurs limitations, les fichiers CSV sont fréquemment utilisés pour l’échange de données, car ils sont pris en charge par un large éventail d’entreprises, de consommateurs et d’applications scientifiques. Par exemple, les programmes de bases de données et de feuilles de calcul peuvent importer et exporter des fichiers CSV. De même, la plupart des moteurs de traitement par lot et de flux de données, tels que Spark et Hadoop, prennent en charge en mode natif des fichiers au format CSV sérialisés et désérialisés et offrent plusieurs moyens d’appliquer un schéma lors de la lecture. Cela simplifie l’utilisation des données, en offrant des options de requête les concernant et le stockage des informations dans un format de données plus efficace pour accélérer le traitement.

## <a name="about-json-format"></a>À propos du format JSON

Les données JSON (JavaScript Object Notation) sont représentées sous forme de paires clé-valeur dans un format semi-structuré. Le format JSON est souvent comparé au format XML, puisque les deux sont capables de stocker des données dans un format hiérarchique, avec les données enfant représentées en ligne avec son parent. Les deux sont auto-descriptifs et lisibles par l’utilisateur, mais les documents JSON ont tendance à être beaucoup plus petits, conduisant à leur utilisation fréquente dans l’échange de données en ligne, en particulier avec l’avènement des services Web basés sur REST. 

Les fichiers au format JSON présentent de nombreux avantages sur le format CSV :

* JSON maintient les structures hiérarchiques, permettant une meilleure conservation des données connexes dans un seul document en représentant des relations complexes.
* La plupart des langages de programmation prennent en charge la désérialisation JSON en objets en mode natif ou fournissent des bibliothèques de sérialisation JSON légères.
* JSON prend en charge les listes d’objets, contribuant à éviter les translations désordonnées des listes en un modèle de données relationnelles.
* JSON est un format de fichier fréquemment utilisé pour les bases de données NoSQL, tels que MongoDB, Couchbase et Azure Cosmos DB.

Dans la mesure où un grand nombre de données entrantes sur le réseau est déjà au format JSON, la plupart des langages de programmation sur Web prennent en charge l’utilisation de JSON en mode natif ou via l’utilisation des bibliothèques externes pour sérialiser et désérialiser des données JSON. Cette prise en charge universelle de JSON a conduit à son utilisation dans des formats logiques via la représentation sous forme de structure de données, formats d’échange de données à chaud et le stockage de données des données à froid.

De nombreux moteurs du traitement par lot et de flux de données prend en charge en mode natif la désérialisation et la sérialisation JSON. Bien que les données contenues dans les documents JSON puissent finalement être stockées dans des formats plus performants, tels que Parquet ou Avro, elles servent de données brutes à la source véritable, ce qui est essentielle pour retraiter les données en fonction des besoins.

## <a name="when-to-use-csv-or-json-formats"></a>Quand utiliser des formats CSV ou JSON

Les fichiers CSV sont fréquemment utilisés pour l’exportation et l’importation des données ou leur traitement pour l’analytique et le Machine Learning. Les fichiers au format JSON ont les mêmes avantages, mais sont plus fréquents dans les solutions d’échange de données à chaud. Les documents JSON sont souvent envoyés par des appareils mobiles et Web exécutant des transactions en ligne, par des appareils IoT (internet des objets) pour une communication à sens unique ou bidirectionnelle ou par des applications clientes qui communiquent avec les services SaaS et PaaS ou des architectures sans serveur. 

Les formats de fichiers CSV et JSON facilitent tous les deux les échanges des données entre des systèmes ou des appareils différents. Leurs formats semi-structurés permettent une grande souplesse lors du transfert de presque tous les types de données et la prise en charge universelle de ces formats rendent l’utilisation plus facile. Ils peuvent tous les deux servir de source brute véritable dans les cas où les données traitées sont stockées dans des formats binaires pour des interrogations plus efficaces. 

## <a name="working-with-csv-and-json-data-in-azure"></a>Utilisation des données CSV et JSON dans Azure

Azure offre plusieurs solutions pour l’utilisation des fichiers CSV et JSON, en fonction de vos besoins. L’emplacement de lancement principale de ces fichiers est le stockage Azure ou Azure Data Lake Store. La plupart des services Azure qui fonctionnent avec ces fichiers et autres fichiers texte s’intègrent à un des services de stockage d’objets. Toutefois, dans certaines situations, vous pouvez opter pour importer directement les données dans Azure SQL ou un autre magasin de données. SQL Server dispose d’une prise en charge native pour le stockage et l’utilisation de documents JSON, ce qui simplifie [l’importation et le traitement de ces types de fichiers](/sql/relational-databases/json/import-json-documents-into-sql-server). Vous pouvez utiliser un utilitaire tel que l’importation en bloc SQL pour [importer des fichiers CSV](/sql/relational-databases/json/import-json-documents-into-sql-server) facilement.

Selon le scénario, vous pouvez effectuer le [traitement par lots](../big-data/batch-processing.md) ou le [traitement en temps réel](../big-data/real-time-processing.md) des données.

## <a name="challenges"></a>Défis

L’utilisation de ces formats présente certains défis à prendre en compte :

* Sans restriction du modèle de données, les fichiers CSV et JSON sont sujets à une altération des données (« garbage in, garbage out »). Par exemple, il n’existe aucune notion d’objet date/heure dans l’un ou l’autre fichier, ainsi le format du fichier n’empêche pas l’insertion de « ABC123 » dans un champ de date, par exemple.

* L’utilisation de fichiers CSV et JSON en tant que solution de stockage à froid n’est pas bien adaptée à l’utilisation du Big Data. Dans la plupart des cas, ils ne peuvent pas être fractionnés en partitions pour un traitement parallèle et ne peuvent pas être compressés aussi bien que des formats binaires. Cela conduit souvent au traitement et au stockage de ces données dans des formats optimisés pour la lecture tels que Parquet et ORC (ligne optimisée en colonnes), lesquels fournissent également des index et statistiques en ligne sur les données contenues.

* Vous pouvez avoir besoin d’appliquer un schéma sur les données semi-structurées pour faciliter la requête et l’analyse. En règle générale, cela requiert le stockage des données sous une autre forme qui convient aux besoins de stockage de données de votre environnement, comme dans une base de données.

