---
title: Sélectionner une analytique de données et une technologie de création de rapports
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 05e33a3da0933036a604d2bc4cc5a20ae70fe772
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428311"
---
# <a name="choosing-a-data-analytics-technology-in-azure"></a>Sélectionner une technologie d’analytique de données dans Azure

La plupart des solutions Big Data ont pour but de fournir des informations sur les données via l’analyse et les rapports. Cela peut inclure des visualisations et des rapports préconfigurés, ou une exploration interactive des données. 

## <a name="what-are-your-options-when-choosing-a-data-analytics-technology"></a>Quelles sont vos options quant au choix d’une technologie d’analytique de données ?

Il existe plusieurs options pour l’analyse, les visualisations et la création de rapports dans Azure, selon vos besoins :

- [Power BI](/power-bi/)
- [Blocs-notes Jupyter](https://jupyter.readthedocs.io/en/latest/index.html)
- [Blocs-notes Zeppelin](https://zeppelin.apache.org/)
- [Blocs-notes Microsoft Azure](https://notebooks.azure.com/)

### <a name="power-bi"></a>Power BI

[Power BI](/power-bi/) est une suite d’outils d’analytique métier. Elle peut se connecter à des centaines de sources de données et peut être utilisée pour l’analyse ad hoc. Consultez [cette liste](/power-bi/desktop-data-sources) des sources de données actuellement disponibles. Utilisez [Power BI Embedded](https://azure.microsoft.com/services/power-bi-embedded/) intégration Power BI dans vos propres applications sans nécessiter de licences supplémentaires.

Les organisations peuvent utiliser Power BI pour générer des rapports et les publier. Tout le monde peut créer des tableaux de bord personnalisés, avec gouvernance et [sécurité intégrée](/power-bi/service-admin-power-bi-security). Power BI utilise [Azure Active Directory](/azure/active-directory/) (Azure AD) pour authentifier les utilisateurs qui se connectent au service Power BI, et utilise les informations de connexion Power BI chaque fois qu’un utilisateur tente d’accéder aux ressources nécessitant une authentification.

### <a name="jupyter-notebooks"></a>Blocs-notes Jupyter 

Les [blocs-notes Jupyter](https://jupyter.readthedocs.io/en/latest/index.html) fournissent un interpréteur de commandes basé sur navigateur permettant aux scientifiques de données de créer des fichiers *blocs-notes* contenant du code Python, Scala ou R ainsi que du texte Markdown. C’est un moyen efficace de collaborer en partageant et en documentant le code et les résultats dans un document unique.

La plupart des types de clusters HDInsight, notamment Spark ou Hadoop, sont [préconfigurés avec des blocs-notes Jupyter](/azure/hdinsight/spark/apache-spark-jupyter-notebook-kernels) pour interagir avec les données et envoyer les travaux pour le traitement. Selon le type du cluster HDInsight que vous utilisez, un ou plusieurs noyaux seront fournis pour l’interprétation et l’exécution de votre code. Par exemple, les clusters Spark sur HDInsight fournissent des noyaux Spark liés que vous pouvez sélectionner pour exécuter du code Python ou Scala à l’aide du moteur Spark.

Les blocs-notes Jupyter fournissent un environnement idéal pour l’analyse, la visualisation et le traitement de vos données avant de générer des visualisations plus avancées avec un outil BI ou de création de rapports comme Power BI.

### <a name="zeppelin-notebooks"></a>Blocs-notes Zeppelin

Les [blocs-notes Zeppelin](https://zeppelin.apache.org/) offrent également un interpréteur de commandes basé sur navigateur, similaire à Jupyter en termes de fonctionnalité. Certains clusters HDInsight sont [préconfigurés avec des blocs-notes Zeppelin](/azure/hdinsight/spark/apache-spark-zeppelin-notebook). Toutefois, si vous utilisez un cluster [HDInsight Interactive Query](/azure/hdinsight/interactive-query/apache-interactive-query-get-started) (Hive LLAP), [Zeppelin](/azure/hdinsight/hdinsight-connect-hive-zeppelin) est actuellement le seul choix possible de blocs-notes permettant d’exécuter des requêtes Hive interactives. En outre, si vous utilisez un [cluster HDInsight joint à un domaine](/azure/hdinsight/domain-joined/apache-domain-joined-introduction), les blocs-notes Zeppelin constituent le seul type vous permettant d’attribuer différentes connexions utilisateur afin de contrôler l’accès aux ordinateurs portables et aux tables Hive sous-jacentes.

### <a name="microsoft-azure-notebooks"></a>Blocs-notes Microsoft Azure

Les [blocs-notes Azure](https://notebooks.azure.com/) représente un service en ligne basé sur des blocs-notes Jupyter permettant aux scientifiques de données de créer, d’exécuter et de partager des blocs-notes Jupyter dans des bibliothèques cloud. Les blocs-notes Azure fournissent des environnements d’exécution pour Python 2, Python 3, F# et R, et incluent plusieurs bibliothèques de graphiques pour visualiser vos données, notamment ggplot, matplotlib, bokeh et seaborn.

Contrairement aux blocs-notes Jupyter en cours d’exécution sur un cluster HDInsight, et connectés au compte de stockage par défaut du cluster, les blocs-notes Azure ne fournissent aucune donnée. Vous devez [charger des données](https://notebooks.azure.com/Microsoft/libraries/samples/html/Getting%20to%20your%20Data%20in%20Azure%20Notebooks.ipynb) de plusieurs façons, par exemple en les téléchargeant à partir d’une source en ligne, en interagissant avec des objets blob Azure ou le stockage de tables, en vous connectant à une base de données SQL, ou en chargeant des données à l’aide de l’Assistant Copie pour Azure Data Factory.

Principaux avantages :

* Service gratuit&mdash;pas d’abonnement Azure requis.
* Inutile d’installer Jupyter et les distributions R ou Python connexes localement&mdash;utilisez simplement un navigateur.
* Gérez vos propres bibliothèques en ligne et accédez-y sur n’importe quel appareil.
* Partagez vos blocs-notes avec des collaborateurs.

Considérations :

* Vous ne pourrez pas accéder à vos blocs-notes en mode hors connexion.
* Les capacités de traitement limitées du service de blocs-notes gratuit peuvent ne pas suffire pour l’apprentissage de modèles volumineux ou complexes.

## <a name="key-selection-criteria"></a>Critères de sélection principaux

Pour restreindre les choix, commencez par répondre à ces questions :

- Devez-vous vous connecter à plusieurs sources de données, en indiquant un emplacement centralisé afin de créer des rapports pour les données réparties dans l’ensemble de votre domaine ? Dans ce cas, choisissez une option vous permettant de vous connecter à des centaines de sources de données.

- Souhaitez-vous incorporer des visualisations dynamiques dans un site web ou une application externe ? Dans ce cas, choisissez une option proposant des fonctionnalités d’incorporation.

- Souhaitez-vous concevoir vos visualisations et vos rapports en mode hors connexion ? Si oui, choisissez une option proposant des fonctionnalités hors connexion.

- Avez-vous besoin d’une grande puissance de traitement pour l’apprentissage de modèles IA volumineux ou complexes, ou pour utiliser des jeux de données très volumineux ? Si oui, choisissez une option permettant de se connecter à un cluster de données volumineux.

## <a name="capability-matrix"></a>Matrice des fonctionnalités

Les tableaux suivants résument les principales différences entre les fonctionnalités. 

### <a name="general-capabilities"></a>Fonctionnalités générales

| | Power BI | Blocs-notes Jupyter | Blocs-notes Zeppelin | Blocs-notes Microsoft Azure |
| --- | --- | --- | --- | --- |
| Se connecter à un cluster Big Data pour un traitement avancé | Oui | OUI | Oui | Non  |
| Service géré | Oui | Oui <sup>1</sup> | Oui <sup>1</sup> | Oui |
| Se connecter à des centaines de sources de données | Oui | Non  | Non  | Non  |
| Fonctionnalités hors ligne | Oui <sup>2</sup> | Non  | Non  | Non  |
| Fonctionnalités d’incorporation | Oui | Non  | Non  | Non  |
| Actualisation automatique des données | Oui | Non  | Non  | Non  |
| Accès à de nombreux packages open source | Non  | Oui <sup>3</sup> | Oui <sup>3</sup> | Oui <sup>4</sup> |
| Options de nettoyage/transformer de données | [Power Query](https://powerbi.microsoft.com/blog/getting-started-with-power-query-part-i/), R | 40 langues, y compris Python, R, Julia et Scala | Plus de 20 interpréteurs, y compris Python, JDBC et R | Python, F#, R |
| Tarifs | Gratuit pour Power BI Desktop (création), consultez la section [Tarification](https://powerbi.microsoft.com/pricing/) pour les options d’hébergement | Gratuit | Gratuit | Gratuit |
| Collaboration multi-utilisateur | [Oui](/power-bi/service-how-to-collaborate-distribute-dashboards-reports) | Oui (via le partage ou avec un serveur multi-utilisateur comme [JupyterHub](https://github.com/jupyterhub/jupyterhub)) | Oui | Oui (via le partage) |

[1] Si utilisé dans un cluster HDInsight géré.

[2] En utilisant Power BI Desktop.

[2] Vous pouvez rechercher dans le [référentiel Maven](https://search.maven.org/) des packages proposés par la communauté.

[3] Les packages Python peuvent être installés à l’aide de pip ou conda. Les packages R peuvent être installés à partir de CRAN ou de GitHub. Les packages en F # peuvent être installés via nuget.org à l’aide du [Gestionnaire de dépendances Paket](https://fsprojects.github.io/Paket/).

