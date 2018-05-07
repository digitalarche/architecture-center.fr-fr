# <a name="data-lakes"></a><span data-ttu-id="8d274-101">Lacs de données</span><span class="sxs-lookup"><span data-stu-id="8d274-101">Data lakes</span></span>

<span data-ttu-id="8d274-102">Un lac de données est un référentiel de stockage qui contient une grande quantité de données au format natif, brut.</span><span class="sxs-lookup"><span data-stu-id="8d274-102">A data lake is a storage repository that holds a large amount of data in its native, raw format.</span></span> <span data-ttu-id="8d274-103">Les lacs de données sont optimisés pour s’adapter à des téraoctets et pétaoctets de données.</span><span class="sxs-lookup"><span data-stu-id="8d274-103">Data lake stores are optimized for scaling to terabytes and petabytes of data.</span></span> <span data-ttu-id="8d274-104">En général, les données proviennent de plusieurs sources hétérogènes et peuvent être structurées, semi-structurées ou non structurées.</span><span class="sxs-lookup"><span data-stu-id="8d274-104">The data typically comes from multiple heterogeneous sources, and may be structured, semi-structured, or unstructured.</span></span> <span data-ttu-id="8d274-105">L’idée est de stocker des éléments dans leur état d’origine sans leur faire subir de transformation.</span><span class="sxs-lookup"><span data-stu-id="8d274-105">The idea with a data lake is to store everything in its original, untransformed state.</span></span> <span data-ttu-id="8d274-106">Cette approche diffère d’un [entrepôt de données](../relational-data/data-warehousing.md) classique, qui transforme et traite les données au moment de l’ingestion.</span><span class="sxs-lookup"><span data-stu-id="8d274-106">This approach differs from a traditional [data warehouse](../relational-data/data-warehousing.md), which transforms and processes the data at the time of ingestion.</span></span>

<span data-ttu-id="8d274-107">Avantages d’un lac de données :</span><span class="sxs-lookup"><span data-stu-id="8d274-107">Advantages of a data lake:</span></span>

- <span data-ttu-id="8d274-108">Les données ne sont jamais rejetées, car elles sont stockées dans leur format brut.</span><span class="sxs-lookup"><span data-stu-id="8d274-108">Data is never thrown away, because the data is stored in its raw format.</span></span> <span data-ttu-id="8d274-109">Cela est particulièrement utile dans un environnement de données volumineux, lorsque vous ne savez pas à l’avance quelles informations sont disponibles à partir des données.</span><span class="sxs-lookup"><span data-stu-id="8d274-109">This is especially useful in a big data environment, when you may not know in advance what insights are available from the data.</span></span>
- <span data-ttu-id="8d274-110">Les utilisateurs peuvent explorer les données et créer leurs propres requêtes.</span><span class="sxs-lookup"><span data-stu-id="8d274-110">Users can explore the data and create their own queries.</span></span>
- <span data-ttu-id="8d274-111">Peut être plus rapide que les outils ETL traditionnels.</span><span class="sxs-lookup"><span data-stu-id="8d274-111">May be faster than traditional ETL tools.</span></span>
- <span data-ttu-id="8d274-112">Plus flexible qu’un entrepôt de données, car il peut stocker des données non structurées et semi-structurées.</span><span class="sxs-lookup"><span data-stu-id="8d274-112">More flexible than a data warehouse, because it can store unstructured and semi-structured data.</span></span> 

<span data-ttu-id="8d274-113">Une solution Data Lake complète regroupe à la fois le stockage et le traitement.</span><span class="sxs-lookup"><span data-stu-id="8d274-113">A complete data lake solution consists of both storage and processing.</span></span> <span data-ttu-id="8d274-114">Le stockage Data Lake est conçu pour la tolérance aux pannes, l’évolutivité infinie et l’ingestion à débit élevé de données de différentes formes et tailles.</span><span class="sxs-lookup"><span data-stu-id="8d274-114">Data lake storage is designed for fault-tolerance, infinite scalability, and high-throughput ingestion of data with varying shapes and sizes.</span></span> <span data-ttu-id="8d274-115">Le traitement Date Lake implique un ou plusieurs moteurs de traitement configurés pour ces objectifs et qui peuvent fonctionner sur des données stockées dans un volume Data Lake à grande échelle.</span><span class="sxs-lookup"><span data-stu-id="8d274-115">Data lake processing involves one or more processing engines built with these goals in mind, and can operate on data stored in a data lake at scale.</span></span>

## <a name="when-to-use-a-data-lake"></a><span data-ttu-id="8d274-116">Quand utiliser un lac de données</span><span class="sxs-lookup"><span data-stu-id="8d274-116">When to use a data lake</span></span>

<span data-ttu-id="8d274-117">Les utilisations courantes d’un lac de données incluent [l’exploration de données](./interactive-data-exploration.md), l’analyse de données et l’apprentissage automatique.</span><span class="sxs-lookup"><span data-stu-id="8d274-117">Typical uses for a data lake include [data exploration](./interactive-data-exploration.md), data analytics, and machine learning.</span></span> 

<span data-ttu-id="8d274-118">Un lac de données peut également agir en tant que source de données pour un entrepôt de données.</span><span class="sxs-lookup"><span data-stu-id="8d274-118">A data lake can also act as the data source for a data warehouse.</span></span> <span data-ttu-id="8d274-119">Selon cette approche, les données brutes sont ingérées dans le lac de données, puis converties dans un format structuré utilisable dans une requête.</span><span class="sxs-lookup"><span data-stu-id="8d274-119">With this approach, the raw data is ingested into the data lake and then transformed into a structured queryable format.</span></span> <span data-ttu-id="8d274-120">En général, cette transformation utilise un pipeline [ELT](../relational-data/etl.md#extract-load-and-transform-elt) (extraction - chargement - transformation), où les données sont ingérées et transformées sur place.</span><span class="sxs-lookup"><span data-stu-id="8d274-120">Typically this transformation uses an [ELT](../relational-data/etl.md#extract-load-and-transform-elt) (extract-load-transform) pipeline, where the data is ingested and transformed in place.</span></span> <span data-ttu-id="8d274-121">Les données source déjà relationnelles peuvent aller directement dans l’entrepôt de données à l’aide d’un processus ETL, sans passer par le lac de données.</span><span class="sxs-lookup"><span data-stu-id="8d274-121">Source data that is already relational may go directly into the data warehouse, using an ETL process, skipping the data lake.</span></span>

<span data-ttu-id="8d274-122">Les magasins Data Lake sont souvent utilisés pour la diffusion en continu d’événements ou dans des scénarios IoT, car ils permettent de conserver de grandes quantités de données relationnelles et non relationnelles sans transformation ou définition de schéma.</span><span class="sxs-lookup"><span data-stu-id="8d274-122">Data lake stores are often used in event streaming or IoT scenarios, because they can persist large amounts of relational and nonrelational data without transformation or schema definition.</span></span> <span data-ttu-id="8d274-123">Ils sont conçus pour gérer de grands volumes de petites écritures à faible latence, et sont optimisés pour un débit important.</span><span class="sxs-lookup"><span data-stu-id="8d274-123">They are built to handle high volumes of small writes at low latency, and are optimized for massive throughput.</span></span>

## <a name="challenges"></a><span data-ttu-id="8d274-124">Défis</span><span class="sxs-lookup"><span data-stu-id="8d274-124">Challenges</span></span>

- <span data-ttu-id="8d274-125">L’absence de schéma ou de métadonnées descriptives peut compliquer l’utilisation ou l’interrogation des données.</span><span class="sxs-lookup"><span data-stu-id="8d274-125">Lack of a schema or descriptive metadata can make the data hard to consume or query.</span></span>
- <span data-ttu-id="8d274-126">Le manque de cohérence sémantique au niveau des données peut rendre difficile l’analyse des données, à moins que les utilisateurs soient des experts en la matière.</span><span class="sxs-lookup"><span data-stu-id="8d274-126">Lack of semantic consistency across the data can make it challenging to perform analysis on the data, unless users are highly skilled at data analytics.</span></span>
- <span data-ttu-id="8d274-127">Il peut être difficile de garantir la qualité des données acheminées vers le lac de données.</span><span class="sxs-lookup"><span data-stu-id="8d274-127">It can be hard to guarantee the quality of the data going into the data lake.</span></span> 
- <span data-ttu-id="8d274-128">Sans une gouvernance appropriée, les questions de confidentialité et de contrôle d’accès peuvent être problématiques.</span><span class="sxs-lookup"><span data-stu-id="8d274-128">Without proper governance, access control and privacy issues can be problems.</span></span> <span data-ttu-id="8d274-129">Quelles informations sont transmises au lac de données, qui peut avoir accès à ces données et dans quel but ?</span><span class="sxs-lookup"><span data-stu-id="8d274-129">What information is going into the data lake, who can access that data, and for what uses?</span></span>
- <span data-ttu-id="8d274-130">Un lac de données n’est peut-être pas la meilleure solution pour intégrer des données déjà relationnelles.</span><span class="sxs-lookup"><span data-stu-id="8d274-130">A data lake may not be the best way to integrate data that is already relational.</span></span>
- <span data-ttu-id="8d274-131">En soi, un lac de données n’offre pas de vues intégrées ou globales au sein de l’organisation.</span><span class="sxs-lookup"><span data-stu-id="8d274-131">By itself, a data lake does not provide integrated or holistic views across the organization.</span></span> 
- <span data-ttu-id="8d274-132">Un lac de données peut devenir un déversoir de données qui ne sont en réalité jamais analysées ou extraites pour obtenir des informations.</span><span class="sxs-lookup"><span data-stu-id="8d274-132">A data lake may become a dumping ground for data that is never actually analyzed or mined for insights.</span></span>

## <a name="relevant-azure-services"></a><span data-ttu-id="8d274-133">Services Azure appropriés</span><span class="sxs-lookup"><span data-stu-id="8d274-133">Relevant Azure services</span></span>

- <span data-ttu-id="8d274-134">[Data Lake Store](/azure/data-lake-store/) est un référentiel hyperscale compatible Hadoop.</span><span class="sxs-lookup"><span data-stu-id="8d274-134">[Data Lake Store](/azure/data-lake-store/) is a hyper-scale, Hadoop-compatible repository.</span></span>
- <span data-ttu-id="8d274-135">[Azure Data Lake Analytics](/azure/data-lake-analytics/) est un service de travaux d’analyse à la demande, qui simplifie l’analyse du Big Data.</span><span class="sxs-lookup"><span data-stu-id="8d274-135">[Data Lake Analytics](/azure/data-lake-analytics/) is an on-demand analytics job service to simplify big data analytics.</span></span>
