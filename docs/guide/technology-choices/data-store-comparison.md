---
title: Critères de sélection d’une banque de données
titleSuffix: Azure Application Architecture Guide
description: Vue d’ensemble des options de calcul Azure.
author: MikeWasson
ms.date: 06/01/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: f7c19501b9f28db3892285b5f35a33f02edd87ab
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54488135"
---
# <a name="criteria-for-choosing-a-data-store"></a>Critères de sélection d’une banque de données

Azure prend en charge divers types de solutions de stockage de données, chacune offrant différentes fonctionnalités et capacités. Cet article décrit les critères de comparaison que vous devez employer pour évaluer une banque de données. L’objectif ici est de vous aider à identifier les types de stockage de données qui vous permettront de répondre aux besoins de votre solution.

## <a name="general-considerations"></a>Considérations d’ordre général

Pour débuter votre comparaison, rassemblez le maximum d’informations ci-dessous pour définir vos besoins en données. Ces informations vous aideront à identifier les types de stockage de données qui répondront à vos besoins.

### <a name="functional-requirements"></a>Spécifications fonctionnelles

- **Format de données** : quel type de données prévoyez-vous de stocker ? Les données transactionnelles, les objets JSON, les données de télémétrie, les index de recherche et les fichiers plats comptent parmi les types courants.

- **Taille des données** : quelle est la taille des entités que vous avez besoin de stocker ? Ces entités devront-elles être conservées dans un document unique ou peuvent-elles être scindées en plusieurs documents, tables, collections, etc ?

- **Échelle et structure** : de quelle capacité de stockage globale avez-vous besoin ? Prévoyez-vous de partitionner vos données ?

- **Relations entre les données** : vos données devront-elles prendre en charge des relations un-à-plusieurs ou plusieurs à plusieurs ? Les relations proprement dites constituent-elles une part importante des données ? Prévoyez-vous de joindre ou de combiner les données d’un même jeu de données ou de plusieurs jeux de données externes ?

- **Modèle de cohérence** : quelle importance accordez-vous à ce que les mises à jour effectuées dans un nœud apparaissent dans les autres nœuds, avant que d’autres modifications puissent avoir lieu ? Pouvez-vous accepter la cohérence éventuelle ? Avez-vous besoin de garanties ACID pour les transactions ?

- **Flexibilité de schéma** : quel type de schéma appliquerez-vous à vos données ? Allez-vous utiliser un schéma fixe, une approche de schéma à l’écriture ou une approche de schéma à la lecture ?

- **Accès concurrentiel**. quel type de mécanisme d’accès concurrentiel souhaitez-vous utiliser durant la mise à jour et la synchronisation des données ? L’application sera-t-elle appelée à procéder à de nombreuses mises à jour qui pourraient potentiellement entrer en conflit ? Dans ce cas, vous aurez peut-être besoin du verrouillage des enregistrements et du contrôle d’accès concurrentiel pessimiste. Sinon, pouvez vous prendre en charge les contrôles d’accès concurrentiel optimiste ? Dans ce cas, le contrôle d’accès concurrentiel basé sur l’horodatage est-il suffisant ou avez-vous besoin de l’ajout de la fonctionnalité de contrôle d’accès concurrentiel multiversion ?

- **Déplacement des données** : votre solution devra-t-elle effectuer des tâches ETL pour déplacer les données vers d’autres banques ou entrepôts de données ?

- **Cycle de vie des données** : les données sont-elles écrites une fois et lues autant de fois que souhaité ? Peuvent-elles être déplacées dans un espace de stockage froid ?

- **Autres fonctionnalités prises en charge** : avez-vous besoin d’autres fonctionnalités spécifiques, telles que la validation de schéma, l’agrégation, l’indexation, la recherche en texte intégral, MapReduce ou d’autres fonctions de requête ?

### <a name="non-functional-requirements"></a>Spécifications non fonctionnelles

- **Performances et scalabilité** : quels sont vos besoins en matière de performances de données ? Avez-vous des exigences spécifiques concernant la vitesse d’ingestion des données et la vitesse de traitement des données ? Quels sont les temps de réponse acceptables pour l’interrogation et l’agrégation des données une fois celles-ci ingérées ? De quelle taille aurez-vous besoin pour augmenter les capacités de la banque de données ? Votre charge de travail est-elle plutôt axée sur la lecture ou sur l’écriture ?

- **Fiabilité** : quel contrat SLA global devez-vous prendre en charge ? Quel niveau de tolérance aux pannes devez-vous offrir aux consommateurs de données ? De quelles fonctionnalités de sauvegarde et de restauration avez-vous besoin ?

- **Réplication** : vos données devront-elles être réparties entre plusieurs réplicas ou régions ? De quel type de fonctionnalités de réplication de données avez-vous besoin ?

- **Limites** : les limites d’une banque de données déterminée répondent-elles à vos besoins en matière d’échelle, de nombre de connexions et de débit ?

### <a name="management-and-cost"></a>Gestion et coût

- **Service managé** : si possible, utilisez un service de données managé, sauf si vous avez besoin de fonctionnalités spécifiques que seule une banque de données hébergée par IaaS peut offrir.

- **Disponibilité dans les régions** : pour les services managés, le service est-il disponible dans toutes les régions Azure ? Votre solution doit-elle être hébergée dans certaines régions Azure ?

- **Portabilité** : vos données devront-elles être migrées localement, vers des centres de données externes ou vers d’autres environnements d’hébergement cloud ?

- **Licences** : avez-vous une préférence quant au type de licence : propriétaire ou OSS ? Existe-t-il d’autres restrictions externes quant au type de licence que vous pouvez utiliser ?

- **Coût global** : quel est le coût global d’utilisation du service dans votre solution ? Combien d’instances devront s’exécuter pour répondre à vos besoins en matière de durée de fonctionnement et de débit ? Prenez en compte les coûts d’exploitation dans ce calcul. L’un des raisons de préférer les services managés est leur coût d’exploitation réduit.

- **Rentabilité** : pouvez-vous partitionner vos données de façon à profiter d’un stockage plus économique ? Par exemple, pouvez-vous transférer les objets volumineux d’une base de données relationnelle coûteuse vers une banque d’objets ?

### <a name="security"></a>Sécurité

- **Sécurité**. de quel type de chiffrement avez-vous besoin ? Avez-vous besoin d’un chiffrement au repos ? Quel mécanisme d’authentification voulez-vous utiliser pour vous connecter à vos données ?

- **Audit** : quel type de journal d’audit avez-vous besoin de générer ?

- **Spécifications réseau** : devez-vous limiter ou gérer l’accès à vos données à partir d’autres ressources réseau ? Les données ne doivent-elle être accessibles qu’à l’intérieur de l’environnement Azure ? Les données doivent-elle être accessibles à partir d’adresses IP ou de sous-réseaux spécifiques ? Doivent-elles être accessibles à partir d’applications ou de services hébergés localement ou dans d’autres centres de données externes ?

### <a name="devops"></a>DevOps

- **Ensemble de compétences** : votre équipe est-elle encline à utiliser certains langages de programmation, systèmes d’exploitation et autres technologies plutôt que d’autres ? Votre équipe aurait des difficultés à en utiliser d’autres ?

- **Clients** : existe-t-il un bon support client pour vos langages de développement?

Les sections suivantes comparent les différents modèles de banque de données en termes de profil de charge de travail, de types de données et d’exemples de cas d’usage.

## <a name="relational-database-management-systems-rdbms"></a>Systèmes de gestion de base de données relationnelle (SGBDR)

<!-- markdownlint-disable MD033 -->

<table>
<tr><td><strong>Charge de travail</strong></td>
    <td>
        <ul>
            <li>La création d’enregistrements et la mise à jour des données existantes se produisent régulièrement.</li>
            <li>Plusieurs opérations doivent être effectuées dans une même transaction.</li>
            <li>Nécessite des fonctions d’agrégation pour effectuer une tabulation croisée.</li>
            <li>Exige une forte intégration avec des outils de création de rapports.</li>
            <li>Les relations sont mises en œuvre en utilisant des contraintes de base de données.</li>
            <li>Des index sont utilisés pour optimiser les performances des requêtes.</li>
            <li>Autorise l’accès à des sous-ensembles spécifiques de données.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Type de données</strong></td>
    <td>
        <ul>
            <li>Les données sont très normalisées.</li>
            <li>Des schémas de base de données sont exigés et appliqués.</li>
            <li>Relations plusieurs à plusieurs entre les entités de données de la base de données.</li>
            <li>Des contraintes sont définies dans le schéma et imposées aux données de la base de données.</li>
            <li>Les données nécessitent un haut niveau d’intégrité. Les index et les relations doivent être gérés avec précision.</li>
            <li>Les données nécessitent une cohérence forte. Les transactions s’exécutent de façon à assurer une cohérence à 100 % de toutes les données pour l’ensemble des utilisateurs et des processus.</li>
            <li>Les entrées de données individuelles sont censées être d’une taille petite à moyenne.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Exemples</strong></td>
    <td>
        <ul>
            <li>Applications métier (gestion du capital humain, gestion de la relation client, planification des ressources d’entreprise)</li>
            <li>Gestion des stocks</li>
            <li>Base de données de création de rapports</li>
            <li>Comptabilité</li>
            <li>Gestion des ressources</li>
            <li>Gestion des fonds</li>
            <li>Gestion des commandes</li>
        </ul>
    </td>
</tr>
</table>

## <a name="document-databases"></a>Bases de données de documents

<table>
<tr><td><strong>Charge de travail</strong></td>
    <td>
        <ul>
            <li>Usage général.</li>
            <li>Les opérations d’insertion et de mise à jour sont courantes. La création d’enregistrements et la mise à jour des données existantes se produisent régulièrement.</li>
            <li>Pas d’anomalie d’impédance objet-relationnel. Les documents peuvent mieux correspondre aux structures d’objet utilisées dans le code d’application.</li>
            <li>L’accès concurrentiel optimiste est plus couramment utilisé.</li>
            <li>Les données doivent être modifiées et traitées par l’application consommatrice.</li>
            <li>Les données exigent la présence d’un index sur plusieurs champs.</li>
            <li>Les documents individuels sont récupérés et écrits en un seul bloc.</li>
    </td>
</tr>
<tr><td><strong>Type de données</strong></td>
    <td>
        <ul>
            <li>Les données peuvent être gérées de façon dénormalisée.</li>
            <li>Les données des documents individuels sont relativement peu volumineuses.</li>
            <li>Chaque type de document peut utiliser son propre schéma.</li>
            <li>Les documents peuvent inclure des champs facultatifs.</li>
            <li>Les données des documents sont semi-structurées, ce qui signifie que les types de données de chaque champ ne sont pas strictement définis.</li>
            <li>L’agrégation de données est prise en charge.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Exemples</strong></td>
    <td>
        <ul>
            <li>Catalogue produits</li>
            <li>Comptes d'utilisateurs</li>
            <li>Nomenclature</li>
            <li>Personnalisation</li>
            <li>Gestion de contenu</li>
            <li>Données d’exploitation</li>
            <li>Gestion des stocks</li>
            <li>Données d’historique des transactions</li>
            <li>Vue matérialisée d’autres banques NoSQL. Remplace l’indexation de fichiers/objets blob.</li>
        </ul>
    </td>
</tr>
</table>

## <a name="keyvalue-stores"></a>Magasins de clés/valeurs

<table>
<tr><td><strong>Charge de travail</strong></td>
    <td>
        <ul>
            <li>L’identification des données et l’accès à celles-ci se fait au moyen d’une clé d’ID unique, comme un dictionnaire.</li>
            <li>Scalabilité importante.</li>
            <li>Utilisation de jointures, de verrous ou d’unions non obligatoire.</li>
            <li>Aucun mécanisme d’agrégation n’est utilisé.</li>
            <li>Les index secondaires ne sont généralement pas utilisés.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Type de données</strong></td>
    <td>
        <ul>
            <li>Les données ont tendance à être volumineuses.</li>
            <li>Chaque clé est associée à une valeur unique, qui est un objet blob de données non managé.</li>
            <li>Absence d’application de schéma.</li>
            <li>Absence de relations entre les entités.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Exemples</strong></td>
    <td>
        <ul>
            <li>Mise en cache des données</li>
            <li>Gestion des sessions</li>
            <li>Gestion des préférences et des profils utilisateur</li>
            <li>Recommandation de produits et affichage d’annonces</li>
            <li>Dictionnaires</li>
        </ul>
    </td>
</tr>
</table>

## <a name="graph-databases"></a>Bases de données de graphiques

<table>
<tr><td><strong>Charge de travail</strong></td>
    <td>
        <ul>
            <li>Les relations entre les éléments de données sont très complexes, impliquant de nombreux sauts entre les éléments de données associés.</li>
            <li>Les relations entre les éléments de données sont dynamiques et changent dans le temps.</li>
            <li>Les relations entre les objets sont des entités de première classe, sans qu’il soit nécessaire de traverser des clés étrangères et des jointures.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Type de données</strong></td>
    <td>
        <ul>
            <li>Les données sont constituées de nœuds et de relations.</li>
            <li>Les nœuds s’apparentent à des lignes de table ou à des documents JSON.</li>
            <li>Les relations sont tout aussi importantes que les nœuds et sont directement exposées dans le langage de requête.</li>
            <li>Les objets composites, tels qu’une personne ayant plusieurs numéros de téléphone, ont tendance à être divisés en nœuds distincts plus petits, en association avec des relations qui peuvent être traversées. </li>
        </ul>
    </td>
</tr>
<tr><td><strong>Exemples</strong></td>
    <td>
        <ul>
            <li>Organigrammes</li>
            <li>Graphiques de réseau sociaux</li>
            <li>Détection des fraudes</li>
            <li>Analytics</li>
            <li>Moteurs de recommandation</li>
        </ul>
    </td>
</tr>
</table>

## <a name="column-family-databases"></a>Bases de données de familles de colonnes

<table>
<tr><td><strong>Charge de travail</strong></td>
    <td>
        <ul>
            <li>La plupart des bases de données de familles de colonnes effectuent les opérations d’écriture de manière extrêmement rapide.</li>
            <li>Les opérations de mise à jour et de suppression sont rares.</li>
            <li>Conçu pour offrir un débit élevé et un accès à faible latence.</li>
            <li>Permet aux requêtes d’accéder facilement à un ensemble particulier de champs au sein d’un enregistrement bien plus volumineux.</li>
            <li>Scalabilité importante.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Type de données</strong></td>
    <td>
        <ul>
            <li>Les données sont stockées dans des tables comprenant une colonne clé et une ou plusieurs familles de colonnes.</li>
            <li>Certaines colonnes peuvent varier par lignes individuelles.</li>
            <li>Les cellules individuelles sont accessibles via des commandes get et put.</li>
            <li>Plusieurs lignes sont retournées en utilisant une commande scan.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Exemples</strong></td>
    <td>
        <ul>
            <li>Recommandations</li>
            <li>Personnalisation</li>
            <li>données de capteur</li>
            <li>Télémétrie</li>
            <li>Messagerie</li>
            <li>Analytique des réseaux sociaux</li>
            <li>Analytique web</li>
            <li>Surveillance de l’activité</li>
            <li>Météo et autres données de séries chronologiques</li>
        </ul>
    </td>
</tr>
</table>

## <a name="search-engine-databases"></a>Bases de données de moteur de recherche

<table>
<tr><td><strong>Charge de travail</strong></td>
    <td>
        <ul>
            <li>Indexation de données de plusieurs sources et services.</li>
            <li>Les requêtes sont de type ad hoc et peuvent être complexes.</li>
            <li>Nécessite une agrégation.</li>
            <li>La recherche en texte intégral est obligatoire.</li>
            <li>Nécessite une requête en libre service ad hoc.</li>
            <li>Nécessite une analyse des données avec un index sur tous les champs.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Type de données</strong></td>
    <td>
        <ul>
            <li>Semi-structurées ou non structurées</li>
            <li>Texte</li>
            <li>Texte avec référence à des données structurées</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Exemples</strong></td>
    <td>
        <ul>
            <li>Catalogue produits</li>
            <li>Recherche sur site</li>
            <li>Journalisation</li>
            <li>Analytics</li>
            <li>Sites marchands</li>
        </ul>
    </td>
</tr>
</table>

## <a name="data-warehouse"></a>Entrepôt de données

<table>
<tr><td><strong>Charge de travail</strong></td>
    <td>
        <ul>
            <li>Analyse de données</li>
            <li>Décisionnel d’entreprise</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Type de données</strong></td>
    <td>
        <ul>
            <li>Données d’historique de plusieurs sources.</li>
            <li>Généralement dénormalisé dans un schéma &quot;en étoile&quot; ou &quot;en flocon&quot;, constitué de tables de faits et de dimensions.</li>
            <li>Généralement chargé de façon régulière avec de nouvelles données.</li>
            <li>Les tables de dimension comprennent souvent plusieurs versions historiques d’une entité, appelée <em>dimension à variation lente</em>.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Exemples</strong></td>
    <td>Entrepôt de données d’entreprise fournissant des données pour des modèles, rapports et tableaux de bord analytiques.
    </td>
</tr>
</table>

## <a name="time-series-databases"></a>Bases de données de séries chronologiques

<table>
<tr><td><strong>Charge de travail</strong></td>
    <td>
        <ul>
            <li>Les opérations sont dans une très large proportion (95-99 %) des écritures.</li>
            <li>Les enregistrements sont généralement ajoutés de manière séquentielle dans un ordre chronologique.</li>
            <li>Les mises à jour sont rares.</li>
            <li>Les suppressions se produisent en masse et concernent des blocs ou des enregistrements contigus.</li>
            <li>La taille des demandes de lecture peut dépasser la taille de mémoire disponible.</li>
            <li>Il est courant que plusieurs lectures se produisent simultanément.</li>
            <li>Les données sont lues de manière séquentielle dans un ordre chronologique croissant ou décroissant.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Type de données</strong></td>
    <td>
        <ul>
            <li>Horodatage utilisé comme clé primaire et mécanisme de tri.</li>
            <li>Mesures de l’entrée ou descriptions de ce que l’entrée représente.</li>
            <li>Balises qui définissent des informations supplémentaires sur le type, l’origine et d’autres informations sur l’entrée.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Exemples</strong></td>
    <td>
        <ul>
            <li>Surveillance et données de télémétrie des événements.</li>
            <li>Données de capteur ou autres données IoT.</li>
        </ul>
    </td>
</tr>
</table>

## <a name="object-storage"></a>Stockage d’objets

<table>
<tr><td><strong>Charge de travail</strong></td>
    <td>
        <ul>
            <li>Identifié par la clé.</li>
            <li>Les objets peuvent offrir un accès public ou privé.</li>
            <li>Le contenu est généralement une ressource, par exemple une feuille de calcul, une image ou un fichier vidéo.</li>
            <li>Le contenu doit être durable (persistant) et extérieur à une couche Application ou à une machine virtuelle.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Type de données</strong></td>
    <td>
        <ul>
            <li>Données volumineuses.</li>
            <li>Données blob.</li>
            <li>Valeur opaque.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Exemples</strong></td>
    <td>
        <ul>
            <li>Images, vidéos, documents Office, PDF</li>
            <li>CSS, scripts, CSV</li>
            <li>HTML statique, JSON</li>
            <li>Fichiers journaux et d’audit</li>
            <li>Sauvegardes de base de données</li>
        </ul>
    </td>
</tr>
</table>

## <a name="shared-files"></a>Fichiers partagés

<table>
<tr><td><strong>Charge de travail</strong></td>
    <td>
        <ul>
            <li>Migration à partir d’applications existantes qui interagissent avec le système de fichiers.</li>
            <li>Nécessite une interface SMB.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Type de données</strong></td>
    <td>
        <ul>
            <li>Fichiers figurant dans un ensemble hiérarchique de dossiers.</li>
            <li>Accessible avec des bibliothèques d’E/S standard.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Exemples</strong></td>
    <td>
        <ul>
            <li>Fichiers hérités</li>
            <li>Contenu partagé accessible à un certain nombre de machines virtuelles ou d’instances d’application</li>
        </ul>
    </td>
</tr>
</table>

<!-- markdownlint-enable MD033 -->
