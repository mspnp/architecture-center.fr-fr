---
title: Transaction de compensation
description: "Annulez le travail effectué par une série d’étapes qui définissent ensemble une opération cohérente."
keywords: "modèle de conception"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories: resiliency
ms.openlocfilehash: f8337717c4afd6b558f0da8e1ded3a8071340db7
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2017
---
# <a name="compensating-transaction-pattern"></a><span data-ttu-id="2d1e4-104">Modèle de transaction de compensation</span><span class="sxs-lookup"><span data-stu-id="2d1e4-104">Compensating Transaction pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="2d1e4-105">Annulez le travail effectué par une série d’étapes, qui définissent ensemble une opération cohérente, dans le cas où une ou plusieurs de ces étapes échouent.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-105">Undo the work performed by a series of steps, which together define an eventually consistent operation, if one or more of the steps fail.</span></span> <span data-ttu-id="2d1e4-106">Les opérations qui suivent le modèle de cohérence éventuelle sont couramment rencontrées dans les applications hébergées dans le cloud qui implémentent des flux de travail et des processus professionnels complexes.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-106">Operations that follow the eventual consistency model are commonly found in cloud-hosted applications that implement complex business processes and workflows.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="2d1e4-107">Contexte et problème</span><span class="sxs-lookup"><span data-stu-id="2d1e4-107">Context and problem</span></span>

<span data-ttu-id="2d1e4-108">Les applications qui s’exécutent dans le cloud modifient fréquemment des données.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-108">Applications running in the cloud frequently modify data.</span></span> <span data-ttu-id="2d1e4-109">Ces données peuvent provenir de diverses sources de données réparties dans différents emplacements géographiques.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-109">This data might be spread across various data sources held in different geographic locations.</span></span> <span data-ttu-id="2d1e4-110">Pour éviter la contention et améliorer les performances dans un environnement distribué, une application ne devrait pas tenter de garantir une cohérence transactionnelle élevée.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-110">To avoid contention and improve performance in a distributed environment, an application shouldn't try to provide strong transactional consistency.</span></span> <span data-ttu-id="2d1e4-111">Au lieu de cela, l’application devrait implémenter une cohérence éventuelle.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-111">Rather, the application should implement eventual consistency.</span></span> <span data-ttu-id="2d1e4-112">Dans ce modèle, une opération métier type se compose d’une série d’étapes distinctes.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-112">In this model, a typical business operation consists of a series of separate steps.</span></span> <span data-ttu-id="2d1e4-113">Pendant l’exécution de ces étapes, la vue d’ensemble de l’état du système peut être incohérente, mais lorsque l’opération est terminée et que toutes les étapes ont été exécutées, le système devrait à nouveau devenir cohérent.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-113">While these steps are being performed, the overall view of the system state might be inconsistent, but when the operation has completed and all of the steps have been executed the system should become consistent again.</span></span>

> <span data-ttu-id="2d1e4-114">Le [Manuel d’introduction à la cohérence des données](https://msdn.microsoft.com/library/dn589800.aspx) explique pourquoi les transactions distribuées n’offrent que peu d’évolutivité et présente les principes du modèle de cohérence éventuelle.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-114">The [Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx) provides information about why distributed transactions don't scale well, and the principles of the eventual consistency model.</span></span>

<span data-ttu-id="2d1e4-115">L’une des difficultés du modèle de cohérence éventuelle est la gestion d’une étape qui a échoué.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-115">A challenge in the eventual consistency model is how to handle a step that has failed.</span></span> <span data-ttu-id="2d1e4-116">Dans ce cas, il peut être nécessaire d’annuler tout le travail effectué par les étapes précédentes de l’opération.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-116">In this case it might be necessary to undo all of the work completed by the previous steps in the operation.</span></span> <span data-ttu-id="2d1e4-117">Toutefois, les données ne peuvent pas simplement être restaurées car elles peuvent avoir été modifiées par d’autres instances simultanées de l’application.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-117">However, the data can't simply be rolled back because other concurrent instances of the application might have changed it.</span></span> <span data-ttu-id="2d1e4-118">Même dans les cas où les données n’ont pas été modifiées par une instance simultanée, l’annulation d’une étape ne suffira peut-être pas à restaurer l’état d’origine.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-118">Even in cases where the data hasn't been changed by a concurrent instance, undoing a step might not simply be a matter of restoring the original state.</span></span> <span data-ttu-id="2d1e4-119">Il peut être nécessaire d’appliquer des règles spécifiques à l’entreprise (reportez-vous à l’exemple du site web de voyage dans la section Exemple).</span><span class="sxs-lookup"><span data-stu-id="2d1e4-119">It might be necessary to apply various business-specific rules (see the travel website described in the Example section).</span></span>

<span data-ttu-id="2d1e4-120">Si une opération qui implémente la cohérence éventuelle s’étend sur plusieurs banques de données hétérogènes, l’annulation des différentes étapes de l’opération nécessitera d’accéder à chacune des banques de données.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-120">If an operation that implements eventual consistency spans several heterogeneous data stores, undoing the steps in the operation will require visiting each data store in turn.</span></span> <span data-ttu-id="2d1e4-121">Le travail effectué dans chaque banque de données doit être annulé de manière fiable pour rétablir la cohérence du système.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-121">The work performed in every data store must be undone reliably to prevent the system from remaining inconsistent.</span></span>

<span data-ttu-id="2d1e4-122">Il arrive qu’une base de données ne contienne pas toutes les données affectées par une opération qui implémente la cohérence éventuelle.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-122">Not all data affected by an operation that implements eventual consistency might be held in a database.</span></span> <span data-ttu-id="2d1e4-123">Dans un environnement d’architecture orientée services, une opération peut appeler une action dans un service et entraîner une modification de l’état de ce service.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-123">In a service oriented architecture (SOA) environment an operation could invoke an action in a service, and cause a change in the state held by that service.</span></span> <span data-ttu-id="2d1e4-124">Pour annuler l’opération, ce changement d’état doit également être annulé.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-124">To undo the operation, this state change must also be undone.</span></span> <span data-ttu-id="2d1e4-125">Cela peut nécessiter d’appeler à nouveau le service et d’effectuer une autre action qui annule les effets de la première action.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-125">This can involve invoking the service again and performing another action that reverses the effects of the first.</span></span>

## <a name="solution"></a><span data-ttu-id="2d1e4-126">Solution</span><span class="sxs-lookup"><span data-stu-id="2d1e4-126">Solution</span></span>

<span data-ttu-id="2d1e4-127">La solution consiste à implémenter une transaction de compensation.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-127">The solution is to implement a compensating transaction.</span></span> <span data-ttu-id="2d1e4-128">Les étapes d’une transaction de compensation doivent annuler les effets des étapes de l’opération d’origine.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-128">The steps in a compensating transaction must undo the effects of the steps in the original operation.</span></span> <span data-ttu-id="2d1e4-129">Une transaction de compensation ne peut pas simplement remplacer l’état actuel par l’état dans lequel se trouvait le système au début de l’opération car cette approche pourrait annuler les modifications apportées par d’autres instances simultanées d’une application.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-129">A compensating transaction might not be able to simply replace the current state with the state the system was in at the start of the operation because this approach could overwrite changes made by other concurrent instances of an application.</span></span> <span data-ttu-id="2d1e4-130">Au lieu de cela, il faut mettre en place un processus intelligent qui prend en compte toute tâche effectuée par les instances simultanées.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-130">Instead, it must be an intelligent process that takes into account any work done by concurrent instances.</span></span> <span data-ttu-id="2d1e4-131">Ce processus varie généralement en fonction de l’application et de la nature du travail effectué par l’opération d’origine.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-131">This process will usually be application specific, driven by the nature of the work performed by the original operation.</span></span>

<span data-ttu-id="2d1e4-132">Une approche courante consiste à utiliser un flux de travail pour implémenter une opération cohérente qui nécessite une compensation.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-132">A common approach is to use a workflow to implement an eventually consistent operation that requires compensation.</span></span> <span data-ttu-id="2d1e4-133">Au cours de l’exécution de l’opération d’origine, le système enregistre des informations sur chaque étape et identifie la manière dont le travail effectué par l’étape en cours peut être annulé.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-133">As the original operation proceeds, the system records information about each step and how the work performed by that step can be undone.</span></span> <span data-ttu-id="2d1e4-134">Si l’opération échoue à un moment donné, le flux de travail effectue un retour en arrière sur les étapes effectuées et annule chacune de ces étapes.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-134">If the operation fails at any point, the workflow rewinds back through the steps it's completed and performs the work that reverses each step.</span></span> <span data-ttu-id="2d1e4-135">Notez qu’une transaction de compensation n’annule pas forcément les étapes dans l’ordre où elles ont été effectuées dans l’opération d’origine et qu’il est possible d’effectuer certaines des étapes d’annulation en parallèle.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-135">Note that a compensating transaction might not have to undo the work in the exact reverse order of the original operation, and it might be possible to perform some of the undo steps in parallel.</span></span>

> <span data-ttu-id="2d1e4-136">Cette approche est similaire à la stratégie Sagas présentée sur [le blog de Clemens Vasters](http://vasters.com/clemensv/2012/09/01/Sagas.aspx).</span><span class="sxs-lookup"><span data-stu-id="2d1e4-136">This approach is similar to the Sagas strategy discussed in [Clemens Vasters’ blog](http://vasters.com/clemensv/2012/09/01/Sagas.aspx).</span></span>

<span data-ttu-id="2d1e4-137">Une transaction de compensation est également une opération de cohérence éventuelle qui peut échouer.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-137">A compensating transaction is also an eventually consistent operation and it could also fail.</span></span> <span data-ttu-id="2d1e4-138">Le système doit être en mesure de relancer la transaction de compensation à partir du point de défaillance et de poursuivre son exécution.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-138">The system should be able to resume the compensating transaction at the point of failure and continue.</span></span> <span data-ttu-id="2d1e4-139">Il peut être nécessaire de répéter une étape qui a échoué. Les étapes d’une transaction de compensation doivent donc être définies en tant que commandes idempotentes.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-139">It might be necessary to repeat a step that's failed, so the steps in a compensating transaction should be defined as idempotent commands.</span></span> <span data-ttu-id="2d1e4-140">Pour plus d’informations, consultez l’article [Idempotency Patterns](http://blog.jonathanoliver.com/2010/04/idempotency-patterns/) sur le blog de Jonathan Oliver.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-140">For more information, see [Idempotency Patterns](http://blog.jonathanoliver.com/2010/04/idempotency-patterns/) on Jonathan Oliver’s blog.</span></span>

<span data-ttu-id="2d1e4-141">Dans certains cas, il n’est pas possible de reprendre l’opération à partir d’une étape qui a échoué, sauf via une intervention manuelle.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-141">In some cases it might not be possible to recover from a step that has failed except through manual intervention.</span></span> <span data-ttu-id="2d1e4-142">Dans ces situations, le système doit générer une alerte et fournir autant d’informations que possible sur la raison de l’échec.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-142">In these situations the system should raise an alert and provide as much information as possible about the reason for the failure.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="2d1e4-143">Problèmes et considérations</span><span class="sxs-lookup"><span data-stu-id="2d1e4-143">Issues and considerations</span></span>

<span data-ttu-id="2d1e4-144">Prenez en compte les points suivants lorsque vous choisissez comment implémenter ce modèle :</span><span class="sxs-lookup"><span data-stu-id="2d1e4-144">Consider the following points when deciding how to implement this pattern:</span></span>

<span data-ttu-id="2d1e4-145">Il n’est pas toujours facile de déterminer quand une étape d’une opération qui implémente la cohérence éventuelle a échoué.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-145">It might not be easy to determine when a step in an operation that implements eventual consistency has failed.</span></span> <span data-ttu-id="2d1e4-146">Une étape peut ne pas échouer immédiatement mais être bloquée.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-146">A step might not fail immediately, but instead could block.</span></span> <span data-ttu-id="2d1e4-147">Il peut alors être nécessaire d’implémenter un mécanisme impliquant un délai d’expiration.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-147">It might be necessary to implement some form of time-out mechanism.</span></span>

<span data-ttu-id="2d1e4-148">-Une logique de compensation peut difficilement être généralisée.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-148">-Compensation logic isn't easily generalized.</span></span> <span data-ttu-id="2d1e4-149">La transaction de compensation varie en fonction de chaque application.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-149">A compensating transaction is application specific.</span></span> <span data-ttu-id="2d1e4-150">Elle s’appuie sur les informations dont dispose l’application pour pouvoir annuler les effets de chaque étape d’une opération ayant échoué.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-150">It relies on the application having sufficient information to be able to undo the effects of each step in a failed operation.</span></span>

<span data-ttu-id="2d1e4-151">Vous devez définir les étapes d’une transaction de compensation en tant que commandes idempotentes.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-151">You should define the steps in a compensating transaction as idempotent commands.</span></span> <span data-ttu-id="2d1e4-152">Ainsi, les étapes pourront être répétées même si la transaction de compensation échoue.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-152">This enables the steps to be repeated if the compensating transaction itself fails.</span></span>

<span data-ttu-id="2d1e4-153">L’infrastructure qui gère les étapes de l’opération d’origine et de la transaction de compensation doit être résiliente.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-153">The infrastructure that handles the steps in the original operation, and the compensating transaction, must be resilient.</span></span> <span data-ttu-id="2d1e4-154">Elle ne doit pas perdre les informations requises pour compenser une étape ayant échoué, et elle doit être en mesure de contrôler de manière fiable la progression de la logique de compensation.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-154">It must not lose the information required to compensate for a failing step, and it must be able to reliably monitor the progress of the compensation logic.</span></span>

<span data-ttu-id="2d1e4-155">Une transaction de compensation ne rétablit pas forcément l’état des données dans le système tel qu’il l’était au début de l’opération d’origine.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-155">A compensating transaction doesn't necessarily return the data in the system to the state it was in at the start of the original operation.</span></span> <span data-ttu-id="2d1e4-156">Au lieu de cela, elle compense le travail effectué par les étapes qui ont été exécutées avec succès avant l’échec de l’opération.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-156">Instead, it compensates for the work performed by the steps that completed successfully before the operation failed.</span></span>

<span data-ttu-id="2d1e4-157">Il n’est pas nécessaire que les étapes de la transaction de compensation soient effectuées dans l’ordre inverse des étapes de l’opération d’origine.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-157">The order of the steps in the compensating transaction doesn't necessarily have to be the exact opposite of the steps in the original operation.</span></span> <span data-ttu-id="2d1e4-158">Par exemple, une banque de données peut être plus sensible aux incohérences qu’une autre. Par conséquent, les étapes de la transaction de compensation qui annulent les modifications apportées à cette banque de données devront être exécutées en premier.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-158">For example, one data store might be more sensitive to inconsistencies than another, and so the steps in the compensating transaction that undo the changes to this store should occur first.</span></span>

<span data-ttu-id="2d1e4-159">Pour augmenter les chances de réussite de l’opération de compensation, il peut être judicieux d’appliquer un verrou avec un délai d’expiration court à chaque ressource requise pour exécuter une opération et d’obtenir ces ressources en amont.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-159">Placing a short-term timeout-based lock on each resource that's required to complete an operation, and obtaining these resources in advance, can help increase the likelihood that the overall activity will succeed.</span></span> <span data-ttu-id="2d1e4-160">Le travail doit être effectué uniquement une fois toutes les ressources obtenues.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-160">The work should be performed only after all the resources have been acquired.</span></span> <span data-ttu-id="2d1e4-161">Toutes les actions doivent être finalisées avant expiration des verrous.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-161">All actions must be finalized before the locks expire.</span></span>

<span data-ttu-id="2d1e4-162">Envisagez d’utiliser la logique de nouvelle tentative qui est généralement plus indulgente pour limiter les erreurs qui déclenchent une transaction de compensation.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-162">Consider using retry logic that is more forgiving than usual to minimize failures that trigger a compensating transaction.</span></span> <span data-ttu-id="2d1e4-163">Lorsqu’une étape d’une opération qui implémente la cohérence éventuelle échoue, essayez de traiter l’erreur comme une exception temporaire et répétez l’étape.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-163">If a step in an operation that implements eventual consistency fails, try handling the failure as a transient exception and repeat the step.</span></span> <span data-ttu-id="2d1e4-164">Arrêtez l’opération et démarrez une transaction de compensation uniquement si une étape échoue à plusieurs reprises ou de manière définitive.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-164">Only stop the operation and initiate a compensating transaction if a step fails repeatedly or irrecoverably.</span></span>

> <span data-ttu-id="2d1e4-165">Les difficultés de mise en œuvre d’une transaction de compensation sont quasiment les mêmes que pour l’implémentation de la cohérence éventuelle.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-165">Many of the challenges of implementing a compensating transaction are the same as those with implementing eventual consistency.</span></span> <span data-ttu-id="2d1e4-166">Pour en savoir plus, reportez-vous à la section Considerations for Implementing Eventual Consistency (Considérations relatives à l’implémentation de la cohérence éventuelle) du [Manuel d’introduction à la cohérence des données](https://msdn.microsoft.com/library/dn589800.aspx).</span><span class="sxs-lookup"><span data-stu-id="2d1e4-166">See the section Considerations for Implementing Eventual Consistency in the [Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx) for more information.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="2d1e4-167">Quand utiliser ce modèle</span><span class="sxs-lookup"><span data-stu-id="2d1e4-167">When to use this pattern</span></span>

<span data-ttu-id="2d1e4-168">Utilisez ce modèle uniquement pour les opérations qui doivent être annulées si elles échouent.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-168">Use this pattern only for operations that must be undone if they fail.</span></span> <span data-ttu-id="2d1e4-169">Si possible, concevez vos solutions de manière à ne pas avoir à gérer la complexité liée aux transactions de compensation.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-169">If possible, design solutions to avoid the complexity of requiring compensating transactions.</span></span>

## <a name="example"></a><span data-ttu-id="2d1e4-170">Exemple</span><span class="sxs-lookup"><span data-stu-id="2d1e4-170">Example</span></span>

<span data-ttu-id="2d1e4-171">Prenons l’exemple d’un site web sur lequel les clients peuvent réserver leur itinéraire de voyage.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-171">A travel website lets customers book itineraries.</span></span> <span data-ttu-id="2d1e4-172">Un seul itinéraire peut comprendre plusieurs vols et hôtels.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-172">A single itinerary might comprise a series of flights and hotels.</span></span> <span data-ttu-id="2d1e4-173">Un client qui part de Seattle en direction de Londres, puis de Paris, pourrait suivre les étapes suivantes pour créer son itinéraire :</span><span class="sxs-lookup"><span data-stu-id="2d1e4-173">A customer traveling from Seattle to London and then on to Paris could perform the following steps when creating an itinerary:</span></span>

1. <span data-ttu-id="2d1e4-174">Réservation d’un siège sur le vol F1 reliant Seattle à Londres.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-174">Book a seat on flight F1 from Seattle to London.</span></span>
2. <span data-ttu-id="2d1e4-175">Réservation d’un siège sur le vol F2 reliant Londres à Paris.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-175">Book a seat on flight F2 from London to Paris.</span></span>
3. <span data-ttu-id="2d1e4-176">Réservation d’un siège sur le vol F3 reliant Paris à Seattle.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-176">Book a seat on flight F3 from Paris to Seattle.</span></span>
4. <span data-ttu-id="2d1e4-177">Réservation d’une chambre à l’hôtel H1 à Londres.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-177">Reserve a room at hotel H1 in London.</span></span>
5. <span data-ttu-id="2d1e4-178">Réservation d’une chambre à l’hôtel H2 à Paris.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-178">Reserve a room at hotel H2 in Paris.</span></span>

<span data-ttu-id="2d1e4-179">Ces étapes constituent une opération de cohérence éventuelle, même si chaque étape correspond à une action distincte.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-179">These steps constitute an eventually consistent operation, although each step is a separate action.</span></span> <span data-ttu-id="2d1e4-180">Par conséquent, outre l’exécution de ces étapes, le système doit enregistrer les opérations de réversion requises pour annuler chaque étape au cas où le client décide d’annuler l’itinéraire.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-180">Therefore, as well as performing these steps, the system must also record the counter operations necessary to undo each step in case the customer decides to cancel the itinerary.</span></span> <span data-ttu-id="2d1e4-181">Les étapes nécessaires pour effectuer les opérations de réversion peuvent ensuite être exécutées dans le cadre d’une transaction de compensation.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-181">The steps necessary to perform the counter operations can then run as a compensating transaction.</span></span>

<span data-ttu-id="2d1e4-182">Notez que les étapes de la transaction de compensation ne seront pas forcément effectuées dans l’ordre inverse des étapes d’origine, et que la logique de chaque étape de la transaction de compensation devra prendre en compte les éventuelles règles inhérentes à l’entreprise.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-182">Notice that the steps in the compensating transaction might not be the exact opposite of the original steps, and the logic in each step in the compensating transaction must take into account any business-specific rules.</span></span> <span data-ttu-id="2d1e4-183">Par exemple, l’annulation de la réservation d’un vol n’impliquera pas forcément un remboursement total de la somme payée par le client.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-183">For example, unbooking a seat on a flight might not entitle the customer to a complete refund of any money paid.</span></span> <span data-ttu-id="2d1e4-184">La figure illustre l’exécution d’une transaction de compensation afin d’annuler une transaction à long terme pour réserver un itinéraire de voyage.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-184">The figure illustrates generating a compensating transaction to undo a long-running transaction to book a travel itinerary.</span></span>

![Exécution d’une transaction de compensation afin d’annuler une transaction à long terme pour réserver un itinéraire de voyage](./_images/compensating-transaction-diagram.png)


> <span data-ttu-id="2d1e4-186">Il est parfois possible d’exécuter en parallèle les étapes de la transaction de compensation. Cela dépend de la façon dont vous avez conçu la logique de compensation pour chaque étape.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-186">It might be possible for the steps in the compensating transaction to be performed in parallel, depending on how you've designed the compensating logic for each step.</span></span>

<span data-ttu-id="2d1e4-187">Dans de nombreuses solutions d’entreprise, l’échec d’une seule étape ne nécessite pas toujours une réinitialisation de l’état d’origine du système via une transaction de compensation.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-187">In many business solutions, failure of a single step doesn't always necessitate rolling the system back by using a compensating transaction.</span></span> <span data-ttu-id="2d1e4-188">Par exemple, si&mdash;après avoir réservé les vols F1, F2 et F3 dans le scénario du site web de voyage&mdash;le client ne parvient pas à réserver une chambre à l’hôtel H1, il est préférable de proposer au client une chambre dans un autre hôtel de la ville plutôt que d’annuler tous les vols.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-188">For example, if&mdash;after having booked flights F1, F2, and F3 in the travel website scenario&mdash;the customer is unable to reserve a room at hotel H1, it's preferable to offer the customer a room at a different hotel in the same city rather than canceling the flights.</span></span> <span data-ttu-id="2d1e4-189">Le client peut toujours décider d’annuler (auquel cas la transaction de compensation s’exécute et annule les réservations effectuées sur les vols F1, F2 et F3), mais cette décision doit être prise par le client et non pas par le système.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-189">The customer can still decide to cancel (in which case the compensating transaction runs and undoes the bookings made on flights F1, F2, and F3), but this decision should be made by the customer rather than by the system.</span></span>

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="2d1e4-190">Conseils et modèles connexes</span><span class="sxs-lookup"><span data-stu-id="2d1e4-190">Related patterns and guidance</span></span>

<span data-ttu-id="2d1e4-191">Les modèles et les conseils suivants peuvent aussi présenter un intérêt quand il s’agit d’implémenter ce modèle :</span><span class="sxs-lookup"><span data-stu-id="2d1e4-191">The following patterns and guidance might also be relevant when implementing this pattern:</span></span>

- <span data-ttu-id="2d1e4-192">[Manuel d’introduction à la cohérence des données](https://msdn.microsoft.com/library/dn589800.aspx).</span><span class="sxs-lookup"><span data-stu-id="2d1e4-192">[Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx).</span></span> <span data-ttu-id="2d1e4-193">Le modèle de transaction de compensation est souvent utilisé pour annuler les opérations qui implémentent le modèle de cohérence éventuelle.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-193">The Compensating Transaction pattern is often used to undo operations that implement the eventual consistency model.</span></span> <span data-ttu-id="2d1e4-194">Ce manuel présente les avantages et les inconvénients de la cohérence éventuelle.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-194">This primer provides information on the benefits and tradeoffs of eventual consistency.</span></span>

- <span data-ttu-id="2d1e4-195">[Modèle Planificateur-Agent-Superviseur](scheduler-agent-supervisor.md).</span><span class="sxs-lookup"><span data-stu-id="2d1e4-195">[Scheduler-Agent-Supervisor Pattern](scheduler-agent-supervisor.md).</span></span> <span data-ttu-id="2d1e4-196">Ce modèle décrit comment implémenter des systèmes résilients qui exécutent des opérations métiers utilisant des ressources et des services distribués.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-196">Describes how to implement resilient systems that perform business operations that use distributed services and resources.</span></span> <span data-ttu-id="2d1e4-197">Parfois, il peut être nécessaire d’annuler le travail effectué par une opération à l’aide d’une transaction de compensation.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-197">Sometimes, it might be necessary to undo the work performed by an operation by using a compensating transaction.</span></span>

- <span data-ttu-id="2d1e4-198">[Modèle Nouvelle tentative](./retry.md).</span><span class="sxs-lookup"><span data-stu-id="2d1e4-198">[Retry Pattern](./retry.md).</span></span> <span data-ttu-id="2d1e4-199">Les transactions de compensation peuvent s’avérer coûteuses. Il est possible de limiter leur utilisation en implémentant une stratégie efficace consistant à relancer les opérations ayant échoué en suivant le modèle Nouvelle tentative.</span><span class="sxs-lookup"><span data-stu-id="2d1e4-199">Compensating transactions can be expensive to perform, and it might be possible to minimize their use by implementing an effective policy of retrying failing operations by following the Retry pattern.</span></span>