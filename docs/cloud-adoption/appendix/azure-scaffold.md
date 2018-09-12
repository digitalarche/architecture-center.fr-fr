---
title: Meilleures pratiques pour les entreprises effectuant une migration vers Azure
description: Décrit la structure que les entreprises peuvent utiliser pour garantir un environnement sécurisé et facile à gérer.
author: rdendtler
ms.date: 03/31/2017
ms.author: rodend;karlku;tomfitz
ms.openlocfilehash: 91adce796ae7785d3831e9628fce0193076eec9b
ms.sourcegitcommit: f4069cf68456b5c74acb1b890dc4e45e11f12b59
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/04/2018
ms.locfileid: "43675781"
---
# <a name="azure-enterprise-scaffold---prescriptive-subscription-governance"></a>Structure d’entreprise Azure : gouvernance normative de l’abonnement
Les entreprises adoptent de plus en plus le cloud public en raison de son agilité et de la flexibilité. Elles utilisent les atouts du cloud pour générer des revenus ou optimiser les ressources de l’entreprise. Microsoft Azure offre une multitude de services que les entreprises peuvent combiner sous forme de blocs de construction pour traiter une large gamme de charges de travail et d’applications. 

Mais il est souvent difficile de savoir par où commencer. Une fois l’utilisation d’Azure décidée, certaines questions se posent fréquemment :

* « Comment respecter nos obligations légales en matière de souveraineté des données dans certains pays ? »
* « Comment vérifier que quelqu'un ne modifie pas par inadvertance un système critique ? »
* « Comment savoir ce que chaque ressource prend en charge afin de les comptabiliser et de les facturer avec exactitude ? »

La perspective d’un abonnement vide sans garde-fou est décourageante. Elle peut même vous décourager de migrer vers Azure.

Cet article fournit un point de départ aux techniciens qui leur permettra de répondre aux besoins en matière de gouvernance et de les mettre en correspondance avec le besoin de souplesse. Il introduit le concept d’une structure d’entreprise qui guide les organisations dans l’implémentation et la gestion de leurs abonnements Azure. 

## <a name="need-for-governance"></a>Besoin de gouvernance
Lors de la migration vers Azure, vous devez prendre en compte la gouvernance assez tôt pour garantir une utilisation efficace du cloud au sein de l’entreprise. Malheureusement, le temps et la bureaucratie nécessaires à la création d’un système de gouvernance complet impliquent que certaines divisions s’adressent directement à des fournisseurs sans impliquer le service informatique de l’entreprise. Cette approche peut exposer l’entreprise à des vulnérabilités si les ressources ne sont pas gérées correctement. Les caractéristiques du cloud public (agilité, la flexibilité et tarification basée sur la consommation) sont importantes pour les groupes professionnels qui doivent répondre rapidement aux demandes des clients (internes et externes). Mais le service informatique de l’entreprise doit s’assurer que les données et les systèmes sont efficacement protégés.

Dans la réalité, la structure est utilisée pour créer la base de l’architecture. Un échafaudage trace les grandes lignes et fournit des points d’ancrage pour le montage de systèmes plus permanents. Une structure d’entreprise revient au même : il s’agit d’un ensemble de contrôles flexibles et de fonctionnalités Azure qui fournissent la structure de l’environnement et des points d’ancrage pour les services basés sur le cloud public. Elle fournit aux constructeurs (service informatique et groupes professionnels) une base pour créer et lier de nouveaux services.

Elle est basée sur des pratiques que nous avons développées au cours de nos nombreuses collaborations avec des clients de tailles diverses. Nous comptons parmi ces clients aussi bien des petites entreprises développant des solutions dans le cloud que des entreprises du Fortune 500 et des éditeurs de logiciels indépendants qui migrent et développent des solutions dans le cloud. La structure d’entreprise est spécialement conçue pour plus de flexibilité afin de prendre en charge les charges de travail informatiques traditionnelles et les charges de travail agiles, comme lorsque les développeurs créent des applications de type software-as-a-service (SaaS) basées sur des fonctionnalités Azure.

La structure d’entreprise constitue le fondement de chaque nouvel abonnement dans Azure. Elle permet aux administrateurs de s’assurer que les charges de travail répondent à la configuration requise minimale en matière de gouvernance d’une organisation sans empêcher les groupes professionnels et les développeurs d’atteindre rapidement leurs objectifs.

> [!IMPORTANT]
> La gouvernance est essentielle au succès d’Azure. Cet article vise l’implémentation technique d’une structure d’entreprise, mais traite uniquement de processus plus larges et de relations entre les composants. La gouvernance de stratégies s’applique du haut vers le bas et dépend des objectifs de l’entreprise. Naturellement, la création d’un modèle de gouvernance pour Azure inclut des représentants du service informatique. Mais, plus important encore, elle doit également intégrer des responsables de groupes professionnels et de la gestion des risques et de la sécurité. Au final, une structure d’entreprise consiste à limiter les risques métier afin de faciliter la mission et la poursuite des objectifs d’une organisation.
> 
> 

L’illustration suivante décrit les composants de la structure. Elle est basée sur un plan englobant services, comptes et abonnements. Les piliers en sont des stratégies Resource Manager et des normes d’affectation de noms fortes. Le reste de la structure se base sur des fonctionnalités Azure et des fonctionnalités qui permettent de développer un environnement sécurisé et facile à gérer.

![composants d’une structure](./_images/components.png)

> [!NOTE]
> Azure a évolué rapidement depuis son lancement en 2008. Cette croissance a conduit les ingénieurs Microsoft à repenser leur approche de la gestion et du déploiement de services. Le modèle Azure Resource Manager a été introduit dans 2014 et remplace le modèle de déploiement classique. Resource Manager permet aux organisations de déployer, d’organiser et de contrôler les plus facilement ressources Azure. Resource Manager comprend la parallélisation lors de la création de ressources pour un déploiement plus rapide de solutions complexes et interdépendantes. Il inclut également un contrôle d’accès granulaire et la possibilité de baliser des ressources avec des métadonnées. Microsoft vous recommande de créer des ressources par le biais du modèle Resource Manager. La structure d’entreprise est explicitement conçue pour le modèle Resource Manager.
> 
> 

## <a name="define-your-hierarchy"></a>Définition de votre hiérarchie
La base de la structure est l’inscription à Azure Enterprise (et au portail d’entreprise). L’inscription d’une entreprise définit la forme et l’utilisation des services Azure en son sein et la structure de gouvernance principale. Dans le contrat d’entreprise, les clients sont en mesure de subdiviser l’environnement en services, comptes et abonnements. Un abonnement Azure est l’unité de base contenant toutes les ressources. Il définit également plusieurs limites au sein d’Azure, par exemple le nombre de cœurs, de ressources, etc.

![hiérarchie](./_images/agreement.png)

Chaque entreprise est différente et la hiérarchie de l’image précédente permet une grande flexibilité dans la manière dont Azure est organisé au sein de l’entreprise. Avant d’implémenter les instructions contenues dans ce document, vous devez modéliser votre hiérarchie et comprendre l’impact sur la facturation, l’accès aux ressources et la complexité.

Les trois modèles courants pour les inscriptions Azure sont :

* Le modèle **fonctionnel**
  
    ![functional](./_images/functional.png)
* Le modèle **division** 
  
    ![business](./_images/business.png)
* Le modèle **géographique**
  
    ![geographic](./_images/geographic.png)

Vous appliquez la structure au niveau de l’abonnement pour étendre les exigences de gouvernance de l’entreprise dans l’abonnement.

## <a name="naming-standards"></a>Normes d’attribution de noms
Le premier pilier de la structure est constitué de normes d’attribution de noms. Des normes d’attribution de noms bien conçues vous permettent d’identifier des ressources dans le portail, sur une facture et dans des scripts. Vous disposez déjà probablement de conventions d’attribution de noms pour l’infrastructure locale. Lorsque vous ajoutez Azure à votre environnement, vous devez étendre ces normes d’attribution de noms à vos ressources Azure. Les normes d’attribution de noms facilitent la gestion de l’environnement à tous les niveaux.

> [!TIP]
> Pour les conventions de dénomination :
> * Passez en revue et adoptez autant que possible les [modèles et pratiques recommandées](../../best-practices/naming-conventions.md). Ce guide vous permet de déterminer une norme d’attribution de noms explicite.
> * Utilisez la casse mixte (camelCasing) pour les noms des ressources (par exemple, myResourceGroup et vnetNetworkName). Remarque : il existe certaines ressources, notamment des comptes de stockage, pour lesquelles la seule option est l’utilisation de minuscules (et aucun autre caractère spécial).
> * Envisagez d’utiliser des stratégies Azure Resource Manager (décrites dans la section suivante) pour appliquer des normes d’attribution de noms.
> 
> Les conseils précédents vous aident à implémenter une convention de dénomination cohérente.

## <a name="policies-and-auditing"></a>Stratégies et audit
Le deuxième pilier de la structure implique la création de [stratégies Azure](/azure/azure-policy/azure-policy-introduction) et [l’audit du journal d’activité](/azure/azure-resource-manager/resource-group-audit). Les stratégies Resource Manager vous donnent la possibilité de gérer les risques dans Azure. Vous pouvez définir des stratégies qui garantissent la souveraineté des données en limitant, en appliquant ou en auditant certaines actions. 

* La stratégie est un système **d’autorisation** par défaut. Vous contrôlez les actions en définissant et en affectant des stratégies à des ressources qui refusent ou auditent des actions sur les ressources.
* Les stratégies sont décrites par des définitions de stratégie dans un langage de définition de stratégie (conditions if-then).
* Vous créez des stratégies avec des fichiers au format JSON (Javascript Object Notation). Après avoir défini une stratégie, vous l’affectez à une portée particulière : abonnement, groupe de ressources ou ressource.

Les stratégies incluent plusieurs actions qui permettent d’adopter une approche affinée de vos scénarios. Les actions sont les suivantes :

* **Refuser** : bloque la requête de ressource
* **Auditer** : autorise la requête, mais ajoute une ligne au journal d’activité (qui peut être utilisée pour fournir des alertes ou déclencher des runbooks)
* **Ajouter** : ajoute les informations spécifiées à la ressource. Par exemple, s’il n’existe pas de balise « CostCenter » sur une ressource, ajoutez cette balise avec une valeur par défaut.

### <a name="common-uses-of-resource-manager-policies"></a>Utilisations courantes de stratégies Resource Manager
Les stratégies Azure Resource Manager sont un outil puissant dans le kit de ressources Azure. Elles vous permettent d’éviter des coûts inattendus, d’identifier un centre de coût pour les ressources par le biais d’un balisage et de vous assurer que les exigences de conformité sont satisfaites. Lorsque les stratégies sont combinées aux fonctionnalités d’audit intégrées, vous pouvez créer des solutions complexes et flexibles. Les stratégies permettent aux entreprises de fournir les contrôles pour les charges de travail de « l’informatique traditionnelle » et « agiles », comme par exemple le développement d’applications de clients. Les modèles les plus courants pour les stratégies sont les suivants :

* **Conformité géographique/souveraineté des données** : Azure est utilisable dans différentes régions du monde. Souvent, les entreprises souhaitent contrôler l’endroit où les ressources sont créées (si vous souhaitez garantir la souveraineté des données ou garantir que les ressources sont créées près des consommateurs finaux des ressources).
* **Gestion des coûts** : un abonnement Azure peut contenir des ressources de nombreux types et à différentes échelles. Les sociétés souhaitent souvent éviter que les abonnements standard évitent l’utilisation de ressources inutilement volumineuses, qui pourrait coûter des centaines de dollars par mois ou plus.
* **Gouvernance par défaut par le biais de balises nécessaires** : l’exigence de balises est l’une des fonctionnalités les plus courantes et les plus souhaitées. Avec les stratégies Azure Resource Manager, les entreprises sont en mesure de garantir qu’une ressource est correctement balisée. Les balises les plus courantes sont : service, propriétaire de ressource et type d’environnement (par exemple, production, test et développement)

**Exemples**

Abonnement de type « informatique traditionnelle » pour les applications métier

* Application de balises de type service et propriétaire à toutes les ressources
* Restriction de la création de ressources à la région Amérique du Nord
* Restriction de la capacité à créer des machines virtuelles de série G et des clusters HDInsight

Environnement « Agile » pour une division de création d’applications cloud

* Pour répondre aux exigences de souveraineté des données, activez la création de ressources UNIQUEMENT dans une région spécifique.
* Appliquez la balise d’environnement à toutes les ressources. Si une ressource est créée sans balise, ajoutez la balise **Environnement : inconnu** à la ressource.
* Auditez lorsque les ressources sont créées en dehors de l’Amérique du Nord, mais n’empêchez pas
* l’audit lorsque des ressources onéreuses sont créées.

> [!TIP]
> L’utilisation la plus courante de stratégies Resource Manager sur plusieurs organisations consiste à contrôler *où* les ressources peuvent être créées et les *types* de ressources qui peuvent être créés. En plus du contrôle de *l’endroit* et du *type*, de nombreuses entreprises utilisent des stratégies pour s’assurer que les ressources ont les métadonnées appropriées pour la facturation de la consommation. Nous vous recommandons d’appliquer des stratégies au niveau de l’abonnement pour :
> 
> * la conformité géographique/la souveraineté des données ;
> * la gestion des coûts ;
> * les balises nécessaires (déterminées selon les besoins métier, notamment BillTo et propriétaire de l’application).
> 
> Vous pouvez appliquer des stratégies supplémentaires à des niveaux inférieurs de l’étendue.
> 
> 

### <a name="audit---what-happened"></a>Audit - que s’est-il passé ?
Pour voir la manière dont votre environnement fonctionne, vous devez auditer l’activité de l’utilisateur. La plupart des types de ressources dans Azure créent des journaux de diagnostic capables d’analyser par le biais d’un outil de journal ou dans Azure Log Analytics. Vous pouvez collecter des journaux d’activité sur plusieurs abonnements pour fournir une vue au niveau d’un service ou de l’entreprise. Les enregistrements d’audit sont un outil de diagnostic important et un mécanisme essentiel pour déclencher des événements dans l’environnement Azure.

Les journaux d’activité du Resource Manager permettent de déterminer les **opérations** qui ont eu lieu et qui les a exécutées. Les journaux d’activité peuvent être collectés et agrégés à l’aide d’outils tels que Log Analytics.


## <a name="resource-group"></a>Groupe de ressources
Resource Manager vous permet de répartir des ressources dans des groupes explicites pour la gestion, la facturation ou l’affinité naturelle. Comme mentionné précédemment, Azure dispose de deux modèles de déploiement. Dans l’ancien modèle standard, l’unité de base de gestion était l’abonnement. Il était difficile de répartir des ressources au sein d’un abonnement, ce qui entraînait la création d’un grand nombre d’abonnements. Avec le modèle Resource Manager, des groupes de ressources ont été introduits. Les groupes de ressources sont des conteneurs de ressources qui ont un cycle de vie commun ou partagent un attribut tel que « tous les serveurs SQL » ou « Application A ».

Les groupes de ressources ne peuvent pas être contenus les uns dans les autres et les ressources ne peuvent appartenir qu’à un groupe de ressources. Vous pouvez appliquer certaines actions à toutes les ressources dans un groupe de ressources. Par exemple, la suppression d’un groupe de ressources supprime toutes les ressources du groupe de ressources. En règle générale, vous placez une application complète ou un système associé dans le même groupe de ressources. Par exemple, une application à trois niveaux appelée Contoso Web Application contient le serveur web, le serveur d’applications et le serveur SQL dans le même groupe de ressources.

> [!TIP]
> La façon dont vous organisez vos groupes de ressources peut aller de charges de travail de type « informatique traditionnelle » à des charges de travail de type « informatique agile » :
> 
> * Les charges de travail de type « informatique traditionnelle » sont généralement regroupées par éléments au sein du même cycle de vie, par exemple en tant qu’application. Le regroupement par application permet de gestion des applications individuelles.
> * Les charges de travail de type « agile » ont tendance à se concentrer sur les applications cloud pour les clients externes. Les groupes de ressources doivent refléter les couches de déploiement (par exemple en tant que couche Web ou couche Application) et de gestion.
> 
> Comprendre votre charge de travail vous aide à développer une stratégie de groupe de ressources.


## <a name="resource-tags"></a>Balises de ressource
Comme les utilisateurs de votre organisation ajoutent des ressources à l’abonnement, il devient plus en plus important d’associer des ressources au service, au client et à l’environnement appropriés. Vous pouvez attacher des métadonnées aux ressources par le biais de [balises](/azure/azure-resource-manager/resource-group-using-tags). Vous utilisez des balises pour fournir des informations sur la ressource ou le propriétaire. Les balises vous permettent non seulement d’agréger et de regrouper les ressources de différentes façons, mais également d’utiliser ces données pour les besoins de facturation interne. Vous pouvez baliser des ressources avec jusqu'à 15 paires clé-valeur. 

Les balises de ressources sont flexibles et doivent être attachées à la plupart des ressources. Exemples de balises de ressources courantes :

* BillTo
* Service (ou unité commerciale)
* Environnement (production, déploiement intermédiaire, développement)
* Couche (couche Web, couche Application)
* Propriétaire de l’application
* ProjectName

![tags](./_images/resource-group-tagging.png)

Pour plus d’exemples de balises, consultez la rubrique [Conventions d’affectation de noms recommandées pour les ressources Azure](../../best-practices/naming-conventions.md).

> [!TIP]
> Envisagez une stratégie qui impose de balisage pour les éléments suivants :
> 
> * Groupes de ressources
> * Stockage
> * Virtual Machines
> * Environnements de service d’applications/serveurs web
> 
> Cette stratégie de balisage identifie les métadonnées nécessaires dans vos abonnements pour l’activité, la finance, la sécurité, la gestion des risques et la gestion globale de l’environnement. 



## <a name="role-based-access-control"></a>Contrôle d’accès en fonction du rôle
Vous vous demandez probablement « qui doit avoir accès aux ressources ? » et « comment contrôler cet accès ? ». Il est crucial d’autoriser ou non l’accès au portail Azure et de contrôler l’accès aux ressources dans le portail. 

Lorsqu’Azure a été initialement lancé, les contrôles d’accès à un abonnement étaient basiques : administrateur ou coadministrateur. L’accès à un abonnement dans le modèle classique impliquait l’accès à toutes les ressources dans le portail. Ce manque de contrôle précis a conduit à une prolifération d’abonnements afin de fournir un niveau de contrôle d’accès raisonnable pour une inscription Azure.

Cette prolifération d’abonnements n’est plus nécessaire. Avec un [contrôle d’accès en fonction du rôle](/azure/role-based-access-control/overview), vous pouvez affecter des utilisateurs aux rôles standards (par exemple, les rôles « lecteur » et « rédacteur » courants). Vous pouvez également définir des rôles personnalisés.

> [!TIP]
> Pour implémenter un contrôle d’accès en fonction du rôle :
> * Connectez votre magasin d’identités d’entreprise (généralement Active Directory) à Azure Active Directory à l’aide de l’outil AD Connect.
> * Contrôlez les rôles Administrateur/Coadministrateur d’un abonnement à l’aide d’une identité gérée. **N’** affectez pas Admin/Co-admin à un nouveau propriétaire d’abonnement. Utilisez plutôt des rôles RBAC (contrôle d’accès en fonction du rôle) pour octroyer des droits de**propriétaire** à un groupe ou à une personne.
> * Ajoutez des utilisateurs Azure à un groupe (par exemple, les propriétaires de l’application X) dans Active Directory. Utilisez le groupe synchronisé pour affecter aux membres de groupes les droits nécessaires pour gérer le groupe de ressources contenant l’application.
> * Suivez le principe de l’octroi des **privilèges minimum** nécessaires pour exécuter le travail prévu. Par exemple : 
>   * Groupe de déploiement : un groupe qui ne peut que déployer des ressources.
>   * Gestion des machines virtuelles : un groupe qui est capable de redémarrer les machines virtuelles (pour les opérations)
> 
> Ces conseils vous aident à gérer l’accès utilisateur au sein de votre abonnement.

## <a name="azure-resource-locks"></a>Verrous de ressource Azure
À mesure que votre organisation ajoute des services principaux à l’abonnement, il devient de plus en plus important de s’assurer que ces services sont disponibles afin d’éviter une interruption de service. Les [verrous de ressource](/azure/azure-resource-manager/resource-group-lock-resources) vous permettent de limiter les opérations sur les ressources de grande valeur si leur suppression ou leur modification pourraient avoir un impact significatif sur vos applications ou votre infrastructure de cloud. Vous pouvez appliquer des verrous à un abonnement, un groupe de ressources ou une ressource. En règle générale, vous appliquez des verrous à des ressources fondamentales telles que les réseaux virtuels, les passerelles et les comptes de stockage. 

Les verrous de ressource prennent actuellement en charge deux valeurs : CanNotDelete et ReadOnly. CanNotDelete signifie que les utilisateurs (disposant des droits appropriés) peuvent toujours lire ou modifier une ressource, mais pas la supprimer. ReadOnly signifie que les utilisateurs autorisés ne peuvent pas supprimer ou modifier une ressource.

Pour créer ou supprimer des verrous de gestion, vous devez avoir accès à des actions `Microsoft.Authorization/*` ou `Microsoft.Authorization/locks/*`.
Parmi les rôles prédéfinis, seuls les rôles Propriétaire et Administrateur de l'accès utilisateur peuvent effectuer ces actions.

> [!TIP]
> Les options de réseau principal doivent être protégées par des verrous. Une suppression accidentelle d’une passerelle ou d’un réseau VPN de site à site serait désastreuse pour un abonnement Azure. Azure ne vous permet pas de supprimer un réseau virtuel en cours d’utilisation, mais la mise en œuvre de restrictions supplémentaires est une précaution utile. 
> 
> * Réseau virtuel : CanNotDelete
> * Groupe de sécurité réseau : CanNotDelete
> * Stratégies : CanNotDelete
> 
> Les stratégies sont également essentielles à la maintenance de contrôles appropriés. Nous recommandons l’application d’un verrou **CanNotDelete** aux stratégies en cours d’utilisation.

## <a name="core-networking-resources"></a>Ressources de réseau principal
L’accès aux ressources peut être interne (dans le réseau de l’entreprise) ou externe (par le biais d’internet). Il est facile pour les utilisateurs de votre organisation de placer par inadvertance des ressources au mauvais endroit et de potentiellement les exposer à un accès malveillant. Comme pour les appareils locaux, les entreprises doivent ajouter des contrôles appropriés pour s’assurer que les utilisateurs d’Azure prennent les bonnes décisions. Pour la gouvernance de l’abonnement, nous identifions des ressources principales qui fournissent le contrôle d’accès de base. Les ressources principales sont constituées des éléments suivants :

* Les **réseaux virtuels** sont des objets Conteneur de sous-réseaux. Bien qu’ils ne soient pas strictement nécessaires, ils sont souvent utilisés lors de la connexion d’applications à des ressources d’entreprise internes.
* Les **Groupes de sécurité réseau** sont similaires à un pare-feu et fournissent des règles pour la manière dont une ressource peut « communiquer » à travers le réseau. Ils permettent un contrôle granulaire sur la manière dont un sous-réseau (ou une machine virtuelle) peut se connecter à Internet ou à d’autres sous-réseaux dans le même réseau virtuel, ou même si une connexion doit être établie.

![réseau principal](./_images/core-network.png)

> [!TIP]
> Pour la mise en réseau :
> * Créez des réseaux virtuels dédiés aux charges de travail externes et aux charges de travail internes. Cette approche réduit le risque de placement par inadvertance de machines virtuelles destinées à des charges de travail internes dans un espace externe.
> * Configurez des groupes de sécurité réseau pour limiter l’accès. Au minimum, bloquez l’accès à internet depuis des réseaux virtuels internes et bloquez l’accès au réseau d’entreprise depuis des réseaux virtuels externes.
> 
> Ces conseils vous aident à mettre en œuvre des ressources de mise en réseau sécurisées.

## <a name="automation"></a>Automatisation
La gestion individuelle des ressources prend du temps et sujette aux erreurs dans certaines opérations. Azure fournit diverses fonctionnalités d’automatisation, notamment Azure Automation, Logic Apps et Azure Functions. [Azure Automation](/azure/automation/automation-intro) permet aux administrateurs de créer et définir des runbooks afin de gérer les travaux courants de gestion des ressources. Vous créez des runbooks à l’aide d’un éditeur de code PowerShell ou d’un éditeur graphique. Vous pouvez générer des flux de travail complexes à plusieurs étapes. Azure Automation est souvent utilisé pour gérer les travaux courants telles que l’arrêt des ressources inutilisées ou la création de ressources en réponse à un déclencheur spécifique sans avoir besoin d’une intervention humaine.

> [!TIP]
> Pour l’automatisation :
> * Créez un compte Azure Automation et passez en revue les runbooks disponibles (graphique et ligne de commande) disponibles dans la [Galerie de runbooks](/azure/automation/automation-runbook-gallery).
> * Importer et personnaliser des runbooks clés pour votre usage personnel.
> 
> Un scénario courant consiste à démarrer/arrêter les machines virtuelles sur une planification. Il existe des exemples de runbooks disponibles dans la galerie qui traitent ce scénario et vous apprennent à l’étendre.
> 
> 

## <a name="azure-security-center"></a>Azure Security Center
Parmi les plus grands blocages à l’adoption du cloud, on compte les préoccupations liées à la sécurité. Les gestionnaires des risques informatiques et les services de sécurité doivent s’assurer que les ressources dans Azure sont sécurisées. 

[L’Azure Security Center](/azure/security-center/security-center-intro) fournit une vue centralisée de l’état de la sécurité des ressources dans les abonnements et fournit des recommandations qui aident à empêcher la compromission des ressources. Il peut autoriser des stratégies plus granulaires (par exemple, l’application de stratégies à des groupes de ressources spécifiques qui permettent à l’entreprise d’adapter sa position en fonction des risques auxquels elle est confrontée). Enfin, l’Azure Security Center est une plateforme ouverte qui permet aux partenaires de Microsoft et aux éditeurs de logiciels indépendants de créer un logiciel qui se connecte à l’Azure Security Center pour améliorer leurs fonctionnalités. 

> [!TIP]
> L’Azure Security Center est activé par défaut dans chaque abonnement. Toutefois, vous devez activer la collecte des données depuis des machines virtuelles pour autoriser l’Azure Security Center à installer son agent et à commencer la collecte des données.
> 
> ![la collection de données](./_images/data-collection.png)
> 
> 

## <a name="next-steps"></a>Étapes suivantes
* Maintenant que vous connaissez mieux la gouvernance des abonnements, il est temps pour voir ces recommandations dans la pratique. Voir la rubrique [Examples of implementing Azure subscription governance](azure-scaffold-examples.md) (Exemples d’implémentation de la gouvernance des abonnements Azure).

> [!div class="nextstepaction"]
> [Implémenter un exemple](azure-scaffold-examples.md)
