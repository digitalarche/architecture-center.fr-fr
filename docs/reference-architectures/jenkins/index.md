---
title: Exécuter un serveur Jenkins sur Azure
titleSuffix: Azure Reference Architectures
description: Cette architecture recommandée montre comment déployer et utiliser un serveur Jenkins scalable de classe entreprise sur Azure sécurisé avec l’authentification unique (SSO).
author: njray
ms.date: 04/30/2018
ms.custom: seodec18
ms.openlocfilehash: 26bf9cadc8db0cd4fcc61023619ca61bb7b87855
ms.sourcegitcommit: bb7fcffbb41e2c26a26f8781df32825eb60df70c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/20/2018
ms.locfileid: "53644153"
---
# <a name="run-a-jenkins-server-on-azure"></a>Exécuter un serveur Jenkins sur Azure

Cette architecture de référence montre comment déployer et utiliser un serveur Jenkins professionnel et évolutif sur Azure sécurisé avec l’authentification unique (SSO). L’architecture utilise également Azure Monitor pour surveiller l’état du serveur Jenkins. [**Déployez cette solution**](#deploy-the-solution).

![Serveur Jenkins s’exécutant sur Azure][0]

*Téléchargez un [fichier Visio](https://archcenter.blob.core.windows.net/cdn/Jenkins-architecture.vsdx) contenant ce diagramme d’architecture.*

Cette architecture prend en charge la récupération d’urgence avec les services Azure, mais ne couvre pas les scénarios plus avancés de montée en puissance impliquant plusieurs maîtres ou la haute disponibilité (HA) sans temps d’arrêt. Pour des informations générales sur les différents composants Azure, notamment un didacticiel pas à pas sur la création d’un pipeline d’intégration continue/de déploiement continu sur Azure, consultez [Jenkins sur Azure][jenkins-on-azure].

Ce document porte sur les opérations Azure principales nécessaires à la prise en charge de Jenkins, notamment l’utilisation du stockage Azure pour mettre à jour les artefacts de build, les éléments de sécurité nécessaires à l’authentification unique, d’autres services qui peuvent être intégrés et l’évolutivité pour le pipeline. Cette architecture est conçue pour fonctionner avec un référentiel de contrôle de code source existant. Par exemple, un scénario courant est de démarrer des travaux Jenkins selon les validations de GitHub.

## <a name="architecture"></a>Architecture

L’architecture est constituée des composants suivants :

- **Groupe de ressources**. Un [groupe de ressources][rg] sert à regrouper des ressources Azure afin de pouvoir les gérer en fonction de la durée de vie, du propriétaire et d’autres critères. Utiliser des groupes de ressources vous permet de déployer et de surveiller les ressources Azure en tant que groupe et de suivre les coûts de facturation par groupe de ressources. Vous pouvez également supprimer des ressources dans un ensemble, ce qui est très utile pour les déploiements de test.

- **Serveur Jenkins**. Une machine virtuelle est déployée pour exécuter [Jenkins][azure-market] en tant que serveur Automation et servir en tant que maître Jenkins. Cette architecture de référence utilise le modèle de solution [ pour Jenkins sur Azure][solution], installé sur une machine virtuelle Linux (Ubuntu 16.04 LTS) sur Azure. D’autres offres Jenkins sont disponibles sur la Place de marché Microsoft Azure.

  > [!NOTE]
  > Nginx est installé sur la machine virtuelle en tant que proxy inverse pour Jenkins. Vous pouvez configurer Nginx pour activer SSL pour le serveur Jenkins.
  >

- **Réseau virtuel**. Un [réseau virtuel][vnet] connecte des ressources Azure entre elles et fournit une isolation logique. Dans cette architecture, le serveur Jenkins s’exécute dans un réseau virtuel.

- **Sous-réseaux**. Le serveur Jenkins est isolé dans un [sous-réseau][subnet] pour le rendre plus facile à gérer et répartir le trafic réseau sans impact sur les performances.

- **NSG**. Utilisez des [groupes de sécurité réseau][nsg] (NSG) pour limiter le trafic réseau à partir d’Internet vers le sous-réseau d’un réseau virtuel.

- **Disques managés**. Un [disque managé][managed-disk] est un disque dur virtuel (VHD) persistant utilisé pour le stockage d’application et également pour maintenir l’état du serveur Jenkins et permettre une récupération d’urgence. Les disques de données sont stockés dans Stockage Azure. Le [stockage Premium][premium] est recommandé pour de hautes performances.

- **Stockage Blob Azure**. Le [plug-in Stockage Microsoft Azure][configure-storage] utilise le stockage Blob Azure pour stocker les artefacts de build qui sont créés et partagés avec d’autres builds Jenkins.

- **Azure Active Directory (Azure AD)**. [Azure AD][azure-ad] prend en charge l’authentification utilisateur, ce qui vous permet de configurer l’authentification unique. Les [principaux de service][service-principal] Azure AD définissent la stratégie et les autorisations pour chaque autorisation de rôle dans le flux de travail via le [contrôle d’accès en fonction du rôle][rbac] (RBAC). Chaque principal de service est associé à un travail Jenkins.

- **Azure Key Vault**. Pour gérer les clés secrètes et de chiffrement utilisées pour configurer les ressources Azure lorsque les secrets sont requis, cette architecture utilise [Key Vault][key-vault]. Pour obtenir de l’aide supplémentaire concernant le stockage des secrets associés à l’application dans le pipeline, consultez également le plug-in pour Jenkins [Informations d’identification Azure][configure-credential].

- **Services de surveillance Azure**. Ce service [analyse][monitor] la machine virtuelle Azure hébergeant Jenkins. Ce déploiement surveille l’état de la machine virtuelle et l’utilisation de l’UC et envoie des alertes.

## <a name="recommendations"></a>Recommandations

Les recommandations suivantes s’appliquent à la plupart des scénarios. Suivez ces recommandations, sauf si vous avez un besoin spécifique qui vous oblige à les ignorer.

### <a name="azure-ad"></a>Azure AD

Le locataire [Azure AD][azure-ad] pour votre abonnement Azure est utilisé pour activer l’authentification unique pour les utilisateurs Jenkins et pour configurer les [principaux de service][service-principal] qui permettent aux travaux Jenkins d’accéder aux ressources Azure.

L’autorisation et l’authentification unique sont implémentées par le plug-in Azure AD installé sur le serveur Jenkins. L’authentification unique vous permet de vous authentifier à l’aide des informations d’identification Azure AD de votre organisation lors de la connexion au serveur Jenkins. Lorsque vous configurez le plug-in Azure AD, vous pouvez spécifier le niveau d’accès autorisé d’un utilisateur au serveur Jenkins.

Pour fournir des travaux Jenkins avec accès aux ressources Azure, un administrateur Azure AD crée des principaux de service. Ces derniers accordent aux applications, dans ce cas, les travaux Jenkins : [un accès autorisé et authentifié][ad-sp] aux ressources Azure.

Le [contrôle d’accès en fonction du rôle (RBAC)][rbac] définit et contrôle davantage l’accès aux ressources Azure pour les utilisateurs ou les principaux de service via leur rôle assigné. Les rôles intégrés et personnalisés sont pris en charge. Les rôles peuvent également aider à sécuriser le pipeline et garantissent que les responsabilités de l’agent ou de l’utilisateur sont affectées et autorisées correctement. En outre, la fonctionnalité RBAC peut être configurée pour limiter l’accès aux ressources Azure. Par exemple, un utilisateur peut être limité à l’utilisation des composants dans un groupe de ressources particulier.

### <a name="storage"></a>Stockage

Utilisez le [plug-in Jenkins Stockage Microsoft Azure][storage-plugin], qui est installé à partir de la Place de marché Microsoft Azure, pour stocker les artefacts de build qui peuvent être partagés avec d’autres builds et d’autres tests. Un compte de stockage Azure doit être configuré avant que ce plug-in puisse être utilisé par les travaux Jenkins.

### <a name="jenkins-azure-plugins"></a>Plug-ins Jenkins Azure

Le modèle de solution pour Jenkins sur Azure installe plusieurs plug-ins Azure. L’équipe DevOps Azure génère et gère le modèle de solution et les plug-ins suivants, qui fonctionnent avec les autres offres Jenkins dans la Place de marché Microsoft Azure, ainsi que n’importe quel maître Jenkins configuré localement :

- Le [plug-in Azure AD][configure-azure-ad] permet au serveur Jenkins de prendre en charge l’authentification unique pour les utilisateurs basés sur Azure AD.

- Le plug-in [Agents de machine virtuelle Azure][configure-agent] utilise un modèle Azure Resource Manager pour créer des agents Jenkins dans les machines virtuelles Azure.

- Le plug-in [Informations d’identification Azure][configure-credential] vous permet de stocker les informations d’identification du principal de service Azure dans Jenkins.

- Le [plug-in Stockage Microsoft Azure][configure-storage] télécharge les artefacts de build vers le [stockage Blob Azure][blob] ou télécharge les dépendances de build à partir de ce dernier.

Nous vous recommandons également de consulter la liste en constante évolution de tous les plug-ins Azure disponibles et fonctionnant avec les ressources Azure. Pour consulter la liste la plus récente, visitez l’[Index de plug-ins Jenkins][index] et recherchez Azure. Par exemple, les plug-ins suivants peuvent être déployés :

- Les [Agents Azure Container][container-agents] vous aident à exécuter un conteneur en tant qu’agent dans Jenkins.

- Le [déploiement continu Kubernetes](https://aka.ms/azjenkinsk8s) déploie des configurations de ressources vers un cluster Kubernetes.

- [Azure Container Service][acs] déploie des configurations sur Azure Container Service avec Kubernetes, DC/OS avec Marathon ou Docker Swarm.

- [Azure Functions][functions] déploie votre projet sur Azure Functions.

- [Azure App Service][app-service] déploie vers Azure App Service.

## <a name="scalability-considerations"></a>Considérations relatives à l’extensibilité

Jenkins peut mettre à l’échelle pour prendre en charge des charges de travail très importantes. Pour les builds élastiques, n’exécutez pas les builds sur le serveur maître Jenkins. Au lieu de cela, déchargez les tâches de build sur les agents Jenkins, qui peuvent être mis à l’échelle élastiquement selon les besoins. Tenez compte de deux options pour la mise à l’échelle des agents :

- Utilisez le plug-in [agents de machine virtuelle Azure][vm-agent] pour créer des agents Jenkins qui s’exécutent dans des machines virtuelles Azure. Ce plug-in permet la montée en charge élastique des agents et peut utiliser différents types de machines virtuelles. Vous pouvez sélectionner une autre image de base à partir de la Place de marché Microsoft Azure ou utiliser une image personnalisée. Pour plus d’informations sur la façon dont les agents Jenkins mettent à l’échelle, consultez [Architecture pour la mise à l’échelle][scale] dans la documentation Jenkins.

- Utilisez le plug-in [Agents Azure Container][container-agents] pour exécuter un conteneur en tant qu’agent dans [Azure Container Service avec Kubernetes](/azure/container-service/kubernetes/) ou dans [Azure Container Instances](/azure/container-instances/).

Généralement, la mise à l’échelle de machines virtuelles coûte plus cher que celle de conteneurs. Toutefois, pour mettre des conteneurs à l’échelle, votre processus de génération doit s’exécuter avec des conteneurs.

De plus, utilisez le stockage Azure pour partager des artefacts de build qui peuvent être utilisés dans l’étape suivante du pipeline par d’autres agents de build.

### <a name="scaling-the-jenkins-server"></a>Mise à l’échelle du serveur Jenkins

Vous pouvez réduire ou augmenter la puissance de la machine virtuelle du serveur Jenkins en modifiant sa taille. Le [modèle de solution pour Jenkins sur Azure][azure-market] spécifie la taille de DS2 v2 (avec deux UC, 7 Go) par défaut. Cette taille gère une charge de travail d’équipe petite à moyenne. Modifiez la taille de machine virtuelle en choisissant une autre option lors de la création du serveur.

La sélection d’un serveur de taille appropriée dépend de la taille de la charge de travail attendue. La communauté Jenkins gère un [guide de sélection][selection-guide] pour vous aider à identifier la configuration qui répond le mieux à vos besoins. Azure propose de nombreuses [tailles pour les machines virtuelles Linux][sizes-linux] pour répondre à toutes les conditions requises. Pour plus d’informations sur la mise à l’échelle du maître Jenkins, reportez-vous aux [meilleures pratiques][best-practices] de la communauté Jenkins, qui incluent également des détails sur la mise à l’échelle du maître Jenkins.

## <a name="availability-considerations"></a>Considérations relatives à la disponibilité

Dans le contexte d’un serveur Jenkins, la disponibilité signifie être en mesure de récupérer les informations d’état associées à votre flux de travail, telles que les résultats de tests, les bibliothèques que vous avez créées ou d’autres artefacts. Les états ou les artefacts de flux de travail critiques doivent être tenus à jour pour récupérer le flux de travail si le serveur Jenkins tombe en panne. Pour évaluer les exigences de disponibilité, tenez compte des deux mesures courantes :

- L’objectif de délai de récupération (RTO) spécifie la durée pendant laquelle le fonctionnement peut se poursuivre sans Jenkins.

- L’objectif de point de récupération (RPO) indique la quantité de données que vous pouvez vous permettre de perdre si une interruption de service affecte Jenkins.

Dans la pratique, le RTO et le RPO impliquent la redondance et la sauvegarde. La disponibilité n’est pas une question de récupération du matériel, qui fait partie d’Azure, mais plutôt de maintien à jour de l’état de votre serveur Jenkins. Microsoft propose un [contrat de niveau de service][sla] (SLA) pour les instances de machine virtuelle uniques. Si ce contrat de niveau de service (SLA) ne répond pas à vos exigences de durée de fonctionnement, assurez vous de disposer d’un plan de récupération d’urgence, ou envisagez l’utilisation d’un déploiement de [serveur Jenkins multimaître][multi-master] (non traitée dans ce document).

Envisagez l’utilisation des [scripts][disaster] de récupération d’urgence à l’étape 7 du déploiement pour créer un compte de stockage Azure avec des disques managés pour stocker l’état du serveur Jenkins. Si Jenkins tombe en panne, il peut être restauré à l’état stocké dans ce compte de stockage distinct.

## <a name="security-considerations"></a>Considérations relatives à la sécurité

Utilisez l’une des approches suivantes pour verrouiller la sécurité sur un serveur de base Jenkins, étant donné qu’il n’est pas sécurisé dans son état de base.

- Configurer un moyen de connexion sécurisé au serveur Jenkins. Cette architecture utilise le protocole HTTP et a une adresse IP publique, mais HTTP n’est pas sécurisé par défaut. Envisagez de configurer [HTTPS sur le serveur Nginx][nginx] utilisé pour une ouverture de session sécurisée.

    > [!NOTE]
    > Lors de l’ajout du protocole SSL sur votre serveur, créez une règle de groupe de sécurité réseau (NSG) pour le sous-réseau Jenkins afin d’ouvrir le port 443. Pour plus d’informations, consultez [Guide d’ouverture de ports vers une machine virtuelle avec le portail Azure][port443].
    >

- Assurez-vous que la configuration de Jenkins empêche la falsification de requête intersites (Gérer Jenkins \> Configurer la sécurité globale). Il s’agit de la valeur par défaut pour le serveur Microsoft Jenkins.

- Configurer l’accès en lecture seule au tableau de bord Jenkins à l’aide du [plug-in de stratégie d’autorisation de matrice][matrix].

- Installer le plug-in [Informations d’identification Azure][configure-credential] pour utiliser Key Vault pour gérer les secrets des ressources Azure, des agents dans le pipeline et des composants tiers.

- Utilisez RBAC pour restreindre l’accès au principal du service à la condition minimale requise pour exécuter les travaux. Cela permet de limiter l’étendue des dommages causés par un travail non fiable.

Les travaux Jenkins nécessitent souvent des secrets pour accéder aux services Azure qui requièrent une autorisation, tels que Azure Container Service. Utilisez [Key Vault][key-vault] avec le [plug-in Informations d’identification Azure][configure-credential] pour gérer ces secrets en toute sécurité. Utilisez Key Vault pour stocker les informations d’identification du principal de service, les mots de passe, les jetons et d’autres secrets.

Pour obtenir un aperçu global de l’état de toutes vos ressources Azure en termes de sécurité, utilisez [Azure Security Center][security-center]. Il surveille les problèmes potentiels de sécurité et fournit une image complète de la sécurité de votre déploiement. Le Centre de sécurité est configuré pour chaque abonnement Azure. Activez la collecte de données de sécurité comme décrit dans le [Guide de démarrage rapide d’Azure Security Center][quick-start]. Lorsque la collecte de données est activée, Security Center analyse automatiquement les machines virtuelles créées dans le cadre de cet abonnement.

Le serveur Jenkins a son propre système de gestion d’utilisateur et la communauté Jenkins fournit les meilleures pratiques pour la [sécurisation d’une instance Jenkins sur Azure][secure-jenkins]. Le modèle de solution pour Jenkins sur Azure implémente ces meilleures pratiques.

## <a name="manageability-considerations"></a>Considérations relatives à la facilité de gestion

Utilisez des groupes de ressources pour organiser les ressources Azure qui sont déployées. Déployez des environnements de production ainsi que des environnements de développement ou de test dans des groupes de ressources distincts, afin de pouvoir surveiller les ressources de chaque environnement et regrouper les coûts de facturation par groupe de ressources. Vous pouvez également supprimer des ressources dans un ensemble, ce qui est très utile pour les déploiements de test.

Azure propose plusieurs fonctionnalités d’[analyse et de diagnostics][monitoring-diag] de l’infrastructure globale. Pour surveiller l’utilisation de l’UC, cette architecture déploie Azure Monitor. Par exemple, vous pouvez utiliser Azure Monitor pour surveiller l’utilisation de l’UC et envoyer une notification si l’utilisation de l’UC dépasse 80 %. (Une utilisation élevée de l’UC indique que vous devriez mettre à l’échelle la machine virtuelle du serveur Jenkins.) Vous pouvez aussi informer un utilisateur désigné si la machine virtuelle échoue ou est indisponible.

## <a name="communities"></a>Communautés

Les communautés peuvent répondre aux questions et vous aider à paramétrer un déploiement réussi. Tenez compte des éléments suivants :

- [Blog de la communauté Jenkins](https://jenkins.io/node/)
- [Forum Azure](https://azure.microsoft.com/support/forums/)
- [Stack Overflow Jenkins](https://stackoverflow.com/tags/jenkins/info)

Pour plus de meilleures pratiques de la communauté Jenkins, visitez [meilleures pratiques Jenkins][jenkins-best].

## <a name="deploy-the-solution"></a>Déployer la solution

Pour déployer cette architecture, suivez les étapes ci-dessous pour installer le [modèle de solution pour Jenkins sur Azure][azure-market], puis installez les scripts de récupération d’urgence et de surveillance dans les étapes ci-dessous.

### <a name="prerequisites"></a>Prérequis

- Cette architecture de référence requiert un abonnement Azure.
- Pour créer un principal de service Azure, vous devez disposer des droits d’administrateur pour le locataire Azure AD qui est associé au serveur Jenkins déployé.
- Ces instructions supposent que l’administrateur Jenkins est également un utilisateur Azure avec au moins des privilèges de collaborateur.

### <a name="step-1-deploy-the-jenkins-server"></a>Étape 1 : Déployer le serveur Jenkins

1. Ouvrez l’[image de place de marché Azure pour Jenkins][azure-market] dans votre navigateur web et cliquez sur **OBTENIR MAINTENANT** sur le côté gauche de la page.

2. Passez en revue les détails de tarification et sélectionnez **Continuer**, puis sélectionnez **Créer** pour configurer le serveur Jenkins dans le portail Azure.

Pour obtenir des instructions détaillées, consultez [Créer un serveur Jenkins sur une machine virtuelle Linux Azure à partir du portail Azure][create-jenkins]. Pour cette architecture de référence, il suffit de rendre le serveur opérationnel avec l’ouverture de session d’administrateur. Vous pouvez ensuite le configurer pour utiliser divers autres services.

### <a name="step-2-set-up-sso"></a>Étape 2 : Configurer l’authentification unique

L’étape est exécutée par l’administrateur Jenkins, qui doit également disposer d’un compte utilisateur dans le répertoire d’abonnement Azure AD et doit avoir le rôle Collaborateur.

Utilisez le [plug-in Azure AD][configure-azure-ad] à partir du centre de mise à jour Jenkins sur le serveur Jenkins et suivez les instructions pour configurer l’authentification unique.

### <a name="step-3-provision-jenkins-server-with-azure-vm-agent-plugin"></a>Étape 3 : Provisionner le serveur Jenkins avec le plug-in Agent de machine virtuelle Azure

L’étape est exécutée par l’administrateur Jenkins pour configurer le plug-in Agent de machine virtuelle Azure, qui est déjà installé.

[Pour configurer le plug-in, procédez comme suit][configure-agent]. Pour obtenir un didacticiel sur la configuration des principaux de service pour le plug-in, consultez [Mettre à l’échelle vos déploiements Jenkins pour répondre à la demande avec des agents de machines virtuelles Azure][scale-agent].

### <a name="step-4-provision-jenkins-server-with-azure-storage"></a>Étape 4 : Provisionner le serveur Jenkins avec Stockage Azure

L’étape est exécutée par l’administrateur Jenkins qui configure le plug-in Stockage Microsoft Azure, qui est déjà installé.

[Pour configurer le plug-in, procédez comme suit][configure-storage].

### <a name="step-5-provision-jenkins-server-with-azure-credential-plugin"></a>Étape 5 : Provisionner le serveur Jenkins avec le plug-in Informations d’identification Azure

L’étape est exécutée par l’administrateur Jenkins pour configurer le plug-in Informations d’identification Azure, qui est déjà installé.

[Pour configurer le plug-in, procédez comme suit][configure-credential].

### <a name="step-6-provision-jenkins-server-for-monitoring-by-the-azure-monitor-service"></a>Étape 6 : Provisionner le serveur Jenkins pour la supervision par le service Azure Monitor

Pour configurer la surveillance de votre serveur Jenkins, suivez les instructions de [Création d’alertes de métrique dans Azure Monitor pour les services Azure][create-metric].

### <a name="step-7-provision-jenkins-server-with-managed-disks-for-disaster-recovery"></a>Étape 7 : Provisionner le serveur Jenkins avec Managed Disks pour la reprise d’activité

Le groupe de produits Microsoft Jenkins a créé des scripts de récupération d’urgence qui génèrent un disque managé utilisé pour enregistrer l’état de Jenkins. Si le serveur tombe en panne, il peut être restauré à son état le plus récent.

Télécharger et exécuter des scripts de récupération d’urgence à partir de [GitHub][disaster].

Vous pouvez consulter l’[exemple de scénario Azure](/azure/architecture/example-scenario) suivant qui décrit des solutions spécifiques à l’aide des mêmes technologies :

- [Pipeline CI/CD pour les charges de travail basées sur des conteneurs](/azure/architecture/example-scenario/apps/devops-with-aks)

<!-- links -->

[acs]: https://aka.ms/azjenkinsacs
[ad-sp]: /azure/active-directory/develop/active-directory-integrating-applications
[app-service]: https://plugins.jenkins.io/azure-app-service
[azure-ad]: /azure/active-directory/
[azure-market]: https://azuremarketplace.microsoft.com/marketplace/apps/azure-oss.jenkins?tab=Overview
[best-practices]: https://jenkins.io/doc/book/architecting-for-scale/
[blob]: /azure/storage/common/storage-java-jenkins-continuous-integration-solution
[configure-azure-ad]: https://plugins.jenkins.io/azure-ad
[configure-agent]: https://plugins.jenkins.io/azure-vm-agents
[configure-credential]: https://plugins.jenkins.io/azure-credentials
[configure-storage]: https://plugins.jenkins.io/windows-azure-storage
[container-agents]: https://aka.ms/azcontaineragent
[create-jenkins]: /azure/jenkins/install-jenkins-solution-template
[create-metric]: /azure/monitoring-and-diagnostics/insights-alerts-portal
[disaster]: https://github.com/Azure/jenkins/tree/master/disaster_recovery
[functions]: https://aka.ms/azjenkinsfunctions
[index]: https://plugins.jenkins.io
[jenkins-best]: https://wiki.jenkins.io/display/JENKINS/Jenkins+Best+Practices
[jenkins-on-azure]: /azure/jenkins/
[key-vault]: /azure/key-vault/
[managed-disk]: /azure/virtual-machines/linux/managed-disks-overview
[matrix]: https://plugins.jenkins.io/matrix-auth
[monitor]: /azure/monitoring-and-diagnostics/
[monitoring-diag]: /azure/architecture/best-practices/monitoring
[multi-master]: https://jenkins.io/doc/book/architecting-for-scale/
[nginx]: https://www.digitalocean.com/community/tutorials/how-to-create-an-ssl-certificate-on-nginx-for-ubuntu-14-04
[nsg]: /azure/virtual-network/virtual-networks-nsg
[quick-start]: /azure/security-center/security-center-get-started
[port443]: /azure/virtual-machines/windows/nsg-quickstart-portal
[premium]: /azure/virtual-machines/linux/premium-storage
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rg]: /azure/azure-resource-manager/resource-group-overview
[scale]: https://jenkins.io/doc/book/architecting-for-scale/
[scale-agent]: /azure/jenkins/jenkins-azure-vm-agents
[selection-guide]: https://jenkins.io/doc/book/hardware-recommendations/
[service-principal]: /azure/active-directory/develop/active-directory-application-objects
[secure-jenkins]: https://jenkins.io/blog/2017/04/20/secure-jenkins-on-azure/
[security-center]: /azure/security-center/security-center-intro
[sizes-linux]: /azure/virtual-machines/linux/sizes?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json
[solution]: https://azure.microsoft.com/blog/announcing-the-solution-template-for-jenkins-on-azure/
[sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/
[storage-plugin]: https://wiki.jenkins.io/display/JENKINS/Windows+Azure+Storage+Plugin
[subnet]: /azure/virtual-network/virtual-network-manage-subnet
[vm-agent]: https://wiki.jenkins.io/display/JENKINS/Azure+VM+Agents+plugin
[vnet]: /azure/virtual-network/virtual-networks-overview
[0]: ./images/jenkins-server.png
