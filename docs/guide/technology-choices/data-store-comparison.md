---
title: Critères de sélection d’une banque de données
description: Vue d’ensemble des options de calcul Azure
author: MikeWasson
ms.date: 06/01/2018
ms.openlocfilehash: f8996cdeb937a28b3f3056da3921a3f89dd36b1a
ms.sourcegitcommit: dbbf914757b03cdee7a274204f9579fa63d7eed2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/02/2018
ms.locfileid: "50916428"
---
# <a name="criteria-for-choosing-a-data-store"></a><span data-ttu-id="0bdba-103">Critères de sélection d’une banque de données</span><span class="sxs-lookup"><span data-stu-id="0bdba-103">Criteria for choosing a data store</span></span>

<span data-ttu-id="0bdba-104">Azure prend en charge divers types de solutions de stockage de données, chacune offrant différentes fonctionnalités et capacités.</span><span class="sxs-lookup"><span data-stu-id="0bdba-104">Azure supports many types of data storage solutions, each providing different features and capabilities.</span></span> <span data-ttu-id="0bdba-105">Cet article décrit les critères de comparaison que vous devez employer pour évaluer une banque de données.</span><span class="sxs-lookup"><span data-stu-id="0bdba-105">This article describes the comparison criteria you should use when evaluating a data store.</span></span> <span data-ttu-id="0bdba-106">L’objectif ici est de vous aider à identifier les types de stockage de données qui vous permettront de répondre aux besoins de votre solution.</span><span class="sxs-lookup"><span data-stu-id="0bdba-106">The goal is to help you determine which data storage types can meet your solution's requirements.</span></span>

## <a name="general-considerations"></a><span data-ttu-id="0bdba-107">Considérations d’ordre général</span><span class="sxs-lookup"><span data-stu-id="0bdba-107">General Considerations</span></span>

<span data-ttu-id="0bdba-108">Pour débuter votre comparaison, rassemblez le maximum d’informations ci-dessous pour définir vos besoins en données.</span><span class="sxs-lookup"><span data-stu-id="0bdba-108">To start your comparison, gather as much of the following information as you can about your data needs.</span></span> <span data-ttu-id="0bdba-109">Ces informations vous aideront à identifier les types de stockage de données qui répondront à vos besoins.</span><span class="sxs-lookup"><span data-stu-id="0bdba-109">This information will help you to determine which data storage types will meet your needs.</span></span>

### <a name="functional-requirements"></a><span data-ttu-id="0bdba-110">Spécifications fonctionnelles</span><span class="sxs-lookup"><span data-stu-id="0bdba-110">Functional requirements</span></span>

- <span data-ttu-id="0bdba-111">**Format de données** :</span><span class="sxs-lookup"><span data-stu-id="0bdba-111">**Data format**.</span></span> <span data-ttu-id="0bdba-112">quel type de données prévoyez-vous de stocker ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-112">What type of data are you intending to store?</span></span> <span data-ttu-id="0bdba-113">Les données transactionnelles, les objets JSON, les données de télémétrie, les index de recherche et les fichiers plats comptent parmi les types courants.</span><span class="sxs-lookup"><span data-stu-id="0bdba-113">Common types include transactional data, JSON objects, telemetry, search indexes, or flat files.</span></span>
- <span data-ttu-id="0bdba-114">**Taille des données** :</span><span class="sxs-lookup"><span data-stu-id="0bdba-114">**Data size**.</span></span> <span data-ttu-id="0bdba-115">quelle est la taille des entités que vous avez besoin de stocker ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-115">How large are the entities you need to store?</span></span> <span data-ttu-id="0bdba-116">Ces entités devront-elles être conservées dans un document unique ou peuvent-elles être scindées en plusieurs documents, tables, collections, etc ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-116">Will these entities need to be maintained as a single document, or can they be split across multiple documents, tables, collections, and so forth?</span></span>
- <span data-ttu-id="0bdba-117">**Échelle et structure** :</span><span class="sxs-lookup"><span data-stu-id="0bdba-117">**Scale and structure**.</span></span> <span data-ttu-id="0bdba-118">de quelle capacité de stockage globale avez-vous besoin ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-118">What is the overall amount of storage capacity you need?</span></span> <span data-ttu-id="0bdba-119">Prévoyez-vous de partitionner vos données ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-119">Do you anticipate partitioning your data?</span></span> 
- <span data-ttu-id="0bdba-120">**Relations entre les données** :</span><span class="sxs-lookup"><span data-stu-id="0bdba-120">**Data relationships**.</span></span> <span data-ttu-id="0bdba-121">vos données devront-elles prendre en charge des relations un-à-plusieurs ou plusieurs à plusieurs ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-121">Will your data need to support one-to-many or many-to-many relationships?</span></span> <span data-ttu-id="0bdba-122">Les relations proprement dites constituent-elles une part importante des données ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-122">Are relationships themselves an important part of the data?</span></span> <span data-ttu-id="0bdba-123">Prévoyez-vous de joindre ou de combiner les données d’un même jeu de données ou de plusieurs jeux de données externes ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-123">Will you need to join or otherwise combine data from within the same dataset, or from external datasets?</span></span> 
- <span data-ttu-id="0bdba-124">**Modèle de cohérence** :</span><span class="sxs-lookup"><span data-stu-id="0bdba-124">**Consistency model**.</span></span> <span data-ttu-id="0bdba-125">quelle importance accordez-vous à ce que les mises à jour effectuées dans un nœud apparaissent dans les autres nœuds, avant que d’autres modifications puissent avoir lieu ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-125">How important is it for updates made in one node to appear in other nodes, before further changes can be made?</span></span> <span data-ttu-id="0bdba-126">Pouvez-vous accepter la cohérence éventuelle ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-126">Can you accept eventual consistency?</span></span> <span data-ttu-id="0bdba-127">Avez-vous besoin de garanties ACID pour les transactions ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-127">Do you need ACID guarantees for transactions?</span></span>
- <span data-ttu-id="0bdba-128">**Flexibilité de schéma** :</span><span class="sxs-lookup"><span data-stu-id="0bdba-128">**Schema flexibility**.</span></span> <span data-ttu-id="0bdba-129">quel type de schéma appliquerez-vous à vos données ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-129">What kind of schemas will you apply to your data?</span></span> <span data-ttu-id="0bdba-130">Allez-vous utiliser un schéma fixe, une approche de schéma à l’écriture ou une approche de schéma à la lecture ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-130">Will you use a fixed schema, a schema-on-write approach, or a schema-on-read approach?</span></span>
- <span data-ttu-id="0bdba-131">**Accès concurrentiel**.</span><span class="sxs-lookup"><span data-stu-id="0bdba-131">**Concurrency**.</span></span> <span data-ttu-id="0bdba-132">quel type de mécanisme d’accès concurrentiel souhaitez-vous utiliser durant la mise à jour et la synchronisation des données ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-132">What kind of concurrency mechanism do you want to use when updating and synchronizing data?</span></span> <span data-ttu-id="0bdba-133">L’application sera-t-elle appelée à procéder à de nombreuses mises à jour qui pourraient potentiellement entrer en conflit ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-133">Will the application perform many updates that could potentially conflict.</span></span> <span data-ttu-id="0bdba-134">Dans ce cas, vous aurez peut-être besoin du verrouillage des enregistrements et du contrôle d’accès concurrentiel pessimiste.</span><span class="sxs-lookup"><span data-stu-id="0bdba-134">If so, you may require record locking and pessimistic concurrency control.</span></span> <span data-ttu-id="0bdba-135">Sinon, pouvez vous prendre en charge les contrôles d’accès concurrentiel optimiste ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-135">Alternatively, can you support optimistic concurrency controls?</span></span> <span data-ttu-id="0bdba-136">Dans ce cas, le contrôle d’accès concurrentiel basé sur l’horodatage est-il suffisant ou avez-vous besoin de l’ajout de la fonctionnalité de contrôle d’accès concurrentiel multiversion ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-136">If so, is simple timestamp-based concurrency control enough, or do you need the added functionality of multi-version concurrency control?</span></span>
- <span data-ttu-id="0bdba-137">**Déplacement des données** :</span><span class="sxs-lookup"><span data-stu-id="0bdba-137">**Data movement**.</span></span> <span data-ttu-id="0bdba-138">votre solution devra-t-elle effectuer des tâches ETL pour déplacer les données vers d’autres banques ou entrepôts de données ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-138">Will your solution need to perform ETL tasks to move data to other stores or data warehouses?</span></span>
- <span data-ttu-id="0bdba-139">**Cycle de vie des données** :</span><span class="sxs-lookup"><span data-stu-id="0bdba-139">**Data lifecycle**.</span></span> <span data-ttu-id="0bdba-140">les données sont-elles écrites une fois et lues autant de fois que souhaité ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-140">Is the data write-once, read-many?</span></span> <span data-ttu-id="0bdba-141">Peuvent-elles être déplacées dans un espace de stockage froid ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-141">Can it be moved into cool or cold storage?</span></span>
- <span data-ttu-id="0bdba-142">**Autres fonctionnalités prises en charge** :</span><span class="sxs-lookup"><span data-stu-id="0bdba-142">**Other supported features**.</span></span> <span data-ttu-id="0bdba-143">avez-vous besoin d’autres fonctionnalités spécifiques, telles que la validation de schéma, l’agrégation, l’indexation, la recherche en texte intégral, MapReduce ou d’autres fonctions de requête ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-143">Do you need any other specific features, such as schema validation, aggregation, indexing, full-text search, MapReduce, or other query capabilities?</span></span>

### <a name="non-functional-requirements"></a><span data-ttu-id="0bdba-144">Spécifications non fonctionnelles</span><span class="sxs-lookup"><span data-stu-id="0bdba-144">Non-functional requirements</span></span>

- <span data-ttu-id="0bdba-145">**Performances et scalabilité** :</span><span class="sxs-lookup"><span data-stu-id="0bdba-145">**Performance and scalability**.</span></span> <span data-ttu-id="0bdba-146">quels sont vos besoins en matière de performances de données ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-146">What are your data performance requirements?</span></span> <span data-ttu-id="0bdba-147">Avez-vous des exigences spécifiques concernant la vitesse d’ingestion des données et la vitesse de traitement des données ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-147">Do you have specific requirements for data ingestion rates and data processing rates?</span></span> <span data-ttu-id="0bdba-148">Quels sont les temps de réponse acceptables pour l’interrogation et l’agrégation des données une fois celles-ci ingérées ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-148">What are the acceptable response times for querying and aggregation of data once ingested?</span></span> <span data-ttu-id="0bdba-149">De quelle taille aurez-vous besoin pour augmenter les capacités de la banque de données ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-149">How large will you need the data store to scale up?</span></span> <span data-ttu-id="0bdba-150">Votre charge de travail est-elle plutôt axée sur la lecture ou sur l’écriture ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-150">Is your workload more read-heavy or write-heavy?</span></span>
- <span data-ttu-id="0bdba-151">**Fiabilité** :</span><span class="sxs-lookup"><span data-stu-id="0bdba-151">**Reliability**.</span></span> <span data-ttu-id="0bdba-152">quel contrat SLA global devez-vous prendre en charge ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-152">What overall SLA do you need to support?</span></span> <span data-ttu-id="0bdba-153">Quel niveau de tolérance aux pannes devez-vous offrir aux consommateurs de données ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-153">What level of fault-tolerance do you need to provide for data consumers?</span></span> <span data-ttu-id="0bdba-154">De quelles fonctionnalités de sauvegarde et de restauration avez-vous besoin ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-154">What kind of backup and restore capabilities do you need?</span></span> 
- <span data-ttu-id="0bdba-155">**Réplication** :</span><span class="sxs-lookup"><span data-stu-id="0bdba-155">**Replication**.</span></span> <span data-ttu-id="0bdba-156">vos données devront-elles être réparties entre plusieurs réplicas ou régions ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-156">Will your data need to be distributed among multiple replicas or regions?</span></span> <span data-ttu-id="0bdba-157">De quel type de fonctionnalités de réplication de données avez-vous besoin ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-157">What kind of data replication capabilities do you require?</span></span> 
- <span data-ttu-id="0bdba-158">**Limites** :</span><span class="sxs-lookup"><span data-stu-id="0bdba-158">**Limits**.</span></span> <span data-ttu-id="0bdba-159">les limites d’une banque de données déterminée répondent-elles à vos besoins en matière d’échelle, de nombre de connexions et de débit ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-159">Will the limits of a particular data store support your requirements for scale, number of connections, and throughput?</span></span> 

### <a name="management-and-cost"></a><span data-ttu-id="0bdba-160">Gestion et coût</span><span class="sxs-lookup"><span data-stu-id="0bdba-160">Management and cost</span></span>

- <span data-ttu-id="0bdba-161">**Service managé** :</span><span class="sxs-lookup"><span data-stu-id="0bdba-161">**Managed service**.</span></span> <span data-ttu-id="0bdba-162">si possible, utilisez un service de données managé, sauf si vous avez besoin de fonctionnalités spécifiques que seule une banque de données hébergée par IaaS peut offrir.</span><span class="sxs-lookup"><span data-stu-id="0bdba-162">When possible, use a managed data service, unless you require specific capabilities that can only be found in an IaaS-hosted data store.</span></span>
- <span data-ttu-id="0bdba-163">**Disponibilité dans les régions** :</span><span class="sxs-lookup"><span data-stu-id="0bdba-163">**Region availability**.</span></span> <span data-ttu-id="0bdba-164">pour les services managés, le service est-il disponible dans toutes les régions Azure ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-164">For managed services, is the service available in all Azure regions?</span></span> <span data-ttu-id="0bdba-165">Votre solution doit-elle être hébergée dans certaines régions Azure ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-165">Does your solution need to be hosted in certain Azure regions?</span></span>
- <span data-ttu-id="0bdba-166">**Portabilité** :</span><span class="sxs-lookup"><span data-stu-id="0bdba-166">**Portability**.</span></span> <span data-ttu-id="0bdba-167">vos données devront-elles être migrées localement, vers des centres de données externes ou vers d’autres environnements d’hébergement cloud ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-167">Will your data need to migrated to on-premises, external datacenters, or other cloud hosting environments?</span></span>
- <span data-ttu-id="0bdba-168">**Licences** :</span><span class="sxs-lookup"><span data-stu-id="0bdba-168">**Licensing**.</span></span> <span data-ttu-id="0bdba-169">avez-vous une préférence quant au type de licence : propriétaire ou OSS ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-169">Do you have a preference of a proprietary versus OSS license type?</span></span> <span data-ttu-id="0bdba-170">Existe-t-il d’autres restrictions externes quant au type de licence que vous pouvez utiliser ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-170">Are there any other external restrictions on what type of license you can use?</span></span>
- <span data-ttu-id="0bdba-171">**Coût global** :</span><span class="sxs-lookup"><span data-stu-id="0bdba-171">**Overall cost**.</span></span> <span data-ttu-id="0bdba-172">quel est le coût global d’utilisation du service dans votre solution ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-172">What is the overall cost of using the service within your solution?</span></span> <span data-ttu-id="0bdba-173">Combien d’instances devront s’exécuter pour répondre à vos besoins en matière de durée de fonctionnement et de débit ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-173">How many instances will need to run, to support your uptime and throughput requirements?</span></span> <span data-ttu-id="0bdba-174">Prenez en compte les coûts d’exploitation dans ce calcul.</span><span class="sxs-lookup"><span data-stu-id="0bdba-174">Consider operations costs in this calculation.</span></span> <span data-ttu-id="0bdba-175">L’un des raisons de préférer les services managés est leur coût d’exploitation réduit.</span><span class="sxs-lookup"><span data-stu-id="0bdba-175">One reason to prefer managed services is the reduced operational cost.</span></span>
- <span data-ttu-id="0bdba-176">**Rentabilité** :</span><span class="sxs-lookup"><span data-stu-id="0bdba-176">**Cost effectiveness**.</span></span> <span data-ttu-id="0bdba-177">pouvez-vous partitionner vos données de façon à profiter d’un stockage plus économique ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-177">Can you partition your data, to store it more cost effectively?</span></span> <span data-ttu-id="0bdba-178">Par exemple, pouvez-vous transférer les objets volumineux d’une base de données relationnelle coûteuse vers une banque d’objets ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-178">For example, can you move large objects out of an expensive relational database into an object store?</span></span>

### <a name="security"></a><span data-ttu-id="0bdba-179">Sécurité</span><span class="sxs-lookup"><span data-stu-id="0bdba-179">Security</span></span>

- <span data-ttu-id="0bdba-180">**Sécurité**.</span><span class="sxs-lookup"><span data-stu-id="0bdba-180">**Security**.</span></span> <span data-ttu-id="0bdba-181">de quel type de chiffrement avez-vous besoin ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-181">What type of encryption do you require?</span></span> <span data-ttu-id="0bdba-182">Avez-vous besoin d’un chiffrement au repos ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-182">Do you need encryption at rest?</span></span> <span data-ttu-id="0bdba-183">Quel mécanisme d’authentification voulez-vous utiliser pour vous connecter à vos données ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-183">What authentication mechanism do you want to use to connect to your data?</span></span>
- <span data-ttu-id="0bdba-184">**Audit** :</span><span class="sxs-lookup"><span data-stu-id="0bdba-184">**Auditing**.</span></span> <span data-ttu-id="0bdba-185">quel type de journal d’audit avez-vous besoin de générer ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-185">What kind of audit log do you need to generate?</span></span>
- <span data-ttu-id="0bdba-186">**Spécifications réseau** :</span><span class="sxs-lookup"><span data-stu-id="0bdba-186">**Networking requirements**.</span></span> <span data-ttu-id="0bdba-187">devez-vous limiter ou gérer l’accès à vos données à partir d’autres ressources réseau ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-187">Do you need to restrict or otherwise manage access to your data from other network resources?</span></span> <span data-ttu-id="0bdba-188">Les données ne doivent-elle être accessibles qu’à l’intérieur de l’environnement Azure ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-188">Does data need to be accessible only from inside the Azure environment?</span></span> <span data-ttu-id="0bdba-189">Les données doivent-elle être accessibles à partir d’adresses IP ou de sous-réseaux spécifiques ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-189">Does the data need to be accessible from specific IP addresses or subnets?</span></span> <span data-ttu-id="0bdba-190">Doivent-elles être accessibles à partir d’applications ou de services hébergés localement ou dans d’autres centres de données externes ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-190">Does it need to be accessible from applications or services hosted on-premises or in other external datacenters?</span></span>

### <a name="devops"></a><span data-ttu-id="0bdba-191">DevOps</span><span class="sxs-lookup"><span data-stu-id="0bdba-191">DevOps</span></span>

- <span data-ttu-id="0bdba-192">**Ensemble de compétences** :</span><span class="sxs-lookup"><span data-stu-id="0bdba-192">**Skill set**.</span></span> <span data-ttu-id="0bdba-193">votre équipe est-elle encline à utiliser certains langages de programmation, systèmes d’exploitation et autres technologies plutôt que d’autres ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-193">Are there particular programming languages, operating systems, or other technology that your team is particularly adept at using?</span></span> <span data-ttu-id="0bdba-194">Votre équipe aurait des difficultés à en utiliser d’autres ?</span><span class="sxs-lookup"><span data-stu-id="0bdba-194">Are there others that would be difficult for your team to work with?</span></span>
- <span data-ttu-id="0bdba-195">**Clients** : existe-t-il un bon support client pour vos langages de développement?</span><span class="sxs-lookup"><span data-stu-id="0bdba-195">**Clients** Is there good client support for your development languages?</span></span>

<span data-ttu-id="0bdba-196">Les sections suivantes comparent les différents modèles de banque de données en termes de profil de charge de travail, de types de données et d’exemples de cas d’usage.</span><span class="sxs-lookup"><span data-stu-id="0bdba-196">The following sections compare various data store models in terms of workload profile, data types, and example use cases.</span></span>

## <a name="relational-database-management-systems-rdbms"></a><span data-ttu-id="0bdba-197">Systèmes de gestion de base de données relationnelle (SGBDR)</span><span class="sxs-lookup"><span data-stu-id="0bdba-197">Relational database management systems (RDBMS)</span></span>

<table>
<tr><td><span data-ttu-id="0bdba-198"><strong>Charge de travail</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-198"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-199">La création d’enregistrements et la mise à jour des données existantes se produisent régulièrement.</span><span class="sxs-lookup"><span data-stu-id="0bdba-199">Both the creation of new records and updates to existing data happen regularly.</span></span></li>
            <li><span data-ttu-id="0bdba-200">Plusieurs opérations doivent être effectuées dans une même transaction.</span><span class="sxs-lookup"><span data-stu-id="0bdba-200">Multiple operations have to be completed in a single transaction.</span></span></li>
            <li><span data-ttu-id="0bdba-201">Nécessite des fonctions d’agrégation pour effectuer une tabulation croisée.</span><span class="sxs-lookup"><span data-stu-id="0bdba-201">Requires aggregation functions to perform cross-tabulation.</span></span></li>
            <li><span data-ttu-id="0bdba-202">Exige une forte intégration avec des outils de création de rapports.</span><span class="sxs-lookup"><span data-stu-id="0bdba-202">Strong integration with reporting tools is required.</span></span></li>
            <li><span data-ttu-id="0bdba-203">Les relations sont mises en œuvre en utilisant des contraintes de base de données.</span><span class="sxs-lookup"><span data-stu-id="0bdba-203">Relationships are enforced using database constraints.</span></span></li>
            <li><span data-ttu-id="0bdba-204">Des index sont utilisés pour optimiser les performances des requêtes.</span><span class="sxs-lookup"><span data-stu-id="0bdba-204">Indexes are used to optimize query performance.</span></span></li>
            <li><span data-ttu-id="0bdba-205">Autorise l’accès à des sous-ensembles spécifiques de données.</span><span class="sxs-lookup"><span data-stu-id="0bdba-205">Allows access to specific subsets of data.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="0bdba-206"><strong>Type de données</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-206"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-207">Les données sont très normalisées.</span><span class="sxs-lookup"><span data-stu-id="0bdba-207">Data is highly normalized.</span></span></li>
            <li><span data-ttu-id="0bdba-208">Des schémas de base de données sont exigés et appliqués.</span><span class="sxs-lookup"><span data-stu-id="0bdba-208">Database schemas are required and enforced.</span></span></li>
            <li><span data-ttu-id="0bdba-209">Relations plusieurs à plusieurs entre les entités de données de la base de données.</span><span class="sxs-lookup"><span data-stu-id="0bdba-209">Many-to-many relationships between data entities in the database.</span></span></li>
            <li><span data-ttu-id="0bdba-210">Des contraintes sont définies dans le schéma et imposées aux données de la base de données.</span><span class="sxs-lookup"><span data-stu-id="0bdba-210">Constraints are defined in the schema and imposed on any data in the database.</span></span></li>
            <li><span data-ttu-id="0bdba-211">Les données nécessitent un haut niveau d’intégrité.</span><span class="sxs-lookup"><span data-stu-id="0bdba-211">Data requires high integrity.</span></span> <span data-ttu-id="0bdba-212">Les index et les relations doivent être gérés avec précision.</span><span class="sxs-lookup"><span data-stu-id="0bdba-212">Indexes and relationships need to be maintained accurately.</span></span></li>
            <li><span data-ttu-id="0bdba-213">Les données nécessitent une cohérence forte.</span><span class="sxs-lookup"><span data-stu-id="0bdba-213">Data requires strong consistency.</span></span> <span data-ttu-id="0bdba-214">Les transactions s’exécutent de façon à assurer une cohérence à 100 % de toutes les données pour l’ensemble des utilisateurs et des processus.</span><span class="sxs-lookup"><span data-stu-id="0bdba-214">Transactions operate in a way that ensures all data are 100% consistent for all users and processes.</span></span></li>
            <li><span data-ttu-id="0bdba-215">Les entrées de données individuelles sont censées être d’une taille petite à moyenne.</span><span class="sxs-lookup"><span data-stu-id="0bdba-215">Size of individual data entries is intended to be small to medium-sized.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="0bdba-216"><strong>Exemples</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-216"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-217">Applications métier (gestion du capital humain, gestion de la relation client, planification des ressources d’entreprise)</span><span class="sxs-lookup"><span data-stu-id="0bdba-217">Line of business  (human capital management, customer relationship management, enterprise resource planning)</span></span></li>
            <li><span data-ttu-id="0bdba-218">Gestion des stocks</span><span class="sxs-lookup"><span data-stu-id="0bdba-218">Inventory management</span></span></li>
            <li><span data-ttu-id="0bdba-219">Base de données de création de rapports</span><span class="sxs-lookup"><span data-stu-id="0bdba-219">Reporting database</span></span></li>
            <li><span data-ttu-id="0bdba-220">Comptabilité</span><span class="sxs-lookup"><span data-stu-id="0bdba-220">Accounting</span></span></li>
            <li><span data-ttu-id="0bdba-221">Gestion des ressources</span><span class="sxs-lookup"><span data-stu-id="0bdba-221">Asset management</span></span></li>
            <li><span data-ttu-id="0bdba-222">Gestion des fonds</span><span class="sxs-lookup"><span data-stu-id="0bdba-222">Fund management</span></span></li>
            <li><span data-ttu-id="0bdba-223">Gestion des commandes</span><span class="sxs-lookup"><span data-stu-id="0bdba-223">Order management</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="document-databases"></a><span data-ttu-id="0bdba-224">Bases de données de documents</span><span class="sxs-lookup"><span data-stu-id="0bdba-224">Document databases</span></span>

<table>
<tr><td><span data-ttu-id="0bdba-225"><strong>Charge de travail</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-225"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-226">Usage général.</span><span class="sxs-lookup"><span data-stu-id="0bdba-226">General purpose.</span></span></li>
            <li><span data-ttu-id="0bdba-227">Les opérations d’insertion et de mise à jour sont courantes.</span><span class="sxs-lookup"><span data-stu-id="0bdba-227">Insert and update operations are common.</span></span> <span data-ttu-id="0bdba-228">La création d’enregistrements et la mise à jour des données existantes se produisent régulièrement.</span><span class="sxs-lookup"><span data-stu-id="0bdba-228">Both the creation of new records and updates to existing data happen regularly.</span></span></li>
            <li><span data-ttu-id="0bdba-229">Pas d’anomalie d’impédance objet-relationnel.</span><span class="sxs-lookup"><span data-stu-id="0bdba-229">No object-relational impedance mismatch.</span></span> <span data-ttu-id="0bdba-230">Les documents peuvent mieux correspondre aux structures d’objet utilisées dans le code d’application.</span><span class="sxs-lookup"><span data-stu-id="0bdba-230">Documents can better match the object structures used in application code.</span></span></li>
            <li><span data-ttu-id="0bdba-231">L’accès concurrentiel optimiste est plus couramment utilisé.</span><span class="sxs-lookup"><span data-stu-id="0bdba-231">Optimistic concurrency is more commonly used.</span></span></li>
            <li><span data-ttu-id="0bdba-232">Les données doivent être modifiées et traitées par l’application consommatrice.</span><span class="sxs-lookup"><span data-stu-id="0bdba-232">Data must be modified and processed by consuming application.</span></span></li>
            <li><span data-ttu-id="0bdba-233">Les données exigent la présence d’un index sur plusieurs champs.</span><span class="sxs-lookup"><span data-stu-id="0bdba-233">Data requires index on multiple fields.</span></span></li>
            <li><span data-ttu-id="0bdba-234">Les documents individuels sont récupérés et écrits en un seul bloc.</span><span class="sxs-lookup"><span data-stu-id="0bdba-234">Individual documents are retrieved and written as a single block.</span></span></li>
    </td>
</tr>
<tr><td><span data-ttu-id="0bdba-235"><strong>Type de données</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-235"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-236">Les données peuvent être gérées de façon dénormalisée.</span><span class="sxs-lookup"><span data-stu-id="0bdba-236">Data can be managed in de-normalized way.</span></span></li>
            <li><span data-ttu-id="0bdba-237">Les données des documents individuels sont relativement peu volumineuses.</span><span class="sxs-lookup"><span data-stu-id="0bdba-237">Size of individual document data is relatively small.</span></span></li>
            <li><span data-ttu-id="0bdba-238">Chaque type de document peut utiliser son propre schéma.</span><span class="sxs-lookup"><span data-stu-id="0bdba-238">Each document type can use its own schema.</span></span></li>
            <li><span data-ttu-id="0bdba-239">Les documents peuvent inclure des champs facultatifs.</span><span class="sxs-lookup"><span data-stu-id="0bdba-239">Documents can include optional fields.</span></span></li>
            <li><span data-ttu-id="0bdba-240">Les données des documents sont semi-structurées, ce qui signifie que les types de données de chaque champ ne sont pas strictement définis.</span><span class="sxs-lookup"><span data-stu-id="0bdba-240">Document data is semi-structured, meaning that data types of each field are not strictly defined.</span></span></li>
            <li><span data-ttu-id="0bdba-241">L’agrégation de données est prise en charge.</span><span class="sxs-lookup"><span data-stu-id="0bdba-241">Data aggregation is supported.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="0bdba-242"><strong>Exemples</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-242"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-243">Catalogue produits</span><span class="sxs-lookup"><span data-stu-id="0bdba-243">Product catalog</span></span></li>
            <li><span data-ttu-id="0bdba-244">Comptes d'utilisateurs</span><span class="sxs-lookup"><span data-stu-id="0bdba-244">User accounts</span></span></li>
            <li><span data-ttu-id="0bdba-245">Nomenclature</span><span class="sxs-lookup"><span data-stu-id="0bdba-245">Bill of materials</span></span></li>
            <li><span data-ttu-id="0bdba-246">Personnalisation</span><span class="sxs-lookup"><span data-stu-id="0bdba-246">Personalization</span></span></li>
            <li><span data-ttu-id="0bdba-247">Gestion de contenu</span><span class="sxs-lookup"><span data-stu-id="0bdba-247">Content management</span></span></li>
            <li><span data-ttu-id="0bdba-248">Données d’exploitation</span><span class="sxs-lookup"><span data-stu-id="0bdba-248">Operations data</span></span></li>
            <li><span data-ttu-id="0bdba-249">Gestion des stocks</span><span class="sxs-lookup"><span data-stu-id="0bdba-249">Inventory management</span></span></li>
            <li><span data-ttu-id="0bdba-250">Données d’historique des transactions</span><span class="sxs-lookup"><span data-stu-id="0bdba-250">Transaction history data</span></span></li>
            <li><span data-ttu-id="0bdba-251">Vue matérialisée d’autres banques NoSQL.</span><span class="sxs-lookup"><span data-stu-id="0bdba-251">Materialized view of other NoSQL stores.</span></span> <span data-ttu-id="0bdba-252">Remplace l’indexation de fichiers/objets blob.</span><span class="sxs-lookup"><span data-stu-id="0bdba-252">Replaces file/BLOB indexing.</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="keyvalue-stores"></a><span data-ttu-id="0bdba-253">Magasins de clés/valeurs</span><span class="sxs-lookup"><span data-stu-id="0bdba-253">Key/value stores</span></span>

<table>
<tr><td><span data-ttu-id="0bdba-254"><strong>Charge de travail</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-254"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-255">L’identification des données et l’accès à celles-ci se fait au moyen d’une clé d’ID unique, comme un dictionnaire.</span><span class="sxs-lookup"><span data-stu-id="0bdba-255">Data is identified and accessed using a single ID key, like a dictionary.</span></span></li>
            <li><span data-ttu-id="0bdba-256">Scalabilité importante.</span><span class="sxs-lookup"><span data-stu-id="0bdba-256">Massively scalable.</span></span></li>
            <li><span data-ttu-id="0bdba-257">Utilisation de jointures, de verrous ou d’unions non obligatoire.</span><span class="sxs-lookup"><span data-stu-id="0bdba-257">No joins, lock, or unions are required.</span></span></li>
            <li><span data-ttu-id="0bdba-258">Aucun mécanisme d’agrégation n’est utilisé.</span><span class="sxs-lookup"><span data-stu-id="0bdba-258">No aggregation mechanisms are used.</span></span></li>
            <li><span data-ttu-id="0bdba-259">Les index secondaires ne sont généralement pas utilisés.</span><span class="sxs-lookup"><span data-stu-id="0bdba-259">Secondary indexes are generally not used.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="0bdba-260"><strong>Type de données</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-260"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-261">Les données ont tendance à être volumineuses.</span><span class="sxs-lookup"><span data-stu-id="0bdba-261">Data size tends to be large.</span></span></li>
            <li><span data-ttu-id="0bdba-262">Chaque clé est associée à une valeur unique, qui est un objet blob de données non managé.</span><span class="sxs-lookup"><span data-stu-id="0bdba-262">Each key is associated with a single value, which is an unmanaged data BLOB.</span></span></li>
            <li><span data-ttu-id="0bdba-263">Absence d’application de schéma.</span><span class="sxs-lookup"><span data-stu-id="0bdba-263">There is no schema enforcement.</span></span></li>
            <li><span data-ttu-id="0bdba-264">Absence de relations entre les entités.</span><span class="sxs-lookup"><span data-stu-id="0bdba-264">No relationships between entities.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="0bdba-265"><strong>Exemples</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-265"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-266">Mise en cache des données</span><span class="sxs-lookup"><span data-stu-id="0bdba-266">Data caching</span></span></li>
            <li><span data-ttu-id="0bdba-267">Gestion des sessions</span><span class="sxs-lookup"><span data-stu-id="0bdba-267">Session management</span></span></li>
            <li><span data-ttu-id="0bdba-268">Gestion des préférences et des profils utilisateur</span><span class="sxs-lookup"><span data-stu-id="0bdba-268">User preference and profile management</span></span></li>
            <li><span data-ttu-id="0bdba-269">Recommandation de produits et affichage d’annonces</span><span class="sxs-lookup"><span data-stu-id="0bdba-269">Product recommendation and ad serving</span></span></li>
            <li><span data-ttu-id="0bdba-270">Dictionnaires</span><span class="sxs-lookup"><span data-stu-id="0bdba-270">Dictionaries</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="graph-databases"></a><span data-ttu-id="0bdba-271">Bases de données de graphiques</span><span class="sxs-lookup"><span data-stu-id="0bdba-271">Graph databases</span></span>

<table>
<tr><td><span data-ttu-id="0bdba-272"><strong>Charge de travail</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-272"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-273">Les relations entre les éléments de données sont très complexes, impliquant de nombreux sauts entre les éléments de données associés.</span><span class="sxs-lookup"><span data-stu-id="0bdba-273">The relationships between data items are very complex, involving many hops between related data items.</span></span></li>
            <li><span data-ttu-id="0bdba-274">Les relations entre les éléments de données sont dynamiques et changent dans le temps.</span><span class="sxs-lookup"><span data-stu-id="0bdba-274">The relationship between data items are dynamic and change over time.</span></span></li>
            <li><span data-ttu-id="0bdba-275">Les relations entre les objets sont des entités de première classe, sans qu’il soit nécessaire de traverser des clés étrangères et des jointures.</span><span class="sxs-lookup"><span data-stu-id="0bdba-275">Relationships between objects are first-class citizens, without requiring foreign-keys and joins to traverse.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="0bdba-276"><strong>Type de données</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-276"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-277">Les données sont constituées de nœuds et de relations.</span><span class="sxs-lookup"><span data-stu-id="0bdba-277">Data is comprised of nodes and relationships.</span></span></li>
            <li><span data-ttu-id="0bdba-278">Les nœuds s’apparentent à des lignes de table ou à des documents JSON.</span><span class="sxs-lookup"><span data-stu-id="0bdba-278">Nodes are similar to table rows or JSON documents.</span></span></li>
            <li><span data-ttu-id="0bdba-279">Les relations sont tout aussi importantes que les nœuds et sont directement exposées dans le langage de requête.</span><span class="sxs-lookup"><span data-stu-id="0bdba-279">Relationships are just as important as nodes, and are exposed directly in the query language.</span></span></li>
            <li><span data-ttu-id="0bdba-280">Les objets composites, tels qu’une personne ayant plusieurs numéros de téléphone, ont tendance à être divisés en nœuds distincts plus petits, en association avec des relations qui peuvent être traversées.</span><span class="sxs-lookup"><span data-stu-id="0bdba-280">Composite objects, such as a person with multiple phone numbers, tend to be broken into separate, smaller nodes, combined with traversable relationships</span></span> </li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="0bdba-281"><strong>Exemples</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-281"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-282">Organigrammes</span><span class="sxs-lookup"><span data-stu-id="0bdba-282">Organization charts</span></span></li>
            <li><span data-ttu-id="0bdba-283">Graphiques de réseau sociaux</span><span class="sxs-lookup"><span data-stu-id="0bdba-283">Social graphs</span></span></li>
            <li><span data-ttu-id="0bdba-284">Détection des fraudes</span><span class="sxs-lookup"><span data-stu-id="0bdba-284">Fraud detection</span></span></li>
            <li><span data-ttu-id="0bdba-285">Analytics</span><span class="sxs-lookup"><span data-stu-id="0bdba-285">Analytics</span></span></li>
            <li><span data-ttu-id="0bdba-286">Moteurs de recommandation</span><span class="sxs-lookup"><span data-stu-id="0bdba-286">Recommendation engines</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="column-family-databases"></a><span data-ttu-id="0bdba-287">Bases de données de familles de colonnes</span><span class="sxs-lookup"><span data-stu-id="0bdba-287">Column-family databases</span></span>

<table>
<tr><td><span data-ttu-id="0bdba-288"><strong>Charge de travail</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-288"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-289">La plupart des bases de données de familles de colonnes effectuent les opérations d’écriture de manière extrêmement rapide.</span><span class="sxs-lookup"><span data-stu-id="0bdba-289">Most column-family databases perform write operations extremely quickly.</span></span></li>
            <li><span data-ttu-id="0bdba-290">Les opérations de mise à jour et de suppression sont rares.</span><span class="sxs-lookup"><span data-stu-id="0bdba-290">Update and delete operations are rare.</span></span></li>
            <li><span data-ttu-id="0bdba-291">Conçu pour offrir un débit élevé et un accès à faible latence.</span><span class="sxs-lookup"><span data-stu-id="0bdba-291">Designed to provide high throughput and low-latency access.</span></span></li>
            <li><span data-ttu-id="0bdba-292">Permet aux requêtes d’accéder facilement à un ensemble particulier de champs au sein d’un enregistrement bien plus volumineux.</span><span class="sxs-lookup"><span data-stu-id="0bdba-292">Supports easy query access to a particular set of fields within a much larger record.</span></span></li>
            <li><span data-ttu-id="0bdba-293">Scalabilité importante.</span><span class="sxs-lookup"><span data-stu-id="0bdba-293">Massively scalable.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="0bdba-294"><strong>Type de données</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-294"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-295">Les données sont stockées dans des tables comprenant une colonne clé et une ou plusieurs familles de colonnes.</span><span class="sxs-lookup"><span data-stu-id="0bdba-295">Data is stored in tables consisting of a key column and one or more column families.</span></span></li>
            <li><span data-ttu-id="0bdba-296">Certaines colonnes peuvent varier par lignes individuelles.</span><span class="sxs-lookup"><span data-stu-id="0bdba-296">Specific columns can vary by individual rows.</span></span></li>
            <li><span data-ttu-id="0bdba-297">Les cellules individuelles sont accessibles via des commandes get et put.</span><span class="sxs-lookup"><span data-stu-id="0bdba-297">Individual cells are accessed via get and put commands</span></span></li>
            <li><span data-ttu-id="0bdba-298">Plusieurs lignes sont retournées en utilisant une commande scan.</span><span class="sxs-lookup"><span data-stu-id="0bdba-298">Multiple rows are returned using a scan command.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="0bdba-299"><strong>Exemples</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-299"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-300">Recommandations</span><span class="sxs-lookup"><span data-stu-id="0bdba-300">Recommendations</span></span></li>
            <li><span data-ttu-id="0bdba-301">Personnalisation</span><span class="sxs-lookup"><span data-stu-id="0bdba-301">Personalization</span></span></li>
            <li><span data-ttu-id="0bdba-302">données de capteur</span><span class="sxs-lookup"><span data-stu-id="0bdba-302">Sensor data</span></span></li>
            <li><span data-ttu-id="0bdba-303">Télémétrie</span><span class="sxs-lookup"><span data-stu-id="0bdba-303">Telemetry</span></span></li>
            <li><span data-ttu-id="0bdba-304">Messagerie</span><span class="sxs-lookup"><span data-stu-id="0bdba-304">Messaging</span></span></li>
            <li><span data-ttu-id="0bdba-305">Analytique des réseaux sociaux</span><span class="sxs-lookup"><span data-stu-id="0bdba-305">Social media analytics</span></span></li>
            <li><span data-ttu-id="0bdba-306">Analytique web</span><span class="sxs-lookup"><span data-stu-id="0bdba-306">Web analytics</span></span></li>
            <li><span data-ttu-id="0bdba-307">Surveillance de l’activité</span><span class="sxs-lookup"><span data-stu-id="0bdba-307">Activity monitoring</span></span></li>
            <li><span data-ttu-id="0bdba-308">Météo et autres données de séries chronologiques</span><span class="sxs-lookup"><span data-stu-id="0bdba-308">Weather and other time-series data</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="search-engine-databases"></a><span data-ttu-id="0bdba-309">Bases de données de moteur de recherche</span><span class="sxs-lookup"><span data-stu-id="0bdba-309">Search engine databases</span></span>

<table>
<tr><td><span data-ttu-id="0bdba-310"><strong>Charge de travail</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-310"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-311">Indexation de données de plusieurs sources et services.</span><span class="sxs-lookup"><span data-stu-id="0bdba-311">Indexing data from multiple sources and services.</span></span></li>
            <li><span data-ttu-id="0bdba-312">Les requêtes sont de type ad hoc et peuvent être complexes.</span><span class="sxs-lookup"><span data-stu-id="0bdba-312">Queries are ad-hoc and can be complex.</span></span></li>
            <li><span data-ttu-id="0bdba-313">Nécessite une agrégation.</span><span class="sxs-lookup"><span data-stu-id="0bdba-313">Requires aggregation.</span></span></li>
            <li><span data-ttu-id="0bdba-314">La recherche en texte intégral est obligatoire.</span><span class="sxs-lookup"><span data-stu-id="0bdba-314">Full text search is required.</span></span></li>
            <li><span data-ttu-id="0bdba-315">Nécessite une requête en libre service ad hoc.</span><span class="sxs-lookup"><span data-stu-id="0bdba-315">Ad hoc self-service query is required.</span></span></li>
            <li><span data-ttu-id="0bdba-316">Nécessite une analyse des données avec un index sur tous les champs.</span><span class="sxs-lookup"><span data-stu-id="0bdba-316">Data analysis with index on all fields is required.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="0bdba-317"><strong>Type de données</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-317"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-318">Semi-structurées ou non structurées</span><span class="sxs-lookup"><span data-stu-id="0bdba-318">Semi-structured or unstructured</span></span></li>
            <li><span data-ttu-id="0bdba-319">Texte</span><span class="sxs-lookup"><span data-stu-id="0bdba-319">Text</span></span></li>
            <li><span data-ttu-id="0bdba-320">Texte avec référence à des données structurées</span><span class="sxs-lookup"><span data-stu-id="0bdba-320">Text with reference to structured data</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="0bdba-321"><strong>Exemples</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-321"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-322">Catalogue produits</span><span class="sxs-lookup"><span data-stu-id="0bdba-322">Product catalogs</span></span></li>
            <li><span data-ttu-id="0bdba-323">Recherche sur site</span><span class="sxs-lookup"><span data-stu-id="0bdba-323">Site search</span></span></li>
            <li><span data-ttu-id="0bdba-324">Journalisation</span><span class="sxs-lookup"><span data-stu-id="0bdba-324">Logging</span></span></li>
            <li><span data-ttu-id="0bdba-325">Analytics</span><span class="sxs-lookup"><span data-stu-id="0bdba-325">Analytics</span></span></li>
            <li><span data-ttu-id="0bdba-326">Sites marchands</span><span class="sxs-lookup"><span data-stu-id="0bdba-326">Shopping sites</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="data-warehouse"></a><span data-ttu-id="0bdba-327">Entrepôt de données</span><span class="sxs-lookup"><span data-stu-id="0bdba-327">Data warehouse</span></span>

<table>
<tr><td><span data-ttu-id="0bdba-328"><strong>Charge de travail</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-328"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-329">Analyse de données</span><span class="sxs-lookup"><span data-stu-id="0bdba-329">Data analytics</span></span></li>
            <li><span data-ttu-id="0bdba-330">Décisionnel d’entreprise</span><span class="sxs-lookup"><span data-stu-id="0bdba-330">Enterprise BI</span></span>   </li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="0bdba-331"><strong>Type de données</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-331"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-332">Données d’historique de plusieurs sources.</span><span class="sxs-lookup"><span data-stu-id="0bdba-332">Historical data from multiple sources.</span></span></li>
            <li><span data-ttu-id="0bdba-333">Généralement dénormalisé dans un schéma &quot;en étoile&quot; ou &quot;en flocon&quot;, constitué de tables de faits et de dimensions.</span><span class="sxs-lookup"><span data-stu-id="0bdba-333">Usually denormalized in a &quot;star&quot; or &quot;snowflake&quot; schema, consisting of fact and dimension tables.</span></span></li>
            <li><span data-ttu-id="0bdba-334">Généralement chargé de façon régulière avec de nouvelles données.</span><span class="sxs-lookup"><span data-stu-id="0bdba-334">Usually loaded with new data on a scheduled basis.</span></span></li>
            <li><span data-ttu-id="0bdba-335">Les tables de dimension comprennent souvent plusieurs versions historiques d’une entité, appelée <em>dimension à variation lente</em>.</span><span class="sxs-lookup"><span data-stu-id="0bdba-335">Dimension tables often include multiple historic versions of an entity, referred to as a <em>slowly changing dimension</em>.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="0bdba-336"><strong>Exemples</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-336"><strong>Examples</strong></span></span></td>
    <td><span data-ttu-id="0bdba-337">Entrepôt de données d’entreprise fournissant des données pour des modèles, rapports et tableaux de bord analytiques.</span><span class="sxs-lookup"><span data-stu-id="0bdba-337">An enterprise data warehouse that provides data for analytical models, reports, and dashboards.</span></span>
    </td>
</tr>
</table>


## <a name="time-series-databases"></a><span data-ttu-id="0bdba-338">Bases de données de séries chronologiques</span><span class="sxs-lookup"><span data-stu-id="0bdba-338">Time series databases</span></span>

<table>
<tr><td><span data-ttu-id="0bdba-339"><strong>Charge de travail</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-339"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-340">Les opérations sont dans une très large proportion (95-99 %) des écritures.</span><span class="sxs-lookup"><span data-stu-id="0bdba-340">An overwhelming proportion of operations (95-99%) are writes.</span></span></li>
            <li><span data-ttu-id="0bdba-341">Les enregistrements sont généralement ajoutés de manière séquentielle dans un ordre chronologique.</span><span class="sxs-lookup"><span data-stu-id="0bdba-341">Records are generally appended sequentially in time order.</span></span></li>
            <li><span data-ttu-id="0bdba-342">Les mises à jour sont rares.</span><span class="sxs-lookup"><span data-stu-id="0bdba-342">Updates are rare.</span></span></li>
            <li><span data-ttu-id="0bdba-343">Les suppressions se produisent en masse et concernent des blocs ou des enregistrements contigus.</span><span class="sxs-lookup"><span data-stu-id="0bdba-343">Deletes occur in bulk, and are made to contiguous blocks or records.</span></span></li>
            <li><span data-ttu-id="0bdba-344">La taille des demandes de lecture peut dépasser la taille de mémoire disponible.</span><span class="sxs-lookup"><span data-stu-id="0bdba-344">Read requests can be larger than available memory.</span></span></li>
            <li><span data-ttu-id="0bdba-345">Il est courant que plusieurs lectures se produisent simultanément.</span><span class="sxs-lookup"><span data-stu-id="0bdba-345">It&#39;s common for multiple reads to occur simultaneously.</span></span></li>
            <li><span data-ttu-id="0bdba-346">Les données sont lues de manière séquentielle dans un ordre chronologique croissant ou décroissant.</span><span class="sxs-lookup"><span data-stu-id="0bdba-346">Data is read sequentially in either ascending or descending time order.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="0bdba-347"><strong>Type de données</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-347"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-348">Horodatage utilisé comme clé primaire et mécanisme de tri.</span><span class="sxs-lookup"><span data-stu-id="0bdba-348">A time stamp that is used as the primary key and sorting mechanism.</span></span></li>
            <li><span data-ttu-id="0bdba-349">Mesures de l’entrée ou descriptions de ce que l’entrée représente.</span><span class="sxs-lookup"><span data-stu-id="0bdba-349">Measurements from the entry or descriptions of what the entry represents.</span></span></li>
            <li><span data-ttu-id="0bdba-350">Balises qui définissent des informations supplémentaires sur le type, l’origine et d’autres informations sur l’entrée.</span><span class="sxs-lookup"><span data-stu-id="0bdba-350">Tags that define additional information about the type, origin, and other information about the entry.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="0bdba-351"><strong>Exemples</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-351"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-352">Surveillance et données de télémétrie des événements.</span><span class="sxs-lookup"><span data-stu-id="0bdba-352">Monitoring and event telemetry.</span></span></li>
            <li><span data-ttu-id="0bdba-353">Données de capteur ou autres données IoT.</span><span class="sxs-lookup"><span data-stu-id="0bdba-353">Sensor or other IoT data.</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="object-storage"></a><span data-ttu-id="0bdba-354">Stockage d’objets</span><span class="sxs-lookup"><span data-stu-id="0bdba-354">Object storage</span></span>

<table>
<tr><td><span data-ttu-id="0bdba-355"><strong>Charge de travail</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-355"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-356">Identifié par la clé.</span><span class="sxs-lookup"><span data-stu-id="0bdba-356">Identified by key.</span></span></li>
            <li><span data-ttu-id="0bdba-357">Les objets peuvent offrir un accès public ou privé.</span><span class="sxs-lookup"><span data-stu-id="0bdba-357">Objects may be publicly or privately accessible.</span></span></li>
            <li><span data-ttu-id="0bdba-358">Le contenu est généralement une ressource, par exemple une feuille de calcul, une image ou un fichier vidéo.</span><span class="sxs-lookup"><span data-stu-id="0bdba-358">Content is typically an asset such as a spreadsheet, image, or video file.</span></span></li>
            <li><span data-ttu-id="0bdba-359">Le contenu doit être durable (persistant) et extérieur à une couche Application ou à une machine virtuelle.</span><span class="sxs-lookup"><span data-stu-id="0bdba-359">Content must be durable (persistent), and external to any application tier or virtual machine.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="0bdba-360"><strong>Type de données</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-360"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-361">Données volumineuses.</span><span class="sxs-lookup"><span data-stu-id="0bdba-361">Data size is large.</span></span></li>
            <li><span data-ttu-id="0bdba-362">Données blob.</span><span class="sxs-lookup"><span data-stu-id="0bdba-362">Blob data.</span></span></li>
            <li><span data-ttu-id="0bdba-363">Valeur opaque.</span><span class="sxs-lookup"><span data-stu-id="0bdba-363">Value is opaque.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="0bdba-364"><strong>Exemples</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-364"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-365">Images, vidéos, documents Office, PDF</span><span class="sxs-lookup"><span data-stu-id="0bdba-365">Images, videos, office documents, PDFs</span></span></li>
            <li><span data-ttu-id="0bdba-366">CSS, scripts, CSV</span><span class="sxs-lookup"><span data-stu-id="0bdba-366">CSS, Scripts, CSV</span></span></li>
            <li><span data-ttu-id="0bdba-367">HTML statique, JSON</span><span class="sxs-lookup"><span data-stu-id="0bdba-367">Static HTML, JSON</span></span></li>
            <li><span data-ttu-id="0bdba-368">Fichiers journaux et d’audit</span><span class="sxs-lookup"><span data-stu-id="0bdba-368">Log and audit files</span></span></li>
            <li><span data-ttu-id="0bdba-369">Sauvegardes de base de données</span><span class="sxs-lookup"><span data-stu-id="0bdba-369">Database backups</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="shared-files"></a><span data-ttu-id="0bdba-370">Fichiers partagés</span><span class="sxs-lookup"><span data-stu-id="0bdba-370">Shared files</span></span>

<table>
<tr><td><span data-ttu-id="0bdba-371"><strong>Charge de travail</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-371"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-372">Migration à partir d’applications existantes qui interagissent avec le système de fichiers.</span><span class="sxs-lookup"><span data-stu-id="0bdba-372">Migration from existing apps that interact with the file system.</span></span></li>
            <li><span data-ttu-id="0bdba-373">Nécessite une interface SMB.</span><span class="sxs-lookup"><span data-stu-id="0bdba-373">Requires SMB interface.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="0bdba-374"><strong>Type de données</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-374"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-375">Fichiers figurant dans un ensemble hiérarchique de dossiers.</span><span class="sxs-lookup"><span data-stu-id="0bdba-375">Files in a hierarchical set of folders.</span></span></li>
            <li><span data-ttu-id="0bdba-376">Accessible avec des bibliothèques d’E/S standard.</span><span class="sxs-lookup"><span data-stu-id="0bdba-376">Accessible with standard I/O libraries.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="0bdba-377"><strong>Exemples</strong></span><span class="sxs-lookup"><span data-stu-id="0bdba-377"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="0bdba-378">Fichiers hérités</span><span class="sxs-lookup"><span data-stu-id="0bdba-378">Legacy files</span></span></li>
            <li><span data-ttu-id="0bdba-379">Contenu partagé accessible à un certain nombre de machines virtuelles ou d’instances d’application</span><span class="sxs-lookup"><span data-stu-id="0bdba-379">Shared content accessible among a number of VMs or app instances</span></span></li>
        </ul>
    </td>
</tr>
</table>
