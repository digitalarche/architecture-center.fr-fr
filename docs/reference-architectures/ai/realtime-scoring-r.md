---
title: Scoring en temps réel des modèles Machine Learning R
description: Implémentez un service de prédiction en temps réel en R en exécutant Machine Learning Server dans Azure Kubernetes Service (AKS).
author: njray
ms.date: 12/12/2018
ms.custom: azcat-ai
ms.openlocfilehash: 6f3447d1dcab801ccdaf4cf88611725cc00eb68d
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/08/2019
ms.locfileid: "54112275"
---
# <a name="real-time-scoring-of-r-machine-learning-models"></a>Scoring en temps réel des modèles Machine Learning R

Cette architecture de référence montre comment implémenter un service de prédiction (synchrone) en temps réel en R à l’aide de Microsoft Machine Learning Server exécuté dans Azure Kubernetes Service (AKS). Cette architecture est conçue pour être générique et pour s’adapter à n’importe quel modèle prédictif créé en R que vous souhaitez déployer en tant que service en temps réel. **[Déployez cette solution][github]**.

## <a name="architecture"></a>Architecture

![Scoring en temps réel des modèles Machine Learning R sur Azure][0]

Cette architecture de référence adopte une approche basée sur le conteneur. Une image Docker est générée contenant R, ainsi que les divers artefacts nécessaires pour scorer les nouvelles données. Ces derniers incluent l’objet de modèle lui-même et un script de scoring. Cette image est envoyée (push) à un registre Docker hébergé dans Azure, puis déployée sur un cluster Kubernetes, également dans Azure.

L’architecture de ce workflow inclut les composants suivants.

- **[Azure Container Registry][acr]** est utilisé pour stocker les images de ce workflow. Les registres créés avec Container Registry peuvent être gérés par le biais du client et de l’[API Docker Registry V2][docker] standard.

- **[Azure Kubernetes Service][aks]** est utilisé pour héberger le déploiement et le service. Les clusters créés avec AKS peuvent être gérés à l’aide du client et de l’[API Kubernetes][k-api] (kubectl) standard.

- **[Microsoft Machine Learning Server][mmls]** est utilisé pour définir l’API REST du service et inclut l’[opérationnalisation du modèle][operationalization]. Ce processus de serveur web orienté service écoute les demandes, qui sont ensuite remises à d’autres processus en arrière-plan qui exécutent le code R réel pour générer les résultats. Tous ces processus s’exécutent sur un seul nœud dans cette configuration, laquelle est wrappée dans un conteneur. Pour plus d’informations sur l’utilisation de ce service en dehors d’un environnement de test ou de développement, contactez votre représentant Microsoft.

## <a name="performance-considerations"></a>Considérations relatives aux performances

Les charges de travail de machine learning ont tendance à utiliser beaucoup de ressources, à la fois lors de l’entraînement et du scoring de nouvelles données. En règle générale, évitez d’exécuter plusieurs processus de scoring par cœur. Machine Learning Server vous permet de définir le nombre de processus R s’exécutant dans chaque conteneur. La valeur par défaut s’élève à cinq processus. Lors de la création d’un modèle relativement simple, comme une régression linéaire avec un petit nombre de variables ou un petit arbre de décision, vous pouvez augmenter le nombre de processus. Supervisez la charge du processeur sur vos nœuds de cluster pour déterminer la limite appropriée du nombre de conteneurs.

Un cluster compatible GPU peut accélérer certains types de charges de travail, en particulier les modèles d’apprentissage profond. Toutes les charges de travail ne peuvent pas exploiter les GPU &mdash; uniquement celles qui font une utilisation intensive de l’algèbre de la matrice. Par exemple, les modèles basés sur une arborescence, notamment les forêts aléatoires et les modèles de boosting, ne dérivent généralement aucun avantage des GPU.

Certains types de modèles comme les forêts aléatoires sont massivement parallélisables sur des CPU. Le cas échéant, accélérez le scoring d’une seule demande en distribuant la charge de travail sur plusieurs cœurs. Toutefois, cette opération réduit votre capacité à traiter plusieurs demandes de scoring étant en fonction d’une taille fixe de cluster.

En général, les modèles R open source stockent toutes leurs données en mémoire. Veillez donc à ce que vos nœuds disposent de suffisamment de mémoire pour prendre en charge les processus que vous envisagez d’exécuter simultanément. Si vous utilisez Machine Learning Server pour ajuster vos modèles, utilisez les bibliothèques capables de traiter des données sur le disque, plutôt que de tout lire dans la mémoire. Les besoins en mémoire peuvent ainsi s’en retrouver considérablement réduits. Que vous utilisiez Machine Learning Server ou R open source, supervisez vos nœuds pour vous assurer que vos processus de scoring ne sont pas sous-alimentés en mémoire.

## <a name="security-considerations"></a>Considérations relatives à la sécurité

### <a name="network-encryption"></a>Chiffrement réseau

Dans cette architecture de référence, HTTPS est activé pour la communication avec le cluster et un certificat intermédiaire [Let’s Encrypt][encrypt] est utilisé. À des fins de production, remplacez-le par votre propre certificat issu d’une autorité de signature appropriée.

### <a name="authentication-and-authorization"></a>Authentification et autorisation

L’[opérationnalisation du modèle][operationalization] Machine Learning Server nécessite l’authentification des demandes de scoring. Dans ce déploiement, un nom d’utilisateur et un mot de passe sont utilisés. Dans un environnement d’entreprise, vous pouvez activer l’authentification avec [Azure Active Directory][AAD] ou créer un front-end distinct avec [Gestion des API Azure][API].

Pour que l’opérationnalisation du modèle fonctionne correctement avec Machine Learning Server sur des conteneurs, vous devez installer un certificat JSON Web Token (JWT). Ce déploiement utilise un certificat fourni par Microsoft. Dans un environnement de production, fournissez le vôtre.

Pour le trafic entre Container Registry et AKS, envisagez d’activer le [contrôle d’accès en fonction du rôle][rbac] (RBAC) pour limiter les privilèges d’accès à ceux qui sont nécessaires.

### <a name="separate-storage"></a>Stockage distinct

Cette architecture de référence regroupe l’application (R) et les données (objet de modèle et script de scoring) dans une seule image. Dans certains cas, il peut être plus judicieux de les séparer. Vous pouvez placer les données du modèle et le code dans un [stockage][storage] d’objet blob ou de fichier Azure, puis les récupérer au moment de l’initialisation du conteneur. Dans ce cas, veillez à ce que le compte de stockage soit défini pour autoriser uniquement un accès authentifié et exiger HTTPS.

## <a name="monitoring-and-logging-considerations"></a>Supervision et enregistrement des considérations

Utilisez le [tableau de bord Kubernetes][dashboard] pour superviser l’état général de votre cluster AKS. Consultez le panneau de présentation du cluster dans le portail Azure pour plus d’informations. Les ressources [GitHub][github] montrent également comment afficher le tableau de bord à partir de R.

Bien que le tableau de bord vous donne un aperçu de l’intégrité globale de votre cluster, il est également important de suivre l’état des conteneurs individuels. Pour cela, activez [Azure Monitor Insights][monitor] à partir du panneau de présentation du cluster dans le portail Azure ou consultez [Azure Monitor pour les conteneurs][monitor-containers] (en préversion).

## <a name="cost-considerations"></a>Considérations relatives au coût

Machine Learning Server est concédé sous licence par cœur et tous les cœurs inclus dans le cluster qui exécutera Machine Learning Server sont comptabilisés. Si vous êtes client Machine Learning Server ou Microsoft SQL Server professionnel, contactez votre représentant Microsoft pour plus d’informations tarifaires.

Une alternative open source à Machine Learning Server est [Plumber][plumber], un package R qui transforme votre code en API REST. Plumber est moins complet que Machine Learning Server. Par exemple, par défaut, il n’inclut pas de fonctionnalités pour l’authentification des demandes. Si vous utilisez Plumber, il est recommandé d’activer [Gestion des API Azure][API] pour gérer les détails de l’authentification.

En plus des licences, la principale considération en matière de coûts est liée aux ressources de calcul du cluster Kubernetes. Le cluster doit être suffisamment grand pour gérer le volume de demandes attendu aux heures de pointe, mais cette approche laisse les ressources inactives aux autres moments. Pour limiter l’impact des ressources inactives, activez la [mise à l’échelle automatique horizontale][autoscaler] pour le cluster à l’aide de l’outil kubectl. Ou utilisez la [mise à l’échelle automatique du cluster][cluster-autoscaler] AKS.

## <a name="deploy-the-solution"></a>Déployer la solution

L’implémentation de référence de cette architecture est disponible sur [GitHub][github]. Suivez les étapes qui y sont décrites pour déployer un modèle prédictif simple en tant que service.

<!-- links -->
[AAD]: /azure/active-directory/fundamentals/active-directory-whatis
[API]: /azure/api-management/api-management-key-concepts
[ACR]: /azure/container-registry/container-registry-intro
[AKS]: /azure/aks/intro-kubernetes
[autoscaler]: https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/
[cluster-autoscaler]: /azure/aks/autoscaler
[monitor]: /azure/monitoring/monitoring-container-insights-overview
[dashboard]: /azure/aks/kubernetes-dashboard
[docker]: https://docs.docker.com/registry/spec/api/
[encrypt]: https://letsencrypt.org/
[gitHub]: https://github.com/Azure/RealtimeRDeployment
[K-API]: https://kubernetes.io/docs/reference/
[MMLS]: /machine-learning-server/what-is-machine-learning-server
[monitor-containers]: /azure/azure-monitor/insights/container-insights-overview
[operationalization]: /machine-learning-server/what-is-operationalization
[plumber]: https://www.rplumber.io
[RBAC]: /azure/role-based-access-control/overview
[storage]: /azure/storage/common/storage-introduction
[0]: ./_images/realtime-scoring-r.png
