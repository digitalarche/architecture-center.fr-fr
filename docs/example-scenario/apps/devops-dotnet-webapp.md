---
title: Concevoir un pipeline CI/CD à l’aide d’Azure DevOps
titleSuffix: Azure Example Scenarios
description: Générez et publiez une application .NET sur Azure Web Apps à l’aide d’Azure DevOps.
author: christianreddington
ms.date: 12/06/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: fasttrack, seodec18
social_image_url: /azure/architecture/example-scenario/apps/media/architecture-devops-dotnet-webapp.svg
ms.openlocfilehash: 0d6ac13fb357657a2ec5e6284abadb46d6926907
ms.sourcegitcommit: 3b15d65e7c35a19506e562c444343f8467b6a073
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/25/2019
ms.locfileid: "54908492"
---
# <a name="design-a-cicd-pipeline-using-azure-devops"></a>Concevoir un pipeline CI/CD à l’aide d’Azure DevOps

Ce scénario fournit des conseils d’architecture et de conception pour la création d’un pipeline d’intégration continue (CI) et de déploiement continu (CD). Dans cet exemple, le pipeline CI/CD déploie une application web .NET à deux niveaux sur Azure App Service.

La migration vers des processus CI/CD modernes offre de nombreux avantages pour la génération, le déploiement, le test et la supervision de l’application. En utilisant Azure DevOps avec d’autres services comme App Service, les organisations peuvent se concentrer sur le développement de leurs applications, plutôt que sur la gestion de l’infrastructure sous-jacente.

## <a name="relevant-use-cases"></a>Cas d’usage appropriés

Les processus Azure DevOps et CI/CD peuvent être envisagés pour ce qui suit :

- Accélération du développement d’applications et des cycles de vie de développement
- Qualité et cohérence de la construction dans un processus de compilation et de mise en production automatisé
- Amélioration de la stabilité et du temps d’activité des applications

## <a name="architecture"></a>Architecture

![Diagramme de l’architecture des composants Azure impliqués dans un scénario DevOps utilisant Azure DevOps et Azure App Service][architecture]

Les données circulent dans le scénario comme suit :

1. Un développeur modifie le code source de l’application.
2. Le code de l’application, y compris le fichier web.config, est validé dans le dépôt de code source Azure Repos.
3. L’intégration continue déclenche la génération de l’application et des tests unitaires avec Azure Test Plans.
4. Dans Azure Pipelines, le déploiement continu déclenche un déploiement automatisé d’artefacts d’application *avec des valeurs de configuration propres à l’environnement*.
5. Les artefacts sont déployés sur Azure App Service.
6. Azure Application Insights collecte et analyse les données relatives à l’intégrité, aux performances et à l’utilisation.
7. Les développeurs supervisent et gèrent les informations relatives à l’intégrité, aux performances et à l’utilisation.
8. Les informations du backlog sont utilisées pour classer par priorité les nouvelles fonctionnalités et les correctifs de bogues avec Azure Boards.

### <a name="components"></a>Composants

- [Azure DevOps][vsts] est un service qui vous permet de gérer votre cycle de vie de développement de bout en bout, depuis la planification et la gestion du projet jusqu’à la compilation et la mise en production, en passant par la gestion du code.

- [Azure Web Apps][web-apps] est un service PaaS pour l’hébergement d’applications web, d’API REST et de back ends mobiles. Bien que cet article se concentre sur .NET, il existe plusieurs options de plateforme de développement supplémentaires prises en charge.

- [Application Insights][application-insights] est un service interne extensible de gestion des performances des applications (APM) destiné aux développeurs web sur de multiples plateformes.

### <a name="alternatives"></a>Autres solutions

Bien que cet article se concentre sur Azure DevOps, [Azure DevOps Server][azure-devops-server] (autrefois appelé Team Foundation Server) peut faire office de substitut local. Vous pouvez également utiliser un ensemble de technologies pour un pipeline de développement open source à l’aide de [Jenkins][jenkins-on-azure].

Dans une perspective IaC (Infrastructure as Code), les [modèles Resource Manager][arm-templates] sont inclus dans le projet Azure DevOps, mais vous pouvez également utiliser d’autres technologies comme [Terraform][terraform] ou [Chef][chef]. Si vous préférez un déploiement IaaS (infrastructure as a service) et que vous avez besoin d’une gestion de la configuration, tournez-vous vers [Azure Automation State Configuration][desired-state-configuration], [Ansible][ansible] ou [Chef][chef].

Examinez ci-après les alternatives possibles à l’hébergement dans Azure Web Apps :

- Les [Machines virtuelles Azure][compare-vm-hosting] permettent de gérer les charges de travail qui exigent un degré de contrôle élevé ou qui dépendent de services et de composants du système d’exploitation non utilisables avec Web Apps (par exemple, GAC Windows ou COM).

- [Service Fabric][service-fabric] est une option appropriée si l’architecture de la charge de travail s’articule autour de composants distribués qui tirent avantage d’un déploiement et d’une exécution sur un cluster avec un haut degré de contrôle. Service Fabric peut également être utilisé pour héberger des conteneurs.

- [Azure Functions][azure-functions] fournit une approche serverless efficace si l’architecture de la charge de travail s’articule autour de composants distribués précis, nécessitant des dépendances minimales, où les différents composants doivent uniquement être exécutés à la demande (et non de façon continue) et où l’orchestration des composants n’est pas obligatoire.

Cet [arbre de décision pour les services de calcul Azure](/azure/architecture/guide/technology-choices/compute-decision-tree) peut vous aider dans vos choix de migration.

## <a name="management-and-security-considerations"></a>Considérations relatives à la gestion et à la sécurité

- Envisagez de tirer profit de l’une des [tâches de création de jetons][vsts-tokenization] disponibles sur VSTS marketplace.

- Les tâches [Azure Key Vault][download-keyvault-secrets] peuvent télécharger des secrets à partir d’un coffre de clés Azure dans votre production. Vous pouvez ensuite utiliser ces secrets en tant que variables dans votre définition de mise en production, ce qui vous évite d’avoir à les stocker dans le contrôle de code source.

- Utilisez des [variables de mise en production][vsts-release-variables] dans vos définitions de mise en production pour piloter les changements de configuration de vos environnements. Les variables de mise en production peuvent être étendues à toute une version ou à un environnement donné. Si vous utilisez des variables pour les informations de secret, veillez à sélectionner l’icône en forme de cadenas.

- Les [portes de déploiement][vsts-deployment-gates] doivent être utilisées dans votre pipeline de mise en production. Vous pourrez ainsi tirer profit des données de supervision en les combinant avec des systèmes externes (par exemple, systèmes de gestion des incidents ou d’autres systèmes sur mesure) pour déterminer s’il faut promouvoir une mise en production.

- Lorsqu’une intervention manuelle dans un pipeline de mise en production est nécessaire, utilisez la fonctionnalité [approbations][vsts-approvals].

- Envisagez d’utiliser [Application Insights][application-insights] et des outils de supervision supplémentaires dès que possible dans votre pipeline de mise en production. De nombreuses organisations attendent d’être dans leur environnement de production pour commencer la supervision. La supervision de vos autres environnements vous permet d’identifier les éventuels bogues plus tôt dans le processus de développement et d’éviter ainsi l’apparition de problèmes dans votre environnement de production.

## <a name="deploy-the-scenario"></a>Déployez le scénario

### <a name="prerequisites"></a>Prérequis

- Vous devez disposer d’un compte Azure existant. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

- Vous devez inscrire une organisation Azure DevOps. Pour plus d’informations, consultez [Démarrage rapide : Créer votre organisation][vsts-account-create].

### <a name="walk-through"></a>Procédure pas à pas

Le [projet Azure DevOps](/azure/devops-project/azure-devops-project-github) déploie automatiquement un plan App Service, une instance App Service ainsi qu’une ressource App Insights, et configure automatiquement le projet Azure DevOps.

Une fois le projet Azure DevOps déployé et la compilation effectuée, vérifiez les modifications du code, les éléments de travail et les résultats des tests associés. Vous remarquerez qu’aucun résultat de test n’est affiché, car le code ne contient aucun test à exécuter.

Le projet crée un pipeline de mise en production et un déclenchement de déploiement continu, ce qui déploie notre application dans l’environnement de développement. Dans le cadre d’un processus de déploiement continu, il est possible que les mises en production couvrent plusieurs environnements. Une mise en production peut s’étendre sur chaque infrastructure (à l’aide de techniques telles qu’IaC) et peut également déployer les packages d’application requis en même temps que toutes les tâches de post-configuration.

## <a name="pricing"></a>Tarifs

Les coûts d’Azure DevOps varient selon le nombre d’utilisateurs de votre organisation qui nécessitent un accès, ainsi qu’en fonction d’autres facteurs tels que le nombre de builds/mises en production simultanés requis et le nombre d’utilisateurs de test. Pour plus d’informations, consultez la page [Tarification Azure DevOps][vsts-pricing-page].

La [calculatrice de prix][vsts-pricing-calculator] fournit une estimation pour l’exécution d’Azure DevOps avec 20 utilisateurs.

Azure DevOps est facturé par mois et par utilisateur. Des frais supplémentaires peuvent s’appliquer en fonction des pipelines simultanés requis, ainsi que de l’ajout d’utilisateurs de test et de licences de base utilisateur.

## <a name="related-resources"></a>Ressources associées

Pour plus d’informations sur les déploiements CI/CD et Azure DevOps, consultez les ressources suivantes :

- [Qu’est DevOps ?][devops-whatis]
- [DevOps at Microsoft - How we work with Azure DevOps][devops-microsoft] (DevOps chez Microsoft - Comment nous travaillons avec Azure DevOps)
- [Tutoriels pas à pas : DevOps avec Azure DevOps][devops-with-vsts]
- [Liste de contrôle DevOps][devops-checklist]
- [Créer un pipeline CI/CD pour .NET avec Azure DevOps Project][devops-project-create]

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
