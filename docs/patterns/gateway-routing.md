---
title: Modèle de routage de passerelle
titleSuffix: Cloud Design Patterns
description: Acheminez les requêtes vers plusieurs services à l’aide d’un seul point de terminaison.
keywords: modèle de conception
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: 4db98038f582e0315a743a55d46013d2eda187e3
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/04/2019
ms.locfileid: "54010475"
---
# <a name="gateway-routing-pattern"></a>Modèle de routage de passerelle

Acheminez les requêtes vers plusieurs services à l’aide d’un seul point de terminaison. Ce modèle est utile lorsque vous souhaitez exposer plusieurs services sur un seul point de terminaison et quand vous voulez créer un routage vers le service approprié basé sur la requête.

## <a name="context-and-problem"></a>Contexte et problème

Lorsqu’un client a besoin d’utiliser plusieurs services, configurer un point de terminaison distinct pour chaque service et gérer chacun de ces points de terminaison peut s’avérer complexe. Par exemple, une application de e-commerce peut fournir des services comme faire des recherches, laisser des avis, remplir son panier, effectuer des paiements et consulter l’historique des commandes. Chaque service possède une API différente avec laquelle le client doit interagir, et le client doit avoir des informations sur tous les points de terminaison pour pouvoir se connecter aux services. En cas de modification d’une API, le client doit également être mis à jour. Si vous refactorisez un service en deux ou plusieurs services distincts, le code doit être modifié dans le service et le client.

## <a name="solution"></a>Solution

Placez une passerelle devant un ensemble d’applications, de services ou de déploiements. Utilisez le routage de couche Application 7 pour acheminer la requête vers les instances appropriées.

Avec ce modèle, l’application cliente doit uniquement connaître des informations sur un point de terminaison unique et communiquer avec celui-ci. Si un service est consolidé ou décomposé, le client ne doit pas nécessairement être mis à jour. Il peut continuer à envoyer des requêtes à la passerelle et seul le routage change.

Une passerelle vous permet également d’extraire les services principaux à partir des clients, ce qui simplifie les appels des clients lors de l’application des modifications dans les services principaux derrière la passerelle. Les appels des clients peuvent être acheminés vers n’importe quels services nécessaires pour gérer le comportement du client attendu, ce qui vous permet d’ajouter, de fractionner et de réorganiser des services derrière la passerelle sans modifier le client.

![Diagramme du modèle de routage de passerelle](./_images/gateway-routing.png)

Ce modèle peut également simplifier le déploiement en vous permettant de gérer la façon dont les mises à jour sont déployées aux utilisateurs. Lorsqu’une nouvelle version de votre service est déployée, elle peut être déployée en parallèle avec la version existante. Le routage vous permet de contrôler quelle version du service est présentée aux clients, ce qui vous donne la possibilité d’utiliser diverses stratégies de mise en production (déploiement de mises à jour incrémentielles, parallèles ou complètes). Les problèmes détectés après avoir déployé le nouveau service peuvent être corrigés rapidement en modifiant la configuration au niveau de la passerelle, sans affecter les clients.

## <a name="issues-and-considerations"></a>Problèmes et considérations

- Le service de passerelle peut introduire un point de défaillance unique. Vérifiez qu’il est correctement conçu pour répondre à vos besoins de disponibilité. Pensez aux capacités de tolérance de panne et de résilience lors de l’implémentation.
- Le service de passerelle peut introduire un goulot d’étranglement. Vérifiez que les performances de la passerelle sont appropriées pour gérer la charge et qu’elle peut facilement évoluer en fonction de vos envies de croissance futures.
- Effectuez un test de charge sur la passerelle pour vous assurer de ne pas introduire d’échecs en cascade dans les services.
- Le routage de passerelle est au niveau 7. Il peut être basé sur IP, port, en-tête ou URL.

## <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle

Utilisez ce modèle dans les situations suivantes :

- Un client doit utiliser plusieurs services accessibles derrière une passerelle.
- Vous souhaitez simplifier les applications clientes à l’aide d’un seul point de terminaison.
- Vous avez besoin d’acheminer les requêtes de points de terminaison adressables externes vers des points de terminaison virtuels internes, par exemple exposer des ports sur une machine virtuelle vers des adresses IP virtuelles de cluster.

Ce modèle peut ne pas convenir lorsque vous avez une application simple qui utilise uniquement un ou deux services.

## <a name="example"></a>Exemples

Avec Nginx en tant que routeur, voici un exemple de fichier de configuration simple d’un serveur qui achemine des requêtes pour des applications se trouvant sur différents répertoires virtuels vers des machines distinctes au niveau du back end.

```console
server {
    listen 80;
    server_name domain.com;

    location /app1 {
        proxy_pass http://10.0.3.10:80;
    }

    location /app2 {
        proxy_pass http://10.0.3.20:80;
    }

    location /app3 {
        proxy_pass http://10.0.3.30:80;
    }
}
```

## <a name="related-guidance"></a>Aide connexe

- [Backends for Frontends pattern](./backends-for-frontends.md) (Modèle de services principaux destinés aux frontaux)
- [Gateway Aggregation pattern](./gateway-aggregation.md) (Modèle d’agrégation de passerelle)
- [Modèle de déchargement de passerelle](./gateway-offloading.md)
