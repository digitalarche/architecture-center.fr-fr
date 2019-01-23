---
title: Entraînement distribué de modèles Deep Learning sur Azure
description: Cette architecture de référence montre comment procéder à un entraînement distribué de modèles Deep Learning (apprentissage profond) sur plusieurs clusters de machines virtuelles compatibles GPU à l’aide d’Azure Batch AI.
author: njray
ms.date: 01/14/19
ms.custom: azcat-ai
ms.openlocfilehash: 800defeb851f5a31dc730038c3699e1a3d54b923
ms.sourcegitcommit: d5ea427c25f9f7799cc859b99f328739ca2d8c1c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/15/2019
ms.locfileid: "54307756"
---
# <a name="distributed-training-of-deep-learning-models-on-azure"></a>Entraînement distribué de modèles Deep Learning sur Azure

Cette architecture de référence montre comment effectué un entraînement distribué de modèles Deep Learning sur plusieurs clusters de machines virtuelles compatibles GPU. Le scénario est une classification d’images, mais la solution peut être généralisée à d’autres scénarios de deep learning, tels que la segmentation et la détection d’objet.

Une implémentation de référence pour cette architecture est disponible sur [GitHub][github].

![Architecture du deep learning distribué][0]

**Scénario** : La classification d’images est une technique largement appliquée dans la vision par ordinateur et souvent traitée par l’entraînement d’un réseau neuronal convolutif (CNN). Pour les modèles particulièrement grands avec des jeux de données volumineux, le processus d’entraînement peut prendre plusieurs semaines ou mois sur un seul GPU. Dans certains cas, les modèles sont tellement grands qu’il n’est pas possible d’ajuster des tailles de lot raisonnables sur le GPU. L’utilisation de l’entraînement distribué peut alors raccourcir la durée d’entraînement.

Dans ce scénario précis, un [modèle CNN ResNet50][resnet] est entraîné avec [Horovod][horovod] sur le [jeu de données Imagenet][imagenet] et des données synthétiques. L’implémentation de référence montre comment accomplir cette tâche en utilisant trois des frameworks de deep learning les plus connus : TensorFlow, Keras et PyTorch.

Il existe plusieurs façons d’entraîner un modèle Deep Learning de façon distribuée, notamment les approches à parallélisme de données et à parallélisme de modèle s’appuyant sur des mises à jour synchrones ou asynchrones. Actuellement, le scénario le plus courant est le parallélisme de données avec mises à jour synchrones. Cette approche est la plus facile à implémenter et est suffisante pour la plupart des cas d’usage.

Dans l’entraînement distribué à parallélisme de données avec mises à jour synchrones, le modèle est répliqué sur *n* appareils. Un mini-lot d’exemples d’entraînement est divisé en *n* micro-lots. Chaque appareil effectue les propagations vers l’avant et vers l’arrière pour un micro-lot. Quand un appareil a terminé le traitement, il partage les mises à jour avec les autres appareils. Ces valeurs sont utilisées pour calculer les pondérations mises à jour de tout le mini-lot, et ces pondérations sont synchronisées entre les modèles. Ce scénario est décrit dans le dépôt [GitHub][github].

![Entraînement distribué à parallélisme de données][1]

Cette architecture peut également servir pour le parallélisme de modèle et les mises à jour asynchrones. Dans l’entraînement distribué à parallélisme de modèle, le modèle est divisé entre *n* appareils, chaque appareil détenant une partie du modèle. Dans l’implémentation la plus simple, chaque appareil peut contenir une couche du réseau, et les informations sont transmises entre les appareils pendant la propagation vers l’avant et vers l’arrière. Les réseaux neuronaux plus grands peuvent être entraînés de cette façon, mais au détriment des performances, car les appareils s’attendent les uns les autres continuellement pour effectuer la propagation vers l’avant ou vers l’arrière. Certaines techniques avancées tentent de pallier partiellement ce problème au moyen de gradients synthétiques.

Les étapes de l’entraînement sont les suivantes :

1. Créez des scripts prévus pour s’exécuter sur le cluster et entraîner votre modèle, puis transférez-les sur le stockage de fichiers.

1. Écrivez les données dans le stockage Blob.

1. Créez un serveur de fichiers Batch AI et téléchargez les données sur ce serveur depuis le stockage Blob.

1. Créez les conteneurs Docker pour chaque framework de deep learning et transférez-les sur un registre de conteneurs (Docker Hub).

1. Créez un pool Batch AI qui monte également le serveur de fichiers Batch AI.

1. Envoyer des travaux. Chacun extrait l’image Docker et les scripts appropriés.

1. Une fois le travail terminé, écrivez tous les résultats dans le stockage Fichier.

## <a name="architecture"></a>Architecture

Cette architecture est constituée des composants suivants.

**[Azure Batch AI][batch-ai]** joue le rôle central dans cette architecture en effectuant des opérations de scale-up et des scale-down sur les ressources en fonction des besoins. Batch AI est un service qui permet de provisionner et de gérer des clusters de machines virtuelles, de planifier des travaux, de collecter des résultats, de mettre à l’échelle des ressources, de gérer les défaillances et de créer le stockage approprié. Il prend en charge les machines virtuelles compatibles GPU pour les charges de travail de Deep Learning. Un kit SDK Python et une interface de ligne de commande (CLI) sont disponibles pour Batch AI.

> [!NOTE]
> La date du retrait du service Azure Batch AI est fixée au mois de mars 2019 ; ses capacités d’entraînement et de scoring à grande échelle sont désormais disponibles dans [Azure Machine Learning service][amls]. Cette architecture de référence sera bientôt actualisée pour utiliser Machine Learning qui offre une cible de calcul managée appelée [Capacité de calcul Azure Machine Learning][aml-compute] pour l’entraînement, le déploiement et le scoring de modèles Machine Learning.

Le **[stockage Blob][azure-blob]** sert à stocker les données. Ces données sont téléchargées sur un serveur de fichiers Batch AI pendant l’entraînement.

Le service **[Azure Files][files]** est utilisé pour stocker les scripts, journaux et résultats finaux provenant de l’entraînement. Le stockage Fichier fonctionne bien pour le stockage des journaux et des scripts, mais il n’est pas aussi performant que le stockage Blob et ne doit donc pas être utilisé pour les tâches gourmandes en données.

Le **[serveur de fichiers Batch AI][batch-ai-files]** est un partage NFS à un seul nœud servant à stocker les données d’entraînement dans cette architecture. Batch AI crée un partage NFS et le monte sur le cluster. Les serveurs de fichiers Batch AI sont le moyen recommandé pour délivrer des données sur le cluster avec le débit nécessaire.

**[Docker Hub][docker]** est utilisé pour stocker l’image Docker que Batch AI utilise pour exécuter l’entraînement. DockerHub a été choisi pour cette architecture, car il est simple d’utilisation et constitue le référentiel d’images par défaut des utilisateurs de Docker. [Azure Container Registry][acr] peut aussi être utilisé.

## <a name="performance-considerations"></a>Considérations relatives aux performances

Azure fournit quatre [types de machines virtuelles compatibles GPU][gpu] qui conviennent pour l’entraînement des modèles Deep Learning. En termes de prix et de vitesse, elles varient de la plus faible à la plus élevée comme suit :

| **Série Machines virtuelles Azure** | **GPU NVIDIA** |
|---------------------|----------------|
| NC                  | K80            |
| ND                  | P40            |
| NCv2                | P100           |
| NCv3                | V100           |

Nous recommandons d’effectuer un scaling-up de votre entraînement avant de procéder à un scaling-out. Par exemple, essayez un seul V100 avant d’opter pour un cluster de K80.

Le graphique suivant montre les différences de performance entre différents types de GPU, selon les [tests d’évaluation][benchmark] effectués avec TensorFlow et Horovod sur Batch AI. Le graphique renseigne sur le débit de 32 clusters GPU, dans divers modèles, sur des versions MPI et des types de GPU différents. Les modèles ont été implémentés dans TensorFlow 1.9

![Débits des modèles TensorFlow sur les clusters GPU][2]

Chaque série de machines virtuelles affichée dans le précédent tableau comprend une configuration avec InfiniBand. Utilisez les configurations InfiniBand lors de l’exécution de l’entraînement distribué pour accélérer la communication entre les nœuds. InfiniBand augmente également l’efficacité de la mise à l’échelle de l’entraînement pour les frameworks qui peuvent en tirer parti. Pour plus d’informations, consultez la [comparaison des tests d’évaluation][benchmark] Infiniband.

Même si Batch AI peut monter le stockage Blob à l’aide de l’adaptateur [blobfuse][blobfuse], nous ne recommandons pas cette utilisation du stockage Blob pour l’entraînement distribué, car les performances ne sont pas suffisantes pour gérer le débit nécessaire. Déplacez plutôt les données sur un serveur de fichiers Batch AI, comme indiqué dans le schéma de l’architecture.

## <a name="scalability-considerations"></a>Considérations relatives à l’extensibilité

L’efficacité de la mise à l’échelle de l’entraînement distribué est toujours inférieure à 100 % en raison de la charge réseau ; la synchronisation de l’ensemble du modèle entre les appareils se transforme en goulot d’étranglement. Ainsi, l’entraînement distribué est la solution la mieux adaptée pour les grands modèles qui ne peuvent pas être entraînés avec une taille de lot appropriée sur un GPU, ou pour les problèmes qui ne peuvent pas être résolus en distribuant le modèle de manière simple et parallèle.

L’entraînement distribué n’est pas recommandé pour effectuer des recherches d’hyperparamètres. L’efficacité de la mise à l’échelle a une incidence sur les performances, et rend l’approche distribuée moins efficace que l’entraînement de plusieurs configurations de modèle séparément.

Une solution permettant d’augmenter l’efficacité de la mise à l’échelle consiste à accroître la taille de lot. Cette opération doit, toutefois, être effectuée avec précaution, car l’augmentation de la taille de lot sans ajuster les autres paramètres peut nuire aux performances finales du modèle.

## <a name="storage-considerations"></a>Considérations relatives au stockage

Dans l’entraînement de modèles Deep Learning, un aspect souvent négligé est l’endroit où sont stockées les données. Si le stockage est trop lent pour répondre aux exigences des GPU, les performances d’entraînement peuvent se détériorer.

Batch AI prend en charge de nombreuses solutions de stockage. Cette architecture utilise un serveur de fichiers Batch AI parce qu’il offre le meilleur compromis entre la facilité d’utilisation et les performances. Pour de meilleures performances, chargez les données localement. Cette opération peut toutefois s’avérer fastidieuse, car tous les nœuds doivent télécharger les données depuis le stockage Blob, et avec le jeu de données ImageNet, cette opération peut prendre des heures. Le [Stockage Blob Azure Premium][blob] (préversion publique limitée) est une autre bonne solution à prendre en compte.

Ne montez pas le stockage Blob ni le stockage Fichier en tant que magasins de données pour l’entraînement distribué. Ils sont trop lents et gêneront les performances de l’entraînement.

## <a name="security-considerations"></a>Considérations relatives à la sécurité

### <a name="restrict-access-to-azure-blob-storage"></a>Restreindre l’accès au stockage Blob Azure

Cette architecture utilise des [clés de compte de stockage][security-guide] pour accéder au stockage Blob. Pour une protection et un contrôle accrus, envisagez d’utiliser une signature d’accès partagé (SAP). Celle-ci octroie un accès limité aux objets contenus dans le stockage, sans qu’il soit nécessaire de coder en dur les clés de compte ou de les enregistrer en texte clair. L’utilisation d’une signature SAS permet aussi de s’assurer que le compte de stockage obéit à une gouvernance appropriée, et que l’accès est octroyé uniquement aux personnes censées en bénéficier.

Dans les scénarios faisant intervenir des données plus sensibles, veillez à ce que toutes vos clés de stockage soient protégées, car elles octroient un accès complet à toutes les données d’entrée et de sortie de la charge de travail.

### <a name="encrypt-data-at-rest-and-in-motion"></a>Chiffrer les données au repos ou en déplacement

Dans les scénarios qui utilisent des données sensibles, chiffrez les données au repos, autrement dit les données présentes dans le stockage. Chaque fois que des données se déplacent d’un emplacement à l’autre, utilisez SSL pour sécuriser le transfert des données. Pour plus d’informations, consultez le [guide de sécurité du stockage Azure][security-guide].

### <a name="secure-data-in-a-virtual-network"></a>Protéger les données dans un réseau virtuel

Pour les déploiements de production, envisagez de déployer le cluster Batch AI dans un sous-réseau du réseau virtuel que vous spécifiez. Les nœuds de calcul du cluster peuvent ainsi communiquer de façon sécurisée avec d’autres machines virtuelles ou avec un réseau local. Vous pouvez aussi utiliser des [points de terminaison de service][endpoints] avec le stockage Blob pour octroyer un accès à partir d’un réseau virtuel, ou utiliser un système NFS à un seul nœud à l’intérieur du réseau virtuel avec Batch AI.

## <a name="monitoring-considerations"></a>Surveillance - Éléments à prendre en compte

Pendant que vous exécutez votre tâche, il est important de superviser la progression et de vérifier que tout fonctionne comme prévu. Cependant, superviser un cluster de nœuds actifs peut s’avérer ardu.

Les serveurs de fichiers Batch AI peuvent être gérés via le portail Azure ou par le biais de l’interface [Azure CLI][cli] et du kit SDK Python. Pour vous faire une idée de l’état général du cluster, accédez à **Batch AI** dans le portail Azure pour inspecter l’état des nœuds du cluster. Si un nœud est inactif ou si une tâche échoue, les journaux d’erreurs sont enregistrés dans le stockage Blob ; ils sont aussi accessibles dans le portail Azure, sous **Tâches**.

Enrichissez la supervision en connectant les journaux à [Azure Application Insights][ai] ou en exécutant des processus distincts qui demandent l’état du cluster Batch AI et de ses tâches.

Batch AI journalise automatiquement tous les stdout/stderr dans le compte du stockage Blob associé. Utilisez un outil de navigation de stockage, comme l’[Explorateur Stockage Azure][storage-explorer], pour faciliter l’expérience de navigation dans les fichiers journaux.

Il est également possible de diffuser en continu les journaux pour chaque tâche. Pour plus d’informations sur cette option, consultez les étapes de développement sur [GitHub][github].

## <a name="deployment"></a>Déploiement

L’implémentation de référence de cette architecture est disponible sur [GitHub][github]. Suivez les étapes qui y sont décrites pour effectuer un entraînement distribué de modèles Deep Learning sur les clusters de machines virtuelles compatibles GPU.

## <a name="next-steps"></a>Étapes suivantes

La sortie de cette architecture est un modèle entraîné qui est enregistré dans le stockage d’objets blob. Vous pouvez faire fonctionner ce modèle pour le scoring en temps réel ou le scoring par lots. Pour en savoir plus, consultez les architectures de référence suivantes :

- [Scoring en temps réel des modèles Python Scikit-Learn et des modèles Deep Learning sur Azure][real-time-scoring]
- [Scoring par lots dans Azure pour les modèles Deep Learning][batch-scoring]

[0]: ./_images/distributed_dl_architecture.png
[1]: ./_images/distributed_dl_flow.png
[2]: ./_images/distributed_dl_tests.png
[acr]: /azure/container-registry/container-registry-intro
[ai]: /azure/application-insights/app-insights-overview
[aml-compute]: /azure/machine-learning/service/how-to-set-up-training-targets#amlcompute
[amls]: /azure/machine-learning/service/overview-what-is-azure-ml
[azure-blob]: /azure/storage/blobs/storage-blobs-introduction
[batch-ai]: /azure/batch-ai/overview
[batch-ai-files]: /azure/batch-ai/resource-concepts#file-server
[batch-scoring]: /azure/architecture/reference-architectures/ai/batch-scoring-deep-learning
[benchmark]: https://github.com/msalvaris/BatchAIHorovodBenchmark
[blob]: https://azure.microsoft.com/en-gb/blog/introducing-azure-premium-blob-storage-limited-public-preview/
[blobfuse]: https://github.com/Azure/azure-storage-fuse
[cli]: https://github.com/Azure/BatchAI/blob/master/documentation/using-azure-cli-20.md
[docker]: https://hub.docker.com/
[endpoints]: /azure/storage/common/storage-network-security?toc=%2fazure%2fvirtual-network%2ftoc.json#grant-access-from-a-virtual-network
[files]: /azure/storage/files/storage-files-introduction
[github]: https://github.com/Azure/DistributedDeepLearning/
[gpu]: /azure/virtual-machines/windows/sizes-gpu
[horovod]: https://github.com/uber/horovod
[imagenet]: http://www.image-net.org/
[real-time-scoring]: /azure/architecture/reference-architectures/ai/realtime-scoring-python
[resnet]: https://arxiv.org/abs/1512.03385
[security-guide]: /azure/storage/common/storage-security-guide
[storage-explorer]: /azure/vs-azure-tools-storage-manage-with-storage-explorer?tabs=windows
[tutorial]: https://github.com/Azure/DistributedDeepLearning