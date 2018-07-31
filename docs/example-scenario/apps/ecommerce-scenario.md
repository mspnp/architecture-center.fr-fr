---
title: Serveur frontal e-commerce sur Azure
description: Scénario éprouvé d’hébergement d’un site de e-commerce sur Azure
author: masonch
ms.date: 7/13/18
ms.openlocfilehash: 568821e97c6b90a36429dfa8ec0ef9ed38c7963c
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060970"
---
# <a name="e-commerce-front-end-on-azure"></a><span data-ttu-id="28df0-103">Serveur frontal e-commerce sur Azure</span><span class="sxs-lookup"><span data-stu-id="28df0-103">E-Commerce front-end on Azure</span></span>

<span data-ttu-id="28df0-104">Cet exemple de scénario vous guide tout au long d’une implémentation d’un serveur frontal e-commerce à l’aide des outils de Azure Platform-as-a-Service (PaaS).</span><span class="sxs-lookup"><span data-stu-id="28df0-104">This example scenario walks you through an implementation of an e-commerce front end using Azure Platform-as-a-Service (PaaS) tools.</span></span> <span data-ttu-id="28df0-105">De nombreux sites Web de e-commerce sont confrontés à des variations de la saisonnalité et du trafic au fil du temps.</span><span class="sxs-lookup"><span data-stu-id="28df0-105">Many e-commerce websites face seasonality and traffic variability over time.</span></span> <span data-ttu-id="28df0-106">Quand la demande en produits et services augmente, de façon prévisible ou non, l’utilisation des outils PaaS vous permet de gérer davantage de clients et de transactions automatiquement.</span><span class="sxs-lookup"><span data-stu-id="28df0-106">When demand for your products or services takes off, whether predictably or unpredictably, using PaaS tools will allow you to handle more customers and more transactions automatically.</span></span> <span data-ttu-id="28df0-107">De plus, ce scénario tire parti de l’approche économique du cloud en payant uniquement la capacité que vous utilisez.</span><span class="sxs-lookup"><span data-stu-id="28df0-107">Additionally, this scenario takes advantage of cloud economics by paying only for the capacity you use.</span></span>

<span data-ttu-id="28df0-108">Ce document vous aide à découvrir les différents composants PaaS Azure et les considérations utilisées pour mettre ensemble et pour déployer un exemple d’application de e-commerce, *Relecloud Concerts*, une plateforme de création de tickets de concert en ligne.</span><span class="sxs-lookup"><span data-stu-id="28df0-108">This document will help you will learn about various Azure PaaS components and considerations used to bring together to deploy a sample e-commerce application, *Relecloud Concerts*, an online concert ticketing platform.</span></span>

## <a name="potential-use-cases"></a><span data-ttu-id="28df0-109">Cas d’usage potentiels</span><span class="sxs-lookup"><span data-stu-id="28df0-109">Potential use cases</span></span>

<span data-ttu-id="28df0-110">Pensez à ce scénario pour les cas d’usage suivants :</span><span class="sxs-lookup"><span data-stu-id="28df0-110">Consider this scenario for the following use cases:</span></span>

* <span data-ttu-id="28df0-111">La création d’une application nécessitant une mise à l’échelle élastique pour gérer les pics d’utilisateurs à des moments différents.</span><span class="sxs-lookup"><span data-stu-id="28df0-111">Building an application that needs elastic scale to handle bursts of users at different times.</span></span>
* <span data-ttu-id="28df0-112">La création d’une application conçue pour fonctionner avec une haute disponibilité dans différentes régions Azure du monde entier.</span><span class="sxs-lookup"><span data-stu-id="28df0-112">Building an application that is designed to operate at high availability in different Azure regions around the world.</span></span>

## <a name="architecture"></a><span data-ttu-id="28df0-113">Architecture</span><span class="sxs-lookup"><span data-stu-id="28df0-113">Architecture</span></span>

![Exemple d’architecture de scénario pour une application de e-commerce][architecture-diagram]

<span data-ttu-id="28df0-115">Ce scénario couvre l’achat de tickets à partir d’un site de e-commerce, les données transitent par le scénario comme suit :</span><span class="sxs-lookup"><span data-stu-id="28df0-115">This scenario covers purchasing tickets from an e-commerce site, the data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="28df0-116">Azure Traffic Manager achemine la demande d’un utilisateur vers le site de e-commerce hébergé dans Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="28df0-116">Azure Traffic Manager routes a user's request to the e-commerce site hosted in Azure App Service.</span></span>
2. <span data-ttu-id="28df0-117">Azure CDN sert des images statiques et le contenu à l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="28df0-117">Azure CDN serves static images and content to the user.</span></span>
3. <span data-ttu-id="28df0-118">L’utilisateur se connecte à l’application via un client Azure Active Directory B2C.</span><span class="sxs-lookup"><span data-stu-id="28df0-118">User signs in to the application through an Azure Active Directory B2C tenant.</span></span>
4. <span data-ttu-id="28df0-119">L’utilisateur recherche des concerts à l’aide d’Azure Search.</span><span class="sxs-lookup"><span data-stu-id="28df0-119">User searches for concerts using Azure Search.</span></span>
5. <span data-ttu-id="28df0-120">Le site web extrait les détails du concert à partir d’Azure SQL Database.</span><span class="sxs-lookup"><span data-stu-id="28df0-120">Web site pulls concert details from Azure SQL Database.</span></span> 
6. <span data-ttu-id="28df0-121">Le site web fait référence aux images de tickets achetés dans le stockage Blob.</span><span class="sxs-lookup"><span data-stu-id="28df0-121">Web site refers to purchased ticket images in Blob Storage.</span></span>
7. <span data-ttu-id="28df0-122">Les résultats de requête de la base de données sont mis en cache dans Cache Redis Azure pour offrir de meilleures performances.</span><span class="sxs-lookup"><span data-stu-id="28df0-122">Database query results are cached in Azure Redis Cache for better performance.</span></span>
8. <span data-ttu-id="28df0-123">L’utilisateur envoie des commandes de tickets et des revues de concert qui sont placées dans la file d’attente.</span><span class="sxs-lookup"><span data-stu-id="28df0-123">User submits ticket orders and concert reviews which are placed in the queue.</span></span>
9. <span data-ttu-id="28df0-124">Azure Functions traite le paiement de la commande et les revues de concert.</span><span class="sxs-lookup"><span data-stu-id="28df0-124">Azure Functions processes order payment and concert reviews.</span></span>
10. <span data-ttu-id="28df0-125">Cognitives services fournit une analyse de la revue du concert afin de déterminer le sentiment (positif ou négatif).</span><span class="sxs-lookup"><span data-stu-id="28df0-125">Cognitive services provide an analysis of the concert review to determine the sentiment (positive or negative).</span></span>
11. <span data-ttu-id="28df0-126">Application Insights fournit des métriques de performances pour surveiller l’intégrité de l’application web.</span><span class="sxs-lookup"><span data-stu-id="28df0-126">Application Insights provides performance metrics for monitoring the health of the web application.</span></span>

### <a name="components"></a><span data-ttu-id="28df0-127">Composants</span><span class="sxs-lookup"><span data-stu-id="28df0-127">Components</span></span>

* <span data-ttu-id="28df0-128">[Azure CDN][docs-cdn] fournit le contenu statique, mis en cache à partir d’emplacements proches des utilisateurs pour réduire la latence.</span><span class="sxs-lookup"><span data-stu-id="28df0-128">[Azure CDN][docs-cdn] delivers static, cached content from locations close to users to reduce latency.</span></span>
* <span data-ttu-id="28df0-129">[Azure Traffic Manager][docs-traffic-manager] contrôle la distribution du trafic utilisateur pour les points de terminaison de service dans différentes régions Azure.</span><span class="sxs-lookup"><span data-stu-id="28df0-129">[Azure Traffic Manager][docs-traffic-manager] controls the distribution of user traffic for service endpoints in different Azure regions.</span></span>
* <span data-ttu-id="28df0-130">[App Services - Web Apps][docs-webapps] héberge des applications web permettant une mise à l'échelle automatique et une haute disponibilité sans avoir à gérer l’infrastructure.</span><span class="sxs-lookup"><span data-stu-id="28df0-130">[App Services - Web Apps][docs-webapps] hosts web applications allowing auto-scale and high availability without having to manage infrastructure.</span></span>
* <span data-ttu-id="28df0-131">[Azure Active Directory - B2C][docs-b2c] est un service de gestion des identités qui vous permet de personnaliser et de contrôler la façon dont les clients s’inscrivent, se connectent et gèrent leurs profils dans une application.</span><span class="sxs-lookup"><span data-stu-id="28df0-131">[Azure Active Directory - B2C][docs-b2c] is an identity management service that enables customization and control over how customers sign up, sign in, and manage their profiles in an application.</span></span>
* <span data-ttu-id="28df0-132">[Files d’attente de stockage][docs-storage-queues] stocke un grand nombre de messages de file d’attente accessibles par une application.</span><span class="sxs-lookup"><span data-stu-id="28df0-132">[Storage Queues][docs-storage-queues] stores large numbers of queue messages that can be accessed by an application.</span></span>
* <span data-ttu-id="28df0-133">[Functions][docs-functions] constitue les options de calcul sans serveur qui permettent aux applications d’être exécutées à la demande sans avoir à gérer l’infrastructure.</span><span class="sxs-lookup"><span data-stu-id="28df0-133">[Functions][docs-functions] are serverless compute options that allow applications to run on-demand without having to manage infrastructure.</span></span>
* <span data-ttu-id="28df0-134">[Cognitive Services - Sentiment Analysis][docs-sentiment-analysis] utilise les API de Machine Learning et permet aux développeurs d’ajouter facilement des fonctionnalités intelligentes telles que la détection d’émotion et vidéo, la reconnaissance faciale, vocale et de la vision ainsi que la compréhension vocale et du langage, dans des applications.</span><span class="sxs-lookup"><span data-stu-id="28df0-134">[Cognitive Services - Sentiment Analysis][docs-sentiment-analysis] uses machine learning APIs and enables developers to easily add intelligent features – such as emotion and video detection; facial, speech and vision recognition; and speech and language understanding – into applications.</span></span>
* <span data-ttu-id="28df0-135">La [Recherche Azure][docs-search] est une solution cloud de recherche en tant que service, qui offre une expérience de recherche riche concernant du contenu privé et hétérogène dans les applications web, mobiles et d’entreprise.</span><span class="sxs-lookup"><span data-stu-id="28df0-135">[Azure Search][docs-search] is a search-as-a-service cloud solution that provides a rich search experience over private, heterogenous content in web, mobile, and enterprise applications.</span></span>
* <span data-ttu-id="28df0-136">Le [stockage Blob][docs-storage-blobs] est optimisé pour stocker de grandes quantités de données non structurées, telles que des données texte ou binaires.</span><span class="sxs-lookup"><span data-stu-id="28df0-136">[Storage Blobs][docs-storage-blobs] are optimized to store large amounts of unstructured data, such as text or binary data.</span></span>
* <span data-ttu-id="28df0-137">[Cache redis][docs-redis-cache] améliore les performances et l’évolutivité des systèmes qui s’appuient sur les magasins de données back-end en copiant temporairement des données fréquemment utilisées dans un stockage rapide situé près de l’application.</span><span class="sxs-lookup"><span data-stu-id="28df0-137">[Redis Cache][docs-redis-cache] improves the performance and scalability of systems that rely heavily on backend data-stores by temporarily copying frequently accessed data to fast storage located close to the application.</span></span>
* <span data-ttu-id="28df0-138">[SQL Database][docs-sql-database] est un service administré de bases de données relationnelles à usage général de Microsoft Azure qui prend en charge des structures telles que les données relationnelles, JSON, les données spatiales et XML.</span><span class="sxs-lookup"><span data-stu-id="28df0-138">[SQL Database][docs-sql-database] is a general-purpose relational database managed service in Microsoft Azure that supports structures such as relational data, JSON, spatial, and XML.</span></span>
* <span data-ttu-id="28df0-139">[Application Insights][docs-application-insights] est conçu pour vous aider à améliorer en permanence les performances et la convivialité en détectant automatiquement les anomalies de performances via des outils d’analytique intégrés pour aider à comprendre ce que font les utilisateurs avec une application.</span><span class="sxs-lookup"><span data-stu-id="28df0-139">[Application Insights][docs-application-insights] is designed to help you continuously improve performance and usability by automatically detecting performance anomalies through built-in analytics tools to help understand what users do with an app.</span></span>

### <a name="alternatives"></a><span data-ttu-id="28df0-140">Autres solutions</span><span class="sxs-lookup"><span data-stu-id="28df0-140">Alternatives</span></span>

<span data-ttu-id="28df0-141">Bien d’autres technologies sont disponibles pour créer une application visible par le client consacrée au e-commerce à grande échelle.</span><span class="sxs-lookup"><span data-stu-id="28df0-141">Many other technologies are available for building a customer facing application focused on e-commerce at scale.</span></span> <span data-ttu-id="28df0-142">Elles couvrent le front-end de l’application, ainsi que la couche Données.</span><span class="sxs-lookup"><span data-stu-id="28df0-142">These cover both the front end of the application as well as the data tier.</span></span>

<span data-ttu-id="28df0-143">Les autres options pour la couche Web et les fonctions incluent :</span><span class="sxs-lookup"><span data-stu-id="28df0-143">Other options for the web tier and functions include:</span></span>

* <span data-ttu-id="28df0-144">[Service Fabric][docs-service-fabric] : une plateforme concentrée autour de la création des composants distribués qui bénéficient d’un déploiement et d’une exécution sur un cluster avec un degré élevé de contrôle.</span><span class="sxs-lookup"><span data-stu-id="28df0-144">[Service Fabric][docs-service-fabric] - A platform focused around building distributed components that benefit from being deployed and run across a cluster with a high degree of control.</span></span> <span data-ttu-id="28df0-145">Service Fabric peut également être utilisé pour héberger des conteneurs.</span><span class="sxs-lookup"><span data-stu-id="28df0-145">Service Fabric can also be used to host containers.</span></span>
* <span data-ttu-id="28df0-146">[Azure Kubernetes Service][docs-kubernetes-service] : une plateforme pour la création et le déploiement des solutions basées sur un conteneur qui peuvent être utilisées comme une implémentation d’une architecture de microservices.</span><span class="sxs-lookup"><span data-stu-id="28df0-146">[Azure Kubernetes Service][docs-kubernetes-service] - A platform for building and deploying container based solutions which can be used as one implementation of a microservices architecture.</span></span> <span data-ttu-id="28df0-147">Cela offre une agilité aux différents composants de l’application, pour leur permettre d’être mis à l’échelle indépendamment à la demande.</span><span class="sxs-lookup"><span data-stu-id="28df0-147">This allows for agility of different components of the application to be able to scale independently on demand.</span></span>
* <span data-ttu-id="28df0-148">[Azure Container Instances][docs-container-instances] : une façon rapide de déployer et d’exécuter des conteneurs avec un cycle de vie court.</span><span class="sxs-lookup"><span data-stu-id="28df0-148">[Azure Container Instances][docs-container-instances] - A way of quickly deploying and running containers with a short lifecycle.</span></span> <span data-ttu-id="28df0-149">Les conteneurs ici sont généralement déployés pour exécuter une tâche de traitement rapide, comme le traitement d’un message ou un calcul puis ils sont mis hors service une fois l’opération terminée.</span><span class="sxs-lookup"><span data-stu-id="28df0-149">Containers here are usually deployed to run a quick processing job such as processing a message or performing a calculation and then deprovisioned as soon as they are complete.</span></span>
* <span data-ttu-id="28df0-150">[Service Bus][service bus] peut être utilisé à la place de la file d'attente de stockage.</span><span class="sxs-lookup"><span data-stu-id="28df0-150">[Service Bus][service-bus] could be used in place of Storage Queue's.</span></span>

<span data-ttu-id="28df0-151">D’autres options pour la couche Données incluent :</span><span class="sxs-lookup"><span data-stu-id="28df0-151">Other options for the data tier include:</span></span>

* <span data-ttu-id="28df0-152">[Cosmos DB][docs-cosmosdb] : un service de base de données multimodèle mondialement distribué de Microsoft.</span><span class="sxs-lookup"><span data-stu-id="28df0-152">[Cosmos DB][docs-cosmosdb] - Microsoft's globally distributed, multi-model database.</span></span> <span data-ttu-id="28df0-153">Cela fournit une plateforme pour exécuter d’autres modèles de données tels que Mongodb, Cassandra, des données graphiques ou un stockage de tables simple.</span><span class="sxs-lookup"><span data-stu-id="28df0-153">This provides a platform to run other data models such as Mongo DB, Cassandra, Graph data, or simple table storage.</span></span>

## <a name="considerations"></a><span data-ttu-id="28df0-154">Considérations</span><span class="sxs-lookup"><span data-stu-id="28df0-154">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="28df0-155">Disponibilité</span><span class="sxs-lookup"><span data-stu-id="28df0-155">Availability</span></span>

* <span data-ttu-id="28df0-156">Envisagez l’utilisation des [modèles de conception standards pour la disponibilité][design-patterns-availability] lorsque vous générez votre application cloud.</span><span class="sxs-lookup"><span data-stu-id="28df0-156">Consider leveraging the [typical design patterns for availability][design-patterns-availability] when building your cloud application.</span></span>
* <span data-ttu-id="28df0-157">Passez en revue les considérations sur la disponibilité dans [l’architecture de référence d’application web de App Service][app-service-reference-architecture] appropriée</span><span class="sxs-lookup"><span data-stu-id="28df0-157">Review the availability considerations in the appropriate [App Service web application reference architecture][app-service-reference-architecture]</span></span>
* <span data-ttu-id="28df0-158">Pour voir d’autres considérations relatives à la disponibilité, consultez la [liste de contrôle de la disponibilité][availability] dans le centre des architectures.</span><span class="sxs-lookup"><span data-stu-id="28df0-158">For additional considerations concerning availability, please see the [availability checklist][availability] in the architecture center.</span></span>

### <a name="scalability"></a><span data-ttu-id="28df0-159">Extensibilité</span><span class="sxs-lookup"><span data-stu-id="28df0-159">Scalability</span></span>

* <span data-ttu-id="28df0-160">Lors de la création d’une application cloud, soyez conscient des [modèles de conception standards pour l’évolutivité][design-patterns-scalability].</span><span class="sxs-lookup"><span data-stu-id="28df0-160">When building a cloud application be aware of the [typical design patterns for scalability][design-patterns-scalability].</span></span>
* <span data-ttu-id="28df0-161">Passez en revue les considérations sur l’extensibilité dans [l’architecture de référence d’application web de App Service][app-service-reference-architecture] appropriée</span><span class="sxs-lookup"><span data-stu-id="28df0-161">Review the scalability considerations in the appropriate [App Service web application reference architecture][app-service-reference-architecture]</span></span>
* <span data-ttu-id="28df0-162">Pour consulter d’autres rubriques relatives à l’extensibilité, consultez la [liste de contrôle de l’extensibilité][scalability] dans le centre des architectures.</span><span class="sxs-lookup"><span data-stu-id="28df0-162">For other scalability topics please see the [scalability checklist][scalability] available in the architecture center.</span></span>

### <a name="security"></a><span data-ttu-id="28df0-163">Sécurité</span><span class="sxs-lookup"><span data-stu-id="28df0-163">Security</span></span>

* <span data-ttu-id="28df0-164">Envisagez l’utilisation des [modèles de conception standards pour la sécurité][design-patterns-security] le cas échéant.</span><span class="sxs-lookup"><span data-stu-id="28df0-164">Consider leveraging the [typical design patterns for security][design-patterns-security] where appropriate.</span></span>
* <span data-ttu-id="28df0-165">Passez en revue les considérations sur la sécurité dans [l’architecture de référence d’application web de App Service][app-service-reference-architecture] appropriée.</span><span class="sxs-lookup"><span data-stu-id="28df0-165">Review the security considerations in the appropriate [App Service web application reference architecture][app-service-reference-architecture].</span></span>
* <span data-ttu-id="28df0-166">Prenez en compte le suivi d’un processus de [cycle de vie de développement sécurisé][secure-development] pour aider les développeurs à créer des logiciels plus sécurisés et gérer les exigences de conformité de sécurité, tout en réduisant les coûts de développement.</span><span class="sxs-lookup"><span data-stu-id="28df0-166">Consider following a [secure development lifecycle][secure-development] process to help developers build more secure software and address security compliance requirements while reducing development cost.</span></span>
* <span data-ttu-id="28df0-167">Passez en revue le plan d’architecture pour [la conformité avec Azure PCI DSS][pci-dss-blueprint].</span><span class="sxs-lookup"><span data-stu-id="28df0-167">Review the blueprint architecture for [Azure PCI DSS compliance][pci-dss-blueprint].</span></span>

### <a name="resiliency"></a><span data-ttu-id="28df0-168">Résilience</span><span class="sxs-lookup"><span data-stu-id="28df0-168">Resiliency</span></span>

* <span data-ttu-id="28df0-169">Envisagez l’utilisation du [modèle disjoncteur][circuit-breaker] pour permettre la gestion des erreurs sans perte de données dans le cas où une partie de l’application ne serait pas disponible.</span><span class="sxs-lookup"><span data-stu-id="28df0-169">Consider leveraging the [circuit breaker pattern][circuit-breaker] to provide graceful error handling should one part of the application not be available.</span></span>
* <span data-ttu-id="28df0-170">Examinez les [modèles de conception standards pour la résilience][design-patterns-resiliency] et envisagez de les implémenter le cas échéant.</span><span class="sxs-lookup"><span data-stu-id="28df0-170">Review the [typical design patterns for resiliency][design-patterns-resiliency] and consider implementing these where appropriate.</span></span>
* <span data-ttu-id="28df0-171">Vous pouvez trouver plusieurs [pratiques recommandées de la résilience pour App Service][resiliency-app-service] dans le centre d’architecture.</span><span class="sxs-lookup"><span data-stu-id="28df0-171">You can find a number of [resiliency recommended practices for App Service][resiliency-app-service] on the architecture center.</span></span>
* <span data-ttu-id="28df0-172">Envisagez d’utiliser la [géo-réplication][sql-geo-replication] active pour la couche Données et le stockage [géoredondant][storage-geo-redudancy] des images et des files d’attente.</span><span class="sxs-lookup"><span data-stu-id="28df0-172">Consider using active [geo-replication][sql-geo-replication] for the data tier and [geo-redundant][storage-geo-redudancy] storage for images and queues.</span></span>
* <span data-ttu-id="28df0-173">Pour plus d’informations sur la [résilience][resiliency], consultez l’article approprié dans le centre d’architecture.</span><span class="sxs-lookup"><span data-stu-id="28df0-173">For a deeper discussion on [resiliency][resiliency] please see the relevant article in the architecture center.</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="28df0-174">Déployez le scénario</span><span class="sxs-lookup"><span data-stu-id="28df0-174">Deploy the scenario</span></span>

<span data-ttu-id="28df0-175">Pour déployer ce scénario, vous pouvez suivre ce [didacticiel pas à pas][end-to-end-walkthrough] montrant comment déployer manuellement chaque composant.</span><span class="sxs-lookup"><span data-stu-id="28df0-175">To deploy this scenario, you can follow this [step-by-step tutorial][end-to-end-walkthrough] demonstrating how to manually deploy each component.</span></span> <span data-ttu-id="28df0-176">Ce didacticiel fournit également un exemple d’application .NET qui exécute une simple application d’achat de tickets.</span><span class="sxs-lookup"><span data-stu-id="28df0-176">This tutorial also provides a .NET sample application that runs a simple ticket purchasing application.</span></span> <span data-ttu-id="28df0-177">En outre, il existe un modèle ARM pour automatiser le déploiement de la plupart des ressources Azure.</span><span class="sxs-lookup"><span data-stu-id="28df0-177">Additionally, there is an ARM template to automate the deployment of most of the Azure resources.</span></span>

## <a name="pricing"></a><span data-ttu-id="28df0-178">Tarifs</span><span class="sxs-lookup"><span data-stu-id="28df0-178">Pricing</span></span>

<span data-ttu-id="28df0-179">Explorez le coût d’exécution de ce scénario, tous les services sont préconfigurés dans le calculateur de coûts.</span><span class="sxs-lookup"><span data-stu-id="28df0-179">Explore the cost of running this scenario, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="28df0-180">Pour pouvoir observer l’évolution de la tarification pour votre cas d’usage particulier, modifiez les variables appropriées en fonction du trafic que vous escomptez.</span><span class="sxs-lookup"><span data-stu-id="28df0-180">To see how the pricing would change for your particular use case change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="28df0-181">Nous proposons trois exemples de profils de coût basés sur la quantité de trafic que vous escomptez :</span><span class="sxs-lookup"><span data-stu-id="28df0-181">We have provided three sample cost profiles based on amount of traffic you expect to get:</span></span>

* <span data-ttu-id="28df0-182">[Petite][small-pricing] : cela représente les composants nécessaires pour générer la sortie pour une instance d’un niveau de production minimal.</span><span class="sxs-lookup"><span data-stu-id="28df0-182">[Small][small-pricing]: This represents the components necessary to build the out for a minimum production level instance.</span></span> <span data-ttu-id="28df0-183">Ici, nous supposons la présence d’une petite quantité d’utilisateurs, quelques milliers par mois.</span><span class="sxs-lookup"><span data-stu-id="28df0-183">Here we are assuming a small amount of users, numbering only in a few thousand per month.</span></span> <span data-ttu-id="28df0-184">L’application utilise une seule instance d’une application web standard qui sera suffisante pour permettre une mise à l’échelle automatique.</span><span class="sxs-lookup"><span data-stu-id="28df0-184">The app is using a single instance of a standard web app which will be enough to enable autoscaling.</span></span> <span data-ttu-id="28df0-185">Tous les autres composants sont mis à l’échelle à un niveau de base qui offrira des coûts minimaux mais qui assurera toujours une prise en charge du contrat SLA et une capacité suffisante pour gérer une charge de travail d’un niveau de production.</span><span class="sxs-lookup"><span data-stu-id="28df0-185">Each of the other components are scaled to a basic tier which will allow for a minimum amount of cost but still ensure that there is SLA support and enough capacity to handle a production level workload.</span></span>
* <span data-ttu-id="28df0-186">[Moyen][medium-pricing] : cela représente les composants indiquant un déploiement de taille modérée.</span><span class="sxs-lookup"><span data-stu-id="28df0-186">[Medium][medium-pricing]: This represents the components indicative of a moderate size deployment.</span></span> <span data-ttu-id="28df0-187">Ici, nous estimons environ 100 000 utilisateurs du système au cours d’un mois.</span><span class="sxs-lookup"><span data-stu-id="28df0-187">Here we estimate approximately 100,000 users using the system over the course of a month.</span></span> <span data-ttu-id="28df0-188">Le trafic attendu est géré dans une instance de service d’application unique avec un niveau standard modéré.</span><span class="sxs-lookup"><span data-stu-id="28df0-188">The expected traffic is handled in a single app service instance with a moderate standard tier.</span></span> <span data-ttu-id="28df0-189">En outre, des niveaux modérés des services Cognitive et de recherche sont ajoutés à la calculatrice.</span><span class="sxs-lookup"><span data-stu-id="28df0-189">Additionally, moderate tiers of cognitive and search services are added to the calculator.</span></span>
* <span data-ttu-id="28df0-190">[Grand][large-pricing] : cela représente une application destinée à grande échelle, de l’ordre de plusieurs millions d’utilisateurs par mois, déplaçant des téraoctets de données.</span><span class="sxs-lookup"><span data-stu-id="28df0-190">[Large][large-pricing]: This represents an application meant for high scale, at the order of millions of users per month moving terabytes of data.</span></span> <span data-ttu-id="28df0-191">À ce niveau de performances d’utilisation élevées, un niveau Premium est requis pour les applications web déployées dans plusieurs régions exposées par Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="28df0-191">At this level of usage high performance, premium tier web apps deployed in multiple regions fronted by traffic manager is required.</span></span> <span data-ttu-id="28df0-192">Les données incluent : le stockage, les bases de données et un réseau de distribution de contenu, qui sont configurés pour plusieurs téraoctets de données.</span><span class="sxs-lookup"><span data-stu-id="28df0-192">Data consists of the following: storage, databases, and CDN, are configured for terabytes of data.</span></span>

## <a name="related-resources"></a><span data-ttu-id="28df0-193">Ressources associées</span><span class="sxs-lookup"><span data-stu-id="28df0-193">Related Resources</span></span>

* <span data-ttu-id="28df0-194">[Architecture de référence pour une application web multirégion][multi-region-web-app]</span><span class="sxs-lookup"><span data-stu-id="28df0-194">[Reference Architecture for Multi-Region Web Application][multi-region-web-app]</span></span>
* <span data-ttu-id="28df0-195">[eShop sur l’exemple de référence de conteneurs][microservices-ecommerce]</span><span class="sxs-lookup"><span data-stu-id="28df0-195">[eShop on Containers Reference Example][microservices-ecommerce]</span></span>

<!-- links -->
[small-pricing]: https://azure.com/e/90fbb6a661a04888a57322985f9b34ac
[medium-pricing]: https://azure.com/e/38d5d387e3234537b6859660db1c9973
[large-pricing]: https://azure.com/e/f07f99b6c3134803a14c9b43fcba3e2f
[app-service-reference-architecture]: /azure/architecture/reference-architectures/app-service-web-app/
[architecture-diagram]: ./media/architecture-diagram-ecommerce-solution.png
[availability]: /azure/architecture/checklist/availability
[circuit-breaker]: /azure/architecture/patterns/circuit-breaker
[design-patterns-availability]: /azure/architecture/patterns/category/availability
[design-patterns-resiliency]: /azure/architecture/patterns/category/resiliency
[design-patterns-scalability]: /azure/architecture/patterns/category/performance-scalability
[design-patterns-security]: /azure/architecture/patterns/category/security
[docs-application-insights]: /azure/application-insights/app-insights-overview
[docs-b2c]: /azure/active-directory-b2c/active-directory-b2c-overview
[docs-cdn]: /azure/cdn/cdn-overview
[docs-container-instances]: /azure/container-instances/
[docs-kubernetes-service]: /azure/aks/
[docs-cosmosdb]: /azure/cosmos-db/
[docs-functions]: /azure/azure-functions/functions-overview
[docs-redis-cache]: /azure/redis-cache/cache-overview
[docs-search]: /azure/search/search-what-is-azure-search
[docs-service-fabric]: /azure/service-fabric/
[docs-sentiment-analysis]: /azure/cognitive-services/welcome
[docs-sql-database]: /azure/sql-database/sql-database-technical-overview
[docs-storage-blobs]: /azure/storage/blobs/storage-blobs-introduction
[docs-storage-queues]: /azure/storage/queues/storage-queues-introduction
[docs-traffic-manager]: /azure/traffic-manager/traffic-manager-overview
[docs-webapps]: /azure/app-service/app-service-web-overview
[end-to-end-walkthrough]: https://github.com/Azure/fta-customerfacingapps/tree/master/ecommerce/articles
[microservices-ecommerce]: https://github.com/dotnet-architecture/eShopOnContainers
[multi-region-web-app]: /azure/architecture/reference-architectures/app-service-web-app/multi-region
[pci-dss-blueprint]: /azure/security/blueprints/payment-processing-blueprint
[resiliency-app-service]: /azure/architecture/checklist/resiliency-per-service#app-service
[resiliency]: /azure/architecture/checklist/resiliency
[scalability]: /azure/architecture/checklist/scalability
[secure-development]: https://www.microsoft.com/en-us/SDL/process/design.aspx
[sql-geo-replication]: /azure/sql-database/sql-database-geo-replication-overview
[storage-geo-redudancy]: /azure/storage/common/storage-redundancy-grs