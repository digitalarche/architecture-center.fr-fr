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

# <a name="building-a-cicd-pipeline-for-microservices-on-kubernetes"></a><span data-ttu-id="7b38f-103">Création d’un pipeline de CI/CD pour les microservices sur Kubernetes</span><span class="sxs-lookup"><span data-stu-id="7b38f-103">Building a CI/CD pipeline for microservices on Kubernetes</span></span>

<span data-ttu-id="7b38f-104">Il peut être difficile de créer un processus CI/CD fiable pour une architecture de microservices.</span><span class="sxs-lookup"><span data-stu-id="7b38f-104">It can be challenging to create a reliable CI/CD process for a microservices architecture.</span></span> <span data-ttu-id="7b38f-105">Les équipes individuelles doivent être en mesure de publier les services rapidement et de façon fiable, sans interrompre les autres équipes ou déstabiliser l’application dans son ensemble.</span><span class="sxs-lookup"><span data-stu-id="7b38f-105">Individual teams must be able to release services quickly and reliably, without disrupting other teams or destabilizing the application as a whole.</span></span>

<span data-ttu-id="7b38f-106">Cet article décrit un exemple de pipeline CI/CD pour le déploiement de microservices Azure Kubernetes Service (AKS).</span><span class="sxs-lookup"><span data-stu-id="7b38f-106">This article describes an example CI/CD pipeline for deploying microservices to Azure Kubernetes Service (AKS).</span></span> <span data-ttu-id="7b38f-107">Chaque équipe et le projet sont différente, donc ne pas prendre cet article comme un ensemble de règles strictes.</span><span class="sxs-lookup"><span data-stu-id="7b38f-107">Every team and project is different, so don't take this article as a set of hard-and-fast rules.</span></span> <span data-ttu-id="7b38f-108">Au lieu de cela, il est destiné à être un point de départ pour la conception de votre propre processus CI/CD.</span><span class="sxs-lookup"><span data-stu-id="7b38f-108">Instead, it's meant to be a starting point for designing your own CI/CD process.</span></span>

<span data-ttu-id="7b38f-109">L’exemple de pipeline décrit ici a été créé pour une implémentation de référence de microservices appelée l’application Drone Delivery, que vous trouverez sur [GitHub][ri].</span><span class="sxs-lookup"><span data-stu-id="7b38f-109">The example pipeline described here was created for a microservices reference implementation called the Drone Delivery application, which you can find on [GitHub][ri].</span></span> <span data-ttu-id="7b38f-110">Décrit le scénario d’application [ici](./design/index.md#reference-implementation).</span><span class="sxs-lookup"><span data-stu-id="7b38f-110">The application scenario is described [here](./design/index.md#reference-implementation).</span></span>

<span data-ttu-id="7b38f-111">Les objectifs du pipeline peuvent être résumés comme suit :</span><span class="sxs-lookup"><span data-stu-id="7b38f-111">The goals of the pipeline can be summarized as follows:</span></span>

- <span data-ttu-id="7b38f-112">Équipes peuvent générer et déployer leurs services de manière indépendante.</span><span class="sxs-lookup"><span data-stu-id="7b38f-112">Teams can build and deploy their services independently.</span></span>
- <span data-ttu-id="7b38f-113">Modifications du code qui passent le processus d’intégration continue sont automatiquement déployées sur un environnement de type production.</span><span class="sxs-lookup"><span data-stu-id="7b38f-113">Code changes that pass the CI process are automatically deployed to a production-like environment.</span></span>
- <span data-ttu-id="7b38f-114">Critères de qualité sont appliquées à chaque étape du pipeline.</span><span class="sxs-lookup"><span data-stu-id="7b38f-114">Quality gates are enforced at each stage of the pipeline.</span></span>
- <span data-ttu-id="7b38f-115">Une nouvelle version d’un service peut être déployée côte à côte avec la version précédente.</span><span class="sxs-lookup"><span data-stu-id="7b38f-115">A new version of a service can be deployed side by side with the previous version.</span></span>

<span data-ttu-id="7b38f-116">Pour plus d’informations, consultez [CI/CD pour les architectures de microservices](./ci-cd.md).</span><span class="sxs-lookup"><span data-stu-id="7b38f-116">For more background, see [CI/CD for microservices architectures](./ci-cd.md).</span></span>

## <a name="assumptions"></a><span data-ttu-id="7b38f-117">Hypothèses</span><span class="sxs-lookup"><span data-stu-id="7b38f-117">Assumptions</span></span>

<span data-ttu-id="7b38f-118">Pour les besoins de cet exemple, voici quelques hypothèses sur l’équipe de développement et le code base :</span><span class="sxs-lookup"><span data-stu-id="7b38f-118">For purposes of this example, here are some assumptions about the development team and the code base:</span></span>

- <span data-ttu-id="7b38f-119">Le référentiel de code est un monorepo, avec les dossiers organisés par microservice.</span><span class="sxs-lookup"><span data-stu-id="7b38f-119">The code repository is a monorepo, with folders organized by microservice.</span></span>
- <span data-ttu-id="7b38f-120">La stratégie de création de branche de l’équipe est basée sur un [développement de type tronc](https://trunkbaseddevelopment.com/).</span><span class="sxs-lookup"><span data-stu-id="7b38f-120">The team's branching strategy is based on [trunk-based development](https://trunkbaseddevelopment.com/).</span></span>
- <span data-ttu-id="7b38f-121">L’équipe utilise [mise en production des branches](/azure/devops/repos/git/git-branching-guidance?view=azure-devops#manage-releases) pour gérer les versions.</span><span class="sxs-lookup"><span data-stu-id="7b38f-121">The team uses [release branches](/azure/devops/repos/git/git-branching-guidance?view=azure-devops#manage-releases) to manage releases.</span></span> <span data-ttu-id="7b38f-122">Des versions distinctes sont créées pour chaque microservice.</span><span class="sxs-lookup"><span data-stu-id="7b38f-122">Separate releases are created for each microservice.</span></span>
- <span data-ttu-id="7b38f-123">Le processus CI/CD utilise [Azure Pipelines](/azure/devops/pipelines/?view=azure-devops) pour générer, tester et déployer des microservices sur AKS.</span><span class="sxs-lookup"><span data-stu-id="7b38f-123">The CI/CD process uses [Azure Pipelines](/azure/devops/pipelines/?view=azure-devops) to build, test, and deploy the microservices to AKS.</span></span>
- <span data-ttu-id="7b38f-124">Les images de conteneur pour chaque microservice sont stockées dans [Azure Container Registry](/azure/container-registry/).</span><span class="sxs-lookup"><span data-stu-id="7b38f-124">The container images for each microservice are stored in [Azure Container Registry](/azure/container-registry/).</span></span>
- <span data-ttu-id="7b38f-125">L’équipe utilise des graphiques Helm pour empaqueter chaque microservice.</span><span class="sxs-lookup"><span data-stu-id="7b38f-125">The team uses Helm charts to package each microservice.</span></span>

<span data-ttu-id="7b38f-126">Ces hypothèses génèrent beaucoup de détails spécifiques du pipeline CI/CD.</span><span class="sxs-lookup"><span data-stu-id="7b38f-126">These assumptions drive many of the specific details of the CI/CD pipeline.</span></span> <span data-ttu-id="7b38f-127">Toutefois, l’approche de base décrite ici être adapté pour d’autres processus, les outils et les services, tels que Jenkins ou Docker Hub.</span><span class="sxs-lookup"><span data-stu-id="7b38f-127">However, the basic approach described here be adapted for other processes, tools, and services, such as Jenkins or Docker Hub.</span></span>

## <a name="validation-builds"></a><span data-ttu-id="7b38f-128">Builds de validation</span><span class="sxs-lookup"><span data-stu-id="7b38f-128">Validation builds</span></span>

<span data-ttu-id="7b38f-129">Supposons qu’un développeur travaille sur un microservice appelé le Service de remise.</span><span class="sxs-lookup"><span data-stu-id="7b38f-129">Suppose that a developer is working on a microservice called the Delivery Service.</span></span> <span data-ttu-id="7b38f-130">Lors du développement d’une nouvelle fonctionnalité, le développeur archive le code dans une branche de fonctionnalité.</span><span class="sxs-lookup"><span data-stu-id="7b38f-130">While developing a new feature, the developer checks code into a feature branch.</span></span> <span data-ttu-id="7b38f-131">Par convention, les branches de fonctionnalité sont nommées `feature/*`.</span><span class="sxs-lookup"><span data-stu-id="7b38f-131">By convention, feature branches are named `feature/*`.</span></span>

![Workflow CI/CD](./images/aks-cicd-1.png)

<span data-ttu-id="7b38f-133">Le fichier de définition de build inclut un déclencheur qui filtre par le nom de branche et le chemin d’accès source :</span><span class="sxs-lookup"><span data-stu-id="7b38f-133">The build definition file includes a trigger that filters by the branch name and the source path:</span></span>

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

<span data-ttu-id="7b38f-134">&#11162;Consultez le [fichier source](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/delivery/azure-pipelines-validation.yml).</span><span class="sxs-lookup"><span data-stu-id="7b38f-134">&#11162; See the [source file](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/delivery/azure-pipelines-validation.yml).</span></span>

<span data-ttu-id="7b38f-135">Avec cette approche, chaque équipe peut avoir son propre du pipeline de build.</span><span class="sxs-lookup"><span data-stu-id="7b38f-135">Using this approach, each team can have its own build pipeline.</span></span> <span data-ttu-id="7b38f-136">Seul le code qui est archivé dans le `/src/shipping/delivery` dossier déclenche une build du Service de remise.</span><span class="sxs-lookup"><span data-stu-id="7b38f-136">Only code that is checked into the `/src/shipping/delivery` folder triggers a build of the Delivery Service.</span></span> <span data-ttu-id="7b38f-137">Envoyant des validations vers une branche qui correspond au filtre déclenche une build CI.</span><span class="sxs-lookup"><span data-stu-id="7b38f-137">Pushing commits to a branch that matches the filter triggers a CI build.</span></span> <span data-ttu-id="7b38f-138">À ce stade du workflow , la build CI exécute une vérification minimale du code :</span><span class="sxs-lookup"><span data-stu-id="7b38f-138">At this point in the workflow, the CI build runs some minimal code verification:</span></span>

1. <span data-ttu-id="7b38f-139">Générer le code.</span><span class="sxs-lookup"><span data-stu-id="7b38f-139">Build the code.</span></span>
1. <span data-ttu-id="7b38f-140">Exécuter des tests unitaires.</span><span class="sxs-lookup"><span data-stu-id="7b38f-140">Run unit tests.</span></span>

<span data-ttu-id="7b38f-141">L’objectif est de conserver des durées de génération courte, pour permettre au développeur d’obtenir des commentaires rapide.</span><span class="sxs-lookup"><span data-stu-id="7b38f-141">The goal is to keep build times short, so the developer can get quick feedback.</span></span> <span data-ttu-id="7b38f-142">Une fois la fonctionnalité est prête à fusionner dans la branche maître, le développeur s’ouvre une demande de tirage.</span><span class="sxs-lookup"><span data-stu-id="7b38f-142">Once the feature is ready to merge into master, the developer opens a PR.</span></span> <span data-ttu-id="7b38f-143">Cette action déclenche une autre build CI qui effectue des vérifications supplémentaires :</span><span class="sxs-lookup"><span data-stu-id="7b38f-143">This triggers another CI build that performs some additional checks:</span></span>

1. <span data-ttu-id="7b38f-144">Générer le code.</span><span class="sxs-lookup"><span data-stu-id="7b38f-144">Build the code.</span></span>
1. <span data-ttu-id="7b38f-145">Exécuter des tests unitaires.</span><span class="sxs-lookup"><span data-stu-id="7b38f-145">Run unit tests.</span></span>
1. <span data-ttu-id="7b38f-146">Générez l’image de conteneur du runtime.</span><span class="sxs-lookup"><span data-stu-id="7b38f-146">Build the runtime container image.</span></span>
1. <span data-ttu-id="7b38f-147">Exécuter des analyses de vulnérabilité sur l’image.</span><span class="sxs-lookup"><span data-stu-id="7b38f-147">Run vulnerability scans on the image.</span></span>

![Workflow CI/CD](./images/aks-cicd-2.png)

> [!NOTE]
> <span data-ttu-id="7b38f-149">Dans les référentiels de DevOps Azure, vous pouvez définir [stratégies](/azure/devops/repos/git/branch-policies) pour protéger les branches.</span><span class="sxs-lookup"><span data-stu-id="7b38f-149">In Azure DevOps Repos, you can define [policies](/azure/devops/repos/git/branch-policies) to protect branches.</span></span> <span data-ttu-id="7b38f-150">Par exemple, la stratégie peut exiger une build CI réussie ainsi qu’une validation d’un approbateur pour fusionner dans la branche master.</span><span class="sxs-lookup"><span data-stu-id="7b38f-150">For example, the policy could require a successful CI build plus a sign-off from an approver in order to merge into master.</span></span>

## <a name="full-cicd-build"></a><span data-ttu-id="7b38f-151">Version CI/CD complet</span><span class="sxs-lookup"><span data-stu-id="7b38f-151">Full CI/CD build</span></span>

<span data-ttu-id="7b38f-152">À un moment donné, l’équipe est prête à déployer une nouvelle version du service « Delivery ».</span><span class="sxs-lookup"><span data-stu-id="7b38f-152">At some point, the team is ready to deploy a new version of the Delivery service.</span></span> <span data-ttu-id="7b38f-153">Le Gestionnaire de versions crée une branche à partir du maître avec ce modèle d’affectation de noms : `release/<microservice name>/<semver>`.</span><span class="sxs-lookup"><span data-stu-id="7b38f-153">The release manager creates a branch from master with this naming pattern: `release/<microservice name>/<semver>`.</span></span> <span data-ttu-id="7b38f-154">Par exemple : `release/delivery/v1.0.2`.</span><span class="sxs-lookup"><span data-stu-id="7b38f-154">For example, `release/delivery/v1.0.2`.</span></span>

![Workflow CI/CD](./images/aks-cicd-3.png)

<span data-ttu-id="7b38f-156">La création de cette branche déclenche une build CI complet qui s’exécute toutes les étapes précédentes plus (+) :</span><span class="sxs-lookup"><span data-stu-id="7b38f-156">Creation of this branch triggers a full CI build that runs all of the previous steps plus:</span></span>

1. <span data-ttu-id="7b38f-157">Envoyer l’image de conteneur à Azure Container Registry.</span><span class="sxs-lookup"><span data-stu-id="7b38f-157">Push the container image to Azure Container Registry.</span></span> <span data-ttu-id="7b38f-158">L’image est étiquetée avec le numéro de version tiré du nom de la branche.</span><span class="sxs-lookup"><span data-stu-id="7b38f-158">The image is tagged with the version number taken from the branch name.</span></span>
2. <span data-ttu-id="7b38f-159">Exécutez `helm package` pour empaqueter le graphique Helm pour le service.</span><span class="sxs-lookup"><span data-stu-id="7b38f-159">Run `helm package` to package the Helm chart for the service.</span></span> <span data-ttu-id="7b38f-160">Le graphique est également marqué d’un numéro de version.</span><span class="sxs-lookup"><span data-stu-id="7b38f-160">The chart is also tagged with a version number.</span></span>
3. <span data-ttu-id="7b38f-161">Envoyer le package Helm à Container Registry.</span><span class="sxs-lookup"><span data-stu-id="7b38f-161">Push the Helm package to Container Registry.</span></span>

<span data-ttu-id="7b38f-162">En supposant que cette build réussit, elle déclenche un processus de déploiement (CD) à l’aide de des Pipelines Azure [pipeline de versions](/azure/devops/pipelines/release/what-is-release-management).</span><span class="sxs-lookup"><span data-stu-id="7b38f-162">Assuming this build succeeds, it triggers a deployment (CD) process using an Azure Pipelines [release pipeline](/azure/devops/pipelines/release/what-is-release-management).</span></span> <span data-ttu-id="7b38f-163">Ce pipeline comporte les étapes suivantes :</span><span class="sxs-lookup"><span data-stu-id="7b38f-163">This pipeline has the following steps:</span></span>

1. <span data-ttu-id="7b38f-164">Déployer le graphique Helm dans un environnement d’assurance qualité.</span><span class="sxs-lookup"><span data-stu-id="7b38f-164">Deploy the Helm chart to a QA environment.</span></span>
1. <span data-ttu-id="7b38f-165">Un approbateur effectue une validation avant que le package passe en production.</span><span class="sxs-lookup"><span data-stu-id="7b38f-165">An approver signs off before the package moves to production.</span></span> <span data-ttu-id="7b38f-166">Consultez [Contrôler le déploiement de mise en production avec des approbations](/azure/devops/pipelines/release/approvals/approvals).</span><span class="sxs-lookup"><span data-stu-id="7b38f-166">See [Release deployment control using approvals](/azure/devops/pipelines/release/approvals/approvals).</span></span>
1. <span data-ttu-id="7b38f-167">Modifier le balisage l’image Docker pour l’espace de noms de production dans Azure Container Registry.</span><span class="sxs-lookup"><span data-stu-id="7b38f-167">Retag the Docker image for the production namespace in Azure Container Registry.</span></span> <span data-ttu-id="7b38f-168">Par exemple, si l’étiquette actuelle est `myrepo.azurecr.io/delivery:v1.0.2`, l’étiquette de production est `myrepo.azurecr.io/prod/delivery:v1.0.2`.</span><span class="sxs-lookup"><span data-stu-id="7b38f-168">For example, if the current tag is `myrepo.azurecr.io/delivery:v1.0.2`, the production tag is `myrepo.azurecr.io/prod/delivery:v1.0.2`.</span></span>
1. <span data-ttu-id="7b38f-169">Déployez le graphique Helm dans l’environnement de production.</span><span class="sxs-lookup"><span data-stu-id="7b38f-169">Deploy the Helm chart to the production environment.</span></span>

<span data-ttu-id="7b38f-170">Même dans un monorepo, ces tâches peuvent être limitées à des microservices individuels, afin que les équipes peuvent déployer avec haute vitesse.</span><span class="sxs-lookup"><span data-stu-id="7b38f-170">Even in a monorepo, these tasks can be scoped to individual microservices, so that teams can deploy with high velocity.</span></span> <span data-ttu-id="7b38f-171">Le processus comporte des étapes manuelles : Approbation des demandes de tirage, création de branches de mise en production et approbation des déploiements sur le cluster de production.</span><span class="sxs-lookup"><span data-stu-id="7b38f-171">The process has some manual steps: Approving PRs, creating release branches, and approving deployments into the production cluster.</span></span> <span data-ttu-id="7b38f-172">Ces étapes sont manuels par stratégie &mdash; qu’ils peuvent être automatisées si l’organisation préfère.</span><span class="sxs-lookup"><span data-stu-id="7b38f-172">These steps are manual by policy &mdash; they could be automated if the organization prefers.</span></span>

## <a name="isolation-of-environments"></a><span data-ttu-id="7b38f-173">Isolation des environnements</span><span class="sxs-lookup"><span data-stu-id="7b38f-173">Isolation of environments</span></span>

<span data-ttu-id="7b38f-174">Vous avez plusieurs environnements sur lesquels vous déployez les services, notamment des environnements de développement, de test de détection de fumée, de test d’intégration, de test de chargement et, enfin, de production.</span><span class="sxs-lookup"><span data-stu-id="7b38f-174">You will have multiple environments where you deploy services, including environments for development, smoke testing, integration testing, load testing, and finally production.</span></span> <span data-ttu-id="7b38f-175">Ces environnements ont besoin d’un certain niveau d’isolation.</span><span class="sxs-lookup"><span data-stu-id="7b38f-175">These environments need some level of isolation.</span></span> <span data-ttu-id="7b38f-176">Dans Kubernetes, vous avez le choix entre l’isolation physique et l’isolation logique.</span><span class="sxs-lookup"><span data-stu-id="7b38f-176">In Kubernetes, you have a choice between physical isolation and logical isolation.</span></span> <span data-ttu-id="7b38f-177">Dans le cadre de l’isolation physique, le déploiement est effectué sur des clusters distincts.</span><span class="sxs-lookup"><span data-stu-id="7b38f-177">Physical isolation means deploying to separate clusters.</span></span> <span data-ttu-id="7b38f-178">L’isolation logique utilise des espaces de noms et des stratégies, comme décrit précédemment.</span><span class="sxs-lookup"><span data-stu-id="7b38f-178">Logical isolation makes use of namespaces and policies, as described earlier.</span></span>

<span data-ttu-id="7b38f-179">Nous vous recommandons de créer un cluster de production dédié ainsi qu’un cluster distinct pour vos environnements de développement/test.</span><span class="sxs-lookup"><span data-stu-id="7b38f-179">Our recommendation is to create a dedicated production cluster along with a separate cluster for your dev/test environments.</span></span> <span data-ttu-id="7b38f-180">L’isolation logique permet de séparer les environnements au sein du cluster de développement/test.</span><span class="sxs-lookup"><span data-stu-id="7b38f-180">Use logical isolation to separate environments within the dev/test cluster.</span></span> <span data-ttu-id="7b38f-181">Les services déployés sur le cluster de développement/test ne doivent jamais avoir accès à des magasins de données qui contiennent des données métier.</span><span class="sxs-lookup"><span data-stu-id="7b38f-181">Services deployed to the dev/test cluster should never have access to data stores that hold business data.</span></span>

## <a name="build-process"></a><span data-ttu-id="7b38f-182">Processus de génération</span><span class="sxs-lookup"><span data-stu-id="7b38f-182">Build process</span></span>

<span data-ttu-id="7b38f-183">Dans la mesure du possible, empaqueter votre processus de génération dans un conteneur Docker.</span><span class="sxs-lookup"><span data-stu-id="7b38f-183">When possible, package your build process into a Docker container.</span></span> <span data-ttu-id="7b38f-184">Qui vous permet de créer vos artefacts de code à l’aide de Docker, sans avoir besoin de configurer l’environnement de génération sur chaque ordinateur de build.</span><span class="sxs-lookup"><span data-stu-id="7b38f-184">That allows you to build your code artifacts using Docker, without needing to configure the build environment on each build machine.</span></span> <span data-ttu-id="7b38f-185">Un processus de génération en conteneur facilite pour monter en charge le pipeline CI en ajoutant de nouveaux agents de build.</span><span class="sxs-lookup"><span data-stu-id="7b38f-185">A containerized build process makes it easy to scale out the CI pipeline by adding new build agents.</span></span> <span data-ttu-id="7b38f-186">En outre, les développeurs de l’équipe peuvent générer le code simplement en exécutant le conteneur de build.</span><span class="sxs-lookup"><span data-stu-id="7b38f-186">Also, any developer on the team can build the code simply by running the build container.</span></span>

<span data-ttu-id="7b38f-187">À l’aide de builds à plusieurs étapes dans Docker, vous pouvez définir l’environnement de génération et de l’image d’exécution dans un fichier Dockerfile unique.</span><span class="sxs-lookup"><span data-stu-id="7b38f-187">By using multi-stage builds in Docker, you can define the build environment and the runtime image in a single Dockerfile.</span></span> <span data-ttu-id="7b38f-188">Par exemple, Voici un fichier Dockerfile qui crée une application ASP.NET Core :</span><span class="sxs-lookup"><span data-stu-id="7b38f-188">For example, here's a Dockerfile that builds an ASP.NET Core application:</span></span>

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

<span data-ttu-id="7b38f-189">&#11162;Consultez le [fichier source](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/workflow/Dockerfile).</span><span class="sxs-lookup"><span data-stu-id="7b38f-189">&#11162; See the [source file](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/workflow/Dockerfile).</span></span>

<span data-ttu-id="7b38f-190">Ce fichier Dockerfile définit plusieurs étapes de génération.</span><span class="sxs-lookup"><span data-stu-id="7b38f-190">This Dockerfile defines several build stages.</span></span> <span data-ttu-id="7b38f-191">Notez que l’étape nommée `base` utilise le runtime ASP.NET Core, lors de l’étape nommée `build` utilise le Kit de développement logiciel complet ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="7b38f-191">Notice that the stage named `base` uses the ASP.NET Core runtime, while the stage named `build` uses the full ASP.NET Core SDK.</span></span> <span data-ttu-id="7b38f-192">Le `build` étape est utilisée pour générer le projet ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="7b38f-192">The `build` stage is used to build the ASP.NET Core project.</span></span> <span data-ttu-id="7b38f-193">Mais le conteneur d’exécution final est généré à partir de `base`, qui contient uniquement le runtime et est considérablement plus petit que l’image complète de kit de développement logiciel.</span><span class="sxs-lookup"><span data-stu-id="7b38f-193">But the final runtime container is built from `base`, which contains just the runtime and is significantly smaller than the full SDK image.</span></span>

### <a name="building-a-test-runner"></a><span data-ttu-id="7b38f-194">Création d’un test runner</span><span class="sxs-lookup"><span data-stu-id="7b38f-194">Building a test runner</span></span>

<span data-ttu-id="7b38f-195">Une autre bonne pratique consiste à exécuter des tests unitaires dans le conteneur.</span><span class="sxs-lookup"><span data-stu-id="7b38f-195">Another good practice is to run unit tests in the container.</span></span> <span data-ttu-id="7b38f-196">Par exemple, voici la partie d’un fichier Docker qui génère un test runner :</span><span class="sxs-lookup"><span data-stu-id="7b38f-196">For example, here is part of a Docker file that builds a test runner:</span></span>

```
FROM build AS testrunner
WORKDIR /src/tests

COPY Fabrikam.Workflow.Service.Tests/*.csproj .
RUN dotnet restore Fabrikam.Workflow.Service.Tests.csproj

COPY Fabrikam.Workflow.Service.Tests/. .
ENTRYPOINT ["dotnet", "test", "--logger:trx"]
```

<span data-ttu-id="7b38f-197">Un développeur peut utiliser ce fichier Docker pour exécuter les tests localement :</span><span class="sxs-lookup"><span data-stu-id="7b38f-197">A developer can use this Docker file to run the tests locally:</span></span>

```bash
docker build . -t delivery-test:1 --target=testrunner
docker run -p 8080:8080 delivery-test:1
```

<span data-ttu-id="7b38f-198">Le pipeline de CI doit également exécuter les tests dans le cadre de l’étape de vérification de build.</span><span class="sxs-lookup"><span data-stu-id="7b38f-198">The CI pipeline should also run the tests as part of the build verification step.</span></span>

<span data-ttu-id="7b38f-199">Notez que ce fichier utilise le Docker `ENTRYPOINT` commande pour exécuter les tests, pas le Docker `RUN` commande.</span><span class="sxs-lookup"><span data-stu-id="7b38f-199">Note that this file uses the Docker `ENTRYPOINT` command to run the tests, not the Docker `RUN` command.</span></span>

- <span data-ttu-id="7b38f-200">Si vous utilisez le `RUN` l’exécution des tests chaque fois que vous générez l’image de la commande.</span><span class="sxs-lookup"><span data-stu-id="7b38f-200">If you use the `RUN` command, the tests run every time you build the image.</span></span> <span data-ttu-id="7b38f-201">À l’aide de `ENTRYPOINT`, les tests sont participer.</span><span class="sxs-lookup"><span data-stu-id="7b38f-201">By using `ENTRYPOINT`, the tests are opt-in.</span></span> <span data-ttu-id="7b38f-202">Elles s’exécutent uniquement lorsque vous ciblez le `testrunner` étape.</span><span class="sxs-lookup"><span data-stu-id="7b38f-202">They run only when you explicitly target the `testrunner` stage.</span></span>
- <span data-ttu-id="7b38f-203">Un test ayant échoué n’entraîne pas la Docker `build` Échec de commande.</span><span class="sxs-lookup"><span data-stu-id="7b38f-203">A failing test doesn't cause the Docker `build` command to fail.</span></span> <span data-ttu-id="7b38f-204">De cette façon, vous pouvez distinguer le conteneur de générer des échecs d’échecs de test.</span><span class="sxs-lookup"><span data-stu-id="7b38f-204">That way you can distinguish container build failures from test failures.</span></span>
- <span data-ttu-id="7b38f-205">Résultats des tests peuvent être enregistrés sur un volume monté.</span><span class="sxs-lookup"><span data-stu-id="7b38f-205">Test results can be saved to a mounted volume.</span></span>

### <a name="container-best-practices"></a><span data-ttu-id="7b38f-206">Meilleures pratiques de conteneur</span><span class="sxs-lookup"><span data-stu-id="7b38f-206">Container best practices</span></span>

<span data-ttu-id="7b38f-207">Voici certains autres meilleures pratiques à prendre en compte pour les conteneurs :</span><span class="sxs-lookup"><span data-stu-id="7b38f-207">Here are some other best practices to consider for containers:</span></span>

- <span data-ttu-id="7b38f-208">Définissez des conventions d’organisation pour les balises de conteneurs, la gestion des versions et des conventions de d’affectation de noms pour les ressources déployées sur le cluster (pod, services, etc.).</span><span class="sxs-lookup"><span data-stu-id="7b38f-208">Define organization-wide conventions for container tags, versioning, and naming conventions for resources deployed to the cluster (pods, services, and so on).</span></span> <span data-ttu-id="7b38f-209">Cela peut simplifier le diagnostic des problèmes de déploiement.</span><span class="sxs-lookup"><span data-stu-id="7b38f-209">That can make it easier to diagnose deployment issues.</span></span>

- <span data-ttu-id="7b38f-210">Durant le cycle de développement et de test, le processus d’intégration et de livraison continue génère de nombreuses images de conteneur.</span><span class="sxs-lookup"><span data-stu-id="7b38f-210">During the development and test cycle, the CI/CD process will build many container images.</span></span> <span data-ttu-id="7b38f-211">Seuls certains de ces images sont candidates à la publication, puis uniquement certains de ces candidats seront promues en production de version.</span><span class="sxs-lookup"><span data-stu-id="7b38f-211">Only some of those images are candidates for release, and then only some of those release candidates will get promoted to production.</span></span> <span data-ttu-id="7b38f-212">Disposer d’une stratégie claire du contrôle de version, afin que vous sachiez quelles images sont actuellement déployés en production et pouvant revenir à une version antérieure si nécessaire.</span><span class="sxs-lookup"><span data-stu-id="7b38f-212">Have a clear versioning strategy, so that you know which images are currently deployed to production, and can roll back to a previous version if necessary.</span></span>

- <span data-ttu-id="7b38f-213">Déployez toujours les balises de version d’un conteneur spécifique, pas `latest`.</span><span class="sxs-lookup"><span data-stu-id="7b38f-213">Always deploy specific container version tags, not `latest`.</span></span>

- <span data-ttu-id="7b38f-214">Utilisez [espaces de noms](/azure/container-registry/container-registry-best-practices#repository-namespaces) dans Azure Container Registry pour isoler les images qui sont approuvées pour la production à partir d’images qui sont toujours en cours de test.</span><span class="sxs-lookup"><span data-stu-id="7b38f-214">Use [namespaces](/azure/container-registry/container-registry-best-practices#repository-namespaces) in Azure Container Registry to isolate images that are approved for production from images that are still being tested.</span></span> <span data-ttu-id="7b38f-215">Ne déplacez pas une image dans l’espace de noms de production jusqu'à ce que vous êtes prêt à déployer en production.</span><span class="sxs-lookup"><span data-stu-id="7b38f-215">Don't move an image into the production namespace until you're ready to deploy it into production.</span></span> <span data-ttu-id="7b38f-216">Si vous combinez cette pratique à la gestion sémantique des versions des images de conteneurs, vous pouvez réduire les risques de déploiement accidentel d’une version non approuvée pour publication.</span><span class="sxs-lookup"><span data-stu-id="7b38f-216">If you combine this practice with semantic versioning of container images, it can reduce the chance of accidentally deploying a version that wasn't approved for release.</span></span>

- <span data-ttu-id="7b38f-217">Suivez le principe du moindre privilège en exécutant des conteneurs en tant qu’un utilisateur sans privilège.</span><span class="sxs-lookup"><span data-stu-id="7b38f-217">Follow the principle of least privilege by running containers as a nonprivileged user.</span></span> <span data-ttu-id="7b38f-218">Dans Kubernetes, vous pouvez créer une stratégie de sécurité pod qui empêche l’exécution en tant que conteneurs *racine*.</span><span class="sxs-lookup"><span data-stu-id="7b38f-218">In Kubernetes, you can create a pod security policy that prevents containers from running as *root*.</span></span> <span data-ttu-id="7b38f-219">Consultez [Pods empêcher de s’exécuter avec des privilèges Root](https://docs.bitnami.com/kubernetes/how-to/secure-kubernetes-cluster-psp/).</span><span class="sxs-lookup"><span data-stu-id="7b38f-219">See [Prevent Pods From Running With Root Privileges](https://docs.bitnami.com/kubernetes/how-to/secure-kubernetes-cluster-psp/).</span></span>

## <a name="helm-charts"></a><span data-ttu-id="7b38f-220">Graphiques Helm</span><span class="sxs-lookup"><span data-stu-id="7b38f-220">Helm charts</span></span>

<span data-ttu-id="7b38f-221">Envisagez d’utiliser Helm pour gérer la génération et le déploiement des services.</span><span class="sxs-lookup"><span data-stu-id="7b38f-221">Consider using Helm to manage building and deploying services.</span></span> <span data-ttu-id="7b38f-222">Voici quelques-unes des fonctionnalités de Helm qui aident à CI/CD :</span><span class="sxs-lookup"><span data-stu-id="7b38f-222">Here are some of the features of Helm that help with CI/CD:</span></span>

- <span data-ttu-id="7b38f-223">Souvent, un seul microservice est défini par plusieurs objets Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="7b38f-223">Often a single microservice is defined by multiple Kubernetes objects.</span></span> <span data-ttu-id="7b38f-224">Helm permet à ces objets être empaquetés dans un même graphique Helm.</span><span class="sxs-lookup"><span data-stu-id="7b38f-224">Helm allows these objects to be packaged into a single Helm chart.</span></span>
- <span data-ttu-id="7b38f-225">Un graphique peut être déployé avec une seule commande Helm, plutôt qu’une série de commandes kubectl.</span><span class="sxs-lookup"><span data-stu-id="7b38f-225">A chart can be deployed with a single Helm command, rather than a series of kubectl commands.</span></span>
- <span data-ttu-id="7b38f-226">Les graphiques sont gérées explicitement.</span><span class="sxs-lookup"><span data-stu-id="7b38f-226">Charts are explicitly versioned.</span></span> <span data-ttu-id="7b38f-227">Utilisez Helm pour publier une version, afficher les versions et revenir à une version précédente.</span><span class="sxs-lookup"><span data-stu-id="7b38f-227">Use Helm to release a version, view releases, and roll back to a previous version.</span></span> <span data-ttu-id="7b38f-228">Suivi des mises à jour et des révisions, à l’aide du contrôle de version sémantique, avec possibilité de restaurer une version précédente</span><span class="sxs-lookup"><span data-stu-id="7b38f-228">Tracking updates and revisions, using semantic versioning, along with the ability to roll back to a previous version.</span></span>
- <span data-ttu-id="7b38f-229">Graphiques helm les modèles permettent d’éviter la duplication des informations, telles que les étiquettes et les sélecteurs, dans plusieurs fichiers.</span><span class="sxs-lookup"><span data-stu-id="7b38f-229">Helm charts use templates to avoid duplicating information, such as labels and selectors, across many files.</span></span>
- <span data-ttu-id="7b38f-230">Helm peut gérer les dépendances entre les graphiques.</span><span class="sxs-lookup"><span data-stu-id="7b38f-230">Helm can manage dependencies between charts.</span></span>
- <span data-ttu-id="7b38f-231">Les graphiques peuvent être stockées dans un référentiel Helm, tels que Azure Container Registry et intégrés dans le pipeline de génération.</span><span class="sxs-lookup"><span data-stu-id="7b38f-231">Charts can be stored in a Helm repository, such as Azure Container Registry, and integrated into the build pipeline.</span></span>

<span data-ttu-id="7b38f-232">Pour plus d’informations sur l’utilisation de Container Registry comme dépôt Helm, consultez [Utiliser Azure Container Registry comme référentiel Helm pour les graphiques de votre application](/azure/container-registry/container-registry-helm-repos).</span><span class="sxs-lookup"><span data-stu-id="7b38f-232">For more information about using Container Registry as a Helm repository, see [Use Azure Container Registry as a Helm repository for your application charts](/azure/container-registry/container-registry-helm-repos).</span></span>

<span data-ttu-id="7b38f-233">Un seul microservice peut impliquer plusieurs fichiers de configuration Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="7b38f-233">A single microservice may involve multiple Kubernetes configuration files.</span></span> <span data-ttu-id="7b38f-234">La mise à jour un service peut signifier toucher tous ces fichiers pour mettre à jour les sélecteurs, les étiquettes et les balises d’image.</span><span class="sxs-lookup"><span data-stu-id="7b38f-234">Updating a service can mean touching all of these files to update selectors, labels, and image tags.</span></span> <span data-ttu-id="7b38f-235">Helm les traite comme un package unique, appelé un graphique et vous permet de mettre à jour facilement les fichiers YAML à l’aide de variables.</span><span class="sxs-lookup"><span data-stu-id="7b38f-235">Helm treats these as a single package called a chart and allows you to easily update the YAML files by using variables.</span></span> <span data-ttu-id="7b38f-236">Helm utilise un langage de modèle (basé sur des modèles de Go) pour vous permettre d’écrire des fichiers de configuration YAML paramétrables.</span><span class="sxs-lookup"><span data-stu-id="7b38f-236">Helm uses a template language (based on Go templates) to let you write parameterized YAML configuration files.</span></span>

<span data-ttu-id="7b38f-237">Par exemple, voici la partie d’un fichier YAML qui définit un déploiement :</span><span class="sxs-lookup"><span data-stu-id="7b38f-237">For example, here's part of a YAML file that defines a deployment:</span></span>

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

<span data-ttu-id="7b38f-238">&#11162;Consultez le [fichier source](https://github.com/mspnp/microservices-reference-implementation/blob/master/charts/package/templates/package-deploy.yaml).</span><span class="sxs-lookup"><span data-stu-id="7b38f-238">&#11162; See the [source file](https://github.com/mspnp/microservices-reference-implementation/blob/master/charts/package/templates/package-deploy.yaml).</span></span>

<span data-ttu-id="7b38f-239">Vous pouvez voir que le nom du déploiement, les étiquettes et conteneur de spécification de tous les paramètres de modèle d’utilisation, qui sont fournies au moment du déploiement.</span><span class="sxs-lookup"><span data-stu-id="7b38f-239">You can see that the deployment name, labels, and container spec all use template parameters, which are provided at deployment time.</span></span> <span data-ttu-id="7b38f-240">Par exemple, à partir de la ligne de commande :</span><span class="sxs-lookup"><span data-stu-id="7b38f-240">For example, from the command line:</span></span>

```bash
helm install $HELM_CHARTS/package/ \
     --set image.tag=0.1.0 \
     --set image.repository=package \
     --set dockerregistry=$ACR_SERVER \
     --namespace backend \
     --name package-v0.1.0
```

<span data-ttu-id="7b38f-241">Bien que votre pipeline CI/CD peut installer un graphique directement dans Kubernetes, nous vous recommandons de création d’une archive de graphique (fichier .tgz) et en plaçant le graphique depuis un référentiel Helm tels que Azure Container Registry.</span><span class="sxs-lookup"><span data-stu-id="7b38f-241">Although your CI/CD pipeline could install a chart directly to Kubernetes, we recommend creating a chart archive (.tgz file) and pushing the chart to a Helm repository such as Azure Container Registry.</span></span> <span data-ttu-id="7b38f-242">Pour plus d’informations, consultez [les applications basées sur le Docker de Package dans les graphiques Helm dans les Pipelines Azure](/azure/devops/pipelines/languages/helm?view=azure-devops).</span><span class="sxs-lookup"><span data-stu-id="7b38f-242">For more information, see [Package Docker-based apps in Helm charts in Azure Pipelines](/azure/devops/pipelines/languages/helm?view=azure-devops).</span></span>

<span data-ttu-id="7b38f-243">Envisagez de déploiement Helm sur son propre espace de noms et à l’aide du contrôle d’accès en fonction du rôle (RBAC) pour restreindre le pouvoir déployer vers les espaces de noms.</span><span class="sxs-lookup"><span data-stu-id="7b38f-243">Consider deploying Helm to its own namespace and using role-based access control (RBAC) to restrict which namespaces it can deploy to.</span></span> <span data-ttu-id="7b38f-244">Pour plus d’informations, consultez [contrôle d’accès en fonction du rôle](https://helm.sh/docs/using_helm/#helm-and-role-based-access-control) dans la documentation de Helm.</span><span class="sxs-lookup"><span data-stu-id="7b38f-244">For more information, see [Role-based Access Control](https://helm.sh/docs/using_helm/#helm-and-role-based-access-control) in the Helm documentation.</span></span>

### <a name="revisions"></a><span data-ttu-id="7b38f-245">Révisions</span><span class="sxs-lookup"><span data-stu-id="7b38f-245">Revisions</span></span>

<span data-ttu-id="7b38f-246">Graphiques helm ont toujours un numéro de version, qui doit utiliser [semver](https://semver.org/).</span><span class="sxs-lookup"><span data-stu-id="7b38f-246">Helm charts always have a version number, which must use [semantic versioning](https://semver.org/).</span></span> <span data-ttu-id="7b38f-247">Un graphique peut également avoir un `appVersion`.</span><span class="sxs-lookup"><span data-stu-id="7b38f-247">A chart can also have an `appVersion`.</span></span> <span data-ttu-id="7b38f-248">Ce champ est facultatif et ne doit pas être liées à la version de graphique.</span><span class="sxs-lookup"><span data-stu-id="7b38f-248">This field is optional, and doesn't have to be related to the chart version.</span></span> <span data-ttu-id="7b38f-249">Certaines équipes souhaiterez versions d’application séparément à partir de mises à jour pour les graphiques.</span><span class="sxs-lookup"><span data-stu-id="7b38f-249">Some teams might want to application versions separately from updates to the charts.</span></span> <span data-ttu-id="7b38f-250">Mais une approche plus simple consiste à utiliser un numéro de version, donc il existe une relation 1:1 entre la version de graphique et de la version de l’application.</span><span class="sxs-lookup"><span data-stu-id="7b38f-250">But a simpler approach is to use one version number, so there's a 1:1 relation between chart version and application version.</span></span> <span data-ttu-id="7b38f-251">De cette façon, vous pouvez stocker un graphique par version et déployer facilement la version souhaitée :</span><span class="sxs-lookup"><span data-stu-id="7b38f-251">That way, you can store one chart per release and easily deploy the desired release:</span></span>

```bash
helm install <package-chart-name> --version <desiredVersion>
```

<span data-ttu-id="7b38f-252">Une autre bonne pratique consiste à fournir une annotation de la cause de modification dans le modèle de déploiement :</span><span class="sxs-lookup"><span data-stu-id="7b38f-252">Another good practice is to provide a change-cause annotation in the deployment template:</span></span>

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

<span data-ttu-id="7b38f-253">Cela vous permet d’afficher le champ de la cause de modification pour chaque révision, à l’aide de la `kubectl rollout history` commande.</span><span class="sxs-lookup"><span data-stu-id="7b38f-253">This lets you view the change-cause field for each revision, using the `kubectl rollout history` command.</span></span> <span data-ttu-id="7b38f-254">Dans l’exemple précédent, la cause de modification est fournie comme paramètre graphique Helm.</span><span class="sxs-lookup"><span data-stu-id="7b38f-254">In the previous example, the change-cause is provided as a Helm chart parameter.</span></span>

```bash
$ kubectl rollout history deployments/delivery-v010 -n backend
deployment.extensions/delivery-v010
REVISION  CHANGE-CAUSE
1         Initial deployment
```

<span data-ttu-id="7b38f-255">Vous pouvez également utiliser le `helm list` commande pour afficher l’historique de révision :</span><span class="sxs-lookup"><span data-stu-id="7b38f-255">You can also use the `helm list` command to view the revision history:</span></span>

```bash
~$ helm list
NAME            REVISION    UPDATED                     STATUS        CHART            APP VERSION     NAMESPACE
delivery-v0.1.0 1           Sun Apr  7 00:25:30 2019    DEPLOYED      delivery-v0.1.0  v0.1.0          backend
```

> [!TIP]
> <span data-ttu-id="7b38f-256">Utilisez le `--history-max` indicateur lors de l’initialisation de Helm.</span><span class="sxs-lookup"><span data-stu-id="7b38f-256">Use the `--history-max` flag when initializing Helm.</span></span> <span data-ttu-id="7b38f-257">Ce paramètre limite le nombre de révisions Tiller enregistre dans son historique.</span><span class="sxs-lookup"><span data-stu-id="7b38f-257">This setting limits the number of revisions that Tiller saves in its history.</span></span> <span data-ttu-id="7b38f-258">Tiller stocke l’historique de révision dans configmaps.</span><span class="sxs-lookup"><span data-stu-id="7b38f-258">Tiller stores revision history in configmaps.</span></span> <span data-ttu-id="7b38f-259">Si vous sortez souvent mises à jour, le configmaps peut devenir très volumineux, sauf si vous limitez la taille de l’historique.</span><span class="sxs-lookup"><span data-stu-id="7b38f-259">If you're releasing updates frequently, the configmaps can grow very large unless you limit the history size.</span></span>

## <a name="azure-devops-pipeline"></a><span data-ttu-id="7b38f-260">Pipeline de DevOps Azure</span><span class="sxs-lookup"><span data-stu-id="7b38f-260">Azure DevOps Pipeline</span></span>

<span data-ttu-id="7b38f-261">Dans les Pipelines d’Azure, les pipelines sont divisés en *générer des pipelines* et *mise en production de pipelines*.</span><span class="sxs-lookup"><span data-stu-id="7b38f-261">In Azure Pipelines, pipelines are divided into *build pipelines* and *release pipelines*.</span></span> <span data-ttu-id="7b38f-262">Le pipeline de build s’exécute le processus d’intégration continue et crée les artefacts de build.</span><span class="sxs-lookup"><span data-stu-id="7b38f-262">The build pipeline runs the CI process and creates build artifacts.</span></span> <span data-ttu-id="7b38f-263">Pour une architecture de microservices sur Kubernetes, ces artefacts sont les images de conteneur et les graphiques Helm qui définissent chaque microservice.</span><span class="sxs-lookup"><span data-stu-id="7b38f-263">For a microservices architecture on Kubernetes, these artifacts are the container images and Helm charts that define each microservice.</span></span> <span data-ttu-id="7b38f-264">Le pipeline de mise en production exécute ce processus CD qui déploie un microservice dans un cluster.</span><span class="sxs-lookup"><span data-stu-id="7b38f-264">The release pipeline runs that CD process that deploys a microservice into a cluster.</span></span>

<span data-ttu-id="7b38f-265">Selon le flux CI décrit plus haut dans cet article, un pipeline de build peut être constitué des tâches suivantes :</span><span class="sxs-lookup"><span data-stu-id="7b38f-265">Based on the CI flow described earlier in this article, a build pipeline might consist of the following tasks:</span></span>

1. <span data-ttu-id="7b38f-266">Créer le conteneur d’exécuteur de test.</span><span class="sxs-lookup"><span data-stu-id="7b38f-266">Build the test runner container.</span></span>

    ```yaml
    - task: Docker@1
      inputs:
        azureSubscriptionEndpoint: $(AzureSubscription)
        azureContainerRegistry: $(AzureContainerRegistry)
        arguments: '--pull --target testrunner'
        dockerFile: $(System.DefaultWorkingDirectory)/$(dockerFileName)
        imageName: '$(imageName)-test'
    ```

1. <span data-ttu-id="7b38f-267">Exécuter les tests, en appelant docker à exécuter sur le conteneur d’exécuteur de test.</span><span class="sxs-lookup"><span data-stu-id="7b38f-267">Run the tests, by invoking docker run against the test runner container.</span></span>

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

1. <span data-ttu-id="7b38f-268">Publier les résultats des tests.</span><span class="sxs-lookup"><span data-stu-id="7b38f-268">Publish the test results.</span></span> <span data-ttu-id="7b38f-269">Consultez [intégrer build et tâches de test](/azure/devops/pipelines/languages/docker?view=azure-devops&tabs=yaml#integrate-build-and-test-tasks).</span><span class="sxs-lookup"><span data-stu-id="7b38f-269">See [Integrate build and test tasks](/azure/devops/pipelines/languages/docker?view=azure-devops&tabs=yaml#integrate-build-and-test-tasks).</span></span>

    ```yaml
    - task: PublishTestResults@2
      inputs:
        testResultsFormat: 'VSTest'
        testResultsFiles: 'TestResults/*.trx'
        searchFolder: '$(System.DefaultWorkingDirectory)'
        publishRunAttachments: true
    ```

1. <span data-ttu-id="7b38f-270">Créer le conteneur de runtime.</span><span class="sxs-lookup"><span data-stu-id="7b38f-270">Build the runtime container.</span></span>

    ```yaml
    - task: Docker@1
      inputs:
        azureSubscriptionEndpoint: $(AzureSubscription)
        azureContainerRegistry: $(AzureContainerRegistry)
        dockerFile: $(System.DefaultWorkingDirectory)/$(dockerFileName)
        includeLatestTag: false
        imageName: '$(imageName)'
    ```

1. <span data-ttu-id="7b38f-271">Push vers le conteneur vers Azure Container Registry (ou autres container registry).</span><span class="sxs-lookup"><span data-stu-id="7b38f-271">Push to the container to Azure Container Registry (or other container registry).</span></span>

    ```yaml
    - task: Docker@1
      inputs:
        azureSubscriptionEndpoint: $(AzureSubscription)
        azureContainerRegistry: $(AzureContainerRegistry)
        command: 'Push an image'
        imageName: '$(imageName)'
        includeSourceTags: false
    ```

1. <span data-ttu-id="7b38f-272">Le graphique Helm du package.</span><span class="sxs-lookup"><span data-stu-id="7b38f-272">Package the Helm chart.</span></span>

    ```yaml
    - task: HelmDeploy@0
      inputs:
        command: package
        chartPath: $(chartPath)
        chartVersion: $(Build.SourceBranchName)
        arguments: '--app-version $(Build.SourceBranchName)'
    ```

1. <span data-ttu-id="7b38f-273">Envoyer le package Helm à Azure Container Registry (ou un autre référentiel Helm).</span><span class="sxs-lookup"><span data-stu-id="7b38f-273">Push the Helm package to Azure Container Registry (or other Helm repository).</span></span>

    ```yaml
    task: AzureCLI@1
      inputs:
        azureSubscription: $(AzureSubscription)
        scriptLocation: inlineScript
        inlineScript: |
        az acr helm push $(System.ArtifactsDirectory)/$(repositoryName)-$(Build.SourceBranchName).tgz --name $(AzureContainerRegistry);
    ```

<span data-ttu-id="7b38f-274">&#11162;Consultez le [fichier source](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/delivery/azure-pipelines-ci.yml).</span><span class="sxs-lookup"><span data-stu-id="7b38f-274">&#11162; See the [source file](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/delivery/azure-pipelines-ci.yml).</span></span>

<span data-ttu-id="7b38f-275">La sortie du pipeline CI est une image de conteneur de prêt pour la production et un graphique Helm mis à jour pour le microservice.</span><span class="sxs-lookup"><span data-stu-id="7b38f-275">The output from the CI pipeline is a production-ready container image and an updated Helm chart for the microservice.</span></span> <span data-ttu-id="7b38f-276">À ce stade, le pipeline de version peut reprendre.</span><span class="sxs-lookup"><span data-stu-id="7b38f-276">At this point, the release pipeline can take over.</span></span> <span data-ttu-id="7b38f-277">Il effectue les étapes suivantes :</span><span class="sxs-lookup"><span data-stu-id="7b38f-277">It performs the following steps:</span></span>

- <span data-ttu-id="7b38f-278">Déployer des environnements QA/dev/intermédiaires.</span><span class="sxs-lookup"><span data-stu-id="7b38f-278">Deploy to dev/QA/staging environments.</span></span>
- <span data-ttu-id="7b38f-279">Attendez un approbateur approuver ou rejeter le déploiement.</span><span class="sxs-lookup"><span data-stu-id="7b38f-279">Wait for an approver to approve or reject the deployment.</span></span>
- <span data-ttu-id="7b38f-280">Modifier le balisage d’image de conteneur pour la mise en production</span><span class="sxs-lookup"><span data-stu-id="7b38f-280">Retag the container image for release</span></span>
- <span data-ttu-id="7b38f-281">Envoyer la balise de version dans le Registre de conteneur.</span><span class="sxs-lookup"><span data-stu-id="7b38f-281">Push the release tag to the container registry.</span></span>
- <span data-ttu-id="7b38f-282">Mettre à niveau le graphique Helm dans le cluster de production.</span><span class="sxs-lookup"><span data-stu-id="7b38f-282">Upgrade the Helm chart in the production cluster.</span></span>

<span data-ttu-id="7b38f-283">Pour plus d’informations sur la création d’un pipeline de versions, consultez [pipelines de lancement, versions de brouillon et options de version](/azure/devops/pipelines/release/?view=azure-devops).</span><span class="sxs-lookup"><span data-stu-id="7b38f-283">For more information about creating a release pipeline, see [Release pipelines, draft releases, and release options](/azure/devops/pipelines/release/?view=azure-devops).</span></span>

<span data-ttu-id="7b38f-284">Le diagramme suivant illustre le processus de CI/CD de bout en bout décrit dans cet article :</span><span class="sxs-lookup"><span data-stu-id="7b38f-284">The following diagram shows the end-to-end CI/CD process described in this article:</span></span>

![Pipeline de CD/CD](./images/aks-cicd-flow.png)

## <a name="next-steps"></a><span data-ttu-id="7b38f-286">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="7b38f-286">Next steps</span></span>

<span data-ttu-id="7b38f-287">Cet article a été basé sur une implémentation de référence que vous trouverez sur [GitHub][ri].</span><span class="sxs-lookup"><span data-stu-id="7b38f-287">This article was based on a reference implementation that you can find on [GitHub][ri].</span></span>

[ri]: https://github.com/mspnp/microservices-reference-implementation