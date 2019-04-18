---
title: Utiliser des tableaux de bord pour visualiser les métriques Azure Databricks
description: Comment déployer un tableau de bord Grafana pour analyser les performances dans Azure Databricks
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: a84203a9188848e6363a80ac455332e8f6a73cda
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640309"
---
# <a name="use-dashboards-to-visualize-azure-databricks-metrics"></a>Utiliser des tableaux de bord pour visualiser les métriques Azure Databricks

Cet article explique comment configurer un tableau de bord Grafana pour surveiller les travaux Azure Databricks pour les problèmes de performances.

[Azure Databricks](/azure/azure-databricks/) est rapide, puissante et de collaboration [Apache Spark](https://spark.apache.org/)– en fonction du service d’analytique qui facilite la rapidement développer et déployer l’analytique de big data et les solutions d’intelligence artificielle (IA). La surveillance est un composant essentiel de l’utilisation de charges de travail Azure Databricks en production. La première étape consiste à collecter des mesures dans un espace de travail pour l’analyse. Dans Azure, la meilleure solution pour la gestion des données de journal est [Azure Monitor](/azure/azure-monitor/). Databricks Azure ne prend pas en charge envoi de données de journal à Azure monitor, mais un [bibliothèque pour cette fonctionnalité](https://github.com/mspnp/spark-monitoring) est disponible dans [Github](https://github.com).

Cette bibliothèque Active la journalisation des métriques de service Azure Databricks aussi structure Apache Spark streaming des métriques d’événement de requête. Une fois que vous avez correctement déployé cette bibliothèque dans un cluster Azure Databricks, vous pouvez plus déployer un ensemble de [Grafana](https://granfana.com) des tableaux de bord que vous pouvez déployer dans le cadre de votre environnement de production.

![Capture d’écran du tableau de bord](./_images/dashboard-screenshot.png)

## <a name="prerequisites"></a>Conditions préalables

Clone le [référentiel Github](https://github.com/mspnp/spark-monitoring) et [suivez les instructions de déploiement](./configure-cluster.md) pour générer et configurer la journalisation d’Azure Monitor pour la bibliothèque Azure Databricks envoyer les journaux à votre espace de travail Analytique des journaux Azure.

## <a name="deploy-the-azure-log-analytics-workspace"></a>Déployer l’espace de travail Analytique des journaux Azure

Pour déployer l’espace de travail Analytique des journaux Azure, procédez comme suit :

1. Accédez à la `/perftools/deployment/loganalytics` directory.
1. Déployer le **logAnalyticsDeploy.json** modèle Azure Resource Manager. Pour plus d’informations sur le déploiement de modèles Resource Manager, consultez [déployer des ressources avec des modèles Resource Manager et Azure CLI][rm-cli]. Le modèle possède les paramètres suivants :

    * **location** : La région où l’espace de travail Analytique de journal et les tableaux de bord est déployés.
    * **serviceTier**: Niveau tarifaire de l’espace de travail Rhe. Consultez [ici] [ sku] pour obtenir la liste des valeurs valides.
    * **dataRetention** (facultatif) : Le nombre de jours de données de journal est conservé dans l’espace de travail Analytique de journal. La valeur par défaut est de 30 jours. Si le niveau tarifaire est `Free`, la rétention des données doit être de 7 jours.
    * **WorkspaceName** (facultatif) : Nom de l’espace de travail. Si non spécifié, le modèle génère un nom.

    ```bash
    az group deployment create --resource-group <resource-group-name> --template-file logAnalyticsDeploy.json --parameters location='East US' serviceTier='Standalone'
    ```

Ce modèle crée l’espace de travail et crée également un ensemble de requêtes prédéfinies qui sont utilisés par le tableau de bord.

## <a name="deploy-grafana-in-a-virtual-machine"></a>Déployer Grafana dans une machine virtuelle

Grafana est un projet open source que vous pouvez déployer pour visualiser les métriques de série de temps stockées dans votre espace de travail Analytique des journaux Azure à l’aide du plug-in Grafana pour Azure Monitor. Grafana s’exécute sur une machine virtuelle (VM) et nécessite un compte de stockage, de réseau virtuel et d’autres ressources. Pour déployer une machine virtuelle avec le bitnami certifié Grafana image et les ressources associées, procédez comme suit :

1. Utilisez l’interface CLI pour accepter les termes d’image de place de marché Azure de Grafana.

    ```bash
    az vm image accept-terms --publisher bitnami --offer grafana --plan default
    ```

1. Accédez à la `/spark-monitoring/perftools/deployment/grafana` répertoire dans votre copie locale du référentiel GitHub.
1. Déployer le **grafanaDeploy.json** modèle Resource Manager comme suit :

    ```bash
    export DATA_SOURCE="https://raw.githubusercontent.com/mspnp/spark-monitoring/master/perftools/deployment/grafana/AzureDataSource.sh"
    az group deployment create \
        --resource-group <resource-group-name> \
        --template-file grafanaDeploy.json \
        --parameters adminPass='<vm password>' dataSource=$DATA_SOURCE
    ```

Une fois le déploiement terminé, l’image bitnami de Grafana est installé sur l’ordinateur virtuel.

## <a name="update-the-grafana-password"></a>Mettre à jour le mot de passe de Grafana

Dans le cadre du processus d’installation, le script d’installation Grafana génère un mot de passe temporaire pour le **administrateur** utilisateur. Vous avez besoin de ce mot de passe temporaire pour vous connecter. Pour obtenir le mot de passe temporaire, procédez comme suit :  

1. Connectez-vous au portail Azure.  
1. Sélectionnez le groupe de ressources dans lequel les ressources ont été déployées.
1. Sélectionnez la machine virtuelle où Grafana a été installé. Si vous avez utilisé le nom de paramètre par défaut dans le modèle de déploiement, le nom de la machine virtuelle est précédé **sparkmonitoring-vm-grafana**.
1. Dans le **Support + dépannage** , cliquez sur **diagnostics de démarrage** pour ouvrir la page de diagnostics de démarrage.
1. Cliquez sur **journal série** dans la page de diagnostics de démarrage.
1. Recherchez la chaîne suivante : « Configuration de mot de passe applications Bitnami. »
1. Copiez le mot de passe dans un emplacement sûr.

Ensuite, modifiez le mot de passe administrateur Grafana en suivant ces étapes :

1. Dans le portail Azure, sélectionnez la machine virtuelle, puis cliquez sur **vue d’ensemble**.
1. Copiez l’adresse IP publique.
1. Ouvrez un navigateur web et accédez à l’URL suivante : `http://<IP address>:3000`.
1. À l’écran de connexion Grafana, entrez **administrateur** le nom d’utilisateur et utilisez le mot de passe Grafana lors des étapes précédentes.
1. Une fois connecté, sélectionnez **Configuration** (l’icône d’engrenage).
1. Sélectionnez **administrateur du serveur**.
1. Sur le **utilisateurs** onglet, sélectionnez le **administrateur** connexion.
1. Mettre à jour le mot de passe.

## <a name="create-an-azure-monitor-data-source"></a>Créer une source de données Azure Monitor

1. Créer un service principal qui permet de Grafana gérer l’accès à votre espace de travail Analytique de journal. Pour plus d’informations, consultez [créer un principal du service avec Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)

    ```bash
    az ad sp create-for-rbac --name http://<service principal name> --role "Log Analytics Reader"
    ```

1. Notez les valeurs de l’appId, password et tenant dans la sortie de cette commande :

    ```json
    {
        "appId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "displayName": "azure-cli-2019-03-27-00-33-39",
        "name": "http://<service principal name>",
        "password": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "tenant": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
    ```

1. Connectez-vous à Grafana comme décrit précédemment. Sélectionnez **Configuration** (l’icône d’engrenage), puis **des Sources de données**.
1. Dans le **des Sources de données** , cliquez sur **ajouter une source de données**.
1. Sélectionnez **Azure Monitor** comme type de source de données.
1. Dans le **paramètres** section, entrez un nom pour la source de données dans le **nom** zone de texte.
1. Dans le **détails de l’API Azure Monitor** section, entrez les informations suivantes :

    * ID d'abonnement : ID de votre abonnement Azure.
    * Id de locataire : L’ID de client précédemment.
    * Id de client : La valeur de « appId » indiquée précédemment.
    * Clé secrète client : La valeur de « password » indiquée précédemment.

1. Dans le **détails de l’API Azure Log Analytique** section, vérifiez le **même détails comme les API Azure Monitor** case à cocher.
1. Cliquez sur **enregistrer et tester**. Si la source de données Analytique de journal est configurée correctement, un message de réussite s’affiche.

## <a name="create-the-dashboard"></a>Créer le tableau de bord

Créer des tableaux de bord dans Grafana en suivant ces étapes :

1. Accédez à la `/perftools/dashboards/grafana` répertoire dans votre copie locale du référentiel GitHub.
1. Exécutez le script qui suit :

    ```bash
    export WORKSPACE=<your Azure Log Analytics workspace ID>
    export LOGTYPE=SparkListenerEvent_CL

    sh DashGen.sh
    ```

    La sortie du script est un fichier nommé **SparkMonitoringDash.json**.

1. Revenez au tableau de bord Grafana et sélectionnez **créer** (l’icône plus (+)).
1. Sélectionnez **Importer**.
1. Cliquez sur **télécharger le fichier .json**.
1. Sélectionnez le **SparkMonitoringDash.json** fichier créé à l’étape 2.
1. Dans le **Options** section sous **ALA**, sélectionnez la source de données Azure Monitor créée précédemment.
1. Cliquez sur **Importer**.

## <a name="visualizations-in-the-dashboards"></a>Visualisations dans les tableaux de bord

Les tableaux de bord Azure Log Analytique et de Grafana comprennent un ensemble de visualisations de série chronologique. Chaque graphique est tracé de séries chronologiques des données métriques associées à un Apache Spark [travail](https://spark.apache.org/docs/latest/job-scheduling.html), étapes de travail et des tâches qui composent chaque étape.

Les visualisations sont les suivantes :

### <a name="job-latency"></a>Latence du travail

Cette visualisation présente une latence d’exécution d’une tâche, qui est une vue grossière sur les performances globales d’un travail. Affiche la durée d’exécution du travail à partir du début jusqu'à la fin. Remarque heure de début de la tâche n’est pas identique à l’heure d’envoi de travail. Latence est représentée en tant que centiles (10 %, 30 %, 50 %, 90 %) de l’exécution du travail indexée par ID de cluster et les ID d’application.

### <a name="stage-latency"></a>Latence de la scène

La visualisation montre la latence de chaque étape par étape individuel par cluster et par application. Cette visualisation est utile pour identifier une phase spécifique qui s’exécute lentement.

### <a name="task-latency"></a>Latence de la tâche

Cette visualisation montre la latence d’exécution de tâches. Latence est représentée comme un centile de l’exécution de tâches par cluster, le nom de la scène et application.

### <a name="sum-task-execution-per-host"></a>Exécution de la tâche somme par hôte

Cette visualisation affiche la somme de la latence de l’exécution de tâches par hôte en cours d’exécution sur un cluster. Affichage de latence d’exécution de tâches par hôte identifie hôtes qui ont beaucoup globale tâche une latence plus élevée que les autres hôtes. Cela peut signifier que les tâches ont été inefficace ou inégalement distribuées aux ordinateurs hôtes.

### <a name="task-metrics"></a>Mesures de tâches

Cette visualisation présente un ensemble de métriques d’exécution pour l’exécution d’une tâche donnée. Ces mesures comprennent la taille et la durée d’un mélange de données et durée des opérations de sérialisation et la désérialisation. Pour l’ensemble des mesures, afficher la requête Analytique de journal pour le panneau. Cette visualisation est utile pour comprendre les opérations qui constituent une tâche et l’identification la consommation des ressources de chaque opération. Les pointes du graphique représentent des opérations coûteuses qui doivent être examinées.

### <a name="cluster-throughput"></a>Débit de cluster

Cette visualisation est une vue de haut niveau d’éléments de travail indexées par le cluster et des applications pour représenter la quantité de travail effectué par le cluster et des applications. Il indique le nombre de travaux, tâches et étapes réussies par cluster, l’application et étape par incréments d’une minute. 

### <a name="streaming-throughputlatency"></a>Diffusion en continu de débit et de latence

Cette visualisation concerne les mesures associées à une requête de diffusion en continu structurée. Les graphiques présente le nombre de lignes d’entrée par seconde et le nombre de lignes traitées par seconde. Les mesures de diffusion en continu sont également représentés par l’application. Ces mesures sont envoyés lorsque l’événement OnQueryProgress est généré comme traitement de la requête de diffusion en continu structurée et la visualisation représente latence en tant que la quantité de temps de diffusion en continu, en millisecondes, nécessaire pour exécuter un traitement de requêtes.

### <a name="resource-consumption-per-executor"></a>Consommation des ressources par exécuteur

Voici un ensemble de visualisations de tableau de bord afficher le type de ressource particulier et la façon dont elle est consommée par exécuteur sur chaque cluster. Ces visualisations pour identifier les valeurs hors norme la consommation de ressources par exécuteur. Par exemple, si la répartition du travail pour un exécuteur particulier est déréglée, la consommation de ressources va être élevée par rapport aux autres exécuteurs en cours d’exécution sur le cluster. Cela peut être identifiée par les pics de la consommation des ressources pour un exécuteur.

### <a name="executor-compute-time-metrics"></a>Mesures de temps de calcul exécuteur

Voici un ensemble de visualisations du tableau de bord qui afficher le rapport de l’exécuteur sérialiser le temps, désérialiser le temps, le temps processeur et l’heure de machine virtuelle Java pour les temps de calcul exécuteur globale. Cet exemple montre visuellement la quantité chacune de ces quatre mesures contribuent à l’exécuteur globale de traitement.

### <a name="shuffle-metrics"></a>Métriques de lecture aléatoire

L’ensemble final d’afficher des visualisations mesures associées à une requête de diffusion en continu structurée entre tous les exécuteurs de lecture aléatoire les données. Ceux-ci incluent la lecture aléatoire octets de lecture, de lecture aléatoire d’octets écrits, de lecture aléatoire de mémoire et l’utilisation du disque dans les requêtes où le système de fichiers est utilisé.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Résoudre les goulots d’étranglement de performances](./performance-troubleshooting.md)

<!-- links -->

[rm-cli]: /azure/azure-resource-manager/resource-group-template-deploy-cli
[sku]: /azure/templates/Microsoft.OperationalInsights/2015-11-01-preview/workspaces#sku-object