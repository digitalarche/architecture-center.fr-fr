---
title: Migration d’une application web héritée vers une architecture basée sur les API sur Azure
description: Utilisez Gestion des API Azure pour moderniser une application web héritée.
author: begim
ms.date: 09/13/2018
ms.custom: fasttrack
ms.openlocfilehash: ea063653b4962e42cbec7f9d98c16e22e987efd1
ms.sourcegitcommit: a0e8d11543751d681953717f6e78173e597ae207
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2018
ms.locfileid: "53004716"
---
# <a name="migrating-a-legacy-web-application-to-an-api-based-architecture-on-azure"></a>Migration d’une application web héritée vers une architecture basée sur les API sur Azure

Une entreprise d’e-commerce dans le secteur du tourisme modernise sa pile logicielle héritée basée sur un navigateur. Même si la pile existante est en grande partie monolithique, certains [services HTTP basés sur SOAP][soap] sont présents en raison d’un projet récent. L’entreprise envisage de créer des sources de revenus supplémentaires pour monétiser certaines données de propriété intellectuelle interne qu’elle développe.

Les objectifs du projet incluent le traitement de la dette technique, l’amélioration de la maintenance continue et l’accélération du développement des fonctionnalités avec moins de bogues de régression. Le projet va utiliser un processus itératif afin d’éviter les risques, avec quelques étapes effectuées en parallèle :

* L’équipe de développement va moderniser le back end de l’application, qui se compose de bases de données relationnelles hébergées sur des machines virtuelles.
* L’équipe de développement interne va écrire de nouvelles fonctionnalités métier, exposées sur les nouvelles API HTTP.
* Une équipe de développement externe va créer une nouvelle interface utilisateur basée sur un navigateur et hébergée sur Azure.

Les nouvelles fonctionnalités de l’application seront proposées au fur et à mesure. Ces fonctionnalités vont progressivement remplacer la fonctionnalité existante d’interface utilisateur du serveur client basée sur un navigateur (hébergée en local) qui joue un rôle essentiel dans l’entreprise d’e-commerce à ce jour.

L’équipe de direction ne souhaite pas effectuer de modernisations inutiles. En outre, elle veut garder le contrôle sur l’étendue et les coûts. Pour ce faire, elle a décidé de conserver ses services HTTP SOAP existants. Elle souhaite aussi ne pas apporter trop de modifications à l’interface utilisateur en place. Le service [Gestion des API Azure ][apim] peut être utilisé pour traiter la plupart des exigences et contraintes du projet.

## <a name="architecture"></a>Architecture

![Diagramme de l’architecture][architecture]

La nouvelle interface utilisateur va être hébergée en tant qu’application PaaS sur Azure et reposer sur les API HTTP nouvelles et existantes. Ces API sont fournies avec un ensemble d’interfaces mieux conçues offrant de meilleures performances, une intégration plus simple et une extensibilité future.

### <a name="components-and-security"></a>Composants et sécurité

1. L’application web locale existante continue d’utiliser directement les services web locaux existants.
2. Les appels en provenance de l’application web existante vers les services HTTP existants restent inchangés. Ces appels sont internes au réseau de l’entreprise.
3. Les appels entrants sont effectués à partir d’Azure vers les services internes existants:
    * L’équipe de sécurité autorise le trafic en provenance de l’instance Gestion des API Azure à traverser le pare-feu de l’entreprise vers les services locaux existants [à l’aide de protocoles de transport sécurisés (HTTPS/SSL)][apim-ssl].
    * L’équipe des opérations autorise les appels entrants vers les services uniquement si ceux-ci proviennent de l’instance Gestion des API Azure. Cette condition est remplie en [autorisant l’adresse IP de l’instance Gestion des API Azure][apim-whitelist-ip] dans le périmètre du réseau de l’entreprise.
    * Un nouveau module est configuré dans le pipeline de demande de services HTTP local (pour agir **uniquement** sur ces connexions provenant de l’extérieur), ce qui valide [un certificat que Gestion des API Azure fournira][apim-mutualcert-auth].
1. La nouvelle API :
    * Est exposée uniquement via l’instance Gestion des API Azure, qui fournit la façade de l’API. La nouvelle API n’est pas accessible directement.
    * Est développée et publiée en tant [qu’application d’API web PaaS Azure][azure-api-apps].
    * Est autorisée (via [Paramètres de l’application web][azure-appservice-ip-restrict]) à accepter uniquement [l’adresse IP virtuelle Gestion des API Azure][apim-faq-vip].
    * Est hébergée dans Azure Web Apps avec les protocoles de transport sécurisés/SSL activés.
    * A une autorisation activée, [fournie par Azure App Service][azure-appservice-auth] à l’aide d’Azure Active Directory et d’OAuth2.
2. La nouvelle application web basée sur un navigateur repose sur l’instance Gestion des API Azure pour l’API HTTP existante **et** la nouvelle API.

L’instance Gestion des API Azure est configurée pour mapper les services HTTP hérités à un nouveau contrat d’API. En procédant ainsi, la nouvelle interface utilisateur web n’a pas connaissance de l’intégration à un ensemble de services/d’API hérités et d’API nouvelles. À l’avenir, l’équipe projet va progressivement étendre les fonctionnalités aux nouvelles API et supprimer les services d’origine. Ces modifications sont gérées au sein de la configuration de Gestion des API Azure, ce qui permet de laisser l’interface utilisateur frontale telle qu’elle est et ainsi d’éviter tout travail de redéveloppement.

### <a name="alternatives"></a>Autres solutions

* Si l’organisation avait pour objectif de migrer l’ensemble de son infrastructure sur Azure, y compris les machines virtuelles hébergeant les applications héritées, le service Gestion des API Azure reste une option intéressante. En effet, il peut servir de façade pour tous les points de terminaison HTTP adressables.
* Si le client a décidé de conserver les points de terminaison existants privés et de ne pas les exposer publiquement, son instance Gestion des API peut être associée à un [réseau virtuel Azure][azure-vnet] :
  * Dans un [scénario lift and shift Azure][azure-vm-lift-shift] associé à son réseau virtuel Azure déployé, le client peut répondre directement au service back end via des adresses IP privées.
  * Dans le scénario local, l’instance Gestion des API peut communiquer avec le service interne de façon privée via une [passerelle VPN Azure et une connexion VPN IPSec de site à site][azure-vpn] ou via [ExpressRoute][azure-er], ce qui crée un [scénario local et Azure hybride][azure-hybrid].
* L’instance Gestion des API peut rester privée si vous la déployez en mode interne. Le déploiement peut ensuite être utilisé avec une instance [Azure Application Gateway][azure-appgw] pour autoriser l’accès public à certaines API tout en conservant d’autres API en interne. Pour plus d’informations, consultez [Intégrer le service Gestion des API dans un réseau virtuel interne avec Application Gateway][apim-vnet-internal].

> [!NOTE]
> Pour des informations générales sur la connexion de Gestion des API à un réseau virtuel, [rendez-vous ici][apim-vnet].

### <a name="availability-and-scalability"></a>Disponibilité et extensibilité

* Le service Gestion des API Azure peut faire l’objet d’une [augmentation de la taille des instances][apim-scaleout] en choisissant un niveau tarifaire, puis en ajoutant des unités.
* La mise à l’échelle se produit également [automatiquement grâce à la fonctionnalité de mise à l’échelle automatique][apim-autoscale].
* Le [déploiement dans plusieurs régions][apim-multi-regions] permet de bénéficier d’options de basculement et peut être effectué dans le [niveau Premium][apim-pricing].
* Envisagez [d’intégrer Azure Application Insights][azure-apim-ai] qui expose également des métriques via [Azure Monitor][azure-mon] pour la supervision.

## <a name="deployment"></a>Déploiement

Pour commencer, [créez une instance Gestion des API Azure dans le portail.][apim-create]

Vous pouvez également choisir un [modèle de démarrage rapide][azure-quickstart-templates-apim] Azure Resource Manager qui correspond à votre cas d’usage spécifique.

## <a name="pricing"></a>Tarifs

Le service Gestion des API est disponible en quatre niveaux : Développeur, De base, Standard et Premium. Vous trouverez des conseils détaillés sur la différence entre ces niveaux dans [Tarification Gestion des API.][apim-pricing]

Les clients peuvent mettre à l’échelle Gestion des API en ajoutant et en supprimant des unités. La capacité de chaque unité dépend de son niveau.

> [!NOTE]
> Le niveau Développeur peut être utilisé pour l’évaluation des fonctionnalités du service Gestion des API. Le niveau Développeur ne doit pas servir pour la production.

Pour afficher les coûts prévus et effectuer une personnalisation en fonction des besoins de votre déploiement, vous pouvez modifier le nombre d’unités d’échelle et d’instances App Service dans la [calculatrice de prix Azure][pricing-calculator].

## <a name="related-resources"></a>Ressources associées

Passez en revue l’intégralité des [articles de référence et de la documentation][apim] sur le service Gestion des API Azure.


<!-- links -->
[architecture]: ./media/architecture-apim-api-scenario.png
[apim-create]: /azure/api-management/get-started-create-service-instance
[apim-git]: /azure/api-management/api-management-configuration-repository-git
[apim-multi-regions]: /azure/api-management/api-management-howto-deploy-multi-region
[apim-autoscale]: /azure/api-management/api-management-howto-autoscale
[apim-scaleout]: /azure/api-management/upgrade-and-scale
[azure-apim-ai]: /azure/api-management/api-management-howto-app-insights
[azure-ai]: /azure/application-insights/
[azure-mon]: /azure/monitoring-and-diagnostics/monitoring-overview
[azure-appgw]: /azure/application-gateway/application-gateway-introduction
[apim-vnet-internal]: /azure/api-management/api-management-howto-integrate-internal-vnet-appgateway
[apim-vnet]: /azure/api-management/api-management-using-with-vnet
[azure-hybrid]: /azure/architecture/reference-architectures/hybrid-networking/
[azure-er]: /azure/expressroute/expressroute-introduction
[azure-vpn]: /azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal
[azure-vnet]: /azure/virtual-network/virtual-networks-overview
[azure-appservice-auth]: /azure/app-service/app-service-authentication-overview#identity-providers
[apim-faq-vip]: /azure/api-management/api-management-faq#is-the-api-management-gateway-ip-address-constant-can-i-use-it-in-firewall-rules
[azure-appservice-ip-restrict]: /azure/app-service/app-service-ip-restrictions
[azure-api-apps]: /azure/app-service/
[apim-ssl]: /azure/api-management/api-management-howto-manage-protocols-ciphers
[apim-mutualcert-auth]: /azure/api-management/api-management-howto-mutual-certificates
[apim-whitelist-ip]: /azure/api-management/api-management-faq#is-the-api-management-gateway-ip-address-constant-can-i-use-it-in-firewall-rules
[anti-corruption-layer-pattern]: /azure/architecture/patterns/anti-corruption-layer
[apim]: /azure/api-management/api-management-key-concepts
[apim-api-design-guidance]: /azure/architecture/best-practices/api-design
[visualstudio-youtube-solid-design]: https://youtu.be/agkWYPUcLpg
[azure-vm-lift-shift]: https://azure.microsoft.com/resources/azure-virtual-datacenter-lift-and-shift-guide/
[standard-pricing-calc]: https://azure.com/e/
[premium-pricing-calc]: https://azure.com/e/
[apim-pricing]: https://azure.microsoft.com/pricing/details/api-management/
[azure-quickstart-templates-apim]: https://azure.microsoft.com/resources/templates/?term=API+Management&pageNumber=1
[soap]: https://en.wikipedia.org/wiki/SOAP
[pricing-calculator]: https://azure.com/e/0e916a861fac464db61342d378cc0bd6
