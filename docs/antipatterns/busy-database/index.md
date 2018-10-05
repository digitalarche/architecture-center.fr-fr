---
title: Antimodèle de base de données occupé
description: Le déchargement du traitement sur un serveur de base de données peut entraîner des problèmes de performances et d’évolutivité.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: a14a350aefc1801ae08cb4a8d0eb3d5b248c92bf
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428905"
---
# <a name="busy-database-antipattern"></a>Antimodèle de base de données occupé

Suite au déchargement du traitement sur un serveur de base de données, ce dernier peut consacrer une part importante de son temps à exécuter du code plutôt qu’à répondre aux demandes de stockage et de récupération des données. 

## <a name="problem-description"></a>Description du problème

De nombreux systèmes de base de données peuvent exécuter du code. Il peut s’agir de procédures et de déclencheurs stockés. Souvent, il est plus efficace d’effectuer ce traitement proche des données que de transmettre les données à une application cliente. Toutefois, un abus de ces fonctionnalités peut nuire aux performances, pour plusieurs raisons :

- Il se peut que le serveur de base de données passe trop de temps à traiter des opérations, plutôt qu’à accepter de nouvelles demandes de client et à extraire des données.
- Une base de données est généralement une ressource partagée pouvant devenir un goulot d’étranglement pendant les périodes d’utilisation intensive.
- Les coûts d’exécution peuvent être excessifs si la banque de données est mesurée. Cela est particulièrement vrai pour les services de base de données managés. Par exemple, il peut y avoir des frais liés à Microsoft Azure SQL Database pour les [unités de transaction de base de données][dtu] (DTU).
- Les bases de données ont une capacité finie de montée en puissance, et il n’est pas simple de procéder à une mise à l’échelle horizontale. Par conséquent, il peut être préférable de transférer le traitement vers une ressource de calcul, comme une machine virtuelle ou une application App Service pouvant facilement augmenter la taille des instances.

Cet antimodèle survient généralement pour les raisons suivantes :

- La base de données est considérée comme un service plutôt qu’un référentiel. Une application peut utiliser le serveur de base de données pour mettre en forme les données (par exemple, la conversion en XML), manipuler les données de chaîne ou effectuer des calculs complexes.
- Les développeurs tentent d’écrire des requêtes dont les résultats peuvent être affichés directement pour les utilisateurs. Par exemple, une requête pour combiner des champs, ou formater des dates, des heures et des devises en fonction de paramètres régionaux.
- Les développeurs essaient de corriger l’antimodèle [Récupération superflus] [ ExtraneousFetching] en envoyant des calculs à la base de données.
- Les procédures stockées sont utilisées pour encapsuler une logique métier, peut-être parce qu’elles sont considérées comme étant plus faciles à gérer et à mettre à jour.

L’exemple suivant récupère les 20 commandes les plus utiles pour un secteur de vente défini et met en forme les résultats au format XML.
Il utilise des fonctions Transact-SQL pour analyser les données et convertir les résultats au format XML. Vous trouverez l’exemple complet [ici][sample-app].

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

Il s’agit clairement d’une requête complexe. Comme nous le verrons plus tard, il s’avère qu'un nombre important de ressources de traitement sont consommées sur le serveur de base de données.

## <a name="how-to-fix-the-problem"></a>Comment corriger le problème

Pour corriger le problème, une solution consiste à déplacer le traitement du serveur de base de données vers les autres niveaux d’application. Dans l’idéal, vous devez limiter la base de données à l’exécution d’opérations d’accès aux données, en vous servant uniquement des fonctionnalités pour lesquelles la base de données est optimisée, par exemple l’agrégation de données.

Par exemple, le code Transact-SQL précédent peut être remplacé par une instruction qui extrait simplement les données à traiter.

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

L’application utilise ensuite les API `System.Xml.Linq` .NET Framework afin de mettre les résultats au format XML.

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
> Ce code est relativement complexe. Pour une nouvelle application, vous préférerez peut-être utiliser une bibliothèque de sérialisation. Toutefois, on suppose ici que l’équipe de développement refactorise une application existante, par conséquent, la méthode doit retourner exactement le même format que le code d’origine.

## <a name="considerations"></a>Considérations

- De nombreux systèmes de base de données sont hautement optimisés pour effectuer certains types de traitement des données, tels que le calcul de valeurs agrégées sur des jeux de données volumineux. Ne placez pas ces types de traitement en dehors de la base de données.

- Ne déplacez pas le traitement, sous peine d’entraîner le transfert par la base de données d’un volume beaucoup plus important de données via le réseau. Consultez [Antimodèle de récupération superflue][ExtraneousFetching].

- Si vous déplacez le traitement vers une couche Application, il peut être nécessaire d’augmenter la taille des instances de cette couche pour gérer le surplus de travail.

## <a name="how-to-detect-the-problem"></a>Comment détecter le problème

Une base de données occupée se traduit par une baisse disproportionnée du débit et des temps de réponse pour les opérations qui accèdent à la base de données. 

Vous pouvez procéder de la manière suivante pour identifier ce problème : 

1. Utilisez l’analyse des performances pour déterminer le temps consacré par le système de production aux activités liées à la base de données.

2. Examinez le travail effectué par la base de données durant ces périodes.

3. Si vous soupçonnez certaines opérations d’entraîner une activité de base de données trop importante, effectuez un test de charge dans un environnement contrôlé. Chaque test doit exécuter un mélange d’opérations suspectes avec une charge utilisateur variable. Examinez les données télémétriques issues des tests de charge pour observer l’utilisation de la base de données.

4. Si l’activité de la base de données révèle un traitement important, mais un trafic de données peu élevé, examinez le code source pour déterminer s’il est possible d’effectuer le traitement ailleurs de manière plus performante.

Si le volume d’activité de la base de données est faible ou si les temps de réponse sont relativement courts, il est peu probable qu’une base de données occupée soit la cause du problème de performances.

## <a name="example-diagnosis"></a>Exemple de diagnostic

Les sections suivantes appliquent ces étapes à l’exemple d’application décrit précédemment.

### <a name="monitor-the-volume-of-database-activity"></a>Analyser le volume d’activité de la base de données

Le graphique suivant montre les résultats de l’exécution d’un test de charge sur l’application exemple à l’aide d’une rampe de charge pouvant atteindre 50 utilisateurs simultanés. Le volume des demandes atteint rapidement la limite et reste à ce niveau, alors que le temps de réponse moyen augmente régulièrement. Notez qu’une échelle logarithmique est utilisée pour ces deux mesures.

![Résultats du test de charge pour un traitement dans la base de données][ProcessingInDatabaseLoadTest]

Le graphique suivant montre l’utilisation du processeur et des DTU sous forme de pourcentage de quota de service. Les DTU fournissent une mesure de la quantité de traitement effectuée par la base de données. Le graphique montre que l’utilisation du processeur et des DTU a rapidement atteint 100 %.

![Moniteur Microsoft Azure SQL Database montrant les performances de la base de données lors du traitement][ProcessingInDatabaseMonitor]

### <a name="examine-the-work-performed-by-the-database"></a>Examiner le travail effectué par la base de données

Il se peut que les tâches effectuées par la base de données soient des opérations d’accès aux données authentiques, plutôt qu’un traitement. Par conséquent, il est important de comprendre les instructions SQL en cours d’exécution lorsque la base de données est occupée. Analysez le système pour capturer le trafic SQL et mettre en corrélation les opérations SQL avec les demandes d’application.

Si les opérations de base de données sont uniquement des opérations d’accès aux données avec peu de traitement, il se peut que le problème soit lié à une [récupération superflue][ExtraneousFetching].

### <a name="implement-the-solution-and-verify-the-result"></a>Implémenter la solution et vérifier le résultat

Le graphique suivant montre un test de charge utilisant le code mis à jour. Le débit est considérablement plus élevé avec plus de 400 demandes par seconde contre 12 précédemment. Le temps de réponse moyen est également beaucoup plus faible avec un peu plus de 0,1 seconde contre plus de 4 secondes.

![Résultats du test de charge pour un traitement dans la base de données][ProcessingInClientApplicationLoadTest]

L’utilisation du processeur et des DTU indique que le système a pris plus de temps pour atteindre le niveau de saturation, malgré le débit accru.

![Moniteur Microsoft Azure SQL Database montrant les performances de la base de données lors du traitement dans l’application cliente][ProcessingInClientApplicationMonitor]

## <a name="related-resources"></a>Ressources associées 

- [Antimodèle de récupération superflue][ExtraneousFetching]


[dtu]: /azure/sql-database/sql-database-service-tiers-dtu
[ExtraneousFetching]: ../extraneous-fetching/index.md
[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/BusyDatabase

[ProcessingInDatabaseLoadTest]: ./_images/ProcessingInDatabaseLoadTest.jpg
[ProcessingInClientApplicationLoadTest]: ./_images/ProcessingInClientApplicationLoadTest.jpg
[ProcessingInDatabaseMonitor]: ./_images/ProcessingInDatabaseMonitor.jpg
[ProcessingInClientApplicationMonitor]: ./_images/ProcessingInClientApplicationMonitor.jpg
