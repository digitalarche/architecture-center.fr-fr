---
title: Antimodèle E/S bavardes
description: Un grand nombre de demandes d’E/S peut nuire aux performances et à la réactivité.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: daa0c581d31c9389e2853f84075dc44d1e5ba78b
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/31/2018
ms.locfileid: "43325874"
---
# <a name="chatty-io-antipattern"></a><span data-ttu-id="a98ff-103">Antimodèle E/S bavardes</span><span class="sxs-lookup"><span data-stu-id="a98ff-103">Chatty I/O antipattern</span></span>

<span data-ttu-id="a98ff-104">L’effet cumulatif d’un grand nombre de demandes d’E/S peut avoir un impact significatif sur les performances et la réactivité.</span><span class="sxs-lookup"><span data-stu-id="a98ff-104">The cumulative effect of a large number of I/O requests can have a significant impact on performance and responsiveness.</span></span>

## <a name="problem-description"></a><span data-ttu-id="a98ff-105">Description du problème</span><span class="sxs-lookup"><span data-stu-id="a98ff-105">Problem description</span></span>

<span data-ttu-id="a98ff-106">Les appels réseau et les autres opérations d’E/S sont lents par nature si on les compare aux tâches de calcul.</span><span class="sxs-lookup"><span data-stu-id="a98ff-106">Network calls and other I/O operations are inherently slow compared to compute tasks.</span></span> <span data-ttu-id="a98ff-107">Chaque demande d’E/S a généralement une surcharge significative, et l’effet cumulatif de nombreuses opérations d’E/S peut ralentir le système.</span><span class="sxs-lookup"><span data-stu-id="a98ff-107">Each I/O request typically has significant overhead, and the cumulative effect of numerous I/O operations can slow down the system.</span></span> <span data-ttu-id="a98ff-108">Voici certaines causes courantes d’E/S bavardes.</span><span class="sxs-lookup"><span data-stu-id="a98ff-108">Here are some common causes of chatty I/O.</span></span>

### <a name="reading-and-writing-individual-records-to-a-database-as-distinct-requests"></a><span data-ttu-id="a98ff-109">Lecture et écriture d’enregistrements individuels dans une base de données sous forme de demandes distinctes</span><span class="sxs-lookup"><span data-stu-id="a98ff-109">Reading and writing individual records to a database as distinct requests</span></span>

<span data-ttu-id="a98ff-110">L’exemple suivant lit à partir d’une base de données de produits.</span><span class="sxs-lookup"><span data-stu-id="a98ff-110">The following example reads from a database of products.</span></span> <span data-ttu-id="a98ff-111">Il existe trois tables `Product`, `ProductSubcategory`, et `ProductPriceListHistory`.</span><span class="sxs-lookup"><span data-stu-id="a98ff-111">There are three tables, `Product`, `ProductSubcategory`, and `ProductPriceListHistory`.</span></span> <span data-ttu-id="a98ff-112">Le code récupère tous les produits d’une sous-catégorie, ainsi que les informations de tarification, en exécutant une série de requêtes :</span><span class="sxs-lookup"><span data-stu-id="a98ff-112">The code retrieves all of the products in a subcategory, along with the pricing information, by executing a series of queries:</span></span>  

1. <span data-ttu-id="a98ff-113">Interrogation de la sous-catégorie à partir de la table `ProductSubcategory`.</span><span class="sxs-lookup"><span data-stu-id="a98ff-113">Query the subcategory from the `ProductSubcategory` table.</span></span>
2. <span data-ttu-id="a98ff-114">Recherche de tous les produits de cette sous-catégorie en interrogeant la table `Product`.</span><span class="sxs-lookup"><span data-stu-id="a98ff-114">Find all products in that subcategory by querying the `Product` table.</span></span>
3. <span data-ttu-id="a98ff-115">Pour chaque produit, interrogation des données de tarification à partir de la table `ProductPriceListHistory`.</span><span class="sxs-lookup"><span data-stu-id="a98ff-115">For each product, query the pricing data from the `ProductPriceListHistory` table.</span></span>

<span data-ttu-id="a98ff-116">L’application utilise [Entity Framework] [ ef] pour interroger la base de données.</span><span class="sxs-lookup"><span data-stu-id="a98ff-116">The application uses [Entity Framework][ef] to query the database.</span></span> <span data-ttu-id="a98ff-117">Vous trouverez l’exemple complet [ici][code-sample].</span><span class="sxs-lookup"><span data-stu-id="a98ff-117">You can find the complete sample [here][code-sample].</span></span> 

```csharp
public async Task<IHttpActionResult> GetProductsInSubCategoryAsync(int subcategoryId)
{
    using (var context = GetContext())
    {
        // Get product subcategory.
        var productSubcategory = await context.ProductSubcategories
                .Where(psc => psc.ProductSubcategoryId == subcategoryId)
                .FirstOrDefaultAsync();

        // Find products in that category.
        productSubcategory.Product = await context.Products
            .Where(p => subcategoryId == p.ProductSubcategoryId)
            .ToListAsync();

        // Find price history for each product.
        foreach (var prod in productSubcategory.Product)
        {
            int productId = prod.ProductId;
            var productListPriceHistory = await context.ProductListPriceHistory
                .Where(pl => pl.ProductId == productId)
                .ToListAsync();
            prod.ProductListPriceHistory = productListPriceHistory;
        }
        return Ok(productSubcategory);
    }
}
```

<span data-ttu-id="a98ff-118">Cet exemple illustre le problème explicitement, mais parfois un O/RM peut masquer le problème, s’il extrait implicitement les enregistrements enfants un par un.</span><span class="sxs-lookup"><span data-stu-id="a98ff-118">This example shows the problem explicitly, but sometimes an O/RM can mask the problem, if it implicitly fetches child records one at a time.</span></span> <span data-ttu-id="a98ff-119">Ce problème est appelé « Problème N+1 ».</span><span class="sxs-lookup"><span data-stu-id="a98ff-119">This is known as the "N+1 problem".</span></span> 

### <a name="implementing-a-single-logical-operation-as-a-series-of-http-requests"></a><span data-ttu-id="a98ff-120">Implémentation d’une opération logique unique sous forme de série de requêtes HTTP</span><span class="sxs-lookup"><span data-stu-id="a98ff-120">Implementing a single logical operation as a series of HTTP requests</span></span>

<span data-ttu-id="a98ff-121">Cela se produit souvent lorsque les développeurs tentent de suivre un paradigme orienté objet et de traiter des objets distants comme s’il s’agissait d’objets locaux dans la mémoire.</span><span class="sxs-lookup"><span data-stu-id="a98ff-121">This often happens when developers try to follow an object-oriented paradigm, and treat remote objects as if they were local objects in memory.</span></span> <span data-ttu-id="a98ff-122">Cela peut entraîner de trop nombreux allers-retours réseau.</span><span class="sxs-lookup"><span data-stu-id="a98ff-122">This can result in too many network round trips.</span></span> <span data-ttu-id="a98ff-123">Par exemple, l’API web suivante expose les propriétés individuelles des objets `User` par le biais des méthodes GET HTTP individuelles.</span><span class="sxs-lookup"><span data-stu-id="a98ff-123">For example, the following web API exposes the individual properties of `User` objects through individual HTTP GET methods.</span></span> 

```csharp
public class UserController : ApiController
{
    [HttpGet]
    [Route("users/{id:int}/username")]
    public HttpResponseMessage GetUserName(int id)
    {
        ...
    }

    [HttpGet]
    [Route("users/{id:int}/gender")]
    public HttpResponseMessage GetGender(int id)
    {
        ...
    }

    [HttpGet]
    [Route("users/{id:int}/dateofbirth")]
    public HttpResponseMessage GetDateOfBirth(int id)
    {
        ...
    }
}
```

<span data-ttu-id="a98ff-124">Tandis que cette approche est techniquement correcte, la plupart des clients auront probablement besoin d’obtenir plusieurs propriétés pour chaque `User`, ce qui donne un code client qui ressemble à ce qui suit :</span><span class="sxs-lookup"><span data-stu-id="a98ff-124">While there's nothing technically wrong with this approach, most clients will probably need to get several properties for each `User`, resulting in client code like the following.</span></span> 

```csharp
HttpResponseMessage response = await client.GetAsync("users/1/username");
response.EnsureSuccessStatusCode();
var userName = await response.Content.ReadAsStringAsync();

response = await client.GetAsync("users/1/gender");
response.EnsureSuccessStatusCode();
var gender = await response.Content.ReadAsStringAsync();

response = await client.GetAsync("users/1/dateofbirth");
response.EnsureSuccessStatusCode();
var dob = await response.Content.ReadAsStringAsync();
```

### <a name="reading-and-writing-to-a-file-on-disk"></a><span data-ttu-id="a98ff-125">Lecture et écriture sur un fichier du disque</span><span class="sxs-lookup"><span data-stu-id="a98ff-125">Reading and writing to a file on disk</span></span>

<span data-ttu-id="a98ff-126">Les E/S d’un fichier impliquent l’ouverture d’un fichier et le déplacement vers le point approprié avant la lecture ou l’écriture des données.</span><span class="sxs-lookup"><span data-stu-id="a98ff-126">File I/O involves opening a file and moving to the appropriate point before reading or writing data.</span></span> <span data-ttu-id="a98ff-127">Une fois l’opération terminée, le fichier peut être fermé pour économiser les ressources du système d’exploitation.</span><span class="sxs-lookup"><span data-stu-id="a98ff-127">When the operation is complete, the file might be closed to save operating system resources.</span></span> <span data-ttu-id="a98ff-128">Une application qui lit et écrit en permanence de petites quantités d’informations dans un fichier génère une surcharge d’E/S.</span><span class="sxs-lookup"><span data-stu-id="a98ff-128">An application that continually reads and writes small amounts of information to a file will generate significant I/O overhead.</span></span> <span data-ttu-id="a98ff-129">Les petites requêtes en écriture peuvent également entraîner une fragmentation du fichier, ralentissant davantage les opérations d’E/S qui s’ensuivent.</span><span class="sxs-lookup"><span data-stu-id="a98ff-129">Small write requests can also lead to file fragmentation, slowing subsequent I/O operations still further.</span></span> 

<span data-ttu-id="a98ff-130">L’exemple suivant utilise un `FileStream` pour écrire un objet `Customer` dans un fichier.</span><span class="sxs-lookup"><span data-stu-id="a98ff-130">The following example uses a `FileStream` to write a `Customer` object to a file.</span></span> <span data-ttu-id="a98ff-131">La création de `FileStream` a pour effet d’ouvrir le fichier et sa suppression entraîne la fermeture de ce dernier.</span><span class="sxs-lookup"><span data-stu-id="a98ff-131">Creating the `FileStream` opens the file, and disposing it closes the file.</span></span> <span data-ttu-id="a98ff-132">(l’instruction `using` supprime automatiquement l’objet `FileStream`.) Si l’application appelle cette méthode à plusieurs reprises lors de l’ajout de nouveaux clients, la surcharge d’E/S peut rapidement s’accumuler.</span><span class="sxs-lookup"><span data-stu-id="a98ff-132">(The `using` statement automatically disposes the `FileStream` object.) If the application calls this method repeatedly as new customers are added, the I/O overhead can accumulate quickly.</span></span>

```csharp
private async Task SaveCustomerToFileAsync(Customer cust)
{
    using (Stream fileStream = new FileStream(CustomersFileName, FileMode.Append))
    {
        BinaryFormatter formatter = new BinaryFormatter();
        byte [] data = null;
        using (MemoryStream memStream = new MemoryStream())
        {
            formatter.Serialize(memStream, cust);
            data = memStream.ToArray();
        }
        await fileStream.WriteAsync(data, 0, data.Length);
    }
}
```

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="a98ff-133">Comment corriger le problème</span><span class="sxs-lookup"><span data-stu-id="a98ff-133">How to fix the problem</span></span>

<span data-ttu-id="a98ff-134">Réduire la quantité de requêtes d’E/S en plaçant les données dans des requêtes plus grandes et moins importantes en nombre.</span><span class="sxs-lookup"><span data-stu-id="a98ff-134">Reduce the number of I/O requests by packaging the data into larger, fewer requests.</span></span>

<span data-ttu-id="a98ff-135">Récupérer les données d’une base de données à l’aide d’une requête unique et non par le biais de plusieurs petites requêtes.</span><span class="sxs-lookup"><span data-stu-id="a98ff-135">Fetch data from a database as a single query, instead of several smaller queries.</span></span> <span data-ttu-id="a98ff-136">Voici une version révisée du code qui extrait les informations de produit.</span><span class="sxs-lookup"><span data-stu-id="a98ff-136">Here's a revised version of the code that retrieves product information.</span></span>

```csharp
public async Task<IHttpActionResult> GetProductCategoryDetailsAsync(int subCategoryId)
{
    using (var context = GetContext())
    {
        var subCategory = await context.ProductSubcategories
                .Where(psc => psc.ProductSubcategoryId == subCategoryId)
                .Include("Product.ProductListPriceHistory")
                .FirstOrDefaultAsync();

        if (subCategory == null)
            return NotFound();

        return Ok(subCategory);
    }
}
```

<span data-ttu-id="a98ff-137">Suivez les principes de conception REST pour les API web.</span><span class="sxs-lookup"><span data-stu-id="a98ff-137">Follow REST design principles for web APIs.</span></span> <span data-ttu-id="a98ff-138">Voici une version révisée de l’API web de l’exemple précédent.</span><span class="sxs-lookup"><span data-stu-id="a98ff-138">Here's a revised version of the web API from the earlier example.</span></span> <span data-ttu-id="a98ff-139">Au lieu de méthodes GET distinctes pour chaque propriété, il existe une seule méthode GET qui renvoie `User`.</span><span class="sxs-lookup"><span data-stu-id="a98ff-139">Instead of separate GET methods for each property, there is a single GET method that returns the `User`.</span></span> <span data-ttu-id="a98ff-140">Cela entraîne un corps de réponse plus important par requête. Toutefois, chaque client est susceptible d’effectuer moins d’appels d’API.</span><span class="sxs-lookup"><span data-stu-id="a98ff-140">This results in a larger response body per request, but each client is likely to make fewer API calls.</span></span>

```csharp
public class UserController : ApiController
{
    [HttpGet]
    [Route("users/{id:int}")]
    public HttpResponseMessage GetUser(int id)
    {
        ...
    }
}

// Client code
HttpResponseMessage response = await client.GetAsync("users/1");
response.EnsureSuccessStatusCode();
var user = await response.Content.ReadAsStringAsync();
```

<span data-ttu-id="a98ff-141">Pour les E/S de fichier, envisagez de placer les données en mémoire tampon et de les écrire dans un fichier sous la forme d’une seule opération.</span><span class="sxs-lookup"><span data-stu-id="a98ff-141">For file I/O, consider buffering data in memory and then writing the buffered data to a file as a single operation.</span></span> <span data-ttu-id="a98ff-142">Cette approche réduit la surcharge liée à l’ouverture et à la fermeture répétée du fichier et permet de réduire la fragmentation du fichier sur le disque.</span><span class="sxs-lookup"><span data-stu-id="a98ff-142">This approach reduces the overhead from repeatedly opening and closing the file, and helps to reduce fragmentation of the file on disk.</span></span>

```csharp
// Save a list of customer objects to a file
private async Task SaveCustomerListToFileAsync(List<Customer> customers)
{
    using (Stream fileStream = new FileStream(CustomersFileName, FileMode.Append))
    {
        BinaryFormatter formatter = new BinaryFormatter();
        foreach (var cust in customers)
        {
            byte[] data = null;
            using (MemoryStream memStream = new MemoryStream())
            {
                formatter.Serialize(memStream, cust);
                data = memStream.ToArray();
            }
            await fileStream.WriteAsync(data, 0, data.Length);
        }
    }
}

// In-memory buffer for customers.
List<Customer> customers = new List<Customers>();

// Create a new customer and add it to the buffer
var cust = new Customer(...);
customers.Add(cust);

// Add more customers to the list as they are created
...

// Save the contents of the list, writing all customers in a single operation
await SaveCustomerListToFileAsync(customers);
```

## <a name="considerations"></a><span data-ttu-id="a98ff-143">Considérations</span><span class="sxs-lookup"><span data-stu-id="a98ff-143">Considerations</span></span>

- <span data-ttu-id="a98ff-144">Les deux premiers exemples effectuent *moins* d’appels d’E/S et chacun d’eux récupère *plus* d’informations.</span><span class="sxs-lookup"><span data-stu-id="a98ff-144">The first two examples make *fewer* I/O calls, but each one retrieves *more* information.</span></span> <span data-ttu-id="a98ff-145">Vous devez faire un compromis entre ces deux facteurs.</span><span class="sxs-lookup"><span data-stu-id="a98ff-145">You must consider the tradeoff between these two factors.</span></span> <span data-ttu-id="a98ff-146">La réponse appropriée dépend des modèles d’utilisation réels.</span><span class="sxs-lookup"><span data-stu-id="a98ff-146">The right answer will depend on the actual usage patterns.</span></span> <span data-ttu-id="a98ff-147">Par exemple, dans l’exemple d’API web, les clients ont souvent seulement besoin du nom d’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="a98ff-147">For example, in the web API example, it might turn out that clients often need just the user name.</span></span> <span data-ttu-id="a98ff-148">Dans ce cas, il peut être utile de l’exposer sous la forme d’un appel d’API distinct.</span><span class="sxs-lookup"><span data-stu-id="a98ff-148">In that case, it might make sense to expose it as a separate API call.</span></span> <span data-ttu-id="a98ff-149">Pour plus d’informations, consultez l’antimodèle [Récupération superflue][extraneous-fetching].</span><span class="sxs-lookup"><span data-stu-id="a98ff-149">For more information, see the [Extraneous Fetching][extraneous-fetching] antipattern.</span></span>

- <span data-ttu-id="a98ff-150">Lors de la lecture des données, ne créez pas de requêtes d’E/S trop grandes.</span><span class="sxs-lookup"><span data-stu-id="a98ff-150">When reading data, do not make your I/O requests too large.</span></span> <span data-ttu-id="a98ff-151">Une application doit récupérer uniquement les informations qu’elle est susceptible d’utiliser.</span><span class="sxs-lookup"><span data-stu-id="a98ff-151">An application should only retrieve the information that it is likely to use.</span></span> 

- <span data-ttu-id="a98ff-152">Parfois, cela permet de partitionner les informations d’un objet en deux blocs, un pour *les données fréquemment sollicitées* qui concerne la plupart des requêtes et un pour les *données moins souvent sollicitées*, qui sont rarement utilisées.</span><span class="sxs-lookup"><span data-stu-id="a98ff-152">Sometimes it helps to partition the information for an object into two chunks, *frequently accessed data* that accounts for most requests, and *less frequently accessed data* that is used rarely.</span></span> <span data-ttu-id="a98ff-153">Souvent les données fréquemment sollicitées constituent une petite partie de l’ensemble des données d’un objet. Par conséquent, le fait de renvoyer cette partie uniquement peut permettre de réduire la surcharge d’E/S.</span><span class="sxs-lookup"><span data-stu-id="a98ff-153">Often the most frequently accessed data is a relatively small portion of the total data for an object, so returning just that portion can save significant I/O overhead.</span></span>

- <span data-ttu-id="a98ff-154">Lors de l’écriture de données, évitez de verrouiller les ressources plus longtemps que nécessaire, pour réduire les risques de contention au cours d’une opération de longue durée.</span><span class="sxs-lookup"><span data-stu-id="a98ff-154">When writing data, avoid locking resources for longer than necessary, to reduce the chances of contention during a lengthy operation.</span></span> <span data-ttu-id="a98ff-155">Si une opération d’écriture s’étend sur plusieurs magasins de données, fichiers ou services, il convient d’adopter une approche cohérente.</span><span class="sxs-lookup"><span data-stu-id="a98ff-155">If a write operation spans multiple data stores, files, or services, then adopt an eventually consistent approach.</span></span> <span data-ttu-id="a98ff-156">Reportez-vous aux [conseils en termes de cohérence des données][data-consistency-guidance].</span><span class="sxs-lookup"><span data-stu-id="a98ff-156">See [Data Consistency guidance][data-consistency-guidance].</span></span>

- <span data-ttu-id="a98ff-157">Le fait de placer les données en mémoire tampon avant de les écrire les rend vulnérables en cas de blocage du processus.</span><span class="sxs-lookup"><span data-stu-id="a98ff-157">If you buffer data in memory before writing it, the data is vulnerable if the process crashes.</span></span> <span data-ttu-id="a98ff-158">Si le débit de données connaît généralement des pics ou est relativement peu important, il peut être plus sûr de mettre les données en mémoire tampon dans une file d’attente durable externe, tels que des[concentrateurs d’événements](http://azure.microsoft.com/services/event-hubs/).</span><span class="sxs-lookup"><span data-stu-id="a98ff-158">If the data rate typically has bursts or is relatively sparse, it may be safer to buffer the data in an external durable queue such as [Event Hubs](http://azure.microsoft.com/services/event-hubs/).</span></span>

- <span data-ttu-id="a98ff-159">Envisagez la mise en cache des données que vous récupérez à partir d’un service ou d’une base de données.</span><span class="sxs-lookup"><span data-stu-id="a98ff-159">Consider caching data that you retrieve from a service or a database.</span></span> <span data-ttu-id="a98ff-160">Cela peut permettre de réduire le volume d’E/S en évitant de répéter les requêtes pour les mêmes données.</span><span class="sxs-lookup"><span data-stu-id="a98ff-160">This can help to reduce the volume of I/O by avoiding repeated requests for the same data.</span></span> <span data-ttu-id="a98ff-161">Pour plus d’informations, voir [Meilleures pratiques pour la mise en cache][caching-guidance].</span><span class="sxs-lookup"><span data-stu-id="a98ff-161">For more information, see [Caching best practices][caching-guidance].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="a98ff-162">Comment détecter le problème</span><span class="sxs-lookup"><span data-stu-id="a98ff-162">How to detect the problem</span></span>

<span data-ttu-id="a98ff-163">Des E/S bavardes se traduisent par une latence élevée et un débit faible.</span><span class="sxs-lookup"><span data-stu-id="a98ff-163">Symptoms of chatty I/O include high latency and low throughput.</span></span> <span data-ttu-id="a98ff-164">Les utilisateurs finaux sont susceptibles de signaler des temps de réponse prolongés ou des défaillances causées par l’expiration du délai d’expiration des services, en raison d’une contention accrue des ressources d’E/S.</span><span class="sxs-lookup"><span data-stu-id="a98ff-164">End users are likely to report extended response times or failures caused by services timing out, due to increased contention for I/O resources.</span></span>

<span data-ttu-id="a98ff-165">Vous pouvez procéder de la manière suivante pour identifier la cause des problèmes.</span><span class="sxs-lookup"><span data-stu-id="a98ff-165">You can perform the following steps to help identify the causes of any problems:</span></span>

1. <span data-ttu-id="a98ff-166">Effectuez l’analyse du processus du système de production pour identifier les opérations pour lesquelles les temps de réponse sont médiocres.</span><span class="sxs-lookup"><span data-stu-id="a98ff-166">Perform process monitoring of the production system to identify operations with poor response times.</span></span>
2. <span data-ttu-id="a98ff-167">Effectuez un test de charge pour chaque opération identifiée à l’étape précédente.</span><span class="sxs-lookup"><span data-stu-id="a98ff-167">Perform load testing of each operation identified in the previous step.</span></span>
3. <span data-ttu-id="a98ff-168">Lors des tests de charge, rassemblez les données de télémétrie concernant les requêtes d’accès aux données effectuées par chaque opération.</span><span class="sxs-lookup"><span data-stu-id="a98ff-168">During the load tests, gather telemetry data about the data access requests made by each operation.</span></span>
4. <span data-ttu-id="a98ff-169">Recueillez les statistiques détaillées de chaque requête envoyée à un magasin de données.</span><span class="sxs-lookup"><span data-stu-id="a98ff-169">Gather detailed statistics for each request sent to a data store.</span></span>
5. <span data-ttu-id="a98ff-170">Profilez l’application dans l’environnement de test pour déterminer les endroits où des goulots d’étranglement peuvent se produire.</span><span class="sxs-lookup"><span data-stu-id="a98ff-170">Profile the application in the test environment to establish where possible I/O bottlenecks might be occurring.</span></span> 

<span data-ttu-id="a98ff-171">Recherchez la présence des symptômes suivants :</span><span class="sxs-lookup"><span data-stu-id="a98ff-171">Look for any of these symptoms:</span></span>

- <span data-ttu-id="a98ff-172">Un grand nombre de petites requêtes d’E/S effectuées pour le même fichier.</span><span class="sxs-lookup"><span data-stu-id="a98ff-172">A large number of small I/O requests made to the same file.</span></span>
- <span data-ttu-id="a98ff-173">Un grand nombre de petites requêtes de réseau effectuées par une instance d’application pour le même service.</span><span class="sxs-lookup"><span data-stu-id="a98ff-173">A large number of small network requests made by an application instance to the same service.</span></span>
- <span data-ttu-id="a98ff-174">Un grand nombre de petites requêtes effectuées par une instance d’application pour le même magasin de données.</span><span class="sxs-lookup"><span data-stu-id="a98ff-174">A large number of small requests made by an application instance to the same data store.</span></span>
- <span data-ttu-id="a98ff-175">Les applications et services sont de plus en plus liés aux E/S.</span><span class="sxs-lookup"><span data-stu-id="a98ff-175">Applications and services becoming I/O bound.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="a98ff-176">Exemple de diagnostic</span><span class="sxs-lookup"><span data-stu-id="a98ff-176">Example diagnosis</span></span>

<span data-ttu-id="a98ff-177">Les sections suivantes appliquent ces étapes à l’exemple présenté plus haut qui interroge une base de données.</span><span class="sxs-lookup"><span data-stu-id="a98ff-177">The following sections apply these steps to the example shown earlier that queries a database.</span></span>

### <a name="load-test-the-application"></a><span data-ttu-id="a98ff-178">Effectuer un test de charge de l’application</span><span class="sxs-lookup"><span data-stu-id="a98ff-178">Load test the application</span></span>

<span data-ttu-id="a98ff-179">Ce graphique montre les résultats du test de charge.</span><span class="sxs-lookup"><span data-stu-id="a98ff-179">This graph shows the results of load testing.</span></span> <span data-ttu-id="a98ff-180">Le temps de réponse médian est mesuré en dixième de seconde par requête.</span><span class="sxs-lookup"><span data-stu-id="a98ff-180">Median response time is measured in 10s of seconds per request.</span></span> <span data-ttu-id="a98ff-181">Le graphique présente une latence très élevée.</span><span class="sxs-lookup"><span data-stu-id="a98ff-181">The graph shows very high latency.</span></span> <span data-ttu-id="a98ff-182">Avec une charge de 1 000 utilisateurs, un utilisateur peut devoir attendre presque une minute pour voir les résultats d’une requête.</span><span class="sxs-lookup"><span data-stu-id="a98ff-182">With a load of 1000 users, a user might have to wait for nearly a minute to see the results of a query.</span></span> 

![Résultats du test de charge des indicateurs clés pour l’exemple d’application d’E/S bavardes][key-indicators-chatty-io]

> [!NOTE]
> <span data-ttu-id="a98ff-184">L’application a été déployée sous forme d’application web Azure App Service à l’aide d’Azure SQL Database.</span><span class="sxs-lookup"><span data-stu-id="a98ff-184">The application was deployed as an Azure App Service web app, using Azure SQL Database.</span></span> <span data-ttu-id="a98ff-185">Le test de charge a utilisé une charge de travail d’étape simulée pouvant atteindre 1 000 utilisateurs simultanés.</span><span class="sxs-lookup"><span data-stu-id="a98ff-185">The load test used a simulated step workload of up to 1000 concurrent users.</span></span> <span data-ttu-id="a98ff-186">La base de données a été configurée avec un pool de connexions prenant en charge jusqu’à 1 000 connexions simultanées afin de réduire le risque de voir la contention des connexions affecter les résultats.</span><span class="sxs-lookup"><span data-stu-id="a98ff-186">The database was configured with a connection pool supporting up to 1000 concurrent connections, to reduce the chance that contention for connections would affect the results.</span></span> 

### <a name="monitor-the-application"></a><span data-ttu-id="a98ff-187">Analyser l’application</span><span class="sxs-lookup"><span data-stu-id="a98ff-187">Monitor the application</span></span>

<span data-ttu-id="a98ff-188">Vous pouvez utiliser un package d’analyse des performances d’application (APM) pour capturer et analyser les métriques clés permettant d’identifier les E/S bavardes.</span><span class="sxs-lookup"><span data-stu-id="a98ff-188">You can use an application performance monitoring (APM) package to capture and analyze the key metrics that might identify chatty I/O.</span></span> <span data-ttu-id="a98ff-189">Les mesures importantes dépendent de la charge de travail d’E/S.</span><span class="sxs-lookup"><span data-stu-id="a98ff-189">Which metrics are important will depend on the I/O workload.</span></span> <span data-ttu-id="a98ff-190">Pour cet exemple, les requêtes d’ES/S intéressantes étaient celles de la base de données.</span><span class="sxs-lookup"><span data-stu-id="a98ff-190">For this example, the interesting I/O requests were the database queries.</span></span> 

<span data-ttu-id="a98ff-191">L’illustration suivante montre les résultats générés à l’aide de l’[APM New Relic][new-relic].</span><span class="sxs-lookup"><span data-stu-id="a98ff-191">The following image shows results generated using [New Relic APM][new-relic].</span></span> <span data-ttu-id="a98ff-192">Le temps de réponse moyen de la base de données a atteint un pic d’environ 5,6 secondes par requête lors de la charge de travail maximale.</span><span class="sxs-lookup"><span data-stu-id="a98ff-192">The average database response time peaked at approximately 5.6 seconds per request during the maximum workload.</span></span> <span data-ttu-id="a98ff-193">Le système a pu prendre en charge une moyenne de 410 requêtes par minute au cours du test.</span><span class="sxs-lookup"><span data-stu-id="a98ff-193">The system was able to support an average of 410 requests per minute throughout the test.</span></span>

![Vue d’ensemble du trafic accédant à la base de données AdventureWorks2012][databasetraffic]

### <a name="gather-detailed-data-access-information"></a><span data-ttu-id="a98ff-195">Collecter des informations détaillées concernant l’accès aux données</span><span class="sxs-lookup"><span data-stu-id="a98ff-195">Gather detailed data access information</span></span>

<span data-ttu-id="a98ff-196">Un examen approfondi des données d’analyse montre que l’application exécute trois instructions SQL SELECT différentes.</span><span class="sxs-lookup"><span data-stu-id="a98ff-196">Digging deeper into the monitoring data shows the application executes three different SQL SELECT statements.</span></span> <span data-ttu-id="a98ff-197">Celles-ci correspondent aux requêtes générées par Entity Framework pour extraire des données à partir des tables `ProductListPriceHistory`, `Product`, et `ProductSubcategory`.</span><span class="sxs-lookup"><span data-stu-id="a98ff-197">These correspond to the requests generated by Entity Framework to fetch data from the `ProductListPriceHistory`, `Product`, and `ProductSubcategory` tables.</span></span>
<span data-ttu-id="a98ff-198">En outre, la requête qui récupère des données de la table `ProductListPriceHistory` est de loin l’instruction SELECT la plus fréquemment exécutée par ordre de grandeur.</span><span class="sxs-lookup"><span data-stu-id="a98ff-198">Furthermore, the query that retrieves data from the `ProductListPriceHistory` table is by far the most frequently executed SELECT statement, by an order of magnitude.</span></span>

![Requêtes exécutées par l’exemple d’application testé][queries]

<span data-ttu-id="a98ff-200">Il s’avère que la méthode `GetProductsInSubCategoryAsync`, présentée précédemment, exécute 45 requêtes SELECT.</span><span class="sxs-lookup"><span data-stu-id="a98ff-200">It turns out that the `GetProductsInSubCategoryAsync` method, shown earlier, performs 45 SELECT queries.</span></span> <span data-ttu-id="a98ff-201">Chaque requête entraîne l’ouverture d’une nouvelle connexion SQL par l’application.</span><span class="sxs-lookup"><span data-stu-id="a98ff-201">Each query causes the application to open a new SQL connection.</span></span>

![Statistiques de requêtes pour l’exemple d’application testé][queries2]

> [!NOTE]
> <span data-ttu-id="a98ff-203">Cette illustration montre les informations de suivi de l’instance la plus lente de l’opération `GetProductsInSubCategoryAsync` du test de charge.</span><span class="sxs-lookup"><span data-stu-id="a98ff-203">This image shows trace information for the slowest instance of the `GetProductsInSubCategoryAsync` operation in the load test.</span></span> <span data-ttu-id="a98ff-204">Dans un environnement de production, il est utile d’examiner les traces des instances les plus lentes, pour voir s’il existe un modèle suggérant un problème.</span><span class="sxs-lookup"><span data-stu-id="a98ff-204">In a production environment, it's useful to examine traces of the slowest instances, to see if there is a pattern that suggests a problem.</span></span> <span data-ttu-id="a98ff-205">Si vous vous contentez d’examiner les valeurs moyennes, vous pouvez négliger des problèmes qui pourraient s’aggraver en charge.</span><span class="sxs-lookup"><span data-stu-id="a98ff-205">If you just look at the average values, you might overlook problems that will get dramatically worse under load.</span></span>

<span data-ttu-id="a98ff-206">L’image suivante montre les instructions SQL réelles qui ont été émises.</span><span class="sxs-lookup"><span data-stu-id="a98ff-206">The next image shows the actual SQL statements that were issued.</span></span> <span data-ttu-id="a98ff-207">La requête qui extrait les informations de prix est exécutée pour chacun des produits de la sous-catégorie de produit.</span><span class="sxs-lookup"><span data-stu-id="a98ff-207">The query that fetches price information is run for each individual product in the product subcategory.</span></span> <span data-ttu-id="a98ff-208">L’utilisation d’une jointure réduirait considérablement le nombre d’appels de la base de données.</span><span class="sxs-lookup"><span data-stu-id="a98ff-208">Using a join would considerably reduce the number of database calls.</span></span>

![Détails relatifs à la requête pour l’exemple d’application testé][queries3]

<span data-ttu-id="a98ff-210">Si vous utilisez un O/RM, tel que Entity Framework, le suivi des requêtes SQL peut donner un aperçu de la manière dont le O/RM traduit les appels par programmation en instructions SQL et indiquer les zones où l’accès aux données peut être optimisé.</span><span class="sxs-lookup"><span data-stu-id="a98ff-210">If you are using an O/RM, such as Entity Framework, tracing the SQL queries can provide insight into how the O/RM translates programmatic calls into SQL statements, and indicate areas where data access might be optimized.</span></span> 

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="a98ff-211">Implémenter la solution et vérifier le résultat</span><span class="sxs-lookup"><span data-stu-id="a98ff-211">Implement the solution and verify the result</span></span>

<span data-ttu-id="a98ff-212">La réécriture de l’appel à Entity Framework a donné les résultats suivants.</span><span class="sxs-lookup"><span data-stu-id="a98ff-212">Rewriting the call to Entity Framework produced the following results.</span></span>

![Résultats du test de charge des indicateurs clés pour l’API regroupée de l’exemple d’application d’E/S bavardes][key-indicators-chunky-io]

<span data-ttu-id="a98ff-214">Ce test de charge a été effectué sur le même déploiement à l’aide du même profil de charge.</span><span class="sxs-lookup"><span data-stu-id="a98ff-214">This load test was performed on the same deployment, using the same load profile.</span></span> <span data-ttu-id="a98ff-215">Cette fois le graphique indique une latence plus faible.</span><span class="sxs-lookup"><span data-stu-id="a98ff-215">This time the graph shows much lower latency.</span></span> <span data-ttu-id="a98ff-216">La durée moyenne de la requête pour 1 000 utilisateurs est comprise entre 5 et 6 secondes contre presque une minute.</span><span class="sxs-lookup"><span data-stu-id="a98ff-216">The average request time at 1000 users is between 5 and 6 seconds, down from nearly a minute.</span></span>

<span data-ttu-id="a98ff-217">Cette fois le système a pris en charge en moyenne 3 970 requêtes par minute, par rapport aux 410 du test précédent.</span><span class="sxs-lookup"><span data-stu-id="a98ff-217">This time the system supported an average of 3,970 requests per minute, compared to 410 for the earlier test.</span></span>

![Vue d’ensemble de la transaction pour l’API regroupée][databasetraffic2]

<span data-ttu-id="a98ff-219">Le suivi de l’instruction SQL indique que toutes les données sont extraites dans une instruction SELECT unique.</span><span class="sxs-lookup"><span data-stu-id="a98ff-219">Tracing the SQL statement shows that all the data is fetched in a single SELECT statement.</span></span> <span data-ttu-id="a98ff-220">Bien que cette requête soit beaucoup plus complexe, elle est exécutée une seule fois par opération.</span><span class="sxs-lookup"><span data-stu-id="a98ff-220">Although this query is considerably more complex, it is performed only once per operation.</span></span> <span data-ttu-id="a98ff-221">Et tandis que des jointures complexes peuvent devenir coûteuses, les systèmes de base de données relationnelle sont optimisés pour ce type de requête.</span><span class="sxs-lookup"><span data-stu-id="a98ff-221">And while complex joins can become expensive, relational database systems are optimized for this type of query.</span></span>  

![Détails de la requête de l’API regroupée][queries4]

## <a name="related-resources"></a><span data-ttu-id="a98ff-223">Ressources associées</span><span class="sxs-lookup"><span data-stu-id="a98ff-223">Related resources</span></span>

- <span data-ttu-id="a98ff-224">[Meilleures pratiques en matière de conception d’API][api-design]</span><span class="sxs-lookup"><span data-stu-id="a98ff-224">[API Design best practices][api-design]</span></span>
- <span data-ttu-id="a98ff-225">[Meilleures pratiques en matière de mise en cache][caching-guidance]</span><span class="sxs-lookup"><span data-stu-id="a98ff-225">[Caching best practices][caching-guidance]</span></span>
- <span data-ttu-id="a98ff-226">[Data Consistency Primer][data-consistency-guidance]</span><span class="sxs-lookup"><span data-stu-id="a98ff-226">[Data Consistency Primer][data-consistency-guidance]</span></span>
- <span data-ttu-id="a98ff-227">[Antimodèle de récupération superflue][extraneous-fetching]</span><span class="sxs-lookup"><span data-stu-id="a98ff-227">[Extraneous Fetching antipattern][extraneous-fetching]</span></span>
- <span data-ttu-id="a98ff-228">[Absence d’antimodèle de mise en cache][no-cache]</span><span class="sxs-lookup"><span data-stu-id="a98ff-228">[No Caching antipattern][no-cache]</span></span>

[api-design]: ../../best-practices/api-design.md
[caching-guidance]: ../../best-practices/caching.md
[code-sample]:  https://github.com/mspnp/performance-optimization/tree/master/ChattyIO
[data-consistency-guidance]: http://https://msdn.microsoft.com/library/dn589800.aspx
[ef]: /ef/
[extraneous-fetching]: ../extraneous-fetching/index.md
[new-relic]: https://newrelic.com/application-monitoring
[no-cache]: ../no-caching/index.md

[key-indicators-chatty-io]: _images/ChattyIO.jpg
[key-indicators-chunky-io]: _images/ChunkyIO.jpg
[databasetraffic]: _images/DatabaseTraffic.jpg
[databasetraffic2]: _images/DatabaseTraffic2.jpg
[queries]: _images/DatabaseQueries.jpg
[queries2]: _images/DatabaseQueries2.jpg
[queries3]: _images/DatabaseQueries3.jpg
[queries4]: _images/DatabaseQueries4.jpg

