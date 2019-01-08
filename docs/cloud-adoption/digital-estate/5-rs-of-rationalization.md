---
title: Les 5 R de la rationalisation
titleSuffix: Enterprise Cloud Adoption
description: Décrit les options qui sont disponibles lors de la rationalisation d’un patrimoine numérique
author: BrianBlanchard
ms.date: 12/10/2018
ms.openlocfilehash: 66eee782b63210bcba3022e3be5b8fdee11716ec
ms.sourcegitcommit: bb7fcffbb41e2c26a26f8781df32825eb60df70c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/20/2018
ms.locfileid: "53643677"
---
# <a name="enterprise-cloud-adoption-the-5-rs-of-rationalization"></a><span data-ttu-id="39a83-103">Adoption du cloud d’entreprise : Les 5 R de la rationalisation</span><span class="sxs-lookup"><span data-stu-id="39a83-103">Enterprise Cloud Adoption: The 5 Rs of rationalization</span></span>

<span data-ttu-id="39a83-104">La rationalisation du cloud est le processus qui consiste à évaluer des ressources dans le but de déterminer la meilleure approche à adopter pour la migration ou la modernisation de chaque ressource présente dans le cloud.</span><span class="sxs-lookup"><span data-stu-id="39a83-104">Cloud Rationalization is the process of evaluating assets to determine the best approach to migrating or modernizing each asset in the cloud.</span></span> <span data-ttu-id="39a83-105">Pour plus d’informations sur le processus de rationalisation, consultez [Qu’est-ce qu’un patrimoine numérique ?](overview.md)</span><span class="sxs-lookup"><span data-stu-id="39a83-105">For more information about the process of rationalization, see [What is a digital estate?](overview.md)</span></span>

<span data-ttu-id="39a83-106">Les « 5 R de la rationalisation » listés ici décrivent les options de rationalisation les plus courantes.</span><span class="sxs-lookup"><span data-stu-id="39a83-106">The "5 Rs of rationalization" listed here describe the most common options for rationalization.</span></span>

## <a name="rehost"></a><span data-ttu-id="39a83-107">Réhébergement</span><span class="sxs-lookup"><span data-stu-id="39a83-107">Rehost</span></span>

<span data-ttu-id="39a83-108">Également appelé « lift and shift », le réhébergement déplace une ressource vers le fournisseur de cloud choisi, en apportant des modifications minimales à l’architecture globale.</span><span class="sxs-lookup"><span data-stu-id="39a83-108">Also known as "lift and shift," a rehost effort moves the current state asset to the chosen cloud provider, with minimal change to overall architecture.</span></span>

<span data-ttu-id="39a83-109">Les moteurs les plus courants sont les suivants :</span><span class="sxs-lookup"><span data-stu-id="39a83-109">Common drivers could include:</span></span>

* <span data-ttu-id="39a83-110">Réduire les dépenses d’investissement</span><span class="sxs-lookup"><span data-stu-id="39a83-110">Reduce CapEx</span></span>
* <span data-ttu-id="39a83-111">Libérer de l’espace au sein du centre de données</span><span class="sxs-lookup"><span data-stu-id="39a83-111">Free up datacenter space</span></span>
* <span data-ttu-id="39a83-112">Obtenir un retour sur investissement rapide</span><span class="sxs-lookup"><span data-stu-id="39a83-112">Quick cloud ROI</span></span>

<span data-ttu-id="39a83-113">Facteurs d’analyse quantitative :</span><span class="sxs-lookup"><span data-stu-id="39a83-113">Quantitative analysis factors:</span></span>

* <span data-ttu-id="39a83-114">Taille de la machine virtuelle (processeur, mémoire, stockage)</span><span class="sxs-lookup"><span data-stu-id="39a83-114">VM size (CPU, memory, storage)</span></span>
* <span data-ttu-id="39a83-115">Dépendances (trafic réseau)</span><span class="sxs-lookup"><span data-stu-id="39a83-115">Dependencies (network traffic)</span></span>
* <span data-ttu-id="39a83-116">Compatibilité des ressources</span><span class="sxs-lookup"><span data-stu-id="39a83-116">Asset compatibility</span></span>

<span data-ttu-id="39a83-117">Facteurs d’analyse qualitative :</span><span class="sxs-lookup"><span data-stu-id="39a83-117">Qualitative analysis factors:</span></span>

* <span data-ttu-id="39a83-118">Tolérance au changement</span><span class="sxs-lookup"><span data-stu-id="39a83-118">Tolerance for change</span></span>
* <span data-ttu-id="39a83-119">Priorités de l’entreprise</span><span class="sxs-lookup"><span data-stu-id="39a83-119">Business priorities</span></span>
* <span data-ttu-id="39a83-120">Événements critiques pour l’entreprise</span><span class="sxs-lookup"><span data-stu-id="39a83-120">Critical business events</span></span>
* <span data-ttu-id="39a83-121">Dépendances de processus</span><span class="sxs-lookup"><span data-stu-id="39a83-121">Process dependencies</span></span>

## <a name="refactor"></a><span data-ttu-id="39a83-122">Refactorisation</span><span class="sxs-lookup"><span data-stu-id="39a83-122">Refactor</span></span>

<span data-ttu-id="39a83-123">Les options PaaS permettent de réduire les coûts d’exploitation associés à de nombreuses applications.</span><span class="sxs-lookup"><span data-stu-id="39a83-123">Platform as a Service (PaaS) options can reduce operational costs associated with many applications.</span></span> <span data-ttu-id="39a83-124">Il paraît plus prudent de refactoriser légèrement une application pour qu’elle corresponde à un modèle PaaS.</span><span class="sxs-lookup"><span data-stu-id="39a83-124">It can be prudent to slightly refactor an application to fit a PaaS based model.</span></span>

<span data-ttu-id="39a83-125">La refactorisation s’applique également au processus de développement d’applications, dans lequel du code est refactorisé pour permettre à une application de répondre à de nouvelles opportunités commerciales.</span><span class="sxs-lookup"><span data-stu-id="39a83-125">Refactor also refers to the application development process of refactoring code to allow an application to deliver on new business opportunities.</span></span>

<span data-ttu-id="39a83-126">Les moteurs les plus courants sont les suivants :</span><span class="sxs-lookup"><span data-stu-id="39a83-126">Common drivers could include:</span></span>

* <span data-ttu-id="39a83-127">Mises à jour plus rapides</span><span class="sxs-lookup"><span data-stu-id="39a83-127">Faster, shorter, updates</span></span>
* <span data-ttu-id="39a83-128">Portabilité du code</span><span class="sxs-lookup"><span data-stu-id="39a83-128">Code portability</span></span>
* <span data-ttu-id="39a83-129">Une plus grande efficacité du cloud (ressources, vitesse, coût)</span><span class="sxs-lookup"><span data-stu-id="39a83-129">Greater cloud efficiency (resources, speed, cost)</span></span>

<span data-ttu-id="39a83-130">Facteurs d’analyse quantitative :</span><span class="sxs-lookup"><span data-stu-id="39a83-130">Quantitative analysis factors:</span></span>

* <span data-ttu-id="39a83-131">Taille des ressources d’application (processeur, mémoire, stockage)</span><span class="sxs-lookup"><span data-stu-id="39a83-131">Application asset size (CPU, memory, storage)</span></span>
* <span data-ttu-id="39a83-132">Dépendances (trafic réseau)</span><span class="sxs-lookup"><span data-stu-id="39a83-132">Dependencies (network traffic)</span></span>
* <span data-ttu-id="39a83-133">Trafic utilisateur (consultations de page, temps passé sur une page, temps de chargement)</span><span class="sxs-lookup"><span data-stu-id="39a83-133">User traffic (page views, time on page, load time)</span></span>
* <span data-ttu-id="39a83-134">Plateforme de développement (langages, plateforme de données, services de niveau intermédiaire)</span><span class="sxs-lookup"><span data-stu-id="39a83-134">Development platform (languages, data platform, middle tier services)</span></span>

<span data-ttu-id="39a83-135">Facteurs d’analyse qualitative :</span><span class="sxs-lookup"><span data-stu-id="39a83-135">Qualitative analysis factors:</span></span>

* <span data-ttu-id="39a83-136">Investissements continus</span><span class="sxs-lookup"><span data-stu-id="39a83-136">Continued business investments</span></span>
* <span data-ttu-id="39a83-137">Options de cloud bursting/Chronologies</span><span class="sxs-lookup"><span data-stu-id="39a83-137">Bursting options/timelines</span></span>
* <span data-ttu-id="39a83-138">Dépendances des processus métier</span><span class="sxs-lookup"><span data-stu-id="39a83-138">Business process dependencies</span></span>

## <a name="rearchitect"></a><span data-ttu-id="39a83-139">Réarchitecture</span><span class="sxs-lookup"><span data-stu-id="39a83-139">Rearchitect</span></span>

<span data-ttu-id="39a83-140">Certaines applications vieillissantes ne sont pas compatibles avec les fournisseurs de cloud, en raison des décisions qui ont été prises concernant l’architecture lors de la conception de l’application.</span><span class="sxs-lookup"><span data-stu-id="39a83-140">Some aging applications aren't compatible with cloud providers because of the architectural decisions made when the application was built.</span></span> <span data-ttu-id="39a83-141">Dans ce cas, l’application doit subir une réarchitecture avant d’être transformée.</span><span class="sxs-lookup"><span data-stu-id="39a83-141">In these cases, the application may need to be rearchitected prior to transformation.</span></span>

<span data-ttu-id="39a83-142">Dans d’autres cas, les applications qui sont compatibles avec le cloud mais qui ne sont pas natives du cloud peuvent bénéficier d’une meilleure rentabilité et d’une efficacité opérationnelle si vous les réarchitecturez pour en faire des applications natives du cloud.</span><span class="sxs-lookup"><span data-stu-id="39a83-142">In other cases, applications that are cloud compatible, but not cloud native benefits, may produce cost efficiencies and operational efficiencies by rearchitecting the solution to be a cloud native application.</span></span>

<span data-ttu-id="39a83-143">Les moteurs les plus courants sont les suivants :</span><span class="sxs-lookup"><span data-stu-id="39a83-143">Common drivers could include:</span></span>

* <span data-ttu-id="39a83-144">Mise à l’échelle et agilité de l’application</span><span class="sxs-lookup"><span data-stu-id="39a83-144">Application scale and agility</span></span>
* <span data-ttu-id="39a83-145">Adoption facilitée des nouvelles fonctionnalités cloud</span><span class="sxs-lookup"><span data-stu-id="39a83-145">Easier adoption of new cloud capabilities</span></span>
* <span data-ttu-id="39a83-146">Piles de technologies mixtes</span><span class="sxs-lookup"><span data-stu-id="39a83-146">Mix of technology stacks</span></span>

<span data-ttu-id="39a83-147">Facteurs d’analyse quantitative :</span><span class="sxs-lookup"><span data-stu-id="39a83-147">Quantitative analysis factors:</span></span>

* <span data-ttu-id="39a83-148">Taille des ressources d’application (processeur, mémoire, stockage)</span><span class="sxs-lookup"><span data-stu-id="39a83-148">Application asset size (CPU, memory, storage)</span></span>
* <span data-ttu-id="39a83-149">Dépendances (trafic réseau)</span><span class="sxs-lookup"><span data-stu-id="39a83-149">Dependencies (network traffic)</span></span>
* <span data-ttu-id="39a83-150">Trafic utilisateur (consultations de page, temps passé sur une page, temps de chargement)</span><span class="sxs-lookup"><span data-stu-id="39a83-150">User traffic (page views, time on page, load time)</span></span>
* <span data-ttu-id="39a83-151">Plateforme de développement (langages, plateforme de données, services de niveau intermédiaire)</span><span class="sxs-lookup"><span data-stu-id="39a83-151">Development platform (languages, data platform, middle tier services)</span></span>

<span data-ttu-id="39a83-152">Facteurs d’analyse qualitative :</span><span class="sxs-lookup"><span data-stu-id="39a83-152">Qualitative analysis factors:</span></span>

* <span data-ttu-id="39a83-153">Investissements continus en augmentation</span><span class="sxs-lookup"><span data-stu-id="39a83-153">Growing business investments</span></span>
* <span data-ttu-id="39a83-154">Coûts d’exploitation</span><span class="sxs-lookup"><span data-stu-id="39a83-154">Operational costs</span></span>
* <span data-ttu-id="39a83-155">Boucles de rétroaction et investissements DevOps potentiels</span><span class="sxs-lookup"><span data-stu-id="39a83-155">Potential feedback loops and DevOps investments</span></span>

## <a name="rebuild"></a><span data-ttu-id="39a83-156">Reconstruction</span><span class="sxs-lookup"><span data-stu-id="39a83-156">Rebuild</span></span>

<span data-ttu-id="39a83-157">Dans certains scénarios, les efforts nécessaires à la réarchitecture d’une application peuvent être trop importants pour justifier tout investissement supplémentaire.</span><span class="sxs-lookup"><span data-stu-id="39a83-157">In some scenarios, the delta that must be overcome to carry forward an application can be too large to justify further investment.</span></span> <span data-ttu-id="39a83-158">C’est particulièrement vrai pour les applications qui répondaient autrefois aux besoins de l’entreprise, mais qui désormais ne sont plus prises en charge ou ne sont plus alignées sur les processus actuels de l’entreprise.</span><span class="sxs-lookup"><span data-stu-id="39a83-158">This is especially true for applications that used to meet the needs of the business, but are now unsupported or misaligned with how the business processes are executed today.</span></span> <span data-ttu-id="39a83-159">Dans ce cas, une nouvelle base de code est créée pour obtenir une application native du cloud.</span><span class="sxs-lookup"><span data-stu-id="39a83-159">In this case, a new code base is created to align with a cloud native approach.</span></span>

<span data-ttu-id="39a83-160">Les moteurs les plus courants sont les suivants :</span><span class="sxs-lookup"><span data-stu-id="39a83-160">Common drivers could include:</span></span>

* <span data-ttu-id="39a83-161">Accélérer l’innovation</span><span class="sxs-lookup"><span data-stu-id="39a83-161">Accelerate innovation</span></span>
* <span data-ttu-id="39a83-162">Créer des applications plus rapidement</span><span class="sxs-lookup"><span data-stu-id="39a83-162">Build apps faster</span></span>
* <span data-ttu-id="39a83-163">Réduire les coûts d’exploitation</span><span class="sxs-lookup"><span data-stu-id="39a83-163">Reduce operational cost</span></span>

<span data-ttu-id="39a83-164">Facteurs d’analyse quantitative :</span><span class="sxs-lookup"><span data-stu-id="39a83-164">Quantitative analysis factors:</span></span>

* <span data-ttu-id="39a83-165">Taille des ressources d’application (processeur, mémoire, stockage)</span><span class="sxs-lookup"><span data-stu-id="39a83-165">Application asset size (CPU, memory, storage)</span></span>
* <span data-ttu-id="39a83-166">Dépendances (trafic réseau)</span><span class="sxs-lookup"><span data-stu-id="39a83-166">Dependencies (network traffic)</span></span>
* <span data-ttu-id="39a83-167">Trafic utilisateur (consultations de page, temps passé sur une page, temps de chargement)</span><span class="sxs-lookup"><span data-stu-id="39a83-167">User traffic (page views, time on page, load time)</span></span>
* <span data-ttu-id="39a83-168">Plateforme de développement (langages, plateforme de données, services de niveau intermédiaire)</span><span class="sxs-lookup"><span data-stu-id="39a83-168">Development platform (languages, data platform, middle tier services)</span></span>

<span data-ttu-id="39a83-169">Facteurs d’analyse qualitative :</span><span class="sxs-lookup"><span data-stu-id="39a83-169">Qualitative analysis Factors:</span></span>

* <span data-ttu-id="39a83-170">Baisse de la satisfaction des utilisateurs finaux</span><span class="sxs-lookup"><span data-stu-id="39a83-170">Declining end user satisfaction</span></span>
* <span data-ttu-id="39a83-171">Processus d’entreprise limités dans leur fonctionnalité</span><span class="sxs-lookup"><span data-stu-id="39a83-171">Business processes limited by functionality</span></span>
* <span data-ttu-id="39a83-172">Amélioration potentielle des coûts, de l’expérience ou du chiffre d’affaires</span><span class="sxs-lookup"><span data-stu-id="39a83-172">Potential cost, experience, or revenue gains</span></span>

## <a name="replace"></a><span data-ttu-id="39a83-173">Replace</span><span class="sxs-lookup"><span data-stu-id="39a83-173">Replace</span></span>

<span data-ttu-id="39a83-174">Les solutions sont généralement implémentées à l’aide de la technologie et de l’approche les mieux adaptées sur le moment.</span><span class="sxs-lookup"><span data-stu-id="39a83-174">Solutions are generally implemented using the best technology and approach available at the time.</span></span> <span data-ttu-id="39a83-175">Dans certains cas, les applications SaaS peuvent fournir toutes les fonctionnalités nécessaires à l’application hébergée.</span><span class="sxs-lookup"><span data-stu-id="39a83-175">In some cases, Software as a Service (SaaS) applications can meet all of the functionality required of the hosted application.</span></span> <span data-ttu-id="39a83-176">Dans ces scénarios, une charge de travail peut être programmée pour un remplacement futur, ce qui évite tout effort de transformation.</span><span class="sxs-lookup"><span data-stu-id="39a83-176">In these scenarios, a workload could be slated for future replacement, effectively removing it from the transformation effort.</span></span>

<span data-ttu-id="39a83-177">Les moteurs les plus courants sont les suivants :</span><span class="sxs-lookup"><span data-stu-id="39a83-177">Common drivers could include:</span></span>

* <span data-ttu-id="39a83-178">Normaliser les bonnes pratiques du secteur</span><span class="sxs-lookup"><span data-stu-id="39a83-178">Standardize around industry-best practices</span></span>
* <span data-ttu-id="39a83-179">Accélérer l’adoption des approches basées sur les processus d’entreprise</span><span class="sxs-lookup"><span data-stu-id="39a83-179">Accelerate adoption of business process driven approaches</span></span>
* <span data-ttu-id="39a83-180">Réallouer les investissements de développement dans des applications qui créent un avantage compétitif ou autre avantage</span><span class="sxs-lookup"><span data-stu-id="39a83-180">Reallocate development investments into applications that create competitive differentiation or advantages</span></span>

<span data-ttu-id="39a83-181">Facteurs d’analyse quantitative :</span><span class="sxs-lookup"><span data-stu-id="39a83-181">Quantitative analysis factors:</span></span>

* <span data-ttu-id="39a83-182">Réductions des coûts d’exploitation généraux</span><span class="sxs-lookup"><span data-stu-id="39a83-182">General operating cost reductions</span></span>
* <span data-ttu-id="39a83-183">Taille de la machine virtuelle (processeur, mémoire, stockage)</span><span class="sxs-lookup"><span data-stu-id="39a83-183">VM size (CPU, memory, storage)</span></span>
* <span data-ttu-id="39a83-184">Dépendances (trafic réseau)</span><span class="sxs-lookup"><span data-stu-id="39a83-184">Dependencies (network traffic)</span></span>
* <span data-ttu-id="39a83-185">Ressources à mettre hors service</span><span class="sxs-lookup"><span data-stu-id="39a83-185">Assets to be retired</span></span>

<span data-ttu-id="39a83-186">Facteurs d’analyse qualitative :</span><span class="sxs-lookup"><span data-stu-id="39a83-186">Qualitative analysis factors:</span></span>

* <span data-ttu-id="39a83-187">Analyse coûts-avantages de l’architecture actuelle par rapport à une solution SaaS</span><span class="sxs-lookup"><span data-stu-id="39a83-187">Cost benefit analysis of current architecture vs SaaS solution</span></span>
* <span data-ttu-id="39a83-188">Diagramme des processus métier</span><span class="sxs-lookup"><span data-stu-id="39a83-188">Business process maps</span></span>
* <span data-ttu-id="39a83-189">Schémas de données</span><span class="sxs-lookup"><span data-stu-id="39a83-189">Data schemas</span></span>
* <span data-ttu-id="39a83-190">Processus personnalisés ou automatisés</span><span class="sxs-lookup"><span data-stu-id="39a83-190">Custom or automated processes</span></span>

## <a name="next-steps"></a><span data-ttu-id="39a83-191">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="39a83-191">Next steps</span></span>

<span data-ttu-id="39a83-192">Collectivement, ces 5 R de la rationalisation ne peuvent pas être appliqués à un patrimoine numérique en vue d’effectuer des décisions de rationalisation concernant l’état futur de chaque application.</span><span class="sxs-lookup"><span data-stu-id="39a83-192">Collectively, these 5 Rs of Rationalization can be applied to a digital estate to make rationalization decisions regarding the future state of each application.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="39a83-193">Qu’est-ce qu’un patrimoine numérique ?</span><span class="sxs-lookup"><span data-stu-id="39a83-193">What is a digital estate?</span></span>](overview.md)
