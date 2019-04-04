---
title: Supervision d’Azure Databricks avec Azure Monitor
description: Bibliothèque scala pour activer la supervision d’Azure Databricks dans Azure Log Analytics
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: 4544d3abc3264ec459a80ac1a61a912e6d30d6b2
ms.sourcegitcommit: 1a3cc91530d56731029ea091db1f15d41ac056af
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/03/2019
ms.locfileid: "58887690"
---
# <a name="monitoring-azure-databricks"></a>Supervision Azure Databricks

[Azure Databricks](/azure/azure-databricks/) est un service d’analytique rapide et puissant basé sur [Apache Spark](https://spark.apache.org/), qui permet de développer et de déployer de manière simple et rapide l’analytique de big data et des solutions d’intelligence artificielle (IA). Nombre d’utilisateurs tirent parti de la simplicité des notebooks dans leurs solutions Azure Databricks. Pour les utilisateurs qui ont besoin d’options de calcul plus robustes, Azure Databricks prend en charge l’exécution distribuée de code d’application personnalisé.

La supervision étant un élément primordial de toute solution de niveau production, Azure Databricks offre des fonctionnalités solides pour superviser les métriques d’application personnalisées, le streaming des événements de requête et les messages de journal d’application. Azure Databricks peut envoyer ces données de supervision à différents services de journalisation.

Les articles suivants expliquent comment envoyer des données de supervision depuis Azure Databricks vers [Azure Monitor](/azure/azure-monitor/overview), qui est la plateforme de données de supervision pour Azure. Suivez ces articles dans l’ordre.

1. [Configurer Azure Databricks pour envoyer des métriques à Azure Monitor](./configure-cluster.md)
1. [Envoyer les journaux des applications Azure Databricks à Azure Monitor](./application-logs.md)
1. [Utiliser des tableaux de bord pour visualiser les métriques Azure Databricks](./dashboards.md)

La bibliothèque de code qui accompagne ces articles étend la fonctionnalité de supervision principale d’Azure Databricks pour envoyer les métriques, les événements et les informations de journalisation Spark à Azure Monitor.

Ces articles et la bibliothèque de code qui les accompagne s’adressent aux développeurs de solutions Apache Spark et Azure Databricks. Le code doit être intégré dans des fichiers JAR (Java Archive), puis déployé sur un cluster Azure Databricks. Le code est un mélange de [Scala](https://www.scala-lang.org/) et de Java, avec un ensemble de fichiers POM (project object model) [Maven](https://maven.apache.org) correspondants pour générer les fichiers JAR de sortie. Comme prérequis, il est recommandé de connaître Java, Scala et Maven.

## <a name="next-steps"></a>Étapes suivantes

Commencez par créer la bibliothèque de code, puis déployez-la sur votre cluster Azure Databricks.

> [!div class="nextstepaction"]
> [Configurer Azure Databricks pour envoyer des métriques à Azure Monitor](./configure-cluster.md)