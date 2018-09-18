---
title: 'Adoption du cloud d’entreprise : Déployer une charge de travail de base'
description: Décrit comment déployer une charge de travail de base sur Azure
author: petertaylor9999
ms.date: 09/10/2018
ms.openlocfilehash: e615ba33fb713278a3695057e61d99c92b72b3f2
ms.sourcegitcommit: c49aeef818d7dfe271bc4128b230cfc676f05230
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/11/2018
ms.locfileid: "44389302"
---
# <a name="enterprise-cloud-adoption-deploy-a-basic-workload"></a><span data-ttu-id="ee257-103">Adoption du cloud d’entreprise : Déployer une charge de travail de base</span><span class="sxs-lookup"><span data-stu-id="ee257-103">Enterprise Cloud Adoption: deploy a basic workload</span></span>

<span data-ttu-id="ee257-104">Le terme **charge de travail** définit généralement une unité arbitraire de fonctionnalité, par exemple une application ou un service.</span><span class="sxs-lookup"><span data-stu-id="ee257-104">The term **workload** is typically understood to define an arbitrary unit of functionality such as an application or service.</span></span> <span data-ttu-id="ee257-105">Une charge de travail est envisagée sous l’angle des artefacts de code qui sont déployés sur un serveur, mais également sous l’angle de tous les autres services nécessaires.</span><span class="sxs-lookup"><span data-stu-id="ee257-105">We think about a workload in terms of the code artifacts that are deployed to a server, but also in terms of any other services that are necessary.</span></span> <span data-ttu-id="ee257-106">Cette définition se révèle pertinente dans le cas d’une application ou d’un service local(e), mais dans le cloud, elle mérite d’être développée.</span><span class="sxs-lookup"><span data-stu-id="ee257-106">This is a useful definition for an on-premises application or service but in the cloud we need to expand on it.</span></span>

<span data-ttu-id="ee257-107">Dans le cloud, une charge de travail englobe non seulement tous les artefacts, mais également les ressources cloud.</span><span class="sxs-lookup"><span data-stu-id="ee257-107">In the cloud, a workload not only encompasses all the artifacts but also includes the cloud resources as well.</span></span> <span data-ttu-id="ee257-108">Les ressources cloud sont ici incluses dans notre définition en raison d’un concept appelé infrastructure en tant que code.</span><span class="sxs-lookup"><span data-stu-id="ee257-108">We include cloud resources as part of our definition because of a concept known as infrastructure-as-code.</span></span> <span data-ttu-id="ee257-109">Comme vous l’avez appris dans l’article [Fonctionnement d’Azure](../getting-started/what-is-azure.md), les ressources dans Azure sont déployées par un service d’orchestrateur.</span><span class="sxs-lookup"><span data-stu-id="ee257-109">As you learned in the [how does Azure work?](../getting-started/what-is-azure.md), resources in Azure are deployed by an orchestrator service.</span></span> <span data-ttu-id="ee257-110">Le service d’orchestrateur expose cette fonctionnalité via une API web, laquelle peut être appelée à l’aide de plusieurs outils, tels que Powershell, l’interface de ligne de commande (CLI) d’Azure et le portail Azure.</span><span class="sxs-lookup"><span data-stu-id="ee257-110">The orchestrator service exposes this functionality through a web API, and this web API can be called using several tools such as Powershell, the Azure command line interface (CLI), and the Azure portal.</span></span> <span data-ttu-id="ee257-111">Cela signifie que nous pouvons spécifier nos ressources dans un fichier lisible sur ordinateur qui peut être stocké en même temps que les artefacts de code associés à notre application.</span><span class="sxs-lookup"><span data-stu-id="ee257-111">This means that we can specify our resources in a machine-readable file that can be stored along with the code artifacts associated with our application.</span></span>

<span data-ttu-id="ee257-112">Nous pouvons ainsi définir une charge de travail en termes d’artefacts de code et de ressources cloud nécessaires, ce qui nous permet d’isoler nos charges de travail.</span><span class="sxs-lookup"><span data-stu-id="ee257-112">This enables us to define a workload in terms of code artifacts and the necessary cloud resources, and this further enables us to isolate our workloads.</span></span> <span data-ttu-id="ee257-113">Les charges de travail peuvent être isolées par la façon dont les ressources sont organisées, par la topologie du réseau ou par d’autres attributs.</span><span class="sxs-lookup"><span data-stu-id="ee257-113">Workloads may be isolated by the way resources are organized, by network topology, or by other attributes.</span></span> <span data-ttu-id="ee257-114">L’isolement des charges de travail a pour but d’associer les ressources spécifiques d’une charge de travail à une équipe donnée, afin que cette équipe puisse gérer indépendamment tous les aspects de ces ressources.</span><span class="sxs-lookup"><span data-stu-id="ee257-114">The goal of workload isolation is to associate a workload's specific resources to a team so the team can independently manage all aspects of those resources.</span></span> <span data-ttu-id="ee257-115">Cela permet à plusieurs équipes de partager des services de gestion de ressources dans Azure tout en évitant une suppression ou une modification involontaire des ressources d’une autre équipe.</span><span class="sxs-lookup"><span data-stu-id="ee257-115">This enables multiple teams to share resource management services in Azure while preventing the unintentional deletion or modification of each other's resources.</span></span>

<span data-ttu-id="ee257-116">Cet isolement permet également de favoriser ce que l’on appelle le DevOps.</span><span class="sxs-lookup"><span data-stu-id="ee257-116">This isolation also enables another concept known as DevOps.</span></span> <span data-ttu-id="ee257-117">Le DevOps inclut les pratiques de développement logiciel couvrant le développement logiciel et les opérations informatiques décrits ci-dessus, mais en y ajoutant autant que possible le recours à l’automatisation.</span><span class="sxs-lookup"><span data-stu-id="ee257-117">DevOps includes the software development practices that include both software development and IT operations above, but adds the use of automation as much as possible.</span></span> <span data-ttu-id="ee257-118">Le principe d’intégration continue/livraison continue (CI/CD) est au cœur du DevOps.</span><span class="sxs-lookup"><span data-stu-id="ee257-118">One of the principles of DevOps is known as continuous integration and continuous delivery (CI/CD).</span></span> <span data-ttu-id="ee257-119">L’intégration continue fait référence aux processus de génération automatisés qui sont exécutés à chaque fois qu’un développeur valide une modification de code. La livraison continue renvoie à des processus automatisés qui déploient ce code vers différents environnements, par exemple un environnement de développement à des fins de test ou un environnement de production pour le déploiement final.</span><span class="sxs-lookup"><span data-stu-id="ee257-119">Continuous integration refers to the automated build processes that are run each time a developer commits a code change, and continuous delivery refers to the automated processes that deploy this code to various environments such as a development environment for testing or a production environment for final deployment.</span></span>

## <a name="basic-workload"></a><span data-ttu-id="ee257-120">Charge de travail de base</span><span class="sxs-lookup"><span data-stu-id="ee257-120">Basic workload</span></span>

<span data-ttu-id="ee257-121">Une **charge de travail de base** est généralement définie comme une application web unique, ou un réseau virtuel avec une machine virtuelle.</span><span class="sxs-lookup"><span data-stu-id="ee257-121">A **basic workload** is typically defined as a single web application, or a virtual network (VNet) with virtual machine (VM).</span></span> 

> [!NOTE]
> <span data-ttu-id="ee257-122">Ce guide ne couvre pas le développement d’applications.</span><span class="sxs-lookup"><span data-stu-id="ee257-122">This guide does not cover application development.</span></span> <span data-ttu-id="ee257-123">Pour plus d’informations sur le développement d’applications sur Azure, consultez le [Guide d’Architecture des Applications Azure](/azure/architecture/guide/).</span><span class="sxs-lookup"><span data-stu-id="ee257-123">For more information about developing applications on Azure, see the [Azure Application Architecture Guide](/azure/architecture/guide/).</span></span>

<span data-ttu-id="ee257-124">Que la charge de travail soit une application web ou une machine virtuelle, chacun de ces déploiements nécessite un **groupe de ressources**.</span><span class="sxs-lookup"><span data-stu-id="ee257-124">Regardless of whether the workload is a web application or a VM, each of these deployments requires a **resource group**.</span></span> <span data-ttu-id="ee257-125">Un utilisateur autorisé à créer un groupe de ressources doit le faire avant de suivre les étapes ci-dessous.</span><span class="sxs-lookup"><span data-stu-id="ee257-125">A user with permission to create a resource group must do that before following the steps below.</span></span>

## <a name="basic-web-application-paas"></a><span data-ttu-id="ee257-126">Application web de base (PaaS)</span><span class="sxs-lookup"><span data-stu-id="ee257-126">Basic web application (PaaS)</span></span>

<span data-ttu-id="ee257-127">Pour une application web de base, sélectionnez un des Démarrages rapides de 5 minutes à partir de la [documentation web apps](/azure/app-service?toc=/azure/architecture/cloud-adoption-guide/toc.json) et suivez les étapes.</span><span class="sxs-lookup"><span data-stu-id="ee257-127">For a basic web application, select one of the 5-minute quickstarts from the [web apps documentation](/azure/app-service?toc=/azure/architecture/cloud-adoption-guide/toc.json) and follow the steps.</span></span> 

> [!NOTE]
> <span data-ttu-id="ee257-128">Certaines des Démarrages rapides déploieront un groupe de ressources par défaut.</span><span class="sxs-lookup"><span data-stu-id="ee257-128">Some of the quickstarts will deploy a resource group by default.</span></span> <span data-ttu-id="ee257-129">Dans ce cas, il n’est pas nécessaire de créer un groupe de ressources de façon explicite.</span><span class="sxs-lookup"><span data-stu-id="ee257-129">In this case, it's not necessary to create a resource group explicitly.</span></span> <span data-ttu-id="ee257-130">Vous pouvez également déployer l’application web pour le groupe de ressources créé ci-dessus.</span><span class="sxs-lookup"><span data-stu-id="ee257-130">Otherwise, deploy the web application to the resource group created above.</span></span>

<span data-ttu-id="ee257-131">Une fois que votre charge de travail simple a été déployée, vous pouvez trouver plus d’informations sur les pratiques éprouvées pour le déploiement d’une [application web de base](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json) dans Azure.</span><span class="sxs-lookup"><span data-stu-id="ee257-131">Once your simple workload has been deployed, you can learn more about the proven practices for deploying a [basic web application](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json) to Azure.</span></span>

## <a name="single-windows-or-linux-vm-iaas"></a><span data-ttu-id="ee257-132">Machine virtuelle Windows ou Linux simple (IaaS)</span><span class="sxs-lookup"><span data-stu-id="ee257-132">Single Windows or Linux VM (IaaS)</span></span>

<span data-ttu-id="ee257-133">Pour une charge de travail simple exécutable sur une machine virtuelle, la première étape consiste à déployer un réseau virtuel.</span><span class="sxs-lookup"><span data-stu-id="ee257-133">For a simple workload that runs on a virtual machine, the first step is to deploy a virtual network.</span></span> <span data-ttu-id="ee257-134">Toutes les ressources IaaS dans Azure telles que les machines virtuelles, les équilibreurs de charge et les passerelles nécessitent un réseau virtuel.</span><span class="sxs-lookup"><span data-stu-id="ee257-134">All IaaS resources in Azure such as virtual machines, load balancers, and gateways require a virtual network.</span></span> <span data-ttu-id="ee257-135">Une fois que vous en saurez plus sur [les réseaux virtuels Azure](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json), suivez les étapes pour [déployer un réseau virtuel dans Azure à l’aide du portail](/azure/virtual-network/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json).</span><span class="sxs-lookup"><span data-stu-id="ee257-135">Learn about [Azure virtual networks](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json), and then follow the steps to [deploy a Virtual Network to Azure using the portal](/azure/virtual-network/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json).</span></span> <span data-ttu-id="ee257-136">Lorsque vous spécifiez les paramètres pour le réseau virtuel dans le portail Azure, spécifiez le nom du groupe de ressources créé ci-dessus.</span><span class="sxs-lookup"><span data-stu-id="ee257-136">When you specify the settings for the virtual network in the Azure portal, specify the name of the resource group created above.</span></span>

<span data-ttu-id="ee257-137">L’étape suivante consiste à décider s’il faut déployer une machine virtuelle simple Windows ou Linux.</span><span class="sxs-lookup"><span data-stu-id="ee257-137">The next step is to decide whether to deploy a single Windows or Linux VM.</span></span> <span data-ttu-id="ee257-138">[Pour une machine virtuelle Windows,déployer une machine virtuelle Windows sur Azure à l’aide du portail](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json).</span><span class="sxs-lookup"><span data-stu-id="ee257-138">For a Windows VM, follow the steps to [deploy a Windows VM to Azure with the portal](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json).</span></span> <span data-ttu-id="ee257-139">Là encore, lorsque vous spécifiez les paramètres de la machine virtuelle dans le portail Azure, spécifiez le nom du groupe de ressources créé ci-dessus.</span><span class="sxs-lookup"><span data-stu-id="ee257-139">Again, when you specify the settings for the virtual machine in the Azure portal, specify the name of the resource group created above.</span></span>

<span data-ttu-id="ee257-140">Une fois que vous avez suivi les étapes et déployé la machine virtuelle, vous pouvez en savoir plus sur les [pratiques éprouvées pour l’exécution d’une machine virtuelle Windows dans Azure](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json).</span><span class="sxs-lookup"><span data-stu-id="ee257-140">Once you've followed the steps and deployed the VM, you can learn about [proven practices for running a Windows VM on Azure](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json).</span></span> <span data-ttu-id="ee257-141">[Pour une machine virtuelle Linux, déployer une machine virtuelle Linux dans Azure à l’aide du portail](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json).</span><span class="sxs-lookup"><span data-stu-id="ee257-141">For a Linux VM, follow the steps to [deploy a Linux VM to Azure with the portal](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json).</span></span> <span data-ttu-id="ee257-142">Vous pouvez également en apprendre davantage sur les [pratiques éprouvées pour l’exécution d’une machine virtuelle Linux dans Azure](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json).</span><span class="sxs-lookup"><span data-stu-id="ee257-142">You can also learn more about [proven practices for running a Linux VM on Azure](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json).</span></span>