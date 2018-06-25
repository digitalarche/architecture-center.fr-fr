---
title: 'Adoption d’Azure : niveau intermédiaire'
description: Décrit le niveau intermédiaire de connaissances que doit posséder une entreprise pour adopter Azure
author: petertay
ms.openlocfilehash: 227d9558647ed8076b2832d95e192f2f0c43b9db
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206359"
---
# <a name="azure-cloud-adoption-guide-intermediate-overview"></a>Guide d’adoption du cloud Azure : vue d’ensemble du niveau intermédiaire

Pendant la phase d’adoption fondamentale, vous avez eu l’occasion de vous familiariser avec les concepts de base de la gouvernance des ressources Azure. La phase fondamentale a été conçue pour vous guider dans votre parcours d’adoption d’Azure ; vous y avez appris à déployer une simple charge de travail avec une petite équipe. Dans la réalité, la plupart des organisations possèdent de nombreuses équipes qui travaillent simultanément sur différentes charges de travail. Comme vous pouvez l’imaginer, un simple modèle de gouvernance ne suffit pas pour gérer les scénarios d’organisation et de développement plus complexes.

Le niveau intermédiaire de l’adoption d’Azure consiste à concevoir un modèle de gouvernance pour plusieurs équipes travaillant sur plusieurs nouvelles charges de travail de développement Azure.  

Cette étape du guide s’adresse aux membres suivants de votre organisation :
- *Finance :* propriétaire des ressources financières engagées dans Azure, il est responsable du développement des stratégies et procédures dédiées au suivi des coûts de consommation des ressources, y compris la facturation et la refacturation.
- *Informatique centrale :* chargée d’administrer les ressources cloud de votre organisation, y compris la gestion des ressources et l’accès aux ressources, ainsi que l’analyse et le contrôle d’intégrité des charges de travail.
- *Propriétaire de l’infrastructure partagée* : rôles techniques responsables de la connectivité réseau entre le réseau local et le cloud.
- *Opérations de sécurité* : responsables de l’implémentation de la stratégie de sécurité nécessaire pour étendre la limite de sécurité locale à Azure. Peuvent également gérer l’infrastructure de sécurité dans Azure pour le stockage des clés secrètes.
- *Propriétaire de la charge de travail :* responsable de la publication d’une charge de travail dans Azure. Selon la structure des équipes de développement de votre organisation, il peut s’agir d’un responsable du développement, d’un responsable de gestion de programmes ou d’un responsable d’ingénierie de compilation. Une partie du processus de publication peut inclure le déploiement de ressources dans Azure.
  - *Contributeur de la charge de travail :* chargé de contribuer à la publication d’une charge de travail dans Azure. Peut avoir besoin d’un accès en lecture aux ressources Azure pour l’analyse des performances ou le paramétrage. N’a pas besoin de l’autorisation de créer, mettre à jour ou supprimer des ressources.

## <a name="section-1-azure-concepts-for-multiple-workloads-and-multiple-teams"></a>Section 1 : concepts Azure pour plusieurs charges de travail et plusieurs équipes

Dans la phase d’adoption fondamentale, vous avez appris certaines notions de base sur les mécanismes internes d’Azure et avez découvert les modalités de création, de lecture, de mise à jour et de suppression des ressources. Vous avez également acquis des connaissances sur la notion d’identité et savez maintenant qu’Azure utilise uniquement Azure Active Directory (AD) pour authentifier et autoriser les utilisateurs qui ont besoin d’accéder à ces ressources.

Vous avez également appris à configurer les outils de gouvernance d’Azure pour gérer la manière dont votre organisation utilise les ressources Azure. Pendant la phase fondamentale, nous avons vu comment gérer l’accès d’une seule équipe aux ressources nécessaires au déploiement d’une simple charge de travail. Mais dans la réalité, votre organisation recense plusieurs équipes qui travaillent simultanément sur plusieurs charges de travail. 

Avant de commencer, voyons ce que l’on entend précisément par la notion de **charge de travail**. Cette expression définit généralement une unité arbitraire de fonctionnalité, par exemple une application ou un service. Une charge de travail est envisagée sous l’angle des artefacts de code qui sont déployés sur un serveur, ainsi que de tous les autres services (base de données, par exemple) nécessaires. Cette définition se révèle pertinente dans le cas d’une application ou d’un service local(e), mais dans le cloud, elle mérite d’être développée. 

Dans le cloud, une charge de travail englobe non seulement tous les artefacts, mais également les ressources cloud. Les ressources cloud sont ici incluses dans notre définition en raison d’un concept appelé **infrastructure en tant que code**. Comme vous l’avez appris dans l’article explicatif relatif au fonctionnement d’Azure, les ressources dans Azure sont déployées par un service d’orchestrateur. Le service d’orchestrateur expose cette fonctionnalité via une API web, laquelle peut être appelée à l’aide de plusieurs outils, tels que Powershell, l’interface de ligne de commande (CLI) d’Azure et le portail Azure. Cela signifie que nous pouvons spécifier nos ressources dans un fichier lisible sur ordinateur qui peut être stocké en même temps que les artefacts de code associés à notre application.

Nous pouvons ainsi définir une charge de travail en termes d’artefacts de code et de ressources cloud nécessaires, ce qui nous permet **d’isoler** nos charges de travail. Les charges de travail peuvent être isolées par la façon dont les ressources sont organisées, par la topologie du réseau ou par d’autres attributs. L’isolement des charges de travail a pour but d’associer les ressources spécifiques d’une charge de travail à une équipe donnée, afin que cette équipe puisse gérer indépendamment tous les aspects de ces ressources. Cela permet à plusieurs équipes de partager des services de gestion de ressources dans Azure tout en évitant une suppression ou une modification involontaire des ressources d’une autre équipe.

Cet isolement permet également de favoriser ce que l’on appelle le [DevOps](https://azure.microsoft.com/solutions/devops/). Le DevOps inclut les pratiques de développement logiciel couvrant le développement logiciel et les opérations informatiques décrits ci-dessus, mais en y ajoutant autant que possible le recours à l’automatisation. Le principe d’intégration continue/livraison continue (CI/CD) est au cœur du DevOps. L’intégration continue fait référence aux processus de génération automatisés qui sont exécutés à chaque fois qu’un développeur valide une modification de code. La livraison continue renvoie à des processus automatisés qui déploient ce code vers différents **environnements**, par exemple un **environnement de développement** à des fins de test ou un **environnement de production** pour le déploiement final.

## <a name="section-2-governance-design-for-multiple-teams-and-multiple-workloads"></a>Section 2 : conception de la gouvernance pour plusieurs équipes et plusieurs charges de travail

Dans la [phase fondamentale](/azure/architecture/cloud-adoption-guide/adoption-intro/overview) du guide d’adoption du cloud Azure, vous avez découvert le concept de la gouvernance du cloud. Vous avez appris à concevoir un modèle de gouvernance simple pour une seule équipe travaillant sur une charge de travail unique. 

Dans la phase intermédiaire, le [guide de conception de la gouvernance](governance-design-guide.md) développe ces concepts fondamentaux pour ajouter plusieurs équipes, plusieurs charges de travail et plusieurs environnements. Une fois que vous aurez parcouru les exemples de ce document, vous pourrez appliquer les principes de conception à la conception et à l’implémentation du propre modèle de gouvernance de votre organisation.

## <a name="section-3-implementing-a-resource-management-model"></a>Section 3 : implémentation d’un modèle de gestion des ressources

Le modèle de gouvernance du cloud de votre organisation représente l’intersection entre les outils de gestion de l’accès aux ressources Azure, vos employés et les règles de gestion d’accès que vous avez définies. Dans le guide de conception de la gouvernance, vous avez découvert plusieurs modèles différents pour gouverner l’accès aux ressources Azure. Nous allons maintenant parcourir les étapes nécessaires pour implémenter le modèle de gestion des ressources avec un seul abonnement pour chacun des environnements **d’infrastructure partagée**, de **production** et de **développement** du guide de conception. Nous allons prendre un seul **propriétaire d’abonnement** pour les trois environnements. Chaque charge de travail sera isolée dans un **groupe de ressources** avec un **propriétaire de charge de travail** disposant du rôle de **contributeur**.

> [!NOTE]
> Pour en savoir plus sur la relation entre les comptes et les abonnements Azure, consultez [Présentation de l'accès aux ressources dans Azure][understand-resource-access-in-azure]. 

Procédez comme suit :

1. Créez un [compte Azure](/azure/active-directory/sign-up-organization) si vous n’en avez pas encore. La personne qui s’inscrit pour le compte Azure devient l’administrateur de compte Azure et la direction de votre organisation doit choisir une personne pour assumer ce rôle. Cette personne possède les responsabilités suivantes :
    * Créer des abonnements ;
    * Créer et administrer les locataires [Azure Active Directory (AD)](/azure/active-directory/active-directory-whatis) qui stockent les identités d’utilisateur pour ces abonnements.    
2. L’équipe dirigeante de votre organisation désigne les personnes qui auront les responsabilités suivantes :
    * Gestion des identités d’utilisateur ; un [locataire Azure AD](/azure/active-directory/develop/active-directory-howto-tenant) est créé par défaut lors de la création du compte Azure de votre organisation et l’administrateur de compte est ajouté en tant [qu’administrateur global Azure AD](/azure/active-directory/active-directory-assign-admin-roles-azure-portal#details-about-the-global-administrator-role) par défaut. Votre organisation peut désigner un autre utilisateur chargé de gérer les identités d’utilisateur en [attribuant à cet utilisateur le rôle d’administrateur global Azure AD](/azure/active-directory/active-directory-users-assign-role-azure-portal). 
    * Abonnements, ce qui signifie que ces utilisateurs :
        * Gèrent les coûts associés à l’utilisation des ressources dans cet abonnement ;
        * Implémentent et gèrent le modèle d’autorisation de privilège minimal pour accéder aux ressources ; et
        * Effectuent un suivi des limites de service.
    * Services d’infrastructure partagée (si votre organisation décide d’utiliser ce modèle), ce qui signifie que cet utilisateur est responsable de :
        * La connectivité entre le réseau local et Azure ; et 
        * La propriété de la connectivité réseau dans Azure via l’appariement de réseaux virtuels.
    * Propriétaires de charges de travail. 
3. L’administrateur global Azure AD crée les [nouveaux comptes d’utilisateur](/azure/active-directory/add-users-azure-active-directory) pour :
    * La personne qui sera le **propriétaire de l’abonnement** pour chaque abonnement associé à chaque environnement. Notez que cela est nécessaire uniquement si **l’administrateur de services fédérés** de l’abonnement n’est pas chargé de gérer l’accès aux ressources pour chaque abonnement/environnement ;
    * La personne qui sera **l’utilisateur des opérations réseau** ; et
    * Les personnes qui seront les **propriétaires de charges de travail**.
4. L’administrateur de compte Azure crée les trois abonnements suivants à l’aide du [portail de compte Azure](https://account.azure.com) :
    * Un abonnement pour l’environnement **d’infrastructure partagée** ;
    * Un abonnement pour l’environnement de **production** ; 
    * Un abonnement pour l’environnement de **développement**. 
5. L’administrateur de compte Azure [ajoute le propriétaire de service d’abonnement à chaque abonnement](/azure/billing/billing-add-change-azure-subscription-administrator#add-an-rbac-owner-admin-for-a-subscription-in-azure-portal).
6. Un processus d’approbation est créé pour permettre aux **propriétaires de charges de travail** de demander la création de groupes de ressources. Le processus d’approbation peut être implémenté de différentes façons (par e-mail par exemple) mais vous pouvez également utiliser un outil de gestion des processus tel que les [flux de travail Sharepoint](https://support.office.com/article/introduction-to-sharepoint-workflow-07982276-54e8-4e17-8699-5056eff4d9e3). Le processus d’approbation peut se dérouler comme suit :  
    * Le **propriétaire de la charge de travail** prépare une nomenclature pour les ressources Azure nécessaires, dans l’environnement de **développement** et/ou dans l’environnement de **production**, et la soumet au **propriétaire de l’abonnement**.
    * Le **propriétaire de l’abonnement** passe en revue la nomenclature et valide les ressources requises pour s’assurer que les ressources demandées sont adaptées à l’utilisation prévue ; par exemple en vérifiant que les [ tailles de machine virtuelle](/azure/virtual-machines/windows/sizes) demandées sont correctes.
    * Si la demande n’est pas approuvée, le **propriétaire de la charge de travail** en est informé. Si la demande est approuvée, le **propriétaire de l’abonnement** [crée le groupe de ressources demandé](/azure/azure-resource-manager/resource-group-portal#manage-resource-groups) suivant les [conventions d’affectation de noms](/azure/architecture/best-practices/naming-conventions) de votre organisation, [ajoute le **propriétaire de la charge de travail**](/azure/role-based-access-control/role-assignments-portal#add-access) avec le rôle de [ **contributeur**](/azure/role-based-access-control/built-in-roles#contributor) et envoie une notification au **propriétaire de la charge de travail** pour l’informer de la création du groupe de ressources.
7. Un processus d’approbation est créé pour permettre aux propriétaires de charges de travail de demander au propriétaire de l’infrastructure partagée l’établissement d’une connexion pour l’appariement des réseaux virtuels. Comme dans l’étape précédente, ce processus d’approbation peut être implémenté par e-mail ou à l’aide d’un outil de gestion de processus.

Maintenant que vous avez implémenté votre modèle de gouvernance, vous pouvez déployer vos services d’infrastructure partagée.

## <a name="section-4-deploy-shared-infrastructure-services"></a>Section 4 : déployer des services d’infrastructure partagée

Il existe plusieurs [architectures de réseau hybride de référence ](/azure/architecture/reference-architectures/hybrid-networking/) que votre organisation peut utiliser pour connecter son réseau local à Azure. Chacune de ces architectures de référence inclut un déploiement qui requiert un identifiant d’abonnement. Au cours du déploiement, spécifiez l’identifiant d’abonnement correspondant à l’abonnement associé à votre environnement **d’infrastructure partagée**. Vous devez également modifier les fichiers de modèle pour spécifier le groupe de ressources géré par votre utilisateur des **opérations réseau** ou utiliser les groupes de ressources par défaut dans le déploiement et y ajouter l’utilisateur des **opérations réseau**  avec le rôle de **contributeur**.

<!-- links -->
[understand-resource-access-in-azure]: /azure/role-based-access-control/rbac-and-directory-admin-roles