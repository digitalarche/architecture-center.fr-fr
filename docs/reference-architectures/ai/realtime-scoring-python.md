---
title: Scoring en temps réel des modèles de Python
titleSuffix: Azure Reference Architectures
description: Cette architecture de référence montre comment déployer des modèles Python en tant que services web sur Azure pour élaborer des prédictions en temps réel.
author: msalvaris
ms.date: 01/28/2019
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: azcat-ai
ms.openlocfilehash: ba2d9a295e5a231f0ffca9e3cf2d53ace4deddfe
ms.sourcegitcommit: 1ee873aaf40010eb2a38314ac56974bc9e227736
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/28/2019
ms.locfileid: "55141030"
---
# <a name="real-time-scoring-of-python-scikit-learn-and-deep-learning-models-on-azure"></a>Scoring en temps réel des modèles Python Scikit-Learn et Apprentissage profond sur Azure

Cette architecture de référence montre comment déployer des modèles Python en tant que services web pour élaborer des prédictions en temps réel avec le service Azure Machine Learning. Deux scénarios sont couverts : le déploiement de modèles Python standards et les exigences spécifiques du déploiement de modèles d’apprentissage profond. Les deux scénarios utilisent l’architecture illustrée.

Deux implémentations de référence pour cette architecture sont disponibles sur GitHub, une pour [les modèles Python standards][github-python] et l’autre pour [les modèles d’apprentissage profond][github-dl].

![Diagramme d’architecture pour le scoring en temps réel des modèles Python sur Azure](./_images/python-model-architecture.png)

## <a name="scenarios"></a>Scénarios

Les implémentations de référence montrent deux scénarios à l’aide de cette architecture.

**Scénario 1 : Correspondance du Forum aux questions**. Ce scénario montre comment déployer un modèle de correspondance des questions fréquentes (FAQ) comme un service web pour fournir des prédictions pour les questions de l’utilisateur. Pour ce scénario, la mention « Entrée de données » dans le diagramme d’architecture fait référence aux chaînes de texte contenant les questions de l’utilisateur à faire correspondre avec une liste des questions fréquentes. Ce scénario est conçu pour la bibliothèque de Machine Learning [scikit-learn][scikit] pour Python, mais il peut être étendu à n’importe quel scénario utilisant des modèles Python pour effectuer des prédictions en temps réel.

Ce scénario utilise un sous-ensemble de données de question Stack Overflow qui comprend des questions d’origine marquées comme JavaScript, leurs questions dupliquées et leurs réponses. Il effectue l’apprentissage d’un pipeline scikit-learn pour prédire la probabilité de correspondance d’une question dupliquée avec chacune des questions d’origine. Ces prédictions sont faites en temps réel à l’aide d’un point de terminaison API REST.

Le flux d’application pour cette architecture est le suivant :

1. Le modèle entraîné est inscrit dans le registre des modèles Machine Learning.
2. Le service Machine Learning crée une image Docker qui inclut le modèle et le script de scoring.
3. Machine Learning déploie l’image de scoring sur Azure Kubernetes Service (AKS) comme service web.
4. Le client envoie une requête HTTP POST avec les données encodées de la question.
5. Le service web créé par Machine Learning extrait la question de la requête.
6. La question est envoyée au modèle de pipeline Scikit-learn pour la caractérisation et le scoring. 
7. Les questions fréquentes correspondantes sont retournées avec leur score au client.

Voici une capture d’écran de l’exemple d’application qui consomme les résultats :

![Capture d’écran de l’exemple d’application](./_images/python-faq-matches.png)

**Scénario 2 : Classification d’image**. Ce scénario montre comment déployer un modèle de réseau de neurones convolutifs (CNN) comme un service web pour fournir des prévisions sur des images. Pour ce scénario, la mention « Entrée de données » dans le diagramme d’architecture fait référence aux fichiers d’image. Les CNN sont très efficaces dans la vision par ordinateur pour des tâches telles que la classification d’images et la détection d’objets. Ce scénario est conçu pour les infrastructures TensorFlow, Keras (avec le serveur principal TensorFlow) et PyTorch. Toutefois, il peut être généralisé aux scénarios utilisant des modèles d’apprentissage profond pour effectuer des prédictions en temps réel.

Ce scénario utilise un modèle ResNet-152 préentraîné formé sur un jeu de données ImageNet-1 K (1 000 classes) pour prédire la catégorie (voir la figure ci-dessous) à laquelle appartient une image. Ces prédictions sont faites en temps réel à l’aide d’un point de terminaison API REST.

![Exemple de prédictions](./_images/python-example-predictions.png)

Le flux d’application pour le modèle d’apprentissage profond est comme suit :

1. Le modèle d’apprentissage profond (deep learning) est inscrit dans le registre des modèles Machine Learning.
2. Le service Machine Learning crée une image Docker qui inclut le modèle et le script de scoring.
3. Machine Learning déploie l’image de scoring sur Azure Kubernetes Service (AKS) comme service web.
4. Le client envoie une requête HTTP POST avec les données encodées de l’image.
5. Le service web créé par Machine Learning prétraite les données de l’image et les envoie au modèle pour le calcul du score. 
6. Les catégories prédites sont retournées avec leur score au client.

## <a name="architecture"></a>Architecture

Cette architecture est constituée des composants suivants.

Le **[service Azure Machine Learning][aml]** est un service cloud utilisé pour entraîner, déployer, automatiser et gérer des modèles Machine Learning, le tout à l’échelle étendue fournie par le cloud. Dans cette architecture, il est utilisé pour gérer le déploiement de modèles ainsi que pour l’authentification, le routage et l’équilibrage de charge du service web.

**[Machine virtuelle][vm]**. La machine virtuelle est affichée comme un exemple d’appareil &mdash; local ou dans le cloud &mdash; pouvant envoyer une requête HTTP.

**[Azure Kubernetes Service][aks]** (AKS) est utilisé pour déployer l’application au sein d’un cluster Kubernetes. AKS simplifie le déploiement et les opérations de Kubernetes. Le cluster peut être configuré à l’aide de machines virtuelles compatibles processeur uniquement pour des modèles Python standards ou des machines virtuelles compatibles GPU pour les modèles d’apprentissage profond.

**[Azure Container Registry] [acr]** permet le stockage des images pour tous les types de déploiements de conteneurs Docker, notamment DC/OS, Docker Swarm et Kubernetes. Les images de scoring sont déployées en tant que conteneurs sur Azure Kubernetes Service et utilisées pour exécuter le script de scoring. L’image utilisée ici est créée par Machine Learning à partir du modèle entraîné et du script de scoring, puis elle est envoyée à Azure Container Registry.

## <a name="performance-considerations"></a>Considérations relatives aux performances

Pour les architectures de scoring en temps réel, les performances de débit constituent un facteur important. Pour les modèles Python standards, il est généralement admis que les processeurs sont suffisants pour gérer la charge de travail.

Toutefois pour les charges de travail d’apprentissage profond, où la vitesse constitue un goulot d’étranglement, les GPU offrent généralement de meilleures [performances][gpus-vs-cpus] par rapport aux processeurs. Pour faire correspondre les performances de GPU en utilisant des processeurs, un cluster disposant d’un grand nombre de processeurs est généralement nécessaire.

Vous pouvez utiliser des processeurs pour cette architecture dans les deux cas, mais pour les modèles d’apprentissage profond, les GPU offrent des débits considérablement supérieurs par rapport à un cluster de processeurs d’un coût similaire. AKS prend en charge l’utilisation de GPU, qui est l’un des avantages de l’utilisation d’AKS pour cette architecture. En outre, les déploiements d’apprentissage profond utilisent généralement modèles comportant un grand nombre de paramètres. L’utilisation des GPU empêche la contention de ressources entre le modèle et le service web, ce qui constitue un problème dans les déploiements constitués uniquement de processeurs.

## <a name="scalability-considerations"></a>Considérations relatives à l’extensibilité

Pour les modèles Python standard, où le cluster AKS est provisionné avec des machines virtuelles pourvues seulement de processeurs, faites attention lors du [scale-out du nombre de pods][manually-scale-pods]. L’objectif est d’utiliser pleinement le cluster. L’extensibilité dépend des requêtes du processeur et des limites définies pour les pods. Machine Learning via Kubernetes prend également en charge la [mise à l’échelle automatique des pods][autoscale-pods] en fonction de l’utilisation du processeur ou d’autres métriques. Le [dispositif de mise à l’échelle automatique de cluster][autoscaler] (en préversion) peut mettre à l’échelle les nœuds d’agent en fonction des pods en attente.

Pour des scénarios d’apprentissage profond, à l’aide de machines virtuelles compatibles GPU, les limites de ressources sur les pods sont telles qu’un GPU est attribué à un pod. Selon le type de machine virtuelle utilisé, vous devez [mettre à l’échelle les nœuds du cluster][scale-cluster] pour répondre à la demande du service. Vous pouvez faire cela facilement à l’aide d’Azure CLI et de kubectl.

## <a name="monitoring-and-logging-considerations"></a>Supervision et enregistrement des considérations

### <a name="aks-monitoring"></a>Supervision AKS

Pour visualiser les performances de AKS, utilisez la fonctionnalité [Azure Monitor pour les conteneurs][monitor-containers]. Cela collecte des métriques sur la mémoire et le processeur à partir des contrôleurs, des nœuds et des conteneurs qui sont disponibles dans Kubernetes via l’API Metrics.

Lors du déploiement de votre application, surveillez le cluster AKS pour vous assurer qu’il fonctionne comme prévu, tous les nœuds sont opérationnels, et tous les pods sont en cours d’exécution. Bien que vous puissiez utiliser l’outil en ligne de commande [kubectl][kubectl] pour récupérer l’état du pod, Kubernetes comprend également un tableau de bord web pour la surveillance de l’état du cluster et la gestion de base.

![Capture d’écran du tableau de bord Kubernetes](./_images/python-kubernetes-dashboard.png)

Pour afficher l’état global du cluster et des nœuds, accédez à la section **Nœuds** du tableau de bord Kubernetes. Si un nœud est inactif ou a échoué, vous pouvez afficher les journaux d’erreurs à partir de cette page. De même, accédez aux sections **Pods** et **Déploiements** pour plus d’informations sur le nombre de pods et l’état de votre déploiement.

### <a name="aks-logs"></a>Journaux AKS

AKS enregistre automatiquement tous les stdout/stderr au sein des journaux des pods dans le cluster. Utilisez kubectl pour les voir, de même que les événements et les journaux au niveau du nœud. Pour plus d’informations, consultez les étapes de déploiement.

Utilisez [Azure Monitor pour les conteneurs][monitor-containers] pour collecter des métriques et des journaux par l’intermédiaire d’une version en conteneur de l’agent Log Analytics pour Linux, stocké dans votre espace de travail Log Analytics.

## <a name="security-considerations"></a>Considérations relatives à la sécurité

[Azure Security Center][security-center] vous offre un aperçu global de l’état de toutes vos ressources Azure en termes de sécurité. Il surveille les problèmes potentiels de sécurité et fournit une image complète de la sécurité de votre déploiement, même s’il ne surveille pas les nœuds d’agent AKS. Le Centre de sécurité est configuré pour chaque abonnement Azure. Activez la collecte de données de sécurité comme décrit dans [Intégrer un abonnement Azure à Security Center Standard][get-started]. Une fois la collecte de données activée, le Centre de sécurité analyse automatiquement les machines virtuelles créées dans le cadre de cet abonnement.

**Opérations**. Pour vous connecter à un cluster AKS à l’aide de votre jeton d’authentification Azure Active Directory (Azure AD), configurez AKS pour utiliser Azure AD pour [l’authentification utilisateur][aad-auth]. Les administrateurs de cluster peuvent également configurer le contrôle d’accès en fonction du rôle (RBAC) Kubernetes basé sur une identité utilisateur ou l’appartenance à un groupe de répertoires.

Utilisez le contrôle [RBAC][rbac] pour contrôler l’accès aux ressources Azure que vous déployez. Le contrôle RBAC vous permet d’affecter des rôles d’autorisation aux membres de votre équipe DevOps. Un utilisateur peut être affecté à plusieurs rôles, et vous pouvez créer des rôles personnalisés pour d’autres [autorisations] plus affinées.

**HTTPS**. Comme meilleure pratique de sécurité, l’application doit appliquer le protocole HTTPS et rediriger les requêtes HTTP. Utilisez un [contrôleur d’entrée][ingress-controller] pour déployer un proxy inverse qui arrête le protocole SSL et redirige les requêtes HTTP. Pour plus d’informations, consultez [Créer un contrôleur d’entrée HTTPS dans Azure Kubernetes Service (AKS)][https-ingress].

**Authentification**. Cette solution ne restreint pas l’accès aux points de terminaison. Pour déployer l’architecture dans un environnement d’entreprise, sécurisez les points de terminaison via des clés API et ajoutez une forme d’authentification utilisateur à l’application cliente.

**Registre de conteneurs**. Cette solution utilise Azure Container Registry pour stocker l’image Docker. Le code dont dépend l’application, ainsi que le modèle, sont contenus dans cette image. Les applications d’entreprise doivent utiliser un registre privé pour se prémunir contre l’exécution de code malveillant et pour mieux éviter que les informations contenues dans le conteneur soient compromises.

**Protection DDOS**. Envisagez l’activation de la [Protection DDoS Standard][ddos]. Bien que la protection DDoS soit activée sur la plateforme Azure, la protection DDoS standard offre des capacités d’atténuation des risques spécifiquement adaptées aux ressources de réseau virtuel Azure.

**Journalisation**. Utilisez les meilleures pratiques avant de stocker des données de journal, comme le nettoyage des mots de passe utilisateur et d’autres informations pouvant être utilisées pour commettre une usurpation de sécurité.

## <a name="deployment"></a>Déploiement

Pour déployer cette architecture de référence, suivez les étapes décrites dans les dépôts GitHub :

- [Modèles Python standards][github-python]
- [Modèles d’apprentissage profond][github-dl]

<!-- links -->

[aad-auth]: /azure/aks/aad-integration
[acr]: /azure/container-registry/
[something]: https://kubernetes.io/docs/reference/access-authn-authz/authentication/
[aks]: /azure/aks/intro-kubernetes
[autoscaler]: /azure/aks/autoscaler
[autoscale-pods]: /azure/aks/tutorial-kubernetes-scale#autoscale-pods
[azcopy]: /azure/storage/common/storage-use-azcopy-linux
[ddos]: /azure/virtual-network/ddos-protection-overview
[get-started]: /azure/security-center/security-center-get-started
[github-python]: https://github.com/Microsoft/MLAKSDeployAML
[github-dl]: https://github.com/Microsoft/AKSDeploymentTutorial_AML
[gpus-vs-cpus]: https://azure.microsoft.com/en-us/blog/gpus-vs-cpus-for-deployment-of-deep-learning-models/
[https-ingress]: /azure/aks/ingress-tls
[ingress-controller]: https://kubernetes.io/docs/concepts/services-networking/ingress/
[kubectl]: https://kubernetes.io/docs/tasks/tools/install-kubectl/
[aml]: /azure/machine-learning/service/overview-what-is-azure-ml
[manually-scale-pods]: /azure/aks/tutorial-kubernetes-scale#manually-scale-pods
[monitor-containers]: /azure/monitoring/monitoring-container-insights-overview
[autorisations]: /azure/aks/concepts-identity
[rbac]: /azure/active-directory/role-based-access-control-what-is
[scale-cluster]: /azure/aks/scale-cluster
[scikit]: https://pypi.org/project/scikit-learn/
[security-center]: /azure/security-center/security-center-intro
[vm]: /azure/virtual-machines/
