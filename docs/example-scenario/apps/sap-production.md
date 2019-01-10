---
title: Exécution des charges de travail de production SAP avec une base de données Oracle
titleSuffix: Azure Example Scenarios
description: Exécutez un déploiement de production SAP dans Azure à l’aide d’une base de données Oracle.
author: DharmeshBhagat
ms.date: 09/12/2018
ms.custom: fasttrack
ms.openlocfilehash: 02a6eb43d3e11604857b8bd1f461c22a48f655c7
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/08/2019
ms.locfileid: "54110925"
---
# <a name="running-sap-production-workloads-using-an-oracle-database-on-azure"></a>Exécution des charges de travail de production SAP à l’aide d’Oracle Database sur Azure

Les systèmes SAP sont utilisés pour exécuter des applications métier stratégiques. Toute panne perturbe les processus clés et peut entraîner une augmentation des dépenses ou des pertes de revenus. Pour éviter ce genre de situation, une infrastructure SAP hautement disponible et résiliente en cas de pannes est nécessaire.

La création d’un environnement SAP hautement disponible requiert de supprimer les points de défaillance uniques des processus et de l’architecture de votre système. Les points de défaillance uniques peuvent être liés à des pannes de site, à des erreurs au niveau des composants système ou encore à des erreurs humaines.

Cet exemple de scénario illustre un déploiement SAP sur des machines virtuelles Windows ou Linux sur Azure, avec une base de données Oracle HA. Pour votre déploiement SAP, vous pouvez utiliser des machines virtuelles de diverses tailles en fonction de vos besoins.

## <a name="relevant-use-cases"></a>Cas d’usage appropriés

Les autres cas d’usage appropriés sont les suivants :

- Charges de travail stratégiques s’exécutant sur SAP.
- Charges de travail SAP non stratégiques.
- Environnements de test pour SAP qui simulent un environnement à haute disponibilité.

## <a name="architecture"></a>Architecture

![Vue d’ensemble de l’architecture d’un environnement de production SAP dans Azure][architecture]

Cet exemple inclut une configuration à haute disponibilité pour une base de données Oracle, des services centraux SAP et plusieurs serveurs d’application SAP s’exécutant sur différentes machines virtuelles. Le réseau Azure utilise une [topologie hub-and-spoke](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke) pour des raisons de sécurité. Les données circulent dans la solution comme suit :

1. Les utilisateurs accèdent au système SAP via l’interface utilisateur SAP, un navigateur web ou d’autres outils clients comme Microsoft Excel. Une connexion ExpressRoute fournit un accès depuis le réseau local d’une organisation aux ressources s’exécutant dans Azure.
2. La connexion ExpressRoute se termine dans Azure au niveau de la passerelle de réseau virtuel ExpressRoute. Le trafic réseau est acheminé vers un sous-réseau de passerelle via la passerelle ExpressRoute créée dans le réseau virtuel hub.
3. Le réseau virtuel hub est associé à un réseau virtuel spoke. Le sous-réseau de couche Application héberge les machines virtuelles exécutant SAP dans un groupe à haute disponibilité.
4. Les serveurs de gestion d’identité fournissent des services d’authentification pour la solution.
5. Le serveur de rebond est utilisé par les administrateurs système pour gérer de façon sécurisée les ressources déployées dans Azure.

### <a name="components"></a>Composants

- Les [réseaux virtuels](/azure/virtual-network/virtual-networks-overview) sont utilisés dans ce scénario pour créer une topologie hub-and-spoke virtuelle dans Azure.

- Les [machines virtuelles](/azure/virtual-machines/windows/overview) fournissent les ressources de calcul pour chaque niveau de la solution. Chaque cluster de machine virtuelle est configuré comme un [groupe à haute disponibilité](/azure/virtual-machines/windows/regions-and-availability#availability-sets).

- [ExpressRoute](/azure/expressroute/expressroute-introduction) étend votre réseau local au cloud Microsoft via une connexion privée établie par un fournisseur de connectivité.

- Les [groupes de sécurité réseau](/azure/virtual-network/security-overview) limitent l’accès réseau aux ressources d’un réseau virtuel. Un groupe de sécurité réseau contient une liste de règles de sécurité qui autorisent ou refusent le trafic réseau en fonction de l’adresse IP source ou de destination, du port et du protocole.

- Les [groupes de ressources](/azure/azure-resource-manager/resource-group-overview#resource-groups) jouent le rôle de conteneurs logiques pour des ressources Azure.

### <a name="alternatives"></a>Autres solutions

SAP fournit des options flexibles pour différentes combinaisons de système d’exploitation, de gestionnaire de bases de données et de types de machines virtuelles dans un environnement Azure. Pour plus d’informations, consultez la [note SAP 1928533](https://launchpad.support.sap.com/#/notes/1928533) « SAP Applications on Azure: Supported Products and Azure VM Types (Applications SAP sur Azure : produits et types de machines virtuelles pris en charge).

## <a name="considerations"></a>Considérations

- Les pratiques recommandées sont définies pour la création d’environnements SAP hautement disponibles dans Azure. Pour plus d’informations, consultez [Scénarios et architecture de haute disponibilité pour SAP NetWeaver](/azure/virtual-machines/workloads/sap/sap-high-availability-architecture-scenarios). Consultez aussi [Haute disponibilité des applications SAP sur des machines virtuelles Azure](/azure/virtual-machines/workloads/sap/high-availability-guide).

- Les bases de données Oracle ont également des pratiques recommandées pour Azure. Pour plus d’informations, consultez [Concevoir et implémenter une base de données Oracle dans Azure](/azure/virtual-machines/workloads/oracle/oracle-design).

- Oracle Data Guard est utilisé pour supprimer les points de défaillance uniques des bases de données Oracle stratégiques. Pour plus d’informations, consultez [Implémenter Oracle Data Guard sur une machine virtuelle Azure Linux](/azure/virtual-machines/workloads/oracle/configure-oracle-dataguard).

- Microsoft Azure offre des services d’infrastructure qui peuvent être utilisés pour déployer des produits SAP avec une base de données Oracle. Pour plus d’informations, consultez [Déploiement SGBD de machines virtuelles Oracle Azure pour charge de travail SAP](/azure/virtual-machines/workloads/sap/dbms_guide_oracle).

## <a name="pricing"></a>Tarifs

Pour vous aider à explorer le coût d’exécution de ce scénario, tous les services sont préconfigurés dans les exemples du calculateur de coûts ci-après. Pour pouvoir observer l’évolution de la tarification pour votre cas d’usage particulier, modifiez les variables appropriées en fonction du trafic que vous escomptez.

Nous proposons quatre exemples de profils de coût basés sur la quantité de trafic que vous prévoyez de recevoir :

|Taille|SAP|Type de machine virtuelle de base de données|Stockage de base de données|Machine virtuelle (A)SCS|Stockage (A)SCS|Type de machine virtuelle d’application|Stockage d’application|Calculatrice de tarification Azure|
|----|----|-------|-------|-----|---|---|--------|---------------|
|Petite|30000|DS13_v2|4xP20, 1xP20|DS11_v2|1x P10|DS13_v2|1x P10|[Petite](https://azure.com/e/45880ba0bfdf47d497851a7cf2650c7c)|
|Moyenne|70000|DS14_v2|6xP20, 1xP20|DS11_v2|1x P10|4x DS13_v2|1x P10|[Moyenne](https://azure.com/e/9a523f79591347ca9a48c3aaa1406f8a)|
grand|180000|E32s_v3|5xP30, 1xP20|DS11_v2|1x P10|6x DS14_v2|1x P10|[Grande](https://azure.com/e/f70fccf571e948c4b37d4fecc07cbf42)|
Très grande|250 000|M64s|6xP30, 1xP30|DS11_v2|1x P10|10x DS14_v2|1x P10|[Très grande](https://azure.com/e/58c636922cf94faf9650f583ff35e97b)|

> [!NOTE]
> Cette tarification est fournie à titre informatif et indique uniquement les coûts relatifs au stockage et aux machines virtuelles. Elle ne tient pas compte des frais associés à la mise en réseau, au stockage de sauvegarde et à l’entrée/la sortie des données.

- [Petit](https://azure.com/e/45880ba0bfdf47d497851a7cf2650c7c) : Un petit système se compose d’une machine virtuelle de type DS13_v2 pour le serveur de base de données avec 8 processeurs virtuels, 56 Go de RAM et 112 Go de stockage temporaire, en plus de cinq disques de stockage Premium de 512 Go. Un serveur d’instance centrale SAP utilisant une machine virtuelle de type DS11_v2 avec 2 processeurs virtuels, 14 Go de RAM et 28 Go de stockage temporaire. Une machine virtuelle de type DS13_v2 pour le serveur d’application SAP avec 8 processeurs virtuels, 56 Go de RAM et 400 Go de stockage temporaire, en plus d’un disque de stockage Premium de 128 Go.

- [Moyen](https://azure.com/e/9a523f79591347ca9a48c3aaa1406f8a) : Un système moyen se compose d’une machine virtuelle de type DS14_v2 pour le serveur de base de données avec 16 processeurs virtuels, 112 Go de RAM et 800 Go de stockage temporaire, en plus de sept disques de stockage Premium de 512 Go. Un serveur d’instance centrale SAP utilisant une machine virtuelle de type DS11_v2 avec 2 processeurs virtuels, 14 Go de RAM et 28 Go de stockage temporaire. Quatre machines virtuelles de type DS13_v2 pour le serveur d’application SAP avec 8 processeurs virtuels, 56 Go de RAM et 400 Go de stockage temporaire, en plus d’un disque de stockage Premium de 128 Go.

- [Grand](https://azure.com/e/f70fccf571e948c4b37d4fecc07cbf42) : Un grand système se compose d’une machine virtuelle de type E32s_v3 pour le serveur de base de données avec 32 processeurs virtuels, 256 Go de RAM et 800 Go de stockage temporaire, en plus de trois disques de stockage Premium de 512 Go et d’un disque de stockage Premium de 128 Go. Un serveur d’instance centrale SAP utilisant une machine virtuelle de type DS11_v2 avec 2 processeurs virtuels, 14 Go de RAM et 28 Go de stockage temporaire. Six machines virtuelles de type DS14_v2 pour les serveurs d’application SAP avec 16 processeurs virtuels, 112 Go de RAM et 224 Go de stockage temporaire, en plus de six disques de stockage Premium de 128 Go.

- [Très grand](https://azure.com/e/58c636922cf94faf9650f583ff35e97b) : Un très grand système se compose d’une machine virtuelle de type M64s pour le serveur de base de données avec 64 processeurs virtuels, 1 024 Go de RAM et 2 000 Go de stockage temporaire, en plus de sept disques de stockage Premium de 1 024 Go. Un serveur d’instance centrale SAP utilisant une machine virtuelle de type DS11_v2 avec 2 processeurs virtuels, 14 Go de RAM et 28 Go de stockage temporaire. 10 machines virtuelles de type DS14_v2 pour les serveurs d’application SAP avec 16 processeurs virtuels, 112 Go de RAM et 224 Go de stockage temporaire, en plus de dix disques de stockage Premium de 128 Go.

## <a name="deployment"></a>Déploiement

Utilisez le lien suivant pour déployer l’infrastructure sous-jacente de ce scénario.

<!-- markdownlint-disable MD033 -->

<a
href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fsap-3tier-distributed-ora%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

<!-- markdownlint-enable MD033 -->

> [!NOTE]
> SAP et Oracle ne sont pas installés au cours de ce déploiement. Vous devez déployer ces composants séparément.

## <a name="related-resources"></a>Ressources associées

Pour en savoir plus sur l’exécution de charges de travail de production SAP dans Azure, documentez-vous sur les architectures de référence suivantes :

- [Déployer SAP NetWeaver (Windows) pour AnyDB sur des machines virtuelles Azure](/azure/architecture/reference-architectures/sap/sap-netweaver)
- [SAP S/4HANA pour machines virtuelles Linux sur Azure](/azure/architecture/reference-architectures/sap/sap-s4hana)
- [Exécution de SAP HANA sur les grandes instances Azure](/azure/architecture/reference-architectures/sap/hana-large-instances)

<!-- links -->
[architecture]: media/architecture-sap-production.png
