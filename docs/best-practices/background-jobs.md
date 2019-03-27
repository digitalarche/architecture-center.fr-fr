---
title: Conseils sur les travaux en arrière-plan
titleSuffix: Best practices for cloud applications
description: Conseils portant sur l’exécution de tâches en arrière-plan indépendamment de l’interface utilisateur.
author: dragon119
ms.date: 11/05/2018
ms.topic: best-practice
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 4b96c19dd8613a941a7408e1b99945d5fa0f5364
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58248964"
---
# <a name="background-jobs"></a>Travaux en arrière-plan

Plusieurs types d’applications requièrent des tâches en arrière-plan qui s’exécutent indépendamment de l’interface utilisateur. Citons, par exemple, les traitements par lots, les tâches de traitement intensif et les processus de longue durée, comme les workflows. Les travaux en arrière-plan peuvent être exécutés sans nécessiter l’intervention de l’utilisateur. L’application peut démarrer le travail et continuer à traiter les demandes interactives des utilisateurs. Ce processus peut favoriser la réduction de la charge sur l’interface utilisateur de l’application, ce qui peut améliorer la disponibilité et limiter les temps de réponse interactifs.

Par exemple, si une application doit générer des images miniatures chargées par les utilisateurs, elle peut effectuer ce travail en arrière-plan et enregistrer les miniatures dans le stockage une fois le travail terminé, sans que l’utilisateur ait besoin d’attendre la fin du processus. De même, un utilisateur qui lance une commande peut lancer un workflow en arrière-plan pour traiter cette dernière, tandis que l’interface utilisateur permet à l’utilisateur de continuer à parcourir l’application web. Une fois le travail en arrière-plan terminé, le système peut mettre à jour la commande stockée et envoyer un e-mail de confirmation de la commande à l’utilisateur.

Si vous envisagez d’implémenter une tâche en tant que travail en arrière-plan, vous devez en premier lieu savoir si cette tâche peut s’exécuter sans intervention de l’utilisateur et sans que l’interface utilisateur ait besoin d’attendre la fin du travail. En effet, il n’est pas recommandé d’exécuter en arrière-plan des tâches qui obligent l’utilisateur ou l’interface utilisateur à attendre la fin de leur exécution.

## <a name="types-of-background-jobs"></a>Types de travaux en arrière-plan

Les travaux en arrière-plan incluent généralement un ou plusieurs des types de travaux suivants :

- Travaux nécessitant beaucoup de ressources processeur, comme les calculs mathématiques, les analyses de modèles structurels, etc.
- Travaux nécessitant un grand nombre d’E/S, comme l’exécution d’une suite de transactions de stockage ou l’indexation de fichiers.
- Travaux de traitement par lots, comme les opérations de traitement planifiées ou les mises à jour de données exécutées la nuit.
- Workflows de longue durée, comme le traitement de commandes ou l’approvisionnement de services et systèmes.
- Tâches de traitement de données sensibles, qui nécessitent d’être déplacées vers un emplacement plus sécurisé pour être traitées. Par exemple, vous devez éviter de traiter des données sensibles dans une application web. Utilisez plutôt un modèle tel que le [modèle opérateur de contrôle d’appels](../patterns/gatekeeper.md) pour transférer les données à un processus en arrière-plan isolé qui peut accéder au stockage protégé.

## <a name="triggers"></a>Déclencheurs

Vous pouvez lancer un travail en arrière-plan de différentes manières. Ces tâches appartiennent toutes à l’une des catégories suivantes :

- [**Déclencheurs pilotés par les événements**](#event-driven-triggers). La tâche est lancée suite à un événement, généralement une action effectuée par un utilisateur ou une étape d’un workflow.
- [**Déclencheurs pilotés par les planifications**](#schedule-driven-triggers). La tâche est appelée sur la base d’une planification basée sur un minuteur. Il peut s’agir d’une planification périodique ou d’un appel unique prévu pour une date ultérieure.

### <a name="event-driven-triggers"></a>Déclencheurs pilotés par les événements

Lors d’un appel piloté par un événement, un déclencheur est utilisé pour démarrer la tâche en arrière-plan. Voici quelques exemples d’utilisation de déclencheurs pilotés par les événements :

- L’interface utilisateur, ou un autre travail, place un message dans une file d’attente. Ce message contient des données relatives à une action qui a eu lieu, comme une commande passée par un utilisateur. La tâche en arrière-plan est à l’écoute sur cette file d’attente et détecte l’arrivée d’un nouveau message. Elle le lit et utilise les données qu’il inclut en tant qu’entrée pour le travail en arrière-plan.
- L’interface utilisateur, ou l’autre travail, enregistre ou met à jour une valeur dans le stockage. La tâche en arrière-plan surveille le stockage et détecte les modifications éventuelles. Elle lit les données et les utilise en tant qu’entrée pour le travail en arrière-plan.
- L’interface utilisateur, ou l’autre travail, envoie une demande à un point de terminaison, tel qu’un URI HTTPS, ou à une API exposée en tant que service web. Elle transmet les données nécessaires pour terminer la tâche en arrière-plan dans le cadre de la demande. Le point de terminaison ou le service web appelle la tâche en arrière-plan, qui utilise les données en tant qu’entrée.

L’appel piloté par un événement est adapté à plusieurs types de tâches, comme le traitement d’images, les workflows, l’envoi d’informations à des services distants, l’envoi d’e-mails et l’attribution de nouveaux utilisateurs dans des applications multilocataires.

### <a name="schedule-driven-triggers"></a>Déclencheurs pilotés par les planifications

Lors d’un appel piloté par une planification, un minuteur est utilisé pour démarrer la tâche en arrière-plan. Voici quelques exemples d’utilisation de déclencheurs pilotés par des planifications :

- Un minuteur exécuté en local dans l’application ou en tant qu’élément du système d’exploitation de l’application appelle une tâche en arrière-plan de manière régulière.
- Un minuteur exécuté dans une autre application, ou un service de minuteur comme Azure Scheduler, envoie régulièrement une demande à un service web ou à une API. Le service web ou l’API appelle la tâche en arrière-plan.
- Une application ou un processus distincts démarrent un minuteur, qui déclenche l’appel de la tâche en arrière-plan après un certain délai, ou à une heure spécifique.

L’appel piloté par une planification est adapté pour plusieurs types de tâches, comme les routines de traitement par lots (par exemple, mise à jour des listes de produits associés sur la base du comportement récent des utilisateurs), les tâches de traitement des données de routine (comme la mise à jour d’index ou la génération de résultats cumulés), l’analyse des données pour les rapports quotidiens, le nettoyage des données de rétention et la vérification de la cohérence des données.

Si vous utilisez une tâche pilotée par une planification qui doit s’exécuter en tant qu’instance unique, tenez compte de ce qui suit :

- Si l’instance de calcul qui exécute le planificateur (par exemple, une machine virtuelle utilisant des tâches Windows planifiées) est mise à l’échelle, plusieurs instances du planificateur peuvent s’exécuter. Ces instances peuvent alors démarrer plusieurs instances de la tâche.
- Si les tâches s’exécutent plus longtemps que la période séparant deux événements du planificateur, le planificateur peut démarrer une autre instance de la tâche alors que l’instance précédente est toujours en cours d’exécution.

## <a name="returning-results"></a>Renvoi de résultats

Les travaux en arrière-plan s’exécutent de façon asynchrone dans un processus distinct, voire à un autre emplacement, à partir de l’interface utilisateur ou du processus qui a appelé la tâche en arrière-plan. Dans l’idéal, les tâches en arrière-plan sont des opérations de type « Fire and Forget » ; la progression de leur exécution n’a aucun impact sur l’interface utilisateur ou le processus appelant. Cela signifie que le processus appelant n’attend pas la fin des tâches. Il ne peut donc pas déterminer automatiquement quand la tâche est terminée.

Si vous avez besoin qu’une tâche en arrière-plan communique avec la tâche appelante pour indiquer sa progression ou sa fin, vous devez implémenter un mécanisme à cette fin. Voici quelques exemples :

- Écriture de la valeur d’un indicateur d’état dans l’espace de stockage accessible à l’interface utilisateur ou à la tache appelante, qui peut surveiller ou vérifier cette valeur, le cas échéant. Les autres données que la tâche en arrière-plan doit renvoyer à la tâche appelante peuvent être placées dans le même espace de stockage.
- Établissement d’une file d’attente de réponse, que l’interface utilisateur ou la tâche appelante écoute. La tâche en arrière-plan peut envoyer des messages à la file d’attente, indiquant l’état et la fin du processus. Les données que la tâche en arrière-plan doit renvoyer à la tâche appelante peuvent être placées dans ces messages. Si vous utilisez Microsoft Azure Service Bus, vous pouvez utiliser les propriétés **ReplyTo** et **CorrelationId** pour implémenter cette fonctionnalité.
- Exposition d’une API ou d’un point de terminaison à partir de la tâche en arrière-plan, à laquelle l’interface utilisateur ou la tâche appelante peut accéder pour obtenir des informations sur l’état. Les données que la tâche en arrière-plan doit renvoyer à la tâche appelante peuvent être incluses dans la réponse.
- Rappel de la tâche en arrière-plan vers l’interface utilisateur ou la tâche appelante, via une API, pour indiquer l’état à différents points prédéfinis ou lors de l’achèvement. Cette opération peut être pilotée par des événements déclenchés en local, ou par un mécanisme de publication et d’abonnement. Les données que la tâche en arrière-plan doit renvoyer à la tâche appelante peuvent être incluses dans la charge utile de l’événement ou de la demande.

## <a name="hosting-environment"></a>Environnement d’hébergement

Vous pouvez héberger des tâches en arrière-plan à l’aide de différents services de plateforme Microsoft Azure :

- [**Azure Web Apps et WebJobs**](#azure-web-apps-and-webjobs). Vous pouvez utiliser WebJobs pour exécuter des tâches personnalisées basées sur différents types de scripts ou de programmes exécutables dans le contexte de l’application web.
- [**Azure Virtual Machines**](#azure-virtual-machines). Si vous utilisez un service Windows ou souhaitez utiliser le Planificateur de tâches de Windows, une pratique courante consiste à héberger les tâches en arrière-plan sur une machine virtuelle dédiée.
- [**Azure Batch**](#azure-batch). Batch est un service de plateforme qui planifie l’exécution des travaux nécessitant beaucoup de ressources système sur une collection gérée de machines virtuelles. Il peut automatiquement mettre à l’échelle les ressources de calcul.
- [**Azure Kubernetes Service**](#azure-kubernetes-service) (AKS). Azure Kubernetes Service fournit un environnement d’hébergement géré pour Kubernetes sur Azure.

Les sections suivantes décrivent chacune de ces options plus en détail, en ajoutant des informations qui vous permettront de choisir l’action la plus appropriée.

### <a name="azure-web-apps-and-webjobs"></a>Azure Web Apps et WebJobs

Vous pouvez utiliser Azure WebJobs pour exécuter des travaux personnalisés comme tâches en arrière-plan dans une application web Azure. Les tâches web s’exécutent dans le contexte de votre application web en tant que processus continu. Les tâches web s’exécutent également en réponse à un événement déclencheur d’Azure Scheduler ou à des facteurs externes, tels que des modifications apportées aux objets blob de stockage et aux files d’attente de messages. Les travaux peuvent être démarrés et arrêtés à la demande, et arrêtés normalement. Si une tâche web exécutée en continu échoue, elle est redémarrée automatiquement. Les actions liées aux nouvelles tentatives et aux erreurs sont configurables.

Quand vous configurez une tâche web :

- Si vous souhaitez que la tâche réponde à un déclencheur piloté par les événements, configurez-la avec l’option **Exécuter en continu**. Le script ou programme est stocké dans le dossier nommé site/wwwroot/app_data/jobs/continuous.
- Si vous souhaitez que la tâche réponde à un déclencheur piloté par les planifications, configurez-la avec l’option **Exécuter selon une planification**. Le script ou programme est stocké dans le dossier nommé site/wwwroot/app_data/jobs/triggered.
- Si vous choisissez l’option **Exécuter à la demande** lorsque vous configurez un travail, ce dernier exécute le même code qu’avec l’option **Exécuter selon une planification**.

Les tâches web Azure s’exécutent dans le bac à sable (sandbox) de l’application web. Cela signifie qu’elles peuvent accéder aux variables d’environnement et partager des informations, telles que les chaînes de connexion, avec l’application web. Le travail peut accéder à l’identificateur unique de la machine qui l’exécute. La chaîne de connexion **AzureWebJobsStorage** fournit l’accès aux files d’attente, objets blob et tables de stockage Azure pour les données d’application, et à Service Bus pour la messagerie et les communications. La chaîne de connexion **AzureWebJobsDashboard** fournit l’accès aux fichiers journaux d’actions des tâches.

Les tâches web Azure présentent les caractéristiques suivantes :

- **Sécurité** : les tâches web sont protégées par les informations d’identification sur le déploiement de l’application web.
- **Types de fichiers pris en charge** : vous pouvez définir les tâches web avec des scripts de commande (.cmd), des fichiers de commandes (.bat), des scripts PowerShell (.ps1), des scripts Shell bash (.sh), des scripts PHP (.php), des scripts Python (.py), du code JavaScript (.js) ou des programmes exécutables (.exe, .jar, etc.).
- **Déploiement** : vous pouvez déployer les scripts et exécutables à l’aide du [portail Azure](/azure/app-service-web/web-sites-create-web-jobs), de [Visual Studio](/azure/app-service-web/websites-dotnet-deploy-webjobs) ou du [SDK Azure WebJobs](/azure/app-service/webjobs-sdk-get-started), ou en les copiant directement aux emplacements suivants :
  - Pour une exécution déclenchée : site/wwwroot/app_data/jobs/triggered/{job name}
  - Pour une exécution en continu : site/wwwroot/app_data/jobs/continuous/{job name}
- **Journalisation** : la commande Console.Out est traitée (marquée) comme étant de type INFO. La commande Console.Error est traitée comme étant de type ERROR. Vous pouvez accéder aux données de surveillance et de diagnostic à l’aide du portail Azure. Vous pouvez télécharger les fichiers journaux directement à partir du site. Ces éléments sont sauvegardés aux emplacements suivants :
  - Pour une exécution déclenchée : Vfs/data/jobs/triggered/jobName
  - Pour une exécution en continu : Vfs/data/jobs/continuous/jobName
- **Configuration** : vous pouvez configurer des tâches web à l’aide du portail, de l’API REST et de PowerShell. Utilisez un fichier de configuration nommé settings.job, situé dans le même répertoire racine que le script de travail, pour fournir des informations sur la configuration d’un travail. Par exemple : 
  - { "stopping_wait_time": 60 }
  - { "is_singleton": true }

#### <a name="considerations"></a>Considérations

- Par défaut, WebJobs est mis à l’échelle par rapport à l’application web. Toutefois, vous pouvez configurer les tâches pour qu’elles s’exécutent sur une instance unique, en définissant la propriété de configuration **is_singleton** sur la valeur **true**. Les tâches web d’une instance unique sont utiles pour les tâches que vous ne voulez pas mettre à l’échelle ou exécuter en tant qu’instances multiples simultanées (par exemple, la réindexation, l’analyse des données et autres tâches similaires).
- Pour réduire l’impact des travaux sur les performances de l’application web, envisagez de créer une instance vide de l’application web Azure dans un nouveau plan de service d’application, pour y héberger les tâches web de longue durée ou nécessitant beaucoup de ressources.

### <a name="azure-virtual-machines"></a>Machines virtuelles Azure

Vous pouvez implémenter des tâches en arrière-plan de façon à ce qu’elles ne soient pas déployées sur Azure Web Apps, ou parce que ce n’est peut-être pas pratique. Les services Windows, ainsi que les utilitaires et programmes exécutables tiers, en sont des exemples typiques. Il peut également s’agir des programmes écrits pour un environnement d’exécution différent de celui qui héberge l’application. C’est, par exemple, un programme Unix ou Linux que vous souhaitez exécuter à partir d’une application Windows ou .NET Vous avez le choix entre plusieurs systèmes d’exploitation pour une machine virtuelle Azure. Vous pouvez ensuite exécuter votre service ou programme exécutable sur cette machine virtuelle.

Pour savoir quand utiliser Virtual Machines, consultez [Comparaison entre Azure App Service, Azure Cloud Services et Azure Virtual Machines](/azure/app-service-web/choose-web-site-cloud-service-vm/). Pour plus d’informations sur les options disponibles pour les machines virtuelles, consultez [Tailles des machines virtuelles dans Azure](/azure/virtual-machines/windows/sizes). Pour plus d’informations sur les systèmes d’exploitation et les images préconfigurées disponibles pour Virtual Machines, consultez le [Marketplace de machines virtuelles Azure](https://azure.microsoft.com/gallery/virtual-machines/).

Pour lancer la tâche en arrière-plan dans un ordinateur virtuel distinct, vous disposez d’un éventail d’options :

- Vous pouvez exécuter la tâche à la demande directement à partir de votre application, en envoyant une demande à un point de terminaison exposé par la tâche. Cette action transmet toutes les données dont la tâche a besoin. Le point de terminaison appelle la tâche.
- Vous pouvez configurer la tâche pour qu’elle s’exécute selon une planification précise, à l’aide d’un planificateur ou d’un minuteur fourni par le système d’exploitation choisi. Par exemple, sur Windows, utilisez le Planificateur de tâches de Windows pour exécuter des scripts et des tâches. Ou, si vous avez installé SQL Server sur la machine virtuelle, utilisez SQL Server Agent pour faire la même opération.
- Vous pouvez utiliser Microsoft Azure Scheduler pour lancer la tâche en ajoutant un message à une file d’attente sur laquelle la tâche est à l’écoute, ou en envoyant une requête à une API exposée par la tâche.

Consultez la section relative aux [déclencheurs](#triggers) (ci-dessus) pour en savoir plus sur le lancement de tâches en arrière-plan.

<!-- markdownlint-disable MD024 -->

#### <a name="considerations"></a>Considérations

<!-- markdownlint-enable MD024 -->

Si vous envisagez de déployer des tâches en arrière-plan sur une machine virtuelle Azure, tenez compte des points suivants :

- L’hébergement de tâches en arrière-plan sur une machine virtuelle Azure distincte vous permet une certaine flexibilité et vous offre un contrôle précis sur l’initiation, l’exécution, la planification et l’allocation des ressources. Toutefois, cela augmente le coût d’exécution si un ordinateur virtuel doit être déployé uniquement pour exécuter des tâches en arrière-plan.
- Il n’existe pas d’outil de suivi des tâches dans le portail Azure, ni de fonctionnalité de redémarrage automatique des tâches ayant échoué, mais vous pouvez surveiller l’état de base de la machine virtuelle et le gérer au moyen des [applets de commande Azure Resource Manager](https://msdn.microsoft.com/library/mt125356.aspx). Toutefois, il n’existe aucune fonction permettant de contrôler les processus et threads dans les nœuds de calcul. En règle générale, l’utilisation d’une machine virtuelle nécessite des efforts supplémentaires pour implémenter un mécanisme afin de collecter des données d’instrumentation dans la tâche et de rassembler les données du système d’exploitation sur l’ordinateur virtuel. Pour ce faire, essayez d’utiliser le composant [System Center Management Pack pour Azure](https://www.microsoft.com/download/details.aspx?id=50013).
- Vous pouvez envisager de créer des sondes de surveillance exposées via des points de terminaison HTTP. Le code de ces sondes peut effectuer des contrôles d’intégrité, collecter des statistiques et des informations opérationnelles ou collecter les informations sur les erreurs et les renvoyer à une application de gestion. Pour plus d’informations, consultez [Modèle Supervision de point de terminaison d’intégrité](../patterns/health-endpoint-monitoring.md).

Pour plus d'informations, consultez les pages suivantes :

- [Machines virtuelles](https://azure.microsoft.com/services/virtual-machines/)
- [FAQ sur les machines virtuelles Azure](/azure/virtual-machines/virtual-machines-linux-classic-faq?toc=%2fazure%2fvirtual-machines%2flinux%2fclassic%2ftoc.json)

### <a name="azure-batch"></a>Azure Batch

Envisagez d’utiliser [Azure Batch](/azure/batch/) si vous devez exécuter d’importantes charges de travail de calcul haute performance parallèles sur des dizaines, des centaines ou des milliers de machines virtuelles.

Le service Batch approvisionne les machines virtuelles, leur affecte des tâches, puis exécute ces dernières et surveille l’état d’avancement de l’exécution. Batch peut également augmenter automatiquement la taille des machines virtuelles afin de traiter la charge de travail. Enfin, Batch offre une fonctionnalité de planification des travaux. Azure Batch prend en charge les machines virtuelles Linux et Windows.

<!-- markdownlint-disable MD024 -->

#### <a name="considerations"></a>Considérations

<!-- markdownlint-enable MD024 -->

Le service Batch offre de bons résultats avec les charges de travail intrinsèquement parallèles. Il peut également effectuer des calculs parallèles avec une opération de réduction à la fin, ou exécuter des [applications MPI (Message Passing Interface)](/azure/batch/batch-mpi) pour les tâches parallèles qui nécessitent une transmission de messages entre les nœuds.

Un travail Azure Batch s’exécute sur un pool de nœuds (machines virtuelles). Une approche possible consiste à allouer un pool uniquement lorsque cela se révèle nécessaire, puis à le supprimer une fois le travail terminé. Cette approche optimise l’utilisation, car les nœuds ne sont pas inactifs, mais le travail doit attendre que des nœuds soient alloués. Une autre possibilité consiste à créer un pool à l’avance. Cette approche réduit le temps nécessaire au démarrage d’un travail, mais peut entraîner l’existence de nœuds inactifs. Pour plus d’informations, consultez la section [Durée de vie de nœud de pool et de calcul](/azure/batch/batch-api-basics#pool-and-compute-node-lifetime).

Pour plus d'informations, consultez les pages suivantes :

- [Présentation d’Azure Batch](/azure/batch/batch-technical-overview)
- [Développer des solutions de calcul parallèles à grande échelle avec Batch](/azure/batch/batch-api-basics)
- [Solutions Batch et HPC pour les charges de travail de calcul à grande échelle](/azure/batch/batch-hpc-solutions)

### <a name="azure-kubernetes-service"></a>Azure Kubernetes Service

Azure Kubernetes Service (AKS) gère votre environnement Kubernetes hébergé, ce qui vous permet de déployer et de gérer facilement des applications en conteneur.

Les conteneurs peuvent se révéler utiles pour l’exécution de travaux en arrière-plan. Voici certains des avantages obtenus :

- Les conteneurs prennent en charge l’hébergement haute densité. Vous pouvez isoler une tâche en arrière-plan dans un conteneur, tout en plaçant plusieurs conteneurs dans chaque machine virtuelle.
- L’orchestrateur de conteneurs gère l’équilibrage de charge interne, la configuration du réseau interne et d’autres tâches de configuration.
- Les conteneurs peuvent être démarrés et arrêtés selon les besoins.
- Le service Azure Container Registry vous permet d’inscrire vos conteneurs à l’intérieur des limites d’Azure. Il offre des avantages en matière de sécurité, de confidentialité et de proximité.

<!-- markdownlint-disable MD024 -->

#### <a name="considerations"></a>Considérations

<!-- markdownlint-enable MD024 -->

- Cette approche nécessite une bonne compréhension de la procédure d’utilisation d’un orchestrateur de conteneurs. Selon les compétences dont dispose votre équipe DevOps, ceci peut ou non constituer un problème.

Pour plus d'informations, consultez les pages suivantes :

- [Vue d’ensemble des conteneurs dans Azure](https://azure.microsoft.com/overview/containers/)

- [Présentation des registres de conteneurs Docker privés](/azure/container-registry/container-registry-intro)

## <a name="partitioning"></a>Partitionnement

Si vous décidez d’inclure des tâches en arrière-plan dans une instance de calcul existante, vous devez déterminer l’effet de cette opération sur les attributs de qualité de l’instance de calcul et de la tâche en arrière-plan elle-même. Ces facteurs vous aident à décider s’il faut colocaliser les tâches avec l’instance de calcul existante, ou les séparer dans une instance de calcul distincte :

- **Disponibilité** : les tâches en arrière-plan n’ont pas nécessairement besoin du même niveau de disponibilité que d’autres parties de l’application, comme c’est le cas pour l’interface utilisateur et d’autres composants intervenant directement dans les interactions des utilisateurs. Par ailleurs, les tâches en arrière-plan peuvent être plus tolérantes envers un problème de latence ou l’échec d’une nouvelle tentative de connexion, voire d’autres facteurs affectant la disponibilité, car les opérations peuvent être mises en file d’attente. Toutefois, la capacité doit être suffisante pour empêcher la sauvegarde de demandes susceptibles de bloquer les files d’attente et d’affecter l’application dans son ensemble.

- **Scalabilité** : les tâches en arrière-plan n’ont souvent pas les mêmes exigences de scalabilité que l’interface utilisateur et les éléments interactifs de l’application. Il peut être nécessaire de mettre l’interface utilisateur à l’échelle pour qu’elle puisse gérer les pics de demande. Les tâches en arrière-plan en attente peuvent alors être exécutées pendant les périodes moins chargées par un nombre plus réduit d’instances de calcul.

- **Résilience** : l’échec d’une instance de calcul qui héberge simplement des tâches en arrière-plan peut ne pas affecter l’application dans son ensemble s’il est possible de mettre en file d’attente ou de reporter les demandes relatives à ces tâches jusqu’à ce que la tâche soit redevenue disponible. Si l’instance de calcul et/ou les tâches peuvent être redémarrées dans un intervalle approprié, l’échec peut être transparent pour les utilisateurs de l’application.

- **Sécurité** : les tâches en arrière-plan peuvent avoir des restrictions ou exigences de sécurité différentes par rapport à l’interface utilisateur ou à d’autres éléments de l’application. En utilisant une instance de calcul distincte, vous pouvez spécifier un environnement de sécurité différent pour les tâches. Vous pouvez également utiliser des modèles comme celui de l’opérateur de contrôle d’appels pour isoler les instances de calcul en arrière-plan de l’interface utilisateur, afin d’optimiser la sécurité et la séparation.

- **Performances** : vous pouvez choisir le type d’instance de calcul des tâches en arrière-plan de façon qu’il réponde précisément aux exigences de ces tâches en termes de performances. Cela peut se traduire par l’utilisation d’une option de calcul moins coûteuse si les tâches ne nécessitent pas les mêmes fonctions de traitement que l’interface utilisateur, ou d’une instance plus volumineuse s’il leur faut davantage de capacité et de ressources.

- **Facilité de gestion** : les tâches en arrière-plan ne suivent pas forcément le même rythme de développement et de déploiement que celui du code de l’application principale ou de l’interface utilisateur. Le fait de déployer une instance de calcul distincte peut permettre de simplifier les mises à jour et le contrôle de version.

- **Coût** : l’ajout d’instances de calcul pour exécuter des tâches en arrière-plan augmente les coûts d’hébergement. Vous devez déterminer soigneusement l’équilibre entre l’ajout de capacité additionnelle et ces coûts supplémentaires.

Pour plus d’informations, consultez [Modèle d’élection du responsable](../patterns/leader-election.md) et [Modèle des consommateurs concurrents](../patterns/competing-consumers.md).

## <a name="conflicts"></a>Conflits

Si vous avez plusieurs instances d’un travail en arrière-plan, il est possible qu’elles soient en concurrence pour l’accès aux ressources et aux services (les bases de données et le stockage, par exemple). Cet accès simultané peut provoquer une contention des ressources et compromettre alors la disponibilité des services et l’intégrité des données dans le stockage. Vous pouvez résoudre une contention de ressources par une approche de verrouillage pessimiste. Cela empêche les instances concurrentes d’une tâche d’accéder simultanément à un service ou d’endommager les données.

Vous pouvez aussi définir des tâches en arrière-plan en tant que singletons, afin qu’une seule instance s’exécute à la fois. Toutefois, si vous choisissez cette solution, vous perdez les avantages d’une configuration à plusieurs instances en termes de fiabilité et de performances. Cela est particulièrement vrai si l’interface utilisateur est capable de garantir l’activité d’au moins une tâche en arrière-plan.

Vous devez impérativement vous assurer que la tâche en arrière-plan peut redémarrer automatiquement et qu’elle a une capacité suffisante pour répondre aux pics de demande. Pour cela, allouez une instance de calcul avec un nombre de ressources suffisant et/ou implémentez un mécanisme de mise en file d’attente capable de stocker les demandes à exécuter ultérieurement (quand la demande sera moins forte).

## <a name="coordination"></a>Coordination

Les tâches en arrière-plan peuvent être complexes et nécessiter le lancement de plusieurs tâches individuelles pour produire un résultat ou remplir toutes les exigences. Dans ces scénarios, on divise souvent la tâche en sous-tâches ou étapes discrètes de plus petite taille, qui peuvent être exécutées par plusieurs consommateurs. Les travaux à plusieurs étapes peuvent être plus performants et plus flexibles, car chaque étape est réutilisable dans plusieurs travaux. Il est également aisé d’ajouter, de supprimer ou de modifier l’ordre de ces étapes.

La coordination de plusieurs étapes ou tâches peut s’avérer délicate, mais vous pouvez vous aider de trois modèles courants pour implémenter une solution :

- **Décomposition d’une tâche en plusieurs étapes réutilisables**. Une application a parfois besoin d’exécuter une série de tâches plus ou moins complexes sur les informations qu’elle traite. Il existe une approche simple, mais assez rigide, permettant d’implémenter cette application. Elle consiste à effectuer ce traitement comme un module monolithique. Toutefois, cette approche est susceptible de réduire les opportunités de refactorisation, d’optimisation ou de réutilisation du code, si certaines parties du même processus de traitement doivent être utilisées à un autre emplacement de l’application. Pour plus d’informations, consultez [Modèle de canaux et de filtres](../patterns/pipes-and-filters.md).

- **Gestion de l’exécution des étapes d’une tâche**. Une application peut effectuer des tâches comprenant plusieurs étapes, dont certaines nécessitent l’utilisation de services distants ou l’accès à des ressources distantes. Les étapes peuvent être indépendantes les unes des autres, mais elles restent orchestrées par la logique d’application qui implémente la tâche. Pour plus d’informations, consultez [Modèle de superviseur de l’agent du planificateur](../patterns/scheduler-agent-supervisor.md).

- **Gestion de la récupération des étapes en échec d’une tâche**. Une application peut avoir besoin d’annuler le travail effectué par une série d’étapes (qui définissent une opération cohérente au final) si une ou plusieurs de ces étapes échouent. Pour plus d’informations, consultez [Modèle de transaction de compensation](../patterns/compensating-transaction.md).

## <a name="resiliency-considerations"></a>Remarques relatives à la résilience

Les tâches en arrière-plan doivent être assez résilientes pour pouvoir fournir des services fiables à l’application. Quand vous planifiez et créez des tâches en arrière-plan, tenez compte des points suivants :

- Les tâches en arrière-plan doivent être en mesure de gérer correctement les redémarrages, sans corrompre les données ni introduire d’incohérences dans l’application. Pour les tâches de longue durée ou à plusieurs étapes, envisagez d’utiliser des *points de contrôle*, en enregistrant l’état des travaux dans le stockage persistant ou en tant que messages dans une file d’attente, le cas échéant. Par exemple, vous pouvez conserver les informations d’état dans un message placé en file d’attente, et mettre à jour ces informations de manière incrémentielle en parallèle avec la progression de la tâche. Cette tâche peut ainsi être traitée à partir du dernier point de contrôle au lieu d’être réexécutée depuis le début. Lorsque vous utilisez des files d’attente Azure Service Bus, vous pouvez recourir à des sessions de messages pour activer le même scénario. Les sessions vous permettent d’enregistrer et de récupérer l’état de traitement de l’application (à l’aide des méthodes [SetState](/dotnet/api/microsoft.servicebus.messaging.messagesession.setstate?view=azureservicebus-4.0.0) et [GetState](/dotnet/api/microsoft.servicebus.messaging.messagesession.getstate?view=azureservicebus-4.0.0)). Pour plus d’informations sur la conception de processus et workflows à plusieurs étapes fiables, consultez [Modèle de superviseur de l’agent du planificateur](../patterns/scheduler-agent-supervisor.md).

- Quand vous utilisez des files d’attente pour communiquer avec les tâches en arrière-plan, ces files peuvent jouer le rôle de mémoire tampon pour stocker les demandes qui sont envoyées aux tâches à un moment où l’application connaît une surcharge. Ainsi, les tâches peuvent se mettre au même niveau que l’interface utilisateur lorsque l’activité baisse. Cela signifie également que les redémarrages ne bloqueront pas l’interface utilisateur. Pour plus d’informations, consultez [Modèle de nivellement de charge basé sur une file d’attente](../patterns/queue-based-load-leveling.md). Si certaines tâches sont plus importantes que d’autres, vous pouvez envisager d’implémenter le [modèle de file d’attente avec un ordre de priorité](../patterns/priority-queue.md) pour vous assurer que ces tâches s’exécutent avant les autres.

- Les tâches en arrière-plan qui sont lancées par des messages ou qui traitent des messages doivent être conçues pour gérer les incohérences, comme les messages qui arrivent dans le désordre, les messages qui entraînent une erreur de manière récurrente (souvent appelés *messages incohérents*) et les messages qui sont remis plusieurs fois. Tenez compte des éléments suivants :

  - Les messages devant être traités dans un ordre spécifique, comme ceux qui modifient les données sur la base de la valeur des données existante (par exemple, en ajoutant une valeur à une valeur existante), risquent de ne pas arriver dans le même ordre que lors de leur envoi. Ils peuvent aussi être gérés par différentes instances d’une tâche en arrière-plan et selon un ordre différent en raison des charges variables sur chaque instance. Les messages devant être traités dans un ordre spécifique doivent inclure un numéro, une clé ou tout autre indicateur de séquence que les tâches en arrière-plan peuvent utiliser pour garantir leur traitement dans l’ordre adéquat. Si vous utilisez Microsoft Azure Service Bus, vous pouvez utiliser des sessions de messagerie afin de garantir un ordre de livraison adéquat. Toutefois, la conception d’un processus ne tenant pas compte de l’ordre des messages, quand cela possible, est généralement plus efficace.

  - En règle générale, une tâche en arrière-plan obtient un aperçu des messages de la file d’attente, ce qui les masque aux autres consommateurs de messages pendant un temps. Elle supprime ensuite les messages dont le traitement est terminé. Si une tâche en arrière-plan échoue à traiter un message, ce dernier réapparaît dans la file d’attente après expiration du délai de l’aperçu. Le message sera traité par une autre instance de la tâche ou lors du prochain cycle de traitement de cette instance. Si ce message provoque une erreur répétée dans le consommateur, il bloque la tâche, la file d’attente, voire l’application elle-même lorsque la file d’attente est saturée. Pour cette raison, il est essentiel de détecter et de supprimer les messages incohérents de la file d’attente. Si vous utilisez Microsoft Azure Service Bus, vous pouvez déplacer les messages entraînant une erreur (manuellement ou automatiquement) vers une file d’attente de lettres mortes associée.

  - Les files d’attente bénéficient nécessairement *d’au moins un* mécanisme de livraison, mais elles peuvent remettre le même message plusieurs fois. En outre, si une tâche en arrière-plan échoue après avoir traité un message, mais avant de l’avoir supprimé de la file d’attente, ce message pourra à nouveau être traité. Les tâches en arrière-plan doivent être idempotentes : cela signifie que le traitement d’un même message plusieurs fois n’entraîne aucune erreur ni aucune incohérence dans les données de l’application. Par nature, certaines opérations sont idempotentes (comme la définition d’une valeur stockée sur une nouvelle valeur spécifique). Toutefois, si, par exemple, vous ajoutez une valeur à une valeur stockée existante sans vérifier que cette dernière n’a pas été modifiée depuis le moment où le message a été envoyé pour la première fois, des incohérences apparaîtront. Les files d’attente Microsoft Azure Service Bus peuvent être configurées pour supprimer automatiquement les messages en double.

  - Certains systèmes de messagerie, comme les files d’attente de stockage Microsoft Azure et les files Azure Service Bus, prennent en charge une propriété « de-queue count », qui indique le nombre de lectures d’un message depuis la file d’attente. Cette propriété peut être utile lors du traitement de messages incohérents et répétés. Pour en savoir plus, voir [Primer de messagerie asynchrone](https://msdn.microsoft.com/library/dn589781.aspx) et [Modèles d’idempotence](https://blog.jonathanoliver.com/idempotency-patterns/).

## <a name="scaling-and-performance-considerations"></a>Remarques relatives à la mise à l’échelle et aux performances

Les tâches en arrière-plan doivent offrir des performances suffisantes pour éviter de bloquer l’application ou entraîner des incohérences, suite à une opération en retard lorsque le système est soumis à une charge. En général, vous améliorez les performances lorsque vous mettez à l’échelle les instances de calcul qui hébergent les tâches en arrière-plan. Quand vous planifiez et créez des tâches en arrière-plan, tenez compte des points suivants en matière de performances et d’extensibilité :

- Azure prend en charge la mise à l’échelle automatique (augmentation et diminution de la taille des instances) en fonction de la demande et de la charge actuelles, ou selon une planification prédéfinie, pour Web Apps et les déploiements hébergés par les machines virtuelles. Cette fonctionnalité permet de garantir que l’application dans son ensemble offre des performances acceptables, tout en réduisant les coûts d’exécution.

- Si les tâches en arrière-plan présentent des performances différentes de celles d’autres éléments de l’application (par exemple, l’interface utilisateur ou des composants comme la couche d’accès aux données), l’hébergement des tâches en arrière-plan dans un service de calcul distinct permet aux tâches en arrière-plan et à l’interface utilisateur d’être mis à l’échelle de manière indépendante, afin de gérer la charge. Si plusieurs tâches en arrière-plan présentent des performances très différentes, envisagez de les séparer et de mettre chaque type de tâches à l’échelle de manière indépendante. Néanmoins, veuillez noter que cela peut s’avérer plus coûteux.

- Une simple mise à l’échelle des ressources de calcul peut se révéler insuffisante pour empêcher une baisse des performances en cas de charge élevée. Vous devrez peut-être aussi mettre à l’échelle les files d’attente de stockage et d’autres ressources pour empêcher la formation d’un goulot d’étranglement au niveau d’un point donné de la chaîne de traitement. Tenez également compte d’autres limitations comme le débit maximal du stockage et d’autres services sur lesquels reposent l’application et les tâches en arrière-plan.

- La conception des tâches en arrière-plan doit prévoir une mise à l’échelle. Ainsi, elles doivent être en mesure de détecter, de manière dynamique, le nombre de files d’attente de stockage utilisées, afin de pouvoir écouter ou envoyer les messages à la file d’attente appropriée.

- Par défaut, les tâches web sont mises à l’échelle en fonction de l’instance d’application web Azure associée. Cependant, si vous souhaitez qu’une tâche web s’exécute uniquement en tant qu’instance unique, vous pouvez créer un fichier Settings.job qui contient les données JSON **{ "is_singleton": true }**. Cela oblige Azure à exécuter une seule instance de la tâche web, même s’il existe plusieurs instances de l’application web associée. Cela peut être utile pour les tâches planifiées devant s’exécuter en tant qu’instances uniques.

## <a name="related-patterns"></a>Modèles associés

- [Recommandations en matière de partitionnement du calcul](https://msdn.microsoft.com/library/dn589773.aspx)
