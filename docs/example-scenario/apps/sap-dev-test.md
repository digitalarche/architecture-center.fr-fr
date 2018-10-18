---
title: Environnements de développement/test pour les charges de travail SAP sur Azure
description: Créez un environnement de développement/test pour les charges de travail SAP.
author: AndrewDibbins
ms.date: 7/11/18
ms.openlocfilehash: b47e4cb527d3e4ecd74bee7bcf08f2794da56d6c
ms.sourcegitcommit: 62945777e519d650159f0f963a2489b6bb6ce094
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2018
ms.locfileid: "48876787"
---
# <a name="devtest-environments-for-sap-workloads-on-azure"></a>Environnements de développement/test pour les charges de travail SAP sur Azure

Cet exemple montre comment créer un environnement de développement/test pour SAP NetWeaver dans un environnement Windows ou Linux sur Azure. La base de données utilisée est AnyDB, le terme SAP pour tout SGBD pris en charge (qui n’est pas SAP HANA). Étant donné que cette architecture est conçue pour les environnements hors production, elle est déployée avec une seule machine virtuelle et sa taille peut être modifiée pour prendre en compte les besoins de votre organisation.

Pour les cas d'usage de production, examinez les architectures de référence SAP disponibles ci-dessous :

* [SAP NetWeaver pour AnyDB][sap-netweaver]
* [SAP S/4HANA][sap-hana]
* [SAP sur des Instances de grande taille Azure][sap-large]

## <a name="relevant-use-cases"></a>Cas d’utilisation appropriés

Pensez à ce scénario pour les cas d’usage suivants :

* Charges de travail SAP non productives non critiques (bac à sable, développement, test, assurance qualité)
* Charges de travail SAP Business non critiques

## <a name="architecture"></a>Architecture

![Diagramme d’architecture pour les environnements de développement/test des charges de travail SAP](media/architecture-sap-dev-test.png)

Ce scénario illustre le provisionnement d’une base de données système SAP et d’un serveur d’application SAP uniques sur une seule machine virtuelle. Les données circulent dans le scénario comme suit :

1. Les clients utilisent l’interface utilisateur SAP ou d’autres outils clients (Excel, un navigateur Web ou une autre application Web) pour accéder au système SAP basé sur Azure.
2. La connectivité est assurée par une connexion ExpressRoute établie qui se termine dans Azure au niveau de la passerelle ExpressRoute. Le trafic réseau est acheminé via la passerelle ExpressRoute vers le sous-réseau de passerelle et depuis le sous-réseau de passerelle vers le sous-réseau spoke de niveau application (voir le modèle [hub-spoke][hub-spoke]) et via une passerelle de sécurité réseau vers la machine virtuelle d’applications SAP.
3. Les serveurs de gestion d’identité fournissent des services d’authentification.
4. Le serveur jumpbox offre des fonctionnalités de gestion locale.

### <a name="components"></a>Composants

* Les [réseaux virtuels](/azure/virtual-network/virtual-networks-overview) constituent la base de la communication réseau dans Azure.
* Les [machines virtuelles Azure](/azure/virtual-machines/windows/overview) fournissent une infrastructure sécurisée et virtualisée à la demande et à grande échelle avec un serveur Windows ou Linux.
* [ExpressRoute](/azure/expressroute/expressroute-introduction) vous permet d’étendre vos réseaux locaux au cloud de Microsoft via une connexion privée assurée par un fournisseur de connectivité.
* Les [groupes de sécurité réseau](/azure/virtual-network/security-overview) vous permettent de limiter le trafic réseau vers les ressources d’un réseau virtuel. Un groupe de sécurité réseau contient une liste de règles de sécurité qui autorisent ou refusent le trafic réseau entrant ou sortant en fonction de l’adresse IP source ou de destination, du port et du protocole. 
* Les [groupes de ressources](/azure/azure-resource-manager/resource-group-overview#resource-groups) jouent le rôle de conteneurs logiques pour des ressources Azure.

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

Pour vous aider à explorer le coût d’exécution de ce scénario, tous les services sont préconfigurés dans les exemples du calculateur de coûts ci-après. Pour pouvoir observer l’évolution de la tarification pour votre cas d’usage particulier, modifiez les variables appropriées en fonction du trafic que vous escomptez.

Nous proposons quatre exemples de profils de coût basés sur la quantité de trafic que vous prévoyez de recevoir :

|Taille|SAP|Type de machine virtuelle|Stockage|Calculatrice de tarification Azure|
|----|----|-------|-------|---------------|
|Petite|8000|D8s_v3|2xP20, 1xP10|[Petite](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1)|
|Moyenne|16000|D16s_v3|3xP20, 1xP10|[Moyenne](https://azure.com/e/465bd07047d148baab032b2f461550cd)|
grand|32000|E32s_v3|3xP20, 1xP10|[Grande](https://azure.com/e/ada2e849d68b41c3839cc976000c6931)|
Très grande|64 000|M64s|4xP20, 1xP10|[Très grande](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef)|

> [!NOTE]
> Cette tarification, fournie à titre informatif, indique uniquement les coûts relatifs au stockage et aux machines virtuelles. Elle ne tient pas compte de la mise en réseau, du stockage de sauvegarde et des frais d’entrée/de sortie de données.

* [Petit](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1) : machine virtuelle de type D8s_v3 avec 8 processeurs virtuels, 32 Go de RAM et 200 Go de stockage temporaire, en plus de deux disques de Stockage Premium de 512 Go et un de 128 Go.
* [Moyen](https://azure.com/e/465bd07047d148baab032b2f461550cd) : machine virtuelle de type D16s_v3 avec 16 processeurs virtuels, 64 Go de RAM et 400 Go de stockage temporaire, en plus de trois disques de Stockage Premium de 512 Go et un de 128 Go.
* [Grand](https://azure.com/e/ada2e849d68b41c3839cc976000c6931) : machine virtuelle de type E32s_v3 avec 32 processeurs virtuels, 256 Go de RAM et 512 Go de stockage temporaire, en plus de trois disques de Stockage Premium de 512 Go et un de 128 Go.
* [Très grand](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef) : machine virtuelle de type M64s avec 64 processeurs virtuels, 1 024 Go de RAM et 2 000 Go de stockage temporaire, en plus de quatre disques de Stockage Premium de 512 Go et un de 128 Go.

## <a name="deployment"></a>Déploiement

Cliquez ici pour déployer l’infrastructure sous-jacente à ce scénario.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fsap-2tier%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

> [!NOTE]
> SAP et Oracle ne sont pas installés au cours de ce déploiement. Vous devez déployer ces composants séparément.

<!-- links -->
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[sap-netweaver]: /azure/architecture/reference-architectures/sap/sap-netweaver
[sap-hana]: /azure/architecture/reference-architectures/sap/sap-s4hana
[sap-large]: /azure/architecture/reference-architectures/sap/hana-large-instances
[hub-spoke]: /azure/architecture/reference-architectures/hybrid-networking/hub-spoke
