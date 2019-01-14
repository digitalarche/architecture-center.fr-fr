---
title: Passerelles d’API
description: Passerelles API dans les microservices.
author: MikeWasson
ms.date: 10/23/2018
ms.openlocfilehash: cd13f9a36120bb0093675e1dec683d480e168db5
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/08/2019
ms.locfileid: "54113261"
---
# <a name="designing-microservices-api-gateways"></a>Conception de microservices : Passerelles d’API

Dans une architecture microservices, un client peut interagir avec plusieurs services frontaux. Dans ce cas, comment un client sait-il quels points de terminaison appeler ? Que se passe-t-il lors de l’introduction de nouveaux services ou lorsque des services existants sont refactorisés ? Comment les services gèrent-ils l’arrêt SSL, l’authentification et les autres problèmes ? Une *passerelle d’API* permet de relever ces défis.

![Diagramme d’une passerelle API](./images/gateway.png)

<!-- markdownlint-disable MD026 -->

## <a name="what-is-an-api-gateway"></a>Qu'est-ce qu'une passerelle d'API ?

<!-- markdownlint-enable MD026 -->

Une passerelle d’API se situe entre les clients et les services. Elle agit comme un proxy inverse, en acheminant les requêtes des clients vers les services. Elle peut également effectuer diverses tâches transversales telles que l’authentification, l’arrêt SSL et la limitation du débit. Si vous ne pouvez pas déployer de passerelle, les clients doivent envoyer des requêtes directement aux services frontaux. Toutefois, il existe certains problèmes potentiels relatifs à l’exposition directe des services aux clients :

- Elle peut entraîner un code client complexe. Le client doit effectuer le suivi de plusieurs points de terminaison et gérer les échecs de façon résiliente.
- Elle crée un couplage entre le client et le serveur principal. Le client doit connaître la façon dont les services individuels sont décomposés. Cela rend plus difficiles la maintenance du client et la refactorisation des services.
- Une seule opération peut nécessiter des appels à plusieurs services. Cela peut entraîner plusieurs allers-retours réseau entre le client et le serveur, ajoutant ainsi une latence importante.
- Chaque service public doit gérer des problèmes tels que l’authentification, le SSL et la limitation du débit client.
- Les services doivent exposer un protocole pratique pour le client, comme HTTP ou WebSocket. Cela limite le choix des [protocoles de communication](./interservice-communication.md).
- Les services avec des points de terminaison publics représentent une surface d’attaque potentielle et doivent être renforcés.

Une passerelle permet de résoudre ces problèmes en dissociant les clients des services. Les passerelles peuvent exécuter un certain nombre de fonctions différentes, et vous n’aurez peut-être pas besoin de toutes. Les fonctions peuvent être regroupées dans les modèles de conception suivants :

[Routage de passerelle](../patterns/gateway-routing.md) : utilisez la passerelle comme un proxy inverse pour acheminer les requêtes vers un ou plusieurs services principaux, à l’aide du routage de couche 7. La passerelle fournit un point de terminaison unique pour les clients et permet de découpler les clients des services.

[Agrégation de passerelle](../patterns/gateway-aggregation.md) : utilisez la passerelle pour agréger plusieurs requêtes individuelles dans une requête unique. Ce modèle s’applique lorsqu’une opération requiert des appels à plusieurs services principaux. Le client envoie une seule requête à la passerelle. La passerelle répartit les requêtes entre les différents services principaux, puis agrège les résultats et les renvoie au client. Cela permet de réduire les échanges excessifs entre le client et les services principaux.

[Déchargement de passerelle](../patterns/gateway-offloading.md) : utilisez la passerelle pour décharger des fonctionnalités à partir des services individuels vers la passerelle, en particulier les problèmes transversaux. Il peut être utile de regrouper ces fonctions à un seul endroit, plutôt que de confier leur mise en œuvre à chaque service. Cela est particulièrement vrai pour les fonctionnalités dont l’implémentation correcte nécessite des compétences particulières, telles que l’authentification et l’autorisation.

Voici quelques exemples de fonctionnalités qui peuvent être déchargées sur une passerelle :

- Arrêt SSL
- Authentification
- Liste verte d’adresses IP
- Limitation du débit client
- Enregistrement et surveillance
- Mise en cache des réponses
- Pare-feu d’application web
- Compression GZIP
- Maintenance du contenu statique

## <a name="choosing-a-gateway-technology"></a>Choix d’une technologie de passerelle

Voici quelques options justifiant l’implémentation d’une passerelle d’API dans votre application.

- **Serveur proxy inverse** : Nginx et HAProxy sont des serveurs proxy inverse courants qui prennent en charge des fonctionnalités telles que l’équilibrage de charge, le SSL et le routage de couche 7. Tous deux sont des produits libres et open source, avec des éditions payantes qui fournissent des fonctionnalités supplémentaires et des options de support. Nginx et HAProxy sont des produits matures, dotés de riches ensembles de fonctionnalités et aux hautes performances. Vous pouvez les étendre avec des modules tiers ou en écrivant des scripts personnalisés dans Lua. Nginx prend également en charge un module de création de script basé sur JavaScript, appelé NginScript.

- **Contrôleur d’entrée de maillage de service** : si vous utilisez un maillage de service tel que linkerd ou Istio, considérez les fonctionnalités fournies par le contrôleur d’entrée de ce service de maillage. Par exemple, le contrôleur d’entrée Istio prend en charge le routage de couche 7, les redirections HTTP, les nouvelles tentatives et d’autres fonctionnalités.

- [Azure Application Gateway](/azure/application-gateway/) : Application Gateway est un service managé d’équilibrage de charge qui peut exécuter un routage de couche 7et un arrêt SSL. Il fournit également un pare-feu d’application web (WAF).

- [Azure API Management](/azure/api-management/) : API Management est une solution clé en main pour la publication d’API à destination des clients internes et externes. Elle fournit des fonctionnalités utiles pour gérer une API publique, y compris la limitation du débit, la liste verte d’adresses IP et l’authentification à l’aide d’Azure Active Directory ou d’autres fournisseurs d’identité. API Management n’effectue aucun équilibrage de charge. Par conséquent, elle doit être utilisée conjointement à un équilibreur de charge, tel qu’Application Gateway ou un proxy inverse. Pour plus d’informations sur l’utilisation d’API Management avec Application Gateway, voir [Intégration de la gestion des API dans un réseau virtuel interne avec Application Gateway](/azure/api-management/api-management-howto-integrate-internal-vnet-appgateway).

Lors du choix d’une technologie de passerelle, considérez les points suivants :

**Fonctionnalités** : les options répertoriées ci-dessus prennent en charge le routage de couche 7, mais la prise en charge des autres fonctionnalités varie. Selon les fonctionnalités dont vous avez besoin, vous pouvez déployer plusieurs passerelles.

**Déploiement**. Azure Application Gateway et API Management sont des services managés. Nginx et HAProxy s’exécutent généralement dans des conteneurs situés à l’intérieur du cluster, mais ils peuvent également être déployés vers des machines virtuelles dédiées en dehors du cluster. Cela isole la passerelle du reste de la charge de travail, mais entraîne une surcharge de gestion plus élevée.

**Gestion** : lors de l’ajout ou de la mise à jour de services, les règles de routage de passerelle peuvent nécessiter une mise à jour. Envisagez le mode de gestion de ce processus. Des considérations similaires s’appliquent à la gestion des certificats SSL, des listes vertes d’adresses IP et d’autres aspects de la configuration.

## <a name="deploying-nginx-or-haproxy-to-kubernetes"></a>Déploiement Nginx ou HAProxy vers Kubernetes

Vous pouvez déployer Nginx ou HAProxy sur Kubernetes en tant que [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) ou [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) qui spécifie l’image de conteneur Nginx ou HAProxy. Utilisez un élément ConfigMap pour stocker le fichier de configuration du proxy et montez l’élément ConfigMap en tant que volume. Créez un service de type LoadBalancer pour exposer la passerelle via un équilibreur de charge Azure.

Une alternative consiste à créer un contrôleur d’entrée. Un contrôleur d’entrée est une ressource Kubernetes qui déploie un équilibreur de charge ou un serveur proxy inverse. Plusieurs implémentations existent, y compris Nginx et HAProxy. Une ressource séparée appelée « entrée » définit des paramètres pour le contrôleur d’entrée, tels que des règles de routage et des certificats TLS. De cette façon, vous n’avez pas besoin de gérer des fichiers de configuration complexes propres à une technologie de serveur proxy spécifique.

La passerelle est un point de défaillance unique ou un goulot d’étranglement potentiel dans le système. Déployez donc toujours au moins deux réplicas pour une haute disponibilité. Vous devrez peut-être mettre à l’échelle les réplicas, en fonction de la charge.

Envisagez également d’exécuter la passerelle sur un ensemble dédié de nœuds dans le cluster. Cette approche présente les avantages suivants :

- Isolement : tout le trafic entrant est envoyé vers un ensemble fixe de nœuds, qui peut être isolé des services principaux.

- Configuration stable : si la passerelle est mal configurée, l’application entière peut devenir indisponible.

- Les performances. vous pouvez utiliser une configuration de machine virtuelle spécifique pour la passerelle pour des raisons de performances.

> [!div class="nextstepaction"]
> [Enregistrement et surveillance](./logging-monitoring.md)
