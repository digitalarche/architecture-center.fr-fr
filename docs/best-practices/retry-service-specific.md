---
title: Guide spécifique relatif au service de nouvelle tentative
description: Guide spécifique relatif au service pour définir le mécanisme de nouvelle tentative.
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: f02843f179671da04bc2f09326b58075b432ba95
ms.sourcegitcommit: 85334ab0ccb072dac80de78aa82bcfa0f0044d3f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/11/2018
ms.locfileid: "35253075"
---
# <a name="retry-guidance-for-specific-services"></a><span data-ttu-id="191e1-103">Guide du mécanisme de nouvelle tentative relatif aux différents services</span><span class="sxs-lookup"><span data-stu-id="191e1-103">Retry guidance for specific services</span></span>

<span data-ttu-id="191e1-104">La plupart des services Azure et des kits de développement logiciel (SDK) client incluent un mécanisme de nouvelle tentative.</span><span class="sxs-lookup"><span data-stu-id="191e1-104">Most Azure services and client SDKs include a retry mechanism.</span></span> <span data-ttu-id="191e1-105">Cependant, ceux-ci diffèrent, car chaque service a des exigences et des caractéristiques différentes, et donc chaque mécanisme de nouvelle tentative est réglé sur un service spécifique.</span><span class="sxs-lookup"><span data-stu-id="191e1-105">However, these differ because each service has different characteristics and requirements, and so each retry mechanism is tuned to a specific service.</span></span> <span data-ttu-id="191e1-106">Ce guide résume les fonctionnalités de mécanisme de nouvelle tentative pour la majorité des services Azure et inclut des informations pour vous aider à utiliser, adapter ou étendre le mécanisme de nouvelle tentative pour ce service.</span><span class="sxs-lookup"><span data-stu-id="191e1-106">This guide summarizes the retry mechanism features for the majority of Azure services, and includes information to help you use, adapt, or extend the retry mechanism for that service.</span></span>

<span data-ttu-id="191e1-107">Pour obtenir des instructions générales sur la gestion des erreurs temporaires, des nouvelles tentatives de connexion et des opérations en fonction des services et des ressources, consultez [Guide sur les nouvelles tentatives](./transient-faults.md).</span><span class="sxs-lookup"><span data-stu-id="191e1-107">For general guidance on handling transient faults, and retrying connections and operations against services and resources, see [Retry guidance](./transient-faults.md).</span></span>

<span data-ttu-id="191e1-108">Le tableau suivant récapitule les fonctionnalités de nouvelle tentative pour les services Azure décrits dans ce guide.</span><span class="sxs-lookup"><span data-stu-id="191e1-108">The following table summarizes the retry features for the Azure services described in this guidance.</span></span>

| <span data-ttu-id="191e1-109">**Service**</span><span class="sxs-lookup"><span data-stu-id="191e1-109">**Service**</span></span> | <span data-ttu-id="191e1-110">**Fonctionnalités de nouvelle tentative**</span><span class="sxs-lookup"><span data-stu-id="191e1-110">**Retry capabilities**</span></span> | <span data-ttu-id="191e1-111">**Configuration de la stratégie**</span><span class="sxs-lookup"><span data-stu-id="191e1-111">**Policy configuration**</span></span> | <span data-ttu-id="191e1-112">**Portée**</span><span class="sxs-lookup"><span data-stu-id="191e1-112">**Scope**</span></span> | <span data-ttu-id="191e1-113">**Fonctionnalités de télémétrie**</span><span class="sxs-lookup"><span data-stu-id="191e1-113">**Telemetry features**</span></span> |
| --- | --- | --- | --- | --- |
| <span data-ttu-id="191e1-114">**[Azure Active Directory](#azure-active-directory)**</span><span class="sxs-lookup"><span data-stu-id="191e1-114">**[Azure Active Directory](#azure-active-directory)**</span></span> |<span data-ttu-id="191e1-115">Native dans la bibliothèque ADAL</span><span class="sxs-lookup"><span data-stu-id="191e1-115">Native in ADAL library</span></span> |<span data-ttu-id="191e1-116">Incorporée dans la bibliothèque ADAL</span><span class="sxs-lookup"><span data-stu-id="191e1-116">Embeded into ADAL library</span></span> |<span data-ttu-id="191e1-117">Interne</span><span class="sxs-lookup"><span data-stu-id="191e1-117">Internal</span></span> |<span data-ttu-id="191e1-118">Aucun</span><span class="sxs-lookup"><span data-stu-id="191e1-118">None</span></span> |
| <span data-ttu-id="191e1-119">**[Cosmos DB](#cosmos-db)**</span><span class="sxs-lookup"><span data-stu-id="191e1-119">**[Cosmos DB](#cosmos-db)**</span></span> |<span data-ttu-id="191e1-120">Native dans le service</span><span class="sxs-lookup"><span data-stu-id="191e1-120">Native in service</span></span> |<span data-ttu-id="191e1-121">Non configurable</span><span class="sxs-lookup"><span data-stu-id="191e1-121">Non-configurable</span></span> |<span data-ttu-id="191e1-122">Globale</span><span class="sxs-lookup"><span data-stu-id="191e1-122">Global</span></span> |<span data-ttu-id="191e1-123">TraceSource</span><span class="sxs-lookup"><span data-stu-id="191e1-123">TraceSource</span></span> |
| <span data-ttu-id="191e1-124">**[Event Hubs](#azure-event-hubs)**</span><span class="sxs-lookup"><span data-stu-id="191e1-124">**[Event Hubs](#azure-event-hubs)**</span></span> |<span data-ttu-id="191e1-125">Native dans le client</span><span class="sxs-lookup"><span data-stu-id="191e1-125">Native in client</span></span> |<span data-ttu-id="191e1-126">Par programme</span><span class="sxs-lookup"><span data-stu-id="191e1-126">Programmatic</span></span> |<span data-ttu-id="191e1-127">Client</span><span class="sxs-lookup"><span data-stu-id="191e1-127">Client</span></span> |<span data-ttu-id="191e1-128">Aucun</span><span class="sxs-lookup"><span data-stu-id="191e1-128">None</span></span> |
| <span data-ttu-id="191e1-129">**[Cache Redis](#azure-redis-cache)**</span><span class="sxs-lookup"><span data-stu-id="191e1-129">**[Redis Cache](#azure-redis-cache)**</span></span> |<span data-ttu-id="191e1-130">Native dans le client</span><span class="sxs-lookup"><span data-stu-id="191e1-130">Native in client</span></span> |<span data-ttu-id="191e1-131">Par programme</span><span class="sxs-lookup"><span data-stu-id="191e1-131">Programmatic</span></span> |<span data-ttu-id="191e1-132">Client</span><span class="sxs-lookup"><span data-stu-id="191e1-132">Client</span></span> |<span data-ttu-id="191e1-133">TextWriter</span><span class="sxs-lookup"><span data-stu-id="191e1-133">TextWriter</span></span> |
| <span data-ttu-id="191e1-134">**[Recherche](#azure-search)**</span><span class="sxs-lookup"><span data-stu-id="191e1-134">**[Search](#azure-search)**</span></span> |<span data-ttu-id="191e1-135">Native dans le client</span><span class="sxs-lookup"><span data-stu-id="191e1-135">Native in client</span></span> |<span data-ttu-id="191e1-136">Par programme</span><span class="sxs-lookup"><span data-stu-id="191e1-136">Programmatic</span></span> |<span data-ttu-id="191e1-137">Client</span><span class="sxs-lookup"><span data-stu-id="191e1-137">Client</span></span> |<span data-ttu-id="191e1-138">ETW ou personnalisé</span><span class="sxs-lookup"><span data-stu-id="191e1-138">ETW or Custom</span></span> |
| <span data-ttu-id="191e1-139">**[Service Bus](#service-bus)**</span><span class="sxs-lookup"><span data-stu-id="191e1-139">**[Service Bus](#service-bus)**</span></span> |<span data-ttu-id="191e1-140">Native dans le client</span><span class="sxs-lookup"><span data-stu-id="191e1-140">Native in client</span></span> |<span data-ttu-id="191e1-141">Par programme</span><span class="sxs-lookup"><span data-stu-id="191e1-141">Programmatic</span></span> |<span data-ttu-id="191e1-142">Gestionnaire d’espace de noms, fabrique de messagerie et client</span><span class="sxs-lookup"><span data-stu-id="191e1-142">Namespace Manager, Messaging Factory, and Client</span></span> |<span data-ttu-id="191e1-143">ETW</span><span class="sxs-lookup"><span data-stu-id="191e1-143">ETW</span></span> |
| <span data-ttu-id="191e1-144">**[Service Fabric](#service-fabric)**</span><span class="sxs-lookup"><span data-stu-id="191e1-144">**[Service Fabric](#service-fabric)**</span></span> |<span data-ttu-id="191e1-145">Native dans le client</span><span class="sxs-lookup"><span data-stu-id="191e1-145">Native in client</span></span> |<span data-ttu-id="191e1-146">Par programme</span><span class="sxs-lookup"><span data-stu-id="191e1-146">Programmatic</span></span> |<span data-ttu-id="191e1-147">Client</span><span class="sxs-lookup"><span data-stu-id="191e1-147">Client</span></span> |<span data-ttu-id="191e1-148">Aucun</span><span class="sxs-lookup"><span data-stu-id="191e1-148">None</span></span> | 
| <span data-ttu-id="191e1-149">**[Base de données SQL avec ADO.NET](#sql-database-using-adonet)**</span><span class="sxs-lookup"><span data-stu-id="191e1-149">**[SQL Database with ADO.NET](#sql-database-using-adonet)**</span></span> |[<span data-ttu-id="191e1-150">Polly</span><span class="sxs-lookup"><span data-stu-id="191e1-150">Polly</span></span>](#transient-fault-handling-with-polly) |<span data-ttu-id="191e1-151">Déclarative et par programme</span><span class="sxs-lookup"><span data-stu-id="191e1-151">Declarative and programmatic</span></span> |<span data-ttu-id="191e1-152">Instructions uniques ou blocs de code</span><span class="sxs-lookup"><span data-stu-id="191e1-152">Single statements or blocks of code</span></span> |<span data-ttu-id="191e1-153">Personnalisée</span><span class="sxs-lookup"><span data-stu-id="191e1-153">Custom</span></span> |
| <span data-ttu-id="191e1-154">**[Base de données SQL avec Entity Framework](#sql-database-using-entity-framework-6)**</span><span class="sxs-lookup"><span data-stu-id="191e1-154">**[SQL Database with Entity Framework](#sql-database-using-entity-framework-6)**</span></span> |<span data-ttu-id="191e1-155">Native dans le client</span><span class="sxs-lookup"><span data-stu-id="191e1-155">Native in client</span></span> |<span data-ttu-id="191e1-156">Par programme</span><span class="sxs-lookup"><span data-stu-id="191e1-156">Programmatic</span></span> |<span data-ttu-id="191e1-157">Globale par domaine d’application</span><span class="sxs-lookup"><span data-stu-id="191e1-157">Global per AppDomain</span></span> |<span data-ttu-id="191e1-158">Aucun</span><span class="sxs-lookup"><span data-stu-id="191e1-158">None</span></span> |
| <span data-ttu-id="191e1-159">**[SQL Database avec Entity Framework Core](#sql-database-using-entity-framework-core)**</span><span class="sxs-lookup"><span data-stu-id="191e1-159">**[SQL Database with Entity Framework Core](#sql-database-using-entity-framework-core)**</span></span> |<span data-ttu-id="191e1-160">Native dans le client</span><span class="sxs-lookup"><span data-stu-id="191e1-160">Native in client</span></span> |<span data-ttu-id="191e1-161">Par programme</span><span class="sxs-lookup"><span data-stu-id="191e1-161">Programmatic</span></span> |<span data-ttu-id="191e1-162">Globale par domaine d’application</span><span class="sxs-lookup"><span data-stu-id="191e1-162">Global per AppDomain</span></span> |<span data-ttu-id="191e1-163">Aucun</span><span class="sxs-lookup"><span data-stu-id="191e1-163">None</span></span> |
| <span data-ttu-id="191e1-164">**[Stockage](#azure-storage)**</span><span class="sxs-lookup"><span data-stu-id="191e1-164">**[Storage](#azure-storage)**</span></span> |<span data-ttu-id="191e1-165">Native dans le client</span><span class="sxs-lookup"><span data-stu-id="191e1-165">Native in client</span></span> |<span data-ttu-id="191e1-166">Par programme</span><span class="sxs-lookup"><span data-stu-id="191e1-166">Programmatic</span></span> |<span data-ttu-id="191e1-167">Opérations individuelles et du client</span><span class="sxs-lookup"><span data-stu-id="191e1-167">Client and individual operations</span></span> |<span data-ttu-id="191e1-168">TraceSource</span><span class="sxs-lookup"><span data-stu-id="191e1-168">TraceSource</span></span> |

> [!NOTE]
> <span data-ttu-id="191e1-169">Pour la plupart des mécanismes de nouvelle tentative intégrés Azure, il n’existe actuellement aucune manière d’appliquer une autre stratégie de nouvelle tentative selon le type d’erreur ou d’exception.</span><span class="sxs-lookup"><span data-stu-id="191e1-169">For most of the Azure built-in retry mechanisms, there is currently no way apply a different retry policy for different types of error or exception.</span></span> <span data-ttu-id="191e1-170">Vous devez configurer une stratégie qui fournit les performances et la disponibilité moyennes optimales.</span><span class="sxs-lookup"><span data-stu-id="191e1-170">You should configure a policy that provides the optimum average performance and availability.</span></span> <span data-ttu-id="191e1-171">Une façon d’ajuster la stratégie consiste à analyser les fichiers journaux pour déterminer le type d’erreurs temporaires qui se produisent.</span><span class="sxs-lookup"><span data-stu-id="191e1-171">One way to fine-tune the policy is to analyze log files to determine the type of transient faults that are occurring.</span></span> 

## <a name="azure-active-directory"></a><span data-ttu-id="191e1-172">Azure Active Directory</span><span class="sxs-lookup"><span data-stu-id="191e1-172">Azure Active Directory</span></span>
<span data-ttu-id="191e1-173">Azure Active Directory (AD) est une solution cloud complète de gestion des accès et des identités qui combine des services d’annuaire essentiels, une gouvernance avancée des identités, des services de sécurité et une gestion des accès aux applications.</span><span class="sxs-lookup"><span data-stu-id="191e1-173">Azure Active Directory (Azure AD) is a comprehensive identity and access management cloud solution that combines core directory services, advanced identity governance, security, and application access management.</span></span> <span data-ttu-id="191e1-174">Azure AD offre également aux développeurs une plate-forme de gestion des identités pour permettre un contrôle d’accès à leurs applications, en fonction d’une stratégie et de règles centralisés.</span><span class="sxs-lookup"><span data-stu-id="191e1-174">Azure AD also offers developers an identity management platform to deliver access control to their applications, based on centralized policy and rules.</span></span>

> [!NOTE]
> <span data-ttu-id="191e1-175">Pour obtenir des conseils de nouvelle tentative sur les points de terminaison Managed Service Identity, consultez [Utilisation d’une identité du service administré (MSI) d’une machine virtuelle Azure pour obtenir des jetons](/azure/active-directory/managed-service-identity/how-to-use-vm-token#error-handling).</span><span class="sxs-lookup"><span data-stu-id="191e1-175">For retry guidance on Managed Service Identity endpoints, see [How to use an Azure VM Managed Service Identity (MSI) for token acquisition](/azure/active-directory/managed-service-identity/how-to-use-vm-token#error-handling).</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="191e1-176">Mécanisme de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="191e1-176">Retry mechanism</span></span>
<span data-ttu-id="191e1-177">La bibliothèque d’authentification d’Active Directory (ADAL) intègre un mécanisme de nouvelle tentative pour Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="191e1-177">There is a built-in retry mechanism for Azure Active Directory in the Active Directory Authentication Library (ADAL).</span></span> <span data-ttu-id="191e1-178">Pour éviter les verrouillages imprévus, il est recommandé que les bibliothèques et codes d’application tiers ne procèdent **pas** à de nouvelles tentatives pour les connexions ayant échoué, mais autorisent la bibliothèque ADAL à gérer les nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="191e1-178">To avoid unexpected lockouts, we recommend that third party libraries and application code do **not** retry failed connections, but allow ADAL to handle retries.</span></span> 

### <a name="retry-usage-guidance"></a><span data-ttu-id="191e1-179">Guide d’utilisation des nouvelles tentatives</span><span class="sxs-lookup"><span data-stu-id="191e1-179">Retry usage guidance</span></span>
<span data-ttu-id="191e1-180">Respectez les consignes suivantes lors de l’utilisation d’Azure Active Directory :</span><span class="sxs-lookup"><span data-stu-id="191e1-180">Consider the following guidelines when using Azure Active Directory:</span></span>

* <span data-ttu-id="191e1-181">Dans la mesure du possible, utilisez la bibliothèque ADAL et la prise en charge intégrée des nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="191e1-181">When possible, use the ADAL library and the built-in support for retries.</span></span>
* <span data-ttu-id="191e1-182">Si vous utilisez l’API REST pour Azure Active Directory, réessayez l’opération si le code de résultat est 429 (trop de demandes) ou si vous rencontrez une erreur dans la plage 5xx.</span><span class="sxs-lookup"><span data-stu-id="191e1-182">If you are using the REST API for Azure Active Directory, retry the operation if the result code is 429 (Too Many Requests) or an error in the 5xx range.</span></span> <span data-ttu-id="191e1-183">N’effectuez pas de nouvelle tentative pour toute autre erreur.</span><span class="sxs-lookup"><span data-stu-id="191e1-183">Do not retry for any other errors.</span></span>
* <span data-ttu-id="191e1-184">Une stratégie de temporisation exponentielle est recommandée pour une utilisation dans des scénarios de traitement par lots avec Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="191e1-184">An exponential back-off policy is recommended for use in batch scenarios with Azure Active Directory.</span></span>

<span data-ttu-id="191e1-185">Pensez à commencer par les paramètres ci-après pour les opérations liées aux nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="191e1-185">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="191e1-186">Il s’agit de paramètres généraux. Vous devez par conséquent surveiller les opérations et optimiser les valeurs en fonction de votre propre scénario.</span><span class="sxs-lookup"><span data-stu-id="191e1-186">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="191e1-187">**Contexte**</span><span class="sxs-lookup"><span data-stu-id="191e1-187">**Context**</span></span> | <span data-ttu-id="191e1-188">**Exemple de cible E2E<br />latence max**</span><span class="sxs-lookup"><span data-stu-id="191e1-188">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="191e1-189">**Stratégie de nouvelle tentative**</span><span class="sxs-lookup"><span data-stu-id="191e1-189">**Retry strategy**</span></span> | <span data-ttu-id="191e1-190">**Paramètres**</span><span class="sxs-lookup"><span data-stu-id="191e1-190">**Settings**</span></span> | <span data-ttu-id="191e1-191">**Valeurs**</span><span class="sxs-lookup"><span data-stu-id="191e1-191">**Values**</span></span> | <span data-ttu-id="191e1-192">**Fonctionnement**</span><span class="sxs-lookup"><span data-stu-id="191e1-192">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="191e1-193">Interactif, interface utilisateur,</span><span class="sxs-lookup"><span data-stu-id="191e1-193">Interactive, UI,</span></span><br /><span data-ttu-id="191e1-194">ou premier plan</span><span class="sxs-lookup"><span data-stu-id="191e1-194">or foreground</span></span> |<span data-ttu-id="191e1-195">2 secondes</span><span class="sxs-lookup"><span data-stu-id="191e1-195">2 sec</span></span> |<span data-ttu-id="191e1-196">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="191e1-196">FixedInterval</span></span> |<span data-ttu-id="191e1-197">Nombre de tentatives</span><span class="sxs-lookup"><span data-stu-id="191e1-197">Retry count</span></span><br /><span data-ttu-id="191e1-198">Intervalle avant nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="191e1-198">Retry interval</span></span><br /><span data-ttu-id="191e1-199">Première nouvelle tentative rapide</span><span class="sxs-lookup"><span data-stu-id="191e1-199">First fast retry</span></span> |<span data-ttu-id="191e1-200">3</span><span class="sxs-lookup"><span data-stu-id="191e1-200">3</span></span><br /><span data-ttu-id="191e1-201">500 ms</span><span class="sxs-lookup"><span data-stu-id="191e1-201">500 ms</span></span><br /><span data-ttu-id="191e1-202">true</span><span class="sxs-lookup"><span data-stu-id="191e1-202">true</span></span> |<span data-ttu-id="191e1-203">Tentative 1 - délai 0 s</span><span class="sxs-lookup"><span data-stu-id="191e1-203">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="191e1-204">Tentative 2 - délai 500 ms</span><span class="sxs-lookup"><span data-stu-id="191e1-204">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="191e1-205">Tentative 3 - délai 500 ms</span><span class="sxs-lookup"><span data-stu-id="191e1-205">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="191e1-206">Arrière-plan ou </span><span class="sxs-lookup"><span data-stu-id="191e1-206">Background or</span></span><br /><span data-ttu-id="191e1-207">lot</span><span class="sxs-lookup"><span data-stu-id="191e1-207">batch</span></span> |<span data-ttu-id="191e1-208">60 secondes</span><span class="sxs-lookup"><span data-stu-id="191e1-208">60 sec</span></span> |<span data-ttu-id="191e1-209">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="191e1-209">ExponentialBackoff</span></span> |<span data-ttu-id="191e1-210">Nombre de tentatives</span><span class="sxs-lookup"><span data-stu-id="191e1-210">Retry count</span></span><br /><span data-ttu-id="191e1-211">Temporisation min</span><span class="sxs-lookup"><span data-stu-id="191e1-211">Min back-off</span></span><br /><span data-ttu-id="191e1-212">Temporisation max</span><span class="sxs-lookup"><span data-stu-id="191e1-212">Max back-off</span></span><br /><span data-ttu-id="191e1-213">Temporisation delta</span><span class="sxs-lookup"><span data-stu-id="191e1-213">Delta back-off</span></span><br /><span data-ttu-id="191e1-214">Première nouvelle tentative rapide</span><span class="sxs-lookup"><span data-stu-id="191e1-214">First fast retry</span></span> |<span data-ttu-id="191e1-215">5.</span><span class="sxs-lookup"><span data-stu-id="191e1-215">5</span></span><br /><span data-ttu-id="191e1-216">0 seconde</span><span class="sxs-lookup"><span data-stu-id="191e1-216">0 sec</span></span><br /><span data-ttu-id="191e1-217">60 secondes</span><span class="sxs-lookup"><span data-stu-id="191e1-217">60 sec</span></span><br /><span data-ttu-id="191e1-218">2 secondes</span><span class="sxs-lookup"><span data-stu-id="191e1-218">2 sec</span></span><br /><span data-ttu-id="191e1-219">false</span><span class="sxs-lookup"><span data-stu-id="191e1-219">false</span></span> |<span data-ttu-id="191e1-220">Tentative 1 - délai 0 s</span><span class="sxs-lookup"><span data-stu-id="191e1-220">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="191e1-221">Tentative 2 - délai ~2 s</span><span class="sxs-lookup"><span data-stu-id="191e1-221">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="191e1-222">Tentative 3 - délai ~6 s</span><span class="sxs-lookup"><span data-stu-id="191e1-222">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="191e1-223">Tentative 4 - délai ~14 s</span><span class="sxs-lookup"><span data-stu-id="191e1-223">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="191e1-224">Tentative 5 - délai ~30 s</span><span class="sxs-lookup"><span data-stu-id="191e1-224">Attempt 5 - delay ~30 sec</span></span> |

### <a name="more-information"></a><span data-ttu-id="191e1-225">Plus d’informations</span><span class="sxs-lookup"><span data-stu-id="191e1-225">More information</span></span>
* <span data-ttu-id="191e1-226">[Bibliothèques d’authentification d’Azure Active Directory][adal]</span><span class="sxs-lookup"><span data-stu-id="191e1-226">[Azure Active Directory Authentication Libraries][adal]</span></span>

## <a name="cosmos-db"></a><span data-ttu-id="191e1-227">Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="191e1-227">Cosmos DB</span></span>

<span data-ttu-id="191e1-228">Cosmos DB est une base de données multimodèle entièrement gérée qui prend en charge les données JSON sans schéma.</span><span class="sxs-lookup"><span data-stu-id="191e1-228">Cosmos DB is a fully-managed multi-model database that supports schema-less JSON data.</span></span> <span data-ttu-id="191e1-229">Elle offre des performances configurables et fiables, un traitement transactionnel JavaScript natif et est conçue pour le cloud avec une mise à l’échelle élastique.</span><span class="sxs-lookup"><span data-stu-id="191e1-229">It offers configurable and reliable performance, native JavaScript transactional processing, and is built for the cloud with elastic scale.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="191e1-230">Mécanisme de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="191e1-230">Retry mechanism</span></span>
<span data-ttu-id="191e1-231">La classe `DocumentClient` retente automatiquement les tentatives ayant échoué.</span><span class="sxs-lookup"><span data-stu-id="191e1-231">The `DocumentClient` class automatically retries failed attempts.</span></span> <span data-ttu-id="191e1-232">Pour définir le nombre de nouvelles tentatives et le délai d’attente maximal, configurez [ConnectionPolicy.RetryOptions].</span><span class="sxs-lookup"><span data-stu-id="191e1-232">To set the number of retries and the maximum wait time, configure [ConnectionPolicy.RetryOptions].</span></span> <span data-ttu-id="191e1-233">Les exceptions déclenchées par le client sont au-delà de la stratégie de nouvelle tentative ou ne sont pas des erreurs temporaires.</span><span class="sxs-lookup"><span data-stu-id="191e1-233">Exceptions that the client raises are either beyond the retry policy or are not transient errors.</span></span>

<span data-ttu-id="191e1-234">Si Cosmos DB limite le client, il renvoie une erreur HTTP 429.</span><span class="sxs-lookup"><span data-stu-id="191e1-234">If Cosmos DB throttles the client, it returns an HTTP 429 error.</span></span> <span data-ttu-id="191e1-235">Vérifiez le code d’état dans `DocumentClientException`.</span><span class="sxs-lookup"><span data-stu-id="191e1-235">Check the status code in the `DocumentClientException`.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="191e1-236">Configuration de la stratégie</span><span class="sxs-lookup"><span data-stu-id="191e1-236">Policy configuration</span></span>
<span data-ttu-id="191e1-237">Le tableau suivant présente les paramètres par défaut pour la classe `RetryOptions`.</span><span class="sxs-lookup"><span data-stu-id="191e1-237">The following table shows the default settings for the `RetryOptions` class.</span></span>

| <span data-ttu-id="191e1-238">Paramètre</span><span class="sxs-lookup"><span data-stu-id="191e1-238">Setting</span></span> | <span data-ttu-id="191e1-239">Valeur par défaut</span><span class="sxs-lookup"><span data-stu-id="191e1-239">Default value</span></span> | <span data-ttu-id="191e1-240">Description</span><span class="sxs-lookup"><span data-stu-id="191e1-240">Description</span></span> |
| --- | --- | --- |
| <span data-ttu-id="191e1-241">MaxRetryAttemptsOnThrottledRequests</span><span class="sxs-lookup"><span data-stu-id="191e1-241">MaxRetryAttemptsOnThrottledRequests</span></span> |<span data-ttu-id="191e1-242">9</span><span class="sxs-lookup"><span data-stu-id="191e1-242">9</span></span> |<span data-ttu-id="191e1-243">Nombre maximal de nouvelles tentatives en cas d’échec de la requête, car Cosmos DB a appliqué une limite du débit au client.</span><span class="sxs-lookup"><span data-stu-id="191e1-243">The maximum number of retries if the request fails because Cosmos DB applied rate limiting on the client.</span></span> |
| <span data-ttu-id="191e1-244">MaxRetryWaitTimeInSeconds</span><span class="sxs-lookup"><span data-stu-id="191e1-244">MaxRetryWaitTimeInSeconds</span></span> |<span data-ttu-id="191e1-245">30</span><span class="sxs-lookup"><span data-stu-id="191e1-245">30</span></span> |<span data-ttu-id="191e1-246">La durée maximale de nouvelle tentative en secondes.</span><span class="sxs-lookup"><span data-stu-id="191e1-246">The maximum retry time in seconds.</span></span> |

### <a name="example"></a><span data-ttu-id="191e1-247">Exemples</span><span class="sxs-lookup"><span data-stu-id="191e1-247">Example</span></span>
```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey); ;
var options = client.ConnectionPolicy.RetryOptions;
options.MaxRetryAttemptsOnThrottledRequests = 5;
options.MaxRetryWaitTimeInSeconds = 15;
```

### <a name="telemetry"></a><span data-ttu-id="191e1-248">Télémétrie</span><span class="sxs-lookup"><span data-stu-id="191e1-248">Telemetry</span></span>
<span data-ttu-id="191e1-249">Les nouvelles tentatives sont enregistrées sous forme de messages de trace non structurés via un **TraceSource**.NET.</span><span class="sxs-lookup"><span data-stu-id="191e1-249">Retry attempts are logged as unstructured trace messages through a .NET **TraceSource**.</span></span> <span data-ttu-id="191e1-250">Vous devez configurer un **TraceListener** pour capturer les événements et les écrire dans un journal de destination approprié.</span><span class="sxs-lookup"><span data-stu-id="191e1-250">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span>

<span data-ttu-id="191e1-251">Par exemple, si vous ajoutez ce qui suit à votre fichier App.config, les traces seront générées dans un fichier texte dans le même emplacement que le fichier exécutable :</span><span class="sxs-lookup"><span data-stu-id="191e1-251">For example, if you add the following to your App.config file, traces will be generated in a text file in the same location as the executable:</span></span>

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

## <a name="event-hubs"></a><span data-ttu-id="191e1-252">Event Hubs</span><span class="sxs-lookup"><span data-stu-id="191e1-252">Event Hubs</span></span>

<span data-ttu-id="191e1-253">Azure Event Hubs est un service d’ingestion de données de télémétrie à très grande échelle qui collecte, transforme et stocke des millions d’événements.</span><span class="sxs-lookup"><span data-stu-id="191e1-253">Azure Event Hubs is a hyper-scale telemetry ingestion service that collects, transforms, and stores millions of events.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="191e1-254">Mécanisme de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="191e1-254">Retry mechanism</span></span>
<span data-ttu-id="191e1-255">Le comportement du mécanisme de nouvelle tentative dans la bibliothèque de client Azure Event Hubs est contrôlé par la propriété `RetryPolicy` sur la classe `EventHubClient`.</span><span class="sxs-lookup"><span data-stu-id="191e1-255">Retry behavior in the Azure Event Hubs Client Library is controlled by the `RetryPolicy` property on the `EventHubClient` class.</span></span> <span data-ttu-id="191e1-256">La stratégie par défaut effectue de nouvelles tentatives avec une interruption exponentielle lorsqu’Azure Event Hubs renvoie une exception temporaire `EventHubsException` ou une exception `OperationCanceledException`.</span><span class="sxs-lookup"><span data-stu-id="191e1-256">The default policy retries with exponential backoff when Azure Event Hub returns a transient `EventHubsException` or an `OperationCanceledException`.</span></span>

### <a name="example"></a><span data-ttu-id="191e1-257">Exemples</span><span class="sxs-lookup"><span data-stu-id="191e1-257">Example</span></span>
```csharp
EventHubClient client = EventHubClient.CreateFromConnectionString("[event_hub_connection_string]");
client.RetryPolicy = RetryPolicy.Default;
```

### <a name="more-information"></a><span data-ttu-id="191e1-258">Plus d’informations</span><span class="sxs-lookup"><span data-stu-id="191e1-258">More information</span></span>
[<span data-ttu-id="191e1-259">Bibliothèque de client .NET Standard pour Azure Event Hubs</span><span class="sxs-lookup"><span data-stu-id="191e1-259"> .NET Standard client library for Azure Event Hubs</span></span>](https://github.com/Azure/azure-event-hubs-dotnet)

## <a name="azure-redis-cache"></a><span data-ttu-id="191e1-260">Cache Redis Azure</span><span class="sxs-lookup"><span data-stu-id="191e1-260">Azure Redis Cache</span></span>
<span data-ttu-id="191e1-261">Cache Redis Azure est un service de cache de faible latence et d’accès aux données rapide basé sur le cache Redis open source connu.</span><span class="sxs-lookup"><span data-stu-id="191e1-261">Azure Redis Cache is a fast data access and low latency cache service based on the popular open source Redis Cache.</span></span> <span data-ttu-id="191e1-262">Il est sécurisé, géré par Microsoft et accessible à partir de n’importe quelle application dans Azure.</span><span class="sxs-lookup"><span data-stu-id="191e1-262">It is secure, managed by Microsoft, and is accessible from any application in Azure.</span></span>

<span data-ttu-id="191e1-263">Les instructions de cette section sont basées sur l’utilisation du client StackExchange.Redis pour accéder au cache.</span><span class="sxs-lookup"><span data-stu-id="191e1-263">The guidance in this section is based on using the StackExchange.Redis client to access the cache.</span></span> <span data-ttu-id="191e1-264">Vous trouverez une liste des autres clients appropriés sur le [site web de Redis](http://redis.io/clients), qui peuvent avoir des mécanismes de nouvelle tentative différents.</span><span class="sxs-lookup"><span data-stu-id="191e1-264">A list of other suitable clients can be found on the [Redis website](http://redis.io/clients), and these may have different retry mechanisms.</span></span>

<span data-ttu-id="191e1-265">Notez que le client StackExchange.Redis utilise le multiplexage via une connexion unique.</span><span class="sxs-lookup"><span data-stu-id="191e1-265">Note that the StackExchange.Redis client uses multiplexing through a single connection.</span></span> <span data-ttu-id="191e1-266">L’utilisation recommandée consiste à créer une instance du client au démarrage de l’application et à utiliser cette instance pour toutes les opérations sur le cache.</span><span class="sxs-lookup"><span data-stu-id="191e1-266">The recommended usage is to create an instance of the client at application startup and use this instance for all operations against the cache.</span></span> <span data-ttu-id="191e1-267">Pour cette raison, la connexion au cache est effectuée une seule fois. Ainsi, toutes les instructions de cette section sont associées à la stratégie de nouvelle tentative pour cette connexion initiale et non pour chaque opération qui accède au cache.</span><span class="sxs-lookup"><span data-stu-id="191e1-267">For this reason, the connection to the cache is made only once, and so all of the guidance in this section is related to the retry policy for this initial connection—and not for each operation that accesses the cache.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="191e1-268">Mécanisme de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="191e1-268">Retry mechanism</span></span>
<span data-ttu-id="191e1-269">Le client StackExchange.Redis utilise une classe de gestionnaire de connexions qui est configurée par le biais d’un ensemble d’options, incluant :</span><span class="sxs-lookup"><span data-stu-id="191e1-269">The StackExchange.Redis client uses a connection manager class that is configured through a set of options, incuding:</span></span>

- <span data-ttu-id="191e1-270">**ConnectRetry**.</span><span class="sxs-lookup"><span data-stu-id="191e1-270">**ConnectRetry**.</span></span> <span data-ttu-id="191e1-271">Nombre de fois où une connexion au cache ayant échoué fera l’objet d’une nouvelle tentative.</span><span class="sxs-lookup"><span data-stu-id="191e1-271">The number of times a failed connection to the cache will be retried.</span></span>
- <span data-ttu-id="191e1-272">**ReconnectRetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="191e1-272">**ReconnectRetryPolicy**.</span></span> <span data-ttu-id="191e1-273">Stratégie de nouvelle tentative à utiliser.</span><span class="sxs-lookup"><span data-stu-id="191e1-273">The retry strategy to use.</span></span>
- <span data-ttu-id="191e1-274">**ConnectTimeout**.</span><span class="sxs-lookup"><span data-stu-id="191e1-274">**ConnectTimeout**.</span></span> <span data-ttu-id="191e1-275">Temps d’attente maximal en millisecondes.</span><span class="sxs-lookup"><span data-stu-id="191e1-275">The maximum waiting time in milliseconds.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="191e1-276">Configuration de la stratégie</span><span class="sxs-lookup"><span data-stu-id="191e1-276">Policy configuration</span></span>
<span data-ttu-id="191e1-277">Les stratégies de nouvelle tentative sont configurées par programme en définissant les options pour le client avant de se connecter au cache.</span><span class="sxs-lookup"><span data-stu-id="191e1-277">Retry policies are configured programmatically by setting the options for the client before connecting to the cache.</span></span> <span data-ttu-id="191e1-278">Cela peut être effectué en créant une instance de la classe **ConfigurationOptions**, en remplissant ses propriétés et en la transmettant à la méthode **Connect**.</span><span class="sxs-lookup"><span data-stu-id="191e1-278">This can be done by creating an instance of the **ConfigurationOptions** class, populating its properties, and passing it to the **Connect** method.</span></span>

<span data-ttu-id="191e1-279">Les classes intégrées prennent en charge un délai linéaire (constant) et une interruption exponentielle avec des intervalles avant nouvelle tentative aléatoires.</span><span class="sxs-lookup"><span data-stu-id="191e1-279">The built-in classes support linear (constant) delay and exponential backoff with randomized retry intervals.</span></span> <span data-ttu-id="191e1-280">Vous pouvez également créer une stratégie de nouvelle tentative personnalisée en implémentant l’interface **IReconnectRetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="191e1-280">You can also create a custom retry policy by implementing the **IReconnectRetryPolicy** interface.</span></span>

<span data-ttu-id="191e1-281">L’exemple ci-après configure une stratégie de nouvelle tentative à l’aide d’une interruption exponentielle.</span><span class="sxs-lookup"><span data-stu-id="191e1-281">The following example configures a retry strategy using exponential backoff.</span></span>

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

<span data-ttu-id="191e1-282">Vous pouvez également spécifier les options sous forme de chaîne et la transmettre à la méthode **Connect** .</span><span class="sxs-lookup"><span data-stu-id="191e1-282">Alternatively, you can specify the options as a string, and pass this to the **Connect** method.</span></span> <span data-ttu-id="191e1-283">Notez que la propriété **ReconnectRetryPolicy** n’est pas définissable de cette façon, mais uniquement au moyen du code.</span><span class="sxs-lookup"><span data-stu-id="191e1-283">Note that the **ReconnectRetryPolicy** property cannot be set this way, only through code.</span></span>

```csharp
var options = "localhost,connectRetry=3,connectTimeout=2000";
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="191e1-284">Vous pouvez également spécifier des options directement lorsque vous vous connectez au cache.</span><span class="sxs-lookup"><span data-stu-id="191e1-284">You can also specify options directly when you connect to the cache.</span></span>

```csharp
var conn = ConnectionMultiplexer.Connect("redis0:6380,redis1:6380,connectRetry=3");
```

<span data-ttu-id="191e1-285">Pour plus d’informations, consultez la section concernant la [configuration de StackExchange.Redis](https://stackexchange.github.io/StackExchange.Redis/Configuration) dans la documentation de StackExchange.Redis.</span><span class="sxs-lookup"><span data-stu-id="191e1-285">For more information, see [Stack Exchange Redis Configuration](https://stackexchange.github.io/StackExchange.Redis/Configuration) in the StackExchange.Redis documentation.</span></span>

<span data-ttu-id="191e1-286">Le tableau suivant présente les paramètres par défaut pour la stratégie de nouvelle tentative intégrée.</span><span class="sxs-lookup"><span data-stu-id="191e1-286">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="191e1-287">**Contexte**</span><span class="sxs-lookup"><span data-stu-id="191e1-287">**Context**</span></span> | <span data-ttu-id="191e1-288">**Paramètre**</span><span class="sxs-lookup"><span data-stu-id="191e1-288">**Setting**</span></span> | <span data-ttu-id="191e1-289">**Valeur par défaut**</span><span class="sxs-lookup"><span data-stu-id="191e1-289">**Default value**</span></span><br /><span data-ttu-id="191e1-290">(v 1.2.2)</span><span class="sxs-lookup"><span data-stu-id="191e1-290">(v 1.2.2)</span></span> | <span data-ttu-id="191e1-291">**Signification**</span><span class="sxs-lookup"><span data-stu-id="191e1-291">**Meaning**</span></span> |
| --- | --- | --- | --- |
| <span data-ttu-id="191e1-292">ConfigurationOptions</span><span class="sxs-lookup"><span data-stu-id="191e1-292">ConfigurationOptions</span></span> |<span data-ttu-id="191e1-293">ConnectRetry</span><span class="sxs-lookup"><span data-stu-id="191e1-293">ConnectRetry</span></span><br /><br /><span data-ttu-id="191e1-294">ConnectTimeout</span><span class="sxs-lookup"><span data-stu-id="191e1-294">ConnectTimeout</span></span><br /><br /><span data-ttu-id="191e1-295">SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="191e1-295">SyncTimeout</span></span><br /><br /><span data-ttu-id="191e1-296">ReconnectRetryPolicy</span><span class="sxs-lookup"><span data-stu-id="191e1-296">ReconnectRetryPolicy</span></span> |<span data-ttu-id="191e1-297">3</span><span class="sxs-lookup"><span data-stu-id="191e1-297">3</span></span><br /><br /><span data-ttu-id="191e1-298">5 000 ms maximum plus SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="191e1-298">Maximum 5000 ms plus SyncTimeout</span></span><br /><span data-ttu-id="191e1-299">1 000</span><span class="sxs-lookup"><span data-stu-id="191e1-299">1000</span></span><br /><br /><span data-ttu-id="191e1-300">LinearRetry 5 000 ms</span><span class="sxs-lookup"><span data-stu-id="191e1-300">LinearRetry 5000 ms</span></span> |<span data-ttu-id="191e1-301">Le nombre de répétitions de tentatives de connexion pendant l’opération de connexion initiale.</span><span class="sxs-lookup"><span data-stu-id="191e1-301">The number of times to repeat connect attempts during the initial connection operation.</span></span><br /><span data-ttu-id="191e1-302">Délai d’attente (ms) pour les opérations de connexion.</span><span class="sxs-lookup"><span data-stu-id="191e1-302">Timeout (ms) for connect operations.</span></span> <span data-ttu-id="191e1-303">Pas un délai entre chaque tentative.</span><span class="sxs-lookup"><span data-stu-id="191e1-303">Not a delay between retry attempts.</span></span><br /><span data-ttu-id="191e1-304">Temps (ms) pour permettre des opérations synchrones.</span><span class="sxs-lookup"><span data-stu-id="191e1-304">Time (ms) to allow for synchronous operations.</span></span><br /><br /><span data-ttu-id="191e1-305">Nouvelle tentative toutes les 5 000 ms.</span><span class="sxs-lookup"><span data-stu-id="191e1-305">Retry every 5000 ms.</span></span>|

> [!NOTE]
> <span data-ttu-id="191e1-306">Dans le cas des opérations synchrones, le paramètre `SyncTimeout` peut augmenter la latence de bout en bout, mais la définition d’une valeur trop faible risque d’entraîner des délais d’expiration excessifs.</span><span class="sxs-lookup"><span data-stu-id="191e1-306">For synchronous operations, `SyncTimeout` can add to the end-to-end latency, but setting the value too low can cause excessive timeouts.</span></span> <span data-ttu-id="191e1-307">Consultez l’article [Résolution des problèmes du cache Redis Azure][redis-cache-troubleshoot].</span><span class="sxs-lookup"><span data-stu-id="191e1-307">See [How to troubleshoot Azure Redis Cache][redis-cache-troubleshoot].</span></span> <span data-ttu-id="191e1-308">En règle générale, utilisez des opérations asynchrones plutôt que des opérations synchrones.</span><span class="sxs-lookup"><span data-stu-id="191e1-308">In general, avoid using synchronous operations, and use asynchronous operations instead.</span></span> <span data-ttu-id="191e1-309">Pour plus d’informations, consultez [Pipelines et multiplexeurs](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).</span><span class="sxs-lookup"><span data-stu-id="191e1-309">For more information see [Pipelines and Multiplexers](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).</span></span>
>
>

### <a name="retry-usage-guidance"></a><span data-ttu-id="191e1-310">Guide d’utilisation des nouvelles tentatives</span><span class="sxs-lookup"><span data-stu-id="191e1-310">Retry usage guidance</span></span>
<span data-ttu-id="191e1-311">Respectez les consignes suivantes lors de l’utilisation du cache Redis Azure :</span><span class="sxs-lookup"><span data-stu-id="191e1-311">Consider the following guidelines when using Azure Redis Cache:</span></span>

* <span data-ttu-id="191e1-312">Le client StackExchange Redis gère ses propres nouvelles tentatives, mais uniquement lors de l’établissement d’une connexion au cache au premier démarrage de l’application.</span><span class="sxs-lookup"><span data-stu-id="191e1-312">The StackExchange Redis client manages its own retries, but only when establishing a connection to the cache when the application first starts.</span></span> <span data-ttu-id="191e1-313">Vous pouvez configurer le délai de connexion, le nombre de nouvelles tentatives et l’intervalle avant nouvelle tentative qui sont nécessaires pour établir cette connexion. Toutefois, la stratégie de nouvelle tentative ne s’applique pas aux opérations sur le cache.</span><span class="sxs-lookup"><span data-stu-id="191e1-313">You can configure the connection timeout, the number of retry attempts, and the time between retries to establish this connection, but the retry policy does not apply to operations against the cache.</span></span>
* <span data-ttu-id="191e1-314">Au lieu d’utiliser un grand nombre de nouvelles tentatives, envisagez une solution de secours en accédant plutôt à la source de données d’origine.</span><span class="sxs-lookup"><span data-stu-id="191e1-314">Instead of using a large number of retry attempts, consider falling back by accessing the original data source instead.</span></span>

### <a name="telemetry"></a><span data-ttu-id="191e1-315">Télémétrie</span><span class="sxs-lookup"><span data-stu-id="191e1-315">Telemetry</span></span>
<span data-ttu-id="191e1-316">Vous pouvez collecter des informations sur les connexions (mais pas sur d’autres opérations) à l’aide d’un élément **TextWriter**.</span><span class="sxs-lookup"><span data-stu-id="191e1-316">You can collect information about connections (but not other operations) using a **TextWriter**.</span></span>

```csharp
var writer = new StringWriter();
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="191e1-317">Vous trouverez ci-dessous un exemple de la sortie générée.</span><span class="sxs-lookup"><span data-stu-id="191e1-317">An example of the output this generates is shown below.</span></span>

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

### <a name="examples"></a><span data-ttu-id="191e1-318">Exemples</span><span class="sxs-lookup"><span data-stu-id="191e1-318">Examples</span></span>
<span data-ttu-id="191e1-319">L’exemple de code ci-après configure un délai (linéaire) constant entre les nouvelles tentatives lors de l’initialisation du client StackExchange.Redis.</span><span class="sxs-lookup"><span data-stu-id="191e1-319">The following code example configures a constant (linear) delay between retries when initializing the StackExchange.Redis client.</span></span> <span data-ttu-id="191e1-320">Cet exemple indique comment définir la configuration à l’aide d’une instance **ConfigurationOptions**.</span><span class="sxs-lookup"><span data-stu-id="191e1-320">This example shows how to set the configuration using a **ConfigurationOptions** instance.</span></span>

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

<span data-ttu-id="191e1-321">L’exemple ci-après définit la configuration en spécifiant les options sous forme de chaîne.</span><span class="sxs-lookup"><span data-stu-id="191e1-321">The next example sets the configuration by specifying the options as a string.</span></span> <span data-ttu-id="191e1-322">Le délai de connexion correspond au temps d’attente maximal autorisé pour la connexion au cache. Il ne s’agit pas du délai entre chaque nouvelle tentative.</span><span class="sxs-lookup"><span data-stu-id="191e1-322">The connection timeout is the maximum period of time to wait for a connection to the cache, not the delay between retry attempts.</span></span> <span data-ttu-id="191e1-323">Notez que la propriété **ReconnectRetryPolicy** est uniquement définissable par le biais du code.</span><span class="sxs-lookup"><span data-stu-id="191e1-323">Note that the **ReconnectRetryPolicy** property can only be set by code.</span></span>

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

<span data-ttu-id="191e1-324">Pour plus d’exemples, consultez [Configuration](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) sur le site web de projet.</span><span class="sxs-lookup"><span data-stu-id="191e1-324">For more examples, see [Configuration](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) on the project website.</span></span>

### <a name="more-information"></a><span data-ttu-id="191e1-325">Plus d’informations</span><span class="sxs-lookup"><span data-stu-id="191e1-325">More information</span></span>
* [<span data-ttu-id="191e1-326">Site web Redis</span><span class="sxs-lookup"><span data-stu-id="191e1-326">Redis website</span></span>](http://redis.io/)

## <a name="azure-search"></a><span data-ttu-id="191e1-327">Recherche Azure</span><span class="sxs-lookup"><span data-stu-id="191e1-327">Azure Search</span></span>
<span data-ttu-id="191e1-328">Azure Search permet d’ajouter des fonctionnalités de recherche puissantes et sophistiquées à un site web ou une application, d’ajuster rapidement et aisément les résultats de recherche, et de construire des modèles de classement enrichis et optimisés.</span><span class="sxs-lookup"><span data-stu-id="191e1-328">Azure Search can be used to add powerful and sophisticated search capabilities to a website or application, quickly and easily tune search results, and construct rich and fine-tuned ranking models.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="191e1-329">Mécanisme de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="191e1-329">Retry mechanism</span></span>
<span data-ttu-id="191e1-330">Le comportement de nouvelle tentative dans le Kit de développement logiciel (SDK) Recherche Azure est contrôlé par la méthode `SetRetryPolicy` sur les classes [SearchServiceClient] et [SearchIndexClient].</span><span class="sxs-lookup"><span data-stu-id="191e1-330">Retry behavior in the Azure Search SDK is controlled by the `SetRetryPolicy` method on the [SearchServiceClient] and [SearchIndexClient] classes.</span></span> <span data-ttu-id="191e1-331">La stratégie par défaut effectue une nouvelle tentative avec temporisation exponentielle lorsque Recherche Azure renvoie une réponse 5xx ou 408 (Délai d’expiration de la demande).</span><span class="sxs-lookup"><span data-stu-id="191e1-331">The default policy retries with exponential backoff when Azure Search returns a 5xx or 408 (Request Timeout) response.</span></span>

### <a name="telemetry"></a><span data-ttu-id="191e1-332">Télémétrie</span><span class="sxs-lookup"><span data-stu-id="191e1-332">Telemetry</span></span>
<span data-ttu-id="191e1-333">Effectuez le suivi avec ETW ou via l’inscription d’un fournisseur de suivi personnalisé.</span><span class="sxs-lookup"><span data-stu-id="191e1-333">Trace with ETW or by registering a custom trace provider.</span></span> <span data-ttu-id="191e1-334">Pour plus d’informations, consultez la [documentation AutoRest][autorest].</span><span class="sxs-lookup"><span data-stu-id="191e1-334">For more information, see the [AutoRest documentation][autorest].</span></span>

## <a name="service-bus"></a><span data-ttu-id="191e1-335">Service Bus</span><span class="sxs-lookup"><span data-stu-id="191e1-335">Service Bus</span></span>
<span data-ttu-id="191e1-336">Service Bus est une plate-forme de messagerie cloud qui propose l’échange de messages d’une façon faiblement couplée avec une amélioration de la mise à l’échelle et de la résilience pour les composants d’une application, si cette dernière est hébergée dans le cloud ou sur site.</span><span class="sxs-lookup"><span data-stu-id="191e1-336">Service Bus is a cloud messaging platform that provides loosely coupled message exchange with improved scale and resiliency for components of an application, whether hosted in the cloud or on-premises.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="191e1-337">Mécanisme de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="191e1-337">Retry mechanism</span></span>
<span data-ttu-id="191e1-338">Service Bus met en œuvre des nouvelles tentatives à l’aide d’implémentations de la classe de base [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) .</span><span class="sxs-lookup"><span data-stu-id="191e1-338">Service Bus implements retries using implementations of the [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) base class.</span></span> <span data-ttu-id="191e1-339">Tous les clients Service Bus exposent une propriété **RetryPolicy** qui peut être définie sur une des implémentations de la classe de base **RetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="191e1-339">All of the Service Bus clients expose a **RetryPolicy** property that can be set to one of the implementations of the **RetryPolicy** base class.</span></span> <span data-ttu-id="191e1-340">Les implémentations intégrées sont :</span><span class="sxs-lookup"><span data-stu-id="191e1-340">The built-in implementations are:</span></span>

* <span data-ttu-id="191e1-341">La [classe RetryExponential](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx).</span><span class="sxs-lookup"><span data-stu-id="191e1-341">The [RetryExponential Class](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx).</span></span> <span data-ttu-id="191e1-342">Elle expose les propriétés qui contrôlent l’intervalle de temporisation, le nombre de nouvelles tentatives et la propriété **TerminationTimeBuffer** utilisée pour limiter la durée totale nécessaire pour terminer l’opération.</span><span class="sxs-lookup"><span data-stu-id="191e1-342">This exposes properties that control the back-off interval, the retry count, and the **TerminationTimeBuffer** property that is used to limit the total time for the operation to complete.</span></span>
* <span data-ttu-id="191e1-343">La [classe NoRetry](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx).</span><span class="sxs-lookup"><span data-stu-id="191e1-343">The [NoRetry Class](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx).</span></span> <span data-ttu-id="191e1-344">Elle est utilisée lorsque de nouvelles tentatives au niveau de l’API Service Bus ne sont pas requises, notamment lorsque les nouvelles tentatives sont gérées par un autre processus dans le cadre d’un lot ou d’une opération en plusieurs étapes.</span><span class="sxs-lookup"><span data-stu-id="191e1-344">This is used when retries at the Service Bus API level are not required, such as when retries are managed by another process as part of a batch or multiple step operation.</span></span>

<span data-ttu-id="191e1-345">Les actions de Service Bus peuvent renvoyer une plage d’exceptions, comme celles répertoriées dans [Annexe : Exceptions de messagerie](http://msdn.microsoft.com/library/hh418082.aspx).</span><span class="sxs-lookup"><span data-stu-id="191e1-345">Service Bus actions can return a range of exceptions, as listed in [Appendix: Messaging Exceptions](http://msdn.microsoft.com/library/hh418082.aspx).</span></span> <span data-ttu-id="191e1-346">La liste fournit plus d’informations sur ces exceptions si celles-ci indiquent qu’une nouvelle tentative de l’opération est requise.</span><span class="sxs-lookup"><span data-stu-id="191e1-346">The list provides information about which if these indicate that retrying the operation is appropriate.</span></span> <span data-ttu-id="191e1-347">Par exemple, une exception [ServerBusyException](http://msdn.microsoft.com/library/microsoft.servicebus.messaging.serverbusyexception.aspx) indique que le client doit attendre un certain temps, puis recommencer l’opération.</span><span class="sxs-lookup"><span data-stu-id="191e1-347">For example, a [ServerBusyException](http://msdn.microsoft.com/library/microsoft.servicebus.messaging.serverbusyexception.aspx) indicates that the client should wait for a period of time, then retry the operation.</span></span> <span data-ttu-id="191e1-348">La survenue d’une exception **ServerBusyException** fait également basculer le Service Bus vers un mode différent, dans lequel un délai supplémentaire de 10 secondes est ajouté aux délais de nouvelle tentative calculés.</span><span class="sxs-lookup"><span data-stu-id="191e1-348">The occurrence of a **ServerBusyException** also causes Service Bus to switch to a different mode, in which an extra 10-second delay is added to the computed retry delays.</span></span> <span data-ttu-id="191e1-349">Ce mode est réinitialisé au bout d’une courte période.</span><span class="sxs-lookup"><span data-stu-id="191e1-349">This mode is reset after a short period.</span></span>

<span data-ttu-id="191e1-350">Les exceptions renvoyées à partir du Service Bus exposent la propriété **IsTransient** qui indique si le client doit retenter l’opération.</span><span class="sxs-lookup"><span data-stu-id="191e1-350">The exceptions returned from Service Bus expose the **IsTransient** property that indicates if the client should retry the operation.</span></span> <span data-ttu-id="191e1-351">La stratégie **RetryExponential** intégrée repose sur la propriété **IsTransient** de la classe **MessagingException**, qui est la classe de base pour toutes les exceptions Service Bus.</span><span class="sxs-lookup"><span data-stu-id="191e1-351">The built-in **RetryExponential** policy relies on the **IsTransient** property in the **MessagingException** class, which is the base class for all Service Bus exceptions.</span></span> <span data-ttu-id="191e1-352">Si vous créez des implémentations personnalisées de la classe de base **RetryPolicy**, vous pouvez combiner le type d’exception et la propriété **IsTransient** pour permettre un contrôle plus précis des actions liées aux nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="191e1-352">If you create custom implementations of the **RetryPolicy** base class you could use a combination of the exception type and the **IsTransient** property to provide more fine-grained control over retry actions.</span></span> <span data-ttu-id="191e1-353">Par exemple, vous pouvez détecter une exception **QuotaExceededException** et prendre des mesures pour vider la file d’attente avant de tenter de lui renvoyer un message.</span><span class="sxs-lookup"><span data-stu-id="191e1-353">For example, you could detect a **QuotaExceededException** and take action to drain the queue before retrying sending a message to it.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="191e1-354">Configuration de la stratégie</span><span class="sxs-lookup"><span data-stu-id="191e1-354">Policy configuration</span></span>
<span data-ttu-id="191e1-355">Les stratégies de nouvelle tentative sont définies par programme et peuvent être considérées comme stratégie par défaut pour un **NamespaceManager** et un **MessagingFactory**, ou individuellement pour chaque client de messagerie.</span><span class="sxs-lookup"><span data-stu-id="191e1-355">Retry policies are set programmatically, and can be set as a default policy for a **NamespaceManager** and for a **MessagingFactory**, or individually for each messaging client.</span></span> <span data-ttu-id="191e1-356">Pour définir la stratégie de nouvelle tentative par défaut pour une session de messagerie, vous définissez la propriété **RetryPolicy** de l’élément **NamespaceManager**.</span><span class="sxs-lookup"><span data-stu-id="191e1-356">To set the default retry policy for a messaging session you set the **RetryPolicy** of the **NamespaceManager**.</span></span>

```csharp
namespaceManager.Settings.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                                maxBackoff: TimeSpan.FromSeconds(30),
                                                                maxRetryCount: 3);
```

<span data-ttu-id="191e1-357">Pour définir la stratégie de nouvelle tentative par défaut pour tous les clients créée à partir d’une fabrique de messagerie, vous définissez l’élément **RetryPolicy** de **MessagingFactory**.</span><span class="sxs-lookup"><span data-stu-id="191e1-357">To set the default retry policy for all clients created from a messaging factory, you set the **RetryPolicy** of the **MessagingFactory**.</span></span>

```csharp
messagingFactory.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                    maxBackoff: TimeSpan.FromSeconds(30),
                                                    maxRetryCount: 3);
```

<span data-ttu-id="191e1-358">Pour définir la stratégie de nouvelle tentative pour un client de messagerie ou remplacer sa stratégie par défaut, vous définissez sa propriété **RetryPolicy** à l’aide d’une instance de la classe de stratégie requise :</span><span class="sxs-lookup"><span data-stu-id="191e1-358">To set the retry policy for a messaging client, or to override its default policy, you set its **RetryPolicy** property using an instance of the required policy class:</span></span>

```csharp
client.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                            maxBackoff: TimeSpan.FromSeconds(30),
                                            maxRetryCount: 3);
```

<span data-ttu-id="191e1-359">Impossible de définir la stratégie de nouvelle tentative au niveau de l’opération individuelle.</span><span class="sxs-lookup"><span data-stu-id="191e1-359">The retry policy cannot be set at the individual operation level.</span></span> <span data-ttu-id="191e1-360">Elle s’applique à toutes les opérations pour le client de messagerie.</span><span class="sxs-lookup"><span data-stu-id="191e1-360">It applies to all operations for the messaging client.</span></span>
<span data-ttu-id="191e1-361">Le tableau suivant présente les paramètres par défaut pour la stratégie de nouvelle tentative intégrée.</span><span class="sxs-lookup"><span data-stu-id="191e1-361">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="191e1-362">Paramètre</span><span class="sxs-lookup"><span data-stu-id="191e1-362">Setting</span></span> | <span data-ttu-id="191e1-363">Valeur par défaut</span><span class="sxs-lookup"><span data-stu-id="191e1-363">Default value</span></span> | <span data-ttu-id="191e1-364">Signification</span><span class="sxs-lookup"><span data-stu-id="191e1-364">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="191e1-365">Stratégie</span><span class="sxs-lookup"><span data-stu-id="191e1-365">Policy</span></span> | <span data-ttu-id="191e1-366">Exponentielle</span><span class="sxs-lookup"><span data-stu-id="191e1-366">Exponential</span></span> | <span data-ttu-id="191e1-367">Temporisation exponentielle.</span><span class="sxs-lookup"><span data-stu-id="191e1-367">Exponential back-off.</span></span> |
| <span data-ttu-id="191e1-368">MinimalBackoff</span><span class="sxs-lookup"><span data-stu-id="191e1-368">MinimalBackoff</span></span> | <span data-ttu-id="191e1-369">0</span><span class="sxs-lookup"><span data-stu-id="191e1-369">0</span></span> | <span data-ttu-id="191e1-370">Intervalle de temporisation minimal.</span><span class="sxs-lookup"><span data-stu-id="191e1-370">Minimum back-off interval.</span></span> <span data-ttu-id="191e1-371">Cette valeur est ajoutée à l’intervalle avant nouvelle tentative calculé à partir de deltaBackoff.</span><span class="sxs-lookup"><span data-stu-id="191e1-371">This is added to the retry interval computed from deltaBackoff.</span></span> |
| <span data-ttu-id="191e1-372">MaximumBackoff</span><span class="sxs-lookup"><span data-stu-id="191e1-372">MaximumBackoff</span></span> | <span data-ttu-id="191e1-373">30 secondes</span><span class="sxs-lookup"><span data-stu-id="191e1-373">30 seconds</span></span> | <span data-ttu-id="191e1-374">Intervalle de temporisation maximal.</span><span class="sxs-lookup"><span data-stu-id="191e1-374">Maximum back-off interval.</span></span> <span data-ttu-id="191e1-375">Le paramètre MaximumBackoff est utilisé si l’intervalle avant nouvelle tentative calculé est supérieur à la valeur MaxBackoff.</span><span class="sxs-lookup"><span data-stu-id="191e1-375">MaximumBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> |
| <span data-ttu-id="191e1-376">DeltaBackoff</span><span class="sxs-lookup"><span data-stu-id="191e1-376">DeltaBackoff</span></span> | <span data-ttu-id="191e1-377">3 secondes</span><span class="sxs-lookup"><span data-stu-id="191e1-377">3 seconds</span></span> | <span data-ttu-id="191e1-378">Intervalle de temporisation entre les tentatives.</span><span class="sxs-lookup"><span data-stu-id="191e1-378">Back-off interval between retries.</span></span> <span data-ttu-id="191e1-379">Des multiples de cette durée seront utilisés pour les tentatives suivantes.</span><span class="sxs-lookup"><span data-stu-id="191e1-379">Multiples of this timespan will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="191e1-380">TimeBuffer</span><span class="sxs-lookup"><span data-stu-id="191e1-380">TimeBuffer</span></span> | <span data-ttu-id="191e1-381">5 secondes</span><span class="sxs-lookup"><span data-stu-id="191e1-381">5 seconds</span></span> | <span data-ttu-id="191e1-382">Mémoire tampon de délai d’expiration associé à la nouvelle tentative.</span><span class="sxs-lookup"><span data-stu-id="191e1-382">The termination time buffer associated with the retry.</span></span> <span data-ttu-id="191e1-383">Les nouvelles tentatives seront abandonnées si le temps restant est inférieur à la valeur TimeBuffer.</span><span class="sxs-lookup"><span data-stu-id="191e1-383">Retry attempts will be abandoned if the remaining time is less than TimeBuffer.</span></span> |
| <span data-ttu-id="191e1-384">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="191e1-384">MaxRetryCount</span></span> | <span data-ttu-id="191e1-385">10</span><span class="sxs-lookup"><span data-stu-id="191e1-385">10</span></span> | <span data-ttu-id="191e1-386">Nombre maximal de nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="191e1-386">The maximum number of retries.</span></span> |
| <span data-ttu-id="191e1-387">ServerBusyBaseSleepTime</span><span class="sxs-lookup"><span data-stu-id="191e1-387">ServerBusyBaseSleepTime</span></span> | <span data-ttu-id="191e1-388">10 secondes</span><span class="sxs-lookup"><span data-stu-id="191e1-388">10 seconds</span></span> | <span data-ttu-id="191e1-389">Si la dernière exception rencontrée était **ServerBusyException**, cette valeur sera ajoutée à l’intervalle avant nouvelle tentative calculé.</span><span class="sxs-lookup"><span data-stu-id="191e1-389">If the last exception encountered was **ServerBusyException**, this value will be added to the computed retry interval.</span></span> <span data-ttu-id="191e1-390">Cette valeur ne peut pas être modifiée.</span><span class="sxs-lookup"><span data-stu-id="191e1-390">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="191e1-391">Guide d’utilisation des nouvelles tentatives</span><span class="sxs-lookup"><span data-stu-id="191e1-391">Retry usage guidance</span></span>
<span data-ttu-id="191e1-392">Respectez les consignes suivantes lors de l’utilisation de Service Bus :</span><span class="sxs-lookup"><span data-stu-id="191e1-392">Consider the following guidelines when using Service Bus:</span></span>

* <span data-ttu-id="191e1-393">Lors de l’utilisation de l’implémentation intégrée de l’élément **RetryExponential** , ne mettez pas en œuvre d’opération de secours dès que la stratégie réagit aux exceptions liées à un serveur occupé et bascule automatiquement vers un mode de nouvelle tentative approprié</span><span class="sxs-lookup"><span data-stu-id="191e1-393">When using the built-in **RetryExponential** implementation, do not implement a fallback operation as the policy reacts to Server Busy exceptions and automatically switches to an appropriate retry mode.</span></span>
* <span data-ttu-id="191e1-394">Service Bus prend en charge une fonctionnalité appelée Espaces de noms associés, qui met en œuvre le basculement automatique vers une file d’attente de sauvegarde dans un espace de noms séparé en cas d’échec de la file d’attente dans l’espace de noms principal.</span><span class="sxs-lookup"><span data-stu-id="191e1-394">Service Bus supports a feature called Paired Namespaces, which implements automatic failover to a backup queue in a separate namespace if the queue in the primary namespace fails.</span></span> <span data-ttu-id="191e1-395">Les messages de la file d’attente secondaire peuvent être renvoyés à la file d’attente principale lors de la récupération</span><span class="sxs-lookup"><span data-stu-id="191e1-395">Messages from the secondary queue can be sent back to the primary queue when it recovers.</span></span> <span data-ttu-id="191e1-396">Cette fonctionnalité permet de traiter les erreurs temporaires.</span><span class="sxs-lookup"><span data-stu-id="191e1-396">This feature helps to address transient failures.</span></span> <span data-ttu-id="191e1-397">Pour plus d’informations, consultez [Modèles de messagerie asynchrone et haute disponibilité](http://msdn.microsoft.com/library/azure/dn292562.aspx).</span><span class="sxs-lookup"><span data-stu-id="191e1-397">For more information, see [Asynchronous Messaging Patterns and High Availability](http://msdn.microsoft.com/library/azure/dn292562.aspx).</span></span>

<span data-ttu-id="191e1-398">Pensez à commencer par les paramètres suivants pour les opérations liées aux nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="191e1-398">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="191e1-399">Il s’agit de paramètres généraux. Vous devez par conséquent surveiller les opérations et optimiser les valeurs en fonction de votre propre scénario.</span><span class="sxs-lookup"><span data-stu-id="191e1-399">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="191e1-400">Context</span><span class="sxs-lookup"><span data-stu-id="191e1-400">Context</span></span> | <span data-ttu-id="191e1-401">Exemple de latence maximale</span><span class="sxs-lookup"><span data-stu-id="191e1-401">Example maximum latency</span></span> | <span data-ttu-id="191e1-402">Stratégie de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="191e1-402">Retry policy</span></span> | <span data-ttu-id="191e1-403">Paramètres</span><span class="sxs-lookup"><span data-stu-id="191e1-403">Settings</span></span> | <span data-ttu-id="191e1-404">Fonctionnement</span><span class="sxs-lookup"><span data-stu-id="191e1-404">How it works</span></span> |
|---------|---------|---------|---------|---------|
| <span data-ttu-id="191e1-405">Interactif, interface utilisateur ou premier plan</span><span class="sxs-lookup"><span data-stu-id="191e1-405">Interactive, UI, or foreground</span></span> | <span data-ttu-id="191e1-406">2 secondes\*</span><span class="sxs-lookup"><span data-stu-id="191e1-406">2 seconds\*</span></span>  | <span data-ttu-id="191e1-407">Exponentielle</span><span class="sxs-lookup"><span data-stu-id="191e1-407">Exponential</span></span> | <span data-ttu-id="191e1-408">MinimumBackoff = 0</span><span class="sxs-lookup"><span data-stu-id="191e1-408">MinimumBackoff = 0</span></span> <br/> <span data-ttu-id="191e1-409">MaximumBackoff = 30 s</span><span class="sxs-lookup"><span data-stu-id="191e1-409">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="191e1-410">DeltaBackoff = 300 ms</span><span class="sxs-lookup"><span data-stu-id="191e1-410">DeltaBackoff = 300 msec.</span></span> <br/> <span data-ttu-id="191e1-411">TimeBuffer = 300 ms</span><span class="sxs-lookup"><span data-stu-id="191e1-411">TimeBuffer = 300 msec.</span></span> <br/> <span data-ttu-id="191e1-412">MaxRetryCount = 2</span><span class="sxs-lookup"><span data-stu-id="191e1-412">MaxRetryCount = 2</span></span> | <span data-ttu-id="191e1-413">Tentative 1 : délai 0 s</span><span class="sxs-lookup"><span data-stu-id="191e1-413">Attempt 1: Delay 0 sec.</span></span> <br/> <span data-ttu-id="191e1-414">Tentative 2 : délai ~ 300 ms</span><span class="sxs-lookup"><span data-stu-id="191e1-414">Attempt 2: Delay ~300 msec.</span></span> <br/> <span data-ttu-id="191e1-415">Tentative 3 : délai ~ 900 ms</span><span class="sxs-lookup"><span data-stu-id="191e1-415">Attempt 3: Delay ~900 msec.</span></span> |
| <span data-ttu-id="191e1-416">Arrière-plan ou lot</span><span class="sxs-lookup"><span data-stu-id="191e1-416">Background or batch</span></span> | <span data-ttu-id="191e1-417">30 secondes</span><span class="sxs-lookup"><span data-stu-id="191e1-417">30 seconds</span></span> | <span data-ttu-id="191e1-418">Exponentielle</span><span class="sxs-lookup"><span data-stu-id="191e1-418">Exponential</span></span> | <span data-ttu-id="191e1-419">MinimumBackoff = 1</span><span class="sxs-lookup"><span data-stu-id="191e1-419">MinimumBackoff = 1</span></span> <br/> <span data-ttu-id="191e1-420">MaximumBackoff = 30 s</span><span class="sxs-lookup"><span data-stu-id="191e1-420">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="191e1-421">DeltaBackoff = 1,75 s</span><span class="sxs-lookup"><span data-stu-id="191e1-421">DeltaBackoff = 1.75 sec.</span></span> <br/> <span data-ttu-id="191e1-422">TimeBuffer = 5 s</span><span class="sxs-lookup"><span data-stu-id="191e1-422">TimeBuffer = 5 sec.</span></span> <br/> <span data-ttu-id="191e1-423">MaxRetryCount = 3</span><span class="sxs-lookup"><span data-stu-id="191e1-423">MaxRetryCount = 3</span></span> | <span data-ttu-id="191e1-424">Tentative 1 : délai ~ 1 s</span><span class="sxs-lookup"><span data-stu-id="191e1-424">Attempt 1: Delay ~1 sec.</span></span> <br/> <span data-ttu-id="191e1-425">Tentative 2 : délai ~ 3 s</span><span class="sxs-lookup"><span data-stu-id="191e1-425">Attempt 2: Delay ~3 sec.</span></span> <br/> <span data-ttu-id="191e1-426">Tentative 3 : délai ~ 6 ms</span><span class="sxs-lookup"><span data-stu-id="191e1-426">Attempt 3: Delay ~6 msec.</span></span> <br/> <span data-ttu-id="191e1-427">Tentative 4 : délai ~ 13 ms</span><span class="sxs-lookup"><span data-stu-id="191e1-427">Attempt 4: Delay ~13 msec.</span></span> |

<span data-ttu-id="191e1-428">\* N’inclut pas le délai supplémentaire qui s’ajoute en cas de réception d’une réponse du type Serveur occupé.</span><span class="sxs-lookup"><span data-stu-id="191e1-428">\* Not including additional delay that is added if a Server Busy response is received.</span></span>

### <a name="telemetry"></a><span data-ttu-id="191e1-429">Télémétrie</span><span class="sxs-lookup"><span data-stu-id="191e1-429">Telemetry</span></span>
<span data-ttu-id="191e1-430">Service Bus enregistre les nouvelles tentatives sous forme d’événements ETW à l’aide d’ **EventSource**.</span><span class="sxs-lookup"><span data-stu-id="191e1-430">Service Bus logs retries as ETW events using an **EventSource**.</span></span> <span data-ttu-id="191e1-431">Vous devez associer un élément **EventListener** à la source d’événement pour capturer les événements et les afficher dans la visionneuse de performances ou les écrire dans un journal de destination approprié.</span><span class="sxs-lookup"><span data-stu-id="191e1-431">You must attach an **EventListener** to the event source to capture the events and view them in Performance Viewer, or write them to a suitable destination log.</span></span> <span data-ttu-id="191e1-432">Vous pouvez utiliser le [bloc applicatif de journalisation sémantique](http://msdn.microsoft.com/library/dn775006.aspx) pour effectuer cette opération.</span><span class="sxs-lookup"><span data-stu-id="191e1-432">You could use the [Semantic Logging Application Block](http://msdn.microsoft.com/library/dn775006.aspx) to do this.</span></span> <span data-ttu-id="191e1-433">Les événements de nouvelle tentative se présentent sous la forme suivante :</span><span class="sxs-lookup"><span data-stu-id="191e1-433">The retry events are of the following form:</span></span>

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

### <a name="examples"></a><span data-ttu-id="191e1-434">Exemples</span><span class="sxs-lookup"><span data-stu-id="191e1-434">Examples</span></span>
<span data-ttu-id="191e1-435">L’exemple de code suivant montre comment définir la stratégie de nouvelle tentative pour :</span><span class="sxs-lookup"><span data-stu-id="191e1-435">The following code example shows how to set the retry policy for:</span></span>

* <span data-ttu-id="191e1-436">Un gestionnaire d’espace de noms.</span><span class="sxs-lookup"><span data-stu-id="191e1-436">A namespace manager.</span></span> <span data-ttu-id="191e1-437">La stratégie s’applique à toutes les opérations sur ce gestionnaire et ne peut pas être remplacée pour des opérations individuelles.</span><span class="sxs-lookup"><span data-stu-id="191e1-437">The policy applies to all operations on that manager, and cannot be overridden for individual operations.</span></span>
* <span data-ttu-id="191e1-438">Une fabrique de messagerie.</span><span class="sxs-lookup"><span data-stu-id="191e1-438">A messaging factory.</span></span> <span data-ttu-id="191e1-439">La stratégie s’applique à tous les clients créés à partir de cette fabrique et ne peut pas être remplacée lors de la création de clients individuels.</span><span class="sxs-lookup"><span data-stu-id="191e1-439">The policy applies to all clients created from that factory, and cannot be overridden when creating individual clients.</span></span>
* <span data-ttu-id="191e1-440">Un client de messagerie individuel.</span><span class="sxs-lookup"><span data-stu-id="191e1-440">An individual messaging client.</span></span> <span data-ttu-id="191e1-441">Après avoir créé un client, vous pouvez définir la stratégie de nouvelle tentative pour ce client.</span><span class="sxs-lookup"><span data-stu-id="191e1-441">After a client has been created, you can set the retry policy for that client.</span></span> <span data-ttu-id="191e1-442">La stratégie s’applique à toutes les opérations sur ce client.</span><span class="sxs-lookup"><span data-stu-id="191e1-442">The policy applies to all operations on that client.</span></span>

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

### <a name="more-information"></a><span data-ttu-id="191e1-443">Plus d’informations</span><span class="sxs-lookup"><span data-stu-id="191e1-443">More information</span></span>
* [<span data-ttu-id="191e1-444">Modèles de messagerie asynchrone et haute disponibilité</span><span class="sxs-lookup"><span data-stu-id="191e1-444">Asynchronous Messaging Patterns and High Availability</span></span>](http://msdn.microsoft.com/library/azure/dn292562.aspx)

## <a name="service-fabric"></a><span data-ttu-id="191e1-445">Service Fabric</span><span class="sxs-lookup"><span data-stu-id="191e1-445">Service Fabric</span></span>

<span data-ttu-id="191e1-446">La distribution de services fiables dans un cluster Service Fabric offre une protection contre la plupart des erreurs temporaires potentielles décrites dans cet article.</span><span class="sxs-lookup"><span data-stu-id="191e1-446">Distributing reliable services in a Service Fabric cluster guards against most of the potential transient faults discussed in this article.</span></span> <span data-ttu-id="191e1-447">Toutefois, certaines erreurs temporaires restent possibles.</span><span class="sxs-lookup"><span data-stu-id="191e1-447">Some transient faults are still possible, however.</span></span> <span data-ttu-id="191e1-448">Par exemple, si le service d’affectation de noms reçoit une requête au moment précis où il fait l’objet d’un changement de routage, il lève une exception.</span><span class="sxs-lookup"><span data-stu-id="191e1-448">For example, the naming service might be in the middle of a routing change when it gets a request, causing it to throw an exception.</span></span> <span data-ttu-id="191e1-449">En revanche, si la même requête arrive 100 millisecondes plus tard, elle aboutira probablement.</span><span class="sxs-lookup"><span data-stu-id="191e1-449">If the same request comes 100 milliseconds later, it will probably succeed.</span></span>

<span data-ttu-id="191e1-450">Service Fabric gère ce type d’erreur temporaire en interne.</span><span class="sxs-lookup"><span data-stu-id="191e1-450">Internally, Service Fabric manages this kind of transient fault.</span></span> <span data-ttu-id="191e1-451">Vous pouvez configurer certains paramètres à l’aide de la classe `OperationRetrySettings` lorsque vous configurez vos services.</span><span class="sxs-lookup"><span data-stu-id="191e1-451">You can configure some settings by using the `OperationRetrySettings` class while setting up your services.</span></span>  <span data-ttu-id="191e1-452">Le code ci-après présente un exemple.</span><span class="sxs-lookup"><span data-stu-id="191e1-452">The following code shows an example.</span></span> <span data-ttu-id="191e1-453">Dans la plupart des cas, cette opération n’est pas nécessaire, et les paramètres par défaut font l’affaire.</span><span class="sxs-lookup"><span data-stu-id="191e1-453">In most cases, this should not be necessary, and the default settings will be fine.</span></span>

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

### <a name="more-information"></a><span data-ttu-id="191e1-454">Plus d’informations</span><span class="sxs-lookup"><span data-stu-id="191e1-454">More information</span></span>

* [<span data-ttu-id="191e1-455">Gestion des exceptions à distance</span><span class="sxs-lookup"><span data-stu-id="191e1-455">Remote Exception Handling</span></span>](https://github.com/Microsoft/azure-docs/blob/master/articles/service-fabric/service-fabric-reliable-services-communication-remoting.md#remoting-exception-handling)

## <a name="sql-database-using-adonet"></a><span data-ttu-id="191e1-456">Base de données SQL utilisant ADO.NET</span><span class="sxs-lookup"><span data-stu-id="191e1-456">SQL Database using ADO.NET</span></span>
<span data-ttu-id="191e1-457">La base de données SQL est une base de données SQL hébergée disponible en différentes tailles et sous forme de service standard (partagé) et premium (non partagé).</span><span class="sxs-lookup"><span data-stu-id="191e1-457">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="191e1-458">Mécanisme de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="191e1-458">Retry mechanism</span></span>
<span data-ttu-id="191e1-459">Les bases de données SQL n’intègrent aucune prise en charge pour les nouvelles tentatives en cas d’accès à l’aide d’ADO.NET.</span><span class="sxs-lookup"><span data-stu-id="191e1-459">SQL Database has no built-in support for retries when accessed using ADO.NET.</span></span> <span data-ttu-id="191e1-460">Toutefois, les codes de retour issus des demandes peuvent être utilisés pour déterminer le motif d’échec de ces dernières.</span><span class="sxs-lookup"><span data-stu-id="191e1-460">However, the return codes from requests can be used to determine why a request failed.</span></span> <span data-ttu-id="191e1-461">Pour plus d’informations sur la limitation de SQL Database, consultez l’article [Limites de ressources de base de données SQL Azure](/azure/sql-database/sql-database-resource-limits).</span><span class="sxs-lookup"><span data-stu-id="191e1-461">For more information about SQL Database throttling, see [Azure SQL Database resource limits](/azure/sql-database/sql-database-resource-limits).</span></span> <span data-ttu-id="191e1-462">Pour obtenir la liste des codes d’erreur pertinents, consultez l’article [Codes d’erreur SQL pour les applications clientes SQL Database](/azure/sql-database/sql-database-develop-error-messages).</span><span class="sxs-lookup"><span data-stu-id="191e1-462">For a list of relevant error codes, see [SQL error codes for SQL Database client applications](/azure/sql-database/sql-database-develop-error-messages).</span></span>

<span data-ttu-id="191e1-463">Vous pouvez utiliser la bibliothèque Polly afin d’implémenter le mécanisme de nouvelles tentatives pour SQL Database.</span><span class="sxs-lookup"><span data-stu-id="191e1-463">You can use the Polly library to implement retries for SQL Database.</span></span> <span data-ttu-id="191e1-464">Consultez la section [Gestion des erreurs temporaires avec Polly](#transient-fault-handling-with-polly).</span><span class="sxs-lookup"><span data-stu-id="191e1-464">See [Transient fault handling with Polly](#transient-fault-handling-with-polly).</span></span>

### <a name="retry-usage-guidance"></a><span data-ttu-id="191e1-465">Guide d’utilisation des nouvelles tentatives</span><span class="sxs-lookup"><span data-stu-id="191e1-465">Retry usage guidance</span></span>
<span data-ttu-id="191e1-466">Respectez les consignes suivantes lorsque vous accédez à la base de données SQL à l’aide d’ADO.NET :</span><span class="sxs-lookup"><span data-stu-id="191e1-466">Consider the following guidelines when accessing SQL Database using ADO.NET:</span></span>

* <span data-ttu-id="191e1-467">Choisissez l’option de service appropriée (partagée ou premium).</span><span class="sxs-lookup"><span data-stu-id="191e1-467">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="191e1-468">Une instance partagée peut être soumise à des délais de connexion plus longs que d’ordinaire et à des limitations en raison de l’utilisation du serveur partagé par d’autres locataires.</span><span class="sxs-lookup"><span data-stu-id="191e1-468">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="191e1-469">Si des performances prévisibles et des opérations fiables à faible latence sont requises, pensez à choisir l’option premium.</span><span class="sxs-lookup"><span data-stu-id="191e1-469">If more predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="191e1-470">Veillez à effectuer de nouvelles tentatives au niveau ou à l’étendue qui convient pour éviter les opérations non idempotent à l’origine d’une incohérence dans les données.</span><span class="sxs-lookup"><span data-stu-id="191e1-470">Ensure that you perform retries at the appropriate level or scope to avoid non-idempotent operations causing inconsistency in the data.</span></span> <span data-ttu-id="191e1-471">Idéalement, toutes les opérations doivent être idempotent afin qu’elles puissent être répétées sans entraîner d’incohérence.</span><span class="sxs-lookup"><span data-stu-id="191e1-471">Ideally, all operations should be idempotent so that they can be repeated without causing inconsistency.</span></span> <span data-ttu-id="191e1-472">Lorsque cela n’est pas le cas, la nouvelle tentative doit être effectuée à un niveau ou à une étendue qui autorise l’annulation de toutes les modifications associées en cas d’échec d’une opération ; par exemple, depuis une étendue transactionnelle.</span><span class="sxs-lookup"><span data-stu-id="191e1-472">Where this is not the case, the retry should be performed at a level or scope that allows all related changes to be undone if one operation fails; for example, from within a transactional scope.</span></span> <span data-ttu-id="191e1-473">Pour plus d’informations, consultez [Couche d’accès aux données de Cloud Service Fundamentals - Gestion des erreurs temporaires](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).</span><span class="sxs-lookup"><span data-stu-id="191e1-473">For more information, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).</span></span>
* <span data-ttu-id="191e1-474">Une stratégie d’intervalle fixe n’est pas recommandée pour une utilisation avec la base de données SQL Azure à l’exception des scénarios interactifs comportant seulement quelques nouvelles tentatives à intervalles très courts.</span><span class="sxs-lookup"><span data-stu-id="191e1-474">A fixed interval strategy is not recommended for use with Azure SQL Database except for interactive scenarios where there are only a few retries at very short intervals.</span></span> <span data-ttu-id="191e1-475">Envisagez plutôt d’utiliser une stratégie de temporisation exponentielle pour la majorité des scénarios.</span><span class="sxs-lookup"><span data-stu-id="191e1-475">Instead, consider using an exponential back-off strategy for the majority of scenarios.</span></span>
* <span data-ttu-id="191e1-476">Choisissez une valeur appropriée pour les délais d’attente de connexion et de commande lors de la définition des connexions.</span><span class="sxs-lookup"><span data-stu-id="191e1-476">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="191e1-477">Un délai d’attente trop court peut entraîner des échecs de connexion prématurés lorsque la base de données est occupée.</span><span class="sxs-lookup"><span data-stu-id="191e1-477">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="191e1-478">Un délai d’attente trop long peut empêcher la logique de nouvelle tentative de fonctionner correctement en attendant trop longtemps avant la détection d’un échec de connexion.</span><span class="sxs-lookup"><span data-stu-id="191e1-478">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="191e1-479">La valeur du délai d’attente est un composant de la latence de bout en bout ; Il est en fait ajouté au délai de nouvelle tentative spécifié dans la stratégie de nouvelle tentative pour chaque nouvelle tentative.</span><span class="sxs-lookup"><span data-stu-id="191e1-479">The value of the timeout is a component of the end-to-end latency; it is effectively added to the retry delay specified in the retry policy for every retry attempt.</span></span>
* <span data-ttu-id="191e1-480">Fermez la connexion après un certain nombre de nouvelles tentatives, même lorsque vous utilisez une logique de nouvelle tentative à temporisation exponentielle et renouvelez l’opération sur une nouvelle connexion.</span><span class="sxs-lookup"><span data-stu-id="191e1-480">Close the connection after a certain number of retries, even when using an exponential back off retry logic, and retry the operation on a new connection.</span></span> <span data-ttu-id="191e1-481">Le fait d’effectuer la même opération à plusieurs reprises sur la même connexion peut être un facteur entraînant des problèmes de connexion.</span><span class="sxs-lookup"><span data-stu-id="191e1-481">Retrying the same operation multiple times on the same connection can be a factor that contributes to connection problems.</span></span> <span data-ttu-id="191e1-482">Pour voir un exemple de cette technique, consultez [Couche d’accès aux données de Cloud Service Fundamentals - Gestion des erreurs temporaires](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).</span><span class="sxs-lookup"><span data-stu-id="191e1-482">For an example of this technique, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).</span></span>
* <span data-ttu-id="191e1-483">Lorsque le regroupement de connexions est en cours d’utilisation (par défaut), il est probable que la même connexion sera choisie à partir du pool, même après la fermeture et la réouverture d’une connexion.</span><span class="sxs-lookup"><span data-stu-id="191e1-483">When connection pooling is in use (the default) there is a chance that the same connection will be chosen from the pool, even after closing and reopening a connection.</span></span> <span data-ttu-id="191e1-484">Si c’est le cas, une technique permettant de résoudre ce problème consiste à appeler la méthode **ClearPool** de la classe **SqlConnection** pour marquer la connexion comme n’étant pas réutilisable.</span><span class="sxs-lookup"><span data-stu-id="191e1-484">If this is the case, a technique to resolve it is to call the **ClearPool** method of the **SqlConnection** class to mark the connection as not reusable.</span></span> <span data-ttu-id="191e1-485">Toutefois, vous ne devez effectuer cette opération qu’après l’échec de plusieurs tentatives de connexion, et uniquement lorsque vous rencontrez la classe spécifique d’erreurs temporaires, relative notamment à des délais d’attente SQL (code d’erreur -2) liés à des connexions défectueuses.</span><span class="sxs-lookup"><span data-stu-id="191e1-485">However, you should do this only after several connection attempts have failed, and only when encountering the specific class of transient failures such as SQL timeouts (error code -2) related to faulty connections.</span></span>
* <span data-ttu-id="191e1-486">Si le code d’accès aux données utilise des transactions lancées en tant qu’instances **TransactionScope** , la logique de nouvelle tentative doit ouvrir de nouveau la connexion et lancer une nouvelle étendue de transaction.</span><span class="sxs-lookup"><span data-stu-id="191e1-486">If the data access code uses transactions initiated as **TransactionScope** instances, the retry logic should reopen the connection and initiate a new transaction scope.</span></span> <span data-ttu-id="191e1-487">Pour cette raison, le bloc de code renouvelable doit englober l’ensemble de la portée de la transaction.</span><span class="sxs-lookup"><span data-stu-id="191e1-487">For this reason, the retryable code block should encompass the entire scope of the transaction.</span></span>

<span data-ttu-id="191e1-488">Pensez à commencer par les paramètres suivants pour les opérations liées aux nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="191e1-488">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="191e1-489">Il s’agit de paramètres généraux. Vous devez par conséquent surveiller les opérations et optimiser les valeurs en fonction de votre propre scénario.</span><span class="sxs-lookup"><span data-stu-id="191e1-489">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="191e1-490">**Contexte**</span><span class="sxs-lookup"><span data-stu-id="191e1-490">**Context**</span></span> | <span data-ttu-id="191e1-491">**Exemple de cible E2E<br />latence max**</span><span class="sxs-lookup"><span data-stu-id="191e1-491">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="191e1-492">**Stratégie de nouvelle tentative**</span><span class="sxs-lookup"><span data-stu-id="191e1-492">**Retry strategy**</span></span> | <span data-ttu-id="191e1-493">**Paramètres**</span><span class="sxs-lookup"><span data-stu-id="191e1-493">**Settings**</span></span> | <span data-ttu-id="191e1-494">**Valeurs**</span><span class="sxs-lookup"><span data-stu-id="191e1-494">**Values**</span></span> | <span data-ttu-id="191e1-495">**Fonctionnement**</span><span class="sxs-lookup"><span data-stu-id="191e1-495">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="191e1-496">Interactif, interface utilisateur,</span><span class="sxs-lookup"><span data-stu-id="191e1-496">Interactive, UI,</span></span><br /><span data-ttu-id="191e1-497">ou premier plan</span><span class="sxs-lookup"><span data-stu-id="191e1-497">or foreground</span></span> |<span data-ttu-id="191e1-498">2 secondes</span><span class="sxs-lookup"><span data-stu-id="191e1-498">2 sec</span></span> |<span data-ttu-id="191e1-499">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="191e1-499">FixedInterval</span></span> |<span data-ttu-id="191e1-500">Nombre de tentatives</span><span class="sxs-lookup"><span data-stu-id="191e1-500">Retry count</span></span><br /><span data-ttu-id="191e1-501">Intervalle avant nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="191e1-501">Retry interval</span></span><br /><span data-ttu-id="191e1-502">Première nouvelle tentative rapide</span><span class="sxs-lookup"><span data-stu-id="191e1-502">First fast retry</span></span> |<span data-ttu-id="191e1-503">3</span><span class="sxs-lookup"><span data-stu-id="191e1-503">3</span></span><br /><span data-ttu-id="191e1-504">500 ms</span><span class="sxs-lookup"><span data-stu-id="191e1-504">500 ms</span></span><br /><span data-ttu-id="191e1-505">true</span><span class="sxs-lookup"><span data-stu-id="191e1-505">true</span></span> |<span data-ttu-id="191e1-506">Tentative 1 - délai 0 s</span><span class="sxs-lookup"><span data-stu-id="191e1-506">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="191e1-507">Tentative 2 - délai 500 ms</span><span class="sxs-lookup"><span data-stu-id="191e1-507">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="191e1-508">Tentative 3 - délai 500 ms</span><span class="sxs-lookup"><span data-stu-id="191e1-508">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="191e1-509">Arrière-plan</span><span class="sxs-lookup"><span data-stu-id="191e1-509">Background</span></span><br /><span data-ttu-id="191e1-510">ou lot</span><span class="sxs-lookup"><span data-stu-id="191e1-510">or batch</span></span> |<span data-ttu-id="191e1-511">30 secondes</span><span class="sxs-lookup"><span data-stu-id="191e1-511">30 sec</span></span> |<span data-ttu-id="191e1-512">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="191e1-512">ExponentialBackoff</span></span> |<span data-ttu-id="191e1-513">Nombre de tentatives</span><span class="sxs-lookup"><span data-stu-id="191e1-513">Retry count</span></span><br /><span data-ttu-id="191e1-514">Temporisation min</span><span class="sxs-lookup"><span data-stu-id="191e1-514">Min back-off</span></span><br /><span data-ttu-id="191e1-515">Temporisation max</span><span class="sxs-lookup"><span data-stu-id="191e1-515">Max back-off</span></span><br /><span data-ttu-id="191e1-516">Temporisation delta</span><span class="sxs-lookup"><span data-stu-id="191e1-516">Delta back-off</span></span><br /><span data-ttu-id="191e1-517">Première nouvelle tentative rapide</span><span class="sxs-lookup"><span data-stu-id="191e1-517">First fast retry</span></span> |<span data-ttu-id="191e1-518">5.</span><span class="sxs-lookup"><span data-stu-id="191e1-518">5</span></span><br /><span data-ttu-id="191e1-519">0 seconde</span><span class="sxs-lookup"><span data-stu-id="191e1-519">0 sec</span></span><br /><span data-ttu-id="191e1-520">60 secondes</span><span class="sxs-lookup"><span data-stu-id="191e1-520">60 sec</span></span><br /><span data-ttu-id="191e1-521">2 secondes</span><span class="sxs-lookup"><span data-stu-id="191e1-521">2 sec</span></span><br /><span data-ttu-id="191e1-522">false</span><span class="sxs-lookup"><span data-stu-id="191e1-522">false</span></span> |<span data-ttu-id="191e1-523">Tentative 1 - délai 0 s</span><span class="sxs-lookup"><span data-stu-id="191e1-523">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="191e1-524">Tentative 2 - délai ~2 s</span><span class="sxs-lookup"><span data-stu-id="191e1-524">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="191e1-525">Tentative 3 - délai ~6 s</span><span class="sxs-lookup"><span data-stu-id="191e1-525">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="191e1-526">Tentative 4 - délai ~14 s</span><span class="sxs-lookup"><span data-stu-id="191e1-526">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="191e1-527">Tentative 5 - délai ~30 s</span><span class="sxs-lookup"><span data-stu-id="191e1-527">Attempt 5 - delay ~30 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="191e1-528">Les cibles de latence de bout en bout supposent le délai d’attente par défaut pour les connexions au service.</span><span class="sxs-lookup"><span data-stu-id="191e1-528">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="191e1-529">Si vous spécifiez des délais de connexion, la latence de bout en bout sera prolongée de ce temps supplémentaire pour toute nouvelle tentative.</span><span class="sxs-lookup"><span data-stu-id="191e1-529">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="191e1-530">Exemples</span><span class="sxs-lookup"><span data-stu-id="191e1-530">Examples</span></span>
<span data-ttu-id="191e1-531">Cette section vous indique comment utiliser Polly pour accéder à Azure SQL Database à l’aide d’un ensemble de stratégies de nouvelle tentative configurées dans la classe `Policy`.</span><span class="sxs-lookup"><span data-stu-id="191e1-531">This section shows how you can use Polly to access Azure SQL Database using a set of retry policies configured in the `Policy` class.</span></span>

<span data-ttu-id="191e1-532">Le code ci-après présente l’utilisation d’une méthode d’extension sur la classe `SqlCommand` qui appelle `ExecuteAsync` avec une interruption exponentielle.</span><span class="sxs-lookup"><span data-stu-id="191e1-532">The following code shows an extension method on the `SqlCommand` class that calls `ExecuteAsync` with exponential backoff.</span></span>

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

<span data-ttu-id="191e1-533">Cette méthode d’extension asynchrone est utilisable comme suit.</span><span class="sxs-lookup"><span data-stu-id="191e1-533">This asynchronous extension method can be used as follows.</span></span>

```csharp
var sqlCommand = sqlConnection.CreateCommand();
sqlCommand.CommandText = "[some query]";

using (var reader = await sqlCommand.ExecuteReaderWithRetryAsync())
{
    // Do something with the values
}
```

### <a name="more-information"></a><span data-ttu-id="191e1-534">Plus d’informations</span><span class="sxs-lookup"><span data-stu-id="191e1-534">More information</span></span>
* [<span data-ttu-id="191e1-535">Couche d’accès aux données de Cloud Service Fundamentals - Gestion des erreurs temporaires</span><span class="sxs-lookup"><span data-stu-id="191e1-535">Cloud Service Fundamentals Data Access Layer – Transient Fault Handling</span></span>](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)

<span data-ttu-id="191e1-536">Pour obtenir des conseils généraux sur le moyen de tirer le meilleur parti de SQL Database, consultez l’article [Azure SQL Database Performance and Elasticity Guide (Guide relatif à l’élasticité et aux performances d’Azure SQL Database)](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).</span><span class="sxs-lookup"><span data-stu-id="191e1-536">For general guidance on getting the most from SQL Database, see [Azure SQL Database Performance and Elasticity Guide](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).</span></span>

## <a name="sql-database-using-entity-framework-6"></a><span data-ttu-id="191e1-537">Base de données SQL utilisant Entity Framework 6</span><span class="sxs-lookup"><span data-stu-id="191e1-537">SQL Database using Entity Framework 6</span></span>
<span data-ttu-id="191e1-538">La base de données SQL est une base de données SQL hébergée disponible en différentes tailles et sous forme de service standard (partagé) et premium (non partagé).</span><span class="sxs-lookup"><span data-stu-id="191e1-538">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span> <span data-ttu-id="191e1-539">Entity Framework est un mappeur relationnel objet qui permet aux développeurs .NET de travailler avec des données relationnelles à l’aide d’objets spécifiques de domaine.</span><span class="sxs-lookup"><span data-stu-id="191e1-539">Entity Framework is an object-relational mapper that enables .NET developers to work with relational data using domain-specific objects.</span></span> <span data-ttu-id="191e1-540">Il élimine le recours à la plupart du code d’accès aux données que les développeurs doivent généralement écrire.</span><span class="sxs-lookup"><span data-stu-id="191e1-540">It eliminates the need for most of the data-access code that developers usually need to write.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="191e1-541">Mécanisme de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="191e1-541">Retry mechanism</span></span>
<span data-ttu-id="191e1-542">La prise en charge de la fonctionnalité de nouvelle tentative est assurée lors de l’accès à la base de données SQL à l’aide d’Entity Framework 6.0 et version supérieure via un mécanisme appelé [Résilience des connexions/logique de nouvelle tentative](http://msdn.microsoft.com/data/dn456835.aspx).</span><span class="sxs-lookup"><span data-stu-id="191e1-542">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher through a mechanism called [Connection Resiliency / Retry Logic](http://msdn.microsoft.com/data/dn456835.aspx).</span></span> <span data-ttu-id="191e1-543">Les principales fonctionnalités de ce mécanisme sont :</span><span class="sxs-lookup"><span data-stu-id="191e1-543">The main features of the retry mechanism are:</span></span>

* <span data-ttu-id="191e1-544">L’abstraction principale est l’interface **IDbExecutionStrategy** .</span><span class="sxs-lookup"><span data-stu-id="191e1-544">The primary abstraction is the **IDbExecutionStrategy** interface.</span></span> <span data-ttu-id="191e1-545">Cette interface :</span><span class="sxs-lookup"><span data-stu-id="191e1-545">This interface:</span></span>
  * <span data-ttu-id="191e1-546">Définit les méthodes synchrones et asynchrones **Execute** \*.</span><span class="sxs-lookup"><span data-stu-id="191e1-546">Defines synchronous and asynchronous **Execute**\* methods.</span></span>
  * <span data-ttu-id="191e1-547">Définit les classes qui peuvent être utilisées directement ou configurées sur un contexte de base de données comme une stratégie par défaut mappée sur un nom de fournisseur ou un nom de fournisseur et un nom de serveur.</span><span class="sxs-lookup"><span data-stu-id="191e1-547">Defines classes that can be used directly or can be configured on a database context as a default strategy, mapped to provider name, or mapped to a provider name and server name.</span></span> <span data-ttu-id="191e1-548">En cas de configuration sur un contexte, les nouvelles tentatives se produisent au niveau des opérations de base de données individuelles. Il peut y en avoir plusieurs pour une opération de contexte donnée.</span><span class="sxs-lookup"><span data-stu-id="191e1-548">When configured on a context, retries occur at the level of individual database operations, of which there might be several for a given context operation.</span></span>
  * <span data-ttu-id="191e1-549">Définit à quel moment et comment proposer une nouvelle tentative en cas d’échec de connexion.</span><span class="sxs-lookup"><span data-stu-id="191e1-549">Defines when to retry a failed connection, and how.</span></span>
* <span data-ttu-id="191e1-550">Il inclut plusieurs implémentations intégrées de l’interface **IDbExecutionStrategy** :</span><span class="sxs-lookup"><span data-stu-id="191e1-550">It includes several built-in implementations of the **IDbExecutionStrategy** interface:</span></span>
  * <span data-ttu-id="191e1-551">Par défaut : aucune nouvelle tentative.</span><span class="sxs-lookup"><span data-stu-id="191e1-551">Default - no retrying.</span></span>
  * <span data-ttu-id="191e1-552">Par défaut pour la base de données SQL (automatique) : aucune nouvelle tentative, mais inspecte les exceptions et y inclut la suggestion d’utiliser la stratégie de base de données SQL.</span><span class="sxs-lookup"><span data-stu-id="191e1-552">Default for SQL Database (automatic) - no retrying, but inspects exceptions and wraps them with suggestion to use the SQL Database strategy.</span></span>
  * <span data-ttu-id="191e1-553">Par défaut pour la base de données SQL : exponentielle (héritée de la classe de base) et logique de détection de base de données SQL.</span><span class="sxs-lookup"><span data-stu-id="191e1-553">Default for SQL Database - exponential (inherited from base class) plus SQL Database detection logic.</span></span>
* <span data-ttu-id="191e1-554">Il implémente une stratégie de temporisation exponentielle qui inclut la répartition aléatoire.</span><span class="sxs-lookup"><span data-stu-id="191e1-554">It implements an exponential back-off strategy that includes randomization.</span></span>
* <span data-ttu-id="191e1-555">Les classes de nouvelle tentative intégrées sont avec état et ne sont pas thread-safe.</span><span class="sxs-lookup"><span data-stu-id="191e1-555">The built-in retry classes are stateful and are not thread safe.</span></span> <span data-ttu-id="191e1-556">Toutefois, elles peuvent être réutilisées une fois l’opération en cours terminée.</span><span class="sxs-lookup"><span data-stu-id="191e1-556">However, they can be reused after the current operation is completed.</span></span>
* <span data-ttu-id="191e1-557">Si le nombre de tentatives spécifié est dépassé, les résultats sont inclus dans une nouvelle exception.</span><span class="sxs-lookup"><span data-stu-id="191e1-557">If the specified retry count is exceeded, the results are wrapped in a new exception.</span></span> <span data-ttu-id="191e1-558">Il ne se propage pas dans l’exception en cours.</span><span class="sxs-lookup"><span data-stu-id="191e1-558">It does not bubble up the current exception.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="191e1-559">Configuration de la stratégie</span><span class="sxs-lookup"><span data-stu-id="191e1-559">Policy configuration</span></span>
<span data-ttu-id="191e1-560">La prise en charge de la nouvelle tentative est assurée lors de l’accès à la base de données SQL à l’aide d’Entity Framework 6.0 et versions ultérieures.</span><span class="sxs-lookup"><span data-stu-id="191e1-560">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher.</span></span> <span data-ttu-id="191e1-561">Les stratégies de nouvelle tentative sont configurées par programme.</span><span class="sxs-lookup"><span data-stu-id="191e1-561">Retry policies are configured programmatically.</span></span> <span data-ttu-id="191e1-562">La configuration ne peut pas être modifiée opération par opération.</span><span class="sxs-lookup"><span data-stu-id="191e1-562">The configuration cannot be changed on a per-operation basis.</span></span>

<span data-ttu-id="191e1-563">Lorsque vous configurez une stratégie sur le contexte comme stratégie par défaut, vous définissez une fonction qui crée une nouvelle stratégie à la demande.</span><span class="sxs-lookup"><span data-stu-id="191e1-563">When configuring a strategy on the context as the default, you specify a function that creates a new strategy on demand.</span></span> <span data-ttu-id="191e1-564">Le code suivant montre comment vous pouvez créer une classe de configuration de nouvelle tentative qui étend la classe de base **DbConfiguration** .</span><span class="sxs-lookup"><span data-stu-id="191e1-564">The following code shows how you can create a retry configuration class that extends the **DbConfiguration** base class.</span></span>

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

<span data-ttu-id="191e1-565">Vous pouvez alors la spécifier comme stratégie de nouvelle tentative par défaut pour toutes les opérations à l’aide de la méthode **SetConfiguration** de l’instance **DbConfiguration** lorsque l’application démarre.</span><span class="sxs-lookup"><span data-stu-id="191e1-565">You can then specify this as the default retry strategy for all operations using the **SetConfiguration** method of the **DbConfiguration** instance when the application starts.</span></span> <span data-ttu-id="191e1-566">Par défaut, Entity Framework détecte et utilise automatiquement la classe de configuration.</span><span class="sxs-lookup"><span data-stu-id="191e1-566">By default, EF will automatically discover and use the configuration class.</span></span>

```csharp
DbConfiguration.SetConfiguration(new BloggingContextConfiguration());
```

<span data-ttu-id="191e1-567">Vous pouvez spécifier la classe de configuration de nouvelle tentative pour un contexte en annotant la classe de contexte avec un attribut **DbConfigurationType** .</span><span class="sxs-lookup"><span data-stu-id="191e1-567">You can specify the retry configuration class for a context by annotating the context class with a **DbConfigurationType** attribute.</span></span> <span data-ttu-id="191e1-568">Toutefois, si vous avez une seule classe de configuration, Entity Framework l’utilisera sans avoir à annoter le contexte.</span><span class="sxs-lookup"><span data-stu-id="191e1-568">However, if you have only one configuration class, EF will use it without the need to annotate the context.</span></span>

```csharp
[DbConfigurationType(typeof(BloggingContextConfiguration))]
public class BloggingContext : DbContext
```

<span data-ttu-id="191e1-569">Si vous devez utiliser des stratégies de nouvelle tentative différentes pour des opérations spécifiques ou désactiver des nouvelles tentatives pour des opérations spécifiques, vous pouvez créer une classe de configuration qui vous permet de suspendre ou d’échanger des stratégies en définissant un indicateur dans le **CallContext**.</span><span class="sxs-lookup"><span data-stu-id="191e1-569">If you need to use different retry strategies for specific operations, or disable retries for specific operations, you can create a configuration class that allows you to suspend or swap strategies by setting a flag in the **CallContext**.</span></span> <span data-ttu-id="191e1-570">La classe de configuration peut utiliser cet indicateur pour changer de stratégie ou désactiver la stratégie indiquée et utilisée comme stratégie par défaut.</span><span class="sxs-lookup"><span data-stu-id="191e1-570">The configuration class can use this flag to switch strategies, or disable the strategy you provide and use a default strategy.</span></span> <span data-ttu-id="191e1-571">Pour plus d’informations, consultez [Suspendre la stratégie d’exécution](http://msdn.microsoft.com/dn307226#transactions_workarounds) dans la page de Restrictions liées aux stratégies d’exécution de nouvelle tentative (Entity Framework 6 et versions supérieures).</span><span class="sxs-lookup"><span data-stu-id="191e1-571">For more information, see [Suspend Execution Strategy](http://msdn.microsoft.com/dn307226#transactions_workarounds) in the page Limitations with Retrying Execution Strategies (EF6 onwards).</span></span>

<span data-ttu-id="191e1-572">Une autre technique d’utilisation de stratégies de nouvelle tentative spécifiques pour des opérations individuelles consiste à créer une instance de la classe de stratégie requise et à fournir la configuration requise au moyen de paramètres.</span><span class="sxs-lookup"><span data-stu-id="191e1-572">Another technique for using specific retry strategies for individual operations is to create an instance of the required strategy class and supply the desired settings through parameters.</span></span> <span data-ttu-id="191e1-573">Vous pouvez ensuite faire appel à sa méthode **ExecuteAsync** .</span><span class="sxs-lookup"><span data-stu-id="191e1-573">You then invoke its **ExecuteAsync** method.</span></span>

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

<span data-ttu-id="191e1-574">La façon la plus simple d’utiliser une classe **DbConfiguration** consiste à la localiser dans le même assembly que la classe **DbContext**.</span><span class="sxs-lookup"><span data-stu-id="191e1-574">The simplest way to use a **DbConfiguration** class is to locate it in the same assembly as the **DbContext** class.</span></span> <span data-ttu-id="191e1-575">Toutefois, cela n’est pas approprié lorsque le même contexte est requis dans différents scénarios, comme dans le cas de stratégies de nouvelle tentative en arrière-plan et interactives.</span><span class="sxs-lookup"><span data-stu-id="191e1-575">However, this is not appropriate when the same context is required in different scenarios, such as different interactive and background retry strategies.</span></span> <span data-ttu-id="191e1-576">Si les différents contextes s’exécutent dans un AppDomain séparé, vous pouvez utiliser la prise en charge intégrée pour spécifier des classes de configuration dans le fichier de configuration ou la définir explicitement à l’aide de code.</span><span class="sxs-lookup"><span data-stu-id="191e1-576">If the different contexts execute in separate AppDomains, you can use the built-in support for specifying configuration classes in the configuration file or set it explicitly using code.</span></span> <span data-ttu-id="191e1-577">Si les différents contextes doivent s’exécuter dans le même AppDomain, une solution personnalisée sera nécessaire.</span><span class="sxs-lookup"><span data-stu-id="191e1-577">If the different contexts must execute in the same AppDomain, a custom solution will be required.</span></span>

<span data-ttu-id="191e1-578">Pour plus d’informations, consultez [Configuration basée sur le code (Entity Framework 6 et versions supérieures)](http://msdn.microsoft.com/data/jj680699.aspx).</span><span class="sxs-lookup"><span data-stu-id="191e1-578">For more information, see [Code-Based Configuration (EF6 onwards)](http://msdn.microsoft.com/data/jj680699.aspx).</span></span>

<span data-ttu-id="191e1-579">Le tableau suivant présente les paramètres par défaut pour la stratégie de nouvelle tentative intégrée lors de l’utilisation d’Entity Framework 6.</span><span class="sxs-lookup"><span data-stu-id="191e1-579">The following table shows the default settings for the built-in retry policy when using EF6.</span></span>

| <span data-ttu-id="191e1-580">Paramètre</span><span class="sxs-lookup"><span data-stu-id="191e1-580">Setting</span></span> | <span data-ttu-id="191e1-581">Valeur par défaut</span><span class="sxs-lookup"><span data-stu-id="191e1-581">Default value</span></span> | <span data-ttu-id="191e1-582">Signification</span><span class="sxs-lookup"><span data-stu-id="191e1-582">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="191e1-583">Stratégie</span><span class="sxs-lookup"><span data-stu-id="191e1-583">Policy</span></span> | <span data-ttu-id="191e1-584">Exponentielle</span><span class="sxs-lookup"><span data-stu-id="191e1-584">Exponential</span></span> | <span data-ttu-id="191e1-585">Temporisation exponentielle.</span><span class="sxs-lookup"><span data-stu-id="191e1-585">Exponential back-off.</span></span> |
| <span data-ttu-id="191e1-586">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="191e1-586">MaxRetryCount</span></span> | <span data-ttu-id="191e1-587">5.</span><span class="sxs-lookup"><span data-stu-id="191e1-587">5</span></span> | <span data-ttu-id="191e1-588">Nombre maximal de nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="191e1-588">The maximum number of retries.</span></span> |
| <span data-ttu-id="191e1-589">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="191e1-589">MaxDelay</span></span> | <span data-ttu-id="191e1-590">30 secondes</span><span class="sxs-lookup"><span data-stu-id="191e1-590">30 seconds</span></span> | <span data-ttu-id="191e1-591">Délai maximal entre les nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="191e1-591">The maximum delay between retries.</span></span> <span data-ttu-id="191e1-592">Cette valeur n’affecte pas le mode de calcul de la série de délais.</span><span class="sxs-lookup"><span data-stu-id="191e1-592">This value does not affect how the series of delays are computed.</span></span> <span data-ttu-id="191e1-593">Elle définit uniquement une limite supérieure.</span><span class="sxs-lookup"><span data-stu-id="191e1-593">It only defines an upper bound.</span></span> |
| <span data-ttu-id="191e1-594">DefaultCoefficient</span><span class="sxs-lookup"><span data-stu-id="191e1-594">DefaultCoefficient</span></span> | <span data-ttu-id="191e1-595">1 seconde</span><span class="sxs-lookup"><span data-stu-id="191e1-595">1 second</span></span> | <span data-ttu-id="191e1-596">Coefficient du calcul de temporisation exponentielle.</span><span class="sxs-lookup"><span data-stu-id="191e1-596">The coefficient for the exponential back-off computation.</span></span> <span data-ttu-id="191e1-597">Cette valeur ne peut pas être modifiée.</span><span class="sxs-lookup"><span data-stu-id="191e1-597">This value cannot be changed.</span></span> |
| <span data-ttu-id="191e1-598">DefaultRandomFactor</span><span class="sxs-lookup"><span data-stu-id="191e1-598">DefaultRandomFactor</span></span> | <span data-ttu-id="191e1-599">1.1</span><span class="sxs-lookup"><span data-stu-id="191e1-599">1.1</span></span> | <span data-ttu-id="191e1-600">Multiplicateur utilisé pour l’ajout d’un délai aléatoire pour chaque entrée.</span><span class="sxs-lookup"><span data-stu-id="191e1-600">The multiplier used to add a random delay for each entry.</span></span> <span data-ttu-id="191e1-601">Cette valeur ne peut pas être modifiée.</span><span class="sxs-lookup"><span data-stu-id="191e1-601">This value cannot be changed.</span></span> |
| <span data-ttu-id="191e1-602">DefaultExponentialBase</span><span class="sxs-lookup"><span data-stu-id="191e1-602">DefaultExponentialBase</span></span> | <span data-ttu-id="191e1-603">2</span><span class="sxs-lookup"><span data-stu-id="191e1-603">2</span></span> | <span data-ttu-id="191e1-604">Multiplicateur utilisé pour le calcul du délai suivant.</span><span class="sxs-lookup"><span data-stu-id="191e1-604">The multiplier used to calculate the next delay.</span></span> <span data-ttu-id="191e1-605">Cette valeur ne peut pas être modifiée.</span><span class="sxs-lookup"><span data-stu-id="191e1-605">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="191e1-606">Guide d’utilisation des nouvelles tentatives</span><span class="sxs-lookup"><span data-stu-id="191e1-606">Retry usage guidance</span></span>
<span data-ttu-id="191e1-607">Respectez les consignes suivantes lorsque vous accédez à la base de données SQL à l’aide d’Entity Framework 6 :</span><span class="sxs-lookup"><span data-stu-id="191e1-607">Consider the following guidelines when accessing SQL Database using EF6:</span></span>

* <span data-ttu-id="191e1-608">Choisissez l’option de service appropriée (partagée ou premium).</span><span class="sxs-lookup"><span data-stu-id="191e1-608">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="191e1-609">Une instance partagée peut être soumise à des délais de connexion plus longs que d’ordinaire et à des limitations en raison de l’utilisation du serveur partagé par d’autres locataires.</span><span class="sxs-lookup"><span data-stu-id="191e1-609">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="191e1-610">Si des performances prévisibles et des opérations fiables à faible latence sont requises, pensez à choisir l’option premium.</span><span class="sxs-lookup"><span data-stu-id="191e1-610">If predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="191e1-611">Une stratégie d’intervalle fixe n’est pas recommandée pour une utilisation avec la base de données SQL Azure.</span><span class="sxs-lookup"><span data-stu-id="191e1-611">A fixed interval strategy is not recommended for use with Azure SQL Database.</span></span> <span data-ttu-id="191e1-612">Utilisez plutôt une stratégie de temporisation exponentielle car le service peut être surchargé et des délais plus longs laissent davantage de temps de récupération.</span><span class="sxs-lookup"><span data-stu-id="191e1-612">Instead, use an exponential back-off strategy because the service may be overloaded, and longer delays allow more time for it to recover.</span></span>
* <span data-ttu-id="191e1-613">Choisissez une valeur appropriée pour les délais d’attente de connexion et de commande lors de la définition des connexions.</span><span class="sxs-lookup"><span data-stu-id="191e1-613">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="191e1-614">Basez le délai d’attente à la fois sur la conception de la logique métier et sur les tests.</span><span class="sxs-lookup"><span data-stu-id="191e1-614">Base the timeout on both your business logic design and through testing.</span></span> <span data-ttu-id="191e1-615">Vous devrez peut-être modifier cette valeur au fil du temps en fonction de l’évolution des volumes de données ou des processus d’entreprise.</span><span class="sxs-lookup"><span data-stu-id="191e1-615">You may need to modify this value over time as the volumes of data or the business processes change.</span></span> <span data-ttu-id="191e1-616">Un délai d’attente trop court peut entraîner des échecs de connexion prématurés lorsque la base de données est occupée.</span><span class="sxs-lookup"><span data-stu-id="191e1-616">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="191e1-617">Un délai d’attente trop long peut empêcher la logique de nouvelle tentative de fonctionner correctement en attendant trop longtemps avant la détection d’un échec de connexion.</span><span class="sxs-lookup"><span data-stu-id="191e1-617">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="191e1-618">La valeur d’expiration est un composant de la latence de bout en bout, bien que vous ne puissiez pas aisément déterminer le nombre de commandes qui seront exécutées lors de l’enregistrement du contexte.</span><span class="sxs-lookup"><span data-stu-id="191e1-618">The value of the timeout is a component of the end-to-end latency, although you cannot easily determine how many commands will execute when saving the context.</span></span> <span data-ttu-id="191e1-619">Vous pouvez modifier le délai d’attente par défaut en définissant la propriété **CommandTimeout** de l’instance **DbContext**.</span><span class="sxs-lookup"><span data-stu-id="191e1-619">You can change the default timeout by setting the **CommandTimeout** property of the **DbContext** instance.</span></span>
* <span data-ttu-id="191e1-620">Entity Framework prend en charge les configurations de nouvelle tentative définies dans les fichiers de configuration.</span><span class="sxs-lookup"><span data-stu-id="191e1-620">Entity Framework supports retry configurations defined in configuration files.</span></span> <span data-ttu-id="191e1-621">Toutefois, pour une flexibilité maximale sur Azure, vous devez envisager la création de la configuration par programme au sein de l’application.</span><span class="sxs-lookup"><span data-stu-id="191e1-621">However, for maximum flexibility on Azure you should consider creating the configuration programmatically within the application.</span></span> <span data-ttu-id="191e1-622">Les paramètres spécifiques pour les stratégies de nouvelle tentative, tels que le nombre de nouvelles tentatives et les intervalles, peuvent être stockés dans le fichier de configuration du service et être utilisés en cours d’exécution pour créer les stratégies appropriées.</span><span class="sxs-lookup"><span data-stu-id="191e1-622">The specific parameters for the retry policies, such as the number of retries and the retry intervals, can be stored in the service configuration file and used at runtime to create the appropriate policies.</span></span> <span data-ttu-id="191e1-623">Cela permet de modifier les paramètres sans avoir à redémarrer l’application.</span><span class="sxs-lookup"><span data-stu-id="191e1-623">This allows the settings to be changed without requiring the application to be restarted.</span></span>

<span data-ttu-id="191e1-624">Pensez à commencer par les paramètres ci-après pour les opérations liées aux nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="191e1-624">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="191e1-625">Vous ne pouvez pas spécifier le délai entre chaque nouvelle tentative (il est fixe et généré sous la forme d’une séquence exponentielle).</span><span class="sxs-lookup"><span data-stu-id="191e1-625">You cannot specify the delay between retry attempts (it is fixed and generated as an exponential sequence).</span></span> <span data-ttu-id="191e1-626">Vous ne pouvez spécifier que les valeurs maximales, comme indiqué ici, sauf si vous créez une stratégie de nouvelle tentative personnalisée.</span><span class="sxs-lookup"><span data-stu-id="191e1-626">You can specify only the maximum values, as shown here; unless you create a custom retry strategy.</span></span> <span data-ttu-id="191e1-627">Il s’agit de paramètres généraux. Vous devez par conséquent surveiller les opérations et optimiser les valeurs en fonction de votre propre scénario.</span><span class="sxs-lookup"><span data-stu-id="191e1-627">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="191e1-628">**Contexte**</span><span class="sxs-lookup"><span data-stu-id="191e1-628">**Context**</span></span> | <span data-ttu-id="191e1-629">**Exemple de cible E2E<br />latence max**</span><span class="sxs-lookup"><span data-stu-id="191e1-629">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="191e1-630">**Stratégie de nouvelle tentative**</span><span class="sxs-lookup"><span data-stu-id="191e1-630">**Retry policy**</span></span> | <span data-ttu-id="191e1-631">**Paramètres**</span><span class="sxs-lookup"><span data-stu-id="191e1-631">**Settings**</span></span> | <span data-ttu-id="191e1-632">**Valeurs**</span><span class="sxs-lookup"><span data-stu-id="191e1-632">**Values**</span></span> | <span data-ttu-id="191e1-633">**Fonctionnement**</span><span class="sxs-lookup"><span data-stu-id="191e1-633">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="191e1-634">Interactif, interface utilisateur,</span><span class="sxs-lookup"><span data-stu-id="191e1-634">Interactive, UI,</span></span><br /><span data-ttu-id="191e1-635">ou premier plan</span><span class="sxs-lookup"><span data-stu-id="191e1-635">or foreground</span></span> |<span data-ttu-id="191e1-636">2 secondes</span><span class="sxs-lookup"><span data-stu-id="191e1-636">2 seconds</span></span> |<span data-ttu-id="191e1-637">Exponentielle</span><span class="sxs-lookup"><span data-stu-id="191e1-637">Exponential</span></span> |<span data-ttu-id="191e1-638">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="191e1-638">MaxRetryCount</span></span><br /><span data-ttu-id="191e1-639">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="191e1-639">MaxDelay</span></span> |<span data-ttu-id="191e1-640">3</span><span class="sxs-lookup"><span data-stu-id="191e1-640">3</span></span><br /><span data-ttu-id="191e1-641">750 ms</span><span class="sxs-lookup"><span data-stu-id="191e1-641">750 ms</span></span> |<span data-ttu-id="191e1-642">Tentative 1 - délai 0 s</span><span class="sxs-lookup"><span data-stu-id="191e1-642">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="191e1-643">Tentative 2 - délai 750 ms</span><span class="sxs-lookup"><span data-stu-id="191e1-643">Attempt 2 - delay 750 ms</span></span><br /><span data-ttu-id="191e1-644">Tentative 3 - délai 750 ms</span><span class="sxs-lookup"><span data-stu-id="191e1-644">Attempt 3 – delay 750 ms</span></span> |
| <span data-ttu-id="191e1-645">Arrière-plan</span><span class="sxs-lookup"><span data-stu-id="191e1-645">Background</span></span><br /> <span data-ttu-id="191e1-646">ou lot</span><span class="sxs-lookup"><span data-stu-id="191e1-646">or batch</span></span> |<span data-ttu-id="191e1-647">30 secondes</span><span class="sxs-lookup"><span data-stu-id="191e1-647">30 seconds</span></span> |<span data-ttu-id="191e1-648">Exponentielle</span><span class="sxs-lookup"><span data-stu-id="191e1-648">Exponential</span></span> |<span data-ttu-id="191e1-649">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="191e1-649">MaxRetryCount</span></span><br /><span data-ttu-id="191e1-650">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="191e1-650">MaxDelay</span></span> |<span data-ttu-id="191e1-651">5.</span><span class="sxs-lookup"><span data-stu-id="191e1-651">5</span></span><br /><span data-ttu-id="191e1-652">12 secondes</span><span class="sxs-lookup"><span data-stu-id="191e1-652">12 seconds</span></span> |<span data-ttu-id="191e1-653">Tentative 1 - délai 0 s</span><span class="sxs-lookup"><span data-stu-id="191e1-653">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="191e1-654">Tentative 2 - délai ~1 s</span><span class="sxs-lookup"><span data-stu-id="191e1-654">Attempt 2 - delay ~1 sec</span></span><br /><span data-ttu-id="191e1-655">Tentative 3 - délai ~3 s</span><span class="sxs-lookup"><span data-stu-id="191e1-655">Attempt 3 - delay ~3 sec</span></span><br /><span data-ttu-id="191e1-656">Tentative 4 - délai ~7 s</span><span class="sxs-lookup"><span data-stu-id="191e1-656">Attempt 4 - delay ~7 sec</span></span><br /><span data-ttu-id="191e1-657">Tentative 5 - délai 12 s</span><span class="sxs-lookup"><span data-stu-id="191e1-657">Attempt 5 - delay 12 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="191e1-658">Les cibles de latence de bout en bout supposent le délai d’attente par défaut pour les connexions au service.</span><span class="sxs-lookup"><span data-stu-id="191e1-658">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="191e1-659">Si vous spécifiez des délais de connexion, la latence de bout en bout sera prolongée de ce temps supplémentaire pour toute nouvelle tentative.</span><span class="sxs-lookup"><span data-stu-id="191e1-659">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="191e1-660">Exemples</span><span class="sxs-lookup"><span data-stu-id="191e1-660">Examples</span></span>
<span data-ttu-id="191e1-661">L’exemple de code suivant définit une solution d’accès aux données simple qui utilise Entity Framework.</span><span class="sxs-lookup"><span data-stu-id="191e1-661">The following code example defines a simple data access solution that uses Entity Framework.</span></span> <span data-ttu-id="191e1-662">Il spécifie une stratégie de nouvelle tentative spécifique par la définition de l’instance d’une classe nommée **BlogConfiguration** qui étend **DbConfiguration**.</span><span class="sxs-lookup"><span data-stu-id="191e1-662">It sets a specific retry strategy by defining an instance of a class named **BlogConfiguration** that extends **DbConfiguration**.</span></span>

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

<span data-ttu-id="191e1-663">Vous trouverez plusieurs exemples d’utilisation du mécanisme de nouvelle tentative d’Entity Framework dans [Résilience des connexions/logique de nouvelle tentative](http://msdn.microsoft.com/data/dn456835.aspx).</span><span class="sxs-lookup"><span data-stu-id="191e1-663">More examples of using the Entity Framework retry mechanism can be found in [Connection Resiliency / Retry Logic](http://msdn.microsoft.com/data/dn456835.aspx).</span></span>

### <a name="more-information"></a><span data-ttu-id="191e1-664">Plus d’informations</span><span class="sxs-lookup"><span data-stu-id="191e1-664">More information</span></span>
* [<span data-ttu-id="191e1-665">Guide relatif à l’élasticité et aux performances de la base de données SQL Azure</span><span class="sxs-lookup"><span data-stu-id="191e1-665">Azure SQL Database Performance and Elasticity Guide</span></span>](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)

## <a name="sql-database-using-entity-framework-core"></a><span data-ttu-id="191e1-666">Base de données SQL utilisant Entity Framework Core</span><span class="sxs-lookup"><span data-stu-id="191e1-666">SQL Database using Entity Framework Core</span></span>
<span data-ttu-id="191e1-667">[Entity Framework Core](/ef/core/) est un mappeur Objet Relationnel qui permet aux développeurs .NET Core de travailler avec des données à l’aide d’objets spécifiques du domaine.</span><span class="sxs-lookup"><span data-stu-id="191e1-667">[Entity Framework Core](/ef/core/) is an object-relational mapper that enables .NET Core developers to work with data using domain-specific objects.</span></span> <span data-ttu-id="191e1-668">Il élimine le recours à la plupart du code d’accès aux données que les développeurs doivent généralement écrire.</span><span class="sxs-lookup"><span data-stu-id="191e1-668">It eliminates the need for most of the data-access code that developers usually need to write.</span></span> <span data-ttu-id="191e1-669">Cette version d’Entity Framework a été écrite à partir de zéro et n’hérite pas automatiquement de toutes les fonctionnalités d’EF6.x.</span><span class="sxs-lookup"><span data-stu-id="191e1-669">This version of Entity Framework was written from the ground up, and doesn't automatically inherit all the features from EF6.x.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="191e1-670">Mécanisme de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="191e1-670">Retry mechanism</span></span>
<span data-ttu-id="191e1-671">La prise en charge du mécanisme de nouvelle tentative est assurée lors de l’accès à SQL Database utilisant Entity Framework Core par le biais d’un mécanisme appelé [Résilience de connexion](/ef/core/miscellaneous/connection-resiliency).</span><span class="sxs-lookup"><span data-stu-id="191e1-671">Retry support is provided when accessing SQL Database using Entity Framework Core through a mechanism called [Connection Resiliency](/ef/core/miscellaneous/connection-resiliency).</span></span> <span data-ttu-id="191e1-672">La résilience de connexion a été introduite dans EF Core 1.1.0.</span><span class="sxs-lookup"><span data-stu-id="191e1-672">Connection resiliency was introduced in EF Core 1.1.0.</span></span>

<span data-ttu-id="191e1-673">L’abstraction principale est l’interface `IExecutionStrategy`.</span><span class="sxs-lookup"><span data-stu-id="191e1-673">The primary abstraction is the `IExecutionStrategy` interface.</span></span> <span data-ttu-id="191e1-674">La stratégie d’exécution de SQL Server, incluant SQL Azure, tient compte des types d’exceptions qui peuvent faire l’objet de nouvelles tentatives et présente des valeurs par défaut sensibles pour le nombre maximal de nouvelles tentatives, le délai entre deux tentatives, et ainsi de suite.</span><span class="sxs-lookup"><span data-stu-id="191e1-674">The execution strategy for SQL Server, including SQL Azure, is aware of the exception types that can be retried and has sensible defaults for maximum retries, delay between retries, and so on.</span></span>

### <a name="examples"></a><span data-ttu-id="191e1-675">Exemples</span><span class="sxs-lookup"><span data-stu-id="191e1-675">Examples</span></span>

<span data-ttu-id="191e1-676">Le code ci-après autorise les nouvelles tentatives automatiques lors de la configuration de l’objet DbContext, qui représente une session avec la base de données.</span><span class="sxs-lookup"><span data-stu-id="191e1-676">The following code enables automatic retries when configuring the DbContext object, which represents a session with the database.</span></span> 

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(
            @"Server=(localdb)\mssqllocaldb;Database=EFMiscellanous.ConnectionResiliency;Trusted_Connection=True;",
            options => options.EnableRetryOnFailure());
}
```

<span data-ttu-id="191e1-677">Le code ci-après indique comment exécuter une transaction avec de nouvelles tentatives automatiques à l’aide d’une stratégie d’exécution.</span><span class="sxs-lookup"><span data-stu-id="191e1-677">The following code shows how to execute a transaction with automatic retries, by using an execution strategy.</span></span> <span data-ttu-id="191e1-678">La transaction est définie dans un délégué.</span><span class="sxs-lookup"><span data-stu-id="191e1-678">The transaction is defined in a delegate.</span></span> <span data-ttu-id="191e1-679">En cas d’échec passager, la stratégie d’exécution appelle de nouveau le délégué.</span><span class="sxs-lookup"><span data-stu-id="191e1-679">If a transient failure occurs, the execution strategy will invoke the delegate again.</span></span>

```csharp
using (var db = new BloggingContext())
{
    var strategy = db.Database.CreateExecutionStrategy();

    strategy.Execute(() =>
    {
        using (var transaction = db.Database.BeginTransaction())
        {
            db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/dotnet" });
            db.SaveChanges();

            db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/visualstudio" });
            db.SaveChanges();

            transaction.Commit();
        }
    });
}
```

## <a name="azure-storage"></a><span data-ttu-id="191e1-680">Stockage Azure</span><span class="sxs-lookup"><span data-stu-id="191e1-680">Azure Storage</span></span>
<span data-ttu-id="191e1-681">Les services de stockage Azure incluent le stockage des tables et des objets blob, les fichiers et les files d’attente de stockage.</span><span class="sxs-lookup"><span data-stu-id="191e1-681">Azure storage services include table and blob storage, files, and storage queues.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="191e1-682">Mécanisme de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="191e1-682">Retry mechanism</span></span>
<span data-ttu-id="191e1-683">Les nouvelles tentatives se produisent au niveau de chaque opération REST et font partie intégrante de l’implémentation de l’API client.</span><span class="sxs-lookup"><span data-stu-id="191e1-683">Retries occur at the individual REST operation level and are an integral part of the client API implementation.</span></span> <span data-ttu-id="191e1-684">Le kit de développement logiciel (SDK) de stockage client utilise des classes qui implémentent l’ [interface IExtendedRetryPolicy](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx).</span><span class="sxs-lookup"><span data-stu-id="191e1-684">The client storage SDK uses classes that implement the [IExtendedRetryPolicy Interface](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx).</span></span>

<span data-ttu-id="191e1-685">Il existe différentes implémentations de l’interface.</span><span class="sxs-lookup"><span data-stu-id="191e1-685">There are different implementations of the interface.</span></span> <span data-ttu-id="191e1-686">Les clients de stockage peuvent choisir parmi les stratégies spécialement conçues pour l’accès à des tables, des objets BLOB et des files d’attente.</span><span class="sxs-lookup"><span data-stu-id="191e1-686">Storage clients can choose from policies specifically designed for accessing tables, blobs, and queues.</span></span> <span data-ttu-id="191e1-687">Chaque implémentation utilise une stratégie de nouvelle tentative différente qui définit essentiellement l’intervalle avant une nouvelle tentative et d’autres détails.</span><span class="sxs-lookup"><span data-stu-id="191e1-687">Each implementation uses a different retry strategy that essentially defines the retry interval and other details.</span></span>

<span data-ttu-id="191e1-688">Les classes intégrées prennent en charge la stratégie linéaire (délai constant) et exponentielle avec répartition aléatoire des intervalles entre chaque nouvelle tentative.</span><span class="sxs-lookup"><span data-stu-id="191e1-688">The built-in classes provide support for linear (constant delay) and exponential with randomization retry intervals.</span></span> <span data-ttu-id="191e1-689">Il n’existe également aucune stratégie de nouvelle tentative à utiliser lorsqu’un autre processus gère les nouvelles tentatives à un niveau supérieur.</span><span class="sxs-lookup"><span data-stu-id="191e1-689">There is also a no retry policy for use when another process is handling retries at a higher level.</span></span> <span data-ttu-id="191e1-690">Toutefois, vous pouvez implémenter vos propres classes de nouvelle tentative si vous avez des exigences spécifiques non fournies par les classes intégrées.</span><span class="sxs-lookup"><span data-stu-id="191e1-690">However, you can implement your own retry classes if you have specific requirements not provided by the built-in classes.</span></span>

<span data-ttu-id="191e1-691">Les autres alternatives passent d’un emplacement de service de stockage principal à un emplacement de service de stockage secondaire si vous utilisez le stockage géo-redondant avec accès en lecture (RA-GRS) et que le résultat de la demande est une erreur renouvelable.</span><span class="sxs-lookup"><span data-stu-id="191e1-691">Alternate retries switch between primary and secondary storage service location if you are using read access geo-redundant storage (RA-GRS) and the result of the request is a retryable error.</span></span> <span data-ttu-id="191e1-692">Pour plus d’informations, consultez [Options de redondance du stockage Azure](http://msdn.microsoft.com/library/azure/dn727290.aspx) .</span><span class="sxs-lookup"><span data-stu-id="191e1-692">See [Azure Storage Redundancy Options](http://msdn.microsoft.com/library/azure/dn727290.aspx) for more information.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="191e1-693">Configuration de la stratégie</span><span class="sxs-lookup"><span data-stu-id="191e1-693">Policy configuration</span></span>
<span data-ttu-id="191e1-694">Les stratégies de nouvelle tentative sont configurées par programme.</span><span class="sxs-lookup"><span data-stu-id="191e1-694">Retry policies are configured programmatically.</span></span> <span data-ttu-id="191e1-695">Une procédure classique consiste à créer et remplir une instance **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions** ou **QueueRequestOptions**.</span><span class="sxs-lookup"><span data-stu-id="191e1-695">A typical procedure is to create and populate a **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions**, or **QueueRequestOptions** instance.</span></span>

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

<span data-ttu-id="191e1-696">L’instance d’options de demande peut ensuite être définie sur le client et toutes les opérations avec le client utiliseront les options de demande spécifiées.</span><span class="sxs-lookup"><span data-stu-id="191e1-696">The request options instance can then be set on the client, and all operations with the client will use the specified request options.</span></span>

```csharp
client.DefaultRequestOptions = interactiveRequestOption;
var stats = await client.GetServiceStatsAsync();
```

<span data-ttu-id="191e1-697">Vous pouvez remplacer les options de demande client en transmettant une instance remplie de la classe d’options de demande en tant que paramètre aux méthodes d’opération.</span><span class="sxs-lookup"><span data-stu-id="191e1-697">You can override the client request options by passing a populated instance of the request options class as a parameter to operation methods.</span></span>

```csharp
var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
```

<span data-ttu-id="191e1-698">Vous utilisez une instance **OperationContext** pour spécifier le code à exécuter lorsqu’une nouvelle tentative se produit et qu’une opération est terminée.</span><span class="sxs-lookup"><span data-stu-id="191e1-698">You use an **OperationContext** instance to specify the code to execute when a retry occurs and when an operation has completed.</span></span> <span data-ttu-id="191e1-699">Ce code peut collecter des informations sur l’opération à utiliser dans les journaux et la télémétrie</span><span class="sxs-lookup"><span data-stu-id="191e1-699">This code can collect information about the operation for use in logs and telemetry.</span></span>

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

<span data-ttu-id="191e1-700">En plus d’indiquer si un échec donne droit à une nouvelle tentative, les stratégies de nouvelle tentative étendues renvoient un objet **RetryContext** indiquant le nombre de nouvelles tentatives, les résultats de la dernière demande, et si la tentative suivante se déroulera dans l’emplacement principal ou secondaire (voir le tableau ci-dessous pour plus d’informations).</span><span class="sxs-lookup"><span data-stu-id="191e1-700">In addition to indicating whether a failure is suitable for retry, the extended retry policies return a **RetryContext** object that indicates the number of retries, the results of the last request, whether the next retry will happen in the primary or secondary location (see table below for details).</span></span> <span data-ttu-id="191e1-701">Les propriétés de l’objet **RetryContext** peuvent être utilisées pour décider de l’éventualité d’une nouvelle tentative et du moment où elle doit se dérouler.</span><span class="sxs-lookup"><span data-stu-id="191e1-701">The properties of the **RetryContext** object can be used to decide if and when to attempt a retry.</span></span> <span data-ttu-id="191e1-702">Pour plus d’informations, reportez-vous à la [méthode IExtendedRetryPolicy.Evaluate](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx).</span><span class="sxs-lookup"><span data-stu-id="191e1-702">For more details, see [IExtendedRetryPolicy.Evaluate Method](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx).</span></span>

<span data-ttu-id="191e1-703">Les tableaux ci-après présentent les paramètres par défaut pour les stratégies de nouvelle tentative intégrées.</span><span class="sxs-lookup"><span data-stu-id="191e1-703">The following tables show the default settings for the built-in retry policies.</span></span>

<span data-ttu-id="191e1-704">**Options de requête**</span><span class="sxs-lookup"><span data-stu-id="191e1-704">**Request options**</span></span>

| <span data-ttu-id="191e1-705">**Paramètre**</span><span class="sxs-lookup"><span data-stu-id="191e1-705">**Setting**</span></span> | <span data-ttu-id="191e1-706">**Valeur par défaut**</span><span class="sxs-lookup"><span data-stu-id="191e1-706">**Default value**</span></span> | <span data-ttu-id="191e1-707">**Signification**</span><span class="sxs-lookup"><span data-stu-id="191e1-707">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="191e1-708">MaximumExecutionTime</span><span class="sxs-lookup"><span data-stu-id="191e1-708">MaximumExecutionTime</span></span> | <span data-ttu-id="191e1-709">Aucun</span><span class="sxs-lookup"><span data-stu-id="191e1-709">None</span></span> | <span data-ttu-id="191e1-710">Durée d’exécution maximale pour la demande, y compris toutes les nouvelles tentatives potentielles.</span><span class="sxs-lookup"><span data-stu-id="191e1-710">Maximum execution time for the request, including all potential retry attempts.</span></span> <span data-ttu-id="191e1-711">Si elle n’est pas indiquée, la durée d’exécution autorisée d’une demande est illimitée.</span><span class="sxs-lookup"><span data-stu-id="191e1-711">If it is not specified, then the amount of time that a request is permitted to take is unlimited.</span></span> <span data-ttu-id="191e1-712">En d’autres termes, la demande peut se bloquer.</span><span class="sxs-lookup"><span data-stu-id="191e1-712">In other words, the request might hang.</span></span> |
| <span data-ttu-id="191e1-713">ServerTimeout</span><span class="sxs-lookup"><span data-stu-id="191e1-713">ServerTimeout</span></span> | <span data-ttu-id="191e1-714">Aucun</span><span class="sxs-lookup"><span data-stu-id="191e1-714">None</span></span> | <span data-ttu-id="191e1-715">Intervalle du délai d’attente du serveur pour la demande (la valeur est arrondie aux secondes).</span><span class="sxs-lookup"><span data-stu-id="191e1-715">Server timeout interval for the request (value is rounded to seconds).</span></span> <span data-ttu-id="191e1-716">Si non spécifié, il utilisera la valeur par défaut pour toutes les demandes au serveur.</span><span class="sxs-lookup"><span data-stu-id="191e1-716">If not specified, it will use the default value for all requests to the server.</span></span> <span data-ttu-id="191e1-717">En règle générale, la meilleure option consiste à omettre ce paramètre afin que le serveur par défaut soit utilisé.</span><span class="sxs-lookup"><span data-stu-id="191e1-717">Usually, the best option is to omit this setting so that the server default is used.</span></span> | 
| <span data-ttu-id="191e1-718">LocationMode</span><span class="sxs-lookup"><span data-stu-id="191e1-718">LocationMode</span></span> | <span data-ttu-id="191e1-719">Aucun</span><span class="sxs-lookup"><span data-stu-id="191e1-719">None</span></span> | <span data-ttu-id="191e1-720">Si le compte de stockage est créé avec l’option de duplication du stockage géo-redondant avec accès en lecture (RA-GRS), vous pouvez utiliser le mode de d’emplacement pour indiquer l’emplacement devant recevoir la demande.</span><span class="sxs-lookup"><span data-stu-id="191e1-720">If the storage account is created with the Read access geo-redundant storage (RA-GRS) replication option, you can use the location mode to indicate which location should receive the request.</span></span> <span data-ttu-id="191e1-721">Par exemple, si **PrimaryThenSecondary** est spécifié, les demandes sont dans un premier temps toujours envoyées vers l’emplacement principal.</span><span class="sxs-lookup"><span data-stu-id="191e1-721">For example, if **PrimaryThenSecondary** is specified, requests are always sent to the primary location first.</span></span> <span data-ttu-id="191e1-722">En cas d’échec, la demande est envoyée vers l’emplacement secondaire.</span><span class="sxs-lookup"><span data-stu-id="191e1-722">If a request fails, it is sent to the secondary location.</span></span> |
| <span data-ttu-id="191e1-723">RetryPolicy</span><span class="sxs-lookup"><span data-stu-id="191e1-723">RetryPolicy</span></span> | <span data-ttu-id="191e1-724">ExponentialPolicy</span><span class="sxs-lookup"><span data-stu-id="191e1-724">ExponentialPolicy</span></span> | <span data-ttu-id="191e1-725">Voir ci-dessous pour obtenir plus d’informations sur chaque option.</span><span class="sxs-lookup"><span data-stu-id="191e1-725">See below for details of each option.</span></span> |

<span data-ttu-id="191e1-726">**Stratégie exponentielle**</span><span class="sxs-lookup"><span data-stu-id="191e1-726">**Exponential policy**</span></span> 

| <span data-ttu-id="191e1-727">**Paramètre**</span><span class="sxs-lookup"><span data-stu-id="191e1-727">**Setting**</span></span> | <span data-ttu-id="191e1-728">**Valeur par défaut**</span><span class="sxs-lookup"><span data-stu-id="191e1-728">**Default value**</span></span> | <span data-ttu-id="191e1-729">**Signification**</span><span class="sxs-lookup"><span data-stu-id="191e1-729">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="191e1-730">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="191e1-730">maxAttempt</span></span> | <span data-ttu-id="191e1-731">3</span><span class="sxs-lookup"><span data-stu-id="191e1-731">3</span></span> | <span data-ttu-id="191e1-732">Nombre de nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="191e1-732">Number of retry attempts.</span></span> |
| <span data-ttu-id="191e1-733">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="191e1-733">deltaBackoff</span></span> | <span data-ttu-id="191e1-734">4 secondes</span><span class="sxs-lookup"><span data-stu-id="191e1-734">4 seconds</span></span> | <span data-ttu-id="191e1-735">Intervalle de temporisation entre les tentatives.</span><span class="sxs-lookup"><span data-stu-id="191e1-735">Back-off interval between retries.</span></span> <span data-ttu-id="191e1-736">Multiples de ce laps de temps, y compris un élément aléatoire, seront utilisés pour les tentatives suivantes.</span><span class="sxs-lookup"><span data-stu-id="191e1-736">Multiples of this timespan, including a random element, will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="191e1-737">MinBackoff</span><span class="sxs-lookup"><span data-stu-id="191e1-737">MinBackoff</span></span> | <span data-ttu-id="191e1-738">3 secondes</span><span class="sxs-lookup"><span data-stu-id="191e1-738">3 seconds</span></span> | <span data-ttu-id="191e1-739">Ajouté à tous les intervalles des tentatives calculés à partir de deltaBackoff.</span><span class="sxs-lookup"><span data-stu-id="191e1-739">Added to all retry intervals computed from deltaBackoff.</span></span> <span data-ttu-id="191e1-740">Cette valeur ne peut pas être modifiée.</span><span class="sxs-lookup"><span data-stu-id="191e1-740">This value cannot be changed.</span></span>
| <span data-ttu-id="191e1-741">MaxBackoff</span><span class="sxs-lookup"><span data-stu-id="191e1-741">MaxBackoff</span></span> | <span data-ttu-id="191e1-742">120 secondes</span><span class="sxs-lookup"><span data-stu-id="191e1-742">120 seconds</span></span> | <span data-ttu-id="191e1-743">MaxBackoff est utilisé si l’intervalle des tentatives calculé est supérieur à MaxBackoff.</span><span class="sxs-lookup"><span data-stu-id="191e1-743">MaxBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> <span data-ttu-id="191e1-744">Cette valeur ne peut pas être modifiée.</span><span class="sxs-lookup"><span data-stu-id="191e1-744">This value cannot be changed.</span></span> |

<span data-ttu-id="191e1-745">**Stratégie linéaire**</span><span class="sxs-lookup"><span data-stu-id="191e1-745">**Linear policy**</span></span>

| <span data-ttu-id="191e1-746">**Paramètre**</span><span class="sxs-lookup"><span data-stu-id="191e1-746">**Setting**</span></span> | <span data-ttu-id="191e1-747">**Valeur par défaut**</span><span class="sxs-lookup"><span data-stu-id="191e1-747">**Default value**</span></span> | <span data-ttu-id="191e1-748">**Signification**</span><span class="sxs-lookup"><span data-stu-id="191e1-748">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="191e1-749">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="191e1-749">maxAttempt</span></span> | <span data-ttu-id="191e1-750">3</span><span class="sxs-lookup"><span data-stu-id="191e1-750">3</span></span> | <span data-ttu-id="191e1-751">Nombre de nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="191e1-751">Number of retry attempts.</span></span> |
| <span data-ttu-id="191e1-752">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="191e1-752">deltaBackoff</span></span> | <span data-ttu-id="191e1-753">30 secondes</span><span class="sxs-lookup"><span data-stu-id="191e1-753">30 seconds</span></span> | <span data-ttu-id="191e1-754">Intervalle de temporisation entre les tentatives.</span><span class="sxs-lookup"><span data-stu-id="191e1-754">Back-off interval between retries.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="191e1-755">Guide d’utilisation des nouvelles tentatives</span><span class="sxs-lookup"><span data-stu-id="191e1-755">Retry usage guidance</span></span>
<span data-ttu-id="191e1-756">Respectez les consignes suivantes lorsque vous accédez aux services de stockage Azure à l’aide de l’API client de stockage :</span><span class="sxs-lookup"><span data-stu-id="191e1-756">Consider the following guidelines when accessing Azure storage services using the storage client API:</span></span>

* <span data-ttu-id="191e1-757">Utilisez les stratégies de nouvelle tentative intégrées de l’espace de noms Microsoft.WindowsAzure.Storage.RetryPolicies, lorsqu’elles sont adaptées à vos besoins.</span><span class="sxs-lookup"><span data-stu-id="191e1-757">Use the built-in retry policies from the Microsoft.WindowsAzure.Storage.RetryPolicies namespace where they are appropriate for your requirements.</span></span> <span data-ttu-id="191e1-758">Dans la plupart des cas, ces stratégies suffisent.</span><span class="sxs-lookup"><span data-stu-id="191e1-758">In most cases, these policies will be sufficient.</span></span>
* <span data-ttu-id="191e1-759">Utilisez la stratégie **ExponentialRetry** dans les opérations de traitement par lots, les tâches en arrière-plan ou les scénarios non interactifs.</span><span class="sxs-lookup"><span data-stu-id="191e1-759">Use the **ExponentialRetry** policy in batch operations, background tasks, or non-interactive scenarios.</span></span> <span data-ttu-id="191e1-760">Dans ces scénarios, vous pouvez généralement accorder plus de temps à la récupération du service, ce qui par conséquent accroît les chances de réussite de l’opération.</span><span class="sxs-lookup"><span data-stu-id="191e1-760">In these scenarios, you can typically allow more time for the service to recover—with a consequently increased chance of the operation eventually succeeding.</span></span>
* <span data-ttu-id="191e1-761">Vous pouvez spécifier la propriété **MaximumExecutionTime** du paramètre **RequestOptions** pour limiter la durée d’exécution totale. Prenez en compte le type et la taille de l’opération lorsque vous choisissez une valeur de délai d’attente.</span><span class="sxs-lookup"><span data-stu-id="191e1-761">Consider specifying the **MaximumExecutionTime** property of the **RequestOptions** parameter to limit the total execution time, but take into account the type and size of the operation when choosing a timeout value.</span></span>
* <span data-ttu-id="191e1-762">Si vous avez besoin d’implémenter une nouvelle tentative personnalisée, évitez de créer des wrappers autour des classes de client de stockage.</span><span class="sxs-lookup"><span data-stu-id="191e1-762">If you need to implement a custom retry, avoid creating wrappers around the storage client classes.</span></span> <span data-ttu-id="191e1-763">Utilisez plutôt les fonctionnalités permettant d’étendre les stratégies existantes via l’interface **IExtendedRetryPolicy** .</span><span class="sxs-lookup"><span data-stu-id="191e1-763">Instead, use the capabilities to extend the existing policies through the **IExtendedRetryPolicy** interface.</span></span>
* <span data-ttu-id="191e1-764">Si vous utilisez le stockage géo-redondant avec accès en lecture (RA-GRS), vous pouvez utiliser **LocationMode** pour indiquer que les nouvelles tentatives accéderont à la copie en lecture seule secondaire du magasin en cas d’échec de l’accès principal.</span><span class="sxs-lookup"><span data-stu-id="191e1-764">If you are using read access geo-redundant storage (RA-GRS) you can use the **LocationMode** to specify that retry attempts will access the secondary read-only copy of the store should the primary access fail.</span></span> <span data-ttu-id="191e1-765">Toutefois, lorsque vous utilisez cette option vous devez vous assurer que votre application peut fonctionner correctement avec des données qui peuvent être obsolètes si la réplication à partir du magasin principal n’est pas encore terminée.</span><span class="sxs-lookup"><span data-stu-id="191e1-765">However, when using this option you must ensure that your application can work successfully with data that may be stale if the replication from the primary store has not yet completed.</span></span>

<span data-ttu-id="191e1-766">Pensez à commencer par les paramètres suivants pour les opérations liées aux nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="191e1-766">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="191e1-767">Il s’agit de paramètres généraux. Vous devez par conséquent surveiller les opérations et optimiser les valeurs en fonction de votre propre scénario.</span><span class="sxs-lookup"><span data-stu-id="191e1-767">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>  

| <span data-ttu-id="191e1-768">**Contexte**</span><span class="sxs-lookup"><span data-stu-id="191e1-768">**Context**</span></span> | <span data-ttu-id="191e1-769">**Exemple de cible E2E<br />latence max**</span><span class="sxs-lookup"><span data-stu-id="191e1-769">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="191e1-770">**Stratégie de nouvelle tentative**</span><span class="sxs-lookup"><span data-stu-id="191e1-770">**Retry policy**</span></span> | <span data-ttu-id="191e1-771">**Paramètres**</span><span class="sxs-lookup"><span data-stu-id="191e1-771">**Settings**</span></span> | <span data-ttu-id="191e1-772">**Valeurs**</span><span class="sxs-lookup"><span data-stu-id="191e1-772">**Values**</span></span> | <span data-ttu-id="191e1-773">**Fonctionnement**</span><span class="sxs-lookup"><span data-stu-id="191e1-773">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="191e1-774">Interactif, interface utilisateur,</span><span class="sxs-lookup"><span data-stu-id="191e1-774">Interactive, UI,</span></span><br /><span data-ttu-id="191e1-775">ou premier plan</span><span class="sxs-lookup"><span data-stu-id="191e1-775">or foreground</span></span> |<span data-ttu-id="191e1-776">2 secondes</span><span class="sxs-lookup"><span data-stu-id="191e1-776">2 seconds</span></span> |<span data-ttu-id="191e1-777">Linéaire</span><span class="sxs-lookup"><span data-stu-id="191e1-777">Linear</span></span> |<span data-ttu-id="191e1-778">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="191e1-778">maxAttempt</span></span><br /><span data-ttu-id="191e1-779">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="191e1-779">deltaBackoff</span></span> |<span data-ttu-id="191e1-780">3</span><span class="sxs-lookup"><span data-stu-id="191e1-780">3</span></span><br /><span data-ttu-id="191e1-781">500 ms</span><span class="sxs-lookup"><span data-stu-id="191e1-781">500 ms</span></span> |<span data-ttu-id="191e1-782">Tentative 1 - délai 500 ms</span><span class="sxs-lookup"><span data-stu-id="191e1-782">Attempt 1 - delay 500 ms</span></span><br /><span data-ttu-id="191e1-783">Tentative 2 - délai 500 ms</span><span class="sxs-lookup"><span data-stu-id="191e1-783">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="191e1-784">Tentative 3 - délai 500 ms</span><span class="sxs-lookup"><span data-stu-id="191e1-784">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="191e1-785">Arrière-plan</span><span class="sxs-lookup"><span data-stu-id="191e1-785">Background</span></span><br /><span data-ttu-id="191e1-786">ou lot</span><span class="sxs-lookup"><span data-stu-id="191e1-786">or batch</span></span> |<span data-ttu-id="191e1-787">30 secondes</span><span class="sxs-lookup"><span data-stu-id="191e1-787">30 seconds</span></span> |<span data-ttu-id="191e1-788">Exponentielle</span><span class="sxs-lookup"><span data-stu-id="191e1-788">Exponential</span></span> |<span data-ttu-id="191e1-789">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="191e1-789">maxAttempt</span></span><br /><span data-ttu-id="191e1-790">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="191e1-790">deltaBackoff</span></span> |<span data-ttu-id="191e1-791">5.</span><span class="sxs-lookup"><span data-stu-id="191e1-791">5</span></span><br /><span data-ttu-id="191e1-792">4 secondes</span><span class="sxs-lookup"><span data-stu-id="191e1-792">4 seconds</span></span> |<span data-ttu-id="191e1-793">Tentative 1 - délai ~3 sec</span><span class="sxs-lookup"><span data-stu-id="191e1-793">Attempt 1 - delay ~3 sec</span></span><br /><span data-ttu-id="191e1-794">Tentative 2 - délai ~7 sec</span><span class="sxs-lookup"><span data-stu-id="191e1-794">Attempt 2 - delay ~7 sec</span></span><br /><span data-ttu-id="191e1-795">Tentative 3 - délai ~15 sec</span><span class="sxs-lookup"><span data-stu-id="191e1-795">Attempt 3 - delay ~15 sec</span></span> |

### <a name="telemetry"></a><span data-ttu-id="191e1-796">Télémétrie</span><span class="sxs-lookup"><span data-stu-id="191e1-796">Telemetry</span></span>
<span data-ttu-id="191e1-797">Les nouvelles tentatives sont enregistrées dans un **TraceSource**.</span><span class="sxs-lookup"><span data-stu-id="191e1-797">Retry attempts are logged to a **TraceSource**.</span></span> <span data-ttu-id="191e1-798">Vous devez configurer un **TraceListener** pour capturer les événements et les écrire dans un journal de destination approprié.</span><span class="sxs-lookup"><span data-stu-id="191e1-798">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span> <span data-ttu-id="191e1-799">Vous pouvez utiliser le **TextWriterTraceListener** ou **XmlWriterTraceListener** pour écrire les données dans un fichier journal, le **EventLogTraceListener** pour écrire dans le journal des événements Windows, ou le **EventProviderTraceListener** pour écrire les données de trace dans le sous-système ETW.</span><span class="sxs-lookup"><span data-stu-id="191e1-799">You can use the **TextWriterTraceListener** or **XmlWriterTraceListener** to write the data to a log file, the **EventLogTraceListener** to write to the Windows Event Log, or the **EventProviderTraceListener** to write trace data to the ETW subsystem.</span></span> <span data-ttu-id="191e1-800">Vous pouvez également configurer le vidage automatique de la mémoire tampon et le niveau de détail des événements qui seront enregistrés (par exemple, erreur, avertissement, information et informations détaillées).</span><span class="sxs-lookup"><span data-stu-id="191e1-800">You can also configure auto-flushing of the buffer, and the verbosity of events that will be logged (for example, Error, Warning, Informational, and Verbose).</span></span> <span data-ttu-id="191e1-801">Pour plus d’informations, consultez [Journalisation côté client avec la bibliothèque cliente de stockage .NET](http://msdn.microsoft.com/library/azure/dn782839.aspx).</span><span class="sxs-lookup"><span data-stu-id="191e1-801">For more information, see [Client-side Logging with the .NET Storage Client Library](http://msdn.microsoft.com/library/azure/dn782839.aspx).</span></span>

<span data-ttu-id="191e1-802">Les opérations peuvent recevoir une instance **OperationContext**, qui expose un événement **Retrying** pouvant être utilisé pour associer la logique personnalisée de télémétrie.</span><span class="sxs-lookup"><span data-stu-id="191e1-802">Operations can receive an **OperationContext** instance, which exposes a **Retrying** event that can be used to attach custom telemetry logic.</span></span> <span data-ttu-id="191e1-803">Pour plus d’informations, consultez [OperationContext.Retrying Event](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).</span><span class="sxs-lookup"><span data-stu-id="191e1-803">For more information, see [OperationContext.Retrying Event](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).</span></span>

### <a name="examples"></a><span data-ttu-id="191e1-804">Exemples</span><span class="sxs-lookup"><span data-stu-id="191e1-804">Examples</span></span>
<span data-ttu-id="191e1-805">L’exemple de code suivant montre comment créer deux instances **TableRequestOptions** avec des paramètres de nouvelle tentative différents ; une pour les demandes interactives et une pour les demandes d’arrière-plan.</span><span class="sxs-lookup"><span data-stu-id="191e1-805">The following code example shows how to create two **TableRequestOptions** instances with different retry settings; one for interactive requests and one for background requests.</span></span> <span data-ttu-id="191e1-806">L’exemple définit ensuite ces deux jeux stratégies de nouvelle tentative sur le client de sorte qu’elles s’appliquent à toutes les demandes et définit également la stratégie interactive sur une demande spécifique afin qu’elle remplace les paramètres par défaut appliqués au client.</span><span class="sxs-lookup"><span data-stu-id="191e1-806">The example then sets these two retry policies on the client so that they apply for all requests, and also sets the interactive strategy on a specific request so that it overrides the default settings applied to the client.</span></span>

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

### <a name="more-information"></a><span data-ttu-id="191e1-807">Plus d’informations</span><span class="sxs-lookup"><span data-stu-id="191e1-807">More information</span></span>
* [<span data-ttu-id="191e1-808">Recommandations de stratégie de nouvelle tentative de Bibliothèque cliente de stockage Azure</span><span class="sxs-lookup"><span data-stu-id="191e1-808">Azure Storage Client Library Retry Policy Recommendations</span></span>](https://azure.microsoft.com/blog/2014/05/22/azure-storage-client-library-retry-policy-recommendations/)
* [<span data-ttu-id="191e1-809">Bibliothèque cliente de stockage 2.0 - Mise en œuvre de stratégies de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="191e1-809">Storage Client Library 2.0 – Implementing Retry Policies</span></span>](http://gauravmantri.com/2012/12/30/storage-client-library-2-0-implementing-retry-policies/)

## <a name="general-rest-and-retry-guidelines"></a><span data-ttu-id="191e1-810">Instructions générales relatives aux nouvelles tentatives et à REST</span><span class="sxs-lookup"><span data-stu-id="191e1-810">General REST and retry guidelines</span></span>
<span data-ttu-id="191e1-811">Prenez en compte les éléments suivants lors de l’accès aux services Azure ou tiers :</span><span class="sxs-lookup"><span data-stu-id="191e1-811">Consider the following when accessing Azure or third party services:</span></span>

* <span data-ttu-id="191e1-812">Utilisez une approche systématique pour gérer les nouvelles tentatives, peut-être en tant que code réutilisable, afin de pouvoir appliquer une méthodologie cohérente sur tous les clients et toutes les solutions.</span><span class="sxs-lookup"><span data-stu-id="191e1-812">Use a systematic approach to managing retries, perhaps as reusable code, so that you can apply a consistent methodology across all clients and all solutions.</span></span>
* <span data-ttu-id="191e1-813">Envisagez d’utiliser une infrastructure de nouvelle tentative, telle que le bloc applicatif de gestion des erreurs temporaires pour gérer les nouvelles tentatives si le service cible ou le client ne dispose d’aucun mécanisme de nouvelle tentative intégré.</span><span class="sxs-lookup"><span data-stu-id="191e1-813">Consider using a retry framework such as the Transient Fault Handling Application Block to manage retries if the target service or client has no built-in retry mechanism.</span></span> <span data-ttu-id="191e1-814">Cela vous permettra d’implémenter un comportement cohérent de nouvelle tentative, et peut fournir une stratégie de nouvelle tentative par défaut appropriée pour le service cible.</span><span class="sxs-lookup"><span data-stu-id="191e1-814">This will help you implement a consistent retry behavior, and it may provide a suitable default retry strategy for the target service.</span></span> <span data-ttu-id="191e1-815">Toutefois, vous devrez peut-être créer un code personnalisé de nouvelle tentative pour les services ne disposant pas d’un comportement standard, qui ne reposent pas sur les exceptions pour indiquer les erreurs temporaires, ou si vous souhaitez utiliser une réponse **Retry-Response** pour gérer le comportement de la nouvelle tentative.</span><span class="sxs-lookup"><span data-stu-id="191e1-815">However, you may need to create custom retry code for services that have non-standard behavior, that do not rely on exceptions to indicate transient failures, or if you want to use a **Retry-Response** reply to manage retry behavior.</span></span>
* <span data-ttu-id="191e1-816">La logique de détection temporaire dépend de l’API client réelle utilisée pour appeler les appels REST.</span><span class="sxs-lookup"><span data-stu-id="191e1-816">The transient detection logic will depend on the actual client API you use to invoke the REST calls.</span></span> <span data-ttu-id="191e1-817">Certains clients, tels que la nouvelle classe **HttpClient** , ne lanceront pas d’exceptions pour les demandes terminées avec un code d’état HTTP indiquant un échec.</span><span class="sxs-lookup"><span data-stu-id="191e1-817">Some clients, such as the newer **HttpClient** class, will not throw exceptions for completed requests with a non-success HTTP status code.</span></span> <span data-ttu-id="191e1-818">Cela améliore les performances, mais vous empêche d’utiliser le bloc applicatif de gestion des erreurs temporaires.</span><span class="sxs-lookup"><span data-stu-id="191e1-818">This improves performance but prevents the use of the Transient Fault Handling Application Block.</span></span> <span data-ttu-id="191e1-819">Dans ce cas, vous pourriez encapsuler l’appel à l’API REST avec le code qui génère des exceptions pour les codes d’état HTTP indiquant un échec, qui peuvent ensuite être traités par le bloc.</span><span class="sxs-lookup"><span data-stu-id="191e1-819">In this case you could wrap the call to the REST API with code that produces exceptions for non-success HTTP status codes, which can then be processed by the block.</span></span> <span data-ttu-id="191e1-820">Vous pouvez également utiliser un mécanisme différent pour piloter les nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="191e1-820">Alternatively, you can use a different mechanism to drive the retries.</span></span>
* <span data-ttu-id="191e1-821">Le code d’état HTTP renvoyé par le service peut permettre d’indiquer si l’échec est temporaire.</span><span class="sxs-lookup"><span data-stu-id="191e1-821">The HTTP status code returned from the service can help to indicate whether the failure is transient.</span></span> <span data-ttu-id="191e1-822">Vous devrez peut-être examiner les exceptions générées par un client ou l’infrastructure de nouvelle tentative pour accéder au code d’état ou pour déterminer le type d’exception équivalent.</span><span class="sxs-lookup"><span data-stu-id="191e1-822">You may need to examine the exceptions generated by a client or the retry framework to access the status code or to determine the equivalent exception type.</span></span> <span data-ttu-id="191e1-823">Les codes HTTP suivants indiquent habituellement qu’une nouvelle tentative est appropriée :</span><span class="sxs-lookup"><span data-stu-id="191e1-823">The following HTTP codes typically indicate that a retry is appropriate:</span></span>
  * <span data-ttu-id="191e1-824">408 Délai d’expiration de la requête</span><span class="sxs-lookup"><span data-stu-id="191e1-824">408 Request Timeout</span></span>
  * <span data-ttu-id="191e1-825">429 Trop de demandes</span><span class="sxs-lookup"><span data-stu-id="191e1-825">429 Too Many Requests</span></span>
  * <span data-ttu-id="191e1-826">500 Erreur interne du serveur</span><span class="sxs-lookup"><span data-stu-id="191e1-826">500 Internal Server Error</span></span>
  * <span data-ttu-id="191e1-827">502 Passerelle incorrecte</span><span class="sxs-lookup"><span data-stu-id="191e1-827">502 Bad Gateway</span></span>
  * <span data-ttu-id="191e1-828">503 Service indisponible</span><span class="sxs-lookup"><span data-stu-id="191e1-828">503 Service Unavailable</span></span>
  * <span data-ttu-id="191e1-829">504 Dépassement du délai de la passerelle</span><span class="sxs-lookup"><span data-stu-id="191e1-829">504 Gateway Timeout</span></span>
* <span data-ttu-id="191e1-830">Si vous basez votre logique de nouvelle tentative sur les exceptions, ce qui suit indique généralement un échec temporaire lorsqu’aucune connexion n’a pu être établie :</span><span class="sxs-lookup"><span data-stu-id="191e1-830">If you base your retry logic on exceptions, the following typically indicate a transient failure where no connection could be established:</span></span>
  * <span data-ttu-id="191e1-831">WebExceptionStatus.ConnectionClosed</span><span class="sxs-lookup"><span data-stu-id="191e1-831">WebExceptionStatus.ConnectionClosed</span></span>
  * <span data-ttu-id="191e1-832">WebExceptionStatus.ConnectFailure</span><span class="sxs-lookup"><span data-stu-id="191e1-832">WebExceptionStatus.ConnectFailure</span></span>
  * <span data-ttu-id="191e1-833">WebExceptionStatus.Timeout</span><span class="sxs-lookup"><span data-stu-id="191e1-833">WebExceptionStatus.Timeout</span></span>
  * <span data-ttu-id="191e1-834">WebExceptionStatus.RequestCanceled</span><span class="sxs-lookup"><span data-stu-id="191e1-834">WebExceptionStatus.RequestCanceled</span></span>
* <span data-ttu-id="191e1-835">Dans le cas d’un service à l’état non disponible, le service peut indiquer le délai approprié avant l’exécution d’une nouvelle tentative dans l’en-tête de réponse **Retry-After** ou dans un en-tête personnalisé différent.</span><span class="sxs-lookup"><span data-stu-id="191e1-835">In the case of a service unavailable status, the service might indicate the appropriate delay before retrying in the **Retry-After** response header or a different custom header.</span></span> <span data-ttu-id="191e1-836">Les services peuvent également envoyer des informations supplémentaires sous la forme d’en-têtes personnalisés ou en les intégrant au contenu de la réponse.</span><span class="sxs-lookup"><span data-stu-id="191e1-836">Services might also send additional information as custom headers, or embedded in the content of the response.</span></span> <span data-ttu-id="191e1-837">Le bloc applicatif de gestion des erreurs temporaires ne peut pas utiliser les en-têtes « retry-after » personnalisés ou standard.</span><span class="sxs-lookup"><span data-stu-id="191e1-837">The Transient Fault Handling Application Block cannot use the standard or any custom “retry-after” headers.</span></span>
* <span data-ttu-id="191e1-838">N’effectuez pas de nouvelle tentative pour les codes d’état représentant des erreurs du client (erreurs comprises dans la plage 4xx), excepté pour le code 408 Délai d’expiration de la requête.</span><span class="sxs-lookup"><span data-stu-id="191e1-838">Do not retry for status codes representing client errors (errors in the 4xx range) except for a 408 Request Timeout.</span></span>
* <span data-ttu-id="191e1-839">Testez minutieusement vos stratégies et mécanismes de nouvelle tentative sous différentes conditions, en utilisant différents états de réseau et en variant les charges du système.</span><span class="sxs-lookup"><span data-stu-id="191e1-839">Thoroughly test your retry strategies and mechanisms under a range of conditions, such as different network states and varying system loadings.</span></span>

### <a name="retry-strategies"></a><span data-ttu-id="191e1-840">Stratégie de nouvelle tentative</span><span class="sxs-lookup"><span data-stu-id="191e1-840">Retry strategies</span></span>
<span data-ttu-id="191e1-841">Voici les types d’intervalles de stratégie de nouvelle tentative classiques :</span><span class="sxs-lookup"><span data-stu-id="191e1-841">The following are the typical types of retry strategy intervals:</span></span>

* <span data-ttu-id="191e1-842">**Exponentielle**.</span><span class="sxs-lookup"><span data-stu-id="191e1-842">**Exponential**.</span></span> <span data-ttu-id="191e1-843">Une stratégie de nouvelle tentative qui effectue un nombre spécifié de tentatives en utilisant une approche de secours exponentielle répartie de manière aléatoire pour déterminer l’intervalle entre deux tentatives.</span><span class="sxs-lookup"><span data-stu-id="191e1-843">A retry policy that performs a specified number of retries, using a randomized exponential back off approach to determine the interval between retries.</span></span> <span data-ttu-id="191e1-844">Par exemple : </span><span class="sxs-lookup"><span data-stu-id="191e1-844">For example:</span></span>

    ```csharp
    var random = new Random();

    var delta = (int)((Math.Pow(2.0, currentRetryCount) - 1.0) *
                random.Next((int)(this.deltaBackoff.TotalMilliseconds * 0.8),
                (int)(this.deltaBackoff.TotalMilliseconds * 1.2)));
    var interval = (int)Math.Min(checked(this.minBackoff.TotalMilliseconds + delta),
                    this.maxBackoff.TotalMilliseconds);
    retryInterval = TimeSpan.FromMilliseconds(interval);
    ```

* <span data-ttu-id="191e1-845">**Incrémentielle**.</span><span class="sxs-lookup"><span data-stu-id="191e1-845">**Incremental**.</span></span> <span data-ttu-id="191e1-846">Une stratégie de nouvelle tentative avec un nombre spécifié de nouvelles tentatives et un intervalle de temps incrémentiel entre deux tentatives.</span><span class="sxs-lookup"><span data-stu-id="191e1-846">A retry strategy with a specified number of retry attempts and an incremental time interval between retries.</span></span> <span data-ttu-id="191e1-847">Par exemple : </span><span class="sxs-lookup"><span data-stu-id="191e1-847">For example:</span></span>

    ```csharp
    retryInterval = TimeSpan.FromMilliseconds(this.initialInterval.TotalMilliseconds +
                    (this.increment.TotalMilliseconds * currentRetryCount));
    ```

* <span data-ttu-id="191e1-848">**LinearRetry**.</span><span class="sxs-lookup"><span data-stu-id="191e1-848">**LinearRetry**.</span></span> <span data-ttu-id="191e1-849">Une stratégie de nouvelle tentative qui effectue un nombre spécifié de nouvelles tentatives en utilisant un intervalle fixe spécifique entre deux tentatives.</span><span class="sxs-lookup"><span data-stu-id="191e1-849">A retry policy that performs a specified number of retries, using a specified fixed time interval between retries.</span></span> <span data-ttu-id="191e1-850">Par exemple : </span><span class="sxs-lookup"><span data-stu-id="191e1-850">For example:</span></span>

    ```csharp
    retryInterval = this.deltaBackoff;
    ```

### <a name="transient-fault-handling-with-polly"></a><span data-ttu-id="191e1-851">Gestion des erreurs temporaires avec Polly</span><span class="sxs-lookup"><span data-stu-id="191e1-851">Transient fault handling with Polly</span></span>
<span data-ttu-id="191e1-852">[Polly] [polly]est une bibliothèque permettant de gérer par programme les nouvelles tentatives et les stratégies de [disjoncteur][circuit-breaker].</span><span class="sxs-lookup"><span data-stu-id="191e1-852">[Polly][polly] is a library to programatically handle retries and [circuit breaker][circuit-breaker] strategies.</span></span> <span data-ttu-id="191e1-853">Le projet Polly est un membre de l’organisation [.NET Foundation][dotnet-foundation].</span><span class="sxs-lookup"><span data-stu-id="191e1-853">The Polly project is a member of the [.NET Foundation][dotnet-foundation].</span></span> <span data-ttu-id="191e1-854">Dans le cas des services pour lesquels le client ne prend pas en charge les nouvelles tentatives en mode natif, Polly constitue une alternative valide et évite d’avoir à écrire un code de nouvelle tentative personnalisé, qui peut être difficile à implémenter correctement.</span><span class="sxs-lookup"><span data-stu-id="191e1-854">For services where the client does not natively support retries, Polly is a valid alternative and avoids the need to write custom retry code, which can be hard to implement correctly.</span></span> <span data-ttu-id="191e1-855">Polly vous offre également un moyen de tracer les erreurs lorsqu’elles se produisent, ce qui vous permet de journaliser les nouvelles tentatives.</span><span class="sxs-lookup"><span data-stu-id="191e1-855">Polly also provides a way to trace errors when they occur, so that you can log retries.</span></span>


<!-- links -->

[adal]: /azure/active-directory/develop/active-directory-authentication-libraries
[autorest]: https://github.com/Azure/autorest/tree/master/docs
[circuit-breaker]: ../patterns/circuit-breaker.md
[ConnectionPolicy.RetryOptions]: https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.connectionpolicy.retryoptions.aspx
[dotnet-foundation]: https://dotnetfoundation.org/
[polly]: http://www.thepollyproject.org
[redis-cache-troubleshoot]: /azure/redis-cache/cache-how-to-troubleshoot
[SearchIndexClient]: https://msdn.microsoft.com/library/azure/microsoft.azure.search.searchindexclient.aspx
[SearchServiceClient]: https://msdn.microsoft.com/library/microsoft.azure.search.searchserviceclient.aspx


### <a name="more-information"></a><span data-ttu-id="191e1-859">Plus d’informations</span><span class="sxs-lookup"><span data-stu-id="191e1-859">More information</span></span>
* [<span data-ttu-id="191e1-860">Résilience de connexion</span><span class="sxs-lookup"><span data-stu-id="191e1-860">Connection Resiliency</span></span>](/ef/core/miscellaneous/connection-resiliency)
* [<span data-ttu-id="191e1-861">Data Points - EF Core 1.1 (Points de données - EF Core 1.1)</span><span class="sxs-lookup"><span data-stu-id="191e1-861">Data Points - EF Core 1.1</span></span>](https://msdn.microsoft.com/magazine/mt745093.aspx)


