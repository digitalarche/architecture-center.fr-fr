---
title: Bot conversationnel pour les réservations d’hôtel
titleSuffix: Azure Example Scenarios
description: Créez un bot conversationnel pour les applications de commerce avec Azure Bot Service.
author: iainfoulds
ms.date: 07/05/2018
ms.openlocfilehash: 31a7384b11262ac967ab5f8a6c5e7f17e9a00b6f
ms.sourcegitcommit: bb7fcffbb41e2c26a26f8781df32825eb60df70c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/20/2018
ms.locfileid: "53643847"
---
# <a name="conversational-chatbot-for-hotel-reservations-on-azure"></a>Bot conversationnel pour les réservations d’hôtel sur Azure

Cet exemple de scénario s’applique aux entreprises qui souhaitent intégrer aux applications un bot conversationnel. Dans ce scénario, un bot conversationnel en C# est utilisé pour une chaîne d’hôtels qui permet aux clients de vérifier la disponibilité des chambres et de les réserver via une application web ou mobile.

Utilisations éventuelles : permettre aux clients d’afficher la disponibilité de l’hôtel et de réserver des chambres, consulter le menu à emporter d’un restaurant et passer une commande, ou rechercher et commander des impressions de photos. Traditionnellement, les entreprises devaient embaucher et former des conseillers pour le service client afin de répondre à ces demandes de clients, lesquels devaient patienter jusqu’à ce qu’un agent soit disponible pour leur fournir de l’aide.

En utilisant les services Azure tels que Bot Service et Language Understanding ou API Microsoft Speech, les sociétés peuvent aider les clients et traiter leurs commandes ou leurs réservations à l’aide de bots automatisés, évolutifs.

## <a name="relevant-use-cases"></a>Cas d’usage appropriés

Les autres cas d’usage appropriés sont les suivants :

- La consultation du menu à emporter d’un restaurant et le passage d’une commande
- La vérification de la disponibilité d’un hôtel et la réservation d’une chambre
- La recherche des photos disponibles et la commande des impressions

## <a name="architecture"></a>Architecture

![Présentation de l’architecture des composants Azure impliqués dans un bot conversationnel][architecture]

Ce scénario présente un bot conversationnel qui fonctionne comme le concierge d’un hôtel. Les données circulent dans le scénario comme suit :

1. Le client accède au bot conversationnel avec une application web ou mobile.
2. L’utilisateur est authentifié à l’aide d’Azure Active Directory B2C (entreprise-client).
3. En interagissant avec Bot Service, l’utilisateur demande des informations sur la disponibilité de l’hôtel.
4. Cognitive Services traite la requête en langage naturel pour comprendre la communication du client.
5. Lorsque l’utilisateur est satisfait des résultats, le bot ajoute ou met à jour la réservation du client dans une base de données SQL Database.
6. Application Insights collecte les données de télémétrie d’exécution au cours du processus pour aider l’équipe DevOps en fournissant des informations sur les performances et l’utilisation du bot.

### <a name="components"></a>Composants

- [Azure Active Directory][aad-docs] est le service informatique mutualisé de Microsoft pour la gestion des annuaires et des identités. Azure AD prend en charge un connecteur B2C, ce qui vous permet d’identifier les personnes à l’aide d’ID externes comme un compte Google, Facebook ou Microsoft.
- [App Service][appservice-docs] vous permet de créer et d’héberger des applications web dans le langage de programmation de votre choix sans gérer l’infrastructure.
- [Bot Service][botservice-docs] fournit des outils pour créer, tester, déployer et gérer des robots intelligents.
- [Cognitive Services][cognitive-docs] permet d’utiliser des algorithmes intelligents pour voir, écouter, énoncer, comprendre et interpréter les besoins des utilisateurs selon des modes de communication naturels.
- [SQL Database][sqldatabase-docs] est un service de base de données cloud relationnelle entièrement géré qui fournit la compatibilité du moteur SQL Server.
- [Application Insights][appinsights-docs] est un service de gestion des performances des applications (APM) extensible qui vous permet de surveiller les performances des applications, telles que votre bot conversationnel.

### <a name="alternatives"></a>Autres solutions

- [L’API Microsoft Speech][speech-api] peut être utilisée pour modifier la façon dont les clients interagissent avec votre bot.
- [QnA Maker][qna-maker] peut être utilisé pour ajouter rapidement des connaissances à votre bot à partir d’un contenu semi-structuré comme un Forum Aux Questions.
- [Traduction de texte Translator Text][translator] est un service auquel vous pouvez penser pour ajouter facilement la prise en charge multilingue à votre bot.

## <a name="considerations"></a>Considérations

### <a name="availability"></a>Disponibilité

Ce scénario utilise Azure SQL Database pour stocker les réservations de clients. SQL Database inclut des bases de données redondantes dans une zone, des groupes de basculement et la géoréplication. Pour en savoir plus, consultez la section relative aux [fonctionnalités de disponibilité d’Azure SQL Database][sqlavailability-docs].

Pour consulter d’autres rubriques relatives à la disponibilité, consultez la [liste de contrôle de la disponibilité][availability] dans le Centre des architectures Azure.

### <a name="scalability"></a>Extensibilité

Ce scénario utilise Azure App Service. Avec App Service, vous pouvez automatiquement mettre à l’échelle le nombre d’instances qui exécutent votre bot. Cette fonctionnalité vous permet de faire face à la demande de la clientèle pour votre application web et votre bot conversationnel. Pour plus d’informations sur la mise à l’échelle automatique, voir [Meilleures pratiques de mise à l’échelle automatique][autoscaling] dans le Centre des architectures Azure.

Pour consulter d’autres rubriques relatives à l’extensibilité, consultez la [liste de contrôle de l’extensibilité][scalability] dans le Centre des architectures Azure.

### <a name="security"></a>Sécurité

Ce scénario utilise Azure Active Directory B2C (entreprise-client) pour authentifier les utilisateurs. Avec AAD B2C, votre bot conversationnel ne stocke pas les informations des comptes clients sensibles ni les informations d’identification. Pour plus d’informations, consultez la [Vue d’ensemble d’Azure Active Directory B2C][aadb2c-docs].

Les informations stockées dans Azure SQL Database sont chiffrées au repos avec un chiffrement Transparent Data Encryption (TDE). SQL Database présente également la fonctionnalité Always Encrypted qui chiffre les données durant l’interrogation et le traitement. Pour plus d’informations sur la sécurité de SQL Database, consultez la section relative à la [sécurité et à la conformité d’Azure SQL Database][sqlsecurity-docs].

Pour obtenir des conseils d’ordre général sur la conception de solutions sécurisées, consultez la [documentation sur la sécurité Azure][security].

### <a name="resiliency"></a>Résilience

Ce scénario utilise Azure SQL Database pour stocker les réservations de clients. SQL Database inclut des bases de données redondantes dans une zone, des groupes de basculement, la géoréplication et des sauvegardes automatiques. Ces fonctionnalités permettent à l’application de continuer à s’exécuter en cas d’événement de maintenance ou de panne. Pour en savoir plus, consultez la section relative aux [fonctionnalités de disponibilité d’Azure SQL Database][sqlavailability-docs].

Pour surveiller l’intégrité de votre application, ce scénario utilise Application Insights. Avec Application Insights, vous pouvez générer des alertes et résoudre les problèmes de performances qui auraient un impact sur l’expérience client et la disponibilité du bot conversationnel. Pour plus d’informations, consultez l’article [Présentation d’Application Insights][appinsights-docs].

Pour obtenir des conseils d’ordre général sur la conception de solutions résilientes, consultez l’article [Conception d’applications résilientes pour Azure][resiliency].

## <a name="deploy-the-scenario"></a>Déployez le scénario

Ce scénario est divisé en trois composants pour vous permettre d’explorer les zones sur lesquelles vous êtes le plus axé :

- [Composants d’infrastructure](#deploy-infrastructure-components). Utilisez un modèle Azure Resource Manager pour déployer les composants d’infrastructure de base d’un service App Service, de Web App, d’Application Insights, d’un compte de stockage, de SQL Server et d’une base de données.
- [Bot conversationnel d’application web](#deploy-web-app-chatbot). Utilisez l’interface de ligne de commande Azure pour déployer un bot avec Bot Service et l’application Language Understanding Intelligent Service (LUIS).
- [Exemple d’application de bot conversationnel en C#](#deploy-chatbot-c-application-code). Utilisez Visual Studio pour vérifier le code de l’exemple d’application en C# de réservation d’hôtel et le déployer vers un bot dans Azure.

### <a name="prerequisites"></a>Prérequis

Vous devez disposer d’un compte Azure existant. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

### <a name="walk-through"></a>Procédure pas à pas

Pour déployer les composants d’infrastructure avec un modèle Resource Manager, procédez comme suit.

<!-- markdownlint-disable MD033 -->

1. Cliquez sur le bouton **Déployer sur Azure** :<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fcommerce-chatbot.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Attendez l’ouverture de la solution Template deployment dans le portail Azure, puis procédez comme suit :
   - Sélectionnez **Créer un nouveau groupe de ressources**, puis indiquez un nom, par exemple *myCommerceChatBotInfrastructure* dans la zone de texte.
   - Dans la zone de liste déroulante **Emplacement**, sélectionnez une région.
   - Indiquez un nom d’utilisateur et un mot de passe sécurisé pour le compte Administrateur de SQL Server.
   - Passez en revue les termes et conditions, puis cochez la case **J’accepte les termes et conditions mentionnés ci-dessus**.
   - Sélectionnez le bouton **Acheter**.

<!-- markdownlint-enable MD033 -->

Le déploiement prend quelques minutes.

### <a name="deploy-web-app-chatbot"></a>Déployer le bot conversationnel d’application web

Pour créer le bot conversationnel, utilisez l’interface de ligne de commande Azure. L’exemple suivant illustre l’installation de l’extension CLI pour Bot Service, la création d’un groupe de ressources, puis le déploiement d’un bot qui utilise Application Insights. Lorsque vous y êtes invité, authentifiez votre compte Microsoft et laissez le bot s’enregistrer auprès de Bot Service et de l’application LUIS.

```azurecli-interactive
# Install the Azure CLI extension for the Bot Service
az extension add --name botservice --yes

# Create a resource group
az group create --name myCommerceChatbot --location eastus

# Create a Web App Chatbot that uses Application Insights
az bot create \
    --resource-group myCommerceChatbot \
    --name commerceChatbot \
    --location eastus \
    --kind webapp \
    --sku S1 \
    --insights eastus
```

### <a name="deploy-chatbot-c-application-code"></a>Déployer le code de l’application en C# du bot conversationnel

Un exemple d’application en C# est disponible sur GitHub :

- [Exemple de code C# de bot commercial](https://github.com/Microsoft/AzureBotServices-scenarios/tree/master/CSharp/Commerce/src)

L’exemple d’application inclut les composants d’authentification Azure Active Directory et l’intégration au composant LUIS de Cognitive Services. L’application requiert Visual Studio pour générer et déployer le scénario. Vous trouverez des informations supplémentaires sur la configuration d’AAD B2C et de l’application LUIS dans la documentation du référentiel GitHub.

## <a name="pricing"></a>Tarifs

Pour explorer le coût d’exécution de ce scénario, tous les services sont préconfigurés dans le calculateur de coûts. Pour pouvoir observer l’évolution de la tarification pour votre cas d’usage particulier, modifiez les variables appropriées en fonction du trafic que vous escomptez.

Nous proposons trois exemples de profils de coût selon le nombre de messages que devra traiter le bot conversationnel :

- [Petit][small-pricing] : < 10 000 messages traités par mois.
- [Moyen][medium-pricing] : < 500 000 messages traités par mois.
- [Grand][large-pricing] : < 10 000 000 messages traités par mois.

## <a name="related-resources"></a>Ressources associées

Pour obtenir un ensemble de didacticiels interactifs sur Azure Bot Service, consultez la [section des didacticiels][botservice-docs] de la documentation.

<!-- links -->

[aadb2c-docs]: /azure/active-directory-b2c/active-directory-b2c-overview
[aad-docs]: /azure/active-directory/
[appinsights-docs]: /azure/application-insights/app-insights-overview
[appservice-docs]: /azure/app-service/
[architecture]: ./media/architecture-commerce-chatbot.png
[autoscaling]: ../../best-practices/auto-scaling.md
[availability]: ../../checklist/availability.md
[botservice-docs]: /azure/bot-service/
[cognitive-docs]: /azure/cognitive-services/
[resiliency]: ../../resiliency/index.md
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[scalability]: ../../checklist/scalability.md
[sqlavailability-docs]: /azure/sql-database/sql-database-technical-overview#availability-capabilities
[sqldatabase-docs]: /azure/sql-database/
[sqlsecurity-docs]: /azure/sql-database/sql-database-technical-overview#advanced-security-and-compliance
[qna-maker]: /azure/cognitive-services/QnAMaker/Overview/overview
[speech-api]: /azure/cognitive-services/speech/home
[translator]: /azure/cognitive-services/translator/translator-info-overview

[small-pricing]: https://azure.com/e/dce05b6184904c50b38e1a8654f726b6
[medium-pricing]: https://azure.com/e/304d17106afc480dbc414f9726078a03
[large-pricing]: https://azure.com/e/8319dd5e5e3d4f118f9029e32a80e887