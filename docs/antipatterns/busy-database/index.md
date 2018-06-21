---
title: Antimodèle de base de données occupé
description: Le déchargement du traitement sur un serveur de base de données peut entraîner des problèmes de performances et d’évolutivité.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 9fdbde0731a1be570ef611894a9d23a1be87f4e7
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2017
ms.locfileid: "24538791"
---
# <a name="busy-database-antipattern"></a><span data-ttu-id="6d486-103">Antimodèle de base de données occupé</span><span class="sxs-lookup"><span data-stu-id="6d486-103">Busy Database antipattern</span></span>

<span data-ttu-id="6d486-104">Suite au déchargement du traitement sur un serveur de base de données, ce dernier peut consacrer une part importante de son temps à exécuter du code plutôt qu’à répondre aux demandes de stockage et de récupération des données.</span><span class="sxs-lookup"><span data-stu-id="6d486-104">Offloading processing to a database server can cause it to spend a significant proportion of time running code, rather than responding to requests to store and retrieve data.</span></span> 

## <a name="problem-description"></a><span data-ttu-id="6d486-105">Description du problème</span><span class="sxs-lookup"><span data-stu-id="6d486-105">Problem description</span></span>

<span data-ttu-id="6d486-106">De nombreux systèmes de base de données peuvent exécuter du code.</span><span class="sxs-lookup"><span data-stu-id="6d486-106">Many database systems can run code.</span></span> <span data-ttu-id="6d486-107">Il peut s’agir de procédures et de déclencheurs stockés.</span><span class="sxs-lookup"><span data-stu-id="6d486-107">Examples include stored procedures and triggers.</span></span> <span data-ttu-id="6d486-108">Souvent, il est plus efficace d’effectuer ce traitement proche des données que de transmettre les données à une application cliente.</span><span class="sxs-lookup"><span data-stu-id="6d486-108">Often, it's more efficient to perform this processing close to the data, rather than transmitting the data to a client application for processing.</span></span> <span data-ttu-id="6d486-109">Toutefois, un abus de ces fonctionnalités peut nuire aux performances, pour plusieurs raisons :</span><span class="sxs-lookup"><span data-stu-id="6d486-109">However, overusing these features can hurt performance, for several reasons:</span></span>

- <span data-ttu-id="6d486-110">Il se peut que le serveur de base de données passe trop de temps à traiter des opérations, plutôt qu’à accepter de nouvelles demandes de client et à extraire des données.</span><span class="sxs-lookup"><span data-stu-id="6d486-110">The database server may spend too much time processing, rather than accepting new client requests and fetching data.</span></span>
- <span data-ttu-id="6d486-111">Une base de données est généralement une ressource partagée pouvant devenir un goulot d’étranglement pendant les périodes d’utilisation intensive.</span><span class="sxs-lookup"><span data-stu-id="6d486-111">A database is usually a shared resource, so it can become a bottleneck during periods of high use.</span></span>
- <span data-ttu-id="6d486-112">Les coûts d’exécution peuvent être excessifs si la banque de données est mesurée.</span><span class="sxs-lookup"><span data-stu-id="6d486-112">Runtime costs may be excessive if the data store is metered.</span></span> <span data-ttu-id="6d486-113">Cela est particulièrement vrai pour les services de base de données managés.</span><span class="sxs-lookup"><span data-stu-id="6d486-113">That's particularly true of managed database services.</span></span> <span data-ttu-id="6d486-114">Par exemple, il peut y avoir des frais liés à Microsoft Azure SQL Database pour les [unités de transaction de base de données][dtu] (DTU).</span><span class="sxs-lookup"><span data-stu-id="6d486-114">For example, Azure SQL Database charges for [Database Transaction Units][dtu] (DTUs).</span></span>
- <span data-ttu-id="6d486-115">Les bases de données ont une capacité finie de montée en puissance pour faire évoluer, et il n’est pas simple de procéder à une mise à l’échelle horizontale d’une base de données.</span><span class="sxs-lookup"><span data-stu-id="6d486-115">Databases have finite capacity to scale up, and it's not trivial to scale a database horizontally.</span></span> <span data-ttu-id="6d486-116">Par conséquent, il peut être préférable de transférer le traitement vers une ressource de calcul, comme une machine virtuelle ou une application App Service qui peut facilement augmenter la taille des instances.</span><span class="sxs-lookup"><span data-stu-id="6d486-116">Therefore, it may be better to move processing into a compute resource, such as a VM or App Service app, that can easily scale out.</span></span>

<span data-ttu-id="6d486-117">Cet antimodèle survient généralement pour les raisons suivantes :</span><span class="sxs-lookup"><span data-stu-id="6d486-117">This antipattern typically occurs because:</span></span>

- <span data-ttu-id="6d486-118">La base de données est considérée comme un service plutôt qu’un référentiel.</span><span class="sxs-lookup"><span data-stu-id="6d486-118">The database is viewed as a service rather than a repository.</span></span> <span data-ttu-id="6d486-119">Une application peut utiliser le serveur de base de données pour mettre en forme les données (par exemple, la conversion en XML), manipuler les données de chaîne ou effectuer des calculs complexes.</span><span class="sxs-lookup"><span data-stu-id="6d486-119">An application might use the database server to format data (for example, converting to XML), manipulate string data, or perform complex calculations.</span></span>
- <span data-ttu-id="6d486-120">Les développeurs tentent d’écrire des requêtes dont les résultats peuvent être affichés directement pour les utilisateurs.</span><span class="sxs-lookup"><span data-stu-id="6d486-120">Developers try to write queries whose results can be displayed directly to users.</span></span> <span data-ttu-id="6d486-121">Par exemple, une requête peut combiner des champs, ou formater des dates, des heures et des devises en fonction de paramètres régionaux.</span><span class="sxs-lookup"><span data-stu-id="6d486-121">For example a query might combine fields, or format dates, times, and currency according to locale.</span></span>
- <span data-ttu-id="6d486-122">Les développeurs essaient de corriger l’antimodèle [Récupération superflus] [ ExtraneousFetching] en envoyant des calculs à la base de données.</span><span class="sxs-lookup"><span data-stu-id="6d486-122">Developers are trying to correct the [Extraneous Fetching][ExtraneousFetching] antipattern by pushing computations to the database.</span></span>
- <span data-ttu-id="6d486-123">Les procédures stockées sont utilisées pour encapsuler une logique métier, peut-être parce qu’elles sont considérées comme étant plus faciles à gérer et à mettre à jour.</span><span class="sxs-lookup"><span data-stu-id="6d486-123">Stored procedures are used to encapsulate business logic, perhaps because they are considered easier to maintain and update.</span></span>

<span data-ttu-id="6d486-124">L’exemple suivant récupère les 20 commandes les plus utiles pour un secteur de vente défini et met en forme les résultats au format XML.</span><span class="sxs-lookup"><span data-stu-id="6d486-124">The following example retrieves the 20 most valuable orders for a specified sales territory and formats the results as XML.</span></span>
<span data-ttu-id="6d486-125">Il utilise des fonctions Transact-SQL pour analyser les données et convertir les résultats au format XML.</span><span class="sxs-lookup"><span data-stu-id="6d486-125">It uses Transact-SQL functions to parse the data and convert the results to XML.</span></span> <span data-ttu-id="6d486-126">Vous trouverez l’exemple complet [ici][sample-app].</span><span class="sxs-lookup"><span data-stu-id="6d486-126">You can find the complete sample [here][sample-app].</span></span>

```SQL
SELECT TOP 20
  soh.[SalesOrderNumber]  AS '@OrderNumber',
  soh.[Status]            AS '@Status',
  soh.[ShipDate]          AS '@ShipDate',
  YEAR(soh.[OrderDate])   AS '@OrderDateYear',
  MONTH(soh.[OrderDate])  AS '@OrderDateMonth',
  soh.[DueDate]           AS '@DueDate',
  FORMAT(ROUND(soh.[SubTotal],2),'C')
                          AS '@SubTotal',
  FORMAT(ROUND(soh.[TaxAmt],2),'C')
                          AS '@TaxAmt',
  FORMAT(ROUND(soh.[TotalDue],2),'C')
                          AS '@TotalDue',
  CASE WHEN soh.[TotalDue] > 5000 THEN 'Y' ELSE 'N' END
                          AS '@ReviewRequired',
  (
  SELECT
    c.[AccountNumber]     AS '@AccountNumber',
    UPPER(LTRIM(RTRIM(REPLACE(
    CONCAT( p.[Title], ' ', p.[FirstName], ' ', p.[MiddleName], ' ', p.[LastName], ' ', p.[Suffix]),
    '  ', ' '))))         AS '@FullName'
  FROM [Sales].[Customer] c
    INNER JOIN [Person].[Person] p
  ON c.[PersonID] = p.[BusinessEntityID]
  WHERE c.[CustomerID] = soh.[CustomerID]
  FOR XML PATH ('Customer'), TYPE
  ),

  (
  SELECT
    sod.[OrderQty]      AS '@Quantity',
    FORMAT(sod.[UnitPrice],'C')
                        AS '@UnitPrice',
    FORMAT(ROUND(sod.[LineTotal],2),'C')
                        AS '@LineTotal',
    sod.[ProductID]     AS '@ProductId',
    CASE WHEN (sod.[ProductID] >= 710) AND (sod.[ProductID] <= 720) AND (sod.[OrderQty] >= 5) THEN 'Y' ELSE 'N' END
                        AS '@InventoryCheckRequired'

  FROM [Sales].[SalesOrderDetail] sod
  WHERE sod.[SalesOrderID] = soh.[SalesOrderID]
  ORDER BY sod.[SalesOrderDetailID]
  FOR XML PATH ('LineItem'), TYPE, ROOT('OrderLineItems')
  )

FROM [Sales].[SalesOrderHeader] soh
WHERE soh.[TerritoryId] = @TerritoryId
ORDER BY soh.[TotalDue] DESC
FOR XML PATH ('Order'), ROOT('Orders')
```

<span data-ttu-id="6d486-127">Il s’agit clairement d’une requête complexe.</span><span class="sxs-lookup"><span data-stu-id="6d486-127">Clearly, this is complex query.</span></span> <span data-ttu-id="6d486-128">Comme nous le verrons plus tard, il s’avère qu’il utilise des ressources de traitement importantes sur le serveur de base de données.</span><span class="sxs-lookup"><span data-stu-id="6d486-128">As we'll see later, it turns out to use significant processing resources on the database server.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="6d486-129">Comment corriger le problème</span><span class="sxs-lookup"><span data-stu-id="6d486-129">How to fix the problem</span></span>

<span data-ttu-id="6d486-130">Déplacer le traitement du serveur de base de données vers les autres niveaux d’application.</span><span class="sxs-lookup"><span data-stu-id="6d486-130">Move processing from the database server into other application tiers.</span></span> <span data-ttu-id="6d486-131">Dans l’idéal, vous devez limiter la base de données à l’exécution d’opérations d’accès aux données, en vous servant uniquement des fonctionnalités pour lesquelles la base de données est optimisée, par exemple l’agrégation dans un SGBDR.</span><span class="sxs-lookup"><span data-stu-id="6d486-131">Ideally, you should limit the database to performing data access operations, using only the capabilities that the database is optimized for, such as aggregation in an RDBMS.</span></span>

<span data-ttu-id="6d486-132">Par exemple, le code Transact-SQL précédent peut être remplacé par une instruction qui extrait simplement les données à traiter.</span><span class="sxs-lookup"><span data-stu-id="6d486-132">For example, the previous Transact-SQL code can be replaced with a statement that simply retrieves the data to be processed.</span></span>

```SQL
SELECT
soh.[SalesOrderNumber]  AS [OrderNumber],
soh.[Status]            AS [Status],
soh.[OrderDate]         AS [OrderDate],
soh.[DueDate]           AS [DueDate],
soh.[ShipDate]          AS [ShipDate],
soh.[SubTotal]          AS [SubTotal],
soh.[TaxAmt]            AS [TaxAmt],
soh.[TotalDue]          AS [TotalDue],
c.[AccountNumber]       AS [AccountNumber],
p.[Title]               AS [CustomerTitle],
p.[FirstName]           AS [CustomerFirstName],
p.[MiddleName]          AS [CustomerMiddleName],
p.[LastName]            AS [CustomerLastName],
p.[Suffix]              AS [CustomerSuffix],
sod.[OrderQty]          AS [Quantity],
sod.[UnitPrice]         AS [UnitPrice],
sod.[LineTotal]         AS [LineTotal],
sod.[ProductID]         AS [ProductId]
FROM [Sales].[SalesOrderHeader] soh
INNER JOIN [Sales].[Customer] c ON soh.[CustomerID] = c.[CustomerID]
INNER JOIN [Person].[Person] p ON c.[PersonID] = p.[BusinessEntityID]
INNER JOIN [Sales].[SalesOrderDetail] sod ON soh.[SalesOrderID] = sod.[SalesOrderID]
WHERE soh.[TerritoryId] = @TerritoryId
AND soh.[SalesOrderId] IN (
    SELECT TOP 20 SalesOrderId
    FROM [Sales].[SalesOrderHeader] soh
    WHERE soh.[TerritoryId] = @TerritoryId
    ORDER BY soh.[TotalDue] DESC)
ORDER BY soh.[TotalDue] DESC, sod.[SalesOrderDetailID]
```

<span data-ttu-id="6d486-133">L’application utilise ensuite les API `System.Xml.Linq` .NET Framework afin de mettre les résultats au format XML.</span><span class="sxs-lookup"><span data-stu-id="6d486-133">The application then uses the .NET Framework `System.Xml.Linq` APIs to format the results as XML.</span></span>

```csharp
// Create a new SqlCommand to run the Transact-SQL query
using (var command = new SqlCommand(...))
{
    command.Parameters.AddWithValue("@TerritoryId", id);

    // Run the query and create the initial XML document
    using (var reader = await command.ExecuteReaderAsync())
    {
        var lastOrderNumber = string.Empty;
        var doc = new XDocument();
        var orders = new XElement("Orders");
        doc.Add(orders);

        XElement lineItems = null;
        // Fetch each row in turn, format the results as XML, and add them to the XML document
        while (await reader.ReadAsync())
        {
            var orderNumber = reader["OrderNumber"].ToString();
            if (orderNumber != lastOrderNumber)
            {
                lastOrderNumber = orderNumber;

                var order = new XElement("Order");
                orders.Add(order);
                var customer = new XElement("Customer");
                lineItems = new XElement("OrderLineItems");
                order.Add(customer, lineItems);

                var orderDate = (DateTime)reader["OrderDate"];
                var totalDue = (Decimal)reader["TotalDue"];
                var reviewRequired = totalDue > 5000 ? 'Y' : 'N';

                order.Add(
                    new XAttribute("OrderNumber", orderNumber),
                    new XAttribute("Status", reader["Status"]),
                    new XAttribute("ShipDate", reader["ShipDate"]),
                    ... // More attributes, not shown.

                    var fullName = string.Join(" ",
                        reader["CustomerTitle"],
                        reader["CustomerFirstName"],
                        reader["CustomerMiddleName"],
                        reader["CustomerLastName"],
                        reader["CustomerSuffix"]
                    )
                   .Replace("  ", " ") //remove double spaces
                   .Trim()
                   .ToUpper();

               customer.Add(
                    new XAttribute("AccountNumber", reader["AccountNumber"]),
                    new XAttribute("FullName", fullName));
            }

            var productId = (int)reader["ProductID"];
            var quantity = (short)reader["Quantity"];
            var inventoryCheckRequired = (productId >= 710 && productId <= 720 && quantity >= 5) ? 'Y' : 'N';

            lineItems.Add(
                new XElement("LineItem",
                    new XAttribute("Quantity", quantity),
                    new XAttribute("UnitPrice", ((Decimal)reader["UnitPrice"]).ToString("C")),
                    new XAttribute("LineTotal", RoundAndFormat(reader["LineTotal"])),
                    new XAttribute("ProductId", productId),
                    new XAttribute("InventoryCheckRequired", inventoryCheckRequired)
                ));
        }
        // Match the exact formatting of the XML returned from SQL
        var xml = doc
            .ToString(SaveOptions.DisableFormatting)
            .Replace(" />", "/>");
    }
}
```

> [!NOTE]
> <span data-ttu-id="6d486-134">Ce code est relativement complexe.</span><span class="sxs-lookup"><span data-stu-id="6d486-134">This code is somewhat complex.</span></span> <span data-ttu-id="6d486-135">Pour une nouvelle application, vous préférerez peut-être utiliser une bibliothèque de sérialisation.</span><span class="sxs-lookup"><span data-stu-id="6d486-135">For a new application, you might prefer to use a serialization library.</span></span> <span data-ttu-id="6d486-136">Toutefois, l’on suppose ici que l’équipe de développement refactorise une application existante, par conséquent, la méthode doit retourner exactement le même format que le code d’origine.</span><span class="sxs-lookup"><span data-stu-id="6d486-136">However, the assumption here is that the development team is refactoring an existing application, so the method needs to return the exact same format as the original code.</span></span>

## <a name="considerations"></a><span data-ttu-id="6d486-137">Considérations</span><span class="sxs-lookup"><span data-stu-id="6d486-137">Considerations</span></span>

- <span data-ttu-id="6d486-138">De nombreux systèmes de base de données sont hautement optimisés pour effectuer certains types de traitement des données, tels que le calcul de valeurs agrégées sur des jeux de données volumineux.</span><span class="sxs-lookup"><span data-stu-id="6d486-138">Many database systems are highly optimized to perform certain types of data processing, such as calculating aggregate values over large datasets.</span></span> <span data-ttu-id="6d486-139">Ne placez pas ces types de traitement en dehors de la base de données.</span><span class="sxs-lookup"><span data-stu-id="6d486-139">Don't move those types of processing out of the database.</span></span>

- <span data-ttu-id="6d486-140">Ne déplacez pas le traitement, sous peine d’entraîner le transfert par la base de données d’un volume beaucoup plus important de données via le réseau.</span><span class="sxs-lookup"><span data-stu-id="6d486-140">Do not relocate processing if doing so causes the database to transfer far more data over the network.</span></span> <span data-ttu-id="6d486-141">Consultez [Antimodèle de récupération superflue][ExtraneousFetching].</span><span class="sxs-lookup"><span data-stu-id="6d486-141">See the [Extraneous Fetching antipattern][ExtraneousFetching].</span></span>

- <span data-ttu-id="6d486-142">Si vous déplacez le traitement vers une couche Application, il peut être nécessaire d’augmenter la taille des instances de cette couche pour gérer le surplus de travail.</span><span class="sxs-lookup"><span data-stu-id="6d486-142">If you move processing to an application tier, that tier may need to scale out to handle the additional work.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="6d486-143">Comment détecter le problème</span><span class="sxs-lookup"><span data-stu-id="6d486-143">How to detect the problem</span></span>

<span data-ttu-id="6d486-144">Une base de données occupée se traduit par une baisse disproportionnée du débit et des temps de réponse pour les opérations qui accèdent à la base de données.</span><span class="sxs-lookup"><span data-stu-id="6d486-144">Symptoms of a busy database include a disproportionate decline in throughput and response times in operations that access the database.</span></span> 

<span data-ttu-id="6d486-145">Vous pouvez procéder de la manière suivante pour identifier ce problème :</span><span class="sxs-lookup"><span data-stu-id="6d486-145">You can perform the following steps to help identify this problem:</span></span> 

1. <span data-ttu-id="6d486-146">Utilisez l’analyse des performances pour déterminer le temps consacré par le système de production aux activités liées à la base de données.</span><span class="sxs-lookup"><span data-stu-id="6d486-146">Use performance monitoring to identify how much time the production system spends performing database activity.</span></span>

2. <span data-ttu-id="6d486-147">Examinez le travail effectué par la base de données durant ces périodes.</span><span class="sxs-lookup"><span data-stu-id="6d486-147">Examine the work performed by the database during these periods.</span></span>

3. <span data-ttu-id="6d486-148">Si vous soupçonnez certaines opérations d’entraîner une activité de base de données trop importante, effectuez un test de charge dans un environnement contrôlé.</span><span class="sxs-lookup"><span data-stu-id="6d486-148">If you suspect that particular operations might cause too much database activity, perform load testing in a controlled environment.</span></span> <span data-ttu-id="6d486-149">Chaque test doit exécuter un mélange d’opérations suspectes avec une charge utilisateur variable.</span><span class="sxs-lookup"><span data-stu-id="6d486-149">Each test should run a mixture of the suspect operations with a variable user load.</span></span> <span data-ttu-id="6d486-150">Examinez les données télémétriques issues des tests de charge pour observer l’utilisation de la base de données.</span><span class="sxs-lookup"><span data-stu-id="6d486-150">Examine the telemetry from the load tests to observe how the database is used.</span></span>

4. <span data-ttu-id="6d486-151">Si l’activité de la base de données révèle un traitement important, mais un trafic de données peu élevé, examinez le code source pour déterminer s’il est possible d’effectuer le traitement ailleurs de manière plus performante.</span><span class="sxs-lookup"><span data-stu-id="6d486-151">If the database activity reveals significant processing but little data traffic, review the source code to determine whether the processing can better be performed elsewhere.</span></span>

<span data-ttu-id="6d486-152">Si le volume d’activité de la base de données est faible ou si les temps de réponse sont relativement courts, il est peu probable qu’une base de données occupée soit la cause du problème de performances.</span><span class="sxs-lookup"><span data-stu-id="6d486-152">If the volume of database activity is low or response times are relatively fast, then a busy database is unlikely to be a performance problem.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="6d486-153">Exemple de diagnostic</span><span class="sxs-lookup"><span data-stu-id="6d486-153">Example diagnosis</span></span>

<span data-ttu-id="6d486-154">Les sections suivantes appliquent ces étapes à l’exemple d’application décrit précédemment.</span><span class="sxs-lookup"><span data-stu-id="6d486-154">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="monitor-the-volume-of-database-activity"></a><span data-ttu-id="6d486-155">Analyser le volume d’activité de la base de données</span><span class="sxs-lookup"><span data-stu-id="6d486-155">Monitor the volume of database activity</span></span>

<span data-ttu-id="6d486-156">Le graphique suivant montre les résultats de l’exécution d’un test de charge dans l’exemple d’application à l’aide d’une charge dans l’étape pouvant atteindre 50 utilisateurs simultanés.</span><span class="sxs-lookup"><span data-stu-id="6d486-156">The following graph shows the results of running a load test against the sample application, using a step load of up to 50 concurrent users.</span></span> <span data-ttu-id="6d486-157">Le volume des demandes atteint rapidement la limite et reste à ce niveau, alors que le temps de réponse moyen augmente régulièrement.</span><span class="sxs-lookup"><span data-stu-id="6d486-157">The volume of requests quickly reaches a limit and stays at that level, while the average response time steadily increases.</span></span> <span data-ttu-id="6d486-158">Notez qu’une échelle logarithmique est utilisée pour ces deux mesures.</span><span class="sxs-lookup"><span data-stu-id="6d486-158">Note that a logarithmic scale is used for those two metrics.</span></span>

![Résultats du test de charge pour un traitement dans la base de données][ProcessingInDatabaseLoadTest]

<span data-ttu-id="6d486-160">Le graphique suivant montre l’utilisation du processeur et des DTU sous forme de pourcentage de quota de service.</span><span class="sxs-lookup"><span data-stu-id="6d486-160">The next graph shows CPU utilization and DTUs as a percentage of service quota.</span></span> <span data-ttu-id="6d486-161">Les DTU fournissent une mesure de la quantité de traitement effectuée par la base de données.</span><span class="sxs-lookup"><span data-stu-id="6d486-161">DTUs provides a measure of how much processing the database performs.</span></span> <span data-ttu-id="6d486-162">Le graphique montre que l’utilisation du processeur et des DTU a rapidement atteint 100 %.</span><span class="sxs-lookup"><span data-stu-id="6d486-162">The graph shows that CPU and DTU utilization both quickly reached 100%.</span></span>

![Moniteur Microsoft Azure SQL Database montrant les performances de la base de données lors du traitement][ProcessingInDatabaseMonitor]

### <a name="examine-the-work-performed-by-the-database"></a><span data-ttu-id="6d486-164">Examiner le travail effectué par la base de données</span><span class="sxs-lookup"><span data-stu-id="6d486-164">Examine the work performed by the database</span></span>

<span data-ttu-id="6d486-165">Il se peut que les tâches effectuées par la base de données soient des opérations d’accès aux données authentiques, plutôt qu’un traitement. Par conséquent, il est important de comprendre les instructions SQL en cours d’exécution lorsque la base de données est occupée.</span><span class="sxs-lookup"><span data-stu-id="6d486-165">It could be that the tasks performed by the database are genuine data access operations, rather than processing, so it is important to understand the SQL statements being run while the database is busy.</span></span> <span data-ttu-id="6d486-166">Analysez le système pour capturer le trafic SQL et mettre en corrélation les opérations SQL avec les demandes d’application.</span><span class="sxs-lookup"><span data-stu-id="6d486-166">Monitor the system to capture the SQL traffic and correlate the SQL operations with application requests.</span></span>

<span data-ttu-id="6d486-167">Si les opérations de base de données sont uniquement des opérations d’accès aux données avec peu de traitement, il se peut que le problème soit lié à une [récupération superflue][ExtraneousFetching].</span><span class="sxs-lookup"><span data-stu-id="6d486-167">If the database operations are purely data access operations, without a lot of processing, then the problem might be [Extraneous Fetching][ExtraneousFetching].</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="6d486-168">Implémenter la solution et vérifier le résultat</span><span class="sxs-lookup"><span data-stu-id="6d486-168">Implement the solution and verify the result</span></span>

<span data-ttu-id="6d486-169">Le graphique suivant montre un test de charge utilisant le code mis à jour.</span><span class="sxs-lookup"><span data-stu-id="6d486-169">The following graph shows a load test using the updated code.</span></span> <span data-ttu-id="6d486-170">Le débit est considérablement plus élevé avec plus de 400 demandes par seconde contre 12 précédemment.</span><span class="sxs-lookup"><span data-stu-id="6d486-170">Throughput is significantly higher, over 400 requests per second versus 12 earlier.</span></span> <span data-ttu-id="6d486-171">Le temps de réponse moyen est également beaucoup plus faible avec un peu plus de 0,1 seconde contre plus de 4 secondes.</span><span class="sxs-lookup"><span data-stu-id="6d486-171">The average response time is also much lower, just above 0.1 seconds compared to over 4 seconds.</span></span>

![Résultats du test de charge pour un traitement dans la base de données][ProcessingInClientApplicationLoadTest]

<span data-ttu-id="6d486-173">L’utilisation du processeur et des DTU indique que le système a pris plus de temps pour atteindre le niveau de saturation, malgré le débit accru.</span><span class="sxs-lookup"><span data-stu-id="6d486-173">CPU and DTU utilization shows that the system took longer to reach saturation, despite the increased throughput.</span></span>

![Moniteur Microsoft Azure SQL Database montrant les performances de la base de données lors du traitement dans l’application cliente][ProcessingInClientApplicationMonitor]

## <a name="related-resources"></a><span data-ttu-id="6d486-175">Ressources associées</span><span class="sxs-lookup"><span data-stu-id="6d486-175">Related resources</span></span> 

- <span data-ttu-id="6d486-176">[Antimodèle de récupération superflue][ExtraneousFetching]</span><span class="sxs-lookup"><span data-stu-id="6d486-176">[Extraneous Fetching antipattern][ExtraneousFetching]</span></span>


[dtu]: /sql-database/sql-database-what-is-a-dtu
[ExtraneousFetching]: ../extraneous-fetching/index.md
[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/BusyDatabase

[ProcessingInDatabaseLoadTest]: ./_images/ProcessingInDatabaseLoadTest.jpg
[ProcessingInClientApplicationLoadTest]: ./_images/ProcessingInClientApplicationLoadTest.jpg
[ProcessingInDatabaseMonitor]: ./_images/ProcessingInDatabaseMonitor.jpg
[ProcessingInClientApplicationMonitor]: ./_images/ProcessingInClientApplicationMonitor.jpg
