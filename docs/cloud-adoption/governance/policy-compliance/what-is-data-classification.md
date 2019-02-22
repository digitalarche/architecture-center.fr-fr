---
title: 'Framework d’adoption du cloud : Qu’est-ce que la classification des données ?'
description: Qu’est-ce que la classification des données ?
author: BrianBlanchard
ms.date: 2/11/2019
ms.openlocfilehash: 07268e7242d92ac2581bf28b378a3c43d166620c
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900901"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-is-data-classification"></a>Qu’est-ce que la classification des données ?

Cet article offre une introduction sur le thème général de la classification des données. La classification des données est un point de départ couramment utilisé pour établir une gouvernance.

## <a name="business-risks-and-governance"></a>Risques commerciaux et gouvernance

Dans la plupart des organisations, on peut généralement déceler trois risques commerciaux principaux qui motivent les investissements de gouvernance :

* Responsabilité encourue en cas de violations des données
* Interruption des activités suite à des pannes
* Dépenses non planifiées ou inattendues

Ces trois risques commerciaux connaissent de nombreuses variantes. Toutefois, ils ont tendance à être assez courants.

## <a name="understand-then-mitigate"></a>Comprendre les risques pour mieux les atténuer

Avant d’atténuer un risque, il convient de le comprendre. En cas de responsabilité face à une violation des données, cette compréhension passe par la classification des données. Le processus de classification des données consiste à associer une caractéristique de métadonnées à chaque ressource du patrimoine numérique afin d’identifier le type de données associé à cette ressource.

Selon Microsoft, toutes les ressources qui sont des candidates potentielles à la migration ou au déploiement vers le cloud doivent être associées à des métadonnées qui permettent d’enregistrer la classification des données, l’importance dans l’entreprise et le responsable de facturation. Ces trois points de classification peuvent se révéler très utiles à la compréhension et l’atténuation des risques.

## <a name="microsofts-data-classification"></a>Classification des données Microsoft

La liste ci-dessous répertorie les classifications utilisées par Microsoft. En fonction de votre secteur d’activité ou de vos exigences de sécurité existantes, il se peut que des normes de classifications des données existent déjà au sein de votre organisation. Si ce n’est pas le cas, nous vous invitons à utiliser cet exemple de classification. Il vous permettra de mieux comprendre votre patrimoine numérique et votre profil de risque.  

* **Non professionnelles :** Données concernant votre vie personnelle qui n’appartiennent pas à Microsoft
* **Public :** Données d’entreprise qui sont mises à disposition librement et destinées à être utilisées publiquement
* **Généralités :** Données d’entreprise non destinées au public
* **Confidentielles :** Données d’entreprise susceptibles de nuire à Microsoft si elles sont partagées
* **Hautement confidentielles :** Données d’entreprise susceptibles de porter un préjudice important à Microsoft si elles sont partagées

## <a name="tagging-data-classification-in-azure"></a>Balisage de classification des données dans Azure

Chaque fournisseur de cloud doit proposer un mécanisme permettant d’enregistrer des métadonnées associées à n’importe quel type de ressource. Les métadonnées sont essentielles à la bonne gestion des ressources dans le cloud. Azure offre une approche avec balises de ressources pour stocker les métadonnées. Pour plus d’informations sur le balisage des ressources dans Azure, consultez l’article [Utilisation de balises pour organiser vos ressources Azure](/azure/azure-resource-manager/resource-group-using-tags).

## <a name="next-steps"></a>Étapes suivantes

Appliquez les classifications des données dans le cadre d’un parcours de gouvernance exploitable.

> [!div class="nextstepaction"]
> [Commencer un parcours de gouvernance exploitable](../journeys/overview.md)
