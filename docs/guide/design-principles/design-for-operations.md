---
title: Penser la conception des applications pour les opérations
description: Concevez vos applications en offrant à l’équipe des opérations les outils dont elle a besoin
author: MikeWasson
ms.date: 08/30/2018
ms.openlocfilehash: a73479a7661c042d05db61907d1f993fc04ac11d
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/31/2018
ms.locfileid: "43326236"
---
# <a name="design-for-operations"></a>Penser la conception des applications pour les opérations

## <a name="design-an-application-so-that-the-operations-team-has-the-tools-they-need"></a>Concevez vos applications en offrant à l’équipe des opérations les outils dont elle a besoin

Le cloud a considérablement modifié le rôle de l’équipe des opérations. Cette dernière n’est plus chargée de gérer le matériel et l’infrastructure qui héberge l’application.  Toutefois, les opérations continuent de jouer un rôle crucial dans l’exécution d’une application cloud réussie. Voici quelques-unes des fonctions clés assurées par l’équipe des opérations :

- Déploiement
- Surveillance
- Escalade
- Réponse aux incidents
- Audit de sécurité

La mise en œuvre d’une journalisation et d’un traçage robustes est particulièrement importante dans les applications cloud. Impliquez l’équipe des opérations dans la conception et la planification pour vous assurer que l’application lui fournit les données et les analyses dont elle a besoin.  <!-- to do: Link to DevOps checklist -->

## <a name="recommendations"></a>Recommandations

**Rendez tous les éléments observables**. Une fois qu’une solution est déployée et en cours d’exécution, les journaux et les traces constituent votre principale source d’informations concernant le système. Le *traçage* enregistre un chemin d’accès au sein du système et se révèle utile pour l’identification des goulots d’étranglement, des problèmes de performances et des points de défaillance. La *journalisation* capture les différents événements, tels que les changements d’état de l’application, les erreurs et les exceptions. Ouvrez une session en production, faute de quoi vous ne serez pas en mesure d’acquérir de précieuses informations au moment précis où vous en aurez le plus besoin.

**Instrumentez l’application à des fins de surveillance**. La surveillance vous permet de déterminer l’efficacité (ou l’inefficacité) d’une application en termes de disponibilité, de performances et d’intégrité du système. Par exemple, la surveillance vous indique si vous respectez votre Contrat de niveau de service (SLA). La surveillance s’effectue pendant le fonctionnement normal du système. Elle doit se dérouler le plus en temps réel possible afin que le personnel chargé des opérations puisse réagir rapidement aux problèmes. Dans l’idéal, la surveillance peut vous signaler les problèmes avant qu’ils n’entraînent une défaillance critique. Pour plus d’informations, consultez l’article [Surveillance et diagnostics][monitoring].

**Instrumentez l’application à des fins d’analyse de la cause racine**. L’analyse de la cause racine désigne le processus de recherche de la cause sous-jacente des défaillances. Cette analyse survient après l’apparition d’une défaillance. 

**Utilisez un traçage distribué**. Utilisez un système de traçage distribué conçu pour la concurrence, le comportement asynchrone et la mise à l’échelle cloud. Les traces doivent inclure un ID de corrélation qui franchit les limites de services. Une même opération peut appeler plusieurs services d’application. Si une opération échoue, l’ID de corrélation permet d’identifier la cause de l’échec. 

**Normalisez les journaux et les métriques**. L’équipe des opérations devra agréger les journaux relatifs aux différents services de votre solution. Si chaque service utilise son propre format de journalisation, il devient impossible ou difficile d’obtenir des informations utiles auprès des multiples services. Définissez un schéma commun incluant des champs tels que l’ID de corrélation, le nom de l’événement, l’adresse IP de l’expéditeur, etc. Les différents services peuvent dériver des schémas personnalisés qui héritent du schéma de base et contiennent des champs supplémentaires.

**Automatisez les tâches de gestion**, y compris l’approvisionnement, le déploiement et la surveillance. L’automatisation d’une tâche la rend reproductible et moins sujette aux erreurs humaines. 

**Traitez la configuration sous forme de code**. Intégrez les fichiers de configuration au sein d’un système de gestion de version pour être en mesure d’effectuer le suivi et le contrôle de version de vos modifications, et de procéder à une restauration si nécessaire. 


<!-- links -->

[monitoring]: ../../best-practices/monitoring.md


