---
title: Confiance décentralisée entre les banques sur Azure
description: Établissez un environnement de confiance pour communiquer et partager des informations sans avoir recours à une base de données centralisée.
author: vitoc
ms.date: 09/09/2018
ms.openlocfilehash: fe27f885635ce5ae4ce368992affa1a85d7af416
ms.sourcegitcommit: 62945777e519d650159f0f963a2489b6bb6ce094
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2018
ms.locfileid: "48876752"
---
# <a name="decentralized-trust-between-banks-on-azure"></a>Confiance décentralisée entre les banques sur Azure

Cet exemple de scénario est utile pour les banques ou autres établissements souhaitant mettre en place un environnement fiable pour le partage des informations sans avoir recours à une base de données centralisée. Pour les besoins de cet exemple, nous allons décrire le scénario dans le contexte de la gestion des informations de cote de crédit entre les banques. Néanmoins, l’architecture peut être appliquée à tout autre scénario dans lequel un consortium d’organisations souhaite partager des informations validées entre elles, sans avoir recours à un système central exécuté par une seule des parties.

En règle générale, les banques au sein d’un système financier s’appuient sur des sources centralisées telles que des bureaux de crédit pour obtenir des informations sur la cote de crédit d’un particulier, ainsi que son historique. Une approche centralisée concentre des risques opérationnels et requiert de faire appel à un tiers, ce qui est parfois inutile.

Grâce aux technologies de registre distribué, un consortium de banques peut mettre en place un système décentralisé plus efficace et moins susceptible d’être attaqué. En outre, ce système peut faire office de plateforme sur laquelle des structures innovantes peuvent être implémentées pour résoudre les problèmes classiques de confidentialité, de vitesse et de coûts.

Cet exemple vous montre comment les services Azure tels que les groupes de machines virtuelles identiques, les réseaux virtuels, Key Vault, Stockage, Load Balancer et Monitor peuvent être rapidement approvisionnés pour le déploiement d’une blockchain Ethereum PoA privée efficace où les banques membres peuvent créer leurs propres nœuds.

## <a name="relevant-use-cases"></a>Cas d’usage appropriés

Ces autres cas d’usage ont des modèles de conception similaires :

* Déplacement des budgets alloués entre les différentes divisions d’une multinationale
* Paiements transfrontaliers
* Scénarios de financement commercial
* Systèmes de fidélisation impliquant différentes sociétés
* Écosystèmes de chaîne d’approvisionnement et bien plus encore

## <a name="architecture"></a>Architecture

![Schéma d’une architecture fiable bancaire décentralisée](./media/architecture-decentralized-trust.png)

Ce scénario couvre les composants principaux nécessaires pour créer un réseau de blockchain d’entreprise privé, surveillé, sécurisé et évolutif au sein d’un consortium de deux membres minimum. Les informations concernant la façon dont ces composants sont approvisionnés (c.-à-d. au sein de différents abonnements et groupes de ressources), ainsi que les exigences de connectivité (c.-à-d. VPN ou ExpressRoute) doivent être étudiées en fonction de la stratégie de votre organisation. Voici comment les données circulent :

1. La banque A crée/met à jour le dossier de crédit d’un particulier en envoyant une transaction au réseau de blockchain via JSON-RPC.
2. Les données circulent depuis le serveur d’application privé de la banque A vers l’équilibreur de charge Azure, puis vers une machine virtuelle de nœud de validation sur le groupe de machines virtuelles identiques.
3. Le réseau Ethereum PoA crée un bloc à un moment prédéfini (2 secondes pour ce scénario).
4. La transaction est englobée dans le bloc créé et validée sur l’ensemble du réseau de blockchain.
5. La banque B peut lire le dossier de crédit créé par la banque A en communiquant avec son propre nœud comme via JSON-RPC.

### <a name="components"></a>Composants

* Les machines virtuelles au sein de Virtual Machine Scale Sets fournissent la fonctionnalité de calcul à la demande pour héberger les processus de validateur de la blockchain
* Key Vault est utilisé en tant que fonctionnalité de stockage sécurisé pour les clés privées de chaque validateur
* Load Balancer répartit les requêtes de RPC, de peering et de Governance DApp
* Stockage héberge les informations réseau persistantes et coordonne les baux
* Operations Management Suite (un regroupement de plusieurs services Azure) fournit des insights sur les nœuds disponibles, les transactions par minute et les membres du consortium

### <a name="alternatives"></a>Autres solutions

L’approche Ethereum PoA est choisie pour cet exemple, car il s’agit d’un bon point de départ pour un consortium d’organisations souhaitant créer un environnement dans lequel des informations peuvent être échangées et partagées de façon fiable, décentralisée et simple. Les modèles de solution Azure disponibles constituent également un moyen rapide et pratique pour aider le leader d’un consortium à mettre en place une blockchain Ethereum PoA. En outre, ils permettent aux organisations membres du consortium de créer rapidement leurs ressources Azure au sein de leur groupe de ressources et abonnement pour rejoindre un réseau existant.

Pour d’autres scénarios étendus ou différents, des problèmes relatifs à la confidentialité des transactions peuvent se poser. Par exemple, dans un scénario de transfert de titres, les membres d’un consortium ne souhaitent peut-être pas que leurs transactions soient visibles par les autres membres. Des alternatives à l’approche Ethereum PoA répondant à ces problèmes existent :

* Corda
* Quorum
* Hyperledger

## <a name="considerations"></a>Considérations

### <a name="availability"></a>Disponibilité

[Azure Monitor][monitor] est utilisé pour surveiller en permanence le réseau de blockchain et ainsi garantir la disponibilité. Un lien vers un tableau de bord de supervision personnalisé basé sur Azure Monitor vous est envoyé après le déploiement du modèle de solution de blockchain utilisé dans ce scénario. Le tableau de bord affiche les nœuds qui signalent des pulsations au cours des 30 dernières minutes, ainsi que d’autres statistiques utiles. 

Pour consulter d’autres rubriques relatives à la disponibilité, consultez la [liste de contrôle de la disponibilité][availability] dans le Centre des architectures Azure.

### <a name="scalability"></a>Extensibilité

Un problème courant des blockchains est le nombre de transactions qu’une blockchain peut inclure dans une durée prédéfinie. Ce scénario utilise PoA (Proof-of-Authority) quand une telle extensibilité peut être mieux gérée qu’avec PoW (Proof-of-Work). Dans les réseaux basés sur PoA, les participants au consensus sont connus et gérés, ce qui est plus approprié pour une blockchain privée destinée à un consortium d’organisations qui se connaissent entre elles. Les paramètres tels que la durée de bloc moyenne, les transactions par minute et la consommation des ressources de calcul peuvent être facilement surveillées via le tableau de bord personnalisé. Les ressources peuvent ensuite être ajustées en fonction des exigences de mise à l’échelle.

Pour obtenir des conseils d’ordre général sur la conception de scénarios évolutifs, consultez la [liste de contrôle de l’extensibilité][scalability] dans le Centre des architectures Azure.

### <a name="security"></a>Sécurité

[Azure Key Vault][vault] est utilisé pour stocker et gérer facilement les clés privées des validateurs. Le déploiement par défaut dans cet exemple crée un réseau de blockchain qui est accessible via Internet. Pour les scénarios de production où un réseau privé est souhaité, les membres peuvent être connectés entre eux par le biais de connexions de passerelle VPN entre des réseaux virtuels. La procédure à suivre pour configurer un VPN est indiquée dans la section sur les ressources associées ci-dessous.

Pour obtenir des conseils d’ordre général sur la conception de solutions sécurisées, consultez la [documentation sur la sécurité Azure][security].

### <a name="resiliency"></a>Résilience

La blockchain Ethereum PoA peut offrir un certain degré de résilience puisque les nœuds de validateur peuvent être déployés dans différentes régions. Azure permet d’effectuer des déploiements dans plus de 54 régions du monde entier. Une blockchain comme celle de notre exemple offre des possibilités uniques et nouvelles de coopération pour accroître la résilience. La résilience du réseau n’est pas fournie uniquement pour une seule partie centralisée, mais pour tous les membres du consortium. Une blockchain basée sur PoA permet d’obtenir une résilience réseau plus planifiée et délibérée.

Pour obtenir des conseils d’ordre général sur la conception de solutions résilientes, consultez l’article [Conception d’applications résilientes pour Azure][resiliency].

## <a name="pricing"></a>Tarifs

Pour explorer le coût d’exécution de ce scénario, tous les services sont préconfigurés dans le calculateur de coûts. Pour connaître les prix en fonction de votre cas d’usage particulier, modifiez les variables appropriées en fonction des exigences de disponibilité et de performances souhaitées.

Nous proposons trois exemples de profils de coût basés sur le nombre d’instances de groupe identique de machines virtuelles exécutant vos applications (les instances peuvent se trouver dans différentes régions).

* [Petit][small-pricing] : 2 machines virtuelles par mois avec la supervision désactivée
* [Moyen][medium-pricing] : 7 machines virtuelles par mois avec la supervision activée
* [Grand][large-pricing] : 15 machines virtuelles par mois avec la supervision activée

Les tarifs ci-dessus sont donnés pour un membre du consortium qui souhaite mettre en place ou rejoindre un réseau de blockchain. En général, dans un consortium regroupant plusieurs entreprises ou organisations, chaque membre bénéficie de son propre abonnement Azure.

## <a name="next-steps"></a>Étapes suivantes

Pour voir un exemple de ce scénario, déployez [l’application de démonstration de blockchain Ethereum PoA][deploy] sur Azure, puis parcourez le [fichier README du code source du scénario][source].

## <a name="related-resources"></a>Ressources associées

Pour plus d’informations sur l’utilisation du modèle de solution Ethereum PoA pour Azure, consultez ce [guide d’utilisation][guide].

<!-- links -->
[small-pricing]: https://azure.com/e/4e429d721eb54adc9a1558fae3e67990
[medium-pricing]: https://azure.com/e/bb42cd77437744be8ed7064403bfe2ef
[large-pricing]: https://azure.com/e/e205b443de3e4adfadf4e09ffee30c56
[guide]: /azure/blockchain-workbench/ethereum-poa-deployment
[deploy]: https://portal.azure.com/?pub_source=email&pub_status=success#create/microsoft-azure-blockchain.azure-blockchain-ethereumethereum-poa-consortium
[source]: https://github.com/vitoc/creditscoreblockchain
[monitor]: /azure/monitoring-and-diagnostics/monitoring-overview-azure-monitor
[availability]: /azure/architecture/checklist/availability
[scalability]: /azure/architecture/checklist/scalability
[resiliency]: ../../resiliency/index.md
[security]: /azure/security/
[vault]: https://azure.microsoft.com/services/key-vault/
