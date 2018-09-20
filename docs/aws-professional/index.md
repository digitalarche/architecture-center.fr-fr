---
title: Azure pour les professionnels AWS
description: Apprenez les principes fondamentaux des comptes, plateforme et services de Microsoft Azure. Découvrez également les similitudes et les différences clés entre les plateformes AWS et Azure. Tirez parti de votre expérience AWS dans Azure.
keywords: Experts AWS, comparaison de Azure, comparaison de AWS, différences entre Azure et AWS, Azure et AWS
author: lbrader
ms.date: 03/24/2017
pnp.series.title: Azure for AWS Professionals
ms.openlocfilehash: f90186ba36f035871ec918c65b05eebe44190ffa
ms.sourcegitcommit: f1dcc388c8b4fc983549c36d7e6b009fa1f072ba
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/19/2018
ms.locfileid: "46329893"
---
# <a name="azure-for-aws-professionals"></a>Azure pour les professionnels AWS

Cet article aide les experts de Amazon Web Services (AWS) à comprendre les principes fondamentaux des comptes, plateforme et services Microsoft Azure. Il couvre également les similitudes et les différences clés entre les plateformes AWS et Azure.

Vous apprendrez ce qui suit :

* Comment les comptes et les ressources sont organisés dans Azure.
* Comment les solutions disponibles sont structurées dans Azure.
* Comment les principaux services Azure diffèrent des services AWS.

Azure et AWS ont développé leurs fonctionnalités indépendamment au fil du temps, de sorte qu’ils ont des différences importantes de conception et d’implémentation.

## <a name="overview"></a>Vue d’ensemble

Comme AWS, Microsoft Azure est construit autour d’un ensemble basique de services de calcul, de stockage, de base de données et de réseau. Dans de nombreux cas, les deux plateformes offrent une équivalence de base entre les produits et services qu’elles proposent. AWS et Azure permettent de concevoir des solutions hautement disponibles basées sur des ordinateurs hôtes Windows ou Linux. Par conséquent, si vous êtes habitué au développement à l’aide de Linux et de la technologie OSS, les deux plateformes peuvent faire le travail.

Si les fonctionnalités de ces deux plateformes sont similaires, les ressources fournissant ces fonctionnalités sont souvent organisées différemment. Les relations exactes un à un entre les services requis pour générer une solution ne sont pas toujours claires. Il existe également des cas où un service particulier peut être proposé sur une plateforme, mais pas sur l’autre. Consultez les [graphiques des services comparables de Azure et AWS](services.md).

## <a name="accounts-and-subscriptions"></a>Comptes et abonnements

Vous pouvez acheter des services Azure à l’aide de plusieurs options de tarification, en fonction de la taille et des besoins de votre organisation. Consultez la page [Vue d’ensemble de la tarification](https://azure.microsoft.com/pricing/) pour plus d’informations.

Les [abonnements Azure](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-infrastructure-subscription-accounts-guidelines/) sont un regroupement de ressources avec un propriétaire attribué, responsable de la facturation et de la gestion des autorisations. À la différence de AWS, où toutes les ressources créées sous le compte AWS sont liées à ce compte, les abonnements existent indépendamment du compte de leur propriétaire et ils peuvent être réattribués à de nouveaux propriétaires selon les besoins.

![Comparaison de la structure et de la propriété des comptes AWS et des abonnements Azure](./images/azure-aws-account-compare.png "Comparaison de la structure et de la propriété des comptes AWS et des abonnements Azure")
<br/>*Comparaison de la structure et de la propriété des comptes AWS et des abonnements Azure*
<br/><br/>

Les abonnements sont attribués à trois types de comptes d’administrateur :

-   **Administrateur de compte** -Propriétaire de l’abonnement et compte facturé pour les ressources utilisées dans l’abonnement. L’administrateur de compte ne peut être modifié qu’en transférant la propriété de l’abonnement.

-   **Administrateur de service** -Ce compte dispose des droits pour créer et gérer des ressources au sein de l’abonnement, mais il n’est pas responsable de la facturation. Par défaut, les statuts Administrateur de compte et Administrateur de service sont attribués au même compte. L’administrateur de compte peut attribuer le statut Administrateur de service à un utilisateur distinct afin qu’il gère les aspects techniques et opérationnels d’un abonnement. Il n’existe qu’un seul administrateur de service par abonnement.

-   **Coadministrateur** -Il peut y avoir plusieurs comptes de coadministrateurs attribués à un abonnement. Les coadministrateurs ne peuvent pas changer l’administrateur de service, autrement, ils ont un contrôle total sur les utilisateurs et les ressources de l’abonnement.

Au sein d’un abonnement, des rôles de niveau d’utilisateur et des autorisations individuelles peuvent également être attribués à des ressources spécifiques, de la même façon que l’attribution des autorisations aux utilisateurs et des groupes de IAM dans AWS. Dans Azure, tous les comptes d’utilisateur sont associés avec un compte Microsoft ou bien un compte d’organisation (un compte géré via Azure Active Directory).

Comme les comptes AWS, les abonnements ont des quotas et des limites de service par défaut. Pour connaître la liste complète de ces limites, consultez [Abonnement Azure et limites, quotas et contraintes du service](https://azure.microsoft.com/documentation/articles/azure-subscription-service-limits/).
Ces limites peuvent être augmentées jusqu’à la limite maximale grâce au [dépôt d’une requête de support dans le portail de gestion](https://blogs.msdn.microsoft.com/girishp/2015/09/20/increasing-core-quota-limits-in-azure/).

### <a name="see-also"></a>Voir aussi

-   [Ajout ou modification de rôles d’administrateur Azure](https://azure.microsoft.com/documentation/articles/billing-add-change-azure-subscription-administrator/)

-   [Comment télécharger votre facture Azure et vos données d’utilisation quotidienne](https://azure.microsoft.com/documentation/articles/billing-download-azure-invoice-daily-usage-date/)

## <a name="resource-management"></a>Gestion des ressources

Le terme « ressource » dans Azure est utilisé de la même façon que dans AWS, référant à n’importe quel objet de stockage, instance de calcul, périphérique réseau ou toute autre entité que vous pouvez créer ou configurer au sein de la plateforme.

Les ressources Azure sont déployées et gérées à partir de l’un des deux modèles suivants : [Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview) ou le [modèle de déploiement classique Azure](/azure/azure-resource-manager/resource-manager-deployment-model) plus ancien.
Chaque nouvelle ressource est créée à l’aide du modèle de gestionnaire des ressources.

### <a name="resource-groups"></a>Groupes de ressources

Azure et AWS possèdent des entités nommées « groupes de ressources »qui organisent des ressources telles que les machines virtuelles, le stockage et les appareils de réseau virtuel. Toutefois, les [groupes de ressources Azure](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/) ne sont pas directement comparables aux groupes de ressources AWS.

Alors que AWS permet à une ressource d’être référencée dans plusieurs groupes de ressources, une ressource Azure est toujours associée à un seul groupe de ressources. Une ressource créée dans un groupe de ressources peut être déplacée dans un autre groupe, mais elle ne peut appartenir à un seul groupe de ressources à la fois. Les groupes de ressources constituent le regroupement de base utilisé par Azure Resource Manager.

Les ressources peuvent également être organisées à l’aide de [balises](https://azure.microsoft.com/documentation/articles/resource-group-using-tags/).
Les balises sont des paires clé-valeur qui vous permettent de regrouper des ressources dans votre abonnement, quel que soit leur groupe de ressources.

### <a name="management-interfaces"></a>Interfaces de gestion

Azure propose plusieurs façons de gérer vos ressources :

-   [Interface Web](https://azure.microsoft.com/documentation/articles/resource-group-portal/).
    Comme le tableau de bord AWS, le portail Azure fournit une interface de gestion web complète pour les ressources Azure.

-   [API REST](https://azure.microsoft.com/documentation/articles/resource-manager-rest-api/).
    L’API REST de Azure Resource Manager fournit un accès par programme à la plupart des fonctionnalités disponibles depuis le portail Azure.

-   [Ligne de commande](https://azure.microsoft.com/documentation/articles/xplat-cli-azure-resource-manager/).
    L’outil Azure CLI 2.0 fournit une interface de ligne de commande capable de créer et de gérer des ressources Azure. Azure CLI est disponible pour [Windows, Linux et Mac OS](https://aka.ms/azurecli2).

-   [PowerShell](https://azure.microsoft.com/documentation/articles/powershell-azure-resource-manager/).
    Les modules Azure pour PowerShell permettent d’exécuter des tâches de gestion automatisées à l’aide d’un script. PowerShell est disponible pour [Windows, Linux et Mac OS](https://github.com/PowerShell/PowerShell).

-   [Modèles](https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/).
    Les modèles de Azure Resource Manager fournissent des fonctionnalités de gestion de ressources basées sur le modèle JSON, similaires au service AWS CloudFormation.

Dans chacune de ces interfaces, le groupe de ressources est essentiel pour la création, le déploiement et les modifications des ressources Azure. Son rôle est similaire à celui d’une « pile » dans le regroupement des ressources AWS pendant les déploiements de CloudFormation.

La syntaxe et la structure de ces interfaces sont différentes de leurs équivalents AWS, mais elles disposent de fonctionnalités comparables. En outre, plusieurs outils de gestion tiers utilisés sur AWS, comme [Terraform de Hashicorp](https://www.terraform.io/docs/providers/azurerm/) et [Netflix Spinnaker](http://www.spinnaker.io/), sont également disponibles sur Azure.

### <a name="see-also"></a>Voir aussi

-   [Instructions pour les groupes de ressources Azure](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/)

## <a name="regions-and-zones-high-availability"></a>Régions et zones (haute disponibilité)

Les défaillances n’ont pas toutes la même incidence. Certaines défaillances matérielles, comme une panne de disque, peuvent affecter un seul ordinateur hôte. Un commutateur réseau défaillant peut impacter un rack entier de serveurs. Vous déplorerez moins fréquemment des défaillances perturbant un centre de données dans son ensemble, comme une panne d’alimentation. Exceptionnellement, une région entière peut être indisponible.

La redondance est l’un des moyens de rendre une application résiliente. Toutefois, vous devez planifier en fonction de cette redondance lorsque vous concevez l’application. Par ailleurs, le niveau de redondance dont vous avez besoin dépend des exigences de votre entreprise. Toutes les applications ne nécessitent pas une redondance entre les régions à titre de prévention contre les pannes régionales. En général, il existe un compromis entre redondance et fiabilité supérieures d’un côté contre complexité et coûts plus élevés de l’autre.  

Dans AWS, une région est divisée en deux zones de disponibilité ou plus. Une zone de disponibilité correspond à un centre de données physiquement isolé dans la région géographique. Azure offre un certain nombre de fonctionnalités servant à rendre une application redondante à tous les nivaux de défaillances, dont les **groupes à haute disponibilité**, les **zones de disponibilité** et les **régions jumelées**. 

![](../resiliency/images/redundancy.svg)

Le tableau suivant récapitule chaque option.

| &nbsp; | Groupe à haute disponibilité | Zone de disponibilité | Région jumelée |
|--------|------------------|-------------------|---------------|
| Étendue de la défaillance | Rack | Centre de données | Région |
| Routage des requêtes | Équilibreur de charge | Équilibreur de charge entre les zones | Traffic Manager |
| Latence du réseau | Très faible | Faible | Moyenne à élevée |
| Réseau virtuel  | Réseau virtuel | Réseau virtuel | Homologation de réseaux virtuels entre régions |

### <a name="availability-sets"></a>Groupes à haute disponibilité 

Pour vous protéger contre les défaillances matérielles localisées, comme une panne de disque ou de commutateur réseau, déployez au moins deux machines virtuelles dans un groupe à haute disponibilité. Un groupe à haute disponibilité se compose d’au moins deux *domaines d’erreur* qui partagent une source d’alimentation et un commutateur réseau. Les machines virtuelles d’un groupe à haute disponibilité sont distribuées entre les domaines d’erreur. Ainsi, si une défaillance matérielle affecte un domaine d’erreur, le trafic réseau peut toujours être acheminé vers les machines virtuelles des autres domaines d’erreur. Pour plus d’informations sur les groupes à haute disponibilité, consultez la section [Gestion de la disponibilité des machines virtuelles Windows dans Azure](/azure/virtual-machines/windows/manage-availability).

Lorsque des instances de machine virtuelle sont ajoutées aux groupes à haute disponibilité, un [domaine de mise à jour](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-manage-availability/) leur est également attribué. Un domaine de mise à jour est un groupe de machines virtuelles définies pour des événements de maintenance planifiée en même temps. La distribution de machines virtuelles sur plusieurs domaines de mise à jour garantit qu’une mise à jour planifiée et des événements de mise à jour corrective affectent uniquement un sous-ensemble de ces machines virtuelles à un moment donné.

Les groupes à haute disponibilité devraient être organisés en fonction du rôle de l’instance dans votre application pour garantir qu’une instance de chaque rôle est opérationnelle. Par exemple, dans une application web à trois couches, créez des groupes à haute disponibilité distincts pour les couches frontales, d’application et de données.

![Groupes à haute disponibilité d’Azure pour chaque rôle d’application](./images/three-tier-example.png "Groupes à haute disponibilité pour chaque rôle d’application")

### <a name="availability-zones"></a>Zones de disponibilité

Une [zone de disponibilité](/azure/availability-zones/az-overview) est une zone physiquement séparée au sein d’une région Azure. Chaque zone de disponibilité possède une source d’alimentation, un réseau et un système de refroidissement propres. Le déploiement des machines virtuelles entre les zones de disponibilité aide à protéger une application contre les défaillances à l’échelle du centre de données. 

### <a name="paired-regions"></a>Régions jumelées

Pour protéger une application contre une panne régionale, vous pouvez la déployer dans plusieurs régions, en vous appuyant sur [Azure Traffic Manager][traffic-manager] pour distribuer le trafic Internet entre les différentes régions. Chaque région Azure est jumelée à une autre région. Ensemble, elles forment une [paire régionale][paired-regions]. Une région se trouve dans la même zone géographique que la région avec laquelle elle est jumelée (à l’exception de Brésil Sud) pour répondre aux exigences de la résidence de données en termes d’impôts et d’application de la loi.

Contrairement aux zones de disponibilité, physiquement séparées des centres de données mais pouvant être dans des zones géographiques relativement proches, les régions jumelées associées sont généralement séparées de plusieurs centaines de kilomètres. Cela permet de s’assurer que les sinistres importants n’affectent que l’une des régions jumelées. Les paires voisines peuvent être définies pour synchroniser une base de données et des données de service de stockage, et elles sont configurées de sorte que les mises à jour de plateforme soient déployées sur une région de la paire à la fois.

[Le stockage géo-redondant](https://azure.microsoft.com/documentation/articles/storage-redundancy/#geo-redundant-storage) de Azure est automatiquement sauvegardé vers la région jumelée appropriée. Pour toutes les autres ressources, la création d’une solution entièrement redondante à l’aide de régions jumelées implique la création d’une copie complète de votre solution dans les deux régions.


### <a name="see-also"></a>Voir aussi

-   [Régions et disponibilité des machines virtuelles dans Azure](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-regions-and-availability/)

-   [Haute disponibilité des applications Azure](../resiliency/high-availability-azure-applications.md)

-   [Récupération d’urgence des applications Microsoft Azure](../resiliency/disaster-recovery-azure-applications.md)

-   [Maintenance planifiée des machines virtuelles Linux dans Azure](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-planned-maintenance/)

## <a name="services"></a>Services

Pour obtenir la liste des correspondances des services entre les plateformes, consultez [Comparaison des services AWS et Azure](./services.md).

Tous les produits et les services Azure ne sont pas disponibles dans toutes les régions. Consultez la page [Produits par région](https://azure.microsoft.com/regions/services/) pour plus d’informations. Vous pouvez trouver les garanties de temps d’activité et les stratégies de crédit de temps d’arrêt pour chaque produit ou service Azure sur la page [Contrats de niveau de service](https://azure.microsoft.com/support/legal/sla/).

Les sections suivantes fournissent une brève explication des différences entre les plateformes Azure et AWS pour les services et fonctionnalités couramment utilisés.

### <a name="compute-services"></a>Services de calcul

#### <a name="ec2-instances-and-azure-virtual-machines"></a>Instances EC2 et machines virtuelles Azure

Bien que les types d’instance de AWS et les tailles de machine virtuelle de Azure se décomposent de la même façon, il existe des différences au niveau de la mémoire RAM, de l’UC et des capacités de stockage.

-   [Types d’instance EC2 Amazon](https://aws.amazon.com/ec2/instance-types/)

-   [Tailles des machines virtuelles dans Azure (Windows)](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/)

-   [Tailles des machines virtuelles dans Azure (Linux)](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-sizes/)

Contrairement à la facturation par seconde de AWS, les machines virtuelles Azure à la demande sont facturées à la minute.

#### <a name="ebs-and-azure-storage-for-vm-disks"></a>EBS et stockage Azure pour les disques de machine virtuelle

Le stockage de données durable pour les machines virtuelles Azure est fourni par [des disques de données](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-about-disks-vhds/) résidant dans un stockage Blob. Cela est similaire à la façon dont les instances EC2 stockent des volumes de disque sur Elastic Block Store (EBS). Le [stockage temporaire Azure](https://blogs.msdn.microsoft.com/mast/2013/12/06/understanding-the-temporary-drive-on-windows-azure-virtual-machines/) fournit également aux machines virtuelles un stockage de lecture-écriture temporaire à faible latence en tant que stockage d’instance EC2 (également nommé stockage éphémère).

Les E/S de disque les plus performantes sont prises en charge à l’aide de [Azure Stockage Premium](https://docs.microsoft.com/azure/storage/storage-premium-storage).
Cela est similaire aux options de stockage IOPS configurées fournies par AWS.

#### <a name="lambda-azure-functions-azure-web-jobs-and-azure-logic-apps"></a>Lambda, Azure Functions, Azure Web-Jobs et Azure Logic Apps

[Azure Functions](https://azure.microsoft.com/services/functions/) est l’équivalent principal de AWS Lambda en fournissant un code sans serveur, à la demande.
Toutefois, la fonctionnalité Lambda chevauche également d’autres services Azure :

-   [Webjobs](https://azure.microsoft.com/documentation/articles/web-sites-create-web-jobs/) - vous permet de créer des tâches en arrière-plan planifiées ou bien exécutées en continue.

-   [Logic Apps](https://azure.microsoft.com/services/logic-apps/) -fournit des services de communications, d’intégration et de gestion de règles d’entreprise.

#### <a name="autoscaling-azure-vm-scaling-and-azure-app-service-autoscale"></a>Mise à l’échelle automatique, mise à l’échelle de machine virtuelle Azure et mise à l’échelle automatique de Azure App Service

La mise à l’échelle automatique dans Azure est gérée par deux services :

-   [Virtual Machine Scale Sets](https://azure.microsoft.com/documentation/articles/virtual-machine-scale-sets-overview/) -vous permet de déployer et de gérer un ensemble identique de machines virtuelles. Le nombre d’instances peut être mis à l’échelle automatiquement en fonction des besoins de performances.

-   [Mise à l’échelle automatique de App Service](https://azure.microsoft.com/documentation/articles/web-sites-scale/) -fournit la possibilité de mettre automatiquement à l’échelle les solutions de Azure App Service.


#### <a name="container-service"></a>Service de conteneur
[Azure Container Service](https://docs.microsoft.com/azure/container-service/container-service-intro) prend en charge les conteneurs Docker gérés par Docker Swarm, Kubernetes ou DC/OS.

#### <a name="other-compute-services"></a>Autres services de calcul 


Azure offre plusieurs services de calcul qui n’ont pas d’équivalents directs dans AWS :

-   [Azure Batch](https://azure.microsoft.com/documentation/articles/batch-technical-overview/) -vous permet de gérer un travail de calcul intensif dans une collection évolutive de machine virtuelles.

-   [Service Fabric](https://azure.microsoft.com/documentation/articles/service-fabric-overview/) -plateforme pour le développement et l’hébergement évolutif de solutions de [microservice](https://azure.microsoft.com/documentation/articles/service-fabric-overview-microservices/).

#### <a name="see-also"></a>Voir aussi

-   [Création d’une machine virtuelle Linux sur Azure à l’aide du portail](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-quick-create-portal/)

-   [Architectures de référence Azure : exécution d’une machine virtuelle Linux sur Azure](https://azure.microsoft.com/documentation/articles/guidance-compute-single-vm-linux/)

-   [Prise en main des applications web Node.js dans Azure App Service](https://azure.microsoft.com/documentation/articles/app-service-web-nodejs-get-started/)

-   [Architecture de référence Azure : application web de base](https://azure.microsoft.com/documentation/articles/guidance-web-apps-basic/)

-   [Créer votre première fonction Azure](https://azure.microsoft.com/documentation/articles/functions-create-first-azure-function/)

### <a name="storage"></a>Stockage

#### <a name="s3ebsefs-and-azure-storage"></a>S3/EBS/EFS et Stockage Azure

Au sein de la plateforme AWS, le stockage cloud est divisé en trois services :

-   **Simple Storage Service (S3)** - stockage d’objet de base. Rend les données disponibles via une API accessible par internet.

-   **Elastic Block Storage (EBS)** - stockage de niveau bloc, prévu pour l’accès par une machine virtuelle unique.

-   **Système de fichiers élastique (EFS)** - stockage de fichiers conçu pour une utilisation en tant que stockage partagé pour des milliers d’instances EC2.

Dans le stockage Azure, les [comptes de stockage](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/) liés aux abonnements vous permettent de créer et de gérer les services de stockage suivants :

-   [Stockage Blob](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/) - stocke tout type de données texte ou binaires, par exemple, un document, un fichier multimédia ou un programme d’installation d’application. Vous pouvez définir le stockage Blob pour un accès privé ou un partage public du contenu sur internet. Le stockage Blob a le même objectif que S3 et EBS de AWS.
-   [Stockage Table](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-table-storage/) - stocke des jeux de données structurés. Stockage Table est un magasin de données de clés d’attributs NoSQL qui permet le développement et l’accès rapide à de grosses quantités de données. Similaire aux services SimpleDB et DynamoDB de AWS.

-   [Stockage de files d’attente](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/) - fournit une messagerie pour le traitement des flux de travail et pour la communication entre les composants des services cloud.

-   [Stockage Fichier](https://azure.microsoft.com/documentation/articles/storage-java-how-to-use-file-storage/) - offre un stockage partagé pour les applications héritées utilisant le protocole SMB. Stockage de fichier est utilisé d’une façon similaire à EFS dans la plateforme AWS.
 
#### <a name="glacier-and-azure-storage"></a>Glacier et Stockage Azure 

Le service [Stockage Blob Archive Azure](/azure/storage/blobs/storage-blob-storage-tiers#archive-access-tier) est comparable au service de stockage Glacier AWS. Il est destiné aux données rarement sollicitées qui sont stockées pendant 180 jours au moins et qui peuvent tolérer plusieurs heures de latence de récupération. 

Dans le cas des données qui sont peu consultées mais qui doivent être immédiatement accessibles, le [niveau de stockage Blob à froid Azure](/azure/storage/blobs/storage-blob-storage-tiers#cool-access-tier) offre un stockage plus économique que le stockage Blob standard. Ce niveau de stockage est comparable au service de stockage AWS S3 - Infrequent Access.

#### <a name="see-also"></a>Voir aussi

-   [Liste de contrôle des performances et de l’évolutivité de Microsoft Azure Storage](https://azure.microsoft.com/documentation/articles/storage-performance-checklist/)

-   [Guide de sécurité du Stockage Azure](https://azure.microsoft.com/documentation/articles/storage-security-guide/)

-   [Modèles et pratique : Aide relative au réseau de distribution de contenu (CDN)](https://azure.microsoft.com/documentation/articles/best-practices-cdn/)

### <a name="networking"></a>Mise en réseau

#### <a name="elastic-load-balancing-azure-load-balancer-and-azure-application-gateway"></a>Elastic Load Balancing, Azure Load Balancer et Azure Application Gateway

Les équivalents Azure des deux services de Elastic Load Balancing sont :

-   [Load Balancer](https://azure.microsoft.com/documentation/articles/load-balancer-overview/) -possède les mêmes fonctionnalités que Classic Load Balancer de AWS, vous permettant de répartir le trafic pour plusieurs machines virtuelles au niveau du réseau. Il possède également une capacité de basculement.

-   [Application Gateway](https://azure.microsoft.com/documentation/articles/application-gateway-introduction/) - offre un routage basé sur des règles au niveau de l’application comparable à l’application Load Balancer de AWS.

#### <a name="route-53-azure-dns-and-azure-traffic-manager"></a>Route 53, Azure DNS et Azure Traffic Manager

Dans AWS, Route 53 fournit un service de gestion du nom DNS et un service de routage et de basculement au niveau du DNS. Dans Azure, ceci est géré par deux services :

-   [Azure DNS](https://azure.microsoft.com/documentation/services/dns/) effectue la gestion du DNS et du domaine.

-   [Traffic Manager][traffic-manager] s’occupe des fonctionnalités de routage du trafic au niveau DNS, d’équilibrage de charge et du basculement.

#### <a name="direct-connect-and-azure-expressroute"></a>Connexion directe et Azure ExpressRoute

Azure fournit des connexions dédiées de site à site similaires via son service [ExpressRoute](https://azure.microsoft.com/documentation/services/expressroute/). ExpressRoute vous permet de connecter votre réseau local directement aux ressources Azure à l’aide d’une connexion de réseau privé dédiée. Azure offre également des [connexions VPN de site à site](https://azure.microsoft.com/documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/) plus conventionnelles à moindre coût.

#### <a name="see-also"></a>Voir aussi

-   [Créer un réseau virtuel au moyen du portail Azure](https://azure.microsoft.com/documentation/articles/virtual-networks-create-vnet-arm-pportal/)

-   [Planifier et concevoir des réseaux virtuels Azure](https://azure.microsoft.com/documentation/articles/virtual-network-vnet-plan-design-arm/)

-   [Bonnes pratiques en matière de sécurité du réseau Azure](https://azure.microsoft.com/documentation/articles/azure-security-network-security-best-practices/)

### <a name="database-services"></a>Services de base de données

#### <a name="rds-and-azure-relational-database-services"></a>Services de base de données relationnelle et RDS

Azure fournit plusieurs services de base de données relationnelle différents qui sont l’équivalent du RDS d’AWS.

-   [Base de données SQL](https://docs.microsoft.com/azure/sql-database/sql-database-technical-overview)
-   [Azure Database pour MySQL](https://docs.microsoft.com/azure/mysql/overview)
-   [Base de données Azure pour PostgreSQL](https://docs.microsoft.com/azure/postgresql/overview)

D’autres moteurs de base de données tels que [SQL Server](https://azure.microsoft.com/services/virtual-machines/sql-server/), [Oracle](https://azure.microsoft.com/campaigns/oracle/) et [MySQL](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-classic-mysql-2008r2/) peuvent être déployés à l’aide des instances de machines virtuelles Azure.

Les coûts pour RDS de AWS sont déterminés par la quantité de ressources matérielles utilisées par votre instance, tels que l’UC, la mémoire RAM, le stockage et la bande passante réseau. Dans les services de base de données Azure, le coût dépend de la taille de votre base de données, des connexions simultanées et des niveaux de débit.

#### <a name="see-also"></a>Voir aussi

-   [Didacticiels de Azure SQL Database](https://azure.microsoft.com/documentation/articles/sql-database-explore-tutorials/)

-   [Configurer la géoréplication pour Azure SQL Database avec le portail Azure](https://azure.microsoft.com/documentation/articles/sql-database-geo-replication-portal/)

-   [Présentation de Cosmos DB : une base de données NoSQL JSON](/azure/cosmos-db/sql-api-introduction)

-   [Utilisation du stockage Table Azure à partir de Node.js](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-table-storage/)

### <a name="security-and-identity"></a>Sécurité et identité

#### <a name="directory-service-and-azure-active-directory"></a>Service d’annuaire et Azure Active Directory

Azure répartit des services d’annuaire dans les offres suivantes :

-   [Azure Active Directory](https://azure.microsoft.com/documentation/articles/active-directory-whatis/) - service de gestion d’annuaires et d’identités basé sur le cloud.

-   [Azure Active Directory B2B](https://azure.microsoft.com/documentation/articles/active-directory-b2b-collaboration-overview/) - activer l’accès à vos applications d’entreprise à partir des identités gérées par les partenaires.

-   [Azure Active Directory B2C](https://azure.microsoft.com/documentation/articles/active-directory-b2c-overview/) - service offrant la prise en charge de la gestion des utilisateurs et de l’authentification unique pour les applications accessibles aux clients.

-   [Azure Active Directory Domain Services](https://azure.microsoft.com/documentation/articles/active-directory-ds-overview/) -service de contrôleur de domaine hébergé, offrant des fonctionnalités de gestion d’utilisateurs et de jonction de domaine compatibles avec Active Directory.

#### <a name="web-application-firewall"></a>Pare-feu d’application web

En plus du [Pare-feu d’applications web sur Application Gateway](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/), vous pouvez également [utiliser des pare-feu d’applications web](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/) de fournisseurs tiers comme [Barracuda Networks](https://azure.microsoft.com/marketplace/partners/barracudanetworks/waf/).

#### <a name="see-also"></a>Voir aussi

-   [Prise en main de la sécurité de Microsoft Azure](https://azure.microsoft.com/documentation/articles/azure-security-getting-started/)

-   [Bonnes pratiques en matière de sécurité du contrôle d’accès et de la gestion des identités Azure](https://azure.microsoft.com/documentation/articles/azure-security-identity-management-best-practices/)

### <a name="application-and-messaging-services"></a>Applications et services de messagerie

#### <a name="simple-email-service"></a>Simple Email Service

AWS fournit Simple Email Service (SES) pour l’envoi de notification, et d’e-mails transactionnels ou de marketing. Dans Azure, des solutions tierces telles que [Sendgrid](https://sendgrid.com/partners/azure/) fournissent des services de messagerie.

#### <a name="simple-queueing-service"></a>Service de mise en file d’attente simple

Simple Queueing Service (SQS) de AWS fournit un système de messagerie pour connecter des applications, services et appareils au sein de la plate-forme AWS. Azure possède deux services fournissant des fonctionnalités similaires :

-   [Stockage de files d’attente](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/) - un service de messagerie en cloud qui permet la communication entre des composants d’application au sein de la plateforme Azure.

-   [Service Bus](https://azure.microsoft.com/services/service-bus/) -un système de messagerie plus robuste pour connecter des applications, services et appareils. À l’aide du [Service Bus Relay](https://docs.microsoft.com/azure/service-bus-relay/relay-what-is-it) concerné, Service Bus peut également se connecter aux services et applications hébergées à distance.

#### <a name="device-farm"></a>Batterie d’appareils

La batterie d’appareils de AWS fournit des services de tests inter-périphériques. Dans Azure, [Xamarin Test Cloud](https://www.xamarin.com/test-cloud) fournit un test frontal entre les périphériques semblables pour les appareils mobiles.

En plus des tests frontaux, [Azure DevTest Labs](https://azure.microsoft.com/services/devtest-lab/) fournit des ressources de test principales pour les environnements Linux et Windows.

#### <a name="see-also"></a>Voir aussi

-   [Utilisation du stockage de files d'attente à partir de Node.js](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/)

-   [Utilisation des files d’attente Service Bus](https://azure.microsoft.com/documentation/articles/service-bus-nodejs-how-to-use-queues/)

### <a name="analytics-and-big-data"></a>Analytics et Big data

[Cortana Intelligence Suite](https://azure.microsoft.com/suites/cortana-intelligence-suite/) est le package Azure de produits et de services conçus pour capturer, organiser, analyser et visualiser de grandes quantités de données. La suite Cortana comprend les services suivants :

-   [HDInsight](https://azure.microsoft.com/documentation/services/hdinsight/) - distribution Apache gérée et comprenant Hadoop, Spark, Storm ou HBase.

-   [Data Factory](https://azure.microsoft.com/documentation/services/data-factory/) - fournit des fonctionnalités de pipeline de données et d’orchestration de données.

-   [SQL Data Warehouse](https://azure.microsoft.com/documentation/services/sql-data-warehouse/) -stockage de données relationnelles à grande échelle.

-   [Data Lake Store](https://azure.microsoft.com/documentation/services/data-lake-store/) - stockage à grande échelle optimisé pour les charges de travail analytique de Big data.

-   [Machine Learning](https://azure.microsoft.com/documentation/services/machine-learning/) - utilisé pour générer et appliquer une analyse prédictive sur des données.

-   [Stream Analytics](https://azure.microsoft.com/documentation/services/stream-analytics/) - analyse de données en temps réel.

-   [Data Lake Analytics](https://azure.microsoft.com/documentation/articles/data-lake-analytics-overview/) - service d’analyse à grande échelle optimisé pour fonctionner avec Data Lake Store

-   [PowerBI](https://powerbi.microsoft.com/) - utilisé pour alimenter la visualisation des données.

#### <a name="see-also"></a>Voir aussi

-   [Galerie Cortana Intelligence](https://gallery.cortanaintelligence.com/)

-   [Présentation des solutions de Big Data Microsoft](https://msdn.microsoft.com/library/dn749804.aspx)

-   [Azure Data Lake & Azure HDInsight Blog](https://blogs.msdn.microsoft.com/azuredatalake/)

### <a name="internet-of-things"></a>Internet des Objets

#### <a name="see-also"></a>Voir aussi

-   [Mise en route de Azure IoT Hub](https://azure.microsoft.com/documentation/articles/iot-hub-csharp-csharp-getstarted/)

-   [Comparaison entre IoT Hub et Event Hubs](https://azure.microsoft.com/documentation/articles/iot-hub-compare-event-hubs/)

### <a name="mobile-services"></a>Services mobiles

#### <a name="notifications"></a>Notifications

Notification Hubs ne prend pas en charge l’envoi de messages SMS ou de courriers électroniques, des services tiers sont donc nécessaires pour ces types d’envoi.

#### <a name="see-also"></a>Voir aussi

-   [Créer une application Android](https://azure.microsoft.com/documentation/articles/app-service-mobile-android-get-started/)

-   [Authentification et autorisation dans Azure Mobile Apps](https://azure.microsoft.com/documentation/articles/app-service-mobile-auth/)

-   [Envoi de notifications Push avec Azure Notification Hubs](https://azure.microsoft.com/documentation/articles/notification-hubs-android-push-notification-google-fcm-get-started/)

### <a name="management-and-monitoring"></a>Gestion et surveillance

#### <a name="see-also"></a>Voir aussi
-   [Guide de surveillance et de diagnostic](https://azure.microsoft.com/documentation/articles/best-practices-monitoring/)

-   [Meilleures pratiques relatives à la création de modèles Azure Resource Manager](https://azure.microsoft.com/documentation/articles/resource-manager-template-best-practices/)

-   [Modèles de démarrage rapide de Azure Resource Manager](https://azure.microsoft.com/documentation/templates/)


## <a name="next-steps"></a>Étapes suivantes

-   [Vue d’ensemble interactif de la plateforme Azure](http://azureplatform.azurewebsites.net/)

-   [Prise en main d’Azure](https://azure.microsoft.com/get-started/)

-   [Architectures de solution Azure](https://azure.microsoft.com/solutions/architecture/)

-   [Architectures de référence Azure](https://azure.microsoft.com/documentation/articles/guidance-architecture/)

-   [Cours en ligne gratuit : Microsoft Azure pour les experts de AWS](http://aka.ms/azureforaws)


<!-- links -->

[paired-regions]: https://azure.microsoft.com/documentation/articles/best-practices-availability-paired-regions/
[traffic-manager]: /azure/traffic-manager/
