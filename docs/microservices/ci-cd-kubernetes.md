---
title: Création d’un pipeline de CI/CD pour les microservices sur Kubernetes
description: Décrit un exemple de pipeline CI/CD pour le déploiement de microservices Azure Kubernetes Service (AKS).
author: MikeWasson
ms.date: 04/11/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: e7da8a1b4111fb7856b919b40d033833a4e475e0
ms.sourcegitcommit: d58e6b2b891c9c99e951c59f15fce71addcb96b1
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/12/2019
ms.locfileid: "59533197"
---
<!-- markdownlint-disable MD040 -->

# <a name="building-a-cicd-pipeline-for-microservices-on-kubernetes"></a>Création d’un pipeline de CI/CD pour les microservices sur Kubernetes

Il peut être difficile de créer un processus CI/CD fiable pour une architecture de microservices. Les équipes individuelles doivent être en mesure de publier les services rapidement et de façon fiable, sans interrompre les autres équipes ou déstabiliser l’application dans son ensemble.

Cet article décrit un exemple de pipeline CI/CD pour le déploiement de microservices Azure Kubernetes Service (AKS). Chaque équipe et le projet sont différente, donc ne pas prendre cet article comme un ensemble de règles strictes. Au lieu de cela, il est destiné à être un point de départ pour la conception de votre propre processus CI/CD.

L’exemple de pipeline décrit ici a été créé pour une implémentation de référence de microservices appelée l’application Drone Delivery, que vous trouverez sur [GitHub][ri]. Décrit le scénario d’application [ici](./design/index.md#reference-implementation).

Les objectifs du pipeline peuvent être résumés comme suit :

- Équipes peuvent générer et déployer leurs services de manière indépendante.
- Modifications du code qui passent le processus d’intégration continue sont automatiquement déployées sur un environnement de type production.
- Critères de qualité sont appliquées à chaque étape du pipeline.
- Une nouvelle version d’un service peut être déployée côte à côte avec la version précédente.

Pour plus d’informations, consultez [CI/CD pour les architectures de microservices](./ci-cd.md).

## <a name="assumptions"></a>Hypothèses

Pour les besoins de cet exemple, voici quelques hypothèses sur l’équipe de développement et le code base :

- Le référentiel de code est un monorepo, avec les dossiers organisés par microservice.
- La stratégie de création de branche de l’équipe est basée sur un [développement de type tronc](https://trunkbaseddevelopment.com/).
- L’équipe utilise [mise en production des branches](/azure/devops/repos/git/git-branching-guidance?view=azure-devops#manage-releases) pour gérer les versions. Des versions distinctes sont créées pour chaque microservice.
- Le processus CI/CD utilise [Azure Pipelines](/azure/devops/pipelines/?view=azure-devops) pour générer, tester et déployer des microservices sur AKS.
- Les images de conteneur pour chaque microservice sont stockées dans [Azure Container Registry](/azure/container-registry/).
- L’équipe utilise des graphiques Helm pour empaqueter chaque microservice.

Ces hypothèses génèrent beaucoup de détails spécifiques du pipeline CI/CD. Toutefois, l’approche de base décrite ici être adapté pour d’autres processus, les outils et les services, tels que Jenkins ou Docker Hub.

## <a name="validation-builds"></a>Builds de validation

Supposons qu’un développeur travaille sur un microservice appelé le Service de remise. Lors du développement d’une nouvelle fonctionnalité, le développeur archive le code dans une branche de fonctionnalité. Par convention, les branches de fonctionnalité sont nommées `feature/*`.

![Workflow CI/CD](./images/aks-cicd-1.png)

Le fichier de définition de build inclut un déclencheur qui filtre par le nom de branche et le chemin d’accès source :

```yaml
trigger:
  batch: true
  branches:
    include:
    - master
    - feature/*
    - topic/*

    exclude:
    - feature/experimental/*
    - topic/experimental/*

  paths:
     include:
     - /src/shipping/delivery/
```

&#11162;Consultez le [fichier source](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/delivery/azure-pipelines-validation.yml).

Avec cette approche, chaque équipe peut avoir son propre du pipeline de build. Seul le code qui est archivé dans le `/src/shipping/delivery` dossier déclenche une build du Service de remise. Envoyant des validations vers une branche qui correspond au filtre déclenche une build CI. À ce stade du workflow , la build CI exécute une vérification minimale du code :

1. Générer le code.
1. Exécuter des tests unitaires.

L’objectif est de conserver des durées de génération courte, pour permettre au développeur d’obtenir des commentaires rapide. Une fois la fonctionnalité est prête à fusionner dans la branche maître, le développeur s’ouvre une demande de tirage. Cette action déclenche une autre build CI qui effectue des vérifications supplémentaires :

1. Générer le code.
1. Exécuter des tests unitaires.
1. Générez l’image de conteneur du runtime.
1. Exécuter des analyses de vulnérabilité sur l’image.

![Workflow CI/CD](./images/aks-cicd-2.png)

> [!NOTE]
> Dans les référentiels de DevOps Azure, vous pouvez définir [stratégies](/azure/devops/repos/git/branch-policies) pour protéger les branches. Par exemple, la stratégie peut exiger une build CI réussie ainsi qu’une validation d’un approbateur pour fusionner dans la branche master.

## <a name="full-cicd-build"></a>Version CI/CD complet

À un moment donné, l’équipe est prête à déployer une nouvelle version du service « Delivery ». Le Gestionnaire de versions crée une branche à partir du maître avec ce modèle d’affectation de noms : `release/<microservice name>/<semver>`. Par exemple : `release/delivery/v1.0.2`.

![Workflow CI/CD](./images/aks-cicd-3.png)

La création de cette branche déclenche une build CI complet qui s’exécute toutes les étapes précédentes plus (+) :

1. Envoyer l’image de conteneur à Azure Container Registry. L’image est étiquetée avec le numéro de version tiré du nom de la branche.
2. Exécutez `helm package` pour empaqueter le graphique Helm pour le service. Le graphique est également marqué d’un numéro de version.
3. Envoyer le package Helm à Container Registry.

En supposant que cette build réussit, elle déclenche un processus de déploiement (CD) à l’aide de des Pipelines Azure [pipeline de versions](/azure/devops/pipelines/release/what-is-release-management). Ce pipeline comporte les étapes suivantes :

1. Déployer le graphique Helm dans un environnement d’assurance qualité.
1. Un approbateur effectue une validation avant que le package passe en production. Consultez [Contrôler le déploiement de mise en production avec des approbations](/azure/devops/pipelines/release/approvals/approvals).
1. Modifier le balisage l’image Docker pour l’espace de noms de production dans Azure Container Registry. Par exemple, si l’étiquette actuelle est `myrepo.azurecr.io/delivery:v1.0.2`, l’étiquette de production est `myrepo.azurecr.io/prod/delivery:v1.0.2`.
1. Déployez le graphique Helm dans l’environnement de production.

Même dans un monorepo, ces tâches peuvent être limitées à des microservices individuels, afin que les équipes peuvent déployer avec haute vitesse. Le processus comporte des étapes manuelles : Approbation des demandes de tirage, création de branches de mise en production et approbation des déploiements sur le cluster de production. Ces étapes sont manuels par stratégie &mdash; qu’ils peuvent être automatisées si l’organisation préfère.

## <a name="isolation-of-environments"></a>Isolation des environnements

Vous avez plusieurs environnements sur lesquels vous déployez les services, notamment des environnements de développement, de test de détection de fumée, de test d’intégration, de test de chargement et, enfin, de production. Ces environnements ont besoin d’un certain niveau d’isolation. Dans Kubernetes, vous avez le choix entre l’isolation physique et l’isolation logique. Dans le cadre de l’isolation physique, le déploiement est effectué sur des clusters distincts. L’isolation logique utilise des espaces de noms et des stratégies, comme décrit précédemment.

Nous vous recommandons de créer un cluster de production dédié ainsi qu’un cluster distinct pour vos environnements de développement/test. L’isolation logique permet de séparer les environnements au sein du cluster de développement/test. Les services déployés sur le cluster de développement/test ne doivent jamais avoir accès à des magasins de données qui contiennent des données métier.

## <a name="build-process"></a>Processus de génération

Dans la mesure du possible, empaqueter votre processus de génération dans un conteneur Docker. Qui vous permet de créer vos artefacts de code à l’aide de Docker, sans avoir besoin de configurer l’environnement de génération sur chaque ordinateur de build. Un processus de génération en conteneur facilite pour monter en charge le pipeline CI en ajoutant de nouveaux agents de build. En outre, les développeurs de l’équipe peuvent générer le code simplement en exécutant le conteneur de build.

À l’aide de builds à plusieurs étapes dans Docker, vous pouvez définir l’environnement de génération et de l’image d’exécution dans un fichier Dockerfile unique. Par exemple, Voici un fichier Dockerfile qui crée une application ASP.NET Core :

```
FROM microsoft/dotnet:2.2-runtime AS base
WORKDIR /app

FROM microsoft/dotnet:2.2-sdk AS build
WORKDIR /src/Fabrikam.Workflow.Service

COPY Fabrikam.Workflow.Service/Fabrikam.Workflow.Service.csproj .
RUN dotnet restore Fabrikam.Workflow.Service.csproj

COPY Fabrikam.Workflow.Service/. .
RUN dotnet build Fabrikam.Workflow.Service.csproj -c Release -o /app

FROM build AS publish
RUN dotnet publish Fabrikam.Workflow.Service.csproj -c Release -o /app

FROM base AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "Fabrikam.Workflow.Service.dll"]
```

&#11162;Consultez le [fichier source](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/workflow/Dockerfile).

Ce fichier Dockerfile définit plusieurs étapes de génération. Notez que l’étape nommée `base` utilise le runtime ASP.NET Core, lors de l’étape nommée `build` utilise le Kit de développement logiciel complet ASP.NET Core. Le `build` étape est utilisée pour générer le projet ASP.NET Core. Mais le conteneur d’exécution final est généré à partir de `base`, qui contient uniquement le runtime et est considérablement plus petit que l’image complète de kit de développement logiciel.

### <a name="building-a-test-runner"></a>Création d’un test runner

Une autre bonne pratique consiste à exécuter des tests unitaires dans le conteneur. Par exemple, voici la partie d’un fichier Docker qui génère un test runner :

```
FROM build AS testrunner
WORKDIR /src/tests

COPY Fabrikam.Workflow.Service.Tests/*.csproj .
RUN dotnet restore Fabrikam.Workflow.Service.Tests.csproj

COPY Fabrikam.Workflow.Service.Tests/. .
ENTRYPOINT ["dotnet", "test", "--logger:trx"]
```

Un développeur peut utiliser ce fichier Docker pour exécuter les tests localement :

```bash
docker build . -t delivery-test:1 --target=testrunner
docker run -p 8080:8080 delivery-test:1
```

Le pipeline de CI doit également exécuter les tests dans le cadre de l’étape de vérification de build.

Notez que ce fichier utilise le Docker `ENTRYPOINT` commande pour exécuter les tests, pas le Docker `RUN` commande.

- Si vous utilisez le `RUN` l’exécution des tests chaque fois que vous générez l’image de la commande. À l’aide de `ENTRYPOINT`, les tests sont participer. Elles s’exécutent uniquement lorsque vous ciblez le `testrunner` étape.
- Un test ayant échoué n’entraîne pas la Docker `build` Échec de commande. De cette façon, vous pouvez distinguer le conteneur de générer des échecs d’échecs de test.
- Résultats des tests peuvent être enregistrés sur un volume monté.

### <a name="container-best-practices"></a>Meilleures pratiques de conteneur

Voici certains autres meilleures pratiques à prendre en compte pour les conteneurs :

- Définissez des conventions d’organisation pour les balises de conteneurs, la gestion des versions et des conventions de d’affectation de noms pour les ressources déployées sur le cluster (pod, services, etc.). Cela peut simplifier le diagnostic des problèmes de déploiement.

- Durant le cycle de développement et de test, le processus d’intégration et de livraison continue génère de nombreuses images de conteneur. Seuls certains de ces images sont candidates à la publication, puis uniquement certains de ces candidats seront promues en production de version. Disposer d’une stratégie claire du contrôle de version, afin que vous sachiez quelles images sont actuellement déployés en production et pouvant revenir à une version antérieure si nécessaire.

- Déployez toujours les balises de version d’un conteneur spécifique, pas `latest`.

- Utilisez [espaces de noms](/azure/container-registry/container-registry-best-practices#repository-namespaces) dans Azure Container Registry pour isoler les images qui sont approuvées pour la production à partir d’images qui sont toujours en cours de test. Ne déplacez pas une image dans l’espace de noms de production jusqu'à ce que vous êtes prêt à déployer en production. Si vous combinez cette pratique à la gestion sémantique des versions des images de conteneurs, vous pouvez réduire les risques de déploiement accidentel d’une version non approuvée pour publication.

- Suivez le principe du moindre privilège en exécutant des conteneurs en tant qu’un utilisateur sans privilège. Dans Kubernetes, vous pouvez créer une stratégie de sécurité pod qui empêche l’exécution en tant que conteneurs *racine*. Consultez [Pods empêcher de s’exécuter avec des privilèges Root](https://docs.bitnami.com/kubernetes/how-to/secure-kubernetes-cluster-psp/).

## <a name="helm-charts"></a>Graphiques Helm

Envisagez d’utiliser Helm pour gérer la génération et le déploiement des services. Voici quelques-unes des fonctionnalités de Helm qui aident à CI/CD :

- Souvent, un seul microservice est défini par plusieurs objets Kubernetes. Helm permet à ces objets être empaquetés dans un même graphique Helm.
- Un graphique peut être déployé avec une seule commande Helm, plutôt qu’une série de commandes kubectl.
- Les graphiques sont gérées explicitement. Utilisez Helm pour publier une version, afficher les versions et revenir à une version précédente. Suivi des mises à jour et des révisions, à l’aide du contrôle de version sémantique, avec possibilité de restaurer une version précédente
- Graphiques helm les modèles permettent d’éviter la duplication des informations, telles que les étiquettes et les sélecteurs, dans plusieurs fichiers.
- Helm peut gérer les dépendances entre les graphiques.
- Les graphiques peuvent être stockées dans un référentiel Helm, tels que Azure Container Registry et intégrés dans le pipeline de génération.

Pour plus d’informations sur l’utilisation de Container Registry comme dépôt Helm, consultez [Utiliser Azure Container Registry comme référentiel Helm pour les graphiques de votre application](/azure/container-registry/container-registry-helm-repos).

Un seul microservice peut impliquer plusieurs fichiers de configuration Kubernetes. La mise à jour un service peut signifier toucher tous ces fichiers pour mettre à jour les sélecteurs, les étiquettes et les balises d’image. Helm les traite comme un package unique, appelé un graphique et vous permet de mettre à jour facilement les fichiers YAML à l’aide de variables. Helm utilise un langage de modèle (basé sur des modèles de Go) pour vous permettre d’écrire des fichiers de configuration YAML paramétrables.

Par exemple, voici la partie d’un fichier YAML qui définit un déploiement :

```yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: {{ include "package.fullname" . | replace "." "" }}
  labels:
    app.kubernetes.io/name: {{ include "package.name" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}
  annotations:
    kubernetes.io/change-cause: {{ .Values.reason }}

...

  spec:
      containers:
      - name: &package-container_name fabrikam-package
        image: {{ .Values.dockerregistry }}/{{ .Values.image.repository }}:{{ .Values.image.tag }}
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        env:
        - name: LOG_LEVEL
          value: {{ .Values.log.level }}
```

&#11162;Consultez le [fichier source](https://github.com/mspnp/microservices-reference-implementation/blob/master/charts/package/templates/package-deploy.yaml).

Vous pouvez voir que le nom du déploiement, les étiquettes et conteneur de spécification de tous les paramètres de modèle d’utilisation, qui sont fournies au moment du déploiement. Par exemple, à partir de la ligne de commande :

```bash
helm install $HELM_CHARTS/package/ \
     --set image.tag=0.1.0 \
     --set image.repository=package \
     --set dockerregistry=$ACR_SERVER \
     --namespace backend \
     --name package-v0.1.0
```

Bien que votre pipeline CI/CD peut installer un graphique directement dans Kubernetes, nous vous recommandons de création d’une archive de graphique (fichier .tgz) et en plaçant le graphique depuis un référentiel Helm tels que Azure Container Registry. Pour plus d’informations, consultez [les applications basées sur le Docker de Package dans les graphiques Helm dans les Pipelines Azure](/azure/devops/pipelines/languages/helm?view=azure-devops).

Envisagez de déploiement Helm sur son propre espace de noms et à l’aide du contrôle d’accès en fonction du rôle (RBAC) pour restreindre le pouvoir déployer vers les espaces de noms. Pour plus d’informations, consultez [contrôle d’accès en fonction du rôle](https://helm.sh/docs/using_helm/#helm-and-role-based-access-control) dans la documentation de Helm.

### <a name="revisions"></a>Révisions

Graphiques helm ont toujours un numéro de version, qui doit utiliser [semver](https://semver.org/). Un graphique peut également avoir un `appVersion`. Ce champ est facultatif et ne doit pas être liées à la version de graphique. Certaines équipes souhaiterez versions d’application séparément à partir de mises à jour pour les graphiques. Mais une approche plus simple consiste à utiliser un numéro de version, donc il existe une relation 1:1 entre la version de graphique et de la version de l’application. De cette façon, vous pouvez stocker un graphique par version et déployer facilement la version souhaitée :

```bash
helm install <package-chart-name> --version <desiredVersion>
```

Une autre bonne pratique consiste à fournir une annotation de la cause de modification dans le modèle de déploiement :

```yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: {{ include "delivery.fullname" . | replace "." "" }}
  labels:
     ...
  annotations:
    kubernetes.io/change-cause: {{ .Values.reason }}
```

Cela vous permet d’afficher le champ de la cause de modification pour chaque révision, à l’aide de la `kubectl rollout history` commande. Dans l’exemple précédent, la cause de modification est fournie comme paramètre graphique Helm.

```bash
$ kubectl rollout history deployments/delivery-v010 -n backend
deployment.extensions/delivery-v010
REVISION  CHANGE-CAUSE
1         Initial deployment
```

Vous pouvez également utiliser le `helm list` commande pour afficher l’historique de révision :

```bash
~$ helm list
NAME            REVISION    UPDATED                     STATUS        CHART            APP VERSION     NAMESPACE
delivery-v0.1.0 1           Sun Apr  7 00:25:30 2019    DEPLOYED      delivery-v0.1.0  v0.1.0          backend
```

> [!TIP]
> Utilisez le `--history-max` indicateur lors de l’initialisation de Helm. Ce paramètre limite le nombre de révisions Tiller enregistre dans son historique. Tiller stocke l’historique de révision dans configmaps. Si vous sortez souvent mises à jour, le configmaps peut devenir très volumineux, sauf si vous limitez la taille de l’historique.

## <a name="azure-devops-pipeline"></a>Pipeline de DevOps Azure

Dans les Pipelines d’Azure, les pipelines sont divisés en *générer des pipelines* et *mise en production de pipelines*. Le pipeline de build s’exécute le processus d’intégration continue et crée les artefacts de build. Pour une architecture de microservices sur Kubernetes, ces artefacts sont les images de conteneur et les graphiques Helm qui définissent chaque microservice. Le pipeline de mise en production exécute ce processus CD qui déploie un microservice dans un cluster.

Selon le flux CI décrit plus haut dans cet article, un pipeline de build peut être constitué des tâches suivantes :

1. Créer le conteneur d’exécuteur de test.

    ```yaml
    - task: Docker@1
      inputs:
        azureSubscriptionEndpoint: $(AzureSubscription)
        azureContainerRegistry: $(AzureContainerRegistry)
        arguments: '--pull --target testrunner'
        dockerFile: $(System.DefaultWorkingDirectory)/$(dockerFileName)
        imageName: '$(imageName)-test'
    ```

1. Exécuter les tests, en appelant docker à exécuter sur le conteneur d’exécuteur de test.

    ```yaml
    - task: Docker@1
      inputs:
        azureSubscriptionEndpoint: $(AzureSubscription)
        azureContainerRegistry: $(AzureContainerRegistry)
        command: 'run'
        containerName: testrunner
        volumes: '$(System.DefaultWorkingDirectory)/TestResults:/app/tests/TestResults'
        imageName: '$(imageName)-test'
        runInBackground: false
    ```

1. Publier les résultats des tests. Consultez [intégrer build et tâches de test](/azure/devops/pipelines/languages/docker?view=azure-devops&tabs=yaml#integrate-build-and-test-tasks).

    ```yaml
    - task: PublishTestResults@2
      inputs:
        testResultsFormat: 'VSTest'
        testResultsFiles: 'TestResults/*.trx'
        searchFolder: '$(System.DefaultWorkingDirectory)'
        publishRunAttachments: true
    ```

1. Créer le conteneur de runtime.

    ```yaml
    - task: Docker@1
      inputs:
        azureSubscriptionEndpoint: $(AzureSubscription)
        azureContainerRegistry: $(AzureContainerRegistry)
        dockerFile: $(System.DefaultWorkingDirectory)/$(dockerFileName)
        includeLatestTag: false
        imageName: '$(imageName)'
    ```

1. Push vers le conteneur vers Azure Container Registry (ou autres container registry).

    ```yaml
    - task: Docker@1
      inputs:
        azureSubscriptionEndpoint: $(AzureSubscription)
        azureContainerRegistry: $(AzureContainerRegistry)
        command: 'Push an image'
        imageName: '$(imageName)'
        includeSourceTags: false
    ```

1. Le graphique Helm du package.

    ```yaml
    - task: HelmDeploy@0
      inputs:
        command: package
        chartPath: $(chartPath)
        chartVersion: $(Build.SourceBranchName)
        arguments: '--app-version $(Build.SourceBranchName)'
    ```

1. Envoyer le package Helm à Azure Container Registry (ou un autre référentiel Helm).

    ```yaml
    task: AzureCLI@1
      inputs:
        azureSubscription: $(AzureSubscription)
        scriptLocation: inlineScript
        inlineScript: |
        az acr helm push $(System.ArtifactsDirectory)/$(repositoryName)-$(Build.SourceBranchName).tgz --name $(AzureContainerRegistry);
    ```

&#11162;Consultez le [fichier source](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/delivery/azure-pipelines-ci.yml).

La sortie du pipeline CI est une image de conteneur de prêt pour la production et un graphique Helm mis à jour pour le microservice. À ce stade, le pipeline de version peut reprendre. Il effectue les étapes suivantes :

- Déployer des environnements QA/dev/intermédiaires.
- Attendez un approbateur approuver ou rejeter le déploiement.
- Modifier le balisage d’image de conteneur pour la mise en production
- Envoyer la balise de version dans le Registre de conteneur.
- Mettre à niveau le graphique Helm dans le cluster de production.

Pour plus d’informations sur la création d’un pipeline de versions, consultez [pipelines de lancement, versions de brouillon et options de version](/azure/devops/pipelines/release/?view=azure-devops).

Le diagramme suivant illustre le processus de CI/CD de bout en bout décrit dans cet article :

![Pipeline de CD/CD](./images/aks-cicd-flow.png)

## <a name="next-steps"></a>Étapes suivantes

Cet article a été basé sur une implémentation de référence que vous trouverez sur [GitHub][ri].

[ri]: https://github.com/mspnp/microservices-reference-implementation