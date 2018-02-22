---
title: "Sélection d’une technologie d’orchestration de pipeline de données"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 17aeb871bc815793295ed610795e5e83de72c637
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-data-pipeline-orchestration-technology-in-azure"></a>Sélection d’une technologie d’orchestration de pipeline de données dans Azure

La plupart des solutions de Big Data se composent d’opérations de traitement des données répétées, encapsulées dans des workflows. Un orchestrateur de pipeline est un outil qui permet d’automatiser ces workflows. Un orchestrateur peut planifier des travaux, exécuter des workflows et coordonner les dépendances entre des tâches.

## <a name="what-are-your-options-for-data-pipeline-orchestration"></a>Quelles sont vos options d’orchestration de pipeline de données ?

Dans Azure, les outils et services suivants répondent aux exigences principales d’orchestration de pipeline, de flux de contrôle et de déplacement des données :

- [Azure Data Factory](/azure/data-factory/)
- [Oozie sur HDInsight](/azure/hdinsight/hdinsight-use-oozie-linux-mac)
- [SQL Server Integration Services (SSIS)](/sql/integration-services/sql-server-integration-services)

Ces services et outils peuvent être utilisés indépendamment l’un de l’autre ou conjointement pour créer une solution hybride. Par exemple, Integration Runtime (IR) dans Azure Data Factory V2 peut exécuter en mode natif des packages SSIS dans un environnement de calcul Azure géré. S’il existe certains recoupements des fonctionnalités entre ces services, il existe aussi quelques différences importantes.

## <a name="key-selection-criteria"></a>Critères de sélection principaux

Pour restreindre les choix, commencez par répondre aux questions suivantes :

- Avez-vous besoin des fonctionnalités de Big Data pour déplacer et transformer vos données ? Généralement, cela signifie des gigaoctets à des téraoctets de données. Dans ce cas, limitez vos options à celles qui sont le mieux adaptées au Big Data.

- Avez-vous besoin d’un service géré qui puisse fonctionner à l’échelle ? Dans ce cas, sélectionnez un des services cloud non limité par votre puissance de traitement local.

- Certaines de vos données sources sont-elles locales ? Dans l’affirmative, recherchez les options qui peuvent fonctionner avec les sources de données ou les destinations locales et sur cloud.

- Vos données sources sont-elles stockées dans le stockage Blob sur un système de fichiers HDFS ? Dans ce cas, choisissez une option qui prend en charge les requêtes Hive.

## <a name="capability-matrix"></a>Matrice des fonctionnalités

Les tableaux suivants résument les principales différences entre les fonctionnalités.

### <a name="general-capabilities"></a>Fonctionnalités générales

| | Azure Data Factory | SQL Server Integration Services (SSIS) | Oozie sur HDInsight
| --- | --- | --- | --- |
| Adresses IP gérées | OUI | Non  | OUI |
| Sur le cloud | OUI | Non (Local) | OUI |
| Configuration requise | Abonnement Azure | SQL Server  | Abonnement Azure, cluster HDInsight |
| Outils de gestion | Portail Azure, PowerShell, CLI, .NET SDK | SSMS, PowerShell | Interpréteur de commandes Bash, API REST Oozie, IU Web Oozie |
| Tarifs | Paiement à l’utilisation | Licences / paiement des fonctionnalités | Aucun frais supplémentaire sur l’exécution du cluster HDInsight |

### <a name="pipeline-capabilities"></a>Fonctionnalités du pipeline

| | Azure Data Factory | SQL Server Integration Services (SSIS) | Oozie sur HDInsight
| --- | --- | --- | --- |
| Copier des données | OUI | OUI | OUI |
| Transformations personnalisées | OUI | OUI | Oui (travaux MapReduce, Pig et Hive) |
| Notation d’Azure Machine Learning | OUI | Oui (avec des scripts) | Non  |
| HDInsight à la demande | OUI | Non  | Non  |
| Azure Batch | OUI | Non  | Non  |
| Pig, Hive, MapReduce | OUI | Non  | OUI |
| Spark | OUI | Non  | Non  |
| Exécuter le Package SSIS | OUI | OUI | Non  |
| Flux de contrôle | OUI | OUI | OUI |
| Accès aux données locales | OUI | OUI | Non  |

### <a name="scalability-capabilities"></a>Fonctionnalités d’extensibilité

| | Azure Data Factory | SQL Server Integration Services (SSIS) | Oozie sur HDInsight
| --- | --- | --- | --- |
| Monter en puissance | OUI | Non  | Non  |
| Montée en charge | OUI | Non  | Oui (via l’ajout de nœuds de travail en cluster) |
| Optimisé pour le Big Data | OUI | Non  | OUI |

