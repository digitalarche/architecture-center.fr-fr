---
title: "Exécuter des machines virtuelles Linux dans plusieurs régions à des fins de haute disponibilité"
description: "Découvrez comment déployer des machines virtuelles dans plusieurs régions Azure à des fins de haute disponibilité et de résilience."
author: MikeWasson
ms.date: 11/22/2016
pnp.series.title: Linux VM workloads
pnp.series.prev: n-tier
ms.openlocfilehash: 3b68f6fc79ba4b29e41ba2b04537b834bb8859b0
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2017
---
# <a name="run-linux-vms-in-multiple-regions-for-high-availability"></a>Exécuter des machines virtuelles Linux dans plusieurs régions à des fins de haute disponibilité

Cette architecture de référence présente un ensemble de pratiques éprouvées pour l’exécution d’une application multiniveau dans plusieurs régions Azure, afin de bénéficier d’une haute disponibilité et d’une infrastructure de récupération d’urgence fiable. 

![[0]][0]

*Téléchargez un [fichier Visio][visio-download] de cette architecture.*

## <a name="architecture"></a>Architecture 

Cette architecture repose sur celle décrite dans l’article [Exécuter des machines virtuelles Linux pour une application multiniveau](n-tier.md). 

* **Régions primaires et secondaires**. Pour obtenir une plus haute disponibilité, utilisez deux régions. L’une est la région primaire, tandis que l’autre sert au basculement.
* **Azure Traffic Manager**. [Traffic Manager][traffic-manager] achemine les requêtes entrantes vers l’une des régions. Pendant le fonctionnement normal, il achemine les requêtes vers la région primaire. Si cette région n’est plus disponible, Traffic Manager bascule vers la région secondaire. Pour plus d’informations, consultez la section [Configuration de Traffic Manager](#traffic-manager-configuration).
* **Groupes de ressources**. Créez des [groupes de ressources][resource groups] distincts pour la région primaire, la région secondaire et Traffic Manager. Vous obtenez ainsi la flexibilité nécessaire pour gérer chaque région comme une collection de ressources unique. Par exemple, vous pourriez redéployer une région sans arrêter l’autre. [Liez les groupes de ressources][resource-group-links] afin de pouvoir exécuter une requête pour répertorier toutes les ressources de l’application.
* **Réseaux virtuels**. Créez un réseau virtuel distinct pour chaque région. Vérifiez que les espaces d’adressage ne se chevauchent pas.
* **Apache Cassandra**. Déployez Cassandra dans des centres de données dans plusieurs régions Azure pour bénéficier d’une haute disponibilité. Dans chaque région, les nœuds sont configurés en mode rack avec des domaines d’erreur et de mise à niveau, afin d’assurer la résilience à l’intérieur de la région.

## <a name="recommendations"></a>Recommandations

Une architecture multirégion peut offrir une disponibilité plus élevée qu’un déploiement dans une seule région. Si une panne régionale affecte la région primaire, vous pouvez utiliser [Traffic Manager][traffic-manager] pour basculer vers la région secondaire. Cette architecture peut également se révéler utile en cas de défaillance d’un sous-système spécifique de l’application.

Plusieurs approches générales permettent de bénéficier d’une haute disponibilité dans l’ensemble des régions :   

* Mode actif/passif avec serveur de secours. Le trafic est dirigé vers une région, tandis que l’autre région est en attente sur le serveur de secours. Le terme « serveur de secours » signifie que les machines virtuelles de la région secondaire sont allouées et en cours d’exécution en permanence.
* Mode actif/passif avec reprise progressive. Le trafic est dirigé vers une région, tandis que l’autre région est en attente sur le centre de données de reprise progressive. Le terme « reprise progressive » signifie que les machines virtuelles de la région secondaire ne sont pas allouées tant qu’elles ne sont pas requises pour le basculement. La mise en œuvre de cette approche se révèle moins coûteuse, mais nécessite davantage de temps en cas de défaillance.
* Mode actif/actif. Les deux régions sont actives, et la charge de travail des requêtes est équilibrée entre les régions. Si l’une des régions n’est plus disponible, elle est mise hors service. 

Cette architecture est axée sur le mode actif/passif avec serveur de secours, et utilise Traffic Manager pour le basculement. Notez que vous pouvez déployer un petit nombre de machines virtuelles pour le serveur de secours, puis monter en charge en fonction des besoins.


### <a name="regional-pairing"></a>Association régionale

Chaque région Azure est associée à une autre région de la même zone géographique. En général, vous devez choisir des régions de la même paire régionale (par exemple, Est des États-Unis 2 et Centre des États-Unis). Cette approche offre les avantages suivants :

* En cas d’interruption de service générale, la récupération d’au moins une région de chaque paire est prioritaire.
* Les mises à jour planifiées du système Azure sont déployées dans les régions associées de manière séquentielle, afin de minimiser les temps d’arrêt possibles.
* Les paires régionales appartiennent à la même zone géographique, afin de répondre aux exigences en matière de résidence des données.

Toutefois, vous devez vérifier que les deux régions prennent en charge tous les services Azure nécessaires pour votre application (voir [Services par région][services-by-region]). Pour plus d’informations sur les paires régionales, consultez l’article [Continuité des activités et récupération d’urgence (BCDR) : régions jumelées d’Azure][regional-pairs].

### <a name="traffic-manager-configuration"></a>Configuration de Traffic Manager

Considérez les points suivants lors de la configuration de Traffic Manager :

* **Routage**. Traffic Manager prend en charge plusieurs [algorithmes de routage][tm-routing]. Pour le scénario décrit dans cet article, utilisez le routage *par priorité* (auparavant désigné sous le terme de routage *par basculement*). Quand cette méthode de routage est configurée, Traffic Manager envoie toutes les requêtes à la région primaire, sauf si elle devient inaccessible. À ce moment-là, les requêtes basculent automatiquement vers la région secondaire. Consultez [Configurer la méthode de routage de basculement][tm-configure-failover].
* **Sonde d’intégrité**. Traffic Manager utilise une [sonde][tm-monitoring] HTTP (ou HTTPS) pour surveiller la disponibilité de chaque région. La sonde vérifie la présence d’une réponse HTTP 200 pour un chemin d’URL spécifié. Une bonne pratique consiste à créer un point de terminaison qui signale l’intégrité globale de l’application et à utiliser ce point de terminaison pour la sonde d’intégrité. Dans le cas contraire, la sonde risque de signaler un point de terminaison intègre alors que des parties critiques de l’application sont défaillantes. Pour plus d’informations, consultez [Modèle de surveillance de point de terminaison d’intégrité][health-endpoint-monitoring-pattern].

Quand Traffic Manager déclenche un basculement, l’application reste inaccessible aux clients pendant un certain laps de temps. Ce laps de temps dépend des facteurs suivants :

* La sonde d’intégrité doit détecter que la région primaire est devenue inaccessible.
* Les serveurs DNS (Domain Name Service) doivent mettre à jour les enregistrements DNS mis en cache pour l’adresse IP, qui dépend de la durée de vie (TTL) DNS. La valeur TTL par défaut est de 300 secondes (5 minutes), mais vous pouvez configurer cette valeur quand vous créez le profil Traffic Manager.

Pour plus d’informations, consultez [À propos de la surveillance de Traffic Manager][tm-monitoring].

En cas de basculement de Traffic Manager, nous vous recommandons de procéder à une restauration manuelle plutôt que d’implémenter une restauration automatique. Dans le cas contraire, l’application risque d’alterner continuellement entre les régions. Vérifiez que tous les sous-systèmes de l’application sont intègres avant d’effectuer la restauration automatique.

Notez que Traffic Manager procède à une restauration automatique par défaut. Pour éviter cela, diminuez manuellement la priorité de la région primaire après un événement de basculement. Par exemple, supposez que la région primaire a la priorité 1, et que la base de données secondaire a la priorité 2. Après un basculement, définissez la priorité de la région primaire sur la valeur 3 afin d’empêcher la restauration automatique. Quand vous êtes prêt à rebasculer vers cette région, redéfinissez sa priorité sur la valeur 1.

La commande [Azure CLI][install-azure-cli] suivante met à jour la priorité :

```bat
azure network traffic-manager  endpoint set --resource-group <resource-group> --profile-name <profile>
    --name <traffic-manager-name> --type AzureEndpoints --priority 3
```    

Une autre approche consiste à désactiver temporairement le point de terminaison jusqu’à ce que vous soyez prêt à effectuer la restauration automatique :

```bat
azure network traffic-manager  endpoint set --resource-group <resource-group> --profile-name <profile>
    --name <traffic-manager-name> --type AzureEndpoints --status Disabled
```    

En fonction de la cause d’un basculement, vous devrez peut-être redéployer les ressources au sein d’une région. Avant d’effectuer une restauration automatique, exécutez un test de disponibilité opérationnelle. Le test doit vérifier entre autres ce qui suit :

* Les machines virtuelles sont configurées correctement. (Tous les logiciels nécessaires sont installés, IIS est en cours d’exécution, et ainsi de suite.)
* Les sous-systèmes d’application sont intègres.
* Test fonctionnel. (Par exemple, le niveau de la base de données est accessible à partir du niveau Web.)

### <a name="cassandra-deployment-across-multiple-regions"></a>Déploiement de Cassandra dans plusieurs régions

Les centres de données Cassandra sont un groupe de nœuds de données associés qui sont configurés dans un cluster à des fins de répartition de charge de travail et de réplication.

Nous vous recommandons [DataStax Enterprise][datastax] pour une utilisation en production. Pour plus d’informations sur l’exécution de DataStax dans Azure, consultez le [Guide de déploiement de DataStax Enterprise pour Azure][cassandra-in-azure]. Les recommandations générales suivantes s’appliquent à toute édition de Cassandra : 

* Attribuez une adresse IP publique à chaque nœud. Cela permet aux clusters de communiquer d’une région à une autre à l’aide de l’infrastructure principale Azure, et de bénéficier ainsi d’un débit élevé à faible coût.
* Sécurisez les nœuds à l’aide des configurations de pare-feu et de groupe de sécurité réseau appropriées, en autorisant le trafic uniquement vers et à partir des hôtes connus, notamment les clients et les autres nœuds du cluster. Notez que Cassandra utilise différents ports pour la communication : OpsCenter, Spark et ainsi de suite. Pour plus d’informations sur l’utilisation des ports dans Cassandra, consultez [Configuration de l’accès aux ports de pare-feu][cassandra-ports].
* Utiliser le chiffrement SSL pour toutes les communications [client à nœud][ssl-client-node] et [nœud à nœud][ssl-node-node].
* Dans une région, suivez les instructions fournies dans [Recommandations relatives à Cassandra](n-tier.md#cassandra).

## <a name="availability-considerations"></a>Considérations relatives à la disponibilité

Avec une application multiniveau complexe, vous n’aurez peut-être pas besoin de répliquer l’ensemble de l’application dans la région secondaire. Au lieu de cela, vous pourrez simplement répliquer un sous-système critique nécessaire pour prendre en charge la continuité d’activité.

Traffic Manager est un point de défaillance possible dans le système. Si le service Traffic Manager échoue, les clients ne peuvent plus accéder à votre application pendant le temps d’arrêt. Examinez le [Contrat de niveau de service (SLA) pour Traffic Manager][tm-sla] et déterminez si Traffic Manager peut à lui seul répondre à vos exigences métiers en matière de haute disponibilité. Si ce n’est pas le cas, ajoutez une autre solution de gestion du trafic en tant que restauration automatique. Si le service Azure Traffic Manager échoue, modifiez vos enregistrements CNAME dans DNS pour qu’ils pointent vers l’autre service de gestion du trafic. (Cette opération doit être effectuée manuellement, et votre application reste inaccessible tant que ces modifications DNS n’ont pas été propagées.)

Pour le cluster Cassandra, les scénarios de basculement à prendre en compte dépendent des niveaux de cohérence utilisés par l’application, ainsi que du nombre de réplicas utilisés. Pour plus d’informations sur l’utilisation et les niveaux de cohérence dans Cassandra, consultez [Configuring data consistency ][cassandra-consistency] (Configuration de la cohérence des données) et [Cassandra: How many nodes are talked to with Quorum?][cassandra-consistency-usage] (Cassandra : quel est le nombre de nœuds en communication avec le quorum ?) La disponibilité des données dans Cassandra est déterminée par le niveau de cohérence utilisé par l’application et le mécanisme de réplication. Pour plus d’informations sur la réplication dans Cassandra, consultez [Data Replication in NoSQL Databases Explained][cassandra-replication] (Explication de la réplication des données dans les bases de données NoSQL).

## <a name="manageability-considerations"></a>Considérations relatives à la facilité de gestion

Quand vous mettez à jour votre déploiement, mettez à jour une seule région à la fois, afin de réduire le risque de défaillance globale due à une configuration incorrecte ou à une erreur dans l’application.

Testez la résilience aux défaillances du système. Voici quelques scénarios courants de défaillance à tester :

* Arrêt des instances de machine virtuelle.
* Pression sur les ressources telles que le processeur et la mémoire.
* Déconnexion/délai de réseau.
* Blocage de processus.
* Expiration de certificats.
* Simulation de défaillances matérielles.
* Arrêt du service DNS sur les contrôleurs de domaine.

Mesurez les temps de récupération et vérifiez qu’ils répondent aux besoins de votre entreprise. Testez également des combinaisons de défaillances.


<!-- Links -->
[hybrid-vpn]: ../hybrid-networking/vpn.md

[cassandra-in-azure]: https://academy.datastax.com/resources/deployment-guide-azure
[cassandra-consistency]: http://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_config_consistency_c.html
[cassandra-replication]: http://www.planetcassandra.org/data-replication-in-nosql-databases-explained/
[cassandra-consistency-usage]: https://medium.com/@foundev/cassandra-how-many-nodes-are-talked-to-with-quorum-also-should-i-use-it-98074e75d7d5#.b4pb4alb2
[cassandra-ports]: https://docs.datastax.com/en/datastax_enterprise/5.0/datastax_enterprise/sec/configFirewallPorts.html
[datastax]: https://www.datastax.com/products/datastax-enterprise
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[install-azure-cli]: /azure/xplat-cli-install
[regional-pairs]: /azure/best-practices-availability-paired-regions
[resource groups]: /azure/azure-resource-manager/resource-group-overview
[resource-group-links]: /azure/resource-group-link-resources
[services-by-region]: https://azure.microsoft.com/regions/#services
[ssl-client-node]: http://docs.datastax.com/en/cassandra/2.0/cassandra/security/secureSSLClientToNode_t.html
[ssl-node-node]: http://docs.datastax.com/en/cassandra/2.0/cassandra/security/secureSSLNodeToNode_t.html
[tablediff]: https://msdn.microsoft.com/library/ms162843.aspx
[tm-configure-failover]: /azure/traffic-manager/traffic-manager-configure-failover-routing-method
[tm-monitoring]: /azure/traffic-manager/traffic-manager-monitoring
[tm-routing]: /azure/traffic-manager/traffic-manager-routing-methods
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager/v1_0/
[traffic-manager]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[wsfc]: https://msdn.microsoft.com/library/hh270278.aspx
[0]: ./images/multi-region-application-diagram.png "Architecture réseau hautement disponible pour les applications multiniveaux Azure"
