---
title: 'Framework d’adoption du cloud : Présentation de la conformité réglementaire'
description: Qu’est-ce que la conformité réglementaire ?
author: BrianBlanchard
ms.date: 2/8/2019
ms.openlocfilehash: 5ff7b591d7cab647a99cee223e6271928e185a2f
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901106"
---
# <a name="introduction-to-regulatory-compliance"></a>Présentation de la conformité réglementaire

Il s’agit d’un article introductif sur la conformité réglementaire. Il ne vise donc pas l’implémentation d’une stratégie de conformité. Il cherche uniquement à familiariser le lecteur au sujet. Des informations plus détaillées sur les [offres de conformité Azure](https://aka.ms/allcompliance) sont disponibles dans le [Centre de gestion de la confidentialité Microsoft]. En outre, toute la documentation téléchargeable est disponible aux clients Azure, et soumise à un accord de non-divulgation, sur le [portail d’approbation des services Microsoft](https://servicetrust.microsoft.com/).

La conformité réglementaire fait référence à la discipline et au processus consistant à garantir qu’une entreprise respecte les lois définies par les organes directeurs dans sa zone géographique ou son secteur. Pour la conformité réglementaire informatique, les personnes et/ou les processus supervisent les systèmes de l’entreprise, dans le but de détecter et d’empêcher les violations de stratégies et de procédures établies par la loi et les règlements en vigueur. Cette notion concerne une très grande quantité de processus d’application et de supervision. En fonction du secteur et de la zone géographique, ces processus peuvent être assez longs et complexes.

Pour les organisations multinationales (en particulier dans des secteurs fortement réglementés, tels que les services de santé et financiers), la conformité peut être très difficile. Les normes et réglementations abondent et, bien sûr, sont régulièrement modifiées. Il est donc difficile pour les entreprises de suivre l’évolution des lois internationales sur le traitement des données électroniques.

Comme avec les contrôles de sécurité, les organisations doivent comprendre la répartition des responsabilités en matière de conformité réglementaire dans le cloud. Les fournisseurs de cloud mettent tout en œuvre pour que leurs plateformes et leurs services soient conformes. Toutefois, les organisations doivent également vérifier que leurs applications, et celles fournies par des tiers, sont conformes. De même, les applications des secteurs réglementés qui utilisent des services cloud peuvent nécessiter une certification du fournisseur de cloud.

Voici des descriptions de règles de conformité dans divers secteurs et zones géographiques :

## <a name="hipaa"></a>HIPAA

Une application de santé qui traite des informations PHI (Protected Health Information, ou informations de santé protégées) doit respecter les règles de confidentialité et les règles de sécurité édictées par la loi Health Information Portability and Accountability Act (HIPAA). Au minimum, la loi HIPAA peut exiger qu’un établissement de soins de santé reçoive par écrit une garantie du fournisseur de cloud, indiquant que les données PHI reçues ou créées seront protégées.

## <a name="pci"></a>PCI

La norme de sécurité de l’industrie des cartes de paiement (PCI DSS) concerne les informations propriétaires pour les organisations, lorsque celles-ci traitent les cartes de crédit venant des grands systèmes de carte comme Visa, MasterCard, American Express, Discover et JCB. La norme PCI est mandatée par les sociétés de carte de paiement, et administrée par le Payment Card Industry Security Standards Council. La norme a été créée pour augmenter les contrôles sur les données des titulaires de carte, et ainsi réduire les fraudes. La validation de la conformité est effectuée tous les ans par un Qualified Security Assessor (QSA) externe ou un Internal Security Assessor (ISA) spécifique à la firme. Il crée alors un Report on Compliance (ROC, ou rapport sur la conformité) pour les organisations traitant de larges volumes de transactions. Pour les entreprises, cette validation est effectuée par un Self-Assessment Questionnaire (SAQ).

## <a name="pii"></a>PII

Les informations d’identification personnelle (PII) correspondent à un point de données qui peut être utilisé pour identifier un consommateur, un employé, un partenaire ou toute autre entité légale ou vivante. Beaucoup de nouvelles lois, surtout celles liées à la confidentialité et aux données PII individuelles, exigent que les entreprises se conforment aux normes, le prouvent, et signalent les violations qui se produisent.

## <a name="gdpr"></a>RGPD

Un des développements les plus importants dans ce sens est la promulgation récente par la Commission européenne du Règlement général sur la protection des données (RGPD). Celui-ci a été conçu pour renforcer la protection des données pour les personnes au sein de l’Union européenne. Le RGPD requiert que les données concernant les personnes, par exemple « un nom, une adresse postale, une photo, une adresse e-mail, des coordonnées bancaires, des publications sur des réseaux sociaux, des informations médicales ou l’adresse IP d’un ordinateur » restent sur des serveurs au sein de l’UE et ne soient pas transférées. Il requiert également que les entreprises informent les personnes en cas de violations des données, et engagent un Délégué à la protection des données. D’autres pays ont développé des réglementations semblables, ou sont en train de le faire.

## <a name="compliant-foundation-in-azure"></a>Conformité dans Azure

Pour aider les clients à répondre à leurs propres obligations en matière de conformité dans les industries et sur les marchés réglementés du monde entier, Azure gère la plus grande gamme de solutions de conformité du secteur &mdash; en termes de largeur (nombre total d’offres) et de profondeur (nombre de services B2C dans l’étendue d’évaluation). Les offres de conformité Azure sont regroupées dans quatre segments : applicables globalement, gouvernement des États-Unis, propres à un secteur d’activité, et propres à une région ou un pays.

Les offres de conformité Azure sont basées sur divers types de garanties, notamment des certifications, attestations, validations, autorisations et évaluations officielles créées par des sociétés d’audit tierces indépendantes, ainsi que des modifications contractuelles, des auto-évaluations et des documents de conseils pour les clients créés par Microsoft. Chaque description d’offre de ce document fournit une instruction mise à jour indiquant quels services Azure orientés client se trouvent dans le champ d’application de l’évaluation, et donne des liens vers des ressources téléchargeables pour aider les clients à gérer leurs obligations en matière de conformité.

Des informations plus détaillées sur les offres de conformité Azure sont disponible dans le [Centre de gestion de la confidentialité Microsoft](/trustcenter/compliance/complianceofferings). En outre, toute la documentation téléchargeable est disponible aux clients Azure, et soumise à un accord de non-divulgation, sur le [portail d’approbation des services Microsoft](https://servicetrust.microsoft.com) dans les sections suivantes :

* Rapports d’audit : Comprend les sections sur les rapports FedRAMP, d’évaluation GRC, ISO, PCI DSS et SOC
* Ressources de protection des données : Inclut des guides de conformité, des FAQ, des livres blancs, des tests et des évaluations de sécurité
