---
title: Guide spécifique relatif au service de nouvelle tentative
titleSuffix: Best practices for cloud applications
description: Guide spécifique relatif au service pour définir le mécanisme de nouvelle tentative.
author: dragon119
ms.date: 08/13/2018
ms.topic: best-practice
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 170a38f6b8a6c107670561e63f236e43af948d7d
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59641040"
---
# <a name="retry-guidance-for-specific-services"></a><span data-ttu-id="01d51-103">Guide du mécanisme de nouvelle tentative relatif aux différents services</span><span class="sxs-lookup"><span data-stu-id="01d51-103">Retry guidance for specific services</span></span>

<span data-ttu-id="01d51-104">La plupart des services Azure et des kits de développement logiciel (SDK) client incluent un mécanisme de nouvelle tentative.</span><span class="sxs-lookup"><span data-stu-id="01d51-104">Most Azure services and client SDKs include a retry mechanism.</span></span> <span data-ttu-id="01d51-105">Cependant, ceux-ci diffèrent, car chaque service a des exigences et des caractéristiques différentes, et donc chaque mécanisme de nouvelle tentative est réglé sur un service spécifique.</span><span class="sxs-lookup"><span data-stu-id="01d51-105">However, these differ because each service has different characteristics and requirements, and so each retry mechanism is tuned to a specific service.</span></span> <span data-ttu-id="01d51-106">Ce guide résume les fonctionnalités de mécanisme de nouvelle tentative pour la majorité des services Azure et inclut des informations pour vous aider à utiliser, adapter ou étendre le mécanisme de nouvelle tentative pour ce service.</span><span class="sxs-lookup"><span data-stu-id="01d51-106">This guide summarizes the retry mechanism features for the majority of Azure services, and includes information to help you use, adapt, or extend the retry mechanism for that service.</span></span>

<span data-ttu-id="01d51-107">Pour obtenir des instructions générales sur la gestion des erreurs temporaires, des nouvelles tentatives de connexion et des opérations en fonction des services et des ressources, consultez [Guide sur les nouvelles tentatives](./transient-faults.md).</span><span class="sxs-lookup"><span data-stu-id="01d51-107">For general guidance on handling transient faults, and retrying connections and operations against services and resources, see [Retry guidance](./transient-faults.md).</span></span>

<span data-ttu-id="01d51-108">Le tableau suivant récapitule les fonctionnalités de nouvelle tentative pour les services Azure décrits dans ce guide.</span><span class="sxs-lookup"><span data-stu-id="01d51-108">The following table summarizes the retry features for the Azure services described in this guidance.</span></span>

| <span data-ttu-id="01d51-109">**Service**</span><span class="sxs-lookup"><span data-stu-id="01d51-109">**Service**</span></span> | <span data-ttu-id="01d51-110">**Fonctionnalités de nouvelle tentative**</span><span class="sxs-lookup"><span data-stu-id="01d51-110">**Retry capabilities**</span></span> | <span data-ttu-id="01d51-111">**Configuration de la stratégie**</span><span class="sxs-lookup"><span data-stu-id="01d51-111">**Policy configuration**</span></span> | <span data-ttu-id="01d51-112">**Portée**</span><span class="sxs-lookup"><span data-stu-id="01d51-112">**Scope**</span></span> | <span data-ttu-id="01d51-113">**Fonctionnalités de télémétrie**</span><span class="sxs-lookup"><span data-stu-id="01d51-113">**Telemetry features**</span></span> |
| --- | --- | --- | --- | --- |
| <span data-ttu-id="01d51-114">**[Azure Active Directory](#azure-active-directory)**</span><span class="sxs-lookup"><span data-stu-id="01d51-114">**[Azure Active Directory](#azure-active-directory)**</span></span> |<span data-ttu-id="01d51-115">Native dans la bibliothèque ADAL</span><span class="sxs-lookup"><span data-stu-id="01d51-115">Native in ADAL library</span></span> |<span data-ttu-id="01d51-116">Incorporé dans la bibliothèque ADAL</span><span class="sxs-lookup"><span data-stu-id="01d51-116">Embedded into ADAL library</span></span> |<span data-ttu-id="01d51-117">Interne</span><span class="sxs-lookup"><span data-stu-id="01d51-117">Internal</span></span> |<span data-ttu-id="01d51-118">Aucun</span><span class="sxs-lookup"><span data-stu-id="01d51-118">None</span></span> |
| <span data-ttu-id="01d51-119">**[Cosmos DB](#cosmos-db)**</span><span class="sxs-lookup"><span data-stu-id="01d51-119">**[Cosmos DB](#cosmos-db)**</span></span> |<span data-ttu-id="01d51-120">Native dans le service</span><span class="sxs-lookup"><span data-stu-id="01d51-120">Native in service</span></span> |<span data-ttu-id="01d51-121">Non configurable</span><span class="sxs-lookup"><span data-stu-id="01d51-121">Non-configurable</span></span> |<span data-ttu-id="01d51-122">Globale</span><span class="sxs-lookup"><span data-stu-id="01d51-122">Global</span></span> |<span data-ttu-id="01d51-123">TraceSource</span><span class="sxs-lookup"><span data-stu-id="01d51-123">TraceSource</span></span> |
| <span data-ttu-id="01d51-124">**Data Lake Store**</span><span class="sxs-lookup"><span data-stu-id="01d51-124">**Data Lake Store**</span></span> |<span data-ttu-id="01d51-125">Native dans le client</span><span class="sxs-lookup"><span data-stu-id="01d51-125">Native in client</span></span> |<span data-ttu-id="01d51-126">Non configurable</span><span class="sxs-lookup"><span data-stu-id="01d51-126">Non-configurable</span></span> |<span data-ttu-id="01d51-127">Opérations individuelles</span><span class="sxs-lookup"><span data-stu-id="01d51-127">Individual operations</span></span> |<span data-ttu-id="01d51-128">Aucun</span><span class="sxs-lookup"><span data-stu-id="01d51-128">None</span></span> |
| <span data-ttu-id="01d51-129">**[Event Hubs](#event-hubs)**</span><span class="sxs-lookup"><span data-stu-id="01d51-129">**[Event Hubs](#event-hubs)**</span></span> |<span data-ttu-id="01d51-130">Native dans le client</span><span class="sxs-lookup"><span data-stu-id="01d51-130">Native in client</span></span> |<span data-ttu-id="01d51-131">Par programme</span><span class="sxs-lookup"><span data-stu-id="01d51-131">Programmatic</span></span> |<span data-ttu-id="01d51-132">Client</span><span class="sxs-lookup"><span data-stu-id="01d51-132">Client</span></span> |<span data-ttu-id="01d51-133">Aucun</span><span class="sxs-lookup"><span data-stu-id="01d51-133">None</span></span> |
| <span data-ttu-id="01d51-134">**[IoT Hub](#iot-hub)**</span><span class="sxs-lookup"><span data-stu-id="01d51-134">**[IoT Hub](#iot-hub)**</span></span> |<span data-ttu-id="01d51-135">Natif dans le SDK client</span><span class="sxs-lookup"><span data-stu-id="01d51-135">Native in client SDK</span></span> |<span data-ttu-id="01d51-136">Par programme</span><span class="sxs-lookup"><span data-stu-id="01d51-136">Programmatic</span></span> |<span data-ttu-id="01d51-137">Client</span><span class="sxs-lookup"><span data-stu-id="01d51-137">Client</span></span> |<span data-ttu-id="01d51-138">Aucun</span><span class="sxs-lookup"><span data-stu-id="01d51-138">None</span></span> |
| <span data-ttu-id="01d51-139">**[Cache Redis](#azure-redis-cache)**</span><span class="sxs-lookup"><span data-stu-id="01d51-139">**[Redis Cache](#azure-redis-cache)**</span></span> |<span data-ttu-id="01d51-140">Native dans le client</span><span class="sxs-lookup"><span data-stu-id="01d51-140">Native in client</span></span> |<span data-ttu-id="01d51-141">Par programme</span><span class="sxs-lookup"><span data-stu-id="01d51-141">Programmatic</span></span> |<span data-ttu-id="01d51-142">Client</span><span class="sxs-lookup"><span data-stu-id="01d51-142">Client</span></span> |<span data-ttu-id="01d51-143">TextWriter</span><span class="sxs-lookup"><span data-stu-id="01d51-143">TextWriter</span></span> |
| <span data-ttu-id="01d51-144">**[Recherche](#azure-search)**</span><span class="sxs-lookup"><span data-stu-id="01d51-144">**[Search](#azure-search)**</span></span> |<span data-ttu-id="01d51-145">Native dans le client</span><span class="sxs-lookup"><span data-stu-id="01d51-145">Native in client</span></span> |<span data-ttu-id="01d51-146">Par programme</span><span class="sxs-lookup"><span data-stu-id="01d51-146">Programmatic</span></span> |<span data-ttu-id="01d51-147">Client</span><span class="sxs-lookup"><span data-stu-id="01d51-147">Client</span></span> |<span data-ttu-id="01d51-148">ETW ou personnalisé</span><span class="sxs-lookup"><span data-stu-id="01d51-148">ETW or Custom</span></span> |
| <span data-ttu-id="01d51-149">**[Service Bus](#service-bus)**</span><span class="sxs-lookup"><span data-stu-id="01d51-149">**[Service Bus](#service-bus)**</span></span> |<span data-ttu-id="01d51-150">Native dans le client</span><span class="sxs-lookup"><span data-stu-id="01d51-150">Native in client</span></span> |<span data-ttu-id="01d51-151">Par programme</span><span class="sxs-lookup"><span data-stu-id="01d51-151">Programmatic</span></span> |<span data-ttu-id="01d51-152">Gestionnaire d’espace de noms, fabrique de messagerie et client</span><span class="sxs-lookup"><span data-stu-id="01d51-152">Namespace Manager, Messaging Factory, and Client</span></span> |<span data-ttu-id="01d51-153">ETW</span><span class="sxs-lookup"><span data-stu-id="01d51-153">ETW</span></span> |
| <span data-ttu-id="01d51-154">**[Service Fabric](#service-fabric)**</span><span class="sxs-lookup"><span data-stu-id="01d51-154">**[Service Fabric](#service-fabric)**</span></span> |<span data-ttu-id="01d51-155">Native dans le client</span><span class="sxs-lookup"><span data-stu-id="01d51-155">Native in client</span></span> |<span data-ttu-id="01d51-156">Par programme</span><span class="sxs-lookup"><span data-stu-id="01d51-156">Programmatic</span></span> |<span data-ttu-id="01d51-157">Client</span><span class="sxs-lookup"><span data-stu-id="01d51-157">Client</span></span> |<span data-ttu-id="01d51-158">Aucun</span><span class="sxs-lookup"><span data-stu-id="01d51-158">None</span></span> |
| <span data-ttu-id="01d51-159">**[Base de données SQL avec ADO.NET](#sql-database-using-adonet)**</span><span class="sxs-lookup"><span data-stu-id="01d51-159">**[SQL Database with ADO.NET](#sql-database-using-adonet)**</span></span> |[<span data-ttu-id="01d51-160">Polly</span><span class="sxs-lookup"><span data-stu-id="01d51-160">Polly</span></span>](#transient-fault-handling-with-polly) |<span data-ttu-id="01d51-161">Déclarative et par programme</span><span class="sxs-lookup"><span data-stu-id="01d51-161">Declarative and programmatic</span></span> |<span data-ttu-id="01d51-162">Instructions uniques ou blocs de code</span><span class="sxs-lookup"><span data-stu-id="01d51-162">Single statements or blocks of code</span></span> |<span data-ttu-id="01d51-163">Personnalisée</span><span class="sxs-lookup"><span data-stu-id="01d51-163">Custom</span></span> |
| <span data-ttu-id="01d51-164">**[Base de données SQL avec Entity Framework](#sql-database-using-entity-framework-6)**</span><span class="sxs-lookup"><span data-stu-id="01d51-164">**[SQL Database with Entity Framework](#sql-database-using-entity-framework-6)**</span></span> |<span data-ttu-id="01d51-165">Native dans le client</span><span class="sxs-lookup"><span data-stu-id="01d51-165">Native in client</span></span> |<span data-ttu-id="01d51-166">Par programme</span><span class="sxs-lookup"><span data-stu-id="01d51-166">Programmatic</span></span> |<span data-ttu-id="01d51-167">Globale par domaine d’application</span><span class="sxs-lookup"><span data-stu-id="01d51-167">Global per AppDomain</span></span> |<span data-ttu-id="01d51-168">Aucun</span><span class="sxs-lookup"><span data-stu-id="01d51-168">None</span></span> |
| <span data-ttu-id="01d51-169">**[SQL Database avec Entity Framework Core](#sql-database-using-entity-framework-core)**</span><span class="sxs-lookup"><span data-stu-id="01d51-169">**[SQL Database with Entity Framework Core](#sql-database-using-entity-framework-core)**</span></span> |<span data-ttu-id="01d51-170">Native dans le client</span><span class="sxs-lookup"><span data-stu-id="01d51-170">Native in client</span></span> |<span data-ttu-id="01d51-171">Par programme</span><span class="sxs-lookup"><span data-stu-id="01d51-171">Programmatic</span></span> |<span data-ttu-id="01d51-172">Globale par domaine d’application</span><span class="sxs-lookup"><span data-stu-id="01d51-172">Global per AppDomain</span></span> |<span data-ttu-id="01d51-173">Aucun</span><span class="sxs-lookup"><span data-stu-id="01d51-173">None</span></span> |
| <span data-ttu-id="01d51-174">**[Stockage](#azure-storage)**</span><span class="sxs-lookup"><span data-stu-id="01d51-174">**[Storage](#azure-storage)**</span></span> |<span data-ttu-id="01d51-175">Native dans le client</span><span class="sxs-lookup"><span data-stu-id="01d51-175">Native in client</span></span> |<span data-ttu-id="01d51-176">Par programme</span><span class="sxs-lookup"><span data-stu-id="01d51-176">Programmatic</span></span> |<span data-ttu-id="01d51-177">Opérations individuelles et du client</span><span class="sxs-lookup"><span data-stu-id="01d51-177">Client and individual operations</span></span> |<span data-ttu-id="01d51-178">TraceSource</span><span class="sxs-lookup"><span data-stu-id="01d51-178">TraceSource</span></span> |

> [!NOTE]
> <span data-ttu-id="01d51-179">Pour la plupart des mécanismes de nouvelle tentative intégrés Azure, il n’existe actuellement aucune manière d’appliquer une autre stratégie de nouvelle tentative selon le type d’erreur ou d’exception.</span><span class="sxs-lookup"><span data-stu-id="01d51-179">For most of the Azure built-in retry mechanisms, there is currently no way apply a different retry policy for different types of error or exception.</span></span> <span data-ttu-id="01d51-180">Vous devez configurer une stratégie qui fournit les performances et la disponibilité moyennes optimales.</span><span class="sxs-lookup"><span data-stu-id="01d51-180">You should configure a policy that provides the optimum average performance and availability.</span></span> <span data-ttu-id="01d51-181">Une façon d’ajuster la stratégie consiste à analyser les fichiers journaux pour déterminer le type d’erreurs temporaires qui se produisent.</span><span class="sxs-lookup"><span data-stu-id="01d51-181">One way to fine-tune the policy is to analyze log files to determine the type of transient faults that are occurring.</span></span>

<!-- markdownlint-disable MD024 MD033 -->

## <a name="azure-active-directory"></a><span data-ttu-id="01d51-182">Azure Active Directory</span><span class="sxs-lookup"><span data-stu-id="01d51-182">Azure Active Directory</span></span>

<span data-ttu-id="01d51-183">Azure Active Directory (AD) est une solution cloud complète de gestion des accès et des identités qui combine des services d’annuaire essentiels, une gouvernance avancée des identités, des services de sécurité et une gestion des accès aux applications.</span><span class="sxs-lookup"><span data-stu-id="01d51-183">Azure Active Directory (Azure AD) is a comprehensive identity and access management cloud solution that combines core directory services, advanced identity governance, security, and application access management.</span></span> <span data-ttu-id="01d51-184">Azure AD offre également aux développeurs une plate-forme de gestion des identités pour permettre un contrôle d’accès à leurs applications, en fonction d’une stratégie et de règles centralisés.</span><span class="sxs-lookup"><span data-stu-id="01d51-184">Azure AD also offers developers an identity management platform to deliver access control to their applications, based on centralized policy and rules.</span></span>

> [!NOTE]
> <span data-ttu-id="01d51-185">Pour obtenir des conseils de nouvelle tentative sur les points de terminaison Managed Service Identity, consultez [Utilisation d’une identité du service administré (MSI) d’une machine virtuelle Azure pour obtenir des jetons](/azure/active-directory/managed-service-identity/how-to-use-vm-token#error-handling).</span><span class="sxs-lookup"><span data-stu-id="01d51-185">For retry guidance on Managed Service Identity endpoints, see [How to use an Azure VM Managed Service Identity (MSI) for token acquisition](/azure/active-directory/managed-service-identity/how-to-use-vm-token#error-handling).</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="01d51-186">Mécanisme de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="01d51-186">Retry mechanism</span></span>

<span data-ttu-id="01d51-187">La bibliothèque d’authentification d’Active Directory (ADAL) intègre un mécanisme de nouvelle tentative pour Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="01d51-187">There is a built-in retry mechanism for Azure Active Directory in the Active Directory Authentication Library (ADAL).</span></span> <span data-ttu-id="01d51-188">Pour éviter les verrouillages imprévus, il est recommandé que les bibliothèques et codes d’application tiers ne procèdent **pas** à de nouvelles tentatives pour les connexions ayant échoué, mais autorisent la bibliothèque ADAL à gérer les nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="01d51-188">To avoid unexpected lockouts, we recommend that third party libraries and application code do **not** retry failed connections, but allow ADAL to handle retries.</span></span>

### <a name="retry-usage-guidance"></a><span data-ttu-id="01d51-189">Guide d’utilisation des nouvelles tentatives</span><span class="sxs-lookup"><span data-stu-id="01d51-189">Retry usage guidance</span></span>

<span data-ttu-id="01d51-190">Respectez les consignes suivantes lors de l’utilisation d’Azure Active Directory :</span><span class="sxs-lookup"><span data-stu-id="01d51-190">Consider the following guidelines when using Azure Active Directory:</span></span>

- <span data-ttu-id="01d51-191">Dans la mesure du possible, utilisez la bibliothèque ADAL et la prise en charge intégrée des nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="01d51-191">When possible, use the ADAL library and the built-in support for retries.</span></span>
- <span data-ttu-id="01d51-192">Si vous utilisez l’API REST pour Azure Active Directory, réessayez l’opération si le code de résultat est 429 (trop de demandes) ou si vous rencontrez une erreur dans la plage 5xx.</span><span class="sxs-lookup"><span data-stu-id="01d51-192">If you are using the REST API for Azure Active Directory, retry the operation if the result code is 429 (Too Many Requests) or an error in the 5xx range.</span></span> <span data-ttu-id="01d51-193">N’effectuez pas de nouvelle tentative pour toute autre erreur.</span><span class="sxs-lookup"><span data-stu-id="01d51-193">Do not retry for any other errors.</span></span>
- <span data-ttu-id="01d51-194">Une stratégie de temporisation exponentielle est recommandée pour une utilisation dans des scénarios de traitement par lots avec Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="01d51-194">An exponential back-off policy is recommended for use in batch scenarios with Azure Active Directory.</span></span>

<span data-ttu-id="01d51-195">Pensez à commencer par les paramètres ci-après pour les opérations liées aux nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="01d51-195">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="01d51-196">Il s’agit de paramètres généraux. Vous devez par conséquent surveiller les opérations et optimiser les valeurs en fonction de votre propre scénario.</span><span class="sxs-lookup"><span data-stu-id="01d51-196">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="01d51-197">**Contexte**</span><span class="sxs-lookup"><span data-stu-id="01d51-197">**Context**</span></span> | <span data-ttu-id="01d51-198">**Exemple de cible E2E<br />latence max**</span><span class="sxs-lookup"><span data-stu-id="01d51-198">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="01d51-199">**Stratégie de nouvelle tentative**</span><span class="sxs-lookup"><span data-stu-id="01d51-199">**Retry strategy**</span></span> | <span data-ttu-id="01d51-200">**Paramètres**</span><span class="sxs-lookup"><span data-stu-id="01d51-200">**Settings**</span></span> | <span data-ttu-id="01d51-201">**Valeurs**</span><span class="sxs-lookup"><span data-stu-id="01d51-201">**Values**</span></span> | <span data-ttu-id="01d51-202">**Fonctionnement**</span><span class="sxs-lookup"><span data-stu-id="01d51-202">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="01d51-203">Interactif, interface utilisateur,</span><span class="sxs-lookup"><span data-stu-id="01d51-203">Interactive, UI,</span></span><br /><span data-ttu-id="01d51-204">ou premier plan</span><span class="sxs-lookup"><span data-stu-id="01d51-204">or foreground</span></span> |<span data-ttu-id="01d51-205">2 secondes</span><span class="sxs-lookup"><span data-stu-id="01d51-205">2 sec</span></span> |<span data-ttu-id="01d51-206">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="01d51-206">FixedInterval</span></span> |<span data-ttu-id="01d51-207">Nombre de tentatives</span><span class="sxs-lookup"><span data-stu-id="01d51-207">Retry count</span></span><br /><span data-ttu-id="01d51-208">Intervalle avant nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="01d51-208">Retry interval</span></span><br /><span data-ttu-id="01d51-209">Première nouvelle tentative rapide</span><span class="sxs-lookup"><span data-stu-id="01d51-209">First fast retry</span></span> |<span data-ttu-id="01d51-210">3</span><span class="sxs-lookup"><span data-stu-id="01d51-210">3</span></span><br /><span data-ttu-id="01d51-211">500 ms</span><span class="sxs-lookup"><span data-stu-id="01d51-211">500 ms</span></span><br /><span data-ttu-id="01d51-212">true</span><span class="sxs-lookup"><span data-stu-id="01d51-212">true</span></span> |<span data-ttu-id="01d51-213">Tentative 1 - délai 0 s</span><span class="sxs-lookup"><span data-stu-id="01d51-213">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="01d51-214">Tentative 2 - délai 500 ms</span><span class="sxs-lookup"><span data-stu-id="01d51-214">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="01d51-215">Tentative 3 - délai 500 ms</span><span class="sxs-lookup"><span data-stu-id="01d51-215">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="01d51-216">Arrière-plan ou </span><span class="sxs-lookup"><span data-stu-id="01d51-216">Background or</span></span><br /><span data-ttu-id="01d51-217">lot</span><span class="sxs-lookup"><span data-stu-id="01d51-217">batch</span></span> |<span data-ttu-id="01d51-218">60 secondes</span><span class="sxs-lookup"><span data-stu-id="01d51-218">60 sec</span></span> |<span data-ttu-id="01d51-219">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="01d51-219">ExponentialBackoff</span></span> |<span data-ttu-id="01d51-220">Nombre de tentatives</span><span class="sxs-lookup"><span data-stu-id="01d51-220">Retry count</span></span><br /><span data-ttu-id="01d51-221">Temporisation min</span><span class="sxs-lookup"><span data-stu-id="01d51-221">Min back-off</span></span><br /><span data-ttu-id="01d51-222">Temporisation max</span><span class="sxs-lookup"><span data-stu-id="01d51-222">Max back-off</span></span><br /><span data-ttu-id="01d51-223">Temporisation delta</span><span class="sxs-lookup"><span data-stu-id="01d51-223">Delta back-off</span></span><br /><span data-ttu-id="01d51-224">Première nouvelle tentative rapide</span><span class="sxs-lookup"><span data-stu-id="01d51-224">First fast retry</span></span> |<span data-ttu-id="01d51-225">5.</span><span class="sxs-lookup"><span data-stu-id="01d51-225">5</span></span><br /><span data-ttu-id="01d51-226">0 seconde</span><span class="sxs-lookup"><span data-stu-id="01d51-226">0 sec</span></span><br /><span data-ttu-id="01d51-227">60 secondes</span><span class="sxs-lookup"><span data-stu-id="01d51-227">60 sec</span></span><br /><span data-ttu-id="01d51-228">2 secondes</span><span class="sxs-lookup"><span data-stu-id="01d51-228">2 sec</span></span><br /><span data-ttu-id="01d51-229">false</span><span class="sxs-lookup"><span data-stu-id="01d51-229">false</span></span> |<span data-ttu-id="01d51-230">Tentative 1 - délai 0 s</span><span class="sxs-lookup"><span data-stu-id="01d51-230">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="01d51-231">Tentative 2 - délai ~2 s</span><span class="sxs-lookup"><span data-stu-id="01d51-231">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="01d51-232">Tentative 3 - délai ~6 s</span><span class="sxs-lookup"><span data-stu-id="01d51-232">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="01d51-233">Tentative 4 - délai ~14 s</span><span class="sxs-lookup"><span data-stu-id="01d51-233">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="01d51-234">Tentative 5 - délai ~30 s</span><span class="sxs-lookup"><span data-stu-id="01d51-234">Attempt 5 - delay ~30 sec</span></span> |

### <a name="more-information"></a><span data-ttu-id="01d51-235">Plus d’informations</span><span class="sxs-lookup"><span data-stu-id="01d51-235">More information</span></span>

- <span data-ttu-id="01d51-236">[Bibliothèques d’authentification d’Azure Active Directory][adal]</span><span class="sxs-lookup"><span data-stu-id="01d51-236">[Azure Active Directory Authentication Libraries][adal]</span></span>

## <a name="cosmos-db"></a><span data-ttu-id="01d51-237">Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="01d51-237">Cosmos DB</span></span>

<span data-ttu-id="01d51-238">Cosmos DB est une base de données multimodèle entièrement gérée qui prend en charge les données JSON sans schéma.</span><span class="sxs-lookup"><span data-stu-id="01d51-238">Cosmos DB is a fully-managed multi-model database that supports schema-less JSON data.</span></span> <span data-ttu-id="01d51-239">Elle offre des performances configurables et fiables, un traitement transactionnel JavaScript natif et est conçue pour le cloud avec une mise à l’échelle élastique.</span><span class="sxs-lookup"><span data-stu-id="01d51-239">It offers configurable and reliable performance, native JavaScript transactional processing, and is built for the cloud with elastic scale.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="01d51-240">Mécanisme de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="01d51-240">Retry mechanism</span></span>

<span data-ttu-id="01d51-241">La classe `DocumentClient` retente automatiquement les tentatives ayant échoué.</span><span class="sxs-lookup"><span data-stu-id="01d51-241">The `DocumentClient` class automatically retries failed attempts.</span></span> <span data-ttu-id="01d51-242">Pour définir le nombre de nouvelles tentatives et le délai d’attente maximal, configurez [ConnectionPolicy.RetryOptions].</span><span class="sxs-lookup"><span data-stu-id="01d51-242">To set the number of retries and the maximum wait time, configure [ConnectionPolicy.RetryOptions].</span></span> <span data-ttu-id="01d51-243">Les exceptions déclenchées par le client sont au-delà de la stratégie de nouvelle tentative ou ne sont pas des erreurs temporaires.</span><span class="sxs-lookup"><span data-stu-id="01d51-243">Exceptions that the client raises are either beyond the retry policy or are not transient errors.</span></span>

<span data-ttu-id="01d51-244">Si Cosmos DB limite le client, il renvoie une erreur HTTP 429.</span><span class="sxs-lookup"><span data-stu-id="01d51-244">If Cosmos DB throttles the client, it returns an HTTP 429 error.</span></span> <span data-ttu-id="01d51-245">Vérifiez le code d’état dans `DocumentClientException`.</span><span class="sxs-lookup"><span data-stu-id="01d51-245">Check the status code in the `DocumentClientException`.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="01d51-246">Configuration de la stratégie</span><span class="sxs-lookup"><span data-stu-id="01d51-246">Policy configuration</span></span>

<span data-ttu-id="01d51-247">Le tableau suivant présente les paramètres par défaut pour la classe `RetryOptions`.</span><span class="sxs-lookup"><span data-stu-id="01d51-247">The following table shows the default settings for the `RetryOptions` class.</span></span>

| <span data-ttu-id="01d51-248">Paramètre</span><span class="sxs-lookup"><span data-stu-id="01d51-248">Setting</span></span> | <span data-ttu-id="01d51-249">Valeur par défaut</span><span class="sxs-lookup"><span data-stu-id="01d51-249">Default value</span></span> | <span data-ttu-id="01d51-250">Description</span><span class="sxs-lookup"><span data-stu-id="01d51-250">Description</span></span> |
| --- | --- | --- |
| <span data-ttu-id="01d51-251">MaxRetryAttemptsOnThrottledRequests</span><span class="sxs-lookup"><span data-stu-id="01d51-251">MaxRetryAttemptsOnThrottledRequests</span></span> |<span data-ttu-id="01d51-252">9</span><span class="sxs-lookup"><span data-stu-id="01d51-252">9</span></span> |<span data-ttu-id="01d51-253">Nombre maximal de nouvelles tentatives en cas d’échec de la requête, car Cosmos DB a appliqué une limite du débit au client.</span><span class="sxs-lookup"><span data-stu-id="01d51-253">The maximum number of retries if the request fails because Cosmos DB applied rate limiting on the client.</span></span> |
| <span data-ttu-id="01d51-254">MaxRetryWaitTimeInSeconds</span><span class="sxs-lookup"><span data-stu-id="01d51-254">MaxRetryWaitTimeInSeconds</span></span> |<span data-ttu-id="01d51-255">30</span><span class="sxs-lookup"><span data-stu-id="01d51-255">30</span></span> |<span data-ttu-id="01d51-256">La durée maximale de nouvelle tentative en secondes.</span><span class="sxs-lookup"><span data-stu-id="01d51-256">The maximum retry time in seconds.</span></span> |

### <a name="example"></a><span data-ttu-id="01d51-257">Exemples</span><span class="sxs-lookup"><span data-stu-id="01d51-257">Example</span></span>

```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey); ;
var options = client.ConnectionPolicy.RetryOptions;
options.MaxRetryAttemptsOnThrottledRequests = 5;
options.MaxRetryWaitTimeInSeconds = 15;
```

### <a name="telemetry"></a><span data-ttu-id="01d51-258">Télémétrie</span><span class="sxs-lookup"><span data-stu-id="01d51-258">Telemetry</span></span>

<span data-ttu-id="01d51-259">Les nouvelles tentatives sont enregistrées sous forme de messages de trace non structurés via un **TraceSource**.NET.</span><span class="sxs-lookup"><span data-stu-id="01d51-259">Retry attempts are logged as unstructured trace messages through a .NET **TraceSource**.</span></span> <span data-ttu-id="01d51-260">Vous devez configurer un **TraceListener** pour capturer les événements et les écrire dans un journal de destination approprié.</span><span class="sxs-lookup"><span data-stu-id="01d51-260">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span>

<span data-ttu-id="01d51-261">Par exemple, si vous ajoutez ce qui suit à votre fichier App.config, les traces seront générées dans un fichier texte dans le même emplacement que le fichier exécutable :</span><span class="sxs-lookup"><span data-stu-id="01d51-261">For example, if you add the following to your App.config file, traces will be generated in a text file in the same location as the executable:</span></span>

```xml
<configuration>
  <system.diagnostics>
    <switches>
      <add name="SourceSwitch" value="Verbose"/>
    </switches>
    <sources>
      <source name="DocDBTrace" switchName="SourceSwitch" switchType="System.Diagnostics.SourceSwitch" >
        <listeners>
          <add name="MyTextListener" type="System.Diagnostics.TextWriterTraceListener" traceOutputOptions="DateTime,ProcessId,ThreadId" initializeData="CosmosDBTrace.txt"></add>
        </listeners>
      </source>
    </sources>
  </system.diagnostics>
</configuration>
```

## <a name="event-hubs"></a><span data-ttu-id="01d51-262">Event Hubs</span><span class="sxs-lookup"><span data-stu-id="01d51-262">Event Hubs</span></span>

<span data-ttu-id="01d51-263">Azure Event Hubs est un service d’ingestion de données de télémétrie à très grande échelle qui collecte, transforme et stocke des millions d’événements.</span><span class="sxs-lookup"><span data-stu-id="01d51-263">Azure Event Hubs is a hyper-scale telemetry ingestion service that collects, transforms, and stores millions of events.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="01d51-264">Mécanisme de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="01d51-264">Retry mechanism</span></span>

<span data-ttu-id="01d51-265">Le comportement du mécanisme de nouvelle tentative dans la bibliothèque de client Azure Event Hubs est contrôlé par la propriété `RetryPolicy` sur la classe `EventHubClient`.</span><span class="sxs-lookup"><span data-stu-id="01d51-265">Retry behavior in the Azure Event Hubs Client Library is controlled by the `RetryPolicy` property on the `EventHubClient` class.</span></span> <span data-ttu-id="01d51-266">La stratégie par défaut effectue de nouvelles tentatives avec une interruption exponentielle lorsqu’Azure Event Hubs renvoie une exception temporaire `EventHubsException` ou une exception `OperationCanceledException`.</span><span class="sxs-lookup"><span data-stu-id="01d51-266">The default policy retries with exponential backoff when Azure Event Hub returns a transient `EventHubsException` or an `OperationCanceledException`.</span></span>

### <a name="example"></a><span data-ttu-id="01d51-267">Exemples</span><span class="sxs-lookup"><span data-stu-id="01d51-267">Example</span></span>

```csharp
EventHubClient client = EventHubClient.CreateFromConnectionString("[event_hub_connection_string]");
client.RetryPolicy = RetryPolicy.Default;
```

### <a name="more-information"></a><span data-ttu-id="01d51-268">Plus d’informations</span><span class="sxs-lookup"><span data-stu-id="01d51-268">More information</span></span>

[<span data-ttu-id="01d51-269">Bibliothèque de client .NET Standard pour Azure Event Hubs</span><span class="sxs-lookup"><span data-stu-id="01d51-269">.NET Standard client library for Azure Event Hubs</span></span>](https://github.com/Azure/azure-event-hubs-dotnet)

## <a name="iot-hub"></a><span data-ttu-id="01d51-270">IoT Hub</span><span class="sxs-lookup"><span data-stu-id="01d51-270">IoT Hub</span></span>

<span data-ttu-id="01d51-271">Azure IoT Hub est un service de connexion, de surveillance et de gestion des appareils pour le développement d’applications IoT.</span><span class="sxs-lookup"><span data-stu-id="01d51-271">Azure IoT Hub is a service for connecting, monitoring, and managing devices to develop Internet of Things (IoT) applications.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="01d51-272">Mécanisme de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="01d51-272">Retry mechanism</span></span>

<span data-ttu-id="01d51-273">Azure IoT device SDK peut détecter les erreurs dans le réseau, le protocole ou l’application.</span><span class="sxs-lookup"><span data-stu-id="01d51-273">The Azure IoT device SDK can detect errors in the network, protocol, or application.</span></span> <span data-ttu-id="01d51-274">Selon le type d’erreur, le Kit de développement logiciel vérifie si une nouvelle tentative doit être effectuée.</span><span class="sxs-lookup"><span data-stu-id="01d51-274">Based on the error type, the SDK checks whether a retry needs to be performed.</span></span> <span data-ttu-id="01d51-275">Si l’erreur est *récupérable*, le Kit de développement lance une nouvelle tentative à l’aide de la stratégie de nouvelle tentative configurée.</span><span class="sxs-lookup"><span data-stu-id="01d51-275">If the error is *recoverable*, the SDK begins to retry using the configured retry policy.</span></span>

<span data-ttu-id="01d51-276">La stratégie de nouvelle tentative par défaut est *temporisation exponentielle avec instabilité aléatoire*, mais celle-ci peut être configurée.</span><span class="sxs-lookup"><span data-stu-id="01d51-276">The default retry policy is *exponential back-off with random jitter*, but it can be configured.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="01d51-277">Configuration de la stratégie</span><span class="sxs-lookup"><span data-stu-id="01d51-277">Policy configuration</span></span>

<span data-ttu-id="01d51-278">La configuration de la stratégie diffère selon le langage.</span><span class="sxs-lookup"><span data-stu-id="01d51-278">Policy configuration differs by language.</span></span> <span data-ttu-id="01d51-279">Pour plus d’informations, consultez [Configuration de stratégie de nouvelle tentative d’IoT Hub](/azure/iot-hub/iot-hub-reliability-features-in-sdks#retry-policy-apis).</span><span class="sxs-lookup"><span data-stu-id="01d51-279">For more details, see [IoT Hub retry policy configuration](/azure/iot-hub/iot-hub-reliability-features-in-sdks#retry-policy-apis).</span></span>

### <a name="more-information"></a><span data-ttu-id="01d51-280">Plus d’informations</span><span class="sxs-lookup"><span data-stu-id="01d51-280">More information</span></span>

- [<span data-ttu-id="01d51-281">Stratégie de nouvelle tentative d’IoT Hub</span><span class="sxs-lookup"><span data-stu-id="01d51-281">IoT Hub retry policy</span></span>](/azure/iot-hub/iot-hub-reliability-features-in-sdks)
- [<span data-ttu-id="01d51-282">Résoudre les problèmes de déconnexion d’appareil IoT Hub</span><span class="sxs-lookup"><span data-stu-id="01d51-282">Troubleshoot IoT Hub device disconnection</span></span>](/azure/iot-hub/iot-hub-troubleshoot-connectivity)

## <a name="azure-redis-cache"></a><span data-ttu-id="01d51-283">Cache Redis Azure</span><span class="sxs-lookup"><span data-stu-id="01d51-283">Azure Redis Cache</span></span>

<span data-ttu-id="01d51-284">Cache Redis Azure est un service de cache de faible latence et d’accès aux données rapide basé sur le cache Redis open source connu.</span><span class="sxs-lookup"><span data-stu-id="01d51-284">Azure Redis Cache is a fast data access and low latency cache service based on the popular open source Redis Cache.</span></span> <span data-ttu-id="01d51-285">Il est sécurisé, géré par Microsoft et accessible à partir de n’importe quelle application dans Azure.</span><span class="sxs-lookup"><span data-stu-id="01d51-285">It is secure, managed by Microsoft, and is accessible from any application in Azure.</span></span>

<span data-ttu-id="01d51-286">Les instructions de cette section sont basées sur l’utilisation du client StackExchange.Redis pour accéder au cache.</span><span class="sxs-lookup"><span data-stu-id="01d51-286">The guidance in this section is based on using the StackExchange.Redis client to access the cache.</span></span> <span data-ttu-id="01d51-287">Vous trouverez une liste des autres clients appropriés sur le [site web de Redis](https://redis.io/clients), qui peuvent avoir des mécanismes de nouvelle tentative différents.</span><span class="sxs-lookup"><span data-stu-id="01d51-287">A list of other suitable clients can be found on the [Redis website](https://redis.io/clients), and these may have different retry mechanisms.</span></span>

<span data-ttu-id="01d51-288">Notez que le client StackExchange.Redis utilise le multiplexage via une connexion unique.</span><span class="sxs-lookup"><span data-stu-id="01d51-288">Note that the StackExchange.Redis client uses multiplexing through a single connection.</span></span> <span data-ttu-id="01d51-289">L’utilisation recommandée consiste à créer une instance du client au démarrage de l’application et à utiliser cette instance pour toutes les opérations sur le cache.</span><span class="sxs-lookup"><span data-stu-id="01d51-289">The recommended usage is to create an instance of the client at application startup and use this instance for all operations against the cache.</span></span> <span data-ttu-id="01d51-290">Pour cette raison, la connexion au cache est effectuée qu’une seule fois, et par conséquent, toutes les instructions de cette section est associé à la stratégie de nouvelle tentative pour cette connexion initiale &mdash; et non pour chaque opération qui accède au cache.</span><span class="sxs-lookup"><span data-stu-id="01d51-290">For this reason, the connection to the cache is made only once, and so all of the guidance in this section is related to the retry policy for this initial connection &mdash; and not for each operation that accesses the cache.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="01d51-291">Mécanisme de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="01d51-291">Retry mechanism</span></span>

<span data-ttu-id="01d51-292">Le client StackExchange.Redis utilise une classe de gestionnaire de connexions qui est configurée par le biais d’un ensemble d’options, incluant :</span><span class="sxs-lookup"><span data-stu-id="01d51-292">The StackExchange.Redis client uses a connection manager class that is configured through a set of options, including:</span></span>

- <span data-ttu-id="01d51-293">**ConnectRetry**.</span><span class="sxs-lookup"><span data-stu-id="01d51-293">**ConnectRetry**.</span></span> <span data-ttu-id="01d51-294">Nombre de fois où une connexion au cache ayant échoué fera l’objet d’une nouvelle tentative.</span><span class="sxs-lookup"><span data-stu-id="01d51-294">The number of times a failed connection to the cache will be retried.</span></span>
- <span data-ttu-id="01d51-295">**ReconnectRetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="01d51-295">**ReconnectRetryPolicy**.</span></span> <span data-ttu-id="01d51-296">Stratégie de nouvelle tentative à utiliser.</span><span class="sxs-lookup"><span data-stu-id="01d51-296">The retry strategy to use.</span></span>
- <span data-ttu-id="01d51-297">**ConnectTimeout**.</span><span class="sxs-lookup"><span data-stu-id="01d51-297">**ConnectTimeout**.</span></span> <span data-ttu-id="01d51-298">Temps d’attente maximal en millisecondes.</span><span class="sxs-lookup"><span data-stu-id="01d51-298">The maximum waiting time in milliseconds.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="01d51-299">Configuration de la stratégie</span><span class="sxs-lookup"><span data-stu-id="01d51-299">Policy configuration</span></span>

<span data-ttu-id="01d51-300">Les stratégies de nouvelle tentative sont configurées par programme en définissant les options pour le client avant de se connecter au cache.</span><span class="sxs-lookup"><span data-stu-id="01d51-300">Retry policies are configured programmatically by setting the options for the client before connecting to the cache.</span></span> <span data-ttu-id="01d51-301">Cela peut être effectué en créant une instance de la classe **ConfigurationOptions**, en remplissant ses propriétés et en la transmettant à la méthode **Connect**.</span><span class="sxs-lookup"><span data-stu-id="01d51-301">This can be done by creating an instance of the **ConfigurationOptions** class, populating its properties, and passing it to the **Connect** method.</span></span>

<span data-ttu-id="01d51-302">Les classes intégrées prennent en charge un délai linéaire (constant) et une interruption exponentielle avec des intervalles avant nouvelle tentative aléatoires.</span><span class="sxs-lookup"><span data-stu-id="01d51-302">The built-in classes support linear (constant) delay and exponential backoff with randomized retry intervals.</span></span> <span data-ttu-id="01d51-303">Vous pouvez également créer une stratégie de nouvelle tentative personnalisée en implémentant l’interface **IReconnectRetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="01d51-303">You can also create a custom retry policy by implementing the **IReconnectRetryPolicy** interface.</span></span>

<span data-ttu-id="01d51-304">L’exemple ci-après configure une stratégie de nouvelle tentative à l’aide d’une interruption exponentielle.</span><span class="sxs-lookup"><span data-stu-id="01d51-304">The following example configures a retry strategy using exponential backoff.</span></span>

```csharp
var deltaBackOffInMilliseconds = TimeSpan.FromSeconds(5).Milliseconds;
var maxDeltaBackOffInMilliseconds = TimeSpan.FromSeconds(20).Milliseconds;
var options = new ConfigurationOptions
{
    EndPoints = {"localhost"},
    ConnectRetry = 3,
    ReconnectRetryPolicy = new ExponentialRetry(deltaBackOffInMilliseconds, maxDeltaBackOffInMilliseconds),
    ConnectTimeout = 2000
};
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="01d51-305">Vous pouvez également spécifier les options sous forme de chaîne et la transmettre à la méthode **Connect** .</span><span class="sxs-lookup"><span data-stu-id="01d51-305">Alternatively, you can specify the options as a string, and pass this to the **Connect** method.</span></span> <span data-ttu-id="01d51-306">Notez que la propriété **ReconnectRetryPolicy** n’est pas définissable de cette façon, mais uniquement au moyen du code.</span><span class="sxs-lookup"><span data-stu-id="01d51-306">Note that the **ReconnectRetryPolicy** property cannot be set this way, only through code.</span></span>

```csharp
var options = "localhost,connectRetry=3,connectTimeout=2000";
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="01d51-307">Vous pouvez également spécifier des options directement lorsque vous vous connectez au cache.</span><span class="sxs-lookup"><span data-stu-id="01d51-307">You can also specify options directly when you connect to the cache.</span></span>

```csharp
var conn = ConnectionMultiplexer.Connect("redis0:6380,redis1:6380,connectRetry=3");
```

<span data-ttu-id="01d51-308">Pour plus d’informations, consultez la section concernant la [configuration de StackExchange.Redis](https://stackexchange.github.io/StackExchange.Redis/Configuration) dans la documentation de StackExchange.Redis.</span><span class="sxs-lookup"><span data-stu-id="01d51-308">For more information, see [Stack Exchange Redis Configuration](https://stackexchange.github.io/StackExchange.Redis/Configuration) in the StackExchange.Redis documentation.</span></span>

<span data-ttu-id="01d51-309">Le tableau suivant présente les paramètres par défaut pour la stratégie de nouvelle tentative intégrée.</span><span class="sxs-lookup"><span data-stu-id="01d51-309">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="01d51-310">**Contexte**</span><span class="sxs-lookup"><span data-stu-id="01d51-310">**Context**</span></span> | <span data-ttu-id="01d51-311">**Paramètre**</span><span class="sxs-lookup"><span data-stu-id="01d51-311">**Setting**</span></span> | <span data-ttu-id="01d51-312">**Valeur par défaut**</span><span class="sxs-lookup"><span data-stu-id="01d51-312">**Default value**</span></span><br /><span data-ttu-id="01d51-313">(v 1.2.2)</span><span class="sxs-lookup"><span data-stu-id="01d51-313">(v 1.2.2)</span></span> | <span data-ttu-id="01d51-314">**Signification**</span><span class="sxs-lookup"><span data-stu-id="01d51-314">**Meaning**</span></span> |
| --- | --- | --- | --- |
| <span data-ttu-id="01d51-315">ConfigurationOptions</span><span class="sxs-lookup"><span data-stu-id="01d51-315">ConfigurationOptions</span></span> |<span data-ttu-id="01d51-316">ConnectRetry</span><span class="sxs-lookup"><span data-stu-id="01d51-316">ConnectRetry</span></span><br /><br /><span data-ttu-id="01d51-317">ConnectTimeout</span><span class="sxs-lookup"><span data-stu-id="01d51-317">ConnectTimeout</span></span><br /><br /><span data-ttu-id="01d51-318">SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="01d51-318">SyncTimeout</span></span><br /><br /><span data-ttu-id="01d51-319">ReconnectRetryPolicy</span><span class="sxs-lookup"><span data-stu-id="01d51-319">ReconnectRetryPolicy</span></span> |<span data-ttu-id="01d51-320">3</span><span class="sxs-lookup"><span data-stu-id="01d51-320">3</span></span><br /><br /><span data-ttu-id="01d51-321">5 000 ms maximum plus SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="01d51-321">Maximum 5000 ms plus SyncTimeout</span></span><br /><span data-ttu-id="01d51-322">1 000</span><span class="sxs-lookup"><span data-stu-id="01d51-322">1000</span></span><br /><br /><span data-ttu-id="01d51-323">LinearRetry 5 000 ms</span><span class="sxs-lookup"><span data-stu-id="01d51-323">LinearRetry 5000 ms</span></span> |<span data-ttu-id="01d51-324">Le nombre de répétitions de tentatives de connexion pendant l’opération de connexion initiale.</span><span class="sxs-lookup"><span data-stu-id="01d51-324">The number of times to repeat connect attempts during the initial connection operation.</span></span><br /><span data-ttu-id="01d51-325">Délai d’attente (ms) pour les opérations de connexion.</span><span class="sxs-lookup"><span data-stu-id="01d51-325">Timeout (ms) for connect operations.</span></span> <span data-ttu-id="01d51-326">Pas un délai entre chaque tentative.</span><span class="sxs-lookup"><span data-stu-id="01d51-326">Not a delay between retry attempts.</span></span><br /><span data-ttu-id="01d51-327">Temps (ms) pour permettre des opérations synchrones.</span><span class="sxs-lookup"><span data-stu-id="01d51-327">Time (ms) to allow for synchronous operations.</span></span><br /><br /><span data-ttu-id="01d51-328">Nouvelle tentative toutes les 5 000 ms.</span><span class="sxs-lookup"><span data-stu-id="01d51-328">Retry every 5000 ms.</span></span>|

> [!NOTE]
> <span data-ttu-id="01d51-329">Dans le cas des opérations synchrones, le paramètre `SyncTimeout` peut augmenter la latence de bout en bout, mais la définition d’une valeur trop faible risque d’entraîner des délais d’expiration excessifs.</span><span class="sxs-lookup"><span data-stu-id="01d51-329">For synchronous operations, `SyncTimeout` can add to the end-to-end latency, but setting the value too low can cause excessive timeouts.</span></span> <span data-ttu-id="01d51-330">Consultez l’article [Résolution des problèmes du cache Redis Azure][redis-cache-troubleshoot].</span><span class="sxs-lookup"><span data-stu-id="01d51-330">See [How to troubleshoot Azure Redis Cache][redis-cache-troubleshoot].</span></span> <span data-ttu-id="01d51-331">En règle générale, utilisez des opérations asynchrones plutôt que des opérations synchrones.</span><span class="sxs-lookup"><span data-stu-id="01d51-331">In general, avoid using synchronous operations, and use asynchronous operations instead.</span></span> <span data-ttu-id="01d51-332">Pour plus d’informations, consultez [Pipelines et multiplexeurs](https://github.com/StackExchange/StackExchange.Redis/blob/master/docs/PipelinesMultiplexers.md).</span><span class="sxs-lookup"><span data-stu-id="01d51-332">For more information see [Pipelines and Multiplexers](https://github.com/StackExchange/StackExchange.Redis/blob/master/docs/PipelinesMultiplexers.md).</span></span>

### <a name="retry-usage-guidance"></a><span data-ttu-id="01d51-333">Guide d’utilisation des nouvelles tentatives</span><span class="sxs-lookup"><span data-stu-id="01d51-333">Retry usage guidance</span></span>

<span data-ttu-id="01d51-334">Respectez les consignes suivantes lors de l’utilisation du cache Redis Azure :</span><span class="sxs-lookup"><span data-stu-id="01d51-334">Consider the following guidelines when using Azure Redis Cache:</span></span>

- <span data-ttu-id="01d51-335">Le client StackExchange Redis gère ses propres nouvelles tentatives, mais uniquement lors de l’établissement d’une connexion au cache au premier démarrage de l’application.</span><span class="sxs-lookup"><span data-stu-id="01d51-335">The StackExchange Redis client manages its own retries, but only when establishing a connection to the cache when the application first starts.</span></span> <span data-ttu-id="01d51-336">Vous pouvez configurer le délai de connexion, le nombre de nouvelles tentatives et l’intervalle avant nouvelle tentative qui sont nécessaires pour établir cette connexion. Toutefois, la stratégie de nouvelle tentative ne s’applique pas aux opérations sur le cache.</span><span class="sxs-lookup"><span data-stu-id="01d51-336">You can configure the connection timeout, the number of retry attempts, and the time between retries to establish this connection, but the retry policy does not apply to operations against the cache.</span></span>
- <span data-ttu-id="01d51-337">Au lieu d’utiliser un grand nombre de nouvelles tentatives, envisagez une solution de secours en accédant plutôt à la source de données d’origine.</span><span class="sxs-lookup"><span data-stu-id="01d51-337">Instead of using a large number of retry attempts, consider falling back by accessing the original data source instead.</span></span>

### <a name="telemetry"></a><span data-ttu-id="01d51-338">Télémétrie</span><span class="sxs-lookup"><span data-stu-id="01d51-338">Telemetry</span></span>

<span data-ttu-id="01d51-339">Vous pouvez collecter des informations sur les connexions (mais pas sur d’autres opérations) à l’aide d’un élément **TextWriter**.</span><span class="sxs-lookup"><span data-stu-id="01d51-339">You can collect information about connections (but not other operations) using a **TextWriter**.</span></span>

```csharp
var writer = new StringWriter();
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="01d51-340">Vous trouverez ci-dessous un exemple de la sortie générée.</span><span class="sxs-lookup"><span data-stu-id="01d51-340">An example of the output this generates is shown below.</span></span>

```text
localhost:6379,connectTimeout=2000,connectRetry=3
1 unique nodes specified
Requesting tie-break from localhost:6379 > __Booksleeve_TieBreak...
Allowing endpoints 00:00:02 to respond...
localhost:6379 faulted: SocketFailure on PING
localhost:6379 failed to nominate (Faulted)
> UnableToResolvePhysicalConnection on GET
No masters detected
localhost:6379: Standalone v2.0.0, master; keep-alive: 00:01:00; int: Connecting; sub: Connecting; not in use: DidNotRespond
localhost:6379: int ops=0, qu=0, qs=0, qc=1, wr=0, sync=1, socks=2; sub ops=0, qu=0, qs=0, qc=0, wr=0, socks=2
Circular op-count snapshot; int: 0 (0.00 ops/s; spans 10s); sub: 0 (0.00 ops/s; spans 10s)
Sync timeouts: 0; fire and forget: 0; last heartbeat: -1s ago
resetting failing connections to retry...
retrying; attempts left: 2...
...
```

### <a name="examples"></a><span data-ttu-id="01d51-341">Exemples</span><span class="sxs-lookup"><span data-stu-id="01d51-341">Examples</span></span>

<span data-ttu-id="01d51-342">L’exemple de code ci-après configure un délai (linéaire) constant entre les nouvelles tentatives lors de l’initialisation du client StackExchange.Redis.</span><span class="sxs-lookup"><span data-stu-id="01d51-342">The following code example configures a constant (linear) delay between retries when initializing the StackExchange.Redis client.</span></span> <span data-ttu-id="01d51-343">Cet exemple indique comment définir la configuration à l’aide d’une instance **ConfigurationOptions**.</span><span class="sxs-lookup"><span data-stu-id="01d51-343">This example shows how to set the configuration using a **ConfigurationOptions** instance.</span></span>

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();
            {
                try
                {
                    var retryTimeInMilliseconds = TimeSpan.FromSeconds(4).Milliseconds; // delay between retries

                    // Using object-based configuration.
                    var options = new ConfigurationOptions
                                        {
                                            EndPoints = { "localhost" },
                                            ConnectRetry = 3,
                                            ReconnectRetryPolicy = new LinearRetry(retryTimeInMilliseconds)
                                        };
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

<span data-ttu-id="01d51-344">L’exemple ci-après définit la configuration en spécifiant les options sous forme de chaîne.</span><span class="sxs-lookup"><span data-stu-id="01d51-344">The next example sets the configuration by specifying the options as a string.</span></span> <span data-ttu-id="01d51-345">Le délai de connexion correspond au temps d’attente maximal autorisé pour la connexion au cache. Il ne s’agit pas du délai entre chaque nouvelle tentative.</span><span class="sxs-lookup"><span data-stu-id="01d51-345">The connection timeout is the maximum period of time to wait for a connection to the cache, not the delay between retry attempts.</span></span> <span data-ttu-id="01d51-346">Notez que la propriété **ReconnectRetryPolicy** est uniquement définissable par le biais du code.</span><span class="sxs-lookup"><span data-stu-id="01d51-346">Note that the **ReconnectRetryPolicy** property can only be set by code.</span></span>

```csharp
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();
            {
                try
                {
                    // Using string-based configuration.
                    var options = "localhost,connectRetry=3,connectTimeout=2000";
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

<span data-ttu-id="01d51-347">Pour plus d’exemples, consultez [Configuration](https://github.com/StackExchange/StackExchange.Redis/blob/master/docs/Configuration.md) sur le site web de projet.</span><span class="sxs-lookup"><span data-stu-id="01d51-347">For more examples, see [Configuration](https://github.com/StackExchange/StackExchange.Redis/blob/master/docs/Configuration.md) on the project website.</span></span>

### <a name="more-information"></a><span data-ttu-id="01d51-348">Plus d’informations</span><span class="sxs-lookup"><span data-stu-id="01d51-348">More information</span></span>

- [<span data-ttu-id="01d51-349">Site web Redis</span><span class="sxs-lookup"><span data-stu-id="01d51-349">Redis website</span></span>](https://redis.io/)

## <a name="azure-search"></a><span data-ttu-id="01d51-350">Recherche Azure</span><span class="sxs-lookup"><span data-stu-id="01d51-350">Azure Search</span></span>

<span data-ttu-id="01d51-351">Azure Search permet d’ajouter des fonctionnalités de recherche puissantes et sophistiquées à un site web ou une application, d’ajuster rapidement et aisément les résultats de recherche, et de construire des modèles de classement enrichis et optimisés.</span><span class="sxs-lookup"><span data-stu-id="01d51-351">Azure Search can be used to add powerful and sophisticated search capabilities to a website or application, quickly and easily tune search results, and construct rich and fine-tuned ranking models.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="01d51-352">Mécanisme de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="01d51-352">Retry mechanism</span></span>

<span data-ttu-id="01d51-353">Le comportement de nouvelle tentative dans le Kit de développement logiciel (SDK) Recherche Azure est contrôlé par la méthode `SetRetryPolicy` sur les classes [SearchServiceClient] et [SearchIndexClient].</span><span class="sxs-lookup"><span data-stu-id="01d51-353">Retry behavior in the Azure Search SDK is controlled by the `SetRetryPolicy` method on the [SearchServiceClient] and [SearchIndexClient] classes.</span></span> <span data-ttu-id="01d51-354">La stratégie par défaut effectue une nouvelle tentative avec temporisation exponentielle lorsque Recherche Azure renvoie une réponse 5xx ou 408 (Délai d’expiration de la demande).</span><span class="sxs-lookup"><span data-stu-id="01d51-354">The default policy retries with exponential backoff when Azure Search returns a 5xx or 408 (Request Timeout) response.</span></span>

### <a name="telemetry"></a><span data-ttu-id="01d51-355">Télémétrie</span><span class="sxs-lookup"><span data-stu-id="01d51-355">Telemetry</span></span>

<span data-ttu-id="01d51-356">Effectuez le suivi avec ETW ou via l’inscription d’un fournisseur de suivi personnalisé.</span><span class="sxs-lookup"><span data-stu-id="01d51-356">Trace with ETW or by registering a custom trace provider.</span></span> <span data-ttu-id="01d51-357">Pour plus d’informations, consultez la [documentation AutoRest][autorest].</span><span class="sxs-lookup"><span data-stu-id="01d51-357">For more information, see the [AutoRest documentation][autorest].</span></span>

## <a name="service-bus"></a><span data-ttu-id="01d51-358">Service Bus</span><span class="sxs-lookup"><span data-stu-id="01d51-358">Service Bus</span></span>

<span data-ttu-id="01d51-359">Service Bus est une plate-forme de messagerie cloud qui propose l’échange de messages d’une façon faiblement couplée avec une amélioration de la mise à l’échelle et de la résilience pour les composants d’une application, si cette dernière est hébergée dans le cloud ou sur site.</span><span class="sxs-lookup"><span data-stu-id="01d51-359">Service Bus is a cloud messaging platform that provides loosely coupled message exchange with improved scale and resiliency for components of an application, whether hosted in the cloud or on-premises.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="01d51-360">Mécanisme de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="01d51-360">Retry mechanism</span></span>

<span data-ttu-id="01d51-361">Service Bus met en œuvre des nouvelles tentatives à l’aide d’implémentations de la classe de base [RetryPolicy](/dotnet/api/microsoft.servicebus.retrypolicy) .</span><span class="sxs-lookup"><span data-stu-id="01d51-361">Service Bus implements retries using implementations of the [RetryPolicy](/dotnet/api/microsoft.servicebus.retrypolicy) base class.</span></span> <span data-ttu-id="01d51-362">Tous les clients Service Bus exposent une propriété **RetryPolicy** qui peut être définie sur une des implémentations de la classe de base **RetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="01d51-362">All of the Service Bus clients expose a **RetryPolicy** property that can be set to one of the implementations of the **RetryPolicy** base class.</span></span> <span data-ttu-id="01d51-363">Les implémentations intégrées sont :</span><span class="sxs-lookup"><span data-stu-id="01d51-363">The built-in implementations are:</span></span>

- <span data-ttu-id="01d51-364">La [classe RetryExponential](/dotnet/api/microsoft.servicebus.retryexponential).</span><span class="sxs-lookup"><span data-stu-id="01d51-364">The [RetryExponential class](/dotnet/api/microsoft.servicebus.retryexponential).</span></span> <span data-ttu-id="01d51-365">Elle expose les propriétés qui contrôlent l’intervalle de temporisation, le nombre de nouvelles tentatives et la propriété **TerminationTimeBuffer** utilisée pour limiter la durée totale nécessaire pour terminer l’opération.</span><span class="sxs-lookup"><span data-stu-id="01d51-365">This exposes properties that control the back-off interval, the retry count, and the **TerminationTimeBuffer** property that is used to limit the total time for the operation to complete.</span></span>

- <span data-ttu-id="01d51-366">La [classe NoRetry](/dotnet/api/microsoft.servicebus.noretry).</span><span class="sxs-lookup"><span data-stu-id="01d51-366">The [NoRetry class](/dotnet/api/microsoft.servicebus.noretry).</span></span> <span data-ttu-id="01d51-367">Elle est utilisée lorsque de nouvelles tentatives au niveau de l’API Service Bus ne sont pas requises, notamment lorsque les nouvelles tentatives sont gérées par un autre processus dans le cadre d’un lot ou d’une opération en plusieurs étapes.</span><span class="sxs-lookup"><span data-stu-id="01d51-367">This is used when retries at the Service Bus API level are not required, such as when retries are managed by another process as part of a batch or multiple step operation.</span></span>

<span data-ttu-id="01d51-368">Les actions de Service Bus peuvent renvoyer une plage d’exceptions, comme celles répertoriées dans [Exceptions de messagerie Service Bus](/azure/service-bus-messaging/service-bus-messaging-exceptions).</span><span class="sxs-lookup"><span data-stu-id="01d51-368">Service Bus actions can return a range of exceptions, as listed in [Service Bus messaging exceptions](/azure/service-bus-messaging/service-bus-messaging-exceptions).</span></span> <span data-ttu-id="01d51-369">La liste fournit plus d’informations sur ces exceptions si celles-ci indiquent qu’une nouvelle tentative de l’opération est requise.</span><span class="sxs-lookup"><span data-stu-id="01d51-369">The list provides information about which if these indicate that retrying the operation is appropriate.</span></span> <span data-ttu-id="01d51-370">Par exemple, une exception **ServerBusyException** indique que le client doit attendre un certain temps, puis recommencer l’opération.</span><span class="sxs-lookup"><span data-stu-id="01d51-370">For example, a **ServerBusyException** indicates that the client should wait for a period of time, then retry the operation.</span></span> <span data-ttu-id="01d51-371">La survenue d’une exception **ServerBusyException** fait également basculer le Service Bus vers un mode différent, dans lequel un délai supplémentaire de 10 secondes est ajouté aux délais de nouvelle tentative calculés.</span><span class="sxs-lookup"><span data-stu-id="01d51-371">The occurrence of a **ServerBusyException** also causes Service Bus to switch to a different mode, in which an extra 10-second delay is added to the computed retry delays.</span></span> <span data-ttu-id="01d51-372">Ce mode est réinitialisé au bout d’une courte période.</span><span class="sxs-lookup"><span data-stu-id="01d51-372">This mode is reset after a short period.</span></span>

<span data-ttu-id="01d51-373">Les exceptions renvoyées à partir du Service Bus exposent la propriété **IsTransient** qui indique si le client doit retenter l’opération.</span><span class="sxs-lookup"><span data-stu-id="01d51-373">The exceptions returned from Service Bus expose the **IsTransient** property that indicates if the client should retry the operation.</span></span> <span data-ttu-id="01d51-374">La stratégie **RetryExponential** intégrée repose sur la propriété **IsTransient** de la classe **MessagingException**, qui est la classe de base pour toutes les exceptions Service Bus.</span><span class="sxs-lookup"><span data-stu-id="01d51-374">The built-in **RetryExponential** policy relies on the **IsTransient** property in the **MessagingException** class, which is the base class for all Service Bus exceptions.</span></span> <span data-ttu-id="01d51-375">Si vous créez des implémentations personnalisées de la classe de base **RetryPolicy**, vous pouvez combiner le type d’exception et la propriété **IsTransient** pour permettre un contrôle plus précis des actions liées aux nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="01d51-375">If you create custom implementations of the **RetryPolicy** base class you could use a combination of the exception type and the **IsTransient** property to provide more fine-grained control over retry actions.</span></span> <span data-ttu-id="01d51-376">Par exemple, vous pouvez détecter une exception **QuotaExceededException** et prendre des mesures pour vider la file d’attente avant de tenter de lui renvoyer un message.</span><span class="sxs-lookup"><span data-stu-id="01d51-376">For example, you could detect a **QuotaExceededException** and take action to drain the queue before retrying sending a message to it.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="01d51-377">Configuration de la stratégie</span><span class="sxs-lookup"><span data-stu-id="01d51-377">Policy configuration</span></span>

<span data-ttu-id="01d51-378">Les stratégies de nouvelle tentative sont définies par programme et peuvent être considérées comme stratégie par défaut pour un **NamespaceManager** et un **MessagingFactory**, ou individuellement pour chaque client de messagerie.</span><span class="sxs-lookup"><span data-stu-id="01d51-378">Retry policies are set programmatically, and can be set as a default policy for a **NamespaceManager** and for a **MessagingFactory**, or individually for each messaging client.</span></span> <span data-ttu-id="01d51-379">Pour définir la stratégie de nouvelle tentative par défaut pour une session de messagerie, vous définissez la propriété **RetryPolicy** de l’élément **NamespaceManager**.</span><span class="sxs-lookup"><span data-stu-id="01d51-379">To set the default retry policy for a messaging session you set the **RetryPolicy** of the **NamespaceManager**.</span></span>

```csharp
namespaceManager.Settings.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                                maxBackoff: TimeSpan.FromSeconds(30),
                                                                maxRetryCount: 3);
```

<span data-ttu-id="01d51-380">Pour définir la stratégie de nouvelle tentative par défaut pour tous les clients créée à partir d’une fabrique de messagerie, vous définissez l’élément **RetryPolicy** de **MessagingFactory**.</span><span class="sxs-lookup"><span data-stu-id="01d51-380">To set the default retry policy for all clients created from a messaging factory, you set the **RetryPolicy** of the **MessagingFactory**.</span></span>

```csharp
messagingFactory.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                    maxBackoff: TimeSpan.FromSeconds(30),
                                                    maxRetryCount: 3);
```

<span data-ttu-id="01d51-381">Pour définir la stratégie de nouvelle tentative pour un client de messagerie ou remplacer sa stratégie par défaut, vous définissez sa propriété **RetryPolicy** à l’aide d’une instance de la classe de stratégie requise :</span><span class="sxs-lookup"><span data-stu-id="01d51-381">To set the retry policy for a messaging client, or to override its default policy, you set its **RetryPolicy** property using an instance of the required policy class:</span></span>

```csharp
client.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                            maxBackoff: TimeSpan.FromSeconds(30),
                                            maxRetryCount: 3);
```

<span data-ttu-id="01d51-382">Impossible de définir la stratégie de nouvelle tentative au niveau de l’opération individuelle.</span><span class="sxs-lookup"><span data-stu-id="01d51-382">The retry policy cannot be set at the individual operation level.</span></span> <span data-ttu-id="01d51-383">Elle s’applique à toutes les opérations pour le client de messagerie.</span><span class="sxs-lookup"><span data-stu-id="01d51-383">It applies to all operations for the messaging client.</span></span>
<span data-ttu-id="01d51-384">Le tableau suivant présente les paramètres par défaut pour la stratégie de nouvelle tentative intégrée.</span><span class="sxs-lookup"><span data-stu-id="01d51-384">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="01d51-385">Paramètre</span><span class="sxs-lookup"><span data-stu-id="01d51-385">Setting</span></span> | <span data-ttu-id="01d51-386">Valeur par défaut</span><span class="sxs-lookup"><span data-stu-id="01d51-386">Default value</span></span> | <span data-ttu-id="01d51-387">Signification</span><span class="sxs-lookup"><span data-stu-id="01d51-387">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="01d51-388">Stratégie</span><span class="sxs-lookup"><span data-stu-id="01d51-388">Policy</span></span> | <span data-ttu-id="01d51-389">Exponentielle</span><span class="sxs-lookup"><span data-stu-id="01d51-389">Exponential</span></span> | <span data-ttu-id="01d51-390">Temporisation exponentielle.</span><span class="sxs-lookup"><span data-stu-id="01d51-390">Exponential back-off.</span></span> |
| <span data-ttu-id="01d51-391">MinimalBackoff</span><span class="sxs-lookup"><span data-stu-id="01d51-391">MinimalBackoff</span></span> | <span data-ttu-id="01d51-392">0</span><span class="sxs-lookup"><span data-stu-id="01d51-392">0</span></span> | <span data-ttu-id="01d51-393">Intervalle de temporisation minimal.</span><span class="sxs-lookup"><span data-stu-id="01d51-393">Minimum back-off interval.</span></span> <span data-ttu-id="01d51-394">Cette valeur est ajoutée à l’intervalle avant nouvelle tentative calculé à partir de deltaBackoff.</span><span class="sxs-lookup"><span data-stu-id="01d51-394">This is added to the retry interval computed from deltaBackoff.</span></span> |
| <span data-ttu-id="01d51-395">MaximumBackoff</span><span class="sxs-lookup"><span data-stu-id="01d51-395">MaximumBackoff</span></span> | <span data-ttu-id="01d51-396">30 secondes</span><span class="sxs-lookup"><span data-stu-id="01d51-396">30 seconds</span></span> | <span data-ttu-id="01d51-397">Intervalle de temporisation maximal.</span><span class="sxs-lookup"><span data-stu-id="01d51-397">Maximum back-off interval.</span></span> <span data-ttu-id="01d51-398">Le paramètre MaximumBackoff est utilisé si l’intervalle avant nouvelle tentative calculé est supérieur à la valeur MaxBackoff.</span><span class="sxs-lookup"><span data-stu-id="01d51-398">MaximumBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> |
| <span data-ttu-id="01d51-399">DeltaBackoff</span><span class="sxs-lookup"><span data-stu-id="01d51-399">DeltaBackoff</span></span> | <span data-ttu-id="01d51-400">3 secondes</span><span class="sxs-lookup"><span data-stu-id="01d51-400">3 seconds</span></span> | <span data-ttu-id="01d51-401">Intervalle de temporisation entre les tentatives.</span><span class="sxs-lookup"><span data-stu-id="01d51-401">Back-off interval between retries.</span></span> <span data-ttu-id="01d51-402">Des multiples de cette durée seront utilisés pour les tentatives suivantes.</span><span class="sxs-lookup"><span data-stu-id="01d51-402">Multiples of this timespan will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="01d51-403">TimeBuffer</span><span class="sxs-lookup"><span data-stu-id="01d51-403">TimeBuffer</span></span> | <span data-ttu-id="01d51-404">5 secondes</span><span class="sxs-lookup"><span data-stu-id="01d51-404">5 seconds</span></span> | <span data-ttu-id="01d51-405">Mémoire tampon de délai d’expiration associé à la nouvelle tentative.</span><span class="sxs-lookup"><span data-stu-id="01d51-405">The termination time buffer associated with the retry.</span></span> <span data-ttu-id="01d51-406">Les nouvelles tentatives seront abandonnées si le temps restant est inférieur à la valeur TimeBuffer.</span><span class="sxs-lookup"><span data-stu-id="01d51-406">Retry attempts will be abandoned if the remaining time is less than TimeBuffer.</span></span> |
| <span data-ttu-id="01d51-407">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="01d51-407">MaxRetryCount</span></span> | <span data-ttu-id="01d51-408">10</span><span class="sxs-lookup"><span data-stu-id="01d51-408">10</span></span> | <span data-ttu-id="01d51-409">Nombre maximal de nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="01d51-409">The maximum number of retries.</span></span> |
| <span data-ttu-id="01d51-410">ServerBusyBaseSleepTime</span><span class="sxs-lookup"><span data-stu-id="01d51-410">ServerBusyBaseSleepTime</span></span> | <span data-ttu-id="01d51-411">10 secondes</span><span class="sxs-lookup"><span data-stu-id="01d51-411">10 seconds</span></span> | <span data-ttu-id="01d51-412">Si la dernière exception rencontrée était **ServerBusyException**, cette valeur sera ajoutée à l’intervalle avant nouvelle tentative calculé.</span><span class="sxs-lookup"><span data-stu-id="01d51-412">If the last exception encountered was **ServerBusyException**, this value will be added to the computed retry interval.</span></span> <span data-ttu-id="01d51-413">Cette valeur ne peut pas être modifiée.</span><span class="sxs-lookup"><span data-stu-id="01d51-413">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="01d51-414">Guide d’utilisation des nouvelles tentatives</span><span class="sxs-lookup"><span data-stu-id="01d51-414">Retry usage guidance</span></span>

<span data-ttu-id="01d51-415">Respectez les consignes suivantes lors de l’utilisation de Service Bus :</span><span class="sxs-lookup"><span data-stu-id="01d51-415">Consider the following guidelines when using Service Bus:</span></span>

- <span data-ttu-id="01d51-416">Lors de l’utilisation de l’implémentation intégrée de l’élément **RetryExponential** , ne mettez pas en œuvre d’opération de secours dès que la stratégie réagit aux exceptions liées à un serveur occupé et bascule automatiquement vers un mode de nouvelle tentative approprié</span><span class="sxs-lookup"><span data-stu-id="01d51-416">When using the built-in **RetryExponential** implementation, do not implement a fallback operation as the policy reacts to Server Busy exceptions and automatically switches to an appropriate retry mode.</span></span>
- <span data-ttu-id="01d51-417">Service Bus prend en charge une fonctionnalité appelée Espaces de noms associés, qui met en œuvre le basculement automatique vers une file d’attente de sauvegarde dans un espace de noms séparé en cas d’échec de la file d’attente dans l’espace de noms principal.</span><span class="sxs-lookup"><span data-stu-id="01d51-417">Service Bus supports a feature called Paired Namespaces, which implements automatic failover to a backup queue in a separate namespace if the queue in the primary namespace fails.</span></span> <span data-ttu-id="01d51-418">Les messages de la file d’attente secondaire peuvent être renvoyés à la file d’attente principale lors de la récupération</span><span class="sxs-lookup"><span data-stu-id="01d51-418">Messages from the secondary queue can be sent back to the primary queue when it recovers.</span></span> <span data-ttu-id="01d51-419">Cette fonctionnalité permet de traiter les erreurs temporaires.</span><span class="sxs-lookup"><span data-stu-id="01d51-419">This feature helps to address transient failures.</span></span> <span data-ttu-id="01d51-420">Pour plus d’informations, consultez [Modèles de messagerie asynchrone et haute disponibilité](/azure/service-bus-messaging/service-bus-async-messaging).</span><span class="sxs-lookup"><span data-stu-id="01d51-420">For more information, see [Asynchronous Messaging Patterns and High Availability](/azure/service-bus-messaging/service-bus-async-messaging).</span></span>

<span data-ttu-id="01d51-421">Pensez à commencer par les paramètres suivants pour les opérations liées aux nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="01d51-421">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="01d51-422">Il s’agit de paramètres généraux. Vous devez par conséquent surveiller les opérations et optimiser les valeurs en fonction de votre propre scénario.</span><span class="sxs-lookup"><span data-stu-id="01d51-422">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="01d51-423">Context</span><span class="sxs-lookup"><span data-stu-id="01d51-423">Context</span></span> | <span data-ttu-id="01d51-424">Exemple de latence maximale</span><span class="sxs-lookup"><span data-stu-id="01d51-424">Example maximum latency</span></span> | <span data-ttu-id="01d51-425">Stratégie de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="01d51-425">Retry policy</span></span> | <span data-ttu-id="01d51-426">Paramètres</span><span class="sxs-lookup"><span data-stu-id="01d51-426">Settings</span></span> | <span data-ttu-id="01d51-427">Fonctionnement</span><span class="sxs-lookup"><span data-stu-id="01d51-427">How it works</span></span> |
|---------|---------|---------|---------|---------|
| <span data-ttu-id="01d51-428">Interactif, interface utilisateur ou premier plan</span><span class="sxs-lookup"><span data-stu-id="01d51-428">Interactive, UI, or foreground</span></span> | <span data-ttu-id="01d51-429">2 secondes\*</span><span class="sxs-lookup"><span data-stu-id="01d51-429">2 seconds\*</span></span>  | <span data-ttu-id="01d51-430">Exponentielle</span><span class="sxs-lookup"><span data-stu-id="01d51-430">Exponential</span></span> | <span data-ttu-id="01d51-431">MinimumBackoff = 0</span><span class="sxs-lookup"><span data-stu-id="01d51-431">MinimumBackoff = 0</span></span> <br/> <span data-ttu-id="01d51-432">MaximumBackoff = 30 s</span><span class="sxs-lookup"><span data-stu-id="01d51-432">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="01d51-433">DeltaBackoff = 300 ms</span><span class="sxs-lookup"><span data-stu-id="01d51-433">DeltaBackoff = 300 msec.</span></span> <br/> <span data-ttu-id="01d51-434">TimeBuffer = 300 ms</span><span class="sxs-lookup"><span data-stu-id="01d51-434">TimeBuffer = 300 msec.</span></span> <br/> <span data-ttu-id="01d51-435">MaxRetryCount = 2</span><span class="sxs-lookup"><span data-stu-id="01d51-435">MaxRetryCount = 2</span></span> | <span data-ttu-id="01d51-436">Tentative 1 : Délai 0 s.</span><span class="sxs-lookup"><span data-stu-id="01d51-436">Attempt 1: Delay 0 sec.</span></span> <br/> <span data-ttu-id="01d51-437">Tentative 2 : Délai ~300 ms.</span><span class="sxs-lookup"><span data-stu-id="01d51-437">Attempt 2: Delay ~300 msec.</span></span> <br/> <span data-ttu-id="01d51-438">Tentative 3 : Délai ~900 ms.</span><span class="sxs-lookup"><span data-stu-id="01d51-438">Attempt 3: Delay ~900 msec.</span></span> |
| <span data-ttu-id="01d51-439">Arrière-plan ou lot</span><span class="sxs-lookup"><span data-stu-id="01d51-439">Background or batch</span></span> | <span data-ttu-id="01d51-440">30 secondes</span><span class="sxs-lookup"><span data-stu-id="01d51-440">30 seconds</span></span> | <span data-ttu-id="01d51-441">Exponentielle</span><span class="sxs-lookup"><span data-stu-id="01d51-441">Exponential</span></span> | <span data-ttu-id="01d51-442">MinimumBackoff = 1</span><span class="sxs-lookup"><span data-stu-id="01d51-442">MinimumBackoff = 1</span></span> <br/> <span data-ttu-id="01d51-443">MaximumBackoff = 30 s</span><span class="sxs-lookup"><span data-stu-id="01d51-443">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="01d51-444">DeltaBackoff = 1,75 s</span><span class="sxs-lookup"><span data-stu-id="01d51-444">DeltaBackoff = 1.75 sec.</span></span> <br/> <span data-ttu-id="01d51-445">TimeBuffer = 5 s</span><span class="sxs-lookup"><span data-stu-id="01d51-445">TimeBuffer = 5 sec.</span></span> <br/> <span data-ttu-id="01d51-446">MaxRetryCount = 3</span><span class="sxs-lookup"><span data-stu-id="01d51-446">MaxRetryCount = 3</span></span> | <span data-ttu-id="01d51-447">Tentative 1 : Délai ~1 s.</span><span class="sxs-lookup"><span data-stu-id="01d51-447">Attempt 1: Delay ~1 sec.</span></span> <br/> <span data-ttu-id="01d51-448">Tentative 2 : Délai ~3 s.</span><span class="sxs-lookup"><span data-stu-id="01d51-448">Attempt 2: Delay ~3 sec.</span></span> <br/> <span data-ttu-id="01d51-449">Tentative 3 : Délai ~6 ms.</span><span class="sxs-lookup"><span data-stu-id="01d51-449">Attempt 3: Delay ~6 msec.</span></span> <br/> <span data-ttu-id="01d51-450">Tentative 4 : Délai ~13 ms.</span><span class="sxs-lookup"><span data-stu-id="01d51-450">Attempt 4: Delay ~13 msec.</span></span> |

<span data-ttu-id="01d51-451">\* N’inclut pas le délai supplémentaire qui s’ajoute en cas de réception d’une réponse du type Serveur occupé.</span><span class="sxs-lookup"><span data-stu-id="01d51-451">\* Not including additional delay that is added if a Server Busy response is received.</span></span>

### <a name="telemetry"></a><span data-ttu-id="01d51-452">Télémétrie</span><span class="sxs-lookup"><span data-stu-id="01d51-452">Telemetry</span></span>

<span data-ttu-id="01d51-453">Service Bus enregistre les nouvelles tentatives sous forme d’événements ETW à l’aide d’ **EventSource**.</span><span class="sxs-lookup"><span data-stu-id="01d51-453">Service Bus logs retries as ETW events using an **EventSource**.</span></span> <span data-ttu-id="01d51-454">Vous devez associer un élément **EventListener** à la source d’événement pour capturer les événements et les afficher dans la visionneuse de performances ou les écrire dans un journal de destination approprié.</span><span class="sxs-lookup"><span data-stu-id="01d51-454">You must attach an **EventListener** to the event source to capture the events and view them in Performance Viewer, or write them to a suitable destination log.</span></span> <span data-ttu-id="01d51-455">Les événements de nouvelle tentative se présentent sous la forme suivante :</span><span class="sxs-lookup"><span data-stu-id="01d51-455">The retry events are of the following form:</span></span>

```text
Microsoft-ServiceBus-Client/RetryPolicyIteration
ThreadID="14,500"
FormattedMessage="[TrackingId:] RetryExponential: Operation Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05 at iteration 0 is retrying after 00:00:00.1000000 sleep because of Microsoft.ServiceBus.Messaging.MessagingCommunicationException: The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3, TimeStamp:9/5/2014 10:00:13 PM."
trackingId=""
policyType="RetryExponential"
operation="Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05"
iteration="0"
iterationSleep="00:00:00.1000000"
lastExceptionType="Microsoft.ServiceBus.Messaging.MessagingCommunicationException"
exceptionMessage="The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3,TimeStamp:9/5/2014 10:00:13 PM"
```

### <a name="examples"></a><span data-ttu-id="01d51-456">Exemples</span><span class="sxs-lookup"><span data-stu-id="01d51-456">Examples</span></span>

<span data-ttu-id="01d51-457">L’exemple de code suivant montre comment définir la stratégie de nouvelle tentative pour :</span><span class="sxs-lookup"><span data-stu-id="01d51-457">The following code example shows how to set the retry policy for:</span></span>

- <span data-ttu-id="01d51-458">Un gestionnaire d’espace de noms.</span><span class="sxs-lookup"><span data-stu-id="01d51-458">A namespace manager.</span></span> <span data-ttu-id="01d51-459">La stratégie s’applique à toutes les opérations sur ce gestionnaire et ne peut pas être remplacée pour des opérations individuelles.</span><span class="sxs-lookup"><span data-stu-id="01d51-459">The policy applies to all operations on that manager, and cannot be overridden for individual operations.</span></span>
- <span data-ttu-id="01d51-460">Une fabrique de messagerie.</span><span class="sxs-lookup"><span data-stu-id="01d51-460">A messaging factory.</span></span> <span data-ttu-id="01d51-461">La stratégie s’applique à tous les clients créés à partir de cette fabrique et ne peut pas être remplacée lors de la création de clients individuels.</span><span class="sxs-lookup"><span data-stu-id="01d51-461">The policy applies to all clients created from that factory, and cannot be overridden when creating individual clients.</span></span>
- <span data-ttu-id="01d51-462">Un client de messagerie individuel.</span><span class="sxs-lookup"><span data-stu-id="01d51-462">An individual messaging client.</span></span> <span data-ttu-id="01d51-463">Après avoir créé un client, vous pouvez définir la stratégie de nouvelle tentative pour ce client.</span><span class="sxs-lookup"><span data-stu-id="01d51-463">After a client has been created, you can set the retry policy for that client.</span></span> <span data-ttu-id="01d51-464">La stratégie s’applique à toutes les opérations sur ce client.</span><span class="sxs-lookup"><span data-stu-id="01d51-464">The policy applies to all operations on that client.</span></span>

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Messaging;

namespace RetryCodeSamples
{
    class ServiceBusCodeSamples
    {
        private const string connectionString =
            @"Endpoint=sb://[my-namespace].servicebus.windows.net/;
                SharedAccessKeyName=RootManageSharedAccessKey;
                SharedAccessKey=C99..........Mk=";

        public async static Task Samples()
        {
            const string QueueName = "TestQueue";

            ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.Http;

            var namespaceManager = NamespaceManager.CreateFromConnectionString(connectionString);

            // The namespace manager will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for all operations on the namespace manager.
                namespaceManager.Settings.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);

                // Policies cannot be specified on a per-operation basis.
                if (!await namespaceManager.QueueExistsAsync(QueueName))
                {
                    await namespaceManager.CreateQueueAsync(QueueName);
                }
            }

            var messagingFactory = MessagingFactory.Create(
                namespaceManager.Address, namespaceManager.Settings.TokenProvider);
            // The messaging factory will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for clients created from it.
                messagingFactory.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);


                // Policies cannot be specified on a per-operation basis.
                var session = await messagingFactory.AcceptMessageSessionAsync();
            }

            {
                var client = messagingFactory.CreateQueueClient(QueueName);
                // The client inherits the policy from the factory that created it.


                // Set different values for the retry policy on the client.
                client.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0.1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);

                // Policies cannot be specified on a per-operation basis.
                var session = await client.AcceptMessageSessionAsync();
            }
        }
    }
}
```

### <a name="more-information"></a><span data-ttu-id="01d51-465">Plus d’informations</span><span class="sxs-lookup"><span data-stu-id="01d51-465">More information</span></span>

- [<span data-ttu-id="01d51-466">Modèles de messagerie asynchrone et haute disponibilité</span><span class="sxs-lookup"><span data-stu-id="01d51-466">Asynchronous messaging patterns and high availability</span></span>](/azure/service-bus-messaging/service-bus-async-messaging)

## <a name="service-fabric"></a><span data-ttu-id="01d51-467">Service Fabric</span><span class="sxs-lookup"><span data-stu-id="01d51-467">Service Fabric</span></span>

<span data-ttu-id="01d51-468">La distribution de services fiables dans un cluster Service Fabric offre une protection contre la plupart des erreurs temporaires potentielles décrites dans cet article.</span><span class="sxs-lookup"><span data-stu-id="01d51-468">Distributing reliable services in a Service Fabric cluster guards against most of the potential transient faults discussed in this article.</span></span> <span data-ttu-id="01d51-469">Toutefois, certaines erreurs temporaires restent possibles.</span><span class="sxs-lookup"><span data-stu-id="01d51-469">Some transient faults are still possible, however.</span></span> <span data-ttu-id="01d51-470">Par exemple, si le service d’affectation de noms reçoit une requête au moment précis où il fait l’objet d’un changement de routage, il lève une exception.</span><span class="sxs-lookup"><span data-stu-id="01d51-470">For example, the naming service might be in the middle of a routing change when it gets a request, causing it to throw an exception.</span></span> <span data-ttu-id="01d51-471">En revanche, si la même requête arrive 100 millisecondes plus tard, elle aboutira probablement.</span><span class="sxs-lookup"><span data-stu-id="01d51-471">If the same request comes 100 milliseconds later, it will probably succeed.</span></span>

<span data-ttu-id="01d51-472">Service Fabric gère ce type d’erreur temporaire en interne.</span><span class="sxs-lookup"><span data-stu-id="01d51-472">Internally, Service Fabric manages this kind of transient fault.</span></span> <span data-ttu-id="01d51-473">Vous pouvez configurer certains paramètres à l’aide de la classe `OperationRetrySettings` lorsque vous configurez vos services.</span><span class="sxs-lookup"><span data-stu-id="01d51-473">You can configure some settings by using the `OperationRetrySettings` class while setting up your services.</span></span> <span data-ttu-id="01d51-474">Le code ci-après présente un exemple.</span><span class="sxs-lookup"><span data-stu-id="01d51-474">The following code shows an example.</span></span> <span data-ttu-id="01d51-475">Dans la plupart des cas, cette opération n’est pas nécessaire, et les paramètres par défaut font l’affaire.</span><span class="sxs-lookup"><span data-stu-id="01d51-475">In most cases, this should not be necessary, and the default settings will be fine.</span></span>

```csharp
FabricTransportRemotingSettings transportSettings = new FabricTransportRemotingSettings
{
    OperationTimeout = TimeSpan.FromSeconds(30)
};

var retrySettings = new OperationRetrySettings(TimeSpan.FromSeconds(15), TimeSpan.FromSeconds(1), 5);

var clientFactory = new FabricTransportServiceRemotingClientFactory(transportSettings);

var serviceProxyFactory = new ServiceProxyFactory((c) => clientFactory, retrySettings);

var client = serviceProxyFactory.CreateServiceProxy<ISomeService>(
    new Uri("fabric:/SomeApp/SomeStatefulReliableService"),
    new ServicePartitionKey(0));
```

### <a name="more-information"></a><span data-ttu-id="01d51-476">Plus d’informations</span><span class="sxs-lookup"><span data-stu-id="01d51-476">More information</span></span>

- [<span data-ttu-id="01d51-477">Gestion des exceptions à distance</span><span class="sxs-lookup"><span data-stu-id="01d51-477">Remote exception handling</span></span>](/azure/service-fabric/service-fabric-reliable-services-communication-remoting#remoting-exception-handling)

## <a name="sql-database-using-adonet"></a><span data-ttu-id="01d51-478">Base de données SQL utilisant ADO.NET</span><span class="sxs-lookup"><span data-stu-id="01d51-478">SQL Database using ADO.NET</span></span>

<span data-ttu-id="01d51-479">La base de données SQL est une base de données SQL hébergée disponible en différentes tailles et sous forme de service standard (partagé) et premium (non partagé).</span><span class="sxs-lookup"><span data-stu-id="01d51-479">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="01d51-480">Mécanisme de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="01d51-480">Retry mechanism</span></span>

<span data-ttu-id="01d51-481">Les bases de données SQL n’intègrent aucune prise en charge pour les nouvelles tentatives en cas d’accès à l’aide d’ADO.NET.</span><span class="sxs-lookup"><span data-stu-id="01d51-481">SQL Database has no built-in support for retries when accessed using ADO.NET.</span></span> <span data-ttu-id="01d51-482">Toutefois, les codes de retour issus des demandes peuvent être utilisés pour déterminer le motif d’échec de ces dernières.</span><span class="sxs-lookup"><span data-stu-id="01d51-482">However, the return codes from requests can be used to determine why a request failed.</span></span> <span data-ttu-id="01d51-483">Pour plus d’informations sur la limitation de SQL Database, consultez l’article [Limites de ressources de base de données SQL Azure](/azure/sql-database/sql-database-resource-limits).</span><span class="sxs-lookup"><span data-stu-id="01d51-483">For more information about SQL Database throttling, see [Azure SQL Database resource limits](/azure/sql-database/sql-database-resource-limits).</span></span> <span data-ttu-id="01d51-484">Pour obtenir la liste des codes d’erreur pertinents, consultez l’article [Codes d’erreur SQL pour les applications clientes SQL Database](/azure/sql-database/sql-database-develop-error-messages).</span><span class="sxs-lookup"><span data-stu-id="01d51-484">For a list of relevant error codes, see [SQL error codes for SQL Database client applications](/azure/sql-database/sql-database-develop-error-messages).</span></span>

<span data-ttu-id="01d51-485">Vous pouvez utiliser la bibliothèque Polly afin d’implémenter le mécanisme de nouvelles tentatives pour SQL Database.</span><span class="sxs-lookup"><span data-stu-id="01d51-485">You can use the Polly library to implement retries for SQL Database.</span></span> <span data-ttu-id="01d51-486">Consultez la section [Gestion des erreurs temporaires avec Polly](#transient-fault-handling-with-polly).</span><span class="sxs-lookup"><span data-stu-id="01d51-486">See [Transient fault handling with Polly](#transient-fault-handling-with-polly).</span></span>

### <a name="retry-usage-guidance"></a><span data-ttu-id="01d51-487">Guide d’utilisation des nouvelles tentatives</span><span class="sxs-lookup"><span data-stu-id="01d51-487">Retry usage guidance</span></span>

<span data-ttu-id="01d51-488">Respectez les consignes suivantes lorsque vous accédez à la base de données SQL à l’aide d’ADO.NET :</span><span class="sxs-lookup"><span data-stu-id="01d51-488">Consider the following guidelines when accessing SQL Database using ADO.NET:</span></span>

- <span data-ttu-id="01d51-489">Choisissez l’option de service appropriée (partagée ou premium).</span><span class="sxs-lookup"><span data-stu-id="01d51-489">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="01d51-490">Une instance partagée peut être soumise à des délais de connexion plus longs que d’ordinaire et à des limitations en raison de l’utilisation du serveur partagé par d’autres locataires.</span><span class="sxs-lookup"><span data-stu-id="01d51-490">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="01d51-491">Si des performances prévisibles et des opérations fiables à faible latence sont requises, pensez à choisir l’option premium.</span><span class="sxs-lookup"><span data-stu-id="01d51-491">If more predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
- <span data-ttu-id="01d51-492">Veillez à effectuer de nouvelles tentatives au niveau ou à l’étendue qui convient pour éviter les opérations non idempotent à l’origine d’une incohérence dans les données.</span><span class="sxs-lookup"><span data-stu-id="01d51-492">Ensure that you perform retries at the appropriate level or scope to avoid non-idempotent operations causing inconsistency in the data.</span></span> <span data-ttu-id="01d51-493">Idéalement, toutes les opérations doivent être idempotent afin qu’elles puissent être répétées sans entraîner d’incohérence.</span><span class="sxs-lookup"><span data-stu-id="01d51-493">Ideally, all operations should be idempotent so that they can be repeated without causing inconsistency.</span></span> <span data-ttu-id="01d51-494">Lorsque cela n’est pas le cas, la nouvelle tentative doit être effectuée à un niveau ou à une étendue qui autorise l’annulation de toutes les modifications associées en cas d’échec d’une opération ; par exemple, depuis une étendue transactionnelle.</span><span class="sxs-lookup"><span data-stu-id="01d51-494">Where this is not the case, the retry should be performed at a level or scope that allows all related changes to be undone if one operation fails; for example, from within a transactional scope.</span></span> <span data-ttu-id="01d51-495">Pour plus d’informations, consultez [Couche d’accès aux données de Cloud Service Fundamentals - Gestion des erreurs temporaires](https://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).</span><span class="sxs-lookup"><span data-stu-id="01d51-495">For more information, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](https://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).</span></span>
- <span data-ttu-id="01d51-496">Une stratégie d’intervalle fixe n’est pas recommandée pour une utilisation avec la base de données SQL Azure à l’exception des scénarios interactifs comportant seulement quelques nouvelles tentatives à intervalles très courts.</span><span class="sxs-lookup"><span data-stu-id="01d51-496">A fixed interval strategy is not recommended for use with Azure SQL Database except for interactive scenarios where there are only a few retries at very short intervals.</span></span> <span data-ttu-id="01d51-497">Envisagez plutôt d’utiliser une stratégie de temporisation exponentielle pour la majorité des scénarios.</span><span class="sxs-lookup"><span data-stu-id="01d51-497">Instead, consider using an exponential back-off strategy for the majority of scenarios.</span></span>
- <span data-ttu-id="01d51-498">Choisissez une valeur appropriée pour les délais d’attente de connexion et de commande lors de la définition des connexions.</span><span class="sxs-lookup"><span data-stu-id="01d51-498">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="01d51-499">Un délai d’attente trop court peut entraîner des échecs de connexion prématurés lorsque la base de données est occupée.</span><span class="sxs-lookup"><span data-stu-id="01d51-499">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="01d51-500">Un délai d’attente trop long peut empêcher la logique de nouvelle tentative de fonctionner correctement en attendant trop longtemps avant la détection d’un échec de connexion.</span><span class="sxs-lookup"><span data-stu-id="01d51-500">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="01d51-501">La valeur du délai d’attente est un composant de la latence de bout en bout ; Il est en fait ajouté au délai de nouvelle tentative spécifié dans la stratégie de nouvelle tentative pour chaque nouvelle tentative.</span><span class="sxs-lookup"><span data-stu-id="01d51-501">The value of the timeout is a component of the end-to-end latency; it is effectively added to the retry delay specified in the retry policy for every retry attempt.</span></span>
- <span data-ttu-id="01d51-502">Fermez la connexion après un certain nombre de nouvelles tentatives, même lorsque vous utilisez une logique de nouvelle tentative à temporisation exponentielle et renouvelez l’opération sur une nouvelle connexion.</span><span class="sxs-lookup"><span data-stu-id="01d51-502">Close the connection after a certain number of retries, even when using an exponential back off retry logic, and retry the operation on a new connection.</span></span> <span data-ttu-id="01d51-503">Le fait d’effectuer la même opération à plusieurs reprises sur la même connexion peut être un facteur entraînant des problèmes de connexion.</span><span class="sxs-lookup"><span data-stu-id="01d51-503">Retrying the same operation multiple times on the same connection can be a factor that contributes to connection problems.</span></span> <span data-ttu-id="01d51-504">Pour voir un exemple de cette technique, consultez [Couche d’accès aux données de Cloud Service Fundamentals - Gestion des erreurs temporaires](https://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).</span><span class="sxs-lookup"><span data-stu-id="01d51-504">For an example of this technique, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](https://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).</span></span>
- <span data-ttu-id="01d51-505">Lorsque le regroupement de connexions est en cours d’utilisation (par défaut), il est probable que la même connexion sera choisie à partir du pool, même après la fermeture et la réouverture d’une connexion.</span><span class="sxs-lookup"><span data-stu-id="01d51-505">When connection pooling is in use (the default) there is a chance that the same connection will be chosen from the pool, even after closing and reopening a connection.</span></span> <span data-ttu-id="01d51-506">Si c’est le cas, une technique permettant de résoudre ce problème consiste à appeler la méthode **ClearPool** de la classe **SqlConnection** pour marquer la connexion comme n’étant pas réutilisable.</span><span class="sxs-lookup"><span data-stu-id="01d51-506">If this is the case, a technique to resolve it is to call the **ClearPool** method of the **SqlConnection** class to mark the connection as not reusable.</span></span> <span data-ttu-id="01d51-507">Toutefois, vous ne devez effectuer cette opération qu’après l’échec de plusieurs tentatives de connexion, et uniquement lorsque vous rencontrez la classe spécifique d’erreurs temporaires, relative notamment à des délais d’attente SQL (code d’erreur -2) liés à des connexions défectueuses.</span><span class="sxs-lookup"><span data-stu-id="01d51-507">However, you should do this only after several connection attempts have failed, and only when encountering the specific class of transient failures such as SQL timeouts (error code -2) related to faulty connections.</span></span>
- <span data-ttu-id="01d51-508">Si le code d’accès aux données utilise des transactions lancées en tant qu’instances **TransactionScope** , la logique de nouvelle tentative doit ouvrir de nouveau la connexion et lancer une nouvelle étendue de transaction.</span><span class="sxs-lookup"><span data-stu-id="01d51-508">If the data access code uses transactions initiated as **TransactionScope** instances, the retry logic should reopen the connection and initiate a new transaction scope.</span></span> <span data-ttu-id="01d51-509">Pour cette raison, le bloc de code renouvelable doit englober l’ensemble de la portée de la transaction.</span><span class="sxs-lookup"><span data-stu-id="01d51-509">For this reason, the retryable code block should encompass the entire scope of the transaction.</span></span>

<span data-ttu-id="01d51-510">Pensez à commencer par les paramètres suivants pour les opérations liées aux nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="01d51-510">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="01d51-511">Il s’agit de paramètres généraux. Vous devez par conséquent surveiller les opérations et optimiser les valeurs en fonction de votre propre scénario.</span><span class="sxs-lookup"><span data-stu-id="01d51-511">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="01d51-512">**Contexte**</span><span class="sxs-lookup"><span data-stu-id="01d51-512">**Context**</span></span> | <span data-ttu-id="01d51-513">**Exemple de cible E2E<br />latence max**</span><span class="sxs-lookup"><span data-stu-id="01d51-513">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="01d51-514">**Stratégie de nouvelle tentative**</span><span class="sxs-lookup"><span data-stu-id="01d51-514">**Retry strategy**</span></span> | <span data-ttu-id="01d51-515">**Paramètres**</span><span class="sxs-lookup"><span data-stu-id="01d51-515">**Settings**</span></span> | <span data-ttu-id="01d51-516">**Valeurs**</span><span class="sxs-lookup"><span data-stu-id="01d51-516">**Values**</span></span> | <span data-ttu-id="01d51-517">**Fonctionnement**</span><span class="sxs-lookup"><span data-stu-id="01d51-517">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="01d51-518">Interactif, interface utilisateur,</span><span class="sxs-lookup"><span data-stu-id="01d51-518">Interactive, UI,</span></span><br /><span data-ttu-id="01d51-519">ou premier plan</span><span class="sxs-lookup"><span data-stu-id="01d51-519">or foreground</span></span> |<span data-ttu-id="01d51-520">2 secondes</span><span class="sxs-lookup"><span data-stu-id="01d51-520">2 sec</span></span> |<span data-ttu-id="01d51-521">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="01d51-521">FixedInterval</span></span> |<span data-ttu-id="01d51-522">Nombre de tentatives</span><span class="sxs-lookup"><span data-stu-id="01d51-522">Retry count</span></span><br /><span data-ttu-id="01d51-523">Intervalle avant nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="01d51-523">Retry interval</span></span><br /><span data-ttu-id="01d51-524">Première nouvelle tentative rapide</span><span class="sxs-lookup"><span data-stu-id="01d51-524">First fast retry</span></span> |<span data-ttu-id="01d51-525">3</span><span class="sxs-lookup"><span data-stu-id="01d51-525">3</span></span><br /><span data-ttu-id="01d51-526">500 ms</span><span class="sxs-lookup"><span data-stu-id="01d51-526">500 ms</span></span><br /><span data-ttu-id="01d51-527">true</span><span class="sxs-lookup"><span data-stu-id="01d51-527">true</span></span> |<span data-ttu-id="01d51-528">Tentative 1 - délai 0 s</span><span class="sxs-lookup"><span data-stu-id="01d51-528">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="01d51-529">Tentative 2 - délai 500 ms</span><span class="sxs-lookup"><span data-stu-id="01d51-529">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="01d51-530">Tentative 3 - délai 500 ms</span><span class="sxs-lookup"><span data-stu-id="01d51-530">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="01d51-531">Arrière-plan</span><span class="sxs-lookup"><span data-stu-id="01d51-531">Background</span></span><br /><span data-ttu-id="01d51-532">ou lot</span><span class="sxs-lookup"><span data-stu-id="01d51-532">or batch</span></span> |<span data-ttu-id="01d51-533">30 secondes</span><span class="sxs-lookup"><span data-stu-id="01d51-533">30 sec</span></span> |<span data-ttu-id="01d51-534">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="01d51-534">ExponentialBackoff</span></span> |<span data-ttu-id="01d51-535">Nombre de tentatives</span><span class="sxs-lookup"><span data-stu-id="01d51-535">Retry count</span></span><br /><span data-ttu-id="01d51-536">Temporisation min</span><span class="sxs-lookup"><span data-stu-id="01d51-536">Min back-off</span></span><br /><span data-ttu-id="01d51-537">Temporisation max</span><span class="sxs-lookup"><span data-stu-id="01d51-537">Max back-off</span></span><br /><span data-ttu-id="01d51-538">Temporisation delta</span><span class="sxs-lookup"><span data-stu-id="01d51-538">Delta back-off</span></span><br /><span data-ttu-id="01d51-539">Première nouvelle tentative rapide</span><span class="sxs-lookup"><span data-stu-id="01d51-539">First fast retry</span></span> |<span data-ttu-id="01d51-540">5.</span><span class="sxs-lookup"><span data-stu-id="01d51-540">5</span></span><br /><span data-ttu-id="01d51-541">0 seconde</span><span class="sxs-lookup"><span data-stu-id="01d51-541">0 sec</span></span><br /><span data-ttu-id="01d51-542">60 secondes</span><span class="sxs-lookup"><span data-stu-id="01d51-542">60 sec</span></span><br /><span data-ttu-id="01d51-543">2 secondes</span><span class="sxs-lookup"><span data-stu-id="01d51-543">2 sec</span></span><br /><span data-ttu-id="01d51-544">false</span><span class="sxs-lookup"><span data-stu-id="01d51-544">false</span></span> |<span data-ttu-id="01d51-545">Tentative 1 - délai 0 s</span><span class="sxs-lookup"><span data-stu-id="01d51-545">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="01d51-546">Tentative 2 - délai ~2 s</span><span class="sxs-lookup"><span data-stu-id="01d51-546">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="01d51-547">Tentative 3 - délai ~6 s</span><span class="sxs-lookup"><span data-stu-id="01d51-547">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="01d51-548">Tentative 4 - délai ~14 s</span><span class="sxs-lookup"><span data-stu-id="01d51-548">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="01d51-549">Tentative 5 - délai ~30 s</span><span class="sxs-lookup"><span data-stu-id="01d51-549">Attempt 5 - delay ~30 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="01d51-550">Les cibles de latence de bout en bout supposent le délai d’attente par défaut pour les connexions au service.</span><span class="sxs-lookup"><span data-stu-id="01d51-550">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="01d51-551">Si vous spécifiez des délais de connexion, la latence de bout en bout sera prolongée de ce temps supplémentaire pour toute nouvelle tentative.</span><span class="sxs-lookup"><span data-stu-id="01d51-551">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>

### <a name="examples"></a><span data-ttu-id="01d51-552">Exemples</span><span class="sxs-lookup"><span data-stu-id="01d51-552">Examples</span></span>

<span data-ttu-id="01d51-553">Cette section vous indique comment utiliser Polly pour accéder à Azure SQL Database à l’aide d’un ensemble de stratégies de nouvelle tentative configurées dans la classe `Policy`.</span><span class="sxs-lookup"><span data-stu-id="01d51-553">This section shows how you can use Polly to access Azure SQL Database using a set of retry policies configured in the `Policy` class.</span></span>

<span data-ttu-id="01d51-554">Le code ci-après présente l’utilisation d’une méthode d’extension sur la classe `SqlCommand` qui appelle `ExecuteAsync` avec une interruption exponentielle.</span><span class="sxs-lookup"><span data-stu-id="01d51-554">The following code shows an extension method on the `SqlCommand` class that calls `ExecuteAsync` with exponential backoff.</span></span>

```csharp
public async static Task<SqlDataReader> ExecuteReaderWithRetryAsync(this SqlCommand command)
{
    GuardConnectionIsNotNull(command);

    var policy = Policy.Handle<Exception>().WaitAndRetryAsync(
        retryCount: 3, // Retry 3 times
        sleepDurationProvider: attempt => TimeSpan.FromMilliseconds(200 * Math.Pow(2, attempt - 1)), // Exponential backoff based on an initial 200ms delay.
        onRetry: (exception, attempt) =>
        {
            // Capture some info for logging/telemetry.
            logger.LogWarn($"ExecuteReaderWithRetryAsync: Retry {attempt} due to {exception}.");
        });

    // Retry the following call according to the policy.
    await policy.ExecuteAsync<SqlDataReader>(async token =>
    {
        // This code is executed within the Policy

        if (conn.State != System.Data.ConnectionState.Open) await conn.OpenAsync(token);
        return await command.ExecuteReaderAsync(System.Data.CommandBehavior.Default, token);

    }, cancellationToken);
}
```

<span data-ttu-id="01d51-555">Cette méthode d’extension asynchrone est utilisable comme suit.</span><span class="sxs-lookup"><span data-stu-id="01d51-555">This asynchronous extension method can be used as follows.</span></span>

```csharp
var sqlCommand = sqlConnection.CreateCommand();
sqlCommand.CommandText = "[some query]";

using (var reader = await sqlCommand.ExecuteReaderWithRetryAsync())
{
    // Do something with the values
}
```

### <a name="more-information"></a><span data-ttu-id="01d51-556">Plus d’informations</span><span class="sxs-lookup"><span data-stu-id="01d51-556">More information</span></span>

- [<span data-ttu-id="01d51-557">Couche d’accès aux données de Cloud Service Fundamentals - Gestion des erreurs temporaires</span><span class="sxs-lookup"><span data-stu-id="01d51-557">Cloud Service Fundamentals Data Access Layer – Transient Fault Handling</span></span>](https://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)

<span data-ttu-id="01d51-558">Pour obtenir des conseils généraux sur le moyen de tirer le meilleur parti de SQL Database, consultez [Guide relatif à l’élasticité et aux performances d’Azure SQL Database](https://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).</span><span class="sxs-lookup"><span data-stu-id="01d51-558">For general guidance on getting the most from SQL Database, see [Azure SQL Database performance and elasticity guide](https://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).</span></span>

## <a name="sql-database-using-entity-framework-6"></a><span data-ttu-id="01d51-559">Base de données SQL utilisant Entity Framework 6</span><span class="sxs-lookup"><span data-stu-id="01d51-559">SQL Database using Entity Framework 6</span></span>

<span data-ttu-id="01d51-560">La base de données SQL est une base de données SQL hébergée disponible en différentes tailles et sous forme de service standard (partagé) et premium (non partagé).</span><span class="sxs-lookup"><span data-stu-id="01d51-560">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span> <span data-ttu-id="01d51-561">Entity Framework est un mappeur relationnel objet qui permet aux développeurs .NET de travailler avec des données relationnelles à l’aide d’objets spécifiques de domaine.</span><span class="sxs-lookup"><span data-stu-id="01d51-561">Entity Framework is an object-relational mapper that enables .NET developers to work with relational data using domain-specific objects.</span></span> <span data-ttu-id="01d51-562">Il élimine le recours à la plupart du code d’accès aux données que les développeurs doivent généralement écrire.</span><span class="sxs-lookup"><span data-stu-id="01d51-562">It eliminates the need for most of the data-access code that developers usually need to write.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="01d51-563">Mécanisme de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="01d51-563">Retry mechanism</span></span>

<span data-ttu-id="01d51-564">La prise en charge de la fonctionnalité de nouvelle tentative est assurée lors de l’accès à SQL Database à l’aide d’Entity Framework 6.0 et version supérieure par le biais d’un mécanisme appelé [résilience de connexion/logique de nouvelle tentative](/ef/ef6/fundamentals/connection-resiliency/retry-logic).</span><span class="sxs-lookup"><span data-stu-id="01d51-564">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher through a mechanism called [Connection resiliency / retry logic](/ef/ef6/fundamentals/connection-resiliency/retry-logic).</span></span> <span data-ttu-id="01d51-565">Les principales fonctionnalités de ce mécanisme sont :</span><span class="sxs-lookup"><span data-stu-id="01d51-565">The main features of the retry mechanism are:</span></span>

- <span data-ttu-id="01d51-566">L’abstraction principale est l’interface **IDbExecutionStrategy** .</span><span class="sxs-lookup"><span data-stu-id="01d51-566">The primary abstraction is the **IDbExecutionStrategy** interface.</span></span> <span data-ttu-id="01d51-567">Cette interface :</span><span class="sxs-lookup"><span data-stu-id="01d51-567">This interface:</span></span>
  - <span data-ttu-id="01d51-568">Définit les méthodes synchrones et asynchrones **Execute** \*.</span><span class="sxs-lookup"><span data-stu-id="01d51-568">Defines synchronous and asynchronous **Execute**\* methods.</span></span>
  - <span data-ttu-id="01d51-569">Définit les classes qui peuvent être utilisées directement ou configurées sur un contexte de base de données comme une stratégie par défaut mappée sur un nom de fournisseur ou un nom de fournisseur et un nom de serveur.</span><span class="sxs-lookup"><span data-stu-id="01d51-569">Defines classes that can be used directly or can be configured on a database context as a default strategy, mapped to provider name, or mapped to a provider name and server name.</span></span> <span data-ttu-id="01d51-570">En cas de configuration sur un contexte, les nouvelles tentatives se produisent au niveau des opérations de base de données individuelles. Il peut y en avoir plusieurs pour une opération de contexte donnée.</span><span class="sxs-lookup"><span data-stu-id="01d51-570">When configured on a context, retries occur at the level of individual database operations, of which there might be several for a given context operation.</span></span>
  - <span data-ttu-id="01d51-571">Définit à quel moment et comment proposer une nouvelle tentative en cas d’échec de connexion.</span><span class="sxs-lookup"><span data-stu-id="01d51-571">Defines when to retry a failed connection, and how.</span></span>
- <span data-ttu-id="01d51-572">Il inclut plusieurs implémentations intégrées de l’interface **IDbExecutionStrategy** :</span><span class="sxs-lookup"><span data-stu-id="01d51-572">It includes several built-in implementations of the **IDbExecutionStrategy** interface:</span></span>
  - <span data-ttu-id="01d51-573">Par défaut : aucune nouvelle tentative.</span><span class="sxs-lookup"><span data-stu-id="01d51-573">Default - no retrying.</span></span>
  - <span data-ttu-id="01d51-574">Par défaut pour la base de données SQL (automatique) : aucune nouvelle tentative, mais inspecte les exceptions et y inclut la suggestion d’utiliser la stratégie de base de données SQL.</span><span class="sxs-lookup"><span data-stu-id="01d51-574">Default for SQL Database (automatic) - no retrying, but inspects exceptions and wraps them with suggestion to use the SQL Database strategy.</span></span>
  - <span data-ttu-id="01d51-575">Par défaut pour la base de données SQL : exponentielle (héritée de la classe de base) et logique de détection de base de données SQL.</span><span class="sxs-lookup"><span data-stu-id="01d51-575">Default for SQL Database - exponential (inherited from base class) plus SQL Database detection logic.</span></span>
- <span data-ttu-id="01d51-576">Il implémente une stratégie de temporisation exponentielle qui inclut la répartition aléatoire.</span><span class="sxs-lookup"><span data-stu-id="01d51-576">It implements an exponential back-off strategy that includes randomization.</span></span>
- <span data-ttu-id="01d51-577">Les classes de nouvelle tentative intégrées sont avec état et ne sont pas thread-safe.</span><span class="sxs-lookup"><span data-stu-id="01d51-577">The built-in retry classes are stateful and are not thread safe.</span></span> <span data-ttu-id="01d51-578">Toutefois, elles peuvent être réutilisées une fois l’opération en cours terminée.</span><span class="sxs-lookup"><span data-stu-id="01d51-578">However, they can be reused after the current operation is completed.</span></span>
- <span data-ttu-id="01d51-579">Si le nombre de tentatives spécifié est dépassé, les résultats sont inclus dans une nouvelle exception.</span><span class="sxs-lookup"><span data-stu-id="01d51-579">If the specified retry count is exceeded, the results are wrapped in a new exception.</span></span> <span data-ttu-id="01d51-580">Il ne se propage pas dans l’exception en cours.</span><span class="sxs-lookup"><span data-stu-id="01d51-580">It does not bubble up the current exception.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="01d51-581">Configuration de la stratégie</span><span class="sxs-lookup"><span data-stu-id="01d51-581">Policy configuration</span></span>

<span data-ttu-id="01d51-582">La prise en charge de la nouvelle tentative est assurée lors de l’accès à la base de données SQL à l’aide d’Entity Framework 6.0 et versions ultérieures.</span><span class="sxs-lookup"><span data-stu-id="01d51-582">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher.</span></span> <span data-ttu-id="01d51-583">Les stratégies de nouvelle tentative sont configurées par programme.</span><span class="sxs-lookup"><span data-stu-id="01d51-583">Retry policies are configured programmatically.</span></span> <span data-ttu-id="01d51-584">La configuration ne peut pas être modifiée opération par opération.</span><span class="sxs-lookup"><span data-stu-id="01d51-584">The configuration cannot be changed on a per-operation basis.</span></span>

<span data-ttu-id="01d51-585">Lorsque vous configurez une stratégie sur le contexte comme stratégie par défaut, vous définissez une fonction qui crée une nouvelle stratégie à la demande.</span><span class="sxs-lookup"><span data-stu-id="01d51-585">When configuring a strategy on the context as the default, you specify a function that creates a new strategy on demand.</span></span> <span data-ttu-id="01d51-586">Le code suivant montre comment vous pouvez créer une classe de configuration de nouvelle tentative qui étend la classe de base **DbConfiguration** .</span><span class="sxs-lookup"><span data-stu-id="01d51-586">The following code shows how you can create a retry configuration class that extends the **DbConfiguration** base class.</span></span>

```csharp
public class BloggingContextConfiguration : DbConfiguration
{
  public BlogConfiguration()
  {
    // Set up the execution strategy for SQL Database (exponential) with 5 retries and 4 sec delay
    this.SetExecutionStrategy(
         "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4)));
  }
}
```

<span data-ttu-id="01d51-587">Vous pouvez alors la spécifier comme stratégie de nouvelle tentative par défaut pour toutes les opérations à l’aide de la méthode **SetConfiguration** de l’instance **DbConfiguration** lorsque l’application démarre.</span><span class="sxs-lookup"><span data-stu-id="01d51-587">You can then specify this as the default retry strategy for all operations using the **SetConfiguration** method of the **DbConfiguration** instance when the application starts.</span></span> <span data-ttu-id="01d51-588">Par défaut, Entity Framework détecte et utilise automatiquement la classe de configuration.</span><span class="sxs-lookup"><span data-stu-id="01d51-588">By default, EF will automatically discover and use the configuration class.</span></span>

```csharp
DbConfiguration.SetConfiguration(new BloggingContextConfiguration());
```

<span data-ttu-id="01d51-589">Vous pouvez spécifier la classe de configuration de nouvelle tentative pour un contexte en annotant la classe de contexte avec un attribut **DbConfigurationType** .</span><span class="sxs-lookup"><span data-stu-id="01d51-589">You can specify the retry configuration class for a context by annotating the context class with a **DbConfigurationType** attribute.</span></span> <span data-ttu-id="01d51-590">Toutefois, si vous avez une seule classe de configuration, Entity Framework l’utilisera sans avoir à annoter le contexte.</span><span class="sxs-lookup"><span data-stu-id="01d51-590">However, if you have only one configuration class, EF will use it without the need to annotate the context.</span></span>

```csharp
[DbConfigurationType(typeof(BloggingContextConfiguration))]
public class BloggingContext : DbContext
```

<span data-ttu-id="01d51-591">Si vous devez utiliser des stratégies de nouvelle tentative différentes pour des opérations spécifiques ou désactiver des nouvelles tentatives pour des opérations spécifiques, vous pouvez créer une classe de configuration qui vous permet de suspendre ou d’échanger des stratégies en définissant un indicateur dans le **CallContext**.</span><span class="sxs-lookup"><span data-stu-id="01d51-591">If you need to use different retry strategies for specific operations, or disable retries for specific operations, you can create a configuration class that allows you to suspend or swap strategies by setting a flag in the **CallContext**.</span></span> <span data-ttu-id="01d51-592">La classe de configuration peut utiliser cet indicateur pour changer de stratégie ou désactiver la stratégie indiquée et utilisée comme stratégie par défaut.</span><span class="sxs-lookup"><span data-stu-id="01d51-592">The configuration class can use this flag to switch strategies, or disable the strategy you provide and use a default strategy.</span></span> <span data-ttu-id="01d51-593">Pour plus d’informations, consultez [Suspendre la stratégie d’exécution](/ef/ef6/fundamentals/connection-resiliency/retry-logic#workaround-suspend-execution-strategy) (Entity Framework 6 et versions ultérieures).</span><span class="sxs-lookup"><span data-stu-id="01d51-593">For more information, see [Suspend Execution Strategy](/ef/ef6/fundamentals/connection-resiliency/retry-logic#workaround-suspend-execution-strategy) (EF6 onwards).</span></span>

<span data-ttu-id="01d51-594">Une autre technique d’utilisation de stratégies de nouvelle tentative spécifiques pour des opérations individuelles consiste à créer une instance de la classe de stratégie requise et à fournir la configuration requise au moyen de paramètres.</span><span class="sxs-lookup"><span data-stu-id="01d51-594">Another technique for using specific retry strategies for individual operations is to create an instance of the required strategy class and supply the desired settings through parameters.</span></span> <span data-ttu-id="01d51-595">Vous pouvez ensuite faire appel à sa méthode **ExecuteAsync** .</span><span class="sxs-lookup"><span data-stu-id="01d51-595">You then invoke its **ExecuteAsync** method.</span></span>

```csharp
var executionStrategy = new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4));
var blogs = await executionStrategy.ExecuteAsync(
    async () =>
    {
        using (var db = new BloggingContext("Blogs"))
        {
            // Acquire some values asynchronously and return them
        }
    },
    new CancellationToken()
);
```

<span data-ttu-id="01d51-596">La façon la plus simple d’utiliser une classe **DbConfiguration** consiste à la localiser dans le même assembly que la classe **DbContext**.</span><span class="sxs-lookup"><span data-stu-id="01d51-596">The simplest way to use a **DbConfiguration** class is to locate it in the same assembly as the **DbContext** class.</span></span> <span data-ttu-id="01d51-597">Toutefois, cela n’est pas approprié lorsque le même contexte est requis dans différents scénarios, comme dans le cas de stratégies de nouvelle tentative en arrière-plan et interactives.</span><span class="sxs-lookup"><span data-stu-id="01d51-597">However, this is not appropriate when the same context is required in different scenarios, such as different interactive and background retry strategies.</span></span> <span data-ttu-id="01d51-598">Si les différents contextes s’exécutent dans un AppDomain séparé, vous pouvez utiliser la prise en charge intégrée pour spécifier des classes de configuration dans le fichier de configuration ou la définir explicitement à l’aide de code.</span><span class="sxs-lookup"><span data-stu-id="01d51-598">If the different contexts execute in separate AppDomains, you can use the built-in support for specifying configuration classes in the configuration file or set it explicitly using code.</span></span> <span data-ttu-id="01d51-599">Si les différents contextes doivent s’exécuter dans le même AppDomain, une solution personnalisée sera nécessaire.</span><span class="sxs-lookup"><span data-stu-id="01d51-599">If the different contexts must execute in the same AppDomain, a custom solution will be required.</span></span>

<span data-ttu-id="01d51-600">Pour plus d’informations, consultez [Configuration basée sur le code](/ef/ef6/fundamentals/configuring/code-based) (Entity Framework 6 et versions supérieures).</span><span class="sxs-lookup"><span data-stu-id="01d51-600">For more information, see [Code-Based Configuration](/ef/ef6/fundamentals/configuring/code-based) (EF6 onwards).</span></span>

<span data-ttu-id="01d51-601">Le tableau suivant présente les paramètres par défaut pour la stratégie de nouvelle tentative intégrée lors de l’utilisation d’Entity Framework 6.</span><span class="sxs-lookup"><span data-stu-id="01d51-601">The following table shows the default settings for the built-in retry policy when using EF6.</span></span>

| <span data-ttu-id="01d51-602">Paramètre</span><span class="sxs-lookup"><span data-stu-id="01d51-602">Setting</span></span> | <span data-ttu-id="01d51-603">Valeur par défaut</span><span class="sxs-lookup"><span data-stu-id="01d51-603">Default value</span></span> | <span data-ttu-id="01d51-604">Signification</span><span class="sxs-lookup"><span data-stu-id="01d51-604">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="01d51-605">Stratégie</span><span class="sxs-lookup"><span data-stu-id="01d51-605">Policy</span></span> | <span data-ttu-id="01d51-606">Exponentielle</span><span class="sxs-lookup"><span data-stu-id="01d51-606">Exponential</span></span> | <span data-ttu-id="01d51-607">Temporisation exponentielle.</span><span class="sxs-lookup"><span data-stu-id="01d51-607">Exponential back-off.</span></span> |
| <span data-ttu-id="01d51-608">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="01d51-608">MaxRetryCount</span></span> | <span data-ttu-id="01d51-609">5.</span><span class="sxs-lookup"><span data-stu-id="01d51-609">5</span></span> | <span data-ttu-id="01d51-610">Nombre maximal de nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="01d51-610">The maximum number of retries.</span></span> |
| <span data-ttu-id="01d51-611">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="01d51-611">MaxDelay</span></span> | <span data-ttu-id="01d51-612">30 secondes</span><span class="sxs-lookup"><span data-stu-id="01d51-612">30 seconds</span></span> | <span data-ttu-id="01d51-613">Délai maximal entre les nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="01d51-613">The maximum delay between retries.</span></span> <span data-ttu-id="01d51-614">Cette valeur n’affecte pas le mode de calcul de la série de délais.</span><span class="sxs-lookup"><span data-stu-id="01d51-614">This value does not affect how the series of delays are computed.</span></span> <span data-ttu-id="01d51-615">Elle définit uniquement une limite supérieure.</span><span class="sxs-lookup"><span data-stu-id="01d51-615">It only defines an upper bound.</span></span> |
| <span data-ttu-id="01d51-616">DefaultCoefficient</span><span class="sxs-lookup"><span data-stu-id="01d51-616">DefaultCoefficient</span></span> | <span data-ttu-id="01d51-617">1 seconde</span><span class="sxs-lookup"><span data-stu-id="01d51-617">1 second</span></span> | <span data-ttu-id="01d51-618">Coefficient du calcul de temporisation exponentielle.</span><span class="sxs-lookup"><span data-stu-id="01d51-618">The coefficient for the exponential back-off computation.</span></span> <span data-ttu-id="01d51-619">Cette valeur ne peut pas être modifiée.</span><span class="sxs-lookup"><span data-stu-id="01d51-619">This value cannot be changed.</span></span> |
| <span data-ttu-id="01d51-620">DefaultRandomFactor</span><span class="sxs-lookup"><span data-stu-id="01d51-620">DefaultRandomFactor</span></span> | <span data-ttu-id="01d51-621">1.1</span><span class="sxs-lookup"><span data-stu-id="01d51-621">1.1</span></span> | <span data-ttu-id="01d51-622">Multiplicateur utilisé pour l’ajout d’un délai aléatoire pour chaque entrée.</span><span class="sxs-lookup"><span data-stu-id="01d51-622">The multiplier used to add a random delay for each entry.</span></span> <span data-ttu-id="01d51-623">Cette valeur ne peut pas être modifiée.</span><span class="sxs-lookup"><span data-stu-id="01d51-623">This value cannot be changed.</span></span> |
| <span data-ttu-id="01d51-624">DefaultExponentialBase</span><span class="sxs-lookup"><span data-stu-id="01d51-624">DefaultExponentialBase</span></span> | <span data-ttu-id="01d51-625">2</span><span class="sxs-lookup"><span data-stu-id="01d51-625">2</span></span> | <span data-ttu-id="01d51-626">Multiplicateur utilisé pour le calcul du délai suivant.</span><span class="sxs-lookup"><span data-stu-id="01d51-626">The multiplier used to calculate the next delay.</span></span> <span data-ttu-id="01d51-627">Cette valeur ne peut pas être modifiée.</span><span class="sxs-lookup"><span data-stu-id="01d51-627">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="01d51-628">Guide d’utilisation des nouvelles tentatives</span><span class="sxs-lookup"><span data-stu-id="01d51-628">Retry usage guidance</span></span>

<span data-ttu-id="01d51-629">Respectez les consignes suivantes lorsque vous accédez à la base de données SQL à l’aide d’Entity Framework 6 :</span><span class="sxs-lookup"><span data-stu-id="01d51-629">Consider the following guidelines when accessing SQL Database using EF6:</span></span>

- <span data-ttu-id="01d51-630">Choisissez l’option de service appropriée (partagée ou premium).</span><span class="sxs-lookup"><span data-stu-id="01d51-630">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="01d51-631">Une instance partagée peut être soumise à des délais de connexion plus longs que d’ordinaire et à des limitations en raison de l’utilisation du serveur partagé par d’autres locataires.</span><span class="sxs-lookup"><span data-stu-id="01d51-631">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="01d51-632">Si des performances prévisibles et des opérations fiables à faible latence sont requises, pensez à choisir l’option premium.</span><span class="sxs-lookup"><span data-stu-id="01d51-632">If predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>

- <span data-ttu-id="01d51-633">Une stratégie d’intervalle fixe n’est pas recommandée pour une utilisation avec la base de données SQL Azure.</span><span class="sxs-lookup"><span data-stu-id="01d51-633">A fixed interval strategy is not recommended for use with Azure SQL Database.</span></span> <span data-ttu-id="01d51-634">Utilisez plutôt une stratégie de temporisation exponentielle car le service peut être surchargé et des délais plus longs laissent davantage de temps de récupération.</span><span class="sxs-lookup"><span data-stu-id="01d51-634">Instead, use an exponential back-off strategy because the service may be overloaded, and longer delays allow more time for it to recover.</span></span>

- <span data-ttu-id="01d51-635">Choisissez une valeur appropriée pour les délais d’attente de connexion et de commande lors de la définition des connexions.</span><span class="sxs-lookup"><span data-stu-id="01d51-635">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="01d51-636">Basez le délai d’attente à la fois sur la conception de la logique métier et sur les tests.</span><span class="sxs-lookup"><span data-stu-id="01d51-636">Base the timeout on both your business logic design and through testing.</span></span> <span data-ttu-id="01d51-637">Vous devrez peut-être modifier cette valeur au fil du temps en fonction de l’évolution des volumes de données ou des processus d’entreprise.</span><span class="sxs-lookup"><span data-stu-id="01d51-637">You may need to modify this value over time as the volumes of data or the business processes change.</span></span> <span data-ttu-id="01d51-638">Un délai d’attente trop court peut entraîner des échecs de connexion prématurés lorsque la base de données est occupée.</span><span class="sxs-lookup"><span data-stu-id="01d51-638">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="01d51-639">Un délai d’attente trop long peut empêcher la logique de nouvelle tentative de fonctionner correctement en attendant trop longtemps avant la détection d’un échec de connexion.</span><span class="sxs-lookup"><span data-stu-id="01d51-639">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="01d51-640">La valeur d’expiration est un composant de la latence de bout en bout, bien que vous ne puissiez pas aisément déterminer le nombre de commandes qui seront exécutées lors de l’enregistrement du contexte.</span><span class="sxs-lookup"><span data-stu-id="01d51-640">The value of the timeout is a component of the end-to-end latency, although you cannot easily determine how many commands will execute when saving the context.</span></span> <span data-ttu-id="01d51-641">Vous pouvez modifier le délai d’attente par défaut en définissant la propriété **CommandTimeout** de l’instance **DbContext**.</span><span class="sxs-lookup"><span data-stu-id="01d51-641">You can change the default timeout by setting the **CommandTimeout** property of the **DbContext** instance.</span></span>

- <span data-ttu-id="01d51-642">Entity Framework prend en charge les configurations de nouvelle tentative définies dans les fichiers de configuration.</span><span class="sxs-lookup"><span data-stu-id="01d51-642">Entity Framework supports retry configurations defined in configuration files.</span></span> <span data-ttu-id="01d51-643">Toutefois, pour une flexibilité maximale sur Azure, vous devez envisager la création de la configuration par programme au sein de l’application.</span><span class="sxs-lookup"><span data-stu-id="01d51-643">However, for maximum flexibility on Azure you should consider creating the configuration programmatically within the application.</span></span> <span data-ttu-id="01d51-644">Les paramètres spécifiques pour les stratégies de nouvelle tentative, tels que le nombre de nouvelles tentatives et les intervalles, peuvent être stockés dans le fichier de configuration du service et être utilisés en cours d’exécution pour créer les stratégies appropriées.</span><span class="sxs-lookup"><span data-stu-id="01d51-644">The specific parameters for the retry policies, such as the number of retries and the retry intervals, can be stored in the service configuration file and used at runtime to create the appropriate policies.</span></span> <span data-ttu-id="01d51-645">Cela permet de modifier les paramètres sans avoir à redémarrer l’application.</span><span class="sxs-lookup"><span data-stu-id="01d51-645">This allows the settings to be changed without requiring the application to be restarted.</span></span>

<span data-ttu-id="01d51-646">Pensez à commencer par les paramètres ci-après pour les opérations liées aux nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="01d51-646">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="01d51-647">Vous ne pouvez pas spécifier le délai entre chaque nouvelle tentative (il est fixe et généré sous la forme d’une séquence exponentielle).</span><span class="sxs-lookup"><span data-stu-id="01d51-647">You cannot specify the delay between retry attempts (it is fixed and generated as an exponential sequence).</span></span> <span data-ttu-id="01d51-648">Vous ne pouvez spécifier que les valeurs maximales, comme indiqué ici, sauf si vous créez une stratégie de nouvelle tentative personnalisée.</span><span class="sxs-lookup"><span data-stu-id="01d51-648">You can specify only the maximum values, as shown here; unless you create a custom retry strategy.</span></span> <span data-ttu-id="01d51-649">Il s’agit de paramètres généraux. Vous devez par conséquent surveiller les opérations et optimiser les valeurs en fonction de votre propre scénario.</span><span class="sxs-lookup"><span data-stu-id="01d51-649">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="01d51-650">**Contexte**</span><span class="sxs-lookup"><span data-stu-id="01d51-650">**Context**</span></span> | <span data-ttu-id="01d51-651">**Exemple de cible E2E<br />latence max**</span><span class="sxs-lookup"><span data-stu-id="01d51-651">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="01d51-652">**Stratégie de nouvelle tentative**</span><span class="sxs-lookup"><span data-stu-id="01d51-652">**Retry policy**</span></span> | <span data-ttu-id="01d51-653">**Paramètres**</span><span class="sxs-lookup"><span data-stu-id="01d51-653">**Settings**</span></span> | <span data-ttu-id="01d51-654">**Valeurs**</span><span class="sxs-lookup"><span data-stu-id="01d51-654">**Values**</span></span> | <span data-ttu-id="01d51-655">**Fonctionnement**</span><span class="sxs-lookup"><span data-stu-id="01d51-655">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="01d51-656">Interactif, interface utilisateur,</span><span class="sxs-lookup"><span data-stu-id="01d51-656">Interactive, UI,</span></span><br /><span data-ttu-id="01d51-657">ou premier plan</span><span class="sxs-lookup"><span data-stu-id="01d51-657">or foreground</span></span> |<span data-ttu-id="01d51-658">2 secondes</span><span class="sxs-lookup"><span data-stu-id="01d51-658">2 seconds</span></span> |<span data-ttu-id="01d51-659">Exponentielle</span><span class="sxs-lookup"><span data-stu-id="01d51-659">Exponential</span></span> |<span data-ttu-id="01d51-660">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="01d51-660">MaxRetryCount</span></span><br /><span data-ttu-id="01d51-661">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="01d51-661">MaxDelay</span></span> |<span data-ttu-id="01d51-662">3</span><span class="sxs-lookup"><span data-stu-id="01d51-662">3</span></span><br /><span data-ttu-id="01d51-663">750 ms</span><span class="sxs-lookup"><span data-stu-id="01d51-663">750 ms</span></span> |<span data-ttu-id="01d51-664">Tentative 1 - délai 0 s</span><span class="sxs-lookup"><span data-stu-id="01d51-664">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="01d51-665">Tentative 2 - délai 750 ms</span><span class="sxs-lookup"><span data-stu-id="01d51-665">Attempt 2 - delay 750 ms</span></span><br /><span data-ttu-id="01d51-666">Tentative 3 - délai 750 ms</span><span class="sxs-lookup"><span data-stu-id="01d51-666">Attempt 3 – delay 750 ms</span></span> |
| <span data-ttu-id="01d51-667">Arrière-plan</span><span class="sxs-lookup"><span data-stu-id="01d51-667">Background</span></span><br /> <span data-ttu-id="01d51-668">ou lot</span><span class="sxs-lookup"><span data-stu-id="01d51-668">or batch</span></span> |<span data-ttu-id="01d51-669">30 secondes</span><span class="sxs-lookup"><span data-stu-id="01d51-669">30 seconds</span></span> |<span data-ttu-id="01d51-670">Exponentielle</span><span class="sxs-lookup"><span data-stu-id="01d51-670">Exponential</span></span> |<span data-ttu-id="01d51-671">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="01d51-671">MaxRetryCount</span></span><br /><span data-ttu-id="01d51-672">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="01d51-672">MaxDelay</span></span> |<span data-ttu-id="01d51-673">5.</span><span class="sxs-lookup"><span data-stu-id="01d51-673">5</span></span><br /><span data-ttu-id="01d51-674">12 secondes</span><span class="sxs-lookup"><span data-stu-id="01d51-674">12 seconds</span></span> |<span data-ttu-id="01d51-675">Tentative 1 - délai 0 s</span><span class="sxs-lookup"><span data-stu-id="01d51-675">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="01d51-676">Tentative 2 - délai ~1 s</span><span class="sxs-lookup"><span data-stu-id="01d51-676">Attempt 2 - delay ~1 sec</span></span><br /><span data-ttu-id="01d51-677">Tentative 3 - délai ~3 s</span><span class="sxs-lookup"><span data-stu-id="01d51-677">Attempt 3 - delay ~3 sec</span></span><br /><span data-ttu-id="01d51-678">Tentative 4 - délai ~7 s</span><span class="sxs-lookup"><span data-stu-id="01d51-678">Attempt 4 - delay ~7 sec</span></span><br /><span data-ttu-id="01d51-679">Tentative 5 - délai 12 s</span><span class="sxs-lookup"><span data-stu-id="01d51-679">Attempt 5 - delay 12 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="01d51-680">Les cibles de latence de bout en bout supposent le délai d’attente par défaut pour les connexions au service.</span><span class="sxs-lookup"><span data-stu-id="01d51-680">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="01d51-681">Si vous spécifiez des délais de connexion, la latence de bout en bout sera prolongée de ce temps supplémentaire pour toute nouvelle tentative.</span><span class="sxs-lookup"><span data-stu-id="01d51-681">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>

### <a name="examples"></a><span data-ttu-id="01d51-682">Exemples</span><span class="sxs-lookup"><span data-stu-id="01d51-682">Examples</span></span>

<span data-ttu-id="01d51-683">L’exemple de code suivant définit une solution d’accès aux données simple qui utilise Entity Framework.</span><span class="sxs-lookup"><span data-stu-id="01d51-683">The following code example defines a simple data access solution that uses Entity Framework.</span></span> <span data-ttu-id="01d51-684">Il spécifie une stratégie de nouvelle tentative spécifique par la définition de l’instance d’une classe nommée **BlogConfiguration** qui étend **DbConfiguration**.</span><span class="sxs-lookup"><span data-stu-id="01d51-684">It sets a specific retry strategy by defining an instance of a class named **BlogConfiguration** that extends **DbConfiguration**.</span></span>

```csharp
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Data.Entity.SqlServer;
using System.Threading.Tasks;

namespace RetryCodeSamples
{
    public class BlogConfiguration : DbConfiguration
    {
        public BlogConfiguration()
        {
            // Set up the execution strategy for SQL Database (exponential) with 5 retries and 12 sec delay.
            // These values could be loaded from configuration rather than being hard-coded.
            this.SetExecutionStrategy(
                    "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(12)));
        }
    }

    // Specify the configuration type if more than one has been defined.
    // [DbConfigurationType(typeof(BlogConfiguration))]
    public class BloggingContext : DbContext
    {
        // Definition of content goes here.
    }

    class EF6CodeSamples
    {
        public async static Task Samples()
        {
            // Execution strategy configured by DbConfiguration subclass, discovered automatically or
            // or explicitly indicated through configuration or with an attribute. Default is no retries.
            using (var db = new BloggingContext("Blogs"))
            {
                // Add, edit, delete blog items here, then:
                await db.SaveChangesAsync();
            }
        }
    }
}
```

<span data-ttu-id="01d51-685">Vous trouverez plusieurs exemples d’utilisation du mécanisme de nouvelle tentative d’Entity Framework dans [Résilience des connexions/logique de nouvelle tentative](/ef/ef6/fundamentals/connection-resiliency/retry-logic).</span><span class="sxs-lookup"><span data-stu-id="01d51-685">More examples of using the Entity Framework retry mechanism can be found in [Connection Resiliency / Retry Logic](/ef/ef6/fundamentals/connection-resiliency/retry-logic).</span></span>

### <a name="more-information"></a><span data-ttu-id="01d51-686">Plus d’informations</span><span class="sxs-lookup"><span data-stu-id="01d51-686">More information</span></span>

- [<span data-ttu-id="01d51-687">Guide relatif à l’élasticité et aux performances d’Azure SQL Database</span><span class="sxs-lookup"><span data-stu-id="01d51-687">Azure SQL Database performance and elasticity guide</span></span>](https://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)

## <a name="sql-database-using-entity-framework-core"></a><span data-ttu-id="01d51-688">Base de données SQL utilisant Entity Framework Core</span><span class="sxs-lookup"><span data-stu-id="01d51-688">SQL Database using Entity Framework Core</span></span>

<span data-ttu-id="01d51-689">[Entity Framework Core](/ef/core/) est un mappeur Objet Relationnel qui permet aux développeurs .NET Core de travailler avec des données à l’aide d’objets spécifiques du domaine.</span><span class="sxs-lookup"><span data-stu-id="01d51-689">[Entity Framework Core](/ef/core/) is an object-relational mapper that enables .NET Core developers to work with data using domain-specific objects.</span></span> <span data-ttu-id="01d51-690">Il élimine le recours à la plupart du code d’accès aux données que les développeurs doivent généralement écrire.</span><span class="sxs-lookup"><span data-stu-id="01d51-690">It eliminates the need for most of the data-access code that developers usually need to write.</span></span> <span data-ttu-id="01d51-691">Cette version d’Entity Framework a été écrite à partir de zéro et n’hérite pas automatiquement de toutes les fonctionnalités d’EF6.x.</span><span class="sxs-lookup"><span data-stu-id="01d51-691">This version of Entity Framework was written from the ground up, and doesn't automatically inherit all the features from EF6.x.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="01d51-692">Mécanisme de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="01d51-692">Retry mechanism</span></span>

<span data-ttu-id="01d51-693">La prise en charge du mécanisme de nouvelle tentative est assurée lors de l’accès à SQL Database utilisant Entity Framework Core par le biais d’un mécanisme appelé [résilience de connexion](/ef/core/miscellaneous/connection-resiliency).</span><span class="sxs-lookup"><span data-stu-id="01d51-693">Retry support is provided when accessing SQL Database using Entity Framework Core through a mechanism called [connection resiliency](/ef/core/miscellaneous/connection-resiliency).</span></span> <span data-ttu-id="01d51-694">La résilience de connexion a été introduite dans EF Core 1.1.0.</span><span class="sxs-lookup"><span data-stu-id="01d51-694">Connection resiliency was introduced in EF Core 1.1.0.</span></span>

<span data-ttu-id="01d51-695">L’abstraction principale est l’interface `IExecutionStrategy`.</span><span class="sxs-lookup"><span data-stu-id="01d51-695">The primary abstraction is the `IExecutionStrategy` interface.</span></span> <span data-ttu-id="01d51-696">La stratégie d’exécution de SQL Server, incluant SQL Azure, tient compte des types d’exceptions qui peuvent faire l’objet de nouvelles tentatives et présente des valeurs par défaut sensibles pour le nombre maximal de nouvelles tentatives, le délai entre deux tentatives, et ainsi de suite.</span><span class="sxs-lookup"><span data-stu-id="01d51-696">The execution strategy for SQL Server, including SQL Azure, is aware of the exception types that can be retried and has sensible defaults for maximum retries, delay between retries, and so on.</span></span>

### <a name="examples"></a><span data-ttu-id="01d51-697">Exemples</span><span class="sxs-lookup"><span data-stu-id="01d51-697">Examples</span></span>

<span data-ttu-id="01d51-698">Le code ci-après autorise les nouvelles tentatives automatiques lors de la configuration de l’objet DbContext, qui représente une session avec la base de données.</span><span class="sxs-lookup"><span data-stu-id="01d51-698">The following code enables automatic retries when configuring the DbContext object, which represents a session with the database.</span></span>

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(
            @"Server=(localdb)\mssqllocaldb;Database=EFMiscellanous.ConnectionResiliency;Trusted_Connection=True;",
            options => options.EnableRetryOnFailure());
}
```

<span data-ttu-id="01d51-699">Le code ci-après indique comment exécuter une transaction avec de nouvelles tentatives automatiques à l’aide d’une stratégie d’exécution.</span><span class="sxs-lookup"><span data-stu-id="01d51-699">The following code shows how to execute a transaction with automatic retries, by using an execution strategy.</span></span> <span data-ttu-id="01d51-700">La transaction est définie dans un délégué.</span><span class="sxs-lookup"><span data-stu-id="01d51-700">The transaction is defined in a delegate.</span></span> <span data-ttu-id="01d51-701">En cas d’échec passager, la stratégie d’exécution appelle de nouveau le délégué.</span><span class="sxs-lookup"><span data-stu-id="01d51-701">If a transient failure occurs, the execution strategy will invoke the delegate again.</span></span>

```csharp
using (var db = new BloggingContext())
{
    var strategy = db.Database.CreateExecutionStrategy();

    strategy.Execute(() =>
    {
        using (var transaction = db.Database.BeginTransaction())
        {
            db.Blogs.Add(new Blog { Url = "https://blogs.msdn.com/dotnet" });
            db.SaveChanges();

            db.Blogs.Add(new Blog { Url = "https://blogs.msdn.com/visualstudio" });
            db.SaveChanges();

            transaction.Commit();
        }
    });
}
```

## <a name="azure-storage"></a><span data-ttu-id="01d51-702">Stockage Azure</span><span class="sxs-lookup"><span data-stu-id="01d51-702">Azure Storage</span></span>

<span data-ttu-id="01d51-703">Les services de stockage Azure incluent le stockage des tables et des objets blob, les fichiers et les files d’attente de stockage.</span><span class="sxs-lookup"><span data-stu-id="01d51-703">Azure Storage services include table and blob storage, files, and storage queues.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="01d51-704">Mécanisme de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="01d51-704">Retry mechanism</span></span>

<span data-ttu-id="01d51-705">Les nouvelles tentatives se produisent au niveau de chaque opération REST et font partie intégrante de l’implémentation de l’API client.</span><span class="sxs-lookup"><span data-stu-id="01d51-705">Retries occur at the individual REST operation level and are an integral part of the client API implementation.</span></span> <span data-ttu-id="01d51-706">Le kit de développement logiciel (SDK) de stockage client utilise des classes qui implémentent l’ [interface IExtendedRetryPolicy](/dotnet/api/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy).</span><span class="sxs-lookup"><span data-stu-id="01d51-706">The client storage SDK uses classes that implement the [IExtendedRetryPolicy Interface](/dotnet/api/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy).</span></span>

<span data-ttu-id="01d51-707">Il existe différentes implémentations de l’interface.</span><span class="sxs-lookup"><span data-stu-id="01d51-707">There are different implementations of the interface.</span></span> <span data-ttu-id="01d51-708">Les clients de stockage peuvent choisir parmi les stratégies spécialement conçues pour l’accès à des tables, des objets BLOB et des files d’attente.</span><span class="sxs-lookup"><span data-stu-id="01d51-708">Storage clients can choose from policies specifically designed for accessing tables, blobs, and queues.</span></span> <span data-ttu-id="01d51-709">Chaque implémentation utilise une stratégie de nouvelle tentative différente qui définit essentiellement l’intervalle avant une nouvelle tentative et d’autres détails.</span><span class="sxs-lookup"><span data-stu-id="01d51-709">Each implementation uses a different retry strategy that essentially defines the retry interval and other details.</span></span>

<span data-ttu-id="01d51-710">Les classes intégrées prennent en charge la stratégie linéaire (délai constant) et exponentielle avec répartition aléatoire des intervalles entre chaque nouvelle tentative.</span><span class="sxs-lookup"><span data-stu-id="01d51-710">The built-in classes provide support for linear (constant delay) and exponential with randomization retry intervals.</span></span> <span data-ttu-id="01d51-711">Il n’existe également aucune stratégie de nouvelle tentative à utiliser lorsqu’un autre processus gère les nouvelles tentatives à un niveau supérieur.</span><span class="sxs-lookup"><span data-stu-id="01d51-711">There is also a no retry policy for use when another process is handling retries at a higher level.</span></span> <span data-ttu-id="01d51-712">Toutefois, vous pouvez implémenter vos propres classes de nouvelle tentative si vous avez des exigences spécifiques non fournies par les classes intégrées.</span><span class="sxs-lookup"><span data-stu-id="01d51-712">However, you can implement your own retry classes if you have specific requirements not provided by the built-in classes.</span></span>

<span data-ttu-id="01d51-713">Les autres alternatives passent d’un emplacement de service de stockage principal à un emplacement de service de stockage secondaire si vous utilisez le stockage géo-redondant avec accès en lecture (RA-GRS) et que le résultat de la demande est une erreur renouvelable.</span><span class="sxs-lookup"><span data-stu-id="01d51-713">Alternate retries switch between primary and secondary storage service location if you are using read access geo-redundant storage (RA-GRS) and the result of the request is a retryable error.</span></span> <span data-ttu-id="01d51-714">Pour plus d’informations, consultez [Options de redondance du stockage Azure](/azure/storage/common/storage-redundancy) .</span><span class="sxs-lookup"><span data-stu-id="01d51-714">See [Azure Storage Redundancy Options](/azure/storage/common/storage-redundancy) for more information.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="01d51-715">Configuration de la stratégie</span><span class="sxs-lookup"><span data-stu-id="01d51-715">Policy configuration</span></span>

<span data-ttu-id="01d51-716">Les stratégies de nouvelle tentative sont configurées par programme.</span><span class="sxs-lookup"><span data-stu-id="01d51-716">Retry policies are configured programmatically.</span></span> <span data-ttu-id="01d51-717">Une procédure classique consiste à créer et remplir une instance **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions** ou **QueueRequestOptions**.</span><span class="sxs-lookup"><span data-stu-id="01d51-717">A typical procedure is to create and populate a **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions**, or **QueueRequestOptions** instance.</span></span>

```csharp
TableRequestOptions interactiveRequestOption = new TableRequestOptions()
{
  RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
  // For Read-access geo-redundant storage, use PrimaryThenSecondary.
  // Otherwise set this to PrimaryOnly.
  LocationMode = LocationMode.PrimaryThenSecondary,
  // Maximum execution time based on the business use case.
  MaximumExecutionTime = TimeSpan.FromSeconds(2)
};
```

<span data-ttu-id="01d51-718">L’instance d’options de demande peut ensuite être définie sur le client et toutes les opérations avec le client utiliseront les options de demande spécifiées.</span><span class="sxs-lookup"><span data-stu-id="01d51-718">The request options instance can then be set on the client, and all operations with the client will use the specified request options.</span></span>

```csharp
client.DefaultRequestOptions = interactiveRequestOption;
var stats = await client.GetServiceStatsAsync();
```

<span data-ttu-id="01d51-719">Vous pouvez remplacer les options de demande client en transmettant une instance remplie de la classe d’options de demande en tant que paramètre aux méthodes d’opération.</span><span class="sxs-lookup"><span data-stu-id="01d51-719">You can override the client request options by passing a populated instance of the request options class as a parameter to operation methods.</span></span>

```csharp
var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
```

<span data-ttu-id="01d51-720">Vous utilisez une instance **OperationContext** pour spécifier le code à exécuter lorsqu’une nouvelle tentative se produit et qu’une opération est terminée.</span><span class="sxs-lookup"><span data-stu-id="01d51-720">You use an **OperationContext** instance to specify the code to execute when a retry occurs and when an operation has completed.</span></span> <span data-ttu-id="01d51-721">Ce code peut collecter des informations sur l’opération à utiliser dans les journaux d’activité et la télémétrie</span><span class="sxs-lookup"><span data-stu-id="01d51-721">This code can collect information about the operation for use in logs and telemetry.</span></span>

```csharp
// Set up notifications for an operation
var context = new OperationContext();
context.ClientRequestID = "some request id";
context.Retrying += (sender, args) =>
{
    /* Collect retry information */
};
context.RequestCompleted += (sender, args) =>
{
    /* Collect operation completion information */
};
var stats = await client.GetServiceStatsAsync(null, context);
```

<span data-ttu-id="01d51-722">En plus d’indiquer si un échec donne droit à une nouvelle tentative, les stratégies de nouvelle tentative étendues renvoient un objet **RetryContext** indiquant le nombre de nouvelles tentatives, les résultats de la dernière demande, et si la tentative suivante se déroulera dans l’emplacement principal ou secondaire (voir le tableau ci-dessous pour plus d’informations).</span><span class="sxs-lookup"><span data-stu-id="01d51-722">In addition to indicating whether a failure is suitable for retry, the extended retry policies return a **RetryContext** object that indicates the number of retries, the results of the last request, whether the next retry will happen in the primary or secondary location (see table below for details).</span></span> <span data-ttu-id="01d51-723">Les propriétés de l’objet **RetryContext** peuvent être utilisées pour décider de l’éventualité d’une nouvelle tentative et du moment où elle doit se dérouler.</span><span class="sxs-lookup"><span data-stu-id="01d51-723">The properties of the **RetryContext** object can be used to decide if and when to attempt a retry.</span></span> <span data-ttu-id="01d51-724">Pour plus d’informations, reportez-vous à la [méthode IExtendedRetryPolicy.Evaluate](/dotnet/api/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate).</span><span class="sxs-lookup"><span data-stu-id="01d51-724">For more details, see [IExtendedRetryPolicy.Evaluate Method](/dotnet/api/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate).</span></span>

<span data-ttu-id="01d51-725">Les tableaux ci-après présentent les paramètres par défaut pour les stratégies de nouvelle tentative intégrées.</span><span class="sxs-lookup"><span data-stu-id="01d51-725">The following tables show the default settings for the built-in retry policies.</span></span>

<span data-ttu-id="01d51-726">**Options de requête :**</span><span class="sxs-lookup"><span data-stu-id="01d51-726">**Request options:**</span></span>

| <span data-ttu-id="01d51-727">**Paramètre**</span><span class="sxs-lookup"><span data-stu-id="01d51-727">**Setting**</span></span> | <span data-ttu-id="01d51-728">**Valeur par défaut**</span><span class="sxs-lookup"><span data-stu-id="01d51-728">**Default value**</span></span> | <span data-ttu-id="01d51-729">**Signification**</span><span class="sxs-lookup"><span data-stu-id="01d51-729">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="01d51-730">MaximumExecutionTime</span><span class="sxs-lookup"><span data-stu-id="01d51-730">MaximumExecutionTime</span></span> | <span data-ttu-id="01d51-731">Aucun</span><span class="sxs-lookup"><span data-stu-id="01d51-731">None</span></span> | <span data-ttu-id="01d51-732">Durée d’exécution maximale pour la demande, y compris toutes les nouvelles tentatives potentielles.</span><span class="sxs-lookup"><span data-stu-id="01d51-732">Maximum execution time for the request, including all potential retry attempts.</span></span> <span data-ttu-id="01d51-733">Si elle n’est pas indiquée, la durée d’exécution autorisée d’une demande est illimitée.</span><span class="sxs-lookup"><span data-stu-id="01d51-733">If it is not specified, then the amount of time that a request is permitted to take is unlimited.</span></span> <span data-ttu-id="01d51-734">En d’autres termes, la demande peut se bloquer.</span><span class="sxs-lookup"><span data-stu-id="01d51-734">In other words, the request might hang.</span></span> |
| <span data-ttu-id="01d51-735">ServerTimeout</span><span class="sxs-lookup"><span data-stu-id="01d51-735">ServerTimeout</span></span> | <span data-ttu-id="01d51-736">Aucun</span><span class="sxs-lookup"><span data-stu-id="01d51-736">None</span></span> | <span data-ttu-id="01d51-737">Intervalle du délai d’attente du serveur pour la demande (la valeur est arrondie aux secondes).</span><span class="sxs-lookup"><span data-stu-id="01d51-737">Server timeout interval for the request (value is rounded to seconds).</span></span> <span data-ttu-id="01d51-738">Si non spécifié, il utilisera la valeur par défaut pour toutes les demandes au serveur.</span><span class="sxs-lookup"><span data-stu-id="01d51-738">If not specified, it will use the default value for all requests to the server.</span></span> <span data-ttu-id="01d51-739">En règle générale, la meilleure option consiste à omettre ce paramètre afin que le serveur par défaut soit utilisé.</span><span class="sxs-lookup"><span data-stu-id="01d51-739">Usually, the best option is to omit this setting so that the server default is used.</span></span> |
| <span data-ttu-id="01d51-740">LocationMode</span><span class="sxs-lookup"><span data-stu-id="01d51-740">LocationMode</span></span> | <span data-ttu-id="01d51-741">Aucun</span><span class="sxs-lookup"><span data-stu-id="01d51-741">None</span></span> | <span data-ttu-id="01d51-742">Si le compte de stockage est créé avec l’option de duplication du stockage géo-redondant avec accès en lecture (RA-GRS), vous pouvez utiliser le mode de d’emplacement pour indiquer l’emplacement devant recevoir la demande.</span><span class="sxs-lookup"><span data-stu-id="01d51-742">If the storage account is created with the Read access geo-redundant storage (RA-GRS) replication option, you can use the location mode to indicate which location should receive the request.</span></span> <span data-ttu-id="01d51-743">Par exemple, si **PrimaryThenSecondary** est spécifié, les demandes sont dans un premier temps toujours envoyées vers l’emplacement principal.</span><span class="sxs-lookup"><span data-stu-id="01d51-743">For example, if **PrimaryThenSecondary** is specified, requests are always sent to the primary location first.</span></span> <span data-ttu-id="01d51-744">En cas d’échec, la demande est envoyée vers l’emplacement secondaire.</span><span class="sxs-lookup"><span data-stu-id="01d51-744">If a request fails, it is sent to the secondary location.</span></span> |
| <span data-ttu-id="01d51-745">RetryPolicy</span><span class="sxs-lookup"><span data-stu-id="01d51-745">RetryPolicy</span></span> | <span data-ttu-id="01d51-746">ExponentialPolicy</span><span class="sxs-lookup"><span data-stu-id="01d51-746">ExponentialPolicy</span></span> | <span data-ttu-id="01d51-747">Voir ci-dessous pour obtenir plus d’informations sur chaque option.</span><span class="sxs-lookup"><span data-stu-id="01d51-747">See below for details of each option.</span></span> |

<span data-ttu-id="01d51-748">**Stratégie exponentielle :**</span><span class="sxs-lookup"><span data-stu-id="01d51-748">**Exponential policy:**</span></span>

| <span data-ttu-id="01d51-749">**Paramètre**</span><span class="sxs-lookup"><span data-stu-id="01d51-749">**Setting**</span></span> | <span data-ttu-id="01d51-750">**Valeur par défaut**</span><span class="sxs-lookup"><span data-stu-id="01d51-750">**Default value**</span></span> | <span data-ttu-id="01d51-751">**Signification**</span><span class="sxs-lookup"><span data-stu-id="01d51-751">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="01d51-752">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="01d51-752">maxAttempt</span></span> | <span data-ttu-id="01d51-753">3</span><span class="sxs-lookup"><span data-stu-id="01d51-753">3</span></span> | <span data-ttu-id="01d51-754">Nombre de nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="01d51-754">Number of retry attempts.</span></span> |
| <span data-ttu-id="01d51-755">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="01d51-755">deltaBackoff</span></span> | <span data-ttu-id="01d51-756">4 secondes</span><span class="sxs-lookup"><span data-stu-id="01d51-756">4 seconds</span></span> | <span data-ttu-id="01d51-757">Intervalle de temporisation entre les tentatives.</span><span class="sxs-lookup"><span data-stu-id="01d51-757">Back-off interval between retries.</span></span> <span data-ttu-id="01d51-758">Multiples de ce laps de temps, y compris un élément aléatoire, seront utilisés pour les tentatives suivantes.</span><span class="sxs-lookup"><span data-stu-id="01d51-758">Multiples of this timespan, including a random element, will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="01d51-759">MinBackoff</span><span class="sxs-lookup"><span data-stu-id="01d51-759">MinBackoff</span></span> | <span data-ttu-id="01d51-760">3 secondes</span><span class="sxs-lookup"><span data-stu-id="01d51-760">3 seconds</span></span> | <span data-ttu-id="01d51-761">Ajouté à tous les intervalles des tentatives calculés à partir de deltaBackoff.</span><span class="sxs-lookup"><span data-stu-id="01d51-761">Added to all retry intervals computed from deltaBackoff.</span></span> <span data-ttu-id="01d51-762">Cette valeur ne peut pas être modifiée.</span><span class="sxs-lookup"><span data-stu-id="01d51-762">This value cannot be changed.</span></span>
| <span data-ttu-id="01d51-763">MaxBackoff</span><span class="sxs-lookup"><span data-stu-id="01d51-763">MaxBackoff</span></span> | <span data-ttu-id="01d51-764">120 secondes</span><span class="sxs-lookup"><span data-stu-id="01d51-764">120 seconds</span></span> | <span data-ttu-id="01d51-765">MaxBackoff est utilisé si l’intervalle des tentatives calculé est supérieur à MaxBackoff.</span><span class="sxs-lookup"><span data-stu-id="01d51-765">MaxBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> <span data-ttu-id="01d51-766">Cette valeur ne peut pas être modifiée.</span><span class="sxs-lookup"><span data-stu-id="01d51-766">This value cannot be changed.</span></span> |

<span data-ttu-id="01d51-767">**Stratégie linéaire :**</span><span class="sxs-lookup"><span data-stu-id="01d51-767">**Linear policy:**</span></span>

| <span data-ttu-id="01d51-768">**Paramètre**</span><span class="sxs-lookup"><span data-stu-id="01d51-768">**Setting**</span></span> | <span data-ttu-id="01d51-769">**Valeur par défaut**</span><span class="sxs-lookup"><span data-stu-id="01d51-769">**Default value**</span></span> | <span data-ttu-id="01d51-770">**Signification**</span><span class="sxs-lookup"><span data-stu-id="01d51-770">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="01d51-771">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="01d51-771">maxAttempt</span></span> | <span data-ttu-id="01d51-772">3</span><span class="sxs-lookup"><span data-stu-id="01d51-772">3</span></span> | <span data-ttu-id="01d51-773">Nombre de nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="01d51-773">Number of retry attempts.</span></span> |
| <span data-ttu-id="01d51-774">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="01d51-774">deltaBackoff</span></span> | <span data-ttu-id="01d51-775">30 secondes</span><span class="sxs-lookup"><span data-stu-id="01d51-775">30 seconds</span></span> | <span data-ttu-id="01d51-776">Intervalle de temporisation entre les tentatives.</span><span class="sxs-lookup"><span data-stu-id="01d51-776">Back-off interval between retries.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="01d51-777">Guide d’utilisation des nouvelles tentatives</span><span class="sxs-lookup"><span data-stu-id="01d51-777">Retry usage guidance</span></span>

<span data-ttu-id="01d51-778">Respectez les consignes suivantes lorsque vous accédez aux services de stockage Azure à l’aide de l’API client de stockage :</span><span class="sxs-lookup"><span data-stu-id="01d51-778">Consider the following guidelines when accessing Azure storage services using the storage client API:</span></span>

- <span data-ttu-id="01d51-779">Utilisez les stratégies de nouvelle tentative intégrées de l’espace de noms Microsoft.WindowsAzure.Storage.RetryPolicies, lorsqu’elles sont adaptées à vos besoins.</span><span class="sxs-lookup"><span data-stu-id="01d51-779">Use the built-in retry policies from the Microsoft.WindowsAzure.Storage.RetryPolicies namespace where they are appropriate for your requirements.</span></span> <span data-ttu-id="01d51-780">Dans la plupart des cas, ces stratégies suffisent.</span><span class="sxs-lookup"><span data-stu-id="01d51-780">In most cases, these policies will be sufficient.</span></span>

- <span data-ttu-id="01d51-781">Utilisez la stratégie **ExponentialRetry** dans les opérations de traitement par lots, les tâches en arrière-plan ou les scénarios non interactifs.</span><span class="sxs-lookup"><span data-stu-id="01d51-781">Use the **ExponentialRetry** policy in batch operations, background tasks, or non-interactive scenarios.</span></span> <span data-ttu-id="01d51-782">Dans ces scénarios, vous pouvez généralement accorder plus de temps pour le service à récupérer &mdash; avec conséquent accroît les chances de l’opération de réussite.</span><span class="sxs-lookup"><span data-stu-id="01d51-782">In these scenarios, you can typically allow more time for the service to recover &mdash; with a consequently increased chance of the operation eventually succeeding.</span></span>

- <span data-ttu-id="01d51-783">Vous pouvez spécifier la propriété **MaximumExecutionTime** du paramètre **RequestOptions** pour limiter la durée d’exécution totale. Prenez en compte le type et la taille de l’opération lorsque vous choisissez une valeur de délai d’attente.</span><span class="sxs-lookup"><span data-stu-id="01d51-783">Consider specifying the **MaximumExecutionTime** property of the **RequestOptions** parameter to limit the total execution time, but take into account the type and size of the operation when choosing a timeout value.</span></span>

- <span data-ttu-id="01d51-784">Si vous avez besoin d’implémenter une nouvelle tentative personnalisée, évitez de créer des wrappers autour des classes de client de stockage.</span><span class="sxs-lookup"><span data-stu-id="01d51-784">If you need to implement a custom retry, avoid creating wrappers around the storage client classes.</span></span> <span data-ttu-id="01d51-785">Utilisez plutôt les fonctionnalités permettant d’étendre les stratégies existantes via l’interface **IExtendedRetryPolicy** .</span><span class="sxs-lookup"><span data-stu-id="01d51-785">Instead, use the capabilities to extend the existing policies through the **IExtendedRetryPolicy** interface.</span></span>

- <span data-ttu-id="01d51-786">Si vous utilisez le stockage géo-redondant avec accès en lecture (RA-GRS), vous pouvez utiliser **LocationMode** pour indiquer que les nouvelles tentatives accéderont à la copie en lecture seule secondaire du magasin en cas d’échec de l’accès principal.</span><span class="sxs-lookup"><span data-stu-id="01d51-786">If you are using read access geo-redundant storage (RA-GRS) you can use the **LocationMode** to specify that retry attempts will access the secondary read-only copy of the store should the primary access fail.</span></span> <span data-ttu-id="01d51-787">Toutefois, lorsque vous utilisez cette option vous devez vous assurer que votre application peut fonctionner correctement avec des données qui peuvent être obsolètes si la réplication à partir du magasin principal n’est pas encore terminée.</span><span class="sxs-lookup"><span data-stu-id="01d51-787">However, when using this option you must ensure that your application can work successfully with data that may be stale if the replication from the primary store has not yet completed.</span></span>

<span data-ttu-id="01d51-788">Pensez à commencer par les paramètres suivants pour les opérations liées aux nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="01d51-788">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="01d51-789">Il s’agit de paramètres généraux. Vous devez par conséquent surveiller les opérations et optimiser les valeurs en fonction de votre propre scénario.</span><span class="sxs-lookup"><span data-stu-id="01d51-789">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="01d51-790">**Contexte**</span><span class="sxs-lookup"><span data-stu-id="01d51-790">**Context**</span></span> | <span data-ttu-id="01d51-791">**Exemple de cible E2E<br />latence max**</span><span class="sxs-lookup"><span data-stu-id="01d51-791">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="01d51-792">**Stratégie de nouvelle tentative**</span><span class="sxs-lookup"><span data-stu-id="01d51-792">**Retry policy**</span></span> | <span data-ttu-id="01d51-793">**Paramètres**</span><span class="sxs-lookup"><span data-stu-id="01d51-793">**Settings**</span></span> | <span data-ttu-id="01d51-794">**Valeurs**</span><span class="sxs-lookup"><span data-stu-id="01d51-794">**Values**</span></span> | <span data-ttu-id="01d51-795">**Fonctionnement**</span><span class="sxs-lookup"><span data-stu-id="01d51-795">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="01d51-796">Interactif, interface utilisateur,</span><span class="sxs-lookup"><span data-stu-id="01d51-796">Interactive, UI,</span></span><br /><span data-ttu-id="01d51-797">ou premier plan</span><span class="sxs-lookup"><span data-stu-id="01d51-797">or foreground</span></span> |<span data-ttu-id="01d51-798">2 secondes</span><span class="sxs-lookup"><span data-stu-id="01d51-798">2 seconds</span></span> |<span data-ttu-id="01d51-799">Linéaire</span><span class="sxs-lookup"><span data-stu-id="01d51-799">Linear</span></span> |<span data-ttu-id="01d51-800">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="01d51-800">maxAttempt</span></span><br /><span data-ttu-id="01d51-801">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="01d51-801">deltaBackoff</span></span> |<span data-ttu-id="01d51-802">3</span><span class="sxs-lookup"><span data-stu-id="01d51-802">3</span></span><br /><span data-ttu-id="01d51-803">500 ms</span><span class="sxs-lookup"><span data-stu-id="01d51-803">500 ms</span></span> |<span data-ttu-id="01d51-804">Tentative 1 - délai 500 ms</span><span class="sxs-lookup"><span data-stu-id="01d51-804">Attempt 1 - delay 500 ms</span></span><br /><span data-ttu-id="01d51-805">Tentative 2 - délai 500 ms</span><span class="sxs-lookup"><span data-stu-id="01d51-805">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="01d51-806">Tentative 3 - délai 500 ms</span><span class="sxs-lookup"><span data-stu-id="01d51-806">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="01d51-807">Arrière-plan</span><span class="sxs-lookup"><span data-stu-id="01d51-807">Background</span></span><br /><span data-ttu-id="01d51-808">ou lot</span><span class="sxs-lookup"><span data-stu-id="01d51-808">or batch</span></span> |<span data-ttu-id="01d51-809">30 secondes</span><span class="sxs-lookup"><span data-stu-id="01d51-809">30 seconds</span></span> |<span data-ttu-id="01d51-810">Exponentielle</span><span class="sxs-lookup"><span data-stu-id="01d51-810">Exponential</span></span> |<span data-ttu-id="01d51-811">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="01d51-811">maxAttempt</span></span><br /><span data-ttu-id="01d51-812">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="01d51-812">deltaBackoff</span></span> |<span data-ttu-id="01d51-813">5.</span><span class="sxs-lookup"><span data-stu-id="01d51-813">5</span></span><br /><span data-ttu-id="01d51-814">4 secondes</span><span class="sxs-lookup"><span data-stu-id="01d51-814">4 seconds</span></span> |<span data-ttu-id="01d51-815">Tentative 1 - délai ~3 sec</span><span class="sxs-lookup"><span data-stu-id="01d51-815">Attempt 1 - delay ~3 sec</span></span><br /><span data-ttu-id="01d51-816">Tentative 2 - délai ~7 sec</span><span class="sxs-lookup"><span data-stu-id="01d51-816">Attempt 2 - delay ~7 sec</span></span><br /><span data-ttu-id="01d51-817">Tentative 3 - délai ~15 sec</span><span class="sxs-lookup"><span data-stu-id="01d51-817">Attempt 3 - delay ~15 sec</span></span> |

### <a name="telemetry"></a><span data-ttu-id="01d51-818">Télémétrie</span><span class="sxs-lookup"><span data-stu-id="01d51-818">Telemetry</span></span>

<span data-ttu-id="01d51-819">Les nouvelles tentatives sont enregistrées dans un **TraceSource**.</span><span class="sxs-lookup"><span data-stu-id="01d51-819">Retry attempts are logged to a **TraceSource**.</span></span> <span data-ttu-id="01d51-820">Vous devez configurer un **TraceListener** pour capturer les événements et les écrire dans un journal de destination approprié.</span><span class="sxs-lookup"><span data-stu-id="01d51-820">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span> <span data-ttu-id="01d51-821">Vous pouvez utiliser le **TextWriterTraceListener** ou **XmlWriterTraceListener** pour écrire les données dans un fichier journal, le **EventLogTraceListener** pour écrire dans le journal des événements Windows, ou le **EventProviderTraceListener** pour écrire les données de trace dans le sous-système ETW.</span><span class="sxs-lookup"><span data-stu-id="01d51-821">You can use the **TextWriterTraceListener** or **XmlWriterTraceListener** to write the data to a log file, the **EventLogTraceListener** to write to the Windows Event Log, or the **EventProviderTraceListener** to write trace data to the ETW subsystem.</span></span> <span data-ttu-id="01d51-822">Vous pouvez également configurer le vidage automatique de la mémoire tampon et le niveau de détail des événements qui seront enregistrés (par exemple, erreur, avertissement, information et informations détaillées).</span><span class="sxs-lookup"><span data-stu-id="01d51-822">You can also configure auto-flushing of the buffer, and the verbosity of events that will be logged (for example, Error, Warning, Informational, and Verbose).</span></span> <span data-ttu-id="01d51-823">Pour plus d’informations, consultez [Journalisation côté client avec la bibliothèque cliente de stockage .NET](/rest/api/storageservices/Client-side-Logging-with-the-.NET-Storage-Client-Library).</span><span class="sxs-lookup"><span data-stu-id="01d51-823">For more information, see [Client-side Logging with the .NET Storage Client Library](/rest/api/storageservices/Client-side-Logging-with-the-.NET-Storage-Client-Library).</span></span>

<span data-ttu-id="01d51-824">Les opérations peuvent recevoir une instance **OperationContext**, qui expose un événement **Retrying** pouvant être utilisé pour associer la logique personnalisée de télémétrie.</span><span class="sxs-lookup"><span data-stu-id="01d51-824">Operations can receive an **OperationContext** instance, which exposes a **Retrying** event that can be used to attach custom telemetry logic.</span></span> <span data-ttu-id="01d51-825">Pour plus d’informations, consultez [OperationContext.Retrying Event](/dotnet/api/microsoft.windowsazure.storage.operationcontext.retrying).</span><span class="sxs-lookup"><span data-stu-id="01d51-825">For more information, see [OperationContext.Retrying Event](/dotnet/api/microsoft.windowsazure.storage.operationcontext.retrying).</span></span>

### <a name="examples"></a><span data-ttu-id="01d51-826">Exemples</span><span class="sxs-lookup"><span data-stu-id="01d51-826">Examples</span></span>

<span data-ttu-id="01d51-827">L’exemple de code suivant montre comment créer deux instances **TableRequestOptions** avec des paramètres de nouvelle tentative différents ; une pour les demandes interactives et une pour les demandes d’arrière-plan.</span><span class="sxs-lookup"><span data-stu-id="01d51-827">The following code example shows how to create two **TableRequestOptions** instances with different retry settings; one for interactive requests and one for background requests.</span></span> <span data-ttu-id="01d51-828">L’exemple définit ensuite ces deux jeux stratégies de nouvelle tentative sur le client de sorte qu’elles s’appliquent à toutes les demandes et définit également la stratégie interactive sur une demande spécifique afin qu’elle remplace les paramètres par défaut appliqués au client.</span><span class="sxs-lookup"><span data-stu-id="01d51-828">The example then sets these two retry policies on the client so that they apply for all requests, and also sets the interactive strategy on a specific request so that it overrides the default settings applied to the client.</span></span>

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.RetryPolicies;
using Microsoft.WindowsAzure.Storage.Table;

namespace RetryCodeSamples
{
    class AzureStorageCodeSamples
    {
        private const string connectionString = "UseDevelopmentStorage=true";

        public async static Task Samples()
        {
            var storageAccount = CloudStorageAccount.Parse(connectionString);

            TableRequestOptions interactiveRequestOption = new TableRequestOptions()
            {
                RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
                // For Read-access geo-redundant storage, use PrimaryThenSecondary.
                // Otherwise set this to PrimaryOnly.
                LocationMode = LocationMode.PrimaryThenSecondary,
                // Maximum execution time based on the business use case.
                MaximumExecutionTime = TimeSpan.FromSeconds(2)
            };

            TableRequestOptions backgroundRequestOption = new TableRequestOptions()
            {
                // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
                // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
                MaximumExecutionTime = TimeSpan.FromSeconds(30),
                // PrimaryThenSecondary in case of Read-access geo-redundant storage, else set this to PrimaryOnly
                LocationMode = LocationMode.PrimaryThenSecondary
            };

            var client = storageAccount.CreateCloudTableClient();
            // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
            // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
            // ServerTimeout and MaximumExecutionTime are not set

            {
                // Set properties for the client (used on all requests unless overridden)
                // Different exponential policy parameters for background scenarios
                client.DefaultRequestOptions = backgroundRequestOption;
                // Linear policy for interactive scenarios
                client.DefaultRequestOptions = interactiveRequestOption;
            }

            {
                // set properties for a specific request
                var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
            }

            {
                // Set up notifications for an operation
                var context = new OperationContext();
                context.ClientRequestID = "some request id";
                context.Retrying += (sender, args) =>
                {
                    /* Collect retry information */
                };
                context.RequestCompleted += (sender, args) =>
                {
                    /* Collect operation completion information */
                };
                var stats = await client.GetServiceStatsAsync(null, context);
            }
        }
    }
}
```

### <a name="more-information"></a><span data-ttu-id="01d51-829">Plus d’informations</span><span class="sxs-lookup"><span data-stu-id="01d51-829">More information</span></span>

- [<span data-ttu-id="01d51-830">Recommandations de stratégie de nouvelle tentative de bibliothèque de client de stockage Azure</span><span class="sxs-lookup"><span data-stu-id="01d51-830">Azure Storage client Library retry policy recommendations</span></span>](https://azure.microsoft.com/blog/2014/05/22/azure-storage-client-library-retry-policy-recommendations/)

- [<span data-ttu-id="01d51-831">Bibliothèque de client de stockage 2.0 - Mise en œuvre de stratégies de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="01d51-831">Storage Client Library 2.0 – Implementing retry policies</span></span>](https://gauravmantri.com/2012/12/30/storage-client-library-2-0-implementing-retry-policies/)

## <a name="general-rest-and-retry-guidelines"></a><span data-ttu-id="01d51-832">Instructions générales relatives aux nouvelles tentatives et à REST</span><span class="sxs-lookup"><span data-stu-id="01d51-832">General REST and retry guidelines</span></span>

<span data-ttu-id="01d51-833">Prenez en compte les éléments suivants lors de l’accès aux services Azure ou tiers :</span><span class="sxs-lookup"><span data-stu-id="01d51-833">Consider the following when accessing Azure or third party services:</span></span>

- <span data-ttu-id="01d51-834">Utilisez une approche systématique pour gérer les nouvelles tentatives, peut-être en tant que code réutilisable, afin de pouvoir appliquer une méthodologie cohérente sur tous les clients et toutes les solutions.</span><span class="sxs-lookup"><span data-stu-id="01d51-834">Use a systematic approach to managing retries, perhaps as reusable code, so that you can apply a consistent methodology across all clients and all solutions.</span></span>

- <span data-ttu-id="01d51-835">Pensez à utiliser une infrastructure de nouvelle tentative, telle que [Polly][polly] pour gérer les nouvelles tentatives si le service cible ou le client ne dispose d’aucun mécanisme de nouvelle tentative intégré.</span><span class="sxs-lookup"><span data-stu-id="01d51-835">Consider using a retry framework such as [Polly][polly] to manage retries if the target service or client has no built-in retry mechanism.</span></span> <span data-ttu-id="01d51-836">Cela vous permettra d’implémenter un comportement cohérent de nouvelle tentative, et peut fournir une stratégie de nouvelle tentative par défaut appropriée pour le service cible.</span><span class="sxs-lookup"><span data-stu-id="01d51-836">This will help you implement a consistent retry behavior, and it may provide a suitable default retry strategy for the target service.</span></span> <span data-ttu-id="01d51-837">Toutefois, vous devrez peut-être créer un code personnalisé de nouvelle tentative pour les services ne disposant pas d’un comportement standard, qui ne reposent pas sur les exceptions pour indiquer les erreurs temporaires, ou si vous souhaitez utiliser une réponse **Retry-Response** pour gérer le comportement de la nouvelle tentative.</span><span class="sxs-lookup"><span data-stu-id="01d51-837">However, you may need to create custom retry code for services that have non-standard behavior, that do not rely on exceptions to indicate transient failures, or if you want to use a **Retry-Response** reply to manage retry behavior.</span></span>

- <span data-ttu-id="01d51-838">La logique de détection temporaire dépend de l’API client réelle utilisée pour appeler les appels REST.</span><span class="sxs-lookup"><span data-stu-id="01d51-838">The transient detection logic will depend on the actual client API you use to invoke the REST calls.</span></span> <span data-ttu-id="01d51-839">Certains clients, tels que la nouvelle classe **HttpClient** , ne lanceront pas d’exceptions pour les demandes terminées avec un code d’état HTTP indiquant un échec.</span><span class="sxs-lookup"><span data-stu-id="01d51-839">Some clients, such as the newer **HttpClient** class, will not throw exceptions for completed requests with a non-success HTTP status code.</span></span>

- <span data-ttu-id="01d51-840">Le code d’état HTTP renvoyé par le service peut permettre d’indiquer si l’échec est temporaire.</span><span class="sxs-lookup"><span data-stu-id="01d51-840">The HTTP status code returned from the service can help to indicate whether the failure is transient.</span></span> <span data-ttu-id="01d51-841">Vous devrez peut-être examiner les exceptions générées par un client ou l’infrastructure de nouvelle tentative pour accéder au code d’état ou pour déterminer le type d’exception équivalent.</span><span class="sxs-lookup"><span data-stu-id="01d51-841">You may need to examine the exceptions generated by a client or the retry framework to access the status code or to determine the equivalent exception type.</span></span> <span data-ttu-id="01d51-842">Les codes HTTP suivants indiquent habituellement qu’une nouvelle tentative est appropriée :</span><span class="sxs-lookup"><span data-stu-id="01d51-842">The following HTTP codes typically indicate that a retry is appropriate:</span></span>
  
  - <span data-ttu-id="01d51-843">408 Délai d’expiration de la requête</span><span class="sxs-lookup"><span data-stu-id="01d51-843">408 Request Timeout</span></span>
  - <span data-ttu-id="01d51-844">429 Trop de demandes</span><span class="sxs-lookup"><span data-stu-id="01d51-844">429 Too Many Requests</span></span>
  - <span data-ttu-id="01d51-845">500 Erreur interne du serveur</span><span class="sxs-lookup"><span data-stu-id="01d51-845">500 Internal Server Error</span></span>
  - <span data-ttu-id="01d51-846">502 Passerelle incorrecte</span><span class="sxs-lookup"><span data-stu-id="01d51-846">502 Bad Gateway</span></span>
  - <span data-ttu-id="01d51-847">503 Service indisponible</span><span class="sxs-lookup"><span data-stu-id="01d51-847">503 Service Unavailable</span></span>
  - <span data-ttu-id="01d51-848">504 Dépassement du délai de la passerelle</span><span class="sxs-lookup"><span data-stu-id="01d51-848">504 Gateway Timeout</span></span>

- <span data-ttu-id="01d51-849">Si vous basez votre logique de nouvelle tentative sur les exceptions, ce qui suit indique généralement un échec temporaire lorsqu’aucune connexion n’a pu être établie :</span><span class="sxs-lookup"><span data-stu-id="01d51-849">If you base your retry logic on exceptions, the following typically indicate a transient failure where no connection could be established:</span></span>

  - <span data-ttu-id="01d51-850">WebExceptionStatus.ConnectionClosed</span><span class="sxs-lookup"><span data-stu-id="01d51-850">WebExceptionStatus.ConnectionClosed</span></span>
  - <span data-ttu-id="01d51-851">WebExceptionStatus.ConnectFailure</span><span class="sxs-lookup"><span data-stu-id="01d51-851">WebExceptionStatus.ConnectFailure</span></span>
  - <span data-ttu-id="01d51-852">WebExceptionStatus.Timeout</span><span class="sxs-lookup"><span data-stu-id="01d51-852">WebExceptionStatus.Timeout</span></span>
  - <span data-ttu-id="01d51-853">WebExceptionStatus.RequestCanceled</span><span class="sxs-lookup"><span data-stu-id="01d51-853">WebExceptionStatus.RequestCanceled</span></span>

- <span data-ttu-id="01d51-854">Dans le cas d’un service à l’état non disponible, le service peut indiquer le délai approprié avant l’exécution d’une nouvelle tentative dans l’en-tête de réponse **Retry-After** ou dans un en-tête personnalisé différent.</span><span class="sxs-lookup"><span data-stu-id="01d51-854">In the case of a service unavailable status, the service might indicate the appropriate delay before retrying in the **Retry-After** response header or a different custom header.</span></span> <span data-ttu-id="01d51-855">Les services peuvent également envoyer des informations supplémentaires sous la forme d’en-têtes personnalisés ou en les intégrant au contenu de la réponse.</span><span class="sxs-lookup"><span data-stu-id="01d51-855">Services might also send additional information as custom headers, or embedded in the content of the response.</span></span>

- <span data-ttu-id="01d51-856">N’effectuez pas de nouvelle tentative pour les codes d’état représentant des erreurs du client (erreurs comprises dans la plage 4xx), excepté pour le code 408 Délai d’expiration de la requête.</span><span class="sxs-lookup"><span data-stu-id="01d51-856">Do not retry for status codes representing client errors (errors in the 4xx range) except for a 408 Request Timeout.</span></span>

- <span data-ttu-id="01d51-857">Testez minutieusement vos stratégies et mécanismes de nouvelle tentative sous différentes conditions, en utilisant différents états de réseau et en variant les charges du système.</span><span class="sxs-lookup"><span data-stu-id="01d51-857">Thoroughly test your retry strategies and mechanisms under a range of conditions, such as different network states and varying system loadings.</span></span>

### <a name="retry-strategies"></a><span data-ttu-id="01d51-858">Stratégie de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="01d51-858">Retry strategies</span></span>

<span data-ttu-id="01d51-859">Voici les types d’intervalles de stratégie de nouvelle tentative classiques :</span><span class="sxs-lookup"><span data-stu-id="01d51-859">The following are the typical types of retry strategy intervals:</span></span>

- <span data-ttu-id="01d51-860">**Exponentielle**.</span><span class="sxs-lookup"><span data-stu-id="01d51-860">**Exponential**.</span></span> <span data-ttu-id="01d51-861">Une stratégie de nouvelle tentative qui effectue un nombre spécifié de tentatives en utilisant une approche de secours exponentielle répartie de manière aléatoire pour déterminer l’intervalle entre deux tentatives.</span><span class="sxs-lookup"><span data-stu-id="01d51-861">A retry policy that performs a specified number of retries, using a randomized exponential back off approach to determine the interval between retries.</span></span> <span data-ttu-id="01d51-862">Par exemple : </span><span class="sxs-lookup"><span data-stu-id="01d51-862">For example:</span></span>

    ```csharp
    var random = new Random();

    var delta = (int)((Math.Pow(2.0, currentRetryCount) - 1.0) *
                random.Next((int)(this.deltaBackoff.TotalMilliseconds * 0.8),
                (int)(this.deltaBackoff.TotalMilliseconds * 1.2)));
    var interval = (int)Math.Min(checked(this.minBackoff.TotalMilliseconds + delta),
                    this.maxBackoff.TotalMilliseconds);
    retryInterval = TimeSpan.FromMilliseconds(interval);
    ```

- <span data-ttu-id="01d51-863">**Incrémentielle**.</span><span class="sxs-lookup"><span data-stu-id="01d51-863">**Incremental**.</span></span> <span data-ttu-id="01d51-864">Une stratégie de nouvelle tentative avec un nombre spécifié de nouvelles tentatives et un intervalle de temps incrémentiel entre deux tentatives.</span><span class="sxs-lookup"><span data-stu-id="01d51-864">A retry strategy with a specified number of retry attempts and an incremental time interval between retries.</span></span> <span data-ttu-id="01d51-865">Par exemple : </span><span class="sxs-lookup"><span data-stu-id="01d51-865">For example:</span></span>

    ```csharp
    retryInterval = TimeSpan.FromMilliseconds(this.initialInterval.TotalMilliseconds +
                    (this.increment.TotalMilliseconds * currentRetryCount));
    ```

- <span data-ttu-id="01d51-866">**LinearRetry**.</span><span class="sxs-lookup"><span data-stu-id="01d51-866">**LinearRetry**.</span></span> <span data-ttu-id="01d51-867">Une stratégie de nouvelle tentative qui effectue un nombre spécifié de nouvelles tentatives en utilisant un intervalle fixe spécifique entre deux tentatives.</span><span class="sxs-lookup"><span data-stu-id="01d51-867">A retry policy that performs a specified number of retries, using a specified fixed time interval between retries.</span></span> <span data-ttu-id="01d51-868">Par exemple : </span><span class="sxs-lookup"><span data-stu-id="01d51-868">For example:</span></span>

    ```csharp
    retryInterval = this.deltaBackoff;
    ```

### <a name="transient-fault-handling-with-polly"></a><span data-ttu-id="01d51-869">Gestion des erreurs temporaires avec Polly</span><span class="sxs-lookup"><span data-stu-id="01d51-869">Transient fault handling with Polly</span></span>

<span data-ttu-id="01d51-870">[Polly] [ polly] est une bibliothèque à gérer par programme les nouvelles tentatives et [disjoncteur](../patterns/circuit-breaker.md) stratégies.</span><span class="sxs-lookup"><span data-stu-id="01d51-870">[Polly][polly] is a library to programmatically handle retries and [circuit breaker](../patterns/circuit-breaker.md) strategies.</span></span> <span data-ttu-id="01d51-871">Le projet Polly est un membre de l’organisation [.NET Foundation][dotnet-foundation].</span><span class="sxs-lookup"><span data-stu-id="01d51-871">The Polly project is a member of the [.NET Foundation][dotnet-foundation].</span></span> <span data-ttu-id="01d51-872">Dans le cas des services pour lesquels le client ne prend pas en charge les nouvelles tentatives en mode natif, Polly constitue une alternative valide et évite d’avoir à écrire un code de nouvelle tentative personnalisé, qui peut être difficile à implémenter correctement.</span><span class="sxs-lookup"><span data-stu-id="01d51-872">For services where the client does not natively support retries, Polly is a valid alternative and avoids the need to write custom retry code, which can be hard to implement correctly.</span></span> <span data-ttu-id="01d51-873">Polly vous offre également un moyen de tracer les erreurs lorsqu’elles se produisent, ce qui vous permet de journaliser les nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="01d51-873">Polly also provides a way to trace errors when they occur, so that you can log retries.</span></span>

### <a name="more-information"></a><span data-ttu-id="01d51-874">Plus d’informations</span><span class="sxs-lookup"><span data-stu-id="01d51-874">More information</span></span>

- [<span data-ttu-id="01d51-875">Résilience de connexion</span><span class="sxs-lookup"><span data-stu-id="01d51-875">Connection resiliency</span></span>](/ef/core/miscellaneous/connection-resiliency)
- [<span data-ttu-id="01d51-876">Data Points - EF Core 1.1 (Points de données - EF Core 1.1)</span><span class="sxs-lookup"><span data-stu-id="01d51-876">Data Points - EF Core 1.1</span></span>](https://msdn.microsoft.com/magazine/mt745093.aspx)

<!-- links -->

[adal]: /azure/active-directory/develop/active-directory-authentication-libraries
[autorest]: https://github.com/Azure/autorest/tree/master/docs
[ConnectionPolicy.RetryOptions]: https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.connectionpolicy.retryoptions.aspx
[dotnet-foundation]: https://dotnetfoundation.org/
[polly]: http://www.thepollyproject.org
[redis-cache-troubleshoot]: /azure/redis-cache/cache-how-to-troubleshoot
[SearchIndexClient]: https://msdn.microsoft.com/library/azure/microsoft.azure.search.searchindexclient.aspx
[SearchServiceClient]: https://msdn.microsoft.com/library/microsoft.azure.search.searchserviceclient.aspx
