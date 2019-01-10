# <a name="data-lakes"></a>Lacs de données

Un lac de données est un référentiel de stockage qui contient une grande quantité de données au format natif, brut. Les lacs de données sont optimisés pour s’adapter à des téraoctets et pétaoctets de données. En général, les données proviennent de plusieurs sources hétérogènes et peuvent être structurées, semi-structurées ou non structurées. L’idée est de stocker des éléments dans leur état d’origine sans leur faire subir de transformation. Cette approche diffère d’un [entrepôt de données](../relational-data/data-warehousing.md) classique, qui transforme et traite les données au moment de l’ingestion.

Avantages d’un lac de données :

- Les données ne sont jamais rejetées, car elles sont stockées dans leur format brut. Cela est particulièrement utile dans un environnement de données volumineux, lorsque vous ne savez pas à l’avance quelles informations sont disponibles à partir des données.
- Les utilisateurs peuvent explorer les données et créer leurs propres requêtes.
- Peut être plus rapide que les outils ETL traditionnels.
- Plus flexible qu’un entrepôt de données, car il peut stocker des données non structurées et semi-structurées.

Une solution Data Lake complète regroupe à la fois le stockage et le traitement. Le stockage Data Lake est conçu pour la tolérance aux pannes, l’évolutivité infinie et l’ingestion à débit élevé de données de différentes formes et tailles. Le traitement Date Lake implique un ou plusieurs moteurs de traitement configurés pour ces objectifs et qui peuvent fonctionner sur des données stockées dans un volume Data Lake à grande échelle.

## <a name="when-to-use-a-data-lake"></a>Quand utiliser un lac de données

Les utilisations courantes d’un lac de données incluent [l’exploration de données](./interactive-data-exploration.md), l’analyse de données et l’apprentissage automatique.

Un lac de données peut également agir en tant que source de données pour un entrepôt de données. Selon cette approche, les données brutes sont ingérées dans le lac de données, puis converties dans un format structuré utilisable dans une requête. En général, cette transformation utilise un pipeline [ELT](../relational-data/etl.md#extract-load-and-transform-elt) (extraction - chargement - transformation), où les données sont ingérées et transformées sur place. Les données source déjà relationnelles peuvent aller directement dans l’entrepôt de données à l’aide d’un processus ETL, sans passer par le lac de données.

Les magasins Data Lake sont souvent utilisés pour la diffusion en continu d’événements ou dans des scénarios IoT, car ils permettent de conserver de grandes quantités de données relationnelles et non relationnelles sans transformation ou définition de schéma. Ils sont conçus pour gérer de grands volumes de petites écritures à faible latence, et sont optimisés pour un débit important.

## <a name="challenges"></a>Défis

- L’absence de schéma ou de métadonnées descriptives peut compliquer l’utilisation ou l’interrogation des données.
- Le manque de cohérence sémantique au niveau des données peut rendre difficile l’analyse des données, à moins que les utilisateurs soient des experts en la matière.
- Il peut être difficile de garantir la qualité des données acheminées vers le lac de données.
- Sans une gouvernance appropriée, les questions de confidentialité et de contrôle d’accès peuvent être problématiques. Quelles informations sont transmises au lac de données, qui peut avoir accès à ces données et dans quel but ?
- Un lac de données n’est peut-être pas la meilleure solution pour intégrer des données déjà relationnelles.
- En soi, un lac de données n’offre pas de vues intégrées ou globales au sein de l’organisation.
- Un lac de données peut devenir un déversoir de données qui ne sont en réalité jamais analysées ou extraites pour obtenir des informations.

## <a name="relevant-azure-services"></a>Services Azure appropriés

- [Data Lake Store](/azure/data-lake-store/) est un référentiel hyperscale compatible Hadoop.
- [Azure Data Lake Analytics](/azure/data-lake-analytics/) est un service de travaux d’analyse à la demande, qui simplifie l’analyse du Big Data.
