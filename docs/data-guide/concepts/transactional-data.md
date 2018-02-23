---
title: "Données transactionnelles"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 6277badde1845c42858e69f6c8ecc73331e7a945
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/14/2018
---
# <a name="transactional-data"></a>Données transactionnelles

Les données transactionnelles sont des informations qui assurent le suivi des interactions relatives aux activités d’une organisation. Ces interactions sont généralement des transactions commerciales, tels que les paiements reçus des clients, les paiements effectués aux fournisseurs, le déplacement de produits dans le cadre de l’inventaire, les commandes prises ou les services fournis. Les événements transactionnels, qui représentent les transactions proprement dites, contiennent une dimension de temps, des valeurs numériques et des références à d’autres données. 

Les transactions doivent généralement être *atomiques* et *cohérentes*. Atomicité signifie qu’une transaction complète réussit ou échoue toujours en tant que seule unité de travail et n’est jamais laissée dans un état à moitié terminé. Si une transaction ne peut pas être effectuée, le système de base de données doit restaurer toutes les étapes qui ont déjà été effectuées dans le cadre de cette transaction. Dans un système SGBDR traditionnel, cette restauration a lieu automatiquement si une transaction ne peut pas être terminée. Cohérence signifie que les transactions laissent toujours les données dans un état valide. (Il s’agit de descriptions très informelles de l’atomicité et de la cohérence. Il existe des définitions plus formelles de ces propriétés, telles qu’ [ACID](https://en.wikipedia.org/wiki/ACID).)

Les bases de données transactionnelles peuvent prendre en charge une cohérence forte pour les transactions grâce à différentes stratégies de verrouillage, tels que le verrouillage pessimiste, pour garantir que toutes les données sont fortement cohérentes dans le contexte de l’entreprise, pour tous les utilisateurs et les processus. 

L’architecture de déploiement la plus courante utilisant des données transactionnelles est le niveau de magasin de données dans une architecture à 3 couches. Une architecture à 3 couches se compose généralement d’une couche de présentation, d’une couche de logique métier et d’une couche de magasin de données. L’architecture [multiniveau](/azure/architecture/guide/architecture-styles/n-tier) est une architecture de déploiement associée pouvant avoir plusieurs couches intermédiaires qui gèrent la logique métier.

![Exemple d’application à 3 couches](./images/three-tier-application.png)

## <a name="typical-traits-of-transactional-data"></a>Caractéristiques des données transactionnelles classiques

Les données transactionnelles ont tendance à présenter les caractéristiques suivantes :

| Prérequis | DESCRIPTION |
| --- | --- |
| Normalisation | Très normalisées |
| Schéma | Schéma lors de l’écriture, fortement appliqué|
| Cohérence | Cohérence forte, garanties ACID |
| Intégrité | Intégrité élevée |
| Utilisent des transactions | OUI |
| Stratégie de verrouillage | Pessimiste ou optimiste|
| Peut être mise à jour | OUI |
| Modifiable | OUI |
| Charge de travail | Haut volume d’écriture, volume de lecture modéré |
| Indexation | Index primaires et secondaires |
| Taille de donnée | Petite à moyenne taille |
| Modèle | Relationnel |
| Forme des données | Tabulaire |
| Flexibilité de requête | Très flexible |
| Scale | Petite (Mo) à grande (quelques To) | 

## <a name="see-also"></a>Voir aussi

[Traitement transactionnel en ligne](../scenarios/online-transaction-processing.md)