---
title: Pipeline CI/CD avec VSTS
description: Un exemple de compilation et de publication d’une application .NET sur Azure Web Apps
author: christianreddington
ms.date: 07/11/18
ms.openlocfilehash: ae4ac5fc02cc841fc39b3cbef46124fe9da75e9b
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/14/2018
ms.locfileid: "39061010"
---
# <a name="cicd-pipeline-with-vsts"></a>Pipeline CI/CD avec VSTS

DevOps est l’intégration des opérations de développement, d’assurance qualité et informatiques. DevOps nécessite une culture unifiée et un ensemble solide de processus pour la distribution de logiciels.

Cet exemple de scénario illustre comment les équipes de développement peuvent utiliser Visual Studio Team Services pour déployer une application web .NET de deux couches sur Azure App Service. L’application web dépend des services PaaS en aval. Ce document souligne également quelques considérations que vous devez avoir lors de la conception d’un tel scénario à l’aide de la plateforme Azure as a service (PaaS).

L’adoption d’une approche moderne au développement d’applications à l’aide de l’intégration continue (CI) et du déploiement continu (CD) vous aide à accélérer la distribution de la valeur à vos utilisateurs par l’intermédiaire d’un service robuste de conception, de test, d’un déploiement et de surveillance. En utilisant une plateforme telle que Visual Studio Team Services en plus des services Azure tels que App Service, les organisations peuvent s’assurer de rester concentrées sur le développement de leur scénario, plus que la gestion de l’infrastructure pour l’activer.

## <a name="related-use-cases"></a>Cas d’usage connexes

Pensez à DevOps pour les cas d’usage suivants :

* Accélération du développement d’applications et des cycles de vie de déploiement
* Qualité et cohérence de la construction dans un processus de compilation et de mise en production automatisé

## <a name="architecture"></a>Architecture

![Présentation de l’architecture des composants Azure impliqués dans un scénario DevOps utilisant Visual Studio Team Services et Azure App Service][architecture]

Ce scénario couvre un pipeline DevOps pour une application web .NET à l’aide de Visual Studio Team Services (VSTS). Les données circulent dans le scénario comme suit :

1. Changer le code source de l’application.
2. Valider le code de l’application et le fichier web.config de l’application web.
3. L’intégration continue déclenche la génération de l’application et des tests unitaires.
4. Le déclencheur de déploiement continu orchestre le déploiement d’artefacts d’application  *avec des valeurs de configuration paramétrables propres à l’environnement*.
5. Déploiement vers Azure App Service.
6. Azure Application Insights collecte et analyse les données relatives à l’intégrité, aux performances et à l’utilisation.
7. Passez en revue les informations relatives à l’intégrité, aux performances et à l’utilisation.

### <a name="components"></a>Composants

* Les [groupes de ressources][resource-groups] constituent un conteneur logique pour les ressources Azure et fournissent également une limite de contrôle d’accès pour le plan de gestion, considérez un groupe de ressources comme représentant une « unité de déploiement ».
* [Visual Studio Team Services (VSTS)][vsts] est un service qui vous permet de gérer votre cycle de vie de développement de bout en bout, depuis la planification et la gestion du projet jusqu’à la gestion du code, en passant par la compilation et la mise en production.
* [Azure Web Apps][web-apps] est un service Platform as a Service (PaaS) pour l’hébergement d’applications web, d’API REST et de backends mobiles. Bien que cet article se concentre sur .NET, il existe plusieurs options de plateforme de développement supplémentaires prises en charge.
* [Application Insights][application-insights] est un service interne extensible de gestion des performances des applications (APM) destiné aux développeurs web sur de multiples plateformes.

### <a name="alternative-devops-tooling-options"></a>Autres options d’outils DevOps

Alors que cet article se concentre sur Visual Studio Team Services, [Team Foundation Server][team-foundation-server] peut être utilisé comme un substitut local. Sinon, vous pouvez également trouver une collection de technologies utilisées ensemble pour un pipeline de développement Open Source en tirant parti de [Jenkins][jenkins-on-azure].

À partir d’une infrastructure comme perspective de code, [les modèles Azure Resource Manager (ARM)][arm-templates] sont inclus comme une partie du projet Azure DevOps, mais vous pouvez envisager [Terraform][terraform] ou [Chef][chef] si vous avez des investissements ici. Si vous préférez un déploiement basé sur une infrastructure as a service (IaaS) et que vous avez besoin d’une gestion de la configuration, vous pourriez envisager [Azure Desired State Configuration][desired-state-configuration], [ Ansible] [ ansible] ou [Chef][chef].

### <a name="alternatives-to-web-app-hosting"></a>Alternatives à l’hébergement Web App

Alternatives à l’hébergement dans Azure Web Apps :

* [Machine virtuelle][compare-vm-hosting] : pour les charges de travail qui nécessitent un degré élevé de contrôle, ou qui dépendent des composants/services du système d’exploitation qui ne sont pas possibles avec Web Apps (par exemple, le GAC Windows, ou COM)
* [Hébergement de conteneur][azure-containers] : où il existe des dépendances de système d’exploitation et une portabilité d’hébergement, ou une densité d’hébergement, sont également des exigences.
* [Service Fabric][service-fabric] : une bonne option si l’architecture de la charge de travail est concentrée autour des composants distribués qui bénéficient d’un déploiement et d’une exécution sur un cluster avec un degré élevé de contrôle. Service Fabric peut également être utilisé pour héberger des conteneurs.
* [Fonctions Serverless Azure][azure-functions] : une bonne option si l’architecture de la charge de travail est centrée sur des composants distribués affinés , nécessitant des dépendances minimales, où les composants individuels doivent uniquement être exécutés à la demande (pas de façon continue) et où l’orchestration des composants n’est pas obligatoire.

### <a name="devops"></a>DevOps

L’**[Intégration continue (CI)][continuous-integration]** doit viser à illustrer une version stable, avec plusieurs développeurs (individuels ou en équipe) validant en permanence des modifications fréquentes sur la base de code partagée.
Vous devez effectuer les opérations suivantes dans le cadre de votre pipeline d’intégration continue :

* Archiver fréquemment de petites quantités de code (éviter la mise en lots des modifications plus grandes ou plus complexes car il est possible qu’elles soient plus difficiles à fusionner)
* Faire le test unitaire des composants de votre application avec une couverture suffisante du code (y compris les chemins d’accès mécontents)
* Garantir que la version est exécutée sur la branche partagée, principale (ou jonction). Cette branche doit être stable et maintenue comme « déploiement prêt ». Les modifications incomplètes ou en cours doivent être isolées dans une branche distincte avec des fusions « intégration en amont » fréquentes pour éviter des conflits ultérieures.

La **[livraison continue (CD)][continuous-delivery]** doit viser à montrer non seulement une version stable, mais également un déploiement stable. Cela rend la réalisation d’une livraison continue un peu plus difficile, la configuration spécifique à l’environnement est requise ainsi qu’un mécanisme permettant de définir ces valeurs correctement.

En outre, une couverture suffisante des tests d’intégration est exigée pour assurer que les différents composants sont configurés et fonctionnent correctement de bout en bout.

Ceci peut également nécessiter la configuration et la réinitialisation des données spécifiques à l’environnement et la gestion des versions du schéma de la base de données.

La livraison continue peut également s’étendre aux environnements de test de charge et d’acceptation utilisateur.

La livraison continue tire parti de la surveillance continue, dans l’idéal, au sein de tous les environnements.
La cohérence et la fiabilité des tests de déploiement et d’intégration dans des environnements sont facilitées par le script de la création et de la configuration ou par l’infrastructure d’hébergement (quelque chose qui est beaucoup plus facile pour les charges de travail basées sur le cloud, consultez l’infrastructure Azure en tant que code), également appelée [« infrastructure en tant que code »][infra-as-code].

* Démarrez la diffusion en continu dès que possible dans le cycle de vie du projet. Plus vous la démarrez tard, plus ce sera difficile.
* Les tests d’intégration et unitaires doivent avoir la même priorité que les fonctionnalités du projet
* Utilisez des packages de déploiement indépendant de l’environnement et gérez la configuration spécifique à l’environnement par le biais du processus de mise en production.
* Protégez une configuration sensible au sein des outils de gestion des mises en production ou en appelant un module de sécurité matériel (HSM), ou un [coffre de clés][azure-key-vault], au cours du processus de mise en production. Ne stockez pas de configuration sensible au sein du contrôle de code source.

**Apprentissage continu** : la surveillance la plus efficace d’un environnement CD est fournie par les outils Application-Performance-Monitoring (APM), par exemple [Application Insights][application-insights] de Microsoft. Une profondeur suffisante de surveillance pour une charge de travail d’application est essentielle pour comprendre les bogues et les performances en charge. [App Insights peut être intégré dans VSTS pour activer la surveillance continue du pipeline CD][app-insights-cd-monitoring]. Cela peut être utilisé pour permettre la progression automatique à l’étape suivante, sans intervention humaine ou restauration si une alerte est détectée.

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

Examinez les [modèles de conception standards pour la résilience][design-patterns-resiliency] et envisagez de les implémenter le cas échéant.

Vous pouvez trouver plusieurs [pratiques recommandées de la résilience pour App Service][resiliency-app-service] dans le centre d’architecture.

Pour obtenir des conseils d’ordre général sur la conception de solutions résilientes, consultez l’article [Conception d’applications résilientes pour Azure][resiliency].

## <a name="deploy-the-scenario"></a>Déployez le scénario

### <a name="prerequisites"></a>Prérequis

* Vous devez disposer d’un compte Azure existant. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit][azure-free-account] avant de commencer.
* Vous devez disposer d’un compte Visual Studio Team Services (VSTS) existant. Découvrez plus d’informations sur la [création d’un compte Visual Studio Team Services (VSTS)][vsts-account-create].

### <a name="walk-through"></a>Procédure

Dans ce scénario, vous utiliserez un projet Azure DevOps pour créer votre pipeline CI/CD.

Le projet DevOps déploiera pour vous un Plan App Service, App Service et une ressource App Insights, et il configurera également le projet Visual Studio Team Services.

Une fois que vous avez le projet DevOps et que la compilation est terminée, passez en revue les changements de code associés, les éléments de travail et les résultats des tests. Vous remarquerez qu'aucun résultat de test n’est affiché, car le code ne contient aucun test à exécuter.

Revoyez les définitions de mise en production. Notez qu’un pipeline de mise en production a été configuré, libérant notre application dans Dev. Observez qu’il y a un **déclencheur de déploiement continu** défini à partir de l’artefact de version **Drop**, avec des versions automatiques dans les environnements de développement. Dans le cadre d’un processus de déploiement continu, vous pouvez voir les versions s’étendre sur plusieurs environnements. Une version peut s’étendre sur chaque infrastructure (avec des techniques comme l’infrastructure en tant que code) et également déployer les packages d’application requis, ainsi que toutes les tâches de post-configuration.

**Considérations supplémentaires.**

* Envisagez l’utilisation de l’une des [tâches de création de jetons][vsts-tokenization] disponibles dans la place de marché VSTS.
* Envisagez l’utilisation de la tâche VSTS [Déployer : Azure Key Vault][download-keyvault-secrets] pour télécharger les secrets depuis un coffre de clés Azure vers votre version. Vous pouvez ensuite utiliser ces secrets comme variables dans le cadre de la définition de votre version et vous ne devez pas les stocker dans le contrôle de code source.
* Envisagez d’utiliser des [variables de mise en production][vsts-release-variables] dans vos définitions de mise en production pour piloter les changements de configuration de vos environnements. Les variables de mise en production peuvent être étendues à toute une version ou à un environnement donné. Si vous utilisez des variables pour les informations secrètes, veillez à sélectionner l’icône de cadenas.
* Envisagez d’utiliser des [portes de déploiement][vsts-deployment-gates] dans votre pipeline de mise en production. Cela vous permet de tirer parti des données de surveillance en association avec des systèmes externes (gestion des incidents par exemple, ou d’autres systèmes sur mesure) pour déterminer si une version doit être promue.
* Lorsqu’une intervention manuelle dans un pipeline de mise en production est requise, envisagez d’utiliser la fonctionnalité [approbations][vsts-approvals].
* Envisagez d’utiliser [Application Insights][application-insights] et des outils de surveillance supplémentaires dès que possible dans votre pipeline de mise en production. La plupart des organisations commencent seulement à surveiller leur environnement de production, même si vous pouvez identifier les éventuels bogues plus tôt dans le processus et éviter un impact sur vos utilisateurs en production.

## <a name="pricing"></a>Tarifs

Votre évaluation des coûts de Visual Studio Team Services varie en fonction du nombre d’utilisateurs dans votre organisation nécessitant un accès, en plus des facteurs tels que le numéro de build/versions simultanés requis et le nombre d’utilisateurs de test. Ils sont détaillés plus loin sur la [page de tarification VSTS][vsts-pricing-page].

* [Visual Studio Team Services (VSTS)][vsts-pricing-calculator] est un service qui vous permet de gérer votre cycle de vie de développement et qui est payé par utilisateur et de façon mensuelle. Il peut y avoir des frais supplémentaires en fonction des pipelines simultanés nécessaires, en plus des utilisateurs de test supplémentaires et des licences de base utilisateur.

## <a name="related-resources"></a>Ressources associées

* [Qu’est DevOps ?][devops-whatis]
* [DevOps chez Microsoft - Comment nous collaborons avec Visual Studio Team Services][devops-microsoft]
* [Didacticiels pas à pas : DevOps avec Visual Studio Team Services][devops-with-vsts]
* [Créer un pipeline CI/CD pour .NET avec Azure DevOps Project][devops-project-create]

<!-- links -->
[ansible]: /azure/ansible/
[application-insights]: /azure/application-insights/app-insights-overview
[app-service-reference-architecture]: /azure/architecture/reference-architectures/app-service-web-app/
[azure-free-account]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[architecture]: ./media/devops-dotnet-webapp/architecture-devops-dotnet-webapp.png
[availability]: /azure/architecture/checklist/availability
[chef]: /azure/chef/
[design-patterns-availability]: /azure/architecture/patterns/category/availability
[design-patterns-resiliency]: /azure/architecture/patterns/category/resiliency
[design-patterns-scalability]: /azure/architecture/patterns/category/performance-scalability
[design-patterns-security]: /azure/architecture/patterns/category/security
[desired-state-configuration]: /azure/automation/automation-dsc-overview
[devops-microsoft]: /azure/devops/devops-at-microsoft/
[devops-with-vsts]: https://almvm.azurewebsites.net/labs/vsts/
[application-insights]: https://azure.microsoft.com/en-gb/services/application-insights/
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
[vsts-account-create]: /vsts/organizations/accounts/create-account-msa-or-work-student?view=vsts
[vsts-approvals]: /vsts/pipelines/release/approvals/approvals?view=vsts
[devops-project]: https://portal.azure.com/?feature.customportal=false#create/Microsoft.AzureProject
[vsts-deployment-gates]: /vsts/pipelines/release/approvals/gates?view=vsts
[vsts-pricing-calculator]: https://azure.com/e/498aa024454445a8a352e75724f900b1
[vsts-pricing-page]: https://azure.microsoft.com/en-us/pricing/details/visual-studio-team-services/
[vsts-release-variables]: /vsts/pipelines/release/variables?view=vsts&tabs=batch
[vsts-tokenization]: https://marketplace.visualstudio.com/search?term=token&target=VSTS&category=All%20categories&sortBy=Relevance
[azure-key-vault]: /azure/key-vault/key-vault-overview
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[team-foundation-server]: https://visualstudio.microsoft.com/tfs/
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[service-fabric]:/azure/service-fabric/
[azure-functions]:/azure/azure-functions/
[azure-containers]:https://azure.microsoft.com/en-us/overview/containers/
[compare-vm-hosting]:/azure/app-service/choose-web-site-cloud-service-vm
[app-insights-cd-monitoring]:/azure/application-insights/app-insights-vsts-continuous-monitoring
[azure-region-pair-bcdr]:/azure/best-practices-availability-paired-regions
[devops-project-create]: /vsts/pipelines/apps/cd/azure/azure-devops-project-aspnetcore?view=vsts
[security]: /azure/security/