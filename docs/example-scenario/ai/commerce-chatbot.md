---
title: Bot conversationnel pour les réservations d’hôtel
titleSuffix: Azure Example Scenarios
description: Créez un bot conversationnel pour les applications de commerce avec Azure Bot Service.
author: iainfoulds
ms.date: 07/05/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
social_image_url: /azure/architecture/example-scenario/ai/media/architecture-commerce-chatbot.png
ms.openlocfilehash: 48f85e7443bcd6149c8024d20fb50816c1a4df38
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58245850"
---
# <a name="conversational-chatbot-for-hotel-reservations-on-azure"></a><span data-ttu-id="1d294-103">Bot conversationnel pour les réservations d’hôtel sur Azure</span><span class="sxs-lookup"><span data-stu-id="1d294-103">Conversational chatbot for hotel reservations on Azure</span></span>

<span data-ttu-id="1d294-104">Cet exemple de scénario s’applique aux entreprises qui souhaitent intégrer aux applications un bot conversationnel.</span><span class="sxs-lookup"><span data-stu-id="1d294-104">This example scenario is applicable to businesses that need to integrate a conversational chatbot into applications.</span></span> <span data-ttu-id="1d294-105">Dans ce scénario, un bot conversationnel en C# est utilisé pour une chaîne d’hôtels qui permet aux clients de vérifier la disponibilité des chambres et de les réserver via une application web ou mobile.</span><span class="sxs-lookup"><span data-stu-id="1d294-105">In this scenario, a C# chatbot is used for a hotel chain that allows customers to check availability and book accommodation through a web or mobile application.</span></span>

<span data-ttu-id="1d294-106">Utilisations éventuelles : permettre aux clients d’afficher la disponibilité de l’hôtel et de réserver des chambres, consulter le menu à emporter d’un restaurant et passer une commande, ou rechercher et commander des impressions de photos.</span><span class="sxs-lookup"><span data-stu-id="1d294-106">Potential uses include providing a way for customers to view hotel availability and book rooms, review a restaurant take-out menu and place a food order, or search for and order prints of photographs.</span></span> <span data-ttu-id="1d294-107">Traditionnellement, les entreprises devaient embaucher et former des conseillers pour le service client afin de répondre à ces demandes de clients, lesquels devaient patienter jusqu’à ce qu’un agent soit disponible pour leur fournir de l’aide.</span><span class="sxs-lookup"><span data-stu-id="1d294-107">Traditionally, businesses would need to hire and train customer service agents to respond to these customer requests, and customers would have to wait until a representative is available to provide assistance.</span></span>

<span data-ttu-id="1d294-108">En utilisant les services Azure tels que Bot Service et Language Understanding ou API Microsoft Speech, les sociétés peuvent aider les clients et traiter leurs commandes ou leurs réservations à l’aide de bots automatisés, évolutifs.</span><span class="sxs-lookup"><span data-stu-id="1d294-108">By using Azure services such as the Bot Service and Language Understanding or Speech API services, companies can assist customers and process orders or reservations with automated, scalable bots.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="1d294-109">Cas d’usage appropriés</span><span class="sxs-lookup"><span data-stu-id="1d294-109">Relevant use cases</span></span>

<span data-ttu-id="1d294-110">Les autres cas d’usage appropriés sont les suivants :</span><span class="sxs-lookup"><span data-stu-id="1d294-110">Other relevant use cases include:</span></span>

- <span data-ttu-id="1d294-111">La consultation du menu à emporter d’un restaurant et le passage d’une commande</span><span class="sxs-lookup"><span data-stu-id="1d294-111">Viewing a restaurant take-out menu and ordering food</span></span>
- <span data-ttu-id="1d294-112">La vérification de la disponibilité d’un hôtel et la réservation d’une chambre</span><span class="sxs-lookup"><span data-stu-id="1d294-112">Checking hotel availability and reserving a room</span></span>
- <span data-ttu-id="1d294-113">La recherche des photos disponibles et la commande des impressions</span><span class="sxs-lookup"><span data-stu-id="1d294-113">Searching available photos and ordering prints</span></span>

## <a name="architecture"></a><span data-ttu-id="1d294-114">Architecture</span><span class="sxs-lookup"><span data-stu-id="1d294-114">Architecture</span></span>

![Présentation de l’architecture des composants Azure impliqués dans un bot conversationnel][architecture]

<span data-ttu-id="1d294-116">Ce scénario présente un bot conversationnel qui fonctionne comme le concierge d’un hôtel.</span><span class="sxs-lookup"><span data-stu-id="1d294-116">This scenario covers a conversational bot that functions as a concierge for a hotel.</span></span> <span data-ttu-id="1d294-117">Les données circulent dans le scénario comme suit :</span><span class="sxs-lookup"><span data-stu-id="1d294-117">The data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="1d294-118">Le client accède au bot conversationnel avec une application web ou mobile.</span><span class="sxs-lookup"><span data-stu-id="1d294-118">The customer accesses the chatbot with a mobile or web app.</span></span>
2. <span data-ttu-id="1d294-119">L’utilisateur est authentifié à l’aide d’Azure Active Directory B2C (entreprise-client).</span><span class="sxs-lookup"><span data-stu-id="1d294-119">Using Azure Active Directory B2C (Business 2 Customer), the user is authenticated.</span></span>
3. <span data-ttu-id="1d294-120">En interagissant avec Bot Service, l’utilisateur demande des informations sur la disponibilité de l’hôtel.</span><span class="sxs-lookup"><span data-stu-id="1d294-120">Interacting with the Bot Service, the user requests information about hotel availability.</span></span>
4. <span data-ttu-id="1d294-121">Cognitive Services traite la requête en langage naturel pour comprendre la communication du client.</span><span class="sxs-lookup"><span data-stu-id="1d294-121">Cognitive Services processes the natural language request to understand the customer communication.</span></span>
5. <span data-ttu-id="1d294-122">Lorsque l’utilisateur est satisfait des résultats, le bot ajoute ou met à jour la réservation du client dans une base de données SQL Database.</span><span class="sxs-lookup"><span data-stu-id="1d294-122">After the user is happy with the results, the bot adds or updates the customer’s reservation in a SQL Database.</span></span>
6. <span data-ttu-id="1d294-123">Application Insights collecte les données de télémétrie d’exécution au cours du processus pour aider l’équipe DevOps en fournissant des informations sur les performances et l’utilisation du bot.</span><span class="sxs-lookup"><span data-stu-id="1d294-123">Application Insights gathers runtime telemetry throughout the process to help the DevOps team with bot performance and usage.</span></span>

### <a name="components"></a><span data-ttu-id="1d294-124">Composants</span><span class="sxs-lookup"><span data-stu-id="1d294-124">Components</span></span>

- <span data-ttu-id="1d294-125">[Azure Active Directory][aad-docs] est le service informatique mutualisé de Microsoft pour la gestion des annuaires et des identités.</span><span class="sxs-lookup"><span data-stu-id="1d294-125">[Azure Active Directory][aad-docs] is Microsoft’s multi-tenant cloud-based directory and identity management service.</span></span> <span data-ttu-id="1d294-126">Azure AD prend en charge un connecteur B2C, ce qui vous permet d’identifier les personnes à l’aide d’ID externes comme un compte Google, Facebook ou Microsoft.</span><span class="sxs-lookup"><span data-stu-id="1d294-126">Azure AD supports a B2C connector allowing you to identify individuals using external IDs such as Google, Facebook, or a Microsoft Account.</span></span>
- <span data-ttu-id="1d294-127">[App Service][appservice-docs] vous permet de créer et d’héberger des applications web dans le langage de programmation de votre choix sans gérer l’infrastructure.</span><span class="sxs-lookup"><span data-stu-id="1d294-127">[App Service][appservice-docs] enables you to build and host web applications in the programming language of your choice without managing infrastructure.</span></span>
- <span data-ttu-id="1d294-128">[Bot Service][botservice-docs] fournit des outils pour créer, tester, déployer et gérer des robots intelligents.</span><span class="sxs-lookup"><span data-stu-id="1d294-128">[Bot Service][botservice-docs] provides tools to build, test, deploy, and manage intelligent bots.</span></span>
- <span data-ttu-id="1d294-129">[Cognitive Services][cognitive-docs] permet d’utiliser des algorithmes intelligents pour voir, écouter, énoncer, comprendre et interpréter les besoins des utilisateurs selon des modes de communication naturels.</span><span class="sxs-lookup"><span data-stu-id="1d294-129">[Cognitive Services][cognitive-docs] lets you use intelligent algorithms to see, hear, speak, understand, and interpret your user needs through natural methods of communication.</span></span>
- <span data-ttu-id="1d294-130">[SQL Database][sqldatabase-docs] est un service de base de données cloud relationnelle entièrement géré qui fournit la compatibilité du moteur SQL Server.</span><span class="sxs-lookup"><span data-stu-id="1d294-130">[SQL Database][sqldatabase-docs] is a fully managed relational cloud database service that provides SQL Server engine compatibility.</span></span>
- <span data-ttu-id="1d294-131">[Application Insights][appinsights-docs] est un service de gestion des performances des applications (APM) extensible qui vous permet de surveiller les performances des applications, telles que votre bot conversationnel.</span><span class="sxs-lookup"><span data-stu-id="1d294-131">[Application Insights][appinsights-docs] is an extensible Application Performance Management (APM) service that lets you monitor the performance of applications, such as your chatbot.</span></span>

### <a name="alternatives"></a><span data-ttu-id="1d294-132">Autres solutions</span><span class="sxs-lookup"><span data-stu-id="1d294-132">Alternatives</span></span>

- <span data-ttu-id="1d294-133">[L’API Microsoft Speech][speech-api] peut être utilisée pour modifier la façon dont les clients interagissent avec votre bot.</span><span class="sxs-lookup"><span data-stu-id="1d294-133">[Microsoft Speech API][speech-api] can be used to change how customers interface with your bot.</span></span>
- <span data-ttu-id="1d294-134">[QnA Maker][qna-maker] peut être utilisé pour ajouter rapidement des connaissances à votre bot à partir d’un contenu semi-structuré comme un Forum Aux Questions.</span><span class="sxs-lookup"><span data-stu-id="1d294-134">[QnA Maker][qna-maker] can be used as to quickly add knowledge to your bot from semi-structured content like an FAQ.</span></span>
- <span data-ttu-id="1d294-135">[Traduction de texte Translator Text][translator] est un service auquel vous pouvez penser pour ajouter facilement la prise en charge multilingue à votre bot.</span><span class="sxs-lookup"><span data-stu-id="1d294-135">[Translator Text][translator] is a service that you might consider to easily add multi-lingual support to your bot.</span></span>

## <a name="considerations"></a><span data-ttu-id="1d294-136">Considérations</span><span class="sxs-lookup"><span data-stu-id="1d294-136">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="1d294-137">Disponibilité</span><span class="sxs-lookup"><span data-stu-id="1d294-137">Availability</span></span>

<span data-ttu-id="1d294-138">Ce scénario utilise Azure SQL Database pour stocker les réservations de clients.</span><span class="sxs-lookup"><span data-stu-id="1d294-138">This scenario uses Azure SQL Database for storing customer reservations.</span></span> <span data-ttu-id="1d294-139">SQL Database inclut des bases de données redondantes dans une zone, des groupes de basculement et la géoréplication.</span><span class="sxs-lookup"><span data-stu-id="1d294-139">SQL Database includes zone redundant databases, failover groups, and geo-replication.</span></span> <span data-ttu-id="1d294-140">Pour en savoir plus, consultez la section relative aux [fonctionnalités de disponibilité d’Azure SQL Database][sqlavailability-docs].</span><span class="sxs-lookup"><span data-stu-id="1d294-140">For more information, see [Azure SQL Database availability capabilities][sqlavailability-docs].</span></span>

<span data-ttu-id="1d294-141">Pour consulter d’autres rubriques relatives à la disponibilité, consultez la [liste de contrôle de la disponibilité][availability] dans le Centre des architectures Azure.</span><span class="sxs-lookup"><span data-stu-id="1d294-141">For other availability topics, see the [availability checklist][availability] in the Azure Architecture Center.</span></span>

### <a name="scalability"></a><span data-ttu-id="1d294-142">Extensibilité</span><span class="sxs-lookup"><span data-stu-id="1d294-142">Scalability</span></span>

<span data-ttu-id="1d294-143">Ce scénario utilise Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="1d294-143">This scenario uses Azure App Service.</span></span> <span data-ttu-id="1d294-144">Avec App Service, vous pouvez automatiquement mettre à l’échelle le nombre d’instances qui exécutent votre bot.</span><span class="sxs-lookup"><span data-stu-id="1d294-144">With App Service, you can automatically scale the number of instances that run your bot.</span></span> <span data-ttu-id="1d294-145">Cette fonctionnalité vous permet de faire face à la demande de la clientèle pour votre application web et votre bot conversationnel.</span><span class="sxs-lookup"><span data-stu-id="1d294-145">This functionality lets you keep up with customer demand for your web application and chatbot.</span></span> <span data-ttu-id="1d294-146">Pour plus d’informations sur la mise à l’échelle automatique, voir [Meilleures pratiques de mise à l’échelle automatique][autoscaling] dans le Centre des architectures Azure.</span><span class="sxs-lookup"><span data-stu-id="1d294-146">For more information on autoscale, see [Autoscaling best practices][autoscaling] in the Azure Architecture Center.</span></span>

<span data-ttu-id="1d294-147">Pour consulter d’autres rubriques relatives à l’extensibilité, consultez la [liste de contrôle de l’extensibilité][scalability] dans le Centre des architectures Azure.</span><span class="sxs-lookup"><span data-stu-id="1d294-147">For other scalability topics, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="1d294-148">Sécurité</span><span class="sxs-lookup"><span data-stu-id="1d294-148">Security</span></span>

<span data-ttu-id="1d294-149">Ce scénario utilise Azure Active Directory B2C (entreprise-client) pour authentifier les utilisateurs.</span><span class="sxs-lookup"><span data-stu-id="1d294-149">This scenario uses Azure Active Directory B2C (Business 2 Consumer) to authenticate users.</span></span> <span data-ttu-id="1d294-150">Avec AAD B2C, votre bot conversationnel ne stocke pas les informations des comptes clients sensibles ni les informations d’identification.</span><span class="sxs-lookup"><span data-stu-id="1d294-150">With AAD B2C, your chatbot doesn't store any sensitive customer account information or credentials.</span></span> <span data-ttu-id="1d294-151">Pour plus d’informations, consultez la [Vue d’ensemble d’Azure Active Directory B2C][aadb2c-docs].</span><span class="sxs-lookup"><span data-stu-id="1d294-151">For more information, see [Azure Active Directory B2C overview][aadb2c-docs].</span></span>

<span data-ttu-id="1d294-152">Les informations stockées dans Azure SQL Database sont chiffrées au repos avec un chiffrement Transparent Data Encryption (TDE).</span><span class="sxs-lookup"><span data-stu-id="1d294-152">Information stored in Azure SQL Database is encrypted at rest with transparent data encryption (TDE).</span></span> <span data-ttu-id="1d294-153">SQL Database présente également la fonctionnalité Always Encrypted qui chiffre les données durant l’interrogation et le traitement.</span><span class="sxs-lookup"><span data-stu-id="1d294-153">SQL Database also offers Always Encrypted which encrypts data during querying and processing.</span></span> <span data-ttu-id="1d294-154">Pour plus d’informations sur la sécurité de SQL Database, consultez la section relative à la [sécurité et à la conformité d’Azure SQL Database][sqlsecurity-docs].</span><span class="sxs-lookup"><span data-stu-id="1d294-154">For more information on SQL Database security, see [Azure SQL Database security and compliance][sqlsecurity-docs].</span></span>

<span data-ttu-id="1d294-155">Pour obtenir des conseils d’ordre général sur la conception de solutions sécurisées, consultez la [documentation sur la sécurité Azure][security].</span><span class="sxs-lookup"><span data-stu-id="1d294-155">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="1d294-156">Résilience</span><span class="sxs-lookup"><span data-stu-id="1d294-156">Resiliency</span></span>

<span data-ttu-id="1d294-157">Ce scénario utilise Azure SQL Database pour stocker les réservations de clients.</span><span class="sxs-lookup"><span data-stu-id="1d294-157">This scenario uses Azure SQL Database for storing customer reservations.</span></span> <span data-ttu-id="1d294-158">SQL Database inclut des bases de données redondantes dans une zone, des groupes de basculement, la géoréplication et des sauvegardes automatiques.</span><span class="sxs-lookup"><span data-stu-id="1d294-158">SQL Database includes zone redundant databases, failover groups, geo-replication, and automatic backups.</span></span> <span data-ttu-id="1d294-159">Ces fonctionnalités permettent à l’application de continuer à s’exécuter en cas d’événement de maintenance ou de panne.</span><span class="sxs-lookup"><span data-stu-id="1d294-159">These features allow your application to continue running if there is a maintenance event or outage.</span></span> <span data-ttu-id="1d294-160">Pour en savoir plus, consultez la section relative aux [fonctionnalités de disponibilité d’Azure SQL Database][sqlavailability-docs].</span><span class="sxs-lookup"><span data-stu-id="1d294-160">For more information, see [Azure SQL Database availability capabilities][sqlavailability-docs].</span></span>

<span data-ttu-id="1d294-161">Pour surveiller l’intégrité de votre application, ce scénario utilise Application Insights.</span><span class="sxs-lookup"><span data-stu-id="1d294-161">To monitor the health of your application, this scenario uses Application Insights.</span></span> <span data-ttu-id="1d294-162">Avec Application Insights, vous pouvez générer des alertes et résoudre les problèmes de performances qui auraient un impact sur l’expérience client et la disponibilité du bot conversationnel.</span><span class="sxs-lookup"><span data-stu-id="1d294-162">With Application Insights, you can generate alerts and respond to performance issues that would impact the customer experience and availability of the chatbot.</span></span> <span data-ttu-id="1d294-163">Pour plus d’informations, consultez l’article [Présentation d’Application Insights][appinsights-docs].</span><span class="sxs-lookup"><span data-stu-id="1d294-163">For more information, see [What is Application Insights?][appinsights-docs]</span></span>

<span data-ttu-id="1d294-164">Pour obtenir des conseils d’ordre général sur la conception de solutions résilientes, consultez l’article [Conception d’applications résilientes pour Azure][resiliency].</span><span class="sxs-lookup"><span data-stu-id="1d294-164">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="1d294-165">Déployez le scénario</span><span class="sxs-lookup"><span data-stu-id="1d294-165">Deploy the scenario</span></span>

<span data-ttu-id="1d294-166">Ce scénario est divisé en trois composants pour vous permettre d’explorer les zones sur lesquelles vous êtes le plus axé :</span><span class="sxs-lookup"><span data-stu-id="1d294-166">This scenario is divided into three components for you to explore areas that you are most focused on:</span></span>

- <span data-ttu-id="1d294-167">[Composants d’infrastructure](#deploy-infrastructure-components).</span><span class="sxs-lookup"><span data-stu-id="1d294-167">[Infrastructure components](#deploy-infrastructure-components).</span></span> <span data-ttu-id="1d294-168">Utilisez un modèle Azure Resource Manager pour déployer les composants d’infrastructure de base d’un service App Service, de Web App, d’Application Insights, d’un compte de stockage, de SQL Server et d’une base de données.</span><span class="sxs-lookup"><span data-stu-id="1d294-168">Use an Azure Resource Manger template to deploy the core infrastructure components of an App Service, Web App, Application Insights, Storage account, and SQL Server and database.</span></span>
- <span data-ttu-id="1d294-169">[Bot conversationnel d’application web](#deploy-web-app-chatbot).</span><span class="sxs-lookup"><span data-stu-id="1d294-169">[Web App Chatbot](#deploy-web-app-chatbot).</span></span> <span data-ttu-id="1d294-170">Utilisez l’interface de ligne de commande Azure pour déployer un bot avec Bot Service et l’application Language Understanding Intelligent Service (LUIS).</span><span class="sxs-lookup"><span data-stu-id="1d294-170">Use the Azure CLI to deploy a bot with the Bot Service and Language Understanding and Intelligent Services (LUIS) app.</span></span>
- <span data-ttu-id="1d294-171">[Exemple d’application de bot conversationnel en C#](#deploy-chatbot-c-application-code).</span><span class="sxs-lookup"><span data-stu-id="1d294-171">[Sample C# chatbot application](#deploy-chatbot-c-application-code).</span></span> <span data-ttu-id="1d294-172">Utilisez Visual Studio pour vérifier le code de l’exemple d’application en C# de réservation d’hôtel et le déployer vers un bot dans Azure.</span><span class="sxs-lookup"><span data-stu-id="1d294-172">Use Visual Studio to review the sample hotel reservation C# application code and deploy to a bot in Azure.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="1d294-173">Prérequis</span><span class="sxs-lookup"><span data-stu-id="1d294-173">Prerequisites</span></span>

<span data-ttu-id="1d294-174">Vous devez disposer d’un compte Azure existant.</span><span class="sxs-lookup"><span data-stu-id="1d294-174">You must have an existing Azure account.</span></span> <span data-ttu-id="1d294-175">Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.</span><span class="sxs-lookup"><span data-stu-id="1d294-175">If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.</span></span>

### <a name="walk-through"></a><span data-ttu-id="1d294-176">Procédure pas à pas</span><span class="sxs-lookup"><span data-stu-id="1d294-176">Walk-through</span></span>

<span data-ttu-id="1d294-177">Pour déployer les composants d’infrastructure avec un modèle Resource Manager, procédez comme suit.</span><span class="sxs-lookup"><span data-stu-id="1d294-177">To deploy the infrastructure components with a Resource Manager template, perform the following steps.</span></span>

<!-- markdownlint-disable MD033 -->

1. <span data-ttu-id="1d294-178">Cliquez sur le bouton **Déployer sur Azure** :</span><span class="sxs-lookup"><span data-stu-id="1d294-178">Click the **Deploy to Azure** button:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fcommerce-chatbot.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="1d294-179">Attendez l’ouverture de la solution Template deployment dans le portail Azure, puis procédez comme suit :</span><span class="sxs-lookup"><span data-stu-id="1d294-179">Wait for the template deployment to open in the Azure portal, then complete the following steps:</span></span>
   - <span data-ttu-id="1d294-180">Sélectionnez **Créer un nouveau groupe de ressources**, puis indiquez un nom, par exemple *myCommerceChatBotInfrastructure* dans la zone de texte.</span><span class="sxs-lookup"><span data-stu-id="1d294-180">Choose to **Create new** resource group, then provide a name such as *myCommerceChatBotInfrastructure* in the text box.</span></span>
   - <span data-ttu-id="1d294-181">Dans la zone de liste déroulante **Emplacement**, sélectionnez une région.</span><span class="sxs-lookup"><span data-stu-id="1d294-181">Select a region from the **Location** drop-down box.</span></span>
   - <span data-ttu-id="1d294-182">Indiquez un nom d’utilisateur et un mot de passe sécurisé pour le compte Administrateur de SQL Server.</span><span class="sxs-lookup"><span data-stu-id="1d294-182">Provide a username and secure password for the SQL Server administrator account.</span></span>
   - <span data-ttu-id="1d294-183">Passez en revue les termes et conditions, puis cochez la case **J’accepte les termes et conditions mentionnés ci-dessus**.</span><span class="sxs-lookup"><span data-stu-id="1d294-183">Review the terms and conditions, then check **I agree to the terms and conditions stated above**.</span></span>
   - <span data-ttu-id="1d294-184">Sélectionnez le bouton **Acheter**.</span><span class="sxs-lookup"><span data-stu-id="1d294-184">Select the **Purchase** button.</span></span>

<!-- markdownlint-enable MD033 -->

<span data-ttu-id="1d294-185">Le déploiement prend quelques minutes.</span><span class="sxs-lookup"><span data-stu-id="1d294-185">It takes a few minutes for the deployment to complete.</span></span>

### <a name="deploy-web-app-chatbot"></a><span data-ttu-id="1d294-186">Déployer le bot conversationnel d’application web</span><span class="sxs-lookup"><span data-stu-id="1d294-186">Deploy Web App chatbot</span></span>

<span data-ttu-id="1d294-187">Pour créer le bot conversationnel, utilisez l’interface de ligne de commande Azure.</span><span class="sxs-lookup"><span data-stu-id="1d294-187">To create the chatbot, use the Azure CLI.</span></span> <span data-ttu-id="1d294-188">L’exemple suivant illustre l’installation de l’extension CLI pour Bot Service, la création d’un groupe de ressources, puis le déploiement d’un bot qui utilise Application Insights.</span><span class="sxs-lookup"><span data-stu-id="1d294-188">The following example installs the CLI extension for Bot Service, creates a resource group, then deploys a bot that uses Application Insights.</span></span> <span data-ttu-id="1d294-189">Lorsque vous y êtes invité, authentifiez votre compte Microsoft et laissez le bot s’enregistrer auprès de Bot Service et de l’application LUIS.</span><span class="sxs-lookup"><span data-stu-id="1d294-189">When prompted, authenticate your Microsoft account and allow the bot to register itself with the Bot Service and Language Understanding and Intelligent Services (LUIS) app.</span></span>

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

### <a name="deploy-chatbot-c-application-code"></a><span data-ttu-id="1d294-190">Déployer le code de l’application en C# du bot conversationnel</span><span class="sxs-lookup"><span data-stu-id="1d294-190">Deploy chatbot C# application code</span></span>

<span data-ttu-id="1d294-191">Un exemple d’application en C# est disponible sur GitHub :</span><span class="sxs-lookup"><span data-stu-id="1d294-191">A sample C# application is available on GitHub:</span></span>

- [<span data-ttu-id="1d294-192">Exemple de code C# de bot commercial</span><span class="sxs-lookup"><span data-stu-id="1d294-192">Commerce Bot C# sample</span></span>](https://github.com/Microsoft/AzureBotServices-scenarios/tree/master/CSharp/Commerce/src)

<span data-ttu-id="1d294-193">L’exemple d’application inclut les composants d’authentification Azure Active Directory et l’intégration au composant LUIS de Cognitive Services.</span><span class="sxs-lookup"><span data-stu-id="1d294-193">The sample application includes the Azure Active Directory authentication components and integration with the Language Understanding and Intelligent Services (LUIS) component of Cognitive Services.</span></span> <span data-ttu-id="1d294-194">L’application requiert Visual Studio pour générer et déployer le scénario.</span><span class="sxs-lookup"><span data-stu-id="1d294-194">The application requires Visual Studio to build and deploy the scenario.</span></span> <span data-ttu-id="1d294-195">Vous trouverez des informations supplémentaires sur la configuration d’AAD B2C et de l’application LUIS dans la documentation du référentiel GitHub.</span><span class="sxs-lookup"><span data-stu-id="1d294-195">Additional information on configuring AAD B2C and the LUIS app can be found in the GitHub repo documentation.</span></span>

## <a name="pricing"></a><span data-ttu-id="1d294-196">Tarifs</span><span class="sxs-lookup"><span data-stu-id="1d294-196">Pricing</span></span>

<span data-ttu-id="1d294-197">Pour explorer le coût d’exécution de ce scénario, tous les services sont préconfigurés dans le calculateur de coûts.</span><span class="sxs-lookup"><span data-stu-id="1d294-197">To explore the cost of running this scenario, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="1d294-198">Pour pouvoir observer l’évolution de la tarification pour votre cas d’usage particulier, modifiez les variables appropriées en fonction du trafic que vous escomptez.</span><span class="sxs-lookup"><span data-stu-id="1d294-198">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="1d294-199">Nous proposons trois exemples de profils de coût selon le nombre de messages que devra traiter le bot conversationnel :</span><span class="sxs-lookup"><span data-stu-id="1d294-199">We have provided three sample cost profiles based on the number of messages you expect your chatbot to process:</span></span>

- <span data-ttu-id="1d294-200">[Petit][small-pricing] : < 10 000 messages traités par mois.</span><span class="sxs-lookup"><span data-stu-id="1d294-200">[Small][small-pricing]: this pricing example correlates to processing < 10,000 messages per month.</span></span>
- <span data-ttu-id="1d294-201">[Moyen][medium-pricing] : < 500 000 messages traités par mois.</span><span class="sxs-lookup"><span data-stu-id="1d294-201">[Medium][medium-pricing]: this pricing example correlates to processing < 500,000 messages per month.</span></span>
- <span data-ttu-id="1d294-202">[Grand][large-pricing] : < 10 000 000 messages traités par mois.</span><span class="sxs-lookup"><span data-stu-id="1d294-202">[Large][large-pricing]: this pricing example correlates to processing < 10 million messages per month.</span></span>

## <a name="related-resources"></a><span data-ttu-id="1d294-203">Ressources associées</span><span class="sxs-lookup"><span data-stu-id="1d294-203">Related resources</span></span>

<span data-ttu-id="1d294-204">Pour obtenir un ensemble de didacticiels interactifs sur Azure Bot Service, consultez la [section des didacticiels][botservice-docs] de la documentation.</span><span class="sxs-lookup"><span data-stu-id="1d294-204">For a set of guided tutorials for the Azure Bot Service, see the [tutorial section][botservice-docs] of the documentation.</span></span>

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