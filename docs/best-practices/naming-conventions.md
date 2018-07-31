---
title: Conventions d’affectation de noms pour les ressources Azure
description: Conventions d’affectation de noms pour les ressources Azure. Comment nommer les machines virtuelles, comptes de stockage, réseaux, réseaux virtuels, sous-réseaux et autres entités Azure
author: telmosampaio
ms.date: 05/18/2017
pnp.series.title: Best Practices
ms.openlocfilehash: 6ad71a5ee39b8f1863c51dae0120dbdc7baf1f76
ms.sourcegitcommit: c704d5d51c8f9bbab26465941ddcf267040a8459
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/24/2018
ms.locfileid: "39229148"
---
# <a name="naming-conventions"></a>Conventions d’affectation de noms

[!INCLUDE [header](../_includes/header.md)]

Cet article récapitule les règles et restrictions d’affectation de noms relatives aux ressources Azure et fournit un ensemble de recommandations de référence concernant les conventions d’affectation de noms.  Vous pouvez utiliser ces recommandations comme point de départ pour élaborer vos propres conventions adaptées à vos besoins spécifiques.

Le choix du nom de chaque ressource dans Microsoft Azure est essentiel à plusieurs titres :

* Il est difficile de modifier un nom par la suite.
* Les noms doivent répondre aux exigences du type de ressource correspondant.

Des conventions d’affectation de noms cohérentes simplifient la localisation des ressources. Elles peuvent également indiquer le rôle d’une ressource dans une solution.

Pour que les conventions d’affectation de noms soient efficaces, vous devez les établir et les appliquer dans l’ensemble de vos applications et organisations.

## <a name="naming-subscriptions"></a>Affectation de nom aux abonnements
L’affectation de noms détaillés aux abonnements Azure permet de mieux comprendre leur contexte et leur objet.  Lorsque vous travaillez dans un environnement comportant de nombreux abonnements, l’application d’une convention d’affectation de noms partagée permet de gagner en clarté.

À titre d’exemple, nous recommandons le modèle suivant pour nommer des abonnements :

`<Company> <Department (optional)> <Product Line (optional)> <Environment>`

* L’entreprise est généralement la même pour chaque abonnement. Cependant, certaines peuvent comporter des sociétés enfant au sein de la structure organisationnelle. Ces sociétés peuvent être gérées par un groupe informatique centralisé. Dans ces cas-là, il est possible de les différencier en utilisant à la fois le nom de l’entreprise parente (*Contoso*) et le nom de la société enfant (*Northwind*).
* Le service est le nom d’une entité, au sein de l’organisation, qui contient un groupe de personnes. Cet élément dans l’espace de noms est facultatif.
* La gamme de produits est le nom spécifique d’un produit ou d’une fonction réalisé au sein du service. Cet élément est généralement facultatif pour les services et applications internes. En revanche, il est vivement recommandé pour les services destinés au public, qui doivent être faciles à séparer et à identifier (par exemple, pour une séparation claire des enregistrements de facturation).
* L’environnement est le nom qui décrit le cycle de vie de déploiement des applications ou services, par exemple, Dev, AQ ou Prod.

| Company | Department | Gamme de produits ou service | Environnement | Nom complet |
| --- | --- | --- | --- | --- |
| Contoso |SocialGaming |AwesomeService |Production |Contoso SocialGaming AwesomeService Production |
| Contoso |SocialGaming |AwesomeService |Dev |Contoso SocialGaming AwesomeService Dev |
| Contoso |IT |InternalApps |Production |Contoso IT InternalApps Production |
| Contoso |IT |InternalApps |Dev |Contoso IT InternalApps Dev |

Pour plus d’informations sur la façon d’organiser les abonnements des grandes entreprises, consultez notre article sur la [gouvernance normative de l’abonnement][scaffold].

## <a name="use-affixes-to-avoid-ambiguity"></a>Utiliser des affixes pour éviter les ambiguïtés

Lorsque vous nommez des ressources dans Azure, il est recommandé d’utiliser des préfixes ou des suffixes courants pour identifier le type et le contexte de la ressource.  Alors que toutes les informations sur le type, les métadonnées et le contexte sont disponibles par programme, l’application d’affixes courants simplifie l’identification visuelle.  Lorsque vous incorporez des affixes dans votre convention d’affectation de noms, vous devez indiquer clairement si ces affixes doivent être placés au début du nom (préfixe) ou à la fin (suffixe).

Voici deux exemples de noms possibles pour un service hébergeant un moteur de calcul :

* SvcCalculationEngine (préfixe)
* CalculationEngineSvc (suffixe)

Les affixes peuvent faire référence à différents aspects des ressources spécifiques. Le tableau suivant présente des exemples généralement utilisés.

| Aspect | Exemples | Notes |
| --- | --- | --- |
| Environnement |dev, prod, AQ |Identifie l’environnement de la ressource |
| Lieu |uw (ouest des États-Unis), ue (est des États-Unis) |Identifie la région dans laquelle la ressource est déployée |
| Instance |01, 02 |Pour les ressources possédant plusieurs instances nommées (serveurs web, etc.) |
| Produit ou service |service |Identifie le produit, l’application ou le service pris en charge par la ressource |
| Rôle |sql, web, messagerie |Identifie le rôle de la ressource associée |

Lors du développement d’une convention d’affectation de noms spécifique pour votre entreprise ou vos projets, il est important de choisir un ensemble d’affixes courants et de déterminer leur position (suffixe ou préfixe).

## <a name="naming-rules-and-restrictions"></a>Règles et restrictions d’affectation de noms

Chaque ressource ou type de ressource dans Azure applique un ensemble de restrictions d’affectation de noms et une portée associée ; tous les modèles ou conventions d’affectation de noms doivent adhérer aux règles d’affectation de noms et à la portée.  Par exemple, alors que le nom d’une machine virtuelle est mappé sur un nom DNS (et doit donc être unique dans l’ensemble de l’environnement Azure), la portée du nom d’un réseau virtuel se limite au groupe de ressources dans lequel il est créé.

En règle générale, évitez d’utiliser des caractères spéciaux (`-` ou `_`) comme premier ou dernier caractère d’un nom quel qu’il soit. Ces caractères entraînent l’échec de la plupart des règles de validation.

### <a name="general"></a>Généralités

| Entité | Étendue | Longueur | Casse | Caractères valides | Modèle suggéré | Exemples |
| --- | --- | --- | --- | --- | --- | --- |
|Groupe de ressources |Abonnement |1-90 |Non-respect de la casse |Alphanumériques, trait de soulignement, parenthèses, trait d’union, point (sauf à la fin) et caractères Unicode |`<service short name>-<environment>-rg` |`profx-prod-rg` |
|Groupe à haute disponibilité |Groupe de ressources |1-80 |Non-respect de la casse |Alphanumériques, trait de soulignement et trait d’union |`<service-short-name>-<context>-as` |`profx-sql-as` |
|Tag |Entité associée |512 (nom), 256 (valeur) |Non-respect de la casse |Alphanumérique |`"key" : "value"` |`"department" : "Central IT"` |

### <a name="compute"></a>Calcul

| Entité | Étendue | Longueur | Casse | Caractères valides | Modèle suggéré | Exemples |
| --- | --- | --- | --- | --- | --- | --- |
|Machine virtuelle |Groupe de ressources |1-15 (Windows), 1-64 (Linux) |Non-respect de la casse |Alphanumériques et trait d’union |`<name>-<role>-vm<number>` |`profx-sql-vm1` |
|Function App | Globale |1-60 |Non-respect de la casse |Alphanumériques et trait d’union |`<name>-func` |`calcprofit-func` |

> [!NOTE]
> Les machines virtuelles dans Azure portent deux noms distincts : un nom de machine virtuelle et un nom d’hôte. Lorsque vous créez une machine virtuelle dans le portail, le même nom est utilisé pour le nom d’hôte et pour le nom de ressource de machine virtuelle. Les restrictions ci-dessus s’appliquent au nom d’hôte. Le nom de ressource proprement dit peut comporter jusqu’à 64 caractères.

### <a name="storage"></a>Stockage

| Entité | Étendue | Longueur | Casse | Caractères valides | Modèle suggéré | Exemples |
| --- | --- | --- | --- | --- | --- | --- |
|Nom du compte de stockage (données) |Globale |3-24 |Minuscules |Alphanumérique |`<globally unique name><number>` (utilisez une fonction afin de calculer un GUID unique pour l’affectation de noms aux comptes de stockage) |`profxdata001` |
|Nom du compte de stockage (disques) |Globale |3-24 |Minuscules |Alphanumérique |`<vm name without hyphens>st<number>` |`profxsql001st0` |
| Nom du conteneur |Compte de stockage |3-63 |Minuscules |Alphanumériques et trait d’union |`<context>` |`logs` |
|Nom de l’objet blob | Conteneur |1-1024 |Respect de la casse |Tout caractère d’URL |`<variable based on blob usage>` |`<variable based on blob usage>` |
|Nom de la file d'attente |Compte de stockage |3-63 |Minuscules |Alphanumériques et trait d’union |`<service short name>-<context>-<num>` |`awesomeservice-messages-001` |
|Nom de la table | Compte de stockage |3-63 |Non-respect de la casse |Alphanumérique |`<service short name><context>` |`awesomeservicelogs` |
|Nom de fichier | Compte de stockage |3-63 |Minuscules | Alphanumérique |`<variable based on blob usage>` |`<variable based on blob usage>` |
|Data Lake Store | Globale |3-24 |Minuscules | Alphanumérique |`<name>dls` |`telemetrydls` |

### <a name="networking"></a>Mise en réseau

| Entité | Étendue | Longueur | Casse | Caractères valides | Modèle suggéré | Exemples |
| --- | --- | --- | --- | --- | --- | --- |
|Réseau virtuel (VNet) |Groupe de ressources |2-64 |Non-respect de la casse |Alphanumériques, trait d’union, trait de soulignement et point |`<service short name>-vnet` |`profx-vnet` |
|Sous-réseau |Réseau virtuel parent |2-80 |Non-respect de la casse |Alphanumériques, trait d’union, trait de soulignement et point |`<descriptive context>` |`web` |
|Interface réseau |Groupe de ressources |1-80 |Non-respect de la casse |Alphanumériques, trait d’union, trait de soulignement et point |`<vmname>-nic<num>` |`profx-sql1-nic1` |
|Groupe de sécurité réseau |Groupe de ressources |1-80 |Non-respect de la casse |Alphanumériques, trait d’union, trait de soulignement et point |`<service short name>-<context>-nsg` |`profx-app-nsg` |
|Règle de groupe de sécurité réseau |Groupe de ressources |1-80 |Non-respect de la casse |Alphanumériques, trait d’union, trait de soulignement et point |`<descriptive context>` |`sql-allow` |
|Adresse IP publique |Groupe de ressources |1-80 |Non-respect de la casse |Alphanumériques, trait d’union, trait de soulignement et point |`<vm or service name>-pip` |`profx-sql1-pip` |
|Équilibreur de charge |Groupe de ressources |1-80 |Non-respect de la casse |Alphanumériques, trait d’union, trait de soulignement et point |`<service or role>-lb` |`profx-lb` |
|Configuration des règles d’équilibrage de charge |Équilibreur de charge |1-80 |Non-respect de la casse |Alphanumériques, trait d’union, trait de soulignement et point |`<descriptive context>` |`http` |
|Azure Application Gateway |Groupe de ressources |1-80 |Non-respect de la casse |Alphanumériques, trait d’union, trait de soulignement et point |`<service or role>-agw` |`profx-agw` |
|Profil Traffic Manager |Groupe de ressources |1-63 |Non-respect de la casse |Alphanumériques, trait d’union et point |`<descriptive context>` |`app1` |

### <a name="containers"></a>Containers

| Entité | Étendue | Longueur | Casse | Caractères valides | Modèle suggéré | Exemples |
| --- | --- | --- | --- | --- | --- | --- |
|Container Registry | Globale |5 à 50 |Non-respect de la casse | Alphanumérique |`<service short name>registry` |`app1registry` |


## <a name="organize-resources-with-tags"></a>Organiser les ressources à l’aide de balises

Azure Resource Manager prend en charge le balisage d’entités avec des chaînes de texte arbitraires pour identifier le contexte et simplifier l’automatisation.  Par exemple, la balise `"sqlVersion"="sql2014ee"` peut identifier des machines virtuelles exécutant SQL Server 2014 Enterprise Edition. Les balises doivent être intégrées aux conventions d’affectation de noms choisies pour préciser ou améliorer le contexte.

> [!TIP]
> Elles offrent également l’avantage de s’étendre sur les groupes de ressources, ce qui vous permet de lier et mettre en corrélation les entités dans des environnements disparates.

Chaque ressource ou groupe de ressources peut inclure un maximum de **15** balises. Le nom de balise est limité à 512 caractères, et la valeur de balise à 256 caractères.

Pour plus d’informations sur le balisage des ressources, consultez l’article [Organisation des ressources Azure à l’aide de balises](/azure/azure-resource-manager/resource-group-using-tags/).

Les balises sont notamment utilisées dans les cas suivants :

* **Facturation**; regroupement et association des ressources à des codes de facturation interne ou externe
* **Identification du contexte de service**; identification de groupes parmi les groupes de ressources pour les opérations courantes et à des fins de regroupement
* **Contrôle d’accès et contexte de sécurité** : identification du rôle d’administrateur en fonction du portefeuille, du système, du service, de l’application, de l’instance, etc.

> [!TIP]
> Effectuez un balisage précoce et fréquent.  Il est préférable de mettre en place un système de balisage de base et de l’affiner progressivement, plutôt que d’avoir à effectuer des adaptations a posteriori.

Voici un exemple d’approche de balisage courante :

| Nom de la balise | Clé | Exemples | Commentaire |
| --- | --- | --- | --- |
| Facturer à / ID de facturation interne |billTo |`IT-Chargeback-1234` |Un code d’E/S ou de facturation interne |
| Opérateur ou personne directement responsable |managedBy |`joe@contoso.com` |Alias ou adresse de messagerie |
| Nom du projet |projectName |`myproject` |Nom du projet ou de la gamme de produits |
| Version du projet |projectVersion |`3.4` |Version du projet ou de la gamme de produits |
| Environnement |Environnement |`<Production, Staging, QA >` |Identificateur de l’environnement |
| Niveau |Niveau |`Front End, Back End, Data` |Identification du niveau ou du rôle/contexte |
| Profil de données |dataProfile |`Public, Confidential, Restricted, Internal` |Niveau des données stockées dans la ressource |

## <a name="tips-and-tricks"></a>Astuces et conseils

Certains types de ressources peuvent nécessiter une attention particulière pour l’affectation de noms et les conventions.

### <a name="virtual-machines"></a>Machines virtuelles

En particulier dans les topologies de grande taille, un choix de noms judicieux pour les machines virtuelles simplifie l’identification du rôle et de la finalité de chaque machine, et permet une écriture de scripts plus prévisible.

### <a name="storage-accounts-and-storage-entities"></a>Comptes de stockage et entités de stockage

Il existe deux principaux cas d’utilisation de comptes de stockage : sauvegarde des disques pour les machines virtuelles et stockage de données dans les objets blob, les files d’attente et les tables.  Dans le cas des comptes de stockage utilisés pour les disques de machine virtuelle, la convention d’affectation de noms doit impliquer leur association au nom de la machine virtuelle parente (ainsi que l’application d’un suffixe numérique du fait de la nécessité potentielle de disposer de plusieurs comptes de stockage pour les références (SKU) de machines virtuelles haut de gamme).

> [!TIP]
> Les comptes de stockage (pour les données ou les disques) doivent suivre une convention d’affectation de noms permettant d’exploiter plusieurs comptes de stockage (par exemple, par l’utilisation systématique d’un suffixe numérique).

Vous pouvez configurer un nom de domaine personnalisé pour accéder à des données blob dans votre compte de stockage Azure. Le point de terminaison par défaut du service BLOB est https://\<nom\>.blob.core.windows.net.

Cependant, si vous mappez un domaine personnalisé (tel que www.contoso.com) au point de terminaison des objets blob de votre compte de stockage, vous pouvez également accéder aux données d’objets blob de votre compte de stockage à l’aide de ce domaine. Par exemple, avec un nom de domaine personnalisé, `http://mystorage.blob.core.windows.net/mycontainer/myblob` devient accessible en tant que `http://www.contoso.com/mycontainer/myblob`.

Pour plus d’informations sur la configuration de cette fonctionnalité, consultez l’article [Configurer un nom de domaine personnalisé pour un point de terminaison de stockage Blob](/azure/storage/storage-custom-domain-name/).

Pour plus d’informations sur l’affectation de noms aux blobs, conteneurs et tables, consultez les articles répertoriés ci-après :

* [Désignation et référencement des conteneurs, des objets BLOB et des métadonnées](https://msdn.microsoft.com/library/dd135715.aspx)
* [Affectation de noms pour les files d’attente et les métadonnées](https://msdn.microsoft.com/library/dd179349.aspx)
* [Noms de table](https://msdn.microsoft.com/library/azure/dd179338.aspx)

Un nom de blob peut comporter n’importe quelle combinaison de caractères, mais les caractères d’URL réservés doivent être correctement placés dans une séquence d’échappement. Évitez les noms de blob qui se terminent par un point (.),une barre oblique (/) ou une séquence ou combinaison des deux. Par convention, la barre oblique correspond au séparateur de répertoires **virtuel**. N’utilisez pas de barre oblique (\\) dans un nom de blob. Bien que les API de client puissent l’autoriser, le hachage échouera et les signatures ne correspondront pas.

Vous ne pouvez pas modifier le nom d’un compte de stockage ou d’un conteneur après sa création. Si vous souhaitez utiliser un nouveau nom, vous devez le supprimer, puis le recréer.

> [!TIP]
> Nous vous recommandons d’établir une convention d’affectation de noms pour tous les types et comptes de stockage avant de commencer le développement d’un nouveau service ou d’une nouvelle application.

<!-- links -->

[scaffold]: /azure/azure-resource-manager/resource-manager-subscription-governance
