---
title: Concevoir un pipeline CI/CD à l’aide d’Azure DevOps
titleSuffix: Azure Example Scenarios
description: Générez et publiez une application .NET sur Azure Web Apps à l’aide d’Azure DevOps.
author: christianreddington
ms.date: 12/06/2018
ms.custom:
- fasttrack
- seodec18
ms.openlocfilehash: ae2dddd7567c6b69f936b3b9c9339313389e3bf6
ms.sourcegitcommit: bb7fcffbb41e2c26a26f8781df32825eb60df70c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/20/2018
ms.locfileid: "53643796"
---
# <a name="design-a-cicd-pipeline-using-azure-devops"></a><span data-ttu-id="31e0c-103">Concevoir un pipeline CI/CD à l’aide d’Azure DevOps</span><span class="sxs-lookup"><span data-stu-id="31e0c-103">Design a CI/CD pipeline using Azure DevOps</span></span>

<span data-ttu-id="31e0c-104">Ce scénario fournit des conseils d’architecture et de conception pour la création d’un pipeline d’intégration continue (CI) et de déploiement continu (CD).</span><span class="sxs-lookup"><span data-stu-id="31e0c-104">This scenario provides architecture and design guidance for building a continuous integration (CI) and continuous deployment (CD) pipeline.</span></span> <span data-ttu-id="31e0c-105">Dans cet exemple, le pipeline CI/CD déploie une application web .NET à deux niveaux sur Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="31e0c-105">In this example, the CI/CD pipeline deploys a two-tier .NET web application to the Azure App Service.</span></span>

<span data-ttu-id="31e0c-106">La migration vers des processus CI/CD modernes offre de nombreux avantages pour la génération, le déploiement, le test et la supervision de l’application.</span><span class="sxs-lookup"><span data-stu-id="31e0c-106">Migrating to modern CI/CD processes provides many benefits for application builds, deployments, testing, and monitoring.</span></span> <span data-ttu-id="31e0c-107">En utilisant Azure DevOps avec d’autres services comme App Service, les organisations peuvent se concentrer sur le développement de leurs applications, plutôt que sur la gestion de l’infrastructure sous-jacente.</span><span class="sxs-lookup"><span data-stu-id="31e0c-107">By utilizing Azure DevOps along with other services such as App Service, organizations can focus on the development of their apps rather than the management of the supporting infrastructure.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="31e0c-108">Cas d’usage appropriés</span><span class="sxs-lookup"><span data-stu-id="31e0c-108">Relevant use cases</span></span>

<span data-ttu-id="31e0c-109">Les processus Azure DevOps et CI/CD peuvent être envisagés pour ce qui suit :</span><span class="sxs-lookup"><span data-stu-id="31e0c-109">Consider Azure DevOps and CI/CD processes for:</span></span>

- <span data-ttu-id="31e0c-110">Accélération du développement d’applications et des cycles de vie de développement</span><span class="sxs-lookup"><span data-stu-id="31e0c-110">Accelerating application development and development life cycles</span></span>
- <span data-ttu-id="31e0c-111">Qualité et cohérence de la construction dans un processus de compilation et de mise en production automatisé</span><span class="sxs-lookup"><span data-stu-id="31e0c-111">Building quality and consistency into an automated build and release process</span></span>
- <span data-ttu-id="31e0c-112">Amélioration de la stabilité et du temps d’activité des applications</span><span class="sxs-lookup"><span data-stu-id="31e0c-112">Increasing application stability and uptime</span></span>

## <a name="architecture"></a><span data-ttu-id="31e0c-113">Architecture</span><span class="sxs-lookup"><span data-stu-id="31e0c-113">Architecture</span></span>

![Diagramme de l’architecture des composants Azure impliqués dans un scénario DevOps utilisant Azure DevOps et Azure App Service][architecture]

<span data-ttu-id="31e0c-115">Les données circulent dans le scénario comme suit :</span><span class="sxs-lookup"><span data-stu-id="31e0c-115">The data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="31e0c-116">Un développeur modifie le code source de l’application.</span><span class="sxs-lookup"><span data-stu-id="31e0c-116">A developer changes application source code.</span></span>
2. <span data-ttu-id="31e0c-117">Le code de l’application, y compris le fichier web.config, est validé dans le dépôt de code source Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="31e0c-117">Application code including the web.config file is committed to the source code repository in Azure Repos.</span></span>
3. <span data-ttu-id="31e0c-118">L’intégration continue déclenche la génération de l’application et des tests unitaires avec Azure Test Plans.</span><span class="sxs-lookup"><span data-stu-id="31e0c-118">Continuous integration triggers application build and unit tests using Azure Test Plans.</span></span>
4. <span data-ttu-id="31e0c-119">Dans Azure Pipelines, le déploiement continu déclenche un déploiement automatisé d’artefacts d’application *avec des valeurs de configuration propres à l’environnement*.</span><span class="sxs-lookup"><span data-stu-id="31e0c-119">Continuous deployment within Azure Pipelines triggers an automated deployment of application artifacts *with environment-specific configuration values*.</span></span>
5. <span data-ttu-id="31e0c-120">Les artefacts sont déployés sur Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="31e0c-120">The artifacts are deployed to Azure App Service.</span></span>
6. <span data-ttu-id="31e0c-121">Azure Application Insights collecte et analyse les données relatives à l’intégrité, aux performances et à l’utilisation.</span><span class="sxs-lookup"><span data-stu-id="31e0c-121">Azure Application Insights collects and analyzes health, performance, and usage data.</span></span>
7. <span data-ttu-id="31e0c-122">Les développeurs supervisent et gèrent les informations relatives à l’intégrité, aux performances et à l’utilisation.</span><span class="sxs-lookup"><span data-stu-id="31e0c-122">Developers monitor and mange health, performance, and usage information.</span></span>
8. <span data-ttu-id="31e0c-123">Les informations du backlog sont utilisées pour classer par priorité les nouvelles fonctionnalités et les correctifs de bogues avec Azure Boards.</span><span class="sxs-lookup"><span data-stu-id="31e0c-123">Backlog information is used to prioritize new features and bug fixes using Azure Boards.</span></span>

### <a name="components"></a><span data-ttu-id="31e0c-124">Composants</span><span class="sxs-lookup"><span data-stu-id="31e0c-124">Components</span></span>

- <span data-ttu-id="31e0c-125">[Azure DevOps][vsts] est un service qui vous permet de gérer votre cycle de vie de développement de bout en bout, depuis la planification et la gestion du projet jusqu’à la compilation et la mise en production, en passant par la gestion du code.</span><span class="sxs-lookup"><span data-stu-id="31e0c-125">[Azure DevOps][vsts] is a service for managing your development life cycle end-to-end &mdash; from planning and project management, to code management, and continuing to build and release.</span></span>

- <span data-ttu-id="31e0c-126">[Azure Web Apps][web-apps] est un service PaaS pour l’hébergement d’applications web, d’API REST et de back ends mobiles.</span><span class="sxs-lookup"><span data-stu-id="31e0c-126">[Azure Web Apps][web-apps] is a PaaS service for hosting web applications, REST APIs, and mobile back ends.</span></span> <span data-ttu-id="31e0c-127">Bien que cet article se concentre sur .NET, il existe plusieurs options de plateforme de développement supplémentaires prises en charge.</span><span class="sxs-lookup"><span data-stu-id="31e0c-127">While this article focuses on .NET, there are several additional development platform options supported.</span></span>

- <span data-ttu-id="31e0c-128">[Application Insights][application-insights] est un service interne extensible de gestion des performances des applications (APM) destiné aux développeurs web sur de multiples plateformes.</span><span class="sxs-lookup"><span data-stu-id="31e0c-128">[Application Insights][application-insights] is a first-party, extensible Application Performance Management (APM) service for web developers on multiple platforms.</span></span>

### <a name="alternatives"></a><span data-ttu-id="31e0c-129">Autres solutions</span><span class="sxs-lookup"><span data-stu-id="31e0c-129">Alternatives</span></span>

<span data-ttu-id="31e0c-130">Bien que cet article se concentre sur Azure DevOps, [Azure DevOps Server][azure-devops-server] (autrefois appelé Team Foundation Server) peut faire office de substitut local.</span><span class="sxs-lookup"><span data-stu-id="31e0c-130">While this article focuses on Azure DevOps, [Azure DevOps Server][azure-devops-server] (previously known as Team Foundation Server) could be used as an on-premises substitute.</span></span> <span data-ttu-id="31e0c-131">Vous pouvez également utiliser un ensemble de technologies pour un pipeline de développement open source à l’aide de [Jenkins][jenkins-on-azure].</span><span class="sxs-lookup"><span data-stu-id="31e0c-131">Alternatively, you could also use a set of technologies for an open-source development pipeline using [Jenkins][jenkins-on-azure].</span></span>

<span data-ttu-id="31e0c-132">Dans une perspective IaC (Infrastructure as Code), les [modèles Resource Manager][arm-templates] sont inclus dans le projet Azure DevOps, mais vous pouvez également utiliser d’autres technologies comme [Terraform][terraform] ou [Chef][chef].</span><span class="sxs-lookup"><span data-stu-id="31e0c-132">From an infrastructure-as-code perspective, [Resource Manager templates][arm-templates] were used as part of the Azure DevOps project, but you could consider other management technologies such as [Terraform][terraform] or [Chef][chef].</span></span> <span data-ttu-id="31e0c-133">Si vous préférez un déploiement IaaS (infrastructure as a service) et que vous avez besoin d’une gestion de la configuration, tournez-vous vers [Azure Automation State Configuration][desired-state-configuration], [Ansible][ansible] ou [Chef][chef].</span><span class="sxs-lookup"><span data-stu-id="31e0c-133">If you prefer an infrastructure-as-a-service (IaaS)-based deployment and require configuration management, you could consider either [Azure Automation State Configuration][desired-state-configuration], [Ansible][ansible], or [Chef][chef].</span></span>

<span data-ttu-id="31e0c-134">Examinez ci-après les alternatives possibles à l’hébergement dans Azure Web Apps :</span><span class="sxs-lookup"><span data-stu-id="31e0c-134">You could consider these alternatives to hosting in Azure Web Apps:</span></span>

- <span data-ttu-id="31e0c-135">Les [Machines virtuelles Azure][compare-vm-hosting] permettent de gérer les charges de travail qui exigent un degré de contrôle élevé ou qui dépendent de services et de composants du système d’exploitation non utilisables avec Web Apps (par exemple, GAC Windows ou COM).</span><span class="sxs-lookup"><span data-stu-id="31e0c-135">[Azure Virtual Machines][compare-vm-hosting] handles workloads that require a high degree of control, or depend on OS components and services that are not possible with Web Apps (for example, the Windows GAC, or COM).</span></span>

- <span data-ttu-id="31e0c-136">[Service Fabric][service-fabric] est une option appropriée si l’architecture de la charge de travail s’articule autour de composants distribués qui tirent avantage d’un déploiement et d’une exécution sur un cluster avec un haut degré de contrôle.</span><span class="sxs-lookup"><span data-stu-id="31e0c-136">[Service Fabric][service-fabric] is a good option if the workload architecture is focused around distributed components that benefit from being deployed and run across a cluster with a high degree of control.</span></span> <span data-ttu-id="31e0c-137">Service Fabric peut également être utilisé pour héberger des conteneurs.</span><span class="sxs-lookup"><span data-stu-id="31e0c-137">Service Fabric can also be used to host containers.</span></span>

- <span data-ttu-id="31e0c-138">[Azure Functions][azure-functions] fournit une approche serverless efficace si l’architecture de la charge de travail s’articule autour de composants distribués précis, nécessitant des dépendances minimales, où les différents composants doivent uniquement être exécutés à la demande (et non de façon continue) et où l’orchestration des composants n’est pas obligatoire.</span><span class="sxs-lookup"><span data-stu-id="31e0c-138">[Azure Functions][azure-functions] provides an effective serverless approach if the workload architecture is centered around fine grained distributed components, requiring minimal dependencies, where individual components are only required to run on demand (not continuously) and orchestration of components is not required.</span></span>

<span data-ttu-id="31e0c-139">Cet [arbre de décision pour les services de calcul Azure](/azure/architecture/guide/technology-choices/compute-decision-tree) peut vous aider dans vos choix de migration.</span><span class="sxs-lookup"><span data-stu-id="31e0c-139">This [decision tree for Azure compute services](/azure/architecture/guide/technology-choices/compute-decision-tree) may help when choosing the right path to take for a migration.</span></span>

## <a name="management-and-security-considerations"></a><span data-ttu-id="31e0c-140">Considérations relatives à la gestion et à la sécurité</span><span class="sxs-lookup"><span data-stu-id="31e0c-140">Management and Security Considerations</span></span>

- <span data-ttu-id="31e0c-141">Envisagez de tirer profit de l’une des [tâches de création de jetons][vsts-tokenization] disponibles sur VSTS marketplace.</span><span class="sxs-lookup"><span data-stu-id="31e0c-141">Consider leveraging one of the [tokenization tasks][vsts-tokenization] available in the VSTS marketplace.</span></span>

- <span data-ttu-id="31e0c-142">Les tâches [Azure Key Vault][download-keyvault-secrets] peuvent télécharger des secrets à partir d’un coffre de clés Azure dans votre production.</span><span class="sxs-lookup"><span data-stu-id="31e0c-142">[Azure Key Vault][download-keyvault-secrets] tasks can download secrets from an Azure Key Vault into your release.</span></span> <span data-ttu-id="31e0c-143">Vous pouvez ensuite utiliser ces secrets en tant que variables dans votre définition de mise en production, ce qui vous évite d’avoir à les stocker dans le contrôle de code source.</span><span class="sxs-lookup"><span data-stu-id="31e0c-143">You can then use those secrets as variables in your release definition, which avoids storing them in source control.</span></span>

- <span data-ttu-id="31e0c-144">Utilisez des [variables de mise en production][vsts-release-variables] dans vos définitions de mise en production pour piloter les changements de configuration de vos environnements.</span><span class="sxs-lookup"><span data-stu-id="31e0c-144">Use [release variables][vsts-release-variables] in your release definitions to drive configuration changes of your environments.</span></span> <span data-ttu-id="31e0c-145">Les variables de mise en production peuvent être étendues à toute une version ou à un environnement donné.</span><span class="sxs-lookup"><span data-stu-id="31e0c-145">Release variables can be scoped to an entire release or a given environment.</span></span> <span data-ttu-id="31e0c-146">Si vous utilisez des variables pour les informations de secret, veillez à sélectionner l’icône en forme de cadenas.</span><span class="sxs-lookup"><span data-stu-id="31e0c-146">When using variables for secret information, ensure that you select the padlock icon.</span></span>

- <span data-ttu-id="31e0c-147">Les [portes de déploiement][vsts-deployment-gates] doivent être utilisées dans votre pipeline de mise en production.</span><span class="sxs-lookup"><span data-stu-id="31e0c-147">[Deployment gates][vsts-deployment-gates] should be used in your release pipeline.</span></span> <span data-ttu-id="31e0c-148">Vous pourrez ainsi tirer profit des données de supervision en les combinant avec des systèmes externes (par exemple, systèmes de gestion des incidents ou d’autres systèmes sur mesure) pour déterminer s’il faut promouvoir une mise en production.</span><span class="sxs-lookup"><span data-stu-id="31e0c-148">This lets you leverage monitoring data in association with external systems (for example, incident management or additional bespoke systems) to determine whether a release should be promoted.</span></span>

- <span data-ttu-id="31e0c-149">Lorsqu’une intervention manuelle dans un pipeline de mise en production est nécessaire, utilisez la fonctionnalité [approbations][vsts-approvals].</span><span class="sxs-lookup"><span data-stu-id="31e0c-149">Where manual intervention in a release pipeline is required, use the [approvals][vsts-approvals] functionality.</span></span>

- <span data-ttu-id="31e0c-150">Envisagez d’utiliser [Application Insights][application-insights] et des outils de supervision supplémentaires dès que possible dans votre pipeline de mise en production.</span><span class="sxs-lookup"><span data-stu-id="31e0c-150">Consider using [Application Insights][application-insights] and additional monitoring tools as early as possible in your release pipeline.</span></span> <span data-ttu-id="31e0c-151">De nombreuses organisations attendent d’être dans leur environnement de production pour commencer la supervision.</span><span class="sxs-lookup"><span data-stu-id="31e0c-151">Many organizations only begin monitoring in their production environment.</span></span> <span data-ttu-id="31e0c-152">La supervision de vos autres environnements vous permet d’identifier les éventuels bogues plus tôt dans le processus de développement et d’éviter ainsi l’apparition de problèmes dans votre environnement de production.</span><span class="sxs-lookup"><span data-stu-id="31e0c-152">By monitoring your other environments, you can identify bugs earlier in the development process and avoid issues in your production environment.</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="31e0c-153">Déployez le scénario</span><span class="sxs-lookup"><span data-stu-id="31e0c-153">Deploy the scenario</span></span>

### <a name="prerequisites"></a><span data-ttu-id="31e0c-154">Prérequis</span><span class="sxs-lookup"><span data-stu-id="31e0c-154">Prerequisites</span></span>

- <span data-ttu-id="31e0c-155">Vous devez disposer d’un compte Azure existant.</span><span class="sxs-lookup"><span data-stu-id="31e0c-155">You must have an existing Azure account.</span></span> <span data-ttu-id="31e0c-156">Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.</span><span class="sxs-lookup"><span data-stu-id="31e0c-156">If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.</span></span>

- <span data-ttu-id="31e0c-157">Vous devez inscrire une organisation Azure DevOps.</span><span class="sxs-lookup"><span data-stu-id="31e0c-157">You must sign up for an Azure DevOps organization.</span></span> <span data-ttu-id="31e0c-158">Pour plus d’informations, consultez [Démarrage rapide : Créer votre organisation][vsts-account-create].</span><span class="sxs-lookup"><span data-stu-id="31e0c-158">For more information, see [Quickstart: Create your organization][vsts-account-create].</span></span>

### <a name="walk-through"></a><span data-ttu-id="31e0c-159">Procédure pas à pas</span><span class="sxs-lookup"><span data-stu-id="31e0c-159">Walk-through</span></span>

<span data-ttu-id="31e0c-160">Le [projet Azure DevOps](/azure/devops-project/azure-devops-project-github) déploie automatiquement un plan App Service, une instance App Service ainsi qu’une ressource App Insights, et configure automatiquement le projet Azure DevOps.</span><span class="sxs-lookup"><span data-stu-id="31e0c-160">The [Azure DevOps project](/azure/devops-project/azure-devops-project-github) will deploy an App Service Plan, App Service, and an App Insights resource for you, as well as configure the Azure DevOps project for you.</span></span>

<span data-ttu-id="31e0c-161">Une fois le projet Azure DevOps déployé et la compilation effectuée, vérifiez les modifications du code, les éléments de travail et les résultats des tests associés.</span><span class="sxs-lookup"><span data-stu-id="31e0c-161">Once you've deployed the Azure DevOps project and the build is completed, review the associated code changes, work items, and test results.</span></span> <span data-ttu-id="31e0c-162">Vous remarquerez qu’aucun résultat de test n’est affiché, car le code ne contient aucun test à exécuter.</span><span class="sxs-lookup"><span data-stu-id="31e0c-162">You will notice that no test results are displayed, because the code does not contain any tests to run.</span></span>

<span data-ttu-id="31e0c-163">Le projet crée un pipeline de mise en production et un déclenchement de déploiement continu, ce qui déploie notre application dans l’environnement de développement.</span><span class="sxs-lookup"><span data-stu-id="31e0c-163">The project creates a release pipeline and continuous deployment trigger, deploying our application into the Dev environment.</span></span> <span data-ttu-id="31e0c-164">Dans le cadre d’un processus de déploiement continu, il est possible que les mises en production couvrent plusieurs environnements.</span><span class="sxs-lookup"><span data-stu-id="31e0c-164">As part of a continuous deployment process, you may see releases that span multiple environments.</span></span> <span data-ttu-id="31e0c-165">Une mise en production peut s’étendre sur chaque infrastructure (à l’aide de techniques telles qu’IaC) et peut également déployer les packages d’application requis en même temps que toutes les tâches de post-configuration.</span><span class="sxs-lookup"><span data-stu-id="31e0c-165">A release can span both infrastructure (using techniques such as infrastructure-as-code), and can also deploy the application packages required along with any post-configuration tasks.</span></span>

## <a name="pricing"></a><span data-ttu-id="31e0c-166">Tarifs</span><span class="sxs-lookup"><span data-stu-id="31e0c-166">Pricing</span></span>

<span data-ttu-id="31e0c-167">Les coûts d’Azure DevOps varient selon le nombre d’utilisateurs de votre organisation qui nécessitent un accès, ainsi qu’en fonction d’autres facteurs tels que le nombre de builds/mises en production simultanés requis et le nombre d’utilisateurs de test.</span><span class="sxs-lookup"><span data-stu-id="31e0c-167">Azure DevOps costs depend on the number of users in your organization that require access, along with other factors like the number of concurrent build/releases required and number of test users.</span></span> <span data-ttu-id="31e0c-168">Pour plus d’informations, consultez la page [Tarification Azure DevOps][vsts-pricing-page].</span><span class="sxs-lookup"><span data-stu-id="31e0c-168">For more information, see [Azure DevOps pricing][vsts-pricing-page].</span></span>

<span data-ttu-id="31e0c-169">La [calculatrice de prix][vsts-pricing-calculator] fournit une estimation pour l’exécution d’Azure DevOps avec 20 utilisateurs.</span><span class="sxs-lookup"><span data-stu-id="31e0c-169">This [pricing calculator][vsts-pricing-calculator] provides an estimate for running Azure DevOps with 20 users.</span></span>

<span data-ttu-id="31e0c-170">Azure DevOps est facturé par mois et par utilisateur.</span><span class="sxs-lookup"><span data-stu-id="31e0c-170">Azure DevOps is billed on a per-user per-month basis.</span></span> <span data-ttu-id="31e0c-171">Des frais supplémentaires peuvent s’appliquer en fonction des pipelines simultanés requis, ainsi que de l’ajout d’utilisateurs de test et de licences de base utilisateur.</span><span class="sxs-lookup"><span data-stu-id="31e0c-171">There may be additional charges dependent upon concurrent pipelines needed, in addition to any additional test users or user basic licenses.</span></span>

## <a name="related-resources"></a><span data-ttu-id="31e0c-172">Ressources associées</span><span class="sxs-lookup"><span data-stu-id="31e0c-172">Related resources</span></span>

<span data-ttu-id="31e0c-173">Pour plus d’informations sur les déploiements CI/CD et Azure DevOps, consultez les ressources suivantes :</span><span class="sxs-lookup"><span data-stu-id="31e0c-173">Review the following resources to learn more about CI/CD and Azure DevOps:</span></span>

- <span data-ttu-id="31e0c-174">[Qu’est DevOps ?][devops-whatis]</span><span class="sxs-lookup"><span data-stu-id="31e0c-174">[What is DevOps?][devops-whatis]</span></span>
- <span data-ttu-id="31e0c-175">[DevOps at Microsoft - How we work with Azure DevOps][devops-microsoft] (DevOps chez Microsoft - Comment nous travaillons avec Azure DevOps)</span><span class="sxs-lookup"><span data-stu-id="31e0c-175">[DevOps at Microsoft - How we work with Azure DevOps][devops-microsoft]</span></span>
- <span data-ttu-id="31e0c-176">[Tutoriels pas à pas : DevOps avec Azure DevOps][devops-with-vsts]</span><span class="sxs-lookup"><span data-stu-id="31e0c-176">[Step-by-step Tutorials: DevOps with Azure DevOps][devops-with-vsts]</span></span>
- <span data-ttu-id="31e0c-177">[Liste de contrôle DevOps][devops-checklist]</span><span class="sxs-lookup"><span data-stu-id="31e0c-177">[Devops Checklist][devops-checklist]</span></span>
- <span data-ttu-id="31e0c-178">[Créer un pipeline CI/CD pour .NET avec Azure DevOps Project][devops-project-create]</span><span class="sxs-lookup"><span data-stu-id="31e0c-178">[Create a CI/CD pipeline for .NET with the Azure DevOps project][devops-project-create]</span></span>

<!-- links -->

[ansible]: /azure/ansible/
[application-insights]: /azure/application-insights/app-insights-overview
[app-service-reference-architecture]: ../../reference-architectures/app-service-web-app/basic-web-app.md
[arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[architecture]: ./media/architecture-devops-dotnet-webapp.svg
[chef]: /azure/chef/
[design-patterns-availability]: /azure/architecture/patterns/category/availability
[design-patterns-resiliency]: /azure/architecture/patterns/category/resiliency
[design-patterns-scalability]: /azure/architecture/patterns/category/performance-scalability
[design-patterns-security]: /azure/architecture/patterns/category/security
[desired-state-configuration]: /azure/automation/automation-dsc-overview
[devops-microsoft]: /azure/devops/devops-at-microsoft/
[devops-with-vsts]: https://almvm.azurewebsites.net/labs/vsts/
[devops-checklist]: /azure/architecture/checklist/dev-ops
[application-insights]: https://azure.microsoft.com/services/application-insights/
[cloud-based-load-testing]: https://visualstudio.microsoft.com/team-services/cloud-load-testing/
[cloud-based-load-testing-on-premises]: /vsts/test/load-test/clt-with-private-machines?view=vsts
[jenkins-on-azure]: /azure/jenkins/
[devops-whatis]: /azure/devops/what-is-devops
[download-keyvault-secrets]: /vsts/pipelines/tasks/deploy/azure-key-vault?view=vsts
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[resiliency-app-service]: /azure/architecture/checklist/resiliency-per-service#app-service
[vsts]: /vsts/?view=vsts#pivot=services
[continuous-integration]: /azure/devops/what-is-continuous-integration
[continuous-delivery]: /azure/devops/what-is-continuous-delivery
[web-apps]: /azure/app-service/app-service-web-overview
[vsts-account-create]: /azure/devops/organizations/accounts/create-organization-msa-or-work-student?view=vsts
[vsts-approvals]: /vsts/pipelines/release/approvals/approvals?view=vsts
[devops-project]: https://portal.azure.com/?feature.customportal=false#create/Microsoft.AzureProject
[vsts-deployment-gates]: /vsts/pipelines/release/approvals/gates?view=vsts
[vsts-pricing-calculator]: https://azure.com/e/498aa024454445a8a352e75724f900b1
[vsts-pricing-page]: https://azure.microsoft.com/pricing/details/visual-studio-team-services/
[vsts-release-variables]: /vsts/pipelines/release/variables?view=vsts&tabs=batch
[vsts-tokenization]: https://marketplace.visualstudio.com/search?term=token&target=VSTS&category=All%20categories&sortBy=Relevance
[azure-key-vault]: /azure/key-vault/key-vault-overview
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[azure-devops-server]: https://visualstudio.microsoft.com/tfs/
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[service-fabric]: /azure/service-fabric/
[azure-functions]: /azure/azure-functions/
[azure-containers]: https://azure.microsoft.com/overview/containers/
[compare-vm-hosting]: /azure/app-service/choose-web-site-cloud-service-vm
[app-insights-cd-monitoring]: /azure/application-insights/app-insights-vsts-continuous-monitoring
[azure-region-pair-bcdr]: /azure/best-practices-availability-paired-regions
[devops-project-create]: /azure/devops-project/azure-devops-project-aspnet-core
[terraform]: /azure/terraform/
