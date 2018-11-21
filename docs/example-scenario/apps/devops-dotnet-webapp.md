---
title: Pipeline CI/CD avec Azure DevOps
description: Générez et publiez une application .NET sur Azure Web Apps à l’aide d’Azure DevOps.
author: christianreddington
ms.date: 07/11/18
ms.openlocfilehash: 97f16b2d3d9c15bc6f5db6fad4c9d8097243ad3d
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610785"
---
# <a name="cicd-pipeline-with-azure-devops"></a>Pipeline CI/CD avec Azure DevOps

DevOps est l’intégration du développement, de l’assurance qualité et des opérations informatiques. DevOps nécessite une culture unifiée et un ensemble solide de processus pour la distribution de logiciels.

Cet exemple de scénario démontre la manière dont les équipes de développement peuvent utiliser Azure DevOps pour déployer une application web .NET à deux niveaux sur Azure App Service. L’application web dépend des services PaaS (platform as a service) Azure en aval. Ce document souligne également quelques points dont vous tenir compte lorsque vous concevez un scénario de ce type à l’aide d’Azure PaaS.

L’approche moderne en matière de développement d’applications que constituent l’intégration continue et le déploiement continu (CI/CD) vous aide à offrir plus rapidement une valeur ajoutée aux utilisateurs par l’intermédiaire d’un service robuste de compilation, de test, de déploiement et de supervision. En combinant l’utilisation d’une plateforme telle qu’Azure DevOps et de services Azure comme App Service, les organisations peuvent se focaliser sur le développement de leur scénario, plutôt que sur la gestion de l’infrastructure sous-jacente.

## <a name="relevant-use-cases"></a>Cas d’usage appropriés

Pensez à DevOps pour les cas d’usage suivants :

* Accélération du développement d’applications et des cycles de vie de développement
* Qualité et cohérence de la construction dans un processus de compilation et de mise en production automatisé

## <a name="architecture"></a>Architecture

![Présentation de l’architecture des composants Azure impliqués dans un scénario DevOps utilisant Azure DevOps et Azure App Service][architecture]

Ce scénario porte sur un pipeline CI/CD pour une application web .NET utilisant Azure DevOps. Les données circulent dans le scénario comme suit :

1. Changer le code source de l’application.
2. Valider le code de l’application et le fichier web.config de l’application web.
3. L’intégration continue déclenche la génération de l’application et des tests unitaires.
4. Le déclencheur de déploiement continu orchestre le déploiement d’artefacts d’application  *avec des valeurs de configuration paramétrables propres à l’environnement*.
5. Déploiement vers Azure App Service.
6. Azure Application Insights collecte et analyse les données relatives à l’intégrité, aux performances et à l’utilisation.
7. Passez en revue les informations relatives à l’intégrité, aux performances et à l’utilisation.

### <a name="components"></a>Composants

* [Azure DevOps][vsts] est un service qui vous permet de gérer votre cycle de vie de développement de bout en bout, depuis la planification et la gestion du projet jusqu’à la compilation et la mise en production, en passant par la gestion du code.
* [Azure Web Apps][web-apps] est un service PaaS pour l’hébergement d’applications web, d’API REST et de back ends mobiles. Bien que cet article se concentre sur .NET, il existe plusieurs options de plateforme de développement supplémentaires prises en charge.
* [Application Insights][application-insights] est un service interne extensible de gestion des performances des applications (APM) destiné aux développeurs web sur de multiples plateformes.

### <a name="alternative-devops-tooling-options"></a>Autres options d’outils DevOps

Bien que cet article se concentre sur Azure DevOps, [Team Foundation Server][team-foundation-server] peut faire office de substitut local. Une autre possibilité consiste à utiliser un ensemble de technologies pour un pipeline de développement open source à l’aide de [Jenkins][jenkins-on-azure].

Dans une perspective IaC (infrastructure as code), les [modèles Azure Resource Manager][arm-templates] sont inclus dans le projet Azure DevOps, mais vous pouvez éventuellement utiliser [Terraform][terraform] ou [Chef][chef]. Si vous préférez un déploiement IaaS (infrastructure as a service) et que vous avez besoin d’une gestion de la configuration, tournez-vous vers [Azure Automation State Configuration][desired-state-configuration], [Ansible][ansible] ou [Chef][chef].

### <a name="alternatives-to-azure-web-apps"></a>Alternatives à Azure Web Apps

Examinez ci-après les alternatives possibles à l’hébergement dans Azure Web Apps :

* [Machines virtuelles Azure][compare-vm-hosting] : pour les charges de travail qui exigent un degré de contrôle élevé ou qui dépendent de services et de composants du système d’exploitation non utilisables avec Web Apps (par exemple, GAC Windows ou COM).
* [Service Fabric][service-fabric] : option appropriée si l’architecture de la charge de travail s’articule autour de composants distribués qui tirent avantage d’un déploiement et d’une exécution sur un cluster avec un haut degré de contrôle. Service Fabric peut également être utilisé pour héberger des conteneurs.
* [Azure Functions][azure-functions] : approche serverless efficace si l’architecture de la charge de travail s’articule autour de composants distribués de granularité fine, nécessitant des dépendances minimales, où les différents composants doivent uniquement être exécutés à la demande (et non de façon continue) et où l’orchestration des composants n’est pas obligatoire.

Cet [arbre de décision](/azure/architecture/guide/technology-choices/compute-decision-tree) peut vous aider lors du choix de la voie à suivre pour une migration.

### <a name="devops"></a>DevOps

**[L’intégration continue (CI)][continuous-integration]** garantit un build stable, car plusieurs développeurs valident régulièrement les modifications mineures fréquemment apportées au codebase partagé. Dans le cadre de votre pipeline d’intégration continue, vous devez :
* Valider fréquemment les modifications de code mineures. Évitez le traitement par lot des modifications plus importantes ou plus complexes dont la fusion peut se révéler plus difficile.
* Exécuter le test unitaire des composants de votre application avec une couverture du code suffisante, incluant le test des chemins d’accès incorrects.
* Garantir l’exécution du build sur la branche maîtresse (ou jonction) partagée. Cette branche doit être stable et maintenue comme « déploiement prêt ». Les modifications incomplètes ou en cours doivent être isolées dans une branche distincte avec de fréquentes fusions « intégration en aval » pour éviter les conflits ultérieurs.

La **[livraison continue (CD)][continuous-delivery]** garantit aussi bien la stabilité du build que celle du déploiement. La mise en œuvre de cette approche se révèle un peu plus ardue, car elle nécessite une configuration propre à l’environnement, ainsi qu’un mécanisme permettant de définir correctement ces valeurs. Voici les autres points à prendre en compte en matière de livraison continue :
* Une couverture suffisante des tests d’intégration est requise pour garantir la configuration correcte et le bon fonctionnement de bout en bout des différents composants.
* La livraison continue peut également nécessiter la configuration et la réinitialisation des données propres à l’environnement, ainsi que la gestion des versions du schéma de la base de données.
* La livraison continue doit également s’étendre aux environnements de test de charge et d’acceptation utilisateur.
* La livraison continue tire profit d’une supervision continue, au sein de tous les environnements dans l’idéal.
* L’automatisation, par le biais de scripts, de la création et de la configuration de l’infrastructure d’hébergement améliore la cohérence et la fiabilité des déploiements et des tests d’intégration dans les différents environnements. Cette tâche se révèle considérablement plus simple pour les charges de travail cloud. Pour plus d’informations, consultez l’article [Infrastructure as Code][infra-as-code].
* Commencez à appliquer la livraison continue dès que possible dans le cycle de vie du projet. Plus vous tarderez à la mettre en œuvre, plus il vous sera difficile de l’incorporer.
* Les tests d’intégration et unitaires doivent avoir la même priorité que les fonctionnalités d’application.
* Utilisez des packages de déploiement indépendants de l’environnement et gérez la configuration spécifique de l’environnement par le biais du processus de mise en production.
* Protégez la configuration sensible à l’aide des outils de gestion des mises en production, ou en appelant un module de sécurité matériel (HSM) ou le service [Azure Key Vault][azure-key-vault] au cours du processus de mise en production. Ne stockez pas de configuration sensible au sein du contrôle de code source.

**Apprentissage continu**. La supervision la plus efficace d’un environnement de livraison continue (CD) est assurée par les outils d’analyse des performances d’application (APM), par exemple [Application Insights][application-insights]. Un niveau de supervision suffisant pour une charge de travail d’application est essentiel à la compréhension des bogues ou des performances en charge. Il est possible d’intégrer Application Insights à VSTS pour assurer la [supervision continue du pipeline CD][app-insights-cd-monitoring]. Cela peut être utilisé pour permettre la progression automatique à l’étape suivante, sans intervention humaine ou restauration si une alerte est détectée.

## <a name="considerations"></a>Considérations

### <a name="availability"></a>Disponibilité

Envisagez l’utilisation des [modèles de conception standards pour la disponibilité][design-patterns-availability] lorsque vous générez votre application cloud.

Passez en revue les considérations sur la disponibilité dans [l’architecture de référence d’application web de App Service][app-service-reference-architecture] appropriée

Pour consulter d’autres rubriques relatives à la disponibilité, consultez la [liste de contrôle de la disponibilité][availability] dans le Centre des architectures Azure.

### <a name="scalability"></a>Extensibilité

Lors de la création d’une application cloud, soyez conscient des [modèles de conception standards pour l’évolutivité][design-patterns-scalability].

Passez en revue les considérations sur l’extensibilité dans [l’architecture de référence d’application web de App Service][app-service-reference-architecture] appropriée

Pour consulter d’autres rubriques relatives à l’extensibilité, consultez la [liste de contrôle de l’extensibilité][scalability] dans le Centre des architectures Azure.

### <a name="security"></a>Sécurité

Envisagez l’utilisation des [modèles de conception standards pour la sécurité][design-patterns-security] le cas échéant.

Passez en revue les considérations sur la sécurité dans [l’architecture de référence d’application web de App Service][app-service-reference-architecture] appropriée.

Pour obtenir des conseils d’ordre général sur la conception de solutions sécurisées, consultez la [documentation sur la sécurité Azure][security].

### <a name="resiliency"></a>Résilience

Envisagez l’implémentation de [modèles de conception standards pour la résilience][design-patterns-resiliency] le cas échéant.

Vous trouverez différentes [pratiques recommandées pour App Service][resiliency-app-service] dans le Centre des architectures Azure.

Pour obtenir des conseils d’ordre général sur la conception de solutions résilientes, consultez l’article [Conception d’applications résilientes pour Azure][resiliency].

## <a name="deploy-the-scenario"></a>Déployez le scénario

### <a name="prerequisites"></a>Prérequis

* Vous devez disposer d’un compte Azure existant. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit][azure-free-account] avant de commencer.
* Vous devez inscrire une organisation Azure DevOps. Pour plus d’informations, consultez l’article [Quickstart: Create your organization][vsts-account-create] (Démarrage rapide : Créer une organisation dans Azure DevOps).

### <a name="walk-through"></a>Procédure pas à pas

Dans ce scénario, vous allez utiliser le projet Azure DevOps pour créer votre pipeline CI/CD.

Le projet Azure DevOps déploie automatiquement un plan App Service, une instance App Service ainsi qu’une ressource App Insights, et configure le projet Azure DevOps à votre intention.

Une fois le projet Azure DevOps déployé et la compilation effectuée, vérifiez les modifications du code, les éléments de travail et les résultats des tests associés. Vous remarquerez qu’aucun résultat de test n’est affiché, car le code ne contient aucun test à exécuter.

Examinez les définitions de mise en production. Notez qu’un pipeline de mise en production a été configuré, entraînant le déploiement de notre application dans l’environnement de développement. Vous pouvez observer qu’un **déclencheur de déploiement continu** est défini à partir de l’artefact de build **Drop**, avec des mises en production automatiques dans l’environnement de développement. Dans le cadre d’un processus de déploiement continu, il est possible que les mises en production couvrent plusieurs environnements. Une mise en production peut s’étendre sur chaque infrastructure (à l’aide de techniques telles qu’IaC) et peut également déployer les packages d’application requis en même temps que toutes les tâches de post-configuration.

## <a name="additional-considerations"></a>Considérations supplémentaires

* Envisagez de tirer profit de l’une des [tâches de création de jetons][vsts-tokenization] disponibles sur VSTS marketplace.
* Envisagez d’utiliser la tâche VSTS [Déployer : Azure Key Vault][download-keyvault-secrets] pour télécharger les secrets d’une ressource Azure Key Vault dans votre mise en production. Vous pouvez ensuite utiliser ces secrets en tant que variables dans votre définition de mise en production, ce qui vous évite d’avoir à les stocker dans le contrôle de code source.
* Envisagez d’utiliser des [variables de mise en production][vsts-release-variables] dans vos définitions de mise en production pour piloter les changements de configuration de vos environnements. Les variables de mise en production peuvent être étendues à toute une version ou à un environnement donné. Si vous utilisez des variables pour les informations de secret, veillez à sélectionner l’icône en forme de cadenas.
* Envisagez d’utiliser des [portes de déploiement][vsts-deployment-gates] dans votre pipeline de mise en production. Vous pourrez ainsi tirer profit des données de supervision en les combinant avec des systèmes externes (par exemple, systèmes de gestion des incidents ou d’autres systèmes sur mesure) pour déterminer s’il faut promouvoir une mise en production.
* Lorsqu’une intervention manuelle dans un pipeline de mise en production est requise, envisagez d’utiliser la fonctionnalité [approbations][vsts-approvals].
* Envisagez d’utiliser [Application Insights][application-insights] et des outils de supervision supplémentaires dès que possible dans votre pipeline de mise en production. La plupart des organisations se contentent de superviser leur environnement de production. La supervision de vos autres environnements vous permet d’identifier les éventuels bogues plus tôt dans le processus de développement et d’éviter ainsi l’apparition de problèmes dans votre environnement de production.

## <a name="pricing"></a>Tarifs

Les coûts d’Azure DevOps varient selon le nombre d’utilisateurs de votre organisation qui nécessitent un accès, ainsi qu’en fonction d’autres facteurs tels que le nombre de builds/mises en production simultanés requis et le nombre d’utilisateurs de test. Pour plus d’informations, consultez la page [Tarification Azure DevOps][vsts-pricing-page].

* [Azure DevOps][vsts-pricing-calculator] est un service vous permettant de gérer votre cycle de vie de développement. Il est facturé par utilisateur et par mois. Des frais supplémentaires peuvent s’appliquer en fonction des pipelines simultanés requis, ainsi que de l’ajout d’utilisateurs de test et de licences de base utilisateur.

## <a name="related-resources"></a>Ressources associées

* [Qu’est DevOps ?][devops-whatis]
* [DevOps at Microsoft - How we work with Azure DevOps][devops-microsoft] (DevOps chez Microsoft - Comment nous travaillons avec Azure DevOps)
* [Didacticiels pas à pas : DevOps avec Azure DevOps][devops-with-vsts]
* [Liste de contrôle DevOps][devops-checklist]
* [Créer un pipeline CI/CD pour .NET avec Azure DevOps Project][devops-project-create]

<!-- links -->
[ansible]: /azure/ansible/
[application-insights]: /azure/application-insights/app-insights-overview
[app-service-reference-architecture]: ../../reference-architectures/app-service-web-app/basic-web-app.md
[azure-free-account]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[architecture]: ./media/architecture-devops-dotnet-webapp.png
[availability]: /azure/architecture/checklist/availability
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
[resiliency]: /azure/architecture/checklist/resiliency
[scalability]: /azure/architecture/checklist/scalability
[vsts]: /vsts/?view=vsts#pivot=services
[continuous-integration]: /azure/devops/what-is-continuous-integration
[continuous-delivery]: /azure/devops/what-is-continuous-delivery
[web-apps]: /azure/app-service/app-service-web-overview
[terraform]: /azure/terraform/
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
[team-foundation-server]: https://visualstudio.microsoft.com/tfs/
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[service-fabric]: /azure/service-fabric/
[azure-functions]: /azure/azure-functions/
[azure-containers]: https://azure.microsoft.com/overview/containers/
[compare-vm-hosting]: /azure/app-service/choose-web-site-cloud-service-vm
[app-insights-cd-monitoring]: /azure/application-insights/app-insights-vsts-continuous-monitoring
[azure-region-pair-bcdr]: /azure/best-practices-availability-paired-regions
[devops-project-create]: /azure/devops-project/azure-devops-project-aspnet-core
[security]: /azure/security/