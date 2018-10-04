---
title: Consolidation des ressources de calcul
description: Consolidez plusieurs tâches ou opérations en une seule unité de calcul.
keywords: modèle de conception
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
ms.openlocfilehash: bd212b8b4406a08058f811db030843f732e08cdc
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428837"
---
# <a name="compute-resource-consolidation-pattern"></a><span data-ttu-id="c2fc9-104">Modèle de consolidation des ressources de calcul</span><span class="sxs-lookup"><span data-stu-id="c2fc9-104">Compute Resource Consolidation pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="c2fc9-105">Consolidez plusieurs tâches ou opérations en une seule unité de calcul.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-105">Consolidate multiple tasks or operations into a single computational unit.</span></span> <span data-ttu-id="c2fc9-106">Cela permet d’augmenter l’utilisation des ressources de calcul et de réduire les coûts et la surcharge de gestion associés au traitement du calcul dans des applications hébergées dans le cloud.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-106">This can increase compute resource utilization, and reduce the costs and management overhead associated with performing compute processing in cloud-hosted applications.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="c2fc9-107">Contexte et problème</span><span class="sxs-lookup"><span data-stu-id="c2fc9-107">Context and problem</span></span>

<span data-ttu-id="c2fc9-108">Une application cloud implémente souvent des opérations diverses.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-108">A cloud application often implements a variety of operations.</span></span> <span data-ttu-id="c2fc9-109">Dans certaines solutions, il est logique de suivre le principe de conception de séparation des problèmes initial et de diviser ces opérations en unités de calcul distinctes hébergées et déployées de façon individuelle (par exemple en tant qu’applications web App Service séparées, que machines virtuelles séparées ou rôles de service cloud séparés).</span><span class="sxs-lookup"><span data-stu-id="c2fc9-109">In some solutions it makes sense to follow the design principle of separation of concerns initially, and divide these operations into separate computational units that are hosted and deployed individually (for example, as separate App Service web apps, separate Virtual Machines, or separate Cloud Service roles).</span></span> <span data-ttu-id="c2fc9-110">Toutefois, bien que cette stratégie permette de simplifier l’étude logique de la solution, le fait de déployer un grand nombre d’unités de calcul pour la même application peut augmenter les coûts d’hébergement du runtime et rendre plus complexe la gestion du système.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-110">However, although this strategy can help simplify the logical design of the solution, deploying a large number of computational units as part of the same application can increase runtime hosting costs and make management of the system more complex.</span></span>

<span data-ttu-id="c2fc9-111">À titre d’exemple, la figure illustre la structure simplifiée d’une solution hébergée dans le cloud qui est implémentée à l’aide de plusieurs unités de calcul.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-111">As an example, the figure shows the simplified structure of a cloud-hosted solution that is implemented using more than one computational unit.</span></span> <span data-ttu-id="c2fc9-112">Chaque unité de calcul s’exécute dans son propre environnement virtuel.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-112">Each computational unit runs in its own virtual environment.</span></span> <span data-ttu-id="c2fc9-113">Chaque fonction a été implémentée en tant que tâche séparée (de la Tâche A à la Tâche E) s’exécutant dans sa propre unité de calcul.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-113">Each function has been implemented as a separate task (labeled Task A through Task E) running in its own computational unit.</span></span>

![Exécution de tâches dans un environnement cloud à l’aide d’un ensemble d’unités de calcul dédiées](./_images/compute-resource-consolidation-diagram.png)


<span data-ttu-id="c2fc9-115">Chaque unité de calcul consomme des ressources facturables, même lorsqu’elle est inactive ou peu utilisée.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-115">Each computational unit consumes chargeable resources, even when it's idle or lightly used.</span></span> <span data-ttu-id="c2fc9-116">Par conséquent, cette solution n’est pas toujours la plus rentable.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-116">Therefore, this isn't always the most cost-effective solution.</span></span>

<span data-ttu-id="c2fc9-117">Dans Azure, ce problème s’applique aux rôles dans un service cloud, App Services et des machines virtuelles.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-117">In Azure, this concern applies to roles in a Cloud Service, App Services, and Virtual Machines.</span></span> <span data-ttu-id="c2fc9-118">Ces éléments s’exécutent dans leur propre environnement virtuel.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-118">These items run in their own virtual environment.</span></span> <span data-ttu-id="c2fc9-119">L’exécution d’une collection de rôles, sites Web ou de machines virtuelles séparés qui sont conçus pour exécuter un groupe d’opérations bien définies, mais qui ont besoin de communiquer et de coopérer car ils font partie d’une solution unique, peut constituer une utilisation inefficace des ressources.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-119">Running a collection of separate roles, websites, or virtual machines that are designed to perform a set of well-defined operations, but that need to communicate and cooperate as part of a single solution, can be an inefficient use of resources.</span></span>

## <a name="solution"></a><span data-ttu-id="c2fc9-120">Solution</span><span class="sxs-lookup"><span data-stu-id="c2fc9-120">Solution</span></span>

<span data-ttu-id="c2fc9-121">Dans l’optique de faciliter la réduction des coûts, l’augmentation de l’utilisation et l’amélioration de la vitesse de communication et la diminution de la gestion, il est possible de consolider plusieurs tâches ou opérations dans une seule unité de calcul.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-121">To help reduce costs, increase utilization, improve communication speed, and reduce management it's possible to consolidate multiple tasks or operations into a single computational unit.</span></span>

<span data-ttu-id="c2fc9-122">Les tâches peuvent être regroupées en fonction de critères basés sur les fonctionnalités fournies par l’environnement et les coûts associés à ces fonctionnalités.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-122">Tasks can be grouped according to criteria based on the features provided by the environment and the costs associated with these features.</span></span> <span data-ttu-id="c2fc9-123">Une approche courante consiste à rechercher des tâches qui ont un profil similaire en matière d’exigences de traitement, de durée de vie et d’extensibilité.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-123">A common approach is to look for tasks that have a similar profile concerning their scalability, lifetime, and processing requirements.</span></span> <span data-ttu-id="c2fc9-124">Le regroupement de ces éléments leur permet d’effectuer une mise à l’échelle en tant qu’unité.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-124">Grouping these together allows them to scale as a unit.</span></span> <span data-ttu-id="c2fc9-125">L’élasticité fournie par de nombreux environnements cloud permet de démarrer et d’arrêter des instances supplémentaires d’une unité de calcul en fonction de la charge de travail.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-125">The elasticity provided by many cloud environments enables additional instances of a computational unit to be started and stopped according to the workload.</span></span> <span data-ttu-id="c2fc9-126">Par exemple, Azure propose une fonctionnalité de mise à l’échelle automatique que vous pouvez appliquer à des rôles dans un service cloud, App Services et des machines virtuelles.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-126">For example, Azure provides autoscaling that you can apply to roles in a Cloud Service, App Services, and Virtual Machines.</span></span> <span data-ttu-id="c2fc9-127">Pour plus d’informations, consultez [Mise à l’échelle automatique](https://msdn.microsoft.com/library/dn589774.aspx).</span><span class="sxs-lookup"><span data-stu-id="c2fc9-127">For more information, see [Autoscaling Guidance](https://msdn.microsoft.com/library/dn589774.aspx).</span></span>

<span data-ttu-id="c2fc9-128">En tant que contre-exemple pour montrer comment l’extensibilité peut être utilisée pour déterminer quelles opérations ne doivent pas être regroupées, pensez aux deux tâches suivantes :</span><span class="sxs-lookup"><span data-stu-id="c2fc9-128">As a counter example to show how scalability can be used to determine which operations shouldn't be grouped together, consider the following two tasks:</span></span>

- <span data-ttu-id="c2fc9-129">La tâche 1 interroge les messages peu fréquents et non sensibles au temps envoyés à une file d’attente.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-129">Task 1 polls for infrequent, time-insensitive messages sent to a queue.</span></span>
- <span data-ttu-id="c2fc9-130">La tâche 2 gère les pics de trafic réseau de volume élevé.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-130">Task 2 handles high-volume bursts of network traffic.</span></span>

<span data-ttu-id="c2fc9-131">La seconde tâche nécessite de l’élasticité qui peut impliquer le démarrage et l’arrêt d’un grand nombre d’instances de l’unité de calcul.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-131">The second task requires elasticity that can involve starting and stopping a large number of instances of the computational unit.</span></span> <span data-ttu-id="c2fc9-132">Appliquer la même mise à l’échelle à la première tâche entraînerait simplement davantage de tâches d’écoute de messages peu fréquents dans la même file d’attente et constitue un gaspillage de ressources.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-132">Applying the same scaling to the first task would simply result in more tasks listening for infrequent messages on the same queue, and is a waste of resources.</span></span>

<span data-ttu-id="c2fc9-133">Dans de nombreux environnements cloud, il est possible de spécifier les ressources disponibles sur une unité de calcul en termes de nombre de cœurs de processeur, de quantité de mémoire, d’espace disque, etc.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-133">In many cloud environments it's possible to specify the resources available to a computational unit in terms of the number of CPU cores, memory, disk space, and so on.</span></span> <span data-ttu-id="c2fc9-134">En règle générale, plus il y a de ressources spécifiées, plus le coût est élevé.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-134">Generally, the more resources specified, the greater the cost.</span></span> <span data-ttu-id="c2fc9-135">Pour faire des économies, il est important d’optimiser le travail qu’une unité de calcul coûteuse effectue, et de ne pas la laisser inactive pendant une période prolongée.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-135">To save money, it's important to maximize the work an expensive computational unit performs, and not let it become inactive for an extended period.</span></span>

<span data-ttu-id="c2fc9-136">S’il existe des tâches qui nécessitent beaucoup de ressources de processeur lors de courtes périodes de pics d’activité, envisagez de les consolider dans une seule unité de calcul qui fournit la puissance nécessaire.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-136">If there are tasks that require a great deal of CPU power in short bursts, consider consolidating these into a single computational unit that provides the necessary power.</span></span> <span data-ttu-id="c2fc9-137">Toutefois, il est important d’équilibrer ce besoin pour que les ressources coûteuses restent occupées par rapport à la contention qui pourrait se produire si elles étaient trop sollicitées.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-137">However, it's important to balance this need to keep expensive resources busy against the contention that could occur if they are over stressed.</span></span> <span data-ttu-id="c2fc9-138">Les tâches gourmandes en ressources de calcul et longues à exécuter ne doivent pas partager la même unité de calcul par exemple.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-138">Long-running, compute-intensive tasks shouldn't share the same computational unit, for example.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="c2fc9-139">Problèmes et considérations</span><span class="sxs-lookup"><span data-stu-id="c2fc9-139">Issues and considerations</span></span>

<span data-ttu-id="c2fc9-140">Prenez en compte les points suivants lorsque vous implémentez ce modèle :</span><span class="sxs-lookup"><span data-stu-id="c2fc9-140">Consider the following points when implementing this pattern:</span></span>

<span data-ttu-id="c2fc9-141">**L’extensibilité et l’élasticité**.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-141">**Scalability and elasticity**.</span></span> <span data-ttu-id="c2fc9-142">De nombreuses solutions cloud implémentent l’extensibilité et l’élasticité au niveau de l’unité de calcul en démarrant et arrêtant des instances d’unités.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-142">Many cloud solutions implement scalability and elasticity at the level of the computational unit by starting and stopping instances of units.</span></span> <span data-ttu-id="c2fc9-143">Évitez de regrouper des tâches ayant des exigences d’extensibilité en conflit dans la même unité de calcul.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-143">Avoid grouping tasks that have conflicting scalability requirements in the same computational unit.</span></span>

<span data-ttu-id="c2fc9-144">**Durée de vie**.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-144">**Lifetime**.</span></span> <span data-ttu-id="c2fc9-145">L’infrastructure cloud recycle périodiquement l’environnement virtuel qui héberge une unité de calcul.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-145">The cloud infrastructure periodically recycles the virtual environment that hosts a computational unit.</span></span> <span data-ttu-id="c2fc9-146">Lorsqu’il existe de nombreuses tâches longues à exécuter à l’intérieur d’une unité de calcul, il peut être nécessaire de configurer l’unité pour l’empêcher d’être recyclée tant que ces tâches ne sont pas terminées.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-146">When there are many long-running tasks inside a computational unit, it might be necessary to configure the unit to prevent it from being recycled until these tasks have finished.</span></span> <span data-ttu-id="c2fc9-147">Vous pouvez également concevoir les tâches en utilisant une approche de création de points de contrôle qui leur permet de s’arrêter de façon nette et de reprendre à partir du point où elles ont été interrompues lorsque l’unité de calcul est redémarrée.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-147">Alternatively, design the tasks by using a check-pointing approach that enables them to stop cleanly, and continue at the point they were interrupted when the computational unit is restarted.</span></span>

<span data-ttu-id="c2fc9-148">**Cadence de mise en production**.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-148">**Release cadence**.</span></span> <span data-ttu-id="c2fc9-149">Si l’implémentation ou la configuration d’une tâche change fréquemment, il peut être nécessaire d’arrêter l’unité de calcul hébergeant le code mis à jour, de reconfigurer et de redéployer l’unité puis de la redémarrer.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-149">If the implementation or configuration of a task changes frequently, it might be necessary to stop the computational unit hosting the updated code, reconfigure and redeploy the unit, and then restart it.</span></span> <span data-ttu-id="c2fc9-150">Ce processus nécessitera également l’arrêt, le redéploiement et le redémarrage de toutes les autres tâches dans la même unité de calcul.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-150">This process will also require that all other tasks within the same computational unit are stopped, redeployed, and restarted.</span></span>

<span data-ttu-id="c2fc9-151">**Sécurité**.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-151">**Security**.</span></span> <span data-ttu-id="c2fc9-152">Des tâches dans la même unité de calcul peuvent partager le même contexte de sécurité et être en mesure d’accéder aux mêmes ressources.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-152">Tasks in the same computational unit might share the same security context and be able to access the same resources.</span></span> <span data-ttu-id="c2fc9-153">Il doit y avoir un niveau élevé d’approbation entre les tâches et vous devez être sûr qu’une tâche ne va pas corrompre une autre ou y nuire.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-153">There must be a high degree of trust between the tasks, and confidence that one task isn't going to corrupt or adversely affect another.</span></span> <span data-ttu-id="c2fc9-154">En outre, augmenter le nombre de tâches s’exécutant dans une unité de calcul augmente la surface d’attaque de l’unité.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-154">Additionally, increasing the number of tasks running in a computational unit increases the attack surface of the unit.</span></span> <span data-ttu-id="c2fc9-155">Chaque tâche est uniquement aussi sûre que celle avec le plus de vulnérabilités.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-155">Each task is only as secure as the one with the most vulnerabilities.</span></span>

<span data-ttu-id="c2fc9-156">**Tolérance de panne**.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-156">**Fault tolerance**.</span></span> <span data-ttu-id="c2fc9-157">Si une tâche dans une unité de calcul échoue ou se comporte de façon anormale, elle peut affecter les autres tâches s’exécutant dans la même unité.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-157">If one task in a computational unit fails or behaves abnormally, it can affect the other tasks running within the same unit.</span></span> <span data-ttu-id="c2fc9-158">Par exemple, si une tâche ne démarre pas correctement, elle peut entraîner l’échec de l’ensemble de la logique de démarrage de l’unité de calcul et empêcher les autres taches de la même unité de s’exécuter.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-158">For example, if one task fails to start correctly it can cause the entire startup logic for the computational unit to fail, and prevent other tasks in the same unit from running.</span></span>

<span data-ttu-id="c2fc9-159">**Contention**.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-159">**Contention**.</span></span> <span data-ttu-id="c2fc9-160">Évitez d’introduire de la contention entre les tâches qui sont en concurrence pour bénéficier des ressources de la même unité de calcul.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-160">Avoid introducing contention between tasks that compete for resources in the same computational unit.</span></span> <span data-ttu-id="c2fc9-161">Dans l’idéal, les tâches qui partagent la même unité de calcul doivent présenter des caractéristiques d’utilisation de ressources différentes.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-161">Ideally, tasks that share the same computational unit should exhibit different resource utilization characteristics.</span></span> <span data-ttu-id="c2fc9-162">Par exemple, deux tâches gourmandes en ressources de calcul ne doivent probablement pas résider dans la même unité de calcul, comme c’est le cas pour deux tâches qui utilisent d’importantes quantités de mémoire.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-162">For example, two compute-intensive tasks should probably not reside in the same computational unit, and neither should two tasks that consume large amounts of memory.</span></span> <span data-ttu-id="c2fc9-163">Toutefois, associer une tâche gourmande en ressources de calcul à une tâche requérant une grande quantité de mémoire est possible.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-163">However, mixing a compute intensive task with a task that requires a large amount of memory is a workable combination.</span></span>

> [!NOTE]
>  <span data-ttu-id="c2fc9-164">Envisagez de consolider des ressources de calcul uniquement pour un système qui a été en production pendant un certain temps, afin que les opérateurs et développeurs puissent surveiller le système et créer une _carte thermique_ qui identifie la façon dont chaque tâche utilise les différentes ressources.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-164">Consider consolidating compute resources only for a system that's been in production for a period of time so that operators and developers can monitor the system and create a _heat map_ that identifies how each task utilizes differing resources.</span></span> <span data-ttu-id="c2fc9-165">Cette carte peut être utilisée pour déterminer les tâches les plus adaptées au partage de ressources de calcul.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-165">This map can be used to determine which tasks are good candidates for sharing compute resources.</span></span>

<span data-ttu-id="c2fc9-166">**Complexité** :</span><span class="sxs-lookup"><span data-stu-id="c2fc9-166">**Complexity**.</span></span> <span data-ttu-id="c2fc9-167">Combiner plusieurs tâches dans une seule unité de calcul complique le code dans l’unité et le rend plus difficile à tester, déboguer et maintenir.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-167">Combining multiple tasks into a single computational unit adds complexity to the code in the unit, possibly making it more difficult to test, debug, and maintain.</span></span>

<span data-ttu-id="c2fc9-168">**Architecture logique stable**.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-168">**Stable logical architecture**.</span></span> <span data-ttu-id="c2fc9-169">Concevez et implémentez le code dans chaque tâche de façon à ce qu’il n’ait pas besoin d’être modifié même si l’environnement physique dans lequel la tâche s’exécute change.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-169">Design and implement the code in each task so that it shouldn't need to change, even if the physical environment the task runs in does change.</span></span>

<span data-ttu-id="c2fc9-170">**Autres stratégies**.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-170">**Other strategies**.</span></span> <span data-ttu-id="c2fc9-171">Consolider des ressources de calcul n’est qu’une des méthodes permettant de réduire les coûts associés à l’exécution simultanée de plusieurs tâches.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-171">Consolidating compute resources is only one way to help reduce costs associated with running multiple tasks concurrently.</span></span> <span data-ttu-id="c2fc9-172">Cette opération nécessite une planification et une surveillance précises pour vous assurer que l’approche reste efficace.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-172">It requires careful planning and monitoring to ensure that it remains an effective approach.</span></span> <span data-ttu-id="c2fc9-173">D’autres stratégies peuvent être plus appropriées, selon la nature du travail et l’emplacement où se trouvent les utilisateurs qui exécutent ces tâches.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-173">Other strategies might be more appropriate, depending on the nature of the work and where the users these tasks are running are located.</span></span> <span data-ttu-id="c2fc9-174">Par exemple, la décomposition fonctionnelle de la charge de travail (comme décrit dans [Compute Partitioning Guidance](https://msdn.microsoft.com/library/dn589773.aspx) (Conseils sur le partitionnement du calcul)) peut être une option plus judicieuse.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-174">For example, functional decomposition of the workload (as described by the [Compute Partitioning Guidance](https://msdn.microsoft.com/library/dn589773.aspx)) might be a better option.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="c2fc9-175">Quand utiliser ce modèle</span><span class="sxs-lookup"><span data-stu-id="c2fc9-175">When to use this pattern</span></span>

<span data-ttu-id="c2fc9-176">Utilisez ce modèle pour les tâches qui ne sont pas rentables si elles s’exécutent dans leurs propres unités de calcul.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-176">Use this pattern for tasks that are not cost effective if they run in their own computational units.</span></span> <span data-ttu-id="c2fc9-177">Si une tâche est la plupart du temps à l’état inactif, l’exécuter dans une unité dédiée peut être coûteux.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-177">If a task spends much of its time idle, running this task in a dedicated unit can be expensive.</span></span>

<span data-ttu-id="c2fc9-178">Ce modèle peut ne pas être adapté pour les tâches qui effectuent des opérations critiques tolérantes aux pannes ou pour les tâches qui traitent des données privées ou extrêmement sensibles et qui nécessitent leur propre contexte de sécurité.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-178">This pattern might not be suitable for tasks that perform critical fault-tolerant operations, or tasks that process highly sensitive or private data and require their own security context.</span></span> <span data-ttu-id="c2fc9-179">Ces tâches doivent s’exécutent dans leur propre environnement isolé, dans une unité de calcul distincte.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-179">These tasks should run in their own isolated environment, in a separate computational unit.</span></span>

## <a name="example"></a><span data-ttu-id="c2fc9-180">Exemples</span><span class="sxs-lookup"><span data-stu-id="c2fc9-180">Example</span></span>

<span data-ttu-id="c2fc9-181">Lors de la création d’un service cloud sur Azure, il est possible de consolider le traitement effectué par plusieurs tâches dans un rôle unique.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-181">When building a cloud service on Azure, it’s possible to consolidate the processing performed by multiple tasks into a single role.</span></span> <span data-ttu-id="c2fc9-182">Il s’agit généralement d’un rôle de travail qui exécute des tâches de traitement asynchrone ou en arrière-plan.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-182">Typically this is a worker role that performs background or asynchronous processing tasks.</span></span>

> <span data-ttu-id="c2fc9-183">Dans certains cas, il est possible d’inclure les tâches de traitement asynchrone ou en arrière-plan dans le rôle Web.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-183">In some cases it's possible to include background or asynchronous processing tasks in the web role.</span></span> <span data-ttu-id="c2fc9-184">Cette technique permet de réduire les coûts et de simplifier le déploiement, même si elle peut avoir un impact sur l’extensibilité et la réactivité de l’interface publique fournie par le rôle Web.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-184">This technique helps to reduce costs and simplify deployment, although it can impact the scalability and responsiveness of the public-facing interface provided by the web role.</span></span> 

<span data-ttu-id="c2fc9-185">Le rôle est responsable du démarrage et de l’arrêt des tâches.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-185">The role is responsible for starting and stopping the tasks.</span></span> <span data-ttu-id="c2fc9-186">Lorsque le contrôleur de structure Azure charge un rôle, il déclenche l’événement `Start` pour le rôle.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-186">When the Azure fabric controller loads a role, it raises the `Start` event for the role.</span></span> <span data-ttu-id="c2fc9-187">Vous pouvez remplacer la méthode `OnStart` de la classe `WebRole` ou `WorkerRole` pour gérer cet événement, par exemple, pour initialiser les données et autres ressources dont les tâches de cette méthode dépendent.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-187">You can override the `OnStart` method of the `WebRole` or `WorkerRole` class to handle this event, perhaps to initialize the data and other resources the tasks in this method depend on.</span></span>

<span data-ttu-id="c2fc9-188">Lorsque la méthode `OnStart` se termine, le rôle peut commencer à répondre aux requêtes.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-188">When the `OnStart` method completes, the role can start responding to requests.</span></span> <span data-ttu-id="c2fc9-189">Vous pouvez obtenir plus d’informations et de conseils sur l’utilisation des méthodes `OnStart` et `Run` dans un rôle en consultant la section [Application Startup Processes](https://msdn.microsoft.com/library/ff803371.aspx#sec16) (Processus de démarrage d’application) dans le guide des pratiques et modèles [Moving Applications to the Cloud](https://msdn.microsoft.com/library/ff728592.aspx) (Déplacer des applications dans le cloud).</span><span class="sxs-lookup"><span data-stu-id="c2fc9-189">You can find more information and guidance about using the `OnStart` and `Run` methods in a role in the [Application Startup Processes](https://msdn.microsoft.com/library/ff803371.aspx#sec16) section in the patterns & practices guide [Moving Applications to the Cloud](https://msdn.microsoft.com/library/ff728592.aspx).</span></span>

> <span data-ttu-id="c2fc9-190">Gardez le code de la méthode `OnStart` aussi concis que possible.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-190">Keep the code in the `OnStart` method as concise as possible.</span></span> <span data-ttu-id="c2fc9-191">Azure n’impose aucune limite quant au temps nécessaire pour que cette méthode se termine, mais le rôle ne pourra pas commencer à répondre aux requêtes réseau qui lui sont envoyées tant que cette méthode n’est pas terminée.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-191">Azure doesn't impose any limit on the time taken for this method to complete, but the role won't be able to start responding to network requests sent to it until this method completes.</span></span>

<span data-ttu-id="c2fc9-192">Lorsque la méthode `OnStart` est terminée, le rôle exécute la méthode `Run`.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-192">When the `OnStart` method has finished, the role executes the `Run` method.</span></span> <span data-ttu-id="c2fc9-193">À ce stade, le contrôleur de structure peut démarrer l’envoi de requêtes au rôle.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-193">At this point, the fabric controller can start sending requests to the role.</span></span>

<span data-ttu-id="c2fc9-194">Placez le code qui crée les tâches dans la méthode `Run`.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-194">Place the code that actually creates the tasks in the `Run` method.</span></span> <span data-ttu-id="c2fc9-195">Notez que la méthode `Run` définit la durée de vie de l’instance de rôle.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-195">Note that the `Run` method defines the lifetime of the role instance.</span></span> <span data-ttu-id="c2fc9-196">Lorsque cette méthode est terminée, le contrôleur de structure fera en sorte d’arrêter le rôle.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-196">When this method completes, the fabric controller will arrange for the role to be shut down.</span></span>

<span data-ttu-id="c2fc9-197">Lorsqu’un rôle s’arrête ou est recyclé, le contrôleur de structure empêche la réception d’autres requêtes entrantes de l’équilibreur de charge et déclenche l’événement `Stop`.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-197">When a role shuts down or is recycled, the fabric controller prevents any more incoming requests being received from the load balancer and raises the `Stop` event.</span></span> <span data-ttu-id="c2fc9-198">Vous pouvez capturer cet événement en remplaçant la méthode `OnStop` du rôle et effectuer une réorganisation avant la fin du rôle.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-198">You can capture this event by overriding the `OnStop` method of the role and perform any tidying up required before the role terminates.</span></span>

> <span data-ttu-id="c2fc9-199">Les actions de la méthode `OnStop` doivent être effectuées dans un délai de cinq minutes (ou 30 secondes si vous utilisez l’émulateur Azure sur un ordinateur local).</span><span class="sxs-lookup"><span data-stu-id="c2fc9-199">Any actions performed in the `OnStop` method must be completed within five minutes (or 30 seconds if you are using the Azure emulator on a local computer).</span></span> <span data-ttu-id="c2fc9-200">Dans le cas contraire, le contrôleur de structure Azure suppose que le rôle est bloqué et va forcer son arrêt.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-200">Otherwise the Azure fabric controller assumes that the role has stalled and will force it to stop.</span></span>

<span data-ttu-id="c2fc9-201">Les tâches sont démarrées par la méthode `Run` qui attend que les tâches se terminent.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-201">The tasks are started by the `Run` method that waits for the tasks to complete.</span></span> <span data-ttu-id="c2fc9-202">Les tâches implémentent la logique métier du service cloud et peuvent répondre aux messages publiés dans le rôle via l’équilibreur de charge Azure.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-202">The tasks implement the business logic of the cloud service, and can respond to messages posted to the role through the Azure load balancer.</span></span> <span data-ttu-id="c2fc9-203">La figure illustre le cycle de vie des tâches et des ressources dans un rôle d’un service cloud Azure.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-203">The figure shows the lifecycle of tasks and resources in a role in an Azure cloud service.</span></span>

![Le cycle de vie des tâches et des ressources dans un rôle d’un service cloud Azure](./_images/compute-resource-consolidation-lifecycle.png)


<span data-ttu-id="c2fc9-205">Le fichier _WorkerRole.cs_ du projet _ComputeResourceConsolidation.Worker_ montre un exemple d’implémentation de ce modèle dans un service cloud Azure.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-205">The _WorkerRole.cs_ file in the _ComputeResourceConsolidation.Worker_ project shows an example of how you might implement this pattern in an Azure cloud service.</span></span>

> <span data-ttu-id="c2fc9-206">Le projet _ComputeResourceConsolidation.Worker_ fait partie de la solution _ComputeResourceConsolidation_ disponible au téléchargement sur [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation).</span><span class="sxs-lookup"><span data-stu-id="c2fc9-206">The _ComputeResourceConsolidation.Worker_ project is part of the _ComputeResourceConsolidation_ solution available for download from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation).</span></span>

<span data-ttu-id="c2fc9-207">Les méthodes `MyWorkerTask1` et `MyWorkerTask2` montrent comment effectuer différentes tâches dans le même rôle de travail.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-207">The `MyWorkerTask1` and the `MyWorkerTask2` methods illustrate how to perform different tasks within the same worker role.</span></span> <span data-ttu-id="c2fc9-208">Le code ci-après présente `MyWorkerTask1`.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-208">The following code shows `MyWorkerTask1`.</span></span> <span data-ttu-id="c2fc9-209">Il s’agit d’une tâche simple qui reste inactive pendant 30 secondes avant de générer un message de trace.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-209">This is a simple task that sleeps for 30 seconds and then outputs a trace message.</span></span> <span data-ttu-id="c2fc9-210">Ce processus est répété jusqu’à ce que la tâche soit annulée.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-210">It repeats this process until the task is canceled.</span></span> <span data-ttu-id="c2fc9-211">Le code dans `MyWorkerTask2` est similaire.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-211">The code in `MyWorkerTask2` is similar.</span></span>

```csharp
// A sample worker role task.
private static async Task MyWorkerTask1(CancellationToken ct)
{
  // Fixed interval to wake up and check for work and/or do work.
  var interval = TimeSpan.FromSeconds(30);

  try
  {
    while (!ct.IsCancellationRequested)
    {
      // Wake up and do some background processing if not canceled.
      // TASK PROCESSING CODE HERE
      Trace.TraceInformation("Doing Worker Task 1 Work");

      // Go back to sleep for a period of time unless asked to cancel.
      // Task.Delay will throw an OperationCanceledException when canceled.
      await Task.Delay(interval, ct);
    }
  }
  catch (OperationCanceledException)
  {
    // Expect this exception to be thrown in normal circumstances or check
    // the cancellation token. If the role instances are shutting down, a
    // cancellation request will be signaled.
    Trace.TraceInformation("Stopping service, cancellation requested");

    // Rethrow the exception.
    throw;
  }
}
```

> <span data-ttu-id="c2fc9-212">L’exemple de code montre une implémentation courante d’un processus en arrière-plan.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-212">The sample code shows a common implementation of a background process.</span></span> <span data-ttu-id="c2fc9-213">Dans une application réelle, vous pouvez suivre cette même structure, à ceci près que vous devez placer votre propre logique de traitement dans le corps de la boucle qui attend la requête d’annulation.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-213">In a real world application you can follow this same structure, except that you should place your own processing logic in the body of the loop that waits for the cancellation request.</span></span>

<span data-ttu-id="c2fc9-214">Une fois que le rôle de travail a initialisé les ressources qu’il utilise, la méthode `Run` démarre deux tâches simultanément, comme illustré ici.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-214">After the worker role has initialized the resources it uses, the `Run` method starts the two tasks concurrently, as shown here.</span></span>

```csharp
/// <summary>
/// The cancellation token source use to cooperatively cancel running tasks
/// </summary>
private readonly CancellationTokenSource cts = new CancellationTokenSource();

/// <summary>
/// List of running tasks on the role instance
/// </summary>
private readonly List<Task> tasks = new List<Task>();

// RoleEntry Run() is called after OnStart().
// Returning from Run() will cause a role instance to recycle.
public override void Run()
{
  // Start worker tasks and add to the task list
  tasks.Add(MyWorkerTask1(cts.Token));
  tasks.Add(MyWorkerTask2(cts.Token));

  foreach (var worker in this.workerTasks)
  {
      this.tasks.Add(worker);
  }

  Trace.TraceInformation("Worker host tasks started");
  // The assumption is that all tasks should remain running and not return,
  // similar to role entry Run() behavior.
  try
  {
    Task.WaitAll(tasks.ToArray());
  }
  catch (AggregateException ex)
  {
    Trace.TraceError(ex.Message);

    // If any of the inner exceptions in the aggregate exception
    // are not cancellation exceptions then re-throw the exception.
    ex.Handle(innerEx => (innerEx is OperationCanceledException));
  }

  // If there wasn't a cancellation request, stop all tasks and return from Run()
  // An alternative to canceling and returning when a task exits would be to
  // restart the task.
  if (!cts.IsCancellationRequested)
  {
    Trace.TraceInformation("Task returned without cancellation request");
    Stop(TimeSpan.FromMinutes(5));
  }
}
...
```

<span data-ttu-id="c2fc9-215">Dans cet exemple, la méthode `Run` attend la fin des tâches.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-215">In this example, the `Run` method waits for tasks to be completed.</span></span> <span data-ttu-id="c2fc9-216">Si une tâche est annulée, la méthode `Run` part du principe que le rôle est en cours d’arrêt et attend que les tâches restantes soient annulées avant la fin (elle attend cinq minutes maximum avant de s’arrêter).</span><span class="sxs-lookup"><span data-stu-id="c2fc9-216">If a task is canceled, the `Run` method assumes that the role is being shut down and waits for the remaining tasks to be canceled before finishing (it waits for a maximum of five minutes before terminating).</span></span> <span data-ttu-id="c2fc9-217">Si une tâche échoue en raison d’une exception attendue, la méthode `Run` annule la tâche.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-217">If a task fails due to an expected exception, the `Run` method cancels the task.</span></span>

> <span data-ttu-id="c2fc9-218">Vous pouvez implémenter des stratégies de gestion des exceptions et de surveillance plus complètes dans la méthode `Run` comme le redémarrage des tâches qui ont échoué ou l’intégration du code qui permet au rôle d’arrêter et de démarrer des tâches individuelles.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-218">You could implement more comprehensive monitoring and exception handling strategies in the `Run` method such as restarting tasks that have failed, or including code that enables the role to stop and start individual tasks.</span></span>

<span data-ttu-id="c2fc9-219">La méthode `Stop` présentée dans le code suivant est appelée lorsque le contrôleur de structure arrête l’instance de rôle (elle est appelée à partir de la méthode `OnStop`).</span><span class="sxs-lookup"><span data-stu-id="c2fc9-219">The `Stop` method shown in the following code is called when the fabric controller shuts down the role instance (it's invoked from the `OnStop` method).</span></span> <span data-ttu-id="c2fc9-220">Le code arrête correctement chaque tâche en l’annulant.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-220">The code stops each task gracefully by canceling it.</span></span> <span data-ttu-id="c2fc9-221">Si une tâche prend plus de cinq minutes pour se terminer, le traitement de l’annulation dans la méthode `Stop` cesse d’attendre et le rôle est terminé.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-221">If any task takes more than five minutes to complete, the cancellation processing in the `Stop` method ceases waiting and the role is terminated.</span></span>

```csharp
// Stop running tasks and wait for tasks to complete before returning
// unless the timeout expires.
private void Stop(TimeSpan timeout)
{
  Trace.TraceInformation("Stop called. Canceling tasks.");
  // Cancel running tasks.
  cts.Cancel();

  Trace.TraceInformation("Waiting for canceled tasks to finish and return");

  // Wait for all the tasks to complete before returning. Note that the
  // emulator currently allows 30 seconds and Azure allows five
  // minutes for processing to complete.
  try
  {
    Task.WaitAll(tasks.ToArray(), timeout);
  }
  catch (AggregateException ex)
  {
    Trace.TraceError(ex.Message);

    // If any of the inner exceptions in the aggregate exception
    // are not cancellation exceptions then rethrow the exception.
    ex.Handle(innerEx => (innerEx is OperationCanceledException));
  }
}
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="c2fc9-222">Conseils et modèles connexes</span><span class="sxs-lookup"><span data-stu-id="c2fc9-222">Related patterns and guidance</span></span>

<span data-ttu-id="c2fc9-223">Les modèles et les conseils suivants peuvent aussi présenter un intérêt quand il s’agit d’implémenter ce modèle :</span><span class="sxs-lookup"><span data-stu-id="c2fc9-223">The following patterns and guidance might also be relevant when implementing this pattern:</span></span>

- <span data-ttu-id="c2fc9-224">[Mise à l’échelle automatique](https://msdn.microsoft.com/library/dn589774.aspx).</span><span class="sxs-lookup"><span data-stu-id="c2fc9-224">[Autoscaling Guidance](https://msdn.microsoft.com/library/dn589774.aspx).</span></span> <span data-ttu-id="c2fc9-225">La mise à l’échelle automatique peut être utilisée pour démarrer et arrêter des instances de service qui hébergent des ressources de calcul, en fonction de la demande prévue pour le traitement.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-225">Autoscaling can be used to start and stop instances of service hosting computational resources, depending on the anticipated demand for processing.</span></span>

- <span data-ttu-id="c2fc9-226">[Compute Partitioning Guidance](https://msdn.microsoft.com/library/dn589773.aspx) (Conseils sur le partitionnement du calcul).</span><span class="sxs-lookup"><span data-stu-id="c2fc9-226">[Compute Partitioning Guidance](https://msdn.microsoft.com/library/dn589773.aspx).</span></span> <span data-ttu-id="c2fc9-227">Cet article explique comment allouer les services et composants d’un service cloud pour minimiser les coûts de fonctionnement tout en préservant l’extensibilité, les performances, la disponibilité et la sécurité du service.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-227">Describes how to allocate the services and components in a cloud service in a way that helps to minimize running costs while maintaining the scalability, performance, availability, and security of the service.</span></span>

- <span data-ttu-id="c2fc9-228">Ce modèle comprend un [exemple d’application](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation) téléchargeable.</span><span class="sxs-lookup"><span data-stu-id="c2fc9-228">This pattern includes a downloadable [sample application](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation).</span></span>
