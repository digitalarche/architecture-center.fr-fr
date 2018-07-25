---
title: SAP pour des charges de travail de développement/test
description: Scénario SAP pour un environnement de développement/test
author: AndrewDibbins
ms.date: 7/11/18
ms.openlocfilehash: 675a5cb4b1ee4001ca50d24c145ce1a177f90da4
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060960"
---
# <a name="sap-for-devtest-workloads"></a>SAP pour des charges de travail de développement/test

Cet exemple fournit des conseils pour savoir comment exécuter une implémentation de développement/test de SAP NetWeaver dans un environnement Windows ou Linux sur Azure. La base de données utilisée est AnyDB, le terme SAP pour tout SGBD pris en charge (qui n’est pas SAP HANA). Étant donné que cette architecture est conçue pour les environnements hors production, elle est déployée avec une seule machine virtuelle et sa taille peut être modifiée pour prendre en compte les besoins de votre organisation.

Pour les cas d'usage de production, examinez les architectures de référence SAP disponibles ci-dessous :

* [SAP NetWeaver pour AnyDB][sap-netweaver]
* [SAP S/4Hana][sap-hana]
* [SAP sur des Instances de grande taille Azure][sap-large]

## <a name="related-use-cases"></a>Cas d’usage connexes

Pensez à ce scénario pour les cas d’usage suivants :

* Charges de travail SAP non productives non critiques (bac à sable, développement, test, assurance qualité)
* Charges de travail SAP Professionnel non critique

## <a name="architecture"></a>Architecture

![Diagramme](media/sap-2tier/SAP-Infra-2Tier_finalversion.png)

Ce scénario couvre la fourniture d’une base de données du système SAP unique et un serveur d’application SAP sur une machine virtuelle unique, les données transitent dans le scénario comme suit :

1. Les clients à partir de la couche Présentation utilisent leur GUI SAP ou d’autres interfaces utilisateur (Internet Explorer, Excel ou une autre application web) en local pour accéder à un système Azure basé sur SAP.
2. La connectivité est fournie via l’utilisation de l’ExpressRoute établie. L’ExpressRoute est arrêtée dans Azure au niveau de la passerelle ExpressRoute. Le trafic réseau est acheminé via la passerelle ExpressRoute vers le sous-réseau de passerelle et depuis le sous-réseau de passerelle vers le sous-réseau Spoke de niveau Application (voir le modèle [hub-spoke][hub-spoke]) et via une passerelle de sécurité réseau vers la machine virtuelle d’applications SAP.
3. Les serveurs de gestion d’identité fournissent des services d’authentification.
4. La Jump Box fournit des fonctionnalités de gestion locale.

### <a name="components"></a>Composants

* Les [groupes de ressources](/azure/azure-resource-manager/resource-group-overview#resource-groups) sont des conteneurs logiques pour des ressources Azure.
* Les [réseaux virtuels](/azure/virtual-network/virtual-networks-overview) constituent la base des communications réseau dans Azure
* Les [machines virtuelles Azure](/azure/virtual-machines/windows/overview) fournissent une infrastructure sécurisée et virtualisée à la demande et à grande échelle avec un serveur Windows ou Linux
* [ExpressRoute](/azure/expressroute/expressroute-introduction) vous permet d’étendre vos réseaux locaux au cloud de Microsoft via une connexion privée assurée par un fournisseur de connectivité.
* Les [groupes de sécurité réseau](/azure/virtual-network/security-overview) vous permettent de limiter le trafic réseau vers les ressources d’un réseau virtuel. Un groupe de sécurité réseau contient une liste de règles de sécurité qui autorisent ou refusent le trafic réseau entrant ou sortant en fonction de l’adresse IP source ou de destination, du port et du protocole. 

## <a name="considerations"></a>Considérations

### <a name="availability"></a>Disponibilité

 Microsoft propose un contrat de niveau de service (SLA) pour les instances de machine virtuelle uniques. Pour plus d’informations sur le contrat de niveau de service Microsoft Azure pour les machines virtuelles [Contrat SLA pour les machines virtuelles](https://azure.microsoft.com/support/legal/sla/virtual-machines)

### <a name="scalability"></a>Extensibilité

Pour obtenir des conseils d’ordre général sur la conception de solutions évolutives, consultez la [liste de contrôle de l’extensibilité][scalability] dans le Centre des architectures Azure.

### <a name="security"></a>Sécurité

Pour obtenir des conseils d’ordre général sur la conception de solutions sécurisées, consultez la [documentation sur la sécurité Azure][security].

### <a name="resiliency"></a>Résilience

Pour obtenir des conseils d’ordre général sur la conception de solutions résilientes, consultez l’article [Conception d’applications résilientes pour Azure][resiliency].

## <a name="pricing"></a>Tarifs

Explorez le coût d’exécution de ce scénario, tous les services sont préconfigurés dans le calculateur de coûts.  Pour pouvoir observer l’évolution de la tarification pour votre cas d’usage particulier, modifiez les variables appropriées en fonction du trafic que vous escomptez.

Nous proposons quatre exemples de profils de coût basés sur la quantité de trafic que vous escomptez :

|Taille|SAP|Type de machine virtuelle|Stockage|Calculatrice de tarification Azure|
|----|----|-------|-------|---------------|
|Petite|8000|D8s_v3|2xP20, 1xP10|[Petite](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1)|
|Moyenne|16000|D16s_v3|3xP20, 1xP10|[Moyenne](https://azure.com/e/465bd07047d148baab032b2f461550cd)|
grand|32000|E32s_v3|3xP20, 1xP10|[Grande](https://azure.com/e/ada2e849d68b41c3839cc976000c6931)|
Très grande|64 000|M64s|4xP20, 1xP10|[Très grande](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef)|

Remarque : la tarification est un guide et indique uniquement les machines virtuelles et les coûts de stockage (exclut les frais de la mise en réseau, du stockage des sauvegardes et des entrées/sorties de données).

* [Petite](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1) : un petit système se compose d’une machine virtuelle du type D8s_v3 avec 8 x vCPUs, 32 Go de RAM et un stockage temporaire de 200 Go, en plus de deux disques de stockage premium de 512 Go et un de 128 Go.
* [Moyenne](https://azure.com/e/465bd07047d148baab032b2f461550cd) : un système moyen se compose d’une machine virtuelle du type D16s_v3 avec 16 x vCPUs, 64 Go de RAM et un stockage temporaire de 400 Go, en plus de trois disques de stockage premium de 512 Go et un de 128 Go.
* [Grande](https://azure.com/e/ada2e849d68b41c3839cc976000c6931) : un grand système se compose d’une machine virtuelle du type E32s_v3 avec 32 x vCPUs, 256 Go de RAM et un stockage temporaire de 512 Go, en plus de trois disques de stockage premium de 512 Go et un de 128 Go.
* [Très grande](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef) : un très grand système se compose d’une machine virtuelle du type M64s avec 64 x vCPUs, 1024 Go de RAM et un stockage temporaire de 2000 Go, en plus de quatre disques de stockage premium de 512 Go et un de 128 Go.

## <a name="deployment"></a>Déploiement

Pour déployer l’infrastructure sous-jacente similaire au scénario ci-dessus, veuillez utiliser le bouton Déployer

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fsap-2tier%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

\* SAP ne sera pas installé, vous devrez le faire une fois l’infrastructure construite manuellement.

<!-- links -->
[reference architecture]:  /azure/architecture/reference-architectures/sap
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[sap-netweaver]: /azure/architecture/reference-architectures/sap/sap-netweaver
[sap-hana]: /azure/architecture/reference-architectures/sap/sap-s4hana
[sap-large]: /azure/architecture/reference-architectures/sap/hana-large-instances
[hub-spoke]: /azure/architecture/reference-architectures/hybrid-networking/hub-spoke