---
title: Pipeline CI/CD pour les charges de travail basées sur des conteneurs
description: Créez un pipeline DevOps pour une application web Node.js qui utilise Jenkins, Azure Container Registry, Azure Kubernetes Service, Cosmos DB et Grafana.
author: iainfoulds
ms.date: 07/05/2018
ms.openlocfilehash: db8de674ee2789c5b41cebebee5745ecc8544122
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610836"
---
# <a name="cicd-pipeline-for-container-based-workloads"></a>Pipeline CI/CD pour les charges de travail basées sur des conteneurs

Cet exemple de scénario s’applique aux entreprises qui veulent moderniser le développement des applications à l’aide de conteneurs et de flux de travail DevOps. Dans ce scénario, une application web Node.js est générée et déployée par Jenkins dans Azure Container Registry et Azure Kubernetes Service. Le service Azure Cosmos DB est utilisé pour atteindre un niveau de base de données distribuée mondialement. Pour surveiller et résoudre les problèmes de performances de l’application, la plateforme Azure Monitor est intégrée à un tableau de bord et à une instance Grafana.

Exemples de scénarios d’applications : fournir un environnement de développement automatisé, valider de nouvelles modifications de code et envoyer (push) de nouveaux déploiements dans des environnements de production ou intermédiaires. Traditionnellement, les entreprises devaient générer et compiler des applications et des mises à jour, et gérer une base de code monolithique et volumineuse. Avec une approche moderne en matière de développement d’applications qui utilise l’intégration continue (CI) et le déploiement continu (CD), vous pouvez générer, tester et déployer des services plus rapidement. Cette approche moderne vous permet de proposer plus rapidement des applications et des mises à jour à vos clients et de vous adapter aux variations des besoins de l’entreprise de manière plus agile.

À l’aide des services Azure comme Azure Kubernetes Service, Container Registry et Cosmos DB, les entreprises peuvent utiliser les tout derniers outils et techniques de développement d’applications pour simplifier le processus d’implémentation de la haute disponibilité.

## <a name="relevant-use-cases"></a>Cas d’usage appropriés

Les autres cas d’usage appropriés sont les suivants :

* Modernisation des pratiques de développement d’applications pour une approche basée sur les conteneurs et les microservices.
* Accélération du développement d’applications et des cycles de vie de déploiement.
* Automatisation des déploiements pour tester ou accepter les environnements pour la validation.

## <a name="architecture"></a>Architecture

![Présentation de l’architecture des composants Azure impliqués dans un scénario DevOps utilisant Jenkins, Azure Container Registry et Azure Kubernetes Service][architecture]

Ce scénario porte sur un pipeline DevOps pour une application web Node.js et un serveur principal de base de données. Les données circulent dans le scénario comme suit :

1. Un développeur modifie le code source d’une application web Node.js.
2. La modification du code est validée dans un référentiel de contrôle de code source, par exemple GitHub.
3. Pour démarrer le processus d’intégration continue (CI), un webhook GitHub déclenche une build de projet Jenkins.
4. Le travail de build Jenkins utilise un agent de build dynamique dans Azure Kubernetes Service pour effectuer un processus de génération de conteneur.
5. Une image de conteneur est créée à partir du code dans le contrôle de code source, puis envoyée à une instance d’Azure Container Registry.
6. Via un déploiement continu (CD), Jenkins déploie cette image de conteneur mise à jour vers le cluster Kubernetes.
7. L’application web Node.js utilise Cosmos DB en tant que son serveur principal. Cosmos DB et Azure Kubernetes Service envoient des mesures à Azure Monitor.
8. Une instance Grafana fournit des tableaux de bord visuels des performances d’applications basées sur les données d’Azure Monitor.

### <a name="components"></a>Composants

* [Jenkins][jenkins] est un serveur d’automatisation open source qui peut s’intégrer aux services Azure pour activer l’intégration continue (CI) et le déploiement continu (CD). Dans ce scénario, Jenkins orchestre la création d’images de conteneur basées sur les validations effectuées dans le contrôle de code source, envoie ces images dans Azure Container Registry, puis met à jour les instances des applications dans Azure Kubernetes Service.
* Les [Machines virtuelles Linux Azure][docs-virtual-machines] constituent la plateforme IaaS utilisée pour exécuter les instances Jenkins et Grafana.
* [Azure Container Registry][docs-acr] stocke et gère les images de conteneur qui sont utilisées par le cluster Azure Kubernetes Service. Les images sont stockées de manière sécurisée. La plateforme Azure peut les répliquer dans d’autres régions pour accélérer les déploiements.
* [Azure Kubernetes Service][docs-aks] est une plateforme Kubernetes managée, qui vous permet de déployer et de gérer des applications en conteneur sans avoir à maîtriser l’orchestration de conteneurs. En tant que service Kubernetes hébergé, Azure gère pour vous des tâches critiques telles que l’analyse de l'intégrité et la maintenance.
* [Azure Cosmos DB][docs-cosmos-db] est une base de données multimodèle mondialement distribuée qui vous permet de choisir parmi différents modèles de base de données et de cohérence en fonction de vos besoins. Avec Cosmos DB, vos données peuvent être répliquées dans le monde entier, et il n’existe aucun composant de réplication ou de gestion de cluster à déployer ou configurer.
* [Azure Monitor][docs-azure-monitor] vous permet de suivre les performances, de gérer la sécurité et d’identifier des tendances. Les mesures obtenues par Azure Monitor peuvent être utilisées par d’autres ressources et outils, tels que Grafana.
* [Grafana][grafana] est une solution open source destinée à interroger, visualiser, alerter et comprendre les mesures. Un plug-in de source de données pour Azure Monitor permet à Grafana de créer des tableaux de bord visuels pour surveiller les performances de vos applications exécutées dans Azure Kubernetes Service et utilisant Cosmos DB.

### <a name="alternatives"></a>Autres solutions

* [Azure Pipelines][azure-pipelines] permet d’implémenter un pipeline d’intégration continue (CI), de test et de déploiement continu (CD) pour tous types d’applications.
* La plateforme [Kubernetes][kubernetes] peut être exécutée directement sur des machines virtuelles Azure et non via un service géré si vous souhaitez davantage de contrôle sur le cluster.
* [Service Fabric][service-fabric] est un autre orchestrateur de conteneurs qui peut remplacer AKS.

## <a name="considerations"></a>Considérations

### <a name="availability"></a>Disponibilité

Pour surveiller les performances de vos applications et créer des rapports sur les problèmes, ce scénario combine Azure Monitor et Grafana pour obtenir des tableaux de bord visuels. Ces outils vous permettent de surveiller et de résoudre les problèmes de performances qui peuvent nécessiter des mises à jour du code, qui peuvent ensuite être déployées avec le pipeline CI/CD.

Partie intégrante du cluster Azure Kubernetes Service, un équilibreur de charge distribue le trafic des applications vers un ou plusieurs conteneurs (pods) qui exécutent votre application. Cette méthode d’exécution des applications en conteneur dans Kubernetes assure à vos clients une infrastructure hautement disponible.

Pour plus d’informations sur la disponibilité, voir la [liste de contrôle de la disponibilité][availability] dans le Centre des architectures Azure.

### <a name="scalability"></a>Extensibilité

Azure Kubernetes Service vous permet de mettre à l’échelle le nombre de nœuds de cluster pour répondre aux besoins de vos applications. À mesure que vos applications augmentent, vous pouvez augmenter le nombre de nœuds Kubernetes qui exécutent votre service.

Les données d’application sont stockées dans Azure Cosmos DB, une base de données multimodèle mondialement distribuée qui peut être mise à l’échelle dans le monde entier. Cosmos DB retire de ces données la nécessité de mettre à l’échelle votre infrastructure comme avec les composants de base de données traditionnels. Vous pouvez choisir de répliquer votre instance Cosmos DB dans le monde entier pour répondre aux besoins de vos clients.

Pour plus d’informations sur l’extensibilité, voir la [liste de contrôle de l’extensibilité][scalability] dans le Centre des architectures Azure.

### <a name="security"></a>Sécurité

Pour réduire l’empreinte des attaques, ce scénario n’expose pas l’instance de machine virtuelle Jenkins via HTTP. Pour les tâches de gestion qui nécessitent votre interaction avec Jenkins, vous allez créer une connexion à distance sécurisée à l’aide d’un tunnel SSH à partir de votre ordinateur local. Seule l’authentification par clé publique SSH est autorisée pour les instances de machines virtuelles Jenkins et Grafana. Les connexions par mot de passe sont désactivées. Pour plus d’informations, consultez l’article [Exécuter un serveur Jenkins sur Azure](../../reference-architectures/jenkins/index.md).

Pour assurer la séparation des informations d’identification et des autorisations, ce scénario utilise un principal de service Azure Active Directory (AD) dédié. Les informations d’identification de ce principal de service sont stockées sous la forme d’un objet d’informations d’identification sécurisé dans Jenkins afin qu’elles ne soient pas directement exposées ni visibles dans les scripts ou le pipeline de build.

Pour obtenir des conseils d’ordre général sur la conception de solutions sécurisées, consultez la [documentation sur la sécurité Azure][security].

### <a name="resiliency"></a>Résilience

Ce scénario utilise Azure Kubernetes Service pour votre application. Des composants de résilience sont intégrés à Kubernetes et surveillent et redémarrent les conteneurs (pods) en cas de problème. Si elle est associée à l’exécution de plusieurs nœuds Kubernetes, votre application peut tolérer un pod ou un nœud non disponible.

Pour obtenir des conseils d’ordre général sur la conception de solutions résilientes, consultez l’article [Conception d’applications résilientes pour Azure][resiliency].

## <a name="deploy-the-scenario"></a>Déployez le scénario

**Prérequis.**

* Vous devez disposer d’un compte Azure existant. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.
* Vous avez besoin d’une paire de clés publiques SSH. Pour connaître la procédure à suivre pour créer une paire de clés publiques, voir [Créer et utiliser une paire de clés publiques et privées SSH pour les machines virtuelles Linux dans Azure][sshkeydocs].
* Vous avez besoin d’un principal de service Azure Active Directory (AD) pour l’authentification du service et des ressources. Si nécessaire, vous pouvez créer un principal de service avec [az ad sp create-for-rbac][createsp].

    ```azurecli-interactive
    az ad sp create-for-rbac --name myDevOpsScenario
    ```

    Prenez note des valeurs *appId* et *password* dans la sortie de cette commande. Vous fournissez ces valeurs au modèle lorsque vous déployez le scénario.

Pour déployer ce scénario avec un modèle Azure Resource Manager, procédez comme suit.

1. Cliquez sur le bouton **Déployer sur Azure** :<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fdevops-with-aks%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Attendez l’ouverture de la solution Template deployment dans le portail Azure, puis procédez comme suit :
   * Sélectionnez **Créer un nouveau groupe de ressources**, puis indiquez un nom, par exemple *myAKSDevOpsScenario* dans la zone de texte.
   * Dans la zone de liste déroulante **Emplacement**, sélectionnez une région.
   * Entrez votre ID d’application de principal de service et le mot de passe à partir de la commande `az ad sp create-for-rbac`.
   * Indiquez un nom d’utilisateur et un mot de passe sécurisé pour l’instance Jenkins et la console Grafana.
   * Fournissez une clé SSH pour sécuriser les connexions établies avec les machines virtuelles Linux.
   * Passez en revue les termes et conditions, puis cochez la case **J’accepte les termes et conditions mentionnés ci-dessus**.
   * Sélectionnez le bouton **Acheter**.

Le déploiement prend 15 à 20 minutes.

## <a name="pricing"></a>Tarifs

Pour explorer le coût d’exécution de ce scénario, tous les services sont préconfigurés dans le calculateur de coûts. Pour pouvoir observer l’évolution de la tarification pour votre cas d’usage particulier, modifiez les variables appropriées en fonction du trafic que vous escomptez.

Nous proposons trois exemples de profils de coût basés sur le nombre d’images de conteneur à stocker et les nœuds Kubernetes pour exécuter vos applications.

* [Petit][small-pricing] : 1 000 builds de conteneurs par mois.
* [Moyen][medium-pricing] : 100 000 builds de conteneurs par mois.
* [Grand][large-pricing] : 1 000 000 builds de conteneurs par mois.

## <a name="related-resources"></a>Ressources associées

Ce scénario a utilisé Azure Container Registry et Azure Kubernetes Service pour stocker et exécuter une application basée sur des conteneurs. Le service Azure Container Instances peut également être utilisé pour exécuter des applications basées sur des conteneurs, sans avoir à approvisionner de composants d’orchestration. Pour plus d’informations, consultez l’article [Vue d’ensemble d’Azure Container Instances][docs-aci].

<!-- links -->
[architecture]: ./media/architecture-devops-with-aks.png
[autoscaling]: ../../best-practices/auto-scaling.md
[availability]: ../../checklist/availability.md
[docs-aci]: /azure/container-instances/container-instances-overview
[docs-acr]: /azure/container-registry/container-registry-intro
[docs-aks]: /azure/aks/intro-kubernetes
[docs-azure-monitor]: /azure/monitoring-and-diagnostics/monitoring-overview
[docs-cosmos-db]: /azure/cosmos-db/introduction
[docs-virtual-machines]: /azure/virtual-machines/linux/overview
[createsp]: /cli/azure/ad/sp#az-ad-sp-create
[grafana]: https://grafana.com/
[jenkins]: https://jenkins.io/
[resiliency]: ../../resiliency/index.md
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[scalability]: ../../checklist/scalability.md
[sshkeydocs]: /azure/virtual-machines/linux/mac-create-ssh-keys
[azure-pipelines]: /azure/devops/pipelines
[kubernetes]: https://kubernetes.io/
[service-fabric]: /azure/service-fabric/

[small-pricing]: https://azure.com/e/841f0a75b1ea4802ba1ac8f7918a71e7
[medium-pricing]: https://azure.com/e/eea0e6d79b4e45618a96d33383ec77ba
[large-pricing]: https://azure.com/e/3faab662c54c473da55a1e93a27e0e64