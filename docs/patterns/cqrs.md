---
title: CQRS
description: "Séparez les opérations qui lisent les données des opérations qui mettent à jour les données en utilisant des interfaces distinctes."
keywords: "modèle de conception"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- design-implementation
- performance-scalability
ms.openlocfilehash: 80f4a8880cf2212acf82dadb67b0181e1cbae099
ms.sourcegitcommit: a7aae13569e165d4e768ce0aaaac154ba612934f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/30/2018
---
# <a name="command-and-query-responsibility-segregation-cqrs-pattern"></a><span data-ttu-id="e9f01-104">Modèle de séparation des responsabilités en matière de commande et de requête (CQRS)</span><span class="sxs-lookup"><span data-stu-id="e9f01-104">Command and Query Responsibility Segregation (CQRS) pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="e9f01-105">Séparez les opérations qui lisent les données des opérations qui mettent à jour les données en utilisant des interfaces distinctes.</span><span class="sxs-lookup"><span data-stu-id="e9f01-105">Segregate operations that read data from operations that update data by using separate interfaces.</span></span> <span data-ttu-id="e9f01-106">Cela permet d’optimiser les performances, l’extensibilité et la sécurité.</span><span class="sxs-lookup"><span data-stu-id="e9f01-106">This can maximize performance, scalability, and security.</span></span> <span data-ttu-id="e9f01-107">Ce modèle prend en charge l’évolution du système au fil du temps via davantage de flexibilité et empêche les commandes de mise à jour d’introduire des conflits de fusion au niveau du domaine.</span><span class="sxs-lookup"><span data-stu-id="e9f01-107">Supports the evolution of the system over time through higher flexibility, and prevent update commands from causing merge conflicts at the domain level.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="e9f01-108">Contexte et problème</span><span class="sxs-lookup"><span data-stu-id="e9f01-108">Context and problem</span></span>

<span data-ttu-id="e9f01-109">Dans les systèmes de gestion de données traditionnels, les commandes (mises à jour des données) et les requêtes (demandes de données) sont exécutées par rapport au même ensemble d’entités dans un répertoire de données unique.</span><span class="sxs-lookup"><span data-stu-id="e9f01-109">In traditional data management systems, both commands (updates to the data) and queries (requests for data) are executed against the same set of entities in a single data repository.</span></span> <span data-ttu-id="e9f01-110">Ces entités peuvent être un sous-ensemble de lignes dans une ou plusieurs tables d’une base de données relationnelle telle que SQL Server.</span><span class="sxs-lookup"><span data-stu-id="e9f01-110">These entities can be a subset of the rows in one or more tables in a relational database such as SQL Server.</span></span>

<span data-ttu-id="e9f01-111">En général, dans ces systèmes, toutes les opérations de création, de lecture, de mise à jour et de suppression (CRUD) sont appliquées à la même représentation de l’entité.</span><span class="sxs-lookup"><span data-stu-id="e9f01-111">Typically in these systems, all create, read, update, and delete (CRUD) operations are applied to the same representation of the entity.</span></span> <span data-ttu-id="e9f01-112">Par exemple, un objet de transfert de données (DTO) représentant un client est récupéré dans le magasin de données par la couche d’accès aux données (DAL) et affiché à l’écran.</span><span class="sxs-lookup"><span data-stu-id="e9f01-112">For example, a data transfer object (DTO) representing a customer is retrieved from the data store by the data access layer (DAL) and displayed on the screen.</span></span> <span data-ttu-id="e9f01-113">Un utilisateur met à jour certains champs du DTO (par exemple, via la liaison de données) et le DTO est ensuite enregistré dans le magasin de données par la couche DAL.</span><span class="sxs-lookup"><span data-stu-id="e9f01-113">A user updates some fields of the DTO (perhaps through data binding) and the DTO is then saved back in the data store by the DAL.</span></span> <span data-ttu-id="e9f01-114">Le même DTO est utilisé pour les opérations de lecture et d’écriture.</span><span class="sxs-lookup"><span data-stu-id="e9f01-114">The same DTO is used for both the read and write operations.</span></span> <span data-ttu-id="e9f01-115">La figure illustre une architecture CRUD traditionnelle.</span><span class="sxs-lookup"><span data-stu-id="e9f01-115">The figure illustrates a traditional CRUD architecture.</span></span>

![Une architecture CRUD traditionnelle](./_images/command-and-query-responsibility-segregation-cqrs-tradition-crud.png)

<span data-ttu-id="e9f01-117">Les conceptions CRUD traditionnelles fonctionnent bien quand une logique métier limitée est appliquée aux opérations de données uniquement.</span><span class="sxs-lookup"><span data-stu-id="e9f01-117">Traditional CRUD designs work well when only limited business logic is applied to the data operations.</span></span> <span data-ttu-id="e9f01-118">Les mécanismes structurels fournis par les outils de développement permettent de créer très rapidement un code d’accès aux données, que vous pouvez ensuite personnaliser à votre convenance.</span><span class="sxs-lookup"><span data-stu-id="e9f01-118">Scaffold mechanisms provided by development tools can create data access code very quickly, which can then be customized as required.</span></span>

<span data-ttu-id="e9f01-119">Toutefois, l’approche CRUD traditionnelle présente certains inconvénients :</span><span class="sxs-lookup"><span data-stu-id="e9f01-119">However, the traditional CRUD approach has some disadvantages:</span></span>

- <span data-ttu-id="e9f01-120">Cela signifie souvent qu’il existe une incompatibilité entre les représentations d’écriture et de lecture des données, telles que des colonnes ou propriétés supplémentaires qui doivent être mises à jour correctement même si elles ne sont pas requises dans le cadre d’une opération.</span><span class="sxs-lookup"><span data-stu-id="e9f01-120">It often means that there's a mismatch between the read and write representations of the data, such as additional columns or properties that must be updated correctly even though they aren't required as part of an operation.</span></span>

- <span data-ttu-id="e9f01-121">Il y a des risques de contention des données lorsque les enregistrements sont verrouillés dans le magasin de données d’un domaine collaboratif où plusieurs acteurs travaillent en parallèle sur le même ensemble de données.</span><span class="sxs-lookup"><span data-stu-id="e9f01-121">It risks data contention when records are locked in the data store in a collaborative domain, where multiple actors operate in parallel on the same set of data.</span></span> <span data-ttu-id="e9f01-122">Ou vous pouvez rencontrer des conflits de mise à jour causés par des mises à jour simultanées quand le verrouillage optimiste est utilisé.</span><span class="sxs-lookup"><span data-stu-id="e9f01-122">Or update conflicts caused by concurrent updates when optimistic locking is used.</span></span> <span data-ttu-id="e9f01-123">Ces risques augmentent à mesure que la complexité et le débit du système évoluent.</span><span class="sxs-lookup"><span data-stu-id="e9f01-123">These risks increase as the complexity and throughput of the system grows.</span></span> <span data-ttu-id="e9f01-124">En outre, l’approche traditionnelle peut avoir des conséquences négatives sur les performances en raison de la charge sur la couche d’accès aux données et le magasin de données, et la complexité des requêtes nécessaire pour récupérer des informations.</span><span class="sxs-lookup"><span data-stu-id="e9f01-124">In addition, the traditional approach can have a negative effect on performance due to load on the data store and data access layer, and the complexity of queries required to retrieve information.</span></span>

- <span data-ttu-id="e9f01-125">Elle peut compliquer la gestion de la sécurité et des autorisations, car chaque entité est soumise à des opérations de lecture et d’écriture qui peuvent exposer des données dans le mauvais contexte.</span><span class="sxs-lookup"><span data-stu-id="e9f01-125">It can make managing security and permissions more complex because each entity is subject to both read and write operations, which might expose data in the wrong context.</span></span>

> <span data-ttu-id="e9f01-126">Pour une meilleure compréhension des limites de l’approche CRUD, consultez [CRUD, Only When You Can Afford It](https://blogs.msdn.microsoft.com/maarten_mullender/2004/07/23/crud-only-when-you-can-afford-it-revisited/) (L’approche CRUD uniquement lorsque vous pouvez vous le permettre).</span><span class="sxs-lookup"><span data-stu-id="e9f01-126">For a deeper understanding of the limits of the CRUD approach see [CRUD, Only When You Can Afford It](https://blogs.msdn.microsoft.com/maarten_mullender/2004/07/23/crud-only-when-you-can-afford-it-revisited/).</span></span>

## <a name="solution"></a><span data-ttu-id="e9f01-127">Solution</span><span class="sxs-lookup"><span data-stu-id="e9f01-127">Solution</span></span>

<span data-ttu-id="e9f01-128">Séparation des responsabilités en matière de commande et de requête (CQRS) est un modèle qui sépare les opérations qui lisent les données (requêtes) des opérations qui mettent à jour les données (commandes) en utilisant des interfaces distinctes.</span><span class="sxs-lookup"><span data-stu-id="e9f01-128">Command and Query Responsibility Segregation (CQRS) is a pattern that segregates the operations that read data (queries) from the operations that update data (commands) by using separate interfaces.</span></span> <span data-ttu-id="e9f01-129">Cela signifie que les modèles de données utilisés pour les requêtes et les mises à jour sont différents.</span><span class="sxs-lookup"><span data-stu-id="e9f01-129">This means that the data models used for querying and updates are different.</span></span> <span data-ttu-id="e9f01-130">Les modèles peuvent ensuite être isolés, comme indiqué dans la figure suivante, même si cela n’est pas une exigence absolue.</span><span class="sxs-lookup"><span data-stu-id="e9f01-130">The models can then be isolated, as shown in the following figure, although that's not an absolute requirement.</span></span>

![Une architecture CQRS de base](./_images/command-and-query-responsibility-segregation-cqrs-basic.png)

<span data-ttu-id="e9f01-132">En comparaison avec le modèle de données unique utilisé dans les systèmes basés sur l’approche CRUD, l’utilisation des modèles de requête et de mise à jour séparées pour les données des systèmes basés sur l’approche CQRS simplifie la conception et l’implémentation.</span><span class="sxs-lookup"><span data-stu-id="e9f01-132">Compared to the single data model used in CRUD-based systems, the use of separate query and update models for the data in CQRS-based systems simplifies design and implementation.</span></span> <span data-ttu-id="e9f01-133">Toutefois, à la différence des conceptions CRUD, le code CQRS ne peut pas être généré automatiquement à l’aide des mécanismes structurels.</span><span class="sxs-lookup"><span data-stu-id="e9f01-133">However, one disadvantage is that unlike CRUD designs, CQRS code can't automatically be generated using scaffold mechanisms.</span></span>

<span data-ttu-id="e9f01-134">Le modèle de requête pour la lecture des données et le modèle de mise à jour pour l’écriture de données peuvent accéder au même magasin physique, peut-être à l’aide de vues SQL ou en générant des projections à la volée.</span><span class="sxs-lookup"><span data-stu-id="e9f01-134">The query model for reading data and the update model for writing data can access the same physical store, perhaps by using SQL views or by generating projections on the fly.</span></span> <span data-ttu-id="e9f01-135">Cependant, il est courant pour séparer les données dans des magasins physiques différents pour optimiser les performances, l’extensibilité et la sécurité, comme indiqué dans la figure suivante.</span><span class="sxs-lookup"><span data-stu-id="e9f01-135">However, it's common to separate the data into different physical stores to maximize performance, scalability, and security, as shown in the next figure.</span></span>

![Une architecture CQRS avec des magasins d’écriture et de lecture séparés](./_images/command-and-query-responsibility-segregation-cqrs-separate-stores.png)

<span data-ttu-id="e9f01-137">Le magasin de lecture peut être un réplica en lecture seule du magasin d’écriture, ou les magasins de lecture et d’écriture peuvent avoir une structure complètement différente.</span><span class="sxs-lookup"><span data-stu-id="e9f01-137">The read store can be a read-only replica of the write store, or the read and write stores can have a different structure altogether.</span></span> <span data-ttu-id="e9f01-138">Utiliser plusieurs réplicas en lecture seule du magasin de lecture peut considérablement accroître les performances des requêtes et la réactivité de l’interface utilisateur de l’application, en particulier dans les scénarios distribués où les réplicas en lecture seule se trouvent à proximité des instances d’application.</span><span class="sxs-lookup"><span data-stu-id="e9f01-138">Using multiple read-only replicas of the read store can greatly increase query performance and application UI responsiveness, especially in distributed scenarios where read-only replicas are located close to the application instances.</span></span> <span data-ttu-id="e9f01-139">Certains systèmes de base de données (SQL Server) fournissent des fonctionnalités supplémentaires telles que le basculement des réplicas pour optimiser la disponibilité.</span><span class="sxs-lookup"><span data-stu-id="e9f01-139">Some database systems (SQL Server) provide additional features such as failover replicas to maximize availability.</span></span>

<span data-ttu-id="e9f01-140">La séparation des magasins de lecture et d’écriture permet également de les mettre à l’échelle de façon individuelle en fonction de la charge.</span><span class="sxs-lookup"><span data-stu-id="e9f01-140">Separation of the read and write stores also allows each to be scaled appropriately to match the load.</span></span> <span data-ttu-id="e9f01-141">Par exemple, les magasins de lecture sont généralement confrontés à une charge plus importante que les magasins d’écriture.</span><span class="sxs-lookup"><span data-stu-id="e9f01-141">For example, read stores typically encounter a much higher load than write stores.</span></span>

<span data-ttu-id="e9f01-142">Lorsque le modèle de requête/lecture contient des données dénormalisées (consultez [Materialized View pattern](materialized-view.md) (Modèle de vue matérialisée)), les performances sont optimisées lors de la lecture des données de chacune des vues d’une application ou lors de l’interrogation de données dans le système.</span><span class="sxs-lookup"><span data-stu-id="e9f01-142">When the query/read model contains denormalized data (see [Materialized View pattern](materialized-view.md)), performance is maximized when reading data for each of the views in an application or when querying the data in the system.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="e9f01-143">Problèmes et considérations</span><span class="sxs-lookup"><span data-stu-id="e9f01-143">Issues and considerations</span></span>

<span data-ttu-id="e9f01-144">Prenez en compte les points suivants lorsque vous choisissez comment implémenter ce modèle :</span><span class="sxs-lookup"><span data-stu-id="e9f01-144">Consider the following points when deciding how to implement this pattern:</span></span>

- <span data-ttu-id="e9f01-145">Diviser le magasin de données en magasins physiques distincts pour les opérations de lecture et d’écriture permet d’accroître les performances et la sécurité d’un système, mais peut également apporter de la complexité en termes de résilience et de cohérence éventuelle.</span><span class="sxs-lookup"><span data-stu-id="e9f01-145">Dividing the data store into separate physical stores for read and write operations can increase the performance and security of a system, but it can add complexity in terms of resiliency and eventual consistency.</span></span> <span data-ttu-id="e9f01-146">Le magasin de modèle de lecture peut être mis à jour pour refléter les modifications du magasin de modèle d’écriture, et il peut être difficile de détecter le moment où un utilisateur a émis une requête basée sur des données de lecture périmées, ce qui signifie que l’opération ne peut pas se terminer.</span><span class="sxs-lookup"><span data-stu-id="e9f01-146">The read model store must be updated to reflect changes to the write model store, and it can be difficult to detect when a user has issued a request based on stale read data, which means that the operation can't be completed.</span></span>

    > <span data-ttu-id="e9f01-147">Pour obtenir une description de la cohérence éventuelle, consultez [Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx) (Manuel d’introduction à la cohérence des données).</span><span class="sxs-lookup"><span data-stu-id="e9f01-147">For a description of eventual consistency see the [Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx).</span></span>

- <span data-ttu-id="e9f01-148">Envisagez d’appliquer l’approche CQRS à des sections limitées de votre système, là où elle sera le plus utile.</span><span class="sxs-lookup"><span data-stu-id="e9f01-148">Consider applying CQRS to limited sections of your system where it will be most valuable.</span></span>

- <span data-ttu-id="e9f01-149">Une approche courante pour le déploiement de la cohérence éventuelle consiste à utiliser l’approvisionnement en événements avec le modèle CQRS, afin que le modèle d’écriture soit un flux d’événements en mode ajouter uniquement piloté par l’exécution de commandes.</span><span class="sxs-lookup"><span data-stu-id="e9f01-149">A typical approach to deploying eventual consistency is to use event sourcing in conjunction with CQRS so that the write model is an append-only stream of events driven by execution of commands.</span></span> <span data-ttu-id="e9f01-150">Ces événements sont utilisés pour mettre à jour les vues matérialisées qui agissent en tant que modèle de lecture.</span><span class="sxs-lookup"><span data-stu-id="e9f01-150">These events are used to update materialized views that act as the read model.</span></span> <span data-ttu-id="e9f01-151">Pour plus d’informations, consultez [Approvisionnement en événements et CQRS](https://msdn.microsoft.com/library/dn568103.aspx#EventSourcingandCQRS).</span><span class="sxs-lookup"><span data-stu-id="e9f01-151">For more information see [Event Sourcing and CQRS](https://msdn.microsoft.com/library/dn568103.aspx#EventSourcingandCQRS).</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="e9f01-152">Quand utiliser ce modèle</span><span class="sxs-lookup"><span data-stu-id="e9f01-152">When to use this pattern</span></span>

<span data-ttu-id="e9f01-153">Ce modèle est utile dans les situations suivantes :</span><span class="sxs-lookup"><span data-stu-id="e9f01-153">Use this pattern in the following situations:</span></span>

- <span data-ttu-id="e9f01-154">Les domaines collaboratifs dans lesquels plusieurs opérations sont effectuées en parallèle sur les mêmes données.</span><span class="sxs-lookup"><span data-stu-id="e9f01-154">Collaborative domains where multiple operations are performed in parallel on the same data.</span></span> <span data-ttu-id="e9f01-155">CQRS vous permet de définir des commandes avec suffisamment de granularité pour réduire les conflits de fusion au niveau du domaine (tous les conflits qui surviennent peuvent être fusionnés par la commande), même lors de la mise à jour de ce qui semble être des données du même type.</span><span class="sxs-lookup"><span data-stu-id="e9f01-155">CQRS allows you to define commands with enough granularity to minimize merge conflicts at the domain level (any conflicts that do arise can be merged by the command), even when updating what appears to be the same type of data.</span></span>

- <span data-ttu-id="e9f01-156">Les interfaces utilisateur basées sur les tâches où les utilisateurs sont guidés tout au long d’un processus complexe comme une série d’étapes ou avec des modèles de domaine complexes.</span><span class="sxs-lookup"><span data-stu-id="e9f01-156">Task-based user interfaces where users are guided through a complex process as a series of steps or with complex domain models.</span></span> <span data-ttu-id="e9f01-157">Ce modèle peut également être utile pour les équipes connaissant déjà les techniques de conception pilotée par domaine.</span><span class="sxs-lookup"><span data-stu-id="e9f01-157">Also, useful for teams already familiar with domain-driven design (DDD) techniques.</span></span> <span data-ttu-id="e9f01-158">Le modèle d’écriture possède une pile de traitement de commande complète avec une logique métier, une validation d’entrée et une validation métier pour garantir que tout est toujours cohérent pour chacun des agrégats (chaque cluster d’objets associés traité comme une unité pour les modifications de données) du modèle d’écriture.</span><span class="sxs-lookup"><span data-stu-id="e9f01-158">The write model has a full command-processing stack with business logic, input validation, and business validation to ensure that everything is always consistent for each of the aggregates (each cluster of associated objects treated as a unit for data changes) in the write model.</span></span> <span data-ttu-id="e9f01-159">Le modèle de lecture n’a aucune pile de validation ou de logique métier et renvoie simplement un DTO pour une utilisation dans un modèle de vue.</span><span class="sxs-lookup"><span data-stu-id="e9f01-159">The read model has no business logic or validation stack and just returns a DTO for use in a view model.</span></span> <span data-ttu-id="e9f01-160">Le modèle de lecture est cohérent de manière éventuelle avec le modèle d’écriture.</span><span class="sxs-lookup"><span data-stu-id="e9f01-160">The read model is eventually consistent with the write model.</span></span>

- <span data-ttu-id="e9f01-161">Les scénarios où les performances des lectures de données doivent être ajustées séparément des performances des écritures de données, en particulier lorsque le taux de lecture/d’écriture est très élevé et lorsqu’une mise à l’échelle horizontale est requise.</span><span class="sxs-lookup"><span data-stu-id="e9f01-161">Scenarios where performance of data reads must be fine tuned separately from performance of data writes, especially when the read/write ratio is very high, and when horizontal scaling is required.</span></span> <span data-ttu-id="e9f01-162">Par exemple, dans de nombreux systèmes le nombre d’opérations de lecture est beaucoup plus important que le nombre d’opérations d’écriture.</span><span class="sxs-lookup"><span data-stu-id="e9f01-162">For example, in many systems the number of read operations is many times greater than the number of write operations.</span></span> <span data-ttu-id="e9f01-163">Pour prendre en charge cela, envisagez d’augmenter la taille des instances du modèle de lecture et d’exécuter le modèle d’écriture sur une ou quelques instances uniquement.</span><span class="sxs-lookup"><span data-stu-id="e9f01-163">To accommodate this, consider scaling out the read model, but running the write model on only one or a few instances.</span></span> <span data-ttu-id="e9f01-164">Un petit nombre d’instances de modèle d’écriture permet également de réduire la fréquence des conflits de fusion.</span><span class="sxs-lookup"><span data-stu-id="e9f01-164">A small number of write model instances also helps to minimize the occurrence of merge conflicts.</span></span>

- <span data-ttu-id="e9f01-165">Les scénarios dans lesquels une équipe de développeurs peut se concentrer sur le modèle de domaine complexe appartenant au modèle d’écriture et dans lesquels une autre équipe peut se concentrer sur le modèle de lecture et les interfaces utilisateur.</span><span class="sxs-lookup"><span data-stu-id="e9f01-165">Scenarios where one team of developers can focus on the complex domain model that is part of the write model, and another team can focus on the read model and the user interfaces.</span></span>

- <span data-ttu-id="e9f01-166">Les scénarios dans lesquels le système doit évoluer au fil du temps et peut contenir plusieurs versions du modèle, ou dans lesquels les règles d’entreprise changent régulièrement.</span><span class="sxs-lookup"><span data-stu-id="e9f01-166">Scenarios where the system is expected to evolve over time and might contain multiple versions of the model, or where business rules change regularly.</span></span>

- <span data-ttu-id="e9f01-167">L’intégration à d’autres systèmes, surtout en association avec l’approvisionnement en événements, où l’échec temporal d’un sous-système ne doit pas affecter la disponibilité des autres.</span><span class="sxs-lookup"><span data-stu-id="e9f01-167">Integration with other systems, especially in combination with event sourcing, where the temporal failure of one subsystem shouldn't affect the availability of the others.</span></span>

<span data-ttu-id="e9f01-168">Ce modèle n’est pas recommandé dans les situations suivantes :</span><span class="sxs-lookup"><span data-stu-id="e9f01-168">This pattern isn't recommended in the following situations:</span></span>

- <span data-ttu-id="e9f01-169">Quand le domaine ou les règles d’entreprise sont simples.</span><span class="sxs-lookup"><span data-stu-id="e9f01-169">Where the domain or the business rules are simple.</span></span>

- <span data-ttu-id="e9f01-170">Quand une interface utilisateur simple de style CRUD et les opérations d’accès aux données associées sont suffisantes.</span><span class="sxs-lookup"><span data-stu-id="e9f01-170">Where a simple CRUD-style user interface and the related data access operations are sufficient.</span></span>

- <span data-ttu-id="e9f01-171">Pour l’implémentation dans tout le système.</span><span class="sxs-lookup"><span data-stu-id="e9f01-171">For implementation across the whole system.</span></span> <span data-ttu-id="e9f01-172">Il existe des composants spécifiques d’un scénario de gestion de données global dans lequel le modèle CQRS peut être utile, mais cela peut ajouter un certain degré de complexité alors que cela n’est pas nécessaire.</span><span class="sxs-lookup"><span data-stu-id="e9f01-172">There are specific components of an overall data management scenario where CQRS can be useful, but it can add considerable and unnecessary complexity when it isn't required.</span></span>

## <a name="event-sourcing-and-cqrs"></a><span data-ttu-id="e9f01-173">Approvisionnement en événements et CQRS</span><span class="sxs-lookup"><span data-stu-id="e9f01-173">Event Sourcing and CQRS</span></span>

<span data-ttu-id="e9f01-174">Le modèle CQRS est souvent utilisé avec le modèle d’approvisionnement en événements.</span><span class="sxs-lookup"><span data-stu-id="e9f01-174">The CQRS pattern is often used along with the Event Sourcing pattern.</span></span> <span data-ttu-id="e9f01-175">Les systèmes basés sur le modèle CQRS utilisent des modèles de données de lecture et d’écriture séparés, tous adaptés aux tâches appropriées et souvent situés dans des magasins distincts physiquement.</span><span class="sxs-lookup"><span data-stu-id="e9f01-175">CQRS-based systems use separate read and write data models, each tailored to relevant tasks and often located in physically separate stores.</span></span> <span data-ttu-id="e9f01-176">Lorsqu’il est utilisé avec le modèle [Approvisionnement en événements](event-sourcing.md), le magasin d’événements est le modèle d’écriture, ainsi que la source officielle d’informations.</span><span class="sxs-lookup"><span data-stu-id="e9f01-176">When used with the [Event Sourcing](event-sourcing.md) pattern, the store of events is the write model, and is the official source of information.</span></span> <span data-ttu-id="e9f01-177">Le modèle de lecture d’un système basé sur le modèle CQRS fournit des vues matérialisées des données, généralement sous forme de vues extrêmement dénormalisées.</span><span class="sxs-lookup"><span data-stu-id="e9f01-177">The read model of a CQRS-based system provides materialized views of the data, typically as highly denormalized views.</span></span> <span data-ttu-id="e9f01-178">Ces vues sont personnalisées en fonction des interfaces et affichent les exigences de l’application, ce qui permet d’optimiser les performances de requête et d’affichage.</span><span class="sxs-lookup"><span data-stu-id="e9f01-178">These views are tailored to the interfaces and display requirements of the application, which helps to maximize both display and query performance.</span></span>

<span data-ttu-id="e9f01-179">L’utilisation du flux d’événements en tant que magasin d’écriture au lieu de données réelles à un point dans le temps évite les conflits de mise à jour d’un agrégat unique et optimise les performances et l’extensibilité.</span><span class="sxs-lookup"><span data-stu-id="e9f01-179">Using the stream of events as the write store, rather than the actual data at a point in time, avoids update conflicts on a single aggregate and maximizes performance and scalability.</span></span> <span data-ttu-id="e9f01-180">Les événements peuvent être utilisés pour générer de façon asynchrone des vues matérialisées des données qui servent à remplir le magasin de lecture.</span><span class="sxs-lookup"><span data-stu-id="e9f01-180">The events can be used to asynchronously generate materialized views of the data that are used to populate the read store.</span></span>

<span data-ttu-id="e9f01-181">Étant donné que le magasin d’événements est la source officielle d’informations, il est possible de supprimer les vues matérialisées et de réexécuter tous les événements passés pour créer une représentation de l’état actuel lorsque le système évolue ou lorsque le modèle de lecture doit changer.</span><span class="sxs-lookup"><span data-stu-id="e9f01-181">Because the event store is the official source of information, it is possible to delete the materialized views and replay all past events to create a new representation of the current state when the system evolves, or when the read model must change.</span></span> <span data-ttu-id="e9f01-182">Les vues matérialisées sont en effet un cache durable en lecture seule des données.</span><span class="sxs-lookup"><span data-stu-id="e9f01-182">The materialized views are in effect a durable read-only cache of the data.</span></span>

<span data-ttu-id="e9f01-183">Lorsque vous utilisez CQRS avec le modèle d’approvisionnement en événements, réfléchissez aux points suivants :</span><span class="sxs-lookup"><span data-stu-id="e9f01-183">When using CQRS combined with the Event Sourcing pattern, consider the following:</span></span>

- <span data-ttu-id="e9f01-184">Comme avec n’importe quel système dans lequel les magasins d’écriture et de lecture sont séparés, les systèmes basés sur ce modèle ne sont plus cohérents de manière éventuelle.</span><span class="sxs-lookup"><span data-stu-id="e9f01-184">As with any system where the write and read stores are separate, systems based on this pattern are only eventually consistent.</span></span> <span data-ttu-id="e9f01-185">Il y aura un certain délai entre l’événement généré et le magasin de données mis à jour.</span><span class="sxs-lookup"><span data-stu-id="e9f01-185">There will be some delay between the event being generated and the data store being updated.</span></span>

- <span data-ttu-id="e9f01-186">Le modèle ajoute un certain degré de complexité, car le code doit être créé pour initier et traiter des événements, ainsi que pour assembler ou mettre à jour les vues ou objets appropriés nécessaires par requêtes ou un modèle de lecture.</span><span class="sxs-lookup"><span data-stu-id="e9f01-186">The pattern adds complexity because code must be created to initiate and handle events, and assemble or update the appropriate views or objects required by queries or a read model.</span></span> <span data-ttu-id="e9f01-187">La complexité du modèle CQRS lorsqu’il est utilisé avec le modèle d’approvisionnement en événements peut compliquer une implémentation réussie et nécessite une approche différente pour la conception des systèmes.</span><span class="sxs-lookup"><span data-stu-id="e9f01-187">The complexity of the CQRS pattern when used with the Event Sourcing pattern can make a successful implementation more difficult, and requires a different approach to designing systems.</span></span> <span data-ttu-id="e9f01-188">Toutefois, l’approvisionnement en événements peut faciliter la modélisation du domaine et la régénération des vues ou leur création, car la nature des modifications des données est préservée.</span><span class="sxs-lookup"><span data-stu-id="e9f01-188">However, event sourcing can make it easier to model the domain, and makes it easier to rebuild views or create new ones because the intent of the changes in the data is preserved.</span></span>

- <span data-ttu-id="e9f01-189">Générer des vues matérialisées à utiliser dans le modèle de lecture ou les projections des données en réexécutant et en traitant les événements d’entités spécifiques ou de collections d’entités peut nécessiter une utilisation des ressources et un temps de traitement importants.</span><span class="sxs-lookup"><span data-stu-id="e9f01-189">Generating materialized views for use in the read model or projections of the data by replaying and handling the events for specific entities or collections of entities can require significant processing time and resource usage.</span></span> <span data-ttu-id="e9f01-190">Cela est particulièrement vrai si des valeurs d’une longue période doivent être analysées ou additionnées, car tous les événements associés doivent être examinés.</span><span class="sxs-lookup"><span data-stu-id="e9f01-190">This is especially true if it requires summation or analysis of values over long periods, because all the associated events might need to be examined.</span></span> <span data-ttu-id="e9f01-191">Corrigez ce problème en implémentant des captures instantanées des données à des intervalles planifiés, comme un compteur indiquant le nombre total d’occurrences d’une action spécifique ou l’état actuel d’une entité.</span><span class="sxs-lookup"><span data-stu-id="e9f01-191">Resolve this by implementing snapshots of the data at scheduled intervals, such as a total count of the number of a specific action that have occurred, or the current state of an entity.</span></span>

## <a name="example"></a><span data-ttu-id="e9f01-192">exemples</span><span class="sxs-lookup"><span data-stu-id="e9f01-192">Example</span></span>

<span data-ttu-id="e9f01-193">Le code suivant présente certains extraits d’un exemple d’implémentation CQRS qui utilise diverses définitions pour les modèles de lecture et d’écriture.</span><span class="sxs-lookup"><span data-stu-id="e9f01-193">The following code shows some extracts from an example of a CQRS implementation that uses different definitions for the read and the write models.</span></span> <span data-ttu-id="e9f01-194">Les interfaces du modèle n’imposent pas de fonctionnalités des magasins de données sous-jacents ; elles peuvent également évoluer et être ajustées de façon indépendante car elles sont séparées.</span><span class="sxs-lookup"><span data-stu-id="e9f01-194">The model interfaces don't dictate any features of the underlying data stores, and they can evolve and be fine-tuned independently because these interfaces are separated.</span></span>

<span data-ttu-id="e9f01-195">Le code suivant montre la définition du modèle de lecture.</span><span class="sxs-lookup"><span data-stu-id="e9f01-195">The following code shows the read model definition.</span></span>

```csharp
// Query interface
namespace ReadModel
{
  public interface ProductsDao
  {
    ProductDisplay FindById(int productId);
    ICollection<ProductDisplay> FindByName(string name);
    ICollection<ProductInventory> FindOutOfStockProducts();
    ICollection<ProductDisplay> FindRelatedProducts(int productId);
  }

  public class ProductDisplay
  {
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal UnitPrice { get; set; }
    public bool IsOutOfStock { get; set; }
    public double UserRating { get; set; }
  }

  public class ProductInventory
  {
    public int Id { get; set; }
    public string Name { get; set; }
    public int CurrentStock { get; set; }
  }
}
```

<span data-ttu-id="e9f01-196">Le système permet aux utilisateurs d’évaluer des produits.</span><span class="sxs-lookup"><span data-stu-id="e9f01-196">The system allows users to rate products.</span></span> <span data-ttu-id="e9f01-197">Le code d’application effectue cette opération à l’aide de la commande `RateProduct` indiquée dans le code suivant.</span><span class="sxs-lookup"><span data-stu-id="e9f01-197">The application code does this using the `RateProduct` command shown in the following code.</span></span>

```csharp
public interface ICommand
{
  Guid Id { get; }
}

public class RateProduct : ICommand
{
  public RateProduct()
  {
    this.Id = Guid.NewGuid();
  }
  public Guid Id { get; set; }
  public int ProductId { get; set; }
  public int Rating { get; set; }
  public int UserId {get; set; }
}
```

<span data-ttu-id="e9f01-198">Le système utilise la classe `ProductsCommandHandler` pour gérer les commandes envoyées par l’application.</span><span class="sxs-lookup"><span data-stu-id="e9f01-198">The system uses the `ProductsCommandHandler` class to handle commands sent by the application.</span></span> <span data-ttu-id="e9f01-199">En règle générale, les clients envoient des commandes au domaine via un système de messagerie comme une file d’attente.</span><span class="sxs-lookup"><span data-stu-id="e9f01-199">Clients typically send commands to the domain through a messaging system such as a queue.</span></span> <span data-ttu-id="e9f01-200">Le gestionnaire de commandes accepte ces commandes et appelle les méthodes de l’interface du domaine.</span><span class="sxs-lookup"><span data-stu-id="e9f01-200">The command handler accepts these commands and invokes methods of the domain interface.</span></span> <span data-ttu-id="e9f01-201">La granularité de chaque commande est conçue pour réduire le risque de requêtes en conflit.</span><span class="sxs-lookup"><span data-stu-id="e9f01-201">The granularity of each command is designed to reduce the chance of conflicting requests.</span></span> <span data-ttu-id="e9f01-202">Le code suivant montre une description de la classe `ProductsCommandHandler`.</span><span class="sxs-lookup"><span data-stu-id="e9f01-202">The following code shows an outline of the `ProductsCommandHandler` class.</span></span>

```csharp
public class ProductsCommandHandler :
    ICommandHandler<AddNewProduct>,
    ICommandHandler<RateProduct>,
    ICommandHandler<AddToInventory>,
    ICommandHandler<ConfirmItemShipped>,
    ICommandHandler<UpdateStockFromInventoryRecount>
{
  private readonly IRepository<Product> repository;

  public ProductsCommandHandler (IRepository<Product> repository)
  {
    this.repository = repository;
  }

  void Handle (AddNewProduct command)
  {
    ...
  }

  void Handle (RateProduct command)
  {
    var product = repository.Find(command.ProductId);
    if (product != null)
    {
      product.RateProduct(command.UserId, command.Rating);
      repository.Save(product);
    }
  }

  void Handle (AddToInventory command)
  {
    ...
  }

  void Handle (ConfirmItemsShipped command)
  {
    ...
  }

  void Handle (UpdateStockFromInventoryRecount command)
  {
    ...
  }
}
```

<span data-ttu-id="e9f01-203">Le code suivant illustre l’interface `IProductsDomain` du modèle d’écriture.</span><span class="sxs-lookup"><span data-stu-id="e9f01-203">The following code shows the `IProductsDomain` interface from the write model.</span></span>

```csharp
public interface IProductsDomain
{
  void AddNewProduct(int id, string name, string description, decimal price);
  void RateProduct(int userId, int rating);
  void AddToInventory(int productId, int quantity);
  void ConfirmItemsShipped(int productId, int quantity);
  void UpdateStockFromInventoryRecount(int productId, int updatedQuantity);
}
```

<span data-ttu-id="e9f01-204">Notez également de quelle façon l’interface `IProductsDomain` contient des méthodes qui ont une signification dans le domaine.</span><span class="sxs-lookup"><span data-stu-id="e9f01-204">Also notice how the `IProductsDomain` interface contains methods that have a meaning in the domain.</span></span> <span data-ttu-id="e9f01-205">En règle générale, dans un environnement CRUD ces méthodes auraient des noms génériques tels que `Save` ou `Update`, et auraient un DTO comme argument unique.</span><span class="sxs-lookup"><span data-stu-id="e9f01-205">Typically, in a CRUD environment these methods would have generic names such as `Save` or `Update`, and have a DTO as the only argument.</span></span> <span data-ttu-id="e9f01-206">L’approche CQRS peut être conçue pour répondre aux besoins des systèmes de gestion des stocks et de l’activité de cette organisation.</span><span class="sxs-lookup"><span data-stu-id="e9f01-206">The CQRS approach can be designed to meet the needs of this organization's business and inventory management systems.</span></span>

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="e9f01-207">Conseils et modèles connexes</span><span class="sxs-lookup"><span data-stu-id="e9f01-207">Related patterns and guidance</span></span>

<span data-ttu-id="e9f01-208">Les modèles et les conseils suivants peuvent être utiles quand il s’agit d’implémenter ce modèle :</span><span class="sxs-lookup"><span data-stu-id="e9f01-208">The following patterns and guidance are useful when implementing this pattern:</span></span>

- <span data-ttu-id="e9f01-209">Pour comparer CQRS à d’autres styles d’architecture, consultez [Styles d’architecture](/azure/architecture/guide/architecture-styles/) et [Style d’architecture CQRS](/azure/architecture/guide/architecture-styles/cqrs).</span><span class="sxs-lookup"><span data-stu-id="e9f01-209">For a comparison of CQRS with other architectural styles, see [Architecture styles](/azure/architecture/guide/architecture-styles/) and [CQRS architecture style](/azure/architecture/guide/architecture-styles/cqrs).</span></span>

- <span data-ttu-id="e9f01-210">[Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx) (Manuel d’introduction à la cohérence des données).</span><span class="sxs-lookup"><span data-stu-id="e9f01-210">[Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx).</span></span> <span data-ttu-id="e9f01-211">Cet article explique les problèmes généralement rencontrés en raison de la cohérence éventuelle entre les magasins de données de lecture et d’écriture lors de l’utilisation du modèle CQRS et la façon dont ces problèmes peuvent être résolus.</span><span class="sxs-lookup"><span data-stu-id="e9f01-211">Explains the issues that are typically encountered due to eventual consistency between the read and write data stores when using the CQRS pattern, and how these issues can be resolved.</span></span>

- <span data-ttu-id="e9f01-212">[Partitionnement des données](https://msdn.microsoft.com/library/dn589795.aspx).</span><span class="sxs-lookup"><span data-stu-id="e9f01-212">[Data Partitioning Guidance](https://msdn.microsoft.com/library/dn589795.aspx).</span></span> <span data-ttu-id="e9f01-213">Cet article décrit la façon dont les magasins de données de lecture et d’écriture utilisés dans le modèle CQRS peuvent être divisés en partitions gérées et accessibles de façon distincte pour améliorer l’extensibilité, réduire la contention et optimiser les performances.</span><span class="sxs-lookup"><span data-stu-id="e9f01-213">Describes how the read and write data stores used in the CQRS pattern can be divided into partitions that can be managed and accessed separately to improve scalability, reduce contention, and optimize performance.</span></span>

- <span data-ttu-id="e9f01-214">[Event Sourcing Pattern](event-sourcing.md) (Modèle d’approvisionnement en événements).</span><span class="sxs-lookup"><span data-stu-id="e9f01-214">[Event Sourcing Pattern](event-sourcing.md).</span></span> <span data-ttu-id="e9f01-215">Cet article décrit plus en détail la façon dont l’approvisionnement en événements peut être utilisé avec le modèle CQRS pour simplifier les tâches dans des domaines complexes tout en améliorant les performances, l’extensibilité et la réactivité.</span><span class="sxs-lookup"><span data-stu-id="e9f01-215">Describes in more detail how Event Sourcing can be used with the CQRS pattern to simplify tasks in complex domains while improving performance, scalability, and responsiveness.</span></span> <span data-ttu-id="e9f01-216">Il explique également comment proposer de la cohérence pour les données transactionnelles tout en maintenant un historique et des journaux d’audit complets qui peuvent mettre en place des actions de compensation.</span><span class="sxs-lookup"><span data-stu-id="e9f01-216">As well as how to provide consistency for transactional data while maintaining full audit trails and history that can enable compensating actions.</span></span>

- <span data-ttu-id="e9f01-217">[Modèle de vue matérialisée](materialized-view.md).</span><span class="sxs-lookup"><span data-stu-id="e9f01-217">[Materialized View Pattern](materialized-view.md).</span></span> <span data-ttu-id="e9f01-218">Le mode de lecture d’une implémentation CQRS peut contenir des vues matérialisées des données du modèle d’écriture, ou le modèle de lecture peut être utilisé pour générer des vues matérialisées.</span><span class="sxs-lookup"><span data-stu-id="e9f01-218">The read model of a CQRS implementation can contain materialized views of the write model data, or the read model can be used to generate materialized views.</span></span>

- <span data-ttu-id="e9f01-219">Le guide des modèles et pratiques [CQRS Journey](http://aka.ms/cqrs) (Découverte de CQRS).</span><span class="sxs-lookup"><span data-stu-id="e9f01-219">The patterns & practices guide [CQRS Journey](http://aka.ms/cqrs).</span></span> <span data-ttu-id="e9f01-220">En particulier [Introducing the Command Query Responsibility Segregation Pattern](https://msdn.microsoft.com/library/jj591573.aspx) (Présentation du modèle de séparation des responsabilités en matière de commande et de requête) qui explore le modèle et les situations dans lesquelles il peut vous être utile et [Epilogue: Lessons Learned](https://msdn.microsoft.com/library/jj591568.aspx) (Épilogue : leçons apprises) qui vous aide à comprendre certains des problèmes liés à l’utilisation de ce domaine.</span><span class="sxs-lookup"><span data-stu-id="e9f01-220">In particular, [Introducing the Command Query Responsibility Segregation Pattern](https://msdn.microsoft.com/library/jj591573.aspx) explores the pattern and when it's useful, and [Epilogue: Lessons Learned](https://msdn.microsoft.com/library/jj591568.aspx) helps you understand some of the issues that come up when using this pattern.</span></span>

- <span data-ttu-id="e9f01-221">L’article [CQRS de Martin Fowler](http://martinfowler.com/bliki/CQRS.html) qui explique les notions de base du modèle et propose des liens vers d’autres ressources utiles.</span><span class="sxs-lookup"><span data-stu-id="e9f01-221">The post [CQRS by Martin Fowler](http://martinfowler.com/bliki/CQRS.html), which explains the basics of the pattern and links to other useful resources.</span></span>

- <span data-ttu-id="e9f01-222">[Les articles de Greg Young](http://codebetter.com/gregyoung/) qui abordent de nombreux aspects du modèle CQRS.</span><span class="sxs-lookup"><span data-stu-id="e9f01-222">[Greg Young’s posts](http://codebetter.com/gregyoung/), which explore many aspects of the CQRS pattern.</span></span>
