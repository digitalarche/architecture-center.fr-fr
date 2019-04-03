---
title: Guide du développeur R sur Azure - Programmation R | Microsoft Docs
description: Cet article fournit une vue d’ensemble des différentes façons dont les scientifiques des données peuvent exploiter leurs compétences existantes avec le langage de programmation R dans Azure. Azure propose de nombreux services qui R permet aux développeurs d’étendre leurs charges de travail de science des données dans le cloud.
services: machine-learning
author: AnalyticJeremy
ms.service: machine-learning
ms.workload: data-services
ms.devlang: R
ms.topic: article
ms.date: 03/19/2019
ms.author: jepeach
ms.openlocfilehash: a14c8ce76f78baa7274f22b939eb28cb025ef87e
ms.sourcegitcommit: 1a3cc91530d56731029ea091db1f15d41ac056af
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/03/2019
ms.locfileid: "58888016"
---
# <a name="r-developers-guide-to-azure"></a>Guide du développeur R sur Azure

<img src="../images/logo_r.svg" alt="R logo" align="right" width="200" />

De nombreux scientifiques des données qui ont à traiter des volumes de données toujours plus importants recherchent des moyens d’exploiter la puissance du cloud computing pour leurs analyses.  Cet article fournit une vue d’ensemble des différentes façons dont les scientifiques des données peuvent exploiter leurs compétences existantes avec le [langage de programmation R](https://www.r-project.org) dans Azure.

Microsoft a entièrement adopté le langage de programmation R comme outil privilégié pour les scientifiques de données.  En offrant de nombreuses options différentes aux développeurs R pour exécuter leur code dans Azure, Microsoft permet aux scientifiques des données d’étendre leurs charges de travail de science des données dans le cloud quand ils doivent mener des projets à grande échelle.

Commençons par examiner les différentes options et les scénarios les plus parlants pour chacune d’elles.

## <a name="azure-services-with-r-language-support"></a>Services Azure avec prise en charge du langage R

Cet article couvre les services Azure suivants qui prennent en charge le langage R :

|de diffusion en continu                                                          |Description                                                                       |
|-----------------------------------------------------------------|----------------------------------------------------------------------------------|
|[Machine virtuelle de science des données](#data-science-virtual-machine)    |Une machine virtuelle personnalisée à utiliser comme une station de travail de science des données ou comme une cible de calcul personnalisée|
|[ML Services sur HDInsight](#ml-services-on-hdinsight)            |Système basé sur un cluster pour l’exécution d’analyses R sur des grands jeux de données sur de nombreux nœuds   |
|[Azure Databricks](#azure-databricks)                            |Environnement Spark collaboratif qui prend en charge R et autres langages               |
|[Azure Machine Learning Studio](#azure-machine-learning-studio)  |Exécuter des scripts R personnalisés dans les expériences de machine learning d’Azure                      |
|[Azure Batch](#azure-batch)                                      |Offre différentes options pour exécuter de façon économique du code R sur de nombreux nœuds dans un cluster|
|[Azure Notebooks](#azure-notebooks)                              |une version cloud gratuite des notebooks Jupyter                  |
|[Azure SQL Database](#azure-sql-database)                        |Exécuter des scripts R à l’intérieur du moteur de base de données SQL Server                            |

## <a name="data-science-virtual-machine"></a>Machine virtuelle de science des données

La machine virtuelle [DSVM (Data Science Virtual Machine)](https://docs.microsoft.com/azure/machine-learning/data-science-virtual-machine/overview) est une image de machine virtuelle personnalisée sur la plateforme cloud Microsoft Azure spécialement conçue pour la science des données. Elle a de nombreux outils de science des données courants, notamment :

* [Microsoft R Open](https://mran.microsoft.com/open/)
* [Microsoft Machine Learning Server](https://docs.microsoft.com/machine-learning-server/what-is-machine-learning-server)
* [RStudio Desktop](https://www.rstudio.com/products/rstudio/#Desktop)
* [RStudio Server](https://www.rstudio.com/products/rstudio/#Server)

La machine virtuelle DSVM peut être provisionnée avec Windows ou Linux comme système d’exploitation.  Vous pouvez utiliser la machine virtuelle DSVM de deux façons différentes : comme station de travail interactive ou comme plateforme de calcul pour un cluster personnalisé.

### <a name="as-a-workstation"></a>Comme station de travail

Si vous voulez vous familiariser rapidement et facilement avec R dans le cloud, il s’agit de la meilleure solution.  L’environnement sera familier à quiconque a travaillé avec R sur une station de travail locale.  Cependant, au lieu d’utiliser des ressources locales, l’environnement R s’exécute sur une machine virtuelle dans le cloud.  Si vos données sont déjà stockées dans Azure, ceci a l’avantage de permettre à vos scripts R de s’exécuter « plus près des données ». Au lieu de transférer les données sur Internet, les données sont accessibles sur le réseau interne d’Azure, qui offre des temps d’accès beaucoup plus rapides.

La machine virtuelle DSVM peut être particulièrement pratique pour les petites équipes de développeurs R.  Au lieu d’investir dans des stations de travail puissantes pour chaque développeur et de nécessiter la synchronisation entre les membres de l’équipe des versions des différents packages logiciels qu’ils utilisent, chaque développeur peut lancer une instance de machine virtuelle DSVM chaque fois que c’est nécessaire.

### <a name="as-a-compute-platform"></a>Comme plateforme de calcul

Outre son utilisation comme station de travail, la machine virtuelle DSVM est également utilisée comme plateforme de calcul évolutive pour les projets R.  À l’aide de la [ `AzureDSVM` ](https://github.com/Azure/AzureDSVM) package R, vous pouvez contrôler par programme la création et la suppression d’instances de machine virtuelle DSVM.  Vous pouvez former les instances d’un cluster et déployer une analyse distribuée à effectuer dans le cloud.  L’ensemble de ce processus peut être contrôlé par du code R s’exécutant sur votre station de travail locale.

Pour en savoir plus sur la machine virtuelle DSVM, consultez [Introduction à Azure Data Science Virtual Machine pour Linux et Windows](https://docs.microsoft.com/azure/machine-learning/data-science-virtual-machine/overview).

## <a name="ml-services-on-hdinsight"></a>ML Services sur HDInsight

[Microsoft ML Services](https://docs.microsoft.com/azure/hdinsight/r-server/r-server-overview) offre aux experts en science des données, aux statisticiens et aux programmeurs R un accès à la demande à des méthodes d’analytique évolutives et distribuées sur HDInsight.  Cette solution offre les fonctionnalités d’analytique R les plus récentes sur des jeux de données de virtuellement n’importe quelle taille, chargés dans Stockage Blob Azure ou dans un stockage Azure Data Lake.

Il s’agit d’une solution d’entreprise qui vous permet de mettre à l’échelle votre code R sur un cluster.  En tirant parti des fonctions de Microsoft [`RevoScaleR`](https://docs.microsoft.com/machine-learning-server/r-reference/revoscaler/revoscaler)
package, vos scripts R sur HDInsight peuvent exécuter des fonctions de traitement des données en parallèle sur plusieurs nœuds dans un cluster.  Ceci permet à R de traiter des données sur une plus grande échelle que ce qui est possible avec R en monothread s’exécutant sur une station de travail.

Cette possibilité de mise à l’échelle fait de ML Services sur HDInsight une option intéressante pour les développeurs R qui doivent traiter de très grands jeux de données.  Elle offre une plateforme flexible et évolutive pour l’exécution de vos scripts R dans le cloud.

Pour une procédure pas à pas sur la création d’un cluster de Services ML, consultez [prise en main des Services de machine Learning sur Azure HDInsight](https://docs.microsoft.com/azure/hdinsight/r-server/r-server-get-started).

## <a name="azure-databricks"></a>Azure Databricks

[Azure Databricks](https://azure.microsoft.com/services/databricks/) est une plateforme d’analytique basée sur Apache Spark et optimisée pour la plateforme de services cloud Microsoft Azure.  Conçu avec les créateurs d’Apache Spark, Databricks est intégré dans Azure pour offrir une configuration en un clic, des workflows simplifiés et un espace de travail interactif permettant aux scientifiques des données, aux ingénieurs des données et aux analystes métier de collaborer.

La collaboration dans Databricks est permise par le système de notebooks de la plateforme.  Les utilisateurs peuvent créer, partager et modifier les notebooks avec d’autres utilisateurs des systèmes.  Ces notebooks permettent aux utilisateurs d’écrire du code qui s’exécute sur des clusters Spark gérés dans l’environnement Databricks.  Ces notebooks prennent entièrement en charge R et donnent aux utilisateurs l’accès à Spark via les deux packages `SparkR` et `sparklyr`.

Comme Databricks est basé sur la technologie Spark et est centré sur la collaboration, la plateforme est souvent utilisée par les équipes de scientifiques des données qui travaillent ensemble sur des analyses complexes de grands jeux de données.  Les notebooks dans Databricks prenant en charge d’autres langages en plus de R, il est particulièrement pratique pour les équipes où les analystes utilisent différents langages pour leur travail principal.

L’article [Azure databricks ?](https://docs.microsoft.com/azure/azure-databricks/what-is-azure-databricks) peut fournir plus d’informations sur la plateforme et à commencer.

## <a name="azure-machine-learning-studio"></a>Azure Machine Learning Studio

[Azure Machine Learning Studio](https://azure.microsoft.com/services/machine-learning-studio/) est un outil collaboratif fonctionnant par glisser-déplacer, qui vous permet de générer, tester et déployer des solutions d’analyse prédictive dans le cloud.  Il permet aux scientifiques des données qui débutent de créer et de déployer des modèles de machine learning sans devoir écrire beaucoup de code.

Azure Machine Learning Studio prend en charge R et Python.  Vous pouvez utiliser R avec Azure Machine Learning Studio de deux manières.

### <a name="custom-r-scripts-in-your-experiments"></a>Scripts R personnalisés dans vos expériences

Vous pouvez d’abord étendre les fonctionnalités de manipulation de données et de machine learning de ML Studio en écrivant des scripts R personnalisés.
Bien que ML Studio comprenne un large éventail de modules pour la préparation et l’analyse des données, il ne peut pas être au niveau des fonctionnalités d’un langage mature comme R. Par conséquent, le service a été conçu pour vous permettre d’introduire vos propres scripts R personnalisés dans les cas où les modules fournis ne répondent pas à vos besoins.

Pour tirer parti de cette fonctionnalité, faites glisser un module « Exécuter un script R » dans votre expérience.  Ensuite, utilisez l’éditeur de code dans le volet « Propriétés » pour écrire un nouveau script R ou pour coller un script existant.  Dans le script, vous pouvez référencer des packages R externes.  Vous pouvez utiliser le script pour manipuler des données ou pour former des modèles ML complexes qui ne font pas partie de la bibliothèque de modèles Azure Machine Learning Studio standard.

Pour obtenir une présentation approfondie sur l’utilisation de R dans les expériences de ML Studio, consultez [mise en route avec le langage dans Azure Machine Learning Studio de programmation R](https://docs.microsoft.com/azure/machine-learning/studio/r-quickstart).

### <a name="create-manage-and-deploy-experiments-from-your-local-r-environment"></a>Créer, gérer et déployer des expériences à partir de votre environnement R local

L’autre que vous pouvez utiliser R avec Azure Machine Learning Studio consiste à utiliser le [ `AzureML` ](https://cran.r-project.org/web/packages/AzureML/vignettes/getting_started.html) package pour surveiller et contrôler le processus d’expérimentation avec l’environnement de programmation R.  Ce package, ce qui est géré par Microsoft, vous permet de charger et télécharger des jeux de données vers et depuis Azure Machine Learning Studio, interroger les expériences, de publier R fonctionne en tant que services web et d’exécuter R données par le biais des services web et récupèrent le sortie.

Ce package rend beaucoup plus facile à utiliser Azure Machine Learning Studio en tant que plate-forme de déploiement évolutive pour votre code R.  Au lieu de cliquer et de faire glisser dans l’interface utilisateur, vous pouvez automatiser le processus de déploiement complet avec des outils que vous connaissez déjà.

## <a name="azure-batch"></a>Azure Batch

Pour les travaux R à grande échelle, vous pouvez utiliser [Azure Batch](https://azure.microsoft.com/services/batch/).  Ce service fournit la planification des travaux à l’échelle du cloud et la gestion de la capacité de calcul, qui vous permettent de mettre à l’échelle votre charge de travail R sur des dizaines, des centaines ou des milliers de machines virtuelles.  Comme il s’agit d’une plateforme de traitement généralisée, quelques options sont possibles pour l’exécution des travaux R sur Azure Batch.

Une option consiste à utiliser de Microsoft [ `doAzureParallel` ](https://github.com/Azure/doAzureParallel) package.  Ce package R est un serveur backend parallèle pour le package `foreach`.  Il permet à chaque itération de la boucle `foreach` de s’exécuter en parallèle sur un nœud au sein du cluster Azure Batch.  Pour une introduction au package, consultez le blog [doAzureParallel : Tirer parti de calcul flexibles de d’Azure directement à partir de votre session R](https://azure.microsoft.com/blog/doazureparallel/).

Une autre option pour exécuter un script R dans Azure Batch consiste à faire un bundle de votre code avec « RScript.exe » sous la forme d’une application Batch dans le portail Azure.  Pour une procédure détaillée, consultez [charges de travail R sur Azure Batch](https://azure.microsoft.com/blog/r-workloads-on-azure-batch/).

Une troisième option consiste à utiliser [Azure Distributed Data Engineering Toolkit](https://github.com/Azure/aztk) (AZTK), qui vous permet de provisionner des clusters Spark à la demande avec des conteneurs Docker dans Azure Batch.  Ceci offre un moyen économique d’exécuter des travaux Spark dans Azure.  En utilisant [SparklyR avec AZTK](https://github.com/Azure/aztk/wiki/SparklyR-on-Azure-with-AZTK), vos scripts R peuvent bénéficier d’une augmentation du nombre d’instances dans le cloud facilement et à moindre coût.

## <a name="azure-notebooks"></a>Azure Notebooks

[Azure Notebooks](https://notebooks.azure.com) est une méthode économique et facile pour les développeurs R qui préfèrent travailler avec des notebooks pour placer leur code dans Azure.  C’est un service gratuit pour toute personne développant et exécutant du code dans son navigateur avec [Jupyter](https://jupyter.org/), qui est un projet open source permettant de combiner du texte Markdown, du code exécutable et des graphiques sur un même canevas.

Le niveau de service gratuit d’Azure Notebooks Azure est une option viable pour les projets de faible envergure, car il limite le processus de chaque ordinateur portable à 4 Go de mémoire et la taille des jeux de données à 1 Go. Si vous avez besoin de puissance de calcul et de données au-delà de ces limites, cependant, vous pouvez exécuter des notebooks dans une instance Data Science Virtual Machine. Pour plus d’informations, consultez [Gérer et configurer des projets Azure Notebooks - Niveau Calcul](/azure/notebooks/configure-manage-azure-notebooks-projects#compute-tier).

## <a name="azure-sql-database"></a>Azure SQL Database

[Azure SQL Database](https://azure.microsoft.com/services/sql-database/) est le service de base de données cloud relationnelle intelligent et entièrement managé de Microsoft.  Il vous permet d’utiliser toute la puissance de SQL Server sans devoir configurer l’infrastructure.  Cela inclut les [Machine Learning Services dans SQL Server](https://docs.microsoft.com/sql/advanced-analytics/what-is-sql-server-machine-learning?view=sql-server-2017), qui est une des plus récents ajouts à SQL.

Cette fonctionnalité offre un moteur incorporé d’analytique prédictive et de science des données, qui peut exécuter du code R dans une base de données SQL Server sous forme de procédures stockées, de scripts T-SQL contenant des instructions R ou de code R contenant du code T-SQL.  Au lieu d’extraire des données à partir de la base de données et de les charger dans l’environnement R, vous chargez votre code R directement dans la base de données et vous le laissez s’exécuter directement aux côtés des données.

Si Machine Learning Services fait partie de SQL Server depuis 2016, il est relativement nouveau dans Azure SQL Database.  Il est actuellement en [préversion limitée](https://docs.microsoft.com/sql/advanced-analytics/what-s-new-in-sql-server-machine-learning-services?view=sql-server-2017#azure-sql-database-roadmap), mais il continue d’évoluer.


### <a name="next-steps"></a>Étapes suivantes

* [Exécution de votre code R sur Azure avec mrsdeploy](https://blog.revolutionanalytics.com/2017/03/running-your-r-code-azure.html)
* [Machine Learning Server dans le Cloud](https://docs.microsoft.com/machine-learning-server/install/machine-learning-server-in-the-cloud)
* [Ressources supplémentaires pour Machine Learning Server et Microsoft R](https://docs.microsoft.com/machine-learning-server/resources-more)
* [R sur Azure](https://github.com/yueguoguo/r-on-azure) : une vue d’ensemble des packages, des outils et d’études de cas pour l’utilisation de R avec Azure

---

<sub>Le logo de R est &copy; 2016 la R Foundation et est utilisé dans le cadre de la [les licence Creative Commons Attribution-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-sa/4.0/).</sub>
