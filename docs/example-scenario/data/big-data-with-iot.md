---
title: L’IoT et l’analyse de données dans le secteur de la construction
description: Utilisez des appareils IoT et l’analyse des données pour fournir une gestion et une opération complètes des projets de construction.
author: alexbuckgit
ms.date: 08/29/2018
ms.openlocfilehash: 74868191687e63a54a69fdacb7276983d98faf74
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610921"
---
# <a name="iot-and-data-analytics-in-the-construction-industry"></a>L’IoT et l’analyse de données dans le secteur de la construction

Cet exemple de scénario s’applique aux organisations qui créent des solutions intégrant des données émanant de nombreux appareils IoT (Internet of Things, Internet des objets) au sein d’une architecture d’analyse des données complète afin d’améliorer et d’automatiser le processus de prise de décision. Les applications potentielles de ce scénario comprennent les solutions de construction, d’extraction, de fabrication ou d’autres secteurs d’activité qui impliquent d’importants volumes de données provenant d’une multitude d’entrées de données basées sur IoT.

Dans ce scénario, un fabricant d’équipements de construction construit des véhicules, des compteurs et des drones qui utilisent les technologies IoT et GPS pour émettre des données de télémétrie. La société souhaite moderniser son architecture de données afin de superviser plus étroitement les conditions d’utilisation et l’intégrité de l’équipement. Le remplacement de la solution existante de la société en utilisant l’infrastructure locale mobiliserait trop de temps et d’efforts et ne constituerait pas une évolution suffisante pour traiter le volume de données attendu.

La société souhaite créer une solution de « construction intelligente » basée sur le cloud. Elle doit réunir un jeu de données complet concernant un site de construction et automatiser l’exploitation et la maintenance des différents éléments de ce site. Les objectifs de la société sont les suivants :

* intégration et analyse de la totalité des équipements et des données du site de construction afin de réduire le temps d’arrêt des équipements et de limiter les risques de vol ;
* contrôle à distance et automatique des équipements de construction en vue d’atténuer les effets d’une pénurie de main-d’œuvre, entraînant au final une diminution du nombre d’ouvriers requis et du niveau de qualification exigé ;
* abaissement des coûts d’exploitation et des exigences de main-d’œuvre pour l’infrastructure sous-jacente, tout en améliorant la productivité et la sécurité ;
* mise à l’échelle de l’infrastructure en toute facilité afin de prendre en charge l’augmentation du volume de données de télémétrie ;
* conformité à l’ensemble des exigences légales appropriées par l’approvisionnement de ressources à l’échelle du pays sans compromettre la disponibilité du système ;
* utilisation de logiciels open source afin d’optimiser l’investissement dans les qualifications actuelles des employés.

L’utilisation des services Azure gérés tels qu’IoT Hub et HDInsight permet au client de créer et déployer rapidement une solution complète avec un coût d’exploitation moindre. Si vous avez d’autres besoins en matière d’analytique des données, consultez la liste des [services d’analytique entièrement gérés dans Azure][product-category].

## <a name="relevant-use-cases"></a>Cas d’usage appropriés

Les autres cas d’usage appropriés sont les suivants :

* scénarios de construction, d’extraction ou de fabrication d’équipements ;
* collecte de données d’appareil à grande échelle à des fins de stockage et d’analyse ;
* ingestion et analyse de jeux de données volumineux.

## <a name="architecture"></a>Architecture

![Architecture pour l’IoT et l’analytique des données dans le secteur de la construction][architecture]

Les données circulent dans la solution comme suit :

1. Les équipements de construction collectent les données de capteur et envoient les données des résultats de construction à intervalles réguliers aux services web à charge équilibrée hébergés sur un cluster de machines virtuelles Azure.
2. Les services web personnalisés ingèrent les données des résultats de construction et les stockent dans un cluster Apache Cassandra également exécuté sur des machines virtuelles Azure.
3. Un autre jeu de données est regroupé par les capteurs IoT de divers équipements de construction et envoyé au service IoT Hub.
4. Les données brutes collectées sont envoyées directement par IoT Hub au service Stockage Blob Azure et sont alors immédiatement consultables et analysables.
5. Les données collectées par le biais du service IoT Hub sont traitées en quasi temps réel par un travail Azure Stream Analytics et sont stockées dans une base de données Azure SQL Database.
6. Les analystes et les utilisateurs finals peuvent accéder à l’application web de construction intelligente basée sur le cloud (Smart Construction Cloud) afin de visualiser et d’analyser les données de capteur et l’imagerie. 
7. Les utilisateurs de l’application web lancent des programmes de traitement par lots à la demande. Ces programmes s’exécutent dans Apache Spark sur HDInsight et analysent les nouvelles données stockées dans le cluster Cassandra. 

### <a name="components"></a>Composants

* [IoT Hub](/azure/iot-hub/about-iot-hub) joue le rôle de hub de messages central sécurisant les communications bidirectionnelles entre la plateforme cloud et les équipements et autres éléments du site de construction grâce à la gestion des identités par appareil. IoT Hub peut collecter rapidement les données de chaque appareil en vue de leur ingestion dans le pipeline d’analytique des données. 
* [Azure Stream Analytics](/azure/stream-analytics/stream-analytics-introduction) est un moteur de traitement des événements qui peut analyser d’importants volumes de flux de données en provenance d’appareils et d’autres sources de données. Il prend également en charge l’extraction d’informations à partir de flux de données pour identifier des modèles et des relations. Dans ce scénario, Stream Analytics ingère et analyse les données issues d’appareils IoT et stocke les résultats dans Azure SQL Database. 
* [Azure SQL Database](/azure/sql-database/sql-database-technical-overview) contient les résultats d’analyse des données émanant des appareils IoT et des compteurs. Ces résultats sont alors consultables par les analystes et les utilisateurs par le biais d’une application web basée sur Azure. 
* [Stockage Blob](/azure/storage/blobs/storage-blobs-introduction) stocke les données d’image regroupées à partir des appareils IoT Hub. Les données d’image sont visualisables par l’intermédiaire de l’application web.
* [Traffic Manager](/azure/traffic-manager/traffic-manager-overview) contrôle la distribution du trafic utilisateur pour les points de terminaison de service dans les différentes régions Azure.
* [Load Balancer](/azure/load-balancer/load-balancer-overview) répartit les envois de données des appareils d’équipement de construction entre les différents services web basés sur des machines virtuelles afin de garantir une haute disponibilité.
* [Machines virtuelles Azure](/azure/virtual-machines) héberge les services web qui reçoivent et ingèrent les données des résultats de construction dans la base de données Apache Cassandra.
* [Apache Cassandra](https://cassandra.apache.org) est une base de données NoSQL distribuée utilisée pour le stockage des données de construction à des fins de traitement ultérieur par Apache Spark.
* [Web Apps](/azure/app-service/app-service-web-overview) héberge l’application web des utilisateurs finals, qui permet d’interroger et de visualiser les données et images sources. Les utilisateurs peuvent également lancer des programmes de traitement par lots dans Apache Spark par le biais de l’application.
* [Apache Spark sur HDInsight](/azure/hdinsight/spark/apache-spark-overview) prend en charge le traitement en mémoire pour améliorer les performances des applications d’analyse du Big Data. Dans ce scénario, Spark est utilisé pour exécuter des algorithmes complexes sur les données stockées dans Apache Cassandra.


### <a name="alternatives"></a>Autres solutions

* [Cosmos DB](/azure/cosmos-db/introduction) est une technologie de base de données NoSQL alternative. Cosmos DB fournit une [prise en charge multimaître à l’échelle mondiale](/azure/cosmos-db/multi-region-writers) avec [plusieurs niveaux de cohérence bien définis](/azure/cosmos-db/consistency-levels) pour répondre à diverses exigences clients. Ce service prend également en charge [l’API Cassandra](/azure/cosmos-db/cassandra-introduction). 
* [Azure Databricks](/azure/azure-databricks/what-is-azure-databricks) est une plateforme d’analyse basée sur Apache Spark et optimisée pour Azure. Elle est intégrée à Azure afin d’offrir une configuration en un clic, des workflows simplifiés et un espace de travail collaboratif interactif.
* [Data Lake Storage](/azure/storage/data-lake-storage) constitue une alternative au service Stockage Blob. Pour ce scénario, Data Lake Storage n’était pas disponible dans la région ciblée.
* [Web Apps](/azure/app-service) est également utilisable pour héberger les services web destinés à ingérer les données des résultats de construction.
* De nombreuses options technologiques sont disponibles pour l’ingestion des messages en temps réel, le stockage de données, le traitement de flux, le stockage des données d’analyse et l’analyse et la création de rapports. Pour une vue d’ensemble de ces options, de leurs fonctionnalités et des principaux critères de sélection, consultez l’article [Architectures de Big Data : Traitement en temps réel](/azure/architecture/data-guide/technology-choices/real-time-ingestion) dans le [Guide d’architecture des données Azure](/azure/architecture/data-guide).

## <a name="considerations"></a>Considérations

Le large éventail de régions Azure disponibles joue un rôle important dans ce scénario. L’existence de plusieurs régions Azure dans un même pays peut offrir une capacité de reprise d’activité tout en assurant la conformité aux obligations contractuelles et aux exigences en matière de respect des lois. La communication à haut débit entre les régions Azure constitue également un facteur important dans ce scénario.

La prise en charge par Azure des technologies open source a permis au client de tirer profit des qualifications de sa main-d’œuvre existante. Le client est également en mesure d’accélérer l’adoption des nouvelles technologies tout en allégeant les coûts et les charges de travail d’exploitation par rapport à une solution locale. 

## <a name="pricing"></a>Tarifs

Les considérations ci-après ont une incidence notable sur les coûts de cette solution.

* Les coûts de machine virtuelle Azure augmentent de manière linéaire à mesure que des instances supplémentaires sont approvisionnées. Les machines virtuelles qui sont libérées entraînent uniquement des coûts de stockage, et non des coûts de calcul. Ces machines libérées peuvent être réaffectées par la suite en cas d’augmentation de la demande.
* Les coûts [IoT Hub](https://azure.microsoft.com/pricing/details/iot-hub) dépendent du nombre d’unités IoT approvisionnées, ainsi que du niveau de service choisi, qui détermine le nombre de messages autorisés par jour et par unité. 
* [Stream Analytics](https://azure.microsoft.com/pricing/details/stream-analytics) est facturé selon le nombre d’unités de streaming requis pour traiter les données au sein du service.

## <a name="related-resources"></a>Ressources associées

Pour découvrir une implémentation d’une architecture similaire, lisez le [témoignage client Komatsu][customer-story].

Des conseils en matière d’architectures de Big Data sont fournis dans le [Guide d’architecture des données Azure](/azure/architecture/data-guide).

<!-- links -->
[product-category]: https://azure.microsoft.com/product-categories/analytics/
[customer-site]: https://home.komatsu/en/
[customer-story]: https://customers.microsoft.com/story/komatsu-manufacturing-azure-iot-hub-japan
[architecture]: ./media/architecture-big-data-with-iot.png
