---
title: Principes de conception pour les applications Azure
titleSuffix: Azure Application Architecture Guide
description: Principes de conception pour les applications Azure.
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
---

# <a name="ten-design-principles-for-azure-applications"></a>Dix principes de conception pour les applications Azure

Appliquez ces principes de conception pour rendre votre application plus évolutive, résiliente et gérable.

**[Penser la conception des applications pour la réparation spontanée](self-healing.md)**. Dans un système distribué, des défaillances peuvent se produire. Concevez votre application pour qu’elle se répare spontanément en cas de défaillance.

**[Rendre tous les éléments redondant](redundancy.md)**. Intégrez la redondance à votre application, pour éviter les points de défaillance uniques.

**[Réduire la coordination](minimize-coordination.md)**. Réduisez la coordination entre les services d’application pour atteindre une certaine évolutivité.

**[Penser la conception des applications pour augmenter la taille des instances](scale-out.md)**. Concevez votre application afin qu’elle puisse évoluer horizontalement, par l’ajout ou la suppression de nouvelles instances en fonction de la demande.

**[Partitionner autour des limites](partition.md)**. Utilisez le partitionnement pour contourner les limites liées à la base de données, au réseau et au calcul.

**[Penser la conception des applications pour les opérations](design-for-operations.md)**. Concevez votre application pour que l’équipe des opérations dispose des outils dont elle a besoin.

**[Utiliser les services managés Azure](managed-services.md)**. Si possible, utilisez platform as a service (PaaS) au lieu de infrastructure as a service (IaaS).

**[Utiliser la meilleure banque de données pour le travail](use-the-best-data-store.md)**. Choisissez la technologie de stockage la mieux adaptée à vos données et son mode d’utilisation.

**[Penser la conception des applications en vue de leur évolution](design-for-evolution.md)**. Mêmes les applications les plus réussies changent au fil du temps. Une conception évolutive est essentielle une innovation continue.

**[Créer pour les besoins de l’entreprise](build-for-business.md)**. Chaque décision de conception doit être justifiée par un besoin de l’entreprise.
