---
title: Exécuter une machine virtuelle Windows dans Azure
description: Découvrez comment exécuter une machine virtuelle Windows dans Azure, en privilégiant l’extensibilité, la résilience, la facilité de gestion et la sécurité.
author: telmosampaio
ms.date: 04/03/2018
pnp.series.title: Windows VM workloads
pnp.series.next: multi-vm
pnp.series.prev: ./index
ms.openlocfilehash: 9bc0b4af56b9194fd1bec8a189c86963ad2b0c98
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/06/2018
---
# <a name="run-a-windows-vm-on-azure"></a>Exécuter une machine virtuelle Windows dans Azure

Cette architecture de référence présente un ensemble de pratiques éprouvées pour l’exécution d’une machine virtuelle Windows dans Azure. Elle contient des recommandations pour le provisionnement de la machine virtuelle et des composants réseau et de stockage. Cette architecture peut être utilisée pour exécuter une instance unique de machine virtuelle et sert de base aux architectures plus complexes telles que les applications multiniveaux. [**Déployez cette solution.**](#deploy-the-solution)

![[0]][0]

*Téléchargez un [fichier Visio][visio-download] contenant ce diagramme d’architecture.*

## <a name="architecture"></a>Architecture

L’approvisionnement d’une machine virtuelle Azure requiert des composants supplémentaires en plus de la machine virtuelle elle-même, notamment les ressources de mise en réseau et de stockage.

* **Groupe de ressources.** Un [groupe de ressources][resource-manager-overview] est un conteneur logique qui héberge les ressources Azure associées. En règle générale, regroupez les ressources en fonction de leur durée de vie et de qui va les gérer. 

* **Machine virtuelle**. Vous pouvez approvisionner une machine virtuelle issue d’une liste d’images publiées, d’une image managée personnalisée ou d’un fichier de disque dur virtuel (VHD) chargé(e) dans Stockage Blob Azure.

* **Disques managés**. [Les disques managés Azure][managed-disks] simplifient la gestion des disques en gérant le stockage pour vous. Le disque du système d’exploitation est un disque dur virtuel stocké dans [Stockage Azure][azure-storage], donc il persiste même lorsque l’ordinateur hôte est arrêté. Nous vous recommandons également de créer un ou plusieurs [disques de données][data-disk], qui sont des disques durs virtuels persistants utilisés pour les données d’application.

* **Disque temporaire** La machine virtuelle est créée avec un disque temporaire (le lecteur `D:` sous Windows). Ce disque est stocké sur un lecteur physique de l’ordinateur hôte. Il n’est *pas* enregistré dans Stockage Azure et peut être supprimé lors des redémarrages ou d’autres événements de cycle de vie de la machine virtuelle. N’utilisez ce disque que pour des données temporaires, telles que des fichiers de pagination ou d’échange.

* **Réseau virtuel (VNet).** Chaque machine virtuelle Azure est déployée dans un réseau virtuel qui peut être segmenté en plusieurs sous-réseaux.

* **Interfaces réseau (NIC)**. La carte d’interface réseau permet à la machine virtuelle de communiquer avec le réseau virtuel.  

* **Adresse IP publique.** Une adresse IP publique est nécessaire pour communiquer avec la machine virtuelle &mdash; par exemple via le protocole RDP.  

* **Azure DNS**. [Azure DNS][azure-dns] est un service d’hébergement pour les domaines DNS qui offre une résolution de noms à l’aide de l’infrastructure Microsoft Azure. En hébergeant vos domaines dans Azure, vous pouvez gérer vos enregistrements DNS avec les mêmes informations d’identification, les mêmes API, les mêmes outils et la même facturation que vos autres services Azure.

* **Groupe de sécurité réseau**. Les [groupes de sécurité réseau][nsg] sont utilisés pour autoriser ou refuser le trafic réseau vers des machines virtuelles. Les groupes de sécurité réseau peuvent être associés à des sous-réseaux ou à des instances de machine virtuelle individuelles. 

* **Diagnostics.** La journalisation des diagnostics est essentielle à la gestion et au dépannage de la machine virtuelle.

## <a name="recommendations"></a>Recommandations

Cette architecture présente les recommandations de base pour l’exécution d’une machine virtuelle Windows dans Azure. Nous vous déconseillons toutefois d’utiliser une seule et même machine virtuelle pour les charges de travail critiques, car ce faisant, un point de défaillance unique est créé. Pour augmenter la disponibilité, déployez au moins deux machines virtuelles soumises à l’équilibrage de charge. Pour plus d’informations, voir [Running multiple VMs on Azure (Exécution de plusieurs machines virtuelles sur Azure)][multi-vm].

### <a name="vm-recommendations"></a>Recommandations pour les machines virtuelles

Azure propose de nombreuses tailles de machines virtuelles. Pour plus d’informations, consultez [Tailles des machines virtuelles sur Azure][virtual-machine-sizes]. Si vous déplacez une charge de travail vers Azure, commencez par choisir la taille de machine virtuelle qui correspond le mieux à vos serveurs locaux. Mesurez ensuite les performances de votre charge de travail réelle en termes de processeur, de mémoire et d’opérations d’entrée/sortie par seconde du disque, puis ajustez la taille selon vos besoins. Si vous avez besoin de plusieurs cartes réseau (NIC) pour votre machine virtuelle, notez que le nombre maximal de cartes NIC est défini pour chaque [taille de machine virtuelle][vm-size-tables].

En général, choisissez une région Azure la plus proche possible de vos utilisateurs internes ou de vos clients. Sachez toutefois que certaines tailles de machine virtuelle ne sont pas disponibles dans toutes les régions. Pour en savoir plus, consultez [Services par région][services-by-region]. Pour obtenir la liste des tailles de machine virtuelle disponibles dans une région spécifique, exécutez la commande suivante dans l’interface de ligne de commande (CLI) Azure :

```
az vm list-sizes --location <location>
```

Pour en savoir plus sur le choix d’une image de machine virtuelle publiée, voir [Comment rechercher des images de machine virtuelle Windows][select-vm-image].

Permet la surveillance et le diagnostic, avec notamment des indicateurs d’intégrité de base, des journaux d’infrastructure de diagnostic et des [diagnostics de démarrage][boot-diagnostics]. Les diagnostics de démarrage peuvent vous aider à identifier le problème de démarrage si votre machine virtuelle refuse de démarrer. Pour en savoir plus, voir [Activation de la surveillance et des diagnostics][enable-monitoring].  

### <a name="disk-and-storage-recommendations"></a>Recommandations pour le disque et le stockage

Pour optimiser les performances d’E/S du disque, nous vous recommandons le [Stockage Premium][premium-storage], qui stocke les données sur des disques SSD. Le coût est basé sur la capacité du disque approvisionné. Le nombre d’E/S par seconde et le débit (c’est-à-dire le taux de transfert des données) dépendent également de la taille du disque. Lorsque vous approvisionnez un disque, vous devez donc tenir compte des trois facteurs : capacité, E/S par seconde et débit. 

Nous vous recommandons également d’utiliser des [disques managés][managed-disks]. Les disques managés ne nécessitent pas de compte de stockage. Il vous suffit de spécifier leur taille et leur type, puis de les déployer en tant que ressource hautement disponible.

Ajoutez un ou plusieurs disques de données. Lorsque vous créez un disque dur virtuel, il n’est pas formaté. Connectez-vous à la machine virtuelle pour formater le disque. Lorsque cela est possible, installez les applications sur un disque de données plutôt que sur le disque du système d’exploitation. Certaines applications héritées requièrent peut-être l’installation de composants sur le lecteur C: ; dans ce cas, vous pouvez [redimensionner le disque du système d’exploitation][resize-os-disk] à l’aide de PowerShell.

Créez un compte de stockage pour contenir les journaux de diagnostic. Un compte de stockage localement redondant (LRS) standard suffit pour les journaux de diagnostic.

> [!NOTE]
> Si vous n’utilisez pas de disques managés, créez des comptes de stockage Azure distincts pour chaque machine virtuelle pour stocker les disques durs virtuels (VHD), afin d’éviter d’atteindre les [limites d’E/S par seconde][vm-disk-limits] des comptes de stockage. Tenez compte des limites d’E/S totales du compte de stockage. Pour en savoir plus, voir les [limites du nombre de disques de machine virtuelle][vm-disk-limits].


### <a name="network-recommendations"></a>Recommandations pour le réseau

Cette adresse IP publique peut être dynamique ou statique. Par défaut, elle est dynamique.

* Utilisez une [adresse IP statique][static-ip] si vous avez besoin d’une adresse IP non modifiable &mdash; par exemple si vous devez créer un enregistrement « A » dans le DNS ou ajouter l’adresse IP dans une liste sécurisée.
* Vous pouvez également créer un nom de domaine complet (FQDN) pour l’adresse IP. Vous pouvez inscrire un [enregistrement CNAME][cname-record] dans le DNS qui pointe vers le nom de domaine complet (FQDN). Pour en savoir plus, voir [Créer un nom de domaine complet dans le Portail Azure][fqdn].

Tous les groupes de sécurité réseau contiennent un ensemble de [règles par défaut][nsg-default-rules], notamment une règle qui bloque tout le trafic Internet entrant. Les règles par défaut ne peuvent pas être supprimées, mais d’autres règles peuvent les remplacer. Pour autoriser le trafic Internet, créez des règles qui autorisent le trafic entrant vers des ports spécifiques, par exemple, le port 80 pour HTTP.

Pour activer le protocole RDP, ajoutez une règle de groupe de sécurité réseau qui autorise le trafic entrant sur le port TCP 3389.

## <a name="scalability-considerations"></a>Considérations relatives à l’extensibilité

Vous pouvez réduire ou augmenter la puissance d’une machine virtuelle en en [modifiant la taille][vm-resize]. Pour une mise à l’échelle horizontale, placez plusieurs machines virtuelles derrière un équilibreur de charge. Pour plus d’informations, consultez [Exécuter des machines virtuelles à charge équilibrée à des fins d’extensibilité et de disponibilité][multi-vm].

## <a name="availability-considerations"></a>Considérations relatives à la disponibilité

Pour bénéficier d’une disponibilité plus élevée, déployez plusieurs machines virtuelles dans un groupe à haute disponibilité. Cette approche offre également un [contrat de niveau de service (SLA)][vm-sla] supérieur.

Votre machine virtuelle peut être affectée par la [maintenance planifiée][planned-maintenance] ou la [maintenance non planifiée][manage-vm-availability]. Vous pouvez utiliser les [journaux de redémarrage de machine virtuelle][reboot-logs] pour déterminer si un redémarrage de la machine virtuelle a été provoqué par une maintenance planifiée.

Pour vous protéger contre la perte accidentelle de données pendant les opérations normales (par exemple, en cas d’erreur d’un utilisateur), vous devez également implémenter des sauvegardes ponctuelles, à l’aide de [captures instantanées d’objets blob][blob-snapshot] ou d’un autre outil.

## <a name="manageability-considerations"></a>Considérations relatives à la facilité de gestion

**Groupes de ressources.** Placez les ressources étroitement associées qui partagent le même cycle de vie dans un même [groupe de ressources][resource-manager-overview]. Les groupes de ressources vous permettent de déployer et de surveiller les ressources en tant que groupe et de suivre les coûts de facturation par groupe de ressources. Vous pouvez également supprimer des ressources dans un ensemble, ce qui est très utile pour les déploiements de test. Affectez des noms de ressource explicites pour simplifier la recherche d’une ressource spécifique et comprendre son rôle. Pour plus d’informations, consultez [Conventions d’affectation de noms recommandées pour les ressources Azure][naming-conventions].

**Arrêt d’une machine virtuelle.** Azure établit une distinction entre les états « arrêté » et « désalloué ». Vous payez lorsque l’état de la machine virtuelle est « arrêté », mais pas lorsque la machine virtuelle est désallouée. Le bouton **Arrêter** du portail Azure désalloue la machine virtuelle. Si vous arrêtez la machine virtuelle par le biais du système d’exploitation pendant que vous êtes connecté, la machine virtuelle est arrêtée mais **non** désallouée. Vous serez donc toujours facturé.

**Suppression d’une machine virtuelle.** La suppression d’une machine virtuelle n’entraîne pas celle des disques durs virtuels. Vous pouvez donc supprimer la machine virtuelle, sans risque de perdre des données. Toutefois, vous serez toujours facturé pour le stockage. Pour supprimer le disque dur virtuel, supprimez le fichier de [Stockage Blob][blob-storage]. Pour éviter toute suppression accidentelle, utilisez un [verrou de ressource][resource-lock] pour verrouiller tout le groupe de ressources ou des ressources individuelles, par exemple une machine virtuelle.

## <a name="security-considerations"></a>Considérations relatives à la sécurité

[Azure Security Center][security-center] vous offre un aperçu global de l’état de toutes vos ressources Azure en termes de sécurité. Il surveille les problèmes potentiels de sécurité et fournit une image complète de la sécurité de votre déploiement. Le Centre de sécurité est configuré pour chaque abonnement Azure. Activez la collecte de données de sécurité comme décrit dans le [Guide de démarrage rapide d’Azure Security Center][security-center-get-started]. Une fois la collecte de données activée, le Centre de sécurité analyse automatiquement les machines virtuelles créées dans le cadre de cet abonnement.

**Gestion des correctifs.** Si cette option est activée, Security Center vérifie si des mises à jour critiques et de sécurité sont manquantes. Utilisez les [paramètres de stratégie de groupe][group-policy] sur la machine virtuelle pour activer les mises à jour automatiques du système.

**Logiciel anti-programme malveillant.** Si cette option est activée, le Centre de sécurité vérifie si un logiciel anti-programme malveillant est installé. Vous pouvez également utiliser le Centre de sécurité pour installer des logiciels anti-programme malveillant dans le portail Azure.

**Opérations.** Utilisez le [contrôle d’accès en fonction du rôle][rbac] (RBAC) pour contrôler l’accès aux ressources Azure que vous déployez. Le contrôle RBAC vous permet d’affecter des rôles d’autorisation aux membres de votre équipe DevOps. Par exemple, le rôle Lecteur permet d’afficher des ressources Azure mais pas de les créer, gérer ou supprimer. Certains rôles sont spécifiques à des types de ressources Azure particuliers. Par exemple, le rôle Contributeur de machine virtuelle peut redémarrer ou désallouer une machine virtuelle, réinitialiser le mot de passe administrateur, créer une machine virtuelle, et ainsi de suite. D’autres [rôles RBAC intégrés][rbac-roles] peuvent être utiles dans cette architecture, notamment [Utilisateur DevTest Lab][rbac-devtest] et [Collaborateur de réseau][rbac-network]. Un utilisateur peut être affecté à plusieurs rôles, et vous pouvez créer des rôles personnalisés pour d’autres autorisations plus affinées.

> [!NOTE]
> Le contrôle RBAC ne limite pas les actions qu’un utilisateur connecté peut effectuer sur une machine virtuelle. Ces autorisations dépendent du type de compte installé sur le système d’exploitation invité.   

Utilisez les [journaux d’audit][audit-logs] pour voir les actions d’approvisionnement et d’autres événements concernant la machine virtuelle.

**Chiffrement des données.** Utilisez [Azure Disk Encryption][disk-encryption] si vous devez chiffrer les disques du système d’exploitation et de données. 

## <a name="deploy-the-solution"></a>Déployer la solution

Un déploiement pour cette architecture est disponible sur [GitHub][github-folder]. Il déploie les éléments suivants :

  * Un réseau virtuel avec un seul sous-réseau nommé **web** qui sert à héberger la machine virtuelle.
  * Un groupe de sécurité réseau avec deux règles de trafic entrant pour autoriser le trafic RDP et HTTP sur la machine virtuelle.
  * Une machine virtuelle exécutant la version la plus récente de Windows Server 2016 Datacenter Edition.
  * Un exemple d’extension de script personnalisé qui formate les deux disques de données, et un script DSC PowerShell qui déploie Internet Information Services (IIS).

### <a name="prerequisites"></a>Prérequis


1. Clonez, dupliquez ou téléchargez le fichier zip pour le référentiel GitHub des [architectures de référence][ref-arch-repo].

2. Vérifiez qu’Azure CLI 2.0 est installé sur votre ordinateur. Pour des instructions d’installation de l’interface de ligne de commande, consultez [Installer Azure CLI 2.0][azure-cli-2].

3. Installez le package npm des [modules Azure][azbb].

4. À partir d’une invite de commandes, d’une invite bash ou de l’invite de commandes PowerShell, entrez la commande suivante pour vous connecter à votre compte Azure.

   ```bash
   az login
   ```

### <a name="deploy-the-solution-using-azbb"></a>Déployer la solution à l’aide d’azbb

Pour déployer cette architecture de référence, effectuez les étapes suivantes :

1. Accédez au dossier `virtual-machines\single-vm\parameters\windows` pour le dépôt que vous avez téléchargé à l’étape des prérequis ci-dessus.

2. Ouvrez le fichier `single-vm-v2.json` et entrez un nom d’utilisateur et un mot de passe entre les guillemets, puis enregistrez le fichier.

   ```bash
   "adminUsername": "",
   "adminPassword": "",
   ```

3. Exécutez `azbb` pour déployer l’exemple de machine virtuelle, comme illustré ci-dessous.

   ```bash
   azbb -s <subscription_id> -g <resource_group_name> -l <location> -p single-vm-v2.json --deploy
   ```

Pour vérifier le déploiement, exécutez la commande Azure CLI suivante pour rechercher l’adresse IP publique de la machine virtuelle :

```bash
az vm show -n ra-single-windows-vm1 -g <resource-group-name> -d -o table
```

Si vous accédez à cette adresse dans un navigateur web, vous devez voir la page d’accueil IIS par défaut.

Pour plus d’informations sur la personnalisation de ce déploiement, visitez notre [référentiel GitHub][git].

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur nos [modules Azure][azbbv2].
- Déployer [plusieurs machines virtuelles][multi-vm] dans Azure.

<!-- links -->
[audit-logs]: https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/
[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azbbv2]: https://github.com/mspnp/template-building-blocks
[azure-cli-2]: /cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[azure-storage]: /azure/storage/storage-introduction
[blob-snapshot]: /azure/storage/storage-blob-snapshots
[blob-storage]: /azure/storage/storage-introduction
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[cname-record]: https://en.wikipedia.org/wiki/CNAME_record
[data-disk]: /azure/virtual-machines/virtual-machines-windows-about-disks-vhds
[disk-encryption]: /azure/security/azure-security-disk-encryption
[enable-monitoring]: /azure/monitoring-and-diagnostics/insights-how-to-use-diagnostics
[fqdn]: /azure/virtual-machines/virtual-machines-windows-portal-create-fqdn
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[group-policy]: https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn595129(v=ws.11)
[log-collector]: https://azure.microsoft.com/blog/simplifying-virtual-machine-troubleshooting-using-azure-log-collector/
[manage-vm-availability]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[managed-disks]: /azure/storage/storage-managed-disks-overview
[multi-vm]: multi-vm.md
[naming-conventions]: ../../best-practices/naming-conventions.md
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-default-rules]: /azure/virtual-network/virtual-networks-nsg#default-rules
[planned-maintenance]: /azure/virtual-machines/virtual-machines-windows-planned-maintenance
[premium-storage]: /azure/virtual-machines/windows/premium-storage
[premium-storage-supported]: /azure/virtual-machines/windows/premium-storage#supported-vms
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rbac-roles]: /azure/active-directory/role-based-access-built-in-roles
[rbac-devtest]: /azure/active-directory/role-based-access-built-in-roles#devtest-labs-user
[rbac-network]: /azure/active-directory/role-based-access-built-in-roles#network-contributor
[reboot-logs]: https://azure.microsoft.com/blog/viewing-vm-reboot-logs/
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[resize-os-disk]: /azure/virtual-machines/virtual-machines-windows-expand-os-disk
[resource-lock]: /azure/resource-group-lock-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[security-center]: /azure/security-center/security-center-intro
[security-center-get-started]: /azure/security-center/security-center-get-started
[select-vm-image]: /azure/virtual-machines/virtual-machines-windows-cli-ps-findimage
[services-by-region]: https://azure.microsoft.com/regions/#services
[static-ip]: /azure/virtual-network/virtual-networks-reserved-public-ip
[virtual-machine-sizes]: /azure/virtual-machines/virtual-machines-windows-sizes
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-resize]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[vm-size-tables]: /azure/virtual-machines/virtual-machines-windows-sizes
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[0]: ./images/single-vm-diagram.png "Architecture de machine virtuelle Windows unique dans Azure"
